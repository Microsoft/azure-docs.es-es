---
title: Consideraciones de UWP (MSAL.NET) | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre las consideraciones para usar la Plataforma universal de Windows (UWP) con la Biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/16/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: f716af1d50eb8a24d8ab70f784cba31f65795cb7
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88165913"
---
# <a name="considerations-for-using-universal-windows-platform-with-msalnet"></a>Consideraciones para usar la Plataforma universal de Windows con MSAL.NET
Los desarrolladores de aplicaciones que usan la Plataforma universal de Windows (UWP) con MSAL.NET deben tener en cuenta los conceptos que se presentan en este artículo.

## <a name="the-usecorporatenetwork-property"></a>La propiedad UseCorporateNetwork
En la plataforma de Windows Runtime (WinRT), `PublicClientApplication` tiene la propiedad booleana `UseCorporateNetwork`. Esta propiedad permite que las aplicaciones Windows 8.1 y las aplicaciones para UWP se beneficien de la autenticación integrada de Windows (IWA) si el usuario ha iniciado sesión en una cuenta que tenga un inquilino de Azure Active Directory (Azure AD) federado. Los usuarios que hayan iniciado sesión en el sistema operativo también pueden usar el inicio de sesión único (SSO). Al establecer la propiedad `UseCorporateNetwork`, MSAL.NET usa un agente de autenticación web (WAB).

> [!IMPORTANT]
> Si establece la propiedad `UseCorporateNetwork` en true, se da por supuesto que el desarrollador de aplicaciones ha habilitado IWA en la aplicación. Para habilitar IWA:
> - En `Package.appxmanifest` de la aplicación para UWP, en la pestaña **Funcionalidades**, habilite las siguientes funcionalidades:
>   - **Autenticación empresarial**
>   - **Redes privadas (cliente y servidor)**
>   - **Certificado de usuario compartido**

IWA no está habilitado de forma predeterminada porque Microsoft Store requiere un alto nivel de comprobación antes de aceptar aplicaciones que soliciten las funcionalidades de autenticación empresarial o certificados de usuario compartidos. No todos los desarrolladores desean llevar a cabo este nivel de comprobación.

En la plataforma UWP, la implementación de WAB subyacente no funciona correctamente en escenarios empresariales en los que esté habilitado el acceso condicional. Los usuarios ven los síntomas de este problema al intentar iniciar sesión con Windows Hello. Cuando se pide al usuario que elija un certificado:

- No se encuentra el certificado para el PIN.
- Después de que el usuario elija un certificado, no se le solicitará el PIN.

Puede intentar evitar este problema si utiliza un método alternativo como la autenticación con nombre de usuario-contraseña y teléfono, pero la experiencia no es buena.

## <a name="troubleshooting"></a>Solución de problemas

Algunos clientes han informado del siguiente error de inicio de sesión en entornos empresariales específicos en los que saben que tienen una conexión a Internet y que la conexión funciona con una red pública.

```Text
We can't connect to the service you need right now. Check your network connection or try this again later.
```

Puede evitar este problema si se asegura de que WAB (el componente subyacente de Windows) permite una red privada. Puede hacerlo estableciendo una clave del Registro:

```Text
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\authhost.exe\EnablePrivateNetwork = 00000001
```

Para más información, consulte [Agente de autenticación web: Fiddler](/windows/uwp/security/web-authentication-broker#fiddler).

## <a name="next-steps"></a>Pasos siguientes
En los siguientes ejemplos se proporciona más información.

Muestra | Plataforma | Descripción 
|------ | -------- | -----------|
|[active-directory-dotnet-native-uwp-v2](https://github.com/azure-samples/active-directory-dotnet-native-uwp-v2) | UWP | Una aplicación cliente para UWP que usa MSAL.NET. Obtiene acceso a Microsoft Graph para un usuario que se autentica mediante un punto de conexión de Azure AD 2.0. <br>![Topología](media/msal-net-uwp-considerations/topology-native-uwp.png)|
|[active-directory-xamarin-native-v2](https://github.com/Azure-Samples/active-directory-xamarin-native-v2) | Xamarin iOS, Android, UWP | Una aplicación sencilla de Xamarin Forms que muestra cómo usar MSAL para autenticar cuentas personales de Microsoft y Azure AD a través del punto de conexión de Azure AD 2.0. También muestra cómo acceder a Microsoft Graph, así como el token resultante. <br>![Topología](media/msal-net-uwp-considerations/topology-xamarin-native.png)|