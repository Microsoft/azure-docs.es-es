---
title: Introducción a la arquitectura
description: Proporciona información general sobre la arquitectura, los componentes y los procesos usados por el servicio Azure Backup.
ms.topic: conceptual
ms.date: 02/19/2019
ms.openlocfilehash: 1081de6b467b896bd8cc62b84c9a67c329b11e02
ms.sourcegitcommit: ac7ae29773faaa6b1f7836868565517cd48561b2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88824039"
---
# <a name="azure-backup-architecture-and-components"></a>Arquitectura y componentes de Azure Backup

Puede usar el [servicio Azure Backup](backup-overview.md) para realizar una copia de seguridad de los datos en la plataforma de nube de Microsoft Azure. En este artículo se resume la arquitectura, los componentes y los procesos de Azure Backup.

## <a name="what-does-azure-backup-do"></a>¿Qué hace Azure Backup?

Azure Backup realiza copias de seguridad de los datos, del estado de la máquina y de las cargas de trabajo que se ejecutan en instancias de máquinas locales y máquinas virtuales de Azure. Son varios los escenarios de uso de Azure Backup.

## <a name="how-does-azure-backup-work"></a>¿Cómo funciona Azure Backup?

Puede realizar copias de seguridad de máquinas y datos mediante una serie de métodos:

- **Copia de seguridad de máquinas locales**:
  - Puede hacer una copia de seguridad de máquinas Windows locales directamente en Azure mediante el agente Microsoft Azure Recovery Services (MARS) de Azure Backup. No se admiten máquinas Linux.
  - Puede realizar copias de seguridad de máquinas locales en un servidor de copia de seguridad (System Center Data Protection Manager [DPM] o Microsoft Azure Backup Server [MABS]). Luego, puede hacer la copia de seguridad del servidor de copia de seguridad en un almacén de Recovery Services en Azure.

- **Copia de seguridad de máquinas virtuales de Azure**:
  - Puede hacer una copia de seguridad de las máquinas virtuales de Azure directamente. Azure Backup instala una extensión de copia de seguridad en el agente de máquina virtual de Azure que se ejecuta en la máquina virtual. Esta extensión realiza una copia de seguridad de toda la máquina virtual.
  - Puede hacer una copia de seguridad de determinados archivos y carpetas en la máquina virtual de Azure mediante la ejecución del agente de MARS.
  - Puede hacer una copia de seguridad de las máquinas virtuales de Azure en un servidor MABS que se ejecuta en Azure y, a continuación, puede hacer una copia de seguridad de MABS en un almacén de Recovery Services.

Obtenga información sobre los [elementos de lo que puede hacer una copia de seguridad](backup-overview.md) y los [escenarios de copia de seguridad admitidos](backup-support-matrix.md).

## <a name="where-is-data-backed-up"></a>¿Dónde están los datos de copia de seguridad?

Azure Backup almacena los datos de copia de seguridad en un almacén de Recovery Services. Un almacén es una entidad de almacenamiento en línea de Azure que se usa para contener datos, como copias de seguridad, puntos de recuperación y directivas de copia de seguridad.

Los almacenes de Recovery Services tienen las siguientes características:

- Los almacenes facilitan la tarea de organizar los datos de copia de seguridad, al mismo tiempo que reducen al mínimo su sobrecarga administrativa.
- En cada suscripción de Azure, puede crear hasta 500 almacenes.
- Puede supervisar elementos de copia de seguridad de un almacén, como las máquinas virtuales de Azure y las máquinas locales.
- Puede administrar el acceso al almacén con el [control de acceso basado en rol de Azure (RBAC de Azure)](../role-based-access-control/role-assignments-portal.md).
- Especificará cómo se replican los datos en el almacén para la redundancia:
  - **Almacenamiento con redundancia local (LRS)** : Para protegerse frente a los errores de un centro de datos, puede usar LRS. LRS replica los datos en una unidad de escalado de almacenamiento. [Más información](../storage/common/storage-redundancy.md).
  - **Almacenamiento con redundancia geográfica (GRS)** : Para protegerse contra las interrupciones de toda la región, puede usar GRS. GRS replica los datos en una región secundaria. [Más información](../storage/common/storage-redundancy.md).
  - De forma predeterminada, los almacenes de Recovery Services usan GRS.

## <a name="backup-agents"></a>Agentes de copia de seguridad

