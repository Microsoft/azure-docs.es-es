---
title: 'Transmisión de datos de Stream Analytics a Azure Data Lake Storage Gen1: Azure'
description: Obtenga información sobre cómo usar Azure Data Lake Storage Gen1 como salida para un trabajo de Azure Stream Analytics, con un escenario simple que lee los datos de un blob de Azure Storage.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/30/2018
ms.author: twooley
ms.openlocfilehash: 8ef3d7400b64617df8d81c7b84e680496776b5bd
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88192156"
---
# <a name="stream-data-from-azure-storage-blob-into-azure-data-lake-storage-gen1-using-azure-stream-analytics"></a>Transmisión de datos de Azure Storage Blob a Azure Data Lake Storage Gen1 mediante Stream Analytics
En este artículo aprenderá a utilizar Azure Data Lake Storage Gen1 como salida para un trabajo de Azure Stream Analytics. En este artículo se muestra un escenario simple en el que se leen datos desde un blob de Azure Storage (entrada) y se escriben en Data Lake Storage Gen1 (salida).

## <a name="prerequisites"></a>Prerrequisitos
Antes de empezar este tutorial, debe contar con lo siguiente:

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).

* **Cuenta de Azure Storage**. Usará un contenedor de blobs desde esta cuenta para introducir datos en un trabajo de Stream Analytics. En este tutorial, se asume que tiene una cuenta de almacenamiento denominada **storageforasa** y un contenedor dentro de la cuenta denominado **storageforasacontainer**. Una vez creado el contenedor, va a cargar un archivo de datos de ejemplo en él. 
  
* **Cuenta de Data Lake Storage Gen1**. Siga las instrucciones de [Introducción a Azure Data Lake Storage Gen1 con Azure Portal](data-lake-store-get-started-portal.md). Supongamos que tiene una cuenta de Data Lake Storage Gen1 llamada **myadlsg1**. 

## <a name="create-a-stream-analytics-job"></a>Creación de un trabajo de Stream Analytics
Primero debe crear un trabajo de Stream Analytics que incluya un origen de entrada y un destino de salida. Para este tutorial, el origen es un contenedor de blobs de Azure y el destino es Data Lake Storage Gen1.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. En el panel izquierdo, haga clic en **Trabajos de Stream Analytics** y luego haga clic en **Agregar**.

    ![Creación de un trabajo de Stream Analytics](./media/data-lake-store-stream-analytics/create.job.png "Creación de un trabajo de Stream Analytics")

    > [!NOTE]
    > Asegúrese de crear el trabajo en la misma región que la cuenta de almacenamiento o incurrirá en costos adicionales derivados de mover datos entre regiones.
    >

## <a name="create-a-blob-input-for-the-job"></a>Creación de una entrada de blob para el trabajo

1. Abra la página del trabajo de Stream Analytics y en el panel izquierdo haga clic en la pestaña **Inputs** (Entradas) y luego haga clic en **Agregar**.

    ![Incorporación de una entrada al trabajo](./media/data-lake-store-stream-analytics/create.input.1.png "Agregar una entrada al trabajo")

2. En la hoja **Nueva entrada**, proporcione los valores siguientes.

    ![Incorporación de una entrada al trabajo](./media/data-lake-store-stream-analytics/create.input.2.png "Agregar una entrada al trabajo")

   * En **Alias de entrada**, escriba un nombre único para la entrada del trabajo.
   * En **Source type** (Tipo de origen), seleccione **Flujo de datos**.
   * En **Origen**, seleccione **Blob storage**.
   * En **Suscripción**, seleccione **Usar almacenamiento de blobs de la suscripción actual**.
   * En **Cuenta de almacenamiento**, seleccione la cuenta de almacenamiento que creó como parte de los requisitos previos. 
   * En **Contenedor**, seleccione el contenedor que creó en la cuenta de almacenamiento seleccionada.
   * En **Formato de serialización de eventos**, seleccione **CSV**.
   * En **Delimitador**, seleccione **tabulación**.
   * En **Codificación**, seleccione **UTF-8**.

     Haga clic en **Crear**. El portal ahora agrega la entrada y prueba la conexión a ella.


## <a name="create-a-data-lake-storage-gen1-output-for-the-job"></a>Creación de una salida de Data Lake Storage Gen1 para el trabajo

