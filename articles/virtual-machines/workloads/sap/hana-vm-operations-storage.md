---
title: Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA | Microsoft Docs
description: Recomendaciones de almacenamiento para máquinas virtuales con SAP HANA implementado en ellas.
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 06/30/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: c1e0efc2c64a1cbdcc2c83c019f7743406054afe
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074031"
---
# <a name="sap-hana-azure-virtual-machine-storage-configurations"></a>Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA

Azure proporciona distintos tipos de almacenamiento adecuados para máquinas virtuales de Azure que ejecutan SAP HANA. Los **tipos de almacenamiento de Azure con certificación de SAP HANA** que pueden considerarse para las implementaciones de SAP HANA, como: 

- SSD Premium o Premium Storage de Azure 
- [Disco Ultra](../../linux/disks-enable-ultra-ssd.md)
- [Azure NetApp Files](https://azure.microsoft.com/services/netapp/) 

Para obtener información sobre estos tipos de disco, consulte el artículo [Tipos de Azure Storage para la carga de trabajo de SAP](./planning-guide-storage.md) y [Seleccionar un tipo de disco](../../linux/disks-types.md).

Azure ofrece dos métodos de implementación de VHD en Azure Standard Storage y Azure Premium Storage. Esperamos que aproveche el [disco administrado de Azure](https://azure.microsoft.com/services/managed-disks/) para implementaciones de almacenamiento en bloque de Azure. 

Para obtener una lista de tipos de almacenamiento y sus Acuerdos de Nivel de Servicio sobre IOPS y rendimiento del almacenamiento, revise la [documentación de Azure para Managed Disks](https://azure.microsoft.com/pricing/details/managed-disks/).

> [!IMPORTANT]
> Independientemente del tipo de almacenamiento de Azure elegido, el sistema de archivos que se usa en ese almacenamiento debe ser compatible con SAP para el sistema operativo y el DBMS específicos. La [nota de compatibilidad 405827 de SAP](https://launchpad.support.sap.com/#/notes/405827) enumera los sistemas de archivos admitidos para los diferentes sistemas operativos y bases de datos, incluido SAP HANA. Esto se aplica a todos los volúmenes SAP HANA con acceso de lectura y escritura para cualquier tarea. Especialmente si usa NFS en Azure para SAP HANA, se aplican restricciones adicionales de versiones de NFS como se indica más adelante en este artículo. 


Las condiciones de certificación mínimas de SAP HANA para los diferentes tipos de almacenamiento son: 

- Azure Premium Storage: el [Acelerador de escritura](../../linux/how-to-enable-write-accelerator.md) de Azure debe admitir **/hana/log**. El volumen **/hana/data** puede colocarse en Premium Storage sin el Acelerador de escritura de Azure o bien en un disco Ultra.
- Disco Ultra de Azure al menos para el volumen **/hana/log**. El volumen **/hana/data** se puede colocar en el Premium Storage sin el Acelerador de escritura de Azure o en un disco Ultra para obtener un tiempo de reinicio más rápido.
- Volúmenes de **NFS v4.1** sobre Azure NetApp Files para **/hana/log y /hana/data**. El volumen de /hana/shared puede usar el protocolo NFS v3 o NFS v 4.1.

Algunos de los tipos de almacenamiento se pueden combinar. Por ejemplo, es posible colocar **/hana/data** en Premium Storage y **/hana/log** se puede colocar en un almacenamiento en disco Ultra para obtener la baja latencia necesaria. Si usa un volumen basado en ANF para **/hana/data**, el volumen **/hana/log** se debe basar también en NFS, además de en ANF. **No se admite** el uso de NFS sobre ANF para uno de los volúmenes (como /hana/data) y Azure Premium Storage o almacenamiento en disco Ultra para el otro volumen (como **/hana/log**).

En el mundo local, casi nunca ha tenido que preocuparse sobre los subsistemas de E/S y sus funcionalidades. El motivo era que el proveedor de las aplicaciones tenía que asegurarse de que se cumplieran los requisitos de almacenamiento mínimo para SAP HANA. Mientras crea la infraestructura de Azure, debe tener en cuenta algunos de estos requisitos que emite SAP. Algunas de las características de rendimiento mínimas que recomienda SAP son:

- Lectura o escritura en **/hana/log** de 250 MB/s con tamaños de E/S de 1 MB
- Actividad de lectura de al menos 400 MB/s para **/hana/data** con tamaños de E/S de 16 MB y 64 MB
- Actividad de escritura de al menos 250 MB/s para **/hana/data** con tamaños de E/S de 16 MB y 64 MB

Debido a que la baja latencia de almacenamiento es fundamental para los sistemas DBMS, incluso estos DBMS, como SAP HANA, mantienen los datos en memoria. La ruta crítica en el almacenamiento normalmente está relacionada con las escrituras del registro de transacciones de los sistemas DBMS. Sin embargo, también las operaciones como la escritura de puntos de retorno o la carga de datos en memoria después de la recuperación tras un bloqueo pueden ser críticas. Por ello, es **obligatorio** usar Azure Premium Storage, discos Ultra o ANF para los volúmenes **/hana/data** y **/hana/log**. 


Estos son algunos principios de la selección de la configuración de almacenamiento para HANA:

- Decida el tipo de almacenamiento según [Tipos de Azure Storage para la carga de trabajo de SAP](./planning-guide-storage.md) y [Seleccionar un tipo de disco](../../linux/disks-types.md).
- El rendimiento global de E/S de la VM y los límites de IOPS al elegir una VM o determinar su tamaño. El rendimiento general del almacenamiento de VM está documentado en el artículo [Tamaños de máquina virtual optimizada para memoria](../../sizes-memory.md).
- A la hora de decidirse por la configuración del almacenamiento, intente permanecer por debajo del rendimiento general de la VM con la configuración del volumen **/hana/data**. En cuanto a los puntos de retorno, SAP HANA puede presentar cierta agresividad al emitir E/S. Es fácil alcanzar los límites de rendimiento del volumen **/hana/data** al escribir un punto de retorno. Si los discos que compilan el volumen **/hana/data** tienen un rendimiento superior al que permite la VM, puede que se produzcan situaciones en que el rendimiento usado por la escritura del punto de retorno interfiera con las demandas de rendimiento de las escrituras de registros de fase de puesta al día. Esta situación puede afectar al rendimiento de la aplicación.
- Si usa Azure Premium Storage, la configuración menos costosa es usar administradores de volúmenes lógicos para crear conjuntos de franjas con el fin de compilar los volúmenes **/hana/data** y **/hana/log**.

> [!IMPORTANT]
> Las sugerencias para las configuraciones de almacenamiento se han diseñado como indicaciones con las que empezar. Al ejecutar la carga de trabajo y analizar los patrones de uso del almacenamiento, es posible que se dé cuenta de que no está usando todo el ancho de banda de almacenamiento o IOPS proporcionado. En ese caso, puede considerar la posibilidad de reducir el tamaño del almacenamiento. También puede que, por el contrario, la carga de trabajo necesite más rendimiento de almacenamiento del sugerido con estas configuraciones. Como resultado, es posible que tenga que implementar más capacidad, IOPS o rendimiento. En el campo de la tensión entre la capacidad de almacenamiento necesaria, la latencia de almacenamiento necesaria, el rendimiento de almacenamiento y las IOPS necesarias y la configuración menos costosa, Azure ofrece suficientes tipos de almacenamiento diferentes con distintas funcionalidades y precios de venta para determinar el compromiso adecuado para usted y su carga de trabajo de HANA y adaptarse a este.

## <a name="linux-io-scheduler-mode"></a>Modo de programador de E/S de Linux
Linux tiene varios modos diferentes de programación de E/S. Una recomendación común de los proveedores de Linux y SAP consiste en reconfigurar el modo de programador de E/S para los volúmenes de disco del modo **mq-deadline** o **kyber** en el modo **noop** (no multicola) o **none** (multicola). Los detalles se incluyen en la [nota de SAP n.º 1984787](https://launchpad.support.sap.com/#/notes/1984787). 


## <a name="solutions-with-premium-storage-and-azure-write-accelerator-for-azure-m-series-virtual-machines"></a>Soluciones con Premium Storage y el Acelerador de escritura de Azure para máquinas virtuales de la serie M de Azure
El Acelerador de escritura de Azure es una funcionalidad que está disponible exclusivamente para las máquinas virtuales de la serie M de Azure. Como el nombre indica, el propósito de la funcionalidad es mejorar la latencia de E/S de las escrituras en Azure Premium Storage. Para SAP HANA, el Acelerador de escritura se supone que debe usarse solo en el volumen **/hana/log**. Por lo tanto, **/hana/data** y **/hana/log** son volúmenes independientes y el Acelerador de escritura de Azure admite solo el volumen **/hana/log**. 

> [!IMPORTANT]
> Cuando se usa Azure Premium Storage, es obligatorio usar el [Acelerador de escritura](../../linux/how-to-enable-write-accelerator.md) para el volumen **/hana/log**. El Acelerador de escritura solo está disponible para Premium Storage y para las VM de las series M y Mv2. El Acelerador de escritura no funciona en combinación con otras familias de VM de Azure, como Esv3 o Edsv4.

Las recomendaciones de almacenamiento en caché para los siguientes discos Premium de Azure suponen que se dan las siguientes características de E/S para SAP HANA:

- Apenas hay carga de trabajo de lectura en los archivos de datos de HANA. Las excepciones son las operaciones de E/S de tamaño grande después del reinicio de la instancia HANA o cuando se cargan datos en HANA. Otro caso de operaciones de E/S de lectura de mayor tamaño con archivos de datos pueden ser las copias de seguridad de base de datos de HANA. Como resultado, el almacenamiento en caché de lectura no tiene sentido ya que, en la mayoría de los casos, todos los volúmenes de archivos de datos tienen que leerse por completo. 
- La escritura en los archivos de datos se produce en ráfagas según los puntos de retorno de HANA y la recuperación tras bloqueos de HANA. La escritura de los puntos de retorno es asincrónica y no bloquea las transacciones de usuario. La escritura de datos durante la recuperación tras bloqueo es esencial en el rendimiento a fin de lograr que el sistema responda de nuevo rápidamente. Sin embargo, la recuperación tras bloqueo debería constituir una situación excepcional.
- Apenas hay lecturas de los archivos de puesta al día de HANA. Las operaciones de E/S grandes al realizar copias de seguridad del registro de transacciones, la recuperación tras bloqueo o en la fase de reinicio de una instancia de HANA constituyen excepciones.  
- La carga principal en el caso del archivo de registro de puesta al día SAP HANA la constituyen las escrituras. Según la naturaleza de la carga de trabajo, puede tener operaciones de E/S de solo 4 KB o llegar a superar 1 MB. La latencia de las escrituras en relación con el registro de puesta al día SAP HANA es esencial para el rendimiento.
- Todas las operaciones de escritura se deben conservar en el disco de forma confiable.

**Recomendación: Como resultado de estos patrones de E/S observados por SAP HANA, debe establecer el almacenamiento en caché de los diferentes volúmenes con Azure Premium Storage como sigue:**

- **/hana/data**: sin almacenamiento en caché ni almacenamiento en caché de lectura
- **/hana/log**: sin almacenamiento en caché. Excepción para las máquinas virtuales de las series M y Mv2, en las que el Acelerador de escritura debe estar habilitado sin almacenamiento en caché de lectura. 
- **/hana/shared**: almacenamiento en caché de lecturas
- **Disco de SO**: no cambie el almacenamiento en caché predeterminado establecido por Azure en el momento de la creación de la VM.


Si usa LVM o mdadm para crear conjuntos de franjas en varios discos Premium de Azure, debe definir tamaños de franja. Estos tamaños difieren entre **/hana/data** y **/hana/log**. **Recomendación: En cuanto a tamaños de franja, la recomendación es usar:**

- 256 KB para **/hana/data**
- 64 KB para **/hana/log**

> [!NOTE]
> El tamaño de franja de **/hana/data** ha cambiado con respecto a las recomendaciones anteriores que precisan de 64 KB o 128 KB a 256 KB según las experiencias del cliente con versiones más recientes de Linux. El tamaño de 256 KB proporciona un rendimiento ligeramente superior. También hemos cambiado la recomendación para los tamaños de franja de **/hana/log** de 32 KB a 64 KB para obtener un rendimiento suficiente con tamaños de E/S superiores.

> [!NOTE]
> No es necesario configurar ningún nivel de redundancia con volúmenes RAID, ya que el almacenamiento en bloque de Azure conserva tres imágenes de un VHD. El uso de un conjunto de franjas con discos Premium de Azure es, simplemente, configurar volúmenes que proporcionen suficientes IOPS o rendimiento de E/S.

La acumulación de un número de VHD de Azure por debajo de un conjunto de franjas es acumulativo por parte de las IOPS y del rendimiento de almacenamiento. Por lo tanto, si coloca un conjunto de franjas en 3 x P30 discos de Azure Premium Storage, debe proporcionarle tres veces las IOPS y tres veces el rendimiento de almacenamiento de un solo disco P30 de Azure Premium Storage.

> [!IMPORTANT]
> Si usa LVM o mdadm como administrador de volúmenes para crear conjuntos de bandas en varios discos de Azure Premium, los tres sistemas de archivos de SAP HANA /data, /log y /shared no se deben colocar en un grupo de volúmenes raíz o predeterminado. Se recomienda seguir la guía de los proveedores de Linux, que normalmente indica crear grupos de volúmenes individuales para /data, /log y /shared.


### <a name="azure-burst-functionality-for-premium-storage"></a>Funcionalidad de ráfaga de Azure para Premium Storage
En el caso de los discos de Azure Premium Storage con una capacidad de 512 GiB o inferior, se ofrece funcionalidad de ráfaga. La forma exacta en que funciona la ráfaga de disco se describe en el artículo [Seguridad de disco](../../linux/disk-bursting.md). Al leer el artículo, comprenderá el concepto de acumulación de IOPS y rendimiento en los casos en que la carga de trabajo de E/S esté por debajo del valor de IOPS y de rendimiento de los discos nominal (para obtener más información sobre el rendimiento nominal, consulte [Precios del disco administrado](https://azure.microsoft.com/pricing/details/managed-disks/)). Acumulará la diferencia de IOPS y rendimiento entre el uso actual y los valores nominales del disco. Las ráfagas están limitadas a un máximo de 30 minutos.

Los casos ideales en los que se puede planear esta funcionalidad de ráfaga serán, probablemente, los volúmenes o discos que contengan archivos de datos para distintos DBMS. Se espera que la carga de trabajo de E/S para esos volúmenes, especialmente con sistemas de gama pequeña o mediana, tenga el aspecto siguiente:

- Carga de trabajo de lectura baja a moderada, ya que los datos, idealmente, se almacenan en la memoria caché, o, en el caso de HANA, deben estar completamente en la memoria.
- Ráfagas de escritura desencadenadas por puntos de control o puntos de retorno de la base de datos que se emiten de forma regular
- Carga de trabajo de copia de seguridad que se lee en un flujo continuo en los casos en que las copias de seguridad no se ejecutan mediante instantáneas de almacenamiento
- En el caso de SAP HANA, carga de los datos en memoria después de reiniciar una instancia

Especialmente en sistemas DBMS más pequeños en los que la carga de trabajo controla solo unos cientos de transacciones por segundo, esta funcionalidad de ráfaga puede tener sentido para los discos o volúmenes que almacenan la transacción o el registro de fase de puesta al día. La carga de trabajo esperada en este disco o volúmenes tiene el siguiente aspecto:

- Las escrituras regulares en el disco que dependen de la carga de trabajo y la naturaleza de la carga de trabajo, ya que es probable que todas las confirmaciones emitidas por la aplicación desencadenen una operación de E/S
- Mayor carga de trabajo en el rendimiento para los casos de tareas operativas, como crear o recompilar índices
- Ráfagas de lectura al realizar copias de seguridad de registros de transacciones o fase de puesta al día


### <a name="production-recommended-storage-solution-based-on-azure-premium-storage"></a>Solución de almacenamiento recomendada para producción basada en Azure Premium Storage

> [!IMPORTANT]
> La certificación de SAP HANA para las máquinas virtuales de serie M de Azure es exclusivamente con el Acelerador de escritura de Azure para el volumen **/hana/log**. Como resultado, las implementaciones de SAP HANA en un escenario de producción en máquinas virtuales de la serie M de Azure se espera que estén configuradas con el Acelerador de escritura de Azure para el volumen **/hana/log**.  

> [!NOTE]
> En escenarios que implican Azure Premium Storage, estamos implementando funcionalidades de ráfaga en la configuración. A medida que use herramientas de prueba de almacenamiento de cualquier forma, tenga en cuenta el [funcionamiento de la expansión de discos Premium de Azure](../../linux/disk-bursting.md). Al ejecutar las pruebas de almacenamiento que proporciona la herramienta HWCCT o HCMT de SAP, no se espera que todas las pruebas cumplan los criterios, ya que algunas pruebas superarán los créditos de expansión que se pueden acumular. Especialmente, cuando todas las pruebas se ejecutan secuencialmente sin interrupción.


> [!NOTE]
> Para escenarios de producción, compruebe si un determinado tipo de máquina virtual es compatible con SAP HANA de SAP en la [documentación de SAP para IaaS](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html).

**Recomendación: Las configuraciones recomendadas para escenarios de producción de Azure Premium Storage tienen el siguiente aspecto:**

Configuración para el volumen **/hana/data** de SAP:

| SKU de la máquina virtual | RAM | Máx. E/S de VM<br /> Throughput | /hana/data | Rendimiento de ráfaga máximo | E/S | IOPS de ráfaga |
| --- | --- | --- | --- | --- | --- | --- | 
| M32ts | 192 GiB | 500 MBps | 4 x P6 | 680 Mbps | 960 | 14 000 |
| M32ls | 256 GiB | 500 MBps | 4 x P6 | 680 Mbps | 960 | 14 000 |
| M64ls | 512 GB | 1000 MBps | 4 x P10 |  680 Mbps | 2\.000 | 14 000 |
| M64s | 1000 GiB | 1000 MBps | 4 x P15 | 680 Mbps | 4400 | 14 000 |
| M64ms | 1750 GiB | 1000 MBps | 4 x P20 | 680 Mbps | 9200 | 14 000 |  
| M128s | 2000 GiB | 2000 Mbps | 4 x P20 | 680 Mbps | 9200| 14 000 | 
| M128ms | 3800 GiB | 2000 Mbps | 4 x P30 | 800 Mbps (aprovisionado) | 20.000 | sin ráfaga | 
| M208s_v2 | 2850 GiB | 1000 MBps | 4 x P30 | 800 Mbps (aprovisionado) | 20.000| sin ráfaga | 
| M208ms_v2 | 5700 GiB | 1000 MBps | 4 x P40 | 1000 Mbps (aprovisionado) | 25 000 | sin ráfaga |
| M416s_v2 | 5700 GiB | 2000 Mbps | 4 x P40 | 1000 Mbps (aprovisionado) | 25 000 | sin ráfaga |
| M416ms_v2 | 11 400 GiB | 2000 Mbps | 4 x P50 | 2000 Mbps (aprovisionado) | 25 000 | sin ráfaga |


Para el volumen **/hana/log**. La configuración sería similar a la siguiente:

| SKU de la máquina virtual | RAM | Máx. E/S de VM<br /> Throughput | Volumen **/hana/log** | Rendimiento de ráfaga máximo | E/S | IOPS de ráfaga |
| --- | --- | --- | --- | --- | --- | --- | 
| M32ts | 192 GiB | 500 MBps | 3 x P10 | 510 Mbps | 1500 | 10 500 | 
| M32ls | 256 GiB | 500 MBps | 3 x P10 | 510 Mbps | 1500 | 10 500 | 
| M64ls | 512 GB | 1000 MBps | 3 x P10 | 510 Mbps | 1500 | 10 500 | 
| M64s | 1000 GiB | 1000 MBps | 3 x P15 | 510 Mbps | 3300 | 10 500 | 
| M64ms | 1750 GiB | 1000 MBps | 3 x P15 | 510 Mbps | 3300 | 10 500 |  
| M128s | 2000 GiB | 2000 Mbps | 3 x P15 | 510 Mbps | 3300 | 10 500|  
| M128ms | 3800 GiB | 2000 Mbps | 3 x P15 | 510 Mbps | 3300 | 10 500 | 
| M208s_v2 | 2850 GiB | 1000 MBps | 3 x P15 | 510 Mbps | 3300 | 10 500 |  
| M208ms_v2 | 5700 GiB | 1000 MBps | 3 x P15 | 510 Mbps | 3300 | 10 500 |  
| M416s_v2 | 5700 GiB | 2000 Mbps | 3 x P15 | 510 Mbps | 3300 | 10 500 |  
| M416ms_v2 | 11 400 GiB | 2000 Mbps | 3 x P15 | 510 Mbps | 3300 | 10 500 | 


For the other volumes, the configuration would look like:

| SKU de la máquina virtual | RAM | Máx. E/S de VM<br /> Throughput | /hana/shared | /root volume | /usr/sap |
| --- | --- | --- | --- | --- | --- | --- | --- | -- |
| M32ts | 192 GiB | 500 MBps | 1 x P20 | 1 x P6 | 1 x P6 |
| M32ls | 256 GiB | 500 MBps |  1 x P20 | 1 x P6 | 1 x P6 |
| M64ls | 512 GB | 1000 MBps | 1 x P20 | 1 x P6 | 1 x P6 |
| M64s | 1000 GiB | 1000 MBps | 1 x P30 | 1 x P6 | 1 x P6 |
| M64ms | 1750 GiB | 1000 MBps | 1 x P30 | 1 x P6 | 1 x P6 | 
| M128s | 2000 GiB | 2000 Mbps | 1 x P30 | 1 x P10 | 1 x P6 | 
| M128ms | 3800 GiB | 2000 Mbps | 1 x P30 | 1 x P10 | 1 x P6 |
| M208s_v2 | 2850 GiB | 1000 MBps |  1 x P30 | 1 x P10 | 1 x P6 |
| M208ms_v2 | 5700 GiB | 1000 MBps | 1 x P30 | 1 x P10 | 1 x P6 | 
| M416s_v2 | 5700 GiB | 2000 Mbps |  1 x P30 | 1 x P10 | 1 x P6 | 
| M416ms_v2 | 11 400 GiB | 2000 Mbps | 1 x P30 | 1 x P10 | 1 x P6 | 


Compruebe si el rendimiento del almacenamiento para los diferentes volúmenes sugeridos se adapta a la carga de trabajo que se va a ejecutar. Si la carga de trabajo requiere volúmenes mayores para **/hana/data** y **/hana/log**, debe aumentar el número de VHD de Azure Premium Storage. Ajustar el tamaño de un volumen con más discos duros virtuales de los que se enumeran, aumenta el IOPS y el rendimiento de E/S dentro de los límites del tipo de máquina virtual de Azure.

El Acelerador de escritura de Azure solo funciona en conjunto con [Azure Managed Disks](https://azure.microsoft.com/services/managed-disks/). Así pues, al menos los discos de Azure Premium Storage que forman el volumen **/hana/log** deben implementarse como discos administrados. Puede encontrar instrucciones y restricciones del Acelerador de escritura de Azure en el artículo [Acelerador de escritura](../../linux/how-to-enable-write-accelerator.md).

En el caso de las VM certificadas para HANA de la familia de [Esv3](../../ev3-esv3-series.md?toc=/azure/virtual-machines/linux/toc.json&bc=/azure/virtual-machines/linux/breadcrumb/toc.json#esv3-series) y [Edsv4](../../edv4-edsv4-series.md?toc=/azure/virtual-machines/linux/toc.json&bc=/azure/virtual-machines/linux/breadcrumb/toc.json#edsv4-series), debe aplicar ANF para los volúmenes **/hana/data** y **/hana/log**. O bien debe aprovechar el almacenamiento en disco Ultra de Azure en lugar de Azure Premium Storage solo para el volumen **/hana/log**. Como resultado, las configuraciones del volumen **/hana/data** en Azure Premium Storage podrían tener este aspecto:

| SKU de la máquina virtual | RAM | Máx. E/S de VM<br /> Throughput | /hana/data | Rendimiento de ráfaga máximo | E/S | IOPS de ráfaga |
| --- | --- | --- | --- | --- | --- | --- |
| E20ds_v4 | 160 GiB | 480 Mbps | 3 x P10 | 510 Mbps | 1500 | 10 500 |
| E32ds_v4 | 256 GiB | 768 MBps | 3 x P10 |  510 Mbps | 1500 | 10 500|
| E48ds_v4 | 384 GiB | 1152 Mbps | 3 x P15 |  510 Mbps | 3300  | 10 500 | 
| E64ds_v4 | 504 GiB | 1200 MBps | 3 x P15 |  510 Mbps | 3300 | 10 500 | 
| E64s_v3 | 432 GiB | 1200 MB/s | 3 x P15 |  510 Mbps | 3300 | 10 500 | 

En el caso del resto de volúmenes, incluido **/hana/log** del disco Ultra, la configuración podría tener este aspecto:

| SKU de la máquina virtual | RAM | Máx. E/S de VM<br /> Throughput | Volumen /hana/log | Rendimiento de E/S de /hana/log | Operaciones de E/S por segundo en /hana/log | /hana/shared | /root volume | /usr/sap |
| --- | --- | --- | --- | --- | --- | --- | --- | -- |
| E20ds_v4 | 160 GiB | 480 Mbps | 80 GB | 250 MBps | 1800 | 1 x P15 | 1 x P6 | 1 x P6 |
| E32ds_v4 | 256 GiB | 768 MBps | 128 GB | 250 MBps | 1800 | 1 x P15 | 1 x P6 | 1 x P6 |
| E48ds_v4 | 384 GiB | 1152 Mbps | 192 GB | 250 MBps | 1800 | 1 x P20 | 1 x P6 | 1 x P6 |
| E64ds_v4 | 504 GiB | 1200 MBps | 256 GB | 250 MBps | 1800 | 1 x P20 | 1 x P6 | 1 x P6 |
| E64s_v3 | 432 GiB | 1200 MBps | 220 GB | 250 MBps | 1800 | 1 x P20 | 1 x P6 | 1 x P6 |


## <a name="azure-ultra-disk-storage-configuration-for-sap-hana"></a>Configuración de almacenamiento en el disco Ultra de Azure para SAP HANA
Otro tipo de almacenamiento de Azure se denomina [disco Ultra de Azure](../../windows/disks-types.md#ultra-disk). Una diferencia considerable entre el almacenamiento de Azure que se ofrecía hasta ahora y el disco Ultra es que las funcionalidades del disco ya no están asociadas a su tamaño. Como un cliente, para el disco Ultra puede definir estas funcionalidades:

- El tamaño de disco, de 4 GiB a 65 536 GiB
- Intervalo de 100 operaciones de E/S por segundo a 160 000 operaciones de E/S por segundo (el máximo depende también de los tipos de máquina virtual)
- Rendimiento del almacenamiento de 300 MB/s a 2000 MB por segundo

El disco Ultra ofrece la posibilidad de definir un único disco que responda a su rango de necesidades de tamaño, IOPS y rendimiento de disco. En lugar de usar administradores de volúmenes lógicos como LVM o MDADM además de Azure Premium Storage para construir los volúmenes que cumplan los requisitos de rendimiento de almacenamiento y de IOPS. Puede ejecutar una configuración combinada de disco Ultra y Premium Storage. Como resultado, puede limitar el uso del disco Ultra a los volúmenes **/hana/data** y **/hana/log** críticos para el rendimiento y cubrir el resto de volúmenes con Azure Premium Storage.

Otra ventaja del disco Ultra puede ser una mejor latencia de lectura en comparación con Premium Storage. Una latencia de lectura más rápida puede tener ventajas si desea reducir los tiempos de inicio de HANA y la carga posterior de los datos en la memoria. También se pueden apreciar las ventajas del almacenamiento en disco Ultra cuando HANA está escribiendo puntos de retorno. 

> [!NOTE]
> El disco Ultra todavía no está presente en todas las regiones de Azure y aún no es compatible con todos los tipos de máquinas virtuales que se indican a continuación. Para obtener información detallada sobre dónde está disponible el disco Ultra y qué familias de máquinas virtuales se admiten, consulte el artículo [¿Qué tipos de disco están disponibles en Azure?](../../windows/disks-types.md#ultra-disk).

### <a name="production-recommended-storage-solution-with-pure-ultra-disk-configuration"></a>Solución de almacenamiento recomendada para producción con configuración de disco Ultra pura
En esta configuración, puede conservar los volúmenes **/hana/data** y **/hana/log** por separado. Los valores sugeridos se derivan de los KPI que SAP tiene para certificar los tipos de máquina virtual de SAP HANA y las configuraciones de almacenamiento como se recomienda en las [notas del producto SAP TDI Storage](https://www.sap.com/documents/2015/03/74cdb554-5a7c-0010-82c7-eda71af511fa.html).

A menudo, las recomendaciones superan los requisitos mínimos de SAP como se indicó anteriormente en este artículo. Las recomendaciones enumeradas son un compromiso entre las recomendaciones de tamaño de SAP y el rendimiento de almacenamiento máximo que proporcionan los diferentes tipos de máquinas virtuales.

| SKU de la máquina virtual | RAM | Máx. E/S de VM<br /> Throughput | Volumen /hana/data | Rendimiento de E/S de /hana/data | Operaciones de E/S por segundo en /hana/data | Volumen /hana/log | Rendimiento de E/S de /hana/log | Operaciones de E/S por segundo en /hana/log |
| --- | --- | --- | --- | --- | --- | --- | --- | -- |
| E20ds_v4 | 160 GiB | 480 MB/s | 200 GB | 400 MBps | 2500 | 80 GB | 250 MB | 1800 |
| E32ds_v4 | 256 GiB | 768 MB/s | 300 GB | 400 MBps | 2500 | 128 GB | 250 MBps | 1800 |
| E48ds_v4 | 384 GiB | 1152 MB/s | 460 GB | 400 MBps | 3000 | 192 GB | 250 MBps | 1800 |
| E64ds_v4 | 504 GiB | 1200 MB/s | 610 GB | 400 MBps | 3500 |  256 GB | 250 MBps | 1800 |
| E64s_v3 | 432 GiB | 1200 MB/s | 610 GB | 400 MBps | 3500 | 220 GB | 250 MB | 1800 |
| M32ts | 192 GiB | 500 MB/s | 250 GB | 400 MBps | 2500 | 96 GB | 250 MBps  | 1800 |
| M32ls | 256 GiB | 500 MB/s | 300 GB | 400 MBps | 2500 | 256 GB | 250 MBps  | 1800 |
| M64ls | 512 GB | 1000 MB/s | 620 GB | 400 MBps | 3500 | 256 GB | 250 MBps  | 1800 |
| M64s | 1000 GiB | 1000 MB/s |  1200 GB | 600 MBps | 5\.000 | 512 GB | 250 MBps  | 2500 |
| M64ms | 1750 GiB | 1000 MB/s | 2100 GB | 600 MBps | 5\.000 | 512 GB | 250 MBps  | 2500 |
| M128s | 2000 GiB | 2000 MB/s |2400 GB | 750 Mbps | 7000 | 512 GB | 250 MBps  | 2500 | 
| M128ms | 3800 GiB | 2000 MB/s | 4800 GB | 750 Mbps |7000 | 512 GB | 250 MBps  | 2500 | 
| M208s_v2 | 2850 GiB | 1000 MB/s | 3500 GB | 750 Mbps | 7000 | 512 GB | 250 MBps  | 2500 | 
| M208ms_v2 | 5700 GiB | 1000 MB/s | 7200 GB | 750 Mbps | 7000 | 512 GB | 250 MBps  | 2500 | 
| M416s_v2 | 5700 GiB | 2000 MB/s | 7200 GB | 1000 MBps | 9000 | 512 GB | 400 MBps  | 4\.000 | 
| M416ms_v2 | 11 400 GiB | 2000 MB/s | 14 400 GB | 1500 MBps | 9000 | 512 GB | 400 MBps  | 4\.000 |   

**Los valores de la lista están diseñados como punto de partida y deben valorarse según las demandas reales.** La ventaja del disco Ultra de Azure es que los valores de IOPS y de rendimiento se pueden adaptar sin necesidad de apagar la máquina virtual o detener la carga de trabajo que se aplica al sistema.   

> [!NOTE]
> Por ahora las instantáneas de almacenamiento con el disco Ultra no están disponibles. Esto bloquea el uso de las instantáneas de máquina virtual con los servicios de Azure Backup


## <a name="nfs-v41-volumes-on-azure-netapp-files"></a>Volúmenes de NFS v4.1 en Azure NetApp Files
Azure NetApp Files proporciona recursos compartidos de NFS nativos que se pueden usar para los volúmenes **/hana/shared**, **/hana/data** y **/hana/log**. El uso de recursos compartidos de NFS basados en ANF para los volúmenes **/hana/data** y **/hana/log** requiere el uso del protocolo NFS v4.1. El protocolo NFS v3 no admite el uso de los volúmenes **/hana/data** y **/hana/log** al basar los recursos compartidos en ANF. 

> [!IMPORTANT]
> **No** se admite el protocolo NFS v3 implementado en Azure NetApp Files para su uso con **/hana/data** y **/hana/log**. El uso de NFS 4.1 es obligatorio para los volúmenes **/hana/data** y **/hana/log** desde un punto de vista funcional. Por su parte, para el volumen **/hana/shared**, se puede usar el protocolo NFS v3 o NFS v4.1 desde un punto de vista funcional.

### <a name="important-considerations"></a>Consideraciones importantes
A la hora de considerar Azure NetApp Files para SAP Netweaver y SAP HANA, tenga en cuenta los siguientes aspectos importantes:

- El grupo de capacidad mínimo es de 4 TiB.  
- El tamaño del volumen mínimo es de 100 GiB.
- Azure NetApp Files y todas las máquinas virtuales en las que los volúmenes de Azure NetApp Files se montarán, se deben implementar en la misma red virtual de Azure o en [redes virtuales emparejadas](../../../virtual-network/virtual-network-peering-overview.md) de la misma región.  
- La red virtual seleccionada debe tener una subred delegada en Azure NetApp Files.
- El rendimiento de un volumen de Azure NetApp es una función de la cuota del volumen y del nivel de servicio, como se documenta en [Nivel de servicio para Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md). Al ajustar el tamaño de los volúmenes de Azure NetApp de HANA, asegúrese de que el rendimiento resultante cumple los requisitos del sistema HANA.  
- Azure NetApp Files ofrece la [directiva de exportación](../../../azure-netapp-files/azure-netapp-files-configure-export-policy.md): puede controlar los clientes permitidos, el tipo de acceso (lectura y escritura, solo lectura, etc.). 
- La característica Azure NetApp Files no depende aún de la zona. En la actualidad, la característica Azure NetApp Files no se implementa en todas las zonas de disponibilidad de una región de Azure. Tenga en cuenta las posibles implicaciones de latencia en algunas regiones de Azure.  
- Es importante que las máquinas virtuales se implementen muy cerca del almacenamiento de Azure NetApp para conseguir una latencia baja. 
- El identificador de usuario para <b>sid</b>adm y el identificador de grupo para `sapsys` en las máquinas virtuales deben coincidir con la configuración de Azure NetApp Files. 

> [!IMPORTANT]
> Para las cargas de trabajo de SAP HANA, una baja latencia resulta fundamental. Trabaje con su representante de Microsoft para asegurarse de que las máquinas virtuales y los volúmenes de Azure NetApp Files se implementan en ubicaciones muy cercanas entre sí.  

> [!IMPORTANT]
> Si hay una discrepancia entre el identificador de usuario para <b>sid</b>adm y el identificador de grupo para `sapsys` entre la máquina virtual y la configuración de Azure NetApp, los permisos de archivos en volúmenes de Azure NetApp, montados en máquinas virtuales, aparecerán como `nobody`. Asegúrese de especificar el identificador de usuario correcto para <b>sid</b>adm y el identificador de grupo para `sapsys` al [incorporar un nuevo sistema](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxjSlHBUxkJBjmARn57skvdUQlJaV0ZBOE1PUkhOVk40WjZZQVJXRzI2RC4u) a Azure NetApp Files.

### <a name="sizing-for-hana-database-on-azure-netapp-files"></a>Ajuste del tamaño de la base de datos HANA en Azure NetApp Files

El rendimiento de un volumen de Azure NetApp es una función del tamaño del volumen y del nivel de servicio, como se documenta en [Nivel de servicio para Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md). 

Al diseñar la infraestructura de SAP en Azure, debe tener en cuenta los requisitos mínimos de rendimiento del almacenamiento de SAP, lo cual se traduce en estas características de rendimiento mínimas:

- Habilitar la lectura/escritura en **/hana/log** de 250 MB/s con tamaños de E/S de 1 MB  
- Habilitar la actividad de lectura de al menos 400 MB/s para **/hana/data** con tamaños de E/S de 16 MB y 64 MB  
- Habilitar la actividad de escritura de al menos 250 MB/s para **/hana/data** con tamaños de E/S de 16 MB y 64 MB  

Los [límites de rendimiento de Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md) por 1 TiB de cuota de volumen son los siguientes:
- Capa de almacenamiento Premium: 64 MiB/s  
- Capa de almacenamiento Ultra: 128 MiB/s  

> [!IMPORTANT]
> Independientemente de la capacidad que implemente en un volumen NFS individual, se espera que el rendimiento se nivele en el rango de 1,2-1,4 GB/s de ancho de banda usado por un consumidor en una máquina virtual. Esto tiene que ver con la arquitectura subyacente de la oferta de ANF y los límites de sesiones de Linux relacionadas alrededor de NFS. Las cifras relativas al rendimiento y a la capacidad de proceso que se indican en el artículo [Banco de pruebas de rendimiento de resultados de pruebas para Azure NetApp Files](../../../azure-netapp-files/performance-benchmarks-linux.md) se obtuvieron en un volumen NFS compartido con varias máquinas virtuales cliente y como resultado con varias sesiones. Dicho escenario es diferente del escenario que medimos en SAP. Ahí medimos la capacidad de procesamiento de una sola máquina virtual en un volumen NFS hospedado en ANF.

Para cumplir los requisitos de rendimiento mínimo de SAP para los datos y el registro, y de acuerdo con las directrices para `/hana/shared`, los tamaños recomendados serían los siguientes:

| Volumen | Size<br /> Capa Premium Storage | Size<br /> Capa de almacenamiento Ultra | Protocolo NFS admitido |
| --- | --- | --- |
| /hana/log/ | 4 TiB | 2 TiB | v4.1 |
| /hana/data | 6,3 TiB | 3,2 TiB | v4.1 |
| /hana/shared | Max (512 GB, 1xRAM) por cuatro nodos de trabajo | Max (512 GB, 1xRAM) por cuatro nodos de trabajo | v3 o v4.1 |


> [!NOTE]
> Las recomendaciones de tamaño de Azure NetApp Files que se han indicado se dirigen a satisfacer los requisitos mínimos que SAP expresa a sus proveedores de infraestructura. En escenarios reales de implementaciones de clientes y de cargas de trabajo, es posible que no sea suficiente. Utilice estas recomendaciones como punto de partida y adáptelas en función de los requisitos de la carga de trabajo específica.  

Por tanto, podría considerar la posibilidad de implementar un rendimiento similar para los volúmenes de ANF, tal como se indicó anteriormente en el almacenamiento de disco Ultra. Tenga en cuenta también los tamaños enumerados para los volúmenes de las diferentes SKU de máquina virtual como ya se ha hecho en las tablas de disco Ultra.

> [!TIP]
> Puede cambiar el tamaño de los volúmenes de Azure NetApp Files de manera dinámica, sin necesidad de `unmount` los volúmenes, detener las máquinas virtuales o detener SAP HANA. Esto permite que la aplicación se adapte a las demandas de rendimiento esperadas e imprevistas.

La documentación sobre cómo implementar una configuración de escalabilidad horizontal de SAP HANA con un nodo de espera mediante los volúmenes de NFS v4.1 hospedados en ANF y publicados en [Escalabilidad horizontal de SAP HANA con nodo en espera en máquinas virtuales de Azure con Azure NetApp Files en SUSE Linux Enterprise Server](./sap-hana-scale-out-standby-netapp-files-suse.md).


## <a name="next-steps"></a>Pasos siguientes
Para más información, consulte:

- [Guía sobre la alta disponibilidad de SAP HANA para máquinas virtuales de Azure](./sap-hana-availability-overview.md).
