---
title: Matriz de compatibilidad para copias de seguridad de máquinas virtuales de Azure
description: Proporciona un resumen de opciones de compatibilidad y limitaciones para realizar copias de seguridad de máquinas virtuales de Azure con el servicio Azure Backup.
ms.topic: conceptual
ms.date: 09/13/2019
ms.custom: references_regions
ms.openlocfilehash: c50a19acceeb68ead4d86a59bbe8275ae97ecb5f
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019596"
---
# <a name="support-matrix-for-azure-vm-backup"></a>Matriz de compatibilidad para copias de seguridad de máquinas virtuales de Azure

Puede usar el [servicio Azure Backup](backup-overview.md) para realizar copias de seguridad de cargas de trabajo y máquinas locales, y de las máquinas virtuales de Azure. En este artículo se resumen las configuraciones y limitaciones de compatibilidad para realizar copias de seguridad de máquinas virtuales de Azure con Azure Backup.

Otras matrices de compatibilidad:

- [Matriz de compatibilidad general](backup-support-matrix.md) para Azure Backup
- [Matriz de compatibilidad](backup-support-matrix-mabs-dpm.md) para copias de seguridad de Azure Backup Server o System Center Data Protection Manager (DPM)
- [Matriz de compatibilidad](backup-support-matrix-mars-agent.md) para la copia de seguridad con el agente de Microsoft Azure Recovery Services (MARS)

## <a name="supported-scenarios"></a>Escenarios admitidos

A continuación, se muestra cómo puede realizar copias de seguridad y restauraciones de máquinas virtuales de Azure con el servicio Azure Backup.

**Escenario** | **Backup** | **Agent** |**Restauración**
--- | --- | --- | ---
copia de seguridad directa de máquinas virtuales de Azure  | Copia de seguridad de toda la máquina virtual.  | No se necesita ningún agente adicional en la VM de Azure. Azure Backup instala y usa una extensión para el [agente de máquina virtual de Azure](../virtual-machines/extensions/agent-windows.md) que se ejecuta en la máquina virtual. | Realice la restauración como sigue:<br/><br/> - **Creación de una máquina virtual básica**. Esto es útil si la máquina virtual no tiene ninguna configuración especial, como varias direcciones IP.<br/><br/> - **Restauración del disco de máquina virtual**. Restaure el disco. Conéctelo a una máquina virtual existente o cree una máquina virtual a partir del disco mediante PowerShell.<br/><br/> - **Sustitución del disco de máquina virtual**. Si existe una máquina virtual y esta usa discos administrados (sin cifrar), puede restaurar un disco y usarlo para reemplazar un disco existente en la máquina virtual.<br/><br/> - **Restauración de archivos y carpetas específicos**. Puede restaurar archivos o carpetas de una máquina virtual en vez de toda la máquina virtual.
Copia de seguridad directa de máquinas virtuales de Azure (solo Windows)  | Copia de seguridad de archivos, carpetas o volúmenes específicos. | Instale el [agente de Azure Recovery Services](backup-azure-file-folder-backup-faq.md).<br/><br/> Puede ejecutar al agente de MARS junto con la extensión de copia de seguridad del agente de máquina virtual de Azure para realizar copias de seguridad de la máquina virtual en el nivel de archivo o carpeta. | Restauración de archivos y carpetas específicos.
Copia de seguridad de una máquina virtual de Azure en un servidor de copia de seguridad  | Copia de seguridad de archivos, carpetas y volúmenes; archivos de estado del sistema y de reconstrucción completa; datos de aplicaciones para System Center DPM o Microsoft Azure Backup Server (MABS).<br/><br/> Después, DPM/MABS realiza una copia de seguridad en el almacén de Backup. | Instale al agente de protección de DPM/MABS en la máquina virtual. El agente de MARS se instala en DPM/MABS.| Restauración de archivos, carpetas y volúmenes; archivos de estado del sistema y de reconstrucción completa; datos de aplicaciones.

