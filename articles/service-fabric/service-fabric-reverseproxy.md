---
title: Proxy inverso de Azure Service Fabric
description: Utilice un proxy inverso de Service Fabric para comunicarse de manera interna y externa con los microservicios a través del clúster.
author: BharatNarasimman
ms.topic: conceptual
ms.date: 11/03/2017
ms.author: bharatn
ms.custom: devx-track-csharp
ms.openlocfilehash: fd8e6dd712801de49971c1ef27cea664d73a4cb0
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89012779"
---
# <a name="reverse-proxy-in-azure-service-fabric"></a>Proxy inverso en Azure Service Fabric
El servidor proxy inverso creado en Azure Service Fabric ayuda a que los microservicios que se ejecutan en un clúster de Service Fabric detecten otros servicios que tienen puntos de conexión HTTP y se comuniquen con estos servicios.

## <a name="microservices-communication-model"></a>Modelo de comunicación de microservicios
Los microservicios de Service Fabric se ejecutan en un subconjunto de nodos del clúster y se pueden migrar entre los nodos por varias razones. Como resultado, los puntos de conexión de los microservicios pueden cambiar dinámicamente. Para detectar otros servicios y comunicarse con ellos en el clúster, el microservicio debe pasar por los siguientes pasos:

1. Resuelva la ubicación del servicio mediante el servicio de nombres.
2. Conéctese al servicio.
3. Ajuste los pasos anteriores en un bucle que implemente la solución de servicio y vuelva a intentar que se apliquen directivas con los errores de conexión.

Para más información, consulte [Conexión y comunicación con los servicios](service-fabric-connect-and-communicate-with-services.md).

### <a name="communicating-by-using-the-reverse-proxy"></a>Comunicación mediante el proxy inverso
El servidor proxy inverso es un servicio que se ejecuta en cada nodo y gestiona la resolución de puntos de conexión, el reintento automático y otros errores de conexión en nombre de los servicios para clientes. Se puede configurar para la aplicación de varias directivas cuando gestiona las solicitudes desde los servicios para clientes. El uso de un servidor proxy inverso permite que el servicio para clientes use las bibliotecas de comunicación HTTP del lado cliente, sin necesidad de resolución especial ni lógica de reintento en el servicio. 

El servidor proxy inverso expone uno o varios puntos de conexión en el nodo local de los servicios para clientes para su uso en el envío de solicitudes a otros servicios.

![Comunicación interna][1]

> [!NOTE]
> **Plataformas compatibles**
>
> El servidor proxy inverso de Service Fabric admite actualmente las siguientes plataformas:
> * *Clúster de Windows*: Windows 8 y posterior o Windows Server 2012 y posterior
> * *Clúster de Linux*: el servidor proxy inverso no está disponible actualmente para clústeres de Linux
>

## <a name="reaching-microservices-from-outside-the-cluster"></a>Comunicación externa con los microservicios a través del clúster
La comunicación externa predeterminada de los microservicios se basa en un modelo manual; es decir, no se puede acceder directamente a los servicios desde los clientes externos. [Azure Load Balancer](../load-balancer/load-balancer-overview.md) actúa como límite de red entre los microservicios y los clientes externos. Además, realiza la traducción de direcciones de red y reenvía las solicitudes externas a los puntos de conexión IP:port internos. Para hacer que los clientes externos puedan acceder directamente al punto de conexión de un microservicio, primero debe configurar Load Balancer para que reenvíe el tráfico a cada puerto que utilice el servicio del clúster. Además, la mayoría de los microservicios, especialmente los microservicios con estado, no se encuentran en todos los nodos del clúster. Los microservicios se pueden mover entre nodos durante la conmutación por error. En tales casos, el equilibrador de carga no puede determinar eficazmente la ubicación del nodo de destino de las réplicas al que debe reenviar el tráfico.

### <a name="reaching-microservices-via-the-reverse-proxy-from-outside-the-cluster"></a>Comunicación externa con los microservicios a través del clúster mediante el proxy inverso
En lugar de configurar el puerto de un servicio individual de Load Balancer, puede configurar solo el puerto del proxy inverso de Load Balancer. Esta configuración permite a los clientes externos al clúster comunicarse con los servicios internos de este a través del proxy inverso sin realizar más ajustes en la configuración.

