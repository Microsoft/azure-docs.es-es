---
title: Supervisión de varias series temporales en una sola regla de alerta de métrica
description: Alerta a escala mediante una sola regla de alerta para varias series temporales
author: harelbr
ms.author: harelbr
ms.topic: conceptual
ms.date: 08/11/2020
ms.subservice: alerts
ms.openlocfilehash: f7ca91ca49d9357285e1307c5051ef5685ad24c9
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88186814"
---
# <a name="monitor-multiple-time-series-in-a-single-metric-alert-rule"></a>Supervisión de varias series temporales en una sola regla de alerta de métrica

Es posible utilizar una sola regla de alerta de métrica para supervisar una o varias series temporales de métricas, de modo que la supervisión de los recursos a escala resulte más sencilla.

## <a name="metric-time-series"></a>Serie temporal de métricas

Una serie temporal de métricas es una serie de medidas (o "valores de métricas") capturadas durante un período de tiempo. 

Por ejemplo:

- El uso de CPU de una máquina virtual.
- Los bytes entrantes (entrada) a una cuenta de almacenamiento.
- El número de solicitudes con error de una aplicación web



## <a name="alert-rule-on-a-single-time-series"></a>Regla de alerta en una sola serie temporal
Una regla de alerta supervisa una sola serie temporal cuando cumple todas las condiciones siguientes:
-   La regla supervisa un único recurso de destino. 
-   Contiene una única condición.
-   Evalúa una métrica sin elegir las dimensiones (suponiendo que la métrica admita dimensiones).

Un ejemplo de este tipo de regla de alerta (mostrando solo las propiedades pertinentes):
-   Recurso de destino: *myVM1*
-   Métrica: *Porcentaje de la CPU*
-   Operador: *Mayor que*
-   Umbral: *70*


En esta regla de alerta, se supervisa una sola serie temporal de métricas:
-   Porcentaje de CPU donde *Recurso*="myVM1" > 70 %

![Una regla de alerta en una sola serie temporal](media/alerts-metric-multiple-time-series-single-rule/simple-alert-rule.png)

## <a name="alert-rule-on-multiple-time-series"></a>Regla de alerta en varias series temporales
Una regla de alerta supervisa varias series temporales si usa al menos una de las siguientes características: 
-   Varios recursos
-   Varias condiciones 
-   Varias dimensiones


## <a name="multiple-resources-multi-resource"></a>Varios recursos

Una sola regla de alerta de métrica puede supervisar varios recursos, siempre que los recursos sean del mismo tipo y existan en la misma región de Azure. El uso de este tipo de regla reduce la complejidad y el número total de reglas de alerta que se deben mantener. 

Un ejemplo de este tipo de regla de alerta:
-   Recurso de destino: *myVM1, myVM2*
-   Métrica: *Porcentaje de la CPU*
-   Operador: *Mayor que*
-   Umbral: *70*

En esta regla de alerta, dos series temporales de métricas se supervisan por separado:
-   Porcentaje de CPU donde *Recurso*="myVM1" > 70 %
-   Porcentaje de CPU donde *Recurso*="myVM2" > 70 %

![Una regla de alerta de varios recursos](media/alerts-metric-multiple-time-series-single-rule/multi-resource-alert-rule.png)
 
En una regla de alerta de varios recursos, la condición se evalúa **por separado** para cada uno de los recursos (o, con más precisión, para cada una de las series temporales de métricas que corresponden a cada recurso). Esto significa que las alertas también se activan para cada recurso por separado.

Por ejemplo, supongamos que hemos establecido la regla de alerta anterior para supervisar la CPU por encima del 70 %. En el período de tiempo evaluado (es decir, los últimos 5 minutos):
-   El *porcentaje de CPU* de *myVM1* es superior al 70 %. 
-   El *porcentaje de CPU* de *myVM2* es del 50 %.

La regla de alerta se desencadena en *myVM1*, pero no en *myVM2*. Estas alertas desencadenadas son independientes. También pueden resolverse en momentos diferentes según el comportamiento individual de cada una de las máquinas virtuales.

