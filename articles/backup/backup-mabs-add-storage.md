---
title: Uso de Modern Backup Storage con Azure Backup Server
description: Obtenga información sobre las nuevas características de Azure Backup Server. En este artículo se describe cómo actualizar la instalación de la instancia de Backup Server.
ms.topic: conceptual
ms.date: 11/13/2018
ms.openlocfilehash: 997b2052751473d095b6534ef0bcbe3ed6a099ad
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89378243"
---
# <a name="add-storage-to-azure-backup-server"></a>Adición de almacenamiento a Azure Backup Server

Azure Backup Server v2 (y versiones posteriores) es compatible con Modern Backup Storage, que proporciona ahorros de almacenamiento de hasta el 50 por ciento, copias de seguridad que son tres veces más rápidas y un almacenamiento más eficaz. También ofrece almacenamiento con reconocimiento de la carga de trabajo.

> [!NOTE]
> Para utilizar Modern Backup Storage, debe ejecutar Backup Server V2 o V3 en Windows Server 2016 o V3 en Windows Server 2019.
> Si ejecuta Backup Server V2 en una versión anterior de Windows Server, Azure Backup Server no puede sacar partido de Modern Backup Storage. En su lugar, protege las cargas de trabajo como lo hace con Backup Server V1. Para más información, consulte la [matriz de protección](backup-mabs-protection-matrix.md) de la versión de Backup Server.
>
> Para lograr los mejores rendimientos de copia de seguridad, se recomienda implementar MABS V3 con almacenamiento en capas en Windows Server 2019. Consulte el artículo de DPM "[Configuración de MBS con el almacenamiento en capas](/system-center/dpm/add-storage?view=sc-dpm-2019#set-up-mbs-with-tiered-storage)" para obtener los pasos de configuración del almacenamiento en capas.

## <a name="volumes-in-backup-server"></a>Volúmenes en Backup Server

Backup Server V2 o versiones posteriores acepta volúmenes de almacenamiento. Al agregar un volumen, Backup Server da formato al volumen en Sistema de archivos resistente (ReFS), que requiere Modern Backup Storage. Para agregar un volumen y expandirlo más adelante si es necesario, se recomienda utilizar este flujo de trabajo:

1. Configure Backup Server en una máquina virtual.
2. Cree un volumen en un disco virtual en un grupo de almacenamiento:
    1. Agregue un disco a un grupo de almacenamiento y cree un disco virtual con la distribución simple.
    2. Agregue discos adicionales y extienda el disco virtual.
    3. Cree volúmenes en el disco virtual.
3. Agregue los volúmenes a Backup Server.
4. Configure el almacenamiento con reconocimiento de la carga de trabajo.

## <a name="create-a-volume-for-modern-backup-storage"></a>Creación de un volumen para Modern Backup Storage

El uso de Backup Server V2 o versiones posteriores con volúmenes como almacenamiento en disco puede ayudarle a mantener el control sobre el almacenamiento. Un volumen puede ser un único disco. Sin embargo, si desea extender el almacenamiento en el futuro, cree un volumen de un disco creado mediante el uso de espacios de almacenamiento. Esto puede ayudarle si desea ampliar el volumen para el almacenamiento de copias de seguridad. En esta sección se ofrecen procedimientos recomendados para crear un volumen con esta configuración.

1. En Administrador del servidor, seleccione **Servicios de archivos y almacenamiento** > **Volúmenes** > **Grupos de almacenamiento**. En **DISCOS FÍSICOS**, seleccione **Nuevo grupo de almacenamiento**.

    ![Creación de un nuevo grupo de almacenamiento](./media/backup-mabs-add-storage/mabs-add-storage-1.png)

2. En el cuadro desplegable **TAREAS**, seleccione **Nuevo disco virtual**.

    ![Adición de un disco virtual](./media/backup-mabs-add-storage/mabs-add-storage-2.png)

3. Seleccione el grupo de almacenamiento y, después, seleccione **Agregar disco físico**.

    ![Adición de un disco físico](./media/backup-mabs-add-storage/mabs-add-storage-3.png)

4. Seleccione el disco físico y, después, seleccione **Extender disco virtual**.

    ![Extensión del disco virtual](./media/backup-mabs-add-storage/mabs-add-storage-4.png)

5. Seleccione el disco virtual y, después, seleccione **Nuevo volumen**.

    ![Creación de un nuevo volumen](./media/backup-mabs-add-storage/mabs-add-storage-5.png)

6. En el cuadro de diálogo **Seleccionar el servidor y el disco**, seleccione el servidor y el nuevo disco. Después, seleccione **Siguiente**.

    ![Selección del servidor y disco](./media/backup-mabs-add-storage/mabs-add-storage-6.png)

## <a name="add-volumes-to-backup-server-disk-storage"></a>Adición de volúmenes al almacenamiento de disco de Backup Server

> [!NOTE]
>
> - Agregue solo un disco al grupo para mantener el recuento de columnas en 1. Después, puede agregar discos según sea necesario posteriormente.
> - Si agrega varios discos al bloque de almacenamiento, el número de discos se almacena como el número de columnas. Cuando se agregan más discos, solo pueden ser un múltiplo del número de columnas.

