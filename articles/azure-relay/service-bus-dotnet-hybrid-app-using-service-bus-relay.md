---
title: Aplicación híbrida en la nube o local (.NET) de Windows Communication Foundation (WCF) con Azure Relay | Microsoft Docs
description: Aprenda a exponer un servicio WCF local a una aplicación web en la nube con Azure Relay
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 06/23/2020
ms.openlocfilehash: d6299899ee45bbad8b5e5fdbca01805604794d62
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88922555"
---
# <a name="expose-an-on-premises-wcf-service-to-a-web-application-in-the-cloud-by-using-azure-relay"></a>Exposición de un servicio WCF local a una aplicación web en la nube con Azure Relay

En este artículo se muestra cómo compilar una aplicación de nube híbrida con Microsoft Azure y Visual Studio. Va a crear una aplicación que utiliza varios recursos de Azure en la nube. En este tutorial, aprenderá a:

* Crear o adaptar un servicio web existente para su consumo por una solución web.
* Utilizar Windows Communication Foundation (WCF) con el servicio Azure Relay para compartir datos entre una aplicación de Azure y un servicio web hospedado en otra parte.

En este tutorial, hará lo siguiente:

> [!div class="checklist"]
>
> * Instalar los requisitos previos del tutorial.
> * Revise el escenario.
> * Cree un espacio de nombres.
> * Cree un servidor local.
> * Cree una aplicación ASP .NET.
> * Ejecute localmente la aplicación.
> * Implemente la aplicación web en Azure.
> * Ejecute la aplicación en Azure.

## <a name="prerequisites"></a>Prerequisites

Para completar este tutorial, debe cumplir los siguientes requisitos previos:

