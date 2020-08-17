---
title: 'Consulta de Azure Data Lake Analytics: Visual Studio'
description: Aprenda a instalar Data Lake Tools para Visual Studio y a desarrollar y probar scripts U-SQL.
services: data-lake-analytics
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: ad8a6992-02c7-47d4-a108-62fc5a0777a3
ms.topic: how-to
ms.date: 08/30/2019
ms.openlocfilehash: 2f02c16d73ca07c1909bf84bfb7e158b429d3096
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87125979"
---
# <a name="develop-u-sql-scripts-by-using-data-lake-tools-for-visual-studio"></a>Desarrollo de scripts U-SQL mediante Data Lake Tools para Visual Studio

[!INCLUDE [get-started-selector](../../includes/data-lake-analytics-selector-get-started.md)]

Azure Data Lake y Herramientas de Stream Analytics incluyen una funcionalidad relacionada con dos servicios de Azure: Azure Data Lake Analytics y Azure Stream Analytics. Para obtener más información acerca de los escenarios de Azure Stream Analytics, consulte [Herramientas de Azure Stream Analytics para Visual Studio](../stream-analytics/stream-analytics-tools-for-visual-studio-install.md).

En este artículo se describe cómo puede usar Visual Studio para crear cuentas de Azure Data Lake Analytics. Puede definir trabajos en [U-SQL](data-lake-analytics-u-sql-get-started.md) y enviarlos al servicio de Data Lake Analytics. Para obtener más información acerca de Análisis de Data Lake, consulte [Información general sobre Análisis de Azure Data Lake](data-lake-analytics-overview.md).

