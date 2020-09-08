---
title: 'Uso de Spark para leer y escribir datos de HBase: Azure HDInsight'
description: Use el conector Spark HBase para leer y escribir datos de un clúster Spark en un clúster HBase.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 08/12/2020
ms.openlocfilehash: 9454cb83d535d97a3dd95cd9f5d0636769797d08
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88166950"
---
# <a name="use-apache-spark-to-read-and-write-apache-hbase-data"></a>Uso de Apache Spark para leer y escribir datos de Apache HBase

Apache HBase se consulta, normalmente, con su API de bajo nivel (instrucciones scan, get y put) o con una sintaxis SQL que utiliza Apache Phoenix. Apache también proporciona el conector HBase de Apache Spark. Este es una alternativa práctica y eficaz para consultar y modificar los datos que almacena HBase.

## <a name="prerequisites"></a>Requisitos previos

* Dos clústeres de HDInsight independientes implementados en la misma [red virtual](./hdinsight-plan-virtual-network-deployment.md). HBase y Spark con al menos Spark 2.1 (HDInsight 3.6) instalado. Para obtener más información, consulte [Crear clústeres basados en Linux en HDInsight con Azure Portal](hdinsight-hadoop-create-linux-clusters-portal.md).

* El esquema de URI para el almacenamiento principal de clústeres. Este esquema sería wasb:// para Azure Blob Storage, `abfs://` para Azure Data Lake Storage Gen2 o adl:// para Azure Data Lake Storage Gen1. Si se habilita la transferencia segura para Blob Storage, el identificador URI sería `wasbs://`.  Consulte también el artículo acerca de la [transferencia segura](../storage/common/storage-require-secure-transfer.md).

## <a name="overall-process"></a>Proceso general

El proceso de alto nivel para habilitar el clúster de Spark para consultar el clúster de HBase es el siguiente:

1. Prepare algunos datos de ejemplo en HBase.
2. Obtenga el archivo hbase-site.xml de la carpeta de configuración del clúster de HBase (/etc/hbase/conf) y coloque una copia de hbase-site.xml en la carpeta de configuración de Spark 2 (/etc/spark2/conf). (OPCIONAL: usar el script proporcionado por el equipo de HDInsight para automatizar este proceso)
4. Ejecute `spark-shell` haciendo referencia al conector Spark HBase por sus coordenadas Maven en la opción `packages`.
5. Defina un catálogo que asigne el esquema de Spark a HBase.
6. Interactúe con los datos de HBase mediante las API de DataFrame o RDD.

## <a name="prepare-sample-data-in-apache-hbase"></a>Preparación de datos de ejemplo en Apache HBase

En este paso, se crea y rellena una tabla en Apache HBase que, después, se podrá consultar mediante Spark.

1. Use el comando `ssh` para conectarse al clúster de HBase. Modifique el comando siguiente. Para ello, reemplace `HBASECLUSTER` por el nombre del clúster de Hbase y, luego, escriba el comando:

    ```cmd
    ssh sshuser@HBASECLUSTER-ssh.azurehdinsight.net
    ```

2. Use el comando `hbase shell` para iniciar el shell interactivo de HBase. Escriba el siguiente comando en la conexión SSH:

    ```bash
    hbase shell
    ```

3. Use el comando `create` para crear una tabla de HBase con dos familias de columnas. Escriba el comando siguiente:

    ```hbase
    create 'Contacts', 'Personal', 'Office'
    ```

4. Use el comando `put` para insertar valores en una columna especificada de una fila concreta de una tabla determinada. Escriba el comando siguiente:

    ```hbase
    put 'Contacts', '1000', 'Personal:Name', 'John Dole'
    put 'Contacts', '1000', 'Personal:Phone', '1-425-000-0001'
    put 'Contacts', '1000', 'Office:Phone', '1-425-000-0002'
    put 'Contacts', '1000', 'Office:Address', '1111 San Gabriel Dr.'
    put 'Contacts', '8396', 'Personal:Name', 'Calvin Raji'
    put 'Contacts', '8396', 'Personal:Phone', '230-555-0191'
    put 'Contacts', '8396', 'Office:Phone', '230-555-0191'
    put 'Contacts', '8396', 'Office:Address', '5415 San Gabriel Dr.'
    ```

5. Use el comando `exit` para detener el shell interactivo de HBase. Escriba el comando siguiente:

    ```hbase
    exit
    ```
    
## <a name="run-scripts-to-set-up-connection-between-clusters"></a>Ejecución de scripts para configurar la conexión entre clústeres

Para configurar la comunicación entre los clústeres, siga los pasos que se indican a continuación para ejecutar dos scripts en los clústeres. Estos scripts automatizarán el proceso de copia de archivos descrito en la sección "Configuración manual de la comunicación" más adelante. 

