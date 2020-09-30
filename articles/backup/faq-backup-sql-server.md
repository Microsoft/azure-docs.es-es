---
title: 'Preguntas más frecuentes: copia de seguridad de bases de datos de SQL Server en máquinas virtuales de Azure'
description: Busque las respuestas a preguntas habituales sobre la copia de seguridad de bases de datos de SQL Server en máquinas virtuales de Azure con Azure Backup.
ms.reviewer: vijayts
ms.topic: conceptual
ms.date: 04/23/2019
ms.openlocfilehash: c1efbc44ae1e4bea96c063e4dc734f1cc53a42ac
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89180106"
---
# <a name="faq-about-sql-server-databases-that-are-running-on-an-azure-vm-backup"></a>Preguntas más frecuentes sobre las bases de datos de SQL Server que se ejecutan en una copia de seguridad de máquina virtual de Azure

Este artículo responde preguntas comunes sobre las copias de seguridad de bases de datos de SQL Server que se ejecutan en máquinas virtuales de Azure con el servicio [Azure Backup](backup-overview.md).

## <a name="can-i-use-azure-backup-for-iaas-vm-as-well-as-sql-server-on-the-same-machine"></a>¿Puedo usar Azure Backup para máquinas virtuales de IaaS y SQL Server en el mismo equipo?

Sí, puede tener la copia de seguridad de máquina virtual y la copia de seguridad de SQL en la misma máquina virtual. En este caso, se desencadena internamente una copia de seguridad completa de solo copia en la máquina virtual no truncar los registros.

## <a name="does-the-solution-retry-or-auto-heal-the-backups"></a>¿La solución reintenta o recupera automáticamente las copias de seguridad?

En algunas circunstancias, el servicio de Azure Backup desencadena copias de seguridad de corrección. La recuperación automática puede ocurrir si se cumple cualquiera de las seis condiciones que se mencionan a continuación:

- Si la copia de seguridad diferencial o de registro produce un error debido a un error de validación de LSN, la siguiente copia de seguridad diferencial o de registro se convierte en su lugar en una copia de seguridad completa.
- Si no se produjo ninguna copia de seguridad completa antes de la copia de seguridad diferencial o de registro, esa copia de seguridad diferencial o de registro se convierte en su lugar en una copia de seguridad completa.
- Si el momento específico de la copia de seguridad completa más reciente es anterior a 15 días, la siguiente copia de seguridad diferencial o de registro se convierte en su lugar en una copia de seguridad completa.
- Todos los trabajos de copia de seguridad que se cancelen debido a una actualización de la extensión se vuelven a desencadenar una vez completada la actualización e iniciada la extensión.
- Si decide sobrescribir la base de datos durante la restauración, se produce un error en la siguiente copia de seguridad diferencial o de registro y en su lugar se desencadena una copia de seguridad completa.
- En los casos en los que se requiera una copia de seguridad completa para restablecer las cadenas de registro debido a un cambio en el modelo de recuperación de la base de datos, se desencadena una completa automáticamente en la siguiente programación.

La reparación automática como capacidad está habilitada para todos los usuarios de manera predeterminada. Sin embargo, si decide no participar en ella, siga estos pasos:

- En la instancia de SQL Server, en la carpeta *C:\Archivos de programa\Azure Workload Backup\bin*, cree o edite el archivo **ExtensionSettingsOverrides.json**.
- En **ExtensionSettingsOverrides.json**, establezca *{"EnableAutoHealer": false}* .
- Guarde los cambios y cierre el archivo.
- En la instancia de SQL Server, abra el **Administrador de tareas** y, a continuación, reinicie el servicio **AzureWLBackupCoordinatorSvc**.

## <a name="can-i-control-how-many-concurrent-backups-run-on-the-sql-server"></a>¿Puedo controlar cuántas copias de seguridad simultáneas se ejecutan en el servidor de SQL Server?