![Comunicación externa][0]

> [!WARNING]
> Al configurar el puerto del proxy inverso de Load Balancer, se podrá acceder de manera externa a todos los microservicios del clúster que exponen un punto de conexión HTTP. Esto significa que un usuario malintencionado determinado podría detectar microservicios destinados a ser internos. Potencialmente, implica vulnerabilidades graves que podrían aprovecharse, como, por ejemplo:
>
> * Un usuario malintencionado puede iniciar un ataque de denegación de servicio mediante llamadas reiteradas a un servicio interno que no tenga una superficie expuesta a ataques suficientemente protegida.
> * Un usuario malintencionado puede entregar paquetes con formato incorrecto a un servicio interno y provocar un comportamiento no deseado.
> * Un servicio destinado a ser interno podría devolver información privada o confidencial que no se pretende exponer a servicios fuera del clúster, y ponerla así a disposición de un usuario malintencionado. 
>
> Asegúrese de entender completamente y de mitigar las posibles repercusiones para la seguridad del clúster y las aplicaciones que se ejecutan en este, antes de hacer público el puerto de proxy inverso. 
>


## <a name="uri-format-for-addressing-services-by-using-the-reverse-proxy"></a>Formato del URI para comunicarse con los servicios mediante el proxy inverso
El proxy inverso utiliza un formato de identificador uniforme de recursos (URI) específico para identificar a qué partición de servicio se debe reenviar la solicitud entrante:

```
http(s)://<Cluster FQDN | internal IP>:Port/<ServiceInstanceName>/<Suffix path>?PartitionKey=<key>&PartitionKind=<partitionkind>&ListenerName=<listenerName>&TargetReplicaSelector=<targetReplicaSelector>&Timeout=<timeout_in_seconds>
```

