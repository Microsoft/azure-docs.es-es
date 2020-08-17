---
title: Ejemplos de Azure Storage con JavaScript | Microsoft Docs
description: Consulte, descargue y ejecute código de ejemplo y aplicaciones para Azure Storage. Descubra ejemplos introductorios de blobs, colas, tablas y archivos, usando las bibliotecas de cliente de almacenamiento de JavaScript y Node.js.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 02/19/2020
ms.service: storage
ms.subservice: common
ms.topic: sample
ms.custom: devx-track-javascript
ms.openlocfilehash: 466a1318dbe0a9299eebb9259ef8e0fe9bbbc0e7
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87433081"
---
# <a name="azure-storage-samples-using-v12-javascript-client-libraries"></a>Ejemplos de Azure Storage mediante las bibliotecas de cliente de JavaScript v12

En la tabla siguiente se proporciona información general sobre el repositorio de ejemplos y los escenarios que abarca cada ejemplo. Haga clic en los vínculos para ver el código de ejemplo correspondiente en GitHub.

> [!NOTE]
> En estos ejemplos se usa la biblioteca JavaScript v12 de Azure Storage más reciente. Para ver el código heredado de la versión v11, consulte [Introducción a Azure Blob service en Node.js](https://github.com/Azure-Samples/storage-blob-node-getting-started) en el repositorio de GitHub.

## <a name="blob-samples"></a>Ejemplos de blob

### <a name="authentication"></a>Authentication

:::row:::
   :::column span="":::
      [Autenticación mediante una cadena de conexión.](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/withConnString.js#L14)
   :::column-end:::
   :::column span="":::
      [Autenticación mediante una cadena de conexión de SAS](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/withConnString.js#L14)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Autenticación mediante una credencial de clave compartida](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/sharedKeyCred.js#L5)
   :::column-end:::
   :::column span="":::
      [Autenticación mediante AnonymousCredential](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/anonymousCred.js#L18)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Autenticación mediante Azure Active Directory](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/azureAdAuth.js#L47)
   :::column-end:::
   :::column span="":::
      [Autenticación mediante un proxy](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/proxyAuth.js#L28)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Conexión mediante una canalización personalizada](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/customPipeline.js#L26)
   :::column-end:::
:::row-end:::

### <a name="blob-service"></a>Blob service

:::row:::
   :::column span="2":::
      [Creación de un cliente de Blob service mediante una dirección URL de SAS](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/advanced.js#L39)
   :::column-end:::
:::row-end:::

### <a name="container"></a>Contenedor

:::row:::
   :::column span="":::
      [Creación de un contenedor](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/basic.js#L53)
   :::column-end:::
   :::column span="":::
      [Creación de un contenedor mediante una credencial de clave compartida](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/readingSnapshot.js#L38)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Enumeración de contenedores](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/basic.js#L48)
   :::column-end:::
   :::column span="":::
      [Enumeración de contenedores mediante un iterador](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/iterators-containers.js#L28)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Enumeración de contenedores por página](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/iterators-containers.js#L53)
   :::column-end:::
   :::column span="":::
      [Eliminación de un contenedor](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/basic.js#L82)
   :::column-end:::
:::row-end:::

### <a name="blob"></a>Blob

:::row:::
   :::column span="":::
      [Creación de un blob](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/basic.js#L60)
   :::column-end:::
   :::column span="":::
      [Enumeración de blobs](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/basic.js#L67)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Descarga de un blob](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/basic.js#L73)
   :::column-end:::
   :::column span="":::
      [Enumeración de blobs mediante un iterador](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/iterators-blobs.js#L41)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Enumeración de blobs por página](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/iterators-blobs.js#L66)
   :::column-end:::
   :::column span="":::
      [Enumeración de blobs por jerarquía](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/iterators-blobs-hierarchy.js#L70)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Enumeración de blobs sin usar await](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/iterators-without-await.js#L42)
   :::column-end:::
   :::column span="":::
      [Creación de una instantánea de un blob](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/readingSnapshot.js#L56)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Descarga de una instantánea de un blob](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/readingSnapshot.js#L61)
   :::column-end:::
   :::column span="":::
      [Carga de un flujo en un blob en paralelo](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/advanced.js#L74)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Descarga de un blob en bloques en paralelo](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/advanced.js#L99)
   :::column-end:::
   :::column span="":::
      [Establecimiento del nivel de acceso en un blob](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/advanced.js#L118)
   :::column-end:::
:::row-end:::

### <a name="troubleshooting"></a>Solución de problemas

:::row:::
   :::column span="2":::
      [Desencadenamiento de un error recuperable mediante un cliente de contenedor](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-blob/samples/javascript/errorsAndResponses.js#L33)
   :::column-end:::
:::row-end:::


## <a name="data-lake-storage-gen2-samples"></a>Ejemplos de Data Lake Storage Gen2

:::row:::
   :::column span="":::
      [Creación de un cliente del servicio de Data Lake](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-datalake/samples/javascript/basic.js#L36)
   :::column-end:::
   :::column span="":::
      [Creación de un sistema de archivos](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-datalake/samples/javascript/basic.js#L47)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Enumeración de sistemas de archivos](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-datalake/samples/javascript/basic.js#L42)
   :::column-end:::
   :::column span="":::
      [Creación de un archivo](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-datalake/samples/javascript/basic.js#L54)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Enumeración de las rutas de acceso en un sistema de archivos](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-datalake/samples/javascript/basic.js#L63)
   :::column-end:::
   :::column span="":::
      [Descarga de un archivo](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-datalake/samples/javascript/basic.js#L69)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Eliminación de un sistema de archivos](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-datalake/samples/javascript/basic.js#L78)
   :::column-end:::
:::row-end:::

## <a name="azure-files-samples"></a>Ejemplos de Azure Files

### <a name="authentication"></a>Authentication

:::row:::
   :::column span="":::
      [Autenticación mediante una cadena de conexión.](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/withConnString.js)
   :::column-end:::
   :::column span="":::
      [Autenticación mediante una credencial de clave compartida](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/sharedKeyCred.js)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Autenticación mediante AnonymousCredential](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/anonymousCred.js)
   :::column-end:::
   :::column span="":::
      [Conexión mediante una canalización personalizada](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/customPipeline.js)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Conexión mediante un proxy](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/proxyAuth.js)
   :::column-end:::
:::row-end:::

### <a name="share"></a>Compartir

:::row:::
   :::column span="":::
      [Creación de un recurso compartido](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/advanced.js#L48)
   :::column-end:::
   :::column span="":::
      [Enumeración de recursos compartidos](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/iterators-shares.js#L27)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Enumeración de recursos compartidos por página](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/iterators-shares.js#L51)
   :::column-end:::
   :::column span="":::
      [Eliminación de un recurso compartido](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/advanced.js#L104)
   :::column-end:::
:::row-end:::

### <a name="directory"></a>Directorio

:::row:::
   :::column span="":::
      [Creación de un directorio](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/advanced.js#L54)
   :::column-end:::
   :::column span="":::
      [Enumeración de archivos y directorios](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/iterators-files-and-directories.js#L56)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Enumeración de archivos y directorios por página](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/iterators-files-and-directories.js#L90)
   :::column-end:::
:::row-end:::

### <a name="file"></a>Archivo

:::row:::
   :::column span="":::
      [Carga de un archivo en paralelo](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/advanced.js#L65)
   :::column-end:::
   :::column span="":::
      [Carga de un flujo legible en paralelo](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/advanced.js#L74)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Descarga de un archivo en paralelo](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/advanced.js#L93)
   :::column-end:::
   :::column span="":::
      [Enumeración de identificadores de archivos](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/iterators-handles.js#L43)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="2":::
      [Enumeración de identificadores de archivo por página](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-file-share/samples/javascript/iterators-handles.js#L79)
   :::column-end:::
:::row-end:::

## <a name="queue-samples"></a>Ejemplos de cola

### <a name="authentication"></a>Authentication

:::row:::
   :::column span="":::
      [Autenticación mediante una cadena de conexión.](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/withConnString.js)
   :::column-end:::
   :::column span="":::
      [Autenticación mediante una credencial de clave compartida](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/sharedKeyCred.js)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Autenticación mediante AnonymousCredential](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/anonymousCred.js)
   :::column-end:::
   :::column span="":::
      [Conexión mediante una canalización personalizada](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/customPipeline.js)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Conexión mediante un proxy](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/proxyAuth.js)
   :::column-end:::
   :::column span="":::
      [Autenticación mediante Azure Active Directory](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/azureAdAuth.js)
   :::column-end:::
:::row-end:::

### <a name="queue-service"></a>Queue service

:::row:::
   :::column span="2":::
      [Creación de un cliente de Queue service](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/basic.js#L42)
   :::column-end:::
:::row-end:::

### <a name="queue"></a>Cola

:::row:::
   :::column span="":::
      [Creación de una cola](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/basic.js#L54)
   :::column-end:::
   :::column span="":::
      [Enumeración de las colas](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/iterators.js#L27)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Enumeración de colas por página](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/iterators.js#L51)
   :::column-end:::
   :::column span="":::
      [Eliminación de una cola](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/basic.js#L89)
   :::column-end:::
:::row-end:::

### <a name="message"></a>Message

:::row:::
   :::column span="":::
      [Envío de un mensaje en una cola](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/basic.js#L62)
   :::column-end:::
   :::column span="":::
      [Inspección de mensajes](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/basic.js#L68)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Recepción de mensajes](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/basic.js#L76)
   :::column-end:::
   :::column span="":::
      [Eliminación de mensajes](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/storage/storage-queue/samples/javascript/basic.js#L80)
   :::column-end:::
:::row-end:::

## <a name="table-samples-v11"></a>Ejemplos de tabla (v11)

:::row:::
   :::column span="":::
      [Entidades por lote](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/basic.js#L87)
   :::column-end:::
   :::column span="":::
      [Creación de tablas](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/basic.js#L41)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Eliminación de una entidad o tabla](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/basic.js#L67)
   :::column-end:::
   :::column span="":::
      [Inserción, combinación o sustitución de entidades](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/basic.js#L49)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Enumeración de tablas](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/advanced.js#L63)
   :::column-end:::
   :::column span="":::
      [Consulta de entidades](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/basic.js#L59)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Consulta de tablas](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/basic.js#L140)
   :::column-end:::
   :::column span="":::
      [Consulta por rango](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/basic.js#L102)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Firma de acceso compartido (SAS)](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/advanced.js#L87)
   :::column-end:::
   :::column span="":::
      [ACL de tabla](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/advanced.js#L255)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Reglas del uso compartido de recursos entre orígenes (CORS) de tabla](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/advanced.js#L149)
   :::column-end:::
   :::column span="":::
      [Propiedades de tabla](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/advanced.js#L188)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      [Estadísticas de tabla](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/advanced.js#L243)
   :::column-end:::
   :::column span="":::
      [Actualización de entidades](https://github.com/Azure-Samples/storage-table-node-getting-started/blob/master/basic.js#L49)
   :::column-end:::
:::row-end:::

## <a name="azure-code-sample-libraries"></a>Bibliotecas de ejemplo de código de Azure

Para ver todas las bibliotecas de ejemplo de JavaScript, vaya a:

* [Ejemplos de código de blobs de Azure](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-blob/samples/javascript)
* [Ejemplos de código de Azure Data Lake](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-file-datalake/samples/javascript)
* [Ejemplos de código de Azure Files](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-file-share/samples/javascript)
* [Ejemplos de código de cola de Azure](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-queue/samples/javascript)

Puede explorar y clonar el repositorio de GitHub para cada biblioteca.

## <a name="getting-started-guides"></a>Guías de introducción

Consulte las guías siguientes si busca instrucciones sobre cómo instalar las bibliotecas de cliente de Azure Storage y cómo empezar a usarlas.

* [Introducción a Azure Blob Storage en JavaScript](../blobs/storage-quickstart-blobs-nodejs.md)
* [Introducción a Azure Queue Storage en JavaScript](../queues/storage-quickstart-queues-nodejs.md)
* [Introducción a Azure Table Storage en JavaScript](../../cosmos-db/table-storage-how-to-use-nodejs.md)

## <a name="next-steps"></a>Pasos siguientes

Para información sobre ejemplos para otros lenguajes:

* .NET: [Azure Storage samples using .NET](storage-samples-dotnet.md) (Ejemplos de Azure Storage con .NET)
* Java: [Azure Storage samples using Java](storage-samples-java.md) (Ejemplos de Azure Storage con Java)
* Python: [Ejemplos de Azure Storage con Python](storage-samples-python.md)
* El resto de lenguajes: [Ejemplos de Azure Storage](storage-samples.md)
