---
title: Integración de datos mediante Azure Data Factory y Azure Data Share
description: Copie, transforme y comparta datos mediante Azure Data Factory y Azure Data Share.
author: djpmsft
ms.author: daperlov
ms.service: data-factory
ms.topic: tutorial
ms.custom: seo-lt-2019
ms.date: 01/08/2020
ms.openlocfilehash: c43cac4d599753ecc3486ef7b86aa54b5697b0f6
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89435666"
---
# <a name="data-integration-using-azure-data-factory-and-azure-data-share"></a>Integración de datos mediante Azure Data Factory y Azure Data Share

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

A medida que los clientes se embarcan en sus modernos proyectos de almacenamiento y análisis de datos, no solo necesitan más datos sino también más visibilidad de los datos en sus diferentes estados. En este taller se analiza cómo las mejoras en Azure Data Factory y Azure Data Share simplifican la integración y administración de los datos en Azure. 

Desde la habilitación de procesos de ETL/ELT sin código hasta la creación de una vista completa de los datos, las mejoras en Azure Data Factory permitirán a los ingenieros de datos aportar de forma segura más datos y, por tanto, más valor a la empresa. Azure Data Share permitirá compartir datos entre negocios de una manera controlada.

En este taller usará Azure Data Factory (ADF) para la ingesta de datos de Azure SQL Database en Azure Data Lake Storage Gen2 (ADLS Gen2). Cuando ya disponga de los datos en el lago, los transformará mediante flujos de datos de asignación, el servicio de transformación nativo de Azure Data Factory, y los recibirá en Azure Synapse Analytics (anteriormente SQL DW). A continuación, compartirá la tabla que incluye los datos transformados junto con algunos datos adicionales mediante Azure Data Share. 

