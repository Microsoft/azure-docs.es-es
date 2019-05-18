---
title: Integración de Azure Active Directory para Azure Red Hat OpenShift | Microsoft Docs
description: Obtenga información sobre cómo crear un grupo de seguridad de Azure AD y el usuario para probar aplicaciones en un clúster de Microsoft Azure Red Hat OpenShift.
author: tylermsft
ms.author: twhitney
ms.service: openshift
manager: jeconnoc
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 05/13/2019
ms.openlocfilehash: f6b87748c33c1afd047ae25dfb7df3670a73e7c8
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65779677"
---
# <a name="azure-active-directory-integration-for-azure-red-hat-openshift"></a>Integración de Azure Active Directory para Azure Red Hat OpenShift

Si aún no ha creado un inquilino de Azure Active Directory (Azure AD), siga las instrucciones de [crear un inquilino de Azure AD para Azure Red Hat OpenShift](howto-create-tenant.md) antes de continuar con estas instrucciones.

Microsoft Azure Red Hat OpenShift necesita permisos para realizar tareas en nombre del clúster. Si su organización no dispone de un usuario de Azure AD, grupo de seguridad de Azure AD o un registro de aplicación de Azure AD para usarla como la entidad de servicio, siga estas instrucciones para crearlos.

## <a name="create-a-new-azure-active-directory-user"></a>Creación de un usuario de Azure Active Directory nuevo

