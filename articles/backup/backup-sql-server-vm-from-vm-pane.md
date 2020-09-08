---
title: Copia de seguridad de una máquina virtual con SQL Server desde el panel de la máquina virtual
description: En este artículo, aprenderá a realizar copias de seguridad de bases de datos de SQL Server en máquinas virtuales de Azure desde el panel de la máquina virtual.
ms.topic: conceptual
ms.date: 08/13/2020
ms.openlocfilehash: 4f4ea202ee96e93a621c8dd0025c9ebc8b8d445d
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88891664"
---
# <a name="back-up-a-sql-server-from-the-vm-pane"></a>Copia de seguridad de SQL Server desde el panel de la máquina virtual

En este artículo se explica cómo realizar copias de seguridad de SQL Server cuando se ejecuta en máquinas virtuales de Azure con el servicio [Azure Backup](backup-overview.md). Puede realizar una copia de seguridad de máquinas virtuales de SQL Server mediante dos métodos:

- Una sola máquina virtual de Azure con SQL Server: en las instrucciones de este artículo se describe cómo realizar una copia de seguridad de una VM con SQL Server directamente desde la vista de la máquina virtual.
- Varias máquinas virtuales de Azure con SQL Server: puede configurar un almacén de Recovery Services y configurar la copia de seguridad de varias máquinas virtuales. Siga las instrucciones de [este artículo](backup-sql-server-database-azure-vms.md) para este escenario.

## <a name="before-you-start"></a>Antes de comenzar

1. Compruebe el entorno con la [matriz de compatibilidad](sql-support-matrix.md).
2. Obtenga [información general](backup-azure-sql-database.md) acerca de Azure Backup para una VM con SQL Server.
3. Compruebe que la máquina virtual tenga [conectividad de red](backup-sql-server-database-azure-vms.md#establish-network-connectivity).

## <a name="configure-backup-on-the-sql-server"></a>Configuración de la copia de seguridad en SQL Server

Puede habilitar la copia de seguridad en la VM con SQL Server desde el panel de **copia de seguridad** de la máquina virtual. Este método consigue dos cosas:

- Registra la máquina virtual de SQL en el servicio Azure Backup para concederle acceso.
- Protege todas las instancias de SQL Server que se ejecutan dentro de la máquina virtual. Esto significa que la directiva de copia de seguridad se aplica a todas las bases de datos existentes, así como a las bases de datos que se agregarán a estas instancias en el futuro.

1. Seleccione el banner de la parte superior de la página para abrir la vista de copia de seguridad de SQL Server.

    ![Vista de copia de seguridad de SQL Server](./media/backup-sql-server-vm-from-vm-pane/sql-server-backup-view.png)

    >[!NOTE]
    >¿No ve el banner? El banner solo se muestra para las máquinas virtuales con SQL Server que se crean mediante imágenes de Azure Marketplace. Además, se muestra para las máquinas virtuales que están protegidas con la copia de seguridad de la máquina virtual de Azure. Para otras imágenes, puede configurar la copia de seguridad como se explica [aquí](backup-sql-server-database-azure-vms.md).

2. Escriba el nombre del almacén de Recovery Services. Un almacén es una entidad lógica para almacenar y administrar todas las copias de seguridad. Si crea un nuevo almacén:

    - Se creará en la misma suscripción y región que la VM con SQL Server que está protegiendo.
    - Se creará con la configuración de almacenamiento con redundancia geográfica (GRS) para todas las copias de seguridad. Si desea cambiar el tipo de redundancia, debe hacerlo antes de proteger la máquina virtual. Para obtener más información, consulte [este artículo](backup-create-rs-vault.md#set-storage-redundancy).

3. Elija una **directiva de copia de seguridad**. Puede elegir una de las directivas predeterminadas o cualquier otra existente que haya creado en el almacén. Si desea crear una nueva directiva, puede consultar [este artículo](backup-sql-server-database-azure-vms.md#create-a-backup-policy) para obtener una guía paso a paso.

    ![Elección de una directiva de copia de seguridad](./media/backup-sql-server-vm-from-vm-pane/backup-policy.png)

4. Seleccione **Habilitar copia de seguridad**. La operación puede tardar varios minutos en completarse.

    ![Selección de la opción de habilitar copia de seguridad](./media/backup-sql-server-vm-from-vm-pane/enable-backup.png)

5. Una vez completada la operación, verá el **nombre del almacén** en el banner.

    ![El nombre del almacén aparece en el banner.](./media/backup-sql-server-vm-from-vm-pane/vault-name.png)

6. Seleccione el banner para ir a la vista de almacén, donde puede ver todas las máquinas virtuales registradas y su estado de protección.

    ![Vista de almacén con máquinas virtuales registradas](./media/backup-sql-server-vm-from-vm-pane/vault-view.png)

7. En el caso de las imágenes que no son del Marketplace, el registro puede realizarse correctamente, pero es posible que la **configuración de la copia de seguridad** no se desencadene hasta que la extensión de Azure Backup tenga permiso en SQL Server. En tales casos, la columna **Preparación para la copia de seguridad** presenta el mensaje **No está listo**. Debe [asignar los permisos adecuados](backup-azure-sql-database.md#set-vm-permissions) manualmente para las imágenes que no son del Marketplace para que se desencadene la opción de configuración de la copia de seguridad.

    ![La preparación para la copia de seguridad no está lista.](./media/backup-sql-server-vm-from-vm-pane/backup-readiness-not-ready.png)

8. Para otras operaciones o supervisiones que necesite hacer en la copia de seguridad de la VM con SQL Server, vaya al almacén de Recovery Services correspondiente. Vaya a **Elementos de copia de seguridad** para ver todas las bases de datos de las que se ha realizado una copia de seguridad en este almacén y desencadenar operaciones como copia de seguridad y restauración a petición. Del mismo modo, vaya a **Trabajos de copia de seguridad** para [supervisar](manage-monitor-sql-database-backup.md) trabajos correspondientes a operaciones como la configuración de la protección, la copia de seguridad y la restauración.

    ![Visualización de copias de seguridad de las bases de datos en elementos de copia de seguridad](./media/backup-sql-server-vm-from-vm-pane/backup-items.png)

>[!NOTE]
>La copia de seguridad no se configura automáticamente en ninguna de las nuevas instancias de SQL Server que se puedan agregar posteriormente a la máquina virtual protegida. Para configurar la copia de seguridad en las instancias recién agregadas, debe ir al almacén en el que está registrada la máquina virtual y seguir los pasos indicados [aquí](backup-sql-server-database-azure-vms.md).

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo:

- [Restauración de bases de datos de SQL Server con copia de seguridad](restore-sql-database-azure-vm.md)
- [Administración de bases de datos de SQL Server con copia de seguridad](manage-monitor-sql-database-backup.md)