Sí. Puede limitar la velocidad a la que se ejecuta la directiva de copia de seguridad para minimizar el impacto en una instancia de SQL Server. Para cambiar la configuración:

1. En la instancia de SQL Server, en la carpeta *C:\Archivos de programa\Azure Workload Backup\bin*, cree el archivo *ExtensionSettingsOverrides.json*.
2. En el archivo *ExtensionSettingsOverrides.json*, cambie el valor de **DefaultBackupTasksThreshold** por uno inferior (por ejemplo, 5). <br>
  `{"DefaultBackupTasksThreshold": 5}`
<br>
El valor predeterminado de DefaultBackupTasksThreshold es **20**.

3. Guarde los cambios y cierre el archivo.
4. En la instancia de SQL Server, abra el **Administrador de tareas**. Reinicie el servicio **AzureWLBackupCoordinatorSvc**.<br/> <br/>
 Aunque este método resulta útil si la aplicación está consumiendo muchos recursos, [Resource Governor](/sql/relational-databases/resource-governor/resource-governor) de SQL Server es una forma más genérica de especificar límites en la cantidad de CPU, E/S física y memoria que las solicitudes entrantes de la aplicación pueden utilizar.

> [!NOTE]
> En la experiencia de usuario puede seguir avanzando y programando tantas copias de seguridad como quiera en un momento dado. Sin embargo, se procesarán en una ventana deslizante (por ejemplo, de cinco, de acuerdo con el ejemplo anterior).

## <a name="can-i-run-a-full-backup-from-a-secondary-replica"></a>¿Puedo ejecutar una copia de seguridad completa desde una réplica secundaria?

Según las limitaciones de SQL, puede ejecutar copias de seguridad completas de solo copia en la réplica secundaria. Sin embargo, no se permite la copia de seguridad completa.

## <a name="can-i-protect-availability-groups-on-premises"></a>¿Puedo proteger los grupos de disponibilidad locales?

No. Azure Backup protege las bases de datos de SQL Server que se ejecutan en Azure. Si se distribuye un grupo de disponibilidad entre máquinas locales y de Azure, se podrá proteger este solo si la réplica principal se ejecuta en Azure. Además, Azure Backup protege solo los nodos que se ejecutan en la misma región de Azure que el almacén de Recovery Services.

## <a name="can-i-protect-availability-groups-across-regions"></a>¿Puedo proteger los grupos de disponibilidad entre regiones?

El almacén de Azure Backup Recovery Services puede detectar y proteger todos los nodos que se encuentran en la misma región que el almacén. Si su grupo de disponibilidad Always On de SQL Server abarca varias regiones de Azure, configure la copia de seguridad de la región que tenga el nodo principal. Azure Backup puede detectar todas las bases de datos del grupo de disponibilidad en función de sus preferencias con respecto a la copia de seguridad y protegerlas. Cuando no se cumple la preferencia de copia de seguridad, se produce un error en las copias de seguridad y se emite la alerta de error.

## <a name="do-successful-backup-jobs-create-alerts"></a>¿Generan alertas los trabajos de copia de seguridad que se han realizado correctamente?

No. Los trabajos de copia de seguridad correctos no generan alertas. Las alertas se envían únicamente para los trabajos de copia de seguridad con errores. El comportamiento detallado para alertas del portal está documentado [aquí](backup-azure-monitoring-built-in-monitor.md). Sin embargo, si le interesa recibir alertas incluso para trabajos realizados correctamente, puede usar [Supervisión mediante Azure Monitor](backup-azure-monitoring-use-azuremonitor.md).

## <a name="can-i-see-scheduled-backup-jobs-in-the-backup-jobs-menu"></a>¿Se pueden ver los trabajos de copia de seguridad programados en el menú de Trabajos de copia de seguridad?

