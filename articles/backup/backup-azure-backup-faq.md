---
title: Respuestas a preguntas comunes
description: 'Respuestas a preguntas comunes sobre las características de Azure Backup, incluidos los almacenes de Recovery Services, las copias de seguridad que puede realizar, cómo funciona, el cifrado y los límites. '
ms.topic: conceptual
ms.date: 07/07/2019
ms.openlocfilehash: ea4cc792100edf59188a9be99c384747267dc0d8
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88892769"
---
# <a name="azure-backup---frequently-asked-questions"></a>Preguntas más frecuentes de Azure Backup

En este artículo se responde a preguntas habituales sobre el servicio Azure Backup.

## <a name="recovery-services-vault"></a>Almacén de Recovery Services

### <a name="is-there-any-limit-on-the-number-of-vaults-that-can-be-created-in-each-azure-subscription"></a>¿Hay algún límite del número de almacenes que se pueden crear en cada suscripción de Azure?

Sí. Se pueden crear hasta 500 almacenes de Recovery Services por cada región admitida de Azure Backup por suscripción. Si necesita más almacenes, cree otra suscripción.

### <a name="are-there-limits-on-the-number-of-serversmachines-that-can-be-registered-against-each-vault"></a>¿Hay algún límite en el número de servidores o máquinas que se pueden registrar en cada almacén?

Puede registrar hasta 1000 máquinas virtuales de Azure por almacén. Si usa el agente de Microsoft Azure Backup, puede registrar hasta 50 agentes de MARS por almacén. Así mismo, puede registrar 50 servidores de MABS o DPM en un almacén.

### <a name="how-many-datasourcesitems-can-be-protected-in-a-vault"></a>¿Cuántos orígenes de datos o elementos se pueden proteger en un almacén?

Puede proteger hasta 2000 orígenes de datos o elementos para todas las cargas de trabajo (por ejemplo, máquina virtual de IaaS, SQL, AFS, etc.) de un almacén.
Por ejemplo, si ya ha protegido 500 máquinas virtuales y 400 recursos compartidos de Azure Files en el almacén, no puede proteger más de 1 100 bases de datos SQL en él.

### <a name="how-many-policies-can-i-create-per-vault"></a>¿Cuántas directivas se pueden crear por almacén?

Solo puede tener hasta 200 directivas por almacén.

### <a name="if-my-organization-has-one-vault-how-can-i-isolate-data-from-different-servers-in-the-vault-when-restoring-data"></a>Si mi organización tiene un almacén, ¿cómo puedo aislar datos de servidores distintos en el almacén al restaurar los datos?

Los datos del servidor que desea recuperar de forma conjunta deben usar la misma frase de contraseña al configurar la copia de seguridad. Si desea aislar la recuperación en un uno o varios servidores específicos, use una frase de contraseña solo para dichos servidores. Por ejemplo, los servidores de recursos humanos podrían usar una frase de contraseña de cifrado, los servidores de contabilidad, otra y los servidores de almacenamiento, otra distinta.

### <a name="can-i-move-my-vault-between-subscriptions"></a>¿Puedo mover mi almacén entre suscripciones?

Sí. Para mover un almacén de Recovery Services, consulte este [artículo](backup-azure-move-recovery-services-vault.md)

### <a name="can-i-move-backup-data-to-another-vault"></a>¿Puedo mover datos de copia de seguridad a otro almacén?

No. Los datos de copia de seguridad almacenados en un almacén no se pueden mover a otro almacén.

### <a name="can-i-change-from-grs-to-lrs-after-a-backup"></a>¿Puedo cambiar de GRS a LRS después de una copia de seguridad?

El tipo de replicación de almacenamiento se establece de forma predeterminada como almacenamiento con redundancia geográfica. Una vez que configure la copia de seguridad, se deshabilitará la opción para modificarla y no se podrá cambiar.

![Tipo de replicación de almacenamiento](./media/backup-azure-backup-faq/storage-replication-type.png)

