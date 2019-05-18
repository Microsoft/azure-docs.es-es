---
title: 'Configuración de una aplicación para exponer API web: plataforma de identidad de Microsoft'
description: Aprenda a configurar una aplicación para exponer un nuevo permiso o ámbito y rol para que la aplicación esté disponible para las aplicaciones cliente.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/08/2019
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: aragra, lenalepa, sureshja
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1148fb221e82963ea39ece3e18598e38d633238d
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413953"
---
# <a name="quickstart-configure-an-application-to-expose-web-apis"></a>Inicio rápido: Configuración de una aplicación para exponer las API web

Puede desarrollar una API web y ponerla a disposición de las aplicaciones cliente si expone los [permisos o ámbitos](developer-glossary.md#scopes) y los [roles](developer-glossary.md#roles). Una API web configurada correctamente se pone a disposición de otras aplicaciones del mismo modo que otras API web de Microsoft, incluidas Graph API y las API de Office 365.

En esta guía de inicio rápido, aprenderá a configurar una aplicación para que exponga un nuevo ámbito y lo ponga a disposición de las aplicaciones cliente.

## <a name="prerequisites"></a>Requisitos previos

Para comenzar, asegúrese de que completa estos requisitos previos:

* Aprenda sobre los [permisos y consentimiento](v2-permissions-and-consent.md) admitidos, que son importantes comprender al crear aplicaciones que necesitan ser utilizadas por otros usuarios o aplicaciones.
* Tiene un inquilino que tenga las aplicaciones registradas en él.
  * Si no tiene aplicaciones registradas, [aprenda a registrar aplicaciones con la plataforma de identidad de Microsoft](quickstart-register-app.md).

## <a name="sign-in-to-the-azure-portal-and-select-the-app"></a>Inicio de sesión en Azure Portal y selección de la aplicación

Antes de configurar la aplicación, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. Si la cuenta proporciona acceso a más de un inquilino, haga clic en la cuenta en la esquina superior derecha y establezca la sesión del portal en el inquilino de Azure AD deseado.
1. En el panel de navegación izquierdo, seleccione el servicio **Azure Active Directory** y, después, seleccione **Registros de aplicaciones**.
1. Busque y seleccione la aplicación que desea configurar. Cuando haya seleccionado la aplicación, verá la página **Introducción** de la aplicación o la de registro principal.
1. Elija el método que desea utilizar, la interfaz de usuario o el manifiesto de aplicación, para exponer un ámbito nuevo:
    * [Exposición de un ámbito nuevo mediante la interfaz de usuario](#expose-a-new-scope-through-the-ui)
    * [Exposición de un ámbito nuevo o un rol mediante el manifiesto de aplicación](#expose-a-new-scope-or-role-through-the-application-manifest)

## <a name="expose-a-new-scope-through-the-ui"></a>Exposición de un ámbito nuevo mediante la interfaz de usuario

[![Exposición de una API mediante la interfaz de usuario](./media/quickstart-update-azure-ad-app-preview/expose-api-through-ui-expanded.png)](./media/quickstart-update-azure-ad-app-preview/expose-api-through-ui-expanded.png#lightbox)

Para exponer un ámbito nuevo mediante la interfaz de usuario:

1. Desde la página **Introducción** de la aplicación, seleccione la sección **Exponer una API**.

1. Seleccione **Agregar un ámbito**.

1. Si no ha establecido un **URI de identificador de aplicación**, verá un símbolo del sistema para especificarlo. Escriba el URI de identificador de aplicación o utilice el que se proporciona y, a continuación, seleccione **Guardar y continuar**.

1. Cuando se muestre la página **Agregar un ámbito**, escriba la información del ámbito:

    | Campo | DESCRIPCIÓN |
    |-------|-------------|
    | **Nombre de ámbito** | Escriba un nombre significativo para el ámbito.<br><br>Por ejemplo, `Employees.Read.All`. |
    | **¿Quién puede dar el consentimiento?** | Seleccione si los usuarios pueden dar su consentimiento a este ámbito, o si se requiere el consentimiento del administrador. Seleccione **Solo administradores** para permisos con más privilegios. |
    | **Nombre para mostrar del consentimiento del administrador** | Escriba una descripción significativa para el ámbito, que verán los administradores.<br><br>Por ejemplo: `Read-only access to Employee records` |
    | **Descripción del consentimiento del administrador** | Escriba una descripción significativa para el ámbito, que verán los administradores.<br><br>Por ejemplo: `Allow the application to have read-only access to all Employee data.` |

    Si los usuarios pueden dar su consentimiento al ámbito, agregue también los valores de los campos siguientes:

    | Campo | DESCRIPCIÓN |
    |-------|-------------|
    | **Nombre para mostrar del consentimiento del usuario** | Escriba un nombre significativo para el ámbito, que verán los usuarios.<br><br>Por ejemplo: `Read-only access to your Employee records` |
    | **Descripción del consentimiento del usuario** | Escriba una descripción significativa para el ámbito, que verán los usuarios.<br><br>Por ejemplo: `Allow the application to have read-only access to your Employee data.` |

1. Establezca el **estado** y seleccione **Agregar ámbito** cuando haya terminado.

1. Siga los pasos para [comprobar que la API web se expone a otras aplicaciones](#verify-the-web-api-is-exposed-to-other-applications).

## <a name="expose-a-new-scope-or-role-through-the-application-manifest"></a>Exposición de un ámbito nuevo o un rol mediante el manifiesto de aplicación

[![Exposición de un ámbito nuevo que utilice la colección oauth2Permissions en el manifiesto](./media/quickstart-update-azure-ad-app-preview/expose-new-scope-through-app-manifest-expanded.png)](./media/quickstart-update-azure-ad-app-preview/expose-new-scope-through-app-manifest-expanded.png#lightbox)

Para exponer un ámbito nuevo mediante el manifiesto de aplicación:

1. Desde la página **Introducción** de la aplicación, seleccione la sección **Manifiesto**. Se abre un editor de manifiestos basado en web que le permite **Editar** el manifiesto desde el portal. Si lo desea, puede seleccionar **Descargar**, editar el manifiesto de forma local y, a continuación, usar **Cargar** para volver a aplicarlo a la aplicación.
    
    El ejemplo siguiente muestra cómo exponer un ámbito nuevo llamado `Employees.Read.All` en el recurso o API, mediante la adición del siguiente elemento JSON a la colección `oauth2Permissions`.

      ```json
      {
        "adminConsentDescription": "Allow the application to have read-only access to all Employee data.",
        "adminConsentDisplayName": "Read-only access to Employee records",
        "id": "2b351394-d7a7-4a84-841e-08a6a17e4cb8",
        "isEnabled": true,
        "type": "User",
        "userConsentDescription": "Allow the application to have read-only access to your Employee data.",
        "userConsentDisplayName": "Read-only access to your Employee records",
        "value": "Employees.Read.All"
      }
      ```

   > [!NOTE]
   > El valor `id` debe generarse mediante programación o con una herramienta de generación de GUID, como [guidgen](https://msdn.microsoft.com/library/ms241442%28v=vs.80%29.aspx). `id` representa un identificador único para el ámbito que se expone mediante la API web. Una vez que un cliente se configura apropiadamente con permisos para tener acceso a la API web, Azure AD emite un token de acceso de OAuth 2.0. Cuando el cliente llama a la API web, presenta el token de acceso que tiene la notificación de ámbito (scp) establecida para los permisos solicitados en el registro de la aplicación.
   >
   > Puede exponer ámbitos adicionales posteriormente si es necesario. Tenga en cuenta que la API web podría exponer varios ámbitos asociados a diversas funciones diferentes. El recurso puede controlar el acceso a la API web en tiempo de ejecución, mediante la evaluación de las notificaciones de ámbito (`scp`) del token de acceso de OAuth 2.0 recibido.

1. Cuando termine, haga clic en **Guardar**. Ahora la API web está configurada para que la usen otras aplicaciones del directorio.
1. Siga los pasos para [comprobar que la API web se expone a otras aplicaciones](#verify-the-web-api-is-exposed-to-other-applications).

## <a name="verify-the-web-api-is-exposed-to-other-applications"></a>Comprobación de que la web API se expone a otras aplicaciones

1. Vuelva al inquilino de Azure AD, seleccione **Registros de aplicaciones** y busque o seleccione la aplicación cliente que desea configurar.
1. Repita los pasos descritos en Configuración de una aplicación cliente para tener acceso a las API web.
1. Cuando llegue al paso **Seleccionar una API**, seleccione el recurso. Debería ver el nuevo ámbito, disponible para las solicitudes de permiso de clientes.

## <a name="more-on-the-application-manifest"></a>Más sobre el manifiesto de aplicación

El manifiesto de aplicación actúa como un mecanismo para actualizar la entidad de aplicación, que define todos los atributos de configuración de identidad de una aplicación de Azure AD. Para más información sobre la entidad Application y su esquema, consulte la [documentación sobre la entidad Application de Graph API](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#application-entity). El artículo contiene una completa información de referencia sobre los miembros de la entidad Application utilizados para especificar los permisos para la API, incluidos:  

* El miembro appRoles, que es una colección de entidades [AppRole](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#approle-type) usadas para definir los [permisos de aplicación](developer-glossary.md#permissions) para una API web.
* El miembro oauth2Permissions, que es una colección de entidades [OAuth2Permission](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#oauth2permission-type) usadas para definir los [permisos delegados](developer-glossary.md#permissions) para una API web.

Para más información sobre los conceptos del manifiesto de aplicación en general, consulte [Descripción del manifiesto de aplicación de Azure Active Directory](reference-app-manifest.md).

## <a name="next-steps"></a>Pasos siguientes

Obtenga información acerca de estas otras guías de inicio rápido de administración de aplicaciones relacionadas para aplicaciones:

* [Registro de una aplicación con la plataforma de identidad de Microsoft](quickstart-register-app.md)
* [Configuración de una aplicación cliente para tener acceso a las API web](quickstart-configure-app-access-web-apis.md)
* [Modificación de las cuentas compatibles con una aplicación](quickstart-modify-supported-accounts.md)
* [Eliminación de una aplicación registrada con la plataforma de identidad de Microsoft](quickstart-remove-app.md)

Para obtener más información acerca de los dos objetos de Azure AD que representan una aplicación registrada y la relación entre ellos, consulte [Objetos de aplicación y de entidad de servicio](app-objects-and-service-principals.md).

Para obtener más información acerca de las directrices de personalización de marca que hay que usar al desarrollar aplicaciones con Azure Active Directory, consulte [Directrices de personalización de marca para aplicaciones](howto-add-branding-in-azure-ad-apps.md).
