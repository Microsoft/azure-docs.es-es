---
title: archivo de inclusión
description: archivo de inclusión
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: include
ms.custom: include file
ms.date: 07/30/2020
ms.openlocfilehash: 55bc13caec8a48ed058ea38bdc0ebe487839eef3
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87461141"
---
[Documentación de referencia](https://docs.microsoft.com/dotnet/api/Microsoft.Azure.CognitiveServices.Personalizer?view=azure-dotnet-preview) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Personalizer) | [Paquete (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Personalizer/) | [Ejemplos](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/dotnet/Personalizer)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)
* La versión actual de [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).

## <a name="using-this-quickstart"></a>Uso de este inicio rápido

Hay que dar varios pasos para usar este inicio rápido:

* En Azure Portal, cree un recurso de Personalizer
* En la página **Configuración** del recurso Personalizer de Azure Portal, cambie la frecuencia de actualización del modelo a un intervalo muy corto.
* En un editor de código, cree un archivo de código y edítelo
* En la línea de comandos o en el terminal, instale el SDK desde la línea de comandos
* En la línea de comandos o en el terminal, ejecute el archivo de código

[!INCLUDE [Create Azure resource for Personalizer](create-personalizer-resource.md)]

[!INCLUDE [!Change model frequency](change-model-frequency.md)]

## <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

Cree una nueva aplicación de consola de .NET Core en el IDE o editor que prefiera.

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `new` dotnet para crear una nueva aplicación de consola con el nombre `personalizer-quickstart`. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: `Program.cs`.

```console
dotnet new console -n personalizer-quickstart
```

Cambie el directorio a la carpeta de aplicaciones recién creada. Para compilar la aplicación:

```console
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

## <a name="install-the-sdk"></a>Instalación del SDK

Dentro del directorio de aplicaciones, instale la biblioteca cliente de Personalizer para .NET con el siguiente comando:

```console
dotnet add package Microsoft.Azure.CognitiveServices.Personalizer --version 0.8.0-preview
```

Si usa el IDE de Visual Studio, la biblioteca cliente estará disponible como un paquete de NuGet descargable.

## <a name="object-model"></a>Modelo de objetos

El cliente de Personalizer es un objeto [PersonalizerClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.personalizer.personalizerclient?view=azure-dotnet) que se autentica en Azure mediante Microsoft.Rest.ServiceClientCredentials, que contiene la clave.

Para solicitar el mejor elemento del contenido, cree un elemento [RankRequest](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.personalizer.models.rankrequest?view=azure-dotnet-preview) y, luego, páselo al método [client.Rank](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.personalizer.personalizerclientextensions.rank?view=azure-dotnet-preview). El método Rank devuelve un elemento RankResponse.

Para enviar una puntuación de recompensa a Personalizer, cree un objeto [RewardRequest](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.personalizer.models.rewardrequest?view=azure-dotnet-preview) y, luego, páselo al método [client.Reward](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.personalizer.personalizerclientextensions.reward?view=azure-dotnet-preview).

La determinación de la puntuación de recompensa en este inicio rápido no es importante. En un sistema de producción, la determinación de lo que afecta a la [puntuación de recompensa](../concept-rewards.md) y cuánto le afecta, puede ser un proceso complejo que decida cambiar con el tiempo. Esta debe ser una de las decisiones de diseño principales en la arquitectura de Personalizer.

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo realizar las siguientes tareas con la biblioteca cliente de Personalizer para .NET:

* [Creación de un cliente de Personalizer](#create-a-personalizer-client)
* [API Rank](#request-the-best-action)
* [API Reward](#send-a-reward)

## <a name="add-the-dependencies"></a>Adición de las dependencias

En el directorio del proyecto, abra el archivo **Program.cs** en el editor o IDE que prefiera. Reemplace el código existente `using` por las siguientes directivas `using`:

[!code-csharp[Using statements](~/cognitive-services-quickstart-code/dotnet/Personalizer/Program.cs?name=Dependencies)]

## <a name="add-personalizer-resource-information"></a>Incorporación de la información de recursos de Personalizer

En la clase **Program**, edite las variables key y endpoint del principio del archivo de código para la clave y el punto de conexión de Azure de su recurso. 

[!code-csharp[Create variables to hold the Personalizer resource key and endpoint values found in the Azure portal.](~/cognitive-services-quickstart-code/dotnet/Personalizer/Program.cs?name=classVariables)]

## <a name="create-a-personalizer-client"></a>Creación de un cliente de Personalizer

A continuación, cree un método para devolver un cliente de Personalizer. El parámetro para el método es `PERSONALIZER_RESOURCE_ENDPOINT` y el elemento ApiKey es `PERSONALIZER_RESOURCE_KEY`.

[!code-csharp[Create the Personalizer client](~/cognitive-services-quickstart-code/dotnet/Personalizer/Program.cs?name=authorization)]

## <a name="get-food-items-as-rankable-actions"></a>Obtención de elementos alimenticios como acciones clasificables

Las acciones representan las opciones de contenido entre las que quiere que Personalizer seleccione el mejor elemento de contenido. Agregue los métodos siguientes a la clase Program para representar el conjunto de acciones y sus características. 

[!code-csharp[Food items as actions](~/cognitive-services-quickstart-code/dotnet/Personalizer/Program.cs?name=createAction)]

## <a name="get-user-preferences-for-context"></a>Obtención de las preferencias del usuario para el contexto

Agregue los métodos siguientes a la clase Program para obtener una entrada de usuario desde la línea de comandos para la hora del día y la preferencia alimentaria actual. Se usarán como características de contexto.

[!code-csharp[Present time out day preference to the user](~/cognitive-services-quickstart-code/dotnet/Personalizer/Program.cs?name=createUserFeatureTimeOfDay)]

[!code-csharp[Present food taste preference to the user](~/cognitive-services-quickstart-code/dotnet/Personalizer/Program.cs?name=createUserFeatureTastePreference)]

Ambos métodos usan el método `GetKey` para leer la selección del usuario desde la línea de comandos.

[!code-csharp[Read user's choice from the command line](~/cognitive-services-quickstart-code/dotnet/Personalizer/Program.cs?name=readCommandLine)]

## <a name="create-the-learning-loop"></a>Creación del bucle de aprendizaje

El bucle de aprendizaje de Personalizer es un ciclo de llamadas [Rank](#request-the-best-action) y [Reward](#send-a-reward). En este inicio rápido, cada llamada Rank para personalizar el contenido, va seguida de una llamada Reward para indicar a Personalizer cómo es de eficaz el funcionamiento del servicio.

El siguiente código pasa por un ciclo en el que se pregunta al usuario sus preferencias en la línea de comandos, se envía esa información a Personalizer para que seleccione la mejor acción, se presenta la selección al cliente para que elija de la lista y, después, se envía una puntuación de recompensa a Personalizer en la que se señala el grado de acierto que tuvo el servicio al clasificar la selección.

[!code-csharp[Learning loop](~/cognitive-services-quickstart-code/dotnet/Personalizer/Program.cs?name=mainLoop)]

Agregue los métodos siguientes, que [obtienen las opciones de contenido](#get-food-items-as-rankable-actions), antes de ejecutar el archivo de código:

* `GetActions`
* `GetUsersTimeOfDay`
* `GetUsersTastePreference`
* `GetKey`

## <a name="request-the-best-action"></a>Solicitud de la mejor acción

Para completar la solicitud Rank, el programa solicita las preferencias del usuario para crear un elemento `currentContent` de las opciones de contenido. El proceso puede crear contenido para excluir de las acciones, que se muestra como `excludeActions`. Para recibir la respuesta, la solicitud Rank necesita las acciones y sus características, las características de currentContext, excludeActions y un identificador de evento único.

Este inicio rápido tiene características de contexto simples de hora del día y preferencias alimentarias del usuario. En los sistemas de producción, la determinación y [evaluación](../concept-feature-evaluation.md) de [acciones y características](../concepts-features.md) no es una cuestión trivial.

[!code-csharp[The Personalizer learning loop ranks the request.](~/cognitive-services-quickstart-code/dotnet/Personalizer/Program.cs?name=rank)]

## <a name="send-a-reward"></a>Envío de una recompensa

Para obtener la puntuación de recompensa que se enviará en la solicitud Reward, el programa obtiene la selección del usuario desde la línea de comandos, asigna un valor numérico a la selección y, después, envía el identificador único del evento y la puntuación de recompensa como un valor numérico a API Reward.

En este inicio rápido se asigna un número simple como puntuación de recompensa: 0 o 1. En sistemas de producción, determinar cuándo y qué enviar a la llamada [Reward](../concept-rewards.md) puede ser más complicado, en función de las necesidades específicas.

[!code-csharp[The Personalizer learning loop ranks the request.](~/cognitive-services-quickstart-code/dotnet/Personalizer/Program.cs?name=reward)]

## <a name="run-the-program"></a>Ejecución del programa

Ejecute la aplicación con el comando `run` dotnet desde el directorio de aplicaciones.

```console
dotnet run
```

![El programa del inicio rápido realiza dos preguntas para recopilar las preferencias del usuario, conocidas como características, y, después, proporciona la acción principal.](../media/csharp-quickstart-commandline-feedback-loop/quickstart-program-feedback-loop-example.png)

El [código fuente de este inicio rápido](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/dotnet/Personalizer) está disponible.
