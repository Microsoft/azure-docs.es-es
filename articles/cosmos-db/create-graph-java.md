---
title: Compilación de una base de datos de grafos con Java en Azure Cosmos DB
description: Se presenta un ejemplo de código de Java que puede usar para conectarse y consultar datos de grafos de Azure Cosmos DB con Gremlin.
author: luisbosquez
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.devlang: java
ms.topic: quickstart
ms.date: 03/26/2019
ms.author: lbosq
ms.custom: seo-java-july2019, seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: 05fc7a7a5bd7e045125cb303e1f5c29d550c58ef
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323679"
---
# <a name="quickstart-build-a-graph-database-with-the-java-sdk-and-the-azure-cosmos-db-gremlin-api"></a>Inicio rápido: Creación de una base de datos de grafos con el SDK de Java y una cuenta de Azure Cosmos DB para Gremlin API

> [!div class="op_single_selector"]
> * [Gremlin Console](create-graph-gremlin-console.md)
> * [.NET](create-graph-dotnet.md)
> * [Java](create-graph-java.md)
> * [Node.js](create-graph-nodejs.md)
> * [Python](create-graph-python.md)
> * [PHP](create-graph-php.md)
>  

En este inicio rápido, se crea y administra una cuenta de Azure Cosmos DB para Gremlin (Graph) API desde Azure Portal, y se agregan datos mediante una aplicación de Java clonada desde GitHub. Azure Cosmos DB es un servicio de base de datos multimodelo que permite crear y consultar rápidamente bases de datos de documentos, tablas, claves-valores y grafos con funcionalidades de distribución global y escala horizontal.

