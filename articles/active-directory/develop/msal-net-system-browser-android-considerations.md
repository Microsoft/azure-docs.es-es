---
title: Consideraciones de Xamarin Android (biblioteca de autenticación de Microsoft para. NET) | Azure
description: Obtenga información sobre las consideraciones específicas al usar Xamarin Android con la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
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
ms.openlocfilehash: c57feb33967732481d78e0ddaba5e90f4f82f327
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544440"
---
# <a name="xamarin-android-specific-considerations-with-msalnet"></a>Consideraciones específicas de Android de Xamarin con MSAL.NET
En este artículo se describe algunas consideraciones al usar el explorador del sistema de Xamarin Android con la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).

A partir de MSAL.NET 2.4.0-preview, MSAL.NET es compatible con los exploradores que no sean de Chrome y ya no requiere Chrome instalarse en el dispositivo Android para la autenticación.

Se recomienda que usar los exploradores que admiten las pestañas personalizadas, como los siguientes:

| Exploradores con compatibilidad con las pestañas personalizadas | Nombre del paquete |
|------| ------- |
|Chrome | com.android.chrome|
|Microsoft Edge | com.microsoft.emmx|
|Firefox | org.mozilla.firefox|
|Ecosia | com.ecosia.android|
|Kiwi | com.kiwibrowser.browser|
|Audaces | com.brave.browser|

Además de los exploradores con compatibilidad de las fichas personalizadas, basándose en nuestras pruebas, algunos exploradores que no son compatibles con las pestañas personalizadas también funciona para la autenticación: Opera, Opera Mini, InBrowser y Maxthon. Para obtener más información, lea [tabla los resultados de pruebas](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Android-system-browser#devices-and-browsers-tested).

## <a name="known-issues"></a>Problemas conocidos

- Si el usuario no tiene ningún explorador habilitada en el dispositivo, se producirá MSAL.NET un `AndroidActivityNotFound` excepción. 
  - **Mitigación**: Informar al usuario que debe habilitar un explorador (preferiblemente uno con compatibilidad con las pestañas personalizadas) en su dispositivo.

- Si se produce un error de autenticación (p. ej. la autenticación se lanza con DuckDuckGo), MSAL.NET devolverá un `AuthenticationCanceled MsalClientException`. 
  - **Raíz del problema**: Un explorador con compatibilidad de las pestañas personalizadas no se habilitó en el dispositivo. La autenticación se inicia con un explorador alternativo, que no se puede completar la autenticación. 
  - **Mitigación**: Informar al usuario que debe instalar un explorador (preferiblemente uno con el soporte técnico de la pestaña personalizada) en su dispositivo.

## <a name="devices-and-browsers-tested"></a>Dispositivos y exploradores probados
En la tabla siguiente se enumera los dispositivos y exploradores que se han probado.

| | Browser&ast;     |  Resultado  | 
| ------------- |:-------------:|:-----:|
| Huawei / uno + | Chrome&ast; | Sin errores|
| Huawei / uno + | Borde&ast; | Sin errores|
| Huawei / uno + | Firefox&ast; | Sin errores|
| Huawei / uno + | Audaces&ast; | Sin errores|
| Por lo menos una | Ecosia&ast; | Sin errores|
| Por lo menos una | Kiwi&ast; | Sin errores|
| Huawei / uno + | Opera | Sin errores|
| Huawei | OperaMini | Sin errores|
| Huawei / uno + | InBrowser | Sin errores|
| Por lo menos una | Maxthon | Sin errores|
| Huawei / uno + | DuckDuckGo | Autenticación de usuario cancelado|
| Huawei / uno + | Explorador de comunicaciones unificadas | Autenticación de usuario cancelado|
| Por lo menos una | Dolphin | Autenticación de usuario cancelado|
| Por lo menos una | Explorador de CM | Autenticación de usuario cancelado|
| Huawei / uno + | ninguna instalada | AndroidActivityNotFound p. ej.|

&ast; Es compatible con las pestañas personalizadas

## <a name="next-steps"></a>Pasos siguientes
Código de fragmentos de código e información adicional sobre el uso de explorador del sistema con Xamarin Android, lea esta [guía](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/MSAL.NET-uses-web-browser#choosing-between-embedded-web-browser-or-system-browser-on-xamarinandroid).  