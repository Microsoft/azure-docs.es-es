---
title: Creación de un rol personalizado de Azure Resource Manager y asignación a una entidad de servicio en Azure
description: En este artículo se proporcionan instrucciones sobre cómo crear un rol personalizado de Azure Resource Manager y asignarlo a una entidad de servicio para Análisis de vídeos en vivo en IoT Edge mediante la CLI de Azure.
ms.topic: how-to
ms.date: 05/27/2020
ms.openlocfilehash: eb4c9a1f90ab50f7070184fc9a394d9e6edb833a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87043171"
---
# <a name="create-custom-azure-resource-manager-role-and-assign-to-service-principal"></a>Creación de un rol personalizado de Azure Resource Manager y asignación a una entidad de servicio

La instancia del módulo Análisis de vídeos en vivo en IoT Edge necesita una cuenta activa de Azure Media Services para que funcione correctamente. La relación entre el módulo Análisis de vídeos en vivo en IoT Edge y la cuenta de Azure Media Services se establece mediante un conjunto de propiedades del módulo gemelo. Una de esas propiedades gemelas es una [entidad de servicio](../../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) que permite que la instancia del módulo se comunique con las operaciones necesarias y las desencadene en la cuenta de Media Services. Para minimizar los posibles usos indebidos o la exposición accidental de datos del dispositivo perimetral, esta entidad de servicio debe tener la cantidad mínima de privilegios.

En este artículo se describen los pasos para crear un rol personalizado de Azure Resource Manager con Azure Cloud Shell, que se usa para crear una entidad de servicio.

## <a name="prerequisites"></a>Requisitos previos  

Los requisitos previos para este artículo son los siguientes:

* Suscripción a Azure como propietario.
* Una instancia de Azure Active Directory con privilegios para crear una aplicación y asignar una entidad de servicio a un rol.

