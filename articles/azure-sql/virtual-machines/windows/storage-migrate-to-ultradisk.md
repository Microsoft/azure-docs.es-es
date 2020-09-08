---
title: Migración del disco de registro al disco Ultra
description: Obtenga información sobre cómo migrar su servidor SQL Server en un disco de registro de Azure Virtual Machine (VM) a un disco Ultra de Azure para aprovechar las ventajas de un alto rendimiento y una baja latencia.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-sql
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 07/09/2020
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 42738ff71432284a156d0dfbb1f6cf160cbf4032
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88653261"
---
# <a name="migrate-log-disk-to-ultra-disk"></a>Migración del disco de registro al disco Ultra
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Los discos Ultra de Azure ofrecen un alto rendimiento, un número elevado de IOPS y un almacenamiento en disco coherente y de baja latencia para SQL Server en Azure Virtual Machine (VM). 

En este artículo se explica cómo migrar el disco de registro a un SSD Ultra para aprovechar las ventajas de rendimiento que ofrecen los discos Ultra. 

## <a name="back-up-database"></a>Copia de seguridad de base de datos

Haga una [copia de seguridad completa](backup-restore.md) de la base de datos. 

## <a name="attach-disk"></a>Conexión de disco

Conecte el SSD Ultra a la máquina virtual una vez que haya habilitado la compatibilidad con discos Ultra en la máquina virtual. 

