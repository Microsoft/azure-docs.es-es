---
title: Consideraciones sobre la plataforma de Windows universales (biblioteca de autenticación de Microsoft para. NET) | Azure
description: Obtenga información sobre las consideraciones específicas al usar la plataforma Universal de Windows con la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/24/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 83fb999b0cf66cfd8d96e82d23ed43626352a8aa
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544135"
---
# <a name="universal-windows-platform-specific-considerations-with-msalnet"></a>Consideraciones específicas de la plataforma Windows universales con MSAL.NET
En Xamarin iOS, hay varias consideraciones que debe tener en cuenta al usar MSAL.NET.

## <a name="the-usecorporatenetwork-property"></a>La propiedad UseCorporateNetwork
En la plataforma de WinRT, `PublicClientApplication` tiene la siguiente propiedad boolean ``UseCorporateNetwork``. Esta propiedad permite que las aplicaciones de Win8.1 y UWP para beneficiarse de la autenticación integrada de Windows (y por lo tanto, inicio de sesión único con el usuario con sesión iniciada con el sistema operativo) si el usuario se ha iniciado sesión con una cuenta de Azure federado inquilino de AD. Esto aprovecha WAB (agente de autenticación Web). 

> [!IMPORTANT]
> Si establece esta propiedad en true, se da por supuesto que el desarrollador de aplicaciones ha habilitado la autenticación integrada de Windows (IWA) en la aplicación. Para esto:
> - En el ``Package.appxmanifest`` para su aplicación para UWP, en el **capacidades** pestaña, habilite las siguientes funcionalidades:
>   - Autenticación empresarial
>   - Redes privadas (cliente y servidor)
>   - Certificado de usuario compartido

IWA no está habilitada de forma predeterminada porque las aplicaciones que solicitan la funcionalidad de autenticación empresarial o certificados de usuario compartido requieren un mayor nivel de comprobación a ser aceptados en el Store de Windows y no todos los desarrolladores que desee realizar mayor nivel de comprobación. 

La implementación subyacente en la plataforma UWP (WAB) no funciona correctamente en escenarios empresariales donde se ha habilitado el acceso condicional. El síntoma es que el usuario intenta iniciar sesión con hello de Windows y se propone para elegir un certificado, pero el certificado para el pin no se encuentra, o el usuario lo elige, pero nunca se le solicita por el Pin. Una solución alternativa es usar un método alternativo (nombre de usuario/contraseña + teléfono autenticación), pero la experiencia no es válida. 

## <a name="next-steps"></a>Pasos siguientes
En los ejemplos siguientes se proporcionan más detalles:

Muestra | Plataforma | DESCRIPCIÓN 
|------ | -------- | -----------|
|[active-directory-dotnet-native-uwp-v2](https://github.com/azure-samples/active-directory-dotnet-native-uwp-v2) | UWP | Una aplicación de cliente de plataforma Universal de Windows mediante msal.net, obtener acceso a Microsoft Graph para un usuario que se autentica con el punto de conexión de Azure AD v2.0. <br>![Topología](media/msal-net-uwp-considerations/topology-native-uwp.png)|
|[https://github.com/Azure-Samples/active-directory-xamarin-native-v2](https://github.com/Azure-Samples/active-directory-xamarin-native-v2) | Xamarin iOS, Android, UWP | Una aplicación de Xamarin Forms sencilla que muestra cómo usar MSAL para autenticar MSA y Azure AD a través del punto de conexión AAD v2.0 y tener acceso a Microsoft Graph con el token resultante. <br>![Topología](media/msal-net-uwp-considerations/topology-xamarin-native.png)|