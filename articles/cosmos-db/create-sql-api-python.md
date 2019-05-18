---
title: 'Azure Cosmos DB: Compilación de una aplicación con Python y SQL API'
description: En este tema se presenta un código de ejemplo de Python que se puede usar para conectarse a SQL API de Azure Cosmos DB y realizar consultas.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: python
ms.topic: quickstart
ms.date: 09/24/2018
ms.author: sngun
ms.openlocfilehash: 91a5bf71983511f58fc13a0edf4a1109c3829602
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65410677"
---
# <a name="azure-cosmos-db-build-a-python-application-using-azure-cosmos-db-sql-api-account"></a>Azure Cosmos DB: Compilación de una aplicación Python mediante una cuenta de SQL API de Azure Cosmos DB

> [!div class="op_single_selector"]
> * [.NET](create-sql-api-dotnet.md)
> * [.NET (versión preliminar)](create-sql-api-dotnet-preview.md)
> * [Java](create-sql-api-java.md)
> * [Node.js](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Xamarin](create-sql-api-xamarin-dotnet.md)
>  

Azure Cosmos DB es un servicio de base de datos con varios modelos y de distribución global de Microsoft. Puede crear y consultar documentos, claves y valores y bases de datos de grafos rápidamente. Todas estas operaciones se benefician de la distribución y la escalabilidad de Azure Cosmos DB.

