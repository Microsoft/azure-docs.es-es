---
title: Crecimiento automático del almacenamiento mediante Azure PowerShell en Azure Database for MariaDB
description: En este artículo se describe cómo habilitar el crecimiento automático de almacenamiento mediante PowerShell en Azure Database for MariaDB.
author: ajlam
ms.author: andrela
ms.service: mariadb
ms.topic: how-to
ms.date: 5/26/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 2d03a67fc8a8172573598662ad9770b28493e9a2
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87497110"
---
# <a name="auto-grow-storage-in-azure-database-for-mariadb-server-using-powershell"></a>Crecimiento automático del almacenamiento en el servidor de Azure Database for MariaDB mediante PowerShell

En este artículo se describe cómo configurar el almacenamiento en el servidor Azure Database for MariaDB para que aumente sin que ello afecte a la carga de trabajo.

El aumento automático del almacenamiento impide que el servidor [alcance el límite de almacenamiento](/azure/mariadb/concepts-pricing-tiers#reaching-the-storage-limit) y se vuelva de solo lectura. En el caso de los servidores con 100 GB o menos de almacenamiento aprovisionado, el tamaño aumenta en 5 GB cuando el espacio disponible es inferior al 10 %. En el caso de los servidores con más de 100 GB de almacenamiento aprovisionado, el tamaño aumenta un 5 % cuando el espacio disponible es inferior a 10 GB. Los límites de almacenamiento máximos se aplican como se especifica en la sección de almacenamiento de los [planes de tarifa de Azure Database for MariaDB](/azure/mariadb/concepts-pricing-tiers#storage).

> [!IMPORTANT]
> Recuerde que el almacenamiento solo se puede escalar verticalmente, no reducir.

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía, necesita:

- El [módulo Az PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps) instalado localmente o [Azure Cloud Shell](https://shell.azure.com/) en el explorador
- Un [servidor de Azure Database for MariaDB](quickstart-create-mariadb-server-database-using-azure-powershell.md)

> [!IMPORTANT]
> Mientras el módulo de PowerShell Az.MariaDb se encuentre en versión preliminar, debe instalarlo por separado desde el módulo Az de PowerShell con el siguiente comando: `Install-Module -Name Az.MariaDb -AllowPrerelease`.
> Una vez que el módulo Az.MariaDb de PowerShell esté disponible con carácter general, formará parte de las futuras versiones del módulo Az de PowerShell y estará disponible de forma nativa en Azure Cloud Shell.

Si decide usar PowerShell de forma local, conéctese a su cuenta de Azure con el cmdlet [Connect-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/connect-azaccount).

## <a name="enable-mariadb-server-storage-auto-grow"></a>Habilitación del crecimiento automático del almacenamiento de servidores MariaDB

Habilite el almacenamiento de aumento automático de servidor en un servidor existente con el siguiente comando:

```azurepowershell-interactive
Update-AzMariaDbServer -Name mydemoserver -ResourceGroupName myresourcegroup -StorageAutogrow Enabled
```

Habilite el almacenamiento de aumento automático de servidor mientras crea un servidor con el siguiente comando:

```azurepowershell-interactive
$Password = Read-Host -Prompt 'Please enter your password' -AsSecureString
New-AzMariaDbServer -Name mydemoserver -ResourceGroupName myresourcegroup -Sku GP_Gen5_2 -StorageAutogrow Enabled -Location westus -AdministratorUsername myadmin -AdministratorLoginPassword $Password
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación y administración de réplicas de lectura en Azure Database for MariaDB mediante PowerShell](howto-read-replicas-powershell.md).
