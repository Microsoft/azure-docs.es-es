---
title: 'Tutorial: Tutorial de REST mediante Azure Relay'
description: 'Tutorial: Creación de una aplicación host de Azure Relay que expone una interfaz basada en REST.'
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.date: 06/23/2020
ms.openlocfilehash: 0620f55650d0e4da0cd7a616649df952f3017455
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88922334"
---
# <a name="tutorial-azure-wcf-relay-rest-tutorial"></a>Tutorial: Tutorial de REST de Azure WCF Relay

En este tutorial se describe cómo compilar una aplicación host de Azure Relay que expone una interfaz basada en REST. REST permite a un cliente web, como un explorador web, tener acceso a la API de Service Bus a través de solicitudes HTTP.

En este tutorial se usa el modelo de programación REST de Windows Communication Foundation (WCF) para construir un servicio REST en Azure Relay. Para más información, consulte [Modelo de programación REST de WCF](/dotnet/framework/wcf/feature-details/wcf-web-http-programming-model) y [Diseño e implementación de servicios](/dotnet/framework/wcf/designing-and-implementing-services).

En este tutorial, hará lo siguiente:

> [!div class="checklist"]
>
> * Instalar los requisitos previos del tutorial.
> * Creación de un espacio de nombres de Relay
> * Definir un contrato de servicio WCF basado en REST.
> * Implementar el contrato de WCF basado en REST.
> * Hospedar y ejecutar el servicio WCF basado en REST.
> * Ejecutar y probar el servicio.

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, debe cumplir los siguientes requisitos previos:

* Suscripción a Azure. Si no tiene una, [cree una cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* [Visual Studio 2015 o posterior](https://www.visualstudio.com). En los ejemplos de este tutorial se usa Visual Studio 2019.
* SDK de Azure para .NET. Instálelo desde la [página de descargas de SDK](https://azure.microsoft.com/downloads/).

## <a name="create-a-relay-namespace"></a>Creación de un espacio de nombres de Relay

Para comenzar a usar las características de Relay en Azure, primero debe crear un espacio de nombres de servicio. Un espacio de nombres proporciona un contenedor con un ámbito para el desvío de recursos de Azure en la aplicación. Siga [estas instrucciones](relay-create-namespace-portal.md) para crear un espacio de nombres de Relay.

## <a name="define-a-rest-based-wcf-service-contract-to-use-with-azure-relay"></a>Definición de un contrato de servicio de WCF basado en REST para utilizarlo con Azure Relay

Cuando se crea un servicio WCF de tipo REST, debe definir el contrato. El contrato especifica qué operaciones admite el host. Una operación de servicio se parece a un método de servicio web. Defina un contrato con una interfaz de C++, C# o Visual Basic. Cada método de la interfaz corresponde a una operación de servicio específica. Aplique el atributo [ServiceContractAttribute](/dotnet/api/system.servicemodel.servicecontractattribute) a cada interfaz y el atributo [OperationContractAttribute](/dotnet/api/system.servicemodel.operationcontractattribute) a cada operación. 

> [!TIP]
> Si un método en una interfaz que tiene [ServiceContractAttribute](/dotnet/api/system.servicemodel.servicecontractattribute) no tiene [OperationContractAttribute](/dotnet/api/system.servicemodel.operationcontractattribute), ese método no se expone. El código utilizado para estas tareas aparece en el ejemplo que sigue al procedimiento.

La diferencia principal entre un contrato de WCF y un contrato de estilo REST es la adición de una propiedad a [OperationContractAttribute](/dotnet/api/system.servicemodel.operationcontractattribute): [WebGetAttribute](/dotnet/api/system.servicemodel.web.webgetattribute). Esta propiedad permite asignar un método de la interfaz a un método en el otro lado de la interfaz. En este ejemplo, se usa el atributo [WebGetAttribute](/dotnet/api/system.servicemodel.web.webgetattribute) para vincular un método a `HTTP GET`. Este enfoque permite a Service Bus recuperar e interpretar con precisión los comandos enviados a la interfaz.

### <a name="to-create-a-contract-with-an-interface"></a>Para crear un contrato con una interfaz

1. Inicie Microsoft Visual Studio como administrador. Para hacerlo, haga clic con el botón derecho en el icono del programa Visual Studio y, después, seleccione **Ejecutar como administrador**.
1. En Visual Studio, seleccione **Crear un proyecto nuevo**.
1. En **Crear un proyecto**, elija **Aplicación de consola (.NET Framework)** para C# y seleccione **Siguiente**.
1. Asigne al proyecto el nombre de *ImageListener*. Utilice la **ubicación** personalizada y, a continuación, seleccione **Crear**.

   Para un proyecto de C#, Visual Studio crea un archivo *Program.cs*. Esta clase contiene un método `Main()` vacío, necesario para que un proyecto de aplicación de consola se compile correctamente.

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto **ImageListener** y seleccione **Administrar paquetes NuGet**.
1. Seleccione **Examinar** y, después, busque y elija **WindowsAzure.ServiceBus**. Seleccione **Instalar** y acepte las condiciones de uso.

    Este paso permite agregar referencias a Service Bus y *System.ServiceModel.dll*. Este paquete agrega automáticamente referencias a las bibliotecas de Service Bus y a `System.ServiceModel` de WCF.

1. Agregue explícitamente una referencia a `System.ServiceModel.Web.dll` en el proyecto. En el **Explorador de soluciones**, haga clic con el botón derecho en **Referencias** en la carpeta del proyecto y seleccione **Agregar referencia**.
1. En **Agregar referencia**, seleccione **Marco** y escriba *System.ServiceModel.Web* en **Buscar**. Active la casilla **System.ServiceModel.Web** y haga clic en **Aceptar**.

A continuación, realice los siguientes cambios en el código del proyecto:

1. Agregue las siguientes instrucciones `using` al principio del archivo *Program.cs*.

    ```csharp
    using System.ServiceModel;
    using System.ServiceModel.Channels;
    using System.ServiceModel.Web;
    using System.IO;
    ```

    * [System.ServiceModel](/dotnet/api/system.servicemodel) es el espacio de nombres que permite el acceso mediante programación a las características básicas de WCF. WCF Relay utiliza muchos de los objetos y atributos de WCF para definir contratos de servicio. Usará este espacio de nombres en la mayoría de las aplicaciones de Relay.
    * [System.ServiceModel.Channels](/dotnet/api/system.servicemodel.channels) ayuda a definir el canal, que es el objeto a través del cual se comunica con Azure Relay y el explorador web del cliente.
    * [System.ServiceModel.Web](/dotnet/api/system.servicemodel.web) contiene los tipos que permiten crear aplicaciones basadas en web.

1. Cambie el nombre al espacio de nombres `ImageListener` a `Microsoft.ServiceBus.Samples`.

    ```csharp
    namespace Microsoft.ServiceBus.Samples
    {
        ...
    ```

1. Inmediatamente después de la llave de apertura de la declaración del espacio de nombres, defina una nueva interfaz denominada `IImageContract` y aplique el atributo `ServiceContractAttribute` a la interfaz con un valor de `https://samples.microsoft.com/ServiceModel/Relay/RESTTutorial1`. 

    ```csharp
    [ServiceContract(Name = "ImageContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/RESTTutorial1")]
    public interface IImageContract
    {
    }
    ```

    El valor del espacio de nombres difiere del espacio de nombres que utiliza en todo el ámbito de su código. El valor del espacio de nombres es el identificador único de este contrato, y debe tener información de la versión. Para obtener más información, vea [Control de versiones del servicio](/dotnet/framework/wcf/service-versioning). La especificación del espacio de nombres impide explícitamente que el valor del espacio de nombres predeterminado se agregue al nombre del contrato.

1. Dentro de la interfaz `IImageContract`, declare un método para la operación sencilla que el contrato `IImageContract` expone en la interfaz y aplique el atributo `OperationContract` al método que desea exponer como parte del contrato público de Service Bus.

    ```csharp
    public interface IImageContract
    {
        [OperationContract]
        Stream GetImage();
    }
    ```

1. En el atributo `OperationContract`, agregue el valor `WebGet`.

    ```csharp
    public interface IImageContract
    {
        [OperationContract, WebGet]
        Stream GetImage();
    }
    ```

   Al agregar el valor `WebGet`, permite que el servicio de retransmisión enrute las solicitudes HTTP GET a `GetImage` y convierta los valores devueltos de `GetImage` en una respuesta `HTTP GETRESPONSE`. Más adelante en el tutorial, utilizará un explorador web para tener acceso a este método y mostrar la imagen en el explorador.

1. Inmediatamente después de la definición `IImageContract`, declare un canal que herede de las interfaces `IImageContract` y `IClientChannel`.

    ```csharp
    public interface IImageChannel : IImageContract, IClientChannel { }
    ```

   Un canal es el objeto de WCF a través del cual el servicio y el cliente se pasan información entre sí. Más adelante, creará el canal en su aplicación host. Azure Relay usa después este canal para pasar las solicitudes HTTP GET del explorador a su implementación de `GetImage`. La retransmisión también usa el canal para tomar el valor devuelto de `GetImage` y convertirlo en `HTTP GETRESPONSE` para el explorador del cliente.

1. Seleccione **Compilar** > **Compilar solución** para confirmar la precisión del trabajo realizado.

### <a name="example-that-defines-a-wcf-relay-contract"></a>Ejemplo que define un contrato de WCF Relay

En el ejemplo de código siguiente se muestra una interfaz básica que define un contrato de WCF Relay.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.ServiceModel;
using System.ServiceModel.Channels;
using System.ServiceModel.Web;
using System.IO;

namespace Microsoft.ServiceBus.Samples
{

    [ServiceContract(Name = "IImageContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IImageContract
    {
        [OperationContract, WebGet]
        Stream GetImage();
    }

    public interface IImageChannel : IImageContract, IClientChannel { }

    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

## <a name="implement-the-rest-based-wcf-service-contract"></a>Implementación del contrato de servicio de WCF basado en REST

Para crear un servicio de WCF Relay de tipo REST, primero debe crear el contrato mediante una interfaz. El siguiente paso es implementar la interfaz. Este procedimiento implica la creación de una clase denominada `ImageService` que implemente la interfaz `IImageContract` definida por el usuario. Después de implementar el contrato, a continuación se configura la interfaz mediante un archivo *App.config*. El archivo de configuración contiene la información necesaria para la aplicación. Esta información incluye el nombre del servicio, el nombre del contrato y el tipo de protocolo que se utiliza para comunicarse con el servicio de retransmisión. El código utilizado para estas tareas aparece en el ejemplo que sigue al procedimiento.

Al igual que con los pasos anteriores, hay pocas diferencias entre implementar un contrato de estilo REST y un contrato de WCF Relay.

### <a name="to-implement-a-rest-style-service-bus-contract"></a>Para implementar un contrato de Service Bus de estilo REST

1. Cree una nueva clase denominada `ImageService` directamente después de la definición de la interfaz `IImageContract`. La clase `ImageService` implementa la interfaz `IImageContract`.

    ```csharp
    class ImageService : IImageContract
    {
    }
    ```

    Al igual que otras implementaciones de interfaz, puede implementar la definición en un archivo diferente. Sin embargo, para este tutorial, la implementación aparece en el mismo archivo que la definición de interfaz y el método `Main()`.

1. Aplique el atributo [ServiceBehaviorAttribute](/dotnet/api/system.servicemodel.servicebehaviorattribute) a la clase `IImageService` para indicar que la clase es una implementación de un contrato de WCF.

    ```csharp
    [ServiceBehavior(Name = "ImageService", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    class ImageService : IImageContract
    {
    }
    ```

    Como se mencionó anteriormente, este espacio de nombres no es un espacio de nombres tradicional. Forma parte de la arquitectura de WCF que identifica el contrato. Para más información, consulte [Nombres de contratos de datos](/dotnet/framework/wcf/feature-details/data-contract-names/).

1. Agregue una imagen *.jpg* al proyecto. Este archivo es una imagen que el servicio muestra en el explorador de recepción.

   1. Haga clic con el botón derecho en el proyecto y seleccione **Agregar**.
   1. A continuación, seleccione **Elemento existente**.
   1. Utilice **Agregar elemento existente** para buscar un .jpg adecuado y, después, seleccione **Agregar**. Al agregar el archivo, seleccione **Todos los archivos** en la lista desplegable situada junto a **Nombre de archivo**.

   En el resto de este tutorial se supone que el nombre de la imagen es *image.jpg*. Si tiene un archivo diferente, debe cambiar el nombre de la imagen o cambiar su código para compensar.

1. Para asegurarse de que el servicio en ejecución pueda encontrar el archivo de imagen, en el **Explorador de soluciones**, haga clic con el botón derecho en ese archivo y, luego, seleccione **Propiedades**. En **Propiedades**, establezca **Copiar en el directorio de salida** en **Copiar si es posterior**.

1. Use el procedimiento que se describe en [Para crear un contrato con una interfaz](#to-create-a-contract-with-an-interface) para agregar una referencia al ensamblado *System.Drawing.dll* en el proyecto.

1. Agregue las siguientes instrucciones `using` asociadas:

    ```csharp
    using System.Drawing;
    using System.Drawing.Imaging;
    using Microsoft.ServiceBus;
    using Microsoft.ServiceBus.Web;
    ```

1. En la clase `ImageService`, agregue el siguiente constructor que carga el mapa de bits y se prepara para enviarlo al explorador del cliente:

    ```csharp
    class ImageService : IImageContract
    {
        const string imageFileName = "image.jpg";
   
        Image bitmap;
   
        public ImageService()
        {
            this.bitmap = Image.FromFile(imageFileName);
        }
    }
    ```

1. Directamente después del código anterior, agregue el siguiente método `GetImage` en la clase `ImageService` para devolver un mensaje HTTP que contiene la imagen.

    ```csharp
    public Stream GetImage()
    {
        MemoryStream stream = new MemoryStream();
        this.bitmap.Save(stream, ImageFormat.Jpeg);
   
        stream.Position = 0;
        WebOperationContext.Current.OutgoingResponse.ContentType = "image/jpeg";
   
        return stream;
    }
    ```

    Esta implementación usa `MemoryStream` para recuperar la imagen y prepararla para el streaming al explorador. Este inicia la posición de la secuencia en cero, declara el contenido de la secuencia como *.jpg* y transmite la información.

1. Seleccione **Compilar** > **Compilar solución**.

### <a name="to-define-the-configuration-for-running-the-web-service-on-service-bus"></a>Para definir la configuración para ejecutar el servicio web en Service Bus

1. En el **Explorador de soluciones**, haga doble clic en **App.config** para abrir el archivo en el editor de Visual Studio.

    El archivo *App.config* incluye el nombre del servicio, el punto de conexión y el enlace. El punto de conexión es la ubicación que Azure Relay expone para que los clientes y hosts se comuniquen entre sí. El enlace es el tipo de protocolo que se utiliza para la comunicación. La principal diferencia aquí es que el punto de conexión de servicio configurado hace referencia a un enlace [WebHttpRelayBinding](/dotnet/api/microsoft.servicebus.webhttprelaybinding).

1. El elemento XML `<system.serviceModel>` es un elemento de WCF que define uno o varios servicios. En este caso, se utiliza para definir el nombre del servicio y el punto de conexión. En la parte inferior del elemento `<system.serviceModel>` (pero aún dentro de `<system.serviceModel>`), agregue un elemento `<bindings>` con el siguiente contenido:

    ```xml
    <bindings>
        <!-- Application Binding -->
        <webHttpRelayBinding>
            <binding name="default">
                <security relayClientAuthenticationType="None" />
            </binding>
        </webHttpRelayBinding>
    </bindings>
    ```

    Este contenido define los enlaces utilizados en la aplicación. Puede definir varios enlaces, pero para este tutorial va a definir solo uno.

    Este código anterior define un enlace [WebHttpRelayBinding](/dotnet/api/microsoft.servicebus.webhttprelaybinding) de WCF Relay con `relayClientAuthenticationType` establecido en `None`. Este valor indica que un punto de conexión que usa este enlace no requiere una credencial de cliente.

1. Después del elemento `<bindings>`, agregue un elemento `<services>`. Al igual que los enlaces, puede definir varios servicios en un solo archivo de configuración. Sin embargo, para este tutorial, solo se define uno.

    ```xml
    <services>
        <!-- Application Service -->
        <service name="Microsoft.ServiceBus.Samples.ImageService"
             behaviorConfiguration="default">
            <endpoint name="RelayEndpoint"
                    contract="Microsoft.ServiceBus.Samples.IImageContract"
                    binding="webHttpRelayBinding"
                    bindingConfiguration="default"
                    behaviorConfiguration="sbTokenProvider"
                    address="" />
        </service>
    </services>
    ```

    Este contenido configura un servicio que utiliza el valor predeterminado definido anteriormente `webHttpRelayBinding`. También usa el valor predeterminado `sbTokenProvider`, que se define en el paso siguiente.

1. Después del elemento `<services>`, cree un elemento `<behaviors>` con el contenido siguiente, pero reemplace `SAS_KEY` por la clave de firma de acceso compartido (SAS). Para obtener una clave SAS de [Azure portal][Azure portal], consulte [Obtención de las credenciales de administración](service-bus-relay-tutorial.md#get-management-credentials).

    ```xml
    <behaviors>
        <endpointBehaviors>
            <behavior name="sbTokenProvider">
                <transportClientEndpointBehavior>
                    <tokenProvider>
                        <sharedAccessSignature keyName="RootManageSharedAccessKey" key="YOUR_SAS_KEY" />
                    </tokenProvider>
                </transportClientEndpointBehavior>
            </behavior>
            </endpointBehaviors>
            <serviceBehaviors>
                <behavior name="default">
                    <serviceDebug httpHelpPageEnabled="false" httpsHelpPageEnabled="false" />
                </behavior>
            </serviceBehaviors>
    </behaviors>
    ```

1. Aún en *App.config*, en el elemento `<appSettings>`, reemplace todo el valor de la cadena de conexión por la cadena de conexión que obtuvo antes en el portal.

    ```xml
    <appSettings>
       <!-- Service Bus specific app settings for messaging connections -->
       <add key="Microsoft.ServiceBus.ConnectionString"
           value="Endpoint=sb://yourNamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=YOUR_SAS_KEY"/>
    </appSettings>
    ```

1. Seleccione **Compilar** > **Compilar solución** para compilar toda la solución.

### <a name="example-that-implements-the-rest-based-wcf-service-contract"></a>Ejemplo que implementa el contrato de servicio WCF basado en REST

El código siguiente muestra la implementación de contrato y servicio para un servicio basado en REST que se ejecuta en Service Bus con el enlace `WebHttpRelayBinding`.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.ServiceModel;
using System.ServiceModel.Channels;
using System.ServiceModel.Web;
using System.IO;
using System.Drawing;
using System.Drawing.Imaging;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Web;

namespace Microsoft.ServiceBus.Samples
{


    [ServiceContract(Name = "ImageContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IImageContract
    {
        [OperationContract, WebGet]
        Stream GetImage();
    }

    public interface IImageChannel : IImageContract, IClientChannel { }

    [ServiceBehavior(Name = "ImageService", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    class ImageService : IImageContract
    {
        const string imageFileName = "image.jpg";

        Image bitmap;

        public ImageService()
        {
            this.bitmap = Image.FromFile(imageFileName);
        }

        public Stream GetImage()
        {
            MemoryStream stream = new MemoryStream();
            this.bitmap.Save(stream, ImageFormat.Jpeg);

            stream.Position = 0;
            WebOperationContext.Current.OutgoingResponse.ContentType = "image/jpeg";

            return stream;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

En el ejemplo siguiente se muestra el archivo *App.config* asociado con el servicio.

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <startup> 
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5.2"/>
    </startup>
    <system.serviceModel>
        <extensions>
            <!-- In this extension section we are introducing all known service bus extensions. User can remove the ones they don't need. -->
            <behaviorExtensions>
                <add name="connectionStatusBehavior"
                    type="Microsoft.ServiceBus.Configuration.ConnectionStatusElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="transportClientEndpointBehavior"
                    type="Microsoft.ServiceBus.Configuration.TransportClientEndpointBehaviorElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="serviceRegistrySettings"
                    type="Microsoft.ServiceBus.Configuration.ServiceRegistrySettingsElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
            </behaviorExtensions>
            <bindingElementExtensions>
                <add name="netMessagingTransport"
                    type="Microsoft.ServiceBus.Messaging.Configuration.NetMessagingTransportExtensionElement, Microsoft.ServiceBus,  Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="tcpRelayTransport"
                    type="Microsoft.ServiceBus.Configuration.TcpRelayTransportElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="httpRelayTransport"
                    type="Microsoft.ServiceBus.Configuration.HttpRelayTransportElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="httpsRelayTransport"
                    type="Microsoft.ServiceBus.Configuration.HttpsRelayTransportElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="onewayRelayTransport"
                    type="Microsoft.ServiceBus.Configuration.RelayedOnewayTransportElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
            </bindingElementExtensions>
            <bindingExtensions>
                <add name="basicHttpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.BasicHttpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="webHttpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.WebHttpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="ws2007HttpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.WS2007HttpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="netTcpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.NetTcpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="netOnewayRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.NetOnewayRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="netEventRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.NetEventRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="netMessagingBinding"
                    type="Microsoft.ServiceBus.Messaging.Configuration.NetMessagingBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
            </bindingExtensions>
        </extensions>
      <bindings>
        <!-- Application Binding -->
        <webHttpRelayBinding>
          <binding name="default">
            <security relayClientAuthenticationType="None" />
          </binding>
        </webHttpRelayBinding>
      </bindings>
      <services>
        <!-- Application Service -->
        <service name="Microsoft.ServiceBus.Samples.ImageService"
             behaviorConfiguration="default">
          <endpoint name="RelayEndpoint"
                  contract="Microsoft.ServiceBus.Samples.IImageContract"
                  binding="webHttpRelayBinding"
                  bindingConfiguration="default"
                  behaviorConfiguration="sbTokenProvider"
                  address="" />
        </service>
      </services>
      <behaviors>
        <endpointBehaviors>
          <behavior name="sbTokenProvider">
            <transportClientEndpointBehavior>
              <tokenProvider>
                <sharedAccessSignature keyName="RootManageSharedAccessKey" key="YOUR_SAS_KEY" />
              </tokenProvider>
            </transportClientEndpointBehavior>
          </behavior>
        </endpointBehaviors>
        <serviceBehaviors>
          <behavior name="default">
            <serviceDebug httpHelpPageEnabled="false" httpsHelpPageEnabled="false" />
          </behavior>
        </serviceBehaviors>
      </behaviors>
    </system.serviceModel>
    <appSettings>
        <!-- Service Bus specific app settings for messaging connections -->
        <add key="Microsoft.ServiceBus.ConnectionString"
            value="Endpoint=sb://yourNamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=YOUR_SAS_KEY>"/>
    </appSettings>
</configuration>
```

## <a name="host-the-rest-based-wcf-service-to-use-azure-relay"></a>Hospedaje del servicio WCF basado en REST para usar Azure Relay

Esta sección describe cómo ejecutar un servicio web mediante una aplicación de consola con WCF Relay. En el ejemplo que sigue al procedimiento aparece una lista completa del código escrito en esta sección.

### <a name="to-create-a-base-address-for-the-service"></a>Para crear una dirección base para el servicio

1. En la declaración de la función `Main()`, cree una variable para almacenar el espacio de nombres de su proyecto. Asegúrese de reemplazar `yourNamespace` por el nombre del espacio de nombres de Relay que creó antes.

    ```csharp
    string serviceNamespace = "yourNamespace";
    ```

    Service Bus utiliza el nombre del espacio de nombres para crear un URI único.

1. Cree una instancia `Uri` para la dirección base del servicio que se basa en el espacio de nombres.

    ```csharp
    Uri address = ServiceBusEnvironment.CreateServiceUri("https", serviceNamespace, "Image");
    ```

### <a name="to-create-and-configure-the-web-service-host"></a>Para crear y configurar el host del servicio web

Todavía en `Main()`, cree el host del servicio web, utilizando la dirección URI creada anteriormente en esta sección.
  
```csharp
WebServiceHost host = new WebServiceHost(typeof(ImageService), address);
```

El host de servicio es el objeto de WCF que crea una instancia de la aplicación host. En este ejemplo se pasa el tipo de host que desea crear (que es un `ImageService`) y también la dirección en la que desea exponer la aplicación host.

### <a name="to-run-the-web-service-host"></a>Para ejecutar el host del servicio web

1. Todavía en `Main()`, agregue la siguiente línea para abrir el servicio.

    ```csharp
    host.Open();
    ```

    El servicio está ahora en ejecución.

1. Muestre un mensaje que indica que el servicio se está ejecutando y cómo detenerlo.

    ```csharp
    Console.WriteLine("Copy the following address into a browser to see the image: ");
    Console.WriteLine(address + "GetImage");
    Console.WriteLine();
    Console.WriteLine("Press [Enter] to exit");
    Console.ReadLine();
    ```

1. Cuando termine, cierre el host del servicio.

    ```csharp
    host.Close();
    ```

### <a name="example-of-the-service-contract-and-implementation"></a>Ejemplo del contrato de servicio y de la implementación

En el ejemplo siguiente se incluye el contrato de servicio y la implementación de los pasos anteriores en el tutorial y se hospeda el servicio en una aplicación de consola. Compile el código siguiente en un archivo ejecutable denominado *ImageListener.exe*.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.ServiceModel;
using System.ServiceModel.Channels;
using System.ServiceModel.Web;
using System.IO;
using System.Drawing;
using System.Drawing.Imaging;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Web;

namespace Microsoft.ServiceBus.Samples
{

    [ServiceContract(Name = "ImageContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IImageContract
    {
        [OperationContract, WebGet]
        Stream GetImage();
    }

    public interface IImageChannel : IImageContract, IClientChannel { }

    [ServiceBehavior(Name = "ImageService", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    class ImageService : IImageContract
    {
        const string imageFileName = "image.jpg";

        Image bitmap;

        public ImageService()
        {
            this.bitmap = Image.FromFile(imageFileName);
        }

        public Stream GetImage()
        {
            MemoryStream stream = new MemoryStream();
            this.bitmap.Save(stream, ImageFormat.Jpeg);

            stream.Position = 0;
            WebOperationContext.Current.OutgoingResponse.ContentType = "image/jpeg";

            return stream;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            string serviceNamespace = "InsertServiceNamespaceHere";
            Uri address = ServiceBusEnvironment.CreateServiceUri("https", serviceNamespace, "Image");

            WebServiceHost host = new WebServiceHost(typeof(ImageService), address);
            host.Open();

            Console.WriteLine("Copy the following address into a browser to see the image: ");
            Console.WriteLine(address + "GetImage");
            Console.WriteLine();
            Console.WriteLine("Press [Enter] to exit");
            Console.ReadLine();

            host.Close();
        }
    }
}
```

## <a name="run-and-test-the-service"></a>Ejecución y prueba del servicio

Después de compilar la solución, haga lo siguiente para ejecutar la aplicación:

1. Seleccione F5 o busque la ubicación del archivo ejecutable, *ImageListener\bin\Debug\ImageListener.exe*, para ejecutar el servicio. Mantenga la aplicación en ejecución, ya que se necesita para el paso siguiente.
1. Copie y pegue la dirección desde el símbolo del sistema en un explorador para ver la imagen.
1. Cuando haya terminado, seleccione ENTRAR en la ventana del símbolo del sistema para cerrar la aplicación.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha creado una aplicación que utiliza el servicio Azure Relay, consulte los artículos siguientes para obtener más información:

* [¿Qué es Relay de Azure?](relay-what-is-it.md)
* [Exposición de un servicio WCF REST local a un cliente externo mediante el uso de Azure WCF Relay](service-bus-relay-tutorial.md)

[Azure portal]: https://portal.azure.com
