---
title: Configuración de TLS de Azure Database for MySQL en Azure Portal
description: Obtenga información sobre cómo configurar los valores de TLS mediante Azure Portal para Azure Database for MySQL
author: kummanish
ms.author: manishku
ms.service: mysql
ms.topic: how-to
ms.date: 06/02/2020
ms.openlocfilehash: d94e589a19b29e68883c0217b62b883f9d026789
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90902733"
---
# <a name="configuring-tls-settings-in-azure-database-for-mysql-using-azure-portal"></a>Configuración de los valores de TLS en Azure Database for MySQL mediante Azure Portal

En este artículo se describe cómo configurar un servidor de Azure Database for MySQL para aplicar la versión de TLS mínima permitida para que pasen las conexiones y se denieguen todas las conexiones con una versión de TLS inferior a la versión de TLS mínima configurada, lo que mejora la seguridad de la red.

Puede aplicar la versión de TLS para conectarse a su Azure Database for MySQL. Los clientes tienen ahora la opción de establecer la versión de TLS mínima para su servidor de bases de datos. Por ejemplo, si se establece la versión de TLS mínima en 1.0 significa que los clientes se conectarán mediante TLS 1.0, 1.1 y 1.2. Como alternativa, si se establece en 1.2 significa que solo se permiten los clientes que se conectan mediante TLS 1.2 o versión posterior y se rechazan todas las conexiones entrantes con TLS 1.0 y TLS 1.1.

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía, necesita:

* Una base de datos de [Azure Database for MySQL](quickstart-create-mysql-server-database-using-azure-portal.md)

## <a name="set-tls-configurations-for-azure-database-for-mysql"></a>Establecimiento de las configuraciones de TLS para Azure Database for MySQL

Siga estos pasos para establecer la versión de TLS mínima del servidor de MySQL:

1. En [Azure Portal](https://portal.azure.com/), seleccione el servidor de Azure Database for MySQL existente.

1. En la página del servidor de MySQL, en **Configuración**, haga clic en **Seguridad de la conexión** para abrir la página de configuración de seguridad de la conexión.

1. En **Versión de TLS mínima**, seleccione **1.2** para denegar las conexiones con una versión de TLS inferior a TLS 1.2 para el servidor de MySQL.

    :::image type="content" source="./media/howto-tls-configurations/setting-tls-value.png" alt-text="Configuración de TLS para Azure Database for MySQL":::

1. Haga clic en **Guardar** para guardar los cambios.

1. Se enviará una notificación para confirmar la correcta habilitación de la configuración de seguridad de la conexión.

    :::image type="content" source="./media/howto-tls-configurations/setting-tls-value-success.png" alt-text="Configuración correcta de TLS para Azure Database for MySQL":::

## <a name="next-steps"></a>Pasos siguientes

- Información acerca de la [creación de alertas basadas en métricas](howto-alert-on-metric.md).