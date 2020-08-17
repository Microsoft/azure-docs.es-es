---
title: Diseño de un sistema de protección de contenido con control de acceso mediante Azure Media Services | Microsoft Docs
description: Más información sobre cómo obtener licencias de Microsoft Smooth Streaming Client Porting Kit.
services: media-services
documentationcenter: ''
author: willzhan
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/14/2019
ms.author: willzhan
ms.reviewer: kilroyh;yanmf;juliako
ms.openlocfilehash: 254659c58b9830645211596da0095c33d70e8d95
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87072029"
---
# <a name="design-of-a-content-protection-system-with-access-control-using-azure-media-services"></a>Diseño de un sistema de protección de contenido con control de acceso mediante Azure Media Services 

## <a name="overview"></a>Información general

El diseño y la creación de un subsistema de administración de derechos digitales (DRM) para una solución OTT o de streaming en línea es una tarea compleja. Los operadores o proveedores de vídeo en línea suelen subcontratar esta tarea a proveedores de servicios DRM especializados. El objetivo de este documento es presentar un diseño de referencia y la implementación del subsistema DRM de un extremo a otro en una solución de streaming en línea o de OTT.

Este documento va dirigido a ingenieros que trabajan en subsistemas DRM de soluciones OTT o soluciones de varias pantallas o streaming en línea, o a cualquier lector interesado en subsistemas DRM. Se supone que los lectores están familiarizados con al menos una de las tecnologías DRM del mercado, como PlayReady, Widevine, FairPlay o Adobe Access.