Para obtener más información sobre las reglas de alertas de varios recursos y los tipos de recursos compatibles con esta funcionalidad, consulte [Supervisión a escala mediante alertas de métricas en Azure Monitor](alerts-metric-overview.md#monitoring-at-scale-using-metric-alerts-in-azure-monitor).

> [!NOTE] 
> En una regla de alerta de métrica que supervisa varios recursos solo se permite una condición.

## <a name="multiple-conditions-multi-condition"></a>Varias condiciones

Una sola regla de alertas de métrica también puede supervisar hasta cinco condiciones por regla de alerta. 

Por ejemplo:

- Recurso de destino: *myVM1*
- Condition1
  - Métrica: *Porcentaje de la CPU*
  - Operador: *Mayor que*
  - Umbral: *70*
- Condición2
  - Métrica: *Red entrante total*
  - Operador: *Mayor que*
  - Umbral: *20 MB*

Para esta regla de alerta, se están supervisando dos series temporales de métricas:

- Porcentaje de CPU donde *Recurso*="myVM1" > 70 %
- Red entrante total donde *Recurso*="myVM1" > 20 MB

![Una regla de alerta de varias condiciones](media/alerts-metric-multiple-time-series-single-rule/multi-condition-alert-rule.png)
 
Se usa un operador "AND" entre las condiciones. La regla de alerta activa una alerta cuando se cumplen **todas** las condiciones. La alerta desencadenada se resuelve si ya no se cumple al menos una de las condiciones. 

> [!NOTE]
> Existen restricciones al usar dimensiones en una regla de alerta con varias condiciones. Para obtener más información, consulte [Restricciones al usar dimensiones en una regla de alertas de métricas con varias condiciones](alerts-troubleshoot-metric.md#restrictions-when-using-dimensions-in-a-metric-alert-rule-with-multiple-conditions).


## <a name="multiple-dimensions-multi-dimension"></a>Varias dimensiones

Una sola regla de alerta de métrica también puede supervisar varios valores de dimensión de una métrica. Las dimensiones de una métrica son pares nombre-valor que transportan datos adicionales para describir el valor de la métrica. Por ejemplo, la métrica **Transacciones** de una cuenta de almacenamiento tiene una dimensión denominada **Nombre de API** que describe el nombre de la API a la que llama cada transacción (por ejemplo, GetBlob, DeleteBlob, PutPage). El uso de dimensiones es opcional, pero permite filtrar la métrica y supervisar solo series temporales específicas, en lugar de supervisar la métrica como un agregado de todos los valores de dimensión colocados juntos. 

Por ejemplo, puede elegir que se desencadene una alerta cuando el número de transacciones sea elevado en todos los nombres de API (que son los datos agregados) o bien dividirla para que se produzca una alerta solo cuando el número de transacciones sea elevado para los nombres de API específicos. 

Ejemplo de una regla de alerta que supervisa varias dimensiones:

- Recurso de destino: *myStorage1*
- Métrica: *Transactions*
- Dimensions
  * Nombre de API = *GetBlob, DeleteBlob, PutPage*
- Operador: *Mayor que*
- Umbral: *70*

Para esta regla de alerta, se están supervisando tres series temporales de métricas:

- Transacciones donde *Recurso*="myStorage1" y *Nombre de la API*="GetBlob" > 70
- Transacciones donde *Recurso*="myStorage1" y *Nombre de la API*="DeleteBlob" > 70
- Transacciones donde *Recurso*="myStorage1" y *Nombre de la API*="PutPage" > 70

![Una regla de alerta de varias dimensiones con valores de una dimensión](media/alerts-metric-multiple-time-series-single-rule/multi-dimension-1.png)

Una regla de alertas de métricas de varias dimensiones también puede supervisar varios valores de dimensión de **diferentes** dimensiones de una métrica. En este caso, la regla de alerta supervisa **de forma independiente** todas las combinaciones de valores de dimensión de la selección.

Ejemplo de este tipo de alerta:

- Recurso de destino: *myStorage1*
- Métrica: *Transactions*
- Dimensions
  * Nombre de API = *GetBlob, DeleteBlob, PutPage*
  * Autenticación = *SAS, AccountKey*
- Operador: *Mayor que*
- Umbral: *70*

En esta regla de alerta, seis series temporales de métricas se supervisan por separado:

- Transacciones donde *Recurso*="myStorage1", *Nombre de la API*="GetBlob"y *Autenticación*="SAS" > 70
- Transacciones donde *Recurso*="myStorage1", *Nombre de la API*="GetBlob" y *Autenticación*="AccountKey" > 70
- Transacciones donde *Recurso*="myStorage1", *Nombre de la API*="DeleteBlob"y *Autenticación*="SAS" > 70
- Transacciones donde *Recurso*="myStorage1", *Nombre de la API*="DeleteBlob" y *Autenticación*="AccountKey" > 70
- Transacciones donde *Recurso*="myStorage1", *Nombre de la API*="PutPage" y *Autenticación*="SAS" > 70
- Transacciones donde *Recurso*="myStorage1", *Nombre de la API*="PutPage" y *Autenticación*="AccountKey" > 70

![Una regla de alerta de varias dimensiones con valores de varias dimensiones](media/alerts-metric-multiple-time-series-single-rule/multi-dimension-2.png)
 
### <a name="advanced-multi-dimension-features"></a>Características avanzadas de varias dimensiones

1.  **Seleccionar todas las dimensiones actuales y futuras**: puede optar por supervisar todos los valores posibles de una dimensión, incluidos los valores futuros. Este tipo de regla de alerta se escalará automáticamente para supervisar todos los valores de la dimensión sin necesidad de modificar la regla de alerta cada vez que se agregue o se quite un valor de dimensión.
2.  **Excluir dimensiones**: la selección del operador "≠" (excluir) para un valor de dimensión equivale a seleccionar todos los demás valores de esa dimensión, incluidos los valores futuros.
3.  **Dimensiones nuevas y personalizadas**: los valores de dimensión que se muestran en Azure Portal se basan en los datos de métrica recopilados en los últimos tres días. Si aún no se ha emitido el valor de dimensión que está buscando, puede agregar un valor de dimensión personalizada.

![Características avanzadas de varias dimensiones](media/alerts-metric-multiple-time-series-single-rule/advanced-features.png)


## <a name="metric-alerts-pricing"></a>Precios de alertas de métricas

Los precios de las reglas de alertas de métricas están disponibles en la [página de precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

Al crear una regla de alertas de métrica, la estimación de precios proporcionada se basa en las características seleccionadas y el número de series temporales supervisadas, que se determina a partir de la configuración de reglas y los valores de métricas actuales. Sin embargo, el cargo mensual se basa en las evaluaciones reales de la serie temporal y, por tanto, puede diferir de la estimación original si alguna serie temporal no tiene datos para evaluar, o si la regla de alerta utiliza características que pueden hacer que se escale dinámicamente.

Por ejemplo, una regla de alerta puede mostrar una estimación de precios elevada si aprovecha la característica de varias dimensiones y hay un gran número de combinaciones de valores de dimensión seleccionados, lo que da lugar a la supervisión de muchas series temporales. Sin embargo, el cargo real de esa regla de alerta puede ser menor si no todas las series temporales resultantes de las combinaciones de valores de dimensión tienen realmente datos para evaluar.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la supervisión a escala mediante alertas de métricas y [umbrales dinámicos](alerts-dynamic-thresholds.md).
