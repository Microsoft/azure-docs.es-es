---
title: Reinicio de un servidor mediante la CLI de Azure en Azure Database for MySQL
description: En este artículo se describe cómo reiniciar un servidor de Azure Database for MySQL mediante la CLI de Azure.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: how-to
ms.date: 3/18/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 9707aaa911fc8ceb8c03b369bd6479646ca3418c
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500732"
---
# <a name="restart-azure-database-for-mysql-server-using-the-azure-cli"></a>Reinicio de un servidor de Azure Database for MySQL mediante la CLI de Azure
En este tema se describe cómo reiniciar un servidor de Azure Database for MySQL. Es posible que deba reiniciar el servidor por motivos de mantenimiento, lo que causa una breve interrupción del servicio mientras el servidor realiza la operación.

Si el servicio está ocupado, se bloqueará el reinicio del servidor. Por ejemplo, el servicio puede estar procesando una operación solicitada anteriormente, como el escalado de núcleos virtuales.

El tiempo necesario para completar un reinicio depende el proceso de recuperación de MySQL. Para reducir el tiempo de reinicio, se recomienda que minimice la cantidad de actividades que se ejecutan en el servidor antes del reinicio.

## <a name="prerequisites"></a>Requisitos previos
Para completar esta guía, necesita:
- Un [servidor de Azure Database for MySQL](quickstart-create-server-up-azure-cli.md)

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

> [!IMPORTANT]
> Esta guía de procedimientos requiere el uso de la CLI de Azure versión 2.0 o posterior. Para confirmar la versión, en el símbolo del sistema de la CLI de Azure, escriba `az --version`. Para la instalación o la actualización, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).


## <a name="restart-the-server"></a>Reinicio del servidor

Reinicie el servidor con el siguiente comando:

```azurecli-interactive
az mysql server restart --name mydemoserver --resource-group myresourcegroup
```

## <a name="next-steps"></a>Pasos siguientes

Aprenda a [establecer parámetros en Azure Database for MySQL](howto-configure-server-parameters-using-cli.md).
