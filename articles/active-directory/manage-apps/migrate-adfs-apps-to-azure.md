---
title: Mover aplicaciones de AD FS a Azure AD | Microsoft Docs
description: Este artículo tiene como finalidad ayudar a las organizaciones a comprender cómo mover aplicaciones a Azure AD, centrándose en las aplicaciones SaaS federadas.
services: active-directory
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: conceptual
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 03/02/2018
ms.author: mimart
ms.collection: M365-identity-device-management
ms.openlocfilehash: f77e322ffd7eec78fe13650f40c93f914706d557
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65824620"
---
# <a name="move-applications-from-ad-fs-to-azure-ad"></a>Mover aplicaciones de AD FS a Azure AD 

Este artículo le ayudará a comprender cómo mover aplicaciones de AD FS a Azure Active Directory (Azure AD). Se centra en las aplicaciones SaaS federadas. 

Este artículo no proporciona instrucciones paso a paso. Se proporciona una guía conceptual para ayudarle en la migración y que pueda comprender como convertir las configuraciones locales a Azure AD. También trata escenarios comunes.

## <a name="introduction"></a>Introducción

Si tiene un directorio local que contiene cuentas de usuario, lo más probable es que tenga al menos una o dos aplicaciones. Y estas aplicaciones están configuradas para que los usuarios accedan mediante inicio de sesión con esas identidades.

Si su organización es como la mayoría de las organizaciones, es probable que en algún momento del camino adopte aplicaciones e identidades en la nube. Quizás ya trabaja con Office 365 y Azure AD Connect. Puede que haya configurado aplicaciones SaaS basadas en la nube para algunas cargas de trabajo principales, pero no todas ellas.  

Muchas organizaciones tienen aplicaciones SaaS o aplicaciones de línea de negocio (LOB) personalizadas que están federadas directamente con un servicio de inicio de sesión local como Servicios de federación de Active Directory (AD FS), además de Office 365 y aplicaciones basadas en Azure AD. En esta guía se describe por qué y cómo mover las aplicaciones a Azure AD.

>[!NOTE]
>Esta guía proporciona información detallada sobre la configuración y la migración de aplicaciones SaaS, con información general sobre las aplicaciones de línea de negocio personalizadas. En un futuro se podrá disponer de instrucciones detalladas sobre las aplicaciones de línea de negocio personalizadas.

![Aplicaciones conectadas directamente en un entorno local](media/migrate-adfs-apps-to-azure/migrate1.png)

![Aplicaciones federadas a través de Azure AD](media/migrate-adfs-apps-to-azure/migrate2.png)

## <a name="reasons-for-moving-apps-to-azure-ad"></a>Razones para mover aplicaciones a Azure AD

En organizaciones que ya usan AD FS, Ping u otro proveedor de autenticación local, mover las aplicaciones a Azure AD ofrece las siguientes ventajas:

**Acceso más seguro**
- Configuración pormenorizada de los controles de acceso por aplicación, incluido Azure Multi-Factor Authentication, mediante el [acceso condicional de Azure AD](../active-directory-conditional-access-azure-portal.md). Las directivas pueden aplicarse a aplicaciones SaaS y personalizadas de la misma manera que lo podría estar haciendo actualmente con Office 365.
- Para detectar amenazas y proteger el inicio de sesión, basado en el aprendizaje automático y la heurística que identifican el tráfico de riesgo, aproveche las ventajas de [Azure AD Identity Protection](../active-directory-identityprotection.md).

**Colaboración B2B de Azure AD**
- Una vez que el inicio de sesión en aplicaciones SaaS se basa en Azure AD, puede conceder a los asociados acceso a los recursos en la nube con la [colaboración B2B de Azure AD](../b2b/what-is-b2b.md).

**Experiencia de administración más sencilla y funcionalidades adicionales de Azure AD**
- Azure AD, como un proveedor de identidades para aplicaciones SaaS, admite funcionalidades adicionales, como:
  - Certificados de firma de tokens por aplicación.
  - [Fechas de expiración de certificado configurables](manage-certificates-for-federated-single-sign-on.md).
  - [Aprovisionamiento automático](user-provisioning.md) de cuentas de usuario (en las principales aplicaciones de Azure Marketplace) basada en identidades de Azure AD.

