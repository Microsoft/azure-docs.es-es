---
title: Llamada a aplicaciones lógicas con Azure Functions
description: Creación de funciones de Azure que llaman a aplicaciones lógicas o las desencadenan mediante la escucha a Azure Service Bus
services: logic-apps
ms.suite: integration
ms.reviewer: jehollan, klam, logicappspm
ms.topic: article
ms.date: 11/08/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: fcf7f1a27633c978c10f541d0a341225fbcb126d
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89013782"
---
# <a name="call-or-trigger-logic-apps-by-using-azure-functions-and-azure-service-bus"></a>Llamada o desencadenamiento de aplicaciones lógicas con Azure Functions y Azure Service Bus

Puede utilizar [Azure Functions](../azure-functions/functions-overview.md) para desencadenar una aplicación lógica cuando necesite implementar una tarea o un cliente de escucha de ejecución prolongada. Por ejemplo, puede crear una función de Azure que escuche en una cola de [Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md) y que active inmediatamente una aplicación lógica como desencadenador de push.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* Un espacio de nombres de Azure Service Bus. Si no tiene un espacio de nombres [creélo primero](../service-bus-messaging/service-bus-create-namespace-portal.md).

* Una aplicación de función de Azure, que es un contenedor para las funciones de Azure. Si no tiene una aplicación de función, [créela primero](../azure-functions/functions-create-first-azure-function.md) y asegúrese de seleccionar .NET como la pila en tiempo de ejecución.

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md)

## <a name="create-logic-app"></a>Creación de la aplicación lógica

