---
title: 'Ingesta de datos de Azure HPC Cache: copia manual'
description: Uso de comandos cp para trasladar datos a un destino de almacenamiento de Azure Blob Storage en Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 10/30/2019
ms.author: v-erkel
ms.openlocfilehash: f96a0fa264124f9d050e667b003d98579da63b77
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87092344"
---
# <a name="azure-hpc-cache-data-ingest---manual-copy-method"></a>Ingesta de datos de Azure HPC Cache: método de copia manual

En este artículo se proporcionan instrucciones detalladas para copiar manualmente los datos en un contenedor de Azure Blob Storage y usarlos con Azure HPC Cache. Se utilizan operaciones paralelas multiproceso para optimizar la velocidad de copia.

Para más información sobre cómo mover datos a Blob Storage para Azure HPC Cache, consulte [Traslado de datos a Azure Blob Storage](hpc-cache-ingest.md).

## <a name="simple-copy-example"></a>Ejemplo de copia simple

Puede crear una copia de varios subprocesos de forma manual en un cliente; para ello, ejecute más de un comando de copia a la vez en segundo plano en los conjuntos predefinidos de archivos o rutas.

El comando ``cp`` de Linux/UNIX incluye el argumento ``-p`` que se usa para conservar la propiedad y los metadatos de tipo "mtime". Agregar este argumento a los comandos que tiene a continuación es opcional. Recuerde que si agrega el argumento aumentará a su vez la cantidad de llamadas del sistema de archivos que se envían desde el cliente al sistema de archivos de destino para la modificación de metadatos.

En este simple ejemplo se copian dos archivos en paralelo:

```bash
cp /mnt/source/file1 /mnt/destination1/ & cp /mnt/source/file2 /mnt/destination1/ &
```

Después de emitir este comando, el comando `jobs` mostrará que se están ejecutando dos subprocesos.

## <a name="copy-data-with-predictable-file-names"></a>Copia de datos con nombres de archivo predecibles

Si los nombres de archivo son predecibles, puede usar expresiones para crear subprocesos de copia paralelos.

Por ejemplo, si su directorio contiene 1000 archivos que están numerados secuencialmente de `0001` a `1000`, puede usar las siguientes expresiones para crear diez subprocesos paralelos para que cada uno copie cien archivos:

```bash
cp /mnt/source/file0* /mnt/destination1/ & \
cp /mnt/source/file1* /mnt/destination1/ & \
cp /mnt/source/file2* /mnt/destination1/ & \
cp /mnt/source/file3* /mnt/destination1/ & \
cp /mnt/source/file4* /mnt/destination1/ & \
cp /mnt/source/file5* /mnt/destination1/ & \
cp /mnt/source/file6* /mnt/destination1/ & \
cp /mnt/source/file7* /mnt/destination1/ & \
cp /mnt/source/file8* /mnt/destination1/ & \
cp /mnt/source/file9* /mnt/destination1/
```

## <a name="copy-data-with-unstructured-file-names"></a>Copia de datos con nombres de archivo no estructurados

Si la estructura de nomenclatura de archivos no es predecible, puede agrupar los archivos en función de los nombres de directorio.

En este ejemplo se recopilan los directorios completos que se enviarán a los comandos ``cp`` que se ejecutan como tareas en segundo plano:

```bash
/root
|-/dir1
| |-/dir1a
| |-/dir1b
| |-/dir1c
   |-/dir1c1
|-/dir1d
```

Después de recopilar los archivos, puede ejecutar comandos de copia en paralelo para copiar de forma recursiva los subdirectorios y todo su contenido:

```bash
cp /mnt/source/* /mnt/destination/
mkdir -p /mnt/destination/dir1 && cp /mnt/source/dir1/* mnt/destination/dir1/ &
cp -R /mnt/source/dir1/dir1a /mnt/destination/dir1/ &
cp -R /mnt/source/dir1/dir1b /mnt/destination/dir1/ &
cp -R /mnt/source/dir1/dir1c /mnt/destination/dir1/ & # this command copies dir1c1 via recursion
cp -R /mnt/source/dir1/dir1d /mnt/destination/dir1/ &
```

## <a name="when-to-add-mount-points"></a>Cuándo agregar puntos de montaje

Una vez que haya suficientes subprocesos paralelos en un único punto de montaje del sistema de archivos de destino, llegará a un punto en el que el simple hecho de agregar más subprocesos no proporcionará más rendimiento. (El rendimiento se medirá en archivos/segundo o bytes/segundo, según el tipo de datos). O, lo que es peor, el exceso de subprocesos puede causar una degradación del rendimiento.

Si le sucede esto, puede agregar puntos de montaje del lado cliente a otras direcciones de montaje de Azure HPC Cache; para ello, use la misma ruta de montaje remota del sistema de archivos:

```bash
10.1.0.100:/nfs on /mnt/sourcetype nfs (rw,vers=3,proto=tcp,addr=10.1.0.100)
10.1.1.101:/nfs on /mnt/destination1type nfs (rw,vers=3,proto=tcp,addr=10.1.1.101)
10.1.1.102:/nfs on /mnt/destination2type nfs (rw,vers=3,proto=tcp,addr=10.1.1.102)
10.1.1.103:/nfs on /mnt/destination3type nfs (rw,vers=3,proto=tcp,addr=10.1.1.103)
```

Si decide agregar puntos de montaje del lado cliente, podrá bifurcar comandos de copia adicionales a los puntos de montaje `/mnt/destination[1-3]` adicionales; gracias a ello logrará un mayor paralelismo.

Por ejemplo, si sus archivos son muy grandes, puede definir los comandos de copia para usar distintas rutas de acceso de destino, y así poder enviar más comandos en paralelo desde el cliente que realiza la copia.

```bash
cp /mnt/source/file0* /mnt/destination1/ & \
cp /mnt/source/file1* /mnt/destination2/ & \
cp /mnt/source/file2* /mnt/destination3/ & \
cp /mnt/source/file3* /mnt/destination1/ & \
cp /mnt/source/file4* /mnt/destination2/ & \
cp /mnt/source/file5* /mnt/destination3/ & \
cp /mnt/source/file6* /mnt/destination1/ & \
cp /mnt/source/file7* /mnt/destination2/ & \
cp /mnt/source/file8* /mnt/destination3/ & \
```

En el ejemplo anterior, los tres procesos de copia de archivos del cliente se encargan de administrar los tres puntos de montaje de destino.

## <a name="when-to-add-clients"></a>Cuándo agregar clientes

Por último, cuando haya alcanzado las capacidades del cliente, si decide agregar más subprocesos de copia o puntos de montaje adicionales no se producirá ningún aumento adicional de archivos/segundo o bytes/segundo. En esa situación, puede implementar otro cliente con el mismo conjunto de puntos de montaje; este ejecutará su propio conjuntos de procesos de copia de archivos.

Ejemplo:

```bash
Client1: cp -R /mnt/source/dir1/dir1a /mnt/destination/dir1/ &
Client1: cp -R /mnt/source/dir2/dir2a /mnt/destination/dir2/ &
Client1: cp -R /mnt/source/dir3/dir3a /mnt/destination/dir3/ &

Client2: cp -R /mnt/source/dir1/dir1b /mnt/destination/dir1/ &
Client2: cp -R /mnt/source/dir2/dir2b /mnt/destination/dir2/ &
Client2: cp -R /mnt/source/dir3/dir3b /mnt/destination/dir3/ &

Client3: cp -R /mnt/source/dir1/dir1c /mnt/destination/dir1/ &
Client3: cp -R /mnt/source/dir2/dir2c /mnt/destination/dir2/ &
Client3: cp -R /mnt/source/dir3/dir3c /mnt/destination/dir3/ &

Client4: cp -R /mnt/source/dir1/dir1d /mnt/destination/dir1/ &
Client4: cp -R /mnt/source/dir2/dir2d /mnt/destination/dir2/ &
Client4: cp -R /mnt/source/dir3/dir3d /mnt/destination/dir3/ &
```

## <a name="create-file-manifests"></a>Crear manifiestos de archivos

Después de comprender la información anterior (esto es, varios subprocesos de copia por destino, varios destinos por cliente, varios clientes por sistema de archivos de origen accesible a través de la red), tenga en cuenta esta recomendación: compile manifiestos de archivo y úselos con comandos de copia en varios clientes.

En este escenario se usa el comando UNIX ``find`` para crear manifiestos de archivos o directorios:

```bash
user@build:/mnt/source > find . -mindepth 4 -maxdepth 4 -type d
./atj5b55c53be6-01/support/gsi/2018-07-22T21:12:06EDT
./atj5b55c53be6-01/support/pcap/2018-07-23T01:34:57UTC
./atj5b55c53be6-01/support/trace/rolling
./atj5b55c53be6-03/support/gsi/2018-07-22T21:12:06EDT
./atj5b55c53be6-03/support/pcap/2018-07-23T01:34:57UTC
./atj5b55c53be6-03/support/trace/rolling
./atj5b55c53be6-02/support/gsi/2018-07-22T21:12:06EDT
./atj5b55c53be6-02/support/pcap/2018-07-23T01:34:57UTC
./atj5b55c53be6-02/support/trace/rolling
```

Redirija este resultado a un archivo `find . -mindepth 4 -maxdepth 4 -type d > /tmp/foo`.

A continuación, puede iterar el manifiesto mediante comandos BASH para contar los archivos y determinar los tamaños de los subdirectorios:

```bash
ben@xlcycl1:/sps/internal/atj5b5ab44b7f > for i in $(cat /tmp/foo); do echo " `find ${i} |wc -l` `du -sh ${i}`"; done
244    3.5M    ./atj5b5ab44b7f-02/support/gsi/2018-07-18T00:07:03EDT
9      172K    ./atj5b5ab44b7f-02/support/gsi/stats_2018-07-18T05:01:00UTC
124    5.8M    ./atj5b5ab44b7f-02/support/gsi/stats_2018-07-19T01:01:01UTC
152    15M     ./atj5b5ab44b7f-02/support/gsi/stats_2018-07-20T01:01:00UTC
131    13M     ./atj5b5ab44b7f-02/support/gsi/stats_2018-07-20T21:59:41UTC_partial
789    6.2M    ./atj5b5ab44b7f-02/support/gsi/2018-07-20T21:59:41UTC
134    12M     ./atj5b5ab44b7f-02/support/gsi/stats_2018-07-20T22:22:55UTC_hpccache_catchup
7      16K     ./atj5b5ab44b7f-02/support/pcap/2018-07-18T17:12:19UTC
8      83K     ./atj5b5ab44b7f-02/support/pcap/2018-07-18T17:17:17UTC
575    7.7M    ./atj5b5ab44b7f-02/support/cores/armada_main.2000.1531980253.gsi
33     4.4G    ./atj5b5ab44b7f-02/support/trace/rolling
281    6.6M    ./atj5b5ab44b7f-01/support/gsi/2018-07-18T00:07:03EDT
15     182K    ./atj5b5ab44b7f-01/support/gsi/stats_2018-07-18T05:01:00UTC
244    17M     ./atj5b5ab44b7f-01/support/gsi/stats_2018-07-19T01:01:01UTC
299    31M     ./atj5b5ab44b7f-01/support/gsi/stats_2018-07-20T01:01:00UTC
256    29M     ./atj5b5ab44b7f-01/support/gsi/stats_2018-07-20T21:59:41UTC_partial
889    7.7M    ./atj5b5ab44b7f-01/support/gsi/2018-07-20T21:59:41UTC
262    29M     ./atj5b5ab44b7f-01/support/gsi/stats_2018-07-20T22:22:55UTC_hpccache_catchup
11     248K    ./atj5b5ab44b7f-01/support/pcap/2018-07-18T17:12:19UTC
11     88K     ./atj5b5ab44b7f-01/support/pcap/2018-07-18T17:17:17UTC
645    11M     ./atj5b5ab44b7f-01/support/cores/armada_main.2019.1531980253.gsi
33     4.0G    ./atj5b5ab44b7f-01/support/trace/rolling
244    2.1M    ./atj5b5ab44b7f-03/support/gsi/2018-07-18T00:07:03EDT
9      158K    ./atj5b5ab44b7f-03/support/gsi/stats_2018-07-18T05:01:00UTC
124    5.3M    ./atj5b5ab44b7f-03/support/gsi/stats_2018-07-19T01:01:01UTC
152    15M     ./atj5b5ab44b7f-03/support/gsi/stats_2018-07-20T01:01:00UTC
131    12M     ./atj5b5ab44b7f-03/support/gsi/stats_2018-07-20T21:59:41UTC_partial
789    8.4M    ./atj5b5ab44b7f-03/support/gsi/2018-07-20T21:59:41UTC
134    14M     ./atj5b5ab44b7f-03/support/gsi/stats_2018-07-20T22:25:58UTC_hpccache_catchup
7      159K    ./atj5b5ab44b7f-03/support/pcap/2018-07-18T17:12:19UTC
7      157K    ./atj5b5ab44b7f-03/support/pcap/2018-07-18T17:17:17UTC
576    12M     ./atj5b5ab44b7f-03/support/cores/armada_main.2013.1531980253.gsi
33     2.8G    ./atj5b5ab44b7f-03/support/trace/rolling
```

Por último, debe crear los comandos de copia de archivos reales para los clientes.

Si tiene cuatro clientes, use este comando:

```bash
for i in 1 2 3 4 ; do sed -n ${i}~4p /tmp/foo > /tmp/client${i}; done
```

Si tiene cinco clientes, use algo como esto:

```bash
for i in 1 2 3 4 5; do sed -n ${i}~5p /tmp/foo > /tmp/client${i}; done
```

Y para seis... Extrapole los datos según sea necesario.

```bash
for i in 1 2 3 4 5 6; do sed -n ${i}~6p /tmp/foo > /tmp/client${i}; done
```

Obtendrá *N* archivos resultantes, uno para cada uno de sus *N* clientes que tengan los nombres de ruta de acceso a los directorios de nivel cuatro y que se obtuvieron como parte de la salida del comando `find`.

Use cada archivo para construir el comando de copia:

```bash
for i in 1 2 3 4 5 6; do for j in $(cat /tmp/client${i}); do echo "cp -p -R /mnt/source/${j} /mnt/destination/${j}" >> /tmp/client${i}_copy_commands ; done; done
```

La información anterior le proporcionará *N* archivos, cada uno con un comando de copia por línea que se puede ejecutar como un script BASH en el cliente.

El objetivo es ejecutar varios subprocesos de estos scripts simultáneamente por cliente y en paralelo en varios clientes.
