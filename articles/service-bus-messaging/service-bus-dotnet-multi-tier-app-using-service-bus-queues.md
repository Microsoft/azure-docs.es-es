---
title: Aplicación de niveles múltiples .NET mediante Azure Service Bus | Microsoft Docs
description: Un tutorial .NET que le ayuda a desarrollar una aplicación de varios niveles en Azure que usa colas de Service Bus para comunicarse entre los niveles.
ms.devlang: dotnet
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 044d0698f1ea181e8f508f92ad2c30ec29b6490b
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88067841"
---
# <a name="net-multi-tier-application-using-azure-service-bus-queues"></a>Aplicación de niveles múltiples .NET con colas de Azure Service Bus

Desarrollar para Microsoft Azure es muy sencillo con Visual Studio y el SDK de Azure para .NET gratis. Este tutorial lo guía por los pasos para crear una aplicación que usa varios recursos de Azure que se ejecutan en su entorno local.

Aprenderá lo siguiente:

* Cómo habilitar el equipo para el desarrollo de Azure con una única descarga e instalación.
* Cómo utilizar Visual Studio para desarrollar para Azure.
* Cómo crear una aplicación de niveles múltiples en Azure mediante el uso de roles web y de trabajo.
* Cómo comunicarse entre niveles mediante las colas de Service Bus.

[!INCLUDE [create-account-note](../../includes/create-account-note.md)]

En este tutorial compilará y ejecutará la aplicación de niveles múltiples en un servicio en la nube de Azure. El front-end es un rol web de ASP.NET MVC y el back-end es un rol de trabajo que usa una cola de Service Bus. Puede crear la misma aplicación de niveles múltiples con el front-end como un proyecto web, que se implementa en un sitio web de Azure, en lugar de en un servicio en la nube. También puede probar el tutorial de [aplicación híbrida en la nube/local .NET](../azure-relay/service-bus-dotnet-hybrid-app-using-service-bus-relay.md).

La siguiente captura de pantalla muestra la aplicación completada:

![Captura de pantalla de la página Enviar de la aplicación][0]

## <a name="scenario-overview-inter-role-communication"></a>Información general del escenario: comunicación entre roles
Para enviar una orden para su procesamiento, el componente de la interfaz de usuario de front-end, que se ejecuta en el rol web, debe interactuar con la lógica del nivel intermedio que se ejecuta en el rol de trabajo. En ese ejemplo se utiliza la mensajería de Service Bus para la comunicación entre los niveles.

El uso de la mensajería de Service Bus entre los niveles de web e intermedio desacopla los dos componentes. Al contrario que en la mensajería directa (es decir, TCP o HTTP), el nivel web no se conecta directamente al nivel intermedio; por el contrario, inserta unidades de trabajo, como mensajes, en Service Bus, que los conserva de manera confiable hasta que el nivel intermedio esté preparado para consumirlas y procesarlas.

Service Bus ofrece dos entidades para admitir la mensajería asincrónica: colas y temas. Con las colas, cada mensaje enviado a la cola lo consume un único receptor. Los temas admiten el patrón de publicación/suscripción, en el cual cada mensaje publicado está disponible para una suscripción registrada con el tema. Cada suscripción mantiene lógicamente su propia cola de mensajes. Las suscripciones también se pueden configurar con reglas de filtro que restringen el conjunto de mensajes pasados a la cola de suscripción a aquellos que coinciden con el filtro. En este ejemplo se usan las colas de Service Bus.

![Diagrama que muestra la comunicación entre el rol web, Service Bus y el rol de trabajo.][1]

Este mecanismo de comunicación tiene varias ventajas sobre la mensajería directa:

* **Desacoplamiento temporal.** Con el patrón de mensajería asincrónica, los productores y los consumidores no tienen que estar en línea al mismo tiempo. Service Bus almacena los mensajes de manera confiable hasta que la parte consumidora esté lista para recibirlos. De esta forma, los componentes de la aplicación distribuida se pueden desconectar, ya sea voluntariamente, por ejemplo, para mantenimiento, o debido a un bloqueo del componente, sin que afecte al sistema en su conjunto. Es más, puede que la aplicación consumidora solo necesite estar en línea durante determinados períodos del día.
* **Redistribución de la carga.** En muchas aplicaciones, la carga del sistema varía con el tiempo, mientras que el tiempo de procesamiento requerido para cada unidad de trabajo suele ser constante. La intermediación de productores y consumidores de mensajes con una cola implica que la aplicación consumidora (el trabajador) solo necesita ser aprovisionada para acomodar una carga promedio en lugar de una carga pico. La profundidad de la cola aumenta y se contrae a medida que varíe la carga entrante, lo que permite ahorrar dinero directamente en función de la cantidad de infraestructura requerida para dar servicio a la carga de la aplicación.
* **Equilibrio de carga.** A medida que aumenta la carga, se pueden agregar más procesos de trabajo para que puedan leerse desde la cola. Cada mensaje se procesa únicamente por uno de los procesos de trabajo. Además, este equilibrio de carga permite el uso óptimo de las máquinas de trabajo aunque estas máquinas difieran en términos de capacidad de procesamiento, ya que extraerán mensajes a su frecuencia máxima propia. Este patrón con frecuencia se denomina patrón de *consumo de competidor*.
  
  ![Diagrama que muestra la comunicación entre el rol web, Service Bus y dos roles de trabajo.][2]

En las secciones siguientes se explica el código que implementa esta arquitectura.

## <a name="create-a-namespace"></a>Creación de un espacio de nombres

El primer paso consiste en crear un *espacio de nombres* y obtener una [clave de Firma de acceso compartido (SAS)](service-bus-sas.md) para dicho espacio de nombres. Un espacio de nombres proporciona un límite de aplicación para cada aplicación que se expone a través de Service Bus. El sistema genera una clave SAS cuando se crea un espacio de nombres. La combinación del espacio de nombres y la clave SAS proporciona las credenciales de Service Bus para autenticar el acceso a una aplicación.

