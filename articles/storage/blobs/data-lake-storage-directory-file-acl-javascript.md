---
title: Uso de JavaScript con archivos y ACL en Azure Data Lake Storage Gen2
description: Use la biblioteca cliente de Azure Storage Data Lake para JavaScript con el fin de administrar directorios y listas de control de acceso (ACL) de archivos y directorios en cuentas de almacenamiento que tengan habilitado el espacio de nombres jerárquico (HNS).
author: normesta
ms.service: storage
ms.date: 03/20/2020
ms.author: normesta
ms.topic: how-to
ms.subservice: data-lake-storage-gen2
ms.reviewer: prishet
ms.custom: devx-track-javascript
ms.openlocfilehash: 648a47b451e8857c1cd84c2bcecdaae7fcaa065d
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87430534"
---
# <a name="use-javascript-to-manage-directories-files-and-acls-in-azure-data-lake-storage-gen2"></a>Uso de JavaScript para administrar directorios, archivos y ACL en Azure Data Lake Storage Gen2

En este artículo se muestra cómo usar JavaScript para crear y administrar directorios, archivos y permisos en cuentas de almacenamiento que tengan habilitado el espacio de nombres jerárquico (HNS). 

[Paquete (Administrador de paquetes de Node)](https://www.npmjs.com/package/@azure/storage-file-datalake) | [Ejemplos](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-file-datalake/samples) | [Enviar comentarios](https://github.com/Azure/azure-sdk-for-java/issues)

## <a name="prerequisites"></a>Requisitos previos

> [!div class="checklist"]
> * Suscripción a Azure. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
> * Una cuenta de almacenamiento que tenga habilitado el espacio de nombres jerárquico (HNS). Siga [estas](data-lake-storage-quickstart-create-account.md) instrucciones para crear uno.
> * Si utiliza este paquete en una aplicación de Node.js, necesitará Node.js 8.0.0 o una versión posterior.

## <a name="set-up-your-project"></a>Configurar su proyecto

Instale la biblioteca cliente de Data Lake para JavaScript; para ello, abra una ventana de terminal y, a continuación, escriba el siguiente comando.

```javascript
npm install @azure/storage-file-datalake
```

Importe el paquete `storage-file-datalake` colocando esta instrucción en la parte superior del archivo de código. 

```javascript
const AzureStorageDataLake = require("@azure/storage-file-datalake");
```

## <a name="connect-to-the-account"></a>Conexión con la cuenta 

Para usar los fragmentos de código de este artículo, tiene que crear una instancia de **DataLakeServiceClient** que represente la cuenta de almacenamiento. 

### <a name="connect-by-using-an-account-key"></a>Conexión con una clave de cuenta

Es la manera más sencilla de conectarse a una cuenta. 

En este ejemplo se crea una instancia de **DataLakeServiceClient** con una clave de cuenta.

```javascript

function GetDataLakeServiceClient(accountName, accountKey) {

  const sharedKeyCredential = 
     new StorageSharedKeyCredential(accountName, accountKey);
  
  const datalakeServiceClient = new DataLakeServiceClient(
      `https://${accountName}.dfs.core.windows.net`, sharedKeyCredential);

  return datalakeServiceClient;             
}      

