---
title: Protección de contenido de un extremo a otro con Azure AD
description: En este artículo obtendrá información sobre cómo proteger el contenido con Azure Media Services y Azure Active Directory
services: media-services
documentationcenter: ''
author: willzhan
manager: femila
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-javascript
ms.openlocfilehash: 006e312e67f5f4014248c44a799c2dde826801c2
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89258850"
---
# <a name="tutorial-end-to-end-content-protection-using-azure-ad"></a>Tutorial: Protección de contenido de un extremo a otro con Azure AD

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Con este tutorial y el ejemplo de reproductor proporcionado, puede configurar un subsistema de protección de contenido multimedia de un extremo a otro en Azure Media Services (AMS) y Azure Active Directory (AAD) para transmitir contenido multimedia con todos los formatos DRM/AES-128, protocolos de streaming, códec y contenedores compatibles con AMS. El ejemplo es lo suficientemente genérico para proteger el acceso a cualquier API REST protegida por OAuth 2 mediante el flujo de código de autorización con clave de prueba para el intercambio de códigos (PKCE). (El servicio de entrega de licencias de Azure Media Services es solo uno de ellos). También funciona para Microsoft Graph API o cualquier API REST de desarrollo personalizado protegida con el flujo de código de autorización de OAuth 2. Este es el documento complementario al [código de ejemplo](https://github.com/Azure-Samples/media-services-content-protection-azure-ad).

En este tutorial, aprenderá lo siguiente:

> [!div class="checklist"]
>
> * Tener en cuenta los requisitos de la autenticación
> * Comprender cómo funciona la aplicación
> * Registrar una aplicación de recursos de back-end
> * Registrar una aplicación cliente
> * Configurar la directiva de clave de contenido de la cuenta de Media Services y las directivas de streaming
> * Configurar la aplicación de reproducción

Si no tiene una suscripción de Azure Media Services, cree una [cuenta de evaluación gratuita](https://azure.microsoft.com/free/) de Azure y, a continuación, cree una cuenta de Media Services.

### <a name="duration"></a>Duration
El tutorial debería tardar unas dos horas en completarse una vez que la tecnología de los requisitos previos esté lista para su uso.

## <a name="prerequisites"></a>Requisitos previos

Se usan los siguientes conceptos y versiones más recientes de la tecnología. Se recomienda que se familiarice con ellos antes de comenzar este tutorial.

### <a name="prerequisite-knowledge"></a>Conocimiento de los requisitos previos

Es opcional, pero se recomienda que esté familiarizado con los siguientes conceptos antes de comenzar este tutorial:

* Administración de derechos digitales (DRM)
* [Azure Media Services (AMS) v3](./media-services-overview.md)
* Las [directivas de clave de contenido](content-key-policy-concept.md) de AMS mediante la API de AMS v3, Azure Portal o la [herramienta Azure Media Services Explorer (AMSE)](https://github.com/Azure/Azure-Media-Services-Explorer)
* Puntos de conexión v2 de Azure AD en la [Plataforma de identidad de Microsoft](../../active-directory/develop/index.yml)
* Autenticación moderna en la nube, como [OAuth 2.0 y OpenID Connect](../../active-directory/develop/active-directory-v2-protocols.md)
  * [Flujo de código de autorización en OAuth 2.0](../../active-directory/develop/v2-oauth2-auth-code-flow.md) y por qué se necesita PKCE
  * [Permisos delegados frente a permisos de aplicación](../../active-directory/develop/developer-glossary.md#permissions)
* El [token JWT](../../active-directory/develop/access-tokens.md), sus notificaciones y la sustitución de claves de firma (se incluye en el ejemplo).

### <a name="prerequisite-code-and-installations"></a>Requisitos previos de código e instalaciones

* El código de ejemplo. Descargue el [código de ejemplo](https://github.com/Azure-Samples/media-services-content-protection-azure-ad).
* Instalación de Visual Studio Code. Descargue Visual Studio Code aquí: [https://code.visualstudio.com/download](https://code.visualstudio.com/download).
* Una instalación de Node.js. Descargue Node.js aquí: [https://nodejs.org](https://nodejs.org). NPM viene incluido en la instalación.
* Una [suscripción de Azure](https://azure.microsoft.com/free/).
* Una cuenta de Azure Media Services (AMS).
* @azure/msal-browser v2.0, uno de los miembros de la familia de SDK de la [Biblioteca de autenticación de Microsoft (MSAL)](../../active-directory/develop/msal-overview.md) para distintas plataformas de cliente
* La versión más reciente de [Azure Media Player](https://github.com/Azure-Samples/azure-media-player-samples) (incluida en el ejemplo).
* Las credenciales de FPS de Apple si quiere incluir FairPlay DRM y el certificado de aplicación hospedado con CORS que es accesible mediante JavaScript del lado cliente.

> [!IMPORTANT]
> En este tutorial se usa .NET para crear la restricción de la directiva de clave de contenido.  Si no es un desarrollador de .NET, pero desea probar Node.js para conectarse a Azure Media Services, consulte [Conexión a la API v3 de Media Services: Node.js](configure-connect-nodejs-howto.md). También hay un módulo de Node.js disponible para controlar la sustitución de claves de forma automática. Consulte [Módulo passport-ad](https://github.com/AzureAD/passport-azure-ad) de Node.js.

## <a name="consider-the-authentication-and-authorization-requirements"></a>Tener en cuenta los requisitos de la autenticación y la autorización

Se presentan algunos desafíos en el diseño del subsistema. Tiene varias partes móviles, hay restricciones de la aplicación cliente y la sustitución de clave de Azure AD, que se produce cada seis semanas.

La aplicación de página única (SPA) que se usa en este tutorial tiene en cuenta los desafíos de los requisitos de autenticación y las restricciones que se indican a continuación. Usa:

* Puntos de conexión v2 de Azure AD,ya que la plataforma para desarrolladores de Azure AD (puntos de conexión v1) se traslada a la Plataforma de identidad de Microsoft (puntos de conexión v2).
* Flujo de código de autorización, porque el flujo de concesión implícita de OAuth 2 ha quedado en desuso.
* Una aplicación sujeta a las siguientes restricciones:
    * Un cliente público no puede ocultar el secreto de cliente.  El flujo de código de autorización por sí solo requiere ocultar el secreto de cliente, por lo que se usa el flujo de código de autorización con PKCE.
    * Una aplicación basada en explorador sujeta a un espacio aislado de seguridad del explorador (restricción de CORS/preparatoria).
    * Una aplicación basada en explorador que usa JavaScript moderno sujeta a las restricciones de seguridad de JavaScript (accesibilidad del encabezado personalizado, identificador de correlación).

## <a name="understand-the-subsystem-design"></a>Descripción del diseño del subsistema

El diseño del subsistema se muestra en el diagrama siguiente.  Tiene tres capas:

* Capa del área de operaciones (en color negro), para configurar la directiva de clave de contenido y publicar contenido para streaming
* Puntos de conexión públicos (en color azul), que son puntos de conexión orientados al cliente y al reproductor que proporcionan autenticación, autorización, licencia de DRM y contenido cifrado
* Aplicación de reproducción (en color azul claro), que integra todos los componentes y
    * controla la autenticación de usuario mediante Azure AD.
    * controla la adquisición del token de acceso de Azure AD.
    * recibe el manifiesto y el contenido cifrado de AMS/CDN.
    * adquiere la licencia de DRM de Azure Media Services.
    * controla el descifrado, la descodificación y la visualización del contenido.

![pantalla para analizar tokens JWT](media/aad-ams-content-protection/subsystem.svg)

Consulte [Diseño del sistema de protección de contenido con DRM múltiple con control de acceso](./design-multi-drm-system-with-access-control.md) para más información sobre el subsistema.

## <a name="understand-the-single-page-app"></a>Descripción de la aplicación de página única

La aplicación de reproducción es una aplicación de página única (SPA), desarrollada en Visual Studio Code con:

* Node.js con JavaScript ES 6
* @azure/msal-browser 2.0 beta
* SDK de Azure Media Player
* Flujo de OAuth 2 en puntos de conexión v2 de Azure AD (Plataforma de identidad de Microsoft)

La aplicación de reproducción de página única realiza las siguientes acciones:

* Autenticación de usuario para los usuarios nativos del inquilino y usuarios invitados de otros inquilinos de AAD o cuentas de MSA. Los usuarios pueden elegir iniciar sesión mediante un elemento emergente del explorador o mediante redireccionamiento (para los exploradores que no permiten elementos emergentes, como Safari).
* Adquisición del `access_token` mediante el flujo de código de autorización con PKCE.
* Renovación del `access_token` (los tokens emitidos por AAD son válidos durante 1 hora), para lo que también se adquiere un `refresh_token`.
* Análisis de los tokens JWT (`access_token` y `id_token`) para pruebas e inspección.
* Adquisición de licencias de DRM para las tres claves de contenido DRM o AES-128.
* Streaming del contenido con varias combinaciones de DRM, protocolo de streaming y formato de contenedor. Se genera la cadena de formato correcta para cada combinación.
* Descifrado, descodificación y visualización.
* Llamadas a Microsoft Graph API para la solución de problemas. <!--See more details in the subsection Shortest path: testing my protected asset in my subscription with your hosted player app and underlying tenant. -->

Pantalla de inicio de sesión, adquisición de tokens, renovación de tokens y visualización de tokens:

 ![Pantalla para inicio de sesión, adquisición de tokens, renovación de tokens y visualización de tokens](media/aad-ams-content-protection/token-acquisition.png)

Pantalla para analizar tokens JWT (access_token o id_token):

![pantalla para analizar tokens JWT](media/aad-ams-content-protection/parsing-jwt-tokens.png)

La pantalla para probar el contenido protegido con diferentes combinaciones de DRM/AES, protocolo de streaming y formato de contenedor:

![Pantalla para analizar tokens JWT](media/aad-ams-content-protection/testing-protected-content.png)
-->

<!-- You can see a hosted version of the sample at [https://aka.ms/ott](https://aka.ms/ott)-->

## <a name="choose-an-azure-ad-tenant"></a>Elección de un inquilino de Azure AD

> [!NOTE]
> De aquí en delante, se supone que ha iniciado sesión en Azure Portal y tiene al menos un inquilino de Azure AD.

Elija un inquilino de Azure AD para usarlo en el ejemplo de un extremo a otro. Tiene dos opciones:

* Un inquilino de Azure AD existente. Cualquier suscripción de Azure debe tener un inquilino de Azure AD, pero varias suscripciones de Azure pueden usar un inquilino de Azure AD.
* Un nuevo inquilino de Azure AD que *no* se utiliza en ninguna suscripción de Azure. Si elige la opción del nuevo inquilino, la cuenta de Media Services y la aplicación de reproducción de ejemplo deben estar en una suscripción de Azure que use un inquilino de Azure AD independiente. Esto proporciona cierta flexibilidad. Por ejemplo, podría usar su propio inquilino de Azure AD, pero también la cuenta de Media Services del cliente en la suscripción de Azure del cliente.

## <a name="register-the-backend-resource-app"></a>Registro de la aplicación de recursos de back-end

1. Vaya al inquilino de Azure AD que ha elegido o creado.
1. Seleccione **Azure Active Directory** en el menú.
1. Seleccione **Registros de aplicaciones** en el menú.
1. Haga clic en **+ Nuevo registro**.
1. Asigne a la aplicación el nombre *LicenseDeliveryResource2* (donde 2 indica puntos de conexión v2 de AAD).
1. Seleccione **Cuentas de este directorio organizativo solo ([*nombreDelInquilino*]: inquilino único)** . Si desea habilitar el acceso a varios inquilinos, seleccione una de las otras opciones multiinquilino.
1. El **identificador URI de redirección** es opcional y se puede cambiar más adelante.
1. Haga clic en **Registrar**. Aparecerá la vista Registros de aplicaciones.
1. Seleccione **Manifiesto** en el menú. Aparecerá la vista Manifiesto.
1. Cambie el valor de `accessTokenAcceptedVersion` a *2* (sin comillas).
1. Cambie el valor de `groupMembershipClaims` a *"SecurityGroup"* (con comillas).
1. Haga clic en **Save**(Guardar).
1. Seleccione **Exponer una API** en el menú. Aparecerá la vista Agregar un ámbito. (Azure proporciona un identificador URI de identificador de aplicación, pero si desea cambiarlo, puede editarlo en el campo URI de id. de aplicación).
1. Haga clic en **Guardar y continuar**. La vista cambiará. Para cada uno de los valores de la columna Configuración de la tabla siguiente, escriba el valor en la columna Valor y, a continuación, haga clic en **Agregar ámbito**.

| Configuración | Value | Descripción |
| ------- | ----- | ----------- |
| Nombre del ámbito | *DRM.License.Delivery* | Cómo aparecerá el ámbito cuando se solicite el acceso a esta API y, en los tokens de acceso, cuándo se ha concedido el ámbito a una aplicación cliente. Debe ser único en esta aplicación. Un procedimiento recomendado es usar "recurso.operación.restricción" como patrón para generar el nombre. |
| ¿Quién puede dar el consentimiento? | *Administradores y usuarios* | Determina si los usuarios pueden dar su consentimiento a este ámbito en los directorios donde esté habilitado el consentimiento del usuario. |
| Nombre para mostrar del consentimiento del administrador | *Entrega de licencia DRM* | Nombre del ámbito que se mostrará en la pantalla de consentimiento cuando los administradores den su consentimiento a este ámbito. |
| Descripción del consentimiento del administrador** | *Ámbito de los recursos de back-end de entrega de licencias de DRM* | Descripción detallada del ámbito que se muestra cuando los administradores del inquilino expanden un ámbito en la pantalla de consentimiento. |
| Nombre para mostrar del consentimiento del usuario | *DRM.License.Delivery* | Nombre del ámbito que se mostrará en la pantalla de consentimiento cuando los usuarios den su consentimiento a este ámbito. |
| Descripción del consentimiento del usuario | *Ámbito de los recursos de back-end de entrega de licencias de DRM* | Esta es una descripción detallada del ámbito que se muestra cuando los usuarios expanden un ámbito en la pantalla de consentimiento. |
| State | *Enabled* | Determina si este ámbito está disponible para que los clientes lo soliciten. Establézcalo en "Deshabilitado" para los ámbitos que no desea que estén visibles para los clientes. Solo se pueden eliminar los ámbitos deshabilitados y se recomienda esperar al menos una semana después de que se haya deshabilitado un ámbito antes de eliminarlo para asegurarse de que ningún cliente todavía lo usa. |

## <a name="register-the-client-app"></a>Registro de la aplicación cliente

1. Vaya al inquilino de Azure AD que ha elegido o creado.
1. Seleccione **Azure Active Directory** en el menú.
1. Seleccione **Registros de aplicaciones** en el menú.
1. Haga clic en **+ Nuevo registro**.
1. Asigne un nombre a la aplicación cliente, por ejemplo, *AMS AAD Content Protection*.
1. Seleccione **Cuentas de este directorio organizativo solo ([*nombreDelInquilino*]: inquilino único)** . Si desea habilitar el acceso a varios inquilinos, seleccione una de las otras opciones multiinquilino.
1. El **identificador URI de redirección** es opcional y se puede cambiar más adelante.
1. Haga clic en **Registrar**.
1. En el menú, seleccione **Permisos de API**.
1. Haga clic en **+ Agregar un permiso**. Se abrirá la vista Solicitud de permisos de API.
1. Haga clic en la pestaña **Mi aplicación** y seleccione la aplicación *LicenseDeliveryResource2* que creó en la última sección.
1. Haga clic en la flecha de DRM y compruebe el permiso *DRM.License.Delivery*.
1. Haga clic en **Agregar permisos**. Se cerrará la vista Agregar permisos.
1. Seleccione **Manifiesto** en el menú. Aparecerá la vista Manifiesto.
1. Busque y agregue los siguientes pares de valores al atributo `replyUrlsWithType`:

   ```json
   "replyUrlsWithType": [
        {
            "url": "https://npmwebapp.azurewebsites.net/",
            "type": "SPA"
        },
        {
            "url": "http://localhost:3000/",
            "type": "SPA"
        }
    ],
   ```

    > [!NOTE]
    > En este momento, aún no tiene la dirección URL de la aplicación de reproducción.  Si ejecuta la aplicación desde el servidor web localhost, puede usar solo el par de valores de localhost. Una vez implementada la aplicación de reproducción, puede agregar la entrada aquí con la dirección URL implementada.  Si olvida hacerlo, verá un mensaje de error en el inicio de sesión de Azure AD.

1. Haga clic en **Save**(Guardar).
1. Por último, para comprobar que la configuración es correcta, seleccione **Autenticación**.  Aparecerá la vista Autenticación. La aplicación cliente aparecerá como una aplicación de página única (SPA), se mostrará el identificador URI de redirección y el tipo de concesión será Flujo de código de autorización con PKCE.

### <a name="set-up-the-media-services-account-content-key-policy-and-streaming-policies"></a>Configuración de la directiva de clave de contenido de la cuenta de Media Services y las directivas de streaming

**En esta sección se da por supuesto que es un desarrollador de .NET y está familiarizado con el uso de la API v3 de AMS.**

> [!NOTE]
> En el momento de la redacción de este documento, no se puede usar Azure Portal para la configuración de la directiva de clave de la cuenta de Media Services porque no se admite el uso de una clave de firma de token asimétrica con OpenID-Config.  La configuración debe admitir la sustitución de la clave de Azure AD porque el token emitido por Azure AD está firmado por una clave asimétrica y la clave se sustituye cada seis semanas. Por lo tanto, en este tutorial se usan .NET y la API v3 de AMS.

Se aplica la configuración de la [directiva de clave de contenido](content-key-policy-concept.md) y las [directivas de streaming](streaming-policy-concept.md) para DRM y AES-128.  Cambie el valor de `ContentKeyPolicyRestriction` en la directiva de clave de contenido.

A continuación se muestra el código de .NET para crear la restricción de la directiva de clave de contenido.

```dotnetcli
ContentKeyPolicyRestriction objContentKeyPolicyRestriction;

//use Azure Active Directory OpenId discovery document, supporting key rollover
objContentKeyPolicyRestriction = new ContentKeyPolicyTokenRestriction()
    {
        OpenIdConnectDiscoveryDocument = ConfigAccessor.AppSettings["ida_AADOpenIdDiscoveryDocument"]
    };

string audience = ConfigAccessor.AppSettings["ida_audience"];
string issuer   = ConfigAccessor.AppSettings["ida_issuer"];

ContentKeyPolicyTokenRestriction objContentKeyPolicyTokenRestriction = (ContentKeyPolicyTokenRestriction)objContentKeyPolicyRestriction;
objContentKeyPolicyTokenRestriction.Audience             = audience;
objContentKeyPolicyTokenRestriction.Issuer               = issuer;
objContentKeyPolicyTokenRestriction.RestrictionTokenType = ContentKeyPolicyRestrictionTokenType.Jwt;

objContentKeyPolicyRestriction = objContentKeyPolicyTokenRestriction;

return objContentKeyPolicyRestriction;
```

Cambie los valores de `ida_AADOpenIdDiscoveryDocument`, `ida_audience` y `ida_issuer` en el código anterior. Para buscar los valores de estos elementos en Azure Portal:

1. Seleccione el inquilino de AAD que usó anteriormente, haga clic en **Registros de aplicaciones** en el menú y, a continuación, haga clic en el vínculo **Puntos de conexión**.
1. Seleccione y copie el valor del campo **Documento de metadatos de OpenIdConnect** y péguelo en el código como el valor de `ida_AADOpenIdDiscoveryDocument`.
1. El valor de `ida_audience` es el identificador de aplicación (cliente) de la aplicación registrada *LicenseDeliveryResource2*.
1. El valor de `ida_issuer` es la dirección URL `https://login.microsoftonline.com/[tenant_id]/v2.0`. Reemplace [*tenant_id*] por el identificador del inquilino.

## <a name="set-up-the-sample-player-app"></a>Configuración de la aplicación de reproducción de ejemplo

Si todavía no lo ha hecho, clone o descargue la aplicación desde el repositorio de origen: [https://github.com/Azure-Samples/media-services-content-protection-azure-ad](https://github.com/Azure-Samples/media-services-content-protection-azure-ad).

Tiene dos opciones para configurar la aplicación de reproducción:

* Personalización mínima (solo se reemplazan algunos valores de cadena constantes como client_id, tenant_id y la dirección URL de streaming), pero debe usar Visual Studio Code y Node.js.
* Si prefiere usar otro IDE y una plataforma web como ASP.NET Core, puede colocar los archivos de la página web, los archivos JavaScript y el archivo CSS en el proyecto, ya que la propia aplicación de reproducción no usa ningún código del servidor.

### <a name="option-1"></a>Opción 1

1. Inicie Visual Studio Code.
1. Para abrir el proyecto, haga clic en Archivo -> Abrir carpeta y busque y seleccione la carpeta principal del archivo *package.json*.
1. Abra el archivo JavaScript *public/javascript/constants.js*.
1. Reemplace `OAUTH2_CONST.CLIENT_ID` por el valor de `client_id` de la aplicación cliente registrada en el inquilino de AAD.  Puede encontrar el valor de `client_id` en la sección Información general de la aplicación registrada en Azure Portal. Nota: Es el identificador de cliente, no el identificador de objeto.
1. Reemplace `OAUTH2_CONST.TENANT_ID` por el valor de `tenant_id` del inquilino de Azure AD. Para encontrar el valor de `tenant_id`, haga clic en el menú de Azure Active Directory. El valor de tenant_id aparecerá en la sección Información general.
1. Reemplace `OAUTH2_CONST.SCOPE` por el ámbito que agregó en la aplicación cliente registrada. Para encontrar el ámbito, vaya a la aplicación cliente registrada en el menú de **Registros de aplicaciones** y, a continuación, seleccione la aplicación cliente:
    1. Seleccione la aplicación cliente, haga clic en el menú **Permisos de API** y, a continuación, seleccione el ámbito *DRM.License.Delivery* en el permiso de API *LicenseDeliveryResource2*. El permiso debe tener un formato similar a *api://df4ed433-dbf0-4da6-b328-e1fe05786db5/DRM.License.Delivery*. **Importante**: Mantenga el espacio delante de `offline_access` en `OAUTH2_CONST.SCOPE`.
1. Reemplace las dos cadenas constantes por `AMS_CONST`, como se muestra a continuación. Una es la dirección URL de streaming protegida del recurso de prueba y la otra es la dirección URL del certificado de aplicación de FPS, si desea incluir el caso de prueba FairPlay. En caso contrario, puede dejarlo como está para `AMS_CONST.APP_CERT_URL`. A continuación, haga clic en **Guardar**.

```javascript
//defaults in ams.js
class AMS_CONST {
    static get DEFAULT_URL() {
        return "https://eventgridmediaservice-usw22.streaming.media.azure.net/9591e337-ae90-420e-be30-1da36c06665b/MicrosoftElite01.ism/manifest(format=mpd-time-csf,encryption=cenc)";
    }
    //FairPlay application cert URL
    static get APP_CERT_URL() {
     return `${window.location.href}cert/FPSAC.cer`;
    }
}
```

Para probar localmente:

1. En Visual Studio Code (VSC), seleccione **Ver** en el menú principal y, a continuación, seleccione **Terminal**.
1. Si aún no ha instalado npm, escriba `npm install` en el símbolo del sistema.
1. Escriba `npm start` en el símbolo del sistema. (Si no se inicia npm, intente cambiar el directorio a `npmweb`; para ello, escriba `cd npmweb` en el símbolo del sistema).
1. Use un explorador para ir a `http://localhost:3000`.

En función del explorador que use, elija la combinación correcta de DRM/AES, protocolo de streaming y formato de contenedor para probar después del inicio de sesión (adquisición de `access_token`). Si realiza la prueba en Safari en macOS, active la opción API de redireccionamiento, ya que Safari no permite elementos emergentes. La mayoría de los demás exploradores permiten tanto ventanas emergentes como opciones de redireccionamiento.

### <a name="option-2"></a>Opción 2

Si tiene previsto usar otro IDE o plataforma web o un servidor web, como IIS, que se ejecuta en el equipo de desarrollo, copie los siguientes archivos en un nuevo directorio del servidor web local. A continuación se muestran las rutas de acceso en las que se encuentran en el código descargado.

* *views/index.ejs* (cambie el sufijo a .html)
* *views/jwt.ejs* (cambie el sufijo a .html)
* *views/info.ejs* (cambie el sufijo a .html)
* *public/* * (archivos JavaScript, CSS e imágenes, como se muestra a continuación)

1. Copie los archivos que se encuentran en la carpeta *view* a la carpeta raíz del nuevo directorio.
1. Copie las *carpetas* que se encuentran en la carpeta *public* a la carpeta raíz del nuevo directorio.
1. Cambie las extensiones de los archivos `.ejs` a `.html`. (No se usa ninguna variable del servidor, por lo que puede cambiarlo de forma segura).
1. Abra *index.html* en VSC (u otro editor de código) y cambie las rutas de acceso de `<script>` y `<link>` para que reflejen dónde se encuentran los archivos.  Si siguió los pasos anteriores, solo tiene que eliminar la barra `\` en la ruta de acceso.  Por ejemplo, `<script type="text/javascript" src="/javascript/constants.js"></script>` se convierte en `<script type="text/javascript" src="javascript/constants.js"></script>`.
1. Personalice las constantes del archivo *javascript/constants.js*, como en la opción 1.

## <a name="common-customer-scenarios"></a>Escenarios de cliente comunes

Ahora que ha completado el tutorial y tiene un subsistema en funcionamiento, puede intentar modificarlo para los siguientes escenarios de cliente:

### <a name="role-based-access-control-rbac-for-license-delivery-via-azure-ad-group-membership"></a>Control de acceso basado en roles (RBAC) para la entrega de licencias mediante la pertenencia a grupos de Azure AD

Hasta ahora, el sistema permite obtener una licencia válida y reproducir el contenido protegido a cualquier usuario que pueda iniciar sesión.

Es un requisito habitual de los clientes que un subconjunto de usuarios autenticados pueda ver el contenido, mientras que otros no, por ejemplo, un cliente que ofrece suscripciones Básicas y Premium para su contenido de vídeo. Los clientes que pagan por una suscripción Básica no deben ser capaces de ver el contenido que requiere una suscripción Premium. A continuación se indican los pasos adicionales necesarios para cumplir este requisito:

#### <a name="set-up-the-azure-ad-tenant"></a>Configuración del inquilino de Azure AD

1. Configure dos cuentas en el inquilino. Se pueden llamar *premium_user* y *basic_user*.
1. Cree un grupo de usuarios y llámelo *PremiumGroup*.
1. Agregue a *premium_user* como miembro de *PremiumGroup*, pero no agregue al usuario *basic_user* al grupo.
1. Tome nota del **identificador de objeto** del grupo *PremiumGroup*.

#### <a name="set-up-the-media-services-account"></a>Configuración de la cuenta de Media Services

Modifique `ContentKeyPolicyRestriction` (como se muestra en la sección anterior en la configuración de la cuenta de Media Services); para ello, agregue una notificación llamada *groups*, en la que `ida_EntitledGroupObjectId` tiene el identificador de objeto de *PremiumGroup* como su valor:

```dotnetcli

var tokenClaims = new ContentKeyPolicyTokenClaim[] { new ContentKeyPolicyTokenClaim("groups", ConfigAccessor.AppSettings["ida_EntitledGroupObjectId"])
//, other claims, if any.
};

if (tokenClaims != null && tokenClaims.Length > 0)
{
     objContentKeyPolicyTokenRestriction.RequiredClaims = new List<ContentKeyPolicyTokenClaim>(tokenClaims);
}
```

La notificación *groups* es miembro de un [conjunto de notificaciones restringidas](../../active-directory/develop/active-directory-claims-mapping.md#claim-sets) de Azure AD.

#### <a name="test"></a>Prueba

1. Inicie sesión con la cuenta del usuario *premium_user*. Debe poder reproducir el contenido protegido.
1. Inicie sesión con la cuenta del usuario *basic_user*. Debería recibir un error que indica que el vídeo está cifrado, pero no hay ninguna clave para descifrarlo. Si observa los eventos, los errores y las descargas con la lista desplegable de la parte inferior de la capa de superposición de diagnóstico del reproductor, el mensaje de error debe indicar un error de adquisición de licencia debido a que falta el valor de una notificación para la notificación de grupos en el token JWT emitido por punto de conexión de tokens de Azure AD.

### <a name="supporting-multiple-media-service-accounts-across-multiple-subscriptions"></a>Compatibilidad con varias cuentas de Media Services (en varias suscripciones)

Un cliente puede tener varias cuentas de Media Services en una o varias suscripciones de Azure. Por ejemplo, un cliente puede tener una cuenta de Media Services como la principal de producción, otra como secundaria o de copia de seguridad y otra para desarrollo y pruebas.

Lo único que debe hacer es asegurarse de que se usa el mismo conjunto de parámetros que utilizó en la sección (Configuración de la cuenta de Media Services) para crear el elemento `ContentKeyPolicyRestriction` en todas las cuentas de Media Services.

### <a name="supporting-a-customer-its-vendors-andor-subsidiaries-across-multiple-aad-tenants"></a>Compatibilidad con un cliente, sus proveedores y sus subsidiarias en varios inquilinos de AAD

Como usuarios de la solución, los subsidiarios de un cliente, los proveedores y los asociados pueden residir en distintos inquilinos de AAD, como `mycustomer.com`, `mysubsidiary.com` y `myparther.com`. Aunque esta solución se crea en un único inquilino de AAD específico, como `mycustomer.com`, puede hacer que funcione para los usuarios de otros inquilinos.

Utilizando `mycustomer.com` para esta solución, agregue un usuario de `mypartner.com` como usuario invitado al inquilino de `mycustomer.com`. Asegúrese de que el usuario de `mypartner.com` activa la cuenta invitado. La cuenta de invitado puede ser de otro inquilino de AAD o una cuenta de `outlook.com`.

Observe que los usuarios invitados de `mypartner.com`, después de su activación en `mycustomer.com`, todavía se autentican con su propio inquilino de AAD original, `mypartner.com`, pero el token `access_token` lo emite `mycustomer.com`.

### <a name="supporting-a-customer-tenantsubscription-with-a-setup-in-your-subscriptiontenant"></a>Compatibilidad con una suscripción o un inquilino del cliente con una instalación en su suscripción o inquilino

Puede usar su instalación para probar el contenido protegido en la cuenta o la suscripción de Media Services de su cliente. Debe configurarlo con un inquilino de Azure AD y una cuenta de Media Services en la misma suscripción. La cuenta de Media Services del cliente se encontraría en la suscripción de Azure del cliente con el inquilino de Azure AD del cliente.

1. Agregue la cuenta de su cliente a su inquilino como una cuenta de invitado.
1. Colabore con su cliente para preparar el contenido protegido de la cuenta de Media Services de su cliente, para lo que debe proporcionarle los tres parámetros indicados en la sección Configuración de la cuenta de Media Services.

A continuación, el cliente puede ir a su instalación, iniciar sesión con la cuenta de invitado y probar su propio contenido protegido. También puede iniciar sesión con su propia cuenta y probar el contenido del cliente.

La solución de ejemplo se puede configurar en un inquilino de Microsoft con una suscripción de Microsoft o un inquilino personalizado con una suscripción de Microsoft. La instancia de Azure Media Services puede ser de otra suscripción con su propio inquilino.

## <a name="clean-up-resources"></a>Limpieza de recursos

> [!WARNING]
> Si no va a seguir usando esta aplicación, elimine los recursos que ha creado durante este tutorial. De lo contrario, se le cobrará por ellos.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Inicio rápido: Cifrado del contenido](encrypt-content-quickstart.md)
