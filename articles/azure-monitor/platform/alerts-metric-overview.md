---
title: Comprender cómo funcionan las alertas de métricas en Azure Monitor.
description: Obtenga información general acerca de lo que puede hacer con las alertas de métricas y cómo funcionan en Azure Monitor.
ms.date: 07/16/2020
ms.topic: conceptual
ms.subservice: alerts
ms.openlocfilehash: 9ac034eb144e5873f8f73ab7b1d7459394ce4775
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327147"
---
# <a name="understand-how-metric-alerts-work-in-azure-monitor"></a>Comprender cómo funcionan las alertas de métricas en Azure Monitor

Las alertas de métricas en Azure Monitor funcionan en la parte superior de las métricas multidimensionales. Estas métricas pueden ser [métricas de plataforma](alerts-metric-near-real-time.md#metrics-and-dimensions-supported), [métricas personalizadas](./metrics-custom-overview.md), [registros populares de Azure Monitor convertidos en métricas](./alerts-metric-logs.md) y métricas de Application Insights. Las alertas de métricas se evalúan a intervalos regulares para comprobar si las condiciones de una o varias series temporales para las métricas son verdaderas y recibirá una notificación cuando se cumplan las evaluaciones. Como las alertas de métricas tienen un estado, solo envían notificaciones cuando cambia ese estado.

## <a name="how-do-metric-alerts-work"></a>Cómo funcionan las alertas de métricas

Para definir una regla de alerta de métrica, especifique el recurso de destino que se supervisará, el nombre, la condición de la métrica, el tipo de condición (estático o dinámico) y la condición (un operador y un umbral o sensibilidad) y un grupo de acción que se activará cuando a su vez se active la regla de alerta. Los tipos de condición afectan a la manera en que se determinan los umbrales. [Más información sobre las opciones de tipo y sensibilidad de la condición de umbrales dinámicos](alerts-dynamic-thresholds.md).

### <a name="alert-rule-with-static-condition-type"></a>Regla de alerta con el tipo de condición estática

Supongamos que ha creado una regla de alerta de métrica de umbral estático simple de la siguiente manera:

- Recurso de objetivo (el recurso de Azure que quiere supervisar): myVM
- Métrica: Porcentaje de CPU
- Tipo de condición: estática
- Agregación de tiempo (estadística que se ejecuta sobre valores de métrica sin procesar. [Las agregaciones de tiempo compatibles](metrics-charts.md#changing-aggregation) son Min, Max, Avg, Total y Count): Average
- Período (la ventana temporal según la cual se comprueban los valores de la métrica): En los últimos 5 minutos
- Frecuencia (es decir, la frecuencia con la que la alerta de métricas comprueba si se cumplen las condiciones): 1 min
- Operador: Mayor que
- Umbral: 70

Desde el momento en que se crea la regla de alerta, el proceso de supervisión se ejecuta cada minuto y examina los valores métricos de los últimos 5 minutos; además, comprueba si el promedio de esos valores es superior a 70. Si se cumple la condición, es decir, si el porcentaje medio de la CPU supera el valor de 70 durante los últimos 5 minutos, la regla de alerta desencadena una notificación activada. Si configuró un correo electrónico o una acción de webhook en el grupo de acciones asociado a la regla de alerta, recibirá una notificación activada en ambos recursos.

Si utiliza varias condiciones en una regla, la regla une las condiciones. Es decir, una alerta se desencadena cuando todas las condiciones de regla la alerta se evalúan como true y se resuelve cuando una de las condiciones deja de ser true. Un ejemplo de este tipo de regla de alerta sería supervisar una máquina virtual de Azure y generar una alerta si el porcentaje de CPU supera el 90 % y la longitud de la cola es de más de 300 elementos.

### <a name="alert-rule-with-dynamic-condition-type"></a>Regla de alerta con el tipo de condición dinámica

Supongamos que ha creado una regla de alerta de métrica simple de umbrales dinámicos de la siguiente manera:

- Recurso de objetivo (el recurso de Azure que quiere supervisar): myVM
- Métrica: Porcentaje de CPU
- Tipo de condición: Dinámica
- Agregación de tiempo (estadística que se ejecuta sobre valores de métrica sin procesar. [Las agregaciones de tiempo compatibles](metrics-charts.md#changing-aggregation) son Min, Max, Avg, Total y Count): Average
- Período (la ventana temporal según la cual se comprueban los valores de la métrica): En los últimos 5 minutos
- Frecuencia (es decir, la frecuencia con la que la alerta de métricas comprueba si se cumplen las condiciones): 1 min
- Operador: Mayor que
- Sensibilidad: Media
- Períodos de retroceso: 4
- Número de infracciones: 4

Una vez creada la regla de alerta, el algoritmo de aprendizaje automático de los umbrales dinámicos adquirirá los datos históricos disponibles, calculará el umbral que mejor se adapte al patrón de comportamiento de las series de métricas y aprenderá continuamente en función de los nuevos datos para que el umbral sea más preciso.

Desde el momento en que se crea la regla de alerta, el monitor se ejecuta cada minuto y examina los valores de métrica de los últimos 20 minutos agrupados en períodos de cinco minutos y comprueba si el promedio de los valores del período en cada uno de los cuatro períodos supera el umbral esperado. Si se cumple la condición, es decir, el porcentaje promedio de CPU en los últimos 20 minutos (cuatro períodos de cinco minutos) desviado cuatro veces del comportamiento esperado, la regla de alerta desencadena una notificación activada. Si configuró un correo electrónico o una acción de webhook en el grupo de acciones asociado a la regla de alerta, recibirá una notificación activada en ambos recursos.

### <a name="view-and-resolution-of-fired-alerts"></a>Visualización y resolución de alertas desencadenadas

Los ejemplos anteriores de activación de regla de alerta también se pueden ver en Azure Portal, en la hoja **Todas las alertas**.

Digamos que el uso en "myVM" continúa estando por encima del umbral en las comprobaciones posteriores; por lo tanto, la regla de alerta no volverá a activarse hasta que se resuelvan las condiciones.

Después de algún tiempo, si el uso en "myVM" vuelve a ser normal y a estar por debajo del umbral. Debido a ello, la regla de alerta supervisa la condición dos veces más para enviar una notificación resuelta. La regla de alerta envía un mensaje resuelto o desactivado cuando la condición de alerta no se cumple durante tres períodos consecutivos, para así reducir el ruido en caso de que haya una oscilación en las condiciones.

Cuando la notificación resuelta se envíe a través del webhook o del correo electrónico, el estado de la instancia de alerta (llamada estado de supervisión) de Azure Portal también se establecerá como Resuelta.

### <a name="using-dimensions"></a>Usar las dimensiones

Las alertas de métricas en Azure Monitor también admiten la supervisión de las combinaciones de valores de varias dimensiones con una regla. Aquí le explicaremos por qué debería usar combinaciones de varias dimensiones con la ayuda de un ejemplo.

Supongamos que tiene un plan de App Service para su sitio web. Quiere supervisar el uso de la CPU en varias instancias de la aplicación o el sitio web. Puede hacerlo mediante una regla de alerta de métrica, tal como se indica a continuación:

- Recurso de destino: myAppServicePlan
- Métrica: Porcentaje de CPU
- Tipo de condición: estática
- Dimensions
  - Instance = InstanceName1, InstanceName2
- Agregación de tiempo: Average
- Período: En los últimos 5 minutos
- Frecuencia: 1 min
- Operador: GreaterThan
- Umbral: 70

Igual que antes, esta regla supervisa si el uso medio de la CPU durante los últimos 5 minutos supera el 70 %. Sin embargo, con la misma regla puede supervisar dos instancias de su sitio web. Cada instancia será supervisada individualmente y, por consiguiente, recibirá notificaciones de forma individual.

Supongamos que tiene una aplicación web con una demanda masiva y es necesario agregar más instancias. La regla anterior únicamente supervisa dos instancias. Sin embargo, puede crear una regla de la siguiente manera:

- Recurso de destino: myAppServicePlan
- Métrica: Porcentaje de CPU
- Tipo de condición: estática
- Dimensions
  - Instancia: *
- Agregación de tiempo: Average
- Período: En los últimos 5 minutos
- Frecuencia: 1 min
- Operador: GreaterThan
- Umbral: 70

Esta regla supervisará automáticamente todos los valores de la instancia, es decir, puede supervisar sus instancias a medida que aparecen sin necesidad de modificar su regla de alerta de métrica nuevamente.

Al supervisar varias dimensiones, las reglas de alertas de umbrales dinámicos pueden crear umbrales personalizados para cientos de series de métricas a la vez. Los umbrales dinámicos dan como resultado menos reglas de alertas que administrar y un ahorro de tiempo significativo en la administración y creación de reglas de alertas.

Supongamos que tiene una aplicación web con muchas instancias y no sabe cuál es el umbral más adecuado. Las reglas anteriores siempre utilizarán el umbral del 70 %. Sin embargo, puede crear una regla de la siguiente manera:

- Recurso de destino: myAppServicePlan
- Métrica: Porcentaje de CPU
- Tipo de condición: Dinámica
- Dimensions
  - Instancia: *
- Agregación de tiempo: Average
- Período: En los últimos 5 minutos
- Frecuencia: 1 min
- Operador: GreaterThan
- Sensibilidad: Media
- Períodos de retroceso: 1
- Número de infracciones: 1

Esta regla supervisa si el uso medio de la CPU durante los últimos cinco minutos supera el comportamiento esperado para cada instancia. La misma regla que puede supervisar las instancias a medida que aparecen sin necesidad de volver a modificar la regla de alerta de métrica. Cada instancia obtendrá un umbral que se ajuste al patrón de comportamiento de las series de métricas y cambiará continuamente en función de los nuevos datos para hacer que el umbral sea más preciso. Igual que antes, cada instancia se supervisará individualmente y, por consiguiente, recibirá notificaciones de forma individual.

El aumento de los períodos de retroceso y del número de infracciones también puede permitir que las alertas de filtrado solo alerten sobre la definición de una desviación significativa. [Más información sobre las opciones avanzadas de umbrales dinámicos](alerts-dynamic-thresholds.md#what-do-the-advanced-settings-in-dynamic-thresholds-mean).

> [!NOTE]
>
> Se recomienda elegir una *Granularidad de agregación (período)* mayor que la *Frecuencia de evaluación*, con el fin de reducir la probabilidad de que falte la primera evaluación de las series temporales agregadas en los casos siguientes:
> - Regla de alertas de métricas que supervisa varias dimensiones: Cuando se agrega una nueva combinación de valores de dimensión.
> - Regla de alertas de métricas que supervisa varios recursos: Cuando se agrega un nuevo recurso al ámbito.
> - Regla de alertas de métricas que supervisa una métrica que no se emite de manera continua (métrica dispersa): Cuando la métrica se emite después de un período de más de 24 horas en el que no se emitió.



## <a name="monitoring-at-scale-using-metric-alerts-in-azure-monitor"></a>Supervisión a escala mediante alertas de métricas en Azure Monitor

Hasta ahora, ha visto cómo se puede usar una única alerta de métrica para supervisar una o varias series temporales de métricas relacionadas con un único recurso de Azure. Con frecuencia, es posible que desee aplicar la misma regla de alertas a muchos recursos. Azure Monitor también permite supervisar varios recursos (del mismo tipo) utilizando una sola regla de alertas de métricas con los recursos que se encuentran en la misma región de Azure. 

Esta característica se admite actualmente en las métricas de plataforma (no métricas personalizadas) de los siguientes servicios en las siguientes nubes de Azure:

| Servicio | Public Azure (Azure público) | Government | China |
|:--------|:--------|:--------|:--------|
| Máquinas virtuales  | **Sí** | No | No |
| Bases de datos de SQL Server | **Sí** | **Sí** | No |
| Grupos elásticos de SQL Server | **Sí** | **Sí** | No |
| Dispositivos Data Box Edge | **Sí** | **Sí** | No |

El ámbito de supervisión se puede especificar con una sola alerta de métrica de tres formas distintas. Por ejemplo, respecto a las máquinas virtuales, el ámbito se puede especificar como:  

- una lista de máquinas virtuales (de una región de Azure) en una suscripción
- todas las máquinas virtuales (de una región de Azure) en uno o varios grupos de recursos de una suscripción
- todas las máquinas virtuales (de una región de Azure) en una suscripción

> [!NOTE]
>
> El ámbito de una regla de alerta de métrica de varios recursos debe contener al menos un recurso del tipo de recurso seleccionado.

La creación de reglas de alertas de métrica que supervisen varios recursos es similar a [crear cualquier otra alerta de métrica](alerts-metric.md) que supervise un único recurso. La única diferencia es que debe seleccionar todos los recursos que desea supervisar. Estas reglas también se pueden crear mediante las [plantillas de Azure Resource Manager](./alerts-metric-create-templates.md#template-for-a-metric-alert-that-monitors-multiple-resources). Recibirá notificaciones diferentes de cada máquina virtual.

> [!NOTE]
>
> En una regla de alerta de métrica que supervisa varios recursos solo se permite una condición.

## <a name="typical-latency"></a>Latencia típica

En cuanto a las alertas de métricas, normalmente recibirá una notificación en menos de 5 minutos si configura la frecuencia de la regla de alerta en "1 min". En caso de que haya cargas pesadas en los sistemas de notificaciones, es posible que vea una latencia más larga.

## <a name="supported-resource-types-for-metric-alerts"></a>Tipos de recursos admitidos en las alertas métricas

En este [artículo](./alerts-metric-near-real-time.md#metrics-and-dimensions-supported) puede encontrar la lista completa de tipos de recursos admitidos.


## <a name="next-steps"></a>Pasos siguientes

- [Aprenda a crear, ver y administrar las alertas de métricas en Azure](alerts-metric.md)
- [Aprenda a implementar alertas de métricas con plantillas de Azure Resource Manager](./alerts-metric-create-templates.md)
- [Obtenga más información sobre los grupos de acciones](action-groups.md)
- [Más información sobre el tipo de condición de umbrales dinámicos](alerts-dynamic-thresholds.md)

