---
title: Configuración de parámetros de Azure Database for PostgreSQL mediante Azure PowerShell
description: En este artículo se describe cómo configurar los parámetros de servicio de Azure Database for PostgreSQL mediante PowerShell.
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.devlang: azurepowershell
ms.topic: how-to
ms.date: 06/08/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 2948e0b916eab75f0e0cf1d0ba1b5be1fcf188f4
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87493489"
---
# <a name="customize-azure-database-for-postgresql-server-parameters-using-powershell"></a>Personalización de parámetros de servidor de Azure Database for PostgreSQL mediante PowerShell.

Puede enumerar, mostrar y actualizar los parámetros de configuración de un servidor de Azure Database for PostgreSQL mediante PowerShell. En el nivel del servidor, se expone y se puede modificar un subconjunto de las opciones de configuración del motor.

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía, necesita:

- El [módulo Az PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps) instalado localmente o [Azure Cloud Shell](https://shell.azure.com/) en el explorador
- Un [servidor de Azure Database for PostgreSQL](quickstart-create-postgresql-server-database-using-azure-powershell.md).

> [!IMPORTANT]
> Mientras el módulo Az.PostgreSql PowerShell se encuentre en versión preliminar, debe instalarlo por separado del módulo Az PowerShell mediante el siguiente comando: `Install-Module -Name Az.PostgreSql -AllowPrerelease`.
> Una vez que el módulo Az.PostgreSql PowerShell esté disponible con carácter general, formará parte de las futuras versiones del módulo Az PowerShell y estará disponible de forma nativa en Azure Cloud Shell.

Si decide usar PowerShell de forma local, conéctese a su cuenta de Azure con el cmdlet [Connect-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/connect-azaccount).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="list-server-configuration-parameters-for-azure-database-for-postgresql-server"></a>Lista de los parámetros de configuración del servidor de Azure Database for PostgreSQL

Para obtener una lista de todos los parámetros modificables en un servidor y sus valores, ejecute el cmdlet `Get-AzPostgreSqlConfiguration`.

En el ejemplo siguiente se enumeran los parámetros de configuración del servidor **mydemoserver** incluido en el grupo de recursos **myresourcegroup**.

```azurepowershell-interactive
Get-AzPostgreSqlConfiguration -ResourceGroupName myresourcegroup -ServerName mydemoserver
```

Para ver la definición de cada uno de los parámetros enumerados, consulte la sección de referencia de PostgreSQL en [Variables de entorno](https://www.postgresql.org/docs/12/libpq-envars.html).

## <a name="show-server-configuration-parameter-details"></a>Presentación de los detalles de los parámetros de configuración del servidor

Para mostrar los detalles de un parámetro de configuración determinado de un servidor, ejecute el cmdlet `Get-AzPostgreSqlConfiguration` y especifique el parámetro **Name**.

En este ejemplo se muestran detalles del parámetro de configuración **slow\_query\_log** del servidor **mydemoserver** incluido en el grupo de recursos **myresourcegroup**.

```azurepowershell-interactive
Get-AzPostgreSqlConfiguration -Name slow_query_log -ResourceGroupName myresourcegroup -ServerName mydemoserver
```

## <a name="modify-a-server-configuration-parameter-value"></a>Modificación de un valor de los parámetros de configuración del servidor

También puede modificar el valor de un determinado parámetro de configuración del servidor; esta acción actualizará el valor de configuración subyacente del motor del servidor PostgreSQL. Para actualizar la configuración, use el cmdlet `Update-AzPostgreSqlConfiguration`.

Para actualizar el parámetro de configuración **slow\_query\_log** del servidor **mydemoserver** incluido en el grupo de recursos **myresourcegroup**.

```azurepowershell-interactive
Update-AzPostgreSqlConfiguration -Name slow_query_log -ResourceGroupName myresourcegroup -ServerName mydemoserver -Value On
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Aumento automático del almacenamiento en el servidor de Azure Database for PostgreSQL mediante PowerShell](howto-auto-grow-storage-powershell.md).
