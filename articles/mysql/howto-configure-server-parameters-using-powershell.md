---
title: Configuración de parámetros de Azure Database for MySQL mediante Azure PowerShell
description: En este artículo se describe cómo configurar los parámetros de servicio de Azure Database for MySQL mediante PowerShell.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.devlang: azurepowershell
ms.topic: how-to
ms.date: 4/29/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: cb17c8631392972c429220b846720447be48cecf
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495818"
---
# <a name="configure-server-parameters-in-azure-database-for-mysql-using-powershell"></a>Configuración de parámetros del servidor en Azure Database for MySQL con PowerShell

Puede enumerar, mostrar y actualizar los parámetros de configuración de un servidor de Azure Database for MySQL mediante PowerShell. En el nivel del servidor, se expone y se puede modificar un subconjunto de las opciones de configuración del motor.

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía, necesita:

- El [módulo Az PowerShell](/powershell/azure/install-az-ps) instalado localmente o [Azure Cloud Shell](https://shell.azure.com/) en el explorador
- Un [servidor de Azure Database for MySQL](quickstart-create-mysql-server-database-using-azure-powershell.md)

> [!IMPORTANT]
> Mientras el módulo Az.MySql PowerShell se encuentra en versión preliminar, debe instalarlo por separado desde el módulo Az PowerShell con el siguiente comando: `Install-Module -Name Az.MySql -AllowPrerelease`.
> Una vez que el módulo Az.MySql PowerShell esté disponible con carácter general, formará parte de las futuras versiones del módulo Az PowerShell y estará disponible de forma nativa en Azure Cloud Shell.

Si decide usar PowerShell de forma local, conéctese a su cuenta de Azure con el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/Connect-AzAccount).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="list-server-configuration-parameters-for-azure-database-for-mysql-server"></a>Lista de los parámetros de configuración del servidor de Azure Database for MySQL

Para obtener una lista de todos los parámetros modificables en un servidor y sus valores, ejecute el cmdlet `Get-AzMySqlConfiguration`.

En el ejemplo siguiente se enumeran los parámetros de configuración del servidor **mydemoserver** incluido en el grupo de recursos **myresourcegroup**.

```azurepowershell-interactive
Get-AzMySqlConfiguration -ResourceGroupName myresourcegroup -ServerName mydemoserver
```

Para ver la definición de cada uno de los parámetros enumerados, consulte la sección de referencia de MySQL en [Server System Variables](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html) (Variables del sistema del servidor).

## <a name="show-server-configuration-parameter-details"></a>Presentación de los detalles de los parámetros de configuración del servidor

Para mostrar los detalles de un parámetro de configuración determinado de un servidor, ejecute el cmdlet `Get-AzMySqlConfiguration` y especifique el parámetro **Name**.

En este ejemplo se muestran detalles del parámetro de configuración **slow\_query\_log** del servidor **mydemoserver** incluido en el grupo de recursos **myresourcegroup**.

```azurepowershell-interactive
Get-AzMySqlConfiguration -Name slow_query_log -ResourceGroupName myresourcegroup -ServerName mydemoserver
```

## <a name="modify-a-server-configuration-parameter-value"></a>Modificación de un valor de los parámetros de configuración del servidor

También puede modificar el valor de un determinado parámetro de configuración del servidor; esta acción actualizará el valor de configuración subyacente del motor del servidor de MySQL. Para actualizar la configuración, use el cmdlet `Update-AzMySqlConfiguration`.

Para actualizar el parámetro de configuración **slow\_query\_log** del servidor **mydemoserver** incluido en el grupo de recursos **myresourcegroup**.

```azurepowershell-interactive
Update-AzMySqlConfiguration -Name slow_query_log -ResourceGroupName myresourcegroup -ServerName mydemoserver -Value On
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Aumento automático del almacenamiento en el servidor de Azure Database for MySQL mediante PowerShell](howto-auto-grow-storage-powershell.md)