El disco Ultra es compatible con un subconjunto de regiones y tamaños de máquinas virtuales. Antes de continuar, compruebe que la máquina virtual se encuentra en una región, zona y tamaño que admite el disco Ultra. Puede [determinar y validar el tamaño de la máquina virtual y la región](../../../virtual-machines/disks-enable-ultra-ssd.md#determine-vm-size-and-region-availability) mediante la CLI de Azure o PowerShell. 

### <a name="enable-compatibility"></a>Habilitación de la compatibilidad

Para habilitar la compatibilidad, siga estos pasos:

1. Vaya a la máquina virtual en [Azure Portal](https://portal.azure.com/). 
1. Detenga/desasigne la máquina virtual. 
1. Seleccione **Discos** en **Configuración** y, a continuación, seleccione **Configuración adicional**. 

   :::image type="content" source="media/storage-migrate-to-ultradisk/additional-disks-settings-azure-portal.png" alt-text="Selección de la configuración adicional para Discos en Configuración en Azure Portal":::

1. Seleccione **Sí** para **Habilitar la compatibilidad con discos Ultra**. 

   :::image type="content" source="../../../virtual-machines/media/virtual-machines-disks-getting-started-ultra-ssd/ultra-options-yes-enable.png" alt-text="Selección de configuración adicional para Discos en Configuración en Azure Portal":::

1. Seleccione **Guardar**. 



### <a name="attach-disk"></a>Conexión de disco

Use Azure Portal para conectar un disco Ultra a la máquina virtual. Para obtener más información, consulte [Adjuntar un disco Ultra](../../../virtual-machines/disks-enable-ultra-ssd.md#attach-an-ultra-disk-using-the-azure-portal).

Una vez conectado el disco, inicie la máquina virtual una vez más mediante Azure Portal. 



## <a name="format-disk"></a>Aplicación de formato al disco

Conéctese a la máquina virtual y aplique formato al disco Ultra.  

Para aplicar formato al disco Ultra, siga estos pasos:

1. Conéctese a la máquina virtual mediante el Protocolo de escritorio remoto (RDP).
1. Utilice [Administración del disco](/windows-server/storage/disk-management/overview-of-disk-management) para dar formato al disco Ultra recién conectado y hacerle particiones. 


## <a name="use-disk-for-log"></a>Uso de disco para registro

Configure SQL Server para usar la nueva unidad de registro. Para ello, puede usar Transact-SQL (T-SQL) o SQL Server Management Studio (SSMS). La cuenta usada para la cuenta de servicio de SQL Server debe tener control total sobre la nueva ubicación del archivo de registro. 

### <a name="configure-permissions"></a>Configuración de permisos

1. Compruebe la cuenta de servicio usada por SQL Server. Puede hacerlo mediante el Administrador de configuración de SQL Server o Services.msc.
1. Vaya a su nuevo disco. 
1. Cree una carpeta (o varias carpetas) que se usarán para el archivo de registro. 
1. Haga clic con el botón derecho en la carpeta y seleccione **Propiedades**.
1. En la pestaña **Seguridad**, conceda acceso de control total a la cuenta de servicio de SQL Server. 
1. Seleccione **Aceptar** para guardar la configuración. 
1. Repita este paso para cada carpeta de nivel raíz donde tenga previsto tener datos SQL. 

### <a name="use-new-log-drive"></a>Uso de nueva unidad de registro 

Una vez concedido el permiso, use Transact-SQL (T-SQL) o SQL Server Management Studio (SSMS) para desasociar la base de datos y trasladar los archivos de registro existentes a la nueva ubicación.

   > [!CAUTION]
   > Al desasociar la base de datos, esta se desconectará, cerrando las conexiones y revirtiendo las transacciones sobre la marcha. Continúe con precaución y durante una ventana de mantenimiento de tiempo de inactividad. 



# <a name="transact-sql-t-sql"></a>[Transact-SQL (T-SQL)](#tab/tsql)

Use T-SQL para trasladar los archivos existentes a una nueva ubicación:

1. Conéctese a la base de datos en SQL Server Management Studio y abra una ventana de **Nueva consulta**. 
1. Obtenga los archivos y las ubicaciones existentes:

   ```sql
   USE AdventureWorks
   GO

   sp_helpfile
   GO
   ```

1. Desasocie la base de datos: 

   ```sql
   USE master
   GO

   sp_detach_db 'AdventureWorks'
   GO
   ```

1. Use el explorador de archivos para trasladar el archivo de registro a la nueva ubicación en el disco Ultra. 

1. Adjunte la base de datos, especificando las nuevas ubicaciones de archivo: 

   ```sql
    sp_attach_db 'AdventureWorks'
   'E:\Fixed_FG\AdventureWorks.mdf',
   'E:\Fixed_FG\AdventureWorks_2.ndf',
   'F:\New_Log\AdventureWorks_log.ldf'
   GO
   ```

Llegados a este punto, la base de datos pasa a estar en línea con el registro en la nueva ubicación. 



# <a name="sql-server-management-studio-ssms"></a>[SQL Server Management Studio (SSMS)](#tab/ssms)

Use SSMS para trasladar los archivos existentes a una nueva ubicación:

1. Conéctese a la base de datos en SQL Server Management Studio (SSMS). 
1. Haga clic con el botón derecho en la base de datos, seleccione **Propiedades** y, luego, **Archivos**. 
1. Anote la ruta de acceso de los archivos existentes. 
1. Seleccione **Aceptar** para cerrar el cuadro de diálogo. 
1. Haga clic con el botón derecho en la base de datos, seleccione **Tareas** > **Desasociar**. 
1. Siga el asistente para desasociar la base de datos. 
1. Use el Explorador de archivos para trasladar manualmente el archivo de registro a la nueva ubicación.
1. Asociación de la base de datos en SQL Server Management Studio
   1. Haga clic con el botón derecho en **Bases de datos** en el **Explorador de objetos** y seleccione **Adjuntar base de datos**. 
   1. Mediante el cuadro de diálogo, agregue cada archivo, incluido el archivo de registro en su nueva ubicación. 
   1. Seleccione **Aceptar** para adjuntar la base de datos. 

Llegados a este punto, la base de datos pasa a estar en línea con el registro en la nueva ubicación.

---


## <a name="next-steps"></a>Pasos siguientes

Revise las [prácticas recomendadas de rendimiento](performance-guidelines-best-practices.md) para obtener una configuración adicional para mejorar el rendimiento. 

Para obtener información general de SQL Server en Azure Virtual Machines, vea los siguientes artículos:

- [Introducción a SQL Server en máquinas virtuales Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
- [Introducción a SQL Server en máquinas virtuales Linux](../linux/sql-server-on-linux-vm-what-is-iaas-overview.md)
