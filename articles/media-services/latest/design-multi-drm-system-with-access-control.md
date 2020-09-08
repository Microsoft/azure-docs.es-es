---
title: 'Sistema de protección de contenido con varios DRM: Azure Media Services, versión 3'
description: En este artículo se proporciona una descripción detallada de cómo diseñar un sistema de protección de contenido con varios DRM con Azure Media Services.
services: media-services
documentationcenter: ''
author: willzhan
manager: steveng
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: willzhan
ms.custom: seodec18
ms.openlocfilehash: 58edf1e0257cf9de8d8f3a3b56f295dcaf1f6cbf
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89298205"
---
# <a name="design-of-a-multi-drm-content-protection-system-with-access-control"></a>Diseño del sistema de protección de contenido con DRM múltiple con control de acceso

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

El diseño y la creación de un subsistema de administración de derechos digitales (DRM) para una solución OTT o de streaming en línea es una tarea compleja. Los operadores o proveedores de vídeo en línea suelen subcontratar esta tarea a proveedores de servicios DRM especializados. El objetivo de este documento es presentar un diseño de referencia y una implementación de referencia del subsistema DRM de un extremo a otro en una solución de streaming en línea o de OTT.

Este documento va dirigido a ingenieros que trabajan en subsistemas DRM de soluciones OTT o soluciones de varias pantallas o streaming en línea, o a cualquier lector interesado en subsistemas DRM. Se supone que los lectores están familiarizados con al menos una de las tecnologías DRM del mercado, como PlayReady, Widevine, FairPlay o Adobe Access.

