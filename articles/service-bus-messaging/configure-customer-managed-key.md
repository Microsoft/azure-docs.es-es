---
title: Configuración de su propia clave para cifrar datos en reposo de Azure Service Bus
description: En este artículo se proporciona información sobre cómo configurar su propia clave para cifrar datos en reposo de Azure Service Bus.
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: e3da167fcdd3bac53de86dae07242cf8bccb621c
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400592"
---
# <a name="configure-customer-managed-keys-for-encrypting-azure-service-bus-data-at-rest-by-using-the-azure-portal"></a>Configuración de claves administradas por el cliente para cifrar datos en reposo de Azure Service Bus mediante Azure Portal
Azure Service Bus Premium proporciona cifrado de datos en reposo con Azure Storage Service Encryption (Azure SSE). Service Bus Premium se basa en Azure Storage para almacenar los datos y, de forma predeterminada, todos los datos que se almacenan con Azure Storage se cifran mediante claves administradas por Microsoft. 

## <a name="overview"></a>Información general
Azure Service Bus ahora admite la opción de cifrado de datos en reposo con claves administradas por Microsoft o claves administradas por el cliente (Bring Your Own Key – BYOK). Esta característica permite crear, rotar, deshabilitar y revocar el acceso a las claves administradas por el cliente que se usan para cifrar datos en reposo de Azure Service Bus.

La habilitación de la característica BYOK es un proceso que solo hay que configurar una vez en el espacio de nombres.

> [!NOTE]
> Hay algunas advertencias para la clave administrada por el cliente para el cifrado en el lado del servicio. 
>   * Esta característica se admite en el nivel [Premium de Azure Service Bus](service-bus-premium-messaging.md). No se puede habilitar para los espacios de nombres estándar de Service Bus.
>   * El cifrado solo se puede habilitar para espacios de nombres nuevos o vacíos. Si el espacio de nombres contiene colas o temas, se producirá un error en la operación de cifrado.

Puede usar Azure Key Vault para administrar las claves y auditar su uso. Puede crear sus propias claves y almacenarlas en un almacén de claves, o puede usar las API de Azure Key Vault para generarlas. Para obtener más información sobre Azure Key Vault, consulte [¿Qué es Azure Key Vault?](../key-vault/general/overview.md)

En este artículo se muestra cómo configurar un almacén de claves con claves administradas del cliente mediante Azure Portal. Para obtener información sobre cómo crear un almacén de claves mediante Azure Portal, consulte [Inicio rápido: Establecimiento y recuperación de un secreto de Azure Key Vault mediante Azure Portal](../key-vault/secrets/quick-create-portal.md).

> [!IMPORTANT]
> El uso de claves administradas por el cliente con Azure Service Bus requiere que el almacén de claves tenga configuradas dos propiedades obligatorias. Son las siguientes:  **Eliminación temporal** y **No purgar**. Estas propiedades están habilitadas de manera predeterminada cuando crea un nuevo almacén de claves en Azure Portal. Sin embargo, si tiene que habilitar estas propiedades en un almacén de claves existente, deberá usar PowerShell o la CLI de Azure.

## <a name="enable-customer-managed-keys"></a>Habilitar claves administradas del cliente
Para habilitar claves administradas del cliente en Azure Portal, siga estos pasos:

1. Vaya al espacio de nombres Premium de Service Bus.
2. En la página **Configuración** del espacio de nombres de Service Bus, seleccione **Cifrado**.
3. Seleccione **Cifrado de claves en reposo que administra el cliente** como se muestra en la siguiente imagen.

    ![Habilitación de clave administrada por el cliente](./media/configure-customer-managed-key/enable-customer-managed-key.png)


## <a name="set-up-a-key-vault-with-keys"></a>Configuración de un almacén de claves con claves