Azure Backup proporciona diferentes agentes de copia de seguridad, según el tipo de máquina de la que se esté haciendo una copia de seguridad:

**Agent** | **Detalles**
--- | ---
**Agente de MARS** | <ul><li>Se ejecuta en servidores Windows locales individuales para hacer una copia de seguridad de archivos, carpetas y del estado del sistema.</li> <li>Se ejecuta en máquinas virtuales de Azure para hacer una copia de seguridad de archivos, carpetas y del estado del sistema.</li> <li>Se ejecuta en servidores DPM o MABS para hacer una copia de seguridad del disco de almacenamiento local de DPM o MABS en Azure.</li></ul>
**Extensión de máquina virtual de Azure** | Se ejecuta en máquinas virtuales de Azure para copiarlas en un almacén.

## <a name="backup-types"></a>Tipos de copia de seguridad

La siguiente tabla explica los distintos tipos de copias de seguridad y cuando se usan:

**Tipo de copia de seguridad** | **Detalles** | **Uso**
--- | --- | ---
**Completa** | Una copia de seguridad completa contiene todo el origen de los datos. Requiere más ancho de banda de red que las copias de seguridad incrementales o diferenciales. | Se usa para la copia de seguridad inicial.
**Diferencial** |  Una copia de seguridad diferencial almacena los bloques que cambiaron desde la copia de seguridad completa inicial. Usa una menor cantidad de almacenamiento y red y no conserva copias redundantes de datos sin modificar.<br/><br/> Ineficaz porque los bloques de datos sin modificar entre las copias de seguridad posteriores se transfieren y almacenan. | Azure Backup no usa este tipo.
**Incremental** | Una copia de seguridad incremental almacena solo los bloques de datos que cambiaron desde la copia de seguridad anterior. Alta eficacia de almacenamiento y red. <br/><br/> Con una copia de seguridad incremental, no es necesario complementar con copias de seguridad completas. | Se usa en DPM o MABS para copias de seguridad de disco y se emplea en todas las copias de seguridad en Azure. No se utiliza para la copia de seguridad de SQL Server.

## <a name="sql-server-backup-types"></a>Tipos de copia de seguridad de SQL Server

La siguiente tabla explica los distintos tipos de copias de seguridad que se utilizan para las bases de datos de SQL Server y con qué frecuencia se usan:

**Tipo de copia de seguridad** | **Detalles** | **Uso**
--- | --- | ---
**Copia de seguridad completa** | una copia de seguridad completa de la base de datos realiza una copia de seguridad de toda la base de datos. Contiene todos los datos de una base de datos específica o de un conjunto de archivos o grupos de archivos. Una copia de seguridad completa también contiene suficientes registros para recuperar esos datos. | A lo sumo, puede desencadenar una copia de seguridad completa al día.<br/><br/> Puede elegir realizar una copia de seguridad completa en un intervalo diario o semanal.
**Copia de seguridad diferencial**: | la copia de seguridad diferencial se basa en la copia de seguridad de datos completa anterior más reciente.<br/><br/> Captura solo los datos que han cambiado desde la copia de seguridad completa. |  A lo sumo, puede desencadenar una copia de seguridad diferencial al día.<br/><br/> No se puede configurar una copia de seguridad completa y una copia de seguridad diferencial en el mismo día.
**Copia de seguridad del registro de transacciones**: | una copia de seguridad de registros permite realizar la restauración a un momento dado con una precisión de un segundo. | A lo sumo, puede configurar las copias de seguridad del registro de transacciones cada 15 minutos.

### <a name="comparison-of-backup-types"></a>Comparación de tipos de copia de seguridad

El consumo de almacenamiento, el objetivo de tiempo de recuperación (RTO) y el consumo de red varían según el tipo de copia de seguridad. En la siguiente imagen se muestra una comparación de los tipos de copia de seguridad:

- El origen de datos A se compone de 10 bloques de almacenamiento A1-A10, y todos los meses se hace una copia de seguridad de ellos.
- Los bloques A2, A3, A4 y A9 cambian en el primer mes y el bloque A5 cambia en el siguiente mes.
- Para las copias de seguridad diferenciales, en el segundo mes, se realiza una copia de seguridad de los bloques modificados A2, A3, A4 y A9. En el tercer mes, se vuelve a hacer una copia de seguridad de estos mismos bloques, junto con el bloque A5 modificado. Los bloques modificados se siguen copiando hasta que tiene lugar la siguiente copia de seguridad completa.
- Para las copias de seguridad incrementales, en el segundo mes, los bloques A2, A3, A4 y A9 se marcan como modificados y se transfieren. En el tercer mes, solo el bloque A5 modificado se marca y se transfiere.

