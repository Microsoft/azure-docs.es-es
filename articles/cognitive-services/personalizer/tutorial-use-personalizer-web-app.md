---
title: 'Uso de una aplicación web: Personalizer'
description: Personalice una aplicación web de .NET en C# con un bucle de Personalizer que proporcione el contenido correcto a un usuario en función de las acciones (con características) y de las características del contexto.
ms.topic: tutorial
ms.date: 06/10/2020
ms.openlocfilehash: 6e3373ef8ace401ff40b3d48026fc46b96f1ad79
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87446366"
---
# <a name="tutorial-add-personalizer-to-a-net-web-app"></a>Tutorial: Incorporación de Personalizer a una aplicación web de .NET

Personalice una aplicación web de .NET en C# con un bucle de Personalizer que proporcione el contenido correcto a un usuario en función de las acciones (con características) y de las características del contexto.

**En este tutorial, aprenderá a:**

<!-- green checkmark -->
> [!div class="checklist"]
> * Configurar la clave y el punto de conexión de Personalizer
> * Recopilar características
> * Llamar a las API Rank y Reward
> * Mostrar la acción superior, que se designa como _rewardActionId_



## <a name="select-the-best-content-for-a-web-app"></a>Selección del mejor contenido para una aplicación web

Una aplicación web debe usar Personalizer cuando hay una lista de _acciones_ (algún tipo de contenido) en la página web que debe personalizarse para que se muestre un único elemento superior (rewardActionId). Entre los ejemplos de listas de acciones se incluyen artículos de noticias, ubicaciones de botones y opciones de palabras para nombres de producto.

Envíe la lista de acciones, junto con las características de contexto, al bucle de Personalizer. Personalizer selecciona la acción más adecuada y, a continuación, la aplicación web muestra esa acción.

En este tutorial, las acciones son tipos de comida:

* pasta
* helado
* zumo
* ensalada
* palomitas
* café
* sopa

Para ayudar a Personalizer a obtener información sobre sus acciones, envíe tanto las _acciones con características_ como las _características de contexto_ con cada solicitud de API Rank.

Una **característica** del modelo es la información sobre la acción o el contexto que se puede agregar (agrupar) entre los miembros de la base de usuarios de la aplicación web. Una característica _no_ específica de forma individual (como un identificador de usuario) o muy específica (por ejemplo, una hora exacta del día).

### <a name="actions-with-features"></a>Acciones con características

Cada acción (elemento de contenido) tiene características que ayudan a distinguir el elemento de comida.

Las características no se configuran como parte de la configuración del bucle en Azure Portal. En vez de eso, se envían como un objeto JSON con cada llamada de la API Rank. Esto proporciona cierta flexibilidad para que las acciones y sus características crezcan, cambien o se reduzcan con el tiempo, lo que permite a Personalizer seguir las tendencias.

```csharp
 /// <summary>
  /// Creates personalizer actions feature list.
  /// </summary>
  /// <returns>List of actions for personalizer.</returns>
  private IList<RankableAction> GetActions()
  {
      IList<RankableAction> actions = new List<RankableAction>
      {
          new RankableAction
          {
              Id = "pasta",
              Features =
              new List<object>() { new { taste = "savory", spiceLevel = "medium" }, new { nutritionLevel = 5, cuisine = "italian" } }
          },

          new RankableAction
          {
              Id = "ice cream",
              Features =
              new List<object>() { new { taste = "sweet", spiceLevel = "none" }, new { nutritionalLevel = 2 } }
          },

          new RankableAction
          {
              Id = "juice",
              Features =
              new List<object>() { new { taste = "sweet", spiceLevel = "none" }, new { nutritionLevel = 5 }, new { drink = true } }
          },

          new RankableAction
          {
              Id = "salad",
              Features =
              new List<object>() { new { taste = "sour", spiceLevel = "low" }, new { nutritionLevel = 8 } }
          },

          new RankableAction
          {
              Id = "popcorn",
              Features =
              new List<object>() { new { taste = "salty", spiceLevel = "none" }, new { nutritionLevel = 3 } }
          },

          new RankableAction
          {
              Id = "coffee",
              Features =
              new List<object>() { new { taste = "bitter", spiceLevel = "none" }, new { nutritionLevel = 3 }, new { drink = true } }
          },

          new RankableAction
          {
              Id = "soup",
              Features =
              new List<object>() { new { taste = "sour", spiceLevel = "high" }, new { nutritionLevel =  7} }
          }
      };

      return actions;
  }
```


