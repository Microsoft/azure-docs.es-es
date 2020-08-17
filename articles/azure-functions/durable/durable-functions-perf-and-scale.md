---
title: 'Rendimiento y escalado horizontal en Durable Functions: Azure'
description: Introducción a la extensión Durable Functions de Azure Functions.
author: cgillum
ms.topic: conceptual
ms.date: 11/03/2019
ms.author: azfuncdf
ms.openlocfilehash: 58c28160de15bc99c94c84ab23fdbb358125132d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87033588"
---
# <a name="performance-and-scale-in-durable-functions-azure-functions"></a>Rendimiento y escalado horizontal en Durable Functions (Azure Functions)

Para optimizar el rendimiento y la escalabilidad, es importante comprender las características únicas de escalado de [Durable Functions](durable-functions-overview.md).

Para entender el comportamiento de escalado, tendrá que comprender algunos de los detalles del proveedor de Azure Storage subyacente.

## <a name="history-table"></a>Tabla del historial

La tabla **History** es una tabla de Azure Storage que contiene los eventos del historial de todas las instancias de orquestación dentro de una central de tareas. El nombre de esta tabla está en formato *TaskHubName*History. Según se ejecutan las instancias, se van agregando nuevas filas a esta tabla. La clave de partición de esta tabla proviene del identificador de instancia de la orquestación. El identificador de instancia es aleatorio en la mayoría de los casos, lo que garantiza una distribución óptima de las particiones internas en Azure Storage.

Cuando sea necesario ejecutar una instancia de orquestación, las filas correspondientes de la tabla History se cargan en memoria. Estos *eventos del historial* se reproducen luego en el código de función de orquestación para regresar a su estado de punto de control previo. El uso del historial de ejecución para reconstruir el estado de esta manera se ve influenciado por el [patrón Event Sourcing](/azure/architecture/patterns/event-sourcing).

## <a name="instances-table"></a>Tabla Instances

La tabla **Instances** es otra tabla de Azure Storage que contiene los estados de todas las instancias de orquestación y entidad dentro de una central de tareas. Según se crean las instancias, se van agregando nuevas filas a esta tabla. La clave de partición de esta tabla es el identificador de instancia de orquestación o clave de entidad, y la clave de fila es una constante fija. Hay una fila por cada instancia de orquestación o entidad.

