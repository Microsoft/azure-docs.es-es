---
title: Configuración de la instancia de Metrics Advisor mediante el portal web
titleSuffix: Azure Cognitive Services
description: Cómo configurar la instancia de Metrics Advisor y ajustar los resultados de la detección de anomalías.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 09/10/2020
ms.author: aahi
ms.openlocfilehash: 80e6ffb79aae5ffc0fe1fd8c9d73d97cc3bdde1e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932043"
---
# <a name="how-to-configure-metrics-and-fine-tune-detecting-configuration"></a>Procedimientos: Configuración de métricas y ajuste de la configuración de la detección

Use este artículo para empezar a configurar la instancia de Metrics Advisor mediante el portal web. Para examinar las métricas de una fuente de distribución de datos específica, vaya a la página **Fuentes de distribución de datos** y seleccione una de las fuentes de distribución. Se mostrará una lista de las métricas asociadas a la misma.

:::image type="content" source="../media/metrics/select-metric.png" alt-text="Selección de una métrica" lightbox="../media/metrics/select-metric.png":::

Haga clic en uno de los nombres de métrica para ver sus detalles. En esta vista detallada, puede cambiar a otra métrica de la misma fuente de distribución de datos mediante la lista desplegable en la esquina superior derecha de la pantalla.

La primera vez que vea los detalles de las métricas, podrá cargar una serie temporal permitiendo a Metrics Advisor elegir una por usted o especificando los valores que se van a incluir para cada dimensión. 

También puede seleccionar intervalos de tiempo y cambiar el diseño de la página.

> [!NOTE]
> - La hora de inicio es inclusiva.
> - Y la hora de finalización es exclusiva. 

Puede hacer clic en la pestaña **Incidentes** para ver las anomalías y encontrar un vínculo al [Centro de incidentes](diagnose-incident.md).

## <a name="tune-the-detecting-configuration"></a>Ajuste de la configuración de detección

Una métrica puede aplicar una o varias configuraciones de detección. Existe una configuración de detección para cada métrica, que puede editar o agregarle, según sus necesidades de supervisión.

### <a name="tune-the-configuration-for-all-series-in-current-metric"></a>Ajuste la configuración de todas las series de la métrica actual

Esta configuración se aplicará a todas las series de esta métrica, excepto las que tengan una configuración independiente. Una configuración de nivel de métrica se aplica de forma predeterminada cuando se incorporan los datos, y se muestra en el panel izquierdo. Los usuarios pueden editar la configuración de nivel de métrica directamente en la página Métrica. 

Existen parámetros adicionales como **Dirección** y **Anomalía válida** que se pueden usar para ajustar aún más la configuración. También puede combinar diferentes métodos de detección. 

:::image type="content" source="../media/configuration-combination.png" alt-text="Combinación de configuraciones" lightbox="../media/configuration-combination.png":::

### <a name="tune-the-configuration-for-a-specific-series-or-group"></a>Ajuste de la configuración de una serie o grupo específicos

Haga clic en **Configuración avanzada** bajo las opciones de configuración de nivel de métrica para ver la configuración de nivel de grupo. Puede agregar una configuración de una serie individual o un grupo de series haciendo clic en el icono **+** de esta ventana. Los parámetros son similares a los parámetros de configuración de nivel de métrica, pero es posible que tenga que especificar al menos un valor de dimensión de una configuración de nivel de grupo para identificar un grupo de series. Especifique, además, todos los valores de dimensión de una configuración de nivel de serie para identificar una serie específica. 

Esta configuración se aplicará al grupo de series o series específicas en lugar de la configuración de nivel de métrica. Después de establecer las condiciones para este grupo, guárdelo.

:::image type="content" source="../media/advanced-configuration.png" alt-text="Configuración avanzada" lightbox="../media/advanced-configuration.png":::

### <a name="anomaly-detection-methods"></a>Métodos de detección de anomalías

Metrics Advisor ofrece varios métodos de detección de anomalías. Puede usar uno o combinarlos mediante operadores lógicos haciendo clic en el botón **+** . 

**Detección inteligente** 

La detección inteligente está basada en el aprendizaje automático, que aprende patrones de datos históricos y los usa para la detección futura. Al usar este método, la **sensibilidad** es el parámetro más importante para optimizar los resultados de la detección. Puede arrastrarlo a un valor menor o mayor para que afecte a la visualización en el lado derecho de la página. Elija uno que se ajuste a sus datos y guárdelo. 


