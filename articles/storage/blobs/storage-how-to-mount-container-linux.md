---
title: Cómo montar Azure Blob Storage como sistema de archivos en Linux | Microsoft Docs
description: Aprenda a montar un contenedor de Azure Blob Storage con blobfuse, un controlador del sistema de archivos virtual en Linux.
author: rishabpoh
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.date: 2/1/2019
ms.author: ripohane
ms.reviewer: dineshm
ms.openlocfilehash: 297595c6c4a9c82c3d0293f2cea2db66ea9ca54a
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89180412"
---
# <a name="how-to-mount-blob-storage-as-a-file-system-with-blobfuse"></a>Cómo montar el almacenamiento de blobs como sistema de archivos con blobfuse

## <a name="overview"></a>Información general
[Blobfuse](https://github.com/Azure/azure-storage-fuse) es un controlador de sistema de archivos virtual para Azure Blob Storage. Blobfuse le permite obtener acceso a los datos de blob en bloques existentes en la cuenta de almacenamiento, a través del sistema de archivos de Linux. Blobfuse usa el esquema de directorio virtual con el uso de la barra oblicua "/" como delimitador.  

En esta guía, se muestra cómo usar blobfuse, montar un contenedor de almacenamiento de blobs en Linux y obtener acceso a los datos. Para obtener más información sobre blobfuse, lea los detalles en [el repositorio de blobfuse](https://github.com/Azure/azure-storage-fuse).

> [!WARNING]
> Blobfuse no garantiza el cumplimiento al 100 % con POSIX ya que, simplemente, convierte las solicitudes en [API REST de Blob](https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api). Por ejemplo, las operaciones de cambio de nombre son atómicas en POSIX, pero no en blobfuse.
> Para obtener una lista completa de las diferencias entre un sistema de archivos nativo y blobfuse, visite [el repositorio de código fuente de blobfuse](https://github.com/azure/azure-storage-fuse).
> 

## <a name="install-blobfuse-on-linux"></a>Instalar blobfuse en Linux
Los archivos binarios de Blobfuse están disponibles en [los repositorios de software de Microsoft para Linux](https://docs.microsoft.com/windows-server/administration/Linux-Package-Repository-for-Microsoft-Software) correspondientes a las distribuciones de Ubuntu y RHEL. Para instalar Blobfuse en estas distribuciones, configure uno de los repositorios de la lista. Si no hay ningún archivo binario disponible para su distribución, puede compilar archivos binarios a partir del código fuente siguiendo los [pasos de instalación que se detallan en Azure Storage](https://github.com/Azure/azure-storage-fuse/wiki/1.-Installation#option-2---build-from-source).

Blobfuse admite la instalación en Ubuntu 14.04, 16.04 y 18.04. Ejecute este comando para asegurarse de que tiene una de estas versiones implementadas:
```
lsb_release -a
```

### <a name="configure-the-microsoft-package-repository"></a>Configurar el repositorio de paquetes de Microsoft
Configure el [repositorio de paquetes de Linux para productos de Microsoft](https://docs.microsoft.com/windows-server/administration/Linux-Package-Repository-for-Microsoft-Software).

Por ejemplo, en una distribución Enterprise Linux 6:
```bash
sudo rpm -Uvh https://packages.microsoft.com/config/rhel/6/packages-microsoft-prod.rpm
```

Del mismo modo, cambie la dirección URL a `.../rhel/7/...` para que señale a una distribución Enterprise Linux 7.

Otro ejemplo en una distribución Ubuntu 14.04:
```bash
wget https://packages.microsoft.com/config/ubuntu/14.04/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
```

Del mismo modo, cambie la dirección URL a `.../ubuntu/16.04/...` o `.../ubuntu/18.04/...` para hacer referencia a otra versión de Ubuntu.

### <a name="install-blobfuse"></a>Instalar blobfuse

En una distribución Ubuntu/Debian:
```bash
sudo apt-get install blobfuse
```

En una distribución Enterprise Linux:
```bash    
sudo yum install blobfuse
```

## <a name="prepare-for-mounting"></a>Preparación del montaje
Blobfuse requiere una ruta de acceso temporal en el sistema de archivos para almacenar en búfer y en caché los archivos abiertos, lo que ayuda a proporcionar un rendimiento similar al nativo. Para esta ruta de acceso temporal, elija el disco con mayor rendimiento o use un disco RAM para obtener el rendimiento óptimo. 

> [!NOTE]
> Blobfuse almacena todo el contenido de los archivos abiertos en la ruta de acceso temporal. Asegúrese de que tiene espacio suficiente para dar cabida a todos los archivos abiertos. 
> 

### <a name="optional-use-a-ramdisk-for-the-temporary-path"></a>(Opcional) Usar un disco RAM para la ruta de acceso temporal
En el ejemplo siguiente, se crea un disco RAM de 16 GB, así como un directorio para blobfuse. Elija el tamaño según sus necesidades. Este disco RAM permite que blobfuse abra archivos de hasta 16 GB de tamaño. 
```bash
sudo mkdir /mnt/ramdisk
sudo mount -t tmpfs -o size=16g tmpfs /mnt/ramdisk
sudo mkdir /mnt/ramdisk/blobfusetmp
sudo chown <youruser> /mnt/ramdisk/blobfusetmp
```

### <a name="use-an-ssd-as-a-temporary-path"></a>Usar una SSD para la ruta de acceso temporal
En Azure, puede usar los discos efímeros (SSD) disponibles en las máquinas virtuales para proporcionar un búfer de baja latencia para blobfuse. En las distribuciones de Ubuntu, este disco efímero se monta en "/mnt". En las distribuciones de RedHat y CentOS, el disco está montado en "/mnt/resource/".

Asegúrese de que el usuario obtenga acceso a la ruta de acceso temporal:
```bash
sudo mkdir /mnt/resource/blobfusetmp -p
sudo chown <youruser> /mnt/resource/blobfusetmp
```

### <a name="configure-your-storage-account-credentials"></a>Configurar las credenciales de la cuenta de almacenamiento
Blobfuse requiere que sus credenciales se almacenen en un archivo de texto con el formato siguiente: 

```
accountName myaccount
accountKey storageaccesskey
containerName mycontainer
```
`accountName` es el prefijo de la cuenta de almacenamiento (no la dirección URL completa).

Cree este archivo mediante:

```
touch ~/fuse_connection.cfg
```

Después de crear y editar este archivo, asegúrese de restringir el acceso para que ningún otro usuario pueda leerlo.
```bash
chmod 600 fuse_connection.cfg
```

> [!NOTE]
> Si ha creado el archivo de configuración en Windows, asegúrese de ejecutar `dos2unix` para corregirlo y convertirlo en formato Unix. 
>

### <a name="create-an-empty-directory-for-mounting"></a>Crear un directorio vacío para el montaje
```bash
mkdir ~/mycontainer
```

## <a name="mount"></a>Montaje

> [!NOTE]
> Para obtener una lista completa de opciones de montaje, consulte [el repositorio de blobfuse](https://github.com/Azure/azure-storage-fuse#mount-options).  
> 

Para montar blobfuse, ejecute el siguiente comando con su usuario. Este comando monta el contenedor especificado en "/path/to/fuse_connection.cfg" en la ubicación "/mycontainer".

```bash
sudo blobfuse ~/mycontainer --tmp-path=/mnt/resource/blobfusetmp  --config-file=/path/to/fuse_connection.cfg -o attr_timeout=240 -o entry_timeout=240 -o negative_timeout=120
```

Ahora debería tener acceso a los blobs en bloque a través de las API normales del sistema de archivos. El usuario que monta el directorio es la única persona que puede obtener acceso al mismo de forma predeterminada, lo que protege el acceso. Para permitir el acceso a todos los usuarios, se puede montar con la opción ```-o allow_other```. 

```bash
cd ~/mycontainer
mkdir test
echo "hello world" > test/blob.txt
```

## <a name="next-steps"></a>Pasos siguientes

* [Página de inicio de blobfuse](https://github.com/Azure/azure-storage-fuse#blobfuse)
* [Informe de problemas de blobfuse](https://github.com/Azure/azure-storage-fuse/issues) 

