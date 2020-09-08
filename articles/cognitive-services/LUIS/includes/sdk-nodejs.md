---
title: archivo de inclusión
description: archivo de inclusión
services: cognitive-services
author: diberry
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.date: 09/01/2020
ms.topic: include
ms.custom: include file, devx-track-javascript, cog-serv-seo-aug-2020
ms.author: diberry
ms.openlocfilehash: b961bb0598a1f6e9cdc7b4df27af50e7af1d9d81
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89323108"
---
Use las bibliotecas de cliente de Language Understanding (LUIS) para Node.js para:

* Creación de una aplicación
* Agregar una intención, una entidad de aprendizaje automático, con una expresión de ejemplo
* Entrenamiento y publicación de la aplicación
* Consulta de runtime de predicción

[Documentación de referencia](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/?view=azure-node-latest) |  Código fuente de la biblioteca de [creación](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-luis-authoring) y [prediccion](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-luis-runtime) | NPM de [creación](https://www.npmjs.com/package/@azure/cognitiveservices-luis-authoring) y [predicción](https://www.npmjs.com/package/@azure/cognitiveservices-luis-runtime) | [Ejemplos](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/LUIS/sdk-3x/index.js)

## <a name="prerequisites"></a>Prerrequisitos

* [Node.js](https://nodejs.org)
* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* Cuando tenga la suscripción a Azure, [cree un recurso de creación de Language Understanding](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesLUISAllInOne) en Azure Portal para obtener la clave de creación y el punto de conexión. Espere a que se implemente y haga clic en el botón **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que [cree](../luis-how-to-azure-subscription.md#create-luis-resources-in-azure-portal) para conectar la aplicación a la creación de Language Understanding. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente. Puede usar el plan de tarifa gratuito (`F0`) para probar el servicio.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-javascript-application"></a>Creación de una aplicación de JavaScript

1. En una ventana de consola, cree un directorio para la aplicación y muévala a ese directorio.

    ```console
    mkdir quickstart-sdk && cd quickstart-sdk
    ```

1. Inicialice el directorio como una aplicación de JavaScript, para lo que debe crear un archivo `package.json`.

    ```console
    npm init -y
    ```

1. Cree un archivo denominado `index.js` para el código de JavaScript.

    ```console
    touch index.js
    ```

### <a name="install-the-npm-libraries"></a>Instalación de las bibliotecas de NPM

Dentro del directorio de la aplicación, instale las dependencias con los siguientes comandos, que se ejecutan línea a línea:

```console
npm install @azure/ms-rest-js
npm install @azure/cognitiveservices-luis-authoring
npm install @azure/cognitiveservices-luis-runtime
```

Su archivo `package.json` debe ser similar a este:

```json
{
  "name": "quickstart-sdk",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "@azure/cognitiveservices-luis-authoring": "^4.0.0-preview.3",
    "@azure/cognitiveservices-luis-runtime": "^5.0.0",
    "@azure/ms-rest-js": "^2.0.8"
  }
}
```

## <a name="authoring-object-model"></a>Creación de un modelo de objetos

El cliente de creación de Language Understanding (LUIS) es un objeto [LUISAuthoringClient](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/luisauthoringclient?view=azure-node-latest) que se autentica en Azure, que contiene la clave de creación.

## <a name="code-examples-for-authoring"></a>Ejemplos de código para la creación

Una vez creado el cliente, úselo para acceder a la funcionalidad, entre la que se incluye:

* Aplicaciones: [agregar](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/apps?view=azure-node-latest#add-applicationcreateobject--msrest-requestoptionsbase-), [eliminar](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/apps?view=azure-node-latest#deletemethod-string--models-appsdeletemethodoptionalparams-), [publicar](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/apps?view=azure-node-latest#publish-string--applicationpublishobject--msrest-requestoptionsbase-)
* Expresiones de ejemplo: [adición por lote](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/examples?view=azure-node-latest#batch-string--string--examplelabelobject----msrest-requestoptionsbase-), [eliminación por identificador](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/examples?view=azure-node-latest#deletemethod-string--string--number--msrest-requestoptionsbase-)
* Características: administración de [listas de frases](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/features?view=azure-node-latest#addphraselist-string--string--phraselistcreateobject--msrest-requestoptionsbase-)
* Modelo: administración de [intenciones](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/model?view=azure-node-latest#addintent-string--string--modelcreateobject--msrest-requestoptionsbase-) y entidades
* Patrón: administración de [patrones](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/pattern?view=azure-node-latest#addpattern-string--string--patternrulecreateobject--msrest-requestoptionsbase-)
* Entrenamiento: [entrenamiento](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/train?view=azure-node-latest#trainversion-string--string--msrest-requestoptionsbase-) de la aplicación y sondeo del [estado del entrenamiento](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/train?view=azure-node-latest#getstatus-string--string--msrest-requestoptionsbase-)
* [Versiones](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/versions?view=azure-node-latest): administración con clonación, exportación y eliminación

## <a name="prediction-object-model"></a>Modelo de objetos de predicción

El cliente de creación de Language Understanding (LUIS) es un objeto [LUISAuthoringClient](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-runtime/luisruntimeclient?view=azure-node-latest) que se autentica en Azure, que contiene la clave de creación.

## <a name="code-examples-for-prediction-runtime"></a>Ejemplos de código para el entorno de ejecución de predicción

Una vez creado el cliente, úselo para acceder a la funcionalidad, entre la que se incluye:

* [Predicción](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-runtime/predictionoperations?view=azure-node-latest#getslotprediction-string--string--predictionrequest--models-predictiongetslotpredictionoptionalparams-) por ranura `staging` o `production`
* [Predicción por versión](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-runtime/predictionoperations?view=azure-node-latest#getversionprediction-string--string--predictionrequest--models-predictiongetversionpredictionoptionalparams-)

[!INCLUDE [Bookmark links to same article](sdk-code-examples.md)]

## <a name="add-the-dependencies"></a>Adición de las dependencias

Abra el archivo `index.js` en el editor o IDE con nombre que prefiera y agregue las siguientes dependencias.

[!code-javascript[Add NPM libraries to code file](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=Dependencies)]

## <a name="add-boilerplate-code"></a>Adición de código reutilizable

1. Agregue el método `quickstart` y su llamada. Este método contiene la mayoría del código restante. A este método se le llama al final del archivo.

    ```javascript
    const quickstart = async () => {

        // add calls here


    }
    quickstart()
        .then(result => console.log("Done"))
        .catch(err => {
            console.log(`Error: ${err}`)
            })
    ```

1. Agregue el código restante en el método del inicio rápido, salvo que se especifique lo contrario.

## <a name="create-variables-for-the-app"></a>Creación de variables para la aplicación

Cree dos conjuntos de variables: cambie el primer conjunto y el segundo déjelo como aparece en el ejemplo de código. 

1. Cree variables para almacenar la clave de creación y los nombres de recursos.

    [!code-javascript[Variables you need to change](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=VariablesYouChange)]

1. Cree variables para almacenar los puntos de conexión, el nombre de la aplicación, su versión y el nombre de la intención.

    [!code-javascript[Variables you don't need to change](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=VariablesYouDontNeedToChangeChange)]

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree un objeto [CognitiveServicesCredentials](https://docs.microsoft.com/javascript/api/@azure/ms-rest-js/apikeycredentials?view=azure-node-latest) con la clave y úselo con el punto de conexión para crear un objeto [LUISAuthoringClient](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/luisauthoringclient?view=azure-node-latest).

[!code-javascript[Authenticate the client](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=AuthoringCreateClient)]

## <a name="create-a-luis-app"></a>Creación de una aplicación de LUIS

Las aplicaciones de LUIS contienen el modelo de procesamiento de lenguaje natural (NLP), que incluye intenciones, entidades y expresiones de ejemplo.

Cree el método [add](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/apps?view=azure-node-latest) de un objeto [AppsOperation](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/apps?view=azure-node-latest) para crear la aplicación. El nombre y la referencia cultural del idioma son propiedades necesarias.

[!code-javascript[Create a LUIS app](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=AuthoringCreateApplication)]


## <a name="create-intent-for-the-app"></a>Creación de una intención para la aplicación
El objeto principal en el modelo de una aplicación LUIS es la intención. Esta se alinea con una agrupación de _intenciones_ de expresiones de usuario. Un usuario puede formular una pregunta o realizar una declaración buscando una respuesta _intencionada_ concreta de un bot (u otra aplicación cliente). Ejemplos de intenciones pueden ser la reserva de un vuelo, preguntar sobre el tiempo en una ciudad de destino o pedir la información de contacto del servicio de atención al cliente.

Use el método [model.add_intent](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/model?view=azure-node-latest#addintent-string--string--modelcreateobject--msrest-requestoptionsbase-) con el nombre de la intención única y, después, pase el identificador de la aplicación, el identificador de la versión y el nuevo nombre de la intención.

El valor `intentName` está codificado de forma rígida en `OrderPizzaIntent` como parte de las variables de la sección [Creación de variables para la aplicación](#create-variables-for-the-app).

[!code-javascript[Create intent for the app](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=AddIntent)]

## <a name="create-entities-for-the-app"></a>Creación de entidades para la aplicación

Si bien las entidades no son necesarias, se encuentran en la mayoría de las aplicaciones. La entidad extrae información de la expresión del usuario, necesaria para cumplir la intención del usuario. Hay varios tipos de entidades [precompiladas](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/model?view=azure-node-latest#addcustomprebuiltentity-string--string--prebuiltdomainmodelcreateobject--msrest-requestoptionsbase-) y personalizadas, cada una con sus propios modelos de objeto de transformación de datos.  Las entidades precompiladas comunes que se pueden agregar a una aplicación incluyen [number](../luis-reference-prebuilt-number.md), [datetimeV2](../luis-reference-prebuilt-datetimev2.md), [geographyV2](../luis-reference-prebuilt-geographyv2.md) y [ordinal](../luis-reference-prebuilt-ordinal.md).

Es importante saber que las entidades no se marcan con una intención. Pueden aplicarse, y así suelen hacerlo, a muchas intenciones. Solo las expresiones de usuario de ejemplo se marcan para una única intención específica.

Los métodos de creación de entidades forman parte de la clase [Model](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/model?view=azure-node-latest). Cada tipo de entidad tiene su propio modelo de objeto de transformación de datos (DTO).

El código de creación de entidad crea una entidad de aprendizaje automático con subentidades y características aplicadas a las subentidades de `Quantity`.

:::image type="content" source="../media/quickstart-sdk/machine-learned-entity.png" alt-text="Captura de pantalla parcial del portal que muestra la entidad creada, una entidad de aprendizaje automático con subentidades y características aplicadas a las subentidades "Quantity".":::

[!code-javascript[Create entities for the app](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=AuthoringAddEntities)]

Coloque el siguiente método encima del método `quickstart` para buscar el identificador de la subentidad Quantity con el fin de asignar las características a esa subentidad.

[!code-javascript[Find subentity id](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=AuthoringSortModelObject)]

## <a name="add-example-utterance-to-intent"></a>Adición de expresiones de ejemplo a la intención

Con el fin de determinar la intención de una expresión y extraer las entidades, la aplicación necesita ejemplos de expresiones. Los ejemplos deben tener como objetivo una única intención específica y deben marcar todas las entidades personalizadas. No es necesario marcar las entidades precompiladas.

Para agregar expresiones de ejemplo, cree una lista de objetos [ExampleLabelObject](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/examplelabelobject?view=azure-node-latest), un objeto para cada expresión de ejemplo. Cada ejemplo debe marcar todas las entidades con un diccionario de pares nombre-valor de nombre de entidad y valor de entidad. El valor de la entidad debe reflejar exactamente lo que aparece en el texto de la expresión de ejemplo.

:::image type="content" source="../media/quickstart-sdk/labeled-example-machine-learned-entity.png" alt-text="Captura de pantalla parcial que muestra la expresión del ejemplo con etiquetas en el portal. ":::

Llame a [examples.add](https://docs.microsoft.com//javascript/api/@azure/cognitiveservices-luis-authoring/examples?view=azure-node-latest#add-string--string--examplelabelobject--models-examplesaddoptionalparams-) con el identificador de la aplicación, el identificador de la versión y el ejemplo.

[!code-javascript[Add example utterance to intent](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=AuthoringAddLabeledExamples)]

## <a name="train-the-app"></a>Entrenamiento de la aplicación

Una vez creado el modelo, debe entrenarse la aplicación de LUIS para esta versión del modelo. Un modelo entrenado puede usarse en un [contenedor](../luis-container-howto.md) o bien [publicarse](../luis-how-to-publish-app.md) en los espacios de ensayo o del producto.

El método [train.trainVersion](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/train?view=azure-node-latest#trainversion-string--string--msrest-requestoptionsbase-) necesita el identificador de la aplicación y el identificador de la versión.

Un modelo muy pequeño, como el que se muestra en este inicio rápido, se entrenará muy rápidamente. En el caso de las aplicaciones en el nivel de producción, el entrenamiento de la aplicación debe incluir una llamada de sondeo al método [get_status](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/train?view=azure-node-latest#getstatus-string--string--msrest-requestoptionsbase-) para determinar si el entrenamiento se ha realizado correctamente. La respuesta es una lista de objetos [ModelTrainingInfo](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/modeltraininginfo?view=azure-node-latest) con un estado independiente para cada objeto. Para que el entrenamiento se considere completo, todos los objetos deben ser correctos.

[!code-javascript[Train the app](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=TrainAppVersion)]

## <a name="publish-app-to-production-slot"></a>Publicación de la aplicación en el espacio de producción

Publique la aplicación de LUIS con el método [app.publish](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-authoring/apps?view=azure-node-latest#publish-string--applicationpublishobject--msrest-requestoptionsbase-). Así se publica la versión entrenada actual en el espacio especificado en el punto de conexión. La aplicación cliente usa este punto de conexión para enviar expresiones del usuario y con ellas predecir la intención y extraer entidades.

[!code-javascript[Publish app to production slot](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=PublishVersion)]


## <a name="authenticate-the-prediction-runtime-client"></a>Autenticación del cliente del entorno de ejecución de predicción

Use un objeto msRest.ApiKeyCredentials con su clave y úselo con su punto de conexión para crear un objeto [LUIS.LUISRuntimeClient](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-runtime/luisruntimeclient?view=azure-node-latest).

[!INCLUDE [Caution about using authoring key](caution-authoring-key.md)]

[!code-javascript [Authenticate the prediction runtime client](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=PredictionCreateClient)]

## <a name="get-prediction-from-runtime"></a>Obtención de la predicción a partir del entorno de ejecución

Agregue el siguiente código para crear la solicitud al entorno de ejecución de predicción. La expresión del usuario forma parte del objeto [predictionRequest](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-runtime/predictionrequest?view=azure-node-latest).

El método **[luisRuntimeClient.prediction.getSlotPrediction](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-luis-runtime/predictionoperations?view=azure-node-latest#getslotprediction-string--string--predictionrequest--models-predictiongetslotpredictionoptionalparams-)** necesita varios parámetros como, por ejemplo, el identificador de la aplicación, el nombre de la ranura y el objeto de la solicitud de predicción para completar la solicitud. Las demás opciones, como verbose, muestran todas las intenciones y el registro es opcional.

[!code-javascript [Get prediction from runtime](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=QueryPredictionEndpoint)]

[!INCLUDE [Prediction JSON response](sdk-json.md)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación con el comando `node index.js` en el archivo de inicio rápido.

```console
node index.js
```
