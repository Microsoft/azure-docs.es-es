---
title: Novedades Notas de la versión de Azure Active Directory | Microsoft Docs
description: Obtenga información acerca de las novedades de Azure Active Directory, como, por ejemplo, las notas de la versión más recientes, los problemas conocidos, las correcciones de errores, las funcionalidades en desuso y los próximos cambios.
services: active-directory
author: eross-msft
manager: daveba
featureFlags:
- clicktale
ms.assetid: 06a149f7-4aa1-4fb9-a8ec-ac2633b031fb
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 02/28/2019
ms.author: lizross
ms.reviewer: dhanyahk
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 455be9ba95975ba5c3bd870cbe01ca916d85ef41
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544423"
---
# <a name="whats-new-in-azure-active-directory"></a>¿Cuáles son las novedades de Azure Active Directory?

>Recibir notificaciones sobre cuándo se debe volver a visitar esta página para las actualizaciones, copie y pegue esta dirección URL: `https://docs.microsoft.com/api/search/rss?search=%22release+notes+for+azure+AD%22&locale=en-us` en su ![icono de lector RSS sobre](./media/whats-new/feed-icon-16x16.png) lector de fuentes.

En Azure AD se realizan mejoras de forma continua. Para mantenerse al día con los avances más recientes, este artículo proporciona información acerca de los elementos siguientes:

- Versiones más recientes
- Problemas conocidos
- Corrección de errores
- Funciones obsoletas
- Planes de cambios

Esta página se actualiza mensualmente, por lo que se recomienda visitarla con frecuencia. Si busca elementos que tengan más de 6 meses, puede encontrarlos en el [Archivo de novedades de Azure Active Directory](whats-new-archive.md).

---

## <a name="april-2019"></a>Abril de 2019

### <a name="azure-active-directory-azure-ad-entitlement-management-is-now-available-public-preview"></a>Administración de derechos de Azure Active Directory (Azure AD) está ahora disponible (versión preliminar)

**Tipo:** Nueva característica  
**Categoría del servicio:** Identity Governance  
**Funcionalidad del producto:** Identity Governance

