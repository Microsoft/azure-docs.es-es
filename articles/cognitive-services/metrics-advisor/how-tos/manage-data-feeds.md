---
title: Administración de fuentes de distribución de datos en Metrics Advisor
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo administrar las fuentes de distribución de datos que ha agregado a Metrics Advisor.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 08/28/2020
ms.author: aahi
ms.openlocfilehash: 22a1be7231b730c83ca97b0f128e43258ed24533
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931984"
---
# <a name="how-to-manage-your-data-feeds"></a>Procedimientos: Administración de las fuentes de distribución de datos

Obtenga información sobre cómo administrar las fuentes de distribución de datos incorporadas a Metrics Advisor. En este artículo se explica paso a paso cómo administrar fuentes de distribución de datos en Metrics Monitor.

## <a name="edit-a-data-feed"></a>Edición de una fuente de distribución de datos

> [!NOTE]
> Los detalles siguientes no se pueden cambiar una vez creada una fuente de distribución de datos. 
> * Identificador de la fuente de distribución de datos
> * Hora de creación
> * Dimensión
> * Tipo de origen
> * Granularidad

Solo el administrador de una fuente de distribución de datos puede realizar cambios en ella. 

Para pausar o reactivar una fuente de distribución de datos, siga estos pasos:

1. En la página de lista de fuentes de distribución de datos, haga clic en la operación que desea realizar en la fuente de distribución de datos.

2. En la página de detalles de la fuente de distribución de datos, haga clic en el botón del interruptor **Status** (Estado).

Para eliminar una fuente de distribución de datos, siga estos pasos: 

1. En la página lista de fuentes de distribución de datos, haga clic en **Delete** (Eliminar), en la fuente de distribución de datos.

2. En la página de detalles de la fuente de distribución de datos, haga clic en **Delete** (Eliminar).

Al cambiar la hora de inicio, debe comprobar de nuevo el esquema. Puede cambiarla en **Edit parameters** (Editar parámetros).

##  <a name="backfill-your-data-feed"></a>Reposición de la fuente de distribución de datos

Seleccione el botón **Backfill** (Reposición) para desencadenar una ingesta inmediata en una marca de tiempo con el fin de corregir una ingesta errónea o invalidar los datos actuales.
- La hora de inicio es inclusiva.
- Y la hora de finalización es exclusiva.
- La detección de anomalías se vuelve a desencadenar solo en el intervalo seleccionado.

:::image type="content" source="../media/datafeeds/backfill-datafeed.png" alt-text="Reposición de la fuente de distribución de datos":::

## <a name="manage-permission-of-a-data-feed"></a>Administración de permisos de una fuente de distribución de datos

El acceso al área de trabajo se controla mediante el recurso de Metrics Advisor, que usa Azure Active Directory para la autenticación. Otra capa de control de permisos se aplica a los datos de métricas.

Metrics Advisor permite conceder permisos a distintos grupos de personas en fuentes de distribución de datos diferentes. Hay dos tipos de roles: 

- Administrador: Tiene permisos completos para administrar una fuente de distribución de datos, y también para modificarla y eliminarla.
- Visor: Tiene acceso a una vista de solo lectura de la fuente de distribución de datos.
 

## <a name="advanced-settings"></a>Configuración avanzada

Hay varios valores de configuración avanzada opcionales al crear una fuente de distribución de datos, que se pueden modificar en la página de detalles de la fuente de distribución de datos.

### <a name="ingestion-options"></a>Opciones de ingesta

* **Ingestion time offset** (Desfase en el tiempo de ingesta): De forma predeterminada, los datos se ingieren según la granularidad especificada. Por ejemplo, una métrica con una marca de tiempo *daily* (diaria) se ingerirá un día después de su marca de tiempo. Puede usar el desfase para retrasar el tiempo de ingesta con un número *positivo*, o bien adelantarlo con un número *negativo*.

* **Max concurrency** (Simultaneidad máxima): Establezca este parámetro si el origen de datos admite la simultaneidad limitada. De lo contrario, deje el valor predeterminado.

