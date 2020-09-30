---
title: Crecimiento automático del almacenamiento mediante Azure Portal en Azure Database for MySQL
description: En este artículo se describe cómo habilitar el aumento automático de almacenamiento en Azure Database for MySQL mediante Azure Portal
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.topic: how-to
ms.date: 3/18/2020
ms.openlocfilehash: d4dc5c2690be7b9abbda685e78ea562878626b5c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90902853"
---
# <a name="auto-grow-storage-in-azure-database-for-mysql-using-the-azure-portal"></a>Aumento automático del almacenamiento en Azure Database for MySQL mediante Azure Portal
En este artículo se describe cómo configurar el almacenamiento en el servidor Azure Database for MySQL para que aumente sin que ello afecte a la carga de trabajo.

Cuando un servidor alcanza el límite de almacenamiento asignado, se marca como de solo lectura. Sin embargo, si habilita el aumento automático de almacenamiento en el servidor, acomodará los datos que se vayan acumulando. Para servidores con menos de 100 GB de almacenamiento aprovisionado, el tamaño del almacenamiento aprovisionado se incrementa en 5 GB tan pronto como el almacenamiento disponible se encuentre por debajo de 1 GB o el 10 % del almacenamiento aprovisionado. En cuanto a servidores con más de 100 GB de almacenamiento aprovisionado, el tamaño del almacenamiento aprovisionado se incrementa en un 5 % cuando el espacio de almacenamiento disponible es inferior al 5 % del tamaño de almacenamiento aprovisionado. Se aplican los límites máximos de almacenamiento según lo especificado [aquí](https://docs.microsoft.com/azure/mysql/concepts-pricing-tiers#storage).

## <a name="prerequisites"></a>Requisitos previos
Para completar esta guía, necesita:
- Un [servidor de Azure Database for MySQL](quickstart-create-mysql-server-database-using-azure-portal.md)

## <a name="enable-storage-auto-grow"></a>Habilitación del aumento automático de almacenamiento 

Siga estos pasos para establecer el aumento automático de almacenamiento en el servidor de MySQL:

1. En [Azure Portal](https://portal.azure.com/), seleccione el servidor de Azure Database for MySQL existente.

2. En la página del servidor de MySQL, en el encabezado **Configuración**, haga clic en **Plan de tarifa** para abrir la página de planes de tarifa.

3. En la sección de aumento automático, seleccione **Sí** para habilitarlo para el almacenamiento.

    :::image type="content" source="./media/howto-auto-grow-storage-portal/3-auto-grow.png" alt-text="Azure Database for MySQL: Settings_Pricing_tier: aumento automático":::

4. Haga clic en **Aceptar** para guardar los cambios.

5. Se enviara una notificación de confirmación de la habilitación del aumento automático.

    :::image type="content" source="./media/howto-auto-grow-storage-portal/5-auto-grow-success.png" alt-text="Azure Database for MySQL: aumento automático correcto":::

## <a name="next-steps"></a>Pasos siguientes

Aprenda sobre la [creación de alertas de métricas](howto-alert-on-metric.md).
