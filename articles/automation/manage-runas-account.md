---
title: Administración de una cuenta de ejecución de Azure Automation
description: En este artículo se describe cómo administrar la cuenta de ejecución con PowerShell o desde Azure Portal.
services: automation
ms.subservice: shared-capabilities
ms.date: 06/26/2020
ms.topic: conceptual
ms.openlocfilehash: 3e5f75a5ff9c6baff9bbefea7846ffe78655c6a9
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401765"
---
# <a name="manage-an-azure-automation-run-as-account"></a>Administración de una cuenta de ejecución de Azure Automation

Las cuentas de ejecución de Azure Automation proporcionan autenticación para administrar los recursos de Azure con los cmdlets de Azure. Cuando crea una cuenta de ejecución, se crea un nuevo usuario de la entidad de servicio en Azure Active Directory (AD) y se asigna el rol Colaborador a este usuario en el nivel de suscripción.

## <a name="types-of-run-as-accounts"></a>Tipos de cuentas de ejecución

Azure Automation usa dos tipos de cuentas de ejecución:

* Cuenta de ejecución de Azure
* Cuenta de ejecución de Azure clásico

>[!NOTE]
>Las suscripciones del Proveedor de soluciones en la nube (CSP) de Azure solo admiten el modelo de Azure Resource Manager. Los servicios que no son de Azure Resource Manager no están disponibles en el programa. Cuando se usa una suscripción al programa CSP, no se crea la cuenta de ejecución de Azure clásico, sino la cuenta de ejecución de Azure. Para más información acerca de las suscripciones de CSP, consulte [Servicios disponibles en las suscripciones de CSP](/azure/cloud-solution-provider/overview/azure-csp-available-services).

