---
title: Consideraciones sobre el explorador del sistema de Xamarin Android con (MSAL.NET) | Azure
titleSuffix: Microsoft identity platform
description: Conozca las consideraciones para usar los exploradores del sistema en Xamarin Android con la Biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 10/30/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 8f755b42249a88a2d37117003d561f79aea6b170
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88165998"
---
#  <a name="xamarin-android-system-browser-considerations-for-using-msalnet"></a>Consideraciones del explorador del sistema de Xamarin Android para usar MSAL.NET

En este artículo se describe lo que debe tener en cuenta al usar el explorador del sistema en Xamarin Android con la Biblioteca de autenticación de Microsoft para .NET (MSAL.NET).

A partir de la versión preliminar de MSAL.NET 2.4.0, MSAL.NET admite exploradores distintos de Chrome. Ya no es necesario que Chrome esté instalado en el dispositivo Android para la autenticación.

Se recomienda usar exploradores que admitan pestañas personalizadas. Aquí se proporcionan algunos ejemplos de estos exploradores:

| Exploradores que admiten pestañas personalizadas | Nombre del paquete |
|------| ------- |
|Chrome | com.android.chrome|
|Microsoft Edge | com.microsoft.emmx|
|Firefox | org.mozilla.firefox|
|Ecosia | com.ecosia.android|
|Kiwi | com.kiwibrowser.browser|
|Brave | com.brave.browser|

Además de identificar los exploradores que admiten pestañas personalizadas, nuestras pruebas revelan que algunos exploradores que no las admiten también funcionan para la autenticación. Estos exploradores incluyen Opera, Opera Mini, InBrowser y Maxthon. 

## <a name="tested-devices-and-browsers"></a>Dispositivos y exploradores probados
En la tabla siguiente se enumeran los dispositivos y exploradores que se han probado para determinar si son compatibles con la autenticación.

| Dispositivo | Browser     |  Resultado  | 
| ------------- |:-------------:|:-----:|
| Huawei/One+ | Chrome\* | Pass (pasado)|
| Huawei/One+ | Microsoft Edge\* | Pass (pasado)|
| Huawei/One+ | Firefox\* | Pass (pasado)|
| Huawei/One+ | Brave\* | Pass (pasado)|
| One+ | Ecosia\* | Pass (pasado)|
| One+ | Kiwi\* | Pass (pasado)|
| Huawei/One+ | Opera | Pass (pasado)|
| Huawei | OperaMini | Pass (pasado)|
| Huawei/One+ | InBrowser | Pass (pasado)|
| One+ | Maxthon | Pass (pasado)|
| Huawei/One+ | DuckDuckGo | Autenticación cancelada por el usuario|
| Huawei/One+ | Explorador UC | Autenticación cancelada por el usuario|
| One+ | Dolphin | Autenticación cancelada por el usuario|
| One+ | Explorador CM | Autenticación cancelada por el usuario|
| Huawei/One+ | Ninguno instalado | Excepción AndroidActivityNotFound|

\* admite las pestañas personalizadas

## <a name="known-issues"></a>Problemas conocidos

Si el usuario no tiene ningún explorador habilitado en el dispositivo, MSAL.NET iniciará una excepción `AndroidActivityNotFound`.  
  - **Mitigación**: pida al usuario que habilite un explorador en su dispositivo. Recomiende un explorador que admita pestañas personalizadas.

Si se produce un error de autenticación (por ejemplo, si la autenticación se inicia con DuckDuckGo), MSAL.NET devolverá `AuthenticationCanceled MsalClientException`. 
  - **Problema raíz**: no se ha habilitado un explorador que admite pestañas personalizadas en el dispositivo. La autenticación se inició con un explorador que no pudo completar la autenticación. 
  - **Mitigación**: pida al usuario que habilite un explorador en su dispositivo. Recomiende un explorador que admita pestañas personalizadas.

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información y ejemplos de código, consulte [Elección entre un explorador web insertado y un explorador del sistema en Xamarin Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/MSAL.NET-uses-web-browser#choosing-between-embedded-web-browser-or-system-browser-on-xamarinandroid) e [Interfaz de usuario web integrada frente el sistema](msal-net-web-browsers.md#embedded-vs-system-web-ui).  