## <a name="context-features"></a>Características de contexto

Las características de contexto ayudan a Personalizer a entender el contexto de las acciones. El contexto de esta aplicación de ejemplo incluye:

* hora del día (mañana, mediodía, tarde, noche)
* preferencia del usuario para el sabor: salado, dulce, amargo, agrio o agridulce
* contexto del explorador: agente de usuario, ubicación geográfica, origen de referencia

```csharp
/// <summary>
/// Get users time of the day context.
/// </summary>
/// <returns>Time of day feature selected by the user.</returns>
private string GetUsersTimeOfDay()
{
    Random rnd = new Random();
    string[] timeOfDayFeatures = new string[] { "morning", "noon", "afternoon", "evening", "night", "midnight" };
    int timeIndex = rnd.Next(timeOfDayFeatures.Length);
    return timeOfDayFeatures[timeIndex];
}

/// <summary>
/// Gets user food preference.
/// </summary>
/// <returns>Food taste feature selected by the user.</returns>
private string GetUsersTastePreference()
{
    Random rnd = new Random();
    string[] tasteFeatures = new string[] { "salty", "bitter", "sour", "savory", "sweet" };
    int tasteIndex = rnd.Next(tasteFeatures.Length);
    return tasteFeatures[tasteIndex];
}
```

## <a name="how-does-the-web-app-use-personalizer"></a>¿Cómo usa la aplicación web Personalizer?

La aplicación web usa Personalizer para seleccionar la mejor acción de la lista de opciones de comida. Para ello, envía la siguiente información a cada llamada de la API Rank:
* **acciones** con sus características como `taste` y `spiceLevel`
* características de **contexto** como `time` del día, preferencias de `taste` del usuario y la información del agente de usuario del explorador.
* **acciones a excluir** como, por ejemplo, zumo
* **eventId**, que es diferente para cada llamada a la API Rank.

## <a name="personalizer-model-features-in-a-web-app"></a>Características del modelo de Personalizer en una aplicación web

Personalizer necesita características para las acciones (contenido) y el contexto actual (usuario y entorno). Las características se usan para alinear acciones con el contexto actual del modelo. El modelo es una representación del conocimiento anterior de Personalizer sobre las acciones, el contexto y sus características que le permiten tomar decisiones fundadas.

El modelo, incluidas las características, se actualiza según una programación basada en la configuración de la **frecuencia de actualización del modelo** en Azure Portal.

> [!CAUTION]
> Las características de esta aplicación están diseñadas para ilustrar las características y los valores de estas, pero no son necesariamente las mejores características a usar en una aplicación web.

### <a name="plan-for-features-and-their-values"></a>Planeación de características y sus valores

Las características deben seleccionarse con la misma planeación y diseño que se aplicarían a cualquier esquema o modelo de la arquitectura técnica. Los valores de las características se pueden establecer con lógica de negocios o con sistemas de terceros. Los valores de las características no deben ser tan específicos que no se puedan aplicar a un grupo o una clase de características.

### <a name="generalize-feature-values"></a>Generalización de los valores de las características

#### <a name="generalize-into-categories"></a>Generalización en categorías

Esta aplicación usa `time` como una característica pero agrupa el tiempo en categorías como `morning`, `afternoon`, `evening` y `night`. Este es un ejemplo del uso de una información de tiempo no excesivamente específica, como `10:05:01 UTC+2`.

