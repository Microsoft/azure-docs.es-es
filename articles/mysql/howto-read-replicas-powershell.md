---
title: Administración de réplicas de lectura en Azure Database for MySQL mediante Azure PowerShell
description: Aprenda a configurar y administrar réplicas de lectura en Azure Database for MySQL mediante PowerShell.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: how-to
ms.date: 6/10/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e99d378b49596ac208f7235a4d1d8b2b1c8c5310
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498793"
---
# <a name="how-to-create-and-manage-read-replicas-in-azure-database-for-mysql-using-powershell"></a>Creación y administración de réplicas de lectura en Azure Database for MySQL mediante PowerShell

En este artículo aprenderá a crear y administrar réplicas de lectura en el servicio Azure Database for MySQL mediante PowerShell. Para más información acerca de las réplicas de lectura, consulte la [introducción](concepts-read-replicas.md).

## <a name="azure-powershell"></a>Azure PowerShell

Puede crear y administrar réplicas de lectura mediante PowerShell.

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía, necesita:

- El [módulo Az PowerShell](/powershell/azure/install-az-ps) instalado localmente o [Azure Cloud Shell](https://shell.azure.com/) en el explorador
- Un [servidor de Azure Database for MySQL](quickstart-create-mysql-server-database-using-azure-powershell.md)

> [!IMPORTANT]
> Mientras el módulo Az.MySql PowerShell se encuentra en versión preliminar, debe instalarlo por separado desde el módulo Az PowerShell con el siguiente comando: `Install-Module -Name Az.MySql -AllowPrerelease`.
> Una vez que el módulo Az.MySql PowerShell esté disponible con carácter general, formará parte de las futuras versiones del módulo Az PowerShell y estará disponible de forma nativa en Azure Cloud Shell.

Si decide usar PowerShell de forma local, conéctese a su cuenta de Azure con el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/Connect-AzAccount).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

> [!IMPORTANT]
> La característica de réplica de lectura solo está disponible para servidores de Azure Database for MySQL en los planes de tarifa De uso general u Optimizada para memoria. Asegúrese de que el servidor maestro está en uno de estos planes de tarifa.

### <a name="create-a-read-replica"></a>Creación de una réplica de lectura

> [!IMPORTANT]
> Cuando se crea una réplica para un servidor maestro que no tiene réplicas existentes, el maestro se reiniciará en primer lugar para prepararse para la replicación. Téngalo en cuenta y realice estas operaciones durante un período de poca actividad.

Un servidor de réplica de lectura se puede crear mediante el comando siguiente:

```azurepowershell-interactive
Get-AzMySqlServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  New-AzMySqlServerReplica -Name mydemoreplicaserver -ResourceGroupName myresourcegroup
```

El comando `New-AzMySqlServerReplica` requiere los siguientes parámetros:

| Configuración | Valor de ejemplo | Descripción  |
| --- | --- | --- |
| ResourceGroupName |  myresourcegroup |  Grupo de recursos donde se crea el servidor de réplica.  |
| Nombre | mydemoreplicaserver | Nombre del nuevo servidor de réplica que se crea. |

Para crear una réplica de lectura entre regiones, use el parámetro **Location**. En el siguiente ejemplo, se crea una réplica en la región **Oeste de EE. UU.** .

```azurepowershell-interactive
Get-AzMySqlServer -Name mrdemoserver -ResourceGroupName myresourcegroup |
  New-AzMySqlServerReplica -Name mydemoreplicaserver -ResourceGroupName myresourcegroup -Location westus
```

Para más información sobre las regiones en las que puede crear una réplica, consulte el [artículo sobre los conceptos de la réplica de lectura](concepts-read-replicas.md).

De forma predeterminada, las réplicas de lectura se crean con la misma configuración de servidor que el maestro, a menos que se especifique el parámetro **Sku**.

> [!NOTE]
> Se recomienda mantener la configuración del servidor de réplica con valores iguales o mayores que el maestro para asegurarse de que la réplica trabajar al mismo nivel que el servidor maestro.

### <a name="list-replicas-for-a-master-server"></a>Lista de réplicas de un servidor maestro

Para ver todas las réplicas de un determinado servidor maestro, ejecute el siguiente comando:

```azurepowershell-interactive
Get-AzMySqlReplica -ResourceGroupName myresourcegroup -ServerName mydemoserver
```

El comando `Get-AzMySqlReplica` requiere los siguientes parámetros:

| Configuración | Valor de ejemplo | Descripción  |
| --- | --- | --- |
| ResourceGroupName |  myresourcegroup |  Grupo de recursos donde se creará el servidor de réplica.  |
| nombreDeServidor | mydemoserver | Nombre o identificador del servidor maestro. |

### <a name="delete-a-replica-server"></a>Eliminación de un servidor de réplica

La eliminación de un servidor de réplica de lectura se puede realizar mediante la ejecución del cmdlet `Remove-AzMySqlServer`.

```azurepowershell-interactive
Remove-AzMySqlServer -Name mydemoreplicaserver -ResourceGroupName myresourcegroup
```

### <a name="delete-a-master-server"></a>Eliminación de un servidor maestro

> [!IMPORTANT]
> Al eliminar un servidor maestro, se detiene la replicación en todos los servidores de réplica y se elimina el propio servidor maestro. Los servidores de réplica se convierten en servidores independientes que ahora admiten tanto lectura como escritura.

Para eliminar un servidor maestro, puede ejecutar el cmdlet `Remove-AzMySqlServer`.

```azurepowershell-interactive
Remove-AzMySqlServer -Name mydemoserver -ResourceGroupName myresourcegroup
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Reinicio de un servidor de Azure Database for MySQL mediante PowerShell](howto-restart-server-powershell.md)
