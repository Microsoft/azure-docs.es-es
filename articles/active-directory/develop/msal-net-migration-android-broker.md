---
title: Migración de aplicaciones Xamarin Android mediante agentes a MSAL.NET
titleSuffix: Microsoft identity platform
description: Aprenda a migrar aplicaciones Xamarin Android que usan Microsoft Authenticator o el Portal de empresa de Intune de ADAL.NET a MSAL.NET.
author: aiwang
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 08/31/2020
ms.author: aiwang
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: 47902b29da1dfe20fb54d633a0559e6a337fd771
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89183523"
---
# <a name="migrate-android-applications-that-use-a-broker-from-adalnet-to-msalnet"></a>Migración de aplicaciones Android que usan un agente de ADAL.NET a MSAL.NET

Si tiene una aplicación Xamarin Android que usa actualmente la Biblioteca de autenticación de Active Directory para .NET (ADAL.NET) y un [agente de autenticación](brokered-auth.md), es el momento de migrar a la [Biblioteca de autenticación de Microsoft para .NET ](msal-overview.md) (MSAL.NET).

## <a name="prerequisites"></a>Requisitos previos

* Una aplicación Xamarin Android ya integrada con un agente ([Microsoft Authenticator](https://play.google.com/store/apps/details?id=com.azure.authenticator) o el [Portal de empresa de Intune](https://play.google.com/store/apps/details?id=com.microsoft.windowsintune.companyportal)) y ADAL.NET que necesita migrar a MSAL.NET.

## <a name="step-1-enable-the-broker"></a>Paso 1: Habilitar el agente

<table>
<tr><td>Código de ADAL actual:</td><td>Homólogo de MSAL:</td></tr>
<tr><td>
En ADAL.NET, la compatibilidad con el agente se habilita en cada contexto de autenticación.

Para llamar al agente, tenía que establecer una marca `useBroker` en *true* en el constructor `PlatformParameters`:

```CSharp
public PlatformParameters(
        Activity callerActivity,
        bool useBroker)
```

En el código del representador de páginas específico de la plataforma para Android, se establece la marca `useBroker` en true:

```CSharp
page.BrokerParameters = new PlatformParameters(
        this,
        true,
        PromptBehavior.SelectAccount);
```

A continuación, incluya los parámetros en la llamada de adquisición de token:

```CSharp
AuthenticationResult result =
        await
            AuthContext.AcquireTokenAsync(
                Resource,
                ClientId,
                new Uri(RedirectURI),
                platformParameters)
                .ConfigureAwait(false);
```

</td><td>
En MSAL.NET, la compatibilidad con el agente se habilita según el elemento PublicClientApplication.

Use el parámetro `WithBroker()` (establecido en true de manera predeterminada) para llamar al agente:

```CSharp
var app = PublicClientApplicationBuilder
                .Create(ClientId)
                .WithBroker()
                .WithRedirectUri(redirectUriOnAndroid)
                .Build();
```

Después, en la llamada a AcquireToken:

```CSharp
result = await app.AcquireTokenInteractive(scopes)
             .WithParentActivityOrWindow(App.RootViewController)
             .ExecuteAsync();
```
</table>

## <a name="step-2-set-an-activity"></a>Paso 2: Establecer una actividad

En ADAL.NET, se pasaba una actividad (normalmente MainActivity) como parte de PlatformParameters, tal como se muestra en [Paso 1: Habilitar el agente](#step-1-enable-the-broker).

MSAL.NET también utiliza una actividad, pero no es necesario en el uso normal de Android sin un agente. Para usar el agente, establezca la actividad para enviar y recibir respuestas del agente.

<table>
<tr><td>Código de ADAL actual:</td><td>Homólogo de MSAL:</td></tr>
<tr><td>
La actividad se pasa a PlatformParameters en la plataforma específica de Android.

```CSharp
page.BrokerParameters = new PlatformParameters(
          this,
          true,
          PromptBehavior.SelectAccount);
```
</td><td>

En MSAL.NET, haga dos cosas para establecer la actividad para Android:

1. En `MainActivity.cs`, establezca el elemento `App.RootViewController` en `MainActivity` para asegurarse de que haya una actividad con la llamada al agente.

    Si no se establece correctamente, puede obtener este error: `"Activity_required_for_android_broker":"Activity is null, so MSAL.NET cannot invoke the Android broker. See https://aka.ms/Brokered-Authentication-for-Android"`

1. En la llamada a AcquireTokenInteractive, use `.WithParentActivityOrWindow(App.RootViewController)` y pase la referencia a la actividad que usará. En este ejemplo se usará MainActivity.

**Por ejemplo:**

En *App.cs*:

```CSharp
   public static object RootViewController { get; set; }
```

En *MainActivity.cs*:

```CSharp
   LoadApplication(new App());
   App.RootViewController = this;
```

En la llamada a AcquireToken:

```CSharp
result = await app.AcquireTokenInteractive(scopes)
             .WithParentActivityOrWindow(App.RootViewController)
             .ExecuteAsync();
```
</table>

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las consideraciones específicas de Android al usar MSAL.NET con Xamarin, consulte [Requisitos de configuración y sugerencias para la solución de problemas de Xamarin Android con MSAL.NET](msal-net-xamarin-android-considerations.md).