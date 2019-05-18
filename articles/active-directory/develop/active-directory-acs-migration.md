---
title: Migración desde Azure Access Control Service | Microsoft Docs
description: Aprenda sobre las opciones para mover aplicaciones y servicios fuera de Azure Access Control Service (ACS).
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 10/03/2018
ms.author: ryanwi
ms.reviewer: jlu, annaba, hirsin
ms.collection: M365-identity-device-management
ms.openlocfilehash: 84a8c2954473401a9e57cba045907c60862ed61f
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65546237"
---
# <a name="how-to-migrate-from-the-azure-access-control-service"></a>Procedimientos para: Migración desde Azure Access Control Service

Microsoft Azure Access Control Service (ACS), un servicio de Azure Active Directory (Azure AD), se retirará el 7 de noviembre de 2018. Para entonces, las aplicaciones y servicios que utilizan Access Control deben migrarse completamente a un mecanismo de autenticación diferente. En este artículo se describen las recomendaciones para los clientes actuales que pretenden dejar de usar Access Control. Si actualmente no utiliza Access Control, no es necesario realizar ninguna acción.

## <a name="overview"></a>Información general

Access Control es un servicio de autenticación en la nube que ofrece una manera sencilla de autenticar y autorizar a los usuarios que quieran obtener acceso a los servicios y aplicaciones web. Permite que muchas características de autenticación y autorización se factoricen a partir del código. Los programadores y arquitectos de Microsoft .NET, las aplicaciones web ASP.NET y los servicios web de Windows Communication Foundation (WCF) usan principalmente Access Control.

Los casos de uso de Access Control pueden dividirse en tres categorías principales:

- Autenticar determinados servicios en la nube de Microsoft, incluidos Azure Service Bus y Dynamics CRM. Las aplicaciones cliente que pueden obtener tokens de Access Control que pueden usarse a fin de autenticar estos servicios para realizar diversas acciones.
- Agregar el proceso de autenticación a aplicaciones web personalizadas y preestablecidas (como SharePoint). Mediante la autenticación "pasiva" de Access Control, las aplicaciones web pueden permitir que se inicie sesión con una cuenta de Microsoft (anteriormente Live ID), con cuentas de Google, Facebook, Yahoo, Azure AD y los Servicios de federación de Active Directory (AD FS).
- Proteger los servicios web integrados personalizados con tokens que haya emitido Access Control. Mediante la autenticación "activa", los servicios web pueden garantizar que solo se permite el acceso de clientes conocidos que se hayan autenticado con Access Control.

Cada uno de estos casos de uso y sus estrategias recomendadas de migración se describen en las secciones siguientes.

> [!WARNING]
> En la mayoría de los casos, se requieren cambios de código significativos para migrar los servicios y las aplicaciones existentes a las tecnologías más recientes. Le recomendamos que comience la planificación y la ejecución de la migración inmediatamente para evitar la posibilidad de que se produzcan interrupciones o tiempos de inactividad.

Access Control tiene los siguientes componentes:

- Un servicio de token seguro (STS) que recibe las solicitudes de autenticación y emite de vuelta tokens de seguridad.
- El Portal de Azure clásico, donde se crean, eliminan, habilitan y deshabilitan los espacios de nombres de Access Control.
- Un portal de administración de Access Control donde puede personalizar y configurar los espacios de nombres de Access Control.
- Un servicio de administración que se puede usar para automatizar las funciones de los portales.
- Un motor de reglas de transformación de tokens que se puede usar para definir una lógica compleja a fin de manipular el formato de salida de los tokens emitidos por Access Control.

Para utilizar estos componentes, debe crear uno o varios espacios de nombres de Access Control. Un *espacio de nombres* es una instancia dedicada de Access Control con el que se comunican los servicios y las aplicaciones. Asimismo, está aislado del resto de los clientes de Access Control. Otros clientes de Access Control usan sus propios espacios de nombres. Un espacio de nombres en Access Control tiene una dirección URL dedicada con el siguiente aspecto:

```HTTP
https://<mynamespace>.accesscontrol.windows.net
```

Toda comunicación con las operaciones de administración y STS se realiza en esta dirección URL. Use rutas de acceso diferentes para distintos fines. Para determinar si sus aplicaciones o servicios utilizan Access Control, supervise todo el tráfico a https://&lt;espacio de nombres&gt;.accesscontrol.windows.net. Access Control maneja todo el tráfico dirigido a esta dirección URL y debe interrumpirse. 

La excepción a esto es todo el tráfico a `https://accounts.accesscontrol.windows.net`. El tráfico a esta dirección URL ya se controla mediante un servicio diferente y **no** se ve afectado debido al desuso de Access Control. 

