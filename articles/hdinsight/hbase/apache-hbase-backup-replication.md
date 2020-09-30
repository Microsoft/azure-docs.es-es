---
title: 'Copia de seguridad y replicación para Apache HBase y Phoenix: Azure HDInsight'
description: Configuración de la copia de seguridad y la replicación de Apache HBase y Apache Phoenix en Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/19/2019
ms.openlocfilehash: 5c0694f9ef16de9c69d424b5005ca0d5a277a77f
ms.sourcegitcommit: 59ea8436d7f23bee75e04a84ee6ec24702fb2e61
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/07/2020
ms.locfileid: "89505036"
---
# <a name="set-up-backup-and-replication-for-apache-hbase-and-apache-phoenix-on-hdinsight"></a>Configuración de la copia de seguridad y la replicación de Apache HBase y Apache Phoenix en HDInsight

Apache HBase admite varios enfoques para protegerse frente a la pérdida de datos:

* Copia de la carpeta `hbase`
* Exportación y posterior importación
* Copia de las tablas
* Instantáneas
* Replicación

> [!NOTE]  
> Apache Phoenix almacena sus metadatos en tablas HBase, por lo que se crea una copia de seguridad de los metadatos cuando se crea una copia de seguridad de las tablas de catálogos del sistema HBase.

En las secciones siguientes se describe el escenario de uso de cada uno de estos enfoques.

## <a name="copy-the-hbase-folder"></a>Copia de la carpeta hbase

Con este enfoque, se copian todos los datos de HBase sin poder seleccionar un subconjunto de tablas o familias de columnas. Los enfoques posteriores proporcionan un mayor control.

HBase en HDInsight usa el almacenamiento predeterminado seleccionado al crear el clúster, ya sean blobs de Azure Storage o Azure Data Lake Storage. Cualquiera sea el caso, HBase almacena sus archivos de datos y metadatos en la ruta de acceso siguiente:

`/hbase`

* En una cuenta de Azure Storage, la carpeta `hbase` reside en la raíz del contenedor de blobs:

  `wasbs://<containername>@<accountname>.blob.core.windows.net/hbase`

* En Azure Data Lake Storage, la carpeta `hbase` reside en la ruta de acceso raíz que especificó al aprovisionar un clúster. Esta ruta de acceso raíz habitualmente tiene una carpeta `clusters`, con una subcarpeta con el nombre del clúster HDInsight:

  `/clusters/<clusterName>/hbase`

En cualquier caso, la carpeta `hbase` contiene todos los datos que HBase ha vaciado en el disco, pero no puede contener los datos en memoria. Antes de que pueda confiar en esta carpeta como una representación exacta de los datos de HBase, debe cerrar el clúster.

Una vez que elimina el clúster, puede dejar los datos en su lugar o copiarlos a una ubicación nueva:

* Cree una instancia de HDInsight que apunte a la ubicación de almacenamiento actual. La instancia nueva se crea con todos los datos existentes.

* Copie la carpeta `hbase` a un contenedor de blobs de Azure Storage o a una ubicación de Data Lake Storage diferente y luego inicie un clúster nuevo con esos datos. Para Azure Storage use [AzCopy](../../storage/common/storage-use-azcopy.md) y para Data Lake Storage, [AdlCopy](../../data-lake-store/data-lake-store-copy-data-azure-storage-blob.md).

## <a name="export-then-import"></a>Exportación y posterior importación

