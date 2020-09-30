---
title: Aumento del rendimiento de los discos administrados de Azure
description: Conozca los niveles de rendimiento de los discos administrados y cómo actualizarlos.
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 09/22/2020
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: aa188babf56d4a825059fe6103e2e07745eb134f
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90974128"
---
# <a name="performance-tiers-for-managed-disks-preview"></a>Niveles de rendimiento de discos administrados (versión preliminar)

Actualmente, Azure Disk Storage ofrece funcionalidades de expansión integradas para lograr un mayor rendimiento en el control del tráfico inesperado a corto plazo. Los discos SSD Premium cuentan con la flexibilidad de aumentar su rendimiento sin aumentar el tamaño real, lo que le permite satisfacer las necesidades de rendimiento de la carga de trabajo y reducir los costos, esta característica está actualmente en versión preliminar. Estas prestaciones resultan muy convenientes con eventos que requieren temporalmente un nivel de rendimiento más alto de forma constante, como las compras navideñas, las pruebas de rendimiento o la ejecución de un entorno de aprendizaje. Para administrar estos eventos, puede seleccionar un nivel de rendimiento superior, mientras sea necesario, y volver al nivel original cuando ya no se necesite rendimiento adicional.

## <a name="how-it-works"></a>Cómo funciona

La primera vez que implementa o aprovisiona un disco, el nivel de rendimiento de línea de base para ese disco se establece en función del tamaño del disco aprovisionado. Se puede seleccionar un nivel de rendimiento superior para satisfacer una mayor demanda y, cuando ya no sea necesario, puede volver al nivel de rendimiento de línea de base inicial. Por ejemplo, si aprovisiona un disco P10 (128 GiB), el nivel de rendimiento de línea de base se establece como P10 (500 IOPS y 100 MB/s). Puede actualizar el nivel para que coincida con el rendimiento de P50 (7500 IOPS y 250 MB/s) sin aumentar el tamaño del disco y volver a P10 cuando ya no se necesite ese mayor rendimiento.

| Tamaño del disco | Rendimiento de línea de base | Se puede actualizar a |
|----------------|-----|-------------------------------------|
| 4 GiB | P1 | P2, P3, P4, P6, P10, P15, P20, P30, P40, P50 |
| 8 GiB | P2 | P3, P4, P6, P10, P15, P20, P30, P40, P50 |
| 16 GiB | P3 | P4, P6, P10, P15, P20, P30, P40, P50 | 
| 32 GiB | P4 | P6, P10, P15, P20, P30, P40, P50 |
| 64 GiB | P6 | P10, P15, P20, P30, P40, P50 |
| 128 GB | P10 | P15, P20, P30, P40, P50 |
| 256 GiB | P15 | P20, P30, P40, P50 |
| 512 GB | P20 | P30, P40, P50 |
| 1 TiB | P30 | P40, P50 |
| 2 TiB | P40 | P50 |
| 4 TiB | P50 | None |
| 8 TiB | P60 |  P70, P80 |
| 16 TiB | P70 | P80 |
| 32 TiB | P80 | None |

## <a name="restrictions"></a>Restricciones

- Actualmente solo se admite para los discos SSD Premium.
- Antes de cambiar de nivel, los discos deben desasociarse de las máquinas virtuales en ejecución.
- El uso de los niveles de rendimiento P60, P70 y P80 está restringido a los discos de 4096 GiB o superior.

## <a name="regional-availability"></a>Disponibilidad regional

El ajuste del nivel de rendimiento de un disco administrado solo está disponible actualmente para los SSD Premium en las siguientes regiones:

- Centro-Oeste de EE. UU. 
- Este de EE. UU. 2 
- Oeste de Europa
- Este de Australia 
- Sudeste de Australia 
- Sur de la India

## <a name="createupdate-a-data-disk-with-a-tier-higher-than-the-baseline-tier"></a>Creación o actualización de un disco de datos con un nivel superior al nivel de línea de base

1. Creación de un disco de datos vacío con un nivel superior al nivel de línea de base o actualización del nivel de un disco a uno superior al de línea de base mediante la plantilla de ejemplo [CreateUpdateDataDiskWithTier.json](https://github.com/Azure/azure-managed-disks-performance-tiers/blob/main/CreateUpdateDataDiskWithTier.json)

     ```cli
     subscriptionId=<yourSubscriptionIDHere>
     resourceGroupName=<yourResourceGroupNameHere>
     diskName=<yourDiskNameHere>
     diskSize=<yourDiskSizeHere>
     performanceTier=<yourDesiredPerformanceTier>
     region=<yourRegionHere>
    
     az login
    
     az account set --subscription $subscriptionId
    
     az group deployment create -g $resourceGroupName \
     --template-uri "https://raw.githubusercontent.com/Azure/azure-managed-disks-performance-tiers/main/CreateUpdateDataDiskWithTier.json" \
     --parameters "region=$region" "diskName=$diskName" "performanceTier=$performanceTier" "dataDiskSizeInGb=$diskSize"
     ```

1. Confirmación del nivel del disco

    ```cli
    az resource show -n $diskName -g $resourceGroupName --namespace Microsoft.Compute --resource-type disks --api-version 2020-06-30 --query [properties.tier] -o tsv
     ```

## <a name="createupdate-an-os-disk-with-a-tier-higher-than-the-baseline-tier"></a>Creación o actualización de un disco del sistema operativo con un nivel superior al nivel de línea de base

1. Creación de un disco del sistema operativo a partir de una imagen del marketplace o actualización del nivel de un disco del sistema operativo a uno superior al de línea de base mediante la plantilla de ejemplo [CreateUpdateOSDiskWithTier.json](https://github.com/Azure/azure-managed-disks-performance-tiers/blob/main/CreateUpdateOSDiskWithTier.json)

     ```cli
     resourceGroupName=<yourResourceGroupNameHere>
     diskName=<yourDiskNameHere>
     performanceTier=<yourDesiredPerformanceTier>
     region=<yourRegionHere>
    
     az group deployment create -g $resourceGroupName \
     --template-uri "https://raw.githubusercontent.com/Azure/azure-managed-disks-performance-tiers/main/CreateUpdateOSDiskWithTier.json" \
     --parameters "region=$region" "diskName=$diskName" "performanceTier=$performanceTier"
     ```
 
 1. Confirmación del nivel del disco
 
     ```cli
     az resource show -n $diskName -g $resourceGroupName --namespace Microsoft.Compute --resource-type disks --api-version 2020-06-30 --query [properties.tier] -o tsv
     ```

## <a name="next-steps"></a>Pasos siguientes

Si debe cambiar el tamaño de un disco para aprovechar las ventajas de los niveles de rendimiento mayores, consulte nuestros artículos sobre el tema:

- [Expansión de discos duros virtuales en una VM Linux con la CLI de Azure](linux/expand-disks.md)
- [Expansión de un disco administrado asociado a una máquina virtual de Windows](windows/expand-os-disk.md)