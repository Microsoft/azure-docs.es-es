---
title: Reconocimiento de intenciones a partir de contenido de voz mediante el SDK de Voz para C#
titleSuffix: Azure Cognitive Services
description: En esta guía aprenderá a reconocer intenciones a partir de contenido de voz con el SDK de Voz para C#.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 02/10/2020
ms.author: trbye
ms.custom: devx-track-csharp
ms.openlocfilehash: 1138a970bf7c52182f13d0fd14d0178a2d0cfeba
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88918798"
---
# <a name="how-to-recognize-intents-from-speech-using-the-speech-sdk-for-c"></a>Reconocimiento de intenciones a partir de contenido de voz mediante el SDK de Voz para C#

El [SDK de Voz](speech-sdk.md) de Cognitive Services se integra con [Language Understanding Intelligent Service (LUIS)](https://www.luis.ai/home) para proporcionar un **reconocimiento de la intención.** Una intención es algo que el usuario quiere hacer: reservar un vuelo, comprobar el tiempo o hacer una llamada. El usuario puede utilizar cualquier término que le parezca natural. Mediante el aprendizaje automático, LUIS asigna las solicitudes de los usuarios a las intenciones que se hayan definido.

> [!NOTE]
> Una aplicación LUIS define las intenciones y entidades que desea reconocer. Es independiente de la aplicación C# que utiliza el servicio Voz. En este artículo, con la palabra "aplicación" se indica tanto la aplicación LUIS, como el código C#.

En esta guía, se utiliza el SDK de Voz para desarrollar una aplicación de consola C# que deriva las intenciones de las expresiones de los usuarios mediante el micrófono del dispositivo. Aprenderá a:

> [!div class="checklist"]
>
> - Crear un proyecto de Visual Studio que haga referencia al paquete NuGet del SDK de Voz
> - Crear una configuración de voz y obtener un reconocedor de intenciones
> - Obtener el modelo para la aplicación LUIS y agregar las intenciones que necesita
> - Especificar el idioma para el reconocimiento de voz
> - Reconocer la voz a partir de un archivo
> - Usar el reconocimiento asincrónico, continuo y controlado por eventos

## <a name="prerequisites"></a>Prerrequisitos

Asegúrese de disponer de los siguientes elementos antes de empezar esta guía:

- Una cuenta de LUIS. Puede obtener una gratis mediante el [portal de LUIS](https://www.luis.ai/home).
- [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) (cualquier edición).

## <a name="luis-and-speech"></a>LUIS y voz

LUIS se integra con el servicio Voz para reconocer las intenciones a partir de contenido de voz. No necesita una suscripción al servicio Voz, solo LUIS.

LUIS usa tres tipos de claves:

| Tipo de clave  | Propósito                                               |
| --------- | ----------------------------------------------------- |
| Creación | Le permite crear y modificar aplicaciones de LUIS mediante programación |
| Inicio   | Permite probar la aplicación de LUIS mediante el uso solo de texto.   |
| Punto de conexión  | Autoriza el acceso a una aplicación de LUIS concreta            |

Para esta guía, necesitará el tipo de clave de punto de conexión. Esta guía utiliza la aplicación de LUIS Home Automation de ejemplo, que se puede crear siguiendo el inicio rápido [Uso de automatización del hogar compilada previamente](https://docs.microsoft.com/azure/cognitive-services/luis/luis-get-started-create-app). Si ha creado su propia aplicación de LUIS, puede usarla si lo prefiere.

Al crear una aplicación LUIS, el propio LUIS genera automáticamente una clave de inicio para que pueda probarla aplicación mediante consultas de texto. Esta clave no permite la integración del servicio de voz y no funcionará con esta guía. Cree un recurso de LUIS en el panel de Azure y asígnelo a la aplicación de LUIS. Puede usar el nivel de suscripción gratis para esta guía.

Después de crear el recurso de LUIS en el panel de Azure, inicie sesión en el [portal de LUIS](https://www.luis.ai/home), elija la aplicación en la página **My Apps** (Mis aplicaciones) y, después, cambie a la página **Manage** (Administrar) de la aplicación. Por último, seleccione **Keys and Endpoints** (Claves y puntos de conexión) en la barra lateral.

![Configuración de los puntos de conexión y claves del portal de LUIS](media/sdk/luis-keys-endpoints-page.png)

En la página **Keys and Endpoint settings** (Configuración de claves y puntos de conexión):

1. Desplácese hacia abajo hasta la sección **Resources and Keys** (Recursos y claves) y seleccione **Assign resource** (Asignar recurso).
1. En el cuadro de diálogo **Assign a key to your app** (Asignar una clave a la aplicación), realices los siguientes cambios:

   - En **Tenant** (Inquilino), elija **Microsoft**.
   - En **Subscription Name** (Nombre de suscripción), elija la suscripción de Azure que contiene el recurso de LUIS que desea usar.
   - En **Key** (Clave), elija el recurso de LUIS que desea usar con la aplicación.

   En unos instantes, la nueva suscripción aparecerá en la tabla en la parte inferior de la página.

1. Seleccione el icono situado junto a una clave para copiarla al Portapapeles. (Puede usar cualquiera de las claves).

![Claves de suscripción de la aplicación LUIS](media/sdk/luis-keys-assigned.png)

## <a name="create-a-speech-project-in-visual-studio"></a>Creación de un proyecto de contenido de voz en Visual Studio

[!INCLUDE [Create project](../../../includes/cognitive-services-speech-service-create-speech-project-vs-csharp.md)]

## <a name="add-the-code"></a>Adición del código

Después, agregue al código al proyecto.

1. En el **Explorador de soluciones**, abra el archivo **Program.cs**.

1. Reemplace el bloque de sentencias `using` del principio del archivo por las siguientes declaraciones:

   [!code-csharp[Top-level declarations](~/samples-cognitive-services-speech-sdk/samples/csharp/sharedcontent/console/intent_recognition_samples.cs#toplevel)]

1. Reemplace el método `Main()` proporcionado por el siguiente equivalente asincrónico:

   ```csharp
   public static async Task Main()
   {
       await RecognizeIntentAsync();
       Console.WriteLine("Please press Enter to continue.");
       Console.ReadLine();
   }
   ```

1. Cree un método asincrónico vacío `RecognizeIntentAsync()`, tal como se muestra aquí:

   ```csharp
   static async Task RecognizeIntentAsync()
   {
   }
   ```

1. En el cuerpo del nuevo método, agregue este código:

   [!code-csharp[Intent recognition by using a microphone](~/samples-cognitive-services-speech-sdk/samples/csharp/sharedcontent/console/intent_recognition_samples.cs#intentRecognitionWithMicrophone)]

1. Reemplace los marcadores de posición en este método por su clave de suscripción, región e identificador de aplicación de LUIS, como se indica a continuación.

   | Marcador de posición | Reemplazar por |
   | ----------- | ------------ |
   | `YourLanguageUnderstandingSubscriptionKey` | La clave de punto de conexión de LUIS. Una vez más, este elemento se debe obtener en el panel de Azure, no una "clave de inicio". Se puede encontrar en la página **Keys and Endpoints** (Claves y puntos de conexión), (en **Manage [Administrar]** ) en el [portal de LUIS](https://www.luis.ai/home). |
   | `YourLanguageUnderstandingServiceRegion` | El identificador corto para la región en la que se encuentra la suscripción a LUIS, como `westus` para Oeste de EE. UU. Consulte [Regiones](regions.md). |
   | `YourLanguageUnderstandingAppId` | El id. de la aplicación LUIS. Se puede encontrar en la página **Settings** (Configuración) del [portal de LUIS](https://www.luis.ai/home). |

Con estos cambios realizados, puede crear (**Control-Mayús-B**) y ejecutar (**F5**) la aplicación de la guía. Cuando se le solicite, pruebe a decir "Apagar las luces" al micrófono del equipo. La aplicación muestra el resultado en la ventana de la consola.

Las secciones siguientes incluyen una explicación del código.

## <a name="create-an-intent-recognizer"></a>Creación de un reconocedor de intenciones

En primer lugar, es preciso crear una configuración de voz desde la región y la clave del punto de conexión de LUIS. Las configuraciones de voz se pueden utilizar para crear reconocedores para las distintas funcionalidades del SDK de Voz. La configuración de voz tiene varias formas de especificar la suscripción que se desea usar; aquí usamos `FromSubscription`, que toma la clave de suscripción y la región.

> [!NOTE]
> Utilice la clave y la región de su suscripción a LUIS, no de su suscripción al servicio de voz.

Después, cree un reconocedor de intenciones mediante `new IntentRecognizer(config)`. Dado que la configuración ya sabe la suscripción que hay que utilizar, no es preciso volver a especificar la clave de suscripción y el punto de conexión al crear el reconocedor.

## <a name="import-a-luis-model-and-add-intents"></a>Importación de un modelo de LUIS y adición de intenciones

Ahora importe el modelo desde la aplicación LUIS mediante `LanguageUnderstandingModel.FromAppId()` y agregue las intenciones de LUIS que desee reconocer mediante el método `AddIntent()` del reconocedor. Estos dos pasos mejoran la precisión del reconocimiento de voz al indicar palabras que el usuario probablemente utilizará en sus solicitudes. No es preciso agregar todas las intenciones de la aplicación, salvo que se necesite reconocer todas en la aplicación.

Para agregar intenciones, es preciso especificar tres argumentos: el modelo de LUIS (que se ha creado y se llama `model`), el nombre de la intención y un identificador de la intención. La diferencia entre el identificador y el nombre es como sigue.

| Argumento de `AddIntent()`&nbsp; | Propósito |
| --------------------------- | ------- |
| `intentName` | El nombre de la intención, tal como se define en la aplicación LUIS. Este valor debe coincidir exactamente con el nombre de la intención de LUIS. |
| `intentID` | Identificador asignado a una intención reconocida por el SDK de Voz. Este valor puede ser el que desee; no es preciso que se corresponda con el nombre de la intención definido en la aplicación LUIS. Si se controlan varias intenciones con el mismo código, por ejemplo, se puede utilizar el mismo identificador para ellos. |

La aplicación de LUIS Home Automation tiene dos intenciones: una para encender un dispositivo y otra para apagar un dispositivo. En las líneas siguientes se agregan estas intenciones al reconocedor; reemplace las tres líneas `AddIntent` del método `RecognizeIntentAsync()` por este código.

```csharp
recognizer.AddIntent(model, "HomeAutomation.TurnOff", "off");
recognizer.AddIntent(model, "HomeAutomation.TurnOn", "on");
```

En lugar de agregar intenciones individuales, puede usar el método `AddAllIntents` para agregar todas las intenciones de un modelo al reconocedor.

## <a name="start-recognition"></a>Inicio del reconocimiento

Cuando haya creado el reconocedor y haya agregado las intenciones, puede empezar el reconocimiento. El SDK de Voz admite tanto el reconocimiento continuo como el de una sola emisión.

| Modo de reconocimiento | Métodos para la llamada | Resultado |
| ---------------- | --------------- | ------ |
| Emisión única | `RecognizeOnceAsync()` | Devuelve la intención reconocida, si la hubiera, después de una expresión. |
| Continuo | `StartContinuousRecognitionAsync()`<br>`StopContinuousRecognitionAsync()` | Reconoce varias expresiones; emite eventos (por ejemplo, `IntermediateResultReceived`) cuando los resultados están disponibles. |

La aplicación utiliza el modo de emisión única y, por lo tanto, llama a `RecognizeOnceAsync()` para iniciar el reconocimiento. El resultado es un objeto `IntentRecognitionResult` que contiene información sobre la intención reconocida. La respuesta JSON de LUIS se extrae mediante la siguiente expresión:

```csharp
result.Properties.GetProperty(PropertyId.LanguageUnderstandingServiceResponse_JsonResult)
```

La aplicación no analiza el resultado de JSON. Solo muestra el texto JSON en la ventana de la consola.

![Resultados individuales del reconocimiento de LUIS](media/sdk/luis-results.png)

## <a name="specify-recognition-language"></a>Especificación de un idioma de reconocimiento

De forma predeterminada, LUIS reconoce las intenciones en idioma inglés de Estados Unidos (`en-us`). Al asignar un código de configuración regional a la propiedad `SpeechRecognitionLanguage` de la configuración de voz, puede reconocer las intenciones en otros idiomas. Por ejemplo, agregue `config.SpeechRecognitionLanguage = "de-de";` en la aplicación antes de crear el reconocedor para reconocer las intenciones en idioma alemán. Para obtener más información, consulte [Idiomas admitidos: LUIS](../LUIS/luis-language-support.md#languages-supported).

## <a name="continuous-recognition-from-a-file"></a>Reconocimiento continuo desde un archivo

El código siguiente muestra dos funcionalidades adicionales de reconocimiento de intenciones mediante el SDK de Voz. La primera, mencionada anteriormente, es el reconocimiento continuo, donde el reconocedor emite eventos cuando los resultados están disponibles. Estos eventos los pueden procesar los controladores de eventos que proporcione. Con el reconocimiento continuo, se llama al método `StartContinuousRecognitionAsync()` del reconocedor para iniciar el reconocimiento, en lugar de a `RecognizeOnceAsync()`.

La otra funcionalidad es leer el audio que contiene la voz que se va a procesar desde un archivo WAV. La implementación implica la creación de una configuración de audio que se puede utilizar al crear el reconocedor de intenciones. El archivo debe ser monocanal (mono) con una frecuencia de muestreo de 16 kHz.

Para probar estas características, elimine o marque como comentario el cuerpo del método `RecognizeIntentAsync()` y agregue el siguiente código en su lugar.

[!code-csharp[Intent recognition by using events from a file](~/samples-cognitive-services-speech-sdk/samples/csharp/sharedcontent/console/intent_recognition_samples.cs#intentContinuousRecognitionWithFile)]

Revise el código para incluir la clave del punto de conexión de LUIS, la región y el identificador de la aplicación y para agregar las intenciones de automatización de dispositivos del hogar, como antes. Cambie `whatstheweatherlike.wav` por el nombre del archivo de audio grabado. Luego realice la compilación, copie el archivo de audio en el directorio de compilación y ejecute la aplicación.

Por ejemplo, si dice "Apagar las luces", hace una pausa y, después, dice "Encender las luces" en el archivo de audio grabado, puede aparecer una salida en la consola similar a la siguiente:

![Resultados del reconocimiento que hace LUIS de un archivo de audio](media/sdk/luis-results-2.png)

[!INCLUDE [Download the sample](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]
Busque el código de este artículo en la carpeta **samples/csharp/sharedcontent/console**.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Inicio rápido: Reconocimiento de voz a través de un micrófono](~/articles/cognitive-services/Speech-Service/quickstarts/speech-to-text-from-microphone.md?pivots=programming-language-csharp&tabs=dotnetcore)
