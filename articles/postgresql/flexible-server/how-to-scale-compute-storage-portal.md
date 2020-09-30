---
title: Operaciones de escalado en un servidor flexible de Azure Database for PostgreSQL mediante Azure Portal
description: En este artículo se explica cómo realizar operaciones de escalado en Azure Database for PostgreSQL mediante Azure Portal.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 1542bba53b51ffdf2129953a81e5d13975ade434
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932254"
---
# <a name="scale-operations-in-flexible-server"></a>Operaciones de escalado en un servidor flexible

> [!IMPORTANT]
> La opción de implementación Servidor flexible de Azure Database for PostgreSQL está en versión preliminar

En este artículo se indican los pasos necesarios para realizar operaciones de escalado de proceso y de almacenamiento. Podrá cambiar los niveles de proceso entre las SKU por ráfagas, de uso general y con optimización para memoria, lo que incluye elegir el número de núcleos virtuales más apropiado para ejecutar la aplicación. También podrá escalar verticalmente el almacenamiento. Las IOPS esperadas se muestran según el nivel de proceso, los núcleos virtuales y la capacidad de almacenamiento. La estimación de costos también se muestra en función del elemento que se haya seleccionado.

> [!IMPORTANT]
> El almacenamiento no se puede reducir verticalmente.

## <a name="pre-requisites"></a>Requisitos previos

Para completar esta guía, necesita:

-   Un servidor flexible de Azure Database for PostgreSQL. Este mismo procedimiento también se puede aplicar a un servidor flexible configurado con redundancia de zona.
> [!IMPORTANT]
> Cuando se configura con alta disponibilidad, no se puede elegir una SKU por ráfagas. Durante la operación de escalado, en primer lugar, el servidor en espera se escala al tamaño deseado, luego se conmuta por error el servidor principal y se escala el principal. 

## <a name="scaling-the-compute-tier-and-size"></a>Escalado del nivel de proceso y del tamaño

Siga estos pasos para elegir el nivel de proceso.
 
1.  En  [Azure Portal](https://portal.azure.com/), elija el servidor flexible desde el que desea restaurar la copia de seguridad.

2.  Haga clic en **Proceso y almacenamiento**.

3.  Se muestra una página con la configuración actual.
 :::image type="content" source="./media/how-to-scale-compute-storage-portal/click-compute-storage.png" alt-text="Vista de Proceso y almacenamiento":::

4.  Hay tres clases de proceso (por ráfagas, de uso general y optimizada para memoria) y puede elegir cualquiera de ellas.
   :::image type="content" source="./media/how-to-scale-compute-storage-portal/list-compute-tiers.png" alt-text="enumeración de niveles de proceso":::


5.  Si le parecen bien los tamaños de memoria y los núcleos virtuales predeterminados, puede omitir el paso siguiente.

6.  Si desea cambiar el número de núcleos virtuales, puede hacer clic en la lista desplegable **Compute size** (Tamaño de proceso) y en el número deseado de núcleos virtuales o memoria en la lista.
    
    - Nivel de proceso Por ráfagas: :::image type="content" source="./media/how-to-scale-compute-storage-portal/compute-burstable-dropdown.png" alt-text="proceso por ráfagas":::

    - Nivel de proceso De uso general: :::image type="content" source="./media/how-to-scale-compute-storage-portal/compute-general-purpose-dropdown.png" alt-text="proceso de uso general":::

    - Nivel de proceso Optimizado para memoria: :::image type="content" source="./media/how-to-scale-compute-storage-portal/compute-memory-optimized-dropdown.png" alt-text="proceso optimizado para memoria":::

7.  Haga clic en **Guardar**. 
8.  Aparecerá un mensaje de confirmación. Si desea continuar, haga clic en **Aceptar**. 
9.  Una notificación sobre la operación de escalado en curso.


## <a name="scaling-storage-size"></a>Escalado del tamaño de almacenamiento

Siga estos pasos para aumentar el tamaño de almacenamiento.

1.  En  [Azure Portal](https://portal.azure.com/), elija el servidor flexible cuyo tamaño de almacenamiento desea aumentar.
2.  Haga clic en **Proceso y almacenamiento**.

3.  Se muestra una página con la configuración actual.
   
:::image type="content" source="./media/how-to-scale-compute-storage-portal/click-compute-storage.png" alt-text="haga clic en Proceso y almacenamiento":::
4.  Aparece el campo **Storage size in GiB** (Tamaño de almacenamiento, en GiB) con una barra deslizante con el tamaño actual.

5.  Deslice la barra hasta el tamaño que desee. Se muestra el número de IOPS correspondiente. Este número de IOPS depende del nivel de proceso y del tamaño. También se muestra la información del costo. 

 :::image type="content" source="./media/how-to-scale-compute-storage-portal/storage-scaleup.png" alt-text="escalado vertical del almacenamiento":::

6.  Si está conforme con el tamaño de almacenamiento, haga clic en **Guardar**. 
7.  Aparecerá un mensaje de confirmación. Si desea continuar, haga clic en **Aceptar**. 
8.  Una notificación sobre la operación de escalado en curso.

## <a name="next-steps"></a>Pasos siguientes

-   Más información sobre la [continuidad empresarial](./concepts-business-continuity.md).
-   Más información acerca de la [alta disponibilidad](./concepts-high-availability.md).
-   Más información sobre [copia de seguridad y recuperación](./concepts-backup-restore.md).
