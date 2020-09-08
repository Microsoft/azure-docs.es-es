---
title: Uso de Apache Beeline con Apache Hive en Azure HDInsight
description: Aprenda a usar el cliente de Beeline para ejecutar consultas de Hive con Hadoop en HDInsight. Beeline es una utilidad para trabajar con HiveServer2 sobre JDBC.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 08/21/2020
ms.custom: contperfq1
ms.openlocfilehash: f6d8f804fa26383435d191af27289ffd2ecb3e0b
ms.sourcegitcommit: afa1411c3fb2084cccc4262860aab4f0b5c994ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/23/2020
ms.locfileid: "88755099"
---
# <a name="use-the-apache-beeline-client-with-apache-hive"></a>Usar el cliente de Apache Beeline con Apache Hive

Aprenda a usar [Apache Beeline](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-Beeline–NewCommandLineShell) para ejecutar consultas de Apache Hive en HDInsight.

Beeline es un cliente de Hive que se incluye en los nodos principales del clúster de HDInsight. Para conectarse al cliente de Beeline instalado en el clúster de HDInsight o instalar Beeline en local, vea [Conexión a Apache Beeline o instalación](connect-install-beeline.md). Beeline usa JDBC para conectarse a HiveServer2, un servicio hospedado en el clúster de HDInsight. Beeline también se puede usar para tener acceso a Hive en HDInsight de forma remota a través de Internet. En los ejemplos siguientes se proporcionan las cadenas de conexión más habituales usadas para conectarse a HDInsight desde Beeline.

## <a name="prerequisites-for-examples"></a>Requisitos previos para los ejemplos

* Un clúster de Hadoop en HDInsight. Consulte [Introducción a HDInsight en Linux](./apache-hadoop-linux-tutorial-get-started.md).

* Observe el esquema de URI para el almacenamiento principal del clúster. Por ejemplo, `wasb://` para Azure Storage, `abfs://` para Azure Data Lake Storage Gen2 o `adl://` para Azure Data Lake Storage Gen1. Si se habilita la transferencia segura para Azure Storage, el identificador URI es `wasbs://`. Para más información, consulte el artículo sobre la [transferencia segura](../../storage/common/storage-require-secure-transfer.md).

* Opción 1: Un cliente SSH. Para más información, consulte [Conexión a través de SSH con HDInsight (Apache Hadoop)](../hdinsight-hadoop-linux-use-ssh-unix.md). En la mayoría de los pasos descritos en este documento se da por hecho que está usando Beeline desde una sesión de SSH al clúster.

* Opción 2:  Cliente de Beeline local.

## <a name="run-a-hive-query"></a>Ejecución de una consulta de Hive

Este ejemplo se basa en el uso del cliente de Beeline desde una conexión SSH.

1. Abra una conexión de SSH al clúster con el código siguiente. Reemplace `sshuser` por el usuario de SSH del clúster y `CLUSTERNAME` por el nombre de su clúster. Cuando se le solicite, escriba la contraseña de la cuenta de usuario de SSH.

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

2. Conéctese a HiveServer2 con el cliente de Beeline de la sesión de SSH abierta. Para ello, escriba el comando siguiente:

    ```bash
    beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http'
    ```

3. Los comandos de Beeline empiezan con un carácter `!`. Por ejemplo `!help` muestra la ayuda. Sin embargo, el `!` se puede omitir en algunos comandos. Por ejemplo, `help` también funciona.

    Hay un `!sql`, que se usa para ejecutar instrucciones de HiveQL. Sin embargo, HiveQL es de uso tan frecuente que puede omitir el elemento `!sql`que le precede. Las dos instrucciones siguientes son equivalentes:

    ```hiveql
    !sql show tables;
    show tables;
    ```

    En un nuevo clúster, solo debe aparecer una tabla: **hivesampletable**.

4. Use el siguiente comando para mostrar el esquema de hivesampletable:

    ```hiveql
    describe hivesampletable;
    ```

    Este comando devuelve la siguiente información:

    ```output
    +-----------------------+------------+----------+--+
    |       col_name        | data_type  | comment  |
    +-----------------------+------------+----------+--+
    | clientid              | string     |          |
    | querytime             | string     |          |
    | market                | string     |          |
    | deviceplatform        | string     |          |
    | devicemake            | string     |          |
    | devicemodel           | string     |          |
    | state                 | string     |          |
    | country               | string     |          |
    | querydwelltime        | double     |          |
    | sessionid             | bigint     |          |
    | sessionpagevieworder  | bigint     |          |
    +-----------------------+------------+----------+--+
    ```

    Esta información describe las columnas de la tabla.

