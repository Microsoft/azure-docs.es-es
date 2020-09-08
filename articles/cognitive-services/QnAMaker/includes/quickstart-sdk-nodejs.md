---
title: 'Inicio rápido: Biblioteca cliente de QnA Maker para Node.js'
description: Este inicio rápido le muestra cómo empezar a trabajar con la biblioteca cliente de QnA Maker para Node.js.
ms.topic: quickstart
ms.date: 06/18/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: 4085ec5aaff75f33fc7a87708693084831ff5d8d
ms.sourcegitcommit: c293217e2d829b752771dab52b96529a5442a190
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/15/2020
ms.locfileid: "88246066"
---
Use la biblioteca cliente de QnA Maker para Node.js con los siguientes fines:

* Crear una base de conocimiento
* Actualizar una base de conocimiento
* Publicar una base de conocimiento
* Obtener la clave del punto de conexión de tiempo de ejecución de predicción
* Espera de una tarea de ejecución prolongada
* Descargar una base de conocimiento
* Obtención de respuesta
* Eliminación de una base de conocimiento

[Documentación de referencia](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/?view=azure-node-latest) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-qnamaker) | [Paquete (npm)](https://www.npmjs.com/package/@azure/cognitiveservices-qnamaker) | [Ejemplos de Node.js](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/QnAMaker/sdk/qnamaker_quickstart.js)

[!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* La versión actual de [Node.js](https://nodejs.org).
* Cuando tenga la suscripción a Azure, cree un [recurso de QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) en Azure Portal para obtener la clave de creación y el recurso. Tras su implementación, seleccione **Ir al recurso**.
    * Necesitará la clave y el nombre del recurso que cree para conectar la aplicación a QnA Maker API. En una sección posterior de este mismo inicio rápido pegará la clave y el nombre del recurso en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-nodejs-application"></a>Creación de una aplicación Node.js

En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para la aplicación y vaya a él.

```console
mkdir qnamaker_quickstart && cd qnamaker_quickstart
```

Ejecute el comando `npm init -y` para crear una aplicación de nodo con un archivo `package.json`.

```console
npm init -y
```

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Instale los siguientes paquetes NPM:

```console
npm install @azure/cognitiveservices-qnamaker
npm install @azure/cognitiveservices-qnamaker-runtime
npm install @azure/ms-rest-js
```

El archivo `package.json` de la aplicación se actualiza con las dependencias.

Cree un archivo llamado index.js e importe las bibliotecas siguientes:

[!code-javascript[Dependencies](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=Dependencies)]

Cree una variable para la clave de Azure y el nombre del recurso. Tanto las direcciones URL de creación como de predicción usan el nombre del recurso como subdominio.

> [!IMPORTANT]
> Vaya a Azure Portal y busque la clave y el punto de conexión del recurso de QnA Maker que creó en los requisitos previos. Los encontrará en la página de **clave y punto de conexión** del recurso, en **Administración de recursos**.
> Necesitará la clave completa para crear la base de conocimiento. Solo necesita el nombre del recurso del punto de conexión. El formato es `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com`.
> Recuerde quitar la clave del código cuando haya terminado y no hacerla nunca pública. En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Por ejemplo, [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-overview) proporciona almacenamiento de claves seguro.

[!code-javascript[Set the resource key and resource name](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=Resourcevariables)]

## <a name="object-models"></a>Modelos de objetos

QnA Maker usa dos modelos de objetos diferentes:
* **[QnAMakerClient](#qnamakerclient-object-model)** , el objeto para crear, administrar, publicar y descargar la base de conocimiento.
* **[QnAMakerRuntime](#qnamakerruntimeclient-object-model)** , el objeto para consultar la base de conocimiento con GenerateAnswer API y enviar nuevas preguntas sugeridas mediante Train API (como parte del [aprendizaje activo](../concepts/active-learning-suggestions.md)).


### <a name="qnamakerclient-object-model"></a>Modelo de objetos QnAMakerClient

El cliente de QnA Maker es un objeto [QnAMakerClient](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/qnamakerclient?view=azure-node-latest) que se autentica en Azure mediante sus credenciales y que contiene la clave.

Una vez creado el cliente, utilice la propiedad [knowledgebase](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/qnamakerclient?view=azure-node-latest#knowledgebase) para crear, administrar y publicar la base de conocimiento.

Administre la base de conocimiento mediante el envío de un objeto JSON. En el caso de operaciones inmediatas, un método suele devolver un objeto JSON que indica el estado. En el caso de operaciones de ejecución prolongada, la respuesta es el identificador de la operación. Llame al método [client.operations.getDetails](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/operations?view=azure-node-latest#getdetails-string--msrest-requestoptionsbase-) con el identificador de la operación para determinar el [estado de la solicitud](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/operation?view=azure-node-latest).

### <a name="qnamakerruntimeclient-object-model"></a>Modelo de objetos QnAMakerRuntimeClient

El cliente de QnA Maker de predicción es un objeto QnAMakerRuntimeClient que se autentica en Azure mediante Microsoft.Rest.ServiceClientCredentials, que contiene la clave de tiempo de ejecución de predicción que devuelve la llamada del cliente de creación, [client.EndpointKeys.getKeys](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/endpointkeys?view=azure-node-latest#getkeys-msrest-requestoptionsbase-), una vez publicada la base de conocimiento.


## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo realizar las siguientes acciones con la biblioteca cliente de QnA Maker para .NET:

* [Autenticación del cliente de creación](#authenticate-the-client-for-authoring-the-knowledge-base)
* [Creación de una base de conocimientos](#create-a-knowledge-base)
* [Actualización de una base de conocimientos](#update-a-knowledge-base)
* [Descarga de una base de conocimiento](#download-a-knowledge-base)
* [Publicación de una base de conocimientos](#publish-a-knowledge-base)
* [Eliminación de una base de conocimiento](#delete-a-knowledge-base)
* [Obtención de la clave de tiempo de ejecución de consulta](#get-query-runtime-key)
* [Obtención del estado de una operación](#get-status-of-an-operation)
* [Autenticación del cliente de tiempo de ejecución de consulta](#authenticate-the-runtime-for-generating-an-answer)
* [Generación de una respuesta de la base de conocimiento](#generate-an-answer-from-the-knowledge-base)



## <a name="authenticate-the-client-for-authoring-the-knowledge-base"></a>Autenticación del cliente para la creación de la base de conocimiento

Cree una instancia de un cliente con la clave y el punto de conexión. Cree un objeto ServiceClientCredentials con la clave y úselo con el punto de conexión para crear un objeto [QnAMakerClient](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/qnamakerclient?view=azure-node-latest).

[!code-javascript[Create QnAMakerClient object with key and endpoint](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=AuthorizationAuthor)]

## <a name="create-a-knowledge-base"></a>Creación de una base de conocimientos

Una base de conocimiento almacena pares de preguntas y respuestas para el objeto [CreateKbDTO](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/createkbdto?view=azure-node-latest) procedentes de tres orígenes:

* Para **contenido editorial**, use el objeto [QnADTO](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/qnadto?view=azure-node-latest).
    * Para usar metadatos y avisos de seguimiento, utilice el contexto editorial, ya que estos datos se agregan en el nivel de un par de QnA individual.
* Para **archivos**, use el objeto [FileDTO](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/filedto?view=azure-node-latest). El objeto FileDTO incluye el nombre de archivo así como la dirección URL pública para llegar al archivo.
* En el caso de las **direcciones URL**, use una lista de cadenas para representar las direcciones URL disponibles públicamente.

El paso de creación también incluye las propiedades de la base de conocimiento:
* `defaultAnswerUsedForExtraction`: lo que se devuelve cuando no se encuentra ninguna respuesta.
* `enableHierarchicalExtraction`: crea automáticamente relaciones de mensajes entre los pares de QnA extraídos.
* `language`: al crear la primera base de conocimiento de un recurso, establezca el lenguaje que se va a usar en el índice de Azure Search.

Llame al método [create](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/knowledgebase?view=azure-node-latest#create-createkbdto--servicecallback-operation--) con la información de la base de conocimiento. La información de la base de conocimiento es básicamente un objeto JSON.

Cuando el método create realice una devolución, pase el identificador de la operación devuelta al método [wait_for_operation](#get-status-of-an-operation) para sondear el estado. El método wait_for_operation devuelve una salida cuando finaliza la operación. Analice el valor del encabezado `resourceLocation` de la operación devuelta para obtener el nuevo identificador de la base de conocimiento.

[!code-javascript[Create a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=CreateKBMethod&highlight=39,46)]

Asegúrese de incluir la función [`wait_for_operation`](#get-status-of-an-operation), a la que se hace referencia en el código anterior, con el fin de crear correctamente una base de conocimiento.

## <a name="update-a-knowledge-base"></a>Actualización de una base de conocimientos

Para actualizar una base de conocimiento, pase el identificador de la base de conocimiento y un objeto [UpdateKbOperationDTO](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/updatekboperationdto?view=azure-node-latest) que contenga los objetos de DTO [add](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/updatekboperationdto?view=azure-node-latest#add), [update](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/updatekboperationdto?view=azure-node-latest#update) y [delete](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/updatekboperationdto?view=azure-node-latest#deleteproperty) al método [update](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/knowledgebase?view=azure-node-latest#update-string--updatekboperationdto--msrest-requestoptionsbase-). Los DTO son también básicamente objetos JSON. Use el método [wait_for_operation](#get-status-of-an-operation) para determinar si la actualización se realizó correctamente.

[!code-javascript[Update a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=UpdateKBMethod&highlight=74,81)]

Asegúrese de incluir la función [`wait_for_operation`](#get-status-of-an-operation), a la que se hace referencia en el código anterior, con el fin de actualizar correctamente una base de conocimiento.

## <a name="download-a-knowledge-base"></a>Descarga de una base de conocimiento

Use el método [download](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/knowledgebase?view=azure-node-latest#download-string--models-environmenttype--msrest-requestoptionsbase-) para descargar la base de datos como una lista de [QnADocumentsDTO](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/qnadocumentsdto?view=azure-node-latest). Esto _no_ equivale a las exportaciones del portal de QnA Maker desde la página **Configuración** ya que el resultado de este método no es un archivo TSV.

[!code-javascript[Download a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=DownloadKB&highlight=2)]


## <a name="publish-a-knowledge-base"></a>Publicación de una base de conocimientos

Publique la base de conocimiento mediante el método [publish](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/knowledgebase?view=azure-node-latest#publish-string--msrest-requestoptionsbase-). Esto toma el modelo actual guardado y entrenado, al que hace referencia el identificador de la base de conocimiento, y lo publica en un punto de conexión. Compruebe el código de respuesta HTTP para validar que la publicación se haya realizado correctamente.

[!code-javascript[Publish a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=PublishKB&highlight=3)]


## <a name="get-query-runtime-key"></a>Obtención de la clave de tiempo de ejecución de consulta

Una vez publicada una base de conocimiento, necesita la clave de tiempo de ejecución de consulta para consultar el tiempo de ejecución. Esta clave no es la misma que la que se usa para crear el objeto de cliente original.

Use el método [EndpointKeys.getKeys](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/endpointkeys?view=azure-node-latest) para obtener la clase [EndpointKeysDTO](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/endpointkeysdto?view=azure-node-latest).

Use cualquiera de las propiedades de clave devueltas en el objeto para consultar la base de conocimiento.

[!code-javascript[Get query runtime key](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=GetQueryEndpointKey&highlight=4)]

## <a name="authenticate-the-runtime-for-generating-an-answer"></a>Autenticación del tiempo de ejecución para generar una respuesta

Cree un QnAMakerRuntimeClient para consultar la base de conocimiento con el fin de generar una respuesta o entrenar el aprendizaje activo.

[!code-javascript[Authenticate the runtime](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=AuthorizationQuery)]

Use QnAMakerRuntimeClient para obtener una respuesta de la base de conocimiento o enviar a esta nuevas preguntas sugeridas para el [aprendizaje activo](../concepts/active-learning-suggestions.md).

## <a name="generate-an-answer-from-the-knowledge-base"></a>Generación de una respuesta de la base de conocimiento

Genere una respuesta a partir de una base de conocimiento publicada mediante el método RuntimeClient.runtime.generateAnswer. Este método acepta el identificador de la base de conocimiento y QueryDTO. Acceda a las propiedades adicionales de QueryDTO, como Top y Context, que se usarán en el bot de chat.

[!code-javascript[Generate an answer from a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=GenerateAnswer&highlight=3)]

Este es un ejemplo sencillo de consulta de la base de conocimiento. Para comprender los escenarios de consulta avanzados, revise [otros ejemplos de consulta](../quickstarts/get-answer-from-knowledge-base-using-url-tool.md?pivots=url-test-tool-curl#use-curl-to-query-for-a-chit-chat-answer).

## <a name="delete-a-knowledge-base"></a>Eliminación de una base de conocimiento

Elimine la base de conocimiento con el método [delete](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/knowledgebase?view=azure-node-latest#deletemethod-string--msrest-requestoptionsbase-) y el parámetro del identificador de la base de conocimiento.

[!code-javascript[Delete a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=DeleteKB&highlight=3)]

## <a name="get-status-of-an-operation"></a>Obtención del estado de una operación

Algunos métodos, como Create y Update, pueden tardar bastante tiempo en que en lugar de esperar a que finalice el proceso, se devuelva una [operación](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/operations?view=azure-node-latest). Use el [identificador](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-qnamaker/operation?view=azure-node-latest#operationid) de la operación para realizar un sondeo (con lógica de reintento) para determinar el estado del método original.

La llamada _delayTimer_ del siguiente bloque de código se usa para simular la lógica de reintentos. Reemplácela por su propia lógica de reintentos.

[!code-javascript[Monitor an operation](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=MonitorOperation&highlight=8)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación con el comando `node index.js` desde el directorio de la aplicación.

```console
node index.js
```

El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/QnAMaker/sdk/qnamaker_quickstart.js).