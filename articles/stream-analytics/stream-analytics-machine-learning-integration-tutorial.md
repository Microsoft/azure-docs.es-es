---
title: Integración de Azure Stream Analytics con Azure Machine Learning
description: En este artículo se explica cómo configurar rápidamente un trabajo sencillo de Azure Stream Analytics que integre Azure Machine Learning mediante una función definida por el usuario.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 08/12/2020
ms.custom: seodec18
ms.openlocfilehash: 26a1208131f1d9d3df7dccd8e27bda37992f043f
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88236687"
---
# <a name="do-sentiment-analysis-with-azure-stream-analytics-and-azure-machine-learning-studio-classic"></a>Realización de análisis de opinión con Azure Stream Analytics y Azure Machine Learning Studio (clásico)

En este artículo se muestra cómo configurar un trabajo sencillo de Azure Stream Analytics que usa Azure Machine Learning Studio (clásico) para el análisis de opinión. Un modelo de análisis de opinión de Machine Learning de la galería de Cortana Intelligence se usa para analizar datos de texto que se están transmitiendo y determinar la puntuación de opiniones.

> [!TIP]
> Es muy recomendable usar las [UDF de Azure Machine Learning](machine-learning-udf.md) en lugar de las de Azure Machine Learning Studio (clásico) para mejorar el rendimiento y la confiabilidad.

Puede aplicar lo que aprenda en este artículo a escenarios como estos:

* Análisis de opiniones en tiempo real en datos de Twitter que se están transmitiendo.
* Análisis de registros de chats de clientes con el personal de soporte técnico.
* Evaluación de comentarios en foros, blogs y vídeos.
* Muchos otros escenarios de puntuación predictiva en tiempo real.

El trabajo de Streaming Analytics que cree aplicará el modelo de análisis de opiniones como una función definida por el usuario (UDF) a los datos de texto de ejemplo del almacén de blobs. La salida (el resultado del análisis de opiniones) se escribe en el mismo almacén de blobs en otro archivo CSV. 

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar, asegúrese de que dispone de lo siguiente:

* Una suscripción de Azure activa.