Más información sobre la copia de seguridad [mediante un servidor de copias de seguridad](backup-architecture.md#architecture-back-up-to-dpmmabs) y sobre los [requisitos de compatibilidad](backup-support-matrix-mabs-dpm.md).

## <a name="supported-backup-actions"></a>Acciones de copia de seguridad admitidas

**Acción** | **Soporte técnico**
--- | ---
Copia de seguridad de una máquina virtual apagada o sin conexión | Compatible.<br/><br/> La instantánea es coherente solo con bloqueos, no con aplicaciones.
Copia de seguridad de discos después de migrar a discos administrados | Compatible.<br/><br/> La copia de seguridad seguirá funcionando. No se requiere ninguna acción.
Copia de seguridad de discos administrados después de habilitar el bloqueo del grupo de recursos | No compatible.<br/><br/> Azure Backup no puede eliminar los puntos de restauración anteriores y las copias de seguridad empezarán a generar errores cuando se alcance el límite máximo de puntos de restauración.
Modificar directiva de copia de seguridad de una máquina virtual | Compatible.<br/><br/> Se realizará la copia de seguridad de la máquina virtual con la programación y la configuración de retención de la nueva directiva. Si la configuración de retención está extendida, los puntos de recuperación existentes se marcan y se mantienen. Si se reducen, los puntos de recuperación existentes se eliminarán en el siguiente trabajo de limpieza y, después, se eliminarán por completo.
Cancelación de un trabajo de copia de seguridad| Compatible durante el proceso de instantáneas.<br/><br/> No se admite cuando se transfiere la instantánea al almacén.
Copia de seguridad de la máquina virtual en otra región o suscripción |No compatible.<br><br>Para realizar una copia de seguridad correcta, las máquinas virtuales deben estar en la misma suscripción que el almacén de copia de seguridad.
Copias de seguridad por día (mediante la extensión de máquina virtual de Azure) | Una copia de seguridad programada por día.<br/><br/>El servicio de Azure Backup admite hasta nueve copias de seguridad a petición al día, pero Microsoft recomienda no más de cuatro copias de seguridad diarias a petición para garantizar el máximo rendimiento.
Copias de seguridad por día (mediante el agente de MARS) | Tres copias de seguridad programadas por día.
Copias de seguridad por día (mediante DPM/MABS) | Dos copias de seguridad programadas por día.
Copia de seguridad mensual o anual| No se admite cuando la copia de seguridad se realiza con la extensión de máquina virtual de Azure. Solo se admiten copias de seguridad diarias y semanales.<br/><br/> Puede configurar la directiva para conservar las copias de seguridad diarias y semanales durante el período de retención mensual o anual.
Ajuste automático del reloj | No compatible.<br/><br/> Azure Backup no se ajusta automáticamente a los cambios al horario de verano cuando realiza la copia de seguridad de una máquina virtual.<br/><br/>  Modifique la directiva de forma manual según sea necesario.
[Características de seguridad para copias de seguridad híbridas](./backup-azure-security-feature.md) |No se pueden deshabilitar las características de seguridad.
Copia de seguridad de la máquina virtual cuya hora se ha cambiado | No compatible.<br/><br/> Si se cambia la hora de la máquina a una fecha y hora futuras después de habilitar la copia de seguridad para esa máquina virtual, aunque se revierta el cambio horario, no se garantiza que la copia de seguridad se efectúe correctamente.
Máquinas virtuales de Azure en [conjuntos de escalado de máquinas virtuales](../virtual-machine-scale-sets/overview.md) | La copia de seguridad y restauración se admiten para las máquinas virtuales con el [modo de orquestación](../virtual-machine-scale-sets/orchestration-modes.md#orchestration-modes) establecido en 3. <br><br>Los conjuntos de disponibilidad no se admiten.

## <a name="operating-system-support-windows"></a>Compatibilidad con sistema operativo (Windows)

En la tabla siguiente se resumen los sistemas operativos compatibles para realizar copias de seguridad de máquinas virtuales de Azure con Windows.

**Escenario** | **SO compatible**
--- | ---
Copia de seguridad con la extensión del agente de máquina virtual de Azure | - Windows 10 Client (solo 64 bits) <br/><br/>- Windows Server 2019 (Datacenter/Datacenter Core/Standard) <br/><br/> - Windows Server 2016 (Datacenter/Datacenter Core/Standard) <br/><br/> - Windows Server 2012 R2 (Datacenter/Standard) <br/><br/> - Windows Server 2012 (Datacenter/Standard) <br/><br/> - Windows Server 2008 R2 (RTM and SP1 Standard)  <br/><br/> - Windows Server 2008 (solo 64 bits)
Copia de seguridad con el agente de MARS | Sistemas operativos [compatibles](backup-support-matrix-mars-agent.md#supported-operating-systems).
Copia de seguridad con DPM/MABS | Sistemas operativos compatibles para copia de seguridad con [MABS](backup-mabs-protection-matrix.md) y [DPM](/system-center/dpm/dpm-protection-matrix).

Azure Backup no admite sistemas operativos de 32 bits.

## <a name="support-for-linux-backup"></a>Compatibilidad con copia de seguridad de Linux

Esto es lo que se admite si quiere hacer copias de seguridad de máquinas Linux.

**Acción** | **Soporte técnico**
--- | ---
Copia de seguridad de máquinas virtuales de Azure con Linux mediante el agente de máquina virtual de Azure para Linux | Copia de seguridad coherente con archivos.<br/><br/> Copia de seguridad coherente con la aplicación mediante [scripts personalizados](backup-azure-linux-app-consistent.md).<br/><br/> Durante la restauración, puede crear una máquina virtual, restaurar un disco y usarlo para crear una máquina virtual, o restaurar un disco y usarlo para reemplazar un disco en una máquina virtual existente. También puede restaurar archivos y carpetas individuales.
Copia de seguridad de máquinas virtuales de Azure con Linux mediante el agente de MARS | No compatible.<br/><br/> El agente MARS solo puede instalarse en máquinas Windows.
Copia de seguridad de máquinas virtuales de Azure con Linux mediante DPM/MABS | No compatible.

## <a name="operating-system-support-linux"></a>Compatibilidad con sistema operativo (Linux)

Para las copias de seguridad de máquinas virtuales de Azure con Linux, Azure Backup admite la [lista de distribuciones aprobadas por Azure](../virtual-machines/linux/endorsed-distros.md) para Linux. Tenga en cuenta lo siguiente:

- Azure Backup no admite Core OS Linux.
- Azure Backup no admite sistemas operativos de 32 bits.
- Otras distribuciones del tipo "traiga su propio Linux" podrían funcionar, siempre que el [agente de máquina virtual de Azure para Linux](../virtual-machines/extensions/agent-linux.md) esté disponible en la máquina virtual y haya compatibilidad con Python.
- Azure Backup no admite máquinas virtuales Linux con proxy configurado si no tienen la versión Python 2.7 instalada.

## <a name="backup-frequency-and-retention"></a>Frecuencia y retención de copias de seguridad

**Configuración** | **Límites**
--- | ---
Puntos de recuperación máximos por instancia protegida (máquina/carga de trabajo) | 9999.
Tiempo de expiración máximo de un punto de recuperación | Ilimitado.
Frecuencia máxima de copia de seguridad en un almacén (extensión de máquina virtual de Azure) | una vez al día.
Frecuencia máxima de copia de seguridad en el almacén (agente de MARS) | Tres copias de seguridad por día.
Frecuencia máxima de copia de seguridad en DPM/MABS | Cada 15 minutos para SQL Server.<br/><br/> Una vez cada hora para otras cargas de trabajo.
Retención de punto de recuperación | Diariamente, semanalmente, mensualmente y anualmente.
Período de retención máximo | Depende de la frecuencia de la copia de seguridad.
Puntos de recuperación en disco DPM/MABS | 64 para servidores de archivos y 448 para servidores de aplicaciones.<br/><br/> Los puntos de recuperación de cinta son ilimitados para DPM en el entorno local.

## <a name="supported-restore-methods"></a>Métodos de restauración admitidos

**Opción de restauración** | **Detalles**
--- | ---
**Crear una máquina virtual** | Crea y pone en funcionamiento rápidamente una máquina virtual básica a partir de un punto de restauración.<br/><br/> Puede especificar un nombre para la máquina virtual, seleccionar el grupo de recursos y la red virtual (VNet) en que se va a colocar y especificar una cuenta de almacenamiento para la máquina virtual restaurada. La nueva máquina virtual debe crearse en la misma región que la máquina virtual de origen.
**Restaurar disco** | Restaura un disco de máquina virtual, que luego se puede usar para crear una máquina virtual.<br/><br/> Azure Backup proporciona una plantilla para ayudar a personalizar y crear una máquina virtual. <br/><br> El trabajo de restauración genera una plantilla que puede descargar y usar para especificar la configuración de una máquina virtual personalizada y crear una máquina virtual.<br/><br/> Los discos se copian en el grupo de recursos que especifique.<br/><br/> Como alternativa, puede conectar el disco a una máquina virtual existente o crear una máquina virtual mediante PowerShell.<br/><br/> Esta opción es útil si desea personalizar la máquina virtual, agregar la configuración que no existía en el momento de la copia de seguridad o agregar valores que deben configurarse mediante la plantilla o PowerShell.
**Reemplazar el existente** | Puede restaurar un disco y usarlo para reemplazar un disco en la máquina virtual existente.<br/><br/> La máquina virtual actual debe existir. Si se ha eliminado, esta opción no se puede usar.<br/><br/> Azure Backup toma una instantánea de la máquina virtual existente antes de reemplazar el disco, y la almacena en la ubicación de almacenamiento provisional especificada. Los discos existentes conectados a la máquina virtual se reemplazan por el punto de restauración seleccionado.<br/><br/> La instantánea se copia en el almacén y se conserva de acuerdo con la directiva de retención. <br/><br/> Después de la operación de reemplazo de disco, el disco original se conserva en el grupo de recursos. Puede optar por eliminar manualmente los discos originales si no son necesarios. <br/><br/>Esta opción es compatible con máquinas virtuales administradas no cifradas. No se admite para discos no administrados, [máquinas virtuales generalizadas](../virtual-machines/windows/capture-image-resource.md) o para máquinas virtuales [creadas con imágenes personalizadas](https://azure.microsoft.com/resources/videos/create-a-custom-virtual-machine-image-in-azure-resource-manager-with-powershell/).<br/><br/> Si el punto de restauración tiene más o menos discos que la máquina virtual actual, el número de discos del punto de restauración solo reflejará la configuración de la máquina virtual.<br><br> También se admite el reemplazo de instancias existentes para las VM con recursos vinculados, como [identidades administradas asignadas por el usuario](../active-directory/managed-identities-azure-resources/overview.md) y [Key Vault](../key-vault/general/overview.md).
**Entre regiones (región secundaria)** | La restauración entre regiones puede usarse para restaurar VM de Azure en la región secundaria, que es una [región emparejada de Azure](../best-practices-availability-paired-regions.md#what-are-paired-regions).<br><br> Puede restaurar todas las VM de Azure del punto de recuperación seleccionado si la copia de seguridad se realiza en la región secundaria.<br><br> Esta característica está disponible para las opciones siguientes:<br> <li> [Crear una máquina virtual](./backup-azure-arm-restore-vms.md#create-a-vm) <br> <li> [Restaurar discos](./backup-azure-arm-restore-vms.md#restore-disks) <br><br> Actualmente no se admite la opción [Reemplazar los discos existentes](./backup-azure-arm-restore-vms.md#replace-existing-disks).<br><br> Permisos<br> La operación de restauración en la región secundaria pueden llevarla a cabo los administradores de copias de seguridad y los administradores de aplicaciones.

## <a name="support-for-file-level-restore"></a>Compatibilidad con restauración de nivel de archivo

**Restauración** | **Compatible**
--- | ---
Restaurar archivos entre sistemas operativos | Puede restaurar archivos en cualquier máquina que tenga el mismo sistema operativo que la máquina virtual de copia de seguridad, o bien uno compatible. Consulte la [tabla de sistemas operativos compatibles](backup-azure-restore-files-from-vm.md#system-requirements).
Restaurar archivos desde máquinas virtuales cifradas | No compatible.
Restaurar archivos desde cuentas de almacenamiento con acceso restringido a la red | No compatible.
Restaurar archivos en máquinas virtuales con espacios de almacenamiento de Windows | La restauración no se admite en la misma máquina virtual.<br/><br/> En su lugar, restaure los archivos en una máquina virtual compatible.
Restaurar archivos en máquinas virtuales Linux con matrices RAID/LVM | La restauración no se admite en la misma máquina virtual.<br/><br/> Restaure en una máquina virtual compatible.
Restaurar archivos con una configuración de red especial | La restauración no se admite en la misma máquina virtual. <br/><br/> Restaure en una máquina virtual compatible.

## <a name="support-for-vm-management"></a>Compatibilidad con la administración de máquina virtual

En la tabla siguiente se resume la compatibilidad con copias de seguridad durante las tareas de administración de máquinas virtuales, como agregar o reemplazar discos de máquina virtual.

**Restauración** | **Compatible**
--- | ---
Restaurar por suscripción, región o zona. | No compatible.
Restaurar a una máquina virtual existente | Utilice la opción Reemplazar disco.
Restaurar disco con cuenta de almacenamiento habilitada para Azure Storage Service Encryption (SSE) | No compatible.<br/><br/> Restaure en una cuenta que no tenga SSE habilitado.
Restaurar en cuentas de almacenamiento mixtas |No compatible.<br/><br/> Según el tipo de cuenta de almacenamiento, todos los discos restaurados serán premium o estándar y no mixtos.
Restaurar la máquina virtual directamente en un conjunto de disponibilidad | En el caso de los discos administrados, puede restaurar el disco y usar la opción del conjunto de disponibilidad en la plantilla.<br/><br/> No se admite para discos no administrados. Para discos no administrados, restaure el disco y, después, cree una máquina virtual en el conjunto de disponibilidad.
Restaurar copia de seguridad de máquinas virtuales no administradas después de actualizar a máquinas virtuales administradas| Compatible.<br/><br/> Puede restaurar discos y después crear una máquina virtual administrada.
Restaurar máquina virtual a un punto de restauración antes de que la máquina virtual se migre a discos administrados | Compatible.<br/><br/> Restaure a discos no administrados (opción predeterminada), convierta los discos restaurados en discos administrados y cree una máquina virtual con los discos administrados.
Restaure una máquina virtual que se haya eliminado. | Compatible.<br/><br/> Puede restaurar la máquina virtual desde un punto de recuperación.
Restaurar una máquina virtual de controlador de dominio que forma parte de una configuración de varios controladores de dominio mediante el portal | Se admite si restaura el disco y crea una máquina virtual mediante PowerShell.
Restaurar una máquina virtual en una red virtual distinta |Compatible.<br/><br/> Las redes virtuales deben pertenecer a la misma suscripción y región.

## <a name="vm-compute-support"></a>Compatibilidad con proceso de máquina virtual

**Proceso** | **Soporte técnico**
--- | ---
Tamaño de VM |Cualquier tamaño de máquina virtual de Azure con al menos 2 núcleos de CPU y 1 GB de RAM<br/><br/> [Más información.](../virtual-machines/sizes.md)
Copia de seguridad de máquinas virtuales en [conjuntos de disponibilidad](../virtual-machines/availability.md#availability-sets) | Compatible.<br/><br/> No se puede restaurar una máquina virtual en un conjunto disponible con la opción para crear rápidamente una máquina virtual. En su lugar, cuando restaure la máquina virtual, restaure el disco y úselo para implementar una máquina virtual, o bien restaure un disco y úselo para reemplazar un disco existente.
Copia de seguridad de máquinas virtuales implementadas con la [ventaja de uso híbrido (HUB)](../virtual-machines/windows/hybrid-use-benefit-licensing.md) | Compatible.
Copia de seguridad de máquinas virtuales implementadas en un [conjunto de escalado](../virtual-machine-scale-sets/overview.md) |Compatible. [El modo de orquestación](../virtual-machine-scale-sets/orchestration-modes.md) debe establecerse en 2 para el dominio de error. El conjunto de disponibilidad no se admite.
Copia de seguridad de máquinas virtuales implementadas desde [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps?filters=virtual-machine-images)<br/><br/> (Publicado por Microsoft y terceros) |Compatible.<br/><br/> La máquina virtual debe ejecutar un sistema operativo compatible.<br/><br/> Al recuperar archivos en la máquina virtual, puede restaurar solo en un sistema operativo compatible (no en un sistema operativo anterior ni posterior). No restauramos las máquinas virtuales de Azure Marketplace cuya copia de seguridad se ha realizado como máquinas virtuales, ya que estas necesitan información de compra. Solo se restauran como discos.
Copia de seguridad de máquinas virtuales implementadas desde una imagen personalizada (terceros) |Compatible.<br/><br/> La máquina virtual debe ejecutar un sistema operativo compatible.<br/><br/> Al recuperar archivos en la máquina virtual, puede restaurar solo en un sistema operativo compatible (no en un sistema operativo anterior ni posterior).
Copia de seguridad de máquinas virtuales migradas a Azure| Compatible.<br/><br/> Para realizar copias de seguridad de la máquina virtual, el agente de máquina virtual debe estar instalado en la máquina migrada.
Copia de seguridad con coherencia con múltiples máquinas virtuales | Azure Backup no proporciona coherencia de datos y aplicaciones entre varias máquinas virtuales.
Copia de seguridad con [Configuración de diagnóstico](../azure-monitor/platform/platform-logs-overview.md)  | No compatible. <br/><br/> Si la restauración de la máquina virtual de Azure con la configuración de diagnóstico se desencadena mediante la opción [Crear nueva](backup-azure-arm-restore-vms.md#create-a-vm), se produce un error en la restauración.
Restauración de máquinas virtuales ancladas por zona | Compatible (para máquinas virtuales cuya copia de seguridad se ha realizado después de enero de 2019 y en las que hay [zonas de disponibilidad](https://azure.microsoft.com/global-infrastructure/availability-zones/) disponibles).<br/><br/>Actualmente se admite la restauración en la misma zona que está anclada en las máquinas virtuales. Sin embargo, si la zona no está disponible, se producirá un error en la restauración.
Máquinas virtuales de Gen2 | Compatible <br> Azure Backup admite la copia de seguridad y la restauración de [máquinas virtuales de Gen2](https://azure.microsoft.com/updates/generation-2-virtual-machines-in-azure-public-preview/). Cuando estas máquinas virtuales se restauran a partir del punto de recuperación, se restauran como [máquinas virtuales de Gen2](https://azure.microsoft.com/updates/generation-2-virtual-machines-in-azure-public-preview/).
Copia de seguridad de máquinas virtuales de Azure con bloqueos | No se admite para máquinas virtuales no administradas. <br><br> Se admite para máquinas virtuales administradas.
[Máquinas virtuales de Spot](../virtual-machines/spot-vms.md) | No compatible. Azure Backup restaura las máquinas virtuales de Spot como máquinas virtuales de Azure convencionales.

## <a name="vm-storage-support"></a>Compatibilidad con almacenamiento de máquina virtual

**Componente** | **Soporte técnico**
--- | ---
Discos de datos de máquinas virtuales de Azure | La compatibilidad con la copia de seguridad de máquinas virtuales de Azure con hasta 32 discos está en versión preliminar pública en todas las regiones.<br><br> La compatibilidad con la copia de seguridad de máquinas virtuales de Azure con discos no administrados o máquinas virtuales clásicas solo tiene hasta 16 discos.
Tamaño del disco de datos | El tamaño de disco individual puede ser de hasta 32 TB y se admite un máximo de 256 TB si se combinan todos los discos de una máquina virtual.
Tipo de almacenamiento | HDD estándar, SSD estándar y SSD Premium.
Discos administrados | Compatible.
Discos cifrados | Compatible.<br/><br/> Se puede realizar una copia de seguridad (con o sin la aplicación Azure AD) de las máquinas virtuales de Azure que tengan habilitado Azure Disk Encryption.<br/><br/> Las máquinas virtuales cifradas no se pueden recuperar a nivel de archivo o carpeta. Tiene que recuperar la máquina virtual completa.<br/><br/> Puede habilitar el cifrado en máquinas virtuales que ya estén protegidas con Azure Backup.
Discos con el Acelerador de escritura habilitado | No compatible.<br/><br/> Azure Backup excluye automáticamente los discos que tienen habilitado el Acelerador de escritura (WA) durante la copia de seguridad. Como no tienen copia de seguridad, no puede restaurar estos discos desde puntos de recuperación de la máquina virtual. <br><br> **Nota importante**: Las máquinas virtuales con discos WA necesitan conectividad a Internet para una copia de seguridad correcta (aunque esos discos se excluyan de la copia de seguridad).
Copia de seguridad y restauración de discos y máquinas virtuales desduplicados | Azure Backup no admite la desduplicación. Para más información, consulte este [artículo](./backup-support-matrix.md#disk-deduplication-support). <br/> <br/>  - Azure Backup no se desduplica entre máquinas virtuales en el almacén de Recovery Services <br/> <br/>  - Si hay máquinas virtuales en estado de desduplicación durante la restauración, los archivos no se pueden restaurar porque el almacén no entiende el formato. Sin embargo, puede realizar correctamente la restauración de una máquina virtual completa.
Agregar disco a una máquina virtual protegida | Compatible.
Cambiar tamaño de disco de una máquina virtual protegida | Compatible.
Almacenamiento compartido| No se admite la copia de seguridad de máquinas virtuales mediante el Volumen compartido de clúster (CSV) o el Servidor de archivos de escalabilidad horizontal. Es probable que los escritores de CSV experimenten errores durante la copia de seguridad. En la restauración, es posible que los discos que contienen volúmenes CSV no aparezcan.
[Discos compartidos](../virtual-machines/disks-shared-enable.md) | No compatible.

## <a name="vm-network-support"></a>Compatibilidad con red de VM

**Componente** | **Soporte técnico**
--- | ---
Número de interfaces de red (NIC) | Hasta el número máximo de NIC admitidas por un tamaño específico de máquina virtual de Azure.<br/><br/> Las NIC se crean cuando se crea la máquina virtual durante el proceso de restauración.<br/><br/> El número de NIC de la máquina virtual restaurada refleja el número de NIC de la máquina virtual cuando se habilita la protección. La eliminación de NIC después de habilitar la protección no afecta al recuento.
Equilibrador de carga interno y externo |Compatible. <br/><br/> [Obtenga más información](backup-azure-arm-restore-vms.md#restore-vms-with-special-configurations) sobre cómo restaurar máquinas virtuales con una configuración de red especial.
Varias direcciones IP reservadas |Compatible. <br/><br/> [Obtenga más información](backup-azure-arm-restore-vms.md#restore-vms-with-special-configurations) sobre cómo restaurar máquinas virtuales con una configuración de red especial.
Máquinas virtuales con varios adaptadores de red| Compatible. <br/><br/> [Obtenga más información](backup-azure-arm-restore-vms.md#restore-vms-with-special-configurations) sobre cómo restaurar máquinas virtuales con una configuración de red especial.
Máquinas virtuales con direcciones IP públicas| Compatible.<br/><br/> Asocie una dirección IP pública existente a la NIC, o cree una dirección y asóciela a la NIC después de realizar la restauración.
Grupo de seguridad de red (NSG) en la NIC o la subred. |Compatible.
Dirección IP estática | No compatible.<br/><br/> A una nueva máquina virtual que se crea a partir de un punto de restauración se le asigna una dirección IP dinámica.<br/><br/> En el caso de las máquinas virtuales clásicas, no puede realizar una copia de seguridad de una máquina virtual con una dirección IP reservada y ningún punto de conexión definido.
Dirección IP dinámica |Compatible.<br/><br/> Si la NIC de la máquina virtual de origen usa una dirección IP dinámica, de forma predeterminada, la NIC de la máquina virtual restaurada también la usará.
Administrador de tráfico de Azure| Compatible.<br/><br/>Si la máquina virtual de la que se ha realizado la copia de seguridad está en Traffic Manager, agregue manualmente la máquina virtual restaurada a la misma instancia de Traffic Manager.
Azure DNS |Compatible.
DNS personalizado |Compatible.
Conectividad saliente mediante un proxy HTTP | Compatible.<br/><br/> No se admite un proxy basado autenticado.
Puntos de conexión de servicio de red virtual| Compatible.<br/><br/> La configuración del firewall y de la cuenta de almacenamiento de la red virtual deben permitir el acceso desde todas las redes.

## <a name="vm-security-and-encryption-support"></a>Compatibilidad con cifrado y seguridad de máquina virtual

Azure Backup admite el cifrado para datos en tránsito y en reposo:

Tráfico de red en Azure:

- El l tráfico de copia de seguridad de servidores al almacén de Recovery Services se cifra mediante el Estándar de cifrado avanzado 256.
- Los datos de copia de seguridad se envían a través de un vínculo HTTPS seguro.
- Los datos de copia de seguridad se almacenan en el almacén de Recovery Services en su forma cifrada.
- Solo tiene la clave de cifrado para desbloquear estos datos. Microsoft no puede descifrar los datos de copia de seguridad en ningún momento.

  > [!WARNING]
  > Después de configurar el almacén, solo el usuario tiene acceso a la clave de cifrado. Microsoft nunca mantiene una copia y no tiene acceso a la clave. Si la clave se pierde, Microsoft no puede recuperar los datos de copia de seguridad.

Seguridad de los datos:

- Cuando se realiza la copia de seguridad de máquinas virtuales de Azure, se requiere la configuración del cifrado *en* la máquina virtual.
- Azure Backup admite Azure Disk Encryption, que usa BitLocker en máquinas virtuales Windows y **dm-crypt** en máquinas virtuales Linux.
- En el back-end, Azure Backup usa el [cifrado del servicio Azure Storage](../storage/common/storage-service-encryption.md), que protege los datos en reposo.

**Máquina** | **En tránsito** | **En reposo**
--- | --- | ---
Máquinas Windows locales sin DPM/MABS | ![Sí][green] | ![Sí][green]
Máquinas virtuales de Azure | ![Sí][green] | ![Sí][green]
Máquinas virtuales de Azure o locales con DPM | ![Sí][green] | ![Sí][green]
Máquinas virtuales de Azure o locales con MABS | ![Sí][green] | ![Sí][green]

## <a name="vm-compression-support"></a>Compatibilidad de compresión de máquina virtual

Backup admite la compresión del tráfico de copia de seguridad, tal y como se resume en la siguiente tabla. Tenga en cuenta lo siguiente:

- Para las máquinas virtuales de Azure, la extensión de la máquina virtual lee directamente los datos de la cuenta de Azure Storage a través de la red de almacenamiento. No es necesario comprimir este tráfico.
- Si utiliza DPM o MABS, puede comprimir los datos antes de realizar la copia de seguridad en DPM/MABS para ahorrar ancho de banda.

**Máquina** | **Comprimir a MABS/DPM (TCP)** | **Comprimir al almacén (HTTPS)**
--- | --- | ---
Máquinas Windows locales sin DPM/MABS | N/D | ![Sí][green]
Máquinas virtuales de Azure | N/D | N/D
Máquinas virtuales de Azure o locales con DPM | ![Sí][green] | ![Sí][green]
Máquinas virtuales de Azure o locales con MABS | ![Sí][green] | ![Sí][green]

## <a name="next-steps"></a>Pasos siguientes

- [Copia de seguridad de máquinas virtuales de Azure](backup-azure-arm-vms-prepare.md).
- [Copia de seguridad directa de máquinas Windows](tutorial-backup-windows-server-to-azure.md), sin un servidor de copia de seguridad.
- [Configuración de MABS](backup-azure-microsoft-azure-backup.md) para la copia de seguridad en Azure y, luego, copia de seguridad de las cargas de trabajo en MABS.
- [Configuración de DPM](backup-azure-dpm-introduction.md) para la copia de seguridad en Azure y, luego, copia de seguridad de las cargas de trabajo en DPM.

[green]: ./media/backup-support-matrix/green.png
[yellow]: ./media/backup-support-matrix/yellow.png
[red]: ./media/backup-support-matrix/red.png
