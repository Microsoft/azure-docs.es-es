---
title: Realización de una llamada telefónica desde Twilio (PHP) | Microsoft Docs
description: Aprenda a realizar llamadas telefónicas y a enviar mensajes SMS con el servicio de la API de Twilio en Azure. Ejemplos para la aplicación PHP.
documentationcenter: php
services: ''
author: georgewallace
ms.assetid: 44e35adc-be06-4700-beee-8c9e2c20c540
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: PHP
ms.topic: article
ms.date: 11/25/2014
ms.author: gwallace
ms.openlocfilehash: e75b5a265ba85813581e39a548d897ea57a33da6
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88080565"
---
# <a name="how-to-make-a-phone-call-using-twilio-in-a-php-application-on-azure"></a>Realización de una llamada telefónica con Twilio en una aplicación PHP en Azure
El siguiente ejemplo muestra cómo se puede usar Twilio para realizar una llamada desde una página web PHP hospedada en Azure. La aplicación resultante le pedirá al usuario los valores de una llamada telefónica, como se muestra en la siguiente captura de pantalla.

![Formulario de llamada de Azure con Twilio y PHP][twilio_php]

Tendrá que hacer lo siguiente para utilizar el código de este tema:

1. Adquiera una cuenta de Twilio y un token de autenticación desde la [consola de Twilio][twilio_console]. Para empezar con Twilio, evalúe los precios en [https://www.twilio.com/pricing][twilio_pricing]. Puede registrarse para obtener una cuenta de evaluación en [https://www.twilio.com/try-twilio][try_twilio].
2. Instale la [biblioteca de Twilio para PHP](https://github.com/twilio/twilio-php) como paquete PEAR. Para obtener más información, consulte el [archivo Léame](https://github.com/twilio/twilio-php/blob/master/README.md).
3. Instale el SDK de Azure para PHP. 
<!-- For an overview of the SDK and instructions on installing it, see [Set up the Azure SDK for PHP](app-service-web/web-sites-php-mysql-deploy-use-git.md) -->

## <a name="create-a-web-form-for-making-a-call"></a>Creación de un formulario web para hacer una llamada
El código HTML siguiente muestra cómo crear una página web (**callform.html**) que recupera datos de usuarios para realizar una llamada:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Automated call form</title>
</head>
<body>
  <h1>Automated Call Form</h1>
  <p>Fill in all fields and click <b>Make this call</b>.</p>
  <form action="makecall.php" method="post">
    <table>
      <tr>
        <td>To:</td>
        <td><input name="callTo" size="50" type="text" value=""></td>
      </tr>
      <tr>
        <td>From:</td>
        <td><input name="callFrom" size="50" type="text" value=""></td>
      </tr>
      <tr>
        <td>Call message:</td>
        <td><input name="callText" size="100" type="text" value="Hello. This is the call text. Good bye."></td>
      </tr>
      <tr>
        <td colspan="2"><input type="submit" value="Make this call"></td>
      </tr>
    </table>
  </form><br>
</body>
</html>
```

## <a name="create-the-code-to-make-the-call"></a>Creación del código para realizar la llamada
El código siguiente muestra cómo crear **makecall.php**, que se llama cuando el usuario envía el formulario mostrado por **callform.html**. El código que se muestra a continuación crea el mensaje de llamada y genera la llamada. Además, asegúrese de usar la cuenta de Twilio y el token de autenticación de la [consola de Twilio][twilio_console] en lugar de los valores de marcador de posición asignados a **$sid** y **$token** en el código siguiente.

```html
<html>
<head><title>Making call...</title></head>
<body>
<p>Your call is being made.</p>

<?php
require_once 'path/to/vendor/autoload.php';

$sid   = "your_account_sid";
$token = "your_authentication_token";

$from_number = $_POST['callFrom']; // Calls must be made from a registered Twilio number.
$to_number   = $_POST['callTo'];
$message     = $_POST['callText'];

$client = new Twilio\Rest\Client($sid, $token);

$call = $client->calls->create(
            $to_number,
            $from_number,
            array('url' => https://twimlets.com/message?Message=' . urlencode($message))
        );

echo "Call status: " . $call->status . "<br />";
echo "URI resource: " . $call->uri . "<br />";
?>
</body>
</html>
```

Además de realizar la llamada, **makecall.php** muestra algunos metadatos, como se muestra en la imagen siguiente. Para obtener más información sobre los metadatos de llamada, vea [https://www.twilio.com/docs/api/rest/call#instance-properties][twilio_call_properties].

![Respuesta de llamada de Azure con Twilio y PHP][twilio_php_response]

## <a name="run-the-application"></a>Ejecución de la aplicación
El paso siguiente es [implementar la aplicación en Azure Web Apps con Git](app-service/quickstart-php.md) (aunque no toda la información que aparece es relevante). 

## <a name="next-steps"></a>Pasos siguientes
Este código se ha proporcionado para mostrar la funcionalidad básica usando Twilio en PHP en Azure. Antes de implementarlo en Azure en producción, es posible que desee agregar más controles de errores u otras características. Por ejemplo:

* En lugar de usar un formulario web, puede usar blobs de Azure Storage o una instancia de SQL Database para almacenar los números de teléfono y el texto de llamada. Para obtener información sobre el uso de blobs de almacenamiento de Azure en PHP, vea [Uso de Azure Storage con aplicaciones de PHP][howto_blob_storage_php]. Para obtener información sobre el uso de SQL Database en PHP, vea [Uso de SQL Database con aplicaciones de PHP][howto_sql_azure_php].
* El código de **makecall.php** usa una dirección URL proporcionada por Twilio ([https://twimlets.com/message][twimlet_message_url]) para proporcionar una respuesta de lenguaje de marcado de Twilio (TwiML) que indica a Twilio cómo proceder con la llamada. Por ejemplo, la TwiML que se devuelve puede contener un verbo `<Say>` que se traduce en el texto que se está hablando con el destinatario de la llamada. En lugar de usar la dirección URL proporcionada por Twilio, puede crear su propio servicio para responder a la solicitud de Twilio. Para obtener más información, vea [Uso de Twilio para capacidades de voz y SMS en PHP][howto_twilio_voice_sms_php]. Puede encontrar más información sobre TwiML en [https://www.twilio.com/docs/api/twiml][twiml] y sobre `<Say>` y otros verbos de Twilio en [https://www.twilio.com/docs/api/twiml/say][twilio_say].
* Lea las directrices de seguridad de Twilio en [https://www.twilio.com/docs/security][twilio_docs_security].

Para obtener más información sobre Twilio, vea [https://www.twilio.com/docs][twilio_docs].

## <a name="see-also"></a>Consulte también
* [Uso de Twilio para capacidades de voz y SMS en PHP](partner-twilio-php-how-to-use-voice-sms.md)

[twilio_console]: https://www.twilio.com/console
[twilio_pricing]: https://www.twilio.com/pricing
[try_twilio]: https://www.twilio.com/try-twilio
[twilio_api]: https://www.twilio.com/docs/api
[verify_phone]: https://www.twilio.com/console/phone-numbers/verified
[twimlet_message_url]: https://twimlets.com/message
[twiml]: https://www.twilio.com/docs/api/twiml
[twilio_api_service]: https://api.twilio.com
[build_php_azure_app]: http://azurephp.interoperabilitybridges.com/articles/build-and-deploy-a-windows-azure-php-application
[howto_twilio_voice_sms_php]: partner-twilio-php-how-to-use-voice-sms.md
[howto_blob_storage_php]: https://azure.microsoft.com/documentation/articles/storage-php-how-to-use-blobs/
[howto_sql_azure_php]: https://azure.microsoft.com/documentation/articles/sql-database-php-how-to-use/
[twilio_call_properties]: https://www.twilio.com/docs/api/rest/call#instance-properties
[twilio_docs_security]: https://www.twilio.com/docs/security
[twilio_docs]: https://www.twilio.com/docs
[twilio_say]: https://www.twilio.com/docs/api/twiml/say
[ssl_validation]: http://readthedocs.org/docs/twilio-php/en/latest/usage/rest.html
[twilio_php]: ./media/partner-twilio-php-make-phone-call/WA_TwilioPHPCallForm.jpg
[twilio_php_response]: ./media/partner-twilio-php-make-phone-call/WA_TwilioPHPMakeCall.jpg
[twilio_php_github]: https://github.com/twilio/twilio-php
