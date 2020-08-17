---
title: archivo de inclusión
description: archivo de inclusión
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: include
ms.custom: include file, devx-track-javascript
ms.date: 07/30/2020
ms.openlocfilehash: 4f57d3a6c959a8ec912722a617496c52f412c13f
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87461160"
---
[Documentación de referencia](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-personalizer/?view=azure-node-latest) |[Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-personalizer) | [Paquete (NPM)](https://www.npmjs.com/package/@azure/cognitiveservices-personalizer) | [Ejemplos](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/javascript/Personalizer)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)
* La versión actual de [Node.js](https://nodejs.org) y NPM.

## <a name="using-this-quickstart"></a>Uso de este inicio rápido


Hay que dar varios pasos para usar este inicio rápido:

* En Azure Portal, cree un recurso de Personalizer
* En la página **Configuración** del recurso Personalizer de Azure Portal, cambie la frecuencia de actualización del modelo a un intervalo muy corto.
* En un editor de código, cree un archivo de código y edítelo
* En la línea de comandos o en el terminal, instale el SDK desde la línea de comandos
* En la línea de comandos o en el terminal, ejecute el archivo de código

[!INCLUDE [Create Azure resource for Personalizer](create-personalizer-resource.md)]

[!INCLUDE [Change model frequency](change-model-frequency.md)]

## <a name="create-a-new-nodejs-application"></a>Creación de una aplicación Node.js

En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para la aplicación y vaya a él.

```console
mkdir myapp && cd myapp
```

Ejecute el comando `npm init -y` para crear un archivo `package.json`.

```console
npm init -y
```

## <a name="install-the-nodejs-library-for-personalizer"></a>Instalación de la biblioteca de Node.js para Personalizer

Instale la biblioteca cliente de Personalizer para Node.js con el siguiente comando:

```console
npm install @azure/cognitiveservices-personalizer --save
```

Instale los paquetes de NPM restantes para este inicio rápido:

```console
npm install @azure/ms-rest-azure-js @azure/ms-rest-js readline-sync uuid --save
```

## <a name="object-model"></a>Modelo de objetos

El cliente de Personalizer es un objeto [PersonalizerClient](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-personalizer/personalizerclient?view=azure-node-latest) que se autentica en Azure mediante Microsoft.Rest.ServiceClientCredentials, que contiene la clave.

Para solicitar el mejor elemento del contenido, cree un elemento [RankRequest](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-personalizer/rankrequest?view=azure-node-latest) y, luego, páselo al método [client.Rank](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-personalizer/personalizerclient?view=azure-node-latest#rank-rankrequest--msrest-requestoptionsbase-). El método Rank devuelve un elemento RankResponse.

Para enviar una recompensa a Personalizer, cree un [RewardRequest](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-personalizer/rewardrequest?view=azure-node-latest) y, a continuación, páselo al método [Reward](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-personalizer/events?view=azure-node-latest#reward-string--rewardrequest--servicecallback-void--) en la clase de eventos.

La determinación de la recompensa en este inicio rápido es trivial. En un sistema de producción, la determinación de lo que afecta a la [puntuación de recompensa](../concept-rewards.md) y cuánto le afecta, puede ser un proceso complejo que decida cambiar con el tiempo. Esta debe ser una de las decisiones de diseño principales en la arquitectura de su instancia de Personalizer.

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo realizar las siguientes acciones con la biblioteca cliente de Personalizer para Node.js:

* [Creación de un cliente de Personalizer](#create-a-personalizer-client)
* [API Rank](#request-the-best-action)
* [API Reward](#send-a-reward)

## <a name="create-a-new-nodejs-application"></a>Creación de una aplicación Node.js

Cree una aplicación de Node.js en el IDE o editor que prefiera con el nombre `sample.js`.

## <a name="add-the-dependencies"></a>Adición de las dependencias

Abra el archivo **sample.js** en el editor o IDE que prefiera. Agregue el siguiente `requires` para agregar los paquetes de NPM:

[!code-javascript[Add module dependencies](~/cognitive-services-quickstart-code/javascript/Personalizer/sample.js?name=Dependencies)]

## <a name="add-personalizer-resource-information"></a>Incorporación de la información de recursos de Personalizer

Edite las variables key y endpoint de la parte superior del archivo del código para la clave y el punto de conexión de su recurso. 

[!code-javascript[Add Personalizer resource information](~/cognitive-services-quickstart-code/javascript/Personalizer/sample.js?name=AuthorizationVariables)]

## <a name="create-a-personalizer-client"></a>Creación de un cliente de Personalizer

A continuación, cree un método para devolver un cliente de Personalizer. El parámetro para el método es `PERSONALIZER_RESOURCE_ENDPOINT` y el elemento ApiKey es `PERSONALIZER_RESOURCE_KEY`.

[!code-javascript[Create a Personalizer client](~/cognitive-services-quickstart-code/javascript/Personalizer/sample.js?name=Client)]

## <a name="get-content-choices-represented-as-actions"></a>Obtención de opciones de contenido representadas como acciones

Las acciones representan las opciones de contenido entre las que quiere que Personalizer seleccione el mejor elemento de contenido. Agregue los métodos siguientes a la clase Program para representar el conjunto de acciones y sus características.

[!code-javascript[Create user features](~/cognitive-services-quickstart-code/javascript/Personalizer/sample.js?name=createUserFeatureTimeOfDay)]

[!code-javascript[Create actions](~/cognitive-services-quickstart-code/javascript/Personalizer/sample.js?name=getActions)]

## <a name="create-the-learning-loop"></a>Creación del bucle de aprendizaje

El bucle de aprendizaje de Personalizer es un ciclo de llamadas [Rank](#request-the-best-action) y [Reward](#send-a-reward). En este inicio rápido, cada llamada Rank para personalizar el contenido, va seguida de una llamada Reward para indicar a Personalizer cómo es de eficaz el funcionamiento del servicio.

El siguiente código pasa por un ciclo en el que se pregunta al usuario sus preferencias en la línea de comandos, se envía esa información a Personalizer para que seleccione la mejor acción, se presenta la selección al cliente para que elija de la lista y, después, se envía una recompensa a Personalizer en la que se señala el grado de acierto que tuvo el servicio al clasificar la selección.

[!code-javascript[Create the learning loop](~/cognitive-services-quickstart-code/javascript/Personalizer/sample.js?name=mainLoop)]

Observe detalladamente las llamadas de clasificación y recompensa en las secciones siguientes.

Agregue los métodos siguientes, que [obtienen las opciones de contenido](#get-content-choices-represented-as-actions), antes de ejecutar el archivo de código:

* getActionsList
* getContextFeaturesList

## <a name="request-the-best-action"></a>Solicitud de la mejor acción

Para completar la solicitud Rank, el programa solicita las preferencias del usuario para crear opciones de contenido. El proceso puede crear contenido para excluir de las acciones, que se muestra como `excludeActions`. Para recibir la respuesta clasificada, la solicitud Rank necesita las [acciones](../concepts-features.md#actions-represent-a-list-of-options) y sus características, las características de currentContext, excludeActions y un identificador de evento único.

Este inicio rápido tiene características de contexto simples de hora del día y preferencias alimentarias del usuario. En los sistemas de producción, la determinación y [evaluación](../concept-feature-evaluation.md) de [acciones y características](../concepts-features.md) no es una cuestión trivial.

[!code-javascript[The Personalizer learning loop ranks the request.](~/cognitive-services-quickstart-code/javascript/Personalizer/sample.js?name=rank)]

## <a name="send-a-reward"></a>Envío de una recompensa


Para obtener la puntuación de recompensa que se enviará en la solicitud Reward, el programa obtiene la selección del usuario desde la línea de comandos, asigna un valor numérico a la selección y, después, envía el identificador único del evento y la puntuación de recompensa como un valor numérico a API Reward.

En este inicio rápido se asigna un número simple como puntuación de recompensa: 0 o 1. En sistemas de producción, determinar cuándo y qué enviar a la llamada [Reward](../concept-rewards.md) puede ser más complicado, en función de las necesidades específicas.

[!code-javascript[The Personalizer learning loop sends a reward.](~/cognitive-services-quickstart-code/javascript/Personalizer/sample.js?name=reward)]

## <a name="run-the-program"></a>Ejecución del programa

Ejecute la aplicación con el comando de Node.js desde el directorio de aplicación.

```console
node sample.js
```

![El programa del inicio rápido realiza dos preguntas para recopilar las preferencias del usuario, conocidas como características, y, después, proporciona la acción principal.](../media/csharp-quickstart-commandline-feedback-loop/quickstart-program-feedback-loop-example.png)
