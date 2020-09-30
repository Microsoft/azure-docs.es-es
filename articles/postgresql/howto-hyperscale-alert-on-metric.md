---
title: 'Configuración de alertas en Azure Database for PostgreSQL: Hiperescala (Citus)'
description: Este artículo describe cómo configurar y obtener acceso a las alertas métricas de Azure Database for PostgreSQL - Hiperescala (Citus)
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 3/16/2020
ms.openlocfilehash: b5ccd1281e50ca10b8edd7d7567a000b97107901
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907436"
---
# <a name="use-the-azure-portal-to-set-up-alerts-on-metrics-for-azure-database-for-postgresql---hyperscale-citus"></a>Use Azure Portal para configurar alertas métricas en Azure Database for PostgreSQL - Hiperescala (Citus)

En este artículo se explica cómo configurar alertas de Azure Database for PostgreSQL mediante Azure Portal. Puede recibir una alerta basada en las [métricas de supervisión](concepts-hyperscale-monitoring.md) para los servicios de Azure.

Configuraremos una alerta que se active cuando el valor de una métrica específica cruce un umbral. La alerta se activa cuando se cumple la condición por primera vez y continúa después.

Puede configurar una alerta para realizar las siguientes acciones cuando se desencadene:
* Envía notificaciones por correo electrónico al administrador del servicio y a los coadministradores.
* Enviar un correo electrónico a direcciones de correo electrónico adicionales que especifique.
* Llamar a un webhook.

