---
title: Proporcionar HttpClient y proxy (MSAL.NET) | Azure
titleSuffix: Microsoft identity platform
description: Aprenda a proporcionar su propio HttpClient y proxy para conectarse a Azure AD mediante la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 04/23/2019
ms.author: jmprieur
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 6099bfe339ae398e4683f17a4397656a4507b6ef
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88166015"
---
# <a name="providing-your-own-httpclient-and-proxy-using-msalnet"></a>Proporcionar su propio HttpClient y proxy mediante MSAL.NET
Al [inicializar una aplicación cliente pública](msal-net-initializing-client-applications.md), puede usar `.WithHttpClientFactory method` para proporcionar su propio HttpClient.  Cuando proporciona su propio HttpClient, tiene acceso a escenarios avanzados; así, puede controlar un proxy HTTP de forma pormenorizada, personalizar los encabezados de agente de usuario o forzar a que MSAL use un HttpClient específico (por ejemplo, en aplicaciones web o API de ASP.NET Core).

## <a name="initialize-with-httpclientfactory"></a>Inicialización con HttpClientFactory
En el ejemplo siguiente se muestra cómo crear un objeto `HttpClientFactory` y, luego, inicializar una aplicación cliente pública con él:

```csharp
IMsalHttpClientFactory httpClientFactory = new MyHttpClientFactory();

var pca = PublicClientApplicationBuilder.Create(MsalTestConstants.ClientId) 
                                        .WithHttpClientFactory(httpClientFactory)
                                        .Build();
```

## <a name="httpclient-and-xamarin-ios"></a>HttpClient y Xamarin iOS
Al usar Xamarin iOS, se recomienda crear un objeto `HttpClient` que use explícitamente el controlador basado en `NSURLSession`-para iOS 7 y versiones más recientes. MSAL.NET crea automáticamente un objeto `HttpClient` que usa `NSURLSessionHandler` para iOS 7 y versiones más recientes. Para más información, lea la [documentación de Xamarin iOS para HttpClient](/xamarin/cross-platform/macios/http-stack).