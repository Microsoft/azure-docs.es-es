---
title: 'Selección de un tipo de disco para máquinas virtuales IaaS de Azure: discos administrados'
description: Obtenga información sobre los tipos de disco de Azure disponibles para las máquinas virtuales, incluidos los discos ultra, SSD Premium, SSD estándar y HDD estándar.
author: roygara
ms.author: rogarana
ms.date: 06/03/2020
ms.topic: conceptual
ms.service: virtual-machines
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: 53089fa42c536cbdc59865f80f63a77c76720e2c
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88752015"
---
# <a name="what-disk-types-are-available-in-azure"></a>¿Qué tipos de disco están disponibles en Azure?

Actualmente, Azure Managed Disks ofrece cuatro tipos de discos. Cada uno estos tipos está pensado para escenarios de clientes específicos.

## <a name="disk-comparison"></a>Comparación de discos

En la tabla siguiente se proporciona una comparación entre los discos Ultra, las unidades de estado sólido (SSD) prémium, los discos SSD estándar y las unidades de disco duro (HDD) estándar para Managed Disks, a fin de ayudarle a decidir qué opción debe usar.

| Detail | Disco Ultra | SSD Premium | SSD estándar | HDD estándar |
| ------ | ---------- | ----------- | ------------ | ------------ |
|Tipo de disco   |SSD   |SSD   |SSD   |HDD   |
|Escenario   |Cargas de trabajo de uso intensivo de E/S, como [SAP HANA](workloads/sap/hana-vm-operations-storage.md), bases de datos de nivel superior (por ejemplo, SQL y Oracle) y otras cargas de trabajo con muchas transacciones.   |Cargas de trabajo confidenciales de producción y rendimiento   |Servidores web, aplicaciones empresariales poco utilizadas y desarrollo y pruebas   |Copia de seguridad, no crítico, acceso poco frecuente   |
|Tamaño máximo del disco   |65 536 gibibyte (GiB)    |32 767 GiB    |32 767 GiB   |32 767 GiB   |
|Rendimiento máx.   |2000 MB/s    |900 MB/s   |750 MB/s   |500 MB/s   |
|IOPS máx.   |160 000    |20.000   |6,000   |2\.000   |

## <a name="ultra-disk"></a>Disco Ultra

Los discos Ultra de Azure ofrecen un alto rendimiento, un número elevado de IOPS y un almacenamiento en disco coherente y de baja latencia para máquinas virtuales IaaS de Azure. Algunas ventajas adicionales de los discos Ultra incluyen la capacidad de cambiar dinámicamente el rendimiento del disco junto con sus cargas de trabajo sin tener que reiniciar las máquinas virtuales. Además, los discos Ultra son adecuados para cargas de trabajo con grandes cantidades de datos, como SAP HANA, bases de datos de nivel superior y cargas de trabajo que admitan muchas transacciones. Los discos Ultra solo se pueden utilizar como discos de datos. Por ello, recomendamos usar los discos SSD Premium como discos de sistema operativo.

### <a name="performance"></a>Rendimiento

Cuando aprovisione un disco ultra, puede configurar de forma independiente la capacidad y el rendimiento del disco. Los discos Ultra vienen en varios tamaños fijos que van desde los 4 GiB hasta los 64 TiB y cuentan con un modelo de configuración de rendimiento flexible que le permite configurar las unidades IOPS y el rendimiento de forma independiente.

Estas son algunas funcionalidades clave de los discos Ultra:

- Capacidad de disco: intervalos de capacidad de discos Ultra desde 4 GiB hasta 64 TiB.
- IOPS de disco: los dispositivos Ultra admiten límites de IOPS de 300 IOPS/GiB y hasta un máximo de 160 K IOPS por disco. Para recuperar la tasa de unidades IOPS que aprovisionó, asegúrese de que la cantidad de IOPS de disco seleccionadas sea menor que el límite de IOPS de la máquina virtual. El valor mínimo garantizado de IOPS por disco es 2 IOPS/GiB, con una base de referencia general mínima de 100 IOPS. Por ejemplo, si tuviera un disco Ultra de 4 GiB, tendrá un mínimo de 100 IOPS, en lugar de ocho IOPS.
- Rendimiento del disco: con los discos Ultra, el límite de rendimiento de un solo disco es de 256 KiB/s por cada IOPS aprovisionada, y hasta 2000 MBps como máximo por disco (donde MBps = 10^6 bytes por segundo). El rendimiento mínimo garantizado por disco es 4 KiB/s por cada IOPS aprovisionada, con una base de referencia total como mínima de 1 MBps.
- Los discos Ultra admiten el ajuste de los atributos de rendimiento del disco (IOPS y rendimiento) en tiempo de ejecución sin necesidad de desasociar el disco de la máquina virtual. Cuando se ha enviado una operación de cambio de tamaño del rendimiento del disco en un disco, este cambio puede tardar hasta una hora en surtir efecto. Hay un límite de cuatro operaciones de cambio de tamaño de rendimiento en un período de 24 horas. Se puede producir un error en la operación de ajuste de tamaño del rendimiento debido a la falta de capacidad de ancho de banda de rendimiento.

