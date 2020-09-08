---
title: Tutorial sobre la aplicación web ASP.NET Core MVC con Azure Cosmos DB
description: 'Tutorial de ASP.NET Core MVC para crear una aplicación web MVC con Azure Cosmos DB. Almacenará el código JSON y accederá a los datos desde una aplicación de tareas pendientes hospedada en Azure App Service: tutorial de ASP NET Core MVC paso a paso.'
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 05/08/2020
ms.author: sngun
ms.custom: devx-track-dotnet
ms.openlocfilehash: 6772150338dd0d172f2f100c2aa8cae7175b18d6
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051308"
---
# <a name="tutorial-develop-an-aspnet-core-mvc-web-application-with-azure-cosmos-db-by-using-net-sdk"></a>Tutorial: Desarrollar una aplicación web ASP.NET Core MVC con Azure Cosmos DB mediante el SDK de .NET

> [!div class="op_single_selector"]
> * [.NET](sql-api-dotnet-application.md)
> * [Java](sql-api-java-application.md)
> * [Node.js](sql-api-nodejs-application.md)
> * [Python](sql-api-python-application.md)
> * [Xamarin](mobile-apps-with-xamarin.md)

En este tutorial aprenderá a usar Azure Cosmos DB para almacenar datos, y acceder a estos, desde una aplicación de ASP.NET MVC hospedada en Azure. En este tutorial, usará el SDK de .NET V3. En la siguiente imagen se muestra la página web que se compilará mediante el ejemplo de este artículo:

:::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-image01.png" alt-text="Captura de pantalla de la aplicación web MVC de lista de tareas pendientes creada por este tutorial: tutorial paso a paso de ASP NET Core MVC":::

Si no tiene tiempo para completar el tutorial, puede descargar el proyecto de ejemplo completo de [GitHub][GitHub].

Esta tutorial abarca lo siguiente:

> [!div class="checklist"]
>
> * Creación de una cuenta de Azure Cosmos
> * Creación de una aplicación ASP.NET Core MVC
> * Conexión de la aplicación a Azure Cosmos DB
> * Operaciones de creación, lectura, actualización y eliminación (CRUD) en los datos

