---
title: Learn about Language Understanding Intelligent Service (LUIS) in Azure | Microsoft Docs
description: Learn how to use Language Understanding Intelligent Service (LUIS) to bring the power of machine learning to your applications.
services: cognitive-services
author: cahann
manager: hsalama
ms.service: cognitive-services
ms.technology: luis
ms.topic: article
ms.date: 03/01/2017
ms.author: cahann
ms.translationtype: Human Translation
ms.sourcegitcommit: a30a90682948b657fb31dd14101172282988cbf0
ms.openlocfilehash: 7c1885da0aed2e8b21f028a4366b667b0b6b5166
ms.contentlocale: es-es
ms.lasthandoff: 05/25/2017

---

# <a name="learn-about-language-understanding-intelligent-service-luis"></a>Información acerca de Language Understanding Intelligent Service (LUIS)

Uno de los problemas principales en las interacciones entre un humano y PC es el poder hacer entender a una computadora lo que una persona necesita. Language Understanding Intelligent Service (LUIS) ['Lenguaje Comprensión del Servicio Inteligente' por sus siglas en inglés] permite que los desarrolladores puedan crear aplicaciones inteligentes que puedan entender el lenguaje humano y reaccionen de acuerdo a las peticiones del usuario. LUIS usa el poder de Machine Learning para resolver la dificultad de extraer el significado de lo ingresado por lenguaje natural, y de esta forma su aplicación no lo tendría que realizar. Cualquier aplicación que converse con usuarios, como un sistema de diálogos o un chat bot, puede pasar la información ingresada a una app LUIS y recibir resultados que proporcionan la comprensión del lenguaje.

## <a name="what-is-a-luis-app"></a>¿Qué es una app de LUIS?

Una aplicación LUIS es el lugar para que el desarrollador defina un modelo de lenguaje customizado. La salida de una aplicación LUIS es un servicio web con un enpoint HTTP que hace referencia a la aplicación cliente para agregar comprensión del lenguaje natural a la misma. Una aplicación LUIS toma un enunciado del usuario y extrae **Intentos** y **Entidades** que corresponden a las actividades en la lógica de la aplicación cliente. La aplicación cliente puede tomar la acción adecuada basándose en los intentos del usuario que LUIS reconoce.

![LUIS recognizes user intent](./media/luis-overview/luis-overview-process.png)

## <a name="key-concepts"></a>Conceptos clave

* **¿Qué es un enunciado?** Un enunciado en la entrada textual del usuario, que su aplicación necesita interpretar. Puede ser una oración como "Resérvame un boleto a París", o un fragmento de una oración, como "Reservación" o "Vuelo a París". Los enunciados no siempre serán formadas correctamente y se pueden tener varios enunciados para un intento en particular. Ver [Agregando un ejemplo de enunciado][add-example-utterances] para información sobre cómo entrenar a la aplicación LUIS para que pueda entender los enunciados del usuario.
* **¿Qué son los intentos?** Los intentos son como los verbos de una oración. Un intento representa acciones que el usuario quiere realizar. Es un propósito o meta expresada por la entrada textual del usuario, como reservación de un vuelo, pagar una factura, o encontrar noticias. Puede definir un conjunto de intentos correspondientes a acciones que el usuario desea realizar en la aplicación. Una aplicación de viajes puede definir el intento "ReservaVuelo" para que LUIS pueda extraer del enunciado "Resérvame un boleto a París". 
* **¿Qué son las entidades?** Si los intentos son verbos, entonces las entidades son sustantivos. Una entidad representa la instancia de una clase de objeto que es relevante para el intento del usuario. En el enunciado "Resérvame un boleto a París", "Paris" es una entidad de tipo ubicación. Al reconocer las entidades que son mencionadas por la entrada del usuario, LUIS ayuda en escoger las acciones que debe tomar para cumplir un intento. Vea [Entidades en LUIS](luis-concept-entity-types.md) para más detalles en los tipos de entidades que LUIS provee. 

## <a name="plan-your-luis-app"></a>Planear la aplicación LUIS
Antes de comenzar creando la interfaz web de LUIS, planifique su aplicación LUIS boceto de un esquema o un esquema para describir los intentos y entidades en su aplicación. Generalmente, crea los intentaos para reaccionar a una acción en la aplicación cliente o bot y crea una entidad para modelar algunos parámetros requeridos para ejecutar una acción. Por ejemplo, el intento "ReservaVuelo" puede reaccionar a una llamada en una API externa para la reservación de un boleto de avión, que requiere de las entidades como destino, fecha y aerolínea. Vea [Planifique su aplicación](Plan-your-app.md) para ejemplos y guías sobre cómo escoger los intentos y entidades para reflejar las funciones y relaciones en una aplicación.

