---
title: Conexión y comunicación con servicios en Azure Service Fabric
description: Aprenda a resolver, conectar y comunicar mediante servicios de Service Fabric.
author: vturecek
ms.topic: conceptual
ms.date: 11/01/2017
ms.author: vturecek
ms.custom: devx-track-csharp
ms.openlocfilehash: cf39fcbfbde8a81400cd93c7f99b066a99f643bd
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89005386"
---
# <a name="connect-and-communicate-with-services-in-service-fabric"></a>Conexión y comunicación con servicios en Service Fabric
En Service Fabric, un servicio se ejecuta en algún lugar en un clúster de Service Fabric que normalmente se distribuye entre varias máquinas virtuales. Se puede mover de un lugar a otro, ya sea por indicación del propietario del servicio o automáticamente mediante Service Fabric. Los servicios no están enlazados estáticamente a un equipo o una dirección determinados.

Una aplicación de Service Fabric se compone, por lo general, de muchos servicios diferentes, donde cada uno de ellos realiza una tarea especializada. Estos servicios pueden comunicarse entre sí para formar una función completa, como representar diferentes partes de una aplicación web. También hay aplicaciones cliente que se conectan a los servicios y se comunican con ellos. Este documento describe cómo establecer la comunicación con los servicios de Service Fabric y entre ellos.

## <a name="bring-your-own-protocol"></a>Traiga su propio protocolo
Service Fabric le ayuda a administrar el ciclo de vida de los servicios, pero no toma ninguna decisión sobre qué hacen sus servicios. Esto incluye la comunicación. Cuando Service Fabric abre un servicio, este tiene la oportunidad de configurar un punto de conexión para las solicitudes entrantes mediante cualquier protocolo o pila de comunicación que desee. El servicio escuchará en una dirección **IP:port** normal mediante cualquier esquema de direccionamiento, como un URI. Varias instancias de servicio o réplicas pueden compartir un proceso de host, en cuyo caso deberán usar puertos diferentes o utilizar un mecanismo de uso compartido de puertos, como el controlador del kernel http.sys en Windows. En ambos casos, cada instancia de servicio o réplica de un proceso de host debe ser direccionable de forma exclusiva.

![puntos de conexión del servicio][1]

## <a name="service-discovery-and-resolution"></a>Detección y resolución de servicios
En un sistema distribuido, los servicios se pueden mover de una máquina a otra con el tiempo. Esto puede ocurrir por diversos motivos, incluidos el equilibrio, las actualizaciones, las conmutaciones por error o el escalado de recursos. Esto significa que el punto de conexión del servicio experimenta cambios a medida que el servicio se mueve a nodos con direcciones IP diferentes y se puede abrir en puertos diferentes si el servicio utiliza un puerto seleccionado de forma dinámica.

![Distribución de servicios][7]

Service Fabric proporciona un servicio de detección y resolución denominado servicio de nomenclatura. El servicio de nomenclatura mantiene una tabla que asigna instancias de servicio con nombre a las direcciones del punto de conexión a las que escuchan. Todas las instancias de servicio con nombre en Service Fabric tienen nombres únicos como URI como, por ejemplo, `"fabric:/MyApplication/MyService"`. El nombre del servicio no cambia durante el ciclo de vida del mismo, solo las direcciones de punto de conexión pueden cambiar cuando se mueve el servicio. Esto es análogo a los sitios web que tienen direcciones URL constantes, pero donde la dirección IP puede cambiar. Y de forma similar a un DNS en la web, que resuelve direcciones URL del sitio web en direcciones IP, Service Fabric tiene un registrador que asigna nombres de servicio a la dirección del punto de conexión.

![puntos de conexión del servicio][2]

La resolución y conexión a servicios implica que los pasos siguientes se ejecuten en un bucle:

* **Resolver**: obtenga el punto de conexión que publicó un servicio mediante Naming Service.
* **Conectar**: conéctese al servicio a través de cualquier protocolo que se use en ese punto de conexión.
* **Reintentar**: un intento de conexión puede producir un error por diversos motivos; por ejemplo, si el servicio se movió desde la última vez que se resolvió la dirección del punto de conexión. En ese caso, se deben probar de nuevo los pasos de conexión y resolución anteriores, y este ciclo se repetirá hasta que la conexión sea correcta.

