---
title: Creación de una función en Azure desencadenada por mensajes en cola
description: Use Azure Functions para crear una función sin servidor que se invoque mediante mensajes enviados a una cola de Azure.
ms.assetid: 361da2a4-15d1-4903-bdc4-cc4b27fc3ff4
ms.topic: how-to
ms.date: 10/01/2018
ms.custom: mvc, cc996988-fb4f-47
ms.openlocfilehash: d722d420597bb459d3e7b6d2ca33fdc49bfe6f09
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90981574"
---
# <a name="create-a-function-triggered-by-azure-queue-storage"></a>Crear una función desencadenada por Azure Queue Storage

Obtenga información sobre cómo crear una función que se desencadena cuando se envían mensajes a una cola de Azure Storage.

## <a name="prerequisites"></a>Requisitos previos

- Suscripción a Azure. Si no tiene una, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="create-an-azure-function-app"></a>Creación de una Function App de Azure

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

   :::image type="content" source="./media/functions-create-storage-queue-triggered-function/function-app-create-success.png" alt-text="La aplicación de funciones se creó correctamente." border="true":::

Después, cree una función en la nueva Function App.

<a name="create-function"></a>

## <a name="create-a-queue-triggered-function"></a>Creación de una función desencadenada por el servicio Queue

1. Seleccione **Functions** y, a continuación, **+ Agregar** para agregar una función nueva.

   :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-app-quickstart-choose-template.png" alt-text="La aplicación de funciones se creó correctamente." border="true":::

1. Elija la plantilla **Azure Queue Storage trigger** (Desencadenador de Azure Queue Storage).

1. Utilice la configuración que se especifica en la tabla debajo de la imagen.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-create-queue-storage-trigger-portal.png" alt-text="La aplicación de funciones se creó correctamente." border="true":::


    | Configuración | Valor sugerido | Descripción |
    |---|---|---|
    | **Nombre** | Único en la Function App | Nombre de la función desencadenada por la cola. |
    | **Nombre de la cola**   | myqueue-items    | Nombre de la cola a la que se va a conectar en la cuenta de almacenamiento. |
    | **Conexión de la cuenta de almacenamiento** | AzureWebJobsStorage | Puede usar la conexión de cuenta de almacenamiento que ya usa la Function App o crear una nueva.  |    

1. Seleccione **Crear función** para crear la función.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-create-queue-storage-trigger-portal-3.png" alt-text="La aplicación de funciones se creó correctamente." border="true":::

Después, conéctese a su cuenta de Azure Storage y cree la cola de almacenamiento **myqueue-items**.

## <a name="create-the-queue"></a>Creación de la cola

1. En la función, en la página **Información general**, seleccione el grupo de recursos.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-storage-resource-group.png" alt-text="La aplicación de funciones se creó correctamente." border="true":::

1. Busque y seleccione la cuenta de almacenamiento del grupo de recursos.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-storage-account-access.png" alt-text="La aplicación de funciones se creó correctamente." border="true":::

1. Elija **Colas** y, a continuación, elija **+ Cola**. 

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-storage-add-queue.png" alt-text="La aplicación de funciones se creó correctamente." border="true":::

1. En el campo **Nombre**, escriba `myqueue-items` y seleccione **Crear**.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-storage-name-queue.png" alt-text="La aplicación de funciones se creó correctamente." border="true":::

Ahora que tiene una cola de almacenamiento, puede probar la función. Para ello, agregue un mensaje a la cola.

## <a name="test-the-function"></a>Prueba de la función

1. De nuevo en Azure Portal, vaya a la función. Expanda **Registros** en la parte inferior de la página y asegúrese de que el streaming de registros no está en pausa.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-queue-storage-log-expander.png" alt-text="La aplicación de funciones se creó correctamente." border="true":::.

1. En una ventana del explorador independiente, vaya al grupo de recursos de Azure Portal y seleccione la cuenta de almacenamiento.

1. Seleccione **Colas** y, a continuación, el contenedor **myqueue-items**.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-storage-queue.png" alt-text="La aplicación de funciones se creó correctamente." border="true":::

1. Seleccione **Agregar mensaje** y escriba "¡Hola mundo!". en **Texto del mensaje**. Seleccione **Aceptar**.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-storage-queue-test.png" alt-text="La aplicación de funciones se creó correctamente." border="true":::

1. Espere unos segundos y, después, vuelva a los registros de función para comprobar que se ha leído el mensaje nuevo de la cola.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/function-app-in-portal-editor.png" alt-text="La aplicación de funciones se creó correctamente." border="true":::

1. En Storage Queue, seleccione **Actualizar** y verifique que el mensaje se haya procesado y ya no esté en la cola.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [Next steps note](../../includes/functions-quickstart-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

Ha creado una función que se ejecuta cuando se agrega un mensaje a una cola de almacenamiento. Para obtener más información sobre los desencadenadores de Queue Storage, vea [Enlaces de colas de Storage en Azure Functions](functions-bindings-storage-queue.md).

Ahora que ha creado su primera función, vamos a agregar un enlace de salida a la función que escribe un mensaje en otra cola.

> [!div class="nextstepaction"]
> [Agregar mensajes a una cola de Azure Storage mediante funciones](functions-integrate-storage-queue-output-binding.md)
