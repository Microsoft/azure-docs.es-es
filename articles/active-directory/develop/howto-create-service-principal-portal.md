---
title: Creación de la identidad de la aplicación de Azure en el portal | Microsoft Docs
description: Describe cómo crear una nueva aplicación de Azure Active Directory y una entidad de servicio que puede utilizarse con el control de acceso basado en rol en Azure Resource Manager para administrar el acceso a los recursos.
services: active-directory
documentationcenter: na
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/14/2019
ms.author: ryanwi
ms.reviewer: tomfitz
ms.custom: seoapril2019
ms.collection: M365-identity-device-management
ms.openlocfilehash: d0208d25e4583672ad2110d959f8e255affbf3e0
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65764851"
---
# <a name="how-to-use-the-portal-to-create-an-azure-ad-application-and-service-principal-that-can-access-resources"></a>Procedimientos para: Uso del portal para crear una aplicación de Azure AD y una entidad de servicio con acceso a los recursos

Este artículo muestra cómo crear una nueva aplicación de Azure Active Directory (Azure AD) y la entidad de servicio que se puede usar con el control de acceso basado en roles. Cuando haya código que deba tener acceso a ciertos recursos o modificarlos, puede crear una identidad para la aplicación. Esta identidad se conoce como una entidad de servicio. Entonces puede asignar los permisos necesarios a la entidad de servicio. En este artículo se muestra cómo usar el portal para crear la entidad de servicio. Se centra en una aplicación de un único inquilino donde la aplicación está diseñada para ejecutarse en una sola organización. Normalmente, utiliza aplicaciones de inquilino único para aplicaciones de línea de negocio que se ejecutan dentro de su organización.

> [!IMPORTANT]
> En lugar de crear una entidad de servicio, considere el uso de identidades administradas para recursos de Azure para la identidad de la aplicación. Si el código se ejecuta en un servicio que admite las identidades administradas y tiene acceso a los recursos que admiten la autenticación de Azure AD, las identidades administradas son una opción mejor para usted. Para obtener más información sobre las identidades administradas para recursos de Azure, incluidos los servicios que actualmente lo admiten, consulte [¿Qué es Managed Identities for Azure Resources?](../managed-identities-azure-resources/overview.md).

## <a name="create-an-azure-active-directory-application"></a>Creación de una aplicación de Azure Active Directory

