---
title: 'Inicio y detención de Azure Database for PostgreSQL con Azure Portal: servidor flexible'
description: En este artículo se explica cómo iniciar y detener operaciones en Azure Database for PostgreSQL mediante Azure Portal.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 4c393e0048a0058ebe0fbf2b0ee65f6ae2e184c5
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932176"
---
# <a name="stopstart-an-azure-database-for-postgresql---flexible-server-preview"></a>Inicio y detención de Azure Database for PostgreSQL: servidor flexible (versión preliminar)

> [!IMPORTANT]
> Azure Database for PostgreSQL: servidor flexible está actualmente en versión preliminar.

En este artículo se proporcionan instrucciones paso a paso para iniciar y detener un servidor flexible.

## <a name="pre-requisites"></a>Requisitos previos

Para completar esta guía, necesita:

-   Debe tener Azure Database for PostgreSQL: servidor flexible.

## <a name="stop-a-running-server"></a>Detención de un servidor en ejecución

1.  En [Azure Portal](https://portal.azure.com/), elija el servidor flexible que quiera detener.

2.  En la página **Información general**, haga clic en el botón **Detener** de la barra de herramientas.

> [!NOTE]
> Una vez detenido el servidor, las demás operaciones de administración no estarán disponibles para el servidor flexible.

Tenga en cuenta que los servidores detenidos se volverán a iniciar automáticamente transcurridos siete días. Las actualizaciones de mantenimiento pendientes se aplicarán la próxima vez que el servidor se inicie.

## <a name="start-a-stopped-server"></a>Inicio de un servidor detenido

1.  En [Azure Portal](https://portal.azure.com/), elija el servidor flexible que quiera iniciar.

2.  En la página **Información general**, haga clic en el botón **Iniciar** de la barra de herramientas.

> [!NOTE]
> Una vez iniciado el servidor, todas las operaciones de administración estarán disponibles para el servidor flexible.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre las [opciones de proceso y almacenamiento de Azure Database for PostgreSQL: servidor flexible](./concepts-compute-storage.md).
