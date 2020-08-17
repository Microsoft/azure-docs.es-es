---
title: Conexión a Azure Cosmos DB mediante herramientas de análisis de BI
description: Obtenga información sobre cómo usar el controlador ODBC de Azure Cosmos DB para crear tablas y vistas de forma que los datos normalizados puedan verse en el software de análisis de datos y BI.
author: SnehaGunda
ms.service: cosmos-db
ms.topic: how-to
ms.date: 10/02/2019
ms.author: sngun
ms.openlocfilehash: 1bda235e5f3f867762457d0dc8214bbadc88059e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87084830"
---
# <a name="connect-to-azure-cosmos-db-using-bi-analytics-tools-with-the-odbc-driver"></a>Conexión a Azure Cosmos DB mediante herramientas de análisis de BI con el controlador ODBC

El controlador ODBC de Azure Cosmos DB le permite conectarse a Azure Cosmos DB mediante herramientas de análisis de BI, como SQL Server Integration Services, Power BI Desktop y Tableau de forma que pueda analizar y crear visualizaciones de los datos de Azure Cosmos DB en esas soluciones.

El controlador ODBC de Azure Cosmos DB es compatible con ODBC 3.8 y admite la sintaxis de ANSI SQL-92. El controlador ofrece características enriquecidas que le ayudarán a volver a normalizar datos en Azure Cosmos DB. Mediante el controlador, puede representar datos en Azure Cosmos DB como tablas y vistas. El controlador permite realizar operaciones de SQL en las tablas y vistas, así como agrupar por consultas, inserciones, actualizaciones y eliminaciones.

> [!NOTE]
> La conexión a Azure Cosmos DB con el controlador ODBC actualmente solo se admite para cuentas de API de SQL de Azure Cosmos DB.

## <a name="why-do-i-need-to-normalize-my-data"></a>¿Por qué es necesario normalizar los datos?
Azure Cosmos DB es una base de datos sin esquema, lo que permite el desarrollo rápido de aplicaciones y la capacidad de iterar en los modelos de datos sin que se limite a un esquema estricto. Una sola base de datos de Azure Cosmos puede contener documentos JSON de varias estructuras. Esto es ideal para el desarrollo rápido de aplicaciones, pero cuando desea analizar y crear informes de los datos mediante análisis de datos y herramientas de BI, suele ser normal eliminar el formato de los datos y que estos se ajusten a un esquema específico.

Aquí es donde entra en juego el controlador ODBC. Mediante el controlador ODBC, ahora puede volver a normalizar datos de Azure Cosmos DB en tablas y vistas que se ajusten a sus necesidades de informes y análisis de datos. Los esquemas que ese han vuelto a normalizar no tienen ningún efecto en los datos subyacentes y no obligan a los desarrolladores a ajustarse a ellos. En su lugar, le permiten aprovechar las herramientas compatibles con ODBC para acceder a los datos. Por tanto, la base de datos de Azure Cosmos no solo será ahora la favorita del equipo de desarrollo, sino que a los analistas de datos también les encantará.

Empecemos con el controlador ODBC.

## <a name="step-1-install-the-azure-cosmos-db-odbc-driver"></a><a id="install"></a>Paso 1: Instalación del controlador ODBC de Azure Cosmos DB

