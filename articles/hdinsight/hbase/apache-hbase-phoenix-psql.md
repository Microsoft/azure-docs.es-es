---
title: 'Carga masiva en Phoenix Apache con psql: Azure HDInsight'
description: Utilice la herramienta psql para cargar datos de forma masiva en tablas de Apache Phoenix en Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/17/2019
ms.openlocfilehash: 513d48114d73b2c91ebecf06f9492a6ebf0ba1cf
ms.sourcegitcommit: 59ea8436d7f23bee75e04a84ee6ec24702fb2e61
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/07/2020
ms.locfileid: "89504713"
---
# <a name="bulk-load-data-into-apache-phoenix-using-psql"></a>Carga masiva de datos en Apache Phoenix mediante psql

[Apache Phoenix](https://phoenix.apache.org/) es una base de datos relacional de código abierto y masivamente paralela que se basa en [Apache HBase](../hbase/apache-hbase-overview.md). Phoenix proporciona consultas de tipo SQL a través de HBase. Phoenix usa controladores JDBC para permitir a los usuarios crear, eliminar y modificar índices, vistas, secuencias y tablas SQL, así como realizar operaciones de upsert en filas de forma individual o masiva. Phoenix utiliza la compilación nativa de NoSQL (en lugar de usar MapReduce para compilar consultas) para crear aplicaciones de baja latencia a partir de HBase. Phoenix agrega coprocesadores para admitir la ejecución de código proporcionado por clientes en el espacio de direcciones del servidor, ejecutando el código colocado con los datos. Esto minimiza la transferencia de datos de cliente/servidor.  Para trabajar con datos mediante Phoenix en HDInsight, primero debe crear tablas y, después, cargar datos en ellas.

## <a name="bulk-loading-with-apache-phoenix"></a>Carga de datos masiva con Apache Phoenix

Hay varias maneras de obtener datos en HBase, incluido el uso de API de cliente, un trabajo MapReduce con TableOutputFormat, o la introducción de datos manualmente con el shell de HBase. Phoenix proporciona dos métodos para cargar datos CSV en tablas de Phoenix: una herramienta de carga de cliente denominada `psql` y una herramienta de carga masiva basada en MapReduce.

La herramienta `psql` tiene un subproceso único y es perfecta para cargar megabytes o gigabytes de datos. Todos los archivos CSV que se carguen deben tener la extensión de archivo "csv".  También puede especificar archivos de script SQL en la línea de comandos `psql` con la extensión de archivo "SQL".

La carga masiva con MapReduce se usa para volúmenes de datos mucho más grandes, normalmente en escenarios de producción, ya que usa varios subprocesos.

Antes de iniciar la carga de datos, compruebe que Phoenix está habilitado y que la configuración de tiempo de espera de consulta es la prevista.  Acceda al panel de [Apache Ambari](https://ambari.apache.org/) del clúster de HDInsight, seleccione HBase y, después, la pestaña Configuración.  Desplácese hacia abajo para comprobar que Apache Phoenix está establecido en `enabled` tal como se muestra:

![Configuración del clúster de Apache Phoenix HDInsight](./media/apache-hbase-phoenix-psql/apache-ambari-phoenix.png)

### <a name="use-psql-to-bulk-load-tables"></a>Usar `psql` para cargar tablas de forma masiva

1. Cree un archivo llamado `createCustomersTable.sql` y copie el código siguiente en el archivo. Guarde y cierre el archivo.

    ```sql
    CREATE TABLE Customers (
        ID varchar NOT NULL PRIMARY KEY,
        Name varchar,
        Income decimal,
        Age INTEGER,
        Country varchar);
    ```

1. Cree un archivo llamado `listCustomers.sql` y copie el código siguiente en el archivo. Guarde y cierre el archivo.

    ```sql
    SELECT * from Customers;
    ```

1. Cree un archivo llamado `customers.csv` y copie el código siguiente en el archivo. Guarde y cierre el archivo.

    ```txt
    1,Samantha,260000.0,18,US
    2,Sam,10000.5,56,US
    3,Anton,550150.0,42,Norway
    ```

1. Cree un archivo llamado `customers2.csv` y copie el código siguiente en el archivo. Guarde y cierre el archivo.

    ```txt
    4,Nicolle,180000.0,22,US
    5,Kate,210000.5,24,Canada
    6,Ben,45000.0,32,Poland
    ```

1. Abra un símbolo del sistema y cambie el directorio a la ubicación de los archivos recién creados. Reemplace CLUSTERNAME, a continuación, por el nombre real del clúster de HBase. A continuación, ejecute el código para cargar los archivos en el nodo principal del clúster:

    ```cmd
    scp customers.csv customers2.csv createCustomersTable.sql listCustomers.sql sshuser@CLUSTERNAME-ssh.azurehdinsight.net:/tmp
    ```

1. Use el [comando SSH](../hdinsight-hadoop-linux-use-ssh-unix.md) para conectarse al clúster. Modifique el comando siguiente: reemplace CLUSTERNAME por el nombre del clúster y, luego, escriba el comando:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Desde la sesión de SSH, cambie el directorio a la ubicación de la herramienta **psql**. Ejecute el siguiente comando:

    ```bash
    cd /usr/hdp/current/phoenix-client/bin
    ```

1. Cargue los datos de forma masiva. El código siguiente creará la tabla **Customers** y, a continuación, cargará los datos.

    ```bash
    python psql.py /tmp/createCustomersTable.sql /tmp/customers.csv
    ```

    Una vez finalizada la operación `psql`, debería ver un mensaje similar al siguiente:

    ```output
    csv columns from database.
    CSV Upsert complete. 3 rows upserted
    Time: 0.081 sec(s)
    ```

1. Puede seguir usando `psql` para ver el contenido de la tabla Customers. Ejecute el código siguiente:

    ```bash
    python psql.py /tmp/listCustomers.sql
    ```

    Como alternativa, puede usar el [shell de HBase](./query-hbase-with-hbase-shell.md) o [Apache Zeppelin](./apache-hbase-phoenix-zeppelin.md) para consultar los datos.

1. Cargue los datos adicionales. Ahora que la tabla ya existe, el comando especifica la tabla. Ejecute el siguiente comando:

    ```bash
    python psql.py -t CUSTOMERS /tmp/customers2.csv
    ```

## <a name="use-mapreduce-to-bulk-load-tables"></a>Usar MapReduce para cargar tablas de forma masiva

Para las cargas de mayor rendimiento distribuidas en el clúster, use la herramienta de carga de MapReduce. En primer lugar, este cargador convierte todos los datos en HFiles y, después, proporciona los archivos HFiles creados a HBase.

1. En esta sección se continúa con la sesión de SSH y los objetos creados anteriormente. Cree la tabla **Customers** y el archivo **customers.csv**, según sea necesario, siguiendo los pasos descritos anteriormente. Si es necesario, vuelva a establecer la conexión SSH.

1. Trunque el contenido de la tabla **Customers**. Ejecute los siguientes comandos en la sesión de SSH abierta:

    ```bash
    hbase shell
    truncate 'CUSTOMERS'
    exit
    ```

1. Copie el archivo `customers.csv` del nodo principal a Azure Storage.

    ```bash
    hdfs dfs -put /tmp/customers.csv wasbs:///tmp/customers.csv
    ```

1. Cambie al directorio de ejecución para el comando de carga masiva de MapReduce:

    ```bash
    cd /usr/hdp/current/phoenix-client
    ```

1. Inicie el cargador de CSV MapReduce con el comando `hadoop` con el archivo JAR del cliente de Phoenix:

    ```bash
    HADOOP_CLASSPATH=/usr/hdp/current/hbase-client/lib/hbase-protocol.jar:/etc/hbase/conf hadoop jar phoenix-client.jar org.apache.phoenix.mapreduce.CsvBulkLoadTool --table Customers --input /tmp/customers.csv
    ```

    Una vez completada la carga, debería ver un mensaje similar al siguiente:

    ```output
    19/12/18 18:30:57 INFO client.ConnectionManager$HConnectionImplementation: Closing master protocol: MasterService
    19/12/18 18:30:57 INFO client.ConnectionManager$HConnectionImplementation: Closing zookeeper sessionid=0x26f15dcceff02c3
    19/12/18 18:30:57 INFO zookeeper.ZooKeeper: Session: 0x26f15dcceff02c3 closed
    19/12/18 18:30:57 INFO zookeeper.ClientCnxn: EventThread shut down
    19/12/18 18:30:57 INFO mapreduce.AbstractBulkLoadTool: Incremental load complete for table=CUSTOMERS
    19/12/18 18:30:57 INFO mapreduce.AbstractBulkLoadTool: Removing output directory /tmp/50254426-aba6-400e-88eb-8086d3dddb6
    ```

1. Para usar MapReduce con Azure Data Lake Storage, busque el directorio raíz de Data Lake Storage, que es el valor `hbase.rootdir` de `hbase-site.xml`. En el siguiente comando, el directorio raíz de Data Lake Storage es `adl://hdinsightconf1.azuredatalakestore.net:443/hbase1`. En este comando, especifique las carpetas de entrada y salida de Data Lake Storage como parámetros:

    ```bash
    cd /usr/hdp/current/phoenix-client

    $HADOOP_CLASSPATH=$(hbase mapredcp):/etc/hbase/conf hadoop jar /usr/hdp/2.4.2.0-258/phoenix/phoenix-4.4.0.2.4.2.0-258-client.jar

    org.apache.phoenix.mapreduce.CsvBulkLoadTool --table Customers --input adl://hdinsightconf1.azuredatalakestore.net:443/hbase1/data/hbase/temp/input/customers.csv –zookeeper ZookeeperQuorum:2181:/hbase-unsecure --output  adl://hdinsightconf1.azuredatalakestore.net:443/hbase1/data/hbase/output1
    ```

1. Para consultar y ver los datos, puede usar **psql**, como se describió anteriormente. También puede usar el [shell de HBase](./query-hbase-with-hbase-shell.md) o [Apache Zeppelin](./apache-hbase-phoenix-zeppelin.md).

## <a name="recommendations"></a>Recomendaciones

* Use el mismo medio de almacenamiento, Azure Storage (WASB) o Azure Data Lake Storage (ADL), para las carpetas de entrada y salida. Para transferir datos de Azure Storage a Data Lake Storage, puede usar el comando `distcp`:

    ```bash
    hadoop distcp wasb://@.blob.core.windows.net/example/data/gutenberg adl://.azuredatalakestore.net:443/myfolder
    ```

* Use nodos de trabajo de tamaño grande. Los procesos de asignación de la copia masiva de MapReduce generan grandes cantidades de resultados temporales que llenan el espacio disponible que no es DFS. Para una gran cantidad de carga masiva, use más nodos de trabajo y más grandes. El número de nodos de trabajo que asigne al clúster afecta directamente a la velocidad de procesamiento.

* Divida los archivos de entrada en fragmentos de aproximadamente 10 GB. La carga masiva es una operación de almacenamiento intensivo, por lo que dividir los archivos de entrada en varios fragmentos mejora el rendimiento.

* Evite puntos de acceso del servidor de región. Si la clave de fila aumenta de forma uniforme, las escrituras secuenciales de HBase pueden inducir puntos de acceso del servidor de región. *Cifrar* con sal la clave de fila reduce las escrituras secuenciales. Phoenix proporciona una manera de cifrar de forma transparente la clave de fila con un byte cifrado con sal para una determinada tabla, tal como se indica a continuación.

## <a name="next-steps"></a>Pasos siguientes

* [Bulk Data Loading with Apache Phoenix](https://phoenix.apache.org/bulk_dataload.html) (Carga de datos masiva con Apache Phoenix)
* [Usar Apache Phoenix con clústeres de Apache HBase basados en Linux en HDInsight](../hbase/apache-hbase-query-with-phoenix.md)
* [Salted Tables](https://phoenix.apache.org/salted.html) (Tablas cifradas)
* [Apache Phoenix Grammar](https://phoenix.apache.org/language/index.html) (Gramática de Apache Phoenix)
