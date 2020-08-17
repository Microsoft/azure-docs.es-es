---
title: Exportación a SQL desde Azure Application Insights | Microsoft Docs
description: Exportación continua de datos de Application Insights a mediante el Stream Analytics
ms.topic: conceptual
ms.date: 09/11/2017
ms.openlocfilehash: 9c559a61794b36ea1bc33abc14271151fbea9d4c
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87311235"
---
# <a name="walkthrough-export-to-sql-from-application-insights-using-stream-analytics"></a>Tutorial: exportación a SQL desde Application Insights mediante Stream Analytics
En este artículo se muestra cómo trasladar los datos de telemetría desde [Azure Application Insights][start] a Azure SQL Database mediante la [Exportación continua][export] y [Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics/). 

La Exportación continua traslada los datos de telemetría a Azure Storage en formato JSON. Analizaremos los objetos JSON mediante Azure Stream Analytics y crearemos filas en una tabla de base de datos.

(De manera más general, la Exportación continua es la forma de realizar su propio análisis de la telemetría que las aplicaciones envían a Application Insights. Se puede adaptar este ejemplo de código para realizar otras operaciones con la telemetría exportada, como la adición de datos).

Comenzaremos con la suposición de que ya dispone de la aplicación que desea supervisar.

En este ejemplo, vamos a usar los datos de la vista de página, pero se puede ampliar fácilmente el mismo patrón a otros tipos de datos, como eventos y excepciones personalizados. 

## <a name="add-application-insights-to-your-application"></a>Agregar Application Insights a una aplicación
Primeros pasos:

1. [Configure Application Insights para sus páginas web](./javascript.md). 
   
    (En este ejemplo, nos centraremos en el procesamiento de datos de vistas de página de los exploradores cliente, pero también puede configurar Application Insights para el lado servidor de una aplicación [Java](./java-get-started.md) o [ASP.NET](./asp-net.md) y procesar peticiones, dependencias y otra telemetría de servidor).
2. Publique la aplicación y vea los datos de telemetría que aparecen en el recurso de Application Insights.

## <a name="create-storage-in-azure"></a>Creación de almacenamiento en Azure
La exportación continua siempre envía los datos a una cuenta de Azure Storage, por lo que necesitará crear primero el almacenamiento.

1. Cree una cuenta de almacenamiento en su suscripción en el [portal de Azure][portal].
   
    ![En Azure Portal, elija Nuevo, Datos, Almacenamiento. Seleccione Clásico y elija Crear. Proporcione un nombre de almacenamiento.](./media/code-sample-export-sql-stream-analytics/040-store.png)
2. Crear un contenedor
   
    ![En el nuevo almacenamiento, seleccione Contenedores, haga clic en el icono Contenedores y luego, en Agregar.](./media/code-sample-export-sql-stream-analytics/050-container.png)
3. Copie la clave de acceso de almacenamiento.
   
    La necesitará en seguida para configurar la entrada para el servicio de análisis de transmisiones.
   
    ![En el almacenamiento, abra Configuración, Claves y realice una copia de la clave de acceso principal.](./media/code-sample-export-sql-stream-analytics/21-storage-key.png)

## <a name="start-continuous-export-to-azure-storage"></a>Inicio de la exportación continua al almacenamiento de Azure
1. En el portal de Azure, busque el recurso de Application Insights que ha creado para la aplicación.
   
    ![Elija Examinar, Application Insights, su aplicación.](./media/code-sample-export-sql-stream-analytics/060-browse.png)
2. Cree una exportación continua.
   
    ![Elija Configuración, Exportación continua, Agregar.](./media/code-sample-export-sql-stream-analytics/070-export.png)

    Seleccione la cuenta de almacenamiento que creó anteriormente:

    ![Establezca el destino de exportación.](./media/code-sample-export-sql-stream-analytics/080-add.png)

    Configure los tipos de eventos que desea ver:

    ![Elija los tipos de evento.](./media/code-sample-export-sql-stream-analytics/085-types.png)


1. Permita que se acumulen algunos datos. Póngase cómo y deje que los usuarios usen su aplicación durante un tiempo. Así, aparecerá la telemetría y verá gráficos estadísticos en el [explorador de métricas](../platform/metrics-charts.md) y eventos individuales en la [búsqueda de diagnóstico](./diagnostic-search.md). 
   
    Y, además, exportará los datos en el almacenamiento. 
