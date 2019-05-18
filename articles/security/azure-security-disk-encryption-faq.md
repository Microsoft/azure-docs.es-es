---
title: 'Preguntas más frecuentes: Azure Disk Encryption para máquinas virtuales IaaS | Microsoft Docs'
description: En este artículo se ofrecen respuestas a las preguntas más frecuentes sobre Microsoft Azure Disk Encryption para máquinas virtuales IaaS con Windows y Linux.
author: msmbaldwin
ms.service: security
ms.topic: article
ms.author: mbaldwin
ms.date: 04/16/2019
ms.custom: seodec18
ms.openlocfilehash: 6fd9ce1947b8207aced44204fc2989622a1998f2
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65761919"
---
# <a name="azure-disk-encryption-for-iaas-vms-faq"></a>Preguntas más frecuentes de Azure Disk Encryption para máquinas virtuales IaaS

En este artículo se ofrecen respuestas a las preguntas más frecuentes (P+F) sobre Azure Disk Encryption para máquinas virtuales IaaS con Windows y Linux. Para más información sobre este servicio, consulte [Azure Disk Encryption para máquinas virtuales IaaS de Windows y Linux](azure-security-disk-encryption-overview.md).

## <a name="where-is-azure-disk-encryption-in-general-availability-ga"></a>¿Dónde está Azure Disk Encryption en la disponibilidad general (GA)?

Azure Disk Encryption para máquinas virtuales IaaS Windows y Linux tiene disponibilidad general en todas las regiones públicas de Azure.

## <a name="what-user-experiences-are-available-with-azure-disk-encryption"></a>¿Qué experiencias del usuario están disponibles con Azure Disk Encryption?

La disponibilidad general de Azure Disk Encryption admite plantillas de Azure Resource Manager, Azure PowerShell y CLI de Azure. La diferentes experiencias de usuario le proporcionan flexibilidad. Dispone de tres opciones distintas para habilitar el cifrado de disco para las máquinas virtuales IaaS. Para más información acerca de la experiencia del usuario e instrucciones paso a paso disponibles en Azure Disk Encryption, consulte [Habilitación de Azure Disk Encryption para Windows](azure-security-disk-encryption-windows.md) y [Habilitación de Azure Disk Encryption para Linux](azure-security-disk-encryption-linux.md).

## <a name="how-much-does-azure-disk-encryption-cost"></a>¿Cuánto cuesta Azure Disk Encryption?

