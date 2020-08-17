---
title: Uso de la fuente de cambios de Azure Cosmos DB para visualizar análisis de datos en tiempo real
description: En este artículo se describe cómo una empresa minorista puede utilizar la fuente de cambios para reconocer patrones de usuario, realizar análisis de datos en tiempo real y visualizarlos.
author: SnehaGunda
ms.service: cosmos-db
ms.devlang: java
ms.topic: how-to
ms.date: 05/28/2019
ms.author: sngun
ms.custom: devx-track-java
ms.openlocfilehash: c9abc4dc89651eec7df635fb415314b2c12da3a6
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87319769"
---
# <a name="use-azure-cosmos-db-change-feed-to-visualize-real-time-data-analytics"></a>Uso de la fuente de cambios de Azure Cosmos DB para visualizar análisis de datos en tiempo real

La fuente de cambios de Azure Cosmos es un mecanismo para obtener una fuente continua e incremental de los registros de un contenedor de Azure Cosmos DB a medida que esos registros se crean o modifican. La compatibilidad con la fuente de cambios funciona escuchando al contenedor para detectar los cambios. A continuación, muestra la lista ordenada de los documentos que han cambiado en el orden en el que se modificaron. Para más información sobre la fuente de cambios, vea el artículo [Compatibilidad con la fuente de cambios en Azure Cosmos DB](change-feed.md). 

En este artículo se describe cómo una empresa de comercio electrónico puede utilizar la fuente de cambios para reconocer patrones de usuario, realizar análisis de datos en tiempo real y obtener su visualización. Analizará eventos tales como que un usuario vea un artículo, agregue un artículo al carro o compre un artículo. Cuando se produce uno de estos eventos, se crea un nuevo registro que la fuente de cambios registra. La fuente de cambios desencadena entonces una serie de pasos que se traducen en la visualización de métricas que analizan el rendimiento y la actividad de la empresa. Las métricas de ejemplo que se pueden visualizar incluyen los ingresos, los visitantes únicos del sitio, los artículos más populares y el precio medio de los artículos que se ven en comparación con los que se agregan al carro y con los que se compran. Estas métricas de ejemplo pueden ayudar a una empresa de comercio electrónico a evaluar la popularidad de su sitio, desarrollar sus estrategias de publicidad y precios y tomar decisiones sobre el inventario en el que invertir.

Si está interesado en ver un vídeo sobre la solución antes de empezar, vea el siguiente vídeo:

> [!VIDEO https://www.youtube.com/embed/AYOiMkvxlzo]
>

## <a name="solution-components"></a>Componentes de soluciones
El siguiente diagrama representa el flujo de datos y los componentes implicados en la solución:

:::image type="content" source="./media/changefeed-ecommerce-solution/project-visual.png" alt-text="Elemento visual de proyecto" border="false":::
 
1. **Generación de datos:** se utiliza un simulador de datos para generar datos comerciales que representan eventos tales como que un usuario vea un artículo, agregue un artículo al carro y compre un artículo. Puede generar un conjunto grande de datos de ejemplo mediante el generador de datos. Los datos de ejemplo generados contienen documentos con el formato siguiente:
   
   ```json
   {      
     "CartID": 2486,
     "Action": "Viewed",
     "Item": "Women's Denim Jacket",
     "Price": 31.99
   }
   ```

2. **Cosmos DB:** Los datos generados se almacenan en un contenedor de Azure Cosmos.  

3. **Fuente de cambios:** la fuente de cambios escuchará para detectar cambios en el contenedor de Azure Cosmos. Cada vez que se agregue un nuevo documento a la colección (es decir, cuando se produce un evento tal como que un usuario vea un artículo, agregue un artículo al carro o compre un artículo), la fuente de cambios desencadenará una [función de Azure](../azure-functions/functions-overview.md).  

4. **Función de Azure:** procesa los nuevos datos y los envía a un [Centro de eventos de Azure](../event-hubs/event-hubs-about.md).  

5. **Centro de eventos:** el Centro de eventos de Azure almacena estos eventos y los envía a [Azure Stream Analytics](../stream-analytics/stream-analytics-introduction.md) para realizar análisis adicionales.  

6. **Azure Stream Analytics:** Azure Stream Analytics define las consultas para procesar los eventos y realizar análisis de datos en tiempo real. Estos datos se envían luego a [Microsoft Power BI](https://docs.microsoft.com/power-bi/desktop-what-is-desktop).  

7. **Power BI:** Power BI se usa para visualizar los datos enviados por Azure Stream Analytics. Puede crear un panel para ver cómo cambian las métricas en tiempo real.  

## <a name="prerequisites"></a>Requisitos previos

* Microsoft .NET Framework 4.7.1 o superior

* Microsoft .NET Core 2.1 (o superior)

* Visual Studio con cargas de trabajo de desarrollo de la Plataforma universal de Windows, desarrollo de escritorio de .NET y desarrollo de ASP.NET y web

* Suscripción de Microsoft Azure

* Cuenta de Microsoft Power BI

* Descargue el [laboratorio de fuente de cambios de Azure Cosmos DB](https://github.com/Azure-Samples/azure-cosmos-db-change-feed-dotnet-retail-sample) desde GitHub. 

## <a name="create-azure-resources"></a>Creación de recursos de Azure 

Cree los recursos de Azure (cuenta de almacenamiento de Azure Cosmos DB, Centro de eventos, Stream Analytics) que la solución requiere. Implementará estos recursos mediante una plantilla de Azure Resource Manager. Siga estos pasos para implementar los recursos: 

1. Establezca la directiva de ejecución de Windows PowerShell en **Sin restricciones**. Para ello, abra **PowerShell como administrador** y ejecute los siguientes comandos:

   ```powershell
   Get-ExecutionPolicy
   Set-ExecutionPolicy Unrestricted 
   ```

2. En el repositorio de GitHub que descargó en el paso anterior, navegue a la carpeta **Azure Resource Manager** y abra el archivo denominado **parameters.json**.  

3. Especifique valores para los parámetros cosmosdbaccount_name, eventhubnamespace_name y storageaccount_name, tal y como se indica en el archivo **parameters.json**. Más adelante tendrá que usar los nombres que asigne a cada uno de los recursos.  

4. Desde **Windows PowerShell**, navegue hasta la carpeta **Azure Resource Manager** y ejecute el comando siguiente:

   ```powershell
   .\deploy.ps1
   ```
5. Cuando se le solicite, escriba su **id. de suscripción** a Azure, **changefeedlab** como nombre del grupo de recursos y **run1** como nombre de la implementación. Puede que, una vez iniciada, la implementación de recursos tarde hasta 10 minutos en completarse.

## <a name="create-a-database-and-the-collection"></a>Creación de una base de datos y la colección

Ahora creará una colección que contenga los eventos de sitio de comercio electrónico. Cuando un usuario ve un artículo, agrega un artículo al carro o compra un artículo, la colección recibirá un registro que incluye la acción ("visto", "agregado" o "comprado"), el nombre del artículo implicado, el precio del artículo implicado y el número de identificación del carro de usuario implicado.

1. Vaya a [Azure Portal](https://portal.azure.com/) y busque la **cuenta de Azure Cosmos DB** creada por la implementación de la plantilla.  

2. En el panel **Explorador de datos**, seleccione **Nueva colección** y rellene el formulario con los datos siguientes:  

   * En el campo **Id. de base de datos**, seleccione **Crear nuevo** y escriba **changefeedlabdatabase**. Deje la casilla **Provision database throughput** (Aprovisionar rendimiento de la base de datos) sin activar.  
   * En el campo de identificador **Colección**, escriba **changefeedlabcollection**.  
   * En el campo **Clave de partición**, escriba **/Item**. Esta clave distingue mayúsculas de minúsculas, así que asegúrese de escribirlo correctamente.  
   * En el campo **Rendimiento**, escriba **10 000**.  
   * Seleccione el botón **Aceptar**.  

3. Cree entonces otra colección de nombre **leases** para el procesamiento de la fuente de cambios. La colección de concesiones coordina el procesamiento de la fuente de cambios entre varios roles de trabajo. Se utiliza una colección independiente para almacenar las concesiones con una concesión por partición.  

4. Vuelva al panel **Explorador de datos**, seleccione **Nueva colección** y rellene el formulario con los datos siguientes:

   * En el campo **Id. de base de datos**, seleccione **Usar existente** y escriba **changefeedlabdatabase**.  
   * En el campo **Id. de colección**, escriba **leases**.  
   * Como **Capacidad de almacenamiento**, seleccione **Fija**.  
   * Deje el campo **Rendimiento** establecido en su valor predeterminado.  
   * Seleccione el botón **Aceptar**.

## <a name="get-the-connection-string-and-keys"></a>Obtención de la cadena de conexión y las claves

### <a name="get-the-azure-cosmos-db-connection-string"></a>Obtención de la cadena de conexión de Azure Cosmos DB

1. Vaya a [Azure Portal](https://portal.azure.com/) y busque la **cuenta de Azure Cosmos DB** creada por la implementación de la plantilla.  

2. Navegue al panel **Claves**, copie el valor PRIMARY CONNECTION STRING y péguelo en el Bloc de notas u otro documento al que tenga acceso en todo el laboratorio. Debe etiquetarla como **Cadena de conexión de Cosmos DB**. Más adelante tendrá que copiar la cadena en el código, así que anótela y recuerde dónde la almacena.

### <a name="get-the-storage-account-key-and-connection-string"></a>Obtención de la cadena de conexión y la clave de cuenta de almacenamiento

Las cuentas de Azure Storage permiten a los usuarios almacenar datos. En este laboratorio, usará una cuenta de almacenamiento para almacenar los datos que utiliza la función de Azure. La función de Azure se desencadena cuando se realiza ninguna modificación en la colección.

1. Vuelva al grupo de recursos y abra la cuenta de almacenamiento que creó anteriormente  

2. Seleccione **Claves de acceso** en el menú de la izquierda.  

3. Copie los valores bajo **clave 1** en el Bloc de notas u otro documento al que tenga acceso en todo el laboratorio. Debe etiquetar la **Clave** como **Clave de almacenamiento** y la **Cadena de conexión** como **Cadena de conexión de almacenamiento**. Más adelante tendrá que copiar estas cadenas en el código, así que anótelas y recuerde dónde las almacena.  

### <a name="get-the-event-hub-namespace-connection-string"></a>Obtención de la cadena de conexión de Espacio de nombres del centro de eventos

Un Centro de eventos de Azure recibe los datos de eventos y los almacena, procesa y reenvía. En este laboratorio, cada vez que ocurra un nuevo evento (es decir, un artículo es visto por un usuario, agregado al carro del usuario o comprado por un usuario), el Centro de eventos de Azure recibirá un documento que luego reenviará a Azure Stream Analytics.

1. Vuelva al grupo de recursos y abra el **Espacio de nombres del centro de eventos** que creó y denominó en el prelaboratorio.  

2. Seleccione **Directivas de acceso compartido** en el menú de la izquierda.  

3. Seleccione **RootManageSharedAccessKey**. Copie el valor de **Cadena de conexión: clave principal** en el Bloc de notas u otro documento al que tenga acceso durante todo el laboratorio. Debe etiquetarla como cadena de conexión de **Espacio de nombres del centro de eventos**. Más adelante tendrá que copiar la cadena en el código, así que anótela y recuerde dónde la almacena.

## <a name="set-up-azure-function-to-read-the-change-feed"></a>Configuración de la función de Azure para leer la fuente de cambios

Cuando se crea un documento o se modifica un documento actual en un contenedor de Cosmos, la fuente de cambios agrega automáticamente el documento modificado a su historial de cambios en la colección. Ahora compilará y ejecutará una función de Azure que procese la fuente de cambios. Cuando se cree o modifique un documento en la colección que ha creado, la fuente de cambios desencadenará la función de Azure. La función de Azure enviará el documento modificado al Centro de eventos.

1. Vuelva al repositorio que clonó en el dispositivo.  

2. Haga clic en el archivo de nombre **ChangeFeedLabSolution.sln** y seleccione **Open With Visual Studio** (Abrir con Visual Studio).  

3. En Visual Studio, navegue a **local.settings.json**. Utilice los valores que registró anteriormente para rellenar los espacios en blanco.  

4. Navegue a **ChangeFeedProcessor.cs**. En los parámetros de la función **Ejecutar**, lleve a cabo las siguientes acciones:  

   * Reemplace el texto **YOUR COLLECTION NAME HERE** (EL NOMBRE DE LA COLECCIÓN AQUÍ) por el nombre de la colección. Si ha seguido las instrucciones anteriores, el nombre de la colección es changefeedlabcollection.  
   * Reemplace el texto **YOUR LEASES COLLECTION NAME HERE** (EL NOMBRE DE LA COLECCIÓN DE CONCESIONES AQUÍ) por el nombre de la colección de concesiones. Si ha seguido las instrucciones anteriores, el nombre de la colección de concesiones es **leases**.  
   * En la parte superior de Visual Studio, asegúrese de que el cuadro Proyecto de inicio situado a la izquierda de la flecha verde indica **ChangeFeedFunction**.  
   * Seleccione **Iniciar** en la parte superior de la página para ejecutar el programa  
   * Puede confirmar que la función se ejecuta cuando la aplicación de consola indica "Job host started" (host de trabajo iniciado).

## <a name="insert-data-into-azure-cosmos-db"></a>Inserción de datos en Azure Cosmos DB 

Para ver cómo la fuente de cambios procesa nuevas acciones en un sitio de comercio electrónico, hay que simular los datos que representan a los usuarios que ven los artículos del catálogo de productos, agregan esos artículos a sus carros y compran los artículos de sus carros. Estos datos son arbitrarios y su finalidad es replicar el aspecto que tendrían los datos en un sitio de comercio electrónico.

1. Navegue de nuevo al repositorio en el Explorador de archivos y haga clic con el botón derecho en **ChangeFeedFunction.sln** para volver a abrirlo en una nueva ventana de Visual Studio.  

2. Navegue al archivo **App.config**. En el bloque `<appSettings>`, agregue el punto de conexión y la **PRIMARY KEY** única de la cuenta de Azure Cosmos DB que recuperó anteriormente.  

3. Agregue los nombres **collection** y **database**. (Estos nombres deben ser **changefeedlabcollection** y **changefeedlabdatabase** a menos que elija asignarles otros nombres).

   :::image type="content" source="./media/changefeed-ecommerce-solution/update-connection-string.png" alt-text="Actualización de cadenas de conexión":::
 
4. Guarde los cambios efectuados en todos los archivos editados.  

5. En la parte superior de Visual Studio, asegúrese de que el cuadro **Proyecto de inicio** situado a la izquierda de la flecha verde indica **DataGenerator**. Seleccione **Iniciar** en la parte superior de la página para ejecutar el programa.  
 
6. Espere a que el programa se ejecute. Las estrellas indican que están llegando datos. Mantenga el programa en ejecución: es importante que se recopilen grandes cantidades de datos.  

7. Si va a [Azure Portal](https://portal.azure.com/), luego a la cuenta de Cosmos DB en su grupo de recursos y después a **Data Explorer**, verá los datos aleatorios importados en **changefeedlabcollection**.
 
   :::image type="content" source="./media/changefeed-ecommerce-solution/data-generated-in-portal.png" alt-text="Datos generados en el portal":::

## <a name="set-up-a-stream-analytics-job"></a>Configuración de un trabajo de Stream Analytics

Azure Stream Analytics es un servicio en la nube de procesamiento en tiempo real totalmente administrado. En este laboratorio, se utilizará Stream Analytics para procesar nuevos eventos del Centro de eventos (es decir, cada vez que un artículo se vea, se agregue a un carro o se compre), incorporar esos eventos al análisis de datos en tiempo real y enviarlos a Power BI para su visualización.

1. Desde [Azure Portal](https://portal.azure.com/), vaya al grupo de recursos y luego a **streamjob1** (el trabajo de Stream Analytics que creó en el prelaboratorio).  

2. Seleccione **Entradas** tal y como se muestra a continuación.  

   :::image type="content" source="./media/changefeed-ecommerce-solution/create-input.png" alt-text="Creación de una entrada":::

3. Seleccione **+ Add stream input** (+ Agregar entrada de flujo). Seleccione **Centro de eventos** en el menú desplegable.  

4. Rellene el nuevo formulario de entrada con los siguientes datos:

   * En el campo de alias **Entrada**, escriba **input**.  
   * Seleccione la opción for **Select Event Hub from your subscriptions** (Seleccionar centro de eventos de entre las suscripciones).  
   * Establezca el campo **Suscripción** en su suscripción.  
   * En el campo **Espacio de nombres del centro de eventos**, escriba el nombre del Espacio de nombres del centro de eventos que creó en el prelaboratorio.  
   * En el campo **Nombre del centro de eventos**, seleccione la opción para **Usar existente** y elija **event-hub1** en el menú desplegable.  
   * Deje el campo de nombre **Event Hub policy** (Directiva del centro de eventos) establecido en su valor predeterminado.  
   * Deje **Formato de serialización de eventos**como **JSON**.  
   * Deje el campo **Codificación** establecido en **UTF-8**.  
   * Deje el campo **Tipo de compresión de eventos** establecido en **Ninguno**.  
   * Seleccione el botón **Guardar**.

5. Navegue a la página de trabajo de Stream Analytics y seleccione **Salidas**.  

6. Seleccione **+Agregar**. Seleccione **Power BI** en el menú desplegable.  

7. Para crear una salida de Power BI para visualizar el precio medio, realice las siguientes acciones:

   * En el campo **Alias de salida**, escriba **averagePriceOutput**.  
   * Deje el campo **Área de trabajo de grupo** establecido en **Autorizar conexión para cargar áreas de trabajo**.  
   * En el campo **Nombre del conjunto de datos**, escriba **averagePrice**.  
   * En el campo **Nombre de la tabla**, escriba **averagePrice**.  
   * Seleccione el botón **Autorizar** y siga las instrucciones para autorizar la conexión en Power BI.  
   * Seleccione el botón **Guardar**.  

8. Vuelva a **streamjob1** y seleccione **Editar consulta**.

   :::image type="content" source="./media/changefeed-ecommerce-solution/edit-query.png" alt-text="Editar consulta":::
 
9. Pegue la siguiente consulta en la ventana de consulta. La consulta **AVERAGE PRICE** calcula el precio medio de todos los artículos que ven los usuarios, el precio medio de todos los artículos que se agregan a los carros de los usuarios y el precio medio de todos los artículos que compran los usuarios. Esta métrica puede ayudar a las empresas de comercio electrónico a decidir los precios a los que venden los artículos y el inventario en el que deben invertir. Por ejemplo, si el precio medio de los artículos vistos es mucho mayor que el precio medio de los artículos comprados, una empresa podría optar por agregar artículos menos costosos a su inventario.

   ```sql
   /*AVERAGE PRICE*/      
   SELECT System.TimeStamp AS Time, Action, AVG(Price)  
    INTO averagePriceOutput  
    FROM input  
    GROUP BY Action, TumblingWindow(second,5) 
   ```
10. Seleccione **Guardar** en la esquina superior izquierda.  

11. Vuelva ahora a **streamjob1** y seleccione el botón **Iniciar** situado en la parte superior de la página. Azure Stream Analytics puede tardar unos minutos en iniciarse, pero, finalmente, verá que cambia de "Iniciando" a "En ejecución".

## <a name="connect-to-power-bi"></a>Conexión a Power BI

Power BI es un conjunto de herramientas de análisis de negocios que sirve para analizar datos y compartir conocimientos. Es un buen ejemplo de cómo pueden visualizarse los datos analizados estratégicamente.

1. Inicie sesión en Power BI y navegue a **Mi área de trabajo** abriendo el menú de la izquierda de la página.  

2. Seleccione **+Crear** en la esquina superior derecha y seleccione **Panel** para crear un panel.  

3. Seleccione **+Agregar icono** en la esquina superior derecha.  

4. Seleccione **Datos de transmisión personalizados** y el botón **Siguiente**.  
 
5. Seleccione **averagePrice** en **SUS CONJUNTOS DE DATOS** y elija **Siguiente**.  

6. En el campo **Tipo de visualización**, seleccione **Gráfico de barras agrupadas** en el menú desplegable. En **Eje**, agregue la acción. Omita **Leyenda** sin agregar nada. Luego, en la sección siguiente denominada **Valor**, agregue **avg**. Seleccione **Siguiente**, agregue el título del gráfico y elija **Aplicar**. Debería ver un nuevo gráfico en el panel.  

7. Ahora, si quiere visualizar más métricas, puede volver a **streamjob1** y crear tres salidas más con los siguientes campos.

   a. **Alias de salida:** incomingRevenueOutput, nombre del conjunto de datos: incomingRevenue, nombre de la tabla: incomingRevenue  
   b. **Alias de salida:** top5Output, nombre del conjunto de datos: top5, nombre de la tabla: top5  
   c. **Alias de salida:** uniqueVisitorCountOutput, nombre del conjunto de datos: uniqueVisitorCount, nombre de la tabla: uniqueVisitorCount

   Seleccione **Editar consulta** y pegue las siguientes consultas **por encima** de la que ya escribió.

   ```sql
    /*TOP 5*/
    WITH Counter AS
    (
    SELECT Item, Price, Action, COUNT(*) AS countEvents
    FROM input
    WHERE Action = 'Purchased'
    GROUP BY Item, Price, Action, TumblingWindow(second,30)
    ), 
    top5 AS
    (
    SELECT DISTINCT
    CollectTop(5)  OVER (ORDER BY countEvents) AS topEvent
    FROM Counter
    GROUP BY TumblingWindow(second,30)
    ), 
    arrayselect AS 
    (
    SELECT arrayElement.ArrayValue
    FROM top5
    CROSS APPLY GetArrayElements(top5.topevent) AS arrayElement
    ) 
    SELECT arrayvalue.value.item, arrayvalue.value.price,   arrayvalue.value.countEvents
    INTO top5Output
    FROM arrayselect

    /*REVENUE*/
    SELECT System.TimeStamp AS Time, SUM(Price)
    INTO incomingRevenueOutput
    FROM input
    WHERE Action = 'Purchased'
    GROUP BY TumblingWindow(hour, 1)

    /*UNIQUE VISITORS*/
    SELECT System.TimeStamp AS Time, COUNT(DISTINCT CartID) as uniqueVisitors
    INTO uniqueVisitorCountOutput
    FROM input
    GROUP BY TumblingWindow(second, 5)
   ```
   
   La consulta de TOP 5 calcula los cinco primeros artículos, clasificados por el número de veces que se han comprado. Esta métrica puede ayudar a las empresas de comercio electrónico a evaluar los artículos que son más populares y puede influir en las decisiones de publicidad, precios e inventario de la empresa.

   La consulta REVENUE calcula los ingresos sumando los precios de todos los artículos comprados cada minuto. Esta métrica puede ayudar a las empresas de comercio electrónico a evaluar su rendimiento financiero y también reconocer las horas del día que contribuyen a la mayoría de los ingresos. Esta métrica puede incidir en la estrategia general de la empresa, en particular sobre marketing.

   La consulta UNIQUE VISITORS calcula cuántos visitantes únicos están en el sitio cada 5 segundos detectando los identificadores de carro únicos. Esta métrica puede ayudar a las empresas de comercio electrónico a evaluar la actividad del sitio y elaborar estrategias para conseguir más clientes.

8. Ahora también puede agregar iconos para estos conjuntos de datos.

   * En el caso de Top 5, tendría sentido hacer un gráfico de columnas agrupadas con los artículos como eje y el recuento como valor.  
   * En el caso de Revenue, tendría sentido hacer un gráfico de líneas con el tiempo como eje y el total de los precios como valor. La ventana de tiempo que se muestra debe ser lo más grande posible con el fin de entregar la mayor cantidad de información posible.  
   * En el caso de Unique Visitors, tendría sentido crear una visualización de tarjeta con el número de visitantes únicos como valor.

   Este es el aspecto de un panel de ejemplo con estos gráficos:

   :::image type="content" source="./media/changefeed-ecommerce-solution/visualizations.png" alt-text="Visualizaciones":::

## <a name="optional-visualize-with-an-e-commerce-site"></a>Opcional: Visualización con un sitio de comercio electrónico

Ahora verá cómo puede usar la nueva herramienta de análisis de datos para conectarse con un sitio de comercio electrónico real. Para compilar el sitio de comercio electrónico, utilice una base de datos de Azure Cosmos para almacenar la lista de categorías de producto (Mujer, Hombre, Unisex), el catálogo de productos y una lista de los artículos más populares.

1. Vaya de nuevo a [Azure Portal](https://portal.azure.com/), luego a su **cuenta de Cosmos DB** y después a **Data Explorer**.  

   Agregue dos colecciones en **changefeedlabdatabase** - **products** y **categories** con capacidad de almacenamiento fija.

   Agregue otra colección en **changefeedlabdatabase** denominada **topItems** y **/Item** como la clave de partición.

2. Seleccione la colección **topItems** y, en **Escala y configuración**, establezca **Período de vida** como **30 segundos** para que la colección topItems se actualice cada 30 segundos.

   :::image type="content" source="./media/changefeed-ecommerce-solution/time-to-live.png" alt-text="Período de vida":::

3. Para rellenar la colección **topItems** con los artículos comprados con más frecuencia, navegue a **streamjob1** y agregue una nueva **Salida**. Seleccione **Cosmos DB**.

4. Rellene todos los campos obligatorios tal y como se muestra a continuación.

   :::image type="content" source="./media/changefeed-ecommerce-solution/cosmos-output.png" alt-text="Salida de Cosmos":::
 
5. Si ha agregado la consulta TOP 5 opcional en la parte anterior del laboratorio, continúe con la parte 5a. Si no, siga a la parte 5b.

   5a. En **streamjob1**, seleccione **Editar consulta** y pegue la siguiente consulta en el editor de consultas de Azure Stream Analytics por debajo de la consulta TOP 5, pero por encima del resto de las consultas.

   ```sql
   SELECT arrayvalue.value.item AS Item, arrayvalue.value.price, arrayvalue.value.countEvents
   INTO topItems
   FROM arrayselect
   ```
   5b. En **streamjob1**, seleccione **Editar consulta** y pegue la siguiente consulta en el editor de consultas de Azure Stream Analytics por encima del resto de consultas.

   ```sql
   /*TOP 5*/
   WITH Counter AS
   (
   SELECT Item, Price, Action, COUNT(*) AS countEvents
   FROM input
   WHERE Action = 'Purchased'
   GROUP BY Item, Price, Action, TumblingWindow(second,30)
   ), 
   top5 AS
   (
   SELECT DISTINCT
   CollectTop(5)  OVER (ORDER BY countEvents) AS topEvent
   FROM Counter
   GROUP BY TumblingWindow(second,30)
   ), 
   arrayselect AS 
   (
   SELECT arrayElement.ArrayValue
   FROM top5
   CROSS APPLY GetArrayElements(top5.topevent) AS arrayElement
   ) 
   SELECT arrayvalue.value.item AS Item, arrayvalue.value.price, arrayvalue.value.countEvents
   INTO topItems
   FROM arrayselect
   ```

6. Abra **EcommerceWebApp.sln** y navegue al archivo **Web.config** en el **Explorador de soluciones**.  

7. En el bloque `<appSettings>`, agregue el **URI** y la **PRIMARY KEY** que guardó anteriormente donde indica **your URI here** (aquí su URI) y **your primary key here** (aquí su clave principal). Agregue el **nombre de la base de datos** y el **nombre de la colección** tal como se indica. (Estos nombres deben ser **changefeedlabdatabase** y **changefeedlabcollection** a menos que decida asignarles otro nombre).

   Rellene el **nombre de la colección de productos**, el **nombre de la colección de categorías** y el **nombre de la colección de elementos superiores** tal como se indica. (Estos nombres deben ser **products, categories y topItems** a menos que decida asignarles otro nombre).  

8. Abra la **carpeta de desprotección** en **EcommerceWebApp.sln.** Abra el archivo **Web.config** en esa carpeta.  

9. En el bloque `<appSettings>`, agregue el **URI** y la **PRIMARY KEY** que guardó antes cuando se le indicó. Agregue el **nombre de la base de datos** y el **nombre de la colección** tal como se indica. (Estos nombres deben ser **changefeedlabdatabase** y **changefeedlabcollection** a menos que decida asignarles otro nombre).  

10. Pulse **Iniciar** en la parte superior de la página para ejecutar el programa.  

11. Ahora puede experimentar en el sitio de comercio electrónico. Cuando vea un artículo, agregue un artículo al carro, cambie la cantidad de un artículo del carro o compre un artículo, estos eventos se pasarán mediante la fuente de cambios de Cosmos DB al Centro de eventos, a ASA y luego a Power BI. Se recomienda seguir ejecutando DataGenerator para generar datos de tráfico web significativos y proporcionar un conjunto realista de "Productos novedosos" en el sitio de comercio electrónico.

## <a name="delete-the-resources"></a>Eliminación de los recursos

Para eliminar los recursos creados durante este laboratorio, vaya al grupo de recursos en [Azure Portal](https://portal.azure.com/), luego seleccione **Eliminar grupo de recursos** en el menú de la parte superior de la página y siga las instrucciones que se ofrecen.

## <a name="next-steps"></a>Pasos siguientes 
  
* Para más información sobre la fuente de cambios, vea [Compatibilidad con la fuente de cambios en Azure Cosmos DB](change-feed.md). 