En el modo de detección inteligente, los parámetros de versión de límite y sensibilidad se usan para ajustar el resultado de la detección de anomalías.

La sensibilidad puede afectar al ancho del intervalo de valores esperado de cada punto. Cuando aumente, el intervalo de valores esperado será más estricto y se notificarán más anomalías:

:::image type="content" source="../media/metrics/smart-detection-high-sensitivity.png" alt-text="Detección inteligente con alta sensibilidad":::

Cuando se reduzca la sensibilidad, el intervalo de valores esperado será más amplio y se notificarán menos anomalías:

:::image type="content" source="../media/metrics/smart-detection-low-sensitivity.png" alt-text="Detección inteligente con baja sensibilidad":::

**Umbral de cambio** 

El umbral de cambio se suele usar cuando los datos de métricas permanecen con carácter general en torno a un intervalo determinado. El umbral se establece según el **porcentaje de cambio**. El modo **Umbral de cambio** puede detectar anomalías en los escenarios:

* Los datos suelen ser estables y fluidos. Desea recibir una notificación cuando hay fluctuaciones.
* Los datos suelen ser bastante inestables y fluctúan mucho. Desea recibir una notificación cuando son demasiado estables o planos.

Siga estos pasos para usar dicho modo:

1. Seleccione **Umbral de cambio** como su método de detección de anomalías cuando establezca las configuraciones de detección de anomalías para las métricas o series temporales.
    
    :::image type="content" source="../media/metrics/change-threshold.png" alt-text="umbral de cambio":::

2. Seleccione el parámetro **fuera del intervalo** o **en el intervalo** en función del escenario.

    Si desea detectar fluctuaciones, seleccione **fuera del intervalo**. Por ejemplo, con la configuración siguiente, cualquier punto de datos que cambie más de un 10 % en comparación con el anterior se detectará como un valor atípico.
    :::image type="content" source="../media/metrics/out-of-the-range.png" alt-text="parámetro fuera del intervalo":::

    Si desea detectar líneas planas en los datos, seleccione **en el intervalo**. Por ejemplo, con la configuración siguiente, cualquier punto de datos que cambie dentro del 0,01 % en comparación con el anterior se detectará como un valor atípico. Como el umbral es tan pequeño (0,01 %), detecta líneas planas en los datos como valores atípicos.

    :::image type="content" source="../media/metrics/in-the-range.png" alt-text="Parámetro en el intervalo":::

3. Establezca el porcentaje de cambio que contará como anomalía y los puntos de datos capturados anteriormente que se usarán para la comparación. Esta comparación siempre se encuentra entre el punto de datos actual y un solo punto de datos N se señala antes que este.
    
    **Dirección** solo es válido si usa el modo **fuera del intervalo**:
    
    * **Arriba** configura la detección para detectar solo anomalías cuando (punto de datos actual) - (punto de datos de comparación) > **+** umbral%.
    * **Abajo** configura la detección para detectar solo anomalías cuando (punto de datos actual) - (punto de datos de comparación) < **-** umbral%.
 
**Umbral saturado**

 El umbral saturado es un método básico para la detección de anomalías. Puede establecer un límite superior o inferior para determinar el intervalo de valores esperado. Cualquier punto que salga del límite se identificará como anomalía. 


## <a name="preset-events"></a>Eventos preestablecidos

En ocasiones, los eventos y sucesos esperados (como vacaciones) pueden generar datos anómalos. Mediante el uso de eventos preestablecidos, puede agregar marcas al resultado de la detección de anomalías, durante las horas especificadas. Esta característica debe configurarse después de incorporarse la fuente de distribución de datos. Cada métrica solo puede tener una configuración de eventos preestablecidos.

> [!Note]
> La configuración de eventos preestablecidos tendrá en cuenta las vacaciones durante la detección de anomalías y puede cambiar los resultados. Se aplicará a los puntos de datos ingeridos después de guardar la configuración. 

Haga clic en el botón **Configure Preset Event** (Configurar evento preestablecido) situado junto a la lista desplegable de métricas en cada página de detalles de métrica.
 
:::image type="content" source="../media/metrics/preset-event-button.png" alt-text="Botón Evento preestablecido":::

En la ventana que aparece, configure las opciones según su uso. Asegúrese de que se ha seleccionado **Enable holiday event** (Habilitar evento festivo) para usar la configuración. 

