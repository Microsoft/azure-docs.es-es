---
title: Carga de datos de imagen en la nube con Azure Storage | Microsoft Docs
description: Uso de Azure Blob Storage con una aplicación web para almacenar datos de la aplicación
services: storage
author: normesta
ms.service: storage
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 11/26/2018
ms.author: normesta
ms.reviewer: seguler
ms.custom: mvc
ms.openlocfilehash: 26b92db330c882aaf258b6e24560cbf2f7930a5f
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65237140"
---
# <a name="tutorial-upload-image-data-in-the-cloud-with-azure-storage"></a>Tutorial: Carga de datos de imagen en la nube con Azure Storage

Este tutorial es la primera parte de una serie. En este tutorial se muestra cómo implementar una aplicación web que usa la biblioteca de cliente de Azure Storage para cargar imágenes en una cuenta de almacenamiento. Cuando haya terminado, tendrá una aplicación web que almacena y muestra imágenes desde Azure Storage.

# <a name="nettabdotnet"></a>[\.NET](#tab/dotnet)
![Vista del contenedor de imágenes](media/storage-upload-process-images/figure2.png)

# <a name="nodejs-v2-sdktabnodejs"></a>[Node.js V2 SDK](#tab/nodejs)
![Vista del contenedor de imágenes](media/storage-upload-process-images/upload-app-nodejs-thumb.png)

# <a name="nodejs-v10-sdktabnodejsv10"></a>[Node.js V10 SDK](#tab/nodejsv10)
![Vista del contenedor de imágenes](media/storage-upload-process-images/upload-app-nodejs-thumb.png)

---

En la primera parte de la serie, se aprende a:

> [!div class="checklist"]
> * Crear una cuenta de almacenamiento
> * Crear un contenedor y establecer permisos
> * Recuperar una clave de acceso
> * Implementar una aplicación web en Azure
> * Configuración de aplicaciones
> * Interactuar con la aplicación web

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesitará una suscripción de Azure. Cree una [cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) antes de comenzar.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Para instalar y usar la CLI de forma local, en este tutorial necesitará ejecutar la CLI de Azure versión 2.0.4 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli). 

## <a name="create-a-resource-group"></a>Crear un grupo de recursos 