En este laboratorio se usan datos de taxis de la ciudad de Nueva York. Para importarlos en la base de datos de SQL Database, descargue el [archivo taxi-data.bacpac](https://github.com/djpmsft/ADF_Labs/blob/master/sample-data/taxi-data.bacpac).

## <a name="prerequisites"></a>Requisitos previos

* **Suscripción de Azure**: Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

* **Azure SQL Database**: si no tiene una base de datos de SQL, aprenda a [crear una cuenta de Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-single-database-get-started?tabs=azure-portal).

* **Cuenta de almacenamiento de Azure Data Lake Storage Gen2**: si no la tiene, aprenda a [crear una cuenta de almacenamiento de ADLS Gen2](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-quickstart-create-account).

* **Azure Synapse Analytics (anteriormente SQL DW)** : si no la tiene, aprenda a [crear una instancia de Azure Synapse Analytics](https://docs.microsoft.com/azure/sql-data-warehouse/create-data-warehouse-portal).

* **Azure Data Factory**: si no la ha creado, consulte cómo [crear una factoría de datos](https://docs.microsoft.com/azure/data-factory/quickstart-create-data-factory-portal).

* **Azure Data Share**: si no lo ha creado, consulte cómo [crear un recurso compartido de datos](https://docs.microsoft.com/azure/data-share/share-your-data#create-a-data-share-account).

## <a name="set-up-your-azure-data-factory-environment"></a>Configuración del entorno de Azure Data Factory

En esta sección aprenderá a acceder a la experiencia del usuario de Azure Data Factory (ADF UX) en Azure Portal. Una vez en la experiencia de usuario de ADF, configurará tres servicios vinculados para cada uno de los almacenes de datos que usamos: Azure SQL DB, ADLS Gen2 y Azure Synapse Analytics.

En Azure Data Factory, los servicios vinculados definen la información de conexión a los recursos externos. Actualmente, Azure Data Factory admite más de 85 conectores.

### <a name="open-the-azure-data-factory-ux"></a>Apertura de la experiencia de usuario de Azure Data Factory

1. Abra [Azure Portal](https://portal.azure.com) en Microsoft Edge o Google Chrome.
1. Mediante la barra de búsqueda de la parte superior de la página, busque "Factorías de datos".

    ![Portal 1](media/lab-data-flow-data-share/portal1.png)
1. Haga clic en el recurso de factoría de datos para abrir su hoja de recursos.

    ![Portal 2](media/lab-data-flow-data-share/portal2.png)
1. Haga clic en **Crear y supervisar** para abrir la experiencia de usuario de ADF. También se puede acceder a la experiencia de usuario de ADF en adf.azure.com.

    ![Portal 3](media/lab-data-flow-data-share/portal3.png)
1. Se le redirigirá a la página principal de la experiencia del usuario de ADF. Esta página contiene inicios rápidos, vídeos de instrucciones y vínculos a tutoriales para aprender los conceptos de factoría de datos. Para iniciar la creación, haga clic en el icono de lápiz en la barra lateral izquierda.

    ![Configuración en el portal](media/lab-data-flow-data-share/configure1.png)

### <a name="create-an-azure-sql-database-linked-service"></a>Creación de un servicio vinculado de Azure SQL Database

1. En la página de creación se crean los recursos de Azure Data Factory, como canalizaciones, conjuntos de datos, flujos de datos, desencadenadores y servicios vinculados. Para crear un servicio vinculado, haga clic en el botón **Connections** (Conexiones) en la esquina inferior derecha.

    ![Configuración en el portal 2](media/lab-data-flow-data-share/configure2.png)
1. En la pestaña Connections (Conexiones), haga clic en **New** (Nuevo) para agregar un nuevo servicio vinculado.

    ![Configuración en el portal 3](media/lab-data-flow-data-share/configure3.png)
1. El primer servicio vinculado que configurará es el correspondiente a Azure SQL Database. Puede usar la barra de búsqueda para filtrar la lista de almacenes de datos. Haga clic en el icono **Azure SQL Database** y, después, en Continue (Continuar).

    ![Configuración en el portal 4](media/lab-data-flow-data-share/configure4.png)
1. En el panel de configuración de Azure SQL Database, escriba "SQLDB" como nombre del servicio vinculado. Escriba las credenciales para permitir que la factoría de datos se conecte a la base de datos. Si utiliza la autenticación de SQL, escriba el nombre del servidor, la base de datos, el nombre de usuario y la contraseña. Para comprobar que la información de conexión es correcta, haga clic en **Test connection** (Probar conexión). Cuando haya terminado, haga clic en **Create** (Crear).

    ![Configuración en el portal 5](media/lab-data-flow-data-share/configure5.png)

### <a name="create-an-azure-synapse-analytics-linked-service"></a>Creación de un servicio vinculado de Azure Synapse Analytics

1. Repita el mismo proceso para agregar un servicio vinculado de Azure Synapse Analytics. En la pestaña Connections (Conexiones), haga clic en **New** (Nuevo). Seleccione el icono **Azure Synapse Analytics (formerly SQL DW)** (Azure Synapse Analytics [anteriormente SQL DW]) y haga clic en Continue (Continuar).

    ![Configuración en el portal 6](media/lab-data-flow-data-share/configure6.png)
1. En el panel de configuración del servicio vinculado, escriba "SQLDW" como nombre del servicio vinculado. Escriba las credenciales para permitir que la factoría de datos se conecte a la base de datos. Si utiliza la autenticación de SQL, escriba el nombre del servidor, la base de datos, el nombre de usuario y la contraseña. Para comprobar que la información de conexión es correcta, haga clic en **Test connection** (Probar conexión). Cuando haya terminado, haga clic en **Create** (Crear).

    ![Configuración en el portal 7](media/lab-data-flow-data-share/configure7.png)

### <a name="create-an-azure-data-lake-storage-gen2-linked-service"></a>Creación de un servicio vinculado de Azure Data Lake Storage Gen2

1. El último servicio vinculado que se necesita para este laboratorio es el que corresponde a Azure Data Lake Storage Gen2.  En la pestaña Connections (Conexiones), haga clic en **New** (Nuevo). Seleccione el icono **Azure Data Lake Storage Gen2** y haga clic en Continue (Continuar).

    ![Configuración en el portal 8](media/lab-data-flow-data-share/configure8.png)
1. En el panel de configuración del servicio vinculado, escriba "ADLSGen2" como nombre del servicio vinculado. Si usa la autenticación de clave de cuenta, seleccione la cuenta de almacenamiento de ADLS Gen2 en la lista desplegable **Storage account name** (Nombre de la cuenta de Storage). Para comprobar que la información de conexión es correcta, haga clic en **Test connection** (Probar conexión). Cuando haya terminado, haga clic en **Create** (Crear).

    ![Configuración en el portal 9](media/lab-data-flow-data-share/configure9.png)

### <a name="turn-on-data-flow-debug-mode"></a>Activación del modo de depuración de flujos de datos

En la sección *Transformación de datos mediante flujos de datos de asignación* creará este tipo de flujos de datos. Antes de crear flujos de datos de asignación, se recomienda activar el modo de depuración, que permite probar la lógica de transformación en segundos en un clúster de Spark activo.

Para activar la depuración, haga clic en el control deslizante **Data flow debug** (Depuración de flujo de datos) en la barra superior de Azure Data Factory. Haga clic en OK (Aceptar) cuando aparezca el cuadro de diálogo de confirmación. El clúster tardará entre 5 y 7 minutos en iniciarse. Continúe en *Ingesta de datos de Azure SQL Database en ADLS Gen2 mediante la actividad de copia* mientras se inicializa.

![Configuración en el portal 10](media/lab-data-flow-data-share/configure10.png)

## <a name="ingest-data-using-the-copy-activity"></a>Ingesta de datos mediante la actividad de copia

En esta sección, creará una canalización con una actividad de copia que ingiere una tabla de una instancia de Azure SQL Database en una cuenta de almacenamiento de ADLS Gen2. Aprenderá a agregar una canalización, a configurar un conjunto de datos y a depurar una canalización a través de la experiencia del usuario de ADF. El patrón de configuración que se usa en esta sección se puede aplicar a la copia desde un almacén de datos relacional a un almacén de datos basado en archivos.

En Azure Data Factory, una canalización es una agrupación lógica de actividades que realizan conjuntamente una tarea. Una actividad define una operación que se realizará en los datos. Un conjunto de datos apunta a los datos que se desean usar en un servicio vinculado.

### <a name="create-a-pipeline-with-a-copy-activity"></a>Creación de una canalización con una actividad de copia

1. En el panel de recursos de Azure Data Factory, haga clic en el icono de signo más para abrir el menú de nuevo recurso. Seleccione **Pipeline** (Canalización).

    ![Copia en el portal 1](media/lab-data-flow-data-share/copy1.png)
1. En la pestaña **General** del lienzo de la canalización, asigne un nombre descriptivo a la canalización, como "IngestAndTransformTaxiData".

    ![Copia en el portal 2](media/lab-data-flow-data-share/copy2.png)
1. En el panel de actividades del lienzo de la canalización, abra el acordeón **Move and Transform** (Mover y transformar) y arrastre la actividad **Copy data** (Copiar datos) al lienzo. Asigne un nombre descriptivo a la actividad de copia, como "IngestIntoADLS".

    ![Copia en el portal 3](media/lab-data-flow-data-share/copy3.png)

### <a name="configure-azure-sql-db-source-dataset"></a>Configuración del conjunto de datos de origen de Azure SQL Database

1. Haga clic en la pestaña **Source** (Origen) de la actividad de copia. Para crear un nuevo conjunto de datos, haga clic en **New** (Nuevo). El origen será la tabla "dbo.TripData" que se encuentra en el servicio vinculado "SQLDB" configurado anteriormente.

    ![Copia en el portal 4](media/lab-data-flow-data-share/copy4.png)
1. Busque **Azure SQL Database** y haga clic en Continue (Continuar).

    ![Copia en el portal 5](media/lab-data-flow-data-share/copy5.png)
1. Asigne al conjunto de datos el nombre "TripData". Seleccione "SQLDB" como servicio vinculado. Seleccione "dbo.TripData" en la lista desplegable de nombre de tabla. Importe el esquema con la opción **From connection/store** (Desde la conexión o almacén). Haga clic en OK (Aceptar) cuando haya finalizado.

    ![Copia en el portal 6](media/lab-data-flow-data-share/copy6.png)

Ha creado correctamente el conjunto de datos de origen. Asegúrese de que la configuración de origen tiene seleccionado el valor predeterminado **Table** (Tabla) en el campo de uso de consulta.

### <a name="configure-adls-gen2-sink-dataset"></a>Configuración del conjunto de datos del receptor de ADLS Gen2

1. Haga clic en la pestaña **Sink** (Receptor) de la actividad de copia. Para crear un nuevo conjunto de datos, haga clic en **New** (Nuevo).

    ![Copia en el portal 7](media/lab-data-flow-data-share/copy7.png)
1. Busque **Azure Data Lake Storage Gen2** y haga clic en Continue (Continuar).

    ![Copia en el portal 8](media/lab-data-flow-data-share/copy8.png)
1. En el panel de selección de formato, elija **DelimitedText**, ya que escribe en un archivo CSV. Haga clic en Continue (Continuar).

    ![Copia en el portal 9](media/lab-data-flow-data-share/copy9.png)
1. Asigne el nombre "TripDataCSV" al conjunto de datos del receptor. Seleccione "ADLSGen2" como servicio vinculado. Indique dónde desea escribir el archivo CSV. Por ejemplo, puede escribir los datos en el archivo `trip-data.csv` del contenedor `staging-container`. Establezca **First row as header** (Primera fila como encabezado) en verdadero, ya que desea que los datos de salida tengan encabezados. Puesto que todavía no existe ningún archivo en el destino, establezca **Import schema** (Importar esquema) en **None** (Ninguno). Haga clic en OK (Aceptar) cuando haya finalizado.

    ![Copia en el portal 10](media/lab-data-flow-data-share/copy10.png)

### <a name="test-the-copy-activity-with-a-pipeline-debug-run"></a>Prueba de la actividad de copia con una ejecución de depuración de la canalización

1. Para comprobar que la actividad de copia funciona correctamente, haga clic en **Debug** (Depurar) en la parte superior del lienzo de la canalización para realizar una ejecución de depuración. Este tipo de ejecución permite probar la canalización completa o hasta un punto de interrupción, antes de publicarla en el servicio de Azure Data Factory.

    ![Copia en el portal 11](media/lab-data-flow-data-share/copy11.png)
1. Para supervisar la ejecución de depuración, vaya a la pestaña **Output** (Salida) del lienzo de la canalización. La pantalla de supervisión se actualizará automáticamente cada 20 segundos o cuando haga clic manualmente en el botón de actualización. La actividad de copia tiene una vista de supervisión especial a la que se puede acceder haciendo clic en el icono con forma de gafas de la columna **Actions** (Acciones).

    ![Copia en el portal 12](media/lab-data-flow-data-share/copy12.png)
1. La vista de supervisión de la copia proporciona los detalles de ejecución y las características de rendimiento de la actividad. Puede ver información como los datos leídos y escritos, las filas leídas y escritas, los archivos leídos y escritos, y el rendimiento. Si ha configurado todo correctamente, debería ver 49 999 filas escritas en un archivo del receptor de ADLS.

    ![Copia en el portal 13](media/lab-data-flow-data-share/copy13.png)
1. Antes de pasar a la sección siguiente, se recomienda publicar los cambios en el servicio de Azure Data Factory haciendo clic en **Publish all** (Publicar todo) en la barra superior de Azure Data Factory. Aunque no se trata en este laboratorio, Azure Data Factory admite la integración completa de Git. Esta integración permite realizar control de versiones, el almacenamiento iterativo en un repositorio y la colaboración en una factoría de datos. Para más información, consulte [Control de código fuente en Azure Data Factory](https://docs.microsoft.com/azure/data-factory/source-control#troubleshooting-git-integration).

    ![Publicación en el portal 1](media/lab-data-flow-data-share/publish1.png)

## <a name="transform-data-using-mapping-data-flow"></a>Transformación de datos mediante Mapping Data Flow

Ahora que ha copiado correctamente los datos en Azure Data Lake Storage, es el momento de combinarlos y agregarlos a un almacén de datos. Usaremos el flujo de datos de asignación, el servicio de transformación con un diseño visual de Azure Data Factory. Los flujos de datos de asignación permiten a los usuarios desarrollar lógica de transformación sin código y ejecutarlos en clústeres de Spark administrados por el servicio ADF.

El flujo de datos creado en este paso realiza una combinación interna del conjunto de datos "TripDataCSV" creado en la sección anterior con una tabla "dbo.TripFares" almacenada en "SQLDB" basándose en cuatro columnas de clave. Después, los datos se agregan en función de la columna `payment_type` para calcular la media de determinados campos y se escriben en una tabla de Azure Synapse Analytics.

### <a name="add-a-data-flow-activity-to-your-pipeline"></a>Incorporación de una actividad de flujo de datos a la canalización

1. En el panel de actividades del lienzo de la canalización, abra el acordeón **Move and Transform** (Mover y transformar) y arrastre la actividad **Data flow** (Flujo de datos) al lienzo.

    ![Flujo de datos en el portal 1](media/lab-data-flow-data-share/dataflow1.png)
1. En el panel lateral que se abre, seleccione **Create new data flow (Crear nuevo flujo de datos)** y elija **Mapping data flow** (Flujo de datos de asignación). Haga clic en **OK**.

    ![Flujo de datos en el portal 2](media/lab-data-flow-data-share/dataflow2.png)
1. Se le redirigirá al lienzo de flujo de datos, donde creará la lógica de transformación. En la pestaña General, asigne el nombre "JoinAndAggregateData" al flujo de datos.

    ![Flujo de datos en el portal 3](media/lab-data-flow-data-share/dataflow3.png)

### <a name="configure-your-trip-data-csv-source"></a>Configuración del origen de archivo CSV para los datos de carreras

1. Lo primero que desea hacer es configurar las dos transformaciones de origen. El primer origen apuntará al conjunto de datos "TripDataCSV" de tipo DelimitedText. Para agregar una transformación de origen, haga clic en el cuadro **Add Source** (Agregar origen) en el lienzo.

    ![Flujo de datos en el portal 4](media/lab-data-flow-data-share/dataflow4.png)
1. Asigne al origen el nombre "TripDataCSV" y seleccione el conjunto de datos "TripDataCSV" de la lista desplegable de origen. Si lo recuerda, inicialmente no importó un esquema al crear este conjunto de datos, ya que no había ningún dato en él. Puesto que ahora existe `trip-data.csv`, haga clic en **Edit** (Editar) para ir a la pestaña de configuración del conjunto de datos.

    ![Flujo de datos en el portal 5](media/lab-data-flow-data-share/dataflow5.png)
1. Vaya a la pestaña **Schema** (Esquema) y haga clic en **Import schema** (Importar esquema). Seleccione **From Connection/Store** (Desde la conexión o almacén) para importar directamente desde el almacén de archivos. Deben aparecer 14 columnas de tipo cadena.

    ![Flujo de datos en el portal 6](media/lab-data-flow-data-share/dataflow6.png)
1. Vuelva al flujo de datos "JoinAndAggregateData". Si el clúster de depuración se ha iniciado (lo que se indica mediante un círculo verde junto al control deslizante de depuración), puede obtener una instantánea de los datos en la pestaña **Data Preview** (Vista previa de datos). Haga clic en **Refresh** (Actualizar) para capturar una vista previa de los datos.

    ![Flujo de datos en el portal 7](media/lab-data-flow-data-share/dataflow7.png)

> [!Note]
> La vista previa de los datos no escribe datos.

### <a name="configure-your-trip-fares-sql-db-source"></a>Configuración del origen de Azure SQL Database para las tarifas de carrera

1. El segundo origen que va a agregar apuntará a la tabla de Azure SQL Database "dbo.TripFares". Bajo el origen "TripDataCSV", habrá otro cuadro **Add Source** (Agregar origen). Haga clic en él para agregar una nueva transformación de origen.

    ![Flujo de datos en el portal 8](media/lab-data-flow-data-share/dataflow8.png)
1. Asigne a este origen el nombre "TripFaresSQL". Haga clic en **New** (Nuevo) junto al campo del conjunto de datos de origen para crear un nuevo conjunto de datos de Azure SQL Database.

    ![Flujo de datos en el portal 9](media/lab-data-flow-data-share/dataflow9.png)
1. Seleccione el icono **Azure SQL Database** y haga clic en Continue (Continuar). *Nota: puede observar que muchos de los conectores de Azure Data Factory no se admiten en el flujo de datos de asignación. Para transformar los datos de uno de estos orígenes, realice la ingesta en un origen compatible mediante la actividad de copia*.

    ![Flujo de datos en el portal 10](media/lab-data-flow-data-share/dataflow10.png)
1. Llame al conjunto de datos "TripFares". Seleccione "SQLDB" como servicio vinculado. Seleccione "dbo.TripFares" en la lista desplegable de nombre de tabla. Importe el esquema con la opción **From connection/store** (Desde la conexión o almacén). Haga clic en OK (Aceptar) cuando haya finalizado.

    ![Flujo de datos en el portal 11](media/lab-data-flow-data-share/dataflow11.png)
1. Para comprobar los datos, capture una vista previa en la pestaña **Data Preview** (Vista previa de datos).

    ![Flujo de datos en el portal 12](media/lab-data-flow-data-share/dataflow12.png)

### <a name="inner-join-tripdatacsv-and-tripfaressql"></a>Combinación interna de TripDataCSV y TripFaresSQL

1. Para agregar una nueva transformación, haga clic en el icono de signo más situado en la esquina inferior derecha de "TripDataCSV". En **Multiple inputs/outputs** (Varias entradas y salidas), seleccione **Join** (Combinación).

    ![Combinación en el portal 1](media/lab-data-flow-data-share/join1.png)
1. Asigne a la transformación de combinación el nombre "InnerJoinWithTripFares". Seleccione "TripFaresSQL" en la lista desplegable del flujo derecho. Seleccione **Inner** (Interna) como tipo de combinación. Para más información sobre los diferentes tipos de combinación en el flujo de datos de asignación, consulte [Tipos de combinación](https://docs.microsoft.com/azure/data-factory/data-flow-join#join-types).

    Seleccione en cada flujo las columnas con las que desea establecer coincidencias, a través de la lista desplegable **Join conditions** (Condiciones de combinación). Para agregar otra condición de combinación, haga clic en el icono de signo más junto a una condición existente. De forma predeterminada, todas las condiciones de combinación se unen con un operador AND, lo que significa que se deben cumplir todas las condiciones para establecer una coincidencia. En este laboratorio, deseamos establecer las coincidencias de las columnas `medallion`, `hack_license`, `vendor_id` y `pickup_datetime`.

    ![Combinación en el portal 2](media/lab-data-flow-data-share/join2.png)
1. Compruebe que ha combinado correctamente 25 columnas junto con una vista previa de los datos.

    ![Combinación en el portal 3](media/lab-data-flow-data-share/join3.png)

### <a name="aggregate-by-payment_type"></a>Agregado por payment_type

1. Después de completar la transformación de combinación, agregue una transformación de agregado; para ello, haga clic en el icono de signo más situado junto a "InnerJoinWithTripFares". Elija **Aggregate** (Agregado) en **Schema modifier** (Modificador de esquema).

    ![Agregación en el portal 1](media/lab-data-flow-data-share/agg1.png)
1. Denomine "AggregateByPaymentType" a la transformación de agregado. Seleccione `payment_type` como columna de agrupación.

    ![Agregación en el portal 2](media/lab-data-flow-data-share/agg2.png)
1. Vaya a la pestaña **Agregados**. Aquí, especificará dos agregaciones:
    * La tarifa media agrupada por tipo de pago
    * La distancia total de la carrera agrupada por tipo de pago

    En primer lugar, creará la expresión para la tarifa media. En el cuadro de texto con la etiqueta **Add or select a column** (Agregar o seleccionar una columna), escriba "tarifa_media".

    ![Agregación en el portal 3](media/lab-data-flow-data-share/agg3.png)
1. Para especificar una expresión de agregación, haga clic en el cuadro azul con la etiqueta **Enter expression** (Escribir expresión). Se abrirá el generador de expresiones de flujo de datos, una herramienta que se usa para crear estas expresiones de forma visual mediante un esquema de entrada, funciones y operaciones integradas, y parámetros definidos por el usuario. Para más información sobre las funcionalidades del generador de expresiones, consulte la [documentación del generador de expresiones](https://docs.microsoft.com/azure/data-factory/concepts-data-flow-expression-builder).

    Para obtener la tarifa media, utilice la función de agregación `avg()` para agregar la columna `total_amount` convertida en un entero con `toInteger()`. En el lenguaje de expresiones de flujo de datos, esta operación se define como `avg(toInteger(total_amount))`. Haga clic en **Save and finish** (Guardar y finalizar) cuando haya terminado.

    ![Agregación en el portal 4](media/lab-data-flow-data-share/agg4.png)
1. Para agregar otra expresión de agregación, haga clic en el icono de signo más situado junto a `average_fare`. Seleccione **Add column** (Agregar columna).

    ![Agregación en el portal 5](media/lab-data-flow-data-share/agg5.png)
1. En el cuadro de texto con la etiqueta **Add or select a column** (Agregar o seleccionar una columna), escriba "total_trip_distance". Como en el último paso, abra el generador de expresiones para escribir la expresión.

    Para obtener la distancia total de la carrera, utilice la función de agregación `sum()` para agregar la columna `trip_distance` convertida en un entero con `toInteger()`. En el lenguaje de expresiones de flujo de datos, esta operación se define como `sum(toInteger(trip_distance))`. Haga clic en **Save and finish** (Guardar y finalizar) cuando haya terminado.

    ![Agregación en el portal 6](media/lab-data-flow-data-share/agg6.png)
1. Pruebe la lógica de la transformación en la pestaña **Data Preview** (Vista previa de datos). Como puede ver, hay bastantes menos filas y columnas que antes. Solo continúan las tres columnas de agrupación y agregación definidas en esta transformación. Dado que solo hay cinco grupos de tipos de pago en el ejemplo, solo se genera una salida de cinco filas.

    ![Agregación en el portal 7](media/lab-data-flow-data-share/agg7.png)

### <a name="configure-you-azure-synapse-analytics-sink"></a>Configuración del receptor de Azure Synapse Analytics

1. Ahora que hemos terminado la lógica de la transformación, estamos preparados para recibir los datos en una tabla de Azure Synapse Analytics. Agregue una transformación de receptor en la sección **Destination** (Destino).

    ![Receptor en el portal 1](media/lab-data-flow-data-share/sink1.png)
1. Asigne al receptor el nombre "SQLDWSink". Haga clic en **New** (Nuevo) junto al campo de conjunto de datos del receptor, para crear un nuevo conjunto de datos de Azure Synapse Analytics.

    ![Receptor en el portal 2](media/lab-data-flow-data-share/sink2.png)

1. Seleccione el icono **Azure Synapse Analytics (formerly SQL DW)** (Azure Synapse Analytics [anteriormente SQL DW]) y haga clic en Continue (Continuar).

    ![Receptor en el portal 3](media/lab-data-flow-data-share/sink3.png)
1. Llame al conjunto de datos "AggregatedTaxiData". Seleccione "SQLDW" como servicio vinculado. Seleccione **Create new table** (Crear nueva tabla) y asigne a la nueva tabla el nombre dbo.AggregateTaxiData. Haga clic en OK (Aceptar) cuando haya finalizado.

    ![Receptor en el portal 4](media/lab-data-flow-data-share/sink4.png)
1. Vaya a la pestaña **Settings** (Configuración) del receptor. Dado que estamos creando una nueva tabla, debemos seleccionar **Recreate table** (Volver a crear tabla) en Table action (Acción de tabla). Anule la selección de **Enable staging** (Habilitar almacenamiento provisional), que alterna si la inserción se realizará fila a fila o bien por lotes.

    ![Receptor en el portal 5](media/lab-data-flow-data-share/sink5.png)

Ha creado correctamente el flujo de datos. Ahora es el momento de ejecutarlo en una actividad de canalización.

### <a name="debug-your-pipeline-end-to-end"></a>Depuración de la canalización completa

1. Vuelva a la pestaña de la canalización **IngestAndTransformData**. Observe el cuadro verde en la actividad de copia "IngestIntoADLS". Arrástrela sobre la actividad de flujo de datos "JoinAndAggregateData". De este modo se crea una condición "en caso de éxito", que determina que la actividad de flujo de datos solo se ejecutará si la copia se realiza correctamente.

    ![Canalización en el portal 1](media/lab-data-flow-data-share/pipeline1.png)
1. Como hicimos en la actividad de copia, haga clic en **Debug** (Depurar) para llevar a cabo una ejecución de depuración. En las ejecuciones de depuración, la actividad de flujo de datos usará el clúster de depuración activo en lugar de crear un nuevo clúster. Esta canalización tardará poco más de un minuto en ejecutarse.

    ![Canalización en el portal 2](media/lab-data-flow-data-share/pipeline2.png)
1. Al igual que ocurre con la actividad de copia, el flujo de datos tiene una vista de supervisión especial a la que se accede mediante el icono con forma de gafas al finalizar la actividad.

    ![Canalización en el portal 3](media/lab-data-flow-data-share/pipeline3.png)
1. En la vista de supervisión, puede ver un gráfico de flujo de datos simplificado junto con los tiempos de ejecución y las filas de cada fase de la ejecución. Si se ejecuta correctamente, en esta actividad se deben agregar 49 999 filas en cinco filas.

    ![Canalización en el portal 4](media/lab-data-flow-data-share/pipeline4.png)
1. Puede hacer clic en una transformación para obtener detalles adicionales sobre su ejecución, como la información de creación de particiones y las columnas nuevas, actualizadas o eliminadas.

    ![Canalización en el portal 5](media/lab-data-flow-data-share/pipeline5.png)

Ya ha completado la parte de Azure Data Factory de este laboratorio. Publique los recursos si desea ponerlos en funcionamiento con desencadenadores. Ejecutó correctamente una canalización que realizó la ingesta de datos de Azure SQL Database en Azure Data Lake Storage mediante la actividad de copia y, a continuación, agregó esos datos a una instancia de Azure Synapse Analytics. Para comprobar que los datos se escribieron correctamente, examine la propia instancia de SQL Server.

## <a name="share-data-using-azure-data-share"></a>Uso compartido de datos mediante Azure Data Share

En esta sección aprenderá a configurar un nuevo recurso compartido de datos mediante Azure Portal. Esto implicará la creación de un nuevo recurso compartido de datos que contendrá conjuntos de datos de Azure Data Lake Store Gen2 y Azure Synapse Analytics (anteriormente SQL Data Warehouse). Después, configurará una programación de instantáneas, que proporcionará a los consumidores de datos una opción para actualizar automáticamente los datos que se compartan con ellos. A continuación, invitará a los destinatarios al recurso compartido de datos. 

Después de crear el recurso compartido de datos, cambiará su papel y se convertirá en el *consumidor de datos*. Como consumidor de datos, pasará por el flujo de aceptación de una invitación al recurso compartido de datos, la configuración de la ubicación donde quiere recibir los datos y la asignación de los conjuntos de datos a diferentes ubicaciones de almacenamiento. A continuación, desencadenará una instantánea que copiará los datos compartidos en el destino especificado. 

### <a name="sharing-data-data-provider-flow"></a>Uso compartido de datos (flujo del proveedor de datos)

1. Abra Azure Portal en Microsoft Edge o Google Chrome.

1. Mediante la barra de búsqueda de la parte superior de la página, busque **Recursos compartidos de datos**.

    ![Anuncios del portal](media/lab-data-flow-data-share/portal-ads.png)

1. Seleccione la cuenta de recurso compartido de datos con "Provider" en el nombre. Por ejemplo, **DataProvider0102**. 

1. Seleccione **Empezar a compartir los datos**.

    ![Inicio del uso compartido](media/lab-data-flow-data-share/ads-start-sharing.png)

1. Seleccione **+ Crear** para comenzar a configurar el nuevo recurso compartido de datos. 

1. En **Nombre del recurso compartido**, especifique un nombre de su elección. Este es el nombre del recurso compartido que verá el consumidor de datos; por tanto, asegúrese de darle un nombre descriptivo como, por ejemplo, TaxiData.

1. En **Descripción**, incluya una frase que describa el contenido del recurso compartido de datos. Este recurso contendrá datos de carreras de taxis de todo el mundo, guardados en una serie de almacenes entre los que se incluyen Azure Synapse Analytics y Azure Data Lake Store. 

1. En **Condiciones de uso**, especifique un conjunto de condiciones que le gustaría que el consumidor de datos aceptase. Algunos ejemplos pueden ser "No distribuya estos datos fuera de la organización" o "Consulte el contrato legal". 

    ![Detalles del recurso compartido](media/lab-data-flow-data-share/ads-details.png)

1. Seleccione **Continuar**. 

1. Seleccione **Agregar conjuntos de datos**. 

    ![Agregar el conjunto de datos 1](media/lab-data-flow-data-share/add-dataset.png)

1. Seleccione **Azure Synapse Analytics** (anteriormente SQL Data Warehouse) para elegir la tabla de la instancia de Azure Synapse Analytics en la que se descargaron las transformaciones de ADF.

    ![Agregar conjunto de datos de SQL](media/lab-data-flow-data-share/add-dataset-sql.png)

> [!NOTE]
> SQL Data Warehouse ahora se conoce como Azure Synapse Analytics.

1. Se le proporcionará un script para que lo ejecute antes de continuar. Este script crea un usuario en la base de datos SQL que permite que la identidad MSI de Azure Data Share se autentique en su nombre. 

> [!IMPORTANT]
> Antes de ejecutar el script, debe establecerse como administrador de Active Directory para la instancia de SQL Server. 

1. Abra una nueva pestaña y vaya a Azure Portal. Copie el script proporcionado para crear un usuario en la base de datos cuyos datos desea compartir. Para ello, inicie sesión en la base de datos de EDW mediante el Explorador de consultas (versión preliminar) con la autenticación de AAD. 

    Deberá modificar el script para que el usuario creado esté incluido entre corchetes. Por ejemplo:
    
    create user [dataprovider-xxxx] from external login;  exec sp_addrolemember db_owner, [dataprovider-xxxx];
    
1. Vuelva a la instancia de Azure Data Share donde estaba agregando los conjuntos de datos al recurso compartido de datos. 

1. Seleccione **EDW** y, a continuación, seleccione **AggregatedTaxiData** para la tabla. 

1. Seleccione **Agregar conjunto de datos**.

    Ahora contamos con una tabla SQL que forma parte de nuestro conjunto de datos. A continuación, agregaremos otros conjuntos de datos desde Azure Data Lake Store. 

1. Seleccione **Agregar conjunto de datos** y elija **Azure Data Lake Store Gen2**.

    ![Agregar conjunto de datos de ADLS](media/lab-data-flow-data-share/add-dataset-adls.png)

1. Seleccione **Siguiente**.

1. Expanda *wwtaxidata*. Expanda *Boston Taxi Data*. Tenga en cuenta que puede compartir hasta el nivel de archivo. 

1. Seleccione la carpeta *Boston Taxi Data* para agregarla completa a al recurso compartido de datos. 

1. Seleccione **Agregar conjuntos de datos**.

1. Revise los conjuntos de datos que se han agregado. Se deberían haber agregado una tabla SQL y una carpeta de ADLS Gen2 al recurso compartido de datos. 

1. Seleccione **Continuar**

1. En esta pantalla, puede agregar destinatarios al recurso compartido de datos. Los destinatarios que agregue recibirán invitaciones a ese recurso. Para este laboratorio, debe agregar 2 direcciones de correo electrónico:

    1. La dirección de correo electrónico de la suscripción de Azure en la que se encuentra. 

        ![Adición de destinatarios](media/lab-data-flow-data-share/add-recipients.png)

    1. Agregue el consumidor de datos ficticio denominado *janedoe@fabrikam.com* .

1. En esta pantalla, puede realizar una configuración de instantánea para el consumidor de datos. Esto les permitirá recibir actualizaciones periódicas de los datos según el intervalo que defina. 

1. Marque **Programación de instantáneas** y configure una actualización cada hora de los datos mediante el menú desplegable *Periodicidad*.  

1. Seleccione **Crear**.

    Ahora tiene un recurso compartido de datos activo. Revisemos qué puede ver como proveedor de datos cuando crea un recurso compartido de datos. 

1. Seleccione el recurso compartido de datos que ha creado, de nombre **TaxiData**. Puede navegar hasta él seleccionando **Recursos compartidos enviados** en **Recurso compartido de datos**. 

1. Haga clic en Programación de instantáneas. Puede deshabilitar la programación de instantáneas, si lo desea. 

1. A continuación, seleccione la pestaña **Conjuntos de datos**. Puede agregar otros conjuntos de datos a este recurso compartido de datos una vez creado. 

1. Seleccione la pestaña **Suscripciones a recursos compartidos**. Todavía no existen suscripciones a recursos compartidos porque el consumidor de datos aún no ha aceptado la invitación.

1. Vaya a la pestaña **Invitaciones**. Aquí verá una lista de las invitaciones pendientes. 

    ![Invitaciones pendientes](media/lab-data-flow-data-share/pending-invites.png)

1. Seleccione la invitación a *janedoe@fabrikam.com* . Seleccione Eliminar. Si el destinatario todavía no ha aceptado la invitación, ya no podrá hacerlo. 

1. Seleccione la pestaña **Historial** . Aún no se muestra nada, porque el consumidor de datos todavía no ha aceptado la invitación ni desencadenado una instantánea. 

### <a name="receiving-data-data-consumer-flow"></a>Recepción de datos (flujo de consumidor de datos)

Ahora que hemos revisado el recurso compartido de datos, estamos preparados para cambiar de contexto y pasar a nuestro papel como consumidor de datos. 

En estos momentos debería tener una invitación de Microsoft Azure para Azure Data Share en la bandeja de entrada. Inicie Outlook Web Access (outlook.com) e inicie sesión con las credenciales proporcionadas para la suscripción de Azure.

En el correo electrónico que debe haber recibido, haga clic en "Ver invitación >". Ahora va a simular la experiencia del consumidor de datos cuando acepta una invitación del proveedor de datos para el recurso compartido de datos. 

![Invitación por correo electrónico](media/lab-data-flow-data-share/email-invite.png)

Es posible que se le pida que seleccione una suscripción. Asegúrese de seleccionar la suscripción en la que ha estado trabajando para este laboratorio. 

1. Haga clic en la invitación titulada *TaxiData*. 

1. En esta pantalla de invitación, observará varios detalles sobre el recurso compartido de datos que configuró anteriormente como proveedor de datos. Revise los detalles y acepte las condiciones de uso, si se indicaron.

1. Seleccione la suscripción y el grupo de recursos que ya existen para el laboratorio. 

1. En **Data share account** (Cuenta de Azure Data Share), seleccione **DataConsumer**. También puede crear una nueva cuenta de Azure Data Share. 

1. Junto a **Received share name** (Nombre de recurso compartido recibido), observará que el nombre del recurso compartido predeterminado es el nombre especificado por el proveedor de datos. Asigne al recurso compartido un nombre descriptivo para los datos que va a recibir, por ejemplo **TaxiDataShare**.

    ![Aceptación de invitación](media/lab-data-flow-data-share/consumer-accept.png)

1. Puede optar por elegir la opción **Accept and configure now** (Aceptar y configurar ahora) o bien **Accept and configure later** (Aceptar y configurar más adelante). Si decide aceptar y configurar ahora, especificará una cuenta de almacenamiento en la que se deberán copiar todos los datos. Si decide aceptar y configurar más adelante, no se asignarán los conjuntos de datos del recurso compartido y deberá asignarlos manualmente. Elegiremos la opción para más adelante. 

1. Seleccione **Accept and configure later** (Aceptar y configurar más adelante). 

    Al configurar esta opción, se crea una suscripción al recurso compartido, pero no hay ninguna ubicación en la que recibir los datos, ya que no se ha asignado ningún destino. 

    A continuación, configuraremos las asignaciones de conjunto de datos para el recurso compartido de datos. 

1. Seleccione el recurso compartido recibido (el nombre que especificó en el paso 5).

    La opción **Desencadenar instantánea** está atenuada, pero el recurso compartido está activo. 

1. Seleccione la pestaña **Conjuntos de datos**. Tenga en cuenta que cada conjunto de datos tiene el estado No asignado, lo que significa que no tiene ningún destino en el que copiar los datos. 

    ![Conjuntos de datos no asignados](media/lab-data-flow-data-share/unmapped.png)

1. Seleccione la tabla de Azure Synapse Analytics y, a continuación, seleccione **+ Asignar a destino**.

1. En el lado derecho de la pantalla, seleccione la lista desplegable **Tipo de datos de destino**. 

    Puede asignar los datos de SQL a una amplia gama de almacenes de datos. En este caso, los asignaremos a una instancia de Azure SQL Database.

    ![mapping](media/lab-data-flow-data-share/mapping-options.png)
    
    (Opcional) Seleccione **Azure Data Lake Store Gen2** como tipo de datos de destino. 
    
    (Opcional) Seleccione la suscripción, el grupo de recursos y la cuenta de almacenamiento con los que ha estado trabajando. 
    
    (Opcional) Puede elegir si recibe los datos en el lago de datos en formato CSV o Parquet. 

1. Junto a **Tipo de datos de destino**, seleccione Azure SQL Database. 

1. Seleccione la suscripción, el grupo de recursos y la cuenta de almacenamiento con los que ha estado trabajando. 

    ![Asignación a SQL](media/lab-data-flow-data-share/map-to-sqldb.png)

1. Para poder continuar, debe crear un nuevo usuario en la instancia de SQL Server mediante la ejecución del script proporcionado. En primer lugar, copie el script proporcionado en el portapapeles. 

1. Abra una nueva pestaña de Azure Portal. No cierre la pestaña existente, ya que tendrá que volver a ella en un momento. 

1. En la nueva pestaña que ha abierto, vaya a **SQL Database**.

1. Seleccione la base de datos SQL (solo debe haber una en la suscripción). Tenga cuidado de no seleccionar el almacenamiento de datos. 

1. Seleccione **Editor de consultas (versión preliminar)** .

1. Use la autenticación de AAD para iniciar sesión en el editor de consultas. 

1. Ejecute la consulta proporcionada en el recurso compartido de datos (que copió en el portapapeles en el paso 14). 

    Este comando permite al servicio Azure Data Share usar identidades administradas para que los servicios de Azure se autentiquen en la instancia de SQL Server y poder copiar datos en ella. 

1. Vuelva a la pestaña original y seleccione **Asignar a destino**.

1. A continuación, seleccione la carpeta Azure Data Lake Gen2 que forma parte del conjunto de datos y asígnela a una cuenta de Azure Blob Storage. 

    ![storage](media/lab-data-flow-data-share/storage-map.png)

    Con todos los conjuntos de datos asignados, ya está listo para empezar a recibir datos del proveedor de datos. 

    ![Asignado](media/lab-data-flow-data-share/all-mapped.png)
    
1. Seleccione **Detalles**. 

    Observe que la opción **Desencadenar instantánea** ya no está atenuada, puesto que ahora el recurso compartido de datos tiene destinos en los que copiar.

1. Seleccione Desencadenar instantánea -> Copia completa. 

    ![desencadenador](media/lab-data-flow-data-share/trigger-full.png)

    Se iniciará la copia de datos en la nueva cuenta de Azure Data Share. En un escenario real, estos datos procederían de un tercero. 

    Los datos tardarán entre 3 y 5 minutos en aparecer. Para supervisar el progreso, haga clic en la pestaña **Historial**. 

    Mientras espera, vaya hasta el recurso compartido de datos original (TaxiData) y vea el estado de **Suscripciones a recursos compartidos** y la pestaña **Historial**. Observe que ahora hay una suscripción activa; como proveedor de datos, también puede supervisar cuándo el consumidor de datos ha empezado a recibir los datos compartidos con él. 

1. Vuelva al recurso compartido de datos del consumidor de datos. Una vez que el estado del desencadenador sea correcto, vaya a la base de datos SQL de destino y al lago de datos para comprobar que los datos se han descargado en los almacenes correspondientes. 

¡Enhorabuena, ha completado el laboratorio!