Después de habilitar las claves administradas por el cliente, debe asociar la clave administrada por el cliente con su espacio de nombres de Azure Service Bus. Service Bus solo admite Azure Key Vault. Si habilita la opción **Encryption with customer-managed key** (Cifrado con clave administrada por el cliente) en la sección anterior, debe importar la clave en Azure Key Vault. Recuerde que las claves deben tener habilitadas las opciones **Eliminación temporal** y **No purgar**. Estas opciones se pueden configurar mediante [PowerShell](../key-vault/general/soft-delete-powershell.md) o la [CLI](../key-vault/general/soft-delete-cli.md#enabling-purge-protection).

1. Para crear un nuevo almacén de claves, siga el [inicio rápido](../key-vault/general/overview.md) de Azure Key Vault. Para más información sobre cómo importar claves existentes, consulte [Información acerca de claves, secretos y certificados](../key-vault/general/about-keys-secrets-certificates.md).
1. Para activar la eliminación temporal y la protección de purgas al crear un almacén, use el comando [az keyvault create](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-create).

    ```azurecli-interactive
    az keyvault create --name contoso-SB-BYOK-keyvault --resource-group ContosoRG --location westus --enable-soft-delete true --enable-purge-protection true
    ```    
1. Para agregar la protección de purgas a un almacén existente (que ya tenga habilitada la eliminación temporal), use el comando [az keyvault update](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-update).

    ```azurecli-interactive
    az keyvault update --name contoso-SB-BYOK-keyvault --resource-group ContosoRG --enable-purge-protection true
    ```
1. Siga estos pasos para crear claves:
    1. Para crear una nueva clave, seleccione **Generar/Importar** en el menú **Claves** en **Configuración**.
        
        ![Seleccione el botón Generate/Import (Generar/importar).](./media/configure-customer-managed-key/select-generate-import.png)

    1. Establezca **Opciones** en **Generar** y asigne un nombre a la clave.

        ![Crear una clave](./media/configure-customer-managed-key/create-key.png) 

    1. Ahora puede seleccionar esta clave para asociarla con el espacio de nombres de Service Bus para el cifrado en la lista desplegable. 

        ![Selección de clave del almacén de claves](./media/configure-customer-managed-key/select-key-from-key-vault.png)
        > [!NOTE]
        > Para lograr redundancia, puede agregar hasta 3 claves. En caso de que una de las claves haya expirado o no sea accesible, se usarán las demás claves para el cifrado.
        
    1. Rellene los detalles de la clave y haga clic en **Seleccionar**. Esto habilitará el cifrado de datos en reposo en el espacio de nombres con una clave administrada por el cliente. 


    > [!IMPORTANT]
    > Si desea usar la clave administrada por el cliente junto con la recuperación ante desastres geográfica, revise la siguiente información: 
    >
    > Para habilitar el cifrado en reposo con la clave administrada por el cliente, se configura una [directiva de acceso](../key-vault/general/secure-your-key-vault.md) para la identidad administrada de Service Bus en el almacén de claves de Azure especificado. Esto garantiza el acceso controlado al almacén de claves de Azure desde el espacio de nombres de Azure Service Bus.
    >
    > Debido a esto:
    > 
    >   * Si la [recuperación ante desastres geográfica](service-bus-geo-dr.md) ya está habilitada para el espacio de nombres de Service Bus y desea habilitar la clave administrada por el cliente, siga estos pasos: 
    >     * Interrumpa el emparejamiento
    >     * [Configure la directiva de acceso](../key-vault/general/assign-access-policy-portal.md) para la identidad administrada de los espacios de nombres principal y secundario en el almacén de claves.
    >     * Configure el cifrado en el espacio de nombres principal.
    >     * Vuelva a emparejar los espacios de nombres principal y secundario.
    > 
    >   * Si desea habilitar la recuperación ante desastres geográfica en un espacio de nombres de Service Bus en el que la clave administrada por el cliente ya está configurada, haga lo siguiente:
    >     * [Configure la directiva de acceso](../key-vault/general/assign-access-policy-portal.md) para la identidad administrada del espacio de nombres secundario en el almacén de claves.
    >     * Empareje los espacios de nombres principal y secundario.


## <a name="rotate-your-encryption-keys"></a>Rotación de las claves de cifrado

Puede rotar la clave en el almacén de claves mediante el mecanismo de rotación de los Azure Key Vault. También es posible establecer fechas de activación y expiración para automatizar la rotación de claves. El servicio Service Bus detectará nuevas versiones de clave y comenzará a usarlas automáticamente.

## <a name="revoke-access-to-keys"></a>Revocación del acceso a las claves

Al revocar el acceso a las claves de cifrado, no se purgan los datos de Service Bus. Sin embargo, no se podrá acceder a los datos desde el espacio de nombres de Service Bus. Puede revocar la clave de cifrado mediante la directiva de acceso o eliminando la clave. Obtenga más información sobre las directivas de acceso y la protección del almacén de claves en [Protección del acceso a un almacén de claves](../key-vault/general/secure-your-key-vault.md).

Una vez revocada la clave de cifrado, el servicio Service Bus en el espacio de nombres cifrado dejará de ser operativo. Si el acceso a la clave está habilitado o si se ha restaurado la clave eliminada, el servicio Service Bus seleccionará la clave para que pueda acceder a los datos desde el espacio de nombres de Service Bus cifrado.

## <a name="use-resource-manager-template-to-enable-encryption"></a>Uso de la plantilla de Resource Manager para habilitar el cifrado
En esta sección se muestra cómo realizar las siguientes tareas con **plantillas de Azure Resource Manager**. 

1. Cree un espacio de nombres de Service Bus **premium** con una **identidad de servicio administrada**.
2. Cree un **almacén de claves** y conceda a la identidad de servicio acceso a él. 
3. Actualice el espacio de nombres de Service Bus con la información del almacén de claves (clave/valor). 


### <a name="create-a-premium-service-bus-namespace-with-managed-service-identity"></a>Creación de un espacio de nombres de Service Bus con una identidad de servicio administrada
En esta sección se muestra cómo crear un espacio de nombres de Azure Service Bus con la identidad de servicio administrada mediante una plantilla de Azure Resource Manager y PowerShell. 

1. Cree una plantilla de Azure Resource Manager para crear un espacio de nombres de nivel premium de Service Bus con una identidad de servicio administrada. Asigne el nombre al archivo: **CreateServiceBusPremiumNamespace.json**: 

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "namespaceName":{
             "type":"string",
             "metadata":{
                "description":"Name for the Namespace."
             }
          },
          "location":{
             "type":"string",
             "defaultValue":"[resourceGroup().location]",
             "metadata":{
                "description":"Specifies the Azure location for all resources."
             }
          }
       },
       "resources":[
          {
             "type":"Microsoft.ServiceBus/namespaces",
             "apiVersion":"2018-01-01-preview",
             "name":"[parameters('namespaceName')]",
             "location":"[parameters('location')]",
             "identity":{
                "type":"SystemAssigned"
             },
             "sku":{
                "name":"Premium",
                "tier":"Premium",
                "capacity":1
             },
             "properties":{
    
             }
          }
       ],
       "outputs":{
          "ServiceBusNamespaceId":{
             "type":"string",
             "value":"[resourceId('Microsoft.ServiceBus/namespaces',parameters('namespaceName'))]"
          }
       }
    }
    ```
2. Cree un archivo de parámetros de plantilla llamado: **CreateServiceBusPremiumNamespaceParams.json**. 

    > [!NOTE]
    > Reemplace los siguientes valores: 
    > - `<ServiceBusNamespaceName>`: nombre del espacio de nombres de Service Bus.
    > - `<Location>`: ubicación del espacio de nombres de Service Bus.

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "namespaceName":{
             "value":"<ServiceBusNamespaceName>"
          },
          "location":{
             "value":"<Location>"
          }
       }
    }
    ```
