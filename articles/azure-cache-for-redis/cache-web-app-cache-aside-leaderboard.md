---
title: 'Tutorial: Creación de una aplicación web (cache-aside): Azure Cache for Redis'
description: Aprenda a crear una aplicación web con Azure Cache for Redis que use el patrón cache-aside.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: tutorial
ms.custom: devx-track-csharp, mvc
ms.date: 03/30/2018
ms.openlocfilehash: 71f1e2b50daf333e19bc11bce119f37cec28d146
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88209202"
---
# <a name="tutorial-create-a-cache-aside-leaderboard-on-aspnet"></a>Tutorial: Creación de una tabla de clasificación cache-aside en ASP.NET

En este tutorial actualizará la aplicación web ASP.NET *ContosoTeamStats*, creada en el artículo de [inicio rápido de ASP.NET para Azure Cache for Redis](cache-web-app-howto.md), para incluir una tabla de clasificación que use el [patrón cache-aside](https://docs.microsoft.com/azure/architecture/patterns/cache-aside) con Azure Cache for Redis. La aplicación de ejemplo muestra una lista de estadísticas de equipos de una base de datos, además de distintas formas de usar Azure Cache for Redis para almacenar y recuperar datos de la caché a fin de mejorar el rendimiento. Al finalizar el tutorial, tiene una aplicación web en ejecución que lee y escribe en una base de datos, optimizada con Azure Cache for Redis y hospedada en Azure.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Mejorar el rendimiento de los datos y reducir la carga de bases de datos mediante el almacenamiento y la recuperación de datos con Azure Cache for Redis.
> * Usar un conjunto clasificado por Redis para recuperar los cinco mejores equipos.
> * Aprovisionar los recursos de Azure para la aplicación mediante una plantilla de Resource Manager.
> * Publicar la aplicación en Azure mediante Visual Studio.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

Para realizar este tutorial, debe disponer de los siguientes requisitos previos:

* Este tutorial continúa donde lo dejó en el artículo de [inicio rápido de ASP.NET para Azure Cache for Redis](cache-web-app-howto.md). Si aún lo ha hecho ya, complete esta guía de inicio rápido.
* Instale [Visual Studio 2019](https://www.visualstudio.com/downloads/) con las cargas de trabajo siguientes:
    * ASP.NET y desarrollo web
    * Desarrollo de Azure
    * Desarrollo del escritorio de .NET con SQL Server Express LocalDB o [SQL Server 2017 Express edition](https://www.microsoft.com/sql-server/sql-server-editions-express).

## <a name="add-a-leaderboard-to-the-project"></a>Incorporación de una tabla de clasificación al proyecto

En esta sección del tutorial, configurará el proyecto *ContosoTeamStats* con una tabla de clasificación que muestra las estadísticas de partidos ganados, perdidos y empatados de una lista de equipos ficticios.

### <a name="add-the-entity-framework-to-the-project"></a>Incorporación de Entity Framework al proyecto

1. En Visual Studio, abra la solución *ContosoTeamStats* que creó en el artículo de [inicio rápido de ASP.NET para Azure Cache for Redis](cache-web-app-howto.md).
2. Haga clic en **Herramientas > Administrador de paquetes NuGet > Consola del Administrador de paquetes**.
3. Ejecute el siguiente comando en la ventana **Consola del Administrador de paquetes** para instalar EntityFramework:

    ```powershell
    Install-Package EntityFramework
    ```

Para más información acerca de este paquete, consulte la página [EntityFramework](https://www.nuget.org/packages/EntityFramework/) de NuGet.

### <a name="add-the-team-model"></a>Incorporación del modelo del equipo

1. Haga clic con el botón derecho en **Modelos** en el **Explorador de soluciones** y elija **Agregar**, **Clase**.

1. Escriba `Team` para el nombre de clase y haga clic en **Agregar**.

    ![Agregar de clase de modelo](./media/cache-web-app-cache-aside-leaderboard/cache-model-add-class-dialog.png)

1. Reemplace las instrucciones `using` de la parte superior del archivo *Team.cs* por las siguientes instrucciones `using`:

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Data.Entity;
    using System.Data.Entity.SqlServer;
    ```

1. Reemplace la definición de la clase `Team` por el siguiente fragmento de código que contiene una definición de clase `Team` actualizada, así como algunas otras clases de asistente de Entity Framework. Este tutorial utiliza el enfoque de Code First con Entity Framework. Este enfoque permite a Entity Framework crear la base de datos desde el código. Para más información sobre el enfoque de Code First en Entity Framework, consulte [Code First para una nueva base de datos](/ef/ef6/modeling/code-first/workflows/new-database).

    ```csharp
    public class Team
    {
        public int ID { get; set; }
        public string Name { get; set; }
        public int Wins { get; set; }
        public int Losses { get; set; }
        public int Ties { get; set; }

        static public void PlayGames(IEnumerable<Team> teams)
        {
            // Simple random generation of statistics.
            Random r = new Random();

            foreach (var t in teams)
            {
                t.Wins = r.Next(33);
                t.Losses = r.Next(33);
                t.Ties = r.Next(0, 5);
            }
        }
    }

    public class TeamContext : DbContext
    {
        public TeamContext()
            : base("TeamContext")
        {
        }

        public DbSet<Team> Teams { get; set; }
    }

    public class TeamInitializer : CreateDatabaseIfNotExists<TeamContext>
    {
        protected override void Seed(TeamContext context)
        {
            var teams = new List<Team>
            {
                new Team{Name="Adventure Works Cycles"},
                new Team{Name="Alpine Ski House"},
                new Team{Name="Blue Yonder Airlines"},
                new Team{Name="Coho Vineyard"},
                new Team{Name="Contoso, Ltd."},
                new Team{Name="Fabrikam, Inc."},
                new Team{Name="Lucerne Publishing"},
                new Team{Name="Northwind Traders"},
                new Team{Name="Consolidated Messenger"},
                new Team{Name="Fourth Coffee"},
                new Team{Name="Graphic Design Institute"},
                new Team{Name="Nod Publishers"}
            };

            Team.PlayGames(teams);

            teams.ForEach(t => context.Teams.Add(t));
            context.SaveChanges();
        }
    }

    public class TeamConfiguration : DbConfiguration
    {
        public TeamConfiguration()
        {
            SetExecutionStrategy("System.Data.SqlClient", () => new SqlAzureExecutionStrategy());
        }
    }
    ```

1. En el **Explorador de soluciones**, haga doble clic en el archivo **Web.config** para abrirlo.

    ![Web.config](./media/cache-web-app-cache-aside-leaderboard/cache-web-config.png)

1. Agregue la siguiente sección `connectionStrings` en la sección `configuration`. El nombre de la cadena de conexión debe coincidir con el de la clase de contexto de la base de datos de Entity Framework, que es `TeamContext`.

    Esta cadena de conexión da por supuesto que se cumplen los [requisitos previos](#prerequisites) y que se ha instalado SQL Server Express LocalDB, que forma parte de la carga de trabajo del *desarrollo del escritorio de .NET* que se instala con Visual Studio 2019.

    ```xml
    <connectionStrings>
        <add name="TeamContext" connectionString="Data Source=(LocalDB)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\Teams.mdf;Integrated Security=True" providerName="System.Data.SqlClient" />
    </connectionStrings>
    ```

    En el ejemplo siguiente se muestra la nueva sección `connectionStrings` después de `configSections` dentro de la sección `configuration`:

    ```xml
    <configuration>
        <configSections>
        <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
        </configSections>
        <connectionStrings>
        <add name="TeamContext" connectionString="Data Source=(LocalDB)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\Teams.mdf;Integrated Security=True"     providerName="System.Data.SqlClient" />
        </connectionStrings>
        ...
    ```

### <a name="add-the-teamscontroller-and-views"></a>Incorporación de TeamsController y las vistas

1. En Visual Studio, compile el proyecto. 

1. En el **Explorador de soluciones**, haga clic con el botón derecho en la carpeta **Controladores** y elija **Agregar**, **Controlador**.

1. Seleccione **Controlador de MVC 5 con vistas que usa Entity Framework** y haga clic en **Agregar**. Si se produce un error después de hacer clic en **Agregar**, asegúrese de que ha compilado el proyecto primero.

    ![Agregar controlador](./media/cache-web-app-cache-aside-leaderboard/cache-add-controller-class.png)

1. Seleccione **Team (ContosoTeamStats.Models)** en la lista desplegable **Clase de modelo**. Seleccione **TeamContext (ContosoTeamStats.Models)** en la lista desplegable **Clase de contexto de datos**. Escriba `TeamsController` en el cuadro de texto de nombre **Controlador** (si no se rellena automáticamente). Haga clic en **Agregar** para crear la clase de controlador y agregue las vistas predeterminadas.

    ![Configurar controlador](./media/cache-web-app-cache-aside-leaderboard/cache-configure-controller.png)

1. En el **Explorador de soluciones**, expanda **Global.asax** y haga doble clic en **Global.asax.cs** para abrirlo.

    ![Global.asax.cs](./media/cache-web-app-cache-aside-leaderboard/cache-global-asax.png)

1. Agregue las dos instrucciones siguientes `using` a la parte superior del archivo, debajo de las restantes instrucciones `using`:

    ```csharp
    using System.Data.Entity;
    using ContosoTeamStats.Models;
    ```

1. Agregue la siguiente línea de código al final del método `Application_Start` :

    ```csharp
    Database.SetInitializer<TeamContext>(new TeamInitializer());
    ```

1. En el **Explorador de soluciones**, expanda `App_Start` y haga doble clic en `RouteConfig.cs`.

    ![RouteConfig.cs](./media/cache-web-app-cache-aside-leaderboard/cache-RouteConfig-cs.png)

1. En el método `RegisterRoutes`, reemplace `controller = "Home"` en la ruta `Default` por `controller = "Teams"` como se muestra en el ejemplo siguiente:

    ```csharp
    routes.MapRoute(
        name: "Default",
        url: "{controller}/{action}/{id}",
        defaults: new { controller = "Teams", action = "Index", id = UrlParameter.Optional }
    );
    ```

### <a name="configure-the-layout-view"></a>Configuración de la vista Diseño

1. En el **Explorador de soluciones**, expanda la carpeta **Vistas** y luego la carpeta **Compartido** y haga doble clic en **_Layout.cshtml**. 

    ![_Layout.cshtml](./media/cache-web-app-cache-aside-leaderboard/cache-layout-cshtml.png)

1. Cambie el contenido del elemento `title` y sustituya `My ASP.NET Application` por `Contoso Team Stats`, como se muestra en el ejemplo siguiente:

    ```html
    <title>@ViewBag.Title - Contoso Team Stats</title>
    ```

1. En la sección `body`, agregue la siguiente nueva instrucción `Html.ActionLink` para *Contoso Team Stats* inmediatamente debajo del vínculo de *Azure Cache for Redis Test* (Prueba de Azure Cache for Redis).

    ```csharp
    @Html.ActionLink("Contoso Team Stats", "Index", "Teams", new { area = "" }, new { @class = "navbar-brand" })`
    ```

    ![Cambios de código](./media/cache-web-app-cache-aside-leaderboard/cache-layout-cshtml-code.png)

1. Presione **Ctrl+F5** para compilar y ejecutar la aplicación. Esta versión de la aplicación lee los resultados directamente de la base de datos. Observe las acciones **Crear nuevo**, **Editar**, **Detalles** y **Eliminar** que se han agregado automáticamente a la aplicación mediante el scaffold **Controlador de MVC 5 con vistas que usa Entity Framework**. En la siguiente sección del tutorial, agregará Azure Cache for Redis para optimizar el acceso a los datos y proporcionar características adicionales a la aplicación.

    ![Aplicación de inicio](./media/cache-web-app-cache-aside-leaderboard/cache-starter-application.png)

## <a name="configure-the-app-for-azure-cache-for-redis"></a>Configuración de la aplicación para Azure Cache for Redis

En esta sección del tutorial, configura la aplicación de ejemplo para almacenar y recuperar estadísticas de equipos de Contoso de una instancia de Azure Cache for Redis mediante el cliente de caché de [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis).

### <a name="add-a-cache-connection-to-the-teams-controller"></a>Adición de una conexión de caché al controlador de los equipos

Ya instaló el paquete de biblioteca de cliente *StackExchange.Redis* en el inicio rápido. También ha configurado el valor de la aplicación *CacheConnection* para que se ejecute localmente y con la instancia de App Service publicada. Utilice esta misma biblioteca de cliente y la información de *CacheConnection* en *TeamsController*.

1. En el **Explorador de soluciones**, expanda la carpeta **Controladores** y haga doble clic en **TeamsController.cs** para abrirlo.

    ![Controlador de equipos](./media/cache-web-app-cache-aside-leaderboard/cache-teamscontroller.png)

1. Agregue las dos siguientes instrucciones `using` a **TeamsController.cs**:

    ```csharp
    using System.Configuration;
    using StackExchange.Redis;
    ```

1. Agregue las dos propiedades siguientes a la clase `TeamsController`:

    ```csharp
    // Redis Connection string info
    private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
    {
        string cacheConnection = ConfigurationManager.AppSettings["CacheConnection"].ToString();
        return ConnectionMultiplexer.Connect(cacheConnection);
    });

    public static ConnectionMultiplexer Connection
    {
        get
        {
            return lazyConnection.Value;
        }
    }
    ```

### <a name="update-the-teamscontroller-to-read-from-the-cache-or-the-database"></a>Actualización de la clase TeamsController para que lea de la caché o de la base de datos

En este ejemplo, se pueden recuperar estadísticas de equipos de la base de datos o de la caché. Las estadísticas de equipos se almacenan en la memoria caché como un elemento `List<Team>`serializado y también como un conjunto ordenado mediante tipos de datos de Redis. Al recuperar elementos de un conjunto ordenado, puede recuperar algunos, todos o consultar algunos de ellos. En este ejemplo, consultará el conjunto de los 5 mejores equipos ordenado por el número de victorias.

No es necesario almacenar las estadísticas de equipos en varios formatos en la caché para usar Azure Cache for Redis. En este tutorial se utilizan varios formatos para demostrar algunas de las diferentes maneras y diferentes tipos de datos que puede usar para almacenar datos en caché.

1. Agregue las siguientes instrucciones `using` al archivo `TeamsController.cs` encima de las restantes instrucciones `using`:

    ```csharp
    using System.Diagnostics;
    using Newtonsoft.Json;
    ```

1. Reemplace la implementación del método `public ActionResult Index()` actual por la siguiente implementación:

    ```csharp
    // GET: Teams
    public ActionResult Index(string actionType, string resultType)
    {
        List<Team> teams = null;

        switch(actionType)
        {
            case "playGames": // Play a new season of games.
                PlayGames();
                break;

            case "clearCache": // Clear the results from the cache.
                ClearCachedTeams();
                break;

            case "rebuildDB": // Rebuild the database with sample data.
                RebuildDB();
                break;
        }

        // Measure the time it takes to retrieve the results.
        Stopwatch sw = Stopwatch.StartNew();

        switch(resultType)
        {
            case "teamsSortedSet": // Retrieve teams from sorted set.
                teams = GetFromSortedSet();
                break;

            case "teamsSortedSetTop5": // Retrieve the top 5 teams from the sorted set.
                teams = GetFromSortedSetTop5();
                break;

            case "teamsList": // Retrieve teams from the cached List<Team>.
                teams = GetFromList();
                break;

            case "fromDB": // Retrieve results from the database.
            default:
                teams = GetFromDB();
                break;
        }

        sw.Stop();
        double ms = sw.ElapsedTicks / (Stopwatch.Frequency / (1000.0));

        // Add the elapsed time of the operation to the ViewBag.msg.
        ViewBag.msg += " MS: " + ms.ToString();

        return View(teams);
    }
    ```

1. Agregue los tres métodos siguientes a la clase `TeamsController` para implementar los tipos de acción `playGames`, `clearCache` y `rebuildDB` desde la instrucción switch agregada en el fragmento de código anterior.

    El método `PlayGames` actualiza las estadísticas de equipos mediante la simulación de una temporada de juegos, guarda los resultados en la base de datos y borra de la caché los datos ahora obsoletos.

    ```csharp
    void PlayGames()
    {
        ViewBag.msg += "Updating team statistics. ";
        // Play a "season" of games.
        var teams = from t in db.Teams
                    select t;

        Team.PlayGames(teams);

        db.SaveChanges();

        // Clear any cached results
        ClearCachedTeams();
    }
    ```

    El método `RebuildDB` reinicializa la base de datos con el conjunto predeterminado de equipos, genera estadísticas para ellos y borra de la memoria caché los datos ahora obsoletos.

    ```csharp
    void RebuildDB()
    {
        ViewBag.msg += "Rebuilding DB. ";
        // Delete and re-initialize the database with sample data.
        db.Database.Delete();
        db.Database.Initialize(true);

        // Clear any cached results
        ClearCachedTeams();
    }
    ```

    El método `ClearCachedTeams` quita de la memoria caché las estadísticas de equipos almacenadas.

    ```csharp
    void ClearCachedTeams()
    {
        IDatabase cache = Connection.GetDatabase();
        cache.KeyDelete("teamsList");
        cache.KeyDelete("teamsSortedSet");
        ViewBag.msg += "Team data removed from cache. ";
    }
    ```

1. Agregue los cuatro métodos siguientes a la clase `TeamsController` para implementar las distintas maneras de recuperar las estadísticas de equipos de la memoria caché y la base de datos. Cada uno de estos métodos devuelve un elemento `List<Team>`, que luego se muestra mediante la vista.

    El método `GetFromDB` lee las estadísticas de equipos de la base de datos.
    ```csharp
    List<Team> GetFromDB()
    {
        ViewBag.msg += "Results read from DB. ";
        var results = from t in db.Teams
            orderby t.Wins descending
            select t;

        return results.ToList<Team>();
    }
    ```

    El método `GetFromList` lee las estadísticas de equipos de la memoria caché como un elemento `List<Team>` serializado. Si las estadísticas no están presentes en la memoria caché, se produce un error de caché. En un error de caché, las estadísticas de los equipos se leen de la base de datos y luego se almacenan en la caché para la próxima solicitud. En este ejemplo, se usa JSON.NET para serializar los objetos .NET tanto a la caché como desde ella. Para más información, consulte [Trabajar con objetos .NET en la memoria caché](cache-dotnet-how-to-use-azure-redis-cache.md#work-with-net-objects-in-the-cache).

    ```csharp
    List<Team> GetFromList()
    {
        List<Team> teams = null;

        IDatabase cache = Connection.GetDatabase();
        string serializedTeams = cache.StringGet("teamsList");
        if (!String.IsNullOrEmpty(serializedTeams))
        {
            teams = JsonConvert.DeserializeObject<List<Team>>(serializedTeams);

            ViewBag.msg += "List read from cache. ";
        }
        else
        {
            ViewBag.msg += "Teams list cache miss. ";
            // Get from database and store in cache
            teams = GetFromDB();

            ViewBag.msg += "Storing results to cache. ";
            cache.StringSet("teamsList", JsonConvert.SerializeObject(teams));
        }
        return teams;
    }
    ```

    El método `GetFromSortedSet` lee las estadísticas de equipos de un conjunto ordenado almacenado en caché. Si se produce un error de caché, las estadísticas de equipos se leen de la base de datos y se almacenan en la caché como un conjunto ordenado.

    ```csharp
    List<Team> GetFromSortedSet()
    {
        List<Team> teams = null;
        IDatabase cache = Connection.GetDatabase();
        // If the key teamsSortedSet is not present, this method returns a 0 length collection.
        var teamsSortedSet = cache.SortedSetRangeByRankWithScores("teamsSortedSet", order: Order.Descending);
        if (teamsSortedSet.Count() > 0)
        {
            ViewBag.msg += "Reading sorted set from cache. ";
            teams = new List<Team>();
            foreach (var t in teamsSortedSet)
            {
                Team tt = JsonConvert.DeserializeObject<Team>(t.Element);
                teams.Add(tt);
            }
        }
        else
        {
            ViewBag.msg += "Teams sorted set cache miss. ";

            // Read from DB
            teams = GetFromDB();

            ViewBag.msg += "Storing results to cache. ";
            foreach (var t in teams)
            {
                Console.WriteLine("Adding to sorted set: {0} - {1}", t.Name, t.Wins);
                cache.SortedSetAdd("teamsSortedSet", JsonConvert.SerializeObject(t), t.Wins);
            }
        }
        return teams;
    }
    ```

    El método `GetFromSortedSetTop5` lee los cinco mejores equipos del conjunto ordenado almacenado en la caché. Para comenzar, comprueba que en la memoria caché exista la clave `teamsSortedSet` . Si esta clave no existe, se llama al método `GetFromSortedSet` para leer las estadísticas de equipos y almacenarlas en la memoria caché. Luego, se consulta en el conjunto almacenado en la caché los cinco mejores equipo, y se devuelven.

    ```csharp
    List<Team> GetFromSortedSetTop5()
    {
        List<Team> teams = null;
        IDatabase cache = Connection.GetDatabase();

        // If the key teamsSortedSet is not present, this method returns a 0 length collection.
        var teamsSortedSet = cache.SortedSetRangeByRankWithScores("teamsSortedSet", stop: 4, order: Order.Descending);
        if(teamsSortedSet.Count() == 0)
        {
            // Load the entire sorted set into the cache.
            GetFromSortedSet();

            // Retrieve the top 5 teams.
            teamsSortedSet = cache.SortedSetRangeByRankWithScores("teamsSortedSet", stop: 4, order: Order.Descending);
        }

        ViewBag.msg += "Retrieving top 5 teams from cache. ";
        // Get the top 5 teams from the sorted set
        teams = new List<Team>();
        foreach (var team in teamsSortedSet)
        {
            teams.Add(JsonConvert.DeserializeObject<Team>(team.Element));
        }
        return teams;
    }
    ```

### <a name="update-the-create-edit-and-delete-methods-to-work-with-the-cache"></a>Actualizar los métodos Create, Edit y Delete para trabajar con la caché

El código de scaffolding que se generó como parte de este ejemplo incluye métodos para agregar, editar y eliminar equipos. Cada vez que se agrega, edita o elimina un equipo, los datos de la caché se vuelven obsoletos. En esta sección modificará estos tres métodos para borrar los equipos almacenados en la caché, con el fin de actualizar la caché.

1. Vaya al método `Create(Team team)` en la clase `TeamsController`. Agregue una llamada al método `ClearCachedTeams`, tal como se muestra en el ejemplo siguiente:

    ```csharp
    // POST: Teams/Create
    // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
    // more details see https://go.microsoft.com/fwlink/?LinkId=317598.
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult Create([Bind(Include = "ID,Name,Wins,Losses,Ties")] Team team)
    {
        if (ModelState.IsValid)
        {
            db.Teams.Add(team);
            db.SaveChanges();
            // When a team is added, the cache is out of date.
            // Clear the cached teams.
            ClearCachedTeams();
            return RedirectToAction("Index");
        }

        return View(team);
    }
    ```

2. Vaya al método `Edit(Team team)` en la clase `TeamsController`. Agregue una llamada al método `ClearCachedTeams`, tal como se muestra en el ejemplo siguiente:

    ```csharp
    // POST: Teams/Edit/5
    // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
    // more details see https://go.microsoft.com/fwlink/?LinkId=317598.
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult Edit([Bind(Include = "ID,Name,Wins,Losses,Ties")] Team team)
    {
        if (ModelState.IsValid)
        {
            db.Entry(team).State = EntityState.Modified;
            db.SaveChanges();
            // When a team is edited, the cache is out of date.
            // Clear the cached teams.
            ClearCachedTeams();
            return RedirectToAction("Index");
        }
        return View(team);
    }
    ```

3. Vaya al método `DeleteConfirmed(int id)` en la clase `TeamsController`. Agregue una llamada al método `ClearCachedTeams`, tal como se muestra en el ejemplo siguiente:

    ```csharp
    // POST: Teams/Delete/5
    [HttpPost, ActionName("Delete")]
    [ValidateAntiForgeryToken]
    public ActionResult DeleteConfirmed(int id)
    {
        Team team = db.Teams.Find(id);
        db.Teams.Remove(team);
        db.SaveChanges();
        // When a team is deleted, the cache is out of date.
        // Clear the cached teams.
        ClearCachedTeams();
        return RedirectToAction("Index");
    }
    ```

### <a name="add-caching-methods-to-the-teams-index-view"></a>Incorporación de los métodos de almacenamiento en caché a la vista del índice de equipos

1. En el **Explorador de soluciones**, expanda la carpeta **Vistas** y luego la carpeta **Teams** y haga doble clic en **Index.cshtml**.

    ![Index.cshtml](./media/cache-web-app-cache-aside-leaderboard/cache-views-teams-index-cshtml.png)

1. Cerca de la parte superior del archivo, busque el siguiente elemento de párrafo:

    ![Tabla de acciones](./media/cache-web-app-cache-aside-leaderboard/cache-teams-index-table.png)

    Este vínculo crea un equipo nuevo. Reemplace el elemento de párrafo por la siguiente tabla. Esta tabla incluye vínculos de acción para crear un nuevo equipo, reproducir una nueva temporada, borrar la caché, recuperar los equipos de la caché en varios formatos, recuperar los equipos de la base de datos y volver a compilar la base de datos con datos de ejemplo nuevos.

    ```html
    <table class="table">
        <tr>
            <td>
                @Html.ActionLink("Create New", "Create")
            </td>
            <td>
                @Html.ActionLink("Play Season", "Index", new { actionType = "playGames" })
            </td>
            <td>
                @Html.ActionLink("Clear Cache", "Index", new { actionType = "clearCache" })
            </td>
            <td>
                @Html.ActionLink("List from Cache", "Index", new { resultType = "teamsList" })
            </td>
            <td>
                @Html.ActionLink("Sorted Set from Cache", "Index", new { resultType = "teamsSortedSet" })
            </td>
            <td>
                @Html.ActionLink("Top 5 Teams from Cache", "Index", new { resultType = "teamsSortedSetTop5" })
            </td>
            <td>
                @Html.ActionLink("Load from DB", "Index", new { resultType = "fromDB" })
            </td>
            <td>
                @Html.ActionLink("Rebuild DB", "Index", new { actionType = "rebuildDB" })
            </td>
        </tr>
    </table>
    ```

1. Desplácese hasta el final del archivo **Index.cshtml** y agregue el siguiente elemento `tr` de modo que esta sea la última fila de la última tabla del archivo:

    ```html
    <tr><td colspan="5">@ViewBag.Msg</td></tr>
    ```
    Esta fila muestra el valor de `ViewBag.Msg`, que contiene un informe con el estado de la operación actual. El valor de `ViewBag.Msg` se establece al hacer clic en cualquiera de los vínculos de acción del paso anterior.

    ![Mensaje de estado](./media/cache-web-app-cache-aside-leaderboard/cache-status-message.png)

1. Presione **F6** para compilar el proyecto.

## <a name="run-the-app-locally"></a>Ejecución de la aplicación de forma local

Ejecute la aplicación localmente en el equipo para comprobar la funcionalidad que se ha agregado a para dar soporte a los equipos.

En esta prueba, tanto la aplicación como la base de datos se ejecutan localmente. Pero la instancia de Azure Cache for Redis se hospeda de forma remota en Azure. Por consiguiente, es probable que el funcionamiento de la memoria caché sea ligeramente inferior al de la base de datos. Para obtener el mejor rendimiento, la aplicación cliente y la instancia de Azure Cache for Redis deben estar en la misma ubicación. En la sección siguiente, implementará todos los recursos en Azure para ver el rendimiento mejorado que supone usar una memoria caché.

Para ejecutar la aplicación localmente:

1. Presione **Ctrl+F5** para ejecutar la aplicación.

    ![Aplicación que se ejecuta localmente](./media/cache-web-app-cache-aside-leaderboard/cache-local-application.png)

1. Pruebe todos los métodos nuevos que se han agregado a la vista. Dado que la memoria caché es remota en estas pruebas, la base de datos debería superar ligeramente la memoria caché.

## <a name="publish-and-run-in-azure"></a>Publicación y ejecución en Azure

### <a name="provision-a-database-for-the-app"></a>Aprovisionamiento de una base de datos para la aplicación

En esta sección aprovisionará una nueva base de datos en SQL Database para que la use la aplicación mientras esté hospedada en Azure.

1. En la esquina superior izquierda de [Azure Portal](https://portal.azure.com/), haga clic en **Crear un recurso**.

1. En la página **Nuevos**, haga clic en **Databases** > **SQL Database**.

1. Utilice la siguiente configuración para la nueva instancia de SQL Database:

   | Configuración       | Valor sugerido | Descripción |
   | ------------ | ------------------ | ------------------------------------------------- |
   | **Nombre de la base de datos** | *ContosoTeamsDatabase* | Para conocer los nombres de base de datos válidos, consulte [Database Identifiers](https://docs.microsoft.com/sql/relational-databases/databases/database-identifiers) (Identificadores de base de datos). |
   | **Suscripción** | *Su suscripción*  | Seleccione la misma suscripción que usó para crear la memoria caché y hospedar la instancia de App Service. |
   | **Grupos de recursos**  | *TestResourceGroup* | Haga clic en **Usar existente** y use el grupo de recursos en el que colocó la memoria caché y App Service. |
   | **Seleccionar origen** | **Base de datos en blanco** | Comience con una base de datos en blanco. |

1. En **Servidor**, haga clic en **Configurar los valores obligatorios** > **Crear un nuevo servidor** y especifique la siguiente información y, después, haga clic en el botón **Seleccionar**:

   | Configuración       | Valor sugerido | Descripción |
   | ------------ | ------------------ | ------------------------------------------------- |
   | **Nombre del servidor** | Cualquier nombre globalmente único | Para conocer cuáles son los nombres de servidor válidos, consulte el artículo [Naming conventions](/azure/architecture/best-practices/resource-naming) (Convenciones de nomenclatura). |
   | **Inicio de sesión del administrador del servidor** | Cualquier nombre válido | Para conocer los nombres de inicio de sesión válidos, consulte [Database Identifiers](https://docs.microsoft.com/sql/relational-databases/databases/database-identifiers) (Identificadores de base de datos). |
   | **Contraseña** | Cualquier contraseña válida | La contraseña debe tener un mínimo de 8 caracteres y debe contener caracteres de tres de las siguientes categorías: caracteres en mayúsculas, caracteres en minúsculas, números y caracteres no alfanuméricos. |
   | **Ubicación** | *Este de EE. UU.* | Seleccione la región en la que creó la caché y la instancia de App Service. |

1. Haga clic en **Anclar al panel** y, después, en **Crear** para crear la nueva base de datos y el servidor.

1. Una vez que se crea la base de datos nueva, haga clic en **Mostrar las cadenas de conexión de la base de datos** y copie la cadena de conexión de **ADO.NET**.

    ![Mostrar cadenas de conexión](./media/cache-web-app-cache-aside-leaderboard/cache-show-connection-strings.png)

1. En Azure Portal, vaya a su instancia de App Service y haga clic en **Configuración de la aplicación**, **Agregar nueva cadena de conexión** en la sección Cadenas de conexión.

1. Agregue una cadena de conexión denominada *TeamContext* para que coincida con la clase de contexto de la base de datos de Entity Framework. Pegue la cadena de conexión de la base de datos nueva como valor. Asegúrese de reemplazar los siguientes marcadores de posición en la cadena de conexión y haga clic en **Guardar**:

    | Marcador de posición | Valor sugerido |
    | --- | --- |
    | *{su_nombredeusuario}* | Use el **inicio de sesión de administrador de servidor** del servidor que acaba de crear. |
    | *{su_contraseña}* | Use la contraseña del servidor que acaba de crear. |

    Al agregar el nombre de usuario y la contraseña como una configuración de la aplicación, su nombre de usuario y su contraseña no se incluyen en el código. Este enfoque le ayuda a proteger dichas credenciales.

### <a name="publish-the-application-updates-to-azure"></a>Publicación de las actualizaciones de la aplicación en Azure

En este paso del tutorial, publicará las actualizaciones de la aplicación en Azure para ejecutarla en la nube.

1. Haga clic con el botón derecho en el proyecto **ContosoTeamStats** de Visual Studio y elija **Publicar**.

    ![Publicar](./media/cache-web-app-cache-aside-leaderboard/cache-publish-app.png)

2. Haga clic en **Publicar** para usar el mismo perfil de publicación que creó en la guía de inicio rápido.

3. Una vez completada la publicación, Visual Studio inicia la aplicación en el explorador web predeterminado.

    ![Caché agregada](./media/cache-web-app-cache-aside-leaderboard/cache-added-to-application.png)

    En la tabla siguiente se describe cada vínculo de acción de la aplicación de ejemplo:

    | Acción | Descripción |
    | --- | --- |
    | Crear nuevo |Crear un nuevo equipo. |
    | Reproducir temporada |Reproducir una temporada de juegos, actualizar las estadísticas de equipos y borrar de la caché los datos de equipo no actualizados. |
    | Borrar caché |Borrar las estadísticas de equipos de la caché. |
    | Mostrar de la caché |Recuperar las estadísticas de equipos de la caché. Si se produce un error de caché, cargue las estadísticas de la base de datos y guárdelas en la caché para la próxima vez. |
    | Conjunto ordenado de caché |Recuperar las estadísticas de equipos de la caché con un conjunto ordenado. Si se produce un error de caché, cargue las estadísticas de la base de datos y guárdelas en la caché mediante un conjunto ordenado. |
    | 5 equipos principales de la caché |Recuperar los 5 equipos principales de la caché mediante un conjunto ordenado. Si se produce un error de caché, cargue las estadísticas de la base de datos y guárdelas en la caché mediante un conjunto ordenado. |
    | Cargar de la base de datos |Recuperar las estadísticas de equipos de la base de datos. |
    | Recompilar base de datos |Volver a compilar la base de datos y cargarla con los datos de equipo de ejemplo. |
    | Editar, Detalles, Eliminar |Editar un equipo, ver los detalles de un equipo, eliminar un equipo. |

Haga clic en algunas de las acciones y experimente con la recuperación de los datos desde distintos orígenes. Observe las diferencias en el tiempo que tardan en completarse las diferentes formas de recuperar los datos desde la base de datos y desde la caché.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con la aplicación del tutorial de ejemplo, puede eliminar los recursos de Azure utilizados para ahorrar costos y recursos. Todos los recursos deben estar en el mismo grupo de recursos, por lo que se pueden eliminar conjuntamente en una sola operación mediante la eliminación del grupo de recursos. En las instrucciones de este tema se usa un grupo de recursos llamado *TestResources*.

> [!IMPORTANT]
> La eliminación de un grupo de recursos es irreversible y el grupo de recursos y todos los recursos que contiene se eliminarán de forma permanente. Asegúrese de no eliminar por accidente el grupo de recursos o los recursos equivocados. Si ha creado los recursos para hospedar este ejemplo en un grupo de recursos existente que contenga recursos que desee mantener, puede eliminar cada uno de los recursos individualmente en sus hojas respectivas.
>

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y haga clic en **Grupos de recursos**.
2. Escriba el nombre de su grupo de recursos en el cuadro de texto **Filtrar elementos** .
3. Haga clic en **...**  a la derecha del grupo de recursos y haga clic en **Eliminar grupo de recursos**.

    ![Eliminar](./media/cache-web-app-cache-aside-leaderboard/cache-delete-resource-group.png)

4. Se le pedirá que confirme la eliminación del grupo de recursos. Escriba el nombre del grupo de recursos para confirmar y haga clic en **Eliminar**.

    Transcurridos unos instantes, el grupo de recursos y todos los recursos que contiene se eliminan.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Escalado de Azure Cache for Redis](./cache-how-to-scale.md)
