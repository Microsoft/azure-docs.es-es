---
title: Autenticación basada en token (HTTP/2) para APNs en Azure Notification Hubs | Microsoft Docs
description: Obtenga información sobre cómo usar la nueva autenticación de token para APNs.
services: notification-hubs
documentationcenter: .net
author: sethmanheim
manager: femila
editor: jwargo
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: dotnet
ms.topic: article
ms.date: 11/14/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 02/13/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: e8258aeef33be9192608f0d4cb29b46f08da64e2
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89010637"
---
# <a name="token-based-http2-authentication-for-apns"></a>Autenticación basada en token (HTTP/2) para APNs

## <a name="overview"></a>Información general

Este artículo explica cómo usar el nuevo protocolo HTTP/2 de APNs con la autenticación basada en token.

Entre las ventajas principales de usar el nuevo protocolo destacan las siguientes:

* La generación de tokens es relativamente simple (comparada con los certificados)
* Ya no hay fechas de expiración: es usted quien controla los tokens de autenticación y su revocación.
* Ahora las cargas pueden ser de hasta 4 KB.
* Los comentarios son sincrónicos.
* Está en el protocolo más reciente de Apple: los certificados siguen usando el protocolo binario, que está marcado como en desuso

Para usar este mecanismo nuevo, basta con que realice dos pasos:

* Obtenga la información necesaria en el portal de la cuenta de Desarrollador de Apple.
* Configure el centro de notificaciones con la nueva información.

Ahora, Notification Hubs está configurados para usar el nuevo sistema de autenticación con APNs.

Tenga en cuenta que si migró del uso de credenciales de certificado para APNS, las propiedades del token sobrescriben su certificado en nuestro sistema, pero su aplicación continúa recibiendo notificaciones sin problemas.

## <a name="obtaining-authentication-information-from-apple"></a>Obtener información de autenticación de Apple

Para habilitar la autenticación basada en token, necesita las siguientes propiedades de la cuenta de Desarrollador de Apple:

### <a name="key-identifier"></a>Identificador de clave

El identificador de clave se puede obtener de la página **Claves** en **Certificados, identificadores y perfiles**, en su cuenta de Desarrollador de Apple:

![Certificados](./media/notification-hubs-push-notification-http2-token-authentification/keys.png)

![Identificadores](./media/notification-hubs-push-notification-http2-token-authentification/obtaining-auth-information-from-apple.png)

### <a name="application-identifier-and-application-name"></a>Identificador y nombre de la aplicación

El nombre y el identificador de la aplicación también están disponibles en la página **Certificados, identificadores y perfiles** en la cuenta del desarrollador:

![Certificados e identificadores](./media/notification-hubs-push-notification-http2-token-authentification/app-name.png)

### <a name="configure-via-the-net-sdk-or-the-azure-portal"></a>Configurar mediante SDK de .NET o el Azure Portal

Puede configurar el centro para que use la autenticación basada en token mediante nuestro [SDK de cliente más reciente](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs) o en el Azure Portal. Para habilitar la autenticación basada en token en el portal, inicie sesión en Azure Portal y vaya al panel **​​Configuración> Apple (APNS)** de su centro de notificaciones. Seleccione **token** en la propiedad **Modo de autenticación** para actualizar el concentrador con todas las propiedades de token relevantes.

![Configurara token](./media/notification-hubs-push-notification-http2-token-authentification/azure-portal-apns-settings.png)

* Escriba las propiedades que ha recuperado de la cuenta de Desarrollador de Apple.
* Elija el modo de aplicación (**producción** o **espacio aislado**).
* Haga clic en el botón **Guardar** para actualizar las credenciales de APNS.

Las credenciales basadas en token se componen de los siguientes campos:

* **El identificador de la clave**: Identificador de la clave privada generada en el portal para Desarrolladores de Apple, por ejemplo, `2USFGKSKLT`.
* **Identificador del equipo**: También se denomina "prefijo" o "Prefijo de aplicación." Este es el identificador de la organización en el portal para Desarrolladores de Apple, por ejemplo, `S4V3D7CHJR`.
* **Identificador de agrupación**: También se denomina "Identificador de la aplicación." Este es el identificador de agrupación de la aplicación, por ejemplo, `com.example.myapp`. Tenga en cuenta que puede usar una clave para muchas aplicaciones. Este valor se asigna al encabezado HTTP `apns-topic` al enviar una notificación y se usa para apuntar a la aplicación específica. Tenga en cuenta que no puede establecer el valor de `apns-topic` explícitamente.
* **Token**: También se denomina "clave" o "clave privada." Este se obtiene del archivo. P8 generado en el portal para Desarrolladores de Apple. La clave debe tener APNS habilitado (que se selecciona en el portal de Desarrolladores de Apple al generar la clave). El valor debe tener el encabezado PE/pie de página que se elimina cuando lo proporcione al NH Portal/API.
* **Punto de conexión**: Este es un comando en la hoja del portal de Notification Hubs y un campo de cadena en la API. Los valores válidos son `https://api.development.push.apple.com:443/3/device` y `https://api.sandbox.push.apple.com:443/3/device`. Notification Hubs utiliza este valor para el entorno de producción o espacio aislado, para enviar notificaciones. Esto debe coincidir con el derecho `aps-environment` en la aplicación; de lo contrario, los tokens del dispositivo APNS generados no coinciden con el entorno y las notificaciones no se envían.

A continuación se muestra un ejemplo de código que ilustra el uso correcto:

```csharp
NamespaceManager nm = NamespaceManager.CreateFromConnectionString(_endpoint);
string token = "YOUR PRIVATE KEY HERE";
string keyId = "YOUR KEY ID HERE";
string appName = "YOUR APP NAME HERE";
string appId = "YOUR APP ID HERE";
NotificationHubDescription desc = new NotificationHubDescription("PATH TO YOUR HUB");
desc.ApnsCredential = new ApnsCredential(token, keyId, appId, appName);
desc.ApnsCredential.Endpoint = @"https://api.development.push.apple.com:443/3/device";
nm.UpdateNotificationHubAsync(desc);
```

## <a name="next-steps"></a>Pasos siguientes

* [Creación de un centro de notificaciones de Azure en Azure Portal](create-notification-hub-portal.md)
* [Configuración de un centro de notificaciones en el Azure Portal](create-notification-hub-portal.md)