1. Descargue los controladores para su entorno:

    | Instalador | Sistemas operativos admitidos| 
    |---|---| 
    |[Microsoft Azure Cosmos DB ODBC 64-bit.msi](https://aka.ms/cosmos-odbc-64x64) para Windows de 64 bits| Versiones de 64 bits de Windows 8.1 o posterior, Windows 8, Windows 7, Windows Server 2012 R2, Windows Server 2012 y Windows Server 2008 R2.| 
    |[Microsoft Azure Cosmos DB ODBC 32x64-bit.msi](https://aka.ms/cosmos-odbc-32x64) para Windows de 32 bits en 64 bits| Versiones de 64 bits de Windows 8.1 o posterior, Windows 8, Windows 7, Windows XP, Windows Vista, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 y Windows Server 2003.| 
    |[Microsoft Azure Cosmos DB ODBC 32-bit.msi](https://aka.ms/cosmos-odbc-32x32) para Windows de 32 bits|Versiones de 32 bits de Windows 8.1 o posterior, Windows 8, Windows 7, Windows XP y Windows Vista.|

    Ejecute el archivo msi localmente, lo que iniciará el **Asistente para instalación del controlador ODBC de Microsoft Azure Cosmos DB**. 

1. Complete el asistente para instalación con la entrada predeterminada para instalar el controlador ODBC.

1. Abra la aplicación **Administrador de orígenes de datos ODBC** en su equipo. Para ello, puede escribir **orígenes de datos ODBC** en el cuadro de búsqueda de Windows. 
    Puede confirmar que el controlador se ha instalado haciendo clic en la pestaña **Controladores** y asegurándose de que **el controlador ODBC de Microsoft Azure Cosmos DB** aparece en la lista.

    :::image type="content" source="./media/odbc-driver/odbc-driver.png" alt-text="Administrador de orígenes de datos ODBC de Azure Cosmos DB":::

## <a name="step-2-connect-to-your-azure-cosmos-database"></a><a id="connect"></a>Paso 2: Conexión a la base de datos de Azure Cosmos

1. Después de [instalar el controlador ODBC de Azure Cosmos DB](#install), en la ventana **Administrador de orígenes de datos ODBC**, haga clic en **Agregar**. Puede crear un DSN de usuario o de sistema. En este ejemplo, va a crear un DSN de usuario.

1. En la ventana **Crear un nuevo origen de datos**, seleccione **Microsoft Azure Cosmos DB ODBC Driver** (Controlador ODBC de Microsoft Azure Cosmos DB) y haga clic en **Finalizar**.

1. En la ventana **Azure Cosmos DB ODBC Driver DSN Setup** (Configuración de DSN del controlador ODBC de Azure Cosmos DB), rellene la información siguiente: 

    :::image type="content" source="./media/odbc-driver/odbc-driver-dsn-setup.png" alt-text="Ventana Azure Cosmos DB ODBC Driver DSN Setup (Configuración de DSN del controlador ODBC de Azure Cosmos DB)":::
    - **Nombre del origen de datos**: el propio nombre descriptivo para el DSN de ODBC. Este nombre es único en su cuenta de Azure Cosmos DB, por lo que debe ser apropiado si tiene varias cuentas.
    - **Descripción**: Una breve descripción del origen de datos.
    - **Host**: URI de la cuenta de Azure Cosmos DB. Puede recuperar esta información de la página Claves de Azure Cosmos DB en Azure Portal, como se muestra en la captura de pantalla siguiente. 
    - **Clave de acceso**: clave principal o secundaria, de solo escritura o solo lectura de la página Claves de Azure Cosmos DB de Azure Portal, como se muestra en la captura de pantalla siguiente. Se recomienda usar la clave de solo lectura si DSN se utiliza para procesamiento e informes de datos de solo lectura.
    :::image type="content" source="./media/odbc-driver/odbc-cosmos-account-keys.png" alt-text="Página Claves de Azure Cosmos DB":::
    - **Encrypt Access Key for** (Cifrar la clave de acceso para): seleccione la mejor opción en función de los usuarios de esta máquina. 
    
1. Haga clic en el botón **Probar** para asegurarse de que puede conectarse a su cuenta de Azure Cosmos DB. 

1.  Haga clic en **Opciones avanzadas** y establezca los siguientes valores:
    *  **Versión de API REST**: Seleccione la [versión de la API REST](/rest/api/cosmos-db/) de las operaciones. El valor predeterminado es 2015-12-16. Si tiene contenedores con [claves de partición grandes](large-partition-keys.md) y necesita la versión 2018-12-31 de la API REST:
        - Escriba **2018-12-31** como versión de la API REST.
        - En el menú **Inicio**, escriba "regedit" para buscar y abrir la aplicación **Editor del Registro**.
        - En el Editor del Registro, vaya a la siguiente ruta: **Computer\HKEY_LOCAL_MACHINE\SOFTWARE\ODBC\ODBC.INI**
        - Cree una nueva subclave con el mismo nombre que el DSN; por ejemplo, "Contoso Account ODBC DSN".
        - Vaya a la subclave "Contoso Account ODBC DSN".
        - Haga clic con el botón derecho para agregar un nuevo **valor de cadena**:
            - Nombre del valor: **IgnoreSessionToken**
            - Datos del valor: **1**
            :::image type="content" source="./media/odbc-driver/cosmos-odbc-edit-registry.png" alt-text="Configuración del Editor del Registro":::
    - **Coherencia de consultas**: seleccione el [nivel de coherencia](consistency-levels.md) para las operaciones. El valor predeterminado es Sesión.
    - **Número de reintentos**: escriba el número de veces que se volverá a intentar una operación si la solicitud inicial no se completa debido a una limitación de la velocidad del servicio.
    - **Archivo de esquema**: Tiene varias opciones.
        - De forma predeterminada, si esta entrada se deja como está (en blanco), el controlador buscará todos los contenedores de la primera página para determinar el esquema de cada contenedor. Esto se conoce como "asignación de contenedores". Sin un archivo de esquema definido, el controlador tiene que realizar el examen para cada sesión de controlador y podría dar lugar a un tiempo de inicio mayor de una aplicación con DSN. Se recomienda asociar siempre un archivo de esquema para un DSN.
        - Si ya tiene un archivo de esquema (posiblemente uno que creó mediante el Editor de esquemas), puede hacer clic en **Examinar**, navegar al archivo, hacer clic en **Guardar** y, luego, hacer clic en **Aceptar**.
        - Si desea crear un nuevo esquema, haga clic en **Aceptar**y luego en **Editor de esquemas** en la ventana principal. Después, continúe con la información del Editor de esquemas. Tras crear el nuevo archivo de esquema, recuerde que debe volver a la ventana **Opciones avanzadas** para incluir el archivo de esquema recién creado.

1. Una vez que complete y cierre la ventana **Azure Cosmos DB ODBC Driver DSN Setup** (Configuración de DSN del controlador ODBC de Azure Cosmos DB), el nuevo DSN de usuario se agregará a la pestaña DSN de usuario.

    :::image type="content" source="./media/odbc-driver/odbc-driver-user-dsn.png" alt-text="Nuevo DSN de ODBC de Azure Cosmos DB en la pestaña DSN de usuario":::

## <a name="step-3-create-a-schema-definition-using-the-container-mapping-method"></a><a id="#container-mapping"></a>Paso 3: Creación de una definición de esquema con el método de asignación de contenedores

Hay dos tipos de métodos de muestreo que puede usar: **asignación de contenedores** o **delimitadores de tabla**. En una sesión de muestreo, se pueden utilizar los dos métodos, pero solo puede usarse uno de ellos en cada contenedor. En el procedimiento siguiente, se crea un esquema para los datos de uno o varios contenedores mediante el método de asignación de contenedores. Este método de muestreo recupera los datos de la página de un contenedor para determinar la estructura de los datos. Transpone un contenedor a una tabla del lado ODBC. Este método de muestreo es rápido y eficaz cuando los datos de un contenedor son homogéneos. Si los datos de un contenedor son heterogéneos, es recomendable usar el [método de asignación de delimitadores de tabla](#table-mapping), ya que proporciona un método de muestreo más robusto para determinar las estructuras de datos del contenedor. 

1. Después de completar los pasos 1 a 4 de [Conexión a la base de datos de Azure Cosmos](#connect), haga clic en **Editor de esquemas** en la ventana **Azure Cosmos DB ODBC Driver DSN Setup** (Configuración de DSN del controlador ODBC de Azure Cosmos DB).

    :::image type="content" source="./media/odbc-driver/odbc-driver-schema-editor.png" alt-text="Botón Editor de esquemas en la ventana Azure Cosmos DB ODBC Driver DSN Setup (Configuración de DSN del controlador ODBC de Azure Cosmos DB)":::
1. En la ventana **Editor de esquemas**, haga clic en **Crear nuevo**.
    En la ventana **Generate Schema** (Generar esquema), se muestran todos los contenedores de la cuenta de Azure Cosmos DB. 

1. Seleccione uno o varios contenedores para realizar un muestreo y haga clic en **Muestra**. 

1. En la pestaña **Vista de diseño**, se representan la base de datos, el esquema y la tabla. En la vista de tabla, el examen muestra el conjunto de propiedades asociadas a los nombres de columna (Nombre SQL, Nombre de origen, etc.).
    Para cada columna, puede modificar el nombre SQL de la columna, el tipo de SQL, la longitud SQL (si procede), la escala (si procede), la precisión (si procede) y si acepta valores NULL.
    - Puede establecer **Ocultar columna** en **true** si desea excluir esa columna de los resultados de la consulta. Las columnas marcadas con Ocultar columna = true no se devuelven para la selección y proyección, aunque aún forman parte del esquema. Por ejemplo, puede ocultar todas las propiedades necesarias del sistema de Azure Cosmos DB a partir de "_".
    - La columna **identificador** es el único campo que no se puede ocultar, ya que se usa como la clave principal en el esquema normalizado. 

1. Cuando haya terminado de definir el esquema, haga clic en **Archivo** | **Guardar**, navegue hasta el directorio para guardar el esquema y, luego, haga clic en **Guardar**.

1. Para utilizar este esquema con un DSN, abra la **ventana de Configuración de DSN del controlador ODBC de Azure Cosmos DB** (con el Administrador de orígenes de datos ODBC), haga clic en **Opciones avanzadas** y, luego, en el cuadro **Archivo de esquema**, desplácese al esquema guardado. Al guardar un archivo de esquema en un DSN existente se modifica la conexión de DSN para tener como ámbito los datos y la estructura definidos por el esquema.

## <a name="step-4-create-a-schema-definition-using-the-table-delimiters-mapping-method"></a><a id="table-mapping"></a>Paso 4: Creación de una definición de esquema mediante el método de asignación de delimitadores de tabla

Hay dos tipos de métodos de muestreo que puede usar: **asignación de contenedores** o **delimitadores de tabla**. En una sesión de muestreo, se pueden utilizar los dos métodos, pero solo puede usarse uno de ellos en cada contenedor. 

Los pasos siguientes crean un esquema de los datos de uno o varios contenedores utilizando el método de asignación de **delimitadores de tabla**. Es recomendable que utilice este método de muestreo cuando los contenedores tengan tipos de datos heterogéneos. Puede usar este método para limitar el muestreo a un conjunto de atributos y sus valores correspondientes. Por ejemplo, si un documento contiene una propiedad "Type", puede limitar el muestreo a los valores de esta propiedad. El resultado final del muestreo sería un conjunto de tablas para cada uno de los valores para el tipo especificado. Por ejemplo, Tipo = Vehículo generará una tabla de vehículos, mientras que Tipo = Plano generaría una tabla de planos.

1. Después de completar los pasos 1 a 4 de [Conexión a la base de datos de Azure Cosmos](#connect), haga clic en **Editor de esquemas** en la ventana Azure Cosmos DB ODBC Driver DSN Setup (Configuración de DSN del controlador ODBC de Azure Cosmos DB).

1. En la ventana **Editor de esquemas**, haga clic en **Crear nuevo**.
    En la ventana **Generate Schema** (Generar esquema), se muestran todos los contenedores de la cuenta de Azure Cosmos DB. 

1. Seleccione un contenedor en la pestaña **Vista de ejemplo**, en la columna **Mapping Definition** (Definición de asignación) del contenedor, y haga clic en **Editar**. A continuación, en la ventana **Mapping Definition** (Definición de asignación), seleccione el método **Table Delimiters** (Delimitadores de tabla). A continuación, haga lo siguiente:

    a. En el cuadro **Atributos**, escriba el nombre de una propiedad de delimitador. Se trata de una propiedad del documento a la que desea limitar el ámbito del muestreo, por ejemplo, Ciudad; luego, presione Intro. 

    b. Si solo desea definir el ámbito del muestreo a ciertos valores para el atributo que escribió antes, seleccione dicho atributo en el cuadro de selección, escriba un valor en el cuadro **Valor** (por ejemplo, Seattle) y presione Entrar. Puede seguir agregando más valores para los atributos. Solo tiene que asegurarse de que se selecciona el atributo correcto cuando está escribiendo valores.

    Por ejemplo, si incluye Ciudad como un valor de **Atributos** y desea limitar la tabla para incluir solo filas con Nueva York y Dubai como valores de ciudad, tendría que escribir Ciudad en el cuadro Atributos, y Nueva York y luego Dubái en el cuadro **Valores**.

1. Haga clic en **OK**. 

1. Después de completar las definiciones de asignación de los contenedores que desee incluir en el muestreo, en la ventana **Editor de esquemas**, haga clic en **Muestra**.
     Para cada columna, puede modificar el nombre SQL de la columna, el tipo de SQL, la longitud SQL (si procede), la escala (si procede), la precisión (si procede) y si acepta valores NULL.
    - Puede establecer **Ocultar columna** en **true** si desea excluir esa columna de los resultados de la consulta. Las columnas marcadas con Ocultar columna = true no se devuelven para la selección y proyección, aunque aún forman parte del esquema. Por ejemplo, puede ocultar todas las propiedades necesarias del sistema de Azure Cosmos DB a partir de `_`.
    - La columna **identificador** es el único campo que no se puede ocultar, ya que se usa como la clave principal en el esquema normalizado. 

1. Cuando haya terminado de definir el esquema, haga clic en **Archivo** | **Guardar**, navegue hasta el directorio para guardar el esquema y, luego, haga clic en **Guardar**.

1. De vuelta a la ventana **Azure Cosmos DB ODBC Driver DSN Setup** (Configuración de DSN del controlador ODBC de Azure Cosmos DB), haga clic en **Opciones avanzadas**. A continuación, en el cuadro **Archivo de esquema** , navegue hasta el archivo de esquema guardado y haga clic en **Aceptar**. Haga clic en **Aceptar** de nuevo para guardar el DSN. Esto guarda el esquema que ha creado en el DSN. 

## <a name="optional-set-up-linked-server-connection"></a>(Opcional) Configuración de la conexión de servidor vinculado

Puede consultar Azure Cosmos DB desde SQL Server Management Studio (SSMS) mediante la configuración de una conexión de servidor vinculado.

1. Cree un origen de datos del sistema, tal como se describe en el [Paso 2](#connect), denominado, por ejemplo, `SDS Name`.

1. [Instale SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) y conéctese al servidor. 

1. En el editor de consultas de SSMS, cree un objeto de servidor vinculado `DEMOCOSMOS` para el origen de datos con los siguientes comandos. Reemplace `DEMOCOSMOS` por el nombre del servidor vinculado, y `SDS Name` por el nombre del origen de datos del sistema.

    ```sql
    USE [master]
    GO
    
    EXEC master.dbo.sp_addlinkedserver @server = N'DEMOCOSMOS', @srvproduct=N'', @provider=N'MSDASQL', @datasrc=N'SDS Name'
    
    EXEC master.dbo.sp_addlinkedsrvlogin @rmtsrvname=N'DEMOCOSMOS', @useself=N'False', @locallogin=NULL, @rmtuser=NULL, @rmtpassword=NULL
    
    GO
    ```
    
Para ver el nuevo nombre del servidor vinculado, actualice la lista de servidores vinculados.

:::image type="content" source="./media/odbc-driver/odbc-driver-linked-server-ssms.png" alt-text="Servidor vinculado en SSMS":::

### <a name="query-linked-database"></a>Consulta de la base de datos vinculada

Para consultar la base de datos vinculada, escriba una consulta SSMS. En este ejemplo, la consulta selecciona los datos de la tabla del contenedor `customers`:

```sql
SELECT * FROM OPENQUERY(DEMOCOSMOS, 'SELECT *  FROM [customers].[customers]')
```

Ejecutar la consulta. El resultado debe ser similar a lo siguiente:

```
attachments/  1507476156    521 Bassett Avenue, Wikieup, Missouri, 5422   "2602bc56-0000-0000-0000-59da42bc0000"   2015-02-06T05:32:32 +05:00 f1ca3044f17149f3bc61f7b9c78a26df
attachments/  1507476156    167 Nassau Street, Tuskahoma, Illinois, 5998   "2602bd56-0000-0000-0000-59da42bc0000"   2015-06-16T08:54:17 +04:00 f75f949ea8de466a9ef2bdb7ce065ac8
attachments/  1507476156    885 Strong Place, Cassel, Montana, 2069       "2602be56-0000-0000-0000-59da42bc0000"   2015-03-20T07:21:47 +04:00 ef0365fb40c04bb6a3ffc4bc77c905fd
attachments/  1507476156    515 Barwell Terrace, Defiance, Tennessee, 6439     "2602c056-0000-0000-0000-59da42bc0000"   2014-10-16T06:49:04 +04:00      e913fe543490432f871bc42019663518
attachments/  1507476156    570 Ruby Street, Spokane, Idaho, 9025       "2602c156-0000-0000-0000-59da42bc0000"   2014-10-30T05:49:33 +04:00 e53072057d314bc9b36c89a8350048f3
```

> [!NOTE]
> El servidor vinculado de Cosmos DB no es compatible con la nomenclatura de cuatro partes. Se devuelve un error similar al siguiente mensaje:

```
Msg 7312, Level 16, State 1, Line 44

Invalid use of schema or catalog for OLE DB provider "MSDASQL" for linked server "DEMOCOSMOS". A four-part name was supplied, but the provider does not expose the necessary interfaces to use a catalog or schema.
``` 

## <a name="optional-creating-views"></a>(Opcional) Creación de vistas
Puede definir y crear vistas como parte del proceso de muestreo. Estas vistas son equivalentes a las vistas SQL. Son de solo lectura y con el ámbito de las selecciones y proyecciones de la consulta SQL de Azure Cosmos DB definida. 

Para crear una vista de los datos, en la ventana **Editor de esquemas**, en la columna **View Definitions** (Definiciones de vista), haga clic en **Agregar** en la fila del contenedor que se va a incluir en el muestreo. 

:::image type="content" source="./media/odbc-driver/odbc-driver-create-view.png" alt-text="Creación de una vista de datos":::


A continuación, en la ventana **View Definitions** (Definiciones de vista), haga lo siguiente:

1. Haga clic en **Nuevo**, escriba un nombre para la vista, por ejemplo, EmployeesfromSeattleView y luego haga clic en **Aceptar**.

1. En la ventana **Editar vista**, escriba una consulta de Azure Cosmos DB. Debe tratarse de una [consulta de Azure Cosmos DB SQL](how-to-sql-query.md), por ejemplo `SELECT c.City, c.EmployeeName, c.Level, c.Age, c.Manager FROM c WHERE c.City = "Seattle"`, y, a continuación, haga clic en **Aceptar**.

    :::image type="content" source="./media/odbc-driver/odbc-driver-create-view-2.png" alt-text="Adición de consultas al crear una vista":::


Puede crear tantas vistas como desee. Una vez que haya terminado de definir las vistas, puede muestrear los datos. 

## <a name="step-5-view-your-data-in-bi-tools-such-as-power-bi-desktop"></a>Paso 5: Visualización de los datos en herramientas de BI como Power BI Desktop

Puede utilizar el nuevo DSN para conectar Azure Cosmos DB con cualquier herramienta compatible con ODBC; este paso simplemente muestra cómo conectarse a Power BI Desktop y crear una visualización de Power BI.

1. Abra Power BI Desktop.

1. Haga clic en **Obtener datos**.

    :::image type="content" source="./media/odbc-driver/odbc-driver-power-bi-get-data.png" alt-text="Obtención de datos en Power BI Desktop":::

1. En la ventana **Obtener datos**, haga clic en **Otros** | **ODBC** | **Conectar**.

    :::image type="content" source="./media/odbc-driver/odbc-driver-power-bi-get-data-2.png" alt-text="Elección de origen de datos ODBC en Obtener datos de Power BI":::

1. En la ventana **Desde ODBC**, seleccione el nombre del origen de datos que ha creado el nombre y luego haga clic en **Aceptar**. Puede dejar las entradas **Opciones avanzadas** en blanco.

   :::image type="content" source="./media/odbc-driver/odbc-driver-power-bi-get-data-3.png" alt-text="Elección del nombre del origen de datos (DSN) en Get Data de Power BI":::

1. En la ventana **Acceda a un origen de datos a través de un controlador ODBC**, seleccione **Predeterminado o personalizado** y luego haga clic en **Conectar**. No es necesario incluir **Propiedades de la cadena de conexión de credenciales**.

1. En la ventana **Navegador**, en el panel izquierdo, expanda la base de datos, el esquema y luego seleccione la tabla. El panel de resultados incluye los datos que usan el esquema que ha creado.

    :::image type="content" source="./media/odbc-driver/odbc-driver-power-bi-get-data-4.png" alt-text="Selección de la tabla en Obtener datos de Power BI":::

1. Para visualizar los datos en Power BI Desktop, active la casilla junto al nombre de tabla y luego haga clic en **Cargar**.

1. En Power BI Desktop, en el extremo izquierdo, seleccione la pestaña Datos :::image type="icon" source="./media/odbc-driver/odbc-driver-data-tab.png"::: para confirmar que los datos se importaron. 

1. Ahora puede crear objetos visuales con Power BI haciendo clic en la pestaña Informe :::image type="icon" source="./media/odbc-driver/odbc-driver-report-tab.png":::, haciendo clic en **Nuevo objeto visual** y, por último, personalizando el icono. Para más información sobre cómo crear visualizaciones en Power BI Desktop, consulte [Tipos de visualización en Power BI](https://powerbi.microsoft.com/documentation/powerbi-service-visualization-types-for-reports-and-q-and-a/). 

## <a name="troubleshooting"></a>Solución de problemas

Si recibe el siguiente error, asegúrese de que los valores de **Host** y **Clave de acceso** copiados de Azure Portal en el [paso 2](#connect) son correctos y luego vuelva a intentarlo. Utilice los botones de copia situados a la derecha de los valores de **Host** y **Clave de acceso** en Azure Portal para copiar los valores sin errores.

```output
[HY000]: [Microsoft][Azure Cosmos DB] (401) HTTP 401 Authentication Error: {"code":"Unauthorized","message":"The input authorization token can't serve the request. Please check that the expected payload is built as per the protocol, and check the key being used. Server used the following payload to sign: 'get\ndbs\n\nfri, 20 jan 2017 03:43:55 gmt\n\n'\r\nActivityId: 9acb3c0d-cb31-4b78-ac0a-413c8d33e373"}
```


## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Cosmos DB, consulte [Bienvenido a Azure Cosmos DB](introduction.md).
