---
title: 'Inicio rápido: Cassandra API con Python: Azure Cosmos DB'
description: En esta guía de inicio rápido se muestra cómo usar la Apache Cassandra API de Azure Cosmos DB para crear una aplicación de perfil con Python.
author: TheovanKraay
ms.author: thvankra
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.devlang: python
ms.topic: quickstart
ms.date: 08/13/2020
ms.custom: devx-track-python
ms.openlocfilehash: f376a1f3601c976ff1efdaee1da6181510a9cf64
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88234948"
---
# <a name="quickstart-build-a-cassandra-app-with-python-sdk-and-azure-cosmos-db"></a>Inicio rápido: Creación de una aplicación de Cassandra con el SDK para Python y Azure Cosmos DB

> [!div class="op_single_selector"]
> * [.NET](create-cassandra-dotnet.md)
> * [.NET Core](create-cassandra-dotnet-core.md)
> * [Java v3](create-cassandra-java.md)
> * [Java v4](create-cassandra-java-v4.md)
> * [Node.js](create-cassandra-nodejs.md)
> * [Python](create-cassandra-python.md)
>  

En este inicio rápido se crea una cuenta de Cassandra API de Azure Cosmos DB y se utiliza una aplicación Python de Cassandra clonada desde GitHub para crear un contenedor y una base de datos de Cassandra. Azure Cosmos DB es un servicio de base de datos multimodelo que permite crear y consultar rápidamente bases de datos de documentos, tablas, claves-valores y grafos con funcionalidades de distribución global y escala horizontal.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). O bien, [pruebe gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) sin una suscripción de Azure.
- [Python 2.7.14+ o 3.4+](https://www.python.org/downloads/).
- [Git](https://git-scm.com/downloads).
- [Controlador de Python para Apache Cassandra](https://github.com/datastax/python-driver).

## <a name="create-a-database-account"></a>Creación de una cuenta de base de datos

Para poder crear una base de datos de documentos, debe crear una cuenta de Cassandra con Azure Cosmos DB.

[!INCLUDE [cosmos-db-create-dbaccount-cassandra](../../includes/cosmos-db-create-dbaccount-cassandra.md)]

## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Ahora vamos a clonar una aplicación de la API de Cassandra desde GitHub, establecer la cadena de conexión y ejecutarla. Verá lo fácil que es trabajar con datos mediante programación. 

1. Abra un símbolo del sistema. Cree una carpeta nueva denominada `git-samples`. Después cierre el símbolo del sistema.

    ```bash
    md "C:\git-samples"
    ```

2. Abra una ventana de terminal de Git, como git bash y utilice el comando `cd` para cambiar a la nueva carpeta para instalar la aplicación de ejemplo.

    ```bash
    cd "C:\git-samples"
    ```

3. Ejecute el comando siguiente para clonar el repositorio de ejemplo. Este comando crea una copia de la aplicación de ejemplo en el equipo.

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-cassandra-python-getting-started.git
    ```

## <a name="review-the-code"></a>Revisión del código

Este paso es opcional. Si le interesa aprender cómo el código crea los recursos de base de datos, puede revisar los siguientes fragmentos de código. Los fragmentos de código se han tomado del archivo *pyquickstart.py*. En caso contrario, puede ir directamente a [Actualización de la cadena de conexión](#update-your-connection-string). 

* `cluster` se inicializa con la información de `contactPoint` y `port` que se recupera de Azure Portal. Luego, `cluster` se conecta a Cassandra API de Azure Cosmos DB, para lo que usa el método `connect()`. Una conexión autorizada se establece mediante el nombre de usuario, la contraseña y el certificado predeterminado, o bien un certificado explícito si se proporciona en el archivo de configuración.

  :::code language="python" source="~/cosmosdb-cassandra-python-sample/pyquickstart.py" id="authenticateAndConnect":::

* Se crea un espacio de claves.

  :::code language="python" source="~/cosmosdb-cassandra-python-sample/pyquickstart.py" id="createKeyspace":::

* Se crea una tabla.

  :::code language="python" source="~/cosmosdb-cassandra-python-sample/pyquickstart.py" id="createTable":::

* Se insertan entidades de clave/valor.

  :::code language="python" source="~/cosmosdb-cassandra-python-sample/pyquickstart.py" id="insertData":::

* Realice una consulta para obtener todos los valores de clave.

  :::code language="python" source="~/cosmosdb-cassandra-python-sample/pyquickstart.py" id="queryAllItems":::
    
* Realice una consulta para obtener un valor de clave.

  :::code language="python" source="~/cosmosdb-cassandra-python-sample/pyquickstart.py" id="queryByID":::

## <a name="update-your-connection-string"></a>Actualización de la cadena de conexión

Ahora vuelva a Azure Portal para obtener la información de la cadena de conexión y cópiela en la aplicación. La cadena de conexión permite a la aplicación comunicarse con la base de datos hospedada.

1. En la cuenta de Azure Cosmos DB, en [Azure Portal](https://portal.azure.com/), seleccione **Cadena de conexión**. 

1. Use el botón :::image type="icon" source="./media/create-cassandra-python/copy.png"::: del lado derecho de la pantalla para copiar el valor superior, CONTACT POINT (Punto de contacto). 

   :::image type="content" source="./media/create-cassandra-python/keys.png" alt-text="Visualizar y copiar un punto de contacto, una contraseña y un nombre de usuario de acceso en la hoja de la cadena de conexión de Azure Portal":::

1. Abra el archivo *config.py*. 

1. Pegue el valor de CONTACT POINT (Punto de contacto) del portal en `<FILLME>` en la línea 10.

    La línea 10 ahora debe ser similar a 

    `'contactPoint': 'cosmos-db-quickstarts.cassandra.cosmosdb.azure.com:10350'`

1. Copie el valor de NOMBRE DE USUARIO del portal y péguelo en `<FILLME>` en la línea 6.

    La línea 6 ahora debe ser similar a 

    `'username': 'cosmos-db-quickstart',`
    
1. Copie el valor de CONTRASEÑA del portal y péguelo en `<FILLME>` en la línea 8.

    La línea 8 ahora debe ser similar a

    `'password' = '2Ggkr662ifxz2Mg==`';`

1. Guarde el archivo *config.py*.
    
## <a name="use-the-x509-certificate"></a>Uso del certificado X509

1. Descargue el certificado Baltimore CyberTrust Root de forma loca desde [https://cacert.omniroot.com/bc2025.crt](https://cacert.omniroot.com/bc2025.crt). Cambie el nombre del archivo con la extensión de archivo *.cer*.

   El certificado tiene el número de serie `02:00:00:b9` y la huella digital SHA1 `d4🇩🇪20:d0:5e:66:fc:53:fe:1a:50:88:2c:78:db:28:52:ca:e4:74`.

2. Abra *pyquickstart.py* y cambie `path\to\cert` para que apunte al certificado nuevo.

3. Guarde *pyquickstart.py*.

## <a name="run-the-python-app"></a>Ejecutar la aplicación de Python

1. Utilice el comando cd en el terminal git para cambiarlo a la carpeta `azure-cosmos-db-cassandra-python-getting-started`. 

2. Ejecute los comandos siguientes para instalar los módulos necesarios:

    ```python
    python -m pip install cassandra-driver
    python -m pip install prettytable
    python -m pip install requests
    python -m pip install pyopenssl
    ```

2. Ejecute el comando siguiente para iniciar la aplicación de Python:

    ```
    python pyquickstart.py
    ```

3. Compruebe los resultados previstos desde la línea de comandos.

    Presione CTRL+C para detener la ejecución del programa y cerrar la ventana de la consola. 

    :::image type="content" source="./media/create-cassandra-python/output.png" alt-text="Visualizar y comprobar el resultado":::
    
4. En Azure Portal abra **Explorador de datos** para consultar, modificar y trabajar con estos nuevos datos. 

    :::image type="content" source="./media/create-cassandra-python/data-explorer.png" alt-text="Visualizar los datos en el Explorador de datos":::

## <a name="review-slas-in-the-azure-portal"></a>Revisión de los SLA en Azure Portal

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha aprendido a crear una cuenta de Azure Cosmos DB con Cassandra API y a ejecutar una aplicación Python de Cassandra que cree un contenedor y una base de datos de Cassandra. Ya puede importar datos adicionales en la cuenta de Azure Cosmos DB. 

> [!div class="nextstepaction"]
> [Importación de datos de Cassandra en Azure Cosmos DB](cassandra-import-data.md)