* Suscripción a Azure. Si no tiene una, [cree una cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* [Visual Studio 2015 o posterior](https://www.visualstudio.com). En los ejemplos de este tutorial se usa Visual Studio 2019.
* SDK de Azure para .NET. Instálelo desde la [página de descargas de SDK](https://azure.microsoft.com/downloads/).

## <a name="how-azure-relay-helps-with-hybrid-solutions"></a>Cómo ayuda Relay de Azure con soluciones híbridas

Las soluciones empresariales suelen estar compuestas de una combinación de código personalizado y funcionalidad existente. El código personalizado aborda los requisitos empresariales nuevos y únicos. Las soluciones y los sistemas que ya están en vigor proporcionan funcionalidad existente.

Los arquitectos de soluciones están comenzando a utilizar la nube para abordar con más facilidad los requisitos de escala y reducir los costes operativos. De esta manera, se dan cuenta de que los recursos de servicio existentes que les gustaría usar como base de sus soluciones se encuentran dentro del firewall corporativo y no resulta sencillo para la solución en la nube. Muchos de los servicios internos no están construidos ni hospedados de una forma que se puedan exponer fácilmente en los servidores perimetrales de la red corporativa.

[Azure Relay](https://azure.microsoft.com/services/service-bus/) toma los servicios web de WCF existentes y permite el acceso seguro a los mismos a las soluciones que están fuera del perímetro corporativo, sin necesidad de realizar cambios molestos en la infraestructura de la red corporativa. Dichos servicios de retransmisión aún se hospedan en el entorno existente, pero delegan la escucha de sesiones y solicitudes de entrada en el servicio de relé hospedado en la nube. Azure Relay también protege dichos servicios del acceso no autorizado mediante el uso de la autenticación de [Firma de acceso compartido (SAS)](../service-bus-messaging/service-bus-sas.md).

## <a name="review-the-scenario"></a>Revisión del escenario

En este tutorial, va a crear un sitio web de ASP.NET que le permitirá ver una lista de productos en la página del inventario de productos.

![Escenario][0]

En este tutorial se asume que la información de los productos se encuentra en un sistema local y se usa Relay de Azure para obtener acceso a dicho sistema. Un servicio web que se ejecuta en una aplicación de consola simple simula esta situación. Contiene un conjunto de productos en memoria. Puede ejecutar esta aplicación de consola en su propio equipo e implementar el rol web en Azure. Al hacerlo, verá cómo el rol web que se ejecuta en el centro de datos de Azure llama al equipo. Esta llamada se produce aunque el equipo casi seguro esté detrás de al menos un firewall y de una capa de traducción de direcciones de red (NAT).

## <a name="set-up-the-development-environment"></a>Configuración del entorno de desarrollo

Antes de comenzar a desarrollar aplicaciones de Azure, obtenga las herramientas y configure el entorno de desarrollo:

1. Instale el SDK de Azure para .NET desde la [página de descargas](https://azure.microsoft.com/downloads/) del SDK.
1. En la columna **.NET**, elija la versión de [Visual Studio](https://www.visualstudio.com) que está usando. En este tutorial se usa Visual Studio 2019.
1. Cuando se le solicite ejecutar o guardar el instalador, haga clic en **Ejecutar**.
1. En el cuadro de diálogo **Instalador de plataforma web**, seleccione **Instalar** y continúe con la instalación.

Cuando la instalación haya finalizado, dispondrá de todo lo necesario para iniciar el desarrollo de la aplicación. El SDK incluye las herramientas que le permiten desarrollar fácilmente aplicaciones Azure en Visual Studio.

## <a name="create-a-namespace"></a>Creación de un espacio de nombres

El primer paso consiste en crear un espacio de nombres y obtener una clave de [firma de acceso compartido (SAS)](../service-bus-messaging/service-bus-sas.md). Un espacio de nombres proporciona un límite de aplicación para cada aplicación que se expone a través del servicio de retransmisión. El sistema genera una clave SAS automáticamente cuando se crea un espacio de nombres de servicio. La combinación del espacio de nombres de servicio y la clave SAS proporciona las credenciales para que Azure autentique el acceso a una aplicación.

[!INCLUDE [relay-create-namespace-portal](../../includes/relay-create-namespace-portal.md)]

## <a name="create-an-on-premises-server"></a>Creación de un servidor local

En primer lugar, cree un sistema de catálogo de productos local ficticio.  Este proyecto es una aplicación de consola de Visual Studio y emplea el [paquete NuGet de Azure Service Bus](https://www.nuget.org/packages/WindowsAzure.ServiceBus/) para incluir las bibliotecas y los valores de configuración de Service Bus. <a name="create-the-project"></a>

1. Inicie Microsoft Visual Studio como administrador. Para hacerlo, haga clic con el botón derecho en el icono del programa Visual Studio y, después, seleccione **Ejecutar como administrador**.
1. En Visual Studio, seleccione **Crear un proyecto nuevo**.
1. En **Crear un proyecto**, seleccione **Aplicación de consola (.NET Framework)** para C# y seleccione **Siguiente**.
1. Asigne al proyecto el nombre *ProductsServer* y seleccione **Crear**.

   ![Configuración del nuevo proyecto][11]

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto **ProductsServer** y después seleccione **Administrar paquetes NuGet**.
1. Seleccione **Examinar** y, después, busque y elija **WindowsAzure.ServiceBus**. Seleccione **Instalar** y acepte las condiciones de uso.

   ![Selección de un paquete NuGet][13]

   Ahora se hace referencia a los ensamblados del cliente requeridos.

1. Agregue una clase nueva para el contrato de su producto. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto **ProductsServer** y seleccione **Agregar** > **Clase**.
1. En **Nombre**, escriba el nombre *ProductsContract.cs* y seleccione **Agregar**.

Realice los siguientes cambios en el código para la solución:

1. En *ProductsContract.cs*, sustituya la definición del espacio de nombres por el siguiente código, que define el contrato del servicio.

    ```csharp
    namespace ProductsServer
    {
        using System.Collections.Generic;
        using System.Runtime.Serialization;
        using System.ServiceModel;

        // Define the data contract for the service
        [DataContract]
        // Declare the serializable properties.
        public class ProductData
        {
            [DataMember]
            public string Id { get; set; }
            [DataMember]
            public string Name { get; set; }
            [DataMember]
            public string Quantity { get; set; }
        }

        // Define the service contract.
        [ServiceContract]
        interface IProducts
        {
            [OperationContract]
            IList<ProductData> GetProducts();

        }

        interface IProductsChannel : IProducts, IClientChannel
        {
        }
    }
    ```

1. En *Program.cs*, sustituya la definición del espacio de nombres por el siguiente código, que agrega el servicio de perfil y su host.

    ```csharp
    namespace ProductsServer
    {
        using System;
        using System.Linq;
        using System.Collections.Generic;
        using System.ServiceModel;

        // Implement the IProducts interface.
        class ProductsService : IProducts
        {

            // Populate array of products for display on website
            ProductData[] products =
                new []
                    {
                        new ProductData{ Id = "1", Name = "Rock",
                                         Quantity = "1"},
                        new ProductData{ Id = "2", Name = "Paper",
                                         Quantity = "3"},
                        new ProductData{ Id = "3", Name = "Scissors",
                                         Quantity = "5"},
                        new ProductData{ Id = "4", Name = "Well",
                                         Quantity = "2500"},
                    };

            // Display a message in the service console application
            // when the list of products is retrieved.
            public IList<ProductData> GetProducts()
            {
                Console.WriteLine("GetProducts called.");
                return products;
            }

        }

        class Program
        {
            // Define the Main() function in the service application.
            static void Main(string[] args)
            {
                var sh = new ServiceHost(typeof(ProductsService));
                sh.Open();

                Console.WriteLine("Press ENTER to close");
                Console.ReadLine();

                sh.Close();
            }
        }
    }
    ```

1. En el **Explorador de soluciones**, haga doble clic en **App.config** para abrir el archivo en el editor de Visual Studio. En la parte inferior del elemento `<system.ServiceModel>`, pero aún dentro de `<system.ServiceModel>`, agregue el siguiente código XML. Asegúrese de reemplazar `yourServiceNamespace` por el nombre de su espacio de nombres y `yourKey` por la clave SAS que recuperó anteriormente en el portal:

    ```xml
    <system.serviceModel>
    ...
      <services>
         <service name="ProductsServer.ProductsService">
           <endpoint address="sb://yourServiceNamespace.servicebus.windows.net/products" binding="netTcpRelayBinding" contract="ProductsServer.IProducts" behaviorConfiguration="products"/>
         </service>
      </services>
      <behaviors>
         <endpointBehaviors>
           <behavior name="products">
             <transportClientEndpointBehavior>
                <tokenProvider>
                   <sharedAccessSignature keyName="RootManageSharedAccessKey" key="yourKey" />
                </tokenProvider>
             </transportClientEndpointBehavior>
           </behavior>
         </endpointBehaviors>
      </behaviors>
    </system.serviceModel>
    ```

    > [!NOTE]
    > El error provocado por `transportClientEndpointBehavior` es solo una advertencia y no es un problema de bloqueo en este ejemplo.

1. Aún en *App.config*, en el elemento `<appSettings>`, reemplace el valor de la cadena de conexión por la cadena de conexión que obtuvo antes en el portal.

    ```xml
    <appSettings>
       <!-- Service Bus specific app settings for messaging connections -->
       <add key="Microsoft.ServiceBus.ConnectionString"
           value="Endpoint=sb://yourNamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=yourKey"/>
    </appSettings>
    ```

1. Seleccione Ctrl+Mayús+B, o bien, seleccione **Compilar** > **Compilar solución** para compilar la aplicación y comprobar la exactitud del trabajo hasta el momento.

## <a name="create-an-aspnet-application"></a>Creación de una aplicación ASP.NET

En esta sección va a crear una aplicación de ASP.NET simple que mostrará los datos recuperados del servicio de producto.

### <a name="create-the-project"></a>Creación del proyecto

1. Asegúrese de que se está ejecutando Visual Studio como administrador.
1. En Visual Studio, seleccione **Crear un proyecto nuevo**.
1. En **Crear un proyecto**, seleccione **Aplicación web ASP.NET Core (.NET Framework)** para C# y, después, **Siguiente**.
1. Asigne al proyecto el nombre *ProductsPortal* y seleccione **Crear**.
1. En **Crear una aplicación web ASP.NET**, elija **MVC** y seleccione **Cambiar** en **Autenticación**.

   ![Selección de la aplicación web ASP.NET][16]

1. En **Cambiar autenticación**, elija **Sin autenticación** y, a continuación, seleccione **Aceptar**. En este tutorial, va a implementar una aplicación que no necesita un inicio de sesión de usuario.

    ![Especificación de la autenticación][18]

1. De vuelta en **Crear una aplicación web ASP.NET**, seleccione **Crear** para crear la aplicación MVC.
1. Configure los recursos de Azure para una aplicación web nueva. Siga los pasos descritos en la sección [Publicación de la aplicación web](../app-service/quickstart-dotnet-framework.md#launch-the-publish-wizard). A continuación, vuelva a este tutorial y siga con el paso siguiente.
1. En **Explorador de soluciones**, haga clic con el botón derecho en **Modelos** y después seleccione **Agregar** > **Clase**.
1. Asigne a la clase el nombre *Product.cs* y, después, seleccione **Agregar**.

    ![Creación del modelo de producto][17]

### <a name="modify-the-web-application"></a>Modificación de la aplicación web

1. En el archivo *Product.cs* en Visual Studio, sustituya la definición del espacio de nombres existente con el código siguiente:

   ```csharp
    // Declare properties for the products inventory.
    namespace ProductsWeb.Models
    {
       public class Product
       {
           public string Id { get; set; }
           public string Name { get; set; }
           public string Quantity { get; set; }
       }
    }
    ```

1. En el **Explorador de soluciones**, expanda **Controladores** y luego haga doble clic en el archivo **HomeController.cs** para abrirlo en Visual Studio.
1. En el archivo *HomeController.cs*, sustituya la definición de espacio de nombres existente con el código siguiente:

    ```csharp
    namespace ProductsWeb.Controllers
    {
        using System.Collections.Generic;
        using System.Web.Mvc;
        using Models;

        public class HomeController : Controller
        {
            // Return a view of the products inventory.
            public ActionResult Index(string Identifier, string ProductName)
            {
                var products = new List<Product>
                    {new Product {Id = Identifier, Name = ProductName}};
                return View(products);
            }
         }
    }
    ```

1. En el **Explorador de soluciones**, expanda **Vistas** > **Compartido** y luego haga doble clic en **_Layout.cshtml** para abrirlo en el editor de Visual Studio.
1. Cambie todas las repeticiones de `My ASP.NET Application` por *Northwind Traders Products*.
1. Quite los vínculos `Home`, `About` y `Contact`. En el siguiente ejemplo, elimine el código resaltado.

    ![Eliminación de los elementos de lista generados][41]

1. En el **Explorador de soluciones**, expanda **Vistas** > **Inicio** y luego haga doble clic en **Index.cshtml** para abrirlo en el editor de Visual Studio. Reemplace todo el contenido del archivo con el código siguiente:

   ```html
   @model IEnumerable<ProductsWeb.Models.Product>

   @{
            ViewBag.Title = "Index";
   }

   <h2>Prod Inventory</h2>

   <table>
             <tr>
                 <th>
                     @Html.DisplayNameFor(model => model.Name)
                 </th>
                 <th></th>
                 <th>
                     @Html.DisplayNameFor(model => model.Quantity)
                 </th>
             </tr>

   @foreach (var item in Model) {
             <tr>
                 <td>
                     @Html.DisplayFor(modelItem => item.Name)
                 </td>
                 <td>
                     @Html.DisplayFor(modelItem => item.Quantity)
                 </td>
             </tr>
   }

   </table>
   ```

1. Para comprobar la precisión del trabajo realizado hasta el momento, puede seleccionar Ctrl+Mayús+B para compilar el proyecto.

### <a name="run-the-app-locally"></a>Ejecución de la aplicación de forma local

Ejecute la aplicación para comprobar que funciona.

1. Asegúrese de que **ProductsPortal** es el proyecto activo. Haga clic con el botón derecho en el nombre del proyecto en el **Explorador de soluciones** y seleccione **Establecer como proyecto de inicio**.
1. En Visual Studio, seleccione F5.

La aplicación debería aparecer ejecutándose en un explorador.

![Aplicación web][21]

## <a name="put-the-pieces-together"></a>Combinación de todos los componentes

El siguiente paso es conectar el servidor de productos local con la aplicación ASP.NET.

1. Si no está abierto, abra en Visual Studio el proyecto **ProductsPortal** que ha creado en la sección [Creación de una aplicación ASP.NET](#create-an-aspnet-application).
1. Agregue el paquete NuGet a las referencias del proyecto de forma similar al paso de la sección [Creación de un servidor local](#create-an-on-premises-server). En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto **ProductsPortal** y luego seleccione **Administrar paquetes NuGet**.
1. Busque *WindowsAzure.ServiceBus* y seleccione el elemento **WindowsAzure.ServiceBus**. Después, finalice la instalación y cierre este cuadro de diálogo.
1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto **ProductsPortal** y, a continuación, seleccione **Agregar** > **Elemento existente**.
1. Desplácese al archivo *ProductsContract.cs* desde el proyecto de consola **ProductsServer**. Resalte *ProductsContract.cs*. Seleccione la flecha abajo situada junto a **Agregar** y, después, elija **Agregar como vínculo**.

   ![Incorporación como vínculo][24]

1. Ahora abra el archivo *HomeController.cs* en el editor de Visual Studio y sustituya la definición del espacio de nombres por el código siguiente. Asegúrese de reemplazar `yourServiceNamespace` por el nombre de su espacio de nombres de servicio y `yourKey` por la clave de SAS. Este código permite que el cliente llame al servicio local y que se devuelva el resultado de la llamada.

   ```csharp
   namespace ProductsWeb.Controllers
   {
       using System.Linq;
       using System.ServiceModel;
       using System.Web.Mvc;
       using Microsoft.ServiceBus;
       using Models;
       using ProductsServer;

       public class HomeController : Controller
       {
           // Declare the channel factory.
           static ChannelFactory<IProductsChannel> channelFactory;

           static HomeController()
           {
               // Create shared access signature token credentials for authentication.
               channelFactory = new ChannelFactory<IProductsChannel>(new NetTcpRelayBinding(),
                   "sb://yourServiceNamespace.servicebus.windows.net/products");
               channelFactory.Endpoint.Behaviors.Add(new TransportClientEndpointBehavior {
                   TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(
                       "RootManageSharedAccessKey", "yourKey") });
           }

           public ActionResult Index()
           {
               using (IProductsChannel channel = channelFactory.CreateChannel())
               {
                   // Return a view of the products inventory.
                   return this.View(from prod in channel.GetProducts()
                                    select
                                        new Product { Id = prod.Id, Name = prod.Name,
                                            Quantity = prod.Quantity });
               }
           }
       }
   }
   ```

1. En el **Explorador de soluciones**, haga clic con el botón derecho en la solución **ProductsPortal**. Asegúrese de hacer clic con el botón derecho en la solución, no en el proyecto. Seleccione **Agregar** > **Proyecto existente**.
1. Desplácese al proyecto **ProductsServer** y haga doble clic en la solución *ProductsServer.csproj* para agregarla.
1. **ProductsServer** debe estar en ejecución para mostrar los datos en **ProductsPortal**. En el **Explorador de soluciones**, haga clic con el botón derecho en la solución **ProductsPortal** y, a continuación, seleccione **Propiedades** para mostrar **Páginas de propiedades**.
1. Seleccione **Propiedades comunes** > **Proyecto de inicio** y elija **Proyectos de inicio múltiples**. Asegúrese de que aparezcan **ProductsServer** y **ProductsPortal** en ese orden y que la **acción** para ambos sea **Iniciar**.

      ![Proyectos de inicio múltiples][25]

1. Seleccione **Propiedades comunes** > **Dependencias del proyecto** en el lado izquierdo.
1. En **Proyectos**, elija **ProductsPortal**. Asegúrese de que **ProductsServer** esté seleccionado.

    ![Dependencias del proyecto][26]

1. En **Proyectos**, elija **ProductsServer**. Asegúrese de que **ProductsPortal** no está seleccionado y, después, seleccione **Aceptar** para guardar los cambios.

## <a name="run-the-project-locally"></a>Ejecución del proyecto de forma local

Para probar la aplicación localmente, en Visual Studio, seleccione F5. El servidor local, **ProductsServer**, se debe iniciar primero y luego la aplicación **ProductsPortal** se debe iniciar en una ventana del explorador. Esta vez verá que el inventario de productos muestra los datos recuperados del sistema local del servicio de productos.

![Aplicación web][10]

Seleccione **Actualizar** en la página **ProductsPortal**. Cada vez que actualice la página, verá que la aplicación de servidor muestra un mensaje cuando se llama a `GetProducts()` desde **ProductsServer**.

Cierre ambas aplicaciones antes de continuar con el paso siguiente.

## <a name="deploy-the-productsportal-project-to-an-azure-web-app"></a>Implementación del proyecto ProductsPortal en una aplicación web de Azure

El siguiente paso consiste en volver a publicar el front-end de **ProductsPortal** de la aplicación web de Azure.

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto **ProductsPortal** y seleccione **Publicar**. En la página **Publicar**, seleccione **Publicar**.

   > [!NOTE]
   > Puede que vea un mensaje de error en la ventana del explorador cuando el proyecto web **ProductsPortal** se inicie automáticamente después de la implementación. Esto es normal y se produce porque la aplicación **ProductsServer** no se está ejecutando todavía.
   >

1. Copie la dirección URL de la aplicación web implementada. Necesitará el URI más adelante. También puede obtener esta dirección URL en la ventana de **actividad de Azure App Service** en Visual Studio:

   ![Dirección URL de la aplicación implementada][9]

1. Cierre la ventana del explorador para detener la aplicación en ejecución.

<a name="set-productsportal-as-web-app"></a>Antes de ejecutar la aplicación en la nube, debe asegurarse de que **ProductsPortal** se inicie desde dentro de Visual Studio como una aplicación web.

1. En Visual Studio, haga clic con el botón derecho en el proyecto **ProductsPortal** y seleccione **Propiedades**.
1. Seleccione **Web**. En **Acción de inicio**, elija **Dirección URL de inicio**. Escriba la dirección URL de la aplicación web implementada anteriormente, en este ejemplo, `https://productsportal20190906122808.azurewebsites.net/`.

    ![Dirección URL de inicio][27]

1. Seleccione **Archivo** > **Guardar todo**.
1. Seleccione **Compilar** > **Recompilar solución**.

## <a name="run-the-application"></a>Ejecución de la aplicación

Seleccione F5 para compilar y ejecutar la aplicación. Debe iniciarse primero en el servidor local, que es la aplicación de consola **ProductsServer**, y después la aplicación **ProductsPortal** en una ventana de explorador, tal como se muestra aquí:

   ![Ejecute la aplicación web en Azure][1]

El inventario de productos muestra los datos recuperados del sistema local del servicio de productos y muestra esos datos en la aplicación web. Compruebe la dirección URL para asegurarse de que **ProductsPortal** se ejecuta en la nube como una aplicación web de Azure.

   > [!IMPORTANT]
   > La aplicación de consola **ProductsServer** se debe estar ejecutando y debe poder suministrar los datos a la aplicación **ProductsPortal**. Si el explorador muestra un error, espere unos segundos a que **ProductsServer** se cargue y muestre el siguiente mensaje; después, actualice el explorador.
   >

En el explorador, actualice la página **ProductsPortal**. Cada vez que actualice la página, verá que la aplicación de servidor muestra un mensaje cuando se llama a `GetProducts()` desde **ProductsServer**.

![Salida actualizada][38]

## <a name="next-steps"></a>Pasos siguientes

Avance al siguiente tutorial:

> [!div class="nextstepaction"]
>[Exposición de un servicio WCF local en un cliente WCF fuera de la red](service-bus-relay-tutorial.md)

[0]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hybrid.png
[1]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/run-web-app.png
[NuGet]: https://nuget.org

[11]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/configure-productsserver.png
[13]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/install-nuget-service-bus-productsserver.png
[15]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-2.png
[16]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/choose-web-application-template.png
[17]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/add-class-productsportal.png
[18]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/change-authentication.png
[9]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/web-publish-activity.png
[10]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/run-web-app-locally.png

[21]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/run-web-app-locally-no-content.png
[24]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/add-existing-item-link.png
[25]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-13.png
[26]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-14.png
[27]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/launch-app-as-web-app.png

[36]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/App2.png
[37]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-service1.png
[38]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-service2.png
[41]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-multi-tier-40.png
[43]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-43.png
