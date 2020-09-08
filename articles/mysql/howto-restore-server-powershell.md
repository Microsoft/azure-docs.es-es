---
title: Copia de seguridad y restauración en Azure Database for MySQL mediante Azure PowerShell
description: Obtenga información sobre cómo realizar una copia de seguridad y la restauración de un servidor en Azure Database for MySQL mediante Azure PowerShell.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.devlang: azurepowershel
ms.topic: how-to
ms.date: 4/28/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 5571d5a937fc48030c38ebe78c86ef27d6727a67
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87837319"
---
# <a name="how-to-back-up-and-restore-an-azure-database-for-mysql-server-using-powershell"></a>Copia de seguridad y restauración de un servidor de Azure Database for MySQL mediante PowerShell

Periódicamente, se realizan copias de seguridad de los servidores de Azure Database for MySQL para habilitar las características de restauración. Con esta característica, puede restaurar el servidor y todas sus bases de datos en un servidor nuevo a un momento dado anterior.

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía, necesita:

- El [módulo Az PowerShell](/powershell/azure/install-az-ps) instalado localmente o [Azure Cloud Shell](https://shell.azure.com/) en el explorador
- Un [servidor de Azure Database for MySQL](quickstart-create-mysql-server-database-using-azure-powershell.md)

> [!IMPORTANT]
> Mientras el módulo Az.MySql PowerShell se encuentra en versión preliminar, debe instalarlo por separado desde el módulo Az PowerShell con el siguiente comando: `Install-Module -Name Az.MySql -AllowPrerelease`.
> Una vez que el módulo Az.MySql PowerShell esté disponible con carácter general, formará parte de las futuras versiones del módulo Az PowerShell y estará disponible de forma nativa en Azure Cloud Shell.

Si decide usar PowerShell de forma local, conéctese a su cuenta de Azure con el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/Connect-AzAccount).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="set-backup-configuration"></a>Configuración de copia de seguridad

En el momento de crear el servidor, elegirá la configuración del servidor para copias de seguridad con redundancia local o con redundancia geográfica.

> [!NOTE]
> Después de crear un servidor, no se puede cambiar el tipo de redundancia elegido, redundancia geográfica o redundancia local.

Al crear un servidor mediante el comando `New-AzMySqlServer`, el parámetro **GeoRedundantBackup** decide la opción de redundancia de copia de seguridad. Si está **Habilitado**, se toman las copias de seguridad con redundancia geográfica. Si está **Deshabilitado**, se toman las copias de seguridad con redundancia local.

El período de retención de la copia de seguridad se configura mediante el parámetro **BackupRetentionDay**.

Para más información acerca de cómo establecer estos valores durante la creación del servidor, consulte [Creación de un servidor de Azure Database for MySQL mediante PowerShell](quickstart-create-mysql-server-database-using-azure-powershell.md).

El período de retención de copia de seguridad de un servidor se puede cambiar de la forma siguiente:

```azurepowershell-interactive
Update-AzMySqlServer -Name mydemoserver -ResourceGroupName myresourcegroup -BackupRetentionDay 10
```

En el ejemplo anterior se cambia el período de retención de copia de seguridad de mydemoserver a 10 días.

El período de retención de copia de seguridad establece durante cuánto tiempo se puede realizar una restauración a un momento dado, porque se basa en las copias de seguridad disponibles. La restauración a un momento dado se describe con más detalle en la sección siguiente.

## <a name="server-point-in-time-restore"></a>Restauración del servidor a un momento dado

Puede restaurar el servidor a un momento dado anterior. Los datos restaurados se copian en un nuevo servidor y el existente no cambia. Por ejemplo, si una tabla se ha eliminado por error, puede restaurar al momento justo antes de la eliminación. Así podrá recuperar la tabla y los datos que faltan de la copia restaurada del servidor.

Para restaurar el servidor, use el cmdlet de PowerShell de `Restore-AzMySqlServer`.

### <a name="run-the-restore-command"></a>Ejecutar el comando restore

Para restaurar el servidor, ejecute el ejemplo siguiente desde PowerShell.

```azurepowershell-interactive
$restorePointInTime = (Get-Date).AddMinutes(-10)
Get-AzMySqlServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  Restore-AzMySqlServer -Name mydemoserver-restored -ResourceGroupName myresourcegroup -RestorePointInTime $restorePointInTime -UsePointInTimeRestore
```

El conjunto de parámetros **PointInTimeRestore** del cmdlet `Restore-AzMySqlServer` requiere los parámetros siguientes:

