---
title: Análisis de registros de sitios web mediante Azure Data Lake Analytics
description: Aprenda a analizar los registros de sitios web mediante Azure Data Lake Analytics para ejecutar funciones y consultas de U-SQL.
services: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: 3a196735-d0d9-4deb-ba68-c4b3f3be8403
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 12/05/2016
ms.openlocfilehash: 5ad837c51fafd7df2b019c78ae4adaf4967f2df7
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132643"
---
# <a name="analyze-website-logs-using-azure-data-lake-analytics"></a>Análisis de registros de sitios web mediante Azure Data Lake Analytics
Aprenda a analizar registros de sitios web mediante Análisis de Data Lake, especialmente en lo que se refiere a averiguar qué sitios de referencia presentaban errores cuando intentaron visitar el sitio web.

## <a name="prerequisites"></a>Requisitos previos
* **Visual Studio 2015 o Visual Studio 2013**.
* **[Data Lake Tools para Visual Studio](https://aka.ms/adltoolsvs)** .

    Después de instalar Data Lake Tools for Visual Studio, verá un elemento **Data Lake** en el menú **Herramientas** de Visual Studio:

    ![Menú de Visual Studio U-SQL](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-data-lake-tools-menu.png)
* **Conocimientos básicos de Análisis de Data Lake y Data Lake Tools for Visual Studio**. Para empezar, consulte:

  * [Desarrollo de scripts U-SQL mediante Data Lake Tools for Visual Studio](data-lake-analytics-data-lake-tools-get-started.md).
* **Una cuenta de Análisis de Data Lake.**  Consulte [Creación de una cuenta de Azure Data Lake Analytics](data-lake-analytics-get-started-portal.md).
* **Instale los datos de ejemplo.** En Azure Portal, abra su cuenta de Data Lake Analytics, haga clic en **Scripts de ejemplo** en el menú izquierdo y, luego, haga clic en **Copiar datos de muestra**. 

## <a name="connect-to-azure"></a>Conexión con Azure
Para poder crear y probar cualquier script U-SQL, primero debe conectarse a Azure.

### <a name="to-connect-to-data-lake-analytics"></a>Para conectarse a Data Lake Analytics

1. Abra Visual Studio.
2. Haga clic en **Data Lake > Opciones y configuración**.
3. Haga clic en **Iniciar sesión** o **Cambiar usuario** si alguien ya inició sesión y siga las instrucciones.
4. Haga clic en **Aceptar** para cerrar el cuadro de diálogo Opciones y configuración.

### <a name="to-browse-your-data-lake-analytics-accounts"></a>Para examinar las cuentas de Data Lake Analytics

1. En Visual Studio, presione **CTRL+ALT+S** para abrir el **Explorador de servidores**.
2. En el **Explorador de servidores**, expanda **Azure** y después **Data Lake Analytics**. Verá una lista de las cuentas de Análisis de Data Lake, si las hay. No se pueden crear cuentas de Análisis de Data Lake desde Visual Studio. Para crear una cuenta, consulte [Tutorial: Introducción a Azure Data Lake Analytics mediante Azure Portal](data-lake-analytics-get-started-portal.md) o [Tutorial: Introducción a Azure Data Lake Analytics mediante Azure PowerShell](data-lake-analytics-get-started-powershell.md).

## <a name="develop-u-sql-application"></a>Desarrollo de aplicaciones U-SQL
Una aplicación U-SQL es principalmente un script U-SQL. Para obtener más información acerca de U-SQL, consulte [Introducción a U-SQL](data-lake-analytics-u-sql-get-started.md).

Puede agregar operadores de suma definidos por el usuario a la aplicación.  Para obtener más información, consulte [Desarrollo de operadores U-SQL definidos por el usuario para trabajos de Análisis de Data Lake](data-lake-analytics-u-sql-develop-user-defined-operators.md).

### <a name="to-create-and-submit-a-data-lake-analytics-job"></a>Para crear y enviar un trabajo de Data Lake Analytics

1. Haga clic en el **Archivo > Nuevo > Proyecto**.

2. Seleccione el tipo de proyecto U-SQL.

   ![nuevo proyecto de Visual Studio U-SQL](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-data-lake-tools-new-project.png)

3. Haga clic en **OK**. Visual Studio crea una solución con un archivo Script.usql.

4. Incluya el siguiente script en el archivo Script.usql:

   ```usql
   // Create a database for easy reuse, so you don't need to read from a file very time.
   CREATE DATABASE IF NOT EXISTS SampleDBTutorials;

   // Create a Table valued function. TVF ensures that your jobs fetch data from he weblog file with the correct schema.
   DROP FUNCTION IF EXISTS SampleDBTutorials.dbo.WeblogsView;
   CREATE FUNCTION SampleDBTutorials.dbo.WeblogsView()
   RETURNS @result TABLE
   (
       s_date DateTime,
       s_time string,
       s_sitename string,
       cs_method string,
       cs_uristem string,
       cs_uriquery string,
       s_port int,
       cs_username string,
       c_ip string,
       cs_useragent string,
       cs_cookie string,
       cs_referer string,
       cs_host string,
       sc_status int,
       sc_substatus int,
       sc_win32status int,
       sc_bytes int,
       cs_bytes int,
       s_timetaken int
   )
   AS
   BEGIN

       @result = EXTRACT
           s_date DateTime,
           s_time string,
           s_sitename string,
           cs_method string,
           cs_uristem string,
           cs_uriquery string,
           s_port int,
           cs_username string,
           c_ip string,
           cs_useragent string,
           cs_cookie string,
           cs_referer string,
           cs_host string,
           sc_status int,
           sc_substatus int,
           sc_win32status int,
           sc_bytes int,
           cs_bytes int,
           s_timetaken int
       FROM @"/Samples/Data/WebLog.log"
       USING Extractors.Text(delimiter:' ');
       RETURN;
   END;

   // Create a table for storing referrers and status
   DROP TABLE IF EXISTS SampleDBTutorials.dbo.ReferrersPerDay;
   @weblog = SampleDBTutorials.dbo.WeblogsView();
   CREATE TABLE SampleDBTutorials.dbo.ReferrersPerDay
   (
       INDEX idx1
       CLUSTERED(Year ASC)
       DISTRIBUTED BY HASH(Year)
   ) AS

   SELECT s_date.Year AS Year,
       s_date.Month AS Month,
       s_date.Day AS Day,
       cs_referer,
       sc_status,
       COUNT(DISTINCT c_ip) AS cnt
   FROM @weblog
   GROUP BY s_date,
           cs_referer,
           sc_status;
   ```

    Para comprender U-SQL, consulte [Introducción al lenguaje U-SQL de Análisis de Data Lake](data-lake-analytics-u-sql-get-started.md).

5. Agregue un nuevo script de U-SQL al proyecto y escriba lo siguiente:

   ```usql
   // Query the referrers that ran into errors
   @content =
       SELECT *
       FROM SampleDBTutorials.dbo.ReferrersPerDay
       WHERE sc_status >=400 AND sc_status < 500;

   OUTPUT @content
   TO @"/Samples/Outputs/UnsuccessfulResponses.log"
   USING Outputters.Tsv();
   ```

6. Vuelva al primer script de U-SQL y junto al botón **Enviar** , especifique la cuenta de Análisis.

7. En el **Explorador de soluciones**, haga clic con el botón derecho en **Script.usql** y después haga clic en **Build Script** (Compilar script). Compruebe el resultado en el panel de resultados.

8. En el **Explorador de soluciones**, haga clic con el botón derecho en **Script.usql** y después haga clic en **Submit Script** (Enviar script).

9. Compruebe que la **Cuenta de Análisis** es aquella en la que desea ejecutar el trabajo y después haga clic en **Enviar**. Los resultados del envío y el vínculo del trabajo están disponibles en la ventana de resultados de Data Lake Tools for Visual Studio cuando se completa el envío.

10. Espere a que el trabajo se complete correctamente.  Si se produce un error en el trabajo, lo más probable es que falte el archivo de origen.  Consulte la sección Requisitos previos de este tutorial. Para obtener más información sobre la solución de problemas, consulte [Supervisión y solución de problemas con trabajos de Análisis de Azure Data Lake](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md).

    Una vez completado el trabajo, verá la siguiente pantalla:

    ![análisis de data lake analizar registros web registros de sitios web](./media/data-lake-analytics-analyze-weblogs/data-lake-analytics-analyze-weblogs-job-completed.png)

11. Ahora repita los pasos del 7 al 10 para **Script1.usql**.

### <a name="to-see-the-job-output"></a>Para ver la salida del trabajo

1. En el **Explorador de servidores**, expanda **Azure**, **Data Lake Analytics**, la cuenta de Data Lake Analytics y **Cuentas de almacenamiento**. A continuación, haga clic con el botón derecho en la cuenta predeterminada de Data Lake Store y después haga clic en **Explorador**.
2. Haga doble clic en **Ejemplos** para abrir la carpeta y después haga doble clic en **Salidas**.
3. Haga doble clic en **UnsuccessfulResponses.log**.
4. También puede hacer doble clic en el archivo de salida en la vista de gráfico del trabajo para ir directamente a la salida.

## <a name="next-steps"></a>Pasos siguientes
Para empezar a trabajar con Análisis de Data Lake usando diferentes herramientas, consulte:

* [Introducción a Data Lake Analytics mediante Azure Portal](data-lake-analytics-get-started-portal.md)
* [Introducción a Data Lake Analytics mediante Azure PowerShell](data-lake-analytics-get-started-powershell.md)
* [Introducción a Data Lake Analytics mediante .NET SDK](data-lake-analytics-get-started-net-sdk.md)
