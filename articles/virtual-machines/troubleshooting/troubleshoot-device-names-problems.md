---
title: 'Solución de problemas: se cambian los nombres de dispositivo de máquinas virtuales Linux en Azure | Microsoft Docs'
description: En este artículo se explica por qué se cambian los nombres de los dispositivos de máquinas virtuales Linux y cómo solucionar el problema.
services: virtual-machines-linux
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: ''
ms.service: virtual-machines-linux
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.date: 11/01/2018
ms.author: genli
ms.openlocfilehash: 31f64a504156134b1d622705d5301d9cd5a5f5b1
ms.sourcegitcommit: afa1411c3fb2084cccc4262860aab4f0b5c994ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/23/2020
ms.locfileid: "88756833"
---
# <a name="troubleshoot-linux-vm-device-name-changes"></a>Solución de problemas: se cambian los nombres de dispositivo de máquinas virtuales Linux

En este artículo se explica por qué se cambian los nombres de dispositivos después de reiniciar una máquina virtual Linux o de volver a adjuntar los discos de datos. En este artículo también se brindan soluciones para este problema.

## <a name="symptoms"></a>Síntomas
Es posible que experimente los siguientes problemas al ejecutar máquinas virtuales Linux en Microsoft Azure:

- La máquina virtual no puede arrancar después de un reinicio.
- Cuando los discos de datos se desasocian y se vuelven a adjuntar, se cambian los nombres de los dispositivos de los discos.
- Se produce un error en una aplicación o un script que hace referencia a un disco con el nombre de dispositivo porque se cambió el nombre de dispositivo.

## <a name="cause"></a>Causa

No se garantiza que las rutas de acceso de dispositivo en Linux sean coherentes entre reinicios. Los nombres de dispositivo constan de números principales (letras) y secundarios. Cuando el controlador del dispositivo de almacenamiento de Linux detecta un dispositivo nuevo, le asigna números de dispositivo principales y secundarios del intervalo disponible al dispositivo. Cuando se quita un dispositivo, los números de dispositivo se liberan para reutilizarlos.

El problema se produce porque el análisis de dispositivos en Linux lo programa el subsistema de SCSI para que se realice de forma asincrónica. Como resultado, un nombre de ruta de acceso de dispositivo puede variar entre los distintos reinicios.

## <a name="solution"></a>Solución

Para resolver este problema, use nombres persistentes. Hay cuatro maneras de usar la nomenclatura persistente: por etiqueta de sistema de archivos, por UUID, por id. o por ruta de acceso. Se recomienda usar la etiqueta de sistema de archivos o UUID para las máquinas virtuales Linux de Azure.

