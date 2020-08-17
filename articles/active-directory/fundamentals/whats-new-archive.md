---
title: Archivo de ¿Cuáles son las novedades de Azure Active Directory? | Microsoft Docs
description: Las notas de la versión sobre novedades de la sección Introducción de este conjunto de contenidos incluye seis meses de actividad. Después de 6 meses, los elementos se quitan del artículo principal y se colocan en este artículo de archivo.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 06/30/2020
ms.author: ajburnle
ms.reviewer: dhanyahk
ms.custom: it-pro, seo-update-azuread-jan, has-adal-ref
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4fd7ce0229d5e26bc4bf3b05f7b97d20d2a23dff
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87927205"
---
# <a name="archive-for-whats-new-in-azure-active-directory"></a>Archivo de ¿Cuáles son las novedades de Azure Active Directory?

El artículo principal de notas de la versión [¿Cuáles son las novedades de Azure Active Directory?](whats-new.md) contiene las actualizaciones de los últimos seis meses, mientras que este artículo contiene toda la información más antigua.

Las notas de la versión ¿Cuáles son las novedades de Azure Active Directory? proporcionan información sobre:

- Versiones más recientes
- Problemas conocidos
- Corrección de errores
- Funciones obsoletas
- Planes de cambios

---
 
## <a name="january-2020"></a>Enero de 2020
 
### <a name="the-new-my-apps-portal-is-now-generally-available"></a>El nuevo portal Mis aplicaciones ya está disponible con carácter general

**Tipo:** Plan de cambio  
**Categoría del servicio:** Mis aplicaciones  
**Funcionalidad del producto:** Experiencias de usuario final
 
