---
title: Copia de seguridad de máquinas Windows con el agente de MARS
description: Use el agente de Microsoft Azure Recovery Services (MARS) para realizar copias de seguridad de máquinas Windows.
ms.topic: conceptual
ms.date: 03/03/2020
ms.openlocfilehash: 55eb8bdf069fd1b5793468ae52f764eb4288f261
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89182095"
---
# <a name="back-up-windows-server-files-and-folders-to-azure"></a>Copia de seguridad de carpetas y archivos de Windows Server en Azure

En este artículo se explica cómo realizar copias de seguridad de máquinas Windows mediante el servicio [Azure Backup](backup-overview.md) y el agente de Microsoft Azure Recovery Services (MARS), también conocido como agente Azure Backup. MARS también se conoce como agente Azure Backup.

En este artículo, aprenderá a:

> [!div class="checklist"]
>
> * Comprobar los requisitos previos
> * Crear una directiva y programación de copia de seguridad
> * Realizar una copia de seguridad a petición

## <a name="before-you-start"></a>Antes de comenzar

* Obtenga información acerca de cómo [Azure Backup usa el agente de MARS para realizar copias de seguridad de máquinas Windows](backup-architecture.md#architecture-direct-backup-of-on-premises-windows-server-machines-or-azure-vm-files-or-folders).
* Obtenga información acerca de la [arquitectura de copia de seguridad](backup-architecture.md#architecture-back-up-to-dpmmabs) que ejecuta el agente de MARS en un servidor MABS secundario o un servidor Data Protection Manager.
* Revise [qué admite y de qué puede hacer una copia de seguridad](backup-support-matrix-mars-agent.md) el agente de MARS.
* [Compruebe el acceso a Internet](install-mars-agent.md#verify-internet-access) en las máquinas de las que desee realizar copias de seguridad.
* Si el agente de MARS no está instalado, obtenga información sobre cómo instalarlo [aquí](install-mars-agent.md).

## <a name="create-a-backup-policy"></a>Crear una directiva de copia de seguridad

La directiva de copia de seguridad especifica cuándo tomar instantáneas de los datos para crear puntos de recuperación. También especifica cuánto tiempo se conservan los puntos de recuperación. El agente de MARS se usa para configurar una directiva de copia de seguridad.

Azure Backup no tiene en cuenta automáticamente el horario de verano. Esta configuración predeterminada podría producir una discrepancia entre la hora real y la hora programada para la copia de seguridad.

Para crear una directiva de copia de seguridad:

1. Después de descargar y registrar el agente de MARS, abra la consola del agente. Para encontrarlo, busque **Microsoft Azure Backup**en la máquina.  

1. En **Acciones**, seleccione **Programar copia de seguridad**.

    ![Programar una copia de seguridad de Windows Server](./media/backup-configure-vault/schedule-first-backup.png)
1. En el Asistente para programar copias de seguridad, seleccione **Introducción** > **Siguiente**.
1. En **Seleccionar los elementos de los que se debe realizar una copia de seguridad**, seleccione **Agregar elementos**.

    ![Agregar elementos para la copia de seguridad](./media/backup-azure-manage-mars/select-item-to-backup.png)

1. En el cuadro **Seleccionar elementos**, seleccione los elementos de los que desea realizar una copia de seguridad y, a continuación, seleccione **Aceptar**.

    ![Seleccionar elementos de los que se va a hacer una copia de seguridad](./media/backup-azure-manage-mars/selected-items-to-backup.png)

1. En la página **Seleccionar los elementos de los que se debe realizar una copia de seguridad**, seleccione **Siguiente**.
1. En la página **Especifique la programación de copia de seguridad**, especifique cuándo desea realizar copias de seguridad diarias o semanales. Luego, seleccione **Siguiente**.

    * Cuando se realiza una copia de seguridad, se crea un punto de recuperación.
    * El número de puntos de recuperación creados en el entorno depende de la programación de copia de seguridad.
    * Puede programar hasta tres copias de seguridad diarias al día. En el ejemplo siguiente, se realizan dos copias de seguridad diarias, una a medianoche y otra a las 6:00 p. m.

        ![Configuración de una programación de copia de seguridad diaria](./media/backup-configure-vault/day-schedule.png)

    * También puede ejecutar copias de seguridad semanales. En el ejemplo siguiente, se hacen copias de seguridad los domingos y miércoles alternativos a las 9:30 a. m. y a la 1:00 a. m.

        ![Configuración de una programación de copia de seguridad semanal](./media/backup-configure-vault/week-schedule.png)

1. En la página **Seleccionar directiva de retención**, especifique cómo desea almacenar las copias históricas de los datos. Luego, seleccione **Siguiente**.

    * La configuración de retención especifica qué puntos de recuperación se almacenan y durante cuánto tiempo.
    * Con una configuración de retención diaria se indica que, en el momento especificado para la retención diaria, se conservará el punto de recuperación más reciente durante el número de días especificado. También puede especificar una directiva de retención mensual para indicar que el punto de recuperación creado el 30 de cada mes debe almacenarse durante 12 meses.
    * La retención diaria y semanal de los puntos de recuperación normalmente coincide con la programación de copia de seguridad. Por tanto, cuando la programación desencadena una copia de seguridad, el punto de recuperación que la crea se almacena durante el tiempo que especifica la directiva de retención diaria o semanal.
    * En el ejemplo siguiente:

        * Las copias de seguridad diarias a medianoche ya la 6:00 p. m. se conservan durante siete días.
        * Las copias de seguridad realizadas un sábado a medianoche y a las 6:00 p. m. se conservan durante cuatro semanas.
        * Las copias de seguridad tomadas el sábado del mes a medianoche y a las 6:00 p. m. se conservan durante 12 meses.
        * Las copias de seguridad tomadas el último sábado de marzo se conservan durante 10 años.

        ![Ejemplo de una directiva de retención](./media/backup-configure-vault/retention-example.png)

1. En la página **Elija el tipo de copia de seguridad inicial**, decida si desea realizar la copia de seguridad inicial a través de la red o utilizar la copia de seguridad sin conexión. Para realizar la copia de seguridad inicial a través de la red, seleccione **Automáticamente a través de la red** > **Siguiente**.

    Para más información sobre la copia de seguridad sin conexión, vea [Copia de seguridad sin conexión con Azure Data Box](offline-backup-azure-data-box.md).

    ![Elección de un tipo de copia de seguridad inicial](./media/backup-azure-manage-mars/choose-initial-backup-type.png)

1. En la página **Confirmación**, revise la información y, luego, seleccione **Finalizar**.

    ![Confirmación del tipo de copia de seguridad](./media/backup-azure-manage-mars/confirm-backup-type.png)

1. Cuando el asistente termine de crear la programación de copia de seguridad, seleccione **Cerrar**.

    ![Visualización del progreso de la programación de copia de seguridad](./media/backup-azure-manage-mars/confirm-modify-backup-process.png)

Debe crear una directiva en cada máquina en la que esté instalado el agente.

### <a name="do-the-initial-backup-offline"></a>Realización de la copia de seguridad inicial sin conexión

Puede ejecutar una copia de seguridad inicial automáticamente a través de la red o sin conexión. La propagación sin conexión de una copia de seguridad inicial resulta útil si tiene grandes cantidades de datos que requerirán una gran cantidad de ancho de banda de red para transferirse.

Para realizar una transferencia sin conexión:

1. Escriba los datos de copia de seguridad en una ubicación de almacenamiento provisional.
1. Use la herramienta AzureOfflineBackupDiskPrep para copiar los datos de la ubicación de almacenamiento provisional en uno o más discos SATA.

    La herramienta crea un trabajo de importación de Azure. Para más información, consulte [¿Qué es el servicio Azure Import/Export?](../storage/common/storage-import-export-service.md).
1. Envíe los discos SATA a un centro de datos de Azure.

    En el centro de datos, los datos de los discos se copian en una cuenta de almacenamiento de Azure. Azure Backup copia los datos de la cuenta de almacenamiento en el almacén y se programan copias de seguridad incrementales.

Para más información sobre la propagación sin conexión, consulte [Copia de seguridad sin conexión con Azure Data Box](offline-backup-azure-data-box.md).

### <a name="enable-network-throttling"></a>Habilitación de la limitación de la red

Puede controlar el modo en que el agente de MARS usa el ancho de banda de red si habilita un límite de ancho de banda. El límite es útil si tiene que realizar una copia de seguridad de los datos durante las horas de trabajo, pero desea controlar la cantidad de ancho de banda que se usa para la actividad de copia de seguridad y restauración.

El límite de ancho de banda de red de Azure Backup utiliza [Calidad de servicio (QoS)](/windows-server/networking/technologies/qos/qos-policy-top) en el sistema operativo local.

El límite de ancho de banda de red para las copias de seguridad está disponible en Windows Server 2012 y versiones posteriores, y en Windows 8 y versiones posteriores. Los sistemas operativos deben ejecutar los Service Pack más recientes.

Para habilitar el límite de ancho de banda de red:

1. En el agente de MARS, seleccione **Cambiar propiedades**.
1. En la pestaña **Limitación**, seleccione la opción **Habilitar el límite de uso del ancho de banda de Internet para operaciones de copia de seguridad**.

    ![Configuración del límite de ancho de banda de red para las operaciones de copia de seguridad](./media/backup-configure-vault/throttling-dialog.png)
1. Especifique el ancho de banda permitido durante el horario laboral y fuera de él. Los valores de ancho de banda comienzan en 512 Kbps y llegan hasta 1023 MBps. Después, seleccione **Aceptar**.

## <a name="run-an-on-demand-backup"></a>Ejecución de una copia de seguridad a petición

1. En el agente de MARS, seleccione **Hacer copia ahora**.

    ![Copia de seguridad ahora en Windows Server](./media/backup-configure-vault/backup-now.png)

1. Si la versión del agente de MARS es la 2.0.9169.0 o posterior, puede establecer una fecha de retención personalizada. En la sección **Conservar la copia de seguridad hasta**, elija una fecha en el calendario.

   ![Uso del calendario para personalizar una fecha de retención](./media/backup-configure-vault/mars-ondemand.png)

1. En la página **Confirmación**, revise la configuración y seleccione **Copia de seguridad**.
1. Seleccione **Cerrar** para cerrar el asistente. Si lo hace antes de que finalice la copia de seguridad, el asistente se sigue ejecutando en segundo plano.

Una vez que finalice la copia de seguridad inicial, el estado **Trabajo completado** se refleja en la consola de Backup.

## <a name="set-up-on-demand-backup-policy-retention-behavior"></a>Configuración del comportamiento de la retención de la directiva de copia de seguridad a petición

> [!NOTE]
> Esta información solo se aplica a las versiones de agente de MARS anteriores a 2.0.9169.0.
>

| Opción de programación de copia de seguridad | Duración de la retención de datos
| -- | --
| Día | **Retención predeterminada**: Equivalente a "retención en días para copias de seguridad diarias". <br/><br/> **Excepción**: si se produce un error en una copia de seguridad programada diariamente establecida para retención a largo plazo (semanas, meses o años), se desencadena una copia de seguridad a petición justo después de que el error se considere para la retención a largo plazo. De lo contrario, se considerará la siguiente copia de seguridad programada para la retención a largo plazo.<br/><br/> **Escenario de ejemplo**: se produjo un error en la copia de seguridad programada el jueves a las 8:00 a. m. Esta copia de seguridad se debía considerar para la retención semanal, mensual o anual. Por lo tanto, la primera copia de seguridad a petición desencadenada antes de la siguiente programada el viernes a las 8:00 a. m. se etiqueta automáticamente para la retención semanal, mensual o anual. Esta copia de seguridad sustituye a la copia del jueves 8:00 a. m.
| Semana | **Retención predeterminada**: un día. Las copias de seguridad a petición realizadas para un origen de datos que tiene una directiva de copia de seguridad semanal se eliminan al día siguiente. Se eliminan incluso si son las más recientes del origen de datos. <br/><br/> **Excepción**: si se produce un error en una copia de seguridad programada semanalmente establecida para retención a largo plazo (semanas, meses o años), se desencadena una copia de seguridad a petición justo después de que se considere el error para la retención a largo plazo. De lo contrario, se considerará la siguiente copia de seguridad programada para la retención a largo plazo. <br/><br/> **Escenario de ejemplo**: se produjo un error en la copia de seguridad programada el jueves a las 8:00 a. m. Esta copia de seguridad se debe considerar para la retención mensual o anual. Por lo tanto, la primera copia de seguridad a petición que se desencadene antes de la siguiente programada el jueves a las 8:00 a. m. se etiqueta automáticamente para la retención mensual o anual. Esta copia de seguridad sustituye a la copia del jueves 8:00 a. m.

Para más información, consulte [Creación de una directiva de copia de seguridad](#create-a-backup-policy).

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [restaurar archivos en Azure](backup-azure-restore-windows-server.md).
* Busque [Preguntas comunes acerca de la realización de copias de seguridad de archivos y carpetas](backup-azure-file-folder-backup-faq.md).
