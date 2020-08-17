---
title: Procedimientos recomendados de escalado automático
description: Patrones de escalado automático en Azure para Web Apps, conjunto de escalado de máquinas virtuales y Cloud Services
ms.topic: conceptual
ms.date: 07/07/2017
ms.subservice: autoscale
ms.openlocfilehash: 414716fbbb36167e52c4f3b98c70ae7696ffea8f
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327062"
---
# <a name="best-practices-for-autoscale"></a>Procedimientos recomendados de escalado automático
La escalabilidad automática de Azure Monitor solo se aplica a [Virtual Machine Scale Sets](https://azure.microsoft.com/services/virtual-machine-scale-sets/), [Cloud Services](https://azure.microsoft.com/services/cloud-services/), [App Service - Web Apps](https://azure.microsoft.com/services/app-service/web/) y los [servicios de API Management](../../api-management/api-management-key-concepts.md).

## <a name="autoscale-concepts"></a>Conceptos de escalado automático

* Un recurso solo puede tener *una* configuración de escalado automático.
* Una configuración de escalado automático puede tener uno o varios perfiles y cada perfil, a su vez, puede tener una o varias reglas de escalado automático.
* Una configuración de escalado automático escala instancias *horizontalmente* aumentando las instancias y las *reduce horizontalmente* disminuyendo el número de instancias.
  Una configuración de escalado automático presenta unos valores máximo, mínimo y predeterminado de instancias.
* Un trabajo de escalado automático siempre lee la métrica asociada por la que realizar la escala y comprueba si se rebasó el umbral establecido para el escalado horizontal o la reducción horizontal. En [Métricas comunes de escalado automático de Azure Monitor](autoscale-common-metrics.md) encontrará una lista de métricas por las que el escalado automático puede escalar.
* Todos los umbrales se calculan en el nivel de instancia. Por ejemplo, "escalar horizontalmente por una instancia cuando el uso medio de la CPU es superior al 80 % cuando el número de instancias es 2" significa escalar horizontalmente cuando el uso medio de la CPU en todas las instancias sea superior al 80 %.
* Todos los errores de escalado automático se registran en el registro de actividad. Después, puede configurar una [alerta de registro de actividad](./activity-log-alerts.md) de forma que pueda recibir una notificación por correo electrónico, SMS o webhook siempre que haya un error de escalabilidad automática.
* De forma similar, todas las acciones de escalado correctas se publican en el registro de actividad. Después, puede configurar una alerta de registro de actividad de forma que pueda recibir una notificación por correo electrónico, SMS o webhook siempre que haya una acción de escalabilidad automática correcta. Puede configurar notificaciones de correo electrónico o webhook para recibir una notificación cada vez que se lleve a cabo una acción de escalado correcta a través de la pestaña de notificaciones de la configuración de escalado automático.

## <a name="autoscale-best-practices"></a>Procedimientos recomendados de escalado automático

Use los procedimientos recomendados al usar el escalado automático.

### <a name="ensure-the-maximum-and-minimum-values-are-different-and-have-an-adequate-margin-between-them"></a>Asegúrese de que los valores máximo y mínimo son diferentes y de que hay margen suficiente entre ellos

Si tiene una configuración en la que el valor mínimo es 2, el valor máximo es 2 y el número de instancias es 2, no se puede ejecutar ninguna acción de escalado. Mantenga un margen suficiente entre los números de instancias máximo y mínimo, que son inclusivos. El escalado automático siempre escala entre estos límites.

### <a name="manual-scaling-is-reset-by-autoscale-min-and-max"></a>El escalado manual se restablece al valor máximo y mínimo de escalado automático.

Si actualiza manualmente el recuento de instancias a un valor superior o inferior al máximo, el motor de escalado automático se ajusta automáticamente al valor mínimo (si está por debajo) o al máximo (si está por encima). Por ejemplo, establezca el intervalo entre 3 y 6. Si tiene una instancia en ejecución, el motor de escalabilidad automática escala a tres instancias cuando vuelve a ejecutarse. Del mismo modo, si establece manualmente la escalabilidad en ocho instancias, en la siguiente ejecución de escalabilidad automática se escalará de vuelta a seis instancias.  La escalabilidad manual es temporal a menos que restablezca también las reglas de escalabilidad automática.

### <a name="always-use-a-scale-out-and-scale-in-rule-combination-that-performs-an-increase-and-decrease"></a>Use siempre una combinación de reglas de escalado horizontal y reducción horizontal que realice un aumento y una disminución.
Si usa un único elemento de la combinación, la escalabilidad automática solo ocurrirá en una dirección (escalabilidad o reducción horizontal) hasta que alcance el conteo máximo o mínimo de instancias que se ha definido en el perfil. Esta no es la mejor opción; idealmente, querrá que el recurso escale verticalmente en los momentos de uso elevado para garantizar la disponibilidad. De forma similar, en los momentos de poco uso querrá que el recurso se reduzca verticalmente para ahorrar costos.

### <a name="choose-the-appropriate-statistic-for-your-diagnostics-metric"></a>Elija la estadística adecuada para la métrica de diagnósticos
Para las métricas de diagnóstico, puede elegir entre *Promedio*, *Mínimo*, *Máximo* y *Total* como métrica a partir de la que escalar. La estadística más común es *Promedio*.

### <a name="choose-the-thresholds-carefully-for-all-metric-types"></a>Elija los umbrales cuidadosamente para todos los tipos de métrica
Es recomendable tener cuidado a la hora de elegir los diferentes umbrales de escalado y reducción horizontal en función de las situaciones prácticas.

*No se recomiendan* opciones de escalado automático como las de los siguientes ejemplos, con valores de umbral iguales o similares en condiciones de escalado o reducción horizontal:

* Aumentar las instancias en 1 cuando el número de subprocesos >= 600
* Disminuir las instancias en 1 cuando el número de subprocesos <= 600

Veamos un ejemplo de lo que puede llevar a producir un comportamiento confuso. Considere la siguiente secuencia.

1. Imaginemos que hay dos instancias inicialmente y después aumenta el número promedio de subprocesos por instancia a 625.
2. La escalabilidad automática escala horizontalmente para agregar una tercera instancia.
3. Imaginemos ahora que el número promedio de subprocesos en la instancia se reduce a 575.
4. Antes de reducir verticalmente, el escalado automático intenta evaluar cuál será el estado final si reduce horizontalmente. Por ejemplo, 575 x 3 (número de instancias actual) = 1725/2 (número final de instancias al reducir verticalmente) = 862,5 subprocesos. Esto significa que el escalado automático tiene que volver a escalar horizontalmente de inmediato (incluso después de haber reducido horizontalmente) si el número promedio de subprocesos sigue siendo el mismo o incluso si se reduce solo una pequeña cantidad. Sin embargo, si se volviera a escalar verticalmente, todo el proceso se repetiría, dando lugar a un bucle infinito.
5. Para evitar esta situación (conocida como "inestable"), el escalado automático nunca reduce verticalmente. En su lugar, pasa esto por alto y vuelve a evaluar la situación la siguiente vez que el trabajo del servicio se ejecuta. El estado oscilante puede ser confuso para muchos usuarios, ya que puede dar la impresión de que la escalabilidad automática no funciona cuando el número promedio de subprocesos es 575.

La estimación durante una reducción horizontal está diseñada para evitar situaciones "oscilantes", donde las acciones de reducción horizontal y escalado horizontal avanzan y retroceden continuamente. Recuerde este comportamiento cuando elija los mismos umbrales de escalado horizontal y reducción horizontal.

Nuestra recomendación es establecer un margen suficiente entre el escalado horizontal y en los umbrales. Por ejemplo, echemos un vistazo a esta siguiente combinación de reglas, que es mejor.

* Aumentar las instancias en 1 cuando el porcentaje de CPU > = 80
* Disminuir las instancias en 1 cuando el porcentaje de CPU > = 60

En este caso  

1. Imaginemos que hay 2 instancias para empezar.
2. Si el promedio de porcentaje de CPU entre instancias llega a 80, el escalado automático escala horizontalmente agregando una tercera instancia.
3. Imaginemos ahora que, con el tiempo, el porcentaje de CPU cae a 60.
4. La regla de reducción horizontal del escalado automático calcula el estado final como si fuese a reducirse horizontalmente. Por ejemplo, 60 x 3 (número de instancias actual) = 180/2 (número final de instancias al reducir verticalmente) = 90. Por tanto, el escalado automático no reduce horizontalmente porque tendría que volver a escalar horizontalmente de inmediato. En su lugar, omite la reducción vertical.
5. La próxima vez que comprueba el escalado automático, la CPU ha seguido cayendo a 50. Vuelve a calcular: 50 x 3 instancias = 150/2 instancias = 75, lo que está por debajo del umbral de escalado horizontal de 80. Por tanto, reduce horizontalmente a 2 instancias.

### <a name="considerations-for-scaling-threshold-values-for-special-metrics"></a>Consideraciones para establecer valores de umbral de escalado en métricas especiales
 En el caso de las métricas especiales, como la métrica longitud de cola de Service Bus o de Storage, el umbral es el promedio de mensajes disponibles por número actual de instancias. Elija cuidadosamente el valor de umbral para esta métrica.

Veamos esto con un ejemplo para procurar que entienda el comportamiento de mejor forma.

* Aumentar las instancias en 1 cuando el número de mensajes de la cola de almacenamiento > = 50
* Disminuir las instancias en 1 cuando el número de mensajes de la cola de almacenamiento < = 10

Considere la siguiente secuencia:

1. Hay dos instancias de la cola de almacenamiento.
2. Siguen llegando mensajes y, al revisar la cola de almacenamiento, el recuento total es 50. Se supone que el escalado automático tendría que iniciar una acción de escalado horizontal. Sin embargo, vemos que sigue siendo de 50/2 = 25 mensajes por instancia. Por lo tanto, el escalado horizontal no se ha producido. Para que el primer escalado horizontal se produzca, el número total de mensajes en la cola de almacenamiento debe ser 100.
3. Tras esto, imaginemos que el número total de mensajes llega a 100.
4. Se agrega una tercera instancia de la cola de almacenamiento debido a una acción de escalado horizontal.  La siguiente acción de escalado horizontal no tendrá lugar hasta que el número total de mensajes en la cola llegue a 150 porque 150/3 = 50.
5. Ahora, disminuye el número de mensajes en la cola. Con tres instancias, la primera acción de reducción horizontal ocurre cuando el número total de mensajes en la cola llega a 30, porque 30/3 equivale a 10 mensajes por instancia, que es el umbral de reducción horizontal.

### <a name="considerations-for-scaling-when-multiple-profiles-are-configured-in-an-autoscale-setting"></a>Consideraciones de escalado cuando hay varios perfiles configurados en una configuración de escalado automático
En una configuración de escalado automático, puede elegir un perfil predeterminado, que se aplica siempre (independientemente de programaciones o períodos de tiempo), o bien optar por perfil periódico o por un perfil para un período fijo con una fecha y un intervalo de tiempo.

Cuando el servicio de escalado automático procesa estos perfiles, siempre los comprueba en el siguiente orden:

1. Perfil de fecha fija
2. Perfil periódico
3. Perfil predeterminado ("Siempre")

Si se cumple una condición de perfil, el escalado automático no comprobará la siguiente condición de perfil por debajo de este. El escalado automático solo procesa un perfil cada vez. Esto significa que, si queremos incluir una condición de procesamiento de un perfil de nivel inferior, tendremos que incluir también las reglas del perfil actual.

Vamos a analizarlo con un ejemplo:

La siguiente imagen muestra una configuración de escalado automático con un perfil predeterminado con un número de instancias mínimo = 2 y un número de instancias máximo = 10. En este ejemplo, las reglas están configuradas para escalar horizontalmente cuando el número de mensajes en la cola sea mayor que 10 y, asimismo, reducir horizontalmente cuando el número de mensajes en la cola sea inferior a tres. Ahora el recurso puede escalar entre dos y diez instancias.

Además, hay un perfil periódico establecido para el lunes. Está configurado para un número de instancias mínimo de tres y un número de instancias máximo de diez. Esto significa que, la primera vez que la escalabilidad automática compruebe esta condición el lunes, si el recuento de instancias es dos, lo escala al nuevo mínimo de tres. Mientras el escalado automático siga encontrando una coincidencia con esta condición de perfil (lunes), solo procesa las reglas de escalado/reducción horizontal basadas en CPU configuradas en este perfil. En este momento, no comprueba la longitud de la cola. Sin embargo, si queremos que compruebe la condición de longitud de cola, tendremos que incluir esas reglas del perfil predeterminado también en el perfil del lunes.

De forma similar, cuando el escalado automático regresa al perfil predeterminado, primero comprueba si se cumplen las condiciones mínima y máxima. Si el número de instancias en ese momento es 12, reducirá horizontalmente a 10, que es el máximo permitido en el perfil predeterminado.

![configuración de escalado automático](./media/autoscale-best-practices/insights-autoscale-best-practices-2.png)

### <a name="considerations-for-scaling-when-multiple-rules-are-configured-in-a-profile"></a>Consideraciones de escalado cuando hay varias reglas configuradas en un perfil

Hay casos en los que puede que sea necesario establecer varias reglas en un perfil. El motor de escalabilidad automática usa el siguiente conjunto de reglas de escalado automático cuando se establecen varias reglas.

Al *escalar horizontalmente*, el escalado automático se ejecuta si se cumple cualquier regla.
Al *reducir horizontalmente*, el escalado automático necesita que todas las reglas se cumplan.

Para ilustrar esto, imaginemos que tiene las siguientes cuatro reglas de escalabilidad automática:

* Con una CPU < 30 %, se reduce horizontalmente en 1
* Con una memoria < 50 %, se reduce horizontalmente en 1
* Con una CPU> 75 %, se escala horizontalmente en 1
* Con una memoria > 75 %, se escala horizontalmente en 1

Por tanto, sucederá lo siguiente:

* Con una CPU del 76 % y una memoria del 50 %, se escala horizontalmente.
* Con una CPU del 50 % y una memoria del 76 %, se escala horizontalmente.

Por otro lado, con una CPU del 25 % y una memoria del 51 %, el escalado automático **no** reduce horizontalmente. Para ello, la CPU debe ser 29 % y la memoria, 49 %.

### <a name="always-select-a-safe-default-instance-count"></a>Seleccione siempre un número predeterminado de instancias seguro
El número predeterminado de instancias es importante, porque el escalado automático escala el servicio a dicho número cuando no hay métricas disponibles. Por tanto, seleccione un número predeterminado de instancias que sea seguro para sus cargas de trabajo.

### <a name="configure-autoscale-notifications"></a>Configure notificaciones de escalado automático
El escalado automático se publicará en el registro de actividad si se produce alguna de las condiciones siguientes:

* El escalado automático genera una operación de escala.
* El servicio de escalado automático completa correctamente una acción de escalado.
* El servicio de escalado automático no puede realizar una acción de escalado.
* No hay métricas disponibles para que el servicio de escalado automático tome una decisión de escalado.
* Vuelve a haber métricas disponibles (recuperación) para poder tomar una decisión de escalado.

También puede usar una alerta de registro de actividades para supervisar el mantenimiento del motor de escalado automático. Aquí puede ver ejemplos para [crear una alerta de registro de actividades para supervisar todas las operaciones del motor de escalado automático en su suscripción](https://github.com/Azure/azure-quickstart-templates/tree/master/monitor-autoscale-alert) o para [crear una alerta de registro de actividades para supervisar todas las operaciones con errores de escalado automático y reducción horizontal en su suscripción](https://github.com/Azure/azure-quickstart-templates/tree/master/monitor-autoscale-failed-alert).

Además de utilizar alertas de registro de actividad, puede configurar notificaciones de correo electrónico o webhook para recibir una notificación cada vez que se lleve a cabo una acción de escalado correcta a través de la pestaña de notificaciones de la configuración de escalado automático.

## <a name="next-steps"></a>Pasos siguientes
- [Cree una alerta de registro de actividades para supervisar todas las operaciones del motor de escalado automático en su suscripción.](https://github.com/Azure/azure-quickstart-templates/tree/master/monitor-autoscale-alert)
- [Cree una alerta de registro de actividades para supervisar todas las operaciones con errores de escalado automático y reducción horizontal en su suscripción.](https://github.com/Azure/azure-quickstart-templates/tree/master/monitor-autoscale-failed-alert)