## <a name="prerequisites"></a>Requisitos previos
- Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). 
- [Java Development Kit (JDK) 8](https://www.azul.com/downloads/azure-only/zulu/?&version=java-8-lts&architecture=x86-64-bit&package=jdk). Apunte su variable de entorno `JAVA_HOME` a la carpeta donde está instalado el JDK.
- Un [archivo binario de Maven](https://maven.apache.org/download.cgi). 
- [Git](https://www.git-scm.com/downloads). 

## <a name="create-a-database-account"></a>Creación de una cuenta de base de datos

Para poder crear una base de datos de grafos, debe crear una cuenta de base de datos de Gremlin (Graph) con Azure Cosmos DB.

[!INCLUDE [cosmos-db-create-dbaccount-graph](../../includes/cosmos-db-create-dbaccount-graph.md)]

## <a name="add-a-graph"></a>Agregar un grafo

[!INCLUDE [cosmos-db-create-graph](../../includes/cosmos-db-create-graph.md)]

## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Ahora vamos a empezar a trabajar con el código. Vamos a clonar una aplicación de Gremlin API desde GitHub, establecer la cadena de conexión y ejecutarla. Verá lo fácil que es trabajar con datos mediante programación.  

1. Abra un símbolo del sistema, cree una carpeta nueva denominada ejemplos de GIT y, después, cierre el símbolo del sistema.

    ```bash
    md "C:\git-samples"
    ```

2. Abra una ventana de terminal de Git, como git bash y utilice el comando `cd` para cambiar a una carpeta para instalar la aplicación de ejemplo.  

    ```bash
    cd "C:\git-samples"
    ```

3. Ejecute el comando siguiente para clonar el repositorio de ejemplo. Este comando crea una copia de la aplicación de ejemplo en el equipo. 

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-graph-java-getting-started.git
    ```

## <a name="review-the-code"></a>Revisión del código

Este paso es opcional. Si está interesado en aprender cómo se crean los recursos de base de datos en el código, puede revisar los siguientes fragmentos de código. En caso contrario, puede ir directamente a [Actualización de la cadena de conexión](#update-your-connection-information).

Los siguientes fragmentos de código se toman del archivo *C:\git-samples\azure-cosmos-db-graph-java-getting-started\src\GetStarted\Program.java*.

Esta aplicación de consola de Java usa una base de datos de [Gremlin API](graph-introduction.md) con el controlador [Apache TinkerPop](https://tinkerpop.apache.org/) de OSS. 

- El valor de `Client` de Gremlin se inicializa a partir de la configuración del archivo *C:\git-samples\azure-cosmos-db-graph-java-getting-started\src\remote.yaml*.

    ```java
    cluster = Cluster.build(new File("src/remote.yaml")).create();
    ...
    client = cluster.connect();
    ```

- Se ejecuta una serie de pasos de Gremlin mediante el método `client.submit`.

    ```java
    ResultSet results = client.submit(gremlin);

    CompletableFuture<List<Result>> completableFutureResults = results.all();
    List<Result> resultList = completableFutureResults.get();

    for (Result result : resultList) {
        System.out.println(result.toString());
    }
    ```

## <a name="update-your-connection-information"></a>Actualización de la información de conexión

Ahora vuelva a Azure Portal para obtener la información de conexión y cópiela en la aplicación. Esta configuración permitirá que la aplicación se comunique con la base de datos hospedada.

1. En la cuenta de Azure Cosmos DB, en [Azure Portal](https://portal.azure.com/), seleccione **Claves**. 

    Copie la primera parte del valor URI.

    :::image type="content" source="./media/create-graph-java/copy-access-key-azure-portal.png" alt-text="Visualización y copia de una clave de acceso en Azure Portal, página Claves":::

2. Abra el archivo *src/remote.yaml* y pegue el valor del identificador único de `$name$` en `hosts: [$name$.graphs.azure.com]`.

    La línea 1 de *remote.yaml* debería ser parecida a esta 

    `hosts: [test-graph.graphs.azure.com]`

3. Cambio `graphs` a `gremlin.cosmosdb` en el valor `endpoint`. (Si creó su cuenta de base de datos de grafos antes del 20 de diciembre de 2017, no realice cambios en el valor del punto de conexión y continúe al siguiente paso).

    El valor del punto de conexión debe tener el siguiente aspecto:

    `"endpoint": "https://testgraphacct.gremlin.cosmosdb.azure.com:443/"`

4. En Azure Portal, use el botón Copiar para copiar la clave principal y péguela sobre `$masterKey$` en `password: $masterKey$`.

    La línea 4 de *remote.yaml* debería ser parecida a esta 

    `password: 2Ggkr662ifxz2Mg==`

5. Cambie la línea 3 de *remote.yaml* de

    `username: /dbs/$database$/colls/$collection$`

    to 

    `username: /dbs/sample-database/colls/sample-graph`

    Si ha utilizado un nombre único para el gráfico o la base de datos de ejemplo, actualice los valores según corresponda.

6. Guarde el archivo *remote.yaml*.

## <a name="run-the-console-app"></a>Ejecutar la aplicación de consola

1. En la ventana del terminal de git, use `cd` para cambiar a la carpeta azure-cosmos-db-graph-java-getting-started.

    ```git
    cd "C:\git-samples\azure-cosmos-db-graph-java-getting-started"
    ```

2. En la ventana del terminal de GIT, use el comando siguiente para instalar los paquetes Java necesarios.

   ```git
   mvn package
   ```

3. En la ventana de terminal de GIT, use el siguiente comando para iniciar la aplicación de Java.
    
    ```git
    mvn exec:java -D exec.mainClass=GetStarted.Program
    ```

    En la ventana del terminal se muestran los vértices y los bordes que se agregan al grafo. 
    
    Si se producen errores de tiempo de espera, compruebe que actualizó la información de conexión correctamente en [Actualización de la información de conexión](#update-your-connection-information) e intente ejecutar de nuevo el último comando. 
    
    Una vez que el programa se detenga, seleccione ENTRAR y, después, vuelva a cambiar a Azure Portal en el explorador de Internet. 

<a id="add-sample-data"></a>
## <a name="review-and-add-sample-data"></a>Revise y agregue datos de ejemplo

Ahora puede volver al Explorador de datos y ver los vértices que se agregan al grafo, y agregar puntos de datos adicionales.

1. En la cuenta de Azure Cosmos DB, en Azure Portal, seleccione **Explorador de datos**, expanda **sample-graph**, seleccione **Grafo** y, por último, seleccione **Aplicar filtro**. 

   :::image type="content" source="./media/create-graph-java/azure-cosmosdb-data-explorer-expanded.png" alt-text="Creación de documentos en el Explorador de datos en Azure Portal":::

2. En la lista **Resultados**, observe los nuevos usuarios agregados al grafo. Seleccione **ben** y observe que el usuario se ha conectado a robin. Puede mover los vértices arrastrando y colocando, acercarse y alejarse desplazando la rueda del mouse y ampliar el tamaño del gráfico con la flecha doble. 

   :::image type="content" source="./media/create-graph-java/azure-cosmosdb-graph-explorer-new.png" alt-text="Nuevos vértices en el grafo del Explorador de datos de Azure Portal":::

3. Vamos a agregar algunos usuarios nuevos. Seleccione **Nuevo vértice** para agregar datos al grafo.

   :::image type="content" source="./media/create-graph-java/azure-cosmosdb-data-explorer-new-vertex.png" alt-text="Creación de documentos en el Explorador de datos en Azure Portal":::

4. En el cuadro de etiqueta, escriba *person*.

5. Seleccione **Agregar propiedad** para agregar cada una de las siguientes propiedades. Tenga en cuenta que puede crear propiedades únicas para cada persona en el grafo. Solo se requiere la clave de identificador.

    key|value|Notas
    ----|----|----
    id|ashley|Identificador único del vértice. Si no se especifica un identificador, se genera uno automáticamente.
    gender|mujer| 
    técnico | Java | 

    > [!NOTE]
    > En esta guía de inicio rápido se crea una colección sin particiones. Sin embargo, si crea una colección con particiones especificando una clave de partición durante la creación de la colección, debe incluir la clave de partición como una clave en cada nuevo vértice. 

6. Seleccione **Aceptar**. Puede que tenga que expandir la pantalla para ver **Aceptar** en la parte inferior de la pantalla.

7. Seleccione **Nuevo vértice** de nuevo y agregue otro usuario. 

8. Escriba una etiqueta de *persona*.

9. Seleccione **Agregar propiedad** para agregar cada una de las siguientes propiedades:

    key|value|Notas
    ----|----|----
    id|rakesh|Identificador único del vértice. Si no se especifica un identificador, se genera uno automáticamente.
    gender|hombre| 
    centro educativo|MIT| 

10. Seleccione **Aceptar**. 

11. Seleccione el botón **Aplicar filtro** con el filtro `g.V()` predeterminado para mostrar todos los valores en el gráfico. Todos los usuarios se muestran ahora en la lista **Resultados**. 

    Si agrega más datos, puede usar filtros para limitar los resultados. De forma predeterminada, el Explorador de datos usa `g.V()` para recuperar todos los vértices en un gráfico. Se puede cambiar a otra [consulta de gráfico](tutorial-query-graph.md), como `g.V().count()`, para devolver un recuento de todos los vértices en el gráfico en formato JSON. Si ha cambiado el filtro, vuelva a cambiar el filtro a `g.V()` y seleccione **Aplicar filtro** para volver a mostrar todos los resultados.

12. Ahora puede conectar a rakesh y ashley. Asegúrese de que **ashley** está seleccionado en la lista **Results** (Resultados) y seleccione :::image type="content" source="./media/create-graph-java/edit-pencil-button.png" alt-text="Change the target of a vertex in a graph"::: (Cambiar el destino de un vértice de un grafo) al lado de **Targets** (Destinos) en la parte inferior derecha. Para ver el botón, es posible que tenga que ensanchar la ventana.

    :::image type="content" source="./media/create-graph-java/azure-cosmosdb-data-explorer-edit-target.png" alt-text="Cambio del destino de un vértice en un grafo: Azure Cosmos DB":::

13. En el cuadro **Destino**, escriba *rakesh*, en el cuadro **Edge label** (Etiqueta de borde), escriba *sabe* y, a continuación, seleccione la casilla.

    :::image type="content" source="./media/create-graph-java/azure-cosmosdb-data-explorer-set-target.png" alt-text="Adición de una conexión en el Explorador de datos: Azure Cosmos DB":::

14. Ahora seleccione **rakesh** en la lista de resultados y compruebe que ashley y rakesh están conectados. 

    :::image type="content" source="./media/create-graph-java/azure-cosmosdb-graph-explorer.png" alt-text="Dos vértices conectados en el Explorador de datos: Azure Cosmos DB":::

Esto completa la parte de la creación de recursos de este tutorial. Puede seguir agregando vértices en el grafo, modificar los vértices existentes o cambiar las consultas. Ahora vamos a revisar las métricas que proporciona Azure Cosmos DB y, a continuación, limpiaremos los recursos. 

## <a name="review-slas-in-the-azure-portal"></a>Revisión de los SLA en Azure Portal

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear una cuenta de Azure Cosmos DB, a crear un grafo mediante el Explorador de datos y a ejecutar una aplicación de Java que agrega datos al grafo. Ahora puede crear consultas más complejas e implementar con Gremlin una lógica eficaz de recorrido del grafo. 

> [!div class="nextstepaction"]
> [Consulta mediante Gremlin](tutorial-query-graph.md)