En el [portal Azure](https://portal.azure.com), asegúrese de que el inquilino aparece bajo el nombre de usuario en la parte superior derecha del portal:

![Captura de pantalla del portal de inquilino aparece en la esquina superior derecha](./media/howto-create-tenant/tenant-callout.png) si se muestra el inquilino incorrecto, haga clic en el nombre de usuario en la esquina superior derecha, a continuación, haga clic en **Cambiar directorio**y seleccione el inquilino correcto de la **todas Directorios** lista.

Crear un nuevo usuario de administrador global de Azure Active Directory para iniciar sesión en el clúster de Azure Red Hat OpenShift.

1. Vaya a la [los usuarios de todos los usuarios](https://portal.azure.com/#blade/Microsoft_AAD_IAM/UsersManagementMenuBlade/AllUsers) hoja.
2. Haga clic en **+ nuevo usuario** para abrir el **usuario** panel.
3. Escriba un **nombre** para este usuario.
4. Crear un **nombre de usuario** según el nombre del inquilino ha creado, con `.onmicrosoft.com` anexado al final. Por ejemplo, `yourUserName@yourTenantName.onmicrosoft.com`. Anote este nombre de usuario. La necesitará para iniciar sesión en el clúster.
5. Haga clic en **rol del directorio** para abrir el panel de la función de directorio y seleccione **administrador Global** y, a continuación, haga clic en **Aceptar** en la parte inferior del panel.
6. En el **usuario** panel, haga clic en **Mostrar contraseña** y anote la contraseña temporal. Después de iniciar sesión por primera vez, se le pedirá que la restablezca.
7. En la parte inferior del panel, haga clic en **crear** para crear el usuario.

## <a name="create-an-azure-ad-security-group"></a>Crear un grupo de seguridad de Azure AD

Para conceder acceso de administrador de clúster, las pertenencias en un grupo de seguridad de Azure AD se sincronizarán con el OpenShift grupo "osa-cliente-admins". Si no se especifica, no se concederán acceso de administrador de clúster.

1. Abra el [grupos de Azure Active Directory](https://portal.azure.com/#blade/Microsoft_AAD_IAM/GroupsManagementMenuBlade/AllGroups) hoja.
2. Haga clic en **+ grupo nuevo**
3. Proporcione un nombre de grupo y una descripción.
4. Establecer **tipo de grupo** a **seguridad**.
5. Establecer **tipo de pertenencia** a **asignado**.

    Agregue el usuario de Azure AD que creó en el paso anterior para este grupo de seguridad.

6. Haga clic en **miembros** para abrir el **seleccionar miembros** panel.
7. En la lista de miembros, seleccione el usuario de Azure AD que haya creado anteriormente.
8. En la parte inferior del portal, haga clic en **seleccione** y, a continuación, **crear** para crear el grupo de seguridad.

    Anote el valor de Id. de grupo

9. Cuando se crea el grupo, se verán en la lista de todos los grupos. Haga clic en el nuevo grupo.
10. En la página que aparece, copie el **Id. de objeto**. Nos referiremos a este valor como `GROUPID` en el [crear un clúster de Azure Red Hat OpenShift](tutorial-create-cluster.md) tutorial.

## <a name="create-an-azure-ad-app-registration"></a>Crear un registro de aplicación de Azure AD

Puede crear automáticamente un cliente de registro de aplicación de Azure Active Directory (Azure AD) como parte de la creación del clúster si se omite el `--aad-client-app-id` marca a la `az openshift create` comando. Este tutorial muestra cómo crear el registro de aplicación de Azure AD para proporcionar información completa.

Si su organización ya no tiene un registro de aplicación de Azure Active Directory (Azure AD) para usarla como una entidad de servicio, siga estas instrucciones para crear uno.

1. Abra el [hoja registros de aplicaciones](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredAppsPreview) y haga clic en **+ nuevo registro**.
2. En el **registrar una aplicación** panel, escriba un nombre para el registro de la aplicación.
3. Asegúrese de que en **admite tipos de cuenta** que **cuentas en este directorio organizativa solo** está seleccionada. Esta es la opción más segura.
4. Una vez que sepamos el URI del clúster, se agregará un URI de redireccionamiento más adelante. Haga clic en el **registrar** botón para crear el registro de aplicación de Azure AD.
5. En la página que aparece, copie el **Id. de aplicación (cliente)**. Nos referiremos a este valor como `APPID` en el [crear un clúster de Azure Red Hat OpenShift](tutorial-create-cluster.md) tutorial.

![Captura de pantalla de la página de objetos de aplicación](./media/howto-create-tenant/get-app-id.png)

### <a name="create-a-client-secret"></a>Creación de un secreto de cliente

Generar un secreto de cliente para autenticar la aplicación a Azure Active Directory.

1. En el **administrar** sección de la página de registros de aplicación, haga clic en **certificados y secretos**.
2. En el **certificados y secretos** panel, haga clic en **+ nuevo secreto de cliente**.  El **agregar un secreto de cliente** aparecerá el panel.
3. Proporcione un **descripción**.
4. Establecer **Expires** a la duración lo prefiere, por ejemplo **en 2 años**.
5. Haga clic en **agregar** y aparecerá el valor de clave en el **los secretos de cliente** sección de la página.
6. Copie el valor de clave. Nos referiremos a este valor como `SECRET` en el [crear un clúster de Azure Red Hat OpenShift](tutorial-create-cluster.md) tutorial.
 
![Captura de pantalla del panel de certificados y secretos](./media/howto-create-tenant/create-key.png)
 
Para obtener más información sobre los objetos de aplicación de Azure, consulte [Application y objetos de entidad de servicio en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/app-objects-and-service-principals).

Para obtener más información sobre cómo crear una nueva aplicación de Azure AD, consulte [registrar una aplicación con el punto de conexión de Azure Active Directory v1.0](https://docs.microsoft.com/azure/active-directory/develop/quickstart-v1-add-azure-ad-app).

## <a name="resources"></a>Recursos

* [Las aplicaciones y los objetos de entidad de servicio en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/app-objects-and-service-principals)  
* [Inicio rápido: Registrar una aplicación con el punto de conexión de Azure Active Directory v1.0](https://docs.microsoft.com/azure/active-directory/develop/quickstart-v1-add-azure-ad-app)  

## <a name="next-steps"></a>Pasos siguientes

Si se han cumplido todos los [requisitos previos de Azure Red Hat OpenShift](howto-setup-environment.md), está listo para crear su primer clúster!

Pruebe el tutorial:
> [!div class="nextstepaction"]
> [Creación de un clúster de Red Hat OpenShift en Azure](tutorial-create-cluster.md)