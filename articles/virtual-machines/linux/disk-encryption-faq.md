---
title: 'Preguntas más frecuentes: Azure Disk Encryption para máquinas virtuales Linux'
description: En este artículo se ofrecen respuestas a las preguntas más frecuentes sobre Microsoft Azure Disk Encryption para máquinas virtuales IaaS Linux.
author: msmbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: conceptual
ms.author: mbaldwin
ms.date: 06/05/2019
ms.custom: seodec18
ms.openlocfilehash: 232bc43440979a08da4a0e77d9b49bf56fdb0ae4
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87374052"
---
# <a name="azure-disk-encryption-for-linux-virtual-machines-faq"></a>Preguntas más frecuentes sobre Azure Disk Encryption para máquinas virtuales Linux

En este artículo se ofrecen respuestas a las preguntas más frecuentes sobre Azure Disk Encryption para máquinas virtuales (VM) Linux. Para más información sobre este servicio, consulte [Introducción a Azure Disk Encryption](disk-encryption-overview.md).

## <a name="what-is-azure-disk-encryption-for-linux-vms"></a>¿Qué es Azure Disk Encryption para VM Linux?

Azure Disk Encryption para VM Linux usa la característica dm-crypt de Linux para proporcionar el cifrado de disco completo del disco del sistema operativo* y los discos de datos. Además, proporciona cifrado del disco temporal cuando se usa la característica [EncryptFormatAll](disk-encryption-linux.md#use-encryptformatall-feature-for-data-disks-on-linux-vms). Los flujos de contenido se cifran desde la VM hasta el back-end de almacenamiento. Por lo tanto, se proporciona el cifrado de un extremo a otro con una clave administrada por el cliente.
 
Consulte [VM y sistemas operativos compatibles](disk-encryption-overview.md#supported-vms-and-operating-systems).

## <a name="where-is-azure-disk-encryption-in-general-availability-ga"></a>¿Dónde está Azure Disk Encryption en la disponibilidad general (GA)?

Azure Disk Encryption para máquinas virtuales Linux tiene disponibilidad general en todas las regiones públicas de Azure.

## <a name="what-user-experiences-are-available-with-azure-disk-encryption"></a>¿Qué experiencias del usuario están disponibles con Azure Disk Encryption?

La disponibilidad general de Azure Disk Encryption admite plantillas de Azure Resource Manager, Azure PowerShell y CLI de Azure. La diferentes experiencias de usuario le proporcionan flexibilidad. Dispone de tres opciones distintas para habilitar el cifrado de discos para las máquinas virtuales. Para más información sobre la experiencia del usuario e instrucciones paso a paso disponibles en Azure Disk Encryption, consulte [Escenarios de Azure Disk Encryption para Linux](disk-encryption-linux.md).

## <a name="how-much-does-azure-disk-encryption-cost"></a>¿Cuánto cuesta Azure Disk Encryption?

El cifrado de discos de máquinas virtuales con Azure Disk Encryption es gratuito, pero hay cargos asociados al uso de Azure Key Vault. Para más información acerca de los costos de Azure Key Vault, consulte la página de [precios de Key Vault](https://azure.microsoft.com/pricing/details/key-vault/).

## <a name="how-can-i-start-using-azure-disk-encryption"></a>¿Cómo puedo empezar a usar Azure Disk Encryption?

Para empezar, lea la [información general sobre Azure Disk Encryption](disk-encryption-overview.md).

## <a name="what-vm-sizes-and-operating-systems-support-azure-disk-encryption"></a>¿Qué tamaños de máquina virtual y sistemas operativos admiten Azure Disk Encryption?

En el artículo [Introducción a Azure Disk Encryption](disk-encryption-overview.md) se enumeran los [tamaños de máquina virtual](disk-encryption-overview.md#supported-vms) y los [sistemas operativos de máquina virtual](disk-encryption-overview.md#supported-operating-systems) compatibles con Azure Disk Encryption.

## <a name="can-i-encrypt-both-boot-and-data-volumes-with-azure-disk-encryption"></a>¿Puedo cifrar los volúmenes de datos y arranque con Azure Disk Encryption?

Sí, puede cifrar los volúmenes de datos y arranque, o bien puede cifrar el volumen de datos sin tener que cifrar primero el volumen del sistema operativo. 

Una vez que se haya cifrado el volumen del sistema operativo no se admite la deshabilitación del cifrado en dicho volumen. Para máquinas virtuales Linux en un conjunto de escalado, solo se puede cifrar el volumen de datos.

## <a name="can-i-encrypt-an-unmounted-volume-with-azure-disk-encryption"></a>¿Puedo cifrar un volumen desmontado con Azure Disk Encryption?

No. Azure Disk Encryption solo cifra volúmenes montados.

## <a name="what-is-storage-server-side-encryption"></a>¿Qué es el cifrado del lado servidor de almacenamiento?

El cifrado del lado servidor de almacenamiento cifra Azure Managed Disks en Azure Storage. Los discos administrados se cifran de manera predeterminada con el cifrado del lado servidor con una clave administrada por la plataforma (desde el 10 de junio de 2017). Puede administrar el cifrado de discos administrados con sus propias claves mediante la especificación de una clave administrada por el cliente. Para obtener más información, consulte: [Cifrado del lado servidor de Azure Managed Disks](disk-encryption.md).
 
## <a name="how-is-azure-disk-encryption-different-from-storage-server-side-encryption-with-customer-managed-key-and-when-should-i-use-each-solution"></a>¿Cuál es diferencia entre Azure Disk Encryption y el cifrado del lado servidor de almacenamiento con la clave administrada por el cliente y cuándo debo usar cada solución?

Azure Disk Encryption proporciona cifrado de un extremo a otro para el disco del sistema operativo, los discos de datos y el disco temporal mediante una clave administrada por el cliente.
- Si los requisitos incluyen el cifrado de todos los elementos anteriores y el cifrado de un extremo a otro, use Azure Disk Encryption. 
- Si los requisitos incluyen el cifrado solo de datos en reposo con la clave administrada por el cliente, use el [cifrado del lado servidor con las claves administradas por el cliente](disk-encryption.md). No se puede cifrar un disco con Azure Disk Encryption y el cifrado del lado servidor de almacenamiento con claves administradas por el cliente. 
- Si su distribución de Linux no se incluye en los [sistemas operativos compatibles con Azure Disk Encryption](disk-encryption-overview.md#supported-operating-systems) o usa un escenario que se ha mencionado en los [escenarios no compatibles con Windows](disk-encryption-linux.md#unsupported-scenarios), considere la posibilidad de usar el [cifrado del lado servidor con las claves administradas por el cliente](disk-encryption.md).
- Si la directiva de la organización permite cifrar contenido en reposo con una clave administrada por Azure, no es necesario realizar ninguna acción: el contenido se cifra de manera predeterminada. En el caso de los discos administrados, el contenido del almacenamiento se cifra de manera predeterminada con el cifrado del lado servidor con una clave administrada por la plataforma. El servicio de Azure Storage administra la clave. 



## <a name="how-do-i-rotate-secrets-or-encryption-keys"></a>¿Cómo se pueden rotar los secretos o las claves de cifrado?

Para rotar los secretos, simplemente llame al mismo comando que usó originalmente para habilitar el cifrado de disco y especifique un almacén de claves diferente. Para rotar la clave de cifrado de claves, llame al mismo comando que usó originalmente para habilitar el cifrado de disco y especifique el cifrado de claves nuevo. 

>[!WARNING]
> - Si usó anteriormente [Azure Disk Encryption con la aplicación Azure AD](disk-encryption-linux-aad.md) para al especificar las credenciales de Azure AD para cifrar esta VM, tendrá que seguir usando esta opción para cifrar la VM. Azure Disk Encryption no se puede usar en esta VM cifrada, ya que no es un escenario compatible, lo que significa que el cambio desde la aplicación de AAD a esta VM cifrada aún no es compatible.

## <a name="how-do-i-add-or-remove-a-key-encryption-key-if-i-didnt-originally-use-one"></a>¿Cómo se agrega o quita una clave de cifrado de clave si originalmente no usé una?

Para agregar una clave de cifrado de clave, llame al comando enable otra vez y pase el parámetro de clave de cifrado de clave. Para quitar una clave de cifrado de clave, llame al comando enable otra vez y pase el parámetro de clave de cifrado de clave.

## <a name="does-azure-disk-encryption-allow-you-to-bring-your-own-key-byok"></a>¿Permite Azure Disk Encryption habilitar la funcionalidad "traiga su propia clave" (BYOK)?

Sí, puede proporcionar sus propias claves de cifrado de claves. Dichas claves están protegidas en Azure Key Vault, que es el almacén de claves de Azure Disk Encryption. Para más información sobre los escenarios de compatibilidad de las claves de cifrado de claves, consulte [Creación y configuración de un almacén de claves para Azure Disk Encryption](disk-encryption-key-vault.md).

## <a name="can-i-use-an-azure-created-key-encryption-key"></a>¿Puedo usar una clave de cifrado de claves creada en Azure?

Sí, puede usar Azure Key Vault para generar la clave de cifrado de claves para usar en Azure Disk Encryption. Dichas claves están protegidas en Azure Key Vault, que es el almacén de claves de Azure Disk Encryption. Para más información sobre la clave de cifrado de claves, consulte [Creación y configuración de un almacén de claves para Azure Disk Encryption](disk-encryption-key-vault.md).

## <a name="can-i-use-an-on-premises-key-management-service-or-hsm-to-safeguard-the-encryption-keys"></a>¿Puedo usar el servicio de administración de claves local o HSM para proteger las claves de cifrado?

No puede utilizar el servicio de administración de claves local ni HSM para proteger las claves de cifrado con Azure Disk Encryption. Para proteger las claves de cifrado, solo se puede utilizar el servicio Azure Key Vault. Para más información sobre los escenarios de compatibilidad de las claves de cifrado de claves, consulte [Creación y configuración de un almacén de claves para Azure Disk Encryption](disk-encryption-key-vault.md).

## <a name="what-are-the-prerequisites-to-configure-azure-disk-encryption"></a>¿Cuáles son los requisitos previos para configurar Azure Disk Encryption?

Hay requisitos previos para Azure Disk Encryption. Consulte el artículo [Creación y configuración de un almacén de claves para Azure Disk Encryption](disk-encryption-key-vault.md) para crear un almacén de claves o configurar un almacén de claves existente para el acceso al cifrado de discos para habilitar el cifrado y proteger los secretos y las claves. Para más información sobre los escenarios de compatibilidad de las claves de cifrado de claves, consulte [Creación y configuración de un almacén de claves para Azure Disk Encryption](disk-encryption-key-vault.md).

## <a name="what-are-the-prerequisites-to-configure-azure-disk-encryption-with-an-azure-ad-app-previous-release"></a>¿Cuáles son los requisitos previos para configurar Azure Disk Encryption con una aplicación de Azure AD (versión anterior)?

Hay requisitos previos para Azure Disk Encryption. Consulte el contenido de [Azure Disk Encryption con Azure AD](disk-encryption-linux-aad.md) para crear una aplicación de Azure Active Directory, crear un almacén de claves o configurar el existente para el acceso al cifrado de disco y habilitar el cifrado y proteger los secretos y las claves. Para más información sobre los escenarios de compatibilidad de las claves de cifrado de claves, consulte [Creación y configuración de un almacén de claves para Azure Disk Encryption con Azure AD](disk-encryption-key-vault-aad.md).

## <a name="is-azure-disk-encryption-using-an-azure-ad-app-previous-release-still-supported"></a>¿Sigue siendo compatible Azure Disk Encryption con una aplicación de Azure AD (versión anterior)?
Sí. Todavía se admite el cifrado de disco mediante una aplicación de Azure AD. Sin embargo, al cifrar nuevas máquinas virtuales, se recomienda utilizar el nuevo método en lugar de cifrar con una aplicación de Azure AD. 

## <a name="can-i-migrate-vms-that-were-encrypted-with-an-azure-ad-app-to-encryption-without-an-azure-ad-app"></a>¿Se pueden migrar las máquinas virtuales cifradas con una aplicación de Azure AD para el cifrado sin una aplicación de Azure AD?
  En la actualidad, no existe una ruta de migración directa para los equipos que se cifraron con una aplicación de Azure AD al cifrado sin una aplicación de Azure AD. Además, tampoco hay una ruta directa desde el cifrado sin una aplicación Azure AD al cifrado con una aplicación de AD. 

## <a name="what-version-of-azure-powershell-does-azure-disk-encryption-support"></a>¿Qué versión de Azure PowerShell es compatible con Azure Disk Encryption?

Utilice la versión más reciente del SDK de Azure PowerShell para configurar Azure Disk Encryption. Descargue la versión más reciente de [Azure PowerShell](https://github.com/Azure/azure-powershell/releases). La versión 1.1.0 del SDK de Azure *no* admite Azure Disk Encryption.

> [!NOTE]
> La extensión de versión preliminar de Azure Disk Encryption para Linux "Microsoft.OSTCExtension.AzureDiskEncryptionForLinux" está en desuso. Esta extensión se publicó para la versión preliminar de Azure Disk Encryption. No debería usar la versión preliminar de la extensión en la implementación de prueba ni producción.

> Para escenarios de implementación, como Azure Resource Manager (ARM), donde necesita implementar la extensión Azure Disk Encryption para VM de Linux con el fin de habilitar el cifrado en la VM IaaS de Linux, debe usar la extensión Azure Disk Encryption compatible con entornos de producción "Microsoft.Azure.Security.AzureDiskEncryptionForLinux".

## <a name="can-i-apply-azure-disk-encryption-on-my-custom-linux-image"></a>¿Puedo aplicar Azure Disk Encryption en mi imagen personalizada de Linux?

No puede aplicar Azure Disk Encryption a una imagen personalizada de Linux. Solo se admiten las imágenes de Linux de la galería para las distribuciones admitidas mencionadas anteriormente. Actualmente no se admiten imágenes personalizadas de Linux.

## <a name="can-i-apply-updates-to-a-linux-red-hat-vm-that-uses-the-yum-update"></a>¿Puedo aplicar actualizaciones a una máquina virtual Red Hat de Linux utilizando la actualización de Yum?

Sí. Puede realizar una actualización de yum en una VM de Red Hat Linux.  Para más información, consulte [Azure Disk Encryption en una red aislada](disk-encryption-isolated-network.md).

## <a name="what-is-the-recommended-azure-disk-encryption-workflow-for-linux"></a>¿Cuál es el flujo de trabajo de Azure Disk Encryption recomendado para Linux?

Para obtener los mejores resultados en Linux, se recomienda el siguiente flujo de trabajo:
* Comenzar en la imagen de la galería en existencias sin modificar correspondiente a la distribución y versión necesarias del sistema operativo
* Realizar una copia de seguridad de las unidades montadas que se van a cifrar.  Esta copia de seguridad permite la recuperación en caso de error, por ejemplo, si la máquina virtual se reinicia antes de que finalice el cifrado.
* Cifrar (puede tardar varias horas o incluso días según las características de la máquina virtual y el tamaño de los discos de datos adjuntos)
* Personalizar y agregar software a la imagen según sea necesario.

Si este flujo de trabajo no es posible, el uso de [Storage Service Encryption](../../storage/common/storage-service-encryption.md) (SSE) en el nivel de la cuenta de almacenamiento de la plataforma puede ser una alternativa al cifrado del disco completo mediante dm-crypt.

## <a name="what-is-the-disk-bek-volume-or-mntazure_bek_disk"></a>¿Qué es el disco "volumen Bek" o "/mnt/azure_bek_disk"?

El "volumen Bek" es un volumen de datos locales que almacena de forma segura las claves de cifrado de las máquinas virtuales cifradas de Azure.
> [!NOTE]
> No elimine ni modifique ningún contenido de este disco. No desmonte el disco ya que la presencia de la clave de cifrado es necesaria para las operaciones de cifrado en la máquina virtual IaaS.


## <a name="what-encryption-method-does-azure-disk-encryption-use"></a>¿Qué método de cifrado usa Azure Disk Encryption?

Azure Disk Encryption usa el valor de descifrado predeterminado de aes-xts-plain64 con una clave maestra de volumen de 256 bits.

## <a name="if-i-use-encryptformatall-and-specify-all-volume-types-will-it-erase-the-data-on-the-data-drives-that-we-already-encrypted"></a>¿Si uso EncryptFormatAll y especifico todos los tipos de volumen se borrarán los datos en las unidades de datos que ya hemos cifrado?
No, no se borran datos de las unidades de datos que ya estén cifradas mediante Azure Disk Encryption. Igual que EncryptFormatAll no volvía a cifrar la unidad del SO, no volverá a cifrar la unidad de datos ya cifrada. Para obtener más información, consulte [Criterios de EncryptFormatAll](disk-encryption-linux.md#use-encryptformatall-feature-for-data-disks-on-linux-vms).        

## <a name="is-xfs-filesystem-supported"></a>¿Se admite el sistema de archivos XFS?
Se admite el cifrado de discos de sistema operativo de XFS.

Solo se admite el cifrado de discos de datos XFS cuando se usa el parámetro EncryptFormatAll. Esto volverá a formatear el volumen, por lo que se borrarán todos los datos que contenga. Para obtener más información, consulte [Criterios de EncryptFormatAll](disk-encryption-linux.md#use-encryptformatall-feature-for-data-disks-on-linux-vms).

## <a name="can-i-backup-and-restore-an-encrypted-vm"></a>¿Puedo hacer una copia de seguridad de una VM cifrada y restaurarla? 

Azure Backup proporciona un mecanismo para la copia de seguridad y restauración VM cifradas dentro de la misma suscripción y región.  Para obtener instrucciones, consulte [Copia de seguridad y restauración de máquinas virtuales cifradas con Azure Backup](../../backup/backup-azure-vms-encryption.md).  Actualmente, no se admite la restauración de una VM cifrada en una región diferente.  

## <a name="where-can-i-go-to-ask-questions-or-provide-feedback"></a>¿Dónde puedo formular preguntas o enviar comentarios?

Puede realizar preguntas o publicar comentarios en la [página de preguntas y respuestas de Microsoft sobre Azure Disk Encryption](/answers/topics/azure-disk-encryption.html).

## <a name="next-steps"></a>Pasos siguientes
En este documento, aprendió más acerca de las preguntas más frecuentes sobre Azure Disk Encryption. Para obtener más información acerca de este servicio, vea los siguientes artículos:

- [Introducción a Azure Disk Encryption](disk-encryption-overview.md)
- [Aplicación de cifrado de discos en Azure Security Center](../../security-center/security-center-virtual-machine-protection.md)
- [Cifrado de datos en reposo de Azure](../../security/fundamentals/encryption-atrest.md)
