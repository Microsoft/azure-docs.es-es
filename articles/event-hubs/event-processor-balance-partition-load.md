---
title: 'Equilibrio de carga de particiones entre varias instancias: Azure Event Hubs | Microsoft Docs'
description: Describe cómo equilibrar la carga de las particiones entre varias instancias de una aplicación mediante un procesador de eventos y el SDK de Azure Event Hubs.
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: 8bf3f05b823a784f4f3fc2074719ed346f769f5e
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88933800"
---
# <a name="balance-partition-load-across-multiple-instances-of-your-application"></a>Equilibrio de carga de particiones entre varias instancias de una aplicación
Para escalar la aplicación de procesamiento de eventos, puede ejecutar varias instancias de la aplicación y equilibrar la carga entre ellas. En las versiones anteriores, [EventProcessorHost](event-hubs-event-processor-host.md) permitía equilibrar la carga entre varias instancias del programa y eventos de punto de comprobación en la recepción. En las versiones más recientes (5.0 y posteriores), **EventProcessorClient** (.NET y Java) o **EventHubConsumerClient** (Python y JavaScript) le permiten hacer lo mismo. El modelo de desarrollo se simplifica mediante el uso de eventos. Para suscribirse a los eventos que le interesen, registre un controlador de eventos.

En este artículo se describe un escenario de ejemplo para el uso de varias instancias para leer eventos desde un centro de eventos y, a continuación, se proporcionan detalles sobre las características del cliente del procesador de eventos, que permite recibir eventos de varias particiones a la vez y equilibrar la carga con otros consumidores que usan el mismo centro de eventos y el mismo grupo de consumidores.

> [!NOTE]
> La clave del escalado de Event Hubs es el concepto de consumidores con particiones. En contraposición al patrón de [consumidores de la competencia](/previous-versions/msp-n-p/dn568101(v=pandp.10)), el patrón de consumidores con particiones permite una alta escalabilidad mediante la eliminación de cuellos de botella de contención y la facilitación del paralelismo de principio a fin.

## <a name="example-scenario"></a>Escenario de ejemplo

Como caso de ejemplo, considere una empresa de seguridad en el hogar que supervisa 100 000 casas. Cada minuto, obtiene los datos de los diversos sensores como el detector de movimiento, el sensor de apertura de puertas y ventanas, el detector de rotura de cristales, etc., instalados en cada casa. La empresa proporciona un sitio web para que los residentes supervisen la actividad de su casa casi en tiempo real.

Cada sensor inserta datos en un centro de eventos. El centro de eventos está configurado con 16 particiones. En el extremo de consumo, necesita un mecanismo que pueda leer estos eventos, consolidarlos (filtrarlos, agregarlos, etc.) y volcar el agregado a un blob de almacenamiento que, a continuación, se proyecta en una página web fácil de usar.

## <a name="write-the-consumer-application"></a>Escritura de la aplicación de consumidor

Al diseñar el consumidor en un entorno distribuido, el escenario debe controlar los siguientes requisitos:

1. **Escalado:** cree varios consumidores y que cada consumidor tome posesión de la lectura desde varias particiones de Event Hubs.
2. **Equilibrio de carga:** aumente o reduzca dinámicamente los consumidores. Por ejemplo, si se agrega un nuevo tipo de sensor (por ejemplo, un detector de monóxido de carbono) a cada casa, aumenta el número de eventos. En ese caso, el operador (una persona) aumenta el número de instancias de consumidor. A continuación, el grupo de consumidores puede volver a equilibrar el número de particiones que poseen para compartir la carga con los consumidores recién agregados.
3. **Reanudación sin problemas después de los errores:** si un consumidor (**consumidor A**) genera un error (por ejemplo, la máquina virtual que hospeda al consumidor se bloquea de repente), otros consumidores pueden recopilar las particiones que posee el **consumidor A** y continuar. Además, el punto de continuación, llamado *punto de comprobación* o de *desplazamiento*, debe estar en el punto exacto en el que se produjo el error del **consumidor A** o ligeramente antes.
4. **Consumo de eventos:** mientras que los tres puntos anteriores trataban sobre la administración del consumidor, también tiene que haber código para consumir los eventos y hacer algo útil con él. Por ejemplo, agréguelo y cárguelo en el almacenamiento de blobs.

## <a name="event-processor-or-consumer-client"></a>Cliente de consumidor o procesador de eventos

