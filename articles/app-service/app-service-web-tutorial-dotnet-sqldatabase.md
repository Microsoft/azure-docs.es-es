---
title: 'Compilación de una aplicación ASP.NET con SQL Database: Azure App Service | Microsoft Docs'
description: Aprenda a implementar una aplicación de ASP.NET para C# con una base de datos SQL Server en Azure.
services: app-service\web
documentationcenter: ''
author: cephalin
manager: cfowler
editor: ''
ms.assetid: 03c584f1-a93c-4e3d-ac1b-c82b50c75d3e
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: csharp
ms.topic: tutorial
ms.date: 06/25/2018
ms.author: cephalin
ms.custom: seodec18
ms.openlocfilehash: f010e7564f097f28269070d85e2895e73f454054
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415570"
---
# <a name="tutorial-build-an-aspnet-app-in-azure-with-sql-database"></a>Tutorial: Compilación de una aplicación ASP.NET en Azure con SQL Database

[Azure App Service](overview.md) proporciona un servicio de hospedaje web muy escalable y con aplicación de revisiones de un modo automático. En este tutorial se muestra cómo implementar una aplicación ASP.NET orientada a datos en App Service y conectarla a [Azure SQL Database](../sql-database/sql-database-technical-overview.md). Cuando haya terminado, tendrá una aplicación ASP.NET en ejecución en Azure y conectada a SQL Database.

![Aplicación de ASP.NET publicada en Azure App Service](./media/app-service-web-tutorial-dotnet-sqldatabase/azure-app-in-browser.png)

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una base de datos Azure SQL Database
> * Conectar una aplicación ASP.NET a SQL Database
> * Implementación de la aplicación en Azure
> * Actualizar el modelo de datos y volver a implementar la aplicación
> * Transmitir registros desde Azure a un terminal
> * Administrar la aplicación en Azure Portal

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial:

Instalar <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2019</a> con la carga de trabajo de **ASP.NET y desarrollo web**.

Si ya ha instalado Visual Studio, agregue las cargas de trabajo en Visual Studio, para lo que debe hacer clic en **Herramientas** > **Get Tools and Features** (Obtener herramientas y características).

## <a name="download-the-sample"></a>Descarga del ejemplo

