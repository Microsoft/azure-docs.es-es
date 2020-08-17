---
title: Novedades Notas de la versión de Azure Active Directory | Microsoft Docs
description: Obtenga información acerca de las novedades de Azure Active Directory, como, por ejemplo, las notas de la versión más recientes, los problemas conocidos, las correcciones de errores, las funcionalidades en desuso y los próximos cambios.
services: active-directory
author: ajburnle
manager: daveba
featureFlags:
- clicktale
ms.assetid: 06a149f7-4aa1-4fb9-a8ec-ac2633b031fb
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 06/30/2020
ms.author: ajburnle
ms.reviewer: dhanyahk
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6694043f083de30f3bca10051fc7012ef1471c37
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87798824"
---
# <a name="whats-new-in-azure-active-directory"></a>¿Cuáles son las novedades de Azure Active Directory?

>Reciba notificaciones para volver a visitar esta página y obtener actualizaciones; para ello, copie y pegue la dirección URL `https://docs.microsoft.com/api/search/rss?search=%22Release+notes+-+Azure+Active+Directory%22&locale=en-us` en el ![icono del lector de fuentes icono RSS](./media/whats-new/feed-icon-16x16.png) lector de fuentes.

En Azure AD se realizan mejoras de forma continua. Para mantenerse al día de los avances más recientes, este artículo proporciona información acerca de los elementos siguientes:

- Versiones más recientes
- Problemas conocidos
- Corrección de errores
- Funciones obsoletas
- Planes de cambios

Esta página se actualiza mensualmente, por lo que se recomienda visitarla con frecuencia. Si busca elementos que tengan más de 6 meses, puede encontrarlos en el [Archivo de novedades de Azure Active Directory](whats-new-archive.md).

---

## <a name="july-2020"></a>Julio de 2020

### <a name="as-an-it-admin-i-want-to-target-client-apps-using-conditional-access"></a>Como administrador de TI, quiero dirigirme a las aplicaciones cliente mediante el acceso condicional

**Tipo:** Plan de cambio   
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
Con la versión de disponibilidad general (GA) de la condición de las aplicaciones de cliente en el acceso condicional, ahora las nuevas directivas se aplicarán de manera predeterminada a todas las aplicaciones cliente, incluidos los clientes de autenticación heredados. Las directivas existentes permanecerán sin cambios, pero la opción de alternancia *Configurar sí/no* se quitará de las directivas existentes para ver fácilmente las aplicaciones cliente que se aplican a través de la directiva. 

