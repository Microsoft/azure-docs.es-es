---
title: Integración de Azure Active Directory para Red Hat OpenShift en Azure
description: Obtenga información sobre cómo crear un grupo de seguridad y un usuario de Azure AD para probar aplicaciones en el clúster de Red Hat OpenShift en Microsoft Azure.
author: jimzim
ms.author: jzim
ms.service: container-service
ms.topic: conceptual
ms.date: 05/13/2019
ms.openlocfilehash: 8d3326587d9c03e0a3960016b8ba3668c825ee6d
ms.sourcegitcommit: 4feb198becb7a6ff9e6b42be9185e07539022f17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89470092"
---
# <a name="azure-active-directory-integration-for-azure-red-hat-openshift"></a>Integración de Azure Active Directory para Red Hat OpenShift en Azure

Si aún no ha creado un inquilino de Azure Active Directory (Azure AD), siga las indicaciones sobre la [creación de un inquilino de Azure AD para Red Hat OpenShift en Azure](howto-create-tenant.md) antes de continuar con estas instrucciones.

Red Hat OpenShift en Microsoft Azure necesita permisos para realizar tareas en nombre del clúster. Si su organización aún no dispone de un usuario, un grupo de seguridad o un registro de aplicación de Azure AD para usarlos como entidad de servicio, siga estas instrucciones para crearlos.

## <a name="create-a-new-azure-active-directory-user"></a>Creación de un usuario de Azure Active Directory nuevo

