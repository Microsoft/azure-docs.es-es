---
title: Supervisar RU/s normalizadas para un contenedor de Azure Cosmos o una cuenta
description: Obtenga información sobre cómo supervisar el uso de unidades de solicitud normalizadas de una operación en Azure Cosmos DB. Los propietarios de una cuenta de Azure Cosmos DB pueden conocer qué operaciones consumen más unidades de solicitud.
ms.service: cosmos-db
ms.topic: how-to
author: kanshiG
ms.author: govindk
ms.date: 06/25/2020
ms.openlocfilehash: e7005a3786bb2d538450b076c113e159c766d72e
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88642085"
---
# <a name="how-to-monitor-normalized-rus-for-an-azure-cosmos-container-or-an-account"></a>Cómo supervisar RU/s normalizadas para un contenedor de Azure Cosmos o una cuenta

Azure Monitor para Azure Cosmos DB proporciona una vista de métricas para supervisar la cuenta y crear paneles. Las métricas de Azure Cosmos DB se recopilan de forma predeterminada, esta característica no requiere que habilite ni configure nada explícitamente.

La métrica de **Consumo de RU normalizado** se usa para ver la saturación de los intervalos de claves de partición con respecto al tráfico. Azure Cosmos DB distribuye el rendimiento de forma uniforme entre todos los intervalos de claves de partición. Esta métrica proporciona una vista por segundo del uso máximo de rendimiento dentro para el intervalo de claves de partición. Use esta métrica para calcular el uso de RU/s entre los intervalos de claves de partición de un contenedor determinado. Al usar esta métrica, si ve un porcentaje elevado del uso de unidades de solicitud entre todos los intervalos de claves de partición en Azure Monitor, debe aumentar el rendimiento para satisfacer las necesidades de la carga de trabajo. 

## <a name="what-to-expect-and-do-when-normalized-rus-is-higher"></a>Qué esperar y hacer cuando el consumo de RU/s normalizado es mayor

Cuando el consumo de RU normalizado alcanza el 100 % para el intervalo de claves de partición determinado, y si un cliente sigue realizando solicitudes en esa ventana de tiempo de 1 segundo a ese intervalo de claves de partición específico, recibe un error de velocidad limitada. El cliente debe respetar el tiempo de espera sugerido y volver a intentar la solicitud. El SDK facilita la administración de esta situación al reintentar las horas preconfiguradas mediante una espera adecuada.  No es necesario que vea el error de velocidad limitada de RU solo porque la RU normalizada ha alcanzado el 100 %. Esto se debe a que la RU normalizada es un valor único que representa el uso máximo de todos los intervalos de claves de partición; un intervalo de claves de partición puede estar ocupado, pero los demás intervalos de claves de partición pueden atender las solicitudes sin problemas. Por ejemplo, una sola operación, como un procedimiento almacenado que consume todas las RU/s en un intervalo de claves de partición, dará lugar a un breve aumento en el consumo de RU/s normalizado. En tales casos, no habrá errores de velocidad limitada inmediatos si la velocidad de solicitudes es baja o se realizan solicitudes a otras particiones a intervalos de claves de partición distintos. 

Las métricas de Azure Monitor ayudan a encontrar las operaciones por código de estado para SQL API mediante el uso de la métrica de **Total de solicitudes**. Posteriormente, puede filtrar estas solicitudes por el código de estado 429 y dividirlas por **Tipo de operación**.  

Para buscar las solicitudes que tienen una velocidad limitada, el método recomendado es obtener esta información a través de registros de diagnóstico.

Si hay un pico continuo del consumo de RU/s normalizado del 100 % o cercano al 100 % en varios intervalos de claves de partición, se recomienda aumentar el rendimiento. Puede averiguar qué operaciones son de consumo intenso y su uso máximo al usar las métricas de Azure Monitor y los registros de diagnósticos de Azure Monitor.

En resumen, la métrica de **Consumo de RU normalizado** se usa para ver cuál es el intervalo de claves de partición más eficiente en términos de uso. Por tanto, permite sesgar el rendimiento hacia un intervalo de claves de partición. Después, puede consultar el registro **PartitionKeyRUConsumption** en los registros de Azure Monitor para obtener información acerca de las claves de partición lógicas que están activas en términos de uso. Esto apuntará a un cambio ya sea en la opción de clave de partición o a un cambio en la lógica de aplicación. Para resolver la limitación de velocidad, distribuya la carga de datos en varias particiones o simplemente aumente el rendimiento, ya que es realmente necesario. 



## <a name="view-the-normalized-request-unit-consumption-metric"></a>Ver la métrica de consumo normalizado de unidades de solicitud

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

2. Seleccione **Monitor** en la barra de navegación izquierda y, a continuación, seleccione **Métricas**.

   :::image type="content" source="./media/monitor-normalized-request-units/monitor-metrics-blade.png" alt-text="Panel Métricas en Azure Monitor":::

3. En el panel **Métricas** panel > **Seleccionar un recurso** > elija la **suscripción**, y el **grupo de recursos** requeridos. En **Tipo de recurso**, seleccione **Cuentas de Azure Cosmos DB**, elija una de las cuentas de Azure Cosmos existentes y seleccione **Aplicar**.

   :::image type="content" source="./media/monitor-normalized-request-units/select-cosmos-db-account.png" alt-text="Elección de una cuenta de Azure Cosmos DB para ver las métricas":::

4. A continuación, puede seleccionar una métrica de la lista de métricas disponibles. Puede seleccionar métricas específicas de unidades de solicitud, almacenamiento, latencia, disponibilidad, Cassandra, etc. Para conocer los detalles de todas las métricas disponibles en esta lista, consulte el artículo [Métricas por categoría](monitor-cosmos-db-reference.md). En este ejemplo, vamos a seleccionar la métrica de **Consumo de RU normalizado** y **Max** como valor de agregación.

   Además de estos detalles, también puede seleccionar los valores de **Intervalo de tiempo** y **Granularidad de tiempo** de las métricas. Como máximo, puede ver las métricas de los últimos 30 días.  Después de aplicar el filtro, se muestra un gráfico basado en dicho filtro.

   :::image type="content" source="./media/monitor-normalized-request-units/normalized-request-unit-usage-metric.png" alt-text="Elección de una métrica en Azure Portal":::

### <a name="filters-for-normalized-request-unit-consumption"></a>Filtros para el consumo normalizado de unidades de solicitud

También puede filtrar las métricas y el gráfico que se muestra por una propiedad **CollectionName**, **DatabaseName**, **PartitionKeyRangeID** y **Region** concreta. Para filtrar las métricas, seleccione **Agregar filtro** y elija la propiedad necesaria, como **CollectionName** y el valor correspondiente que le interese. A continuación, el gráfico muestra las unidades de consumo de RU normalizado utilizadas para el contenedor durante el período seleccionado.  

Para agrupar las métricas puede usar la opción **Apply splitting** (Aplicar división).  

La métrica de consumo normalizado de unidades de solicitud para cada contenedor aparece como se muestra en la siguiente imagen:

:::image type="content" source="./media/monitor-normalized-request-units/normalized-request-unit-usage-filters.png" alt-text="Aplicar filtros a la métrica de consumo normalizado de unidades de solicitud":::

## <a name="next-steps"></a>Pasos siguientes

* Supervise los datos de Azure Cosmos DB mediante la [configuración de diagnóstico](cosmosdb-monitor-resource-logs.md) en Azure.
* [Auditoría de las operaciones del plano de control de Azure Cosmos DB](audit-control-plane-logs.md)
