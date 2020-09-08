---
title: 'Tutorial: Recuperar elementos en Windows Server'
description: En este tutorial, aprenderá a usar el agente de Microsoft Azure Recovery Services Agent (MARS) para recuperar elementos de Azure en Windows Server.
ms.topic: tutorial
ms.date: 02/14/2018
ms.custom: mvc
ms.openlocfilehash: 746c901747cf1c0b87612a31fbabcb657d5c4a0c
ms.sourcegitcommit: 64ad2c8effa70506591b88abaa8836d64621e166
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88263119"
---
# <a name="recover-files-from-azure-to-a-windows-server"></a>Recuperación de archivos de Azure en Windows Server

Azure Backup permite la recuperación de elementos individuales de las copias de seguridad de Windows Server. La recuperación de archivos individuales es útil si necesita restaurar rápidamente archivos que se eliminaron accidentalmente. En este tutorial se explica cómo puede usar al agente Microsoft Azure Recovery Services (MARS) para recuperar elementos de copias de seguridad realizadas previamente en Azure. En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Iniciar la recuperación de elementos individuales
> * Seleccionar un punto de recuperación
> * Restaurar elementos desde un punto de recuperación

En este tutorial se da por supuesto que ya ha realizado los pasos necesarios para [realizar la copia de seguridad de Windows Server en Azure](backup-windows-with-mars-agent.md) y que tiene al menos una copia de seguridad de los archivos de Windows Server en Azure.

## <a name="initiate-recovery-of-individual-items"></a>Iniciar la recuperación de elementos individuales

Un práctico asistente de interfaz de usuario denominado Microsoft Azure Backup se instala con el agente de Microsoft Azure Recovery Services (MARS). El asistente Microsoft Azure Backup funciona con el agente de Microsoft Azure Recovery Services (MARS) para recuperar datos de copia de seguridad desde los puntos de recuperación almacenados en Azure. Use el asistente Microsoft Azure Backup para identificar los archivos o las carpetas que quiere restaurar en Windows Server.

1. Abra el complemento **Microsoft Azure Backup**. Para encontrarlo, busque **Microsoft Azure Backup**en la máquina.

    ![Complemento Microsoft Azure Backup](./media/tutorial-backup-restore-files-windows-server/mars.png)

2. En el asistente, seleccione **Recuperar datos** en el **panel Acciones** de la consola del agente para iniciar el **Asistente para recuperación de datos**.

    ![Selección de la recuperación de datos](./media/tutorial-backup-restore-files-windows-server/mars-recover-data.png)

3. En la página **Introducción**, seleccione **Este servidor (nombre del servidor)** y seleccione **Siguiente**.

4. En la página **Seleccionar modo de recuperación**, seleccione **Archivos y carpetas individuales** y, a continuación, seleccione **Siguiente** para iniciar el proceso de selección del punto de recuperación.

5. En la página **Seleccionar volumen y fecha**, seleccione el volumen que contiene los archivos o las carpetas que desee restaurar y seleccione **Montar**. Seleccione una fecha y, luego, elija una hora en el menú desplegable correspondiente a un punto de recuperación. Las fechas en **negrita** indican la disponibilidad de al menos un punto de recuperación ese día.

    ![Selección de volumen y fecha](./media/tutorial-backup-restore-files-windows-server/mars-select-date.png)

    Al seleccionar **Montar**, Azure Backup hace que el punto de recuperación esté disponible como un disco. Busque y recupere los archivos del disco.

## <a name="restore-items-from-a-recovery-point"></a>Restaurar elementos desde un punto de recuperación

1. Una vez montado el volumen de recuperación, seleccione **Examinar** para abrir el Explorador de Windows y busque los archivos y carpetas que desee.

    ![Seleccione Examinar](./media/tutorial-backup-restore-files-windows-server/mars-browse-recover.png)

    Puede abrir los archivos directamente desde el volumen de recuperación, así como comprobarlos.

2. En el Explorador de Windows, copie los archivos y carpetas que desee restaurar y péguelos en la ubicación que desee en el servidor.

    ![Copie los archivos y las carpetas](./media/tutorial-backup-restore-files-windows-server/mars-final.png)

3. Cuando termine de restaurar los archivos y carpetas, en la página **Examinar y recuperar archivos** del **Asistente para recuperación de datos**, seleccione **Desmontar**.

    ![Seleccionar el desmontaje](./media/tutorial-backup-restore-files-windows-server/unmount-and-confirm.png)

4. Seleccione **Sí** para confirmar que desee desmontar el volumen.

    Una vez desmontada la instantánea, aparece **Trabajo completado** en el panel **Trabajos** de la consola del agente.

## <a name="next-steps"></a>Pasos siguientes

Con esto finalizan los tutoriales sobre la copia de seguridad y la restauración de datos de Windows Server en Azure. Para obtener más información acerca de Azure Backup, consulte el ejemplo de PowerShell sobre la copia de seguridad de máquinas virtuales cifradas.

> [!div class="nextstepaction"]
> [Copia de seguridad de máquinas virtuales cifradas](./scripts/backup-powershell-sample-backup-encrypted-vm.md)
