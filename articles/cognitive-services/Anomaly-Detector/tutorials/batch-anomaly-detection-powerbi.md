---
title: 'Tutorial: Visualización de anomalías con detección por lotes y Power BI'
titleSuffix: Azure Cognitive Services
description: Aprenda a usar Anomaly Detector API y Power BI para visualizar anomalías en los datos de series temporales.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: tutorial
ms.date: 06/17/2020
ms.author: aahi
ms.openlocfilehash: 527ce1c7d434ae94c91c78c865c00aa0687a73cb
ms.sourcegitcommit: c293217e2d829b752771dab52b96529a5442a190
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/15/2020
ms.locfileid: "88245509"
---
# <a name="tutorial-visualize-anomalies-using-batch-detection-and-power-bi"></a>Tutorial: Visualización de anomalías con detección por lotes y Power BI

Use este tutorial para detectar anomalías dentro de un conjunto de datos de serie temporal como un lote. Con Power BI Desktop, puede tomar un archivo de Excel, preparar los datos para Anomaly Detector API y visualizar las anomalías estadísticas del archivo.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Usar Power BI Desktop para importar y transformar un conjunto de datos de serie temporal
> * Integrar Power BI Desktop con Anomaly Detector API para la detección de anomalías por lotes
> * Visualizar las anomalías que se encuentran en los datos, incluidos los valores esperados y vistos, y los límites de la detección de anomalías.

