---
title: Autenticación con agente en Android | Azure
titlesuffix: Microsoft identity platform
description: Introducción a la autenticación y autorización con agente para Android en la plataforma de identidad de Microsoft
services: active-directory
author: shoatman
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 08/25/2020
ms.author: shoatman
ms.custom: aaddev
ms.reviewer: shoatman, hahamil, brianmel
ms.openlocfilehash: 9042318d29b9a7fc8c2064bdf845d6f0d5a4f3e8
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88853853"
---
# <a name="brokered-authentication-in-android"></a>Autenticación con agente en Android

Debe usar uno de los agentes de autenticación de Microsoft para participar en el inicio de sesión único (SSO) en todo el dispositivo y para cumplir las directivas de acceso condicional de la organización. La integración con un agente proporciona las siguientes ventajas:

- Inicio de sesión único del dispositivo
- Acceso condicional para:
  - Intune App Protection
  - Registro de dispositivos (Workplace Join)
  - Administración de dispositivos móviles
- Administración de cuentas en todo el dispositivo
  -  Mediante la configuración y administración de cuentas de Android
  - "Cuenta profesional": tipo de cuenta personalizada

En Android, el agente de autenticación de Microsoft es un componente que se incluye con la [aplicación Microsoft Authenticator](https://play.google.com/store/apps/details?id=com.azure.authenticator) y el [Portal de empresa de Intune](https://play.google.com/store/apps/details?id=com.microsoft.windowsintune.companyportal).

> [!TIP]
> Solo habrá activa como agente una aplicación que hospede el agente a la vez. La aplicación que está activa como agente viene determinada por el orden de instalación del dispositivo. La primera en instalarse, o la última que quede en el dispositivo, se convierte en el agente activo.

En el siguiente diagrama se ilustra la relación entre la aplicación, la biblioteca de autenticación de Microsoft (MSAL) y los agentes de autenticación de Microsoft.

![Diagrama de implementación del agente](./media/brokered-auth/brokered-deployment-diagram.png)

## <a name="installing-apps-that-host-a-broker"></a>Instalación de aplicaciones que hospedan un agente

El propietario del dispositivo puede instalar aplicaciones que hospeden un agente desde su tienda de aplicaciones (normalmente Google Play Store) en cualquier momento. Sin embargo, algunas API (recursos) están protegidas por las directivas de acceso condicional que requieren que los dispositivos estén:

- registrados (unidos al área de trabajo); o
- inscritos en la administración de dispositivos; o
- inscritos en Intune App Protection.

Si un dispositivo aún no tiene instalada una aplicación de agente, MSAL indica al usuario que instale una en cuanto la aplicación intente obtener un token de forma interactiva. A continuación, la aplicación deberá guiar al usuario por los pasos necesarios para que el dispositivo sea compatible con la directiva requerida.

## <a name="effects-of-installing-and-uninstalling-a-broker"></a>Efectos de la instalación y desinstalación de un agente

### <a name="when-a-broker-is-installed"></a>Cuando se instala un agente

Cuando se instala un agente en un dispositivo, es el agente el que administra todas las solicitudes de token interactivas posteriores (llamadas a `acquireToken()`), en lugar de MSAL localmente. Cualquier estado de SSO disponible anteriormente para MSAL no está disponible para el agente. Como resultado, el usuario tendrá que autenticarse de nuevo o seleccionar una cuenta de la lista de cuentas existente que conoce el dispositivo.

La instalación de un agente no requiere que el usuario vuelva a iniciar sesión. Solo cuando el usuario necesite resolver una excepción `MsalUiRequiredException`, la siguiente solicitud pasará al agente. La excepción `MsalUiRequiredException` se produce por diversos motivos y debe resolverse de forma interactiva. Estos son algunos de los motivos habituales:

- El usuario cambió la contraseña asociada a su cuenta.
- La cuenta del usuario ya no cumple una directiva de acceso condicional.
- El usuario ha revocado su consentimiento para que la aplicación esté asociada a su cuenta.

### <a name="when-a-broker-is-uninstalled"></a>Cuando se desinstala un agente

Si solo hay instalada una aplicación que hospeda un agente y se quita, el usuario deberá volver a iniciar sesión. Al desinstalar el agente activo, se quitan del dispositivo la cuenta y los tokens asociados.

Si el Portal de empresa de Intune está instalado y funcionando como agente activo, y Microsoft Authenticator también está instalado, al desinstalar el Portal de empresa de Intune (agente activo), el usuario deberá volver a iniciar sesión. Cuando vuelva a iniciar sesión, la aplicación Microsoft Authenticator se convierte en el agente activo.

## <a name="integrating-with-a-broker"></a>Integración con un agente

### <a name="generating-a-redirect-uri-for-a-broker"></a>Generar un URI de redirección para un agente

Debe registrar un URI de redirección que sea compatible con el agente. El URI de redirección para el agente debe incluir el nombre del paquete de la aplicación, así como la representación codificada en Base64 de la firma de la aplicación.

El formato del URI de redirección es: `msauth://<yourpackagename>/<base64urlencodedsignature>`

Genere la firma con la dirección URL codificada en Base64 utilizando las claves de firma de la aplicación. Estos son algunos comandos de ejemplo que usan las claves de firma de depuración:

#### <a name="macos"></a>macOS

```bash
keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore | openssl sha1 -binary | openssl base64
```

#### <a name="windows"></a>Windows

```powershell
keytool -exportcert -alias androiddebugkey -keystore %HOMEPATH%\.android\debug.keystore | openssl sha1 -binary | openssl base64
```

Consulte [Firma de la aplicación](https://developer.android.com/studio/publish/app-signing) para más información sobre cómo firmar la aplicación.

> [!IMPORTANT]
> Use su clave de firma de producción para la versión de producción de la aplicación.

### <a name="configure-msal-to-use-a-broker"></a>Configuración de MSAL para usar un agente

Para usar un agente en la aplicación, debe confirmar que ha configurado la redirección del agente. Por ejemplo, para incluir el URI de redirección habilitado para el agente (e indicar que lo ha registrado), incluya lo siguiente en el archivo de configuración de MSAL:

```javascript
"redirect_uri" : "<yourbrokerredirecturi>",
"broker_redirect_uri_registered": true
```

> [!TIP]
> La nueva interfaz de usuario de registro de aplicaciones de Azure Portal ayuda a generar el URI de redirección del agente. Si registró la aplicación con la experiencia anterior o lo hizo con el portal de registro de aplicaciones de Microsoft, puede que tenga que generar el URI de redirección y actualizar la lista de URI de redirección en el portal manualmente.

### <a name="broker-related-exceptions"></a>Excepciones relacionadas con el agente

MSAL se comunica con el agente de dos maneras:

- Servicio enlazado al agente
- Administrador de cuentas de Android

MSAL usa primero el servicio enlazado al agente porque la llamada a este servicio no requiere ningún permiso de Android. Si se produce un error al enlazar con el servicio enlazado, MSAL usará la API de administrador de cuentas de Android. MSAL solo lo hace si la aplicación ya tiene concedido el permiso `"READ_CONTACTS"`.

Si obtiene una excepción `MsalClientException` con el código de error `"BROKER_BIND_FAILURE"`, hay dos opciones:

- Pida al usuario que deshabilite la optimización de energía para la aplicación Microsoft Authenticator y el Portal de empresa de Intune.
- Pida al usuario que conceda el permiso `"READ_CONTACTS"`.

## <a name="verifying-broker-integration"></a>Comprobación de la integración del agente

Es posible que no quede claro inmediatamente que la integración del agente funcione, pero puede seguir estos los pasos siguientes para comprobarlo:

1. En el dispositivo Android, complete una solicitud con el agente.
1. En la configuración del dispositivo Android, busque una cuenta recién creada que se corresponda con la cuenta con la que se autenticó. La cuenta debe ser del tipo *Cuenta profesional*.

Puede quitar la cuenta de la configuración si desea repetir la prueba.
