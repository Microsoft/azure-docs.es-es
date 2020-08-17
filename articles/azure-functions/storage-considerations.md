---
title: Consideraciones de almacenamiento de Azure Functions
description: Conozca los requisitos de almacenamiento de Azure Functions y aprenda a cifrar los datos almacenados.
ms.topic: conceptual
ms.date: 07/27/2020
ms.openlocfilehash: aefd9a35235a09d94973f383603349f6862bbdd9
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318188"
---
# <a name="storage-considerations-for-azure-functions"></a>Consideraciones de almacenamiento de Azure Functions

Azure Functions necesita una cuenta de Azure Storage para crear una instancia de la aplicación de funciones. Esta aplicación de funciones puede utilizar los siguientes servicios de almacenamiento:


|Servicio de Storage  | Uso de Functions  |
|---------|---------|
| [Almacenamiento de blobs de Azure](../storage/blobs/storage-blobs-introduction.md)     | Mantener el estado de los enlaces y las teclas de función.  <br/>También se utiliza en la [central de tareas de Durable Functions](durable/durable-functions-task-hubs.md). |
| [Archivos de Azure](../storage/files/storage-files-introduction.md)  | Recurso compartido de archivos que se utiliza para almacenar y ejecutar el código de la aplicación de funciones en un [plan de consumo](functions-scale.md#consumption-plan) y un [plan prémium](functions-scale.md#premium-plan). |
| [Azure Queue Storage](../storage/queues/storage-queues-introduction.md)     | Se utiliza en la [central de tareas de Durable Functions](durable/durable-functions-task-hubs.md).   |
| [Azure Table Storage](../storage/tables/table-storage-overview.md)  |  Se utiliza en la [central de tareas de Durable Functions](durable/durable-functions-task-hubs.md).       |

> [!IMPORTANT]
> Cuando usa el plan de hospedaje de consumo o prémium, los archivos de configuración de enlace y el código de la función se almacenan en Azure File Storage en la cuenta de almacenamiento principal. Si elimina la cuenta de almacenamiento principal, este contenido se suprimirá y no se podrá recuperar.

## <a name="storage-account-requirements"></a>Requisitos de la cuenta de almacenamiento

Al crear una aplicación de funciones, debe crear o vincular una cuenta de Azure Storage de uso general compatible con Blob, Queue y Table Storage. El motivo es que Azure Functions usa Azure Storage con ciertas operaciones; por ejemplo, para administrar los desencadenadores y ejecutar las funciones de registro. Algunas cuentas de almacenamiento no admiten el uso de colas y tablas; por ejemplo, las cuentas de almacenamiento solo para blobs, Azure Premium Storage y las cuentas de almacenamiento de uso general con replicación ZRS. Estas cuentas no compatibles no se muestran en la hoja Cuenta de almacenamiento cuando se crea una aplicación de funciones.

Para más información sobre los tipos de cuenta de almacenamiento, consulte [Introducción de los servicios Azure Storage](../storage/common/storage-introduction.md#core-storage-services). 

Aunque puede usar una cuenta de almacenamiento existente con la aplicación de funciones, debe asegurarse de que cumple estos requisitos. En las cuentas de almacenamiento que se generan durante el flujo de creación de la aplicación de funciones, el cumplimiento de los requisitos de la cuenta de almacenamiento está garantizado.  

## <a name="storage-account-guidance"></a>Guía de la cuenta de almacenamiento

Cada aplicación de función requiere una cuenta de almacenamiento para funcionar. Si se elimina la cuenta, la aplicación de funciones no se ejecutará. Para más información, consulte este artículo sobre [la solución de problemas relacionados con el almacenamiento](functions-recover-storage-account.md). Estas otras consideraciones que se indican a continuación también se aplican a la cuenta de almacenamiento que se utiliza en las aplicaciones de funciones.

### <a name="storage-account-connection-setting"></a>Configuración de la conexión de la cuenta de almacenamiento

La conexión de la cuenta de almacenamiento se mantiene en [el ajuste de la aplicación AzureWebJobsStorage](./functions-app-settings.md#azurewebjobsstorage). 

Cuando se generan las claves de almacenamiento, debe actualizarse la cadena de conexión de la cuenta de almacenamiento. [Más información sobre la administración de las claves de almacenamiento](../storage/common/storage-account-create.md).

### <a name="shared-storage-accounts"></a>Cuentas de almacenamiento compartidas

Una misma cuenta de almacenamiento puede estar compartida entre varias aplicaciones de funciones sin que se produzcan problemas. Por ejemplo, en Visual Studio, puede desarrollar varias aplicaciones con el Emulador de Azure Storage. En este caso, el emulador actúa como una cuenta de almacenamiento única. La cuenta de almacenamiento que se utiliza en la aplicación de funciones puede usarse también para almacenar los datos de la aplicación. Sin embargo, este enfoque no siempre resulta conveniente en los entornos de producción.

### <a name="optimize-storage-performance"></a>Optimización del rendimiento de almacenamiento

[!INCLUDE [functions-shared-storage](../../includes/functions-shared-storage.md)]

## <a name="storage-data-encryption"></a>Cifrado de datos de almacenamiento

[!INCLUDE [functions-storage-encryption](../../includes/functions-storage-encryption.md)]

## <a name="mount-file-shares-linux"></a>Montaje de recursos compartidos de archivos (Linux)

Puede montar recursos compartidos de Azure Files existentes en las aplicaciones de funciones de Linux. Al montar un recurso compartido en la aplicación de funciones de Linux, puede aprovechar los modelos de aprendizaje automático existentes u otros datos en sus funciones. Puede usar el comando [`az webapp config storage-account add`](/cli/azure/webapp/config/storage-account#az-webapp-config-storage-account-add) para montar un recurso compartido existente en la aplicación de funciones de Linux. 

En este comando, `share-name` es el nombre del recurso compartido existente de Azure Files y `custom-id` puede ser cualquier cadena que defina de forma única el recurso compartido cuando se monta en la aplicación de funciones. Además, `mount-path` es la ruta de acceso desde la que se accede al recurso compartido en la aplicación de funciones. `mount-path` debe estar en el formato `/dir-name` y no puede comenzar con `/home`.

Para ver un ejemplo completo, consulte los scripts de [Creación de una aplicación de funciones de Python y montaje de un recurso compartido de Azure Files](scripts/functions-cli-mount-files-storage-linux.md). 

Actualmente, solo se admite `storage-type` de `AzureFiles`. Solo puede montar cinco recursos compartidos en una aplicación de funciones determinada. El montaje de un recurso compartido de archivos puede aumentar el tiempo de arranque en frío en al menos 200-300 ms o incluso más si la cuenta de almacenamiento se encuentra en una región diferente.

El recurso compartido montado está disponible para el código de función en el valor `mount-path` especificado. Por ejemplo, cuando `mount-path` es `/path/to/mount`, puede acceder al directorio de destino mediante las API del sistema de archivos, como en el siguiente ejemplo de Python:

```python
import os
...

files_in_share = os.listdir("/path/to/mount")
```

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las opciones de hospedaje de Azure Functions.

> [!div class="nextstepaction"]
> [Escalado y hospedaje de Azure Functions](functions-scale.md)
