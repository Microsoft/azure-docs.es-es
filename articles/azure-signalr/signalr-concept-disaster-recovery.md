---
title: Resistencia y recuperación ante desastres en Azure SignalR Service
description: Información general sobre la configuración de varias instancias de servicio de SignalR para conseguir resistencia y recuperación ante desastres
author: chenkennt
ms.service: signalr
ms.topic: conceptual
ms.date: 03/01/2019
ms.author: kenchen
ms.openlocfilehash: eb70e65db4a086afc60e91cadf55a8844b102591
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2019
ms.locfileid: "61402156"
---
# <a name="resiliency-and-disaster-recovery"></a>Resistencia y recuperación ante desastres

La resistencia y la recuperación ante desastres son necesidades comunes de los sistemas en línea. Azure SignalR Service ya garantiza una disponibilidad del 99,9 %, pero sigue siendo un servicio regional.
La instancia del servicio siempre se ejecuta en una región y no conmutación por error a otra región cuando se produzca una interrupción de toda la región.

En su lugar, nuestro SDK de servicio proporciona una funcionalidad para admitir varias instancias de SignalR Service y cambiar automáticamente a otras instancias cuando algunas no estén disponibles.
Con esta característica, podrá realizar la recuperación cuando se produzca un desastre, pero deberá configurar la topología de sistema adecuada usted mismo. En este documento se muestra cómo hacerlo.

## <a name="high-available-architecture-for-signalr-service"></a>Arquitectura de alta disponibilidad para SignalR Service

Para tener resistencia entre regiones para SignalR Service, deberá configurar varias instancias de servicio en diferentes regiones. De manera que cuando una región esté inactiva, las demás se puedan usar como reserva.
Al conectar varias instancias de servicio al servidor de aplicaciones, hay dos roles, principal y secundario.
La principal es una instancia con el tráfico en línea y la secundaria, una instancia totalmente funcional pero de reserva de la principal.
En nuestra implementación de SDK, la negociación solo devuelve puntos de conexión principales, ya que normalmente los clientes solo se conectan a los puntos de conexión principales.
Pero cuando la instancia principal está inactiva, la negociación devuelve puntos de conexión secundarios para que el cliente pueda continuar realizando conexiones.
Instancia principal y del servidor de aplicaciones se conectan a través de conexiones de servidor normal pero instancia secundaria y del servidor de aplicaciones se conectan a través de un tipo especial de conexión denominado conexión débil.
La principal diferencia de una conexión débil es que no acepta el enrutamiento de conexión de cliente, porque la instancia secundaria se encuentra en otra región. Enrutamiento de un cliente en otra región no es una opción óptima (aumenta la latencia).

Una instancia del servicio puede tener distintos roles cuando se conecta a varios servidores de aplicaciones.
Una configuración típica de un escenario de regiones cruzadas tiene dos (o más) pares de instancias de servicio de SignalR y servidores de aplicaciones.
En cada par, el servidor de aplicaciones y SignalR Service se encuentran en la misma región y este último se conecta al servidor de aplicaciones como rol principal.
Entre los pares, el servidor de aplicaciones y SignalR Service también están conectados, pero este último se vuelve secundario al conectarse al servidor de otra región.

Con esta topología, todavía se puede entregar el mensaje de un servidor a todos los clientes, ya que todos los servidores de aplicaciones e instancias de servicio de SignalR están interconectados.
Pero cuando se conecta un cliente, siempre se enruta al servidor de aplicaciones de la misma región para lograr la latencia de red óptima.

A continuación, un diagrama que ilustra esta topología:

![Topología](media/signalr-concept-disaster-recovery/topology.png)

## <a name="configure-app-servers-with-multiple-signalr-service-instances"></a>Configuración de servidores de aplicaciones con varias instancias de servicio de SignalR

Una vez creados los servidores de aplicaciones y SignalR Service en cada región, puede configurar los primeros para que se conecten a todas las instancias de servicio de SignalR.

Esto se puede hacer de dos maneras:

### <a name="through-config"></a>Mediante configuración

Debía haber sido cómo establecer la cadena de conexión de SignalR service a través de settings/web.cofig aplicación o las variables de entorno, a través de una entrada de configuración denominada `Azure:SignalR:ConnectionString`.
Si tiene varios puntos de conexión, puede establecerlos en varias entradas de configuración, cada una de ellas con el siguiente formato:

```
Azure:SignalR:Connection:<name>:<role>
```

Aquí, `<name>` es el nombre del punto de conexión y `<role>`, su rol (principal o secundario).
El nombre es opcional, pero será útil si desea personalizar aún más el comportamiento de enrutamiento entre varios puntos de conexión.

### <a name="through-code"></a>Mediante código

Si prefiere almacenar la cadena de conexión en alguna otra parte, también puede leerla en el código y usarla como parámetros al llamar a `AddAzureSignalR()` (en ASP.NET Core) o `MapAzureSignalR()` (en ASP.NET).

Este es el código de ejemplo:

ASP.NET Core:

```cs
services.AddSignalR()
        .AddAzureSignalR(options => options.Endpoints = new ServiceEndpoint[]
        {
            new ServiceEndpoint("<connection_string1>", EndpointType.Primary, "region1"),
            new ServiceEndpoint("<connection_string2>", EndpointType.Secondary, "region2"),
        });
```

