---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 01/22/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: d2daafa6bf5f9a28ad2b61a97e7a8bd2246ae18d
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65538387"
---
# <a name="what-disk-types-are-available-in-azure"></a>¿Qué tipos de disco están disponibles en Azure?

Actualmente, Azure Managed Disks ofrece cuatro tipos de discos, tres de los cuales están disponibles con carácter general (GA) y otro que está actualmente en versión preliminar. Cada uno de estos cuatro tipos de disco tiene sus propios escenarios de clientes de destino apropiados.

## <a name="disk-comparison"></a>Comparación de discos

En la tabla siguiente se ofrece una comparación entre las unidades de estado sólido (SSD) (versión preliminar), SSD premium, SSD estándar y unidades de disco duro (HDD) estándar para Managed Disks, a fin de ayudar a decidir qué opción usar.

|   | SSD Ultra (versión preliminar)   | SSD Premium   | SSD estándar   | HDD estándar   |
|---------|---------|---------|---------|---------|
|Tipo de disco   |SSD   |SSD   |SSD   |HDD   |
|Escenario   |Cargas de trabajo con uso intensivo de E/S, como SAP HANA, bases de dato de capa superior (por ejemplo, SQL y Oracle) y otras cargas de trabajo con muchas transacciones.   |Cargas de trabajo confidenciales de producción y rendimiento   |Servidores web, aplicaciones empresariales poco utilizadas y desarrollo y pruebas   |Copia de seguridad, no crítico, acceso poco frecuente   |
|Tamaño del disco   |65 536 gibibytes (GiB) (versión preliminar)   |32 767 GiB    |32 767 GiB   |32 767 GiB   |
|Rendimiento máx.   |2000 MiB/s (versión preliminar)   |900 MiB/s   |750 MiB/s   |500 MiB/s   |
|E/S máxima por segundo   |160 000 (versión preliminar)   |20.000   |6,000   |2.000   |

## <a name="ultra-ssd-preview"></a>SSD Ultra (versión preliminar)

Azure SSD Ultra (versión preliminar) le ofrece un alto rendimiento, IOPS elevadas y un almacenamiento en disco consistente y de baja latencia para máquinas virtuales IaaS de Azure. Algunas ventajas adicionales de SSD Ultra incluyen la capacidad de cambiar dinámicamente el rendimiento del disco junto con sus cargas de trabajo sin tener que reiniciar las máquinas virtuales. Asimismo, SSD Ultra es adecuado para cargas de trabajo con grandes cantidades de datos, como SAP HANA, bases de datos de nivel superior y cargas de trabajo que admitan muchas transacciones. SSD Ultra solo puede utilizarse como discos de datos. Por ello, recomendamos usar los discos SSD Premium como discos de sistema operativo.

### <a name="performance"></a>Rendimiento

Cuando aprovisione un disco ultra, puede configurar de forma independiente la capacidad y el rendimiento del disco. Los discos SSD Ultra vienen en varios tamaños fijos que van desde los 4 GiB hasta los 64 TiB, y cuentan con un modelo de configuración de rendimiento flexible que le permite configurar las unidades IOPS y el rendimiento de forma independiente.

Estas son algunas capacidades clave de los discos Ultra SSD:

- Capacidad de disco: Intervalos de capacidad de SSD Ultra desde 4 GiB hasta 64 TiB.
- IOPS de disco: los dispositivos SSD Ultra admiten límites de IOPS de 300 IOPS/GiB y hasta un máximo de 160 K IOPS por disco. Para recuperar la tasa de unidades IOPS que aprovisionó, asegúrese de que la cantidad de IOPS de disco seleccionadas sea menor que la cantidad de IOPS de la máquina virtual. El mínimo de IOPS de disco es de 100 IOPS.
- Rendimiento del disco: con los discos SSD Ultra, el límite de rendimiento de un solo disco es de 256 KiB/s por cada IOPS aprovisionada, y hasta 2000 MBps como máximo por disco (donde MBps = 10^6 bytes por segundo). El rendimiento de disco mínimo es de 1 MiB.
- SSDs Ultra admiten el ajuste de los atributos de rendimiento de disco (IOPS y rendimiento) en tiempo de ejecución sin desconectar el disco de la máquina virtual. Cuando se ha enviado una operación de cambio de tamaño del rendimiento del disco en un disco, este cambio puede tardar hasta una hora en surtir efecto.

### <a name="disk-size"></a>Tamaño del disco

|Tamaño de disco (GiB)  |Capacidades de IOPS  |Capacidad de rendimiento (MBps)  |
|---------|---------|---------|
|4     |1,200         |300         |
|8     |2,400         |600         |
|16     |4,800         |1,200         |
|32     |9600         |2.000         |
|64     |19 200         |2.000         |
|128     |38 400         |2.000         |
|256     |76 800         |2.000         |
|512     |80 000         |2.000         |
|1024 - 65 536 (los tamaños de este intervalo aumentan en incrementos de 1 TiB)     |160 000         |2.000         |

### <a name="transactions"></a>Transacciones

Para SSD ultra, cada operación de E/S menor o igual a 256 KiB de rendimiento se considera una sola operación de E/S. Las operaciones de E/S mayores que 256 KiB de rendimiento se consideran múltiples entradas y salidas del tamaño de 256 KiB.

### <a name="preview-scope-and-limitations"></a>Ámbito y limitaciones de la versión preliminar

Durante la versión preliminar, los discos SSD Ultra:

- Se admiten en el Este de EE. UU. 2, en una única zona de disponibilidad.  
- Solo se podrán usar con las zonas de disponibilidad (los conjuntos de disponibilidad y las implementaciones de máquinas virtuales únicas fuera de las zonas no tendrán la capacidad de adjuntar un disco Ultra).
- Solo son compatibles con las máquinas virtuales ES/DS v3.
- Solo están disponibles como discos de datos y solo admiten el tamaño de sector físico 4k.  
- Solo pueden crearse como discos vacíos.  
- Actualmente solo se pueden implementar mediante plantillas de Azure Resource Manager, CLI y SDK de Python.
- No admitirán todavía instantáneas de disco, imágenes de máquinas virtuales, conjuntos de disponibilidad, Virtual Machine Scale Sets ni Azure Disk Encryption.
- No admitirán todavía la integración con Azure Backup o Azure Site Recovery.
- Tal como sucede con la  [mayoría de las versiones preliminares](https://azure.microsoft.com/support/legal/preview-supplemental-terms/), esta característica no debe usarse para cargas de trabajo de producción hasta que se lance la disponibilidad general (GA).
