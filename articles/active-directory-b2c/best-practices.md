---
title: Procedimientos recomendados para Azure AD B2C
titleSuffix: Azure AD B2C
description: Recomendaciones y procedimientos recomendados que se deben tener en cuenta al trabajar con Azure Active Directory B2C (Azure AD B2C).
services: active-directory-b2c
author: vigunase
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 06/06/2020
ms.author: vigunase
ms.subservice: B2C
ms.openlocfilehash: c8739da859c00a9caf08ac833f7b4ae7ae52e392
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90084318"
---
# <a name="recommendations-and-best-practices-for-azure-active-directory-b2c"></a>Recomendaciones y procedimientos recomendados para Azure Active Directory B2C

Los siguientes procedimientos recomendados y recomendaciones cubren algunos de los aspectos principales de la integración de Azure Active Directory (Azure AD) B2C en entornos de aplicación nuevos o existentes.

## <a name="fundamentals"></a>Aspectos básicos

| Procedimiento recomendado | Descripción |
|--|--|
| Elección de flujos de usuario para la mayoría de los escenarios | Identity Experience Framework de Azure AD B2C es el punto fuerte del servicio. Las directivas describen totalmente las experiencias de identidad tales como el registro, el inicio de sesión y la edición de perfil. Para ayudarle a configurar las tareas más comunes de identidad, el portal de Azure AD B2C incluye directivas predefinidas y configurables denominadas flujos de usuario. Con los flujos de usuario, puede crear experiencias de usuario excelentes en cuestión de minutos, con unos pocos clics. [Obtenga información sobre cuándo usar flujos de usuario frente a directivas personalizadas](custom-policy-overview.md#comparing-user-flows-and-custom-policies).|
| Registros de aplicaciones | Las aplicaciones (web, nativa) y API que se protegen deben registrarse en Azure AD B2C. Si una aplicación tiene tanto una versión web como una nativa de iOS y Android, puede registrarse como una aplicación en Azure AD B2C con el mismo id. de cliente. Obtenga información sobre cómo [registrar aplicaciones OIDC, SAML, web y nativas](https://docs.microsoft.com/azure/active-directory-b2c/tutorial-register-applications?tabs=applications). Obtenga más información sobre los [tipos de aplicaciones que se pueden usar en Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/application-types). |
| Traslado a la facturación de usuarios activos mensuales | Azure AD B2C ha pasado de las autenticaciones activas mensuales a la facturación de usuarios activos mensuales (MAU). La mayoría de los clientes encontrarán este modelo rentable. [Obtenga más información sobre la facturación de usuarios activos mensuales](https://azure.microsoft.com/updates/mau-billing/). |

## <a name="planning-and-design"></a>Planificación y diseño

Defina la arquitectura de aplicaciones y servicios, los sistemas actuales de inventario y planee la migración a Azure AD B2C.

| Procedimiento recomendado | Descripción |
|--|--|
| Diseño de una solución de un extremo a otro | Incluya todas las dependencias de las aplicaciones al planear una integración de Azure AD B2C. Tenga en cuenta todos los servicios y productos que se encuentran actualmente en su entorno o que podrían tener que agregarse a la solución, por ejemplo, Azure Functions, los sistemas de administración de relaciones con clientes (CRM), Azure API Management Gateway y los servicios de almacenamiento. Tenga en cuenta la seguridad y la escalabilidad de todos los servicios. |
| Documentación de las experiencias de los usuarios | Detalle todos los recorridos del usuario que los clientes pueden experimentar en la aplicación. Incluya todas las pantallas y los flujos de bifurcación que puedan encontrarse al interactuar con los aspectos de la identidad y el perfil de la aplicación. Incluya facilidad de uso, accesibilidad y localización en el planeamiento. |
| Elección del protocolo de autenticación adecuado |  Para ver un desglose de los distintos escenarios de aplicación y sus flujos de autenticación recomendados, consulte [Escenarios y flujos de autenticación compatibles](../active-directory/develop/authentication-flows-app-scenarios.md#scenarios-and-supported-authentication-flows). |
| Piloto de una experiencia del usuario de un extremo a otro de prueba de concepto (POC) | Comience con los [ejemplos de código de Microsoft](code-samples.md) y [ejemplos de la comunidad](https://github.com/azure-ad-b2c/samples). |
| Creación de un plan de migración |La planificación previa puede hacer que la migración se realice de forma más fluida. Obtenga más información acerca de la [migración de usuarios](user-migration.md).|
| Facilidad de uso frente a seguridad | La solución debe lograr el equilibrio adecuado entre la facilidad de uso de la aplicación y el nivel de riesgo aceptable de la organización. |
| Traslado de dependencias locales a la nube | Para ayudar a garantizar una solución resistente, considere la posibilidad de mover las dependencias de aplicaciones existentes a la nube. |
| Migración de aplicaciones existentes a b2clogin.com | La retirada de login.microsoftonline.com se aplicará para todos los inquilinos de Azure AD B2C del 4 de diciembre de 2020. [Más información](b2clogin.md). |
| Uso de Identity Protection y acceso condicional | Use estas funcionalidades para un mayor control sobre las autenticaciones de riesgo y las directivas de acceso. Se requiere Azure AD B2C P2. [Más información](conditional-access-identity-protection-overview.md). |

## <a name="implementation"></a>Implementación

Durante la fase de implementación, tenga en cuenta las siguientes recomendaciones.

| Procedimiento recomendado | Descripción |
|--|--|
| Edición de las directivas personalizadas con la extensión de Azure AD B2C para Visual Studio Code | Descargue Visual Studio Code y esta [extensión creada por la comunidad desde Visual Studio Code Marketplace](https://marketplace.visualstudio.com/items?itemName=AzureADB2CTools.aadb2c). Aunque no es un producto oficial de Microsoft, la extensión de Azure AD B2C para Visual Studio Code incluye varias características que ayudan a facilitar el trabajo con directivas personalizadas. |
| Más información acerca de cómo solucionar problemas de Azure AD B2C | Obtenga información acerca de cómo [solucionar problemas de las directivas personalizadas](https://docs.microsoft.com/azure/active-directory-b2c/troubleshoot-custom-policies?tabs=applications) durante el desarrollo. Obtenga información sobre el aspecto de un flujo de autenticación normal y use herramientas para detectar anomalías y errores. Por ejemplo, use [Application Insights](troubleshoot-with-application-insights.md) para revisar los registros de salida de los recorridos del usuario. |
| Aprovechamiento de nuestra biblioteca de patrones de directivas personalizadas probadas | Busque [ejemplos](https://github.com/azure-ad-b2c/samples) de varios recorridos del usuario de administración de identidades y acceso de clientes (CIAM) mejorados de Azure AD B2C. |

## <a name="testing"></a>Prueba

Pruebe y automatice la implementación de Azure AD B2C.

| Procedimiento recomendado | Descripción |
|--|--|
| Cuenta para el tráfico global | Use orígenes de tráfico de una dirección global diferente para probar los requisitos de rendimiento y localización. Asegúrese de que todos los HTML, CSS y las dependencias pueden satisfacer sus necesidades de rendimiento. |
| Pruebas funcionales y de interfaz de usuario | Pruebe los flujos de usuario de un extremo a otro. Agregue pruebas sintéticas cada pocos minutos mediante Selenium, VS web Test, etc. |
| Prueba de penetración | Antes de empezar a trabajar con la solución, realice los ejercicios de pruebas de penetración para comprobar que todos los componentes son seguros, incluidas las dependencias de terceros. Compruebe que ha protegido las API con tokens de acceso y que ha usado el protocolo de autenticación adecuado para el escenario de la aplicación. Obtenga más información sobre las [pruebas de penetración](https://docs.microsoft.com/azure/security/fundamentals/pen-testing) y las [reglas de interacción de pruebas de penetración unificadas de Microsoft Cloud](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1). |
| Pruebas A/B | Pruebe las nuevas características con un pequeño conjunto aleatorio de usuarios antes de implementarlo en todo el rellenado. Con JavaScript habilitado en Azure AD B2C, puede integrarse con herramientas de prueba A/B, como Optimizely, Clarity y otras. |
| Pruebas de carga | Azure AD B2C se puede escalar, pero la aplicación solo se puede escalar si todas sus dependencias se pueden escalar. Pruebe la carga de las API y CDN. |
| Limitaciones |  Azure AD B2C limita el tráfico si se envían demasiadas solicitudes desde el mismo origen en un breve período de tiempo. Use varios orígenes de tráfico durante las pruebas de carga y controle el código de error `AADB2C90229` correctamente en las aplicaciones. |
| Automation | Use canalizaciones de entrega e integración continuas (CI/CD) para automatizar las pruebas y las implementaciones, por ejemplo, [Azure DevOps](deploy-custom-policies-devops.md). |

## <a name="operations"></a>Operaciones

Administre el entorno de Azure AD B2C.

| Procedimiento recomendado | Descripción |
|--|--|
| Creación de múltiples entornos | Para facilitar la implementación de las operaciones e implementaciones, cree entornos independientes para desarrollo, pruebas, preproducción y producción. Cree inquilinos de Azure AD B2C para cada uno. |
| Uso del control de versiones para las directivas personalizadas | Considere la posibilidad de usar GitHub, Azure Repos u otro sistema de control de versiones basado en la nube para las directivas personalizadas de Azure AD B2C. |
| Uso de Microsoft Graph API para automatizar la administración de los inquilinos de B2C | Microsoft Graph API:<br/>Administre [Identity Experience Framework](https://docs.microsoft.com/graph/api/resources/trustframeworkpolicy?view=graph-rest-beta&preserve-view=true) (directivas personalizadas).<br/>[Claves](https://docs.microsoft.com/graph/api/resources/trustframeworkkeyset?view=graph-rest-beta&preserve-view=true)<br/>[Flujos de usuario](https://docs.microsoft.com/graph/api/resources/identityuserflow?view=graph-rest-beta&preserve-view=true) |
| Integración con Azure DevOps | Una canalización de [CI/CD](deploy-custom-policies-devops.md) facilita el traslado de código entre distintos entornos y garantiza la preparación de la producción en todo momento.   |
| Integración con Azure Monitor | Los [eventos de registro de auditoría](view-audit-logs.md) solo se conservan durante siete días. Realice la [integración con Azure Monitor](azure-monitor.md) para conservar los registros para su uso a largo plazo o integrarlos en herramientas de Administración de eventos e información de seguridad (SIEM) de terceros para sacar conclusiones sobre su entorno. |
| Configuración de supervisión y alertas activas | [Siga el comportamiento del usuario](active-directory-b2c-custom-guide-eventlogger-appins.md) en Azure AD B2C mediante Application Insights. |

## <a name="support-and-status-updates"></a>Actualizaciones de compatibilidad y estado

Manténgase al día con el estado del servicio y busque opciones de soporte técnico.

| Procedimiento recomendado | Descripción |
|--|--|
| [Actualizaciones del servicio](https://azure.microsoft.com/updates/?product=active-directory-b2c) |  Manténgase al día con los anuncios y las actualizaciones del producto de Azure AD B2C. |
| [Ayuda y soporte técnico de Microsoft](support-options.md) | Envíe una solicitud de soporte técnico para problemas técnicos de Azure AD B2C. Se ofrece de forma gratuita soporte técnico para la administración de suscripciones y la facturación. |
| [Estado de Azure](https://status.azure.com/status) | Vea el estado de mantenimiento actual de todos los servicios de Azure. |
