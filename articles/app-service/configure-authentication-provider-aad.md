---
title: Configuración de la autenticación de Azure AD
description: Aprenda a configurar la autenticación de Azure Active Directory como proveedor de identidades para una aplicación de App Service o Azure Functions.
ms.assetid: 6ec6a46c-bce4-47aa-b8a3-e133baef22eb
ms.topic: article
ms.date: 04/14/2020
ms.custom: seodec18, fasttrack-edit, has-adal-ref
ms.openlocfilehash: 4022ca97f88e2f06d3b4c1eeb103bb60d1856eda
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962152"
---
# <a name="configure-your-app-service-or-azure-functions-app-to-use-azure-ad-login"></a>Configuración de una aplicación de App Service o Azure Functions para usar el inicio de sesión de Azure AD

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

En este artículo se muestra cómo configurar Azure App Service o Azure Functions para usar Azure Active Directory (Azure AD) como proveedor de autenticación.

> [!NOTE]
> El flujo de configuración rápida establece un registro de aplicación de AAD V1. Si desea usar [Azure Active Directory v2.0](../active-directory/develop/v2-overview.md) (incluido [MSAL](../active-directory/develop/msal-overview.md)), siga las [instrucciones de configuración avanzada](#advanced).

Siga estos procedimientos recomendados para configurar la aplicación y la autenticación:

- Asigne a cada aplicación de App Service sus propios permisos y consentimiento.
- Configure cada aplicación de App Service con su propio registro.
- Evite el uso compartido de permisos entre entornos mediante registros de aplicación independientes para ranuras de implementación independientes. Al probar nuevo código, esta práctica puede ayudar a evitar que los problemas afecten a la aplicación de producción.

> [!NOTE]
> Esta característica no está disponible actualmente en el plan de Consumo para Linux para Azure Functions.

## <a name="configure-with-express-settings"></a><a name="express"> </a>Configuración rápida

> [!NOTE]
> La opción **Rápida** no está disponible en las nubes de las administraciones públicas.

1. En [Azure Portal], busque y seleccione **App Services** y luego elija la aplicación.
2. En el panel de navegación izquierdo, seleccione **Autenticación/Autorización** > **Activado**.
3. Seleccione **Azure Active Directory** > **Rápido**.

   Si prefiere elegir un registro de aplicación existente:

   1. Elija **Seleccionar aplicación de AD existente** y haga clic en **Aplicación de Azure AD**.
   2. Elija un registro de aplicación existente y haga clic en **Aceptar**.

3. Seleccione **Aceptar** para registrar la aplicación de App Service en Azure Active Directory. Se crea un nuevo registro de la aplicación.

    ![Configuración rápida en Azure Active Directory](./media/configure-authentication-provider-aad/express-settings.png)

4. (Opcional) De manera predeterminada, App Service ofrece autenticación pero no restringe el acceso autorizado al contenido del sitio y a las API. Debe autorizar a los usuarios en el código de la aplicación. Para restringir el acceso a la aplicación solo a los usuarios autenticados mediante Azure Active Directory, configure **Acción necesaria cuando la solicitud no está autenticada** en **Iniciar sesión con Azure Active Directory**. Al establecer esta funcionalidad, la aplicación requiere que se autentiquen todas las solicitudes. También redirige todo lo que no está autenticado a Azure Active Directory para la autenticación.

    > [!CAUTION]
    > Este método de restricción del acceso se aplica a todas las llamadas a la aplicación, lo que puede no interesar en las aplicaciones que tienen una página principal disponible públicamente así como en muchas aplicaciones de página única. Para tales aplicaciones, puede ser preferible **Permitir solicitudes anónimas (ninguna acción)** y que la aplicación inicie manualmente el inicio de sesión. Para más información, consulte [Flujo de autenticación](overview-authentication-authorization.md#authentication-flow).
5. Seleccione **Guardar**.

## <a name="configure-with-advanced-settings"></a><a name="advanced"> </a>Configuración avanzada

Puede configurar la aplicación manualmente si quiere usar un registro de aplicaciones de otro inquilino de Azure AD. Para completar esta configuración personalizada:

1. Crear un registro en Azure AD.
2. Proporcionar algunos detalles de registro a App Service.

### <a name="create-an-app-registration-in-azure-ad-for-your-app-service-app"></a><a name="register"> </a>Creación de un registro de aplicaciones en Azure AD para la aplicación App Service

Para configurar la aplicación de App Service, necesitará la siguiente información:

- Id. de cliente
- Id. de inquilino
- Secreto de cliente (opcional)
- URI de Id. de aplicación

Lleve a cabo los siguiente pasos:

1. Inicie sesión en [Azure Portal], busque y seleccione **App Services** y luego elija la aplicación. Anote la **Dirección URL** de la aplicación. La usará para configurar el registro de la aplicación de Azure Active Directory.
1. Haga clic en **Azure Active Directory** > **Registros de aplicaciones** > **Nuevo registro**.
1. En la página **Register an application** (Registrar una aplicación), escriba el **nombre** del registro de aplicaciones.
1. En **URI de redirección**, seleccione **Web** y escriba `<app-url>/.auth/login/aad/callback`. Por ejemplo, `https://contoso.azurewebsites.net/.auth/login/aad/callback`.
1. Seleccione **Crear**.
1. Una vez creado el registro de la aplicación, copie el **Id. de aplicación (cliente)** y el **Id. de directorio (inquilino)** para usarlos más adelante.
1. Seleccione **Autenticación**. En **Concesión implícita**, habilite **Tokens de id.** para permitir que el usuario de OpenID Connect inicie sesión desde App Service.
1. (Opcional) Seleccione **Personalización de marca**. En **URL de página principal**, escriba la dirección URL de la aplicación de App Service y seleccione **Guardar**.
1. Seleccione **Expose an API (Exponer una API)**  > **Set (Conjunto)** . En el caso de una aplicación de un solo inquilino, pegue la dirección URL de la aplicación de App Service y seleccione **Guardar** y, para la aplicación multiinquilino, pegue la dirección URL que se basa en uno de los dominios comprobados por el inquilino y, a continuación, seleccione **Guardar**.

   > [!NOTE]
   > Este valor es el **URI del identificador de la aplicación** del registro de aplicaciones. Si la aplicación web requiere acceso a una API en la nube, al configurar el recurso de App Service en la nube necesitará el valor de **URI de Id. de aplicación** de la aplicación web. Puede utilizarlo, por ejemplo, si desea que el servicio en la nube conceda acceso explícitamente a la aplicación web.

1. Seleccione **Agregar un ámbito**.
   1. En **Nombre de ámbito**, escriba *user_impersonation*.
   1. En los cuadros de texto, escriba el nombre y la descripción del ámbito de consentimiento que quiere que vean los usuarios en la página de consentimiento. Por ejemplo, escriba *Access my app* (Acceder a mi aplicación).
   1. Seleccione la opción **Agregar un ámbito**.
1. (Opcional) Para crear un secreto de cliente, seleccione **Certificates & secrets (Certificados y secretos)**  > **New client secret (Nuevo secreto de cliente)**  > **Add (Agregar)** . Copie el valor del secreto del cliente que se muestra en la página. No se volverá a mostrar.
1. (Opcional) Para agregar varios valores en **Direcciones URL de respuesta**, seleccione **Autenticación**.

### <a name="enable-azure-active-directory-in-your-app-service-app"></a><a name="secrets"> </a>Habilitación de Azure Active Directory en la aplicación de App Service

1. En [Azure Portal], busque y seleccione **App Services** y luego elija la aplicación.
1. En el panel izquierdo, en **Configuración**, seleccione **Autenticación/Autorización** > **Activado**.
1. (Opcional) De forma predeterminada, la autenticación de App Service permite el acceso no autenticado a la aplicación. Para aplicar la autenticación de usuario, establezca la **acción necesaria cuando la solicitud no está autenticada**, en **Log in with Azure Active Directory** (Iniciar sesión con Azure Active Directory).
1. En **Proveedores de autenticación,** seleccione **Azure Active Directory**.
1. En el **modo de administración**, seleccione **Advanced** (Avanzada) y configure la autenticación de App Service de acuerdo con la tabla siguiente:

    |Campo|Descripción|
    |-|-|
    |Id. de cliente| Use el **identificador de la aplicación (cliente)** del registro de aplicaciones. |
    |Dirección URL del emisor| Use `<authentication-endpoint>/<tenant-id>/v2.0` y reemplace *\<authentication-endpoint>* por el [punto de conexión de autenticación del entorno de nube](../active-directory/develop/authentication-national-cloud.md#azure-ad-authentication-endpoints) (por ejemplo, "https://login.microsoft.com" para Azure global), y reemplace también *\<tenant-id>* por el **identificador de directorio (inquilino)** en el que se creó el registro de la aplicación. Este valor se usa para redirigir a los usuarios al inquilino de Azure AD correcto, así como para descargar los metadatos adecuados para determinar las claves de firma de tokens y el valor de notificación del emisor del token correspondientes, por ejemplo. En las aplicaciones que usan Azure AD v1 y en las aplicaciones de Azure Functions, debe omitirse `/v2.0` en la dirección URL.|
    |Secreto de cliente (opcional)| Use el secreto de cliente que generó en el registro de la aplicación.|
    |Audiencias de token permitidas| Si se trata de una aplicación en la nube o una aplicación de servidor y quiere permitir tokens de autenticación desde una aplicación web, agregue aquí el valor de **URI de Id. de aplicación** de la aplicación web. De forma implícita, el **Id. de cliente** se considera *siempre* que es un público permitido. |

2. Seleccione **Aceptar** y después **Guardar**.

Ahora está preparado para usar Azure Active Directory para realizar la autenticación en la aplicación de App Service.

## <a name="configure-a-native-client-application"></a>Configuración de una aplicación de cliente nativo

Puede registrar clientes nativos para permitir la autenticación en la API web hospedada en la aplicación con una biblioteca cliente como la **Biblioteca de autenticación de Active Directory**.

1. En [Azure Portal], seleccione **Azure Active Directory** > **Registros de aplicaciones** > **Nuevo registro**.
1. En la página **Register an application** (Registrar una aplicación), escriba el **nombre** del registro de aplicaciones.
1. En **URI de redirección**, seleccione **Cliente público (móvil y escritorio)** , y escriba la dirección URL `<app-url>/.auth/login/aad/callback`. Por ejemplo, `https://contoso.azurewebsites.net/.auth/login/aad/callback`.

    > [!NOTE]
    > Para una aplicación de Microsoft Store, use el valor de [SID del paquete](/previous-versions/azure/app-service-mobile/app-service-mobile-dotnet-how-to-use-client-library#package-sid) como URI en su lugar.
1. Seleccione **Crear**.
1. Una vez creado el registro de aplicación, copie el valor de **Id. de aplicación (cliente)** .
1. Seleccione **Permisos de API** > **Agregar permiso** > **Mis API**.
1. Seleccione el registro de aplicaciones que creó anteriormente para la aplicación de App Service. Si no ve el registro de aplicación, compruebe que agregó el ámbito **user_impersonation** en [Creación de un registro de aplicaciones en Azure AD para la aplicación App Service](#register).
1. En **Permisos delegados**, seleccione **user_impersonation** y luego seleccione **Agregar permisos**.

Ahora ha configurado una aplicación cliente nativa que puede acceder a la aplicación de App Service en nombre de un usuario.

## <a name="configure-a-daemon-client-application-for-service-to-service-calls"></a>Configuración de una aplicación cliente demonio para llamadas de servicio a servicio

La aplicación puede adquirir un token para llamar a una API web hospedada en App Service o la aplicación de funciones en nombre propio (no en el nombre de un usuario). Este escenario es útil para las aplicaciones de demonio no interactivas que realizan tareas sin un usuario que ha iniciado sesión. Usa la concesión de [credenciales del cliente](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md) OAuth 2.0 estándar.

1. En [Azure Portal], seleccione **Azure Active Directory** > **Registros de aplicaciones** > **Nuevo registro**.
1. En la página **Register an application** (Registrar una aplicación), escriba el **nombre** del registro de aplicaciones de demonio.
1. En el caso de una aplicación de demonio, no necesita un URI de redireccionamiento para que pueda mantenerlo vacío.
1. Seleccione **Crear**.
1. Una vez creado el registro de aplicación, copie el valor de **Id. de aplicación (cliente)** .
1. Seleccione **Certificados y secretos** > **Nuevo secreto de cliente** > **Agregar**. Copie el valor del secreto del cliente que se muestra en la página. No se volverá a mostrar.

Ahora puede [solicitar un token de acceso mediante el Id. de cliente y el secreto de cliente](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md#first-case-access-token-request-with-a-shared-secret) estableciendo el `resource`parámetro en el **URI del id. de aplicación** de la aplicación de destino. Después, el token de acceso resultante se puede presentar a la aplicación de destino mediante el [encabezado de autorización estándar OAuth 2.0](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md#use-the-access-token-to-access-the-secured-resource), y la autenticación o autorización de App Service validarán y usarán el token como de costumbre para indicar ahora que el autor de llamada (una aplicación en este caso, no un usuario) está autenticado.

En la actualidad, esto permite a _cualquier_ aplicación cliente en el inquilino de Azure AD solicitar un token de acceso y autenticarse en la aplicación de destino. Si también desea aplicar _autorización_ para permitir solo determinadas aplicaciones cliente, debe realizar una configuración adicional.

1. [Defina un rol de aplicación](../active-directory/develop/howto-add-app-roles-in-azure-ad-apps.md) en el manifiesto del registro de la aplicación que representa App Service o la aplicación de función que quiere proteger.
1. En el registro de la aplicación que representa el cliente que debe ser autorizado, seleccione **Permisos de API** > **Agregar un permiso** > **Mis API**.
1. Seleccione el registro de aplicaciones que creó anteriormente. Si no ve el registro de la aplicación, asegúrese de que ha [agregado un rol de aplicación](../active-directory/develop/howto-add-app-roles-in-azure-ad-apps.md).
1. En **Permisos de aplicación**, seleccione el rol de aplicación que creó anteriormente y, a continuación, seleccione **Agregar permisos**.
1. Asegúrese de hacer clic en **Conceder consentimiento del administrador** para autorizar a la aplicación cliente solicitar el permiso.
1. Al igual que en el escenario anterior (antes de que se agregaran los roles), ahora puede [solicitar un token de acceso](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md#first-case-access-token-request-with-a-shared-secret) para el mismo destino`resource`, y el token de acceso incluirá una `roles` solicitud que contiene los roles de aplicación que se autorizaron para la aplicación cliente.
1. En el App Service de destino o el código de la aplicación de función, ahora puede validar que los roles esperados están presentes en el token (esto no se realiza mediante autenticación o autorización de App Service). Para más información, consulte [Access user claims](app-service-authentication-how-to.md#access-user-claims) (Acceso a las notificaciones de usuario).

Ahora ha configurado una aplicación cliente demonio que puede acceder a la aplicación de App Service con su propia identidad.

## <a name="next-steps"></a><a name="related-content"> </a>Pasos siguientes

[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]
* [Tutorial: Autenticación y autorización de usuarios de extremo a extremo en Azure App Service](tutorial-auth-aad.md)
<!-- URLs. -->

[Azure Portal]: https://portal.azure.com/