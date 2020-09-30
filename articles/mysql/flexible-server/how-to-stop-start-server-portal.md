---
title: Inicio o detención de un servidor flexible de Azure Database for MySQL mediante Azure Portal
description: En este artículo se explica cómo iniciar y detener operaciones en Azure Database for MySQL mediante Azure Portal.
author: kummanish
ms.author: manishku
ms.service: mysql
ms.topic: how-to
ms.date: 09/21/2020
ms.openlocfilehash: b9f406035d32a9af9ba2f5b085bcaca1b51e9d92
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931897"
---
# <a name="stopstart-an-azure-database-for-mysql---flexible-server-preview"></a>Inicio o detención de un servidor flexible de Azure Database for MySQL (versión preliminar)


> [!IMPORTANT]
> Actualmente, la opción de implementación Servidor flexible de Azure Database for MySQL se encuentra en versión preliminar pública.

En este artículo se proporciona un procedimiento detallado para iniciar y detener un servidor flexible.

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía, necesita:

-   Debe tener un servidor flexible de Azure Database for MySQL.

## <a name="stop-a-running-server"></a>Detención de un servidor en ejecución

1.  En [Azure Portal](https://portal.azure.com/), elija el servidor flexible que quiera detener.

2.  En la página **Información general**, haga clic en el botón **Detener** de la barra de herramientas.

> [!NOTE]
> Una vez detenido el servidor, las restantes operaciones de administración no están disponibles para el servidor flexible.

## <a name="start-a-stopped-server"></a>Inicio de un servidor detenido

1.  En [Azure Portal](https://portal.azure.com/), elija el servidor flexible que desea iniciar.

2.  En la página **Información general**, haga clic en el botón **Detener** de la barra de herramientas.

> [!NOTE]
> Una vez iniciado el servidor, todas las operaciones de administración están disponibles para el servidor flexible.

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre la [conexión de red en servidores flexibles de Azure Database for MySQL](./concepts-networking.md).
- [Creación y administración de una red virtual para un servidor flexible de Azure Database for MySQL mediante Azure Portal](./how-to-manage-virtual-network-portal.md).