## <a name="connecting-to-other-services"></a>Conexión a otros servicios
Los servicios que se conectan entre sí dentro de un clúster pueden acceder, por lo general, directamente a los puntos de conexión de otros servicios ya que los nodos de un clúster se encuentran en la misma red local. A fin de facilitar la conexión entre servicios, Service Fabric ofrece servicios adicionales que usan el Servicio de nombres. Un servicio DNS y un servicio de proxy inverso.


### <a name="dns-service"></a>Servicio DNS
Debido a que muchos servicios, sobre todo los servicios en contenedores, pueden tener un nombre de dirección URL existente, tener la capacidad de resolverlo mediante el protocolo DNS estándar (en lugar del protocolo de servicio de nombres) es muy conveniente, en particular en escenarios "lift-and-shift" de la aplicación. Esto es exactamente lo que hace el servicio DNS. Permite asignar nombres DNS a un nombre de servicio y, por tanto, resolver direcciones IP del punto de conexión. 

Como se muestra en el diagrama siguiente, el servicio DNS, que se ejecuta en el clúster de Service Fabric, asigna los nombres DNS a los nombres de servicio que, a continuación, se resuelven utilizando el Servicio de nombres para devolver las direcciones del punto de conexión a las que conectarse. El nombre DNS para el servicio se proporciona en el momento de la creación. 

![puntos de conexión del servicio][9]

Para más información sobre cómo usar el servicio DNS, vea el artículo [Servicio DNS en Azure Service Fabric](service-fabric-dnsservice.md).

### <a name="reverse-proxy-service"></a>Servicio de proxy inverso
Los servicios de direcciones de proxy inverso del clúster que expone puntos de conexión HTTP, incluidos HTTPS. El proxy inverso simplifica en gran medida la llamada a otros servicios y a sus métodos, mediante la disponibilidad de un formato de identificador URI específico y, además, controla los pasos de resolución, conexión y reintento necesarios para que un servicio se comunique con otro mediante el servicio de nomenclatura. En otras palabras, oculta el Servicio de nombres al usuario al realizar llamadas a otros servicios, por lo que resulta tan fácil como realizar una llamada a una dirección URL.

![puntos de conexión del servicio][10]

Para más información sobre cómo usar el servicio de proxy inverso, vea el artículo [Proxy inverso en Azure Service Fabric](service-fabric-reverseproxy.md).

## <a name="connections-from-external-clients"></a>Conexiones desde clientes externos
Los servicios que se conectan entre sí dentro de un clúster pueden acceder, por lo general, directamente a los puntos de conexión de otros servicios ya que los nodos de un clúster se encuentran en la misma red local. Sin embargo, en algunos entornos, un clúster puede estar detrás de un equilibrador de carga que enruta el tráfico de entrada a través de un conjunto limitado de puertos. En estos casos, los servicios todavía pueden comunicarse entre sí y resolver direcciones mediante el servicio de nomenclatura, pero se deben realizar pasos adicionales para permitir que los clientes externos se conecten a los servicios.

## <a name="service-fabric-in-azure"></a>Service Fabric en Azure
Un clúster de Service Fabric en Azure se coloca detrás de un equilibrador de carga de Azure. Todo el tráfico externo al clúster debe pasar a través del equilibrador de carga. El equilibrador de carga reenviará automáticamente el tráfico entrante de un puerto determinado a un *nodo* aleatorio que tenga el mismo puerto abierto. Azure Load Balancer solo detecta los puertos abiertos en los *nodos*, no detecta puertos abiertos por *servicios* individuales.

![Topología de equilibrador de carga de Azure y Service Fabric][3]

Por ejemplo, para poder aceptar tráfico externo en el puerto **80**, se deben configurar las siguientes condiciones:

