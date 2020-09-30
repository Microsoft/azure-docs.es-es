---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/25/2020
ms.author: glenga
ms.openlocfilehash: 894a89126d1ee3ed909134f3e0dd914166568654
ms.sourcegitcommit: 6e1124fc25c3ddb3053b482b0ed33900f46464b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90606543"
---
La cola se puede ver en [Azure Portal](../articles/storage/queues/storage-quickstart-queues-portal.md) o en el [Explorador de Microsoft Azure Storage](https://storageexplorer.com/). También puede ver la cola en la CLI de Azure, como se describe en los pasos siguientes:

1. Abra el archivo *local.setting.json* del proyecto de Functions y copie el valor de la cadena de conexión. En una ventana de terminal o de comandos, ejecute el siguiente comando para crear una variable de entorno denominada `AZURE_STORAGE_CONNECTION_STRING` y pegue la cadena de conexión concreta en lugar de `<MY_CONNECTION_STRING>`. (Esta variable de entorno significa que no es necesario proporcionar la cadena de conexión a cada comando posterior mediante el argumento `--connection-string`).

    # <a name="bash"></a>[bash](#tab/bash)
    
    ```bash
    export AZURE_STORAGE_CONNECTION_STRING="<MY_CONNECTION_STRING>"
    ```
    
    # <a name="powershell"></a>[PowerShell](#tab/powershell)
    
    ```powershell
    $env:AZURE_STORAGE_CONNECTION_STRING = "<MY_CONNECTION_STRING>"
    ```
    
    # <a name="azure-cli"></a>[CLI de Azure](#tab/cmd)
    
    ```azurecli
    set AZURE_STORAGE_CONNECTION_STRING="<MY_CONNECTION_STRING>"
    ```
    
    ---
    
1. (Opcional) Puede usar el comando [`az storage queue list`](/cli/azure/storage/queue#az-storage-queue-list) para ver las colas de Storage de la cuenta. La salida de este comando debe incluir una cola denominada `outqueue`, la cual se creó cuando la función escribió su primer mensaje en esa cola.
    
    ```azurecli
    az storage queue list --output tsv
    ```

1. Use el comando [`az storage message get`](/cli/azure/storage/message#az-storage-message-get) para leer el mensaje de esta cola, que será el primer nombre que usó al probar la función anteriormente. El comando lee y quita el primer mensaje de la cola. 

    # <a name="bash"></a>[bash](#tab/bash)
    
    ```bash
    echo `echo $(az storage message get --queue-name outqueue -o tsv --query '[].{Message:content}') | base64 --decode`
    ```
    
    # <a name="powershell"></a>[PowerShell](#tab/powershell)
    
    ```powershell
    [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($(az storage message get --queue-name outqueue -o tsv --query '[].{Message:content}')))
    ```
    
    # <a name="azure-cli"></a>[CLI de Azure](#tab/cmd)
    
    ```azurecli
    az storage message get --queue-name outqueue -o tsv --query [].{Message:content} > %TEMP%out.b64 && certutil -decode -f %TEMP%out.b64 %TEMP%out.txt > NUL && type %TEMP%out.txt && del %TEMP%out.b64 %TEMP%out.txt /q
    ```

    Este script usa certutil para descodificar la colección de mensajes codificados en Base64 desde un archivo temporal local. Si no hay ninguna salida, intente quitar `> NUL` del script para dejar de suprimir la salida de certutil, en caso de que se produzca un error. 
    
    ---
    
    Dado que el cuerpo del mensaje se almacena [codificado en Base64](../articles/azure-functions/functions-bindings-storage-queue-trigger.md#encoding), el mensaje se debe descodificar antes de visualizarse. Después de ejecutar `az storage message get`, el mensaje se quita de la cola. Si solo había un mensaje en `outqueue`, no se recuperará al ejecutar este comando una segunda vez y, en su lugar, recibirá un error.
