---
title: Crear una canalización de datos con la API del recopilador de datos de Azure Monitor | Microsoft Docs
description: Puede usar Azure Monitor HTTP Data Collector API para agregar datos POST JSON en el área de trabajo de Log Analytics desde cualquier cliente que pueda llamar a la API de REST. En este artículo se describe cómo cargar datos almacenados en archivos de una manera automatizada.
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 08/09/2018
ms.author: magoedte
ms.openlocfilehash: 53457a044f5c69af7bf68561f24732e8f02219d8
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65603241"
---
# <a name="create-a-data-pipeline-with-the-data-collector-api"></a>Creación de una canalización de datos con Data Collector API

[Data Collector API de Azure Monitor](data-collector-api.md) permite importar los datos de registros personalizados en un área de trabajo de Log Analytics en Azure Monitor. Los únicos requisitos son que los datos tengan formato JSON y estén divididos en segmentos de 30 MB o menos. Se trata de un mecanismo completamente flexible que se puede utilizar de muchas maneras: desde datos enviados directamente por la aplicación hasta cargas ad hoc de un solo uso. Este artículo describe algunos puntos de partida para un escenario común: la necesidad de cargar los datos almacenados en archivos de forma regular y automatizada. Aunque la canalización que se presenta aquí no será la de mayor rendimiento ni optimización, está pensada para servir como punto de partida para la creación de una canalización de producción propia.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="example-problem"></a>Problema de ejemplo
El resto de este artículo, examinaremos los datos de la vista de página de Application Insights. En nuestro escenario hipotético, queremos correlacionar información geográfica que se recopilan de forma predeterminada mediante el SDK de Application Insights a los datos personalizados que contiene la población de cada país o región del mundo, con el fin de identificar donde nos debemos dedicar el marketing más dólares. 