| Configuración | Valor sugerido | Descripción  |
| --- | --- | --- |
| ResourceGroupName |  myresourcegroup |  Grupo de recursos donde existe el servidor de origen.  |
| Nombre | mydemoserver-restored | Nombre del nuevo servidor que se crea mediante el comando de restauración. |
| RestorePointInTime | 2020-03-13T13:59:00Z | Seleccione un momento dado para restaurar. Esta fecha y hora debe estar dentro del período de retención de copia de seguridad del servidor de origen. Use el formato de fecha y hora ISO8601. Por ejemplo, puede usar su propia zona horaria, como **2020-03-13T05:59:00-08:00**. También puede utilizar el formato de hora Zulú UTC, por ejemplo, **2018-03-13T13:59:00Z**. |
| UsePointInTimeRestore | `<SwitchParameter>` | Use el modo de un momento dado para restaurar. |

Cuando se restaura un servidor a un momento dado anterior, se crea un nuevo servidor. El servidor de origen y las bases de datos de ese momento dado anterior se copian en el servidor nuevo.

Los valores de ubicación y plan de tarifa del servidor restaurado son los mismos que los del servidor de origen.

Una vez finalizada la restauración, busque el servidor nuevo y compruebe que los datos se restauraron según lo previsto. El nuevo servidor tiene el mismo nombre de inicio de sesión y contraseña de administrador del servidor que el servidor existente tenía cuando se inició la restauración. La contraseña se puede cambiar en la página **Información general** del nuevo servidor.

El servidor creado durante una restauración no tiene los puntos de conexión de servicio de red virtual que existían en el servidor original. Estas reglas deben configurarse por separado para el nuevo servidor. Se restauran las reglas de firewall del servidor original.

## <a name="geo-restore"></a>Restauración geográfica

Si ha configurado el servidor para copias de seguridad con redundancia geográfica, se puede crear un nuevo servidor a partir de la copia de seguridad del servidor existente. Este nuevo servidor puede crearse en cualquier región en la que Azure Database for MySQL esté disponible.

Para crear un servidor con una copia de seguridad con redundancia geográfica, use el comando `Restore-AzMySqlServer` con el parámetro **UseGeoRestore**.

> [!NOTE]
> Al crear por primera vez un servidor, puede que no esté disponible para la restauración geográfica inmediatamente. Los metadatos pueden tardar unas horas en rellenarse.

Para restaurar geográficamente el servidor, ejecute el ejemplo siguiente desde PowerShell:

```azurepowershell-interactive
Get-AzMySqlServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  Restore-AzMySqlServer -Name mydemoserver-georestored -ResourceGroupName myresourcegroup -Location eastus -Sku GP_Gen5_8 -UseGeoRestore
```

Este ejemplo crea un nuevo servidor denominado **mydemoserver georestored** en la zona horaria del Este de EE. UU. que pertenece a **myresourcegroup**. Se trata de un servidor Gen 5 de uso general con ocho núcleos virtuales. El servidor se crea a partir de la copia de seguridad con redundancia geográfica de **mydemoserver**, que también está en el grupo de recursos **myresourcegroup**.

Para crear el nuevo servidor en otro grupo de recursos desde el servidor existente, especifique el nombre del nuevo grupo de recursos mediante el parámetro **ResourceGroupName** como en el ejemplo siguiente:

```azurepowershell-interactive
Get-AzMySqlServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  Restore-AzMySqlServer -Name mydemoserver-georestored -ResourceGroupName newresourcegroup -Location eastus -Sku GP_Gen5_8 -UseGeoRestore
```

El conjunto de parámetros **GeoRestore** del cmdlet `Restore-AzMySqlServer` requiere los parámetros siguientes:

| Configuración | Valor sugerido | Descripción  |
| --- | --- | --- |
|ResourceGroupName | myresourcegroup | Nombre del grupo de recursos al que pertenece el nuevo servidor.|
|Nombre | mydemoserver-georestored | Nombre del nuevo servidor. |
|Location | estado | Ubicación del nuevo servidor. |
|UseGeoRestore | `<SwitchParameter>` | Use el modo geográfico para restaurar. |

Al crear un nuevo servidor mediante una restauración geográfica, hereda el mismo tamaño de almacenamiento y plan de tarifa que el servidor de origen, a menos que se especifique el parámetro **Sku**.

Una vez finalizada la restauración, busque el servidor nuevo y compruebe que los datos se restauraron según lo previsto. El nuevo servidor tiene el mismo nombre de inicio de sesión y contraseña de administrador del servidor que el servidor existente tenía cuando se inició la restauración. La contraseña se puede cambiar en la página **Información general** del nuevo servidor.

El servidor creado durante una restauración no tiene los puntos de conexión de servicio de red virtual que existían en el servidor original. Estas reglas deben configurarse por separado para este nuevo servidor. Se restauran las reglas de firewall del servidor original.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Generación de una cadena de conexión de Azure Database for MySQL con PowerShell](howto-connection-string-powershell.md)