## <a name="prerequisites"></a>Requisitos previos
* [Una suscripción de Azure](https://azure.microsoft.com/free/cognitive-services)
* [Microsoft Power BI Desktop](https://powerbi.microsoft.com/get-started/), disponible de forma gratuita.
* Un archivo de Excel (.xlsx) que contenga los puntos de datos de la serie temporal. Los datos de ejemplo de este inicio rápido están disponibles en [GitHub](https://go.microsoft.com/fwlink/?linkid=2090962).
* Cuando tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector"  title="Creación de un recurso de Anomaly Detector"  target="_blank">cree un recurso de Anomaly Detector <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Anomaly Detector API. Lo hará más adelante en el inicio rápido.

[!INCLUDE [cognitive-services-anomaly-detector-data-requirements](../../../../includes/cognitive-services-anomaly-detector-data-requirements.md)]

## <a name="load-and-format-the-time-series-data"></a>Carga y formato de los datos de la serie temporal

Para empezar, abra Power BI Desktop y cargue los datos de la serie temporal que descargó en los requisitos previos. Este archivo de Excel contiene una serie de pares de marca de tiempo y valor de hora universal coordinada (UTC).  

> [!NOTE]
> Power BI puede usar datos de una amplia variedad de orígenes como archivos .csv, bases de datos SQL, Azure Blob Storage, etc.  

En la ventana principal de Power BI Desktop, haga clic en la cinta de opciones **Inicio**. En el grupo **Datos externos** de la cinta de opciones, abra el menú desplegable **Obtener datos** y haga clic en **Excel**.

![Una imagen del botón "Obtener datos" en Power BI](../media/tutorials/power-bi-get-data-button.png)

Después de que aparezca el cuadro de diálogo, vaya a la carpeta donde descargó el archivo .xlsx de ejemplo y selecciónelo. Una vez que aparezca el cuadro de diálogo **Navegador**, haga clic en **Sheet1** y, a continuación, en **Editar**.

![Una imagen de la pantalla "Navegador" del origen de datos en Power BI](../media/tutorials/navigator-dialog-box.png)

Power BI convertirá las marcas de tiempo de la primera columna en un tipo de datos `Date/Time`. Estas marcas de tiempo se deben convertir a texto antes de enviarse a Anomaly Detector API. Si el editor Power Query no se abre automáticamente, haga clic en **Editar consultas** en la pestaña Inicio.

Haga clic en la pestaña **Transformar** de la cinta de opciones del editor Power Query. En el grupo **Cualquier columna**, abra el menú desplegable **Tipo de datos:** y seleccione **Texto**.

![Una imagen de la pantalla "Navegador" del origen de datos en Power BI](../media/tutorials/data-type-drop-down.png)

Si recibe un aviso sobre el cambio de tipo de columna, haga clic en **Sustituir la actual**. Después, haga clic en **Cerrar y aplicar** o en **Aplicar** en la pestaña **Inicio** de la cinta de opciones.

## <a name="create-a-function-to-send-the-data-and-format-the-response"></a>Creación de una función para enviar los datos y dar formato a la respuesta

Para dar formato y enviar el archivo de datos a Anomaly Detector API, puede invocar una consulta en la tabla que creó anteriormente. En el editor Power Query, en la pestaña **Inicio** de la cinta de opciones, abra el menú desplegable **Nuevo origen** y haga clic en **Consulta en blanco**.

Asegúrese de que está seleccionada la nueva consulta y, a continuación, haga clic en **Editor avanzado**.

![Una imagen del botón "Editor avanzado" en Power BI](../media/tutorials/advanced-editor-screen.png)

En el Editor avanzado, use el siguiente fragmento de Power Query M para extraer las columnas de la tabla y enviarlas a la API. Después, la consulta creará una tabla a partir de la respuesta JSON y la devolverá. Reemplace la variable `apiKey` por la clave de Anomaly Detector API y `endpoint` por el punto de conexión. Después de haber especificado la consulta en el Editor avanzado, haga clic en **Listo**.

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

Invoque la consulta en la hoja de datos seleccionando `Sheet1` debajo de **Escribir parámetro** y haga clic en **Invocar**.

![Una imagen del botón "Editor avanzado"](../media/tutorials/invoke-function-screenshot.png)

## <a name="data-source-privacy-and-authentication"></a>Privacidad y autenticación en el origen de datos

> [!NOTE]
> Conozca las directivas de la organización sobre la privacidad y el acceso a datos. Consulte los [niveles de privacidad de Power BI Desktop](https://docs.microsoft.com/power-bi/desktop-privacy-levels) para más información.

Es posible que reciba un mensaje de advertencia cuando intente ejecutar la consulta ya que usa un origen de datos externo.

![Una imagen que muestra una advertencia creada por Power BI](../media/tutorials/blocked-function.png)

Para solucionar este problema, haga clic en **Archivo** y en **Opciones y configuración**. Después, haga clic en **Opciones**. A continuación, en **Archivo actual**, seleccione **Privacidad** e **Ignorar los niveles de privacidad y mejorar el rendimiento potencialmente**.

Además, puede recibir un mensaje que le pide que especifique cómo desea conectarse a la API.

![Una imagen que muestra una solicitud para especificar las credenciales de acceso](../media/tutorials/edit-credentials-message.png)

Para solucionar este problema, haga clic en **Editar credenciales** en el mensaje. Después de que aparezca el cuadro de diálogo, seleccione **Anónimo** para conectarse a la API de forma anónima. Haga clic en **Conectar**.

Después, haga clic en **Cerrar y aplicar** en la pestaña **Inicio** de la cinta de opciones para aplicar los cambios.

## <a name="visualize-the-anomaly-detector-api-response"></a>Visualización de la respuesta de Anomaly Detector API

En la pantalla principal de Power BI, empiece a usar las consultas que creó anteriormente para visualizar los datos. Primero, seleccione **Gráfico de líneas** en **Visualizaciones**. A continuación, agregue la marca de tiempo desde la función invocada al **eje** del gráfico de líneas. Haga clic con el botón derecho en él y seleccione **Marca de tiempo**.

![Hacer clic con el botón derecho en el valor de marca de tiempo](../media/tutorials/timestamp-right-click.png)

Agregue los siguientes campos desde **Función invocada** al campo **Valores** del gráfico. Use la siguiente captura de pantalla para ayudar a crear el gráfico.

* Value
* UpperMargins
* LowerMargins
* ExpectedValues

![Una imagen de la nueva pantalla de medida rápida](../media/tutorials/chart-settings.png)

Después de agregar los campos, haga clic en el gráfico y cambie su tamaño para que aparezcan todos los puntos de datos. El gráfico tendrá un aspecto similar al de la siguiente captura de pantalla:

![Una imagen de la nueva pantalla de medida rápida](../media/tutorials/chart-visualization.png)

### <a name="display-anomaly-data-points"></a>Visualización de puntos de datos de anomalías

En el lado derecho de la ventana de Power BI, bajo el panel **CAMPOS**, haga clic con el botón derecho en **Valor** en la **consulta de Función invocada** y haga clic en **Nueva medida rápida**.

![Una imagen de la nueva pantalla de medida rápida](../media/tutorials/new-quick-measure.png)

En la pantalla que aparece, seleccione **Valor filtrado** como cálculo. Establezca **Valor base** en `Sum of Value`. A continuación, arrastre `IsAnomaly` desde los campos de **Función invocada** a **Filtro**. Seleccione `True` en el menú desplegable **Filtro**.

![Una imagen de la nueva pantalla de medida rápida](../media/tutorials/new-quick-measure-2.png)

Después de hacer clic en **Aceptar**, tendrá un campo `Value for True`, en la parte inferior de la lista de los campos. Haga clic con el botón derecho en ella y cambie su nombre a **Anomalías**. Agréguela a los **valores** del gráfico. A continuación, seleccione la herramienta **Formato** y establezca el tipo de eje X en **Categórico**.

![Una imagen de la nueva pantalla de medida rápida](../media/tutorials/format-x-axis.png)

Aplique colores al gráfico haciendo clic en la herramienta **Formato** y en **Colores de datos**. El gráfico debe tener un aspecto parecido al siguiente:

![Una imagen de la nueva pantalla de medida rápida](../media/tutorials/final-chart.png)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
>[Streaming de detección de anomalías con Azure Databricks](anomaly-detection-streaming-databricks.md)
