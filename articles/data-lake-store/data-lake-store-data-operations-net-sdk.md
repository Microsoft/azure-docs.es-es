---
title: 'SDK de .NET para operaciones del sistema de archivos en Data Lake Storage Gen1: Azure'
description: Use el SDK de .NET Azure Data Lake Storage Gen1 para las operaciones del sistema de archivos en Data Lake Storage Gen1 como crear carpetas, etc.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 01/03/2020
ms.author: twooley
ms.custom: devx-track-dotnet
ms.openlocfilehash: 1044af94df43e61ae8c0032041bc80a356331f1a
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89022469"
---
# <a name="filesystem-operations-on-data-lake-storage-gen1-using-the-net-sdk"></a>Operaciones del sistema de archivos en Data Lake Storage Gen1 mediante el SDK de .NET

> [!div class="op_single_selector"]
> * [SDK de .NET](data-lake-store-data-operations-net-sdk.md)
> * [SDK de Java](data-lake-store-get-started-java-sdk.md)
> * [REST API](data-lake-store-data-operations-rest-api.md)
> * [Python](data-lake-store-data-operations-python.md)
>
>

En este artículo, aprenderá a realizar operaciones del sistema de archivos en Data Lake Storage Gen1 mediante el SDK de .NET. Las operaciones de sistema de archivos incluyen, por ejemplo, la creación de carpetas en una cuenta de Data Lake Storage Gen1 o la carga y descarga de archivos.

Para instrucciones sobre cómo realizar operaciones de administración de cuentas en Data Lake Storage Gen1 mediante el SDK de .NET, consulte [Operaciones de administración de cuentas en Data Lake Storage Gen1 con el SDK de .NET](data-lake-store-get-started-net-sdk.md).

## <a name="prerequisites"></a>Prerrequisitos

* **Visual Studio 2013 o superior**. En las instrucciones de este artículo se usa Visual Studio 2019.

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).

* **Cuenta de Azure Data Lake Storage Gen1**. Para obtener instrucciones sobre cómo crear una cuenta, consulte [Introducción a Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md).

## <a name="create-a-net-application"></a>Creación de una aplicación .NET