Para agregar un volumen a Backup Server, en el panel **Administración**, vuelva a examinar el almacenamiento y, después, seleccione **Agregar**. Aparece una lista de todos los volúmenes disponibles para agregarse al almacenamiento de Backup Server. Después de que se hayan agregado los volúmenes disponibles a la lista de volúmenes seleccionados, puede asignarles un nombre descriptivo para facilitar su administración. Para dar formato a estos volúmenes en ReFS para que Backup Server pueda usar las ventajas de Modern Backup Storage, seleccione **Aceptar**.

![Adición de volúmenes disponibles](./media/backup-mabs-add-storage/mabs-add-storage-7.png)

## <a name="set-up-workload-aware-storage"></a>Configuración del almacenamiento con reconocimiento de la carga de trabajo

Con el almacenamiento con reconocimiento de la carga de trabajo, puede seleccionar los volúmenes que almacenen de forma preferente ciertos tipos de cargas de trabajo. Por ejemplo, puede establecer volúmenes costosos que admiten un gran número de operaciones de entrada/salida por segundo (IOPS) para almacenar solo las cargas de trabajo que requieren copias de seguridad frecuentes y de gran volumen. Un ejemplo es SQL Server con registros de transacciones. Otras cargas de trabajo que se almacenan en copias de seguridad con menos frecuencia, como máquinas virtuales, pueden ser guardadas en volúmenes de bajo coste.

### <a name="update-dpmdiskstorage"></a>Update-DPMDiskStorage

Puede configurar el almacenamiento con reconocimiento de la carga de trabajo mediante el cmdlet de PowerShell Update-DPMDiskStorage, que actualiza las propiedades de un volumen en el bloque de almacenamiento en una instancia de Azure Backup Server.

Sintaxis:

`Parameter Set: Volume`

```powershell
Update-DPMDiskStorage [-Volume] <Volume> [[-FriendlyName] <String> ] [[-DatasourceType] <VolumeTag[]> ] [-Confirm] [-WhatIf] [ <CommonParameters>]
```

En la captura de pantalla siguiente se muestra el cmdlet Update-DPMDiskStorage en la ventana de PowerShell.

![El comando Update-DPMDiskStorage en la ventana de PowerShell](./media/backup-mabs-add-storage/mabs-add-storage-8.png)

Los cambios que realice con PowerShell se reflejan en la consola de administrador de Backup Server.

![Discos y volúmenes en la consola de administrador](./media/backup-mabs-add-storage/mabs-add-storage-9.png)

## <a name="migrate-legacy-storage-to-modern-backup-storage"></a>Migrar el almacenamiento heredado a Modern Backup Storage

Después de instalar Backup Server V2 o actualizar a esta versión, y después de actualizar el sistema operativo a Windows Server 2016, actualice los grupos de protección para que usen Modern Backup Storage. De forma predeterminada, los grupos de protección no se cambian. Siguen funcionando tal como se han configurado inicialmente.

La actualización de los grupos de protección para que usen Modern Backup Storage es opcional. Para actualizar el grupo de protección, detenga la protección de todos los orígenes de datos mediante la opción de conservar datos. Después, agregue los orígenes de datos a un nuevo grupo de protección.

1. En la consola de administrador, seleccione la característica **Protección**. En la lista **Miembro del grupo de protección**, haga clic con el botón derecho en el miembro y seleccione **Detener protección de miembro**.

   ![Detener protección de miembro](/system-center/dpm/media/upgrade-to-dpm-2016/dpm-2016-stop-protection1.png)

2. En el cuadro de diálogo **Quitar del grupo**, revise el espacio en disco usado y el espacio disponible para el grupo de almacenamiento. El valor predeterminado es dejar los puntos de recuperación en el disco y permitirles expirar según su directiva de retención asociada. Seleccione **Aceptar**.

   Si quiere devolver de inmediato el espacio en disco usado al grupo de almacenamiento libre, active la casilla **Eliminar réplica en disco** para eliminar los datos de copia de seguridad (y los puntos de recuperación) asociados a ese miembro.

   ![Cuadro de diálogo Quitar del grupo](/system-center/dpm/media/upgrade-to-dpm-2016/dpm-2016-retain-data.png)

3. Cree un grupo de protección que use Modern Backup Storage. Incluya los orígenes de datos no protegidos.

## <a name="add-disks-to-increase-legacy-storage"></a>Agregar discos para aumentar el almacenamiento heredado

Si quiere usar el almacenamiento heredado con Backup Server, es posible que tenga que agregar discos para aumentar el almacenamiento heredado.

Para agregar almacenamiento en disco:

1. En la consola de administrador, seleccione **Administración** > **Almacenamiento en disco** > **Agregar**.

    ![Cuadro de diálogo Agregar almacenamiento en disco](/system-center/dpm/media/upgrade-to-dpm-2016/dpm-2016-add-disk-storage.png)

2. En el cuadro de diálogo **Agregar almacenamiento en disco**, seleccione **Agregar discos**.

3. En la lista de discos disponibles, seleccione los discos que quiera agregar y haga clic en **Agregar** y en **Aceptar**.

## <a name="next-steps"></a>Pasos siguientes

Después de instalar Backup Server, sepa cómo preparar el servidor o empezar a proteger la carga de trabajo.

- [Preparar cargas de trabajo de Backup Server](backup-azure-microsoft-azure-backup.md)
- [Usar Backup Server para hacer una copia de seguridad de un servidor de VMware](backup-azure-backup-server-vmware.md)
- [Usar Backup Server para hacer una copia de seguridad de SQL Server](backup-azure-sql-mabs.md)
