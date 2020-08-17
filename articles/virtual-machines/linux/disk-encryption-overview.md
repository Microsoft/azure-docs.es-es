---
title: Habilitación de Azure Disk Encryption para VM Linux
description: En este artículo se proporcionan instrucciones sobre cómo habilitar Microsoft Azure Disk Encryption para VM Linux.
author: msmbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: conceptual
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18
ms.openlocfilehash: 405ebbbfa4a662dd9ee3c8d10dde8f28e5ce9c66
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87830451"
---
# <a name="azure-disk-encryption-for-linux-vms"></a>Azure Disk Encryption para VM Linux 

Azure Disk Encryption ayuda a custodiar y proteger sus datos con el fin de satisfacer los compromisos de cumplimiento y seguridad de su organización. Usa la característica [DM-Crypt](https://en.wikipedia.org/wiki/Dm-crypt) de Linux para proporcionar cifrado de volumen tanto a los discos de datos como a los del sistema operativo de máquinas virtuales (VM) de Azure y se integra con [Azure Key Vault](../../key-vault/index.yml) para ayudarle a controlar y administrar las claves y los secretos del cifrado de disco. 

Si utiliza [Azure Security Center](../../security-center/index.yml), se le alertará si tiene VM que no estén cifradas. Estas alertas se muestran con gravedad alta y se recomienda cifrar estas máquinas virtuales.

![Alerta de cifrado de discos en Azure Security Center](media/disk-encryption/security-center-disk-encryption-fig1.png)

> [!WARNING]
> - Si ya ha usado Azure Disk Encryption con Azure AD para cifrar una VM, debe seguir usando esta opción para cifrar la VM. Para más información, consulte [Azure Disk Encryption con Azure AD (versión anterior)](disk-encryption-overview-aad.md). 
> - Algunas de las recomendaciones pueden provocar un aumento del uso de datos, de la red o de recursos de proceso, lo que incrementará los costes de las licencias o suscripciones. Para crear recursos en Azure en las regiones admitidas, debe tener una suscripción válida de Azure activa.
> - Actualmente, las máquinas virtuales de generación 2 no admiten Azure Disk Encryption. Vea [Compatibilidad para máquinas virtuales de generación 2 en Azure](../windows/generation-2.md) para obtener más información.

Para obtener información sobre los aspectos básicos de Azure Disk Encryption para Linux en unos minutos, consulte [Inicio rápido: Creación y cifrado de una VM Linux con la CLI de Azure](disk-encryption-cli-quickstart.md) o [Inicio rápido: Creación y cifrado de una VM Linux con Azure PowerShell](disk-encryption-powershell-quickstart.md).

## <a name="supported-vms-and-operating-systems"></a>VM y sistemas operativos compatibles

### <a name="supported-vms"></a>VM admitidas

Las VM Linux están disponibles en una [variedad de tamaños](../sizes.md). Azure Disk Encryption no está disponible en [VM básicas o de serie A](https://azure.microsoft.com/pricing/details/virtual-machines/series/) ni en las máquinas virtuales que no cumplen estos requisitos mínimos de memoria:

| Máquina virtual | Requisito mínimo de memoria |
|--|--|
| VM de Linux cuando solo se cifran volúmenes de datos| 2 GB |
| VM de Linux cuando se cifran volúmenes de datos y del SO, y donde el uso del sistema de archivos raíz (/) es 4 GB o menos | 8 GB |
| VM de Linux cuando se cifran volúmenes de datos y del SO, y donde el uso del sistema de archivos raíz (/) es mayor que 4 GB | El uso del sistema de archivos raíz * 2. Por ejemplo, un uso de 16 GB del sistema de archivos raíz requiere al menos 32 GB de RAM |

Una vez completado el proceso de cifrado de disco del sistema operativo en las máquinas virtuales de Linux, la VM se puede configurar para que se ejecute con menos memoria. 

Azure Disk Encryption también está disponible para las VM con almacenamiento Premium.

Azure Disk Encryption no está disponible en [máquinas virtuales de segunda generación](generation-2.md#generation-1-vs-generation-2-capabilities) ni en las [máquinas virtuales de la serie Lsv2](../lsv2-series.md). Para ver más excepciones, consulte [Azure Disk Encryption: escenarios no admitidos](disk-encryption-linux.md#unsupported-scenarios).

### <a name="supported-operating-systems"></a>Sistemas operativos admitidos

Azure Disk Encryption se admite en un subconjunto de las [distribuciones de Linux aprobadas por Azure](endorsed-distros.md), que es en sí mismo un subconjunto de todas las posibles distribuciones de servidores Linux.

![Diagrama de Venn de distribuciones de servidores Linux que admiten Azure Disk Encryption](./media/disk-encryption/ade-supported-distros.png)

Las distribuciones de servidores Linux que no están aprobadas por Azure no admiten Azure Disk Encryption; de las que están aprobadas, solo las siguientes distribuciones y versiones admiten Azure Disk Encryption:

| Publicador | Oferta | SKU | URN | Tipo de volumen admitido para el cifrado |
| --- | --- |--- | --- |
| Canonical | Ubuntu | 18.04-LTS | Canonical:UbuntuServer:18.04-LTS:latest | Sistema operativo y disco de datos |
| Canonical | Ubuntu 18.04 | 18.04-DAILY-LTS | Canonical:UbuntuServer:18.04-DAILY-LTS:latest | Sistema operativo y disco de datos |
| Canonical | Ubuntu 16.04 | 16.04-DAILY-LTS | Canonical:UbuntuServer:16.04-DAILY-LTS:latest | Sistema operativo y disco de datos |
| Canonical | Ubuntu 14.04.5</br>[con kernel optimizado para Azure 4.15 o posterior](disk-encryption-troubleshooting.md) | 14.04.5-LTS | Canonical:UbuntuServer:14.04.5-LTS:latest | Sistema operativo y disco de datos |
| Canonical | Ubuntu 14.04.5</br>[con kernel optimizado para Azure 4.15 o posterior](disk-encryption-troubleshooting.md) | 14.04.5-DAILY-LTS | Canonical:UbuntuServer:14.04.5-DAILY-LTS:latest | Sistema operativo y disco de datos |
| RedHat | RHEL 7.8 | 7.8 | RedHat:RHEL:7.8:latest | Sistema operativo y disco de datos (véase la nota siguiente) |
| RedHat | RHEL 7.7 | 7,7 | RedHat:RHEL:7.7:latest | Sistema operativo y disco de datos (véase la nota siguiente) |
| RedHat | RHEL 7.7 | 7-LVM | RedHat:RHEL:7-LVM:latest | Sistema operativo y disco de datos (véase la nota siguiente) |
| RedHat | RHEL 7.6 | 7.6 | RedHat:RHEL:7.6:latest | Sistema operativo y disco de datos (véase la nota siguiente) |
| RedHat | RHEL 7.5 | 7.5 | RedHat:RHEL:7.5:latest | Sistema operativo y disco de datos (véase la nota siguiente) |
| RedHat | RHEL 7.4 | 7.4 | RedHat:RHEL:7.4:latest | Sistema operativo y disco de datos (véase la nota siguiente) |
| RedHat | RHEL 7.3 | 7.3 | RedHat:RHEL:7.3:latest | Sistema operativo y disco de datos (véase la nota siguiente) |
| RedHat | RHEL 7.2 | 7.2 | RedHat:RHEL:7.2:latest | Sistema operativo y disco de datos (véase la nota siguiente) |
| RedHat | RHEL 6.8 | 6,8 | RedHat:RHEL:6.8:latest | Disco de datos (véase la nota siguiente) |
| RedHat | RHEL 6.7 | 6.7 | RedHat:RHEL:6.7:latest | Disco de datos (véase la nota siguiente) |
| OpenLogic | CentOS 7.7 | 7,7 | OpenLogic:CentOS:7.7:latest | Sistema operativo y disco de datos |
| OpenLogic | CentOS 7.7 | 7-LVM | OpenLogic:CentOS:7-LVM:latest | Sistema operativo y disco de datos |
| OpenLogic | CentOS 7.6 | 7.6 | OpenLogic:CentOS:7.6:latest | Sistema operativo y disco de datos |
| OpenLogic | CentOS 7.5 | 7.5 | OpenLogic:CentOS:7.5:latest | Sistema operativo y disco de datos |
| OpenLogic | CentOS 7.4 | 7.4 | OpenLogic:CentOS:7.4:latest | Sistema operativo y disco de datos |
| OpenLogic | CentOS 7.3 | 7.3 | OpenLogic:CentOS:7.3:latest | Sistema operativo y disco de datos |
| OpenLogic | CentOS 7.2n | 7.2n | OpenLogic:CentOS:7.2n:latest | Sistema operativo y disco de datos |
| OpenLogic | CentOS 7.1 | 7.1 | OpenLogic:CentOS:7.1:latest | Solo el disco de datos |
| OpenLogic | CentOS 7.0 | 7.0 | OpenLogic:CentOS:7.0:latest | Solo el disco de datos |
| OpenLogic | CentOS 6.8 | 6,8 | OpenLogic:CentOS:6.8:latest | Solo el disco de datos |
| SUSE | openSUSE 42.3 | 42.3 | SUSE:openSUSE-Leap:42.3:latest | Solo el disco de datos |
| SUSE | SLES 12-SP4 | 12-SP4 | SUSE:SLES:12-SP4:latest | Solo el disco de datos |
| SUSE | SLES HPC 12-SP3 | 12-SP3 | SUSE:SLES-HPC:12-SP3:latest | Solo el disco de datos |

> [!NOTE]
> La nueva implementación de Azure Disk Encryption se admite para el sistema operativo RHEL y el disco de datos para las imágenes de Pago por uso de RHEL7.  
>
> ADE también es compatible con las imágenes Gold de tipo Traiga su propia suscripción de RHEL, pero solo **después** de registrar la suscripción. Para más información, consulte [Imágenes Gold de tipo Traiga su propia suscripción (BYOS) de Red Hat Enterprise Linux en Azure](../workloads/redhat/byos.md#encrypt-red-hat-enterprise-linux-bring-your-own-subscription-gold-images).

## <a name="additional-vm-requirements"></a>Requisitos adicionales de VM

Azure Disk Encryption requiere que los módulos dm-crypt y vfat estén presentes en el sistema. Quitar o deshabilitar vfat desde la imagen predeterminada impedirá que el sistema lea el volumen clave y obtenga la clave necesaria para desbloquear los discos en los reinicios posteriores. Los pasos de protección del sistema que quitan el módulo vfat del sistema o aplican la expansión de las carpetas montaje del sistema operativo en las unidades de datos no son compatibles con Azure Disk Encryption. 

Antes de habilitar el cifrado, los discos de datos que se van a cifrar deben aparecer correctamente en /etc/fstab. Use la opción "nofail" al crear entradas y elija un nombre de dispositivo de bloque persistente (dado que los nombres de dispositivo en el formato "/dev/sdX" podrían no asociarse al mismo disco en los diferentes reinicios, especialmente después del cifrado; para obtener más información sobre este comportamiento, vea: [Solución de problemas de cambio de nombre de dispositivo de las máquinas virtuales Linux](../troubleshooting/troubleshoot-device-names-problems.md)).

Asegúrese de que el valor /etc/fstab está configurado correctamente para el montaje. Para configurar estos valores, ejecute el comando mount -a o reinicie la máquina virtual y desencadene el nuevo montaje de ese modo. Cuando haya finalizado, revise la salida del comando lsblk para comprobar que la unidad sigue montada. 

- Si el archivo/etc/fstab no monta la unidad correctamente antes de habilitar el cifrado, Azure Disk Encryption no podrá montarla correctamente.
- El proceso de Azure Disk Encryption sacará la información de montaje de/etc/fstab y la colocará en su propio archivo de configuración como parte del proceso de cifrado. No se alarme si ve que falta la entrada de/etc/fstab después de que se completa el cifrado de la unidad de datos.
- Antes de iniciar el cifrado, asegúrese de detener todos los servicios y procesos que se podrían estar escribiendo en los discos de datos montados y deshabilítelos de modo que no se reinicien automáticamente tras un reinicio. Estos podrían mantener los archivos abiertos en estas particiones, lo que impediría que el procedimiento de cifrado los vuelva a montar, provocando un error de cifrado. 
- Después del reinicio, el proceso de Azure Disk Encryption tardará un tiempo en montar los discos recién cifrados. No estarán disponibles inmediatamente después de un reinicio. El proceso necesita tiempo para iniciar, desbloquear y montar las unidades cifradas antes de que estén disponibles para que otros procesos tengan acceso a ellas. Este proceso puede tardar más de un minuto después del reinicio, dependiendo de las características del sistema.

Este es un ejemplo de los comandos que se usan para montar los discos de datos y crear las entradas de /etc/fstab necesarias:

```bash
UUID0="$(blkid -s UUID -o value /dev/disk/azure/scsi1/lun0)"
UUID1="$(blkid -s UUID -o value /dev/disk/azure/scsi1/lun1)"
mkdir /data0
mkdir /data1
echo "UUID=$UUID0 /data0 ext4 defaults,nofail 0 0" >>/etc/fstab
echo "UUID=$UUID1 /data1 ext4 defaults,nofail 0 0" >>/etc/fstab
mount -a
```
## <a name="networking-requirements"></a>Requisitos de red

Para habilitar la característica Azure Disk Encryption, las VM Linux deben cumplir los siguientes requisitos de configuración de puntos de conexión de red:
  - Para que un token se conecte al almacén de claves, la VM Linux debe poder conectarse a un punto de conexión de Azure Active Directory, \[login.microsoftonline.com\].
  - Para escribir las claves de cifrado en el almacén de claves, la VM Linux debe poder conectarse al punto de conexión del almacén de claves.
  - La VM Linux debe poder conectarse al punto de conexión de Azure Storage que hospeda el repositorio de extensiones de Azure y la cuenta de Azure Storage que hospeda los archivos VHD.
  -  Si su directiva de seguridad limita el acceso desde máquinas virtuales de Azure a Internet, puede resolver el URI anterior y configurar una regla concreta para permitir la conectividad de salida para las direcciones IP. Para más información, consulte [Azure Key Vault detrás de un firewall](../../key-vault/general/access-behind-firewall.md).  

## <a name="encryption-key-storage-requirements"></a>Requisitos de almacenamiento de la clave de cifrado  

Azure Disk Encryption requiere Azure Key Vault para controlar y administrar las claves y los secretos de cifrado de discos. El almacén de claves y las máquinas virtuales deben residir en la misma región y suscripción de Azure.

Para obtener más información, consulte [Creación y configuración de un almacén de claves para Azure Disk Encryption](disk-encryption-key-vault.md).

## <a name="terminology"></a>Terminología
En la siguiente tabla se definen algunos de los términos comunes que se usan en la documentación de cifrado de disco de Azure:

| Terminología | Definición |
| --- | --- |
| Azure Key Vault | Key Vault es un servicio de administración de claves criptográficas basado en módulos de seguridad de hardware validados por el Estándar federal de procesamiento de información (FIPS). Estos estándares ayudan a proteger las claves criptográficas y los secretos confidenciales. Para obtener más información, consulte la documentación de [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) y [Creación y configuración de un almacén de claves para Azure Disk Encryption](disk-encryption-key-vault.md). |
| Azure CLI | [La CLI de Azure](/cli/azure/install-azure-cli) está optimizada para administrar recursos de Azure desde la línea de comandos.|
| DM-Crypt |[DM-Crypt](https://gitlab.com/cryptsetup/cryptsetup/wikis/DMCrypt) es el subsistema de cifrado transparente de disco basado en Linux que se usa para habilitar el cifrado de disco en las VM Linux. |
| Clave de cifrado de claves (KEK) | La clave asimétrica (RSA 2048) que puede usar para proteger o encapsular el secreto. Puede proporcionar una clave protegida mediante módulos de seguridad de hardware (HSM) o una clave protegida mediante software. Para obtener más información, consulte la documentación de [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) y [Creación y configuración de un almacén de claves para Azure Disk Encryption](disk-encryption-key-vault.md). |
| Cmdlets de PowerShell | Para más información, consulte [Cmdlets de Azure PowerShell](/powershell/azure/). |


## <a name="next-steps"></a>Pasos siguientes

- [Inicio rápido: Creación y cifrado de una VM Linux con la CLI de Azure](disk-encryption-cli-quickstart.md)
- [Inicio rápido: Creación y cifrado de una VM Linux con Azure PowerShell](disk-encryption-powershell-quickstart.md)
- [Escenarios de Azure Disk Encryption en VM Linux](disk-encryption-linux.md)
- [Script de la CLI de requisitos previos de Azure Disk Encryption](https://github.com/ejarvi/ade-cli-getting-started)
- [Script de PowerShell de requisitos previos de Azure Disk Encryption](https://github.com/Azure/azure-powershell/tree/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts)
- [Creación y configuración de un almacén de claves para Azure Disk Encryption](disk-encryption-key-vault.md)