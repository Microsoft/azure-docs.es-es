---
title: Uso de Azure Notification Hubs con PHP
description: Obtenga información acerca de cómo usar Azure Notification Hubs desde un back-end de PHP.
services: notification-hubs
documentationcenter: ''
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: php
ms.devlang: php
ms.topic: article
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.openlocfilehash: daebf7c6a5fc9056e16b77a40ee9f90db598749f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87076624"
---
# <a name="how-to-use-notification-hubs-from-php"></a>Uso de Notification Hubs desde PHP

[!INCLUDE [notification-hubs-backend-how-to-selector](../../includes/notification-hubs-backend-how-to-selector.md)]

Puede acceder a todas las características de Notification Hubs desde un back-end de Java, PHP o Ruby usando la interfaz REST de Notification Hubs tal como se describe en el tema de MSDN [API de REST de Notification Hubs](/previous-versions/azure/reference/dn223264(v=azure.100)).

En este tema le mostraremos cómo:

* Crear un cliente REST para las características de Notification Hubs en PHP;
* Siga los pasos descritos en [Envío de notificaciones push a aplicaciones iOS mediante Azure Notification Hubs](ios-sdk-get-started.md) en la plataforma móvil que elija, implementando la parte del back-end en PHP.

## <a name="client-interface"></a>Interfaz del cliente

