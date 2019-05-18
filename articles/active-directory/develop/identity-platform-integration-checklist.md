---
title: Integración con la plataforma Microsoft identity | Azure
description: Obtenga información sobre las prácticas recomendadas y descuidos común cuando se integra con la plataforma Microsoft identity (v2.0).
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: ryanwi
ms.reviewer: lenalepa, sureshja, jesakowi
ms.custom: aaddev
ms.openlocfilehash: 0d44e5d9f0d1ed893a16c318e3cfa0ee1bfc123b
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65823345"
---
# <a name="microsoft-identity-platform-integration-checklist"></a>Lista de comprobación de integración de plataforma de Microsoft identity

La lista de comprobación de integración de plataforma de Microsoft identity está diseñado para guiarle a una integración segura y de alta calidad. Destaca los procedimientos recomendados y los descuidos común cuando se integra con la plataforma Microsoft identity así que revisión la lista de forma periódica para asegurarse de que mantener la calidad y la seguridad de la integración de la aplicación con la plataforma de identidad. La lista de comprobación no está diseñada para revisar toda la aplicación. El contenido de la lista de comprobación está sujetos a cambios a medida que introducimos mejoras en la plataforma.

Si acaba de empezar, consulte el [documentación](index.yml) para obtener información sobre los conceptos básicos sobre autenticación, escenarios de aplicaciones en la plataforma Microsoft identity y mucho más.

## <a name="testing-your-integration"></a>Probar la integración

