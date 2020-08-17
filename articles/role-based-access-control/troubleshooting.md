---
title: Solución de problemas de Azure RBAC
description: Solución de problemas con el control de acceso basado en rol de Azure (Azure RBAC).
services: azure-portal
documentationcenter: na
author: rolyon
manager: mtillman
ms.assetid: df42cca2-02d6-4f3c-9d56-260e1eb7dc44
ms.service: role-based-access-control
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 07/28/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.custom: seohack1
ms.openlocfilehash: 839662e496a61ff9a90a6250b417688b91ccaed1
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87382583"
---
# <a name="troubleshoot-azure-rbac"></a>Solución de problemas de Azure RBAC

En este artículo se responden algunas preguntas comunes acerca del control de acceso basado en rol de Azure (Azure RBAC) para que sepa qué esperar cuando use los roles y para que pueda solucionar los problemas de acceso.

## <a name="azure-role-assignments-limit"></a>Límite de asignaciones de roles de Azure

Azure admite hasta **2000** asignaciones de roles por suscripción. Este límite incluye las asignaciones de roles en los ámbitos de suscripción, grupo de recursos y recurso. Si aparece el mensaje de error "No se pueden crear más asignaciones de roles (código: RoleAssignmentLimitExceeded)" al intentar asignar un rol, pruebe a reducir el número de asignaciones de roles de la suscripción.

> [!NOTE]
> El límite de **2000** asignaciones de roles por suscripción es fijo y no se puede aumentar.

Si está cerca de este límite, estas son algunas maneras de reducir el número de asignaciones de roles:

- Agregue usuarios a grupos y asigne los roles a los grupos. 
- Combine varios roles integrados con un rol personalizado. 
- Cree asignaciones de roles comunes en un ámbito superior, como en un grupo de suscripciones o en un grupo de administración.
- Si tiene Azure AD Premium P2, permita que las asignaciones de roles se puedan establecer en [Azure AD Privileged Identity Management](../active-directory/privileged-identity-management/pim-configure.md) en lugar de asignarlas de forma permanente. 
- Agregue una suscripción adicional. 

