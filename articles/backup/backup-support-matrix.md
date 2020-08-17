---
title: Matriz de compatibilidad de Azure Backup
description: Proporciona un resumen de opciones de compatibilidad y limitaciones para el servicio Azure Backup.
ms.topic: conceptual
ms.date: 02/17/2019
ms.custom: references_regions
ms.openlocfilehash: d75e7053bfff14fbcb6deeae48c48f09e3e9ac0d
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87531887"
---
# <a name="support-matrix-for-azure-backup"></a>Matriz de compatibilidad para Azure Backup

Puede usar [Azure Backup](backup-overview.md) para realizar una copia de seguridad de los datos en la plataforma de nube de Microsoft Azure. En este artículo se resumen las configuraciones y limitaciones de compatibilidad generales para los escenarios e implementaciones de Azure Backup.

Otras matrices de compatibilidad están disponibles:

- Matriz de compatibilidad de la [copia de seguridad de máquinas virtuales de Azure](backup-support-matrix-iaas.md)
- Matriz de compatibilidad para la copia de seguridad mediante [System Center Data Protection Manager (DPM) o Microsoft Azure Backup Server (MABS)](backup-support-matrix-mabs-dpm.md)
- Matriz de compatibilidad para la copia de seguridad con el [agente de Microsoft Azure Recovery Services (MARS)](backup-support-matrix-mars-agent.md)

[!INCLUDE [azure-lighthouse-supported-service](../../includes/azure-lighthouse-supported-service.md)]

## <a name="vault-support"></a>Compatibilidad con almacenes

Azure Backup usa almacenes de Recovery Services para organizar y administrar las copias de seguridad, y para almacenar los datos con copia de seguridad realizada.

En esta tabla se describen las características de los almacenes de Recovery Services:

**Característica** | **Detalles**
--- | ---
**Vaults in subscription** (Almacenes en la suscripción) | Hasta 500 almacenes de Recovery Services en una suscripción única.
**Machines in a vault** (Máquinas en un almacén) | Hasta 1000 máquinas virtuales de Azure en un solo almacén.<br/><br/> Se pueden registrar hasta 50 servidores MABS en un único almacén.
**Orígenes de datos** | El tamaño máximo de un [origen de datos](./backup-azure-backup-faq.md#how-is-the-data-source-size-determined) individual es de 54 400 KB. Este límite no se aplica a las copias de seguridad de máquinas virtuales de Azure. No se aplica ningún límite a la cantidad total de datos de los que se puede hacer copia de seguridad en el almacén.
**Backups to vault** (Copias de seguridad en el almacén) | **Máquinas virtuales de Azure:** una vez al día.<br/><br/>**Máquinas protegidas por DPM/MABS:** dos veces al día.<br/><br/> **Máquinas con copia de seguridad realizada directamente con el agente de MARS:** tres veces al día.
**Backups between vaults** (Copias de seguridad entre almacenes) | Las copias de seguridad se realizan dentro de una región.<br/><br/> Necesita un almacén en cada región de Azure que contenga máquinas virtuales de las que desee realizar copias de seguridad. No se pueden realizar copias de seguridad en una región diferente.
**Move vaults** (Mover almacenes) | Puede [mover almacenes](./backup-azure-move-recovery-services-vault.md) entre suscripciones o entre grupos de recursos de la misma suscripción. Sin embargo, no se admite el traslado de almacenes entre regiones.
**Move data between vaults** (Movimiento de datos entre almacenes) | No se admite el movimiento de datos con copia de seguridad realizada entre almacenes.
**Modify vault storage type** (Modificación del tipo de almacenamiento de almacén) | Puede modificar el tipo de replicación de almacenamiento (almacenamiento con redundancia geográfica o almacenamiento con redundancia local) para un almacén antes de que se almacenen las copias de seguridad. Una vez iniciadas las copias de seguridad en el almacén, el tipo de replicación no se puede modificar.

## <a name="on-premises-backup-support"></a>Compatibilidad con las copias de seguridad locales

Esto es lo que se admite si quiere hacer copias de seguridad de equipos locales:

**Máquina** | **Elementos incluidos en la copia de seguridad** | **Ubicación** | **Características**
--- | --- | --- | ---
**Copia de seguridad directa de máquina Windows con el agente de MARS** | Archivos, carpetas, estado del sistema | Copia de seguridad en el almacén de Recovery Services. | Copia de seguridad tres veces al día.<br/><br/> Copia de seguridad sin reconocimiento de aplicaciones.<br/><br/> Restauración de archivos, carpetas y volúmenes.
**Copia de seguridad directa de máquina Linux con el agente de MARS** | No se admite la copia de seguridad.
**Copia de seguridad en DPM** | Archivos, carpetas, volúmenes, estado del sistema, datos de la aplicación. | Copia de seguridad en almacenamiento local de DPM. DPM, después, realiza copias de seguridad en el almacén. | Instantáneas con reconocimiento de aplicaciones<br/><br/> Granularidad completa para copia de seguridad y recuperación.<br/><br/> Linux admitido en máquinas virtuales (Hyper-V o VMware).<br/><br/> Oracle no se admite.
**Copia de seguridad en MABS** | Archivos, carpetas, volúmenes, estado del sistema, datos de la aplicación. | Copia de seguridad en almacenamiento local de MABS. Después, MABS realiza una copia de seguridad en el almacén. | Instantáneas con reconocimiento de aplicaciones<br/><br/> Granularidad completa para copia de seguridad y recuperación.<br/><br/> Linux admitido en máquinas virtuales (Hyper-V o VMware).<br/><br/> Oracle no se admite.

## <a name="azure-vm-backup-support"></a>Compatibilidad de copia de seguridad de máquinas virtuales de Azure

### <a name="azure-vm-limits"></a>Límites de máquina virtual de Azure

**Límite** | **Detalles**
--- | ---
**Discos de datos de máquinas virtuales de Azure** | Consulte la [matriz de compatibilidad para la copia de seguridad de Azure VM](./backup-support-matrix-iaas.md#vm-storage-support).
**Tamaño del disco de datos de la máquina virtual de Azure** | El tamaño de disco individual puede ser de hasta 32 TB y se admite un máximo de 256 TB si se combinan todos los discos de una máquina virtual.

### <a name="azure-vm-backup-options"></a>Opciones de copia de seguridad de máquinas virtuales de Azure

Esto es lo que se admite si desea hacer una copia de seguridad de las máquinas virtuales de Azure:

**Máquina** | **Elementos incluidos en la copia de seguridad** | **Ubicación** | **Características**
--- | --- | --- | ---
**Copia de seguridad de máquina virtual de Azure con la extensión de máquina virtual** | Toda la máquina virtual | Copia de seguridad en almacén. | Extensión instalada cuando se habilita la copia de seguridad para una máquina virtual.<br/><br/> Copia de seguridad una vez al día.<br/><br/> Copias de seguridad con reconocimiento de aplicaciones para máquinas virtuales Windows; copias de seguridad coherentes con archivos para máquinas virtuales Linux. Puede configurar la coherencia de aplicaciones para máquinas Linux con scripts personalizados.<br/><br/> Restaurar máquina virtual o disco.<br/><br/> No se puede realizar la copia de seguridad de una máquina virtual de Azure en una ubicación local.
**Copia de seguridad de máquina virtual de Azure con el agente de MARS** | Archivos, carpetas, estado del sistema | Copia de seguridad en almacén. | Copia de seguridad tres veces al día.<br/><br/> Si desea realizar copias de seguridad de archivos o carpetas específicos en lugar de toda la máquina virtual, el agente de MARS se puede ejecutar junto con la extensión de máquina virtual.
**Máquina virtual de Azure con DPM** | Archivos, carpetas, volúmenes, estado del sistema, datos de la aplicación. | Copia de seguridad en almacenamiento local de máquina virtual de Azure que se ejecuta en DPM. DPM, después, realiza copias de seguridad en el almacén. | Instantáneas con reconocimiento de aplicaciones.<br/><br/> Granularidad completa para la copia de seguridad y recuperación.<br/><br/> Linux admitido en máquinas virtuales (Hyper-V o VMware).<br/><br/> Oracle no se admite.
**Máquina virtual de Azure con MABS** | Archivos, carpetas, volúmenes, estado del sistema, datos de la aplicación. | Copia de seguridad en almacenamiento local de máquina virtual de Azure que se ejecuta en MABS. Después, MABS realiza una copia de seguridad en el almacén. | Instantáneas con reconocimiento de aplicaciones.<br/><br/> Granularidad completa para la copia de seguridad y recuperación.<br/><br/> Linux admitido en máquinas virtuales (Hyper-V o VMware).<br/><br/> Oracle no se admite.

## <a name="linux-backup-support"></a>Soporte técnico de copia de seguridad de Linux

Esto es lo que se admite si quiere hacer copias de seguridad de máquinas Linux:

**Tipo de copia de seguridad** | **Linux (reconocido por Azure)**
--- | ---
**Copia de seguridad directa de máquina local que ejecuta Linux** | No compatible. El agente de MARS solo puede instalarse en máquinas Windows.
**Uso de la extensión del agente para realizar la copia de seguridad de una máquina virtual de Azure que ejecuta Linux** | Copia de seguridad coherente con la aplicación mediante [scripts personalizados](backup-azure-linux-app-consistent.md).<br/><br/> Recuperación de nivel de archivo.<br/><br/> Restauración mediante la creación de una máquina virtual desde un disco o un punto de recuperación.
**Uso de DPM para realizar la copia de seguridad de máquinas locales que ejecutan Linux** | Copia de seguridad coherente con archivo de máquinas virtuales invitadas de Linux en Hyper-V y VMWare.<br/><br/> Restauración de máquinas virtuales invitadas de Linux en Hyper-V y VMWare.
**Uso de MABS para realizar la copia de seguridad de máquinas locales que ejecutan Linux** | Copia de seguridad coherente con archivo de máquinas virtuales invitadas de Linux en Hyper-V y VMWare.<br/><br/> Restauración de máquinas virtuales invitadas de Linux en Hyper-V y VMWare.
**Uso de MABS o DPM para realizar las copias de seguridad de máquinas virtuales Linux de Azure** | No compatible.

## <a name="daylight-saving-time-support"></a>Compatibilidad con horario de verano

Azure Backup no admite el ajuste automático del reloj para el horario de verano para realizar copias de seguridad de máquinas virtuales de Azure. No adelanta ni retrasa la hora de la copia de seguridad. Para asegurarse de que la copia de seguridad se ejecute en el momento deseado, modifique las directivas de copia de seguridad manualmente según sea necesario.

## <a name="disk-deduplication-support"></a>Compatibilidad con desduplicación de disco

La compatibilidad con la desduplicación de disco es la siguiente:

- La desduplicación de disco puede realizarse en el entorno local cuando se utiliza DPM o MABS para hacer copias de seguridad de las máquinas virtuales de Hyper-V que ejecutan Windows. Windows Server realiza la desduplicación de los datos (en el nivel de host) en discos duros virtuales (VHD) que están conectados a la máquina virtual como almacenamiento de copia de seguridad.
- La desduplicación no se admite en Azure para ninguno de los componentes de Backup. Cuando DPM y MAB se implementan en Azure, los discos de almacenamiento asociados a la máquina virtual no se pueden desduplicar.

## <a name="security-and-encryption-support"></a>Compatibilidad con cifrado y seguridad

Azure Backup admite el cifrado para datos en tránsito y en reposo.

### <a name="network-traffic-to-azure"></a>Tráfico de red en Azure

- El l tráfico de copia de seguridad de servidores al almacén de Recovery Services se cifra mediante el Estándar de cifrado avanzado 256.
- Los datos de copia de seguridad se envían a través de un vínculo HTTPS seguro.

### <a name="data-security"></a>Seguridad de los datos

- Los datos de copia de seguridad se almacenan en el almacén de Recovery Services en su forma cifrada.
- Cuando se realiza una copia de seguridad de los datos desde servidores locales con el agente de MARS, los datos se cifran con una frase de contraseña antes de cargarse en Azure Backup y se descifran solo después de descargarse de Azure Backup.
- Cuando se realiza la copia de seguridad de máquinas virtuales de Azure, se requiere la configuración del cifrado *en* la máquina virtual.
- Azure Backup admite Azure Disk Encryption, que usa BitLocker en máquinas virtuales Windows y **dm-crypt** en máquinas virtuales Linux.
- En el back-end, Azure Backup usa el [cifrado del servicio Azure Storage](../storage/common/storage-service-encryption.md), que protege los datos en reposo.

**Máquina** | **En tránsito** | **En reposo**
--- | --- | ---
**Máquinas Windows locales sin DPM/MABS** | ![Sí][green] | ![Sí][green]
**Máquinas virtuales de Azure** | ![Sí][green] | ![Sí][green]
**Máquinas Windows locales o máquinas virtuales de Azure con DPM** | ![Sí][green] | ![Sí][green]
**Máquinas Windows locales o máquinas virtuales de Azure con MABS** | ![Sí][green] | ![Sí][green]

## <a name="compression-support"></a>Compatibilidad de compresión

Backup admite la compresión del tráfico de copia de seguridad, tal y como se resume en la siguiente tabla.

- Para las máquinas virtuales de Azure, la extensión de la máquina virtual lee directamente los datos de la cuenta de Azure Storage a través de la red de almacenamiento, no es necesario comprimir este tráfico.
- Si usa DPM o MABS, puede comprimir los datos antes de realizar la copia de seguridad para ahorrar ancho de banda.

**Máquina** | **Comprimir a MABS/DPM (TCP)** | **Comprimir al almacén (HTTPS)**
--- | --- | ---
**Copia de seguridad directa de máquinas Windows locales** | N/D | ![Sí][green]
**Copia de seguridad de máquinas virtuales de Azure con la extensión de máquina virtual** | N/D | N/D
**Copia de seguridad en máquinas locales o de Azure mediante DPM/MABS** | ![Sí][green] | ![Sí][green]

## <a name="retention-limits"></a>Límites de retención

**Configuración** | **Límites**
--- | ---
**Número máximo de puntos de recuperación por instancia protegida (máquina o carga de trabajo)** | 9999
**Tiempo máximo de expiración de un punto de recuperación** | Sin límite
**Frecuencia máxima de copia de seguridad en DPM/MABS** | Cada 15 minutos para SQL Server<br/><br/> Una vez cada hora para otras cargas de trabajo.
**Frecuencia máxima de copia de seguridad en el almacén** | **Máquinas Windows locales o máquinas virtuales de Azure que ejecutan MARS:** tres al día.<br/><br/> **DPM/MABS:** Dos por día<br/><br/> **Copia de seguridad de la máquina virtual de Azure:** una al día.
**Retención de punto de recuperación** | Diariamente, semanalmente, mensualmente y anualmente
**Período de retención máximo** | Depende de la frecuencia de la copia de seguridad
**Puntos de recuperación en disco DPM/MABS** | 64 para servidores de archivos; 448 para servidores de aplicaciones. <br/><br/>Puntos de recuperación de cinta ilimitados para DPM en el entorno local.

## <a name="cross-region-restore"></a>Restauración entre regiones

Azure Backup ha agregado la característica Restauración entre regiones para reforzar la disponibilidad de los datos y la capacidad de resistencia, lo que otorga a los clientes control total para restaurar los datos en una región secundaria. Para configurar esta característica, visite [el artículo Establecimiento de la restauración entre regiones](backup-create-rs-vault.md#set-cross-region-restore). Esta característica es compatible con los siguientes tipos de administración:

| Tipo de administración de copias de seguridad | Compatible                                                    | Regiones admitidas |
| ---------------------- | ------------------------------------------------------------ | ----------------- |
| Azure VM               | Sí.   Se admite para las máquinas virtuales cifradas y las máquinas virtuales con discos de menos de 4 TB. | Todas las regiones públicas de Azure.  |
| Agente de MARS/local | No                                                           | N/D               |
| SQL/SAP HANA          | No                                                           | N/D               |
| AFS                    | No                                                           | N/D               |

## <a name="next-steps"></a>Pasos siguientes

- [Revise la matriz de compatibilidad](backup-support-matrix-iaas.md) para la copia de seguridad de máquina virtual de Azure.

[green]: ./media/backup-support-matrix/green.png
[yellow]: ./media/backup-support-matrix/yellow.png
[red]: ./media/backup-support-matrix/red.png
