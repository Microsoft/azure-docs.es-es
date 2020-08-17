---
title: 'Uso de Personalizer en un bot de chat: Personalizer'
description: Personalice un bot de chat de .NET en C# con un bucle de Personalizer que proporcione el contenido correcto a un usuario en función de acciones (con características) y de características del contexto.
ms.topic: tutorial
ms.date: 07/17/2020
ms.openlocfilehash: 8e936b4017ad50434bc7d6b05b4217f82a9080e0
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87131674"
---
# <a name="tutorial-use-personalizer-in-net-chat-bot"></a>Tutorial: Uso de Personalizer en un bot de chat de .NET

Utilice un bot de chat de .NET en C# con un bucle de Personalizer para proporcionar el contenido correcto a un usuario. Este bot de chat sugiere un café o un té determinado a un usuario. El usuario puede aceptar o rechazar la sugerencia. Esto proporciona información a Personalizer para ayudar a que la sugerencia siguiente sea más adecuada.

**En este tutorial, aprenderá a:**

<!-- green checkmark -->
> [!div class="checklist"]
> * Configuración de los recursos de Azure
> * Configurar y ejecutar un bot
> * Interactuar con el bot mediante el emulador de bots
> * Comprender dónde y cómo usa el bot Personalizer


## <a name="how-does-the-chat-bot-work"></a>¿Cómo funciona el bot de chat?

Un bot de chat es generalmente una aplicación que mantiene un diálogo con un usuario. Este bot de chat en concreto usa Personalizer para seleccionar la mejor acción (café o té) que ofrecer al usuario. Personalizer usa el aprendizaje de refuerzo para realizar esa selección.