* El script que ejecute desde el clúster de HBase cargará `hbase-site.xml` y la información de asignación de IP de HBase en el almacenamiento predeterminado asociado al clúster de Spark. 
* El script que ejecute desde el clúster de Spark configurará dos trabajos de cron para ejecutar dos scripts auxiliares periódicamente:  
    1.  Trabajo de cron de HBase: descarga nuevos archivos `hbase-site.xml` y la asignación de IP de HBase de la cuenta de almacenamiento predeterminada de Spark al nodo local
    2.  Trabajo de cron de Spark: comprueba si se ha producido un escalado de Spark y si el clúster es seguro. Si es así, edite `/etc/hosts` para incluir la asignación de IP de HBase almacenada localmente

__NOTA__: Antes de continuar, asegúrese de que ha agregado la cuenta de almacenamiento del clúster de Spark a su clúster de HBase como cuenta de almacenamiento secundaria. Asegúrese de que los scripts estén ordenados como se indica a continuación.


1. Use [Acción de script](hdinsight-hadoop-customize-cluster-linux.md#script-action-to-a-running-cluster) en el clúster de HBase para aplicar los cambios con las siguientes consideraciones: 


    |Propiedad | Value |
    |---|---|
    |URI de script de Bash|`https://hdiconfigactions.blob.core.windows.net/hbasesparkconnectorscript/connector-hbase.sh`|
    |Tipos de nodo|Region|
    |Parámetros|`-s SECONDARYS_STORAGE_URL`|
    |Guardado|sí|

    * `SECONDARYS_STORAGE_URL` es la dirección URL del almacenamiento predeterminado del lado de Spark. Ejemplo de parámetro: `-s wasb://sparkcon-2020-08-03t18-17-37-853z@sparkconhdistorage.blob.core.windows.net`


2.  Use Acción de script en el clúster de Spark para aplicar los cambios con las siguientes consideraciones:

    |Propiedad | Value |
    |---|---|
    |URI de script de Bash|`https://hdiconfigactions.blob.core.windows.net/hbasesparkconnectorscript/connector-spark.sh`|
    |Tipos de nodo|Head, Worker, Zookeeper|
    |Parámetros|`-s "SPARK-CRON-SCHEDULE"` (opcional) `-h "HBASE-CRON-SCHEDULE"` (opcional)|
    |Guardado|sí|


    * Puede especificar la frecuencia con la que desee que este clúster compruebe automáticamente si actualizarse. Valor predeterminado: -s “*/1 * * * *” -h 0 (en este ejemplo, el trabajo de cron de Spark se ejecuta cada minuto, mientras que el trabajo de cron de HBase no se ejecuta)
    * Dado que el trabajo de cron de HBase no se configura de forma predeterminada, debe volver a ejecutar este script cuando realice el escalado a su clúster de HBase. Si el clúster de HBase se escala a menudo, puede optar por configurar el trabajo de cron de HBase automáticamente. Por ejemplo: `-h "*/30 * * * *"` configura el script para realizar comprobaciones cada 30 minutos. Esto ejecutará la programación de cron de HBase periódicamente para automatizar la descarga de nueva información de HBase de la cuenta de almacenamiento común al nodo local.
    
    

## <a name="set-up-communication-manually-optional-if-provided-script-in-above-step-fails"></a>Configuración manual de la comunicación (opcional, si se produce un error en el script proporcionado en el paso anterior)

__NOTA:__ Estos pasos deben realizarse cada vez que se realiza una actividad de escalado en uno de los clústeres.

1. Copie hbase-site.xml del almacenamiento local a la raíz del almacenamiento predeterminado del clúster de Spark.  Edite el siguiente comando para reflejar la configuración.  A continuación, en la sesión SSH abierta para el clúster de HBase, escriba el comando:

    | Valor de sintaxis | Valor nuevo|
    |---|---|
    |[Esquema de URI](hdinsight-hadoop-linux-information.md#URI-and-scheme) | Modifíquelo para reflejar su almacenamiento.  La siguiente sintaxis es para el almacenamiento de blobs con transferencia segura habilitada.|
    |`SPARK_STORAGE_CONTAINER`|Reemplácelo con el nombre del contenedor de almacenamiento predeterminado que se usa para el clúster de Spark.|
    |`SPARK_STORAGE_ACCOUNT`|Reemplácelo por el nombre predeterminado de la cuenta de almacenamiento que se usa para el clúster de Spark.|

    ```bash
    hdfs dfs -copyFromLocal /etc/hbase/conf/hbase-site.xml wasbs://SPARK_STORAGE_CONTAINER@SPARK_STORAGE_ACCOUNT.blob.core.windows.net/
    ```

2. Luego, salga de su conexión ssh con el clúster de HBase.

    ```bash
    exit
    ```


3. Conéctese al nodo principal del clúster Spark mediante SSH. Modifique el comando siguiente al reemplazar `SPARKCLUSTER` por el nombre del clúster de Spark y luego escriba el comando:

    ```cmd
    ssh sshuser@SPARKCLUSTER-ssh.azurehdinsight.net
    ```

4. Escriba el comando siguiente para copiar `hbase-site.xml` del almacenamiento predeterminado del clúster de Spark en la carpeta de configuración de Spark 2 en el almacenamiento local del clúster:

    ```bash
    sudo hdfs dfs -copyToLocal /hbase-site.xml /etc/spark2/conf
    ```

## <a name="run-spark-shell-referencing-the-spark-hbase-connector"></a>Ejecutar el shell Spark haciendo referencia al conector Spark HBase

Después de completar el paso anterior, debería poder ejecutar el shell de Spark con referencia a la versión adecuada del conector Spark HBase. Para encontrar la versión más reciente del núcleo del conector Spark HBase para el escenario del clúster, consulte el [repositorio principal de SHC](https://repo.hortonworks.com/content/groups/public/com/hortonworks/shc/shc-core/).

Como ejemplo, en la tabla siguiente se muestran dos versiones y los comandos correspondientes que el equipo de HDInsight usa actualmente. Si las versiones de HBase y Spark son las mismas que las indicadas en la tabla, puede usar las mismas versiones de los clústeres. 


1. En la sesión SSH que tiene abierta del clúster de Spark, escriba el siguiente comando para iniciar un shell de Spark:

    |Versión de Spark| Versión de HDI de HBase  | Versión de SHC    |  Get-Help  |
    | :-----------:| :----------: | :-----------: |:----------- |
    |      2.1    | HDI 3.6 (HBase 1.1) | 1.1.0.3.1.2.2-1    | `spark-shell --packages com.hortonworks:shc-core:1.1.1-2.1-s_2.11 --repositories https://repo.hortonworks.com/content/groups/public/` |
    |      2.4    | HDI 4.0 (HBase 2.0) | 1.1.1-2.1-s_2.11  | `spark-shell --packages com.hortonworks.shc:shc-core:1.1.0.3.1.2.2-1 --repositories http://repo.hortonworks.com/content/groups/public/` |

2. Mantenga abierta esta instancia de shell de Spark y continúe con [Definir un catálogo y una consulta](#define-a-catalog-and-query). Si no encuentra los archivos JAR correspondientes a sus versiones en el repositorio de SHC Core, siga leyendo. 

Puede compilar los archivos JAR directamente desde la rama GitHub de [Spark-HBase-Connector](https://github.com/hortonworks-spark/shc). Por ejemplo, si ejecuta Spark 2.3 y HBase 1.1, siga estos pasos:

1. Clonación del repositorio:

    ```bash
    git clone https://github.com/hortonworks-spark/shc
    ```
    
2. Vaya a la rama-2.3:

    ```bash
    git checkout branch-2.3
    ```

3. Compilar a partir de la rama (crea un archivo .jar):

    ```bash
    mvn clean package -DskipTests
    ```
    
3. Ejecute el siguiente comando (asegúrese de cambiar el nombre .jar que corresponde al archivo .jar que ha creado):

    ```bash
    spark-shell --jars <path to your jar>,/usr/hdp/current/hbase-client/lib/htrace-core-3.1.0-incubating.jar,/usr/hdp/current/hbase-client/lib/hbase-client.jar,/usr/hdp/current/hbase-client/lib/hbase-common.jar,/usr/hdp/current/hbase-client/lib/hbase-server.jar,/usr/hdp/current/hbase-client/lib/hbase-protocol.jar,/usr/hdp/current/hbase-client/lib/htrace-core-3.1.0-incubating.jar
    ```
    
4. Mantenga abierta esta instancia de shell de Spark y continúe con el paso siguiente. 



## <a name="define-a-catalog-and-query"></a>Definir un catálogo y una consulta

En este paso, se define un objeto de catálogo que asigna el esquema de Apache Spark a Apache HBase.  

1. En el shell abierto de Spark, ejecute las siguientes instrucciones `import`:

    ```scala
    import org.apache.spark.sql.{SQLContext, _}
    import org.apache.spark.sql.execution.datasources.hbase._
    import org.apache.spark.{SparkConf, SparkContext}
    import spark.sqlContext.implicits._
    ```  

1. Escriba el comando siguiente para definir un catálogo para la tabla Contactos que creó en HBase:

    ```scala
    def catalog = s"""{
        |"table":{"namespace":"default", "name":"Contacts"},
        |"rowkey":"key",
        |"columns":{
        |"rowkey":{"cf":"rowkey", "col":"key", "type":"string"},
        |"officeAddress":{"cf":"Office", "col":"Address", "type":"string"},
        |"officePhone":{"cf":"Office", "col":"Phone", "type":"string"},
        |"personalName":{"cf":"Personal", "col":"Name", "type":"string"},
        |"personalPhone":{"cf":"Personal", "col":"Phone", "type":"string"}
        |}
    |}""".stripMargin
    ```

    El código:  

    1. Define un esquema de catálogo para la tabla HBase denominada `Contacts`.  
    1. Identifica el objeto rowkey como `key` y asigna los nombres de columna usados en Spark a la familia de columna, el nombre de columna y el tipo de columna tal como se usa en HBase.  
    1. Define el objeto rowkey en detalle como una columna con nombre (`rowkey`), que tiene una familia de columna específica `cf` de `rowkey`.  

1. Escriba el comando siguiente para definir un método que proporcione un elemento DataFrame alrededor de su tabla `Contacts` en HBase:

    ```scala
    def withCatalog(cat: String): DataFrame = {
        spark.sqlContext
        .read
        .options(Map(HBaseTableCatalog.tableCatalog->cat))
        .format("org.apache.spark.sql.execution.datasources.hbase")
        .load()
     }
    ```

1. Cree una instancia de DataFrame:

    ```scala
    val df = withCatalog(catalog)
    ```  

1. Consulte el elemento DataFrame:

    ```scala
    df.show()
    ```

    Verá dos filas de datos:

    ```output
    +------+--------------------+--------------+-------------+--------------+
    |rowkey|       officeAddress|   officePhone| personalName| personalPhone|
    +------+--------------------+--------------+-------------+--------------+
    |  1000|1111 San Gabriel Dr.|1-425-000-0002|    John Dole|1-425-000-0001|
    |  8396|5415 San Gabriel Dr.|  230-555-0191|  Calvin Raji|  230-555-0191|
    +------+--------------------+--------------+-------------+--------------+
    ```

1. Registre una tabla temporal para poder consultar la tabla HBase mediante Spark SQL:

    ```scala
    df.createTempView("contacts")
    ```

1. Emita una consulta SQL en la tabla `contacts`:

    ```scala
    spark.sqlContext.sql("select personalName, officeAddress from contacts").show
    ```

    Verá resultados parecidos a los siguientes:

    ```output
    +-------------+--------------------+
    | personalName|       officeAddress|
    +-------------+--------------------+
    |    John Dole|1111 San Gabriel Dr.|
    |  Calvin Raji|5415 San Gabriel Dr.|
    +-------------+--------------------+
    ```

## <a name="insert-new-data"></a>Insertar nuevos datos

1. Para insertar un nuevo registro de contacto, defina una clase `ContactRecord`:

    ```scala
    case class ContactRecord(
        rowkey: String,
        officeAddress: String,
        officePhone: String,
        personalName: String,
        personalPhone: String
        )
    ```

1. Cree una instancia de `ContactRecord` y colóquela en una matriz:

    ```scala
    val newContact = ContactRecord("16891", "40 Ellis St.", "674-555-0110", "John Jackson","230-555-0194")

    var newData = new Array[ContactRecord](1)
    newData(0) = newContact
    ```

1. Guarde la matriz de nuevos datos en HBase:

    ```scala
    sc.parallelize(newData).toDF.write.options(Map(HBaseTableCatalog.tableCatalog -> catalog, HBaseTableCatalog.newTable -> "5")).format("org.apache.spark.sql.execution.datasources.hbase").save()
    ```

1. Examine los resultados:

    ```scala  
    df.show()
    ```

    La salida debería ser parecida a la que se muestra a continuación:

    ```output
    +------+--------------------+--------------+------------+--------------+
    |rowkey|       officeAddress|   officePhone|personalName| personalPhone|
    +------+--------------------+--------------+------------+--------------+
    |  1000|1111 San Gabriel Dr.|1-425-000-0002|   John Dole|1-425-000-0001|
    | 16891|        40 Ellis St.|  674-555-0110|John Jackson|  230-555-0194|
    |  8396|5415 San Gabriel Dr.|  230-555-0191| Calvin Raji|  230-555-0191|
    +------+--------------------+--------------+------------+--------------+
    ```

1. Para cerrar el shell de Spark, escriba el comando siguiente:

    ```scala
    :q
    ```

## <a name="next-steps"></a>Pasos siguientes

* [Conector HBase de Azure Spark](https://github.com/hortonworks-spark/shc)
