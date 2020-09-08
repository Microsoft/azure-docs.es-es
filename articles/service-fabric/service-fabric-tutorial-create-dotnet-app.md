---
title: Creación de una aplicación .NET de Service Fabric en Azure
description: En este tutorial, aprenderá a crear una aplicación con un front-end de ASP.NET Core y un back-end de servicio confiable con estado, así como a implementar la aplicación en un clúster.
ms.topic: tutorial
ms.date: 07/10/2019
ms.custom: mvc, devx-track-javascript, devx-track-csharp
ms.openlocfilehash: 847bd2e30977c5a14f8294f6709cbd6d376a8211
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89016281"
---
# <a name="tutorial-create-and-deploy-an-application-with-an-aspnet-core-web-api-front-end-service-and-a-stateful-back-end-service"></a>Tutorial: Creación e implementación de una aplicación con un servicio de front-end de ASP.NET Core Web API y un servicio back-end con estado

Este tutorial es la primera parte de una serie.  Aprenda a crear una aplicación de Azure Service Fabric con un front-end de ASP.NET Core Web API y un servicio back-end con estado para almacenar los datos. Cuando termine, tendrá una aplicación de votación con un front-end web de ASP.NET Core que guarda los resultados de una votación en un servicio back-end con estado en el clúster. Esta serie de tutoriales necesita una máquina de desarrollador de Windows. Si no desea crear manualmente la aplicación de votación, puede [descargar el código fuente](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart/) de la aplicación terminada y pasar directamente al [Tutorial de la aplicación de ejemplo de votación](#walkthrough_anchor).  Si lo prefiere, también puede ver un [vídeo](https://channel9.msdn.com/Events/Connect/2017/E100) de este tutorial.

![Front-end de la API AngularJS+ASP.NET, conexión a un servicio de back-end con estado en Service Fabric](./media/service-fabric-tutorial-create-dotnet-app/application-diagram.png)

En la primera parte de la serie, se aprende a:

> [!div class="checklist"]
> * Crear un servicio de ASP.NET Core Web API como un servicio de confianza con estado
> * Crear un servicio de ASP.NET Core Web Application como un servicio de confianza sin estado
> * Usar el proxy inverso para comunicarse con el servicio con estado

En esta serie de tutoriales, se aprende a:
> [!div class="checklist"]
> * Crear una aplicación de .NET Service Fabric
> * [Implementar la aplicación en un clúster remoto](service-fabric-tutorial-deploy-app-to-party-cluster.md)
> * [Agregar un punto de conexión HTTPS a un servicio de front-end de ASP.NET Core](service-fabric-tutorial-dotnet-app-enable-https-endpoint.md)
> * [Configuración de CI/CD con Azure Pipelines](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)
> * [Configurar la supervisión y el diagnóstico para la aplicación](service-fabric-tutorial-monitoring-aspnet.md)

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar este tutorial:
* Si no tiene ninguna suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* [Instale Visual Studio 2019](https://www.visualstudio.com/) versión 15.5 o posteriores con las cargas de trabajo de **desarrollo de Azure** y de **desarrollo de ASP.NET y web**.
* [Instale el SDK de Service Fabric](service-fabric-get-started.md)

## <a name="create-an-aspnet-web-api-service-as-a-reliable-service"></a>Creación de un servicio de ASP.NET Web API como un servicio de confianza

En primer lugar, cree el front-end web de la aplicación de votación mediante ASP.NET Core. ASP.NET Core es un marco de desarrollo web ligero multiplataforma, que puede usar para crear modernas interfaces de usuario web y API web. Para obtener una descripción completa de cómo se integra ASP.NET Core con Service Fabric, se recomienda fehacientemente leer el artículo [ASP.NET Core en Reliable Services de Service Fabric](service-fabric-reliable-services-communication-aspnetcore.md). De momento, puede seguir este tutorial para empezar a trabajar rápidamente. Para más información sobre ASP.NET Core, vea [Documentación de ASP.NET Core](/aspnet/core/).

1. Inicie Visual Studio como **administrador**.

2. Cree un proyecto con **Archivo**->**Nuevo**->**Proyecto**.

3. En el cuadro de diálogo **Nuevo proyecto**, elija **Nube > Aplicación de Service Fabric**.

4. Asigne el nombre **Voting** a la aplicación y haga clic en **Aceptar**.

   ![Cuadro de diálogo de proyecto nuevo en Visual Studio](./media/service-fabric-tutorial-create-dotnet-app/new-project-dialog.png)

5. En la página **Nuevo servicio de Service Fabric**, elija **ASP.NET Core sin estado**, asigne el nombre de **VotingWeb** al servicio y haga clic en **Aceptar**.
   
   ![Selección del servicio web ASP.NET en el cuadro de diálogo de nuevo servicio](./media/service-fabric-tutorial-create-dotnet-app/new-project-dialog-2.png) 

6. En la página siguiente se proporciona un conjunto de plantillas de proyecto ASP.NET Core. Para este tutorial, elija **Aplicación web (controlador de vista de modelos)** y haga clic en **Aceptar**.
   
   ![Elección del tipo de proyecto ASP.NET](./media/service-fabric-tutorial-create-dotnet-app/vs-new-aspnet-project-dialog.png)

   Visual Studio crea una aplicación y un proyecto de servicio, y los muestra en el Explorador de soluciones.

   ![Explorador de soluciones después de la creación de una aplicación con el servicio ASP.NET Core Web API]( ./media/service-fabric-tutorial-create-dotnet-app/solution-explorer-aspnetcore-service.png)

### <a name="update-the-sitejs-file"></a>Actualización del archivo site.js
Abra **wwwroot/js/site.js**.  Reemplace el contenido por el código de JavaScript siguiente que usan las vistas de inicio y guarde los cambios.

```javascript
var app = angular.module('VotingApp', ['ui.bootstrap']);
app.run(function () { });

app.controller('VotingAppController', ['$rootScope', '$scope', '$http', '$timeout', function ($rootScope, $scope, $http, $timeout) {

    $scope.refresh = function () {
        $http.get('api/Votes?c=' + new Date().getTime())
            .then(function (data, status) {
                $scope.votes = data;
            }, function (data, status) {
                $scope.votes = undefined;
            });
    };

    $scope.remove = function (item) {
        $http.delete('api/Votes/' + item)
            .then(function (data, status) {
                $scope.refresh();
            })
    };

    $scope.add = function (item) {
        var fd = new FormData();
        fd.append('item', item);
        $http.put('api/Votes/' + item, fd, {
            transformRequest: angular.identity,
            headers: { 'Content-Type': undefined }
        })
            .then(function (data, status) {
                $scope.refresh();
                $scope.item = undefined;
            })
    };
}]);
```

### <a name="update-the-indexcshtml-file"></a>Actualización del archivo Index.cshtml

Abra el archivo **Views/Home/Index.cshtml**, la vista específica para el controlador de inicio.  Reemplace su contenido por el siguiente y guarde los cambios.

```html
@{
    ViewData["Title"] = "Service Fabric Voting Sample";
}

<div ng-controller="VotingAppController" ng-init="refresh()">
    <div class="container-fluid">
        <div class="row">
            <div class="col-xs-8 col-xs-offset-2 text-center">
                <h2>Service Fabric Voting Sample</h2>
            </div>
        </div>

        <div class="row">
            <div class="col-xs-8 col-xs-offset-2">
                <form class="col-xs-12 center-block">
                    <div class="col-xs-6 form-group">
                        <input id="txtAdd" type="text" class="form-control" placeholder="Add voting option" ng-model="item"/>
                    </div>
                    <button id="btnAdd" class="btn btn-default" ng-click="add(item)">
                        <span class="glyphicon glyphicon-plus" aria-hidden="true"></span>
                        Add
                    </button>
                </form>
            </div>
        </div>

        <hr/>

        <div class="row">
            <div class="col-xs-8 col-xs-offset-2">
                <div class="row">
                    <div class="col-xs-4">
                        Click to vote
                    </div>
                </div>
                <div class="row top-buffer" ng-repeat="vote in votes.data">
                    <div class="col-xs-8">
                        <button class="btn btn-success text-left btn-block" ng-click="add(vote.key)">
                            <span class="pull-left">
                                {{vote.key}}
                            </span>
                            <span class="badge pull-right">
                                {{vote.value}} Votes
                            </span>
                        </button>
                    </div>
                    <div class="col-xs-4">
                        <button class="btn btn-danger pull-right btn-block" ng-click="remove(vote.key)">
                            <span class="glyphicon glyphicon-remove" aria-hidden="true"></span>
                            Remove
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

### <a name="update-the-_layoutcshtml-file"></a>Actualización del archivo _Layout.cshtml

Abra el archivo **Views/Shared/_Layout.cshtml**, el diseño predeterminado para la aplicación ASP.NET.  Reemplace su contenido por el siguiente y guarde los cambios.


```html
<!DOCTYPE html>
<html ng-app="VotingApp" xmlns:ng="https://angularjs.org">
<head>
    <meta charset="utf-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>@ViewData["Title"]</title>

    <link href="~/lib/bootstrap/dist/css/bootstrap.css" rel="stylesheet"/>
    <link href="~/css/site.css" rel="stylesheet"/>

</head>
<body>
<div class="container body-content">
    @RenderBody()
</div>

<script src="~/lib/jquery/dist/jquery.js"></script>
<script src="~/lib/bootstrap/dist/js/bootstrap.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.7.2/angular.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular-ui-bootstrap/2.5.0/ui-bootstrap-tpls.js"></script>
<script src="~/js/site.js"></script>

@RenderSection("Scripts", required: false)
</body>
</html>
```

### <a name="update-the-votingwebcs-file"></a>Actualización del archivo VotingWeb.cs

Abra el archivo *VotingWeb.cs*, que permite crear el elemento WebHost de ASP.NET Core dentro del servicio sin estado con el servidor web de WebListener.

Agregue la directiva `using System.Net.Http;` en la parte superior del archivo.

Reemplace la función `CreateServiceInstanceListeners()` por el código siguiente y guarde los cambios.

```csharp
protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
{
    return new ServiceInstanceListener[]
    {
        new ServiceInstanceListener(
            serviceContext =>
                new KestrelCommunicationListener(
                    serviceContext,
                    "ServiceEndpoint",
                    (url, listener) =>
                    {
                        ServiceEventSource.Current.ServiceMessage(serviceContext, $"Starting Kestrel on {url}");

                        return new WebHostBuilder()
                            .UseKestrel()
                            .ConfigureServices(
                                services => services
                                    .AddSingleton<HttpClient>(new HttpClient())
                                    .AddSingleton<FabricClient>(new FabricClient())
                                    .AddSingleton<StatelessServiceContext>(serviceContext))
                            .UseContentRoot(Directory.GetCurrentDirectory())
                            .UseStartup<Startup>()
                            .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
                            .UseUrls(url)
                            .Build();
                    }))
    };
}
```

Agregue también el método `GetVotingDataServiceName` siguiente debajo de `CreateServiceInstanceListeners()` y guarde los cambios. `GetVotingDataServiceName` devuelve el nombre del servicio cuando se sondea.

```csharp
internal static Uri GetVotingDataServiceName(ServiceContext context)
{
    return new Uri($"{context.CodePackageActivationContext.ApplicationName}/VotingData");
}
```

### <a name="add-the-votescontrollercs-file"></a>Incorporación del archivo VotesController.cs

Agregue un controlador para definir las acciones de votación. Haga clic con el botón derecho en la carpeta **Controladores** y seleccione **Agregar->Nuevo elemento->Visual C#->ASP.NET Core->Clase**. Asigne al archivo el nombre **VotesController.cs** y haga clic en **Agregar**.  

Reemplace el contenido del archivo *VotesController.cs* por el siguiente y guarde los cambios.  Más adelante, en [Actualización del archivo VotesController.cs](#updatevotecontroller_anchor), este archivo se modifica para leer y escribir los datos de votación desde el servicio back-end.  Por ahora, el controlador devuelve datos de cadena estática a la vista.

```csharp
namespace VotingWeb.Controllers
{
    using System;
    using System.Collections.Generic;
    using System.Fabric;
    using System.Fabric.Query;
    using System.Linq;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Text;
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Mvc;
    using Newtonsoft.Json;

    [Produces("application/json")]
    [Route("api/Votes")]
    public class VotesController : Controller
    {
        private readonly HttpClient httpClient;

        public VotesController(HttpClient httpClient)
        {
            this.httpClient = httpClient;
        }

        // GET: api/Votes
        [HttpGet]
        public async Task<IActionResult> Get()
        {
            List<KeyValuePair<string, int>> votes= new List<KeyValuePair<string, int>>();
            votes.Add(new KeyValuePair<string, int>("Pizza", 3));
            votes.Add(new KeyValuePair<string, int>("Ice cream", 4));

            return Json(votes);
        }
     }
}
```

### <a name="configure-the-listening-port"></a>Configuración del puerto de escucha

Cuando se crea el servicio front-end VotingWeb, Visual Studio selecciona aleatoriamente un puerto en el que el servicio realiza la escucha.  Dado que el servicio VotingWeb actúa como front-end para esta aplicación y acepta tráfico externo, queremos enlazar dicho servicio a un puerto fijo y conocido.  El [manifiesto del servicio](service-fabric-application-and-service-manifests.md) declara los puntos de conexión de servicio.

En el Explorador de soluciones, abra *VotingWeb/PackageRoot/ServiceManifest.xml*.  Busque el elemento **Endpoint** (Punto de conexión) en la sección **Resources** (Recursos) y cambie el valor del **puerto** a **8080**. Para implementar y ejecutar la aplicación localmente, el puerto de escucha de la aplicación debe estar abierto y disponible en el equipo.

```xml
<Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Protocol="http" Name="ServiceEndpoint" Type="Input" Port="8080" />
    </Endpoints>
  </Resources>
```

Actualice también el valor de la propiedad de dirección URL de la aplicación en el proyecto Voting para que un explorador web se abra en el puerto correcto al depurar la aplicación.  En el Explorador de soluciones, seleccione el proyecto **Voting** y actualice la propiedad **dirección URL de la aplicación** a **8080**.

### <a name="deploy-and-run-the-voting-application-locally"></a>Implementación y ejecución local de la aplicación Voting
Ya puede continuar y ejecutar la aplicación Voting para depuración. En Visual Studio, presione **F5** para implementar la aplicación en el clúster local de Service Fabric en modo de depuración. La aplicación presentará un error si no abrió Visual Studio anteriormente como **administrador**.

> [!NOTE]
> La primera vez que ejecute e implemente la aplicación localmente, Visual Studio creará un clúster local de Service Fabric para la depuración.  Es posible que la creación del clúster tarde un tiempo. El estado de creación del clúster se muestra en la ventana de salida de Visual Studio.

Una vez que la aplicación Voting se implemente en el clúster local de Service Fabric, la aplicación web se abrirá automáticamente en una pestaña del explorador y tendrá un aspecto similar al siguiente:

![Front-end de ASP.NET Core](./media/service-fabric-tutorial-create-dotnet-app/debug-front-end.png)

Para detener la depuración de la aplicación, vuelva a Visual Studio y presione **Mayús+F5**.

## <a name="add-a-stateful-back-end-service-to-your-application"></a>Adición de un servicio back-end con estado a la aplicación

Ahora que se ejecuta un servicio de ASP.NET Web API en la aplicación, agreguemos un servicio de confianza con estado para almacenar datos en la aplicación.

Service Fabric permite almacenar de forma coherente y confiable su estado justo dentro de su servicio mediante Reliable Collections. Las colecciones de confianza son un conjunto de clases de colecciones de alta disponibilidad y confiabilidad que le resultará familiar a cualquiera que haya usado colecciones de C#.

En este tutorial creará un servicio, que almacena un valor de contador en una colección de confianza.

1. En el Explorador de soluciones, haga clic con el botón derecho en **Servicios**, en el proyecto de aplicación Voting y seleccione **Agregar > Nuevo servicio de Service Fabric…** .
    
2. En el cuadro de diálogo **Nuevo servicio de Service Fabric**, elija **ASP.NET Core con estado**, asigne el nombre de **VotingData** al servicio y presione **Aceptar**.

    Una vez creado el proyecto de servicio, tendrá dos servicios en la aplicación. Mientras continúa la creación de la aplicación, puede agregar más servicios de la misma forma. Cada uno puede tener versiones y actualizaciones independientes.

3. En la página siguiente se proporciona un conjunto de plantillas de proyecto ASP.NET Core. Para este tutorial, elija **API**.

    Visual Studio crea el proyecto de servicio VotingData y lo muestra en el Explorador de soluciones.

    ![Explorador de soluciones](./media/service-fabric-tutorial-create-dotnet-app/solution-explorer-aspnetcore-webapi-service.png)

### <a name="add-the-votedatacontrollercs-file"></a>Incorporación del archivo VoteDataController.cs

En el proyecto **VotingData** haga clic con el botón derecho en la carpeta **Controladores** y seleccione **Agregar->Nuevo elemento->Clase**. Asigne al archivo el nombre **VoteDataController.cs** y haga clic en **Agregar**. Reemplace el contenido del archivo por el siguiente y guarde los cambios.

```csharp
namespace VotingData.Controllers
{
    using System.Collections.Generic;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.ServiceFabric.Data;
    using Microsoft.ServiceFabric.Data.Collections;

    [Route("api/[controller]")]
    public class VoteDataController : Controller
    {
        private readonly IReliableStateManager stateManager;

        public VoteDataController(IReliableStateManager stateManager)
        {
            this.stateManager = stateManager;
        }

        // GET api/VoteData
        [HttpGet]
        public async Task<IActionResult> Get()
        {
            CancellationToken ct = new CancellationToken();

            IReliableDictionary<string, int> votesDictionary = await this.stateManager.GetOrAddAsync<IReliableDictionary<string, int>>("counts");

            using (ITransaction tx = this.stateManager.CreateTransaction())
            {
                Microsoft.ServiceFabric.Data.IAsyncEnumerable<KeyValuePair<string, int>> list = await votesDictionary.CreateEnumerableAsync(tx);

                Microsoft.ServiceFabric.Data.IAsyncEnumerator<KeyValuePair<string, int>> enumerator = list.GetAsyncEnumerator();

                List<KeyValuePair<string, int>> result = new List<KeyValuePair<string, int>>();

                while (await enumerator.MoveNextAsync(ct))
                {
                    result.Add(enumerator.Current);
                }

                return this.Json(result);
            }
        }

        // PUT api/VoteData/name
        [HttpPut("{name}")]
        public async Task<IActionResult> Put(string name)
        {
            IReliableDictionary<string, int> votesDictionary = await this.stateManager.GetOrAddAsync<IReliableDictionary<string, int>>("counts");

            using (ITransaction tx = this.stateManager.CreateTransaction())
            {
                await votesDictionary.AddOrUpdateAsync(tx, name, 1, (key, oldvalue) => oldvalue + 1);
                await tx.CommitAsync();
            }

            return new OkResult();
        }

        // DELETE api/VoteData/name
        [HttpDelete("{name}")]
        public async Task<IActionResult> Delete(string name)
        {
            IReliableDictionary<string, int> votesDictionary = await this.stateManager.GetOrAddAsync<IReliableDictionary<string, int>>("counts");

            using (ITransaction tx = this.stateManager.CreateTransaction())
            {
                if (await votesDictionary.ContainsKeyAsync(tx, name))
                {
                    await votesDictionary.TryRemoveAsync(tx, name);
                    await tx.CommitAsync();
                    return new OkResult();
                }
                else
                {
                    return new NotFoundResult();
                }
            }
        }
    }
}
```

## <a name="connect-the-services"></a>Conexión de los servicios

En este paso se conectan los dos servicios y se hace que la aplicación web de front-end obtenga y configure la información de votación del servicio back-end.

Service Fabric proporciona una flexibilidad completa en el modo de comunicación con los servicios confiables. Dentro de una única aplicación, es posible que tenga servicios que sean accesibles a través de TCP. Puede haber también otros servicios que podrían estar accesibles a través de una API de REST de HTTP e incluso otros servicios que pueden estar accesibles a través de sockets web. Para más información sobre las opciones disponibles y sus inconvenientes, consulte [Conexión y comunicación con servicios en Service Fabric](service-fabric-connect-and-communicate-with-services.md).

En este tutorial se usa la [Web API de ASP.NET Core](service-fabric-reliable-services-communication-aspnetcore.md) y el [proxy inverso de Service Fabric](service-fabric-reverseproxy.md) para que el servicio web VotingWeb de front-end pueda comunicarse con el servicio VotingData de back-end. El proxy inverso está configurado de manera predeterminada para usar el puerto 19081 y debería funcionar para este tutorial. El puerto del proxy inverso se establece en la plantilla de Azure Resource Manager utilizada para configurar el clúster. Para ver qué puerto se utiliza, busque en la plantilla de clúster en el recurso **Microsoft.ServiceFabric/clusters**: 

```json
"nodeTypes": [
          {
            ...
            "httpGatewayEndpointPort": "[variables('nt0fabricHttpGatewayPort')]",
            "isPrimary": true,
            "vmInstanceCount": "[parameters('nt0InstanceCount')]",
            "reverseProxyEndpointPort": "[parameters('SFReverseProxyPort')]"
          }
        ],
```
Para encontrar el puerto del proxy inverso que se usa en el clúster de desarrollo local, consulte el elemento **HttpApplicationGatewayEndpoint** en el manifiesto del clúster de Service Fabric local:
1. Abra un ventana del explorador y vaya a http:\//localhost:19080 para abrir la herramienta Service Fabric Explorer.
2. Seleccione **Cluster -> Manifest** (Clúster -> Manifiesto).
3. Anote el puerto del elemento HttpApplicationGatewayEndpoint. El valor predeterminado debería ser 19081. Si no es 19081, deberá cambiar el puerto del método GetProxyAddress del siguiente código VotesController.cs.

<a id="updatevotecontroller" name="updatevotecontroller_anchor"></a>

### <a name="update-the-votescontrollercs-file"></a>Actualización del archivo VotesController.cs

En el proyecto **VotingWeb**, abra el archivo *Controllers/VotesController.cs*.  Reemplace el contenido de la definición de clase `VotesController` por el siguiente y guarde los cambios. Si el puerto de proxy inverso que detectó en el paso anterior no es 19081, cambie el puerto que se usa en el método GetProxyAddress de 19081 al puerto que detectó.

```csharp
public class VotesController : Controller
{
    private readonly HttpClient httpClient;
    private readonly FabricClient fabricClient;
    private readonly StatelessServiceContext serviceContext;

    public VotesController(HttpClient httpClient, StatelessServiceContext context, FabricClient fabricClient)
    {
        this.fabricClient = fabricClient;
        this.httpClient = httpClient;
        this.serviceContext = context;
    }

    // GET: api/Votes
    [HttpGet("")]
    public async Task<IActionResult> Get()
    {
        Uri serviceName = VotingWeb.GetVotingDataServiceName(this.serviceContext);
        Uri proxyAddress = this.GetProxyAddress(serviceName);

        ServicePartitionList partitions = await this.fabricClient.QueryManager.GetPartitionListAsync(serviceName);

        List<KeyValuePair<string, int>> result = new List<KeyValuePair<string, int>>();

        foreach (Partition partition in partitions)
        {
            string proxyUrl =
                $"{proxyAddress}/api/VoteData?PartitionKey={((Int64RangePartitionInformation) partition.PartitionInformation).LowKey}&PartitionKind=Int64Range";

            using (HttpResponseMessage response = await this.httpClient.GetAsync(proxyUrl))
            {
                if (response.StatusCode != System.Net.HttpStatusCode.OK)
                {
                    continue;
                }

                result.AddRange(JsonConvert.DeserializeObject<List<KeyValuePair<string, int>>>(await response.Content.ReadAsStringAsync()));
            }
        }

        return this.Json(result);
    }

    // PUT: api/Votes/name
    [HttpPut("{name}")]
    public async Task<IActionResult> Put(string name)
    {
        Uri serviceName = VotingWeb.GetVotingDataServiceName(this.serviceContext);
        Uri proxyAddress = this.GetProxyAddress(serviceName);
        long partitionKey = this.GetPartitionKey(name);
        string proxyUrl = $"{proxyAddress}/api/VoteData/{name}?PartitionKey={partitionKey}&PartitionKind=Int64Range";

        StringContent putContent = new StringContent($"{{ 'name' : '{name}' }}", Encoding.UTF8, "application/json");
        putContent.Headers.ContentType = new MediaTypeHeaderValue("application/json");

        using (HttpResponseMessage response = await this.httpClient.PutAsync(proxyUrl, putContent))
        {
            return new ContentResult()
            {
                StatusCode = (int) response.StatusCode,
                Content = await response.Content.ReadAsStringAsync()
            };
        }
    }

    // DELETE: api/Votes/name
    [HttpDelete("{name}")]
    public async Task<IActionResult> Delete(string name)
    {
        Uri serviceName = VotingWeb.GetVotingDataServiceName(this.serviceContext);
        Uri proxyAddress = this.GetProxyAddress(serviceName);
        long partitionKey = this.GetPartitionKey(name);
        string proxyUrl = $"{proxyAddress}/api/VoteData/{name}?PartitionKey={partitionKey}&PartitionKind=Int64Range";

        using (HttpResponseMessage response = await this.httpClient.DeleteAsync(proxyUrl))
        {
            if (response.StatusCode != System.Net.HttpStatusCode.OK)
            {
                return this.StatusCode((int) response.StatusCode);
            }
        }

        return new OkResult();
    }


    /// <summary>
    /// Constructs a reverse proxy URL for a given service.
    /// Example: http://localhost:19081/VotingApplication/VotingData/
    /// </summary>
    /// <param name="serviceName"></param>
    /// <returns></returns>
    private Uri GetProxyAddress(Uri serviceName)
    {
        return new Uri($"http://localhost:19081{serviceName.AbsolutePath}");
    }

    /// <summary>
    /// Creates a partition key from the given name.
    /// Uses the zero-based numeric position in the alphabet of the first letter of the name (0-25).
    /// </summary>
    /// <param name="name"></param>
    /// <returns></returns>
    private long GetPartitionKey(string name)
    {
        return Char.ToUpper(name.First()) - 'A';
    }
}
```

<a id="walkthrough" name="walkthrough_anchor"></a>

## <a name="walk-through-the-voting-sample-application"></a>Tutorial de la aplicación de ejemplo de votación

La aplicación de votación consta de dos servicios:

* Servicio front-end web (VotingWeb): front-end web de ASP.NET Core, que ofrece servicio a la página web y expone API web para comunicarse con el servicio back-end.
* Servicio back-end (VotingData): servicio web de ASP.NET Core, que expone una API para almacenar los resultados de una votación en un diccionario de confianza que se conserva en el disco.

![Diagrama de la aplicación](./media/service-fabric-tutorial-create-dotnet-app/application-diagram.png)

Al votar en la aplicación, se producen los eventos siguientes:

1. JavaScript envía la solicitud de votación a la API web del servicio front-end web como una solicitud HTTP PUT.

2. El servicio front-end web usa un proxy para localizar y reenviar una solicitud HTTP PUT al servicio back-end.

3. El servicio back-end recibe la solicitud entrante y almacena el resultado actualizado en un diccionario de confianza, que se replica en varios nodos del clúster y se conserva en el disco. Todos los datos de la aplicación se almacenan en el clúster, por lo que no se necesita una base de datos.

## <a name="debug-in-visual-studio"></a>Depurar en Visual Studio

Cuando se depura una aplicación en Visual Studio, se usa un clúster de desarrollo de Service Fabric local. Tiene la opción de ajustar la experiencia de depuración a su escenario. En esta aplicación, los datos se almacenan en el servicio back-end mediante un diccionario de confianza. Visual Studio quita la aplicación de forma predeterminada cuando se detiene el depurador. Cuando se quita la aplicación, los datos del servicio back-end también se quitan. Para conservar los datos entre sesiones de depuración, puede cambiar el **modo de depuración de la aplicación** como una propiedad del proyecto **Voting** en Visual Studio.

Para ver lo que ocurre en el código, siga estos pasos:

1. Abra el archivo **VotingWeb\VotesController.cs** y establezca un punto de interrupción en el método **Put** (línea 72) de la API web.

2. Abra el archivo **VotingData\VoteDataController.cs** y establezca un punto de interrupción en el método **Put** (línea 54) de la API web.

3. Presione **F5** para iniciar la aplicación en modo de depuración.

4. Vuelva al explorador y haga clic en una opción de votación o agregue una nueva opción de votación. Alcanzará el primer punto de interrupción del controlador de API del front-end web.
    

   1. Aquí es donde el código JavaScript del explorador envía una solicitud al controlador de API web del servicio front-end.

      ![Incorporación del servicio front-end de voto](./media/service-fabric-tutorial-create-dotnet-app/addvote-frontend.png)

   2. Primero, construya la dirección URL para el valor de ReverseProxy del servicio back-end **(1)** .
   3. A continuación, envíe la solicitud PUT de HTTP a ReverseProxy **(2)** .
   4. Por último, se devuelve la respuesta del servicio back-end al cliente **(3)** .

5. Presione **F5** para continuar.
   1. Ya está en el punto de interrupción del servicio back-end.

      ![Incorporación del servicio back-end de voto](./media/service-fabric-tutorial-create-dotnet-app/addvote-backend.png)

   2. En la primera línea del método **(1)** se usa `StateManager` para obtener o agregar un diccionario de confianza denominado `counts`.
   3. Todas las interacciones con valores de un diccionario de confianza requieren una transacción. Esta instrucción using **(2)** crea dicha transacción.
   4. Después, en la transacción, se actualiza el valor de la tecla correspondiente para la opción de votación y se confirma la operación **(3)** . Una vez que se devuelve el método Commit, los datos se actualizan en el diccionario y se replican en otros nodos del clúster. Los datos ahora están almacenados de forma segura en el clúster y el servicio back-end puede conmutar por error a otros nodos, mientras sigue teniendo los datos disponibles.
6. Presione **F5** para continuar.

Para detener la sesión de depuración, presione **Mayús+F5**.

## <a name="next-steps"></a>Pasos siguientes

En esta parte del tutorial, ha aprendido a:

> [!div class="checklist"]
> * Crear un servicio de ASP.NET Core Web API como un servicio de confianza con estado
> * Crear un servicio de ASP.NET Core Web Application como un servicio de confianza sin estado
> * Usar el proxy inverso para comunicarse con el servicio con estado

Avance hasta el siguiente tutorial:
> [!div class="nextstepaction"]
> [Implementación de la aplicación en Azure](service-fabric-tutorial-deploy-app-to-party-cluster.md)