La sección **Evento festivo** le ayuda a suprimir anomalías innecesarias detectadas durante las vacaciones. Existen dos opciones para la opción **Estrategia** que puede aplicar:

* **Suppress holiday** (Suprimir vacaciones): suprime todas las anomalías y alertas en los resultados de la detección de anomalías durante el período de vacaciones.
* **Holiday as weekend** (Vacaciones como fin de semana): Calcula los valores esperados medios de varios fines de semana correspondientes antes de las vacaciones y basa el estado de la anomalía en estos valores.

Existen algunos otros valores que puede configurar:

|Opción  |Descripción  |
|---------|---------|
|**Choose one dimension as country** (Elegir una dimensión como país)     | Elija una dimensión que contenga información sobre el país. Por ejemplo, un código de país.         |
|**Country code mapping** (Asignación de código de país)     | Asignación entre un [código de país](https://wikipedia.org/wiki/ISO_3166-1_alpha-2) estándar y los datos del país de la dimensión elegida.        |
|**Holiday options** (Opciones de vacaciones)    | Si se deben tener en cuenta todas las vacaciones, solo las correspondientes a tiempo de permiso retribuido o solo aquellas que no corresponden a tiempo de permiso retribuido.         |
|**Days to expand** (Días para expandir)    |  Días que se ven afectados antes o después de unas vacaciones.        |


La sección **Evento de ciclo** puede utilizarse en algunos escenarios para ayudar a reducir alertas innecesarias mediante el uso de patrones cíclicos en los datos. Por ejemplo: 

- Métricas que tienen varios patrones o ciclos, como un patrón semanal y mensual. 
- Métricas que no tienen un patrón claro, pero cuyos datos son comparables año tras año, mes tras mes, semana tras semana o día tras día.
 
No todas las opciones se pueden seleccionar para cada granularidad. A continuación se indican las opciones disponibles por granularidad:

| Granularidad | Año tras año | Mes tras mes | Semana tras semana | DoD |
|:-|:-|:-|:-|:-|
| Anual | X | X | X | X |
| Mensual | X | X | X | X |
| Cada semana | ✔ | X | X | X |
| Diario | ✔ | ✔ | ✔ | X |
| Cada hora | ✔ | ✔ | ✔ | ✔ |
| Cada minuto | X | X | X | X |
| Cada segundo | X | X | X | X |
| Personalizado* | ✔ | ✔ | ✔ | ✔ |

X: no disponible.  
✔: disponible.
  
\* Al usar una granularidad personalizada en segundos, solo disponible si la métrica tiene más de una hora y menos de un día.

El evento de ciclo se usa para reducir las anomalías si siguen un patrón cíclico, pero notificará una anomalía si varios puntos de datos no siguen el patrón. El **modo strict** se usa para habilitar la notificación de anomalías si incluso un punto de datos no sigue el patrón. 

:::image type="content" source="../media/metrics/preset-events.png" alt-text="configuración de eventos preestablecidos":::

## <a name="view-recent-incidents"></a>Ver incidentes recientes

Metrics Advisor detecta anomalías en todos los datos de serie temporal a medida que se ingieren. Sin embargo, no es necesario escalar todas las anomalías, ya que es posible que no tengan un gran impacto. La agregación se realizará en anomalías para agrupar las relacionadas en incidentes. Puede ver estos incidentes en la pestaña **Incidente** de la página de detalles de métricas. 

Haga clic en un incidente para ir a la página **Análisis de incidentes**, en la cual podrá ver más detalles al respecto. Haga clic en **Manage incidents in new Incident hub** (Administrar incidentes en el nuevo Centro de incidentes) para buscar la página [Centro de incidentes](diagnose-incident.md) donde encontrará todos los incidentes en la métrica específica. 

## <a name="subscribe-anomalies-for-notification"></a>Suscripción a anomalías para su notificación

Si desea recibir una notificación cada vez que se detecte una anomalía, puede suscribirse a las alertas de la métrica mediante un enlace. Consulte [Configuración de alertas y obtención de notificaciones mediante un enlace](alerts.md) para obtener más información.


## <a name="next-steps"></a>Pasos siguientes 
- [Configuración de alertas y obtención de notificaciones mediante un enlace](alerts.md)
- [Ajuste de la detección de anomalías mediante los comentarios](anomaly-feedback.md)
- [Diagnóstico de incidentes](diagnose-incident.md)