Se usa un origen de datos público como [UN World Population Prospects](https://esa.un.org/unpd/wpp/) (Perspectivas de población mundial de la ONU) para este propósito. Los datos tendrán el siguiente esquema simple:

![Esquema de ejemplo simple](./media/create-pipeline-datacollector-api/example-simple-schema-01.png)

En nuestro ejemplo, se supone que se cargará un archivo nuevo con los datos del último año tan pronto como esté disponible.

## <a name="general-design"></a>Diseño general
Se usa una lógica de tipo ETL clásica para diseñar nuestra canalización. La arquitectura tendrá el aspecto siguiente:

![Arquitectura de la canalización de recopilación de datos](./media/create-pipeline-datacollector-api/data-pipeline-dataflow-architecture.png)

En este artículo no se explica cómo crear datos ni cómo [cargarlos en una cuenta de Azure Blob Storage](../../storage/blobs/storage-upload-process-images.md). En su lugar, se recoge el flujo en cuanto se carga un archivo nuevo en el blob. Desde aquí:

1. Un proceso detectará que se han cargado nuevos datos.  En nuestro ejemplo se utiliza una [aplicación lógica de Azure](../../logic-apps/logic-apps-overview.md) que dispone de un desencadenador para detectar la carga de nuevos datos en un blob.

2. Un procesador lee estos nuevos datos y los convierte en JSON, el formato requerido por Azure Monitor. En este ejemplo, usamos una [función de Azure](../../azure-functions/functions-overview.md) como una manera ligera, eficaz y rentable de ejecutar el código de procesamiento. La misma aplicación lógica que se utiliza para detectar nuevos datos inicia la función.

3. Por último, una vez que el objeto JSON está disponible, se envía a Azure Monitor. La misma aplicación lógica envía los datos a Azure Monitor mediante la actividad integrada de Log Analytics Data Collector.

Aunque la configuración detallada del almacenamiento de blobs, la aplicación lógica o la función de Azure no se describen en este artículo, hay disponibles instrucciones detalladas en las páginas de los productos específicos.

Para supervisar esta canalización, se usan Application Insights para supervisar la función de Azure [(más información aquí)](../../azure-functions/functions-monitoring.md) y Azure Monitor para supervisar la aplicación lógica [(más información aquí)](../../logic-apps/logic-apps-monitor-your-logic-apps-oms.md). 

## <a name="setting-up-the-pipeline"></a>Configuración de la canalización
Para establecer la canalización, primero debe asegurarse de que tiene el contenedor de blobs creado y configurado. Del mismo modo, asegúrese de que se ha creado el área de trabajo de Log Analytics donde le gustaría enviar los datos.

## <a name="ingesting-json-data"></a>Ingesta de datos JSON
La ingesta de datos JSON es trivial con Logic Apps y, puesto que no se necesita llevar a cabo ninguna transformación, podemos encapsular toda la canalización en una única aplicación lógica. Una vez que se han configurado el contenedor de blobs y el área de trabajo de Log Analytics, cree una nueva aplicación lógica y configúrela como sigue:

![Ejemplo de flujo de trabajo de Logic Apps](./media/create-pipeline-datacollector-api/logic-apps-workflow-example-01.png)

Guarde la aplicación lógica y pruébela.

## <a name="ingesting-xml-csv-or-other-formats-of-data"></a>Ingesta de XML, CSV u otros formatos de datos
Hoy en día, Logic Apps no tiene funcionalidades integradas para transformar fácilmente XML, CSV u otros tipos al formato JSON. Por lo tanto, es necesario usar otros medios para realizar esta transformación. En este artículo, usamos las funcionalidades de proceso sin servidor de Azure Functions como una manera muy ligera y rentable de hacerlo. 

En este ejemplo se analiza un archivo CSV, pero se puede procesar cualquier otro tipo de archivo de forma similar. Basta con modificar el apartado de deserialización de la función de Azure para reflejar la lógica adecuada para el tipo de datos específico.

1.  Cree una nueva función de Azure, con el runtime v1 de la función y la opción basado en consumo cuando se le solicite.  Seleccione la plantilla **Desencadenador HTTP** para C# como punto de partida, que configura los enlaces tal y como se necesita. 
2.  En la pestaña **Ver archivos** del panel derecho, cree un nuevo archivo llamado **project.json** y pegue el siguiente código de los paquetes de NuGet que estamos usando:

    ![Proyecto de ejemplo de Azure Functions](./media/create-pipeline-datacollector-api/functions-example-project-01.png)
    
    ``` JSON
    {
      "frameworks": {
        "net46":{
          "dependencies": {
            "CsvHelper": "7.1.1",
            "Newtonsoft.Json": "11.0.2"
          }  
        }  
       }  
     }  
    ```

3. Cambie a **run.csx** en el panel derecho y reemplace el código predeterminado por el siguiente. 

    >[!NOTE]
    >Para el proyecto, debe reemplazar el modelo de registro (la clase "PopulationRecord") por su propio esquema de datos.
    >

    ```   
    using System.Net;
    using Newtonsoft.Json;
    using CsvHelper;
    
    class PopulationRecord
    {
        public String Location { get; set; }
        public int Time { get; set; }
        public long Population { get; set; }
    }

    public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
    {
        string filePath = await req.Content.ReadAsStringAsync(); //get the CSV URI being passed from Logic App
        string response = "";

        //get a stream from blob
        WebClient wc = new WebClient();
        Stream s = wc.OpenRead(filePath);         

        //read the stream
        using (var sr = new StreamReader(s))
        {
            var csvReader = new CsvReader(sr);
    
            var records = csvReader.GetRecords<PopulationRecord>(); //deserialize the CSV stream as an IEnumerable
    
            response = JsonConvert.SerializeObject(records); //serialize the IEnumerable back into JSON
        }    

        return response == null
            ? req.CreateResponse(HttpStatusCode.BadRequest, "There was an issue getting data")
            : req.CreateResponse(HttpStatusCode.OK, response);
     }  
    ```

4. Guarde la función.
5. Pruebe la función para asegurarse de que el código funciona correctamente. Cambie a la pestaña **Probar** en el panel derecho y configure la prueba como sigue. Especifique un vínculo a un blob con datos de ejemplo en cuadro de texto **Cuerpo de la solicitud**. Después de hacer clic en **Ejecutar**, debería ver la salida JSON en el cuadro **Salida**:

    ![Código de prueba de Function App](./media/create-pipeline-datacollector-api/functions-test-01.png)

Ahora debemos volver atrás y modificar la aplicación lógica que comenzamos a crear anteriormente para incluir los datos ingeridos y convertidos a formato JSON.  Mediante el Diseñador de vistas, realice la configuración como se indica a continuación y guarde la aplicación lógica:

![Ejemplo completo del flujo de trabajo de Logic Apps](./media/create-pipeline-datacollector-api/logic-apps-workflow-example-02.png)

## <a name="testing-the-pipeline"></a>Prueba de la canalización
Ahora puede cargar un archivo nuevo en el blob que configuró anteriormente y será supervisado por la aplicación lógica. En unos instantes, verá el inicio de una nueva instancia de la aplicación lógica, la llamada a la función de Azure y, después, el envío correcto de los datos a Azure Monitor. 

>[!NOTE]
>Puede tardar hasta 30 minutos para que los datos aparezcan en Azure Monitor la primera vez que envía un nuevo tipo de datos.


## <a name="correlating-with-other-data-in-log-analytics-and-application-insights"></a>Correlación con otros datos de Log Analytics y Application Insights
Para completar nuestro objetivo de poner en correlación los datos de la vista de página de Application Insights con los datos de población que se han ingerido del origen de datos personalizado, ejecute la siguiente consulta desde la ventana de Application Insights Analytics o en el área de trabajo de Log Analytics:

``` KQL
app("fabrikamprod").pageViews
| summarize numUsers = count() by client_CountryOrRegion
| join kind=leftouter (
   workspace("customdatademo").Population_CL
) on $left.client_CountryOrRegion == $right.Location_s
| project client_CountryOrRegion, numUsers, Population_d
```

La salida ahora debe mostrar los dos orígenes de datos unidos.  

![Correlación de datos separados en un ejemplo de resultado de búsqueda](./media/create-pipeline-datacollector-api/correlating-disjoined-data-example-01.png)

## <a name="suggested-improvements-for-a-production-pipeline"></a>Mejoras sugeridas para una canalización de producción
En este artículo se presenta un prototipo funcional, cuya lógica subyacente se puede aplicar a una verdadera solución con calidad de producción. Para esta solución con calidad de producción, se recomiendan las siguientes mejoras:

* Agregue control de errores y lógica de reintento en la aplicación lógica y en la función.
* Agregue lógica para asegurarse de que no se supera el límite de 30 MB en una sola llamada a Log Analytics Ingestion API. Divida los datos en segmentos más pequeños si es necesario.
* Configure una directiva de limpieza en el almacenamiento de blobs. Una vez que se han enviado correctamente al área de trabajo de Log Analytics, a menos que quiera mantener los datos sin procesar disponibles para fines de archivo, no hay ninguna razón para seguir almacenándolos. 
* Compruebe que la supervisión está habilitada en la canalización completa y agregue puntos de seguimiento y alertas según corresponda.
* Aproveche las ventajas del control de código fuente para administrar el código de la función y la aplicación lógica.
* Asegúrese de seguir una directiva de administración de cambios adecuada, de manera que si cambia el esquema, la función y la aplicación lógica se modifican en consecuencia.
* Si va a cargar varios tipos de datos diferentes, divídalos en carpetas individuales dentro del contenedor de blobs y cree la lógica en función del tipo de datos. 


## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre [Data Collector API](data-collector-api.md) para escribir datos en el área de trabajo de Log Analytics desde cualquier cliente de API REST.