Esta tabla se usa para satisfacer las solicitudes de consulta de instancias de las API `GetStatusAsync` (.NET) y `getStatus` (JavaScript), así como de la [API HTTP de consulta de estado](durable-functions-http-api.md#get-instance-status). Se mantiene coherente en última instancia con el contenido de la tabla **History** mencionada anteriormente. El uso de una tabla de Azure Storage independiente para satisfacer con eficiencia operaciones de consulta de instancias de esta manera se ve influenciada por el [patrón Command and Query Responsibility Segregation (CQRS)](/azure/architecture/patterns/cqrs).

## <a name="internal-queue-triggers"></a>Desencadenadores de cola interna

Las funciones de orquestador y de actividad se desencadenan mediante colas internas de la central de tareas de la aplicación de función. El uso de colas de este modo proporciona garantías de entrega confiable de mensajes "al menos una vez". Hay dos tipos de colas en Durable Functions: **de control** y **de elementos de trabajo**.

### <a name="the-work-item-queue"></a>Cola de elementos de trabajo

En Durable Functions hay una cola de elementos de trabajo por central de tareas. Se trata de una cola básica y se comporta del mismo modo que cualquier otra cola `queueTrigger` en Azure Functions. Esta cola se usa para desencadenar *funciones de actividad* sin estado al eliminar de la cola un único mensaje a la vez. Cada uno de estos mensajes contiene entradas de la función de actividad y metadatos adicionales, como qué función se debe ejecutar. Cuando una aplicación de Durable Functions admita el escalado horizontal de varias máquinas virtuales, estas compiten para adquirir el trabajo de la cola de elementos de trabajo.

### <a name="control-queues"></a>Colas de control

Hay varias *colas de control* por central de tareas en Durable Functions. Una *cola de control* es más sofisticada que una cola de elementos de trabajo, más sencilla. Las colas de control se usan para desencadenar las funciones de orquestador y entidad con estado. Dado que las instancias de las funciones de orquestador y entidad son singletons con estado, no es posible usar un modelo de consumidor de competencia para distribuir la carga entre máquinas virtuales. En su lugar, se equilibra la carga de los mensajes del orquestador y la entidad entre las colas de control. Encontrará más detalles sobre este comportamiento en las secciones siguientes.

Las colas de control contienen una gran variedad de tipos de mensajes del ciclo de vida de la orquestación. Algunos ejemplos son los [mensajes de control del orquestador](durable-functions-instance-management.md), los mensajes de *respuesta* de la función de actividad y los del temporizador. Como máximo, se pueden eliminar 32 mensajes de una cola de control en un único sondeo. Estos mensajes contienen datos de carga, así como metadatos, incluido para qué instancia de orquestación están previstos. Si hay varios mensajes eliminados de la cola previstos para la misma instancia de orquestación, se procesarán como lote.

### <a name="queue-polling"></a>Sondeo de cola

La extensión Durable Task implementa un algoritmo de interrupción exponencial aleatorio para reducir el efecto del sondeo de cola inactiva en los costos de transacción de almacenamiento. Cuando se encuentra un mensaje, el tiempo de ejecución comprueba si hay otro mensaje; cuando no se encuentra ningún mensaje, espera un tiempo antes de intentarlo de nuevo. Después de varios intentos fallidos para obtener un mensaje de la cola, el tiempo de espera sigue aumentando hasta que alcanza el tiempo de espera máximo, predeterminado en 30 segundos.

El retraso de sondeo máximo se configura mediante la propiedad `maxQueuePollingInterval` en el [archivo host.json](../functions-host-json.md#durabletask). El establecimiento de esta propiedad en un valor más alto puede provocar mayores latencias de procesamiento de mensajes. Las latencias elevadas solo se pueden esperar después de períodos de inactividad. El establecimiento de esta propiedad en un valor más bajo puede provocar mayores costos de almacenamiento debido al mayor número de transacciones de almacenamiento.

> [!NOTE]
> Cuando se ejecuta en los planes Consumo y Premium de Azure Functions, el [controlador de escala de Azure Functions](../functions-scale.md#how-the-consumption-and-premium-plans-work) sondeará cada cola de control y elemento de trabajo una vez cada 10 segundos. Este sondeo adicional es necesario para determinar cuándo activar instancias de aplicaciones de función y tomar decisiones de escalado. En el momento de escribir este artículo, este intervalo de 10 segundos es constante y no se puede configurar.

### <a name="orchestration-start-delays"></a>Retrasos en el inicio de la orquestación
Las instancias de las orquestaciones se inician con la inserción de un mensaje `ExecutionStarted` en una de las colas de control de la central de tareas. En determinadas condiciones, se pueden observar retrasos de varios segundos entre el momento en que se programa la ejecución de una orquestación y el momento en el que se inicia la ejecución. Durante este intervalo de tiempo, la instancia de la orquestación permanece en estado `Pending`. Hay dos causas posibles de este retraso:

1. **Colas de control pendientes**: si la cola de control de esta instancia contiene un gran número de mensajes, puede tardar un tiempo antes de que el entorno de ejecución reciba y procese el mensaje `ExecutionStarted`. Los trabajos pendientes de mensajes se pueden producir cuando las orquestaciones procesan muchos eventos simultáneamente. Entre los eventos que van a la cola de control se incluyen los eventos de inicio de orquestación, finalización de actividad, temporizadores duraderos, terminación y eventos externos. Si este retraso se produce en circunstancias normales, considere la posibilidad de crear una nueva central de tareas con un mayor número de particiones. La configuración de más particiones hará que el entorno de ejecución cree más colas de control para la distribución de la carga.

2. **Retrasos en los sondeos de retroceso**: otra causa común de los retrasos de la orquestación es el [comportamiento de los sondeos de retroceso que se ha descrito anteriormente para las colas de control](#queue-polling). Sin embargo, este retraso solo se espera cuando una aplicación se escala horizontalmente a dos o más instancias. Si solo hay una instancia de la aplicación o si la instancia de la aplicación que inicia la orquestación también es la misma instancia que sondea la cola de control de destino, no habrá un retraso en el sondeo de la cola. Los retrasos de los sondeos de retroceso se pueden reducir mediante la actualización de las opciones del archivo **host.json**, tal y como se ha descrito anteriormente.

## <a name="storage-account-selection"></a>Selección de una cuenta de almacenamiento

Las colas, tablas y blobs usados por Durable Functions se crean en una cuenta de Azure Storage configurada. Se puede especificar la cuenta que se va a usar mediante el valor `durableTask/storageProvider/connectionStringName` (o `durableTask/azureStorageConnectionStringName` en Durable Functions 1.x) del archivo **host.json**.

### <a name="durable-functions-2x"></a>Durable Functions 2.x

```json
{
  "extensions": {
    "durableTask": {
      "storageProvider": {
        "connectionStringName": "MyStorageAccountAppSetting"
      }
    }
  }
}
```

### <a name="durable-functions-1x"></a>Durable Functions 1.x

```json
{
  "extensions": {
    "durableTask": {
      "azureStorageConnectionStringName": "MyStorageAccountAppSetting"
    }
  }
}
```

Si no se especifica, se utiliza la cuenta de almacenamiento `AzureWebJobsStorage` predeterminada. Para las cargas de trabajo sensibles al rendimiento, se recomienda, sin embargo, configurar una cuenta de almacenamiento que no sea la predeterminada. Durable Functions utiliza Azure Storage de forma intensiva y el uso de una cuenta de almacenamiento dedicada aísla el uso del almacenamiento de Durable Functions del uso interno por parte del host de Azure Functions.

## <a name="orchestrator-scale-out"></a>Escalabilidad horizontal del orquestador

Las funciones de actividad no tienen estado y se escalan horizontalmente de manera automática con la incorporación de máquinas virtuales. En cambio, las funciones de orquestador y las entidades tienen *particiones* en una o más colas de control. El número de colas de control se define en el archivo **host.json**. El siguiente fragmento de código de host.json de ejemplo establece la propiedad `durableTask/storageProvider/partitionCount` (o `durableTask/partitionCount` en Durable Functions 1.x) en `3`.

### <a name="durable-functions-2x"></a>Durable Functions 2.x

```json
{
  "extensions": {
    "durableTask": {
      "storageProvider": {
          "partitionCount": 3
      }
    }
  }
}
```

### <a name="durable-functions-1x"></a>Durable Functions 1.x

```json
{
  "extensions": {
    "durableTask": {
      "partitionCount": 3
    }
  }
}
```

Una central de tareas se puede configurar con entre 1 y 16 particiones. Si no se especifica, el número de participaciones predeterminado es **4**.

Mientras se escala horizontalmente a varias instancias de host de función (normalmente en máquinas virtuales diferentes), cada instancia adquiere un bloqueo en una de las colas de control. Estos bloqueos se implementan internamente como concesiones de almacenamiento de blobs y aseguran que solo se ejecuta una instancia de orquestación o entidad en una única instancia de host a la vez. Si una central de tareas está configurada con tres colas de control, se puede equilibrar la carga de las instancias de orquestación y entidades en tres máquinas virtuales como máximo. Se pueden agregar máquinas virtuales adicionales para aumentar la capacidad de ejecución de la función de actividad,

El siguiente diagrama ilustra cómo interactúa el host de Azure Functions con las entidades de almacenamiento en un entorno de escalado horizontal.

![Diagrama de escalado](./media/durable-functions-perf-and-scale/scale-diagram.png)

Tal como se muestra en el diagrama anterior, todas las máquinas virtuales compiten por los mensajes en la cola de elementos de trabajo. Sin embargo, solo tres de ellas pueden adquirir mensajes de las colas de control y cada una bloquea una sola cola de control.

Las instancias de orquestación y entidades se distribuyen entre todas las instancias de la cola de control. La distribución se realiza al aplicar un algoritmo hash al identificador de instancia de la orquestación o el par de nombre y clave de la entidad. Los identificadores de instancia de orquestación son, de forma predeterminada, GUID aleatorios que aseguran que las instancias se distribuyen equitativamente entre todas las colas de control.

En general, las funciones de orquestador se han diseñado para que sean ligeras y no necesiten grandes capacidades de computación. Por tanto, no es necesario crear un gran número de particiones de cola de control para obtener un rendimiento excelente para las orquestaciones. La mayor parte del trabajo pesado se realiza en las funciones de actividad sin estado, que se pueden escalar en horizontal infinitamente.

## <a name="auto-scale"></a>Escalado automático

Al igual que todas las instancias de Azure Functions que se ejecutan en los planes Consumo y Elástico Premium, Durable Functions admite el escalado automático a través del [controlador de escalado de Azure Functions](../functions-scale.md#runtime-scaling). El controlador de escalado supervisa la latencia de todas las colas emitiendo periódicamente comandos _peek_. En función de las latencias de los mensajes inspeccionados, el controlador de escalado decidirá si desea agregar o quitar máquinas virtuales.

Si el controlador de escalado determina que las latencias de mensaje de la cola de control son demasiado altas, agregará instancias de máquina virtual hasta que la latencia de mensajes se reduzca a un nivel aceptable o alcance el número de particiones de la cola de control. De igual forma, el controlador de escalado va a agregar continuamente instancias de máquina virtual si las latencias de la cola de elementos de trabajo son altas, independientemente del número de particiones.

> [!NOTE]
> A partir de Durable Functions 2.0, las aplicaciones de función se pueden configurar de modo que se ejecuten en los puntos de conexión de servicio protegidos mediante red virtual en el plan Elástico Premium. En esta configuración, los desencadenadores de Durable Functions inician solicitudes de escalado en lugar del controlador de escalado.

## <a name="thread-usage"></a>Uso de subprocesos

Las funciones de orquestador se ejecutan en un solo subproceso para asegurarse de que la ejecución pueda ser determinista entre muchas de las reproducciones. Debido a esta ejecución en un único subproceso, es importante que los subprocesos de las funciones de orquestador no realicen tareas de uso intensivo de CPU, ni operaciones de E/S ni bloqueos por ningún motivo. Cualquier trabajo que requiera operaciones de E/S, bloqueos o varios subprocesos debe derivarse a las funciones de actividad.

Las funciones de actividad tienen el mismo comportamiento que las funciones normales desencadenadas por colas. Pueden realizar operaciones de E/S, ejecutar operaciones de uso intensivo de CPU y utilizar varios subprocesos con seguridad. Dado que los desencadenadores de actividad no tienen estado, se escalan horizontalmente a un número ilimitado de máquinas virtuales sin problema.

Las funciones de entidad también se ejecutan en un único subproceso y las operaciones se procesan de una en una. Pero las funciones de entidad no tienen ninguna restricción en cuanto al tipo de código que se puede ejecutar.

## <a name="concurrency-throttles"></a>Limitaciones de simultaneidad

Azure Functions admite la ejecución de varias funciones simultáneamente dentro de una instancia de aplicación única. Esta ejecución simultánea ayuda a aumentar el paralelismo y minimiza el número de "arranques en frío" que experimentará una aplicación típica a lo largo del tiempo. Pero la alta simultaneidad puede agotar los recursos del sistema por máquina virtual, como las conexiones de red o la memoria disponible. Según las necesidades de la aplicación de función, puede ser necesario limitar la simultaneidad por instancia para evitar la posibilidad de quedarse sin memoria en situaciones de carga alta.

Los límites de simultaneidad de las funciones de actividad, orquestador y entidad se pueden configurar en el archivo **host.json**. La configuración relevante es `durableTask/maxConcurrentActivityFunctions` para las funciones de actividad y `durableTask/maxConcurrentOrchestratorFunctions` para las funciones de orquestador y de entidad.

### <a name="functions-20"></a>Functions 2.0

```json
{
  "extensions": {
    "durableTask": {
      "maxConcurrentActivityFunctions": 10,
      "maxConcurrentOrchestratorFunctions": 10
    }
  }
}
```

### <a name="functions-1x"></a>Functions 1.x

```json
{
  "durableTask": {
    "maxConcurrentActivityFunctions": 10,
    "maxConcurrentOrchestratorFunctions": 10
  }
}
```

En el ejemplo anterior, se pueden ejecutar simultáneamente un máximo de 10 funciones de orquestador o entidad. y 10 funciones de actividad en una única máquina virtual. Si no se especifica, el número de ejecuciones simultáneas de funciones de actividad y orquestador o entidad está limitado a 10 veces el número de núcleos de la máquina virtual.

> [!NOTE]
> Esta configuración resulta útil para ayudar a administrar el uso de memoria y de CPU en una única máquina virtual. Pero al escalar horizontalmente en varias máquinas virtuales, cada una tiene su propio conjunto de límites. Esta configuración no se puede usar para controlar la simultaneidad a nivel global.

## <a name="extended-sessions"></a>Sesiones extendidas

Sesiones extendidas es un valor de configuración que mantiene las orquestaciones y entidades en memoria incluso después de que terminen de procesar mensajes. El efecto típico de habilitar sesiones extendidas es un número de operaciones de E/S reducido en lo que respecta a la cuenta de Azure Storage y un rendimiento general mejorado.

Puede habilitar sesiones extendidas si establece `durableTask/extendedSessionsEnabled` en `true` en el archivo **host.json**. El valor `durableTask/extendedSessionIdleTimeoutInSeconds` se puede usar para controlar cuánto tiempo se retiene una sesión inactiva en memoria:

**Functions 2.0**
```json
{
  "extensions": {
    "durableTask": {
      "extendedSessionsEnabled": true,
      "extendedSessionIdleTimeoutInSeconds": 30
    }
  }
}
```

**Functions 1.0**
```json
{
  "durableTask": {
    "extendedSessionsEnabled": true,
    "extendedSessionIdleTimeoutInSeconds": 30
  }
}
```

Hay dos posibles inconvenientes de este valor que se deben tener en cuenta:

1. Hay un aumento general del uso de memoria de la aplicación de función.
2. Puede haber una disminución general del rendimiento si hay varias ejecuciones de funciones de orquestador o entidad simultáneas y de corta duración.

Por ejemplo, si `durableTask/extendedSessionIdleTimeoutInSeconds` se establece en 30 segundos, un episodio de función de orquestador o entidad de corta duración que se ejecuta en menos de 1 segundo sigue ocupando memoria durante 30 segundos. También se descuenta de la cuota `durableTask/maxConcurrentOrchestratorFunctions` mencionada antes, lo que puede evitar que se ejecuten otras funciones de orquestador o entidad.

En las secciones siguientes se describen los efectos concretos de las sesiones extendidas en las funciones de orquestador y entidad.

### <a name="orchestrator-function-replay"></a>Reproducción de una función de orquestador

Tal como se ha mencionado anteriormente, las funciones de orquestador se reproducen con el contenido de la tabla **History**. De forma predeterminada, el código de la función de orquestador se reproduce cada vez que se elimina un lote de mensajes de una cola de control. Incluso si usa el patrón de distribución ramificada de entrada y salida y está esperando a que se completen todas las tareas (por ejemplo, con `Task.WhenAll` en .NET o `context.df.Task.all` en JavaScript), habrá repeticiones que se produzcan a medida que se procesen lotes de respuestas de tareas a lo largo del tiempo. Cuando se habilitan sesiones extendidas, las instancias de funciones de orquestador se mantienen en memoria más tiempo y se pueden procesar mensajes nuevos sin una reproducción de historial completa.

La mejora del rendimiento de las sesiones extendidas suele notarse en las siguientes situaciones:

* Cuando hay un número limitado de instancias de orquestación que se ejecutan simultáneamente.
* Cuando las orquestaciones tienen un gran número de acciones secuenciales (por ejemplo, cientos de llamadas a funciones de actividad) que se completan rápidamente.
* Cuando las orquestaciones se ramifican en un gran número de acciones que se completan aproximadamente al mismo tiempo.
* Cuando las funciones de orquestador necesitan procesar mensajes grandes o realizar un procesamiento de datos con un uso intensivo de la CPU.

En todas las demás situaciones no suele notarse ninguna mejora del rendimiento para las funciones de orquestador.

> [!NOTE]
> Esta configuración solo se debe usar una vez que haya sido desarrollada y probada totalmente una función de orquestador. El comportamiento de reproducción agresivo predeterminado puede ser útil para detectar infracciones de [restricciones de código de función de orquestador](durable-functions-code-constraints.md) en tiempo de desarrollo y, por tanto, está deshabilitado de forma predeterminada.

### <a name="entity-function-unloading"></a>Descarga de funciones de entidad

Las funciones de entidad procesan hasta 20 operaciones en un único lote. En cuanto una entidad termina de procesar un lote de operaciones, conserva su estado y se descarga de la memoria. Puede retrasar la descarga de entidades de la memoria mediante la configuración de sesiones extendidas. Las entidades continúan conservando sus cambios de estado como antes, pero permanecen en memoria durante el período de tiempo configurado para reducir el número de cargas desde Azure Storage. Esta reducción de cargas desde Azure Storage puede mejorar el rendimiento general de las entidades a las que se accede con frecuencia.

## <a name="performance-targets"></a>Destinos del rendimiento

Si prevé utilizar Durable Functions para una aplicación de producción, es importante tener en cuenta los requisitos de rendimiento en una fase temprana del proceso de planeamiento. En esta sección se describen algunos escenarios de uso básico y las cifras de rendimiento máximo que se esperan.

* **Ejecución de actividad secuencial**: este escenario describe una función de orquestador que ejecuta una serie de funciones de actividad una tras otra. Se parece en gran medida al ejemplo de [encadenamiento de funciones](durable-functions-sequence.md).
* **Ejecución de actividad en paralelo**: este escenario describe una función de orquestador que ejecuta numerosas funciones de actividad en paralelo mediante el patrón de [distribución ramificada de entrada/salida](durable-functions-cloud-backup.md).
* **Procesamiento de respuestas en paralelo**: este escenario es la segunda mitad del patrón de [distribución ramificada de entrada/salida](durable-functions-cloud-backup.md). Se centra en el rendimiento de la distribución ramificada de entrada. Es importante tener en cuenta que, a diferencia de la distribución ramificada de salida, la distribución ramificada de entrada se realiza mediante una única instancia de la función de orquestador y, por lo tanto, solo se puede ejecutar en una sola máquina virtual.
* **Procesamiento de eventos externos**: este escenario representa una única instancia de la función de orquestador que espera en [eventos externos](durable-functions-external-events.md), uno a uno.
* **Procesamiento de operaciones de entidad**: En este escenario se prueba la rapidez con la que una _sola_[entidad Contador](durable-functions-entities.md) puede procesar un flujo constante de operaciones.

> [!TIP]
> A diferencia de la distribución ramificada de salida, las operaciones de la distribución ramificada de entrada están limitadas a una sola máquina virtual. Si la aplicación utiliza el patrón de distribución ramificada de entrada/salida, y le preocupa el rendimiento de la distribución ramificada de entrada, considere la posibilidad de subdividir la distribución ramificada de salida de la función de actividad entre varias [suborquestaciones](durable-functions-sub-orchestrations.md).

En la tabla siguiente se muestran las cifras previstas del rendimiento *máximo* para los escenarios descritos anteriormente. "Instancia" hace referencia a una sola instancia de una función de orquestador que se ejecuta en una única y pequeña máquina virtual ([A1](../../virtual-machines/sizes-previous-gen.md)) de Azure App Service. En todos los casos, se supone que las [sesiones extendidas](#orchestrator-function-replay) están habilitadas. Los resultados reales pueden variar según el trabajo de CPU o de E/S realizado por el código de la función.

| Escenario | Rendimiento máximo |
|-|-|
| Ejecución de actividad secuencial | 5 actividades por segundo y por instancia |
| Ejecución de actividad en paralelo (distribución ramificada de salida) | 100 actividades por segundo y por instancia |
| Procesamiento de respuestas en paralelo (distribución ramificada de entrada) | 150 respuestas por segundo y por instancia |
| Procesamiento de eventos externos | 50 eventos por segundo y por instancia |
| Procesamiento de operaciones de entidad | 64 operaciones por segundo |

> [!NOTE]
> Estos números están actualizados a partir de la versión v1.4.0 (GA) de la extensión de Durable Functions. Estos números pueden cambiar a lo largo del tiempo, ya que la característica evoluciona y se realizan optimizaciones.

Si no ve las cifras de rendimiento que espera, y el uso de CPU y de memoria parece correcto, compruebe si la causa está relacionada con el [mantenimiento de la cuenta de almacenamiento](../../storage/common/storage-monitoring-diagnosing-troubleshooting.md#troubleshooting-guidance). La extensión de Durable Functions puede colocar una carga importante en una cuenta de Azure Storage y unas cargas suficientemente altas pueden provocar limitaciones en la cuenta de almacenamiento.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Información sobre la recuperación ante desastres y distribución geográfica](durable-functions-disaster-recovery-geo-distribution.md)