* Un archivo CSV con algunos datos de Twitter. Puede descargar un archivo de ejemplo de [GitHub](https://github.com/Azure/azure-stream-analytics/blob/master/Sample%20Data/sampleinput.csv), o puede crear su propio archivo. En un escenario real, los datos se obtendrían directamente de un flujo de datos de Twitter.

## <a name="create-a-storage-container-and-upload-the-csv-input-file"></a>Creación de un contenedor de almacenamiento y carga del archivo de entrada CSV

En este paso, cargará un archivo CSV en el contenedor de almacenamiento.

1. En Azure Portal, seleccione **Crear un recurso** > **Almacenamiento** > **Cuenta de almacenamiento**.

2. Rellene la pestaña *Aspectos básicos* con los detalles siguientes y deje los valores predeterminados en el resto de campos:

   |Campo  |Value  |
   |---------|---------|
   |Subscription|Elija su suscripción.|
   |Resource group|Elija el grupo de recursos.|
   |El nombre de la cuenta de almacenamiento|Escriba un nombre para la cuenta de almacenamiento. El nombre debe ser único en Azure.|
   |Location|Elija una ubicación. Todos los recursos deben usar la misma ubicación.|
   |Tipo de cuenta|BlobStorage|

   ![especificación de los detalles de la cuenta de almacenamiento](./media/stream-analytics-machine-learning-integration-tutorial/create-storage-account1.png)

3. Seleccione **Revisar + crear**. A continuación, seleccione **Crear** para implementar la cuenta de almacenamiento.

4. Una vez finalizada la implementación, vaya a la cuenta de almacenamiento. En **Blob service**, seleccione **Contenedores**. Luego, seleccione **+ Contenedor** para crear un contenedor.

   ![Creación de un contenedor de almacenamiento de blobs para entrada](./media/stream-analytics-machine-learning-integration-tutorial/create-storage-account2.png)

5. Proporcione un nombre para el contenedor y compruebe que **Nivel de acceso público** está establecido en **Privado**. Seleccione **Crear** cuando haya terminado.

   ![especificación de los detalles del contenedor de blobs](./media/stream-analytics-machine-learning-integration-tutorial/create-storage-account3.png)

6. Vaya al contenedor recién creado y seleccione **Cargar**. Cargue el archivo **sampleinput.csv** que descargó anteriormente.

   ![botón "Cargar" de un contenedor](./media/stream-analytics-machine-learning-integration-tutorial/create-sa-upload-button.png)

## <a name="add-the-sentiment-analytics-model-from-the-cortana-intelligence-gallery"></a>Adición del modelo de análisis de opinión desde la galería de Cortana Intelligence

Ahora que los datos de ejemplo están en un blob, puede habilitar el modelo de análisis de opiniones de la galería de Cortana Intelligence.

1. Vaya a la página de [modelo predictivo de análisis de opiniones](https://gallery.cortanaintelligence.com/Experiment/Predictive-Mini-Twitter-sentiment-analysis-Experiment-1) de la galería de Cortana Intelligence.  

2. Seleccione **Abrir en Studio (clásico)** .  
   
   ![Machine Learning de Análisis de transmisiones, abrir Machine Learning](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-open-ml-studio.png)  

3. Inicie sesión para dirigirse al área de trabajo. Seleccione una ubicación.

4. Seleccione **Ejecutar** en la parte inferior de la página. El proceso se ejecuta, lo que lleva aproximadamente un minuto.

   ![ejecución del experimento en Machine Learning Studio](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-run-experiment.png)  

5. Una vez que el proceso se ha ejecutado correctamente, seleccione **Implementar servicio web** en la parte inferior de la página.

   ![implementación del experimento como un servicio web en Machine Learning Studio](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-deploy-web-service.png)  

6. Para validar que el modelo de análisis de opinión está listo para su uso, seleccione el botón **Probar**. Proporcione algún texto de entrada, como "Me encanta Microsoft".

   ![prueba del experimento en Machine Learning Studio](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-test.png)  

   Si la prueba funciona, verá un resultado similar al ejemplo siguiente:

   ![resultados de la prueba en Machine Learning Studio](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-test-results.png)  

7. En la columna **Aplicaciones**, seleccione el vínculo **Libro de Excel 2010 o anterior** para descargar un libro de Excel. El libro contiene una clave de API y la dirección URL que se necesitan más adelante para configurar el trabajo de Stream Analytics.

    ![Machine Learning de Stream Analytics, vista rápida](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-quick-glance.png)  

## <a name="create-a-stream-analytics-job-that-uses-the-machine-learning-model"></a>Creación de un trabajo de Análisis de transmisiones que usa el modelo de Machine Learning

Ahora puede crear un trabajo de Stream Analytics que lea los tweets de ejemplo del archivo CSV de almacenamiento de blobs.

### <a name="create-the-job"></a>Creación del trabajo

Vaya a [Azure Portal](https://portal.azure.com) y cree un trabajo de Stream Analytics. Si no está familiarizado con este proceso, consulte [Creación de un trabajo de Stream Analytics mediante Azure Portal](stream-analytics-quick-create-portal.md).

### <a name="configure-the-job-input"></a>Configuración de la entrada de trabajo

El trabajo obtiene su entrada del archivo CSV cargado anteriormente en el almacenamiento de blobs.

1. Vaya a su trabajo de Stream Analytics. En **Topología de trabajo**, seleccione la opción **Entradas**. Seleccione **Agregar entrada de flujo** >**Blob Storage**.

2. Rellene la página **Blob Storage** con los siguientes valores:

   |Campo  |Value  |
   |---------|---------|
   |Alias de entrada|Asigne un nombre a la entrada. Recuerde este alias cuando escriba la consulta.|
   |Subscription|Seleccione su suscripción.|
   |Cuenta de almacenamiento|Seleccione la cuenta de almacenamiento que creó en el paso anterior.|
   |Contenedor|Seleccione el contenedor que creó en el paso anterior.|
   |Formato de serialización de eventos|CSV|

3. Seleccione **Guardar**.

### <a name="configure-the-job-output"></a>Configuración de la salida del trabajo

El trabajo envía los resultados al mismo almacenamiento de blobs del que obtiene la entrada.

1. Vaya a su trabajo de Stream Analytics. En **Topología de trabajo**, seleccione la opción **Salidas**. Seleccione **Agregar** > **Blob Storage**.

2. Rellene el formulario de **Blob Storage** con estos valores:

   |Campo  |Value  |
   |---------|---------|
   |Alias de entrada|Asigne un nombre a la entrada. Recuerde este alias cuando escriba la consulta.|
   |Subscription|Seleccione su suscripción.|
   |Cuenta de almacenamiento|Seleccione la cuenta de almacenamiento que creó en el paso anterior.|
   |Contenedor|Seleccione el contenedor que creó en el paso anterior.|
   |Formato de serialización de eventos|CSV|

3. Seleccione **Guardar**.

### <a name="add-the-machine-learning-function"></a>Incorporación de la función de Machine Learning

Arriba ha publicado un modelo de Machine Learning en un servicio web. En este escenario, cuando se ejecuta el trabajo de Stream Analysis, envía cada tweet de ejemplo de la entrada al servicio web para el análisis de opiniones. El servicio web de Machine Learning devuelve una opinión (`positive`, `neutral` o `negative`) y una probabilidad de que el tweet sea positivo.

En esta sección, se define una función en el trabajo de Stream Analytics. Se puede invocar a la función para enviar un tweet al servicio web y obtener la respuesta.

1. Asegúrese de que tiene la dirección URL y la clave de API del servicio web que ha descargado anteriormente en el libro de Excel.

2. Vaya a su trabajo de Stream Analytics. Luego, seleccione **Functions** >  **+ Agregar** > **Azure ML Studio**

3. Rellene el formulario **Función de Azure Machine Learning** con los valores siguientes:

   |Campo  |Value  |
   |---------|---------|
   | Alias de función | Use el nombre `sentiment` y seleccione **Proporcionar la configuración de la función de Azure Machine Learning manualmente**, que proporciona una opción para especificar la dirección URL y la clave.      |
   | URL| Pegue la dirección URL del servicio web.|
   |Clave | Pegue la clave de API. |

4. Seleccione **Guardar**.

### <a name="create-a-query-to-transform-the-data"></a>Creación de una consulta para transformar los datos

Stream Analytics usa una consulta declarativa basada en SQL para examinar la entrada y procesarla. En esta sección se crea una consulta que lee cada tweet de entrada y luego invoca a la función de Machine Learning para llevar a cabo el análisis de opiniones. La consulta entonces envía el resultado a la salida que se ha definido (almacenamiento de blobs).

1. Vuelva a la información general del trabajo de Stream Analytics.

2. En **Topología de trabajo**, seleccione **Consulta**.

3. Escriba la siguiente consulta:

    ```SQL
    WITH sentiment AS (  
    SELECT text, sentiment1(text) as result 
    FROM <input>  
    )  

    SELECT text, result.[Score]  
    INTO <output>
    FROM sentiment  
    ```    

   La consulta invoca la función creada anteriormente (`sentiment`) para realizar el análisis de opinión en cada tweet de la entrada.

4. Seleccione **Guardar** para guardar la consulta.

## <a name="start-the-stream-analytics-job-and-check-the-output"></a>Inicio del trabajo de Stream Analytics y consulta de la salida

Ya se puede iniciar el trabajo de Stream Analytics.

### <a name="start-the-job"></a>Inicio del trabajo

1. Vuelva a la información general del trabajo de Stream Analytics.

2. En la parte superior de la página, seleccione **Iniciar**.

3. En **Iniciar trabajo**, seleccione **Personalizado** y, luego, seleccione un día antes de la fecha de carga del archivo CSV en el almacenamiento de blobs. Cuando finalice, seleccione **Guardar**.  

### <a name="check-the-output"></a>Consulta de la salida

1. Deje que el trabajo se ejecute durante unos minutos hasta que vea actividad en el cuadro **Supervisión**.

2. Si tiene alguna herramienta que emplee normalmente para examinar el contenido del almacenamiento de blobs, úsela para examinar el contenedor. También puede realizar los siguientes pasos en Azure Portal:

      1. En Azure Portal, busque la cuenta de almacenamiento y, en ella, busque el contenedor. En el contenedor verá dos archivos: el que contiene los tweets de ejemplo y un archivo CSV generado por el trabajo de Stream Analytics.
      2. Haga clic con el botón derecho en el archivo generado y luego seleccione **Descargar**.

3. Abra el archivo CSV generado. Verá algo parecido al siguiente ejemplo:  

   ![Machine Learning de Análisis de transmisiones, vista CSV](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-csv-view.png)  

### <a name="view-metrics"></a>Visualización de métricas

También puede observar las métricas relacionadas con la función de Azure Machine Learning. Las siguientes métricas relacionadas con la función se muestran en el cuadro **Supervisión** de la información general del trabajo:

* **Solicitudes de función** indica el número de solicitudes enviadas al servicio web Machine Learning.  
* **Eventos de la función** indica el número de eventos de la solicitud. De forma predeterminada, cada solicitud de un servicio web Machine Learning contiene hasta 1000 eventos.

## <a name="next-steps"></a>Pasos siguientes

* [Introducción a Azure Stream Analytics](stream-analytics-introduction.md)
* [Referencia del lenguaje de consulta de Azure Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)
* [Integración de la API de REST y Machine Learning](stream-analytics-how-to-configure-azure-machine-learning-endpoints-in-stream-analytics.md)
* [Referencia de API de REST de administración de Azure Stream Analytics](https://msdn.microsoft.com/library/azure/dn835031.aspx)