No es necesario que cree su propia solución para cumplir estos requisitos. Los SDK de Azure Event Hubs proporcionan esta funcionalidad. En los SDK para .NET o Java, se usa un cliente de procesador de eventos (EventProcessorClient) y en los SDK para JavaScript y Python, se usa EventHubConsumerClient. En la versión anterior del SDK, era el host del procesador de eventos (EventProcessorHost) el que admitía estas características.

En la mayoría de los escenarios de producción, se recomienda usar el cliente del procesador de eventos para leer y procesar eventos. El cliente del procesador está diseñado para proporcionar una experiencia sólida para procesar eventos en todas las particiones de un centro de eventos de un modo eficaz y tolerante a errores, a la vez que permite comprobar el progreso. Los clientes del procesador de eventos también pueden trabajar de forma cooperativa en el contexto de un grupo de consumidores para un centro de eventos determinado. Los clientes administrarán de forma automática la distribución y el equilibrio del trabajo a medida que las instancias estén disponibles o no para el grupo.

## <a name="partition-ownership-tracking"></a>Seguimiento de la propiedad de una partición

Por lo general, una instancia de procesador de eventos posee y procesa los eventos de una o de varias particiones. La propiedad de las particiones se distribuye uniformemente entre todas las instancias del procesador de eventos activas asociadas a una combinación de centro de eventos y grupo de consumidores. 

Cada procesador de eventos recibe un identificador único y notifica la propiedad de las particiones mediante la adición o actualización de una entrada en un almacén de puntos de control. Todas las instancias del procesador de eventos se comunican con este almacén periódicamente para actualizar su propio estado de procesamiento, así como para obtener información sobre otras instancias activas. Después, estos datos se usan para equilibrar la carga entre los procesadores activos. Para escalar verticalmente, se pueden unir instancias nuevas al grupo de procesamiento. Cuando las instancias se quedan fuera de servicio, ya sea debido a errores o a una reducción vertical, la propiedad de la partición se transfiere correctamente a otros procesadores activos.

Los registros de propiedad de la partición en el almacén de puntos de control realizan un seguimiento del espacio de nombres de Event Hubs, nombre del centro de eventos, grupo de consumidores, identificador de procesador de eventos (también conocido como propietario), id. de partición y hora de la última modificación.



| Espacio de nombres de Event Hubs               | Nombre del centro de eventos | **Grupo de consumidores** | Propietario                                | Id. de partición | Hora de la última modificación  |
| ---------------------------------- | -------------- | :----------------- | :----------------------------------- | :----------- | :------------------ |
| mynamespace.servicebus.windows.net | myeventhub     | myconsumergroup    | 3be3f9d3-9d9e-4c50-9491-85ece8334ff6 | 0            | 2020-01-15T01:22:15 |
| mynamespace.servicebus.windows.net | myeventhub     | myconsumergroup    | f5cc5176-ce96-4bb4-bbaa-a0e3a9054ecf | 1            | 2020-01-15T01:22:17 |
| mynamespace.servicebus.windows.net | myeventhub     | myconsumergroup    | 72b980e9-2efc-4ca7-ab1b-ffd7bece8472 | 2            | 2020-01-15T01:22:10 |
|                                    |                | :                  |                                      |              |                     |
|                                    |                | :                  |                                      |              |                     |
| mynamespace.servicebus.windows.net | myeventhub     | myconsumergroup    | 844bd8fb-1f3a-4580-984d-6324f9e208af | 15           | 2020-01-15T01:22:00 |

