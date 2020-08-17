---
title: Instalación de controladores de GPU de la serie N de Azure para Linux
description: Instalación de controladores de GPU de NVIDIA para máquinas virtuales de la serie N que se ejecutan en Linux en Azure
services: virtual-machines-linux
author: vikancha-MSFT
ms.service: virtual-machines-linux
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 01/09/2019
ms.author: vikancha
ms.openlocfilehash: 02fbe721f1bf5737ad1d10d656ea75ed1372b484
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87284887"
---
# <a name="install-nvidia-gpu-drivers-on-n-series-vms-running-linux"></a>Instalación de controladores de GPU de NVIDIA en máquinas virtuales de la serie N con Linux

Para aprovechar las funcionalidades de GPU de las VM de la serie N de Azure respaldadas por GPU de NVIDIA, deben instalarse controladores de GPU de NVIDIA. La [extensión de controlador de GPU de NVIDIA](../extensions/hpccompute-gpu-linux.md) instala los controladores CUDA de NVIDIA o GRID adecuados en una máquina virtual de la serie N. Instale o administre la extensión mediante Azure Portal o con herramientas como las plantillas de la CLI de Azure o Azure Resource Manager. Consulte la [documentación de la extensión de controlador de GPU de NVIDIA](../extensions/hpccompute-gpu-linux.md) para ver las distribuciones compatibles y los pasos de implementación.