1. Escriba un servicio que escuche en el puerto 80. Configure el puerto 80 en el archivo ServiceManifest.xml del servicio y abra un agente de escucha en el mismo, por ejemplo, un servidor web autohospedado.

    ```xml
    <Resources>
        <Endpoints>
            <Endpoint Name="WebEndpoint" Protocol="http" Port="80" />
        </Endpoints>
    </Resources>
    ```
    ```csharp
        class HttpCommunicationListener : ICommunicationListener
        {
            ...

            public Task<string> OpenAsync(CancellationToken cancellationToken)
            {
                EndpointResourceDescription endpoint =
                    serviceContext.CodePackageActivationContext.GetEndpoint("WebEndpoint");

                string uriPrefix = $"{endpoint.Protocol}://+:{endpoint.Port}/myapp/";

                this.httpListener = new HttpListener();
                this.httpListener.Prefixes.Add(uriPrefix);
                this.httpListener.Start();

                string publishUri = uriPrefix.Replace("+", FabricRuntime.GetNodeContext().IPAddressOrFQDN);
                return Task.FromResult(publishUri);
            }

            ...
        }

        class WebService : StatelessService
        {
            ...

            protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
            {
                return new[] { new ServiceInstanceListener(context => new HttpCommunicationListener(context))};
            }

            ...
        }
    ```
    ```java
        class HttpCommunicationlistener implements CommunicationListener {
            ...

            @Override
            public CompletableFuture<String> openAsync(CancellationToken arg0) {
                EndpointResourceDescription endpoint =
                    this.serviceContext.getCodePackageActivationContext().getEndpoint("WebEndpoint");
                try {
                    HttpServer server = com.sun.net.httpserver.HttpServer.create(new InetSocketAddress(endpoint.getPort()), 0);
                    server.start();

                    String publishUri = String.format("http://%s:%d/",
                        this.serviceContext.getNodeContext().getIpAddressOrFQDN(), endpoint.getPort());
                    return CompletableFuture.completedFuture(publishUri);
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }

            ...
        }

        class WebService extends StatelessService {
            ...

            @Override
            protected List<ServiceInstanceListener> createServiceInstanceListeners() {
                <ServiceInstanceListener> listeners = new ArrayList<ServiceInstanceListener>();
                listeners.add(new ServiceInstanceListener((context) -> new HttpCommunicationlistener(context)));
                return listeners;       
            }

            ...
        }
    ```
2. Crear un clúster de Service Fabric en Azure y especificar el puerto **80** como un puerto de punto de conexión personalizado para el tipo de nodo que va a hospedar el servicio. Si tiene más de un tipo de nodo, puede configurar una *restricción de colocación* en el servicio para asegurarse de que solo se puede ejecutar en el tipo de nodo que tiene el puerto de punto de conexión personalizado abierto.

    ![Apertura de un puerto en un tipo de nodo][4]
3. Una vez creado el clúster, configure el equilibrador de carga de Azure en el grupo de recursos del clúster para reenviar el tráfico hacia el puerto 80. Al crear un clúster mediante Azure Portal, este se configura automáticamente para cada puerto de punto de conexión personalizado que se ha configurado.

    ![Reenvío del tráfico en el equilibrador de carga de Azure][5]
4. El equilibrador de carga de Azure utiliza un sondeo para determinar si envía o no tráfico a un nodo concreto. El sondeo comprueba periódicamente un punto de conexión de cada nodo para determinar si este está respondiendo o no. Si el sondeo no recibe una respuesta después de un número determinado de veces, el equilibrador de carga deja de enviar tráfico a dicho nodo. Al crear un clúster mediante Azure Portal, se configura automáticamente un sondeo para cada puerto de punto de conexión personalizado que se ha configurado.

    ![Reenvío del tráfico en el equilibrador de carga de Azure][8]

Es importante recordar que Azure Load Balancer y el sondeo solo detectan los *nodos* y no los *servicios* que se ejecutan en dichos nodos. El equilibrador de carga de Azure siempre enviará tráfico a los nodos que respondan al sondeo, por lo que se debe garantizar que haya servicios disponibles en los nodos que puedan responder al sondeo.

