---
title: Traslado de la autenticación de las aplicaciones de AD FS a Azure Active Directory
description: Este artículo tiene como finalidad ayudar a las organizaciones a comprender cómo mover aplicaciones a Azure AD, centrándose en las aplicaciones SaaS federadas.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: how-to
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 04/01/2020
ms.author: kenwith
ms.reviewer: baselden
ms.collection: M365-identity-device-management
ms.openlocfilehash: c9d2f295394d89432f3c6dd99585cc4363d4ff74
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88641371"
---
# <a name="moving-application-authentication-from-active-directory-federation-services-to-azure-active-directory"></a>Traslado de la autenticación de las aplicaciones de los Servicios de federación de Active Directory (AD FS) a Azure Active Directory

[Azure Active Directory (Azure AD)](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-whatis) ofrece una plataforma de identidad universal que proporciona a los usuarios, asociados y clientes una identidad única para acceder a las aplicaciones y colaborar desde cualquier plataforma y dispositivo. Azure AD tiene un [conjunto completo de funcionalidades de administración de identidades](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-whatis). La estandarización de la autorización y la autenticación de las aplicaciones para Azure AD habilita las ventajas que ofrecen estas funcionalidades. 

> [!NOTE]
> Este artículo se centra en el traslado de la autenticación de las aplicaciones desde Active Directory y los Servicios de federación de Active Directory (AD FS) locales a Azure AD. Consulte las notas del producto sobre la [Migración de la autenticación de aplicaciones a Azure AD](https://aka.ms/migrateapps/whitepaper) para ver información general sobre el planeamiento de este traslado. En las notas del producto se explica cómo planear la migración, las pruebas y las conclusiones.

## <a name="introduction"></a>Introducción

Si tiene un directorio local que contiene cuentas de usuario, es probable que tenga muchas aplicaciones en las que los usuarios se deben autenticar. Cada una de estas aplicaciones está configurada para que los usuarios obtengan acceso mediante sus identidades. 


Los usuarios también se pueden autenticar directamente con su instancia de Active Directory local. Los Servicios de federación de Active Directory (AD FS) son un servicio de identidad local basado en estándares. AD FS extiende la capacidad de usar la funcionalidad del inicio de sesión único (SSO) entre socios comerciales de confianza sin requerir que los usuarios inicien sesión de manera separada en cada aplicación. Esto se conoce como Federación.

Muchas organizaciones tienen aplicaciones de software como servicio (SaaS) o de línea de negocio (LOB) personalizadas federadas directamente a AD FS, junto con aplicaciones basadas en Azure AD y Office 365. 

![Aplicaciones conectadas directamente en un entorno local](media/migrate-adfs-apps-to-azure/app-integration-before-migration1.png)

**Para aumentar la seguridad de las aplicaciones, el objetivo es tener un conjunto único de controles de acceso y directivas en los entornos locales y en la nube**. 

![Aplicaciones conectadas a través de Azure AD](media/migrate-adfs-apps-to-azure/app-integration-after-migration1.png)



## <a name="types-of-apps-to-migrate"></a>Tipos de aplicaciones que se van a migrar

La migración de toda la autenticación de las aplicaciones a Azure AD es óptima, ya que ofrece un plano de control único para la administración de identidades y acceso.

Las aplicaciones pueden usar protocolos modernos o heredados para la autenticación. Considere la posibilidad de migrar primero las aplicaciones que usan protocolos de autenticación modernos (como SAML y Open ID Connect). Estas aplicaciones se pueden volver a configurar para autenticarse con Azure AD a través de un conector integrado en la Galería de aplicaciones o al registrar la aplicación en Azure AD. Las aplicaciones que usan protocolos antiguos se pueden integrar con [Application Proxy](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-application-proxy). 

Para más información, consulte ¿[Qué tipos de aplicaciones puedo integrar con Azure AD](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-application-management)?

Puede usar el [informe de actividades de aplicaciones de AD FS para migrar aplicaciones a Azure AD](https://docs.microsoft.com/azure/active-directory/manage-apps/migrate-adfs-application-activity) si tiene [Azure AD Connect Health habilitado](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-health-adfs). 

### <a name="the-migration-process"></a>El proceso de migración

Durante el proceso de mover la autenticación de las aplicaciones a Azure AD, pruebe de manera adecuada las aplicaciones y la configuración. Se recomienda seguir usando los entornos de prueba existentes para las pruebas de migración al entorno de producción. Si actualmente no hay disponible un entorno de prueba, puede configurar uno con [Azure App Service](https://azure.microsoft.com/services/app-service/) o [Azure Virtual Machines](https://azure.microsoft.com/free/virtual-machines/search/?OCID=AID2000128_SEM_lHAVAxZC&MarinID=lHAVAxZC_79233574796345_azure%20virtual%20machines_be_c__1267736956991399_kwd-79233582895903%3Aloc-190&lnkd=Bing_Azure_Brand&msclkid=df6ac75ba7b612854c4299397f6ab5b0&ef_id=XmAptQAAAJXRb3S4%3A20200306231230%3As&dclid=CjkKEQiAhojzBRDg5ZfomsvdiaABEiQABCU7XjfdCUtsl-Abe1RAtAT35kOyI5YKzpxRD6eJS2NM97zw_wcB), dependiendo de la arquitectura de la aplicación.

Puede optar por configurar un inquilino de Azure AD de prueba independiente para usarlo cuando desarrolle las configuraciones de las aplicaciones. 

El proceso de migración puede tener el aspecto siguiente:

**Fase 1: Estado actual: autenticación de las aplicaciones de producción con AD FS**

![Fase de migración 1 ](media/migrate-adfs-apps-to-azure/stage1.jpg)

 
**Fase 2: OPCIONAL: Instancia de prueba de una aplicación que apunta a un inquilino de Azure de prueba**

Actualice la configuración para que apunte la instancia de prueba de la aplicación a un inquilino de Azure AD de prueba y haga los cambios que sean necesarios. La aplicación se puede probar con los usuarios en el inquilino de Azure AD de prueba. Durante el proceso de desarrollo, puede usar herramientas como [Fiddler](https://www.telerik.com/fiddler) para comparar y comprobar solicitudes y respuestas.

Si no es factible configurar un inquilino de prueba independiente, omita esta fase, cree una instancia de prueba de una aplicación y apúntela a su inquilino de Azure AD de producción, tal como se describe en la fase 3.

![Fase de migración 2 ](media/migrate-adfs-apps-to-azure/stage2.jpg)

**Fase 3: Aplicación de prueba que apunta a un inquilino de Azure de producción**

Actualice la configuración para que apunte la instancia de prueba de la aplicación a la instancia de producción de Azure. Ahora puede probar con los usuarios de la instancia de producción. Si es necesario, revise la sección de este artículo que habla sobre la transición de usuarios.

![Fase de migración 3 ](media/migrate-adfs-apps-to-azure/stage3.jpg)

**Fase 4: Aplicación de producción que apunta al inquilino de AD de producción**

Actualice la configuración de la aplicación de producción para que apunte al inquilino de producción de Azure.

![Fase de migración 1 ](media/migrate-adfs-apps-to-azure/stage4.jpg)

 Las aplicaciones que se autentican con AD FS pueden usar grupos de Active Directory para los permisos. Use la [sincronización de Azure AD Connect](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-sync-whatis) para sincronizar los datos de identidad entre el entorno local y Azure AD antes de empezar la migración. Compruebe esos grupos y pertenencias antes de la migración para que pueda conceder acceso a los mismos usuarios cuando se migre la aplicación.

### <a name="line-of-business-lob-apps"></a>Aplicaciones de línea de negocio (LOB)

La organización desarrolla las aplicaciones de LOB de manera interna o están disponibles como un producto empaquetado estándar que se instala en el centro de datos. Entre los ejemplos se incluyen aplicaciones basadas en aplicaciones de SharePoint y Windows Identity Foundation (no SharePoint Online). 

Las aplicaciones de LOB que usan OAuth 2.0, OpenID Connect o WS-Federation se pueden integrar con Azure AD como [registros de aplicación](https://docs.microsoft.com/azure/active-directory/develop/app-registrations-training-guide-for-app-registrations-legacy-users). Integre aplicaciones personalizadas que usan SAML 2.0 o WS-Federation como [aplicaciones que no son de la galería](https://docs.microsoft.com/azure/active-directory/manage-apps/add-non-gallery-app) en la página de las aplicaciones empresariales en [Azure Portal](https://portal.azure.com/).

## <a name="saml-based-single-sign-on"></a>Inicio de sesión único basado en SAML

Las aplicaciones que usan SAML 2.0 para la autenticación se pueden configurar para el [inicio de sesión único basado en SAML](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on) (SSO basado en SAML). Con el [SSO basado en SAML](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on), puede asignar usuarios a roles de aplicación específicos según las reglas que defina en las notificaciones SAML. 

Para configurar una aplicación SaaS para el inicio de sesión único basado en SAML, consulte [Configuración del inicio de sesión único basado en SAML](https://docs.microsoft.com/azure/active-directory/manage-apps/configure-single-sign-on-non-gallery-applications). 

![Capturas de pantallas de usuario de SSO de SAML ](media/migrate-adfs-apps-to-azure/sso-saml-user-attributes-claims.png)

 
Muchas aplicaciones SaaS tiene un [tutorial específico de la aplicación](https://docs.microsoft.com/azure/active-directory/saas-apps/tutorial-list) que lo guía por la configuración del inicio de sesión único basado en SAML.

![tutorial de la aplicación](media/migrate-adfs-apps-to-azure/app-tutorial.png)

Algunas aplicaciones se pueden migrar fácilmente. Las aplicaciones con requisitos más complejos, como las notificaciones personalizadas, pueden requerir configuración adicional en Azure AD o Azure AD Connect. Para información sobre las asignaciones de notificaciones compatibles, consulte [Asignación de notificaciones en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-claims-mapping).

Cuando asigne los atributos, considere estas limitaciones:

* No todos los atributos que se pueden emitir en AD FS se mostrarán en Azure AD como atributos que se van a emitir a los tokens de SAML, incluso si esos atributos están sincronizados. Cuando edite el atributo, la lista desplegable Valor mostrará los distintos atributos que están disponibles en Azure AD. Compruebe la configuración de la [sincronización de Azure AD Connect](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-sync-whatis) para asegurarse de que los atributos necesarios, por ejemplo, samAccountName, se sincronizan con Azure AD. Puede usar los atributos de extensión para emitir cualquier notificación que no forme parte del esquema de usuario estándar en Azure AD.

* En los escenarios más comunes, solo la notificación NameID y otras notificaciones de identificador de usuario comunes son necesarias para una aplicación. Para determinar si se requieren notificaciones adicionales, examine qué notificaciones está emitiendo desde AD FS.

* No todas las notificaciones pueden ser emisiones, porque algunas notificaciones está protegidas en Azure AD. 

* La capacidad de usar tokens SAML cifrados está ahora en versión preliminar. Consulte el procedimiento de [personalización de las notificaciones emitidas en el token SAML para aplicaciones empresariales](https://docs.microsoft.com/azure/active-directory/develop/active-directory-saml-claims-customization).

 

### <a name="software-as-a-service-saas-apps"></a>Aplicaciones de software como servicio (SaaS)

Si el inicio de sesión del usuario en aplicaciones SaaS como Salesforce, ServiceNow o Workday que están integradas con AD FS, entonces se usa el inicio de sesión federado para las aplicaciones SaaS. 

Es posible que la mayoría de las aplicaciones SaaS ya esté configurada en Azure AD. Microsoft tiene muchas conexiones preconfiguradas a aplicaciones SaaS de la [Galería de aplicaciones de Azure AD](https://azuremarketplace.microsoft.com/marketplace/apps/category/azure-active-directory-apps), lo que simplificará la transición. Las aplicaciones de SAML 2.0 se pueden integrar con Azure AD a través de la Galería de aplicaciones de Azure AD o como [aplicaciones que no son de la galería](https://docs.microsoft.com/azure/active-directory/manage-apps/add-non-gallery-app). 

Las aplicaciones que usan OAuth 2.0 u OpenID Connect se pueden integrar con Azure AD de forma parecida a los [registros de aplicación](https://docs.microsoft.com/azure/active-directory/develop/app-registrations-training-guide-for-app-registrations-legacy-users). Las aplicaciones que usan protocolos heredados puede usar [Azure AD Application Proxy](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy) para autenticarse con Azure AD.

Si tiene algún problema para incorporar las aplicaciones SaaS, puede ponerse en contacto con el [alias de soporte técnico de la integración de aplicaciones SaaS](mailto:SaaSApplicationIntegrations@service.microsoft.com).

**Certificado de firma de SAML para SSO**: Los certificados de firma son una parte importante de cualquier implementación de SSO. Azure AD crea los certificados de firma para establecer el SSO federado basado en SAML para las aplicaciones SaaS. Una vez que agregue aplicaciones de la galería o que no son de la galería, configurará la aplicación agregada con la opción de SSO federado. Consulte [Administrar certificados para inicio de sesión único federado en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/manage-certificates-for-federated-single-sign-on).

### <a name="apps-and-configurations-that-can-be-moved-today"></a>Aplicaciones y configuraciones que se pueden migrar actualmente

Las aplicaciones que se pueden mover fácilmente en la actualidad incluyen las aplicaciones SAML 2.0 que usan el conjunto estándar de notificaciones y elementos de configuración:

* Nombre principal del usuario

* Dirección de correo electrónico

* Nombre propio

* Surname

* Atributo alternativo de **NameID** de SAML, lo que incluye el atributo de correo de Azure AD, el prefijo de correo, el identificador de empleado, los atributos de extensión 1 a 15 o el atributo **SamAccountName** local. Para más información, consulte [Edición de la notificación NameIdentifier](https://docs.microsoft.com/azure/active-directory/develop/active-directory-saml-claims-customization).

* Notificaciones personalizadas.

Lo siguiente requiere pasos de configuración adicionales para migrar a Azure AD:

* Autorización personalizada o reglas de Multi-Factor Authentication (MFA) en AD FS. Se configuran mediante el uso de la característica [Acceso condicional de Azure AD](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-azure-portal).

* Aplicaciones con varios puntos de conexión de URL de respuesta. Se configuran en Azure AD mediante PowerShell o en la interfaz de Azure Portal.

* Aplicaciones de WS-Federation, como las aplicaciones de SharePoint, que requieren tokens de la versión 1.1 de SAML. Puede configurarlas manualmente con PowerShell. También puede agregar una plantilla genérica integrada previamente para aplicaciones de SharePoint y SAML 1.1 de la galería. Se admite el protocolo SAML 2.0.

* Reglas de transformaciones de emisión de notificaciones complejas. Para información sobre las asignaciones de notificaciones compatibles, consulte:
   *  [Asignación de notificaciones en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-claims-mapping) 
   * [Personalización de las notificaciones emitidas en el token SAML para aplicaciones empresariales en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-saml-claims-customization)

 

### <a name="apps-and-configurations-not-supported-in-azure-ad-today"></a>Aplicaciones y configuraciones no admitidas actualmente en Azure AD

Las aplicaciones que necesitan las funcionalidades siguientes no se pueden migrar en estos momentos.

**Funcionalidades de protocolo**

* Compatibilidad con el patrón ActAs de WS-Trust

* Resolución de artefactos SAML

* Comprobación de la firma de las solicitudes SAML firmadas  
‎Tenga en cuenta que se aceptan solicitudes firmadas, pero no se comprueba la firma.  
‎Dado que Azure AD solo devolverá el token a los puntos de conexión preconfigurados en la aplicación, es probable que la comprobación de la firma no sea necesaria en la mayoría de los casos.

**Funcionalidades de notificaciones en token**

* Notificaciones de almacenes de atributos que no son el directorio de Azure AD, a menos que los datos estén sincronizados con Azure AD. Para más información, consulte [Introducción a la API de sincronización de Azure AD](https://docs.microsoft.com/graph/api/resources/synchronization-overview?view=graph-rest-beta).

* Emisión de atributos de varios valores de directorio. Por ejemplo, en este momento no se puede emitir una notificación de varios valores para las direcciones de proxy.

## <a name="map-app-settings-from-ad-fs-to-azure-ad"></a>Asignación de la configuración de una aplicación de AD FS a Azure AD

La migración comienza con la evaluación de cómo está configurada la aplicación en el entorno local y la asignación de esa configuración a Azure AD. AD FS y Azure AD funcionan de manera similar, de modo que los conceptos de configuración de direcciones URL e identificadores de confianza de inicio y cierre de sesión se aplican en ambos casos. Documente las opciones de configuración de AD FS de las aplicaciones para que pueda configurarlas fácilmente en Azure AD.

### <a name="map-app-configuration-settings"></a>Asignación de las opciones de configuración de aplicaciones

En la tabla siguiente se describen algunas de las asignaciones más comunes de configuración entre una relación de confianza para usuario autenticado de AD FS y una aplicación empresarial de Azure AD:

* AD FS: busque la configuración en la relación de confianza para usuario autenticado de AD FS correspondiente a la aplicación. Haga clic con el botón derecho en el usuario de confianza y seleccione Propiedades. 

* Azure AD: la configuración se establece dentro de [Azure Portal](https://portal.azure.com/) en las propiedades de inicio de sesión único de cada aplicación.

| Opción de configuración| AD FS| Configuración en Azure AD| Token SAML |
| - | - | - | - |
| **Dirección URL de inicio de sesión de aplicación** <p>La dirección URL para que el usuario inicie sesión en la aplicación en un flujo de SAML iniciado por el proveedor de servicios (SP).| N/D| Abra la configuración básica de SAML desde el inicio de sesión basado en SAML| N/D |
| **Dirección URL de respuesta de la aplicación** <p>La dirección URL de la aplicación desde la perspectiva del proveedor de identidades (IdP). El IdP envía aquí el usuario y el token después de que el usuario inicia sesión en el IdP.  ‎Esto también se conoce como el **punto de conexión del consumidor de aserción de SAML**.| Seleccione la pestaña **Puntos de conexión**| Abra la configuración básica de SAML desde el inicio de sesión basado en SAML| Elemento Destino en el token SAML. Valor de ejemplo: `https://contoso.my.salesforce.com` |
| **Dirección URL de cierre de sesión de la aplicación** <p>Esta es la dirección URL a la que se envían las solicitudes de "limpieza de cierre de sesión" cuando un usuario cierra la sesión de una aplicación. El IdP envía la solicitud para también cerrar la sesión del usuario de todas las demás aplicaciones.| Seleccione la pestaña **Puntos de conexión**| Abra la configuración básica de SAML desde el inicio de sesión basado en SAML| N/D |
| **Identificador de la aplicación** <p>Este es el identificador de la aplicación desde la perspectiva del IdP. El valor de la dirección URL de inicio de sesión se usa a menudo (pero no siempre) como identificador.  ‎En ocasiones, la aplicación lo llama el "identificador de entidad".| Seleccione la pestaña **Identificadores**|Abra la configuración básica de SAML desde el inicio de sesión basado en SAML| Se asigna al elemento **Audiencia** del token SAML. |
| **Metadatos de federación de la aplicación** <p>Se trata de la ubicación de los metadatos de federación de la aplicación. El IdP la usa para actualizar automáticamente valores de configuración específicos, como puntos de conexión o certificados de cifrado.| Seleccione la pestaña **Supervisión**| N/D Azure AD no admite el consumo directo de metadatos de federación de aplicaciones. Puede importar manualmente los metadatos de federación.| N/D |
| **Identificador de usuario / identificador de nombre** <p>Atributo que se usa para indicar de forma única la identidad de usuario de Azure AD o AD FS para la aplicación.  Por lo general, este atributo es el UPN o la dirección de correo electrónico del usuario.| Reglas de notificación. En la mayoría de los casos, la regla de notificación emite una notificación con un tipo que termina con NameIdentifier.| Puede encontrar el identificador bajo el encabezado **Atributos y notificaciones de usuario**. De manera predeterminada, se usa el UPN.| Se asigna al elemento **NameID** en el token SAML. |
| **Otras notificaciones** <p>Entre los ejemplos de otra información de notificación que se envía habitualmente desde el IdP a la aplicación se incluyen el nombre, el apellido, la dirección de correo electrónico y la pertenencia a grupos.| En AD FS, se encuentra como otras reglas de notificación en el usuario de confianza.| Puede encontrar el identificador bajo el encabezado **Atributos y notificaciones de usuario**. Seleccione **Ver** para editar todos los demás atributos de usuario.| N/D |


### <a name="map-identity-provider-idp-settings"></a>Asignación de la configuración del proveedor de identidades (IdP)

Configure las aplicaciones para que apunten a Azure AD en vez de AD FS para SSO. Aquí nos centraremos en las aplicaciones SaaS que usan el protocolo SAML. Sin embargo, este concepto se extiende también a las aplicaciones de LOB personalizadas.

> [!NOTE]
> Los valores de configuración de Azure AD siguen el patrón en el que el identificador del inquilino de Azure reemplaza {tenant-ID} y el identificador de la aplicación reemplaza {Application-ID}. Esta información la encuentra en [Azure Portal](https://portal.azure.com/) en Azure Active Directory > Propiedades: 

* Seleccione el id. de directorio para ver el id. del inquilino. 

* Seleccione el id. de aplicación para ver el id. de la aplicación.

 En un nivel alto, asigne los elementos de configuración de las aplicaciones SaaS clave siguientes a Azure AD. 

 

| Elemento| Valor de configuración |
| - | - |
| Emisor del proveedor de identidades| https:\//sts.windows.net/{tenant-id}/ |
| URL de inicio de sesión del proveedor de identidades| [https://login.microsoftonline.com/{tenant-id}/saml2](https://login.microsoftonline.com/{tenant-id}/saml2) |
| URL de cierre de sesión del proveedor de identidades| [https://login.microsoftonline.com/{tenant-id}/saml2](https://login.microsoftonline.com/{tenant-id}/saml2) |
| Ubicación de los metadatos de federación| [https://login.windows.net/{tenant-id}/federationmetadata/2007-06/federationmetadata.xml?appid={application-id}](https://login.windows.net/{tenant-id}/federationmetadata/2007-06/federationmetadata.xml?appid={application-id}) |


### <a name="map-sso-settings-for-saas-apps"></a>Asignación de la configuración de SSO para aplicaciones SaaS

Las aplicaciones SaaS deben saber dónde enviar las solicitudes de autenticación y cómo validar los tokens recibidos. En la tabla siguiente se describen los elementos para configurar los valores de inicio de sesión único en la aplicación y sus valores o ubicaciones dentro de AD FS y Azure AD.

| Opción de configuración| AD FS| Configuración en Azure AD |
| - | - | - |
| **Dirección URL de inicio de sesión del IdP** <p>Dirección URL de inicio de sesión del IdP desde la perspectiva de la aplicación (donde se redirige al usuario para el inicio de sesión).| La dirección URL de inicio de sesión de AD FS es el nombre del servicio de federación de AD FS seguido de "/adfs/ls/". <p>Por ejemplo: `https://fs.contoso.com/adfs/ls/`| Reemplace {tenant id} por el id. del inquilino. <p> ‎Para las aplicaciones que usan el protocolo SAML-P: [https://login.microsoftonline.com/{tenant-id}/saml2](https://login.microsoftonline.com/{tenant-id}/saml2) <p>‎Para las aplicaciones que usan el protocolo WS-Federation: [https://login.microsoftonline.com/{tenant-id}/wsfed](https://login.microsoftonline.com/{tenant-id}/wsfed) |
| **Dirección URL de cierre de sesión del IdP**<p>Dirección URL de cierre de sesión del IdP desde la perspectiva de la aplicación (donde se redirige al usuario cuando elige cerrar sesión en la aplicación).| La dirección URL de cierre de sesión es la misma que la dirección URL de inicio de sesión o la misma dirección URL seguida de "wa=wsignout1.0". Por ejemplo: `https://fs.contoso.com/adfs/ls/?wa=wsignout1.0`| Reemplace {tenant id} por el id. del inquilino.<p>Para las aplicaciones que usan el protocolo SAML-P:<p>[https://login.microsoftonline.com/{tenant-id}/saml2](https://login.microsoftonline.com/{tenant-id}/saml2) <p> ‎Para las aplicaciones que usan el protocolo WS-Federation: [https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.0](https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.0) |
| **Certificado de firma de tokens**<p>El IdP usa la clave privada del certificado para firmar los tokens emitidos. Se comprueba que el token procede del mismo IdP que se ha configurado como de confianza en la aplicación.| El certificado de firma de token de AD FS se encuentra en Administración de AD FS, en **Certificados**.| Lo encontrará en Azure Portal en las **Propiedades del inicio de sesión único** de la aplicación, bajo el encabezado **Certificado de firma de SAML**. Desde allí puede descargar el certificado para cargarlo en la aplicación.  <p>Si la aplicación tiene más de un certificado, todos los certificados se pueden encontrar en el archivo XML de metadatos de federación. |
| **Identificador/ "emisor"**<p>Identificador del IdP desde la perspectiva de la aplicación (también llamado "Identificador de emisor").<p>‎En el token SAML, el valor aparece como el elemento Emisor.| Por lo general, el identificador de AD FS es el identificador del servicio de federación en Administración de AD FS, bajo **Servicio > Modificar las propiedades del Servicio de federación**. Por ejemplo: `http://fs.contoso.com/adfs/services/trust`| Reemplace {tenant id} por el id. del inquilino.<p>https:\//sts.windows.net/{tenant-id}/ |
| **Metadatos de federación del IdP**<p>Ubicación de los metadatos de federación disponibles públicamente del IdP. (Algunas aplicaciones usan metadatos de federación como alternativa a la configuración de direcciones URL, identificadores y certificados de firma de token de forma individual por el administrador).| Puede encontrar la dirección URL de metadatos de federación de AD FS en Administración de AD FS en **Servicio > Puntos de conexión > Metadatos > Tipo: Metadatos de federación**. Por ejemplo: `https://fs.contoso.com/FederationMetadata/2007-06/FederationMetadata.xml`| El valor correspondiente para Azure AD sigue el patrón [https://login.microsoftonline.com/{TenantDomainName}/FederationMetadata/2007-06/FederationMetadata.xml](https://login.microsoftonline.com/{TenantDomainName}/FederationMetadata/2007-06/FederationMetadata.xml). Reemplace {TenantDomainName} por el nombre del inquilino con el formato "contoso.onmicrosoft.com."   <p>Para más información, consulte [Metadatos de federación](https://docs.microsoft.com/azure/active-directory/azuread-dev/azure-ad-federation-metadata). |


## <a name="represent-ad-fs-security-policies-in-azure-ad"></a>Representación de las directivas de seguridad de AD FS en Azure AD

Cuando mueva la autenticación de las aplicaciones a Azure AD, cree asignaciones desde las directivas de seguridad existentes a sus variantes equivalentes o alternativas disponibles en Azure AD. Asegurarse de que estas asignaciones se pueden realizar mientras se cumplan los estándares de seguridad necesarios por los propietarios de la aplicación hará que el resto de la migración de la aplicación sea mucho más fácil.

Para cada tipo de regla y sus ejemplos, aquí se sugiere cómo se debe ver la regla en AD FS, el código equivalente en el lenguaje de la regla AD FS y cómo se asigna en Azure AD.

### <a name="map-authorization-rules"></a>Asignación de reglas de autorización

A continuación se muestran ejemplos de tipos de reglas de autorización en AD FS y cómo se pueden asignar a Azure AD:

#### <a name="example-1-permit-access-to-all-users"></a>Ejemplo 1: Permitir el acceso a todos los usuarios

Permitir el acceso a todos los usuarios tiene este aspecto en AD FS: 

![Fase de migración 1 ](media/migrate-adfs-apps-to-azure/sso-saml-user-attributes-claims.png)


Esto se asigna a Azure AD de una de las maneras siguientes:

En el [Portal de Azure](https://portal.azure.com/):
* Opción 1: Establezca la opción Asignación de usuarios necesaria en No ![edite la directiva de control de acceso para aplicaciones SaaS ](media/migrate-adfs-apps-to-azure/permit-access-to-all-users-2.png)

    Tenga en cuenta que cambiar la opción Asignación de usuarios necesaria a Sí requiere que los usuarios estén asignados a la aplicación para que obtengan acceso. Cuando se establece en No, todos los usuarios tienen acceso. Este modificador no controla lo que los usuarios ven en la experiencia Aplicaciones. 

 
* Opción 2: En la pestaña Usuarios y grupos, asigne la aplicación al grupo automático "Todos los usuarios". <p>
Debe [habilitar los grupos dinámicos](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-create-rule) en el inquilino de Azure AD para que el grupo predeterminado "Todos los usuarios" esté disponible.

   ![Mis aplicaciones SaaS en Azure AD ](media/migrate-adfs-apps-to-azure/permit-access-to-all-users-3.png)


#### <a name="example-2-allow-a-group-explicitly"></a>Ejemplo 2: Permitir un grupo de manera explícita

Autorización explícita de un grupo en AD FS:


![Reglas de autorización de emisión ](media/migrate-adfs-apps-to-azure/allow-a-group-explicitly-1.png)


La regla se asigna a Azure AD de la siguiente manera:

En [Azure Portal](https://portal.azure.com/), primero [creará un grupo de usuarios](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-groups-create-azure-portal) que se corresponderá con el grupo de usuarios de AD FS y, luego, asignará permisos de aplicación a ese grupo:

![Agregar asignación ](media/migrate-adfs-apps-to-azure/allow-a-group-explicitly-2.png)


#### <a name="example-3-authorize-a-specific-user"></a>Ejemplo 3: Autorizar a un usuario específico

Autorización explícita de un usuario en AD FS:

![Reglas de autorización de emisión ](media/migrate-adfs-apps-to-azure/authorize-a-specific-user-1.png)

La regla se asigna a Azure AD de la siguiente manera:

En [Azure Portal](https://portal.azure.com/), agregue un usuario a la aplicación a través de la pestaña Agregar asignación de la aplicación, tal como se muestra a continuación:

![Mis aplicaciones SaaS en Azure ](media/migrate-adfs-apps-to-azure/authorize-a-specific-user-2.png)

 
### <a name="map-multi-factor-authentication-rules"></a>Asignación de reglas de Multi-Factor Authentication 

Una implementación local de [Multi-Factor Authentication (MFA)](https://docs.microsoft.com/azure/active-directory/authentication/multi-factor-authentication) y AD FS seguirá funcionando después de la migración, ya que está federado con AD FS. Sin embargo, considere la posibilidad de migrar a las funcionalidades de MFA integradas de Azure que están asociadas a los flujos de trabajo de acceso condicional de Azure AD. 

A continuación se muestran ejemplos de tipos de reglas de MFA en AD FS y cómo se pueden asignar a Azure AD en función de condiciones distintas:

Configuración de reglas de MFA en AD FS:

![Configuración de Azure AD MFA](media/migrate-adfs-apps-to-azure/mfa-location-1.png)


#### <a name="example-1-enforce-mfa-based-on-usersgroups"></a>Ejemplo 1: Aplicación de MFA en función de usuarios o grupos

El selector de usuarios o grupos es una regla que permite aplicar MFA en cada grupo (SID de grupo) o en cada usuario (SID principal). Además de las asignaciones de usuarios o grupos, todas las casillas adicionales de la interfaz de usuario de la configuración de AD FS MFA funcionan como reglas adicionales que se evalúan después de aplicar la regla de usuarios o grupos. 


Especifique las reglas de MFA para un usuario o un grupo en Azure AD:

1. Cree una [directiva de acceso condicional nueva](https://docs.microsoft.com/azure/active-directory/authentication/tutorial-enable-azure-mfa?toc=/azure/active-directory/conditional-access/toc.json&bc=/azure/active-directory/conditional-access/breadcrumb/toc.json).

2. Seleccione **Asignaciones**. Agregue los usuarios o los grupos a los que quiere aplicar MFA.

3. Configure las opciones de **controles de acceso**, tal como se muestra a continuación:  
‎

![Configuración de AAD MFA](media/migrate-adfs-apps-to-azure/mfa-usersorgroups.png)

 
 #### <a name="example-2-enforce-mfa-for-unregistered-devices"></a>Ejemplo 2: Aplicación de MFA para dispositivos no registrados

Especifique las reglas de MFA para los dispositivos no registrados en Azure AD:

1. Cree una [directiva de acceso condicional nueva](https://docs.microsoft.com/azure/active-directory/authentication/tutorial-enable-azure-mfa?toc=/azure/active-directory/conditional-access/toc.json&bc=/azure/active-directory/conditional-access/breadcrumb/toc.json).

2. Establezca las **Asignaciones** en **Todos los usuarios**.

3. Configure las opciones de **controles de acceso**, tal como se muestra a continuación:  
‎

![Configuración de AAD MFA](media/migrate-adfs-apps-to-azure/mfa-unregistered-devices.png)

 
Cuando se establece la opción Para varios controles en Requerir uno de los controles seleccionados, significa que si el usuario cumple con cualquiera de las condiciones especificadas por la casilla, se le concederá acceso a la aplicación.

#### <a name="example-3-enforce-mfa-based-on-location"></a>Ejemplo 3: Aplicación de MFA en función de la ubicación

Especifique las reglas de MFA en función de la ubicación de un usuario en Azure AD:

1. Cree una [directiva de acceso condicional nueva](https://docs.microsoft.com/azure/active-directory/authentication/tutorial-enable-azure-mfa?toc=/azure/active-directory/conditional-access/toc.json&bc=/azure/active-directory/conditional-access/breadcrumb/toc.json).

1. Establezca las **Asignaciones** en **Todos los usuarios**.

1. [Configure ubicaciones con nombre en Azure AD](https://docs.microsoft.com/azure/active-directory/active-directory-named-locations), de lo contrario, se confía en la federación desde dentro de la red corporativa. 

1. Configure las **Reglas de condiciones** para especificar las ubicaciones para las que quiere aplicar MFA.

![Configuración de Azure AD MFA](media/migrate-adfs-apps-to-azure/mfa-location-1.png)

5. Configure las opciones de **controles de acceso**, tal como se muestra a continuación:


![Asignación de directivas de control de acceso](media/migrate-adfs-apps-to-azure/mfa-location-2.png)

 
### <a name="map-emit-attributes-as-claims-rule"></a>Asignación de atributos de emisión como regla de notificaciones

A continuación se muestra un ejemplo de cómo se asignan los atributos en AD FS:

![Configuración de Azure AD MFA](media/migrate-adfs-apps-to-azure/map-emit-attributes-as-claimsrule-1.png)


La regla se asigna a Azure AD de la siguiente manera:

En [Azure Portal](https://portal.azure.com/), seleccione **Aplicaciones empresariales**, **Inicio de sesión único** y agregue **Atributos de token SAML**, tal como se muestra a continuación:

![Configuración de Azure AD MFA](media/migrate-adfs-apps-to-azure/map-emit-attributes-as-claimsrule-2.png)



### <a name="map-built-in-access-control-policies"></a>Asignación de directivas de control de acceso integradas

AD FS 2016 tiene varias directivas de control de acceso integradas que puede elegir:

![Control de acceso integrado de Azure AD](media/migrate-adfs-apps-to-azure/map-builtin-access-control-policies-1.png)

 
Para implementar directivas integradas en Azure AD, puede usar una [directiva de acceso condicional nueva](https://docs.microsoft.com/azure/active-directory/authentication/tutorial-enable-azure-mfa?toc=/azure/active-directory/conditional-access/toc.json&bc=/azure/active-directory/conditional-access/breadcrumb/toc.json) y configurar los controles de acceso, o bien puede usar el diseñador de directivas personalizadas en AD FS 2016 para configurar las directivas de control de acceso. El editor de reglas tiene una lista exhaustiva de las opciones Permitir y Excepto que puede ayudarlo a realizar todo tipo de permutaciones.

![Directivas de control de acceso de Azure AD](media/migrate-adfs-apps-to-azure/map-builtin-access-control-policies-2.png)



En esta tabla, se enumeran algunas opciones Permitir y Excepto y cómo se asignan a Azure AD. 


| Opción | ¿Cómo se configura la opción Permitir en Azure AD?| ¿Cómo se configura la opción Excepto en Azure AD? |
| - | - | - |
| Desde una red específica| Se asigna a [Ubicación con nombre](https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations) en Azure AD| Use la opción **Excluir** para las [ubicaciones de confianza](https://docs.microsoft.com/azure/active-directory/conditional-access/location-condition) |
| Desde grupos específicos| [Establezca una asignación de usuario o grupos](https://docs.microsoft.com/azure/active-directory/manage-apps/assign-user-or-group-access-portal)| Use la opción **Excluir** en Usuarios y Grupos |
| Desde dispositivos con un nivel de confianza específico| Establezca esta opción desde el control "Estado del dispositivo" en Asignaciones -> Condiciones| Use la opción **Excluir** en la condición de estado del dispositivo e incluya **Todos los dispositivos** |
| Con notificaciones específicas en la solicitud| No se puede migrar esta configuración| No se puede migrar esta configuración |


Un ejemplo de cómo configurar la opción Excluir para las ubicaciones de confianza en Azure Portal:

![Captura de pantalla de la asignación de directivas de control de acceso](media/migrate-adfs-apps-to-azure/map-builtin-access-control-policies-3.png)



## <a name="transition-users-from-ad-fs-to-azure-ad"></a>Transición de usuarios de AD FS a Azure AD

### <a name="sync-ad-fs-groups-in-azure-ad"></a>Sincronización de grupos de AD FS en Azure AD

Cuando se asignan reglas de autorización, las aplicaciones que se autentican con AD FS pueden usar los grupos de Active Directory para los permisos. En tal caso, use [Azure AD Connect](https://go.microsoft.com/fwlink/?LinkId=615771) para sincronizar estos grupos con Azure AD antes de migrar las aplicaciones. Asegúrese de comprobar esos grupos y pertenencias antes de la migración para que pueda conceder acceso a los mismos usuarios cuando se migre la aplicación.

Para más información, consulte [Requisitos previos para usar atributos de grupo sincronizados desde Active Directory](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-fed-group-claims).

### <a name="setup-user-self-provisioning"></a>Configuración del aprovisionamiento automático de usuarios 

Algunas aplicaciones SaaS admiten la capacidad de aprovisionar automáticamente a los usuarios la primera vez que inician sesión en la aplicación. En Azure Active Directory (Azure AD), el término aprovisionamiento de aplicaciones hace referencia a la creación automática de identidades y roles de usuario en las aplicaciones de nube ([SaaS](https://azure.microsoft.com/overview/what-is-saas/)) a las que los usuarios necesitan acceso. Los usuarios que se migran ya tendrán una cuenta en la aplicación SaaS. Los usuarios nuevos que se agreguen después de la migración deberán aprovisionarse. Pruebe el [aprovisionamiento de aplicaciones SaaS](https://docs.microsoft.com/azure/active-directory/app-provisioning/user-provisioning) una vez que se migre la aplicación.

### <a name="sync-external-users-in-azure-ad"></a>Sincronización de los usuarios externos en Azure AD

Los usuarios externos existentes se pueden configurar de dos maneras principales dentro de AD FS:

#### <a name="external-users-with-a-local-account-within-your-organization"></a>Usuarios externos con una cuenta local dentro de la organización

Todavía podrá seguir usando estas cuentas de la misma manera que las cuentas de usuario internas. Estas cuentas de usuario externas tienen un nombre principal dentro de la organización, aunque el correo electrónico de la cuenta puede apuntar externamente. A medida que la migración progresa, puede aprovechar las ventajas que [Azure AD B2B](https://docs.microsoft.com/azure/active-directory/b2b/what-is-b2b) ofrece al migrar estos usuarios para que usen su propia identidad corporativa cuando dicha identidad está disponible. Esto simplifica el proceso de inicio de sesión para esos usuarios, ya que a menudo inician sesión con su inicio de sesión corporativo propio. También se facilitará la administración de la organización, porque ya no tendrá que administrar cuentas para usuarios externos.

#### <a name="federated-external-identities"></a>Identidades externas federadas

Si actualmente se está federando con una organización externa, hay algunos enfoques que puede asumir:

* [Incorpore usuarios de colaboración B2B de Azure Active Directory en Azure Portal](https://docs.microsoft.com/azure/active-directory/b2b/add-users-administrator). Puede enviar de forma proactiva invitaciones de colaboración B2B desde el portal administrativo de Azure AD a la organización asociada para que los miembros individuales sigan usando las aplicaciones y los recursos que acostumbran usar. 

* [Cree un flujo de trabajo de registro de B2B de autoservicio](https://docs.microsoft.com/azure/active-directory/b2b/self-service-portal) que genere una solicitud para usuarios individuales en la organización asociada mediante la API de invitación de B2B.

Independientemente de la configuración de los usuarios externos existentes, es probable que tengan permisos asociados con su cuenta, ya sea en la pertenencia a grupos o en permisos específicos. Evalúe si estos permisos se deben migrar o limpiar. Las cuentas dentro de su organización que representan a un usuario externo se deben deshabilitar una vez que el usuario ha migrado a una identidad externa. El proceso de migración se debe describir con sus socios comerciales, ya que puede ocurrir una interrupción en su capacidad de conectarse con los recursos.

## <a name="migrate-and-test-your-apps"></a>Migración y prueba de las aplicaciones

Siga el proceso de migración que se detalla en este artículo.

Luego, vaya a [Azure Portal](https://aad.portal.azure.com/) para probar si la migración se realizó correctamente. Siga las instrucciones que se describen a continuación:
1. Seleccione **Aplicaciones empresariales** > **Todas las aplicaciones** y busque la aplicación en la lista.

1. Seleccione **Administrar** > **Usuarios y grupos** para asignar al menos un usuario o grupo a la aplicación.

1. Seleccione **Administrar** > **Acceso condicional**. Revise la lista de directivas y asegúrese de que no está bloqueando el acceso a la aplicación con una [directiva de acceso condicional](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-azure-portal).

En función de cómo configure la aplicación, compruebe que el inicio de sesión único funciona correctamente. 

| Tipo de autenticación| Prueba |
| - | - |
| OAuth / OpenID Connect| Seleccione **Aplicaciones empresariales > Permisos** y asegúrese de haber dado su consentimiento a que la aplicación se use en la organización en la configuración del usuario correspondiente a la aplicación.  
‎ |
| SSO basado en SAML| Use el botón [Probar la configuración de SAML](https://docs.microsoft.com/azure/active-directory/develop/howto-v1-debug-saml-sso-issues) que está en **Inicio de sesión único**.  
‎ |
| SSO basado en contraseñas| Descargue e instale la [Extensión de inicio de](https://docs.microsoft.com/azure/active-directory/user-help/active-directory-saas-access-panel-introduction)[-](https://docs.microsoft.com/azure/active-directory/user-help/active-directory-saas-access-panel-introduction)[sesión seguro de Aplicaciones](https://docs.microsoft.com/azure/active-directory/user-help/active-directory-saas-access-panel-introduction). Esta extensión lo ayudará a iniciar cualquiera de las aplicaciones en la nube de su organización que requiera que se use un proceso de inicio de sesión único.  
‎ |
| Proxy de aplicación| Asegúrese de que el conector esté en ejecución y que esté asignado a la aplicación. Visite la [Solución de problemas y mensajes de error de Proxy de aplicación](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-troubleshoot) para obtener más ayuda.  
‎ |

> [!NOTE]
> Las cookies del entorno de AD FS anterior seguirán siendo persistentes en las máquinas del usuario. Estas cookies pueden causar problemas con la migración, ya que los usuarios podrían dirigirse al entorno de inicio de sesión de AD FS anterior frente al inicio de sesión nuevo de Azure AD. Es posible que tenga que borrar las cookies del explorador del usuario de manera manual o mediante un script. También puede usar System Center Configuration Manager o una plataforma similar.

### <a name="troubleshoot"></a>Solución de problemas

Si hay algún error en la prueba de las aplicaciones migradas, la solución de problemas podría ser el primer paso antes de volver a los usuarios de confianza de AD FS existentes. Consulte [Depuración del inicio de sesión único basado en SAML en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/azuread-dev/howto-v1-debug-saml-sso-issues).

### <a name="rollback-migration"></a>Reversión de la migración

Si se produce un error en la migración, se recomienda dejar los usuarios de confianza existentes en los servidores de AD FS y quitar el acceso a los usuarios de confianza. Esto permitirá una reserva rápida si se necesita durante la implementación.

### <a name="end-user-communication"></a>Comunicación con el usuario final

Aunque la ventana de interrupción planeada misma puede ser mínima, de todos modos debe planear la comunicación con los empleados durante estos períodos de tiempo de forma proactiva, a la vez que se realiza la migración total de AD FS a Azure AD. Asegúrese de que la experiencia de la aplicación tenga un botón Comentarios o punteros al departamento de soporte técnico en caso de problemas.

Una vez completada la implementación, puede enviar la comunicación que informa a los usuarios de la implementación correcta y recordarles los pasos nuevos que deben seguir.

* Indique a los usuarios que usen la página [Aplicaciones](https://myapps.microsoft.com) para acceder a todas las aplicaciones migradas. 

* Recuerde a los usuarios que es posible que necesiten actualizar la configuración de MFA. 

* Si se implementa el autoservicio de restablecimiento de contraseña, es posible que los usuarios necesiten actualizar o comprobar sus métodos de autenticación. Consulte las plantillas de comunicación con el usuario final de [MFA](https://aka.ms/mfatemplates) y [SSPR](https://aka.ms/ssprtemplates).

Comunicación con usuarios externos: por lo general, este grupo de usuario es el que se ve afectado de forma más crítica en caso de problemas. Esto es especialmente cierto si su postura de seguridad dicta un conjunto diferente de reglas de acceso condicional o perfiles de riesgo para los asociados externos. Asegúrese de que los asociados externos conocen la programación de migración a la nube y que tienen un período de tiempo durante el cual se les anima a participar en una implementación piloto que prueba todos los flujos exclusivos de la colaboración externa. Por último, asegúrese de que tienen una manera de acceder al departamento de soporte técnico en caso de problemas importantes.

## <a name="next-steps"></a>Pasos siguientes
Lea [Migrating application authentication to Azure AD](https://aka.ms/migrateapps/whitepaper) (Migración de la autenticación de aplicaciones a Azure AD).<p>
Configure el [acceso condicional](https://docs.microsoft.com/azure/active-directory/conditional-access/overview) y [MFA](https://docs.microsoft.com/azure/active-directory/authentication/concept-mfa-howitworks).