## <a name="build-and-train-a-luis-app"></a>Construya y entrene su aplicación LUIS
Una vez que determino cuales intentos y entidades desea que reconozca su aplicación, puede añadirlos a la aplicación LUIS. Vea [crear una nueva app LUIS](LUIS-get-started-create-app.md), para un tutorial de la creación de una aplicación LUIS. 
Para más detalles sobre los pasos a seguir en la configuración de la aplicación LUIS, vea los siguientes artículos:
1.    [Agregar intentos](Add-intents.md)
2.  [Agregar enunciados](Add-example-utterances.md)
3.    [Add entidades](Add-entities.md)
4.  [Mejorar el performance usando características](Add-Features.md)
5.    [Entrenar y probar](Train-Test.md)
6.  [Usar aprendizaje activo](label-suggested-utterances.md)
7.    [Publicación](PublishApp.md)

También puede ver un [video tutorial](https://www.youtube.com/watch?v=jWeLajon9M8&index=4&list=PLD7HFcN7LXRdHkFBFu4stPPeWJcQ0VFLx) básico sobre los pasos anteriores.

## <a name="improve-performance-using-active-learning"></a>Mejora el performance usando aprendizaje activo
Si la aplicación ya fue desplegada y el tráfico comienza a fluir, LUIS usa aprendizaje activo para mejorarse por si solo. En el proceso de aprendizaje activo, LUIS identifica los enunciados que no está seguro y pide el etiquetado manual de acuerdo al intento y las entidades. Este proceso tiene demasiadas ventajas. LUIS conoce cuales enunciados no están claros, y eso ayuda en el mejoramiento máximo del sistema. LUIS aprende rápido y requiere una mínima cantidad de tiempo y esfuerzo. Es un servicio de Machine Learning activo. Ver [Etiquetado sugerido en enunciados][label-suggested-utterances] para una explicación sobre cómo implementar un aprendizaje activo usando una interfaz web para LUIS.

## <a name="configure-luis-programmatically"></a>Configura LUIS mediante programación
LUIS ofrece una cantidad de REST APIs que pueden ser usados por desarrolladores para automatización del proceso de desarrollo. Estas APIs permiten crear, entrenar y publicar la aplicación.

* [APIs de LUIS](https://dev.projectoxford.ai/docs/services/56d95961e597ed0f04b76e58/operations/5739a8c71984550500affdfa).

## <a name="integrate-luis-with-a-bot"></a>Integra LUIS en un bot
Es fáicl usar LUIS en un bot construido usando [Bot Framework](https://docs.microsoft.com/bot-framework/), que proporciona un SDK de creación de Bots para Node.js o .NET. Pudes referenciar la aplicación LUIS como se muestra en el siguiente ejemplo:

#### <a name="nodejs"></a>Node.js 
```javascript
// Agregar un reconocedor de LUIS en el bot usando la URL del enpoint en tu aplciacion LUIS.
var model = 'https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/2c2afc3e-5f39-4b6f-b8ad-c47ce1b98d8a?subscription-key=9823b65a8c9045f8bce7fee87a5e1fbc';
bot.recognizer(new builder.LuisRecognizer(model));
```

#### <a name="c"></a>C#
```cs
    // El atributo LuisModel especifica tu app ID y la llave de suscripcion de la app LUIS
    [LuisModel("2c2afc3e-5f39-4b6f-b8ad-c47ce1b98d8a", "9823b65a8c9045f8bce7fee87a5e1fbc")]
    [Serializable]
    public class TravelGuidDialog : LuisDialog<object>
    {
      // ...
```

El SDK de Bot Builder proporciona clases que automáticamente maneja los intentos y entidades que se regresan desde la aplicación de LUIS. Para ejemplos de cómo usar dichas clases, se pueden consultar los siguientes ejemplos:


*    [LUIS demo bot (C#)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-LUIS)
*    [LUIS demo bot (Node.js)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-LUIS) 


## <a name="integrate-luis-with-speech"></a>Integra LUIS con Speech
Los endpoints de LUIS funcionan perfectamente con el servicio de Speech de Microsoft Cognitive Services para reconocimiento de voz. En el SDK de la API de Speech para C# puedes agregar el ID de la aplicación y la llave de suscripción, y el resultado del reconocimiento de voz es enviado a LUIS para su interpretación.

Vea [Información general de la API de Speech de Microsoft Cognitive Services](../Speech/Home.md).

<!-- Reference-style links -->
[add-example-utterances]: https://docs.microsoft.com/azure/cognitive-services/luis/add-example-utterances
[pre-built-entities]: https://docs.microsoft.com/azure/cognitive-services/luis/pre-builtentities
[label-suggested-utterances]: label-suggested-utterances.md

<!-- this link not working 5/8 -->
[cs-speech-service]: https://www.microsoft.com/cognitive-services/speech-api