2. Inspeccione los datos exportados, ya sea en el portal (seleccione **Examinar**, seleccione la cuenta de almacenamiento y, después, **Contenedores**) o bien en Visual Studio. En Visual Studio, elija **Ver/Cloud Explorer** y abra Azure/Almacenamiento. (Si no tiene esta opción de menú, deberá instalar Azure SDK: abra el cuadro de diálogo Nuevo proyecto y Visual C#/Nube/Obtener Microsoft Azure SDK para. NET).
   
    ![En Visual Studio, abra Explorador de servidores, Azure, Almacenamiento.](./media/code-sample-export-sql-stream-analytics/087-explorer.png)
   
    Tome nota de la parte común del nombre de la ruta de acceso, que se deriva del nombre de la aplicación y de la clave de instrumentación. 

Los eventos se escriben en archivos de blob en formato JSON. Cada archivo puede contener uno o varios eventos. Así, es probable que queramos leer los datos de eventos y filtrar por los campos que deseemos. Se pueden realizar multitud de acciones con los datos, pero nuestro plan de hoy consiste en usar Stream Analytics para trasladar los datos a SQL Database. De este modo, será más sencillo ejecutar muchas consultas interesantes.

## <a name="create-an-azure-sql-database"></a>Creación de una base de datos de Azure SQL
De nuevo, empiece desde su suscripción en [Azure Portal][portal], cree la base de datos (y un servidor, a menos que ya tenga uno) donde escribirá los datos.

![Nuevo, Datos, SQL.](./media/code-sample-export-sql-stream-analytics/090-sql.png)

Asegúrese de que el servidor permite el acceso a los servicios de Azure:

![Examinar, Servidores, su servidor, Configuración, Firewall, Permitir acceso a Azure.](./media/code-sample-export-sql-stream-analytics/100-sqlaccess.png)

## <a name="create-a-table-in-azure-sql-database"></a>Creación de una tabla en Azure SQL Database
Conéctese a la base de datos creada en la sección anterior con su herramienta preferida de administración. En este tutorial, usaremos [Herramientas de administración de SQL Server](/sql/ssms/sql-server-management-studio-ssms?view=sql-server-ver15) (SSMS).

![Conexión a Azure SQL Database](./media/code-sample-export-sql-stream-analytics/31-sql-table.png)

Cree una nueva consulta y ejecute la siguiente instrucción T-SQL:

```SQL

CREATE TABLE [dbo].[PageViewsTable](
    [pageName] [nvarchar](max) NOT NULL,
    [viewCount] [int] NOT NULL,
    [url] [nvarchar](max) NULL,
    [urlDataPort] [int] NULL,
    [urlDataprotocol] [nvarchar](50) NULL,
    [urlDataHost] [nvarchar](50) NULL,
    [urlDataBase] [nvarchar](50) NULL,
    [urlDataHashTag] [nvarchar](max) NULL,
    [eventTime] [datetime] NOT NULL,
    [isSynthetic] [nvarchar](50) NULL,
    [deviceId] [nvarchar](50) NULL,
    [deviceType] [nvarchar](50) NULL,
    [os] [nvarchar](50) NULL,
    [osVersion] [nvarchar](50) NULL,
    [locale] [nvarchar](50) NULL,
    [userAgent] [nvarchar](max) NULL,
    [browser] [nvarchar](50) NULL,
    [browserVersion] [nvarchar](50) NULL,
    [screenResolution] [nvarchar](50) NULL,
    [sessionId] [nvarchar](max) NULL,
    [sessionIsFirst] [nvarchar](50) NULL,
    [clientIp] [nvarchar](50) NULL,
    [continent] [nvarchar](50) NULL,
    [country] [nvarchar](50) NULL,
    [province] [nvarchar](50) NULL,
    [city] [nvarchar](50) NULL
)

CREATE CLUSTERED INDEX [pvTblIdx] ON [dbo].[PageViewsTable]
(
    [eventTime] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON)

```

![Crear PageViewsTable](./media/code-sample-export-sql-stream-analytics/34-create-table.png)

En este ejemplo, usamos datos de vistas de página. Para ver los demás datos disponibles, observe el resultado de JSON y consulte el [modelo de exportación de datos](./export-data-model.md).

## <a name="create-an-azure-stream-analytics-instance"></a>Creación de una instancia de Azure Stream Analytics
Desde [Azure Portal](https://portal.azure.com/), seleccione el servicio Azure Stream Analytics y cree un nuevo trabajo de Stream Analytics:

![Configuración de Stream Analytics](./media/code-sample-export-sql-stream-analytics/SA001.png)

![Nuevo trabajo de Stream Analytics](./media/code-sample-export-sql-stream-analytics/SA002.png)

Cuando se cree el nuevo trabajo, seleccione **Ir al recurso**.

![Configuración de Stream Analytics](./media/code-sample-export-sql-stream-analytics/SA003.png)

#### <a name="add-a-new-input"></a>Incorporación de una nueva entrada

![Configuración de Stream Analytics](./media/code-sample-export-sql-stream-analytics/SA004.png)

Configúrelo de manera que tome los datos del blob de Exportación continua:

![Configuración de Stream Analytics](./media/code-sample-export-sql-stream-analytics/SA0005.png)

Ahora, necesitará la clave de acceso principal de la cuenta de almacenamiento, que anotó anteriormente. Establézcala como la clave de cuenta de almacenamiento.

#### <a name="set-path-prefix-pattern"></a>Establecimiento del patrón del prefijo de la ruta de acceso

**Asegúrese de establecer el formato de fecha como AAAA-MM-DD (con guiones).**

El patrón del prefijo de la ruta de acceso especifica cómo busca el Stream Analytics los archivos de entrada en el almacenamiento. Deberá establecerlo para que coincida con el modo en que la Exportación continua almacena los datos. Configúrelo como este caso que se muestra a continuación:

```sql
webapplication27_12345678123412341234123456789abcdef0/PageViews/{date}/{time}
```

En este ejemplo:

* `webapplication27` es el nombre del recurso de Application Insights, **todo en minúsculas**. 
* `1234...` es la clave de instrumentación del recurso de Application Insights **con los guiones quitados**. 
* `PageViews` es el tipo de datos que se van a analizar. Los tipos disponibles dependen del filtro definido en la Exportación continua. Examine los datos exportados para ver los demás tipos disponibles y vea el [modelo de exportación de datos](./export-data-model.md).
* `/{date}/{time}` es un patrón escrito literalmente.

Para obtener el nombre y el valor iKey del recurso de Application Insights, abra Essentials en su página de información general o bien abra la Configuración.

> [!TIP]
> Use la función Sample para comprobar que estableció correctamente la ruta de acceso de entrada. Si se produce un error: compruebe que hay datos en el almacenamiento para el intervalo de tiempo de muestreo que eligió. Modifique la definición de entrada y compruebe que establece correctamente la cuenta de almacenamiento, el prefijo de ruta de acceso y el formato de fecha.

 
## <a name="set-query"></a>Establecimiento de la consulta
Abra la sección de consulta:

Reemplace la consulta predeterminada por lo siguiente:

```SQL

    SELECT flat.ArrayValue.name as pageName
    , flat.ArrayValue.count as viewCount
    , flat.ArrayValue.url as url
    , flat.ArrayValue.urlData.port as urlDataPort
    , flat.ArrayValue.urlData.protocol as urlDataprotocol
    , flat.ArrayValue.urlData.host as urlDataHost
    , flat.ArrayValue.urlData.base as urlDataBase
    , flat.ArrayValue.urlData.hashTag as urlDataHashTag
      ,A.context.data.eventTime as eventTime
      ,A.context.data.isSynthetic as isSynthetic
      ,A.context.device.id as deviceId
      ,A.context.device.type as deviceType
      ,A.context.device.os as os
      ,A.context.device.osVersion as osVersion
      ,A.context.device.locale as locale
      ,A.context.device.userAgent as userAgent
      ,A.context.device.browser as browser
      ,A.context.device.browserVersion as browserVersion
      ,A.context.device.screenResolution.value as screenResolution
      ,A.context.session.id as sessionId
      ,A.context.session.isFirst as sessionIsFirst
      ,A.context.location.clientip as clientIp
      ,A.context.location.continent as continent
      ,A.context.location.country as country
      ,A.context.location.province as province
      ,A.context.location.city as city
    INTO
      AIOutput
    FROM AIinput A
    CROSS APPLY GetElements(A.[view]) as flat


```

Observe que las primeras propiedades son específicas de los datos de la vista de página. Las exportaciones de otros tipos de telemetría tendrán diferentes propiedades. Vea la [referencia detallada del modelo de datos para los tipos y valores de propiedad](./export-data-model.md)

## <a name="set-up-output-to-database"></a>Configuración de la salida a la base de datos
Seleccione SQL como salida.

![En Análisis de transmisiones, seleccione Salidas.](./media/code-sample-export-sql-stream-analytics/SA006.png)

Especifique la base de datos.

![Rellene los detalles de la base de datos.](./media/code-sample-export-sql-stream-analytics/SA007.png)

Cierre el asistente y espere una notificación que indica que se ha configurado la salida.

## <a name="start-processing"></a>Inicio del procesamiento
Inicie el trabajo desde la barra de acción:

![En Stream Analytics, haga clic en Inicio.](./media/code-sample-export-sql-stream-analytics/SA008.png)

Puede elegir si desea iniciar el procesamiento de los datos a partir de ahora o bien empezar con datos anteriores. Lo segundo resulta útil si la Exportación continua ya se ha estado ejecutando durante un tiempo.

Después de unos minutos, vuelva a las herramientas de administración de SQL Server y consulte los datos que están entrando. Por ejemplo, utilice una consulta como esta:

```sql
SELECT TOP 100 *
FROM [dbo].[PageViewsTable]
```

## <a name="related-articles"></a>Artículos relacionados
* [Exportación a PowerBI mediante Stream Analytics](./export-power-bi.md)
* [Referencia detallada del modelo de datos para los tipos y valores de propiedad](./export-data-model.md)
* [Exportación continua en Application Insights](./export-telemetry.md)
* [Application Insights](https://azure.microsoft.com/services/application-insights/)

<!--Link references-->

[diagnostic]: ./diagnostic-search.md
[export]: ./export-telemetry.md
[metrics]: ../platform/metrics-charts.md
[portal]: https://portal.azure.com/
[start]: ./app-insights-overview.md