> [!TIP]
> En este tutorial se supone que tiene experiencia previa con ASP.NET Core MVC y Azure App Service. Si no está familiarizado con ASP.NET Core o las [herramientas que son requisito previo](#prerequisites), recomendamos que descargue el proyecto de ejemplo completo de [GitHub][GitHub], que agregue los paquetes NuGet necesarios y que lo ejecute. Una vez compilado el proyecto, puede revisar este artículo para obtener información sobre el código en el contexto del proyecto.

## <a name="prerequisites"></a><a name="prerequisites"></a>Requisitos previos

Antes de seguir las instrucciones del presente artículo, asegúrese de tener los siguientes recursos:

* Una cuenta de Azure activa. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* Visual Studio 2019. [!INCLUDE [cosmos-db-emulator-vs](../../includes/cosmos-db-emulator-vs.md)]  

Todas las capturas de pantallas de este artículo son de Microsoft Visual Studio Community 2019. Si usa una versión diferente, es posible que las pantallas y las opciones no coincidan exactamente. La solución funcionará si cumple los requisitos previos.

## <a name="step-1-create-an-azure-cosmos-account"></a><a name="create-an-azure-cosmos-account"></a>Paso 1: Creación de una cuenta de Azure Cosmos

Para comenzar, crearemos una cuenta de Azure Cosmos. Si ya tiene una cuenta de SQL API en Azure Cosmos DB o si usa el emulador de Azure Cosmos DB, vaya al [paso 2: Creación de una aplicación ASP.NET MVC](#create-a-new-mvc-application).

[!INCLUDE [create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

[!INCLUDE [keys](../../includes/cosmos-db-keys.md)]

En la siguiente sección, creará una aplicación de ASP.NET Core MVC.

## <a name="step-2-create-a-new-aspnet-core-mvc-application"></a><a name="create-a-new-mvc-application"></a>Paso 2: Crear una aplicación ASP.NET Core MVC nueva

1. Abra Visual Studio y seleccione **Crear un proyecto**.

1. En **Crear un proyecto**, busque y elija **Aplicación web ASP.NET Core** para C#. Seleccione **Next** (Siguiente) para continuar.

   :::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-new-project-dialog.png" alt-text="Creación de un proyecto de aplicación web de ASP.NET Core":::

1. En **Configure su nuevo proyecto**, asigne al proyecto el nombre *todo* y seleccione **Crear**.

1. En **Crear una nueva aplicación web ASP.NET Core**, elija **Aplicación web (controlador de vista de modelos)** . Seleccione **Crear** para continuar.

   Visual Studio crea una aplicación MVC vacía.

1. Seleccione **Depurar** > **Iniciar depuración** o F5 para ejecutar la aplicación ASP.NET localmente.

## <a name="step-3-add-azure-cosmos-db-nuget-package-to-the-project"></a><a name="add-nuget-packages"></a>Paso 3: Incorporación del paquete NuGet de Azure Cosmos DB al proyecto

Ahora que tenemos la mayoría del código de plataforma de ASP.NET Core MVC que necesitamos para esta solución, vamos a agregar los paquetes NuGet necesarios para conectarse a Azure Cosmos DB.

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Administrar paquetes NuGet**.

1. En **Administrador de paquetes NuGet**, busque y seleccione **Microsoft.Azure.Cosmos**. Seleccione **Instalar**.

   :::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-nuget.png" alt-text="Instalación de un paquete NuGet":::

   Visual Studio descarga e instala el paquete de Azure Cosmos DB y sus dependencias.

   También puede usar la **Consola del Administrador de paquetes** para instalar el paquete NuGet. Para ello, seleccione **Herramientas** > **Administrador de paquetes NuGet** > **Consola del Administrador de paquetes**. En el símbolo del sistema, escriba el siguiente comando:

   ```ps
   Install-Package Microsoft.Azure.Cosmos
   ```
  
## <a name="step-4-set-up-the-aspnet-core-mvc-application"></a><a name="set-up-the-mvc-application"></a>Paso 4: Configurar la aplicación ASP.NET Core MVC

Ahora vamos a agregar los modelos, las vistas y los controladores a esta aplicación MVC.

### <a name="add-a-model"></a><a name="add-a-model"></a> Incorporación de un modelo

1. En el **Explorador de soluciones**, haga clic con el botón derecho en la carpeta **Models** y seleccione **Agregar** > **Clase**.

1. En **Agregar nuevo elemento**, asigne a la nueva clase el nombre *Item.cs* y seleccione **Agregar**.

1. Reemplace el contenido de la clase *Item.cs* por el código siguiente:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Models/Item.cs":::

Azure Cosmos DB usa JSON para trasladar y almacenar los datos. Puede usar el atributo `JsonProperty` para controlar el modo en que JSON serializa y deserializa los objetos. La clase `Item` muestra el atributo `JsonProperty`. Este código controla el formato del nombre de propiedad que entra en JSON. También cambia el nombre de la propiedad `Completed` de .NET.

### <a name="add-views"></a><a name="add-views"></a>Adición de vistas

A continuación, se agregarán las siguientes vistas.

* Una vista Crear elemento
* Una vista Eliminar elemento
* Una vista para obtener los detalles del elemento
* Una vista Editar elemento
* Una vista para enumerar todos los elementos

#### <a name="create-item-view"></a><a name="AddNewIndexView"></a>Vista Crear elemento

1. En el **Explorador de soluciones**, haga clic con el botón derecho en la carpeta **Views** y seleccione **Agregar** > **Nueva carpeta**. Asigne a la carpeta el nombre *Item*.

1. Haga clic con el botón derecho en la carpeta vacía **Item** y seleccione **Agregar** > **Vista**.

1. En **Agregar vista MVC**, realice los cambios siguientes:

   * En **Nombre de la vista**, escriba *Crear*.
   * En **Plantilla**, seleccione **Crear**.
   * En **Clase de modelo**, seleccione **Item (todo.Models)** .
   * Seleccione **Usar una página de diseño** y escriba *~/Views/Shared/_Layout.cshtml*.
   * Seleccione **Agregar**.

   :::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-add-mvc-view.png" alt-text="Captura de pantalla que muestra el cuadro de diálogo para agregar una vista MVC":::

1. Después, seleccione **Agregar** y permita que Visual Studio cree una vista de plantilla. Reemplace el código del archivo generado por el siguiente contenido:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Views/Item/Create.cshtml":::

#### <a name="delete-item-view"></a><a name="AddEditIndexView"></a>Vista Eliminar elemento

1. En el **Explorador de soluciones**, vuelva a hacer clic con el botón derecho en la carpeta **Item** y seleccione **Agregar** > **Vista**.

1. En **Agregar vista MVC**, realice los cambios siguientes:

   * En el cuadro **Nombre de vista**, escriba *Delete* (Eliminar).
   * En el cuadro **Plantilla**, seleccione **Delete** (Eliminar).
   * En el cuadro **Clase de modelo**, seleccione **Elemento (todo.Models)** .
   * Seleccione **Usar una página de diseño** y escriba *~/Views/Shared/_Layout.cshtml*.
   * Seleccione **Agregar**.

1. Después, seleccione **Agregar** y permita que Visual Studio cree una vista de plantilla. Reemplace el código del archivo generado por el siguiente contenido:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Views/Item/Delete.cshtml":::

#### <a name="add-a-view-to-get-an-item-details"></a><a name="AddItemIndexView"></a>Adición de una vista para obtener los detalles de un elemento

1. En el **Explorador de soluciones**, vuelva a hacer clic con el botón derecho en la carpeta **Item** y seleccione **Agregar** > **Vista**.

1. En **Agregar vista MVC**, proporcione los siguientes valores:

   * En **Nombre de la vista**, escriba *Details* (Detalles).
   * En **Plantilla**, seleccione **Details** (Detalles).
   * En **Clase de modelo**, seleccione **Item (todo.Models)** .
   * Seleccione **Usar una página de diseño** y escriba *~/Views/Shared/_Layout.cshtml*.

1. Después, seleccione **Agregar** y permita que Visual Studio cree una vista de plantilla. Reemplace el código del archivo generado por el siguiente contenido:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Views/Item/Details.cshtml":::

#### <a name="add-an-edit-item-view"></a><a name="AddEditIndexView"></a>Agregar una vista de edición de elementos

1. En el **Explorador de soluciones**, vuelva a hacer clic con el botón derecho en la carpeta **Item** y seleccione **Agregar** > **Vista**.

1. En **Agregar vista MVC**, realice los cambios siguientes:

   * En el cuadro **Nombre de vista**, escriba *Editar*.
   * En el cuadro **Plantilla**, seleccione **Editar**.
   * En el cuadro **Clase de modelo**, seleccione **Elemento (todo.Models)** .
   * Seleccione **Usar una página de diseño** y escriba *~/Views/Shared/_Layout.cshtml*.
   * Seleccione **Agregar**.

1. Después, seleccione **Agregar** y permita que Visual Studio cree una vista de plantilla. Reemplace el código del archivo generado por el siguiente contenido:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Views/Item/Edit.cshtml":::

#### <a name="add-a-view-to-list-all-the-items"></a><a name="AddEditIndexView"></a>Adición de una vista para enumerar todos los elementos

Y, por último, agregue una vista para obtener todos los elementos con los pasos siguientes:

1. En el **Explorador de soluciones**, vuelva a hacer clic con el botón derecho en la carpeta **Item** y seleccione **Agregar** > **Vista**.

1. En **Agregar vista MVC**, realice los cambios siguientes:

   * En el cuadro **Nombre de vista**, escriba *Índice*.
   * En el cuadro **Plantilla**, seleccione **Lista**.
   * En el cuadro **Clase de modelo**, seleccione **Elemento (todo.Models)** .
   * Seleccione **Usar una página de diseño** y escriba *~/Views/Shared/_Layout.cshtml*.
   * Seleccione **Agregar**.

1. Después, seleccione **Agregar** y permita que Visual Studio cree una vista de plantilla. Reemplace el código del archivo generado por el siguiente contenido:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Views/Item/Index.cshtml":::

Una vez hecho esto, cierre todos los documentos *cshtml* en Visual Studio.

### <a name="declare-and-initialize-services"></a><a name="initialize-services"></a>Declaración e inicialización de servicios

Primero, vamos a agregar una clase que contenga la lógica para conectarse a Azure Cosmos DB y usarlo. En este tutorial, se encapsulará esta lógica en una clase llamada `CosmosDBService` y una interfaz llamada `ICosmosDBService`. Este servicio realiza las operaciones CRUD. También realiza las operaciones de fuente de lectura, como enumeración de elementos incompletos y creación, edición y eliminación de los elementos.

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Agregar** > **Nueva carpeta**. Asigne a la carpeta el nombre *Services*.

1. Haga clic con el botón derecho en la carpeta **Services** y seleccione **Agregar** > **Clase**. Asigne a la nueva clase el nombre *CosmosDBService* y seleccione **Agregar**.

1. Reemplace el contenido de *CosmosDBService.cs* por el código siguiente:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Services/CosmosDbService.cs":::

1. Haga clic con el botón derecho en la carpeta **Services** y seleccione **Agregar** > **Clase**. Asigne a la nueva clase el nombre *ICosmosDBService* y seleccione **Agregar**.

1. Agregue el código siguiente a clase *ICosmosDBService*:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Services/ICosmosDbService.cs":::

1. Abra el archivo *Startup.cs* en la solución y agregue el siguiente método **InitializeCosmosClientInstanceAsync**, que lee la configuración e inicializa el cliente.

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Startup.cs" id="InitializeCosmosClientInstanceAsync" :::

1. En el mismo archivo, reemplace el método `ConfigureServices` por:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Startup.cs" id="ConfigureServices":::

   El código de este paso inicializa el cliente en función de la configuración como una instancia singleton que se va a insertar mediante la [inserción de dependencias en ASP.NET Core](https://docs.microsoft.com/aspnet/core/fundamentals/dependency-injection).

   Y asegúrese de cambiar el controlador MVC predeterminado a `Item`, editando las rutas en el método `Configure` del mismo archivo:

   ```csharp
    app.UseEndpoints(endpoints =>
          {
                endpoints.MapControllerRoute(
                   name: "default",
                   pattern: "{controller=Item}/{action=Index}/{id?}");
          });
   ```


1. Defina la configuración en el archivo *appsettings.json* del proyecto, tal y como se muestra en el siguiente fragmento de código:

   :::code language="json" source="~/samples-cosmosdb-dotnet-core-web-app/src/appsettings.json":::

### <a name="add-a-controller"></a><a name="add-a-controller"></a>Adición de un controlador

1. En el **Explorador de soluciones**, haga clic con el botón derecho en la carpeta **Controllers** y seleccione **Agregar** > **Controlador**.

1. En **Agregar scaffold**, seleccione **Controlador de MVC: en blanco** y **Agregar**.

   :::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-controller-add-scaffold.png" alt-text="Selección del controlador de MVC: vacío en Agregar scaffold":::

1. Asigne un nombre al nuevo controlador, *ItemController*.

1. Reemplace el contenido de *ItemController.cs* por el código siguiente:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Controllers/ItemController.cs":::

El atributo **ValidateAntiForgeryToken** se usa aquí para ayudar a proteger esta aplicación frente a los ataques de falsificación de solicitud entre sitios. Las vistas funcionarán también con este token antifalsificación. Para obtener más información y ejemplos, vea [Prevención de ataques de falsificación (CSRF) de solicitudes entre sitios en la aplicación ASP.NET MVC][Preventing Cross-Site Request Forgery]. El código de origen proporcionado en [GitHub][GitHub] tiene la implementación completa en su lugar.

También se usa el atributo **Bind** en el parámetro de método para ayudar a proteger contra ataques de publicación en exceso. Para más información, consulte el [Tutorial: Implementar la funcionalidad CRUD con Entity Framework en ASP.NET MVC][Basic CRUD Operations in ASP.NET MVC].

## <a name="step-5-run-the-application-locally"></a><a name="run-the-application"></a>Paso 5: Ejecución de la aplicación de forma local

Para probar la aplicación en el equipo local, siga estos pasos:

1. Presione F5 en Visual Studio para compilar la aplicación en modo de depuración. De esa forma, se debe compilar la aplicación e iniciar un explorador con la página de cuadrícula vacía que vimos anteriormente:

   :::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-create-an-item-a.png" alt-text="Captura de pantalla de la aplicación web de lista de tareas pendientes creada con este tutorial":::
   
   Si en su lugar se abre la aplicación en la página principal, anexe `/Item` a la dirección URL.

1. Seleccione el vínculo **Crear nuevo** y agregue valores a los campos **Nombre** y **Descripción**. Deje desactivada la casilla **Completado**. Cuando está activada, la aplicación agrega el nuevo elemento en estado completado. El elemento no aparece en la lista inicial.

1. Seleccione **Crear**. La aplicación le devuelve a la vista **Índice** y el elemento aparece en la lista. Puede agregar unos cuantos elementos más a la lista **To-Do** (Tareas pendientes).

    :::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-create-an-item.png" alt-text="Captura de pantalla de la vista de índice":::
  
1. Seleccione **Editar** junto a un **elemento** de la lista. La aplicación abre la vista **Edición**, donde puede actualizar cualquier propiedad del objeto, incluida la marca **Completado**. Si selecciona **Completado** y selecciona **Guardar**, la aplicación muestra el **elemento** como completada en la lista.

   :::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-completed-item.png" alt-text="Captura de pantalla de la vista de índice con el cuadro Completado activado":::

1. Compruebe el estado de los datos en el servicio Azure Cosmos DB con [Cosmos Explorer](https://cosmos.azure.com) o el Explorador de datos del Emulador de Azure Cosmos DB.

1. Una vez que haya probado la aplicación, seleccione Ctrl+F5 para detener la depuración de la aplicación. Está listo para la implementación.

## <a name="step-6-deploy-the-application"></a><a name="deploy-the-application-to-azure"></a>Paso 6: Implementación de la aplicación

Ahora que toda la aplicación funciona correctamente con Azure Cosmos DB, vamos a implementar esta aplicación web en Azure App Service.  

1. Para publicar esta aplicación, haga clic con el botón derecho en el proyecto en el **Explorador de soluciones** y seleccione **Publicar**.

1. En **Elegir un destino de publicación**, seleccione **App Service**.

1. Para usar un perfil de App Service existente, elija **Seleccionar existente** y, a continuación, seleccione **Publicar**.

1. En **App Service**, seleccione una **Suscripción**. Use el filtro **Ver** para ordenar por grupo de recursos o tipo de recurso.

1. Busque el perfil y seleccione **Aceptar**. A continuación, busque la instancia de Azure App Service necesaria y seleccione **Aceptar**.

   :::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-app-service-2019.png" alt-text="Cuadro de diálogo App Service en Visual Studio":::

Otra opción es crear un nuevo perfil:

1. Como en el procedimiento anterior, haga clic con el botón derecho en el proyecto en el **Explorador de soluciones** y seleccione **Publicar**.
  
1. En **Elegir un destino de publicación**, seleccione **App Service**.

1. En **Elegir un destino de publicación**, seleccione **Crear nuevo** y seleccione **Publicar**.

1. En **App Service**, escriba el nombre de la aplicación web y la suscripción, el grupo de recursos y el plan de hospedaje adecuados y, luego, haga clic en **Crear**.

   :::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-create-app-service-2019.png" alt-text="Cuadro de diálogo Crear servicio de aplicaciones en Visual Studio":::

En unos segundos, Visual Studio publica la aplicación web e inicia un explorador donde puede ver su trabajo ejecutándose en Azure.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a crear una aplicación web MVC ASP.NET Core. La aplicación puede acceder a los datos almacenados en Azure Cosmos DB. Ahora puede continuar con estos recursos:

* [Creación de particiones en Azure Cosmos DB](./partitioning-overview.md)
* [Introducción a las consultas SQL](./how-to-sql-query.md)
* [Procedimientos para modelar y crear particiones de datos en Azure Cosmos DB mediante un ejemplo real](./how-to-model-partition-example.md)

[Visual Studio Express]: https://www.visualstudio.com/products/visual-studio-express-vs.aspx
[Microsoft Web Platform Installer]: https://www.microsoft.com/web/downloads/platform.aspx
[Preventing Cross-Site Request Forgery]: https://docs.microsoft.com/aspnet/web-api/overview/security/preventing-cross-site-request-forgery-csrf-attacks
[Basic CRUD Operations in ASP.NET MVC]: https://go.microsoft.com/fwlink/?LinkId=317598
[GitHub]: https://github.com/Azure-Samples/cosmos-dotnet-core-todo-app
