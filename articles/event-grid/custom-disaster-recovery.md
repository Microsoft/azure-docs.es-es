---
title: Recuperación ante desastres en temas personalizados de Azure Event Grid
description: Este tutorial le guía por los pasos para configurar la arquitectura de eventos para recuperación en caso de que el servicio Event Grid deje de funcionar correctamente en una región.
ms.topic: tutorial
ms.date: 07/07/2020
ms.openlocfilehash: dc893973fe1f6367cd9b95de2014d92c045056f4
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87461687"
---
# <a name="build-your-own-disaster-recovery-for-custom-topics-in-event-grid"></a>Creación de una recuperación ante desastres propia para temas personalizados en Event Grid
La recuperación ante desastres se ocupa de recuperarse tras una pérdida grave de funcionalidad de la aplicación. Este tutorial le guía por los pasos para configurar la arquitectura de eventos para recuperación en caso de que el servicio Event Grid deje de funcionar correctamente en una región determinada.

En este tutorial, aprenderá a crear una arquitectura de conmutación por error activo-pasivo para temas personalizados en Event Grid. Para realizar la conmutación por error, creará un reflejo de sus temas y suscripciones entre dos regiones y, luego, administrará una conmutación por error cuando un tema deje de tener un estado correcto. La arquitectura de este tutorial conmuta por error todo el tráfico nuevo. Es importante tener en cuenta que, con esta configuración, los eventos ya en ejecución no se recuperarán hasta que la región comprometida vuelva a tener un estado correcto.

> [!NOTE]
> Event Grid admite ahora la recuperación ante desastres geográfica automática (GeoDR) en el servidor. Aún puede implementar la lógica de recuperación ante desastres en el lado del cliente si desea un mayor control sobre el proceso de conmutación por error. Para obtener más información acerca de GeoDR automática, consulte [Recuperación de desastres geográfica del lado servidor en Azure Event Grid](geo-disaster-recovery.md).

## <a name="create-a-message-endpoint"></a>Creación de un punto de conexión de mensaje

Para probar la configuración de conmutación por error, necesitará un punto de conexión en el que recibir los eventos. El punto de conexión no forma parte de su infraestructura de conmutación por error, pero funcionará como nuestro controlador de eventos para facilitar la prueba.