## <a name="reliable-services-built-in-communication-api-options"></a>Reliable Services: Opciones de API de comunicación incorporadas
El marco de Reliable Services incluye varias opciones de comunicación pregeneradas. La decisión de cuál funcionará mejor para usted depende de la elección del modelo de programación, el marco de comunicación y el lenguaje de programación en que se crean los servicios.

* **No hay ningún protocolo específico:**  si no va a usar un marco de comunicación concreto, pero quiere crear algo que se pueda ejecutar rápidamente, entonces la opción idónea para usted es la [comunicación remota de servicio](service-fabric-reliable-services-communication-remoting.md), que permite realizar llamadas a procedimientos remotos fuertemente tipados para Reliable Services y Reliable Actors. Esta es la manera más sencilla y rápida de empezar con la comunicación del servicio. La comunicación remota de servicio controla la resolución de direcciones, la conexión, los reintentos y el control de errores del servicio. Esto está disponible para aplicaciones de C# y Java.
* **HTTP**: para una comunicación independiente del lenguaje, HTTP proporciona una opción estándar con las herramientas y los servidores HTTP disponibles en muchos lenguajes diferentes, compatibles todos ellos con Service Fabric. Los servicios pueden usar cualquier pila HTTP disponible, incluida la [ASP.NET Web API](./service-fabric-reliable-services-communication-aspnetcore.md) para aplicaciones de C#. Los clientes escritos en C# pueden aprovechar las clases `ICommunicationClient` y `ServicePartitionClient`, mientras que para Java son las clases `CommunicationClient` y `FabricServicePartitionClient`, [para la resolución del servicio, las conexiones HTTP y los bucles de reintento](service-fabric-reliable-services-communication.md).
* **WCF**: si tiene código que usa WCF como marco de comunicación, puede usar `WcfCommunicationListener` para el lado servidor y las clases `WcfCommunicationClient` y `ServicePartitionClient` para el cliente. Pero esto solo está disponible para las aplicaciones de C# en clústeres basados en Windows. Para obtener más información, consulte este artículo sobre la [implementación basada en WCF de la pila de comunicación](service-fabric-reliable-services-communication-wcf.md).

## <a name="using-custom-protocols-and-other-communication-frameworks"></a>Uso de protocolos personalizados y otros marcos de comunicación
Los servicios pueden usar cualquier protocolo o marco de comunicación, ya sea un protocolo binario personalizado sobre sockets de TCP o eventos de streaming a través de [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/) o [IoT Hub de Azure](https://azure.microsoft.com/services/iot-hub/). Service Fabric proporciona las API de comunicación a las que puede conectar su pila de comunicación, al tiempo que se le quita todo el trabajo de detección y conexión. Consulte el artículo sobre el [Modelo de comunicación de Reliable Services](service-fabric-reliable-services-communication.md) para obtener más detalles.

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre los conceptos y las API disponibles en el [modelo de comunicación de Reliable Services](service-fabric-reliable-services-communication.md) y, después, empiece a trabajar rápidamente con [comunicación remota de servicio](service-fabric-reliable-services-communication-remoting.md) o profundice para más información sobre cómo escribir un agente de escucha de comunicación mediante la [API web con autohospedaje OWIN](./service-fabric-reliable-services-communication-aspnetcore.md).

[1]: ./media/service-fabric-connect-and-communicate-with-services/serviceendpoints.png
[2]: ./media/service-fabric-connect-and-communicate-with-services/namingservice.png
[3]: ./media/service-fabric-connect-and-communicate-with-services/loadbalancertopology.png
[4]: ./media/service-fabric-connect-and-communicate-with-services/nodeport.png
[5]: ./media/service-fabric-connect-and-communicate-with-services/loadbalancerport.png
[7]: ./media/service-fabric-connect-and-communicate-with-services/distributedservices.png
[8]: ./media/service-fabric-connect-and-communicate-with-services/loadbalancerprobe.png
[9]: ./media/service-fabric-connect-and-communicate-with-services/dns.png
[10]: ./media/service-fabric-reverseproxy/internal-communication.png
