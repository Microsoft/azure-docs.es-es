---
title: Escenarios comunes de administración de aplicaciones para Azure Active Directory | Microsoft Docs
description: Centralización de la administración de aplicaciones con Azure AD
documentationcenter: ''
author: kenwith
manager: celestedg
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 03/02/2019
ms.author: kenwith
ms.reviewer: arvinh
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8ce3819ff1f9b0c61f7738f90ff17c2798fe888b
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88642119"
---
# <a name="centralize-application-management-with-azure-ad"></a>Centralización de la administración de aplicaciones con Azure AD

Contraseñas, tanto una pesadilla para TI como una molestia para los empleados de todo el mundo. Esta es el motivo por el que cada vez más y más empresas se están pasando a Azure Active Directory, la solución de administración de identidad y acceso de Microsoft para la nube y todos sus otros recursos. Pase de una aplicación a otra sin tener que escribir una contraseña para cada una. Pase de Outlook a WorkDay a ADP tan rápido como pueda abrirlas, de forma rápida y segura. Después, colabore con partners e incluso con otras personas fuera de su organización, todo sin tener que llamar a TI. Lo que es más, Azure AD ayuda a administrar los riesgos al proteger las aplicaciones que usa, con elementos como autenticación multifactor, para verificar quién es; al usar el aprendizaje automático y la inteligencia de seguridad adaptables de forma continua para detectar inicios de sesión sospechosos, lo que le proporciona un acceso seguro a las aplicaciones que necesita, dondequiera que esté. No solo es excelente para los usuarios, sino también para TI. Con las revisiones de acceso Just-in-Time y un conjunto de gobernanza de escala completa, Azure AD le ayuda a mantenerse en cumplimiento, así como a aplicar directivas. Y entienda lo siguiente, incluso puede automatizar el aprovisionamiento de las cuentas de usuario, lo que facilita la administración del acceso. Consulte algunos de los escenarios comunes para los que los clientes usan las funcionalidades de administración de aplicaciones de Azure Active Directory.

**Escenarios comunes**


> [!div class="checklist"]
> * SSO para todas las aplicaciones
> * Automatización de aprovisionamiento y desaprovisionamiento 
> * Protección de las aplicaciones
> * Control del acceso a las aplicaciones
> * Acceso híbrido seguro

## <a name="scenario-1-set-up-sso-for-all-your-applications"></a>Escenario 1: Configuración de SSO para todas las aplicaciones

Ya no más administración de contraseñas. Obtenga acceso de forma segura a todos los recursos que necesita con sus credenciales corporativas. 

