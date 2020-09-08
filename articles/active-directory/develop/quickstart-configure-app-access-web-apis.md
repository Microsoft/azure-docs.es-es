---
title: 'Inicio rápido: Configuración de una aplicación para acceder a una API web | Azure'
titleSuffix: Microsoft identity platform
description: En este inicio rápido, configurará una aplicación registrada con la Plataforma de identidad de Microsoft para que incluya los identificadores URI de redireccionamiento, las credenciales o los permisos para acceder a las API web.
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
ms.reviewer: lenalepa, aragra, sureshja
ms.openlocfilehash: 87c21587567ffe3462e4b702985114ac10454886
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88140809"
---
# <a name="quickstart-configure-a-client-application-to-access-a-web-api"></a>Inicio rápido: Configuración de una aplicación cliente para acceder a las API web

En este inicio rápido, se agregarán URI de redireccionamiento, credenciales o permisos para acceder a las API web de una aplicación. Una aplicación cliente web o confidencial debe establecer credenciales seguras para participar en un flujo de concesión de autorización que requiera autenticación. El método de autenticación predeterminado compatible con Azure Portal es un identificador de cliente y una clave secreta. La aplicación obtiene un token de acceso durante este proceso.

Para que un cliente pueda acceder a una API web expuesta por una aplicación de recursos (como Microsoft Graph API), el marco de consentimiento garantiza que el cliente obtenga la concesión de los permisos necesarios para los solicitados. De forma predeterminada, todas las aplicaciones pueden solicitar permisos de Microsoft Graph API.

## <a name="prerequisites"></a>Prerrequisitos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Finalización de [Inicio rápido: Configuración de una aplicación para exponer las API web](quickstart-configure-app-expose-web-apis.md).

## <a name="sign-in-to-the-azure-portal-and-select-the-app"></a>Inicio de sesión en Azure Portal y selección de la aplicación

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. Si la cuenta proporciona acceso a más de un inquilino, seleccione su cuenta en la esquina superior derecha. Establezca la sesión del portal en el inquilino de Azure AD que desee.
1. Busque y seleccione **Azure Active Directory**. En **Administrar**, seleccione **Registros de aplicaciones**.
1. Busque y seleccione la aplicación que desea configurar. Cuando seleccione la aplicación, verá la página **Información general** de la aplicación o la de registro principal.

Use los procedimientos siguientes para configurar la aplicación y acceder a las API web.

## <a name="add-redirect-uris-to-your-application"></a>Adición de identificadores URI de redirección a la aplicación

Puede agregar URI de redirección personalizados y sugeridos a la aplicación. Para agregar un URI de redirección personalizado para aplicaciones cliente web y públicas:

1. En la página **Información general** de la aplicación, seleccione **Autenticación**.
1. Busque **URI de redirección**. Es posible que tenga que seleccionar **Cambiar a la experiencia anterior**.
1. Seleccione el tipo de aplicación que va a crear: **Web** o **Cliente público o nativo (móvil o escritorio)** .
1. Escriba el identificador URI de redireccionamiento para la aplicación.

   * Para aplicaciones web, proporcione la dirección URL base de la aplicación. Por ejemplo, `http://localhost:31544` podría ser la dirección URL de una aplicación web que se ejecuta en la máquina local. Los usuarios utilizan esta dirección URL para iniciar sesión en una aplicación cliente web.
   * Para aplicaciones públicas, proporcione el identificador URI que utiliza Azure AD para devolver las respuestas de los tokens. Escriba un valor específico para la aplicación, como `https://MyFirstApp`.
1. Seleccione **Guardar**.

Para elegir entre los identificadores URI de redirección sugeridos para clientes públicos, siga estos pasos:

1. En la página **Información general** de la aplicación, seleccione **Autenticación**.
1. Busque **URI de redireccionamiento sugeridos para clientes públicos (para dispositivos móviles o de escritorio)** . Es posible que tenga que seleccionar **Cambiar a la experiencia anterior**.
1. Seleccione uno o varios URI de redirección para la aplicación. También puede escribir un URI de redireccionamiento personalizado. Si no está seguro de lo que debe usar, consulte la documentación de la biblioteca.
1. Seleccione **Guardar**.

