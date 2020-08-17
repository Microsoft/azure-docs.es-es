---
title: 'Registros: Hiperescala (Citus) en Azure Database for PostgreSQL'
description: 'Procedimiento para acceder a los registros de base de datos para Azure Database for PostgreSQL: Hiperescala (Citus)'
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 7/13/2020
ms.openlocfilehash: 4ca64a7793d229e7910fb122fb33996b1ac0c4fe
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095358"
---
# <a name="logs-in-azure-database-for-postgresql---hyperscale-citus"></a>Registros en Hiperescala (Citus) para Azure Database for PostgreSQL

Los registros de PostgreSQL están disponibles en cada nodo de un grupo de servidores de Hiperescala (Citus). Puede enviar registros a un servidor de almacenamiento o a un servicio de análisis. Los registros se pueden usar para identificar, solucionar y reparar errores de configuración y casos de rendimiento no óptimo.

## <a name="accessing-logs"></a>Acceso a los registros

Para acceder a los registros de PostgreSQL para un nodo de trabajo o coordinador de Hiperescala (Citus), abra el nodo en Azure Portal:

![lista de nodos](media/howto-hyperscale-logging/choose-node.png)

En el nodo seleccionado, abra **Configuración de diagnóstico** y haga clic en **+ Agregar configuración de diagnóstico**.

![Botón Agregar configuración de diagnóstico](media/howto-hyperscale-logging/diagnostic-settings.png)

Elija un nombre para la nueva configuración de diagnóstico y active la casilla **PostgreSQLLogs**.  Elija el o los destinos que deben recibir los registros.

![Elección de los registros de PostgreSQL](media/howto-hyperscale-logging/diagnostic-create-setting.png)

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a las consultas de Log Analytics](/azure/azure-monitor/log-query/get-started-portal)
- Información acerca de [Azure Event Hubs](/azure/event-hubs/event-hubs-about)
