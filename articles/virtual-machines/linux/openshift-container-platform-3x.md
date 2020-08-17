---
title: Implementación de OpenShift Container Platform 3.11 en Azure
description: Implementación de OpenShift Container Platform 3.11 en Azure.
author: haroldwongms
manager: mdotson
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: how-to
ms.workload: infrastructure
ms.date: 04/05/2020
ms.author: haroldw
ms.openlocfilehash: 0c60fdfda0c18f5a8feb11c3d9c5a386025670cd
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87368156"
---
# <a name="deploy-openshift-container-platform-311-in-azure"></a>Implementación de OpenShift Container Platform 3.11 en Azure

Puede usar uno de los diversos métodos para implementar OpenShift Container Platform 3.11 en Azure:

- Puede implementar manualmente los componentes necesarios de la infraestructura de Azure y, a continuación, seguir la [documentación de OpenShift Container Platform](https://docs.openshift.com/container-platform).
- También puede usar una plantilla existente de [Resource Manager](https://github.com/Microsoft/openshift-container-platform/) que simplifica la implementación del clúster de OpenShift Container Platform.
- Otra opción consiste en usar la [oferta de Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/osatesting.open-shift-azure-proxy).

En todos los casos, se requiere una suscripción a Red Hat. Durante la implementación, la instancia de Red Hat Enterprise Linux está registrada en la suscripción de Red Hat y asociada al identificador de grupo que contiene los derechos para OpenShift Container Platform.
Asegúrese de que tiene un nombre de usuario, una contraseña y un identificador de grupo válidos para Red Hat Subscription Manager (RHSM). Puede usar una clave de activación, identificador de organización e identificador de grupo. Puede comprobar esta información iniciando sesión en https://access.redhat.com.


## <a name="deploy-using-the-openshift-container-platform-resource-manager-311-template"></a>Implementación de la plantilla de Resource Manager para OpenShift Container Platform 3.11

### <a name="private-clusters"></a>Clústeres privados

La implementación de clústeres de OpenShift privados requiere más aspectos que no tener una dirección IP pública asociada al equilibrador de carga maestro (consola web) o al del equilibrador de carga de infraestructura (enrutador).  Normalmente un clúster privado usa un servidor DNS personalizado (no el predeterminado de Azure DNS), un nombre de dominio personalizado (como contoso.com) y redes virtuales predefinidas.  Para los clústeres privados, deberá configurar la red virtual con todas las subredes adecuadas y la configuración del servidor DNS de antemano.  A continuación, use **existingMasterSubnetReference**, **existingInfraSubnetReference**, **existingCnsSubnetReference** y **existingNodeSubnetReference** para especificar la subred existente para que la use el clúster.

Si se selecciona la opción de clúster maestro privado (**masterClusterType**=private), debe especificarse una dirección IP privada estática para **masterPrivateClusterIp**.  Esta dirección IP se asignará para el front-end del equilibrador de carga maestro.  La dirección IP debe estar dentro del CIDR para la subred maestra y no estar en uso.  **masterClusterDnsType** debe establecerse en "custom" y el nombre DNS maestro debe proporcionarse para **masterClusterDns**.  El nombre DNS debe asignarse a la dirección IP privada estática y se usará para obtener acceso a la consola en los nodos maestros.

Si se selecciona la opción de enrutador privado (**routerClusterType**=private), debe especificarse una dirección IP privada estática para **routerPrivateClusterIp**.  Esta dirección IP se asignará para el front-end del equilibrador de carga de infraestructura.  La dirección IP debe estar dentro del CIDR para la subred de infraestructura y no estar en uso.  **routingSubDomainType** debe establecerse en "custom" y el nombre DNS con caracteres comodín para el enrutamiento se debe proporcionar para **routingSubDomain**.  

Si se seleccionan maestros privados y el enrutador privado, también debe especificarse el nombre de dominio personalizado para **domainName**.

Tras una implementación correcta, el nodo de Bastion es el único nodo con una dirección IP pública en el que puede tener acceso mediante SSH.  Incluso si los nodos maestros están configurados para acceso público, no se exponen para el acceso de SSH.

Para realizar la implementación mediante la plantilla de Resource Manager, se usa un archivo de parámetros para proporcionar los parámetros de entrada. Para personalizar aún más la implementación, puede bifurcar el repositorio de GitHub y cambiar los elementos correspondientes.

Algunas opciones de personalización comunes son, entre otras:

- Tamaño de máquina virtual de bastión (variable en azuredeploy.json)
- Convenciones de nomenclatura (variables en azuredeploy.json)
- Especificaciones del clúster de OpenShift, modificadas mediante el archivo de hosts (deployOpenShift.sh)

### <a name="configure-the-parameters-file"></a>Configuración del archivo de parámetros

La [plantilla de OpenShift Container Platform](https://github.com/Microsoft/openshift-container-platform) tiene varias ramas disponibles para las diferentes versiones de OpenShift Container Platform.  Según sus necesidades, puede implementar directamente desde el repositorio o puede bifurcar el repositorio y realizar cambios personalizados en las plantillas o los scripts antes de implementar.

Use el valor `appId` de la entidad de servicio que creó anteriormente para el parámetro `aadClientId`.

En el ejemplo siguiente se muestra un archivo de parámetros llamado azuredeploy.parameters.json con todas las entradas necesarias.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "_artifactsLocation": {
            "value": "https://raw.githubusercontent.com/Microsoft/openshift-container-platform/master"
        },
        "location": {
            "value": "eastus"
        },
        "masterVmSize": {
            "value": "Standard_E2s_v3"
        },
        "infraVmSize": {
            "value": "Standard_D4s_v3"
        },
        "nodeVmSize": {
            "value": "Standard_D4s_v3"
        },
        "cnsVmSize": {
            "value": "Standard_E4s_v3"
        },
        "osImageType": {
            "value": "defaultgallery"
        },
        "marketplaceOsImage": {
            "value": {
                "publisher": "RedHat",
                "offer": "RHEL",
                "sku": "7-RAW",
                "version": "latest"
            }
        },
        "storageKind": {
            "value": "changeme"
        },
        "openshiftClusterPrefix": {
            "value": "changeme"
        },
        "minorVersion": {
            "value": "69"
        },
        "masterInstanceCount": {
            "value": 3
        },
        "infraInstanceCount": {
            "value": 3
        },
        "nodeInstanceCount": {
            "value": 3
        },
        "cnsInstanceCount": {
            "value": 3
        },
        "osDiskSize": {
            "value": 64
        },
        "dataDiskSize": {
            "value": 64
        },
        "cnsGlusterDiskSize": {
            "value": 128
        },
        "adminUsername": {
            "value": "changeme"
        },
        "enableMetrics": {
            "value": "false"
        },
        "enableLogging": {
            "value": "false"
        },
        "enableCNS": {
            "value": "false"
        },
        "rhsmUsernameOrOrgId": {
            "value": "changeme"
        },
        "rhsmPoolId": {
            "value": "changeme"
        },
        "rhsmBrokerPoolId": {
            "value": "changeme"
        },
        "sshPublicKey": {
            "value": "GEN-SSH-PUB-KEY"
        },
        "keyVaultSubscriptionId": {
            "value": "255a325e-8276-4ada-af8f-33af5658eb34"
        },
        "keyVaultResourceGroup": {
            "value": "changeme"
        },
        "keyVaultName": {
            "value": "changeme"
        },
        "enableAzure": {
            "value": "true"
        },
        "aadClientId": {
            "value": "changeme"
        },
        "domainName": {
            "value": "contoso.com"
        },
        "masterClusterDnsType": {
            "value": "default"
        },
        "masterClusterDns": {
            "value": "console.contoso.com"
        },
        "routingSubDomainType": {
            "value": "nipio"
        },
        "routingSubDomain": {
            "value": "apps.contoso.com"
        },
        "virtualNetworkNewOrExisting": {
            "value": "new"
        },
        "virtualNetworkName": {
            "value": "changeme"
        },
        "addressPrefixes": {
            "value": "10.0.0.0/14"
        },
        "masterSubnetName": {
            "value": "changeme"
        },
        "masterSubnetPrefix": {
            "value": "10.1.0.0/16"
        },
        "infraSubnetName": {
            "value": "changeme"
        },
        "infraSubnetPrefix": {
            "value": "10.2.0.0/16"
        },
        "nodeSubnetName": {
            "value": "changeme"
        },
        "nodeSubnetPrefix": {
            "value": "10.3.0.0/16"
        },
        "existingMasterSubnetReference": {
            "value": "/subscriptions/abc686f6-963b-4e64-bff4-99dc369ab1cd/resourceGroups/vnetresourcegroup/providers/Microsoft.Network/virtualNetworks/openshiftvnet/subnets/mastersubnet"
        },
        "existingInfraSubnetReference": {
            "value": "/subscriptions/abc686f6-963b-4e64-bff4-99dc369ab1cd/resourceGroups/vnetresourcegroup/providers/Microsoft.Network/virtualNetworks/openshiftvnet/subnets/infrasubnet"
        },
        "existingCnsSubnetReference": {
            "value": "/subscriptions/abc686f6-963b-4e64-bff4-99dc369ab1cd/resourceGroups/vnetresourcegroup/providers/Microsoft.Network/virtualNetworks/openshiftvnet/subnets/cnssubnet"
        },
        "existingNodeSubnetReference": {
            "value": "/subscriptions/abc686f6-963b-4e64-bff4-99dc369ab1cd/resourceGroups/vnetresourcegroup/providers/Microsoft.Network/virtualNetworks/openshiftvnet/subnets/nodesubnet"
        },
        "masterClusterType": {
            "value": "public"
        },
        "masterPrivateClusterIp": {
            "value": "10.1.0.200"
        },
        "routerClusterType": {
            "value": "public"
        },
        "routerPrivateClusterIp": {
            "value": "10.2.0.200"
        },
        "routingCertType": {
            "value": "selfsigned"
        },
        "masterCertType": {
            "value": "selfsigned"
        }
    }
}
```

Reemplace los parámetros por su información específica.

Diferentes versiones pueden tener distintos parámetros; por ello, compruebe los parámetros necesarios para la rama que utilice.

### <a name="azuredeployparametersjson-file-explained"></a>Explicación del archivo azuredeploy.Parameters.json

| Propiedad | Descripción | Opciones válidas | Valor predeterminado |
|----------|-------------|---------------|---------------|
| `_artifactsLocation`  | Dirección URL para artefactos (json, scripts, etc.). |  |  https:\//raw.githubusercontent.com/Microsoft/openshift-container-platform/master  |
| `location` | Región de Azure en la que se implementan los recursos. |  |  |
| `masterVmSize` | Tamaño de la VM maestra. Seleccione uno de los tamaños de VM permitidos que aparece en el archivo azuredeploy.json. |  | Standard_E2s_v3 |
| `infraVmSize` | Tamaño de la VM de infraestructura. Seleccione uno de los tamaños de VM permitidos que aparece en el archivo azuredeploy.json. |  | Standard_D4s_v3 |
| `nodeVmSize` | Tamaño de la VM del nodo de la aplicación. Seleccione uno de los tamaños de VM permitidos que aparece en el archivo azuredeploy.json. |  | Standard_D4s_v3 |
| `cnsVmSize` | Tamaño de la VM del nodo de Container Native Storage (CNS). Seleccione uno de los tamaños de VM permitidos que aparece en el archivo azuredeploy.json. |  | Standard_E4s_v3 |
| `osImageType` | Imagen de RHEL que se va a usar. defaultgallery: a petición; Marketplace: imagen de terceros. | defaultgallery <br> marketplace | defaultgallery |
| `marketplaceOsImage` | Si `osImageType` es Marketplace, a continuación, especifique los valores adecuados de "publisher", "offer", "sku" y "version" de la oferta de Marketplace. Este parámetro es un tipo de objeto. |  |  |
| `storageKind` | Tipo de almacenamiento que se va a usar.  | administrado<br> unmanaged | administrado |
| `openshiftClusterPrefix` | Prefijo del clúster usado para configurar los nombres de host para todos los nodos.  Entre 1 y 20 caracteres. |  | mycluster |
| `minoVersion` | Versión secundaria de OpenShift Container Platform 3.11 que se va a implementar. |  | 69 |
| `masterInstanceCount` | Número de nodos maestros que se van a implementar. | 1, 3, 5 | 3 |
| `infraInstanceCount` | Número de nodos de infraestructura que se van a implementar. | 1, 2, 3 | 3 |
| `nodeInstanceCount` | Número de nodos que se van a implementar. | 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30 | 2 |
| `cnsInstanceCount` | Número de nodos de CNS que se van a implementar. | 3, 4 | 3 |
| `osDiskSize` | Tamaño del disco de SO de la VM (en GB). | 64, 128, 256, 512, 1024, 2048 | 64 |
| `dataDiskSize` | Tamaño del disco de datos que se va a adjuntar a los nodos para el volumen de Docker (en GB). | 32, 64, 128, 256, 512, 1024, 2048 | 64 |
| `cnsGlusterDiskSize` | Tamaño del disco de datos que se va a adjuntar a los nodos de CNS por glusterfs (en GB). | 32, 64, 128, 256, 512, 1024, 2048 | 128 |
| `adminUsername` | Nombre de usuario administrador para el inicio de sesión de SO (VM) y de usuario inicial de OpenShift. |  | ocpadmin |
| `enableMetrics` | Habilita las métricas. Las métricas requieren más recursos, por lo que debe seleccionar un tamaño adecuado para la VM de infraestructura. | true <br> false | false |
| `enableLogging` | Habilita el registro. El pod elasticsearch requiere 8 GB de RAM, por lo que debe seleccionar un tamaño adecuado para la VM de infraestructura. | true <br> false | false |
| `enableCNS` | Habilita Container Native Storage. | true <br> false | false |
| `rhsmUsernameOrOrgId` | Id. de organización o nombre de usuario de administrador de suscripciones de Red Hat. |  |  |
| `rhsmPoolId` | Identificador de grupo del administrador de suscripciones de Red Hat que contiene los derechos de OpenShift para nodos de ejecución. |  |  |
| `rhsmBrokerPoolId` | Identificador de grupo del administrador de suscripciones de Red Hat que contiene los derechos de OpenShift para nodos maestros y de infraestructura. Si no tiene diferentes id. de grupo, escriba el mismo id. de grupo que en el caso de "rhsmPoolId". |  |
| `sshPublicKey` | Copie la clave pública SSH aquí. |  |  |
| `keyVaultSubscriptionId` | Identificador de la suscripción que contiene la instancia de Key Vault. |  |  |
| `keyVaultResourceGroup` | Nombre del grupo de recursos que contiene la instancia de Key Vault. |  |  |
| `keyVaultName` | Nombre de la instancia de Key Vault que ha creado. |  |  |
| `enableAzure` | Habilita el proveedor de soluciones en la nube de Azure. | true <br> false | true |
| `aadClientId` | El id. de cliente de Azure Active Directory también se conoce como id. de la aplicación para la entidad de servicio. |  |  |
| `domainName` | Nombre de dominio personalizado que se va a usar (si corresponde). Establecer en "none" si no se implementa un clúster totalmente privado. |  | None |
| `masterClusterDnsType` | Tipo de dominio de la consola web de OpenShift. "default" usará la etiqueta DNS de la dirección IP pública de infraestructura maestra. "custom" permite definir su propio nombre. | default <br> custom | default |
| `masterClusterDns` | Nombre DNS personalizado para tener acceso a la consola web de OpenShift si ha seleccionado "custom" para `masterClusterDnsType`. |  | console.contoso.com |
| `routingSubDomainType` | Si se establece en "nipio", `routingSubDomain` usará nip.io.  Use "custom" si tiene su propio dominio y quiere utilizarlo para el enrutamiento. | nipio <br> custom | nipio |
| `routingSubDomain` | Nombre DNS con carácter comodín que quiere usar para el enrutamiento si ha seleccionado "custom" para `routingSubDomainType`. |  | apps.contoso.com |
| `virtualNetworkNewOrExisting` | Seleccione si quiere usar una red virtual existente o crear una nueva. | existing <br> nuevo | nuevo |
| `virtualNetworkResourceGroupName` | Nombre del grupo de recursos para la nueva red virtual si ha seleccionado "new" para `virtualNetworkNewOrExisting`. |  | resourceGroup().name |
| `virtualNetworkName` | Nombre de la nueva red virtual que se va a crear si ha seleccionado "new" para `virtualNetworkNewOrExisting`. |  | openshiftvnet |
| `addressPrefixes` | Prefijo de dirección de la nueva red virtual. |  | 10.0.0.0/14 |
| `masterSubnetName` | Nombre de la subred maestra. |  | mastersubnet |
| `masterSubnetPrefix` | CIDR usado para la subred maestra: debe ser un subconjunto de los valores de addressPrefix. |  | 10.1.0.0/16 |
| `infraSubnetName` | Nombre de la subred de infraestructura. |  | infrasubnet |
| `infraSubnetPrefix` | CIDR usado para la subred de infraestructura: debe ser un subconjunto de los valores de addressPrefix. |  | 10.2.0.0/16 |
| `nodeSubnetName` | Nombre de la subred del nodo. |  | nodesubnet |
| `nodeSubnetPrefix` | CIDR usado para la subred del nodo: debe ser un subconjunto de los valores de addressPrefix. |  | 10.3.0.0/16 |
| `existingMasterSubnetReference` | Referencia completa de la subred existente para los nodos maestros. No se necesita si crea una nueva red virtual o subred. |  |  |
| `existingInfraSubnetReference` | Referencia completa de la subred existente para los nodos de infraestructura. No se necesita si crea una nueva red virtual o subred. |  |  |
| `existingCnsSubnetReference` | Referencia completa de la subred existente para los nodos de CNS. No se necesita si crea una nueva red virtual o subred. |  |  |
| `existingNodeSubnetReference` | Referencia completa de la subred existente para los nodos de ejecución. No se necesita si crea una nueva red virtual o subred. |  |  |
| `masterClusterType` | Especifique si el clúster usa nodos maestros públicos o privados. Si se eligen nodos privados, los nodos maestros no se expondrán en Internet a través de una dirección IP pública. En su lugar, usará la dirección IP privada que especificó en `masterPrivateClusterIp`. | público <br> privado | público |
| `masterPrivateClusterIp` | Si se seleccionan los nodos maestros privados, debe especificarse una dirección IP privada para que la use el equilibrador de carga interno para los nodos maestros. La dirección IP estática debe estar dentro del bloque de CIDR para la subred maestra y no estar en uso todavía. Si se seleccionan los nodos maestros públicos, este valor no se usará, aunque se debe especificar. |  | 10.1.0.200 |
| `routerClusterType` | Especifique si el clúster usa nodos de infraestructura públicos o privados. Si se eligen nodos privados, los nodos de infraestructura no se expondrán en Internet a través de una dirección IP pública. En su lugar, usará la dirección IP privada que especificó en `routerPrivateClusterIp`. | público <br> privado | público |
| `routerPrivateClusterIp` | Si se seleccionan los nodos de infraestructura privados, debe especificarse una dirección IP privada para que la use el equilibrador de carga interno para los nodos de infraestructura. La dirección IP estática debe estar dentro del bloque de CIDR para la subred de infraestructura y no estar en uso todavía. Si se seleccionan los nodos de infraestructura públicos, este valor no se usará, aunque se debe especificar. |  | 10.2.0.200 |
| `routingCertType` | Use un certificado personalizado para un dominio de enrutamiento o el certificado autofirmado predeterminado; siga las instrucciones de la sección **Certificados personalizados**. | selfsigned <br> custom | selfsigned |
| `masterCertType` | Use un certificado personalizado para un dominio maestro o el certificado autofirmado predeterminado; siga las instrucciones de la sección **Certificados personalizados**. | selfsigned <br> custom | selfsigned |

<br>

### <a name="deploy-using-azure-cli"></a>Implementación con la CLI de Azure

> [!NOTE] 
> El comando siguiente requiere la CLI de Azure 2.0.8 o una versión posterior. Puede comprobar la versión de la CLI con el comando `az --version`. Para actualizar la versión de la CLI, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latesti).

En el ejemplo siguiente se implementa el clúster de OpenShift y todos los recursos relacionados en un grupo de recursos llamado openshiftrg, con el nombre de implementación myOpenShiftCluster. Se hace referencia a la plantilla directamente desde el repositorio de GitHub y se usa un archivo de parámetros local denominado azuredeploy.parameters.json.

```azurecli 
az group deployment create -g openshiftrg --name myOpenShiftCluster \
      --template-uri https://raw.githubusercontent.com/Microsoft/openshift-container-platform/master/azuredeploy.json \
      --parameters @./azuredeploy.parameters.json
```

La implementación tarda al menos 60 minutos en completarse, según el número total de nodos implementados y las opciones configuradas. El nombre de dominio completo DNS del host de tipo bastión y la dirección URL de la consola de OpenShift se imprimen en el terminal cuando concluye la implementación.

```json
{
  "Bastion DNS FQDN": "bastiondns4hawllzaavu6g.eastus.cloudapp.azure.com",
  "OpenShift Console URL": "http://openshiftlb.eastus.cloudapp.azure.com/console"
}
```

Si no desea que la línea de comandos espere hasta que finalice la implementación, agregue `--no-wait` como una de las opciones para la implementación del grupo. La salida de la implementación se puede recuperar en Azure Portal, en la sección de implementación del grupo de recursos.

## <a name="connect-to-the-openshift-cluster"></a>Conexión con el clúster de OpenShift

Cuando finalice la implementación, puede recuperar la conexión de la sección de salida de la implementación. Conéctese a la consola de OpenShift con el explorador mediante la **dirección URL de la consola de OpenShift**. También puede acceder mediante SSH al host de Bastion. El siguiente es un ejemplo en el que el nombre de usuario administrador es clusteradmin y el nombre de dominio completo de DNS de la dirección IP pública del host de tipo bastión es bastiondns4hawllzaavu6g.eastus.cloudapp.azure.com:

```bash
$ ssh clusteradmin@bastiondns4hawllzaavu6g.eastus.cloudapp.azure.com
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede usar el comando [az group delete](/cli/azure/group) para quitar el grupo de recursos, el clúster de OpenShift y todos los recursos relacionados.

```azurecli 
az group delete --name openshiftrg
```

## <a name="next-steps"></a>Pasos siguientes

- [Tareas posteriores a la implementación](./openshift-container-platform-3x-post-deployment.md)
- [Solución de problemas de implementación de OpenShift en Azure](./openshift-container-platform-3x-troubleshooting.md)
- [Introducción a OpenShift Container Platform](https://docs.openshift.com)