3. Ejecute el siguiente comando de PowerShell para implementar la plantilla y crear un espacio de nombres de Service Bus. A continuación, recupere el identificador del espacio de nombres de Service Bus para usarlo más adelante. Antes de ejecutar el comando, reemplace `{MyRG}` por el nombre del grupo de recursos.  

    ```powershell
    $outputs = New-AzResourceGroupDeployment -Name CreateServiceBusPremiumNamespace -ResourceGroupName {MyRG} -TemplateFile ./CreateServiceBusPremiumNamespace.json -TemplateParameterFile ./CreateServiceBusPremiumNamespaceParams.json
    
    $ServiceBusNamespaceId = $outputs.Outputs["serviceBusNamespaceId"].value
    ```
 
### <a name="grant-service-bus-namespace-identity-access-to-key-vault"></a>Concesión de acceso al almacén de claves a la identidad del espacio de nombres de Service Bus

1. Ejecute el siguiente comando para crear un almacén de claves con las características **protección de purga** y **eliminación temporal** habilitadas. 

    ```powershell
    New-AzureRmKeyVault -Name "{keyVaultName}" -ResourceGroupName {RGName}  -Location "{location}" -EnableSoftDelete -EnablePurgeProtection    
    ```
    
    O BIEN
    
    Ejecute el siguiente comando para actualizar un **almacén de claves existente**. Antes de ejecutar el comando, especifique valores para los nombres de grupo de recursos y almacén de claves. 
    
    ```powershell
    ($updatedKeyVault = Get-AzureRmResource -ResourceId (Get-AzureRmKeyVault -ResourceGroupName {RGName} -VaultName {keyVaultName}).ResourceId).Properties| Add-Member -MemberType "NoteProperty" -Name "enableSoftDelete" -Value "true"-Force | Add-Member -MemberType "NoteProperty" -Name "enablePurgeProtection" -Value "true" -Force
    ``` 