En esta explicación de DRM, también incluimos el cifrado común (CENC) con varios DRM. Una tendencia importante en el streaming en línea y en la industria OTT es utilizar CENC con varias DRM nativas en diversas plataformas de cliente. Esta tendencia es un avance respecto a la anterior, que usa un único DRM y su cliente SDK para varias plataformas de cliente. Cuando se utiliza CENC con varias DRM nativas, tanto PlayReady como Widevine se cifran según la especificación [Cifrado común (ISO/IEC 23001-7 CENC)](https://www.iso.org/iso/home/store/catalogue_ics/catalogue_detail_ics.htm?csnumber=65271/).

Las ventajas de CENC con varias DRM son las siguientes:

* Reduce el costo del cifrado porque se emplea un solo procesamiento de cifrado que tiene como destino distintas plataformas con sus DRM nativas.
* Reduce el costo de administrar los recursos cifrados porque solo se necesita una copia de ellos.
* Elimina el costo de licencia de clientes DRM porque el cliente DRM nativo suele ser gratis en su plataforma nativa.

Microsoft es un promotor activo de DASH y CENC junto con algunos de los principales reproductores del sector. Azure Media Services proporciona compatibilidad para DASH y CENC. Para anuncios recientes, consulte los blogs siguientes:

*  [Anuncio de los servicios de entrega de licencias de Google Widevine en Azure Media Services](https://azure.microsoft.com/blog/announcing-general-availability-of-google-widevine-license-services/)
* [Azure Media Services agrega empaquetado de Google Widevine para entregar un proceso de streaming de varios DRM](https://azure.microsoft.com/blog/azure-media-services-adds-google-widevine-packaging-for-delivering-multi-drm-stream/)  

### <a name="goals-of-the-article"></a>Objetivos del artículo

Los objetivos de este artículo son los siguientes:

* Proporcionar un diseño de referencia de un subsistema DRM que usa CENC con varias DRM.
* Proporcionar una implementación de referencia en una plataforma de Azure o Media Services.
* Describir algunos temas de diseño e implementación.

En este artículo, el término "varias DRM" abarca los productos siguientes:

* Microsoft PlayReady
* Google Widevine
* Apple FairPlay 

En la tabla siguiente se resume la aplicación o plataforma nativa y los exploradores que admite cada DRM.

| **Plataforma de cliente** | **Compatibilidad con DRM nativa** | **Explorador o aplicación** | **Formatos de streaming** |
| --- | --- | --- | --- |
| **Televisores inteligentes, operador STB, OTT STB** |PlayReady principalmente o Widevine u otros |Linux, Opera y WebKit, otros |Varios formatos |
| **Dispositivos Windows 10 (Windows PC, tabletas de Windows, Windows Phone, Xbox)** |PlayReady |Microsoft Edge/IE11/EME<br/><br/><br/>Plataforma universal de Windows |DASH (no se admite PlayReady para HLS)<br/><br/>DASH, Smooth Streaming (no se admite PlayReady para HLS) |
| **Dispositivos Android (teléfono, tableta, TV)** |Widevine |Chrome/EME |DASH, HLS |
| **iOS (iPhone, iPad), clientes de OS X y Apple TV** |FairPlay |Safari 8+/EME |HLS |

Teniendo en cuenta el estado actual de implementación para cada DRM, normalmente el servicio quiere implementar dos o tres DRM para asegurarse de que aborda todos los tipos de puntos de conexión de la mejor manera.

Hay un equilibrio entre la complejidad de la lógica del servicio y la complejidad del cliente para alcanzar un determinado nivel de experiencia del usuario en los distintos clientes.

Para realizar la selección, tenga en cuenta lo siguiente:

* PlayReady se implementa de forma nativa en cada dispositivo de Windows, en algunos dispositivos Android y está disponibles mediante los SDK de software en prácticamente cualquier plataforma.
* Widevine se implementa de forma nativa en todos los dispositivos Android, en Chrome y algunos otros dispositivos.
* FairPlay solo está disponible en clientes iOS y Mac OS o a través de iTunes.

Hay dos opciones para un entorno con DRM múltiple típica:

* PlayReady y Widevine
* PlayReady, Widevine y FairPlay

## <a name="a-reference-design"></a>Un diseño de referencia
En esta sección se presenta un diseño de referencia que es independiente de las tecnologías usadas para implementarlo.

Un subsistema DRM puede contener los siguientes componentes:

* Administración de claves
* Empaquetado de DRM
* Entrega de licencias DRM
* Comprobación de derechos
* Autenticación y autorización
* Reproductor
* Origen/red de entrega de contenido (CDN)

En el siguiente diagrama ilustra la interacción de alto nivel entre los componentes de un subsistema DRM:

![Subsistema DRM con CENC](./media/media-services-cenc-with-multidrm-access-control/media-services-generic-drm-subsystem-with-cenc.png)

El diseño tiene tres capas básicas:

* La capa de área de operaciones (en negro) no se expone externamente.
* La capa de red perimetral (azul oscuro) contiene todos los puntos de conexión de cara al público.
* La capa de Internet pública (azul claro) contiene la red CDN y los reproductores con el tráfico que atraviesa la Internet pública.

También debe haber una herramienta de administración del contenido para controlar la protección DRM, con independencia de que el cifrado es estático o dinámico. Las entradas del cifrado de DRM incluyen:

* Contenido de vídeo MBR
* Clave de contenido
* Las direcciones URL de adquisición de licencias

Este es el flujo de alto nivel durante el tiempo de reproducción:

* El usuario está autenticado.
* Se crea un token de autorización para el usuario.
* El contenido protegido con DRM (manifiesto) se descarga en el reproductor.
* El reproductor envía la solicitud de adquisición de licencias a los servidores de licencias junto con el identificador de clave y el token de autorización.

En la siguiente sección se describe el diseño de la administración de claves.

| **ContentKey-to-asset** | **Escenario** |
| --- | --- |
| 1 a 1 |El caso más simple. Proporciona el control más específico. Sin embargo, esta disposición generalmente resulta en un costo más alto de entrega de licencias. Se requiere al menos una solicitud de licencia para cada recurso protegido. |
| 1 a varios |Puede utilizar la misma clave de contenido para varios activos. Por ejemplo, para todos los recursos de un grupo lógico, como un género o un subconjunto del género (o género de película), puede utilizar una única clave de contenido. |
| Varios a 1 |Se necesitan varias claves de contenido para cada activo. <br/><br/>Por ejemplo, si tiene que aplicar protección CENC dinámica con varias DRM para MPEG-DASH y cifrado AES-128 para HLS, necesitará dos claves de contenido distintas. Cada clave de contenido tiene su propio valor ContentKeyType. (Para la clave de contenido que se usa para la protección de CENC dinámica, utilice ContentKeyType.CommonEncryption. Para la clave de contenido que se usa para el cifrado AES-128 dinámico, utilice ContentKeyType.EnvelopeEncryption).<br/><br/>Como otro ejemplo, en la protección CENC de contenido DASH, en teoría, se puede usar una clave de contenido para proteger la transmisión de vídeo y otra para proteger la transmisión de audio. |
| Varios a varios |Combinación de los dos escenarios anteriores. Se utiliza un conjunto de claves de contenido para cada uno de los diversos recursos del mismo "grupo" de recursos. |

Otro factor importante a tener en cuenta es el uso de licencias persistentes y no persistentes.

¿Por qué son importantes estas consideraciones?

Si usa una nube pública para la entrega de licencias, las licencias persistentes y no persistentes tendrán un impacto directo en el costo de entrega de licencias. Los dos casos siguientes de distinto diseño sirven para mostrar lo siguiente:

* Suscripción mensual: Se usa una licencia persistente y la asignación de 1 a varios entre claves de contenido y recursos. Por ejemplo, para todas las películas de niños, usamos una única clave de contenido para el cifrado. En este caso:

    Número total de licencias solicitadas para todas las películas o dispositivos de niños = 1

* Suscripción mensual: Se usa una licencia no persistente y la asignación de 1 a 1 entre claves de contenido y recursos. En este caso:

    El número total de licencias solicitadas para todas las películas o dispositivos de niños = [numero de películas vistas] x [número de sesiones]

Los dos diseños diferentes dan como resultado patrones de solicitud de licencias muy diferentes. Los patrones diferentes dan como resultado patrones un costo de entrega de las licencias distinto si el servicio de entrega de licencias se proporciona mediante una nube pública como Media Services.

## <a name="map-design-to-technology-for-implementation"></a>Asignación del diseño a la tecnología para la implementación
A continuación, el diseño genérico se asigna a las tecnologías de la plataforma Azure o Media Services mediante la especificación de la tecnología que queremos usar en cada bloque de creación.

En la tabla siguiente se muestra la asignación.

| **Bloque de creación** | **Technology** |
| --- | --- |
| **Reproductor** |[Azure Media Player](https://azure.microsoft.com/services/media-services/media-player/) |
| **Proveedor de identidades (IDP)** |Azure Active Directory (Azure AD) |
| **Servicio de token de seguridad (STS)** |Azure AD |
| **Flujo de trabajo de protección DRM** |Protección dinámica de Media Services |
| **Entrega de licencia DRM** |* Entrega de licencias de Media Services (PlayReady, Widevine, FairPlay) <br/>* Servidor de licencias de Axinom <br/>* Servidor de licencias personalizado de PlayReady |
| **Origen** |Punto de conexión de streaming de Media Services |
| **Administración de claves** |No es necesaria para la implementación de referencia. |
| **Administración de contenido** |Una aplicación de consola de C#. |

En otras palabras, se utilizan tanto IDP como STS con Azure AD. La [API de Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/) se usa para el reproductor. Tanto Media Services como Media Player admiten DASH y CENC con varias DRM.

En el diagrama siguiente se muestra la estructura y el flujo generales con la asignación de tecnología anterior:

![CENC en Media Services](./media/media-services-cenc-with-multidrm-access-control/media-services-cenc-subsystem-on-AMS-platform.png)

Para configurar el cifrado CENC dinámico, la herramienta de administración de contenido usa las siguientes entradas:

* Contenido abierto
* Clave de contenido de la generación o administración de claves
* Las direcciones URL de adquisición de licencias
* Una lista de información de Azure AD

Esta es la salida de la herramienta de administración de contenido:

* ContentKeyAuthorizationPolicy contiene la especificación de cómo la entrega de licencias comprueba el estándar JSON Web Token(JWT) y las especificaciones de licencias de DRM.
* AssetDeliveryPolicy contiene las especificaciones de formato de streaming, protección DRM y direcciones URL de adquisición de licencias.

Este es el flujo en tiempo de ejecución:

* Tras la autenticación del usuario, se genera un token JWT.
* Una de las notificaciones contenidas en el token JWT es la notificación de grupos que contiene el identificador de objeto de grupo de EntitledUserGroup. Esta notificación se utiliza para pasar la comprobación de derechos.
* El reproductor descarga el manifiesto de cliente del contenido protegido por CENC de identifica lo siguiente:
   * El identificador de la clave.
   * El contenido está protegido por CENC.
   * Las direcciones URL de adquisición de licencias.
* El reproductor realiza una solicitud de adquisición de licencias basada en el explorador o DRM admitidas. En la solicitud de adquisición de licencias, también se envían el identificador de clave y un token JWT. El servicio de entrega de licencias comprobará el token JWT y las notificaciones contenidas antes de emitir la licencia necesaria.

## <a name="implementation"></a>Implementación
### <a name="implementation-procedures"></a>Procedimientos de implementación
La implementación incluye los siguientes pasos:

1. Preparar los recursos de prueba. Codifique o empaquete un vídeo de prueba en MP4 fragmentado de varias velocidades de bits en Media Services. Este recurso *no* está protegido con DRM. La protección DRM se realiza posteriormente mediante protección dinámica.

2. Cree el identificador de clave y una clave de contenido (opcionalmente a partir de la inicialización de clave). En este caso, no es necesario el sistema de administración de claves porque solo se requiere un identificador de clave y una clave de contenido para un par de recursos de prueba.

3. Utilice la API de Media Services para configurar servicios de entrega de licencias de varias DRM para el recurso de prueba. Si utiliza servidores de licencias personalizados de su empresa o de los proveedores de su empresa en lugar de servicios de licencias en Media Services, puede omitir este paso. Puede especificar las direcciones URL de adquisición de licencias en el paso al configurar la entrega de licencias. Las API de Media Services son necesarias para especificar algunas configuraciones detalladas, como restricción de directivas de autorización y plantillas de respuesta de licencia para los distintos servicios de licencias DRM. En este momento, Azure Portal no proporciona la interfaz de usuario necesaria para esta configuración. Para más información sobre el nivel de API y para ver un código de ejemplo, consulte [Uso del cifrado dinámico común de PlayReady o Widevine](media-services-protect-with-playready-widevine.md).

4. Utilice la API de Media Services para configurar la directiva de entrega de recursos para el recurso de prueba. Para más información sobre el nivel de API y para ver un código de ejemplo, consulte [Uso del cifrado dinámico común de PlayReady o Widevine](media-services-protect-with-playready-widevine.md).

5. Cree y configure un inquilino de Azure AD en Azure.

6. Cree algunas cuentas de usuario y grupos en su inquilino de Azure AD. Cree al menos un grupo de usuarios autorizados y agregue un usuario a este grupo. Los usuarios de este grupo pasan la comprobación de derechos en la adquisición de licencias. Los usuarios que no pertenezcan a este grupo no pasan la comprobación de autenticación y no pueden adquirir una licencia. La pertenencia a este grupo "EntitledUser" es una notificación de grupos necesaria en el token JWT emitido por Azure AD. Especifique este requisito de notificación en el paso al configurar los servicios de entrega de licencias de varias DRM.

7. Cree una aplicación de ASP.NET MVC para hospedar el reproductor de vídeo. Esta aplicación ASP.NET está protegida con autenticación de usuario con respecto al inquilino de Azure AD. Se incluyen las notificaciones adecuadas en los tokens de acceso obtenidos después de la autenticación de usuario. Para este paso se recomienda la API de OpenID Connect. Instale los siguientes paquetes NuGet:

   * Install-Package Microsoft.Azure.ActiveDirectory.GraphClient
   * Install-Package Microsoft.Owin.Security.OpenIdConnect
   * Install-Package Microsoft.Owin.Security.Cookies
   * Install-Package Microsoft.Owin.Host.SystemWeb
   * Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory

8. Cree un reproductor mediante la [API de Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/). Utilice la [API ProtectionInfo de Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/) para especificar la tecnología DRM que se usará en distintas plataformas DRM.

9. En la tabla siguiente se muestra la matriz de prueba.

    | **DRM** | **Browser** | **Resultado para el usuario autorizado** | **Resultado para el usuario no autorizado** |
    | --- | --- | --- | --- |
    | **PlayReady** |Microsoft Edge o Internet Explorer 11 en Windows 10 |Correcto |Incorrecto |
    | **Widevine** |Chrome, Firefox, Opera |Correcto |Incorrecto |
    | **FairPlay** |Safari en macOS      |Correcto |Incorrecto |
    | **AES-128** |Exploradores más modernos  |Correcto |Incorrecto |

Para más información sobre cómo configurar Azure AD para una aplicación de reproductor de ASP.NET MVC, consulte [Integración de la aplicación OWIN basada en MVC de Azure Media Services con Azure Active Directory y restricción de la entrega de claves de contenido basada en notificaciones de JWT](http://gtrifonov.com/2015/01/24/mvc-owin-azure-media-services-ad-integration/).

Para más información, consulte [Autenticación de token JWD en Azure Media Services y cifrado dinámico de Azure](http://gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/).  

Para más información sobre Azure AD:

* Puede encontrar información para desarrolladores en la [Guía para desarrolladores de Azure Active Directory](../../active-directory/azuread-dev/v1-overview.md).
* Puede encontrar información para administradores en [Administración del directorio de Azure AD](../../active-directory/fundamentals/active-directory-whatis.md).

### <a name="some-issues-in-implementation"></a>Algunos problemas de implementación
Utilice la siguiente información para solución de problemas para obtener ayuda con problemas de implementación.

* La dirección URL del emisor debe terminar por "/". La audiencia debe ser el identificador de cliente de la aplicación de reproductor. Además, agregue "/" al final de la dirección URL del emisor.

    ```xml
    <add key="ida:audience" value="[Application Client ID GUID]" />
    <add key="ida:issuer" value="https://sts.windows.net/[AAD Tenant ID]/" />
    ```

    En [JWT Decoder](http://jwt.calebb.net/), debería ver **aud** y **iss**, como se muestra a continuación en el token JWT:

    ![JWT](./media/media-services-cenc-with-multidrm-access-control/media-services-1st-gotcha.png)

* Agregue permisos a la aplicación en Azure AD en la pestaña **Configurar** de la aplicación. Los permisos son necesarios para cada aplicación, tanto para las versiones locales como implementadas.

    ![Permisos](./media/media-services-cenc-with-multidrm-access-control/media-services-perms-to-other-apps.png)

* Utilice al emisor correcto cuando configure la protección de CENC dinámica.

    ```xml
    <add key="ida:issuer" value="https://sts.windows.net/[AAD Tenant ID]/"/>
    ```

    Lo siguiente no funciona:

    ```xml
    <add key="ida:issuer" value="https://willzhanad.onmicrosoft.com/" />
    ```

    El GUID es el identificador del inquilino de Azure AD. El GUID puede encontrarse en el menú emergente **Puntos de conexión** en Azure Portal.

* Otorgue privilegios de notificaciones de pertenencia a grupo. Asegúrese de que lo siguiente se encuentra en el archivo de manifiesto de aplicación de Azure AD: 

    "groupMembershipClaims": "All"    (el valor predeterminado es null)

* Establezca el valor adecuado de TokenType al crear los requisitos de restricción.

    ```csharp
    objTokenRestrictionTemplate.TokenType = TokenType.JWT;
    ```

    Como agregó compatibilidad con el token JWT (Azure AD), además de con SWT (ACS), el valor predeterminado de TokenType es TokenType.JWT. Si utiliza SWT y ACS, debe establecer el token en TokenType.SWT.

## <a name="additional-topics-for-implementation"></a>Temas adicionales para la implementación
En esta sección se van a tratar algunos temas adicionales sobre el diseño e implementación.

### <a name="http-or-https"></a>¿HTTP o HTTPS?
La aplicación de reproductor de ASP.NET MVC debe ser compatible con lo siguiente:

* Autenticación de usuarios mediante Azure AD, que está en HTTPS.
* Intercambio de tokens JWT entre el cliente y Azure AD, que debe estar en HTTPS.
* La adquisición de licencias DRM por el cliente, que debe estar en HTTPS si Media Services proporciona la entrega de licencias. El conjunto de productos de PlayReady no impone HTTPS para la entrega de licencias. Si el servidor de licencias de PlayReady está fuera de Media Services, se podría usar HTTP o HTTPS.

La aplicación de reproductor de ASP. NET utiliza HTTPS como procedimiento recomendado, por lo que Media Player se encuentra en una página en HTTPS. Sin embargo, HTTP es preferible para el streaming, por lo que se debe tener en cuenta la cuestión del contenido mixto.

* El explorador no permite contenido mixto. Sin embargo, los complementos como Silverlight y OSMF para Smooth y DASH sí lo hacen. El contenido mixto es un problema de seguridad debido a la amenaza de la posibilidad de insertar JavaScript malintencionado, que puede poner en riesgo los datos del cliente. Los exploradores lo bloquean de forma predeterminada. La única manera de solucionar esto es en el servidor (origen) al permitir todos los dominios (con independencia de si son HTTPS o HTTP). Probablemente esto tampoco sea una buena idea.
* Evite el contenido mixto. Tanto la aplicación de reproducción y Media Player deben utilizar HTTP o HTTPS. Al reproducir contenido mixto, silverlightSS tech requiere que se borre una advertencia de contenido mixto. flashSS tech controla el contenido mixto sin advertencia de contenido mixto.
* Si su punto de conexión de streaming se ha creado antes de agosto de 2014, no admitirá HTTPS. En este caso, cree y utilice un nuevo punto de conexión de streaming para HTTPS.

En la implementación de referencia, para el contenido protegido con DRM, tanto la aplicación como el proceso de streaming estarán en HTTPS. Para el contenido abierto, el reproductor no necesita autenticación ni licencia, por lo que puede utilizar HTTP o HTTPS.

### <a name="azure-active-directory-signing-key-rollover"></a>Sustitución de claves de firma de Azure Active Directory
La sustitución de claves de firmas es un punto importante de su implementación a tener en cuenta. Si lo ignora, el sistema terminado finalmente deja de funcionar por completo, en un plazo máximo de seis semanas.

Azure AD utiliza el estándar del sector para establecer la confianza entre sí mismo y las aplicaciones que usan Azure AD. En concreto, Azure AD utiliza una clave de firma que consta de un par de claves pública y privada. Cuando Azure AD crea un token de seguridad que contiene información sobre el usuario, Azure AD lo firma con una clave privada antes de enviarlo a la aplicación. Para comprobar que el token es válido y tiene su origen en Azure AD, la solicitud debe validar la firma del token. La aplicación utiliza la clave pública expuesta por Azure AD que se encuentra en el documento de metadatos de federación del inquilino. Esta clave pública, y la clave de firma de la que se deriva, es la misma que se utiliza en todos los inquilinos de Azure AD.

Para más información sobre la sustitución de claves de Azure AD, consulte [Sustitución de claves de firma de Azure Active Directory](../../active-directory/develop/active-directory-signing-key-rollover.md).

Entre el [par de claves pública y privada](https://login.microsoftonline.com/common/discovery/keys/):

* Azure AD utiliza la clave privada para generar un token JWT.
* La clave pública se utiliza en una aplicación, como los servicios de entrega de licencias de DRM en Media Services, para comprobar el token JWT.

Por motivos de seguridad, Azure AD rota el certificado de forma periódica (cada seis semanas). En caso de infracciones de seguridad, la sustitución de claves se produce en cualquier momento. Por lo tanto, los servicios de entrega de licencias en Media Services tienen que actualizar la clave pública utilizada cuando Azure AD rota el par de claves. En caso contrario, se produce un error en la autenticación de tokens en Media Services y no se emite ninguna licencia.

Para configurar este servicio, se establece TokenRestrictionTemplate.OpenIdConnectDiscoveryDocument al configurar los servicios de entrega de licencias de DRM.

Este es el flujo del token JWT:

* Azure AD emite el token JWT con la clave privada actual de un usuario autenticado.
* Cuando un reproductor ve un CENC con contenido protegido por varias DRM, primero busca el token JWT que ha emitido Azure AD.
* El reproductor envía la solicitud de adquisición de licencias con el token JWT a los servicios de entrega de licencias de Media Services.
* Los servicios de entrega de licencias de Media Services utilizan la clave pública actual o válida de Azure AD para comprobar el token JWT antes de emitir las licencias.

Los servicios de entrega de licencias de DRM siempre estarán buscando la clave pública actual o válida de Azure AD. La clave pública presentada por Azure AD es la clave utilizada para comprobar un token JWT emitido por Azure AD.

¿Y si la sustitución de claves ocurre después de que Azure AD genere un token JWT pero antes de que los reproductores envíen el token JWT a los servicios de entrega de licencias de DRM en Media Services para que lo comprueben?

Dado que una clave se puede sustituir en cualquier momento, siempre hay más de una clave pública válida disponible en el documento de metadatos de federación. La entrega de licencias de Media Services puede utilizar cualquiera de las claves especificadas en el documento. Como una clave se puede sustituir pronto, otra podría ser su sustituta, y así sucesivamente.

### <a name="where-is-the-access-token"></a>¿Dónde está el token de acceso?
Si observa cómo una aplicación web llama a una aplicación de API en la sección [Identidad de aplicación con concesión de credenciales de cliente OAuth 2.0 ](../../active-directory/azuread-dev/web-api.md), el flujo de autenticación tiene lugar como se indica a continuación:

* Un usuario inicia sesión en Azure AD en la aplicación web. Para más información, consulte la sección [Aplicación web a API web](../../active-directory/azuread-dev/web-app.md).
* El punto de conexión de autorización de Azure AD redirige al agente de usuario a la aplicación cliente con un código de autorización. El agente de usuario devuelve el código de autorización al URI de redireccionamiento de la aplicación cliente.
* La aplicación web necesita adquirir un token de acceso para poder autenticarse ante la API web y recuperar el recurso deseado. Realiza una solicitud al punto de conexión de token de Azure AD y proporciona las credenciales, el identificador del cliente y el URI del identificador de aplicación de la API web. Presenta el código de autorización para demostrar que el usuario ha dado su consentimiento.
* Azure AD autentica la aplicación y devuelve un token de acceso de JWT que se usa para llamar a la API web.
* A través de HTTPS, la aplicación web usa el token de acceso de JWT devuelto para agregar la cadena JWT con una designación Bearer en el encabezado Authorization de la solicitud a la API web. La API web valida entonces el token JWT. Si la validación es correcta, devuelve el recurso deseado.

En este flujo de identidad de aplicación, la API web confía en que la aplicación web ha autenticado al usuario. Por ello, este patrón se conoce como subsistema de confianza. El [diagrama de flujo de autorización](../../active-directory/azuread-dev/v1-protocols-oauth-code.md) describe cómo funciona el flujo de concesión del código de autorización.

La adquisición de licencias con restricción de token sigue el mismo patrón de subsistema de confianza. El servicio de entrega de licencias de Media Services es el recurso de API web o el recurso de back-end al que una aplicación web necesita acceso. Entonces, ¿dónde está el token de acceso?

El token de acceso se obtiene de Azure AD. Después de autenticar al usuario correctamente, se devuelve un código de autorización. El código de autorización se utiliza entonces junto con el identificador de cliente y la clave de aplicación para intercambiarlos por el token de acceso. El token de acceso se utiliza para acceder a una aplicación "puntero" que señala o representa el servicio de entrega de licencias de Media Services.

Para registrar y configurar la aplicación puntero en Azure AD, siga estos pasos:

1. En el inquilino de Azure AD:

   * Agregue una aplicación (recurso) con la dirección URL de inicio de sesión https://[resource_name].azurewebsites.net/. 
   * Agregue un identificador de aplicación con la URL https://[aad_tenant_name].onmicrosoft.com/[resource_name].

2. Agregue una nueva clave para la aplicación de recursos.

3. Actualice el archivo de manifiesto de la aplicación para que la propiedad groupMembershipClaims tenga el valor "groupMembershipClaims": "All".

4. En la aplicación de Azure AD que apunta a la aplicación web de reproductor, en la sección **Permisos para otras aplicaciones**, agregue la aplicación de recursos que se agregó en el paso 1 anteriormente. En **Permisos delegados**, active seleccione **Acceso [nombre_de_recurso]** . Esta opción da permiso a la aplicación web para crear tokens de acceso que acceden a la aplicación de recursos. Haga esto para la versión local y la implementada de la aplicación web si desarrolla con Visual Studio y la aplicación web de Azure.

El token JWT emitido por Azure AD es el token de acceso que se utiliza para acceder al recurso de puntero.

### <a name="what-about-live-streaming"></a>¿Y qué pasa con el streaming en vivo?
La explicación anterior se centra en los recursos a petición. ¿Y qué pasa con el streaming en vivo?

Puede usar exactamente el mismo diseño y la misma implementación para proteger el streaming en vivo en Media Services, solo hay que tratar el recurso asociado a un programa como un recurso VoD.

En concreto, para hacer streaming en vivo en Media Services, debe crear un canal y, luego, crear un programa bajo él. Para crear el programa, debe crear un recurso que contiene el archivo en vivo del programa. Para proporcionar a CENC protección de varias DRM del contenido en directo, aplique la misma configuración o procesamiento al recurso como si fuera un recurso VoD, antes de iniciar el programa.

### <a name="what-about-license-servers-outside-media-services"></a>¿Y qué sucede con los servidores de licencias que están fuera de Media Services?
Con frecuencia, los clientes invierten en granjas de servidores de licencias en su propio centro de datos o en uno hospedado por proveedores de servicios DRM. Con Content Protection de Media Services, puede funcionar en modo híbrido. El contenido puede alojarse y protegerse dinámicamente en Media Services, mientras que las licencias DRM las entregan servidores externos a Media Services. En este caso, tenga en cuenta los siguientes cambios:

* El servicio de token de seguridad debe emitir tokens que sean aceptables y que se puedan comprobar en la granja de servidores de licencias. Por ejemplo, los servidores de licencias de Widevine proporcionados por Axinom requieren un token JWT específico que contenga un mensaje de derechos. Por lo tanto, debe tener un servicio de token de seguridad para emitir dicho token JWT. Para más información sobre este tipo de implementación, vaya al [centro de documentación de Azure](https://azure.microsoft.com/documentation/) y consulte [Uso de Axinom para entregar licencias de Widevine a Azure Media Services](media-services-axinom-integration.md).
* Ya no necesitará configurar el servicio de entrega de licencias (ContentKeyAuthorizationPolicy) en Media Services. Debe proporcionar las direcciones URL de adquisición de licencias (para PlayReady, Widevine y FairPlay) cuando configura AssetDeliveryPolicy en la configuración de CENC con varias DRM.

### <a name="what-if-i-want-to-use-a-custom-sts"></a>¿Y si quiero usar un STS personalizado?
Un cliente puede optar por usar a un servicio de token de seguridad personalizado para proporcionar tokens JWT. Los motivos incluyen los siguientes:

* El punto de distribución de emisión que utiliza el cliente no es compatible con el servicio de token de seguridad. En este caso, un servicio de token de seguridad personalizado podría ser una opción.
* El cliente puede necesitar un control más flexible o más estricto para integrar el servicio de token de seguridad con el sistema de facturación de los suscriptores del cliente. Por ejemplo, un operador MVPD puede ofrecer varios paquetes de suscriptor OTT, por ejemplo, premium, básico y deportes. El operador puede querer hacer coincidir las notificaciones de un token con el paquete de un suscriptor de forma que solo el contenido de un paquete específico esté disponible. En este caso, un STS personalizado proporciona la flexibilidad y el control necesarios.

Cuando se utiliza un servicio de token de seguridad personalizado, se deben realizar dos cambios:

* Al configurar el servicio de entrega de licencias para un recurso, debe especificar la clave de seguridad utilizada para la comprobación por el servicio de token de seguridad personalizado en lugar de la clave actual de Azure AD. (Más detalles a continuación). 
* Cuando se genera un token JTW, se especifica una clave de seguridad en lugar de la clave privada del certificado X509 actual en Azure AD.

Hay dos tipos de claves de seguridad:

* Clave simétrica: La misma clave se usa para generar y verificar un token JWT.
* Clave asimétrica: Se usa un par de claves público-privadas de un certificado X509 junto con la clave privada para cifrar o generar un token JWT y con la clave pública para verificar el token.

> [!NOTE]
> Si utiliza .NET Framework o C# como plataforma de desarrollo, el certificado X509 usado en una clave de seguridad asimétrica debe tener una longitud de clave de al menos 2048. Se trata de un requisito de la clase System.IdentityModel.Tokens.X509AsymmetricSecurityKey en .NET Framework. De lo contrario, se produce la siguiente excepción:
> 
> IDX10630: El valor 'System.IdentityModel.Tokens.X509AsymmetricSecurityKey' para la firma no puede ser menor que 2048 bits.

## <a name="the-completed-system-and-test"></a>Finalización del sistema y prueba
En esta sección permite recorrer los escenarios siguientes en el sistema de un extremo a otro completado para que se pueda tener una idea general del comportamiento antes de obtener una cuenta de inicio de sesión:

* Si necesita un escenario no integrado:

    * Para los recursos de vídeo hospedados en Media Services que no están protegidos o protegidos por DRM pero sin autenticación de token (mediante una emisión de una licencia a quienquiera que la haya solicitado), puede probarlo sin iniciar sesión. Cambie a HTTP si el streaming de vídeo se realiza a través de HTTP.

* Si necesita un escenario integrado de un extremo a otro:

    * Para los recursos de vídeo bajo protección dinámica de DRM en Media Services, con la autenticación de token y el token JWT generados por Azure AD, debe iniciar sesión.

Para más información sobre la aplicación web del reproductor y su inicio de sesión, consulte [este sitio web](https://openidconnectweb.azurewebsites.net/).

### <a name="user-sign-in"></a>Inicio de sesión de usuario
Para probar el sistema DRM integrado de extremo a extremo, debe tener una cuenta creada o agregada.

¿Qué cuenta?

Aunque Azure inicialmente permitía el acceso únicamente a usuarios de cuentas Microsoft, ahora se permite el acceso a usuarios de ambos sistemas. Todas las propiedades de Azure ahora confían en Azure AD para la autenticación, y Azure AD autentica a los usuarios de la organización. Se ha creado una relación de federación en la que Azure AD confía en el sistema de identidad del consumidor de cuentas Microsoft para autenticar a los usuarios consumidores. Como resultado, Azure AD puede autenticar cuentas Microsoft de invitado, así como cuentas de Azure AD nativas.

Como Azure AD confía en el dominio de la cuenta Microsoft, puede agregar cuentas de cualquiera de los siguientes dominios al inquilino de Azure AD personalizado y usar la cuenta para iniciar sesión:

| **Nombre de dominio** | **Dominio** |
| --- | --- |
| **Dominio de inquilino de Azure AD personalizado** |somename.onmicrosoft.com |
| **Dominio corporativo** |microsoft.com |
| **Dominio de cuentas Microsoft** |outlook.com, live.com, hotmail.com |

Puede ponerse en contacto con cualquiera de los autores para que le creen o le agreguen una cuenta.

Las capturas de pantalla siguientes muestran diferentes páginas de inicio de sesión que se usan con distintas cuentas de dominio:

**Cuenta de dominio de inquilino de Azure AD personalizado**: Página de inicio de sesión personalizada del dominio de inquilino de Azure AD personalizado.

![Cuenta de dominio de inquilino de Azure AD personalizado](./media/media-services-cenc-with-multidrm-access-control/media-services-ad-tenant-domain1.png)

**Cuenta de dominio de Microsoft con tarjeta inteligente**: Página de inicio de sesión personalizada por el responsable de TI corporativo de Microsoft con autenticación en dos fases.

![Cuenta de dominio de inquilino de Azure AD personalizado](./media/media-services-cenc-with-multidrm-access-control/media-services-ad-tenant-domain2.png)

**Cuenta de Microsoft**: Página de inicio de sesión de la cuenta de Microsoft para los consumidores.

![Cuenta de dominio de inquilino de Azure AD personalizado](./media/media-services-cenc-with-multidrm-access-control/media-services-ad-tenant-domain3.png)

### <a name="use-encrypted-media-extensions-for-playready"></a>Uso de extensiones multimedia cifradas para PlayReady
En un explorador moderno con compatibilidad con las extensiones multimedia cifradas (EME) para PlayReady, como Internet Explorer 11 en Windows 8.1 o superior y el explorador de Microsoft Edge en Windows 10, PlayReady es la DRM subyacente para EME.

![Uso de EME para PlayReady](./media/media-services-cenc-with-multidrm-access-control/media-services-eme-for-playready1.png)

El área oscura del reproductor se debe a que la protección de PlayReady impide la realización de una captura de pantalla del vídeo protegido.

En la captura de pantalla siguiente se muestran los complementos del reproductor y la compatibilidad con Microsoft Security Essentials (MSE)/EME:

![Complementos de reproductor para PlayReady](./media/media-services-cenc-with-multidrm-access-control/media-services-eme-for-playready2.png)

EME en Microsoft Edge e Internet Explorer 11 en Windows 10 permiten invocar [PlayReady SL3000](https://www.microsoft.com/playready/features/EnhancedContentProtection.aspx/) en dispositivos de Windows 10 que lo admitan. PlayReady SL3000 desbloquea el flujo del contenido premium mejorado (4K, HDR) y los nuevos modelos de entrega de contenido (para el contenido mejorado).

Para centrarse en los dispositivos de Windows, PlayReady es la única DRM en el hardware disponible en dispositivos de Windows (PlayReady SL3000). Un servicio de streaming puede usar PlayReady mediante EME o una aplicación de Plataforma universal de Windows y ofrecer una mayor calidad de vídeo con PlayReady SL3000 que otro DRM. Normalmente, el contenido de hasta 2K fluye a través de Chrome o Firefox, y el contenido de hasta 4K fluye a través de Microsoft Edge/Internet Explorer 11 o una aplicación de la Plataforma universal de Windows en el mismo dispositivo. La cantidad depende de la implementación y del valor de configuración del servicio.

#### <a name="use-eme-for-widevine"></a>Uso de EME para Widevine
En un explorador moderno con compatibilidad con EME/Widevine, como Chrome 41+ en Windows 10, Windows 8.1, Mac OSX Yosemite y Chrome en Android 4.4.4, Google Widevine es el DRM que subyace a EME.

![Uso de EME para Widevine](./media/media-services-cenc-with-multidrm-access-control/media-services-eme-for-widevine1.png)

Widevine no impide la realización de capturas de pantalla de vídeo protegido.

![Complementos de reproductor para Widevine](./media/media-services-cenc-with-multidrm-access-control/media-services-eme-for-widevine2.png)

### <a name="unentitled-users"></a>Usuarios no autorizados
Si el usuario no es miembro del grupo Usuarios autorizados, el usuario no pasa la comprobación de derechos. El servicio de licencias de DRM de múltiples, a continuación, se niega a emitir la licencia solicitada, como se muestra. La descripción detallada es "Error al adquirir licencias", que es como se ha diseñado.

![Usuarios no autorizados](./media/media-services-cenc-with-multidrm-access-control/media-services-unentitledusers.png)

### <a name="run-a-custom-security-token-service"></a>Ejecución de un servicio de token de seguridad
Si ejecuta a un servicio de token de seguridad personalizado, el token JWT lo emite dicho servicio personalizado mediante una clave simétrica o asimétrica.

En la captura de pantalla siguiente se muestra un escenario que utiliza una clave simétrica (con Chrome):

![Servicio de token de seguridad personalizado con una clave simétrica](./media/media-services-cenc-with-multidrm-access-control/media-services-running-sts1.png)

En la captura de pantalla siguiente se muestra un escenario que utiliza una clave asimétrica a través de un certificado X509 (mediante un explorador moderno de Microsoft):

![Servicio de token de seguridad personalizado con una clave asimétrica](./media/media-services-cenc-with-multidrm-access-control/media-services-running-sts2.png)

En los dos casos anteriores, la autenticación de usuario permanece igual. Tiene lugar mediante Azure AD. La única diferencia es que los tokens JWT los emite el servicio de token de seguridad personalizado en lugar de Azure AD. Cuando se configura la protección CENC dinámica, la restricción del servicio de entrega de licencias especifica el tipo de token JWT, ya sea una clave simétrica o asimétrica.

## <a name="summary"></a>Resumen

En este documento se examina el CENC con varias DRM nativas y el control de acceso mediante la autenticación con tokens, su diseño y su implementación mediante Azure, Media Services y Media Player.

* Se presenta un diseño de referencia que contiene todos los componentes necesarios de un subsistema DRM/CENC,
* y una implementación de referencia en Azure, Media Services y Media Player.
* También se han tratado algunos temas directamente relacionados con el diseño y la implementación.

## <a name="additional-notes"></a>Notas adicionales

* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
 
