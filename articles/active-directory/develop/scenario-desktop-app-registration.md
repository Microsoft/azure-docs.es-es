---
title: 'Registro de aplicaciones de escritorio que llaman a las API web: Plataforma de identidad de Microsoft | Azure'
description: Aprenda a compilar una aplicación de escritorio que llame a las API web (registro de aplicaciones).
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/09/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 6796ac42a10d3b976b23f5af1418b1789011d61b
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89440956"
---
# <a name="desktop-app-that-calls-web-apis-app-registration"></a>Aplicación de escritorio que llama a las API web: Registro de aplicación

En este artículo se tratan los detalles del registro de una aplicación de escritorio.

## <a name="supported-account-types"></a>Tipos de cuenta admitidos

Los tipos de cuenta que se admiten en una aplicación de escritorio dependen de la experiencia que quiera destacar. Debido a esta relación, los tipos de cuenta admitidos dependen de los flujos que desee usar.

### <a name="audience-for-interactive-token-acquisition"></a>Público para la adquisición interactiva de tokens

Si la aplicación de escritorio usa la autenticación interactiva, puede iniciar la sesión de los usuarios desde cualquier [tipo de cuenta](quickstart-register-app.md).

### <a name="audience-for-desktop-app-silent-flows"></a>Audiencia para flujos silenciosos de la aplicación de escritorio

- Para usar la autenticación integrada o un nombre de usuario y una contraseña, la aplicación debe iniciar la sesión de los usuarios en su propio inquilino, por ejemplo, si es un desarrollador de línea de negocio (LOB). O bien, en organizaciones de Azure Active Directory, la aplicación debe iniciar la sesión de los usuarios en su propio inquilino si se trata de un escenario de ISV. Estos flujos de autenticación no son compatibles con las cuentas personales de Microsoft.
- Si quiere utilizar el flujo de código del dispositivo, aún no puede iniciar la sesión de los usuarios con sus cuentas personales de Microsoft.
- Si inicia la sesión de los usuarios con identidades sociales que pasan una autoridad y directiva de negocio a comercio (B2C), solo puede usar la autenticación interactiva y de nombre de usuario y contraseña.

## <a name="redirect-uris"></a>URI de redirección

Los URI de redirección que se utilizan en una aplicación de escritorio dependen del flujo que se quiera utilizar.

- Si usa la autenticación interactiva o el flujo de código del dispositivo, use `https://login.microsoftonline.com/common/oauth2/nativeclient`. Para lograr esta configuración, seleccione la dirección URL correspondiente en la sección **Autenticación** de la aplicación.

  > [!IMPORTANT]
  > Hoy en día, MSAL.NET usa otro URI de redirección de forma predeterminada en las aplicaciones de escritorio que se ejecutan en Windows (`urn:ietf:wg:oauth:2.0:oob`). En el futuro, querremos cambiar este comportamiento predeterminado, por lo que le recomendamos que use `https://login.microsoftonline.com/common/oauth2/nativeclient`.

- Si compila una aplicación nativa de Objective-C o Swift para macOS, registre el URI de redirección en función del identificador de agrupación de la aplicación, con el formato siguiente: msauth.<su.id.de.agrupación.de.aplicación>://auth. Reemplace "<su.id.de.agrupación.de.aplicación>" con el identificador de agrupación de la aplicación.
- Si la aplicación solo utiliza la autenticación integrada de Windows o un nombre de usuario y una contraseña, no es necesario que registre ningún URI de redirección para la aplicación. Estos flujos realizan un recorrido de ida y vuelta al punto de conexión de la plataforma de identidad de Microsoft v2.0. No se volverá a llamar a la aplicación en ningún URI específico.
- Para distinguir el flujo de código de dispositivo, la autenticación integrada de Windows y el nombre de usuario y la contraseña de un flujo de la aplicación cliente confidencial, que tampoco tiene URI de redirección (corresponde al flujo de credenciales de cliente usado en aplicaciones de demonio), deberá expresar que se trata de una aplicación cliente pública. Para lograr esta configuración, vaya a la sección **Autenticación** de su aplicación. En la subsección **Configuración avanzada**, en el párrafo **Tipo de cliente predeterminado**, seleccione **Sí** en la opción **Trate la aplicación como un cliente público**.

  ![Permitir el cliente público](media/scenarios/default-client-type.png)

## <a name="api-permissions"></a>Permisos de API

Las aplicaciones de escritorio llaman a las API en nombre del usuario que tiene la sesión iniciada. Tienen que solicitar permisos delegados. No pueden solicitar permisos de la aplicación, que solo se controlan en las [aplicaciones de demonio](scenario-daemon-overview.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Aplicación de escritorio: configuración de la aplicación](scenario-desktop-app-configuration.md)
