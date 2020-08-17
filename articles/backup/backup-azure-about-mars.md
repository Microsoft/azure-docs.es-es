---
title: Acerca del agente de MARS
description: Obtenga información sobre cómo el agente de MARS admite los escenarios de copia de seguridad
ms.topic: conceptual
ms.date: 08/04/2020
ms.openlocfilehash: 8e4ace0c17dbe75e989981db56583ed9477b3716
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87562606"
---
# <a name="about-the-microsoft-azure-recovery-services-mars-agent"></a>Acerca del agente de Microsoft Azure Recovery Services (MARS)

En este artículo se describe de qué manera el servicio Azure Backup usa el agente de Microsoft Azure Recovery Services (MARS) para crear copias de seguridad y restaurar archivos y carpetas, así como el volumen o el estado del sistema desde un equipo local en Azure.

## <a name="backup-scenarios"></a>Escenarios de copia de seguridad

El agente de MARS admite los escenarios de copia de seguridad siguientes:

![Escenarios de copia de seguridad de MARS](./media/backup-try-azure-backup-in-10-mins/backup-scenarios.png)

- **Archivos y carpetas**: proteja selectivamente archivos y carpetas de Windows.
- **Nivel de volumen**: proteja todo el volumen de Windows de la máquina.
- **Nivel del sistema**: proteja todo el estado del sistema Windows.

### <a name="additional-scenarios"></a>Otros escenarios

- **Hacer una copia de seguridad de archivos y carpetas específicos dentro de la VM de Azure**: el método principal para hacer copias de seguridad de máquinas virtuales (VM) de Azure es mediante una extensión de Azure Backup en la VM. La extensión realiza una copia de seguridad de toda la máquina virtual. Si quiere hacer una copia de seguridad de archivos y carpetas específicos dentro de una VM, puede instalar el agente de MARS en las máquinas virtuales de Azure. Para obtener más información, consulte [Arquitectura: copia de seguridad integrada de máquina virtual de Azure](./backup-architecture.md#architecture-built-in-azure-vm-backup).

- **Propagación sin conexión**: las copias de seguridad completas iniciales de los datos en Azure normalmente transfieren grandes cantidades de datos y requieren más ancho de banda de red. Las copias de seguridad posteriores solo transfieren los datos Delta, o incrementales. Azure Backup permite comprimir las copias de seguridad iniciales. A través del proceso de *propagación sin conexión*, Azure Backup puede usar discos para cargar los datos comprimidos iniciales de la copia de seguridad sin conexión en Azure. Para obtener más información, consulte [Copia de seguridad sin conexión de Azure Backup con Azure Data Box](offline-backup-azure-data-box.md).

## <a name="restore-scenarios"></a>Escenarios de restauración

El agente de MARS admite los siguientes escenarios de restauración:

![Escenarios de recuperación de MARS](./media/backup-try-azure-backup-in-10-mins/restore-scenarios.png)

- **Mismo servidor**: el servidor en el que se creó originalmente la copia de seguridad.
  - **Archivos y carpetas**: elija los archivos y carpetas individuales que quiere restaurar.
  - **Nivel de volumen**: elija el volumen y el punto de recuperación que quiere restaurar. A continuación, restáurelo en la misma ubicación o en una ubicación alternativa en el mismo equipo.  Cree una copia de los archivos existentes, sobrescriba los archivos existentes u omita la recuperación de los archivos existentes.
  - **Nivel del sistema**: elija el estado del sistema y el punto de recuperación que quiere restaurar en una ubicación especificada de la misma máquina.

- **Servidor alternativo**: un servidor distinto del que se usó para realizar la copia de seguridad.
  - **Archivos y carpetas**: elija los archivos y carpetas individuales cuyo punto de recuperación quiere restaurar en un equipo de destino.
  - **Nivel de volumen**: elija el volumen y el punto de recuperación que quiere restaurar en otra ubicación. Cree una copia de los archivos existentes, sobrescriba los archivos existentes u omita la recuperación de los archivos existentes.
  - **Nivel del sistema**: elija el estado del sistema y el punto de recuperación que quiere restaurar como archivo de estado del sistema en una máquina alternativa.

## <a name="backup-process"></a>Proceso de copia de seguridad

1. En Azure Portal, cree un [almacén de Recovery Services](install-mars-agent.md#create-a-recovery-services-vault) y elija los archivos, las carpetas y el estado del sistema de entre los **objetivos de copia de seguridad**.
2. [Descargue las credenciales del almacén de Recovery Services y el instalador del agente](./install-mars-agent.md#download-the-mars-agent) en una máquina local.

3. [Instale el agente](./install-mars-agent.md#install-and-register-the-agent) y use las credenciales del almacén que descargó para registrar la máquina en el almacén de Recovery Services.
4. Desde la consola del agente en el cliente, [configure la copia de seguridad](./backup-windows-with-mars-agent.md#create-a-backup-policy) para especificar de qué se hará una copia de seguridad, cuándo se hará (programación), cuánto tiempo se conservará en Azure (directiva de retención) e inicie la protección.

![Diagrama del agente de Azure Backup](./media/backup-try-azure-backup-in-10-mins/backup-process.png)

### <a name="additional-information"></a>Información adicional

- La **copia de seguridad inicial** (primera copia de seguridad) se ejecuta de acuerdo con la configuración de la copia de seguridad.  El agente de MARS usa VSS para tomar una instantánea de un momento dado de los volúmenes seleccionados para la copia de seguridad. El agente solo usa la operación de escritor del sistema de Windows para capturar la instantánea. No usa instancias de VSS Writer de aplicaciones y no captura instantáneas coherentes con la aplicación. Después de tomar la instantánea con VSS, el agente de MARS crea un disco duro virtual en la carpeta de caché que especificó al configurar la copia de seguridad. El agente también almacena las sumas de comprobación para cada bloque de datos.

- Se ejecutan **copias de seguridad incrementales (subsiguientes)** según la programación que especifique. Durante las copias de seguridad incrementales, se identifican los archivos modificados y se crea un disco duro virtual. Este disco se comprime y cifra, y se envía al almacén. Una vez finalizada la copia de seguridad incremental, el nuevo disco duro virtual se combina con el disco duro virtual creado después de la replicación inicial. Este disco duro virtual combinado proporciona el estado más reciente que se usará para la comparación de la copia de seguridad en curso.

- El agente de MARS puede ejecutar el trabajo de copia de seguridad en **modo optimizado** con el diario de cambios USN (número de secuencia actualizada) o **modo no optimizado** al comprobar los cambios en los directorios o archivos examinando todo el volumen. El modo no optimizado es más lento porque el agente tiene que examinar cada uno de los archivos del volumen y compararlo con los metadatos para establecer qué archivos se han modificado.  La **copia de seguridad inicial** siempre se ejecutará en modo no optimizado. Si se produce un error en el trabajo de copia de seguridad anterior, el siguiente trabajo de copia de seguridad programado se ejecutará en modo no optimizado. Para obtener más información sobre estos modos y cómo verificarlos, consulte [este artículo](backup-azure-troubleshoot-slow-backup-performance-issue.md#cause-backup-job-running-in-unoptimized-mode).

## <a name="next-steps"></a>Pasos siguientes

[Matriz de compatibilidad del agente de MARS](./backup-support-matrix-mars-agent.md)

[Preguntas más frecuentes del agente de MARS](./backup-azure-file-folder-backup-faq.md)
