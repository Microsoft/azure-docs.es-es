---
title: 'Tutorial: Uso de Apache HBase en Azure HDInsight'
description: Siga este tutorial de Apache HBase para empezar a usar Hadoop en HDInsight. Cree tablas desde el shell de HBase y consúltelas mediante Hive.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: tutorial
ms.custom: hdinsightactive,hdiseo17may2017
ms.date: 04/14/2020
ms.openlocfilehash: a19e2c6647f1ff072c61044e8e5777d5d3f8d2db
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "85958368"
---
# <a name="tutorial-use-apache-hbase-in-azure-hdinsight"></a>Tutorial: Uso de Apache HBase in Azure HDInsight

Este tutorial muestra cómo crear un clúster de Apache HBase en Azure HDInsight, crear tablas de HBase y consultar tablas mediante Apache Hive.  Para obtener información general de HBase, consulte este artículo de [información general de HBase en HDInsight](./apache-hbase-overview.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear clústeres de Apache HBase
> * Creación de tablas de HBase e inserción de datos
> * Usar Apache Hive para consultar Apache HBase
> * Usar las API de REST de HBase con Curl
> * Comprobar el estado del clúster

## <a name="prerequisites"></a>Prerrequisitos

* Un cliente SSH. Para más información, consulte [Conexión a través de SSH con HDInsight (Apache Hadoop)](../hdinsight-hadoop-linux-use-ssh-unix.md).

* Bash. Los ejemplos de este artículo usan el shell de Bash en Windows 10 para los comandos de curl. Consulte la [Guía de instalación del subsistema de Windows para Linux para Windows 10](https://docs.microsoft.com/windows/wsl/install-win10) para conocer los pasos de instalación.  Otros [shells de Unix](https://www.gnu.org/software/bash/) también funcionarán.  Los ejemplos de curl, con algunas pequeñas modificaciones, pueden funcionar en un símbolo del sistema de Windows.  También puede usar el cmdlet de Windows PowerShell [Invoke-RestMethod](https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/invoke-restmethod).

## <a name="create-apache-hbase-cluster"></a>Crear clústeres de Apache HBase

El siguiente procedimiento utiliza una plantilla de Azure Resource Manager para crear un clúster de HBase. La plantilla también crea la cuenta de Azure Storage predeterminada dependiente. Para comprender los parámetros utilizados en el procedimiento y otros métodos de creación del clúster, consulte [Creación de clústeres de Hadoop basados en Linux en HDInsight](../hdinsight-hadoop-provision-linux-clusters.md).

1. Seleccione la imagen siguiente para abrir la plantilla en Azure Portal. La plantilla se encuentra en las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/).

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-hbase-linux%2Fazuredeploy.json" target="_blank"><img src="./media/apache-hbase-tutorial-get-started-linux/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

2. En el cuadro de diálogo **Implementación personalizada**, escriba los valores siguientes:

    |Propiedad |Descripción |
    |---|---|
    |Subscription|seleccione la suscripción de Azure que usa para crear este clúster.|
    |Resource group|cree un grupo de administración de recursos de Azure o use uno existente.|
    |Location|especifique la ubicación del grupo de recursos. |
    |ClusterName|escriba el nombre del clúster de HBase.|
    |Nombre de inicio de sesión y contraseña del clúster|El nombre de inicio de sesión predeterminado es **admin**.|
    |Nombre de usuario y contraseña de SSH|El nombre de usuario predeterminado es **sshuser**.|

    Otros parámetros son opcionales.  

    Cada clúster tiene una dependencia de cuenta de Azure Storage. Después de eliminar un clúster, los datos permanecen en la cuenta de almacenamiento. El nombre de cuenta de almacenamiento de clúster predeterminado es el nombre del clúster con "store" anexado. Está codificado de forma rígida en la sección de variables de plantilla.

3. Seleccione **Acepto los términos y condiciones indicadas anteriormente** y, después, seleccione **Comprar**. Se tarda aproximadamente 20 minutos en crear un clúster.

Después de que se elimine un clúster de HBase, puede crear otro clúster de HBase mediante el mismo contenedor de blobs predeterminado. El nuevo clúster selecciona las tablas de HBase que creó en el clúster original. Para evitar incoherencias, recomendamos deshabilitar las tablas de HBase antes de eliminar el clúster.

## <a name="create-tables-and-insert-data"></a>Creación de tablas e inserción de datos

Puede usar SSH para conectarse a los clústeres de HBase y, después, usar el [shell de Apache HBase](https://hbase.apache.org/0.94/book/shell.html) para crear tablas de HBase e insertar y consultar datos.

Para la mayoría de las personas, los datos aparecen en formato tabular:

![Datos tabulares de Apache HBase de HDInsight](./media/apache-hbase-tutorial-get-started-linux/hdinsight-hbase-contacts-tabular.png)

En HBase (una implementación de [Cloud BigTable](https://cloud.google.com/bigtable/)), los mismos datos tienen un aspecto similar al siguiente:

![Datos de BigTable de Apache HBase de HDInsight](./media/apache-hbase-tutorial-get-started-linux/hdinsight-hbase-contacts-bigtable.png)

**Para usar el shell de HBase, siga estos pasos:**

1. Use el comando `ssh` para conectarse al clúster de HBase. Modifique el comando siguiente: reemplace `CLUSTERNAME` por el nombre del clúster y, luego, escriba el comando:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Use el comando `hbase shell` para iniciar el shell interactivo de HBase. Escriba el siguiente comando en la conexión SSH:

    ```bash
    hbase shell
    ```

1. Use el comando `create` para crear una tabla de HBase con dos familias de columnas. Los nombres de columna y tabla distinguen mayúsculas de minúsculas. Escriba el comando siguiente:

    ```hbaseshell
    create 'Contacts', 'Personal', 'Office'
    ```

1. Use el comando `list` para mostrar todas las tablas de HBase. Escriba el comando siguiente:

    ```hbase
    list
    ```

1. Use el comando `put` para insertar valores en una columna especificada en una fila especificada de una tabla determinada. Escriba los siguientes comandos:

    ```hbaseshell
    put 'Contacts', '1000', 'Personal:Name', 'John Dole'
    put 'Contacts', '1000', 'Personal:Phone', '1-425-000-0001'
    put 'Contacts', '1000', 'Office:Phone', '1-425-000-0002'
    put 'Contacts', '1000', 'Office:Address', '1111 San Gabriel Dr.'
    ```

1. Use el comando `scan` para buscar y devolver los datos de tabla `Contacts`. Escriba el comando siguiente:

    ```hbase
    scan 'Contacts'
    ```

    ![Shell de HBase de Apache Hadoop de HDInsight](./media/apache-hbase-tutorial-get-started-linux/hdinsight-hbase-shell.png)

1. Use el comando `get` para capturar el contenido de una fila. Escriba el comando siguiente:

    ```hbaseshell
    get 'Contacts', '1000'
    ```

    Observará que los resultados son similares a los del comando `scan` porque solo hay una fila.

    Para más información sobre el esquema de tabla de HBase, consulte [Introduction to Apache HBase Schema Design](http://0b4af6cdc2f0c5998459-c0245c5c937c5dedcca3f1764ecc9b2f.r43.cf2.rackcdn.com/9353-login1210_khurana.pdf) (Introducción al diseño de esquema de Apache HBase). Para ver más comandos de HBase, consulte [Guía de referencia de Apache HBase](https://hbase.apache.org/book.html#quickstart).

1. Use el comando `exit` para detener el shell interactivo de HBase. Escriba el comando siguiente:

    ```hbaseshell
    exit
    ```

**Para cargar datos de forma masiva en la tabla HBase de contactos**

HBase incluye varios métodos de carga de datos en las tablas.  Para obtener más información, vea [Carga masiva](https://hbase.apache.org/book.html#arch.bulk.load).

Se puede encontrar un archivo de datos de ejemplo en un contenedor de blobs público, `wasb://hbasecontacts\@hditutorialdata.blob.core.windows.net/contacts.txt`.  El contenido del archivo de datos es:

`8396    Calvin Raji      230-555-0191    230-555-0191    5415 San Gabriel Dr.`

`16600   Karen Wu         646-555-0113    230-555-0192    9265 La Paz`

`4324    Karl Xie         508-555-0163    230-555-0193    4912 La Vuelta`

`16891   Jonn Jackson     674-555-0110    230-555-0194    40 Ellis St.`

`3273    Miguel Miller    397-555-0155    230-555-0195    6696 Anchor Drive`

`3588    Osa Agbonile     592-555-0152    230-555-0196    1873 Lion Circle`

`10272   Julia Lee        870-555-0110    230-555-0197    3148 Rose Street`

`4868    Jose Hayes       599-555-0171    230-555-0198    793 Crawford Street`

`4761    Caleb Alexander  670-555-0141    230-555-0199    4775 Kentucky Dr.`

`16443   Terry Chander    998-555-0171    230-555-0200    771 Northridge Drive`

Opcionalmente, puede crear un archivo de texto y cargarlo en su propia cuenta de almacenamiento. Para obtener instrucciones, consulte [Carga de datos para trabajos de Apache Hadoop en HDInsight](../hdinsight-upload-data.md).

Este procedimiento usa la tabla HBase `Contacts` que creó en el último procedimiento.

1. Desde la conexión SSH abierta, ejecute el siguiente comando para transformar el archivo de datos en StoreFiles y almacenarlo en una ruta de acceso relativa especificada por `Dimporttsv.bulk.output`.

    ```bash
    hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.columns="HBASE_ROW_KEY,Personal:Name,Personal:Phone,Office:Phone,Office:Address" -Dimporttsv.bulk.output="/example/data/storeDataFileOutput" Contacts wasb://hbasecontacts@hditutorialdata.blob.core.windows.net/contacts.txt
    ```

2. Ejecute el siguiente comando para cargar los datos desde `/example/data/storeDataFileOutput` en la tabla de HBase:

    ```bash
    hbase org.apache.hadoop.hbase.mapreduce.LoadIncrementalHFiles /example/data/storeDataFileOutput Contacts
    ```

3. Puede abrir el shell de HBase y usar el comando `scan` para mostrar el contenido de la tabla.

## <a name="use-apache-hive-to-query-apache-hbase"></a>Usar Apache Hive para consultar Apache HBase

Puede consultar datos en las tablas de HBase mediante [Apache Hive](https://hive.apache.org/). En esta sección, creará una tabla de Hive que se asigna a la tabla de HBase y la usará para consultar los datos de la tabla de HBase.

1. En la conexión SSH abierta, use el siguiente comando para iniciar Beeline:

    ```bash
    beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -n admin
    ```

    Para más información sobre Beeline, consulte [Uso de Hive con Hadoop en HDInsight con Beeline](../hadoop/apache-hadoop-use-hive-beeline.md).

1. Ejecute el siguiente script de [HiveQL](https://cwiki.apache.org/confluence/display/Hive/LanguageManual) para crear una tabla de Hive que se asigne a la tabla de HBase. Asegúrese de haber creado la tabla de ejemplo a la que se hace referencia anteriormente en este artículo mediante el shell de HBase antes de ejecutar esta instrucción.

    ```hiveql
    CREATE EXTERNAL TABLE hbasecontacts(rowkey STRING, name STRING, homephone STRING, officephone STRING, officeaddress STRING)
    STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
    WITH SERDEPROPERTIES ('hbase.columns.mapping' = ':key,Personal:Name,Personal:Phone,Office:Phone,Office:Address')
    TBLPROPERTIES ('hbase.table.name' = 'Contacts');
    ```

1. Ejecute el siguiente script de HiveQL para consultar los datos de la tabla de HBase:

    ```hiveql
    SELECT count(rowkey) AS rk_count FROM hbasecontacts;
    ```

1. Para salir de Beeline, use `!exit`.

1. Para cerrar la conexión SSH, use `exit`.

## <a name="use-hbase-rest-apis-using-curl"></a>Usar las API de REST de HBase con Curl

La API de REST se protege con la [autenticación básica](https://en.wikipedia.org/wiki/Basic_access_authentication). Siempre debe crear solicitudes usando HTTP segura (HTTPS) para así garantizar que las credenciales se envían de manera segura al servidor.

1. Establezca la variable de entorno por facilidad de uso. Edite los comandos siguientes reemplazando `MYPASSWORD` por la contraseña de inicio de sesión del clúster. Reemplace `MYCLUSTERNAME` por el nombre del clúster de  HBase. Después, escriba los comandos.

    ```bash
    export password='MYPASSWORD'
    export clustername=MYCLUSTERNAME
    ```

1. Use el siguiente comando para enumerar las tablas de HBase existentes:

    ```bash
    curl -u admin:$password \
    -G https://$clustername.azurehdinsight.net/hbaserest/
    ```

1. Use el siguiente comando para crear una nueva tabla de HBase con dos familias de columnas:

    ```bash
    curl -u admin:$password \
    -X PUT "https://$clustername.azurehdinsight.net/hbaserest/Contacts1/schema" \
    -H "Accept: application/json" \
    -H "Content-Type: application/json" \
    -d "{\"@name\":\"Contact1\",\"ColumnSchema\":[{\"name\":\"Personal\"},{\"name\":\"Office\"}]}" \
    -v
    ```

    El esquema se ofrece con el formato JSon.
1. Use el siguiente comando para instalar algunos datos:

    ```bash
    curl -u admin:$password \
    -X PUT "https://$clustername.azurehdinsight.net/hbaserest/Contacts1/false-row-key" \
    -H "Accept: application/json" \
    -H "Content-Type: application/json" \
    -d "{\"Row\":[{\"key\":\"MTAwMA==\",\"Cell\": [{\"column\":\"UGVyc29uYWw6TmFtZQ==\", \"$\":\"Sm9obiBEb2xl\"}]}]}" \
    -v
    ```

    Debe codificar en Base64 los valores especificados en el modificador -d. En el ejemplo:

   * MTAwMA==: 1000
   * UGVyc29uYWw6TmFtZQ==: Personal: Nombre
   * Sm9obiBEb2xl: John Doe

     [false-row-key](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/rest/package-summary.html#operation_cell_store_single) permite insertar varios valores (por lotes).

1. Use el siguiente comando para obtener una fila:

    ```bash
    curl -u admin:$password \
    GET "https://$clustername.azurehdinsight.net/hbaserest/Contacts1/1000" \
    -H "Accept: application/json" \
    -v
    ```

Para más información sobre Rest de HBase, consulte la [guía de referencia de Apache HBase](https://hbase.apache.org/book.html#_rest).

> [!NOTE]  
> Thrift no es compatible con HBase en HDInsight.
>
> Al usar Curl o cualquier otra comunicación REST con WebHCat, debe proporcionar el nombre de usuario y la contraseña del administrador del clúster de HDInsight para autenticar las solicitudes. También debe usar el nombre del clúster como parte del identificador uniforme de recursos (URI) que se utiliza para enviar las solicitudes al servidor:
>
> `curl -u <UserName>:<Password> \`
>
> `-G https://<ClusterName>.azurehdinsight.net/templeton/v1/status`
>
> Recibirá una respuesta similar a la siguiente:
>
> `{"status":"ok","version":"v1"}`

## <a name="check-cluster-status"></a>Comprobar el estado del clúster

HBase en HDInsight se incluye con una interfaz de usuario web para la supervisión de clústeres. Mediante la interfaz de usuario web, puede solicitar estadísticas o información acerca de las regiones.

**Para acceder a la interfaz de usuario maestra de HBase**

1. Inicie sesión en la interfaz de usuario de Ambari Web en `https://CLUSTERNAME.azurehdinsight.net`, donde `CLUSTERNAME` es el nombre del clúster de HBase.

1. Seleccione **HBase** en el menú izquierdo.

1. Seleccione **Vínculos rápidos** en la parte superior de la página, seleccione el vínculo del nodo Zookeeper activo y, después, haga clic en **HBase Master UI** (Interfaz de usuario maestra de HBase).  La interfaz de usuario se abre en otra pestaña del explorador:

   ![Interfaz de usuario de Apache HBase Master de HDInsight](./media/apache-hbase-tutorial-get-started-linux/hdinsight-hbase-hmaster-ui.png)

   La interfaz de usuario maestra de HBase contiene las siguientes secciones:

   - servidores regionales
   - maestros de copia de seguridad
   - tablas
   - tareas
   - atributos de software

## <a name="clean-up-resources"></a>Limpieza de recursos

Para evitar incoherencias, recomendamos deshabilitar las tablas de HBase antes de eliminar el clúster. Puede usar el comando de HBase `disable 'Contacts'`. Si no va a seguir usando esta aplicación, puede eliminar el clúster HBase que creó mediante los siguientes pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. En el cuadro **Búsqueda** en la parte superior, escriba **HDInsight**.
1. Seleccione **Clústeres de HDInsight** en **Servicios**.
1. En la lista de clústeres de HDInsight que aparece, haga clic en el signo **...**  situado junto al clúster que ha creado para este tutorial.
1. Haga clic en **Eliminar**. Haga clic en **Sí**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a crear un clúster de Apache HBase. Y también cómo crear tablas y ver los datos de esas tablas desde el shell de HBase. También ha aprendido a usar una consulta de Hive sobre los datos de las tablas de HBase. Y también a usar las API REST de C# para HBase para crear una tabla de HBase y recuperar los datos de la tabla. Para obtener más información, consulte:

> [!div class="nextstepaction"]
> [Información general de HBase de HDInsight](./apache-hbase-overview.md)