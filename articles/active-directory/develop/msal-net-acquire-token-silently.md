---
title: Adquirir un token de forma silenciosa (biblioteca de autenticación de Microsoft para. NET) | Azure
description: Obtenga información sobre cómo adquirir un token de acceso en modo silencioso (desde la caché de tokens) mediante la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
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
ms.date: 04/30/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6331407067a39550d866d7c293a92fac9184b54e
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544233"
---
# <a name="get-a-token-from-the-token-cache-using-msalnet"></a>Obtener un token de la caché de tokens mediante MSAL.NET

Al adquirir un token de acceso mediante la biblioteca de autenticación de Microsoft para .NET (MSAL.NET), se almacena en caché el token. Cuando la aplicación necesita un token, debe llamar primero el `AcquireTokenSilent` método para comprobar si es un token aceptable en la memoria caché. En muchos casos, es posible adquirir otro token con más ámbitos según un token en la memoria caché. También es posible actualizar un token cuando va a caducar (como la caché de tokens también contiene un token de actualización).

El patrón recomendado es llamar a la `AcquireTokenSilent` método primero.  Si `AcquireTokenSilent` se produce un error, a continuación, adquirir un token mediante otros métodos.

En el ejemplo siguiente, primero la aplicación intenta adquirir un token de la caché de tokens.  Si un `MsalUiRequiredException` excepción, la aplicación adquiere un token de forma interactiva. 

```csharp
AuthenticationResult result = null;
var accounts = await app.GetAccountsAsync();

try
{
 result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
        .ExecuteAsync();
}
catch (MsalUiRequiredException ex)
{
 // A MsalUiRequiredException happened on AcquireTokenSilentAsync.
 // This indicates you need to call AcquireTokenAsync to acquire a token
 System.Diagnostics.Debug.WriteLine($"MsalUiRequiredException: {ex.Message}");

 try
 {
    result = await app.AcquireTokenInteractive(scopes)
          .ExecuteAsync();
 }
 catch (MsalException msalex)
 {
    ResultText.Text = $"Error Acquiring Token:{System.Environment.NewLine}{msalex}";
 }
}
catch (Exception ex)
{
 ResultText.Text = $"Error Acquiring Token Silently:{System.Environment.NewLine}{ex}";
 return;
}

if (result != null)
{
 string accessToken = result.AccessToken;
 // Use the token
}
```