**Se mantienen las ventajas de un proveedor de identidades local**
- Mientras aprovecha las ventajas de Azure AD, puede seguir usando su solución local para la autenticación. De este modo, sigue contando con ventajas como las soluciones de autenticación multifactor locales, el registro y la auditoría. 

**Ayuda en la retirada del proveedor de identidades local**
- Para las organizaciones que buscan retirar el producto de autenticación local, migrar las aplicaciones a Azure AD permite una transición fácil al tener ya hecho parte del trabajo. 

## <a name="mapping-types-of-apps-on-premises-to-types-of-apps-in-azure-ad"></a>Asignación de tipos de aplicaciones locales a tipos de aplicaciones de Azure AD
La mayoría de las aplicaciones encajan en unas cuantas categorías basadas en el tipo de inicio de sesión que usan. Estas categorías determinan cómo se representa la aplicación en Azure AD.

En resumen, las aplicaciones SAML 2.0 se pueden integrar con Azure AD a través de la galería de aplicaciones de Azure AD en Marketplace o como aplicaciones que no son de Marketplace. Las aplicaciones que usan OAuth 2.0 u OpenID Connect se pueden integrar con Azure AD de forma parecida a los *registros de aplicación*. Siga leyendo para más información.

### <a name="federated-saas-apps-vs-custom-lob-apps"></a>Aplicaciones SaaS federadas frente a aplicaciones de línea de negocio personalizadas
Las aplicaciones federadas incluyen aplicaciones que se dividen en estas categorías:

- Aplicaciones SaaS 
    - Si los usuarios inician sesión en aplicaciones SaaS como Salesforce, ServiceNow o Workday y se va a realizar la integración con un proveedor de identidades local, como AD FS o Ping, se usará el inicio de sesión federado para aplicaciones SaaS.
    - Por lo general, las aplicaciones usan el protocolo SAML 2.0 para el inicio de sesión federado.
    - Las aplicaciones que entran en esta categoría se pueden integrar con Azure AD como aplicaciones de empresa, bien desde Marketplace o como aplicaciones que no son de Marketplace.
- Aplicaciones de línea de negocio personalizadas
    - Son aplicaciones que no son SaaS, que la organización desarrolla internamente o que están disponibles como un producto empaquetado estándar que se instala en el centro de datos. Se incluyen las aplicaciones de SharePoint y las aplicaciones basadas en Windows Identity Foundation.
    - Las aplicaciones pueden usar SAML 2.0, WS-Federation, OAuth u OpenID Connect para el inicio de sesión federado.
    - Las aplicaciones personalizadas que usan OAuth 2.0, OpenID Connect o WS-Federation se pueden integrar con Azure AD como registros de aplicación. Las aplicaciones personalizadas que utilizan SAML 2.0 o WS-Federation se pueden integrar como aplicaciones que no son de Marketplace dentro de las aplicaciones empresariales.