Si ya ha configurado la copia de seguridad y debe pasar de GRS a LRS, consulte [Procedimiento para cambiar de GRS a LRS después de configurar la copia de seguridad](backup-create-rs-vault.md#how-to-change-from-grs-to-lrs-after-configuring-backup).

### <a name="can-i-do-an-item-level-restore-ilr-for-vms-backed-up-to-a-recovery-services-vault"></a>¿Puedo hacer una restauración a nivel de elemento (ILR) de las máquinas virtuales de las que se realiza una copia de seguridad en un almacén de Recovery Services?

- ILR se admite para máquinas virtuales de Azure copiadas mediante la copia de seguridad de máquina virtual de Azure. Para más información, consulte este [artículo](backup-azure-restore-files-from-vm.md)
- ILR no se admite para puntos de recuperación en línea de máquinas virtuales locales cuya copia de seguridad se realice mediante el servidor de Azure Backup o System Center DPM.

## <a name="azure-backup-agent"></a>Agente de Azure Backup

### <a name="where-can-i-find-common-questions-about-the-azure-backup-agent-for-azure-vm-backup"></a>¿Dónde puedo encontrar preguntas comunes sobre el agente de Azure Backup para la copia de seguridad de máquinas virtuales de Azure?

- Para el agente que se ejecuta en máquinas virtuales de Azure, lea estas [preguntas más frecuentes](backup-azure-vm-backup-faq.md).
- En el caso del agente utilizado para realizar copias de seguridad de las carpetas de archivos de Azure, lea estas [preguntas frecuentes](backup-azure-file-folder-backup-faq.md).

## <a name="general-backup"></a>Copia de seguridad general

### <a name="are-there-limits-on-backup-scheduling"></a>¿Existen límites sobre la programación de copia de seguridad?

Sí.

- Puede hacer copias de seguridad de máquinas Windows Server o Windows hasta tres veces al día. Puede configurar la directiva de programación con programas diarios o semanales.
- Puede realizar copias de seguridad de DPM hasta dos veces al día. Puede configurar la directiva de programación con programas diarios, semanales, mensuales y anuales.
- Puede realizar copias de seguridad de las máquinas virtuales de Azure una vez al día.

### <a name="what-operating-systems-are-supported-for-backup"></a>¿Qué sistemas operativos se admiten para la copia de seguridad?

Azure Backup admite estos sistemas operativos para realizar copias de seguridad de archivos y carpetas y de aplicaciones protegidas con Azure Backup Server y DPM.

**SISTEMA OPERATIVO** | **SKU** | **Detalles**
--- | --- | ---
Estación de trabajo | |
Windows 10 64 bits | Enterprise, Pro, Home | Las máquinas deben ejecutar las actualizaciones y los Service Pack más recientes.
Windows 8.1 64 bits | Enterprise, Pro | Las máquinas deben ejecutar las actualizaciones y los Service Pack más recientes.
Windows 8 64 bits | Enterprise, Pro | Las máquinas deben ejecutar las actualizaciones y los Service Pack más recientes.
Windows 7 64 bits | Ultimate, Enterprise, Professional, Home Premium, Home Basic, Starter | Las máquinas deben ejecutar las actualizaciones y los Service Pack más recientes.
Server | |
Windows Server 2019 64 bits | Standard, Datacenter, Essentials | Con las actualizaciones y los Service Pack más recientes.
Windows Server 2016 64 bits | Standard, Datacenter, Essentials | Con las actualizaciones y los Service Pack más recientes.
Windows Server 2012 R2 64 bits | Standard, Datacenter, Foundation | Con las actualizaciones y los Service Pack más recientes.
Windows Server 2012 64 bits | Datacenter, Foundation, Standard | Con las actualizaciones y los Service Pack más recientes.
Windows Storage Server 2016 64 bits | Standard, Workgroup | Con las actualizaciones y los Service Pack más recientes.
Windows Storage Server 2012 R2 64 bits | Standard, Workgroup y Essential | Con las actualizaciones y los Service Pack más recientes.
Windows Storage Server 2012 64 bits | Standard, Workgroup | Con las actualizaciones y los Service Pack más recientes.
Windows Server 2008 R2 SP1 64 bits | Standard, Enterprise, Datacenter, Foundation | Con las últimas actualizaciones.
Windows Server 2008 64 bits | Standard, Enterprise y Datacenter | Con las últimas actualizaciones.

Azure Backup no admite sistemas operativos de 32 bits.

Para las copias de seguridad de VM Linux de Azure, Azure Backup admite [la lista de distribuciones aprobadas por Azure](../virtual-machines/linux/endorsed-distros.md), excepto Core OS Linux y sistemas operativos de 32 bits. Otras distribuciones del tipo "traiga su propio Linux" podrían funcionar, siempre que el agente de máquina virtual esté disponible en la máquina virtual y haya compatibilidad con Python.

### <a name="are-there-size-limits-for-data-backup"></a>¿Hay límites de tamaño para la copia de seguridad de datos?

Los límites de tamaño son los siguientes:

SO/máquina | Límite de tamaño del origen de datos
--- | ---
Windows 8 o posterior | 54 400 GB
Windows 7 |1700 GB
Windows Server 2012 o superior | 54 400 GB
Windows Server 2008, Windows Server 2008 R2 | 1700 GB
Azure VM | Consulte la [matriz de compatibilidad para la copia de seguridad de Azure VM](./backup-support-matrix-iaas.md#vm-storage-support).

### <a name="how-is-the-data-source-size-determined"></a>¿Cómo se determina el tamaño del origen de datos?

La tabla siguiente explica cómo se determina el tamaño de cada origen de datos.

**Origen de datos** | **Detalles**
--- | ---
Volumen |La cantidad de datos de los que se realiza copia de seguridad de una máquina virtual de un único volumen de la que se realiza la copia de seguridad.
Base de datos de SQL Server |Tamaño de una sola base de datos de la que se hace copia de seguridad.
SharePoint | Suma de las bases de datos de contenido y configuración de la granja de SharePoint de las que se hace copia de seguridad.
Exchange |Suma de todas las bases de datos de un servidor de Exchange de las que se hace copia de seguridad.
Estado del sistema y BMR |Cada copia individual del estado del sistema o BMR del equipo del que se hace copia de seguridad.

### <a name="is-there-a-limit-on-the-amount-of-data-backed-up-using-a-recovery-services-vault"></a>¿Hay un límite en la cantidad de datos de los que se realiza una copia de seguridad con el uso de un almacén de Recovery Services?

No hay ningún límite en la cantidad de datos de los que se puede hacer una copia de seguridad con el uso de un almacén de Recovery Services. Los orígenes de datos individuales (distintos de las máquinas virtuales de Azure) pueden tener un tamaño máximo de 54.400 GB. Para más información sobre los límites, consulte la sección sobre los [límites del almacén en la matriz de compatibilidad](./backup-support-matrix.md#vault-support).

### <a name="why-is-the-size-of-the-data-transferred-to-the-recovery-services-vault-smaller-than-the-data-selected-for-backup"></a>¿Por qué el tamaño de los datos transferidos al almacén de Recovery Services es más pequeño que los datos seleccionados para la copia de seguridad?

Los datos de los que se realiza una copia de seguridad desde el agente de Azure Backup, DPM y Azure Backup Server se comprimen y se cifran antes de ser transferidos. Una vez que se aplica la compresión y el cifrado, los datos del almacén se reducen entre un 30 % y un 40 %.

### <a name="can-i-delete-individual-files-from-a-recovery-point-in-the-vault"></a>¿Puedo eliminar archivos individuales desde un punto de recuperación en el almacén?

No, Azure Backup no admite la eliminación o la purga de elementos individuales de copias de seguridad almacenadas.

### <a name="if-i-cancel-a-backup-job-after-it-starts-is-the-transferred-backup-data-deleted"></a>Si se cancela un trabajo de copia de seguridad una vez iniciado, ¿se eliminan los datos de copia de seguridad transferidos?

No. Todos los datos transferidos al almacén, antes de que se cancelara el trabajo de copia de seguridad permanecen en el almacén.

- Azure Backup usa un mecanismo para agregar ocasionalmente agregar puntos de control a los datos de copia de seguridad durante la copia de seguridad.
- Debido a que hay puntos de control en los datos de copia de seguridad, el siguiente proceso de copia de seguridad puede validar la integridad de los archivos.
- El siguiente trabajo de copia de seguridad será incremental a los datos que ya están en la copia de seguridad. Las copias de seguridad incrementales solo transfieren los datos nuevos o modificados, lo que equivale a una mejor utilización del ancho de banda.

Si cancela un trabajo de copia de seguridad para una máquina virtual de Azure, se omiten los datos transferidos. El siguiente trabajo de copia de seguridad transfiere los datos incrementales desde el último trabajo de copia de seguridad correcto.

## <a name="retention-and-recovery"></a>Retención y recuperación

### <a name="are-the-retention-policies-for-dpm-and-windows-machines-without-dpm-the-same"></a>¿Las directivas de retención para DPM y máquinas Windows sin DPM son las mismas?

Sí, los dos tipos tienen directivas de retención diarias, semanales, mensuales y anuales.

### <a name="can-i-customize-retention-policies"></a>¿Puedo personalizar directivas de retención?

Sí, puede personalizarlas. Por ejemplo, puede configurar requisitos de retención semanales y diarios, pero no anuales ni mensuales.

### <a name="can-i-use-different-times-for-backup-scheduling-and-retention-policies"></a>¿Puedo usar distintas horas para la programación de copias de seguridad y las directivas de retención?

No. Las directivas de retención solo pueden aplicarse a puntos de copia de seguridad. Por ejemplo, esta imagen muestra una directiva de retención para las copias de seguridad realizadas a las 12 a. m. y a las 6 p. m.

![Programar copia de seguridad y retención](./media/backup-azure-backup-faq/Schedule.png)

### <a name="if-a-backup-is-kept-for-a-long-time-does-it-take-more-time-to-recover-an-older-data-point"></a>Si se conserva una copia de seguridad durante un período prolongado, ¿se tarda más tiempo en recuperar un punto de datos más antiguo?

No. El tiempo de recuperación del punto más antiguo o más reciente es el mismo. Cada punto de recuperación se comporta como un punto completo.

### <a name="if-each-recovery-point-is-like-a-full-point-does-it-impact-the-total-billable-backup-storage"></a>Si cada punto de recuperación es como un punto completo, ¿afecta esto al almacenamiento de copia de seguridad facturable total?

Los productos con un punto de retención a largo plazo típicos almacenan los datos de copia de seguridad como puntos completos.

- Los puntos completos *no son eficientes* para el almacenamiento, pero resultan más fáciles y rápidos de restaurar.
- Las copias incrementales son *eficientes* para el almacenamiento, pero requieren que se restaure una cadena de datos, lo que afecta al tiempo de recuperación.

La arquitectura de almacenamiento de Azure Backup le ofrece lo mejor de ambos mundos ya que permite almacenar de forma óptima datos para conseguir restauraciones más rápidas e incurrir en pocos costos de almacenamiento. Esto garantiza que el ancho de banda de entrada y salida se utilice de manera eficiente. La cantidad de almacenamiento de datos y el tiempo necesario para recuperar los datos se reducen al mínimo. Obtenga más información sobre [copias de seguridad incrementales](https://azure.microsoft.com/blog/microsoft-azure-backup-save-on-long-term-storage/).

### <a name="is-there-a-limit-on-the-number-of-recovery-points-that-can-be-created"></a>¿Hay un límite en el número de puntos de recuperación que se pueden crear?

Puede crear hasta 9999 puntos de recuperación por instancia protegida. Una instancia protegida es un equipo, un servidor (físico o virtual) o una carga de trabajo que realiza copias de seguridad en Azure.

- Obtenga más información sobre [copia de seguridad y retención](./backup-support-matrix.md).

### <a name="how-many-times-can-i-recover-data-thats-backed-up-to-azure"></a>¿Cuántas veces puedo recuperar datos de los que se ha realizado una copia de seguridad en Azure?

No hay ningún límite en cuanto al número de recuperaciones de Azure Backup.

### <a name="when-restoring-data-do-i-pay-for-the-egress-traffic-from-azure"></a>Al restaurar los datos, ¿tengo que pagar por el tráfico de salida de Azure?

No. Las recuperaciones son gratuitas y no se cobran por el tráfico de salida.

### <a name="what-happens-when-i-change-my-backup-policy"></a>¿Qué ocurre cuando se cambia mi directiva de copia de seguridad?

Cuando se aplica una nueva directiva, se sigue la programación y retención de la nueva directiva.

- Si se amplía la retención, los puntos de recuperación existentes se marcan para conservarlos de acuerdo con la nueva directiva.
- Si se reduce la retención, se marcan para eliminarse y, posteriormente, se eliminan en el siguiente trabajo de limpieza.

### <a name="how-long-is-data-retained-when-stopping-backups-but-selecting-the-option-to-retain-backup-data"></a>¿Cuánto tiempo se conservan los datos al detener las copias de seguridad con la opción de conservar datos de copia de seguridad seleccionada?

Cuando se detienen las copias de seguridad y se conservan los datos, dejan de aplicarse las reglas de directiva existentes para la eliminación de datos y los datos se conservan indefinidamente hasta que el administrador las inicia para eliminarlos.

## <a name="encryption"></a>Cifrado

### <a name="is-the-data-sent-to-azure-encrypted"></a>¿Se cifran los datos que se envían a Azure?

Sí. Los datos se cifran en la máquina local mediante AES256. Los datos se envían a través de un vínculo HTTPS seguro. Los datos se transmiten en la nube protegidos por el vínculo HTTPS solamente entre el servicio de recuperación y almacenamiento. El protocolo iSCSI protege los datos transmitidos entre el servicio de recuperación y la máquina del usuario. Se usa la tunelización segura para proteger el canal iSCSI.

### <a name="is-the-backup-data-on-azure-encrypted-as-well"></a>¿También se cifran los datos de copia de seguridad en Azure?

Sí. Los datos de Azure están cifrados en reposo.

- Para la copia de seguridad local, el cifrado en reposo se proporciona con la frase de contraseña que proporcione durante la copia de seguridad en Azure.
- Para máquinas virtuales de Azure, los datos están cifrados en reposo con Storage Service Encryption (SSE).

Microsoft no descifra los datos de copia de seguridad en ningún momento.

### <a name="what-is-the-minimum-length-of-the-encryption-key-used-to-encrypt-backup-data"></a>¿Cuál es la longitud mínima de la clave de cifrado usada para cifrar los datos de copia de seguridad?

La clave de cifrado usada por el agente de Microsoft Azure Recovery Services (MARS) se deriva de una frase de contraseña que debe tener 16 caracteres como mínimo. Para máquinas virtuales de Azure, no hay ningún límite de longitud para las claves usadas por Azure KeyVault.

### <a name="what-happens-if-i-misplace-the-encryption-key-can-i-recover-the-data-can-microsoft-recover-the-data"></a>¿Qué sucede si pierdo la clave de cifrado? ¿Puedo recuperar los datos? ¿Microsoft puede recuperar los datos?

La clave usada para cifrar los datos de copia de seguridad solo está presente en el sitio. Microsoft no conserva una copia en Azure y no tiene acceso a la clave. Si la clave se pierde, Microsoft no puede recuperar los datos de copia de seguridad.

## <a name="next-steps"></a>Pasos siguientes

Lea las otras preguntas más frecuentes:

- [Preguntas comunes](backup-azure-vm-backup-faq.md) sobre las copias de seguridad de máquinas virtuales de Azure
- [Preguntas comunes](backup-azure-file-folder-backup-faq.md) sobre el agente de Azure Backup
