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
ms.date: 05/15/2019
ms.author: amverma
ms.openlocfilehash: 3ca9a21d105be6f17c1aa40ae1a0ab7f01c38184
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87083419"
---
# <a name="set-up-message-passing-interface-for-hpc"></a>Configuración de la interfaz de paso de mensajes para HPC

Las cargas de trabajo de la interfaz de paso de mensajes (MPI) son una parte importante de las cargas de trabajo de HPC tradicionales. Los tamaños de las máquinas virtuales habilitadas para SR-IOV en Azure permiten que se utilice casi cualquier tipo de MPI. 

La ejecución de trabajos de MPI en máquinas virtuales requiere la configuración de claves de partición (claves p) en un inquilino. Siga los pasos descritos en la sección [Detección de claves de partición](#discover-partition-keys) para obtener más información sobre cómo determinar los valores de la clave p.

## <a name="ucx"></a>UCX

[UCX](https://github.com/openucx/ucx) ofrece el mejor rendimiento en IB y funciona con MPICH y OpenMPI.

```bash
wget https://github.com/openucx/ucx/releases/download/v1.4.0/ucx-1.4.0.tar.gz
tar -xvf ucx-1.4.0.tar.gz
cd ucx-1.4.0
./configure --prefix=<ucx-install-path>
make -j 8 && make install
```

## <a name="openmpi"></a>OpenMPI

Instale UCX como se ha descrito anteriormente.

```bash
sudo yum install –y openmpi
```

Compile OpenMPI.

```bash
wget https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.0.tar.gz
tar -xvf openmpi-4.0.0.tar.gz
cd openmpi-4.0.0
./configure --with-ucx=<ucx-install-path> --prefix=<ompi-install-path>
make -j 8 && make install
```

Ejecute OpenMPI.

```bash
<ompi-install-path>/bin/mpirun -np 2 --map-by node --hostfile ~/hostfile -mca pml ucx --mca btl ^vader,tcp,openib -x UCX_NET_DEVICES=mlx5_0:1  -x UCX_IB_PKEY=0x0003  ./osu_latency
```

Compruebe la clave de partición como se indicó anteriormente.

## <a name="mpich"></a>MPICH

Instale UCX como se ha descrito anteriormente.

Compile MPICH.

```bash
wget https://www.mpich.org/static/downloads/3.3/mpich-3.3.tar.gz
tar -xvf mpich-3.3.tar.gz
cd mpich-3.3
./configure --with-ucx=<ucx-install-path> --prefix=<mpich-install-path> --with-device=ch4:ucx
make -j 8 && make install
```

Ejecute MPICH.

```bash
<mpich-install-path>/bin/mpiexec -n 2 -hostfile ~/hostfile -env UCX_IB_PKEY=0x0003 -bind-to hwthread ./osu_latency
```

Compruebe la clave de partición como se indicó anteriormente.

## <a name="mvapich2"></a>MVAPICH2

Compile MVAPICH2.

```bash
wget http://mvapich.cse.ohio-state.edu/download/mvapich/mv2/mvapich2-2.3.tar.gz
tar -xv mvapich2-2.3.tar.gz
cd mvapich2-2.3
./configure --prefix=<mvapich2-install-path>
make -j 8 && make install
```

Ejecute MVAPICH2.

```bash
<mvapich2-install-path>/bin/mpirun_rsh -np 2 -hostfile ~/hostfile MV2_CPU_MAPPING=48 ./osu_latency
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

## <a name="intel-mpi"></a>Intel MPI

[Descargue Intel MPI](https://software.intel.com/mpi-library/choose-download).

Cambie la variable de entorno I_MPI_FABRICS según la versión. Para Intel MPI 2018, utilice `I_MPI_FABRICS=shm:ofa` y para 2019, use `I_MPI_FABRICS=shm:ofi`.

De forma predeterminada, el anclaje del proceso funciona correctamente para 15, 30 y 60 PPN.

## <a name="osu-mpi-benchmarks"></a>Pruebas comparativas OSU MPI

[Descargue los bancos de pruebas de MPI de OSU](http://mvapich.cse.ohio-state.edu/benchmarks/) y descomprima el archivo.

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

Tenga en cuenta mientras exista el inquilino (AVSet o VMSS), las PKEY seguirán siendo las mismas. Esto sucede incluso cuando se agregan o eliminan nodos. Los nuevos inquilinos obtienen PKEY diferentes.


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

Obtenga más información acerca de [HPC](/azure/architecture/topics/high-performance-computing/) en Azure.