5. Escriba las siguientes instrucciones para crear una tabla denominada **log4jLogs** con los datos de ejemplo proporcionados con el clúster de HDInsight: (Revise según sea necesario en función de su esquema de URI).

    ```hiveql
    DROP TABLE log4jLogs;
    CREATE EXTERNAL TABLE log4jLogs (
        t1 string,
        t2 string,
        t3 string,
        t4 string,
        t5 string,
        t6 string,
        t7 string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
    STORED AS TEXTFILE LOCATION 'wasbs:///example/data/';
    SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs
        WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log'
        GROUP BY t4;
    ```

    Estas instrucciones realizan las acciones siguientes:

    |. |Descripción |
    |---|---|
    |DROP TABLE|Si la tabla existe, se elimina.|
    |CREATE EXTERNAL TABLE|Crea una tabla **externa** en Hive. Las tablas externas solo almacenan la definición de Tabla en Hive. Los datos permanecen en la ubicación original.|
    |FORMATO DE FILA|Indica el formato de los datos. En este caso, los campos de cada registro se separan mediante un espacio.|
    |STORED AS TEXTFILE LOCATION|Indica dónde se almacenan los datos y en qué formato de archivo.|
    |SELECT|Selecciona un recuento de todas las filas donde la columna **t4** contiene el valor **[ERROR]** . Esta consulta devuelve un valor de **3** ya que hay tres filas que contienen este valor.|
    |INPUT__FILE__NAME LIKE '%.log'|Hive intenta aplicar el esquema a todos los archivos en el directorio. En este caso, el directorio contiene archivos que no coinciden con el esquema. Para evitar que haya datos inservibles en los resultados, esta instrucción indica a Hive que solo se deben devolver datos de archivos que terminen en .log.|

   > [!NOTE]  
   > Las tablas externas se deben utilizar cuando se espera que un origen externo actualice los datos subyacentes. Por ejemplo, un proceso de carga de datos automatizado o una operación de MapReduce.
   >
   > La eliminación de una tabla externa **no** elimina los datos, solamente la definición de tabla.

    La salida de este comando es similar al texto siguiente:

    ```output
    INFO  : Tez session hasn't been created yet. Opening session
    INFO  :

    INFO  : Status: Running (Executing on YARN cluster with App id application_1443698635933_0001)

    INFO  : Map 1: -/-      Reducer 2: 0/1
    INFO  : Map 1: 0/1      Reducer 2: 0/1
    INFO  : Map 1: 0/1      Reducer 2: 0/1
    INFO  : Map 1: 0/1      Reducer 2: 0/1
    INFO  : Map 1: 0/1      Reducer 2: 0/1
    INFO  : Map 1: 0(+1)/1  Reducer 2: 0/1
    INFO  : Map 1: 0(+1)/1  Reducer 2: 0/1
    INFO  : Map 1: 1/1      Reducer 2: 0/1
    INFO  : Map 1: 1/1      Reducer 2: 0(+1)/1
    INFO  : Map 1: 1/1      Reducer 2: 1/1
    +----------+--------+--+
    |   sev    | count  |
    +----------+--------+--+
    | [ERROR]  | 3      |
    +----------+--------+--+
    1 row selected (47.351 seconds)
    ```

6. Salga de Beeline:

    ```bash
    !exit
    ```

## <a name="run-a-hiveql-file"></a>Ejecución de un archivo de HiveQL

Este ejemplo es una continuación del ejemplo anterior. Use los pasos siguientes para crear un archivo y, luego, ejecútelo mediante Beeline.

1. Use el comando siguiente para crear un archivo denominado **query.hql**:

    ```bash
    nano query.hql
    ```

1. Use el texto siguiente como contenido del archivo. Esta consulta crea una nueva tabla "interna" denominada **errorLogs**:

    ```hiveql
    CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) STORED AS ORC;
    INSERT OVERWRITE TABLE errorLogs SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log';
    ```

    Estas instrucciones realizan las acciones siguientes:

    |. |Descripción |
    |---|---|
    |CREATE TABLE IF NOT EXISTS|Si todavía no existe la tabla, se crea. Dado que no se utiliza la palabra clave **EXTERNAL**, esta instrucción crea una tabla interna. Las tablas internas se guardan en el almacenamiento de datos de Hive y Hive las administra por completo.|
    |STORED AS ORC|almacena los datos en el formato de columnas de filas optimizadas (ORC, Optimized Row Columnar). ORC es un formato altamente optimizado y eficiente para almacenar datos de Hive.|
    |INSERT OVERWRITE ... SELECT|selecciona en la tabla **log4jLogs** las filas que contienen **[ERROR]** y, después, inserta los datos en la tabla **errorLogs**.|

    > [!NOTE]  
    > A diferencia de las tablas externas , la eliminación de una tabla interna también elimina los datos subyacentes.

1. Para guardar el archivo, use **Ctrl**+**X**, escriba **Y** y, finalmente, presione **Entrar**.

1. Use el siguiente código para ejecutar el archivo mediante Beeline:

    ```bash
    beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http' -i query.hql
    ```

    > [!NOTE]  
    > El parámetro `-i` inicia Beeline y ejecuta las instrucciones del archivo `query.hql`. Una vez que se completa la consulta, llegará al aviso `jdbc:hive2://headnodehost:10001/>`. También puede ejecutar un archivo mediante el parámetro `-f` que sale de Beeline después de que la consulta se completa.

1. Para comprobar que la tabla **errorLogs** se creó, use la siguiente instrucción para devolver todas las filas de **errorLogs**:

    ```hiveql
    SELECT * from errorLogs;
    ```

    Se deben devolver tres filas de datos y todas ellas deben contener **[ERROR]** en la columna t4.

    ```output
    +---------------+---------------+---------------+---------------+---------------+---------------+---------------+--+
    | errorlogs.t1  | errorlogs.t2  | errorlogs.t3  | errorlogs.t4  | errorlogs.t5  | errorlogs.t6  | errorlogs.t7  |
    +---------------+---------------+---------------+---------------+---------------+---------------+---------------+--+
    | 2012-02-03    | 18:35:34      | SampleClass0  | [ERROR]       | incorrect     | id            |               |
    | 2012-02-03    | 18:55:54      | SampleClass1  | [ERROR]       | incorrect     | id            |               |
    | 2012-02-03    | 19:25:27      | SampleClass4  | [ERROR]       | incorrect     | id            |               |
    +---------------+---------------+---------------+---------------+---------------+---------------+---------------+--+
    3 rows selected (0.813 seconds)
    ```

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información general sobre Hive en HDInsight, consulte [Uso de Apache Hive con Apache Hadoop en HDInsight](hdinsight-use-hive.md).

* Si quiere más información sobre otras formas en que puede trabajar con Hadoop en HDInsight, consulte [Uso de MapReduce con Apache Hadoop en HDInsight](hdinsight-use-mapreduce.md).
