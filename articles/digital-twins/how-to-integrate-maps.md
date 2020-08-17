---
title: Uso de Azure Digital Twins para actualizar un plano interior de Azure Maps
titleSuffix: Azure Digital Twins
description: Vea cómo crear una función de Azure que pueda usar las notificaciones de Azure Digital Twins y el grafo de gemelos para actualizar la información que se muestra en Azure Maps.
author: alexkarcher-msft
ms.author: alkarche
ms.date: 6/3/2020
ms.topic: how-to
ms.service: digital-twins
ms.reviewer: baanders
ms.openlocfilehash: 6aad6201136bb925d5e094de115cc7274cc7872a
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87131419"
---
# <a name="use-azure-digital-twins-to-update-an-azure-maps-indoor-map"></a>Uso de Azure Digital Twins para actualizar un plano interior de Azure Maps

En este artículo se explican los pasos necesarios para usar los datos de Azure Digital Twins a fin de actualizar la información que se muestra en un *plano interior* mediante [Azure Maps](../azure-maps/about-azure-maps.md). Azure Digital Twins almacena un grafo de las relaciones de los dispositivos IoT y enruta la telemetría a distintos puntos de conexión, lo que lo convierte en el servicio perfecto para actualizar las superposiciones de información en los planos.

En estas instrucciones se tratará lo siguiente:

1. Configurar la instancia de Azure Digital Twins para enviar eventos de actualización de gemelos a una función en [Azure Functions](../azure-functions/functions-overview.md).
2. Crear una función de Azure para actualizar un conjunto de estados de características de planos interiores de Azure Maps.
3. Procedimientos para almacenar id. de planos y de conjuntos de estados de características en el grafo de Azure Digital Twins.

### <a name="prerequisites"></a>Requisitos previos

* Siga el [*Tutorial de Azure Digital Twins: Conexión de una solución de un extremo a otro*](./tutorial-end-to-end.md).
    * En él ampliará este gemelo con un punto de conexión y una ruta adicionales. En dicho tutorial también agregará otra función a la aplicación de funciones. 
