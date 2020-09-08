---
title: 'Configuración de identidades administradas en Azure VM con REST: Azure AD'
description: Instrucciones paso a paso para configurar identidades administradas asignadas por el sistema y por el usuario en una VM de Azure, mediante CURL para llamar a la API REST.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 06/25/2018
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 59f60a0167e7ac09b1fdfee87cc8412cdbe19053
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89255977"
---
# <a name="configure-managed-identities-for-azure-resources-on-an-azure-vm-using-rest-api-calls"></a>Configuración de identidades administradas de recursos de Azure en una VM de Azure mediante llamadas a la API REST

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Las identidades administradas de los recursos de Azure proporcionan a los servicios de Azure una identidad del sistema administrada automáticamente en Azure Active Directory. Puede usar esta identidad para autenticar a cualquier servicio que admita la autenticación de Azure AD, sin necesidad de tener credenciales en el código. 

En este artículo, mediante CURL para llamar al punto de conexión REST de Azure Resource Manager, aprenderá a realizar las siguientes operaciones de identidades administradas de recursos de Azure en una VM de Azure:

- Habilitar y deshabilitar la identidad administrada asignada por el sistema en una VM de Azure
- Agregar y quitar una identidad administrada asignada por el usuario en una VM de Azure

## <a name="prerequisites"></a>Requisitos previos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario](overview.md#managed-identity-types)** .
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.
- Si usa Windows, instale el [subsistema de Windows para Linux](/windows/wsl/about) o [Azure Cloud Shell](../../cloud-shell/overview.md) en Azure Portal.
- [Instale la consola local de la CLI de Azure](/cli/azure/install-azure-cli) si utiliza el [subsistema de Windows para Linux](/windows/wsl/about) o un [sistema operativo de distribución de Linux](/cli/azure/install-azure-cli-apt?view=azure-cli-latest).
- Si utiliza la consola local de la CLI de Azure, inicie sesión en Azure mediante `az login` con una cuenta que esté asociada a la suscripción de Azure de la que desea administrar las identidades administradas asignadas por el usuario o por el sistema.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="system-assigned-managed-identity"></a>Identidad administrada asignada por el sistema

En esta sección, aprenderá a habilitar y deshabilitar una identidad administrada asignada por el sistema en una máquina virtual de Azure mediante CURL para llamar al punto de conexión de REST de Azure Resource Manager.

### <a name="enable-system-assigned-managed-identity-during-creation-of-an-azure-vm"></a>Habilitación de una identidad administrada asignada por el sistema durante la creación de una VM de Azure

