---
title: 'Tutorial: Realización de operaciones de ETL mediante Azure Databricks'
description: Aprenda a extraer datos de Data Lake Storage Gen2 a Azure Databricks, transformarlos y luego cargarlos en Azure SQL Data Warehouse.
author: mamccrea
ms.author: mamccrea
ms.reviewer: jasonh
ms.service: azure-databricks
ms.custom: mvc
ms.topic: tutorial
ms.date: 01/24/2019
ms.openlocfilehash: b48ac9cf8eff001e62f54e41b5f76a9d006bc5ba
ms.sourcegitcommit: d2329d88f5ecabbe3e6da8a820faba9b26cb8a02
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/16/2019
ms.locfileid: "56328935"
---
# <a name="tutorial-extract-transform-and-load-data-by-using-azure-databricks"></a>Tutorial: Extracción, transformación y carga de datos mediante Azure Databricks

En este tutorial, realizará una operación de ETL (extracción, transformación y carga de datos) mediante Azure Databricks. Extraerá datos de Azure Data Lake Storage Gen2 a Azure Databricks, donde ejecutará transformaciones en ellos y, luego, cargará los datos transformados en Azure SQL Data Warehouse.

En los pasos de este tutorial se usa el conector SQL Data Warehouse para Azure Databricks para transferir los datos a Azure Databricks. Este conector usa, a su vez, Azure Blob Storage como almacenamiento temporal para los datos transferidos entre un clúster de Azure Databricks y Azure SQL Data Warehouse.

En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
> * Cree un servicio de Azure Databricks.
> * Crear un clúster de Spark en Azure Databricks.
> * Crear un sistema de archivos y cargar datos en Azure Data Lake Storage Gen2.
> * Crear una entidad de servicio.
> * Extraer datos de Data Lake Store.
> * Transformar datos en Azure Databricks.
> * Cargar datos en Azure SQL Data Warehouse

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Complete estas tareas antes de comenzar este tutorial:

* Cree una instancia de Azure SQL Data Warehouse y una regla de firewall de nivel de servidor y conéctese al servidor como administrador. Consulte [Quickstart: Creación de una instancia de Azure SQL Data Warehouse](../sql-data-warehouse/create-data-warehouse-portal.md).