Use la siguiente lista de comprobación para asegurarse de que la aplicación efectivamente está integrada con el [plataforma Microsoft identity](https://docs.microsoft.com/azure/active-directory/develop/).

### <a name="basics"></a>Aspectos básicos

|   |   |
|---|---|
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | Lea y comprenda el [directivas de la plataforma de Microsoft](https://go.microsoft.com/fwlink/?linkid=2090497&clcid=0x409). Asegúrese de que la aplicación se adhiere a los términos que se describen como están diseñados para proteger a usuarios y la plataforma. |

### <a name="ownership"></a>Propiedad

|   |   |
|---|---|
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | Asegúrese de que la información asociada con la cuenta que usó para registrar y administrar las aplicaciones está actualizada. |

### <a name="branding"></a>Personalización de marca

|   |   |
|---|---|
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | Respetar el [directrices para las aplicaciones de marca](howto-add-branding-in-azure-ad-apps.md). |
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | Proporcione un nombre significativo y logotipo para su aplicación. Esta información aparece en la petición de consentimiento de la aplicación. Asegúrese de que su nombre y el logotipo son representativos de su producto de empresa para que los usuarios puedan tomar decisiones informadas. Asegúrese de que no están infringiendo las marcas comerciales. |

### <a name="privacy"></a>Privacidad

|   |   |
|---|---|
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | Proporciona vínculos a los términos de la aplicación de servicio y declaración de privacidad. |

### <a name="security"></a>Seguridad

|   |   |
|---|---|
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | Conservar la propiedad de todos los URI de redireccionamiento y mantenerse los registros DNS para ellos. No use caracteres comodín (*) en el URI. Para las aplicaciones web, asegúrese de que todos los URI son seguras y cifradas (por ejemplo, mediante esquemas de https). Para los clientes públicos, usar los URI de redireccionamiento específicos de la plataforma si es aplicable (principalmente para iOS y Android). En caso contrario, use URI de redireccionamiento con una gran cantidad de aleatoriedad para evitar conflictos al devolver la llamada a la aplicación. Si se usa la aplicación de un agente web aislado, puede usar https://login.microsoftonline.com/nativeclient. Revise y recortar todas innecesarios o sin usar los URI de redireccionamiento de forma periódica. |
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | Si la aplicación se registra en un directorio, minimizar y supervisar manualmente la lista de propietarios de registro de aplicación. |
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | No habilitar la compatibilidad con la [flujo de concesión implícita de OAuth2](v2-oauth2-implicit-grant-flow.md) a menos que requieren de forma explícita. Obtenga información sobre el escenario válido [aquí](v1-oauth2-implicit-grant-flow.md#suitable-scenarios-for-the-oauth2-implicit-grant). |
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | No use [flujo de credenciales de contraseña de propietario de recursos (ROPC)](v2-oauth-ropc.md), que controla directamente las contraseñas de usuario. Este flujo requiere un alto grado de exposición de confianza y de usuario y solo debe usarse cuando no se puede usar flujos de otros y la más seguros. |
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | Proteger y administrar las credenciales de la aplicación. Use [credenciales de certificado](active-directory-certificate-credentials.md), no las credenciales de contraseña (los secretos de cliente). Si debe usar una credencial de contraseña, no establecerlo manualmente. No almacenar las credenciales en el código o configuración y no que nunca puedan controlarse mediante los seres humanos. Si es posible, use [administra las identidades de los recursos de Azure](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview) o [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-whatis) para almacenar y rotación periódica de sus credenciales. |
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | Asegúrese de que la aplicación solicita permisos de privilegios mínimos. Pedir solo los permisos que absolutamente necesita su aplicación y sólo cuando las necesite. Descripción de los diversos [tipos de permisos](v1-permissions-and-consent.md#types-of-permissions). Usar solo los permisos de la aplicación si es necesario; Usar permisos delegados que sea posible. Para obtener una lista completa de permisos de Microsoft Graph, vea este [referencia de permisos](https://docs.microsoft.com/graph/permissions-reference). |
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | Si va a proteger una API mediante la plataforma Microsoft identity, considerar cuidadosamente los permisos que debe exponer. Considere cuál es la granularidad adecuada para su solución y qué permisos requieren el consentimiento del administrador. Comprobar los permisos esperados en los tokens entrantes antes de tomar las decisiones de autorización. |

### <a name="implementation"></a>Implementación

|   |   |
|---|---|
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | Usar soluciones de autenticación moderna (OAuth 2.0, [OpenID Connect](v2-protocols-oidc.md)) para firmar de forma segura en los usuarios. |
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | No implementar los protocolos usted mismo, utilice [bibliotecas de autenticación compatible con Microsoft](reference-v2-libraries.md) (MSAL, middleware de servidor). Asegúrese de que está usando la versión más reciente de la biblioteca de autenticación que haya integrado con. |
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | Si están disponibles a través de los datos de la aplicación requiere [Microsoft Graph](https://developer.microsoft.com/graph), solicitar permisos para estos datos mediante el punto de conexión de Microsoft Graph en lugar de la API individual. |

### <a name="end-user-experience"></a>Experiencia del usuario final

|   |   |
|---|---|
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | [Comprender la experiencia de consentimiento](application-consent-experience.md) y configure las piezas de petición de consentimiento de la aplicación para que los usuarios finales y administradores de tengan suficiente información para determinar si confían en la aplicación. |
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | Minimizar el número de veces que un usuario necesita que escriba las credenciales de inicio de sesión mientras se usa la aplicación al intentar silenciosa autenticación (adquisición del token silenciosa) antes de flujos interactivos. |
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | No use "prompt = consentimiento" para cada inicio de sesión. Usar sólo el símbolo del sistema = consent si ha determinado que debe pedir consentimiento para permisos adicionales (por ejemplo, si ha cambiado los permisos necesarios de la aplicación). |
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | Si procede, enriquecer su aplicación con datos de usuario. Use la [Microsoft Graph API](https://developer.microsoft.com/graph) es una manera fácil de hacerlo. El [Probador](https://developer.microsoft.com/graph/graph-explorer) herramienta que le ayudarán a empezar. |
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | Registrar el conjunto completo de permisos que requiere la aplicación, por lo que los administradores pueden conceder consentimiento fácilmente con su inquilino. Use [consentimiento incremental](azure-ad-endpoint-comparison.md#incremental-and-dynamic-consent) en tiempo de ejecución para ayudar a los usuarios a entender por qué la aplicación solicita permisos que es posible que se refieren a o confundir a los usuarios cuando se solicita en el primer inicio. |
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | Implemente un [limpiar una experiencia de cierre de sesión único](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-6-SignOut). Es el requisito de seguridad y privacidad y hace que para una buena experiencia del usuario. |

### <a name="testing"></a>Pruebas

|   |   |
|---|---|
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | Probar [directivas de acceso condicional](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-6-SignOut) que pueden afectar a la capacidad del usuario para usar la aplicación. |
| ![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) | Probar la aplicación con todas las posibles cuentas que va a admitir (por ejemplo, trabajo o cuentas educativas, cuentas Microsoft personales, cuentas de menores y cuentas independientes). |

## <a name="additional-resources"></a>Recursos adicionales

Explore la información detallada acerca de la versión 2.0:

* [Plataforma de identidad de Microsoft (información general sobre v2.0)](v2-overview.md)
* [Referencia de protocolos de plataforma de identidad de Microsoft](active-directory-v2-protocols.md)
* [Referencia de los tokens de acceso](access-tokens.md)
* [Referencia de los tokens de identificador](id-tokens.md)
* [Referencia de bibliotecas de autenticación](reference-v2-libraries.md)
* [Permisos y consentimiento en la plataforma Microsoft identity](v2-permissions-and-consent.md)
* [Microsoft Graph API](https://developer.microsoft.com/graph)