* Siga el [*Tutorial de Azure Maps: Uso de Creator para crear planos interiores*](../azure-maps/tutorial-creator-indoor-maps.md) a fin de crear un plano interior de Azure Maps con un *conjunto de estados de características*.
    * Los [conjuntos de estados de características](../azure-maps/creator-indoor-maps.md#feature-statesets) son colecciones de propiedades dinámicas (estados) asignadas a las características del conjunto de datos, como salas o equipamiento. En el tutorial anterior de Azure Maps, el conjunto de estados de las características almacena el estado de la sala que se mostrará en un plano.
    * Necesitará el *id. de conjunto de estados* de las características y el *id. de suscripción* de Azure Maps.

### <a name="topology"></a>Topología

En la imagen siguiente se muestra dónde encajan los elementos de integración de planos interiores en este tutorial en un escenario de mayor tamaño de Azure Digital Twins de un extremo a otro.

:::image type="content" source="media/how-to-integrate-maps/maps-integration-topology.png" alt-text="Vista de los servicios de Azure en un escenario de un extremo a otro, resaltando la parte Integración de planos interiores" lightbox="media/how-to-integrate-maps/maps-integration-topology.png":::

## <a name="create-a-function-to-update-a-map-when-twins-update"></a>Creación de una función para actualizar un plano al actualizar los gemelos

En primer lugar, creará una ruta en Azure Digital Twins para reenviar todos los eventos de actualización de gemelos a un tema de Event Grid. Después, usará una función de Azure para leer los mensajes de la actualización y actualizar un conjunto de estados de características en Azure Maps. 

## <a name="create-a-route-and-filter-to-twin-update-notifications"></a>Creación de una ruta y un filtro para las notificaciones de actualización de gemelos

Las instancias de Azure Digital Twins pueden emitir eventos de actualización de gemelos cada vez que se actualiza el estado de uno de estos elementos. El [*Tutorial de Azure Digital Twins: Conexión de una solución de un extremo a otro*](./tutorial-end-to-end.md) vinculado anteriormente le guiará a través de un escenario en el que se usa un termómetro para actualizar un atributo de temperatura conectado al gemelo de una sala. Ampliará esa solución mediante la suscripción a las notificaciones de actualización de gemelos y el uso de dicha información para actualizar los planos.

Este patrón realiza la lectura directamente desde el gemelo de la sala, en lugar de desde el dispositivo IoT, lo que ofrece la flexibilidad de cambiar el origen de datos subyacente para la temperatura sin necesidad de actualizar la lógica de asignación. Por ejemplo, puede agregar varios termómetros o establecer que esta sala comparta un termómetro con otra, todo ello sin necesidad de actualizar la lógica de asignación.

1. Cree un tema de Event Grid, que recibirá eventos de la instancia de Azure Digital Twins.
    ```azurecli
    az eventgrid topic create -g <your-resource-group-name> --name <your-topic-name> -l <region>
    ```

2. Cree un punto de conexión para vincular el tema de Event Grid a Azure Digital Twins.
    ```azurecli
    az dt endpoint create eventgrid --endpoint-name <Event-Grid-endpoint-name> --eventgrid-resource-group <Event-Grid-resource-group-name> --eventgrid-topic <your-Event-Grid-topic-name> -n <your-Azure-Digital-Twins-instance-name>
    ```

3. Cree una ruta en Azure Digital Twins para enviar eventos de actualización de gemelos al punto de conexión.
    ```azurecli
    az dt route create -n <your-Azure-Digital-Twins-instance-name> --endpoint-name <Event-Grid-endpoint-name> --route-name <my_route> --filter "type = 'Microsoft.DigitalTwins.Twin.Update'"
    ```

## <a name="create-an-azure-function-to-update-maps"></a>Creación de una función de Azure para actualizar planos

Va a crear una función desencadenada por Event Grid en la aplicación de funciones siguiendo el tutorial de un extremo a otro ([*Tutorial: Conexión de una solución de un extremo a otro*](./tutorial-end-to-end.md)). Esta función desempaquetará esas notificaciones y enviará actualizaciones a un conjunto de estados de características de Azure Maps para actualizar la temperatura de una sala. 

Vea el documento siguiente para obtener información de referencia: [*Desencadenador de Azure Event Grid para Azure Functions*](https://docs.microsoft.com/azure/azure-functions/functions-bindings-event-grid-trigger).

Reemplace el código de la función por el siguiente. Solo filtrará las actualizaciones de los gemelos de los espacios, leerá la temperatura actualizada y enviará esa información a Azure Maps.

```C#
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using System;
using System.Threading.Tasks;
using System.Net.Http;

namespace SampleFunctionsApp
{
    public static class ProcessDTUpdatetoMaps
    {   //Read maps credentials from application settings on function startup
        private static string statesetID = Environment.GetEnvironmentVariable("statesetID");
        private static string subscriptionKey = Environment.GetEnvironmentVariable("subscription-key");
        private static HttpClient httpClient = new HttpClient();

        [FunctionName("ProcessDTUpdatetoMaps")]
        public static async Task Run([EventGridTrigger]EventGridEvent eventGridEvent, ILogger log)
        {
            JObject message = (JObject)JsonConvert.DeserializeObject(eventGridEvent.Data.ToString());
            log.LogInformation("Reading event from twinID:" + eventGridEvent.Subject.ToString() + ": " +
                eventGridEvent.EventType.ToString() + ": " + message["data"]);

            //Parse updates to "space" twins
            if (message["data"]["modelId"].ToString() == "dtmi:contosocom:DigitalTwins:Space;1")
            {   //Set the ID of the room to be updated in your map. 
                //Replace this line with your logic for retrieving featureID. 
                string featureID = "UNIT103";

                //Iterate through the properties that have changed
                foreach (var operation in message["data"]["patch"])
                {
                    if (operation["op"].ToString() == "replace" && operation["path"].ToString() == "/Temperature")
                    {   //Update the maps feature stateset
                        var postcontent = new JObject(new JProperty("States", new JArray(
                            new JObject(new JProperty("keyName", "temperature"),
                                 new JProperty("value", operation["value"].ToString()),
                                 new JProperty("eventTimestamp", DateTime.Now.ToString("s"))))));

                        var response = await httpClient.PostAsync(
                            $"https://atlas.microsoft.com/featureState/state?api-version=1.0&statesetID={statesetID}&featureID={featureID}&subscription-key={subscriptionKey}",
                            new StringContent(postcontent.ToString()));

                        log.LogInformation(await response.Content.ReadAsStringAsync());
                    }
                }
            }
        }
    }
}
```

Tendrá que establecer dos variables de entorno en la aplicación de funciones. Una es la [clave de suscripción principal de Azure Maps](../azure-maps/quick-demo-map-app.md#get-the-primary-key-for-your-account), y la otra, el [id. del conjunto de estados de Azure Maps](../azure-maps/tutorial-creator-indoor-maps.md#create-a-feature-stateset).

```azurecli-interactive
az functionapp config appsettings set --settings "subscription-key=<your-Azure-Maps-primary-subscription-key> -g <your-resource-group> -n <your-App-Service-(function-app)-name>"
az functionapp config appsettings set --settings "statesetID=<your-Azure-Maps-stateset-ID> -g <your-resource-group> -n <your-App-Service-(function-app)-name>
```

### <a name="view-live-updates-on-your-map"></a>Consulta de actualizaciones directas en el plano

Para ver las actualizaciones directas de la temperatura, siga estos pasos:

1. Comience a enviar datos de IoT simulados mediante la ejecución del proyecto **DeviceSimulator** desde el [*Tutorial de Azure Digital Twins: Conexión de una solución de un extremo a otro*](tutorial-end-to-end.md). Las instrucciones se encuentran en la sección [*Configuración y ejecución de la simulación*](././tutorial-end-to-end.md#configure-and-run-the-simulation).
2. Use [el módulo **Azure Maps Indoor**](../azure-maps/how-to-use-indoor-module.md) para representar planos interiores creados en el Creador de Azure Maps.
    1. Copie el HTML de la sección [*Ejemplo: Uso del módulo de planos interiores*](../azure-maps/how-to-use-indoor-module.md#example-use-the-indoor-maps-module) del [*Tutorial: Uso del módulo de mapas de Azure Maps Indoor*](../azure-maps/how-to-use-indoor-module.md) en un archivo local.
    1. Reemplace los elementos *tilesetId* y *statesetId* en el archivo HTML local por sus valores.
    1. Abra ese archivo en el explorador.

Ambos ejemplos envían la temperatura en un rango compatible, por lo que debería ver el color de la actualización de la sala 121 en el plano aproximadamente cada 30 segundos.

:::image type="content" source="media/how-to-integrate-maps/maps-temperature-update.png" alt-text="Plano de una oficina que muestra la sala 121 de color naranja":::

## <a name="store-your-maps-information-in-azure-digital-twins"></a>Almacenamiento de la información de los planos en Azure Digital Twins

Ahora que tiene una solución codificada de forma rígida para actualizar la información de los planos, puede usar el grafo de Azure Digital Twins para almacenar toda la información necesaria a fin de actualizar el plano interior. Esto incluiría el id. de conjunto de estados, el id. de suscripción de los planos y el id. de característica de cada plano y ubicación, respectivamente. 

Una solución para este ejemplo específico implicaría actualizar cada espacio de nivel superior para que tuviera un id. de conjunto de estados y un atributo de id. de suscripción de planos, así como actualizar cada sala para que tuviera un id. de característica. Tendría que establecer estos valores una vez al inicializar el grafo de gemelos y, después, consultar esos valores para cada evento de actualización de gemelos.

En función de la configuración de la topología, podrá almacenar estos tres atributos en diferentes niveles en correlación con la granularidad del plano.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo administrar, actualizar y recuperar información del grafo de gemelos, vea las referencias siguientes:

* [*Procedimiento: Administración de Digital Twins*](./how-to-manage-twin.md)
* [*Procedimiento: Consulta del grafo gemelo*](./how-to-query-graph.md)