```
> [!NOTE]
> Este método de autorización solo funciona para aplicaciones de Node.js. Si tiene previsto ejecutar el código en un explorador, se puede autorizar mediante Azure Active Directory (AD). 

### <a name="connect-by-using-azure-active-directory-ad"></a>Conexión con Azure Active Directory (AD)

Puede usar la [biblioteca cliente de Azure Identity para JS](https://www.npmjs.com/package/@azure/identity) para autenticar la aplicación con Azure AD.

En este ejemplo se crea una instancia de **DataLakeServiceClient** con un identificador de cliente, un secreto de cliente y un identificador de inquilino.  Para obtener estos valores, consulte [Adquisición de un token de Azure AD para la autorización de solicitudes desde una aplicación cliente](../common/storage-auth-aad-app.md).

```javascript
function GetDataLakeServiceClientAD(accountName, clientID, clientSecret, tenantID) {

  const credential = new ClientSecretCredential(tenantID, clientID, clientSecret);
  
  const datalakeServiceClient = new DataLakeServiceClient(
      `https://${accountName}.dfs.core.windows.net`, credential);

  return datalakeServiceClient;             
}
```

> [!NOTE]
> Para ver más ejemplos, consulte la documentación de [Biblioteca cliente de Azure Identity para JS](https://www.npmjs.com/package/@azure/identity).

## <a name="create-a-container"></a>Crear un contenedor

Un contenedor actúa como sistema de archivos para sus archivos. Puede crear uno mediante la obtención de una instancia de **FileSystemClient** y, a continuación, llamar al método **FileSystemClient.Create**.

En este ejemplo se crea un contenedor denominado `my-file-system`. 

```javascript
async function CreateFileSystem(datalakeServiceClient) {

  const fileSystemName = "my-file-system";
  
  const fileSystemClient = datalakeServiceClient.getFileSystemClient(fileSystemName);

  const createResponse = await fileSystemClient.create();
        
}
```

## <a name="create-a-directory"></a>Creación de un directorio

Cree una referencia de directorio obteniendo una instancia de **DirectoryClient** y, a continuación, llame al método **DirectoryClient.create**.

En este ejemplo se agrega un directorio denominado `my-directory` a un contenedor. 

```javascript
async function CreateDirectory(fileSystemClient) {
   
  const directoryClient = fileSystemClient.getDirectoryClient("my-directory");
  
  await directoryClient.create();

}
```

## <a name="rename-or-move-a-directory"></a>Cambio de nombre o traslado de un directorio

Cambie el nombre de un directorio o muévalo llamando al método **DirectoryClient.rename**. Pase la ruta de acceso del directorio que busca a un parámetro. 

En este ejemplo se cambia el nombre de un subdirectorio a `my-directory-renamed`.

```javascript
async function RenameDirectory(fileSystemClient) {

  const directoryClient = fileSystemClient.getDirectoryClient("my-directory"); 
  await directoryClient.move("my-directory-renamed");

}
```

En este ejemplo se mueve un directorio denominado `my-directory-renamed` a un subdirectorio de un directorio denominado `my-directory-2`. 

```javascript
async function MoveDirectory(fileSystemClient) {

  const directoryClient = fileSystemClient.getDirectoryClient("my-directory-renamed"); 
  await directoryClient.move("my-directory-2/my-directory-renamed");      

}
```

## <a name="delete-a-directory"></a>Eliminación de un directorio

Elimine un directorio llamando al método **DirectoryClient.delete**.

En este ejemplo se elimina un directorio denominado `my-directory`.   

```javascript
async function DeleteDirectory(fileSystemClient) {

  const directoryClient = fileSystemClient.getDirectoryClient("my-directory"); 
  await directoryClient.delete();

}
```

## <a name="manage-a-directory-acl"></a>Administración de una lista de control de acceso de directorio

En este ejemplo se obtiene y después se establece la ACL de un directorio denominado `my-directory`. En este ejemplo se concede al usuario propietario permisos de lectura, escritura y ejecución, permisos de solo lectura y ejecución al grupo propietario, y permisos de lectura y escritura al resto.

> [!NOTE]
> Si la aplicación autoriza el acceso mediante Azure Active Directory (Azure AD), asegúrese de que la entidad de seguridad que la aplicación usa para autorizar el acceso tiene asignado el [rol de propietario de datos de blobs de almacenamiento](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-blob-data-owner). Para más información sobre cómo se aplican los permisos de ACL y las consecuencias de cambiarlos, consulte [Control de acceso en Azure Data Lake Storage Gen2](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-access-control).

```javascript
async function ManageDirectoryACLs(fileSystemClient) {

    const directoryClient = fileSystemClient.getDirectoryClient("my-directory"); 
    const permissions = await directoryClient.getAccessControl();

    console.log(permissions.acl);

    const acl = [
    {
      accessControlType: "user",
      entityId: "",
      defaultScope: false,
      permissions: {
        read: true,
        write: true,
        execute: true
      }
    },
    {
      accessControlType: "group",
      entityId: "",
      defaultScope: false,
      permissions: {
        read: true,
        write: false,
        execute: true
      }
    },
    {
      accessControlType: "other",
      entityId: "",
      defaultScope: false,
      permissions: {
        read: true,
        write: true,
        execute: false
      }

    }

  ];

  await directoryClient.setAccessControl(acl);
}
```

También puede obtener y establecer la ACL del directorio raíz de un contenedor. Para obtener el directorio raíz, pase una cadena vacía (`/`) en el método **DataLakeFileSystemClient.getDirectoryClient**.

## <a name="upload-a-file-to-a-directory"></a>Carga de un archivo en un directorio

Primero, lea un archivo En este ejemplo se usa el módulo `fs` de Node.js. A continuación, cree una referencia de archivo en el directorio de destino creando una instancia de **FileClient** y, a continuación, llamando al método **FileClient.create**. Cargue un archivo llamando al método **FileClient.append**. Asegúrese de completar la carga llamando al método **FileClient.flush**.

En este ejemplo se carga un archivo de texto en un directorio denominado `my-directory`.

```javascript
async function UploadFile(fileSystemClient) {

  const fs = require('fs') 

  var content = "";
  
  fs.readFile('mytestfile.txt', (err, data) => { 
      if (err) throw err; 

      content = data.toString();

  }) 
  
  const fileClient = fileSystemClient.getFileClient("my-directory/uploaded-file.txt");
  await fileClient.create();
  await fileClient.append(content, 0, content.length);
  await fileClient.flush(content.length);

}
```

## <a name="manage-a-file-acl"></a>Administración de una lista de control de acceso de archivo

En este ejemplo se obtiene y después se establece la ACL de un archivo denominado `upload-file.txt`. En este ejemplo se concede al usuario propietario permisos de lectura, escritura y ejecución, permisos de solo lectura y ejecución al grupo propietario, y permisos de lectura y escritura al resto.

> [!NOTE]
> Si la aplicación autoriza el acceso mediante Azure Active Directory (Azure AD), asegúrese de que la entidad de seguridad que la aplicación usa para autorizar el acceso tiene asignado el [rol de propietario de datos de blobs de almacenamiento](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-blob-data-owner). Para más información sobre cómo se aplican los permisos de ACL y las consecuencias de cambiarlos, consulte [Control de acceso en Azure Data Lake Storage Gen2](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-access-control).

```javascript
async function ManageFileACLs(fileSystemClient) {

  const fileClient = fileSystemClient.getFileClient("my-directory/uploaded-file.txt"); 
  const permissions = await fileClient.getAccessControl();

  console.log(permissions.acl);

  const acl = [
  {
    accessControlType: "user",
    entityId: "",
    defaultScope: false,
    permissions: {
      read: true,
      write: true,
      execute: true
    }
  },
  {
    accessControlType: "group",
    entityId: "",
    defaultScope: false,
    permissions: {
      read: true,
      write: false,
      execute: true
    }
  },
  {
    accessControlType: "other",
    entityId: "",
    defaultScope: false,
    permissions: {
      read: true,
      write: true,
      execute: false
    }

  }

];

await fileClient.setAccessControl(acl);        
}
```

## <a name="download-from-a-directory"></a>Descarga de un directorio

En primer lugar, cree una instancia de **FileSystemClient** que represente al archivo que quiere descargar. Use el método **FileSystemClient.read** para leer el archivo. A continuación, escriba el archivo. En este ejemplo se usa el módulo `fs` de Node.js para ello. 

> [!NOTE]
> Este método de descarga de un archivo solo funciona para aplicaciones de Node.js. Si tiene previsto ejecutar el código en un explorador, consulte el archivo Léame de [Azure Storage File Data Lake client library for JavaScript](https://www.npmjs.com/package/@azure/storage-file-datalake) para obtener un ejemplo de cómo hacerlo en un explorador. 

```javascript
async function DownloadFile(fileSystemClient) {

  const fileClient = fileSystemClient.getFileClient("my-directory/uploaded-file.txt");

  const downloadResponse = await fileClient.read();

  const downloaded = await streamToString(downloadResponse.readableStreamBody);
 
  async function streamToString(readableStream) {
    return new Promise((resolve, reject) => {
      const chunks = [];
      readableStream.on("data", (data) => {
        chunks.push(data.toString());
      });
      readableStream.on("end", () => {
        resolve(chunks.join(""));
      });
      readableStream.on("error", reject);
    });
  }   
  
  const fs = require('fs');

  fs.writeFile('mytestfiledownloaded.txt', downloaded, (err) => {
    if (err) throw err;
  });
}

```

## <a name="list-directory-contents"></a>Lista del contenido del directorio

En este ejemplo se imprimen los nombres de cada directorio y archivo que se encuentra en un directorio denominado `my-directory`.

```javascript
async function ListFilesInDirectory(fileSystemClient) {
  
  let i = 1;

  let iter = await fileSystemClient.listPaths({path: "my-directory", recursive: true});

  for await (const path of iter) {
    
    console.log(`Path ${i++}: ${path.name}, is directory: ${path.isDirectory}`);
  }

}
```

## <a name="see-also"></a>Consulte también

* [Paquete (Administrador de paquetes de Node)](https://www.npmjs.com/package/@azure/storage-file-datalake)
* [Muestras](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-file-datalake/samples)
* [Envíenos sus comentarios](https://github.com/Azure/azure-sdk-for-java/issues)