En esta guía de inicio rápido se muestra cómo crear una cuenta de [SQL API](sql-api-introduction.md) de Azure Cosmos DB, una base de datos de documentos y un contenedor mediante Azure Portal. Luego se compila y se ejecuta una aplicación de consola compilada con el SDK de Python para [SQL API](sql-api-sdk-python.md). Este inicio rápido usa la versión 3.0 del [SDK de Python](https://pypi.org/project/azure-cosmos).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)][!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

## <a name="prerequisites"></a>Requisitos previos

* [Python 3.6](https://www.python.org/downloads/), con el ejecutable `python` disponible en `PATH`.
* [Visual Studio Code](https://code.visualstudio.com/)
* [Extensión de Python para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python#overview)

## <a name="create-a-database-account"></a>Creación de una cuenta de base de datos

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

## <a name="add-a-collection"></a>Agregar una colección

[!INCLUDE [cosmos-db-create-collection](../../includes/cosmos-db-create-collection.md)]

## <a name="add-sample-data"></a>Adición de datos de ejemplo

[!INCLUDE [cosmos-db-create-sql-api-add-sample-data](../../includes/cosmos-db-create-sql-api-add-sample-data.md)]

## <a name="query-your-data"></a>Consulta de los datos

[!INCLUDE [cosmos-db-create-sql-api-query-data](../../includes/cosmos-db-create-sql-api-query-data.md)]

## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Ahora vamos a clonar una aplicación de SQL API desde GitHub, a establecer la cadena de conexión y a ejecutarla.

1. Abra un símbolo del sistema, cree una carpeta nueva denominada ejemplos de GIT y, después, cierre el símbolo del sistema.

    ```bash
    md "C:\git-samples"
    ```

2. Abra una ventana de terminal de Git, como git bash y utilice el comando `cd` para cambiar a la nueva carpeta para instalar la aplicación de ejemplo.

    ```bash
    cd "C:\git-samples"
    ```

3. Ejecute el comando siguiente para clonar el repositorio de ejemplo. Este comando crea una copia de la aplicación de ejemplo en el equipo. 

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-python-getting-started.git
    ```  

## <a name="update-your-connection-string"></a>Actualización de la cadena de conexión

Ahora vuelva a Azure Portal para obtener la información de la cadena de conexión y cópiela en la aplicación.

1. En [Azure Portal](https://portal.azure.com/), en la cuenta de Azure Cosmos DB, en el panel de navegación izquierdo haga clic en **Claves**. Tiene que usar los botones de copia del lado derecho de la pantalla para copiar los valores de **URI** y **Clave principal** en el archivo `CosmosGetStarted.py` en el paso siguiente.

    ![Visualización y copia de una clave de acceso en Azure Portal, hoja Claves](./media/create-sql-api-dotnet/keys.png)

2. Abra el archivo `CosmosGetStarted.py` en C:\git-samples\azure-cosmos-db-python-getting-started en Visual Studio Code.

3. Copie el valor **URI** del portal (con el botón de copia) y conviértalo en el valor de la clave de **punto de conexión** en ``CosmosGetStarted.py``. 

    `'ENDPOINT': 'https://FILLME.documents.azure.com',`

4. Después, copie el valor **CLAVE PRINCIPAL** del portal y conviértalo en el valor de **config.PRIMARYKEY** en ``CosmosGetStarted.py``. Ya ha actualizado la aplicación con toda la información que necesita para comunicarse con Azure Cosmos DB. 

    `'PRIMARYKEY': 'FILLME',`

5. Guarde el archivo ``CosmosGetStarted.py``.

## <a name="review-the-code"></a>Revisión del código

Este paso es opcional. Aprenda sobre los recursos de base de datos creados en código o vaya directamente a [Actualización de la cadena de conexión](#update-your-connection-string).

Observe que, si está familiarizado con la versión anterior del SDK de Python, puede estar habituado a ver los términos "colección" y "documento". Dado que Azure Cosmos DB admite varios modelos de API, la versión 3.0+ del SDK de Python usa los términos genéricos "contenedor", que puede ser una colección, un gráfico o una tabla, y "elemento" para describir el contenido del contenedor.

Los fragmentos de código siguientes se han tomado del archivo `CosmosGetStarted.py`.

* Se inicializa CosmosClient. Asegúrese de actualizar los valores de "punto de conexión" y "clave maestra" según se describe en la sección [Actualización de la cadena de conexión](#update-your-connection-string). 

    ```python
    # Initialize the Cosmos client
    client = cosmos_client.CosmosClient(url_connection=config['ENDPOINT'], auth={'masterKey': config['MASTERKEY']})
    ```

* Se crea una base de datos.

    ```python
    # Create a database
    db = client.CreateDatabase({ 'id': config['DATABASE'] })
    ```

* Se crea una colección.

    ```python
    # Create collection options
    options = {
        'offerThroughput': 400
    }

    # Create a container
    container = client.CreateContainer(db['_self'], container_definition, options)
    ```

* Se agregan algunos elementos al contenedor.

    ```python
    # Create and add some items to the container
    item1 = client.CreateItem(container['_self'], {
        'serverId': 'server1',
        'Web Site': 0,
        'Cloud Service': 0,
        'Virtual Machine': 0,
        'message': 'Hello World from Server 1!'
        }
    )

    item2 = client.CreateItem(container['_self'], {
        'serverId': 'server2',
        'Web Site': 1,
        'Cloud Service': 0,
        'Virtual Machine': 0,
        'message': 'Hello World from Server 2!'
        }
    )
    ```

* Se realiza una consulta con SQL.

    ```python
    query = {'query': 'SELECT * FROM server s'}

    options = {}
    options['enableCrossPartitionQuery'] = True
    options['maxItemCount'] = 2

    result_iterable = client.QueryItems(container['_self'], query, options)
    for item in iter(result_iterable):
        print(item['message'])
    ```
   
## <a name="run-the-app"></a>Ejecución de la aplicación

1. En el Visual Studio Code, seleccione **Ver**>**Paleta de comandos**. 

2. En el símbolo del sistema, escriba **Python: Select Interpreter** y seleccione la versión de Python que se va a utilizar.

    El pie de página de Visual Studio Code se actualiza para indicar el intérprete seleccionado. 

3. Seleccione **Ver** > **Terminal integrado** para abrir el terminal integrado de Visual Studio Code.

4. En la ventana del terminal integrado, asegúrese de que está en la carpeta azure-cosmos-db-python-getting-started. Si no lo está, ejecute el siguiente comando para cambiar a la carpeta de ejemplo. 

    ```
    cd "C:\git-samples\azure-cosmos-db-python-getting-started"`
    ```

5. Ejecute el comando siguiente para instalar el paquete azure-cosmos. 

    ```
    pip3 install azure-cosmos
    ```

    Si aparece un error que indica que se deniega el acceso al intentar instalar azure-cosmos, será preciso [ejecutar VS Code como administrador](https://stackoverflow.com/questions/37700536/visual-studio-code-terminal-how-to-run-a-command-with-administrator-rights).

6. Ejecute el siguiente comando para ejecutar el ejemplo y crear y almacenar documentos nuevos en Azure Cosmos DB.

    ```
    python CosmosGetStarted.py
    ```

7. Para confirmar que los nuevos elementos se han creado y guardado, en Azure Portal, seleccione **Explorador de datos**, expanda **coll**, expanda **Documentos** y luego seleccione el documento **server1**. El contenido del documento server1 coincide con el contenido devuelto en la ventana del terminal integrado. 

    ![Ver los nuevos documentos en Azure Portal](./media/create-sql-api-python/azure-cosmos-db-confirm-documents.png)

## <a name="review-slas-in-the-azure-portal"></a>Revisión de los SLA en Azure Portal

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha aprendido a crear una cuenta de Azure Cosmos DB, crear una colección mediante el Explorador de datos y ejecutar una aplicación. Ahora puede importar datos adicionales en la cuenta de Cosmos DB. 

> [!div class="nextstepaction"]
> [Importación de datos en Azure Cosmos DB para SQL API](import-data.md)


