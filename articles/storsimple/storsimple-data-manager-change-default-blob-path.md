---
title: Cambio del valor predeterminado de la ruta de acceso de un blob
description: Aprenda a configurar una función de Azure para cambiar el nombre de una ruta de acceso de archivo de blob predeterminada y así mover los blobs a una ubicación diferente.
author: alkohli
ms.service: storsimple
ms.topic: how-to
ms.date: 01/16/2018
ms.author: alkohli
ms.openlocfilehash: 0a7dab1129eb88d7e58bab8a827d745596bc218d
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88183724"
---
# <a name="change-a-blob-path-from-the-default-path"></a>Cambio del valor predeterminado de la ruta de acceso de un blob

Cuando el servicio StorSimple Data Manager transforma los datos, coloca los blobs transformados de forma predeterminada en un contenedor de almacenamiento del modo especificado durante la creación del repositorio de destino. A medida que los blobs llegan a esta ubicación, es posible que desee moverlos a otra. En este artículo se describe cómo configurar una función de Azure para cambiar el nombre de una ruta de acceso de archivo de blob predeterminada y así mover los blobs a una ubicación diferente.

## <a name="prerequisites"></a>Prerequisites

Asegúrese de que tiene una definición de trabajo configurada correctamente en el servicio StorSimple Data Manager.

## <a name="create-an-azure-function"></a>Creación de una función de Azure

Lleve a cabo estos pasos para crear una función de Azure:

1. Vaya a [Azure Portal](https://portal.azure.com/).

2. Haga clic en **+ Crear un recurso**. En el cuadro **Búsqueda**, escriba **Function App** y presione **Entrar**. Seleccione y haga clic en **Funcion App** en la lista de aplicaciones que aparece.

    ![Escritura de "Function App" en el cuadro de búsqueda](./media/storsimple-data-manager-change-default-blob-path/search-function-app.png)

3. Haga clic en **Crear**.

    ![Botón "Crear" de la ventana de Function App](./media/storsimple-data-manager-change-default-blob-path/create-function-app.png)

4. En la hoja de configuración **Function App**, realice estos pasos:

    1. Proporcione un **nombre de aplicación** único.
    2. En la lista desplegable, seleccione **Suscripción**. Esta suscripción debe ser la misma que la asociada al servicio StorSimple Data Manager.
    3. Selección el grupo de recursos **Crear nuevo**.
    4. En la lista desplegable **Plan de hospedaje**, seleccione **Plan de consumo**.
    5. Especifique una ubicación para que se ejecute la función. Seguramente desee la misma región en la que se encuentra el servicio StorSimple Data Manager y la cuenta de almacenamiento asociada a la definición del trabajo.
    6. Seleccione una cuenta de almacenamiento existente o cree una nueva cuenta de almacenamiento. Una cuenta de almacenamiento se usa de forma interna para la función.

        ![Introducción de los datos de configuración de Function App](./media/storsimple-data-manager-change-default-blob-path/function-app-parameters.png)

    7. Haga clic en **Crear**. La instancia de Function App se creará.
     
        ![Function App creada](./media/storsimple-data-manager-change-default-blob-path/function-app-created.png)

5. Seleccione **Funciones** y haga clic en **+ Nueva función**.

    ![Clic en + Nueva función](./media/storsimple-data-manager-change-default-blob-path/create-new-function.png)

6. Seleccione **C#** como lenguaje. En la matriz de iconos de plantilla, seleccione **C#** en el icono **QueueTrigger-CSharp**.

7. En el **Desencadenador de cola**:

    1. Escriba un **Nombre** para la función.
    2. En el cuadro **Nombre de cola**, escriba el nombre de la definición del trabajo de transformación de datos.
    3. En **Conexión de la cuenta de almacenamiento**, haga clic en **Nuevo**. En la lista de cuentas de almacenamiento, seleccione la cuenta asociada a la definición del trabajo. Anote el nombre de la conexión (destacado). La función de Azure necesitará este nombre más adelante.

        ![Creación de una función de C#](./media/storsimple-data-manager-change-default-blob-path/new-function-parameters.png)

    4. Haga clic en **Crear**. Se crea la **función**.

     
10. En la ventana Función, ejecute el archivo _.csx_.

    ![Creación de una función de C#](./media/storsimple-data-manager-change-default-blob-path/new-function-run-csx.png)
    
    Lleve a cabo los siguiente pasos.

    1. Pegue el código siguiente:

        ```
        using System;
        using System.Configuration;
        using Microsoft.WindowsAzure.Storage.Blob;
        using Microsoft.WindowsAzure.Storage.Queue;
        using Microsoft.WindowsAzure.Storage;
        using System.Collections.Generic;
        using System.Linq;

        public static void Run(QueueItem myQueueItem, TraceWriter log)
        {
            CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConfigurationManager.AppSettings["STORAGE_CONNECTIONNAME"]);

            string storageAccUriEndswith = "windows.net/";
            string uri = myQueueItem.TargetLocation.Replace("%20", " ");
            log.Info($"Blob Uri: {uri}");

            // Remove storage account uri string
            uri = uri.Substring(uri.IndexOf(storageAccUriEndswith) + storageAccUriEndswith.Length);

            string containerName = uri.Substring(0, uri.IndexOf("/")); 

            // Remove container name string
            uri = uri.Substring(containerName.Length + 1);

            // Current blob path
            string blobName = uri; 

            string volumeName = uri.Substring(containerName.Length + 1);
            volumeName = uri.Substring(0, uri.IndexOf("/"));

            // Remove volume name string
            uri = uri.Substring(volumeName.Length + 1);

            string newContainerName = uri.Substring(0, uri.IndexOf("/")).ToLower();
            string newBlobName = uri.Substring(newContainerName.Length + 1);

            log.Info($"Container name: {containerName}");
            log.Info($"Volume name: {volumeName}");
            log.Info($"New container name: {newContainerName}");

            log.Info($"Blob name: {blobName}");
            log.Info($"New blob name: {newBlobName}");

            // Create the blob client.
            CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

            // Container reference
            CloudBlobContainer container = blobClient.GetContainerReference(containerName);
            CloudBlobContainer newContainer = blobClient.GetContainerReference(newContainerName);
            newContainer.CreateIfNotExists();

            if(!container.Exists())
            {
                log.Info($"Container - {containerName} not exists");
                return;
            }

            if(!newContainer.Exists())
            {
                log.Info($"Container - {newContainerName} not exists");
                return;
            }

            CloudBlockBlob blob = container.GetBlockBlobReference(blobName);
            if (!blob.Exists())
            {
                // Skip to copy the blob to new container, if source blob doesn't exist
                log.Info($"The specified blob does not exist.");
                log.Info($"Blob Uri: {blob.Uri}");
                return;
            }

            CloudBlockBlob blobCopy = newContainer.GetBlockBlobReference(newBlobName);
            if (!blobCopy.Exists())
            {
                blobCopy.StartCopy(blob);
                // Delete old blob, after copy to new container
                blob.DeleteIfExists();
                log.Info($"Blob file path renamed completed successfully");
            }
            else
            {
                log.Info($"Blob file path renamed already done");
                // Delete old blob, if already exists.
                blob.DeleteIfExists();
            }
        }

        public class QueueItem
        {
            public string SourceLocation {get;set;}
            public long SizeInBytes {get;set;}
            public string Status {get;set;}
            public string JobID {get;set;}
            public string TargetLocation {get; set;}
        }

        ```

    2. Reemplace **STORAGE_CONNECTIONNAME** en la línea 11 por la conexión de cuenta de almacenamiento (consulte el paso 7c).

        ![Copia del nombre de la conexión de almacenamiento](./media/storsimple-data-manager-change-default-blob-path/new-function-storage-connection-name.png)

    3. **Guarde** la función.

        ![Función Guardar](./media/storsimple-data-manager-change-default-blob-path/save-function.png)

12. Para completar la función, agregue un archivo más siguiendo estos pasos:

    1. Haga clic en **View files** (Ver archivos).

       ![Vínculo "View files" (Ver archivos)](./media/storsimple-data-manager-change-default-blob-path/view-files.png)

    2. Haga clic en **+ Agregar**.
        
        ![Vínculo "View files" (Ver archivos)](./media/storsimple-data-manager-change-default-blob-path/new-function-add-file.png)
    
    3. Escriba **project.json** y presione **Entrar**. En el archivo **project.json**, peque el siguiente código:

        ```
        {
        "frameworks": {
            "net46":{
            "dependencies": {
                "windowsazure.storage": "8.1.1"
            }
            }
        }
        }

        ```

    
    4. Haga clic en **Save**(Guardar).

        ![Vínculo "View files" (Ver archivos)](./media/storsimple-data-manager-change-default-blob-path/new-function-project-json.png)

Acaba de crear una función de Azure. Esta función se desencadena cada vez que el trabajo de transformación de datos genera un blob nuevo.

## <a name="next-steps"></a>Pasos siguientes

[Uso de la IU de StorSimple Data Manager para transformar los datos](storsimple-data-manager-ui.md)