2. Establezca la directiva de acceso del almacén de claves para que la identidad administrada del espacio de nombres de Service Bus pueda acceder al valor de la clave del almacén de claves. Use el identificador del espacio de nombres de Service Bus de la sección anterior. 

    ```powershell
    $identity = (Get-AzureRmResource -ResourceId $ServiceBusNamespaceId -ExpandProperties).Identity
    
    Set-AzureRmKeyVaultAccessPolicy -VaultName {keyVaultName} -ResourceGroupName {RGName} -ObjectId $identity.PrincipalId -PermissionsToKeys get,wrapKey,unwrapKey,list
    ```

### <a name="encrypt-data-in-service-bus-namespace-with-customer-managed-key-from-key-vault"></a>Cifrado de datos en el espacio de nombres de Service Bus con la clave administrada por el cliente desde el almacén de claves
Hasta el momento ha realizado los pasos siguientes: 

1. Ha creado un espacio de nombres premium con una identidad administrada.
2. Cree un almacén de claves y conceda a la identidad administrada acceso al almacén de claves. 

En este paso, actualizará el espacio de nombres de Service Bus con la información del almacén de claves. 

1. Cree un archivo JSON denominado **UpdateServiceBusNamespaceWithEncryption.json** con el siguiente contenido: 

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "namespaceName":{
             "type":"string",
             "metadata":{
                "description":"Name for the Namespace to be created in cluster."
             }
          },
          "location":{
             "type":"string",
             "defaultValue":"[resourceGroup().location]",
             "metadata":{
                "description":"Specifies the Azure location for all resources."
             }
          },
          "keyVaultUri":{
             "type":"string",
             "metadata":{
                "description":"URI of the KeyVault."
             }
          },
          "keyName":{
             "type":"string",
             "metadata":{
                "description":"KeyName."
             }
          }
       },
       "resources":[
          {
             "type":"Microsoft.ServiceBus/namespaces",
             "apiVersion":"2018-01-01-preview",
             "name":"[parameters('namespaceName')]",
             "location":"[parameters('location')]",
             "identity":{
                "type":"SystemAssigned"
             },
             "sku":{
                "name":"Premium",
                "tier":"Premium",
                "capacity":1
             },
             "properties":{
                "encryption":{
                   "keySource":"Microsoft.KeyVault",
                   "keyVaultProperties":[
                      {
                         "keyName":"[parameters('keyName')]",
                         "keyVaultUri":"[parameters('keyVaultUri')]"
                      }
                   ]
                }
             }
          }
       ]
    }
    ``` 

2. Cree un archivo de parámetros de plantilla: **UpdateServiceBusNamespaceWithEncryptionParams.json**.

    > [!NOTE]
    > Reemplace los siguientes valores: 
    > - `<ServiceBusNamespaceName>`: nombre del espacio de nombres de Service Bus.
    > - `<Location>`: ubicación del espacio de nombres de Service Bus.
    > - `<KeyVaultName>`: nombre del almacén de claves.
    > - `<KeyName>`: nombre de la clave del almacén de claves.  

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "namespaceName":{
             "value":"<ServiceBusNamespaceName>"
          },
          "location":{
             "value":"<Location>"
          },
          "keyName":{
             "value":"<KeyName>"
          },
          "keyVaultUri":{
             "value":"https://<KeyVaultName>.vault.azure.net"
          }
       }
    }
    ```             
3. Ejecute el siguiente comando de PowerShell para implementar la plantilla de Resource Manager. Antes de ejecutar el comando, reemplace `{MyRG}` por el nombre del grupo de recursos. 

    ```powershell
    New-AzResourceGroupDeployment -Name UpdateServiceBusNamespaceWithEncryption -ResourceGroupName {MyRG} -TemplateFile ./UpdateServiceBusNamespaceWithEncryption.json -TemplateParameterFile ./UpdateServiceBusNamespaceWithEncryptionParams.json
    ```
    

## <a name="next-steps"></a>Pasos siguientes
Vea los artículos siguientes:
- [Información general de Service Bus](service-bus-messaging-overview.md)
- [Introducción a Azure Key Vault](../key-vault/general/overview.md)