Vamos a saltar directamente a la creación de la identidad. Si experimenta un problema, compruebe los [permisos necesarios](#required-permissions) para asegurarse de que su cuenta puede crear la identidad.

1. Inicie sesión en su cuenta de Azure mediante [Azure Portal](https://portal.azure.com).
1. Seleccione **Azure Active Directory**.
1. Seleccione **App registrations** (Registros de aplicaciones).

   ![Seleccionar registros de aplicaciones](./media/howto-create-service-principal-portal/select-app-registrations.png)

1. Seleccione **Nuevo registro de aplicaciones**.

   ![Agregar aplicación](./media/howto-create-service-principal-portal/select-add-app.png)

1. Proporcione un nombre y una dirección URL para la aplicación. Seleccione **Aplicación web o API** para indicar el tipo de aplicación que desea crear. No se pueden crear credenciales para una [aplicación nativa](../manage-apps/application-proxy-configure-native-client-application.md). No se puede usar ese tipo en una aplicación automatizada. Después de configurar los valores, seleccione **Crear**.

   ![Aplicación de nombre](./media/howto-create-service-principal-portal/create-app.png)

Ha creado una aplicación de Azure AD y una entidad de servicio.

## <a name="assign-the-application-to-a-role"></a>Asignación de la aplicación a un rol

Para acceder a los recursos de la suscripción, debe asignarle a la aplicación un rol. Decida qué rol ofrece los permisos adecuados para la aplicación. Para obtener más información sobre los roles disponibles, vea [RBAC: Roles integrados](../../role-based-access-control/built-in-roles.md).

Puede establecer el ámbito en el nivel de suscripción, grupo de recursos o recurso. Los permisos se heredan en los niveles inferiores del ámbito. Por ejemplo, el hecho de agregar una aplicación al rol Lector para un grupo de recursos significa que esta puede leer el grupo de recursos y los recursos que contenga.

1. Desplácese hasta el nivel de ámbito al que desea asignar la aplicación. Por ejemplo, para asignar un rol en el ámbito de suscripción, seleccione **Todos los servicios** y **Suscripciones**.

   ![Selección de la suscripción](./media/howto-create-service-principal-portal/select-subscription.png)

1. Seleccione la suscripción concreta que se asignará a la aplicación.

   ![Seleccionar suscripción para la asignación](./media/howto-create-service-principal-portal/select-one-subscription.png)

   Si no ve la suscripción que busca, seleccione el **filtro de suscripciones globales**. Asegúrese de que la suscripción que desea está seleccionada para el portal. 

1. Seleccione **Access Control (IAM)**.
1. Seleccione **Agregar asignación de roles**.

   ![Seleccione Agregar asignación de roles.](./media/howto-create-service-principal-portal/select-add.png)

1. Seleccione el rol que quiere asignar a la aplicación. Para permitir que la aplicación pueda ejecutar acciones como **reiniciar**, **iniciar** y **detener** instancias, debe seleccionar el rol **Colaborador**. De manera predeterminada, las aplicaciones de Azure AD no se muestran en las opciones disponibles. Para encontrar la aplicación, busque el nombre y selecciónelo.

   ![Seleccionar rol](./media/howto-create-service-principal-portal/select-role.png)

1. Haga clic en **Guardar** para finalizar la asignación del rol. Verá la aplicación en la lista de usuarios asignados a un rol para ese ámbito.

La entidad de servicio está configurada. Puede empezar a usarla para ejecutar aplicaciones o scripts. En la sección siguiente se muestra cómo obtener valores necesarios al iniciar sesión mediante programación.

## <a name="get-values-for-signing-in"></a>Obtención de valores para iniciar sesión

### <a name="get-tenant-id"></a>Obtención del identificador de inquilino

Al iniciar sesión mediante programación, deberá pasar el id. de inquilino con la solicitud de autenticación.

1. Seleccione **Azure Active Directory**.
1. Seleccione **Propiedades**.

   ![Selección de las propiedades de Azure AD](./media/howto-create-service-principal-portal/select-ad-properties.png)

1. Copie el **Id. de directorio** para obtener el identificador del inquilino.

   ![Id. de inquilino](./media/howto-create-service-principal-portal/copy-directory-id.png)

### <a name="get-application-id-and-authentication-key"></a>Obtención del id. y la clave de autenticación de la aplicación

También necesitará el identificador de la aplicación y una clave de autenticación. Para obtener estos valores, use los pasos siguientes:

1. En **Registros de aplicaciones**, en Azure AD, seleccione su aplicación.

   ![Seleccionar aplicación](./media/howto-create-service-principal-portal/select-app.png)

1. Copie el **id. de aplicación** y almacénelo en el código de la aplicación.

   ![Identificador de cliente](./media/howto-create-service-principal-portal/copy-app-id.png)

1. Seleccione **certificados y secretos**.

   ![Seleccionar Configuración](./media/howto-create-service-principal-portal/select-certs-secrets.png)

1. Seleccione **los secretos de cliente -> nuevo secreto de cliente**.
1. Proporcione una descripción del secreto y una duración. Cuando haya terminado, seleccione **agregar**.

   ![Guardar el secreto](./media/howto-create-service-principal-portal/save-secret.png)

   Después de guardar el secreto de cliente, se muestra el valor del secreto de cliente. Copie este valor porque no podrá recuperar la clave más adelante. Debe proporcionar el valor de la clave junto con el identificador de la aplicación para iniciar sesión como la aplicación. Guarde el valor de clave donde la aplicación pueda recuperarlo.

   ![Copiar el secreto](./media/howto-create-service-principal-portal/copy-secret.png)

## <a name="required-permissions"></a>Permisos necesarios

Debe tener permisos suficientes para registrar una aplicación en su inquilino de Azure AD y asignar la aplicación a un rol en su suscripción de Azure.

### <a name="check-azure-ad-permissions"></a>Comprobación de los permisos de Azure AD

1. Seleccione **Azure Active Directory**.
1. Anote su rol. Si tiene el rol **Usuario**, debe asegurarse de que los no administradores pueden registrar aplicaciones.

   ![Buscar usuario](./media/howto-create-service-principal-portal/view-user-info.png)

1. Seleccione **Configuración de usuario**.

   ![Seleccionar configuración de usuario](./media/howto-create-service-principal-portal/select-user-settings.png)

1. Compruebe la configuración de **App registrations** (Registros de aplicaciones). Este valor solo puede configurarlo un administrador. Si se configura en **Sí**, cualquier usuario en el inquilino de Azure Active Directory puede registrar una aplicación.

   ![Ver registros de aplicaciones](./media/howto-create-service-principal-portal/view-app-registrations.png)

Si la configuración de registro de aplicaciones se establece en **No**, solo los usuarios con un rol de administrador pueden registrar este tipo de aplicaciones. Consulte los [roles disponibles](../users-groups-roles/directory-assign-admin-roles.md#available-roles) y los [permisos de roles](../users-groups-roles/directory-assign-admin-roles.md#role-permissions) para conocer los roles de administrador disponibles y los permisos específicos en Azure AD que se otorgan a cada rol. Si la cuenta está asignada al rol Usuario, pero la opción Registros de aplicaciones está limitada a los administradores, pida al administrador que le asigne un rol de administrador para poder crear y administrar todos los aspectos de los registros de aplicaciones, o que permita a los usuarios registrar las aplicaciones.

### <a name="check-azure-subscription-permissions"></a>Comprobación de los permisos de suscripción de Azure

En su suscripción de Azure, su cuenta debe tener acceso a `Microsoft.Authorization/*/Write` para asignar una aplicación de AD a un rol. Esta acción se concede mediante el rol [Propietario](../../role-based-access-control/built-in-roles.md#owner) o el rol [Administrador de acceso de usuario](../../role-based-access-control/built-in-roles.md#user-access-administrator). Si su cuenta está asignada al rol **Colaborador**, no tiene los permisos adecuados. Al intentar asignar la entidad de servicio a un rol, se muestra un error.

Para comprobar los permisos de su suscripción:

1. Seleccione su cuenta en la esquina superior derecha y seleccione **... -> Mis permisos**.

   ![Seleccionar permisos de usuario](./media/howto-create-service-principal-portal/select-my-permissions.png)

1. En la lista desplegable, seleccione la suscripción en la que desea crear la entidad de servicio. A continuación, seleccione **Click here to view complete access details for this subscription** (Haga clic aquí para ver los detalles de acceso completos para esta suscripción).

   ![Buscar usuario](./media/howto-create-service-principal-portal/view-details.png)

1. Seleccione **las asignaciones de roles** para ver los roles asignados y determine si tiene los permisos adecuados para asignar una aplicación de AD a un rol. En caso contrario, pida al administrador de suscripciones que le agregue al rol Administrador de acceso de usuario. En la siguiente imagen, el usuario está asignado al rol Propietario, lo que significa que el usuario tiene los permisos adecuados.

   ![Mostrar permisos](./media/howto-create-service-principal-portal/view-user-role.png)

## <a name="next-steps"></a>Pasos siguientes

* Para configurar una aplicación multiinquilino, consulte [Guía del desarrollador para la autorización con la API de Azure Resource Manager](../../azure-resource-manager/resource-manager-api-authentication.md).
* Para obtener información sobre cómo especificar directivas de seguridad, consulte [Control de acceso basado en roles de Azure](../../role-based-access-control/role-assignments-portal.md).  
* Para obtener una lista de las acciones que puede conceder o denegar a los usuarios, consulte [Operaciones del proveedor de recursos de Azure Resource Manager](../../role-based-access-control/resource-provider-operations.md).
