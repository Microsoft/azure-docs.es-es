---
title: Cifrado doble de datos en reposo
titleSuffix: Azure HDInsight
description: En este artículo se describen los dos niveles de cifrado disponibles para los datos en reposo en los clústeres de Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: hrasheed
ms.service: hdinsight
ms.topic: conceptual
ms.date: 08/10/2020
ms.openlocfilehash: 97d899d73359cc45daf88940b815ed262c3b4766
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89290844"
---
# <a name="azure-hdinsight-double-encryption-for-data-at-rest"></a>Cifrado doble de datos en reposo en Azure HDInsight

En este artículo se analizan los métodos para el cifrado de datos en reposo en clústeres de Azure HDInsight. El cifrado de datos en reposo hace referencia al cifrado en discos administrados (discos de datos, discos de sistema operativo y discos temporales) conectados a máquinas virtuales de clústeres de HDInsight. 

En este documento no se tratan los datos almacenados en la cuenta de Azure Storage. Los clústeres pueden tener una o varias cuentas de Azure Storage asociadas, en las que el cliente o Microsoft también puede administrar las claves de cifrado, sin embargo, el servicio de cifrado es diferente. Para más información sobre el cifrado de almacenamiento, consulte [Cifrado de Azure Storage para datos en reposo](../storage/common/storage-service-encryption.md).

## <a name="introduction"></a>Introducción