La interfaz del cliente principal puede proporcionar los mismos métodos disponibles en el [SDK de Notification Hubs .NET](https://msdn.microsoft.com/library/jj933431.aspx), que le permite trasladar todos los tutoriales y ejemplos actualmente disponibles en este sitio directamente y con el que contribuye la comunidad en Internet.

Puede encontrar todo el código disponible en el [ejemplo de contenedor REST para PHP].

Por ejemplo, para crear un cliente:

```php
$hub = new NotificationHub("connection string", "hubname");
```

Para enviar una notificación nativa de iOS:

```php
$notification = new Notification("apple", '{"aps":{"alert": "Hello!"}}');
$hub->sendNotification($notification, null);
```

## <a name="implementation"></a>Implementación

Si todavía no lo ha hecho, siga el [tutorial introductorio] hasta la última sección en la que tiene que implementar el back-end.
También puede usar el código del [ejemplo de contenedor REST para PHP] e ir directamente a la sección [Finalización del tutorial](#complete-tutorial).

Puede encontrar todos los detalles para implementar un contenedor REST completo en [MSDN](/previous-versions/azure/reference/dn530746(v=azure.100)). En esta sección se describe la implementación para PHP de los principales pasos requeridos para acceder a puntos de conexión REST de Notification Hubs:

1. Análisis de la cadena de conexión
2. Generación del token de autenticación
3. Realización de la llamada HTTP

### <a name="parse-the-connection-string"></a>Análisis de la cadena de conexión

Esta es la clase principal que implementa el cliente, cuyo constructor analiza la cadena de conexión:

```php
class NotificationHub {
    const API_VERSION = "?api-version=2013-10";

    private $endpoint;
    private $hubPath;
    private $sasKeyName;
    private $sasKeyValue;

    function __construct($connectionString, $hubPath) {
        $this->hubPath = $hubPath;

        $this->parseConnectionString($connectionString);
    }

    private function parseConnectionString($connectionString) {
        $parts = explode(";", $connectionString);
        if (sizeof($parts) != 3) {
            throw new Exception("Error parsing connection string: " . $connectionString);
        }

        foreach ($parts as $part) {
            if (strpos($part, "Endpoint") === 0) {
                $this->endpoint = "https" . substr($part, 11);
            } else if (strpos($part, "SharedAccessKeyName") === 0) {
                $this->sasKeyName = substr($part, 20);
            } else if (strpos($part, "SharedAccessKey") === 0) {
                $this->sasKeyValue = substr($part, 16);
            }
        }
    }
}
```

### <a name="create-a-security-token"></a>Creación de un rol de seguridad

Consulte la documentación de Azure para obtener información acerca de cómo [crear un token de seguridad de SAS](/previous-versions/azure/reference/dn495627(v=azure.100)#create-sas-security-token).

Agregue el método `generateSasToken` a la clase `NotificationHub` para crear el token basándose en el URI de la solicitud actual y en las credenciales extraídas de la cadena de conexión.

```php
private function generateSasToken($uri) {
    $targetUri = strtolower(rawurlencode(strtolower($uri)));

    $expires = time();
    $expiresInMins = 60;
    $expires = $expires + $expiresInMins * 60;
    $toSign = $targetUri . "\n" . $expires;

    $signature = rawurlencode(base64_encode(hash_hmac('sha256', $toSign, $this->sasKeyValue, TRUE)));

    $token = "SharedAccessSignature sr=" . $targetUri . "&sig="
                . $signature . "&se=" . $expires . "&skn=" . $this->sasKeyName;

    return $token;
}
```

### <a name="send-a-notification"></a>Envío de una notificación

En primer lugar, definamos una clase que representa una notificación.

```php
class Notification {
    public $format;
    public $payload;

    # array with keynames for headers
    # Note: Some headers are mandatory: Windows: X-WNS-Type, WindowsPhone: X-NotificationType
    # Note: For Apple you can set Expiry with header: ServiceBusNotification-ApnsExpiry in W3C DTF, YYYY-MM-DDThh:mmTZD (for example, 1997-07-16T19:20+01:00).
    public $headers;

    function __construct($format, $payload) {
        if (!in_array($format, ["template", "apple", "windows", "fcm", "windowsphone"])) {
            throw new Exception('Invalid format: ' . $format);
        }

        $this->format = $format;
        $this->payload = $payload;
    }
}
```

Esta clase es un contenedor para un cuerpo de notificación nativa, o un conjunto de propiedades en el caso de una notificación de plantilla, y un conjunto de encabezados que contienen formato (plataforma o plantilla nativa) y propiedades específicas de la plataforma (como la propiedad de expiración de Apple y los encabezados WNS).

Consulte la [documentación de las API de REST de Notification Hubs](/previous-versions/azure/reference/dn495827(v=azure.100)) y los formatos de las plataformas de notificación específicas para todas las opciones disponibles.

Con esta clase, ahora podemos escribir los métodos de envío de notificaciones dentro de la clase `NotificationHub`:

```php
public function sendNotification($notification, $tagsOrTagExpression="") {
    if (is_array($tagsOrTagExpression)) {
        $tagExpression = implode(" || ", $tagsOrTagExpression);
    } else {
        $tagExpression = $tagsOrTagExpression;
    }

    # build uri
    $uri = $this->endpoint . $this->hubPath . "/messages" . NotificationHub::API_VERSION;
    $ch = curl_init($uri);

    if (in_array($notification->format, ["template", "apple", "fcm"])) {
        $contentType = "application/json";
    } else {
        $contentType = "application/xml";
    }

    $token = $this->generateSasToken($uri);

    $headers = [
        'Authorization: '.$token,
        'Content-Type: '.$contentType,
        'ServiceBusNotification-Format: '.$notification->format
    ];

    if ("" !== $tagExpression) {
        $headers[] = 'ServiceBusNotification-Tags: '.$tagExpression;
    }

    # add headers for other platforms
    if (is_array($notification->headers)) {
        $headers = array_merge($headers, $notification->headers);
    }

    curl_setopt_array($ch, array(
        CURLOPT_POST => TRUE,
        CURLOPT_RETURNTRANSFER => TRUE,
        CURLOPT_SSL_VERIFYPEER => FALSE,
        CURLOPT_HTTPHEADER => $headers,
        CURLOPT_POSTFIELDS => $notification->payload
    ));

    // Send the request
    $response = curl_exec($ch);

    // Check for errors
    if($response === FALSE){
        throw new Exception(curl_error($ch));
    }

    $info = curl_getinfo($ch);

    if ($info['http_code'] <> 201) {
        throw new Exception('Error sending notification: '. $info['http_code'] . ' msg: ' . $response);
    }
} 
```

Los métodos anteriores envían una solicitud POST HTTP al extremo `/messages` del centro de notificaciones, con el cuerpo y encabezados correctos para enviar la notificación.

## <a name="complete-the-tutorial"></a><a name="complete-tutorial"></a>Finalización del tutorial

Ahora puede completar el tutorial introductorio enviando la notificación desde un back-end de PHP.

Inicialice el cliente de Notification Hubs (reemplace la cadena de conexión y el nombre del centro tal como se indica en el [tutorial introductorio]):

```php
$hub = new NotificationHub("connection string", "hubname");
```

Después, agregue el código de envío dependiendo de la plataforma móvil de destino.

### <a name="windows-store-and-windows-phone-81-non-silverlight"></a>Tienda Windows y Windows Phone 8.1 (no Silverlight)

```php
$toast = '<toast><visual><binding template="ToastText01"><text id="1">Hello from PHP!</text></binding></visual></toast>';
$notification = new Notification("windows", $toast);
$notification->headers[] = 'X-WNS-Type: wns/toast';
$hub->sendNotification($notification, null);
```

### <a name="ios"></a>iOS

```php
$alert = '{"aps":{"alert":"Hello from PHP!"}}';
$notification = new Notification("apple", $alert);
$hub->sendNotification($notification, null);
```

### <a name="android"></a>Android

```php
$message = '{"data":{"msg":"Hello from PHP!"}}';
$notification = new Notification("fcm", $message);
$hub->sendNotification($notification, null);
```

### <a name="windows-phone-80-and-81-silverlight"></a>Silverlight para Windows Phone 8.0 y 8.1

```php
$toast = '<?xml version="1.0" encoding="utf-8"?>' .
            '<wp:Notification xmlns:wp="WPNotification">' .
               '<wp:Toast>' .
                    '<wp:Text1>Hello from PHP!</wp:Text1>' .
               '</wp:Toast> ' .
            '</wp:Notification>';
$notification = new Notification("windowsphone", $toast);
$notification->headers[] = 'X-WindowsPhone-Target : toast';
$notification->headers[] = 'X-NotificationClass : 2';
$hub->sendNotification($notification, null);
```

### <a name="kindle-fire"></a>Kindle Fire

```php
$message = '{"data":{"msg":"Hello from PHP!"}}';
$notification = new Notification("adm", $message);
$hub->sendNotification($notification, null);
```

La ejecución del código de PHP debe generar ahora una notificación que aparece en el dispositivo de destino.

## <a name="next-steps"></a>Pasos siguientes

En este tema hemos mostrado cómo crear un sencillo cliente de REST en Java para Notification Hubs. Desde aquí, puede:

* Descargar el [ejemplo de contenedor REST para PHP]completo, que contiene todo el código anterior.
* Continuar aprendiendo sobre la característica de etiquetado de Notification Hubs en el [tutorial Noticias de última hora]
* Obtener más información sobre notificaciones de inserción para usuarios individuales en el [tutorial Notificar a los usuarios]

Para obtener más información, consulte también el [Centro para desarrolladores de PHP](https://azure.microsoft.com/develop/php/).

[ejemplo de contenedor REST para PHP]: https://github.com/Azure/azure-notificationhubs-samples/tree/master/notificationhubs-rest-php
[Envío de notificaciones push a aplicaciones iOS mediante Azure Notification Hubs](ios-sdk-get-started.md))
