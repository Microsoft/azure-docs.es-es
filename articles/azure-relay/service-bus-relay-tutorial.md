---
title: Exposición de un servicio WCF REST local a los clientes mediante Azure Relay
description: En este tutorial se describe la exposición de un servicio WCF REST local a un cliente externo mediante el uso de Azure WCF Relay.
ms.topic: tutorial
ms.custom: devx-track-dotnet
ms.date: 06/23/2020
ms.openlocfilehash: bb2b9b5ed7c263762cc24b8eb2e6d66215147c4c
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935711"
---
# <a name="tutorial-expose-an-on-premises-wcf-rest-service-to-external-client-by-using-azure-wcf-relay"></a>Tutorial: Exposición de un servicio WCF REST local a un cliente externo mediante el uso de Azure WCF Relay

En este tutorial se describe cómo crear un servicio y una aplicación cliente de WCF Relay mediante Azure Relay. Para ver un tutorial parecido donde se usa [Mensajería de Service Bus](../service-bus-messaging/service-bus-messaging-overview.md), consulte [Introducción a las colas de Service Bus](../service-bus-messaging/service-bus-dotnet-get-started-with-queues.md).

El trabajo con este tutorial le proporciona unos conocimientos sobre los pasos para crear una aplicación cliente y de servicio de WCF Relay. Como sus homólogos WCF originales, un servicio es una construcción que expone uno o varios puntos de conexión. Cada uno de ellos expone una o varias operaciones de servicio. El extremo de un servicio especifica una dirección donde se puede encontrar el servicio, un enlace que contiene la información que un cliente debe comunicar al servicio y un contrato que define la funcionalidad que ofrece el servicio a sus clientes. La diferencia principal entre WCF y WCF Relay es que el punto de conexión se expone en la nube en lugar de localmente en su equipo.

Una vez que complete la secuencia de secciones de este tutorial, dispondrá de un servicio en ejecución. También tendrá un cliente que puede invocar las operaciones del servicio. 

En este tutorial, hará lo siguiente:

> [!div class="checklist"]
>
> * Instalar los requisitos previos del tutorial.
> * Creación de un espacio de nombres de Relay
> * Crear un contrato de servicio WCF.
> * Implementar el contrato de WCF.
> * Hospedar y ejecutar un servicio WCF para registrarse con el servicio Relay.
> * Crear un cliente WCF para el contrato de servicio.
> * Configurar el cliente de WCF.
> * Implementar el cliente de WCF.
> * Ejecución de las aplicaciones

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, debe cumplir los siguientes requisitos previos:

* Suscripción a Azure. Si no tiene una, [cree una cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* [Visual Studio 2015 o posterior](https://www.visualstudio.com). En los ejemplos de este tutorial se usa Visual Studio 2019.
* SDK de Azure para .NET. Instálelo desde la [página de descargas de SDK](https://azure.microsoft.com/downloads/).

## <a name="create-a-relay-namespace"></a>Creación de un espacio de nombres de Relay

El primer paso consiste en crear un espacio de nombres y obtener una clave de [firma de acceso compartido (SAS)](../service-bus-messaging/service-bus-sas.md). Un espacio de nombres proporciona un límite de aplicación para cada aplicación que se expone a través del servicio de retransmisión. El sistema genera una clave SAS automáticamente cuando se crea un espacio de nombres de servicio. La combinación del espacio de nombres de servicio y la clave SAS proporciona las credenciales para que Azure autentique el acceso a una aplicación.

[!INCLUDE [relay-create-namespace-portal](../../includes/relay-create-namespace-portal.md)]

## <a name="define-a-wcf-service-contract"></a>Definición de un contrato de servicio de WCF

El contrato de servicio especifica qué operaciones admite este. Las operaciones son métodos o funciones de servicio web. Los contratos se crean mediante la definición de una interfaz de C++, C# o Visual Basic. Cada método de la interfaz corresponde a una operación de servicio específica. Todas las interfaces deben tener el atributo [ServiceContractAttribute](/dotnet/api/system.servicemodel.servicecontractattribute) aplicado, y todas las operaciones deben tener el atributo [OperationContractAttribute](/dotnet/api/system.servicemodel.operationcontractattribute) aplicado. Si un método en una interfaz que tiene el atributo [ServiceContractAttribute](/dotnet/api/system.servicemodel.servicecontractattribute) no tiene el atributo [OperationContractAttribute](/dotnet/api/system.servicemodel.operationcontractattribute), no se expone ese método. El código para estas tareas se ofrece en el ejemplo a continuación del procedimiento. Para leer una descripción completa de los contratos y servicios, consulte [Diseño e implementación de servicios](/dotnet/framework/wcf/designing-and-implementing-services).

### <a name="create-a-relay-contract-with-an-interface"></a>Creación de un contrato de retransmisión con una interfaz

1. Inicie Microsoft Visual Studio como administrador. Para hacerlo, haga clic con el botón derecho en el icono del programa Visual Studio y, después, seleccione **Ejecutar como administrador**.
1. En Visual Studio, seleccione **Crear un proyecto nuevo**.
1. En **Crear un proyecto**, elija **Aplicación de consola (.NET Framework)** para C# y seleccione **Siguiente**.
1. Asigne al proyecto el nombre *EchoService* y seleccione **Crear**.

   ![Creación de una aplicación de consola][2]

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Administrar paquetes NuGet**. En el **Administrador de paquetes NuGet**, seleccione **Examinar** y, después, busque y elija **WindowsAzure.ServiceBus**. Seleccione **Instalar** y acepte las condiciones de uso.

    ![Paquete de Service Bus][3]

   Este paquete agrega automáticamente referencias a las bibliotecas de Service Bus y a `System.ServiceModel` de WCF. [System.ServiceModel](/dotnet/api/system.servicemodel) es el espacio de nombres que permite el acceso mediante programación a las características básicas de WCF. Service Bus utiliza muchos de los objetos y atributos de WCF para definir contratos de servicio.

1. Agregue las siguientes instrucciones `using` al principio de *Program.cs*:

    ```csharp
    using System.ServiceModel;
    using Microsoft.ServiceBus;
    ```

1. Cambie el nombre del espacio de nombres de su nombre predeterminado de `EchoService` a `Microsoft.ServiceBus.Samples`.

   > [!IMPORTANT]
   > En este tutorial se usa el espacio de nombres de C# `Microsoft.ServiceBus.Samples`, que es el espacio de nombres del tipo administrado por contrato que se utiliza en el archivo de configuración en la sección [Configurar el cliente de WCF](#configure-the-wcf-client). Puede especificar cualquier espacio de nombres que desee al compilar este ejemplo. No obstante, el tutorial no funcionará a menos que también modifique los espacios de nombres del contrato y el servicio de manera acorde, en el archivo de configuración de la aplicación. El espacio de nombres especificado en el archivo *App.config* debe ser el mismo que el que se especifique en los archivos de C#.
   >

1. Inmediatamente después de la declaración del espacio de nombres `Microsoft.ServiceBus.Samples`, pero dentro del espacio de nombres, defina una nueva interfaz denominada `IEchoContract` y aplique el atributo `ServiceContractAttribute` a la interfaz con un valor de espacio de nombres de `https://samples.microsoft.com/ServiceModel/Relay/`. Pegue el siguiente fragmento de código después de la declaración de espacios de nombres:

    ```csharp
    [ServiceContract(Name = "IEchoContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
    }
    ```

    El valor del espacio de nombres difiere del espacio de nombres que utiliza en todo el ámbito de su código. En su lugar, el valor del espacio de nombres se usa como identificador único para este contrato. La especificación del espacio de nombres impide explícitamente que el valor del espacio de nombres predeterminado se agregue al nombre del contrato.

   > [!NOTE]
   > Normalmente, el espacio de nombres del contrato de servicio contiene un esquema de nomenclatura que incluye información de versión. Al incluirse la información de versión en el espacio de nombres del contrato de servicio, los servicios pueden aislar los cambios más importantes mediante la definición de un nuevo contrato de servicio con un nuevo espacio de nombres y su exposición en un nuevo extremo. De esta manera, los clientes pueden seguir usando el contrato de servicio anterior sin tener que actualizarse. La información de versión puede constar de una fecha o un número de compilación. Para obtener más información, vea [Control de versiones del servicio](/dotnet/framework/wcf/service-versioning). Para este tutorial, el esquema de nomenclatura del espacio de nombres del contrato de servicio no contiene información de versión.
   >

1. En la interfaz de `IEchoContract`, declare un método para la operación sencilla que el contrato `IEchoContract` expone en la interfaz y aplique el atributo `OperationContractAttribute` al método que quiere exponer como parte del contrato público de Relay WCF de la siguiente forma:

    ```csharp
    [OperationContract]
    string Echo(string text);
    ```

1. Justo después de la definición de interfaz `IEchoContract`, declare un canal que herede de `IEchoContract` y también de la interfaz de `IClientChannel`, como se muestra a continuación:

    ```csharp
    public interface IEchoChannel : IEchoContract, IClientChannel { }
    ```

    Un canal es el objeto de WCF a través del cual el host y el cliente se pasan información entre sí. Más tarde, escribirá código para el canal a fin de enviar información entre las dos aplicaciones.

1. Seleccione **Compilar** > **Compilar solución** o seleccione Ctrl+Mayús+B para confirmar la precisión del trabajo realizado.

### <a name="example-of-a-wcf-contract"></a>Ejemplo de un contrato de WCF

En el ejemplo de código siguiente se muestra una interfaz básica que define un contrato de WCF Relay.

```csharp
using System;
using System.ServiceModel;

namespace Microsoft.ServiceBus.Samples
{
    [ServiceContract(Name = "IEchoContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
        [OperationContract]
        String Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { }

    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

Ahora que la interfaz se ha creado, puede implementarla.

## <a name="implement-the-wcf-contract"></a>Implementación del contrato de WCF

La creación de una instancia de Azure Relay requiere que primero cree el contrato mediante una interfaz. Para más información sobre cómo crear la interfaz, consulte la sección anterior. El procedimiento siguiente implementa la interfaz. Esta tarea conlleva la creación de una clase denominada `EchoService` que implementa la interfaz `IEchoContract` definida por el usuario. Después de implementar la interfaz, se debe configurar con un archivo *App.config*. El archivo de configuración contiene la información necesaria para la aplicación. Esta información incluye el nombre del servicio, el nombre del contrato y el tipo de protocolo que se utiliza para comunicarse con el servicio de retransmisión. El código utilizado para estas tareas se proporciona en el ejemplo que sigue al procedimiento. Para obtener información más general sobre cómo implementar un contrato de servicio, consulte [Implementación de contratos de servicio](/dotnet/framework/wcf/implementing-service-contracts).

1. Cree una nueva clase denominada `EchoService` directamente después de la definición de la interfaz `IEchoContract`. La clase `EchoService` implementa la interfaz `IEchoContract`.

    ```csharp
    class EchoService : IEchoContract
    {
    }
    ```

    Al igual que otras implementaciones de interfaz, puede implementar la definición en un archivo diferente. Sin embargo, para este tutorial, la implementación se ubica en el mismo archivo que la definición de la interfaz y el método `Main()`.

1. Aplique el atributo [ServiceBehaviorAttribute](/dotnet/api/system.servicemodel.servicebehaviorattribute) en la interfaz de `IEchoContract`. El atributo especifica el nombre del servicio y el espacio de nombres. Después de hacerlo, aparecerá la clase `EchoService` de la siguiente manera:

    ```csharp
    [ServiceBehavior(Name = "EchoService", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    class EchoService : IEchoContract
    {
    }
    ```

1. Implemente el método `Echo` definido en la interfaz `IEchoContract` de la clase `EchoService`.

    ```csharp
    public string Echo(string text)
    {
        Console.WriteLine("Echoing: {0}", text);
        return text;
    }
    ```

1. Seleccione **Compilar** > **Compilar solución** o seleccione Ctrl+Mayús+B.

### <a name="define-the-configuration-for-the-service-host"></a>Definición de la configuración del host de servicios

El archivo de configuración es similar a un archivo de configuración de WCF. Incluye el nombre del servicio, el punto de conexión y el enlace. El punto de conexión es la ubicación que Azure Relay expone para que los clientes y hosts se comuniquen entre sí. El enlace es el tipo de protocolo que se utiliza para la comunicación. La principal diferencia es que el punto de conexión de servicio configurado hace referencia a un enlace [NetTcpRelayBinding](/dotnet/api/microsoft.servicebus.nettcprelaybinding) que no forma parte de .NET Framework. [NetTcpRelayBinding](/dotnet/api/microsoft.servicebus.nettcprelaybinding) es uno de los enlaces que define el servicio.

1. En el **Explorador de soluciones**, haga doble clic en **App.config** para abrir el archivo en el editor de Visual Studio.
1. En el elemento `<appSettings>`, reemplace los marcadores de posición con el nombre de su espacio de nombres de servicio y la clave SAS que copió en el paso anterior.
1. Dentro de las etiquetas `<system.serviceModel>`, agregue un elemento `<services>`. Puede definir varias aplicaciones de retransmisión en un único archivo de configuración. Sin embargo, este tutorial solo define una.

    ```xml
    <?xmlversion="1.0"encoding="utf-8"?>
    <configuration>
      <system.serviceModel>
        <services>

        </services>
      </system.serviceModel>
    </configuration>
    ```

1. Dentro del elemento `<services>`, agregue un elemento `<service>` para definir el nombre del servicio.

    ```xml
    <service name="Microsoft.ServiceBus.Samples.EchoService">
    </service>
    ```

1. Dentro del elemento `<service>`, defina la ubicación del contrato del punto de conexión y también el tipo de enlace del extremo.

    ```xml
    <endpoint contract="Microsoft.ServiceBus.Samples.IEchoContract" binding="netTcpRelayBinding"/>
    ```

    El extremo define dónde buscará el cliente la aplicación host. Más adelante en el tutorial, se usa este paso para crear un identificador URI que expone completamente el host a través de Azure Relay. El enlace declara que se está usando TCP como el protocolo para comunicarse con el servicio de retransmisión.

1. Seleccione **Compilar** > **Compilar solución** o seleccione Ctrl+Mayús+B para confirmar la precisión del trabajo realizado.

### <a name="example-of-implementation-of-a-service-contract"></a>Ejemplo de implementación de un contrato de servicio

En el código siguiente se muestra la implementación del contrato de servicio.

```csharp
[ServiceBehavior(Name = "EchoService", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]

    class EchoService : IEchoContract
    {
        public string Echo(string text)
        {
            Console.WriteLine("Echoing: {0}", text);
            return text;
        }
    }
```

En el código siguiente se muestra el formato básico del archivo *App.config* asociado al servicio.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <system.serviceModel>
    <services>
      <service name="Microsoft.ServiceBus.Samples.EchoService">
        <endpoint contract="Microsoft.ServiceBus.Samples.IEchoContract" binding="netTcpRelayBinding" />
      </service>
    </services>
    <extensions>
      <bindingExtensions>
        <add name="netTcpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.NetTcpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
      </bindingExtensions>
    </extensions>
  </system.serviceModel>
</configuration>
```

## <a name="host-and-run-the-wcf-service-to-register-with-the-relay-service"></a>Hospedaje y ejecución de un servicio WCF para registrarse con el servicio Relay

Este paso describe cómo ejecutar un servicio de Azure Relay.

### <a name="create-the-relay-credentials"></a>Creación de las credenciales de retransmisión

1. En `Main()`, cree dos variables para almacenar el espacio de nombres y la clave de SAS que se leen desde la ventana de la consola.

    ```csharp
    Console.Write("Your Service Namespace: ");
    string serviceNamespace = Console.ReadLine();
    Console.Write("Your SAS key: ");
    string sasKey = Console.ReadLine();
    ```

    La clave de SAS se usará más adelante para obtener acceso al proyecto. El espacio de nombres se pasa como parámetro a `CreateServiceUri` para crear un URI de servicio.

1. Con un objeto [TransportClientEndpointBehavior](/dotnet/api/microsoft.servicebus.transportclientendpointbehavior), declare que usará una clave de SAS como tipo de credencial. Agregue el código siguiente directamente después del código de que se agregó en el último paso.

    ```csharp
    TransportClientEndpointBehavior sasCredential = new TransportClientEndpointBehavior();
    sasCredential.TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider("RootManageSharedAccessKey", sasKey);
    ```

### <a name="create-a-base-address-for-the-service"></a>Creación de una dirección base para el servicio

Siguiendo el código que agregó en la sección anterior, cree una instancia de `Uri` para la dirección base del servicio. Este identificador URI especifica el esquema de Service Bus, el espacio de nombres y la ruta de acceso de la interfaz del servicio.

```csharp
Uri address = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, "EchoService");
```

El valor "SB" es una abreviatura del esquema de Service Bus. Indica que se está usando TCP como protocolo. Este esquema también se indicó anteriormente en el archivo de configuración, cuando se especificó [NetTcpRelayBinding](/dotnet/api/microsoft.servicebus.nettcprelaybinding) como el enlace.

Para este tutorial, el identificador URI es `sb://putServiceNamespaceHere.windows.net/EchoService`.

### <a name="create-and-configure-the-service-host"></a>Creación y configuración del host de servicios

1. Cuando aún esté trabajando en `Main()`, establezca el modo de conectividad en `AutoDetect`.

    ```csharp
    ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.AutoDetect;
    ```

    El modo de conectividad describe el protocolo que usa el servicio para comunicarse con el servicio de retransmisión; HTTP o TCP. Con la configuración predeterminada `AutoDetect`, el servicio intenta conectarse a Azure Relay a través de TCP si está disponible, y HTTP si no lo está. Este resultado difiere del protocolo que especifica el servicio para la comunicación del cliente. Dicho protocolo viene determinado por el enlace utilizado. Por ejemplo, un servicio puede usar el enlace [BasicHttpRelayBinding](/dotnet/api/microsoft.servicebus.basichttprelaybinding), que especifica que su punto de conexión se comunica con los clientes a través de HTTP. Ese mismo servicio podría especificar `ConnectivityMode.AutoDetect` para que el servicio se comunique con Azure Relay mediante TCP.

1. Cree el host del servicio, con el identificador URI creado anteriormente en esta sección.

    ```csharp
    ServiceHost host = new ServiceHost(typeof(EchoService), address);
    ```

    El host del servicio es el objeto de WCF que crea una instancia del servicio. En este caso, se pasa el tipo de servicio que quiere crear (un tipo `EchoService`) y también la dirección en la que quiere exponer el servicio.

1. En la parte superior del archivo *Program.cs*, agregue referencias a [System.ServiceModel.Description](/dotnet/api/system.servicemodel.description) y [Microsoft.ServiceBus.Description](/dotnet/api/microsoft.servicebus.description).

    ```csharp
    using System.ServiceModel.Description;
    using Microsoft.ServiceBus.Description;
    ```

1. De nuevo en `Main()`, configure el punto de conexión para permitir el acceso público.

    ```csharp
    IEndpointBehavior serviceRegistrySettings = new ServiceRegistrySettings(DiscoveryType.Public);
    ```

    En este paso se informa al servicio de retransmisión de que la aplicación se puede encontrar públicamente al examinar la fuente Atom del proyecto. Si establece `DiscoveryType` en `private`, un cliente podría seguir accediendo al servicio. Sin embargo, el servicio no aparecería cuando buscase el espacio de nombres de `Relay`. En su lugar, el cliente tendría que conocer de antemano la ruta de acceso del extremo.

1. Aplique las credenciales de servicio a los puntos de conexión de servicio definidos en el archivo *App.config*:

    ```csharp
    foreach (ServiceEndpoint endpoint in host.Description.Endpoints)
    {
        endpoint.Behaviors.Add(serviceRegistrySettings);
        endpoint.Behaviors.Add(sasCredential);
    }
    ```

    Como se indicó en el paso anterior, podría haber declarado varios servicios y puntos de conexión en el archivo de configuración. Si lo hubiera hecho, este código recorrería el archivo de configuración y buscaría todos los extremos a los que se deban aplicar las credenciales. Para este tutorial, el archivo de configuración tiene un solo punto de conexión.

### <a name="open-the-service-host"></a>Apertura del host de servicios

1. Todavía en `Main()`, agregue la siguiente línea para abrir el servicio.

    ```csharp
    host.Open();
    ```

1. Informe al usuario de que se está ejecutando el servicio y explíquele cómo cerrarlo.

    ```csharp
    Console.WriteLine("Service address: " + address);
    Console.WriteLine("Press [Enter] to exit");
    Console.ReadLine();
    ```

1. Cuando termine, cierre el host del servicio.

    ```csharp
    host.Close();
    ```

1. Seleccione Ctrl+Mayús+B para compilar el proyecto.

### <a name="example-that-hosts-a-service-in-a-console-application"></a>Ejemplo que hospeda un servicio en una aplicación de consola

El código de servicio completado debería tener el siguiente formato. En el código se incluye el contrato de servicio y la implementación de los pasos anteriores del tutorial. Además, hospeda el servicio en una aplicación de consola.

```csharp
using System;
using System.ServiceModel;
using System.ServiceModel.Description;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Description;

namespace Microsoft.ServiceBus.Samples
{
    [ServiceContract(Name = "IEchoContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
        [OperationContract]
        String Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { };

    [ServiceBehavior(Name = "EchoService", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    class EchoService : IEchoContract
    {
        public string Echo(string text)
        {
            Console.WriteLine("Echoing: {0}", text);
            return text;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.AutoDetect;         

            Console.Write("Your Service Namespace: ");
            string serviceNamespace = Console.ReadLine();
            Console.Write("Your SAS key: ");
            string sasKey = Console.ReadLine();

           // Create the credentials object for the endpoint.
            TransportClientEndpointBehavior sasCredential = new TransportClientEndpointBehavior();
            sasCredential.TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider("RootManageSharedAccessKey", sasKey);

            // Create the service URI based on the service namespace.
            Uri address = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, "EchoService");

            // Create the service host reading the configuration.
            ServiceHost host = new ServiceHost(typeof(EchoService), address);

            // Create the ServiceRegistrySettings behavior for the endpoint.
            IEndpointBehavior serviceRegistrySettings = new ServiceRegistrySettings(DiscoveryType.Public);

            // Add the Relay credentials to all endpoints specified in configuration.
            foreach (ServiceEndpoint endpoint in host.Description.Endpoints)
            {
                endpoint.Behaviors.Add(serviceRegistrySettings);
                endpoint.Behaviors.Add(sasCredential);
            }

            // Open the service.
            host.Open();

            Console.WriteLine("Service address: " + address);
            Console.WriteLine("Press [Enter] to exit");
            Console.ReadLine();

            // Close the service.
            host.Close();
        }
    }
}
```

## <a name="create-a-wcf-client-for-the-service-contract"></a>Crear un cliente WCF para el contrato de servicio

La siguiente tarea consiste en crear una aplicación cliente y definir el contrato de servicio que se implementará más adelante. Estos pasos son similares a los que se usaron para crear un servicio: definir un contrato, editar un archivo *App.config*, usar credenciales para conectarse al servicio de retransmisión, etc. El código utilizado para estas tareas se proporciona en el ejemplo que sigue al procedimiento.

1. Cree un nuevo proyecto en la solución actual de Visual Studio para el cliente:

   1. En el **Explorador de soluciones**, haga clic con el botón derecho en la solución actual (no en el proyecto) y seleccione **Agregar** > **Nuevo proyecto**.
   1. En **Agregar un nuevo proyecto**, seleccione **Aplicación de consola (.NET Framework)** para C# y seleccione **Siguiente**.
   1. Asigne al proyecto el nombre de *EchoClient* y seleccione **Crear**.

1. En el **Explorador de soluciones**, en el proyecto **EchoClient**, haga doble clic en el archivo **Program.cs** para abrirlo en el editor en caso de que no esté ya abierto.
1. Cambie el nombre del espacio de nombres de su nombre predeterminado de `EchoClient` a `Microsoft.ServiceBus.Samples`.
1. Instale el [paquete NuGet de Service Bus](https://www.nuget.org/packages/WindowsAzure.ServiceBus):

   1. En el **Explorador de soluciones**, haga clic con el botón derecho en **EchoClient** y, después, seleccione **Administrar paquetes NuGet**.
   1. Seleccione **Examinar** y, después, busque y seleccione **WindowsAzure.ServiceBus**. Seleccione **Instalar** y acepte las condiciones de uso.

      ![Instalar el paquete de Service Bus][4]

1. Agregue una instrucción `using` para el espacio de nombres [System.ServiceModel](/dotnet/api/system.servicemodel) en el archivo *Program.cs*.

    ```csharp
    using System.ServiceModel;
    ```

1. Agregue la definición del contrato de servicio al espacio de nombres, como se muestra en el ejemplo siguiente. Esta definición es idéntica a la definición que se usa en el proyecto **Service**. Agregue este código en la parte superior del espacio de nombres `Microsoft.ServiceBus.Samples`.

    ```csharp
    [ServiceContract(Name = "IEchoContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
        [OperationContract]
        string Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { }
    ```

1. Seleccione Ctrl+Mayús+B para compilar el cliente.

### <a name="example-of-the-echoclient-project"></a>Ejemplo del proyecto EchoClient

En el código siguiente se muestra el estado actual del archivo *Program.cs* en el proyecto **EchoClient**.

```csharp
using System;
using Microsoft.ServiceBus;
using System.ServiceModel;

namespace Microsoft.ServiceBus.Samples
{

    [ServiceContract(Name = "IEchoContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
        [OperationContract]
        string Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { }


    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

## <a name="configure-the-wcf-client"></a>Configurar el cliente de WCF

En este paso, creará un archivo *App.config* para una aplicación cliente básica que accede al servicio creado anteriormente en este tutorial. Este archivo *App.config* define el contrato, el enlace y el nombre del punto de conexión. El código utilizado para estas tareas se proporciona en el ejemplo que sigue al procedimiento.

1. En el **Explorador de soluciones**, en el proyecto **EchoClient**, haga doble clic en **App.config** para abrir el archivo en el editor de Visual Studio.
1. En el elemento `<appSettings>`, reemplace los marcadores de posición con el nombre de su espacio de nombres de servicio y la clave SAS que copió en el paso anterior.
1. En el elemento `system.serviceModel`, agregue un elemento `<client>`.

    ```xml
    <?xmlversion="1.0"encoding="utf-8"?>
    <configuration>
      <system.serviceModel>
        <client>
        </client>
      </system.serviceModel>
    </configuration>
    ```

    En este código se declara que está definiendo una aplicación cliente de estilo WCF.

1. Dentro del elemento `client`, defina el nombre, el contrato y el tipo de enlace para el punto de conexión.

    ```xml
    <endpoint name="RelayEndpoint"
                    contract="Microsoft.ServiceBus.Samples.IEchoContract"
                    binding="netTcpRelayBinding"/>
    ```

    Este código define el nombre del punto de conexión. En este paso se define el contrato definido en el servicio y el hecho de que la aplicación cliente use TCP para comunicarse con Azure Relay. El nombre del extremo se usa en el paso siguiente para vincular la configuración de este extremo con el identificador URI del servicio.

1. Seleccione **Archivo** > **Guardar todo**.

### <a name="example-of-the-appconfig-file"></a>Ejemplo del archivo App.config

En el código siguiente se muestra el archivo *App.config* del cliente de eco.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <system.serviceModel>
    <client>
      <endpoint name="RelayEndpoint"
                      contract="Microsoft.ServiceBus.Samples.IEchoContract"
                      binding="netTcpRelayBinding"/>
    </client>
    <extensions>
      <bindingExtensions>
        <add name="netTcpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.NetTcpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
      </bindingExtensions>
    </extensions>
  </system.serviceModel>
</configuration>
```

## <a name="implement-the-wcf-client"></a>Implementación del cliente de WCF

En esta sección, implementará una aplicación cliente básica que accede al servicio que creó anteriormente en este tutorial. De forma parecida al servicio, el cliente realiza muchas de las mismas operaciones para acceder a Azure Relay:

* Establece el modo de conectividad.
* Crea el identificador URI que localiza el servicio de host.
* Define las credenciales de seguridad.
* Aplica a las credenciales para la conexión.
* Abre la conexión.
* Realiza las tareas específicas de la aplicación.
* Cierra la conexión.

Sin embargo, una de las principales diferencias es que la aplicación cliente usa un canal para conectarse al servicio de retransmisión. El servicio utiliza una llamada a **ServiceHost**. El código utilizado para estas tareas se proporciona en el ejemplo que sigue al procedimiento.

### <a name="implement-a-client-application"></a>Implementación de una aplicación cliente

1. Establezca el modo de conectividad a `AutoDetect`. Agregue el código siguiente dentro del método `Main()` de la aplicación **EchoClient**.

    ```csharp
    ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.AutoDetect;
    ```

1. Defina variables para contener los valores del espacio de nombres del servicio y la clave SAS que se leen desde la consola.

    ```csharp
    Console.Write("Your Service Namespace: ");
    string serviceNamespace = Console.ReadLine();
    Console.Write("Your SAS Key: ");
    string sasKey = Console.ReadLine();
    ```

1. Cree el identificador URI que define la ubicación del host en el proyecto de Relay.

    ```csharp
    Uri serviceUri = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, "EchoService");
    ```

1. Cree el objeto de credenciales para el extremo del espacio de nombres de servicio.

    ```csharp
    TransportClientEndpointBehavior sasCredential = new TransportClientEndpointBehavior();
    sasCredential.TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider("RootManageSharedAccessKey", sasKey);
    ```

1. Cree el generador de canales que carga la configuración que se describe en el archivo *App.config*.

    ```csharp
    ChannelFactory<IEchoChannel> channelFactory = new ChannelFactory<IEchoChannel>("RelayEndpoint", new EndpointAddress(serviceUri));
    ```

    Un generador de canales es un objeto de WCF que crea un canal a través del cual se comunican las aplicaciones cliente y de servicio.

1. Aplique las credenciales.

    ```csharp
    channelFactory.Endpoint.Behaviors.Add(sasCredential);
    ```

1. Cree y abra el canal al servicio.

    ```csharp
    IEchoChannel channel = channelFactory.CreateChannel();
    channel.Open();
    ```

1. Escriba la interfaz de usuario básica y la funcionalidad del eco.

    ```csharp
    Console.WriteLine("Enter text to echo (or [Enter] to exit):");
    string input = Console.ReadLine();
    while (input != String.Empty)
    {
        try
        {
            Console.WriteLine("Server echoed: {0}", channel.Echo(input));
        }
        catch (Exception e)
        {
            Console.WriteLine("Error: " + e.Message);
        }
        input = Console.ReadLine();
    }
    ```

    El código usa la instancia del objeto de canal como un proxy para el servicio.

1. Cierre el canal y el generador.

    ```csharp
    channel.Close();
    channelFactory.Close();
    ```

### <a name="example-code-for-this-tutorial"></a>Código de ejemplo de este tutorial

El código completo debería tener el siguiente aspecto. En este código se muestra cómo crear una aplicación cliente, cómo llamar a las operaciones del servicio y cómo cerrar el cliente cuando finaliza la llamada a la operación.

```csharp
using System;
using Microsoft.ServiceBus;
using System.ServiceModel;

namespace Microsoft.ServiceBus.Samples
{
    [ServiceContract(Name = "IEchoContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
        [OperationContract]
        String Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { }

    class Program
    {
        static void Main(string[] args)
        {
            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.AutoDetect;


            Console.Write("Your Service Namespace: ");
            string serviceNamespace = Console.ReadLine();
            Console.Write("Your SAS Key: ");
            string sasKey = Console.ReadLine();



            Uri serviceUri = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, "EchoService");

            TransportClientEndpointBehavior sasCredential = new TransportClientEndpointBehavior();
            sasCredential.TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider("RootManageSharedAccessKey", sasKey);

            ChannelFactory<IEchoChannel> channelFactory = new ChannelFactory<IEchoChannel>("RelayEndpoint", new EndpointAddress(serviceUri));

            channelFactory.Endpoint.Behaviors.Add(sasCredential);

            IEchoChannel channel = channelFactory.CreateChannel();
            channel.Open();

            Console.WriteLine("Enter text to echo (or [Enter] to exit):");
            string input = Console.ReadLine();
            while (input != String.Empty)
            {
                try
                {
                    Console.WriteLine("Server echoed: {0}", channel.Echo(input));
                }
                catch (Exception e)
                {
                    Console.WriteLine("Error: " + e.Message);
                }
                input = Console.ReadLine();
            }

            channel.Close();
            channelFactory.Close();

        }
    }
}
```

## <a name="run-the-applications"></a>Ejecución de las aplicaciones

1. Seleccione Ctrl+Mayús+B para compilar la solución. Esta acción compila el proyecto de cliente y el proyecto de servicio que creó en los pasos anteriores.
1. Antes de ejecutar la aplicación cliente, debe asegurarse de que se está ejecutando la aplicación del servicio. En el **Explorador de soluciones**, haga clic con el botón derecho en la solución **EchoService** y, después, seleccione **Propiedades**.
1. En **Páginas de propiedades**, **Propiedades comunes** > **Proyecto de inicio**, elija **Proyectos de inicio múltiples**. Asegúrese de que **EchoService** aparece en primer lugar en la lista.
1. Defina el cuadro **Acción** para los proyectos **EchoService** y **EchoClient** en **Iniciar**.

    ![Páginas de propiedades del proyecto][5]

1. Seleccione **Dependencias del proyecto**. En **Proyectos**, seleccione **EchoClient**. En **Depende de:** , asegúrese de que **EchoService** está seleccionado.

    ![Dependencias del proyecto][6]

1. Seleccione **Aceptar** para cerrar **Páginas de propiedades**.
1. Seleccione F5 para ejecutar ambos proyectos.
1. Ambas ventanas de consola se abrirán y le solicitarán el espacio de nombres. Primero debe ejecutarse el servicio, por ello, en la ventana de la consola de **EchoService** escriba el espacio de nombres y después seleccione Entrar.
1. A continuación, la consola le pedirá la clave SAS. Escríbala y seleccione ENTRAR.

    Este un ejemplo del resultado de la ventana de consola. Los valores siguientes son solo ejemplos.

    `Your Service Namespace: myNamespace`

    `Your SAS Key: <SAS key value>`

    La aplicación del servicio imprime en la ventana de la consola la dirección en la que está escuchando, tal y como se muestra en el ejemplo siguiente.

    `Service address: sb://mynamespace.servicebus.windows.net/EchoService/`

    `Press [Enter] to exit`

1. En la ventana de la consola de **EchoClient** escriba la misma información que especificó anteriormente para la aplicación de servicio. Especifique los mismos valores de espacio de nombres de servicio y clave SAS para la aplicación cliente.
1. Tras especificar estos valores, el cliente abrirá un canal al servicio y se le solicitará que escriba algo de texto, como se muestra en el siguiente ejemplo de salida de consola.

    `Enter text to echo (or [Enter] to exit):`

    Escriba algo de texto para enviarlo a la aplicación de servicio y seleccione ENTRAR. Este texto se enviará al servicio a través de la operación de servicio de eco y aparecerá en la ventana de la consola del servicio, como se muestra en el siguiente ejemplo de salida.

    `Echoing: My sample text`

    La aplicación cliente recibe el valor devuelto de la operación `Echo`, que es el texto original, y lo imprime en la ventana de consola. El siguiente texto muestra un ejemplo de la salida de la ventana de consola del cliente.

    `Server echoed: My sample text`

1. Puede continuar enviando mensajes de texto desde el cliente al servicio de esta manera. Cuando haya terminado, seleccione ENTRAR en las ventanas de las consolas del cliente y el servicio para finalizar ambas aplicaciones.

## <a name="next-steps"></a>Pasos siguientes

Avance al siguiente tutorial:

> [!div class="nextstepaction"]
>[Exposición de un servicio WCF REST local en un cliente fuera de la red](service-bus-relay-rest-tutorial.md)

[2]: ./media/service-bus-relay-tutorial/configure-echoservice-console-app.png
[3]: ./media/service-bus-relay-tutorial/install-nuget-service-bus.png
[4]: ./media/service-bus-relay-tutorial/install-nuget-service-bus-client.png
[5]: ./media/service-bus-relay-tutorial/set-projects.png
[6]: ./media/service-bus-relay-tutorial/set-depend.png
