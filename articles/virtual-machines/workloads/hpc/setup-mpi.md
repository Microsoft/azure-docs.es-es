---
title: 'Configuración de la interfaz de paso de mensajes para HPC: Azure Virtual Machines | Microsoft Docs'
description: Obtenga información sobre cómo configurar MPI para HPC en Azure.
services: virtual-machines
documentationcenter: ''
author: vermagit
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: article
ms.date: 08/06/2020
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: 210b2935cd2df81b0ff079c9a1c945fe770933f9
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87926525"
---
# <a name="set-up-message-passing-interface-for-hpc"></a>Configuración de la interfaz de paso de mensajes para HPC

La [interfaz de paso de mensajes (MPI)](https://en.wikipedia.org/wiki/Message_Passing_Interface) es una biblioteca abierta y con estándar de facto para operaciones de paralelización de memoria distribuida. Se suele usar en muchas cargas de trabajo de HPC. Las cargas de trabajo de HPC de las máquinas virtuales de [serie H](../../sizes-hpc.md) y [serie N](../../sizes-gpu.md) [compatibles con RDMA](../../sizes-hpc.md#rdma-capable-instances) pueden usar MPI para comunicarse a través de la red InfiniBand de baja latencia y ancho de banda alto.

Los tamaños de máquina virtual habilitados para SR-IOV en Azure (HBv2, HB, HC, NCv3 y NDv2) permiten que se use prácticamente cualquier tipo de MPI con Mellanox OFED. En las máquinas virtuales que no están habilitadas para SR-IOV, las implementaciones de MPI compatibles usan la interfaz Microsoft Network Direct (ND) para comunicarse entre máquinas virtuales. Por lo tanto, solo se admiten las versiones de Microsoft MPI (MS-MPI) 2012 R2 o posterior e Intel MPI 5.x. Las versiones posteriores (2017 y 2018) de la biblioteca en tiempo de ejecución de MPI pueden ser o no compatibles con los controladores de Azure RDMA.

En el caso de las [máquinas virtuales compatibles con RDMA](../../sizes-hpc.md#rdma-capable-instances) habilitadas para SR-IOV, la [ versión 7.6 o posterior de CentOS-HPC](https://techcommunity.microsoft.com/t5/Azure-Compute/CentOS-HPC-VM-Image-for-SR-IOV-enabled-Azure-HPC-VMs/ba-p/665557) de las imágenes de máquina virtual en Marketplace se han optimizado y cargado previamente con los controladores OFED para RDMA y varias bibliotecas de MPI y paquetes de informática científica más usados, son la manera más fácil de empezar.

Aunque el ejemplo que se muestra aquí es para RHEL/CentOS, los pasos son generales y se pueden usar en cualquier sistema operativo de Linux compatible, como Ubuntu (16.04, 18.04, 19.04, 20.04) y SLES (12 SP4 y 15). Hay más ejemplos para configurar otras implementaciones de MPI en otras distribuciones en el repositorio [azhpc-images](https://github.com/Azure/azhpc-images/blob/master/ubuntu/ubuntu-18.x/ubuntu-18.04-hpc/install_mpis.sh).

> [!NOTE]
> La ejecución de trabajos de MPI en máquinas virtuales habilitadas para SR-IOV requiere la configuración de claves de partición (claves p) en un inquilino por motivos de aislamiento y seguridad. Siga los pasos descritos en la sección [Detección de claves de partición](#discover-partition-keys) para obtener más información sobre cómo determinar los valores de la clave p y configurarlos correctamente para un trabajo de MPI.

## <a name="ucx"></a>UCX

[Unified Communication X (UCX)](https://github.com/openucx/ucx) es un marco de API de comunicaciones para HPC. Está optimizado para la comunicación de MPI a través de InfiniBand y funciona con muchas implementaciones de MPI, como OpenMPI y MPICH.

```bash
wget https://github.com/openucx/ucx/releases/download/v1.4.0/ucx-1.4.0.tar.gz
tar -xvf ucx-1.4.0.tar.gz
cd ucx-1.4.0
./configure --prefix=<ucx-install-path>
make -j 8 && make install
```

## <a name="hpc-x"></a>HPC-X

El [kit de herramientas de software HPC-X](https://www.mellanox.com/products/hpc-x-toolkit) contiene UCX y HCOLL.

```bash
HPCX_VERSION="v2.6.0"
HPCX_DOWNLOAD_URL=https://azhpcstor.blob.core.windows.net/azhpc-images-store/hpcx-v2.6.0-gcc-MLNX_OFED_LINUX-5.0-1.0.0.0-redhat7.7-x86_64.tbz
get --retry-connrefused --tries=3 --waitretry=5 $HPCX_DOWNLOAD_URL
tar -xvf hpcx-${HPCX_VERSION}-gcc-MLNX_OFED_LINUX-5.0-1.0.0.0-redhat7.7-x86_64.tbz
mv hpcx-${HPCX_VERSION}-gcc-MLNX_OFED_LINUX-5.0-1.0.0.0-redhat7.7-x86_64 ${INSTALL_PREFIX}
HPCX_PATH=${INSTALL_PREFIX}/hpcx-${HPCX_VERSION}-gcc-MLNX_OFED_LINUX-5.0-1.0.0.0-redhat7.7-x86_64
```

Ejecución de HPC-X

```bash
${HPCX_PATH}mpirun -np 2 --map-by ppr:2:node -x UCX_TLS=rc ${HPCX_PATH}/ompi/tests/osu-micro-benchmarks-5.3.2/osu_latency
```

## <a name="openmpi"></a>OpenMPI

Instale UCX como se describió anteriormente. HCOLL forma parte del [kit de herramientas de software HPC-X](https://www.mellanox.com/products/hpc-x-toolkit) y no requiere una instalación especial.

Instale OpenMPI desde los paquetes disponibles en el repositorio.

```bash
sudo yum install –y openmpi
```

Compile OpenMPI.

```bash
OMPI_VERSION="4.0.3"
OMPI_DOWNLOAD_URL=https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-${OMPI_VERSION}.tar.gz
wget --retry-connrefused --tries=3 --waitretry=5 $OMPI_DOWNLOAD_URL
tar -xvf openmpi-${OMPI_VERSION}.tar.gz
cd openmpi-${OMPI_VERSION}
./configure --prefix=${INSTALL_PREFIX}/openmpi-${OMPI_VERSION} --with-ucx=${UCX_PATH} --with-hcoll=${HCOLL_PATH} --enable-mpirun-prefix-by-default --with-platform=contrib/platform/mellanox/optimized && make -j$(nproc) && make install
```

Ejecute OpenMPI.

```bash
${INSTALL_PREFIX}/bin/mpirun -np 2 --map-by node --hostfile ~/hostfile -mca pml ucx --mca btl ^vader,tcp,openib -x UCX_NET_DEVICES=mlx5_0:1  -x UCX_IB_PKEY=0x0003  ./osu_latency
```

Compruebe la clave de partición como se indicó anteriormente.

## <a name="intel-mpi"></a>Intel MPI

Descargue su elección de versión de [Intel MPI](https://software.intel.com/mpi-library/choose-download). Cambie la variable de entorno I_MPI_FABRICS según la versión. Para Intel MPI 2018, utilice `I_MPI_FABRICS=shm:ofa` y para 2019, use `I_MPI_FABRICS=shm:ofi`.

### <a name="non-sr-iov-vms"></a>Máquinas virtuales que no son SR-IOV
En el caso de las máquinas virtuales que no son SR-IOV, un ejemplo de descarga de la [versión de evaluación gratuita](https://registrationcenter.intel.com/en/forms/?productid=1740) del runtime 5.x es el siguiente:
```bash
wget http://registrationcenter-download.intel.com/akdlm/irc_nas/tec/9278/l_mpi_p_5.1.3.223.tgz
```
Para ver los pasos de instalación, vea la [guía de instalación de la Biblioteca de Intel MPI](https://registrationcenter-download.intel.com/akdlm/irc_nas/1718/INSTALL.html?lang=en&fileExt=.html).
Si lo desea, habilite ptrace para los procesos que no son de depurador no raíz (necesario para las versiones más recientes de Intel MPI).
```bash
echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
```

### <a name="suse-linux"></a>SUSE Linux
Para las versiones de imágenes de máquina virtual de SUSE Linux Enterprise Server SLES 12 SP3 para HPC, SLES 12 SP3 para HPC (Premium), SLES 12 SP1 para HPC, SLES 12 SP1 para HPC (Premium), SLES 12 SP4 y SLES 15, los controladores RDMA se instalan y los paquetes de Intel MPI se distribuyen en la máquina virtual. Instale MPI ejecutando el comando siguiente:
```bash
sudo rpm -v -i --nodeps /opt/intelMPI/intel_mpi_packages/*.rpm
```

## <a name="mpich"></a>MPICH

Instale UCX como se describió anteriormente. Compile MPICH.

```bash
wget https://www.mpich.org/static/downloads/3.3/mpich-3.3.tar.gz
tar -xvf mpich-3.3.tar.gz
cd mpich-3.3
./configure --with-ucx=${UCX_PATH} --prefix=${INSTALL_PREFIX} --with-device=ch4:ucx
make -j 8 && make install
```

Ejecute MPICH.

```bash
${INSTALL_PREFIX}/bin/mpiexec -n 2 -hostfile ~/hostfile -env UCX_IB_PKEY=0x0003 -bind-to hwthread ./osu_latency
```

Compruebe la clave de partición como se indicó anteriormente.

## <a name="mvapich2"></a>MVAPICH2

Compile MVAPICH2.

```bash
wget http://mvapich.cse.ohio-state.edu/download/mvapich/mv2/mvapich2-2.3.tar.gz
tar -xv mvapich2-2.3.tar.gz
cd mvapich2-2.3
./configure --prefix=${INSTALL_PREFIX}
make -j 8 && make install
```

Ejecute MVAPICH2.

```bash
${INSTALL_PREFIX}/bin/mpirun_rsh -np 2 -hostfile ~/hostfile MV2_CPU_MAPPING=48 ./osu_latency
```

## <a name="platform-mpi-community-edition"></a>Platform MPI Community Edition

Instale los paquetes necesarios para Platform MPI.

```bash
sudo yum install libstdc++.i686
sudo yum install glibc.i686
Download platform MPI at https://www.ibm.com/developerworks/downloads/im/mpi/index.html 
sudo ./platform_mpi-09.01.04.03r-ce.bin
```

Siga el proceso de instalación.

## <a name="osu-mpi-benchmarks"></a>Pruebas comparativas OSU MPI

Descargue los [bancos de pruebas de MPI de OSU](http://mvapich.cse.ohio-state.edu/benchmarks/) y descomprima el archivo.

```bash
wget http://mvapich.cse.ohio-state.edu/download/mvapich/osu-micro-benchmarks-5.5.tar.gz
tar –xvf osu-micro-benchmarks-5.5.tar.gz
cd osu-micro-benchmarks-5.5
```

Compilar las pruebas comparativas con una biblioteca de MPI particular:

```bash
CC=<mpi-install-path/bin/mpicc>CXX=<mpi-install-path/bin/mpicxx> ./configure 
make
```

Las pruebas comparativas de MPI están en la carpeta `mpi/`.


## <a name="discover-partition-keys"></a>Detección de claves de partición

Detecte las claves de partición (p-keys) para comunicarse con otras máquinas virtuales dentro del mismo inquilino (conjunto de disponibilidad o conjunto de escalado de máquinas virtuales).

```bash
/sys/class/infiniband/mlx5_0/ports/1/pkeys/0
/sys/class/infiniband/mlx5_0/ports/1/pkeys/1
```

La mayor de las dos es la clave de inquilino que debe usarse con MPI. Ejemplo: Si las siguientes son las claves p, se debe usar 0x800b con MPI.

```bash
cat /sys/class/infiniband/mlx5_0/ports/1/pkeys/0
0x800b
cat /sys/class/infiniband/mlx5_0/ports/1/pkeys/1
0x7fff
```

Utilice la partición que no sea la clave de partición predeterminada (0x7fff). UCX requiere el MSB de la clave p para borrarse. Por ejemplo, establezca UCX_IB_PKEY como 0x000b para 0x800b.

Tenga en cuenta también que, siempre y cuando el inquilino exista (conjunto de disponibilidad o conjunto de escalado de máquinas virtuales), las PKEY seguirán siendo las mismas. Esto sucede incluso cuando se agregan o eliminan nodos. Los nuevos inquilinos obtienen PKEY diferentes.


## <a name="set-up-user-limits-for-mpi"></a>Configuración de los límites de usuario para MPI

Configure los límites de usuario para MPI.

```bash
cat << EOF | sudo tee -a /etc/security/limits.conf
*               hard    memlock         unlimited
*               soft    memlock         unlimited
*               hard    nofile          65535
*               soft    nofile          65535
EOF
```

## <a name="set-up-ssh-keys-for-mpi"></a>Configure las claves SSH para MPI

Configure las claves SSH para los tipos MPI que lo requieran.

```bash
ssh-keygen -f /home/$USER/.ssh/id_rsa -t rsa -N ''
cat << EOF > /home/$USER/.ssh/config
Host *
    StrictHostKeyChecking no
EOF
cat /home/$USER/.ssh/id_rsa.pub >> /home/$USER/.ssh/authorized_keys
chmod 600 /home/$USER/.ssh/authorized_keys
chmod 644 /home/$USER/.ssh/config
```

La sintaxis anterior asume un directorio principal compartido; de lo contrario, hay que copiar el directorio.ssh a cada nodo.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre las máquinas virtuales de las series [H](../../sizes-hpc.md#rdma-capable-instances) y [N](../../sizes-hpc.md) [habilitadas para InfiniBand](../../sizes-gpu.md)
- En los artículos [Introducción a las máquinas virtuales de la serie HB](hb-series-overview.md) e [Introducción a las máquinas virtuales de la serie HC](hc-series-overview.md), aprenderá a configurar de forma óptima las cargas de trabajo para mejorar el rendimiento y la escalabilidad.
- En los [blogs de la comunidad de Azure Compute Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute), encontrará los anuncios más recientes y algunos ejemplos y resultados de HPC.
- Si desea una visión general de la arquitectura de la ejecución de cargas de trabajo de HPC, consulte [Informática de alto rendimiento (HPC) en Azure](/azure/architecture/topics/high-performance-computing/).