El ejemplo de código disponible [en GitHub](https://github.com/Azure-Samples/data-lake-store-adls-dot-net-get-started/tree/master/AdlsSDKGettingStarted) le guía a través del proceso de creación de archivos en el almacén, concatenación de archivos, descarga de un archivo y eliminación de algunos archivos en el almacén. Esta sección del artículo le guía a través de las principales partes del código.

1. En Visual Studio, en el menú **Archivo**, seleccione **Nuevo** y, a continuación, **Proyecto**.
1. Elija **Aplicación de consola (.NET Framework)** y, a continuación, seleccione **Siguiente**.
1. En el **nombre del proyecto**, escriba `CreateADLApplication` y, a continuación, seleccione **Crear**.
1. Agregue los paquetes NuGet al proyecto.

   1. Haga clic con el botón derecho en el Explorador de soluciones y haga clic en **Administrar paquetes de NuGet**.
   1. En la pestaña **Administrador de paquetes NuGet**, asegúrese de que el **Origen del paquete** esté establecido en **nuget.org**. Asegúrese de que la casilla **Incluir versión preliminar** esté activada.
   1. Busque e instale los siguientes paquetes NuGet:

      * `Microsoft.Azure.DataLake.Store` Este artículo se usa v 1.0.0.
      * `Microsoft.Rest.ClientRuntime.Azure.Authentication` Este artículo se usa v2.3.1.

      Cierre el **Administrador de paquetes NuGet**.

1. Abra **Program.cs**, elimine el código existente e incluya las siguientes instrucciones para agregar referencias a espacios de nombres.

    ```
    using System;
    using System.IO;using System.Threading;
    using System.Linq;
    using System.Text;
    using System.Collections.Generic;
    using System.Security.Cryptography.X509Certificates; // Required only if you're using an Azure AD application created with certificates
                
    using Microsoft.Rest;
    using Microsoft.Rest.Azure.Authentication;
    using Microsoft.Azure.DataLake.Store;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    ```

1. Declare las variables como se indica a continuación y proporcione los valores de los marcadores de posición. Además, asegúrese de que la ruta de acceso local y el nombre de archivo que proporciona existen en el equipo.

    ```
    namespace SdkSample
    {
        class Program
        {
            private static string _adlsg1AccountName = "<DATA-LAKE-STORAGE-GEN1-NAME>.azuredatalakestore.net";
        }
    }
    ```

En las secciones restantes del artículo, puede ver cómo usar los métodos de .NET disponibles para realizar operaciones como la autenticación, la carga de archivos, etc.

## <a name="authentication"></a>Authentication

* Para la autenticación del usuario final para la aplicación, consulte [End-user authentication with Data Lake Storage Gen1 using .NET SDK](data-lake-store-end-user-authenticate-net-sdk.md) (Autenticación del usuario final con Data Lake Storage Gen1 mediante el SDK de .NET).
* Para la autenticación entre servicios para la aplicación, consulte [Service-to-service authentication with Data Lake Storage Gen1 using .NET SDK](data-lake-store-service-to-service-authenticate-net-sdk.md) (Autenticación entre servicios con Data Lake Storage Gen1 mediante el SDK de .NET).

## <a name="create-client-object"></a>Creación del objeto de cliente

El fragmento de código siguiente crea los objetos de cliente del sistema de archivos de Data Lake Storage Gen1, que se usan para emitir solicitudes al servicio.

```
// Create client objects
AdlsClient client = AdlsClient.CreateClient(_adlsg1AccountName, adlCreds);
```

## <a name="create-a-file-and-directory"></a>Creación de un archivo y un directorio

Agregue el siguiente fragmento de código a su aplicación. Este fragmento de código agrega un archivo y cualquier directorio primario que no exista.

```
// Create a file - automatically creates any parent directories that don't exist
// The AdlsOutputStream preserves record boundaries - it does not break records while writing to the store

using (var stream = client.CreateFile(fileName, IfExists.Overwrite))
{
    byte[] textByteArray = Encoding.UTF8.GetBytes("This is test data to write.\r\n");
    stream.Write(textByteArray, 0, textByteArray.Length);

    textByteArray = Encoding.UTF8.GetBytes("This is the second line.\r\n");
    stream.Write(textByteArray, 0, textByteArray.Length);
}
```

## <a name="append-to-a-file"></a>Anexión a un archivo

El fragmento de código siguiente anexa datos a un archivo existente en la cuenta de Data Lake Storage Gen1.

```
// Append to existing file

using (var stream = client.GetAppendStream(fileName))
{
    byte[] textByteArray = Encoding.UTF8.GetBytes("This is the added line.\r\n");
    stream.Write(textByteArray, 0, textByteArray.Length);
}
```

## <a name="read-a-file"></a>Lectura de un archivo

El fragmento de código siguiente lee el contenido de un archivo en Data Lake Storage Gen1.

```
//Read file contents

using (var readStream = new StreamReader(client.GetReadStream(fileName)))
{
    string line;
    while ((line = readStream.ReadLine()) != null)
    {
        Console.WriteLine(line);
    }
}
```

## <a name="get-file-properties"></a>Obtención de las propiedades del archivo

El fragmento de código siguiente devuelve las propiedades asociadas a un archivo o un directorio.

```
// Get file properties
var directoryEntry = client.GetDirectoryEntry(fileName);
PrintDirectoryEntry(directoryEntry);
```

La definición del método `PrintDirectoryEntry` está disponible como parte del ejemplo [en GitHub](https://github.com/Azure-Samples/data-lake-store-adls-dot-net-get-started/tree/master/AdlsSDKGettingStarted).

## <a name="rename-a-file"></a>Cambio del nombre de un archivo

El fragmento de código siguiente cambia el nombre de un archivo existente en una cuenta de Data Lake Storage Gen1.

```
// Rename a file
string destFilePath = "/Test/testRenameDest3.txt";
client.Rename(fileName, destFilePath, true);
```

## <a name="enumerate-a-directory"></a>Enumeración de un directorio

El fragmento de código siguiente enumera los directorios en una cuenta de Data Lake Storage Gen1.

```
// Enumerate directory
foreach (var entry in client.EnumerateDirectory("/Test"))
{
    PrintDirectoryEntry(entry);
}
```

La definición del método `PrintDirectoryEntry` está disponible como parte del ejemplo [en GitHub](https://github.com/Azure-Samples/data-lake-store-adls-dot-net-get-started/tree/master/AdlsSDKGettingStarted).

## <a name="delete-directories-recursively"></a>Eliminación de directorios de forma recursiva

El fragmento de código siguiente elimina un directorio y todos sus subdirectorios, de forma recursiva.

```
// Delete a directory and all its subdirectories and files
client.DeleteRecursive("/Test");
```

## <a name="samples"></a>Ejemplos

Estos son algunos ejemplos que muestran cómo usar el SDK del sistema de archivos Data Lake Storage Gen1.

* [Ejemplo básico en GitHub](https://github.com/Azure-Samples/data-lake-store-adls-dot-net-get-started/tree/master/AdlsSDKGettingStarted)
* [Ejemplo avanzado en GitHub](https://github.com/Azure-Samples/data-lake-store-adls-dot-net-samples)

## <a name="see-also"></a>Consulte también

* [Operaciones de administración de cuentas en Data Lake Storage Gen1 con el SDK de .NET](data-lake-store-get-started-net-sdk.md)
* [Data Lake Storage Gen1 .NET SDK Reference](https://docs.microsoft.com/dotnet/api/overview/azure/data-lake-store?view=azure-dotnet) (Referencia del SDK de .NET de Data Lake Storage Gen1)

## <a name="next-steps"></a>Pasos siguientes

* [Protección de datos en Data Lake Storage Gen1](data-lake-store-secure-data.md)