El bot de chat debe administrar los turnos de la conversación. El bot de chat utiliza [Bot Framework](https://github.com/microsoft/botframework-sdk) para administrar la arquitectura del bot y la conversación, y el servicio de Cognitive Services [Language Understanding](../LUIS/index.yml) (LUIS) para comprender la intención del lenguaje natural del usuario.

El bot de chat es un sitio web con una ruta específica disponible para responder solicitudes, `http://localhost:3978/api/messages`. Puede usar el emulador de bots para interactuar visualmente con el bot de chat en ejecución mientras desarrolla un bot en su entorno local.

### <a name="user-interactions-with-the-bot"></a>Interacciones del usuario con el bot

Este es un bot de chat sencillo que permite introducir consultas de texto.

|El usuario escribe texto|El bot responde con texto|Descripción de la acción que realiza el bot para determinar el texto de la respuesta|
|--|--|--|
|No se escribe texto: el bot inicia la conversación.|`This is a simple chatbot example that illustrates how to use Personalizer. The bot learns what coffee or tea order is preferred by customers given some context information (such as weather, temperature, and day of the week) and information about the user.`<br>`To use the bot, just follow the prompts. To try out a new imaginary context, type “Reset” and a new one will be randomly generated.`<br>`Welcome to the coffee bot, please tell me if you want to see the menu or get a coffee or tea suggestion for today. Once I’ve given you a suggestion, you can reply with ‘like’ or ‘don’t like’. It’s Tuesday today and the weather is Snowy.`|El bot inicia la conversación con un texto informativo y describe el contexto: `Tuesday`, `Snowy`.|
|`Show menu`|`Here is our menu: Coffee: Cappuccino Espresso Latte Macchiato Mocha Tea: GreenTea Rooibos`|Determina la intención de la consulta mediante LUIS y, a continuación, muestra las opciones de café y té del menú. Las características de las acciones son |
|`What do you suggest`|`How about Latte?`|Determina la intención de la consulta mediante LUIS y, a continuación, llama a la **API Rank** y muestra la opción superior como una pregunta `How about {response.RewardActionId}?`. También muestra la llamada y respuesta JSON con fines ilustrativos.|
|`I like it`|`That’s great! I’ll keep learning your preferences over time.`<br>`Would you like to get a new suggestion or reset the simulated context to a new day?`|Determina la intención de la consulta mediante LUIS y, a continuación, llama a la **API Reward** con la recompensa de `1` y muestra la llamada y la respuesta JSON con fines ilustrativos.|
|`I don't like it`|`Oh well, maybe I’ll guess better next time.`<br>`Would you like to get a new suggestion or reset the simulated context to a new day?`|Determina la intención de la consulta mediante LUIS y, a continuación, llama a la **API Reward** con la recompensa de `0` y muestra la llamada y la respuesta JSON con fines ilustrativos.|
|`Reset`|Devuelve texto informativo.|Determina la intención de la consulta mediante LUIS y, a continuación, muestra el texto informativo y restablece el contexto.|


### <a name="personalizer-in-this-bot"></a>Personalizer en este bot

Este bot de chat utiliza Personalizer para seleccionar la acción principal (café o té específico) en función de una lista de _acciones_ (cierto tipo de contenido) y características de contexto.

El bot envía la lista de acciones, junto con las características de contexto, al bucle de Personalizer. Personalizer devuelve la acción más adecuada al bot que, a su vez, la mostrará.

En este tutorial, las **acciones** son tipos de café y té:

|Café|Té|
|--|--|
|Cappuccino<br>Espresso<br>Latte<br>Mocca|Té verde<br>Rooibos|

**API Rank:** Para ayudar a Personalizer a obtener información sobre las acciones, el bot envía lo siguiente con cada solicitud de la API Rank:

* Acciones _con características_
* Características de contexto

Una **característica** del modelo es la información sobre la acción o el contexto que se puede agregar (agrupar) entre los miembros de la base de usuarios del bot de chat. Una característica _no_ específica de forma individual (como un identificador de usuario) o muy específica (por ejemplo, una hora exacta del día).

Las características se usan para alinear acciones con el contexto actual del modelo. El modelo es una representación del conocimiento anterior de Personalizer sobre las acciones, el contexto y sus características que le permiten tomar decisiones fundadas.

El modelo, incluidas las características, se actualiza según una programación basada en la configuración de la **frecuencia de actualización del modelo** en Azure Portal.

Las características deben seleccionarse con la misma planeación y diseño que se aplicarían a cualquier esquema o modelo de la arquitectura técnica. Los valores de las características se pueden establecer con lógica de negocios o con sistemas de terceros.

> [!CAUTION]
> Las características de esta aplicación son para demostración y puede que no sean necesariamente las más adecuadas para su aplicación web en su caso de uso.

#### <a name="action-features"></a>Características de acción

Cada acción (elemento de contenido) tiene características que ayudan a distinguir el elemento de café o té.

Las características no se configuran como parte de la configuración del bucle en Azure Portal. En vez de eso, se envían como un objeto JSON con cada llamada de la API Rank. Esto proporciona cierta flexibilidad para que las acciones y sus características crezcan, cambien o se reduzcan con el tiempo, lo que permite a Personalizer seguir las tendencias.

Las características de café y té incluyen:

* Lugar de origen del grano de café, como Kenia y Brasil
* ¿Es el café o el té de origen ecológico?
* Tueste claro u oscuro del café

Aunque el café tiene tres características en la lista anterior, el té solo tiene una. Pase solo características a Personalizer que tengan sentido para la acción. No pase un valor vacío para una característica si no es aplicable a la acción.

#### <a name="context-features"></a>Características de contexto

Las características de contexto ayudan a Personalizer a comprender el contexto del entorno, como el dispositivo de visualización, el usuario, la ubicación y otras características que son pertinentes para su caso de uso.

El contexto de este bot de chat incluye:

* Condiciones meteorológicas (nieve, lluvia, soleado)
* Día de la semana

La selección de características se aleatoriza en este bot de chat. En un bot real, use datos reales para las características de contexto.

### <a name="design-considerations-for-this-bot"></a>Consideraciones de diseño para este bot

Hay algunas precauciones que se deben tener en cuenta sobre esta conversación:
* **Interacción del bot**: la conversación es muy sencilla porque demuestra el rango y la recompensa en un caso de uso simple. No demuestra la funcionalidad completa del SDK de Bot Framework o del emulador.
* **Personalizer**: las características se seleccionan de forma aleatoria para simular el uso. No aleatorice características en un escenario de Personalizer de producción.
* **Language Understanding (LUIS)** : las pocas expresiones de ejemplo del modelo de LUIS están pensadas para este ejemplo únicamente. No utilice tan pocas expresiones de ejemplo en su aplicación de LUIS de producción.


## <a name="install-required-software"></a>Instalación del software necesario
- [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/). El repositorio de ejemplos descargables incluye instrucciones si prefiere usar la CLI de .NET Core.
- [Microsoft Bot Framework Emulator](https://aka.ms/botframeworkemulator) es una aplicación de escritorio que permite que los desarrolladores de bots prueben y depuren sus bots en localhost o mediante la ejecución remota mediante un túnel.

## <a name="download-the-sample-code-of-the-chat-bot"></a>Descargue el código de ejemplo del bot de chat.

El bot de chat está disponible en el repositorio de ejemplos de Personalizer. Clone o [descargue](https://github.com/Azure-Samples/cognitive-services-personalizer-samples/archive/master.zip) el repositorio y, a continuación, abra el ejemplo del directorio `/samples/ChatbotExample` con Visual Studio 2019.

Para clonar el repositorio, use el siguiente comando de Git en un shell de Bash (terminal).

```bash
git clone https://github.com/Azure-Samples/cognitive-services-personalizer-samples.git
```

## <a name="create-and-configure-personalizer-and-luis-resources"></a>Creación y configuración de recursos de Personalizer y LUIS

### <a name="create-azure-resources"></a>Creación de recursos de Azure

Para usar este bot de chat, debe crear recursos de Azure para Personalizer y Language Understanding (LUIS).

* [Cree recursos de LUIS](../luis/luis-how-to-azure-subscription.md#create-luis-resources-in-azure-portal). Seleccione **ambos** en el paso de creación porque necesita tanto recursos de creación como de predicción.
* [Cree el recurso de Personalizer](how-to-create-resource.md) y, a continuación, copie la clave y el punto de conexión de Azure Portal. Deberá establecer estos valores en el archivo `appsettings.json` del proyecto de .NET.

### <a name="create-luis-app"></a>Creación de una aplicación de LUIS

Si no está familiarizado con LUIS, debe [iniciar sesión](https://www.luis.ai) y migrar inmediatamente su cuenta. No es necesario crear nuevos recursos. En su lugar, seleccione los recursos que creó en la sección anterior de este tutorial.

1. Para crear una nueva aplicación de LUIS, en el [portal de LUIS](https://www.luis.ai), seleccione su suscripción y recurso de creación.
1. A continuación, en la misma página, seleccione **+ Nueva aplicación para la conversación** y, después, seleccione **Importar como JSON**.
1. En el cuadro de diálogo emergente, seleccione **Elegir archivo** y, a continuación, seleccione el archivo `/samples/ChatbotExample/CognitiveModels/coffeebot.json`. Escriba el nombre `Personalizer Coffee bot`.
1. En la barra de navegación superior derecha del portal de LUIS, seleccione el botón **Entrenar**.
1. Seleccione el botón **Publicar** para publicar la aplicación en el **Espacio de producción** para el tiempo de ejecución de predicción.
1. Seleccione **Administrar** y, después, **Configuración**. Copie el valor de **Id. de aplicación**. Deberá establecer este valor en el archivo `appsettings.json` del proyecto de .NET.
1. En la misma sección **Administrar**, seleccione **Recursos de Azure**. Esto muestra los recursos asociados en la aplicación.
1. Seleccione **Agregar recurso de predicción**. En el cuadro de diálogo emergente, seleccione su suscripción y el recurso de predicción creado en una sección anterior de este tutorial y, después, seleccione **Listo**.
1. Copie los valores de la **Clave principal** y de la **Dirección URL de punto de conexión**. Deberá establecer estos valores en el archivo `appsettings.json` del proyecto de .NET.

### <a name="configure-bot-with-appsettingsjson-file"></a>Configuración del bot con el archivo appsettings.json

1. Abra el archivo de la solución del bot de chat, `ChatbotSamples.sln`, con Visual Studio 2019.
1. Abra `appsettings.json` en el directorio raíz del proyecto.
1. Establezca las cinco configuraciones copiadas en la sección anterior de este tutorial.

    ```json
    {
      "PersonalizerChatbot": {
        "LuisAppId": "",
        "LuisAPIKey": "",
        "LuisServiceEndpoint": "",
        "PersonalizerServiceEndpoint": "",
        "PersonalizerAPIKey": ""
      }
    }
    ```

### <a name="build-and-run-the-bot"></a>Compilación y ejecución del bot

Una vez configurado el archivo `appsettings.json`, estará listo para compilar y ejecutar el bot de chat. Al hacerlo, en un explorador se abrirá el sitio web en ejecución: `http://localhost:3978`.

:::image type="content" source="media/tutorial-chat-bot/chat-bot-web-site.png" alt-text="Captura de pantalla del explorador que muestra el sitio web del bot de chat.":::

Deje el sitio web en ejecución, ya que en el tutorial se explica qué está haciendo el bot, de modo que pueda interactuar con él.


## <a name="set-up-the-bot-emulator"></a>Configuración del emulador de bot

1. Abra Bot Emulator y seleccione **Open Bot** (Abrir bot).

    :::image type="content" source="media/tutorial-chat-bot/bot-emulator-startup.png" alt-text="Captura de la pantalla de inicio del emulador de bot.":::


1. Configure el bot con la siguiente **dirección URL del bot** y seleccione **Connect** (Conectar):

    `http://localhost:3978/api/messages`

    :::image type="content" source="media/tutorial-chat-bot/bot-emulator-open-bot-settings.png" alt-text="Captura de pantalla de la configuración de bot abierta en el emulador de bot.":::

    El emulador se conecta al bot de chat y muestra el texto informativo, junto con información de registro y depuración que resulta útil para el desarrollo local.

    :::image type="content" source="media/tutorial-chat-bot/bot-emulator-bot-conversation-first-turn.png" alt-text="Captura de pantalla del emulador de bot en el primer turno de la conversación.":::

## <a name="use-the-bot-in-the-bot-emulator"></a>Uso del bot en el emulador de bot

1. Escriba `I would like to see the menu` para solicitar ver el menú. El bot de chat muestra los elementos.
1. Escriba `Please suggest a drink for me.` para que el bot sugiera un elemento. El emulador muestra la solicitud y la respuesta de clasificación en la ventana de chat, por lo que puede ver el JSON completo. El bot realizará una sugerencia, como `How about Latte?`.
1. Responda que esa sugerencia le parece bien, es decir, que acepta la selección mejor clasificada de Personalizer: `I like it.`. El emulador muestra la solicitud de recompensa con la puntuación de recompensa 1 y la respuesta en la ventana de chat, por lo que puede ver el JSON completo. El bot responde con `That’s great! I’ll keep learning your preferences over time.` y `Would you like to get a new suggestion or reset the simulated context to a new day?`:

    Si responde con `no` a la selección, se envía a Personalizer la puntuación de recompensa de 0.


## <a name="understand-the-net-code-using-personalizer"></a>Descripción del código de .NET con Personalizer

La solución de .NET es un bot de chat de Bot Framework sencillo. El código relacionado con Personalizer se encuentra en las siguientes carpetas:
* `/samples/ChatbotExample/Bots`
    * Archivo `PersonalizerChatbot.cs` para la interacción entre el bot y Personalizer.
* `/samples/ChatbotExample/ReinforcementLearning`: administra las acciones y características del modelo de Personalizer.
* `/samples/ChatbotExample/Model`: archivos para las acciones y características de Personalizer, así como para las intenciones de LUIS

### <a name="personalizerchatbotcs---working-with-personalizer"></a>PersonalizerChatbot.cs: trabajar con Personalizer

La clase `PersonalizerChatbot` se deriva de `Microsoft.Bot.Builder.ActivityHandler`. Tiene tres propiedades y métodos para administrar el flujo de conversación.

> [!CAUTION]
> No copie el código de este tutorial. Utilice el código de ejemplo del [repositorio de ejemplos de Personalizer](https://github.com/Azure-Samples/cognitive-services-personalizer-samples).

```csharp
public class PersonalizerChatbot : ActivityHandler
{

    private readonly LuisRecognizer _luisRecognizer;
    private readonly PersonalizerClient _personalizerClient;

    private readonly RLContextManager _rlFeaturesManager;

    public PersonalizerChatbot(LuisRecognizer luisRecognizer, RLContextManager rlContextManager, PersonalizerClient personalizerClient)
            {
                _luisRecognizer = luisRecognizer;
                _rlFeaturesManager = rlContextManager;
                _personalizerClient = personalizerClient;
            }
    }

    public override async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        await base.OnTurnAsync(turnContext, cancellationToken);

        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            // Check LUIS model
            var recognizerResult = await _luisRecognizer.RecognizeAsync(turnContext, cancellationToken);
            var topIntent = recognizerResult?.GetTopScoringIntent();
            if (topIntent != null && topIntent.HasValue && topIntent.Value.intent != "None")
            {
                Intents intent = (Intents)Enum.Parse(typeof(Intents), topIntent.Value.intent);
                switch (intent)
                {
                    case Intents.ShowMenu:
                        await turnContext.SendActivityAsync($"Here is our menu: \n Coffee: {CoffeesMethods.DisplayCoffees()}\n Tea: {TeaMethods.DisplayTeas()}", cancellationToken: cancellationToken);
                        break;
                    case Intents.ChooseRank:
                        // Here we generate the event ID for this Rank.
                        var response = await ChooseRankAsync(turnContext, _rlFeaturesManager.GenerateEventId(), cancellationToken);
                        _rlFeaturesManager.CurrentPreference = response.Ranking;
                        await turnContext.SendActivityAsync($"How about {response.RewardActionId}?", cancellationToken: cancellationToken);
                        break;
                    case Intents.RewardLike:
                        if (!string.IsNullOrEmpty(_rlFeaturesManager.CurrentEventId))
                        {
                            await RewardAsync(turnContext, _rlFeaturesManager.CurrentEventId, 1, cancellationToken);
                            await turnContext.SendActivityAsync($"That's great! I'll keep learning your preferences over time.", cancellationToken: cancellationToken);
                            await SendByebyeMessageAsync(turnContext, cancellationToken);
                        }
                        else
                        {
                            await turnContext.SendActivityAsync($"Not sure what you like. Did you ask for a suggestion?", cancellationToken: cancellationToken);
                        }

                        break;
                    case Intents.RewardDislike:
                        if (!string.IsNullOrEmpty(_rlFeaturesManager.CurrentEventId))
                        {
                            await RewardAsync(turnContext, _rlFeaturesManager.CurrentEventId, 0, cancellationToken);
                            await turnContext.SendActivityAsync($"Oh well, maybe I'll guess better next time.", cancellationToken: cancellationToken);
                            await SendByebyeMessageAsync(turnContext, cancellationToken);
                        }
                        else
                        {
                            await turnContext.SendActivityAsync($"Not sure what you dislike. Did you ask for a suggestion?", cancellationToken: cancellationToken);
                        }

                        break;
                    case Intents.Reset:
                        _rlFeaturesManager.GenerateRLFeatures();
                        await SendResetMessageAsync(turnContext, cancellationToken);
                        break;
                    default:
                        break;
                }
            }
            else
            {
                var msg = @"Could not match your message with any of the following LUIS intents:
                        'ShowMenu'
                        'ChooseRank'
                        'RewardLike'
                        'RewardDislike'.
                        Try typing 'Show me the menu','What do you suggest','I like it','I don't like it'.";
                await turnContext.SendActivityAsync(msg);
            }
        }
        else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
        {
            // Generate a new weekday and weather condition
            // These will act as the context features when we call rank with Personalizer
            _rlFeaturesManager.GenerateRLFeatures();

            // Send a welcome message to the user and tell them what actions they may perform to use this bot
            await SendWelcomeMessageAsync(turnContext, cancellationToken);
        }
        else
        {
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected", cancellationToken: cancellationToken);
        }
    }

    // code removed for brevity, full sample code available for download
    private async Task SendWelcomeMessageAsync(ITurnContext turnContext, CancellationToken cancellationToken)
    private async Task SendResetMessageAsync(ITurnContext turnContext, CancellationToken cancellationToken)
    private async Task SendByebyeMessageAsync(ITurnContext turnContext, CancellationToken cancellationToken)
    private async Task<RankResponse> ChooseRankAsync(ITurnContext turnContext, string eventId, CancellationToken cancellationToken)
    private async Task RewardAsync(ITurnContext turnContext, string eventId, double reward, CancellationToken cancellationToken)
}
```

Los métodos precedidos por `Send` administran la conversación con el bot y LUIS. Los métodos `ChooseRankAsync` y `RewardAsync` interactúan con Personalizer.

#### <a name="calling-rank-api-and-display-results"></a>Llamada a la API Rank y visualización de los resultados

El método `ChooseRankAsync` compila los datos JSON que se van a enviar a la API Rank de Personalizer mediante la recopilación de las acciones con características y las características de contexto.

```csharp
private async Task<RankResponse> ChooseRankAsync(ITurnContext turnContext, string eventId, CancellationToken cancellationToken)
{
    IList<object> contextFeature = new List<object>
    {
        new { weather = _rlFeaturesManager.RLFeatures.Weather.ToString() },
        new { dayofweek = _rlFeaturesManager.RLFeatures.DayOfWeek.ToString() },
    };

    Random rand = new Random(DateTime.UtcNow.Millisecond);
    IList<RankableAction> actions = new List<RankableAction>();
    var coffees = Enum.GetValues(typeof(Coffees));
    var beansOrigin = Enum.GetValues(typeof(CoffeeBeansOrigin));
    var organic = Enum.GetValues(typeof(Organic));
    var roast = Enum.GetValues(typeof(CoffeeRoast));
    var teas = Enum.GetValues(typeof(Teas));

    foreach (var coffee in coffees)
    {
        actions.Add(new RankableAction
        {
            Id = coffee.ToString(),
            Features =
            new List<object>()
            {
                new { BeansOrigin = beansOrigin.GetValue(rand.Next(0, beansOrigin.Length)).ToString() },
                new { Organic = organic.GetValue(rand.Next(0, organic.Length)).ToString() },
                new { Roast = roast.GetValue(rand.Next(0, roast.Length)).ToString() },
            },
        });
    }

    foreach (var tea in teas)
    {
        actions.Add(new RankableAction
        {
            Id = tea.ToString(),
            Features =
            new List<object>()
            {
                new { Organic = organic.GetValue(rand.Next(0, organic.Length)).ToString() },
            },
        });
    }

    // Sending a rank request to Personalizer
    // Here we are asking Personalizer to decide which drink the user is most likely to want
    // based on the current context features (weather, day of the week generated in RLContextManager)
    // and the features of the drinks themselves
    var request = new RankRequest(actions, contextFeature, null, eventId);
    await turnContext.SendActivityAsync(
        "===== DEBUG MESSAGE CALL TO RANK =====\n" +
        "This is what is getting sent to Rank:\n" +
        $"{JsonConvert.SerializeObject(request, Formatting.Indented)}\n",
        cancellationToken: cancellationToken);
    var response = await _personalizerClient.RankAsync(request, cancellationToken);
    await turnContext.SendActivityAsync(
        $"===== DEBUG MESSAGE RETURN FROM RANK =====\n" +
        "This is what Rank returned:\n" +
        $"{JsonConvert.SerializeObject(response, Formatting.Indented)}\n",
        cancellationToken: cancellationToken);
    return response;
}
```

#### <a name="calling-reward-api-and-display-results"></a>Llamada a la API Reward y visualización de los resultados

El método `RewardAsync` compila los datos JSON que se van a enviar a la API Reward de Personalizer mediante la determinación de la puntuación La puntuación se determina a partir de la intención de LUIS identificada en el texto del usuario y enviada desde el método `OnTurnAsync`.

```csharp
private async Task RewardAsync(ITurnContext turnContext, string eventId, double reward, CancellationToken cancellationToken)
{
    await turnContext.SendActivityAsync(
        "===== DEBUG MESSAGE CALL REWARD =====\n" +
        "Calling Reward:\n" +
        $"eventId = {eventId}, reward = {reward}\n",
        cancellationToken: cancellationToken);

    // Sending a reward request to Personalizer
    // Here we are responding to the drink ranking Personalizer provided us
    // If the user liked the highest ranked drink, we give a high reward (1)
    // If they did not, we give a low reward (0)
    await _personalizerClient.RewardAsync(eventId, new RewardRequest(reward), cancellationToken);
}
```

## <a name="design-considerations-for-a-bot"></a>Consideraciones de diseño de un bot

Este ejemplo está pensado para mostrar una solución sencilla de un extremo a otro de Personalizer en un bot. Es posible que su caso de uso sea más complejo.

Si piensa usar Personalizer en un bot de producción, planee lo siguiente:
* Acceso en tiempo real a Personalizer _cada vez_ que necesite una selección clasificada. La API Rank no se puede procesar por lotes ni almacenar en caché.  La llamada de recompensa se puede retrasar o descargar en un proceso independiente y, si no se devuelve una recompensa en el período de tiempo determinado, se establece un valor de recompensa predeterminado para el evento.
* Cálculo de la recompensa basado en casos de uso: en este ejemplo se han mostrado dos recompensas de cero y uno sin ningún intervalo entre ellas y sin valores negativos para una puntuación. Es posible que su sistema necesite una puntuación más pormenorizada.
* Canales de bot: En este ejemplo se usa un solo canal, pero si piensa usar más de un canal o variaciones de bots en un solo canal, es posible que tengan que considerarse como parte de las características de contexto del modelo de Personalizer.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado este tutorial, elimine los recursos siguientes:

* Elimine el directorio del proyecto de ejemplo.
* Elimine el recurso de Personalizer y LUIS en Azure Portal.

## <a name="next-steps"></a>Pasos siguientes
* [Funcionamiento de Personalizer](how-personalizer-works.md)
* [Características](concepts-features.md): aprenda conceptos acerca de las características con las acciones y el contexto.
* [Recompensas](concept-rewards.md): obtenga información sobre el cálculo de recompensas.
