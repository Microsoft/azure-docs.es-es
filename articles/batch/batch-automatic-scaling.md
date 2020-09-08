---
title: Escalar automáticamente los nodos de proceso en un grupo de Azure Batch
description: Habilite el escalado automático en un grupo en la nube para ajustar de forma dinámica el número de nodos de ejecución del grupo.
ms.topic: how-to
ms.date: 07/27/2020
ms.custom: H1Hack27Feb2017, fasttrack-edit, devx-track-csharp
ms.openlocfilehash: e3e7a354e015ffa8a6164de59edcf572ab773319
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88932328"
---
# <a name="create-an-automatic-formula-for-scaling-compute-nodes-in-a-batch-pool"></a>Creación de una fórmula automática para escalar nodos de ejecución en un grupo de Batch

Azure Batch puede escalar grupos automáticamente en función de los parámetros que defina, lo que le permite ahorrar tiempo y dinero. Con el escalado automático, Batch agrega nodos dinámicamente a un grupo a medida que aumenta la demanda de tareas y quita nodos de ejecución a medida que disminuye.

Para habilitar el escalado automático en un grupo de nodos de ejecución, asocie el grupo con una *fórmula de escalabilidad automática* que defina. El servicio Batch usa la fórmula de escalabilidad automática para determinar cuántos nodos se necesitan para ejecutar la carga de trabajo. Estos nodos pueden ser nodos dedicados o [nodos de prioridad baja](batch-low-pri-vms.md). Batch revisará periódicamente los datos de las métricas de servicio y los usará para ajustar el número de nodos del grupo en función de la fórmula y en un intervalo definido por el usuario.

Puede habilitar el escalado automático al crear un grupo o aplicarlo a un grupo existente. Batch le permite evaluar las fórmulas antes de asignarlas a grupos y supervisar el estado de las ejecuciones de escalado automático. Una vez que configure un grupo con el escalado automático, más adelante puede hacer cambios en la fórmula.

