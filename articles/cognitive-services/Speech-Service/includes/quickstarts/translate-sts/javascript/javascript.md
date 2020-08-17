---
author: IEvangelist
ms.service: cognitive-services
ms.topic: include
ms.date: 04/03/2020
ms.author: trbye
ms.custom: devx-track-javascript
ms.openlocfilehash: 4c685a5d9b2cda5ee7884fb9806f55878653605f
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87404349"
---
## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar:

> [!div class="checklist"]
> * <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesSpeechServices" target="_blank">Creación de un recurso de voz de Azure <span class="docon docon-navigate-external x-hidden-focus"></span></a>
> * [Configuración del entorno de desarrollo y creación de un proyecto vacío](../../../../quickstarts/setup-platform.md)

## <a name="create-a-new-website-folder"></a>Creación de una nueva carpeta Website

Cree una nueva carpeta vacía. En caso de que desee hospedar el ejemplo en un servidor web, asegúrese de que este puede acceder a la carpeta.

## <a name="unpack-the-speech-sdk-for-javascript-into-that-folder"></a>Desempaquete el SDK de Voz para JavaScript en esa carpeta

Descargue el SDK de Voz en forma de [paquete .zip](https://aka.ms/csspeech/jsbrowserpackage) y desempaquételo en la carpeta recién creada. Como resultado se desempaquetan dos archivos, `microsoft.cognitiveservices.speech.sdk.bundle.js` y `microsoft.cognitiveservices.speech.sdk.bundle.js.map`.
El último archivo es opcional y es útil para la depuración en el código del SDK.

## <a name="create-an-indexhtml-page"></a>Creación de una página index.html

Cree un nuevo archivo llamado `index.html` en la carpeta y ábralo con un editor de texto.

1. Cree el siguiente esqueleto HTML:

 [!code-html [SampleCode](~/samples-cognitive-services-speech-sdk/quickstart/javascript/browser/index-sts.html)]

## <a name="create-the-token-source-optional"></a>Creación del origen del token (opcional)

En caso de que desea hospedar la página web en un servidor web, opcionalmente puede proporcionar un origen del token para la aplicación de demostración.
De esa forma, la clave de la suscripción nunca saldrá el servidor, lo que permitirá a los usuarios utilizar las funcionalidades de voz sin escribir ningún código de autorización.

Cree un nuevo archivo llamado `token.php`. En este ejemplo se supone que el servidor web admite el lenguaje de scripting PHP con cURL habilitado. Escriba el siguiente código:

```php
<?php
header('Access-Control-Allow-Origin: ' . $_SERVER['SERVER_NAME']);

// Replace with your own subscription key and service region (e.g., "westus").
$subscriptionKey = 'YourSubscriptionKey';
$region = 'YourServiceRegion';

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, 'https://' . $region . '.api.cognitive.microsoft.com/sts/v1.0/issueToken');
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS, '{}');
curl_setopt($ch, CURLOPT_HTTPHEADER, array('Content-Type: application/json', 'Ocp-Apim-Subscription-Key: ' . $subscriptionKey));
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
echo curl_exec($ch);
?>
```

> [!NOTE]
> Los tokens de autorización tienen una vigencia limitada.
> Este ejemplo simplificado no muestra cómo actualizar automáticamente tokens de autorización. Como usuario, puede volver a cargar la página manualmente o presione la tecla F5 para actualizarla.

## <a name="build-and-run-the-sample-locally"></a>Compilación y ejecución local del ejemplo

Para iniciar la aplicación, haga doble clic en el archivo index.html o ábralo con el explorador web que prefiera. Presentará una interfaz gráfica de usuario simple que le permitirá escribir su clave de suscripción y [región](../../../../regions.md), y desencadenará una transcripción a texto de la voz de entrada.

## <a name="build-and-run-the-sample-via-a-web-server"></a>Compilación y ejecución del ejemplo mediante un servidor web

Para iniciar la aplicación, abra el explorador web que prefiera y apunte a la dirección URL pública en la que hospeda la carpeta, escriba su [región](../../../../regions.md) y desencadene la transcripción a texto de la voz de entrada. Si lo ha configurado, adquirirá un token del origen del token.

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [footer](./footer.md)]