---
title: 'Registros: Hiperescala (Citus) en Azure Database for PostgreSQL'
description: 'Procedimiento para acceder a los registros de base de datos para Azure Database for PostgreSQL: Hiperescala (Citus)'
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 7/13/2020
ms.openlocfilehash: 1dc7bc8e119de7c8fdcf09713286be2633457486
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90895866"
---
# <a name="logs-in-azure-database-for-postgresql---hyperscale-citus"></a>Registros en Hiperescala (Citus) para Azure Database for PostgreSQL

Los registros de PostgreSQL están disponibles en cada nodo de un grupo de servidores de Hiperescala (Citus). Puede enviar registros a un servidor de almacenamiento o a un servicio de análisis. Los registros se pueden usar para identificar, solucionar y reparar errores de configuración y casos de rendimiento no óptimo.

## <a name="accessing-logs"></a>Acceso a los registros

Para acceder a los registros de PostgreSQL para un nodo de trabajo o coordinador de Hiperescala (Citus), abra el nodo en Azure Portal:

:::image type="content" source="media/howto-hyperscale-logging/choose-node.png" alt-text="lista de nodos":::

En el nodo seleccionado, abra **Configuración de diagnóstico** y haga clic en **+ Agregar configuración de diagnóstico**.

:::image type="content" source="media/howto-hyperscale-logging/diagnostic-settings.png" alt-text="Botón Agregar configuración de diagnóstico":::

Elija un nombre para la nueva configuración de diagnóstico y active la casilla **PostgreSQLLogs**.  Elija el o los destinos que deben recibir los registros.

:::image type="content" source="media/howto-hyperscale-logging/diagnostic-create-setting.png" alt-text="Elección de los registros de PostgreSQL":::

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a las consultas de Log Analytics](/azure/azure-monitor/log-query/get-started-portal)
- Información acerca de [Azure Event Hubs](/azure/event-hubs/event-hubs-about)