### <a name="non-federated-apps"></a>Aplicaciones no federadas
Es posible integrar aplicaciones no federadas con Azure AD mediante Azure AD Application Proxy y las funcionalidades relacionadas. Las aplicaciones no federadas incluyen:
- Aplicaciones que usan Windows Integrated Authentication directamente con Active Directory. Estas aplicaciones se pueden integrar con Azure AD mediante [Azure AD Application Proxy](application-proxy-add-on-premises-application.md).
- Aplicaciones que se integran con el proveedor de inicio de sesión único mediante un agente y que usan encabezados para la autorización. Las aplicaciones locales que usan un agente instalado para el inicio de sesión y la autorización basada en encabezados se pueden configurar para el inicio de sesión basado en Azure AD mediante Azure AD Application Proxy con [Ping Access for Azure AD](https://blogs.technet.microsoft.com/enterprisemobility/2017/06/15/ping-access-for-azure-ad-is-now-generally-available-ga/).

## <a name="translating-on-premises-federated-apps-to-azure-ad"></a>Conversión de aplicaciones federadas locales a Azure AD 
AD FS y Azure AD funcionan de forma similar, de modo que los conceptos de configuración de direcciones URL e identificadores de confianza de inicio y cierre de sesión se aplican en ambos casos. Sin embargo, debe entender algunas pequeñas diferencias al llevar a cabo la transición.

Las tablas siguientes asignan las ideas principales que comparten las aplicaciones de AD FS, Azure AD y SaaS para ayudarle en la transición. 

### <a name="representing-the-app-in-azure-ad-or-ad-fs"></a>Representación de la aplicación en Azure AD o AD FS
La migración comienza con la evaluación de cómo está configurada la aplicación en el entorno local y la asignación de esa configuración a Azure AD. La tabla siguiente es una asignación de los elementos de configuración de usuario de confianza de AD FS a los elementos de Azure AD correspondientes.  
- Término de AD FS: Usuario de confianza o relación de confianza de usuario autenticado.
- Términos de Azure AD: Aplicación empresarial o registro de aplicación (en función del tipo de aplicación).

|Elemento de configuración de aplicación|DESCRIPCIÓN|Ubicación en la configuración de AD FS|Ubicación correspondiente en la configuración de Azure AD|Elemento de token de SAML|
|-----|-----|-----|-----|-----|
|URL de inicio de sesión de aplicación|URL de la página de inicio de sesión de esta aplicación. Aquí es donde el usuario iría a iniciar sesión en la aplicación en un flujo de SAML iniciado por el SP.|N/D|En Azure AD, la dirección URL de inicio de sesión se configura en Azure Portal, en las propiedades de **inicio de sesión único** de la aplicación, como dirección URL de inicio de sesión.</br></br>(Puede que tenga que seleccionar **Mostrar configuración avanzada de URL** para ver la dirección URL de inicio de sesión).|N/D|
|Dirección URL de respuesta de la aplicación|Dirección URL de la aplicación desde la perspectiva del proveedor de identidades (IdP). Aquí es donde se envían el usuario y el token una vez que el usuario ha iniciado sesión en el IdP.</br></br> Esto se conoce también como el "punto de conexión del consumidor de aserción de SAML".|Se encuentra en la relación de confianza de usuario autenticado de AD FS de la aplicación. Haga clic con el botón derecho en el usuario de confianza, seleccione **Propiedades** y, a continuación, seleccione la pestaña **Puntos de conexión**.|En Azure AD, la dirección URL de respuesta se configura en Azure Portal, en las propiedades de **Inicio de sesión único** de la aplicación, como dirección URL de respuesta.</br></br>(Puede que tenga que seleccionar **Mostrar configuración avanzada de URL** para ver la dirección URL de respuesta).|Se asigna al elemento **Destino** en el token SAML.</br></br> Valor de ejemplo: `https://contoso.my.salesforce.com`|
|Dirección URL de cierre de sesión de la aplicación|La dirección URL a la que se envían solicitudes de "limpieza de cierre de sesión" cuando un usuario cierra la sesión en una aplicación, con el fin de cerrar la sesión de todas las demás aplicaciones en las que el IDP ha iniciado la sesión del usuario.|Se encuentra en Administración de AD FS, en **Relaciones de confianza de usuario autenticado**. Haga clic con el botón derecho en el usuario de confianza, seleccione **Propiedades** y, a continuación, seleccione la pestaña **Puntos de conexión**.|N/D Azure AD no admite el "cierre de sesión único", en el sentido de cerrar la sesión de todas las aplicaciones. Simplemente se cierra la sesión del usuario de Azure AD.|N/D|
|Identificador de la aplicación|Identificador de la aplicación desde la perspectiva del IdP. El valor de la dirección URL de inicio de sesión se usa a menudo (pero no siempre) como identificador.</br></br> En ocasiones, la aplicación lo llama "identificador de entidad".|En AD FS, se trata del identificador del usuario de confianza. Haga clic con el botón derecho en la relación de confianza de usuario autenticado, seleccione **Propiedades** y, a continuación, seleccione la pestaña **Identificadores**.|En Azure AD, el identificador se configura en Azure Portal en las propiedades de **inicio de sesión único** de la aplicación como Identificador en **Dominio y direcciones URL**. (Tendrá que seleccionar la casilla de verificación **Mostrar configuración avanzada de direcciones URL**.)|Se corresponde con el elemento **Audiencia** del token SAML.|
|Metadatos de federación de la aplicación|Ubicación de los metadatos de federación de la aplicación. El IdP la usa para actualizar automáticamente valores de configuración específicos, como puntos de conexión o certificados de cifrado.|La dirección URL de metadatos de federación de la aplicación se encuentra en la relación de confianza de usuario autenticado de AD FS de la aplicación. Haga clic con el botón derecho en la relación de confianza, seleccione **Propiedades** y, a continuación, seleccione la pestaña **Supervisión**.|N/D Azure AD no admite el consumo directo de metadatos de federación de aplicaciones.|N/D|
|Identificador de usuario/**NameID**|Atributo que se usa para indicar de forma única la identidad de usuario de Azure AD o AD FS para la aplicación.</br></br> Normalmente, este atributo es el UPN o la dirección de correo electrónico del usuario.|En AD FS, se encuentra como una regla de notificación en el usuario de confianza. En la mayoría de los casos, la regla de notificación emite una notificación con un tipo que termina con "nameidentifier".|En Azure AD, el identificador de usuario puede encontrarse en Azure Portal en las propiedades de **inicio de sesión único** de la aplicación en el encabezado **Atributos de usuario**.</br></br>De forma predeterminada, se usa el UPN.|Se comunica desde el IdP a la aplicación como el elemento **NameID** en el token SAML.|
|Otras notificaciones que se envían a la aplicación|Además del identificador de usuario/**NameID**, normalmente se envía otra información de notificación desde el IdP a la aplicación. Por ejemplo, el nombre, los apellidos, la dirección de correo electrónico y los grupos de los que forma parte el usuario.|En AD FS, se encuentra como otras reglas de notificación en el usuario de confianza.|En Azure AD, se puede encontrar en Azure Portal en las propiedades de **inicio de sesión único** de la aplicación en el encabezado **Atributos de usuario**. Seleccione **Ver** para editar todos los demás atributos de usuario.|N/D|  

### <a name="representing-azure-ad-as-an-identity-provider-in-an-saas-app"></a>Representación de Azure AD como un proveedor de identidades en una aplicación SaaS
Como parte de la migración, la aplicación se debe configurar para que apunte a Azure AD (en lugar de al proveedor de identidades local). Esta sección se centra principalmente en aplicaciones SaaS que usan el protocolo SAML y no en las aplicaciones de línea de negocio personalizadas. Sin embargo, los conceptos descritos se pueden extender a las aplicaciones de línea de negocio personalizadas. 

En un nivel general, hay algunos elementos clave que hay que tener en cuenta al apuntar una aplicación SaaS a Azure AD:
- Emisor del proveedor de identidades: https&#58;//sts.windows.net/{tenant-id}/
- Dirección URL de inicio de sesión del proveedor de identidades: https&#58;//login.microsoftonline.com/{tenant-id}/saml2
- Dirección URL de cierre de sesión del proveedor de identidades: https&#58;//login.microsoftonline.com/{tenant-id}/saml2 
- Ubicación de los metadatos de federación: https&#58;//login.windows.net/{tenant-id}/federationmetadata/2007-06/federationmetadata.xml?appid={application-id} 

Reemplace {tenant-id} por el identificador de inquilino que puede encontrar en Azure Portal en **Azure Active Directory** > **Propiedades** como **Identificador de directorio**. Reemplace {application-id} por el identificador de la aplicación que puede encontrar en las propiedades de la aplicación como **Identificador de la aplicación**.

En la siguiente tabla se describen los elementos de configuración principales del IdP para configurar los valores de inicio de sesión único en la aplicación y sus valores o ubicaciones dentro de AD FS y Azure AD. El marco de referencia de la tabla es la aplicación SaaS, que necesita saber dónde enviar las solicitudes de autenticación y cómo validar los tokens recibidos.

|Elemento configuración|DESCRIPCIÓN|AD FS|Azure AD|
|---|---|---|---|
|URL de </br>inicio de sesión del </br>Dirección URL|Dirección URL de inicio de sesión del IdP desde la perspectiva de la aplicación (donde se redirige al usuario para el inicio de sesión).|La dirección URL de inicio de sesión de AD FS es el nombre del servicio de federación de AD FS seguido de "/adfs/ls/". Por ejemplo: https&#58;//fs.contoso.com/adfs/ls/|El valor correspondiente para Azure AD sigue el patrón en el que {tenant-id} se reemplaza por el identificador del inquilino. Lo puede encontrar en Azure Portal en **Azure Active Directory** > **Propiedades** como **Identificador de directorio**.</br></br>Para aplicaciones que usan el protocolo SAML-P: https&#58;//login.microsoftonline.com/{tenant-id}/saml2 </br></br>Para aplicaciones que usan el protocolo WS-Federation: https&#58;//login.microsoftonline.com/{tenant-id}/wsfed|
|URL de </br>cierre de sesión </br>Dirección URL|Dirección URL de cierre de sesión del IdP desde la perspectiva de la aplicación (donde se redirige al usuario cuando elige cerrar sesión en la aplicación).|En AD FS, la dirección URL de cierre de sesión es la misma que la dirección URL de inicio de sesión o la misma dirección URL seguida de "wa=wsignout1.0". Por ejemplo: https&#58;//fs.contoso.com/adfs/ls/?wa=wsignout1.0|El valor correspondiente para Azure AD depende de si la aplicación admite el cierre de sesión con SAML 2.0.</br></br>Si la aplicación admite el cierre de sesión con SAML, el valor sigue el patrón en el que el valor de {tenant-id} se reemplaza por el identificador del inquilino. Lo puede encontrar en Azure Portal en **Azure Active Directory** > **Propiedades** como **Identificador de directorio**: https&#58;//login.microsoftonline.com/{tenant-id}/saml2</br></br>Si la aplicación no admite el cierre de sesión con SAML: https&#58;//login.microsoftonline.com/common/wsfederation?wa=wsignout1.0|
|Token </br>de firma </br>certificado|Certificado cuya clave privada usa el IDP para firmar los tokens emitidos. Se comprueba que el token procede del mismo IdP que se ha configurado como de confianza en la aplicación.|El certificado de firma de token de AD FS se encuentra en Administración de AD FS, en **Certificados**.|En Azure AD, el certificado de firma de token se puede encontrar en Azure Portal, en las propiedades de **inicio de sesión único** de la aplicación, bajo el encabezado **Certificado de firma de SAML**. Desde allí puede descargar el certificado para cargarlo en la aplicación.</br></br> Si la aplicación tiene más de un certificado, todos los certificados se pueden encontrar en el archivo XML de metadatos de federación.|
|Identificador/</br>"emisor"|Identificador del IdP desde la perspectiva de la aplicación (también llamado "Identificador de emisor").</br></br>En el token SAML, el valor aparece como el elemento **Emisor**.|El identificador de AD FS normalmente es el identificador del servicio de federación en Administración de AD, en **Servicio** > **Modificar las propiedades del Servicio de federación**. Por ejemplo: http&#58;//fs.contoso.com/adfs/services/trust|El valor correspondiente para Azure AD sigue el patrón en el que el valor de {tenant-id} se reemplaza por el identificador del inquilino. Lo puede encontrar en Azure Portal en **Azure Active Directory** > **Propiedades** como **Identificador de directorio**: https&#58;//sts.windows.net/{tenant-id}/|
|URL de </br>federación </br>metadatos|Ubicación de los metadatos de federación disponibles públicamente del IdP. (Algunas aplicaciones usan metadatos de federación como alternativa a la configuración de direcciones URL, identificadores y certificados de firma de token de forma individual por el administrador).|Puede encontrar la URL de metadatos de federación de AD FS en Administración de AD FS en **Servicio** > **Puntos de conexión** > **Metadatos** >  **Tipo: Metadatos de federación**. Por ejemplo: https&#58;//fs.contoso.com/FederationMetadata/2007-06/FederationMetadata.xml|El valor correspondiente para Azure AD sigue el patrón https&#58;//login.microsoftonline.com/{TenantDomainName}/FederationMetadata/2007-06/FederationMetadata.xml. El valor de {TenantDomainName} se reemplaza por el nombre del inquilino con el formato "contoso.onmicrosoft.com". </br></br>Para más información, consulte [Metadatos de federación](../develop/azure-ad-federation-metadata.md).

## <a name="moving-saas-apps"></a>Migración de aplicaciones SaaS
El movimiento de aplicaciones SaaS desde AD FS u otro proveedor de identidades a Azure AD es actualmente un proceso manual. Para obtener instrucciones específicas de la aplicación, consulte la [lista de tutoriales sobre la integración de aplicaciones SaaS que se encuentra en Marketplace](../saas-apps/tutorial-list.md).

En los tutoriales de integración se supone que va a realizar una integración green field. Hay unos cuantos conceptos clave específicos de la migración que debe conocer mientras planea, evalúa, configura y migra completamente las aplicaciones:  
- Algunas aplicaciones se pueden migrar fácilmente. Las aplicaciones con requisitos más complejos, como las notificaciones personalizadas, pueden requerir configuración adicional en Azure AD o Azure AD Connect.
- En los escenarios más comunes, solo la notificación **NameID** y otras notificaciones de identificador de usuario comunes son necesarias para una aplicación. Para determinar si se requieren notificaciones adicionales, examine qué notificaciones está emitiendo desde AD FS o desde el proveedor de identidades de terceros.
- Después de determinar qué notificaciones adicionales se necesitan, asegúrese de que están disponibles en Azure AD. Compruebe la configuración de sincronización de Azure AD Connect para asegurarse de que los atributos necesarios, por ejemplo **samAccountName**, se sincronizan con Azure AD.
- Cuando los atributos están disponibles en Azure AD, puede agregar reglas de emisión de notificaciones en Azure AD para incluir esos atributos como notificaciones en los tokens emitidos. Estas reglas se agregan en las propiedades de **inicio de sesión único** de la aplicación en Azure AD.

### <a name="assess-what-can-be-moved"></a>Evaluación de lo que se puede mover
Las aplicaciones SAML 2.0 se pueden integrar con Azure AD a través de la galería de aplicaciones de Azure AD en Marketplace o como aplicaciones que no son de Marketplace.  

Algunas configuraciones requieren pasos adicionales de configuración en Azure AD y otras no se admiten actualmente. Para determinar lo que se puede trasladar, examine la configuración actual de cada una de las aplicaciones. En concreto, busque lo siguiente:
- Reglas de notificación configuradas (reglas de transformación de emisión).
- Formato y atributo de **NameID** de SAML.
- Versiones de token SAML emitidas.
- Otras configuraciones, como las reglas de autorización de emisión o las directivas de control de acceso y las reglas de autenticación multifactor (autenticación adicional).

#### <a name="what-can-be-moved-today"></a>¿Qué se puede mover actualmente?
Las aplicaciones que se pueden mover fácilmente en la actualidad incluyen las aplicaciones SAML 2.0 que usan el conjunto estándar de notificaciones y elementos de configuración. Estas aplicaciones pueden constar de:
- Nombre principal de usuario.
- Dirección de correo electrónico.
- Nombre propio.
- Apellido.
- Atributo alternativo de **NameID** de SAML, lo que incluye el atributo de correo de Azure AD, el prefijo de correo, el identificador de empleado, los atributos de extensión 1 a 15 o el atributo **SamAccountName** local. Para más información, consulte [Edición de la notificación NameIdentifier](../develop/active-directory-saml-claims-customization.md).
- Notificaciones personalizadas. Para obtener información acerca de las asignaciones de notificaciones admitidas, consulte [Asignación de notificaciones en Azure Active Directory](../develop/active-directory-claims-mapping.md) y [Personalización de las notificaciones emitidas en el token SAML para aplicaciones empresariales en Azure Active Directory](../develop/active-directory-saml-claims-customization.md).

Además de las notificaciones y los elementos **NameID** personalizados, las configuraciones que requieren pasos adicionales en Azure AD como parte de la migración son:
- Autorización personalizada o reglas de autenticación multifactor en AD FS. Se configuran mediante el uso de la característica [Acceso condicional de Azure AD](../active-directory-conditional-access-azure-portal.md).
- Aplicaciones con varios puntos de conexión SAML. Se configuran en Azure AD mediante PowerShell. (Esta funcionalidad no está disponible en el portal).
- Aplicaciones de WS-Federation, como las aplicaciones de SharePoint, que requieren tokens de la versión 1.1 de SAML. Se deben configurar manualmente mediante el uso de PowerShell.

#### <a name="apps-and-configurations-not-supported-in-azure-ad-today"></a>Aplicaciones y configuraciones no admitidas actualmente en Azure AD
Las aplicaciones que necesitan las siguientes funcionalidades no se pueden migrar actualmente. Proporcione comentarios en la sección correspondiente si tiene aplicaciones que necesitan estas características.
- Funcionalidades de protocolo:
    - Compatibilidad con el cierre de sesión único (SLO) de SAML de todas las aplicaciones que han iniciado sesión.
    - Compatibilidad con el patrón ActAs de WS-Trust.
    - Resolución de artefactos SAML.
    - Comprobación de la firma de las solicitudes SAML firmadas. Tenga en cuenta que se aceptan solicitudes firmadas, pero no se comprueba la firma.
- Funcionalidades de token: 
    - Cifrado de token SAML. 
    - Token de la versión 1.1 de SAML en respuestas de protocolo SAML. 
- Funcionalidades de notificaciones en token:
    - Emisión de nombres de grupos locales como notificaciones.
    - Notificaciones de almacenes distintos de Azure AD.
    - Reglas de transformación de emisión de notificaciones complejas. Para obtener información acerca de las asignaciones de notificaciones admitidas, consulte [Asignación de notificaciones en Azure Active Directory](../develop/active-directory-claims-mapping.md) y [Personalización de las notificaciones emitidas en el token SAML para aplicaciones empresariales en Azure Active Directory](../develop/active-directory-saml-claims-customization.md).
    - Emisión de extensiones de directorio como notificaciones.
    - Especificación personalizada del formato de **NameID**.
    - Emisión de atributos de valores múltiples.

>[!NOTE]
>Azure AD evoluciona constantemente para agregar funcionalidades en este área. Actualizamos este artículo de forma regular. 

### <a name="configure-azure-ad"></a>Configuración de Azure AD    
#### <a name="configure-single-sign-on-sso-settings-for-the-saas-app"></a>Configuración del inicio de sesión único (SSO) para la aplicación SaaS

En Azure AD, puede configurar el inicio de sesión de SAML (tal como necesita la aplicación) en las propiedades de **Inicio de sesión único** de la aplicación, en **Atributos de usuario**.

![Propiedades de inicio de sesión único con la sección "Atributos de usuario"](media/migrate-adfs-apps-to-azure/migrate3.png)

Seleccione **Ver y editar todos los demás atributos de usuario** para ver los atributos que se envían como notificaciones en el token de seguridad.

![Lista de atributos](media/migrate-adfs-apps-to-azure/migrate4.png)

Seleccione una fila del atributo concreto para editar o seleccione **Agregar atributo** para agregar un nuevo atributo.

![Panel "Editar atributo"](media/migrate-adfs-apps-to-azure/migrate5.png)

#### <a name="assign-users-to-the-app"></a>Asignación de usuarios a la aplicación
Para que los usuarios de Azure AD puedan iniciar sesión en una aplicación SaaS, debe concederles acceso.

Para asignar los usuarios en el portal de Azure AD, vaya a la página de la aplicación SaaS y, a continuación, seleccione **Usuarios y grupos** en la barra lateral. Para agregar un usuario o un grupo, seleccione **Agregar usuario** en la parte superior del panel. 

![Botón "Agregar usuario" en "Usuarios y grupos"](media/migrate-adfs-apps-to-azure/migrate6.png) 

![Panel "Agregar asignación"](media/migrate-adfs-apps-to-azure/migrate7.png)

Para comprobar el acceso, los usuarios deben ver la aplicación SaaS en su [panel de acceso](../user-help/active-directory-saas-access-panel-introduction.md) cuando inician sesión. Pueden encontrar el panel de acceso en https://myapps.microsoft.com. En este ejemplo, se ha asignado acceso correctamente a un usuario en Salesforce y ServiceNow.

![Ejemplo de panel de acceso con las aplicaciones Salesforce y ServiceNow](media/migrate-adfs-apps-to-azure/migrate8.png)

### <a name="configure-the-saas-app"></a>Configuración de la aplicación SaaS
El proceso de traslado de federación local a Azure AD depende de si la aplicación SaaS con la que trabaja admite varios proveedores de identidades. Estas son algunas preguntas comunes sobre la compatibilidad con varios IdP:

   **P: ¿Qué significa que una aplicación admita varios IdP?**
    
   R: Las aplicaciones SaaS que admiten varios IdP le permiten proporcionar toda la información sobre el nuevo IdP (en nuestro caso, Azure AD) antes de confirmar el cambio de la experiencia de inicio de sesión. Después de realizar la configuración, puede cambiar la configuración de autenticación de la aplicación para que apunte a Azure AD.

   **P: ¿Por qué es importante que la aplicación SaaS admita varios IdP?**

   R: Si no se admiten varios IdP, el administrador tiene que reservar un breve período de tiempo como interrupción del servicio o de mantenimiento durante el cual puede configurar Azure AD como el nuevo IdP de una aplicación. Durante esta interrupción, se debe notificar a los usuarios que no podrán iniciar sesión con sus cuentas.

   Si una aplicación admite varios IdP, el IdP adicional se puede configurar de antemano. El administrador, a continuación, puede cambiar el IdP a Azure.

   Si la aplicación admite varios IdP y elige varios IdP para controlar la autenticación de inicio de sesión al mismo tiempo, el usuario puede elegir el IdP para autenticarse en su página de inicio de sesión.

#### <a name="example-support-for-multiple-idps"></a>Ejemplo: Compatibilidad con varios IdP
Por ejemplo, en Salesforce, puede encontrar la configuración del IdP en **Configuración** > **Configuración de la empresa** > **Mi dominio** >  **Configuración de autenticación**.

![Sección "Configuración de autenticación" en la aplicación Salesforce](media/migrate-adfs-apps-to-azure/migrate9.png)

Debido a la configuración creada anteriormente en **Identidad** > **Configuración de inicio de sesión único**, podrá cambiar el IdP para configurar la autenticación. Por ejemplo, puede cambiarlo de AD FS a Azure AD. 

![Selección de Azure AD como el servicio de autenticación](media/migrate-adfs-apps-to-azure/migrate10.png)

### <a name="optional-configure-user-provisioning-in-azure-ad"></a>Opcional: Configuración del aprovisionamiento de usuarios en Azure AD
Si desea que Azure AD controle directamente el aprovisionamiento de usuarios para una aplicación SaaS, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](user-provisioning.md).

## <a name="next-steps"></a>Pasos siguientes

- [Administración de aplicaciones con Azure Active Directory](what-is-application-management.md)
- [Administración del acceso a aplicaciones](what-is-access-management.md)
- [Federación de Azure AD Connect](../hybrid/how-to-connect-fed-whatis.md)
