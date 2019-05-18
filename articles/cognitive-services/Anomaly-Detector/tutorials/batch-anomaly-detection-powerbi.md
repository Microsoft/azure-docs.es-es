---
title: Visualizar las anomalías con detección de lote y Power BI
titlesuffix: Azure Cognitive Services
description: Usar la API del Detector de anomalías y Power BI para visualizar las anomalías a lo largo de los datos de series temporales.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: article
ms.date: 04/30/2019
ms.author: aahi
ms.openlocfilehash: de353c946da46b2bbe4e755e21e355f6bd18725b
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65827132"
---
# <a name="tutorial-visualize-anomalies-using-batch-detection-and-power-bi"></a>Tutorial: Visualizar las anomalías con detección de lote y Power BI

Use este tutorial para detectar anomalías dentro de un conjunto de datos de serie temporal como un lote. Con Power BI desktop, tomará un archivo de Excel, preparar los datos para la API del Detector de anomalías y visualizar las anomalías estadísticas en ella.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Usar Power BI Desktop para importar y transformar un conjunto de datos de serie temporal
> * Integración de Power BI Desktop con la API del Detector de anomalías para la detección de anomalías de lote
> * Visualizar las anomalías que se encuentra dentro de los datos, incluidos los valores esperados y vistos y los límites de detección de anomalías.

## <a name="prerequisites"></a>Requisitos previos

