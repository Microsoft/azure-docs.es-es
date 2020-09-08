---
title: 'Inicio rápido: Configuración de una aplicación para exponer una API web | Azure'
titleSuffix: Microsoft identity platform
description: En este inicio rápido, aprenderá a configurar una aplicación para exponer un nuevo permiso o ámbito y un rol para que la aplicación esté disponible para las aplicaciones cliente.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 08/05/2020
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: aragra, lenalepa, sureshja
ms.openlocfilehash: 93b0c3392a32a6ff18a285d34fdaede6ceea6528
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87830298"
---
# <a name="quickstart-configure-an-application-to-expose-a-web-api"></a>Inicio rápido: Configuración de una aplicación para exponer una API web

Puede desarrollar una API web y ponerla a disposición de las aplicaciones cliente si expone los [permisos o ámbitos](developer-glossary.md#scopes) y los [roles](developer-glossary.md#roles). Una API web configurada correctamente se pone a disposición de otras aplicaciones del mismo modo que otras API web de Microsoft, incluidas Graph API y las API de Office 365.

En este inicio rápido, aprenderá a configurar una aplicación para que exponga un nuevo ámbito y lo ponga a disposición de las aplicaciones cliente.

## <a name="prerequisites"></a>Prerrequisitos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Finalización de [Inicio rápido: Registro de una aplicación en la plataforma de identidad de Microsoft](quickstart-register-app.md).

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

[![Muestra cómo exponer una API mediante la interfaz de usuario](./media/quickstart-update-azure-ad-app-preview/expose-api-through-ui-expanded.png)](./media/quickstart-update-azure-ad-app-preview/expose-api-through-ui-expanded.png#lightbox).

Para exponer un ámbito nuevo mediante la interfaz de usuario:

1. Desde la página **Introducción** de la aplicación, seleccione la sección **Exponer una API**.

1. Seleccione **Agregar un ámbito**.

1. Si no ha establecido un **URI de identificador de aplicación**, verá un símbolo del sistema para especificarlo. Escriba el URI de identificador de aplicación o utilice el que se proporciona y, a continuación, seleccione **Guardar y continuar**.

1. Cuando se muestre la página **Agregar un ámbito**, escriba la información del ámbito:

    | Campo | Descripción |
    |-------|-------------|
    | **Nombre de ámbito** | Escriba un nombre significativo para el ámbito.<br><br>Por ejemplo, `Employees.Read.All`. |
    | **¿Quién puede dar el consentimiento?** | Seleccione si los usuarios pueden dar su consentimiento a este ámbito, o si se requiere el consentimiento del administrador. Seleccione **Solo administradores** para permisos con más privilegios. |
    | **Nombre para mostrar del consentimiento del administrador** | Escriba una descripción significativa para el ámbito, que verán los administradores.<br><br>Por ejemplo: `Read-only access to Employee records` |
    | **Descripción del consentimiento del administrador** | Escriba una descripción significativa para el ámbito, que verán los administradores.<br><br>Por ejemplo: `Allow the application to have read-only access to all Employee data.` |

    Si los usuarios pueden dar su consentimiento al ámbito, agregue también los valores de los campos siguientes:

    | Campo | Descripción |
    |-------|-------------|
    | **Nombre para mostrar del consentimiento del usuario** | Escriba un nombre significativo para el ámbito, que verán los usuarios.<br><br>Por ejemplo: `Read-only access to your Employee records` |
    | **Descripción del consentimiento del usuario** | Escriba una descripción significativa para el ámbito, que verán los usuarios.<br><br>Por ejemplo: `Allow the application to have read-only access to your Employee data.` |

1. Establezca el **estado** y seleccione **Agregar ámbito** cuando haya terminado.

1. (Opcional) Para eliminar la solicitud de consentimiento por parte de los usuarios de su aplicación en los ámbitos que ha definido, puede "preautorizar" a la aplicación cliente para que acceda a su API web. Debe preautorizar *únicamente* las aplicaciones cliente en las que confíe, ya que los usuarios no tendrán la oportunidad de rechazar el consentimiento.
    1. En **Aplicaciones cliente autorizadas**, seleccione **Agregar una aplicación cliente**
    1. Escriba el valor de **Id. de aplicación (cliente)** de la aplicación cliente que desea preautorizar. Por ejemplo, el de una aplicación web que ha registrado previamente.
    1. En **Ámbitos autorizados**, seleccione los ámbitos en que desea suprimir la solicitud de consentimiento y, después, seleccione **Agregar aplicación**.

    La aplicación cliente es ahora una aplicación cliente preautorizada (PCA) y no se pedirá a los usuarios su consentimiento al iniciar sesión en ella.

1. Siga los pasos para [comprobar que la API web se expone a otras aplicaciones](#verify-the-web-api-is-exposed-to-other-applications).

## <a name="expose-a-new-scope-or-role-through-the-application-manifest"></a>Exposición de un ámbito nuevo o un rol mediante el manifiesto de aplicación

El manifiesto de la aplicación actúa como un mecanismo para actualizar la entidad de aplicación que define los atributos de un registro de aplicación de Azure AD.

[![Exposición de un ámbito nuevo que utilice la colección oauth2Permissions en el manifiesto](./media/quickstart-update-azure-ad-app-preview/expose-new-scope-through-app-manifest-expanded.png)](./media/quickstart-update-azure-ad-app-preview/expose-new-scope-through-app-manifest-expanded.png#lightbox)

Para exponer un nuevo ámbito editando el manifiesto de la aplicación:

1. Desde la página **Introducción** de la aplicación, seleccione la sección **Manifiesto**. Se abre un editor de manifiestos basado en web que le permite **Editar** el manifiesto desde el portal. Si lo desea, puede seleccionar **Descargar**, editar el manifiesto de forma local y, a continuación, usar **Cargar** para volver a aplicarlo a la aplicación.

    El ejemplo siguiente muestra cómo exponer un ámbito nuevo llamado `Employees.Read.All` en el recurso o API, mediante la adición del siguiente elemento JSON a la colección `oauth2Permissions`.

    Genere el valor `id` mediante programación o mediante una herramienta de generación de GUID como [guidgen](https://www.microsoft.com/download/details.aspx?id=55984).

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

1. Cuando termine, haga clic en **Guardar**. Ahora la API web está configurada para que la usen otras aplicaciones del directorio.
1. Siga los pasos para [comprobar que la API web se expone a otras aplicaciones](#verify-the-web-api-is-exposed-to-other-applications).

Para obtener más información sobre la entidad de aplicación y su esquema, consulte la documentación de referencia de los tipos de recursos [Aplicación][ms-graph-application] de Microsoft Graph.

Para obtener más información sobre el manifiesto de la aplicación, incluida la referencia de su esquema, consulte [Descripción del manifiesto de aplicación de Azure AD](reference-app-manifest.md).

## <a name="verify-the-web-api-is-exposed-to-other-applications"></a>Comprobación de que la web API se expone a otras aplicaciones

1. Vuelva al inquilino de Azure AD, seleccione **Registros de aplicaciones** y busque o seleccione la aplicación cliente que desea configurar.
1. Repita los pasos descritos en [Configuración de una aplicación cliente para tener acceso a las API web](quickstart-configure-app-access-web-apis.md).
1. Cuando llegue al paso en que debe [seleccionar una API](quickstart-configure-app-access-web-apis.md#add-permissions-to-access-web-apis), seleccione el recurso (el registro de la aplicación de API web).
    * Si creó el registro de la aplicación API web desde Azure Portal, el recurso de la API se muestra en la pestaña **Mis API**.
    * Si permitió que Visual Studio creara el registro de la aplicación de API web durante la creación del proyecto, el recurso de la API aparecerá en la pestaña **API usadas en mi organización**.

Una vez que haya seleccionado el recurso de la API web, debería ver el nuevo ámbito, disponible para las solicitudes de permiso de clientes.

## <a name="using-the-exposed-scopes"></a>Uso de los ámbitos expuestos

Una vez que un cliente se configure apropiadamente con permisos para acceder a la API web, Azure AD puede emitir un token de acceso de OAuth 2.0. Cuando el cliente llama a la API web, presenta el token de acceso que tiene la notificación de ámbito (`scp`) establecida para los permisos solicitados en su registro de aplicación.

Puede exponer ámbitos adicionales posteriormente si es necesario. Tenga en cuenta que la API web podría exponer varios ámbitos asociados a diversas funciones diferentes. El recurso puede controlar el acceso a la API web en tiempo de ejecución, mediante la evaluación de las notificaciones de ámbito (`scp`) del token de acceso de OAuth 2.0 recibido.

En las aplicaciones, el valor de ámbito completo es una concatenación del **URI del identificador de aplicación** (el recurso) y el **nombre del ámbito** de la API web.

Por ejemplo, si el identificador URI de aplicación de la API web es `https://contoso.com/api` y el nombre del ámbito es `Employees.Read.All`, el ámbito completo es:

`https://contoso.com/api/Employees.Read.All`

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha expuesto su API web mediante la configuración de sus ámbitos, configure el registro de la aplicación cliente con permiso para acceder a esos ámbitos.

> [!div class="nextstepaction"]
> [Configuración de una aplicación para acceder a una API web](quickstart-configure-app-access-web-apis.md)

<!-- REF LINKS -->
[ms-graph-application]: /graph/api/resources/application
