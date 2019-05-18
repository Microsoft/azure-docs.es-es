---
title: 'Aplicación móvil que llama a las API web (pasar a producción): la plataforma de identidad de Microsoft'
description: Aprenda a crear una aplicación móvil que llama a web API (pasar a producción)
services: active-directory
documentationcenter: dev-center-name
author: danieldobalian
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: dadobali
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5cc8b7c86495c2a60b07bef0a825e3872f787520
ms.sourcegitcommit: f013c433b18de2788bf09b98926c7136b15d36f1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65550401"
---
# <a name="mobile-app-that-calls-web-apis---move-to-production"></a>Aplicación móvil que llama a web API: pasar a producción

Este artículo proporciona detalles sobre cómo mejorar la calidad y la confiabilidad de la aplicación antes de moverla a producción.

## <a name="handling-errors-in-mobile-applications"></a>Control de errores en aplicaciones móviles

Una serie de condiciones de error puede producirse en la aplicación en este momento. Los escenarios principales para controlar son errores silenciosos y reservas de interacción. Otras condiciones que se deben considerar para la producción incluyen situaciones sin red, interrupciones de servicio, los requisitos de consentimiento del administrador y otros casos específicos del escenario.

Cada biblioteca MSAL tiene contenido wiki y de código de ejemplo que se describe cómo administrar estas condiciones:

- [MSAL Android Wiki](https://github.com/AzureAD/microsoft-authentication-library-for-android)
- [MSAL iOS Wiki](https://github.com/AzureAD/microsoft-authentication-library-for-objc/wiki)
- [MSAL.NET Wiki](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki)

## <a name="mitigating-and-investigating-issues"></a>Mitigar e investigar los problemas

Para diagnosticar problemas en su aplicación, lo ayuda a recopilar datos. Para obtener información sobre los tipos de datos puede recopilar, ver los wikis de plataforma MSAL.

- Podrían pedir ayuda a los usuarios cuando tienen problemas. Es una práctica recomendada capturar y temporalmente almacenar los registros y proporcione una ubicación donde los usuarios pueden cargarlos. MSAL proporciona extensiones de registro para capturar información detallada acerca de la autenticación.
- Si está disponible, habilitar la telemetría a través de MSAL para recopilar datos acerca de cómo los usuarios inician sesión en la aplicación.

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [Move to production common steps](../../../includes/active-directory-develop-scenarios-production.md)]