No hay ningún cargo por cifrar discos de máquina virtual con Azure Disk Encryption, pero hay cargos asociados al uso de Azure Key Vault. Para más información acerca de los costos de Azure Key Vault, consulte la página de [precios de Key Vault](https://azure.microsoft.com/pricing/details/key-vault/).


## <a name="which-virtual-machine-tiers-does-azure-disk-encryption-support"></a>¿Qué niveles de máquina virtual admite Azure Disk Encryption?

Azure Disk Encryption está disponible en las máquinas virtuales de nivel estándar, incluidas las de IaaS de las series [A, D, DS, G, GS y F](https://azure.microsoft.com/pricing/details/virtual-machines/). También está disponible para las máquinas virtuales con almacenamiento premium. No está disponible en las máquinas virtuales de nivel básico.

## <a name="bkmk_LinuxOSSupport"></a>¿Qué distribuciones de Linux son compatibles con Azure Disk Encryption?

Azure Disk Encryption es compatible con un subconjunto de la [distribuciones de Linux aprobadas por Azure](../virtual-machines/linux/endorsed-distros.md), que es en sí mismo un subconjunto de todos los servidores posibles las distribuciones de Linux.

 ![Diagrama de Venn de distribuciones de Linux server que admiten Azure Disk Encryption](./media/azure-security-disk-encryption-faq/ade-supported-distros.png)

Las distribuciones de servidor de Linux que no están aprobadas por Azure no son compatibles con Azure Disk Encryption y, de los usuarios que están aprobadas, solo las siguientes distribuciones y versiones admiten Azure Disk Encryption:

| Distribución de Linux | Version | Tipo de volumen admitido para el cifrado|
| --- | --- |--- |
| Ubuntu | 18,04| Sistema operativo y disco de datos |
| Ubuntu | 16.04| Sistema operativo y disco de datos |
| Ubuntu | 14.04.5</br>[con kernel optimizado para Azure 4.15 o posterior](azure-security-disk-encryption-tsg.md#bkmk_Ubuntu14) | Sistema operativo y disco de datos |
| RHEL | 7.6 | Sistema operativo y disco de datos* |
| RHEL | 7.5 | Sistema operativo y disco de datos* |
| RHEL | 7.4 | Sistema operativo y disco de datos* |
| RHEL | 7.3 | Sistema operativo y disco de datos* |
| RHEL | 7.2 | Sistema operativo y disco de datos* |
| RHEL | 6.8 | Disco de datos* |
| RHEL | 6.7 | Disco de datos* |
| CentOS | 7.5 | Sistema operativo y disco de datos |
| CentOS | 7.4 | Sistema operativo y disco de datos |
| CentOS | 7.3 | Sistema operativo y disco de datos |
| CentOS | 7.2n | Sistema operativo y disco de datos |
| CentOS | 6.8 | Disco de datos |
| openSUSE | 42.3 | Disco de datos |
| SLES | 12-SP4 | Disco de datos |
| SLES | 12-SP3 | Disco de datos |

> [!NOTE]
> Nueva implementación de ADE es compatible con RHEL del sistema operativo y disco de datos para las imágenes de pago por uso de RHEL7. ADE actualmente no se admite para las imágenes de tipo "Trae tu propia suscripción" (BYOS) de RHEL. Consulte también el [Azure Disk Encryption para Linux](azure-security-disk-encryption-linux.md) artículo para obtener más información. __

## <a name="how-can-i-start-using-azure-disk-encryption"></a>¿Cómo puedo empezar a usar Azure Disk Encryption?

Para empezar, lea la [información general sobre Azure Disk Encryption](azure-security-disk-encryption-overview.md).

## <a name="can-i-encrypt-both-boot-and-data-volumes-with-azure-disk-encryption"></a>¿Puedo cifrar los volúmenes de datos y arranque con Azure Disk Encryption?

Sí, puede cifrar los volúmenes de datos y de arranque para las máquinas virtuales IaaS de Windows y Linux. En las máquinas virtuales de Windows, no se pueden cifrar los datos sin cifrar primero el volumen del sistema operativo. En las de Linux, se puede cifrar el volumen de datos sin tener que cifrar primero el volumen del sistema operativo. Una vez que haya cifrado el volumen del sistema operativo para Linux, no se puede deshabilitar el cifrado en un volumen del sistema operativo para las máquinas virtuales IaaS Linux. Para máquinas virtuales Linux en un conjunto de escalado, se puede cifrar solo el volumen de datos.

## <a name="can-i-encrypt-an-unmounted-volume-with-azure-disk-encryption"></a>¿Puedo cifrar un volumen desmontado con Azure Disk Encryption?

No, Azure Disk Encryption solo cifra los volúmenes montados.

## <a name="how-do-i-rotate-secrets-or-encryption-keys"></a>¿Cómo se pueden girar secretos o claves de cifrado?

Para cambiar los secretos, llamaremos el mismo comando que se usaron originalmente para habilitar el cifrado de disco, especifica un almacén de claves diferente. Para rotar la clave de cifrado de claves, llame al mismo comando que se usaron originalmente para habilitar el cifrado de disco, especificando el cifrado de claves nuevo. 

## <a name="how-do-i-add-or-remove-a-key-encryption-key-if-i-didnt-originally-use-one"></a>¿Cómo agregar o quitar una clave de cifrado de claves si originalmente no usar uno?

Para agregar una clave de cifrado de clave, llame al comando enable nuevo pasando el parámetro de clave de cifrado de claves. Para quitar una clave de cifrado de clave, llame al comando enable nuevo sin el parámetro de clave de cifrado de claves.

## <a name="does-azure-disk-encryption-allow-you-to-bring-your-own-key-byok"></a>¿Permite Azure Disk Encryption habilitar la funcionalidad "traiga su propia clave" (BYOK)?

Sí, puede proporcionar sus propias claves de cifrado de claves. Dichas claves están protegidas en Azure Key Vault, que es el almacén de claves de Azure Disk Encryption. Para más información acerca de los escenarios de compatibilidad con clave de cifrado de claves, consulte [Requisitos previos de Azure Disk Encryption](azure-security-disk-encryption-prerequisites.md).

## <a name="can-i-use-an-azure-created-key-encryption-key"></a>¿Puedo usar una clave de cifrado de claves creada en Azure?

Sí, puede usar Azure Key Vault para generar la clave de cifrado de claves para usar en Azure Disk Encryption. Dichas claves están protegidas en Azure Key Vault, que es el almacén de claves de Azure Disk Encryption. Para más información acerca de las claves de cifrado de claves, consulte [Requisitos previos de Azure Disk Encryption](azure-security-disk-encryption-prerequisites.md).

## <a name="can-i-use-an-on-premises-key-management-service-or-hsm-to-safeguard-the-encryption-keys"></a>¿Puedo usar el servicio de administración de claves local o HSM para proteger las claves de cifrado?

No puede utilizar el servicio de administración de claves local ni HSM para proteger las claves de cifrado con Azure Disk Encryption. Para proteger las claves de cifrado, solo se puede utilizar el servicio Azure Key Vault. Para más información acerca de los escenarios de compatibilidad con claves de cifrado de claves, consulte [Requisitos previos de Azure Disk Encryption](azure-security-disk-encryption-prerequisites.md).

## <a name="what-are-the-prerequisites-to-configure-azure-disk-encryption"></a>¿Cuáles son los requisitos previos para configurar Azure Disk Encryption?

Hay requisitos previos para Azure Disk Encryption. Consulte el artículo [Requisitos previos de Azure Disk Encryption](azure-security-disk-encryption-prerequisites.md) para crear un nuevo almacén de claves o configurar el existente para el acceso al cifrado de disco y habilitar el cifrado y proteger los secretos y las claves. Para más información acerca de los escenarios de compatibilidad con claves de cifrado de claves, consulte la [información general sobre Azure Disk Encryption](azure-security-disk-encryption-overview.md).

## <a name="what-are-the-prerequisites-to-configure-azure-disk-encryption-with-an-azure-ad-app-previous-release"></a>¿Cuáles son los requisitos previos para configurar Azure Disk Encryption con una aplicación de Azure AD (versión anterior)?

Hay requisitos previos para Azure Disk Encryption. Consulte el artículo [Requisitos previos de Azure Disk Encryption](azure-security-disk-encryption-prerequisites-aad.md) para crear una aplicación de Azure Active Directory, crear un nuevo almacén de claves o configurar el existente para el acceso al cifrado de disco y habilitar el cifrado y proteger los secretos y las claves. Para más información acerca de los escenarios de compatibilidad con claves de cifrado de claves, consulte la [información general sobre Azure Disk Encryption](azure-security-disk-encryption-overview.md).

## <a name="is-azure-disk-encryption-using-an-azure-ad-app-previous-release-still-supported"></a>¿Sigue siendo compatible Azure Disk Encryption con una aplicación de Azure AD (versión anterior)?
Sí. Todavía se admite el cifrado de disco mediante una aplicación de Azure AD. Sin embargo, al cifrar nuevas máquinas virtuales, se recomienda utilizar el nuevo método en lugar de cifrar con una aplicación de Azure AD. 

## <a name="can-i-migrate-vms-that-were-encrypted-with-an-azure-ad-app-to-encryption-without-an-azure-ad-app"></a>¿Se pueden migrar las máquinas virtuales cifradas con una aplicación de Azure AD para el cifrado sin una aplicación de Azure AD?
  En la actualidad, no existe una ruta de migración directa para los equipos que se cifraron con una aplicación de Azure AD al cifrado sin una aplicación de Azure AD. Además, tampoco hay una ruta directa desde el cifrado sin una aplicación Azure AD al cifrado con una aplicación de AD. 

## <a name="what-version-of-azure-powershell-does-azure-disk-encryption-support"></a>¿Qué versión de Azure PowerShell es compatible con Azure Disk Encryption?

Utilice la versión más reciente del SDK de Azure PowerShell para configurar Azure Disk Encryption. Descargue la versión más reciente de [Azure PowerShell](https://github.com/Azure/azure-powershell/releases). La versión 1.1.0 del SDK de Azure *no* admite Azure Disk Encryption.

> [!NOTE]
> La extensión de la versión preliminar de Azure Disk Encryption para Linux está en desuso. Para obtener más información, consulte [Desuso de la extensión de vista previa de Azure Disk Encryption para máquinas virtuales de IaaS de Linux](https://blogs.msdn.microsoft.com/azuresecurity/2017/07/12/deprecating-azure-disk-encryption-preview-extension-for-linux-iaas-vms/).

## <a name="can-i-apply-azure-disk-encryption-on-my-custom-linux-image"></a>¿Puedo aplicar Azure Disk Encryption en mi imagen personalizada de Linux?

No puede aplicar Azure Disk Encryption a una imagen personalizada de Linux. Solo se admiten las imágenes de Linux de la galería para las distribuciones admitidas mencionadas anteriormente. Actualmente no se admiten imágenes personalizadas de Linux.

## <a name="can-i-apply-updates-to-a-linux-red-hat-vm-that-uses-the-yum-update"></a>¿Puedo aplicar actualizaciones a una máquina virtual Red Hat de Linux utilizando la actualización de Yum?

Sí, puede realizar una actualización de yum en una máquina virtual de Red Hat Linux.  Para obtener más información, consulte [administración de paquetes de Linux detrás de un firewall](azure-security-disk-encryption-tsg.md#linux-package-management-behind-a-firewall).

## <a name="what-is-the-recommended-azure-disk-encryption-workflow-for-linux"></a>¿Cuál es el flujo de trabajo de Azure Disk Encryption recomendado para Linux?

Para obtener los mejores resultados en Linux, se recomienda el siguiente flujo de trabajo:
* Comenzar en la imagen de la galería en existencias sin modificar correspondiente a la distribución y versión necesarias del sistema operativo
* Realizar una copia de seguridad de las unidades montadas que se van a cifrar.  Esta copia de seguridad permite la recuperación en caso de error, por ejemplo, si la máquina virtual se reinicia antes de que finalice el cifrado.
* Cifrar (puede tardar varias horas o incluso días según las características de la máquina virtual y el tamaño de los discos de datos adjuntos)
* Personalizar y agregar software a la imagen según sea necesario.

Si este flujo de trabajo no es posible, el uso de [Storage Service Encryption](../storage/common/storage-service-encryption.md) (SSE) en el nivel de la cuenta de almacenamiento de la plataforma puede ser una alternativa al cifrado del disco completo mediante dm-crypt.

## <a name="what-is-the-disk-bek-volume-or-mntazurebekdisk"></a>¿Qué es el disco "volumen Bek" o "/mnt/azure_bek_disk"?

"Volumen Bek" para Windows o "/mnt/azure_bek_disk" para Linux es un volumen de datos locales que almacena de forma segura las claves de cifrado para máquinas virtuales cifradas de IaaS de Azure.
> [!NOTE]
> No elimine ni modifique ningún contenido de este disco. No desmonte el disco ya que la presencia de la clave de cifrado es necesaria para las operaciones de cifrado en la máquina virtual IaaS.


## <a name="what-encryption-method-does-azure-disk-encryption-use"></a>¿Qué método de cifrado usa Azure Disk Encryption?

En Windows, ADE usa el método de cifrado BitLocker AES256 (AES256WithDiffuser en versiones anteriores a Windows Server 2012). En Linux, ADE usa el valor predeterminado de descifrado de xts-aes-plain64 con una clave maestra de volumen de 256 bits.

## <a name="if-i-use-encryptformatall-and-specify-all-volume-types-will-it-erase-the-data-on-the-data-drives-that-we-already-encrypted"></a>¿Si uso EncryptFormatAll y especifico todos los tipos de volumen se borrarán los datos en las unidades de datos que ya hemos cifrado?
No, no se borran datos de las unidades de datos que ya estén cifradas mediante Azure Disk Encryption. Igual que EncryptFormatAll no volvía a cifrar la unidad del SO, no volverá a cifrar la unidad de datos ya cifrada. Para obtener más información, consulte [Criterios de EncryptFormatAll](azure-security-disk-encryption-linux.md#bkmk_EFACriteria).        

## <a name="is-xfs-filesystem-supported"></a>¿Se admite el sistema de archivos XFS?
Se admiten volúmenes XFS para el cifrado de disco de datos solo con el EncryptFormatAll. Esto volverá a formatear el volumen, borrar todos los datos previamente no existe. Para obtener más información, consulte [Criterios de EncryptFormatAll](azure-security-disk-encryption-linux.md#bkmk_EFACriteria).

## <a name="can-i-backup-and-restore-an-encrypted-vm"></a>¿Puedo de copia de seguridad y restaurar una máquina virtual cifrada? 

Azure Backup proporciona un mecanismo para copias de seguridad y restauración VM cifrada dentro de la misma suscripción y región.  Para obtener instrucciones, consulte [copia de seguridad y restauración de máquinas virtuales cifradas con Azure Backup](https://docs.microsoft.com/azure/backup/backup-azure-vms-encryption).  No se admite actualmente la restauración de una máquina virtual cifrada en una región diferente.  

## <a name="where-can-i-go-to-ask-questions-or-provide-feedback"></a>¿Dónde puedo formular preguntas o enviar comentarios?

Puede realizar preguntas o publicar comentarios en el [Foro de Azure Disk Encryption](https://social.msdn.microsoft.com/Forums/en-US/home?forum=AzureDiskEncryption).

## <a name="next-steps"></a>Pasos siguientes
En este documento, aprendió más acerca de las preguntas más frecuentes sobre Azure Disk Encryption. Para obtener más información acerca de este servicio, vea los siguientes artículos:

- [Introducción a Azure Disk Encryption](azure-security-disk-encryption-overview.md)
- [Aplicación de cifrado de discos en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-apply-disk-encryption)
- [Cifrado de datos en reposo de Azure](https://docs.microsoft.com/azure/security/azure-security-encryption-atrest)
