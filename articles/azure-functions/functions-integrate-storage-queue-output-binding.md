---
title: Agregar mensajes a una cola de Azure Storage con Functions
description: Use Azure Functions para crear una función sin servidor que se invoca mediante una solicitud HTTP y crea un mensaje en una cola de Azure Storage.
ms.assetid: 0b609bc0-c264-4092-8e3e-0784dcc23b5d
ms.topic: how-to
ms.date: 04/24/2020
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: 9c635b01bcd04bd03191fca2590b0189bad0f544
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90982027"
---
# <a name="add-messages-to-an-azure-storage-queue-using-functions"></a>Agregar mensajes a una cola de Azure Storage con Functions

En Azure Functions, los enlaces de entrada y salida proporcionan una forma declarativa de hacer que los datos de servicios externos estén disponibles para su código. En esta guía de inicio rápido, se utiliza en enlace de salida para crear un mensaje en una cola cuando una solicitud HTTP desencadena una función. Utilice el contenedor de Azure Storage para ver los mensajes de cola que crea la función.

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía de inicio rápido:

- Suscripción a Azure. Si no tiene una, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

- Siga las instrucciones de [Creación de su primera función desde Azure Portal](functions-create-first-azure-function.md) y no realice el paso **Limpieza de recursos**. Esta guía de inicio rápido crea la aplicación de función y la función que se va a utilizar aquí.

## <a name="add-an-output-binding"></a><a name="add-binding"></a>Agregar un enlace de salida

En esta sección, se va a utilizar la interfaz de usuario del portal para agregar un enlace de salida de almacenamiento en cola a la función que se ha creado anteriormente. Con este enlace podrá escribir un código mínimo para crear un mensaje en una cola. No tiene que escribir código para tareas como la apertura de una conexión de almacenamiento, la creación de una cola o la obtención de una referencia a una cola. El sistema en tiempo de ejecución de Azure Functions y el enlace de salida en cola se encargan automáticamente de esas tareas.

1. En Azure Portal, abra la página correspondiente a la aplicación de función que ha creado en [Creación de su primera función desde Azure Portal](functions-create-first-azure-function.md). Para abrir la página, busque y seleccione **Aplicación de funciones**. Posteriormente, seleccione la aplicación de funciones.

1. Seleccione la aplicación de funciones y, luego, seleccione la función que creó en la guía de inicio rápido anterior.

1. Seleccione **Integración** y, a continuación, seleccione **+ Agregar salida**.

   :::image type="content" source="./media/functions-integrate-storage-queue-output-binding/function-create-output-binding.png" alt-text="Cree un enlace de salida para la función." border="true":::

1. Seleccione el tipo de enlace **Azure Queue Storage** y agregue la configuración como se especifica en la tabla que sigue a esta captura de pantalla: 

    :::image type="content" source="./media/functions-integrate-storage-queue-output-binding/function-create-output-binding-details.png" alt-text="Cree un enlace de salida para la función." border="true":::
    
    | Configuración      |  Valor sugerido   | Descripción                              |
    | ------------ |  ------- | -------------------------------------------------- |
    | **Nombre del parámetro de mensaje** | outputQueueItem | El nombre del parámetro del enlace de salida. | 
    | **Nombre de la cola**   | outqueue  | El nombre de la cola a la que se va a conectar en la cuenta de almacenamiento. |
    | **Conexión de la cuenta de almacenamiento** | AzureWebJobsStorage | Puede usar la conexión de cuenta de almacenamiento que ya usa la Function App o crear una nueva.  |

1. Seleccione **Aceptar** para agregar el enlace.

Ahora que tiene definido un enlace de salida, necesita actualizar el código que va a usar el enlace para agregar mensajes a una cola.  

## <a name="add-code-that-uses-the-output-binding"></a>Adición de código que utilice el enlace de salida

En esta sección, va a agregar código que escribe un mensaje en la cola de salida. El mensaje incluye el valor que se pasa al desencadenador HTTP en la cadena de consulta. Por ejemplo, si la cadena de consulta incluye `name=Azure`, el mensaje de cola será *Name passed to the function: Azure* (Nombre pasado a la función: Azure).

