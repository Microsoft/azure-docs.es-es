---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 05/14/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 30df480eb314594cbc4d949302aff11e6d764b6f
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65815493"
---
## <a name="premium-ssd"></a>SSD Premium

Los discos SSD Premium de Azure ofrecen soporte de disco de alto rendimiento y latencia baja para máquinas virtuales (VM) con cargas de trabajo con uso intensivo de entrada/salida (E/S). Para aprovechar la ventaja de la velocidad y el rendimiento de discos de Premium Storage, puede migrar los discos de VM existentes a SSD Premium. Los discos SSD Premium son adecuados para aplicaciones de producción críticas. Solo se puede usar SSD Premium con series de máquinas virtuales que son compatibles con el almacenamiento premium.

Para más información sobre los tipos de máquinas virtuales individuales y los tamaños en Azure para Windows, incluidos los tamaños están premium compatible con almacenamiento de información, consulte [tamaños de máquina virtual de Windows](../articles/virtual-machines/windows/sizes.md). Para más información sobre tamaños de Azure y tipos de máquinas virtuales individuales para Linux, incluidos los tamaños están premium compatible con almacenamiento de información, consulte [tamaños de VM de Linux](../articles/virtual-machines/linux/sizes.md).

### <a name="disk-size"></a>Tamaño del disco
[!INCLUDE [disk-storage-premium-ssd-sizes](disk-storage-premium-ssd-sizes.md)]

Cuando se aprovisiona un disco de Premium Storage, a diferencia de Standard Storage, se garantizan la capacidad, las E/S por segundo y el rendimiento del mismo. Por ejemplo, si crea un disco P50, Azure aprovisiona una capacidad de almacenamiento de 4095 GB, 7500 E/S por segundo y un rendimiento de 250 MB/s para él. La aplicación puede usar toda la capacidad y el rendimiento o parte de ellos. Los discos SSD premium están diseñados para proporcionar rendimiento de destino el 99,9 % del tiempo.

### <a name="transactions"></a>Transacciones

Para SSD premium, cada operación de E/S menor o igual a 256 KiB de rendimiento se considera una sola operación de E/S. Las operaciones de E/S mayores que 256 KiB de rendimiento se consideran múltiples entradas y salidas del tamaño de 256 KiB.

## <a name="standard-ssd"></a>SSD estándar

Los discos SSD estándar de Azure son una opción de almacenamiento rentable, optimizada para cargas de trabajo que necesitan un rendimiento constante en niveles inferiores de IOPS. Los discos SSD estándar ofrecen una buena experiencia de nivel de entrada para aquellos que desean pasarse a la nube, en especial si tiene problemas con la variación de las cargas de trabajo que se ejecutan en las soluciones de disco duro locales. En comparación con unidades de disco duro estándares, SSDs estándares ofrecen una mejor disponibilidad, coherencia, confiabilidad y la latencia. Los discos SSD estándar son convenientes para servidores web, servidores de aplicaciones con IOPS bajas, aplicaciones empresariales de poco uso y cargas de trabajo de desarrollo/pruebas. Como estándar HDD, SSD estándar está disponible en todas las máquinas virtuales de Azure.

### <a name="disk-size"></a>Tamaño del disco
[!INCLUDE [disk-storage-standard-ssd-sizes](disk-storage-standard-ssd-sizes.md)]

SSDs estándares están diseñadas para proporcionar las latencias de milisegundos para la mayoría de las operaciones de E/S y para entregar la IOPS y el rendimiento hasta los límites descritos en la anterior tabla 99% del tiempo. IOPS y el rendimiento real pueden variar a veces, según los patrones de tráfico. Los discos SSD estándar proporcionarán un rendimiento más coherente que los discos HDD, con una latencia menor.

### <a name="transactions"></a>Transacciones

Para SSD estándar, cada operación de E/S menor o igual a 256 KiB de rendimiento se considera una sola operación de E/S. Las operaciones de E/S mayores que 256 KiB de rendimiento se consideran múltiples entradas y salidas del tamaño de 256 KiB. Estas transacciones tienen un impacto en la facturación.

## <a name="standard-hdd"></a>HDD estándar

Los discos HDD estándar de Azure ofrecen compatibilidad de discos confiable y de bajo coste para las máquinas virtuales que ejecutan cargas de trabajo que no tienen en cuenta la latencia. También admite blobs, tablas, colas y archivos. Con Standard Storage, los datos se almacenan en unidades de disco duro (HDD). Cuando se trabaja con máquinas virtuales, se pueden usar discos SSD y HDD estándar para escenarios de desarrollo/pruebas y para cargas de trabajo menos críticas. Unidades de disco duro estándares están disponibles en todas las regiones de Azure y se pueden utilizar con todas las máquinas virtuales de Azure.

### <a name="disk-size"></a>Tamaño del disco
[!INCLUDE [disk-storage-standard-hdd-sizes](disk-storage-standard-hdd-sizes.md)]

### <a name="transactions"></a>Transacciones

Para unidades de disco duro estándar, cada operación de E/S se considera como una única transacción, independientemente del tamaño de E/S. Estas transacciones tienen un impacto en la facturación.

## <a name="billing"></a>Facturación

Al usar Managed Disks, se aplican las siguientes consideraciones de facturación:

- Tipo de disco
- Tamaño del disco administrado
- Instantáneas
- Transferencias de datos de salida
- Número de transacciones

**Tamaño del disco administrado**: los discos administrados se facturan por el tamaño aprovisionado. Azure asigna el tamaño aprovisionado (redondeado al alza) a la oferta de tamaño de disco más cercana. Para obtener detalles sobre los tamaños de disco ofrecidos, consulte las tablas anteriores. Cada disco se asigna a una oferta de tamaño de disco aprovisionado compatible y se factura según corresponda. Por ejemplo, si ha aprovisionado un disco SSD estándar de 200 GiB, se asigna a la oferta de tamaño de disco E15 (256 GiB). La facturación de cualquier disco aprovisionado se prorratea cada hora con el precio mensual de la oferta de Premium Storage. Por ejemplo, si ha aprovisionado un disco E10 y lo ha eliminado después de 20 horas, se factura la oferta E10 prorrateada a 20 horas. Esto es así independientemente de la cantidad de datos que se escriba en el disco.

**Instantáneas**: Las instantáneas se facturan según el tamaño utilizado. Por ejemplo, si crea una instantánea de un disco administrado con capacidad aprovisionada de 64 GiB y el tamaño de datos usado real es de 10 GiB, solo se le cobra por el tamaño de datos usado de 10 GiB.