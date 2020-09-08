---
title: 'Inicio rápido: Biblioteca de Azure Blob Storage v12: Xamarin'
description: En este inicio rápido, aprenderá a usar la versión 12 de la biblioteca cliente de Azure Blob Storage con Xamarin para crear un contenedor y un blob en Blob Storage (objeto). A continuación, aprenderá a descargar el blob en un dispositivo móvil y a enumerar todos los blobs de un contenedor.
author: codemillmatt
ms.author: masoucou
ms.date: 05/08/2020
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: 4fa7ebc71f2bbe6abe6956ad36daaf983e011a94
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89001334"
---
# <a name="quickstart-azure-blob-storage-client-library-v12-with-xamarin"></a>Inicio rápido: Biblioteca cliente de Azure Blob Storage v12 con Xamarin

Introducción a la biblioteca cliente de Azure Blob Storage v12 con Xamarin. Azure Blob Storage es la solución de almacenamiento de objetos de Microsoft para la nube. Siga los pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas. Blob Storage está optimizado para el almacenamiento de cantidades masivas de datos no estructurados.

La biblioteca cliente de Azure Blob Storage v12 con Xamarin se puede usar para:

* Crear un contenedor
* Cargar un blob en Azure Storage
* Enumerar todos los blobs de un contenedor
* Descargar el blob en un dispositivo
* Eliminación de un contenedor

Vínculos de referencia:

* [Documentación de referencia de API](/dotnet/api/azure.storage.blobs)
* [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Blobs)
* [Paquete (NuGet)](https://www.nuget.org/packages/Azure.Storage.Blobs)
* [Ejemplo](https://github.com/Azure-Samples/storage-blobs-xamarin-quickstart)

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)
* Una cuenta de Azure Storage: [cree una cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account)
* Visual Studio con la [carga de trabajo de desarrollo para dispositivos móviles para .NET](https://docs.microsoft.com/xamarin/get-started/installation/?pivots=windows) instalada o [Visual Studio para Mac](https://docs.microsoft.com/visualstudio/mac/installation?view=vsmac-2019)

## <a name="setting-up"></a>Instalación
    
En esta sección se explica cómo preparar un proyecto para que funcione con la biblioteca cliente de Azure Blob Storage v12 con Xamarin.
    
### <a name="create-the-project"></a>Creación del proyecto

1. Abra Visual Studio y cree una aplicación de Forms en blanco.
1. Asígnele un nombre: BlobQuickstartV12

### <a name="install-the-package"></a>Instalar el paquete

1. Haga clic con el botón derecho en la solución en el panel del Explorador de soluciones y seleccione **Administrar paquetes NuGet para la solución**.
1. Busque **Azure.Storage.Blobs** e instale la versión estable más reciente en todos los proyectos de su solución.

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

En el directorio **BlobQuickstartV12**:

1. Abra el archivo *MainPage.xaml* en el editor.
1. Quite todo lo que haya entre los elementos `<ContentPage></ContentPage>` y reemplácelo por lo siguiente:

```xaml
<StackLayout HorizontalOptions="Center" VerticalOptions="Center">

    <Button x:Name="uploadButton" Text="Upload Blob" Clicked="Upload_Clicked"  IsEnabled="False"/>
    <Button x:Name="listButton" Text="List Blobs" Clicked="List_Clicked"  IsEnabled="False" />
    <Button x:Name="downloadButton" Text="Download Blob" Clicked="Download_Clicked"  IsEnabled="False" />
    <Button x:Name="deleteButton" Text="Delete Container" Clicked="Delete_Clicked" IsEnabled="False" />

    <Label Text="" x:Name="resultsLabel" HorizontalTextAlignment="Center" Margin="0,20,0,0" TextColor="Red" />
        
</StackLayout>
```

[!INCLUDE [storage-quickstart-credentials-xamarin-include](../../../includes/storage-quickstart-credentials-xamarin-include.md)]

## <a name="object-model"></a>Modelo de objetos

Azure Blob Storage está optimizado para el almacenamiento de cantidades masivas de datos no estructurados. Los datos no estructurados son datos que no cumplen un modelo de datos o definición concreta, como texto o datos binarios. Blob Storage ofrece tres tipos de recursos:

* La cuenta de almacenamiento
* Un contenedor en la cuenta de almacenamiento
* Un blob en el contenedor

En el siguiente diagrama se muestra la relación entre estos recursos.

![Diagrama de arquitectura de Blob Storage](./media/storage-blobs-introduction/blob1.png)

Use las siguientes clases de .NET para interactuar con estos recursos:

* [BlobServiceClient](/dotnet/api/azure.storage.blobs.blobserviceclient): La clase `BlobServiceClient` permite manipular recursos de Azure Storage y contenedores de blobs.
* [BlobContainerClient](/dotnet/api/azure.storage.blobs.blobcontainerclient): La clase `BlobContainerClient` permite manipular contenedores de Azure Storage y sus blobs.
* [BlobClient](/dotnet/api/azure.storage.blobs.blobclient): La clase `BlobClient` permite manipular los blobs de Azure Storage.
* [BlobDownloadInfo](/dotnet/api/azure.storage.blobs.models.blobdownloadinfo): La clase `BlobDownloadInfo` representa las propiedades y el contenido devuelto por la descarga de un blob.

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código de ejemplo muestran cómo realizar las siguientes tareas con la biblioteca cliente de Azure Blob Storage para .NET en una aplicación Xamarin.Forms:

* [Creación de variables de nivel de clase](#create-class-level-variables)
* [Creación de un contenedor](#create-a-container)
* [Carga de los blobs en un contenedor](#upload-blobs-to-a-container)
* [Enumeración de los blobs de un contenedor](#list-the-blobs-in-a-container)
* [Descarga de los blobs](#download-blobs)
* [Eliminación de un contenedor](#delete-a-container)

### <a name="create-class-level-variables"></a>Creación de variables de nivel de clase

El código siguiente declara varias variables de nivel de clase. Necesitan comunicarse con Azure Blob Storage a lo largo del resto de este ejemplo.

Estas variables se suman a la cadena de conexión de la cuenta de almacenamiento establecida en la sección [Configuración de una cadena de conexión de almacenamiento](#configure-your-storage-connection-string).

Agregue este código como variables de nivel de clase dentro del archivo *MainPage.xaml.cs*:

```csharp
string storageConnectionString = "{set in the Configure your storage connection string section}";
string fileName = $"{Guid.NewGuid()}-temp.txt";

BlobServiceClient client;
BlobContainerClient containerClient;
BlobClient blobClient;
```

### <a name="create-a-container"></a>Crear un contenedor

Decida un nombre para el nuevo contenedor. El código siguiente anexa un valor de GUID al nombre de contenedor para asegurarse de que sea único.

> [!IMPORTANT]
> Los nombres de contenedor deben estar en minúsculas. Para más información acerca de los contenedores de nomenclatura y los blobs, consulte [Naming and Referencing Containers, Blobs, and Metadata](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) (Asignación de nombres y realización de referencias a contenedores, blobs y metadatos).

Cree una instancia de la clase [BlobServiceClient](/dotnet/api/azure.storage.blobs.blobserviceclient). A continuación, llame al método [CreateBlobContainerAsync](/dotnet/api/azure.storage.blobs.blobserviceclient.createblobcontainerasync) para crear el contenedor en la cuenta de almacenamiento.

Agregue este código al archivo *MainPage.xaml.cs*:

```csharp
protected async override void OnAppearing()
{            
    string containerName = $"quickstartblobs{Guid.NewGuid()}";
    
    client = new BlobServiceClient(storageConnectionString);
    containerClient = await client.CreateBlobContainerAsync(containerName);

    resultsLabel.Text = "Container Created\n";

    blobClient = containerClient.GetBlobClient(fileName);

    uploadButton.IsEnabled = true;
}
```

### <a name="upload-blobs-to-a-container"></a>Carga de los blobs en un contenedor

El siguiente fragmento de código:

1. Crea una `MemoryStream` de texto.
1. Carga el texto en un blob, para lo que llama a la función [UploadAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.uploadblobasync?view=azure-dotnet#Azure_Storage_Blobs_BlobContainerClient_UploadBlobAsync_System_String_System_IO_Stream_System_Threading_CancellationToken_) de la clase [BlobContainerClient](/dotnet/api/azure.storage.blobs.blobcontainerclient) y utiliza tanto el nombre de archivo como el `MemoryStream` de texto. Esta operación crea el blob si no existe y lo sobrescribe, en caso de que ya exista.

Agregue este código al archivo *MainPage.xaml.cs*:

```csharp
async void Upload_Clicked(object sender, EventArgs e)
{                                    
    using MemoryStream memoryStream = new MemoryStream(Encoding.UTF8.GetBytes("Hello World!"));

    await containerClient.UploadBlobAsync(fileName, memoryStream);

    resultsLabel.Text += "Blob Uploaded\n";

    uploadButton.IsEnabled = false;
    listButton.IsEnabled = true;
}
```

### <a name="list-the-blobs-in-a-container"></a>Enumerar los blobs de un contenedor

Enumere los blobs en el contenedor llamando al método [GetBlobsAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobsasync). En este caso, solo se ha agregado un blob al contenedor, por lo que la operación de enumeración devuelve simplemente dicho blob.

Agregue este código al archivo *MainPage.xaml.cs*:

```csharp
async void List_Clicked(object sender, EventArgs e)
{            
    await foreach (BlobItem blobItem in containerClient.GetBlobsAsync())
    {
        resultsLabel.Text += blobItem.Name + "\n";                
    }

    listButton.IsEnabled = false;
    downloadButton.IsEnabled = true;
}
```

### <a name="download-blobs"></a>Descargar blobs

Descargue el blob creado previamente llamando al método [DownloadAsync](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.downloadasync). El código de ejemplo copia la representación de `Stream` del blob primero en un `MemoryStream` y, después, en un `StreamReader` para que se pueda mostrar el texto.

Agregue este código al archivo *MainPage.xaml.cs*:

```csharp
async void Download_Clicked(object sender, EventArgs e)
{
    BlobDownloadInfo downloadInfo = await blobClient.DownloadAsync();

    using MemoryStream memoryStream = new MemoryStream();
    
    await downloadInfo.Content.CopyToAsync(memoryStream);
    memoryStream.Position = 0;

    using StreamReader streamReader = new StreamReader(memoryStream);

    resultsLabel.Text += "Blob Contents: \n";
    resultsLabel.Text += await streamReader.ReadToEndAsync();
    resultsLabel.Text += "\n";

    downloadButton.IsEnabled = false;
    deleteButton.IsEnabled = true;
}
```

### <a name="delete-a-container"></a>Eliminación de un contenedor

El código siguiente limpia los recursos que creó; para ello, elimina todo el contenedor con [DeleteAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.deleteasync).

En primer lugar, la aplicación solicita confirmación antes de eliminar el blob y el contenedor. Esta es una buena oportunidad para verificar que los recursos se han creado correctamente antes de eliminarlos.

Agregue este código al archivo *MainPage.xaml.cs*:

```csharp
async void Delete_Clicked(object sender, EventArgs e)
{            
    var deleteContainer = await Application.Current.MainPage.DisplayAlert("Delete Container",
        "You are about to delete the container proceeed?", "OK", "Cancel");

    if (deleteContainer == false)
        return;

    await containerClient.DeleteAsync();

    resultsLabel.Text += "Container Deleted";

    deleteButton.IsEnabled = false;
}
```

## <a name="run-the-code"></a>Ejecución del código

Cuando la aplicación se inicie, lo primero que hará es crear el contenedor tal como aparece. Luego, tendrá que hacer clic en los botones para cargar, enumerar y descargar los blobs, así como para eliminar el contenedor.

Para ejecutar la aplicación en Windows, presione F5. Para ejecutar la aplicación en Mac, presione Cmd+Enter.

La aplicación escribe en la pantalla después de cada operación. La salida de la aplicación es similar a la del ejemplo siguiente:

```output
Container Created
Blob Uploaded
98d9a472-8e98-4978-ba4f-081d69d2e6f8-temp.txt
Blob Contents:
Hello World!
Container Deleted
```

Antes de comenzar el proceso de limpieza, compruebe que la salida del contenido del blob en la pantalla coincide con el valor que se cargó.

Después de comprobar los valores, confirme la solicitud de eliminación del contenedor y finalice la demostración.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, aprenderá a cargar, descargar y enumerar blobs mediante una biblioteca cliente de Azure Blob Storage v12 con Xamarin.

Para ver las aplicaciones de ejemplo de Blob Storage, siga estos pasos:

> [!div class="nextstepaction"]
> [Ejemplo de la versión 12 del SDK de Azure Blob Storage Xamarin](https://github.com/Azure-Samples/storage-blobs-xamarin-quickstart)

* Para ver tutoriales, ejemplos, inicios rápido y otra documentación, visite [Azure para desarrolladores para dispositivos móviles](/azure/mobile-apps).
* Para más información sobre Xamarin, consulte [Introducción a Xamarin](/xamarin/get-started/).