![Imagen que muestra las comparaciones de métodos de copia de seguridad](./media/backup-architecture/backup-method-comparison.png)

## <a name="backup-features"></a>Características de copia de seguridad

En la tabla siguiente se resumen las características compatibles de los distintos tipos de copia de seguridad:

**Característica** | **Copia de seguridad directa de archivos y carpetas (mediante el agente de MARS)** | **Copia de seguridad de máquina virtual de Azure** | **Máquinas o aplicaciones con DPM o MABS**
--- | --- | --- | ---
Copia de seguridad en almacén | ![Sí][green] | ![Sí][green] | ![Sí][green]
Copia de seguridad en disco DPM o MABS y, luego, en Azure | | | ![Sí][green]
Compresión de los datos enviados para copia de seguridad | ![Sí][green] | No se usa compresión al transferir los datos. El almacenamiento aumenta ligeramente, pero la restauración es más rápida.  | ![Sí][green]
Ejecución de copia de seguridad incremental |![Sí][green] |![Sí][green] |![Sí][green]
Copia de seguridad de discos desduplicados | | | ![Parcialmente][yellow]<br/><br/> Solo para servidores DPM o MABS implementados en el entorno local.

![Clave de tabla](./media/backup-architecture/table-key.png)

## <a name="backup-policy-essentials"></a>Fundamentos de la directiva de copia de seguridad

- Se crea una directiva de copia de seguridad por almacén.
- Se puede crear una directiva de copia de seguridad para la copia de seguridad de las cargas de trabajo siguientes: Máquinas virtuales de Azure, SQL en máquinas virtuales de Azure, SAP HANA en máquinas virtuales de Azure y recursos compartidos de archivos de Azure. La directiva de copia de seguridad de archivos y carpetas que usa el agente de MARS se especifica en la consola de MARS.
  - Recurso compartido de archivos de Azure
- Una directiva puede asignarse a muchos recursos. Una directiva de copia de seguridad de Azure VM puede usarse para proteger varias máquinas virtuales de Azure.
- Una directiva consta de dos componentes:
  - Programación: cuándo realizar la copia de seguridad
  - Retención: cuánto tiempo debe retenerse cada copia de seguridad.
- La programación puede definirse como "diaria" o "semanal" con un punto específico en el tiempo.
- La retención puede definirse para los puntos de copia de seguridad "diarios", "semanal", "mensual" o "anual" .
  - "semanal" hace referencia a una copia de seguridad en un determinado día de la semana.
  - "mensual" hace referencia a una copia de seguridad en un determinado día del mes.
  - "anual" hace referencia a una copia de seguridad en un determinado día del año.
- La retención de los puntos de copia de seguridad anuales y mensuales se conoce como "retención a largo plazo" (LTR).
- Cuando se crea un almacén, también se crea una directiva "DefaultPolicy" que se puede usar para crear copias de seguridad de los recursos.
- Cualquier cambio en el período de retención de una directiva de copia de seguridad se aplicará con efectos retroactivos a todos los puntos de recuperación anteriores, además de a los nuevos.

### <a name="additional-reference"></a>Referencia adicional

