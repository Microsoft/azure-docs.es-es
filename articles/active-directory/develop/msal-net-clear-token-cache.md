---
title: Borrar la caché de tokens mediante la biblioteca de autenticación de Microsoft para .NET - Azure
description: Obtenga información sobre cómo borrar la caché de tokens mediante la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
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
ms.date: 05/07/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: c6763c6b2b1f9b4de7d8669a50a4979a7aac00c7
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544114"
---
# <a name="clear-the-token-cache-using-msalnet"></a>Borrar la caché de tokens mediante MSAL.NET

Cuando se [adquirir un token de acceso](msal-acquire-cache-tokens.md) mediante la biblioteca de autenticación de Microsoft para .NET (MSAL.NET), el token se almacena en caché. Cuando la aplicación necesita un token, debe llamar primero el `AcquireTokenSilent` método para comprobar si es un token aceptable en la memoria caché. 

Borrar la memoria caché se logra mediante la eliminación de las cuentas de la memoria caché. No obstante, esto no elimina la cookie de sesión que se encuentra en el explorador.  El ejemplo siguiente se crea una instancia de una aplicación de cliente público, obtiene las cuentas de la aplicación y quita las cuentas.

```csharp
private readonly IPublicClientApplication _app;
private static readonly string ClientId = ConfigurationManager.AppSettings["ida:ClientId"];
private static readonly string Authority = string.Format(CultureInfo.InvariantCulture, AadInstance, Tenant);

_app = PublicClientApplicationBuilder.Create(ClientId)
                .WithAuthority(Authority)
                .Build();

var accounts = (await _app.GetAccountsAsync()).ToList();

// clear the cache
while (accounts.Any())
{
   await _app.RemoveAsync(accounts.First());
   accounts = (await _app.GetAccountsAsync()).ToList();
}

```

Para obtener más información sobre la adquisición y almacenamiento en caché de tokens, lea [adquirir un token de acceso](msal-acquire-cache-tokens.md).
