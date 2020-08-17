---
title: Nodos de Hiperescala (Citus) en Azure Database for PostgreSQL
description: Obtenga información acerca de los dos tipos de nodos, coordinador y trabajadores, en un grupo de servidores de Azure Database for PostgreSQL.
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 07/28/2020
ms.openlocfilehash: af743ca56572f507091db01f11d3283294a9e3d5
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87382804"
---
# <a name="nodes-in-azure-database-for-postgresql--hyperscale-citus"></a>Nodos de Azure Database for PostgreSQL: Hiperescala (Citus)

El tipo de hospedaje Hiperescala (Citus) permite a los servidores de Azure Database for PostgreSQL (llamados "nodos") coordinarse entre sí en una arquitectura "nada compartido". Los nodos de un grupo de servidores almacenan colectivamente más datos y usan más núcleos de CPU de lo que sería posible en un único servidor. La arquitectura también permite escalar la base de datos al agregar más nodos al grupo de servidores.

## <a name="coordinator-and-workers"></a>Coordinación y trabajos

Cada grupo de servidores tiene un nodo de coordinación y varios trabajos. Las aplicaciones envían sus consultas al nodo de coordinación, que las retransmite a los trabajos pertinentes y acumula los resultados. Las aplicaciones no son capaces de conectarse directamente a los trabajos.

Hiperescala (Citus) permite al administrador de bases de datos *distribuir* tablas y almacenar distintas filas en nodos de trabajo diferentes. Las tablas distribuidas son la clave para el rendimiento de Hiperescala. Si no se distribuyen las tablas, las deja totalmente en el nodo de coordinación y no pueden aprovechar el paralelismo entre máquinas.

Para cada consulta sobre las tablas distribuidas, el coordinador la enruta a un único nodo de trabajo o la paraleliza en varias según si los datos necesarios se encuentren en un único nodo o en varios. El coordinador decide qué hacer tras consultar las tablas de metadatos. Estas tablas hacen un seguimiento de los nombres DNS y del estado de los nodos de trabajo, y de la distribución de datos entre nodos.

## <a name="next-steps"></a>Pasos siguientes
- Más información sobre las [Tablas distribuidas](concepts-hyperscale-distributed-data.md).
