---
title: 'Importación de expresiones con Node.js: LUIS'
titleSuffix: Azure Cognitive Services
description: Aprenda a compilar una aplicación de LUIS mediante programación a partir de datos preexistentes en formato CSV usando la API de creación de LUIS.
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18, devx-track-javascript
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 09/05/2019
ms.author: diberry
ms.openlocfilehash: 669c5f1ae6b667073e885a0077e9f3130ba3c4f0
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87405983"
---
# <a name="build-a-luis-app-programmatically-using-nodejs"></a>Compilar una aplicación de LUIS mediante programación con Node.js

LUIS proporciona una API de programación que hace lo mismo que el sitio web de [LUIS](luis-reference-regions.md). Esto puede ahorrarle tiempo si tiene datos preexistentes. Además, resultaría más rápido crear una aplicación de LUIS mediante programación que si especifica la información manualmente.

[!INCLUDE [Waiting for LUIS portal refresh](./includes/wait-v3-upgrade.md)]

## <a name="prerequisites"></a>Prerequisites

* Inicie sesión en el sitio web de [LUIS](luis-reference-regions.md) y busque la [clave de creación](luis-how-to-azure-subscription.md#authoring-key) en la configuración de la cuenta. Esta clave se usa para llamar a las API de creación.
* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
* Este artículo empieza con un archivo CSV para los archivos de registro de solicitudes de usuario de una empresa hipotética. Descárguela [aquí](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/IoT.csv).
* Instale el Node.js más reciente con NPM. Puede descargarlo [aquí](https://nodejs.org/en/download/).
* **[Recomendado]** Visual Studio Code para IntelliSense y depuración. Descárguelo gratis [aquí](https://code.visualstudio.com/).

Todo el código de este artículo está disponible en el [repositorio Azure-Samples de Language Understanding en GitHub](https://github.com/Azure-Samples/cognitive-services-language-understanding/tree/master/examples/build-app-programmatically-csv).

## <a name="map-preexisting-data-to-intents-and-entities"></a>Asignar datos preexistentes a intenciones y entidades
Aunque tenga en mente un sistema que no se ha creado con LUIS, si contiene datos textuales que se asignan a cosas diferentes que quieren hacer los usuarios, podría crear en LUIS una asignación de las categorías existentes de entradas de usuario a las intenciones. Si puede identificar palabras o frases importantes en lo que han dicho los usuarios, se pueden asignar a entidades.

Abra el archivo [`IoT.csv`](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/IoT.csv). Contiene un registro de consultas de usuario a un servicio hipotético de domótica, así como su clasificación, lo que ha dicho el usuario y algunas columnas con información útil que se ha extraído de estas.

![Archivo CSV de datos existentes](./media/luis-tutorial-node-import-utterances-csv/csv.png)

Verá que la columna **RequestType** podrían ser intenciones. En la columna **Request** (Solicitud) se muestra una expresión de ejemplo. Los demás campos podrían ser entidades si aparecen en la expresión. Dado que hay intenciones, entidades y expresiones de ejemplo, tiene los requisitos de una aplicación de ejemplo sencilla.

## <a name="steps-to-generate-a-luis-app-from-non-luis-data"></a>Pasos para generar una aplicación de LUIS a partir de datos que no son de LUIS
Para generar una nueva aplicación de LUIS desde el archivo CSV:

* Analice los datos del archivo CSV:
    * Convierta a un formato que pueda cargar en LUIS mediante la API de creación.
    * De los datos analizados, recopile información sobre las intenciones y entidades.
* Cree llamadas API de creación para:
    * Crear la aplicación.
    * Agregue las intenciones y entidades que se recopilaron a partir de los datos analizados.
    * Una vez que haya creado la aplicación de LUIS, puede agregar las expresiones de ejemplo a partir de los datos analizados.

Puede ver el flujo de este programa en la última parte del archivo `index.js`. Copie o [descargue](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/index.js) este código y guárdelo en `index.js`.

   [!code-javascript[Node.js code for calling the steps to build a LUIS app](~/samples-luis/examples/build-app-programmatically-csv/index.js)]


## <a name="parse-the-csv"></a>Analizar el archivo CSV

Las entradas de columna que contienen las expresiones en el archivo CSV deben analizarse en un formato JSON que LUIS reconoce. Este formato JSON debe contener un campo `intentName` que identifique la intención de la expresión. También debe contener un campo `entityLabels`, que puede estar vacío si no hay ninguna entidad en la expresión.

Por ejemplo, la entrada de "Turn on the lights" (Encender las luces) se asigna a este archivo JSON:

```json
        {
            "text": "Turn on the lights",
            "intentName": "TurnOn",
            "entityLabels": [
                {
                    "entityName": "Operation",
                    "startCharIndex": 5,
                    "endCharIndex": 6
                },
                {
                    "entityName": "Device",
                    "startCharIndex": 12,
                    "endCharIndex": 17
                }
            ]
        }
```

En este ejemplo, `intentName` procede de la solicitud de usuario del encabezado de columna **Request** (Solicitud) del archivo CSV, mientras que `entityName` procede de las demás columnas con información clave. Por ejemplo, si hay una entrada para **Operation** (Operación) o **Device** (Dispositivo) y esa cadena también aparece en la solicitud real, se puede etiquetar como entidad. En el código siguiente se muestra este proceso de análisis. Puede copiarlo o [descargarlo](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_parse.js) y guardarlo en `_parse.js`.

   [!code-javascript[Node.js code for parsing a CSV file to extract intents, entities, and labeled utterances](~/samples-luis/examples/build-app-programmatically-csv/_parse.js)]



## <a name="create-the-luis-app"></a>Crear la aplicación de LUIS
Una vez analizados los datos en JSON, agréguelos a una aplicación de LUIS. El siguiente código crea la aplicación de LUIS. Cópielo o [descárguelo](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_create.js) y guárdelo en `_create.js`.

   [!code-javascript[Node.js code for creating a LUIS app](~/samples-luis/examples/build-app-programmatically-csv/_create.js)]


## <a name="add-intents"></a>Adición de intenciones
Cuando tenga una aplicación, debe incorporarle intenciones. El siguiente código crea la aplicación de LUIS. Cópielo o [descárguelo](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_intents.js) y guárdelo en `_intents.js`.

   [!code-javascript[Node.js code for creating a series of intents](~/samples-luis/examples/build-app-programmatically-csv/_intents.js)]


## <a name="add-entities"></a>agregar entidades
El código siguiente agrega las entidades a la aplicación de LUIS. Cópielo o [descárguelo](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_entities.js) y guárdelo en `_entities.js`.

   [!code-javascript[Node.js code for creating entities](~/samples-luis/examples/build-app-programmatically-csv/_entities.js)]



## <a name="add-utterances"></a>Adición de grabaciones de voz
Una vez que se hayan definido las entidades e intenciones en la aplicación de LUIS, puede agregar las expresiones. En el siguiente código se usa la API [Utterances_AddBatch](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c09), con la que se pueden agregar hasta 100 expresiones a la vez.  Cópielo o [descárguelo](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_upload.js) y guárdelo en `_upload.js`.

   [!code-javascript[Node.js code for adding utterances](~/samples-luis/examples/build-app-programmatically-csv/_upload.js)]


## <a name="run-the-code"></a>Ejecución del código


### <a name="install-nodejs-dependencies"></a>Instalar las dependencias de Node.js
Instale las dependencias de Node.js de NPM en el terminal o en la línea de comandos.

```console
> npm install
```

### <a name="change-configuration-settings"></a>Cambiar la configuración
Para poder usar esta aplicación, debe cambiar los valores del archivo index.js por su clave de punto de conexión e indicar el nombre que quiere que tenga la aplicación. También puede establecer la referencia cultural de la aplicación o cambiar el número de versión.

Abra el archivo index.js y cambie estos valores en la parte superior del archivo.


```javascript
// Change these values
const LUIS_programmaticKey = "YOUR_AUTHORING_KEY";
const LUIS_appName = "Sample App";
const LUIS_appCulture = "en-us";
const LUIS_versionId = "0.1";
```

### <a name="run-the-script"></a>Ejecute el script.
Ejecute el script desde un terminal o línea de comandos con Node.js.

```console
> node index.js
```

or

```console
> npm start
```

### <a name="application-progress"></a>Progreso de la aplicación
Mientras se ejecuta la aplicación, en la línea de comandos se muestra el progreso. El resultado de la línea de comandos incluye el formato de las respuestas de LUIS.

```console
> node index.js
intents: ["TurnOn","TurnOff","Dim","Other"]
entities: ["Operation","Device","Room"]
parse done
JSON file should contain utterances. Next step is to create an app with the intents and entities it found.
Called createApp, created app with ID 314b306c-0033-4e09-92ab-94fe5ed158a2
Called addIntents for intent named TurnOn.
Called addIntents for intent named TurnOff.
Called addIntents for intent named Dim.
Called addIntents for intent named Other.
Results of all calls to addIntent = [{"response":"e7eaf224-8c61-44ed-a6b0-2ab4dc56f1d0"},{"response":"a8a17efd-f01c-488d-ad44-a31a818cf7d7"},{"response":"bc7c32fc-14a0-4b72-bad4-d345d807f965"},{"response":"727a8d73-cd3b-4096-bc8d-d7cfba12eb44"}]
called addEntity for entity named Operation.
called addEntity for entity named Device.
called addEntity for entity named Room.
Results of all calls to addEntity= [{"response":"6a7e914f-911d-4c6c-a5bc-377afdce4390"},{"response":"56c35237-593d-47f6-9d01-2912fa488760"},{"response":"f1dd440c-2ce3-4a20-a817-a57273f169f3"}]
retrying add examples...

Results of add utterances = [{"response":[{"value":{"UtteranceText":"turn on the lights","ExampleId":-67649},"hasError":false},{"value":{"UtteranceText":"turn the heat on","ExampleId":-69067},"hasError":false},{"value":{"UtteranceText":"switch on the kitchen fan","ExampleId":-3395901},"hasError":false},{"value":{"UtteranceText":"turn off bedroom lights","ExampleId":-85402},"hasError":false},{"value":{"UtteranceText":"turn off air conditioning","ExampleId":-8991572},"hasError":false},{"value":{"UtteranceText":"kill the lights","ExampleId":-70124},"hasError":false},{"value":{"UtteranceText":"dim the lights","ExampleId":-174358},"hasError":false},{"value":{"UtteranceText":"hi how are you","ExampleId":-143722},"hasError":false},{"value":{"UtteranceText":"answer the phone","ExampleId":-69939},"hasError":false},{"value":{"UtteranceText":"are you there","ExampleId":-149588},"hasError":false},{"value":{"UtteranceText":"help","ExampleId":-81949},"hasError":false},{"value":{"UtteranceText":"testing the circuit","ExampleId":-11548708},"hasError":false}]}]
upload done
```




## <a name="open-the-luis-app"></a>Abra la aplicación de LUIS
Una vez concluido el script, puede iniciar sesión en [LUIS](luis-reference-regions.md) y ver la aplicación de LUIS que ha creado en **Mis aplicaciones**. Debería poder ver las expresiones que ha agregado en las intenciones **TurnOn**, **TurnOff** y **None**.

![Intención TurnOn](./media/luis-tutorial-node-import-utterances-csv/imported-utterances-661.png)


## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Probar y entrenar la aplicación en el sitio web de LUIS](luis-interactive-test.md)

## <a name="additional-resources"></a>Recursos adicionales

En esta aplicación de ejemplo se usan las siguientes API de LUIS:
- [create app](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c36)
- [add intents](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c0c)
- [add entities](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c0e)
- [add utterances](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c09)