La entidad de servicio para una cuenta de ejecución no tiene los permisos para leer Azure Active Directory de manera predeterminada. Si quiere agregar permisos para leer o administrar Azure AD, deberá conceder los permisos a la entidad de servicio en **Permisos de API**. Para más información, consulte [Adición de permisos para acceder a las API web](../active-directory/develop/quickstart-configure-app-access-web-apis.md#add-permissions-to-access-web-apis).

### <a name="run-as-account"></a>cuenta de identificación

La cuenta de ejecución administra los recursos del [modelo de implementación de Resource Manager](../azure-resource-manager/management/deployment-models.md). Realiza las tareas que se indican a continuación.

* Crea una aplicación de Azure AD con un certificado autofirmado, crea una cuenta de entidad servicio para la aplicación en Azure AD y asigna el rol Colaborador para esta cuenta en la suscripción actual. Puede cambiar la configuración de certificado a Propietario o a cualquier otro rol. Para más información, consulte [Control de acceso basado en rol en Azure Automation](automation-role-based-access-control.md).
  
* Crea un recurso de certificado de Automation denominado `AzureRunAsCertificate` en la cuenta de Automation especificada. El recurso de certificado contiene la clave privada del certificado que se usa en la aplicación de Azure AD.
  
* Crea un recurso de conexión de Automation denominado `AzureRunAsConnection` en la cuenta de Automation especificada. El recurso de conexión contiene el identificador de la aplicación, el identificador del inquilino, el identificador de la suscripción y la huella digital del certificado.

### <a name="azure-classic-run-as-account"></a>Cuenta de ejecución de Azure clásico

La cuenta de ejecución de Azure clásico administra recursos del [modelo de implementación clásico](../azure-resource-manager/management/deployment-models.md). Debe ser coadministrador de la suscripción para crear o renovar este tipo de cuenta.

La cuenta de ejecución de Azure clásico realiza las siguientes tareas.

  * Crea un certificado de administración en la suscripción.

  * Crea un recurso de certificado de Automation denominado `AzureClassicRunAsCertificate` en la cuenta de Automation especificada. El recurso de certificado contiene la clave privada del certificado que usa el certificado de administración.

  * Crea un recurso de conexión de Automation denominado `AzureClassicRunAsConnection` en la cuenta de Automation especificada. El recurso de conexión contiene el nombre de la suscripción, el id. de suscripción y el nombre del recurso de certificado.

>[!NOTE]
>Las cuentas de ejecución de Azure clásico ya no se crean de forma predeterminada a la vez que se crea una cuenta de Automation. Esta cuenta se crea individualmente siguiendo los pasos que se describen más adelante en este artículo.

## <a name="obtain-run-as-account-permissions"></a><a name="permissions"></a>Obtención de permisos de las cuentas de ejecución

En esta sección se definen los permisos para las cuentas de ejecución normales y las cuentas de ejecución clásicas.

### <a name="get-permissions-to-configure-run-as-accounts"></a>Obtención de permisos para configurar cuentas de ejecución

Para crear o actualizar una cuenta de ejecución, debe tener los permisos y privilegios específicos. Un administrador de aplicaciones en Azure Active Directory y un propietario en una suscripción pueden completar todas las tareas. En una situación en la que tenga separación de funciones, la siguiente tabla muestra una lista de las tareas, el cmdlet equivalente y los permisos necesarios:

|Tarea|Cmdlet  |Permisos mínimos  |Donde se establecen los permisos|
|---|---------|---------|---|
|Crear una aplicación de Azure AD|[New-AzADApplication](/powershell/module/az.resources/new-azadapplication)     | Rol de desarrollador de aplicaciones<sup>1</sup>        |[Azure AD](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app)</br>Inicio > Azure AD > Registros de aplicaciones |
|Agregar una credencial a la aplicación.|[New-AzADAppCredential](/powershell/module/az.resources/new-azadappcredential)     | Administrador de la aplicación o administrador global<sup>1</sup>         |[Azure AD](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app)</br>Inicio > Azure AD > Registros de aplicaciones|
|Crear y obtener una entidad de servicio de Azure AD|[New-AzADServicePrincipal](/powershell/module/az.resources/new-azadserviceprincipal)</br>[Get-AzADServicePrincipal](/powershell/module/az.resources/get-azadserviceprincipal)     | Administrador de la aplicación o administrador global<sup>1</sup>        |[Azure AD](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app)</br>Inicio > Azure AD > Registros de aplicaciones|
|Asignar u obtener el rol de Azure para la entidad de seguridad especificada|[New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment)</br>[Get-AzRoleAssignment](/powershell/module/Az.Resources/Get-AzRoleAssignment)      | Administrador de acceso de usuario o Propietario, o bien tener los permisos siguientes:</br></br><code>Microsoft.Authorization/Operations/read</br>Microsoft.Authorization/permissions/read</br>Microsoft.Authorization/roleDefinitions/read</br>Microsoft.Authorization/roleAssignments/write</br>Microsoft.Authorization/roleAssignments/read</br>Microsoft.Authorization/roleAssignments/delete</code></br></br> | [Suscripción](../role-based-access-control/role-assignments-portal.md)</br>Inicio > Suscripciones > \<subscription name\> Access Control (IAM)|
|Crear o quitar un certificado de Automation|[New-AzAutomationCertificate](/powershell/module/Az.Automation/New-AzAutomationCertificate)</br>[Remove-AzAutomationCertificate](/powershell/module/az.automation/remove-azautomationcertificate)     | Colaborador en el grupo de recursos         |Grupo de recursos de la cuenta de Automation|
|Crear o quitar una conexión de Automation|[New-AzAutomationConnection](/powershell/module/az.automation/new-azautomationconnection)</br>[Remove-AzAutomationConnection](/powershell/module/az.automation/remove-azautomationconnection)|Colaborador en el grupo de recursos |Grupo de recursos de la cuenta de Automation|

<sup>1</sup> Los usuarios que no son administradores en el inquilino de Azure AD pueden [registrar aplicaciones de AD](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app) si la opción **Los usuarios pueden registrar aplicaciones** del inquilino de Azure AD en la página Configuración de usuario está establecida en **Sí**. Si el registro de la aplicación está configurado en **No**, el usuario que realiza esta acción debe ser como se define en esta tabla.

Si no es miembro de la instancia de Active Directory de la suscripción antes de que se le agregue al rol Administrador global de la suscripción, se le agregará como invitado. En este caso, recibirá una advertencia `You do not have permissions to create…` en la página **Agregar cuenta de Automation**.

Si es miembro de la instancia de Active Directory de la suscripción cuando se asigna el rol Administrador global, también puede recibir una advertencia `You do not have permissions to create…` en la página **Agregar cuenta de Automation**. En este caso, puede solicitar la eliminación de la instancia de Active Directory de la suscripción y, a continuación, solicitar que se vuelva a agregar, a fin de convertirse en usuario completo de Active Directory.

Para comprobar que se ha corregido la situación causante del mensaje de error:

1. En el panel de Azure Active Directory de Azure Portal, seleccione **Usuarios y grupos**.
2. Seleccione **Todos los usuarios**.
3. Elija su nombre y, a continuación, seleccione **Perfil**. 
4. Asegúrese de que el valor del atributo **Tipo de usuario**  en el perfil de usuario no esté establecido en **Invitado**.

### <a name="get-permissions-to-configure-classic-run-as-accounts"></a><a name="permissions-classic"></a>Obtención de permisos para configurar cuentas de ejecución clásicas

Para configurar o renovar las cuentas de ejecución clásicas, debe tener el rol Coadministrador en el nivel de suscripción. Para más información sobre los permisos de suscripción clásicos, consulte [Administradores de la suscripción clásica de Azure](../role-based-access-control/classic-administrators.md#add-a-co-administrator).

## <a name="create-a-run-as-account-in-azure-portal"></a>Creación de una cuenta de ejecución en Azure Portal

Realice los pasos que se describen a continuación para actualizar su cuenta de Azure Automation en Azure Portal. Cree las cuentas de ejecución y de ejecución clásica de forma individual. Si no es necesario administrar los recursos clásicos, basta con crear la cuenta de ejecución de Azure.

1. Inicie sesión en el Portal de Azure con una cuenta que sea miembro del rol Administradores de suscripciones y coadministrador de la suscripción.

2. Busque y seleccione **Cuentas de Automation**.

3. En la página Cuentas de Automation, seleccione su cuenta de Automation en la lista.

4. En el panel izquierdo, seleccione **Cuentas de ejecución** en la sección Configuración de la cuenta.

5. En función de la cuenta que necesite, seleccione **Cuenta de ejecución de Azure** o **Cuenta de ejecución de Azure clásica**. 

6. En función de la cuenta que le interese, use el panel **Agregar cuenta de ejecución de Azure** o **Agregar cuenta de ejecución de Azure clásico**. Después de revisar la información general, haga clic en **Crear**.

7. Mientras Azure crea la cuenta de ejecución, se puede seguir el progreso en **Notificaciones** en el menú. También se muestra un banner que indica que se está creando la cuenta. El proceso puede tardar unos minutos en completarse.

## <a name="delete-a-run-as-or-classic-run-as-account"></a>Eliminación de una cuenta de ejecución o de ejecución clásica

En esta sección se describe cómo eliminar una cuenta de ejecución o una cuenta de ejecución clásica. Al realizar esta acción, la cuenta de Automation se conserva. Después de eliminar la cuenta, puede volver a crearla en Azure Portal.

1. Abra la cuenta de Automation en Azure Portal.

2. En el panel izquierdo, seleccione **Cuentas de ejecución** en la sección Configuración de la cuenta.

3. En la página de propiedades Cuentas de ejecución, seleccione la cuenta de ejecución o la cuenta de ejecución clásica que quiere eliminar. 

4. En el panel Propiedades de la cuenta seleccionada, haga clic en **Eliminar**.

   ![Eliminación de una cuenta de ejecución](media/manage-runas-account/automation-account-delete-runas.png)

5. Mientras se está eliminando la cuenta, puede seguir el progreso desde el menú, en **Notificaciones**.

6. Después de eliminar la cuenta, puede volver a crearla en la página de propiedades de Cuentas de ejecución. Para ello, seleccione la opción de creación **Cuenta de ejecución de Azure**.

   ![Nueva creación de la cuenta de ejecución de Automation](media/manage-runas-account/automation-account-create-runas.png)

## <a name="renew-a-self-signed-certificate"></a><a name="cert-renewal"></a>Renovación de un certificado autofirmado

El certificado autofirmado que creó para la cuenta de ejecución expira un año después de la fecha de creación. En algún momento antes de que expire su cuenta de ejecución, debe renovar el certificado. Se puede renovar en cualquier momento antes de que expire. 

Cuando se renueva el certificado autofirmado, el certificado válido actual se conserva para garantizar que los runbooks que se ponen en la cola o que se ejecutan activamente, y que se autentican con la cuenta de ejecución, no se ven negativamente afectados. El certificado es válido hasta la fecha de expiración.

>[!NOTE]
>Si cree que se ha puesto en peligro la cuenta de ejecución, puede eliminarla y volver a crearla.

>[!NOTE]
>Si ha configurado la cuenta de ejecución para usar un certificado emitido por la entidad de certificación de la empresa y usa esta opción para renovar un certificado autofirmado, el certificado de empresa se reemplaza por uno autofirmado.

Realice los pasos siguientes para renovar el certificado autofirmado.

1. Abra la cuenta de Automation en Azure Portal.

1. Seleccione **Cuentas de ejecución** en la sección Configuración de la cuenta.

    ![Panel de propiedades de la cuenta de Automation](media/manage-runas-account/automation-account-properties-pane.png)

1. En la página de propiedades de Cuentas de ejecución, seleccione la cuenta de ejecución o la cuenta de ejecución clásica para la que quiere renovar el certificado.

1. En el panel Propiedades de la cuenta seleccionada, haga clic en **Renovar certificado**.

    ![Renovación del certificado para una cuenta de ejecución](media/manage-runas-account/automation-account-renew-runas-certificate.png)

1. Mientras se está renovando el certificado, puede seguir el progreso desde el menú, en **Notificaciones**.

## <a name="limit-run-as-account-permissions"></a>Limitación de permisos de cuentas de ejecución

Para controlar la orientación de Automation hacia los recursos de Azure, puede ejecutar el script [Update-AutomationRunAsAccountRoleAssignments.ps1](https://aka.ms/AA5hug8). Este script cambia la entidad de servicio de la cuenta de ejecución existente para crear y usar una definición de rol personalizada. Este rol tiene permisos para todos los recursos excepto para [Key Vault](../key-vault/index.yml).

>[!IMPORTANT]
>Después de ejecutar el script **Update-AutomationRunAsAccountRoleAssignments.ps1**, los runbooks que acceden a Key Vault mediante cuentas de ejecución ya no funcionan. Antes de ejecutar el script, debe revisar los runbooks de su cuenta para las llamadas a Azure Key Vault. Para permitir el acceso a Key Vault desde los runbooks de Azure Automation, debe [agregar la cuenta de ejecución a los permisos de Key Vault](#add-permissions-to-key-vault).

Si necesita restringir aún más lo que la entidad de servicio de ejecución puede hacer, puede agregar otros tipos de recursos al elemento `NotActions` de la definición de roles personalizada. El siguiente ejemplo restringe el acceso a `Microsoft.Compute/*`. Si agrega este tipo definición de recurso a `NotActions` para la definición de roles, el rol no podrá acceder a ningún recurso de proceso. Para más información sobre las definiciones de roles, consulte [Descripción de definiciones de roles para los recursos de Azure](../role-based-access-control/role-definitions.md).

```powershell
$roleDefinition = Get-AzRoleDefinition -Name 'Automation RunAs Contributor'
$roleDefinition.NotActions.Add("Microsoft.Compute/*")
$roleDefinition | Set-AzRoleDefinition
```

Puede determinar si la entidad de servicio que se usa en la cuenta de ejecución está en la definición del rol Colaborador o en una personalizada. 

1. Vaya a la cuenta de Automation y seleccione **Cuentas de ejecución** en la sección Configuración de la cuenta.
2. Seleccione **Cuenta de ejecución de Azure**. 
3. Seleccione **Rol** para localizar la definición de roles que se está usando.

:::image type="content" source="media/manage-runas-account/verify-role.png" alt-text="Compruebe el rol de cuenta de ejecución" lightbox="media/manage-runas-account/verify-role-expanded.png":::.


También puede determinar la definición de roles que usan las cuentas de ejecución para varias suscripciones o cuentas de Automation. Para ello, use el script [Check-AutomationRunAsAccountRoleAssignments.ps1](https://aka.ms/AA5hug5) en la Galería de PowerShell.

### <a name="add-permissions-to-key-vault"></a>Incorporación de permisos a Key Vault

Puede permitir que Azure Automation compruebe si Key Vault y la entidad de servicio de la cuenta de ejecución usan una definición de roles personalizada. Debe:

* Conceder permisos a Key Vault.
* Establecer la directiva de acceso.

Puede usar el script [Extend-AutomationRunAsAccountRoleAssignmentToKeyVault.ps1](https://aka.ms/AA5hugb) en el Galería de PowerShell para asignar sus permisos de la cuenta de ejecución a KeyVault. Consulte [Asignación de una directiva de acceso de Key Vault](/azure/key-vault/general/assign-access-policy-powershell) para obtener más detalles sobre cómo establecer permisos en Key Vault.

## <a name="resolve-misconfiguration-issues-for-run-as-accounts"></a>Solución de problemas de configuración incorrecta de las cuentas de ejecución

Puede que algunos elementos de configuración necesarios para la cuenta de ejecución o la cuenta de ejecución clásica se hayan eliminado o no se hayan creado correctamente durante la configuración inicial. Algunos errores de configuración posibles son:

* Recurso de certificado
* Recurso de conexión
* Cuenta de ejecución quitada del rol Colaborador
* Entidad de servicio o aplicación en Azure AD

En estos casos de errores de configuración, la cuenta de Automation detecta los cambios y muestra el estado *Incompleto* en el panel de propiedades de Cuentas de ejecución de la cuenta.

![Estado de configuración incompleta de la cuenta de ejecución](media/manage-runas-account/automation-account-runas-incomplete-config.png)

Al seleccionar la cuenta de ejecución, el panel de propiedades de la cuenta muestra el mensaje de error siguiente:

```text
The Run As account is incomplete. Either one of these was deleted or not created - Azure Active Directory Application, Service Principal, Role, Automation Certificate asset, Automation Connect asset - or the Thumbprint is not identical between Certificate and Connection. Please delete and then re-create the Run As Account.
```

Rápidamente puede resolver estos problemas de la cuenta de ejecución con solo eliminarla cuenta y volver a crearla.

## <a name="next-steps"></a>Pasos siguientes

* [Objetos de aplicación y de entidad de servicio](../active-directory/develop/app-objects-and-service-principals.md).
* [Introducción a los certificados para Azure Cloud Services](../cloud-services/cloud-services-certs-create.md).
