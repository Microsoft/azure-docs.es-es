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
ms.custom: include file, devx-track-dotnet, cog-serv-seo-aug-2020
ms.author: diberry
ms.openlocfilehash: fff4e8c43263dfcc49be6cb6269078643118e8df
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89323119"
---
Use las bibliotecas cliente de Language Understanding (LUIS) para .NET para:
* Crear una aplicación.
* Agregar una intención, una entidad de aprendizaje automático, con una expresión de ejemplo
* Entrenar y publicar una aplicación.
* Consultar el runtime de predicción.

[Documentación de referencia](https://docs.microsoft.com/dotnet/api/overview/azure/cognitiveservices/client/languageunderstanding?view=azure-dotnet) | Código fuente de la biblioteca de [creación](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Language.LUIS.Authoring) y [prediccion](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Language.LUIS.Runtime) | NuGet de [creación](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring/) y [predicción](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime/) | [Ejemplo de C#](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/LanguageUnderstanding/sdk-3x//Program.cs)

## <a name="prerequisites"></a>Requisitos previos

* La versión actual de [.NET Core](https://dotnet.microsoft.com/download/dotnet-core) y de la [CLI de .NET Core](https://docs.microsoft.com/dotnet/core/tools/).
* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* Cuando tenga la suscripción a Azure, [cree un recurso de creación de Language Understanding](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesLUISAllInOne) en Azure Portal para obtener la clave de creación y el punto de conexión. Espere a que se implemente y haga clic en el botón **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que [cree](../luis-how-to-azure-subscription.md#create-luis-resources-in-azure-portal) para conectar la aplicación a la creación de Language Understanding. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente. Puede usar el plan de tarifa gratuito (`F0`) para probar el servicio.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

Cree una nueva aplicación de consola de .NET Core en el IDE o editor que prefiera.

1. En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `new` dotnet para crear una nueva aplicación de consola con el nombre `language-understanding-quickstart`. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: `Program.cs`.

    ```dotnetcli
    dotnet new console -n language-understanding-quickstart
    ```

1. Cambie el directorio a la carpeta de aplicaciones recién creada.

    ```dotnetcli
    cd language-understanding-quickstart
    ```

1. Para compilar la aplicación:

    ```dotnetcli
    dotnet build
    ```

    La salida de la compilación no debe contener advertencias ni errores.

    ```console
    ...
    Build succeeded.
     0 Warning(s)
     0 Error(s)
    ...
    ```

### <a name="install-the-nuget-libraries"></a>Instalación de las bibliotecas de NuGet

En el directorio de la aplicación, instale las bibliotecas cliente de Language Understanding (LUIS) para .NET con los siguientes comandos:

```dotnetcli
dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring --version 3.2.0-preview.3
dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.1.0-preview.1
```

## <a name="authoring-object-model"></a>Creación de un modelo de objetos

El cliente de creación de Language Understanding (LUIS) es un objeto [LUISAuthoringClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.luisauthoringclient?view=azure-dotnet) que se autentica en Azure, que contiene la clave de creación.

## <a name="code-examples-for-authoring"></a>Ejemplos de código para la creación

Una vez creado el cliente, úselo para acceder a la funcionalidad, entre la que se incluye:

* Aplicaciones: [creación](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.appsextensions.addasync?view=azure-dotnet), [eliminación](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.appsextensions.deleteasync?view=azure-dotnet), [publicación](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.appsextensions.publishasync?view=azure-dotnet)
* Expresiones de ejemplo: [adición](https://docs.microsoft.com//dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.examplesextensions.addasync?view=azure-dotnet) y [eliminación por identificador](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.examplesextensions.deleteasync?view=azure-dotnet)
* Características: administración de [listas de frases](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.featuresextensions.addphraselistasync?view=azure-dotnet)
* Modelo: administración de [intenciones](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.modelextensions?view=azure-dotnet) y entidades
* Patrón: administración de [patrones](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.patternextensions?view=azure-dotnet)
* Entrenamiento: [entrenamiento](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.trainextensions.trainversionasync?view=azure-dotnet) de la aplicación y sondeo del [estado del entrenamiento](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.trainextensions.getstatusasync?view=azure-dotnet)
* [Versiones](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.versionsextensions?view=azure-dotnet): administración con clonación, exportación y eliminación

## <a name="prediction-object-model"></a>Modelo de objetos de predicción

El cliente del entorno de ejecución de predicción de Language Understanding (LUIS) es un objeto [LUISRuntimeClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.runtime.luisruntimeclient?view=azure-dotnet) que se autentica en Azure, que contiene la clave del recurso.

## <a name="code-examples-for-prediction-runtime"></a>Ejemplos de código para el entorno de ejecución de predicción

Una vez creado el cliente, úselo para acceder a la funcionalidad, entre la que se incluye:

* Predicción por [almacenamiento provisional o espacio de producción](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.runtime.predictionoperationsextensions.getslotpredictionasync?view=azure-dotnet)
* Predicción por [versión](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.runtime.predictionoperationsextensions.getversionpredictionasync?view=azure-dotnet)


[!INCLUDE [Bookmark links to same article](sdk-code-examples.md)]

## <a name="add-the-dependencies"></a>Adición de las dependencias

En el directorio del proyecto, abra el archivo *Program.cs* en el editor o IDE que prefiera. Reemplace el código existente `using` por las siguientes directivas `using`:

[!code-csharp[Add NuGet libraries to code file](~/cognitive-services-quickstart-code/dotnet/LanguageUnderstanding/sdk-3x//Program.cs?name=Dependencies)]

## <a name="add-boilerplate-code"></a>Adición de código reutilizable

1. Cambie la firma del método `Main` para permitir las llamadas asincrónicas:

    ```csharp
    public static async Task Main()
    ```

1. Agregue el resto del código del método `Main` de la clase `Program`, salvo que se especifique lo contrario.

## <a name="create-variables-for-the-app"></a>Creación de variables para la aplicación

Cree dos conjuntos de variables: cambie el primer conjunto y el segundo déjelo como aparece en el ejemplo de código. 

1. Cree variables para almacenar la clave de creación y los nombres de recursos.

    [!code-csharp[Variables you need to change](~/cognitive-services-quickstart-code/dotnet/LanguageUnderstanding/sdk-3x//Program.cs?name=VariablesYouChange)]

1. Cree variables para almacenar los puntos de conexión, el nombre de la aplicación, su versión y el nombre de la intención.

    [!code-csharp[Variables you don't need to change](~/cognitive-services-quickstart-code/dotnet/LanguageUnderstanding/sdk-3x//Program.cs?name=VariablesYouDontNeedToChangeChange)]

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree un objeto [ApiKeyServiceClientCredentials](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.apikeyserviceclientcredentials?view=azure-dotnet) con la clave y úselo con el punto de conexión para crear un objeto [LUISAuthoringClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.luisauthoringclient?view=azure-dotnet).

[!code-csharp[Authenticate the client](~/cognitive-services-quickstart-code/dotnet/LanguageUnderstanding/sdk-3x//Program.cs?name=AuthoringCreateClient)]

## <a name="create-a-luis-app"></a>Creación de una aplicación de LUIS

Las aplicaciones de LUIS contienen el modelo de procesamiento de lenguaje natural (NLP), que incluye intenciones, entidades y expresiones de ejemplo.

Cree un elemento [ApplicationCreateObject](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.models.applicationcreateobject?view=azure-dotnet). El nombre y la referencia cultural del idioma son propiedades necesarias. Llame al método [Apps.AddAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.appsextensions.addasync?view=azure-dotnet). La respuesta es el id. de la aplicación.

[!code-csharp[Create a LUIS app](~/cognitive-services-quickstart-code/dotnet/LanguageUnderstanding/sdk-3x//Program.cs?name=AuthoringCreateApplication)]

## <a name="create-intent-for-the-app"></a>Creación de una intención para la aplicación
El objeto principal en el modelo de una aplicación LUIS es la intención. La intención se alinea con una agrupación de _intenciones_ de expresión del usuario. Un usuario puede formular una pregunta o realizar una declaración buscando una respuesta _intencionada_ concreta de un bot (u otra aplicación cliente). Ejemplos de intenciones pueden ser la reserva de un vuelo, preguntar sobre el tiempo en una ciudad de destino o pedir la información de contacto del servicio de atención al cliente.

Cree un elemento [ModelCreateObject](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.models.modelcreateobject?view=azure-dotnet) con el nombre de la intención única y, a continuación, pase el identificador de la aplicación, el identificador de la versión y la instancia de ModelCreateObject al método [Model.AddIntentAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.modelextensions.addintentasync?view=azure-dotnet). La respuesta es el identificador de la intención.

El valor `intentName` está codificado de forma rígida en `OrderPizzaIntent` como parte de las variables de la sección [Creación de variables para la aplicación](#create-variables-for-the-app).

[!code-csharp[Create intent for the app](~/cognitive-services-quickstart-code/dotnet/LanguageUnderstanding/sdk-3x//Program.cs?name=AddIntent)]

## <a name="create-entities-for-the-app"></a>Creación de entidades para la aplicación

Si bien las entidades no son necesarias, se encuentran en la mayoría de las aplicaciones. La entidad extrae información de la expresión del usuario, necesaria para cumplir la intención del usuario. Hay varios tipos de entidades [precompiladas](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.modelextensions.addprebuiltasync?view=azure-dotnet) y personalizadas, cada una con sus propios modelos de objeto de transformación de datos.  Las entidades precompiladas comunes que se pueden agregar a una aplicación incluyen [number](../luis-reference-prebuilt-number.md), [datetimeV2](../luis-reference-prebuilt-datetimev2.md), [geographyV2](../luis-reference-prebuilt-geographyv2.md) y [ordinal](../luis-reference-prebuilt-ordinal.md).

Es importante saber que las entidades no se marcan con una intención. Pueden aplicarse, y así suelen hacerlo, a muchas intenciones. Las expresiones de usuario de ejemplo son las únicas marcadas para una intención individual específica.

Los métodos de creación de entidades forman parte de la clase [Model](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.modelextensions?view=azure-dotnet). Cada tipo de entidad tiene su propio modelo de objeto de transformación de datos, que normalmente contiene la palabra `model` en el espacio de nombres [Models](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.models?view=azure-dotnet).

El código de creación de entidad crea una entidad de aprendizaje automático con subentidades y características aplicadas a las subentidades de `Quantity`.

:::image type="content" source="../media/quickstart-sdk/machine-learned-entity.png" alt-text="Captura de pantalla parcial del portal que muestra la entidad creada, una entidad de aprendizaje automático con subentidades y características aplicadas a las subentidades "Quantity".":::

[!code-csharp[Create entities for the app](~/cognitive-services-quickstart-code/dotnet/LanguageUnderstanding/sdk-3x//Program.cs?name=AuthoringAddEntities)]

Use el método siguiente en la clase para buscar el identificador de la subentidad Quantity con el fin de asignar las características a esa subentidad.

[!code-csharp[Find subentity id](~/cognitive-services-quickstart-code/dotnet/LanguageUnderstanding/sdk-3x//Program.cs?name=AuthoringSortModelObject)]

## <a name="add-example-utterance-to-intent"></a>Adición de expresiones de ejemplo a la intención

Con el fin de determinar la intención de una expresión y extraer las entidades, la aplicación necesita ejemplos de expresiones. Los ejemplos deben tener como objetivo una única intención específica y deben marcar todas las entidades personalizadas. No es necesario marcar las entidades precompiladas.

Para agregar expresiones de ejemplo, cree una lista de objetos [ExampleLabelObject](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.models.examplelabelobject?view=azure-dotnet), un objeto para cada expresión de ejemplo. Cada ejemplo debe marcar todas las entidades con un diccionario de pares nombre-valor de nombre de entidad y valor de entidad. El valor de la entidad debe reflejar exactamente lo que aparece en el texto de la expresión de ejemplo.

:::image type="content" source="../media/quickstart-sdk/labeled-example-machine-learned-entity.png" alt-text="Captura de pantalla parcial que muestra la expresión del ejemplo con etiquetas en el portal. ":::

Llame a [Examples.AddAsync](https://docs.microsoft.com//dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.examplesextensions.addasync?view=azure-dotnet) con el identificador de la aplicación, el identificador de la versión y el ejemplo.

[!code-csharp[Add example utterance to intent](~/cognitive-services-quickstart-code/dotnet/LanguageUnderstanding/sdk-3x//Program.cs?name=AuthoringAddLabeledExamples)]

## <a name="train-the-app"></a>Entrenamiento de la aplicación

Una vez creado el modelo, debe entrenarse la aplicación de LUIS para esta versión del modelo. Un modelo entrenado puede usarse en un [contenedor](../luis-container-howto.md) o bien [publicarse](../luis-how-to-publish-app.md) en los espacios de ensayo o del producto.

El método [Train.TrainVersionAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.trainextensions?view=azure-dotnet) necesita el id. de la aplicación y el identificador de la versión.

Un modelo muy pequeño, como el que se muestra en este inicio rápido, se entrenará muy rápidamente. En el caso de las aplicaciones en el nivel de producción, el entrenamiento de la aplicación debe incluir una llamada de sondeo al método [GetStatusAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.trainextensions.getstatusasync?view=azure-dotnet#Microsoft_Azure_CognitiveServices_Language_LUIS_Authoring_TrainExtensions_GetStatusAsync_Microsoft_Azure_CognitiveServices_Language_LUIS_Authoring_ITrain_System_Guid_System_String_System_Threading_CancellationToken_) para determinar cuándo o si el entrenamiento se realizó correctamente. La respuesta es una lista de objetos [ModelTrainingInfo](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.models.modeltraininginfo?view=azure-dotnet) con un estado independiente para cada objeto. Para que el entrenamiento se considere completo, todos los objetos deben ser correctos.

[!code-csharp[Train the app](~/cognitive-services-quickstart-code/dotnet/LanguageUnderstanding/sdk-3x//Program.cs?name=TrainAppVersion)]

## <a name="publish-app-to-production-slot"></a>Publicación de la aplicación en el espacio de producción

La aplicación de LUIS se publica con el método [PublishAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.appsextensions.publishasync?view=azure-dotnet). Así se publica la versión entrenada actual en el espacio especificado en el punto de conexión. La aplicación cliente usa este punto de conexión para enviar expresiones del usuario y con ellas predecir la intención y extraer entidades.

[!code-csharp[Publish app to production slot](~/cognitive-services-quickstart-code/dotnet/LanguageUnderstanding/sdk-3x//Program.cs?name=PublishVersion)]

## <a name="authenticate-the-prediction-runtime-client"></a>Autenticación del cliente del entorno de ejecución de predicción

Use un objeto [ApiKeyServiceClientCredentials](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.runtime.apikeyserviceclientcredentials?view=azure-dotnet) con la clave y úselo con el punto de conexión para crear un objeto [LUISRuntimeClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.runtime.luisruntimeclient?view=azure-dotnet).

[!INCLUDE [Caution about using authoring key](caution-authoring-key.md)]

[!code-csharp[Authenticate the prediction runtime client](~/cognitive-services-quickstart-code/dotnet/LanguageUnderstanding/sdk-3x//Program.cs?name=PredictionCreateClient)]


## <a name="get-prediction-from-runtime"></a>Obtención de la predicción a partir del entorno de ejecución

Agregue el siguiente código para crear la solicitud al entorno de ejecución de predicción.

La expresión de usuario forma parte del objeto [PredictionRequest](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.runtime.models.predictionrequest?view=azure-dotnet).

El método **GetSlotPredictionAsync** necesita varios parámetros como, por ejemplo, el identificador de la aplicación, el nombre del espacio y el objeto de la solicitud de predicción para completar la solicitud. Las demás opciones, como verbose, muestran todas las intenciones y el registro es opcional.

[!code-csharp[Get prediction from runtime](~/cognitive-services-quickstart-code/dotnet/LanguageUnderstanding/sdk-3x//Program.cs?name=QueryPredictionEndpoint)]

[!INCLUDE [Prediction JSON response](sdk-json.md)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación con el comando `dotnet run` desde el directorio de la aplicación.

```dotnetcli
dotnet run
```
