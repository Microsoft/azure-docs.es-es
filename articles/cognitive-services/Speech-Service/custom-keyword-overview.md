---
title: Palabras clave personalizadas - Servicio de Voz
titleSuffix: Azure Cognitive Services
description: Introducción a las características, las funcionalidades y las restricciones de las palabras clave mediante el kit de desarrollo de software (SDK) de Voz.
services: cognitive-services
author: hasyashah
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/06/2020
ms.author: hasshah
ms.custom: devx-track-csharp
ms.openlocfilehash: 8c19650a8f36181e3e003414ff15add071e9a8ac
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88918900"
---
# <a name="what-is-a-keyword"></a>¿Qué es una palabra clave?

Una palabra clave es una palabra o frase corta que permite activar por voz el producto. Por ejemplo, "Hola Cortana" es la palabra clave para el asistente Cortana. La activación por voz permite que los usuarios empiecen a interactuar con su producto sin tener que usar las manos, simplemente diciendo la palabra clave. Como el producto siempre espera escuchar la palabra clave, todo el audio se procesa de manera local en el dispositivo del usuario hasta que se produce una detección para asegurarse de que sus datos sigan siendo lo más privados posible.

## <a name="core-features-of-custom-keyword"></a>Características principales de la Palabra clave personalizada

Con las características de personalización, rendimiento e integración de la Palabra clave personalizada, puede adecuar la activación por voz para que se adapte mejor a las necesidades de los usuarios y a la visión del producto.

| Característica | Descripción |
|----------|----------|
| Personalización de palabras clave | Como extensión de su marca, una palabra clave reafirma el patrimonio que ha creado con sus clientes. El portal de Palabra clave personalizada en Speech Studio le permite especificar cualquier palabra o frase corta que mejor represente su marca. Para personalizar aún más la palabra clave, elija las pronunciaciones correctas, que el modelo de palabra clave generado respetará.
| Comprobación de la palabra clave | Cuando se confía sin problemas en que la palabra clave se detectará de manera local, el audio se envía a la nube para comprobar aún más que un usuario dijo la palabra clave. Comprobar la palabra clave brinda una capa de seguridad adicional al reducir el impacto de una detección local incorrecta y proteger la privacidad de los usuarios.
| Integración del asistente de voz y el SDK de Voz | Las palabras clave generadas en la Palabra clave personalizada en Speech Studio se pueden integrar fácilmente en el dispositivo o la aplicación a través del SDK de Voz. Solo tiene que apuntar el SDK al modelo de palabras clave proporcionado por Speech Studio y el producto se activará por voz, con el respaldo de la comprobación de la palabra clave. Para completar las experiencias de voz del producto, puede crear su propio [Asistente de voz](voice-assistants.md).

## <a name="get-started-with-custom-keywords"></a>Introducción a las palabras clave personalizadas

* Consulte los [conceptos básicos de palabras clave personalizadas](custom-keyword-basics.md) para ver los patrones de uso y diseño básicos.
* Cómo [activar el producto con la voz mediante el SDK de voz, C#](tutorial-voice-enable-your-bot-speech-sdk.md)

## <a name="choose-an-effective-keyword"></a>Elección de una palabra clave eficaz

Crear una palabra clave efectiva es fundamental para garantizar que el dispositivo responda de forma coherente y precisa. La personalización de la palabra clave es una forma eficaz de diferenciar el dispositivo y de reforzar la marca. Tenga en cuenta las siguientes directrices al elegir una palabra clave:

> [!div class="checklist"]
> * La palabra clave debe ser una palabra o frase en inglés.
> * No debería tardar más de dos segundos en decirse.
> * Las palabras de 4 a 7 sílabas funcionan mejor. Por ejemplo, "Hey, Computer" es una buena palabra clave, mientras que "Hey" no es adecuada.
> * Las palabras clave deben seguir las reglas comunes de pronunciación en inglés.
> * Una palabra única o incluso inventada que siga las reglas comunes de pronunciación en inglés puede reducir los falsos positivos. Por ejemplo, "computerama" puede ser una buena palabra clave.
> * No elija una palabra común. Por ejemplo, "eat" y "go" son palabras que la gente dice con frecuencia en una conversación ordinaria. Pueden ser desencadenadores falsos para su dispositivo.
> * Evite usar una palabra clave que pueda tener pronunciaciones alternativas. Los usuarios tendrían que saber la pronunciación "correcta" para que su dispositivo respondiera. Por ejemplo, "509" puede pronunciarse como "cinco cero nueve" o "quinientos nueve". "R.E.I." puede pronunciarse como "r-e-i" o "rey". "Live" puede pronunciarse como "/līv/" o "/liv/".
> * No utilice caracteres especiales, símbolos o dígitos. Por ejemplo, "Go#" y "20 + cats" podrían ser palabras clave problemáticas. Sin embargo, "go sharp" o "twenty plus cats" pueden funcionar. Todavía puede utilizar los símbolos en su marca y utilizar el marketing y la documentación para reforzar la pronunciación adecuada.

> [!NOTE]
> Si elige una palabra registrada como marca comercial como palabra clave, asegúrese de que es el propietario de esa marca comercial, o bien de tener el permiso del propietario para usar la palabra. Microsoft no es responsable de ningún problema legal que pueda surgir de su elección de la palabra clave.

## <a name="see-samples-on-github"></a>Consulte los ejemplos en GitHub

* [Reconocimiento de palabras clave con el SDK de voz, en la Plataforma universal de Windows mediante C#](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/csharp/uwp/keyword-recognizer)
* [Reconocimiento de palabras clave con el SDK de voz, en Android mediante Java](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/java/android/keyword-recognizer)

## <a name="next-steps"></a>Pasos siguientes

* [Obtenga una clave de suscripción gratuita a los servicios de Voz](get-started.md)
* [Obtención del SDK de voz](speech-sdk.md)
* [Más información sobre los Asistentes de voz](voice-assistants.md)
