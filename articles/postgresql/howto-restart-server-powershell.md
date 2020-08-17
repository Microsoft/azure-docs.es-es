---
title: Reinicio de un servidor mediante Azure PowerShell en Azure Database for PostgreSQL
description: En este artículo se describe cómo reiniciar un servidor de Azure Database for PostgreSQL mediante PowerShell.
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: how-to
ms.date: 06/08/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 3fd4bcdd2f80bfd5f79488e7e4c93219be97927c
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503555"
---
# <a name="restart-azure-database-for-postgresql-server-using-powershell"></a>Reinicio de un servidor de Azure Database for PostgreSQL mediante PowerShell.

En este tema se describe cómo reiniciar un servidor de Azure Database for PostgreSQL. Es posible que deba reiniciar el servidor por motivos de mantenimiento, lo que causa una breve interrupción del servicio durante la operación.

Si el servicio está ocupado, se bloquea el reinicio del servidor. Por ejemplo, el servicio puede estar procesando una operación solicitada anteriormente, como el escalado de núcleos virtuales.

El tiempo necesario para completar un reinicio depende el proceso de recuperación de PostgreSQL. Para reducir el tiempo de reinicio, se recomienda que minimice la cantidad de actividades que se ejecutan en el servidor antes del reinicio.

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía, necesita:

- El [módulo Az PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps) instalado localmente o [Azure Cloud Shell](https://shell.azure.com/) en el explorador
- Un [servidor de Azure Database for PostgreSQL](quickstart-create-postgresql-server-database-using-azure-powershell.md).

> [!IMPORTANT]
> Mientras el módulo Az.PostgreSql PowerShell se encuentre en versión preliminar, debe instalarlo por separado del módulo Az PowerShell mediante el siguiente comando: `Install-Module -Name Az.PostgreSql -AllowPrerelease`.
> Una vez que el módulo Az.PostgreSql PowerShell esté disponible con carácter general, formará parte de las futuras versiones del módulo Az PowerShell y estará disponible de forma nativa en Azure Cloud Shell.

Si decide usar PowerShell de forma local, conéctese a su cuenta de Azure con el cmdlet [Connect-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/connect-azaccount).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="restart-the-server"></a>Reinicio del servidor

Reinicie el servidor con el siguiente comando:

```azurepowershell-interactive
Restart-AzPostgreSqlServer -Name mydemoserver -ResourceGroupName myresourcegroup
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de un servidor de Azure Database for PostgreSQL mediante PowerShell](quickstart-create-postgresql-server-database-using-azure-powershell.md)