Cada instancia del procesador de eventos adquiere la propiedad de una partición y empieza el procesamiento de la misma desde el último [punto de control](# Checkpointing) conocido. Si se produce un error en un procesador (la máquina virtual se apaga), otras instancias lo detectan al examinar la hora de la última modificación. Otras instancias intentan obtener la propiedad de las particiones que antes pertenecían a la instancia inactiva, y el almacén de puntos de control garantiza que solo una de las instancias tenga éxito al reclamar la propiedad de una partición. Por lo tanto, en un momento dado, hay, a lo sumo, un procesador que reciba eventos de una partición.

## <a name="receive-messages"></a>Recepción de mensajes

Cuando se crea un procesador de eventos, se especifican las funciones que van a procesar los eventos y los errores. Cada llamada a la función que procesa los eventos entrega un solo evento de una partición específica. Es su responsabilidad administrar este evento. Si desea asegurarse de que el consumidor procesa cada mensaje al menos una vez, debe escribir su propio código con lógica de reintento. Pero tenga cuidado con los mensajes dudosos.

Se recomienda que lo haga relativamente rápido. Es decir, use el menor procesamiento posible. Si tiene que escribir en el almacenamiento y llevar a cabo cierto enrutamiento, es mejor usar dos grupos de consumidores y tener dos procesadores de eventos.

## <a name="checkpointing"></a>Puntos de control

El *punto de control* es un proceso por el que un procesador de eventos marca o confirma la posición del último evento procesado correctamente en una partición. Por lo general, el marcado de un punto de control se realiza en la función que procesa los eventos y se produce en cada partición de un grupo de consumidores. 

Si un procesador de eventos se desconecta de una partición, otra instancia puede reanudar el procesamiento de la partición en el punto de control confirmado previamente por el último procesador de esa partición en ese grupo de consumidores. Cuando se conecta el procesador, pasa el desplazamiento al centro de eventos para especificar la ubicación en la que se va a empezar a leer. De este modo, puede usar puntos de control para que las aplicaciones de nivel inferior marquen los eventos como "completados" y para ofrecer resistencia cuando un procesador de eventos quede fuera de servicio. Es posible volver a los datos más antiguos especificando un desplazamiento inferior desde este proceso de puntos de comprobación. 

Cuando se realiza el punto de control para marcar un evento como procesado, se agrega o se actualiza una entrada en el almacén de puntos de control con el desplazamiento del evento y el número de secuencia. Los usuarios deben decidir la frecuencia de actualización del punto de control. La actualización después de cada evento procesado correctamente puede tener implicaciones en el rendimiento y el costo cuando desencadena una operación de escritura en el almacén de puntos de control subyacente. Además, el punto de control de cada evento único es indicativo de un patrón de mensajería en cola para el que una cola de Service Bus podría ser una opción mejor que un centro de eventos. La ventaja de Event Hubs es que obtiene al menos una entrega a gran escala. Al hacer los sistemas de nivel final idempotentes, es fácil recuperarse de errores o reinicios que hacen que los eventos se reciban múltiples veces.

> [!NOTE]
> Si usa Azure Blob Storage como el almacén de puntos de comprobación en un entorno que admite una versión diferente del SDK de blobs de almacenamiento que las que normalmente están disponibles en Azure, tendrá que utilizar código para cambiar la versión de la API del servicio Storage a la versión que admita ese entorno. Por ejemplo, si ejecuta [Event Hubs en una instancia de Azure Stack Hub versión 2002](/azure-stack/user/event-hubs-overview), la versión más alta disponible para el servicio Storage es 2017-11-09. En este caso, tendrá que usar código para establecer como destino la versión de la API del servicio Storage en 2017-11-09. Para obtener un ejemplo de cómo establecer como destino una versión específica de la API de Storage, vea estos ejemplos en GitHub: 
> - [.NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Azure.Messaging.EventHubs.Processor/samples/Sample10_RunningWithDifferentStorageVersion.cs) 
> - [Java](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs-checkpointstore-blob/src/samples/java/com/azure/messaging/eventhubs/checkpointstore/blob/)
> - [JavaScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/javascript) o [TypeScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/typescript)
> - [Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub-checkpointstoreblob-aio/samples/)

## <a name="thread-safety-and-processor-instances"></a>Seguridad para subprocesos e instancias de procesador

De forma predeterminada, se llama de forma secuencial a la función que procesa los eventos para una partición determinada. Los siguientes eventos y llamadas a esta función desde la misma partición se ponen en cola en segundo plano mientras el suministro de eventos continúa ejecutándose en segundo plano en otros subprocesos. Tenga en cuenta que los eventos de diferentes particiones se pueden procesar simultáneamente y cualquier estado compartido al que se tenga acceso desde varias particiones se debe sincronizar.

## <a name="next-steps"></a>Pasos siguientes
Consulte las siguientes guías de inicio rápido:

- [.NET Core](event-hubs-dotnet-standard-getstarted-send.md)
- [Java](event-hubs-java-get-started-send.md)
- [Python](event-hubs-python-get-started-send.md)
- [JavaScript](event-hubs-node-get-started-send.md)