Para simplificar la prueba, implemente una [aplicación web pregenerada](https://github.com/Azure-Samples/azure-event-grid-viewer) que muestre los mensajes de eventos. La solución implementada incluye un plan de App Service, una aplicación web de App Service y el código fuente desde GitHub.

1. Seleccione **Deploy to Azure** (Implementar en Azure) para implementar la solución en su suscripción. En Azure Portal, proporcione valores para los parámetros.

   <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fazure-event-grid-viewer%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png" alt="Button to Deploy to Aquent." /></a>

1. La implementación puede tardar unos minutos en completarse. Después de que la implementación se haya realizado correctamente, puede ver la aplicación web para asegurarse de que se está ejecutando. En un explorador web, vaya a: `https://<your-site-name>.azurewebsites.net`
Asegúrese de anotar esta dirección URL, ya que la necesitará más adelante.

1. Verá el sitio, pero aún no se ha publicado en él ningún evento.

   ![Visualización del nuevo sitio](./media/blob-event-quickstart-portal/view-site.png)

[!INCLUDE [event-grid-register-provider-portal.md](../../includes/event-grid-register-provider-portal.md)]


## <a name="create-your-primary-and-secondary-topics"></a>Creación de los temas principal y secundario

En primer lugar, cree dos temas de Event Grid. Estos temas funcionarán como el principal y el secundario. De forma predeterminada, los eventos fluirán por el tema principal. Si hay una interrupción del servicio en la región primaria, la base de datos secundaria tomará el relevo.

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 

1. En la esquina superior izquierda del menú principal de Azure, elija **Todos los servicios** > busque **Event Grid** > seleccione **Temas de Event Grid**.

   ![Menú Temas de Event Grid](./media/custom-disaster-recovery/select-topics-menu.png)

    Seleccione la estrella junto a Temas de Event Grid para agregarlo al menú de recursos y acceder a él más fácilmente en el futuro.

1. En el menú Temas de Event Grid, seleccione **+AGREGAR** para crear el tema principal.

   * Asigne al tema un nombre lógico y agregue "-principal" como sufijo para que sea fácil realizar un seguimiento.
   * La región de este tema será la región primaria.

     ![Cuadro de diálogo de creación del tema principal de Event Grid](./media/custom-disaster-recovery/create-primary-topic.png)

1. Una vez creado el tema, vaya a él y copie el **punto de conexión del tema**. Necesitará el URI más adelante.

    ![Tema principal de Event Grid](./media/custom-disaster-recovery/get-primary-topic-endpoint.png)

1. Obtenga la clave de acceso para el tema, que también necesitará más adelante. Haga clic en **Claves de acceso** en el menú de recursos y copie Clave 1.

    ![Obtención de la clave del tema principal](./media/custom-disaster-recovery/get-primary-access-key.png)

1. En la hoja Tema, haga clic en **+Suscripción de eventos** para crear una suscripción que conecte el sitio web receptor de eventos de suscripción que creó en los requisitos previos con el tutorial.

   * Asigne un nombre lógico a la suscripción de eventos y agregue "-principal" como sufijo para que sea fácil realizar un seguimiento.
   * Seleccione el webhook de tipo de punto de conexión.
   * Establezca el punto de conexión en la dirección URL del evento del receptor de eventos, que debe parecerse a: `https://<your-event-reciever>.azurewebsites.net/api/updates`

     ![Suscripción al evento principal de Event Grid](./media/custom-disaster-recovery/create-primary-es.png)

1. Repita el mismo flujo para crear su tema y suscripción secundarios. Esta vez, reemplace el sufijo "-principal" por el sufijo "-secundario" para facilitar el seguimiento. Por último, asegúrese de colocarlo en una región distinta de Azure. Aunque puede colocarlo donde quiera, se recomienda usar las [regiones emparejadas de Azure](../best-practices-availability-paired-regions.md). Si coloca el tema y la suscripción secundarios en una región distinta, se tiene la seguridad de que los nuevos eventos fluirán incluso si la región principal deja de funcionar.

Ahora debería tener:

   * Un sitio web receptor de eventos para pruebas.
   * Un tema principal en su región primaria.
   * Una suscripción al evento principal que conecta el tema principal con el sitio web receptor de eventos.
   * Un tema secundario en la región secundaria.
   * Una suscripción al evento secundario que conecta el tema principal con el sitio web receptor de eventos.

## <a name="implement-client-side-failover"></a>Implementación de conmutación por error en el lado de cliente

Ahora que tiene configurados un par de temas y suscripciones con redundancia regional, está listo para implementar la conmutación por error en el cliente. Hay varias maneras de lograrlo, pero todas las implementaciones de conmutación por error tendrán una característica común: si un tema ya no es correcto, el tráfico se redirigirá al otro tema.

### <a name="basic-client-side-implementation"></a>Implementación básica en el cliente

El siguiente código de ejemplo es un publicador .NET sencillo que siempre intentará publicar primero en el tema principal. Si no tiene éxito, conmutará por error al tema secundario. En cualquier caso, también comprueba la API de mantenimiento del otro tema mediante una operación GET en `https://<topic-name>.<topic-region>.eventgrid.azure.net/api/health`. Un tema correcto siempre debe responder con **200 Correcto** cuando se realiza una operación GET en el punto de conexión **/api/health**.

```csharp
using System;
using System.Net.Http;
using System.Collections.Generic;
using Microsoft.Azure.EventGrid;
using Microsoft.Azure.EventGrid.Models;
using Newtonsoft.Json;

namespace EventGridFailoverPublisher
{
    // This captures the "Data" portion of an EventGridEvent on a custom topic
    class FailoverEventData
    {
        [JsonProperty(PropertyName = "teststatus")]
        public string TestStatus { get; set; }
    }

    class Program
    {
        static void Main(string[] args)
        {
            // TODO: Enter the endpoint each topic. You can find this topic endpoint value
            // in the "Overview" section in the "Event Grid Topics" blade in Azure Portal..
            string primaryTopic = "https://<primary-topic-name>.<primary-topic-region>.eventgrid.azure.net/api/events";
            string secondaryTopic = "https://<secondary-topic-name>.<secondary-topic-region>.eventgrid.azure.net/api/events";

            // TODO: Enter topic key for each topic. You can find this in the "Access Keys" section in the
            // "Event Grid Topics" blade in Azure Portal.
            string primaryTopicKey = "<your-primary-topic-key>";
            string secondaryTopicKey = "<your-secondary-topic-key>";

            string primaryTopicHostname = new Uri( primaryTopic).Host;
            string secondaryTopicHostname = new Uri(secondaryTopic).Host;

            Uri primaryTopicHealthProbe = new Uri("https://" + primaryTopicHostname + "/api/health");
            Uri secondaryTopicHealthProbe = new Uri("https://" + secondaryTopicHostname + "/api/health");

            var httpClient = new HttpClient();

            try
            {
                TopicCredentials topicCredentials = new TopicCredentials(primaryTopicKey);
                EventGridClient client = new EventGridClient(topicCredentials);

                client.PublishEventsAsync(primaryTopicHostname, GetEventsList()).GetAwaiter().GetResult();
                Console.Write("Published events to primary Event Grid topic.");

                HttpResponseMessage health = httpClient.GetAsync(secondaryTopicHealthProbe).Result;
                Console.Write("\n\nSecondary Topic health " + health);
            }
            catch (Microsoft.Rest.Azure.CloudException e)
            {
                TopicCredentials topicCredentials = new TopicCredentials(secondaryTopicKey);
                EventGridClient client = new EventGridClient(topicCredentials);

                client.PublishEventsAsync(secondaryTopicHostname, GetEventsList()).GetAwaiter().GetResult();
                Console.Write("Published events to secondary Event Grid topic. Reason for primary topic failure:\n\n" + e);

                HttpResponseMessage health = httpClient.GetAsync(primaryTopicHealthProbe).Result;
                Console.Write("\n\nPrimary Topic health " + health);
            }

            Console.ReadLine();
        }

        static IList<EventGridEvent> GetEventsList()
        {
            List<EventGridEvent> eventsList = new List<EventGridEvent>();

            for (int i = 0; i < 5; i++)
            {
                eventsList.Add(new EventGridEvent()
                {
                    Id = Guid.NewGuid().ToString(),
                    EventType = "Contoso.Failover.Test",
                    Data = new FailoverEventData()
                    {
                        TestStatus = "success"
                    },
                    EventTime = DateTime.Now,
                    Subject = "test" + i,
                    DataVersion = "2.0"
                });
            }

            return eventsList;
        }
    }
}
```

### <a name="try-it-out"></a>Prueba

Ahora que tiene todos los componentes en su sitio, puede probar la implementación de la conmutación por error. Ejecute el ejemplo anterior en Visual Studio Code o su entorno favorito. Reemplace los siguientes cuatro valores por los puntos de conexión y claves de sus temas:

   * primaryTopic: el punto de conexión del tema principal.
   * secondaryTopic: el punto de conexión del tema secundario.
   * primaryTopicKey: la clave del tema principal.
   * secondaryTopicKey: la clave del tema secundario.

Intente ejecutar el publicador de eventos. Verá que los eventos de prueba se muestran en el visor de Event Grid, como se ilustra a continuación.

![Suscripción al evento principal de Event Grid](./media/custom-disaster-recovery/event-grid-viewer.png)

Para asegurarse de que la conmutación por error funciona, puede cambiar algunos caracteres de la clave del tema principal para que ya no sea válido. Intente ejecutar de nuevo el publicador. Aunque seguirá viendo que aparecen nuevos eventos en el visor de Event Grid, si examina la consola verá que ya no se publican a través del tema secundario.

### <a name="possible-extensions"></a>Posibles extensiones

Hay muchas maneras de ampliar este ejemplo según sus necesidades. En escenarios de gran volumen, puede que quiera comprobar periódicamente la API de mantenimiento del tema de forma independiente. Así, si un tema deja de funcionar, no necesitará comprobarlo con cada publicación. Una vez que sepa que un tema ya no tiene un estado correcto, puede adoptar como valor predeterminado la publicación en el tema secundario.

Igualmente, puede que quiera implementar la lógica de conmutación por recuperación según sus necesidades específicas. Si publicar en el centro de datos más cercano es fundamental para reducir la latencia, puede sondear periódicamente la API de mantenimiento de un tema que se haya conmutado por error. Cuando su estado vuelva a ser correcto, sabrá que es seguro conmutar por recuperación al centro de datos más cercano.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre cómo [recibir eventos en un punto de conexión HTTP](./receive-events.md).
- Descubra cómo [enrutar eventos a Conexiones híbridas](./custom-event-to-hybrid-connection.md).
- Aprenda sobre la [recuperación ante desastres mediante Azure DNS y Traffic Manager](../networking/disaster-recovery-dns-traffic-manager.md).