- Máquina virtual de Azure: ¿Cómo [crear](./backup-azure-vms-first-look-arm.md#back-up-from-azure-vm-settings) y [modificar](./backup-azure-manage-vms.md#manage-backup-policy-for-a-vm) una directiva?
- Base de datos de SQL Server en una máquina virtual de Azure: ¿Cómo [crear](./backup-sql-server-database-azure-vms.md#create-a-backup-policy) y [modificar](./manage-monitor-sql-database-backup.md#modify-policy) una directiva?
- Recurso compartido de archivos de Azure: ¿Cómo [crear](./backup-afs.md) y [modificar](./manage-afs-backup.md#modify-policy) una directiva?
- SAP HANA: ¿Cómo [crear](./backup-azure-sap-hana-database.md#create-a-backup-policy) y [modificar](./sap-hana-db-manage.md#change-policy) una directiva?
- MARS: ¿Cómo [crear](./backup-windows-with-mars-agent.md#create-a-backup-policy) y [modificar](./backup-azure-manage-mars.md#modify-a-backup-policy) una directiva?
- [¿Existen límites sobre la programación de copia de seguridad en función del tipo de carga de trabajo?](./backup-azure-backup-faq.md#are-there-limits-on-backup-scheduling)
- [¿Qué ocurre con los puntos de recuperación existentes cuando se cambia mi directiva de recuperación?](./backup-azure-backup-faq.md#what-happens-when-i-change-my-backup-policy)

## <a name="architecture-built-in-azure-vm-backup"></a>Arquitectura: Copia de seguridad integrada de máquina virtual de Azure

1. Cuando se habilita la copia de seguridad de una máquina virtual de Azure, se ejecuta una copia de seguridad con la programación que especifique.
1. Durante la primera copia de seguridad, se instala una extensión de copia de seguridad en la VM si esta se encuentra en ejecución.
    - En máquinas virtuales Windows, se instala la extensión VMSnapshot.
    - En máquinas virtuales Linux, se instala la extensión VMSnapshot para Linux.
1. La extensión toma una instantánea de nivel de almacenamiento.
    - En el caso de las máquinas virtuales Windows en ejecución, el servicio Backup se coordina con el Servicio de instantáneas de volumen (VSS) de Windows para obtener una instantánea coherente con la aplicación de la máquina virtual. De forma predeterminada, Azure Backup realiza copias de seguridad de VSS completas. Si Azure Backup no puede tomar una instantánea coherente con la aplicación, toma una instantánea coherente con el archivo.
    - En máquinas virtuales Linux, Azure Backup toma una instantánea coherente con el archivo. Para obtener instantáneas coherentes con la aplicación, debe personalizar manualmente los scripts previos y posteriores.
    - Azure Backup se optimiza mediante la copia de seguridad de cada disco de máquina virtual en paralelo. Este servicio lee los bloques de cada disco que se va a copiar y solo almacena los datos cambiados.
1. Después de tomar la instantánea, los datos se transfieren al almacén.
    - Solo se copian los bloques de datos que han cambiado desde la última copia de seguridad.
    - Los datos no se cifran. Azure Backup puede hacer una copia de seguridad de las máquinas virtuales de Azure que se cifraron mediante Azure Disk Encryption.
    - Es posible que los datos de las instantáneas no se copien inmediatamente en el almacén. En momentos de máxima actividad, la copia de seguridad podría durar horas. El tiempo total de copia de seguridad de una VM será inferior a 24 horas para las directivas de copia de seguridad diarias.
1. Una vez enviados los datos al almacén, se crea un punto de recuperación. De manera predeterminada, las instantáneas se conservan durante dos días antes de ser eliminadas. Esta característica permite la operación de restauración a partir de estas instantáneas, lo cual reduce los tiempos de restauración. Reduce el tiempo necesario para transformar y copiar datos desde el almacén. Consulte [Funcionalidad de restauración instantánea de Azure Backup](./backup-instant-restore-capability.md).

No es necesario permitir explícitamente la conectividad a Internet para realizar copias de seguridad de las VM de Azure.

![Copia de seguridad de máquinas virtuales de Azure](./media/backup-architecture/architecture-azure-vm.png)

## <a name="architecture-direct-backup-of-on-premises-windows-server-machines-or-azure-vm-files-or-folders"></a>Arquitectura: copia de seguridad directa de máquinas Windows Server locales o de archivos o carpetas de máquinaz virtuales de Azure

1. Para configurar el escenario, descargue e instale el agente de MARS en la máquina. A continuación, seleccione los elementos objeto de la copia de seguridad, cuándo se ejecutarán las copias de seguridad y cuánto tiempo se deberán mantener en Azure.
1. La copia de seguridad inicial se ejecuta según la configuración de copia de seguridad.
1. El agente de MARS usa VSS para tomar una instantánea de un momento dado de los volúmenes seleccionados para la copia de seguridad.
    - El agente de MARS solo usa la operación de escritura del sistema Windows para capturar la instantánea.
    - Dado que el agente no usa instancias de VSS Writer de aplicaciones, no captura instantáneas coherentes con la aplicación.
1. Después de tomar la instantánea con VSS, el agente de MARS crea un disco duro virtual en la carpeta de caché que especificó al configurar la copia de seguridad. El agente también almacena las sumas de comprobación para cada bloque de datos.
1. Se ejecutan copias de seguridad incrementales según la programación que especifique, a menos que ejecute una copia de seguridad a petición.
1. En las copias de seguridad incrementales, se identifican los archivos modificados y se crea un disco duro virtual. Este disco se comprime y cifra, y se envía al almacén.
1. Una vez finalizada la copia de seguridad incremental, el nuevo disco duro virtual se combina con el disco duro virtual creado después de la replicación inicial. Este disco duro virtual combinado proporciona el estado más reciente que se usará para la comparación de la copia de seguridad en curso.

![Copia de seguridad de máquinas locales de Windows Server con el agente de MARS](./media/backup-architecture/architecture-on-premises-mars.png)

## <a name="architecture-back-up-to-dpmmabs"></a>Arquitectura: copia de seguridad en DPM o MABS

1. El agente de protección de DPM o MABS se instala en las máquinas que quiere proteger. A continuación, agrega las máquinas a un grupo de protección de DPM.
    - Para proteger las máquinas locales, el servidor DPM o MABS debe ubicarse en el entorno local.
    - Para proteger las máquinas virtuales de Azure, el servidor MABS debe ubicarse en Azure, de modo que se ejecuta como una máquina virtual de Azure.
    - Con DPM o MABS puede proteger volúmenes de copia de seguridad, recursos compartidos, archivos y carpetas. También puede proteger el estado del sistema de las máquinas (reconstrucción completa) y proteger aplicaciones específicas con la configuración de la copia de seguridad con reconocimiento de aplicaciones.
1. Al configurar la protección para una máquina o una aplicación en DPM o MABS, selecciona hacer la copia de seguridad en el disco local de DPM o MABS para el almacenamiento a corto plazo, y en Azure para la protección en línea. También especificará cuándo se debe ejecutar la copia de seguridad en el almacenamiento local de DPM o MABS y cuándo la copia de seguridad en línea en Azure.
1. El disco de la carga de trabajo protegida se copia en los discos locales de DPM o MABS, según la programación especificada.
1. Los discos de DPM o MABS se copian en el almacén por el agente de MARS que se ejecuta en el servidor DPM o MABS.

![Copia de seguridad de máquinas y cargas de trabajo protegidas mediante DPM o MABS](./media/backup-architecture/architecture-dpm-mabs.png)

## <a name="azure-vm-storage"></a>Almacenamiento de máquinas virtuales de Azure

Las máquinas virtuales de Azure usan discos para almacenar su sistema operativo, sus aplicaciones y sus datos. Cada máquina virtual de Azure tiene al menos dos discos: un disco de sistema operativo y un disco temporal. Las máquinas virtuales de Azure pueden tener discos de datos para datos de aplicación. Los discos se almacenan como discos duros virtuales.

- Los discos duros virtuales se almacenan como blobs en páginas en cuentas de almacenamiento estándar o premium en Azure:
  - **Almacenamiento estándar**: compatibilidad confiable y de bajo costo con máquinas virtuales que ejecutan cargas de trabajo que no son sensibles a la latencia. El almacenamiento estándar puede usar discos de unidades de estado sólido (SSD) estándar o unidades de disco duro (HDD) estándar.
  - **Almacenamiento premium:** compatibilidad con discos de alto rendimiento. Usa discos SSD premium.
- Existen distintos niveles de rendimiento para los discos:
  - **Disco HDD estándar:** están respaldado por HDD y se usan cuando se busca un almacenamiento rentable.
  - **Disco SSD estándar:** Combina elementos de discos SSD premium y HDD estándar. Ofrece rendimiento y fiabilidad más coherentes que los discos HDD, y siguen siendo rentables.
  - **Disco SSD premium:** respaldado por SSD, y proporciona alto rendimiento y baja latencia para máquinas virtuales que ejecutan cargas de trabajo que consumen elevada E/S.
- Los discos pueden ser administrados o no administrados:
  - **Discos no administrados:** tipo tradicional de discos usado por las máquinas virtuales. Con estos discos, creará la propia cuenta de almacenamiento y especificará esa cuenta al crear el disco. Tiene que averiguar cómo maximizar los recursos de almacenamiento de las máquinas virtuales.
  - **Discos administrados:** Azure crea y administra las cuentas de almacenamiento en su nombre. Usted especifica el tamaño de disco y el nivel de rendimiento y Azure crea discos administrados en su nombre. A medida que agrega discos y escala máquinas virtuales, Azure se ocupa de administrar las cuentas de almacenamiento.

Para obtener más información sobre el almacenamiento en discos y los tipos de discos disponibles para las máquinas virtuales, consulte estos artículos:

- [Introducción a los discos administrados de Azure](../virtual-machines/managed-disks-overview.md)
- [¿Qué tipos de disco están disponibles en Azure?](../virtual-machines/disks-types.md)

### <a name="back-up-and-restore-azure-vms-with-premium-storage"></a>Copia de seguridad y restauración de máquinas virtuales de Azure con Premium Storage

Puede realizar una copia de seguridad de máquinas virtuales de Azure mediante Premium Storage con Azure Backup:

- Durante el proceso de copia de seguridad de máquinas virtuales con Premium Storage, el servicio Backup crea una ubicación de almacenamiento provisional, llamada *AzureBackup-* en la cuenta de almacenamiento. El tamaño de la ubicación de ensayo equivale al de la instantánea del punto de recuperación.
- Asegúrese de que haya espacio disponible suficiente en la cuenta de Premium Storage para dar cabida a la ubicación de almacenamiento provisional. Para obtener más información, consulte [Objetivos de escalabilidad de las cuentas de almacenamiento de blob en páginas prémium](../storage/blobs/scalability-targets-premium-page-blobs.md). No modifique la ubicación de almacenamiento provisional.
- Una vez finalizado el trabajo de copia de seguridad, se elimina esta ubicación.
- El precio del almacenamiento usado para la ubicación de almacenamiento provisional es coherente con todos los [precios de Premium Storage](../virtual-machines/disks-types.md#billing).

Al restaurar máquinas virtuales de Azure mediante Premium Storage, puede restaurarlas a Premium Storage o Standard Storage. Normalmente, las restaurará a Premium Storage. Pero si necesita solo un subconjunto de archivos de la máquina virtual, podría ser más rentable restaurarlas a Standard Storage.

### <a name="back-up-and-restore-managed-disks"></a>Copia de seguridad y restauración de discos administrados

Puede realizar copias de seguridad de máquinas virtuales de Azure con discos administrados:

- La copia de seguridad de máquinas virtuales con discos administrados se hace de la misma manera que con cualquier otra máquina virtual de Azure. Se puede hacer una copia de seguridad de la máquina virtual directamente desde la configuración de dicha máquina, o se puede habilitar la copia de seguridad para las máquinas virtuales en el almacén de Recovery Services.
- Puede realizar la copia de seguridad de máquinas virtuales en discos administrados mediante colecciones de RestorePoint basadas en discos administrados.
- Azure Backup también admite la realización de copias de seguridad de máquinas virtuales con discos administrados cifradas mediante Azure Disk Encryption.

Al restaurar máquinas virtuales con discos administrados, puede restaurarlas en una máquina virtual completa con discos administrados, o en una cuenta de almacenamiento:

- Azure gestiona los discos administrados durante el proceso de restauración. Con la opción de cuenta de almacenamiento, usted gestiona la cuenta de almacenamiento que se crea durante la restauración.
- Si restaura una máquina virtual administrada que está cifrada, asegúrese de que los secretos y claves de máquina virtual existen en el almacén de claves antes de iniciar el proceso de restauración.

## <a name="next-steps"></a>Pasos siguientes

- Revise la matriz de compatibilidad para [conocer las características admitidas y las limitaciones en escenarios de copia de seguridad](backup-support-matrix.md).
- Configure la copia de seguridad para uno de los siguientes escenarios:
  - [Copia de seguridad de máquinas virtuales de Azure](backup-azure-arm-vms-prepare.md).
  - [Copia de seguridad directa de máquinas Windows](tutorial-backup-windows-server-to-azure.md), sin un servidor de copia de seguridad.
  - [Configuración de MABS](backup-azure-microsoft-azure-backup.md) para la copia de seguridad en Azure y, luego, copia de seguridad de las cargas de trabajo en MABS.
  - [Configuración de DPM](backup-azure-dpm-introduction.md) para la copia de seguridad en Azure y, luego, copia de seguridad de las cargas de trabajo en DPM.

[green]: ./media/backup-architecture/green.png
[yellow]: ./media/backup-architecture/yellow.png
[red]: ./media/backup-architecture/red.png
