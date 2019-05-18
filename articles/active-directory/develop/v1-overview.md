---
title: Azure Active Directory para información general de los desarrolladores (v1.0)
description: En este artículo se proporciona una introducción al inicio de sesión en las cuentas profesionales y educativas de Microsoft con el punto de conexión y la plataforma de Azure Active Directory v1.0.
services: active-directory
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: 5c872c89-ef04-4f4c-98de-bc0c7460c7c2
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 10/24/2018
ms.author: ryanwi
ms.reviewer: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 13cc5c7ae428f74f2892e6066dfdcd7efb73efbb
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545231"
---
# <a name="azure-active-directory-for-developers-v10-overview"></a>Azure Active Directory para información general de los desarrolladores (v1.0)

Azure Active Directory (Azure AD) es un servicio de identidad en la nube que permite a los desarrolladores compilar aplicaciones que inician la sesión de los usuarios en una cuenta profesional o educativa de Microsoft de forma segura. Azure AD admite tanto a desarrolladores que compilan aplicaciones de línea de negocio (LOB) de inquilino único como desarrolladores que desean desarrollar aplicaciones multiinquilino. Además del inicio de sesión básico, Azure AD también permite aplicaciones llamar tanto a API de Microsoft como [Microsoft Graph](https://docs.microsoft.com/graph/overview) como a API personalizadas que se basan en la plataforma de Azure AD. En esta documentación se muestra cómo agregar compatibilidad con Azure AD a una aplicación mediante protocolos estándar del sector, como OAuth2.0 y OpenID Connect.

> [!NOTE]
> La mayoría del contenido de esta página se centra en el punto de conexión y la plataforma v1.0, que solo admite cuentas profesionales o educativas de Microsoft. Si quiere iniciar sesión con cuentas personales o de consumidor de Microsoft, consulte la información sobre el [punto de conexión y la plataforma v2.0](v2-overview.md). El punto de conexión v2.0 ofrece una experiencia de desarrollo unificada para las aplicaciones que quieren iniciar sesión en todas las identidades de Microsoft.

| | |
| --- | --- |
|[Conceptos básicos sobre autenticación](authentication-scenarios.md) | Una introducción a la autenticación con Azure AD. |
|[Tipos de aplicaciones](app-types.md) | Una introducción a los escenarios de autenticación que se admiten en Azure AD. |
| | |

## <a name="get-started"></a>Introducción

Las guías de inicio y tutoriales siguientes sobre v1.0 le llevan por los pasos necesarios para compilar una aplicación en su plataforma preferida mediante el SDK de la Biblioteca de autenticación de Azure AD (ADAL). Para comenzar, consulte las **guías de inicio de v1.0** y los **tutoriales de v1.0** en [Plataforma de identidad de Microsoft (Azure Active Directory para desarrolladores)](index.yml).

## <a name="how-to-guides"></a>Guías de procedimientos

Consulte las **guías de procedimientos de v1.0** para encontrar información detallada y tutoriales sobre las tareas más comunes en Azure AD.

## <a name="reference-topics"></a>Temas de referencia

En los siguientes artículos se proporciona información detallada sobre las API, los mensajes de protocolo y los términos que se usan en Azure AD.

|                                                                                   | |
| ----------------------------------------------------------------------------------| --- |
| [Bibliotecas de autenticación (ADAL)](active-directory-authentication-libraries.md)   | Una introducción a las bibliotecas y los SDK que se proporcionan en Azure AD. |
| [Ejemplos de código](sample-v1-code.md)                                  | Una lista de todos los ejemplos de código de Azure AD. |
| [Glosario](developer-glossary.md)                                      | Terminología y definiciones de palabras que se usan en esta documentación. |
|  |  |


[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]