* **http(s):** el proxy inverso puede configurarse para que acepte tráfico HTTP o HTTPS. Para el reenvío HTTPS, vea [Connect to a secure service with the reverse proxy](service-fabric-reverseproxy-configure-secure-communication.md) (Conectar a un servicio seguro con el proxy inverso) después de configurar el proxy inverso para que escuche en HTTPS.
* **Nombre de dominio completo del clúster (FQDN) | dirección IP interna:** para clientes externos, se puede configurar el proxy inverso para que sea accesible a través del dominio del clúster (por ejemplo, micluster.eastus.cloudapp.azure.com). De forma predeterminada, el proxy inverso se ejecuta en todos los nodos. Para el tráfico interno se puede acceder al proxy inverso en el host local o en cualquier dirección IP del nodo interno (por ejemplo, 10.0.0.1).
* **Puerto:** el puerto especificado para el proxy inverso. Por ejemplo, el puerto 19081.
* **ServiceInstanceName:** se trata del nombre completo de la instancia de servicio implementada con la que está tratando de comunicarse (sin el esquema "fabric:/"). Por ejemplo, para establecer conexión con el servicio *fabric:/miAplicación/miServicio/* , podría usar *miaplicación/miservicio*.

    El nombre de instancia del servicio distingue mayúsculas de minúsculas. Un uso de mayúsculas y minúsculas diferente para el nombre de instancia del servicio en la dirección URL hace que se produzca el error 404 (No encontrado) en las solicitudes.
* **Ruta de acceso de sufijo:** se trata de la ruta de acceso URL real del servicio con el que quiere conectarse. Por ejemplo, *myapi/values/add/3*.
* **PartitionKey:** en un servicio particionado, corresponde a la clave de partición calculada de la partición con la que quiere conectarse. Tenga en cuenta que *no* se trata del identificador GUID de la partición. Este parámetro no es necesario para los servicios que utilizan el esquema de partición de singleton.
* **PartitionKind:** es el esquema de particiones del servicio. El valor puede ser Int64Range o Con nombre. Este parámetro no es necesario para los servicios que utilizan el esquema de partición de singleton.
* **ListenerName:** los puntos de conexión del servicio presentan la forma {"Endpoints":{"Listener1":"Endpoint1","Listener2":"Endpoint2" ...}}. Cuando el servicio expone varios puntos de conexión, esto identifica a qué puntos de conexión se debe reenviar la solicitud del cliente. Esto se puede omitir si el servicio tiene solo un agente de escucha.
* **TargetReplicaSelector:** especifica cómo se debe seleccionar la instancia o la réplica de destino.
  * Cuando el servicio de destino tiene estado, el valor de TargetReplicaSelector puede ser uno de los siguientes:  "PrimaryReplica", "RandomSecondaryReplica" o "RandomReplica". El valor predeterminado cuando no se especifica este parámetro es "PrimaryReplica".
  * Cuando el servicio de destino no tiene estado, el proxy inverso elige una instancia aleatoria de la partición de servicio a la que reenviar la solicitud.
* **Tiempo de espera:**  especifica el tiempo de espera de la solicitud HTTP que crea el proxy inverso para conectarse al servicio en nombre de la solicitud de cliente. El valor predeterminado es 120 segundos. Se trata de un parámetro opcional.

### <a name="example-usage"></a>Ejemplo de uso
A modo de ejemplo, veamos el servicio *fabric:/miAplicación/miServicio* que abre un agente de escucha HTTP en la siguiente dirección URL:

```
http://10.0.0.5:10592/3f0d39ad-924b-4233-b4a7-02617c6308a6-130834621071472715/
```

Los recursos para el servicio son los siguientes:

* `/index.html`
* `/api/users/<userId>`

Si el servicio utiliza el esquema de particiones de singleton, no se requieren los parámetros de cadena de consulta *PartitionKey* y *PartitionKind*, y se podrá acceder a él mediante la puerta de enlace de las siguientes formas:

* Externamente: `http://mycluster.eastus.cloudapp.azure.com:19081/MyApp/MyService`
* Internamente: `http://localhost:19081/MyApp/MyService`

Si el servicio usa el esquema de particiones Int64 uniforme, deben utilizarse los parámetros de cadena de consulta *PartitionKey* y *PartitionKind* para comunicarse con una partición del servicio:

* Externamente: `http://mycluster.eastus.cloudapp.azure.com:19081/MyApp/MyService?PartitionKey=3&PartitionKind=Int64Range`
* Internamente: `http://localhost:19081/MyApp/MyService?PartitionKey=3&PartitionKind=Int64Range`

Para acceder a los recursos que expone el servicio, basta con colocar su ruta de acceso después del nombre del servicio en la URL:

* Externamente: `http://mycluster.eastus.cloudapp.azure.com:19081/MyApp/MyService/index.html?PartitionKey=3&PartitionKind=Int64Range`
* Internamente: `http://localhost:19081/MyApp/MyService/api/users/6?PartitionKey=3&PartitionKind=Int64Range`

Después, la puerta de enlace reenviará estas solicitudes a la URL del servicio:

* `http://10.0.0.5:10592/3f0d39ad-924b-4233-b4a7-02617c6308a6-130834621071472715/index.html`
* `http://10.0.0.5:10592/3f0d39ad-924b-4233-b4a7-02617c6308a6-130834621071472715/api/users/6`

## <a name="special-handling-for-port-sharing-services"></a>Control especial de los servicios de uso compartido de puertos
Un proxy inverso de Service Fabric intenta volver a resolver una dirección de servicio y a realizar la solicitud cuando no se puede establecer la comunicación con un servicio. Generalmente, el motivo por el que no podemos comunicarnos con un servicio es porque su instancia o réplica se han movido a otro nodo como parte del ciclo de vida normal. Cuando esto sucede, es posible que el proxy inverso reciba un error de conexión de red que indica que un punto de conexión ya no está abierto en la dirección resuelta originalmente.

Sin embargo, las instancias del servicio o las réplicas pueden compartir un proceso de host y un puerto cuando un servidor web basado en http.sys las hospeda; por ejemplo:

* [System.Net.HttpListener](/dotnet/api/system.net.httplistener?view=netcore-3.1)
* [ASP.NET Core WebListener](https://docs.asp.net/latest/fundamentals/servers.html#weblistener)
* [Katana](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.OwinSelfHost/)

En este caso, es probable que el servidor web esté disponible en el proceso de host y que responda a las solicitudes, pero no la instancia del servicio resuelta o la réplica. Si esto ocurre, la puerta de enlace recibirá una respuesta HTTP 404 del servidor web. Por lo tanto, una respuesta HTTP 404 puede tener dos significados distintos:

- Caso 1: la dirección de servicio es correcta, pero no existe el recurso que ha solicitado el usuario.
- Caso 2: la dirección de servicio es incorrecta y puede que el recurso que ha solicitado el usuario exista en un nodo diferente.

En el primer caso, se trata de una respuesta HTTP 404 normal que se considera un error de usuario. Sin embargo, en el segundo caso, el usuario ha solicitado un recurso que ya existe. El proxy inverso no pudo encontrarlo porque se había movido el servicio. El proxy inverso debe resolver la dirección de nuevo y volver a intentar la solicitud.

Por tanto, el proxy inverso necesita una manera de diferenciar estos dos casos. Para que pueda hacerlo, necesita una indicación del servidor.

* De manera predeterminada, el proxy inverso da por hecho que es el segundo caso y trata de volver a resolver y emitir la solicitud.
* Para que sepa que se trata del primero, el servicio debe devolver el siguiente encabezado de respuesta HTTP:

  `X-ServiceFabric : ResourceNotFound`

Este encabezado de respuesta HTTP indica una situación de HTTP 404 normal; es decir, el recurso solicitado no existe y el proxy inverso no trata de volver a resolver la dirección de servicio.

## <a name="special-handling-for-services-running-in-containers"></a>Control especial para los servicios que se ejecutan en contenedores

Para los servicios que se ejecutan en contenedores, puede usar la variable de entorno `Fabric_NodeIPOrFQDN` para construir la [dirección URL del proxy inverso](#uri-format-for-addressing-services-by-using-the-reverse-proxy) como se muestra en el código siguiente:

```csharp
    var fqdn = Environment.GetEnvironmentVariable("Fabric_NodeIPOrFQDN");
    var serviceUrl = $"http://{fqdn}:19081/DockerSFApp/UserApiContainer";
```
Para el clúster local, `Fabric_NodeIPOrFQDN` está establecido en "localhost" de forma predeterminada. Inicie el clúster local con el parámetro `-UseMachineName` para asegurarse de que los contenedores pueden llegar al proxy inverso que se ejecuta en el nodo. Para más información, consulte [Configuración del entorno de desarrollo para depurar contenedores](service-fabric-how-to-debug-windows-containers.md#configure-your-developer-environment-to-debug-containers).

Los servicios de Service Fabric que se ejecutan dentro de contenedores de Docker Compose requieren una configuración http: o https: especial para la *sección de puertos* del archivo docker-compose.yml. Para obtener más información, consulte [Compatibilidad con la implementación de Docker Compose en Azure Service Fabric](service-fabric-docker-compose.md).

## <a name="next-steps"></a>Pasos siguientes
* [Instalación y configuración del proxy inverso en un clúster](service-fabric-reverseproxy-setup.md)
* [Configuración del reenvío para proteger el servicio HTTP con el proxy inverso](service-fabric-reverseproxy-configure-secure-communication.md)
* [Diagnóstico de eventos de proxy inverso](service-fabric-reverse-proxy-diagnostics.md)
* Vea un ejemplo de comunicación HTTP entre los servicios de un [proyecto de ejemplo en GitHub](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started).
* [Llamadas a procedimiento remoto con la comunicación remota de Reliable Services](service-fabric-reliable-services-communication-remoting.md)
* [API web que usa OWIN en Reliable Services](./service-fabric-reliable-services-communication-aspnetcore.md)
* [Comunicación WCF con la utilización de Reliable Services](service-fabric-reliable-services-communication-wcf.md)

[0]: ./media/service-fabric-reverseproxy/external-communication.png
[1]: ./media/service-fabric-reverseproxy/internal-communication.png