> [!IMPORTANT]
> Al crear una cuenta de Batch puede especificar el [modo de asignación de grupos](accounts.md), lo que determina si se asignan grupos en una suscripción de servicio Batch (opción predeterminada) o en su suscripción de usuario. Si ha creado su cuenta de Batch con la configuración predeterminada del servicio Batch, la cuenta estará limitada a un máximo de núcleos que se pueden usar para el procesamiento. El servicio de Batch escala nodos de ejecución solo hasta ese límite de núcleos. Por este motivo, puede que el servicio de Batch no alcance el número de nodos de ejecución que se especifiquen con una fórmula de escalado automático. Consulte [Cuotas y límites del servicio Azure Batch](batch-quota-limit.md) para más información sobre la visualización y aumento de las cuotas de la cuenta.
>
>Si ha creado su cuenta con el modo de suscripción de usuario, su cuenta comparte la cuota de núcleos de la suscripción. Para más información, consulte [Límites de Virtual Machines](../azure-resource-manager/management/azure-subscription-service-limits.md#virtual-machines-limits) en [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../azure-resource-manager/management/azure-subscription-service-limits.md).

## <a name="autoscale-formulas"></a>Fórmulas de escalabilidad automática

Una fórmula de escalabilidad automática es un valor de cadena que el usuario define y contiene una o varias instrucciones. La fórmula de escalado automático se asigna a un elemento [autoScaleFormula](/rest/api/batchservice/enable-automatic-scaling-on-a-pool) de un grupo (Batch REST) o a una propiedad [CloudPool.AutoScaleFormula](/dotnet/api/microsoft.azure.batch.cloudpool.autoscaleformula) (Batch .NET). El servicio Batch usa la fórmula para determinar el número de nodos de ejecución del grupo durante el intervalo de procesamiento siguiente. La cadena de fórmula no puede superar los 8 KB y puede incluir hasta 100 instrucciones separadas por punto y coma, así como saltos de línea y comentarios.

Puede imaginarse que las fórmulas de escalado automático son un "lenguaje" de escalado automático de Batch. Las instrucciones de fórmula son expresiones de forma libre que pueden incluir variables definidas por el servicio (definidas por el servicio Batch) y variables definidas por el usuario. Las fórmulas pueden realizar diversas operaciones en estos valores mediante funciones, operadores y tipos integrados. Por ejemplo, una instrucción podría tener la forma siguiente:

```
$myNewVariable = function($ServiceDefinedVariable, $myCustomVariable);
```

Las fórmulas suelen tener varias instrucciones que realizan operaciones en los valores obtenidos en las instrucciones anteriores. Por ejemplo, en primer lugar, se obtiene un valor para `variable1`, a continuación, se pasa a una función para rellenar `variable2`:

```
$variable1 = function1($ServiceDefinedVariable);
$variable2 = function2($OtherServiceDefinedVariable, $variable1);
```

Incluya estas instrucciones en la fórmula de escalado automático para llegar a un determinado número de nodos de ejecución. Los nodos dedicados y los nodos de prioridad baja tienen su propia configuración de destino. Una fórmula de escalado automático puede incluir un valor de destino para los nodos dedicados, un valor de destino para los nodos de prioridad baja o ambos.

El número objetivo de nodos puede ser mayor, menor o igual que el número actual de nodos de ese tipo que hay en el grupo. Batch evalúa la fórmula de escalabilidad automática de un grupo según un [intervalo de escalado automático](#automatic-scaling-interval) específico. Batch ajusta el número de destino de cada tipo de nodo del grupo al número que especifica la fórmula de escalado automático en el momento de la evaluación.

### <a name="sample-autoscale-formulas"></a>Fórmulas de escalado automático de ejemplo

A continuación, se presentan ejemplos de dos fórmulas de escalabilidad automática, que se pueden ajustar para trabajar en la mayoría de los escenarios. Las variables `startingNumberOfVMs` y `maxNumberofVMs` de las fórmulas de ejemplo se pueden ajustar a sus necesidades.

#### <a name="pending-tasks"></a>Tareas pendientes

Con esta fórmula de escalado automático, el grupo se crea inicialmente con una sola máquina virtual. La métrica `$PendingTasks` define el número de tareas que están en ejecución o en cola. La fórmula busca el número promedio de tareas pendientes en los últimos 180 segundos y establece la variable `$TargetDedicatedNodes` en consecuencia. La fórmula garantiza que el número de destino de nodos dedicados nunca supere la cantidad de 25 máquinas virtuales. El grupo crece automáticamente a medida que se envían nuevas tareas. Cuando las tareas finalizan, las máquinas virtuales quedan libres y la fórmula de escalabilidad automática reduce el grupo.

Esta fórmula escala los nodos dedicados, pero se puede modificar para escalar también los nodos de prioridad baja.

```
startingNumberOfVMs = 1;
maxNumberofVMs = 25;
pendingTaskSamplePercent = $PendingTasks.GetSamplePercent(180 * TimeInterval_Second);
pendingTaskSamples = pendingTaskSamplePercent < 70 ? startingNumberOfVMs : avg($PendingTasks.GetSample(180 * TimeInterval_Second));
$TargetDedicatedNodes=min(maxNumberofVMs, pendingTaskSamples);
$NodeDeallocationOption = taskcompletion;
```

#### <a name="preempted-nodes"></a>Nodos con prioridad

En este ejemplo se crea un grupo que empieza por 25 nodos de prioridad baja. Cada vez que se adelanta un nodo de prioridad baja, se reemplaza por un nodo dedicado. Como en el primer ejemplo, la variable `maxNumberofVMs` impide que el grupo supere 25 máquinas virtuales. Este ejemplo es útil para sacar provecho de las máquinas virtuales de prioridad baja y, al mismo tiempo, garantizar que solo se produzca un número fijo de retrasos en la duración del grupo.

```
maxNumberofVMs = 25;
$TargetDedicatedNodes = min(maxNumberofVMs, $PreemptedNodeCount.GetSample(180 * TimeInterval_Second));
$TargetLowPriorityNodes = min(maxNumberofVMs , maxNumberofVMs - $TargetDedicatedNodes);
$NodeDeallocationOption = taskcompletion;
```

Aprenderá más sobre [cómo crear fórmulas de escalabilidad automática](#write-an-autoscale-formula) y verá otras [fórmulas de escalabilidad automática de ejemplo](#example-autoscale-formulas) más adelante en este tema.

## <a name="variables"></a>variables

En las fórmulas de escalado automático puede usar tanto variables **definidas por el servicio** como variables **definidas por el usuario**.

Las variables definidas por el servicio están integradas en el servicio de Batch. Algunas variables definidas por el servicio son de lectura y escritura, mientras que otras son de solo lectura.

Las variables definidas por el usuario son las que usted define. En la fórmula de ejemplo mostrada anteriormente, `$TargetDedicatedNodes` y `$PendingTasks` son variables definidas por el servicio, mientras que `startingNumberOfVMs` y `maxNumberofVMs` son variables definidas por el usuario.

> [!NOTE]
> Las variables definidas por el servicio van siempre precedidas de un símbolo de dólar ($). En el caso de las variables definidas por el usuario, el símbolo de dólar es opcional.

En las tablas siguientes se muestran las variables de lectura y escritura y de solo lectura definidas por el servicio Batch.

### <a name="read-write-service-defined-variables"></a>Variables definidas por el servicio de solo escritura

Puede obtener y establecer los valores de estas variables definidas por el servicio para administrar el número de nodos de ejecución de un grupo.

| Variable | Descripción |
| --- | --- |
| $TargetDedicatedNodes |Número objetivo de nodos de ejecución dedicados para el grupo. Esto se especifica como destino porque un grupo podría no lograr siempre el número de nodos deseado. Por ejemplo, podría pasar que el grupo no alcanzara el número de destino si el número de nodos dedicados se modifica con una evaluación de escalado automático antes de que el grupo haya alcanzado el valor de destino inicial. <br /><br /> Podría pasar que un grupo de una cuenta creada en el modo del servicio Batch no alcanzara su valor de destino si el destino supera una cuota de núcleos o de nodos de la cuenta de Batch. Podría pasar que un grupo de una cuenta creada en el modo de suscripción de usuario no alcanzara su valor de destino si el destino supera la cuota de núcleos compartidos de la suscripción.|
| $TargetLowPriorityNodes |Número objetivo de nodos de ejecución de prioridad baja para el grupo. Esto se especifica como destino porque un grupo podría no lograr siempre el número de nodos deseado. Por ejemplo, podría pasar que el grupo no alcanzara el número de destino si el número de nodos de prioridad baja se modifica con una evaluación de escalado automático antes de que el grupo haya alcanzado el valor de destino inicial. Además, también podría pasar que un grupo no alcanzara el valor de destino si el destino supera una cuota de núcleos o de nodos de la cuenta de Batch. <br /><br /> Para obtener más información sobre los nodos de ejecución de prioridad baja, consulte [Uso de máquinas virtuales de prioridad baja con Batch](batch-low-pri-vms.md). |
| $NodeDeallocationOption |La acción que se produce cuando se quitan los nodos de ejecución de un grupo. Los valores posibles son:<ul><li>**requeue**: El valor predeterminado. Finaliza las tareas de inmediato y las vuelve a colocar en la cola de trabajos para que se vuelvan a programar. Esta acción garantiza que se alcance el número de nodos de destino lo más rápido posible. Sin embargo, puede ser menos eficaz, ya que cualquier tarea en ejecución se interrumpirá y, a continuación, deberá reiniciarse completamente. <li>**terminate**: finaliza las tareas de inmediato y las quita de la cola de trabajos.<li>**taskcompletion**: espera a que finalicen las tareas actualmente en ejecución y, a continuación, quita el nodo del grupo. Esta opción se usa para evitar que las tareas se interrumpan y se vuelvan a poner en la cola, lo que hace que cualquier trabajo que haya realizado hasta ese momento sea baldío.<li>**retaineddata**: espera a que todos los datos que se conservan en la tarea local en el nodo se limpien antes de quitar el nodo del grupo.</ul> |

> [!NOTE]
> También se puede especificar la variable `$TargetDedicatedNodes` mediante el alias `$TargetDedicated`. También se puede especificar la variable `$TargetLowPriorityNodes` mediante el alias `$TargetLowPriority`. Si la fórmula establece tanto la variable con nombre completo como su alias, el valor asignado a la variable tendrá prioridad.

### <a name="read-only-service-defined-variables"></a>Variables definidas por el servicio de solo lectura

Puede obtener el valor de estas variables definidas por el servicio para efectuar ajustes basados en las métricas del servicio Batch.

> [!IMPORTANT]
> Las tareas de liberación del trabajo no se incluyen actualmente en las variables que proporcionan recuentos de tareas, como $ActiveTasks y $PendingTasks. En función de la fórmula de escalabilidad automática, esto puede dar lugar a que se quiten los nodos y no haya ninguno disponible para ejecutar las tareas de liberación del trabajo.

| Variable | Descripción |
| --- | --- |
| $CPUPercent |El porcentaje medio de uso de CPU. |
| $WallClockSeconds |El número de segundos consumidos. |
| $MemoryBytes |La media de megabytes usados. |
| $DiskBytes |La media de gigabytes usados en los discos locales. |
| $DiskReadBytes |El número de bytes leídos. |
| $DiskWriteBytes |El número de bytes escritos. |
| $DiskReadOps |El número de operaciones de disco de lectura realizadas. |
| $DiskWriteOps |El número de operaciones de disco de escritura realizadas. |
| $NetworkInBytes |El número de bytes entrantes. |
| $NetworkOutBytes |El número de bytes salientes. |
| $SampleNodeCount |El número de nodos de ejecución. |
| $ActiveTasks |Número de tareas que están listas para su ejecución pero que aún no se están ejecutando. Esto incluye todas las tareas que tienen un estado activo y cuyas dependencias se han cumplido. Todas las tareas que tengan un estado activo pero cuyas dependencias no se hayan cumplido se excluirán del recuento de $ActiveTasks. En una tarea de varias instancias, $ActiveTasks incluirá el número de instancias establecido en la tarea.|
| $RunningTasks |El número de tareas en un estado de ejecución. |
| $PendingTasks |La suma de $ActiveTasks y $RunningTasks. |
| $SucceededTasks |El número de tareas que finalizó correctamente. |
| $FailedTasks |El número de tareas erróneas. |
| $CurrentDedicatedNodes |El número actual de dedicado de nodos de ejecución dedicados. |
| $CurrentLowPriorityNodes |Número actual de nodos de ejecución de prioridad baja, incluidos todos los nodos con prioridad. |
| $PreemptedNodeCount | El número de nodos en el grupo que se encuentran en estado reemplazado. |

> [!TIP]
> Estas variables de solo lectura definidas por el servicio son *objetos* que proporcionan varios métodos para acceder a los datos asociados a cada uno de ellos. Para obtener más información, consulte la sección [Obtención de datos de ejemplo](#obtain-sample-data), descrita más adelante en este artículo.

## <a name="types"></a>Tipos

Las fórmulas de escalabilidad automática admiten los tipos siguientes:

- double
- doubleVec
- doubleVecList
- string
- timestamp: una estructura compuesta que contiene los miembros siguientes:
  - year
  - mes (1-12)
  - día (1-31)
  - día de la semana (en formato de número; por ejemplo, 1 para lunes)
  - hora (en formato de número de 24 horas; por ejemplo, 13 significa 1 p.m.)
  - minuto (00-59)
  - segundo (00-59)
- timeinterval
  - TimeInterval_Zero
  - TimeInterval_100ns
  - TimeInterval_Microsecond
  - TimeInterval_Millisecond
  - TimeInterval_Second
  - TimeInterval_Minute
  - TimeInterval_Hour
  - TimeInterval_Day
  - TimeInterval_Week
  - TimeInterval_Year

## <a name="operations"></a>Operaciones

Estas operaciones se permiten en los tipos que constan en la sección anterior.

| Operación | Operadores admitidos | Tipo de resultado |
| --- | --- | --- |
| double *operador* double |+, -, *, / |double |
| double *operador* timeinterval |* |timeinterval |
| doubleVec *operador* double |+, -, *, / |doubleVec |
| doubleVec *operador* doubleVec |+, -, *, / |doubleVec |
| timeinterval *operador* double |*, / |timeinterval |
| timeinterval *operador* timeinterval |+, - |timeinterval |
| timeinterval *operador* timestamp |+ |timestamp |
| timestamp *operador* timeinterval |+ |timestamp |
| timestamp *operador* timestamp |- |timeinterval |
| *operador* double |-, ! |double |
| *operador* timeinterval |- |timeinterval |
| double *operador* double |<, <=, ==, >=, >, != |double |
| string *operador* string |<, <=, ==, >=, >, != |double |
| timestamp *operador* timestamp |<, <=, ==, >=, >, != |double |
| timeinterval *operador* timeinterval |<, <=, ==, >=, >, != |double |
| double *operador* double |&&, &#124;&#124; |double |

Cuando se prueba un valor double con un operador ternario (`double ? statement1 : statement2`), el valor distinto de cero es **true** y cero es **false**.

## <a name="functions"></a>Functions

Puede usar estas **funciones** predefinidas al definir una fórmula de escalabilidad automática.

| Función | Tipo de valor devuelto | Descripción |
| --- | --- | --- |
| avg(doubleVecList) |double |Devuelve el valor medio de todos los valores de doubleVecList. |
| len(doubleVecList) |double |Devuelve la longitud del vector creado a partir de doubleVecList. |
| lg(double) |double |Devuelve la base logarítmica 2 de double. |
| lg(doubleVecList) |doubleVec |Devuelve la base logarítmica de componentes 2 de doubleVecList. Se debe pasar explícitamente un vec(double) para el parámetro. En caso contrario, se supone la versión double lg(double). |
| ln(double) |double |Devuelve el registro natural de double. |
| ln(doubleVecList) |doubleVec |Devuelve el registro natural de double. |
| log(double) |double |Devuelve la base logarítmica 10 de double. |
| log(doubleVecList) |doubleVec |Devuelve la base logarítmica de componentes 10 de doubleVecList. Se debe pasar explícitamente un vec(double) para un único parámetro double. En caso contrario, se supone la versión double log(double). |
| max(doubleVecList) |double |Devuelve el valor máximo de doubleVecList. |
| min(doubleVecList) |double |Devuelve el valor mínimo de doubleVecList. |
| norm(doubleVecList) |double |Devuelve la norma de dos del vector creado a partir de doubleVecList. |
| percentile(doubleVec v, double p) |double |Devuelve el elemento percentil del vector v. |
| rand() |double |Devuelve un valor aleatorio entre 0,0 y 1,0. |
| range(doubleVecList) |double |Devuelve la diferencia entre los valores máximo y mínimo en doubleVecList. |
| std(doubleVecList) |double |Devuelve la desviación de muestra estándar de los valores en doubleVecList. |
| stop() | |Detiene la evaluación de la expresión de escalado automático. |
| sum(doubleVecList) |double |Devuelve la suma de todos los componentes de doubleVecList. |
| time(string dateTime="") |timestamp |Devuelve la marca de tiempo de la hora actual si no se pasan los parámetros o la marca de hora de la cadena dateTime si se pasó. Los formatos de dateTime compatibles son W3C-DTF y RFC 1123. |
| val(doubleVec v, double i) |double |Devuelve el valor del elemento que está en la ubicación i en el vector v con el índice inicial de cero. |

Algunas de las funciones descritas en la tabla anterior pueden aceptar una lista como argumento. La lista separada por comas es cualquier combinación de *double* y *doubleVec*. Por ejemplo:

`doubleVecList := ( (double | doubleVec)+(, (double | doubleVec) )* )?`

El valor *doubleVecList* se convierte en un *doubleVec* individual antes de la evaluación. Por ejemplo, si `v = [1,2,3]`, entonces, llamar a `avg(v)` es equivalente a llamar a `avg(1,2,3)`. Llamar a `avg(v, 7)` es equivalente a llamar a `avg(1,2,3,7)`.

## <a name="metrics"></a>Métricas

Puede usar tanto métricas de recurso como de tarea al definir una fórmula. El número de nodos dedicados de destino se ajusta en el grupo basándose en los datos de métricas que obtenga y evalúe. Para más información sobre cada métrica, consulte la sección [Variables](#variables) más arriba.

<table>
  <tr>
    <th>Métrica</th>
    <th>Descripción</th>
  </tr>
  <tr>
    <td><b>Recurso</b></td>
    <td><p>Las métricas de recurso se basan en la CPU, el ancho de banda y el uso de la memoria de los nodos de ejecución, así como en el número de nodos.</p>
        <p> Estas variables definidas por el servicio se usan para realizar ajustes basados en el número de nodos:</p>
    <p><ul>
            <li>$TargetDedicatedNodes</li>
            <li>$TargetLowPriorityNodes</li>
            <li>$CurrentDedicatedNodes</li>
            <li>$CurrentLowPriorityNodes</li>
            <li>$PreemptedNodeCount</li>
            <li>$SampleNodeCount</li>
    </ul></p>
    <p>Estas variables definidas por el servicio se usan para realizar ajustes basados en el uso de recursos de nodo:</p>
    <p><ul>
      <li>$CPUPercent</li>
      <li>$WallClockSeconds</li>
      <li>$MemoryBytes</li>
      <li>$DiskBytes</li>
      <li>$DiskReadBytes</li>
      <li>$DiskWriteBytes</li>
      <li>$DiskReadOps</li>
      <li>$DiskWriteOps</li>
      <li>$NetworkInBytes</li>
      <li>$NetworkOutBytes</li></ul></p>
  </tr>
  <tr>
    <td><b>Task</b></td>
    <td><p>Las métricas de tarea se basan en el estado de las tareas, como Activo, Pendiente y Completado. Las siguientes variables definidas por el servicio se usan para realizar ajustes de tamaño de grupo basados en las métricas de tarea:</p>
    <p><ul>
      <li>$ActiveTasks</li>
      <li>$RunningTasks</li>
      <li>$PendingTasks</li>
      <li>$SucceededTasks</li>
            <li>$FailedTasks</li></ul></p>
        </td>
  </tr>
</table>

## <a name="obtain-sample-data"></a>Obtención de datos de muestra

La operación principal de una fórmula de escalabilidad automática es la obtención de datos de métricas de tareas y recursos (muestras) y el ajuste posterior del tamaño de grupo según esos datos. Por lo tanto, es importante tener una idea clara de cómo interactúan las fórmulas de escalabilidad automática con las muestras.

### <a name="methods"></a>Métodos

Las fórmulas de escalabilidad automática actúan en muestras de datos de métricas proporcionados por el servicio Batch. Una fórmula aumentará o reducirá el tamaño del grupo en función de los valores que obtenga. Las variables definidas por el servicio son objetos que proporcionan métodos para acceder a los datos asociados a ese objeto. Por ejemplo, la siguiente expresión muestra una solicitud para obtener los últimos cinco minutos de uso de CPU:

```
$CPUPercent.GetSample(TimeInterval_Minute * 5)
```

Los métodos siguientes se pueden usar para obtener datos de muestra sobre las variables definidas por el servicio.

| Método | Descripción |
| --- | --- |
| GetSample() |El método `GetSample()` devuelve un vector de muestras de datos.<br/><br/>Un ejemplo tiene un valor de 30 segundos de datos de métrica. En otras palabras, las muestras se obtienen cada 30 segundos. No obstante, tal como se mencionó anteriormente, hay un retraso entre cuándo se recopila una muestra y cuándo está disponible para una fórmula. Por lo tanto, puede que no todos los ejemplos durante un período de tiempo determinado estén disponibles para la evaluación a través de una fórmula.<ul><li>`doubleVec GetSample(double count)`: Especifica el número de muestras que se obtienen a partir de las muestras recogidas más recientes. `GetSample(1)` devuelve la última muestra disponible. Sin embargo, para métricas como `$CPUPercent`, `GetSample(1)` no debe usarse, porque es imposible saber *cuándo* se recopiló la muestra. Puede ser reciente o, debido a problemas del sistema, puede ser mucho más antiguo. En estos casos, es mejor usar un intervalo de tiempo como se muestra a continuación.<li>`doubleVec GetSample((timestamp or timeinterval) startTime [, double samplePercent])`: Especifica un período de tiempo para recopilar datos de muestra. Opcionalmente, también especifica el porcentaje de muestras que deben estar disponibles en el marco de tiempo solicitado. Por ejemplo, `$CPUPercent.GetSample(TimeInterval_Minute * 10)` debería devolver 20 muestras si todas las muestras de los últimos 10 minutos están presentes en el historial de `CPUPercent`. Si el último minuto del historial no estaba disponible, solo se devolverán 18 muestras. En este caso, `$CPUPercent.GetSample(TimeInterval_Minute * 10, 95)` generaría un error porque solo está disponible el 90 por ciento de las muestras, pero `$CPUPercent.GetSample(TimeInterval_Minute * 10, 80)` se realizaría correctamente.<li>`doubleVec GetSample((timestamp or timeinterval) startTime, (timestamp or timeinterval) endTime [, double samplePercent])`: Especifica un período de tiempo para recopilar datos, con una hora de inicio y una hora de finalización. Como se mencionó anteriormente, hay un retraso entre cuando se colecta una muestra y cuando está disponible para una fórmula. Tenga en cuenta este retraso a la hora de usar el método `GetSample`. Consulte `GetSamplePercent` a continuación. |
| GetSamplePeriod() |Devuelve el período de las muestras tomadas en un conjunto de datos de muestras históricos. |
| Count() |Devuelve el número total de ejemplos en el historial de métrica. |
| HistoryBeginTime() |Devuelve la marca de tiempo de la muestra de datos disponible más antigua para la métrica. |
| GetSamplePercent() |Devuelve el porcentaje de muestras que están disponibles para un intervalo de tiempo dado. Por ejemplo, `doubleVec GetSamplePercent( (timestamp or timeinterval) startTime [, (timestamp or timeinterval) endTime] )`. Debido a que se produce un error en el método `GetSample` si el porcentaje de muestras devueltas es inferior al valor de `samplePercent` especificado, puede utilizar el método `GetSamplePercent` para comprobarlo primero. A continuación, puede realizar una acción alternativa si no hay suficientes muestras presentes, sin detener la evaluación de escalado automático. |

### <a name="samples"></a>Ejemplos

El servicio de Batch toma periódicamente ejemplos de métricas de tareas y recursos y los pone a disposición de las fórmulas de escalado automático. El servicio Batch graba estas muestras cada 30 segundos. Pero suele producirse un retraso entre el momento en el que se grabaron esas muestras y el momento en el que se ponen a disposición de las fórmulas de escalado automático y estas pueden leer aquellas. Además, es posible que las muestras no se hayan grabado para un intervalo en concreto debido a factores como problemas con la red o de infraestructura.

### <a name="sample-percentage"></a>Porcentaje de muestras

Al pasar `samplePercent` al método `GetSample()` o al llamar al método `GetSamplePercent()`, _porcentaje_ hace referencia a una comparación entre el número posible total de ejemplos que graba el servicio de Batch y el número de muestras que están disponibles para la fórmula de escalado automático.

Echemos un vistazo a un intervalo de tiempo de 10 minutos como ejemplo. Dado que las muestras se graban cada 30 segundos, el número total máximo de muestras que Batch graba en ese intervalo de tiempo de 10 minutos habría sido de 20 muestras (2 por minuto). Pero, debido a la latencia inherente del mecanismo de informes y a otros problemas existentes en Azure, podría haber solo 15 muestras disponibles para leer la fórmula de escalado automático. Así, por ejemplo, durante ese período de 10 minutos, solo el 75 % del número total de ejemplos que se graban podría estar disponible para la fórmula.

### <a name="getsample-and-sample-ranges"></a>GetSample() e intervalos de muestra

Las fórmulas de escalabilidad automática aumentarán y reducirán el tamaño de los grupos al agregar o quitar nodos. Dado que los nodos cuestan dinero, asegúrese de que las fórmulas usan un método de análisis inteligente que se basa en un número de datos suficiente. Se recomienda usar un análisis de tipo tendencias en las fórmulas. Este tipo aumenta y reduce los grupos basándose en un intervalo de ejemplos recopilados.

Para ello, use `GetSample(interval look-back start, interval look-back end)` para devolver un vector de ejemplos:

```
$runningTasksSample = $RunningTasks.GetSample(1 * TimeInterval_Minute, 6 * TimeInterval_Minute);
```

Cuando Batch evalúe la línea anterior, devolverá un intervalo de muestras como vector de valores. Por ejemplo:

```
$runningTasksSample=[1,1,1,1,1,1,1,1,1,1];
```

Una vez recopilado el vector de ejemplos, puede usar funciones como `min()`, `max()` y `avg()` para derivar valores significativos del intervalo recopilado.

Para mayor seguridad, puede forzar el error en una evaluación de fórmula si menos de un determinado porcentaje de ejemplos está disponible para un período de tiempo determinado. Al forzar el error en una evaluación de fórmula, indica a Batch que deje de evaluar la fórmula si el porcentaje de muestras especificado no está disponible. En este caso, no se efectúa ningún cambio en el tamaño del grupo. Para especificar un porcentaje necesario de ejemplos para que la evaluación se realice correctamente, especifíquelo como el tercer parámetro de `GetSample()`. En este caso, se especifica un requisito del 75 por ciento de muestras:

```
$runningTasksSample = $RunningTasks.GetSample(60 * TimeInterval_Second, 120 * TimeInterval_Second, 75);
```

Dado que puede haber un retraso en la disponibilidad de muestras, siempre debe especificar siempre un intervalo de tiempo con una hora de inicio retrospectiva cuya anterioridad sea superior a un minuto. Las muestras tardan aproximadamente un minuto en propagarse por el sistema, por lo que puede que las muestras del rango `(0 * TimeInterval_Second, 60 * TimeInterval_Second)` no estén disponibles. De nuevo, puede utilizar el parámetro de porcentaje de `GetSample()` para forzar un requisito de porcentaje de ejemplos concreto.

> [!IMPORTANT]
> Es muy recomendable que **evite confiar *solo* en `GetSample(1)` en las fórmulas de escalabilidad automática**. Esto se debe a que `GetSample(1)` dice básicamente al servicio Batch "dame la muestra más reciente que tengas, independientemente de cuánto tiempo hace que la tienes". Puesto que es solo una muestra única, y puede ser una muestra más antigua, no puede ser representativa de la imagen más grande del estado reciente de la tarea o el recurso. Si usa `GetSample(1)`, asegúrese de que forma parte de una instrucción más grande y no solo el punto de datos en el que se basa la fórmula.

## <a name="write-an-autoscale-formula"></a>Escritura de una fórmula de escalado automático

Para construir una fórmula de escalado automático, es preciso formar instrucciones que usen los componentes anteriores y luego combinar esas instrucciones en una fórmula completa. En esta sección vamos a crear una fórmula de escalabilidad automática de ejemplo que puede tomar decisiones reales de escalado y hacer ajustes.

En primer lugar, vamos a definir los requisitos de nuestra nueva fórmula de escalado automático. La fórmula debe:

- Aumentar el número objetivo de nodos de ejecución dedicados en un grupo si el uso de la CPU es alto.
- Reducir el número objetivo de nodos de ejecución dedicados en un grupo si el uso de la CPU es bajo.
- Restringir siempre el número máximo de nodos dedicados a 400.
- Al reducir el número de nodos, no quite los nodos que ejecuten tareas; si fuera necesario, espere a que las tareas finalicen para quitar nodos.

La primera instrucción de la fórmula aumentará el número de nodos durante un uso elevado de la CPU. Definiremos una instrucción que rellena una variable definida por el usuario (`$totalDedicatedNodes`) con un valor que representa un 110 por ciento del número de nodos dedicados del destino actual, pero solo si el uso medio mínimo de la CPU en los 10 últimos minutos era superior al 70 por ciento. En caso contrario, usa el valor del número actual de nodos dedicados.

```
$totalDedicatedNodes =
    (min($CPUPercent.GetSample(TimeInterval_Minute * 10)) > 0.7) ?
    ($CurrentDedicatedNodes * 1.1) : $CurrentDedicatedNodes;
```

Para reducir el número de nodos dedicados durante un uso bajo de la CPU, la siguiente instrucción de nuestra fórmula establece la misma variable `$totalDedicatedNodes` en el 90 por ciento del número objetivo actual de nodos dedicados si el uso medio de la CPU en los últimos 60 minutos estaba por debajo del 20 por ciento. De lo contrario, usa el valor actual de `$totalDedicatedNodes` que se rellenamos en la instrucción anterior.

```
$totalDedicatedNodes =
    (avg($CPUPercent.GetSample(TimeInterval_Minute * 60)) < 0.2) ?
    ($CurrentDedicatedNodes * 0.9) : $totalDedicatedNodes;
```

Ahora limitaremos el número objetivo de nodos de ejecución dedicados a un máximo de 400.

```
$TargetDedicatedNodes = min(400, $totalDedicatedNodes)
```

Por último, nos aseguraremos de que los nodos no se quiten hasta que finalicen sus tareas.

```
$NodeDeallocationOption = taskcompletion;
```

Esta es la fórmula completa:

```
$totalDedicatedNodes =
    (min($CPUPercent.GetSample(TimeInterval_Minute * 10)) > 0.7) ?
    ($CurrentDedicatedNodes * 1.1) : $CurrentDedicatedNodes;
$totalDedicatedNodes =
    (avg($CPUPercent.GetSample(TimeInterval_Minute * 60)) < 0.2) ?
    ($CurrentDedicatedNodes * 0.9) : $totalDedicatedNodes;
$TargetDedicatedNodes = min(400, $totalDedicatedNodes)
$NodeDeallocationOption = taskcompletion;
```

> [!NOTE]
> Si elige hacerlo, puede incluir comentarios y saltos de línea en las cadenas de fórmulas.

## <a name="automatic-scaling-interval"></a>Intervalo de escalado automático

De forma predeterminada, el servicio de Batch ajusta el tamaño de un grupo según su fórmula de escalado automático cada 15 minutos. Este intervalo se puede configurar con las siguientes propiedades de grupo:

- [CloudPool.AutoScaleEvaluationInterval](/dotnet/api/microsoft.azure.batch.cloudpool.autoscaleevaluationinterval) (Batch .NET)
- [autoScaleEvaluationInterval](/rest/api/batchservice/enable-automatic-scaling-on-a-pool) (API de REST)

Los intervalos mínimo y máximo son cinco minutos y 168 horas, respectivamente. Si se especifica un intervalo que se sitúa fuera de este margen, el servicio de Batch devolverá un error de solicitud incorrecta (400).

> [!NOTE]
> Actualmente, el escalado automático no está pensado como respuesta inmediata a los cambios, sino para ajustar el tamaño del grupo gradualmente mientras ejecuta una carga de trabajo.

## <a name="create-an-autoscale-enabled-pool-with-batch-sdks"></a>Creación de un grupo habilitado para el escalado automático con SDK de Batch

El escalado automático de grupo se puede configurar con cualquiera de los [SDK de Batch](batch-apis-tools.md#azure-accounts-for-batch-development), la [API de REST de Batch](/rest/api/batchservice/), los [cmdlets de PowerShell de Batch](batch-powershell-cmdlets-get-started.md)y la [CLI de Batch](batch-cli-get-started.md). En esta sección, puede ver ejemplos de .NET y Python.

### <a name="net"></a>.NET

Para crear un grupo con el escalado automático habilitado en. NET, siga estos pasos:

1. Crear el grupo con [BatchClient.PoolOperations.CreatePool](/dotnet/api/microsoft.azure.batch.pooloperations.createpool).
1. Establecer la propiedad [CloudPool.AutoScaleEnabled](/dotnet/api/microsoft.azure.batch.cloudpool.autoscaleenabled) en `true`.
1. Establecer la propiedad [CloudPool.AutoScaleFormula](/dotnet/api/microsoft.azure.batch.cloudpool.autoscaleformula) con la fórmula de escalado automático.
1. (Opcional) Establecer la propiedad [CloudPool.AutoScaleEvaluationInterval](/dotnet/api/microsoft.azure.batch.cloudpool.autoscaleevaluationinterval) (el valor predeterminado es 15 minutos).
1. Confirmar el grupo con [CloudPool.Commit](/dotnet/api/microsoft.azure.batch.cloudpool.commit) o [CommitAsync](/dotnet/api/microsoft.azure.batch.cloudpool.commitasync).

En el ejemplo siguiente se crea un grupo habilitado para la escalabilidad automática en .NET. La fórmula de escalabilidad automática del grupo establece el número objetivo de nodos dedicados en 5 los lunes y en 1 los demás días de la semana. El [intervalo de escalado automático](#automatic-scaling-interval) está establecido en 30 minutos. En este y en otros fragmentos de código de C# de este artículo, `myBatchClient` es una instancia inicializada correctamente de la clase [BatchClient](/dotnet/api/microsoft.azure.batch.batchclient).

```csharp
CloudPool pool = myBatchClient.PoolOperations.CreatePool(
                    poolId: "mypool",
                    virtualMachineSize: "standard_d1_v2",
                    cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));
pool.AutoScaleEnabled = true;
pool.AutoScaleFormula = "$TargetDedicatedNodes = (time().weekday == 1 ? 5:1);";
pool.AutoScaleEvaluationInterval = TimeSpan.FromMinutes(30);
await pool.CommitAsync();
```

> [!IMPORTANT]
> Al crear un grupo habilitado para la escalabilidad automática, no especifique el parámetro _targetDedicatedNodes_ ni el parámetro _targetLowPriorityNodes_ en la llamada a **CreatePool**. En su lugar, especifique las propiedades **AutoScaleEnabled** y **AutoScaleFormula** en el grupo. Los valores de estas propiedades determinan el número de destino de cada tipo de nodo.
>
> Para cambiar manualmente el tamaño de un grupo habilitado para la escalabilidad automática (por ejemplo, con [BatchClient.PoolOperations.ResizePoolAsync](/dotnet/api/microsoft.azure.batch.pooloperations.resizepoolasync)), primero debe deshabilitar el escalado automático en el grupo y, después, cambiar su tamaño.

### <a name="python"></a>Python

Para crear un grupo habilitado para la escalabilidad automática con el SDK de Python:

1. Cree un grupo y especifique su configuración.
1. Agregue el grupo al cliente del servicio.
1. Habilite el escalado automático en el grupo con una fórmula que escriba.

En el ejemplo siguiente se ilustran estos pasos.

```python
# Create a pool; specify configuration
new_pool = batch.models.PoolAddParameter(
    id="autoscale-enabled-pool",
    virtual_machine_configuration=batchmodels.VirtualMachineConfiguration(
        image_reference=batchmodels.ImageReference(
          publisher="Canonical",
          offer="UbuntuServer",
          sku="18.04-LTS",
          version="latest"
            ),
        node_agent_sku_id="batch.node.ubuntu 18.04"),
    vm_size="STANDARD_D1_v2",
    target_dedicated_nodes=0,
    target_low_priority_nodes=0
)
batch_service_client.pool.add(new_pool) # Add the pool to the service client

formula = """$curTime = time();
             $workHours = $curTime.hour >= 8 && $curTime.hour < 18;
             $isWeekday = $curTime.weekday >= 1 && $curTime.weekday <= 5;
             $isWorkingWeekdayHour = $workHours && $isWeekday;
             $TargetDedicated = $isWorkingWeekdayHour ? 20:10;""";

# Enable autoscale; specify the formula
response = batch_service_client.pool.enable_auto_scale(pool_id, auto_scale_formula=formula,
                                            auto_scale_evaluation_interval=datetime.timedelta(minutes=10),
                                            pool_enable_auto_scale_options=None,
                                            custom_headers=None, raw=False)
```

> [!TIP]
> Puede encontrar más ejemplos del uso del SDK de Python en el [repositorio de inicio rápido de Python Batch](https://github.com/Azure-Samples/batch-python-quickstart) en GitHub.

## <a name="enable-autoscaling-on-an-existing-pool"></a>Habilitación del escalado automático en un grupo existente

Cada SDK de Batch proporciona un método para habilitar el escalado automático. Por ejemplo:

- [BatchClient.PoolOperations.EnableAutoScaleAsync](/dotnet/api/microsoft.azure.batch.pooloperations.enableautoscaleasync) (Batch .NET)
- [Activar la escala automática en un grupo de servidores](/rest/api/batchservice/enable-automatic-scaling-on-a-pool) (API de REST)

Al habilitar el escalado automático en un grupo existente, tenga en cuenta lo siguiente:

- Si actualmente el escalado automático está deshabilitado en el grupo, debe especificar una fórmula de escalabilidad automática válida al emitir la solicitud. De manera opcional, puede especificar un intervalo de escalado automático. Si no especifica un intervalo, se usa el valor predeterminado de 15 minutos.
- Si actualmente el escalado automático está habilitado en el grupo, puede especificar una fórmula nueva, un intervalo nuevo o ambos. Debe especificar al menos una de estas propiedades.
  - Si especifica un intervalo de escalado automático nuevo, la programación existente se detiene y se inicia una nueva. La hora de inicio de la nueva programación es la hora a la que se emitió la solicitud para habilitar el escalado automático.
  - Si se omite la fórmula o el intervalo de escalabilidad automática, el servicio Batch sigue usando el valor actual de dicha configuración.

> [!NOTE]
> Si especificó valores para los parámetros *targetDedicatedNodes* o *targetLowPriorityNodes* del método **CreatePool** al crear el grupo en .NET, o bien para los parámetros comparables en otro lenguaje, dichos valores se omitirán cuando se evalúe la fórmula de escalabilidad automática.

Este ejemplo de C# usa la biblioteca [.NET de Batch](/dotnet/api/microsoft.azure.batch) para habilitar el escalado automático en un grupo existente.

```csharp
// Define the autoscaling formula. This formula sets the target number of nodes
// to 5 on Mondays, and 1 on every other day of the week
string myAutoScaleFormula = "$TargetDedicatedNodes = (time().weekday == 1 ? 5:1);";

// Set the autoscale formula on the existing pool
await myBatchClient.PoolOperations.EnableAutoScaleAsync(
    "myexistingpool",
    autoscaleFormula: myAutoScaleFormula);
```

### <a name="update-an-autoscale-formula"></a>Actualización de una fórmula de escalado automático

Para actualizar la fórmula en un grupo existente habilitado para el escalado automático, vuelva a llamar a la operación para habilitar el escalado automático con la nueva fórmula. Por ejemplo, si el escalado automático ya está habilitado en `myexistingpool` cuando se ejecuta el siguiente código de .NET, su fórmula de escalado automático se sustituirá por el contenido de `myNewFormula`.

```csharp
await myBatchClient.PoolOperations.EnableAutoScaleAsync(
    "myexistingpool",
    autoscaleFormula: myNewFormula);
```

### <a name="update-the-autoscale-interval"></a>Actualización del intervalo de escalado automático

Para actualizar el intervalo de evaluación de escalado automático en un grupo existente habilitado para el escalado automático, vuelva a llamar a la operación para habilitar el escalado automático con el nuevo intervalo. Por ejemplo, para establecer el intervalo de evaluación de escalado automático en 60 minutos para un grupo que ya tiene habilitado el escalado automático en .NET:

```csharp
await myBatchClient.PoolOperations.EnableAutoScaleAsync(
    "myexistingpool",
    autoscaleEvaluationInterval: TimeSpan.FromMinutes(60));
```

## <a name="evaluate-an-autoscale-formula"></a>Evaluación de una fórmula de escalado automático

Puede evaluar una fórmula antes de aplicarla a un grupo. Esto le permite probar los resultados de la fórmula antes de ponerla en producción.

Para poder evaluar una fórmula de escalabilidad automática, primero debe habilitar el escalado automático en el grupo con una fórmula válida, como la fórmula de una línea `$TargetDedicatedNodes = 0`. A continuación, use uno de los siguientes métodos para evaluar la fórmula que quiere probar:

- [BatchClient.PoolOperations.EvaluateAutoScale](/dotnet/api/microsoft.azure.batch.pooloperations.evaluateautoscale) o [EvaluateAutoScaleAsync](/dotnet/api/microsoft.azure.batch.pooloperations.evaluateautoscaleasync)

    Estos métodos .NET de Batch requieren el identificador de un grupo existente y una cadena que contenga la fórmula de escalado automático que se evaluará.

- [Evaluación de una fórmula de escalado automático](/rest/api/batchservice/evaluate-an-automatic-scaling-formula)

    En esta solicitud de API de REST, especifique el identificador del grupo en el URI y la fórmula de escalado automático en el elemento *autoScaleFormula* del cuerpo de la solicitud. La respuesta de la operación contiene cualquier información de error que pueda relacionarse con la fórmula.

Este ejemplo de [.NET de Batch](/dotnet/api/microsoft.azure.batch) evalúa una fórmula de escalabilidad automática. Si el grupo aún no usa el escalado automático, primero se habilita.

```csharp
// First obtain a reference to an existing pool
CloudPool pool = await batchClient.PoolOperations.GetPoolAsync("myExistingPool");

// If autoscaling isn't already enabled on the pool, enable it.
// You can't evaluate an autoscale formula on a non-autoscale-enabled pool.
if (pool.AutoScaleEnabled == false)
{
    // We need a valid autoscale formula to enable autoscaling on the
    // pool. This formula is valid, but won't resize the pool:
    await pool.EnableAutoScaleAsync(
        autoscaleFormula: "$TargetDedicatedNodes = $CurrentDedicatedNodes;",
        autoscaleEvaluationInterval: TimeSpan.FromMinutes(5));

    // Batch limits EnableAutoScaleAsync calls to once every 30 seconds.
    // Because we want to apply our new autoscale formula below if it
    // evaluates successfully, and we *just* enabled autoscaling on
    // this pool, we pause here to ensure we pass that threshold.
    Thread.Sleep(TimeSpan.FromSeconds(31));

    // Refresh the properties of the pool so that we've got the
    // latest value for AutoScaleEnabled
    await pool.RefreshAsync();
}

// We must ensure that autoscaling is enabled on the pool prior to
// evaluating a formula
if (pool.AutoScaleEnabled == true)
{
    // The formula to evaluate - adjusts target number of nodes based on
    // day of week and time of day
    string myFormula = @"
        $curTime = time();
        $workHours = $curTime.hour >= 8 && $curTime.hour < 18;
        $isWeekday = $curTime.weekday >= 1 && $curTime.weekday <= 5;
        $isWorkingWeekdayHour = $workHours && $isWeekday;
        $TargetDedicatedNodes = $isWorkingWeekdayHour ? 20:10;
    ";

    // Perform the autoscale formula evaluation. Note that this code does not
    // actually apply the formula to the pool.
    AutoScaleRun eval =
        await batchClient.PoolOperations.EvaluateAutoScaleAsync(pool.Id, myFormula);

    if (eval.Error == null)
    {
        // Evaluation success - print the results of the AutoScaleRun.
        // This will display the values of each variable as evaluated by the
        // autoscale formula.
        Console.WriteLine("AutoScaleRun.Results: " +
            eval.Results.Replace("$", "\n    $"));

        // Apply the formula to the pool since it evaluated successfully
        await batchClient.PoolOperations.EnableAutoScaleAsync(pool.Id, myFormula);
    }
    else
    {
        // Evaluation failed, output the message associated with the error
        Console.WriteLine("AutoScaleRun.Error.Message: " +
            eval.Error.Message);
    }
}
```

Una evaluación correcta de la fórmula que se muestra en este fragmento de código genera unos resultados similares a esto:

```
AutoScaleRun.Results:
    $TargetDedicatedNodes=10;
    $NodeDeallocationOption=requeue;
    $curTime=2016-10-13T19:18:47.805Z;
    $isWeekday=1;
    $isWorkingWeekdayHour=0;
    $workHours=0
```

## <a name="get-information-about-autoscale-runs"></a>Obtención de información sobre la ejecución del escalado automático

Para asegurarse de que la fórmula se ejecuta según lo esperado, le recomendamos que compruebe periódicamente los resultados de las ejecuciones de escalado automático que lleva a cabo Batch en su grupo. Para ello, obtenga (o actualice) una referencia al grupo y examine las propiedades de su última ejecución de escalabilidad automática.

En Batch .NET, la propiedad [CloudPool.AutoScaleRun](/dotnet/api/microsoft.azure.batch.cloudpool.autoscalerun) tiene varias propiedades que proporcionan información sobre la última ejecución de escalado automático efectuada en el grupo:

- [AutoScaleRun.Timestamp](/dotnet/api/microsoft.azure.batch.autoscalerun.timestamp)
- [AutoScaleRun.Results](/dotnet/api/microsoft.azure.batch.autoscalerun.results)
- [AutoScaleRun.Error](/dotnet/api/microsoft.azure.batch.autoscalerun.error)

En la API de REST, la solicitud para [obtener información sobre un grupo](/rest/api/batchservice/get-information-about-a-pool) devuelve información sobre el grupo (por ejemplo, información sobre la última ejecución de escalado automático en la propiedad [autoScaleRun](/rest/api/batchservice/get-information-about-a-pool)).

El ejemplo de C# siguiente usa la biblioteca Batch .NET para imprimir información sobre la última ejecución de escalado automático en el grupo _myPool_.

```csharp
await Cloud pool = myBatchClient.PoolOperations.GetPoolAsync("myPool");
Console.WriteLine("Last execution: " + pool.AutoScaleRun.Timestamp);
Console.WriteLine("Result:" + pool.AutoScaleRun.Results.Replace("$", "\n  $"));
Console.WriteLine("Error: " + pool.AutoScaleRun.Error);
```

Resultado de la muestra del ejemplo anterior:

```
Last execution: 10/14/2016 18:36:43
Result:
  $TargetDedicatedNodes=10;
  $NodeDeallocationOption=requeue;
  $curTime=2016-10-14T18:36:43.282Z;
  $isWeekday=1;
  $isWorkingWeekdayHour=0;
  $workHours=0
Error:
```

## <a name="example-autoscale-formulas"></a>Ejemplo de fórmulas de escalado automático

Echemos un vistazo a algunas fórmulas que muestran distintas formas de ajustar la cantidad de recursos de proceso de un grupo.

### <a name="example-1-time-based-adjustment"></a>Ejemplo 1: Ajuste basado en tiempo

Imagínese que quiere ajustar el tamaño del grupo en función de la hora y el día de la semana. En este ejemplo se muestra cómo aumentar o disminuir en consecuencia el número de nodos del grupo.

En primer lugar, la fórmula obtiene la hora actual. Si se trata de un día laborable (1-5) y dentro del horario laboral (de 8 a.m. a 6 p.m.), el tamaño del grupo de destino se establece en 20 nodos. De lo contrario, se establece en 10 nodos.

```
$curTime = time();
$workHours = $curTime.hour >= 8 && $curTime.hour < 18;
$isWeekday = $curTime.weekday >= 1 && $curTime.weekday <= 5;
$isWorkingWeekdayHour = $workHours && $isWeekday;
$TargetDedicatedNodes = $isWorkingWeekdayHour ? 20:10;
$NodeDeallocationOption = taskcompletion;
```

`$curTime` se puede ajustar para reflejar la zona horaria local mediante la adición de `time()` al producto de `TimeZoneInterval_Hour` y la diferencia horaria con UTC. Por ejemplo, utilice `$curTime = time() + (-6 * TimeInterval_Hour);` para Hora de verano de las Montañas (MDT). Tenga en cuenta que la diferencia horaria debe ajustarse al principio y al final del horario de verano (si procede).

### <a name="example-2-task-based-adjustment"></a>Ejemplo 2: Ajuste basado en tareas

En este ejemplo de C#, el tamaño del grupo se ajusta en función del número de tareas en la cola. Incluimos tanto comentarios como saltos de línea en las cadenas de la fórmula.

```csharp
// Get pending tasks for the past 15 minutes.
$samples = $PendingTasks.GetSamplePercent(TimeInterval_Minute * 15);
// If we have fewer than 70 percent data points, we use the last sample point,
// otherwise we use the maximum of last sample point and the history average.
$tasks = $samples < 70 ? max(0,$PendingTasks.GetSample(1)) : max( $PendingTasks.GetSample(1), avg($PendingTasks.GetSample(TimeInterval_Minute * 15)));
// If number of pending tasks is not 0, set targetVM to pending tasks, otherwise
// half of current dedicated.
$targetVMs = $tasks > 0? $tasks:max(0, $TargetDedicatedNodes/2);
// The pool size is capped at 20, if target VM value is more than that, set it
// to 20. This value should be adjusted according to your use case.
$TargetDedicatedNodes = max(0, min($targetVMs, 20));
// Set node deallocation mode - let running tasks finish before removing a node
$NodeDeallocationOption = taskcompletion;
```

### <a name="example-3-accounting-for-parallel-tasks"></a>Ejemplo 3: Contabilidad para tareas paralelas

Este ejemplo de C# ajusta el tamaño del grupo en función del número de tareas. Esta fórmula también tiene en cuenta el valor de [MaxTasksPerComputeNode](/dotnet/api/microsoft.azure.batch.cloudpool.maxtaskspercomputenode) que se ha establecido para el grupo. Este enfoque es útil en aquellas situaciones en las que la [ejecución de tareas paralelas](batch-parallel-node-tasks.md) se ha habilitado en el grupo.

```csharp
// Determine whether 70 percent of the samples have been recorded in the past
// 15 minutes; if not, use last sample
$samples = $ActiveTasks.GetSamplePercent(TimeInterval_Minute * 15);
$tasks = $samples < 70 ? max(0,$ActiveTasks.GetSample(1)) : max( $ActiveTasks.GetSample(1),avg($ActiveTasks.GetSample(TimeInterval_Minute * 15)));
// Set the number of nodes to add to one-fourth the number of active tasks
// (theMaxTasksPerComputeNode property on this pool is set to 4, adjust
// this number for your use case)
$cores = $TargetDedicatedNodes * 4;
$extraVMs = (($tasks - $cores) + 3) / 4;
$targetVMs = ($TargetDedicatedNodes + $extraVMs);
// Attempt to grow the number of compute nodes to match the number of active
// tasks, with a maximum of 3
$TargetDedicatedNodes = max(0,min($targetVMs,3));
// Keep the nodes active until the tasks finish
$NodeDeallocationOption = taskcompletion;
```

### <a name="example-4-setting-an-initial-pool-size"></a>Ejemplo 4: Configuración de un tamaño de grupo inicial

En este ejemplo se muestra un ejemplo de C# con una fórmula de escalabilidad automática que establece el tamaño del grupo en un número de nodos específico durante un período de tiempo inicial. Después, ajusta el tamaño del grupo en función del número de tareas activas y en ejecución.

En concreto, esta fórmula realiza lo siguiente:

- Establece el tamaño inicial del grupo en cuatro nodos.
- No ajusta el tamaño del grupo en los 10 primeros minutos del ciclo de vida del mismo.
- Después de 10 minutos, obtiene el valor máximo del número de tareas activas y en ejecución en los últimos 60 minutos.
  - Si ambos valores son 0 (lo que indica que no hay tareas activas o en ejecución en los últimos 60 minutos) el tamaño del grupo se establece en 0.
  - Si cualquier valor es mayor que cero, no hay cambios.

```csharp
string now = DateTime.UtcNow.ToString("r");
string formula = string.Format(@"
    $TargetDedicatedNodes = {1};
    lifespan         = time() - time(""{0}"");
    span             = TimeInterval_Minute * 60;
    startup          = TimeInterval_Minute * 10;
    ratio            = 50;

    $TargetDedicatedNodes = (lifespan > startup ? (max($RunningTasks.GetSample(span, ratio), $ActiveTasks.GetSample(span, ratio)) == 0 ? 0 : $TargetDedicatedNodes) : {1});
    ", now, 4);
```

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo [ejecutar varias tareas de manera simultánea en los nodos de ejecución del grupo](batch-parallel-node-tasks.md). Junto con el escalado automático, esto puede ayudar a reducir la duración del trabajo para algunas cargas de trabajo, lo que permite ahorrar dinero.
- Obtenga información sobre cómo [consultar de manera eficaz el servicio Azure Batch](batch-efficient-list-queries.md) para obtener una mayor eficacia.
