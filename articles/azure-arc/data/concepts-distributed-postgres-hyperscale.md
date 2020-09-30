---
title: Conceptos para la distribución de datos y el escalado horizontal del grupo de servidores de Hiperescala de PostgreSQL habilitada para Arc
titleSuffix: Azure Arc enabled data services
description: Conceptos para la distribución de datos del grupo de servidores de Hiperescala de PostgreSQL habilitada para Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: c01da4aed9e27296ea7b570420bb190b16749848
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932353"
---
# <a name="concepts-for-distributing-data-with-arc-enabled-postgresql-hyperscale-server-group"></a>Conceptos para la distribución de datos del grupo de servidores de Hiperescala de PostgreSQL habilitada para Arc

En este artículo se explican los conceptos principales que son importantes para sacar el máximo partido de Hiperescala de PostgreSQL habilitada para Azure Arc.
Los siguientes artículos vinculados señalan los conceptos explicados para Hiperescala (Citus) de Azure Database for PostgreSQL. Se trata de la misma tecnología de Hiperescala de PostgreSQL habilitada para Azure Arc, por lo que se aplican los mismos conceptos y perspectivas.

**¿Cuál es la diferencia entre ellos?**
- _Hiperescala (Citus) de Azure Database for PostgreSQL_

Este es el factor de forma de hiperescala del motor de base de datos de Postgres disponible como base de datos como servicio en Azure (PaaS). Está basado en la extensión Citus, que habilita la experiencia de Hiperescala. En este factor de forma, el servicio se ejecuta en los centros de datos de Microsoft y es operado por Microsoft.

- _Hiperescala de PostgreSQL habilitada para Azure Arc_

Este es el factor de forma de hiperescala del motor de base de datos de Postgres disponible con los servicios de datos habilitados para Azure Arc. En este factor de forma, los clientes proporcionan la infraestructura que hospeda los sistemas y los operan.

A continuación se resumen los conceptos principales relacionados con Hiperescala de PostgreSQL habilitada para Azure Arc:

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="nodes-and-tables"></a>Nodos y tablas
Es importante conocer los siguientes conceptos para aprovechar al máximo las ventajas de Hiperescala de PostgreSQL habilitada para Azure Arc:
- Nodos de Postgres especializados en Hiperescala de PostgreSQL habilitada para Azure Arc: nodo de coordinación y nodos de trabajo
- Tipos de tablas: tablas distribuidas, tablas de referencia y tablas locales
- Particiones de base de datos

Consulte más información en [Nodos y tablas de Azure Database for PostgreSQL: Hiperescala (Citus)](../../postgresql/concepts-hyperscale-nodes.md). 

## <a name="determine-the-application-type"></a>Determinación del tipo de aplicación
Es importante identificar claramente el tipo de aplicación que se va a crear. ¿Por qué? Porque para ejecutar consultas eficaces en un grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc, es necesario que las tablas se distribuyan correctamente entre los servidores. La distribución recomendada varía según el tipo de aplicación y sus patrones de consulta. En general, hay dos tipos de aplicaciones que funcionan bien en Hiperescala de PostgreSQL habilitada para Azure Arc:
- Aplicaciones multiinquilino
- Aplicaciones en tiempo real

El primer paso para el modelado de datos es identificar qué tipo se parece más a la aplicación.

Consulte los detalles en [Determinación del tipo de aplicación](../../postgresql/concepts-hyperscale-app-type.md).


## <a name="choose-a-distribution-column"></a>Elección de una columna de distribución
¿Por qué elegir una columna distribuida?

Esta es una de las decisiones de modelado más importantes que se realizarán. Hiperescala de PostgreSQL habilitada para Azure Arc almacena las filas en particiones en función del valor de la columna de distribución de las filas. La elección correcta agrupa los datos relacionados en los mismos nodos físicos, lo que agiliza las consultas y agrega compatibilidad con todas las características de SQL. Si elige de forma incorrecta, el sistema se ejecutará lentamente y no admitirá todas las características de SQL en los nodos. En este artículo se ofrecen consejos sobre las columnas de distribución para los dos escenarios más comunes de hiperescala.

Consulte los detalles en [Elección de columnas de distribución](../../postgresql/concepts-hyperscale-choose-distribution-column.md).


## <a name="table-colocation"></a>Colocación de tabla

La coubicación consiste en almacenar junta la información relacionada, en los mismos nodos. Las consultas pueden ser rápidas cuando todos los datos necesarios están disponibles sin ningún tráfico de red. La coubicación de los datos relacionados en diferentes nodos permite que las consultas se ejecuten eficazmente en paralelo en cada nodo.

Consulte los detalles en [Coubicación de tablas](../../postgresql/concepts-hyperscale-colocation.md).


## <a name="next-steps"></a>Pasos siguientes
- [Obtenga información sobre la creación de Hiperescala de PostgreSQL habilitada para Azure Arc](create-postgresql-hyperscale-server-group.md).
- [Obtenga información acerca del escalado horizontal de los grupos de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc creados en el controlador de datos de Arc](scale-out-postgresql-hyperscale-server-group.md).
- [Obtenga información sobre los servicios de datos habilitados para Azure Arc](https://azure.microsoft.com/services/azure-arc/hybrid-data-services).
- [Obtenga información sobre Azure Arc](https://aka.ms/azurearc).