### <a name="disk-size"></a>Tamaño del disco

|Tamaño de disco (GiB)  |Capacidad de IOPS  |Capacidad de rendimiento (MBps)  |
|---------|---------|---------|
|4     |1,200         |300         |
|8     |2,400         |600         |
|16     |4,800         |1,200         |
|32     |9 600         |2\.000         |
|64     |19 200         |2\.000         |
|128     |38 400         |2\.000         |
|256     |76 800         |2\.000         |
|512     |80 000         |2\.000         |
|1024 - 65 536 (los tamaños de este intervalo aumentan en incrementos de 1 TiB)     |160 000         |2\.000         |

### <a name="ga-scope-and-limitations"></a>Ámbito y limitaciones de la disponibilidad general

[!INCLUDE [managed-disks-ultra-disks-GA-scope-and-limitations](../../includes/managed-disks-ultra-disks-GA-scope-and-limitations.md)]


Si quiere empezar con un disco ultra, vea nuestro artículo sobre el tema: [Uso de discos Ultra de Azure](disks-enable-ultra-ssd.md).

## <a name="premium-ssd"></a>SSD Premium

Los discos SSD Premium de Azure ofrecen soporte de disco de alto rendimiento y latencia baja para máquinas virtuales (VM) con cargas de trabajo con uso intensivo de entrada/salida (E/S). Para aprovechar la ventaja de la velocidad y el rendimiento de discos de Premium Storage, puede migrar los discos de VM existentes a SSD Premium. Los discos SSD Premium son adecuados para aplicaciones de producción críticas. Los discos SSD Premium solo se pueden usar con series de máquinas virtuales que sean compatibles con el almacenamiento premium.

Para más información sobre los tipos y tamaños de máquinas virtuales individuales de Azure para Windows o Linux, incluidos los tamaños que son compatibles con almacenamiento premium, consulte [Tamaños de las máquinas virtuales en Azure](sizes.md). Para más información sobre los tipos y tamaños de máquinas virtuales individuales de Azure para Linux, incluidos los tamaños que son compatibles con almacenamiento premium, consulte [Tamaños de las máquinas virtuales en Azure](sizes.md). En cualquiera de estos artículos, debe comprobar cada artículo de tamaño de VM individual para determinar si es compatible con almacenamiento Premium.

### <a name="disk-size"></a>Tamaño del disco
[!INCLUDE [disk-storage-premium-ssd-sizes](../../includes/disk-storage-premium-ssd-sizes.md)]

Cuando se aprovisiona un disco de Premium Storage, a diferencia de Standard Storage, se garantizan la capacidad, las E/S por segundo y el rendimiento del mismo. Por ejemplo, si crea un disco P50, Azure aprovisiona una capacidad de almacenamiento de 4095 GB, 7500 E/S por segundo y un rendimiento de 250 MB/s para él. La aplicación puede usar toda la capacidad y el rendimiento o parte de ellos. Los discos SSD Premium están diseñados para proporcionar bajas latencias inferiores a 10 milisegundos y un IOPS y rendimiento que se describen en la tabla anterior como del 99,9 % del tiempo.

## <a name="bursting"></a>Creación de ráfagas

Los tamaños de SSD Premium más pequeños que P30 ahora ofrecen una ráfaga de disco y pueden aumentar los IOPS por disco en hasta 3500 y el ancho de banda en hasta 170 Mbps. La creación de ráfagas está automatizada y funciona de acuerdo con un sistema de crédito. Los créditos se acumulan automáticamente en un depósito de ráfagas cuando el tráfico del disco está por debajo del objetivo de rendimiento aprovisionado y los créditos se consumen automáticamente cuando el tráfico supera el destino, hasta el límite máximo de ráfagas. El límite máximo de ráfagas define el límite superior de IOPS y Bandwidth del disco, incluso si tiene créditos de ráfagas para consumir. La ráfaga de disco proporciona una tolerancia mejorada a cambios imprevisibles en los patrones de E/S. Puede aprovecharla mejor en el arranque del disco del sistema operativo y las aplicaciones con tráfico de picos.    