1. Abra la página del trabajo de Stream Analytics, haga clic en la pestaña **Salidas** y en **Agregar** y seleccione **Data Lake Storage Gen1**.

    ![Incorporación de una salida al trabajo](./media/data-lake-store-stream-analytics/create.output.1.png "Agregar una salida al trabajo")

2. En la hoja **Nueva salida**, proporcione los siguientes valores.

    ![Incorporación de una salida al trabajo](./media/data-lake-store-stream-analytics/create.output.2.png "Agregar una salida al trabajo")

    * En **Alias de salida**, escriba un nombre único para la salida del trabajo. Se trata de un nombre descriptivo utilizado en las consultas para dirigir la salida de la consulta a esta cuenta de Data Lake Storage Gen1.
    * Se le pedirá que autorice el acceso a la cuenta de Data Lake Storage Gen1. Haga clic en **Autorizar**.

3. En la hoja **Nueva salida**, proporcione los siguientes valores.

    ![Incorporación de una salida al trabajo](./media/data-lake-store-stream-analytics/create.output.3.png "Agregar una salida al trabajo")

   * En **Nombre de cuenta**, seleccione la cuenta de Data Lake Storage Gen1 que ya ha creado como ubicación a la que quiere que se envíe la salida del trabajo.
   * En **Patrón de prefijo de la ruta**, escriba una ruta de archivo usada para escribir los archivos en la cuenta de Data Lake Storage Gen1 especificada.
   * En **Date format** (Formato de fecha), si usó un token de fecha en la ruta del prefijo, puede seleccionar el formato de fecha en el que se organizan sus archivos.
   * En **Time format** (Formato de hora), si usó un token de hora en la ruta del prefijo, especifique el formato de hora en el que se organizan sus archivos.
   * En **Formato de serialización de eventos**, seleccione **CSV**.
   * En **Delimitador**, seleccione **tabulación**.
   * En **Codificación**, seleccione **UTF-8**.
    
     Haga clic en **Crear**. El portal ahora agrega la salida y prueba la conexión a ella.
    
## <a name="run-the-stream-analytics-job"></a>Ejecución del trabajo de Stream Analytics

1. Para ejecutar un trabajo de Stream Analytics, debe ejecutar una consulta desde la pestaña **Consulta**. En este tutorial, puede ejecutar la consulta de ejemplo mediante el reemplazo de los marcadores de posición con los alias de entrada y salida del trabajo, tal como se muestra en la captura de pantalla siguiente.

    ![Ejecución de una consulta](./media/data-lake-store-stream-analytics/run.query.png "Ejecutar consulta")

2. Haga clic en **Guardar** en la parte superior de la pantalla y, a continuación, en **Overview** (Introducción), haga clic en **Iniciar**. En el cuadro de diálogo, seleccione **Hora personalizada** y, después, seleccione la fecha y la hora actuales.

    ![Establecimiento del tiempo de trabajo](./media/data-lake-store-stream-analytics/run.query.2.png "Establecer el tiempo de trabajo")

    Haga clic en **Iniciar** para iniciar el trabajo. Puede tardar unos minutos en iniciarse el trabajo.

3. Para desencadenar el trabajo que selecciona los datos del blob, copie un archivo de datos de ejemplo en el contenedor de blobs. Puede obtener un archivo de datos de ejemplo en el [repositorio Git de Azure Data Lake](https://github.com/Azure/usql/tree/master/Examples/Samples/Data/AmbulanceData/Drivers.txt). En este tutorial, vamos a copiar el archivo **vehicle1_09142014.csv**. Puede utilizar varios clientes, como el [explorador de Azure Storage](https://storageexplorer.com/), para cargar datos en un contenedor de blobs.

4. En la pestaña **Overview** (Introducción), en **Monitoring** (Supervisión), consulte cómo se han procesado los datos.

    ![Supervisión del trabajo](./media/data-lake-store-stream-analytics/run.query.3.png "Supervisar el trabajo")

5. Finalmente, puede verificar que los datos de salida del trabajo estén disponibles en la cuenta de Data Lake Storage Gen1. 

    ![Comprobación de la salida](./media/data-lake-store-stream-analytics/run.query.4.png "Comprobar salida")

    En el panel del Explorador de datos, tenga en cuenta que la salida se escribe en una ruta de carpeta, según lo especificado en la configuración de salida de Data Lake Storage Gen1 (`streamanalytics/job/output/{date}/{time}`).  

## <a name="see-also"></a>Consulte también
* [Creación de un clúster de HDInsight para usar Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md)