[!INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

## <a name="create-a-web-role"></a>Creación de un rol web

En esta sección, va a crear el front-end de la aplicación. En primer lugar, va a crear las páginas que mostrará la aplicación.
Después, va a agregar el código que envía elementos a una cola de Service Bus y muestra información de estado sobre la cola.

### <a name="create-the-project"></a>Creación del proyecto

1. Inicie Visual Studio con privilegios de administrador: haga clic con el botón derecho en el icono del programa **Visual Studio** y después haga clic en **Ejecutar como administrador**. El emulador de proceso de Azure, descrito posteriormente en este artículo, requiere que se inicie Visual Studio con privilegios de administrador.
   
   En Visual Studio, en el menú **Archivo**, haga clic en **Nuevo** y, a continuación, en **Proyecto**.
2. En **Plantillas instaladas**, en **Visual C#** , haga clic en **Nube** y, a continuación, en **Azure Cloud Service**. Denomine el proyecto **MultiTierApp**. A continuación, haga clic en **Aceptar**.
   
   ![Captura de pantalla del cuadro de diálogo Nuevo proyecto con Nube seleccionada y Azure Cloud Service Visual C# resaltado y con contorno en rojo.][9]
3. En el panel **Roles**, haga doble clic en **Rol web de ASP.NET**.
   
   ![Captura de pantalla del cuadro de diálogo Nuevo servicio de nube de Microsoft Azure con el rol web ASP.NET seleccionado y WebRole1 también seleccionado.][10]
4. Mantenga el cursor sobre **WebRole1** en **Solución de servicio de nube de Azure**, haga clic en el icono de lápiz y cambie el nombre del rol web a **FrontendWebRole**. A continuación, haga clic en **Aceptar**. (Asegúrese de que escribe "Frontend" con "e" minúscula, no "FrontEnd").
   
   ![Captura de pantalla del cuadro de diálogo Nuevo servicio de nube de Microsoft Azure con la solución con nombre cambiado a FrontendWebRole.][11]
5. En el cuadro de diálogo **Nuevo proyecto de ASP.NET**, en la lista **Seleccionar una plantilla**, haga clic en **MVC**.
   
   ![Captura de pantalla del cuadro de diálogo Nuevo proyecto de ASP.NET con MVC resaltado y con contorno en rojo, y la opción Cambiar autenticación con contorno en rojo.][12]
6. Todavía en el cuadro de diálogo **Nuevo proyecto de ASP.NET**, haga clic en el botón **Cambiar autenticación**. En el cuadro de diálogo **Cambiar autenticación**, asegúrese de que esté seleccionada la opción **Sin autenticación** y luego haga clic en **Aceptar**. En este tutorial, va a implementar una aplicación que no necesita un inicio de sesión de usuario.
   
    ![Captura de pantalla del cuadro de diálogo Cambiar autenticación con la opción Sin autenticación seleccionada y con contorno en rojo.][16]
7. De vuelta en el cuadro de diálogo **Nuevo proyecto de ASP.NET**, haga clic en **Aceptar** para crear el proyecto.
8. En el **Explorador de soluciones**, en el proyecto **FrontendWebRole**, haga clic con el botón derecho en **Referencias** y haga clic en **Administrar paquetes NuGet**.
9. Haga clic en la pestaña **Examinar** y, después, busque **WindowsAzure.ServiceBus**. Seleccione el paquete **WindowsAzure.ServiceBus**, haga clic en **Instalar** y acepte las condiciones de uso.
   
   ![Captura de pantalla del cuadro de diálogo Administrar paquetes NuGet con el WindowsAzure.ServiceBus resaltado y la opción Instalar con contorno en rojo.][13]
   
   Tenga en cuenta que ahora se hace referencia a los ensamblados del cliente requeridos y que se han agregado algunos archivos de código nuevos.
10. En el **Explorador de soluciones**, haga clic con el botón derecho en **Modelos** y, luego, en **Agregar** y, por último, en **Clase**. En el cuadro **Nombre**, escriba el nombre **OnlineOrder.cs**. A continuación, haga clic en **Agregar**.

### <a name="write-the-code-for-your-web-role"></a>Especificación del código del rol web
En esta sección, creará las distintas páginas que va a mostrar la aplicación.

1. En el archivo OnlineOrder.cs en Visual Studio, sustituya la definición del espacio de nombres existentes por el código siguiente:
   
   ```csharp
   namespace FrontendWebRole.Models
   {
       public class OnlineOrder
       {
           public string Customer { get; set; }
           public string Product { get; set; }
       }
   }
   ```
2. En el **Explorador de soluciones**, haga doble clic en **Controllers\HomeController.cs**. Agregue las siguientes instrucciones **using** en la parte superior del archivo para incluir los espacios de nombres en el modelo que acaba de crear, así como Service Bus.
   
   ```csharp
   using FrontendWebRole.Models;
   using Microsoft.ServiceBus.Messaging;
   using Microsoft.ServiceBus;
   ```
3. En el archivo HomeController.cs en Visual Studio, sustituya la definición del espacio de nombres existentes por el código siguiente. Este código contiene métodos para controlar el envío de elementos a la cola.
   
   ```csharp
   namespace FrontendWebRole.Controllers
   {
       public class HomeController : Controller
       {
           public ActionResult Index()
           {
               // Simply redirect to Submit, since Submit will serve as the
               // front page of this application.
               return RedirectToAction("Submit");
           }
   
           public ActionResult About()
           {
               return View();
           }
   
           // GET: /Home/Submit.
           // Controller method for a view you will create for the submission
           // form.
           public ActionResult Submit()
           {
               // Will put code for displaying queue message count here.
   
               return View();
           }
   
           // POST: /Home/Submit.
           // Controller method for handling submissions from the submission
           // form.
           [HttpPost]
           // Attribute to help prevent cross-site scripting attacks and
           // cross-site request forgery.  
           [ValidateAntiForgeryToken]
           public ActionResult Submit(OnlineOrder order)
           {
               if (ModelState.IsValid)
               {
                   // Will put code for submitting to queue here.
   
                   return RedirectToAction("Submit");
               }
               else
               {
                   return View(order);
               }
           }
       }
   }
   ```
4. En el menú **Compilar**, haga clic en **Compilar solución** para probar la precisión del trabajo hasta ahora.
5. Ahora, va a crear la vista para el método `Submit()` que creó antes. Haga clic con el botón derecho en el método `Submit()` (la sobrecarga de `Submit()` que no acepta parámetros) y elija **Agregar vista**.
   
   ![Captura de pantalla del código con el foco en el método Submit y una lista desplegable con la opción Agregar vista resaltada.][14]
6. Se muestra un cuadro de diálogo para crear la vista. En la lista **Plantilla**, elija **Crear**. En la lista **Clase de modelo**, seleccione la clase **OnlineOrder**.
   
   ![Captura de pantalla del cuadro de diálogo Agregar vista con las listas desplegables Plantilla y Clase de modelo con contorno en rojo.][15]
7. Haga clic en **Agregar**.
8. Ahora, cambie el nombre mostrado de la aplicación. En el **Explorador de soluciones**, haga doble clic en el archivo **Views\Shared\\_Layout.cshtml**para abrirlo en el editor de Visual Studio.
9. Reemplace todas las repeticiones de **Mi aplicación ASP.NET** por **Productos de Northwind Traders**.
10. Quite los vínculos **Página principal**, **Acerca de** y **Contacto**. Elimine el código resaltado:
    
    ![Captura de pantalla del código con tres líneas del código de Vínculo de acción HTML resaltado.][28]
11. Finalmente, modifique la página de envío para incluir información sobre la cola. En el **Explorador de soluciones**, haga doble clic en el archivo **Views\Home\Submit.cshtml** para abrirlo en el editor de Visual Studio. Agregue la línea siguiente después de `<h2>Submit</h2>`. Por ahora, `ViewBag.MessageCount` está vacío. Lo rellenará más adelante.
    
    ```html
    <p>Current number of orders in queue waiting to be processed: @ViewBag.MessageCount</p>
    ```
12. Acaba de implementar su interfaz de usuario. Puede presionar **F5** para ejecutar la aplicación y confirmar que tiene el aspecto previsto.
    
    ![Captura de pantalla de la página Enviar de la aplicación][17]

### <a name="write-the-code-for-submitting-items-to-a-service-bus-queue"></a>Especificación del código para enviar elementos a una cola de Service Bus
Ahora, agregue el código para enviar elementos a una cola. En primer lugar, va a crear una clase que contiene la información de conexión a la cola de Service Bus. A continuación, inicialice la conexión en Global.aspx.cs. Finalmente, actualice el código de envío que creó antes en HomeController.cs para enviar realmente los elementos a una cola de Service Bus.

1. En el **Explorador de soluciones**, haga clic con el botón derecho en **FrontendWebRole** (haga clic con el botón derecho en el proyecto, no en el rol). Haga clic en **Agregar** y, a continuación, en **Clase**.
2. Asigne a la clase el nombre **QueueConnector.cs**. Haga clic en **Agregar** para crear la clase.
3. Ahora va a agregar código que encapsula la información de conexión e inicializará la conexión a una cola de Service Bus. Reemplace todo el contenido de QueueConnector.cs por el código siguiente y escriba valores para `your Service Bus namespace` (el nombre del espacio de nombres) y `yourKey`, la **clave principal** que obtuvo antes en Azure Portal.
   
   ```csharp
   using System;
   using System.Collections.Generic;
   using System.Linq;
   using System.Web;
   using Microsoft.ServiceBus.Messaging;
   using Microsoft.ServiceBus;
   
   namespace FrontendWebRole
   {
       public static class QueueConnector
       {
           // Thread-safe. Recommended that you cache rather than recreating it
           // on every request.
           public static QueueClient OrdersQueueClient;
   
           // Obtain these values from the portal.
           public const string Namespace = "your Service Bus namespace";
   
           // The name of your queue.
           public const string QueueName = "OrdersQueue";
   
           public static NamespaceManager CreateNamespaceManager()
           {
               // Create the namespace manager which gives you access to
               // management operations.
               var uri = ServiceBusEnvironment.CreateServiceUri(
                   "sb", Namespace, String.Empty);
               var tP = TokenProvider.CreateSharedAccessSignatureTokenProvider(
                   "RootManageSharedAccessKey", "yourKey");
               return new NamespaceManager(uri, tP);
           }
   
           public static void Initialize()
           {
               // Using Http to be friendly with outbound firewalls.
               ServiceBusEnvironment.SystemConnectivity.Mode =
                   ConnectivityMode.Http;
   
               // Create the namespace manager which gives you access to
               // management operations.
               var namespaceManager = CreateNamespaceManager();
   
               // Create the queue if it does not exist already.
               if (!namespaceManager.QueueExists(QueueName))
               {
                   namespaceManager.CreateQueue(QueueName);
               }
   
               // Get a client to the queue.
               var messagingFactory = MessagingFactory.Create(
                   namespaceManager.Address,
                   namespaceManager.Settings.TokenProvider);
               OrdersQueueClient = messagingFactory.CreateQueueClient(
                   "OrdersQueue");
           }
       }
   }
   ```
4. Ahora, asegúrese de que se llama al método **Initialize**. En el **Explorador de soluciones**, haga doble clic en **Global.asax\Global.asax.cs**.
5. Agregue la siguiente línea de código al final del método **Application_Start**.
   
   ```csharp
   FrontendWebRole.QueueConnector.Initialize();
   ```
6. Por último, actualice el código web que creó anteriormente para enviar elementos a la cola. En el **Explorador de soluciones**, haga doble clic en **Controllers\HomeController.cs**.
7. Actualice el método `Submit()` (la sobrecarga que no acepta parámetros) como sigue para obtener el número de mensajes de la cola.
   
   ```csharp
   public ActionResult Submit()
   {
       // Get a NamespaceManager which allows you to perform management and
       // diagnostic operations on your Service Bus queues.
       var namespaceManager = QueueConnector.CreateNamespaceManager();
   
       // Get the queue, and obtain the message count.
       var queue = namespaceManager.GetQueue(QueueConnector.QueueName);
       ViewBag.MessageCount = queue.MessageCount;
   
       return View();
   }
   ```
8. Actualice el método `Submit(OnlineOrder order)` (la sobrecarga que no acepta parámetros) como sigue para enviar información de pedidos a la cola.
   
   ```csharp
   public ActionResult Submit(OnlineOrder order)
   {
       if (ModelState.IsValid)
       {
           // Create a message from the order.
           var message = new BrokeredMessage(order);
   
           // Submit the order.
           QueueConnector.OrdersQueueClient.Send(message);
           return RedirectToAction("Submit");
       }
       else
       {
           return View(order); 
       }
   }
   ```
9. Ahora puede volver a ejecutar la aplicación. Cada vez que envía un pedido, el contador de mensajes aumenta.
   
   ![Captura de pantalla de la página de envío de la aplicación con el recuento de mensajes incrementado en 1.][18]

## <a name="create-the-worker-role"></a>Creación del rol de trabajo
Ahora creará el rol de trabajo que procesa los envíos del pedido. Este ejemplo utiliza la plantilla de proyecto de Visual Studio de **rol de trabajo con cola de Service Bus**. Ya ha obtenido las credenciales necesarias del portal.

1. Asegúrese de que se ha conectado Visual Studio a su cuenta de Azure.
2. En Visual Studio, en el **Explorador de soluciones**, haga clic con el botón derecho en la carpeta **Roles** en el proyecto **MultiTierApp**.
3. Haga clic en **Agregar** y, a continuación, en **Nuevo proyecto de rol de trabajo**. Aparecerá el cuadro de diálogo **Agregar nuevo proyecto de rol**.
   
   ![Captura de pantalla del panel Explorador de soluciones con la opción Nuevo proyecto de rol de trabajo y la opción Agregar resaltadas.][26]
4. En el cuadro de diálogo **Agregar nuevo proyecto de rol**, haga clic en **Rol de trabajo con cola de Service Bus**.
   
   ![Captura de pantalla del cuadro de diálogo Agregar nuevo proyecto de rol con el rol de trabajo con la opción Cola de Service Bus resaltada y con contorno en rojo.][23]
5. En el cuadro **Nombre**, asigne al proyecto el nombre **OrderProcessingRole**. A continuación, haga clic en **Agregar**.
6. Copie en el Portapapeles la cadena de conexión que obtuvo en el paso 9 de la sección "Creación de un espacio de nombres de Service Bus".
7. En el **Explorador de soluciones**, haga clic con el botón derecho en el elemento **OrderProcessingRole** que creó en el paso 5 (asegúrese de hacer clic con el botón derecho en **OrderProcessingRole** en **Roles** y no en la clase). A continuación, haga clic en **Propiedades**.
8. En la pestaña **Configuración** del cuadro de diálogo **Propiedades**, haga clic dentro del cuadro **Valor** para **Microsoft.ServiceBus.ConnectionString** y, a continuación, pegue el valor de punto de conexión que copió en el paso 6.
   
   ![Captura de pantalla del cuadro de diálogo Propiedades con la pestaña Configuración seleccionada y la fila Microsoft.ServiceBus.ConnectionString de la tabla con contorno en rojo.][25]
9. Cree una clase **OnlineOrder** para representar los pedidos cuando los procese desde la cola. Puede reutilizar una clase que ya ha creado. En el **Explorador de soluciones**, haga clic con el botón derecho en la clase **OrderProcessingRole** (haga clic con el botón derecho en el icono de clase, no en el rol). Haga clic en **Agregar** y, a continuación, en **Elemento existente**.
10. Busque **FrontendWebRole\Models** en la subcarpeta y haga doble clic en **OnlineOrder.cs** para agregarlo a este proyecto.
11. En **WorkerRole.cs**, cambie el valor de la variable **QueueName** de `"ProcessingQueue"` a `"OrdersQueue"`, como se muestra en el código siguiente.
    
    ```csharp
    // The name of your queue.
    const string QueueName = "OrdersQueue";
    ```
12. Agregue la siguiente instrucción using en la parte superior del archivo WorkerRole.cs.
    
    ```csharp
    using FrontendWebRole.Models;
    ```
13. En la función `Run()`, dentro de la llamada `OnMessage()`, reemplace el contenido de la cláusula `try` por el código siguiente.
    
    ```csharp
    Trace.WriteLine("Processing", receivedMessage.SequenceNumber.ToString());
    // View the message as an OnlineOrder.
    OnlineOrder order = receivedMessage.GetBody<OnlineOrder>();
    Trace.WriteLine(order.Customer + ": " + order.Product, "ProcessingMessage");
    receivedMessage.Complete();
    ```
14. Ha completado la aplicación. Puede probar la aplicación completa haciendo clic con el botón derecho en el proyecto MultiTierApp en el Explorador de soluciones, seleccionando **Establecer como proyecto de inicio** y, a continuación, presionando F5. Tenga en cuenta que el contador de mensajes no se aumenta, ya que el rol de trabajo procesa los elementos de la cola y los marca como finalizados. Puede ver el resultado de seguimiento de su rol de trabajo viendo la interfaz de usuario del emulador de Azure Compute. Para ello, haga clic con el botón derecho en el icono del emulador del área de notificación de la barra de tareas y seleccione **Mostrar interfaz de usuario del emulador de proceso**.
    
    ![Captura de pantalla de lo que aparece al hacer clic en el icono del emulador. Mostrar la interfaz de usuario del emulador de proceso se muestra en la lista de opciones.][19]
    
    ![Captura de pantalla del cuadro de diálogo Emulador de proceso de Microsoft Azure (Rápido).][20]

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre Service Bus, consulte los siguientes recursos:  

* [Introducción al uso de las colas de Service Bus][sbacomqhowto]
* [Página de servicio de Service Bus][sbacom]  

Para más información sobre los escenarios de niveles múltiples, consulte:  

* [Aplicación de niveles múltiples .NET mediante tablas, colas y blobs de Storage][mutitierstorage]  

[0]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-app.png
[1]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-100.png
[2]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-101.png
[9]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-10.png
[10]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-11.png
[11]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-02.png
[12]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-12.png
[13]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-13.png
[14]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-33.png
[15]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-34.png
[16]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-14.png
[17]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-app.png
[18]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-app2.png

[19]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-38.png
[20]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-39.png
[23]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/SBWorkerRole1.png
[25]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/SBWorkerRoleProperties.png
[26]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/SBNewWorkerRole.png
[28]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-40.png

[sbacom]: https://azure.microsoft.com/services/service-bus/  
[sbacomqhowto]: service-bus-dotnet-get-started-with-queues.md  
[mutitierstorage]: https://code.msdn.microsoft.com/Windows-Azure-Multi-Tier-eadceb36