La compatibilidad con ráfagas de discos se habilitará en las nuevas implementaciones de los tamaños de disco aplicables de forma predeterminada, sin necesidad de que intervenga el usuario. En el caso de los discos existentes de tamaños aplicables, puede habilitar la ráfaga con cualquiera de estas dos opciones: desasociar y volver a adjuntar el disco, o detener y reiniciar la VM conectada. Todos los tamaños de disco aplicables de la ráfaga comenzarán con un cubo de crédito de ráfaga completo cuando el disco esté conectado a una máquina virtual que admita una duración máxima en el límite máximo de ráfaga de 30 minutos. Para obtener más información sobre el aumento del trabajo de ráfagas en discos de Azure, consulte [Creación de ráfagas SSD Premium](linux/disk-bursting.md). 

### <a name="transactions"></a>Transacciones

Para los discos SSD Premium, cada operación de E/S menor o igual a 256 KiB de rendimiento se considera una sola operación de E/S. Las operaciones de E/S mayores de 256 KiB de rendimiento se consideran varias operaciones de E/S con un tamaño de 256 KiB.

## <a name="standard-ssd"></a>SSD estándar

Los discos SSD estándar de Azure son una opción de almacenamiento rentable, optimizada para cargas de trabajo que necesitan un rendimiento constante en niveles inferiores de IOPS. Los discos SSD estándar ofrecen una buena experiencia de nivel de entrada para aquellos que desean pasarse a la nube, en especial si tiene problemas con la variación de las cargas de trabajo que se ejecutan en las soluciones de disco duro locales. En comparación con los discos HDD estándar, los discos SSD estándar ofrecen mayor disponibilidad, coherencia, confiabilidad y latencia. Los discos SSD estándar son convenientes para servidores web, servidores de aplicaciones con IOPS bajas, aplicaciones empresariales de poco uso y cargas de trabajo de desarrollo/pruebas. Al igual que los discos HDD estándar, los SSD están disponibles en todas las máquinas virtuales de Azure.

### <a name="disk-size"></a>Tamaño del disco
[!INCLUDE [disk-storage-standard-ssd-sizes](../../includes/disk-storage-standard-ssd-sizes.md)]

Los discos SSD estándar están diseñados para proporcionar latencias de menos de 10 milisegundos y un nivel de IOPS y de rendimiento hasta los límites descritos en la tabla anterior durante el 99 % del tiempo. Las IOPS y el rendimiento reales pueden variar a veces, según los patrones de tráfico. Los discos SSD estándar proporcionarán un rendimiento más coherente que los discos HDD, con una latencia menor.

### <a name="transactions"></a>Transacciones

Para los discos SSD estándar, cada operación de E/S menor o igual a 256 KiB de rendimiento se considera una sola operación de E/S. Las operaciones de E/S mayores de 256 KiB de rendimiento se consideran varias operaciones de E/S con un tamaño de 256 KiB. Estas transacciones tienen un impacto en la facturación.

## <a name="standard-hdd"></a>HDD estándar

Los discos HDD estándar de Azure ofrecen compatibilidad de discos confiable y de bajo coste para las máquinas virtuales que ejecutan cargas de trabajo que no tienen en cuenta la latencia. Con Standard Storage, los datos se almacenan en unidades de disco duro (HDD). La latencia, las IOPS y el rendimiento de los discos HDD estándar pueden variar más en comparación con los discos basados en SSD. Los discos HDD estándar están diseñados para ofrecer latencias de escritura inferiores a 10 ms y latencias de lectura inferiores a 20 ms para la mayoría de las operaciones de E/S, aunque el rendimiento real puede variar según el tamaño de E/S y el patrón de carga de trabajo. Cuando se trabaja con máquinas virtuales, se pueden usar discos HDD estándar para escenarios de desarrollo/pruebas y para cargas de trabajo menos críticas. Los discos HDD estándar están disponibles en todas las regiones de Azure y se pueden utilizar con todas las máquinas virtuales de Azure.

### <a name="disk-size"></a>Tamaño del disco
[!INCLUDE [disk-storage-standard-hdd-sizes](../../includes/disk-storage-standard-hdd-sizes.md)]

### <a name="transactions"></a>Transacciones

En el caso de los discos HDD estándar, cada operación de E/S se considera como una única transacción, independientemente del tamaño de esta. Estas transacciones tienen un impacto en la facturación.

## <a name="billing"></a>Facturación

Al usar Managed Disks, se aplican las siguientes consideraciones de facturación:

- Tipo de disco
- Tamaño del disco administrado
- Instantáneas
- Transferencias de datos de salida
- Número de transacciones

