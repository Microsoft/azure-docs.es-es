---
title: Compilación de una aplicación web de Python Flask mediante la API de Azure Cosmos DB para MongoDB
description: Se presenta un ejemplo de código de Python Flask que se puede usar para conectarse a la API de Azure Cosmos DB para MongoDB y realizar consultas.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: python
ms.topic: quickstart
ms.date: 12/26/2018
ms.custom: devx-track-python
ms.openlocfilehash: 1ed15182e700f125072c69ba9bcf60caf4da5c88
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87873048"
---
# <a name="quickstart-build-a-python-app-using-azure-cosmos-dbs-api-for-mongodb"></a>Inicio rápido: Compilación de una aplicación de Python mediante la API de Azure Cosmos DB para MongoDB

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.js](create-mongodb-nodejs.md)
> * [Python](create-mongodb-flask.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-go.md)
>  

En este inicio rápido, se usa una cuenta de Azure Cosmos DB para MongoDB API o el emulador de Azure Cosmos DB para ejecutar una aplicación web de lista de tareas pendientes de Python Flask clonada desde GitHub. Azure Cosmos DB es un servicio de base de datos multimodelo que permite crear y consultar rápidamente bases de datos de documentos, tablas, claves-valores y grafos con funcionalidades de distribución global y escala horizontal.

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). O bien, [pruebe gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) sin una suscripción de Azure. O bien, puede usar el [emulador de Azure Cosmos DB](local-emulator.md). 
- [Python 3.6 o posterior](https://www.python.org/downloads/)
- [Visual Studio Code](https://code.visualstudio.com/Download) con la [Extensión de Python](https://marketplace.visualstudio.com/items?itemName=donjayamanne.python).

## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Ahora clonaremos una aplicación de Flask-MongoDB desde GitHub, estableceremos la cadena de conexión y la ejecutaremos. Verá lo fácil que es trabajar con datos mediante programación.

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
    git clone https://github.com/Azure-Samples/CosmosDB-Flask-Mongo-Sample.git
    ```
3. Ejecute el siguiente comando para instalar los módulos de Python.

    ```bash 
    pip install -r .\requirements.txt
    ```
4. Abra la carpeta en Visual Studio Code.

## <a name="review-the-code"></a>Revisión del código

Este paso es opcional. Si está interesado en aprender cómo se crean los recursos de base de datos en el código, puede revisar los siguientes fragmentos de código. En caso contrario, puede ir directamente a [Ejecutar la aplicación web](#run-the-web-app). 

Los siguientes fragmentos de código se han tomado del archivo *app.py* y utilizan la cadena de conexión para el emulador de Azure Cosmos DB local. La contraseña debe dividirse tal como se muestra a continuación para dar cabida a la barra diagonal, que no se puede analizar de otra manera.

* Inicie el cliente de MongoDB, recupere la base de datos y realice la autenticación.

    ```python
    client = MongoClient("mongodb://127.0.0.1:10250/?ssl=true") #host uri
    db = client.test    #Select the database
    db.authenticate(name="localhost",password='C2y6yDjf5' + r'/R' + '+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw' + r'/Jw==')
    ```

* Recupere la colección o créela si aún no existe.

    ```python
    todos = db.todo #Select the collection
    ```

* Creación de la aplicación

    ```Python
    app = Flask(__name__)
    title = "TODO with Flask"
    heading = "ToDo Reminder"
    ```
    
## <a name="run-the-web-app"></a>Ejecución de la aplicación web

1. Asegúrese de que el emulador de Azure Cosmos DB se está ejecutando.

2. Abra una ventana de terminal y `cd` en el directorio donde se guarda la aplicación.

3. A continuación, establezca la variable de entorno para la aplicación de Flask con `set FLASK_APP=app.py` o `$env:FLASK_APP = app.py` para los editores de PowerShell, o `export FLASK_APP=app.py` si usa un equipo Mac. 

4. Ejecute la aplicación con `flask run` y vaya a *http:\///127.0.0.1:5000/* .

5. Agregue y quite tareas para ver la incorporación o el cambio en la colección.

## <a name="create-a-database-account"></a>Creación de una cuenta de base de datos

Si desea probar el código en una cuenta de Azure Cosmos DB en vivo, vaya a Azure Portal para crear una cuenta.

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount-mongodb.md)]

## <a name="update-your-connection-string"></a>Actualización de la cadena de conexión

Para probar el código en la cuenta de Azure Cosmos DB en vivo, obtenga la información de la cadena de conexión. A continuación, cópiela en la aplicación.

1. En Azure Portal, en la cuenta de Azure Cosmos DB, en el panel de navegación izquierdo, seleccione **Cadena de conexión** y, después, seleccione **Claves de lectura y escritura**. Usará los botones de copia del lado derecho de la pantalla para copiar el nombre de usuario, la cadena de conexión y la contraseña. 

2. Abra el archivo *app.py* en el directorio raíz.

3. Copie el valor del **nombre de usuario** del portal (con el botón de copia) y conviértalo en el valor del **nombre** en el archivo *app.py*.

4. A continuación, copie el valor de la **cadena de conexión** del portal y establézcalo como valor de **MongoClient** en el archivo *app.py*.

5. Por último, copie el valor de la **contraseña** del portal y conviértalo en el valor de la **contraseña** en el archivo *app.py*.

Ya ha actualizado la aplicación con toda la información que necesita para comunicarse con Azure Cosmos DB. Puede ejecutarla como antes.

## <a name="deploy-to-azure"></a>Implementar en Azure

Para implementar esta aplicación, puede crear una aplicación web en Azure y habilitar la implementación continua con una bifurcación de este repositorio de GitHub. Para configurar la implementación continua con GitHub en Azure, siga este [tutorial](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment).

Para la implementación en Azure, debe quitar las claves de la aplicación y asegurarse de que la sección siguiente no tiene comentarios:

```python
    client = MongoClient(os.getenv("MONGOURL"))
    db = client.test    #Select the database
    db.authenticate(name=os.getenv("MONGO_USERNAME"),password=os.getenv("MONGO_PASSWORD"))
```

Después debe agregar los valores de MONGOURL, MONGO_PASSWORD y MONGO_USERNAME a la configuración de la aplicación. Puede seguir este [tutorial](https://docs.microsoft.com/azure/app-service/configure-common#configure-app-settings) para aprender acerca de la configuración de la aplicación en Azure Web Apps.

Si no desea crear una bifurcación de este repositorio, también puede seleccionar el botón **Implementar en Azure** a continuación. Luego, debe ir a Azure y configurar los valores de la aplicación con la información de la cuenta de Azure Cosmos DB.

<a href="https://deploy.azure.com/?repository=https://github.com/heatherbshapiro/To-Do-List---Flask-MongoDB-Example" target="_blank">
<img src="https://azuredeploy.net/deploybutton.png" alt="Click to Deploy to Azure">
</a>

> [!NOTE]
> Si planea almacenar el código en GitHub u otras opciones de control de código fuente, asegúrese de quitar las cadenas de conexión del código. En su lugar, se pueden establecer con la configuración de la aplicación web.

## <a name="review-slas-in-the-azure-portal"></a>Revisión de los SLA en Azure Portal

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear una cuenta de Azure Cosmos DB para MongoDB API y a usar el emulador de Azure Cosmos DB para ejecutar una aplicación web de lista de tareas pendientes de Python Flask clonada desde GitHub. Ahora puede importar datos adicionales en la cuenta de Azure Cosmos DB. 

> [!div class="nextstepaction"]
> [Importación de datos de MongoDB a Azure Cosmos DB](mongodb-migrate.md)