Puede obtener información sobre las reglas de alerta y configurarlas mediante:
* [Azure Portal](../azure-monitor/platform/alerts-metric.md#create-with-azure-portal)
* [CLI de Azure](../azure-monitor/platform/alerts-metric.md#with-azure-cli)
* [API de REST de Azure Monitor](https://docs.microsoft.com/rest/api/monitor/metricalerts)

## <a name="create-an-alert-rule-on-a-metric-from-the-azure-portal"></a>Creación de una regla de alerta sobre una métrica desde Azure Portal
1. En [Azure Portal](https://portal.azure.com/), seleccione el servidor de Azure Database for PostgreSQL que quiera supervisar.

2. En la sección **Supervisión** de la barra lateral, seleccione **Alertas**, tal y como se muestra a continuación:

   :::image type="content" source="./media/howto-hyperscale-alert-on-metric/2-alert-rules.png" alt-text="Selección de Reglas de alerta":::

3. Seleccione **Nueva regla de alertas** (icono +).

4. Se abre la página **Crear regla**, tal y como se muestra a continuación. Rellene la información necesaria:

   :::image type="content" source="./media/howto-hyperscale-alert-on-metric/4-add-rule-form.png" alt-text="Formulario de adición de alerta de métrica":::

5. En la sección de **Condición**, seleccione **Agregar**.

6. Seleccione una métrica de la lista de señales sobre las que desea recibir alertas. En este ejemplo, seleccione "Porcentaje de almacenamiento".
   
   :::image type="content" source="./media/howto-hyperscale-alert-on-metric/6-configure-signal-logic.png" alt-text="Selección de la métrica":::

7. Configure la lógica de alertas:

    * **Operador** (ej. "Mayor que")
    * **Valor umbral** (ej. 85 por ciento)
    * **Granularidad de agregación** cantidad de tiempo en la que se debe satisfacer la regla métrica antes de que la alerta se desencadene (ej. "En los últimos 30 minutos")
    * y **Frecuencia de evaluación** (ej. "1 minuto")
   
   Seleccione **Listo** cuando haya terminado.

   :::image type="content" source="./media/howto-hyperscale-alert-on-metric/7-set-threshold-time.png" alt-text="Selección de la métrica":::

8. En la sección **Grupos de acciones**, seleccione **Crear nuevo** para crear un grupo en el que recibir las notificaciones sobre la alerta.

9. Rellene el formulario "Agregar grupo de acciones" con un nombre, un nombre corto, la suscripción y el grupo de recursos.

    :::image type="content" source="./media/howto-hyperscale-alert-on-metric/9-add-action-group.png" alt-text="Grupo de acción":::

10. Configure el tipo de acción **Correo electrónico/SMS/Push/Voz**.
    
    Elija "Rol de Azure Resource Manager de correo electrónico" para enviar notificaciones a los suscriptores, contribuidores y lectores.
   
    Cuando haya terminado, seleccione **Aceptar**.

    :::image type="content" source="./media/howto-hyperscale-alert-on-metric/10-action-group-type.png" alt-text="Grupo de acciones":::

11. Especifique el nombre de la regla de alertas, la descripción y la gravedad.

    :::image type="content" source="./media/howto-hyperscale-alert-on-metric/11-name-description-severity.png" alt-text="Grupo de acciones"::: 

12. Seleccione **Crear regla de alerta** para crear la alerta.

    En cuestión de minutos, se activa la alerta y se desencadena tal como se describió anteriormente.

### <a name="managing-alerts"></a>Administración de alertas

Una vez que haya creado una alerta, puede seleccionarla y realizar las siguientes acciones:

* Ver un gráfico que muestre el umbral de las métricas y los valores reales del día anterior en relación con esta alerta.
* **Editar** o **eliminar** la regla de alerta.
* **Deshabilitar** o **habilitar** la alerta, si quiere detener temporalmente o reanudar la recepción de notificaciones.

## <a name="suggested-alerts"></a>Alertas sugeridas

### <a name="disk-space"></a>Espacio en disco

La supervisión y las alertas son importantes para cada grupo de servidores de Hiperescala (Citus) de producción. La base de datos PostgreSQL subyacente requiere espacio libre en disco para funcionar correctamente. Si el disco se llena, el nodo de servidor de bases de datos se desconectará y no se podrá iniciar hasta que haya espacio disponible. En ese momento, requiere una solicitud de soporte técnico de Microsoft para corregir la situación.

Se recomienda establecer alertas de espacio en disco en cada nodo de cada grupo de servidores, incluso para usos de no producción. Las alertas de uso de espacio en disco proporcionan la advertencia de avance necesaria para intervenir y mantener los nodos en buen estado. Para obtener los mejores resultados, pruebe una serie de alertas con un uso del 75 %, 85 % y 95 %. Los porcentajes que se deben elegir dependen de la velocidad de la ingesta de datos, ya que la ingesta de datos rápida llena antes el disco.

A medida que el disco se aproxime al límite de espacio, pruebe estas técnicas para tener más espacio libre:

* Revise la directiva de retención de datos. Mueva los datos más antiguos al almacenamiento en frío, si es factible.
* Considere la posibilidad de [agregar nodos](howto-hyperscale-scaling.md#add-worker-nodes) al grupo de servidores y reequilibrar las particiones. El reequilibrio distribuye los datos entre más equipos.
* Considere la posibilidad de [aumentar la capacidad](howto-hyperscale-scaling.md#increase-or-decrease-vcores-on-nodes) de los nodos de trabajo. Cada trabajo puede tener hasta 2 TiB de almacenamiento. Sin embargo, se deben intentar agregar nodos antes de cambiar el tamaño de los nodos, ya que la adición de nodos se completa más rápidamente.

### <a name="cpu-usage"></a>Uso de CPU

La supervisión del uso de CPU resulta útil para establecer una base de referencia para el rendimiento. Por ejemplo, puede observar que el uso de CPU suele ser de, aproximadamente, entre 40-60 %. Si, de repente, el uso de CPU empieza a rondar el 95 %, puede reconocer una anomalía. El uso de CPU puede reflejar el crecimiento orgánico, pero también puede revelar una consulta aislada. Al crear una alerta de CPU, defina una granularidad de agregación larga para capturar los aumentos prolongados e ignorar los picos momentáneos.

## <a name="next-steps"></a>Pasos siguientes
* Obtenga más información sobre cómo [configurar webhooks en las alertas](../azure-monitor/platform/alerts-webhooks.md).
* Obtenga [información general sobre la colección de métricas](../monitoring-and-diagnostics/insights-how-to-customize-monitoring.md) para garantizar que el servicio está disponible y que responder adecuadamente.