A los URI de redirección se les aplican ciertas restricciones. Para más información, consulte [Limitaciones y restricciones de URI de redireccionamiento o URL de respuesta](./reply-url.md).

> [!NOTE]
> Pruebe la nueva experiencia de configuración de **autenticación**, donde puede configurar su aplicación basándose en la plataforma o dispositivo que tenga como destino.
>
> Para acceder a esta vista, seleccione **Probar la nueva experiencia** en la página **Autenticación**.
>
> ![Haga clic en “Probar la nueva experiencia” para ver la vista Configuración de la plataforma.](./media/quickstart-update-azure-ad-app-preview/authentication-try-new-experience-cropped.png)
>
> Se abrirá la [nueva página **Configuraciones de plataforma**](#configure-platform-settings-for-your-application).

### <a name="configure-advanced-settings-for-your-application"></a>Configuración avanzada de la aplicación

Según la aplicación que quiera registrar, hay opciones adicionales que puede que tenga que configurar, como las siguientes:

* **Dirección URL de cierre de sesión**
* Para las aplicaciones de una página, puede habilitar la opción **Concesión implícita** y seleccionar los tokens que quiera que emita el punto de conexión de autorización.
* Para las aplicaciones de escritorio que obtienen tokens con la Autenticación integrada de Windows, un flujo de código del dispositivo o un nombre de usuario y contraseña en la sección **Tipo de cliente predeterminado**, establezca la opción **Considerar la aplicación como un cliente público** en **Sí**.
* En el caso de las aplicaciones heredadas que usaban el SDK de LIVE para la integración con el servicio de cuentas Microsoft, establezca la opción **Compatibilidad con el SDK de Live**. Las nuevas aplicaciones no necesitan esta configuración.
* **Tipo de cliente predeterminado**.
* **Tipos de cuenta admitidos**.

### <a name="modify-supported-account-types"></a>Modificación de los tipos de cuenta admitidos

La opción **Tipos de cuenta admitidos** especifica quién puede usar la aplicación o acceder a la API.

Si configuró los tipos de cuenta admitidos al registrar la aplicación, solo podrá cambiar esta opción mediante el editor del manifiesto de la aplicación si:

* Cambia los tipos de cuenta de **AzureADMyOrg** o **AzureADMultipleOrgs** a **AzureADandPersonalMicrosoftAccount**, o viceversa, o bien
* Cambia los tipos de cuenta de **AzureADMyOrg** a **AzureADMultipleOrgs**, o viceversa.

Para cambiar los tipos de cuenta admitidos de un registro de aplicación existente, actualice la clave `signInAudience`. Para más información, consulte [Configuración del manifiesto de aplicación](reference-app-manifest.md#configure-the-app-manifest).

## <a name="configure-platform-settings-for-your-application"></a>Configuración de la plataforma para su aplicación

![Configuración de la aplicación según la plataforma o dispositivo](./media/quickstart-update-azure-ad-app-preview/authentication-new-platform-configurations.png)

Para configurar la aplicación según la plataforma o dispositivo que tenga como destino:

1. En la página **Configuraciones de plataforma**, haga clic en **Agregar una plataforma** y seleccione una de las opciones disponibles.

   ![Se muestra la página Configurar plataformas.](./media/quickstart-update-azure-ad-app-preview/authentication-platform-configurations-configure-platforms.png)

1. Especifique la información de configuración según la plataforma que haya seleccionado.

   | Plataforma                | Parámetros de configuración            |
   |-------------------------|-----------------------------------|
   | **Web**              | Escriba el **URI de redireccionamiento** para la aplicación. |
   | **iOS/macOS**              | Escriba el **Identificador de paquete** de la aplicación, que encontrará en XCode en Info.plist, o bien en Configuración de compilación. Al agregar del identificador del paquete, se crea automáticamente un URI de redireccionamiento para la aplicación. |
   | **Android**          | Especifique el **Nombre del paquete** de la aplicación, que encontrará en el archivo AndroidManifest.xml.<br/>Genere y especifique el **Hash de firma**. Al agregar el hash de firma, se crea automáticamente un URI de redireccionamiento para la aplicación.  |
   | **Aplicaciones móviles y de escritorio**  | Opcional. Si va a crear aplicaciones para dispositivos y equipos de escritorio, seleccione uno de los **URI de redireccionamiento sugeridos**.<br/>Opcional. Escriba un **URI de redireccionamiento personalizado**, que se usa como la ubicación a la que Azure AD redirigirá a los usuarios en respuesta a las solicitudes de autenticación. Por ejemplo, para las aplicaciones de .NET Core en las que quiera agregar una interacción, use `http://localhost`. |

   > [!NOTE]
   > En Servicios de federación de Active Directory (AD FS) y Azure AD B2C, también debe especificar un número de puerto.  Por ejemplo: `http://localhost:1234`.

   > [!IMPORTANT]
   > En el caso de las aplicaciones móviles que no usen la biblioteca de autenticación de Microsoft (MSAL) más reciente o que no usen un agente, tendrá que configurar los URI de redireccionamiento para estas aplicaciones en **Escritorio y dispositivos**.

Según la plataforma que use, es posible que pueda configurar opciones adicionales. Para las aplicaciones **web**, puede realizar estas acciones:

* Adición de URI de redireccionamiento
* Configure la opción **Concesión implícita** para seleccionar los tokens que quiera que emita el punto de conexión de autorización:

  * Para aplicaciones de una página, seleccione las opciones **Tokens de acceso** y **Tokens de identificador**.
  * Para las aplicaciones web, seleccione **Tokens de identificador**.

## <a name="add-credentials-to-your-web-application"></a>Adición de credenciales a la aplicación web

Para agregar una credencial a la aplicación web, agregue un certificado o cree un secreto de cliente. Para agregar un certificado:

1. Desde la página **Información general** de la aplicación, seleccione la sección **Certificados y secretos**.
1. Seleccione **Cargar certificado**.
1. Seleccione el archivo que desea cargar. Debe ser uno de los siguientes tipos de archivo: .cer, .pem, .crt.
1. Seleccione **Agregar**.

Para agregar un secreto de cliente:

1. En la página **Información general** de la aplicación, seleccione la sección **Certificados y secretos**.
1. Seleccione **Nuevo secreto de cliente**.
1. Agregue una descripción para el secreto de cliente.
1. Seleccione una duración.
1. Seleccione **Agregar**.

> [!NOTE]
> Después de guardar los cambios de configuración, la columna en el extremo derecho contendrá el valor del secreto de cliente. **Asegúrese de copiar el valor** para su uso en el código de la aplicación cliente, ya que no es accesible una vez se sale de esta página.

## <a name="add-permissions-to-access-web-apis"></a>Adición de permisos para acceder a las API web

El [permiso Iniciar sesión y leer el perfil de usuario de Graph API](/graph/permissions-reference#user-permissions) se selecciona de forma predeterminada. Puede seleccionar entre [dos tipos de permisos](developer-glossary.md#permissions) para cada API web:

* **Permisos de aplicación**. La aplicación cliente necesita acceso directo a la API web como sí misma, sin contexto de usuario. Este tipo de permiso requiere el consentimiento del administrador. Este permiso no está disponible para aplicaciones cliente de escritorio y móviles.
* **Permisos delegados**. la aplicación cliente necesita acceso a la API web como el usuario que inició sesión, pero con acceso limitado por el permiso seleccionado. Este tipo de permiso lo puede conceder un usuario a menos que el permiso requiera el consentimiento del administrador.

  > [!NOTE]
  > La adición de un permiso delegado a una aplicación no concede consentimiento automáticamente a los usuarios del inquilino. Los usuarios tienen todavía que conceder manualmente el consentimiento para los permisos delegados agregados en tiempo de ejecución, a menos que lo haga el administrador en nombre de todos los usuarios.

Para agregar permisos para acceder a las API de recursos desde el cliente:

1. En la página **Información general** de la aplicación, seleccione **Permisos de API**.
1. En **Permisos configurados**, seleccione **Agregar un permiso**.
1. De forma predeterminada, la vista le permite seleccionar en las **API de Microsoft**. Seleccione la sección de las API que le interesa:

    * **API de Microsoft**. Le permite seleccionar los permisos para las API de Microsoft, como Microsoft Graph.
    * **API usadas en mi organización**. Le permite seleccionar los permisos para las API que expone la organización o las API con las que la organización se ha integrado.
    * **Mis API**. Le permite seleccionar los permisos para las API que ha expuesto.

1. Una vez que haya seleccionado las API, verá la página **Solicitud de permisos de API**. Si la API expone permisos delegados y de aplicación, seleccione el tipo de permiso que necesita la aplicación.
1. Cuando termine, seleccione **Agregar permisos**.

Vuelva a la página **Permisos de API**. Los permisos se han guardado y agregado a la tabla.

## <a name="understanding-api-permissions-and-admin-consent-ui"></a>Descripción de los permisos de API y la UI de consentimiento del administrador

### <a name="configured-permissions"></a>Permisos configurados

En esta sección se muestran los permisos que se han configurado explícitamente en el objeto de aplicación. Estos permisos forman parte de la lista de acceso a recursos necesarios de la aplicación. Puede agregar o quitar permisos de esta tabla. Como administrador, también puede conceder o revocar el consentimiento del administrador para un conjunto de permisos de una API o permisos individuales.

### <a name="other-permissions-granted"></a>Otros permisos concedidos

Si la aplicación está registrada en un inquilino, es posible que vea una sección adicional titulada **Other permissions granted for Tenant** (Otros permisos concedidos para el inquilino). En esta sección se muestran los permisos concedidos para el inquilino que no se han configurado explícitamente en el objeto de aplicación. Estos permisos se solicitaron y consintieron dinámicamente. Esta sección solo aparece si hay al menos un permiso que se aplique.

Puede agregar un conjunto de permisos de API o permisos individuales que aparecen en esta sección a la sección **Permisos configurados**. Como administrador, en esta sección también puede revocar el consentimiento del administrador para API o permisos individuales.

### <a name="admin-consent-button"></a>Botón Consentimiento del administrador

Si la aplicación está registrada en un inquilino, verá un botón **Grant admin consent for Tenant** (Conceder consentimiento del administrador al inquilino). Estará deshabilitado si no es el administrador o si no se han configurado permisos para la aplicación.
Este botón permite a un administrador conceder el consentimiento del administrador a los permisos configurados para la aplicación. Al hacer clic en el botón de consentimiento del administrador, se abre una nueva ventana con una petición de consentimiento que muestra todos los permisos configurados.

> [!NOTE]
> Hay un retraso entre que se configuran los permisos para la aplicación y que aparecen en la petición de consentimiento. Si no ve todos los permisos configurados en la petición de consentimiento, ciérrela y vuelva a iniciarla.

Si tiene permisos que se han concedido pero no configurado, el botón de consentimiento del administrador le pedirá que controle estos permisos. Puede agregarlos a los permisos configurados o puede quitarlos.

La petición de consentimiento da la opción de **Aceptar** o **Cancelar**. Seleccione **Aceptar** para conceder el consentimiento del administrador. Si selecciona **Cancelar**, no se concederá el consentimiento del administrador. Un mensaje de error indica que se ha rechazado el consentimiento.

> [!NOTE]
> Hay un retraso entre el momento en que se concede el consentimiento del administrador cuando selecciona **Aceptar** en la petición de consentimiento, y el momento en que el estado del consentimiento del administrador se ve reflejado en el portal.

## <a name="next-steps"></a>Pasos siguientes

Avance al siguiente inicio rápido de la serie para aprender a configurar los tipos de cuenta que pueden acceder a la aplicación. Por ejemplo, puede que quiera limitar el acceso solo a los usuarios de su organización (un solo inquilino) o permitir a los usuarios de otros inquilinos de Azure AD (multiinquilino) y a quienes tengan cuentas personales de Microsoft (MSA).

> [!div class="nextstepaction"]
> [Modificación de las cuentas compatibles con una aplicación](quickstart-modify-supported-accounts.md)