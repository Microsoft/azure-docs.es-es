---
title: Restauración de un servidor flexible de Azure Database for PostgreSQL en Azure Portal
description: En este artículo se explica cómo realizar operaciones de restauración en Azure Database for PostgreSQL mediante Azure Portal.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: e69bcb3d9e4dca4c45bf9a6fe8ed4d54e7f4a8cd
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932178"
---
# <a name="point-in-time-restore-of-a-flexible-server"></a>Restauración de un servidor flexible a un momento dado

> [!IMPORTANT]
> La opción de implementación Servidor flexible de Azure Database for PostgreSQL está en versión preliminar

En este artículo se proporciona un procedimiento detallado para llevar a cabo recuperaciones a un momento dado en servidores flexibles mediante copias de seguridad. Puede realizarlas al punto de restauración más antiguo o a un punto de restauración personalizado dentro del período de retención.

## <a name="pre-requisites"></a>Requisitos previos

Para completar esta guía, necesita:

-   Un servidor flexible de Azure Database for PostgreSQL. Este mismo procedimiento también se puede aplicar a un servidor flexible configurado con redundancia de zona.

## <a name="restoring-to-the-earliest-restore-point"></a>Restauración al punto de restauración más antiguo

Siga estos pasos para restaurar un servidor flexible mediante la copia de seguridad más antigua.

1.  En  [Azure Portal](https://portal.azure.com/), elija el servidor flexible desde el que desea restaurar la copia de seguridad.

2.  Haga clic en **Información general** en el panel izquierdo y, después, haga clic en **Restaurar**.
   
   :::image type="content" source="./media/how-to-restore-server-portal/restore-overview.png" alt-text="Introducción a la restauración":::

3.  La página de restauración se mostrará con una opción para elegir entre el punto de restauración más antiguo y un punto de restauración personalizado.

4.  Seleccione **Punto de restauración más antiguo** y especifique un nombre de servidor nuevo en el campo **Restaurar en el servidor nuevo**. Se muestra la marca de tiempo más antigua a la que se puede realizar la restauración. 
   
   :::image type="content" source="./media/how-to-restore-server-portal/restore-earliest.png" alt-text="Punto de restauración más antiguo":::

5.  Haga clic en **Aceptar**.

6.  Se mostrará una notificación en la que se indica que se ha iniciado la operación de restauración.

## <a name="restoring-to-a-custom-restore-point"></a>Restauración a un punto de restauración personalizado

Siga estos pasos para restaurar un servidor flexible mediante la copia de seguridad más antigua.

1.  En  [Azure Portal](https://portal.azure.com/), elija el servidor flexible desde el que desea restaurar la copia de seguridad.

2.  En la página de información general, haga clic en **Restaurar**.
 :::image type="content" source="./media/how-to-restore-server-portal/restore-overview.png" alt-text="Introducción a la restauración":::
    
3.  La página de restauración se mostrará con una opción para elegir entre el punto de restauración más antiguo y un punto de restauración personalizado.

4.  Elija **Punto de restauración personalizado**.

5.  Seleccione la fecha y hora, y especifique un nombre de servidor nuevo en el campo **Restaurar en el servidor nuevo**. 
   
:::image type="content" source="./media/how-to-restore-server-portal/restore-custom.png" alt-text="Hora de restauración personalizada":::
 
6.  Haga clic en **Aceptar**.

7.  Se mostrará una notificación en la que se indica que se ha iniciado la operación de restauración.

## <a name="next-steps"></a>Pasos siguientes

-   Más información sobre la [continuidad empresarial](./concepts-business-continuity.md).
-   Más información sobre la  [alta disponibilidad con redundancia de zona](./concepts-high-availability.md).
-   Más información sobre [copia de seguridad y recuperación](./concepts-backup-restore.md).