Para saber el número de asignaciones de roles, puede consultar el [gráfico de la página Control de acceso (IAM)](role-assignments-list-portal.md#list-number-of-role-assignments) en Azure Portal. También puede usar los siguientes comandos de Azure PowerShell:

```azurepowershell
$scope = "/subscriptions/<subscriptionId>"
$ras = Get-AzRoleAssignment -Scope $scope | Where-Object {$_.scope.StartsWith($scope)}
$ras.Count
```

## <a name="problems-with-azure-role-assignments"></a>Problemas con las asignaciones de roles de Azure

- Si no puede agregar una asignación de roles en Azure Portal, en **Control de acceso (IAM)** porque la opción **Agregar** > **Agregar asignación de roles** está deshabilitada o porque recibe el error de permisos "El cliente con el identificador de objeto no está autorizado para realizar la acción", compruebe que ha iniciado sesión con un usuario que tenga asignado un rol con el permiso `Microsoft.Authorization/roleAssignments/write`, como [Propietario](built-in-roles.md#owner) o [Administrador de acceso de usuario](built-in-roles.md#user-access-administrator), en el ámbito en el que intenta asignar el rol.
- Si usa una entidad de servicio para asignar roles, puede obtener el error "No tiene privilegios suficientes para completar la operación". Por ejemplo, supongamos que tiene una entidad de servicio a la que se le ha asignado el rol de propietario e intenta crear la siguiente asignación de roles como entidad de servicio mediante la CLI de Azure:

    ```azurecli
    az login --service-principal --username "SPNid" --password "password" --tenant "tenantid"
    az role assignment create --assignee "userupn" --role "Contributor"  --scope "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}"
    ```

    Si obtiene el error "No tiene privilegios suficientes para completar la operación", es probable que se deba a que la CLI de Azure está intentando buscar la identidad de asignación en Azure AD y la entidad de servicio no puede leer Azure AD de manera predeterminada.

    Hay dos maneras de resolver este error. La primera consiste en asignar el rol [lectores de directorio](../active-directory/users-groups-roles/directory-assign-admin-roles.md#directory-readers) a la entidad de servicio para que pueda leer los datos en el directorio.

    La segunda manera de resolver este error es crear la asignación de roles mediante el parámetro `--assignee-object-id` en lugar de `--assignee`. Mediante el uso de `--assignee-object-id`, la CLI de Azure omitirá la búsqueda de Azure AD. Debe obtener el identificador de objeto del usuario, del grupo o de la aplicación a los que quiere asignar el rol. Para más información, consulte [Incorporación o eliminación de asignaciones de roles de Azure mediante la CLI de Azure](role-assignments-cli.md#new-service-principal).

    ```azurecli
    az role assignment create --assignee-object-id 11111111-1111-1111-1111-111111111111  --role "Contributor" --scope "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}"
    ```

## <a name="problems-with-custom-roles"></a>Problemas con roles personalizados

- Si necesita conocer los pasos para crear un rol personalizado, consulte los tutoriales sobre roles personalizados con [Azure Portal](custom-roles-portal.md) (actualmente en versión preliminar), [Azure PowerShell](tutorial-custom-role-powershell.md) o la [CLI de Azure](tutorial-custom-role-cli.md).
- Si no puede actualizar un rol personalizado existente, compruebe que haya iniciado sesión con un usuario que tenga asignado un rol con el permiso `Microsoft.Authorization/roleDefinition/write`, como [Propietario](built-in-roles.md#owner) o [Administrador de acceso de usuario](built-in-roles.md#user-access-administrator).
- Si no puede eliminar un rol personalizado y obtiene el mensaje de error "Hay asignaciones de roles existentes que hacen referencia al rol (código: RoleDefinitionHasAssignments)", significa que hay asignaciones de roles que siguen usando el rol personalizado. Quite las asignaciones de roles y vuelva a intentar eliminarlo.
- Si recibe el mensaje de error "Se ha superado el límite de definiciones de roles. No se pueden crear más definiciones de roles (código: RoleDefinitionLimitExceeded)" al intentar crear un nuevo rol personalizado, elimine los roles personalizados que no se usan. Azure admite hasta **5000** roles personalizados en un directorio. (Para Azure Alemania y Azure China 21Vianet, el límite es 2000 roles personalizados).
- Si obtiene un error similar a "El cliente tiene permiso para realizar la acción 'Microsoft.Authorization/roleDefinitions/write' en el ámbito '/subscriptions/{subscriptionid}', aunque la suscripción vinculada no se encontró" al intentar actualizar un rol personalizado, consulte si se han eliminado uno o varios [ámbitos asignables](role-definitions.md#assignablescopes) del directorio. Si el ámbito se ha eliminado, cree una incidencia de soporte técnico porque no hay ninguna solución de autoservicio disponible en este momento.

## <a name="custom-roles-and-management-groups"></a>Roles personalizados y grupos de administración

- Solo puede definir un grupo de administración en `AssignableScopes` de un rol personalizado. La adición de un grupo de administración a `AssignableScopes` está actualmente en versión preliminar.
- No se pueden asignar roles personalizados con `DataActions` en el ámbito del grupo de administración.
- Azure Resource Manager no valida la existencia del grupo de administración en el ámbito asignable de la definición de roles.
- Para obtener más información sobre los roles personalizados y los grupos de administración, consulte el artículo [Organización de los recursos con grupos de administración de Azure](../governance/management-groups/overview.md#azure-custom-role-definition-and-assignment).

## <a name="transferring-a-subscription-to-a-different-directory"></a>Transferencia de una suscripción a un directorio diferente

- Si necesita consultar los pasos para transferir una suscripción a otro directorio de Azure AD, consulte [Transferencia de la propiedad de una suscripción de Azure a otra cuenta](../cost-management-billing/manage/billing-subscription-transfer.md).
- Si transfiere una suscripción a otro directorio de Azure AD, todas las asignaciones de roles se eliminan **permanentemente** del directorio de Azure AD de origen y no se migran al directorio de Azure AD de destino. Debe volver a crear las asignaciones de roles en el directorio de destino. También debe volver a crear manualmente las identidades administradas para los recursos de Azure. Para más información, consulte [Preguntas frecuentes y problemas conocidos con identidades administradas](../active-directory/managed-identities-azure-resources/known-issues.md).
- Si es un administrador global de Azure AD y no tiene acceso a una suscripción después de transferirla de un directorio a otro, active la **Administración del acceso para los recursos de Azure** para [elevar sus permisos de acceso](elevate-access-global-admin.md) temporalmente y obtener acceso a la suscripción.

## <a name="issues-with-service-admins-or-co-admins"></a>Problemas con los administradores o coadministradores de servicios

- Si tiene problemas con el administrador del servicio o los coadministradores, consulte [Agregar o cambiar los administradores de la suscripción de Azure](../cost-management-billing/manage/add-change-subscription-administrator.md) y [Roles de administrador de suscripciones clásico de RBAC de Azure y de administrador de Azure AD](rbac-and-directory-admin-roles.md).

## <a name="access-denied-or-permission-errors"></a>Errores de permiso o acceso denegado

- Si recibe el error de permiso "El cliente con el identificador de objeto no está autorizado para realizar la acción sobre el ámbito (código: AuthorizationFailed)" al intentar crear un recurso, compruebe que ha iniciado sesión con un usuario que tiene asignado un rol con permiso de escritura en el recurso y en el ámbito seleccionado. Por ejemplo, para administrar las máquinas virtuales de un grupo de recursos, debe tener el rol [Colaborador de máquina virtual](built-in-roles.md#virtual-machine-contributor) en el grupo de recursos (o el ámbito primario). Para ver una lista de los permisos para cada rol integrado, consulte [Roles integrados de Azure](built-in-roles.md).
- Si aparece el error de permiso "No tiene permiso para crear una solicitud de soporte técnico" al intentar crear o actualizar una incidencia de soporte técnico, compruebe que ha iniciado sesión con un usuario que tenga asignado un rol con el permiso `Microsoft.Support/supportTickets/write`, como [Colaborador de solicitud de soporte técnico](built-in-roles.md#support-request-contributor).

## <a name="role-assignments-with-identity-not-found"></a>Asignaciones de roles con identidad no encontrada

En la lista de asignaciones de roles para Azure Portal, es posible que observe que la entidad de seguridad (usuario, grupo, entidad de servicio o identidad administrada) aparece como **Identidad no encontrada** con un tipo **Desconocido**.

![Grupo de recursos de aplicación web](./media/troubleshooting/unknown-security-principal.png)

La identidad podría no encontrarse por dos motivos:

- Ha invitado recientemente a un usuario al crear una asignación de roles.
- Ha eliminado una entidad de seguridad que tenía una asignación de roles.

Si ha invitado recientemente a un usuario al crear una asignación de roles, esta entidad de seguridad podría seguir en el proceso de replicación entre regiones. De ser así, espere unos instantes y actualice la lista de asignaciones de roles.

Sin embargo, si esta entidad de seguridad no es un usuario invitado recientemente, podría tratarse de una entidad de seguridad eliminada. Si asigna un rol a una entidad de seguridad y, posteriormente, elimina esa entidad de seguridad sin quitar antes la asignación de roles, la entidad de seguridad se mostrará como **Identidad no encontrada** y con un tipo **Desconocido**.

Si enumera esta asignación de roles mediante Azure PowerShell, puede que vea un elemento `DisplayName` vacío y un elemento `ObjectType` definido como **Unknown**. Por ejemplo, [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment) devuelve una asignación de roles que es similar a la salida siguiente:

```
RoleAssignmentId   : /subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Authorization/roleAssignments/22222222-2222-2222-2222-222222222222
Scope              : /subscriptions/11111111-1111-1111-1111-111111111111
DisplayName        :
SignInName         :
RoleDefinitionName : Storage Blob Data Contributor
RoleDefinitionId   : ba92f5b4-2d11-453d-a403-e96b0029c9fe
ObjectId           : 33333333-3333-3333-3333-333333333333
ObjectType         : Unknown
CanDelegate        : False
```

Del mismo modo, si enumera esta asignación de roles con la CLI de Azure, podría ver un elemento `principalName` vacío. Por ejemplo, [az role assignment list](/cli/azure/role/assignment#az-role-assignment-list) devuelve una asignación de roles que es similar a la salida siguiente:

```
{
    "canDelegate": null,
    "id": "/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Authorization/roleAssignments/22222222-2222-2222-2222-222222222222",
    "name": "22222222-2222-2222-2222-222222222222",
    "principalId": "33333333-3333-3333-3333-333333333333",
    "principalName": "",
    "roleDefinitionId": "/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Authorization/roleDefinitions/ba92f5b4-2d11-453d-a403-e96b0029c9fe",
    "roleDefinitionName": "Storage Blob Data Contributor",
    "scope": "/subscriptions/11111111-1111-1111-1111-111111111111",
    "type": "Microsoft.Authorization/roleAssignments"
}
```

No es problema dejar estas asignaciones de roles donde se ha eliminado la entidad de seguridad. Si lo desea, puede quitar estas asignaciones de roles siguiendo los pasos similares a otras asignaciones de roles. Para obtener información sobre cómo quitar las asignaciones de roles, vea [Azure Portal](role-assignments-portal.md#remove-a-role-assignment), [Azure PowerShell](role-assignments-powershell.md#remove-a-role-assignment) o la [CLI de Azure](role-assignments-cli.md#remove-a-role-assignment).

En PowerShell, si intenta quitar las asignaciones de roles mediante el identificador de objeto y el nombre de la definición de roles, y más de una asignación de roles coincide con los parámetros, obtendrá el mensaje de error: "The provided information does not map to a role assignment" (La información proporcionada no se asigna a una asignación de roles). La salida siguiente muestra un ejemplo del mensaje de error:

```
PS C:\> Remove-AzRoleAssignment -ObjectId 33333333-3333-3333-3333-333333333333 -RoleDefinitionName "Storage Blob Data Contributor"

Remove-AzRoleAssignment : The provided information does not map to a role assignment.
At line:1 char:1
+ Remove-AzRoleAssignment -ObjectId 33333333-3333-3333-3333-333333333333 ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : CloseError: (:) [Remove-AzRoleAssignment], KeyNotFoundException
+ FullyQualifiedErrorId : Microsoft.Azure.Commands.Resources.RemoveAzureRoleAssignmentCommand
```

Si recibe este mensaje de error, asegúrese de especificar también los parámetros `-Scope` o `-ResourceGroupName`.

```
PS C:\> Remove-AzRoleAssignment -ObjectId 33333333-3333-3333-3333-333333333333 -RoleDefinitionName "Storage Blob Data Contributor" - Scope /subscriptions/11111111-1111-1111-1111-111111111111
```

## <a name="role-assignment-changes-are-not-being-detected"></a>No se detectan los cambios en la asignación de roles

Azure Resource Manager a veces almacena en caché las configuraciones y los datos para mejorar el rendimiento. Al agregar o quitar asignaciones de roles, los cambios pueden tardar hasta 30 minutos en aplicarse. Si usa Azure Portal, Azure PowerShell o la CLI de Azure, puede exigir una actualización de los cambios de asignación de roles cerrando e iniciando sesión. Si va a realizar cambios de asignación de roles con llamadas API de REST, puede exigir una actualización renovando el token de acceso.

Si agrega o quita una asignación de roles en el ámbito del grupo de administración y el rol tiene `DataActions`, es posible que el acceso en el plano de datos no se actualice hasta pasadas varias horas. Esto solo se aplica al ámbito del grupo de administración y al plano de datos.

## <a name="web-app-features-that-require-write-access"></a>Características de aplicaciones web que requieren acceso de escritura

Si concede a un usuario acceso de solo lectura a una única aplicación web, se deshabilitan algunas características que no cabría esperar. Las siguientes funcionalidades de administración requieren acceso de **escritura** a una aplicación web (como colaborador o como propietario) y no están disponibles en un escenario de solo lectura.

* Comandos (p. ej., iniciar, parar, etc.)
* Cambiar opciones, como la configuración general, opciones de escala, opciones de copia de seguridad y opciones de supervisión
* Acceder a las credenciales de publicación y otros secretos, como opciones de aplicaciones y cadenas de conexión
* Registros de streaming
* Configuración de registros de recursos
* Consola (símbolo del sistema)
* Implementaciones activas y recientes (para implementaciones git continuas locales)
* Gasto estimado
* Pruebas web
* Red virtual (solo visible para un lector si un usuario con acceso de escritura ha configurado previamente una red virtual)

Si no puede acceder a ninguno de estos iconos, debe pedirle al administrador el acceso de colaborador a la aplicación web.

## <a name="web-app-resources-that-require-write-access"></a>Recursos de aplicaciones web que requieren acceso de escritura

Las aplicaciones web pueden resultar complicadas si entran en juego distintos recursos. Este es un grupo de recursos típico con un par de sitios web:

![Grupo de recursos de aplicación web](./media/troubleshooting/website-resource-model.png)

Como consecuencia, si le concede a alguien acceso solo a la aplicación web, muchas de las funcionalidades de la hoja del sitio web de Azure Portal están deshabilitadas.

Estos elementos requieren acceso de **escritura** al **plan de App Service** que corresponde a su sitio web:  

* Visualización del plan de tarifa de la aplicación web (gratis o estándar).  
* Configuración de escala (número de instancias, tamaño de la máquina virtual y configuración de escala automática).  
* Cuotas (almacenamiento, ancho de banda y CPU).  

Estos elementos requieren acceso de **escritura** a todo el **grupo de recursos** que contiene su sitio web:  

* Enlaces y certificados TLS/SSL (los certificados TLS/SSL se pueden compartir entre sitios en el mismo grupo de recursos y la misma ubicación geográfica)  
* Las reglas de alertas  
* Opciones de escala automática  
* Componentes de Application Insights  
* Pruebas web  

## <a name="virtual-machine-features-that-require-write-access"></a>Características de máquina virtual que requieren acceso de escritura

De manera similar a las aplicaciones web, algunas funciones de la hoja de máquina virtual requieren acceso de escritura a la máquina virtual o a otros recursos del grupo de recursos.

Las máquinas virtuales están relacionadas con los nombres de dominio, las redes virtuales, las cuentas de almacenamiento y las reglas de alerta.

Estos elementos requieren acceso de **escritura** a la **máquina virtual**:

* Puntos de conexión  
* Direcciones IP  
* Discos  
* Extensiones  

Estos requieren acceso de **escritura** a la **máquina virtual** y al **grupo de recursos** (junto con el nombre de dominio) donde se encuentran:  

* Conjunto de disponibilidad  
* El conjunto de carga equilibrada  
* Las reglas de alertas  

Si no puede acceder a ninguno de estos iconos, debe pedirle al administrador el acceso de colaborador al grupo de recursos.

## <a name="azure-functions-and-write-access"></a>Azure Functions y acceso de escritura

Algunas características de [Azure Functions](../azure-functions/functions-overview.md) requieren acceso de escritura. Por ejemplo, si se asigna a un usuario el rol de [lector](built-in-roles.md#reader), este no podrá ver las funciones dentro de una aplicación de función. El portal mostrará **(Sin acceso)** .

![Sin acceso a aplicaciones de función](./media/troubleshooting/functionapps-noaccess.png)

Un lector puede hacer clic en la pestaña **Características de la plataforma** y, a continuación, hacer clic en **Toda las opciones de configuración** para ver algunas opciones de configuración relacionadas con una aplicación de función (similar a una aplicación web), pero no puede modificar ninguna de estas opciones de configuración. Para tener acceso a estas características, necesitará el rol de [colaborador](built-in-roles.md#contributor).

## <a name="next-steps"></a>Pasos siguientes

- [Solución de problemas de usuarios invitados](role-assignments-external-users.md#troubleshoot)
- [Incorporación o eliminación de asignaciones de roles de Azure con Azure Portal](role-assignments-portal.md)
- [Visualización de los registros de actividad de los cambios de RBAC de Azure](change-history-report.md)