Para crear una máquina virtual de Azure que tenga habilitada la identidad administrada asignada por el sistema, la cuenta debe tener la asignación de roles [Colaborador de la máquina Virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  No se requiere ninguna otra asignación de roles de directorio de Azure AD.

1. Cree un [grupo de recursos](../../azure-resource-manager/management/overview.md#terminology) para contener e implementar la máquina virtual y sus recursos relacionados, con [az group create](/cli/azure/group/#az-group-create). Puede omitir este paso si ya tiene un grupo de recursos que le gustaría usar en su lugar:

   ```azurecli-interactive 
   az group create --name myResourceGroup --location westus
   ```

2. Cree una [interfaz de red](/cli/azure/network/nic?view=azure-cli-latest#az-network-nic-create) para la máquina virtual:

   ```azurecli-interactive
    az network nic create -g myResourceGroup --vnet-name myVnet --subnet mySubnet -n myNic
   ```

3. Recupere un token de acceso de portador, que utilizará en el siguiente paso en el encabezado de autorización para crear la máquina virtual con una identidad administrada asignada por el sistema.

   ```azurecli-interactive
   az account get-access-token
   ``` 

4. Cree una máquina virtual con CURL para llamar al punto de conexión REST de Azure Resource Manager. En el ejemplo siguiente, se crea una máquina virtual denominada *myVM* con una identidad administrada asignada por el sistema, como ha identificado en el cuerpo de la solicitud el valor `"identity":{"type":"SystemAssigned"}`. Reemplace `<ACCESS TOKEN>` por el valor que ha recibido en el paso anterior cuando solicitó un token de acceso de portador y el valor `<SUBSCRIPTION ID>` según sea apropiado para su entorno.

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PUT -d '{"location":"westus","name":"myVM","identity":{"type":"SystemAssigned"},"properties":{"hardwareProfile":{"vmSize":"Standard_D2_v2"},"storageProfile":{"imageReference":{"sku":"2016-Datacenter","publisher":"MicrosoftWindowsServer","version":"latest","offer":"WindowsServer"},"osDisk":{"caching":"ReadWrite","managedDisk":{"storageAccountType":"Standard_LRS"},"name":"myVM3osdisk","createOption":"FromImage"},"dataDisks":[{"diskSizeGB":1023,"createOption":"Empty","lun":0},{"diskSizeGB":1023,"createOption":"Empty","lun":1}]},"osProfile":{"adminUsername":"azureuser","computerName":"myVM","adminPassword":"<SECURE PASSWORD STRING>"},"networkProfile":{"networkInterfaces":[{"id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic","properties":{"primary":true}}]}}}' -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PUT https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```
   
   **Encabezados de solicitud**
   
   |Encabezado de solicitud  |Descripción  |
   |---------|---------|
   |*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
   |*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        | 
   
   **Cuerpo de la solicitud**

   ```JSON
     {
       "location":"westus",
       "name":"myVM",
       "identity":{
          "type":"SystemAssigned"
       },
       "properties":{
          "hardwareProfile":{
             "vmSize":"Standard_D2_v2"
          },
          "storageProfile":{
             "imageReference":{
                "sku":"2016-Datacenter",
                "publisher":"MicrosoftWindowsServer",
                "version":"latest",
                "offer":"WindowsServer"
             },
             "osDisk":{
                "caching":"ReadWrite",
                "managedDisk":{
                   "storageAccountType":"Standard_LRS"
                },
                "name":"myVM3osdisk",
                "createOption":"FromImage"
             },
             "dataDisks":[
                {
                   "diskSizeGB":1023,
                   "createOption":"Empty",
                   "lun":0
                },
                {
                   "diskSizeGB":1023,
                   "createOption":"Empty",
                   "lun":1
                }
             ]
          },
          "osProfile":{
             "adminUsername":"azureuser",
             "computerName":"myVM",
             "adminPassword":"myPassword12"
          },
          "networkProfile":{
             "networkInterfaces":[
                {
                   "id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic",
                   "properties":{
                      "primary":true
                   }
                }
             ]
          }
       }
    }  
   ```

### <a name="enable-system-assigned-identity-on-an-existing-azure-vm"></a>Habilitación de la identidad asignada por el sistema en una máquina virtual de Azure existente

Para habilitar una identidad administrada asignada por el sistema en una máquina virtual que en un principio se aprovisionó sin dicha identidad, la cuenta necesita la asignación de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  No se requiere ninguna otra asignación de roles de directorio de Azure AD.

1. Recupere un token de acceso de portador, que utilizará en el siguiente paso en el encabezado de autorización para crear la máquina virtual con una identidad administrada asignada por el sistema.

   ```azurecli-interactive
   az account get-access-token
   ```

2. Use el comando CURL siguiente para llamar al punto de conexión REST de Azure Resource Manager para habilitar la identidad administrada asignada por el sistema en la máquina virtual, como se identifica en el cuerpo de la solicitud por el valor `{"identity":{"type":"SystemAssigned"}` para una máquina virtual denominada *myVM*.  Reemplace `<ACCESS TOKEN>` por el valor que ha recibido en el paso anterior cuando solicitó un token de acceso de portador y el valor `<SUBSCRIPTION ID>` según sea apropiado para su entorno.
   
   > [!IMPORTANT]
   > Para asegurarse de que no elimina ninguna de las identidades administradas asignadas por el usuario que están asignadas a la máquina virtual, necesita enumerar las identidades administradas asignadas por el usuario mediante este comando CURL: `curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>?api-version=2018-06-01' -H "Authorization: Bearer <ACCESS TOKEN>"`. Si tiene asignadas identidades administradas asignadas por el usuario a la máquina virtual como se identifica en el valor `identity` de la respuesta, vaya al paso 3 que muestra cómo conservar las identidades administradas asignadas por el usuario mientras se habilita la identidad administrada asignada por el sistema en la máquina virtual.

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned"}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```
   **Encabezados de solicitud**

   |Encabezado de solicitud  |Descripción  |
   |---------|---------|
   |*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
   |*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        | 
   
   **Cuerpo de la solicitud**
    
   ```JSON
    {  
       "identity":{  
          "type":"SystemAssigned"
       }
    }
   ```

3. Para habilitar la identidad administrada asignada por el sistema en una máquina virtual con identidades administradas existentes asignadas por el usuario, debe agregar `SystemAssigned` al valor `type`.  
   
   Por ejemplo, si la máquina virtual tiene asignadas las identidades administradas asignadas por el usuario `ID1` y `ID2`, y desea agregar una identidad administrada asignada por el sistema a dicha máquina virtual, utilice la siguiente llamada CURL. Reemplace `<ACCESS TOKEN>` y `<SUBSCRIPTION ID>` por los valores adecuados para su entorno.

   La versión de API `2018-06-01` almacena las identidades administradas asignadas por el usuario en el valor `userAssignedIdentities` en un formato de diccionario, en contraposición con el valor `identityIds` en formato de matriz que se usaba en la versión `2017-12-01` de la API.
   
   **VERSIÓN DE API 2018-06-01**

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned, UserAssigned", "userAssignedIdentities":{"/subscriptions/<<SUBSCRIPTION ID>>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{},"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":{}}}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```
   **Encabezados de solicitud**

   |Encabezado de solicitud  |Descripción  |
   |---------|---------|
   |*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
   |*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        | 

   **Cuerpo de la solicitud**

   ```JSON
    {  
       "identity":{  
          "type":"SystemAssigned, UserAssigned",
          "userAssignedIdentities":{  
             "/subscriptions/<<SUBSCRIPTION ID>>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{  
    
             },
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":{  
    
             }
          }
       }
    }
   ```

   **VERSIÓN DE API 2017-12-01**

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01' -X PATCH -d '{"identity":{"type":"SystemAssigned, UserAssigned", "identityIds":["/subscriptions/<<SUBSCRIPTION ID>>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1","/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2"]}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01 HTTP/1.1
   ```
    
   **Encabezados de solicitud**

   |Encabezado de solicitud  |Descripción  |
   |---------|---------|
   |*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
   |*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        | 

   **Cuerpo de la solicitud**

   ```JSON
    {  
       "identity":{  
          "type":"SystemAssigned, UserAssigned",
          "identityIds":[  
             "/subscriptions/<<SUBSCRIPTION ID>>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1",
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2"
          ]
       }
    }
   ```   

