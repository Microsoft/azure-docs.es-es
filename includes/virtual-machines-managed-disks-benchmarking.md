---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 01/11/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: da5811abec889bcc47d08878a0950df7f0983663
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87010839"
---
*Preparación de la memoria caché*  
El disco con almacenamiento en caché de host ReadOnly puede proporcionar un valor de IOPS mayor que el límite del disco. Para obtener este máximo rendimiento de lectura de la caché de host, primero debe preparar la memoria caché de este disco. Así se garantiza que las operaciones de E/S de lectura en las qué la herramienta de pruebas comparativas manejará el volumen de CacheReads alcanzan realmente la memoria caché y no el disco directamente. Los aciertos de caché generan IOPS adicionales desde el único disco con la memoria caché habilitada.

> [!IMPORTANT]
> debe preparar la memoria caché antes de ejecutar pruebas comparativas y cada vez que se reinicie la máquina virtual.

## <a name="tools"></a>Herramientas

### <a name="iometer"></a>Iometer

[Descargue la herramienta Iometer](https://sourceforge.net/projects/iometer/files/iometer-stable/2006-07-27/iometer-2006.07.27.win32.i386-setup.exe/download) en la máquina virtual.

#### <a name="test-file"></a>Archivo de prueba

Iometer usa un archivo de prueba que se almacena en el volumen en el que se ejecuta la prueba comparativa. Realiza lecturas y escrituras en el archivo de prueba para medir la IOPS y el rendimiento del disco. Iometer crea este archivo de prueba si no proporcionó ninguno. Cree un archivo de prueba de 200 GB llamado iobw.tst en los volúmenes CacheReads y NoCacheWrites.

#### <a name="access-specifications"></a>Especificaciones de acceso

Las especificaciones, el tamaño de la E/S de las solicitudes, % de lectura o escritura, % de acceso aleatorio o secuencial se configuran desde la pestaña "Access Specifications" (Especificaciones de acceso) de Iometer. Cree una especificación de acceso para cada uno de los escenarios descritos a continuación. Cree las especificaciones de acceso y "guárdelas" con un nombre apropiado como – RandomWrites\_8K o RandomReads\_8K. Seleccione la especificación correspondiente al ejecutar el escenario de prueba.

A continuación se muestra un ejemplo de especificaciones de acceso para el escenario de IOPS de escritura máxima:  
    ![Ejemplo de especificaciones de acceso para el valor máximo de IOPS de escritura](../articles/virtual-machines/linux/media/premium-storage-performance/image8.png)

#### <a name="maximum-iops-test-specifications"></a>Especificaciones de prueba de valor máximo de IOPS

Para demostrar el número máximo de E/S por segundo, use el tamaño de solicitud más pequeño. Use el tamaño de solicitud de 8K y cree especificaciones de lecturas y escrituras aleatorias.

| Especificación de acceso | Tamaño de la solicitud | % aleatorio | % lectura |
| --- | --- | --- | --- |
| RandomWrites\_8K |8 K |100 |0 |
| RandomReads\_8K |8 K |100 |100 |

#### <a name="maximum-throughput-test-specifications"></a>Especificaciones de prueba de rendimiento máximo

Para demostrar el rendimiento máximo, use el tamaño de la solicitud más grande. Use el tamaño de solicitud de 64 K y cree especificaciones de lecturas y escrituras aleatorias.

| Especificación de acceso | Tamaño de la solicitud | % aleatorio | % lectura |
| --- | --- | --- | --- |
| RandomWrites\_64K |64 K |100 |0 |
| RandomReads\_64K |64 K |100 |100 |

#### <a name="run-the-iometer-test"></a>Ejecución de la prueba Iometer

Realice los siguientes pasos para preparar la memoria caché

1. Cree dos especificaciones de acceso con los valores que se muestran a continuación:

   | Nombre | Tamaño de la solicitud | % aleatorio | % lectura |
   | --- | --- | --- | --- |
   | RandomWrites\_1MB |1 MB |100 |0 |
   | RandomReads\_1MB |1 MB |100 |100 |
1. Ejecute la prueba Iometer para inicializar el disco de la caché con los parámetros siguientes. Use tres subprocesos de trabajo para el volumen de destino y una profundidad de la cola de 128. Establezca la duración del “tiempo de ejecución” de la prueba en 2 horas en la pestaña "Test Setup" (Configuración de prueba).

   | Escenario | Volumen de destino | Nombre | Duration |
   | --- | --- | --- | --- |
   | Inicializar caché de disco |CacheReads |RandomWrites\_1MB |2 horas |
1. Ejecute la prueba Iometer para el preparar el disco de la caché con los parámetros siguientes. Use tres subprocesos de trabajo para el volumen de destino y una profundidad de la cola de 128. Establezca la duración del “tiempo de ejecución” de la prueba en 2 horas en la pestaña "Test Setup" (Configuración de prueba).

   | Escenario | Volumen de destino | Nombre | Duration |
   | --- | --- | --- | --- |
   | Preparación de la caché de disco |CacheReads |RandomReads\_1MB |2 horas |

Una vez preparado el disco de memoria caché, continúe con los escenarios de prueba que se muestran a continuación. Para ejecutar la prueba Iometer, use al menos tres subprocesos de trabajo para **cada** volumen de destino. Para cada subproceso de trabajo, seleccione el volumen de destino, establezca la profundidad de la cola y seleccione una de las especificaciones de prueba guardadas, tal como se muestra en la tabla siguiente, para ejecutar el escenario de prueba correspondiente. La tabla también muestra los resultados esperados para IOPS y rendimiento al ejecutar estas pruebas. Para todos los escenarios, se usa un tamaño pequeño de E/S de 8 KB y una profundidad de la cola alta de 128.

| Escenario de prueba | Volumen de destino | Nombre | Resultado |
| --- | --- | --- | --- |
| Máx. IOPS de lectura |CacheReads |RandomWrites\_8K |50.000 E/S por segundo |
| Máx. IOPS de escritura |NoCacheWrites |RandomReads\_8K |64.000 IOPS |
| Máx. IOPS combinado |CacheReads |RandomWrites\_8K |100.000 IOPS |
| NoCacheWrites |RandomReads\_8K | &nbsp; | &nbsp; |
| Máx. MB/s de lectura |CacheReads |RandomWrites\_64K |524 MB/s |
| Máx. Escritura MB/s |NoCacheWrites |RandomReads\_64K |524 MB/s |
| Combinado MB/s |CacheReads |RandomWrites\_64K |1000 MB/s |
| NoCacheWrites |RandomReads\_64K | &nbsp; | &nbsp; |

A continuación se muestran capturas de pantalla de los resultados de la prueba de Iometer para los escenarios IOPS y rendimiento combinados.

#### <a name="combined-reads-and-writes-maximum-iops"></a>Valor máximo de IOPS de lecturas y escrituras combinadas

![Valor máximo de IOPS de lecturas y escrituras combinadas](../articles/virtual-machines/linux/media/premium-storage-performance/image9.png)

#### <a name="combined-reads-and-writes-maximum-throughput"></a>Rendimiento máximo de lecturas y escrituras combinadas

![Rendimiento máximo de lecturas y escrituras combinadas](../articles/virtual-machines/linux/media/premium-storage-performance/image10.png)

### <a name="fio"></a>FIO

FIO es una popular herramienta para el almacenamiento de información de referencia en las máquinas virtuales de Linux. Tiene flexibilidad para seleccionar distintos tamaños de E/S y lecturas y escrituras secuenciales o aleatorias. Genera subprocesos de trabajo o procesos para realizar las operaciones de E/S especificadas. Puede especificar el tipo de operaciones de E/S que debe realizar cada subproceso de trabajo con archivos de trabajo. Hemos creado un archivo de trabajo por escenario que se ilustra en los ejemplos siguientes. Puede cambiar las especificaciones de estos archivos de trabajo para tener referencia de diferentes cargas de trabajo en Premium Storage. En los ejemplos, usamos una máquina virtual estándar 14 DS  que ejecuta **Ubuntu**. Use la misma configuración descrita al principio de la sección Pruebas comparativas y prepare la memoria caché antes de ejecutar dichas pruebas.

Antes de comenzar, [descargue FIO](https://github.com/axboe/fio) e instálelo en la máquina virtual.

Ejecute el siguiente comando para Ubuntu:

```
apt-get install fio
```

Usamos cuatro subprocesos de trabajo para realizar las operaciones de escritura y cuatro subprocesos de trabajo para realizar las operaciones de lectura en los discos. Los trabajos de escritura dirigen el tráfico del volumen "nocache", que tiene diez discos con la memoria caché establecida en "None". Los trabajos de lectura dirigen el tráfico del volumen "readcache", que tiene un disco con la memoria caché establecida en "ReadOnly".

#### <a name="maximum-write-iops"></a>Valor máximo de IOPS de escritura

Cree el archivo de trabajo con las especificaciones siguientes para obtener la IOPS de escritura máxima. Asígnele el nombre "fiowrite.ini".

```ini
[global]
size=30g
direct=1
iodepth=256
ioengine=libaio
bs=8k

[writer1]
rw=randwrite
directory=/mnt/nocache
[writer2]
rw=randwrite
directory=/mnt/nocache
[writer3]
rw=randwrite
directory=/mnt/nocache
[writer4]
rw=randwrite
directory=/mnt/nocache
```

Tenga en cuenta los siguientes aspectos clave que están en consonancia con las instrucciones de diseño que se tratan en secciones anteriores. Estas especificaciones son esenciales para alcanzar la IOPS máxima  

* Una profundidad de la cola alta de 256.  
* Un tamaño de bloque pequeño de 8 KB.  
* Varios subprocesos que realizan escrituras aleatorias.

Ejecute el siguiente comando para ejecutar la prueba FIO durante 30 segundos:  

```
sudo fio --runtime 30 fiowrite.ini
```

Mientras se ejecuta la prueba, puede ver el número de IOPS de escritura que envían la máquina virtual y los discos Premium. Como se muestra en el ejemplo siguiente, la máquina virtual DS14 está ofreciendo su límite máximo de IOPS de escritura: 50.000 IOPS.  
    ![El número de discos Premium y de VM de IOPS que se entregan.](../articles/virtual-machines/linux/media/premium-storage-performance/image11.png)

#### <a name="maximum-read-iops"></a>Valor máximo de IOPS de lectura

Cree el archivo de trabajo con las especificaciones siguientes para obtener la IOPS de lectura máxima. Asígnele el nombre "fioread.ini".

```ini
[global]
size=30g
direct=1
iodepth=256
ioengine=libaio
bs=8k

[reader1]
rw=randread
directory=/mnt/readcache
[reader2]
rw=randread
directory=/mnt/readcache
[reader3]
rw=randread
directory=/mnt/readcache
[reader4]
rw=randread
directory=/mnt/readcache
```

Tenga en cuenta los siguientes aspectos clave que están en consonancia con las instrucciones de diseño que se tratan en secciones anteriores. Estas especificaciones son esenciales para alcanzar la IOPS máxima

* Una profundidad de la cola alta de 256.  
* Un tamaño de bloque pequeño de 8 KB.  
* Varios subprocesos que realizan escrituras aleatorias.

Ejecute el siguiente comando para ejecutar la prueba FIO durante 30 segundos:

```
sudo fio --runtime 30 fioread.ini
```

Mientras se ejecuta la prueba, puede ver el número de IOPS de lectura que envían los discos Premium y de VM. Como se muestra en el ejemplo siguiente, la máquina virtual DS14 proporciona más de 64.000 IOPS de lectura. Se trata de una combinación del rendimiento de la caché y el disco.  
    ![Captura de pantalla del número de discos Premium y de VM de IOPS que se entregan.](../articles/virtual-machines/linux/media/premium-storage-performance/image12.png)

#### <a name="maximum-read-and-write-iops"></a>Valor máximo de IOPS de lectura y escritura

 Cree el archivo de trabajo con las especificaciones siguientes para obtener la IOPS de lectura y escritura combinadas máxima. Asígnele el nombre "fioreadwrite.ini".

```ini
[global]
size=30g
direct=1
iodepth=128
ioengine=libaio
bs=4k

[reader1]
rw=randread
directory=/mnt/readcache
[reader2]
rw=randread
directory=/mnt/readcache
[reader3]
rw=randread
directory=/mnt/readcache
[reader4]
rw=randread
directory=/mnt/readcache

[writer1]
rw=randwrite
directory=/mnt/nocache
rate_iops=12500
[writer2]
rw=randwrite
directory=/mnt/nocache
rate_iops=12500
[writer3]
rw=randwrite
directory=/mnt/nocache
rate_iops=12500
[writer4]
rw=randwrite
directory=/mnt/nocache
rate_iops=12500
```

Tenga en cuenta los siguientes aspectos clave que están en consonancia con las instrucciones de diseño que se tratan en secciones anteriores. Estas especificaciones son esenciales para alcanzar la IOPS máxima

* Una profundidad de la cola alta de 128.  
* Un tamaño de bloque pequeño de 4 KB.  
* Varios subprocesos que realizan lecturas y escrituras aleatorias.

Ejecute el siguiente comando para ejecutar la prueba FIO durante 30 segundos:

```
sudo fio --runtime 30 fioreadwrite.ini
```

Mientras se ejecuta la prueba, puede ver el número de IOPS de lectura y escritura combinadas que envían la máquina virtual y los discos Premium. Como se muestra en el ejemplo siguiente, la máquina virtual DS14 proporciona más de 100.000 IOPS de lectura y escritura combinadas. Se trata de una combinación del rendimiento de la caché y el disco.  
    ![Valor de IOPS de lectura y escritura combinadas](../articles/virtual-machines/linux/media/premium-storage-performance/image13.png)

#### <a name="maximum-combined-throughput"></a>Rendimiento máximo combinado

 Para obtener el rendimiento de lectura y escritura combinado máximo, use un tamaño de bloque y la profundidad de la cola más grandes con varios subprocesos que realizan lecturas y escrituras. Puede usar un tamaño de bloque de 64 KB y una profundidad de la cola de 128.