Hay tres roles principales de discos administrados en Azure: el disco de datos, el disco de sistema operativo y el disco temporal. Para obtener más información sobre los distintos tipos de discos administrados, consulte [Introducción a los discos administrados de Azure](https://docs.microsoft.com/azure/virtual-machines/windows/managed-disks-overview). 

HDInsight admite varios tipos de cifrado en dos niveles diferentes:

- Cifrado del lado del servidor (SSE): SSE se realiza mediante el servicio de almacenamiento. En HDInsight, SSE se usa para cifrar los discos de sistema operativo y los discos de datos. Se habilita de forma predeterminada. SSE es un servicio de cifrado de nivel 1.
- Cifrado en el host con una clave administrada por la plataforma: Similar a SSE, el servicio de almacenamiento realiza este tipo de cifrado. Sin embargo, solo se aplica a los discos temporales y no está habilitado de manera predeterminada. El cifrado en el host también es un servicio de cifrado de nivel 1.
- Cifrado en reposo con una clave administrada por el cliente: Este tipo de cifrado se puede usar en discos temporales y de datos. No está habilitado de manera predeterminada y requiere que el cliente proporcione su propia clave a través de Azure Key Vault. El cifrado en el host es un servicio de cifrado de nivel 2.

Estos tipos se resumen en la siguiente tabla.

|Tipo de clúster |Disco del sistema operativo (disco administrado) |Disco de datos (disco administrado) |Disco de datos temporales (SSD local) |
|---|---|---|---|
|Kafka, HBase con escrituras aceleradas|Nivel 1: [Cifrado SSE](https://docs.microsoft.com/azure/virtual-machines/windows/managed-disks-overview#encryption) de manera predeterminada|Nivel 1: [Cifrado SSE](https://docs.microsoft.com/azure/virtual-machines/windows/managed-disks-overview#encryption) de manera predeterminada. Nivel 2: Cifrado opcional en reposo con CMK|Nivel 1: Cifrado opcional en reposo con PMK. Nivel 2: Cifrado opcional en reposo con CMK|
|Todos los demás clústeres (Spark, Interactive, Hadoop, HBase sin escrituras aceleradas)|Nivel 1: [Cifrado SSE](https://docs.microsoft.com/azure/virtual-machines/windows/managed-disks-overview#encryption) de manera predeterminada|N/D|Nivel 1: Cifrado opcional en reposo con PMK. Nivel 2: Cifrado opcional en reposo con CMK|

## <a name="encryption-at-rest-using-customer-managed-keys"></a>Cifrado en reposo con claves administradas por el cliente

El cifrado de claves administradas por el cliente consiste en un proceso de un paso controlado durante la creación del clúster sin ningún costo adicional. Solo necesita autorizar una identidad administrada en Azure Key Vault y agregar la clave de cifrado al crear el clúster.

Tanto los discos de datos como los discos temporales de cada nodo del clúster están cifrados con una clave de cifrado de datos (DEK) simétrica. La DEK está protegida mediante la clave de cifrado de claves (KEK) del almacén de claves. Azure HDInsight controla completamente los procesos de cifrado y descifrado.

En el caso de los discos de sistema operativo conectados a las VM del clúster, solo hay disponible un nivel de cifrado (PMK). Se recomienda que los clientes eviten copiar datos confidenciales en discos del sistema operativo si en sus escenarios se requiere un cifrado CMK.

Si el firewall del almacén de claves está habilitado en el almacén de claves donde está almacenada la clave de cifrado del disco, las direcciones IP del proveedor de recursos regional de HDInsight de la región donde se va a implementar el clúster se deben agregar a la configuración del firewall del almacén de claves. Esto es necesario porque HDInsight no es un servicio de almacén de claves de Azure de confianza.

Puede usar Azure Portal o la CLI de Azure para rotar las claves en el almacén de claves de forma segura. Cuando se gira una clave, el clúster de HDInsight comienza a usar la clave nueva en cuestión de minutos. Habilite las características de protección de claves [Eliminación temporal](../key-vault/general/soft-delete-overview.md) para protegerse frente a los escenarios de ransomware y la eliminación accidental. No se admiten almacenes de claves sin esta característica de protección.

### <a name="get-started-with-customer-managed-keys"></a>Introducción a las claves administradas por el cliente

Para crear un clúster de HDInsight habilitado para claves administradas por el cliente, repasaremos los pasos siguientes:

1. Creación de identidades administradas para los recursos de Azure
1. Crear una instancia de Azure Key Vault
1. Crear clave
1. Creación de directivas de acceso
1. Creación de un clúster de HDInsight con la clave administrada por el cliente habilitada
1. Rotación de la clave de cifrado

Cada paso se explica en detalle en una de las siguientes secciones.

### <a name="create-managed-identities-for-azure-resources"></a>Creación de identidades administradas para los recursos de Azure

Cree una identidad administrada asignada por el usuario para autenticarse en Key Vault.

Consulte [Creación de una identidad administrada asignada por el usuario](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md) para conocer los pasos específicos. Para obtener más información sobre cómo funcionan las identidades administradas en Azure HDInsight, vea [Identidades administradas en Azure HDInsight](hdinsight-managed-identities.md). Asegúrese de guardar el identificador de recurso de identidad administrada para cuando lo agregue a la directiva de acceso de Key Vault.

### <a name="create-azure-key-vault"></a>Crear una instancia de Azure Key Vault

Cree un almacén de claves. Consulte [Creación de una instancia de Azure Key Vault](../key-vault/secrets/quick-create-portal.md) para conocer los pasos específicos.

HDInsight solo es compatible con Azure Key Vault. Si tiene su propio almacén de claves, puede importar las claves a Azure Key Vault. Recuerde que el almacén de claves debe tener habilitada la **eliminación temporal**. Para obtener más información sobre cómo importar claves existentes, visite [Información acerca de claves, secretos y certificados](../key-vault/about-keys-secrets-and-certificates.md).

### <a name="create-key"></a>Crear clave

1. En el nuevo almacén de claves, vaya a **Configuración** > **Claves** >  **+ Generate/Import** (+ Generar/Importar).

    ![Generación de una clave nueva en Azure Key Vault](./media/disk-encryption/create-new-key.png "Generar una clave nueva en Azure Key Vault")

1. Proporcione un nombre y, luego, seleccione **Crear**. Mantenga el valor de **Tipo de clave** en **RSA**.

    ![generación de un nombre de clave](./media/disk-encryption/create-key.png "Generación de un nombre de clave")

1. Cuando vuelva a la página **Claves**, seleccione la clave que ha creado.

    ![lista de claves de Key Vault](./media/disk-encryption/key-vault-key-list.png)

1. Seleccione la versión para abrir la página **Versión de la clave**. Cuando usa su propia clave para el cifrado del clúster de HDInsight, tiene que proporcionar el URI de la clave. Copie el **identificador de clave** y guárdelo en algún lugar hasta que vaya a crear el clúster.

    ![obtención del identificador de clave](./media/disk-encryption/get-key-identifier.png)

### <a name="create-access-policy"></a>Creación de directivas de acceso

1. En el nuevo almacén de claves, vaya a **Configuración** > **Directivas de acceso** >  **+ Agregar directiva de acceso**.

    ![Creación de una directiva de acceso de Azure Key Vault](./media/disk-encryption/key-vault-access-policy.png)

1. En la página **Agregar directiva de acceso**, proporcione la siguiente información:

    |Propiedad |Descripción|
    |---|---|
    |Permisos de claves|Seleccione **Obtener**, **Desencapsular clave** y **Encapsular clave**.|
    |Permisos de secretos|Seleccione **Obtener**, **Establecer** y **Eliminar**.|
    |Selección de la entidad de seguridad|Seleccione la identidad administrada asignada por el usuario que creó anteriormente.|

    ![Establecimiento de la entidad de seguridad para la directiva de acceso de Azure Key Vault](./media/disk-encryption/azure-portal-add-access-policy.png)

1. Seleccione **Agregar**.

1. Seleccione **Guardar**.

    ![Guardar la directiva de acceso de Azure Key Vault](./media/disk-encryption/add-key-vault-access-policy-save.png)

### <a name="create-cluster-with-customer-managed-key-disk-encryption"></a>Creación de un clúster con el cifrado de disco mediante claves administradas por el cliente

Ya está listo para crear un clúster de HDInsight. Las claves administradas por el cliente solo pueden aplicarse a los clústeres nuevos durante la creación del clúster. No se puede quitar el cifrado de los clústeres de las claves administradas por el cliente, y no se pueden agregar claves administradas por el cliente a los clústeres existentes.

#### <a name="using-the-azure-portal"></a>Uso de Azure Portal

Durante la creación del clúster, proporcione el **identificador de clave** completo, incluida la versión de la clave. Por ejemplo, `https://contoso-kv.vault.azure.net/keys/myClusterKey/46ab702136bc4b229f8b10e8c2997fa4`. También deberá asignar la identidad administrada al clúster y proporcionar el URI de la clave.

![Creación de un clúster](./media/disk-encryption/create-cluster-portal.png)

#### <a name="using-azure-cli"></a>Uso de la CLI de Azure

En el ejemplo siguiente se muestra cómo usar la CLI de Azure para crear un nuevo clúster de Apache Spark con el cifrado de disco habilitado. Para más información, consulte el comando [az hdinsight create de la CLI de Azure](https://docs.microsoft.com/cli/azure/hdinsight?view=azure-cli-latest#az-hdinsight-create).

```azurecli
az hdinsight create -t spark -g MyResourceGroup -n MyCluster \
-p "HttpPassword1234!" --workernode-data-disks-per-node 2 \
--storage-account MyStorageAccount \
--encryption-key-name SparkClusterKey \
--encryption-key-version 00000000000000000000000000000000 \
--encryption-vault-uri https://MyKeyVault.vault.azure.net \
--assign-identity MyMSI
```

#### <a name="using-azure-resource-manager-templates"></a>Uso de plantillas del Administrador de recursos de Azure

En el ejemplo siguiente se muestra cómo usar una plantilla de Azure Resource Manager para crear un nuevo clúster de Apache Spark con el cifrado de disco habilitado. Para obtener más información, vea [¿Qué son las plantillas de ARM?](https://docs.microsoft.com/azure/azure-resource-manager/templates/overview)

En este ejemplo se usa PowerShell para llamar a la plantilla.

```powershell
$templateFile = "azuredeploy.json"
$ResourceGroupName = "MyResourceGroup"
$clusterName = "MyCluster"
$password = ConvertTo-SecureString 'HttpPassword1234!' -AsPlainText -Force
$diskEncryptionVaultUri = "https://MyKeyVault.vault.azure.net"
$diskEncryptionKeyName = "SparkClusterKey"
$diskEncryptionKeyVersion = "00000000000000000000000000000000"
$managedIdentityName = "MyMSI"

New-AzResourceGroupDeployment `
  -Name mySpark `
  -TemplateFile $templateFile `
  -ResourceGroupName $ResourceGroupName `
  -clusterName $clusterName `
  -clusterLoginPassword $password `
` -sshPassword $password `
  -diskEncryptionVaultUri $diskEncryptionVaultUri `
  -diskEncryptionKeyName $diskEncryptionKeyName `
  -diskEncryptionKeyVersion $diskEncryptionKeyVersion `
  -managedIdentityName $managedIdentityName
```

El contenido de la plantilla de administración de recursos, `azuredeploy.json`:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "0.9.0.0",
  "parameters": {
    "clusterName": {
      "type": "string",
      "metadata": {
        "description": "The name of the HDInsight cluster to create."
      }
    },
    "clusterLoginUserName": {
      "type": "string",
      "defaultValue": "admin",
      "metadata": {
        "description": "These credentials can be used to submit jobs to the cluster and to log into cluster dashboards."
      }
    },
    "clusterLoginPassword": {
      "type": "securestring",
      "metadata": {
        "description": "The password must be at least 10 characters in length and must contain at least one digit, one non-alphanumeric character, and one upper or lower case letter."
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "The location where all azure resources will be deployed."
      }
    },
    "sshUserName": {
      "type": "string",
      "defaultValue": "sshuser",
      "metadata": {
        "description": "These credentials can be used to remotely access the cluster."
      }
    },
    "sshPassword": {
      "type": "securestring",
      "metadata": {
        "description": "The password must be at least 10 characters in length and must contain at least one digit, one non-alphanumeric character, and one upper or lower case letter."
      }
    },
    "headNodeSize": {
      "type": "string",
      "defaultValue": "Standard_D12_v2",
      "metadata": {
        "description": "The VM size of the head nodes."
      }
    },
    "workerNodeSize": {
      "type": "string",
      "defaultValue": "Standard_D13_v2",
      "metadata": {
        "description": "The VM size of the worker nodes."
      }
    },
    "diskEncryptionVaultUri": {
      "type": "string",
      "metadata": {
        "description": "The Key Vault DNSname."
      }
    },
    "diskEncryptionKeyName": {
      "type": "string",
      "metadata": {
        "description": "The Key Vault key name."
      }
    },
    "diskEncryptionKeyVersion": {
      "type": "string",
      "metadata": {
        "description": "The Key Vault key version for the selected key."
      }
    },
    "managedIdentityName": {
      "type": "string",
      "metadata": {
        "description": "The user-assigned managed identity."
      }
    }
  },
  "variables": {
    "defaultStorageAccount": {
      "name": "[uniqueString(resourceGroup().id)]",
      "type": "Standard_LRS"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('defaultStorageAccount').name]",
      "location": "[parameters('location')]",
      "apiVersion": "2019-06-01",
      "sku": {
        "name": "[variables('defaultStorageAccount').type]"
      },
      "kind": "Storage",
      "properties": {}
    },
    {
      "apiVersion": "2018-06-01-preview",
      "name": "[parameters('clusterName')]",
      "type": "Microsoft.HDInsight/clusters",
      "location": "[parameters('location')]",
      "properties": {
        "clusterVersion": "3.6",
        "osType": "Linux",
        "tier": "standard",
        "clusterDefinition": {
          "kind": "spark",
          "componentVersion": {
            "Spark": "2.3"
          },
          "configurations": {
            "gateway": {
              "restAuthCredential.isEnabled": true,
              "restAuthCredential.username": "[parameters('clusterLoginUserName')]",
              "restAuthCredential.password": "[parameters('clusterLoginPassword')]"
            }
          }
        },
        "storageProfile": {
          "storageaccounts": [
            {
              "name": "[replace(replace(reference(resourceId('Microsoft.Storage/storageAccounts', variables('defaultStorageAccount').name), '2019-06-01').primaryEndpoints.blob,'https://',''),'/','')]",
              "isDefault": true,
              "container": "[parameters('clusterName')]",
              "key": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('defaultStorageAccount').name), '2019-06-01').keys[0].value]"
            }
          ]
        },
        "computeProfile": {
          "roles": [
            {
              "name": "headnode",
              "minInstanceCount": 1,
              "targetInstanceCount": 2,
              "hardwareProfile": {
                "vmSize": "[parameters('headNodeSize')]"
              },
              "osProfile": {
                "linuxOperatingSystemProfile": {
                  "username": "[parameters('sshUserName')]",
                  "password": "[parameters('sshPassword')]"
                },
              },
            },
            {
              "name": "workernode",
              "targetInstanceCount": 1,
              "hardwareProfile": {
                "vmSize": "[parameters('workerNodeSize')]"
              },
              "osProfile": {
                "linuxOperatingSystemProfile": {
                  "username": "[parameters('sshUserName')]",
                  "password": "[parameters('sshPassword')]"
                },
              },
            }
          ]
        },
        "minSupportedTlsVersion": "1.2",
        "diskEncryptionProperties": {
          "vaultUri": "[parameters('diskEncryptionVaultUri')]",
          "keyName": "[parameters('diskEncryptionKeyName')]",
          "keyVersion": "[parameters('diskEncryptionKeyVersion')]",
          "msiResourceId": "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('managedIdentityName'))]"
        }
      },
      "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
          "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('managedIdentityName'))]": {}
        }
      }
    }
  ]
}
```

### <a name="rotating-the-encryption-key"></a>Rotación de la clave de cifrado

Puede haber escenarios en los que quiera cambiar las claves de cifrado que se usan en el clúster de HDInsight una vez creado. Esta tarea se puede realizar fácilmente en el portal. En esta operación, el clúster debe tener acceso a la clave actual y a la clave nueva deseada; si no, se producirá un error en la operación de rotación de la clave.

#### <a name="using-the-azure-portal"></a>Uso de Azure Portal

Para rotar la clave, necesita el URI del almacén de claves base. Una vez lo haya hecho, vaya a la sección de propiedades del clúster de HDInsight en el portal y haga clic en **Cambiar clave** en **URL de clave de cifrado de disco**. Escriba la dirección URL de la nueva clave y envíela para rotar la clave.

![rotación de la clave de cifrado de disco](./media/disk-encryption/change-key.png)

#### <a name="using-azure-cli"></a>Uso de la CLI de Azure

En el ejemplo siguiente se muestra cómo girar la clave de cifrado de disco de un clúster de HDInsight existente. Para más información, consulte el comando [az hdinsight rotate-disk-encryption-key de la CLI de Azure](https://docs.microsoft.com/cli/azure/hdinsight?view=azure-cli-latest#az-hdinsight-rotate-disk-encryption-key).

```azurecli
az hdinsight rotate-disk-encryption-key \
--encryption-key-name SparkClusterKey \
--encryption-key-version 00000000000000000000000000000000 \
--encryption-vault-uri https://MyKeyVault.vault.azure.net \
--name MyCluster \
--resource-group MyResourceGroup
```

## <a name="faq-for-customer-managed-key-encryption"></a>Preguntas más frecuentes del cifrado de claves administradas por el cliente

**¿Cómo accede el clúster de HDInsight a mi almacén de claves?**

HDInsight accede a su instancia de Azure Key Vault mediante la identidad administrada que se asocia con el clúster de HDInsight. Esta identidad administrada puede crearse antes o durante la creación del clúster. También debe conceder acceso a la identidad administrada para que pueda usar el almacén de claves donde se almacena la clave.

**¿Esta característica está disponible para todos los clústeres de HDInsight?**

El cifrado de claves administradas por el cliente está disponible para todos los tipos de clúster, excepto Spark 2.1 y 2.2.

**¿Puedo usar varias claves para cifrar diferentes discos o carpetas?**

No, todos los discos administrados y los discos de recursos se cifran con la misma clave.

**¿Qué ocurre si el clúster pierde el acceso al almacén de claves o a la clave?**

Si el clúster pierde acceso a la clave, se mostrarán advertencias en el portal de Apache Ambari. En este estado, la operación **Cambiar clave** dará error. Cuando se restaura el acceso a la clave, las advertencias de Ambari desaparecen y operaciones como la rotación de claves se pueden realizar correctamente.

![alerta de Ambari de acceso a la clave](./media/disk-encryption/ambari-alert.png)

**¿Cómo puedo recuperar el clúster si se eliminan las claves?**

Dado que solo se admiten claves habilitadas para eliminación temporal, si las claves se recuperan en el almacén de claves, el clúster debe volver a acceder a ellas. Para recuperar una clave de Azure Key Vault, consulte [Undo-AzKeyVaultKeyRemoval](/powershell/module/az.keyvault/Undo-AzKeyVaultKeyRemoval) o [az-keyvault-key-recover](/cli/azure/keyvault/key?view=azure-cli-latest#az-keyvault-key-recover).


**Si un clúster se escala verticalmente, ¿los nuevos nodos admitirán las claves administradas por el cliente sin problemas?**

Sí. El clúster necesita tener acceso a la clave del almacén de claves durante el escalado vertical. Se usa la misma clave para cifrar los discos administrados y de recursos en el clúster.

**Las claves administradas por el cliente están disponibles en mi ubicación?**

Las claves administradas por el cliente de HDInsight están disponibles en todas las nubes públicas y nubes nacionales.

## <a name="encryption-at-host-using-platform-managed-keys"></a>Cifrado en el host con claves administrada por la plataforma

### <a name="enable-in-the-azure-portal"></a>Habilitación en Azure Portal

El cifrado en el host se puede habilitar durante la creación del clúster en Azure Portal.

> [!Note]
> Cuando el cifrado en el host está habilitado, no se pueden agregar aplicaciones al clúster de HDInsight desde Azure Marketplace.

:::image type="content" source="media/disk-encryption/encryption-at-host.png" alt-text="Habilite el cifrado en el host.":::

Esta opción habilita el [cifrado en el host](../virtual-machines/linux/disks-enable-host-based-encryption-portal.md) para los discos de datos temporales de VM de HDInsight que usan PMK. El cifrado en el host solo [se admite en determinadas SKU de VM en regiones limitadas](../virtual-machines/linux/disks-enable-host-based-encryption-portal.md), y HDInsight admite las [siguientes SKU y configuraciones de nodo](./hdinsight-supported-node-configuration.md).

Para comprender el tamaño correcto de VM para el clúster de HDInsight, consulte [Selección del tamaño de VM correcto para el clúster de Azure HDInsight](hdinsight-selecting-vm-size.md). La SKU de VM predeterminada para el nodo Zookeeper cuando el cifrado en el host esté habilitado será DS2V2.

### <a name="enable-using-powershell"></a>Habilitar mediante PowerShell

En el fragmento de código siguiente se muestra cómo puede crear un nuevo clúster de Azure HDInsight con cifrado en host habilitado mediante PowerShell. Se usa el parámetro `-EncryptionAtHost $true` para habilitar la característica.

```powershell
$storageAccountResourceGroupName = "Group"
$storageAccountName = "yourstorageacct001"
$storageAccountKey = Get-AzStorageAccountKey `
    -ResourceGroupName $storageAccountResourceGroupName `
    -Name $storageAccountName | %{ $_.Key1 }
$storageContainer = "container002"
# Cluster configuration info
$location = "East US 2"
$clusterResourceGroupName = "Group"
$clusterName = "your-hadoop-002"
$clusterCreds = Get-Credential
# If the cluster's resource group doesn't exist yet, run:
# New-AzResourceGroup -Name $clusterResourceGroupName -Location $location
# Create the cluster
New-AzHDInsightCluster `
    -ClusterType Hadoop `
    -ClusterSizeInNodes 4 `
    -ResourceGroupName $clusterResourceGroupName `
    -ClusterName $clusterName `
    -HttpCredential $clusterCreds `
    -Location $location `
    -DefaultStorageAccountName "$storageAccountName.blob.core.contoso.net" `
    -DefaultStorageAccountKey $storageAccountKey `
    -DefaultStorageContainer $storageContainer `
    -SshCredential $clusterCreds `
    -EncryptionAtHost $true `
```

### <a name="enable-using-azure-cli"></a>Habilitación del uso de la CLI de Azure

En el fragmento de código siguiente se muestra cómo puede crear un nuevo clúster de Azure HDInsight con cifrado en host habilitado mediante la CLI de Azure. Se usa el parámetro `--encryption-at-host true` para habilitar la característica.

```azurecli
az hdinsight create -t spark -g MyResourceGroup -n MyCluster \\
-p "yourpass" \\
--storage-account MyStorageAccount --encryption-at-host true
```

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre Azure Key Vault, consulte [¿Qué es Azure Key Vault?](../key-vault/general/overview.md).
* [Introducción a la seguridad de la empresa en Azure HDInsight](./domain-joined/hdinsight-security-overview.md).