Si decide instalar manualmente los controladores de GPU de NVIDIA, este artículo proporciona los pasos de instalación y verificación, los controladores y las distribuciones compatibles. También está disponible la información de instalación manual del controlador para las [máquinas virtuales Windows](../windows/n-series-driver-setup.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

Para conocer las especificaciones de máquina virtual de la serie N, las capacidades de almacenamiento y los detalles del disco, vea [Tamaño de máquinas virtuales para GPU Linux](../sizes-gpu.md?toc=/azure/virtual-machines/linux/toc.json). 

[!INCLUDE [virtual-machines-n-series-linux-support](../../../includes/virtual-machines-n-series-linux-support.md)]

## <a name="install-cuda-drivers-on-n-series-vms"></a>Instalación de controladores CUDA en máquinas virtuales de la serie N

Estos son los pasos para instalar controladores CUDA desde el kit de herramientas de NVIDIA CUDA en máquinas virtuales de la serie N. 


Los desarrolladores de C y C++, si lo desean, pueden instalar el kit de herramientas completo para crear aplicaciones aceleradas por GPU. Para obtener más información, consulte la [guía de instalación de CUDA](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html).

Para instalar los controladores de CUDA, realice una conexión SSH a cada máquina virtual. Para comprobar que el sistema dispone de una GPU compatible con CUDA, ejecute el siguiente comando:

```bash
lspci | grep -i NVIDIA
```
Verá un resultado similar al siguiente ejemplo (mostrando una tarjeta NVIDIA Tesla K80):

![Resultado del comando de Ispci](./media/n-series-driver-setup/lspci.png)

Luego, ejecute los comandos de instalación específicos de su distribución.

### <a name="ubuntu"></a>Ubuntu 

1. Descargue e instale los controladores de CUDA del sitio web de NVIDIA. Por ejemplo, para Ubuntu 16.04 LTS:
   ```bash
   CUDA_REPO_PKG=cuda-repo-ubuntu1604_10.0.130-1_amd64.deb

   wget -O /tmp/${CUDA_REPO_PKG} http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/${CUDA_REPO_PKG} 

   sudo dpkg -i /tmp/${CUDA_REPO_PKG}

   sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub 

   rm -f /tmp/${CUDA_REPO_PKG}

   sudo apt-get update

   sudo apt-get install cuda-drivers

   ```

   La instalación puede tardar varios minutos.

2. Para instalar (opcional) el kit de herramientas CUDA completo, escriba:

   ```bash
   sudo apt-get install cuda
   ```

3. Reinicie la máquina virtual y continúe para comprobar la instalación.

#### <a name="cuda-driver-updates"></a>Actualizaciones de controladores CUDA

Se recomienda actualizar periódicamente los controladores CUDA después de la implementación.

```bash
sudo apt-get update

sudo apt-get upgrade -y

sudo apt-get dist-upgrade -y

sudo apt-get install cuda-drivers

sudo reboot
```

### <a name="centos-or-red-hat-enterprise-linux"></a>CentOS o Red Hat Enterprise Linux

1. Actualice el kernel (recomendado). Si decide no actualizar el kernel, asegúrese de que las versiones de `kernel-devel` y `dkms` sean adecuadas para el kernel.

   ```
   sudo yum install kernel kernel-tools kernel-headers kernel-devel
  
   sudo reboot

2. Install the latest [Linux Integration Services for Hyper-V and Azure](https://www.microsoft.com/download/details.aspx?id=55106).

   ```bash
   wget https://aka.ms/lis
 
   tar xvzf lis
 
   cd LISISO
 
   sudo ./install.sh
 
   sudo reboot
   ```
 
3. Vuelva a conectarse a la VM y continúe con la instalación con los siguientes comandos:

   ```bash
   sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

   sudo yum install dkms

   CUDA_REPO_PKG=cuda-repo-rhel7-10.0.130-1.x86_64.rpm

   wget http://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64/${CUDA_REPO_PKG} -O /tmp/${CUDA_REPO_PKG}

   sudo rpm -ivh /tmp/${CUDA_REPO_PKG}

   rm -f /tmp/${CUDA_REPO_PKG}

   sudo yum install cuda-drivers
   ```

   La instalación puede tardar varios minutos. 

4. Para instalar (opcional) el kit de herramientas CUDA completo, escriba:

   ```bash
   sudo yum install cuda
   ```

5. Reinicie la máquina virtual y continúe para comprobar la instalación.

### <a name="verify-driver-installation"></a>Comprobación de la instalación del controlador

Para consultar el estado del dispositivo de GPU, acceda mediante SSH a la máquina virtual y ejecute la utilidad de línea de comandos [smi nvidia](https://developer.nvidia.com/nvidia-system-management-interface) que se instala con el controlador. 

Si el controlador está instalado, verá una salida parecida a la siguiente. Tenga en cuenta que **GPU-Util** muestra 0 %, salvo que se esté ejecutando una carga de trabajo de GPU en la máquina virtual. La versión del controlador y los detalles de GPU pueden ser diferentes de los que se muestran.

![Estado del dispositivo de NVIDIA](./media/n-series-driver-setup/smi.png)

## <a name="rdma-network-connectivity"></a>Conectividad de red RDMA

La conectividad de red RDMA puede habilitarse en las VM de la serie N que puedan usar RDMA, como la NC24r, implementadas en el mismo conjunto de disponibilidad o en un único grupo de selección de ubicación en un conjunto de escalado de máquinas virtuales (VM). Ahora, la red RDMA admite el tráfico de interfaz de paso de mensajes (MPI) para aplicaciones que se ejecutan con Intel MPI 5.x o una versión posterior. Requisitos adicionales siguientes:

### <a name="distributions"></a>Distribuciones

Las máquinas virtuales de la serie N que puedan usar RDMA se implementan desde cualquiera de las imágenes de Azure Marketplace que admita la conectividad RDMA en máquinas virtuales de la serie N:
  
* **Ubuntu 16.04 LTS**: configure controladores RDMA en la máquina virtual y regístrese en Intel para descargar Intel MPI:

  [!INCLUDE [virtual-machines-common-ubuntu-rdma](../../../includes/virtual-machines-common-ubuntu-rdma.md)]

* **HPC basada en CentOS 7.4**: los controladores RDMA e Intel MPI 5.1 están instalados en la máquina virtual.

## <a name="install-grid-drivers-on-nv-or-nvv3-series-vms"></a>Instalación de controladores GRID en VM de la serie NV o NVv3

Para instalar los controladores NVIDIA GRID en VM de la serie NV o NVv3, establezca una conexión SSH a cada VM y siga los pasos para su distribución de Linux. 

### <a name="ubuntu"></a>Ubuntu 

1. Ejecute el comando `lspci`. Compruebe que la tarjeta o tarjetas NVIDIA M60 son visibles como dispositivos PCI.

2. Instale las actualizaciones.

   ```bash
   sudo apt-get update

   sudo apt-get upgrade -y

   sudo apt-get dist-upgrade -y

   sudo apt-get install build-essential ubuntu-desktop -y
   
   sudo apt-get install linux-azure -y
   ```
3. Deshabilite el controlador de kernel Nouveau que es incompatible con el controlador NVIDIA. (Utilice solo el controlador NVIDIA en VM NV o NVv2). Para ello, cree un archivo en `/etc/modprobe.d` llamado `nouveau.conf` con el siguiente contenido:

   ```
   blacklist nouveau

   blacklist lbm-nouveau
   ```


4. Reinicie la máquina virtual y vuelva a conectar. Salga del servidor X:

   ```bash
   sudo systemctl stop lightdm.service
   ```

5. Descargue e instale el controlador de GRID:

   ```bash
   wget -O NVIDIA-Linux-x86_64-grid.run https://go.microsoft.com/fwlink/?linkid=874272  

   chmod +x NVIDIA-Linux-x86_64-grid.run

   sudo ./NVIDIA-Linux-x86_64-grid.run
   ``` 

6. Cuando se le pregunte si desea ejecutar la utilidad nvidia-xconfig para actualizar el archivo de configuración de X, seleccione **Sí**.

7. Una vez completada la instalación, copie /etc/nvidia/gridd.conf.template en un nuevo archivo gridd.conf en la ubicación /etc/nvidia/

   ```bash
   sudo cp /etc/nvidia/gridd.conf.template /etc/nvidia/gridd.conf
   ```

8. Agregue lo siguiente a `/etc/nvidia/gridd.conf`:
 
   ```
   IgnoreSP=FALSE
   EnableUI=FALSE
   ```
   
9. Quite lo siguiente de `/etc/nvidia/gridd.conf` si está presente:
 
   ```
   FeatureType=0
   ```
10. Reinicie la máquina virtual y continúe para comprobar la instalación.


### <a name="centos-or-red-hat-enterprise-linux"></a>CentOS o Red Hat Enterprise Linux 

1. Actualice el kernel y DKMS (recomendado). Si decide no actualizar el kernel, asegúrese de que las versiones de `kernel-devel` y `dkms` sean adecuadas para el kernel.
 
   ```bash  
   sudo yum update
 
   sudo yum install kernel-devel
 
   sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
 
   sudo yum install dkms
   
   sudo yum install hyperv-daemons
   ```

2. Deshabilite el controlador de kernel Nouveau que es incompatible con el controlador NVIDIA. (Utilice solo el controlador NVIDIA en VM NV o NV2). Para ello, cree un archivo en `/etc/modprobe.d` llamado `nouveau.conf` con el siguiente contenido:

   ```
   blacklist nouveau

   blacklist lbm-nouveau
   ```
 
3. Reinicie la máquina virtual, vuelva a conectarse e instale la versión más reciente de [Linux Integration Services para Hyper-V y Azure](https://www.microsoft.com/download/details.aspx?id=55106).
 
   ```bash
   wget https://aka.ms/lis

   tar xvzf lis

   cd LISISO

   sudo ./install.sh

   sudo reboot

   ```
 
4. Vuelva a conectarse a la máquina virtual y ejecute el comando `lspci`. Compruebe que la tarjeta o tarjetas NVIDIA M60 son visibles como dispositivos PCI.
 
5. Descargue e instale el controlador de GRID:

   ```bash
   wget -O NVIDIA-Linux-x86_64-grid.run https://go.microsoft.com/fwlink/?linkid=874272  

   chmod +x NVIDIA-Linux-x86_64-grid.run

   sudo ./NVIDIA-Linux-x86_64-grid.run
   ``` 
6. Cuando se le pregunte si desea ejecutar la utilidad nvidia-xconfig para actualizar el archivo de configuración de X, seleccione **Sí**.

7. Una vez completada la instalación, copie /etc/nvidia/gridd.conf.template en un nuevo archivo gridd.conf en la ubicación /etc/nvidia/
  
   ```bash
   sudo cp /etc/nvidia/gridd.conf.template /etc/nvidia/gridd.conf
   ```
  
8. Agregue lo siguiente a `/etc/nvidia/gridd.conf`:
 
   ```
   IgnoreSP=FALSE
   EnableUI=FALSE 
   ```
9. Quite lo siguiente de `/etc/nvidia/gridd.conf` si está presente:
 
   ```
   FeatureType=0
   ```
10. Reinicie la máquina virtual y continúe para comprobar la instalación.


### <a name="verify-driver-installation"></a>Comprobación de la instalación del controlador


Para consultar el estado del dispositivo de GPU, acceda mediante SSH a la máquina virtual y ejecute la utilidad de línea de comandos [smi nvidia](https://developer.nvidia.com/nvidia-system-management-interface) que se instala con el controlador. 

Si el controlador está instalado, verá una salida parecida a la siguiente. Tenga en cuenta que **GPU-Util** muestra 0 %, salvo que se esté ejecutando una carga de trabajo de GPU en la máquina virtual. La versión del controlador y los detalles de GPU pueden ser diferentes de los que se muestran.

![Estado del dispositivo de NVIDIA](./media/n-series-driver-setup/smi-nv.png)
 

### <a name="x11-server"></a>Servidor X11
Si necesita un servidor X11 para conexiones remotas a la VM NV o NVv2, se recomienda [x11vnc](http://www.karlrunge.com/x11vnc/), ya que permite la aceleración de hardware de gráficos. El BusID del dispositivo M60 debe agregarse manualmente al archivo X11 de configuración (normalmente, `etc/X11/xorg.conf`). Agregue una sección `"Device"` similar a la siguiente:
 
```
Section "Device"
    Identifier     "Device0"
    Driver         "nvidia"
    VendorName     "NVIDIA Corporation"
    BoardName      "Tesla M60"
    BusID          "PCI:0@your-BusID:0:0"
EndSection
```
 
Además, actualice su sección `"Screen"` para usar este dispositivo.
 
El BusID decimal puede encontrarse al ejecutar

```bash
nvidia-xconfig --query-gpu-info | awk '/PCI BusID/{print $4}'
```
 
El BusID puede cambiar cuando se reasigna o reinicia una máquina virtual. Por lo tanto, puede ser conveniente crear un script para actualizar el BusID en la configuración de X11 cuando se reinicie una máquina virtual. Por ejemplo, cree un script llamado `busidupdate.sh` (u otro nombre que elija) con un contenido similar al siguiente:

```bash 
#!/bin/bash
XCONFIG="/etc/X11/xorg.conf"
OLDBUSID=`awk '/BusID/{gsub(/"/, "", $2); print $2}' ${XCONFIG}`
NEWBUSID=`nvidia-xconfig --query-gpu-info | awk '/PCI BusID/{print $4}'`

if [[ "${OLDBUSID}" == "${NEWBUSID}" ]] ; then
        echo "NVIDIA BUSID not changed - nothing to do"
else
        echo "NVIDIA BUSID changed from \"${OLDBUSID}\" to \"${NEWBUSID}\": Updating ${XCONFIG}" 
        sed -e 's|BusID.*|BusID          '\"${NEWBUSID}\"'|' -i ${XCONFIG}
fi
```

A continuación, cree una entrada para el script actualizado en `/etc/rc.d/rc3.d` para que se invoque el script como root en el arranque.

## <a name="troubleshooting"></a>Solución de problemas

* Puede establecer el modo de persistencia mediante `nvidia-smi` para que la salida del comando sea más rápida cuando necesita consultar tarjetas. Para establecer el modo de persistencia, ejecute `nvidia-smi -pm 1`. Tenga en cuenta que, si se reinicia la máquina virtual, la configuración del modo desaparece. Siempre puede crear un script de la configuración de modo para ejecutarla al inicio.
* Si ha actualizado los controladores CUDA de NVIDIA a la versión más reciente y observa que la conectividad RDMA ya no funciona, [reinstale los controladores RDMA](#rdma-network-connectivity) para restablecer la conectividad. 

## <a name="next-steps"></a>Pasos siguientes

* Para capturar una imagen de una VM Linux con los controladores de NVIDIA instalados, vea [Procedimiento para generalizar y capturar una máquina virtual Linux](capture-image.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
