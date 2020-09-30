---
title: 'Reinicio de Azure Database for PostgreSQL: Servidor flexible con Azure Portal'
description: En este artículo se explica cómo realizar operaciones de reinicio en Azure Database for PostgreSQL mediante Azure Portal.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 952bd6dddf9f276ed1a4a18f03799147f1902198
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931839"
---
# <a name="restart-azure-database-for-postgresql---flexible-server"></a>Reinicio en Azure Database for PostgreSQL: Servidor flexible

> [!IMPORTANT]
> Azure Database for PostgreSQL: Servidor flexible está en versión preliminar.

En este artículo se proporciona un procedimiento detallado para realizar el reinicio del servidor flexible. Esta operación es útil para aplicar cualquier cambio de parámetro estático que requiera el reinicio del servidor de bases de datos. El procedimiento es el mismo para los servidores configurados con alta disponibilidad con redundancia de zona. 

> [!IMPORTANT]
> Cuando se configura con alta disponibilidad, los servidores principal y en espera se reinician al mismo tiempo.

## <a name="pre-requisites"></a>Requisitos previos

Para completar esta guía, necesita:

-   Debe tener un servidor flexible.

## <a name="restart-your-flexible-server"></a>Reinicio del servidor flexible

Siga estos pasos para reiniciar el servidor flexible.

1.  En  [Azure Portal](https://portal.azure.com/), elija el servidor flexible que quiera detener.

2.  Haga clic en **Información general** en el panel izquierdo y, después, haga clic en **Reiniciar**.
   
     :::image type="content" source="./media/how-to-restart-server-portal/restart-base-page.png" alt-text="Selección de reinicio":::

3.  Aparecerá un mensaje de confirmación emergente.

4.  Haga clic en **Sí** si quiere continuar.
   
     :::image type="content" source="./media/how-to-restart-server-portal/restart-pop-up.png" alt-text="Confirmación de reinicio":::
 
6.  Se mostrará una notificación en la que se indica que se ha iniciado la operación de reinicio.

## <a name="next-steps"></a>Pasos siguientes

-   Más información sobre [continuidad empresarial](./concepts-business-continuity.md)
-   Más información sobre la  [alta disponibilidad con redundancia de zona](./concepts-high-availability.md)
