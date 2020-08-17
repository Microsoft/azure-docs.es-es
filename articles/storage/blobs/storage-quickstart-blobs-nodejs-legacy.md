---
title: 'Inicio rápido: Biblioteca cliente de Azure Blob Storage v10 para JavaScript'
description: Creación, carga y eliminación de blobs y contenedores de Node.js con la biblioteca cliente de Azure Storage v10 para JavaScript
author: mhopkins-msft
ms.author: mhopkins
ms.date: 07/24/2020
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.custom: devx-track-javascript
ms.openlocfilehash: cb1e1baa63e5a38a7f3db251c6c6d551e999befc
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87431060"
---
# <a name="quickstart-manage-blobs-with-javascript-v10-sdk-in-nodejs"></a>Inicio rápido: Administración de blobs con el SDK de JavaScript v10 en Node.js

En este inicio rápido, aprenderá a administrar blobs mediante Node.js. Los blobs son objetos que pueden contener grandes cantidades de texto o datos binarios, como imágenes, documentos, medios de transmisión y datos de archivo. Podrá cargar, descargar, enumerar y eliminar blobs, y podrá administrar los contenedores.

> [!NOTE]
> Esta guía de inicio rápido usa una versión heredada de la biblioteca cliente de Azure Blob Storage. Para empezar a trabajar con la versión más reciente, consulte [Inicio rápido: Administración de blobs con el SDK de JavaScript v12 en Node.js](storage-quickstart-blobs-nodejs.md).

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- Una cuenta de Azure Storage. [Crear una cuenta de almacenamiento](../common/storage-account-create.md).
- [Node.js](https://nodejs.org/en/download/).

## <a name="download-the-sample-application"></a>Descarga de la aplicación de ejemplo

La [aplicación de ejemplo](https://github.com/Azure-Samples/azure-storage-js-v10-quickstart.git) utilizada en esta guía de inicio rápido es una sencilla aplicación de consola de Node.js. Para comenzar, clone el repositorio en el equipo con el comando siguiente:

```bash
git clone https://github.com/Azure-Samples/azure-storage-js-v10-quickstart.git
```

A continuación, cambie las carpetas de la aplicación:

```bash
cd azure-storage-js-v10-quickstart
```

Ahora, abra la carpeta en su entorno de edición de código favorito.

## <a name="configure-your-storage-credentials"></a>Configuración de las credenciales de almacenamiento

Antes de ejecutar la aplicación, debe proporcionar las credenciales de seguridad de la cuenta de almacenamiento. El repositorio de ejemplo incluye un archivo llamado *.env.example*. Cambie el nombre de este archivo quitando la extensión *.example*, lo que dará como resultado un archivo llamado *.env*. Dentro del archivo *.env*, agregue el nombre de la cuenta y los valores de la clave de acceso después de las claves *AZURE_STORAGE_ACCOUNT_NAME* y *AZURE_STORAGE_ACCOUNT_ACCESS_KEY*.

## <a name="install-required-packages"></a>Instalación de los paquetes requeridos

En el directorio de la aplicación, ejecute *npm install* para instalar los paquetes necesarios para la aplicación.

```bash
npm install
```

## <a name="run-the-sample"></a>Ejecución del ejemplo

Ahora que las dependencias están instaladas, puede ejecutar el ejemplo si emite el siguiente comando:

```bash
npm start
```

La salida de la aplicación será parecida a la del ejemplo siguiente:

```bash
Container "demo" is created
Containers:
 - container-one
 - container-two
 - demo
Blob "quickstart.txt" is uploaded
Local file "./readme.md" is uploaded
Blobs in "demo" container:
 - quickstart.txt
 - readme-stream.md
 - readme.md
Blob downloaded blob content: "hello!"
Blob "quickstart.txt" is deleted
Container "demo" is deleted
Done
```

Si usa una cuenta de almacenamiento nueva para este inicio rápido, es posible que solo vea el contenedor de *demostración* en la lista en la etiqueta "*Contenedores:* ".

## <a name="understanding-the-code"></a>Descripción del código

El ejemplo comienza con la importación de un número de clases y funciones desde el espacio de nombres de Azure Blob Storage. Cada uno de los elementos importados se analiza en contexto a medida que se usan en el ejemplo.

```javascript
const {
    Aborter,
    BlobURL,
    BlockBlobURL,
    ContainerURL,
    ServiceURL,
    SharedKeyCredential,
    StorageURL,
    uploadStreamToBlockBlob
} = require('@azure/storage-blob');
```

Las credenciales se leen desde variables de entorno basadas en el contexto adecuado.

```javascript
if (process.env.NODE_ENV !== 'production') {
    require('dotenv').config();
}
```

El módulo *dotenv* carga variables de entorno al ejecutar la aplicación en local para la depuración. Los valores se definen en un archivo denominado *.env* y se cargan en el contexto de ejecución actual. En producción, la configuración del servidor proporciona estos valores y por eso este código se ejecuta solo cuando el script *no* se está ejecutando en un entorno de "producción".

El siguiente bloque de módulos se importa para ayudar a interactuar con el sistema de archivos.

```javascript
const fs = require('fs');
const path = require('path');
```

El propósito de los módulos es el siguiente: 

- *fs* es el módulo de Node.js nativo que se usa para trabajar con el sistema de archivos

- *path* es necesario para determinar la ruta de acceso absoluta del archivo y se usa al cargar un archivo en Blob Storage

A continuación, se leen los valores de las variables de entorno y se reservan en constantes.

```javascript
const STORAGE_ACCOUNT_NAME = process.env.AZURE_STORAGE_ACCOUNT_NAME;
const ACCOUNT_ACCESS_KEY = process.env.AZURE_STORAGE_ACCOUNT_ACCESS_KEY;
```
El siguiente conjunto de constantes ayuda a revelar la intención de los cálculos de tamaño de archivo durante las operaciones de carga.

```javascript
const ONE_MEGABYTE = 1024 * 1024;
const FOUR_MEGABYTES = 4 * ONE_MEGABYTE;
```

Se puede establecer que el tiempo de espera de las solicitudes que realice la API se agote después de un intervalo dado. La clase [Aborter](/javascript/api/%40azure/storage-blob/aborter?view=azure-node-legacy) es responsable de administrar el tiempo de espera de las solicitudes y, en este ejemplo, se usa la constante siguiente para definir los tiempos de espera.

```javascript
const ONE_MINUTE = 60 * 1000;
```

### <a name="calling-code"></a>Llamadas al código

Para admitir la sintaxis *async/await* de JavaScript, todo el código de llamada se encapsula en una función denominada *execute*. Luego se llama a *execute*, que se trata como una promesa.

```javascript
async function execute() {
    // commands... 
}

execute().then(() => console.log("Done")).catch((e) => console.log(e));
```

Todo el código siguiente se ejecuta dentro de la función execute, donde se encuentra el comentario `// commands...`.

En primer lugar, se declaran las variables correspondientes para asignar nombres, probar contenido y apuntar al archivo local que se va a cargar en Blob Storage.

```javascript
const containerName = "demo";
const blobName = "quickstart.txt";
const content = "hello!";
const localFilePath = "./readme.md";
```

Las credenciales de cuenta se usan para crear una canalización, que es la encargada de administrar cómo se envían solicitudes a la API REST. Las canalizaciones son seguras para subprocesos y especifican la lógica para las directivas de reintentos, el registro, las reglas de deserialización de respuestas HTTP, etc.

```javascript
const credentials = new SharedKeyCredential(STORAGE_ACCOUNT_NAME, ACCOUNT_ACCESS_KEY);
const pipeline = StorageURL.newPipeline(credentials);
const serviceURL = new ServiceURL(`https://${STORAGE_ACCOUNT_NAME}.blob.core.windows.net`, pipeline);
```

Las siguientes clases se utilizan en este bloque de código:

- La clase [SharedKeyCredential](/javascript/api/%40azure/storage-blob/sharedkeycredential?view=azure-node-legacy) es la encargada de encapsular las credenciales de la cuenta de almacenamiento para proporcionárselas a una canalización de solicitudes.

- La clase [StorageURL](/javascript/api/%40azure/storage-blob/storageurl?view=azure-node-legacy) es la que se ocupa de crear una nueva canalización.

- La clase [ServiceURL](/javascript/api/%40azure/storage-blob/serviceurl?view=azure-node-legacy) modela una dirección URL que se utiliza en la API REST. Las instancias de esta clase le permiten realizar acciones como, por ejemplo, mostrar los contenedores y proporcionar información de contexto para generar las direcciones URL de estos.

La instancia de *ServiceURL* se usa con instancias de [ContainerURL](/javascript/api/%40azure/storage-blob/containerurl?view=azure-node-legacy) y [BlockBlobURL](/javascript/api/%40azure/storage-blob/blockbloburl?view=azure-node-legacy) para administrar contenedores y blobs en su cuenta de almacenamiento.

```javascript
const containerURL = ContainerURL.fromServiceURL(serviceURL, containerName);
const blockBlobURL = BlockBlobURL.fromContainerURL(containerURL, blobName);
```

Las variables *containerURL* y *blockBlobURL* se reutilizan durante el ejemplo para que actúen sobre la cuenta de almacenamiento. 

En este momento, el contenedor no existe en la cuenta de almacenamiento. La instancia de *ContainerURL* representa una dirección URL sobre la que puede actuar. Mediante el uso de esta instancia, puede crear y eliminar el contenedor. La ubicación de este contenedor equivale a una ubicación como esta:

```bash
https://<ACCOUNT_NAME>.blob.core.windows.net/demo
```

La variable *blockBlobURL* se usa para administrar blobs individuales, lo que le permite cargar, descargar y eliminar contenido del blob. La dirección URL que se representa aquí es similar a esta ubicación:

```bash
https://<ACCOUNT_NAME>.blob.core.windows.net/demo/quickstart.txt
```

Al igual que con el contenedor, el blob en bloques no existe aún. La variable *blockBlobURL* se usa más adelante para crear el blob mediante la carga de contenido.

### <a name="using-the-aborter-class"></a>Uso de la clase Aborter

Se puede establecer que el tiempo de espera de las solicitudes que realice la API se agote después de un intervalo dado. La clase *Aborter* es responsable de administrar el tiempo de espera de las solicitudes. El código siguiente crea un contexto en el que a un conjunto de solicitudes se le dan 30 minutos para ejecutarse.

```javascript
const aborter = Aborter.timeout(30 * ONE_MINUTE);
```

La clase Aborter le proporciona control sobre las solicitudes permitiéndole:

- designar la cantidad de tiempo que se le proporciona a un lote de solicitudes
- designar cuánto tiempo tiene una solicitud individual para ejecutarse en el lote
- cancelar solicitudes
- usar el miembro estático *Aborter.none* para impedir que todas las solicitudes agoten el tiempo de espera a la vez

### <a name="create-a-container"></a>Crear un contenedor

Para crear un contenedor, se usa el método *create* de *ContainerURL*.

```javascript
await containerURL.create(aborter);
console.log(`Container: "${containerName}" is created`);
```

Como el nombre del contenedor se define cuando se llama a *ContainerURL.fromServiceURL(serviceURL, containerName)* , llamar al método *create* es todo lo que necesita para crear el contenedor.

### <a name="show-container-names"></a>Mostrar nombres de contenedor

Las cuentas pueden almacenar un gran número de contenedores. El código siguiente muestra cómo mostrar los contenedores de forma segmentada, lo que permite recorrer de un vistazo un gran número de contenedores. La función *showContainerNames* pasa instancias de *ServiceURL* y *Aborter*.

```javascript
console.log("Containers:");
await showContainerNames(serviceURL, aborter);
```

La función *showContainerNames* usa el método *listContainersSegment* para solicitar lotes de nombres de contenedor de la cuenta de almacenamiento.

```javascript
async function showContainerNames(aborter, serviceURL) {
    let marker = undefined;

    do {
        const listContainersResponse = await serviceURL.listContainersSegment(aborter, marker);
        marker = listContainersResponse.nextMarker;
        for(let container of listContainersResponse.containerItems) {
            console.log(` - ${ container.name }`);
        }
    } while (marker);
}
```

Cuando se devuelve la respuesta, los *containerItems* se recorren en iteración para registrar el nombre en la consola. 

### <a name="upload-text"></a>Cargar texto

Para cargar texto en el blob, use el método *upload*.

```javascript
await blockBlobURL.upload(aborter, content, content.length);
console.log(`Blob "${blobName}" is uploaded`);
```

Aquí el texto y su longitud se pasan al método.

### <a name="upload-a-local-file"></a>Carga de un archivo local

Para cargar un archivo local en el contenedor, necesita una dirección URL del contenedor y la ruta de acceso al archivo.

```javascript
await uploadLocalFile(aborter, containerURL, localFilePath);
console.log(`Local file "${localFilePath}" is uploaded`);
```

La función *uploadLocalFile* llama a la función *uploadFileToBlockBlob*, que toma la ruta de acceso al archivo y una instancia del destino del blob en bloques como argumentos.

```javascript
async function uploadLocalFile(aborter, containerURL, filePath) {

    filePath = path.resolve(filePath);

    const fileName = path.basename(filePath);
    const blockBlobURL = BlockBlobURL.fromContainerURL(containerURL, fileName);

    return await uploadFileToBlockBlob(aborter, filePath, blockBlobURL);
}
```

### <a name="upload-a-stream"></a>Carga de una secuencia

También se admite la carga de secuencias. En este ejemplo se abre un archivo local como una secuencia para pasar al método upload.

```javascript
await uploadStream(containerURL, localFilePath, aborter);
console.log(`Local file "${localFilePath}" is uploaded as a stream`);
```

La función *uploadStream* llama a *uploadStreamToBlockBlob* para cargar la secuencia en el contenedor de almacenamiento.

```javascript
async function uploadStream(aborter, containerURL, filePath) {
    filePath = path.resolve(filePath);

    const fileName = path.basename(filePath).replace('.md', '-stream.md');
    const blockBlobURL = BlockBlobURL.fromContainerURL(containerURL, fileName);

    const stream = fs.createReadStream(filePath, {
      highWaterMark: FOUR_MEGABYTES,
    });

    const uploadOptions = {
        bufferSize: FOUR_MEGABYTES,
        maxBuffers: 5,
    };

    return await uploadStreamToBlockBlob(
                    aborter, 
                    stream, 
                    blockBlobURL, 
                    uploadOptions.bufferSize, 
                    uploadOptions.maxBuffers);
}
```

Durante una carga, *uploadStreamToBlockBlob* asigna búferes a los datos de la memoria caché desde la secuencia en caso de que sea necesario un reintento. El valor *maxBuffers* designa el número máximo de búferes que se usa cuando cada búfer crea una solicitud de carga independiente. Un mayor número de búferes equivale a una mayor velocidad, pero a costa de un mayor uso de la memoria. La velocidad de carga se estanca cuando el número de búferes es lo suficientemente alto como para que el cuello de botella realice la transición a la red o al disco en lugar del cliente.

### <a name="show-blob-names"></a>Mostrar los nombres de blob

Igual que las cuentas pueden contener muchos contenedores, cada contenedor puede contener una gran cantidad de blobs. El acceso a todos los blobs de un contenedor está disponible mediante una instancia de la clase *ContainerURL*.

```javascript
console.log(`Blobs in "${containerName}" container:`);
await showBlobNames(aborter, containerURL);
```

La función *showBlobNames* llama a *listBlobFlatSegment* para solicitar lotes de blobs del contenedor.

```javascript
async function showBlobNames(aborter, containerURL) {
    let marker = undefined;

    do {
        const listBlobsResponse = await containerURL.listBlobFlatSegment(Aborter.none, marker);
        marker = listBlobsResponse.nextMarker;
        for (const blob of listBlobsResponse.segment.blobItems) {
            console.log(` - ${ blob.name }`);
        }
    } while (marker);
}
```

### <a name="download-a-blob"></a>Descarga de un blob

Una vez que se crea un blob, puede descargar el contenido mediante el método *download*.

```javascript
const downloadResponse = await blockBlobURL.download(aborter, 0);
const downloadedContent = await streamToString(downloadResponse.readableStreamBody);
console.log(`Downloaded blob content: "${downloadedContent}"`);
```

La respuesta se devuelve como una secuencia. En este ejemplo, la secuencia se convierte en una cadena mediante el uso de la siguiente función auxiliar *streamToString*.

```javascript
// A helper method used to read a Node.js readable stream into a string
async function streamToString(readableStream) {
    return new Promise((resolve, reject) => {
      const chunks = [];
      readableStream.on("data", data => {
        chunks.push(data.toString());
      });
      readableStream.on("end", () => {
        resolve(chunks.join(""));
      });
      readableStream.on("error", reject);
    });
}
```

### <a name="delete-a-blob"></a>Eliminar un blob

El método *delete* de una instancia de *BlockBlobURL* elimina un blob del contenedor.

```javascript
await blockBlobURL.delete(aborter)
console.log(`Block blob "${blobName}" is deleted`);
```

### <a name="delete-a-container"></a>Eliminación de un contenedor

El método *delete* de una instancia de *ContainerURL* elimina un contenedor de la cuenta de almacenamiento.

```javascript
await containerURL.delete(aborter);
console.log(`Container "${containerName}" is deleted`);
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Todos los datos escritos en la cuenta de almacenamiento se eliminan automáticamente al final del ejemplo de código. 

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido se muestra cómo administrar blobs y contenedores en Azure Blob Storage mediante Node.js. Para más información sobre cómo trabajar con este SDK, vaya al repositorio de GitHub.

> [!div class="nextstepaction"]
> [Repositorio del SDK de Azure Storage v10 para JavaScript](https://github.com/Azure/azure-storage-js)
> [Referencia de la API de JavaScript de Azure Storage](/javascript/api/overview/azure/storage-overview)
