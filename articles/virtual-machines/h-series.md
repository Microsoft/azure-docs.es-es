---
title: 'Serie H: Azure Virtual Machines'
description: Especificaciones de las máquinas virtuales de la serie H.
author: ju-shim
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 08/01/2020
ms.author: amverma
ms.reviewer: jushiman
ms.openlocfilehash: 797a036b9cf2e77dfbcdf8dc7596179c4673e6a6
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87513747"
---
# <a name="h-series"></a>Serie H

Las máquinas virtuales de la serie H están optimizadas para aplicaciones basadas en frecuencias alta de CPU o requisitos de gran cantidad de memoria por núcleo. Las máquinas virtuales de la serie H cuentan con 8 o 16 núcleos de procesador Intel Xeon E5 2667 v3, hasta 14 GB de RAM por núcleo de CPU y no tienen hyperthreading. Las máquinas de la serie H cuentan con una interconexión Mellanox FDR InfiniBand de 56 Gb/s en una configuración de árbol de FAT sin bloqueo para proporcionar un rendimiento de RDMA consistente. Las máquinas virtuales de la serie H admiten Intel MPI 5.x y MS-MPI.

ACU: 290-300

Premium Storage:  No compatible

Almacenamiento en caché de Premium Storage:  No compatible

Migración en vivo: No compatible

Actualizaciones con conservación de memoria: No compatible

| Size | vCPU | Procesador | Memoria (GB) | Ancho de banda de memoria, en GB/s | Frecuencia de CPU base (GHz) | Frecuencia de todos los núcleos (GHz, pico) | Frecuencia de cada núcleo (GHz, pico) | Rendimiento de RDMA (GB/s) | Compatibilidad con MPI | Almacenamiento temporal (GB) | Discos de datos máx. | Rendimiento máximo de discos E/S | NIC Ethernet máx. |
| --- | --- |--- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_H8   | 8  | Intel Xeon E5 2667 v3 | 56 | 40 | 3.2 | 3.3 | 3.6 | - | Intel 5.x, MS-MPI | 1000 | 32 | 32 x 500 | 2 |
| Standard_H16  | 16 | Intel Xeon E5 2667 v3 | 112 | 80 | 3.2 | 3.3 | 3.6 | - | Intel 5.x, MS-MPI | 2000 | 64 | 64 x 500 | 4 |
| Standard_H8m  | 8  | Intel Xeon E5 2667 v3 | 112 | 40 | 3.2 | 3.3 | 3.6 | - | Intel 5.x, MS-MPI | 1000 | 32 | 32 x 500 | 2 |
| Standard_H16m | 16 | Intel Xeon E5 2667 v3 | 224 | 80 | 3.2 | 3.3 | 3.6 | - | Intel 5.x, MS-MPI | 2000 | 64 | 64 x 500 | 4 |
| Standard_H16r <sup>1</sup>  | 16 | Intel Xeon E5 2667 v3 | 112 | 80 | 3.2 | 3.3 | 3.6 | 56 | Intel 5.x, MS-MPI | 2000 | 64 | 64 x 500 | 4 |
| Standard_H16mr <sup>1</sup> | 16 | Intel Xeon E5 2667 v3 | 224 | 80 | 3.2 | 3.3 | 3.6 | 56 | Intel 5.x, MS-MPI | 2000 | 64 | 64 x 500 | 4 |

<sup>1</sup> Para las aplicaciones MPI, la red de back-end RDMA dedicada está habilitada por la red InfiniBand FDR.

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]


## <a name="supported-os-images-linux"></a>Imágenes de sistema operativo compatibles (Linux)
 
Azure Marketplace tiene muchas distribuciones de Linux que admiten la conectividad de RDMA:
  
