---
title: 'Optimización de la recopilación de estadísticas de consulta en Azure Database for PostgreSQL: servidor único'
description: En este artículo se describe cómo puede optimizar la recopilación de estadísticas de consulta en Azure Database for PostgreSQL con un único servidor.
author: dianaputnam
ms.author: dianas
ms.service: postgresql
ms.topic: how-to
ms.date: 5/6/2019
ms.openlocfilehash: bc731f6f6a5a60bce0851bf8fe5874f7149f3899
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90901455"
---
# <a name="optimize-query-statistics-collection-on-an-azure-database-for-postgresql---single-server"></a>Optimización de la recopilación de estadísticas de consulta en Azure Database for PostgreSQL con un único servidor
En este artículo se describe cómo optimizar la recopilación de estadísticas de consulta en un servidor de Azure Database for PostgreSQL.

## <a name="use-pg_stats_statements"></a>Uso de pg_stats_statements
**Pg_stat_statements** es una extensión de PostgreSQL habilitada de manera predeterminada en Azure Database for PostgreSQL. La extensión proporciona un medio para realizar el seguimiento de las estadísticas de ejecución para todas las instrucciones SQL ejecutadas por un servidor. Este módulo enlaza con cada ejecución de la consulta y genera un costo de rendimiento no trivial. Habilitar **pg_stat_statements** fuerza las escrituras de texto de consulta en archivos en disco.

Si tiene consultas únicas con texto de consulta largo o no supervisa activamente **pg_stat_statements**, deshabilite **pg_stat_statements** para obtener un mejor rendimiento. Para hacerlo, cambie la configuración a `pg_stat_statements.track = NONE`.

En algunas cargas de trabajo del cliente se ha visto una mejora del rendimiento de hasta un 50 por ciento al deshabilitar **pg_stat_statements**. La concesión que se hace al deshabilitar pg_stat_statements es la incapacidad de solucionar problemas de rendimiento.

Para establecer `pg_stat_statements.track = NONE`:

- En Azure Portal, vaya a la [página de administración de PostgreSQL y seleccione la hoja de parámetros del servidor](howto-configure-server-parameters-using-portal.md).

  :::image type="content" source="./media/howto-optimize-query-stats-collection/pg_stats_statements_portal.png" alt-text="Hoja de parámetros del servidor de PostgreSQL":::

- Con la [CLI de Azure](howto-configure-server-parameters-using-cli.md), establezca la configuración de az postgres server en `--name pg_stat_statements.track --resource-group myresourcegroup --server mydemoserver --value NONE`.

## <a name="use-the-query-store"></a>Uso del Almacén de consultas 
La característica [Almacén de consultas](concepts-query-store.md) de Azure Database for PostgreSQL proporciona una manera más eficaz de realizar el seguimiento de las estadísticas de consulta. Se recomienda esta característica como alternativa al uso de *pg_stats_statements*. 

## <a name="next-steps"></a>Pasos siguientes
Considere la posibilidad de establecer `pg_stat_statements.track = NONE` en [Azure Portal](howto-configure-server-parameters-using-portal.md) o con la [CLI de Azure](howto-configure-server-parameters-using-cli.md).

Para más información, consulte: 
- [Escenarios de uso del Almacén de consultas](concepts-query-store-scenarios.md) 
- [Procedimientos recomendados sobre Almacén de consultas](concepts-query-store-best-practices.md) 