En este debate, en multi-DRM se incluyen los 3 DRM admitidos por Azure Media Services: Cifrado común (CENC) de PlayReady y Widevine, FairPlay, así como claves sin cifrado AES-128. Una tendencia importante en el streaming en línea y en la industria OTT es utilizar DRM nativas en diversas plataformas de cliente. Esta tendencia es un avance respecto a la anterior, que usa un único DRM y su cliente SDK para varias plataformas de cliente. Cuando se utiliza CENC con varias DRM nativas, tanto PlayReady como Widevine se cifran según la especificación [Cifrado común (ISO/IEC 23001-7 CENC)](https://www.iso.org/iso/home/store/catalogue_ics/catalogue_detail_ics.htm?csnumber=65271/).

Las ventajas de usar varias DRM nativa para la protección de contenido son que:

* Reduce el costo del cifrado porque se emplea un solo procesamiento de cifrado que tiene como destino distintas plataformas con sus DRM nativas.
* Reduce el costo de administrar los recursos porque solo se necesita una copia de recurso en el almacenamiento.
* Elimina el costo de licencia de clientes DRM porque el cliente DRM nativo suele ser gratis en su plataforma nativa.

### <a name="goals-of-the-article"></a>Objetivos del artículo

Los objetivos de este artículo son los siguientes:

* Proporcionar un diseño de referencia de un subsistema DRM que usa las tres DRM (CENC para DASH, FairPlay para HLS y PlayReady para Smooth Streaming).
* Proporcionar una implementación de referencia en la plataforma de Azure y de Azure Media Services.
* Describir algunos temas de diseño e implementación.

La tabla siguiente resume la compatibilidad con DRM nativa en distintas plataformas y la compatibilidad con EME en diferentes exploradores.

| **Plataforma de cliente** | **DRM nativa** | **EME** |
| --- | --- | --- |
| **Televisores inteligentes y descodificadores** | PlayReady, Widevine y/u otras | Explorador incrustados/EME para PlayReady y/o Widevine|
| **Windows 10** | PlayReady | Microsoft Edge/IE11 para PlayReady|
| **Dispositivos Android (teléfono, tableta, TV)** |Widevine |Chrome para Widevine |
| **iOS** | FairPlay | Safari para FairPlay (desde iOS 11.2) |
| **macOS** | FairPlay | Safari para FairPlay (desde Safari 9+ en Mac OS X 10.11+ El Capitan)|
| **tvOS** | FairPlay | |

Teniendo en cuenta el estado actual de implementación para cada DRM, normalmente el servicio quiere implementar dos o tres DRM para asegurarse de que aborda todos los tipos de puntos de conexión de la mejor manera.

Hay un equilibrio entre la complejidad de la lógica del servicio y la complejidad del cliente para alcanzar un determinado nivel de experiencia del usuario en los distintos clientes.

Para realizar la selección, tenga en cuenta lo siguiente:

* PlayReady se implementa de forma nativa en cada dispositivo de Windows, en algunos dispositivos Android y está disponibles mediante los SDK de software en prácticamente cualquier plataforma.
* Widevine se implementa de forma nativa en todos los dispositivos Android, en Chrome y algunos otros dispositivos. Widevine también se admite en los exploradores Firefox y Opera a través de DASH.
* FairPlay está disponible en iOS, macOS y tvOS.


## <a name="a-reference-design"></a>Un diseño de referencia
En esta sección se presenta un diseño de referencia que es independiente de las tecnologías usadas para implementarlo.

Un subsistema DRM puede contener los siguientes componentes:

* Administración de claves
* Empaquetado de cifrado de DRM
* Entrega de licencias DRM
* Control de acceso y comprobación de derechos
* Autenticación y autorización de usuarios
* Aplicación de reproducción
* Origen/red de entrega de contenido (CDN)

En el siguiente diagrama ilustra la interacción de alto nivel entre los componentes de un subsistema DRM:

![Subsistema DRM con CENC](./media/design-multi-drm-system-with-access-control/media-services-generic-drm-subsystem-with-cenc.png)

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
| **Servicio de token seguro (STS)** |Azure AD |
| **Flujo de trabajo de protección DRM** |Protección dinámica de Azure Media Services |
| **Entrega de licencia DRM** |* Entrega de licencias de Media Services (PlayReady, Widevine, FairPlay) <br/>* Servidor de licencias de Axinom <br/>* Servidor de licencias personalizado de PlayReady |
| **Origen** |Punto de conexión de streaming de Azure Media Services |
| **Administración de claves** |No es necesaria para la implementación de referencia. |
| **Administración de contenido** |Una aplicación de consola de C#. |

En otras palabras, Azure AD proporciona tanto IDP como STS. La [API de Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/) se usa para el reproductor. Tanto Azure Media Services como Azure Media Player admiten CENC a través de DASH, FairPlay a través de HLS, PlayReady a través de Smooth Streaming y cifrado AES-128 para DASH, HLS y Smooth.

En el diagrama siguiente se muestra la estructura y el flujo generales con la asignación de tecnología anterior:

![CENC en Media Services](./media/design-multi-drm-system-with-access-control/media-services-cenc-subsystem-on-AMS-platform.png)

Para configurar la protección de contenido con DRM, la herramienta de administración de contenido usa las siguientes entradas:

* Contenido abierto
* Clave de contenido de la administración de claves
* Las direcciones URL de adquisición de licencias
* Una lista de información de Azure AD, como notificaciones de token, emisor y audiencia

Esta es la salida de la herramienta de administración de contenido:

* ContentKeyPolicy describe la plantilla de licencia de DRM para cada tipo de DRM que usa;
* ContentKeyPolicyRestriction describe el control de acceso antes de que se emita una licencia de DRM
* Streamingpolicy describe las diversas combinaciones de DRM formato de contenedor (modo de cifrado, protocolo de streaming, formato de contenedor) para streaming
* StreamingLocator describe la clave de contenido/IV que se usa para cifrado y las direcciones URL de streaming 

Este es el flujo en tiempo de ejecución:

* Tras la autenticación del usuario, se genera un token JWT.
* Una de las notificaciones contenidas en el token JWT es la notificación de grupos que contiene el identificador de objeto de grupo de EntitledUserGroup. Esta notificación se utiliza para pasar la comprobación de derechos.
* El reproductor descarga el manifiesto de cliente del contenido protegido por CENC de identifica lo siguiente:
   * El identificador de la clave.
   * El contenido está protegido con DRM.
   * Las direcciones URL de adquisición de licencias.
* El reproductor realiza una solicitud de adquisición de licencias basada en el explorador o DRM admitidas. En la solicitud de adquisición de licencias, también se envían el identificador de clave y un token JWT. El servicio de entrega de licencias comprobará el token JWT y las notificaciones contenidas antes de emitir la licencia necesaria.

## <a name="implementation"></a>Implementación
### <a name="implementation-procedures"></a>Procedimientos de implementación
La implementación incluye los siguientes pasos:

1. Preparar los recursos de prueba. Codifique o empaquete un vídeo de prueba en MP4 fragmentado de varias velocidades de bits en Media Services. Este recurso *no* está protegido con DRM. La protección DRM se realiza posteriormente mediante protección dinámica.

2. Cree el identificador de clave y una clave de contenido (opcionalmente a partir de la inicialización de clave). En este caso, no es necesario el sistema de administración de claves porque solo se requiere un identificador de clave y una clave de contenido para un par de recursos de prueba.

3. Utilice la API de Media Services para configurar servicios de entrega de licencias de varias DRM para el recurso de prueba. Si utiliza servidores de licencias personalizados de su empresa o de los proveedores de su empresa en lugar de servicios de licencias en Media Services, puede omitir este paso. Puede especificar las direcciones URL de adquisición de licencias en el paso al configurar la entrega de licencias. Las API de Media Services son necesarias para especificar algunas configuraciones detalladas, como restricción de directivas de autorización y plantillas de respuesta de licencia para los distintos servicios de licencias DRM. En este momento, Azure Portal no proporciona la interfaz de usuario necesaria para esta configuración. Para más información sobre el nivel de API y para ver un código de ejemplo, consulte [Uso del cifrado dinámico común de PlayReady o Widevine](protect-with-drm.md).

4. Utilice la API de Media Services para configurar la directiva de entrega de recursos para el recurso de prueba. Para más información sobre el nivel de API y para ver un código de ejemplo, consulte [Uso del cifrado dinámico común de PlayReady o Widevine](protect-with-drm.md).

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

* Puede encontrar información para desarrolladores en la [Guía para desarrolladores de Azure Active Directory](../../active-directory/develop/v2-overview.md).
* Puede encontrar información para administradores en [Administración del directorio de Azure AD](../../active-directory/fundamentals/active-directory-whatis.md).

### <a name="some-issues-in-implementation"></a>Algunos problemas de implementación

Utilice la siguiente información para solución de problemas para obtener ayuda con problemas de implementación.

* La dirección URL del emisor debe terminar por "/". La audiencia debe ser el identificador de cliente de la aplicación de reproductor. Además, agregue "/" al final de la dirección URL del emisor.

    ```xml
    <add key="ida:audience" value="[Application Client ID GUID]" />
    <add key="ida:issuer" value="https://sts.windows.net/[AAD Tenant ID]/" />
    ```

    En [JWT Decoder](http://jwt.calebb.net/), debería ver **aud** y **iss**, como se muestra a continuación en el token JWT:

    ![JWT](./media/design-multi-drm-system-with-access-control/media-services-1st-gotcha.png)

* Agregue permisos a la aplicación en Azure AD en la pestaña **Configurar** de la aplicación. Los permisos son necesarios para cada aplicación, tanto para las versiones locales como implementadas.

    ![Permisos](./media/design-multi-drm-system-with-access-control/media-services-perms-to-other-apps.png)

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

    `objTokenRestrictionTemplate.TokenType = TokenType.JWT;`

    Como agregó compatibilidad con el token JWT (Azure AD), además de con SWT (ACS), el valor predeterminado de TokenType es TokenType.JWT. Si utiliza SWT y ACS, debe establecer el token en TokenType.SWT.

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

![Cuenta uno de dominio de inquilino de Azure AD personalizado](./media/design-multi-drm-system-with-access-control/media-services-ad-tenant-domain1.png)

**Cuenta de dominio de Microsoft con tarjeta inteligente**: Página de inicio de sesión personalizada por el responsable de TI corporativo de Microsoft con autenticación en dos fases.

![Cuenta dos de dominio de inquilino de Azure AD personalizado](./media/design-multi-drm-system-with-access-control/media-services-ad-tenant-domain2.png)

**Cuenta de Microsoft**: Página de inicio de sesión de la cuenta de Microsoft para los consumidores.

![Cuenta tres de dominio de inquilino de Azure AD personalizado](./media/design-multi-drm-system-with-access-control/media-services-ad-tenant-domain3.png)

### <a name="use-encrypted-media-extensions-for-playready"></a>Uso de extensiones multimedia cifradas para PlayReady

En un explorador moderno con compatibilidad con las extensiones multimedia cifradas (EME) para PlayReady, como Internet Explorer 11 en Windows 8.1 o superior y el explorador de Microsoft Edge en Windows 10, PlayReady es la DRM subyacente para EME.

![Uso de EME para PlayReady](./media/design-multi-drm-system-with-access-control/media-services-eme-for-playready1.png)

El área oscura del reproductor se debe a que la protección de PlayReady impide la realización de una captura de pantalla del vídeo protegido.

En la captura de pantalla siguiente se muestran los complementos del reproductor y la compatibilidad con Microsoft Security Essentials (MSE)/EME:

![Complementos de reproductor para PlayReady](./media/design-multi-drm-system-with-access-control/media-services-eme-for-playready2.png)

EME en Microsoft Edge e Internet Explorer 11 en Windows 10 permiten invocar [PlayReady SL3000](https://www.microsoft.com/playready/features/EnhancedContentProtection.aspx/) en dispositivos de Windows 10 que lo admitan. PlayReady SL3000 desbloquea el flujo del contenido premium mejorado (4K, HDR) y los nuevos modelos de entrega de contenido (para el contenido mejorado).

Para centrarse en los dispositivos de Windows, PlayReady es la única DRM en el hardware disponible en dispositivos de Windows (PlayReady SL3000). Un servicio de streaming puede usar PlayReady mediante EME o una aplicación de Plataforma universal de Windows y ofrecer una mayor calidad de vídeo con PlayReady SL3000 que otro DRM. Normalmente, el contenido de hasta 2K fluye a través de Chrome o Firefox, y el contenido de hasta 4K fluye a través de Microsoft Edge/Internet Explorer 11 o una aplicación de la Plataforma universal de Windows en el mismo dispositivo. La cantidad depende de la implementación y del valor de configuración del servicio.

#### <a name="use-eme-for-widevine"></a>Uso de EME para Widevine

En un explorador moderno con compatibilidad con EME/Widevine, como Chrome 41+ en Windows 10, Windows 8.1, Mac OSX Yosemite y Chrome en Android 4.4.4, Google Widevine es el DRM que subyace a EME.

![Uso de EME para Widevine](./media/design-multi-drm-system-with-access-control/media-services-eme-for-widevine1.png)

Widevine no impide la realización de capturas de pantalla de vídeo protegido.

![Complementos de reproductor para Widevine](./media/design-multi-drm-system-with-access-control/media-services-eme-for-widevine2.png)

#### <a name="use-eme-for-fairplay"></a>Uso de EME para FairPlay

De forma similar, puede probar el contenido protegido mediante FairPlay en este reproductor de prueba en Safari en macOS o iOS 11.2 y versiones posteriores.

Asegúrese de que pone "FairPlay" como protectionInfo.type y lo coloca en la dirección URL correcta para el certificado de la aplicación en FPS AC Path (Ruta de acceso de AC para FPS), que es la ruta de acceso del certificado de la aplicación de streaming para FairPlay.

### <a name="unentitled-users"></a>Usuarios no autorizados

Si el usuario no es miembro del grupo Usuarios autorizados, el usuario no pasa la comprobación de derechos. El servicio de licencias de DRM de múltiples, a continuación, se niega a emitir la licencia solicitada, como se muestra. La descripción detallada es "Error al adquirir licencias", que es como se ha diseñado.

![Usuarios no autorizados](./media/design-multi-drm-system-with-access-control/media-services-unentitledusers.png)

### <a name="run-a-custom-security-token-service"></a>Ejecución de un servicio de token de seguridad

Si ejecuta a un servicio de token de seguridad personalizado, el token JWT lo emite dicho servicio personalizado mediante una clave simétrica o asimétrica.

En la captura de pantalla siguiente se muestra un escenario que utiliza una clave simétrica (con Chrome):

![Servicio de token de seguridad personalizado con una clave simétrica](./media/design-multi-drm-system-with-access-control/media-services-running-sts1.png)

En la captura de pantalla siguiente se muestra un escenario que utiliza una clave asimétrica a través de un certificado X509 (mediante un explorador moderno de Microsoft):

![Servicio de token de seguridad personalizado con una clave asimétrica](./media/design-multi-drm-system-with-access-control/media-services-running-sts2.png)

En los dos casos anteriores, la autenticación de usuario permanece igual. Tiene lugar mediante Azure AD. La única diferencia es que los tokens JWT los emite el servicio de token de seguridad personalizado en lugar de Azure AD. Cuando se configura la protección CENC dinámica, la restricción del servicio de entrega de licencias especifica el tipo de token JWT, ya sea una clave simétrica o asimétrica.

## <a name="next-steps"></a>Pasos siguientes

* [Preguntas más frecuentes](frequently-asked-questions.md)
* [Introducción a la protección de contenido](content-protection-overview.md)
* [Protección del contenido con DRM](protect-with-drm.md)