1. En la función, seleccione **Código y prueba** para mostrar su código en el editor.

1. Actualice el código de función dependiendo del lenguaje de la función:

    # <a name="c"></a>[C\#](#tab/csharp)

    Agregue un parámetro **outputQueueItem** para la firma del método, tal como se muestra en el ejemplo siguiente.

    ```cs
    public static async Task<IActionResult> Run(HttpRequest req,
        ICollector<string> outputQueueItem, ILogger log)
    {
        ...
    }
    ```

    En el cuerpo de la función, justo antes de la instrucción `return`, agregue código que utilice el parámetro para crear un mensaje de cola.

    ```cs
    outputQueueItem.Add("Name passed to the function: " + name);
    ```

    # <a name="javascript"></a>[JavaScript](#tab/nodejs)

    Agregue código que utilice el enlace de salida en el objeto `context.bindings` para crear un mensaje de la cola. Agregue este código antes de la instrucción `context.done`.

    ```javascript
    context.bindings.outputQueueItem = "Name passed to the function: " + 
                (req.query.name || req.body.name);
    ```

    ---

1. Seleccione **Guardar** para guardar los cambios.

## <a name="test-the-function"></a>Prueba de la función

1. Después de que se guarden los cambios del código, seleccione **Prueba**.
1. Confirme que la prueba coincide con la imagen siguiente y seleccione **Ejecutar**. 

    :::image type="content" source="./media/functions-integrate-storage-queue-output-binding/functions-test-run-function.png" alt-text="Cree un enlace de salida para la función." border="true":::

    Tenga en cuenta que el **cuerpo de la solicitud** contiene el valor de `name`*Azure*. Este valor aparece en el mensaje de la cola que se crea cuando se invoca la función.
    
    Como alternativa a la selección de **Ejecutar** aquí, puede llamar a la función mediante la introducción de una dirección URL en un navegador y de la especificación del valor `name` en la cadena de consulta. El método de explorador se muestra en la [guía de inicio rápido anterior](functions-create-first-azure-function.md#test-the-function).

1. Compruebe los registros para asegurarse de que la función se ha realizado correctamente. 

Se crea una nueva cola denominada **outqueue** en su cuenta de Storage mediante el entorno de ejecución de Functions cuando el enlace de salida se usa por primera vez. Usará la cuenta de almacenamiento para comprobar que se han creado la cola y un mensaje.

### <a name="find-the-storage-account-connected-to-azurewebjobsstorage"></a>Buscar la cuenta de almacenamiento conectada a AzureWebJobsStorage


1. Vaya a la aplicación de funciones y seleccione **Configuración**.

1. En **Configuración de la aplicación**, seleccione **AzureWebJobsStorage**.

    :::image type="content" source="./media/functions-integrate-storage-queue-output-binding/function-find-storage-account.png" alt-text="Cree un enlace de salida para la función." border="true":::

1. Busque y anote el nombre de la cuenta.

    :::image type="content" source="./media/functions-integrate-storage-queue-output-binding/function-storage-account-name.png" alt-text="Cree un enlace de salida para la función." border="true":::

### <a name="examine-the-output-queue"></a>Examen de la cola de salida

1. En el grupo de recursos de la aplicación de funciones, seleccione la cuenta de almacenamiento que esté usando para esta guía de inicio rápido.

1. En **Queue service**, seleccione **Colas** y seleccione la cola denominada **outqueue**. 

   La cola contiene el mensaje que creó el enlace de salida de la cola al ejecutar la función desencadenada por HTTP. Si se invoca la función con el valor predeterminado `name` de *Azure*, el mensaje de cola es *Name passed to the function: Azure* (Nombre pasado a la función: Azure).

1. Vuelva a ejecutar la función y aparecerá un nuevo mensaje en la cola.  

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [Clean up resources](../../includes/functions-quickstart-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha agregado un enlace de salida a una función existente. Para obtener más información sobre los enlaces a Queue Storage, vea [Enlaces de cola de Storage en Azure Functions](functions-bindings-storage-queue.md).

[!INCLUDE [Next steps note](../../includes/functions-quickstart-next-steps-2.md)]