En el clúster de HDInsight de origen, use la [Utilidad de exportación](https://hbase.apache.org/book.html#export) (que se incluye con HBase) para exportar datos desde una tabla de origen al almacenamiento conectado predeterminado. Luego, puede copiar la carpeta exportada a la ubicación de almacenamiento de destino y ejecutar la [Utilidad de importación](https://hbase.apache.org/book.html#import) en el clúster de HDInsight de destino.

Para exportar los datos de una tabla, primero establezca una conexión SSH al nodo principal del clúster de HDInsight de origen y luego ejecute el comando `hbase` siguiente:

```console
hbase org.apache.hadoop.hbase.mapreduce.Export "<tableName>" "/<path>/<to>/<export>"
```

El directorio de exportación no debe existir. El nombre de la tabla distingue mayúsculas de minúsculas.

Para importar los datos de una tabla, establezca una conexión SSH al nodo principal del clúster de HDInsight de destino y luego ejecute el comando `hbase` siguiente:

```console
hbase org.apache.hadoop.hbase.mapreduce.Import "<tableName>" "/<path>/<to>/<export>"
```

La tabla ya debe existir.

Especifique la ruta de acceso de exportación completa al almacenamiento predeterminado o a cualquiera de las opciones del almacenamiento conectado. Por ejemplo, en Azure Storage:

`wasbs://<containername>@<accountname>.blob.core.windows.net/<path>`

En Azure Data Lake Storage Gen2, la sintaxis es:

`abfs://<containername>@<accountname>.dfs.core.windows.net/<path>`

En Azure Data Lake Storage Gen1, la sintaxis es:

`adl://<accountName>.azuredatalakestore.net:443/<path>`

Este enfoque ofrece granularidad de nivel de tabla. También puede especificar un intervalo de fechas para las filas que se van a incluir, lo que permite realizar el proceso de manera incremental. Cada fecha se expresa en milisegundos desde la época de Unix.

```console
hbase org.apache.hadoop.hbase.mapreduce.Export "<tableName>" "/<path>/<to>/<export>" <numberOfVersions> <startTimeInMS> <endTimeInMS>
```

Tenga en cuenta que debe especificar el número de versiones de cada fila que se va a exportar. Para incluir todas las versiones en el intervalo de fechas, establezca `<numberOfVersions>` en un valor mayor que el máximo posible de versiones de fila, como 100 000.

## <a name="copy-tables"></a>Copia de las tablas

La [utilidad CopyTable](https://hbase.apache.org/book.html#copy.table) copia datos desde una tabla de origen, fila por fila, a una tabla de destino existente con el mismo esquema que la de origen. La tabla de destino se puede abrir en el mismo clúster o en un clúster de HBase distinto. Los nombres de tabla distinguen mayúsculas de minúsculas.

Para usar CopyTable dentro de un clúster, establezca una conexión SSH al nodo principal del clúster de HDInsight de origen y luego ejecute este comando `hbase`:

```console
hbase org.apache.hadoop.hbase.mapreduce.CopyTable --new.name=<destTableName> <srcTableName>
```

Para usar CopyTable para copiar en una tabla de un clúster distinto, agregue el modificador `peer` con la dirección del clúster de destino:

```console
hbase org.apache.hadoop.hbase.mapreduce.CopyTable --new.name=<destTableName> --peer.adr=<destinationAddress> <srcTableName>
```

La dirección de destino se compone de estas tres partes:

`<destinationAddress> = <ZooKeeperQuorum>:<Port>:<ZnodeParent>`

* `<ZooKeeperQuorum>` es una lista de nodos de Apache ZooKeeper separada por comas; por ejemplo:

    zk0-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net,zk4-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net,zk3-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net

* `<Port>` en HDInsight tiene 2181 como valor predeterminado y `<ZnodeParent>` es `/hbase-unsecure`, por lo que el valor `<destinationAddress>` completo sería:

    zk0-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net,zk4-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net,zk3-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net:2181:/hbase-unsecure

Consulte la sección [Recopilación manual de la lista de cuórum de Apache ZooKeeper](#manually-collect-the-apache-zookeeper-quorum-list) de este artículo para conocer los detalles acerca de cómo recuperar estos valores para el clúster de HDInsight.

La utilidad CopyTable también admite parámetros para especificar el intervalo de tiempo de las filas que se copiarán y para especificar el subconjunto de familias de columnas en una tabla que se copiará. Para ver la lista completa de los parámetros compatibles con CopyTable, ejecute CopyTable sin ningún parámetro:

```console
hbase org.apache.hadoop.hbase.mapreduce.CopyTable
```

CopyTable examina todo el contenido de la tabla de origen que se copiará a la tabla de destino. Este proceso puede disminuir el rendimiento del clúster de HBase mientras se ejecuta CopyTable.

> [!NOTE]  
> Para automatizar la copia de los datos entre tablas, consulte el script `hdi_copy_table.sh` en el repositorio [Azure HBase Utils](https://github.com/Azure/hbase-utils/tree/master/replication) de GitHub.

### <a name="manually-collect-the-apache-zookeeper-quorum-list"></a>Recopilación manual de la lista de cuórum de Apache ZooKeeper

Cuando ambos clústeres de HDInsight están en la misma red virtual, como se describe anteriormente, la resolución de nombres de host interno es automática. Para usar CopyTable para los clústeres de HDInsight en dos redes virtuales independientes conectadas por una puerta de enlace de VPN, deberá proporcionar las direcciones IP del host de los nodos ZooKeeper en el cuórum.

Para adquirir los nombres de host del cuórum, ejecute el comando curl siguiente:

```console
curl -u admin:<password> -X GET -H "X-Requested-By: ambari" "https://<clusterName>.azurehdinsight.net/api/v1/clusters/<clusterName>/configurations?type=hbase-site&tag=TOPOLOGY_RESOLVED" | grep "hbase.zookeeper.quorum"
```

El comando curl recupera un documento JSON con información de la configuración de HBase y el comando grep solo devuelve la entrada "hbase.zookeeper.quorum", por ejemplo:

```output
"hbase.zookeeper.quorum" : "zk0-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net,zk4-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net,zk3-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net"
```

El valor de los nombres de host del cuórum es toda la cadena que aparece a la derecha de los dos puntos.

Para recuperar las direcciones IP de estos hosts, use el comando curl siguiente para cada host de la lista anterior:

```console
curl -u admin:<password> -X GET -H "X-Requested-By: ambari" "https://<clusterName>.azurehdinsight.net/api/v1/clusters/<clusterName>/hosts/<zookeeperHostFullName>" | grep "ip"
```

En este comando curl, `<zookeeperHostFullName>` es el nombre DNS completo de un host de ZooKeeper, como el ejemplo `zk0-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net`. La salida del comando contiene la dirección IP del host especificado, por ejemplo:

`100    "ip" : "10.0.0.9",`

Después de recopilar las direcciones IP de todos los nodos de ZooKeeper en el cuórum, vuelva a crear la dirección de destino:

`<destinationAddress>  = <Host_1_IP>,<Host_2_IP>,<Host_3_IP>:<Port>:<ZnodeParent>`

En el ejemplo:

`<destinationAddress> = 10.0.0.9,10.0.0.8,10.0.0.12:2181:/hbase-unsecure`

## <a name="snapshots"></a>Instantáneas

Las [instantáneas](https://hbase.apache.org/book.html#ops.snapshots) le permiten crear una copia de seguridad de los datos de un momento dato en el almacén de datos de HBase. Las instantáneas tienen una sobrecarga mínima y se completan en cuestión de segundos, porque una operación de instantánea es efectivamente una operación de metadatos que captura los nombres de todos los archivos que están en almacenamiento en ese instante. En el momento de una instantánea, no se copia ningún dato real. Las instantáneas se basan en la naturaleza inalterable de los datos almacenados en HDFS, en que todas las actualizaciones, eliminaciones e inserciones se representan como datos nuevos. Puede restaurar (*clonar*) una instantánea en el mismo clúster o exportar una instantánea a otro clúster.

Para crear una instantánea, establezca una conexión SSH al nodo principal del clúster de HDInsight HBase e inicie el shell `hbase`:

```console
hbase shell
```

Dentro del shell hbase, use el comando snapshot con los nombres de la tabla y de esta instantánea:

```console
snapshot '<tableName>', '<snapshotName>'
```

Para restaurar una instantánea por nombre dentro del shell `hbase`, primero deshabilite la tabla, luego restaure la instantánea y vuelva a habilitar la tabla:

```console
disable '<tableName>'
restore_snapshot '<snapshotName>'
enable '<tableName>'
```

Para restaurar una instantánea en una tabla nueva, use clone_snapshot:

```console
clone_snapshot '<snapshotName>', '<newTableName>'
```

Para exportar una instantánea a HDFS para que la use otro clúster, primero cree las instantáneas como se describió previamente y luego use la utilidad ExportSnapshot. Ejecute esta utilidad desde dentro de la sesión SSH al nodo principal y no dentro del shell `hbase`:

```console
hbase org.apache.hadoop.hbase.snapshot.ExportSnapshot -snapshot <snapshotName> -copy-to <hdfsHBaseLocation>
```

`<hdfsHBaseLocation>` puede ser cualquiera de las ubicaciones de almacenamiento accesibles para el clúster de origen y debe apuntar a la carpeta hbase que el clúster de destino usa. Por ejemplo, si tiene una cuenta de Azure Storage secundaria conectada al clúster de origen y esa cuenta proporciona acceso al contenedor que se usa en el almacenamiento predeterminado del clúster de destino, puede usar este comando:

```console
hbase org.apache.hadoop.hbase.snapshot.ExportSnapshot -snapshot 'Snapshot1' -copy-to 'wasbs://secondcluster@myaccount.blob.core.windows.net/hbase'
```

Si no tiene una cuenta de Azure Storage secundaria conectada al clúster de origen o si el clúster de origen es un clúster local (o un clúster que no es de HDI), es posible que experimente problemas de autorización al intentar acceder a la cuenta de almacenamiento del clúster de HDI. Para resolver este problema, especifique la clave de la cuenta de almacenamiento como un parámetro de la línea de comandos, tal como se muestra en el ejemplo siguiente. Puede obtener la clave de la cuenta de almacenamiento en Azure Portal.

```console
hbase org.apache.hadoop.hbase.snapshot.ExportSnapshot -Dfs.azure.account.key.myaccount.blob.core.windows.net=mykey -snapshot 'Snapshot1' -copy-to 'wasbs://secondcluster@myaccount.blob.core.windows.net/hbase'
```

Una vez que se exporta la instantánea, establezca una conexión SSH al nodo principal del clúster de destino y restaure la instantánea con el comando `restore_snapshot`, tal como se describió anteriormente.

Las instantáneas proporcionan una copia de seguridad completa de una tabla en el momento de ejecutar el comando `snapshot`. Las instantáneas no proporcionan la capacidad de realizar instantáneas incrementales por ventanas de tiempo ni especificar subconjuntos de familias de columnas para incluir en la instantánea.

## <a name="replication"></a>Replicación

La [replicación de HBase](https://hbase.apache.org/book.html#_cluster_replication) inserta de manera automática las transacciones provenientes de un clúster de origen a un clúster de destino, mediante un mecanismo asincrónico con un mínimo de sobrecarga sobre el clúster de origen. En HDInsight, puede configurar la replicación entre los clústeres donde:

* Los clústeres de origen y destino están en la misma red virtual.
* Los clústeres de origen y destino están en distintas redes virtuales conectadas por una instancia de VPN Gateway, pero ambos clústeres se encuentran en la misma ubicación geográfica.
* El clúster de origen y los clústeres de destinos están en redes virtuales distintas conectadas por una instancia de VPN Gateway y cada clúster se encuentra en una ubicación geográfica distinta.

Los pasos generales para configurar la replicación son:

1. En el clúster de origen, cree las tablas y rellene los datos.
2. En el clúster de destino, cree tablas de destino vacías con el esquema de la tabla de origen.
3. Registre el clúster de destino como un elemento del mismo nivel en el clúster de origen.
4. Habilite la replicación en las tablas de destino deseadas.
5. Copie los datos existentes de las tablas de origen a las tablas de destino.
6. La replicación copia automáticamente las modificaciones nuevas de los datos de las tablas de origen en las tablas de destino.

Para habilitar la replicación en HDInsight, aplique una acción de script al clúster de HDInsight de origen en ejecución. Para obtener instrucciones acerca de cómo habilitar la replicación en el clúster o experimentar con la replicación en clústeres de ejemplo creados en máquinas virtuales con las plantillas de Azure Resource Manager, consulte el artículo sobre la [configuración de la replicación de Apache HBase](apache-hbase-replication.md). Ese artículo también incluye las instrucciones para habilitar la replicación de los metadatos de Phoenix.

## <a name="next-steps"></a>Pasos siguientes

* [Configuración de la replicación de Apache HBase](apache-hbase-replication.md)
* [Trabajo con la utilidad de importación y exportación de HBase](https://blogs.msdn.microsoft.com/data_otaku/2016/12/21/working-with-the-hbase-import-and-export-utility/)