* Cree una clave maestra de base de datos para la instancia de Azure SQL Data Warehouse. Consulte [Crear la clave maestra de una base de datos](https://docs.microsoft.com/sql/relational-databases/security/encryption/create-a-database-master-key).

* Cree una cuenta de Azure Data Lake Storage Gen2 Consulte [Creación de una cuenta de Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-quickstart-create-account.md).

* Ha creado una cuenta de Azure Blob Storage y, dentro de ella, un contenedor. Además, ha recuperado la clave de acceso para acceder a la cuenta de almacenamiento. Consulte [Quickstart: Cree una cuenta de Azure Blob Storage](../storage/blobs/storage-quickstart-blobs-portal.md).

* Inicie sesión en el [Azure Portal](https://portal.azure.com/).

## <a name="create-an-azure-databricks-service"></a>Creación de un servicio de Azure Databricks

En esta sección, va a crear un servicio de Azure Databricks con Azure Portal.

1. En Azure Portal, seleccione **Crear un recurso** > **Análisis** > **Azure Databricks**.

    ![Databricks en Azure Portal](./media/databricks-extract-load-sql-data-warehouse/azure-databricks-on-portal.png "Databricks en Azure Portal")

2. En **Servicio de Azure Databricks**, proporcione los valores siguientes para crear un servicio de Databricks:

    |Propiedad  |Descripción  |
    |---------|---------|
    |**Workspace name** (Nombre del área de trabajo)     | Proporcione un nombre para el área de trabajo de Databricks.        |
    |**Suscripción**     | En el cuadro desplegable, seleccione la suscripción de Azure.        |
    |**Grupos de recursos**     | Especifique si desea crear un nuevo grupo de recursos o utilizar uno existente. Un grupo de recursos es un contenedor que almacena los recursos relacionados con una solución de Azure. Para más información, consulte [Información general del grupo de recursos de Azure](../azure-resource-manager/resource-group-overview.md). |
    |**Ubicación**     | Seleccione **Oeste de EE. UU. 2**.  Para otras regiones disponibles, consulte [Productos disponibles por región](https://azure.microsoft.com/regions/services/).      |
    |**Plan de tarifa**     |  Seleccione **Estándar**.     |

3. Seleccione **Anclar al panel** y, después, seleccione **Crear**.

4. La cuenta tarda unos minutos en crearse. Durante la creación de la cuenta, el portal muestra el icono **Enviando implementación para Azure Databricks** a la derecha. Para supervisar el estado de la operación, consulte la barra de progreso en la parte superior.

    ![Icono de implementación de Databricks](./media/databricks-extract-load-sql-data-warehouse/databricks-deployment-tile.png "Databricks deployment tile")

## <a name="create-a-spark-cluster-in-azure-databricks"></a>Creación de un clúster de Spark en Azure Databricks

1. En Azure Portal, vaya al servicio de Databricks que ha creado y, después, seleccione **Launch Workspace** (Iniciar área de trabajo).

2. Se le redirigirá al portal de Azure Databricks. En el portal, seleccione **Cluster**.

    ![Databricks en Azure](./media/databricks-extract-load-sql-data-warehouse/databricks-on-azure.png "Databricks en Azure")

3. En la página **Nuevo clúster**, proporcione los valores para crear un clúster.

    ![Creación de un clúster de Databricks Spark en Azure](./media/databricks-extract-load-sql-data-warehouse/create-databricks-spark-cluster.png "Creación de un clúster de Databricks Spark en Azure")

4. Rellene los valores para los campos siguientes y acepte los valores predeterminados para el resto de campos:

    * Escriba un nombre para el clúster.

    * Para este artículo, cree un clúster con el entorno de ejecución **5.1**.

    * Asegúrese de que selecciona la casilla **Terminate after \_\_ minutes of inactivity** (Terminar después de \_\_ minutos de inactividad). Si no se usa el clúster, proporcione una duración (en minutos) para terminar el clúster.

    * Seleccione **Create cluster** (Crear clúster). Después de ejecutar el clúster, puede asociarle cuadernos y ejecutar trabajos de Spark.

## <a name="create-a-file-system-and-upload-sample-data"></a>Creación de un sistema de archivos y carga de datos de ejemplo

En primer lugar, creará un sistema de archivos en su cuenta de Data Lake Storage Gen2. Luego, puede cargar un archivo de datos de ejemplo en Data Lake Store. Usará este archivo más adelante en Azure Databricks para ejecutar algunas transformaciones.

1. Descargue el archivo de datos de ejemplo [small_radio_json.json](https://github.com/Azure/usql/blob/master/Examples/Samples/Data/json/radiowebsite/small_radio_json.json) en el sistema de archivos local.

2. En [Azure Portal](https://portal.azure.com/), vaya a la cuenta de Data Lake Storage Gen2 que ha creado como requisito previo de este tutorial.

3. En la página **Información general** de la cuenta de almacenamiento, seleccione **Abrir en el Explorador**.

   ![Abrir el Explorador de Storage](./media/databricks-extract-load-sql-data-warehouse/data-lake-storage-open-storage-explorer.png "Open Storage Explorer")

4. Seleccione **Abrir Explorador de Azure Storage** para abrir el explorador de Storage.

   ![Segundo aviso para abrir el Explorador de Storage](./media/databricks-extract-load-sql-data-warehouse/data-lake-storage-open-storage-explorer-2.png "Open Storage Explorer second prompt")

   Se abre el Explorador de Storage. Puede crear un sistema de archivos y cargar los datos de ejemplo mediante las instrucciones de este tema: [Inicio rápido: Uso del Explorador de Azure Storage para administrar los datos de una cuenta de Data Lake Storage Gen2](../storage/blobs/data-lake-storage-explorer.md).

<a id="service-principal"/>

## <a name="create-a-service-principal"></a>Creación de una entidad de servicio

Cree a una entidad de servicio siguiendo la guía de este tema: [Uso de portal para crear una aplicación de Azure AD y una entidad de servicio que puedan acceder a los recursos](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal).

Hay un par de cosas que tendrá que hacer cuando realice los pasos de este artículo.

:heavy_check_mark: Al realizar los pasos que se describen en la sección [Asignación de la aplicación a un rol](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#assign-the-application-to-a-role), asegúrese de asignar la aplicación al **rol de colaborador de Blob Storage**.

:heavy_check_mark: Al realizar los pasos que se describen en la sección [Obtención de valores para el inicio de sesión](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#get-values-for-signing-in) del artículo, pegue el identificador del inquilino, el identificador de la aplicación y los valores de la clave de autenticación en una clave de texto, ya que los necesitará pronto.
En primer lugar, va a crear un cuaderno en el área de trabajo de Azure Databricks y, después, ejecutará fragmentos de código para crear el sistema de archivos en la cuenta de almacenamiento.

## <a name="extract-data-from-the-data-lake-store"></a>Extracción de datos de Data Lake Store

En esta sección, creará un cuaderno en el área de trabajo de Azure Databricks y, luego, ejecutará fragmentos de código para extraer datos de Data Lake Store en Azure Databricks.

1. En [Azure Portal](https://portal.azure.com), vaya al servicio de Azure Databricks que ha creado y, después, seleccione **Launch Workspace** (Iniciar área de trabajo).

2. A la izquierda, seleccione **Workspace** (Área de trabajo). En la lista desplegable **Workspace** (Área de trabajo), seleccione **Create** > **Notebook** (Crear > Cuaderno).

    ![Creación de un cuaderno en Databricks](./media/databricks-extract-load-sql-data-warehouse/databricks-create-notebook.png "Create notebook in Databricks")

3. En el cuadro de diálogo **Create Notebook** (Crear cuaderno), escriba un nombre para el cuaderno. Seleccione **Scala** como lenguaje y, a continuación, seleccione el clúster de Spark que creó anteriormente.

    ![Proporcionar detalles para un cuaderno en Databricks](./media/databricks-extract-load-sql-data-warehouse/databricks-notebook-details.png "Provide details for a notebook in Databricks")

4. Seleccione **Crear**.

5. Copie y pegue el siguiente bloque de código en la primera celda.

   ```scala
   spark.conf.set("fs.azure.account.auth.type.<storage-account-name>.dfs.core.windows.net", "OAuth")
   spark.conf.set("fs.azure.account.oauth.provider.type.<storage-account-name>.dfs.core.windows.net", "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider")
   spark.conf.set("fs.azure.account.oauth2.client.id.<storage-account-name>.dfs.core.windows.net", "<application-id>")
   spark.conf.set("fs.azure.account.oauth2.client.secret.<storage-account-name>.dfs.core.windows.net", "<authentication-key>")
   spark.conf.set("fs.azure.account.oauth2.client.endpoint.<storage-account-name>.dfs.core.windows.net", "https://login.microsoftonline.com/<tenant-id>/oauth2/token")
   ```

6. En este bloque de código, reemplace los valores de marcado de posición `application-id`, `authentication-id` y `tenant-id` de este bloque de código por los valores que recopiló al realizar los pasos de la sección Reserva de la configuración de la cuenta de almacenamiento. Reemplace el valor de marcador de posición `storage-account-name` por el nombre de la cuenta de almacenamiento.

7. Presione las teclas **MAYÚS + ENTRAR** para ejecutar el código de este bloque.

8. Ahora puede cargar el archivo JSON de ejemplo como trama de datos en Azure Databricks. Pegue el código siguiente en una nueva celda. Reemplace los marcadores de posición que se muestran entre paréntesis por sus valores.

   ```scala
   val df = spark.read.json("abfss://<file-system-name>@<storage-account-name>.dfs.core.windows.net/small_radio_json.json")
   ```

   * Reemplace el valor de marcador de posición `file-system-name` por el nombre que asignó a su sistema de archivos en el Explorador de Storage.

   * Reemplace el marcador de posición `storage-account-name` por el nombre de la cuenta de almacenamiento.

9. Presione las teclas **MAYÚS + ENTRAR** para ejecutar el código de este bloque.

10. Ejecute el código siguiente para ver el contenido de la trama de datos:

    ```scala
    df.show()
    ```
   Debe ver una salida similar al siguiente fragmento de código:

   ```bash
   +---------------------+---------+---------+------+-------------+----------+---------+-------+--------------------+------+--------+-------------+---------+--------------------+------+-------------+------+
   |               artist|     auth|firstName|gender|itemInSession|  lastName|   length|  level|            location|method|    page| registration|sessionId|                song|status|           ts|userId|
   +---------------------+---------+---------+------+-------------+----------+---------+-------+--------------------+------+--------+-------------+---------+--------------------+------+-------------+------+
   | El Arrebato         |Logged In| Annalyse|     F|            2|Montgomery|234.57914| free  |  Killeen-Temple, TX|   PUT|NextSong|1384448062332|     1879|Quiero Quererte Q...|   200|1409318650332|   309|
   | Creedence Clearwa...|Logged In|   Dylann|     M|            9|    Thomas|340.87138| paid  |       Anchorage, AK|   PUT|NextSong|1400723739332|       10|        Born To Move|   200|1409318653332|    11|
   | Gorillaz            |Logged In|     Liam|     M|           11|     Watts|246.17751| paid  |New York-Newark-J...|   PUT|NextSong|1406279422332|     2047|                DARE|   200|1409318685332|   201|
   ...
   ...
   ```

   Ahora ha extraído los datos de Azure Data Lake Storage Gen2 en Azure Databricks.

## <a name="transform-data-in-azure-databricks"></a>Transformación de datos en Azure Databricks

Los datos de ejemplo sin procesar del archivo **small_radio_json.json** capturan la audiencia de una emisora de radio y presentan una variedad de columnas. En esta sección, va a transformar los datos para recuperar solo columnas específicas del conjunto de datos.

1. En primer lugar, recupere solo las columnas **firstName**, **lastName**, **gender**, **location** y **level** de la trama de datos que ha creado.

   ```scala
   val specificColumnsDf = df.select("firstname", "lastname", "gender", "location", "level")
   specificColumnsDf.show()
   ```

   Recibirá una salida como la que se muestra en el fragmento de código siguiente:

   ```bash
   +---------+----------+------+--------------------+-----+
   |firstname|  lastname|gender|            location|level|
   +---------+----------+------+--------------------+-----+
   | Annalyse|Montgomery|     F|  Killeen-Temple, TX| free|
   |   Dylann|    Thomas|     M|       Anchorage, AK| paid|
   |     Liam|     Watts|     M|New York-Newark-J...| paid|
   |     Tess|  Townsend|     F|Nashville-Davidso...| free|
   |  Margaux|     Smith|     F|Atlanta-Sandy Spr...| free|
   |     Alan|     Morse|     M|Chicago-Napervill...| paid|
   |Gabriella|   Shelton|     F|San Jose-Sunnyval...| free|
   |   Elijah|  Williams|     M|Detroit-Warren-De...| paid|
   |  Margaux|     Smith|     F|Atlanta-Sandy Spr...| free|
   |     Tess|  Townsend|     F|Nashville-Davidso...| free|
   |     Alan|     Morse|     M|Chicago-Napervill...| paid|
   |     Liam|     Watts|     M|New York-Newark-J...| paid|
   |     Liam|     Watts|     M|New York-Newark-J...| paid|
   |   Dylann|    Thomas|     M|       Anchorage, AK| paid|
   |     Alan|     Morse|     M|Chicago-Napervill...| paid|
   |   Elijah|  Williams|     M|Detroit-Warren-De...| paid|
   |  Margaux|     Smith|     F|Atlanta-Sandy Spr...| free|
   |     Alan|     Morse|     M|Chicago-Napervill...| paid|
   |   Dylann|    Thomas|     M|       Anchorage, AK| paid|
   |  Margaux|     Smith|     F|Atlanta-Sandy Spr...| free|
   +---------+----------+------+--------------------+-----+
   ```

2. Puede transformar aún más estos datos para renombrar la columna **level** como **subscription_type**.

   ```scala
   val renamedColumnsDF = specificColumnsDf.withColumnRenamed("level", "subscription_type")
   renamedColumnsDF.show()
   ```

   Recibirá una salida como la que se muestra en el fragmento de código siguiente.

   ```bash
   +---------+----------+------+--------------------+-----------------+
   |firstname|  lastname|gender|            location|subscription_type|
   +---------+----------+------+--------------------+-----------------+
   | Annalyse|Montgomery|     F|  Killeen-Temple, TX|             free|
   |   Dylann|    Thomas|     M|       Anchorage, AK|             paid|
   |     Liam|     Watts|     M|New York-Newark-J...|             paid|
   |     Tess|  Townsend|     F|Nashville-Davidso...|             free|
   |  Margaux|     Smith|     F|Atlanta-Sandy Spr...|             free|
   |     Alan|     Morse|     M|Chicago-Napervill...|             paid|
   |Gabriella|   Shelton|     F|San Jose-Sunnyval...|             free|
   |   Elijah|  Williams|     M|Detroit-Warren-De...|             paid|
   |  Margaux|     Smith|     F|Atlanta-Sandy Spr...|             free|
   |     Tess|  Townsend|     F|Nashville-Davidso...|             free|
   |     Alan|     Morse|     M|Chicago-Napervill...|             paid|
   |     Liam|     Watts|     M|New York-Newark-J...|             paid|
   |     Liam|     Watts|     M|New York-Newark-J...|             paid|
   |   Dylann|    Thomas|     M|       Anchorage, AK|             paid|
   |     Alan|     Morse|     M|Chicago-Napervill...|             paid|
   |   Elijah|  Williams|     M|Detroit-Warren-De...|             paid|
   |  Margaux|     Smith|     F|Atlanta-Sandy Spr...|             free|
   |     Alan|     Morse|     M|Chicago-Napervill...|             paid|
   |   Dylann|    Thomas|     M|       Anchorage, AK|             paid|
   |  Margaux|     Smith|     F|Atlanta-Sandy Spr...|             free|
   +---------+----------+------+--------------------+-----------------+
   ```

## <a name="load-data-into-azure-sql-data-warehouse"></a>Carga de datos en Azure SQL Data Warehouse

En esta sección, cargará los datos transformados en Azure SQL Data Warehouse. Utilice el conector de Azure SQL Data Warehouse para Azure Databricks para cargar directamente una trama de datos como una tabla en SQL Data Warehouse.

Como se mencionó anteriormente, el conector de SQL Data Warehouse usa Azure Blob Storage como almacenamiento temporal para cargar datos entre Azure Databricks y Azure SQL Data Warehouse. Por lo tanto, para comenzar, proporcione la configuración para conectarse a la cuenta de almacenamiento. Ya debe haber creado la cuenta como parte de los requisitos previos de este artículo.

1. Proporcione la configuración para acceder a la cuenta de Azure Storage desde Azure Databricks.

   ```scala
   val blobStorage = "<blob-storage-account-name>.blob.core.windows.net"
   val blobContainer = "<blob-container-name>"
   val blobAccessKey =  "<access-key>"
   ```

2. Especifique una carpeta temporal que usará al mover datos entre Azure Databricks y Azure SQL Data Warehouse.

   ```scala
   val tempDir = "wasbs://" + blobContainer + "@" + blobStorage +"/tempDirs"
   ```

3. Ejecute el siguiente fragmento de código para almacenar las claves de acceso de Azure Blob Storage en la configuración. Esta acción garantiza que no tiene que guardar la clave de acceso en el cuaderno en texto sin formato.

   ```scala
   val acntInfo = "fs.azure.account.key."+ blobStorage
   sc.hadoopConfiguration.set(acntInfo, blobAccessKey)
   ```

4. Proporcione los valores para conectarse a la instancia de Azure SQL Data Warehouse. Debe haber creado una instancia de SQL Data Warehouse como requisito previo.

   ```scala
   //SQL Data Warehouse related settings
   val dwDatabase = "<database-name>"
   val dwServer = "<database-server-name>"
   val dwUser = "<user-name>"
   val dwPass = "<password>"
   val dwJdbcPort =  "1433"
   val dwJdbcExtraOptions = "encrypt=true;trustServerCertificate=true;hostNameInCertificate=*.database.windows.net;loginTimeout=30;"
   val sqlDwUrl = "jdbc:sqlserver://" + dwServer + ".database.windows.net:" + dwJdbcPort + ";database=" + dwDatabase + ";user=" + dwUser+";password=" + dwPass + ";$dwJdbcExtraOptions"
   val sqlDwUrlSmall = "jdbc:sqlserver://" + dwServer + ".database.windows.net:" + dwJdbcPort + ";database=" + dwDatabase + ";user=" + dwUser+";password=" + dwPass
   ```

5. Ejecute el siguiente fragmento de código para cargar la trama de datos transformada, **renamedColumnsDf**, como una tabla en SQL Data Warehouse. Este fragmento de código crea una tabla denominada **SampleTable** en SQL Database.

   ```scala
   spark.conf.set(
       "spark.sql.parquet.writeLegacyFormat",
       "true")

   renamedColumnsDF.write
       .format("com.databricks.spark.sqldw")
       .option("url", sqlDwUrlSmall) 
       .option("dbtable", "SampleTable")
       .option( "forward_spark_azure_storage_credentials","True")
       .option("tempdir", tempDir)
       .mode("overwrite")
       .save()
   ```

6. Conéctese a la instancia de SQL Database y compruebe que aparece la base de datos denominada **SampleTable**.

   ![Comprobación de la tabla de ejemplo](./media/databricks-extract-load-sql-data-warehouse/verify-sample-table.png "Verify sample table")

7. Ejecute una consulta select para comprobar el contenido de la tabla. La tabla debe tener los mismos datos que la trama de datos **renamedColumnsDF**.

    ![Comprobación del contenido de tabla de ejemplo](./media/databricks-extract-load-sql-data-warehouse/verify-sample-table-content.png "Verify the sample table content")

## <a name="clean-up-resources"></a>Limpieza de recursos

Después de terminar el tutorial, puede finalizar el clúster. Desde el área de trabajo de Azure Databricks, seleccione **Clusters** (Clústeres) a la izquierda. Para que el clúster termine, en **Acciones**, apunte a los puntos suspensivos (...) y seleccione el icono **Terminate** (Finalizar).

![Detener un clúster de Databricks](./media/databricks-extract-load-sql-data-warehouse/terminate-databricks-cluster.png "Stop a Databricks cluster")

Si no finaliza manualmente el clúster, este se detendrá automáticamente si seleccionó la casilla **Terminate after \_\_ minutes of inactivity** (Finalizar después de __ minutos de inactividad) al crear el clúster. En tal caso, el clúster se detiene automáticamente si ha estado inactivo durante el tiempo especificado.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial aprendió lo siguiente:

> [!div class="checklist"]
> * Creación de un servicio de Azure Databricks
> * Creación de un clúster de Spark en Azure Databricks
> * Creación de un cuaderno en Azure Databricks
> * Extracción de datos desde la cuenta de Data Lake Storage Gen2
> * Transformación de datos en Azure Databricks
> * Carga de datos en Azure SQL Data Warehouse

Vaya al siguiente tutorial para aprender sobre el streaming de datos en tiempo real en Azure Databricks mediante Azure Event Hubs.

> [!div class="nextstepaction"]
>[Transmisión de datos a Azure Databricks con Event Hubs](databricks-stream-from-eventhubs.md)