En este escenario, tiene una función que ejecuta cada aplicación lógica que desea desencadenar. En primer lugar, cree una aplicación lógica que se inicie con un desencadenador de solicitud HTTP. La función llama a ese punto de conexión siempre que se reciba un mensaje en cola.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y cree la aplicación lógica en blanco.

   Si es la primera vez que usa aplicaciones lógicas, revise [Inicio rápido: Creación de la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

1. En el cuadro de búsqueda, escriba `http request`. En la lista de desencadenadores, seleccione el desencadenador **Cuando se recibe una solicitud HTTP**.

   ![Seleccionar un desencadenador](./media/logic-apps-scenario-function-sb-trigger/when-http-request-received-trigger.png)

   En el desencadenador de solicitud, puede escribir, si lo desea, un esquema JSON para usarlo con el mensaje de la cola. Los esquemas JSON ayudan al Diseñador de aplicación lógica a conocer la estructura de los datos de entrada y facilitan el uso de las salidas en el flujo de trabajo.

1. Para especificar un esquema, especifique el esquema en el cuadro **Esquema JSON de cuerpo de solicitud**, por ejemplo:

   ![Especificación del esquema JSON](./media/logic-apps-scenario-function-sb-trigger/when-http-request-received-trigger-schema.png)

   Si no tiene un esquema, pero tiene una carga de ejemplo en formato JSON, puede generar un esquema a partir de dicha carga.

   1. En el desencadenador de solicitud, seleccione **Use sample payload to generate schema** (Usar una carga de ejemplo para generar el esquema).

   1. En **Permite especificar o pegar una carga de JSON de ejemplo**, proporcione una carga de ejemplo y luego seleccione **Listo**.

      ![Especificación de la carga de ejemplo](./media/logic-apps-scenario-function-sb-trigger/enter-sample-payload.png)

   Esta carga de ejemplo genera este esquema que aparece en el desencadenador:

   ```json
   {
      "type": "object",
      "properties": {
         "address": {
            "type": "object",
            "properties": {
               "number": {
                  "type": "integer"
               },
               "street": {
                  "type": "string"
               },
               "city": {
                  "type": "string"
               },
               "postalCode": {
                  "type": "integer"
               },
               "country": {
                  "type": "string"
               }
            }
         }
      }
   }
   ```

1. Agregue todas las demás acciones que desee ejecutar después de recibir el mensaje de la cola.

   Por ejemplo, puede enviar un correo electrónico con el conector Office 365 Outlook.

1. Guarde la aplicación lógica, que genera la dirección URL de devolución de llamada para el desencadenador en esta aplicación lógica. Más adelante, use esta dirección URL de devolución de llamada en el código para el desencadenador de cola de Azure Service Bus.

   Esta dirección URL aparece en la propiedad **HTTP POST URL**.

   ![Generación de la dirección URL de devolución de llamada para el desencadenador](./media/logic-apps-scenario-function-sb-trigger/callback-URL-for-trigger.png)

## <a name="create-azure-function"></a>Creación de una función de Azure

A continuación, cree la función que actúa como desencadenador y escuche la cola.

1. En Azure Portal, abra y expanda la aplicación de función, si aún no está abierta. 

1. En el nombre de la aplicación de función, expanda **Functions**. En el panel **Functions**, seleccione **Nueva función**.

   ![Expanda "Functions" y seleccione "Nueva función"](./media/logic-apps-scenario-function-sb-trigger/add-new-function-to-function-app.png)

1. Seleccione esta plantilla en función de si ha creado una nueva aplicación de función donde seleccionó .NET como la pila en tiempo de ejecución o si está usando una aplicación de función existente.

   * Para las nuevas aplicaciones de función, seleccione esta plantilla: **Desencadenador de cola de Service Bus**

     ![Seleccione la plantilla para la nueva aplicación de función](./media/logic-apps-scenario-function-sb-trigger/current-add-queue-trigger-template.png)

   * Para una aplicación de función existente, seleccione esta plantilla: **Service Bus Queue trigger: C#** (Desencadenador de cola de Service Bus)

     ![Selección de la plantilla para la aplicación de función existente](./media/logic-apps-scenario-function-sb-trigger/legacy-add-queue-trigger-template.png)

1. En el panel **Azure Service Bus Queue trigger** (Desencadenador de cola de Azure Service Bus), proporcione un nombre para el desencadenador y configure la **conexión de Service Bus** para la cola, que usa el cliente de escucha `OnMessageReceive()` del SDK de Azure Service Bus y seleccione **Create** (Crear).

1. Escriba una función básica para llamar al punto de conexión de la aplicación lógica creado anteriormente usando el mensaje de cola como desencadenador. Antes de escribir la función, revise estas consideraciones:

   * En el ejemplo se usa el tipo de contenido de mensaje `application/json`, pero puede cambiarlo si es necesario.
   
   * Debido a la posible ejecución simultánea de funciones, grandes volúmenes o cargas pesadas, evite crear instancias de la clase [HTTPClient](/dotnet/api/system.net.http.httpclient) con la`using` instrucción y crear directamente instancias de HTTPClient por solicitud. Para obtener más información, consulte el artículo sobre el [uso de HttpClientFactory para implementar solicitudes HTTP resistentes](/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests#issues-with-the-original-httpclient-class-available-in-net-core).
   
   * Si es posible, vuelva a usar la instancia de los clientes HTTP. Para más información, consulte [Manage connections in Azure Functions](../azure-functions/manage-connections.md) (Administración de conexiones en Azure Functions).

   En este ejemplo se utiliza el [método `Task.Run`](/dotnet/api/system.threading.tasks.task.run) en modo [asincrónico](/dotnet/csharp/language-reference/keywords/async). Para obtener más información, consulte [Programación asincrónica con Async y Await](/dotnet/csharp/programming-guide/concepts/async/).

   ```csharp
   using System;
   using System.Threading.Tasks;
   using System.Net.Http;
   using System.Text;

   // Can also fetch from App Settings or environment variable
   private static string logicAppUri = @"https://prod-05.westus.logic.azure.com:443/workflows/<remaining-callback-URL>";

   // Reuse the instance of HTTP clients if possible: https://docs.microsoft.com/azure/azure-functions/manage-connections
   private static HttpClient httpClient = new HttpClient();

   public static async Task Run(string myQueueItem, TraceWriter log) 
   {
      log.Info($"C# ServiceBus queue trigger function processed message: {myQueueItem}");
      var response = await httpClient.PostAsync(logicAppUri, new StringContent(myQueueItem, Encoding.UTF8, "application/json")); 
   }
   ```

1. Para probar la función, agregue un mensaje de cola mediante una herramienta como el [Explorador de Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer).

   La aplicación lógica se desencadena inmediatamente después de que la función recibe el mensaje.

## <a name="next-steps"></a>Pasos siguientes

* [Llamar, desencadenar o anidar flujos de trabajo con puntos de conexión HTTP en Azure Logic Apps](../logic-apps/logic-apps-http-endpoint.md)
