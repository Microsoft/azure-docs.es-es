---
title: Supervisión de las API con Azure API Management, Event Hubs y Moesif
titleSuffix: Azure API Management
description: Aplicación de ejemplo que muestra la directiva log-to-eventhub que conecta Azure API Management, Azure Event Hubs y Moesif para el registro y la supervisión HTTP
services: api-management
documentationcenter: ''
author: darrelmiller
manager: erikre
editor: ''
ms.assetid: c528cf6f-5f16-4a06-beea-fa1207541a47
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.custom: devx-track-csharp
ms.topic: article
ms.date: 01/23/2018
ms.author: apimpm
ms.openlocfilehash: abb9cbb73f8957cec2cb3240bbf186623b9b2ef9
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88205512"
---
# <a name="monitor-your-apis-with-azure-api-management-event-hubs-and-moesif"></a>Supervisión de las API con Azure API Management, Event Hubs y Moesif
El [servicio Administración de API](api-management-key-concepts.md) proporciona muchas capacidades para mejorar el procesamiento de solicitudes de HTTP enviadas a la API HTTP. Sin embargo, la existencia de las solicitudes y respuestas es transitoria. Se realiza la solicitud y fluye a través del servicio Administración de API a la API de back-end. La API procesa la solicitud y se pasa una respuesta al consumidor de API. El servicio API Management mantiene algunas estadísticas importantes acerca de las API que se muestran en el panel de Azure Portal, pero aparte de eso, los detalles desaparecen.

Mediante la directiva log-to-eventhub del servicio API Management, puede enviar los detalles de la solicitud y la respuesta a un [centro de eventos de Azure](../event-hubs/event-hubs-about.md). Existen diversos motivos por los que puede desear generar eventos de mensajes HTTP que se envían a las API. Algunos ejemplos incluyen traza de auditoría de las actualizaciones, análisis de uso, alertas de excepción e integraciones de terceros.

Este artículo muestra cómo capturar el mensaje de solicitud y respuesta de HTTP completo, enviarlo a un centro de eventos y, a continuación, retransmitir ese mensaje a un servicio que proporciona servicios de registro y supervisión de HTTP.

## <a name="why-send-from-api-management-service"></a>¿Por qué se envían desde el servicio Administración de API?
Es posible escribir middleware HTTP que se puede conectar a los marcos API HTTP para capturar solicitudes y respuestas de HTTP e introducirlas en sistemas de registro y supervisión. El inconveniente de este enfoque es que el middleware HTTP debe integrarse en la API de back-end y debe coincidir con la plataforma de la API. Si hay varias API, todas deben implementar el middleware. A menudo existen motivos por los que no se pueden actualizar las API de back-end.

El usp del servicio Azure API Management para la integración con la infraestructura de registro proporciona una solución centralizada e independiente de la plataforma. También es escalable, en parte debido a las capacidades de [replicación geográfica](api-management-howto-deploy-multi-region.md) de Azure API Management.

## <a name="why-send-to-an-azure-event-hub"></a>¿Por qué se envía a un Centro de eventos de Azure?
Es razonable preguntar por qué crear una directiva que es específica de Azure Event Hubs Hay muchos lugares diferentes donde puede ser conveniente registrar mis solicitudes. ¿Por qué no simplemente enviar las solicitudes directamente al destino final?  Es una opción. Sin embargo, al realizar solicitudes de registro desde un servicio API Management, es necesario tener en cuenta cómo afectarán los mensajes de registro al rendimiento de la API. Para manejar los aumentos graduales de la carga puede aumentar las instancias disponibles de los componentes del sistema o aprovechar las ventajas de la replicación geográfica. Sin embargo, picos de tráfico cortos pueden hacer que las solicitudes se retrasen si las solicitudes a la infraestructura de registro empiezan a ralentizarse debido a la carga.

Azure Event Hubs está diseñado para la entrada de grandes volúmenes de datos, con capacidad para tratar con un número mucho mayor de eventos que el número de solicitudes de HTTP que procesan la mayoría de las API. El centro de eventos actúa como un tipo de búfer sofisticado entre el servicio API Management y la infraestructura que almacena y procesa los mensajes. Esto garantiza que no se verá afectado el rendimiento de la API debido a la infraestructura de registro.