Cree un grupo de recursos con el comando [az group create](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.  

En el ejemplo siguiente se crea un grupo de recursos denominado `myResourceGroup`.

```azurecli-interactive
az group create --name myResourceGroup --location southeastasia 
```

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

En el ejemplo se cargan imágenes en un contenedor de blobs en una cuenta de Azure Storage. Una cuenta de almacenamiento proporciona un espacio de nombres único para almacenar y tener acceso a los objetos de datos de almacenamiento de Azure. Cree una cuenta de almacenamiento en el grupo de recursos que ha creado con el comando [az storage account create](/cli/azure/storage/account).

> [!IMPORTANT]
> En la parte 2 del tutorial, usará Azure Event Grid con Blob Storage. Asegúrese de crear la cuenta de almacenamiento en una región de Azure que admita Event Grid. Para obtener una lista de las regiones admitidas, consulte [Productos de Azure por región](https://azure.microsoft.com/global-infrastructure/services/?products=event-grid&regions=all).

En el siguiente comando, sustituya su propio nombre único global por la cuenta de Blob Storage donde vea el marcador de posición `<blob_storage_account>`.  

```azurecli-interactive
blobStorageAccount=<blob_storage_account>

az storage account create --name $blobStorageAccount \
--location southeastasia --resource-group myResourceGroup \
--sku Standard_LRS --kind blobstorage --access-tier hot 
```

## <a name="create-blob-storage-containers"></a>Creación de contenedores de almacenamiento de blobs

La aplicación usa dos contenedores en la cuenta de Blob Storage. Los contenedores son similares a las carpetas y almacenan blobs. El contenedor de *imágenes* es donde la aplicación carga imágenes a alta resolución. En una parte posterior de la serie, una aplicación de función de Azure carga imágenes en miniatura cambiadas de tamaño en el contenedor *thumbnails*.

Obtenga la clave de la cuenta de almacenamiento mediante el comando [az storage account keys list](/cli/azure/storage/account/keys). A continuación, use esta clave para crear dos contenedores con el comando [az storage container create](/cli/azure/storage/container).  

El acceso público del contenedor de *imágenes* está establecido en `off`. El acceso público del contenedor de *miniaturas* está establecido en `container`. La configuración de acceso público `container` permite ver las miniaturas a los usuarios que visitan la página web.

```azurecli-interactive
blobStorageAccountKey=$(az storage account keys list -g myResourceGroup \
-n $blobStorageAccount --query [0].value --output tsv)

az storage container create -n images --account-name $blobStorageAccount \
--account-key $blobStorageAccountKey --public-access off

az storage container create -n thumbnails --account-name $blobStorageAccount \
--account-key $blobStorageAccountKey --public-access container

echo "Make a note of your Blob storage account key..."
echo $blobStorageAccountKey
```

Anote el nombre y la clave de la cuenta de almacenamiento de blobs. La aplicación de ejemplo usa esta configuración para conectarse a la cuenta de almacenamiento y cargar las imágenes. 

## <a name="create-an-app-service-plan"></a>Creación de un plan de App Service

Un [plan de App Service](../../app-service/overview-hosting-plans.md) especifica la ubicación, el tamaño y las características de la granja de servidores web que hospeda la aplicación.

Cree un plan de App Service con el comando [az appservice plan create](/cli/azure/appservice/plan).

En el siguiente ejemplo se crea un plan de App Service denominado `myAppServicePlan` con el plan de tarifa **Gratis**:

```azurecli-interactive
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku Free
```

## <a name="create-a-web-app"></a>Creación de una aplicación web

La aplicación web proporciona un espacio de hospedaje para el código de la aplicación de ejemplo que se implementó desde el repositorio de ejemplo de GitHub. Cree una [aplicación web](../../app-service/overview.md) en el plan de App Service `myAppServicePlan` con el comando [az webapp create](/cli/azure/webapp).  

En el siguiente comando, reemplace `<web_app>` por un nombre único. Los caracteres válidos son `a-z`, `0-9` y `-`. Si el valor de `<web_app>` no es único, recibirá el mensaje de error: Ya existe un _sitio web con el nombre especificado `<web_app>`._ La dirección URL predeterminada de la aplicación web es `https://<web_app>.azurewebsites.net`.  

```azurecli-interactive
webapp=<web_app>

az webapp create --name $webapp --resource-group myResourceGroup --plan myAppServicePlan
```

## <a name="deploy-the-sample-app-from-the-github-repository"></a>Implementación de la aplicación de ejemplo desde el repositorio de GitHub

# <a name="nettabdotnet"></a>[\.NET](#tab/dotnet)

App Service admite varias maneras de implementar contenido en una aplicación web. En este tutorial, se implementa la aplicación web desde un [repositorio de ejemplo público de GitHub](https://github.com/Azure-Samples/storage-blob-upload-from-webapp). Configure la implementación de GitHub local en la aplicación web con el comando [az webapp deployment source config](/cli/azure/webapp/deployment/source).

El proyecto de ejemplo contiene una aplicación [MVC de ASP.NET](https://www.asp.net/mvc). La aplicación acepta una imagen, la guarda en una cuenta de almacenamiento y muestra imágenes de un contenedor de miniaturas. La aplicación web usa los espacios de nombres [Microsoft.WindowsAzure.Storage](/dotnet/api/overview/azure/storage), [Microsoft.WindowsAzure.Storage.Blob](/dotnet/api/microsoft.azure.storage.blob) y Microsoft.WindowsAzure.Storage.Auth de la biblioteca de cliente de Azure Storage para interactuar con Azure Storage.

```azurecli-interactive
az webapp deployment source config --name $webapp \
--resource-group myResourceGroup --branch master --manual-integration \
--repo-url https://github.com/Azure-Samples/storage-blob-upload-from-webapp
```

# <a name="nodejs-v2-sdktabnodejs"></a>[Node.js V2 SDK](#tab/nodejs)
App Service admite varias maneras de implementar contenido en una aplicación web. En este tutorial, se implementa la aplicación web desde un [repositorio de ejemplo público de GitHub](https://github.com/Azure-Samples/storage-blob-upload-from-webapp-node). Configure la implementación de GitHub local en la aplicación web con el comando [az webapp deployment source config](/cli/azure/webapp/deployment/source). 

```azurecli-interactive
az webapp deployment source config --name $webapp \
--resource-group myResourceGroup --branch master --manual-integration \
--repo-url https://github.com/Azure-Samples/storage-blob-upload-from-webapp-node
```

# <a name="nodejs-v10-sdktabnodejsv10"></a>[Node.js V10 SDK](#tab/nodejsv10)
App Service admite varias maneras de implementar contenido en una aplicación web. En este tutorial, se implementa la aplicación web desde un [repositorio de ejemplo público de GitHub](https://github.com/Azure-Samples/storage-blob-upload-from-webapp-node-v10). Configure la implementación de GitHub local en la aplicación web con el comando [az webapp deployment source config](/cli/azure/webapp/deployment/source). 

```azurecli-interactive
az webapp deployment source config --name $webapp \
--resource-group myResourceGroup --branch master --manual-integration \
--repo-url https://github.com/Azure-Samples/storage-blob-upload-from-webapp-node-v10
```

---

## <a name="configure-web-app-settings"></a>Configurar aplicaciones web

# <a name="nettabdotnet"></a>[\.NET](#tab/dotnet)

La aplicación web de ejemplo usa la [biblioteca de cliente de Azure Storage](/dotnet/api/overview/azure/storage?view=azure-dotnet) para solicitar tokens de acceso, que se usan para cargar imágenes. Las credenciales de la cuenta de almacenamiento que usa el SDK de Storage se establecen en la configuración de aplicación de la aplicación web. Agregue la configuración de aplicación a la aplicación implementada con el comando [az webapp config appsettings set](/cli/azure/webapp/config/appsettings).

```azurecli-interactive
az webapp config appsettings set --name $webapp --resource-group myResourceGroup \
--settings AzureStorageConfig__AccountName=$blobStorageAccount \
AzureStorageConfig__ImageContainer=images  \
AzureStorageConfig__ThumbnailContainer=thumbnails \
AzureStorageConfig__AccountKey=$blobStorageAccountKey  
```

# <a name="nodejs-v2-sdktabnodejs"></a>[Node.js V2 SDK](#tab/nodejs)

La aplicación web de ejemplo usa la [biblioteca de cliente de Azure Storage](https://docs.microsoft.com/javascript/api/azure-storage) para solicitar tokens de acceso, que se usan para cargar imágenes. Las credenciales de la cuenta de almacenamiento que usa el SDK de Storage se establecen en la configuración de aplicación de la aplicación web. Agregue la configuración de aplicación a la aplicación implementada con el comando [az webapp config appsettings set](/cli/azure/webapp/config/appsettings).

```azurecli-interactive
storageConnectionString=$(az storage account show-connection-string --resource-group $resourceGroupName \
--name $blobStorageAccount --query connectionString --output tsv)

az webapp config appsettings set --name $webapp --resource-group myResourceGroup \
--settings AzureStorageConfig__ImageContainer=images  \
AzureStorageConfig__ThumbnailContainer=thumbnails \
AzureStorageConfig__AccountName=$blobStorageAccount \
AzureStorageConfig__AccountKey=$blobStorageAccountKey \
AZURE_STORAGE_CONNECTION_STRING=$storageConnectionString
```

# <a name="nodejs-v10-sdktabnodejsv10"></a>[Node.js V10 SDK](#tab/nodejsv10)

La aplicación web de ejemplo usa la [biblioteca de cliente de Azure Storage](https://github.com/Azure/azure-storage-js) para solicitar tokens de acceso, que se usan para cargar imágenes. Las credenciales de la cuenta de almacenamiento que usa el SDK de Storage se establecen en la configuración de aplicación de la aplicación web. Agregue la configuración de aplicación a la aplicación implementada con el comando [az webapp config appsettings set](/cli/azure/webapp/config/appsettings).

```azurecli-interactive
az webapp config appsettings set --name $webapp --resource-group myResourceGroup \
--settings AZURE_STORAGE_ACCOUNT_NAME=$blobStorageAccount \
AZURE_STORAGE_ACCOUNT_ACCESS_KEY=$blobStorageAccountKey
```

---

Una vez que la aplicación web está implementada y configurada, puede probar la funcionalidad de carga de imágenes en la aplicación.

## <a name="upload-an-image"></a>Carga de una imagen

Para probar la aplicación web, vaya a la dirección URL de la aplicación publicada. La dirección URL predeterminada de la aplicación web es `https://<web_app>.azurewebsites.net`.

# <a name="nettabdotnet"></a>[\.NET](#tab/dotnet)

Seleccione la región **Upload photos** (Cargar fotos) para seleccionar y cargar un archivo o arrastre y coloque un archivo en dicha región. La imagen desaparece si se carga correctamente. La sección **Miniaturas generadas** permanecerá vacía hasta que la probemos más adelante en este tema.

![Aplicación ImageResizer](media/storage-upload-process-images/figure1.png)

En el código de ejemplo, la tarea `UploadFiletoStorage` del archivo *Storagehelper.cs* se usa para cargar las imágenes en el contenedor de *imágenes* de la cuenta de almacenamiento mediante el método [UploadFromStreamAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblockblob.uploadfromstreamasync?view=azure-dotnet). El siguiente código de ejemplo contiene la tarea `UploadFiletoStorage`.

```csharp
public static async Task<bool> UploadFileToStorage(Stream fileStream, string fileName, AzureStorageConfig _storageConfig)
{
    // Create storagecredentials object by reading the values from the configuration (appsettings.json)
    StorageCredentials storageCredentials = new StorageCredentials(_storageConfig.AccountName, _storageConfig.AccountKey);

    // Create cloudstorage account by passing the storagecredentials
    CloudStorageAccount storageAccount = new CloudStorageAccount(storageCredentials, true);

    // Create the blob client.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Get reference to the blob container by passing the name by reading the value from the configuration (appsettings.json)
    CloudBlobContainer container = blobClient.GetContainerReference(_storageConfig.ImageContainer);

    // Get the reference to the block blob from the container
    CloudBlockBlob blockBlob = container.GetBlockBlobReference(fileName);

    // Upload the file
    await blockBlob.UploadFromStreamAsync(fileStream);

    return await Task.FromResult(true);
}
```

En la tarea anterior se usan las clases y los métodos siguientes:

|Clase  |Método  |
|---------|---------|
|[StorageCredentials](/dotnet/api/microsoft.azure.cosmos.table.storagecredentials)     |         |
|[CloudStorageAccount](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount)    |  [CreateCloudBlobClient](/dotnet/api/microsoft.azure.storage.blob.blobaccountextensions.createcloudblobclient)       |
|[CloudBlobClient](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient)     |[GetContainerReference](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.getcontainerreference)         |
|[CloudBlobContainer](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer)    | [GetBlockBlobReference](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.getblockblobreference)        |
|[CloudBlockBlob](/dotnet/api/microsoft.azure.storage.blob.cloudblockblob)     | [UploadFromStreamAsync](/dotnet/api/microsoft.azure.storage.file.cloudfile.uploadfromstreamasync)        |

# <a name="nodejs-v2-sdktabnodejs"></a>[Node.js V2 SDK](#tab/nodejs)

Seleccione **Elegir archivo** para seleccionar un archivo y, a continuación, haga clic en **Cargar imagen**. La sección **Miniaturas generadas** permanecerá vacía hasta que la probemos más adelante en este tema. 

![Aplicación de carga de imágenes](media/storage-upload-process-images/upload-app-nodejs.png)

En el código de ejemplo, la ruta `post` es responsable de cargar la imagen en un contenedor de blobs. La ruta usa los módulos para ayudar en el proceso de la carga:

- [multer](https://github.com/expressjs/multer) implementa la estrategia de carga para el controlador de rutas.
- [into-stream](https://github.com/sindresorhus/into-stream) convierte el búfer en un flujo de datos según lo requiere [createBlockBlobFromStream].(https://azure.github.io/azure-sdk-for-node/azure-storage-legacy/latest/BlobService.html)

A medida que el archivo se envía a la ruta, el contenido del archivo permanece en la memoria hasta que el archivo se carga en el contenedor de blobs.

> [!IMPORTANT]
> La carga de archivos muy grandes en la memoria puede tener un efecto negativo en el rendimiento de la aplicación web. Si espera que los usuarios publiquen archivos de gran tamaño, es posible que desee considerar el almacenamiento provisional de los archivos en el sistema de archivos del servidor web y, después, programar las subidas al almacenamiento de blobs. Cuando los archivos estén en el almacenamiento de blobs, puede eliminarlos del sistema de archivos del servidor.

```javascript
const
      express = require('express')
    , router = express.Router()

    , multer = require('multer')
    , inMemoryStorage = multer.memoryStorage()
    , uploadStrategy = multer({ storage: inMemoryStorage }).single('image')

    , azureStorage = require('azure-storage')
    , blobService = azureStorage.createBlobService()

    , getStream = require('into-stream')
    , containerName = 'images'
;

const handleError = (err, res) => {
    res.status(500);
    res.render('error', { error: err });
};

const getBlobName = originalName => {
    const identifier = Math.random().toString().replace(/0\./, ''); // remove "0." from start of string
    return `${originalName}-${identifier}`;
};

router.post('/', uploadStrategy, (req, res) => {

    const
          blobName = getBlobName(req.file.originalname)
        , stream = getStream(req.file.buffer)
        , streamLength = req.file.buffer.length
    ;

    blobService.createBlockBlobFromStream(containerName, blobName, stream, streamLength, err => {

        if(err) {
            handleError(err);
            return;
        }

        res.render('success', { 
            message: 'File uploaded to Azure Blob storage.' 
        });
    });
});
```

# <a name="nodejs-v10-sdktabnodejsv10"></a>[Node.js V10 SDK](#tab/nodejsv10)

Seleccione **Elegir archivo** para seleccionar un archivo y, a continuación, haga clic en **Cargar imagen**. La sección **Miniaturas generadas** permanecerá vacía hasta que la probemos más adelante en este tema. 

![Aplicación de carga de imágenes](media/storage-upload-process-images/upload-app-nodejs.png)

En el código de ejemplo, la ruta `post` es responsable de cargar la imagen en un contenedor de blobs. La ruta usa los módulos para ayudar en el proceso de la carga:

- [multer](https://github.com/expressjs/multer) implementa la estrategia de carga para el controlador de rutas.
- [into-stream](https://github.com/sindresorhus/into-stream) convierte el búfer en un flujo de datos según lo requiere [createBlockBlobFromStream](https://azure.github.io/azure-sdk-for-node/azure-storage-legacy/latest/BlobService.html).

A medida que el archivo se envía a la ruta, el contenido del archivo permanece en la memoria hasta que el archivo se carga en el contenedor de blobs.

> [!IMPORTANT]
> La carga de archivos muy grandes en la memoria puede tener un efecto negativo en el rendimiento de la aplicación web. Si espera que los usuarios publiquen archivos de gran tamaño, es posible que desee considerar el almacenamiento provisional de los archivos en el sistema de archivos del servidor web y, después, programar las subidas al almacenamiento de blobs. Cuando los archivos estén en el almacenamiento de blobs, puede eliminarlos del sistema de archivos del servidor.

```javascript
const {
  Aborter,
  BlobURL,
  BlockBlobURL,
  ContainerURL,
  ServiceURL,
  StorageURL,
  SharedKeyCredential,
  uploadStreamToBlockBlob
} = require('@azure/storage-blob');

const express = require('express');
const router = express.Router();
const multer = require('multer');
const inMemoryStorage = multer.memoryStorage();
const uploadStrategy = multer({ storage: inMemoryStorage }).single('image');
const getStream = require('into-stream');
const containerName = 'images';
const ONE_MEGABYTE = 1024 * 1024;
const uploadOptions = { bufferSize: 4 * ONE_MEGABYTE, maxBuffers: 20 };
const ONE_MINUTE = 60 * 1000;
const aborter = Aborter.timeout(30 * ONE_MINUTE);

const sharedKeyCredential = new SharedKeyCredential(
  process.env.AZURE_STORAGE_ACCOUNT_NAME,
  process.env.AZURE_STORAGE_ACCOUNT_ACCESS_KEY);
const pipeline = StorageURL.newPipeline(sharedKeyCredential);
const serviceURL = new ServiceURL(
  `https://${process.env.AZURE_STORAGE_ACCOUNT_NAME}.blob.core.windows.net`,
  pipeline
);

const getBlobName = originalName => {
  // Use a random number to generate a unique file name, 
  // removing "0." from the start of the string.
  const identifier = Math.random().toString().replace(/0\./, ''); 
  return `${identifier}-${originalName}`;
};

router.post('/', uploadStrategy, async (req, res) => {

    const blobName = getBlobName(req.file.originalname);
    const stream = getStream(req.file.buffer);
    const containerURL = ContainerURL.fromServiceURL(serviceURL, containerName);
    const blobURL = BlobURL.fromContainerURL(containerURL, blobName);
    const blockBlobURL = BlockBlobURL.fromBlobURL(blobURL);

    try {
      
      await uploadStreamToBlockBlob(aborter, stream,
        blockBlobURL, uploadOptions.bufferSize, uploadOptions.maxBuffers);

      res.render('success', { message: 'File uploaded to Azure Blob storage.' });   

    } catch (err) {

      res.render('error', { message: 'Something went wrong.' });

    }
});
```
---

## <a name="verify-the-image-is-shown-in-the-storage-account"></a>Comprobación de que la imagen se muestra en la cuenta de almacenamiento

Inicie sesión en el [Azure Portal](https://portal.azure.com). En el menú izquierdo, seleccione **Cuentas de almacenamiento** y seleccione el nombre de la cuenta de almacenamiento. En **Blob Service**, seleccione **Blobs** y, a continuación, seleccione el contenedor de **imágenes**.

Compruebe que la imagen se muestra en el contenedor.

![Vista del contenedor de imágenes](media/storage-upload-process-images/figure13.png)

## <a name="test-thumbnail-viewing"></a>Prueba de la vista de miniaturas

Para probar la vista de miniaturas, cargará una imagen en el contenedor de **miniaturas** para comprobar que la aplicación puede leer el contenedor de **miniaturas**.

Inicie sesión en el [Azure Portal](https://portal.azure.com). En el menú izquierdo, seleccione **Cuentas de almacenamiento** y seleccione el nombre de la cuenta de almacenamiento. En **Blob Service**, seleccione **Blobs** y, a continuación, seleccione el contenedor de **miniaturas**. Seleccione **Cargar** para abrir el panel **Cargar blob**.

Elija un archivo mediante el selector de archivos y seleccione **Cargar**.

Vuelva a la aplicación para comprobar que la imagen cargada en el contenedor **thumbnails** está visible.

# <a name="nettabdotnet"></a>[\.NET](#tab/dotnet)
![Vista del contenedor de imágenes](media/storage-upload-process-images/figure2.png)

# <a name="nodejs-v2-sdktabnodejs"></a>[Node.js V2 SDK](#tab/nodejs)
![Vista del contenedor de imágenes](media/storage-upload-process-images/upload-app-nodejs-thumb.png)

# <a name="nodejs-v10-sdktabnodejsv10"></a>[Node.js V10 SDK](#tab/nodejsv10)
![Vista del contenedor de imágenes](media/storage-upload-process-images/upload-app-nodejs-thumb.png)

---

En la segunda parte de la serie, automatizará la creación de imágenes en miniatura para que no tenga necesidad de esta imagen. En el contenedor **thumbnails** de Azure Portal, seleccione la imagen que descargó y elija **Eliminar** para eliminarla. 

Puede habilitar CDN para almacenar en caché el contenido de la cuenta de Azure Storage. Para más información sobre cómo habilitar la red CDN con la cuenta de Azure Storage, consulte [Integración de una cuenta de Azure Storage con Azure CDN](../../cdn/cdn-create-a-storage-account-with-cdn.md).

## <a name="next-steps"></a>Pasos siguientes

En la primera parte de la serie, aprendió a configurar una aplicación web para interactuar con el almacenamiento.

Siga con la parte dos de la serie para aprender a usar Event Grid para desencadenar una función de Azure que cambie el tamaño de una imagen.

> [!div class="nextstepaction"]
> [Uso de Event Grid para desencadenar una función de Azure Function que cambie de tamaño una imagen cargada](../../event-grid/resize-images-on-storage-blob-upload-event.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
