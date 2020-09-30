---
title: 'Denegación del acceso a una red pública: Azure portal - Azure Database for MySQL'
description: Información acerca de cómo configurar la denegación del acceso a una red pública mediante Azure Portal para Azure Database for MySQL
author: kummanish
ms.author: manishku
ms.service: mysql
ms.topic: how-to
ms.date: 03/10/2020
ms.openlocfilehash: eb5c4e4c4dfb73b2f7c9dc9f2629296790790885
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90896135"
---
# <a name="deny-public-network-access-in-azure-database-for-mysql-using-azure-portal"></a>Denegación del acceso a una red pública en Azure Database for MySQL mediante Azure Portal

En este artículo se describe cómo puede configurar un servidor de Azure Database for MySQL para denegar todas las configuraciones públicas y permitir solo las conexiones a través de puntos de conexión privados para mejorar aún más la seguridad de la red.

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía, necesita:

* Una base de datos de [Azure Database for MySQL](quickstart-create-mysql-server-database-using-azure-portal.md)

## <a name="set-deny-public-network-access"></a>Establecimiento de la denegación del acceso a una red pública

Siga estos pasos para establecer la denegación del acceso a una red pública del servidor de MySQL:

1. En [Azure Portal](https://portal.azure.com/), seleccione el servidor de Azure Database for MySQL existente.

1. En la página del servidor de MySQL, en **Configuración**, haga clic en **Seguridad de la conexión** para abrir la página de configuración de seguridad de la conexión.

1. En **Deny Public Network Access** (Denegar acceso de red pública), seleccione **Yes** (Sí) para habilitar la denegación del acceso público para el servidor de MySQL.

    :::image type="content" source="./media/howto-deny-public-network-access/setting-deny-public-network-access.PNG" alt-text="Denegación del acceso de red de Azure Database for MySQL":::

1. Haga clic en **Guardar** para guardar los cambios.

1. Se enviará una notificación para confirmar la correcta habilitación de la configuración de seguridad de la conexión.

    :::image type="content" source="./media/howto-deny-public-network-access/setting-deny-public-network-access-success.png" alt-text="Denegación correcta del acceso de red de Azure Database for MySQL":::

## <a name="next-steps"></a>Pasos siguientes

Aprenda sobre la [creación de alertas de métricas](howto-alert-on-metric.md).