En el menú de **Trabajo de copia de seguridad** solo se mostrarán los trabajos de copia de seguridad a petición. Para el trabajo programado, utilice la [Supervisión mediante Azure Monitor](backup-azure-monitoring-use-azuremonitor.md).

## <a name="are-future-databases-automatically-added-for-backup"></a>¿Las bases de datos futuras se agregan automáticamente para copia de seguridad?

Sí, puede obtener esta funcionalidad con la [protección automática](backup-sql-server-database-azure-vms.md#enable-auto-protection).  

## <a name="if-i-delete-a-database-from-an-autoprotected-instance-what-will-happen-to-the-backups"></a>Si elimino una base de datos de una instancia protegida automática, ¿qué ocurrirá con las copias de seguridad?

Si se quita una base de datos de una instancia protegida automáticamente, se siguen intentando las copias de seguridad de la base de datos. Esto implica que la base de datos eliminada comienza a aparecer como incorrecta en **Elementos de copia de seguridad** y todavía está protegida.

La manera correcta de detener la protección de esta base de datos es realizar una acción **Detener copia de seguridad** con **eliminación de datos** en esta base de datos.  

## <a name="if-i-do-stop-backup-operation-of-an-autoprotected-database-what-will-be-its-behavior"></a>Si se detiene la operación de copia de seguridad de una base de datos protegida automáticamente, ¿cuál será su comportamiento?

Si realiza una acción **Detener copia de seguridad con retención de datos**, no se realizarán copias de seguridad futuras y los puntos de recuperación existentes permanecerán intactos. La base de datos se seguirá considerando como protegido y se mostrará en los **Elementos de copia de seguridad**.

Si realiza una acción **Detener copia de seguridad con eliminación de datos**, no se realizarán copias de seguridad futuras y los puntos de recuperación existentes también se eliminarán. La base de datos se seguirá considerando como desprotegida y se mostrará en la instancia de Configurar copia de seguridad. Sin embargo, a diferencia de otras bases de datos desprotegidas que se pueden seleccionar manualmente o que pueden protegerse automáticamente, esta base de datos aparece atenuada y no se pueden seleccionar. La única forma de volver a proteger esta base de datos es deshabilitar la protección automática en la instancia. Ahora puede seleccionar esta base de datos y configurar la protección o volver a habilitar la protección automática en la instancia de nuevo.

## <a name="if-i-change-the-name-of-the-database-after-it-has-been-protected-what-will-be-the-behavior"></a>Si se cambia el nombre de la base de datos después de haberla protegido, ¿cuál será su comportamiento?

Una base de datos después de un cambio de nombre se trata como una nueva base de datos. Por lo tanto, el servicio trata esta situación como si no se encontrara la base de datos y producirá un error en las copias de seguridad.

Puede seleccionar la base de datos, que ahora tiene un nuevo nombre y configurar la protección para la misma. Si la protección automática está habilitada en la instancia, la base de datos con un nuevo nombre se detectará y protegerá automáticamente.

## <a name="why-cant-i-see-an-added-database-for-an-autoprotected-instance"></a>¿Por qué no puedo ver una base de datos agregada para una instancia protegida automáticamente?

Una base de datos que [se agrega a una instancia protegida automáticamente](backup-sql-server-database-azure-vms.md#enable-auto-protection) puede no aparecer inmediatamente entre los elementos protegidos. Esto se debe a que la detección normalmente se ejecuta cada ocho horas. Sin embargo, puede detectar y proteger nuevas bases de datos de inmediato si ejecuta manualmente una detección seleccionando **Volver a detectar bases de datos**, tal como se muestra en la siguiente imagen:

  ![Detectar manualmente una base de datos recién agregada](./media/backup-azure-sql-database/view-newly-added-database.png)

## <a name="next-steps"></a>Pasos siguientes

Aprenda a [hacer una copia de seguridad de una base de datos de SQL Server](backup-azure-sql-database.md) que se ejecuta en una máquina virtual de Azure.
