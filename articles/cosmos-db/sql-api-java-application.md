---
title: 'Tutorial: Compilación de una aplicación web de Java mediante Azure Cosmos DB y la API de SQL'
description: 'Tutorial: En este tutorial de aplicaciones web de Java, aprenderá a usar Azure Cosmos DB y la API de SQL para almacenar datos y acceder a ellos desde una aplicación de Java hospedada en Azure Websites.'
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: tutorial
ms.date: 05/12/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 9daa09209818b9e01738630911a1bb926e3c257d
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88079950"
---
# <a name="tutorial-build-a-java-web-application-using-azure-cosmos-db-and-the-sql-api"></a>Tutorial: Compilación de una aplicación web de Java mediante Azure Cosmos DB y la API de SQL

> [!div class="op_single_selector"]
> * [.NET](sql-api-dotnet-application.md)
> * [Java](sql-api-java-application.md)
> * [Node.js](sql-api-nodejs-application.md)
> * [Python](sql-api-python-application.md)
> * [Xamarin](mobile-apps-with-xamarin.md)
> 

En este tutorial de aplicación web Java aprenderá a usar el servicio [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) para almacenar datos y acceder a ellos desde una aplicación de Java hospedada en Azure App Service Web Apps. En este artículo, aprenderá lo siguiente:

* Cómo crear una aplicación de JavaServer Pages (JSP) básica en Eclipse.
* Cómo trabajar con el servicio Azure Cosmos DB con el [SDK de Java de Azure Cosmos DB](https://github.com/Azure/azure-documentdb-java).

Este tutorial de aplicación Java muestra cómo crear una aplicación de administración de tareas basadas en web que permite crear, recuperar y marcar tareas como completadas, tal como se muestra en la siguiente imagen. Las tareas de la lista de tareas pendientes se almacenan como documentos JSON en Azure Cosmos DB.

:::image type="content" source="./media/sql-api-java-application/image1.png" alt-text="Aplicación Java My ToDo List":::

> [!TIP]
> En este tutorial de desarrollo de aplicaciones se supone que tiene experiencia anterior en el uso de Java. Si no está familiarizado con Java o con las [herramientas de requisitos previos](#Prerequisites), se recomienda que descargue el proyecto completo [todo](https://github.com/Azure-Samples/documentdb-java-todo-app) de GitHub y lo compile mediante las [instrucciones que se encuentran al final de este artículo](#GetProject). Una vez compilado, puede revisar el artículo para obtener información sobre el código en el contexto del proyecto.  
>

## <a name="prerequisites-for-this-java-web-application-tutorial"></a><a id="Prerequisites"></a>Requisitos previos para este tutorial de aplicación web de Java

Antes de comenzar este tutorial de desarrollo de aplicaciones, debe disponer de lo siguiente:

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar. 

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [Kit de desarrollo de Java (JDK) 7+](/java/azure/jdk/?view=azure-java-stable).
* [IDE de Eclipse para desarrolladores de Java EE.](https://www.eclipse.org/downloads/packages/release/luna/sr1/eclipse-ide-java-ee-developers)
* [Un sitio web de Azure con un entorno de tiempo de ejecución Java (por ejemplo, Tomcat o Jetty) habilitado.](../app-service/quickstart-java.md)

Si va a instalar estas herramientas por primera vez, coreservlets.com proporciona un ejemplo paso a paso del proceso de instalación en la sección de inicio rápido de su artículo [Tutorial: Instalación de TomCat7 y uso con Eclipse](http://www.coreservlets.com/Apache-Tomcat-Tutorial/tomcat-7-with-eclipse.html).

## <a name="create-an-azure-cosmos-db-account"></a><a id="CreateDB"></a>Creación de una cuenta de Azure Cosmos DB

Para comenzar, creemos una cuenta de Azure Cosmos DB. Si ya tiene una cuenta o si usa el Emulador de Azure Cosmos DB para este tutorial, puede ir directamente al [Paso 2: Creación de la aplicación JSP de Java](#CreateJSP).

[!INCLUDE [create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

[!INCLUDE [keys](../../includes/cosmos-db-keys.md)]

## <a name="create-the-java-jsp-application"></a><a id="CreateJSP"></a>Creación de la aplicación JSP de Java

Para crear la aplicación JSP:

1. En primer lugar, empezaremos por la creación de un proyecto de Java. Inicie Eclipse, haga clic en **File** (Archivo), haga clic en **New** (Nuevo) y luego haga clic en **Dynamic Web Project** (Proyecto web dinámico). Si **Dynamic Web Project** (Proyecto web dinámico) no aparece como proyecto disponible, haga lo siguiente: haga clic en **File** (Archivo), **New** (Nuevo), **Project** (Proyecto), expanda **Web**, haga clic en **Dynamic Web Project** (Proyecto web dinámico) y haga clic en **Next** (Siguiente).
   
    :::image type="content" source="./media/sql-api-java-application/image10.png" alt-text="Desarrollo de aplicaciones Java JSP":::

1. Escriba un nombre de proyecto en el cuadro **Project name** (Nombre de proyecto) y en el menú desplegable **Target Runtime** (Tiempo de ejecución de destino), seleccione opcionalmente un valor (por ejemplo, Apache Tomcat v7.0) y, a continuación, haga clic en **Finish** (Finalizar). Si selecciona un tiempo de ejecución de destino, puede ejecutar el proyecto localmente a través de Eclipse.

1. En Eclipse, en la vista del explorador de proyectos, expanda el proyecto. Haga clic con el botón derecho en **WebContent**, haga clic en **New** (Nuevo) y, después, en **JSP File** (Archivo JSP).

1. En el cuadro de diálogo **New JSP File** (Nuevo archivo JSP), asigne al archivo el nombre **index.jsp**. Mantenga la carpeta principal como **WebContent**, como se muestra en la ilustración siguiente y, a continuación, haga clic en **Next** (Siguiente).
   
    :::image type="content" source="./media/sql-api-java-application/image11.png" alt-text="Tutorial de creación de un nuevo archivo JSP: aplicación web de Java":::

1. En el cuadro de diálogo **Select JSP Template** (Seleccionar plantilla JSP), para cumplir con el objetivo de este tutorial, seleccione **New JSP File (html)** (Nuevo archivo JSP [html]) y haga clic en **Finish** (Finalizar).

1. Cuando el archivo *index.jsp* se abra en Eclipse, agregue texto para mostrar **Hola a todos** dentro del elemento `<body>` existente. El contenido actualizado `<body>` debe ser similar al código siguiente:

   ```html
   <body>
     <% out.println("Hello World!"); %>
   </body>
   ```

1. Guarde el archivo *index.jsp*.

1. Si ha establecido un tiempo de ejecución de destino en el paso 2, puede hacer clic en **Project** (Proyecto) y, a continuación, en **Run** (Ejecutar) para ejecutar su aplicación de JSP localmente:

   :::image type="content" source="./media/sql-api-java-application/image12.png" alt-text="Hola mundo – Tutorial de aplicación de Java":::

## <a name="install-the-sql-java-sdk"></a><a id="InstallSDK"></a>Instalación del SDK de SQL para Java

La manera más sencilla de insertar el SDK de Java para SQL y sus dependencias es a través de [Apache Maven](https://maven.apache.org/). Para ello, deberá convertir el proyecto en un proyecto de Maven realizando los pasos siguientes:

1. Haga clic con el botón derecho en el proyecto en el explorador de proyectos, haga clic en **Configure** (Configurar) y en **Convert to Maven Project** (Convertir en proyecto Maven).

1. En la ventana **Crear nueva POM**, acepte los valores predeterminados y haga clic en **Finalizar**.

1. En **Explorador de proyectos**, abra el archivo pom.xml.

1. En la ficha **Dependencias**, en el panel **Dependencias**, haga clic en **Agregar**.

1. En la ventana **Seleccionar dependencia** , haga lo siguiente:
   
   * En el cuadro **Id. de grupode**, escriba `com.azure`.
   * En el cuadro **Id. de artefacto**, escriba `azure-cosmos`.
   * En el cuadro **Versión**, escriba `4.0.1-beta.1`.
  
   O bien, puede agregar el XML de dependencia para el identificador de grupo y el identificador de artefacto directamente al archivo *pom.xml*:

   ```xml
   <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-cosmos</artifactId>
      <version>4.0.1-beta.1</version>
   </dependency>
   ```

1. Haga clic en **Aceptar** y Maven instalará el SDK de SQL para Java o guarde el archivo pom.xml.

## <a name="use-the-azure-cosmos-db-service-in-your-java-application"></a><a id="UseService"></a>Uso del servicio Azure Cosmos DB en una aplicación de Java

Ahora vamos a agregar los modelos, las vistas y los controladores a la aplicación web.

### <a name="add-a-model"></a>Adición de un modelo

En primer lugar, vamos a definir un modelo en un nuevo archivo *TodoItem. Java*. La clase `TodoItem` define el esquema de un elemento junto con los métodos Getter y Setter:

:::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/documentdb/sample/model/TodoItem.java":::

### <a name="add-the-data-access-objectdao-classes"></a>Incorporación de clases de objetos de acceso a datos (DAO)

Cree un objeto de acceso a datos (DAO) para abstraer la conservación de los elementos ToDo en Azure Cosmos DB. Para guardar los elementos ToDo en una colección, el cliente necesitará saber en qué base de datos y colección se conservarán (como se hace referencia por los self-links). En general, es mejor almacenar en memoria caché la base de datos y la colección cuando sea posible para evitar recorridos de ida y vuelta adicionales a la base de datos.

1. Para invocar el servicio Azure Cosmos DB, debe crear una nueva instancia del objeto `cosmosClient`. En general, es mejor volver a utilizar el objeto `cosmosClient`, en lugar de construir un nuevo cliente para cada solicitud posterior. Puede volver a usar el cliente si lo define dentro de la clase `cosmosClientFactory`. Actualice el HOST y los valores de MASTER_KEY que guardó en el [paso 1](#CreateDB). Reemplace la variable HOST por el URI y reemplace el valor de MASTER_KEY por la clave principal. Use el código siguiente para crear la clase `CosmosClientFactory` en el archivo *CosmosClientFactory.java*:

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/documentdb/sample/dao/CosmosClientFactory.java":::

1. Cree un nuevo archivo *TodoDao.java* y agregue la clase `TodoDao` para crear, actualizar, leer y eliminar los elementos de la lista de tareas:

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/documentdb/sample/dao/TodoDao.java":::

1. Cree un nuevo archivo *MockDao.java* y agregue la clase `MockDao`, esta clase implementa la clase `TodoDao` para realizar operaciones CRUD en los elementos:

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/documentdb/sample/dao/MockDao.java":::

1. Cree un nuevo archivo *DocDbDao.java* y agregue la clase `DocDbDao`. Esta clase define el código para conservar los elementos de TodoItems en el contenedor, recupera la base de datos y la colección, si existe, o las crean en caso contrario. En este ejemplo se usa [Gson](https://code.google.com/p/google-gson/) para serializar y deserializar los objetos de Java antiguos sin formato de la tabla TodoItem (POJO) en los documentos JSON. Para guardar los elementos ToDo en una colección, el cliente necesitará saber en qué base de datos y colección se conservarán (como se hace referencia por los self-links). Esta clase también define la función auxiliar para recuperar los documentos por otro atributo (por ejemplo, "ID") en lugar de Self-Link. Podemos usar el método auxiliar para recuperar un documento JSON de TodoItem por identificador y deserializarlo en un objeto de Java antiguo sin formato de la tabla TodoItem.

   También puede utilizar el objeto cliente `cosmosClient` para obtener una colección o lista de TodoItems con una consulta SQL. Por último, defina el método Delete para eliminar un elemento TodoItem de la lista. El siguiente código muestra el contenido de la clase `DocDbDao`:

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/documentdb/sample/dao/DocDbDao.java":::

1. A continuación, cree un nuevo archivo *TodoDaoFactory.java* y agregue la clase `TodoDaoFactory` que crea un nuevo objeto DocDbDao:

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/documentdb/sample/dao/TodoDaoFactory.java":::

### <a name="add-a-controller"></a>Adición de un controlador

Agregue el controlador *TodoItemController* a la aplicación. En este proyecto, va a usar [Project Lombok](https://projectlombok.org/) para generar el constructor, los captadores, los establecedores y un generador. Como alternativa, puede escribir este código manualmente o dejar que el IDE lo genere:

:::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/documentdb/sample/controller/TodoItemController.java":::

### <a name="create-a-servlet"></a>Creación de un servlet

A continuación, cree un servlet para enrutar las solicitudes HTTP al controlador. Cree el archivo *ApiServlet.java* y defina el código siguiente en él:

:::code language="java" source="~/samples-cosmosdb-java-v4-web-app/src/com/microsoft/azure/documentdb/sample/ApiServlet.java":::

## <a name="wire-the-rest-of-the-of-java-app-together"></a><a id="Wire"></a>Conexión del resto de la aplicación de Java

Ahora que hemos terminado la parte divertida, todo lo que queda por hacer es crear una interfaz de usuario rápida y conectarla a nuestro DAO.

1. Necesita una interfaz de usuario web para mostrar al usuario. Vamos a volver a escribir el archivo *index.jsp* que creamos anteriormente con el código siguiente:

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/WebContent/index.jsp":::

1. Por último, escriba algo de JavaScript del lado del cliente para vincular juntos la interfaz de usuario web y el servlet:

   :::code language="java" source="~/samples-cosmosdb-java-v4-web-app/WebContent/assets/todo.js":::

1. Ahora todo lo que queda por hacer es probar la aplicación. Ejecute la aplicación localmente y agregue algunos elementos Todo rellenando el nombre del elemento y la categoría y haciendo clic en **Agregar tarea**. Una vez que aparece el elemento, puede actualizar si está completo. Para ello, alterne la casilla y haga clic en **Actualizar tareas**.

## <a name="deploy-your-java-application-to-azure-web-sites"></a><a id="Deploy"></a>Implementación de la aplicación Java en Azure Web Sites

Azure WebSites consigue que la implementación de aplicaciones de Java sea tan sencilla como exportar su aplicación como un archivo WAR y cargarla mediante el control de código fuente (por ejemplo, Git) o FTP.

1. Para exportar la aplicación como un archivo WAR, haga clic con el botón derecho en el proyecto en **Explorador de proyectos**, haga clic en **Exportar** y, a continuación, haga clic en **Archivo WAR**.

1. En la ventana **Exportar WAR** , haga lo siguiente:
   
   * En el cuadro Proyecto web, escriba azure-documentdb-java-sample.
   * En el cuadro de destino, seleccione un destino para guardar el archivo WAR.
   * Haga clic en **Finalizar**

1. Ahora que tiene a mano un archivo WAR, simplemente puede cargarlo en el directorio **webapps** del sitio web de Azure. Para obtener instrucciones sobre cómo cargar el archivo, consulte [Adición de una aplicación Java a Azure App Service Web Apps](../app-service/web-sites-java-add-app.md). Una vez que se cargue el archivo WAR en el directorio webapps, el entorno de tiempo de ejecución detectará que lo ha agregado y lo cargará automáticamente.

1. Para ver el producto finalizado, vaya a `http://YOUR\_SITE\_NAME.azurewebsites.net/azure-java-sample/` y empiece a agregar sus tareas.

## <a name="get-the-project-from-github"></a><a id="GetProject"></a>Obtenga el proyecto desde GitHub

Todos los ejemplos de este tutorial se incluyen en el proyecto [todo](https://github.com/Azure-Samples/documentdb-java-todo-app) en GitHub. Para importar el proyecto todo en Eclipse, asegúrese de disponer del software y los recursos que aparecen en la sección [Requisitos previos](#Prerequisites) y haga lo siguiente:

1. Instale [Project Lombok](https://projectlombok.org/). Lombok se utiliza para generar constructores, captadores y establecedores en el proyecto. Una vez haya descargado el archivo lombok.jar, haga doble clic en él para instalarlo o instálelo desde la línea de comandos.

1. Si Eclipse está abierto, ciérrelo y reinícielo para cargar Lombok.

1. En Eclipse, en el menú **File** (Archivo), haga clic en **Import** (Importar).

1. En la ventana **Import** (Importar), haga clic en **Git**, en **Projects from Git** (Proyectos de Git) y luego en **Next** (Siguiente).

1. En la pantalla **Select Repository Source** (Seleccionar origen del repositorio), haga clic en **Clone URI** (Clonar URI).

1. En la pantalla **Source Git Repository** (Repositorio GIT de origen), en el cuadro **URI**, escriba https://github.com/Azure-Samples/documentdb-java-todo-app.git y, a continuación, haga clic en **Siguiente**.

1. En la pantalla **Branch Selection** (Selección de rama), asegúrese de que está seleccionado **master** (principal) y luego haga clic en **Next** (Siguiente).

1. En la pantalla **Local Destination** (Destino local), haga clic en **Browse** (Examinar) para seleccionar una carpeta donde se pueda copiar el repositorio y luego haga clic en **Next** (Siguiente).

1. En la pantalla **Select a wizard to use for importing projects** (Seleccionar un asistente para usar en la importación de proyectos), asegúrese de que la opción **Import existing projects** (Importar proyectos existentes) esté seleccionada y luego haga clic en **Next** (Siguiente).

1. En la pantalla **Import Projects** (Proyectos de importación), anule la selección del proyecto **DocumentDB** y luego haga clic en **Finish** (Finalizar). El proyecto DocumentDB contiene el SDK de Java de Azure Cosmos DB, que se agregará como dependencia.

1. En **explorador de proyectos**, vaya a azure-documentdb-java-sample\src\com.microsoft.azure.documentdb.sample.dao\DocumentClientFactory.java y reemplace los valores HOST y MASTER_KEY por los de URI y PRIMARY KEY de la cuenta de Azure Cosmos DB y luego guarde el archivo. Para obtener más información, consulte el [paso 1. Cree una cuenta de base de datos de Azure Cosmos](#CreateDB).

1. En **Project Explorer** (Explorador de proyectos), haga clic con el botón derecho en **azure-documentdb-java-sample**, haga clic en **Build Path** (Ruta de acceso de compilación) y luego haga clic en **Configure Build Path** (Configurar ruta de acceso de compilación).

1. En la pantalla **Java Build Path** (Ruta de compilación de Java), en el panel derecho, seleccione la pestaña **Bibliotecas** y luego haga clic en **Add External JARs** (Agregar JAR externos). Desplácese hasta la ubicación del archivo lombok.jar y haga clic en **Abrir** y, a continuación, en **Aceptar**.

1. Use el paso 12 para abrir la ventana **Propiedades** de nuevo y, a continuación, en el panel izquierdo haga clic en **Targeted Runtimes** (Tiempos de ejecución de destino).

1. En la pantalla **Targeted Runtimes** (Tiempos de ejecución de destino), haga clic en **Nuevo**, seleccione **Apache Tomcat v7.0** y luego haga clic en **Aceptar**.

1. Use el paso 12 para abrir de nuevo la ventana **Propiedades** y, a continuación, en el panel izquierdo, haga clic en **Project Facets** (Facetas del proyecto).

1. En la pantalla **Project Facets** (Facetas del proyecto), seleccione **Dynamic Web Module** (Módulo web dinámico) y **Java** y luego haga clic en **Aceptar**.

1. En la ficha **Servidores** en la parte inferior de la pantalla, haga clic con el botón derecho en **Tomcat v7.0 Server at localhost** (Tomcat v7.0 Server en localhost) y luego haga clic en **Add and Remove** (Agregar y quitar).

1. En la ventana **Agregar y quitar**, mueva **azure-documentdb-java-sample** al cuadro **Configurado** y luego haga clic en **Finalizar**.

1. En la pestaña **Servidor**, haga clic en **Tomcat v7.0 Server at localhost** (Tomcat v7.0 Server en localhost) y luego haga clic en **Reiniciar**.

1. En un explorador, vaya a `http://localhost:8080/azure-documentdb-java-sample/` y empiece a agregar a la lista de tareas. Tenga en cuenta que si ha cambiado los valores de puerto predeterminados, debe cambiar 8080 por el valor seleccionado.

1. Para implementar el proyecto en un sitio web de Azure, consulte el [paso 6. Implementación de la aplicación para Azure WebSites](#Deploy).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una aplicación de node.js con Azure Cosmos DB](sql-api-nodejs-application.md)