#### <a name="generalize-into-parts"></a>Generalización en partes

Esta aplicación usa las características de solicitudes HTTP del explorador. Esto empieza con una cadena muy específica con todos los datos, por ejemplo:

```http
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/530.99 (KHTML, like Gecko) Chrome/80.0.3900.140 Safari/537.36
```

La biblioteca de clases **HttpRequestFeatures** generaliza esta cadena en un objeto **userAgentInfo** con valores individuales. Todos los valores que sean demasiado específicos se establecen en una cadena vacía. Cuando se envían las características de contexto de la solicitud, tienen el siguiente formato JSON:

```JSON
{
  "httpRequestFeatures": {
    "_synthetic": false,
    "OUserAgent": {
      "_ua": "",
      "_DeviceBrand": "",
      "_DeviceFamily": "Other",
      "_DeviceIsSpider": false,
      "_DeviceModel": "",
      "_OSFamily": "Windows",
      "_OSMajor": "10",
      "DeviceType": "Desktop"
    }
  }
}
```


## <a name="using-sample-web-app"></a>Uso de la aplicación web de ejemplo

La aplicación web basada en explorador (se proporciona todo el código) necesita la instalación de las siguientes aplicaciones para ejecutarse.

Instale el siguiente software:

* [.NET Core 2.1](https://dotnet.microsoft.com/download/dotnet-core/2.1): el servidor back-end de ejemplo usa .NET Core.
* [Node.js](https://nodejs.org/): el cliente o front-end depende de esta aplicación.
* [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) o la [CLI de .NET Core](https://docs.microsoft.com/dotnet/core/tools/): use el entorno para desarrolladores de Visual Studio 2019 o la CLI de .NET Core para compilar y ejecutar la aplicación.

### <a name="set-up-the-sample"></a>Configuración del ejemplo
1. Clone el repositorio de ejemplos de Azure Personalizer.

    ```bash
    git clone https://github.com/Azure-Samples/cognitive-services-personalizer-samples.git
    ```

1. Vaya a _samples/HttpRequestFeatures_ para abrir la solución `HttpRequestFeaturesExample.sln`.

    Si se le solicita, permita que Visual Studio actualice el paquete .NET de Personalizer.

### <a name="set-up-azure-personalizer-service"></a>Configuración del servicio Azure Personalizer

1. En Azure Portal, [cree un recurso de Personalizer](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesPersonalizer).

1. En Azure Portal, busque `Endpoint` y `Key1` o `Key2` (cualquiera de estos funcionará) en la pestaña **Claves y puntos de conexión**. En este caso, se trata de `PersonalizerServiceEndpoint` y `PersonalizerApiKey`.
1. Rellene `PersonalizerServiceEndpoint` en **appsettings.json**.
1. Configure `PersonalizerApiKey` como un [secreto de aplicación](https://docs.microsoft.com/aspnet/core/security/app-secrets) de una de las siguientes maneras:

    * Si va a utilizar la CLI de .NET Core, puede usar el comando `dotnet user-secrets set "PersonalizerApiKey" "<API Key>"`.
    * Si va a usar Visual Studio, puede hacer clic con el botón derecho en el proyecto y seleccionar la opción de menú **Administrar secretos de usuario** para configurar las claves de Personalizer. Al hacerlo, Visual Studio abrirá un archivo `secrets.json` en el que puede agregar las claves de la siguiente manera:

    ```JSON
    {
      "PersonalizerApiKey": "<your personalizer key here>",
    }
    ```

## <a name="run-the-sample"></a>Ejecución del ejemplo

Compila y ejecuta HttpRequestFeaturesExample con uno de los métodos siguientes:

* Visual Studio 2019: Presione **F5**.
* CLI de .NET Core: `dotnet build` y, después, `dotnet run`.

Mediante un explorador web, puede enviar una solicitud de Rank y una solicitud de Reward y ver sus respuestas, así como las características de la solicitud HTTP extraídas de su entorno.

> [!div class="mx-imgBorder"]
> ![Compile y ejecute el proyecto HTTPRequestFeaturesExample. Se abre una ventana del explorador que muestra la aplicación de página única.](./media/tutorial-web-app/web-app-single-page.png)

## <a name="demonstrate-the-personalizer-loop"></a>Demostración del bucle de Personalizer

1. Seleccione el botón **Generate new Rank Request** (Generar nueva solicitud de Rank) para crear un nuevo objeto JSON para la llamada de la API Rank. Con esto, se crean las acciones (con características) y las características de contexto y se muestran los valores para que pueda ver el aspecto del objeto JSON.

    Para su propia aplicación futura, la generación de acciones y características puede producirse en el cliente, en el servidor, en una combinación de los dos o con llamadas a otros servicios.

1. Seleccione **Send Rank Request** (Enviar solicitud de Rank) para enviar el objeto JSON al servidor. El servidor llama a la API Rank de Personalizer. El servidor recibe la respuesta y devuelve la acción con mejor clasificación al cliente para que la muestre.

1. Establezca el valor de la recompensa y, a continuación, seleccione el botón **Send Reward Request** (Enviar solicitud de Reward). Si no cambia el valor de la recompensa, la aplicación cliente siempre envía el valor de `1` a Personalizer.

    > [!div class="mx-imgBorder"]
    > ![Compile y ejecute el proyecto HTTPRequestFeaturesExample. Se abre una ventana del explorador que muestra la aplicación de página única.](./media/tutorial-web-app/reward-score-api-call.png)

    Para su propia aplicación futura, la generación de la puntuación de recompensa puede producirse después de recopilar información del comportamiento del usuario en el cliente junto con la lógica de negocios en el servidor.

## <a name="understand-the-sample-web-app"></a>Descripción de la aplicación web de ejemplo

La aplicación web de ejemplo tiene un servidor de **.NET para C#** , que administra la recopilación de características y el envío y recepción de llamadas HTTP al punto de conexión de Personalizer.

La aplicación web de ejemplo usa una **aplicación cliente de front-end para knockout** para capturar características y procesar acciones de la interfaz de usuario como hacer clic en los botones y enviar datos al servidor .NET.

En las secciones siguientes se explican las partes del servidor y el cliente que un desarrollador necesita comprender para usar Personalizer.

## <a name="rank-api-client-application-sends-context-to-server"></a>API Rank: La aplicación cliente envía el contexto al servidor

La aplicación cliente recopila el _agente de usuario_ del explorador del usuario.

> [!div class="mx-imgBorder"]
> ![Compile y ejecute el proyecto HTTPRequestFeaturesExample. Se abre una ventana del explorador que muestra la aplicación de página única.](./media/tutorial-web-app/user-agent.png)

## <a name="rank-api-server-application-calls-personalizer"></a>API Rank: La aplicación de servidor llama a Personalizer

Esta es una aplicación web típica de .NET con una aplicación cliente y se le proporciona la mayor parte del código. Se eliminará cualquier código que no sea específico de Personalizer de los siguientes fragmentos de código para que pueda centrarse en el código que sí es específico de esta.

### <a name="create-personalizer-client"></a>Creación de un cliente de Personalizer

En el archivo **Startup.cs** del servidor, el punto de conexión y la clave de Personalizer se usan para crear el cliente de esta. La aplicación cliente no necesita comunicarse con Personalizer en esta aplicación, sino que se basa en el servidor para realizar esas llamadas al SDK.

Este es el código de inicio de .NET del servidor web:

```csharp
using Microsoft.Azure.CognitiveServices.Personalizer;
// ... other using statements removed for brevity

namespace HttpRequestFeaturesExample
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            string personalizerApiKey = Configuration.GetSection("PersonalizerApiKey").Value;
            string personalizerEndpoint = Configuration.GetSection("PersonalizerConfiguration:ServiceEndpoint").Value;
            if (string.IsNullOrEmpty(personalizerEndpoint) || string.IsNullOrEmpty(personalizerApiKey))
            {
                throw new ArgumentException("Missing Azure Personalizer endpoint and/or api key.");
            }
            services.AddSingleton(client =>
            {
                return new PersonalizerClient(new ApiKeyServiceClientCredentials(personalizerApiKey))
                {
                    Endpoint = personalizerEndpoint
                };
            });

            services.AddMvc();
        }

        // ... code removed for brevity
    }
}
```

### <a name="select-best-action"></a>Selección de la mejor acción

En el archivo **PersonalizerController.cs** la API **GenerateRank** del servidor resume la preparación para llamar a la API Rank.

* Cree un nuevo `eventId` para la llamada a Rank.
* Obtenga la lista de acciones.
* Obtenga la lista de características del usuario y cree características de contexto.
* Opcionalmente, establezca las acciones excluidas
* Llame a la API Rank y devuelva los resultados al cliente

```csharp
/// <summary>
/// Creates a RankRequest with user time of day, HTTP request features,
/// and taste as the context and several different foods as the actions
/// </summary>
/// <returns>RankRequest with user info</returns>
[HttpGet("GenerateRank")]
public RankRequest GenerateRank()
{
    string eventId = Guid.NewGuid().ToString();

    // Get the actions list to choose from personalizer with their features.
    IList<RankableAction> actions = GetActions();

    // Get context information from the user.
    HttpRequestFeatures httpRequestFeatures = GetHttpRequestFeaturesFromRequest(Request);
    string timeOfDayFeature = GetUsersTimeOfDay();
    string tasteFeature = GetUsersTastePreference();

    // Create current context from user specified data.
    IList<object> currentContext = new List<object>() {
            new { time = timeOfDayFeature },
            new { taste = tasteFeature },
            new { httpRequestFeatures }
    };

    // Exclude an action for personalizer ranking. This action will be held at its current position.
    IList<string> excludeActions = new List<string> { "juice" };

    // Rank the actions
    return new RankRequest(actions, currentContext, excludeActions, eventId);
}
```

El JSON que se envía a Personalizer, que contiene las acciones (con características) y las características de contexto actuales, tiene el siguiente aspecto:

```json
{
    "contextFeatures": [
        {
            "time": "morning"
        },
        {
            "taste": "savory"
        },
        {
            "httpRequestFeatures": {
                "_synthetic": false,
                "MRefer": {
                    "referer": "http://localhost:51840/"
                },
                "OUserAgent": {
                    "_ua": "",
                    "_DeviceBrand": "",
                    "_DeviceFamily": "Other",
                    "_DeviceIsSpider": false,
                    "_DeviceModel": "",
                    "_OSFamily": "Windows",
                    "_OSMajor": "10",
                    "DeviceType": "Desktop"
                }
            }
        }
    ],
    "actions": [
        {
            "id": "pasta",
            "features": [
                {
                    "taste": "savory",
                    "spiceLevel": "medium"
                },
                {
                    "nutritionLevel": 5,
                    "cuisine": "italian"
                }
            ]
        },
        {
            "id": "ice cream",
            "features": [
                {
                    "taste": "sweet",
                    "spiceLevel": "none"
                },
                {
                    "nutritionalLevel": 2
                }
            ]
        },
        {
            "id": "juice",
            "features": [
                {
                    "taste": "sweet",
                    "spiceLevel": "none"
                },
                {
                    "nutritionLevel": 5
                },
                {
                    "drink": true
                }
            ]
        },
        {
            "id": "salad",
            "features": [
                {
                    "taste": "sour",
                    "spiceLevel": "low"
                },
                {
                    "nutritionLevel": 8
                }
            ]
        },
        {
            "id": "popcorn",
            "features": [
                {
                    "taste": "salty",
                    "spiceLevel": "none"
                },
                {
                    "nutritionLevel": 3
                }
            ]
        },
        {
            "id": "coffee",
            "features": [
                {
                    "taste": "bitter",
                    "spiceLevel": "none"
                },
                {
                    "nutritionLevel": 3
                },
                {
                    "drink": true
                }
            ]
        },
        {
            "id": "soup",
            "features": [
                {
                    "taste": "sour",
                    "spiceLevel": "high"
                },
                {
                    "nutritionLevel": 7
                }
            ]
        }
    ],
    "excludedActions": [
        "juice"
    ],
    "eventId": "82ac52da-4077-4c7d-b14e-190530578e75",
    "deferActivation": null
}
```

### <a name="return-personalizer-rewardactionid-to-client"></a>Devolución de rewardActionId de Personalizer al cliente

La API Rank devuelve el valor de **rewardActionId** de la mejor acción seleccionada al servidor.

Muestra la acción devuelta en **rewardActionId**.

```json
{
    "ranking": [
        {
            "id": "popcorn",
            "probability": 0.833333254
        },
        {
            "id": "salad",
            "probability": 0.03333333
        },
        {
            "id": "juice",
            "probability": 0
        },
        {
            "id": "soup",
            "probability": 0.03333333
        },
        {
            "id": "coffee",
            "probability": 0.03333333
        },
        {
            "id": "pasta",
            "probability": 0.03333333
        },
        {
            "id": "ice cream",
            "probability": 0.03333333
        }
    ],
    "eventId": "82ac52da-4077-4c7d-b14e-190530578e75",
    "rewardActionId": "popcorn"
}
```

### <a name="client-displays-the-rewardactionid-action"></a>El cliente muestra la acción rewardActionId

Para este tutorial, se muestra el valor de `rewardActionId`.

En su propia aplicación futura, puede ser un texto exacto, un botón o una sección de la página web resaltada. La lista se devuelve para cualquier análisis posterior de las puntuaciones, no para una ordenación del contenido. Solo debería aparecer el contenido de `rewardActionId`.

## <a name="reward-api-collect-information-for-reward"></a>API Reward: Recopilación de información para recompensas

La puntuación de [recompensa](concept-rewards.md) debe planearse con cuidado, del mismo modo que se planean las características. Normalmente, la puntuación de recompensa debe ser un valor comprendido entre 0 y 1. El valor se _puede_ calcular en parte en la aplicación cliente, en función de los comportamientos del usuario y, en parte, en el servidor, en función de la lógica de negocios y los objetivos.

Si el servidor no llama a la API Reward durante el **tiempo de espera de recompensa** que se haya configurado en Azure Portal para el recurso de Personalizer, se usará la **recompensa predeterminada** (también configurada en Azure Portal) para ese evento.

En esta aplicación de ejemplo, puede seleccionar un valor para ver cómo afecta la recompensa a las selecciones.

## <a name="additional-ways-to-learn-from-this-sample"></a>Formas adicionales de aprender de este ejemplo

En el ejemplo se usan varios eventos basados en tiempo que se configuran en Azure Portal para el recurso de Personalizer. Experimente con esos valores y, a continuación, vuelva a esta aplicación web de ejemplo para ver cómo afectan los cambios a las llamadas a Rank y Reward:

* Tiempo de espera de las recompensas
* Frecuencia de actualización del modelo

Entre las opciones de configuración adicionales con las que se puede experimentar se incluyen:
* Recompensa predeterminada
* Porcentaje de exploración


## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado este tutorial, elimine los recursos siguientes:

* Elimine el directorio del proyecto de ejemplo.
* Elimine el recurso de Personalizer: considere los recursos de Personalizer como dedicados a las acciones y contextos: por ejemplo, vuelva a usar este recurso solo si utiliza alimentos como dominio de las acciones.


## <a name="next-steps"></a>Pasos siguientes
* [Funcionamiento de Personalizer](how-personalizer-works.md)
* [Características](concepts-features.md): aprenda conceptos acerca de las características con las acciones y el contexto.
* [Recompensas](concept-rewards.md): obtenga información sobre el cálculo de recompensas.