> [!IMPORTANT]
> Le recomendamos que actualice a Herramientas de Azure Data Lake para Visual Studio versión 2.3.3000.4 o posterior. Las versiones anteriores no están disponibles para su descarga y han quedado en desuso.
>
> 1. Compruebe que su versión de Herramientas de Azure Data Lake para Visual Studio es anterior a 2.3.3000.4.
>
>    ![Comprobación de la versión de la herramienta](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-data-lake-tools-about-data-lake.png)
>
> 1. Si su versión es anterior a 2.3.3000.4, vaya al centro de descarga y actualice Herramientas de Azure Data Lake para Visual Studio:
>    - [Para Visual Studio 2017 y 2019](https://marketplace.visualstudio.com/items?itemName=ADLTools.AzureDataLakeandStreamAnalyticsTools)
>    - [Para Visual Studio 2013 y 2015](https://www.microsoft.com/en-us/download/details.aspx?id=49504)

## <a name="prerequisites"></a>Prerequisites

* **Visual Studio**: se admiten todas las ediciones, excepto Express.

  * Visual Studio 2019
  * Visual Studio 2017
  * Visual Studio 2015
  * Visual Studio 2013

* **Microsoft Azure SDK para .NET** versión 2.7.1, o posterior. Instálelo con el [Instalador de plataforma web](https://www.microsoft.com/web/downloads/platform.aspx).
* Una cuenta de **Data Lake Analytics**. Para crear una cuenta, consulte [Introducción a Azure Data Lake Analytics mediante Azure Portal](data-lake-analytics-get-started-portal.md).

## <a name="install-azure-data-lake-tools-for-visual-studio"></a>Instalación de Herramientas de Azure Data Lake para Visual Studio

Este tutorial requiere que esté instalado Data Lake Tools para Visual Studio. Para obtener más información, consulte [Instalación de Data Lake Tools para Visual Studio](data-lake-analytics-data-lake-tools-install.md).

## <a name="connect-to-an-azure-data-lake-analytics-account"></a>Conexión a una cuenta de Azure Data Lake Analytics

1. Abra Visual Studio.

1. Abra el **Explorador de servidores**. Para ello, seleccione **Vista** > **Explorador de servidores**.

1. Haga clic con el botón derecho en **Azure** y, después, seleccione **Connect to Microsoft Azure Subscription** (Conectar a la suscripción de Microsoft Azure). En **Iniciar sesión en su cuenta**, siga las instrucciones.

1. En el **Explorador de servidores**, seleccione **Azure** > **Data Lake Analytics**. Verá una lista de las cuentas de Data Lake Analytics.

## <a name="write-your-first-u-sql-script"></a>Escriba el primer script U-SQL

El siguiente texto es un script U-SQL simple. Define un conjunto de datos pequeño y lo escribe en la instancia predeterminada de Data Lake Store como un archivo denominado `/data.csv`.

```sql
USE DATABASE master;
USE SCHEMA dbo;
@a  = 
    SELECT * FROM 
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS 
              D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();
```

## <a name="submit-a-data-lake-analytics-job"></a>Envío de un trabajo de Análisis de Data Lake

1. Abra Visual Studio, seleccione **Archivo** > **Nuevo** > **Proyecto**.

1. Seleccione el tipo **U-SQL Project** (Proyecto U-SQL) y, luego, seleccione **Next** (Siguiente). En **Configurar el nuevo proyecto**, seleccione **Crear**.

   Visual Studio crea una solución que contiene un archivo **Script.usql**.

1. Pegue el script de [Escriba el primer script U-SQL](#write-your-first-u-sql-script) en la ventana **Script.usql**.

1. En el **Explorador de soluciones**, haga clic con el botón derecho en **Script.usql** y seleccione **Enviar script**.

1. En **Enviar trabajo**, elija su cuenta de Data Lake Analytics y seleccione **Enviar**.

   ![Enviar proyecto U-SQL de Visual Studio](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-submit-job-vs2019.png)

Tras el envío del trabajo, se abre la pestaña **Vista Trabajo** para mostrar el progreso del trabajo.

* **Resumen del trabajo** muestra el resumen del trabajo.
* **Grafo de trabajo** muestra el progreso del trabajo.
* **Operaciones de metadatos** muestra todas las acciones que se realizaron en el catálogo de U-SQL.
* **Datos** muestra todas las entradas y salidas.
* **Historial de estado** muestra los detalles de la escala de tiempo y el estado.
* **Análisis de UA** muestra el número de unidades de análisis que se han usado en el trabajo y explora simulaciones de diferentes estrategias de asignación de unidades de análisis.
* **Diagnósticos** proporciona un análisis avanzado para la optimización de rendimiento y la ejecución del trabajo.

![Gráfico de rendimiento del trabajo de Data Lake Analytics de U-SQL Visual Studio](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-data-lake-tools-performance-graph.png)

Para ver el estado del trabajo más reciente y actualizar la pantalla, seleccione **Actualizar**.

## <a name="check-job-status"></a>Comprobación del estado del trabajo

1. En el **Explorador de servidores**, seleccione **Azure** > **Data Lake Analytics**.

1. Expanda el nombre de la cuenta de Data Lake Analytics.

1. Haga doble clic en **Trabajos**.

1. Seleccione el trabajo que envió anteriormente.

## <a name="see-the-job-output"></a>Visualización de la salida del trabajo

1. En el **Explorador de servidores**, navegue al trabajo que ha enviado.

1. Haga clic en la pestaña **Datos** .

1. En el pestaña **Salidas de trabajo**, seleccione el archivo `"/data.csv"`.

## <a name="next-steps"></a>Pasos siguientes

* [Prueba y depuración de trabajos U-SQL mediante la ejecución local y el SDK de U-SQL para Azure Data Lake](data-lake-analytics-data-lake-tools-local-run.md)
* [Depuración de código de C# en los trabajos de U-SQL con las Herramientas de Azure Data Lake para Visual Studio Code](data-lake-tools-for-vscode-local-run-and-debug.md)
* [Uso de Azure Data Lake Tools para Visual Studio Code](data-lake-analytics-data-lake-tools-for-vscode.md)