|Característica  | Descripción | Recomendación |
|---------|---------|---------|
|SSO|SSO federado basado en estándares con estándares de confianza del sector.|Use siempre [SAML/OIDC](https://docs.microsoft.com/azure/active-directory/manage-apps/isv-choose-multi-tenant-federation) para habilitar SSO cuando la aplicación lo admita.|
|Mis aplicaciones|Proporcione a sus usuarios un lugar centralizado y sencillo desde el que puedan acceder a todas las aplicaciones. Favorezca la productividad con funcionalidades de autoservicio, como la solicitud de acceso a aplicaciones y grupos, o la administración del acceso a recursos en nombre de otros usuarios.| Implemente [Mis aplicaciones](https://docs.microsoft.com/azure/active-directory/manage-apps/access-panel-deployment-plan) en la organización una vez que haya integrado las aplicaciones con Azure AD para el inicio de sesión único.|

## <a name="scenario-2-automate-provisioning-and-deprovisioning"></a>Escenario 2: Automatización de aprovisionamiento y desaprovisionamiento 


La mayoría de las aplicaciones requiere el aprovisionamiento de un usuario en la aplicación antes de tener acceso a los recursos que necesita. El uso de archivos CSV o scripts complejos puede ser costoso y difícil de administrar. Además, los clientes deben asegurarse de que las cuentas se quiten cuando alguien ya no deba tener acceso. Aproveche las herramientas siguientes para automatizar el aprovisionamiento y el desaprovisionamiento. 


|Característica  |Descripción|Recomendación |
|---------|---------|---------|
|Aprovisionamiento de SCIM|[SCIM](https://aka.ms/SCIMOverview) es un procedimiento recomendado del sector para la automatización del aprovisionamiento de usuarios. Cualquier aplicación compatible con SCIM puede integrarse en Azure AD. Cree, actualice y elimine automáticamente cuentas de usuario sin tener que mantener archivos CSV, scripts personalizados ni soluciones locales.|Consulte la lista creciente de aplicaciones [integradas previamente](https://docs.microsoft.com/azure/active-directory/saas-apps/tutorial-list) en la galería de aplicaciones de Azure AD.|
|Microsoft Graph|Aproveche la vastedad de datos con que cuenta Azure AD para enriquecer su aplicación con los datos que necesita.|Aproveche [Microsoft Graph](https://developer.microsoft.com/graph/) para obtener datos de todo el ecosistema de Microsoft. |


## <a name="scenario-3-secure-your-applications"></a>Escenario 3: Protección de las aplicaciones
La identidad es el eje de la seguridad. Si una identidad se ve en peligro, es increíblemente difícil detener el efecto dominó antes de que sea demasiado tarde. El promedio, pasan más de 100 días antes de que las organizaciones detecten que se ha producido una vulneración. Utilice las herramientas que ofrece Azure AD para mejorar la posición de seguridad de sus aplicaciones. 

|Característica  |Descripción| Recomendación |
|---------|---------| ---------|
|Azure MFA|Azure Multi-Factor Authentication (MFA) es la solución de Microsoft de comprobación de dos pasos. El uso de métodos de autenticación aprobados por un administrador permite a Azure MFA ayudar a proteger el acceso a sus datos y aplicaciones, además de satisfacer la exigencia de un proceso de inicio de sesión simple.| [Habilite MFA](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/all-your-creds-are-belong-to-us/ba-p/855124) para los usuarios.  |
|Acceso condicional|Con Acceso condicional, puede implementar decisiones de control de acceso automatizado sobre quién puede acceder a las aplicaciones en nube, en función de condiciones.| Revise los [valores predeterminados de seguridad](https://docs.microsoft.com/azure/active-directory/fundamentals/concept-fundamentals-security-defaults) y las [directivas comunes](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-conditional-access-policy-common) que utilizan los clientes. | 
|Protección de identidad|Identity Protection usa los aprendizajes que Microsoft ha adquirido de su puesto en organizaciones con Azure AD, el espacio de consumidor con cuentas de Microsoft y juegos con Xbox para proteger a los usuarios. Microsoft analiza 6,5 billones de señales al día para identificar y proteger a los clientes frente a amenazas.|Habilite las [directivas de protección de identidad predeterminadas](https://docs.microsoft.com/azure/active-directory/identity-protection/concept-identity-protection-policies) que proporciona el servicio. | 

## <a name="scenario-4-govern-access-to-your-applications"></a>Escenario 4: Control del acceso a las aplicaciones
La gobernanza de identidades ayuda a las organizaciones a alcanzar un equilibrio entre productividad (con qué rapidez puede obtener acceso una persona a las aplicaciones que necesita; por ejemplo, cuando se une a una organización) y seguridad (cómo debe cambiar el acceso de esa persona a lo largo del tiempo; por ejemplo, cuando varía su estado laboral). 

|Característica  |Descripción|Recomendación |
|---------|---------| ---------|
|ELM|La administración de derechos de Azure AD puede ayudar a los usuarios dentro y fuera de su organización a administrar de forma más eficaz el acceso a las aplicaciones.| Permita que los usuarios que no son administradores administren el acceso a sus aplicaciones mediante [paquetes de acceso](https://docs.microsoft.com/azure/active-directory/governance/entitlement-management-access-package-first).|
|Revisiones de acceso|El acceso de los usuarios a las aplicaciones se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado.| [Revise el acceso](https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview) a las aplicaciones más confidenciales. |
|Log Analytics|Genere informes sobre quién tiene acceso a qué aplicaciones y almacene los datos en su herramienta de SIEM de su elección para correlacionar los datos entre los orígenes de datos y a lo largo del tiempo.| Habilite el [análisis de registros](https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-analyze-activity-logs-log-analytics) y configure alertas para eventos críticos relacionados con las aplicaciones. |


## <a name="scenario-5-hybrid-secure-access"></a>Escenario 5: Acceso híbrido seguro
La identidad solo puede ser su plano de control si puede conectar todo entre las aplicaciones locales y en la nube. Aproveche las herramientas que ofrecen Azure AD y sus partners para proteger el acceso a las aplicaciones basadas en la autenticación heredada.

|Característica  |Descripción|Recomendación |
|---------|---------|---------|
|Proxy de aplicación|Hoy en día, los empleados desean ser productivos en cualquier lugar, en cualquier momento y con cualquier dispositivo. Tienen que acceder a aplicaciones SaaS en la nube y a aplicaciones corporativas locales. El proxy de aplicación de Azure AD permite este acceso sólido sin redes privadas virtuales (VPN) costosas y complejas ni zonas desmilitarizadas (DMZ).|Configure el [acceso remoto](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy) para las aplicaciones locales. |
|F5, Akamai, Zscaler|Con la red y el controlador de entrega existentes, puede proteger fácilmente las aplicaciones heredadas que siguen siendo críticas para los procesos empresariales pero que no se pudieron proteger antes de Azure AD. Es probable que ya tenga todo lo que necesita para empezar a proteger estas aplicaciones.| ¿Usa Akamai, Citrix, F5 o Zscaler? Consulte nuestras [soluciones pregeneradas](https://docs.microsoft.com/azure/active-directory/manage-apps/secure-hybrid-access). | 

## <a name="related-articles"></a>Artículos relacionados

- [Administración de aplicaciones](https://docs.microsoft.com/azure/active-directory/manage-apps/index)
- [Aprovisionamiento de aplicaciones](https://docs.microsoft.com/azure/active-directory/app-provisioning/user-provisioning)
- [Acceso híbrido seguro](https://docs.microsoft.com/azure/active-directory/manage-apps/secure-hybrid-access)
- [Gobernanza de identidades](https://docs.microsoft.com/azure/active-directory/governance/identity-governance-overview)
- [Plataforma de identidad de Microsoft](https://docs.microsoft.com/azure/active-directory/develop/v2-overview)
- [Seguridad de identidades](https://docs.microsoft.com/azure/active-directory/conditional-access/index)
