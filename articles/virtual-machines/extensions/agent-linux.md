---
title: Introducción al agente Linux Virtual Machines de Azure
description: Aprenda a instalar y configurar el Agente de Linux (waagent) para administrar la interacción de la máquina virtual con el controlador de tejido de Azure.
author: axayjo
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.topic: article
ms.date: 10/17/2016
ms.author: akjosh
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 2237b0b0d0c1f6e95e100743b377f9c04f57210f
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279710"
---
# <a name="understanding-and-using-the-azure-linux-agent"></a>Información y uso del agente de Linux de Azure

El agente Linux de Microsoft Azure (waagent) administra el aprovisionamiento de Linux & FreeBSD y la interacción de máquina virtual con el controlador de tejido de Azure. Además de disfrutar de la funcionalidad de aprovisionamiento mediante el agente Linux, Azure también ofrece la opción de usar cloud-init para algunos sistemas operativos Linux. El agente Linux proporciona la siguiente funcionalidad para las implementaciones de IaaS de Linux y FreeBSD:

> [!NOTE]
> Para más información, vea el [archivo Léame](https://github.com/Azure/WALinuxAgent/blob/master/README.md).
> 
> 

* **Aprovisionamiento de imágenes**
  
  * Creación de una cuenta de usuario
  * Configuración de los tipos de autenticación de SSH
  * Implementación de claves públicas y pares de claves de SSH
  * Configuración del nombre de host
  * Publicación del nombre de host para DNS de plataforma
  * Informe de huella digital de clave de host de SSH para la plataforma
  * Administración del disco de recursos
  * Formato y montaje del discos de recursos
  * Configuración del espacio de intercambio
* **Redes**
  
  * Administra las rutas para mejorar la compatibilidad con los servidores DHCP de la plataforma
  * Asegura la estabilidad del nombre de interfaz de red
* **Kernel**
  
  * Configura NUMA virtual (deshabilitar para el kernel cuya versión sea inferior a `2.6.37`)
  * Consume entropía de Hyper-V para /dev/random
  * Configura tiempos de espera SCSI para el dispositivo raíz (que podría ser remoto)
* **Diagnóstico**
  
  * Redirección de la consola al puerto serie
* **Implementaciones de SCVMM**
  
  * Detecta y arranca el agente VMM para Linux cuando se ejecuta en un entorno de System Center Virtual Machine Manager 2012 R2
* **Extensión de máquina virtual**
  
  * Inserta un componente creado por Microsoft y sus socios en la VM de Linux (IaaS) para habilitar software y automatización de la configuración
  * Implementación de referencias de extensión de máquina virtual en [https://github.com/Azure/azure-linux-extensions](https://github.com/Azure/azure-linux-extensions)

## <a name="communication"></a>Comunicación
El flujo de información desde la plataforma hasta el agente se produce a través de dos canales:

* Un DVD conectado de tiempo de arranque para las implementaciones de IaaS. Este DVD incluye un archivo de configuración conforme a OVF que incluye toda la información de aprovisionamiento diferente de los pares de clave reales de SSH.
* Un extremo de TCP que expone una API de REST que se usa para obtener la configuración de implementación y topología.

## <a name="requirements"></a>Requisitos
Los siguientes sistemas se han probado y se sabe que funcionan con el agente Linux de Azure:

> [!NOTE]
> Esta lista puede ser distinta de la lista oficial de [distribuciones compatibles](../linux/endorsed-distros.md).
> 
> 

* CoreOS
* CentOS 6.3+
* Red Hat Enterprise Linux 6.7+
* Debian 7.0+
* Ubuntu 12.04+
* openSUSE 12.3+
* SLES 11 SP3+
* Oracle Linux 6.4+

Otros sistemas compatibles:

* FreeBSD 10+ (Agente Linux de Azure v2.0.10+)

El agente de Linux depende de algunos paquetes de sistema para funcionar correctamente:

* Python 2.6
* OpenSSL 1.0+
* OpenSSH 5.3+
* Utilidades del sistema de archivos: sfdisk, fdisk, mkfs, parted
* Herramientas de contraseña: chpasswd, sudo
* Herramientas de procesamiento de texto: sed, grep
* Herramientas de red: ip-route
* Compatibilidad de kernel para el montaje de sistemas de archivos UDF.

Asegúrese de que la máquina virtual tenga acceso a la dirección IP 168.63.129.16. Para más información, vea [¿Qué es la dirección IP 168.63.129.16?](https://docs.microsoft.com/azure/virtual-network/what-is-ip-address-168-63-129-16)


## <a name="installation"></a>Instalación
La instalación con RPM o un paquete de DEB del repositorio de paquetes de su distribución es el método preferido para instalar y actualizar el agente Linux de Azure. Todos los [proveedores de distribución aprobada](../linux/endorsed-distros.md) integran el paquete de agente Linux de Azure en sus imágenes y repositorios.

Consulte la documentación en el [repositorio de agente Linux de Azure en GitHub](https://github.com/Azure/WALinuxAgent) para opciones de instalación avanzada, como la instalación desde el origen o en ubicaciones o prefijos personalizados.

## <a name="command-line-options"></a>Opciones de la línea de comandos
### <a name="flags"></a>Marcas
* verbose: aumenta el nivel de detalle de un comando específico.
* force: omite la confirmación interactiva de algunos comandos.

### <a name="commands"></a>Comandos:
* help: enumera los comandos y las marcas compatibles.
* deprovision: intenta limpiar el sistema y prepararlo para un reaprovisionamiento. La operación siguiente elimina:
  
  * Todas las claves de host de SSH (si Provisioning.RegenerateSshHostKeyPair es "y" en el archivo de configuración)
  * Configuración de Nameserver en /etc/resolv.conf
  * Contraseña raíz desde /etc/shadow (si Provisioning.DeleteRootPassword es "y" en el archivo de configuración)
  * Concesiones del cliente de DHCP en caché
  * Restablece el nombre de host a localhost.localdomain

> [!WARNING]
> El desaprovisionamiento no garantiza que se haya borrado toda información sensible de la imagen y que sea adecuada para su redistribución.
> 
> 

* deprovision+user: realiza todo de -deprovision (arriba) y también elimina la última cuenta de usuario aprovisionada (obtenida de /var/lib/waagent) y los datos asociados. Este parámetro está presente cuando se desaprovisiona una imagen que se estaba aprovisionando anteriormente en Azure de modo que se pueda capturar y volver a usar.
* version: muestra la versión de waagent.
* serialconsole: configura GRUB para marcar ttyS0 (el primer puerto serie) como la consola de arranque. Esto asegura que los registros de arranque del kernel se envíen al puerto serie y queden disponibles para su depuración.
* daemon: ejecuta waagent como un demonio para administrar la interacción con la plataforma. Este argumento se especifica en waagent en el script init de waagent.
* start: ejecuta waagent como proceso en segundo plano.

## <a name="configuration"></a>Configuración
Un archivo de configuración (/etc/waagent.conf) controla las acciones de waagent. A continuación se muestra un archivo de configuración de ejemplo:

```config
Provisioning.Enabled=y
Provisioning.DeleteRootPassword=n
Provisioning.RegenerateSshHostKeyPair=y
Provisioning.SshHostKeyPairType=rsa
Provisioning.MonitorHostName=y
Provisioning.DecodeCustomData=n
Provisioning.ExecuteCustomData=n
Provisioning.AllowResetSysUser=n
Provisioning.PasswordCryptId=6
Provisioning.PasswordCryptSaltLength=10
ResourceDisk.Format=y
ResourceDisk.Filesystem=ext4
ResourceDisk.MountPoint=/mnt/resource
ResourceDisk.MountOptions=None
ResourceDisk.EnableSwap=n
ResourceDisk.SwapSizeMB=0
LBProbeResponder=y
Logs.Verbose=n
OS.RootDeviceScsiTimeout=300
OS.OpensslPath=None
HttpProxy.Host=None
HttpProxy.Port=None
AutoUpdate.Enabled=y
```

A continuación se describen varias opciones de configuración. Las opciones de configuración son de tres tipos: Boolean, String o Integer. Las opciones de configuración booleana se pueden especificar como "y" o "n". La palabra clave especial "None" se puede usar para algunas entradas de la configuración tipo cadena como se detalla a continuación:

**Provisioning.Enabled:**  
```txt
Type: Boolean  
Default: y
```
Esta opción permite que el usuario habilite o deshabilite la funcionalidad de aprovisionamiento en el agente. Los valores válidos son "y" o "n". Si se deshabilita el aprovisionamiento, se mantiene el host SSH y las claves de usuario en la imagen y se ignora toda configuración que se especifique en la API de aprovisionamiento de Azure.

> [!NOTE]
> El parámetro `Provisioning.Enabled` toma el valor predeterminado "n" en imágenes Ubuntu Cloud Images que usan cloud-init para el aprovisionamiento.
> 
> 

**Provisioning.DeleteRootPassword:**  
```txt
Type: Boolean  
Default: n
```
Si se establece, se borra la contraseña raíz en el archivo /etc/shadow durante el proceso de aprovisionamiento.

**Provisioning.RegenerateSshHostKeyPair:**  
```txt
Type: Boolean  
Default: y
```
Si se establece, se eliminan todos los pares de clave de host de SSH (ecdsa, dsa y rsa) durante el proceso de aprovisionamiento desde /etc/ssh/. Además, se genera un solo par de clave nuevo.

El tipo de cifrado para el par de clave nuevo se puede configurar mediante la entrada Provisioning.SshHostKeyPairType. Algunas distribuciones vuelven a crear los pares de clave de SSH para los tipos de cifrado que faltan cuando el demonio de SSH se reinicia (por ejemplo, en un nuevo arranque del equipo).

**Provisioning.SshHostKeyPairType:**  
```txt
Type: String  
Default: rsa
```
Esta opción se puede establecer en un tipo de algoritmo de cifrado que es compatible con el demonio de SSH en la máquina virtual. Los valores generalmente admitidos son "rsa", "dsa" y "ecdsa". "putty.exe" en Windows no admite "ecdsa". Por lo tanto, si pretende usar putty.exe en Windows para conectarse a una implementación de Linux, use "rsa" o "dsa".

**Provisioning.MonitorHostName:**  
```txt
Type: Boolean  
Default: y
```
Si se configura, waagent supervisa la máquina virtual de Linux en busca de cambios en el nombre de host (según los devuelve el comando "hostname") y actualiza automáticamente la configuración de red en la imagen para reflejar el cambio. Para insertar el cambio de nombre en los servidores DNS, la red se reinicia en la máquina virtual. Esta acción tiene como resultado una breve pérdida de la conectividad con Internet.

**Provisioning.DecodeCustomData**  
```txt
Type: Boolean  
Default: n
```
Si se establece, waagent descodifica CustomData desde Base64.

**Provisioning.ExecuteCustomData**  
```txt
Type: Boolean  
Default: n
```
Si se establece, waagent ejecuta CustomData después del aprovisionamiento.

**Provisioning.AllowResetSysUser**
```txt
Type: Boolean
Default: n
```
Esta opción permite restablecer la contraseña del usuario SYS; de forma predeterminada, está deshabilitada.

**Provisioning.PasswordCryptId**  
```txt
Type: String  
Default: 6
```
Algoritmo usado por el cifrado al generar el hash de contraseña.  
 1 - MD5  
 2a - Blowfish  
 5 - SHA-256  
 6 - SHA-512  

**Provisioning.PasswordCryptSaltLength**  
```txt
Type: String  
Default: 10
```
Longitud del valor salt aleatorio usado al generar el hash de contraseña.

**ResourceDisk.Format:**  
```txt
Type: Boolean  
Default: y
```
Si se establece, el disco de recursos proporcionado por la plataforma se formatea y se monta a través de waagent si el tipo de sistema de archivos solicitado por el usuario en "ResourceDisk.Filesystem" es cualquiera, salvo "ntfs". Se deja disponible una sola partición de tipo Linux (83) en el disco. Esta partición no se formatea si se puede montar correctamente.

**ResourceDisk.Filesystem:**  
```txt
Type: String  
Default: ext4
```
De esta manera se especifica el tipo de sistema de archivos para el disco de recursos. Los valores admitidos varían según la distribución de Linux. Si la cadena es X, entonces mkfs.X debe estar presente en la imagen de Linux. Las imágenes de SLES 11 deben usar generalmente "ext3". Las imágenes de FreeBSD deben usar "ufs2".

**ResourceDisk.MountPoint:**  
```txt
Type: String  
Default: /mnt/resource 
```
Esta opción especifica la ruta en la cual se monta el disco de recursos. El disco de recursos es un disco *temporal* que podría vaciarse cuando la máquina virtual se desaprovisiona.

**ResourceDisk.MountOptions**  
```txt
Type: String  
Default: None
```
Especifica las opciones de montaje de disco que se pasarán al comando -o de montaje. Esta es una lista de valores separados por comas, por ejemplo, 'nodev,nosuid'. Consulte mount(8) para obtener detalles.

**ResourceDisk.EnableSwap:**  
```txt
Type: Boolean  
Default: n
```
Si se establece, se crea un archivo de intercambio (/swapfile) en el disco de recursos y se agrega al espacio de intercambio del sistema.

**ResourceDisk.SwapSizeMB:**  
```txt
Type: Integer  
Default: 0
```
El tamaño del archivo de intercambio en megabytes.

**Logs.Verbose:**  
```txt
Type: Boolean  
Default: n
```
Si se establece, se aumenta el nivel de detalle del registro. Waagent crea los registros en /var/log/waagent.log y usa la funcionalidad logrotate del sistema para alternar los registros.

**OS.EnableRDMA**  
```txt
Type: Boolean  
Default: n
```
Si se establece, el agente intenta instalar y cargar un controlador de kernel RDMA que coincida con la versión del firmware del hardware subyacente.

**OS.RootDeviceScsiTimeout:**  
```txt
Type: Integer  
Default: 300
```
Esta opción configura el tiempo de espera SCSI en segundos en el disco del sistema operativo y las unidades de datos. Si no se establece, se usan los valores predeterminados del sistema.

**OS.OpensslPath:**  
```txt
Type: String  
Default: None
```
Esta opción se puede usar para especificar una ruta alternativa para que el binario openssl la use para las operaciones criptográficas.

**HttpProxy.Host, HttpProxy.Port**  
```txt
Type: String  
Default: None
```
Si se establece, el agente usa este servidor proxy para tener acceso a Internet. 

**AutoUpdate.Enabled**
```txt
Type: Boolean
Default: y
```
Habilita o deshabilita la actualización automática del procesamiento del estado del objetivo; está habilitada de forma predeterminada.



## <a name="ubuntu-cloud-images"></a>Ubuntu Cloud Images
Las imágenes de tipo Ubuntu Cloud Images usan [cloud-init](https://launchpad.net/ubuntu/+source/cloud-init) para realizar muchas tareas de configuración que, de lo contrario, serían administradas por el agente Linux de Azure. Se aplican las diferencias siguientes:

* **Provisioning.Enabled** toma el valor predeterminado "n" en Ubuntu Cloud Images que utilizan cloud-init para realizar tareas de aprovisionamiento.
* Los parámetros de configuración siguientes no tienen ningún efecto en Ubuntu Cloud Images que usan cloud-init para administrar el disco de recursos y el espacio de intercambio:
  
  * **ResourceDisk.Format**
  * **ResourceDisk.Filesystem**
  * **ResourceDisk.MountPoint**
  * **ResourceDisk.EnableSwap**
  * **ResourceDisk.SwapSizeMB**

* Para más información, vea los recursos siguientes para configurar el punto de montaje del disco de recursos e intercambiar espacio en Ubuntu Cloud Images durante el aprovisionamiento:
  
  * [Ubuntu Wiki: Configure Swap Partitions](https://go.microsoft.com/fwlink/?LinkID=532955&clcid=0x409) (Configuración de particiones de intercambio)
  * [Inyección de datos personalizados en una máquina virtual de Azure](../windows/tutorial-automate-vm-deployment.md)