* **HPC basada en CentOS**: en las máquinas virtuales no habilitadas para SR-IOV, se admite una HPC basada en CentOS versión 6.5 o posterior, hasta la versión 7.5. En las máquinas virtuales de la serie H, se recomienda usar las versiones 7.1 a 7.5. Los controladores RDMA e Intel MPI 5.1 están instalados en la máquina virtual.
  En las máquinas virtuales habilitadas para SR-IOV, HPC 7.6 basada en CentOS viene optimizada y precargada con los controladores de RDMA y con diversos paquetes de MPI instalados.
  En otras imágenes de máquina virtual de RHEL/CentOS, agregue la extensión InfiniBandLinux para habilitar InfiniBand. Esta extensión de máquina virtual de Linux instala controladores Mellanox OFED (en máquinas virtuales de SR-IOV) para la conectividad RDMA. El siguiente cmdlet de PowerShell instala la versión más reciente (versión 1.0) de la extensión InfiniBandDriverLinux en una máquina virtual compatible con RDMA existente. La máquina virtual compatible con RDMA se denomina *myVM* y se implementa en el grupo de recursos denominado *myResourceGroup* en la región *Oeste de EE. UU.* del siguiente modo:

  ```powershell
  Set-AzVMExtension -ResourceGroupName "myResourceGroup" -Location "westus" -VMName "myVM" -ExtensionName "InfiniBandDriverLinux" -Publisher "Microsoft.HpcCompute" -Type "InfiniBandDriverLinux" -TypeHandlerVersion "1.0"
  ```
  Como alternativa, se pueden incluir extensiones de máquina virtual en las plantillas de Azure Resource Manager para facilitar la implementación, con el siguiente elemento JSON:
  ```json
  "properties":{
  "publisher": "Microsoft.HpcCompute",
  "type": "InfiniBandDriverLinux",
  "typeHandlerVersion": "1.0",
  } 
  ```
  
  El comando siguiente instala la versión más reciente de la extensión InfiniBandDriverLinux, la 1.0, en todas las máquinas virtuales compatibles con RDMA de un conjunto de escalado de máquinas virtuales existente denominado *myVMSS* implementado en el grupo de recursos denominado *myResourceGroup*:
  ```powershell
  $VMSS = Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myVMSS"
  Add-AzVmssExtension -VirtualMachineScaleSet $VMSS -Name "InfiniBandDriverLinux" -Publisher "Microsoft.HpcCompute" -Type "InfiniBandDriverLinux" -TypeHandlerVersion "1.0"
  Update-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "MyVMSS" -VirtualMachineScaleSet $VMSS
  Update-AzVmssInstance -ResourceGroupName "myResourceGroup" -VMScaleSetName "myVMSS" -InstanceId "*"
  ```
  
  > [!NOTE]
  > En las imágenes de HPC basadas en CentOS, las actualizaciones del núcleo están deshabilitadas en el archivo de configuración **yum** . Esto se debe a que los controladores Linux RDMA se distribuyen en forma de paquete RPM y sus actualizaciones de estos podrían no funcionar si se actualiza el kernel.
  >
  

* **SUSE Linux Enterprise Server**: SLES 12 SP3 para HPC, SLES 12 SP3 para HPC (Premium), SLES 12 SP1 para HPC, SLES 12 SP1 para HPC (Premium), SLES 12 SP4 y SLES 15. Los controladores RDMA se instalan y se distribuyen los paquetes Intel MPI en la máquina virtual. Instale MPI mediante la ejecución del comando siguiente:

  ```bash
  sudo rpm -v -i --nodeps /opt/intelMPI/intel_mpi_packages/*.rpm
  ```
  
* **Ubuntu**: Ubuntu Server 16.04 LTS, 18.04 LTS. Configure controladores RDMA en la máquina virtual y regístrese en Intel para descargar Intel MPI:

  [!INCLUDE [virtual-machines-common-ubuntu-rdma](../../includes/virtual-machines-common-ubuntu-rdma.md)]  

  Para obtener más detalles sobre cómo habilitar InfiniBand y cómo configurar MPI, vea [Habilitar InfiniBand](./workloads/hpc/enable-infiniband.md).

## <a name="other-sizes"></a>Otros tamaños

- [Uso general](sizes-general.md)
- [Memoria optimizada](sizes-memory.md)
- [Almacenamiento optimizado](sizes-storage.md)
- [GPU optimizada](sizes-gpu.md)
- [Proceso de alto rendimiento](sizes-hpc.md)
- [Generaciones anteriores](sizes-previous-gen.md)

## <a name="next-steps"></a>Pasos siguientes

- Puede encontrar más información sobre la optimización de aplicaciones de HPC para Azure y algunos ejemplos en [Cargas de trabajo de HPC](./workloads/hpc/overview.md).
- En los [blogs de la comunidad de Azure Compute Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute), encontrará los anuncios más recientes y algunos ejemplos y resultados de HPC.
- Si desea una visión general de la arquitectura de la ejecución de cargas de trabajo de HPC, consulte [Informática de alto rendimiento (HPC) en Azure](/azure/architecture/topics/high-performance-computing/).
- Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.