- [Descarga del proyecto de ejemplo](https://github.com/Azure-Samples/dotnet-sqldb-tutorial/archive/master.zip)
- Extraiga (descomprima) el archivo  *dotnet-sqldb-tutorial-master.zip*.

El proyecto de ejemplo contiene una aplicación básica CRUD (crear, leer, actualizar, eliminar) de [ASP.NET MVC](https://www.asp.net/mvc) que usa [Entity Framework Code First](/aspnet/mvc/overview/getting-started/getting-started-with-ef-using-mvc/creating-an-entity-framework-data-model-for-an-asp-net-mvc-application).

### <a name="run-the-app"></a>Ejecución de la aplicación

Abra el archivo *dotnet-sqldb-tutorial-master/DotNetAppSqlDb.sln* en Visual Studio. 

Escriba `Ctrl+F5` para ejecutar la aplicación sin depurarla. La aplicación se muestra en el explorador predeterminado. Seleccione el vínculo **Crear nuevo** y cree un par de elementos de *tareas pendientes*. 

![Cuadro de diálogo Nuevo proyecto de ASP.NET](media/app-service-web-tutorial-dotnet-sqldatabase/local-app-in-browser.png)

Pruebe los vínculos **Editar**, **Detalles** y **Eliminar**.

La aplicación usa un contexto de base de datos para conectarse con la base de datos. En este ejemplo, el contexto de base de datos emplea una cadena de conexión llamada `MyDbConnection`. La cadena de conexión se establece en el archivo *Web.config* y se hace referencia a ella en el archivo *Models/MyDatabaseContext.cs*. El nombre de la cadena de conexión se usa más adelante en el tutorial para conectar la aplicación de Azure a una base de datos de Azure SQL. 

## <a name="publish-to-azure-with-sql-database"></a>Publicación en Azure con SQL Database

En el **Explorador de soluciones**, haga clic con el botón derecho en su proyecto **DotNetAppSqlDb** y seleccione **Publicar**.

![Publicar desde el Explorador de soluciones](./media/app-service-web-tutorial-dotnet-sqldatabase/solution-explorer-publish.png)

Asegúrese de que **Microsoft Azure App Service** está seleccionado y haga clic en **Publicar**.

![Publicar desde la página de información general del proyecto](./media/app-service-web-tutorial-dotnet-sqldatabase/publish-to-app-service.png)

Al publicar se abre el cuadro de diálogo **Crear servicio de aplicaciones**, que le ayuda a crear todos los recursos de Azure necesarios para ejecutar la aplicación ASP.NET en Azure.

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

En el cuadro de diálogo **Crear App Service**, haga clic en **Agregar una cuenta** y, a continuación, inicie sesión en su suscripción de Azure. Si ya ha iniciado sesión en una cuenta de Microsoft, asegúrese de que esa cuenta contiene la suscripción de Azure. Si la cuenta de Microsoft con la que inició sesión no contiene su suscripción de Azure, haga clic en ella para agregar la cuenta correcta. 

> [!NOTE]
> Si ya ha iniciado sesión, no seleccione **Crear** todavía.
>
>
   
![Inicio de sesión en Azure](./media/app-service-web-tutorial-dotnet-sqldatabase/sign-in-azure.png)

### <a name="configure-the-web-app-name"></a>Configuración del nombre de la aplicación web

Puede mantener el nombre de aplicación web generado, o cambiarlo por otro nombre único (los caracteres válidos son `a-z`, `0-9` y `-`). El nombre de la aplicación web se usa como parte de la dirección URL predeterminada en la aplicación (`<app_name>.azurewebsites.net`, donde `<app_name>` es el nombre de la aplicación web). El nombre de la aplicación web debe ser único entre todas las aplicaciones de Azure. 

![Cuadro de diálogo Create App Service (Crear App Service)](media/app-service-web-tutorial-dotnet-sqldatabase/wan.png)

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

[!INCLUDE [resource-group](../../includes/resource-group.md)]

Junto a **Grupo de recursos**, haga clic en **Nuevo**.

![Junto a Grupo de recursos, haga clic en Nuevo.](media/app-service-web-tutorial-dotnet-sqldatabase/new_rg2.png)

Llame al grupo de recursos **myResourceGroup**.

### <a name="create-an-app-service-plan"></a>Creación de un plan de App Service

[!INCLUDE [app-service-plan](../../includes/app-service-plan.md)]

Junto a **Plan de App Service**, haga clic en **Nuevo**. 

En el cuadro de diálogo **Configurar el plan de App Service**, configure el nuevo plan de App Service con los valores siguientes:

![Creación de un plan de App Service](./media/app-service-web-tutorial-dotnet-sqldatabase/configure-app-service-plan.png)

| Configuración  | Valor sugerido | Para obtener más información |
| ----------------- | ------------ | ----|
|**Plan de App Service**| myAppServicePlan | [Planes de App Service](../app-service/overview-hosting-plans.md) |
|**Ubicación**| Europa occidental | [Regiones de Azure](https://azure.microsoft.com/regions/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) |
|**Tamaño**| Gratuito | [Planes de tarifa](https://azure.microsoft.com/pricing/details/app-service/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)|

### <a name="create-a-sql-server-instance"></a>Creación de una instancia de SQL Server

Antes de crear una base de datos, necesita un [servidor lógico de Azure SQL Database](../sql-database/sql-database-features.md). Un servidor lógico contiene un conjunto de bases de datos administradas como un grupo.

Haga clic en **Create a SQL Database** (Crear una instancia de SQL Database).

![Creación de una instancia de SQL Database](media/app-service-web-tutorial-dotnet-sqldatabase/web-app-name.png)

En el cuadro de diálogo **Configurar SQL Database**, haga clic en **Nuevo** junto a **SQL Server**. 

Se genera un nombre de servidor único. Este nombre se usa como parte de la dirección URL predeterminada del servidor lógico, `<server_name>.database.windows.net`. Debe ser único entre todas las instancias de servidor lógico de Azure. Este nombre se puede cambiar, pero para los fines de este tutorial, conserve el valor generado.

Agregue un nombre de usuario y una contraseña de administrador. Para conocer los requisitos de complejidad de la contraseña, consulte [Directivas de contraseñas](/sql/relational-databases/security/password-policy).

Recuerde este nombre de usuario y esta contraseña. Los necesitará más adelante para administrar la instancia del servidor lógico.

> [!IMPORTANT]
> Aunque la contraseña de las cadenas de conexión está enmascarada (en Visual Studio y también en App Service), el hecho de que se conserva en otro lugar se suma a la superficie expuesta a ataques de la aplicación. App Service puede usar [identidades de servicio administradas](overview-managed-identity.md) para quitar este riesgo eliminando por completo la necesidad de conservar secretos en el código o en la configuración de la aplicación. Para más información, consulte la sección [Pasos siguientes](#next-steps).

![Creación de una instancia de SQL Server](media/app-service-web-tutorial-dotnet-sqldatabase/configure-sql-database-server.png)

Haga clic en **OK**. No cierre aún el cuadro de diálogo **Configurar base de datos SQL**.

### <a name="create-a-sql-database"></a>Creación de una instancia de SQL Database

En el cuadro de diálogo **Configurar SQL Database**: 

* Mantenga el **nombre de base de datos** generado predeterminado.
* En **Nombre de la cadena de conexión**, escriba *MyDbConnection*. Este nombre tiene que coincidir con la cadena de conexión a la que se hace referencia en *Models\MyDatabaseContext.cs*.
* Seleccione **Aceptar**.

![Configuración de SQL Database](media/app-service-web-tutorial-dotnet-sqldatabase/configure-sql-database.png)

El cuadro de diálogo **Create App Service** (Crear App Service) muestra los recursos que ha configurado. Haga clic en **Create**(Crear). 

![Se muestran los recursos que ha creado.](media/app-service-web-tutorial-dotnet-sqldatabase/app_svc_plan_done.png)

Cuando el asistente termina de crear los recursos de Azure, publica la aplicación ASP.NET en Azure. El explorador predeterminado se inicia con la dirección URL a la aplicación implementada. 

Agregue algunos elementos de tareas pendientes.

![Aplicación de ASP.NET publicada en la aplicación de Azure](./media/app-service-web-tutorial-dotnet-sqldatabase/azure-app-in-browser.png)

Felicidades. La aplicación ASP.NET orientada a datos se ejecuta en directo en Azure App Service.

## <a name="access-the-sql-database-locally"></a>Acceso local a la instancia de SQL Database

Visual Studio le permite explorar y administrar su nueva instancia de SQL Database de manera fácil en el **Explorador de objetos de SQL Server**.

### <a name="create-a-database-connection"></a>Creación de una conexión de base de datos

En el menú **Ver**, seleccione **Explorador de objetos de SQL Server**.

En la parte superior del **Explorador de objetos de SQL Server**, haga clic en el botón **Agregar SQL Server**.

### <a name="configure-the-database-connection"></a>Configuración de la conexión de base de datos

En el cuadro de diálogo **Conectar**, expanda el nodo **Azure**. Aquí se muestran todas las instancias de Azure SQL Database.

Seleccione la instancia de SQL Database que creó anteriormente. La conexión que creó anteriormente se rellena automáticamente en la parte inferior.

Escriba la contraseña de administrador de base de datos que creó anteriormente y haga clic en **Conectar**.

![Configuración de la conexión de base de datos de Visual Studio](./media/app-service-web-tutorial-dotnet-sqldatabase/connect-to-sql-database.png)

### <a name="allow-client-connection-from-your-computer"></a>Permitir la conexión de cliente desde el equipo

Se abre el cuadro de diálogo **Create a new firewall rule** (Crear nueva regla de firewall). De forma predeterminada, la instancia de SQL Database solo permite conexiones desde servicios de Azure, como la aplicación de Azure. Para conectarse a la base de datos, cree una regla de firewall en la instancia de SQL Database. La regla de firewall permite la dirección IP pública del equipo local.

El cuadro de diálogo ya se ha rellenado con la dirección IP pública de su equipo.

Asegúrese de que **Add my client IP** (Agregar mi IP de cliente) está seleccionado y haga clic en **Aceptar**. 

![Configuración del firewall para la instancia de SQL Database](./media/app-service-web-tutorial-dotnet-sqldatabase/sql-set-firewall.png)

Una vez que Visual Studio termina de crear la configuración de firewall para su instancia de SQL Database, la conexión se muestra en el **Explorador de objetos de SQL Server**.

Aquí, puede realizar las operaciones de base de datos más comunes, como ejecutar consultas, crear vistas y procedimientos almacenados, entre otras. 

Expanda la conexión > **Bases de datos** > **&lt;base de datos >** > **Tablas**. Haga clic con el botón derecho en la tabla `Todoes` y seleccione **Ver datos**. 

![Exploración de objetos de SQL Database](./media/app-service-web-tutorial-dotnet-sqldatabase/explore-sql-database.png)

## <a name="update-app-with-code-first-migrations"></a>Actualización de aplicaciones con Migraciones de Code First

Puede usar las conocidas herramientas de Visual Studio para actualizar la base de datos y la aplicación en Azure. En este paso, usará Migraciones de Code First en Entity Framework para realizar un cambio en el esquema de la base de datos y publicarlo en Azure.

Para más información sobre el uso de Entity Framework Code First Migrations, consulte [Getting Started with Entity Framework 6 Code First using MVC 5](https://docs.microsoft.com/aspnet/mvc/overview/getting-started/getting-started-with-ef-using-mvc/creating-an-entity-framework-data-model-for-an-asp-net-mvc-application) (Introducción a Entity Framework 6 Code First mediante MVC 5).

### <a name="update-your-data-model"></a>Actualización del modelo de datos

Abra _Models\Todo.cs_ en el editor de código. Agregue la siguiente propiedad a la clase `ToDo`:

```csharp
public bool Done { get; set; }
```

### <a name="run-code-first-migrations-locally"></a>Ejecución de Migraciones de Code First

Ejecute algunos comandos para realizar actualizaciones en la base de datos local. 

En el menú **Herramientas**, haga clic en **Administrador de paquetes NuGet** > **Consola del Administrador de paquetes**.

En la ventana de la consola del Administrador de paquetes, habilite Migraciones de Code First:

```powershell
Enable-Migrations
```

Agregue una migración:

```powershell
Add-Migration AddProperty
```

Actualice la base de datos local:

```powershell
Update-Database
```

Escriba `Ctrl+F5` para ejecutar la aplicación. Pruebe los vínculos de edición, detalles y creación.

Si la aplicación se carga sin errores, la característica Migraciones de Code First ha funcionado correctamente. Sin embargo, la página todavía tiene el mismo aspecto porque la aplicación lógica no usa aún esta propiedad. 

### <a name="use-the-new-property"></a>Uso de la nueva propiedad

Realice algunos cambios en el código para usar la propiedad `Done`. Para simplificar, en este tutorial va a cambiar las vistas `Index` y `Create` para ver la propiedad en acción.

Abra _Controllers\TodosController.cs_.

Busque el método `Create()` en la línea 52 y agregue `Done` a la lista de propiedades del atributo `Bind`. Cuando haya terminado, la firma del método `Create()` se parecerá al código siguiente:

```csharp
public ActionResult Create([Bind(Include = "Description,CreatedDate,Done")] Todo todo)
```

Abra _Views\Todos\Create.cshtml_.

En el código Razor, debería ver un elemento `<div class="form-group">` que usa `model.Description` y, a continuación, otro elemento `<div class="form-group">` que usa `model.CreatedDate`. Inmediatamente después de estos dos elementos, agregue otro elemento `<div class="form-group">` que use `model.Done`:

```csharp
<div class="form-group">
    @Html.LabelFor(model => model.Done, htmlAttributes: new { @class = "control-label col-md-2" })
    <div class="col-md-10">
        <div class="checkbox">
            @Html.EditorFor(model => model.Done)
            @Html.ValidationMessageFor(model => model.Done, "", new { @class = "text-danger" })
        </div>
    </div>
</div>
```

Abra _Views\Todos\Index.cshtml_.

Busque el elemento vacío `<th></th>`. Justo encima de este elemento, agregue el siguiente código Razor:

```csharp
<th>
    @Html.DisplayNameFor(model => model.Done)
</th>
```

Busque el elemento `<td>` que contiene los métodos auxiliares `Html.ActionLink()`. _Encima_ del elemento `<td>`, agregue otro `<td>` con el código Razor siguiente:

```csharp
<td>
    @Html.DisplayFor(modelItem => item.Done)
</td>
```

Eso es todo lo que necesita para ver los cambios en las vistas `Index` y `Create`. 

Escriba `Ctrl+F5` para ejecutar la aplicación.

Ahora puede agregar una tarea pendiente y marcar **Listo**. A continuación se debería mostrar en su página principal como un elemento completado. Recuerde que la vista `Edit` no muestra el campo `Done`, dado que no cambió la vista `Edit`.

### <a name="enable-code-first-migrations-in-azure"></a>Habilitación de Migraciones de Code First en Azure

Ahora que el cambio de código funciona, incluida la migración de la base de datos, lo publicará en la aplicación de Azure y actualizará también su instancia de SQL Database con Migraciones de Code First.

Igual que antes, haga clic con el botón derecho en su proyecto y seleccione **Publicar**.

Haga clic en **Configurar** para abrir la configuración de publicación.

![Abrir la configuración de publicación](./media/app-service-web-tutorial-dotnet-sqldatabase/publish-settings.png)

En el asistente, haga clic en **Siguiente**.

Asegúrese de que la cadena de conexión de su instancia de SQL Database se rellena en **MyDatabaseContext (MyDbConnection)**. Debe seleccionar la base de datos **myToDoAppDb** en la lista desplegable. 

Seleccione **Ejecutar Migraciones de Code First (se ejecuta al iniciar la aplicación)** y luego haga clic en **Guardar**.

![Habilitación de Migraciones de Code First en la aplicación de Azure](./media/app-service-web-tutorial-dotnet-sqldatabase/enable-migrations.png)

### <a name="publish-your-changes"></a>Publicación de los cambios

Ahora que ha habilitado Migraciones de Code First en su aplicación de Azure, publique los cambios en el código.

En la página de publicación, haga clic en **Publicar**.

Intente volver a agregar elementos de tareas pendientes y seleccione **Listo**; se mostrarán en su página de inicio como un elemento completado.

![Aplicación de Azure después de Migraciones de Code First](./media/app-service-web-tutorial-dotnet-sqldatabase/this-one-is-done.png)

Aún se muestran todas las tareas pendientes existentes. Cuando vuelva a publicar su aplicación de ASP.NET, los datos existentes en su instancia de SQL Database no se perderán. Además, Migraciones de Code First solo cambia el esquema de datos y deja intactos los datos existentes.


## <a name="stream-application-logs"></a>Transmisión de registros de aplicación

Puede transmitir mensajes de seguimiento directamente desde la aplicación de Azure hasta Visual Studio.

Abra _Controllers\TodosController.cs_.

Cada acción comienza con un método `Trace.WriteLine()`. Este código se agrega para mostrarle cómo agregar mensajes de seguimiento a la aplicación de Azure.

### <a name="open-server-explorer"></a>Apertura del Explorador de servidores

En el menú **Ver**, seleccione **Explorador de servidores**. Puede configurar el registro de la aplicación de Azure en el **Explorador de servidores**. 

### <a name="enable-log-streaming"></a>Habilitación de la secuencia de registros

En el **Explorador de servidores**, expanda **Azure** > **App Service**.

Expanda el grupo de recursos **myResourceGroup** que creó cuando creó la aplicación de Azure por primera vez.

Haga clic con el botón derecho en la aplicación de Azure y seleccione **Ver registros de streaming**.

![Habilitación de la secuencia de registros](./media/app-service-web-tutorial-dotnet-sqldatabase/stream-logs.png)

Los registros se insertan ahora en la ventana **Salida**. 

![Secuencia de registros en la ventana Salida](./media/app-service-web-tutorial-dotnet-sqldatabase/log-streaming-pane.png)

Sin embargo, no verá aún ninguno de los mensajes de seguimiento. Esto se debe a que cuando selecciona primero **Ver registros de streaming**, la aplicación de Azure establece el nivel de seguimiento en `Error`, que solo registra eventos de error (con el método `Trace.TraceError()`).

### <a name="change-trace-levels"></a>Cambio de los niveles de seguimiento

Para cambiar los niveles de seguimiento para generar otros mensajes de seguimiento, vuelva al **Explorador de servidores**.

Haga clic de nuevo con el botón derecho en la aplicación de Azure y seleccione **Ver configuración**.

En la lista desplegable **Registro de la aplicación (sistema de archivos)**, seleccione **Detallado**. Haga clic en **Save**(Guardar).

![Cambiar el nivel de seguimiento a Detallado](./media/app-service-web-tutorial-dotnet-sqldatabase/trace-level-verbose.png)

> [!TIP]
> Puede experimentar con diferentes niveles de seguimiento para ver qué tipos de mensajes se muestran para cada nivel. Por ejemplo, el nivel **Información** incluye todos los registros creados por `Trace.TraceInformation()`, `Trace.TraceWarning()` y `Trace.TraceError()`, pero no los registros creados por `Trace.WriteLine()`.
>
>

En el explorador, vaya a la aplicación de nuevo en *http://&lt;nombre de la aplicación>.azurewebsites.net* e intente hacer clic en torno a la aplicación de la lista de tareas pendientes en Azure. Los mensajes de seguimiento se insertan ahora en la ventana **Salida** de Visual Studio.

```console
Application: 2017-04-06T23:30:41  PID[8132] Verbose     GET /Todos/Index
Application: 2017-04-06T23:30:43  PID[8132] Verbose     GET /Todos/Create
Application: 2017-04-06T23:30:53  PID[8132] Verbose     POST /Todos/Create
Application: 2017-04-06T23:30:54  PID[8132] Verbose     GET /Todos/Index
```



### <a name="stop-log-streaming"></a>Detención de las secuencias de registro

Para detener el servicio de secuencias de registros, haga clic en el botón **Detener supervisión** en la ventana **Salida**.

![Detención de las secuencias de registro](./media/app-service-web-tutorial-dotnet-sqldatabase/stop-streaming.png)

## <a name="manage-your-azure-app"></a>Administración de la aplicación de Azure

Vaya a [Azure Portal](https://portal.azure.com) para ver la aplicación que ha creado. 



En el menú izquierdo, haga clic en **App Service** y luego haga clic en el nombre de la aplicación de Azure.

![Navegación en el portal a la aplicación de Azure](./media/app-service-web-tutorial-dotnet-sqldatabase/access-portal.png)

Ha llegado a la página de la aplicación. 

De forma predeterminada, el portal muestra la página **Información general**. Esta página proporciona una visión del funcionamiento de la aplicación. En este caso, también puede realizar tareas de administración básicas como examinar, detener, iniciar, reiniciar y eliminar. Las pestañas del lado izquierdo de la página muestran las diferentes páginas de configuración que puede abrir. 

![Página de App Service en Azure Portal](./media/app-service-web-tutorial-dotnet-sqldatabase/web-app-blade.png)

[!INCLUDE [Clean up section](../../includes/clean-up-section-portal-web-app.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial aprendió lo siguiente:

> [!div class="checklist"]
> * Crear una base de datos Azure SQL Database
> * Conectar una aplicación ASP.NET a SQL Database
> * Implementación de la aplicación en Azure
> * Actualizar el modelo de datos y volver a implementar la aplicación
> * Transmitir registros desde Azure a un terminal
> * Administrar la aplicación en Azure Portal

Continúe con el siguiente tutorial para aprender a mejorar fácilmente la seguridad de la conexión con Azure SQL Database.

> [!div class="nextstepaction"]
> [Acceso seguro a SQL Database mediante identidades administradas para recursos de Azure](app-service-web-tutorial-connect-msi.md)