### <a name="disable-system-assigned-managed-identity-from-an-azure-vm"></a>Deshabilitación de una identidad administrada asignada por el sistema de una VM de Azure

Para deshabilitar una identidad administrada asignada por el sistema en una máquina virtual, la cuenta necesita la asignación de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  No se requiere ninguna otra asignación de roles de directorio de Azure AD.

1. Recupere un token de acceso de portador, que utilizará en el siguiente paso en el encabezado de autorización para crear la máquina virtual con una identidad administrada asignada por el sistema.

   ```azurecli-interactive
   az account get-access-token
   ```

2. Actualice la máquina virtual mediante CURL para llamar al punto de conexión REST de Azure Resource Manager para deshabilitar la identidad administrada asignada por el sistema.  En el ejemplo siguiente, se deshabilita una identidad administrada asignada por el sistema en el cuerpo de la solicitud por el valor `{"identity":{"type":"None"}}` desde una máquina virtual denominada *myVM*.  Reemplace `<ACCESS TOKEN>` por el valor que ha recibido en el paso anterior cuando solicitó un token de acceso de portador y el valor `<SUBSCRIPTION ID>` según sea apropiado para su entorno.

   > [!IMPORTANT]
   > Para asegurarse de que no elimina ninguna de las identidades administradas asignadas por el usuario que están asignadas a la máquina virtual, necesita enumerar las identidades administradas asignadas por el usuario mediante este comando CURL: `curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>?api-version=2018-06-01' -H "Authorization: Bearer <ACCESS TOKEN>"`. Si tiene asignadas identidades administradas asignadas por el usuario a la máquina virtual como se identifica en el valor `identity` de la respuesta, vaya al paso 3 que muestra cómo conservar las identidades administradas asignadas por el usuario mientras se deshabilita la identidad administrada asignada por el sistema en la máquina virtual.

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"None"}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```
   **Encabezados de solicitud**

   |Encabezado de solicitud  |Descripción  |
   |---------|---------|
   |*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
   |*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        | 

   **Cuerpo de la solicitud**

   ```JSON
    {  
       "identity":{  
          "type":"None"
       }
    }
   ```

   Para quitar la identidad administrada asignada por el sistema de una máquina virtual que tiene identidades administradas asignadas por el usuario, quite `SystemAssigned` del valor `{"identity":{"type:" "}}` mientras mantiene el valor `UserAssigned` y los valores de diccionario `userAssignedIdentities`, si está utilizando la **versión de API 2018-06-01**. Si está usando la **versión de API 2017-12-01** o versiones anteriores, mantenga la matriz `identityIds`.

## <a name="user-assigned-managed-identity"></a>Identidad administrada asignada por el usuario

En esta sección, aprenderá a agregar y a quitar una identidad administrada asignada por el usuario en una máquina virtual de Azure mediante CURL para llamar al punto de conexión de REST de Azure Resource Manager.

### <a name="assign-a-user-assigned-managed-identity-during-the-creation-of-an-azure-vm"></a>Asignación de una identidad administrada asignada por el usuario durante la creación de una VM de Azure

Para asignar una identidad asignada por un usuario a una máquina virtual, la cuenta debe tener las asignaciones de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) y [Operador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-operator). No se requiere ninguna otra asignación de roles de directorio de Azure AD.

1. Recupere un token de acceso de portador, que utilizará en el siguiente paso en el encabezado de autorización para crear la máquina virtual con una identidad administrada asignada por el sistema.

   ```azurecli-interactive
   az account get-access-token
   ```

2. Cree una [interfaz de red](/cli/azure/network/nic?view=azure-cli-latest#az-network-nic-create) para la máquina virtual:

   ```azurecli-interactive
    az network nic create -g myResourceGroup --vnet-name myVnet --subnet mySubnet -n myNic
   ```

3. Recupere un token de acceso de portador, que utilizará en el siguiente paso en el encabezado de autorización para crear la máquina virtual con una identidad administrada asignada por el sistema.

   ```azurecli-interactive
   az account get-access-token
   ``` 

4. Cree una identidad asignada por el usuario mediante las instrucciones que se encuentran aquí: [Creación de una identidad administrada asignada por el usuario](how-to-manage-ua-identity-rest.md#create-a-user-assigned-managed-identity).

5. Cree una máquina virtual con CURL para llamar al punto de conexión REST de Azure Resource Manager. En el ejemplo siguiente se crea una máquina virtual denominada *myVM* en el grupo de recursos *myResourceGroup* con una identidad administrada asignada por el usuario `ID1`, como se identificó en el cuerpo de la solicitud por el valor `"identity":{"type":"UserAssigned"}`. Reemplace `<ACCESS TOKEN>` por el valor que ha recibido en el paso anterior cuando solicitó un token de acceso de portador y el valor `<SUBSCRIPTION ID>` según sea apropiado para su entorno.
 
   **VERSIÓN DE API 2018-06-01**

   ```bash   
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PUT -d '{"location":"westus","name":"myVM","identity":{"type":"UserAssigned","identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"]},"properties":{"hardwareProfile":{"vmSize":"Standard_D2_v2"},"storageProfile":{"imageReference":{"sku":"2016-Datacenter","publisher":"MicrosoftWindowsServer","version":"latest","offer":"WindowsServer"},"osDisk":{"caching":"ReadWrite","managedDisk":{"storageAccountType":"Standard_LRS"},"name":"myVM3osdisk","createOption":"FromImage"},"dataDisks":[{"diskSizeGB":1023,"createOption":"Empty","lun":0},{"diskSizeGB":1023,"createOption":"Empty","lun":1}]},"osProfile":{"adminUsername":"azureuser","computerName":"myVM","adminPassword":"myPassword12"},"networkProfile":{"networkInterfaces":[{"id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic","properties":{"primary":true}}]}}}' -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
   ```
    
   ```HTTP
   PUT https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```

   **Encabezados de solicitud**

   |Encabezado de solicitud  |Descripción  |
   |---------|---------|
   |*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
   |*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        | 

   **Cuerpo de la solicitud**

   ```JSON
    {  
       "location":"westus",
       "name":"myVM",
       "identity":{  
          "type":"UserAssigned",
          "identityIds":[  
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"
          ]
       },
       "properties":{  
          "hardwareProfile":{  
             "vmSize":"Standard_D2_v2"
          },
          "storageProfile":{  
             "imageReference":{  
                "sku":"2016-Datacenter",
                "publisher":"MicrosoftWindowsServer",
                "version":"latest",
                "offer":"WindowsServer"
             },
             "osDisk":{  
                "caching":"ReadWrite",
                "managedDisk":{  
                   "storageAccountType":"Standard_LRS"
                },
                "name":"myVM3osdisk",
                "createOption":"FromImage"
             },
             "dataDisks":[  
                {  
                   "diskSizeGB":1023,
                   "createOption":"Empty",
                   "lun":0
                },
                {  
                   "diskSizeGB":1023,
                   "createOption":"Empty",
                   "lun":1
                }
             ]
          },
          "osProfile":{  
             "adminUsername":"azureuser",
             "computerName":"myVM",
             "adminPassword":"myPassword12"
          },
          "networkProfile":{  
             "networkInterfaces":[  
                {  
                   "id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic",
                   "properties":{  
                      "primary":true
                   }
                }
             ]
          }
       }
    }

   ```
  
   **VERSIÓN DE API 2017-12-01**

   ```bash   
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01' -X PUT -d '{"location":"westus","name":"myVM","identity":{"type":"UserAssigned","identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"]},"properties":{"hardwareProfile":{"vmSize":"Standard_D2_v2"},"storageProfile":{"imageReference":{"sku":"2016-Datacenter","publisher":"MicrosoftWindowsServer","version":"latest","offer":"WindowsServer"},"osDisk":{"caching":"ReadWrite","managedDisk":{"storageAccountType":"Standard_LRS"},"name":"myVM3osdisk","createOption":"FromImage"},"dataDisks":[{"diskSizeGB":1023,"createOption":"Empty","lun":0},{"diskSizeGB":1023,"createOption":"Empty","lun":1}]},"osProfile":{"adminUsername":"azureuser","computerName":"myVM","adminPassword":"myPassword12"},"networkProfile":{"networkInterfaces":[{"id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic","properties":{"primary":true}}]}}}' -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PUT https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01 HTTP/1.1
   ```

   **Encabezados de solicitud**

   |Encabezado de solicitud  |Descripción  |
   |---------|---------|
   |*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
   |*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        | 

   **Cuerpo de la solicitud**

   ```JSON
    {
       "location":"westus",
       "name":"myVM",
       "identity":{
          "type":"UserAssigned",
          "identityIds":[
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"
          ]
       },
       "properties":{
          "hardwareProfile":{
             "vmSize":"Standard_D2_v2"
          },
          "storageProfile":{
             "imageReference":{
                "sku":"2016-Datacenter",
                "publisher":"MicrosoftWindowsServer",
                "version":"latest",
                "offer":"WindowsServer"
             },
             "osDisk":{
                "caching":"ReadWrite",
                "managedDisk":{
                   "storageAccountType":"Standard_LRS"
                },
                "name":"myVM3osdisk",
                "createOption":"FromImage"
             },
             "dataDisks":[
                {
                   "diskSizeGB":1023,
                   "createOption":"Empty",
                   "lun":0
                },
                {
                   "diskSizeGB":1023,
                   "createOption":"Empty",
                   "lun":1
                }
             ]
          },
          "osProfile":{
             "adminUsername":"azureuser",
             "computerName":"myVM",
             "adminPassword":"myPassword12"
          },
          "networkProfile":{
             "networkInterfaces":[
                {
                   "id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic",
                   "properties":{
                      "primary":true
                   }
                }
             ]
          }
       }
    }
   ```

### <a name="assign-a-user-assigned-managed-identity-to-an-existing-azure-vm"></a>Asignación de una identidad administrada asignada por el usuario a una VM de Azure existente

Para asignar una identidad asignada por un usuario a una máquina virtual, la cuenta debe tener las asignaciones de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) y [Operador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-operator). No se requiere ninguna otra asignación de roles de directorio de Azure AD.

1. Recupere un token de acceso de portador, que utilizará en el siguiente paso en el encabezado de autorización para crear la máquina virtual con una identidad administrada asignada por el sistema.

   ```azurecli-interactive
   az account get-access-token
   ```

2.  Cree una identidad administrada asignada por el usuario mediante las instrucciones que se encuentran aquí: [Creación de una identidad administrada asignada por el usuario](how-to-manage-ua-identity-rest.md#create-a-user-assigned-managed-identity).

3. Para asegurarse de que no elimina las identidades administradas asignadas por el usuario o por el sistema que están asignadas a la máquina virtual, necesita enumerar los tipos de identidades asignados a la máquina virtual mediante el comando CURL siguiente. Si ha administrado identidades asignadas al conjunto de escalado de máquinas virtuales, se enumeran en el valor `identity`.

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>?api-version=2018-06-01' -H "Authorization: Bearer <ACCESS TOKEN>" 
   ```

   ```HTTP
   GET https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>?api-version=2018-06-01 HTTP/1.1
   ```
   **Encabezados de solicitud**

   |Encabezado de solicitud  |Descripción  |
   |---------|---------|
   |*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.

    Si tiene identidades administradas asignadas por el usuario o por el sistema asignadas a la máquina virtual como se identifica en el valor `identity` de la respuesta, vaya al paso 5, en el que se muestra cómo conservar la identidad administrada asignada por el sistema mientras se agrega una identidad administrada asignada por el usuario en la máquina virtual.

4. Si no tiene ninguna identidad administrada asignada por el usuario en la máquina virtual, use el siguiente comando CURL para llamar al punto de conexión REST de Azure Resource Manager para asignar la primera identidad administrada asignada por el usuario a la máquina virtual.

   En el ejemplo siguiente, se asigna una identidad administrada asignada por el usuario, `ID1`, a una máquina virtual llamada *myVM* en el grupo de recursos *myResourceGroup*.  Reemplace `<ACCESS TOKEN>` por el valor que ha recibido en el paso anterior cuando solicitó un token de acceso de portador y el valor `<SUBSCRIPTION ID>` según sea apropiado para su entorno.

   **VERSIÓN DE API 2018-06-01**

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"UserAssigned", "userAssignedIdentities":{"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{}}}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```
   **Encabezados de solicitud**

   |Encabezado de solicitud  |Descripción  |
   |---------|---------|
   |*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
   |*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        |
 
   **Cuerpo de la solicitud**

   ```JSON
    {
       "identity":{
          "type":"UserAssigned",
          "userAssignedIdentities":{
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{
    
             }
          }
       }
    }
   ```

   **VERSIÓN DE API 2017-12-01**

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01' -X PATCH -d '{"identity":{"type":"userAssigned", "identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"]}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01 HTTP/1.1
   ```
   
   **Encabezados de solicitud**

   |Encabezado de solicitud  |Descripción  |
   |---------|---------|
   |*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
   |*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        | 

   **Cuerpo de la solicitud**

   ```JSON
    {
       "identity":{
          "type":"userAssigned",
          "identityIds":[
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"
          ]
       }
    }
   ```

5. Si dispone ya de una identidad asignada por el usuario o por el sistema asignada a la máquina virtual:
   
   **VERSIÓN DE API 2018-06-01**

   Agregue la identidad administrada asignada por el usuario al valor de diccionario `userAssignedIdentities`.
    
   Por ejemplo, si tiene una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario `ID1` actualmente asignadas a la máquina virtual y quiere agregarles la identidad administrada asignada por el usuario `ID2`:

   ```bash
   curl  'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned, UserAssigned", "userAssignedIdentities":{"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{},"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":{}}}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```
   
   **Encabezados de solicitud**

   |Encabezado de solicitud  |Descripción  |
   |---------|---------|
   |*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
   |*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        | 

   **Cuerpo de la solicitud**

   ```JSON
    {
       "identity":{
          "type":"SystemAssigned, UserAssigned",
          "userAssignedIdentities":{
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{
    
             },
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":{
    
             }
          }
       }
    }
   ```

   **VERSIÓN DE API 2017-12-01**

   Conserve las identidades administradas asignadas por el usuario y que le gustaría mantener en el valor de matriz `identityIds` al tiempo que agrega la nueva identidad administrada asignada por el usuario.

   Por ejemplo, si tiene una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario `ID1` actualmente asignadas a la máquina virtual y quiere agregarles la identidad administrada asignada por el usuario `ID2`: 

   ```bash
   curl  'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01' -X PATCH -d '{"identity":{"type":"SystemAssigned,UserAssigned", "identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1","/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2"]}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01 HTTP/1.1
   ```

   **Encabezados de solicitud**

   |Encabezado de solicitud  |Descripción  |
   |---------|---------|
   |*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
   |*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        | 

   **Cuerpo de la solicitud**

   ```JSON
    {
       "identity":{
          "type":"SystemAssigned,UserAssigned",
          "identityIds":[
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1",
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2"
          ]
       }
    }
   ```   

### <a name="remove-a-user-assigned-managed-identity-from-an-azure-vm"></a>Eliminación de una identidad administrada asignada por el usuario de una VM de Azure

Para quitar una identidad asignada por un usuario de una máquina virtual, la cuenta necesita la asignación de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).

1. Recupere un token de acceso de portador, que utilizará en el siguiente paso en el encabezado de autorización para crear la máquina virtual con una identidad administrada asignada por el sistema.

   ```azurecli-interactive
   az account get-access-token
   ```

2. Para asegurarse de que no elimina ninguna identidad administrada asignada por el usuario o por el sistema que está asignada a la máquina virtual y de que no quita la identidad administrada asignada por el sistema, necesita enumerar las identidades administradas mediante el comando CURL siguiente: 

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>?api-version=2018-06-01' -H "Authorization: Bearer <ACCESS TOKEN>"
   ```
 
   ```HTTP
   GET https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>?api-version=2018-06-01 HTTP/1.1
   ```

   **Encabezados de solicitud**

   |Encabezado de solicitud  |Descripción  |
   |---------|---------|
   |*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
   |*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.
 
   Si ha administrado identidades asignadas a la máquina virtual, se enumeran en la respuesta del valor `identity`.

   Por ejemplo, si tiene identidades administradas asignadas por el usuario `ID1` y `ID2` asignadas a la máquina virtual, y solo desea mantener `ID1` asignada y conservar la identidad asignada por el sistema:
   
   **VERSIÓN DE API 2018-06-01**

   Agregue `null` a la identidad administrada asignada por el usuario que quiere quitar:

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned, UserAssigned", "userAssignedIdentities":{"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":null}}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```

   **Encabezados de solicitud**

   |Encabezado de solicitud  |Descripción  |
   |---------|---------|
   |*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
   |*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        | 

   **Cuerpo de la solicitud**

   ```JSON
    {
       "identity":{
          "type":"SystemAssigned, UserAssigned",
          "userAssignedIdentities":{
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":null
          }
       }
    }
   ```

   **VERSIÓN DE API 2017-12-01**

   Conserve solo las identidades administradas asignadas por el usuario que le gustaría mantener en la matriz `identityIds`:

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01' -X PATCH -d '{"identity":{"type":"SystemAssigned, UserAssigned", "identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"]}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01 HTTP/1.1
   ```

   **Encabezados de solicitud**

   |Encabezado de solicitud  |Descripción  |
   |---------|---------|
   |*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
   |*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        | 

   **Cuerpo de la solicitud**

   ```JSON
    {
       "identity":{
          "type":"SystemAssigned, UserAssigned",
          "identityIds":[
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"
          ]
       }
    }
   ```

Si la máquina virtual tiene identidades administradas asignadas tanto por el sistema como por el usuario, puede quitar todas las identidades administradas asignadas por el usuario si cambia para usar solo las identidades administradas asignadas por el sistema mediante el siguiente comando:

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned"}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
```

```HTTP
PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
```

**Encabezados de solicitud**

|Encabezado de solicitud  |Descripción  |
|---------|---------|
|*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
|*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido. | 

**Cuerpo de la solicitud**

```JSON
{
   "identity":{
      "type":"SystemAssigned"
   }
}
```
    
Si la máquina virtual solo tiene identidades administradas asignadas por el usuario y desea quitarlas todas, utilice el comando siguiente:

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"None"}}' -H "Content-Type: application/json" -H Authorization:"Bearer <ACCESS TOKEN>"
```

```HTTP
PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
```

**Encabezados de solicitud**

|Encabezado de solicitud  |Descripción  |
|---------|---------|
|*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
|*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.| 

**Cuerpo de la solicitud**

```JSON
{
   "identity":{
      "type":"None"
   }
}
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo crear, enumerar o eliminar identidades administradas asignadas por el usuario mediante REST, consulte:

- [Creación, enumeración o eliminación de identidades administradas asignadas por el usuario mediante llamadas a la API REST](how-to-manage-ua-identity-rest.md)