---
title: Información de rendimiento de consultas en Azure Database for MySQL
description: En este artículo se describe la característica Información de rendimiento de consultas en Azure Database for MySQL.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: conceptual
ms.date: 5/12/2020
ms.openlocfilehash: 7aefa6cd048c41feb1535f2ae7fa5c87bf628b2d
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90884900"
---
# <a name="query-performance-insight-in-azure-database-for-mysql"></a>Información de rendimiento de consultas en Azure Database for MySQL

**Se aplica a:** Azure Database for MySQL 5.7, 8.0

Query Performance Insight le ayuda a identificar rápidamente cuáles son las consultas que más tardan en ejecutarse, cómo cambian con el tiempo y qué esperas están afectándoles.

## <a name="common-scenarios"></a>Escenarios frecuentes

### <a name="long-running-queries"></a>Consultas de larga duración

- Identificar las consultas de ejecución más largas en las últimas X horas
- Identificar las N principales consultas que esperan recursos
 
### <a name="wait-statistics"></a>Estadísticas de espera

- Descripción de la naturaleza de la espera de una consulta
- Descripción de las tendencias de las esperas de recursos y dónde existe la contención de recursos

## <a name="permissions"></a>Permisos

Se necesitan los permisos **Propietario** o **Colaborador** para ver el texto de las consultas en Query Performance Insight. **Lector** pueden ver las tablas y los gráficos, pero no el texto de consulta.

## <a name="prerequisites"></a>Requisitos previos

Para que Query Performance Insight funcione, deben existir datos en el [Almacén de consultas](concepts-query-store.md).

## <a name="viewing-performance-insights"></a>Ver información de rendimiento

La vista [Query Performance Insight](concepts-query-performance-insight.md) en Azure Portal detectará visualizaciones en la información de clave del Almacén de consultas.

En la página del portal de su servidor de Azure Database for MySQL, seleccione **Información de rendimiento de consultas** en la sección **Rendimiento inteligente** de la barra de menús.

### <a name="long-running-queries"></a>Consultas de larga duración

La pestaña **Consultas de larga duración** muestra las cinco principales consultas por promedio de duración por ejecución, que se agregan en intervalos de 15 minutos. Para ver más consultas, selecciónelas en la lista desplegable **Número de consultas**. Al hacerlo, los colores del gráfico pueden cambiar a un identificador de consulta específico.

Puede hacer clic y arrastrar en el gráfico para restringir a un período de tiempo específico. Como alternativa, puede usar los iconos de acercar y alejar para ver un período de tiempo mayor o menor, respectivamente.

:::image type="content" source="./media/concepts-query-performance-insight/query-performance-insight-landing-page.png" alt-text="Consultas de larga ejecución de Query Performance Insight"::: 

### <a name="wait-statistics"></a>Estadísticas de espera

> [!NOTE]
> Las estadísticas de espera están diseñadas para solucionar problemas con el rendimiento de una consulta. Se recomienda activarlas solo para solucionar problemas. <br>Si recibe el mensaje de error en Azure Portal "*Se produjo un problema en "Microsoft.DBforMySQL"; no se puede realizar la solicitud. Si el problema continua o no se lo esperaba, póngase en contacto con el servicio de soporte técnico y facilíteles esta información.* " durante la visualización de las estadísticas de espera, use un período de tiempo menor.

Las estadísticas de espera brindan una vista de los eventos de espera que se producen durante la ejecución de una consulta específica. Puede encontrar más información sobre los tipos de eventos de espera en la [documentación del motor de MySQL](https://go.microsoft.com/fwlink/?linkid=2098206).

Seleccione la pestaña **Estadísticas de espera** para ver las visualizaciones correspondientes a esperas en el servidor.

Las consultas que se muestran en la vista de estadísticas de espera se agrupan según las consultas que presentan las mayores esperas durante el intervalo de tiempo especificado.

:::image type="content" source="./media/concepts-query-performance-insight/query-performance-insight-wait-statistics.png" alt-text="Estadísticas de espera de Información de rendimiento de consultas":::

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre la [supervisión y el ajuste](concepts-monitoring.md) en Azure Database for MySQL.