Para obtener más información sobre Access Control, consulte [Access Control Service 2.0 (archivado)](https://msdn.microsoft.com/library/hh147631.aspx).

## <a name="find-out-which-of-your-apps-will-be-impacted"></a>Averigüe qué aplicaciones se verán afectadas

Siga los pasos descritos en esta sección para averiguar qué aplicaciones se verán afectadas por la retirada de ACS.

### <a name="download-and-install-acs-powershell"></a>Descargue e instale ACS PowerShell

1. Vaya a la Galería de PowerShell y descargue [Acs.Namespaces](https://www.powershellgallery.com/packages/Acs.Namespaces/1.0.2).
1. Ejecute lo siguiente para instalar el módulo:

    ```powershell
    Install-Module -Name Acs.Namespaces
    ```

1. Ejecute lo siguiente para obtener una lista de todos los comandos posibles:

    ```powershell
    Get-Command -Module Acs.Namespaces
    ```

    Para obtener ayuda sobre un comando específico, ejecute:

    ```
     Get-Help [Command-Name] -Full
    ```
    
    donde `[Command-Name]` es el nombre del comando de ACS.

### <a name="list-your-acs-namespaces"></a>Enumeración del espacio de nombres de ACS

1. Conéctese a ACS con el cmdlet **Connect AcsAccount**.
  
    Puede que tenga que ejecutar `Set-ExecutionPolicy -ExecutionPolicy Bypass` antes de poder ejecutar los comandos y que tenga que ser el administrador de esas suscripciones para poder ejecutar los comandos.

1. Enumere las suscripciones de Azure disponibles con el cmdlet **Get AcsSubscription**.
1. Enumere los espacios de nombres de ACS con el cmdlet **Get AcsNamespace**.

### <a name="check-which-applications-will-be-impacted"></a>Compruebe qué aplicaciones que se verán afectadas

1. Use el espacio de nombres del paso anterior y vaya a `https://<namespace>.accesscontrol.windows.net`

    Por ejemplo, si uno de los espacios de nombres es contoso-test, vaya a `https://contoso-test.accesscontrol.windows.net`

1. En **Relaciones de confianza**, seleccione **Aplicaciones de usuario de confianza** para ver la lista de aplicaciones que se verán afectadas por la retirada de ACS.
1. Repita los pasos 1 y 2 para otros espacios de nombres de ACS que tenga.

## <a name="retirement-schedule"></a>Calendario de retiradas

A partir de noviembre de 2017, todos los componentes de Access Control son totalmente compatibles y están operativos. La única restricción es que [no se pueden crear nuevos espacios de nombres de Access Control a través del Portal de Azure clásico](https://azure.microsoft.com/blog/acs-access-control-service-namespace-creation-restriction/).

Aquí está el calendario que indica el desuso de los componentes de Access Control:

- **Noviembre de 2017**:  la experiencia de administración de Azure AD en el Portal de Azure clásico [se retira](https://blogs.technet.microsoft.com/enterprisemobility/2017/09/18/marching-into-the-future-of-the-azure-ad-admin-experience-retiring-the-azure-classic-portal/). En este momento, la administración del espacio de nombres de Access Control estará disponible en esta nueva dirección URL dedicada: `https://manage.windowsazure.com?restoreClassic=true`. Use esta URl para ver los espacios de nombres existentes y habilitar, deshabilitar y eliminar diversos espacios de nombres si así lo desea.
- **2 de abril de 2018**: el Portal de Azure clásico se ha retirado por completo, lo que significa que la administración del espacio de nombres de Access Control ya no está disponible a través de ninguna dirección URL. No podrá deshabilitar o habilitar, eliminar o enumerar los espacios de nombres de Access Control. Sin embargo, el portal de administración de Access Control será totalmente funcional y se ubicará en `https://\<namespace\>.accesscontrol.windows.net`. Todos los demás componentes de Access Control también seguirán funcionando con normalidad.
- **7 de noviembre de 2018**: todos los componentes de Access Control se cierran permanentemente. Esto incluye el portal de administración de Access Control, el servicio de administración, STS y el motor de reglas de transformación de tokens. En este momento, las solicitudes enviadas a Access Control (ubicado en \<espacio de nombres\>.accesscontrol.windows.net) producirán un error. Debe haber migrado todos los servicios y aplicaciones existentes a otras tecnologías bastante antes de esta fecha.

> [!NOTE]
> Una directiva deshabilita los espacios de nombres que no han solicitado un token durante un período de tiempo. A partir de principios de septiembre de 2018, este período de tiempo es actualmente de 14 días de inactividad, pero se reducirá a 7 días de inactividad en las próximas semanas. Si tiene espacios de nombres de Access Control que actualmente están deshabilitados, puede [descargar e instalar PowerShell ACS](#download-and-install-acs-powershell) para volver a habilitarlos.

## <a name="migration-strategies"></a>Estrategias de migración

En las secciones siguientes se describen recomendaciones detalladas para la migración de Access Control a otras tecnologías de Microsoft.

### <a name="clients-of-microsoft-cloud-services"></a>Clientes de los Servicios en la nube de Microsoft

Cada uno de los Servicios en la nube de Microsoft que aceptan tokens que haya emitido Access Control ahora admite al menos una forma alternativa de autenticación. El mecanismo de autenticación correcto varía para cada servicio. Le recomendamos que consulte la documentación específica de cada servicio para obtener una guía oficial. Para mayor comodidad, cada conjunto de documentación se proporciona aquí:

| Servicio | Guía |
| ------- | -------- |
| Azure Service Bus | [Migración a firmas de acceso compartido](https://docs.microsoft.com/azure/service-bus-messaging/service-bus-migrate-acs-sas) |
| Azure Service Bus Relay | [Migración a firmas de acceso compartido](https://docs.microsoft.com/azure/service-bus-relay/relay-migrate-acs-sas) |
| Azure Managed Cache | [Migración a Azure Cache for Redis](https://docs.microsoft.com/azure/azure-cache-for-redis/cache-faq#which-azure-cache-offering-is-right-for-me) |
| Azure DataMarket | [Migración a Cognitive Services APIs](https://docs.microsoft.com/azure/machine-learning/studio/datamarket-deprecation) |
| Servicios de BizTalk | [Migración a la característica Logic Apps de Azure App Service](https://docs.microsoft.com/azure/machine-learning/studio/datamarket-deprecation) |
| Azure Media Services | [Migración a la autenticación de Azure AD](https://azure.microsoft.com/blog/azure-media-service-aad-auth-and-acs-deprecation/) |
| Azure Backup | [Actualización del agente de Azure Backup](https://docs.microsoft.com/azure/backup/backup-azure-file-folder-backup-faq) |

<!-- Dynamics CRM: Migrate to new SDK, Dynamics team handling privately -->
<!-- Azure RemoteApp deprecated in favor of Citrix: https://www.zdnet.com/article/microsoft-to-drop-azure-remoteapp-in-favor-of-citrix-remoting-technologies/ -->
<!-- Exchange push notifications are moving, customers don't need to move -->
<!-- Retail federation services are moving, customers don't need to move -->
<!-- Azure StorSimple: TODO -->
<!-- Azure SiteRecovery: TODO -->

### <a name="sharepoint-customers"></a>Clientes de SharePoint

SharePoint 2013 y 2016 y SharePoint Online a los clientes cuánto tiempo han usado ACS para realizar la autenticación en la nube, en escenarios híbridos y locales. Algunas características de y casos de uso de SharePoint resultarán afectados por la retirada de ACS, pero otros no. En la tabla siguiente se resume la guía de migración para algunas de las características más populares de SharePoint que hacen uso de ACS:

| Característica | Guía |
| ------- | -------- |
| Autenticación de usuarios desde Azure AD | Anteriormente, Azure AD no admitía los tokens de SAML 1.1 que necesitaba SharePoint para la autenticación, y ACS se usaba como intermediario que permitía la compatibilidad de SharePoint con formatos de token de Azure AD. Ahora, puede [conectar SharePoint directamente a Azure AD con SharePoint de galería de aplicaciones de Azure AD en la aplicación localmente](https://docs.microsoft.com/azure/active-directory/saas-apps/sharepoint-on-premises-tutorial). |
| [Autenticación de aplicaciones y la autenticación de servidor a servidor de SharePoint local](https://technet.microsoft.com/library/jj219571(v=office.16).aspx) | No resulta afectado por la retirada de ACS; no se requieren cambios. | 
| [Autorización de confianza baja para complementos de SharePoint (proveedor hospedado y SharePoint hospedado)](https://docs.microsoft.com/sharepoint/dev/sp-add-ins/three-authorization-systems-for-sharepoint-add-ins) | No resulta afectado por la retirada de ACS; no se requieren cambios. |
| [Búsqueda de SharePoint en la nube híbrida](https://blogs.msdn.microsoft.com/spses/2015/09/15/cloud-hybrid-search-service-application/) | No resulta afectado por la retirada de ACS; no se requieren cambios. |

### <a name="web-applications-that-use-passive-authentication"></a>Aplicaciones web que usan autenticación pasiva

Access Control proporciona a arquitectos y desarrolladores de aplicaciones web que utilizan Access Control para autenticar usuarios, las siguientes características y capacidades :

- Integración profunda con Windows Identity Foundation (WIF).
- Federación con Google, Facebook, Yahoo, Azure Active Directory y cuentas de AD FS y Microsoft.
- Compatibilidad con los siguientes protocolos de autenticación: OAuth 2.0 Draft 13, WS-Trust y Web Services Federation (WS-Federation).
- Compatibilidad con los siguientes formatos de token: JSON Web Token (JWT), SAML 1.1, SAML 2.0 y Simple Web Token (SWT).
- Una experiencia de detección de dominio de inicio, integrada en WIF, que permite a los usuarios seleccionar el tipo de cuenta que usan para iniciar sesión. Esta experiencia es totalmente personalizable y la proporciona la aplicación web.
- La transformación de token que permite la variada personalización de las notificaciones que recibe la aplicación web de ACS, incluyendo:
    - El paso de las notificaciones desde los proveedores de identidad.
    - La incorporación de notificaciones personalizadas adicionales.
    - La lógica de if-then simple para emitir notificaciones bajo ciertas condiciones.

Por desgracia, no hay un servicio que ofrezca todas estas funcionalidades equivalentes. Debe evaluar qué funcionalidades de Access Control necesita y, a continuación, elegir entre usar [Azure Active Directory](https://azure.microsoft.com/develop/identity/signin/), [Azure Active Directory B2C](https://azure.microsoft.com/services/active-directory-b2c/) (Azure AD B2C) u otro servicio de autenticación en la nube.

#### <a name="migrate-to-azure-active-directory"></a>Migración a Azure Active Directory

Una forma de migración sería integrar las aplicaciones y servicios directamente con Azure AD. Azure AD es el proveedor de identidades basado en la nube de cuentas profesionales o educativas de Microsoft. Azure AD es el proveedor de identidades para Office 365, Azure y muchos servicios más. Proporciona funcionalidades de autenticación federada similares a Access Control, pero no admite todas sus características. 

El ejemplo principal es la federación con proveedores de identidades sociales, como Facebook, Google y Yahoo. Si sus usuarios inician sesión con estos tipos de credenciales, Azure AD no es la mejor opción. 

Asimismo, Azure AD no admite necesariamente los mismos protocolos de autenticación que Access Control. Por ejemplo, aunque tanto Access Control como Azure AD admiten OAuth, existen diferencias sutiles entre cada implementación. Las diferentes implementaciones requieren que modifique el código como parte de una migración.

Sin embargo, Azure AD proporciona varias ventajas potenciales a los clientes de Access Control. De forma nativa, es compatible con las cuentas educativas o profesionales de Microsoft hospedadas en la nube y que normalmente usan clientes de Access Control. 

Igualmente, un inquilino de Azure AD también se puede federar en una o más instancias de una cuenta local de Active Directory mediante AD FS. De esta manera, la aplicación puede autenticar a usuarios basados en la nube y usuarios que estén hospedados de manera local. También admite el protocolo WS-Federation, con el que la integración con una aplicación web mediante WIF resulta bastante sencilla.

En la siguiente tablase comparan las características de Access Control pertinentes a las aplicaciones web, con aquellas que están disponibles en Azure AD. 

En líneas generales, *Azure Active Directory probablemente no sea la opción adecuada para la migración, si solo permite a los usuarios iniciar sesión con sus cuentas educativas o profesionales de Microsoft*.

| Funcionalidad | Soporte técnico de Access Control | Soporte técnico de Azure AD |
| ---------- | ----------- | ---------------- |
| **Tipos de cuentas** | | |
| Cuentas educativas o profesionales de Microsoft | Compatible | Compatible |
| Cuentas de Windows Server Active Directory y AD FS |- Compatible a través de la federación con un inquilino de Azure AD. <br />- Compatible a través de la federación directa con AD FS. | Solo es compatible a través de la federación con un inquilino de Azure AD | 
| Cuentas desde otros sistemas de administración de identidades empresariales |- Es posible a través de la federación con un inquilino de Azure AD. <br />- Compatible a través de la federación directa. | Es posible a través de la federación con un inquilino de Azure AD |
| Cuentas de Microsoft para uso personal | Compatible | Compatible a través del protocolo v2.0 OAuth de Azure AD, pero no a través de ningún otro protocolo. | 
| Cuentas de Facebook, Google, Yahoo | Compatible | No se admite ningún tipo. |
| **Compatibilidad con el SDK y protocolos** | | |
| WIF | Compatible | Compatible, pero con instrucciones limitadas disponibles. |
| WS-Federation | Compatible | Compatible |
| OAuth 2.0 | Soporte para Draft 13 | Compatibilidad con RFC 6749, la especificación más moderna. |
| WS-Trust | Compatible | No compatible |
| **Formatos de tokens** | | |
| JWT | Compatible en versión beta | Compatible |
| SAML 1.1 | Compatible | Preview |
| SAML 2.0 | Compatible | Compatible |
| SWT | Compatible | No compatible |
| **Personalizaciones** | | |
| UI de selección de cuenta o detección de dominio de inicio personalizable | Código descargable que se puede incorporar en las aplicaciones | No compatible |
| Carga de certificados de firma de tokens personalizados | Compatible | Compatible |
| Personalización de notificaciones en tokens |- Paso a través de notificaciones de entrada desde proveedores de identidad<br />- Obtención de token de acceso del proveedor de identidades como una notificación<br />- Emisión de notificaciones de salida basadas en valores de las notificaciones de entrada<br />- Emisión de notificaciones de salida con valores constantes. |- No se puede pasar a través notificaciones desde proveedores de identidades federados.<br />- No se puede obtener el token de acceso del proveedor de identidades como una notificación.<br />- No se pueden emitir notificaciones de salida basadas en valores de notificaciones de entrada.<br />- Se pueden emitir notificaciones de salida con valores constantes.<br />- Se pueden emitir notificaciones de salida en función de las propiedades de los usuarios que se sincronizan con Azure AD. |
| **Automation** | | |
| Automatización de las tareas de configuración y administración | Compatible a través del servicio de administración de Access Control | Compatible a través de Microsoft Graph y Graph API de Azure AD. |

Si decide que Azure AD es la forma adecuada para migrar sus aplicaciones y servicios, debe tener en cuenta dos maneras de integrar con ella su aplicación.

Para utilizar WS-Federation o WIF para realizar integraciones con Azure AD, le recomendamos que siga las indicaciones descritas [Configuración del inicio de sesión único federado para una aplicación ajena a la galería](https://docs.microsoft.com/azure/active-directory/application-config-sso-how-to-configure-federated-sso-non-gallery). En este artículo se detalla la configuración de Azure AD para el inicio de sesión único basado en SAML, pero funciona también para configurar WS-Federation. Para poder continuar con este enfoque necesita una licencia Premium de Azure AD. Este enfoque tiene dos ventajas:

- Obtiene toda la flexibilidad de la personalización de tokens de Azure AD. Puede personalizar las notificaciones que emite Azure AD para que coincidan con las que emite Access Control. Esta opción incluye especialmente la notificación del id. de usuario o del identificador de nombre. Para continuar recibiendo identificadores de usuario consistentes de todos sus usuarios una vez haya cambiado de tecnología, debe asegurarse de que los id. de usuario que emita Azure AD coincidan con los que emita Access Control.
- Puede configurar un certificado de firma de tokens específico para su aplicación y cuya vigencia controle.

> [!NOTE]
> Para poder continuar con este enfoque necesita una licencia Premium de Azure AD. Si es un cliente de Access Control y necesita una licencia Premium para configurar el inicio de sesión único de una aplicación, póngase en contacto con nosotros. Estaremos encantados de proporcionarle licencias de desarrollador.

Una solución alternativa es seguir [este código de ejemplo](https://github.com/Azure-Samples/active-directory-dotnet-webapp-wsfederation) que ofrece instrucciones ligeramente diferentes acerca de cómo configurar WS-Federation. Este ejemplo de código no usa WIF, sino el middleware OWIN de ASP.NET 4.5. Sin embargo, las instrucciones para el registro de la aplicación son válidas para las aplicaciones que usan WIF y no requieren una licencia Premium de Azure AD. 

Si elige esta solución, debe entender la [sustitución de claves de firma de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/active-directory-signing-key-rollover). Esta solución usa la clave de firma global de Azure AD para emitir tokens. De forma predeterminada, WIF no actualiza automáticamente las claves de firma. Cuando Azure AD rota sus claves de firma globales, la implementación de WIF debe estar preparada para aceptar los cambios. Para más información, consulte [Sustitución de claves de firma de Azure Active Directory](https://msdn.microsoft.com/library/azure/dn641920.aspx).

Si puede realizar la integración con Azure AD a través de los protocolos de OpenID Connect u OAuth, se recomienda hacerlo. Tenemos una extensa documentación y guías sobre cómo integrar Azure AD en una aplicación web; están disponibles en nuestra [Guía del desarrollador de Azure AD](https://aka.ms/aaddev).

#### <a name="migrate-to-azure-active-directory-b2c"></a>Migrar a Azure Active Directory B2C

La otra forma de migración que hay que tener en cuenta es B2C de Azure AD. Azure AD B2C es un servicio de autenticación en la nube que, de modo similar a Access Control, permite a los desarrolladores externalizar la lógica de administración de identidades y la autenticación a un servicio en la nube. Es un servicio de pago (con niveles Premium y gratuito) diseñado para aquellas aplicaciones orientadas al cliente que pueden tener hasta millones de usuarios activos.

Al igual que Access Control, una de las características más atractivas de Azure AD B2C es que admite muchos tipos diferentes de cuentas. Gracias a Azure AD B2C, los usuarios pueden iniciar sesión mediante una cuenta de Microsoft, Facebook, Google, LinkedIn, GitHub o Yahoo entre otras. Además, Azure AD B2C admite "cuentas locales" o nombres de usuario y contraseñas que los usuarios crean específicamente para la aplicación. Azure AD B2C también proporciona una amplia extensibilidad que puede usar para personalizar los flujos de inicio de sesión. 

Sin embargo, Azure AD B2C no es compatible con la amplitud de protocolos de autenticación y formatos de token que pueden necesitar los clientes de Access Control. Tampoco se puede usar Azure AD B2C para obtener tokens y consultar información adicional sobre el usuario del proveedor de identidad, Microsoft o de otro modo.

En la siguiente tabla se comparan las características de Access Control pertinentes a las aplicaciones web, con aquellas que están disponibles en Azure AD B2C. En general, *B2C de Azure AD es, probablemente, la elección correcta para la migración si su aplicación está orientada hacia el consumidor, o si admite muchos tipos diferentes de cuentas.*

| Funcionalidad | Soporte técnico de Access Control | Compatibilidad de Azure AD B2C |
| ---------- | ----------- | ---------------- |
| **Tipos de cuentas** | | |
| Cuentas educativas o profesionales de Microsoft | Compatible | Compatible a través de directivas personalizadas  |
| Cuentas de Windows Server Active Directory y AD FS | Compatible a través de la federación directa con AD FS | Compatible a través de federación de SAML mediante directivas personalizadas |
| Cuentas desde otros sistemas de administración de identidades empresariales | Compatible a través de la federación directa mediante WS-Federation | Compatible a través de federación de SAML mediante directivas personalizadas |
| Cuentas de Microsoft para uso personal | Compatible | Compatible | 
| Cuentas de Facebook, Google, Yahoo | Compatible | Facebook y Google se admiten de forma nativa, Yahoo se admite a través de la federación de OpenID Connect mediante directivas personalizadas. |
| **Compatibilidad con el SDK y protocolos** | | |
| Windows Identity Foundation (WIF) | Compatible | No compatible |
| WS-Federation | Compatible | No compatible |
| OAuth 2.0 | Soporte para Draft 13 | Compatibilidad con RFC 6749, la especificación más moderna. |
| WS-Trust | Compatible | No compatible |
| **Formatos de tokens** | | |
| JWT | Compatible en versión beta | Compatible |
| SAML 1.1 | Compatible | No compatible |
| SAML 2.0 | Compatible | No compatible |
| SWT | Compatible | No compatible |
| **Personalizaciones** | | |
| UI de selección de cuenta o detección de dominio de inicio personalizable | Código descargable que se puede incorporar en las aplicaciones | Interfaz de usuario totalmente personalizable a través de CSS personalizadas |
| Carga de certificados de firma de tokens personalizados | Compatible | Las claves de firma personalizadas, no los certificados, se admiten a través de directivas personalizadas |
| Personalización de notificaciones en tokens |- Paso a través de notificaciones de entrada desde proveedores de identidad<br />- Obtención de token de acceso del proveedor de identidades como una notificación<br />- Emisión de notificaciones de salida basadas en valores de las notificaciones de entrada<br />- Emisión de notificaciones de salida con valores constantes |- Puede pasar a través de notificaciones de proveedores de identidad; directivas personalizadas necesarias para algunas notificaciones<br />- No se puede obtener el token de acceso del proveedor de identidades como una notificación<br />- No se pueden emitir notificaciones de salida según los valores de las notificaciones de entrada a través de directivas personalizadas<br />- Se pueden emitir notificaciones de salida con valores constantes a través de directivas personalizadas |
| **Automation** | | |
| Automatización de las tareas de configuración y administración | Compatible a través del servicio de administración de Access Control |- Creación de usuarios con permiso a través de Graph API de Azure AD<br />- No se pueden crear mediante programación las directivas, las aplicaciones ni los inquilinos de B2C |

Si decide que Azure AD B2C es la forma adecuada para migrar sus aplicaciones y servicios, debe comenzar con los recursos siguientes:

- [Documentación de Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-overview)
- [Directivas personalizadas de Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-overview-custom)
- [Precios de Azure AD B2C](https://azure.microsoft.com/pricing/details/active-directory-b2c/)

#### <a name="migrate-to-ping-identity-or-auth0"></a>Migrar a la identidad de Ping o Auth0

En algunos casos, es posible que Azure AD y Azure AD B2C no sean suficientes para reemplazar Access Control en las aplicaciones web sin tener que realizar cambios importantes en el código. Algunos ejemplos comunes pueden incluir:

- Aplicaciones web que usen WIF o WS-Federation para el inicio de sesión con proveedores de identidades sociales como Google o Facebook.
- Aplicaciones web que realicen una federación directa a un proveedor de identidades de empresa mediante el protocolo WS-Federation.
- Aplicaciones web que requieren el token de acceso que emite un proveedor de identidades sociales (como Google o Facebook) como una notificación en los tokens que emite Access Control.
- Aplicaciones web con reglas complejas de transformación de tokens que Azure AD o Azure AD B2C no pueden reproducir.
- Aplicaciones web de varios inquilinos que utilizan ACS para administrar de forma centralizada la federación de varios proveedores de identidades.

En estos casos, conviene considerar la posibilidad de migrar la aplicación web a otro servicio de autenticación en la nube. Le recomendamos que tenga en cuenta las opciones siguientes. Cada una de ellas ofrece capacidades similares a Access Control:

|     |     | 
| --- | --- |
| ![Auth0](./media/active-directory-acs-migration/rsz_auth0.png) | [Auth0](https://auth0.com/acs) es un servicio de identidad en la nube flexible que ha creado una [guía de migración de alto nivel para los clientes de Access Control](https://auth0.com/acs) y es compatible con casi todas las características de ACS. |
| ![Ping](./media/active-directory-acs-migration/rsz_ping.png) | La [identidad de Ping](https://www.pingidentity.com) le ofrece dos soluciones similares a ACS. PingOne es un servicio de identidad en la nube que admite muchas de las mismas características que ACS, y PingFederate es similar en el producto de identidad local que ofrece más flexibilidad. Si quiere obtener más detalles acerca de cómo usar estos productos, consulte [Ping's ACS retirement guidance](https://www.pingidentity.com/en/company/blog/2017/11/20/migrating_from_microsoft_acs_to_ping_identity.html) (Guía de retirada de ACS de Ping). |

Nuestro objetivo al trabajar con la identidad de Ping y Auth0 es asegurarnos de que todos los clientes de Access Control tengan una ruta de migración que minimice la cantidad de trabajo necesario para mover las aplicaciones y servicios de Access Control.

<!--

## SharePoint 2010, 2013, 2016

TODO: Azure AD only, use Azure AD SAML 1.1 tokens, when we bring it back online.
Other IDPs: use Auth0? https://auth0.com/docs/integrations/sharepoint.

-->

### <a name="web-services-that-use-active-authentication"></a>Servicios web que usan la autenticación activa

Access Control ofrece las siguientes características y funcionalidades para los servicios web que están protegidos con tokens que el mismo Access Control emitió:

- La capacidad de registrar una o varias *identidades de servicio* en el espacio de nombres de Access Control. Las identidades de servicio se pueden usar para solicitar tokens.
- Compatibilidad con los protocolos OAuth WRAP y OAuth 2.0 Draft 13 para solicitar tokens mediante los siguientes tipos de credenciales:
    - Una contraseña sencilla creada para la identidad del servicio.
    - Un SWT firmado mediante una clave simétrica o un certificado X509.
    - Un token SAML que emitió un proveedor de identidad de confianza (normalmente una instancia de AD FS).
- Compatibilidad con los siguientes formatos de token: JWT, SAML 1.1, SAML 2.0 y SWT.
- Reglas simples de transformación de token.

Las identidades de servicio de Access Control se suelen usar para implementar la autenticación de servidor a servidor (S2S). 

#### <a name="migrate-to-azure-active-directory"></a>Migración a Azure Active Directory

Nuestra recomendación para este tipo de flujo de autenticación es migrar a [Azure Active Directory](https://azure.microsoft.com/develop/identity/signin/). Azure AD es el proveedor de identidades basado en la nube de cuentas profesionales o educativas de Microsoft. Azure AD es el proveedor de identidades para Office 365, Azure y muchos servicios más. 

Asimismo, también puede utilizar Azure AD para realizar la autenticación de un servidor a otro mediante la implementación de Azure AD para la concesión de credenciales de cliente OAuth. En la tabla siguiente se comparan las funcionalidades de Access Control en la autenticación de servidor a servidor, con aquellas que están disponibles en Azure AD.

| Funcionalidad | Soporte técnico de Access Control | Soporte técnico de Azure AD |
| ---------- | ----------- | ---------------- |
| Procedimiento para registrar un servicio web | Crear un usuario de confianza en el portal de administración de Access Control | Crear una aplicación web de Azure AD en Azure Portal |
| Procedimiento de registro de un cliente | Crear una identidad de servicio en el portal de administración de Access Control | Crear otra aplicación web de Azure AD en Azure Portal |
| Protocolo usado |- Protocolo WRAP de OAuth<br />- Concesión de credenciales del cliente de OAuth 2.0 Draft 13 | Concesión de credenciales del cliente de OAuth 2.0 |
| Métodos de autenticación del cliente |- Contraseña simple<br />- SWT firmado<br />- Token SAML del proveedor de identidades federadas |- Contraseña simple<br />- SWT firmado |
| Formatos de tokens |- JWT<br />- SAML 1.1<br />- SAML 2.0<br />- SWT<br /> | Solo JWT |
| Transformación de token |- Incorporación de notificaciones personalizadas<br />- Lógica de emisión de notificaciones if-then simple | Incorporación de notificaciones personalizadas | 
| Automatización de las tareas de configuración y administración | Compatible a través del servicio de administración de Access Control | Compatible a través de Microsoft Graph y Graph API de Azure AD. |

Para obtener información sobre escenarios de implementación entre servidores, consulte los siguientes recursos:

- Sección Servicio a servicio de la [Guía del desarrollador de Azure AD](https://aka.ms/aaddev)
- [Daemon code sample by using simple password client credentials](https://github.com/Azure-Samples/active-directory-dotnet-daemon) (Ejemplo de código Daemon que usa credenciales de cliente con contraseñas simples)
- [Daemon code sample by using certificate client credentials](https://github.com/Azure-Samples/active-directory-dotnet-daemon-certificate-credential) (Ejemplo de código Daemon que usa credenciales de cliente con certificados)

#### <a name="migrate-to-ping-identity-or-auth0"></a>Migrar a la identidad de Ping o Auth0

En algunos casos, es posible que las credenciales del cliente de Azure AD y la concesión de implementación de OAuth no sean suficientes para reemplazar Access Control en la arquitectura sin tener que realizar cambios importantes en el código. Algunos ejemplos comunes pueden incluir:

- Autenticación de servidor a servidor mediante formatos de token distintos de los JWT.
- Autenticación de servidor a servidor mediante un token de entrada que proporciona un proveedor de identidades externo.
- Autenticación de servidor a servidor con las reglas de transformación de tokens que Azure AD no puede reproducir.

En estos casos, conviene considerar la posibilidad de migrar la aplicación web a otro servicio de autenticación en la nube. Le recomendamos que tenga en cuenta las opciones siguientes. Cada una de ellas ofrece capacidades similares a Access Control:

|     |     | 
| --- | --- |
| ![Auth0](./media/active-directory-acs-migration/rsz_auth0.png) | [Auth0](https://auth0.com/acs) es un servicio de identidad en la nube flexible que ha creado una [guía de migración de alto nivel para los clientes de Access Control](https://auth0.com/acs) y es compatible con casi todas las características de ACS. |
| ![Ping](./media/active-directory-acs-migration/rsz_ping.png) | La [identidad de Ping](https://www.pingidentity.com) le ofrece dos soluciones similares a ACS. PingOne es un servicio de identidad en la nube que admite muchas de las mismas características que ACS, y PingFederate es similar en el producto de identidad local que ofrece más flexibilidad. Si quiere obtener más detalles acerca de cómo usar estos productos, consulte [Ping's ACS retirement guidance](https://www.pingidentity.com/en/company/blog/2017/11/20/migrating_from_microsoft_acs_to_ping_identity.html) (Guía de retirada de ACS de Ping). |

Nuestro objetivo al trabajar con la identidad de Ping y Auth0 es asegurarnos de que todos los clientes de Access Control tengan una ruta de migración que minimice la cantidad de trabajo necesario para mover las aplicaciones y servicios de Access Control.

#### <a name="passthrough-authentication"></a>Autenticación de paso a través

Para la autenticación de paso a través con la transformación de tokens arbitraria, no hay ninguna tecnología de Microsoft equivalente a ACS. Si eso es lo que necesitan los clientes, Auth0 puede que sea quien proporciona la solución de aproximación más cercana.

## <a name="questions-concerns-and-feedback"></a>Preguntas, problemas y comentarios

Somos conscientes de que muchos clientes de Access Control no encontrarán una ruta de acceso de migración clara después de leer este artículo. Es posible que necesite un poco más de ayuda para poder determinar el plan correcto que debe seguir. Si desea analizar los escenarios de migración y otras cuestiones, deje un comentario en esta página.