En [Azure Portal](https://portal.azure.com), asegúrese de que el inquilino aparece bajo el nombre de usuario en la parte superior derecha del portal:

![Captura de pantalla del portal en el que aparece el inquilino en la parte superior derecha](./media/howto-create-tenant/tenant-callout.png). Si se muestra el inquilino incorrecto, haga clic en el nombre de usuario en la parte superior derecha, luego haga clic en **Cambiar directorio** y seleccione el inquilino correcto de la lista **Todos los directorios**.

Cree un nuevo usuario "Propietario" de Azure Active Directory para iniciar sesión en el clúster de Red Hat OpenShift en Azure.

1. Vaya a la hoja [Usuarios: todos los usuarios](https://portal.azure.com/#blade/Microsoft_AAD_IAM/UsersManagementMenuBlade/AllUsers).
2. Haga clic en **+Nuevo usuario** para abrir el panel **Usuario**.
3. Escriba un **Nombre** para este usuario.
4. Cree un **Nombre de usuario** basado en el nombre del inquilino que haya creado, con `.onmicrosoft.com` anexado al final. Por ejemplo, `yourUserName@yourTenantName.onmicrosoft.com`. Anote este nombre de usuario. Lo necesitará para conectarse en el clúster.
5. Haga clic en **Rol de directorio** para abrir el panel de rol de directorio, seleccione **Propietario** y luego haga clic en **Aceptar** en la parte inferior del panel.
6. En el panel **Usuario**, haga clic en **Mostrar contraseña** y anote la contraseña temporal. Después de iniciar sesión por primera vez, se le pedirá que la restablezca.
7. Haga clic en **Crear** en la parte inferior del panel para crear el usuario.

## <a name="create-an-azure-ad-security-group"></a>Creación de un grupo de seguridad de Azure AD

Para conceder acceso de administrador de clúster, las pertenencias de un grupo de seguridad de Azure AD se sincronizarán con el grupo de OpenShift "osa-customer-admins". Si no se especifica, no se concederá ningún acceso de administrador de clúster.

1. Abra la hoja [Grupos de Azure Active Directory](https://portal.azure.com/#blade/Microsoft_AAD_IAM/GroupsManagementMenuBlade/AllGroups).
2. Haga clic en **+Nuevo grupo**.
3. Proporcione un nombre de grupo y una descripción.
4. Establezca el **Tipo de grupo** en **Seguridad**.
5. Establezca **Tipo de pertenencia** en **Asignado**.

    Agregue el usuario de Azure AD que ha creado en el paso anterior a este grupo de seguridad.

6. Haga clic en **Miembros** para abrir el panel **Seleccionar miembros**.
7. En la lista de miembros, seleccione el usuario de Azure AD que ha creado anteriormente.
8. Para crear el grupo de seguridad, en la parte inferior del portal haga clic en **Seleccionar** y luego en **Crear**.

    Anote el valor de id. de grupo.

9. Cuando se haya creado el grupo, lo verá en la lista de todos los grupos. Haga clic en el nuevo grupo.
10. En la página que aparece, copie el **Id. de objeto**. En el tutorial [Creación de un clúster de Red Hat OpenShift en Azure](tutorial-create-cluster.md), se hará referencia a este valor como `GROUPID`.

> [!IMPORTANT]
> Para sincronizar este grupo con el grupo de OpenShift osa-customer-admins, cree el clúster mediante la CLI de Azure. Actualmente, Azure Portal no tiene ningún campo para establecer este grupo.

## <a name="create-an-azure-ad-app-registration"></a>Creación de un registro de aplicación de Azure AD

Puede crear automáticamente un cliente de registro de aplicación de Azure Active Directory (Azure AD) como parte de la creación del clúster al omitir la marca `--aad-client-app-id` en el comando `az openshift create`. En este tutorial se muestra cómo crear el registro de aplicación de Azure AD para tener una visión completa.

Si su organización aún no dispone de un registro de aplicación de Azure AD para usarlo como entidad de servicio, siga estas instrucciones para uno.

1. Abra la [hoja de Registros de aplicaciones](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredAppsPreview) y haga clic en **+Nuevo registro**.
2. En el panel **Registrar una aplicación**, escriba el nombre del registro de aplicación.
3. Asegúrese de que en **Tipos de cuenta admitidos** se selecciona **Solo cuentas en este directorio organizativo**. Esta es la opción más segura.
4. Una vez que sepamos el URI del clúster, se agregará un URI de redireccionamiento. Haga clic en el botón **Registro** para crear el registro de aplicación de Azure AD.
5. En la página que aparece, copie el **Id. de aplicación (cliente)** . En el tutorial [Creación de un clúster de Red Hat OpenShift en Azure](tutorial-create-cluster.md), se hará referencia a este valor como `APPID`.

![Captura de pantalla de la página de objeto de aplicación](./media/howto-create-tenant/get-app-id.png)

### <a name="create-a-client-secret"></a>Creación de un secreto de cliente

Genere un secreto de cliente para autenticar la aplicación en Azure Active Directory.

1. En la sección **Administrar** de la página de registros de aplicación, haga clic en **Certificados y secretos**.
2. En el panel **Certificados y secretos**, haga clic en **+Nuevo secreto de cliente**.  Aparece el panel **Agregar un secreto de cliente**.
3. Proporcione una **Descripción**.
4. Establezca **Expira** en la duración que prefiera, por ejemplo **en 2 años**.
5. Haga clic en **Agregar** y aparecerá el valor de clave en la sección **Secretos de cliente** de la página.
6. Copie el valor de la clave. En el tutorial [Creación de un clúster de Red Hat OpenShift en Azure](tutorial-create-cluster.md), se hará referencia a este valor como `SECRET`.

![Captura de pantalla del panel de certificados y secretos](./media/howto-create-tenant/create-key.png)

Para obtener más información acerca de objetos de aplicación de Azure, consulte [Objetos de aplicación y de entidad de servicio de Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md).

Para obtener información sobre cómo crear una nueva aplicación de Azure AD, consulte [Registro de una aplicación con el punto de conexión de Azure Active Directory v1.0](../active-directory/develop/quickstart-register-app.md).

## <a name="add-api-permissions"></a>Adición de permisos de API

[//]: # (No cambie a Microsoft Graph. No funciona con Microsoft Graph.)
1. En la sección **Administrar**, haga clic en **Permisos de API**.
2. Haga clic en **Agregar permiso**, seleccione **Azure Active Directory Graph** y luego **Permisos delegados**.
> [!NOTE]
> Asegúrese de que ha seleccionado "Graph de Azure Active Directory" y no el icono de "Microsoft Graph".

3. Expanda la opción **Usuario** en la lista siguiente y habilite el permiso **User.Read**. Si **User.Read** está habilitado de forma predeterminada, asegúrese de que es el permiso **User.Read** de **Azure Active Directory Graph**.
4. Desplácese hacia arriba y seleccione **Permisos de aplicación**.
5. Expanda la opción **Directorio** en la lista siguiente y habilite **Directory.ReadAll**
6. Para aceptar los cambios, haga clic en **Agregar permisos**.
7. El panel de permisos de API debe mostrar ahora *User.Read* y *Directory.ReadAll*. Tenga en cuenta la advertencia que aparece en la columna **Consentimiento del administrador requerido** junto a *Directory.ReadAll*.
8. Si es el *administrador de suscripciones de Azure*, haga clic en **Conceder consentimiento del administrador para *nombre de la suscripción*** debajo. Si no es el *administrador de suscripciones de Azure*, solicítelo al administrador.

![Captura de pantalla del panel de permisos de API. Permisos User.Read y Directory.ReadAll agregados, consentimiento del administrador requerido para Directory.ReadAll](./media/howto-aad-app-configuration/permissions-required.png)

> [!IMPORTANT]
> La sincronización del grupo de administradores de clúster funcionará solo después de que se conceda el consentimiento. Verá un círculo verde con una marca de verificación y el mensaje "Concedido para *nombre de la suscripción*" en la columna *Consentimiento del administrador requerido*.

Para obtener más información acerca de cómo gestionar los administradores y otros roles, consulte [Agregar o cambiar los administradores de la suscripción de Azure](../cost-management-billing/manage/add-change-subscription-administrator.md).

## <a name="resources"></a>Recursos

* [Objetos de aplicaciones y de entidad de servicio de Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md)
* [Inicio rápido: Registro de una aplicación con el punto de conexión de Azure Active Directory v1.0](../active-directory/develop/quickstart-register-app.md)

## <a name="next-steps"></a>Pasos siguientes

Si se han cumplido todos los [requisitos previos de Red Hat OpenShift en Azure](howto-setup-environment.md), ya está listo para crear su primer clúster.

Pruebe el tutorial:
> [!div class="nextstepaction"]
> [Creación de un clúster de Red Hat OpenShift en Azure](tutorial-create-cluster.md)