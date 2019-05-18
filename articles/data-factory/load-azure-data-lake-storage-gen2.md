---
title: Carga de datos en Azure Data Lake Storage Gen2 con Azure Data Factory
description: Uso de Azure Data Factory para copiar datos en Azure Data Lake Storage Gen2
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 05/13/2019
ms.author: jingwang
ms.openlocfilehash: f8af34207eddb613f7a59bd3e3d300555e10f985
ms.sourcegitcommit: 179918af242d52664d3274370c6fdaec6c783eb6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65560729"
---
# <a name="load-data-into-azure-data-lake-storage-gen2-with-azure-data-factory"></a>Carga de datos en Azure Data Lake Storage Gen2 con Azure Data Factory

‎Azure Data Lake Storage Gen2 es un conjunto de funcionalidades dedicadas al análisis de macrodatos basado en [Azure Blob Storage](../storage/blobs/storage-blobs-introduction.md). Permite establecer una conexión con los datos usando tanto el sistema de archivos como el almacenamiento de objetos.

Azure Data Factory (ADF) es un servicio de integración de datos en la nube totalmente administrado. Puede utilizar el servicio para rellenar el lago con datos de un amplio conjunto de almacenes de datos locales y basados en la nube y ahorrar tiempo al crear las soluciones de análisis. Para una lista detallada de conectores admitidos, consulte la tabla de [Almacenes de datos admitidos](copy-activity-overview.md#supported-data-stores-and-formats).

Azure Data Factory ofrece una solución de movimiento de datos administrados y de escalabilidad horizontal. Debido a la arquitectura con escalabilidad horizontal de ADF, puede ingerir datos con un alto rendimiento. Para más información, consulte el [rendimiento de la actividad de copia](copy-activity-performance.md).

En este artículo se muestra cómo utilizar la herramienta Copiar datos de Data Factory para cargar datos del _servicio Amazon Web Services S3_ en _Azure Data Lake Storage Gen2_. Puede seguir los mismos pasos para copiar datos de otros tipos de almacenes de datos.

>[!TIP]
>Para copiar datos desde Azure Data Lake Storage Gen1 en Gen2, consulte [en este tutorial específico](load-azure-data-lake-storage-gen2-from-gen1.md).

## <a name="prerequisites"></a>Requisitos previos

* Suscripción de Azure: Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Cuenta de Azure Storage con Data Lake Storage Gen2 habilitado: Si no tienes una cuenta de almacenamiento, [crear una cuenta](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM).
* Cuenta de AWS con un cubo de S3 que contiene datos: En este artículo se muestra cómo copiar datos de Amazon S3. Puede usar otros almacenes de datos siguiendo los mismos pasos.

## <a name="create-a-data-factory"></a>Crear una factoría de datos

1. En el menú de la izquierda, seleccione **Crear un recurso** > **Datos y análisis** > **Data Factory**:
   
   ![Selección de la factoría de datos en el panel Nuevo](./media/quickstart-create-data-factory-portal/new-azure-data-factory-menu.png)

2. En la página **Nueva factoría de datos**, proporcione los valores de los campos que se muestran en la imagen siguiente: 
      
   ![Página New data factory (Nueva factoría de datos)](./media/load-azure-data-lake-storage-gen2//new-azure-data-factory.png)
 
    * **Nombre**: escriba un nombre único global para la factoría de datos de Azure. Si recibe el error "El nombre de la factoría de datos \"LoadADLSDemo\" no está disponible", escriba uno diferente. Por ejemplo, podría utilizar el nombre _**suNombre**_**ADFTutorialDataFactory**. Intente crear de nuevo la factoría de datos. Para conocer las reglas de nomenclatura de los artefactos de Data Factory, consulte [Data Factory: reglas de nomenclatura](naming-rules.md).
    * **Suscripción**: seleccione la suscripción de Azure donde desea crear la factoría de datos. 
    * **Grupo de recursos**: seleccione un grupo de recursos existente en la lista desplegable o seleccione la opción **Crear nuevo** y escriba el nombre de un grupo de recursos. Para obtener más información sobre los grupos de recursos, consulte [Uso de grupos de recursos para administrar los recursos de Azure](../azure-resource-manager/resource-group-overview.md).  
    * **Versión**: seleccione **V2**.
    * **Ubicación**: Seleccione la ubicación de la factoría de datos. Solo las ubicaciones admitidas se muestran en la lista desplegable. Los almacenes de datos que las factorías de datos usan pueden estar en otras ubicaciones y regiones. 

3. Seleccione **Crear**.
4. Una vez completada la creación, vaya a la factoría de datos. Verá la página principal de **Factoría de datos**, tal y como se muestra en la siguiente imagen: 
   
   ![Página principal Factoría de datos](./media/load-azure-data-lake-storage-gen2/data-factory-home-page.png)

   Seleccione el icono **Author & Monitor** (Creación y supervisión) para iniciar la aplicación de integración de datos en una pestaña independiente.

## <a name="load-data-into-azure-data-lake-storage-gen2"></a>Carga de datos en Azure Data Lake Storage Gen2

1. En la página **Get started** (Introducción), seleccione el icono **Copy Data** (Copiar datos) para iniciar la herramienta para copiar datos: 

   ![Icono de la herramienta Copy Data](./media/load-azure-data-lake-storage-gen2/copy-data-tool-tile.png)
2. En la página **Properties** (Propiedades), especifique **CopyFromAmazonS3ToADLS** en el campo **Task name** (Nombre de la tarea) y seleccione **Next** (Siguiente):

    ![Página de propiedades](./media/load-azure-data-lake-storage-gen2/copy-data-tool-properties-page.png)
3. En la página **Almacén de datos de origen**, haga clic en **+ Crear nueva conexión**:

    ![Página Source data store (Almacén de datos de origen)](./media/load-azure-data-lake-storage-gen2/source-data-store-page.png)
    
    Seleccione **Amazon S3** en la galería de conectores y seleccione **Continue** (Continuar)
    
    ![Página del almacén de datos de origen S3](./media/load-azure-data-lake-storage-gen2/source-data-store-page-s3.png)
    
4. En la página **Specify Amazon S3 connection** (Especificar conexión de Amazon S3), siga estos pasos:

   1. Especifique el valor de **Access Key ID** (Identificador de clave de acceso).
   2. Especifique el valor de **Secret Access Key** (Clave de acceso secreta).
   3. Haga clic en **Test connection** (Prueba de conexión) para validar la configuración y, después, seleccione **Finish** (Finalizar).
   4. Podrá ver una nueva conexión creada. Seleccione **Next** (Siguiente).
   
      ![Especificación de la cuenta de Amazon S3](./media/load-azure-data-lake-storage-gen2/specify-amazon-s3-account.png)
      
5. En la página **Choose the input file or folder** (Elegir archivo o carpeta de entrada), vaya a la carpeta y el archivo que desea copiar. Seleccione la carpeta o el archivo y seleccione **Choose** (Elegir):

    ![Elegir archivo o carpeta de entrada](./media/load-azure-data-lake-storage-gen2/choose-input-folder.png)

6. Elija el comportamiento de copia seleccionando las opciones **Copy files recursively** (Copiar archivos de forma recursiva) y **Binary copy** (Copia binaria). Seleccione **Next** (Siguiente):

    ![Especificación de carpeta de salida](./media/load-azure-data-lake-storage-gen2/specify-binary-copy.png)
    
7. En la página **Almacén de datos de destino**, haga clic en **+Crear nueva conexión** y después seleccione **Azure Data Lake Storage Gen2** y **Continuar**:

    ![Página Destination data store (Almacén de datos de destino)](./media/load-azure-data-lake-storage-gen2/destination-data-storage-page.png)

8. En la página **Specify Azure Data Lake Storage connection** (Especificar conexión de Azure Data Lake Storage), siga estos pasos:

   1. Seleccione la cuenta habilitada para Data Lake Storage Gen2 en la lista desplegable "Storage account name" (Nombre de la cuenta de almacenamiento).
   2. Seleccione **Finalizar** para crear la conexión. Luego, seleccione **Siguiente**.
   
   ![Especificación de una cuenta de Azure Data Lake Storage Gen2](./media/load-azure-data-lake-storage-gen2/specify-adls.png)

9. En el **elegir el archivo de salida o la carpeta** , escriba **copyfroms3** como el nombre de la carpeta de salida y seleccione **siguiente**. ADF creará el sistema de archivos ADLS Gen2 correspondiente y las subcarpetas durante la copia, si no existe.

    ![Especificación de carpeta de salida](./media/load-azure-data-lake-storage-gen2/specify-adls-path.png)

10. En la página **Settings** (Configuración), seleccione **Next** (Siguiente) para usar las configuraciones predeterminadas:

    ![Página Configuración](./media/load-azure-data-lake-storage-gen2/copy-settings.png)
11. En la página **Summary** (Resumen), revise la configuración y seleccione **Next** (Siguiente):

    ![Página de resumen](./media/load-azure-data-lake-storage-gen2/copy-summary.png)
12. En la página **Deployment** (Implementación), seleccione **Monitor** (Supervisión) para supervisar la canalización:

    ![Página Deployment (Implementación)](./media/load-azure-data-lake-storage-gen2/deployment-page.png)
13. Observe que la pestaña **Monitor** (Supervisión) de la izquierda se selecciona automáticamente. La columna **Actions** (Acciones) incluye los vínculos para ver los detalles de la ejecución de actividad y volver a ejecutar la canalización:

    ![La supervisión de la canalización se ejecuta](./media/load-azure-data-lake-storage-gen2/monitor-pipeline-runs.png)

14. Para ver las ejecuciones de actividad asociadas con la ejecución de canalización, seleccione el vínculo **View Activity Runs** (Ver ejecuciones de actividad) en la columna **Actions** (Acciones). Como solo hay una actividad (actividad de copia) en la canalización, solo verá una entrada. Para volver a la vista de ejecuciones de canalización, seleccione el vínculo **Pipelines** (Canalizaciones) de la parte superior. Seleccione **Refresh** (Actualizar) para actualizar la lista. 

    ![Supervisión de las ejecuciones de actividad](./media/load-azure-data-lake-storage-gen2/monitor-activity-runs.png)

15. Para supervisar los detalles de la ejecución de cada actividad de copia, seleccione el vínculo con un icono de gafas **Details** (Detalles) en **Actions** (Acciones) en la vista de supervisión de la actividad. Puede supervisar detalles como el volumen de datos copiados desde el origen al receptor, el rendimiento de los datos, los pasos de ejecución con su duración correspondiente y las configuraciones que se utilizan:

    ![Detalles de la supervisión de la ejecución de actividad](./media/load-azure-data-lake-storage-gen2/monitor-activity-run-details.png)

16. Verifique que los datos se copian en la cuenta de Data Lake Storage Gen2.

## <a name="next-steps"></a>Pasos siguientes

* [Información general de la actividad de copia](copy-activity-overview.md)
* [Conector de Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md)