Administración de derechos de Azure AD, ahora en versión preliminar pública, permite a los clientes para delegar la administración de paquetes de acceso, que define cómo los empleados y socios comerciales pueden solicitar acceso, quién debe aprobar y cuánto tienen acceso. Suscripción de Azure AD y Office 365 grupos, asignaciones de roles en aplicaciones empresariales y las asignaciones de roles para sitios de SharePoint Online, pueden administrar paquetes de acceso. Obtenga más información sobre la administración de derechos en el [información general de administración de derechos de Azure AD](https://docs.microsoft.com/azure/active-directory/governance/entitlement-management-overview). Para obtener más información sobre la gran variedad de características de gobierno de identidades de Azure AD, incluidos Privileged Identity Management, las revisiones de acceso y los términos de uso, consulte [What ' s gobierno de identidades de Azure AD?](../governance/identity-governance-overview.md).

---

### <a name="configure-a-naming-policy-for-office-365-groups-in-azure-ad-portal-public-preview"></a>Configurar una directiva de nomenclatura para grupos de Office 365 en el portal de Azure AD (versión preliminar)

**Tipo:** Nueva característica  
**Categoría del servicio:** Administración de grupos  
**Funcionalidad del producto:** Colaboración

Los administradores ahora pueden configurar una directiva de nomenclatura de grupos de Office 365, mediante el portal de Azure AD. Este cambio ayuda a aplicar las convenciones de nomenclatura coherentes para los grupos de Office 365 creados o editados por los usuarios de su organización. 

Puede configurar la directiva de nomenclatura para grupos de Office 365 de dos maneras diferentes:

- Definir prefijos o sufijos, que se agregan automáticamente a un nombre de grupo.

- Cargue un conjunto personalizado de palabras bloqueadas para su organización, que no se permiten en los nombres de grupo (por ejemplo, "CEO, nómina, RRHH").

Para obtener más información, consulte [aplicar una directiva de nomenclatura para grupos de Office 365](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-naming-policy).

---

### <a name="azure-ad-activity-logs-are-now-available-in-azure-monitor-general-availability"></a>Registros de actividad de los anuncios de Azure ahora están disponibles en Azure Monitor (Disponibilidad General)

**Tipo:** Nueva característica  
**Categoría del servicio:** Informes  
**Funcionalidad del producto:** Supervisión e informes

Para ayudar a resolver sus comentarios acerca de las visualizaciones con los registros de actividad de Azure AD, hemos introducido una nueva característica de información en Log Analytics. Esta característica le ayuda a obtener información acerca de los recursos de Azure AD mediante el uso de nuestras plantillas interactivos, llamados libros. Estos pregeneradas libros pueden proporcionar detalles para las aplicaciones o usuarios e incluyen:

- **Inicios de sesión.** Proporciona detalles para las aplicaciones y los usuarios, incluidos la ubicación de inicio de sesión, el sistema operativo en uso o cliente de explorador y versión y el número de inicios de sesión correctas o erróneas.

- **Autenticación heredado y el acceso condicional.** Proporciona detalles para las aplicaciones y los usuarios mediante la autenticación heredada, incluido el uso de autenticación multifactor desencadenado por las directivas de acceso condicional, las aplicaciones mediante directivas de acceso condicional, y así sucesivamente.

- **Error de inicio de sesión de análisis.** Le ayuda a determinar si se producen los errores de inicio de sesión debido a la infraestructura, problemas de directiva o una acción del usuario.

- **Informes personalizados.** Puede crear o Editar libros existentes para ayudar a personalizar la característica de información para su organización.

Para obtener más información, consulte [cómo utilizar los libros de Azure Monitor para los informes de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-use-azure-monitor-workbooks).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---april-2019"></a>Nuevas aplicaciones federadas disponibles en la Galería de aplicaciones de Azure AD - abril de 2019

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros

En abril de 2019, hemos agregado compatibilidad con estas 21 nuevas aplicaciones con la federación a la Galería de aplicaciones:

[SAP Fiori](https://docs.microsoft.com/azure/active-directory/saas-apps/sap-fiori-tutorial), [HRworks Single Sign-On](https://docs.microsoft.com/azure/active-directory/saas-apps/hrworks-single-sign-on-tutorial), [estar presentes](https://docs.microsoft.com/azure/active-directory/saas-apps/percolate-tutorial), [MobiControl](https://docs.microsoft.com/azure/active-directory/saas-apps/mobicontrol-tutorial), [Citrix NetScaler](https://docs.microsoft.com/azure/active-directory/saas-apps/citrix-netscaler-tutorial), [ Shibumi](https://docs.microsoft.com/azure/active-directory/saas-apps/shibumi-tutorial), [Benchling](https://docs.microsoft.com/azure/active-directory/saas-apps/benchling-tutorial), [MileIQ](https://mileiq.onelink.me/991934284/7e980085), [PageDNA](https://docs.microsoft.com/azure/active-directory/saas-apps/pagedna-tutorial), [EduBrite LMS](https://docs.microsoft.com/azure/active-directory/saas-apps/edubrite-lms-tutorial), [RStudio Conectar](https://docs.microsoft.com/azure/active-directory/saas-apps/rstudio-connect-tutorial), [AMM](https://docs.microsoft.com/azure/active-directory/saas-apps/amms-tutorial), [Mitel conectar](https://docs.microsoft.com/azure/active-directory/saas-apps/mitel-connect-tutorial), [Alibaba en la nube (SSO basado en roles)](https://docs.microsoft.com/azure/active-directory/saas-apps/alibaba-cloud-service-role-based-sso-tutorial), [Certent capital Management](https://docs.microsoft.com/azure/active-directory/saas-apps/certent-equity-management-tutorial), [Sectigo Certificate Manager](https://docs.microsoft.com/azure/active-directory/saas-apps/sectigo-certificate-manager-tutorial), [GreenOrbit](https://docs.microsoft.com/azure/active-directory/saas-apps/greenorbit-tutorial), [Workgrid](https://docs.microsoft.com/azure/active-directory/saas-apps/workgrid-tutorial), [monday.com](https://docs.microsoft.com/azure/active-directory/saas-apps/mondaycom-tutorial), [ Enterprise SurveyMonkey](https://docs.microsoft.com/azure/active-directory/saas-apps/surveymonkey-enterprise-tutorial), [Indiggo](https://indiggolead.com/)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="new-access-reviews-frequency-option-and-multiple-role-selection"></a>Opción de frecuencia y selección de rol múltiple de revisiones de acceso nuevo

**Tipo:** Nueva característica  
**Categoría del servicio:** Revisiones de acceso  
**Funcionalidad del producto:** Identity Governance

Las revisiones de acceso de las nuevas actualizaciones en Azure AD le permiten:

- Cambio de revisiones de la frecuencia de los derechos de acceso a **semestrales**, además de las opciones existentes de semanalmente, mensualmente, trimestralmente y anualmente.

- Seleccione varios Azure AD y revisión los roles de recursos de Azure al crear un único acceso. En esta situación, todos los roles se configuran con la misma configuración y se notifica a todos los revisores al mismo tiempo.

Para obtener más información sobre cómo crear una revisión de acceso, consulte [crear una revisión de acceso de los grupos o las revisiones de acceso de las aplicaciones de Azure AD](https://docs.microsoft.com/azure/active-directory/governance/create-access-review).

---

### <a name="azure-ad-connect-email-alert-systems-are-transitioning-sending-new-email-sender-information-for-some-customers"></a>Están realizando la transición de Azure AD Connect sistemas de alerta por correo electrónico, enviar la nueva información del remitente de correo electrónico para algunos clientes

**Tipo:** Característica modificada  
**Categoría del servicio:** Sincronización de AD  
**Funcionalidad del producto:** Plataforma

Azure AD Connect está en proceso de transición de nuestros sistemas de alertas de correo electrónico, potencialmente, que muestra a algunos clientes un remitente de correo electrónico nuevo. Para solucionar este problema, debe agregar `azure-noreply@microsoft.com` a la lista blanca de su organización o no podrá continuar recibiendo alertas importantes desde su Office 365, Azure o los servicios de sincronización.

---

### <a name="upn-suffix-changes-are-now-successful-between-federated-domains-in-azure-ad-connect"></a>Los cambios de sufijo UPN ahora son correctas entre dominios federados en Azure AD Connect

**Tipo:** Corregido  
**Categoría del servicio:** Sincronización de AD  
**Funcionalidad del producto:** Plataforma

Ahora puede cambiar correctamente sufijo UPN de un usuario de un dominio federado a otro dominio federado en Azure AD Connect. Esta corrección significa que ya no debería experimentar el mensaje de error FederatedDomainChangeError durante el ciclo de sincronización o recibir una notificación por correo electrónico que indica que, "no se puede actualizar este objeto en Azure Active Directory, porque el [atributo FederatedUser.UserPrincipalName], no es válido. Actualice el valor en los servicios de directorio local".

Para obtener más información, consulte [solución de problemas de errores durante la sincronización](https://docs.microsoft.com/azure/active-directory/hybrid/tshoot-connect-sync-errors#federateddomainchangeerror).

---

### <a name="increased-security-using-the-app-protection-based-conditional-access-policy-in-azure-ad-public-preview"></a>Aumentar la seguridad mediante la directiva de acceso condicional basado en la protección de aplicaciones de Azure AD (versión preliminar)

**Tipo:** Nueva característica  
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad

Acceso condicional basado en protección de aplicación ahora está disponible mediante la **requerir protección de aplicaciones** directiva. Esta nueva directiva ayuda a aumentar la seguridad de su organización al ayudar a evitar que:

- Usuarios obtengan acceso a las aplicaciones sin una licencia de Microsoft Intune.

- Usuarios no podrán obtener una directiva de protección de aplicaciones de Microsoft Intune.

- Usuarios obtengan acceso a las aplicaciones sin una directiva de protección de aplicaciones de Microsoft Intune configurada.

Para obtener más información, consulte [cómo requerir la directiva de protección de aplicaciones para el acceso a la aplicación en la nube con acceso condicional](https://docs.microsoft.com/azure/active-directory/conditional-access/app-protection-based-conditional-access).

---

### <a name="new-support-for-azure-ad-single-sign-on-and-conditional-access-in-microsoft-edge-public-preview"></a>Nueva compatibilidad con Azure AD único inicio de sesión y del acceso condicional en Microsoft Edge (versión preliminar)

**Tipo:** Nueva característica  
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad

Hemos mejorado la compatibilidad de Azure AD para Microsoft Edge, incluido que proporciona nueva compatibilidad para Azure AD único inicio de sesión y del acceso condicional. Si anteriormente usó Microsoft Intune Managed Browser, ahora puede usar Microsoft Edge en su lugar.

Para obtener más información sobre cómo configurar y administrar los dispositivos y aplicaciones con acceso condicional, consulte [requieren administrar los dispositivos de acceso a la aplicación en la nube con acceso condicional](https://docs.microsoft.com/azure/active-directory/conditional-access/require-managed-devices) y [requieren las aplicaciones de cliente para la nube aprobadas acceso a la aplicación con el acceso condicional](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access). Para obtener más información acerca de cómo administrar el acceso mediante Microsoft Edge con directivas de Microsoft Intune, consulte [acceso a administrar Internet mediante un explorador protegido por directivas de Microsoft Intune](https://docs.microsoft.com/intune/app-configuration-managed-browser).

---

## <a name="march-2019"></a>Marzo de 2019

### <a name="identity-experience-framework-and-custom-policy-support-in-azure-active-directory-b2c-is-now-available-ga"></a>Marco de experiencia de identidad y la directiva personalizada de soporte técnico en Azure Active Directory B2C ahora está disponible (GA)

**Tipo:** Nueva característica  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C

Ahora puede crear directivas personalizadas en Azure AD B2C, incluidas las tareas siguientes, que se admite a escala y bajo nuestro SLA de Azure:

- Crear y cargar recorridos de usuario de autenticación personalizada mediante directivas personalizadas.

- Describir los recorridos del usuario paso a paso como intercambios entre proveedores de notificaciones.

- Definir la creación de ramas condicional en recorridos del usuario.

- Transformar y asignar las notificaciones para su uso en decisiones en tiempo real y las comunicaciones.

- Usar servicios con la API de REST habilitada en los recorridos de usuario de autenticación personalizada. Por ejemplo, con los proveedores de correo electrónico, CRM y sistemas de autorización propietario.

- Federación con proveedores de identidades que son compatibles con el protocolo OpenIDConnect. Por ejemplo, con varios inquilinos de Azure AD, los proveedores de cuentas sociales o proveedores de verificación de dos factores.

Para obtener más información acerca de cómo crear directivas personalizadas, consulte [notas de desarrollador para las directivas personalizadas en Azure Active Directory B2C](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-developer-notes-custom) y leer [entrada de blog de Alex Simon, estudios de caso](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-B2C-custom-policies-to-build-your-own-identity-journeys/ba-p/382791).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---march-2019"></a>Nuevas aplicaciones federadas disponibles en la Galería de aplicaciones de Azure AD - marzo de 2019

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros

En marzo de 2019, hemos agregado compatibilidad con estas nuevas 14 aplicaciones con la federación a la Galería de aplicaciones:

[Delegado de Exchange Mobile ISEC7](https://www.isec7.com/english/), [MediusFlow](https://office365.cloudapp.mediusflow.com/), [ePlatform](https://docs.microsoft.com/azure/active-directory/saas-apps/eplatform-tutorial), [Fulcrum](https://docs.microsoft.com/azure/active-directory/saas-apps/fulcrum-tutorial), [ExcelityGlobal](https://docs.microsoft.com/azure/active-directory/saas-apps/excelityglobal-tutorial), [Sistema de auditoría basados en la explicación](https://docs.microsoft.com/azure/active-directory/saas-apps/explanation-based-auditing-system-tutorial), [Lean](https://docs.microsoft.com/azure/active-directory/saas-apps/lean-tutorial), [cuestiones de rendimiento de Powerschool](https://docs.microsoft.com/azure/active-directory/saas-apps/powerschool-performance-matters-tutorial), [Cinode](https://cinode.com/), [Iris Intranet](https://docs.microsoft.com/azure/active-directory/saas-apps/iris-intranet-tutorial), [Empactis](https://docs.microsoft.com/azure/active-directory/saas-apps/empactis-tutorial), [SmartDraw](https://docs.microsoft.com/azure/active-directory/saas-apps/smartdraw-tutorial), [Confirmit horizontes](https://docs.microsoft.com/azure/active-directory/saas-apps/confirmit-horizons-tutorial), [tareas](https://docs.microsoft.com/azure/active-directory/saas-apps/tas-tutorial)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="new-zscaler-and-atlassian-provisioning-connectors-in-the-azure-ad-gallery---march-2019"></a>Nuevo Zscaler y Atlassian aprovisionamiento conectores en la Galería de Azure AD: marzo de 2019

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Integración de terceros

Automatizar la creación, actualizar y eliminar cuentas de usuario para las siguientes aplicaciones:

[Zscaler](https://aka.ms/ZscalerProvisioning), [Zscaler Beta](https://aka.ms/ZscalerBetaProvisioning), [Zscaler One](https://aka.ms/ZscalerOneProvisioning), [Zscaler Two](https://aka.ms/ZscalerTwoProvisioning), [Zscaler tres](https://aka.ms/ZscalerThreeProvisioning), [Zscaler ZSCloud ](https://aka.ms/ZscalerZSCloudProvisioning), [Atlassian Cloud](https://aka.ms/atlassianCloudProvisioning)

Para obtener más información acerca de cómo proteger mejor su organización a través de aprovisionamiento de cuentas de usuario automatizado, consulte [automatizar el aprovisionamiento de usuarios para aplicaciones SaaS con Azure AD](https://aka.ms/ProvisioningDocumentation).

---

### <a name="restore-and-manage-your-deleted-office-365-groups-in-the-azure-ad-portal"></a>Restaurar y administrar los grupos eliminados de Office 365 en el portal de Azure AD

**Tipo:** Nueva característica  
**Categoría del servicio:** Administración de grupos  
**Funcionalidad del producto:** Colaboración

Ahora puede ver y administrar los grupos eliminados de Office 365 desde el portal de Azure AD. Este cambio le ayuda a ver qué grupos están disponibles para restaurar, junto con lo que le permite permanentemente, elimine todos los grupos que no son necesarios para su organización.

Para obtener más información, consulte [restauración caducado o grupos eliminados](https://docs.microsoft.com/azure/active-directory/users-groups-roles/groups-restore-deleted#view-and-manage-the-deleted-office-365-groups-that-are-available-to-restore).

---

### <a name="single-sign-on-is-now-available-for-azure-ad-saml-secured-on-premises-apps-through-application-proxy-public-preview"></a>Inicio de sesión único ya está disponible para las aplicaciones protegidas de SAML de Azure AD en el entorno local a través del Proxy de aplicación (versión preliminar)

**Tipo:** Nueva característica  
**Categoría del servicio:** Proxy de aplicaciones  
**Funcionalidad del producto:** Control de acceso

Ahora puede proporcionar una experiencia de inicio de sesión único (SSO) en local, las aplicaciones autenticadas SAML, junto con el acceso remoto a estas aplicaciones a través del Proxy de aplicación. Para obtener más información acerca de cómo configurar el SSO de SAML con sus aplicaciones locales, consulte [inicio de sesión único en SAML para aplicaciones locales con el Proxy de aplicación (versión preliminar)](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-configure-single-sign-on-on-premises-apps).

---

### <a name="client-apps-in-request-loops-will-be-interrupted-to-improve-reliability-and-user-experience"></a>Se interrumpirán las aplicaciones de cliente en los bucles de solicitud para mejorar la confiabilidad y experiencia del usuario

**Tipo:** Nueva característica  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Autenticación de usuarios

Las aplicaciones cliente pueden emitir incorrectamente cientos de las mismas solicitudes de inicio de sesión a través de un breve período de tiempo. Estas solicitudes, tanto si son correctas o no, contribuyen a una mala experiencia de usuario y ha aumentado la frecuencia de las cargas de trabajo para el proveedor de Identidades, aumentar la latencia para todos los usuarios y reducir la disponibilidad del IDP.

Esta actualización se envía una `invalid_grant` error: `AADSTS50196: The server terminated an operation because it encountered a loop while processing a request` a las aplicaciones cliente que emiten solicitudes de duplicación varias veces durante un breve período de tiempo más allá del ámbito de una operación normal. Las aplicaciones cliente que se produce este problema deben mostrar un aviso interactivo, pedir al usuario que vuelva a iniciarla. Para obtener más información acerca de este cambio y cómo corregir la aplicación si encuentra este error, consulte [cuáles son las novedades para la autenticación?](https://docs.microsoft.com/azure/active-directory/develop/reference-breaking-changes#looping-clients-will-be-interrupted).

---

### <a name="new-audit-logs-user-experience-now-available"></a>Nueva experiencia de usuario de los registros de auditoría ya está disponible

**Tipo:** Característica modificada  
**Categoría del servicio:** Informes  
**Funcionalidad del producto:** Supervisión e informes

Hemos creado una nueva de Azure AD **registros de auditoría** página para ayudar a mejorar la legibilidad y cómo busca la información. Para ver el nuevo **registros de auditoría** página, seleccione **registros de auditoría** en el **actividad** sección de Azure AD.

![Nueva página de registros de auditoría, con información de ejemplo](media/whats-new/audit-logs-page.png)

Para obtener más información sobre la nueva **registros de auditoría** página, vea [auditar los informes de actividad en el portal de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-audit-logs#audit-logs).

---

### <a name="new-warnings-and-guidance-to-help-prevent-accidental-administrator-lockout-from-misconfigured-conditional-access-policies"></a>Nuevas advertencias y orientación para ayudar a evitar el bloqueo accidental del Administrador de directivas de acceso condicional mal configurados

**Tipo:** Característica modificada  
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad

Para ayudar a impedir que los administradores accidentalmente bloqueen a sí mismos fuera de sus propios inquilinos a través de directivas de acceso condicional mal configuradas, hemos creado nuevas advertencias y guía actualizada en el portal de Azure. Para obtener más información acerca de la nueva guía, consulte [¿cuáles son las dependencias del servicio de acceso condicional de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/conditional-access/service-dependencies).

---

### <a name="improved-end-user-terms-of-use-experiences-on-mobile-devices"></a>Términos del usuario final mejorada de experiencias de uso en dispositivos móviles

**Tipo:** Característica modificada  
**Categoría del servicio:** Términos de uso  
**Funcionalidad del producto:** Gobernanza

Hemos actualizado nuestros términos de experiencias de uso existentes para ayudar a mejorar cómo revisar y dar su consentimiento a los términos de uso en un dispositivo móvil. Ahora puede acercar y alejar, vuelva atrás, descargar la información y seleccionar los hipervínculos. Para obtener más información acerca de los términos de uso actualizados, consulte [Azure Active Directory característica términos de uso](https://docs.microsoft.com/azure/active-directory/conditional-access/terms-of-use#what-terms-of-use-looks-like-for-users).

---

### <a name="new-azure-ad-activity-logs-download-experience-available"></a>Disponible de la experiencia de descarga de los nuevos registros de actividad de Azure AD

**Tipo:** Característica modificada  
**Categoría del servicio:** Informes  
**Funcionalidad del producto:** Supervisión e informes

Ahora puede descargar grandes cantidades de registros de actividad directamente desde el portal de Azure. Esta actualización le permite:

- Descargue hasta 250 000 filas.

- Recibir una notificación una vez finalizada la descarga.

- Personalizar el nombre del archivo.

- Determinar el formato de salida, JSON o CSV.

Para obtener más detalles sobre esta característica, consulte [inicio rápido: Descargar un informe de auditoría mediante el portal de Azure](https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-download-audit-report)

---

### <a name="breaking-change-updates-to-condition-evaluation-by-exchange-activesync-eas"></a>Cambio importante: Actualizaciones de evaluación de condición por Exchange ActiveSync (EAS)

**Tipo:** Plan de cambio  
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Control de acceso

Estamos en proceso de actualización de cómo Exchange ActiveSync (EAS) evalúa las condiciones siguientes:

- Ubicación del usuario, según el país, región o dirección IP

- Riesgo de inicio de sesión

- Plataforma de dispositivo

Si anteriormente ha usado estas condiciones en las directivas de acceso condicional, tenga en cuenta que puede cambiar el comportamiento de la condición. Por ejemplo, si usó anteriormente la condición de ubicación del usuario en una directiva, podría encontrar la directiva que se omiten ahora según la ubicación del usuario.

---

## <a name="february-2019"></a>Febrero de 2019

### <a name="configurable-azure-ad-saml-token-encryption-public-preview"></a>Cifrado de token SAML de Azure AD configurable (versión preliminar pública) 

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** SSO

Ahora puede configurar cualquier aplicación SAML admitida para recibir los tokens SAML cifrados. Cuando configura y usado con una aplicación, Azure AD cifra las aserciones de SAML emitidas mediante una clave pública obtenida de un certificado almacenado en Azure AD.

Para obtener más información acerca de cómo configurar el cifrado de tokens SAML, consulte [cifrado de tokens SAML de Azure AD Configure](https://docs.microsoft.com/azure/active-directory/manage-apps/howto-saml-token-encryption).

---

### <a name="create-an-access-review-for-groups-or-apps-using-azure-ad-access-reviews"></a>Creación de una revisión de acceso para grupos o aplicaciones con las revisiones de acceso de Azure AD

**Tipo:** Nueva característica  
**Categoría del servicio:** Revisiones de acceso  
**Funcionalidad del producto:** Gobernanza

Ahora puede incluir varios grupos o la revisión de pertenencia a grupos o asignación de aplicaciones de acceso de aplicaciones en una única instancia de AD Azure. Revisiones de acceso con varios grupos o las aplicaciones se configuran con la misma configuración y se notifica a todos los revisores incluye al mismo tiempo.

Para obtener más información acerca de cómo crear una revisión de acceso con las revisiones de acceso de Azure AD, consulte [crear una revisión de acceso de grupos o aplicaciones en las revisiones de acceso de Azure AD](https://docs.microsoft.com/azure/active-directory/governance/create-access-review)

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---february-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (febrero de 2019)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En febrero de 2019, hemos agregado compatibilidad con estas 27 nuevas aplicaciones con la federación a la Galería de aplicaciones:

[Passport Euromonitor](https://docs.microsoft.com/azure/active-directory/saas-apps/euromonitor-passport-tutorial), [MindTickle](https://docs.microsoft.com/azure/active-directory/saas-apps/mindtickle-tutorial), [INTENCIONADAS](https://seeforgetest-exxon.azurewebsites.net/Account/create?Length=7), [AirStack](https://docs.microsoft.com/azure/active-directory/saas-apps/airstack-tutorial), [Oracle Fusion ERP](https://docs.microsoft.com/azure/active-directory/saas-apps/oracle-fusion-erp-tutorial), [ IDrive](https://docs.microsoft.com/azure/active-directory/saas-apps/idrive-tutorial), [Qmlativ Skyward](https://docs.microsoft.com/azure/active-directory/saas-apps/skyward-qmlativ-tutorial), [Brightidea](https://docs.microsoft.com/azure/active-directory/saas-apps/brightidea-tutorial), [AlertOps](https://docs.microsoft.com/azure/active-directory/saas-apps/alertops-tutorial), [Soloinsight CloudGate SSO](https://docs.microsoft.com/azure/active-directory/saas-apps/soloinsight-cloudgate-sso-tutorial), Haga clic en permisos, [Brandfolder](https://docs.microsoft.com/azure/active-directory/saas-apps/brandfolder-tutorial), [StoregateSmartFile](https://docs.microsoft.com/azure/active-directory/saas-apps/smartfile-tutorial), [Pexip](https://docs.microsoft.com/azure/active-directory/saas-apps/pexip-tutorial), [Stormboard](https://docs.microsoft.com/azure/active-directory/saas-apps/stormboard-tutorial), [sísmicos ](https://docs.microsoft.com/azure/active-directory/saas-apps/seismic-tutorial), [Sueño de un recurso compartido de](https://www.shareadream.org/how-it-works), [Bugsnag](https://docs.microsoft.com/azure/active-directory/saas-apps/bugsnag-tutorial), [webMethods integración en la nube](https://docs.microsoft.com/azure/active-directory/saas-apps/webmethods-integration-cloud-tutorial), [conocimientos desde cualquier lugar LMS](https://docs.microsoft.com/azure/active-directory/saas-apps/knowledge-anywhere-lms-tutorial), [OU Campus](https://docs.microsoft.com/azure/active-directory/saas-apps/ou-campus-tutorial), [Periscope datos](https://docs.microsoft.com/azure/active-directory/saas-apps/periscope-data-tutorial), [Netop Portal](https://docs.microsoft.com/azure/active-directory/saas-apps/netop-portal-tutorial), [smartvid.io](https://docs.microsoft.com/azure/active-directory/saas-apps/smartvid.io-tutorial), [PureCloud por Genesys](https://docs.microsoft.com/azure/active-directory/saas-apps/purecloud-by-genesys-tutorial), [ClickUp plataforma de productividad](https://docs.microsoft.com/azure/active-directory/saas-apps/clickup-productivity-platform-tutorial)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="enhanced-combined-mfasspr-registration"></a>Mejora del registro MFA/SSPR combinado

**Tipo:** Característica modificada  
**Categoría del servicio:** Restablecimiento de contraseñas de autoservicio  
**Funcionalidad del producto:** Autenticación de usuarios
 
En respuesta a comentarios de los clientes, hemos mejorado la experiencia de vista previa de registro MFA o SSPR combinada, ayudando a los usuarios se registren más rápidamente su información de seguridad para SSPR y MFA. 

**Para activar la experiencia mejorada para los usuarios de hoy en día, siga estos pasos:**

1. Como administrador global o administrador de usuarios, inicie sesión en Azure portal y vaya a **Azure Active Directory > configuración de usuario > Administrar la configuración de características de vista previa del panel de acceso**. 

2. En el **usuarios que pueden utilizar la versión preliminar de características para registrar y administrar información de seguridad: actualizar** opción, elija la opción de activar las características para un **grupo seleccionado de usuarios** o para **todos los usuarios** .

Durante las próximas semanas, retiraremos la capacidad de activar el antiguo combinado MFA o SSPR registro experiencia de versión preliminar para los inquilinos que aún no lo tiene activado.

**Para ver si el control se quitará de su inquilino, siga estos pasos:**

1. Como administrador global o administrador de usuarios, inicie sesión en Azure portal y vaya a **Azure Active Directory > configuración de usuario > Administrar la configuración de características de vista previa del panel de acceso**.  

2. Si el **usuarios que pueden utilizar las características de vista previa para registrar y administrar información de seguridad** opción está establecida en **ninguno**, la opción se quitará de su inquilino.

Independientemente de si anteriormente activó en el registro MFA o SSPR combinado antiguo, obtener una vista previa experiencia para los usuarios o no, se desactivará la experiencia anterior en el futuro. Por este motivo, se recomienda encarecidamente mover tan pronto como sea posible a la nueva experiencia mejorada.

Para obtener más información acerca de la experiencia de registro mejorada, consulte el [combinan de fantásticas mejoras para Azure AD MFA y experiencia de registro de restablecimiento de contraseña](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Cool-enhancements-to-the-Azure-AD-combined-MFA-and-password/ba-p/354271).

---

### <a name="updated-policy-management-experience-for-user-flows"></a>Experiencia actualizada de la administración de directivas para flujos de usuario

**Tipo:** Característica modificada  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C

Hemos actualizado el proceso de creación y administración de directivas para los flujos de usuario (anteriormente directivas integradas, conocidos como) sea más fácil. Esta nueva experiencia es ahora el predeterminado para todos los inquilinos de Azure AD.

Puede proporcionar comentarios adicionales y sugerencias mediante la sonrisa o desaprobación iconos en el **nos envíe sus comentarios** área en la parte superior de la pantalla del portal.

Para obtener más información sobre la nueva experiencia de administración de directivas, consulte el [Azure AD B2C tiene ahora la personalización de JavaScript y muchas características nuevas más](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-B2C-now-has-JavaScript-customization-and-many-more-new/ba-p/353595) blog.

---

### <a name="choose-specific-page-element-versions-provided-by-azure-ad-b2c"></a>Elección de versiones específicas de elementos de página proporcionadas por Azure AD B2C

**Tipo:** Nueva característica  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C

Ahora puede elegir una versión específica de los elementos de página proporcionados por Azure AD B2C. Al seleccionar una versión específica, puede probar las actualizaciones antes de que aparezcan en una página y se puede obtener un comportamiento predecible. Además, ahora puede elegir en aplicar las versiones de la página específica para permitir que las personalizaciones de JavaScript. Para activar esta característica, vaya a la **propiedades** página en los flujos de usuario.

Para obtener más información acerca de cómo elegir las versiones específicas de los elementos de página, vea el [Azure AD B2C tiene ahora la personalización de JavaScript y muchas características nuevas más](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-B2C-now-has-JavaScript-customization-and-many-more-new/ba-p/353595) blog.

---

### <a name="configurable-end-user-password-requirements-for-b2c-ga"></a>Requisitos de contraseña del usuario final configurables para B2C (GA)

**Tipo:** Nueva característica  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C

Ahora puede configurar la complejidad de la contraseña de su organización para los usuarios finales, en lugar de tener que usar su dispositivo nativo directiva de contraseña de Azure AD. Desde el **propiedades** hoja de su usuario flujos (anteriormente conocidos como las directivas integradas), puede elegir un nivel de complejidad de contraseña **Simple** o **seguro**, o bien puede crear un **personalizado** conjunto de requisitos.

Para obtener más información acerca de la configuración de requisito de complejidad de contraseña, consulte [configurar requisitos de complejidad de contraseñas en Azure Active Directory B2C](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-reference-password-complexity).

---

### <a name="new-default-templates-for-custom-branded-authentication-experiences"></a>Nuevas plantillas predeterminadas para experiencias de autenticación con marca personalizada

**Tipo:** Nueva característica  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C

Puede usar las nuevas plantillas de forma predeterminada, ubicadas en el **diseños de página** hoja de los flujos de usuario (anteriormente conocido como las directivas integradas) crear un personalizado con marca de experiencia de autenticación para los usuarios.

Para obtener más información sobre el uso de las plantillas, consulte [Azure AD B2C tiene ahora la personalización de JavaScript y muchas características nuevas más](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-B2C-now-has-JavaScript-customization-and-many-more-new/ba-p/353595).

---

## <a name="january-2019"></a>Enero de 2019

### <a name="active-directory-b2b-collaboration-using-one-time-passcode-authentication-public-preview"></a>Colaboración B2B de Active Directory mediante la autenticación con código de acceso de un solo uso (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C

Se ha introducido la autenticación con código de acceso de un solo uso (OTP) para los usuarios invitados de B2B que no pueden autenticarse por otros medios, como Azure AD, una cuenta Microsoft (MSA) o la federación de Google. Este nuevo método de autenticación significa que los usuarios invitados no tienen que crear una nueva cuenta Microsoft. En cambio, al canjear una invitación o acceder a un recurso compartido, un usuario invitado puede solicitar el envío de un código temporal a una dirección de correo electrónico. Con este código temporal, el usuario invitado puede seguir iniciando sesión.

Para más información, consulte [Autenticación con código de acceso de un solo uso de correo electrónico (versión preliminar)](https://docs.microsoft.com/azure/active-directory/b2b/one-time-passcode) y el blog [Azure AD makes sharing and collaboration seamless for any user with any account](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-makes-sharing-and-collaboration-seamless-for-any-user/ba-p/325949) (Azure AD simplifica el uso compartido y la colaboración para cualquier usuario con una cuenta).

### <a name="new-azure-ad-application-proxy-cookie-settings"></a>Nueva configuración de cookies de Azure AD Application Proxy

**Tipo:** Nueva característica  
**Categoría del servicio:** Proxy de aplicaciones  
**Funcionalidad del producto:** Control de acceso

Hemos incluido tres nuevas configuraciones de cookies, disponibles para las aplicaciones que se publican a través de Application Proxy:

- **Usar cookie solo HTTP.** Establece la marca **HTTPOnly** en las cookies de sesión y acceso de Application Proxy. La activación de esta configuración ofrece ventajas de seguridad adicionales, como ayuda para evitar la copia o modificación de cookies por medio de scripting del lado cliente. Se recomienda activar esta marca (elija **Sí**) para disfrutar de estas ventajas.

- **Usar cookies seguras.** Establece la marca **Segura** en las cookies de sesión y acceso de Application Proxy. La activación de esta configuración ofrece ventajas de seguridad adicionales, como la garantía de que las cookies se transmiten solo a través de canales seguros de TLS, como HTTPS. Se recomienda activar esta marca (elija **Sí**) para disfrutar de estas ventajas.

- **Usar cookies persistentes.** Impide que las cookies de acceso expiren cuando se cierra el explorador web. Estas cookies se mantienen vigentes durante toda la duración del token de acceso. Sin embargo, las cookies se restablecen si se alcanza la hora de expiración o si el usuario elimina manualmente la cookie. Se recomienda que mantenga la configuración predeterminada **No**, activando el ajuste solo para las aplicaciones anteriores que no comparten cookies entre procesos.

Para obtener más información acerca de las nuevas cookies, consulte [Configuración de las cookies para el acceso a aplicaciones locales en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-configure-cookie-settings).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---january-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (enero de 2019)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En enero de 2019, hemos agregado estas 35 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones:

[Firstbird](https://docs.microsoft.com/azure/active-directory/saas-apps/firstbird-tutorial), [Folloze](https://docs.microsoft.com/azure/active-directory/saas-apps/folloze-tutorial), [talento paleta](https://docs.microsoft.com/azure/active-directory/saas-apps/talent-palette-tutorial), [Infor CloudSuite](https://docs.microsoft.com/azure/active-directory/saas-apps/infor-cloud-suite-tutorial), [paraguas de Cisco](https://docs.microsoft.com/azure/active-directory/saas-apps/cisco-umbrella-tutorial), [Zscaler Administrador de acceso de Internet](https://docs.microsoft.com/azure/active-directory/saas-apps/zscaler-internet-access-administrator-tutorial), [aviso de caducidad](https://docs.microsoft.com/azure/active-directory/saas-apps/expiration-reminder-tutorial), [InstaVR Visor](https://docs.microsoft.com/azure/active-directory/saas-apps/instavr-viewer-tutorial), [CorpTax](https://docs.microsoft.com/azure/active-directory/saas-apps/corptax-tutorial), [verbo](https://app.verb.net/login), [OpenLattice](https://openlattice.com/agora), [TheOrgWiki](https://www.theorgwiki.com/signup), [Pavaso Digital cierre](https://docs.microsoft.com/azure/active-directory/saas-apps/pavaso-digital-close-tutorial), [GoodPractice Toolkit](https://docs.microsoft.com/azure/active-directory/saas-apps/goodpractice-toolkit-tutorial), [ En la nube Service PICCO](https://docs.microsoft.com/azure/active-directory/saas-apps/cloud-service-picco-tutorial), [AuditBoard](https://docs.microsoft.com/azure/active-directory/saas-apps/auditboard-tutorial), [iProva](https://docs.microsoft.com/azure/active-directory/saas-apps/iprova-tutorial), [Workable](https://docs.microsoft.com/azure/active-directory/saas-apps/workable-tutorial), [CallPlease](https://webapp.callplease.com/create-account/create-account.html), [Sistema SSO GTNexus](https://docs.microsoft.com/azure/active-directory/saas-apps/gtnexus-sso-module-tutorial), [CBRE ServiceInsight](https://docs.microsoft.com/azure/active-directory/saas-apps/cbre-serviceinsight-tutorial), [Deskradar](https://docs.microsoft.com/azure/active-directory/saas-apps/deskradar-tutorial), [Coralogixv](https://docs.microsoft.com/azure/active-directory/saas-apps/coralogix-tutorial), [Signagelive](https://docs.microsoft.com/azure/active-directory/saas-apps/signagelive-tutorial), [Áreas para Enterprise](https://docs.microsoft.com/azure/active-directory/saas-apps/ares-for-enterprise-tutorial), [K2 para Office 365](https://www.k2.com/O365), [Xledger](https://www.xledger.net/), [iDiD Manager](https://docs.microsoft.com/azure/active-directory/saas-apps/idid-manager-tutorial), [HighGear](https://docs.microsoft.com/azure/active-directory/saas-apps/highgear-tutorial), [Visitly](https://docs.microsoft.com/azure/active-directory/saas-apps/visitly-tutorial), [Korn transportar CLP](https://docs.microsoft.com/azure/active-directory/saas-apps/korn-ferry-alp-tutorial), [Acadia](https://docs.microsoft.com/azure/active-directory/saas-apps/acadia-tutorial), [Adoddle cSaas plataforma](https://docs.microsoft.com/azure/active-directory/saas-apps/adoddle-csaas-platform-tutorial)<!-- , [CaféX Portal (Meetings)](https://docs.microsoft.com/azure/active-directory/saas-apps/cafexportal-meetings-tutorial), [MazeMap Link](https://docs.microsoft.com/azure/active-directory/saas-apps/mazemaplink-tutorial)-->  

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="new-azure-ad-identity-protection-enhancements-public-preview"></a>Nuevas mejoras de Azure AD Identity Protection (versión preliminar pública)

**Tipo:** Característica modificada  
**Categoría del servicio:** Protección de identidad  
**Funcionalidad del producto:** Seguridad y protección de la identidad

Nos complace anunciar que hemos agregado las siguientes mejoras a la oferta de versión preliminar pública de Azure AD Identity Protection:

- Una interfaz de usuario actualizada y más integrada

- API adicionales

- Evaluación de riesgos mejorada a través de aprendizaje automático

- Alineación de todo el producto entre usuarios e inicios de sesión no seguros

Para obtener más información sobre las mejoras, consulte [What is Azure Active Directory Identity Protection (refreshed)?](https://aka.ms/IdentityProtectionDocs) (¿Qué es Azure Active Directory Identity Protection [actualizado]?) a fin de conocer los detalles y compartir sus ideas a través de los mensajes en el producto.

---

### <a name="new-app-lock-feature-for-the-microsoft-authenticator-app-on-ios-and-android-devices"></a>Nueva característica Bloqueo de aplicación para la aplicación Microsoft Authenticator en dispositivos iOS y Android

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicación Microsoft Authenticator  
**Funcionalidad del producto:** Seguridad y protección de la identidad

Para proteger sus códigos de acceso de un solo uso, la información de aplicación y la configuración de la aplicación, puede activar la característica Bloqueo de aplicación en la aplicación Microsoft Authenticator. Al activar el Bloqueo de aplicación se le pedirá que se autentique con su PIN o características biométricas cada vez que abra la aplicación Microsoft Authenticator.

Para obtener más información, vea las [Preguntas más frecuentes de la aplicación Microsoft Authenticator](https://docs.microsoft.com/azure/active-directory/user-help/microsoft-authenticator-app-faq).

---

### <a name="enhanced-azure-ad-privileged-identity-management-pim-export-capabilities"></a>Mejora de las capacidades de exportación de Azure AD Privileged Identity Management (PIM)

**Tipo:** Nueva característica  
**Categoría del servicio:** Privileged Identity Management  
**Funcionalidad del producto:** Privileged Identity Management

Los administradores de Privileged Identity Management (PIM) ahora pueden exportar todas las asignaciones de roles activas y válidas para un recurso específico, lo cual incluye las asignaciones de roles para todos los recursos secundarios. Anteriormente, a los administradores les resultaba difícil obtener una lista completa de asignaciones de roles para una suscripción y tenían que exportar dichas asignaciones para cada recurso específico.

Para obtener más información, consulte [Visualización de la actividad y del historial de auditoría para los roles de recursos de Azure en PIM](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/azure-pim-resource-rbac).

---

## <a name="novemberdecember-2018"></a>Noviembre/diciembre de 2018

### <a name="users-removed-from-synchronization-scope-no-longer-switch-to-cloud-only-accounts"></a>Los usuarios que se quitaron del ámbito de sincronización ya no podrán cambiarse a cuentas solo en la nube

**Tipo:** Corregido  
**Categoría del servicio:** User Management  
**Funcionalidad del producto:** Directorio

>[!Important]
>Hemos escuchado y comprendido la frustración que les genera esta corrección. Por lo tanto, hemos revertido este cambio hasta el momento en que podamos ofrecerle una implementación más sencilla para su organización.

Se ha corregido un error en el que la marca DirSyncEnabled de un usuario se cambiaba erróneamente a **False** cuando el objeto de Active Directory Domain Services se excluía del ámbito de sincronización y, después, se movía a la Papelera de reciclaje de Azure AD en el siguiente ciclo de sincronización. Como resultado de esta corrección, si el usuario se excluye del ámbito de sincronización y posteriormente se restaura desde la Papelera de reciclaje de Azure AD, la cuenta de dicho usuario permanece sincronizada desde la instancia local de AD, según lo previsto, y no se puede administrar en la nube, ya que su origen de autoridad (SoA) permanece como AD local.

Antes de esta corrección, había un problema cuando se cambiaba la marca DirSyncEnabled a False. Daba la impresión errónea de que estas cuentas se convertían en objetos solo en la nube y de que las cuentas se podían administrar en la nube. Sin embargo, las cuentas siguen conservando su SoA como propiedades locales y sincronizadas (atributos paralelos) procedentes de la instancia local de AD. Esta situación provocó varios problemas en Azure AD y en otras cargas de trabajo en la nube (como Exchange Online), ya que esperaban tratar estas cuentas como sincronizadas desde AD y, sin embargo, se comportaban como cuentas solo en la nube.

En este momento, la única manera de convertir realmente una cuenta sincronizada de AD en una cuenta de solo nube, es deshabilitando DirSync en el nivel del inquilino, lo que desencadena una operación de back-end para transferir el SoA. Este tipo de cambio de SoA requiere (pero no se limita a) la limpieza de todos los atributos locales relacionados (como los atributos paralelos y de LastDirSyncTime) y el envío de una señal a otras cargas de trabajo en la nube para convertir también su objeto respectivo en una cuenta solo en la nube.

Por lo tanto, esta corrección evita las actualizaciones directas sobre el atributo ImmutableID de un usuario sincronizado desde AD, lo que era necesario en algunos escenarios en el pasado. Por diseño, el atributo ImmutableID de un objeto en Azure AD, como su nombre indica, es inmutable. Existen nuevas características implementadas en los clientes de sincronización de Azure AD Connect y Azure AD Connect Health para abordar estos escenarios:

- **Actualizaciones del atributo ImmutableID a gran escala para muchos usuarios por etapas**
  
  Por ejemplo, debe realizar una migración entre bosques de AD DS larga. Solución: Use Azure AD Connect para la **Configuración del delimitador de origen** y, mientras el usuario migra, copie los valores del atributo ImmutableID existentes de Azure AD al atributo DS-Consistency-Guid del usuario de AD DS local del nuevo bosque. Para obtener más información, consulte [Uso de msDS-ConsistencyGuid como sourceAnchor](/azure/active-directory/hybrid/plan-connect-design-concepts#using-ms-ds-consistencyguid-as-sourceanchor).

- **Actualizaciones del atributo ImmutableID a gran escala para muchos usuarios de una sola vez**

  Por ejemplo, al implementar Azure AD Connect se cometió un error, y ahora es necesario cambiar el atributo SourceAnchor. Solución: Deshabilite DirSync en el nivel del inquilino y borre todos los valores de ImmutableID no válidos. Para obtener más información, vea [Desactivar la sincronización de directorios para Office 365](/office365/enterprise/turn-off-directory-synchronization).

- **Hacer coincidir un usuario local con un usuario existente de Azure AD** Por ejemplo, un usuario que se haya vuelto a crear en AD DS genera un duplicado en la cuenta de Azure AD en lugar de reasignarlo a una cuenta de Azure AD existente (objeto huérfano). Solución: Use Azure AD Connect Health en Azure Portal para reasignar el delimitador de origen o el atributo ImmutableID. Para obtener más información, consulte [Escenario del objeto huérfano](/azure/active-directory/hybrid/how-to-connect-health-diagnose-sync-errors#orphaned-object-scenario).

### <a name="breaking-change-updates-to-the-audit-and-sign-in-logs-schema-through-azure-monitor"></a>Cambio importante: Actualizaciones de los esquemas de registros de auditoría y de inicio de sesión mediante Azure Monitor

**Tipo:** Característica modificada  
**Categoría del servicio:** Informes  
**Funcionalidad del producto:** Supervisión e informes

Actualmente, estamos publicando los flujos de registro de auditorías e inicios de sesión mediante Azure Monitor, por lo que puede integrar fácilmente los archivos de registro con sus herramientas SIEM o con Log Analytics. Basándonos en sus comentarios, y en preparación para el anuncio de disponibilidad general para esta característica, vamos a realizar los siguientes cambios en el esquema. Estos cambios en los esquemas y las actualizaciones de documentación relacionadas se producirán en la primera semana de enero.

#### <a name="new-fields-in-the-audit-schema"></a>Nuevos campos en el esquema de auditoría
Vamos a agregar un nuevo campo **Tipo de operación**, para proporcionar el tipo de operación que se realiza en el recurso. Por ejemplo, **Agregar**, **Actualizar** o **Eliminar**.

#### <a name="changed-fields-in-the-audit-schema"></a>Campos modificados en el esquema de auditoría
Los siguientes campos han cambiado en el esquema de auditoría:

|Nombre del campo|Qué cambia|Valores anteriores|Nuevos valores|
|----------|------------|----------|----------|
|Categoría|Este era el campo **Nombre de servicio**. Ahora es el campo **Categorías de auditoría**. A **Nombre de servicio** se le ha cambiado el nombre por el de campo **loggedByService**.|<ul><li>Account Provisioning (Aprovisionamiento de cuentas)</li><li>Core Directory (Directorio principal)</li><li>Autoservicio de restablecimiento de contraseña</li></ul>|<ul><li>User Management</li><li>Administración de grupos</li><li>Administración de la aplicación</li></ul>|
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

**Tipo:** Característica modificada  
**Categoría del servicio:** Protección de identidad  
**Funcionalidad del producto:** Puntuaciones de riesgo

Las mejoras en el usuario relacionadas con la protección de identidades y en el motor de evaluación de riesgos de inicio de sesión pueden ayudar a mejorar la precisión y cobertura de los riesgos del usuario. Puede que los administradores hayan observado que el nivel de riesgo del usuario ya no está vinculado directamente al nivel de riesgo de detecciones específicas y que hay un aumento en el número y en el nivel de los eventos de inicio de sesión de riesgo.

El modelo de Machine Learning supervisado es el encargado ahora de evaluar las detecciones de riesgo. Este calcula el riesgo del usuario mediante características adicionales de los inicios de sesión del usuario y un patrón de detecciones. Según este modelo, el administrador puede detectar usuarios con puntuaciones de riesgo altas, incluso aunque las detecciones asociadas con ese usuario sean de nivel bajo o medio. 

---

### <a name="administrators-can-reset-their-own-password-using-the-microsoft-authenticator-app-public-preview"></a>Los administradores pueden restablecer sus propias contraseñas mediante la aplicación Microsoft Authenticator (versión preliminar pública)

**Tipo:** Característica modificada  
**Categoría del servicio:** Restablecimiento de contraseñas de autoservicio  
**Funcionalidad del producto:** Autenticación de usuarios

Los administradores de Azure AD ya pueden restablecer sus propias contraseñas mediante las notificaciones de la aplicación Microsoft Authenticator o mediante un código de cualquier aplicación de autenticación móvil, o token de hardware. Para restablecer sus propias contraseñas, los administradores ahora pueden usar dos de los métodos siguientes:

- Notificación de la aplicación Microsoft Authenticator

- Un código de otra aplicación de autenticación móvil o un token de hardware

- Email

- llamada de teléfono

- mensaje de texto

Para más información sobre el uso de la aplicación Microsoft Authenticator para restablecer contraseñas, consulte [Autoservicio de restablecimiento de contraseña de Azure AD: Aplicación móvil y SSPR (versión preliminar)](https://docs.microsoft.com/azure/active-directory/authentication/concept-sspr-howitworks#mobile-app-and-sspr-preview)

---

### <a name="new-azure-ad-cloud-device-administrator-role-public-preview"></a>Nuevo rol de administrador de dispositivos en la nube de Azure AD (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** Administración y registro de dispositivos  
**Funcionalidad del producto:** Control de acceso

Los administradores pueden asignar usuarios al nuevo rol de administrador de dispositivos en la nube para realizar las tareas propias de este rol. Los usuarios asignados a este rol pueden habilitar, deshabilitar y eliminar dispositivos en Azure AD y leer las claves de BitLocker de Windows 10 (si las hay) en Azure Portal.

Para más información acerca de los roles y permisos, consulte [Asignación de roles de administrador en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles).

---

### <a name="manage-your-devices-using-the-new-activity-timestamp-in-azure-ad-public-preview"></a>Administración de dispositivos con la nueva marca de tiempo de actividad de Azure AD (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** Administración y registro de dispositivos  
**Funcionalidad del producto:** Administración del ciclo de vida de dispositivos

Somos conscientes de que con el tiempo debe actualizar y retirar dispositivos de su organización en Azure AD, para evitar tener dispositivos obsoletos en su entorno. Para ayudarle con este proceso, Azure AD ya actualiza los dispositivos con una nueva marca de tiempo de actividad que le ayuda a administrar el ciclo de vida del dispositivo.

Para más información acerca de cómo obtener y usar esta marca de tiempo, consulte [Procedimiento: Administración de dispositivos obsoletos en Azure AD](https://docs.microsoft.com/azure/active-directory/devices/manage-stale-devices).

---

### <a name="administrators-can-require-users-to-accept-a-terms-of-use-on-each-device"></a>Los administradores pueden requerir que los usuarios acepten los términos de uso en cada dispositivo

**Tipo:** Nueva característica  
**Categoría del servicio:** Términos de uso  
**Funcionalidad del producto:** Gobernanza
 
Los administradores ahora pueden activar la opción **Requerir que los usuarios concedan su consentimiento en todos los dispositivos** para requerir que los usuarios acepten los términos de uso en todos los dispositivos que estén usando en su inquilino.

Para más información, consulte [la sección de términos de uso por dispositivo del artículo Característica Azure Active Directory Terms of Use](https://docs.microsoft.com/azure/active-directory/conditional-access/terms-of-use#per-device-terms-of-use).

---

### <a name="administrators-can-configure-a-terms-of-use-to-expire-based-on-a-recurring-schedule"></a>Los administradores pueden configurar la expiración de los términos de uso mediante una programación periódica

**Tipo:** Nueva característica  
**Categoría del servicio:** Términos de uso  
**Funcionalidad del producto:** Gobernanza
 

Los administradores pueden ahora activar la opción **Expirar autorizaciones** para hacer que los términos de uso expiren para todos los usuarios según la programación periódica especificada. La programación puede ser anual, semestral, trimestral o mensual. Después de que caduquen los términos de uso, los usuarios deben Aceptar.

Para más información, consulte [la sección Agregar términos de uso del artículo Característica Azure Active Directory Terms of Use](https://docs.microsoft.com/azure/active-directory/conditional-access/terms-of-use#add-terms-of-use).

---

### <a name="administrators-can-configure-a-terms-of-use-to-expire-based-on-each-users-schedule"></a>Los administradores pueden configurar la expiración de los términos de uso según una programación de cada usuario

**Tipo:** Nueva característica  
**Categoría del servicio:** Términos de uso  
**Funcionalidad del producto:** Gobernanza

Los administradores ahora pueden especificar una duración tras la cual el usuario debe volver a aceptar los términos de uso. Por ejemplo, los administradores pueden especificar que los usuarios deben volver a aceptar los términos de uso cada 90 días.

Para más información, consulte [la sección Agregar términos de uso del artículo Característica Azure Active Directory Terms of Use](https://docs.microsoft.com/azure/active-directory/conditional-access/terms-of-use#add-terms-of-use).
 
---

### <a name="new-azure-ad-privileged-identity-management-pim-emails-for-azure-active-directory-roles"></a>Nuevos correos electrónicos de Azure Active Directory Privileged Identity Management (PIM) para los roles de Azure Active Directory

**Tipo:** Nueva característica  
**Categoría del servicio:** Privileged Identity Management  
**Funcionalidad del producto:** Privileged Identity Management
 
Los clientes que usan Azure AD Privileged Identity Management (PIM) ahora pueden recibir un correo electrónico de resumen semanal que incluye la información siguiente de los últimos siete días:

- Introducción a las asignaciones de roles principales y permanentes

- Número de usuarios que activan roles

- Número de usuarios asignados a roles en PIM

- Número de usuarios asignados a roles fuera de PIM

- Número de usuarios a los que se ha "convertido en permanentes" en PIM

Para más información sobre PIM y las notificaciones de correo electrónico disponibles, consulte [Notificaciones por correo electrónico en PIM](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-email-notifications).

---

### <a name="group-based-licensing-is-now-generally-available"></a>Las licencias basadas en grupos tienen ahora disponibilidad general

**Tipo:** Característica modificada  
**Categoría del servicio:** Otros  
**Funcionalidad del producto:** Directorio

Las licencias basadas en grupos ya no están en la fase de versión preliminar pública sino en la de disponibilidad general. Como parte de esta versión general, se ha logrado que esta característica sea más escalable y se ha agregado la posibilidad de volver a procesar las asignaciones de licencias basadas en grupos para un solo usuario y la posibilidad de usar licencias basadas en grupos con licencias E3/A3 de Office 365.

Para más información acerca de las licencias basadas en grupos, consulte [¿En qué consisten las licencias basadas en grupos de Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-licensing-whatis-azure-portal)

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---november-2018"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: noviembre de 2018

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En noviembre de 2018, hemos agregado 26 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[CoreStack](https://cloud.corestack.io/site/login), [HubSpot](https://docs.microsoft.com/azure/active-directory/saas-apps/HubSpot-tutorial), [GetThere](https://docs.microsoft.com/azure/active-directory/saas-apps/getthere-tutorial), [Gra-Pe](https://docs.microsoft.com/azure/active-directory/saas-apps/grape-tutorial), [eHour](https://getehour.com/try-now), [Consent2Go](https://docs.microsoft.com/azure/active-directory/saas-apps/Consent2Go-tutorial), [Appinux](https://docs.microsoft.com/azure/active-directory/saas-apps/appinux-tutorial), [DriveDollar](https://www.drivedollar.com/Account/Login), [Useall](https://docs.microsoft.com/azure/active-directory/saas-apps/useall-tutorial), [Infinite Campus](https://docs.microsoft.com/azure/active-directory/saas-apps/infinitecampus-tutorial), [Alaya](https://alayagood.com/en/demo/), [HeyBuddy](https://docs.microsoft.com/azure/active-directory/saas-apps/heybuddy-tutorial), [Wrike SAML](https://docs.microsoft.com/azure/active-directory/saas-apps/wrike-tutorial), [Drift](https://docs.microsoft.com/azure/active-directory/saas-apps/drift-tutorial), [Zenegy for Business Central 365](https://accounting.zenegy.com/), [Everbridge Member Portal](https://docs.microsoft.com/azure/active-directory/saas-apps/everbridge-tutorial), [IDEO](https://profile.ideo.com/users/sign_up), [Ivanti Service Manager (ISM)](https://docs.microsoft.com/azure/active-directory/saas-apps/ivanti-service-manager-tutorial), [Peakon](https://docs.microsoft.com/azure/active-directory/saas-apps/peakon-tutorial), [Allbound SSO](https://docs.microsoft.com/azure/active-directory/saas-apps/allbound-sso-tutorial), [Plex Apps - Classic Test](https://test.plexonline.com/signon), [Plex Apps – Classic](https://www.plexonline.com/signon), [Plex Apps - UX Test](https://test.cloud.plex.com/sso), [Plex Apps – UX](https://cloud.plex.com/sso), [Plex Apps – IAM](https://accounts.plex.com/), [CRAFTS - Childcare Records, Attendance, & Financial Tracking System](https://getcrafts.ca/craftsregistration) 

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

## <a name="october-2018"></a>Octubre de 2018

### <a name="azure-ad-logs-now-work-with-azure-log-analytics-public-preview"></a>Los registros de Azure AD ahora funcionan con Azure Log Analytics (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** Informes  
**Funcionalidad del producto:** Supervisión e informes

Nos complace anunciar que ahora puede reenviar los registros de Azure AD a Azure Log Analytics. Esta característica tan solicitada le ayuda a obtener un acceso aún mejor al análisis para su negocio, las operaciones y la seguridad, además de contribuir a supervisar la infraestructura. Para más información, consulte el blog sobre [los registros de actividad de Azure Active Directory en Azure Log Analytics que ya está disponible](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-Active-Directory-Activity-logs-in-Azure-Log-Analytics-now/ba-p/274843).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---october-2018"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: octubre de 2018

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En octubre de 2018, hemos agregado 14 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[My Award Points](https://docs.microsoft.com/azure/active-directory/saas-apps/myawardpoints-tutorial), [Vibe HCM](https://docs.microsoft.com/azure/active-directory/saas-apps/vibehcm-tutorial), ambyint, [MyWorkDrive](https://docs.microsoft.com/azure/active-directory/saas-apps/myworkdrive-tutorial), [BorrowBox](https://docs.microsoft.com/azure/active-directory/saas-apps/borrowbox-tutorial), Dialpad, [ON24 Virtual Environment](https://docs.microsoft.com/azure/active-directory/saas-apps/on24-tutorial), [RingCentral](https://docs.microsoft.com/azure/active-directory/saas-apps/ringcentral-tutorial), [Zscaler Three](https://docs.microsoft.com/azure/active-directory/saas-apps/zscaler-three-tutorial), [Phraseanet](https://docs.microsoft.com/azure/active-directory/saas-apps/phraseanet-tutorial), [Appraisd](https://docs.microsoft.com/azure/active-directory/saas-apps/appraisd-tutorial), [Workspot Control](https://docs.microsoft.com/azure/active-directory/saas-apps/workspotcontrol-tutorial), [Shuccho Navi](https://docs.microsoft.com/azure/active-directory/saas-apps/shucchonavi-tutorial), [Glassfrog](https://docs.microsoft.com/azure/active-directory/saas-apps/glassfrog-tutorial)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](https://aka.ms/appstutorial). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](https://aka.ms/azureadapprequest).

---

### <a name="azure-ad-domain-services-email-notifications"></a>Notificaciones por correo electrónico de Azure AD Domain Services

**Tipo:** Nueva característica  
**Categoría del servicio:** Azure AD Domain Services  
**Funcionalidad del producto:** Azure AD Domain Services

Azure AD Domain Services proporciona alertas en Azure Portal sobre configuraciones incorrectas o problemas con el dominio administrado. Estas alertas incluyen guías paso a paso para que pueda intentar corregir los problemas sin tener que ponerse en contacto con el servicio de soporte técnico.

A partir de octubre, podrá personalizar la configuración de notificaciones para el dominio administrado por lo que, cuando se produzcan nuevas alertas, se enviará un correo electrónico a un grupo de personas designado, sin necesidad de tener que comprobar constantemente si en el portal hay actualizaciones.

Para más información, consulte [Configuración de notificaciones de Azure AD Domain Services](https://docs.microsoft.com/azure/active-directory-domain-services/active-directory-ds-notifications).

---

### <a name="azure-ad-portal-supports-using-the-forcedelete-domain-api-to-delete-custom-domains"></a>El portal de Azure AD admite el uso de la API de dominio ForceDelete para eliminar los dominios personalizados 

**Tipo:** Característica modificada  
**Categoría del servicio:** Administración de directorios  
**Funcionalidad del producto:** Directorio

Nos complace anunciar que ahora puede usar la API de dominio ForceDelete para eliminar los nombres de dominio personalizados cambiando las referencias de forma asincrónica, como los usuarios, los grupos y las aplicaciones del nombre de dominio personalizado (contoso.com) al nombre de dominio predeterminado inicial (contoso.onmicrosoft.com).

Este cambio ayuda a eliminar más rápidamente los nombres de dominio personalizados si la organización ya no utiliza el nombre o si necesita utilizar ese nombre de dominio con otra instancia de Azure AD.

Para más información, consulte [Eliminación de un nombre de dominio personalizado](https://docs.microsoft.com/azure/active-directory/users-groups-roles/domains-manage#delete-a-custom-domain-name).

---