El portal representa la forma más sencilla de comprobar si su cuenta tiene los permisos adecuados. Consulte el [artículo que explica cómo comprobar el permiso requerido](../../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

## <a name="overview"></a>Información general  

Seguiremos los pasos para crear un rol personalizado y vincularlo a una entidad de servicio en el siguiente orden:

1. Cree una cuenta de Media Services, si aún no la tiene.
1. Crear una entidad de servicio.
1. Cree un rol personalizado de Azure Resource Manager con privilegios limitados.
1. "Restrinja" los privilegios de la entidad de servicio mediante el rol personalizado creado.
1. Ejecute una prueba sencilla para ver si se puede restringir correctamente la entidad de servicio.
1. Capture los parámetros que se usarán en los manifiestos de implementación de IoT Edge.

### <a name="create-a-media-services-account"></a>Creación de una cuenta de Media Services  

Si no tiene una cuenta de Media Services, use los pasos siguientes para crear una.

1. Vaya a [Cloud Shell](https://shell.azure.com/).
1. Seleccione "Bash" como su entorno en la lista desplegable del lado izquierdo de la ventana del shell.

    ![Bash](./media/create-custom-azure-resource-manager-role-how-to/bash.png)
1. Establezca la suscripción a Azure como la cuenta predeterminada mediante la siguiente plantilla de comandos:
    
    ```
    az account set --subscription " <yourSubscriptionName or yourSubscriptionId>"
    ```
1. Crear un [grupo de recursos](/cli/azure/group?view=azure-cli-latest#az-group-create) y una [cuenta de almacenamiento](/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-create).
1. Ahora, cree una cuenta de Azure Media Services con la siguiente plantilla de comandos de Cloud Shell:

    ```
    az ams account create --name <yourAMSAccountName>  --resource-group <yourResouceGroup>  --storage-account <yourStorageAccountName>
    ```

### <a name="create-service-principal"></a>Creación de una entidad de servicio  

Ahora, se creará una entidad de servicio que se vinculará a la cuenta de Media Services.

La autenticación basada en contraseña no tiene ningún parámetro de autenticación y se usa con una contraseña aleatoria para la entidad de servicio. En Cloud Shell, use la siguiente plantilla de comandos:

```
az ams account sp create --account-name < yourAMSAccountName > --resource-group < yourResouceGroup >
```

Este comando genera una respuesta como esta:

```
{
  "AadClientId": "00000000-0000-0000-0000-000000000000",
  "AadEndpoint": "https://login.microsoftonline.com",
  "AadSecret": "<yourServicePrincipalPassword>",
  "AadTenantId": "00000000-0000-0000-0000-000000000000",
  "AccountName": " <yourAMSAccountName >",
  "ArmAadAudience": "https://management.core.windows.net/",
  "ArmEndpoint": "https://management.azure.com/",
  "Region": "Central US",
  "ResourceGroup": " <yourResouceGroup >",
  "SubscriptionId": "<yourSubscriptionId>"
}

```
1. La salida de una entidad de servicio con una autenticación de contraseña incluye la clave de contraseña que, en este caso, es el parámetro “AadSecret”. 

    No olvide copiar este valor porque no se puede recuperar. Si olvida la contraseña, [restablezca las credenciales de la entidad de servicio](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest#reset-credentials).
1. El identificador de aplicación y la clave de inquilino aparecen en la salida como "AadClientId" y "AadTenantId", respectivamente. Se usan en la autenticación de la entidad de servicio. Anote sus valores, aunque se pueden recuperar en cualquier momento con [az ad sp list](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-list).

### <a name="create-a-custom-role-definition"></a>Creación de una definición de rol personalizado  

Para crear un rol personalizado, estos son los pasos que debe seguir:

1. Cree un archivo JSON de definición de roles en el sistema local y guarde el siguiente texto en el archivo. 
    1. Reemplace <yourSubscriptionId> por el identificador de la suscripción a Azure.
    1. Las únicas acciones permitidas para este rol son:
        * listContainerSas: ayuda al módulo a enumerar las direcciones URL del contenedor de almacenamiento con firmas de acceso compartido (SAS) para cargar y descargar el contenido de los recursos.
        * Escritura de recursos: ayuda al módulo a crear o actualizar cualquier recurso.
        * listEdgePolicies: enumera las directivas que se aplican al dispositivo perimetral.  
        
        ```
        {
          "Name": "LVAEdge User",
          "IsCustom": true,
          "Description": "Can create assets, view list of containers and view Edge policies",
          "Actions": [
            "Microsoft.Media/mediaservices/assets/listContainerSas/action",
            "Microsoft.Media/mediaservices/assets/write",
            "Microsoft.Media/mediaservices/listEdgePolicies/action"
          ],
          "NotActions": [],
          "DataActions": [],
          "NotDataActions": [],
          "AssignableScopes": [
            "/subscriptions/<yourSubscriptionId>"
          ]
        }
        ```  
          
1. Una vez creada, ejecute la siguiente plantilla de comandos para crear la definición de roles en la suscripción:
    
    ```
    az role definition create --role-definition "<location of the Role Definition JSON file >"
    ```

    Tras la ejecución correcta del comando, verá la siguiente salida:
    
    ```
    {
      "assignableScopes": [
      "/subscriptions/<yourSubscriptionId>"
      ],
      "description": "Can create assets, view list of containers and view Edge policies",
      "id": "/subscriptions/<yourSubscriptionId>/providers/Microsoft.Authorization/roleDefinitions/<unique name>",
      "name": "<unique name>",
      "permissions": [
        {
          "actions": [
            "Microsoft.Media/mediaservices/assets/listContainerSas/action",
            "Microsoft.Media/mediaservices/assets/write",
            "Microsoft.Media/mediaservices/listEdgePolicies/action",
          ],
          "dataActions": [],
          "notActions": [],
          "notDataActions": []
        }
      ],
      "roleName": " LVAEdge User ",
      "roleType": "CustomRole",
      "type": "Microsoft.Authorization/roleDefinitions"
    }
    ```

### <a name="create-role-assignment"></a>Creación de asignaciones de roles  

Para agregar una asignación de roles, necesitará el valor objectId de la entidad de servicio que desea asignar al rol personalizado que acaba de crear.

Use el siguiente comando en Cloud Shell para obtener el valor objectId:

```
az ad sp show --id "<appId>" | Select-String "objectId"
```

> [!NOTE]
> `<appId>` se puede recuperar de la salida del paso [Creación de una entidad de servicio](#create-service-principal).

El comando anterior imprimirá el valor objectId de la entidad de servicio. 

```
“objectId” : “<yourObjectId>”,
```

Use la plantilla [az role assignment create command](/cli/azure/role/assignment?view=azure-cli-latest#az-role-assignment-create) para vincular el rol personalizado a la entidad de servicio:

```
az role assignment create --role “LVAEdge User” --assignee-object-id < objectId>    
```

Parámetros:

|Parámetros|Descripción| 
|---|---|
|--role |Identificador o nombre del rol personalizado. En nuestro caso: “LVAEdge User”.|
|--assignee-object-id|El identificador de objeto de la entidad de servicio que se utilizará.|

El resultado tendrá este aspecto:

```
{
  "canDelegate": null,
  "id": "/subscriptions/<yourSubscriptionId>/providers/Microsoft.Authorization/roleAssignments/<yourCustomRoleId>",
  "name": "<yourCustomRoleId>",
  "principalId": "<yourServicePrincipalId>",
  "principalType": "ServicePrincipal",
  "roleDefinitionId": "/subscriptions/<yourSubscriptionId>/providers/Microsoft.Authorization/roleDefinitions/<yourRoleDefinitionId>",
  "scope": "/subscriptions/<yourSubscriptionId>",
  "type": "Microsoft.Authorization/roleAssignments"
} 
```

### <a name="confirm-that-role-assignment-happened"></a>Confirmación de la realización de la asignación de roles

Para confirmar que la entidad de servicio ya está vinculada con el rol personalizado que se acaba de crear, ejecute el siguiente comando:

```
az role assignment list  --assignee < objectId>
```

El resultado debería tener este aspecto:

```
[
  {
    "canDelegate": null,
    "id": "/subscriptions/xxx/providers/Microsoft.Authorization/roleAssignments/00000000-0000-0000-0000-000000000000",
    "name": "00000000-0000-0000-0000-000000000000",
    "principalId": "<yourServicePrincipalID>",
    "principalName": "<yourServicePrincipalName>",
    "principalType": "ServicePrincipal",
    "roleDefinitionId": "/subscriptions/xxx/providers/Microsoft.Authorization/roleDefinitions/zzz",
    "roleDefinitionName": "LVAEdge User",
    "scope": "/subscriptions/<yourSubscription ID>",
    "type": "Microsoft.Authorization/roleAssignments
  }
]  
```
 
Busque "roleDefinitionName" y compruebe que su valor está establecido en "LVAEdge User". 

Esto confirma que se ha vinculado el rol de usuario personalizado a la entidad de servicio que se usa para nuestra aplicación.

### <a name="test-the-service-principal-rbac"></a>Prueba del rol RBAC de la entidad de servicio  

1. Inicie sesión con la entidad de servicio. Para ello, se necesitan tres fragmentos de información para que Azure Active Directory nos otorgue el token de acceso adecuado que podemos obtener de la salida del paso [Creación de una entidad de servicio](#create-service-principal):
    1. AadClientID 
    1. AadSecret
    1. AadTenantId
1. Ahora, intente iniciar sesión con la siguiente plantilla de comandos:
    
    ```
    az login --service-principal --username "< AadClientID>" --password " <AadSecret>" --tenant "<AadTenantId>"
    ```
3.  Ahora, veamos si el inicio de sesión está restringido a la entidad de servicio con el rol "usuario LVAEdge" al intentar crear un grupo de recursos para comprobar si se produce un error. En Cloud Shell, ejecute el comando siguiente:

    ```
    az group create --location "central us" --name "testresourcegroup"
    ```

    Este comando debe producir un error con el siguiente aspecto:
    
    ```
    The client '<AadClientId>' with object id '<AadClientId>' does not have authorization to perform action 'Microsoft.Resources/subscriptions/resourcegroups/write' over scope '/subscriptions/<yourSubscriptionId>/resourcegroups/testresourcegroup' or the scope is invalid. If access was recently granted, please refresh your credentials.
    ```

## <a name="next-steps"></a>Pasos siguientes  

Tenga en cuenta los siguientes valores de este artículo. Estos valores serán necesarios para configurar las propiedades gemelas del módulo Análisis de vídeos en vivo en IoT Edge; vea [Esquema JSON de módulo gemelo](module-twin-configuration-schema.md).

| Variable de este artículo|Nombre de propiedad gemela para Análisis de vídeos en vivo en IoT Edge|
|---|---|
|AadSecret |    aadServicePrincipalPassword|
|AadTenantId |  aadTenantId|
|AadClientId    |aadServicePrincipalAppId|