Una vez que los datos se han pasado a un centro de eventos, se conservan y esperan a que los consumidores del centro de eventos los procese. Al centro de eventos no le importa cómo se procesa, simplemente se ocupa de asegurarse de que el mensaje se entregue correctamente.

Event Hubs ofrece la posibilidad de transmitir eventos a varios grupos de consumidores. Esto permite que sistemas diferentes procesen los eventos. Esto permite muchos escenarios de integración sin agregar retrasos en el procesamiento de la solicitud de API dentro del servicio Administración de API, ya que solo es necesario generar un evento.

## <a name="a-policy-to-send-applicationhttp-messages"></a>Una directiva para enviar mensajes de aplicación/http
Un Centro de eventos acepta datos de eventos como una cadena simple. El contenido de esa cadena es decisión suya. Para poder empaquetar una solicitud HTTP y enviarla a Event Hubs, es necesario formatear la cadena con la información de solicitud o respuesta. En situaciones como esta, si hay un formato existente que podamos reusar, es posible que no necesitemos escribir nuestro propio código de análisis. Inicialmente consideré el uso de [HAR](http://www.softwareishard.com/blog/har-12-spec/) para enviar solicitudes y respuestas de HTTP. Sin embargo, este formato está optimizado para almacenar una secuencia de solicitudes HTTP en un formato basado en JSON. Contenía un número de elementos obligatorios que agregaba una complejidad innecesaria para el escenario de pasar el mensaje HTTP a través del cable.

Una opción alternativa era usar el tipo de soporte físico `application/http` , como se describe en la especificación HTTP [RFC 7230](https://tools.ietf.org/html/rfc7230). Este tipo de soporte físico usa el mismo formato que se usa para enviar realmente mensajes de HTTP a través del cable, pero el mensaje completo se puede colocar en el cuerpo de otra solicitud de HTTP. En nuestro caso, el cuerpo se usará como mensaje para enviarlo a Event Hubs. Convenientemente, hay un analizador en las bibliotecas de [Microsoft ASP.NET Web API 2.2 Cliente](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Client/) que puede analizar este formato y convertirlo a `HttpRequestMessage` nativo y objetos `HttpResponseMessage`.

Para poder crear este mensaje, se deben aprovechar las [expresiones de directiva](./api-management-policy-expressions.md) basadas en C# en Azure API Management. A continuación se proporciona la directiva que envía un mensaje de solicitud HTTP a Azure Event Hubs.

```xml
<log-to-eventhub logger-id="conferencelogger" partition-id="0">
@{
   var requestLine = string.Format("{0} {1} HTTP/1.1\r\n",
                                               context.Request.Method,
                                               context.Request.Url.Path + context.Request.Url.QueryString);

   var body = context.Request.Body?.As<string>(true);
   if (body != null && body.Length > 1024)
   {
       body = body.Substring(0, 1024);
   }

   var headers = context.Request.Headers
                          .Where(h => h.Key != "Authorization" && h.Key != "Ocp-Apim-Subscription-Key")
                          .Select(h => string.Format("{0}: {1}", h.Key, String.Join(", ", h.Value)))
                          .ToArray<string>();

   var headerString = (headers.Any()) ? string.Join("\r\n", headers) + "\r\n" : string.Empty;

   return "request:"   + context.Variables["message-id"] + "\n"
                       + requestLine + headerString + "\r\n" + body;
}
</log-to-eventhub>
```

### <a name="policy-declaration"></a>Declaración de directiva
Hay algunas cosas específicas que merece la pena mencionar acerca de esta expresión de directiva. La directiva log-to-eventhub tiene un atributo denominado logger-id que hace referencia al nombre del registrador que se ha creado en el servicio API Management. Los detalles de cómo configurar un registrador del centro de eventos en el servicio API Management pueden encontrarse en el documento [Cómo registrar eventos en Azure Event Hubs en Azure API Management](api-management-howto-log-event-hubs.md). El segundo atributo es un parámetro opcional que indica a Event Hubs en qué partición almacenar el mensaje. Event Hubs usa particiones para habilitar la escalabilidad y requiere dos como mínimo. La entrega ordenada de mensajes solo se garantiza dentro de una partición. Si no se indica al centro de eventos en qué partición desea colocar el mensaje, usa un algoritmo round-robin para distribuir la carga. Sin embargo, eso puede hacer que algunos de nuestros mensajes no se procesen en el orden correcto.

### <a name="partitions"></a>Particiones
Para garantizar que nuestros mensajes se entregan a los consumidores en orden y aprovechar la capacidad de distribución de carga de las particiones, elegí enviar mensajes de solicitud de HTTP a una partición y los mensajes de respuesta de HTTP a una segunda partición. Esto garantiza una distribución equilibrada de la carga y se puede tener la seguridad de que todas las solicitudes y todas las respuestas se consumirán en orden. Es posible que una respuesta se consuma antes de la solicitud correspondiente, pero como esto no es un problema porque tenemos un mecanismo diferente para correlacionar las solicitudes a las respuestas y sabemos que las solicitudes van siempre antes las respuestas.

### <a name="http-payloads"></a>Cargas de HTTP
Después de crear el valor `requestLine`, se comprueba si se debe truncar el cuerpo de la solicitud. El cuerpo de la solicitud se trunca a solo 1024. Este valor podría aumentarse; sin embargo, los mensajes individuales del centro de eventos están limitados a 256 KB, por lo que es probable que algunos cuerpos de los mensajes HTTP no quepan en un único mensaje. Al realizar el registro y análisis, una cantidad significativa de información puede derivarse simplemente de la línea de solicitud de HTTP y los encabezados. Además, muchas solicitudes de API devuelven solo cuerpos pequeños, por lo que la pérdida de valor de la información debido al truncamiento de cuerpos grandes es bastante mínima en comparación con la reducción de los costos de almacenamiento, transferencia y transformación para mantener todo el contenido del cuerpo. Una nota final acerca del procesamiento del cuerpo es que tenemos que pasar `true` al método `As<string>()` porque podemos leer el contenido del cuerpo, pero queremos que la API de back-end pueda leerlo también. Al pasar true a este método, hacemos que el cuerpo se almacene en la búfer para que se pueda leer una segunda vez. Es importante tenerlo en cuenta si tiene una API que carga archivos grandes o usa el sondeo largo. En estos casos, es mejor evitar totalmente la lectura del cuerpo.

### <a name="http-headers"></a>Encabezados HTTP
Los encabezados HTTP pueden transferirse al formato del mensaje con un formato simple de par clave/valor. Hemos decidido eliminar ciertos campos de seguridad confidenciales para evitar la pérdida innecesaria de información de credenciales. No es probable que se usen claves de API y otras credenciales para los análisis. Si deseamos realizar el análisis en el usuario y el producto específico que usa, podríamos obtenerlo del objeto `context` y agregarlo al mensaje.

### <a name="message-metadata"></a>Metadatos del mensaje
Cuando se crea el mensaje completo para enviarlo al Centro de eventos, la primera línea no es realmente parte del mensaje `application/http` . La primera línea son metadatos adicionales que incluyen si el mensaje es un mensaje de solicitud o de respuesta y un identificador de mensaje que se usa para correlacionar las solicitudes y las respuestas. El identificador del mensaje se crea mediante otra directiva que tiene este aspecto:

```xml
<set-variable name="message-id" value="@(Guid.NewGuid())" />
```

Se podría haber creado el mensaje de solicitud, almacenarlo en una variable hasta que se devuelva la respuesta y, después, simplemente enviar la solicitud y la respuesta como un solo mensaje. Sin embargo, al enviar la solicitud y la respuesta de forma independiente y usar un identificador de mensaje para correlacionar las dos, obtenemos un poco más flexibilidad en el tamaño de mensaje, la capacidad de aprovechar varias particiones al tiempo que se mantiene el orden de los mensajes y la solicitud aparecerá antes en nuestro panel de registro. También puede haber algunos escenarios donde nunca se envíe una respuesta válida al centro de eventos, posiblemente debido a un error de solicitud irrecuperable en el servicio API Management, pero aún tendremos un registro de la solicitud.

La directiva para enviar el mensaje de respuesta HTTP es parecida a la solicitud; por tanto, la configuración de directiva completa tiene el siguiente aspecto:

```xml
<policies>
  <inbound>
      <set-variable name="message-id" value="@(Guid.NewGuid())" />
      <log-to-eventhub logger-id="conferencelogger" partition-id="0">
      @{
          var requestLine = string.Format("{0} {1} HTTP/1.1\r\n",
                                                      context.Request.Method,
                                                      context.Request.Url.Path + context.Request.Url.QueryString);

          var body = context.Request.Body?.As<string>(true);
          if (body != null && body.Length > 1024)
          {
              body = body.Substring(0, 1024);
          }

          var headers = context.Request.Headers
                               .Where(h => h.Key != "Authorization" && h.Key != "Ocp-Apim-Subscription-Key")
                               .Select(h => string.Format("{0}: {1}", h.Key, String.Join(", ", h.Value)))
                               .ToArray<string>();

          var headerString = (headers.Any()) ? string.Join("\r\n", headers) + "\r\n" : string.Empty;

          return "request:"   + context.Variables["message-id"] + "\n"
                              + requestLine + headerString + "\r\n" + body;
      }
  </log-to-eventhub>
  </inbound>
  <backend>
      <forward-request follow-redirects="true" />
  </backend>
  <outbound>
      <log-to-eventhub logger-id="conferencelogger" partition-id="1">
      @{
          var statusLine = string.Format("HTTP/1.1 {0} {1}\r\n",
                                              context.Response.StatusCode,
                                              context.Response.StatusReason);

          var body = context.Response.Body?.As<string>(true);
          if (body != null && body.Length > 1024)
          {
              body = body.Substring(0, 1024);
          }

          var headers = context.Response.Headers
                                          .Select(h => string.Format("{0}: {1}", h.Key, String.Join(", ", h.Value)))
                                          .ToArray<string>();

          var headerString = (headers.Any()) ? string.Join("\r\n", headers) + "\r\n" : string.Empty;

          return "response:"  + context.Variables["message-id"] + "\n"
                              + statusLine + headerString + "\r\n" + body;
     }
  </log-to-eventhub>
  </outbound>
</policies>
```

La directiva `set-variable` crea un valor que sea accesible por la directiva `log-to-eventhub` en la sección `<inbound>` y la sección `<outbound>`.

## <a name="receiving-events-from-event-hubs"></a>Recepción de eventos de Event Hubs
Se reciben eventos del Centro de eventos de Azure mediante el [protocolo AMQP](https://www.amqp.org/). El equipo de Service Bus de Microsoft hizo que las bibliotecas cliente disponibles para facilitar los eventos de consumo. Existen dos enfoques diferentes admitidos, uno se está un *Consumidor directo* y la otra es emplear la clase `EventProcessorHost`. Ejemplos de estos dos enfoques pueden encontrarse en la [Guía de programación de Event Hubs](../event-hubs/event-hubs-programming-guide.md). La versión abreviada de las diferencias es que `Direct Consumer` proporciona un control total y `EventProcessorHost` realiza parte del trabajo de fontanería por usted, pero hace determinadas suposiciones sobre cómo procesará los eventos.

### <a name="eventprocessorhost"></a>EventProcessorHost
En este ejemplo, se usa `EventProcessorHost` por motivos de simplicidad; sin embargo es posible que no sea la mejor opción para este escenario concreto. `EventProcessorHost` realiza el trabajo duro de asegurarse de que no tiene que preocuparse acerca de problemas de subprocesamiento dentro de una clase de procesador de eventos concreto. Sin embargo, en nuestro escenario, simplemente estamos convirtiendo el mensaje a otro formato y lo pasamos a otro servicio con un método asincrónico. No es necesario actualizar el estado compartido y, por tanto, no hay riesgo de problemas de subprocesamiento. Para la mayoría de los escenarios, `EventProcessorHost` es probablemente la mejor opción y es ciertamente la opción más fácil.

### <a name="ieventprocessor"></a>IEventProcessor
El concepto central al usar `EventProcessorHost` es crear una implementación de la interfaz `IEventProcessor` que contenga el método `ProcessEventAsync`. La esencia de ese método se muestra aquí:

```csharp
async Task IEventProcessor.ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
{

    foreach (EventData eventData in messages)
    {
        _Logger.LogInfo(string.Format("Event received from partition: {0} - {1}", context.Lease.PartitionId,eventData.PartitionKey));

        try
        {
            var httpMessage = HttpMessage.Parse(eventData.GetBodyStream());
            await _MessageContentProcessor.ProcessHttpMessage(httpMessage);
        }
        catch (Exception ex)
        {
            _Logger.LogError(ex.Message);
        }
    }
    ... checkpointing code snipped ...
}
```

Se pasa una lista de objetos EventData al método y realizamos la iteración en esa lista. Se analizan los bytes de cada método en un objeto HttpMessage y ese objeto se pasa a una instancia de IHttpMessageProcessor.

### <a name="httpmessage"></a>HttpMessage
La instancia `HttpMessage` contiene tres elementos de datos:

```csharp
public class HttpMessage
{
    public Guid MessageId { get; set; }
    public bool IsRequest { get; set; }
    public HttpRequestMessage HttpRequestMessage { get; set; }
    public HttpResponseMessage HttpResponseMessage { get; set; }

... parsing code snipped ...

}
```

La instancia `HttpMessage` contiene un GUID `MessageId` que nos permite conectar la solicitud de HTTP a la respuesta de HTTP correspondiente y un valor booleano que identifica si el objeto contiene una instancia de HttpRequestMessage y HttpResponseMessage. Mediante las clases HTTP integradas de `System.Net.Http`, puede aprovechar el código de análisis `application/http` que se incluye en `System.Net.Http.Formatting`.  

### <a name="ihttpmessageprocessor"></a>IHttpMessageProcessor
La instancia `HttpMessage` se reenvía luego a la implementación de `IHttpMessageProcessor`, que es una interfaz que se creó para desacoplar la recepción y la interpretación del evento del centro de eventos de Azure y el procesamiento real de esta.

## <a name="forwarding-the-http-message"></a>Reenvío del mensaje HTTP
En este ejemplo, decidí que sería interesante insertar la solicitud HTTP en el [análisis de API de Moesif](https://www.moesif.com). Moesif es un servicio basado en la nube que se especializa en análisis y depuración HTTP. Tiene un nivel gratuito, por lo que resulta fácil probarlo y nos permite ver en tiempo real cómo fluyen las solicitudes de HTTP a través de nuestro servicio Administración de API.

Las implementación `IHttpMessageProcessor` tiene este aspecto:

```csharp
public class MoesifHttpMessageProcessor : IHttpMessageProcessor
{
    private readonly string RequestTimeName = "MoRequestTime";
    private MoesifApiClient _MoesifClient;
    private ILogger _Logger;
    private string _SessionTokenKey;
    private string _ApiVersion;
    public MoesifHttpMessageProcessor(ILogger logger)
    {
        var appId = Environment.GetEnvironmentVariable("APIMEVENTS-MOESIF-APP-ID", EnvironmentVariableTarget.Process);
        _MoesifClient = new MoesifApiClient(appId);
        _SessionTokenKey = Environment.GetEnvironmentVariable("APIMEVENTS-MOESIF-SESSION-TOKEN", EnvironmentVariableTarget.Process);
        _ApiVersion = Environment.GetEnvironmentVariable("APIMEVENTS-MOESIF-API-VERSION", EnvironmentVariableTarget.Process);
        _Logger = logger;
    }

    public async Task ProcessHttpMessage(HttpMessage message)
    {
        if (message.IsRequest)
        {
            message.HttpRequestMessage.Properties.Add(RequestTimeName, DateTime.UtcNow);
            return;
        }

        EventRequestModel moesifRequest = new EventRequestModel()
        {
            Time = (DateTime) message.HttpRequestMessage.Properties[RequestTimeName],
            Uri = message.HttpRequestMessage.RequestUri.OriginalString,
            Verb = message.HttpRequestMessage.Method.ToString(),
            Headers = ToHeaders(message.HttpRequestMessage.Headers),
            ApiVersion = _ApiVersion,
            IpAddress = null,
            Body = message.HttpRequestMessage.Content != null ? System.Convert.ToBase64String(await message.HttpRequestMessage.Content.ReadAsByteArrayAsync()) : null,
            TransferEncoding = "base64"
        };

        EventResponseModel moesifResponse = new EventResponseModel()
        {
            Time = DateTime.UtcNow,
            Status = (int) message.HttpResponseMessage.StatusCode,
            IpAddress = Environment.MachineName,
            Headers = ToHeaders(message.HttpResponseMessage.Headers),
            Body = message.HttpResponseMessage.Content != null ? System.Convert.ToBase64String(await message.HttpResponseMessage.Content.ReadAsByteArrayAsync()) : null,
            TransferEncoding = "base64"
        };

        Dictionary<string, string> metadata = new Dictionary<string, string>();
        metadata.Add("ApimMessageId", message.MessageId.ToString());

        EventModel moesifEvent = new EventModel()
        {
            Request = moesifRequest,
            Response = moesifResponse,
            SessionToken = _SessionTokenKey != null ? message.HttpRequestMessage.Headers.GetValues(_SessionTokenKey).FirstOrDefault() : null,
            Tags = null,
            UserId = null,
            Metadata = metadata
        };

        Dictionary<string, string> response = await _MoesifClient.Api.CreateEventAsync(moesifEvent);

        _Logger.LogDebug("Message forwarded to Moesif");
    }

    private static Dictionary<string, string> ToHeaders(HttpHeaders headers)
    {
        IEnumerable<KeyValuePair<string, IEnumerable<string>>> enumerable = headers.GetEnumerator().ToEnumerable();
        return enumerable.ToDictionary(p => p.Key, p => p.Value.GetEnumerator()
                                                         .ToEnumerable()
                                                         .ToList()
                                                         .Aggregate((i, j) => i + ", " + j));
    }
}
```

`MoesifHttpMessageProcessor` aprovecha las ventajas de una [biblioteca de API en C# para Moesif](https://www.moesif.com/docs/api?csharp#events) que facilita la inserción de datos de eventos HTTP en su servicio. Para enviar datos HTTP a Collector API de Moesif, necesita una cuenta y un identificador de aplicación. Para obtener el identificador de aplicación de Moesif, cree una cuenta en el [sitio web de Moesif](https://www.moesif.com) y, luego, vaya al _menú de la parte superior derecha_ -> _App Setup_ (Configuración de aplicación).

## <a name="complete-sample"></a>Ejemplo completo
El [código fuente](https://github.com/dgilling/ApimEventProcessor) y las pruebas del ejemplo se encuentran en GitHub. Necesita un [servicio API Management](get-started-create-service-instance.md), [un centro de eventos conectado](api-management-howto-log-event-hubs.md) y una [cuenta de almacenamiento](../storage/common/storage-account-create.md) para ejecutar el ejemplo por su cuenta.   

El ejemplo es simplemente una aplicación de consola sencilla que realiza escuchas de eventos procedentes del centro de eventos, los convierte en objetos `EventRequestModel` y `EventResponseModel` de Moesif y, luego, los reenvía a Collector API de Moesif.

En la siguiente imagen animada, puede ver una solicitud realizada a una API en el portal para desarrolladores, la aplicación de consola que muestra el mensaje que se recibe, procesa y reenvía y, a continuación, la solicitud y respuesta que se muestran en la secuencia de eventos.

![Demostración de solicitud se reenvía a Runscope](./media/api-management-log-to-eventhub-sample/apim-eventhub-runscope.gif)

## <a name="summary"></a>Resumen
El servicio Azure API Management proporciona un lugar ideal para capturar el tráfico HTTP hacia y desde la API. Azure Event Hubs es una solución altamente escalable y de bajo costo para capturar ese tráfico y colocarlo en sistemas de procesamiento secundario para registro, supervisión y otros análisis sofisticados. La conexión a sistemas de supervisión de tráfico de terceros como Moesif se reduce a unas cuantas docenas de líneas de código.

## <a name="next-steps"></a>Pasos siguientes
* Obtenga más información acerca de Azure Event Hubs
  * [Introducción a Azure Event Hubs](../event-hubs/event-hubs-c-getstarted-send.md)
  * [Recepción de mensajes con EventProcessorHost](../event-hubs/event-hubs-dotnet-standard-getstarted-send.md)
  * [Guía de programación de Event Hubs](../event-hubs/event-hubs-programming-guide.md)
* Obtener más información acerca de la integración de API Management y Event Hubs
  * [Cómo registrar eventos en Azure Event Hubs en Azure API Management](api-management-howto-log-event-hubs.md)
  * [Referencia de entidad del registrador](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-logger-entity)
  * [referencia de la directiva log-to-eventhub](./api-management-advanced-policies.md#log-to-eventhub)