Actualice su organización al nuevo portal Mis aplicaciones que ya está disponible con carácter general. Puede encontrar más información sobre el nuevo portal y las colecciones en [Creación de colecciones en el portal Mis aplicaciones](https://docs.microsoft.com/azure/active-directory/manage-apps/access-panel-collections).

---
 
### <a name="workspaces-in-azure-ad-have-been-renamed-to-collections"></a>Las áreas de trabajo de Azure AD se han renombrado a colecciones

**Tipo:** Característica modificada  
**Categoría del servicio:** Mis aplicaciones   
**Funcionalidad del producto:** Experiencias de usuario final
 
Las áreas de trabajo, que son filtros que los administradores pueden configurar para organizar las aplicaciones de sus usuarios, ahora se llamarán colecciones. Obtenga más información sobre cómo configurarlas en [Creación de colecciones en el portal Mis aplicaciones](https://docs.microsoft.com/azure/active-directory/manage-apps/access-panel-collections).

---
 
### <a name="azure-ad-b2c-phone-sign-up-and-sign-in-using-custom-policy-public-preview"></a>Registro e inicio de sesión mediante teléfono en Azure AD B2C con una directiva personalizada (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C
 
Con el registro y el inicio de sesión mediante el número de teléfono, los desarrolladores y las empresas pueden permitir a sus clientes registrarse e iniciar sesión con una contraseña de un solo uso enviada al número de teléfono del usuario a través de un SMS. Esta característica también permite al cliente cambiar su número de teléfono si pierde el acceso a su dispositivo. Con la eficacia de las directivas personalizadas, el registro y el inicio de sesión mediante el teléfono permiten a los desarrolladores y a las empresas comunicar su marca a través de la personalización de la página. Aprenda cómo [configurar el registro y el inicio de sesión en el teléfono con directivas personalizadas en Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/phone-authentication).
 
---
 
### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---january-2020"></a>Nuevos conectores de aprovisionamiento en la galería de aplicaciones de Azure AD (enero de 2020)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

- [Promapp]( https://docs.microsoft.com/azure/active-directory/saas-apps/promapp-provisioning-tutorial)
- [Zscaler Private Access](https://docs.microsoft.com/azure/active-directory/saas-apps/zscaler-private-access-provisioning-tutorial)

Para más información acerca de cómo proteger mejor una organización mediante el aprovisionamiento automatizado de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).

---
 
### <a name="new-federated-apps-available-in-azure-ad-app-gallery---january-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (enero de 2020)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En enero de 2020, hemos agregado estas 33 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones: 

[JOSA](https://docs.microsoft.com/azure/active-directory/saas-apps/josa-tutorial), [Fastly Edge Cloud](https://docs.microsoft.com/azure/active-directory/saas-apps/fastly-edge-cloud-tutorial), [Terraform Enterprise](https://docs.microsoft.com/azure/active-directory/saas-apps/terraform-enterprise-tutorial), [Spintr SSO](https://docs.microsoft.com/azure/active-directory/saas-apps/spintr-sso-tutorial), [Abibot Netlogistik](https://azuremarketplace.microsoft.com/marketplace/apps/aad.abibotnetlogistik), [SkyKick](https://login.skykick.com/login?state=g6Fo2SBTd3M5Q0xBT0JMd3luS2JUTGlYN3pYTE1remJQZnR1c6N0aWTZIDhCSkwzYVQxX2ZMZjNUaWxNUHhCSXg2OHJzbllTcmYto2NpZNkgM0h6czk3ZlF6aFNJV1VNVWQzMmpHeFFDbDRIMkx5VEc&client=3Hzs97fQzhSIWUMUd32jGxQCl4H2LyTG&protocol=oauth2&audience=https://papi.skykick.com&response_type=code&redirect_uri=https://portal.skykick.com/callback&scope=openid%20profile%20offline_access), [Upshotly](https://docs.microsoft.com/azure/active-directory/saas-apps/upshotly-tutorial), [LeaveBot](https://leavebot.io/#home), [DataCamp](https://docs.microsoft.com/azure/active-directory/saas-apps/datacamp-tutorial), [TripActions](https://docs.microsoft.com/azure/active-directory/saas-apps/tripactions-tutorial), [SmartWork](https://www.intumit.com/english/SmartWork.html), [Dotcom-Monitor](https://docs.microsoft.com/azure/active-directory/saas-apps/dotcom-monitor-tutorial), [SSOGEN - Azure AD SSO Gateway for Oracle E-Business Suite - EBS, PeopleSoft y JDE](https://docs.microsoft.com/azure/active-directory/saas-apps/ssogen-tutorial), [Hosted MyCirqa SSO](https://docs.microsoft.com/azure/active-directory/saas-apps/hosted-mycirqa-sso-tutorial), [Yuhu Property Management Platform](https://docs.microsoft.com/azure/active-directory/saas-apps/yuhu-property-management-platform-tutorial), [LumApps](https://sites.lumapps.com/login), [Upwork Enterprise](https://docs.microsoft.com/azure/active-directory/saas-apps/upwork-enterprise-tutorial), [Talentsoft](https://docs.microsoft.com/azure/active-directory/saas-apps/talentsoft-tutorial), [SmartDB for Microsoft Teams](http://teams.smartdb.jp/login/), [PressPage](https://docs.microsoft.com/azure/active-directory/saas-apps/presspage-tutorial), [ContractSafe Saml2 SSO](https://docs.microsoft.com/azure/active-directory/saas-apps/contractsafe-saml2-sso-tutorial), [Maxient Conduct Manager Software](https://docs.microsoft.com/azure/active-directory/saas-apps/maxient-conduct-manager-software-tutorial), [Helpshift](https://docs.microsoft.com/azure/active-directory/saas-apps/helpshift-tutorial), [PortalTalk 365](https://www.portaltalk.com/), [CoreView](https://portal.coreview.com/), [Squelch Cloud Office365 Connector](https://laxmi.squelch.io/login), [PingFlow Authentication](https://app-staging.pingview.io/), [ PrinterLogic SaaS](https://docs.microsoft.com/azure/active-directory/saas-apps/printerlogic-saas-tutorial), [Taskize Connect](https://docs.microsoft.com/azure/active-directory/saas-apps/taskize-connect-tutorial), [Sandwai](https://app.sandwai.com/), [EZRentOut](https://docs.microsoft.com/azure/active-directory/saas-apps/ezrentout-tutorial), [AssetSonar](https://docs.microsoft.com/azure/active-directory/saas-apps/assetsonar-tutorial), [Akari Virtual Assistant](https://akari.io/akari-virtual-assistant/)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="two-new-identity-protection-detections"></a>Dos nuevas detecciones de Identity Protection

**Tipo:** Nueva característica  
**Categoría del servicio:** Protección de identidad  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
Hemos agregado a Identity Protection dos nuevos tipos de detección vinculados al inicio de sesión: Reglas de manipulación sospechosa de la bandeja de entrada y Viaje imposible. Microsoft Cloud App Security (MCAS) descubre estas detecciones sin conexión e influye en los riesgos de usuario y de inicio de sesión en Identity Protection. Para más información sobre estas detecciones, consulte nuestros [tipos de riesgo de inicio de sesión](https://docs.microsoft.com/azure/active-directory/identity-protection/concept-identity-protection-risks#sign-in-risk).
 
---
 
### <a name="breaking-change-uri-fragments-will-not-be-carried-through-the-login-redirect"></a>Cambio de última hora: los fragmentos de URI no se trasladarán a través de la redirección de inicio de sesión

**Tipo:** Característica modificada  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Autenticación de usuarios
 
<a name="starting-on-february-8-2020-when-a-request-is-sent-to-loginmicrosoftonlinecom-to-sign-in-a-user-the-service-will-append-an-empty-fragment-to-the-request--this-prevents-a-class-of-redirect-attacks-by-ensuring-that-the-browser-wipes-out-any-existing-fragment-in-the-request-no-application-should-have-a-dependency-on-this-behavior-for-more-information-see-breaking-changes-in-the-microsoft-identity-platform-documentation"></a>A partir del 8 de febrero de 2020, cuando se envíe una solicitud a login.microsoftonline.com para que un usuario inicie sesión, el servicio anexará un fragmento vacío a la solicitud.  Esto evita una clase de ataques de redireccionamiento, ya que se asegura de que el explorador borra todo fragmento existente en la solicitud. Ninguna aplicación debe tener una dependencia de este comportamiento. Para más información, consulte [Cambios importantes](https://docs.microsoft.com/azure/active-directory/develop/reference-breaking-changes#february-2020) en la documentación de la plataforma de identidad de Microsoft.
---

## <a name="december-2019"></a>Diciembre de 2019

### <a name="integrate-sap-successfactors-provisioning-into-azure-ad-and-on-premises-ad-public-preview"></a>Integración del aprovisionamiento de SAP SuccessFactors en Azure AD y AD local (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad

Ahora puede integrar SAP SuccessFactors como origen de identidad relevante en Azure AD. Esta integración le ayuda a automatizar el ciclo de vida de la identidad de un extremo a otro, incluido el uso de eventos basados en recursos humanos, como nuevas contrataciones o resoluciones de contrato, para controlar el aprovisionamiento de cuentas de Azure AD.

Para obtener más información sobre cómo configurar el aprovisionamiento de entrada de SAP SuccessFactors para Azure AD, consulte el tutorial sobre [configuración del aprovisionamiento automático de SAP SuccessFactors](https://aka.ms/SAPSuccessFactorsInboundTutorial).

---

### <a name="support-for-customized-emails-in-azure-ad-b2c-public-preview"></a>Compatibilidad con correos electrónicos personalizados en Azure AD B2C (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C

Ahora puede usar Azure AD B2C para crear correos electrónicos personalizados cuando los usuarios se suscriban para usar las aplicaciones. Mediante el uso de DisplayControls (actualmente en versión preliminar) y un proveedor de correo electrónico de terceros (por ejemplo, [SendGrid](https://sendgrid.com/), [SparkPost](https://sparkpost.com/) o una API REST personalizada), puede usar su propia plantilla de correo electrónico, dirección **De** y texto del asunto, además de admitir la localización y la configuración personalizada de contraseña de un solo uso (OTP).

Para obtener más información, consulte el artículo sobre [comprobación de correo electrónico personalizada en Azure Active Directory B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-email).

---

### <a name="replacement-of-baseline-policies-with-security-defaults"></a>Sustitución de directivas de base de referencia por valores predeterminados de seguridad

**Tipo:** Característica modificada  
**Categoría del servicio:** Otros  
**Funcionalidad del producto:** Seguridad y protección de la identidad

Como parte de un modelo predeterminado seguro para la autenticación, vamos a quitar las directivas de protección de base de referencia existentes de todos los inquilinos. Se prevé que esta retirada se complete a finales de febrero. El reemplazo de estas directivas de protección de base de referencia son los valores predeterminados de seguridad. Si ha usado directivas de protección de base de referencia, debe planear la migración a la nueva directiva de valores predeterminados de seguridad o al acceso condicional. Si no ha usado estas directivas, no tiene que realizar ninguna acción.

Para obtener más información sobre los nuevos valores predeterminados de seguridad, consulte [¿Qué son los valores predeterminados de seguridad?](https://docs.microsoft.com/azure/active-directory/fundamentals/concept-fundamentals-security-defaults) Para obtener más información sobre las directivas de acceso condicional, consulte [Directivas de acceso condicional habituales](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-conditional-access-policy-common).

---

## <a name="november-2019"></a>Noviembre de 2019

### <a name="support-for-the-samesite-attribute-and-chrome-80"></a>Compatibilidad con el atributo SameSite y Chrome 80

**Tipo:** Plan de cambio  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Autenticación de usuarios

Como parte de un modelo predeterminado y seguro para las cookies, el explorador Chrome 80 cambia el modo en que trata las cookies sin el atributo `SameSite`. Cualquier cookie que no especifique el atributo `SameSite` se tratará como si se hubiera establecido en `SameSite=Lax`, lo que producirá un bloqueo de Chrome en ciertos escenarios de uso compartido de cookies entre dominios de los que puede depender su aplicación. Para mantener el comportamiento de Chrome anterior, puede utilizar el atributo `SameSite=None` y agregar un atributo `Secure` adicional, por lo que solo se puede tener acceso a las cookies entre sitios a través de conexiones HTTPS. Chrome está programado para completar este cambio el 4 de febrero de 2020.

Se recomienda que todos los desarrolladores prueben sus aplicaciones siguiendo esta guía:

- Establecer el valor predeterminado de la opción **Usar cookies seguras** en **Sí**.

- Establecer el valor predeterminado del atributo **SameSite** en **Ninguno**.

- Agregar un atributo `SameSite` adicional de **Seguro**.

Para obtener más información, consulte el artículo sobre [próximos cambios en la cookie de SameSite en ASP.NET y ASP.NET Core](https://devblogs.microsoft.com/aspnet/upcoming-samesite-cookie-changes-in-asp-net-and-asp-net-core/) y [Posible interrupción para los sitios web de los clientes y los servicios de Microsoft en la versión 79 de Chrome Beta y versiones posteriores](https://support.microsoft.com/help/4522904/potential-disruption-to-microsoft-services-in-chrome-beta-version-79).

---

### <a name="new-hotfix-for-microsoft-identity-manager-mim-2016-service-pack-2-sp2"></a>Nueva revisión para Microsoft Identity Manager (MIM) 2016 Service Pack 2 (SP2)

**Tipo:** Corregido  
**Categoría del servicio:** Microsoft Identity Manager  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad

Un paquete acumulativo de revisiones (compilación 4.6.34.0) está disponible para Microsoft Identity Manager (MIM) 2016 Service Pack 2 (SP2). Este paquete acumulativo resuelve problemas y agrega mejoras que se describen en la sección sobre "problemas corregidos y mejoras agregadas en esta actualización".

Para obtener más información y descargar el paquete de revisiones, consulte el artículo [El paquete acumulativo de actualizaciones de Microsoft Identity Manager 2016 Service Pack 2 (compilación 4.6.34.0) está disponible](https://support.microsoft.com/help/4512924/microsoft-identity-manager-2016-service-pack-2-build-4-6-34-0-update-r).

---

### <a name="new-ad-fs-app-activity-report-to-help-migrate-apps-to-azure-ad-public-preview"></a>Nuevo informe de actividad de aplicaciones de AD FS para ayudar a migrar las aplicaciones a Azure AD (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** SSO

Use el nuevo informe de actividad de aplicaciones de Servicios de federación de Active Directory (AD FS) en Azure Portal para identificar las aplicaciones que se pueden migrar a Azure AD. El informe evalúa todas las aplicaciones de AD FS con respecto a la compatibilidad con Azure AD, comprueba si hay problemas y proporciona instrucciones sobre cómo preparar aplicaciones concretas para su migración.

Para obtener más información, consulte [Uso del informe de actividades de aplicaciones de AD FS para migrar aplicaciones a Azure AD](https://docs.microsoft.com/azure/active-directory/manage-apps/migrate-adfs-application-activity).

---

### <a name="new-workflow-for-users-to-request-administrator-consent-public-preview"></a>Nuevo flujo de trabajo para que los usuarios soliciten el consentimiento del administrador (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Control de acceso

El flujo de trabajo de consentimiento del administrador proporciona a los administradores una manera de conceder acceso a las aplicaciones que requieren la aprobación del administrador. Si un usuario intenta acceder a una aplicación, pero no puede dar su consentimiento, puede enviar una solicitud de aprobación del administrador. La solicitud se envía por correo electrónico, y se coloca en una cola a la que se puede acceder desde Azure Portal, a todos los administradores que se han designado como revisores. Una vez que un revisor realiza una acción en una solicitud pendiente, se notifica la acción a los usuarios que realizan la solicitud.

Para obtener más información, consulte [Configuración del flujo de trabajo de consentimiento del administrador (versión preliminar)](https://docs.microsoft.com/azure/active-directory/manage-apps/configure-admin-consent-workflow).

---

### <a name="new-azure-ad-app-registrations-token-configuration-experience-for-managing-optional-claims-public-preview"></a>Nueva experiencia de configuración del token de registros de aplicación de Azure AD para administrar notificaciones opcionales (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** Otros  
**Funcionalidad del producto:** Experiencia para el desarrollador

La nueva hoja de **configuración del token de registros de aplicación de Azure AD** de Azure Portal muestra ahora a los desarrolladores de aplicaciones una lista dinámica de notificaciones opcionales para sus aplicaciones. Esta nueva experiencia ayuda a simplificar las migraciones de aplicaciones de Azure AD y a minimizar las configuraciones incorrectas de las notificaciones opcionales.

Para obtener más información, consulte [Proporcionar notificaciones opcionales a la aplicación de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/active-directory-optional-claims).

---

### <a name="new-two-stage-approval-workflow-in-azure-ad-entitlement-management-public-preview"></a>Nuevo flujo de trabajo de aprobación en dos fases de administración de derechos de Azure AD (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** Otros  
**Funcionalidad del producto:** Administración de derechos

Hemos presentado un nuevo flujo de trabajo de aprobación en dos fases que le permite exigir dos aprobadores para aprobar la solicitud de un usuario a un paquete de acceso. Por ejemplo, puede establecerlo de manera que el administrador del usuario que realiza la solicitud deba dar su aprobación en primer lugar y, a continuación, también puede exigir que un propietario del recurso dé su aprobación. Si uno de los aprobadores no da su aprobación, no se concede el acceso.

Para obtener más información, consulte [Cambio de la configuración de la solicitud y aprobación para un paquete de acceso de administración de derechos de Azure AD](https://docs.microsoft.com/azure/active-directory/governance/entitlement-management-access-package-request-policy).

---

### <a name="updates-to-the-my-apps-page-along-with-new-workspaces-public-preview"></a>Actualizaciones de la página Mis aplicaciones con nuevas áreas de trabajo (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** Mis aplicaciones  
**Funcionalidad del producto:** Integración de terceros

Ahora puede personalizar la forma en que los usuarios de la organización ven y acceden a la experiencia actualizada de Mis aplicaciones. Esta nueva experiencia también incluye la nueva característica de áreas de trabajo, que facilita a los usuarios la búsqueda y organización de aplicaciones.

Para obtener más información sobre la nueva experiencia de Mis aplicaciones y la creación de áreas de trabajo, consulte el artículo sobre [creación de áreas de trabajo en el portal Mis aplicaciones](https://docs.microsoft.com/azure/active-directory/manage-apps/access-panel-workspaces).

---

### <a name="google-social-id-support-for-azure-ad-b2b-collaboration-general-availability"></a>Compatibilidad con identificadores sociales de Google para la colaboración B2B de Azure AD (disponibilidad general)

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** Autenticación de usuarios

La nueva compatibilidad con el uso de identificadores sociales de Google (cuentas Gmail) en Azure AD ayuda a simplificar la colaboración entre sus usuarios y partners. Ya no hay necesidad de que los partners creen y administren una cuenta específica de Microsoft. Microsoft Teams ahora es totalmente compatible con los usuarios de Google en todos los clientes y en los puntos de conexión de autenticación comunes y relacionados con los inquilinos.

Para obtener más información, consulte [Incorporación de Google como proveedor de identidades para los usuarios invitados de B2B](https://docs.microsoft.com/azure/active-directory/b2b/google-federation).

---

### <a name="microsoft-edge-mobile-support-for-conditional-access-and-single-sign-on-general-availability"></a>Compatibilidad de Microsoft Edge para dispositivos móviles con el acceso condicional y el inicio de sesión único (disponibilidad general)

**Tipo:** Nueva característica  
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad

Azure AD para Microsoft Edge en iOS y Android ahora admite acceso condicional e inicio de sesión único de Azure AD:

- **Inicio de sesión único (SSO) de Microsoft Edge:** El inicio de sesión único ahora está disponible en los clientes nativos (como Microsoft Outlook y Microsoft Edge) para todas las aplicaciones conectadas a Azure AD.

- **Acceso condicional de Microsoft Edge:** A través de las directivas de acceso condicional basado en aplicaciones, los usuarios deben usar exploradores protegidos por Microsoft Intune, como Microsoft Edge.

Para obtener más información sobre el acceso condicional y el inicio de sesión único con Microsoft Edge, consulte la entrada de blog sobre [compatibilidad de Microsoft Edge para dispositivos móviles con el acceso condicional y el inicio de sesión único ahora disponible con carácter general](https://techcommunity.microsoft.com/t5/Intune-Customer-Success/Microsoft-Edge-Mobile-Support-for-Conditional-Access-and-Single/ba-p/988179). Para obtener más información sobre cómo configurar las aplicaciones cliente con [acceso condicional basado en aplicaciones](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access) o [acceso condicional basado en dispositivos](https://docs.microsoft.com/azure/active-directory/conditional-access/require-managed-devices), consulte [Administración del acceso web mediante un explorador protegido por directivas de Microsoft Intune](https://docs.microsoft.com/intune/apps/app-configuration-managed-browser).

---

### <a name="azure-ad-entitlement-management-general-availability"></a>Administración de derechos de Azure AD (disponibilidad general)

**Tipo:** Nueva característica  
**Categoría del servicio:** Otros  
**Funcionalidad del producto:** Administración de derechos

La administración de derechos de Azure AD es una nueva característica de gobernanza de identidades, que ayuda a las organizaciones a administrar el ciclo de vida de identidad y acceso a escala. Esta nueva característica ayuda mediante la automatización de los flujos de trabajo de solicitud de acceso, las asignaciones de acceso, las revisiones y la expiración entre grupos, aplicaciones y sitios de SharePoint Online.

Con la administración de derechos de Azure AD, puede administrar el acceso de forma más eficaz tanto para los empleados como para los usuarios ajenos a la organización que han de acceder a esos recursos.

Para obtener más información, consulte [¿Qué es la administración de derechos de Azure AD?](https://docs.microsoft.com/azure/active-directory/governance/entitlement-management-overview#license-requirements)

---

### <a name="automate-user-account-provisioning-for-these-newly-supported-saas-apps"></a>Automatización del aprovisionamiento de cuentas de usuario para estas aplicaciones SaaS recién admitidas

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros  

Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

[SAP Cloud Platform Identity Authentication Service](https://docs.microsoft.com/azure/active-directory/saas-apps/sap-hana-cloud-platform-identity-authentication-tutorial), [RingCentral](https://docs.microsoft.com/azure/active-directory/saas-apps/ringcentral-provisioning-tutorial), [SpaceIQ](https://docs.microsoft.com/azure/active-directory/saas-apps/spaceiq-provisioning-tutorial), [Miro](https://docs.microsoft.com/azure/active-directory/saas-apps/miro-provisioning-tutorial), [Cloudgate](https://docs.microsoft.com/azure/active-directory/saas-apps/soloinsight-cloudgate-sso-provisioning-tutorial), [Infor CloudSuite](https://docs.microsoft.com/azure/active-directory/saas-apps/infor-cloudsuite-provisioning-tutorial), [OfficeSpace Software](https://docs.microsoft.com/azure/active-directory/saas-apps/officespace-software-provisioning-tutorial), [Priority Matrix](https://docs.microsoft.com/azure/active-directory/saas-apps/priority-matrix-provisioning-tutorial)

Para más información acerca de cómo proteger mejor una organización mediante el aprovisionamiento automatizado de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---november-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: noviembre de 2019

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros

En noviembre de 2019, hemos agregado estas 21 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[Airtable](https://docs.microsoft.com/azure/active-directory/saas-apps/airtable-tutorial), [Hootsuite](https://docs.microsoft.com/azure/active-directory/saas-apps/hootsuite-tutorial), [Blue Access for Members (BAM)](https://docs.microsoft.com/azure/active-directory/saas-apps/blue-access-for-members-tutorial), [Bitly](https://docs.microsoft.com/azure/active-directory/saas-apps/bitly-tutorial), [Riva](https://docs.microsoft.com/azure/active-directory/saas-apps/riva-tutorial), [ResLife Portal](https://app.reslifecloud.com/hub5_signin/microsoft_azuread/?g=44BBB1F90915236A97502FF4BE2952CB&c=5&uid=0&ht=2&ref=), [NegometrixPortal Single Sign On (SSO)](https://docs.microsoft.com/azure/active-directory/saas-apps/negometrixportal-tutorial), [TeamsChamp](https://login.microsoftonline.com/551f45da-b68e-4498-a7f5-a6e1efaeb41c/adminconsent?client_id=ca9bbfa4-1316-4c0f-a9ee-1248ac27f8ab&redirect_uri=https://admin.teamschamp.com/api/adminconsent&state=6883c143-cb59-42ee-a53a-bdb5faabf279), [Motus](https://docs.microsoft.com/azure/active-directory/saas-apps/motus-tutorial), [MyAryaka](https://docs.microsoft.com/azure/active-directory/saas-apps/myaryaka-tutorial), [BlueMail](https://loginself1.bluemail.me/), [Beedle](https://teams-web.beedle.co/#/), [Visma](https://docs.microsoft.com/azure/active-directory/saas-apps/visma-tutorial), [OneDesk](https://docs.microsoft.com/azure/active-directory/saas-apps/onedesk-tutorial), [Foko Retail](https://docs.microsoft.com/azure/active-directory/saas-apps/foko-retail-tutorial), [Qmarkets Idea & Innovation Management](https://docs.microsoft.com/azure/active-directory/saas-apps/qmarkets-idea-innovation-management-tutorial), [Netskope User Authentication](https://docs.microsoft.com/azure/active-directory/saas-apps/netskope-user-authentication-tutorial), [uniFLOW Online](https://docs.microsoft.com/azure/active-directory/saas-apps/uniflow-online-tutorial), [Claromentis](https://docs.microsoft.com/azure/active-directory/saas-apps/claromentis-tutorial), [Jisc Student Voter Registration](https://docs.microsoft.com/azure/active-directory/saas-apps/jisc-student-voter-registration-tutorial), [e4enable](https://portal.e4enable.com/)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="new-and-improved-azure-ad-application-gallery"></a>La nueva y mejorada galería de aplicaciones de Azure AD

**Tipo:** Característica modificada  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** SSO

Hemos actualizado la galería de aplicaciones de Azure AD para que le resulte más fácil encontrar aplicaciones integradas previamente que admitan el aprovisionamiento, OpenID Connect y SAML en su inquilino de Azure Active Directory.

Para obtener más información, consulte [Incorporación de una aplicación a un inquilino de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/add-application-portal).

---

### <a name="increased-app-role-definition-length-limit-from-120-to-240-characters"></a>Límite de longitud de definición de roles de aplicación aumentado de 120 a 240 caracteres

**Tipo:** Característica modificada  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** SSO

Hemos escuchado de los clientes que el límite de longitud del valor de definición de rol de aplicación en algunas aplicaciones y servicios es demasiado corto en 120 caracteres. En respuesta, hemos aumentado la longitud máxima de la definición de valor de rol a 240 caracteres.

Para obtener más información sobre el uso de definiciones de roles específicas de la aplicación, consulte Para ver más ejemplos e información, consulte [Procedimiento para agregar roles de aplicación en la aplicación y recibirlos en el token](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps).

---

## <a name="october-2019"></a>Octubre de 2019

### <a name="deprecation-of-the-identityriskevent-api-for-azure-ad-identity-protection-risk-detections"></a>Desuso de la API de identityRiskEvent para las detecciones de riesgo de Azure AD Identity Protection

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Identity Protection Seguridad y protección de la identidad

En respuesta a los comentarios de los desarrolladores, los suscriptores de Azure AD Premium P2 ahora pueden realizar consultas complejas sobre los datos de detección de riesgos de Azure AD Identity Protection mediante la nueva API de riskDetection para Microsoft Graph. La versión beta existente de la API de [identityRiskEvent](https://docs.microsoft.com/graph/api/resources/identityriskevent?view=graph-rest-beta) dejará de devolver datos en torno al **10 de enero de 2020**. Si su organización usa la API de identityRiskEvent, debe realizar la transición a la nueva API de riskDetection.

Para obtener más información sobre la nueva API de riskDetection, consulte la [documentación de referencia de la API de detección de riesgos](https://aka.ms/RiskDetectionsAPI).

---

### <a name="application-proxy-support-for-the-samesite-attribute-and-chrome-80"></a>Compatibilidad del proxy de aplicación con el atributo SameSite y Chrome 80

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Proxy de aplicaciones Control de acceso

Un par de semanas antes del lanzamiento del explorador Chrome 80, tenemos previsto actualizar el modo en que las cookies del proxy de aplicación tratan el atributo **SameSite**. Con el lanzamiento de Chrome 80, cualquier cookie que no especifique el atributo **SameSite** se tratará como si se hubiera establecido en `SameSite=Lax`.

Para ayudar a evitar posibles impactos negativos debido a este cambio, vamos a actualizar las cookies de sesión y el acceso del proxy de aplicación de la siguiente manera:

- Estableciendo el valor predeterminado de la opción **Usar cookies seguras** en **Sí**.

- Estableciendo el valor predeterminado del atributo **SameSite** en **Ninguno**.

    >[!NOTE]
    > Las cookies de acceso del proxy de aplicación siempre se han transmitido exclusivamente a través de canales seguros. Estos cambios solo se aplican a las cookies de sesión.

Para obtener más información sobre la configuración de las cookies del proxy de aplicación, consulte [Configuración de las cookies para el acceso a aplicaciones locales en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-configure-cookie-settings).

---

### <a name="app-registrations-legacy-and-app-management-in-the-application-registration-portal-appsdevmicrosoftcom-is-no-longer-available"></a>La característica Registros de aplicaciones (heredada) y la funcionalidad de administración de aplicaciones en el Portal de registro de aplicaciones (apps.dev.microsoft.com) ya no están disponibles

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** N/D Experiencia para el desarrollador

Los usuarios con cuentas de Azure AD ya no pueden registrar o administrar aplicaciones mediante el Portal de registro de aplicaciones (apps.dev.microsoft.com), ni registrar ni administrar aplicaciones en la experiencia Registros de aplicaciones (característica heredada) de Azure Portal.

Para obtener más información sobre la nueva experiencia Registros de aplicaciones, consulte la sección sobre [Registros de aplicaciones en la guía de aprendizaje de Azure Portal](../develop/app-registrations-training-guide-for-app-registrations-legacy-users.md).

---

### <a name="users-are-no-longer-required-to-re-register-during-migration-from-per-user-mfa-to-conditional-access-based-mfa"></a>Ya no es necesario que los usuarios vuelvan a registrarse durante la migración de MFA por usuario a MFA basada en el acceso condicional

**Tipo:** **Categoría del servicio:** Corregida **Funcionalidad del producto:** MFA Seguridad y protección de la identidad

Hemos corregido un problema conocido por el que se requería que los usuarios se volvieran a registrar si estaban deshabilitados para la autenticación multifactor (MFA) por usuario y luego se habilitaban para la MFA a través de una directiva de acceso condicional.

Para requerir que los usuarios vuelvan a registrarse, puede seleccionar la opción **Requerir volver a registrar MFA** en los métodos de autenticación del usuario en el portal de Azure AD. Para obtener más información sobre la migración de usuarios de MFA por usuario a MFA basada en el acceso condicional, consulte [Conversión de los usuarios de MFA por usuario a MFA basado en acceso condicional](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted#convert-users-from-per-user-mfa-to-conditional-access-based-mfa).

---

### <a name="new-capabilities-to-transform-and-send-claims-in-your-saml-token"></a>Nuevas funcionalidades para transformar y enviar notificaciones en el token SAML

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales SSO

Hemos agregado funcionalidades adicionales que lo ayudarán a personalizar y enviar notificaciones en el token SAML. Estas son algunas de ellas:

- Funciones de transformación de notificaciones adicionales, lo que lo ayudará a modificar el valor que envía en la notificación.

- Capacidad de aplicar varias transformaciones a una única notificación.

- Capacidad de especificar el origen de la notificación, en función del tipo de usuario y el grupo al que pertenece el usuario.

Para obtener información detallada sobre estas nuevas funcionalidades, incluido cómo usarlas, consulte [Personalización de las notificaciones emitidas en el token SAML para aplicaciones empresariales](https://docs.microsoft.com/azure/active-directory/develop/active-directory-saml-claims-customization).

---

### <a name="new-my-sign-ins-page-for-end-users-in-azure-ad"></a>Nueva página Mis inicios de sesión para los usuarios finales en Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Supervisión e informes

Hemos agregado una nueva página **Mis inicios de sesión** (https://mysignins.microsoft.com) ) para que los usuarios de la organización puedan ver su historial de inicio de sesión reciente con el fin de comprobar si hay alguna actividad inusual. Esta nueva página permite a los usuarios comprobar lo siguiente:

- Si alguien intenta adivinar la contraseña.

- Si un atacante inició sesión correctamente en su cuenta y desde qué ubicación.

- Las aplicaciones a las que el atacante ha intentado acceder.

Para obtener más información, consulte la entrada de blog sobre cómo [los usuarios pueden comprobar ahora su historial de inicio de sesión para actividades inusuales](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Users-can-now-check-their-sign-in-history-for-unusual-activity/ba-p/916066).

---

### <a name="migration-of-azure-ad-domain-services-azure-ad-ds-from-classic-to-azure-resource-manager-virtual-networks"></a>Migración de Azure AD Domain Services (Azure AD DS) de las redes virtuales clásicas a las de Azure Resource Manager

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure AD Domain Services Azure AD Domain Services

Tenemos excelentes noticias para los clientes que siguen usando las redes virtuales clásicas. Ahora puede realizar una migración puntual desde una red virtual clásica a una de Resource Manager. Después de migrar a la red virtual de Resource Manager, podrá aprovechar las ventajas de las características adicionales y actualizadas, como las directivas de contraseñas específicas, las notificaciones por correo electrónico y los registros de auditoría.

Para obtener más información, consulte [Versión preliminar: Migración de Azure AD Domain Services desde el modelo de red virtual clásica a Resource Manager](https://docs.microsoft.com/azure/active-directory-domain-services/migrate-from-classic-vnet).

---

### <a name="updates-to-the-azure-ad-b2c-page-contract-layout"></a>Actualizaciones del diseño del contrato de página de Azure AD B2C

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Hemos introducido algunos cambios nuevos en la versión 1.2.0 del contrato de página de Azure AD B2C. En esta versión actualizada, ahora puede controlar el orden de carga de los elementos, lo que también puede ayudar a detener el parpadeo que se produce cuando se carga la hoja de estilos (CSS).

Para ver una lista completa de los cambios realizados en el contrato de página, vea el [registro de cambios de versión](https://docs.microsoft.com/azure/active-directory-b2c/page-layout#120).

---

### <a name="update-to-the-my-apps-page-along-with-new-workspaces-public-preview"></a>Actualización de la página Mis aplicaciones con nuevas áreas de trabajo (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Mis aplicaciones Control de acceso

Ahora puede personalizar la forma en que los usuarios de su organización ven y acceden a la nueva experiencia de Mis aplicaciones, incluido el uso de la nueva característica de áreas de trabajo para que sea más fácil buscar aplicaciones. La nueva funcionalidad de áreas de trabajo actúa como un filtro para las aplicaciones a las que los usuarios de su organización ya tienen acceso.

Para obtener más información sobre cómo implementar la nueva experiencia de Mis aplicaciones y crear áreas de trabajo, consulte el artículo sobre [creación de áreas de trabajo en el portal Mis aplicaciones (versión preliminar)](https://docs.microsoft.com/azure/active-directory/manage-apps/access-panel-workspaces).

---

### <a name="support-for-the-monthly-active-user-based-billing-model-general-availability"></a>Compatibilidad con el modelo de facturación basado en usuarios activos mensuales (disponibilidad general)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Azure AD B2C admite ahora la facturación de usuarios activos mensuales (MAU). La facturación de MAU se basa en el número de usuarios únicos con actividad de autenticación durante un mes natural. Los clientes existentes pueden pasarse a este nuevo método de facturación en cualquier momento.

A partir del 1 de noviembre de 2019, a todos los clientes nuevos se les facturará automáticamente con este método. Este método de facturación beneficia a los clientes gracias a las ventajas que ofrece en cuanto a costos y a la posibilidad de planear con anterioridad.

Para obtener más información, consulte [Actualización al modelo de facturación de usuarios activos mensuales](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-how-to-enable-billing#upgrade-to-monthly-active-users-billing-model).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---october-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: octubre de 2019

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En octubre de 2019, hemos agregado estas 35 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[In Case of Crisis – Mobile](https://docs.microsoft.com/azure/active-directory/saas-apps/in-case-of-crisis-mobile-tutorial), [Juno Journey](https://docs.microsoft.com/azure/active-directory/saas-apps/juno-journey-tutorial), [ExponentHR](https://docs.microsoft.com/azure/active-directory/saas-apps/exponenthr-tutorial), [Tact](https://tact.ai/assistant/), [OpusCapita Cash Management](http://cm1.opuscapita.com/tenantname), [Salestim](https://prd.salestim.io/forms), [Learnster](https://docs.microsoft.com/azure/active-directory/saas-apps/learnster-tutorial), [Dynatrace](https://docs.microsoft.com/azure/active-directory/saas-apps/dynatrace-tutorial), [HunchBuzz](https://login.hunchbuzz.com/integrations/azure/process), [Freshworks](https://docs.microsoft.com/azure/active-directory/saas-apps/freshworks-tutorial), [eCornell](https://docs.microsoft.com/azure/active-directory/saas-apps/ecornell-tutorial), [ShipHazmat](https://docs.microsoft.com/azure/active-directory/saas-apps/shiphazmat-tutorial), [Netskope Cloud Security](https://docs.microsoft.com/azure/active-directory/saas-apps/netskope-cloud-security-tutorial), [Contentful](https://docs.microsoft.com/azure/active-directory/saas-apps/contentful-tutorial), [Bindtuning](https://bindtuning.com/login), [HireVue Coordinate – Europe](https://www.hirevue.com/), [HireVue Coordinate - USOnly](https://www.hirevue.com/), [HireVue Coordinate - US](https://www.hirevue.com/), [WittyParrot Knowledge Box](https://wittyapi.wittyparrot.com/wittyparrot/api/provision/trail/signup), [Cloudmore](https://docs.microsoft.com/azure/active-directory/saas-apps/cloudmore-tutorial), [Visit.org](https://docs.microsoft.com/azure/active-directory/saas-apps/visitorg-tutorial), [Cambium Xirrus EasyPass Portal](https://login.xirrus.com/azure-signup), [Paylocity](https://docs.microsoft.com/azure/active-directory/saas-apps/paylocity-tutorial), [Mail Luck!](https://docs.microsoft.com/azure/active-directory/saas-apps/mail-luck-tutorial), [Teamie](https://theteamie.com/), [Velocity for Teams](https://velocity.peakup.org/teams/login), [SIGNL4](https://account.signl4.com/manage), [EAB Navigate IMPL](https://docs.microsoft.com/azure/active-directory/saas-apps/eab-navigate-impl-tutorial), [ScreenMeet](https://console.screenmeet.com/), [Omega Point](https://pi.ompnt.com/), [Speaking Email for Intune (iPhone)](https://speaking.email/FAQ/98/email-access-via-microsoft-intune), [Speaking Email for Office 365 Direct (iPhone/Android)](https://speaking.email/FAQ/126/email-access-via-microsoft-office-365-direct), [ExactCare SSO](https://docs.microsoft.com/azure/active-directory/saas-apps/exactcare-sso-tutorial), [iHealthHome Care Navigation System](https://ihealthnav.com/account/signin), [Qubie](https://qubie.azurewebsites.net/static/adminTab/authorize.html)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="consolidated-security-menu-item-in-the-azure-ad-portal"></a>Elemento de menú consolidado Seguridad en el portal de Azure AD

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Identity Protection Seguridad y protección de la identidad

Ahora puede acceder a todas las características de seguridad de Azure AD disponibles en el nuevo elemento de menú **Seguridad** y en la barra **Buscar** de Azure Portal. Además, la nueva página de aterrizaje **Seguridad** denominada **Seguridad - Introducción** proporcionará vínculos a nuestra documentación pública, así como guías de seguridad y de implementación.

El nuevo menú **Seguridad** incluye lo siguiente:

- Acceso condicional
- Protección de identidad
- Security Center
- Puntuación segura de identidad
- Métodos de autenticación
- MFA
- Informes de riesgo: usuarios de riesgo, inicios de sesión peligrosos, detecciones de riesgo, etc.
- Y mucho más.

Para obtener más información, visite [Seguridad - Introducción](https://portal.azure.com/#blade/Microsoft_AAD_IAM/SecurityMenuBlade/GettingStarted).

---

### <a name="office-365-groups-expiration-policy-enhanced-with-autorenewal"></a>Directiva de expiración de grupos de Office 365 mejorada con renovación automática

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Administración de grupos Administración del ciclo de vida de la identidad

La directiva de expiración de grupos de Office 365 se ha mejorado para renovar automáticamente los grupos que los miembros estén usando activamente. Los grupos se renuevan en función de la actividad del usuario en todas las aplicaciones de Office 365, como Outlook, SharePoint y Teams.

Esta mejora ayuda a reducir las notificaciones de expiración de los grupos, así como a garantizar que los grupos activos sigan estando disponibles. Si ya tiene una directiva de expiración activa para grupos de Office 365, no es necesario hacer nada para activar esta nueva funcionalidad.

Para obtener más información, vea [Configuración de la directiva de expiración de grupos de Office 365](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-lifecycle).

---

### <a name="updated-azure-ad-domain-services-azure-ad-ds-creation-experience"></a>Experiencia de creación de Azure AD Domain Services (Azure AD DS) actualizada

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Azure AD Domain Services Azure AD Domain Services

Hemos actualizado Azure AD Domain Services (Azure AD DS) para incluir una experiencia de creación nueva y mejorada, que lo ayudará a crear un dominio administrado en tan solo tres clics. Además, ahora puede cargar e implementar instancias de Azure AD DS desde una plantilla.

Para más información, consulte el [Tutorial: Creación y configuración de una instancia de Azure Active Directory Domain Services](https://docs.microsoft.com/azure/active-directory-domain-services/tutorial-create-instance).

---

## <a name="september-2019"></a>Septiembre de 2019

### <a name="plan-for-change-deprecation-of-the-power-bi-content-packs"></a>Planeado su cambio: Desuso de los paquetes de contenido de Power BI

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto**: Informes Supervisión e informes

A partir del 1 de octubre de 2019, Power BI comenzará a dejar de usar todos los paquetes de contenido, incluido el paquete de contenido de Azure AD Power BI. Como alternativa a este paquete de contenido, puede usar libros de Azure AD para obtener información sobre los servicios relacionados con Azure AD. Próximos libros de trabajo adicionales, incluidos los libros sobre las directivas de acceso condicional en modo de solo informes, información basada en el consentimiento de la aplicación, etc.

Para obtener más información acerca de los libros, consulte [Cómo usar los libros de Azure Monitor en informes de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-use-azure-monitor-workbooks). Para obtener más información sobre el desuso de los paquetes de contenido, consulte la publicación de blog [Anuncio de disponibilidad general de aplicaciones de plantilla de Power BI](https://powerbi.microsoft.com/blog/announcing-power-bi-template-apps-general-availability/).

---

### <a name="my-profile-is-renaming-and-integrating-with-the-microsoft-office-account-page"></a>Mi perfil cambia de nombre y se integra con la página de la cuenta de Microsoft Office

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto**: Mi perfil/cuenta Colaboración

A partir de octubre, la experiencia Mi perfil se convertirá en Mi cuenta. Como parte de ese cambio, en todas partes en las que aparezca **Mi perfil**, este pasará a llamarse **Mi cuenta**. Además del cambio de nomenclatura y algunas mejoras de diseño, la experiencia actualizada ofrecerá una integración adicional con la página de la cuenta de Microsoft Office. En concreto, podrá obtener acceso a las instalaciones y suscripciones de Office desde la página **Overview Account** (Descripción general de la cuenta), junto con las preferencias de contacto relacionadas con Office desde la página **Privacy** (Privacidad).

Para obtener más información sobre la experiencia Mi perfil (versión preliminar), consulte la [descripción general del portal Mi perfil (versión preliminar)](https://docs.microsoft.com/azure/active-directory/user-help/myprofile-portal-overview).

---

### <a name="bulk-manage-groups-and-members-using-csv-files-in-the-azure-ad-portal-public-preview"></a>Administración masiva de grupos y miembros mediante archivos CSV en el portal de Azure AD (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración de grupos Colaboración

Nos complace anunciar la disponibilidad de la versión preliminar pública de las experiencias de administración masiva de grupos en el portal de Azure AD. Ahora puede usar un archivo CSV y el portal de Azure AD para administrar grupos y listas de miembros, entre los que se incluyen:

- Incorporación o eliminación de miembros de un grupo.

- Descarga de la lista de grupos del directorio.

- Descarga de la lista de miembros del grupo para un grupo específico.

Para obtener más información, consulte [Agregar miembros de forma masiva](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-bulk-import-members), [Eliminar miembros de forma masiva](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-bulk-remove-members), [Descarga masiva de los miembros de la lista](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-bulk-download-members) y [Descarga masiva de los grupos de la lista](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-bulk-download).

---

### <a name="dynamic-consent-is-now-supported-through-a-new-admin-consent-endpoint"></a>Además se admite el consentimiento dinámico a través de un nuevo punto de conexión de consentimiento del administrador.

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Hemos creado un nuevo punto de conexión de consentimiento del administrador para admitir la opción de consentimiento dinámico, lo que le resultará útil para las aplicaciones que vayan a usar el modelo de consentimiento dinámico en la plataforma de Microsoft Identity.

Para obtener más información sobre cómo usar este nuevo punto de conexión, consulte [Usar el punto de conexión del consentimiento de administrador](https://docs.microsoft.com/azure/active-directory/develop/v2-admin-consent).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---september-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: septiembre de 2019

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En septiembre de 2019, hemos agregado estas 29 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[ScheduleLook](https://schedulelook.bbsonlineservices.net/), [MS Azure SSO Access for Ethidex Compliance Office&trade; - Single  sign-on](https://docs.microsoft.com/azure/active-directory/saas-apps/ms-azure-sso-access-for-ethidex-compliance-office-tutorial), [iServer Portal](https://docs.microsoft.com/azure/active-directory/saas-apps/iserver-portal-tutorial), [SKYSITE](https://docs.microsoft.com/azure/active-directory/saas-apps/skysite-tutorial), [Concur Travel and Expense](https://docs.microsoft.com/azure/active-directory/saas-apps/concur-travel-and-expense-tutorial), [WorkBoard](https://docs.microsoft.com/azure/active-directory/saas-apps/workboard-tutorial), `https://apps.yeeflow.com/`, [ARC Facilities](https://docs.microsoft.com/azure/active-directory/saas-apps/arc-facilities-tutorial), [Luware Stratus Team](https://stratus.emea.luware.cloud/login), [Wide Ideas](https://wideideas.online/wideideas/), [Prisma Cloud](https://docs.microsoft.com/azure/active-directory/saas-apps/prisma-cloud-tutorial), [JDLT Client Hub](https://clients.jdlt.co.uk/login), [RENRAKU](https://docs.microsoft.com/azure/active-directory/saas-apps/renraku-tutorial), [SealPath Secure Browser](https://protection.sealpath.com/SealPathInterceptorWopiSaas/Open/InstallSealPathEditorOneDrive), [Prisma Cloud](https://docs.microsoft.com/azure/active-directory/saas-apps/prisma-cloud-tutorial), `https://app.penneo.com/`, `https://app.testhtm.com/settings/email-integration`, [Cintoo Cloud](https://aec.cintoo.com/login), [Whitesource](https://docs.microsoft.com/azure/active-directory/saas-apps/whitesource-tutorial), [Hosted Heritage Online SSO](https://docs.microsoft.com/azure/active-directory/saas-apps/hosted-heritage-online-sso-tutorial), [IDC](https://docs.microsoft.com/azure/active-directory/saas-apps/idc-tutorial), [CakeHR](https://docs.microsoft.com/azure/active-directory/saas-apps/cakehr-tutorial), [BIS](https://docs.microsoft.com/azure/active-directory/saas-apps/bis-tutorial), [Coo Kai Team Build](https://ms-contacts.coo-kai.jp/), [Sonarqube](https://docs.microsoft.com/azure/active-directory/saas-apps/sonarqube-tutorial), [Adobe Identity Management](https://docs.microsoft.com/azure/active-directory/saas-apps/adobe-identity-management-tutorial), [Discovery Benefits SSO](https://docs.microsoft.com/azure/active-directory/saas-apps/discovery-benefits-sso-tutorial), [Amelio](https://app.amelio.co/), `https://itask.yipinapp.com/`

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="new-azure-ad-global-reader-role"></a>Nuevo rol de lector global de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: RBAC Control de acceso

A partir del 24 de septiembre de 2019, empezaremos a implementar un nuevo rol de Azure Active Directory (AD) denominado Lector global. Este lanzamiento comenzará con los clientes de producción y de la nube global (GCC), y finalizará en octubre a nivel global.

El rol del Lector global es la contrapartida de la opción de solo lectura del administrador global. Los usuarios de este rol pueden leer la configuración y la información administrativa en los servicios de Microsoft 365, pero no pueden llevar a cabo acciones de administración. Por ello, hemos creado el rol del Lector global para que pueda reducir la cantidad de administradores globales en su organización. Debido a que las cuentas de administrador global son a la vez eficaces y vulnerables a los ataques, le recomendamos que tenga menos de cinco administradores globales. Igualmente, le recomendamos usar el rol de Lector global para la planificación, las auditorías o las investigaciones. También le recomendamos usar el rol de Lector global en combinación con otros roles de administrador limitados, como el de administrador de Exchange, para que pueda realizar el trabajo necesario sin tener que usar el rol de administrador global.

El rol del Lector global funciona con el nuevo Centro de administración de Microsoft 365, el Centro de administración de Exchange, el Centro de administración de Teams, el Centro de seguridad, el Centro de cumplimiento, el Centro de administración de Azure AD y el Centro de administración de la administración de dispositivos.

>[!NOTE]
> Al comienzo de la versión preliminar pública, el rol de Lector global no funcionará con: SharePoint, Privileged Access Management, Caja de seguridad del cliente, etiquetas de confidencialidad, Ciclo de vida de Teams, Creación de informes y análisis de llamadas de Teams, Administración de dispositivos de teléfono IP de Teams y Catálogo de aplicaciones de Teams.

Para obtener más información, consulte los [permisos del rol de administrador en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles).

---

### <a name="access-an-on-premises-report-server-from-your-power-bi-mobile-app-using-azure-active-directory-application-proxy"></a>Acceder a un servidor de informes local desde la aplicación Power BI Mobile con Azure Active Directory Application Proxy

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Proxy de aplicaciones Control de acceso

La nueva integración entre la aplicación Power BI Mobile y Azure AD Application Proxy le permite iniciar sesión de forma segura en la aplicación Power BI Mobile y ver cualquiera de los informes de la organización hospedados en la instancia local de Power BI Report Server.

Para obtener más información acerca de la aplicación de Power BI Mobile, incluyendo el lugar desde dónde descargar la aplicación, consulte el [sitio de Power BI](https://powerbi.microsoft.com/mobile/). Para obtener más información sobre cómo configurar la aplicación de Power BI Mobile con Azure AD Application Proxy, consulte [Habilitar el acceso remoto a Power BI Mobile con Azure AD Application Proxy](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-integrate-with-power-bi).

---

### <a name="new-version-of-the-azureadpreview-powershell-module-is-available"></a>Hay una nueva versión del módulo de PowerShell de AzureADPreview disponible

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Otros Directorio

Se agregaron nuevos cmdlets al módulo AzureADPreview, para ayudar a definir y asignar roles personalizados en Azure AD, que incluyen:

- `Add-AzureADMSFeatureRolloutPolicyDirectoryObject`
- `Get-AzureADMSFeatureRolloutPolicy`
- `New-AzureADMSFeatureRolloutPolicy`
- `Remove-AzureADMSFeatureRolloutPolicy`
- `Remove-AzureADMSFeatureRolloutPolicyDirectoryObject`
- `Set-AzureADMSFeatureRolloutPolicy`

---

### <a name="new-version-of-azure-ad-connect"></a>Nueva versión de Azure AD Connect

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Otros Directorio

Hemos publicado una versión actualizada de Azure AD Connect para clientes que usen la actualización automática. Esta nueva versión incluye varias características nuevas, mejoras y correcciones de errores. Para obtener más información sobre esta versión nueva, consulte [Azure AD Connect: historial de versiones](https://docs.microsoft.com/azure/active-directory/hybrid/reference-connect-version-history#14250).

---

### <a name="azure-multi-factor-authentication-mfa-server-version-802-is-now-available"></a>Ya está disponible la versión 8.0.2 del servidor de Multi-Factor Authentication de Azure (MFA)

**Tipo:** **Categoría del servicio:** Corregida **Funcionalidad del producto:** MFA Seguridad y protección de la identidad

Si es un cliente ya existente que activó el servidor MFA antes del 1 de julio de 2019, ya puede descargar la versión más reciente del servidor MFA (versión 8.0.2). En esta nueva versión, hemos realizado lo siguiente:

- Se corrigió un problema por el que, cuando Azure AD Sync cambia el estado de un usuario de "deshabilitado" a "habilitado", se envía un correo electrónico al mismo.

- Se corrigió un problema para que los clientes puedan realizar sus actualizaciones correctamente, al tiempo que se sigue usando la funcionalidad de etiquetas.

- Se agregó el código de país de Kosovo (+ 383).

- Se agregó un registro de auditoría de omisión por única vez a MultiFactorAuthSvc.log.

- Se mejoró el rendimiento del SDK del servicio web.

- Se corrigieron otros errores menores.

A partir del 1 de julio de 2019, Microsoft dejó de ofrecer el servidor de MFA en las nuevas implementaciones. Los clientes nuevos que quieran exigir la autenticación multifactor a sus usuarios deberán usar Azure Multi-Factor Authentication basado en la nube. Para obtener más información, consulte [Planificación de una implementación de Azure Multi-Factor Authentication basada en la nube](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted).

---

## <a name="august-2019"></a>Agosto de 2019

### <a name="enhanced-search-filtering-and-sorting-for-groups-is-available-in-the-azure-ad-portal-public-preview"></a>Funciones de búsqueda, filtrado y ordenación mejorados para los grupos están disponibles en el portal de Azure AD (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración de grupos Colaboración

Nos complace anunciar la disponibilidad de la versión preliminar pública de las experiencias mejoradas relacionadas con grupos en el portal de Azure AD. Estas mejoras ayudan a administrar mejor los grupos y las listas de miembros, ya que proporcionan:

- Funcionalidades de búsqueda avanzada, como la búsqueda de subcadenas en listas de grupos.
- Opciones avanzadas de filtrado y ordenación en las listas de miembros y propietarios.
- Nuevas funcionalidades de búsqueda para listas de miembros y propietarios.
- Recuentos de grupos más precisos para grupos grandes.

Para obtener más información, consulte [Administración de grupos en Azure Portal](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-groups-members-azure-portal?context=azure/active-directory/users-groups-roles/context/ugr-context).

---

### <a name="new-custom-roles-are-available-for-app-registration-management-public-preview"></a>Nuevos roles personalizados están disponibles para la administración de registro de aplicaciones (Versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: RBAC Control de acceso

Los roles personalizados (disponibles con una suscripción Azure AD P1 o P2) ahora pueden ayudarle a proporcionar acceso específico, permitiéndole crear definiciones de roles con permisos específicos y, a continuación, asignar esos roles a recursos específicos. Actualmente, se crean roles personalizados mediante el uso de permisos para administrar los registros de aplicaciones y, a continuación, se asigna el rol a una aplicación específica. Para obtener más información sobre los roles personalizados, consulte [Roles de administrador personalizados en Azure Active Directory (versión preliminar)](https://docs.microsoft.com/azure/active-directory/users-groups-roles/roles-custom-overview).

Si necesita más permisos o recursos admitidos que no vea actualmente, puede enviar comentarios a nuestro [sitio de comentarios de Azure](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032) e incorporaremos su solicitud a nuestra guía de ruta de actualización.

---

### <a name="new-provisioning-logs-can-help-you-monitor-and-troubleshoot-your-app-provisioning-deployment-public-preview"></a>Los nuevos registros de aprovisionamiento pueden ayudarle a supervisar y solucionar problemas de la implementación de aprovisionamiento de aplicaciones (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aprovisionamiento de aplicaciones Administración del ciclo de vida de la identidad

Nuevos registros de aprovisionamiento disponibles para ayudarle a supervisar y solucionar problemas de la implementación de aprovisionamiento de usuarios y grupos. Estos nuevos archivos de registro incluyen información sobre:

- Qué grupos se han creado correctamente en [ServiceNow](https://docs.microsoft.com/azure/active-directory/saas-apps/servicenow-provisioning-tutorial)
- Cómo se han importado los roles de [Amazon Web Services(AWS)](https://docs.microsoft.com/azure/active-directory/saas-apps/amazon-web-service-tutorial#configure-and-test-azure-ad-single-sign-on-for-amazon-web-services-aws)
- Qué empleados no se importaron desde [WorkDay](https://docs.microsoft.com/azure/active-directory/saas-apps/workday-inbound-tutorial)

Para obtener más información, consulte [Informes de aprovisionamiento en el portal de Azure Active Directory (versión preliminar)](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-provisioning-logs).

---

### <a name="new-security-reports-for-all-azure-ad-administrators-general-availability"></a>Nuevos informes de seguridad para todos los administradores de Azure AD (Disponibilidad general)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Identity Protection Seguridad y protección de la identidad

De forma predeterminada, todos los administradores de Azure AD podrán acceder pronto a informes de seguridad modernos dentro de Azure AD. Hasta el final de septiembre, podrá usar el banner de la parte superior de los informes de seguridad modernos para volver a los informes antiguos.

Los informes de seguridad modernos proporcionarán funciones adicionales de las versiones anteriores, entre las que se incluyen:

- Filtrado y ordenación avanzados
- Acciones en masa, como descartar el riesgo del usuario
- Confirmación de entidades en peligro o seguras
- Estado de riesgo, que abarca: En riesgo, Descartado, Corregido y Confirmado (en peligro)
- Nuevas detecciones relacionadas con riesgos (disponibles para suscriptores de Azure AD Premium)

Para obtener más información, consulte [ Usuarios riesgosos ](https://docs.microsoft.com/azure/active-directory/identity-protection/howto-investigate-risk#risky-users), [ Inicios de sesión riesgosos ](https://docs.microsoft.com/azure/active-directory/identity-protection/howto-investigate-risk#risky-sign-ins) y [ Detecciones de riesgos ](https://docs.microsoft.com/azure/active-directory/identity-protection/howto-investigate-risk#risk-detections).

---

### <a name="user-assigned-managed-identity-is-available-for-virtual-machines-and-virtual-machine-scale-sets-general-availability"></a>La identidad administrada asignada por el usuario está disponible para máquinas virtuales y conjuntos de escalado de máquinas virtuales (Disponibilidad general)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Identidades administradas para recursos de Azure Experiencia para el desarrollador

Las identidades administradas asignadas por el usuario ahora están generalmente disponibles para máquinas virtuales y conjuntos de escalado de máquinas virtuales. Como parte de esto, Azure puede crear una identidad en el inquilino de Azure AD que sea de confianza para la suscripción en uso y se puede asignar a una o varias instancias de servicio de Azure. Para obtener más información sobre las identidades administradas asignadas por el usuario, consulte [ ¿Qué son las identidades administradas para los recursos de Azure? ](https://aka.ms/azuremanagedidentity).

---

### <a name="users-can-reset-their-passwords-using-a-mobile-app-or-hardware-token-general-availability"></a>Los usuarios pueden restablecer sus contraseñas mediante una aplicación móvil o un token de hardware (Disponibilidad general)

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autoservicio de restablecimiento de contraseña Autenticación de usuarios

Los usuarios que han registrado una aplicación móvil en su organización ahora pueden restablecer su propia contraseña mediante la aprobación de una notificación de la aplicación de Microsoft Authenticator o mediante la introducción de un código de su aplicación móvil o token de hardware.

Para obtener más información, [Consulte cómo funciona: Autoservicio de restablecimiento de contraseña de Azure AD](https://aka.ms/authappsspr). Para obtener más información sobre la experiencia del usuario, consulte [Información general sobre cómo restablecer su propia contraseña profesional o educativa](https://docs.microsoft.com/azure/active-directory/user-help/user-help-password-reset-overview).

---

### <a name="adalnet-ignores-the-msalnet-shared-cache-for-on-behalf-of-scenarios"></a>ADAL.NET ignora la memoria caché compartida de MSAL.NET para escenarios en representación

**Tipo:** **Categoría del servicio:** Corregida **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

A partir de la versión 5.0.0-preview de la biblioteca de autenticación de Azure AD (ADAL.NET), los desarrolladores de aplicaciones deben [serializar una caché por cuenta para las aplicaciones web y API web](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Token-cache-serialization#custom-token-cache-serialization-in-web-applications--web-api). En caso contrario, algunos escenarios que usan el [flujo en representación de](https://docs.microsoft.com/azure/active-directory/develop/scenario-web-api-call-api-app-configuration#on-behalf-of-flow), junto con algunos casos de uso específicos de `UserAssertion`, pueden dar como resultado una elevación de privilegios. Para evitar esta vulnerabilidad, ADAL.NET ignora ahora la caché compartida de la biblioteca de autenticación de Microsoft para dotnet (MSAL.NET) para escenarios en representación.

Para más información acerca de este problema, consulte [Vulnerabilidad de elevación de privilegios de la Biblioteca de autenticación de Active Directory](https://portal.msrc.microsoft.com/security-guidance/advisory/CVE-2019-1258).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---august-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: agosto de 2019

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En agosto de 2019, hemos agregado estas 26 nuevas aplicaciones con compatibilidad con la federación para la galería de aplicaciones:

[Civic Platform](https://docs.microsoft.com/azure/active-directory/saas-apps/civic-platform-tutorial), [Amazon Business](https://docs.microsoft.com/azure/active-directory/saas-apps/amazon-business-tutorial), [ProNovos Ops Manager](https://docs.microsoft.com/azure/active-directory/saas-apps/pronovos-ops-manager-tutorial), [Cognidox](https://docs.microsoft.com/azure/active-directory/saas-apps/cognidox-tutorial), [Viareport's Inativ Portal (Europe)](https://docs.microsoft.com/azure/active-directory/saas-apps/viareports-inativ-portal-europe-tutorial), [Azure Databricks](https://azure.microsoft.com/services/databricks), [Robin](https://docs.microsoft.com/azure/active-directory/saas-apps/robin-tutorial), [Academy Attendance](https://docs.microsoft.com/azure/active-directory/saas-apps/academy-attendance-tutorial), [Priority Matrix](https://sync.appfluence.com/pmwebng/), [Cousto MySpace](https://cousto.platformers.be/account/login), [Uploadcare](https://uploadcare.com/accounts/signup/), [Carbonite Endpoint Backup](https://docs.microsoft.com/azure/active-directory/saas-apps/carbonite-endpoint-backup-tutorial), [CPQSync by Cincom](https://docs.microsoft.com/azure/active-directory/saas-apps/cpqsync-by-cincom-tutorial), [Chargebee](https://docs.microsoft.com/azure/active-directory/saas-apps/chargebee-tutorial), [deliver.media&trade; Portal](https://portal.deliver.media), [Frontline Education](https://docs.microsoft.com/azure/active-directory/saas-apps/frontline-education-tutorial), [F5](https://www.f5.com/products/security/access-policy-manager), [stashcat AD connect](https://www.stashcat.com), [Blink](https://docs.microsoft.com/azure/active-directory/saas-apps/blink-tutorial), [Vocoli](https://docs.microsoft.com/azure/active-directory/saas-apps/vocoli-tutorial), [ProNovos Analytics](https://docs.microsoft.com/azure/active-directory/saas-apps/pronovos-analytics-tutorial), [Sigstr](https://docs.microsoft.com/azure/active-directory/saas-apps/sigstr-tutorial), [Darwinbox](https://docs.microsoft.com/azure/active-directory/saas-apps/darwinbox-tutorial), [Watch by Colors](https://docs.microsoft.com/azure/active-directory/saas-apps/watch-by-colors-tutorial), [Harness](https://docs.microsoft.com/azure/active-directory/saas-apps/harness-tutorial), [EAB Navigate Strategic Care](https://docs.microsoft.com/azure/active-directory/saas-apps/eab-navigate-strategic-care-tutorial)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="new-versions-of-the-azuread-powershell-and-azureadpreview-powershell-modules-are-available"></a>Están disponibles nuevas versiones de los módulos AzureAD PowerShell y AzureADPreview PowerShell.

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Otros Directorio

Están disponibles nuevas versiones de los módulos AzureAD PowerShell y AzureADPreview PowerShell.

- Se agregó un nuevo parámetro `-Filter` al parámetro `Get-AzureADDirectoryRole` en el módulo AzureAD. Este parámetro le ayuda a filtrar por los roles de directorio devueltos por el cmdlet.
- Se agregaron nuevos cmdlets al módulo AzureADPreview, para ayudar a definir y asignar roles personalizados en Azure AD, que incluyen:

    - `Get-AzureADMSRoleAssignment`
    - `Get-AzureADMSRoleDefinition`
    - `New-AzureADMSRoleAssignment`
    - `New-AzureADMSRoleDefinition`
    - `Remove-AzureADMSRoleAssignment`
    - `Remove-AzureADMSRoleDefinition`
    - `Set-AzureADMSRoleDefinition`

---

### <a name="improvements-to-the-ui-of-the-dynamic-group-rule-builder-in-the-azure-portal"></a>Mejoras en la interfaz de usuario del generador de reglas de grupos dinámicos en Azure Portal

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Administración de grupos Colaboración

Hemos realizado algunas mejoras de la interfaz de usuario en el generador de reglas de grupo dinámico, disponible en el Azure Portal, para ayudarle a configurar más fácilmente una nueva regla o cambiar las reglas existentes. Esta mejora del diseño le permite crear reglas con hasta cinco expresiones, en lugar de solo una. También hemos actualizado la lista de propiedades del dispositivo para quitar las propiedades del dispositivo en desuso.

Para obtener más información, consulte [Administrar reglas de pertenencia dinámica](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-dynamic-membership).

---

### <a name="new-microsoft-graph-app-permission-available-for-use-with-access-reviews"></a>Nuevo permiso de aplicación Microsoft Graph disponible para su uso con las revisiones de acceso

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Revisiones de acceso Identity Governance

Hemos introducido un nuevo permiso de aplicación Microsoft Graph, `AccessReview.ReadWrite.Membership`, que permite a las aplicaciones crear y recuperar automáticamente revisiones de acceso para pertenencias a grupos y asignaciones de aplicaciones. Este permiso se puede usar en los trabajos programados o como parte de la automatización, sin necesidad de un contexto de usuario que haya iniciado sesión.

Para obtener más información, consulte el [Ejemplo de cómo crear revisiones de acceso de Azure AD mediante los permisos de la aplicación Microsoft Graph con el blog de PowerShell ](https://techcommunity.microsoft.com/t5/Azure-Active-Directory/Example-how-to-create-Azure-AD-access-reviews-using-Microsoft/m-p/807241).

---

### <a name="azure-ad-activity-logs-are-now-available-for-government-cloud-instances-in-azure-monitor"></a>Los registros de actividad de Azure AD ahora están disponibles para las instancias de la nube de administración pública en Azure Monitor

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto**: Informes Supervisión e informes

Nos complace anunciar que los registros de actividad de Azure AD ahora están disponibles para las instancias de la nube de administración pública en Azure Monitor Ahora puede enviar registros de Azure AD a su cuenta de almacenamiento o a un centro de eventos para integrarlos con las herramientas SIEM, como [Sumologic](https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-sumologic), [Splunk](https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-splunk) y [Arcsight](https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-arcsight).

Para obtener más información sobre la configuración de Azure Monitor, consulte [Registros de actividad de Azure AD en Azure Monitor](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-activity-logs-azure-monitor#cost-considerations).

---

### <a name="update-your-users-to-the-new-enhanced-security-info-experience"></a>Actualice a sus usuarios a la nueva experiencia de información de seguridad mejorada

**Tipo:** **Categoría del servicio:** Característica modificada  **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

El 25 de septiembre de 2019, desactivaremos la antigua experiencia de información de seguridad no mejorada para registrar y administrar la información de seguridad del usuario y solo activaremos la nueva [versión mejorada](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Cool-enhancements-to-the-Azure-AD-combined-MFA-and-password/ba-p/354271). Esto significa que los usuarios ya no podrán usar la experiencia antigua.

Para más información sobre la experiencia de información de seguridad mejorada, consulte nuestra [documentación de administración](https://aka.ms/securityinfodocs) y nuestra [documentación de usuario](https://aka.ms/securityinfoguide).

#### <a name="to-turn-on-this-new-experience-you-must"></a>Para activar esta nueva experiencia, debe:

1. Inicie sesión en Azure Portal como administrador global o administrador de usuarios.

2. Vaya a **Azure Active Directory > Configuración de usuario > Administrar la configuración de las características en versión preliminar del panel de acceso**.

3. En el área **Mejoras para los usuarios a la hora de utilizar las características en versión preliminar para registrar y administrar la información de seguridad**, seleccione **Seleccionado** y luego elija un grupo de usuarios o elija **Todos** a fin de activar esta función para todos los usuarios del inquilino.

4. En el **Los usuarios pueden usar las características de vista previa para registrar y administrar el área de información **de seguridad**,** seleccione **Ninguno**.

5. Guarde la configuración.

    Después de guardar la configuración, ya no tendrá acceso a la antigua experiencia de información de seguridad.

>[!Important]
>Si no completa estos pasos antes del 25 de septiembre de 2019, el inquilino de Azure Active Directory se habilitará automáticamente para mejorar la experiencia. Si le queda alguna duda, póngase en contacto con nosotros registrationpreview@microsoft.com.

---

### <a name="authentication-requests-using-post-logins-will-be-more-strictly-validated"></a>Las solicitudes de autenticación que utilizan inicios de sesión POST serán validadas de forma más rigurosa

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Estándares

A partir del 2 de septiembre de 2019, las solicitudes de autenticación que usan el método POST se validarán de forma más rigurosa con los estándares HTTP. Concretamente, los espacios y las comillas dobles (") ya no se quitarán de los valores del formulario de solicitud. No se espera que estos cambios interrumpan ningún cliente existente y ayudarán a garantizar que las solicitudes enviadas a Azure AD se controlan de forma confiable en todo momento.

Para obtener más información, consulte [Notificaciones de cambios de última hora de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/reference-breaking-changes#post-form-semantics-will-be-enforced-more-strictly---spaces-and-quotes-will-be-ignored).

---

## <a name="july-2019"></a>Julio de 2019

### <a name="plan-for-change-application-proxy-service-update-to-support-only-tls-12"></a>Planeado su cambio: actualización del servicio Proxy de la aplicación para que admita solo TLS 1.2

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Proxy de aplicaciones Control de acceso

Como ayuda para que le proporcionemos nuestro cifrado más seguro, vamos a empezar a limitar el acceso del servicio Proxy de la aplicación para solo a los protocolos TLS 1.2. Esta limitación se distribuirá inicialmente a los clientes que ya usen protocolos TLS 1,2, por lo que no verá el impacto. Los protocolos TLS 1.0 y TLS 1.1 quedarán totalmente en desuso el 31 de agosto de 2019. Los clientes que aún usen TLS 1.0 y TLS 1.1 recibirán un aviso con tiempo suficiente para prepararse para este cambio.

Para mantener la conexión con el servicio Proxy de la aplicación durante todo este cambio, es aconsejable asegurarse de que las combinaciones de cliente-servidor y explorador web-servidor se actualizan para que usen TLS 1.2. También es aconsejable asegurarse de incluir los sistemas cliente que usan los empleados para acceder a las aplicaciones publicadas mediante el servicio Proxy de la aplicación.

Para obtener más información, vea [Adición de una aplicación local para el acceso remoto mediante el proxy de aplicación en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-add-on-premises-application).

---

### <a name="plan-for-change-design-updates-are-coming-for-the-application-gallery"></a>Planeado su cambio: hay actualizaciones de diseño disponibles para la Galería de aplicaciones

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Aplicaciones empresariales SSO

Se van a realizar nuevos cambios en la interfaz de usuario en el diseño del área **Agregar desde la galería** de la hoja **Agregar una aplicación**. Estos cambios le ayudarán a encontrar más fácilmente las aplicaciones que admiten el aprovisionamiento automático, OpenID Connect, Lenguaje de marcado de aserción de seguridad (SAML) y el inicio de sesión único (SSO) con contraseña.

---

### <a name="plan-for-change-removal-of-the-mfa-server-ip-address-from-the-office-365-ip-address"></a>Planeado su cambio: Eliminación de la dirección IP del servidor MFA de la dirección IP de Office 365

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** MFA Seguridad y protección de la identidad

Vamos a quitar la dirección IP del servidor MFA del [servicio web de URL y dirección IP de Office 365](https://docs.microsoft.com/office365/enterprise/office-365-ip-web-service). Si actualmente utiliza estas páginas para actualizar la configuración del firewall, debe asegurarse de incluir también la lista de direcciones IP documentadas en la sección **requisitos del firewall del servidor Azure Multi-Factor Authentication** del artículo[Introducción a Servidor Azure Multi-Factor Authentication](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfaserver-deploy#azure-multi-factor-authentication-server-firewall-requirements).

---

### <a name="app-only-tokens-now-require-the-client-app-to-exist-in-the-resource-tenant"></a>Los tokens que solo sean de aplicación ahora requieren que la aplicación cliente exista en el inquilino de recursos

**Tipo:** **Categoría del servicio:** Corregida **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

El 26 de julio de 2019 cambiamos la forma en que proporcionamos tokens solo de aplicaciones a través [de la concesión de credenciales de cliente](https://docs.microsoft.com/azure/active-directory/develop/v1-oauth2-client-creds-grant-flow). Anteriormente, las aplicaciones podían obtener tokens para llamar a otras aplicaciones, independientemente de si la aplicación cliente se encontraba en el inquilino. Hemos actualizado este comportamiento para que a los recursos de inquilino único, a veces denominados API Web, solo puedan llamarlos las aplicaciones cliente que existen en el inquilino de recursos.

Si la aplicación no se encuentra en el inquilino de recursos, recibirá el siguiente mensaje de error: `The service principal named <app_name> was not found in the tenant named <tenant_name>. This can happen if the application has not been installed by the administrator of the tenant.` Para solucionar este problema, debe crear la entidad de servicio de la aplicación cliente en el inquilino, para lo que debe usar el [punto de conexión de consentimiento del administrador](https://docs.microsoft.com/azure/active-directory/develop/v2-permissions-and-consent#using-the-admin-consent-endpoint) o [mediante PowerShell](https://docs.microsoft.com/azure/active-directory/develop/howto-authenticate-service-principal-powershell), lo que garantiza que el inquilino ha dado a la aplicación permiso para operar dentro del inquilino.

Para más información, consulte [Novedades en la autenticación](https://docs.microsoft.com/azure/active-directory/develop/reference-breaking-changes#app-only-tokens-for-single-tenant-applications-are-only-issued-if-the-client-app-exists-in-the-resource-tenant).

> [!NOTE]
> El consentimiento existente entre el cliente y la API sigue sin ser necesario. Las aplicaciones aún deben realizar sus propias comprobaciones para saber si tienen autorización.

---

### <a name="new-passwordless-sign-in-to-azure-ad-using-fido2-security-keys"></a>Nuevo inicio de sesión sin contraseña para Azure AD mediante claves de seguridad FIDO2

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Los clientes de Azure AD ahora pueden establecer directivas para administrar las claves de seguridad FIDO2 para los usuarios y grupos de su organización. Los usuarios finales también pueden registrar automáticamente sus claves de seguridad, usar las claves para iniciar sesión con sus cuentas Microsoft en sitios web mientras estén en dispositivos compatibles con FIDO, así como iniciar sesión en sus dispositivos de Windows 10 unidos a Azure AD.

Para más información, consulte [Habilitación del inicio de sesión sin contraseña para Azure AD (versión preliminar)](/azure/active-directory/authentication/concept-authentication-passwordless) para obtener información relacionada con el administrador y [Configuración de la información de seguridad para usar una clave de seguridad (versión preliminar)](https://docs.microsoft.com/azure/active-directory/user-help/security-info-setup-security-key) para la información relacionada con el usuario final.

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---july-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (julio de 2019)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En julio de 2019, hemos agregado estas 18 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[Ungerboeck Software](https://docs.microsoft.com/azure/active-directory/saas-apps/ungerboeck-software-tutorial), [Bright Pattern Omnichannel Contact Center](https://docs.microsoft.com/azure/active-directory/saas-apps/bright-pattern-omnichannel-contact-center-tutorial), [Clever Nelly](https://docs.microsoft.com/azure/active-directory/saas-apps/clever-nelly-tutorial), [AcquireIO](https://docs.microsoft.com/azure/active-directory/saas-apps/acquireio-tutorial), [Looop](https://www.looop.co/schedule-a-demo/), [productboard](https://docs.microsoft.com/azure/active-directory/saas-apps/productboard-tutorial), [MS Azure SSO Access for Ethidex Compliance Office&trade;](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on#password-based-sso), [Hype](https://docs.microsoft.com/azure/active-directory/saas-apps/hype-tutorial), [Abstract](https://docs.microsoft.com/azure/active-directory/saas-apps/abstract-tutorial), [Ascentis](https://docs.microsoft.com/azure/active-directory/saas-apps/ascentis-tutorial), [Flipsnack](https://www.flipsnack.com/accounts/sign-in-sso.html), [Wandera](https://docs.microsoft.com/azure/active-directory/saas-apps/wandera-tutorial), [TwineSocial](https://twinesocial.com/), [Kallidus](https://docs.microsoft.com/azure/active-directory/saas-apps/kallidus-tutorial), [HyperAnna](https://docs.microsoft.com/azure/active-directory/saas-apps/hyperanna-tutorial), [PharmID WasteWitness](https://pharmid.com/), [i2B Connect](https://www.i2b-online.com/sign-up-to-use-i2b-connect-here-sso-access/), [JFrog Artifactory](https://docs.microsoft.com/azure/active-directory/saas-apps/jfrog-artifactory-tutorial)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="automate-user-account-provisioning-for-these-newly-supported-saas-apps"></a>Automatización del aprovisionamiento de cuentas de usuario para estas aplicaciones SaaS recién admitidas

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Supervisión e informes

Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

- [Dialpad](https://docs.microsoft.com/azure/active-directory/saas-apps/dialpad-provisioning-tutorial)

- [Federated Directory](https://docs.microsoft.com/azure/active-directory/saas-apps/federated-directory-provisioning-tutorial)

- [Figma](https://docs.microsoft.com/azure/active-directory/saas-apps/figma-provisioning-tutorial)

- [Leapsome](https://docs.microsoft.com/azure/active-directory/saas-apps/leapsome-provisioning-tutorial)

- [Peakon](https://docs.microsoft.com/azure/active-directory/saas-apps/peakon-provisioning-tutorial)

- [Smartsheet](https://docs.microsoft.com/azure/active-directory/saas-apps/smartsheet-provisioning-tutorial)

Para obtener más información sobre cómo proteger mejor la organización mediante el aprovisionamiento de cuentas de usuario de forma automatizada, vea [Automatización del aprovisionamiento de usuarios para aplicaciones SaaS con Azure AD](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).

---

### <a name="new-azure-ad-domain-services-service-tag-for-network-security-group"></a>Nueva etiqueta de servicio de Azure AD Domain Services para el grupo de seguridad de red

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure AD Domain Services Azure AD Domain Services

Si está cansado de administrar largas listas de rangos y direcciones IP, puede usar la nueva etiqueta de servicio de red **AzureActiveDirectoryDomainServices** en el grupo de seguridad de red de Azure para ayudar a proteger el tráfico que entra en la subred de la red virtual de Azure AD Domain Services.

Para más información acerca de esta nueva etiqueta de servicio, consulte [Grupos de seguridad de red para Azure AD Domain Services](../../active-directory-domain-services/network-considerations.md#network-security-groups-and-required-ports).

---

### <a name="new-security-audits-for-azure-ad-domain-services-public-preview"></a>Nuevas auditorías de seguridad para Azure AD Domain Services (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure AD Domain Services Azure AD Domain Services

Nos complace anunciar el lanzamiento de la auditoría de seguridad de Azure AD Domain Services en versión preliminar pública. La auditoría de seguridad le ayuda a proporcionar información crítica acerca de los servicios de autenticación mediante la transmisión en secuencias de eventos de auditoría de seguridad a recursos de destino, entre los que se incluyen Azure Storage, áreas de trabajo de Azure Log Analytics y Azure Event Hubs, mediante el portal de Azure AD Domain Services.

Para más información, consulte [Habilitación de auditorías de seguridad para Azure AD Domain Services (versión preliminar)](https://docs.microsoft.com/azure/active-directory-domain-services/security-audit-events).

---

### <a name="new-authentication-methods-usage--insights-public-preview"></a>Conclusiones y uso de los nuevos métodos de autenticación (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autoservicio de restablecimiento de contraseña Supervisión e informes

Los informes de las conclusiones y uso de los nuevos métodos de autenticación pueden ayudarle a entender cómo se registran y usan características como Azure Multi-Factor Authentication y el restablecimiento de contraseña de autoservicio en su organización, lo que incluye el número de usuarios registrados para cada característica, la frecuencia con que se usa el restablecimiento de contraseña de autoservicio para restablecer contraseñas y el método de restablecimiento.

Para más información, consulte [Conclusiones y uso de los métodos de autenticación (versión preliminar)](https://docs.microsoft.com/azure/active-directory/authentication/howto-authentication-methods-usage-insights).

---

### <a name="new-security-reports-are-available-for-all-azure-ad-administrators-public-preview"></a>Hay nuevos informes de seguridad disponibles para todos los administradores de Azure AD (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Identity Protection Seguridad y protección de la identidad

Ahora, todos los administradores de Azure ad pueden seleccionar el banner de la parte superior de los informes de seguridad existentes, como el informe **Usuarios marcados en riesgo**, para empezar a usar la nueva experiencia de seguridad, como se muestra en los informes **Usuarios de riesgo**  e **Inicios de sesión de riesgo**. Con el tiempo, todos los informes de seguridad pasarán de las versiones anteriores a las nuevas y los informes nuevos le proporcionarán las siguientes funcionalidades adicionales:

- Filtrado y ordenación avanzados

- Acciones en masa, como descartar el riesgo del usuario

- Confirmación de entidades en peligro o seguras

- Estado de riesgo, que abarca: En riesgo, Descartado, Corregido y Confirmado (en peligro)

Para más información, consulte [Informe de usuarios de riesgo](https://docs.microsoft.com/azure/active-directory/identity-protection/howto-investigate-risk#risky-users) e [Informe de inicios de sesión peligrosos](https://docs.microsoft.com/azure/active-directory/identity-protection/howto-investigate-risk#risky-sign-ins).

---

### <a name="new-security-audits-for-azure-ad-domain-services-public-preview"></a>Nuevas auditorías de seguridad para Azure AD Domain Services (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure AD Domain Services Azure AD Domain Services

Nos complace anunciar el lanzamiento de la auditoría de seguridad de Azure AD Domain Services en versión preliminar pública. La auditoría de seguridad le ayuda a proporcionar información crítica acerca de los servicios de autenticación mediante la transmisión en secuencias de eventos de auditoría de seguridad a recursos de destino, entre los que se incluyen Azure Storage, áreas de trabajo de Azure Log Analytics y Azure Event Hubs, mediante el portal de Azure AD Domain Services.

Para más información, consulte [Habilitación de auditorías de seguridad para Azure AD Domain Services (versión preliminar)](https://docs.microsoft.com/azure/active-directory-domain-services/security-audit-events).

---

### <a name="new-b2b-direct-federation-using-samlws-fed-public-preview"></a>Nueva federación directa B2B mediante SAML/WS-FED (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2B B2B/B2C

La federación directa facilita el trabajo con aquellos con asociados cuya solución de identidad administrada de tecnologías de la información no es Azure AD, ya que se trabaja con sistemas de identidad que admiten los estándares SAML o WS-Fed. Después de configurar una relación de federación directa con un asociado, todos los usuarios a los que invite desde dicho dominio pueden colaborar con usted mediante su cuenta de organización existente, lo que hace que la experiencia de sus invitados sea más fluida.

Para más información, consulte [Federación directa con AD FS y proveedores de terceros para usuarios invitados (versión preliminar)](https://docs.microsoft.com/azure/active-directory/b2b/direct-federation).

---

### <a name="automate-user-account-provisioning-for-these-newly-supported-saas-apps"></a>Automatización del aprovisionamiento de cuentas de usuario para estas aplicaciones SaaS recién admitidas

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Supervisión e informes

Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

- [Dialpad](https://docs.microsoft.com/azure/active-directory/saas-apps/dialpad-provisioning-tutorial)

- [Federated Directory](https://docs.microsoft.com/azure/active-directory/saas-apps/federated-directory-provisioning-tutorial)

- [Figma](https://docs.microsoft.com/azure/active-directory/saas-apps/figma-provisioning-tutorial)

- [Leapsome](https://docs.microsoft.com/azure/active-directory/saas-apps/leapsome-provisioning-tutorial)

- [Peakon](https://docs.microsoft.com/azure/active-directory/saas-apps/peakon-provisioning-tutorial)

- [Smartsheet](https://docs.microsoft.com/azure/active-directory/saas-apps/smartsheet-provisioning-tutorial)

Para más información acerca de cómo proteger mejor una organización mediante el aprovisionamiento automatizado de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).

---

### <a name="new-check-for-duplicate-group-names-in-the-azure-ad-portal"></a>Nueva comprobación de nombres de grupos duplicados en el portal de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración de grupos Colaboración

Ahora, al crear o actualizar un nombre de grupo desde el portal de Azure AD, realizaremos una comprobación para ver si está duplicando un nombre de grupo existente en el recurso. Si determinamos que el nombre ya lo está usando otro grupo, se le pedirá que lo modifique.

Para más información, consulte [Administración de grupos en el portal de Azure AD](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-groups-create-azure-portal?context=azure/active-directory/users-groups-roles/context/ugr-context).

---

### <a name="azure-ad-now-supports-static-query-parameters-in-reply-redirect-uris"></a>Azure AD ahora admite parámetros de consulta estáticos en las direcciones URI de respuesta (redireccionamiento)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Las aplicaciones de Azure AD ahora pueden registrar y usar identificadores URI (redireccionamiento) con parámetros de consulta estáticos (por ejemplo, `https://contoso.com/oauth2?idp=microsoft`) para las solicitudes de OAuth 2.0. El parámetro de consulta estático está sujeto a la coincidencia de cadenas en los identificadores URI de respuesta, al igual que las restantes partes del identificador URI. Si no hay ninguna cadena registrada que coincida con el identificador URI de redireccionamiento con la URL descodificada, se rechazará la solicitud. Si se encuentra el identificador URI, se usa toda la cadena para redirigir al usuario, incluido el parámetro de consulta estático.

Los identificadores URI de redireccionamiento dinámico siguen estando prohibidos, ya que representan un riesgo para la seguridad y no se pueden usar para conservar la información de estado a través de una solicitud de autenticación. Para este propósito, use el parámetro `state`.

Actualmente, las pantallas de registro de aplicaciones del Azure Portal aún bloquean los parámetros de consulta. Sin embargo, puede editar manualmente el manifiesto de la aplicación para agregar y probar los parámetros de consulta en su aplicación. Para más información, consulte [Novedades en la autenticación](https://docs.microsoft.com/azure/active-directory/develop/reference-breaking-changes#redirect-uris-can-now-contain-query-string-parameters).

---

### <a name="activity-logs-ms-graph-apis-for-azure-ad-are-now-available-through-powershell-cmdlets"></a>Los registros de actividad (MS Graph API) para Azure AD ahora están disponibles a través de los cmdlets de PowerShell

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Informes Supervisión e informes

Nos complace anunciar que los registros de actividad de Azure AD (informes de auditoría e inicio de sesión) ahora están disponibles a través del módulo de Azure AD PowerShell. Antes podía crear sus propios scripts mediante los de MS Graph API y ahora hemos ampliado esa funcionalidad a los cmdlets de PowerShell.

Para más información acerca de cómo usar estos cmdlets, consulte [Cmdlets de PowerShell de Azure AD para informes](https://docs.microsoft.com/azure/active-directory/reports-monitoring/reference-powershell-reporting).

---

### <a name="updated-filter-controls-for-audit-and-sign-in-logs-in-azure-ad"></a>Controles de filtro actualizados para los registros de auditoría e inicio de sesión en Azure AD

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto**: Informes Supervisión e informes

Hemos actualizado los informes de auditoría e inicio de sesión, por lo que ahora puede aplicar varios filtros sin tener que agregarlos como columnas en las pantallas de informes. Además, ahora puede decidir el número de filtros que desea mostrar en la pantalla. Estas actualizaciones funcionan conjuntamente para que los informes sean más fáciles de leer y se ajusten más a sus necesidades.

Para más información acerca de estas actualizaciones, consulte [Filtrado de registros de auditoría](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-audit-logs#filtering-audit-logs) y [Filtrado de las actividades de inicio de sesión](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-sign-ins#filter-sign-in-activities)de actividades de inicio de sesión.

---

## <a name="june-2019"></a>Junio de 2019

### <a name="new-riskdetections-api-for-microsoft-graph-public-preview"></a>Nueva API riskDetections de Microsoft Graph (versión preliminar)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Identity Protection Seguridad y protección de la identidad

Nos complace anunciar que la nueva API riskDetections de Microsoft Graph ya se encuentra en fase de versión preliminar pública. Puede usar esta nueva API para ver una lista de detecciones de riesgos de inicio de sesión y usuario relativos a Identity Protection de su organización. También puede utilizarla para consultar más eficazmente las detecciones de riesgos, incluidos detalles sobre el tipo de detección, el estado, el nivel y mucho más.

Para obtener más información, vea la [documentación de referencia de la API de detección de riesgos](https://docs.microsoft.com/graph/api/resources/riskdetection?view=graph-rest-beta).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---june-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (junio de 2019)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En junio de 2019, hemos agregado estas 22 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[Azure AD SAML Toolkit](https://docs.microsoft.com/azure/active-directory/saas-apps/saml-toolkit-tutorial), [Otsuka Shokai (大塚商会)](https://docs.microsoft.com/azure/active-directory/saas-apps/otsuka-shokai-tutorial), [ANAQUA](https://docs.microsoft.com/azure/active-directory/saas-apps/anaqua-tutorial), [Azure VPN Client](https://portal.azure.com/), [ExpenseIn](https://docs.microsoft.com/azure/active-directory/saas-apps/expensein-tutorial), [Helper Helper](https://docs.microsoft.com/azure/active-directory/saas-apps/helper-helper-tutorial), [Costpoint](https://docs.microsoft.com/azure/active-directory/saas-apps/costpoint-tutorial), [GlobalOne](https://docs.microsoft.com/azure/active-directory/saas-apps/globalone-tutorial), [Mercedes-Benz In-Car Office](https://me.secure.mercedes-benz.com/), [Skore](https://app.justskore.it/), [Oracle Cloud Infrastructure Console](https://docs.microsoft.com/azure/active-directory/saas-apps/oracle-cloud-tutorial), [CyberArk SAML Authentication](https://docs.microsoft.com/azure/active-directory/saas-apps/cyberark-saml-authentication-tutorial), [Scrible Edu](https://www.scrible.com/sign-in/#/create-account), [PandaDoc](https://docs.microsoft.com/azure/active-directory/saas-apps/pandadoc-tutorial), [Perceptyx](https://apexdata.azurewebsites.net/docs.microsoft.com/azure/active-directory/saas-apps/perceptyx-tutorial), [Proptimise OS](https://proptimise.co.uk/software/), [Vtiger CRM (SAML)](https://docs.microsoft.com/azure/active-directory/saas-apps/vtiger-crm-saml-tutorial), Oracle Access Manager for Oracle Retail Merchandising, Oracle Access Manager for Oracle E-Business Suite, Oracle IDCS for E-Business Suite, Oracle IDCS for PeopleSoft y Oracle IDCS for JD Edwards

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="automate-user-account-provisioning-for-these-newly-supported-saas-apps"></a>Automatización del aprovisionamiento de cuentas de usuario para estas aplicaciones SaaS recién admitidas

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Supervisión e informes

Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

- [Zoom](https://docs.microsoft.com/azure/active-directory/saas-apps/zoom-provisioning-tutorial)

- [Envoy](https://docs.microsoft.com/azure/active-directory/saas-apps/envoy-provisioning-tutorial)

- [Proxyclick](https://docs.microsoft.com/azure/active-directory/saas-apps/proxyclick-provisioning-tutorial)

- [4me](https://docs.microsoft.com/azure/active-directory/saas-apps/4me-provisioning-tutorial)

Para obtener más información sobre cómo proteger mejor la organización mediante el aprovisionamiento de cuentas de usuario de forma automatizada, vea [Automatización del aprovisionamiento de usuarios para aplicaciones SaaS con Azure AD](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).

---

### <a name="view-the-real-time-progress-of-the-azure-ad-provisioning-service"></a>Consulta del progreso en tiempo real del servicio de aprovisionamiento de Azure AD

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Aprovisionamiento de aplicaciones Administración del ciclo de vida de la identidad

Hemos actualizado la experiencia de aprovisionamiento de Azure AD para incluir una nueva barra de progreso que muestra a qué altura está el proceso de aprovisionamiento de usuario. Esta experiencia mejorada también proporciona información sobre el número de usuarios aprovisionados durante el ciclo actual, así como los usuarios aprovisionados hasta la fecha.

Para obtener más información, vea [Comprobación del estado de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user).

---

### <a name="company-branding-now-appears-on-sign-out-and-error-screens"></a>Ahora, la personalización de marca aparece en las pantallas de cierre de sesión y de error

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Hemos actualizado Azure AD para que, ahora, la personalización de marca de su empresa aparezca en las pantallas de cierre de sesión y de error, así como en la página de inicio de sesión. Para activar esta característica no es preciso hacer nada, Azure AD usa los recursos que ya están configurados en el área **Personalización de marca de empresa** de Azure Portal.

Para obtener más información sobre cómo configurar la personalización de marca de su empresa, vea [Incorporación de la personalización de marca en las páginas de Azure Active Directory de la organización](https://docs.microsoft.com/azure/active-directory/fundamentals/customize-branding).

---

### <a name="azure-multi-factor-authentication-mfa-server-is-no-longer-available-for-new-deployments"></a>El Servidor Azure Multi-Factor Authentication (MFA) ya no está disponible para nuevas implementaciones

**Tipo:** **Categoría del servicio:** En desuso **Funcionalidad del producto:** MFA Seguridad y protección de la identidad

A partir del 1 de julio de 2019, Microsoft ya no ofrecerá el Servidor MFA para nuevas implementaciones. Ahora, los clientes nuevos que quieran exigir la autenticación multifactor en su organización deberán usar Azure Multi-factor Authentication en la nube. Los clientes que tengan activado el Servidor MFA desde antes del 1 de julio no apreciarán ningún cambio. Podrán seguir descargando las versiones más recientes, obteniendo actualizaciones futuras y generando credenciales de activación.

Para obtener más información, vea [Introducción al Servidor Azure Multi-Factor Authentication](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfaserver-deploy). Para obtener más información sobre Azure Multi-Factor Authentication en la nube, vea [Planeación de una implementación de Azure Multi-Factor Authentication en la nube](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted).

---

## <a name="may-2019"></a>Mayo de 2019

### <a name="service-change-future-support-for-only-tls-12-protocols-on-the-application-proxy-service"></a>Cambio de servicio: próxima compatibilidad únicamente con protocolos TLS 1.2 en el servicio Application Proxy

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Proxy de aplicaciones Control de acceso

A fin de proporcionar el mejor cifrado a nuestros clientes, estamos limitando el acceso a únicamente protocolos TLS 1.2 en el servicio Application Proxy. Este cambio se está implantando gradualmente en clientes que ya usan protocolos TLS 1.2 exclusivamente, por lo que no deberían apreciar cambio alguno.

La obsolescencia de TLS 1.0 y TLS 1.1 tendrá lugar el 31 de agosto de 2019, pero avisaremos con antelación para que tenga tiempo para prepararse para este cambio. Para prepararse para este cambio, asegúrese de que sus combinaciones de servidor cliente y servidor de examinador (incluidos los clientes que los usuarios usan para tener acceso a aplicaciones publicadas a través de Application Proxy) se actualizan para usar el protocolo TLS 1.2 y, así, mantengan la conexión con el servicio Application Proxy. Para obtener más información, vea [Adición de una aplicación local para el acceso remoto mediante el proxy de aplicación en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-add-on-premises-application#before-you-begin).

---

### <a name="use-the-usage-and-insights-report-to-view-your-app-related-sign-in-data"></a>Utilización del informe de uso y conclusiones para ver datos de inicio de sesión relativos a las aplicaciones

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Supervisión e informes

Ahora, puede usar el informe de uso y conclusiones, ubicado en el área **Aplicaciones empresariales** de Azure Portal, para obtener una vista de los datos de inicio de sesión relativos a las aplicaciones, con información sobre lo siguiente:

- Aplicaciones más usadas en la organización

- Aplicaciones con los inicios de sesión con más errores

- Principales errores de inicio de sesión de cada aplicación

Para obtener más información sobre esta característica, vea [Informe de uso y conclusiones en el portal de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-usage-insights-report).

---

### <a name="automate-your-user-provisioning-to-cloud-apps-using-azure-ad"></a>Automatización del aprovisionamiento de usuarios a aplicaciones en la nube mediante Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Supervisión e informes

Siga estos nuevos tutoriales para utilizar el servicio de aprovisionamiento de Azure AD para automatizar la creación, eliminación y actualización de cuentas de usuario de las siguientes aplicaciones en la nube:

- [Comeet](https://docs.microsoft.com/azure/active-directory/saas-apps/comeet-recruiting-software-provisioning-tutorial)

- [DynamicSignal](https://docs.microsoft.com/azure/active-directory/saas-apps/dynamic-signal-provisioning-tutorial)

- [KeeperSecurity](https://docs.microsoft.com/azure/active-directory/saas-apps/keeper-password-manager-digitalvault-provisioning-tutorial)

También puede seguir este nuevo [tutorial de Dropbox](https://docs.microsoft.com/azure/active-directory/saas-apps/dropboxforbusiness-provisioning-tutorial), que proporciona información sobre cómo aprovisionar objetos de grupo.

Para obtener más información sobre cómo proteger mejor la organización a través del aprovisionamiento automatizado de cuentas de usuario, vea [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).

---

### <a name="identity-secure-score-is-now-available-in-azure-ad-general-availability"></a>La puntuación segura de identidad ya está disponible en Azure AD (disponibilidad general)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** N/D Seguridad y protección de la identidad

Ahora, puede supervisar y mejorar su posición de seguridad de identidad gracias a la característica de puntuación segura de identidad de Azure AD. La característica de puntuación segura de identidad utiliza un mismo panel para ayudarle a:

- Medir de forma objetiva su nivel de seguridad de la identidad, según una puntuación de entre 1 y 223.

- Planear la realización de mejoras en la seguridad de la identidad.

- Ver si las mejoras han logrado sus objetivos de seguridad.

Para obtener más información sobre la característica de puntuación segura de identidad, vea [¿Qué es la puntuación segura de identidad en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/fundamentals/identity-secure-score)

---

### <a name="new-app-registrations-experience-is-now-available-general-availability"></a>Nueva experiencia Registros de aplicaciones ya disponible (disponibilidad general)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Experiencia para el desarrollador

La nueva experiencia [Registros de aplicaciones](https://aka.ms/appregistrations) ya está disponible en versión de disponibilidad general. Esta nueva experiencia incluye todas las características clave a las que está acostumbrado de Azure Portal y del portal de Registros de aplicaciones, y las mejora a través de lo siguiente:

- **Una mejor administración de las aplicaciones.** En lugar de ver las aplicaciones en diversos portales, ahora puede verlas todas en una misma ubicación.

- **Un registro de aplicaciones simplificado.** Desde la experiencia de navegación mejorada a la experiencia de selección de permisos renovada, ahora es más fácil registrar y administrar sus aplicaciones.

- **Una información más pormenorizada.** Encontrará más detalles sobre la aplicación, incluidas guías de inicio rápido y otras muchas cosas.

Para obtener más información, vea [Plataforma de identidad de Microsoft](https://docs.microsoft.com/azure/active-directory/develop/) y la entrada de blog que anuncia que la [experiencia Registros de aplicación ya está disponible de forma general](https://developer.microsoft.com/identity/blogs/new-app-registrations-experience-is-now-generally-available/) .

---

### <a name="new-capabilities-available-in-the-risky-users-api-for-identity-protection"></a>Nuevas capacidades disponibles en la API correspondiente a los usuarios de riesgo de Identity Protection

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Identity Protection Seguridad y protección de la identidad

Nos complace anunciar que, ahora, puede usar la API correspondiente a los usuarios de riesgo para recuperar el historial de riesgos de los usuarios, descartar usuarios de riesgo y confirmar usuarios como usuarios de riesgo. Este cambio ayuda a actualizar el estado de riesgo de los usuarios de forma más eficaz y a entender sus historiales de riesgos.

Para obtener más información, vea la [documentación de referencia de la API correspondiente a los usuarios de riesgo](https://docs.microsoft.com/graph/api/resources/riskyuser?view=graph-rest-beta).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---may-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (mayo de 2019)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En mayo de 2019, hemos agregado estas 21 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[Freedcamp](https://docs.microsoft.com/azure/active-directory/saas-apps/freedcamp-tutorial), [Real Links](https://docs.microsoft.com/azure/active-directory/saas-apps/real-links-tutorial), [Kianda](https://app.kianda.com/sso/OpenID/AzureAD/), [Simple Sign](https://docs.microsoft.com/azure/active-directory/saas-apps/simple-sign-tutorial), [Braze](https://docs.microsoft.com/azure/active-directory/saas-apps/braze-tutorial), [Displayr](https://docs.microsoft.com/azure/active-directory/saas-apps/displayr-tutorial), [Templafy](https://docs.microsoft.com/azure/active-directory/saas-apps/templafy-tutorial), [Marketo Sales Engage](https://toutapp.com/login), [ACLP](https://docs.microsoft.com/azure/active-directory/saas-apps/aclp-tutorial), [OutSystems](https://docs.microsoft.com/azure/active-directory/saas-apps/outsystems-tutorial), [Meta4 Global HR](https://docs.microsoft.com/azure/active-directory/saas-apps/meta4-global-hr-tutorial), [Quantum Workplace](https://docs.microsoft.com/azure/active-directory/saas-apps/quantum-workplace-tutorial), [Cobalt](https://docs.microsoft.com/azure/active-directory/saas-apps/cobalt-tutorial), [webMethods API Cloud](https://docs.microsoft.com/azure/active-directory/saas-apps/webmethods-integration-cloud-tutorial), [RedFlag](https://pocketstop.com/redflag/), [Whatfix](https://docs.microsoft.com/azure/active-directory/saas-apps/whatfix-tutorial), [Control](https://docs.microsoft.com/azure/active-directory/saas-apps/control-tutorial), [JOBHUB](https://docs.microsoft.com/azure/active-directory/saas-apps/jobhub-tutorial), [NEOGOV](https://docs.microsoft.com/azure/active-directory/saas-apps/neogov-tutorial), [Foodee](https://docs.microsoft.com/azure/active-directory/saas-apps/foodee-tutorial) y [MyVR](https://docs.microsoft.com/azure/active-directory/saas-apps/myvr-tutorial)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="improved-groups-creation-and-management-experiences-in-the-azure-ad-portal"></a>Experiencias mejoradas de administración y creación de grupos en el portal de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración de grupos Colaboración

Hemos realizado mejoras en las experiencias relativas a grupos del portal de Azure AD. Estas mejoras permiten a los administradores administrar de mejor forma las listas de grupos y las listas de miembros, así como proporcionar más opciones de creación.

Estas mejoras incluyen:

- Filtrado básico por tipo de pertenencia y tipo de grupo.

- Incorporación de columnas nuevas, como Origen y Dirección de correo electrónico.

- Posibilidad de seleccionar varios grupos, miembros y listas de propietarios para eliminarlos con facilidad.

- Posibilidad de elegir una dirección de correo electrónico y de agregar propietarios durante la creación del grupo.

Para obtener más información vea [Creación de un grupo básico e incorporación de miembros con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-groups-create-azure-portal).

---

### <a name="configure-a-naming-policy-for-office-365-groups-in-azure-ad-portal-general-availability"></a>Configuración de una directiva de nomenclatura para grupos de Office 365 en el portal de Azure AD (disponibilidad general)

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Administración de grupos Colaboración

Ahora, los administradores pueden configurar una directiva de nomenclatura de grupos de Office 365 mediante el portal de Azure AD. Este cambio ayuda a aplicar las convenciones de nomenclatura coherentes para los grupos de Office 365 creados o editados por los usuarios de su organización.

La directiva de nomenclatura de grupos de Office 365 se puede configurar de dos maneras diferentes:

- Definir prefijos o sufijos, que se agregan automáticamente al nombre de un grupo.

- Cargar un conjunto personalizado de palabras bloqueadas de la organización que no se permiten en los nombres de grupo (por ejemplo, "CEO, nómina, RR. HH.").

Para obtener más información, vea [Aplicación de una directiva de nomenclatura en los grupos de Office 365](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-naming-policy).

---

### <a name="microsoft-graph-api-endpoints-are-now-available-for-azure-ad-activity-logs-general-availability"></a>Ahora, los puntos de conexión de la API de Microsoft Graph están disponibles para los registros de actividad de Azure AD (disponibilidad general)

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto**: Informes Supervisión e informes

Nos complace anunciar la disponibilidad general de compatibilidad de puntos de conexión de la API de Microsoft Graph en los registros de actividad de Azure AD. Con este lanzamiento, ahora puede usar la versión 1.0 tanto de los registros de auditoría de Azure AD como de las API de registros de inicio de sesión.

Para obtener más información, vea [Información general sobre la API de registros de auditoría de Azure AD](https://docs.microsoft.com/graph/api/resources/azure-ad-auditlog-overview?view=graph-rest-1.0).

---

### <a name="administrators-can-now-use-conditional-access-for-the-combined-registration-process-public-preview"></a>Ahora, los administradores pueden usar el acceso condicional en el proceso de registro combinado (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Acceso condicional Seguridad y protección de la identidad

Ahora, los administradores pueden crear directivas de acceso condicional para su uso en la página de registro combinado. Esto incluye la puesta en marcha de directivas que permiten el registro bajo estos supuestos:

- Los usuarios están en una red de confianza.

- Los usuarios presentan un bajo riesgo de inicio de sesión.

- Los usuarios están en un dispositivo administrado.

- Los usuarios han aceptado las condiciones de uso de la organización.

Para obtener más información sobre el acceso condicional y el restablecimiento de contraseña, puede ver la entrada de blog sobre el [acceso condicional en la experiencia de registro combinado de restablecimiento de contraseña y MFA de Azure AD](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Conditional-access-for-the-Azure-AD-combined-MFA-and-password/ba-p/566348). Para obtener más información sobre las directivas de acceso condicional en el proceso de registro combinado, vea [Directivas de acceso condicional para el registro combinado](https://docs.microsoft.com/azure/active-directory/authentication/howto-registration-mfa-sspr-combined#conditional-access-policies-for-combined-registration). Para obtener más información sobre la característica Condiciones de uso de Azure AD, vea [Característica Condiciones de uso de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/conditional-access/terms-of-use).

---

## <a name="april-2019"></a>Abril de 2019

### <a name="new-azure-ad-threat-intelligence-detection-is-now-available-as-part-of-azure-ad-identity-protection"></a>Nueva detección de la inteligencia sobre amenazas de Azure AD ya disponible como parte de Azure AD Identity Protection

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure AD Identity Protection Seguridad y protección de la identidad

La detección de la inteligencia sobre amenazas de Azure AD ya está disponible como parte de la característica actualizada Azure AD Identity Protection. Esta nueva funcionalidad ayuda a identificar una actividad de usuario poco común para el usuario en cuestión o una actividad que encaja con patrones de ataque conocidos según los orígenes de la inteligencia sobre amenazas internas y externas de Microsoft.

Para obtener más información sobre la versión actualizada de Azure AD Identity Protection, vea la entrada de blog sobre las [cuatro grandes mejoras de Azure AD Identity Protection que ya están en fase de versión preliminar pública](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Four-major-Azure-AD-Identity-Protection-enhancements-are-now-in/ba-p/326935) y el artículo [¿Qué es Azure Active Directory Identity Protection (actualizado)?](https://docs.microsoft.com/azure/active-directory/identity-protection/overview-v2) . Para obtener más información sobre la detección de inteligencia sobre amenazas de Azure AD, vea el artículo [Referencia sobre eventos de riesgo de Azure Active Directory Identity Protection](https://docs.microsoft.com/azure/active-directory/identity-protection/concept-identity-protection-risks).

---

### <a name="azure-ad-entitlement-management-is-now-available-public-preview"></a>La administración de derechos de Azure AD ya está disponible (versión preliminar)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Identity Governance Identity Governance

La administración de derechos de Azure AD, ahora en versión preliminar pública, permite a los clientes delegar la administración de paquetes de acceso, que define cómo los empleados y los asociados comerciales pueden solicitar acceso, quién debe aprobar ese acceso y cuánto va a durar ese acceso. Los paquetes de acceso pueden administrar la pertenencia a grupos de Azure AD y Office 365, las asignaciones de roles en aplicaciones empresariales y las asignaciones de roles para sitios de SharePoint Online. Encontrará más información sobre la administración de derechos en el artículo de [información general sobre la administración de derechos de Azure AD](https://docs.microsoft.com/azure/active-directory/governance/entitlement-management-overview). Para obtener más información sobre el amplio abanico de características de Azure AD Identity Governance, incluidos Privileged Identity Management, revisiones del acceso y las condiciones de uso, vea [¿Qué es Azure AD Identity Governance?](../governance/identity-governance-overview.md)

---

### <a name="configure-a-naming-policy-for-office-365-groups-in-azure-ad-portal-public-preview"></a>Configuración de una directiva de nomenclatura para grupos de Office 365 en el portal de Azure AD (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración de grupos Colaboración

Ahora, los administradores pueden configurar una directiva de nomenclatura de grupos de Office 365 mediante el portal de Azure AD. Este cambio ayuda a aplicar las convenciones de nomenclatura coherentes para los grupos de Office 365 creados o editados por los usuarios de su organización.

La directiva de nomenclatura de grupos de Office 365 se puede configurar de dos maneras diferentes:

- Definir prefijos o sufijos, que se agregan automáticamente al nombre de un grupo.

- Cargar un conjunto personalizado de palabras bloqueadas de la organización que no se permiten en los nombres de grupo (por ejemplo, "CEO, nómina, RR. HH.").

Para obtener más información, vea [Aplicación de una directiva de nomenclatura en los grupos de Office 365](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-naming-policy).

---

### <a name="azure-ad-activity-logs-are-now-available-in-azure-monitor-general-availability"></a>Los registros de actividad de Azure AD ya están disponibles en Azure Monitor (disponibilidad general)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Informes Supervisión e informes

Para que sea más fácil atender sus comentarios sobre las visualizaciones de los registros de actividad de Azure AD, hemos incluido una nueva característica Información en Log Analytics. Esta característica sirve para obtener información sobre los recursos de Azure AD mediante nuestras plantillas interactivas, llamadas libros. Estos libros pregenerados proporcionan detalles sobre las aplicaciones o los usuarios, como lo siguiente:

- **Inicios de sesión** Proporciona detalles sobre las aplicaciones y los usuarios, incluida la ubicación de inicio de sesión, el sistema operativo o el cliente y versión del explorador que se usa y el número de inicios de sesión correctos o con errores.

- **Autenticación heredada y acceso condicional** Proporciona detalles sobre las aplicaciones y los usuarios que usan la autenticación heredada, incluido el uso de Multi-Factor Authentication desencadenado por las directivas de acceso condicional, las aplicaciones que usan directivas de acceso condicional, etc.

- **Análisis de errores de inicio de sesión** Ayuda a averiguar si los errores de inicio de sesión se deben a una acción del usuario, a problemas de directiva o a la infraestructura existente.

- **Informes personalizados** Puede crear libros o editar los ya existentes para ayudar a personalizar la característica Información de su organización.

Para obtener más información, vea [Cómo usar los libros de Azure Monitor en informes de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-use-azure-monitor-workbooks).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---april-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (abril de 2019)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En abril de 2019, hemos agregado estas 21 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[SAP Fiori](https://docs.microsoft.com/azure/active-directory/saas-apps/sap-fiori-tutorial), [HRworks Single Sign-On](https://docs.microsoft.com/azure/active-directory/saas-apps/hrworks-single-sign-on-tutorial), [Percolate](https://docs.microsoft.com/azure/active-directory/saas-apps/percolate-tutorial), [MobiControl](https://docs.microsoft.com/azure/active-directory/saas-apps/mobicontrol-tutorial), [Citrix NetScaler](https://docs.microsoft.com/azure/active-directory/saas-apps/citrix-netscaler-tutorial), [Shibumi](https://docs.microsoft.com/azure/active-directory/saas-apps/shibumi-tutorial), [Benchling](https://docs.microsoft.com/azure/active-directory/saas-apps/benchling-tutorial), [MileIQ](https://mileiq.onelink.me/991934284/7e980085), [PageDNA](https://docs.microsoft.com/azure/active-directory/saas-apps/pagedna-tutorial), [EduBrite LMS](https://docs.microsoft.com/azure/active-directory/saas-apps/edubrite-lms-tutorial), [RStudio Connect](https://docs.microsoft.com/azure/active-directory/saas-apps/rstudio-connect-tutorial), [AMMS](https://docs.microsoft.com/azure/active-directory/saas-apps/amms-tutorial), [Mitel Connect](https://docs.microsoft.com/azure/active-directory/saas-apps/mitel-connect-tutorial), [Alibaba Cloud (Role-based SSO)](https://docs.microsoft.com/azure/active-directory/saas-apps/alibaba-cloud-service-role-based-sso-tutorial), [Certent Equity Management](https://docs.microsoft.com/azure/active-directory/saas-apps/certent-equity-management-tutorial), [Sectigo Certificate Manager](https://docs.microsoft.com/azure/active-directory/saas-apps/sectigo-certificate-manager-tutorial), [GreenOrbit](https://docs.microsoft.com/azure/active-directory/saas-apps/greenorbit-tutorial), [Workgrid](https://docs.microsoft.com/azure/active-directory/saas-apps/workgrid-tutorial), [monday.com](https://docs.microsoft.com/azure/active-directory/saas-apps/mondaycom-tutorial), [SurveyMonkey Enterprise](https://docs.microsoft.com/azure/active-directory/saas-apps/surveymonkey-enterprise-tutorial) e [Indiggo](https://indiggolead.com/)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="new-access-reviews-frequency-option-and-multiple-role-selection"></a>Nueva opción de frecuencia de revisiones de acceso y selección de varios roles

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Revisiones de acceso Identity Governance

Las nuevas actualizaciones de las revisiones de acceso de Azure AD permiten lo siguiente:

- Cambiar la frecuencia de las revisiones de acceso a **Semestral**, además de las opciones ya existentes (Semanal, Mensual, Trimestral y Anual).

- Seleccionar varios roles de recursos de Azure y de Azure AD al crear una revisión de acceso único. En esta situación, todos los roles se configuran igual y se notifica a todos los revisores al mismo tiempo.

Para obtener más información sobre cómo crear una revisión de acceso, vea [Creación de una revisión de acceso de los grupos o las aplicaciones en las revisiones de acceso de Azure AD](https://docs.microsoft.com/azure/active-directory/governance/create-access-review).

---

### <a name="azure-ad-connect-email-alert-systems-are-transitioning-sending-new-email-sender-information-for-some-customers"></a>Transición de los sistemas de alertas por correo electrónico de Azure AD Connect en curso; se está enviando información de nuevo remitente de correo electrónico a algunos clientes

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Sincronización de AD Plataforma

Azure AD Connect está en fase de transición de nuestros sistemas de alertas de correo electrónico, lo que puede hacer que algunos clientes vean un nuevo remitente de correo electrónico. Para solucionar este problema, hay que agregar `azure-noreply@microsoft.com` a la lista de permitidos de la organización, o no podrá seguir recibiendo alertas importantes de los servicios de Office 365, Azure o Sync.

---

### <a name="upn-suffix-changes-are-now-successful-between-federated-domains-in-azure-ad-connect"></a>Los cambios en los sufijos UPN ya se realizan correctamente entre dominios federados en Azure AD Connect

**Tipo:** **Categoría del servicio:** Corregida **Funcionalidad del producto:** Sincronización de AD Plataforma

Ahora puede cambiar correctamente el sufijo UPN de un usuario de un dominio federado a otro en Azure AD Connect. Esta corrección implica que ya no debería aparecer el mensaje de error FederatedDomainChangeError durante el ciclo de sincronización, ni recibir una notificación por correo electrónico que informa de que el objeto no se puede actualizar en Azure Active Directory porque el atributo [FederatedUser.UserPrincipalName] no es válido y pide actualizar el valor en los servicios de directorio local.

Para obtener más información, vea [Solución de errores durante la sincronización](https://docs.microsoft.com/azure/active-directory/hybrid/tshoot-connect-sync-errors#federateddomainchangeerror).

---

### <a name="increased-security-using-the-app-protection-based-conditional-access-policy-in-azure-ad-public-preview"></a>Mayor seguridad con la directiva de acceso condicional basada en la protección de aplicaciones en Azure AD (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Acceso condicional Seguridad y protección de la identidad

Ahora, el acceso condicional basado en la protección de aplicaciones está disponible a través de la directiva **Requerir protección de aplicaciones**. Esta nueva directiva ayuda a aumentar la seguridad de la organización al tratar de impedir que:

- Los usuarios obtengan acceso a las aplicaciones sin contar con una licencia de Microsoft Intune.

- Los usuarios no puedan obtener una directiva de protección de aplicaciones de Microsoft Intune.

- Los usuarios obtengan acceso a las aplicaciones sin una directiva de protección de aplicaciones configurada de Microsoft Intune.

Para obtener más información, vea [Cómo usar la aplicación Requerir protección de aplicaciones para el acceso a aplicaciones en la nube mediante el acceso condicional](https://docs.microsoft.com/azure/active-directory/conditional-access/app-protection-based-conditional-access).

---

### <a name="new-support-for-azure-ad-single-sign-on-and-conditional-access-in-microsoft-edge-public-preview"></a>Nueva compatibilidad con el acceso condicional y el inicio de sesión único de Azure AD en Microsoft Edge (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Acceso condicional Seguridad y protección de la identidad

Hemos mejorado la compatibilidad de Azure AD para Microsoft Edge, incluido proporcionar nueva compatibilidad con el inicio de sesión único en Azure AD y el acceso condicional. Si ya usó previamente Microsoft Intune Managed Browser, ahora puede usar Microsoft Edge en su lugar.

Para obtener más información sobre cómo configurar y administrar los dispositivos y aplicaciones mediante el acceso condicional, vea [Requerir dispositivos administrados para el acceso a aplicaciones en la nube con el acceso condicional](https://docs.microsoft.com/azure/active-directory/conditional-access/require-managed-devices) y [Requerir aplicaciones de cliente aprobadas para el acceso a aplicaciones en la nube con el acceso condicional](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access). Para obtener más información sobre cómo administrar el acceso mediante Microsoft Edge con directivas de Microsoft Intune, vea [Administración del acceso web con un explorador protegido por directiva de Microsoft Intune](https://docs.microsoft.com/intune/app-configuration-managed-browser).

---

## <a name="march-2019"></a>Marzo de 2019

### <a name="identity-experience-framework-and-custom-policy-support-in-azure-active-directory-b2c-is-now-available-ga"></a>Identity Experience Framework y las directivas personalizadas ya están disponibles en Azure Active Directory B2C (GA)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Ahora es posible crear directivas personalizadas en Azure AD B2C, incluidas las siguientes tareas, que se admiten a escala y bajo nuestro SLA de Azure:

- Crear y cargar trayectos de autenticación personalizada de los usuarios mediante directivas personalizadas.

- Describir los recorridos del usuario paso a paso como intercambios entre proveedores de notificaciones.

- Definir la creación de ramas condicional en recorridos del usuario.

- Transformar y asignar notificaciones para su uso en las comunicaciones y las decisiones en tiempo real.

- Usar servicios habilitados por la API de REST en los trayectos de autenticación personalizada de los usuarios. Por ejemplo, con proveedores de correo electrónico, CRM y sistemas de autorización propietarios.

- Federarse con proveedores de identidades que cumplen con el protocolo OpenIDConnect. Por ejemplo, con Azure AD multiinquilino, proveedores de cuentas de redes sociales o proveedores de verificación de dos factores.

Para más información sobre la creación de directivas personalizadas, vea [Notas para desarrolladores de directivas personalizadas en Azure Active Directory B2C](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-developer-notes-custom) y lea la [entrada de blog de Alex Simon, que incluye casos prácticos](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-B2C-custom-policies-to-build-your-own-identity-journeys/ba-p/382791).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---march-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (marzo de 2019)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En marzo de 2019, hemos agregado estas 14 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[ISEC7 Mobile Exchange Delegate](https://www.isec7.com/english/), [MediusFlow](https://office365.cloudapp.mediusflow.com/), [ePlatform](https://docs.microsoft.com/azure/active-directory/saas-apps/eplatform-tutorial), [Fulcrum](https://docs.microsoft.com/azure/active-directory/saas-apps/fulcrum-tutorial), [ExcelityGlobal](https://docs.microsoft.com/azure/active-directory/saas-apps/excelityglobal-tutorial), [Explanation-Based Auditing System](https://docs.microsoft.com/azure/active-directory/saas-apps/explanation-based-auditing-system-tutorial), [Lean](https://docs.microsoft.com/azure/active-directory/saas-apps/lean-tutorial), [Powerschool Performance Matters](https://docs.microsoft.com/azure/active-directory/saas-apps/powerschool-performance-matters-tutorial), [Cinode](https://cinode.com/), [Iris Intranet](https://docs.microsoft.com/azure/active-directory/saas-apps/iris-intranet-tutorial), [Empactis](https://docs.microsoft.com/azure/active-directory/saas-apps/empactis-tutorial), [SmartDraw](https://docs.microsoft.com/azure/active-directory/saas-apps/smartdraw-tutorial), [Confirmit Horizons](https://docs.microsoft.com/azure/active-directory/saas-apps/confirmit-horizons-tutorial) y [TAS](https://docs.microsoft.com/azure/active-directory/saas-apps/tas-tutorial)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="new-zscaler-and-atlassian-provisioning-connectors-in-the-azure-ad-gallery---march-2019"></a>Nuevos conectores de aprovisionamiento de Zscaler y Atlassian en la galería Azure AD (marzo de 2019)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aprovisionamiento de aplicaciones Integración de terceros

Automatice la creación, la actualización y la eliminación de cuentas de usuario para las siguientes aplicaciones:

[Zscaler](https://aka.ms/ZscalerProvisioning), [Zscaler Beta](https://aka.ms/ZscalerBetaProvisioning), [Zscaler One](https://aka.ms/ZscalerOneProvisioning), [Zscaler Two](https://aka.ms/ZscalerTwoProvisioning), [Zscaler Three](https://aka.ms/ZscalerThreeProvisioning), [Zscaler ZSCloud](https://aka.ms/ZscalerZSCloudProvisioning) y [Atlassian Cloud](https://aka.ms/atlassianCloudProvisioning)

Para obtener más información sobre cómo proteger mejor la organización a través del aprovisionamiento automatizado de cuentas de usuario, vea [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](https://aka.ms/ProvisioningDocumentation).

---

### <a name="restore-and-manage-your-deleted-office-365-groups-in-the-azure-ad-portal"></a>Restaurar y administrar los grupos de Office 365 eliminados en el portal de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración de grupos Colaboración

Ahora es posible ver y administrar los grupos de Office 365 eliminados desde el portal de Azure AD. Este cambio ayuda a ver qué grupos se pueden restaurar, además de permitir eliminar de forma permanente los grupos que la organización no necesite.

Para obtener más información, vea [Restauración de grupos expirados o eliminados](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-restore-deleted#view-and-manage-the-deleted-office-365-groups-that-are-available-to-restore).

---

### <a name="single-sign-on-is-now-available-for-azure-ad-saml-secured-on-premises-apps-through-application-proxy-public-preview"></a>El inicio de sesión único está ahora disponible para aplicaciones locales protegidas por SAML de Azure AD a través del proxy de la aplicación (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Proxy de aplicaciones Control de acceso

Ahora se puede proporcionar una experiencia de inicio de sesión único (SSO) en aplicaciones locales con autenticación SAML, junto con acceso remoto a estas aplicaciones a través del proxy de la aplicación. Para obtener más información sobre cómo configurar el SSO de SAML con las aplicaciones locales, vea [Inicio de sesión único de SAML para aplicaciones locales con proxy de aplicación (versión preliminar)](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-configure-single-sign-on-on-premises-apps).

---

### <a name="client-apps-in-request-loops-will-be-interrupted-to-improve-reliability-and-user-experience"></a>Se interrumpirán las aplicaciones cliente en bucles de solicitud para mejorar la confiabilidad y la experiencia del usuario

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Las aplicaciones cliente pueden emitir incorrectamente cientos de solicitudes del mismo inicio de sesión durante un breve período de tiempo. Todas estas solicitudes, aprobadas o no, contribuyen a una experiencia de usuario deficiente y a mayores cargas de trabajo para el IDP, lo que aumenta la latencia de todos los usuarios y reduce la disponibilidad del IDP.

Esta actualización envía el error `invalid_grant`: `AADSTS50196: The server terminated an operation because it encountered a loop while processing a request` a las aplicaciones cliente que emiten solicitudes duplicadas varias veces en un breve período de tiempo, más allá del ámbito de operación normal. Las aplicaciones cliente que experimentan este problema deben mostrar un mensaje interactivo que requiera que el usuario vuelva a iniciar sesión. Para obtener más información sobre este cambio y sobre cómo corregir la aplicación si se encuentra con este error, vea [Novedades en la autenticación](https://docs.microsoft.com/azure/active-directory/develop/reference-breaking-changes#looping-clients-will-be-interrupted).

---

### <a name="new-audit-logs-user-experience-now-available"></a>Ya disponible la nueva experiencia de usuario de los registros de auditoría

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto**: Informes Supervisión e informes

Hemos creado una nueva página **Registros de auditoría** de Azure AD para mejorar la legibilidad y la búsqueda de información. Para ver la página **Registros de auditoría**, seleccione **Registros de auditoría** en la sección **Actividad** de Azure AD.

![Nueva página Registros de auditoría con información de ejemplo](media/whats-new/audit-logs-page.png)

Para obtener más información sobre la página **Registros de auditoría**, vea [Informes de actividad de auditoría en el portal de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-audit-logs#audit-logs).

---

### <a name="new-warnings-and-guidance-to-help-prevent-accidental-administrator-lockout-from-misconfigured-conditional-access-policies"></a>Nuevas advertencias e instrucciones para ayudar a evitar el bloqueo accidental del administrador por una mala configuración de las directivas de acceso condicional

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Acceso condicional Seguridad y protección de la identidad

Para evitar que los administradores se bloqueen accidentalmente a sí mismos en sus propios inquilinos a causa de una mala configuración de las directivas de acceso, hemos creado nuevas advertencias y hemos actualizado las instrucciones en Azure Portal. Para obtener más información sobre las nuevas instrucciones, vea [¿Cuáles son las dependencias del servicio de acceso condicional de Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/service-dependencies).

---

### <a name="improved-end-user-terms-of-use-experiences-on-mobile-devices"></a>Experiencia mejorada de las condiciones de uso en dispositivos móviles

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Términos de uso Gobernanza

Hemos actualizado y mejorado el modo en el que los usuarios leen y aceptan las condiciones de uso en un dispositivo móvil. Ahora es posible ampliar y reducir el texto, volver atrás, descargar la información y seleccionar hipervínculos. Para obtener más información sobre las nuevas condiciones de uso, vea [Característica Condiciones de uso de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/conditional-access/terms-of-use#what-terms-of-use-looks-like-for-users).

---

### <a name="new-azure-ad-activity-logs-download-experience-available"></a>Disponible la nueva experiencia de descarga de los registros de actividad de Azure AD

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto**: Informes Supervisión e informes

Ahora se pueden descargar grandes cantidades de registros de actividad directamente desde Azure Portal. Este cambio le permite:

- Descargar hasta 250 000 filas.

- Recibir una notificación cuando se complete la descarga.

- Personalizar el nombre del archivo.

- Determinar el formato de salida como JSON o CSV.

Para más información acerca de esta característica, consulte [Inicio rápido: Descarga de un informe de auditoría mediante Azure Portal](https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-download-audit-report).

---

### <a name="breaking-change-updates-to-condition-evaluation-by-exchange-activesync-eas"></a>Cambio importante: modificaciones en la evaluación de estado de Exchange ActiveSync (EAS)

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Acceso condicional Control de acceso

Estamos actualizando el modo en que Exchange ActiveSync (EAS) evalúa las condiciones siguientes:

- Ubicación del usuario, basada en el país, la región o la dirección IP.

- Riesgo de inicio de sesión

- Plataforma de dispositivo

Si ha usado estas condiciones anteriormente en las directivas de acceso condicional, tenga en cuenta que el comportamiento de la condición puede cambiar. Por ejemplo, si ha usado la condición de ubicación de usuario en una directiva, es posible que la directiva se omita ahora en función de la ubicación del usuario.

---

## <a name="february-2019"></a>Febrero de 2019

### <a name="configurable-azure-ad-saml-token-encryption-public-preview"></a>Cifrado de tokens SAML de Azure AD configurable (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales SSO

Ahora se puede configurar cualquier aplicación SAML compatible para que reciba tokens SAML cifrados. Cuando se configura y se usa con una aplicación, Azure AD cifra las aserciones SAML emitidas mediante una clave pública que se obtiene de un certificado almacenado en Azure AD.

Para obtener más información sobre cómo configurar el cifrado de tokens SAML, vea [Configuración del cifrado de tokens SAML de Azure AD](https://docs.microsoft.com/azure/active-directory/manage-apps/howto-saml-token-encryption).

---

### <a name="create-an-access-review-for-groups-or-apps-using-azure-ad-access-reviews"></a>Creación de una revisión de acceso para grupos o aplicaciones con las revisiones de acceso de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Revisiones de acceso Gobernanza

Ahora se pueden incluir varios grupos o aplicaciones en una única revisión de acceso de Azure AD para comprobar la pertenencia a grupos o la asignación de aplicaciones. Las revisiones de acceso con varios grupos o aplicaciones se configuran usando los mismos valores y todos los revisores incluidos son notificados al mismo tiempo.

Para obtener más información sobre cómo crear una revisión de acceso con las revisiones acceso de Azure AD, vea [Creación de una revisión de acceso de grupos o aplicaciones con las revisiones de acceso de Azure AD](https://docs.microsoft.com/azure/active-directory/governance/create-access-review).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---february-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (febrero de 2019)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En febrero de 2019, hemos agregado estas 27 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones:

[Euromonitor Passport](https://docs.microsoft.com/azure/active-directory/saas-apps/euromonitor-passport-tutorial), [MindTickle](https://docs.microsoft.com/azure/active-directory/saas-apps/mindtickle-tutorial), [FAT FINGER](https://seeforgetest-exxon.azurewebsites.net/Account/create?Length=7), [AirStack](https://docs.microsoft.com/azure/active-directory/saas-apps/airstack-tutorial), [Oracle Fusion ERP](https://docs.microsoft.com/azure/active-directory/saas-apps/oracle-fusion-erp-tutorial), [IDrive](https://docs.microsoft.com/azure/active-directory/saas-apps/idrive-tutorial), [Skyward Qmlativ](https://docs.microsoft.com/azure/active-directory/saas-apps/skyward-qmlativ-tutorial), [Brightidea](https://docs.microsoft.com/azure/active-directory/saas-apps/brightidea-tutorial), [AlertOps](https://docs.microsoft.com/azure/active-directory/saas-apps/alertops-tutorial), [Soloinsight-CloudGate SSO](https://docs.microsoft.com/azure/active-directory/saas-apps/soloinsight-cloudgate-sso-tutorial), Permission Click, [Brandfolder](https://docs.microsoft.com/azure/active-directory/saas-apps/brandfolder-tutorial), [StoregateSmartFile](https://docs.microsoft.com/azure/active-directory/saas-apps/smartfile-tutorial), [Pexip](https://docs.microsoft.com/azure/active-directory/saas-apps/pexip-tutorial), [Stormboard](https://docs.microsoft.com/azure/active-directory/saas-apps/stormboard-tutorial), [Seismic](https://docs.microsoft.com/azure/active-directory/saas-apps/seismic-tutorial), [Share A Dream](https://www.shareadream.org/how-it-works), [Bugsnag](https://docs.microsoft.com/azure/active-directory/saas-apps/bugsnag-tutorial), [webMethods Integration Cloud](https://docs.microsoft.com/azure/active-directory/saas-apps/webmethods-integration-cloud-tutorial), [Knowledge Anywhere LMS](https://docs.microsoft.com/azure/active-directory/saas-apps/knowledge-anywhere-lms-tutorial), [OU Campus](https://docs.microsoft.com/azure/active-directory/saas-apps/ou-campus-tutorial), [Periscope Data](https://docs.microsoft.com/azure/active-directory/saas-apps/periscope-data-tutorial), [Netop Portal](https://docs.microsoft.com/azure/active-directory/saas-apps/netop-portal-tutorial), [smartvid.io](https://docs.microsoft.com/azure/active-directory/saas-apps/smartvid.io-tutorial), [PureCloud by Genesys](https://docs.microsoft.com/azure/active-directory/saas-apps/purecloud-by-genesys-tutorial) y [ClickUp Productivity Platform](https://docs.microsoft.com/azure/active-directory/saas-apps/clickup-productivity-platform-tutorial)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="enhanced-combined-mfasspr-registration"></a>Mejora del registro MFA/SSPR combinado

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autoservicio de restablecimiento de contraseña Autenticación de usuarios

En respuesta a los comentarios de los clientes, se ha mejorado la experiencia en versión preliminar del registro combinado de MFA/SSPR para que los usuarios puedan registrar de forma más rápida su información de seguridad para MFA y SSPR.

**Si quiere activar ahora la experiencia mejorada para los usuarios, siga estos pasos:**

1. Inicie sesión como administrador global o administrador de usuarios en Azure Portal y vaya a **Azure Active Directory > Configuración de usuario > Administrar la configuración de las características en versión preliminar del panel de acceso**.

2. En la opción **Usuarios que pueden utilizar las características en versión preliminar para registrar y administrar la información de seguridad – actualizar**, elija activar las características para un **Grupo seleccionado de usuarios** o para **Todos los usuarios**.

A lo largo de las próximas semanas se retirará la capacidad de activar la antigua experiencia combinada de vista previa de registro MFA/SSPR para los inquilinos que todavía no la hayan activado.

**Para ver si el control se retirará en su inquilino, siga estos pasos:**

1. Inicie sesión como administrador global o administrador de usuarios en Azure Portal y vaya a **Azure Active Directory > Configuración de usuario > Administrar la configuración de las características en versión preliminar del panel de acceso**.

2. Si la opción **Usuarios que pueden utilizar las características en versión preliminar para registrar y administrar la información de seguridad** está establecida en **Ninguno**, la opción se retirará del inquilino.

Independientemente de si con anterioridad se ha activado la antigua experiencia combinada de vista previa de registro MFA/SSPR, la antigua experiencia se desactivará en una fecha futura. Por ello, es muy recomendable que cambie a la nueva experiencia lo antes posible.

Para obtener más información sobre la experiencia de registro mejorada, vea la entrada de blog sobre las [interesantes mejoras en la experiencia de registro combinado MFA y restablecimiento de contraseña de Azure AD](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Cool-enhancements-to-the-Azure-AD-combined-MFA-and-password/ba-p/354271).

---

### <a name="updated-policy-management-experience-for-user-flows"></a>Experiencia actualizada de la administración de directivas para flujos de usuario

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Se ha actualizado el proceso de creación y administración de directivas para facilitar los flujos del usuario (anteriormente conocidos como directivas integradas). Esta experiencia nueva es ahora la predeterminada para todos los inquilinos de Azure AD.

Si quiere hacer algún otro comentario o sugerencia, use los iconos de sonrisa o desaprobación del área **Envíenos comentarios** en la parte superior de la pantalla del portal.

Para obtener más información sobre la nueva experiencia de administración de directivas, vea la entrada de blog [Azure AD B2C tiene ahora la personalización de JavaScript y muchas más características nuevas](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-B2C-now-has-JavaScript-customization-and-many-more-new/ba-p/353595).

---

### <a name="choose-specific-page-element-versions-provided-by-azure-ad-b2c"></a>Elección de versiones específicas de elementos de página proporcionadas por Azure AD B2C

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Ahora puede elegir una versión específica de los elementos de página proporcionados por Azure AD B2C. Si selecciona una versión específica, puede probar las actualizaciones antes de que aparezcan en la página y predecir el comportamiento. Además, ahora puede decidir aplicar versiones específicas de la página para permitir las personalizaciones de JavaScript. Para activar esta característica, vaya a la página **Propiedades** en los flujos de usuario.

Para obtener más información sobre cómo elegir versiones específicas de los elementos de la página, vea la entrada de blog [Azure AD B2C tiene ahora la personalización de JavaScript y muchas más características nuevas](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-B2C-now-has-JavaScript-customization-and-many-more-new/ba-p/353595).

---

### <a name="configurable-end-user-password-requirements-for-b2c-ga"></a>Requisitos de contraseña del usuario final configurables para B2C (GA)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Ya es posible configurar la complejidad de las contraseñas que usan los usuarios finales de la organización, en lugar de tener que utilizar la directiva nativa de contraseñas de Azure AD. En la hoja **Propiedades** de los flujos de usuario (antes conocidos como directivas integradas), puede elegir una complejidad de contraseña **Simple** o **Segura**, o bien puede crear un conjunto de requisitos **Personalizado**.

Para obtener más información sobre la configuración de los requisitos de complejidad de contraseñas, vea [Configuración de los requisitos de complejidad de contraseñas de Azure Active Directory B2C](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-reference-password-complexity).

---

### <a name="new-default-templates-for-custom-branded-authentication-experiences"></a>Nuevas plantillas predeterminadas para experiencias de autenticación con marca personalizada

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Puede usar nuestras nuevas plantillas predeterminadas, ubicadas en la hoja **Diseños de página** de los flujos de usuario (anteriormente conocidos como directivas integradas), para que los usuarios disfruten de una experiencia de autenticación adaptada a su marca.

Para obtener más información sobre el uso de las plantillas, vea [Azure AD B2C tiene ahora la personalización de JavaScript y muchas más características nuevas](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-B2C-now-has-JavaScript-customization-and-many-more-new/ba-p/353595).

---

## <a name="january-2019"></a>Enero de 2019

### <a name="active-directory-b2b-collaboration-using-one-time-passcode-authentication-public-preview"></a>Colaboración B2B de Active Directory mediante la autenticación con código de acceso de un solo uso (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2B B2B/B2C

Se ha introducido la autenticación con código de acceso de un solo uso (OTP) para los usuarios invitados de B2B que no pueden autenticarse por otros medios, como Azure AD, una cuenta Microsoft (MSA) o la federación de Google. Este nuevo método de autenticación significa que los usuarios invitados no tienen que crear una nueva cuenta Microsoft. En cambio, al canjear una invitación o acceder a un recurso compartido, un usuario invitado puede solicitar el envío de un código temporal a una dirección de correo electrónico. Con este código temporal, el usuario invitado puede seguir iniciando sesión.

Para más información, consulte [Autenticación con código de acceso de un solo uso de correo electrónico (versión preliminar)](https://docs.microsoft.com/azure/active-directory/b2b/one-time-passcode) y el blog [Azure AD makes sharing and collaboration seamless for any user with any account](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-makes-sharing-and-collaboration-seamless-for-any-user/ba-p/325949) (Azure AD simplifica el uso compartido y la colaboración para cualquier usuario con una cuenta).

### <a name="new-azure-ad-application-proxy-cookie-settings"></a>Nueva configuración de cookies de Azure AD Application Proxy

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Proxy de aplicaciones Control de acceso

Hemos incluido tres nuevas configuraciones de cookies, disponibles para las aplicaciones que se publican a través de Application Proxy:

- **Usar cookie solo HTTP.** Establece la marca **HTTPOnly** en las cookies de sesión y acceso de Application Proxy. La activación de esta configuración ofrece ventajas de seguridad adicionales, como ayuda para evitar la copia o modificación de cookies por medio de scripting del lado cliente. Se recomienda activar esta marca (elija **Sí**) para disfrutar de estas ventajas.

- **Usar cookies seguras.** Establece la marca **Segura** en las cookies de sesión y acceso de Application Proxy. La activación de esta configuración ofrece ventajas de seguridad adicionales, como la garantía de que las cookies se transmiten solo a través de canales seguros de TLS, como HTTPS. Se recomienda activar esta marca (elija **Sí**) para disfrutar de estas ventajas.

- **Usar cookies persistentes.** Impide que las cookies de acceso expiren cuando se cierra el explorador web. Estas cookies se mantienen vigentes durante toda la duración del token de acceso. Sin embargo, las cookies se restablecen si se alcanza la hora de expiración o si el usuario elimina manualmente la cookie. Se recomienda que mantenga la configuración predeterminada **No**, activando el ajuste solo para las aplicaciones anteriores que no comparten cookies entre procesos.

Para obtener más información acerca de las nuevas cookies, consulte [Configuración de las cookies para el acceso a aplicaciones locales en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-configure-cookie-settings).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---january-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (enero de 2019)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En enero de 2019, hemos agregado estas 35 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones:

[Firstbird](https://docs.microsoft.com/azure/active-directory/saas-apps/firstbird-tutorial), [Folloze](https://docs.microsoft.com/azure/active-directory/saas-apps/folloze-tutorial), [Talent Palette](https://docs.microsoft.com/azure/active-directory/saas-apps/talent-palette-tutorial), [Infor CloudSuite](https://docs.microsoft.com/azure/active-directory/saas-apps/infor-cloud-suite-tutorial), [Cisco Umbrella](https://docs.microsoft.com/azure/active-directory/saas-apps/cisco-umbrella-tutorial), [Zscaler Internet Access Administrator](https://docs.microsoft.com/azure/active-directory/saas-apps/zscaler-internet-access-administrator-tutorial), [Expiration Reminder](https://docs.microsoft.com/azure/active-directory/saas-apps/expiration-reminder-tutorial), [InstaVR Viewer](https://docs.microsoft.com/azure/active-directory/saas-apps/instavr-viewer-tutorial), [CorpTax](https://docs.microsoft.com/azure/active-directory/saas-apps/corptax-tutorial), [Verb](https://app.verb.net/login), [OpenLattice](https://openlattice.com/agora), [TheOrgWiki](https://www.theorgwiki.com/signup), [Pavaso Digital Close](https://docs.microsoft.com/azure/active-directory/saas-apps/pavaso-digital-close-tutorial), [GoodPractice Toolkit](https://docs.microsoft.com/azure/active-directory/saas-apps/goodpractice-toolkit-tutorial), [Cloud Service PICCO](https://docs.microsoft.com/azure/active-directory/saas-apps/cloud-service-picco-tutorial), [AuditBoard](https://docs.microsoft.com/azure/active-directory/saas-apps/auditboard-tutorial), [iProva](https://docs.microsoft.com/azure/active-directory/saas-apps/iprova-tutorial), [Workable](https://docs.microsoft.com/azure/active-directory/saas-apps/workable-tutorial), [CallPlease](https://webapp.callplease.com/create-account/create-account.html), [GTNexus SSO System](https://docs.microsoft.com/azure/active-directory/saas-apps/gtnexus-sso-module-tutorial), [CBRE ServiceInsight](https://docs.microsoft.com/azure/active-directory/saas-apps/cbre-serviceinsight-tutorial), [Deskradar](https://docs.microsoft.com/azure/active-directory/saas-apps/deskradar-tutorial), [Coralogixv](https://docs.microsoft.com/azure/active-directory/saas-apps/coralogix-tutorial), [Signagelive](https://docs.microsoft.com/azure/active-directory/saas-apps/signagelive-tutorial), [ARES for Enterprise](https://docs.microsoft.com/azure/active-directory/saas-apps/ares-for-enterprise-tutorial), [K2 for Office 365](https://www.k2.com/O365), [Xledger](https://www.xledger.net/), [iDiD Manager](https://docs.microsoft.com/azure/active-directory/saas-apps/idid-manager-tutorial), [HighGear](https://docs.microsoft.com/azure/active-directory/saas-apps/highgear-tutorial), [Visitly](https://docs.microsoft.com/azure/active-directory/saas-apps/visitly-tutorial), [Korn Ferry ALP](https://docs.microsoft.com/azure/active-directory/saas-apps/korn-ferry-alp-tutorial), [Acadia](https://docs.microsoft.com/azure/active-directory/saas-apps/acadia-tutorial) y [Adoddle cSaas Platform](https://docs.microsoft.com/azure/active-directory/saas-apps/adoddle-csaas-platform-tutorial)<!-- , [CaféX Portal (Meetings)](https://docs.microsoft.com/azure/active-directory/saas-apps/cafexportal-meetings-tutorial), [MazeMap Link](https://docs.microsoft.com/azure/active-directory/saas-apps/mazemaplink-tutorial)-->

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="new-azure-ad-identity-protection-enhancements-public-preview"></a>Nuevas mejoras de Azure AD Identity Protection (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Identity Protection Seguridad y protección de la identidad

Nos complace anunciar que hemos agregado las siguientes mejoras a la oferta de versión preliminar pública de Azure AD Identity Protection:

- Una interfaz de usuario actualizada y más integrada

- API adicionales

- Evaluación de riesgos mejorada a través de aprendizaje automático

- Alineación de todo el producto entre usuarios e inicios de sesión no seguros

Para obtener más información sobre las mejoras, consulte [What is Azure Active Directory Identity Protection (refreshed)?](https://aka.ms/IdentityProtectionDocs) (¿Qué es Azure Active Directory Identity Protection [actualizado]?) a fin de conocer los detalles y compartir sus ideas a través de los mensajes en el producto.

---

### <a name="new-app-lock-feature-for-the-microsoft-authenticator-app-on-ios-and-android-devices"></a>Nueva característica Bloqueo de aplicación para la aplicación Microsoft Authenticator en dispositivos iOS y Android

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicación Microsoft Authenticator Seguridad y protección de la identidad

Para proteger sus códigos de acceso de un solo uso, la información de aplicación y la configuración de la aplicación, puede activar la característica Bloqueo de aplicación en la aplicación Microsoft Authenticator. Al activar el Bloqueo de aplicación se le pedirá que se autentique con su PIN o características biométricas cada vez que abra la aplicación Microsoft Authenticator.

Para obtener más información, vea las [Preguntas más frecuentes de la aplicación Microsoft Authenticator](https://docs.microsoft.com/azure/active-directory/user-help/microsoft-authenticator-app-faq).

---

### <a name="enhanced-azure-ad-privileged-identity-management-pim-export-capabilities"></a>Mejora de las capacidades de exportación de Azure AD Privileged Identity Management (PIM)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Privileged Identity Management Privileged Identity Management

Los administradores de Privileged Identity Management (PIM) ahora pueden exportar todas las asignaciones de roles activas y válidas para un recurso específico, lo cual incluye las asignaciones de roles para todos los recursos secundarios. Anteriormente, a los administradores les resultaba difícil obtener una lista completa de asignaciones de roles para una suscripción y tenían que exportar dichas asignaciones para cada recurso específico.

Para obtener más información, consulte [Visualización de la actividad y del historial de auditoría para los roles de recursos de Azure en PIM](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/azure-pim-resource-rbac).

---

## <a name="novemberdecember-2018"></a>Noviembre/diciembre de 2018

### <a name="users-removed-from-synchronization-scope-no-longer-switch-to-cloud-only-accounts"></a>Los usuarios que se quitaron del ámbito de sincronización ya no podrán cambiarse a cuentas solo en la nube

**Tipo:** **Categoría del servicio:** Corregida **Funcionalidad del producto:** Administración de usuarios Directorio

>[!Important]
>Hemos escuchado y comprendido la frustración que les genera esta corrección. Por lo tanto, hemos revertido este cambio hasta el momento en que podamos ofrecerle una implementación más sencilla para su organización.

Se ha corregido un error en el que la marca DirSyncEnabled de un usuario se cambiaba erróneamente a **False** cuando el objeto de Active Directory Domain Services se excluía del ámbito de sincronización y, después, se movía a la Papelera de reciclaje de Azure AD en el siguiente ciclo de sincronización. Como resultado de esta corrección, si el usuario se excluye del ámbito de sincronización y posteriormente se restaura desde la Papelera de reciclaje de Azure AD, la cuenta de dicho usuario permanece sincronizada desde la instancia local de AD, según lo previsto, y no se puede administrar en la nube, ya que su origen de autoridad (SoA) permanece como AD local.

Antes de esta corrección, había un problema cuando se cambiaba la marca DirSyncEnabled a False. Daba la impresión errónea de que estas cuentas se convertían en objetos solo en la nube y de que las cuentas se podían administrar en la nube. Sin embargo, las cuentas siguen conservando su SoA como propiedades locales y sincronizadas (atributos paralelos) procedentes de la instancia local de AD. Esta situación provocó varios problemas en Azure AD y en otras cargas de trabajo en la nube (como Exchange Online), ya que esperaban tratar estas cuentas como sincronizadas desde AD y, sin embargo, se comportaban como cuentas solo en la nube.

En este momento, la única manera de convertir realmente una cuenta sincronizada de AD en una cuenta de solo nube, es deshabilitando DirSync en el nivel del inquilino, lo que desencadena una operación de back-end para transferir el SoA. Este tipo de cambio de SoA requiere (pero no se limita a) la limpieza de todos los atributos locales relacionados (como los atributos paralelos y de LastDirSyncTime) y el envío de una señal a otras cargas de trabajo en la nube para convertir también su objeto respectivo en una cuenta solo en la nube.

Por lo tanto, esta corrección evita las actualizaciones directas sobre el atributo ImmutableID de un usuario sincronizado desde AD, lo que era necesario en algunos escenarios en el pasado. Por diseño, el atributo ImmutableID de un objeto en Azure AD, como su nombre indica, es inmutable. Existen nuevas características implementadas en los clientes de sincronización de Azure AD Connect y Azure AD Connect Health para abordar estos escenarios:

- **Actualizaciones del atributo ImmutableID a gran escala para muchos usuarios por etapas**

  Por ejemplo, debe realizar una migración entre bosques de AD DS larga. Solución: Use Azure AD Connect para la **Configuración del delimitador de origen** y, mientras el usuario migra, copie los valores del atributo ImmutableID existentes de Azure AD al atributo DS-Consistency-Guid del usuario de AD DS local del nuevo bosque. Para obtener más información, consulte [Uso de msDS-ConsistencyGuid como sourceAnchor](/azure/active-directory/hybrid/plan-connect-design-concepts#using-ms-ds-consistencyguid-as-sourceanchor).

- **Actualizaciones del atributo ImmutableID a gran escala para muchos usuarios de una sola vez**

  Por ejemplo, al implementar Azure AD Connect se cometió un error, y ahora es necesario cambiar el atributo SourceAnchor. Solución: Deshabilite DirSync en el nivel del inquilino y borre todos los valores de ImmutableID no válidos. Para obtener más información, vea [Desactivar la sincronización de directorios para Office 365](/office365/enterprise/turn-off-directory-synchronization).

- **Hacer coincidir un usuario local con un usuario existente de Azure AD** Por ejemplo, un usuario que se haya vuelto a crear en AD DS genera un duplicado en la cuenta de Azure AD en lugar de reasignarlo a una cuenta de Azure AD existente (objeto huérfano). Solución: Use Azure AD Connect Health en Azure Portal para reasignar el delimitador de origen o el atributo ImmutableID. Para obtener más información, consulte [Escenario del objeto huérfano](/azure/active-directory/hybrid/how-to-connect-health-diagnose-sync-errors#orphaned-object-scenario).

### <a name="breaking-change-updates-to-the-audit-and-sign-in-logs-schema-through-azure-monitor"></a>Cambio de última hora: Actualizaciones de los esquemas de registros de auditoría y de inicio de sesión mediante Azure Monitor

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto**: Informes Supervisión e informes

Actualmente, estamos publicando los flujos de registro de auditorías e inicios de sesión mediante Azure Monitor, por lo que puede integrar fácilmente los archivos de registro con sus herramientas SIEM o con Log Analytics. Basándonos en sus comentarios, y en preparación para el anuncio de disponibilidad general para esta característica, vamos a realizar los siguientes cambios en el esquema. Estos cambios en los esquemas y las actualizaciones de documentación relacionadas se producirán en la primera semana de enero.

#### <a name="new-fields-in-the-audit-schema"></a>Nuevos campos en el esquema de auditoría
Vamos a agregar un nuevo campo **Tipo de operación**, para proporcionar el tipo de operación que se realiza en el recurso. Por ejemplo, **Agregar**, **Actualizar** o **Eliminar**.

#### <a name="changed-fields-in-the-audit-schema"></a>Campos modificados en el esquema de auditoría
Los siguientes campos han cambiado en el esquema de auditoría:

|Nombre del campo|Qué cambia|Valores anteriores|Nuevos valores|
|----------|------------|----------|----------|
|Category|Este era el campo **Nombre de servicio**. Ahora es el campo **Categorías de auditoría**. A **Nombre de servicio** se le ha cambiado el nombre por el de campo **loggedByService**.|<ul><li>Account Provisioning (Aprovisionamiento de cuentas)</li><li>Core Directory (Directorio principal)</li><li>Autoservicio de restablecimiento de contraseña</li></ul>|<ul><li>User Management</li><li>Administración de grupos</li><li>Administración de la aplicación</li></ul>|
|targetResources|Incluye **TargetResourceType** en el nivel superior.|&nbsp;|<ul><li>Directiva</li><li>Aplicación</li><li>Usuario</li><li>Grupo</li></ul>|
|loggedByService|Proporciona el nombre del servicio que generó el registro de auditoría.|Null|<ul><li>Account Provisioning (Aprovisionamiento de cuentas)</li><li>Core Directory (Directorio principal)</li><li>Restablecimiento de la contraseña de autoservicio</li></ul>|
|Resultado|Proporciona el resultado de los registros de auditoría. Anteriormente, este aparecía en una lista, pero ahora se muestra el valor real.|<ul><li>0</li><li>1</li></ul>|<ul><li>Correcto</li><li>Error</li></ul>|

#### <a name="changed-fields-in-the-sign-in-schema"></a>Campos modificados en el esquema de inicio de sesión
Los siguientes campos han cambiado en el esquema de inicio de sesión:

|Nombre del campo|Qué cambia|Valores anteriores|Nuevos valores|
|----------|------------|----------|----------|
|appliedConditionalAccessPolicies|Este era el campo **conditionalaccessPolicies**. Ahora es el campo **appliedConditionalAccessPolicies**.|Sin cambios|Sin cambios|
|conditionalAccessStatus|Proporciona el resultado del estado de la directiva de acceso condicional en el inicio de sesión. Anteriormente, este aparecía en una lista, pero ahora se muestra el valor real.|<ul><li>0</li><li>1</li><li>2</li><li>3</li></ul>|<ul><li>Correcto</li><li>Error</li><li>No aplicado</li><li>Disabled</li></ul>|
|appliedConditionalAccessPolicies: result|Proporciona el resultado del estado individual de la directiva de acceso condicional en el inicio de sesión. Anteriormente, este aparecía en una lista, pero ahora se muestra el valor real.|<ul><li>0</li><li>1</li><li>2</li><li>3</li></ul>|<ul><li>Correcto</li><li>Error</li><li>No aplicado</li><li>Disabled</li></ul>|

Para más información acerca del esquema, consulte [Interpretación del esquema de registros de auditoría de Azure AD en Azure Monitor (versión preliminar)](https://docs.microsoft.com/azure/active-directory/reports-monitoring/reference-azure-monitor-audit-log-schema).

---

### <a name="identity-protection-improvements-to-the-supervised-machine-learning-model-and-the-risk-score-engine"></a>Mejoras de la protección de identidades en el modelo de Machine Learning supervisado y en el motor de puntuaciones de riesgo

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Identity Protection Puntuaciones de riesgo

Las mejoras en el usuario relacionadas con la protección de identidades y en el motor de evaluación de riesgos de inicio de sesión pueden ayudar a mejorar la precisión y cobertura de los riesgos del usuario. Puede que los administradores hayan observado que el nivel de riesgo del usuario ya no está vinculado directamente al nivel de riesgo de detecciones específicas y que hay un aumento en el número y en el nivel de los eventos de inicio de sesión de riesgo.

El modelo de Machine Learning supervisado es el encargado ahora de evaluar las detecciones de riesgo. Este calcula el riesgo del usuario usando características adicionales de los inicios de sesión del usuario y un patrón de detecciones. Según este modelo, el administrador puede detectar usuarios con puntuaciones de riesgo altas, incluso aunque las detecciones asociadas con ese usuario sean de nivel bajo o medio.

---

### <a name="administrators-can-reset-their-own-password-using-the-microsoft-authenticator-app-public-preview"></a>Los administradores pueden restablecer sus propias contraseñas mediante la aplicación Microsoft Authenticator (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autoservicio de restablecimiento de contraseña Autenticación de usuarios

Los administradores de Azure AD ya pueden restablecer sus propias contraseñas mediante las notificaciones de la aplicación Microsoft Authenticator o mediante un código de cualquier aplicación de autenticación móvil, o token de hardware. Para restablecer sus propias contraseñas, los administradores ahora pueden usar dos de los métodos siguientes:

- Notificación de la aplicación Microsoft Authenticator

- Un código de otra aplicación de autenticación móvil o un token de hardware

- Email

- llamada de teléfono

- mensaje de texto

Para más información sobre el uso de la aplicación Microsoft Authenticator para restablecer contraseñas, consulte [Autoservicio de restablecimiento de contraseña de Azure AD: Aplicación móvil y SSPR (versión preliminar)](https://docs.microsoft.com/azure/active-directory/authentication/concept-sspr-howitworks#mobile-app-and-sspr)

---

### <a name="new-azure-ad-cloud-device-administrator-role-public-preview"></a>Nuevo rol de administrador de dispositivos en la nube de Azure AD (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración y registro de dispositivos Control de acceso

Los administradores pueden asignar usuarios al nuevo rol de administrador de dispositivos en la nube para realizar las tareas propias de este rol. Los usuarios asignados a este rol pueden habilitar, deshabilitar y eliminar dispositivos en Azure AD y leer las claves de BitLocker de Windows 10 (si las hay) en Azure Portal.

Para más información acerca de los roles y permisos, consulte [Asignación de roles de administrador en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles).

---

### <a name="manage-your-devices-using-the-new-activity-timestamp-in-azure-ad-public-preview"></a>Administración de dispositivos con la nueva marca de tiempo de actividad de Azure AD (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración y registro de dispositivos Administración del ciclo de vida de dispositivos

Somos conscientes de que, con el paso del tiempo, debe actualizar y retirar los dispositivos de su organización en Azure AD para evitar tener dispositivos obsoletos en su entorno. Para ayudarle con este proceso, Azure AD ya actualiza los dispositivos con una nueva marca de tiempo de actividad que le ayuda a administrar el ciclo de vida del dispositivo.

Para más información acerca de cómo obtener y usar esta marca de tiempo, consulte [Procedimiento: Administración de dispositivos obsoletos en Azure AD](https://docs.microsoft.com/azure/active-directory/devices/manage-stale-devices).

---

### <a name="administrators-can-require-users-to-accept-a-terms-of-use-on-each-device"></a>Los administradores pueden requerir que los usuarios acepten los términos de uso en cada dispositivo

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Gobernanza

Los administradores ahora pueden activar la opción **Requerir que los usuarios concedan su consentimiento en todos los dispositivos** para requerir que los usuarios acepten los términos de uso en todos los dispositivos que estén usando en su inquilino.

Para obtener más información, consulte [la sección de términos de uso por dispositivo del artículo Característica Términos de uso de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/conditional-access/terms-of-use#per-device-terms-of-use).

---

### <a name="administrators-can-configure-a-terms-of-use-to-expire-based-on-a-recurring-schedule"></a>Los administradores pueden configurar la expiración de los términos de uso mediante una programación periódica

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Gobernanza


Los administradores pueden ahora activar la opción **Expirar autorizaciones** para hacer que los términos de uso expiren para todos los usuarios según la programación periódica especificada. La programación puede ser anual, semestral, trimestral o mensual. Una vez que los términos de uso expiran, los usuarios deben volver a aceptarlos.

Para obtener más información, consulte [la sección Agregar términos de uso del artículo Característica Términos de uso de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/conditional-access/terms-of-use#add-terms-of-use).

---

### <a name="administrators-can-configure-a-terms-of-use-to-expire-based-on-each-users-schedule"></a>Los administradores pueden configurar la expiración de los términos de uso según la programación de cada usuario

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Gobernanza

Los administradores ahora pueden especificar una duración tras la cual el usuario debe volver a aceptar los términos de uso. Por ejemplo, los administradores pueden especificar que los usuarios deben volver a aceptar los términos de uso cada 90 días.

Para obtener más información, consulte [la sección Agregar términos de uso del artículo Característica Términos de uso de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/conditional-access/terms-of-use#add-terms-of-use).

---

### <a name="new-azure-ad-privileged-identity-management-pim-emails-for-azure-active-directory-roles"></a>Nuevos correos electrónicos de Azure Active Directory Privileged Identity Management (PIM) para los roles de Azure Active Directory

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Privileged Identity Management Privileged Identity Management

Los clientes que usan Azure AD Privileged Identity Management (PIM) ahora pueden recibir un correo electrónico de resumen semanal que incluye la información siguiente de los últimos siete días:

- Introducción a las asignaciones de roles principales y permanentes

- Número de usuarios que activan roles

- Número de usuarios asignados a roles en PIM

- Número de usuarios asignados a roles fuera de PIM

- Número de usuarios a los que se ha "convertido en permanentes" en PIM

Para más información sobre PIM y las notificaciones de correo electrónico disponibles, consulte [Notificaciones por correo electrónico en PIM](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-email-notifications).

---

### <a name="group-based-licensing-is-now-generally-available"></a>Las licencias basadas en grupos tienen ahora disponibilidad general

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Otros Directorio

Las licencias basadas en grupos ya no están en la fase de versión preliminar pública sino en la de disponibilidad general. Como parte de esta versión general, se ha logrado que esta característica sea más escalable y se ha agregado la posibilidad de volver a procesar las asignaciones de licencias basadas en grupos para un solo usuario y la posibilidad de usar licencias basadas en grupos con licencias E3/A3 de Office 365.

Para más información acerca de las licencias basadas en grupos, consulte [¿En qué consisten las licencias basadas en grupos de Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-licensing-whatis-azure-portal)

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---november-2018"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: noviembre de 2018

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En noviembre de 2018, hemos agregado 26 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[CoreStack](https://cloud.corestack.io/site/login), [HubSpot](https://docs.microsoft.com/azure/active-directory/saas-apps/HubSpot-tutorial), [GetThere](https://docs.microsoft.com/azure/active-directory/saas-apps/getthere-tutorial), [Gra-Pe](https://docs.microsoft.com/azure/active-directory/saas-apps/grape-tutorial), [eHour](https://getehour.com/try-now), [Consent2Go](https://docs.microsoft.com/azure/active-directory/saas-apps/Consent2Go-tutorial), [Appinux](https://docs.microsoft.com/azure/active-directory/saas-apps/appinux-tutorial), [DriveDollar](https://azuremarketplace.microsoft.com/marketplace/apps/savitas.drivedollar-azuread?tab=Overview), [Useall](https://docs.microsoft.com/azure/active-directory/saas-apps/useall-tutorial), [Infinite Campus](https://docs.microsoft.com/azure/active-directory/saas-apps/infinitecampus-tutorial), [Alaya](https://alayagood.com), [HeyBuddy](https://docs.microsoft.com/azure/active-directory/saas-apps/heybuddy-tutorial), [Wrike SAML](https://docs.microsoft.com/azure/active-directory/saas-apps/wrike-tutorial), [Drift](https://docs.microsoft.com/azure/active-directory/saas-apps/drift-tutorial), [Zenegy for Business Central 365](https://accounting.zenegy.com/), [Everbridge Member Portal](https://docs.microsoft.com/azure/active-directory/saas-apps/everbridge-tutorial), [IDEO](https://profile.ideo.com/users/sign_up), [Ivanti Service Manager (ISM)](https://docs.microsoft.com/azure/active-directory/saas-apps/ivanti-service-manager-tutorial), [Peakon](https://docs.microsoft.com/azure/active-directory/saas-apps/peakon-tutorial), [Allbound SSO](https://docs.microsoft.com/azure/active-directory/saas-apps/allbound-sso-tutorial), [Plex Apps - Classic Test](https://test.plexonline.com/signon), [Plex Apps – Classic](https://www.plexonline.com/signon), [Plex Apps - UX Test](https://test.cloud.plex.com/sso), [Plex Apps – UX](https://cloud.plex.com/sso), [Plex Apps – IAM](https://accounts.plex.com/), [CRAFTS - Childcare Records, Attendance, & Financial Tracking System](https://getcrafts.ca/craftsregistration)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

## <a name="october-2018"></a>Octubre de 2018

### <a name="azure-ad-logs-now-work-with-azure-log-analytics-public-preview"></a>Los registros de Azure AD ahora funcionan con Azure Log Analytics (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Informes Supervisión e informes

Nos complace anunciar que ahora puede reenviar los registros de Azure AD a Azure Log Analytics. Esta característica tan solicitada le ayuda a obtener un acceso aún mejor al análisis para su negocio, las operaciones y la seguridad, además de contribuir a supervisar la infraestructura. Para más información, consulte el blog sobre [los registros de actividad de Azure Active Directory en Azure Log Analytics que ya está disponible](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-Active-Directory-Activity-logs-in-Azure-Log-Analytics-now/ba-p/274843).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---october-2018"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: octubre de 2018

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En octubre de 2018, hemos agregado 14 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[My Award Points](https://docs.microsoft.com/azure/active-directory/saas-apps/myawardpoints-tutorial), [Vibe HCM](https://docs.microsoft.com/azure/active-directory/saas-apps/vibehcm-tutorial), ambyint, [MyWorkDrive](https://docs.microsoft.com/azure/active-directory/saas-apps/myworkdrive-tutorial), [BorrowBox](https://docs.microsoft.com/azure/active-directory/saas-apps/borrowbox-tutorial), Dialpad, [ON24 Virtual Environment](https://docs.microsoft.com/azure/active-directory/saas-apps/on24-tutorial), [RingCentral](https://docs.microsoft.com/azure/active-directory/saas-apps/ringcentral-tutorial), [Zscaler Three](https://docs.microsoft.com/azure/active-directory/saas-apps/zscaler-three-tutorial), [Phraseanet](https://docs.microsoft.com/azure/active-directory/saas-apps/phraseanet-tutorial), [Appraisd](https://docs.microsoft.com/azure/active-directory/saas-apps/appraisd-tutorial), [Workspot Control](https://docs.microsoft.com/azure/active-directory/saas-apps/workspotcontrol-tutorial), [Shuccho Navi](https://docs.microsoft.com/azure/active-directory/saas-apps/shucchonavi-tutorial), [Glassfrog](https://docs.microsoft.com/azure/active-directory/saas-apps/glassfrog-tutorial)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="azure-ad-domain-services-email-notifications"></a>Notificaciones por correo electrónico de Azure AD Domain Services

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure AD Domain Services Azure AD Domain Services

Azure AD Domain Services proporciona alertas en Azure Portal sobre configuraciones incorrectas o problemas con el dominio administrado. Estas alertas incluyen guías paso a paso para que pueda intentar corregir los problemas sin tener que ponerse en contacto con el servicio de soporte técnico.

A partir de octubre, podrá personalizar la configuración de notificaciones para el dominio administrado por lo que, cuando se produzcan nuevas alertas, se enviará un correo electrónico a un grupo de personas designado, sin necesidad de tener que comprobar constantemente si en el portal hay actualizaciones.

Para más información, consulte [Configuración de notificaciones de Azure AD Domain Services](https://docs.microsoft.com/azure/active-directory-domain-services/active-directory-ds-notifications).

---

### <a name="azure-ad-portal-supports-using-the-forcedelete-domain-api-to-delete-custom-domains"></a>El portal de Azure AD admite el uso de la API de dominio ForceDelete para eliminar los dominios personalizados

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Administración de directorios Directorio

Nos complace anunciar que ahora puede usar la API de dominio ForceDelete para eliminar los nombres de dominio personalizados cambiando las referencias de forma asincrónica, como los usuarios, los grupos y las aplicaciones del nombre de dominio personalizado (contoso.com) al nombre de dominio predeterminado inicial (contoso.onmicrosoft.com).

Este cambio ayuda a eliminar más rápidamente los nombres de dominio personalizados si la organización ya no utiliza el nombre o si necesita utilizar ese nombre de dominio con otra instancia de Azure AD.

Para más información, consulte [Eliminación de un nombre de dominio personalizado](https://docs.microsoft.com/azure/active-directory/users-groups-roles/domains-manage#delete-a-custom-domain-name).

---

## <a name="september-2018"></a>Septiembre de 2018

### <a name="updated-administrator-role-permissions-for-dynamic-groups"></a>Permisos de rol de administrador actualizados para grupos dinámicos

**Tipo:** **Categoría del servicio:** Corregida **Funcionalidad del producto:** Administración de grupos Colaboración

Se ha corregido un problema, por lo que determinados roles de administrador ya pueden crear y actualizar reglas de pertenencia dinámicas, sin necesidad de ser el propietario del grupo.

Los roles son:

- Administrador global

- Administrador de Intune

- Administrador de usuarios

Para más información, consulte [Creación de un grupo dinámico y comprobación de su estado](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-create-rule)

---

### <a name="simplified-single-sign-on-sso-configuration-settings-for-some-third-party-apps"></a>Valores de configuración simplificada de inicio de sesión único (SSO) para algunas aplicaciones de terceros

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales SSO

Somos conscientes de que la configuración del inicio de sesión único (SSO) para aplicaciones de SaaS (software como servicio) puede ser un desafío debido a la naturaleza única de la configuración de cada aplicación. Hemos creado una configuración simplificada para rellenar de forma automática los valores de configuración de SSO para las siguientes aplicaciones de SaaS de terceros:

- Zendesk

- ArcGis Online

- Jamf Pro

Para empezar a usar esta experiencia de un solo clic, vaya a la página **Azure Portal** > **Configuración de SSO** de la aplicación. Para más información, consulte [Integración de aplicación SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/saas-apps/tutorial-list)

---

### <a name="azure-active-directory---where-is-your-data-located-page"></a>Página Azure Active Directory - Where is your data located? (Azure Active Directory: dónde se encuentran los datos)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Otros GoLocal

Seleccione la región de su empresa en la página **Azure Active Directory - Where is your data located?** (Azure Active Directory: dónde se encuentran los datos) para ver qué centro de datos de Azure alberga los datos en reposo de Azure AD para todos los servicios de Azure AD. Puede filtrar la información por los servicios específicos de Azure AD para la región de su empresa.

Para acceder a esta característica y para más información, consulte [Azure Active Directory - Where is your data located?](https://aka.ms/AADDataMap) (Azure Active Directory: dónde se encuentran los datos).

---

### <a name="new-deployment-plan-available-for-the-my-apps-access-panel"></a>Nuevo plan de implementación disponible para el panel de acceso a Mis aplicaciones

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Mis aplicaciones SSO

Consulte el nuevo plan de implementación disponible para el panel de acceso a Mis aplicaciones (https://aka.ms/deploymentplans).
El panel de acceso a Mis aplicaciones proporciona a los usuarios un único lugar en el que pueden buscar y acceder a sus aplicaciones. Este portal también proporciona a los usuarios oportunidades de autoservicio, como por ejemplo solicitar acceso a aplicaciones y grupos, o administrar el acceso a estos recursos en nombre de otros.

Para más información, consulte [¿Qué es el portal Mis aplicaciones?](https://docs.microsoft.com/azure/active-directory/user-help/active-directory-saas-access-panel-introduction)

---

### <a name="new-troubleshooting-and-support-tab-on-the-sign-ins-logs-page-of-the-azure-portal"></a>Pestaña nueva solución de problemas y soporte técnico de la página de registros de inicios de sesión de Azure Portal

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Informes Supervisión e informes

El fin de la nueva pestaña **Solución de problemas y soporte técnico** de la página **Inicios de sesión** de Azure Portal es ayudar a los administradores e ingenieros de soporte técnico a solucionar problemas relacionados con los inicios de sesión de Azure AD. Esta nueva pestaña proporciona el código de error, mensaje de error y las recomendaciones de corrección (si existen) para ayudar a solucionar el problema. Si no puede resolver el problema, también le ofrecemos una nueva forma de crear una incidencia de soporte técnico mediante **Copiar al Portapapeles**, que rellena los campos **Id. de solicitud** y **Fecha (UTC)** del archivo de registro en su incidencia de soporte técnico.

![Registros de inicio de sesión que se muestran en la nueva pestaña](media/whats-new/troubleshooting-and-support.png)

---

### <a name="enhanced-support-for-custom-extension-properties-used-to-create-dynamic-membership-rules"></a>Compatibilidad mejorada con las propiedades de extensión personalizadas utilizadas para crear reglas de pertenencia dinámica

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Administración de grupos Colaboración

Con esta actualización, ahora puede hacer clic en el vínculo **Get custom extension properties** (Obtener propiedades de extensión personalizadas) desde el generador de reglas de grupo de usuarios dinámico, escriba el identificador de aplicación único y recibir la lista completa de las propiedades de extensión personalizadas para usarla al crear una regla de pertenencia dinámica para los usuarios. Esta lista también se puede actualizar esta lista para obtener nuevas propiedades de extensión personalizada para la aplicación.

Para más información acerca del uso de propiedades de extensión personalizadas para las reglas de pertenencia dinámica, consulte [Propiedades de extensión y propiedades de extensión personalizadas](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-dynamic-membership#extension-properties-and-custom-extension-properties)

---

### <a name="new-approved-client-apps-for-azure-ad-app-based-conditional-access"></a>Nuevas aplicaciones cliente aprobadas para el acceso condicional basado en aplicaciones de Azure AD

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Acceso condicional Protección y seguridad de la identidad

Las siguientes aplicaciones se incluyen en la lista de [aplicaciones cliente aprobadas](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-conditional-access-conditions#client-apps-preview):

- Microsoft To-Do

- Microsoft Stream

Para más información, consulte:

- [Acceso condicional basado en aplicaciones de Azure AD](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access)

---

### <a name="new-support-for-self-service-password-reset-from-the-windows-7881-lock-screen"></a>Nueva compatibilidad con Autoservicio de restablecimiento de contraseña desde la pantalla de bloqueo de Windows 7/8/8.1

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** SSPR Autenticación de usuarios

Después de configurar esta nueva característica los usuarios verán un vínculo para restablecer su contraseña desde la pantalla de **bloqueo** de un dispositivo que ejecuta Windows 7, Windows 8 o Windows 8.1. Al hacer clic en ese vínculo, se guía al usuario por el mismo flujo de restablecimiento de contraseña que por el explorador web.

Para más información, consulte [How to: Enable password reset from Windows 7, 8, and 8.1](https://aka.ms/ssprforwindows78) (Procedimiento para habilitar el restablecimiento de contraseña desde Windows 7, 8, and 8.1)

---

### <a name="change-notice-authorization-codes-will-no-longer-be-available-for-reuse"></a>Aviso de cambio: Los códigos de autorización ya no estarán disponibles para su reutilización

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

A partir del 15 de noviembre de 2018, Azure AD dejará de aceptar los códigos de autenticación que se usaban anteriormente para las aplicaciones. Este cambio de seguridad ayuda a poner Azure AD en consonancia con la especificación de OAuth y se aplicará en los puntos de conexión v1 y v2.

Si la aplicación reutiliza códigos de autorización para obtener tokens para varios recursos, es recomendable que use el código para obtener un token de actualización y, a continuación, utilice este para adquirir tokens adicionales para otros recursos. Los códigos de autorización solo se pueden usar una vez, pero los tokens de actualización se pueden usar varias veces en varios recursos. Una aplicación que intente reutilizar un código de autenticación durante el flujo de código de OAuth obtendrá el error invalid_grant.

Para este y otros cambios relacionados con los protocolos, consulte [la lista completa de las novedades de la autenticación](https://docs.microsoft.com/azure/active-directory/develop/reference-breaking-changes).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---september-2018"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: septiembre de 2018

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En septiembre de 2018, hemos agregado 16 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[Uberflip](https://docs.microsoft.com/azure/active-directory/saas-apps/uberflip-tutorial), [Comeet Recruiting Software](https://docs.microsoft.com/azure/active-directory/saas-apps/comeetrecruitingsoftware-tutorial), [Workteam](https://docs.microsoft.com/azure/active-directory/saas-apps/workteam-tutorial), [ArcGIS Enterprise](https://docs.microsoft.com/azure/active-directory/saas-apps/arcgisenterprise-tutorial), [Nuclino](https://docs.microsoft.com/azure/active-directory/saas-apps/nuclino-tutorial), [JDA Cloud](https://docs.microsoft.com/azure/active-directory/saas-apps/jdacloud-tutorial), [Snowflake](https://docs.microsoft.com/azure/active-directory/saas-apps/snowflake-tutorial), NavigoCloud, [Figma](https://docs.microsoft.com/azure/active-directory/saas-apps/figma-tutorial), join.me, [ZephyrSSO](https://docs.microsoft.com/azure/active-directory/saas-apps/zephyrsso-tutorial), [Silverback](https://docs.microsoft.com/azure/active-directory/saas-apps/silverback-tutorial), Riverbed Xirrus EasyPass, [Rackspace SSO](https://docs.microsoft.com/azure/active-directory/saas-apps/rackspacesso-tutorial), Enlyft SSO for Azure, SurveyMonkey, [Convene](https://docs.microsoft.com/azure/active-directory/saas-apps/convene-tutorial) y [dmarcian](https://docs.microsoft.com/azure/active-directory/saas-apps/dmarcian-tutorial)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="support-for-additional-claims-transformations-methods"></a>Compatibilidad con métodos de transformación de notificaciones adicionales

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales SSO

Hemos introducido nuevos métodos de transformación de notificaciones, ToLower() y ToUpper(), que se pueden aplicar a los tokens SAML desde la página **Configuración del inicio de sesión único** basada en SAML.

Para más información, consulte [Procedimientos para personalizar las notificaciones emitidas en el token SAML para aplicaciones empresariales en Azure AD](https://docs.microsoft.com/azure/active-directory/develop/active-directory-saml-claims-customization)

---

### <a name="updated-saml-based-app-configuration-ui-preview"></a>Interfaz de usuario de la configuración de aplicaciones basadas en SAML actualizada (versión preliminar)

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Aplicaciones empresariales SSO

Como parte de nuestra interfaz de usuario de la configuración de aplicaciones basadas en SAML actualizada, obtendrá:

- Una experiencia actualizada del tutorial para configurar aplicaciones basadas en SAML.

- Mayor visibilidad de lo que falta o es incorrecto en la configuración.

- La capacidad para agregar varias direcciones de correo electrónico para la notificación de expiración de certificados.

- Nuevos métodos de transformación de notificaciones, ToLower() ToUpper(), y más.

- Una forma de cargar su propio certificado de firma de tokens para las aplicaciones empresariales.

- Una forma de establecer el formato de NameID para aplicaciones SAML y una forma de establecer el valor de NameID como extensiones de directorio.

Para activar esta vista actualizada, haga clic en el vínculo **Try out our new experience** (Probar nuestra nueva experiencia) en la parte superior de la página **Inicio de sesión único**. Para más información, consulte el [Tutorial: Configuración del inicio de sesión único basado en SAML para una aplicación con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/configure-single-sign-on-non-gallery-applications).

---

## <a name="august-2018"></a>Agosto de 2018

### <a name="changes-to-azure-active-directory-ip-address-ranges"></a>Cambios realizados en los intervalos de direcciones IP de Azure Active Directory

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Otros Plataforma

Estamos introduciendo intervalos IP mayores en Azure AD, lo que significa que si ha configurado los intervalos IP de Azure AD para los firewalls, enrutadores o grupos de seguridad de red, deberá actualizarlos. Realizamos esta actualización para que no tenga que volver a cambiar su firewall, enrutador ni las configuraciones de intervalo IP de los grupos de seguridad de red cuando Azure AD agregue nuevos puntos de conexión.

El tráfico se moverá a estos nuevos intervalos durante los próximos dos meses. Para continuar con un servicio ininterrumpido, debe agregar estos valores actualizados a las direcciones IP antes del 10 de septiembre de 2018:

- 20.190.128.0/18

- 40.126.0.0/18

Se recomienda encarecidamente no quitar los intervalos IP antiguos hasta que todo el tráfico se haya movido a los nuevos intervalos. Para actualizaciones sobre el movimiento y saber cuándo quitar los intervalos antiguos, consulte [Office 365 URLs and IP address ranges](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2) (Direcciones URL e intervalos IP de Office 365).

---

### <a name="change-notice-authorization-codes-will-no-longer-be-available-for-reuse"></a>Aviso de cambio: Los códigos de autorización ya no estarán disponibles para su reutilización

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

A partir del 15 de noviembre de 2018, Azure AD dejará de aceptar los códigos de autenticación que se usaban anteriormente para las aplicaciones. Este cambio de seguridad ayuda a poner Azure AD en consonancia con la especificación de OAuth y se aplicará en los puntos de conexión v1 y v2.

Si la aplicación reutiliza códigos de autorización para obtener tokens para varios recursos, es recomendable que use el código para obtener un token de actualización y, a continuación, utilice este para adquirir tokens adicionales para otros recursos. Los códigos de autorización solo se pueden usar una vez, pero los tokens de actualización se pueden usar varias veces en varios recursos. Una aplicación que intente reutilizar un código de autenticación durante el flujo de código de OAuth obtendrá el error invalid_grant.

Para este y otros cambios relacionados con los protocolos, consulte [la lista completa de las novedades de la autenticación](https://docs.microsoft.com/azure/active-directory/develop/reference-breaking-changes).

---

### <a name="converged-security-info-management-for-self-service-password-sspr-and-multi-factor-authentication-mfa"></a>Administración de información de seguridad convergida para restablecimiento de contraseña de autoservicio (SSPR) y Multi-Factor Authentication (MFA)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** SSPR Autenticación de usuarios

Esta nueva característica ayuda a los usuarios a administrar su información de seguridad (por ejemplo, número de teléfono, aplicación móvil, etc.) para SSPR y MFA en una sola ubicación y una misma experiencia, al contrario que sucedía anteriormente cuando se realizaba en dos ubicaciones diferentes.

Esta experiencia convergente también funciona para los usuarios que utilizan MFA o SSPR. Además, aunque la organización no aplique el registro de MFA o de SSPR, los usuarios pueden registrar todos los métodos de información de seguridad que permita la organización desde el portal Mis aplicaciones.

Se trata de una versión preliminar pública opcional. Los administradores pueden activar la nueva experiencia (si lo desean) para un grupo de usuarios seleccionado o para todos los usuarios de un inquilino. Para más información sobre la experiencia convergente, consulte el [blog de experiencia convergente](https://cloudblogs.microsoft.com/enterprisemobility/2018/08/06/mfa-and-sspr-updates-now-in-public-preview/)

---

### <a name="new-http-only-cookies-setting-in-azure-ad-application-proxy-apps"></a>Nueva configuración de las cookies solo HTTP en las aplicaciones de Azure AD Application Proxy

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Proxy de aplicaciones Control de acceso

Hay un nuevo valor llamado, **HTTP-Only Cookies** (Cookies solo HTTP) en las aplicaciones de Application Proxy. Este valor ayuda a proporcionar un nivel extra de seguridad incluyendo la marca HTTPOnly en el encabezado de la respuesta HTTP para las cookies de acceso a Application Proxy y las de la sesión, impidiendo el acceso a cookies procedentes de un script en el lado del cliente e impidiendo acciones como la copia o modificación de la cookie. Aunque esta marca no se ha usado anteriormente, las cookies se han cifrado y transmitido siempre a través de una conexión TLS para proteger frente a modificaciones no adecuadas.

Este valor no es compatible con aplicaciones que usan controles ActiveX, como Escritorio remoto. Si este es su caso, es aconsejable que desactive este valor.

Para más información acerca del valor de cookies solo HTTP, consulte [Publicación de aplicaciones mediante Azure AD Application Proxy](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-publish-azure-portal).

---

### <a name="privileged-identity-management-pim-for-azure-resources-supports-management-group-resource-types"></a>Privileged Identity Management (PIM) para recursos de Azure es compatible con los tipos de recursos del grupo de administración

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Privileged Identity Management Privileged Identity Management

La configuración de la activación y asignación Just-In-Time se puede aplicar ahora a los tipos de recursos del grupo de administración, al igual que ya lo hace con las suscripciones, grupos de recursos y recursos (como máquinas virtuales, App Services, etc). Además, cualquier usuario con un rol que proporcione acceso de administrador para un grupo de administración puede detectar y administrar ese recurso en PIM.

Para más información acerca de PIM y los recursos de Azure, consulte [Detectar y administrar recursos de Azure mediante Privileged Identity Management](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-resource-roles-discover-resources)

---

### <a name="application-access-preview-provides-faster-access-to-the-azure-ad-portal"></a>La característica de acceso a la aplicación (versión preliminar) proporciona un acceso más rápido al portal de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Privileged Identity Management Privileged Identity Management

En la actualidad, al activar un rol con PIM, puede que los permisos tarden más de 10 minutos en surtir efecto. Si opta por usar la característica de acceso a la aplicación, que se encuentra actualmente en versión preliminar pública, los administradores podrán acceder al portal de Azure AD tan pronto como se complete la solicitud de activación.

Actualmente, esta característica solo admite la experiencia del portal de Azure AD y los recursos de Azure. Para más información acerca de PIM y la característica de acceso a la aplicación, consulte [¿Qué es Azure AD Privileged Identity Management?](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-configure)

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---august-2018"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: agosto de 2018

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En agosto de 2018, hemos agregado estas 16 nuevas aplicaciones con compatibilidad con la federación para la galería de aplicaciones:

[Hornbill](https://docs.microsoft.com/azure/active-directory/saas-apps/hornbill-tutorial), [Bridgeline Unbound](https://docs.microsoft.com/azure/active-directory/saas-apps/bridgelineunbound-tutorial), [Sauce Labs - Mobile and Web Testing](https://docs.microsoft.com/azure/active-directory/saas-apps/saucelabs-mobileandwebtesting-tutorial), [Meta Networks Connector](https://docs.microsoft.com/azure/active-directory/saas-apps/metanetworksconnector-tutorial), [Way We Do](https://docs.microsoft.com/azure/active-directory/saas-apps/waywedo-tutorial), [Spotinst](https://docs.microsoft.com/azure/active-directory/saas-apps/spotinst-tutorial), [ProMaster (by Inlogik)](https://docs.microsoft.com/azure/active-directory/saas-apps/promaster-tutorial), SchoolBooking, [4me](https://docs.microsoft.com/azure/active-directory/saas-apps/4me-tutorial), [Dossier](https://docs.microsoft.com/azure/active-directory/saas-apps/DOSSIER-tutorial), [N2F - Expense reports](https://docs.microsoft.com/azure/active-directory/saas-apps/n2f-expensereports-tutorial), [Comm100 Live Chat](https://docs.microsoft.com/azure/active-directory/saas-apps/comm100livechat-tutorial), [SafeConnect](https://docs.microsoft.com/azure/active-directory/saas-apps/safeconnect-tutorial), [ZenQMS](https://docs.microsoft.com/azure/active-directory/saas-apps/zenqms-tutorial), [eLuminate](https://docs.microsoft.com/azure/active-directory/saas-apps/eluminate-tutorial), [Dovetale](https://docs.microsoft.com/azure/active-directory/saas-apps/dovetale-tutorial).

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="native-tableau-support-is-now-available-in-azure-ad-application-proxy"></a>Ahora ya está disponible la compatibilidad nativa con Tableau en Azure AD Application Proxy

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Proxy de aplicaciones Control de acceso

Gracias a la actualización del protocolo de concesión de código de OpenID Connect a OAuth 2.0 para nuestro protocolo de autenticación previa, ya no es necesario realizar ninguna configuración adicional para usar Tableau con Application Proxy. Este cambio de protocolo también ayuda a Application Proxy a mejorar su compatibilidad con aplicaciones más modernas mediante el uso exclusivo de redirecciones HTTP, que normalmente son compatibles con las etiquetas de JavaScript y HTML.

---

### <a name="new-support-to-add-google-as-an-identity-provider-for-b2b-guest-users-in-azure-active-directory-preview"></a>Nueva compatibilidad para agregar Google como proveedor de identidades de usuarios invitados de B2B en Azure Active Directory (versión preliminar)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2B B2B/B2C

Mediante la configuración de la federación con Google en su organización, puede permitir que usuarios invitados de Gmail inicien sesión en sus aplicaciones y recursos compartidos con su cuenta de Google existente sin tener que crear una cuenta Microsoft (MSA) personal o una cuenta de Azure AD.

Se trata de una versión preliminar pública opcional. Para más información sobre la federación con Google, consulte [Incorporación de Google como proveedor de identidades para los usuarios invitados de B2B](https://docs.microsoft.com/azure/active-directory/b2b/google-federation).

---

## <a name="july-2018"></a>Julio de 2018

### <a name="improvements-to-azure-active-directory-email-notifications"></a>Mejoras en las notificaciones de Azure Active Directory por correo electrónico

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Otros Administración del ciclo de vida de la identidad

Los correos electrónicos de Azure Active Directory (Azure AD) ahora tienen un diseño actualizado, además de cambios en el nombre para mostrar del remitente y la dirección de correo electrónico del destinatario cuando los mensajes se envían desde los servicios siguientes:

- Revisiones de acceso de Azure AD
- Azure AD Connect Health
- Azure AD Identity Protection
- Administración de identidades con privilegios de Azure AD
- Aplicación empresarial: expiración de las notificaciones de certificado
- Aplicación empresarial: aprovisionamiento de las notificaciones de servicio

Las notificaciones de correo electrónico se enviarán desde la siguiente dirección de correo electrónico y nombre para mostrar:

- Dirección de correo electrónico: azure-noreply@microsoft.com
- Nombre para mostrar: Microsoft Azure

Para un ejemplo de algunos de los nuevos diseños de correo electrónico y más información, consulte [Email notifications in Azure AD PIM](https://go.microsoft.com/fwlink/?linkid=2005832) (Notificaciones por correo electrónico en Azure AD PIM).

---

### <a name="azure-ad-activity-logs-are-now-available-through-azure-monitor"></a>Los registros de actividad de Azure AD ahora están disponibles a través de Azure Monitor

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Informes Supervisión e informes

Los registros de actividad de Azure AD ahora están disponibles en la versión preliminar pública de Azure Monitor (servicio supervisión en toda la plataforma de Azure). Azure Monitor ofrece una retención a largo plazo e integración sin problemas, además de estas mejoras:

- Retención a largo plazo mediante el enrutamiento de los archivos de registro en su propia cuenta de almacenamiento de Azure.

- Integración SIEM perfecta, sin necesidad de escribir o mantener scripts personalizados.

- Integración sin problemas con sus propias soluciones personalizadas, herramientas de análisis o soluciones de administración de incidentes.

Para más información sobre estas nuevas funcionalidades, consulte nuestro blog [Azure AD activity logs in Azure Monitor diagnostics is now in public preview](https://cloudblogs.microsoft.com/enterprisemobility/2018/07/26/azure-ad-activity-logs-in-azure-monitor-diagnostics-now-in-public-preview/) (Los registros de actividad de Azure AD en diagnósticos de Azure Monitor están ahora en versión preliminar pública) y nuestra documentación, [Registros de actividad de Azure AD en Azure Monitor (versión preliminar)](https://docs.microsoft.com/azure/active-directory/reporting-azure-monitor-diagnostics-overview).

---

### <a name="conditional-access-information-added-to-the-azure-ad-sign-ins-report"></a>Información de acceso condicional agregada al informe de inicios de sesión de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Informes Seguridad y protección de la identidad

Esta actualización le permite ver qué directivas se evalúan cuando un usuario inicia sesión, junto con el resultado de la directiva. Además, el informe ahora incluye el tipo de aplicación cliente que el usuario utiliza, de manera que usted puede identificar el tráfico de protocolo heredado. Ahora también se pueden buscar entradas de informe para un identificador de correlación, que puede encontrarse en el mensaje de error de cara al usuario y puede usarse para identificar la solicitud de inicio de sesión coincidente y solucionar problemas en dicha solicitud.

---

### <a name="view-legacy-authentications-through-sign-ins-activity-logs"></a>Visualización de autenticaciones heredadas a través de los registros de actividad de inicios de sesión

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Informes Supervisión e informes

Con la introducción del campo **Aplicación cliente** en los registros de actividad de inicio de sesión, los clientes ya pueden ver los usuarios que utilizan autenticaciones heredadas. Los clientes podrán acceder a esta información mediante Microsoft Graph API de los registros de inicio de sesión o mediante los registros de actividad de inicio de sesión del portal Azure AD, donde se puede usar el control **Aplicación cliente** para filtrar por autenticaciones heredadas. Para más información, consulte la documentación.

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---july-2018"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: julio de 2018

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En julio de 2018, hemos agregado estas 16 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones:

[Innovation Hub](https://docs.microsoft.com/azure/active-directory/saas-apps/innovationhub-tutorial), [Leapsome](https://docs.microsoft.com/azure/active-directory/saas-apps/leapsome-tutorial), [Certain Admin SSO](https://docs.microsoft.com/azure/active-directory/saas-apps/certainadminsso-tutorial), PSUC Staging, [iPass SmartConnect](https://docs.microsoft.com/azure/active-directory/saas-apps/ipasssmartconnect-tutorial), [Screencast-O-Matic](https://docs.microsoft.com/azure/active-directory/saas-apps/screencast-tutorial), PowerSchool Unified Classroom, [Eli Onboarding](https://docs.microsoft.com/azure/active-directory/saas-apps/elionboarding-tutorial), [Bomgar Remote Support](https://docs.microsoft.com/azure/active-directory/saas-apps/bomgarremotesupport-tutorial), [Nimblex](https://docs.microsoft.com/azure/active-directory/saas-apps/nimblex-tutorial), [Imagineer WebVision](https://docs.microsoft.com/azure/active-directory/saas-apps/imagineerwebvision-tutorial), [Insight4GRC](https://docs.microsoft.com/azure/active-directory/saas-apps/insight4grc-tutorial), [SecureW2 JoinNow Connector](https://docs.microsoft.com/azure/active-directory/saas-apps/securejoinnow-tutorial), [Kanbanize](../saas-apps/kanbanize-tutorial.md), [SmartLPA](../saas-apps/smartlpa-tutorial.md) y [Skills Base](https://docs.microsoft.com/azure/active-directory/saas-apps/skillsbase-tutorial).

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="new-user-provisioning-saas-app-integrations---july-2018"></a>Nuevas integraciones de aplicaciones de SaaS de aprovisionamiento de usuarios: julio de 2018

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aprovisionamiento de aplicaciones Integración de terceros

Azure AD permite automatizar la creación, el mantenimiento y la eliminación de identidades de usuario en aplicaciones de SaaS, como Dropbox, Salesforce, ServiceNow, etc. En julio de 2018, hemos agregado compatibilidad con el aprovisionamiento de usuarios a las siguientes aplicaciones de la Galería de aplicaciones de Azure AD:

- [Cisco WebEx](https://docs.microsoft.com/azure/active-directory/saas-apps/cisco-webex-provisioning-tutorial)

- [Bonusly](https://docs.microsoft.com/azure/active-directory/saas-apps/bonusly-provisioning-tutorial)

Para una lista de todas las aplicaciones que admiten el aprovisionamiento de usuarios en la Galería de Azure AD, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial).

---

### <a name="connect-health-for-sync---an-easier-way-to-fix-orphaned-and-duplicate-attribute-sync-errors"></a>Connect Health para sincronización: una manera más sencilla de corregir errores de sincronización de atributos duplicados o huérfanas

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Supervisión e informes

Azure AD Connect Health presenta la corrección de autoservicio para ayudarle a resaltar y corregir errores de sincronización. Esta característica soluciona los errores de sincronización de atributos duplicados y repara los objetos huérfanos desde Azure AD. Este diagnóstico tiene las siguientes ventajas:

- Reduce los errores de sincronización de atributos duplicado, proporcionando correcciones específicas

- Aplica una corrección para escenarios de Azure AD dedicados, resolviendo los errores en un solo paso dedicado

- No es necesaria ninguna actualización o configuración para activar y usar esta característica

Para más información, consulte [Diagnóstico y solución de errores de sincronización de atributos duplicados](https://docs.microsoft.com/azure/active-directory/connect-health/active-directory-aadconnect-health-diagnose-sync-errors).

---

### <a name="visual-updates-to-the-azure-ad-and-msa-sign-in-experiences"></a>Actualizaciones de objetos visuales en Azure AD y experiencias de inicio de sesión de MSA

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Azure AD Autenticación de usuarios

Hemos actualizado la interfaz de usuario de la experiencia de inicio de sesión de los servicios en línea de Microsoft, como Office 365 y Azure. Este cambio hace que las pantallas estén menos saturadas y sean más sencillas. Para más información sobre este cambio, consulte el blog [Upcoming improvements to the Azure AD sign-in experience](https://cloudblogs.microsoft.com/enterprisemobility/2018/04/04/upcoming-improvements-to-the-azure-ad-sign-in-experience/) (Próximas mejoras en la experiencia de inicio de sesión de Azure AD).

---

### <a name="new-release-of-azure-ad-connect---july-2018"></a>Nueva versión de Azure AD Connect: julio de 2018

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Aprovisionamiento de aplicaciones Administración del ciclo de vida de la identidad

La versión más reciente de Azure AD Connect incluye:

- Correcciones de errores y actualizaciones de compatibilidad

- Disponibilidad general de la integración PingFederate

- Actualizaciones en el cliente más reciente de SQL 2012

Para más información sobre esta actualización, consulte [Azure AD Connect: historial de versiones](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-version-history)

---

### <a name="updates-to-the-terms-of-use-end-user-ui"></a>Actualizaciones en la interfaz de usuario del usuario final correspondiente a los términos de uso

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Términos de uso Gobernanza

Estamos actualizando la cadena de aceptación en la interfaz de usuario del usuario final correspondiente a las condiciones de uso.

**Texto actual.** Para acceder a los recursos de [nombreInquilino], debe aceptar las condiciones de uso.<br>**Nuevo texto.** Para acceder al recurso de [nombreInquilino], debe leer las condiciones de uso.

**Texto actual:** Si elige Aceptar, significa que acepta todas las condiciones de uso anteriores.<br>**Nuevo texto:** Haga clic en Aceptar para confirmar que ha leído y comprendido las condiciones de uso.

---

### <a name="pass-through-authentication-supports-legacy-protocols-and-applications"></a>La autenticación de paso a través es compatible con las aplicaciones y protocolos heredados

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

La autenticación de paso a través es compatible con las aplicaciones y los protocolos heredados. Ahora se admiten completamente las siguientes limitaciones:

- Inicios de sesión de usuario en las aplicaciones cliente de Office heredadas, Office 2010 y Office 2013, sin necesidad de autenticación moderna.

- Acceso a uso compartido del calendario e información de disponibilidad en entornos híbridos de Exchange solo en Office 2010.

- Inicios de sesión de usuarios en aplicaciones cliente de Skype Empresarial sin necesidad de autenticación moderna.

- Inicios de sesión de usuario en PowerShell 1.0.

- El Programa de inscripción de dispositivos de Apple (DEP de Apple), mediante el Asistente para configuración de iOS.

---

### <a name="converged-security-info-management-for-self-service-password-reset-and-multi-factor-authentication"></a>Administración de información de seguridad convergida para restablecimiento de contraseña de autoservicio y Multi-Factor Authentication

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** SSPR Autenticación de usuarios

Esta nueva característica permite a los usuarios administrar su información de seguridad (por ejemplo, número de teléfono, dirección de correo electrónico, aplicación móvil, etc.) para el restablecimiento de contraseña de autoservicio (SSPR) y Multi-Factor Authentication (MFA) en una sola experiencia. Los usuarios ya no tendrán que registrar la misma información de seguridad para SSPR y MFA en dos experiencias diferentes. Esta nueva experiencia también se aplica a los usuarios que tienen SSPR o MFA.

Si una organización no aplica el registro MFA o SSPR, los usuarios pueden registrar su información de seguridad a través del portal **Mis aplicaciones**. Desde allí, los usuarios pueden registrar los métodos habilitados para MFA o SSPR.

Se trata de una versión preliminar pública opcional. Los administradores pueden activar la nueva experiencia (si lo desean) para un grupo de usuarios seleccionado o para todos los usuarios en un inquilino.

---

### <a name="use-the-microsoft-authenticator-app-to-verify-your-identity-when-you-reset-your-password"></a>Uso de la aplicación Microsoft Authenticator para verificar su identidad cuando restablezca la contraseña

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** SSPR Autenticación de usuarios

Esta característica permite a los no administradores verificar su identidad al restablecer una contraseña mediante una notificación o un código de Microsoft Authenticator (o cualquier otra aplicación de autenticador). Una vez que los administradores activan este método de restablecimiento de contraseña de autoservicio, los usuarios que hayan registrado una aplicación móvil a través de aka.ms/mfasetup o aka.ms/setupsecurityinfo pueden usar su aplicación móvil como método de verificación al restablecer su contraseña.

La notificación de la aplicación móvil solo se puede activar como parte de una directiva que requiere dos métodos para restablecer la contraseña.

---

## <a name="june-2018"></a>Junio de 2018

### <a name="change-notice-security-fix-to-the-delegated-authorization-flow-for-apps-using-azure-ad-activity-logs-api"></a>Aviso de cambio: Revisión de seguridad para el flujo de autorización delegado para aplicaciones que usan la API de registro de actividad de Azure AD

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto**: Informes Supervisión e informes

Debido a nuestras medidas de seguridad más estrictas, hemos tenido que realizar un cambio en los permisos de las aplicaciones que usan un flujo de autorización delegado para acceder a la [API de registro de actividad de Azure AD](https://aka.ms/aadreportsapi). Este cambio se producirá el **26 de junio de 2018**.

Si alguna de sus aplicaciones usa la API de registro de actividad de Azure AD, siga estos pasos para asegurarse de que la aplicación no se interrumpa después de que se produzca el cambio.

**Para actualizar los permisos de la aplicación**

1. Inicie sesión en Azure Portal, seleccione **Azure Active Directory** y, a continuación, **Registros de aplicaciones**.
2. Seleccione la aplicación que usa la API de registro de actividad de Azure AD, seleccione **Configuración**, **Permisos necesarios** y, a continuación, seleccione la API de **Microsoft Azure Active Directory**.
3. En el área **Permisos delegados** de la hoja **Habilitar acceso**, active la casilla junto a **Leer datos de directorio** y seleccione **Guardar**.
4. Haga clic en **Conceder permisos** y, a continuación, haga clic en **Sí**.

    >[!Note]
    >Debe ser un administrador global para conceder permisos a la aplicación.

Para obtener más información, consulte el área [Conceder permisos](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-api-prerequisites-azure-portal#grant-permissions) de los requisitos previos para obtener acceso al artículo de la API de generación de informes de Azure AD.

---

### <a name="configure-tls-settings-to-connect-to-azure-ad-services-for-pci-dss-compliance"></a>Configuración de las opciones de TLS para conectarse a servicios de Azure AD para el cumplimiento de PCI DSS

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** N/D Plataforma

La Seguridad de la capa de transporte (TLS) es un protocolo que proporciona privacidad e integridad de datos para la comunicación entre dos aplicaciones, y es el protocolo de seguridad más implementado hoy en día.

El [PCI Security Standards Council](https://www.pcisecuritystandards.org/) (Consejo de estándares de seguridad de PCI) ha determinado que las versiones anteriores de TLS y Capa de sockets seguros (SSL) deben deshabilitarse para habilitar protocolos de aplicaciones nuevos y más seguros, con el inicio del cumplimiento a partir del **30 de junio 2018**. Este cambio significa que, si se conecta a servicios de Azure AD y es necesario el cumplimiento con PCI DSS, deberá deshabilitar TLS 1.0. Hay varias versiones de TLS disponibles, pero TLS 1.2 es la más reciente para servicios de Azure Active Directory. Se recomienda encarecidamente moverse directamente a TLS 1.2 para las combinaciones de cliente/servidor y explorador/servidor.

Es posible que los exploradores obsoletos no admitan las versiones más recientes de TLS, como TLS 1.2. Para ver qué versiones de TLS son compatibles con el explorador, vaya al sitio de [Qualys SSL Labs](https://www.ssllabs.com/) y haga clic en **Test your browser** (Probar el explorador). Le recomendamos que actualice a la versión más reciente del explorador web y, si es posible,que habilite solo TLS 1.2.

**Para habilitar TLS 1.2 por explorador**

- **Microsoft Edge e Internet Explorer (ambos se establecen mediante Internet Explorer)**

    1. Abra Internet Explorer, seleccione **Herramientas** > **Opciones de Internet** > **Opciones avanzadas**.
    2. En el área de **Seguridad**, seleccione **use TLS 1.2** (usar TLS 1.2) y, a continuación, seleccione **Aceptar**.
    3. Cierre todas las ventanas del explorador y reinicie Internet Explorer.

- **Google Chrome**

    1. Abra Google Chrome, escriba *chrome://settings/* en la barra de direcciones y presione **ENTRAR**.
    2. Expanda la **Configuración avanzada**, vaya al área de **Sistema** y seleccione **Abrir la configuración de proxy**.
    3. En el cuadro **Propiedades: Internet**, seleccione la pestaña **Opciones avanzadas**, vaya al área de **Seguridad**, seleccione **Usar TLS 1.2** y, a continuación, seleccione  **Aceptar**.
    4. Cierre todas las ventanas del explorador y reinicie Google Chrome.

- **Mozilla Firefox**

    1. Abra Firefox, escriba *about:config* en la barra de direcciones y, a continuación, presione **ENTRAR**.
    2. Busque el término *TLS* y, a continuación, seleccione la entrada **security.tls.version.max**.
    3. Establezca el valor en **3** para forzar al explorador a usar hasta la versión TLS 1.2 y, a continuación, seleccione **Aceptar**.

        >[!NOTE]
        >La versión de Firefox 60.0 admite TLS 1.3, por lo que también puede establecer el valor de security.tls.version.max en **4**.

    4. Cierre todas las ventanas del explorador y reinicie Mozilla Firefox.

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---june-2018"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: junio de 2018

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En junio de 2018, hemos agregado estas 15 nuevas aplicaciones con compatibilidad con la federación para la galería de aplicaciones:

[Skytap](https://docs.microsoft.com/azure/active-directory/active-directory-saas-skytap-tutorial), [Settling music](https://docs.microsoft.com/azure/active-directory/active-directory-saas-settlingmusic-tutorial), [aplicación de LOB con el token SAML 1.1 habilitado](https://docs.microsoft.com/azure/active-directory/active-directory-saas-saml-tutorial), [Supermood](https://docs.microsoft.com/azure/active-directory/active-directory-saas-supermood-tutorial), [Autotask](https://docs.microsoft.com/azure/active-directory/active-directory-saas-autotaskendpointbackup-tutorial), [Endpoint Backup](https://docs.microsoft.com/azure/active-directory/active-directory-saas-autotaskendpointbackup-tutorial), [Skyhigh Networks](https://docs.microsoft.com/azure/active-directory/active-directory-saas-skyhighnetworks-tutorial), Smartway2, [TonicDM](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tonicdm-tutorial), [Moconavi](https://docs.microsoft.com/azure/active-directory/active-directory-saas-moconavi-tutorial), [Zoho One](https://docs.microsoft.com/azure/active-directory/active-directory-saas-zohoone-tutorial), [SharePoint local](https://docs.microsoft.com/azure/active-directory/active-directory-saas-sharepoint-on-premises-tutorial), [ForeSee CX Suite](https://docs.microsoft.com/azure/active-directory/active-directory-saas-foreseecxsuite-tutorial), [Vidyard](https://docs.microsoft.com/azure/active-directory/active-directory-saas-vidyard-tutorial) y [ChronicX](https://docs.microsoft.com/azure/active-directory/active-directory-saas-chronicx-tutorial)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-app-gallery-listing).

---

### <a name="azure-ad-password-protection-is-available-in-public-preview"></a>La Protección con contraseña de Azure AD está disponible en versión preliminar pública.

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Identity Protection Autenticación de usuarios

Use la Protección con contraseña de Azure AD para ayudar a eliminar de su entorno las contraseñas que se pueden averiguar fácilmente. Al eliminar estas contraseñas, ayuda a reducir el riesgo de un tipo de ataque de difusión de contraseña.

En concreto, la Protección con contraseña de Azure AD:

- Le ayuda a proteger las cuentas de la organización en Azure AD y Windows Server Active Directory (AD).
- Impide que los usuarios usen las contraseñas de una lista que incluye más de 500 de las contraseñas utilizadas con más frecuencia y más de un millón de variaciones de sustitución de caracteres de esas contraseñas.
- Le ayuda a administrar la Protección con contraseña de Azure AD desde una única ubicación en el portal de Azure AD, tanto para Azure AD como para Windows Server AD local.

Para obtener más información acerca de la Protección con contraseña de Azure AD, consulte [Elimine las contraseñas incorrectas de su organización](https://aka.ms/aadpasswordprotectiondocs).

---

### <a name="new-all-guests-conditional-access-policy-template-created-during-terms-of-use-creation"></a>Nueva plantilla "todos los invitados" de la directiva de acceso condicional creada durante la creación de los términos de uso

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Gobernanza

Durante la creación de los términos de uso, también se ha creado una nueva plantilla de la directiva de acceso condicional para "todos los invitados" y "todas las aplicaciones". Esta nueva plantilla de la directiva se aplica a los Términos de uso recién creados, lo que facilita el proceso de cumplimiento y creación para los invitados.

Para obtener más información, consulte [Característica Azure Active Directory Terms of Use](https://docs.microsoft.com/azure/active-directory/conditional-access/terms-of-use).

---

### <a name="new-custom-conditional-access-policy-template-created-during-terms-of-use-creation"></a>Nueva plantilla "personalizado" de la directiva de acceso condicional creada durante la creación de los términos de uso

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Gobernanza

Durante la creación de los términos de uso, también se ha creado una nueva plantilla "personalizada" de la directiva de acceso condicional. Esta nueva plantilla de la directiva le permite crear los términos de uso y luego pasar inmediatamente a la hoja de creación de la directiva de acceso condicional, sin necesidad de navegar por el portal de forma manual.

Para obtener más información, consulte [Característica Azure Active Directory Terms of Use](https://docs.microsoft.com/azure/active-directory/conditional-access/terms-of-use).

---

### <a name="new-and-comprehensive-guidance-about-deploying-azure-multi-factor-authentication"></a>Guía nueva y completa acerca de cómo implementar Azure Multi-Factor Authentication

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Otros Seguridad y protección de la identidad

Hemos lanzado la nueva guía paso a paso acerca de cómo implementar Azure Multi-Factor Authentication (MFA) en su organización.

Para ver la guía de implementación de MFA, vaya al repositorio [Guías de implementación de identidad](https://aka.ms/DeploymentPlans) en GitHub. Para enviar comentarios acerca de las guías de implementación, use el [formulario Comentarios del plan de implementación](https://aka.ms/deploymentplanfeedback). Si tiene alguna pregunta acerca de las guías de implementación, póngase en contacto con nosotros en [IDGitDeploy](mailto:idgitdeploy@microsoft.com).

---

### <a name="azure-ad-delegated-app-management-roles-are-in-public-preview"></a>Los roles de administración de aplicaciones delegadas de Azure AD están en versión preliminar pública.

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Control de acceso

Ahora los administradores pueden delegar tareas de administración de aplicaciones sin asignar el rol de administrador global. Los nuevos roles y funcionalidades son:

- **Nuevos roles de administrador de Azure AD estándar:**

    - **Administrador de aplicaciones** Concede la capacidad de administrar todos los aspectos de todas las aplicaciones, incluidos el registro, la configuración de SSO, las asignaciones de aplicaciones y licencias, la configuración del proxy de aplicación y el consentimiento (excepto para los recursos de Azure AD).

    - **Administrador de aplicaciones en la nube** Concede todas las capacidades de administrador de aplicaciones, excepto para el proxy de aplicación porque no proporciona acceso local.

    - **Desarrollador de aplicaciones** Concede permisos para crear registros de aplicaciones, incluso si está desactivada la opción **allow users to register apps** (permitir a los usuarios registrar aplicaciones).

- **Propiedad (configurar el registro por aplicación y la aplicación por empresa, similar al proceso de la propiedad de grupo:**

    - **Propietario de registro de aplicaciones** Concede la capacidad de administrar todos los aspectos del registro de aplicaciones en propiedad, incluidos el manifiesto de la aplicación y la adición de más propietarios.

    - **Propietario de aplicaciones empresariales** Concede la capacidad de administrar muchos aspectos de las aplicaciones empresariales en propiedad, incluidos el consentimiento, la configuración de SSO y las asignaciones de aplicaciones (excepto para los recursos de Azure AD).

Para obtener más información acerca de la versión preliminar pública, consulte el blog [Azure AD delegated application management roles are in public preview!](https://cloudblogs.microsoft.com/enterprisemobility/2018/06/13/hallelujah-azure-ad-delegated-application-management-roles-are-in-public-preview/) (Los roles de administración de aplicaciones delegadas de Azure AD están en versión preliminar pública). . Para obtener más información acerca de los roles y permisos, consulte [Assigning administrator roles in Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-assign-admin-roles-azure-portal) (Asignación de roles de administrador en Azure Active Directory).

---

## <a name="may-2018"></a>Mayo de 2018

### <a name="expressroute-support-changes"></a>Cambios de soporte técnico de ExpressRoute

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Plataforma

Las ofertas de software como servicio, como Azure Active Directory (Azure AD) están diseñadas para funcionar mejor a través de Internet, sin necesidad de ExpressRoute ni otros túneles VPN privados. Por este motivo, en **1 de agosto de 2018**, dejaremos de dar soporte técnico a ExpressRoute para los servicios de Azure AD que usen un emparejamiento público de Azure y a las comunidades de Azure que usen emparejamientos de Microsoft. Los servicios afectados por este cambio podrían observar que el tráfico de Azure AD gradualmente cambia de ExpressRoute a Internet.

Aunque vamos a cambiar el soporte, también sabemos que aún hay situaciones en las que tendrá que utilizar un conjunto de circuitos dedicado para el tráfico de autenticación. Por este motivo, Azure AD seguirá admitiendo restricciones de intervalos de IP por inquilino utilizando ExpressRoute y los servicios que ya tienen emparejamiento de Microsoft con la comunidad "Otros servicios en línea de Office 365". Si los servicios se ven afectados pero requieren ExpressRoute, debe hacer lo siguiente:

- **Si está en un emparejamiento público de Azure.** Cambie a un emparejamiento de Microsoft y suscríbase a la comunidad **Otros servicios en línea de Office 365 (12076:5100)** . Para más información sobre cómo cambiar de emparejamiento público de Azure a emparejamiento de Microsoft, consulte el artículo [Cambiar un emparejamiento público a emparejamiento de Microsoft](https://docs.microsoft.com/azure/expressroute/how-to-move-peering).

- **Si está en un emparejamiento de Microsoft.** Suscríbase a la comunidad **Otros servicio en línea de Office 365 (12076:5100)** . Para más información sobre los requisitos de enrutamiento, consulte la sección [Soporte técnico para las comunidades de BGP](https://docs.microsoft.com/azure/expressroute/expressroute-routing#bgp) en el artículo de requisitos de enrutamiento de ExpressRoute.

Si debe continuar usando circuitos dedicados, deberá hablar con el equipo de su cuenta Microsoft acerca de cómo obtener autorización para utilizar la comunidad **Otros servicios en línea de Office 365 (12076:5100)** . El comité de revisión administrado por MS Office comprobará si necesita los circuitos y se asegurará de que comprende las implicaciones técnicas de su conservación. Las suscripciones no autorizadas que intenten crear filtros de ruta para Office 365 recibirán un mensaje de error.

---

### <a name="microsoft-graph-apis-for-administrative-scenarios-for-tou"></a>Microsoft Graph API para escenarios administrativos de términos de uso

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Experiencia para el desarrollador

Hemos agregado Microsoft Graph API para el funcionamiento de la administración de los términos de uso de Azure AD. Puede crear, actualizar y eliminar el objeto de términos de uso.

---

### <a name="add-azure-ad-multi-tenant-endpoint-as-an-identity-provider-in-azure-ad-b2c"></a>Agregar punto de conexión multiinquilino de Azure AD como proveedor de identidades en Azure AD B2C

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Con las directivas personalizadas, ahora puede agregar el punto de conexión común de Azure AD como proveedor de identidades en Azure AD B2C. Esto le permite tener un único punto de entrada para todos los usuarios de Azure AD que inician sesión en sus aplicaciones. Para más información, consulte [Azure Active Directory B2C: Permitir que los usuarios inicien sesión en un proveedor de identidades de Azure AD multiinquilino mediante directivas personalizadas](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-setup-commonaad-custom).

---

### <a name="use-internal-urls-to-access-apps-from-anywhere-with-our-my-apps-sign-in-extension-and-the-azure-ad-application-proxy"></a>Uso de direcciones URL internas para acceder a aplicaciones desde cualquier lugar con My Apps Sign-in Extension y Azure AD Application Proxy

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Mis aplicaciones SSO

Los usuarios ya pueden acceder a las aplicaciones a través de direcciones URL internas, incluso desde fuera de la red corporativa, mediante My Apps Sign-in Extension para Azure AD. Esto funcionará con todas las aplicaciones que se ha publicado con Azure Active Directory Application Proxy, en cualquier explorador que también tenga instalada la extensión de explorador del Panel de acceso. La funcionalidad de redireccionamiento de direcciones URL se habilita automáticamente una vez que un usuario inicia sesión en la extensión. La extensión se puede descargar en [Microsoft Edge](https://go.microsoft.com/fwlink/?linkid=845176), [Chrome](https://go.microsoft.com/fwlink/?linkid=866367) y [Firefox](https://go.microsoft.com/fwlink/?linkid=866366).

---

### <a name="azure-active-directory---data-in-europe-for-europe-customers"></a>Azure Active Directory: datos en Europa para los clientes europeos

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Otros GoLocal

Los clientes de Europa requieren que sus datos se queden en Europa y no se repliquen fuera de los centros de datos europeos para proteger la privacidad y cumplir la legislación europea. En este [artículo](https://go.microsoft.com/fwlink/?linkid=872328) se proporcionan detalles concretos no solo acerca de qué información de identidad se almacenará en Europa, sino también acerca de la que se almacenará fuera de los centros de datos europeos.

---

### <a name="new-user-provisioning-saas-app-integrations---may-2018"></a>Nuevas integraciones de aplicaciones de SaaS de aprovisionamiento de usuarios: mayo de 2018

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aprovisionamiento de aplicaciones Integración de terceros

Azure AD permite automatizar la creación, el mantenimiento y la eliminación de identidades de usuario en aplicaciones de SaaS, como Dropbox, Salesforce, ServiceNow, etc. En mayo de 2018, hemos agregado compatibilidad con el aprovisionamiento de usuarios a las siguientes aplicaciones de la Galería de aplicaciones de Azure AD:

- [BlueJeans](https://docs.microsoft.com/azure/active-directory/active-directory-saas-bluejeans-provisioning-tutorial)

- [Cornerstone OnDemand](https://docs.microsoft.com/azure/active-directory/active-directory-saas-cornerstone-ondemand-provisioning-tutorial)

- [Zendesk](https://docs.microsoft.com/azure/active-directory/active-directory-saas-zendesk-provisioning-tutorial)

Para obtener una lista de todas las aplicaciones que admiten el aprovisionamiento de usuarios en la Galería de Azure AD, consulte [https://aka.ms/appstutorial](https://aka.ms/appstutorial).

---

### <a name="azure-ad-access-reviews-of-groups-and-app-access-now-provides-recurring-reviews"></a>Ahora, las revisiones de acceso de Azure AD de grupos y aplicaciones proporcionan revisiones periódicas

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Revisiones de acceso Gobernanza

La revisión de acceso de grupos y aplicaciones ya está disponible de forma generalizada como parte de Azure AD Premium P2.  Los administradores podrán configurar las revisiones de acceso de los miembros de un grupo y las asignaciones de aplicaciones para que se repitan automáticamente a intervalos periódicos; por ejemplo, cada mes o cada trimestre.

---

### <a name="azure-ad-activity-logs-sign-ins-and-audit-are-now-available-through-ms-graph"></a>Los registros de actividad de Azure AD (inicios de sesión y auditoría) ahora están disponibles a través de MS Graph

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Informes Supervisión e informes

Los registros de actividad de Azure AD, que incluyen los registros inicios de sesión y de auditorías, ahora están disponibles a través de Microsoft Graph API. Para acceder a dichos registros se han expuesto dos puntos de conexión a través de Microsoft Graph API. Consulte nuestros [documentos](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-api-getting-started-azure-portal) para ver cómo acceder mediante programación a las API de informes de Azure AD.

---

### <a name="improvements-to-the-b2b-redemption-experience-and-leave-an-org"></a>Mejoras en la experiencia de canje de invitaciones B2B y en el abandono de una organización

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2B B2B/B2C

**Canje justo a tiempo:** una vez que se comparte un recurso con un usuario invitado con la API B2B, no es preciso enviar un correo electrónico de invitación especial. En la mayoría de los casos, el usuario invitado puede acceder al recurso y pasar por la experiencia de canje justo a tiempo. Se acabaron los problemas por mensajes de correo electrónico perdidos. Se acabó preguntar a los usuarios invitados: "¿Ha hecho clic en el vínculo de canje que le envió el sistema?". Esto significa que una vez que la SPO utiliza el administrador de invitaciones, los elementos adjuntos de la nube pueden tener la misma dirección URL canónica para todos los usuarios, tanto internos como externos, con cualquier estado de canje.

**Experiencia de canje moderna:** se acabaron las páginas de aterrizaje para el canje con pantalla dividida. Los usuarios verán un consentimiento moderno con la declaración de privacidad de la organización que realiza la invitación, igual que en las aplicaciones de terceros.

**Los usuarios invitados pueden abandonar la organización:** Cuando termina la relación de un usuario con una organización, este puede salir de ella sin ayuda de nadie. No será preciso pedir al administrador de la organización que le invitó que lo "elimine" ni necesitará generar incidencias de soporte técnico.

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---may-2018"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: mayo de 2018

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En mayo de 2018, hemos agregado estas 18 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones:

[AwardSpring](https://docs.microsoft.com/azure/active-directory/active-directory-saas-awardspring-tutorial), Infogix Data3Sixty Govern, [Yodeck](https://docs.microsoft.com/azure/active-directory/active-directory-saas-infogix-tutorial), [Jamf Pro](https://docs.microsoft.com/azure/active-directory/active-directory-saas-jamfprosamlconnector-tutorial), [KnowledgeOwl](https://docs.microsoft.com/azure/active-directory/active-directory-saas-knowledgeowl-tutorial), [Envi MMIS](https://docs.microsoft.com/azure/active-directory/active-directory-saas-envimmis-tutorial), [LaunchDarkly](https://docs.microsoft.com/azure/active-directory/active-directory-saas-launchdarkly-tutorial), [Adobe Captivate Prime](https://docs.microsoft.com/azure/active-directory/active-directory-saas-adobecaptivateprime-tutorial), [Montage Online](https://docs.microsoft.com/azure/active-directory/active-directory-saas-montageonline-tutorial), [まなびポケット](https://docs.microsoft.com/azure/active-directory/active-directory-saas-manabipocket-tutorial), OpenReel, [Arc Publishing - SSO](https://docs.microsoft.com/azure/active-directory/active-directory-saas-arc-tutorial), [PlanGrid](https://docs.microsoft.com/azure/active-directory/active-directory-saas-plangrid-tutorial), [iWellnessNow](https://docs.microsoft.com/azure/active-directory/active-directory-saas-iwellnessnow-tutorial), [Proxyclick](https://docs.microsoft.com/azure/active-directory/active-directory-saas-proxyclick-tutorial), [Riskware](https://docs.microsoft.com/azure/active-directory/active-directory-saas-riskware-tutorial), [Flock](https://docs.microsoft.com/azure/active-directory/active-directory-saas-flock-tutorial), [Reviewsnap](https://docs.microsoft.com/azure/active-directory/active-directory-saas-reviewsnap-tutorial)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial).

Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-app-gallery-listing).

---

### <a name="new-step-by-step-deployment-guides-for-azure-active-directory"></a>Nuevas guías de implementación paso a paso para Azure Active Directory

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Otros Directorio

Novedad: guía paso a paso sobre cómo implementar Azure Active Directory (Azure AD), incluido el autoservicio de restablecimiento de contraseña (SSPR), el inicio de sesión único (SSO), el acceso condicional (CA), el proxy de aplicación, el aprovisionamiento de usuarios, los Servicios de federación de Active Directory (AD FS) en Autenticación de paso a través (PTA) y ADFS para realizar la sincronización de hash de contraseña (PBS).

Para ver las guías de implementación, vaya al repositorio [Guías de implementación de identidad](https://aka.ms/DeploymentPlans) en GitHub. Para enviar comentarios acerca de las guías de implementación, use el [formulario Comentarios del plan de implementación](https://aka.ms/deploymentplanfeedback). Si tiene alguna pregunta acerca de las guías de implementación, póngase en contacto con nosotros en [IDGitDeploy](mailto:idgitdeploy@microsoft.com).

---

### <a name="enterprise-applications-search---load-more-apps"></a>Búsqueda de aplicaciones empresariales: cargar más aplicaciones

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales SSO

¿No consigue encontrar aplicaciones o entidades de servicio? Hemos agregado la posibilidad de cargar más aplicaciones en la lista de todas las aplicaciones empresariales. De manera predeterminada, se muestran 20 aplicaciones. Ahora puede hacer clic en **Cargar más** y ver otras aplicaciones.

---

### <a name="the-may-release-of-aadconnect-contains-a-public-preview-of-the-integration-with-pingfederate-important-security-updates-many-bug-fixes-and-new-great-new-troubleshooting-tools"></a>La versión de mayo de AADConnect contiene una versión preliminar pública de la integración con PingFederate, actualizaciones de seguridad importantes, muchas correcciones de errores y nuevas y magníficas herramientas para la solución de problemas.

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto**: Administración del ciclo de vida de la identidad

La versión de mayo de AADConnect contiene una versión preliminar pública de la integración con PingFederate, actualizaciones de seguridad importantes, muchas correcciones de errores y nuevas y magníficas herramientas para la solución de problemas. Puede encontrar las notas de la versión [aquí](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-version-history#118190).

---

### <a name="azure-ad-access-reviews-auto-apply"></a>Revisiones de acceso de Azure AD: aplicación automática

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Revisiones de acceso Gobernanza

Las revisiones de acceso de grupos y aplicaciones ahora están disponibles de forma generalizada como parte de Azure AD Premium P2. Un administrador puede realizar la configuración necesaria para aplicar automáticamente los cambios del revisor a un grupo o aplicación cuando se complete la revisión de acceso. El administrador también puede especificar lo que ocurre al acceso continuado del usuario si los revisores no responden: quitar el acceso, mantener el acceso o seguir las recomendaciones del sistema.

---

### <a name="id-tokens-can-no-longer-be-returned-using-the-query-response_mode-for-new-apps"></a>Los tokens de identificador no se pueden devolver mediante el valor de response_mode de una consulta en las nuevas aplicaciones.

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Las aplicaciones creadas a partir del 25 de abril de 2018 ya no podrán solicitar un valor de **id_token** mediante el argumento response_mode de **query**.  De esta forma, Azure AD se alinea con las especificaciones de OIDC y se reduce la superficie de ataque de sus aplicaciones.  No se impide que las aplicaciones creadas antes del 25 de abril de 2018 utilicen el argumento response_mode de **query** con un valor de response_type de **id_token**.  El error que se devuelve al solicitar un id_token de AAD es **AADSTS70007: "query" no es un valor de "response_mode" que se admita al solicitar un token**.

Los valores de response_mode **fragment** y **form_post** siguen funcionando (al crear nuevos objetos de aplicación [por ejemplo, para el uso de proxy de aplicación]). Asegúrese de usar uno de estos valores de response_mode antes de que creen una aplicación nueva.

---

## <a name="april-2018"></a>Abril de 2018

### <a name="azure-ad-b2c-access-token-are-ga"></a>Los tokens de acceso de Azure AD B2C son GA

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Ya puede acceder a las API web que protege Azure AD B2C gracias a los tokens de acceso. Esta característica está pasando de la versión preliminar pública a la versión de disponibilidad general. A parte de otras mejoras menores, también se ha mejorado la experiencia de interfaz de usuario para configurar las aplicaciones de Azure AD B2C y las API web.

Para más información, consulte [Azure AD B2C: Solicitud de tokens de acceso](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-access-tokens).

---

### <a name="test-single-sign-on-configuration-for-saml-based-applications"></a>Probar la configuración del inicio de sesión único para aplicaciones basadas en SAML

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales SSO

Al configurar las aplicaciones SSO basadas en SAML, puede probar la integración en la página de configuración. Si se produce un error durante el inicio de sesión, puede proporcionar dicho error en la experiencia de prueba y Azure AD le proporcionará los pasos para resolver ese problema.

Para más información, consulte:

- [Configuración del inicio de sesión único en aplicaciones que no están en la Galería de aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-custom-apps)
- [Cómo depurar el inicio de sesión único basado en SAML en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-saml-debugging)

---

### <a name="azure-ad-terms-of-use-now-has-per-user-reporting"></a>Los términos de uso de Azure AD ahora tienen informes por usuario

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Cumplimiento normativo

Los administradores pueden seleccionar una versión determinada de los Términos de uso y ver qué usuarios han dado su consentimiento y en qué fecha y hora.

Para obtener más información, consulte [Azure AD terms of use feature](https://docs.microsoft.com/azure/active-directory/conditional-access/terms-of-use) (Característica de términos de uso de Azure AD).

---

### <a name="azure-ad-connect-health-risky-ip-for-ad-fs-extranet-lockout-protection"></a>Azure AD Connect Health: Direcciones IP de riesgo para la protección de bloqueo de extranet de AD FS

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Otros Supervisión e informes

Connect Health ahora tiene la capacidad de detectar direcciones IP que superan un umbral de inicios de sesión de U/P erróneos por hora o por día. Estas son las funcionalidades que proporciona esta característica:

- Un informe completo que muestra la dirección IP y el número de inicios de sesión erróneos generados por hora o por día y con un umbral personalizable.
- Alertas basadas en el correo electrónico que muestran el momento en que una dirección IP específica excede el umbral de inicios de sesión de U/P erróneos por hora o por día.
- Una opción de descarga para realizar un análisis detallado de los datos.

Para obtener más información, consulte [Informe de direcciones IP de riesgo](https://aka.ms/aadchriskyip).

---

### <a name="easy-app-config-with-metadata-file-or-url"></a>Configuración sencilla de aplicaciones mediante una dirección URL o un archivo de metadatos

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales SSO

En la página de aplicaciones Enterprise, los administradores pueden cargar un archivo de metadatos SAML para configurar el inicio de sesión basado en SAML tanto de las aplicaciones de la galería de AAD, como de las que no pertenecen a la galería.

Asimismo, puede usar la dirección URL de metadatos de la federación de aplicaciones de Azure AD para configurar el SSO con la aplicación de destino.

Para obtener más información, consulte [Configuring single sign-on to applications that are not in the Azure Active Directory application gallery](https://docs.microsoft.com/azure/active-directory/active-directory-saas-custom-apps) (Configurar el inicio de sesión único de las aplicaciones que no forman parte de la galería de aplicaciones de Azure Active Directory).

---

### <a name="azure-ad-terms-of-use-now-generally-available"></a>Los Términos de uso de Azure AD están disponibles de forma general

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Cumplimiento normativo


Los términos de uso de Azure AD han dejado de estar en versión preliminar pública y ahora están disponibles de forma general.

Para obtener más información, consulte [Azure AD terms of use feature](https://docs.microsoft.com/azure/active-directory/conditional-access/terms-of-use) (Característica de términos de uso de Azure AD).

---

### <a name="allow-or-block-invitations-to-b2b-users-from-specific-organizations"></a>Permitir o bloquear invitaciones a usuarios B2B procedentes de determinadas organizaciones

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2B B2B/B2C


Ya puede especificar con qué organizaciones de socio quiere compartir contenido y colaborar en Azure AD B2B Collaboration. Para hacer esto, puede elegir crear una lista de dominios específicos para otorgar permiso o denegarlo. Cuando se bloquea un dominio mediante estas funcionalidades, los empleados ya no pueden enviar invitaciones a los usuarios de ese dominio.

Esto le ayudará a controlar el acceso a los recursos, a la vez que ofrece una experiencia fluida a los usuarios aprobados.

La característica Colaboración B2B está disponible para todos los clientes de Azure Active Directory y se puede usar con las características de Azure AD Premium, como el acceso condicional y la protección de identidad, para saber al detalle cómo y cuándo inician sesión y obtienen acceso los usuarios comerciales externos.

Para obtener más información, consulte [Allow or block invitations to B2B users from specific organizations](https://docs.microsoft.com/azure/active-directory/active-directory-b2b-allow-deny-list) (Permitir o bloquear invitaciones a usuarios de B2B procedentes de determinadas organizaciones).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En abril de 2018, hemos agregado estas 13 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones:

Criterion HCM, [FiscalNote](https://docs.microsoft.com/azure/active-directory/active-directory-saas-fiscalnote-tutorial), [Secret Server (local)](https://docs.microsoft.com/azure/active-directory/active-directory-saas-secretserver-on-premises-tutorial), [Dynamic Signal](https://docs.microsoft.com/azure/active-directory/active-directory-saas-dynamicsignal-tutorial), [mindWireless](https://docs.microsoft.com/azure/active-directory/active-directory-saas-mindwireless-tutorial), [OrgChart Now](https://docs.microsoft.com/azure/active-directory/active-directory-saas-orgchartnow-tutorial), [Ziflow](https://docs.microsoft.com/azure/active-directory/active-directory-saas-ziflow-tutorial), [AppNeta Performance Monitor](https://docs.microsoft.com/azure/active-directory/active-directory-saas-appneta-tutorial), [Elium](https://docs.microsoft.com/azure/active-directory/active-directory-saas-elium-tutorial) , [Fluxx Labs](https://docs.microsoft.com/azure/active-directory/active-directory-saas-fluxxlabs-tutorial), [Cisco Cloud](https://docs.microsoft.com/azure/active-directory/active-directory-saas-ciscocloud-tutorial), Shelf, [SafetyNet](https://docs.microsoft.com/azure/active-directory/active-directory-saas-safetynet-tutorial)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial).

Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-app-gallery-listing).

---

### <a name="grant-b2b-users-in-azure-ad-access-to-your-on-premises-applications-public-preview"></a>Conceder a los usuarios de B2B en Azure AD acceso a las aplicaciones locales (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2B B2B/B2C

Las organizaciones que usan las funcionalidades de colaboración B2B de Azure Active Directory (Azure AD) para invitar a los usuarios invitados de organizaciones asociadas a su instancia de Azure AD, ahora pueden proporcionar a estos usuarios B2B acceso a las aplicaciones locales. Estas aplicaciones locales pueden usar la autenticación basada en SAML o la autenticación de Windows integrada (IWA) con la delegación limitada de kerberos (KCD).

Para obtener más información, consulte [Conceder a los usuarios de B2B en Azure AD acceso a las aplicaciones locales](https://docs.microsoft.com/azure/active-directory/active-directory-b2b-hybrid-cloud-to-on-premises).

---

### <a name="get-sso-integration-tutorials-from-the-azure-marketplace"></a>Obtener tutoriales de integración de SSO en Azure Marketplace

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Otros Integración de terceros

Si una aplicación incluida en [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/azure-active-directory-apps?page=1) admite el proceso de inicio de sesión único basado en SAML, al hacer clic en **Obtener ahora**  se proporciona el tutorial de integración asociado a esa aplicación.

---

### <a name="faster-performance-of-azure-ad-automatic-user-provisioning-to-saas-applications"></a>Rendimiento más rápido del aprovisionamiento de usuarios automático de Azure AD para aplicaciones SaaS

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Aprovisionamiento de aplicaciones Integración de terceros

Anteriormente, los clientes que utilizaban los conectores de aprovisionamiento de usuarios de Azure Active Directory para aplicaciones de SaaS (por ejemplo, Salesforce, ServiceNow y Box) podían experimentar un rendimiento lento si sus inquilinos de Azure AD contenían más de 100 000 usuarios y grupos combinados, y si usaban asignaciones de usuarios y grupos para determinar qué usuarios debían aprovisionarse.

El 2 de abril de 2018 se implementaron mejoras de rendimiento significativas en el servicio de aprovisionamiento de Azure AD que reducen en gran medida la cantidad de tiempo necesario para realizar las sincronizaciones iniciales entre Azure Active Directory y las aplicaciones de SaaS de destino.

Como resultado, ahora se completan en cuestión de minutos u horas sincronizaciones iniciales que muchos clientes realizaban en sus aplicaciones y que antes tardaban días en completarse o que nunca llegaban a completarse.

Para obtener más información, consulte [¿Qué ocurre durante el aprovisionamiento?](/azure//active-directory/app-provisioning/how-provisioning-works)

---

### <a name="self-service-password-reset-from-windows-10-lock-screen-for-hybrid-azure-ad-joined-machines"></a>Restablecimiento de contraseña de autoservicio desde la pantalla de bloqueo de Windows 10 para máquinas híbridas unidas a Azure AD

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autoservicio de restablecimiento de contraseña Autenticación de usuarios

Hemos actualizado la característica SSPR de Windows 10 para incluir soporte técnico para máquinas híbridas que se hayan unido a Azure AD. Esta característica está disponible en Windows 10 RS4 y permite a los usuarios restablecer la contraseña en la pantalla de bloqueo de una máquina Windows 10. Los usuarios que están habilitados y registrados para poder realizar el restablecimiento de contraseña de autoservicio pueden utilizar esta característica.

Para obtener más información, consulte [Azure AD password reset from the login screen](https://docs.microsoft.com/azure/active-directory/authentication/tutorial-sspr-windows) (Restablecer la contraseña de Azure AD desde la pantalla de inicio de sesión).

---

## <a name="march-2018"></a>Marzo de 2018

### <a name="certificate-expire-notification"></a>Notificación de expiración de certificado

**Tipo:** **Categoría del servicio:** Corregida **Funcionalidad del producto:** Aplicaciones empresariales SSO

Azure AD envía una notificación cuando un certificado de una aplicación incluida o no en una galería está a punto de expirar.

Algunos usuarios no recibían notificaciones sobre aplicaciones empresariales configuradas con el inicio de sesión único basado en SAML. Este problema se ha resuelto. Azure AD envía una notificación para los certificados que expiran en 7, 30 y 60 días. Este evento puede verse en los registros de auditoría.

Para más información, consulte:

- [Administrar certificados para inicio de sesión único federado en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-sso-certs)
- [Informes de actividad de auditoría en el portal de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-activity-audit-logs)

---

### <a name="twitter-and-github-identity-providers-in-azure-ad-b2c"></a>Proveedores de identidades de Twitter y GitHub en Azure AD B2C

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Ahora, puede agregar Twitter o GitHub como proveedores de identidades en Azure AD B2C. Twitter está pasando de la versión preliminar pública a la versión de disponibilidad general. GitHub se va a publicar en la versión preliminar pública.

Para más información, consulte [¿Qué es la colaboración B2B de Azure AD?](https://docs.microsoft.com/azure/active-directory/active-directory-b2b-what-is-azure-ad-b2b)

---

### <a name="restrict-browser-access-using-intune-managed-browser-with-azure-ad-application-based-conditional-access-for-ios-and-android"></a>Restricción del acceso de explorador mediante Intune Managed Browser con acceso condicional basado en aplicaciones de Azure AD para iOS y Android

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Acceso condicional Seguridad y protección de la identidad

**Ahora en versión preliminar pública**

**SSO de Intune Managed Browser:** los empleados pueden utilizar el inicio de sesión único por los clientes nativos (por ejemplo, Microsoft Outlook) e Intune Managed Browser para todas las aplicaciones de Azure AD conectados.

**Compatibilidad con el acceso condicional de Intune Managed Browser:** ahora puede requerir que los empleados usen Intune Managed Browser mediante directivas de acceso condicional basado en aplicaciones.

Lea más sobre esto en nuestra [entrada de blog](https://cloudblogs.microsoft.com/enterprisemobility/2018/03/15/the-intune-managed-browser-now-supports-azure-ad-sso-and-conditional-access/).

Para más información, consulte:

- [Configuración del acceso condicional basado en aplicaciones](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access)

- [Configuración de directivas de Managed Browser](https://aka.ms/managedbrowser)

---

### <a name="app-proxy-cmdlets-in-powershell-ga-module"></a>Cmdlets del proxy de aplicación en el módulo de disponibilidad general de PowerShell

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Proxy de aplicaciones Control de acceso

Ahora, los cmdlets del proxy de aplicación pueden utilizarse en el módulo de disponibilidad general de PowerShell. Es necesario mantener actualizados los módulos de PowerShell; si lleva más de un año de retraso, es posible que algunos cmdlets dejen de funcionar.

Para más información, consulte [AzureAD](https://docs.microsoft.com/powershell/module/Azuread/?view=azureadps-2.0).

---

### <a name="office-365-native-clients-are-supported-by-seamless-sso-using-a-non-interactive-protocol"></a>Loa clientes nativos de Office 365 son compatibles con SSO de conexión directa utilizando un protocolo no interactivo

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Los usuarios que utilizan clientes nativos de Office 365 (versión 16.0.8730.xxxx y posteriores) pueden iniciar sesión sin que sea necesaria su intervención mediante SSO de conexión directa. Esta funcionalidad es posible gracias a la incorporación de un protocolo no interactivo (WS-Trust) en Azure AD.

Para más información, consulte [¿Cómo funciona el inicio de sesión en un cliente nativo con SSO de conexión directa?](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-sso-how-it-works#how-does-sign-in-on-a-native-client-with-seamless-sso-work)

---

### <a name="users-get-a-silent-sign-on-experience-with-seamless-sso-if-an-application-sends-sign-in-requests-to-azure-ads-tenant-endpoints"></a>Con SSO de conexión directa, no es necesaria la intervención del usuario para iniciar sesión cuando una aplicación envía solicitudes de inicio de sesión a puntos de conexión de inquilinos de Azure AD.

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Con el inicio de sesión único de conexión directa, los usuarios no necesitan intervenir para iniciar sesión si una aplicación (por ejemplo, `https://contoso.sharepoint.com`) envía solicitudes de inicio de sesión a puntos de conexión de inquilinos de Azure AD (por ejemplo, `https://login.microsoftonline.com/contoso.com/<..>` o `https://login.microsoftonline.com/<tenant_ID>/<..>`), en lugar de a puntos de conexión comunes de Azure AD (`https://login.microsoftonline.com/common/<...>`).

Para más información, consulte [Inicio de sesión único de conexión directa de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-sso).

---

### <a name="need-to-add-only-one-azure-ad-url-instead-of-two-urls-previously-to-users-intranet-zone-settings-to-roll-out-seamless-sso"></a>Solo es necesario agregar una dirección URL de Azure AD, en lugar de las dos que se requerían anteriormente, en la configuración de la zona de intranet de los usuarios para activar el SSO de conexión directa

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Si desea activar el SSO de conexión directa para los usuarios, solo debe agregar una dirección URL de Azure AD en la configuración de la zona de intranet de los usuarios utilizando una directiva de grupo de Active Directory: `https://autologon.microsoftazuread-sso.com`. Anteriormente, era necesario que los clientes agregaran dos direcciones URL.

Para más información, consulte [Inicio de sesión único de conexión directa de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-sso).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery"></a>Nuevas aplicaciones federadas disponibles en la Galería de aplicaciones de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En abril de 2018, hemos agregado estas 15 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones:

[Boxcryptor](https://docs.microsoft.com/azure/active-directory/active-directory-saas-boxcryptor-tutorial), [CylancePROTECT](https://docs.microsoft.com/azure/active-directory/active-directory-saas-cylanceprotect-tutorial), Wrike, [SignalFx](https://docs.microsoft.com/azure/active-directory/active-directory-saas-signalfx-tutorial), Assistant by FirstAgenda, [YardiOne](https://docs.microsoft.com/azure/active-directory/active-directory-saas-yardione-tutorial), Vtiger CRM, inwink, [Amplitude](https://docs.microsoft.com/azure/active-directory/active-directory-saas-amplitude-tutorial), [Spacio](https://docs.microsoft.com/azure/active-directory/active-directory-saas-spacio-tutorial), [ContractWorks](https://docs.microsoft.com/azure/active-directory/active-directory-saas-contractworks-tutorial), [Bersin](https://docs.microsoft.com/azure/active-directory/active-directory-saas-bersin-tutorial), [Mercell](https://docs.microsoft.com/azure/active-directory/active-directory-saas-mercell-tutorial), [Trisotech Digital Enterprise Server](https://docs.microsoft.com/azure/active-directory/active-directory-saas-trisotechdigitalenterpriseserver-tutorial) y [Qumu Cloud](https://docs.microsoft.com/azure/active-directory/active-directory-saas-qumucloud-tutorial).

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial).

Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-app-gallery-listing).

---

### <a name="pim-for-azure-resources-is-generally-available"></a>PIM de Azure Resources está disponible con carácter general

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Privileged Identity Management Privileged Identity Management

Si utiliza Azure AD Privileged Identity Management para los roles de directorio, ahora puede utilizar las funcionalidades de asignación y acceso con límite de tiempo de PIM con los roles de Recursos de Azure, como Suscripciones, Grupos de recursos, Máquinas virtuales y cualquier otro recurso compatible con Azure Resource Manager. Puede forzar la aplicación de la autenticación multifactor al activar roles Just-In-Time y programar activaciones de forma coordinada con los periodos de cambio aprobados. Además, esta versión incorpora mejoras que no está disponibles en la versión preliminar pública, como una interfaz de usuario actualizada, flujos de trabajo de aprobación y la capacidad de ampliar roles que van a expirar en breve y de renovar roles que han expirado.

Para obtener más información, consulte [PIM para recursos de Azure (versión preliminar)](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/azure-pim-resource-rbac).

---

### <a name="adding-optional-claims-to-your-apps-tokens-public-preview"></a>Incorporación de notificaciones opcionales en los tokens de aplicaciones (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Ahora, las aplicaciones de Azure AD pueden solicitar notificaciones personalizadas u opcionales en los tokens SAML o JWT.  Estas notificaciones sobre el usuario o el inquilino no se incluyen de forma predeterminada en el token por restricciones de tamaño o aplicabilidad.  Actualmente, esta funcionalidad está en versión preliminar pública para las aplicaciones de Azure AD de los puntos de conexión 1.0 y 2.0.  Consulte la documentación para obtener información acerca de qué notificaciones pueden agregarse y cómo editar el manifiesto de aplicación para solicitarlas.

Para más información, consulte [Optional claims in Azure AD](https://docs.microsoft.com/azure/active-directory/develop/active-directory-optional-claims) (Notificaciones opcionales de Azure AD)///.

---

### <a name="azure-ad-supports-pkce-for-more-secure-oauth-flows"></a>Azure AD permite usar PKCE para disponer de flujos de OAuth más seguros

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Los documentos de Azure AD se han actualizado para que tengan en cuenta la compatibilidad con PKCE, lo que permite una comunicación más segura durante el flujo de concesión del código de autorización de OAuth 2.0.  Los puntos de conexión de la versión 1.0 y 2.0 admiten S256 y code_challenges de texto no cifrado.

Para más información, consulte [Solicitud de un código de autorización](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-protocols-oauth-code#request-an-authorization-code).

---

### <a name="support-for-provisioning-all-user-attribute-values-available-in-the-workday-get_workers-api"></a>Compatibilidad con el aprovisionamiento de todos los valores de atributo de usuario disponibles en Workday Get_Workers API

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aprovisionamiento de aplicaciones Integración de terceros

Ahora, la versión preliminar pública de aprovisionamiento de entrada de Workday a Active Directory y Azure AD permite extraer y aprovisionar todos los valores de atributo disponibles en Workday Get_Workers API. De este modo, se amplía la compatibilidad con cientos de atributos estándar y personalizados, que se suman a los que ya venían con la versión inicial del conector de aprovisionamiento de entrada de Workday.

Para más información, consulte: [Personalización de la lista de atributos de usuario de Workday](https://docs.microsoft.com/azure/active-directory/active-directory-saas-workday-inbound-tutorial#customizing-the-list-of-workday-user-attributes)

---

### <a name="changing-group-membership-from-dynamic-to-static-and-vice-versa"></a>Cambio de la pertenencia a grupos de dinámica a estática, y viceversa

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración de grupos Colaboración

Es posible cambiar cómo se administra la pertenencia a un grupo. Esto es útil cuando desea mantener el mismo nombre de grupo y el identificador en el sistema, por lo que cualquier referencia existente al grupo sigue siendo válida; crear un nuevo grupo requeriría actualizar esas referencias.
Hemos actualizado el centro de administración de Azure AD para incorporar la compatibilidad con esta funcionalidad. Ahora, los clientes pueden cambiar los grupos existentes para que tengan una pertenencia dinámica en lugar de una pertenencia asignada, y viceversa. Los cmdlets de PowerShell existentes seguirán estando disponibles.

Para obtener más información, consulte [Reglas de pertenencia dinámica a grupos de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-dynamic-membership).

---

### <a name="improved-sign-out-behavior-with-seamless-sso"></a>Comportamiento de cierre de sesión mejorado con SSO de conexión directa

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Anteriormente, aunque los usuarios cerraran sesión explícitamente en una aplicación protegida con Azure AD, la sesión se iniciaba de nuevo automáticamente mediante SSO de conexión directa si intentaban volver a acceder a una aplicación de Azure AD en la red corporativa desde los dispositivos unidos al dominio. Con este cambio, se puede cerrar sesión.  De este modo, los usuarios pueden elegir otra cuenta de Azure AD al iniciar sesión de nuevo, en lugar de la que la sesión se inicie automáticamente con el SSO de conexión directa.

Para más información, consulte [Inicio de sesión único de conexión directa de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-sso).

---

### <a name="application-proxy-connector-version-154020-released"></a>Se ha publicado la versión 1.5.402.0 del conector del proxy de aplicación

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Proxy de aplicaciones Seguridad y protección de la identidad

Esta versión del conector se irá implementando gradualmente hasta noviembre. Esta nueva versión del conector incluye los siguientes cambios:

- Ahora, el conector establece cookies de nivel de dominio en lugar de cookies de nivel de subdominio. De este modo, la experiencia de SSO resulta más fluida y se evita el envío de peticiones de autenticación redundantes.
- Se admiten solicitudes de codificación fragmentada.
- Se ha mejorado la supervisión del mantenimiento del conector.
- Se han solucionado algunos errores y se han hecho mejoras en la estabilidad.

Para más información, consulte [Descripción de los conectores de Azure Active Directory Application Proxy](https://docs.microsoft.com/azure/active-directory/application-proxy-understand-connectors).

---

## <a name="february-2018"></a>Febrero de 2018

### <a name="improved-navigation-for-managing-users-and-groups"></a>Navegación mejorada para administrar usuarios y grupos

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Administración de directorios Directorio

Se ha simplificado la experiencia de navegación para administrar usuarios y grupos. Ahora puede ir directamente desde la información general del directorio a la lista de todos los usuarios, con un acceso más fácil a la lista de usuarios eliminados. Ahora puede ir directamente desde la información general del directorio a la lista de todos los grupos, con un acceso más fácil a la configuración de administración de grupos. Y también en la página de información general del directorio puede buscar un usuario, grupo, aplicación empresarial o registro de la aplicación.

---

### <a name="availability-of-sign-ins-and-audit-reports-in-microsoft-azure-operated-by-21vianet-azure-china-21vianet"></a>Disponibilidad de los informes de inicios de sesión y auditoría en Microsoft Azure controlado por 21Vianet (Azure China 21Vianet)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure Stack Supervisión e informes

Los informes de registro de actividad de Azure AD ya están disponibles en Microsoft Azure controlado por instancias de 21Vianet (Azure China 21Vianet). Se incluyen los siguientes registros:

- **Registros de actividad de inicios de sesión**: incluye todos los registros de inicios de sesión asociados con el inquilino.

- **Registros de auditoría de contraseñas de autoservicio**: incluye todos los registros de auditoría de SSPR.

- **Registros de auditoría de administración de directorios**: incluye todos los registros de auditoría relacionados con la administración de directorios como, por ejemplo, administración de usuarios, administración de aplicaciones y otros.

Con estos registros, puede obtener información detallada sobre el funcionamiento de su entorno. Los datos proporcionados le permiten:

- Determinar cómo utilizan los usuarios las aplicaciones y servicios.

- Solucionar problemas que impiden a los usuarios finalizar su trabajo.

Para más información sobre cómo usar estos informes, consulte [Informes de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-azure-portal).

---

### <a name="use-report-reader-role-non-admin-role-to-view-azure-ad-activity-reports"></a>Use el rol "Lector de informes" (rol que no es de administrador) para ver los informes de actividad de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Informes Supervisión e informes

Como parte de los comentarios de los clientes para habilitar los roles que no son de administrador para acceder a los registros de actividades de Azure AD, hemos habilitado la posibilidad de que los usuarios con el rol "Lector de informes" puedan acceder a las actividades de inicios de sesión y de auditoría en Azure Portal, así como mediante Microsoft Graph API.

Para más información sobre cómo usar estos informes, consulte [Informes de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-azure-portal).

---

### <a name="employeeid-claim-available-as-user-attribute-and-user-identifier"></a>Notificación de EmployeeID disponible como atributo de usuario e identificador de usuario

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales SSO

Puede configurar **EmployeeID** como el identificador de usuario y el atributo de usuario para los usuarios miembros y para los invitados B2B en aplicaciones de inicio de sesión basadas en SAML desde la interfaz de usuario de la aplicación empresarial.

Para más información, consulte [Personalización de las notificaciones emitidas en el token SAML para aplicaciones empresariales en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-saml-claims-customization).

---

### <a name="simplified-application-management-using-wildcards-in-azure-ad-application-proxy"></a>Administración simplificada de aplicaciones mediante caracteres comodín en Azure AD Application Proxy

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Proxy de aplicaciones Autenticación de usuarios

Para facilitar la implementación de aplicaciones y reducir la carga administrativa, ahora se admite la posibilidad de publicar aplicaciones mediante caracteres comodín. Para publicar una aplicación con comodín, puede seguir el flujo de publicación de aplicaciones estándar, pero usando un carácter comodín en las direcciones URL internas y externas.

Para más información, consulte [Aplicaciones con comodín en Azure Active Directory Application Proxy](https://docs.microsoft.com/azure/active-directory/active-directory-application-proxy-wildcard).

---

### <a name="new-cmdlets-to-support-configuration-of-application-proxy"></a>Nuevos cmdlets para admitir la configuración de Application Proxy

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Proxy de aplicaciones Plataforma

La versión preliminar más reciente del módulo Azure AD PowerShell contiene nuevos cmdlets que permiten a los clientes configurar aplicaciones de Application Proxy mediante PowerShell.

Los nuevos cmdlets son:

- Get-AzureADApplicationProxyApplication
- Get-AzureADApplicationProxyApplicationConnectorGroup
- Get-AzureADApplicationProxyConnector
- Get-AzureADApplicationProxyConnectorGroup
- Get-AzureADApplicationProxyConnectorGroupMembers
- Get-AzureADApplicationProxyConnectorMemberOf
- New-AzureADApplicationProxyApplication
- New-AzureADApplicationProxyConnectorGroup
- Remove-AzureADApplicationProxyApplication
- Remove-AzureADApplicationProxyApplicationConnectorGroup
- Remove-AzureADApplicationProxyConnectorGroup
- Set-AzureADApplicationProxyApplication
- Set-AzureADApplicationProxyApplicationConnectorGroup
- Set-AzureADApplicationProxyApplicationCustomDomainCertificate
- Set-AzureADApplicationProxyApplicationSingleSignOn
- Set-AzureADApplicationProxyConnector
- Set-AzureADApplicationProxyConnectorGroup

---

### <a name="new-cmdlets-to-support-configuration-of-groups"></a>Nuevos cmdlets para admitir la configuración de grupos

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Proxy de aplicaciones Plataforma

La versión más reciente del módulo de Azure AD PowerShell contiene cmdlets para administrar grupos en Azure AD. Estos cmdlets no estaban disponibles anteriormente en el módulo AzureADPreview y ahora se han agregado al módulo AzureAD.

Los cmdlets de grupo que ya se han publicado con disponibilidad general son:

- Get-AzureADMSGroup
- New-AzureADMSGroup
- Remove-AzureADMSGroup
- Set-AzureADMSGroup
- Get-AzureADMSGroupLifecyclePolicy
- New-AzureADMSGroupLifecyclePolicy
- Remove-AzureADMSGroupLifecyclePolicy
- Add-AzureADMSLifecyclePolicyGroup
- Remove-AzureADMSLifecyclePolicyGroup
- Reset-AzureADMSLifeCycleGroup
- Get-AzureADMSLifecyclePolicyGroup

---

### <a name="a-new-release-of-azure-ad-connect-is-available"></a>Está disponible una nueva versión de Azure AD Connect

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Sincronización de AD Plataforma

Azure AD Connect es la herramienta preferida para sincronizar datos entre Azure AD y los orígenes de datos locales, incluidos Windows Server Active Directory y LDAP.

>[!Important]
>Esta compilación introduce cambios en las reglas de sincronización y en el esquema. El servicio de sincronización de Azure AD Connect desencadenará pasos de importación completa y sincronización completa después de una actualización. Para obtener información sobre cómo cambiar este comportamiento, consulte [Aplazamiento de la sincronización completa después de la actualización](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-upgrade-previous-version#how-to-defer-full-synchronization-after-upgrade).

Esta versión incluye las siguientes actualizaciones y cambios:

**Problemas corregidos:**

- Corrección de la ventana de sincronización en las tareas en segundo plano para la página de filtrado de particiones al cambiar a la página siguiente.

- Se ha corregido un error que provocó una infracción de acceso durante la acción personalizada ConfigDB.

- Se ha corregido un error para recuperarse del tiempo de espera de la conexión de SQL.

- Se ha corregido un error que provocaba que los certificados con caracteres comodín de SAN generaran un error al realizar una comprobación de requisitos previos.

- Se ha corregido un error que provocaba que miiserver.exe se bloqueara durante una exportación del conector de AAD.

- Se ha corregido un error en el que un intento de contraseña incorrecta registrado en el controlador de dominio al ejecutarse provocaba que el asistente de AAD Connect cambiara la configuración.

**Nuevas características y mejoras**

- Telemetría de aplicaciones: los administradores pueden activar o desactivar este tipo de datos.

- Datos de mantenimiento de Azure AD: los administradores deben visitar el portal de mantenimiento para controlar la configuración de mantenimiento. Una vez se haya cambiado la directiva del servicio, los agentes la leerán y la aplicarán.

- Se han agregado acciones de configuración de escritura diferida de dispositivo y una barra de progreso para la inicialización de la página.

- Se han mejorado los diagnósticos generales con un informe HTML y una recopilación completa de datos en un informe HTML o de texto ZIP.

- Se ha mejorado la confiabilidad de la actualización automática y se ha agregado telemetría adicional para asegurarse de que se puede determinar el mantenimiento del servidor.

- Se han restringido los permisos disponibles para las cuentas con privilegios de la cuenta del conector AD. Para las nuevas instalaciones, el asistente restringe los permisos que las cuentas con privilegios tienen en la cuenta de MSOL tras la creación de esa cuenta. Los cambios afectan a las instalaciones rápidas y a las instalaciones personalizadas con la opción de creación automática de cuenta.

- Se ha cambiado el instalador, por lo que no se requiere el privilegio de asociación de seguridad en una instalación limpia de AADConnect.

- Se ha agregado una utilidad nueva para solucionar problemas de sincronización de un objeto específico. Actualmente, la utilidad comprueba los siguientes aspectos:

    - Error de coincidencia de UserPrincipalName entre el objeto de usuario sincronizado y la cuenta de usuario del inquilino de Azure AD.

    - Si se filtra el objeto de sincronización debido al filtrado de dominio

    - Si se filtra el objeto de sincronización debido al filtrado de unidad organizativa (UO)

- Se ha agregado una utilidad nueva para sincronizar el hash de contraseña actual almacenado en el Active Directory local para una cuenta de usuario específica. La utilidad no requiere un cambio de contraseña.

---

### <a name="applications-supporting-intune-app-protection-policies-added-for-use-with-azure-ad-application-based-conditional-access"></a>Se han agregado aplicaciones compatibles con directivas de Intune App Protection para usarlas con el acceso condicional basado en aplicaciones de Azure AD

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Acceso condicional Seguridad y protección de la identidad

Se han agregado más aplicaciones que admiten el acceso condicional basado en aplicaciones. Ahora, ya puede acceder a Office 365 y otras aplicaciones en la nube conectadas a Azure AD mediante estas aplicaciones cliente aprobadas.

Las siguientes aplicaciones se agregarán a finales del mes de febrero:

- Microsoft Power BI

- Microsoft Launcher

- Microsoft Invoicing

Para más información, consulte:

- [Requisito de aplicación cliente aprobada](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-conditional-access-conditions#client-apps-preview)
- [Acceso condicional basado en aplicaciones de Azure AD](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access)

---

### <a name="terms-of-use-update-to-mobile-experience"></a>Actualización de los términos de uso relacionados con la experiencia móvil

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Términos de uso Cumplimiento normativo

Cuando aparecen los términos de uso, puede hacer clic en **¿Tiene problemas con la visualización? Haga clic aquí.** Si hace clic en este vínculo se abren los términos de uso de forma nativa en el dispositivo. Independientemente del tamaño de fuente en el documento o el tamaño de pantalla del dispositivo, puede acercar o alejar y leer el documento según sea necesario.

---

## <a name="january-2018"></a>Enero de 2018

### <a name="new-federated-apps-available-in-azure-ad-app-gallery"></a>Nuevas aplicaciones federadas disponibles en la Galería de aplicaciones de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En enero de 2018, se agregaron a la galería de aplicaciones las siguientes aplicaciones nuevas compatibles con la federación:

[IBM OpenPages](https://go.microsoft.com/fwlink/?linkid=864698), [OneTrust Privacy Management Software](https://go.microsoft.com/fwlink/?linkid=861660), [Dealpath](https://go.microsoft.com/fwlink/?linkid=863526), [IriusRisk Federated Directory y [Fidelity NetBenefits](https://go.microsoft.com/fwlink/?linkid=864701).

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial).

Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-app-gallery-listing).

---

### <a name="sign-in-with-additional-risk-detected"></a>Inicio de sesión con riesgo adicional detectado

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Identity Protection Seguridad y protección de la identidad

La perspectiva que se obtiene de una detección de riesgos identificada está asociada a su suscripción de Azure AD. Con la edición de Azure AD Premium P2, obtiene la información más detallada acerca de todas las detecciones subyacentes.

Con la edición de Azure AD Premium P1, las detecciones que no están cubiertas por su licencia aparecen como la detección de riesgos Inicio de sesión con riesgo adicional detectado.

Para más información, consulte [Detecciones de riesgos de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-risk-events).

---

### <a name="hide-office-365-applications-from-end-users-access-panels"></a>Ocultación de aplicaciones de Office 365 de los paneles de acceso del usuario final

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Mis aplicaciones SSO

Ahora puede administrar mejor el modo en que se muestran las aplicaciones de Office 365 en los paneles de acceso de sus usuarios mediante una nueva configuración de usuario. Esta opción le resultará útil para reducir el número de aplicaciones de los paneles de acceso de los usuarios si prefiere mostrar solo las aplicaciones de Office en el Portal de Office. La configuración se encuentra en **Configuración de usuario** y tiene la etiqueta **Los usuarios solo pueden ver las aplicaciones de Office 365 en el Portal de Office 365**.

Para obtener más información, consulte [Ocultación de una aplicación de la experiencia del usuario en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-hide-third-party-app).

---

### <a name="seamless-sign-into-apps-enabled-for-password-sso-directly-from-apps-url"></a>Inicio de sesión sin problemas en aplicaciones habilitadas para SSO de contraseña directamente desde la dirección URL de la aplicación

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Mis aplicaciones SSO

La extensión del explorador Mis aplicaciones ahora está disponible en una cómoda herramienta que ofrece la funcionalidad de inicio de sesión único Mis aplicaciones como acceso directo del explorador. Tras la instalación, los usuarios verán un icono de gofre en el explorador que proporciona acceso rápido a las aplicaciones. Ahora, los usuarios podrán disfrutar de las ventajas siguientes:

- Capacidad para iniciar sesión directamente en aplicaciones de SSO de contraseña desde la página de inicio de sesión de la aplicación.
- Inicio de cualquier aplicación mediante la característica de búsqueda rápida.
- Accesos directos a aplicaciones usadas recientemente desde la extensión.
- La extensión está disponible en Microsoft Edge, Chrome y Firefox.

Para obtener más información, consulte [Extensión de inicio de sesión seguro de Mis aplicaciones](../user-help/my-apps-portal-end-user-access.md#download-and-install-the-my-apps-secure-sign-in-extension).

---

### <a name="azure-ad-administration-experience-in-azure-classic-portal-has-been-retired"></a>Retirada de la experiencia de administración de Azure AD del Portal de Azure clásico

**Tipo:** **Categoría del servicio:** En desuso **Funcionalidad del producto:** Azure AD Directorio

El 8 de enero de 2018, la experiencia de administración de Azure AD se retiró del Portal de Azure clásico. Esto tuvo lugar junto con la retirada del Portal de Azure clásico. En el futuro, deberá usar el [Centro de administración de Azure AD](https://aad.portal.azure.com) para todas las tareas de administración de Azure AD basadas en el portal.

---

### <a name="the-phonefactor-web-portal-has-been-retired"></a>El portal web PhoneFactor se ha retirado.

**Tipo:** **Categoría del servicio:** En desuso **Funcionalidad del producto:** Azure AD Directorio

El 8 de enero de 2018 se retiró el portal web PhoneFactor. Este portal se usaba para la administración del servidor MFA; sin embargo, sus funciones se han movido a Azure Portal en portal.azure.com.

La configuración de MFA se encuentra en: **Azure Active Directory \> Servidor MFA**

---

### <a name="deprecate-azure-ad-reports"></a>Informes de Azure AD obsoletos

**Tipo:** **Categoría del servicio:** En desuso **Funcionalidad del producto**: Informes Administración del ciclo de vida de la identidad


Con la disponibilidad general de la nueva consola de administración de Azure Active Directory y las nuevas API disponibles para los informes de actividad y seguridad, las API de informes que se encontraban en el punto de conexión "/reports" se retiraron el 31 de diciembre de 2017.

**¿Qué está disponible?**

Como parte de la transición a la nueva consola de administración, existen dos nuevas API disponibles para recuperar los registros de actividad de Azure AD. El nuevo conjunto de API proporciona más funciones de filtro y ordenación, así como actividades de auditoría e inicio de sesión. Los datos que estaban disponibles anteriormente en los informes de seguridad ahora son accesibles a través de la API de detecciones de riesgos de Identity Protection en Microsoft Graph.

Para más información, consulte:

- [Introducción a la API de informes de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-api-getting-started-azure-portal)

- [Introducción a Azure Active Directory Identity Protection y Microsoft Graph](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection-graph-getting-started)

---

## <a name="december-2017"></a>Diciembre de 2017

### <a name="terms-of-use-in-the-access-panel"></a>Términos de uso del panel de acceso

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Cumplimiento normativo

Ahora puede ir al panel de acceso y ver los términos de uso previamente aceptados.

Siga estos pasos:

1. Abra el [portal de Mis aplicaciones](https://myapps.microsoft.com) e inicie sesión.

2. En la esquina superior derecha, seleccione su nombre y seleccione **Perfil** en la lista.

3. En su **Perfil**, seleccione **Revisar los términos de uso**.

4. Ahora podrá revisar los términos de uso que aceptó.

Para obtener más información, consulte [Característica Términos de uso de Azure Active Directory (versión preliminar)](https://docs.microsoft.com/azure/active-directory/conditional-access/terms-of-use).

---

### <a name="new-azure-ad-sign-in-experience"></a>Nueva experiencia de inicio de sesión de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure AD Autenticación de usuarios

Azure AD y las interfaces de usuario del sistema de identidades de cuentas de Microsoft se han rediseñado para ofrecer un aspecto más homogéneo. Además, la página de inicio de sesión de Azure AD primero recopila el nombre de usuario, seguido de la credencial en una segunda pantalla.

Para obtener más información, consulte [The new Azure AD Signin Experience is now in Public Preview](https://cloudblogs.microsoft.com/enterprisemobility/2017/08/02/the-new-azure-ad-signin-experience-is-now-in-public-preview/) (Nueva experiencia de inicio de sesión de Azure AD disponible en versión preliminar pública).

---

### <a name="fewer-sign-in-prompts-a-new-keep-me-signed-in-experience-for-azure-ad-sign-in"></a>Menos solicitudes de inicio de sesión: nueva experiencia "Mantener la sesión iniciada" para el inicio de sesión de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure AD Autenticación de usuarios

Hemos sustituido la casilla **Mantener la sesión iniciada** de la página de inicio de sesión de Azure AD por un nuevo mensaje que se muestra al realizar la autenticación correctamente.

Si se responde **Sí** a este mensaje, el servicio proporciona un token de actualización persistente. Este es el mismo comportamiento que se produce cuando se activa la casilla **Mantener la sesión iniciada** en la experiencia antigua. Para los inquilinos federados, este mensaje se muestra al autenticarse correctamente en el servicio federado.

Para más información, consulte [Fewer sign-in prompts: The new "keep me signed in" experience for Azure AD is in preview](https://cloudblogs.microsoft.com/enterprisemobility/2017/09/19/fewer-login-prompts-the-new-keep-me-signed-in-experience-for-azure-ad-is-in-preview/) (Menos avisos de inicio de sesión: la nueva experiencia "Mantener la sesión iniciada" para Azure AD está en versión preliminar).

---

### <a name="add-configuration-to-require-the-terms-of-use-to-be-expanded-prior-to-accepting"></a>Agregar configuración para exigir expandir los términos de uso antes de su aceptación

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Cumplimiento normativo

Se ha agregado una opción para los administradores con el fin de exigir a usuarios finales que expandan los términos de uso antes de aceptarlos.

Seleccione **Activado** o **Desactivado** para exigir que los usuarios expandan los términos de uso. El valor **Activado** exige a usuarios que lean los términos de uso antes de aceptarlos.

Para obtener más información, consulte [Característica Términos de uso de Azure Active Directory (versión preliminar)](https://docs.microsoft.com/azure/active-directory/conditional-access/terms-of-use).

---

### <a name="scoped-activation-for-eligible-role-assignments"></a>Activación de ámbito para las asignaciones de roles válidos

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Privileged Identity Management Privileged Identity Management

Puede usar la activación con ámbito para activar las asignaciones de roles de recursos de Azure válidas con menos autonomía que los valores predeterminados de las asignaciones originales. Un ejemplo de esto es si se le asigna como propietario de una suscripción en su inquilino. Con la activación con ámbito, podrá activar el rol de propietario para hasta cinco de los recursos incluidos en la suscripción (por ejemplo, los grupos de recursos y las máquinas virtuales). La definición del ámbito de la activación puede reducir la posibilidad de ejecutar cambios no deseados en recursos importantes de Azure.

Para más información, vea [¿Qué es Azure AD Privileged Identity Management?](https://docs.microsoft.com/azure/active-directory/active-directory-privileged-identity-management-configure).

---

### <a name="new-federated-apps-in-the-azure-ad-app-gallery"></a>Nuevas aplicaciones federadas en la galería de aplicaciones de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En diciembre de 2017, hemos agregado estas 15 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones:

[Accredible](https://go.microsoft.com/fwlink/?linkid=863523), Adobe Experience Manager, [EFI Digital StoreFront](https://go.microsoft.com/fwlink/?linkid=861685), [Communifire](https://go.microsoft.com/fwlink/?linkid=861676) CybSafe, [FactSet](https://go.microsoft.com/fwlink/?linkid=863525), [IMAGE WORKS](https://go.microsoft.com/fwlink/?linkid=863517), [MOBI](https://go.microsoft.com/fwlink/?linkid=863521), [Integración de Azure AD con MobileIron](https://go.microsoft.com/fwlink/?linkid=858027), [Reflektive](https://go.microsoft.com/fwlink/?linkid=863518), [SAML SSO for Bamboo by resolution GmbH](https://go.microsoft.com/fwlink/?linkid=863520), [SAML SSO for Bitbucket by resolution GmbH](https://go.microsoft.com/fwlink/?linkid=863519), [Vodeclic](https://go.microsoft.com/fwlink/?linkid=863522), WebHR, Zenegy Azure AD Integration.

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial).

Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-app-gallery-listing).

---

### <a name="approval-workflows-for-azure-ad-directory-roles"></a>Flujos de trabajo de aprobación de roles de directorio de Azure AD

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Privileged Identity Management Privileged Identity Management

El flujo de trabajo de aprobación de roles de directorio de Azure AD está disponible de manera general.

Con el flujo de trabajo de aprobación, los administradores de roles con privilegios pueden exigir que los miembros de roles aptos soliciten la activación de roles para poder usar el rol con privilegios. Ahora es posible delegar responsabilidades de aprobación en múltiples usuarios y grupos. Los miembros del rol aptos recibirán notificaciones cuando termine la aprobación y sus roles estén activos.

---

### <a name="pass-through-authentication-skype-for-business-support"></a>Autenticación de paso a través: Compatibilidad con Skype Empresarial

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Ahora, la autenticación de paso a través admite inicios de sesión de usuarios en aplicaciones cliente de Skype Empresarial compatibles con la autenticación moderna, incluidas las topologías híbridas y en línea.

Para obtener más información, consulte [Topologías de Skype Empresarial compatibles con la autenticación moderna](https://technet.microsoft.com/library/mt803262.aspx).

---

### <a name="updates-to-azure-ad-privileged-identity-management-for-azure-rbac-preview"></a>Actualizaciones de Azure AD Privileged Identity Management para RBAC de Azure (versión preliminar)

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Privileged Identity Management Privileged Identity Management

Con la actualización de versión preliminar pública de Azure AD Privileged Identity Management (PIM) para el control de acceso basado en rol (RBAC) de Azure, ahora podrá realizar lo siguiente:

* Usar Just Enough Administration.
* Solicitar aprobación para activar roles de recursos.
* Programar la activación futura de un rol que requiere la aprobación de roles de Azure AD y de Azure.

Para obtener más información, consulte [PIM para recursos de Azure (versión preliminar)](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/azure-pim-resource-rbac).

---

## <a name="november-2017"></a>Noviembre de 2017

### <a name="access-control-service-retirement"></a>Retirada del servicio de control de acceso

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto**: Access Control Service Servicio Access Control

Azure Active Directory Access Control (también conocido como Access Control Service) se retirará a finales de 2018. En las próximas semanas se ofrecerá más información, como, por ejemplo, una programación detallada y una guía de migración de alto nivel. Puede dejar comentarios en esta página con preguntas acerca de Access Control Service y un miembro del equipo le responderá.

---

### <a name="restrict-browser-access-to-the-intune-managed-browser"></a>Restringir el acceso desde el explorador a Intune Managed Browser

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Acceso condicional Protección y seguridad de la identidad

El uso de Intune Managed Browser como aplicación aprobada le permite restringir el acceso desde el explorador a Office 365 y otras aplicaciones en la nube conectadas a Azure AD.

Ahora podrá configurar la siguiente condición para el acceso condicional basado en aplicaciones:

**Aplicaciones cliente:** Browser

**¿Cuál es el efecto del cambio?**

En la actualidad, el acceso está bloqueado cuando se usa esta condición. Cuando la vista previa está disponible, todos los accesos requerirán el uso de la aplicación Manager Browser.

Busque esta funcionalidad y más información en las próximas entradas de blogs y notas de versión.

Para obtener más información, consulte [Acceso condicional en Azure AD](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-azure-portal).

---

### <a name="new-approved-client-apps-for-azure-ad-app-based-conditional-access"></a>Nuevas aplicaciones cliente aprobadas para el acceso condicional basado en aplicaciones de Azure AD

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Acceso condicional Protección y seguridad de la identidad

Las siguientes aplicaciones se incluyen en la lista de [aplicaciones cliente aprobadas](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-conditional-access-conditions#client-apps-preview):

- [Microsoft Kaizala](https://www.microsoft.com/garage/profiles/kaizala/)
- Microsoft StaffHub

Para más información, consulte:

- [Requisito de aplicación cliente aprobada](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-conditional-access-conditions#client-apps-preview)
- [Acceso condicional basado en aplicaciones de Azure AD](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access)

---

### <a name="terms-of-use-support-for-multiple-languages"></a>Disponibilidad de los términos de uso en varios idiomas

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Cumplimiento normativo

Ahora, los administradores pueden crear nuevos términos de uso que contengan varios documentos PDF. Puede etiquetar estos documentos PDF con el idioma correspondiente. De este modo, se mostrará a los usuarios el documento PDF en el idioma correspondiente a sus preferencias. Si no hay ninguna coincidencia, se muestra el idioma predeterminado.

---

### <a name="real-time-password-writeback-client-status"></a>Estado del cliente de escritura diferida de contraseñas en tiempo real

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autoservicio de restablecimiento de contraseña Autenticación de usuarios

Ahora podrá revisar el estado de su cliente de escritura diferida de contraseñas local. Esta opción está disponible en la sección **Integración local** de la página [Restablecimiento de contraseña](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/PasswordReset).

Si hay problemas con la conexión al cliente de escritura diferida local, se mostrará un mensaje de error con los elementos siguientes:

- Información del motivo por el que no puede conectarse a su cliente de escritura diferida local.
- Un vínculo a documentación que le ayuda a resolver el problema.

Para obtener más información, consulte [Integración local](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-how-it-works#on-premises-integration).

---

### <a name="azure-ad-app-based-conditional-access"></a>Acceso condicional basado en aplicaciones de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure AD Protección y seguridad de la identidad

Ahora puede restringir el acceso a Office 365 y a otras aplicaciones en la nube conectadas a Azure AD para [aplicaciones cliente aprobadas](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-conditional-access-conditions#client-apps-preview) que admiten directivas de Intune App Protection mediante el [acceso condicional basado en aplicaciones de Azure AD](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access). Las directivas de Intune App Protection se utilizan para configurar y proteger los datos de empresa en estas aplicaciones cliente.

Al combinar directivas de acceso condicional [basado en aplicaciones](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access) con las directivas de acceso condicional [basado en dispositivos](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-policy-connected-applications), tiene la flexibilidad necesaria para proteger los datos de dispositivos personales y de la empresa.

Ahora puede usar los siguientes controles y condiciones con el acceso condicional basado en aplicaciones:

**Condición de plataforma admitida**

- iOS
- Android

**Condición de aplicaciones cliente**

- Aplicaciones móviles y aplicaciones de escritorio

**Control de acceso**

- Requerir aplicación cliente aprobada

Para obtener más información, consulte [Acceso condicional basado en aplicaciones de Azure AD](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access).

---

### <a name="manage-azure-ad-devices-in-the-azure-portal"></a>Administración de dispositivos de Azure AD en Azure Portal

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración y registro de dispositivos Protección y seguridad de la identidad

Ahora podrá encontrar todos los dispositivos conectados a Azure AD y las actividades relacionadas con dispositivos en un solo lugar. Hay una nueva experiencia de administración para administrar todas las configuraciones e identidades de dispositivos en Azure Portal. En esta versión podrá hacer lo siguiente:

- Ver todos los dispositivos que están disponibles para el acceso condicional en Azure AD.
- Ver propiedades, incluidos los dispositivos unidos a Azure AD híbrido.
- Encontrar las claves de BitLocker para los dispositivos unidos a Azure AD, administrar el dispositivo con Intune y mucho más.
- Administrar la configuración relacionada con dispositivos de Azure AD.

Para obtener más información, consulte [Administración de dispositivos con Azure Portal](https://docs.microsoft.com/azure/active-directory/device-management-azure-portal).

---

### <a name="support-for-macos-as-a-device-platform-for-azure-ad-conditional-access"></a>Compatibilidad con macOS como plataforma de dispositivos para el acceso condicional de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Acceso condicional Protección y seguridad de la identidad

Ahora puede incluir (o excluir) macOS como condición de plataforma de dispositivos en la directiva de acceso condicional de Azure AD. Con la incorporación de macOS a las plataformas de dispositivos compatibles, puede:

- **Inscribir y administrar dispositivos Mac OS con Intune.** Al igual que en otras plataformas, como iOS y Android, existe una aplicación de portal de empresa para macOS que permite realizar inscripciones unificadas. Utilice la nueva aplicación de portal de empresa para macOS para inscribir un dispositivo con Intune y registrarlo con Azure AD.
- **Asegurarse de que los dispositivos macOS se ajustan a las directivas de cumplimiento de su organización definidas en Intune.** En Intune en Azure Portal, ahora podrá configurar directivas de cumplimiento para dispositivos macOS.
- **Restringir el acceso a las aplicaciones de Azure AD a solo los dispositivos macOS que cumplan las directivas.** La creación de directivas de acceso condicional usa macOS como una opción de plataforma de dispositivos independiente. Ahora puede crear directivas de acceso condicional específicas de macOS para el conjunto de aplicaciones de destino en Azure.

Para más información, consulte:

- [Creación de una directiva de cumplimiento para dispositivos macOS con Intune](https://aka.ms/macoscompliancepolicy)
- [Acceso condicional en Azure AD](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-azure-portal)

---

### <a name="network-policy-server-extension-for-azure-multi-factor-authentication"></a>Extensión Servidor de directivas de redes para Azure Multi-Factor Authentication

**Tipo:** **Categoría del servicio:** Nueva característica  **Funcionalidad del producto**: Multi-Factor Authentication Autenticación de usuarios

La extensión Servidor de directivas de redes para Azure Multi-Factor Authentication incorpora funcionalidades de Multi-Factor Authentication basadas en la nube para su infraestructura de autenticación mediante los servidores existentes. Con la extensión Servidor de directivas de redes, podrá agregar mecanismos de verificación mediante llamadas de teléfono, mensajes de texto o aplicaciones de teléfono al flujo de autenticación existente. Para ello, no tendrá que instalar, configurar ni mantener servidores nuevos.

Esta extensión se creó para las organizaciones que quieren proteger las conexiones de redes privadas virtuales sin tener que implementar el Servidor Microsoft Azure Multi-Factor Authentication. La extensión Servidor de directivas de redes actúa como un adaptador entre RADIUS y Azure Multi-Factor Authentication basada en la nube para proporcionar un segundo factor de autenticación a los usuarios federados o sincronizados.

Para obtener más información, consulte [Integración de la infraestructura existente de NPS con Azure Multi-Factor Authentication](https://docs.microsoft.com/azure/multi-factor-authentication/multi-factor-authentication-nps-extension).

---

### <a name="restore-or-permanently-remove-deleted-users"></a>Restauración o eliminación permanente de usuarios eliminados

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración de usuarios Directorio

En el centro de administración de Azure AD, ahora puede:

- Restaurar un usuario eliminado.
- Eliminar un usuario permanentemente.

**Para probarlo:**

1. En el centro de administración de Azure AD, seleccione [Todos los usuarios](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/UserManagementMenuBlade/All) en la sección **Administrar**.

2. En la lista **Mostrar**, seleccione **Usuarios eliminados recientemente**.

3. Seleccione uno o varios usuarios eliminados recientemente y después restáurelos o elimínelos permanentemente.

---

### <a name="new-approved-client-apps-for-azure-ad-app-based-conditional-access"></a>Nuevas aplicaciones cliente aprobadas para el acceso condicional basado en aplicaciones de Azure AD

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Acceso condicional Protección y seguridad de la identidad

Las siguientes aplicaciones se agregaron a la lista de [aplicaciones cliente aprobadas](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-conditional-access-conditions#client-apps-preview):

- Microsoft Planner
- Azure Information Protection

Para más información, consulte:

- [Requisito de aplicación cliente aprobada](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-conditional-access-conditions#client-apps-preview)
- [Acceso condicional basado en aplicaciones de Azure AD](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access)

---

### <a name="use-or-between-controls-in-a-conditional-access-policy"></a>Uso del operador "OR" entre controles de una directiva de acceso condicional

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Acceso condicional Protección y seguridad de la identidad

Ahora puede usar el operador "OR" (requerir uno de los controles seleccionados) en los controles de acceso condicional. Puede usar esta característica para crear directivas con el operador "OR" entre controles de acceso. Por ejemplo, puede usar esta característica para crear una directiva que requiera que un usuario inicie sesión mediante Multi-Factor Authentication "O" que esté en un dispositivo compatible.

Para obtener más información, consulte [Controles en el acceso condicional de Azure AD](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-controls).

---

### <a name="aggregation-of-real-time-risk-detections"></a>Agregación de detecciones de riesgos en tiempo real

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Identity Protection Protección y seguridad de la identidad

En Azure AD Identity Protection, todas las detecciones de riesgos en tiempo real que se han originado desde la misma dirección IP en un día determinado ahora se agregan para cada tipo de detección de riesgos. Este cambio limita el volumen de las detecciones de riesgos que se muestran sin ningún cambio en la seguridad del usuario.

La detección en tiempo real subyacente funciona cada vez que el usuario inicia sesión. Si tiene configurada una directiva de seguridad de riesgo de inicio de sesión para Multi-Factor Authentication o para bloquear el acceso, esta se desencadena durante cada inicio de sesión con riesgo.

---

## <a name="october-2017"></a>Octubre de 2017

### <a name="deprecate-azure-ad-reports"></a>Informes de Azure AD obsoletos

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto**: Informes Administración del ciclo de vida de la identidad

Azure Portal proporciona lo siguiente:

- Una nueva consola de administración de Azure AD.
- Nuevas API de informes de actividades y de seguridad.

Debido a estas nuevas funcionalidades, las API de informes que se encuentran en el punto de conexión /reports se retiraron el 10 de diciembre de 2017.

---

### <a name="automatic-sign-in-field-detection"></a>Detección automática de campos de inicio de sesión

**Tipo:** **Categoría del servicio:** Corregida **Funcionalidad del producto:** Mis aplicaciones Inicio de sesión único

Azure AD admite la detección automática de campos de inicio de sesión para las aplicaciones que presentan un campo de nombre de usuario y contraseña HTML. Estos pasos se documentan en [Captura automática de campos de inicio de sesión para una aplicación](https://docs.microsoft.com/azure/active-directory/manage-apps/configure-password-single-sign-on-non-gallery-applications-problems#manually-capture-sign-in-fields-for-an-app). Puede encontrar esta funcionalidad mediante la adición de una aplicación *situada fuera de la galería* en la página **Aplicaciones empresariales** en [Azure Portal](https://aad.portal.azure.com). Además, en esta nueva aplicación podrá configurar el modo **Inicio de sesión único** en **Inicio de sesión único basado en contraseña**, especificar una URL web y, a continuación, guardar la página.

Debido a un problema del servicio, esta funcionalidad se deshabilitó temporalmente. El problema se ha resuelto y la detección automática del campo de inicio de sesión vuelve a estar disponible.

---

### <a name="new-multi-factor-authentication-features"></a>Nuevas características de Multi-Factor Authentication

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Multi-Factor Authentication Protección y seguridad de la identidad

Multi-Factor Authentication (MFA) es un componente esencial para la protección de su organización. Para hacer que las credenciales tengan mayor capacidad de adaptación y que la experiencia resulte más sencilla, se han agregado las siguientes características:

- Integración de los resultados del desafío multifactor directamente en el informe de inicio de sesión de Azure AD, incluido el acceso mediante programación a los resultados de MFA.
- Integración más profunda de la configuración de MFA en la experiencia de configuración de Azure AD en Azure Portal.

Con esta versión preliminar pública, los informes y la administración de MFA son una parte integrada de la experiencia de configuración principal de AD Azure. Ahora podrá administrar la funcionalidad del portal de administración de MFA en la experiencia de Azure AD.

Para obtener más información, consulte [Referencia para los informes de la autenticación multifactor en Azure Portal](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-activity-sign-ins-mfa).

---

### <a name="terms-of-use"></a>Términos de uso

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Cumplimiento normativo

Puede usar los términos de uso de Azure AD para presentar información a los usuarios, como, por ejemplo, renuncias relevantes para los requisitos legales o de cumplimiento.

Los términos de uso de Azure AD puede utilizarse en los escenarios siguientes:

- Términos de uso generales para todos los usuarios de su organización.
- Términos de uso específicos basados en los atributos de un usuario (por ejemplo, médicos frente a enfermeras o empleados nacionales frente a internacionales, creados por grupos dinámicos).
- Términos de uso específicos para el acceso a aplicaciones empresariales de alto impacto, como, por ejemplo, Salesforce.

Para obtener más información, consulte [Términos de uso de Azure AD](https://docs.microsoft.com/azure/active-directory/conditional-access/terms-of-use).

---

### <a name="enhancements-to-privileged-identity-management"></a>Mejoras en Privileged Identity Management

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Privileged Identity Management Privileged Identity Management

Con Privileged Identity Management (PIM) de Azure AD, podrá administrar, controlar y supervisar el acceso a los recursos de Azure (vista preliminar) dentro de su organización por parte de:

- Suscripciones
- Grupos de recursos
- Máquinas virtuales

Todos los recursos de Azure Portal que usan la funcionalidad RBAC de Azure pueden usar las funcionalidades de seguridad y administración del ciclo de vida que ofrece Privileged Identity Management de Azure AD.

Para obtener más información, consulte [PIM para recursos de Azure (versión preliminar)](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/azure-pim-resource-rbac).

---

### <a name="access-reviews"></a>Revisiones de acceso

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Revisiones de acceso Cumplimiento normativo

Las organizaciones pueden usar las revisiones de acceso (versión preliminar) para administrar de manera eficaz las pertenencias a grupos y el acceso a las aplicaciones empresariales:

- Puede volver a certificar el acceso de usuario invitado mediante las revisiones de acceso a las aplicaciones y la pertenencia a grupos. La información que proporcionan las revisiones de acceso permite a los revisores decidir de manera eficaz si deben permitir el acceso continuado a los invitados.
- Puede volver a certificar el acceso de los empleados a las aplicaciones y la pertenencia a grupos con las revisiones de acceso.

Puede recopilar los controles de revisiones de acceso en programas importantes para que su organización realice un seguimiento de las revisiones de aplicaciones vulnerables o de cumplimiento normativo.

Para obtener más información, consulte [Revisiones de acceso de Azure AD](https://docs.microsoft.com/azure/active-directory/active-directory-azure-ad-controls-access-reviews-overview).

---

### <a name="hide-third-party-applications-from-my-apps-and-the-office-365-app-launcher"></a>Ocultación de aplicaciones de terceros de Mis aplicaciones y del iniciador de aplicaciones de Office 365

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Mis aplicaciones Inicio de sesión único

Ahora podrá administrar mejor las aplicaciones que se muestran en los portales de usuario con la nueva propiedad **hide app**. Puede ocultar aplicaciones para ayudar en casos en los que se muestran iconos de aplicaciones para servicios back-end o iconos duplicados que se acumulan en los iniciadores de aplicaciones de usuarios. El botón de alternancia se encuentra en la sección **Propiedades** de la aplicación de terceros con la etiqueta **Visible to user?** (¿Visible para el usuario?). También puede ocultar una aplicación mediante programación con PowerShell.

Para obtener más información, consulte [Ocultación de una aplicación de la experiencia del usuario en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-hide-third-party-app).


**¿Qué está disponible?**

 Como parte de la transición a la nueva consola de administración, hay disponibles dos nuevas API que permiten recuperar los registros de actividad de Azure AD. El nuevo conjunto de API proporciona más funciones de filtro y ordenación, así como actividades de auditoría e inicio de sesión. Los datos que estaban disponibles anteriormente a través de los informes de seguridad ahora se proporcionan mediante la API de detecciones de riesgos de Identity Protection en Microsoft Graph.


## <a name="september-2017"></a>Septiembre de 2017

### <a name="hotfix-for-identity-manager"></a>Revisión para Identity Manager

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Identity Manager Administración del ciclo de vida de la identidad

Un paquete acumulativo de revisiones (compilación 4.4.1642.0) está disponible desde el 25 de septiembre de 2017 para Microsoft Identity Manager 2016 Service Pack 1. Este paquete acumulativo de revisiones:

- Resuelve problemas e incorpora mejoras.
- Es una actualización acumulativa que reemplaza a todas las actualizaciones de Identity Manager 2016 Service Pack 1 hasta la compilación 4.4.1459.0 de Identity Manager 2016.
- Requiere disponer de Identity Manager 2016, compilación 4.4.1302.0.

Para obtener más información, consulte [Paquete acumulativo de revisiones (compilación 4.4.1642.0) está disponible para Microsoft Identity Manager SP1 de 2016](https://support.microsoft.com/help/4021562).

---