* [Microsoft Power BI Desktop](https://powerbi.microsoft.com/get-started/), disponible de forma gratuita.
* Puntos de excel (.xlsx) del archivo contenedor tiempo serie datos. Los datos de ejemplo para este inicio rápido pueden encontrarse en [GitHub](https://go.microsoft.com/fwlink/?linkid=2090962)

[!INCLUDE [cognitive-services-anomaly-detector-data-requirements](../../../../includes/cognitive-services-anomaly-detector-data-requirements.md)]

[!INCLUDE [cognitive-services-anomaly-detector-signup-requirements](../../../../includes/cognitive-services-anomaly-detector-signup-requirements.md)]

## <a name="load-and-format-the-time-series-data"></a>Carga y dar formato a los datos de serie temporal

Para empezar, abra Power BI Desktop y cargar los datos de series temporales que descargó desde los requisitos previos. Este archivo de excel contiene una serie de pares de marca de tiempo y el valor de hora Universal coordinada (UTC).  

> [!NOTE]
> Power BI puede usar datos de una amplia variedad de orígenes, como archivos .csv, las bases de datos SQL, almacenamiento de blobs de Azure y mucho más.  

En la ventana principal de Power BI Desktop, haga clic en el **inicio** cinta de opciones. En el **datos externos** grupo de la cinta de opciones, abra el **obtener datos** menú desplegable y haga clic en **Excel**.

![Una imagen del botón "Get Data" en Power BI](../media/tutorials/power-bi-get-data-button.png)

Después de que aparezca el cuadro de diálogo, navegue hasta la carpeta donde descargó el archivo .xlsx de ejemplo y selecciónelo. Después de la **navegador** aparece el cuadro de diálogo, haga clic en **Sheet1**y, a continuación, **editar**.

![Una imagen de la pantalla de "Navegador" de origen de datos en Power BI](../media/tutorials/navigator-dialog-box.png)

Power BI, las marcas de tiempo en la primera columna se convertirá un `Date/Time` tipo de datos. Estas marcas de tiempo se deben convertir en texto para enviarlos a la API del Detector de anomalías. Si no se abre automáticamente el editor de consultas de energía, haga clic en **editar consultas** en la pestaña Inicio. 

Haga clic en el **transformar** cinta de opciones en el Editor de Power Query. En el **cualquier columna** grupo, abra el **tipo de datos:** menú desplegable y seleccione **texto**.

![Una imagen de la pantalla de "Navegador" de origen de datos en Power BI](../media/tutorials/data-type-drop-down.png)

Cuando reciba un aviso acerca de cómo cambiar el tipo de columna, haga clic en **actual reemplace**. Después, haga clic en **cerrar y aplicar** o **aplicar** en el **inicio** cinta de opciones. 

## <a name="create-a-function-to-send-the-data-and-format-the-response"></a>Cree una función para enviar los datos y dar formato a la respuesta

Para dar formato y enviar el archivo de datos a la API del Detector de anomalías, puede invocar una consulta en la tabla creada anteriormente. En el Editor de Power Query, desde el **inicio** cinta de opciones, abra el **nuevo origen** menú desplegable y haga clic en **consulta en blanco**.

Asegúrese de que está seleccionada la nueva consulta, a continuación, haga clic en **Editor avanzado**. 

![Una imagen del botón "Editor avanzado" en Power BI](../media/tutorials/advanced-editor-screen.png)

En el Editor avanzado, use el siguiente fragmento de Power Query M para extraer las columnas de la tabla y enviarlo a la API. Después, la consulta cree una tabla de la respuesta JSON y devolverlo. Reemplace el `apiKey` variable con su clave de API del Detector de anomalías válida, y `endpoint` con el punto de conexión. Después de haber especificado la consulta en el Editor avanzado, haga clic en **realiza**.

```M
(table as table) => let

    apikey      = "[Placeholder: Your Anomaly Detector resource access key]",
    endpoint    = "[Placeholder: Your Anomaly Detector resource endpoint]/anomalydetector/v1.0/timeseries/entire/detect",
    inputTable = Table.TransformColumnTypes(table,{{"Timestamp", type text},{"Value", type number}}),
    jsontext    = Text.FromBinary(Json.FromValue(inputTable)),
    jsonbody    = "{ ""Granularity"": ""daily"", ""Sensitivity"": 95, ""Series"": "& jsontext &" }",
    bytesbody   = Text.ToBinary(jsonbody),
    headers     = [#"Content-Type" = "application/json", #"Ocp-Apim-Subscription-Key" = apikey],
    bytesresp   = Web.Contents(endpoint, [Headers=headers, Content=bytesbody]),
    jsonresp    = Json.Document(bytesresp),

    respTable = Table.FromColumns({
                    
                     Table.Column(inputTable, "Timestamp")
                     ,Table.Column(inputTable, "Value")
                     , Record.Field(jsonresp, "IsAnomaly") as list
                     , Record.Field(jsonresp, "ExpectedValues") as list
                     , Record.Field(jsonresp, "UpperMargins")as list
                     , Record.Field(jsonresp, "LowerMargins") as list
                     , Record.Field(jsonresp, "IsPositiveAnomaly") as list
                     , Record.Field(jsonresp, "IsNegativeAnomaly") as list

                  }, {"Timestamp", "Value", "IsAnomaly", "ExpectedValues", "UpperMargin", "LowerMargin", "IsPositiveAnomaly", "IsNegativeAnomaly"}
               ),
    
    respTable1 = Table.AddColumn(respTable , "UpperMargins", (row) => row[ExpectedValues] + row[UpperMargin]),
    respTable2 = Table.AddColumn(respTable1 , "LowerMargins", (row) => row[ExpectedValues] -  row[LowerMargin]),
    respTable3 = Table.RemoveColumns(respTable2, "UpperMargin"),
    respTable4 = Table.RemoveColumns(respTable3, "LowerMargin"),

    results = Table.TransformColumnTypes(

                respTable4,
                {{"Timestamp", type datetime}, {"Value", type number}, {"IsAnomaly", type logical}, {"IsPositiveAnomaly", type logical}, {"IsNegativeAnomaly", type logical},
                 {"ExpectedValues", type number}, {"UpperMargins", type number}, {"LowerMargins", type number}}
              )

 in results
```

Invocar la consulta en la hoja de datos seleccionando `Sheet1` debajo **escribir parámetro**y haga clic en **Invoke**. 

![Una imagen del botón "Editor avanzado"](../media/tutorials/invoke-function-screenshot.png)

## <a name="data-source-privacy-and-authentication"></a>Autenticación y la privacidad del origen de datos

> [!NOTE]
> Tenga en cuenta las directivas de su organización para el acceso y privacidad de los datos. Consulte [niveles de privacidad de Power BI Desktop](https://docs.microsoft.com/power-bi/desktop-privacy-levels) para obtener más información.

Es posible que obtenga un mensaje de advertencia cuando se intenta ejecutar la consulta, dado que usa un origen de datos externo. 

![Una imagen que muestra una advertencia creada por Power BI](../media/tutorials/blocked-function.png)

Para solucionar este problema, haga clic en **archivo**, y **opciones y configuración**. A continuación, haga clic en **opciones**. A continuación **archivo actual**, seleccione **privacidad**, y **ignorar los niveles de privacidad y mejorar el rendimiento potencialmente**. 

Además, puede aparecer un mensaje que le pide que especifique cómo desea conectarse a la API.

![Una imagen que muestra una solicitud para especificar las credenciales de acceso](../media/tutorials/edit-credentials-message.png)

Para solucionar este problema, haga clic en **editar credenciales** en el mensaje. Después de que aparezca el cuadro de diálogo, seleccione **Anonymous** para conectarse a la API de forma anónima. Haga clic en **Conectar**. 

Después, haga clic en **cerrar y aplicar** en el **inicio** cinta de opciones para aplicar los cambios.

## <a name="visualize-the-anomaly-detector-api-response"></a>Visualizar la respuesta de API del Detector de anomalías

En la pantalla principal de Power BI, empiece con las consultas que creó anteriormente para visualizar los datos. Primero seleccione **gráfico de líneas** en **visualizaciones**. A continuación, agregar la marca de tiempo desde la función invocada en el gráfico de líneas **eje**. Haga doble clic en él y seleccione **Timestamp**. 

![Haciendo clic en el valor de marca de tiempo](../media/tutorials/timestamp-right-click.png)

Agregue los siguientes campos de la **invoca la función** para el gráfico **valores** campo. Use la siguiente captura de pantalla para ayudar a crear el gráfico.

    * Valor
    * UpperMargins
    * LowerMargins
    * ExpectedValues

![Una imagen de la nueva pantalla de la medida rápida](../media/tutorials/chart-settings.png)

Después de agregar los campos, haga clic en el gráfico y cambie su tamaño para mostrar todos los puntos de datos. El gráfico tendrá un aspecto similar a la siguiente captura de pantalla:

![Una imagen de la nueva pantalla de la medida rápida](../media/tutorials/chart-visualization.png)

### <a name="display-anomaly-data-points"></a>Muestra los puntos de datos de anomalías

En el lado derecho de la ventana de Power BI, a continuación el **campos** panel, el botón secundario en **valor** bajo el **consulta invoca la función**y haga clic en **rápido nuevo medida**.

![Una imagen de la nueva pantalla de la medida rápida](../media/tutorials/new-quick-measure.png)

En la pantalla que aparece, seleccione **filtrar valor** como el cálculo. Establecer **valor Base** a `Sum of Value`. A continuación, arrastre `IsAnomaly` desde el **invoca la función** campos a **filtro**. Seleccione `True` desde el **filtro** menú desplegable.

![Una imagen de la nueva pantalla de la medida rápida](../media/tutorials/new-quick-measure-2.png)

Después de hacer clic **Aceptar**, tendrá un `Value for True` campo, en la parte inferior de la lista de los campos. Haga clic en él y cambie su nombre a **anomalías**. Agréguelo al gráfico **valores**. A continuación, seleccione el **formato** de herramientas y establezca el tipo de eje x en **categorías**.

![Una imagen de la nueva pantalla de la medida rápida](../media/tutorials/format-x-axis.png)

Aplicar colores al gráfico haciendo clic en el **formato** herramienta y **colores de datos**. El gráfico debe tener un aspecto similar al siguiente:

![Una imagen de la nueva pantalla de la medida rápida](../media/tutorials/final-chart.png)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
>[Detección de anomalías de streaming con Azure Databricks](anomaly-detection-streaming-databricks.md)