ASP.NET:

```cs
app.MapAzureSignalR(GetType().FullName, hub,  options => options.Endpoints = new ServiceEndpoint[]
    {
        new ServiceEndpoint("<connection_string1>", EndpointType.Primary, "region1"),
        new ServiceEndpoint("<connection_string2>", EndpointType.Secondary, "region2"),
    };
```

## <a name="failover-sequence-and-best-practice"></a>Secuencia de conmutación por error y procedimiento recomendado

Ahora tiene la configuración de la topología de sistema correcta. Cada vez que una instancia de servicio de SignalR está inactiva, el tráfico en línea se enruta a otras instancias.
Esto es lo que ocurre cuando una instancia principal está inactiva (y se recupera tras algún tiempo):

1. La instancia principal está inactiva, todas las conexiones a los servidores de esa instancia se interrumpen.
2. Todos los servidores conectados a esta instancia se marcan como sin conexión y la negociación dejará de devolver este punto de conexión y pasará a devolver el secundario.
3. También se cerrarán todas las conexiones de cliente en esta instancia y se volverá a conectar a los clientes. Puesto que los servidores de aplicaciones ahora devuelven el punto de conexión secundario, los clientes se conectarán a la instancia secundaria.
4. Ahora la instancia secundaria toma todo el tráfico en línea. Todos los mensajes del servidor a los clientes pueden entregarse, ya que la instancia secundaria está conectada a todos los servidores de aplicaciones. Pero los mensajes del cliente al servidor solo se enrutan al servidor de aplicaciones de la misma región.
5. Una vez recuperada y en línea la instancia principal, el servidor de aplicaciones restablecerá las conexiones a ella y la marcará como "en línea". La negociación ahora vuelve a devolver el punto de conexión principal, de manera que los clientes se vuelven a conectar a la instancia principal. Pero los clientes existentes no se desconectarán y continuarán enrutados a la instancia secundaria hasta que ellos mismos de desconecten.

A continuación, los diagramas muestran cómo se realiza la conmutación por error en SignalR Service:

Ilustración 1. Antes de la conmutación por error ![Antes de la conmutación por error](media/signalr-concept-disaster-recovery/before-failover.png)

Ilustración 2. Después de la conmutación por error ![Después de la conmutación por error](media/signalr-concept-disaster-recovery/after-failover.png)

Ilustración 3. Poco tiempo después de la recuperación de la instancia principal ![Poco tiempo después de la recuperación de la instancia principal](media/signalr-concept-disaster-recovery/after-recover.png)

Normalmente, solo el servidor de aplicaciones y SignalR Service principales tienen tráfico en línea (en azul).
Después de la conmutación por error, el servidor de aplicaciones y SignalR Service secundarios también se activan.
Cuando SignalR Service principal vuelve a estar en línea, los nuevos clientes se conectarán a él. Pero los clientes existentes se seguirán conectando a la instancia secundaria, por lo que ambas instancias tendrán tráfico.
Una vez desconectados todos los clientes existentes, el sistema volverá a la normalidad (ilustración 1).

Hay dos patrones principales para implementar una arquitectura de alta disponibilidad entre regiones:

1. La primera de ellas consiste en tener un par de servidor de aplicaciones e instancia de servicio de SignalR con todo el tráfico en línea y otro par como copia de seguridad (lo que se denomina "activo/pasivo" en la ilustración 1). 
2. El otro es tener dos (o más) pares de servidores de aplicaciones e instancias de servicio de SignalR y que cada uno tenga parte del tráfico en línea y sirva de copia de seguridad para los demás pares (lo que se denomina "activo/pasivo" en la ilustración 3).

SignalR Service admite ambos patrones, la principal diferencia es la manera de implementar los servidores de aplicaciones.
Si estos son de tipo "activo/pasivo", SignalR también será "activo/pasivo" (ya que el servidor de aplicaciones principal solo devuelve la instancia de servicio de SignalR principal).
Si los servidores de aplicaciones son "activo/activo", SignalR también será "activo/activo" (ya que todos los servidores de aplicaciones devolverán sus propias instancias de SignalR principales, por lo que todos pueden tener tráfico).

Tener en cuenta con independencia de qué patrones opta por usar, deberá conectarse a cada instancia del servicio SignalR a un servidor de aplicaciones como principal.

Además, por la naturaleza de la conexión de SignalR (es una conexión larga), los clientes experimentarán interrupciones de conexión cuando se produzca un desastre y una conmutación por error.
Deberá tratar con esos casos desde el cliente para que sea transparente para los clientes finales. Por ejemplo, vuelva a realizar la conexión cuando se cierre.

## <a name="next-steps"></a>Pasos siguientes

En este artículo ha aprendido a configurar la aplicación para lograr resistencia para SignalR Service. Para conocer más detalles acerca de la conexión cliente/servidor y el enrutamiento de conexión en SignalR Service, consulte [este artículo](signalr-concept-internals.md) para los aspectos internos de SignalR Service.

¿Para escalar escenarios como el particionamiento, que usan varias instancias juntas para controlar el gran número de conexiones, leer [cómo escalar varias instancias](signalr-howto-scale-multi-instances.md)?