* **Stop retry after** (Detener reintento después de): Si se ha producido un error en la ingesta de datos, se volverá a intentar automáticamente dentro de un intervalo de tiempo. El comienzo del período es la hora a la que se produjo la primera ingesta de datos. La duración del período se define según la granularidad. Si deja el valor predeterminado (-1), el valor se determinará en función de la granularidad que se indica a continuación.
    
    | Granularidad       | Stop Retry After (Detener reintento después)           |
    | :------------ | :--------------- |
    | Daily (Diario), Custom (>= 1 Day) (Personalizado [(>=1 día]), Weekly (Semanal), Monthly (Mensual) y Yearly (Anual)     | 7 días          |
    | Hourly (Cada hora) y Custom (>= 1 Day) (Personalizado [(>=1 día])       | 72 horas |

* **Min retry interval** (Intervalo de reintento mínimo): Puede especificar el intervalo mínimo al reintentar la extracción de datos del origen. Si deja el valor predeterminado (-1), el intervalo de reintento se determinará en función de la granularidad que se indica a continuación.
    
    | Granularidad       | Intervalo mínimo de reintento           |
    | :------------ | :--------------- |
    | Daily (Diario), Custom (>= 1 Day) (Personalizado [(>=1 día]), Weekly (Semanal) y Monthly (Mensual)     | 30 minutos          |
    | Hourly (Cada hora) y Custom (>= 1 Day) (Personalizado [(>=1 día])      | 10 minutos |
    | Anual | 1 día          |
 
### <a name="fill-gap-when-detecting"></a>Fill gap when detecting (Llenar los espacios al detectar): 

> [!NOTE]
> Este valor no afectará al origen de datos ni a los gráficos de datos que se muestran en el portal. El rellenado automático solo se produce durante la detección de anomalías.

Algunas series temporales no son continuas. Cuando faltan puntos de datos, Metrics Advisor usará el valor especificado para rellenarlos antes de la detección de anomalías con el fin de mejorar la precisión.
Las opciones son: 

* Usar el valor del punto de datos real anterior. Se utiliza de forma predeterminada.
* Usar un valor específico.

### <a name="action-link-template"></a>Plantilla de vínculo de acción. 

Las plantillas de vínculo de acción se usan para predefinir direcciones URL HTTP accionables, que constan de los marcadores de posición `%datafeed`, `%metric`, `%timestamp`, `%detect_config`y `%tagset`. Puede usar la plantilla para redirigir de una anomalía o un incidente a una dirección URL específica para explorarla en profundidad.

:::image type="content" source="../media/action-link-template.png" alt-text="Plantilla de vínculo de acción" lightbox="../media/action-link-template.png":::

Una vez que haya rellenado el vínculo de acción, haga clic en **Go to action link** (Ir al vínculo de acción) en la opción de acción de la lista de incidentes y en el menú contextual del árbol de incidentes. Reemplace los marcadores de posición de la plantilla de vínculo de acción con los valores correspondientes de la anomalía o el incidente.

| Marcador de posición | Ejemplos | Comentario |
| ---------- | -------- | ------- |
| `%datafeed` | - | Identificador de la fuente de distribución de datos |
| `%metric` | - | Identificador de la métrica |
| `%detect_config` | - | Identificador de la configuración de detección |
| `%timestamp` | - | Marca de tiempo de una anomalía o una hora de finalización de un incidente persistente |
| `%tagset` | `%tagset`, <br> `[%tagset.get("Dim1")]`, <br> `[ %tagset.get("Dim1", "filterVal")]` | Valores de dimensión de una anomalía o anomalía principal de un incidente.   <br> `filterVal` se utiliza para filtrar los valores coincidentes dentro de los corchetes.   |

Ejemplos.

* Si la plantilla de vínculo de acción es `https://action-link/metric/%metric?detectConfigId=%detect_config`:
  * El vínculo de acción `https://action-link/metric/1234?detectConfigId=2345` redirigiría a las anomalías o incidentes correspondientes a la métrica `1234` y la configuración de detección `2345`.

* Si la plantilla de vínculo de acción es `https://action-link?[Dim1=%tagset.get('Dim1','')&][Dim2=%tagset.get('Dim2','')]`: 
    * El vínculo de acción sería `https://action-link?Dim1=Val1&Dim2=Val2` cuando la anomalía es `{ "Dim1": "Val1", "Dim2": "Val2" }`. 
    * El vínculo de acción sería `https://action-link?Dim2=Val2` cuando la anomalía es `{ "Dim1": "", "Dim2": "Val2" } `, ya que `[Dim1=***&]` se omite para la cadena vacía del valor de dimensión. 

* Si la plantilla de vínculo de acción es `https://action-link?filter=[Name/Dim1 eq '%tagset.get('Dim1','')' and ][Name/Dim2 eq '%tagset.get('Dim2','')']`: 
    * El vínculo de acción sería `https://action-link?filter=Name/Dim1 eq 'Val1' and Name/Dim2 eq 'Val2'` cuando la anomalía es `{ "Dim1": "Val1", "Dim2": "Val2" }`. 
    * El vínculo de acción sería `https://action-link?filter=Name/Dim2 eq 'Val2'` cuando la anomalía es `{ "Dim1": "", "Dim2": "Val2" }`, ya que `[Name/Dim1 eq '***' and ]` se omite para la cadena vacía del valor de dimensión. 
   
### <a name="data-feed-not-available-alert-settings"></a>Configuración de alertas de fuentes de distribución de datos no disponibles

Una fuente de distribución de datos se considera como no disponible si no se ingiere ningún dato del origen dentro del período de gracia especificado desde el momento en que se inicia la ingesta de la fuente de distribución de datos. En este caso, se desencadena una alerta.

Para configurar una alerta, debe [crear primero un enlace](alerts.md#create-a-hook). Las alertas se enviarán a través del enlace configurado.

* **Grace period** (Período de gracia): El valor Grace period (Período de gracia) se usa para determinar cuándo se debe enviar una alerta si no se ingieren puntos de datos. El punto de referencia es la hora de la primera ingesta. Si se produce un error en la ingesta, Metrics Advisor seguirá intentándolo en un intervalo normal especificado por la granularidad. Si sigue produciéndose un error después del período de gracia, se enviará una alerta.

* **Auto snooze** (Posponer automáticamente): Cuando esta opción se establece en cero, cada marca de tiempo con *Not Available* (No disponible) desencadenará una alerta. Cuando se especifica un valor distinto de cero, las marcas de tiempo continuas después de la primera marca de tiempo con *Not Available* (No disponible) no desencadenan alertas según la configuración especificada.

## <a name="next-steps"></a>Pasos siguientes
- [Configuración de métricas y ajuste de la configuración de la detección](configure-metrics.md)
- [Ajuste de la detección de anomalías mediante comentarios](anomaly-feedback.md)
- [Diagnóstico de incidentes](diagnose-incident.md)