La mayoría de las distribuciones proporciona los parámetros `fstab` **nofail** o **nobootwait**. Estos parámetros permiten que un sistema arranque cuando el disco no se monta en el inicio. Consulte la documentación de la distribución para más información sobre estos parámetros. Para información sobre cómo configurar una máquina virtual Linux para que use un UUID cuando se agrega un disco de datos, consulte [Conexión a la máquina virtual Linux para montar el nuevo disco](../linux/add-disk.md#format-and-mount-the-disk).

Cuando se instala el agente Linux de Azure en una máquina virtual, el agente usa reglas de Udev para crear un conjunto de vínculos simbólicos en la ruta de acceso /dev/disk/azure path. Las aplicaciones y los scripts usan reglas de Udev para identificar los discos conectados a la máquina virtual, junto con el tipo de disco y los LUN de disco.

Si ya ha editado fstab de tal manera que la máquina virtual no arranca y no puede conectarse mediante SSH a ella, puede utilizar la [consola serie de máquinas virtuales](./serial-console-linux.md) para especificar el [modo de usuario único](./serial-console-grub-single-user-mode.md) y modificar fstab.

### <a name="identify-disk-luns"></a>Identificar LUN de disco

Las aplicaciones usan LUN para encontrar todos los discos conectados y construir los vínculos simbólicos. El agente Linux de Azure incluye reglas de Udev que configuran vínculos simbólicos desde un LUN a los dispositivos:

```console
$ tree /dev/disk/azure

/dev/disk/azure
├── resource -> ../../sdb
├── resource-part1 -> ../../sdb1
├── root -> ../../sda
├── root-part1 -> ../../sda1
└── scsi1
    ├── lun0 -> ../../../sdc
    ├── lun0-part1 -> ../../../sdc1
    ├── lun1 -> ../../../sdd
    ├── lun1-part1 -> ../../../sdd1
    ├── lun1-part2 -> ../../../sdd2
    └── lun1-part3 -> ../../../sdd3
```

La información de LUN desde la cuenta de invitado de Linux se recupera con `lsscsi` o una herramienta similar:

```console
$ sudo lsscsi

[1:0:0:0] cd/dvd Msft Virtual CD/ROM 1.0 /dev/sr0

[2:0:0:0] disk Msft Virtual Disk 1.0 /dev/sda

[3:0:1:0] disk Msft Virtual Disk 1.0 /dev/sdb

[5:0:0:0] disk Msft Virtual Disk 1.0 /dev/sdc

[5:0:0:1] disk Msft Virtual Disk 1.0 /dev/sdd
```

La información de LUN de invitado se usa con los metadatos de suscripción de Azure para buscar el VHD en Azure Storage que contiene los datos de partición. Por ejemplo, puede usar la CLI `az`:

```azurecli
$ az vm show --resource-group testVM --name testVM | jq -r .storageProfile.dataDisks
[
{
"caching": "None",
  "createOption": "empty",
"diskSizeGb": 1023,
  "image": null,
"lun": 0,
"managedDisk": null,
"name": "testVM-20170619-114353",
"vhd": {
  "uri": "https://testVM.blob.core.windows.net/vhd/testVM-20170619-114353.vhd"
}
},
{
"caching": "None",
"createOption": "empty",
"diskSizeGb": 512,
"image": null,
"lun": 1,
"managedDisk": null,
"name": "testVM-20170619-121516",
"vhd": {
  "uri": "https://testVM.blob.core.windows.net/vhd/testVM-20170619-121516.vhd"
  }
  }
]
```

### <a name="discover-filesystem-uuids-by-using-blkid"></a>Detectar UUID del sistema de archivos mediante blkid

Las aplicaciones y los scripts leen la salida de `blkid`, u orígenes de información similares, para construir vínculos simbólicos en la ruta de acceso in the /dev. La salida muestra los UUID de todos los discos que están conectados a la máquina virtual y su archivo de dispositivo asociado:

```console
$ sudo blkid -s UUID

/dev/sr0: UUID="120B021372645f72"
/dev/sda1: UUID="52c6959b-79b0-4bdd-8ed6-71e0ba782fb4"
/dev/sdb1: UUID="176250df-9c7c-436f-94e4-d13f9bdea744"
/dev/sdc1: UUID="b0048738-4ecc-4837-9793-49ce296d2692"
```

Las reglas de Udev del agente Linux de Azure construyen un conjunto de vínculos simbólicos en la ruta de acceso /dev/disk/azure:

```console
$ ls -l /dev/disk/azure

total 0
lrwxrwxrwx 1 root root  9 Jun  2 23:17 resource -> ../../sdb
lrwxrwxrwx 1 root root 10 Jun  2 23:17 resource-part1 -> ../../sdb1
lrwxrwxrwx 1 root root  9 Jun  2 23:17 root -> ../../sda
lrwxrwxrwx 1 root root 10 Jun  2 23:17 root-part1 -> ../../sda1
```

Las aplicaciones usan los vínculos para identificar el dispositivo del disco de arranque y el disco de recursos (efímero). En Azure, las aplicaciones deben hacer referencia a las rutas de acceso /dev/disk/azure/root-part1 o /dev/disk/azure-resource-part1 para detectar estas particiones.

Toda partición adicional de la lista `blkid` reside en un disco de datos. Las aplicaciones mantienen el UUID de estas particiones y usan una ruta de acceso para detectar el nombre de dispositivo en el entorno de tiempo de ejecución:

```console
$ ls -l /dev/disk/by-uuid/b0048738-4ecc-4837-9793-49ce296d2692

lrwxrwxrwx 1 root root 10 Jun 19 15:57 /dev/disk/by-uuid/b0048738-4ecc-4837-9793-49ce296d2692 -> ../../sdc1
```


### <a name="get-the-latest-azure-storage-rules"></a>Obtención de las reglas de Azure Storage más recientes

Ejecute los comandos siguientes para obtener las reglas de Azure Storage más recientes:

```console
# sudo curl -o /etc/udev/rules.d/66-azure-storage.rules https://raw.githubusercontent.com/Azure/WALinuxAgent/master/config/66-azure-storage.rules
# sudo udevadm trigger --subsystem-match=block
```

## <a name="see-also"></a>Consulte también

Para más información, consulte los siguientes artículos.

- [Ubuntu: Using UUID](https://help.ubuntu.com/community/UsingUUID) (Ubuntu: Uso de UUID)
- [RedHat: Persistent Naming](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Storage_Administration_Guide/persistent_naming.html) (RedHat: Nomenclatura persistente)
- [Linux: What UUIDs can do for you](https://www.linux.com/news/what-uuids-can-do-you) (Linux: ¿Qué pueden hacer los UUID por usted?)
- [Udev: Introduction to Device Management In Modern Linux System](https://www.linux.com/news/udev-introduction-device-management-modern-linux-system) (UDev: Introducción a la administración de dispositivos en sistemas Linux modernos)