Al crear una nueva directiva, asegúrese de excluir los usuarios y las cuentas de servicio que siguen usando la autenticación heredada. Si no lo hace, se bloquearán. [Más información](https://aka.ms/caclientapps).
 
---

### <a name="upcoming-scim-compliance-fixes"></a>Próximas correcciones de cumplimiento de SCIM

**Tipo:** Plan de cambio  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
El servicio de aprovisionamiento de Azure AD aprovecha el estándar SCIM para la integración con las aplicaciones. Nuestra implementación del estándar SCIM está evolucionando y esperamos realizar cambios en nuestro comportamiento en torno a la realización de las operaciones de revisión, así como al establecimiento de la propiedad "activa" en un recurso. [Más información](https://docs.microsoft.com/azure/active-directory/app-provisioning/application-provisioning-config-problem-scim-compatibility).
 
---

### <a name="group-owner-setting-on-azure-admin-portal-will-be-changed"></a>Se cambiará la configuración del propietario del grupo en el portal de administración de Azure

**Tipo:** Plan de cambio  
**Categoría del servicio:** Administración de grupos  
**Funcionalidad del producto:** Colaboración

La configuración del propietario en la página de configuración de grupos general puede configurarse para restringir los privilegios de asignación de propietario a un grupo limitado de usuarios en el Panel de acceso y el portal de administración de Azure. Pronto tendremos la capacidad de asignar privilegios de propietario de grupo no solo en estos dos portales de la experiencia de usuario, sino también aplicar la directiva en el back-end para proporcionar un comportamiento coherente entre los puntos de conexión, como PowerShell y Microsoft Graph. 

Comenzaremos a deshabilitar la configuración actual de los clientes que no la estén usando y ofreceremos una opción para el ámbito de los usuarios de privilegios de propietario de grupo en los próximos meses. Para obtener instrucciones sobre la actualización de la configuración de grupo, consulte la edición de la información de grupo mediante [Azure Active Directory](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-groups-settings-azure-portal?context=azure/active-directory/users-groups-roles/context/ugr-context).

---

### <a name="azure-active-directory-registration-service-is-ending-support-for-tls-10-and-11"></a>El servicio de registro de Azure Active Directory va a finalizar la compatibilidad con TLS 1.0 y 1.1

**Tipo:** Plan de cambio  
**Categoría del servicio:** Administración y registro de dispositivos  
**Funcionalidad del producto:** plataforma

La Seguridad de la capa de transporte (TLS) 1.2 y los servidores y clientes de actualización se comunicarán pronto con el servicio Registro de dispositivos de Azure Active Directory. La compatibilidad con TLS 1.0 y 1.1 para la comunicación con el servicio Registro de dispositivos de Azure AD se retirará:
- El 31 de agosto de 2020, en todas las nubes independientes (GCC High, DoD, etc.).
- El 30 de octubre de 2020, en todas las nubes comerciales.

[Más información](https://docs.microsoft.com/azure/active-directory/devices/reference-device-registration-tls-1-2) sobre TLS 1.2 para el servicio de registro de Azure AD.

---

### <a name="windows-hello-for-business-sign-ins-visible-in-azure-ad-sign-in-logs"></a>Inicios de sesión de Windows Hello para empresas visibles en registros de inicio de sesión de Azure AD

**Tipo:** Corregido  
**Categoría del servicio:** Notificación  
**Funcionalidad del producto:** Supervisión e informes
 
Windows Hello para empresas permite a los usuarios finales iniciar sesión en máquinas Windows con un gesto (como un PIN o datos biométricos). Los administradores de Azure AD pueden querer diferenciar los inicios de sesión de Windows Hello para empresas de otros inicios de sesión de Windows como parte del recorrido de una organización para la autenticación sin contraseña. 

Los administradores ya pueden ver si una autenticación de Windows ha usado Windows Hello para empresas mediante la comprobación de la pestaña de detalles de autenticación para un evento de inicio de sesión de Windows en la hoja Inicios de sesión de Azure AD en Azure Portal. Las autenticaciones de Windows Hello para empresas incluirán "WindowsHelloForBusiness" en el campo Método de autenticación. Para obtener más información sobre cómo interpretar los registros de inicio de sesión, consulte la [documentación de registros de inicio de sesión](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-sign-ins).
 
---

### <a name="fixes-to-group-deletion-behavior-and-performance-improvements"></a>Correcciones del comportamiento de eliminación de grupos y mejoras de rendimiento

**Tipo:** Corregido  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
Anteriormente, cuando un grupo cambiaba de "en el ámbito" a "fuera de ámbito" y un administrador hacía clic en reiniciar antes de que se completara el cambio, el objeto de grupo no se eliminaba. Ahora, el objeto de grupo de la aplicación de destino se eliminará cuando salga del ámbito (deshabilitado, eliminado, sin asignar o sin pasar el filtro de ámbito). [Más información](https://docs.microsoft.com/azure/active-directory/app-provisioning/how-provisioning-works#incremental-cycles).
 
---

### <a name="public-preview-admins-can-now-add-custom-content-in-the-email-to-reviewers-when-creating-an-access-review"></a>Versión preliminar pública: los administradores ya pueden agregar contenido personalizado en el correo electrónico para los revisores al crear una revisión de acceso.

**Tipo:** Nueva característica  
**Categoría del servicio:** Revisiones de acceso  
**Funcionalidad del producto:** Identity Governance
 
Cuando se crea una nueva revisión de acceso, el revisor recibe un correo electrónico que le solicita que complete la revisión de acceso. Muchos de nuestros clientes han solicitado la posibilidad de agregar contenido personalizado al correo electrónico, como información de contacto, u otro contenido adicional de soporte técnico para guiar al revisor. 

Ya disponible en la versión preliminar pública, los administradores pueden especificar contenido personalizado en el correo electrónico que se envía a los revisores mediante la adición de contenido en la sección "Opciones avanzadas" de las revisiones de acceso de Azure AD. Para obtener instrucciones sobre cómo crear revisiones de acceso, consulte [Creación de una revisión de acceso de los grupos y las aplicaciones en las revisiones de acceso de Azure AD](https://docs.microsoft.com/azure/active-directory/governance/create-access-review).
 
---

### <a name="authorization-code-flow-for-single-page-apps-available"></a>Flujo de código de autorización para aplicaciones de una sola página disponibles

**Tipo:** Nueva característica  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Experiencia para el desarrollador
 
Debido a las restricciones de cookies de terceros como Safari ITP aplicadas en el explorador, las aplicaciones de página única (SPA) tendrán que usar el flujo de código de autorización en lugar del flujo implícito para mantener el SSO. MSAL.js v2.x admite ahora el flujo de código de autorización. 

Las actualizaciones correspondientes están disponibles en Azure Portal. De este modo, podrá actualizar sus aplicaciones de página única para que sean del tipo "spa" y utilicen el flujo de código de autenticación. Consulte [Inicio de sesión de los usuarios y obtención de un token de acceso en una SPA de JavaScript mediante el flujo de código de autorización](https://docs.microsoft.com/azure/active-directory/develop/quickstart-v2-javascript-auth-code) para obtener más instrucciones.
 
---

### <a name="azure-ad-application-proxy-now-supports-the-remote-desktop-services-web-client"></a>Azure AD Application Proxy ya admite el cliente web de Servicios de Escritorio remoto

**Tipo:** Nueva característica  
**Categoría del servicio:** Proxy de aplicaciones  
**Funcionalidad del producto:** Control de acceso

Azure AD Application Proxy ya admite el cliente web de Servicios de Escritorio remoto (RDS). El cliente web de RDS permite a los usuarios tener acceso a la infraestructura de Escritorio remoto a través de cualquier explorador compatible con HTLM5, como Microsoft Edge, Internet Explorer 11, Google Chrome, etc. Los usuarios pueden interactuar con escritorios o aplicaciones remotos como lo harían con un dispositivo local desde cualquier lugar. Mediante el uso de Azure AD Application Proxy, puede aumentar la seguridad de la implementación de RDS mediante la exigencia de directivas de acceso condicional y autenticación previa para todos los tipos de aplicaciones cliente enriquecidas. Para obtener instrucciones, consulte [Publicación de Escritorio Remoto con Azure AD Application Proxy](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-integrate-with-remote-desktop-services).
 
---

### <a name="next-generation-azure-ad-b2c-user-flows-in-public-preview"></a>Flujos de usuario de última generación de Azure AD B2C en versión preliminar pública

**Tipo:** Nueva característica  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C
 
La experiencia simplificada de flujo de usuario ofrece paridad de características con características en vista previa (GB) y es donde se encuentran todas las características nuevas. Los usuarios podrán habilitar nuevas características en el mismo flujo de usuario, lo que reduce la necesidad de crear varias versiones con cada nueva versión de actualización de características. Por último, la nueva experiencia de usuario fácil de usar simplifica la selección y creación de flujos de usuario. Para probarla, [cree un flujo de usuario](https://docs.microsoft.com/azure/active-directory-b2c/tutorial-create-user-flows). 

Para más información sobre los flujos de usuario, consulte [Versiones de flujos de usuario de Azure Active Directory B2C](https://docs.microsoft.com/azure/active-directory-b2c/user-flow-versions#:~:text=%20%20%20%20User%20flow%20%20%2caccount.%20Usi%20...%20%201%20more%20rows%20).

---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---july-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD, julio de 2020

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En julio de 2020, agregamos 55 aplicaciones nuevas a la galería de aplicaciones con compatibilidad de federación:

[Clap Your Hands](http://www.rmit.com.ar/), [Appreiz](https://microsoftteams.appreiz.com/), [Inextor Vault](https://inexto.com/inexto-suite/inextor), [Beekast](https://my.beekast.com/), [Templafy OpenID Connect](https://app.templafy.com/), [PeterConnects receptionist](https://msteams.peterconnects.com/), [AlohaCloud](https://appfusions.alohacloud.com/auth), [Control Tower](https://bpm.tnxcorp.com/sso/microsoft), [Cocoom](https://start.cocoom.com/), [COINS Construction Cloud](https://sso.coinsconstructioncloud.com/#login/), [Medxnote MT](https://task.teamsmain.medx.im/authorization), [Reflekt](https://reflekt.konsolute.com/login), [Rever](https://app.reverscore.net/access), [MyCompanyArchive](https://login.mycompanyarchive.com/), [GReminders](https://app.greminders.com/o365-oauth), [Titanfile](https://docs.microsoft.com/azure/active-directory/saas-apps/titanfile-tutorial), [Wootric](https://docs.microsoft.com/azure/active-directory/saas-apps/wootric-tutorial), [SolarWinds Orion](https://support.solarwinds.com/SuccessCenter/s/orion-platform?language=en_US),  [OpenText Directory Services](https://docs.microsoft.com/azure/active-directory/saas-apps/opentext-directory-services-tutorial), [Datasite](https://docs.microsoft.com/azure/active-directory/saas-apps/datasite-tutorial), [BlogIn](https://docs.microsoft.com/azure/active-directory/saas-apps/blogin-tutorial), [IntSights](https://docs.microsoft.com/azure/active-directory/saas-apps/intsights-tutorial), [kpifire](https://docs.microsoft.com/azure/active-directory/saas-apps/kpifire-tutorial), [Textline](https://docs.microsoft.com/azure/active-directory/saas-apps/textline-tutorial), [Cloud Academy - SSO](https://docs.microsoft.com/azure/active-directory/saas-apps/cloud-academy-sso-tutorial), [Community Spark](https://docs.microsoft.com/azure/active-directory/saas-apps/community-spark-tutorial), [Chatwork](https://docs.microsoft.com/azure/active-directory/saas-apps/chatwork-tutorial), [CloudSign](https://docs.microsoft.com/azure/active-directory/saas-apps/cloudsign-tutorial), [C3M Cloud Control](https://docs.microsoft.com/azure/active-directory/saas-apps/c3m-cloud-control-tutorial), [SmartHR](https://smarthr.jp/), [NumlyEngage™](https://docs.microsoft.com/azure/active-directory/saas-apps/numlyengage-tutorial), [Michigan Data Hub Single Sign-On](https://docs.microsoft.com/azure/active-directory/saas-apps/michigan-data-hub-single-sign-on-tutorial), [Egress](https://docs.microsoft.com/azure/active-directory/saas-apps/egress-tutorial), [SendSafely](https://docs.microsoft.com/azure/active-directory/saas-apps/sendsafely-tutorial), [Eletive](https://app.eletive.com/), [Right-Hand Cybersecurity ADI](https://right-hand.ai/), [Fyde Enterprise Authentication](https://enterprise.fyde.com/), [Verme](https://docs.microsoft.com/azure/active-directory/saas-apps/verme-tutorial), [Lenses.io](https://docs.microsoft.com/azure/active-directory/saas-apps/lensesio-tutorial), [Momenta](https://docs.microsoft.com/azure/active-directory/saas-apps/momenta-tutorial), [Uprise](https://app.uprise.co/sign-in), [Q](https://q.moduleq.com/login), [CloudCords](https://docs.microsoft.com/azure/active-directory/saas-apps/cloudcords-tutorial), [TellMe Bot](https://tellme365liteweb.azurewebsites.net/), [Inspire](https://app.inspiresoftware.com/), [Maverics Identity Orchestrator SAML Connector](https://www.strata.io/identity-fabric/), [Smartschool (School Management System)](https://smart-schoolapp.com/frmLoginForm), [Zepto - Intelligent timekeeping](https://user.zepto-ai.com/signin), [Studi.ly](https://studi.ly/), [Trackplan](http://www.trackplanfm.com/), [Skedda](https://docs.microsoft.com/azure/active-directory/saas-apps/skedda-tutorial), [WhosOnLocation](https://docs.microsoft.com/azure/active-directory/saas-apps/whos-on-location-tutorial), [Coggle](https://docs.microsoft.com/azure/active-directory/saas-apps/coggle-tutorial), [Kemp LoadMaster](https://kemptechnologies.com/cloud-load-balancer/), [BrowserStack Single Sign-on](https://docs.microsoft.com/azure/active-directory/saas-apps/browserstack-single-sign-on-tutorial)

La documentación de todas las aplicaciones está disponible en https://aka.ms/AppsTutorial.

Para mostrar su aplicación en la galería de aplicaciones de Azure AD, lea los detalles aquí: https://aka.ms/AzureADAppRequest.

---

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---july-2020"></a>Nuevos conectores de aprovisionamiento disponibles en la galería de aplicaciones de Azure AD, julio de 2020

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Integración de terceros

Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para la aplicación recién integrada [LinkedIn Learning](https://docs.microsoft.com/azure/active-directory/saas-apps/linkedin-learning-provisioning-tutorial).

Para más información acerca de cómo proteger mejor una organización mediante el aprovisionamiento automatizado de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).

---

### <a name="view-role-assignments-across-all-scopes-and-ability-to-download-them-to-a-csv-file"></a>Visualización de las asignaciones de roles en todos los ámbitos y capacidad de descargarlas en un archivo CSV

**Tipo:** Característica modificada  
**Categoría del servicio:** RBAC  
**Funcionalidad del producto:** Control de acceso
 
Ahora puede ver las asignaciones de roles en todos los ámbitos de un rol en la pestaña "Roles y administradores" en el portal de Azure AD. También puede descargar estas asignaciones de roles para cada rol en un archivo CSV. Para obtener instrucciones sobre cómo ver y agregar asignaciones de roles, consulte [Visualización y asignación de roles de administrador en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-manage-roles-portal).
 
---

### <a name="azure-multi-factor-authentication-software-development-azure-mfa-sdk-deprecation"></a>Desuso del desarrollo de software de Azure Multi-Factor Authentication (SDK de Azure MFA)

**Tipo:** Obsoleto  
**Categoría del servicio:** MFA  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
El desarrollo de software de Azure Multi-Factor Authentication (SDK de Azure MFA) alcanzó el final del ciclo de vida el 14 de noviembre de 2018, como se anunció por primera vez en noviembre de 2017. Microsoft apagará el servicio de SDK a partir del 30 de septiembre de 2020. Se producirá un error en todas las llamadas realizadas al SDK.

Si su organización usa el SDK de Azure MFA, debe realizar la migración antes del 30 de septiembre de 2020:
- SDK de Azure MFA para MIM:  Si usa el SDK con MIM, debe migrar al Servidor Azure MFA y activar Privileged Access Management (PAM) siguiendo estas [instrucciones](https://docs.microsoft.com/microsoft-identity-manager/working-with-mfaserver-for-mim).   
- SDK de Azure MFA para aplicaciones personalizadas: Considere la posibilidad de integrar la aplicación en Azure AD y usar el acceso condicional para aplicar MFA. Para empezar, revise esta [página](https://docs.microsoft.com/azure/active-directory/manage-apps/plan-an-application-integration). 

---

## <a name="june-2020"></a>Junio de 2020 

### <a name="user-risk-condition-in-conditional-access-policy"></a>Condición de riesgo de usuario de la directiva de acceso condicional

**Tipo:** Plan de cambio  
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 

La compatibilidad con el riesgo de usuario en la directiva de acceso condicional de Azure AD permite crear varias directivas basadas en el riesgo de usuario. Pueden requerirse diferentes niveles de riesgo de usuario mínimo para diferentes usuarios y aplicaciones. En función del riesgo de usuario, puede crear directivas para bloquear el acceso, requerir la autenticación multifactor, proteger el cambio de contraseña o redirigir a Microsoft Cloud App Security para aplicar la directiva de sesión, como la auditoría adicional.

La condición de riesgo de usuario requiere Azure AD Premium P2 porque usa Azure Identity Protection, que es una oferta P2. Para más información sobre el acceso condicional, vea la [documentación sobre el acceso condicional de Azure AD](https://docs.microsoft.com/azure/active-directory/conditional-access/).

---

### <a name="saml-sso-now-supports-apps-that-require-spnamequalifier-to-be-set-when-requested"></a>SSO de SAML admite ahora aplicaciones que requieren establecer SPNameQualifier cuando se solicite

**Tipo:** Corregido  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** SSO
 
Algunas aplicaciones de SAML requieren que se devuelva SPNameQualifier en el sujeto de la aserción cuando se solicita. Ahora Azure AD responde correctamente cuando se solicita un SPNameQualifier en la directiva de NameID de la solicitud. Esto también funciona para el inicio de sesión iniciado por el SP y se seguirá el inicio de sesión iniciado por IdP.  Para más información sobre el protocolo de SAML en Azure Active Directory, vea [Protocolo SAML de inicio de sesión único](https://docs.microsoft.com/azure/active-directory/develop/single-sign-on-saml-protocol).

---

### <a name="azure-ad-b2b-collaboration-supports-inviting-msa-and-google-users-in-azure-government-tenants"></a>La colaboración B2B de Azure AD permite invitar a los usuarios de MSA y Google en inquilinos de Azure Government

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 

Los inquilinos de Azure Government que usan las características de colaboración B2B ahora pueden invitar a los usuarios que tienen una cuenta de Microsoft o Google. Para averiguar si su inquilino puede usar estas funcionalidades, siga las instrucciones que se indican en [¿Cómo puedo saber si la colaboración B2B está disponible en mi inquilino de Azure US Government?](https://docs.microsoft.com/azure/active-directory/b2b/current-limitations#how-can-i-tell-if-b2b-collaboration-is-available-in-my-azure-us-government-tenant)

 
---
 
### <a name="user-object-in-ms-graph-v1-now-includes-externaluserstate-and-externaluserstatechangeddatetime-properties"></a>El objeto de usuario de MS Graph v1 ahora incluye las propiedades externalUserState y externalUserStateChangedDateTime

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 

Las propiedades externalUserState y externalUserStateChangedDateTime se pueden usar para buscar invitados B2B que aún no han aceptado sus invitaciones, así como para crear automatización, como la eliminación de usuarios que no han aceptado sus invitaciones después de un número determinado de días. Estas propiedades están ahora disponibles en MS Graph v1. Para obtener instrucciones sobre el uso de estas propiedades, consulte [Tipo de recurso del usuario](https://docs.microsoft.com/graph/api/resources/user?view=graph-rest-1.0).
 
---

### <a name="manage-authentication-sessions-in-azure-ad-conditional-access-is-now-generally-available"></a>La administración de sesiones de autenticación en el acceso condicional de Azure AD ya está disponible con carácter general

**Tipo:** Nueva característica  
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
Las funcionalidades de administración de sesiones de autenticación permiten configurar la frecuencia con que los usuarios necesitan proporcionar credenciales de inicio de sesión y si necesitan proporcionar credenciales después de cerrar y volver a abrir los exploradores para ofrecer más seguridad y flexibilidad en el entorno.
 
Además, la administración de sesiones de autenticación se aplicaba solo a la autenticación del primer factor en los dispositivos unidos a Azure AD, unidos a Azure AD híbrido y registrados en Azure AD. Ahora la administración de sesiones de autenticación también se aplicará a MFA. Para más información, vea [Configuración de la administración de las sesiones de autenticación con el acceso condicional](https://docs.microsoft.com/azure/active-directory/conditional-access/howto-conditional-access-session-lifetime).

---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---june-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD, junio de 2020

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En junio de 2020, hemos agregado 29 nuevas aplicaciones a la galería de aplicaciones con compatibilidad de federación:

[Shopify Plus](https://docs.microsoft.com/azure/active-directory/saas-apps/shopify-plus-tutorial), [Ekarda](https://docs.microsoft.com/azure/active-directory/saas-apps/ekarda-tutorial), [MailGates](https://docs.microsoft.com/azure/active-directory/saas-apps/mailgates-tutorial), [BullseyeTDP](https://docs.microsoft.com/azure/active-directory/saas-apps/bullseyetdp-tutorial), [Raketa](https://docs.microsoft.com/azure/active-directory/saas-apps/raketa-tutorial), [Segment](https://docs.microsoft.com/azure/active-directory/saas-apps/segment-tutorial), [Ai Auditor](https://www.mindbridge.ai/products/ai-auditor/), [Pobuca Connect](https://app.pobu.ca/), [Proto.io](https://docs.microsoft.com/azure/active-directory/saas-apps/proto.io-tutorial), [Gatekeeper](https://www.gatekeeperhq.com/), [Hub Planner](https://docs.microsoft.com/azure/active-directory/saas-apps/hub-planner-tutorial), [Ansira-Partner Go-to-Market Toolbox](https://ansira.com/technology/channel-engagement), [IBM Digital Business Automation on Cloud](https://docs.microsoft.com/azure/active-directory/saas-apps/ibm-digital-business-automation-on-cloud-tutorial), [Kisi Physical Security](https://docs.microsoft.com/azure/active-directory/saas-apps/kisi-physical-security-tutorial), [ViewpointOne](https://team.viewpoint.com/), [IntelligenceBank](https://docs.microsoft.com/azure/active-directory/saas-apps/intelligencebank-tutorial), [pymetrics](https://docs.microsoft.com/azure/active-directory/saas-apps/pymetrics-tutorial), [Zero](https://www.teamzero.com/), [InStation](https://instation.invillia.com/), [edX for Business SAML 2.0 Integration](https://docs.microsoft.com/azure/active-directory/saas-apps/edx-for-business-saml-integration-tutorial), [MOOC Office 365](https://mooc.office365-training.com/en/), [SmartKargo](https://docs.microsoft.com/azure/active-directory/saas-apps/smartkargo-tutorial), [PKIsigning platform](https://platform.pkisigning.nl/), [SiteIntel](https://docs.microsoft.com/azure/active-directory/saas-apps/siteintel-tutorial), [Field iD](https://docs.microsoft.com/azure/active-directory/saas-apps/field-id-tutorial), [Curricula SAML](https://docs.microsoft.com/azure/active-directory/saas-apps/curricula-saml-tutorial), [Perforce Helix Core - Helix Authentication Service](https://docs.microsoft.com/azure/active-directory/saas-apps/perforce-helix-core-tutorial), [MyCompliance Cloud](https://cloud.metacompliance.com/) y [Smallstep SSH](https://smallstep.com/sso-ssh/)  

La documentación de todas las aplicaciones está disponible aquí: https://aka.ms/AppsTutorial Para mostrar su aplicación en la galería de aplicaciones Azure AD, lea los detalles aquí: https://aka.ms/AzureADAppRequest.

---

### <a name="api-connectors-for-external-identities-self-service-sign-up-are-now-in-public-preview"></a>Los conectores de API para el registro de autoservicio de identidades externas ya está disponible en versión preliminar pública

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
Los conectores de API para identidades externas permiten utilizar las API web para integrar el registro de autoservicio con sistemas en la nube externos. Esto significa que ahora puede invocar las API web como pasos específicos en un flujo de registro para desencadenar flujos de trabajo personalizados basados en la nube. Por ejemplo, puede usar conectores de API para:

- Integrarse con un flujo de trabajo de aprobación personalizado.
- Realizar correcciones de identidad
- Validar los datos de entrada del usuario
- Sobrescribir atributos de usuario
- Ejecutar una lógica de negocios personalizada

Para más información sobre todas las experiencias posibles con los conectores de API, vea [Uso de conectores de API para personalizar y extender el registro de autoservicio](https://docs.microsoft.com/azure/active-directory/b2b/api-connectors-overview) o [Personalización del registro de autoservicio de las identidades externas con las integraciones de API web](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/customize-external-identities-self-service-sign-up-with-web-api/ba-p/1257364#.XvNz2fImuQg.linkedin).
 
---

### <a name="provision-on-demand-and-get-users-into-your-apps-in-seconds"></a>Aprovisionamiento a petición e integración de usuarios en las aplicaciones en cuestión de segundos

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
El servicio de aprovisionamiento de Azure AD actualmente funciona de forma cíclica. El servicio se ejecuta cada 40 minutos. La [funcionalidad de aprovisionamiento a petición](https://aka.ms/provisionondemand) permite elegir un usuario y aprovisionarlo en segundos. Esta funcionalidad permite solucionar problemas de aprovisionamiento rápidamente, sin tener que reiniciar para forzar el inicio del ciclo de aprovisionamiento. 
 
---

### <a name="new-permission-for-using-azure-ad-entitlement-management-in-graph"></a>Nuevo permiso para usar la administración de derechos de Azure AD en Graph

**Tipo:** Nueva característica  
**Categoría del servicio:** Otros  
**Funcionalidad del producto:** Administración de derechos
 
Un nuevo permiso delegado EntitlementManagement.Read.All ahora está disponible para su uso con la API de administración de derechos en la versión beta de Microsoft Graph. Para obtener más información sobre las API disponibles, vea [Trabajar con la API de administración de derechos de Azure AD](https://docs.microsoft.com/graph/api/resources/entitlementmanagement-root?view=graph-rest-beta).

---

### <a name="identity-protection-apis-available-in-v10"></a>API de Identity Protection disponibles en la versión 1.0

**Tipo:** Nueva característica  
**Categoría del servicio:** Protección de identidad  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
Las API riskyUsers y riskDetections de Microsoft Graph ahora están disponibles con carácter general. Ahora que están disponibles en el punto de conexión v 1.0, lo invitamos a utilizarlos en producción. Para más información, vea la [documentación de Microsoft Graph](https://docs.microsoft.com/graph/api/resources/identityprotectionroot?view=graph-rest-1.0).
 
---

### <a name="sensitivity-labels-to-apply-policies-to-microsoft-365-groups-is-now-generally-available"></a>Las etiquetas de confidencialidad para aplicar directivas a grupos de Microsoft 365 ya están disponibles con carácter general

**Tipo:** Nueva característica  
**Categoría del servicio:** Administración de grupos  
**Funcionalidad del producto:** Colaboración
 

Ahora puede crear etiquetas de confidencialidad y usar la configuración de la etiqueta para aplicar directivas a grupos de Microsoft 365, incluida la directiva de privacidad (pública o privada) y la directiva de acceso de usuarios externos. Puede crear una etiqueta con la directiva de privacidad para que sea privada y con la directiva de acceso de usuarios externos para no permitir que se agreguen usuarios invitados. Cuando un usuario aplica esta etiqueta a un grupo, este será privado y no se permitirá agregar usuarios invitados al grupo. 

Las etiquetas de confidencialidad son importantes para proteger los datos críticos para la empresa y permiten administrar grupos a escala, de manera compatible y segura. Para obtener orientación sobre el uso de las etiquetas de confidencialidad, vea [Asignación de etiquetas de confidencialidad a grupos de Office 365 en Azure Active Directory (versión preliminar)](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-assign-sensitivity-labels).
 
---

### <a name="updates-to-support-for-microsoft-identity-manager-for-azure-ad-premium-customers"></a>Actualizaciones de compatibilidad con Microsoft Identity Manager para clientes de Azure AD Premium

**Tipo:** Característica modificada  
**Categoría del servicio:** Microsoft Identity Manager  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
El Soporte técnico de Azure ya está disponible para los componentes de integración de Azure AD de Microsoft Identity Manager 2016, hasta el final del soporte extendido para Microsoft Identity Manager 2016. Para más información, vea [Actualización de soporte técnico para clientes de Azure AD Premium que usan Microsoft Identity Manager](https://docs.microsoft.com/microsoft-identity-manager/support-update-for-azure-active-directory-premium-customers).

---

### <a name="the-use-of-group-membership-conditions-in-sso-claims-configuration-is-increased"></a>Aumento del uso de condiciones de pertenencia a grupos en la configuración de notificaciones de SSO

**Tipo:** Característica modificada  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** SSO
 
Anteriormente, el número de grupos que podía usar al cambiar condicionalmente las notificaciones basadas en la pertenencia a grupos dentro de cualquier configuración de aplicación única estaba limitado a 10. El uso de las condiciones de pertenencia a grupos en la configuración de notificaciones de SSO ahora se ha aumentado a un máximo de 50 grupos. Para más información sobre cómo configurar las notificaciones, vea [Configuración de notificaciones de SSO en aplicaciones empresariales](https://docs.microsoft.com/azure/active-directory/develop/active-directory-saml-claims-customization#emitting-claims-based-on-conditions). 

---

### <a name="enabling-basic-formatting-on-the-sign-in-page-text-component-in-company-branding"></a>Habilitar el formato básico en el componente de texto de la página de inicio de sesión para la personalización de marca de empresas

**Tipo:** Característica modificada  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Autenticación de usuarios
 
La funcionalidad de personalización de marca de la empresa en la experiencia de inicio de sesión de Azure AD/Microsoft 365 se ha actualizado para que el cliente pueda agregar hipervínculos y formatos sencillos, como negrita, subrayado y cursiva. Para obtener orientación sobre el uso de esta funcionalidad, vea [Incorporación de la personalización de marca en la página de inicio de sesión de Azure Active Directory de la organización](https://docs.microsoft.com/azure/active-directory/fundamentals/customize-branding).

---

### <a name="provisioning-performance-improvements"></a>Mejoras en el rendimiento de aprovisionamiento

**Tipo:** Característica modificada  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
El servicio de aprovisionamiento se ha actualizado para reducir el tiempo que tarda un [ciclo incremental](https://docs.microsoft.com/azure/active-directory/app-provisioning/how-provisioning-works#incremental-cycles) en completarse. Esto significa que los usuarios y grupos se aprovisionarán en sus aplicaciones más rápido que antes. Todos los nuevos trabajos de aprovisionamiento creados después del 10/6/2020 se beneficiarán automáticamente de las mejoras de rendimiento. Las aplicaciones configuradas para el aprovisionamiento antes del 10/6/2020 deberán reiniciarse una vez después del 10/6/2020 para beneficiarse de las mejoras de rendimiento. 

---

### <a name="announcing-the-deprecation-of-adal-and-ms-graph-parity"></a>Anuncio de desuso de la paridad de ADAL y MS Graph

**Tipo:** Obsoleto  
**Categoría del servicio:** N/D  
**Funcionalidad del producto:** Administración del ciclo de vida de dispositivos

Ahora que las bibliotecas de autenticación de Microsoft (MSAL) están disponibles, ya no se agregarán nuevas características a las bibliotecas de autenticación de Azure Active Directory (ADAL) y las revisiones de seguridad finalizarán el 30 de junio de 2022. Para más información sobre cómo migrar a MSAL, vea [Migración de aplicaciones a la Biblioteca de autenticación de Microsoft (MSAL)](https://docs.microsoft.com/azure/active-directory/develop/msal-migration).

Además, hemos finalizado el trabajo para que toda la funcionalidad de Azure AD Graph esté disponible mediante MS Graph. Por lo tanto, las Graph API de Azure AD solo recibirán correcciones de errores y de seguridad hasta el 30 de junio de 2022. Para más información, vea [Actualización de las aplicaciones para usar la Biblioteca de autenticación de Microsoft y Microsoft Graph API](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/update-your-applications-to-use-microsoft-authentication-library/ba-p/1257363).
 
---
 
## <a name="may-2020"></a>Mayo de 2020

### <a name="retirement-of-properties-in-signins-riskyusers-and-riskdetections-apis"></a>Retirada de propiedades en las API signIns, riskyUsers y riskDetections

**Tipo:** Plan de cambio  
**Categoría del servicio:** Protección de identidad  
**Funcionalidad del producto:** Seguridad y protección de la identidad

Actualmente, los tipos enumerados se usan para representar la propiedad riskType en riskDetections API y riskyUserHistoryItem (en versión preliminar). Los tipos enumerados también se usan para la propiedad riskEventTypes de signIns API. En el futuro, se representarán estas propiedades como cadenas. 

Los clientes deben realizar la transición a la propiedad riskEventType de la versión beta de riskDetections y riskyUserHistoryItem API, así como a la propiedad riskEventTypes_v2 de la versión beta de signIns API antes del 9 de septiembre de 2020. En esa fecha, se retirarán las propiedades actuales riskType y riskEventTypes. Para más información, vea [Cambios en las propiedades de eventos de riesgo y en las API de Identity Protection en Microsoft Graph](https://developer.microsoft.com/graph/blogs/changes-to-risk-event-properties-and-identity-protection-apis-on-microsoft-graph/).

--- 

### <a name="deprecation-of-riskeventtypes-property-in-signins-v10-api-on-microsoft-graph"></a>Desuso de la propiedad riskEventTypes en signIns v1.0 API en Microsoft Graph

**Tipo:** Plan de cambio  
**Categoría del servicio:** Notificación  
**Funcionalidad del producto:** Seguridad y protección de la identidad

Los tipos enumerados cambiarán a tipos de cadena al representar las propiedades de evento de riesgo en Microsoft Graph en septiembre de 2020. Además de afectar a las API en versión preliminar, este cambio también afectará a signIns API en producción.

Hemos introducido una nueva propiedad riskEventsTypes_v2 (cadena) en signIns v1.0 API. Se retirará la propiedad riskEventTypes (enumeración) actual el 11 de junio de 2022 de acuerdo con la directiva de desuso de Microsoft Graph. Los clientes deben realizar la transición a la propiedad riskEventTypes_v2 de v1.0 signIns API antes del 11 de junio de 2022. Para más información, vea [Desuso de la propiedad riskEventTypes en signIns v1.0 API en Microsoft Graph](https://developer.microsoft.com/graph/blogs/deprecation-of-riskeventtypes-property-in-signins-v1-0-api-on-microsoft-graph//).

--- 

### <a name="upcoming-changes-to-mfa-email-notifications"></a>Próximos cambios en las notificaciones por correo electrónico de MFA

**Tipo:** Plan de cambio  
**Categoría del servicio:** MFA  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 

Vamos a realizar los siguientes cambios en las notificaciones por correo electrónico de MFA en la nube:

Las notificaciones por correo electrónico se enviarán desde las siguientes direcciones: azure-noreply@microsoft.com y msonlineservicesteam@microsoftonline.com. Estamos actualizando el contenido de los correos electrónicos de alerta sobre fraudes a fin de explicar mejor los pasos necesarios para desbloquear los usos.

---

### <a name="new-self-service-sign-up-for-users-in-federated-domains-who-cant-access-microsoft-teams-because-they-arent-synced-to-azure-active-directory"></a>Nuevo registro de autoservicio para usuarios de dominios federados que no pueden acceder a Microsoft Teams porque no están sincronizados con Azure Active Directory.

**Tipo:** Plan de cambio  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Autenticación de usuarios
 

Actualmente, los usuarios que están en dominios federados en Azure AD, pero que no están sincronizados en el inquilino, no pueden acceder a Teams. A partir de finales de junio, esta nueva funcionalidad les permitirá hacerlo gracias a la extensión de la característica de registro con verificación por correo electrónico. Esto permitirá a los usuarios iniciar sesión en un proveedor de identidades (IdP) federado. En el caso de los usuarios que aún no tengan un objeto de usuario en Azure ID, el objeto se creará automáticamente y se autenticará para Teams. El objeto de usuario se marcará como "registro de autoservicio". Se trata de una extensión de la característica existente para que los usuarios de los dominios administrados puedan registrarse mediante la verificación por correo electrónico y se pueda llevar un control utilizando la misma marca. La implementación de este cambio se llevará a cabo durante los dos próximos meses. Vea las actualizaciones de la documentación [aquí](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-self-service-signup).
 
---

### <a name="upcoming-fix-the-oidc-discovery-document-for-the-azure-government-cloud-is-being-updated-to-reference-the-correct-graph-endpoints"></a>Próxima corrección: El documento de detección de OIDC para la nube de Azure Government se está actualizando para que haga referencia a los puntos de conexión de Graph correctos.

**Tipo:** Plan de cambio  
**Categoría del servicio:** Nubes soberanas  
**Funcionalidad del producto:** Autenticación de usuarios
 
A partir de junio, el documento de detección de OIDC [Plataforma de identidad de Microsoft y protocolo OpenID Connect](https://docs.microsoft.com/azure/active-directory/develop/v2-protocols-oidc) sobre el punto de conexión de la [nube de Azure Government](https://docs.microsoft.com/azure/active-directory/develop/authentication-national-cloud) (login.microsoftonline.us), comenzará a devolver el punto de conexión de [Graph de la nube nacional ](https://docs.microsoft.com/graph/deployments) correcto (https://graph.microsoft.us o https://dod-graph.microsoft.us), según el inquilino proporcionado).  Actualmente, proporciona el campo "msgraph_host" del punto de conexión de Graph incorrecto (graph.microsoft.com).  

Esta corrección de errores se implementará gradualmente durante 2 meses aproximadamente.  

---

### <a name="azure-government-users-will-no-longer-be-able-to-sign-in-on-loginmicrosoftonlinecom"></a>Los usuarios de Azure Government ya no podrán iniciar sesión en login.microsoftonline.com

**Tipo:** Plan de cambio  
**Categoría del servicio:** Nubes soberanas  
**Funcionalidad del producto:** Autenticación de usuarios
 
El 1 de junio de 2018, la autoridad oficial de Azure Active Directory (AAD) para Azure Government cambió de https://login-us.microsoftonline.com a https://login.microsoftonline.us. Si posee una aplicación en un inquilino de Azure Government, tiene que actualizarla para que los usuarios inicien sesión en el punto de conexión .us.

El 5 de mayo, Azure AD comenzó a aplicar el cambio relativo al punto de conexión, por lo que los usuarios de Azure Government no podrán iniciar sesión en aplicaciones hospedadas en inquilinos de Azure Government mediante el punto de conexión público (microsoftonline.com). Las aplicaciones afectadas comenzarán a obtener el error AADSTS900439 - USGClientNotSupportedOnPublicEndpoint. 

Este cambio se implementará de forma gradual y se espera que se haya completado para todas las aplicaciones en junio de 2020. Para más información, consulte la [entrada de blog sobre Azure Government](https://devblogs.microsoft.com/azuregov/azure-government-aad-authority-endpoint-update/).

---

### <a name="saml-single-logout-request-now-sends-nameid-in-the-correct-format"></a>La solicitud de cierre de sesión único de SAML ahora envía el valor de NameID con el formato correcto

**Tipo:** Corregido  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Autenticación de usuarios
 
Cuando un usuario hace clic para cerrar sesión (por ejemplo, en el portal Mis aplicaciones), Azure AD envía un mensaje de cierre de sesión único de SAML a cada aplicación que esté activa en la sesión de usuario y tenga configurada una dirección URL de cierre de sesión. Estos mensajes contienen un valor de NameID con formato persistente.

Si el token de inicio de sesión de SAML original utilizaba un formato diferente para NameID (por ejemplo, correo electrónico o UPN), la aplicación SAML no podía correlacionar el valor de NameID en el mensaje de cierre de sesión para una sesión existente, dado que los valores de NameID utilizados en ambos mensajes eran diferentes. Esto hacía que la aplicación SAML descartara el mensaje de cierre de sesión y que la sesión del usuario se mantuviera activa. Esta corrección hace que el mensaje de cierre de sesión sea coherente con el valor de NameID configurado para la aplicación.

---

### <a name="hybrid-identity-administrator-role-is-now-available-with-cloud-provisioning"></a>El rol de administrador de identidades híbridas ahora está disponible con aprovisionamiento en la nube

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento en la nube de Azure AD  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
Los administradores de TI pueden empezar a usar el nuevo rol de administrador de identidades híbridas como el rol con privilegios mínimos para configurar el aprovisionamiento en la nube de Azure ADConnect. Con este nuevo rol, ya no tiene que usar el rol de administrador global para instalar y configurar el aprovisionamiento en la nube. [Más información](https://docs.microsoft.com/azure/active-directory/users-groups-roles/roles-delegate-by-task#connect).
 
---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---may-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD, mayo de 2020

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En mayo de 2020, hemos agregado 36 nuevas aplicaciones a la galería de aplicaciones con compatibilidad de federación:

[Moula](https://moula.com.au/pay/merchants), [Surveypal](https://www.surveypal.com/app), [Kbot365](https://www.konverso.ai/virtual-assistant-digital-workplace/), [TackleBox](http://www.tacklebox.app/), [Powell Teams](https://powell-software.com/en/powell-teams-en/), [Talentsoft Assistant](https://msteams.talent-soft.com/), [ASC Recording Insights](https://teams.asc-recording.app/product), [GO1](https://www.go1.com/), [B-Engaged](https://b-engaged.se/), [Competella Contact Center Workgroup](http://www.competella.com/), [Asite](http://www.asite.com/), [ImageSoft Identity](https://identity.imagesoftinc.com/), [My IBISWorld](https://identity.imagesoftinc.com/), [insuite](https://docs.microsoft.com/azure/active-directory/saas-apps/insuite-tutorial), [Change Process Management](https://docs.microsoft.com/azure/active-directory/saas-apps/change-process-management-tutorial), [Cyara CX Assurance Platform](https://docs.microsoft.com/azure/active-directory/saas-apps/cyara-cx-assurance-platform-tutorial), [Smart Global Governance](https://docs.microsoft.com/azure/active-directory/saas-apps/smart-global-governance-tutorial), [Prezi](https://docs.microsoft.com/azure/active-directory/saas-apps/prezi-tutorial), [Mapbox](https://docs.microsoft.com/azure/active-directory/saas-apps/mapbox-tutorial), [Datava Enterprise Service Platform](https://docs.microsoft.com/azure/active-directory/saas-apps/datava-enterprise-service-platform-tutorial), [Whimsical](https://docs.microsoft.com/azure/active-directory/saas-apps/whimsical-tutorial), [Trelica](https://docs.microsoft.com/azure/active-directory/saas-apps/trelica-tutorial), [EasySSO for Confluence](https://docs.microsoft.com/azure/active-directory/saas-apps/easysso-for-confluence-tutorial), [EasySSO for BitBucket](https://docs.microsoft.com/azure/active-directory/saas-apps/easysso-for-bitbucket-tutorial), [EasySSO for Bamboo](https://docs.microsoft.com/azure/active-directory/saas-apps/easysso-for-bamboo-tutorial), [Torii](https://docs.microsoft.com/azure/active-directory/saas-apps/torii-tutorial), [Axiad Cloud](https://docs.microsoft.com/azure/active-directory/saas-apps/axiad-cloud-tutorial), [Humanage](https://docs.microsoft.com/azure/active-directory/saas-apps/humanage-tutorial), [ColorTokens ZTNA](https://docs.microsoft.com/azure/active-directory/saas-apps/colortokens-ztna-tutorial), [CCH Tagetik](https://docs.microsoft.com/azure/active-directory/saas-apps/cch-tagetik-tutorial), [ShareVault](https://docs.microsoft.com/azure/active-directory/saas-apps/sharevault-tutorial), [Vyond](https://docs.microsoft.com/azure/active-directory/saas-apps/vyond-tutorial), [TextExpander](https://docs.microsoft.com/azure/active-directory/saas-apps/textexpander-tutorial), [Anyone Home CRM](https://docs.microsoft.com/azure/active-directory/saas-apps/anyone-home-crm-tutorial), [askSpoke](https://docs.microsoft.com/azure/active-directory/saas-apps/askspoke-tutorial), [ice Contact Center](https://docs.microsoft.com/azure/active-directory/saas-apps/ice-contact-center-tutorial)

La documentación de todas las aplicaciones está disponible aquí: https://aka.ms/AppsTutorial

Para mostrar su aplicación en la galería de aplicaciones Azure AD, lea los detalles aquí: https://aka.ms/AzureADAppRequest

---

### <a name="report-only-mode-for-conditional-access-is-now-generally-available"></a>El modo Solo informe para el Acceso condicional ya está disponible con carácter general.

**Tipo:** Nueva característica  
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad

El [modo Solo informe para el Acceso condicional de Azure AD](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-conditional-access-report-only) le permite evaluar el resultado de una directiva sin necesidad de aplicar controles de acceso. Puede probar las directivas del modo Solo informe en toda la organización y comprender su impacto antes de habilitarlas, lo que hará que la implementación sea más segura y fácil. En los últimos meses, hemos observado una importante adopción del modo de solo informe, y más de 26 millones de usuarios están incluidos en el ámbito de la directiva de solo informe. Con el anuncio de hoy, se crearán nuevas directivas de acceso condicional de Azure AD en el modo de solo informe de forma predeterminada. Esto significa que puede supervisar el impacto de las directivas desde el momento en el que se crean. Además, aquellos usuarios que utilizan Microsoft Graph API, también pueden [administrar directivas de solo informe mediante programación](https://docs.microsoft.com/graph/api/resources/conditionalaccesspolicy?view=graph-rest-beta). 

---

### <a name="self-service-sign-up-for-guest-users"></a>Registro de autoservicio para usuarios invitados

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
Mediante las identidades externas en Azure AD, puede facilitar a personas ajenas a la organización el acceso a sus aplicaciones y recursos, además de permitirles iniciar sesión con la identidad que prefieran. Al compartir una aplicación con usuarios externos, es posible que no siempre sepa de antemano quién necesitará tener acceso a la aplicación. Con el [registro de autoservicio](https://docs.microsoft.com/azure/active-directory/b2b/self-service-sign-up-overview), puede facilitar a los usuarios invitados el registro y el acceso a una cuenta de invitado para sus aplicaciones de línea de negocio (LOB). El flujo de registro se puede crear y personalizar para admitir Azure AD e identidades sociales. También puede recopilar información adicional sobre el usuario durante el registro.

---

 ### <a name="conditional-access-insights-and-reporting-workbook-is-generally-available"></a>El libro Conditional Access Insights and Reporting (Información detallada e informes del acceso condicional) está disponible con carácter general

**Tipo:** Nueva característica  
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad

El [libro Conditional Access Insights and Reporting (Información detallada e informes del acceso condicional)](https://docs.microsoft.com/azure/active-directory/conditional-access/howto-conditional-access-insights-reporting) ofrece a los administradores un resumen del acceso condicional de Azure AD en su inquilino. Al poder seleccionar una directiva individual, los administradores pueden comprender mejor lo que hace cada directiva y supervisar los cambios en tiempo real. El libro transmite los datos almacenados en Azure Monitor, que puede configurar en unos minutos [según estas instrucciones](https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics). Para encontrar el panel con más facilidad, lo hemos trasladado a la nueva pestaña Conclusiones e informes del menú Acceso condicional de Azure AD.

---

### <a name="policy-details-blade-for-conditional-access-is-in-public-preview"></a>La hoja Detalles de la directiva de Acceso condicional está en versión preliminar pública.

**Tipo:** Nueva característica  
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad

La nueva [hoja Detalles de la directiva](https://docs.microsoft.com/azure/active-directory/conditional-access/troubleshoot-conditional-access) muestra las asignaciones, condiciones y controles que se han llevado a cabo durante la evaluación de la directiva de acceso condicional. Para acceder a la hoja, seleccione una fila en las pestañas Acceso condicional o Solo informe, en los detalles de inicio de sesión.

---

### <a name="new-query-capabilities-for-directory-objects-in-microsoft-graph-are-in-public-preview"></a>Nuevas funciones de consulta para objetos de directorio en Microsoft Graph ahora disponibles en versión preliminar pública

**Tipo:** Nueva característica  
**Categoría del servicio:** MS Graph **Funcionalidad del producto:** Experiencia para el desarrollador

Se están incorporando nuevas funciones a las API de objetos de directorio de Microsoft Graph, que permiten realizar operaciones de recuento, búsqueda, filtrado y ordenación. Esto permitirá a los desarrolladores consultar rápidamente nuestros objetos de directorio sin necesidad de buscar alternativas como la ordenación y el filtrado en memoria. Puede obtener más información en esta [entrada de blog](https://aka.ms/CountFilterMSGraphAAD).

Actualmente nos encontramos en la fase de versión preliminar pública, a la espera de comentarios. Envíe sus comentarios por medio de esta [breve encuesta](https://aka.ms/MsGraphAADSurveyDocs).

---

### <a name="configure-saml-based-single-sign-on-using-microsoft-graph-api-beta"></a>Configuración del inicio de sesión único basado en SAML mediante Microsoft Graph API (beta)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** SSO
 
Ya se ofrece compatibilidad para crear y configurar una aplicación desde la galería de Azure AD mediante MS Graph API en la versión beta. Si necesita configurar el inicio de sesión único basado en SAML para varias instancias de una aplicación, ahorre tiempo y utilice Microsoft Graph API para [automatizar la configuración del inicio de sesión único basado en SAML](https://docs.microsoft.com/azure/active-directory/manage-apps/application-saml-sso-configure-api).
 
---

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---may-2020"></a>Nuevos conectores de aprovisionamiento disponibles en la galería de aplicaciones de Azure AD, mayo de 2020

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Integración de terceros
 
Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

* [8x8](https://docs.microsoft.com/azure/active-directory/saas-apps/8x8-provisioning-tutorial)
* [Juno Journey](https://docs.microsoft.com/azure/active-directory/saas-apps/juno-journey-provisioning-tutorial)
* [MediusFlow](https://docs.microsoft.com/azure/active-directory/saas-apps/mediusflow-provisioning-tutorial)
* [New Relic by Organization](https://docs.microsoft.com/azure/active-directory/saas-apps/new-relic-by-organization-provisioning-tutorial)
* [Oracle Cloud Infrastructure Console](https://docs.microsoft.com/azure/active-directory/saas-apps/oracle-cloud-infratstructure-console-provisioning-tutorial)

Para más información acerca de cómo proteger mejor una organización mediante el aprovisionamiento automatizado de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).

---

### <a name="saml-token-encryption-is-generally-available"></a>El cifrado de tokens SAML está disponible con carácter general

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** SSO
 
El [cifrado de tokens SAML](https://docs.microsoft.com/azure/active-directory/manage-apps/howto-saml-token-encryption) permite configurar las aplicaciones para que reciban aserciones de SAML cifradas. La característica ya está disponible con carácter general en todas las nubes.
 
---

### <a name="group-name-claims-in-application-tokens-is-generally-available"></a>Las notificaciones de nombre de grupo en tokens de aplicación están disponibles con carácter general

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** SSO
 
Las notificaciones de grupo emitidas en un token ahora se pueden limitar a los grupos asignados a la aplicación.  Esto es especialmente importante cuando los usuarios son miembros de un gran número de grupos y había riesgo de superar los límites de tamaño del token. Esta nueva funcionalidad permite [agregar nombres de grupo a los tokens](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-fed-group-claims) con carácter general.
 
---

### <a name="workday-writeback-now-supports-setting-work-phone-number-attributes"></a>La escritura diferida de Workday ahora admite el establecimiento de atributos relativos al número de teléfono del trabajo

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
Hemos mejorado la aplicación de aprovisionamiento de escritura diferida de Workday para que admita la escritura diferida de los atributos relativos al número de teléfono del trabajo y al número de móvil. Además del correo electrónico y el nombre de usuario, ahora puede configurar la aplicación de aprovisionamiento de escritura diferida de Workday para transferir los valores de número de teléfono de Azure AD a Workday. Para más información sobre cómo configurar la escritura diferida de números de teléfono, consulte el tutorial de la aplicación de [escritura diferida de Workday](https://aka.ms/WorkdayWriteback). 

---

### <a name="publisher-verification-preview"></a>Comprobación del publicador (versión preliminar)

**Tipo:** Nueva característica  
**Categoría del servicio:** Otros  
**Funcionalidad del producto:** Experiencia para el desarrollador
 
La comprobación del publicador (versión preliminar) ayuda a los administradores y usuarios finales a reconocer la autenticidad de los desarrolladores de aplicaciones que se integran en la plataforma de identidad de Microsoft. Para más información, consulte [Comprobación del publicador (versión preliminar)](https://docs.microsoft.com/azure/active-directory/develop/publisher-verification-overview).
 
---

### <a name="authorization-code-flow-for-single-page-apps"></a>Flujo de código de autorización para aplicaciones de una sola página

**Tipo:** Característica modificada **Categoría del servicio:** Autenticación **Funcionalidad del producto:** Experiencia para el desarrollador

Debido a las [restricciones de cookies de terceros como Safari ITP](https://docs.microsoft.com/azure/active-directory/develop/reference-third-party-cookies-spas) aplicadas en el explorador, las aplicaciones de página única (SPA) tendrán que usar el flujo de código de autorización en lugar del flujo implícito para mantener el SSO. MSAL.js v2.x admite ahora el flujo de código de autorización. Las actualizaciones correspondientes están disponibles en Azure Portal. De este modo, podrá actualizar sus aplicaciones de página única para que sean del tipo "spa" y utilicen el flujo de código de autenticación. Para más instrucciones, consulte [Inicio rápido: Inicio de sesión de los usuarios y obtención de un token de acceso en una SPA de JavaScript mediante el flujo de código de autorización](https://docs.microsoft.com/azure/active-directory/develop/quickstart-v2-javascript-auth-code).

---

### <a name="improved-filtering-for-devices-is-in-public-preview"></a>El filtrado mejorado para dispositivos está en versión preliminar pública

**Tipo:** Característica modificada   
**Categoría del servicio:** Administración de dispositivos **Funcionalidad del producto:** Administración del ciclo de vida de dispositivos
 
Antes, los únicos filtros que se podían usar eran "Habilitado" y "Fecha de actividad". Ahora, puede [filtrar la lista de dispositivos en función de más propiedades](https://docs.microsoft.com/azure/active-directory/devices/device-management-azure-portal#device-list-filtering-preview), como el tipo de sistema operativo, el tipo de combinación, el cumplimiento, etc. Estas incorporaciones permitirán encontrar fácilmente un dispositivo determinado.

---

### <a name="the-new-app-registrations-experience-for-azure-ad-b2c-is-now-generally-available"></a>La nueva experiencia Registros de aplicaciones para Azure AD B2C ya está disponible con carácter general

**Tipo:** Característica modificada   
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
La nueva experiencia Registros de aplicaciones para Azure AD B2C ya está disponible con carácter general. 

Antes, había que administrar las aplicaciones Azure AD B2C orientadas al consumidor con independencia del resto de aplicaciones mediante la experiencia "Aplicaciones" heredada. Eso implicaba distintas experiencias de creación de aplicaciones en distintas ubicaciones de Azure.

La nueva experiencia muestra todos los registros de aplicaciones de Azure AD B2C y Azure AD en un solo lugar y proporciona una manera coherente de administrarlos. Si necesita administrar una aplicación orientada al cliente o una aplicación que tenga acceso a Microsoft Graph para administrar recursos de Azure AD B2C mediante programación, solo tendrá que aprender una manera de hacer cosas.

Puede acceder a la nueva experiencia desde el servicio Azure AD B2C. Una vez ahí, seleccione la hoja Registros de aplicaciones. También puede acceder a la experiencia desde el servicio de Azure Active Directory.

La experiencia Registros de aplicaciones de Azure AD B2C se basa en la [experiencia de registro de aplicaciones](https://developer.microsoft.com/identity/blogs/new-app-registrations-experience-is-now-generally-available/) general para inquilinos de Azure AD, pero se ha adaptado para Azure AD B2C. La experiencia "Aplicaciones" heredada dejará de utilizarse en el futuro.

Para más información, visite [ La nueva experiencia Registros de aplicaciones para Azure Active Directory B2C ](https://aka.ms/b2cappregtraining).

---

## <a name="april-2020"></a>Abril de 2020

### <a name="combined-security-info-registration-experience-is-now-generally-available"></a>La experiencia combinada del registro de la información de seguridad ya está disponible con carácter general.

**Tipo:** Nueva característica

**Categoría del servicio:** Autenticaciones (inicios de sesión)

**Funcionalidad del producto:** Seguridad y protección de la identidad

La experiencia de registro combinada con autenticación multifactor (MFA) y autoservicio de restablecimiento de contraseña (SSPR) ahora están disponibles con carácter general. Esta nueva experiencia de registro permite a los usuarios registrarse en MFA y SSPR en un único proceso paso a paso. Al implementar la nueva experiencia para la organización, los usuarios pueden registrarse en menos tiempo y con menos complicaciones. Consulte la entrada de blog [aquí](https://bit.ly/3etiRyQ).

---

### <a name="continuous-access-evaluation"></a>Evaluación continua de acceso

**Tipo:** Nueva característica

**Categoría del servicio:** Autenticaciones (inicios de sesión)

**Funcionalidad del producto:** Seguridad y protección de la identidad

Evaluación continua de acceso es una nueva característica de seguridad que permite el cumplimiento casi en tiempo real de las directivas en los usuarios de confianza que consumen tokens de acceso de Azure AD cuando se producen eventos en Azure AD (por ejemplo, la eliminación de cuentas de usuario). Esta característica la vamos a implementar en primer lugar para clientes de Teams y Outlook. Para más información, consulte el [blog](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/moving-towards-real-time-policy-and-security-enforcement/ba-p/1276933) y la [documentación](https://docs.microsoft.com/azure/active-directory/fundamentals/concept-fundamentals-continuous-access-evaluation).

---

### <a name="sms-sign-in-firstline-workers-can-sign-in-to-azure-ad-backed-applications-with-their-phone-number-and-no-password"></a>Inicio de sesión por SMS: Los trabajadores de primera línea pueden iniciar sesión en las aplicaciones respaldadas por Azure AD con su número de teléfono y sin contraseña.

**Tipo:** Nueva característica

**Categoría del servicio:** Autenticaciones (inicios de sesión)

**Funcionalidad del producto:** Autenticación de usuarios

Office está lanzando una serie de aplicaciones empresariales orientadas a móviles dirigidas a organizaciones no tradicionales y a empleados de grandes organizaciones que no usan el correo electrónico como método de comunicación principal. Estas aplicaciones se dirigen a empleados de primera línea, trabajadores sin escritorio, agentes de campo o empleados de comercios minoristas que podrían no recibir una dirección de correo electrónico de su empleador, tener acceso a un equipo o a recursos de TI. Este proyecto permitirá a estos empleados iniciar sesión en las aplicaciones empresariales solo con un número de teléfono y la devolución de un código. Para más información, consulte la [documentación del administrador](https://docs.microsoft.com/azure/active-directory/authentication/howto-authentication-sms-signin) y la [documentación del usuario final](https://docs.microsoft.com/azure/active-directory/user-help/sms-sign-in-explainer).

---

### <a name="invite-internal-users-to-use-b2b-collaboration"></a>Invitar a usuarios internos a usar la colaboración B2B

**Tipo:** Nueva característica

**Categoría del servicio:** B2B

**Funcionalidad del producto:**

Vamos a expandir la funcionalidad de invitación B2B para que se pueda invitar a las cuentas internas existentes a usar las credenciales de colaboración B2B en adelante. Esto se consigue pasando el objeto de usuario a la API de invitación, además de los parámetros típicos, como la dirección de correo electrónico invitada. El identificador de objeto del usuario, su UPN, la pertenencia a grupos, la asignación de aplicaciones, entre otros, permanecen intactos, pero en adelante usarán B2B para autenticarse con las credenciales de su inquilino particular en lugar de con las credenciales internas que usaban antes de la invitación. Para más información, consulte la [documentación](https://docs.microsoft.com/azure/active-directory/b2b/invite-internal-users).

---

### <a name="report-only-mode-for-conditional-access-is-now-generally-available"></a>El modo Solo informe para el Acceso condicional ya está disponible con carácter general.

**Tipo:** Nueva característica

**Categoría del servicio:** Acceso condicional

**Funcionalidad del producto:** Seguridad y protección de la identidad

El [modo Solo informe para el Acceso condicional de Azure AD](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-conditional-access-report-only) le permite evaluar el resultado de una directiva sin necesidad de aplicar controles de acceso. Puede probar las directivas del modo Solo informe en toda la organización y comprender su impacto antes de habilitarlas, lo que hará que la implementación sea más segura y fácil. Durante los últimos meses, hemos visto una importante adopción del modo Solo informe, con más de 26 millones de usuarios. Con este anuncio, se crearán nuevas directivas de acceso condicional de Azure AD en modo Solo informe de forma predeterminada. Esto significa que puede supervisar el impacto de las directivas desde el momento en el que se crean. Además, para aquellos usuarios que utilizan Microsoft Graph API, también puede [administrar directivas de Solo informe mediante programación](https://docs.microsoft.com/graph/api/resources/conditionalaccesspolicy?view=graph-rest-beta). 

---

### <a name="conditional-access-insights-and-reporting-workbook-is-generally-available"></a>El libro Conclusiones e informes de Acceso condicional está disponible con carácter general.

**Tipo:** Nueva característica

**Categoría del servicio:** Acceso condicional

**Funcionalidad del producto:** Seguridad y protección de la identidad

El [libro Conclusiones e informes](https://docs.microsoft.com/azure/active-directory/conditional-access/howto-conditional-access-insights-reporting) de Acceso condicional proporciona a los administradores una vista resumida del acceso condicional de Azure AD en el inquilino. Con la funcionalidad de seleccionar una directiva individual, los administradores pueden comprender mejor lo que hace cada directiva y supervisar los cambios en tiempo real. El libro transmite los datos almacenados en Azure Monitor, que puede configurar en unos minutos [según estas instrucciones](https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics). Para encontrar el panel con más facilidad, lo hemos trasladado a la nueva pestaña Conclusiones e informes del menú Acceso condicional de Azure AD.

---

### <a name="policy-details-blade-for-conditional-access-is-in-public-preview"></a>La hoja Detalles de la directiva de Acceso condicional está en versión preliminar pública.

**Tipo:** Nueva característica

**Categoría del servicio:** Acceso condicional

**Funcionalidad del producto:** Seguridad y protección de la identidad

La nueva [hoja Detalles de la directiva](https://docs.microsoft.com/azure/active-directory/conditional-access/troubleshoot-conditional-access) muestra qué asignaciones, condiciones y controles se cumplieron durante la evaluación de la directiva de acceso condicional. Para acceder a la hoja, seleccione una fila en las pestañas **Acceso condicional** o **Solo informe** de los detalles de inicio de sesión.

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---april-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (abril de 2020)

**Tipo:** Nueva característica

**Categoría del servicio:** Aplicaciones empresariales

**Funcionalidad del producto:** Integración de terceros

En abril de 2020, hemos agregado a la galería de aplicaciones estas 31 nuevas aplicaciones que admiten federación: 

[SincroPool Apps](https://www.sincropool.com/), [SmartDB](https://hibiki.dreamarts.co.jp/smartdb/trial/), [Float](https://docs.microsoft.com/azure/active-directory/saas-apps/float-tutorial), [LMS365](https://lms.365.systems/), [IWT Procurement Suite](https://docs.microsoft.com/azure/active-directory/saas-apps/iwt-procurement-suite-tutorial), [Lunni](https://lunni.fi/), [EasySSO for Jira](https://docs.microsoft.com/azure/active-directory/saas-apps/easysso-for-jira-tutorial), [Virtual Training Academy](https://vta.c3p.ca/app/en/openid?authenticate_with=microsoft), [Meraki Dashboard](https://docs.microsoft.com/azure/active-directory/saas-apps/meraki-dashboard-tutorial), [Office 365 Mover](https://app.mover.io/login), [Speaker Engage](https://speakerengage.com/login.php), [Honestly](https://docs.microsoft.com/azure/active-directory/saas-apps/honestly-tutorial), [Ally](https://docs.microsoft.com/azure/active-directory/saas-apps/ally-tutorial), [DutyFlow](https://app.dutyflow.nl/), [AlertMedia](https://docs.microsoft.com/azure/active-directory/saas-apps/alertmedia-tutorial), [gr8 People](https://docs.microsoft.com/azure/active-directory/saas-apps/gr8-people-tutorial), [Pendo](https://docs.microsoft.com/azure/active-directory/saas-apps/pendo-tutorial), [HighGround](https://docs.microsoft.com/azure/active-directory/saas-apps/highground-tutorial), [Harmony](https://docs.microsoft.com/azure/active-directory/saas-apps/harmony-tutorial), [Timetabling Solutions](https://docs.microsoft.com/azure/active-directory/saas-apps/timetabling-solutions-tutorial), [SynchroNet CLICK](https://docs.microsoft.com/azure/active-directory/saas-apps/synchronet-click-tutorial), [empower](https://www.made-in-office.com/en/), [Fortes Change Cloud](https://docs.microsoft.com/azure/active-directory/saas-apps/fortes-change-cloud-tutorial), [Litmus](https://docs.microsoft.com/azure/active-directory/saas-apps/litmus-tutorial), [GroupTalk](https://recorder.grouptalk.com/), [Frontify](https://docs.microsoft.com/azure/active-directory/saas-apps/frontify-tutorial), [MongoDB Cloud](https://docs.microsoft.com/azure/active-directory/saas-apps/mongodb-cloud-tutorial), [TickitLMS Learn](https://docs.microsoft.com/azure/active-directory/saas-apps/tickitlms-learn-tutorial), [COCO](https://hexaware.com/partnerships-and-alliances/digital-transformation-using-microsoft-azure/), [Nitro Productivity Suite](https://docs.microsoft.com/azure/active-directory/saas-apps/nitro-productivity-suite-tutorial), [Trend Micro Web Security(TMWS)](https://review.docs.microsoft.com/azure/active-directory/saas-apps/trend-micro-tutorial)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="microsoft-graph-delta-query-support-for-oauth2permissiongrant-available-for-public-preview"></a>La compatibilidad con las consultas delta de Microsoft Graph para oAuth2PermissionGrant está disponible para la versión preliminar pública.

**Tipo:** Nueva característica

**Categoría del servicio:** MS Graph

**Funcionalidad del producto:** Experiencia para el desarrollador

Las consultas delta para oAuth2PermissionGrant están disponibles en versión preliminar pública. Ahora puede realizar el seguimiento de los cambios sin necesidad de sondear Microsoft Graph continuamente. [Más información.](https://docs.microsoft.com/graph/api/oAuth2PermissionGrant-delta?view=graph-rest-beta&tabs=http)

---

### <a name="microsoft-graph-delta-query-support-for-organizational-contact-generally-available"></a>La compatibilidad con las consultas delta de Microsoft Graph para contactos de la organización está disponible con carácter general.

**Tipo:** Nueva característica

**Categoría del servicio:** MS Graph

**Funcionalidad del producto:** Experiencia para el desarrollador

Las consultas delta para contactos de la organización están disponibles con carácter general. Ahora puede realizar el seguimiento de los cambios en las aplicaciones de producción sin necesidad de sondear Microsoft Graph continuamente. Reemplace el código existente que ahora sondea continuamente los datos de orgContact por una consulta delta para mejorar significativamente el rendimiento. [Más información.](https://docs.microsoft.com/graph/api/orgcontact-delta?view=graph-rest-1.0&tabs=http)

---

### <a name="microsoft-graph-delta-query-support-for-application-generally-available"></a>La compatibilidad con las consultas delta de Microsoft Graph para aplicaciones está disponible con carácter general.

**Tipo:** Nueva característica

**Categoría del servicio:** MS Graph

**Funcionalidad del producto:** Experiencia para el desarrollador

Las consultas delta para aplicaciones están disponibles con carácter general. Ahora puede realizar el seguimiento de los cambios en las aplicaciones de producción sin necesidad de sondear Microsoft Graph continuamente. Reemplace el código existente que ahora sondea continuamente los datos de la aplicación por una consulta delta para mejorar significativamente el rendimiento. [Más información.](https://docs.microsoft.com/graph/api/application-delta?view=graph-rest-1.0)

---

### <a name="microsoft-graph-delta-query-support-for-administrative-units-available-for-public-preview"></a>La compatibilidad con las consultas delta de Microsoft Graph para unidades administrativas está disponible en versión preliminar pública.

**Tipo:** Nueva característica

**Categoría del servicio:** MS Graph

**Funcionalidad del producto:** Las consultas delta de la experiencia del desarrollador para unidades administrativas están disponibles en versión preliminar pública. Ahora puede realizar el seguimiento de los cambios sin necesidad de sondear Microsoft Graph continuamente. [Más información.](https://docs.microsoft.com/graph/api/administrativeunit-delta?view=graph-rest-beta&tabs=http)

---

### <a name="manage-authentication-phone-numbers-and-more-in-new-microsoft-graph-beta-apis"></a>Administre los números de teléfono de autenticación y mucho más en las nuevas Microsoft Graph API en versión beta.

**Tipo:** Nueva característica

**Categoría del servicio:** MS Graph

**Funcionalidad del producto:** Experiencia para el desarrollador

Estas API son una herramienta fundamental para administrar los métodos de autenticación de los usuarios. Ahora puede realizar un registro previo y administrar mediante programación los autenticadores usados para MFA y el autoservicio de restablecimiento de contraseña (SSPR). Esta ha sido una de las características más solicitadas en las áreas de Azure MFA, SSPR y Microsoft Graph. Las nuevas API que hemos publicado en esta oleada le ofrecen la posibilidad de:

- Leer, agregar, actualizar y eliminar los teléfonos de autenticación de un usuario
- Restablecer la contraseña de un usuario
- Activar y desactivar el inicio de sesión por SMS

Para más información, consulte [Introducción a la API de métodos de autenticación de Azure AD](https://docs.microsoft.com/graph/api/resources/authenticationmethods-overview?view=graph-rest-beta).

---

### <a name="administrative-units-public-preview"></a>Versión preliminar de las unidades administrativas

**Tipo:** Nueva característica

**Categoría del servicio:** RBAC

**Funcionalidad del producto:** Control de acceso

Las unidades administrativas permiten conceder permisos de administrador que están restringidos a un departamento, a una región o a otro segmento de la organización que se defina. Puede usar las unidades administrativas para delegar permisos en los administradores regionales o establecer directivas de forma pormenorizada. Por ejemplo, un administrador de cuentas de usuario podría actualizar la información del perfil, restablecer las contraseñas y asignar las licencias para los usuarios solo en su unidad administrativa.

Mediante el uso de unidades administrativas, un administrador central podría:

- Crear una unidad administrativa para la administración descentralizada de los recursos.
- Asignar un rol con permisos administrativos solo sobre los usuarios de Azure AD de una unidad administrativa.
- Rellenar las unidades administrativas con usuarios y grupos según sea necesario.

Para más información, consulte [Administración de unidades administrativas en Azure Active Directory (versión preliminar)](https://aka.ms/AdminUnitsDocs).

---

### <a name="printer-administrator-and-printer-technician-built-in-roles"></a>Roles integrados Administrador de impresoras y Técnico de impresoras

**Tipo:** Nueva característica

**Categoría del servicio:** RBAC

**Funcionalidad del producto:** Control de acceso

**Administrador de impresoras**: los usuarios con este rol pueden registrar impresoras y administrar todos los aspectos de todas las configuraciones de impresora en la solución de impresión universal de Microsoft, incluida la configuración del conector de impresión universal. Pueden dar su consentimiento a todas las solicitudes de permiso de impresión delegada. Los administradores de impresoras también tienen acceso a los informes de impresión. 

**Técnico de impresoras**: los usuarios con este rol pueden registrar impresoras y administrar el estado de la impresora en la solución de impresión universal de Microsoft. También pueden leer toda la información del conector. Una tarea clave que un Técnico de impresoras no puede realizar es establecer permisos de usuario en las impresoras ni compartir impresoras. [Más información.](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles#printer-administrator)

---

### <a name="hybrid-identity-admin-built-in-role"></a>Rol integrado Administrador de identidades híbridas

**Tipo:** Nueva característica

**Categoría del servicio:** RBAC

**Funcionalidad del producto:** Control de acceso

Los usuarios de este rol pueden habilitar, configurar y administrar servicios y configuraciones relacionados con la habilitación de la identidad híbrida en Azure AD. Este rol concede la capacidad de configurar Azure AD en uno de los tres métodos de autenticación admitidos: la sincronización de hash de contraseña (PHS), la autenticación de paso a través (PTA) o la federación (AD FS o proveedor de federación de terceros), así como implementar la infraestructura local relacionada para habilitarlos. La infraestructura local incluye agentes de aprovisionamiento y PTA. Este rol concede la capacidad de habilitar el inicio de sesión único de conexión directa (S-SSO) para habilitar la autenticación directa en dispositivos que no son de Windows 10 o equipos que no son de Windows Server 2016. Además, este rol concede la capacidad de ver los registros de inicio de sesión y el acceso a los datos de mantenimiento y análisis para la supervisión y la solución de problemas. [Más información.](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles#hybrid-identity-administrator)

---

### <a name="network-administrator-built-in-role"></a>Rol integrado Administrador de red

**Tipo:** Nueva característica

**Categoría del servicio:** RBAC

**Funcionalidad del producto:** Control de acceso

Los usuarios con este rol pueden revisar las recomendaciones de la arquitectura de perímetro de red de Microsoft que se basan en la telemetría de red desde sus ubicaciones de usuario. El rendimiento de red para Office 365 se basa en una arquitectura de perímetro de red de cliente empresarial rigurosa que suele ser específica de la ubicación del usuario. Este rol permite la edición de ubicaciones de usuario detectadas y la configuración de parámetros de red para esas ubicaciones con el fin de facilitar la mejora de las medidas de telemetría y las recomendaciones de diseño. [Más información.](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles#network-administrator)

---

### <a name="bulk-activity-and-downloads-in-the-azure-ad-admin-portal-experience"></a>Actividad y descargas masivas en la experiencia del portal de administración de Azure AD

**Tipo:** Nueva característica

**Categoría del servicio:** User Management

**Funcionalidad del producto:** Directorio

Ahora puede realizar actividades masivas con usuarios y grupos de Azure AD mediante la carga de un archivo CSV en la experiencia del portal de administración de Azure AD. Puede crear usuarios o eliminar usuarios, e invitar a usuarios. También puede agregar y quitar miembros de un grupo.

Además, puede descargar listas de recursos de Azure AD desde la experiencia del portal de administración de Azure AD. Puede descargar la lista de usuarios del directorio, la lista de grupos del directorio y los miembros de un grupo determinado.

Para más información, consulte lo siguiente:

- [Crear usuarios](https://docs.microsoft.com/azure/active-directory/users-groups-roles/users-bulk-add) o [invitar a usuarios ](https://docs.microsoft.com/azure/active-directory/b2b/tutorial-bulk-invite).
- [Eliminar usuarios](https://docs.microsoft.com/azure/active-directory/users-groups-roles/users-bulk-delete) o [restaurar usuarios eliminados](https://docs.microsoft.com/azure/active-directory/users-groups-roles/users-bulk-restore).
- [Descargar la lista de usuarios](https://docs.microsoft.com/azure/active-directory/users-groups-roles/users-bulk-download) o [descargar la lista de grupos](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-bulk-download)
- [Agregar (importar) miembros](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-bulk-import-members), [eliminar miembros](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-bulk-remove-members) o [descargar la lista de miembros](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-bulk-download-members) de un grupo.

---

### <a name="my-staff-delegated-user-management"></a>Administración delegada de usuarios Mi personal

**Tipo:** Nueva característica

**Categoría del servicio:** User Management

**Funcionalidad del producto:**

Mi personal permite a los administradores de primera línea, como el administrador de una tienda, garantizar que los miembros del personal puedan acceder a sus cuentas de Azure AD. En lugar de depender de un departamento de soporte técnico central, las organizaciones pueden delegar tareas comunes, como el restablecimiento de contraseñas o el cambio de números de teléfono, en un administrador de primera línea. Con Mi personal, un usuario que no puede acceder a su cuenta puede recuperar el acceso con tan solo un par de clics, sin necesidad de ayuda del personal de TI o el departamento de soporte técnico. Para más información, consulte [Administración de usuarios con Mi personal (versión preliminar)](https://aka.ms/MyStaffAdminDocs) y [Delegación de la administración de usuarios con Mi personal (versión preliminar)](https://aka.ms/MyStaffUserDocs).

---

### <a name="an-upgraded-end-user-experience-in-access-reviews"></a>Una experiencia de usuario final actualizada en las revisiones de acceso

**Tipo:** Característica modificada

**Categoría del servicio:** Revisiones de acceso

**Funcionalidad del producto:** Identity Governance

Hemos actualizado la experiencia del revisor para las revisiones de acceso de Azure AD en el portal Aplicaciones. A finales de abril, los revisores que han iniciado sesión en la experiencia del revisor de las revisiones de acceso de Azure AD, verán un banner que les permitirá probar la experiencia actualizada en Mi acceso. Tenga en cuenta que la experiencia de revisiones de acceso actualizada ofrece la misma funcionalidad que la experiencia actual, pero con una interfaz de usuario mejorada sobre nuevas funcionalidades para permitir que los usuarios sean productivos. [Aquí encontrará más información sobre la experiencia actualizada](https://docs.microsoft.com/azure/active-directory/governance/perform-access-review). Esta versión preliminar pública durará hasta finales de julio de 2020. A finales de julio, los revisores que no hayan participado en la experiencia en versión preliminar serán redirigidos automáticamente a Mi acceso para realizar las revisiones de acceso. Si desea que los revisores cambien permanentemente a la experiencia en versión preliminar en Mi acceso ahora, [realice una solicitud aquí](https://forms.microsoft.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR5dv-S62099HtxdeKIcgO-NUOFJaRDFDWUpHRk8zQ1BWVU1MMTcyQ1FFUi4u).

---

### <a name="workday-inbound-user-provisioning-and-writeback-apps-now-support-the-latest-versions-of-workday-web-services-api"></a>Las aplicaciones de aprovisionamiento y escritura diferida de usuarios de entrada de WorkDay ahora admiten las versiones más recientes de la API Workday Web Services

**Tipo:** Característica modificada

**Categoría del servicio:** Aprovisionamiento de aplicaciones

**Funcionalidad del producto:** 

Hemos tenido en cuenta los comentarios de los clientes y hemos actualizado las aplicaciones de aprovisionamiento y escritura diferida de usuarios de entrada de WorkDay en la galería de aplicaciones empresariales para admitir las versiones más recientes de la API Workday Web Services (WWS). Con este cambio, los clientes pueden especificar la versión de la API WWS que desean usar en la cadena de conexión. Esto proporciona a los clientes la capacidad de recuperar más atributos de recursos humanos disponibles en las versiones de WorkDay. La aplicación de escritura diferida de WorkDay ahora usa el servicio web de WorkDay recomendado, Change_Work_Contact_Info, para superar las limitaciones de Maintain_Contact_Info.

De forma predeterminada, si no se especifica ninguna versión en la cadena de conexión, las aplicaciones de aprovisionamiento de entrada de WorkDay seguirán usando WWS v21.1. Para cambiar a las versiones más recientes de la API Workday Web Services para el aprovisionamiento de usuarios de entrada, los clientes deben actualizar la cadena de conexión tal y como se documenta [en el tutorial](https://docs.microsoft.com/azure/active-directory/saas-apps/workday-inbound-tutorial#which-workday-apis-does-the-solution-use-to-query-and-update-workday-worker-profiles), y también actualizar los valores de XPATH utilizados para los atributos de WorkDay, tal y como se describe en [Guía de referencia de atributos de Workday](https://docs.microsoft.com/azure/active-directory/app-provisioning/workday-attribute-reference#xpath-values-for-workday-web-services-wws-api-v30). 

Para usar la nueva API para la escritura diferida, no se requieren cambios en la aplicación de aprovisionamiento de escritura diferida de WorkDay. En WorkDay, asegúrese de que la cuenta de usuario del sistema de integración (ISU) de WorkDay tiene permisos para invocar el proceso empresarial Change_Work_Contact, tal y como se describe en la sección del tutorial [Configuración de los permisos de la directiva de seguridad de procesos empresariales](https://docs.microsoft.com/azure/active-directory/saas-apps/workday-inbound-tutorial#configuring-business-process-security-policy-permissions). 

Hemos actualizado la [guía de tutoriales](https://docs.microsoft.com/azure/active-directory/saas-apps/workday-inbound-tutorial) para reflejar la compatibilidad con la nueva versión de la API.

---

### <a name="users-with-default-access-role-are-now-in-scope-for-provisioning"></a>Los usuarios con el rol de acceso predeterminado ahora están dentro del ámbito del aprovisionamiento

**Tipo:** Característica modificada

**Categoría del servicio:** Aprovisionamiento de aplicaciones

**Funcionalidad del producto:** Administración del ciclo de vida de la identidad

Históricamente, los usuarios con el rol de acceso predeterminado han estado fuera del ámbito del aprovisionamiento. Hemos recibido comentarios acerca de que los clientes quieren que los usuarios con este rol estén dentro del ámbito del aprovisionamiento. A partir del 16 de abril de 2020, todas las nuevas configuraciones de aprovisionamiento permiten aprovisionar a los usuarios con el rol de acceso predeterminado. Cambiaremos gradualmente el comportamiento de las configuraciones de aprovisionamiento existentes para admitir el aprovisionamiento de usuarios con este rol. [Más información.](https://docs.microsoft.com/azure/active-directory/app-provisioning/application-provisioning-config-problem-no-users-provisioned)

---

### <a name="updated-provisioning-ui"></a>Actualización de la interfaz de usuario de aprovisionamiento

**Tipo:** Característica modificada

**Categoría del servicio:** Aprovisionamiento de aplicaciones

**Funcionalidad del producto:** Administración del ciclo de vida de la identidad

Hemos actualizado la experiencia de aprovisionamiento para crear una vista de administración más centrada. Cuando vaya a la hoja de aprovisionamiento de una aplicación empresarial que ya se ha configurado, podrá supervisar fácilmente el progreso del aprovisionamiento y administrar acciones como el inicio, la detención y el reinicio del aprovisionamiento. [Más información.](https://docs.microsoft.com/azure/active-directory/app-provisioning/configure-automatic-user-provisioning-portal)

---

### <a name="dynamic-group-rule-validation-is-now-available-for-public-preview"></a>La validación de reglas de grupos dinámicos ahora está disponible en versión preliminar pública

**Tipo:** Característica modificada

**Categoría del servicio:** Administración de grupos

**Funcionalidad del producto:** Colaboración

Azure Active Directory (Azure AD) ahora ofrece los medios para validar las reglas de grupos dinámicos. En la pestaña **Validación de las reglas**, puede validar la regla dinámica con los miembros del grupo de ejemplo, para confirmar que la regla funciona según lo previsto. Al crear o actualizar reglas de grupos dinámicos, los administradores quieren saber si un usuario o un dispositivo será miembro del grupo. Esto ayuda a evaluar si el usuario o el dispositivo cumplen los criterios de la regla y ayuda a solucionar problemas cuando no se espera la pertenencia.

Para más información, consulte [Validación de una regla de pertenencia dinámica a grupos (versión preliminar)](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-dynamic-rule-validation).

---

### <a name="identity-secure-score---security-defaults-and-mfa-improvement-action-updates"></a>Puntuación de seguridad de la identidad: valores predeterminados de seguridad y actualizaciones de las acciones de mejora de MFA

**Tipo:** Característica modificada

**Categoría del servicio:** N/D

**Funcionalidad del producto:** Seguridad y protección de la identidad

**Compatibilidad con los valores predeterminados de seguridad para las acciones de mejora de Azure AD:** La Puntuación de seguridad de Microsoft se actualizará con acciones de mejora para admitir [valores predeterminados de seguridad en Azure AD](https://docs.microsoft.com/azure/active-directory/fundamentals/concept-fundamentals-security-defaults), lo que facilita la protección de la organización con una configuración de seguridad preconfigurada para ataques comunes. Esto afectará a las siguientes acciones de mejora:

- Garantizar que todos los usuarios pueden completar la autenticación multifactor para el acceso seguro
- Requerir MFA para roles administrativos
- Habilitar la directiva para bloquear la autenticación heredada
 
**Actualizaciones de las acciones de mejora de MFA:** Para reflejar la necesidad de las empresas de garantizar la mayor seguridad al aplicar directivas que funcionen con su negocio, la Puntuación de seguridad de Microsoft ha eliminado tres acciones de mejora centradas en la autenticación multifactor y ha agregado otras dos.

Acciones de mejora eliminadas:

- Registrar a todos los usuarios para la autenticación multifactor
- Exigir autenticación multifactor para todos los usuarios
- Requerir MFA para roles con privilegios Azure AD

Acciones de mejora agregadas:

- Garantizar que todos los usuarios pueden completar la autenticación multifactor para el acceso seguro
- Requerir MFA para roles administrativos

Estas nuevas acciones de mejora requerirán el registro de usuarios o administradores para la autenticación multifactor (MFA) en todo el directorio y el establecimiento del conjunto adecuado de directivas que se adapten a las necesidades de la organización. El objetivo principal es tener flexibilidad, asegurando al mismo tiempo que todos los usuarios y administradores puedan autenticarse con varios factores o solicitudes de comprobación de identidad basadas en riesgos. Se puede hacer con varias directivas que apliquen decisiones de ámbito o estableciendo valores predeterminados de seguridad (a partir del 16 de marzo) que permitan a Microsoft decidir cuándo desafiar a los usuarios con MFA. [Más información sobre las novedades en la Puntuación de seguridad de Microsoft](https://docs.microsoft.com/microsoft-365/security/mtp/microsoft-secure-score?view=o365-worldwide#whats-new).

---

## <a name="march-2020"></a>Marzo de 2020

### <a name="unmanaged-azure-active-directory-accounts-in-b2b-update-for-march-2021"></a>Cuentas de Azure Active Directory no administradas en la actualización B2B de marzo de 2021

**Tipo:** Plan de cambio  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
**A partir del 31 de marzo de 2021**, Microsoft dejará de admitir el canje de invitaciones mediante la creación de cuentas de Azure Active Directory (Azure AD) no administradas e inquilinos para escenarios de colaboración B2B. Como preparación para esto, le recomendamos que opte por la [autenticación de código de acceso de un solo uso por correo electrónico](https://docs.microsoft.com/azure/active-directory/b2b/one-time-passcode).

---

### <a name="users-with-the-default-access-role-will-be-in-scope-for-provisioning"></a>Los usuarios con el rol de acceso predeterminado estarán dentro del ámbito del aprovisionamiento.

**Tipo:** Plan de cambio  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
Históricamente, los usuarios con el rol de acceso predeterminado han estado fuera del ámbito del aprovisionamiento. Hemos recibido comentarios acerca de que los clientes quieren que los usuarios con este rol estén dentro del ámbito del aprovisionamiento. Estamos trabajando en la implementación de un cambio para que todas las nuevas configuraciones de aprovisionamiento permitan aprovisionar a los usuarios con el rol de acceso predeterminado. Gradualmente, cambiaremos el comportamiento de las configuraciones de aprovisionamiento existentes para admitir el aprovisionamiento de usuarios con este rol. No se requiere ninguna acción del cliente. Publicaremos una actualización en la [documentación](https://docs.microsoft.com/azure/active-directory/app-provisioning/application-provisioning-config-problem-no-users-provisioned) una vez que este cambio esté en vigor.

---

### <a name="azure-ad-b2b-collaboration-will-be-available-in-microsoft-azure-operated-by-21vianet-azure-china-21vianet-tenants"></a>La colaboración B2B de Azure AD estará disponible en Microsoft Azure controlado por inquilinos de 21Vianet (Azure China 21Vianet).

**Tipo:** Plan de cambio  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
Las funcionalidades de colaboración B2B de Azure AD estarán disponibles en Microsoft Azure controlado por los inquilinos de 21Vianet (Azure China 21Vianet), lo que permite a los usuarios de un inquilino de Azure China 21Vianet colaborar sin problemas con los usuarios de otros inquilinos de Azure China 21Vianet. [Más información sobre la colaboración B2B de Azure AD](https://docs.microsoft.com/azure/active-directory/b2b/).

---
 
### <a name="azure-ad-b2b-collaboration-invitation-email-redesign"></a>Rediseño del correo electrónico de invitación de colaboración B2B de Azure AD

**Tipo:** Plan de cambio  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
Los [correos electrónicos](https://docs.microsoft.com/azure/active-directory/b2b/invitation-email-elements) enviados por el servicio de invitación de colaboración B2B de Azure AD para invitar a los usuarios al directorio se rediseñarán para que la información de la invitación y los siguientes pasos del usuario sean más claros.

---

### <a name="homerealmdiscovery-policy-changes-will-appear-in-the-audit-logs"></a>Los cambios de la directiva HomeRealmDiscovery aparecerán en los registros de auditoría

**Tipo:** Corregido  
**Categoría del servicio:** Auditoría  
**Funcionalidad del producto:** Supervisión e informes
 
Se ha corregido un error por el que los cambios en la [directiva HomeRealmDiscovery](https://docs.microsoft.com/azure/active-directory/manage-apps/configure-authentication-for-federated-users-portal) no se incluían en los registros de auditoría. Ahora podrá ver cuándo y cómo ha cambiado la directiva, y quién la cambió. 

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---march-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (marzo de 2020)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En marzo de 2020, hemos agregado estas 51 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones: 

[Cisco AnyConnect](https://docs.microsoft.com/azure/active-directory/saas-apps/cisco-anyconnect), [Zoho One China](https://docs.microsoft.com/azure/active-directory/saas-apps/zoho-one-china-tutorial), [PlusPlus](https://test.plusplus.app/auth/login/azuread-outlook/), [Profit.co SAML App](https://docs.microsoft.com/azure/active-directory/saas-apps/profitco-saml-app-tutorial), [iPoint Service Provider](https://docs.microsoft.com/azure/active-directory/saas-apps/ipoint-service-provider-tutorial), [contexxt.ai SPHERE](https://contexxt-sphere.com/login), [Wisdom By Invictus](https://docs.microsoft.com/azure/active-directory/saas-apps/wisdom-by-invictus-tutorial), [Flare Digital Signage](https://spark-dev.pixelnebula.com/login), [Logz.io - Cloud Observability for Engineers](https://docs.microsoft.com/azure/active-directory/saas-apps/logzio-cloud-observability-for-engineers-tutorial), [SpectrumU](https://docs.microsoft.com/azure/active-directory/saas-apps/spectrumu-tutorial), [BizzContact](https://bizzcontact.app/), [Elqano SSO](https://docs.microsoft.com/azure/active-directory/saas-apps/elqano-sso-tutorial), [MarketSignShare](http://www.signshare.com/), [CrossKnowledge Learning Suite](https://docs.microsoft.com/azure/active-directory/saas-apps/crossknowledge-learning-suite-tutorial), [Netvision Compas](https://docs.microsoft.com/azure/active-directory/saas-apps/netvision-compas-tutorial), [FCM HUB](https://docs.microsoft.com/azure/active-directory/saas-apps/fcm-hub-tutorial), [RIB A/S Byggeweb Mobile](https://apps.apple.com/us/app/docia/id529058757), [GoLinks](https://docs.microsoft.com/azure/active-directory/saas-apps/golinks-tutorial), [Datadog](https://docs.microsoft.com/azure/active-directory/saas-apps/datadog-tutorial), [Zscaler B2B User Portal](https://docs.microsoft.com/azure/active-directory/saas-apps/zscaler-b2b-user-portal-tutorial), [LIFT](https://docs.microsoft.com/azure/active-directory/saas-apps/lift-tutorial), [Planview Enterprise One](https://docs.microsoft.com/azure/active-directory/saas-apps/planview-enterprise-one-tutorial), [WatchTeams](https://www.devfinition.com/), [Aster](https://demo.asterapp.io/login), [Skills Workflow](https://docs.microsoft.com/azure/active-directory/saas-apps/skills-workflow-tutorial), [Node Insight](https://admin.nodeinsight.com/AADLogin.aspx), [IP Platform](https://docs.microsoft.com/azure/active-directory/saas-apps/ip-platform-tutorial), [InVision](https://docs.microsoft.com/azure/active-directory/saas-apps/invision-tutorial), [Pipedrive](https://docs.microsoft.com/azure/active-directory/saas-apps/pipedrive-tutorial), [Showcase Workshop](https://app.showcaseworkshop.com/), [Greenlight Integration Platform](https://docs.microsoft.com/azure/active-directory/saas-apps/greenlight-integration-platform-tutorial), [Greenlight Compliant Access Management](https://docs.microsoft.com/azure/active-directory/saas-apps/greenlight-compliant-access-management-tutorial), [Grok Learning](https://docs.microsoft.com/azure/active-directory/saas-apps/grok-learning-tutorial), [Miradore Online](https://login.online.miradore.com/), [Khoros Care](https://docs.microsoft.com/azure/active-directory/saas-apps/khoros-care-tutorial), [AskYourTeam](https://docs.microsoft.com/azure/active-directory/saas-apps/askyourteam-tutorial), [TruNarrative](https://docs.microsoft.com/azure/active-directory/saas-apps/trunarrative-tutorial), [Smartwaiver](https://www.smartwaiver.com/m/user/sw_login.php?wms_login), [Bizagi Studio for Digital Process Automation](https://docs.microsoft.com/azure/active-directory/saas-apps/bizagi-studio-for-digital-process-automation-tutorial), [insuiteX](https://www.insuite.jp/), [sybo](https://www.systexsoftware.com.tw/), [Britive](https://docs.microsoft.com/azure/active-directory/saas-apps/britive-tutorial), [WhosOffice](https://docs.microsoft.com/azure/active-directory/saas-apps/whosoffice-tutorial), [E-days](https://docs.microsoft.com/azure/active-directory/saas-apps/e-days-tutorial), [Kollective SDN](https://portal.kollective.app/login), [Witivio](https://app.witivio.com/), [Playvox](https://my.playvox.com/login), [Korn Ferry 360](https://docs.microsoft.com/azure/active-directory/saas-apps/korn-ferry-360-tutorial), [Campus Café](https://docs.microsoft.com/azure/active-directory/saas-apps/campus-cafe-tutorial), [Catchpoint](https://docs.microsoft.com/azure/active-directory/saas-apps/catchpoint-tutorial), [Code42](https://docs.microsoft.com/azure/active-directory/saas-apps/code42-tutorial)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="azure-ad-b2b-collaboration-available-in-azure-government-tenants"></a>Está disponible la colaboración B2B de Azure AD en inquilinos de Azure Government

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
Las características de colaboración B2B de Azure AD ahora están disponibles entre algunos inquilinos de Azure Government.  Para averiguar si su inquilino es capaz de usar estas funcionalidades, siga las instrucciones que se indican en [¿Cómo puedo saber si la colaboración B2B está disponible en mi inquilino de Azure US Government?](https://docs.microsoft.com/azure/active-directory/b2b/current-limitations#how-can-i-tell-if-b2b-collaboration-is-available-in-my-azure-us-government-tenant)

---

### <a name="azure-monitor-integration-for-azure-logs-is-now-available-in-azure-government"></a>La integración de Azure Monitor para los registros de Azure ahora está disponible en Azure Government

**Tipo:** Nueva característica  
**Categoría del servicio:** Notificación  
**Funcionalidad del producto:** Supervisión e informes
 
La integración de Azure Monitor en los registros de Azure AD ahora está disponible en Azure Government. Puede enrutar los registros de Azure AD (registros de auditoría e inicio de sesión) a una cuenta de almacenamiento, un centro de eventos y Log Analytics. Consulte la [documentación detallada](https://aka.ms/aadlogsinamd), así como los [planes de implementación para la elaboración de informes y la supervisión](https://docs.microsoft.com/azure/active-directory/reports-monitoring/plan-monitoring-and-reporting) para escenarios de Azure AD.

---

### <a name="identity-protection-refresh-in-azure-government"></a>Actualización de Identity Protection en Azure Government

**Tipo:** Nueva característica  
**Categoría del servicio:** Protección de identidad  
**Funcionalidad del producto:** Seguridad y protección de la identidad

Nos complace compartir que ahora hemos implementado la experiencia actualizada de [Azure AD Identity Protection](https://aka.ms/IdentityProtectionDocs) en el [portal de Microsoft Azure Government](https://portal.azure.us/). Para obtener más información, consulte nuestra [entrada de blog de anuncio](https://techcommunity.microsoft.com/t5/public-sector-blog/identity-protection-refresh-in-microsoft-azure-government/ba-p/1223667).

---

### <a name="disaster-recovery-download-and-store-your-provisioning-configuration"></a>Recuperación ante desastres: Descarga y almacenamiento de la configuración de aprovisionamiento

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
El servicio de aprovisionamiento de Azure AD proporciona un amplio conjunto de funcionalidades de configuración. Los clientes deben poder guardar su configuración para poder consultarla más adelante o revertir a una versión válida conocida. Hemos agregado la capacidad de descargar la configuración de aprovisionamiento como archivo JSON y cargarlo cuando sea necesario. [Más información](https://docs.microsoft.com/azure/active-directory/app-provisioning/export-import-provisioning-configuration).

---
 
### <a name="sspr-self-service-password-reset-now-requires-two-gates-for-admins-in-microsoft-azure-operated-by-21vianet-azure-china-21vianet"></a>SSPR (autoservicio de restablecimiento de contraseña) ahora requiere dos puertas para los administradores en Microsoft Azure controlado por 21Vianet (Azure China 21Vianet) 

**Tipo:** Característica modificada  
**Categoría del servicio:** Restablecimiento de contraseña de autoservicio  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
Anteriormente, en Microsoft Azure controlado por 21Vianet (Azure China 21Vianet), los administradores que usaban el autoservicio de restablecimiento de contraseña (SSPR) para restablecer sus propias contraseñas solo necesitaban una "puerta" (desafío) para demostrar su identidad. En las nubes públicas y en otras nubes nacionales, los administradores generalmente deben usar dos puertas para demostrar su identidad al usar SSPR. No obstante, dado que no se admiten las llamadas telefónicas ni SMS en Azure China 21Vianet, hemos permitido que los administradores restablezcan la contraseña con una sola puerta.

Estamos creando la paridad de características de SSPR entre Azure China 21Vianet y la nube pública. En el futuro, los administradores deberán usar dos puertas al usar SSPR. Se admitirán SMS, llamadas telefónicas, y códigos y notificaciones de aplicación de autenticador. [Más información](https://docs.microsoft.com/azure/active-directory/authentication/concept-sspr-policy#administrator-reset-policy-differences).

---

### <a name="password-length-is-limited-to-256-characters"></a>La longitud de las contraseñas está limitada a 256 caracteres

**Tipo:** Característica modificada  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Autenticación de usuarios
 
Para garantizar la confiabilidad del servicio de Azure AD, las contraseñas de usuario ahora tienen una longitud máxima de 256 caracteres. A los usuarios con contraseñas más largas que esta se les pedirá que cambien su contraseña en el inicio de sesión subsiguiente, ya sea poniéndose en contacto con su administrador o mediante la característica de autoservicio de restablecimiento de contraseña.

Este cambio se habilitó el 13 de marzo de 2020, a las 10:00 PST (18:00 UTC), y el error es AADSTS 50052, InvalidPasswordExceedsMaxLength. Para obtener más información, consulte el [aviso de cambio importante](https://docs.microsoft.com/azure/active-directory/develop/reference-breaking-changes#user-passwords-will-be-restricted-to-256-characters).

---

### <a name="azure-ad-sign-in-logs-are-now-available-for-all-free-tenants-through-the-azure-portal"></a>Los registros de inicio de sesión de Azure AD ahora están disponibles para todos los inquilinos gratuitos a través de Azure Portal

**Tipo:** Característica modificada  
**Categoría del servicio:** Notificación  
**Funcionalidad del producto:** Supervisión e informes
 
A partir de ahora, los clientes que dispongan de inquilinos gratuitos pueden tener acceso a los [registros de inicio de sesión de Azure AD desde Azure Portal](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-sign-ins) para un máximo de 7 días. Anteriormente, los registros de inicio de sesión solo estaban disponibles para los clientes con licencias de Azure Active Directory Premium. Con este cambio, todos los inquilinos pueden tener acceso a estos registros a través del portal.

> [!NOTE]
> Los clientes todavía necesitan una licencia Premium (Azure Active Directory Premium P1 o P2) para tener acceso a los registros de inicio de sesión a través de Microsoft Graph API y Azure Monitor.

---

### <a name="deprecation-of-directory-wide-groups-option-from-groups-general-settings-on-azure-portal"></a>Opción en desuso de grupos de todo el directorio de la configuración general de grupos en Azure Portal

**Tipo:** Obsoleto  
**Categoría del servicio:** Administración de grupos  
**Funcionalidad del producto:** Colaboración

Para ofrecer a los clientes una manera más flexible de crear grupos de todo el directorio que se adapte mejor a sus necesidades, hemos reemplazado la opción **Grupos de todo el directorio** de la opción **Grupos** > **General** en Azure Portal por un vínculo a la [documentación de grupos dinámicos](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-dynamic-membership). Hemos mejorado la documentación para incluir más instrucciones para que los administradores puedan crear grupos de todos los usuarios que incluyan o excluyan usuarios invitados.

---

## <a name="february-2020"></a>Febrero de 2020

### <a name="upcoming-changes-to-custom-controls"></a>Próximos cambios en los controles personalizados

**Tipo:** Plan de cambio  
**Categoría del servicio:** MFA  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
Tenemos previsto reemplazar la versión preliminar actual de los controles personalizados por un enfoque que permita que las funcionalidades de autenticación proporcionadas por el partner funcionen sin problemas con las experiencias de administrador y usuario final de Azure Active Directory. Hoy en día, las soluciones de MFA de los partners enfrentan las siguientes limitaciones: solo funcionan después de haber escrito una contraseña; no sirven como MFA para la autenticación de nivel superior en otros escenarios clave; y no se integran con las funciones de administración de credenciales administrativas ni de usuario final. La nueva implementación permitirá que los factores de autenticación proporcionados por los partners funcionen junto con factores integrados para escenarios clave, como el registro, el uso, las notificaciones de MFA, la autenticación de nivel superior, la creación de informes y el registro. 

Los controles personalizados se seguirán admitiendo en la versión preliminar junto con el nuevo diseño hasta que alcancen la disponibilidad general. En ese momento, daremos tiempo a los clientes para migrar al nuevo diseño. Debido a las limitaciones del enfoque actual, no se incorporarán nuevos proveedores hasta que el nuevo diseño esté disponible. Estamos trabajando en estrecha colaboración con los clientes y proveedores, y comunicaremos los plazos a medida que nos acerquemos. [Más información](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/upcoming-changes-to-custom-controls/ba-p/1144696#).

---

### <a name="identity-secure-score---mfa-improvement-action-updates"></a>Puntuación de seguridad de la identidad: actualizaciones de acciones de mejora de MFA

**Tipo:** Plan de cambio  
**Categoría del servicio:** MFA  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
Para reflejar la necesidad de las empresas de garantizar la mayor seguridad al aplicar directivas que funcionen con su negocio, la puntuación segura de Microsoft va a quitar tres acciones de mejora centradas en la autenticación multifactor (MFA) y va a añadir otras dos.

Se quitarán las siguientes acciones de mejora:

- Registrar todos los usuarios de MFA
- Exigir autenticación multifactor para todos los usuarios
- Requerir MFA para roles con privilegios Azure AD

Se agregarán las siguientes acciones de mejora:

- Asegurarse de que todos los usuarios pueden completar MFA para el acceso seguro
- Requerir MFA para roles administrativos

Estas nuevas acciones de mejora requerirán el registro de usuarios o administradores para MFA en todo el directorio y el establecimiento del conjunto adecuado de directivas que se adapten a las necesidades de la organización. El objetivo principal es tener flexibilidad, asegurando al mismo tiempo que todos los usuarios y administradores puedan autenticarse con varios factores o solicitudes de comprobación de identidad basadas en riesgos. Esto puede traducirse en la configuración de valores predeterminados de seguridad que permitan a Microsoft decidir cuándo desafiar a los usuarios para MFA o el mantenimiento de varias directivas que apliquen decisiones de ámbito. Como parte de estas actualizaciones de acciones de mejora, las directivas de protección de línea de base ya no se incluirán en los cálculos de puntuación. [Obtenga más información sobre lo próximo de la puntuación segura de Microsoft](https://docs.microsoft.com/microsoft-365/security/mtp/microsoft-secure-score-whats-coming?view=o365-worldwide).

---

### <a name="azure-ad-domain-services-sku-selection"></a>Selección de SKU Azure AD Domain Services

**Tipo:** Nueva característica  
**Categoría del servicio:** Azure AD Domain Services  
**Funcionalidad del producto:** Azure AD Domain Services
 
Hemos escuchado los comentarios de los clientes de Azure AD Domain Services que desean más flexibilidad a la hora de seleccionar los niveles de rendimiento de sus instancias. A partir del 1 de febrero de 2020, cambiamos de un modelo dinámico (donde Azure AD determina el rendimiento y el plan de tarifa en función del número de objetos) a un modelo de selección automática. Ahora los clientes pueden elegir un nivel de rendimiento que se adapte a su entorno. Este cambio también nos permite habilitar nuevos escenarios, como los bosques de recursos, y características Premium, como las copias de seguridad diarias. El número de objetos es ahora ilimitado para todas las SKU, pero seguiremos ofreciendo sugerencias de cantidades de objetos para cada nivel.

**No se requiere ninguna acción inmediata del cliente.** Para los clientes actuales, el nivel dinámico que estaba en uso el 1 de febrero de 2020 determina el nuevo nivel predeterminado. El resultado de este cambio no afecta al precio ni al rendimiento. A partir de entonces, los clientes de Azure AD DS deberán evaluar los requisitos de rendimiento a medida que cambien sus características de carga de trabajo y el tamaño del directorio. El cambio entre los niveles de servicio seguirá siendo una operación sin tiempo de inactividad, y ya no se moverá automáticamente a los clientes a nuevos niveles en función del crecimiento de su directorio. Además, no habrá ningún aumento del precio y los nuevos precios se alinearán con nuestro modelo de facturación actual. Para obtener más información, consulte la [documentación de las SKU de Azure AD DS](https://docs.microsoft.com/azure/active-directory-domain-services/administration-concepts#azure-ad-ds-skus) y la [página de precios de Azure AD Domain Services](https://azure.microsoft.com/pricing/details/active-directory-ds/).

---
 
### <a name="new-federated-apps-available-in-azure-ad-app-gallery---february-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (febrero de 2020)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En febrero de 2020, hemos agregado estas 31 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones: 

[IamIP Patent Platform](https://docs.microsoft.com/azure/active-directory/saas-apps/iamip-patent-platform-tutorial), [Experience Cloud](https://docs.microsoft.com/azure/active-directory/saas-apps/experience-cloud-tutorial), [NS1 SSO For Azure](https://docs.microsoft.com/azure/active-directory/saas-apps/ns1-sso-azure-tutorial), [Barracuda Email Security Service](https://ess.barracudanetworks.com/sso/azure), [ABa Reporting](https://myaba.co.uk/client-access/signin/auth/msad), [In Case of Crisis - Online Portal](https://docs.microsoft.com/azure/active-directory/saas-apps/in-case-of-crisis-online-portal-tutorial), [BIC Cloud Design](https://docs.microsoft.com/azure/active-directory/saas-apps/bic-cloud-design-tutorial), [Beekeeper Azure AD Data Connector](https://docs.microsoft.com/azure/active-directory/saas-apps/beekeeper-azure-ad-data-connector-tutorial), [Korn Ferry Assessments](https://www.kornferry.com/solutions/kf-digital/kf-assess), [Verkada Command](https://docs.microsoft.com/azure/active-directory/saas-apps/verkada-command-tutorial), [Splashtop](https://docs.microsoft.com/azure/active-directory/saas-apps/splashtop-tutorial), [Syxsense](https://docs.microsoft.com/azure/active-directory/saas-apps/syxsense-tutorial), [EAB Navigate](https://docs.microsoft.com/azure/active-directory/saas-apps/eab-navigate-tutorial), [New Relic (Limited Release)](https://docs.microsoft.com/azure/active-directory/saas-apps/new-relic-limited-release-tutorial), [Thulium](https://admin.thulium.com/login/instance), [Ticket Manager](https://docs.microsoft.com/azure/active-directory/saas-apps/ticketmanager-tutorial), [Template Chooser for Teams](https://links.officeatwork.com/templatechooser-download-teams), [Beesy](https://www.beesy.me/index.php/site/login), [Health Support System](https://docs.microsoft.com/azure/active-directory/saas-apps/health-support-system-tutorial), [MURAL](https://app.mural.co/signup), [Hive](https://docs.microsoft.com/azure/active-directory/saas-apps/hive-tutorial), [LavaDo](https://appsource.microsoft.com/product/web-apps/lavaloon.lavado_standard?tab=Overview), [Wakelet](https://wakelet.com/login), [Firmex VDR](https://docs.microsoft.com/azure/active-directory/saas-apps/firmex-vdr-tutorial), [ThingLink for Teachers and Schools](https://www.thinglink.com/), [Coda](https://docs.microsoft.com/azure/active-directory/saas-apps/coda-tutorial), [NearpodApp](https://nearpod.com/signup/?oc=Microsoft&utm_campaign=Microsoft&utm_medium=site&utm_source=product), [WEDO](https://docs.microsoft.com/azure/active-directory/saas-apps/wedo-tutorial), [InvitePeople](https://invitepeople.com/login), [Reprints Desk - Article Galaxy](https://docs.microsoft.com/azure/active-directory/saas-apps/reprints-desk-article-galaxy-tutorial), [TeamViewer](https://docs.microsoft.com/azure/active-directory/saas-apps/teamviewer-tutorial)

 
Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---
 
### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---february-2020"></a>Nuevos conectores de aprovisionamiento en la galería de aplicaciones de Azure AD (febrero de 2020)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

- [Mixpanel](https://docs.microsoft.com/azure/active-directory/saas-apps/mixpanel-provisioning-tutorial)
- [TeamViewer](https://docs.microsoft.com/azure/active-directory/saas-apps/teamviewer-provisioning-tutorial)
- [Azure Databricks](https://docs.microsoft.com/azure/active-directory/saas-apps/azure-databricks-scim-connector-provisioning-tutorial)
- [PureCloud by Genesys](https://docs.microsoft.com/azure/active-directory/saas-apps/purecloud-by-genesys-provisioning-tutorial)
- [Zapier](https://docs.microsoft.com/azure/active-directory/saas-apps/zapier-provisioning-tutorial)

Para más información acerca de cómo proteger mejor una organización mediante el aprovisionamiento automatizado de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).

---
 
### <a name="azure-ad-support-for-fido2-security-keys-in-hybrid-environments"></a>Compatibilidad de Azure AD con las claves de seguridad de FIDO2 en entornos híbridos

**Tipo:** Nueva característica  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Autenticación de usuarios
 
Anunciamos la versión preliminar pública de la compatibilidad de Azure AD con las claves de seguridad de FIDO2 en entornos híbridos. Los usuarios ahora pueden usar las claves de seguridad de FIDO2 para iniciar sesión en sus dispositivos de Windows 10 unidos a Azure AD híbrido y obtener un inicio de sesión fluido en sus recursos locales y en la nube. La compatibilidad con entornos híbridos ha sido la característica más solicitada de nuestros clientes sin contraseña, ya que inicialmente lanzamos la versión preliminar pública para la compatibilidad con FIDO2 en dispositivos unidos a Azure AD. La autenticación sin contraseña mediante tecnologías avanzadas como la biométrica y la criptografía de clave privada y pública es cómoda y fácil de usar a la vez que segura. Con esta versión preliminar pública, ahora puede usar la autenticación moderna como las claves de seguridad de FIDO2 para acceder a los recursos de Active Directory tradicionales. Para obtener más información, vaya a [Inicio de sesión único en recursos locales](https://docs.microsoft.com/azure/active-directory/authentication/howto-authentication-passwordless-security-key-on-premises). 

Para empezar, visite [Habilitación de las claves de seguridad FIDO2 para un inquilino](https://docs.microsoft.com/azure/active-directory/authentication/howto-authentication-passwordless-security-key) para obtener instrucciones paso a paso. 

---
 
### <a name="the-new-my-account-experience-is-now-generally-available"></a>La nueva experiencia de Mi cuenta ya está disponible con carácter general.

**Tipo:** Característica modificada  
**Categoría del servicio:** Mi perfil/cuenta  
**Funcionalidad del producto:** Experiencias de usuario final
 
Mi cuenta, el punto en el que se atienden todas las necesidades de administración de las cuentas de usuario final, ya está disponible con carácter general. Los usuarios finales pueden tener acceso a este nuevo sitio a través de la dirección URL o en el encabezado de la nueva experiencia Mis aplicaciones. Obtenga más información sobre todas las funcionalidades de autoservicio que ofrece la nueva experiencia en [¿Qué es el portal Mi cuenta?](https://docs.microsoft.com/azure/active-directory/user-help/my-account-portal-overview)

---
 
### <a name="my-account-site-url-updating-to-myaccountmicrosoftcom"></a>Actualización de la dirección URL del sitio de Mi cuenta a myaccount.microsoft.com

**Tipo:** Característica modificada  
**Categoría del servicio:** Mi perfil/cuenta  
**Funcionalidad del producto:** Experiencias de usuario final
 
La nueva experiencia de usuario final de Mi cuenta actualizará su dirección URL a `https://myaccount.microsoft.com` el próximo mes. Obtenga más información sobre la experiencia y todas las funcionalidades de autoservicio de cuenta que ofrece a los usuarios finales en la [ayuda del portal Mi cuenta](https://docs.microsoft.com/azure/active-directory/user-help/my-account-portal-overview).

