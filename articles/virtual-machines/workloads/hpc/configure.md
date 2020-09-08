---
title: Configuración y optimización de máquinas virtuales Azure Virtual Machines de la serie H y N habilitadas para InfiniBand
description: Aprenda a configurar y optimizar las máquinas virtuales tanto de la serie H como de la serie N habilitadas para InfiniBand para HPC.
services: virtual-machines
documentationcenter: ''
author: vermagit
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: article
ms.date: 08/07/2020
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: d4661c0819d214a2c750eb1582559f8d8a5959ed
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88006611"
---
# <a name="configure-and-optimize-vms"></a>Configuración y optimización de máquinas virtuales

En este artículo se comparten técnicas conocidas para configurar y optimizar las máquinas virtuales de las series [H](../../sizes-hpc.md) y [N](../../sizes-gpu.md) habilitadas para InfiniBand para HPC.

## <a name="vm-images"></a>Imágenes de VM
En las máquinas virtuales habilitadas para InfiniBand, se necesitan los controladores adecuados para habilitar RDMA. En Linux, las imágenes de las máquinas virtuales de la versión de CentOS-HPC en Marketplace se configuran previamente con los controladores adecuados. Las imágenes de máquinas virtuales de Ubuntu se pueden configurar con los controladores adecuados mediante las [instrucciones que se indican aquí](https://techcommunity.microsoft.com/t5/azure-compute/configuring-infiniband-for-ubuntu-hpc-and-gpu-vms/ba-p/1221351). También se recomienda crear [imágenes de máquina virtual personalizadas](../../linux/tutorial-custom-images.md) con los controladores y la configuración adecuados y reutilizarlas de forma periódica.

> [!NOTE]
> En las máquinas virtuales de la [serie N](../../sizes-gpu.md) habilitadas para GPU, también se requieren los controladores de GPU adecuados, que se pueden agregar a través de las [extensiones de máquina virtual](../../extensions/hpccompute-gpu-linux.md) o [manualmente](../../linux/n-series-driver-setup.md). Algunas imágenes de máquina virtual de Marketplace también vienen preinstaladas con los controladores de GPU de NVIDIA.

### <a name="centos-hpc-vm-images"></a>Imágenes de máquina virtual de CentOS-HPC

#### <a name="non-sr-iov-enabled-vms"></a>Máquinas virtuales no habilitadas para SR-IOV
En el caso de las [máquinas virtuales compatibles con RDMA](../../sizes-hpc.md#rdma-capable-instances) habilitadas para SR-IOV, las versiones 6.5 o posterior de CentOS-HPC (hasta la 7.5) de Marketplace son adecuadas. Por ejemplo, en las [máquinas virtuales de la serie H16](../../h-series.md), se recomienda usar las versiones de la 7.1 a la 7.5. Estas imágenes de máquina virtual se cargan previamente con los controladores de Network Direct para RDMA y la versión 5.1 de Intel MPI.

> [!NOTE]
> En las imágenes de HPC basadas en CentOS para máquinas virtuales no habilitadas para SR-IOV, las actualizaciones del kernel están deshabilitadas en el archivo de configuración **yum**. Esto se debe a que los controladores RDMA de Linux de Network Direct se distribuyen en forma de paquete RPM y sus actualizaciones de estos podrían no funcionar si se actualiza el kernel.

#### <a name="sr-iov-enabled-vms"></a>Máquinas virtuales habilitadas para SR-IOV
  En el caso de las [máquinas virtuales compatibles con RDMA](../../sizes-hpc.md#rdma-capable-instances) habilitadas para SR-IOV, las imágenes de máquinas virtuales de [CentOS-HPC versión 7.6 o posterior](https://techcommunity.microsoft.com/t5/Azure-Compute/CentOS-HPC-VM-Image-for-SR-IOV-enabled-Azure-HPC-VMs/ba-p/665557) en Marketplace son la manera más fácil de comenzar. Estas imágenes de máquina virtual, que se han optimizado y cargado previamente con los controladores OFED para RDMA y varias bibliotecas de MPI y paquetes de informática científica más usados, son la manera más fácil de empezar.

  Vea un ejemplo de scripts usados en la creación de imágenes de máquina virtual de la versión 7.6 y posterior de CentOS-HPC a partir de una imagen base de Marketplace de CentOS base en el [repositorio azhpc-images](https://github.com/Azure/azhpc-images/tree/master/centos).

### <a name="rhelcentos-vm-images"></a>Imágenes de máquina virtual de RHEL/CentOS
Las imágenes de máquinas virtuales que no son HPC basadas en RHEL o CentOS de Marketplace pueden configurarse para usarse en [máquinas virtuales compatibles con RDMA](../../sizes-hpc.md#rdma-capable-instances) habilitadas para SR-IOV. Obtenga más información sobre cómo [habilitar InfiniBand](enable-infiniband.md) y [configurar MPI](setup-mpi.md) en las máquinas virtuales.

  Vea un ejemplo de scripts usados en la creación de imágenes de máquina virtual de la versión 7.6 y posterior de CentOS-HPC a partir de una imagen base de Marketplace de CentOS base en el [repositorio azhpc-images](https://github.com/Azure/azhpc-images/tree/master/centos).

### <a name="ubuntu-vm-images"></a>Imágenes de máquina virtual de Ubuntu
Las imágenes de máquina virtual de Ubuntu Server 16.04 LTS, 18.04 LTS y 20.04 LTS de Marketplace son compatibles con las [máquinas virtuales compatibles con SR-IOV y las no compatibles con SR-IOV](../../sizes-hpc.md#rdma-capable-instances). Obtenga más información sobre cómo [habilitar InfiniBand](enable-infiniband.md) y [configurar MPI](setup-mpi.md) en las máquinas virtuales.

  Vea un ejemplo de scripts que se pueden usar en la creación de las imágenes de máquina virtual de HPC basadas en Ubuntu 18.04 LTS en el [repositorio azhpc-images](https://github.com/Azure/azhpc-images/tree/master/ubuntu/ubuntu-18.x/ubuntu-18.04-hpc).

### <a name="suse-linux-enterprise-server-vm-images"></a>Imágenes de máquina virtual de SUSE Linux Enterprise Server
Las imágenes de máquina virtual de SLES 12 SP3 para HPC, SLES 12 SP3 para HPC (Premium), SLES 12 SP1 para HPC, SLES 12 SP1 para HPC (Premium), SLES 12 SP4 y SLES 15 de Marketplace son compatibles. Estas imágenes de máquina virtual se cargan previamente con los controladores de Network Direct para RDMA y la versión 5.1 de Intel MPI. Obtenga más información sobre cómo [configurar MPI](setup-mpi.md) en las máquinas virtuales.

## <a name="optimize-vms"></a>Optimización de máquinas virtuales

A continuación se muestran algunos valores de optimización opcionales para mejorar el rendimiento de la máquina virtual.

### <a name="update-lis"></a>Actualización de LIS

Si es necesario para la funcionalidad o el rendimiento, los controladores de [Linux Integration Services (LIS)](../../linux/endorsed-distros.md) pueden instalarse o actualizarse en distribuciones de SO compatibles, especialmente en la implementación mediante una imagen personalizada o una versión anterior de sistema operativo; por ejemplo, CentOS/RHEL 6.x u otra anterior a la 7.x.

```bash
wget https://aka.ms/lis
tar xzf lis
pushd LISISO
./upgrade.sh
```

### <a name="reclaim-memory"></a>Reclamación de memoria

Mejore el rendimiento reclamando automáticamente memoria, con el fin de evitar el acceso remoto a la memoria.

```bash
echo 1 >/proc/sys/vm/zone_reclaim_mode
```

Para hacer que se mantenga después de que se reinicie la máquina virtual:

```bash
echo "vm.zone_reclaim_mode = 1" >> /etc/sysctl.conf sysctl -p
```

### <a name="disable-firewall-and-selinux"></a>Deshabilitación del firewall y de SELinux

```bash
systemctl stop iptables.service
systemctl disable iptables.service
systemctl mask firewalld
systemctl stop firewalld.service
systemctl disable firewalld.service
iptables -nL
sed -i -e's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

### <a name="disable-cpupower"></a>Deshabilitación de cpupower

```bash
service cpupower status
if enabled, disable it:
service cpupower stop
sudo systemctl disable cpupower
```

### <a name="configure-walinuxagent"></a>Configuración de WALinuxAgent

```bash
sed -i -e 's/# OS.EnableRDMA=y/OS.EnableRDMA=y/g' /etc/waagent.conf
```
Si lo desea, WALinuxAgent puede estar deshabilitado como un paso anterior al trabajo y habilitado como un paso posterior al trabajo para disfrutar de la máxima disponibilidad de recursos de máquina virtual para la carga de trabajo de HPC.


## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre cómo [habilitar InfiniBand](enable-infiniband.md) en las máquinas virtuales de la serie [H](../../sizes-hpc.md) y [N](../../sizes-gpu.md) habilitadas para InfiniBand.
- Obtenga más información sobre la instalación de varias [bibliotecas de MPI compatibles](setup-mpi.md), así como sobre su configuración óptima de las VM.
- En los artículos [Introducción a las máquinas virtuales de la serie HB](hb-series-overview.md) e [Introducción a las máquinas virtuales de la serie HC](hc-series-overview.md), aprenderá a configurar de forma óptima las cargas de trabajo para mejorar el rendimiento y la escalabilidad.
- En los [blogs de la comunidad de Azure Compute Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute), encontrará los anuncios más recientes y algunos ejemplos y resultados de HPC.
- Si desea una visión general de la arquitectura de la ejecución de cargas de trabajo de HPC, consulte [Informática de alto rendimiento (HPC) en Azure](/azure/architecture/topics/high-performance-computing/).