**Tamaño del disco administrado**: los discos administrados se facturan por el tamaño aprovisionado. Azure asigna el tamaño aprovisionado (redondeado al alza) a la oferta de tamaño de disco más cercana. Para obtener detalles sobre los tamaños de disco ofrecidos, consulte las tablas anteriores. Cada disco se asigna a una oferta de tamaño de disco aprovisionado compatible y se factura según corresponda. Por ejemplo, si ha aprovisionado un disco SSD estándar de 200 GiB, se asigna a la oferta de tamaño de disco E15 (256 GiB). La facturación de cualquier disco aprovisionado se prorratea cada hora con el precio mensual de la oferta de almacenamiento. Por ejemplo, si ha aprovisionado un disco E10 y lo ha eliminado después de 20 horas, se factura la oferta E10 prorrateada a 20 horas. Esto es así independientemente de la cantidad de datos que se escriba en el disco.

**Instantáneas**: Las instantáneas se facturan según el tamaño utilizado. Por ejemplo, si crea una instantánea de un disco administrado con capacidad aprovisionada de 64 GiB y el tamaño de datos usado real es de 10 GiB, solo se le cobra por el tamaño de datos usado de 10 GiB.

Para más información acerca de las instantáneas, consulte la sección sobre las instantáneas en la [introducción a los discos administrados](managed-disks-overview.md).

**Transferencias de datos de salida**: las [transferencias de datos de salida](https://azure.microsoft.com/pricing/details/bandwidth/) (datos que salen de los centros de datos de Azure) se facturan en función del uso de ancho de banda.

**Transacciones**: se le factura por el número de transacciones que realiza en un disco administrado estándar. Para los discos SSD estándar, cada operación de E/S menor o igual a 256 KiB de rendimiento se considera una sola operación de E/S. Las operaciones de E/S mayores de 256 KiB de rendimiento se consideran varias operaciones de E/S con un tamaño de 256 KiB. En el caso de los discos HDD estándar, cada operación de E/S se considera como una única transacción, independientemente del tamaño de esta.

Para obtener información detallada sobre los precios de Managed Disks, incluidos los costes de las transacciones, consulte [Precios de Managed Disks](https://azure.microsoft.com/pricing/details/managed-disks).

### <a name="ultra-disk-vm-reservation-fee"></a>Precio de reserva de máquina virtual para discos Ultra

Las máquinas virtuales de Azure tienen la funcionalidad para indicar si son compatibles con discos Ultra. Una máquina virtual compatible con SSD Ultra asigna capacidad dedicada de ancho de banda entre la instancia de la máquina virtual de proceso y la unidad de escalado de almacenamiento en bloque, para así poder optimizar el rendimiento y reducir la latencia. Al agregar esta funcionalidad en la máquina virtual, se crea un cargo por reserva que solo se cobra si habilita la funcionalidad de discos Ultra en la máquina virtual sin asociarle uno de estos discos. Cuando un disco Ultra está asociado a la máquina virtual compatible con estos discos, este cargo no se aplicará. Este cargo se calcula según la vCPU aprovisionada en la máquina virtual. 

> [!Note]
> En el caso de los [tamaños principales de máquina virtual restringidos](constrained-vcpu.md), la tarifa de la reserva se basa en el número real de vCPU y no en los núcleos restringidos. En el caso de Standard_E32-8s_v3, la tarifa de la reserva se basará en 32 núcleos. 

Para información sobre los precios de los discos Ultra, consulte la [página de precios de discos de Azure](https://azure.microsoft.com/pricing/details/managed-disks/).

### <a name="azure-disk-reservation"></a>Reserva de discos de Azure

La reserva de discos es la opción de adquirir de antemano un año de almacenamiento en discos con descuento, lo que reduce los costos totales. Al comprar una reserva de discos, selecciona una SKU de disco específica en una región de destino, por ejemplo, 10 SSD Premium de P30 (1 TiB) en la región Este de EE. UU. 2 por un plazo de un año. La experiencia de reserva es similar a las instancias reservadas de máquina virtual (VM). Puede agrupar las reservas de discos y máquinas virtuales para maximizar el ahorro. Por ahora, la reserva de discos de Azure ofrece un plan de compromiso de un año para las SKU de SSD Premium de P30 (1 TiB) a P80 (32 TiB) en todas las regiones de producción. Para más información sobre los precios de los discos reservados, consulte la [página de precios de los discos de Azure](https://azure.microsoft.com/pricing/details/managed-disks/).

## <a name="next-steps"></a>Pasos siguientes

Para comenzar, consulte [Precios de Managed Disks](https://azure.microsoft.com/pricing/details/managed-disks/).
