---
title: Desencadenamiento de un trabajo de Batch con Azure Functions
description: 'Tutorial: aplicación de OCR en documentos digitalizados a medida que se agregan a un blob de almacenamiento'
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 05/30/2019
ms.author: peshultz
ms.custom: mvc, devx-track-csharp
ms.openlocfilehash: 6e481219c6be68f9e9da06d92b6c28998cc7a6e2
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88930101"
---
# <a name="tutorial-trigger-a-batch-job-using-azure-functions"></a>Tutorial: Desencadenamiento de un trabajo de Batch con Azure Functions

En este tutorial, obtendrá información sobre cómo desencadenar un trabajo de Batch con Azure Functions. Le guiaremos en un ejemplo en el que se aplica el reconocimiento óptico de caracteres (OCR) a los documentos agregados a un contenedor de blobs de Azure Storage mediante Azure Batch. Para agilizar el procesamiento de OCR, configuraremos una función que ejecuta un trabajo de OCR de Batch cada vez que se agrega un archivo al contenedor de blobs de Azure.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Una cuenta de Azure Batch y una cuenta de Azure Storage vinculada. Consulte [Creación de una cuenta de Batch](quick-create-portal.md#create-a-batch-account) para más información sobre cómo crear y vincular cuentas.
* [Batch Explorer](https://azure.github.io/BatchExplorer/)
* [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/)

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="create-a-batch-pool-and-batch-job-using-batch-explorer"></a>Creación de un grupo y un trabajo de Batch con Batch Explorer

En esta sección, usará Batch Explorer para crear el grupo y el trabajo de Batch que ejecutarán las tareas de OCR. 

### <a name="create-a-pool"></a>Creación de un grupo

1. Inicie sesión en Batch Explorer con sus credenciales de Azure.
1. Para crear un grupo, seleccione **Grupos** en la barra lateral izquierda y, después, el botón **Agregar** situado sobre el formulario de búsqueda. 
    1. Elija un identificador y el nombre para mostrar. Usaremos `ocr-pool` para este ejemplo.
    1. Establezca el tipo de escala en **Tamaño fijo** y establezca el número de nodos dedicados en 3.
    1. Seleccione **Ubuntu 18.04-LTS** como sistema operativo.
    1. Elija `Standard_f2s_v2` como tamaño de la máquina virtual.
    1. Habilite la tarea de inicio y agregue el comando `/bin/bash -c "sudo update-locale LC_ALL=C.UTF-8 LANG=C.UTF-8; sudo apt-get update; sudo apt-get -y install ocrmypdf"`. Asegúrese de establecer la identidad del usuario como **usuario predeterminado de la tarea (Admin)** , que permite que las tareas de inicio incluyan comandos con `sudo`.
    1. Seleccione **Aceptar**.
### <a name="create-a-job"></a>Creación de un trabajo

1. Para crear un trabajo en el grupo, seleccione **Trabajos** en la barra lateral izquierda y, después, el botón **Agregar** situado sobre el formulario de búsqueda. 
    1. Elija un identificador y el nombre para mostrar. Usaremos `ocr-job` para este ejemplo.
    1. Establezca el grupo en `ocr-pool` o en el nombre que eligiera para el grupo.
    1. Seleccione **Aceptar**.


## <a name="create-blob-containers"></a>Creación de contenedores de blobs

Aquí podrá crear contenedores de blobs que almacenarán la entrada y salida de archivos para el trabajo de Batch OCR.

1. Inicie sesión en Explorador de Storage con sus credenciales de Azure.
1. Con la cuenta de almacenamiento vinculada a su cuenta de Batch, cree dos contenedores de blobs (uno para los archivos de entrada y otro para los de salida); para ello, siga los pasos descritos en [Creación de un contenedor de blobs](../vs-azure-tools-storage-explorer-blobs.md#create-a-blob-container).

En este ejemplo, el contenedor de entrada se denomina `input` y es donde se cargan inicialmente todos los documentos sin reconocimiento óptico de caracteres para ser procesados. El contenedor de salida se denomina `output` y es donde el trabajo de Batch escribe los documentos procesados con OCR.  
    * En este ejemplo, llamaremos a nuestro contenedor de entrada `input` y al contenedor de salida, `output`.  
    * El contenedor de entrada es donde se cargan inicialmente todos los documentos sin reconocimiento óptico de caracteres.  
    * El contenedor de salida es donde el trabajo de Batch escribe los documentos procesados con OCR.  

Cree una firma de acceso compartido para el contenedor de salida en el Explorador de Storage. Para ello, haga clic con el botón derecho en el contenedor de salida y seleccione **Obtener firma de acceso compartido**. En **Permisos**, active **Escritura**. No se necesitan más permisos.  

## <a name="create-an-azure-function"></a>Creación de una Función de Azure

En esta sección creará la función de Azure que desencadena el proceso de Batch OCR cuando se carga un archivo en el contenedor de entrada.

1. Siga los pasos de [Crear una función desencadenada por Azure Blob Storage](../azure-functions/functions-create-storage-blob-triggered-function.md) para crear una función.
    1. Cuando se le solicite una cuenta de almacenamiento, utilice la misma cuenta de almacenamiento que ha vinculado a su cuenta de Batch.
    1. En **Pila en tiempo de ejecución**, elija .NET. Escribiremos nuestra función C# para aprovechar el SDK de .NET para Batch.
1. Una vez creada la función desencadenada por blob, use [`run.csx`](https://github.com/Azure-Samples/batch-functions-tutorial/blob/master/run.csx) y [`function.proj`](https://github.com/Azure-Samples/batch-functions-tutorial/blob/master/function.proj) desde GitHub en la función.
    * `run.csx` se ejecuta cuando se agrega un nuevo blob al contenedor de blobs de entrada.
    * `function.proj` enumera las bibliotecas externas en el código de Function, por ejemplo, el SDK de .NET para Batch.
1. Cambie los valores del marcador de posición de las variables en la función `Run()` del archivo `run.csx` para que refleje sus credenciales de Batch y almacenamiento. Puede buscar las credenciales de sus cuentas de Batch y de Storage en Azure Portal, en la sección **Claves** de su cuenta de Batch.
    * Recupere las credenciales de sus cuentas de Batch y de Storage en Azure Portal, en la sección **Claves** de su cuenta de Batch. 

## <a name="trigger-the-function-and-retrieve-results"></a>Desencadenar la función y recuperar resultados

Cargue alguno o todos los archivos analizados desde el directorio [`input_files`](https://github.com/Azure-Samples/batch-functions-tutorial/tree/master/input_files) en GitHub al contenedor de entrada. Supervise Batch Explorer para confirmar que se agrega una tarea a `ocr-pool` para cada archivo. Después de unos segundos, se agrega al contenedor de salida el archivo con el reconocimiento óptico de caracteres aplicado. El archivo entonces es visible y se puede recuperar en el Explorador de Storage.

Además, puede ver el archivo de registros en la parte inferior de la ventana de editor web de Azure Functions, donde verá mensajes similares a este para cada archivo que se carga en el contenedor de entrada:

```
2019-05-29T19:45:25.846 [Information] Creating job...
2019-05-29T19:45:25.847 [Information] Accessing input container <inputContainer>...
2019-05-29T19:45:25.847 [Information] Adding <fileName> as a resource file...
2019-05-29T19:45:25.848 [Information] Name of output text file: <outputTxtFile>
2019-05-29T19:45:25.848 [Information] Name of output PDF file: <outputPdfFile>
2019-05-29T19:45:26.200 [Information] Adding OCR task <taskID> for <fileName> <size of fileName>...
```

Para descargar los archivos de salida desde el Explorador de Storage a la máquina local, seleccione primero los archivos que desee y, a continuación, seleccione **Descargar** en la cinta de opciones superior. 

> [!TIP]
> Se puede buscar en los archivos descargados si se abren en un lector de PDF.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido cómo: 

> [!div class="checklist"]
> * Usar Batch Explorer para crear grupos y trabajos
> * Usar el Explorador de Storage para crear una firma de acceso compartido (SAS) y contenedores de blobs
> * Creación de una función de Azure desencadenada mediante blobs
> * Cargar archivos de entrada en Storage
> * Supervisar la ejecución de las tareas
> * Recuperación de archivos de salida

* Para más muestras de uso de la API de .NET para programar y procesar cargas de trabajo de Batch, consulte [las muestras de GitHub](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp). 

* Para ver más desencadenadores de Azure Functions que puede usar para ejecutar cargas de trabajo de Batch, consulte [la documentación de Azure Functions](../azure-functions/functions-triggers-bindings.md).
