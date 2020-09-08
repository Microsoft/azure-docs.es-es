---
title: Creación de un servicio basado en actores en Azure Service Fabric
description: Aprenda a crear, depurar e implementar su primer servicio basado en actores en C# con Reliable Actors de Service Fabric.
author: vturecek
ms.topic: conceptual
ms.date: 07/10/2019
ms.author: vturecek
ms.custom: devx-track-csharp
ms.openlocfilehash: 3727c41c991b83bc3869279cbb0535a6185ff04d
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89015737"
---
# <a name="getting-started-with-reliable-actors"></a>Introducción a Reliable Actors
> [!div class="op_single_selector"]
> * [C# en Windows](service-fabric-reliable-actors-get-started.md)
> * [Java en Linux](./service-fabric-create-your-first-linux-application-with-java.md)

Este artículo le guía por la creación y depuración de una aplicación sencilla de Reliable Actors en Visual Studio. Para obtener más información sobre Reliable Actors, vea [Introducción a Reliable Actors de Service Fabric](service-fabric-reliable-actors-introduction.md).

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar, asegúrese de que el entorno de desarrollo de Service Fabric, incluido Visual Studio, está configurado en la máquina. Para obtener más información, consulte [cómo configurar el entorno de desarrollo](service-fabric-get-started.md).

## <a name="create-a-new-project-in-visual-studio"></a>Creación de un proyecto en Visual Studio

Inicie Visual Studio 2019 o versiones posteriores como administrador y cree un proyecto de **Aplicación de Service Fabric**:

![Herramientas de Service Fabric para Visual Studio: nuevo proyecto][1]

En el siguiente cuadro de diálogo, elija **Servicio de actor** en **.NET Core 2.0** y escriba un nombre para el servicio.

![Plantillas de proyecto de Service Fabric][5]

El proyecto creado muestra la estructura siguiente:

![Estructura de proyecto de Service Fabric][2]

## <a name="examine-the-solution"></a>Examinar la solución

La solución contiene tres proyectos:

* **El proyecto de aplicación (MyApplication)** . Este proyecto empaqueta todos los servicios a la vez para su implementación. Contiene los scripts de PowerShell y *ApplicationManifest.xml* para administrar la aplicación.

* **El proyecto de interfaz (HelloWorld.Interfaces)** . Este proyecto contiene la definición de la interfaz del actor. Pueden definirse interfaces de actor en cualquier proyecto con cualquier nombre.  La interfaz define el contrato de actor que comparten la implementación del actor y los clientes que llaman al actor.  Dado que los proyectos de cliente pueden depender de ella, suele tener sentido que se defina en un ensamblado independiente de la implementación de actor.

* **El proyecto de servicio de actor (HelloWorld)** . Este proyecto define el servicio de Service Fabric que va a hospedar al actor. Contiene la implementación del actor, *HelloWorld.cs*. La implementación del actor es una clase que deriva de un tipo base `Actor` e implementa las interfaces definidas en el proyecto *MyActor.Interfaces*. Una clase de actor también debe implementar un constructor que acepta una instancia `ActorService` y `ActorId` y los pasa a la clase `Actor` base.
    
    Este proyecto contiene también *Program.cs*, que registra las clases de actor con el runtime de Service Fabric mediante `ActorRuntime.RegisterActorAsync<T>()`. La clase `HelloWorld` ya está registrada. Las implementaciones de actor adicionales que se agreguen al proyecto deben estar registradas también en el método `Main()`.

## <a name="customize-the-helloworld-actor"></a>Personalizar el actor HelloWorld

La plantilla de proyecto define algunos métodos en la interfaz `IHelloWorld` y los implementa en la implementación de actor `HelloWorld`.  Reemplace esos métodos para que el servicio de actor devuelva una cadena "Hello World" simple.

En el archivo *IHelloWorld.cs* del proyecto *HelloWorld.Interfaces*, reemplace la definición de interfaz de la manera siguiente:

```csharp
public interface IHelloWorld : IActor
{
    Task<string> GetHelloWorldAsync();
}
```

En el archivo **HelloWorld.cs** del proyecto **HelloWorld**, reemplace la definición de la clase completa de la manera siguiente:

```csharp
[StatePersistence(StatePersistence.Persisted)]
internal class HelloWorld : Actor, IHelloWorld
{
    public HelloWorld(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task<string> GetHelloWorldAsync()
    {
        return Task.FromResult("Hello from my reliable actor!");
    }
}
```

Presione **Ctrl+Mayús+B** para generar el proyecto y asegurarse de que todo se compila.

## <a name="add-a-client"></a>Agregar un cliente

Cree una aplicación de consola simple para llamar al servicio de actor.

1. Haga clic con el botón derecho en el Explorador de soluciones > **Agregar** > **Nuevo proyecto...** .

2. En los tipos de proyectos de **.NET Core**, elija **Aplicación de consola (.NET Core)** .  Asigne al proyecto el nombre *ActorClient*.
    
    ![Cuadro de diálogo Agregar nuevo proyecto][6]    
    
    > [!NOTE]
    > Una aplicación de consola no es el tipo de aplicación que se utilizaría normalmente como cliente en Service Fabric, pero resulta un ejemplo práctico de depuración y pruebas con el clúster local de Service Fabric.

3. La aplicación de consola debe ser una aplicación de 64 bits para mantener la compatibilidad con el proyecto de interfaz y otras dependencias.  En el Explorador de soluciones, haga clic con el botón derecho en el proyecto **ActorClient** y luego haga clic en **Propiedades**.  En la pestaña **Compilar**, establezca la **Destino de la plataforma** en **x64**.
    
    ![Propiedades de la compilación][8]

4. El proyecto de cliente requiere el paquete NuGet de Reliable Actors.  Haga clic en **Herramientas** > **Administrador de paquetes NuGet** > **Consola del administrador de paquetes**.  En la Consola del Administrador de paquetes, escriba el siguiente comando:
    
    ```powershell
    Install-Package Microsoft.ServiceFabric.Actors -IncludePrerelease -ProjectName ActorClient
    ```

    El paquete NuGet y todas sus dependencias se instalan en el proyecto ActorClient.

5. El proyecto de cliente también requiere una referencia al proyecto de interfaces.  En el proyecto ActorClient, haga clic en **Dependencias** y luego en **Agregar referencia...** .  Seleccione **Proyectos > Solución** (si no está ya seleccionada) y luego marque la casilla situada junto a **HelloWorld.Interfaces**.  Haga clic en **OK**.
    
    ![Cuadro de diálogo Agregar referencia][7]

6. En el proyecto ActorClient, reemplace todo el contenido de *Program.cs* por el código siguiente:
    
    ```csharp
    using System;
    using System.Threading.Tasks;
    using Microsoft.ServiceFabric.Actors;
    using Microsoft.ServiceFabric.Actors.Client;
    using HelloWorld.Interfaces;
    
    namespace ActorClient
    {
        class Program
        {
            static void Main(string[] args)
            {
                IHelloWorld actor = ActorProxy.Create<IHelloWorld>(ActorId.CreateRandom(), new Uri("fabric:/MyApplication/HelloWorldActorService"));
                Task<string> retval = actor.GetHelloWorldAsync();
                Console.Write(retval.Result);
                Console.ReadLine();
            }
        }
    }
    ```

## <a name="running-and-debugging"></a>Ejecución y depuración

Presione **F5** para compilar, implementar y ejecutar la aplicación localmente en el clúster de desarrollo de Service Fabric.  Durante el proceso de implementación, puede ver el progreso en la ventana **Resultados** .

![Ventana de resultados de depuración de Service Fabric][3]

Cuando la salida incluye el texto *La aplicación está lista*, es posible probar el servicio con la aplicación ActorClient.  En el Explorador de soluciones, haga clic con el botón derecho en el proyecto **ActorClient** y luego haga clic en **Depurar** > **Iniciar nueva instancia**.  La aplicación de línea de comandos debe mostrar la salida del servicio de actor.

![Salida de la aplicación][9]

> [!TIP]
> El sistema en tiempo de ejecución de Service Fabric Actors emite algunos [eventos y contadores de rendimiento relacionados con los métodos de actor](service-fabric-reliable-actors-diagnostics.md#actor-method-events-and-performance-counters). Son útiles para la supervisión del rendimiento y los diagnósticos.

## <a name="next-steps"></a>Pasos siguientes
Aprenda más sobre [el uso de la plataforma de Service Fabric por Reliable Actor](service-fabric-reliable-actors-platform.md).


[1]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-newproject.PNG
[2]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-projectstructure.PNG
[3]: ./media/service-fabric-reliable-actors-get-started/debugging-output.PNG
[4]: ./media/service-fabric-reliable-actors-get-started/vs-context-menu.png
[5]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-newproject1.PNG
[6]: ./media/service-fabric-reliable-actors-get-started/new-console-app.png
[7]: ./media/service-fabric-reliable-actors-get-started/add-reference.png
[8]: ./media/service-fabric-reliable-actors-get-started/build-props.png
[9]: ./media/service-fabric-reliable-actors-get-started/app-output.png
