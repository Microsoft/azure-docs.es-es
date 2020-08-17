---
title: 'Tutorial: Creación de una aplicación de alta disponibilidad con Blob Storage'
titleSuffix: Azure Storage
description: Use el almacenamiento con redundancia geográfica con acceso de lectura (RA-GZRS) para garantizar una alta disponibilidad de los datos de la aplicación.
services: storage
author: tamram
ms.service: storage
ms.topic: tutorial
ms.date: 04/16/2020
ms.author: tamram
ms.reviewer: artek
ms.custom: mvc, devx-track-python, devx-track-javascript
ms.subservice: blobs
ms.openlocfilehash: 25f4c0244a027a0354809080bf13923e92d295f9
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87844599"
---
# <a name="tutorial-build-a-highly-available-application-with-blob-storage"></a>Tutorial: Creación de una aplicación de alta disponibilidad con Blob Storage

Este tutorial es la primera parte de una serie. En él, aprenderá a conseguir una alta disponibilidad de los datos de aplicación en Azure.

Cuando haya terminado este tutorial, tendrá una aplicación de consola que carga y recupera un blob a una cuenta de [almacenamiento con redundancia geográfica con acceso de lectura](../common/storage-redundancy.md) (RA-GZRS).

La redundancia geográfica en Azure Storage replica de forma asincrónica las transacciones de una región primaria en una región secundaria que está a cientos de kilómetros de distancia. Este proceso de replicación garantiza que los datos de la región secundaria tengan coherencia final. La aplicación de consola usa el patrón [disyuntor](/azure/architecture/patterns/circuit-breaker) para determinar el punto de conexión al que debe conectarse, además de cambiar automáticamente de un punto de conexión a otro durante la simulación de errores y recuperaciones.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

En la primera parte de la serie, se aprende a:

> [!div class="checklist"]
> * Crear una cuenta de almacenamiento
> * Establecimiento de la cadena de conexión
> * Ejecución de la aplicación de consola

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial:

# <a name="net"></a>[.NET](#tab/dotnet)

* Instale [Visual Studio 2019](https://www.visualstudio.com/downloads/) con la carga de trabajo de **desarrollo de Azure**.

  ![Desarrollo de Azure (en web y la nube)](media/storage-create-geo-redundant-storage/workloads.png)

# <a name="python"></a>[Python](#tab/python)

* Instalación de [Python](https://www.python.org/downloads/)
* Descargue e instale el [SDK de Azure Storage para Python](https://github.com/Azure/azure-storage-python).

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

* [Instale Node.js](https://nodejs.org).

---

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Una cuenta de almacenamiento proporciona un espacio de nombres único para almacenar y tener acceso a los objetos de datos de Azure Storage.

Siga estos pasos para crear una cuenta de almacenamiento que proporcione redundancia geográfica con acceso de lectura (RA-GZRS):

1. Seleccione el botón **Crear un recurso** en Azure Portal.
2. Seleccione **Cuenta de almacenamiento: blob, archivo, tabla, cola** en la página **Nuevo**.
4. Rellene el formulario de la cuenta de almacenamiento con la siguiente información, como se muestra en la siguiente imagen y seleccione **Crear**:

   | Configuración       | Valor de ejemplo | Descripción |
   | ------------ | ------------------ | ------------------------------------------------- |
   | **Suscripción** | *Mi suscripción* | Para más información acerca de sus suscripciones, consulte [Suscripciones](https://account.azure.com/Subscriptions). |
   | **ResourceGroup** | *myResourceGroup* | Para conocer cuáles son los nombres de grupo de recursos válidos, consulte el artículo [Convenciones de nomenclatura](/azure/architecture/best-practices/resource-naming). |
   | **Nombre** | *mystorageaccount* | Un nombre único para la cuenta de almacenamiento. |
   | **Ubicación** | *Este de EE. UU.* | Elija una ubicación. |
   | **Rendimiento** | *Estándar* | El rendimiento estándar es una buena opción para el escenario de ejemplo. |
   | **Tipo de cuenta** | *StorageV2* | Se recomienda utilizar una cuenta de almacenamiento v2 con fines de uso general. Para más información sobre las cuentas de almacenamiento de Azure, vea [Introducción a las cuentas de almacenamiento](../common/storage-account-overview.md). |
   | **Replicación**| *Almacenamiento con redundancia de zona geográfica con acceso de lectura (RA-GZRS)* | La región primaria ofrece redundancia de zona, se replica en una región secundaria y tiene habilitado el acceso de lectura a la región secundaria. |
   | **Nivel de acceso**| *Acceso frecuente* | Use el nivel de acceso frecuente para los datos a los que se accede de manera habitual. |

    ![creación de cuenta de almacenamiento](media/storage-create-geo-redundant-storage/createragrsstracct.png)

## <a name="download-the-sample"></a>Descarga del ejemplo

# <a name="net"></a>[.NET](#tab/dotnet)

[Descargue el proyecto de ejemplo](https://github.com/Azure-Samples/storage-dotnet-circuit-breaker-pattern-ha-apps-using-ra-grs/archive/master.zip) y extraiga (descomprima) el archivo storage-dotnet-circuit-breaker-pattern-ha-apps-using-ra-grs.zip. También puede usar [git](https://git-scm.com/) para descargar una copia de la aplicación en el entorno de desarrollo. El proyecto de ejemplo contiene una aplicación de consola.

```bash
git clone https://github.com/Azure-Samples/storage-dotnet-circuit-breaker-pattern-ha-apps-using-ra-grs.git
```

# <a name="python"></a>[Python](#tab/python)

[Descargue el proyecto de ejemplo](https://github.com/Azure-Samples/storage-python-circuit-breaker-pattern-ha-apps-using-ra-grs/archive/master.zip) y extraiga (descomprima) el archivo storage-python-circuit-breaker-pattern-ha-apps-using-ra-grs.zip. También puede usar [git](https://git-scm.com/) para descargar una copia de la aplicación en el entorno de desarrollo. El proyecto de ejemplo contiene una aplicación de básica de Python.

```bash
git clone https://github.com/Azure-Samples/storage-python-circuit-breaker-pattern-ha-apps-using-ra-grs.git
```

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

[Descargue el proyecto de ejemplo](https://github.com/Azure-Samples/storage-node-v10-ha-ra-grs) y descomprima el archivo. También puede usar [git](https://git-scm.com/) para descargar una copia de la aplicación en el entorno de desarrollo. El proyecto de ejemplo contiene una aplicación básica de Node.js.

```bash
git clone https://github.com/Azure-Samples/storage-node-v10-ha-ra-grs
```

---

## <a name="configure-the-sample"></a>Configuración del ejemplo

# <a name="net"></a>[.NET](#tab/dotnet)

En la aplicación, debe proporcionar la cadena de conexión para la cuenta de almacenamiento. Esta cadena de conexión se puede almacenar en una variable de entorno de la máquina local que ejecuta la aplicación. Siga uno de los estos ejemplos dependiendo de su sistema operativo para crear la variable de entorno.

En Azure Portal, vaya a la cuenta de almacenamiento. Seleccione **Claves de acceso** en **Configuración**, en su cuenta de almacenamiento. Copie la **cadena de conexión** de la clave principal o secundaria. Ejecute uno de los siguientes comandos en función del sistema operativo y reemplace \<yourconnectionstring\> por la cadena de conexión real. Este comando guarda una variable de entorno en la máquina local. En Windows, la variable de entorno no está disponible hasta que vuelve a cargar el **símbolo del sistema** o el shell que usa.

### <a name="linux"></a>Linux

```
export storageconnectionstring=<yourconnectionstring>
```

### <a name="windows"></a>Windows

```powershell
setx storageconnectionstring "<yourconnectionstring>"
```

# <a name="python"></a>[Python](#tab/python)

En la aplicación, debe proporcionar las credenciales de la cuenta de almacenamiento. Esta información se puede almacenar en variables de entorno en la máquina local que ejecuta la aplicación. Siga uno de estos ejemplos (dependiendo de su sistema operativo) para crear las variables de entorno.

En Azure Portal, vaya a la cuenta de almacenamiento. Seleccione **Claves de acceso** en **Configuración**, en su cuenta de almacenamiento. Pegue los valores de **Nombre de la cuenta de almacenamiento** y **Clave** en los siguientes comandos y reemplace los marcadores de posición \<youraccountname\> y \<youraccountkey\>. Este comando guarda las variables de entorno en la máquina local. En Windows, la variable de entorno no está disponible hasta que vuelve a cargar el **símbolo del sistema** o el shell que usa.

### <a name="linux"></a>Linux

```
export accountname=<youraccountname>
export accountkey=<youraccountkey>
```

### <a name="windows"></a>Windows

```powershell
setx accountname "<youraccountname>"
setx accountkey "<youraccountkey>"
```

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

Para ejecutar este ejemplo, debe agregar las credenciales de la cuenta de almacenamiento al archivo `.env.example` y cambiarle el nombre por `.env`.

```
AZURE_STORAGE_ACCOUNT_NAME=<replace with your storage account name>
AZURE_STORAGE_ACCOUNT_ACCESS_KEY=<replace with your storage account access key>
```

Puede encontrar esta información en Azure Portal; vaya a la cuenta de almacenamiento y seleccione **Claves de acceso** en la sección **Configuración**.

Instale las dependencias necesarias. Para ello, abra un símbolo del sistema, vaya a la carpeta de ejemplo y escriba `npm install`.

---

## <a name="run-the-console-application"></a>Ejecución de la aplicación de consola

# <a name="net"></a>[.NET](#tab/dotnet)

En Visual Studio, presione **F5** o seleccione **Iniciar** para iniciar la depuración de la aplicación. Visual Studio restaura automáticamente los paquetes NuGet que falten, si se ha configurado; consulte el artículo sobre la [instalación y la reinstalación de paquetes con la restauración de paquetes](https://docs.microsoft.com/nuget/consume-packages/package-restore#package-restore-overview) para más información.

Se inicia una ventana en la consola y la aplicación comienza a ejecutarse. La aplicación carga la imagen **HelloWorld.png** de la solución en la cuenta de almacenamiento. La aplicación hace comprobaciones para garantizar que la imagen se haya replicado en el punto de conexión secundario con RA-GZRS. A continuación, comienza la descarga de la imagen hasta 999 veces. Cada lectura se representa con una **P** o una **S**. Donde **P** representa el punto de conexión principal y **S**, el secundario.

![Aplicación de consola en ejecución](media/storage-create-geo-redundant-storage/figure3.png)

En el código de ejemplo, la tarea `RunCircuitBreakerAsync` del archivo `Program.cs` se usa para descargar una imagen de la cuenta de almacenamiento con el método [DownloadToFileAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblob.downloadtofileasync). Antes de la descarga, se define [OperationContext](/dotnet/api/microsoft.azure.cosmos.table.operationcontext). El contexto de operación define los controladores de eventos, que se activan cuando una descarga se completa correctamente o si se produce un error de descarga y se vuelve a intentar.

# <a name="python"></a>[Python](#tab/python)

Para ejecutar la aplicación en un terminal o un símbolo del sistema, vaya al directorio **circuitbreaker.py** y escriba `python circuitbreaker.py`. La aplicación carga la imagen **HelloWorld.png** de la solución en la cuenta de almacenamiento. La aplicación hace comprobaciones para garantizar que la imagen se haya replicado en el punto de conexión secundario con RA-GZRS. A continuación, comienza la descarga de la imagen hasta 999 veces. Cada lectura se representa con una **P** o una **S**. Donde **P** representa el punto de conexión principal y **S**, el secundario.

![Aplicación de consola en ejecución](media/storage-create-geo-redundant-storage/figure3.png)

En el código de ejemplo, el método `run_circuit_breaker` del archivo `circuitbreaker.py` se usa para descargar una imagen de la cuenta de almacenamiento con el método [get_blob_to_path](https://docs.microsoft.com/python/api/azure-storage-blob/azure.storage.blob.baseblobservice.baseblobservice?view=azure-python-previous#get-blob-to-path-container-name--blob-name--file-path--open-mode--wb---snapshot-none--start-range-none--end-range-none--validate-content-false--progress-callback-none--max-connections-2--lease-id-none--if-modified-since-none--if-unmodified-since-none--if-match-none--if-none-match-none--timeout-none-).

La función de reintento de objeto de Storage se establece en una directiva de reintento lineal. La función de reintento determina si se debe reintentar una solicitud y especifica el número de segundos que deben transcurrir para reintentar la solicitud. Establezca el valor de **retry\_to\_secondary** en true, si la solicitud debe reintentarse en el elemento secundario en caso de que la solicitud inicial al principal produzca error. En la aplicación de ejemplo, una directiva de reintento personalizada se define en la función `retry_callback` del objeto de almacenamiento.

Antes de la descarga, se definen el objeto de servicio [retry_callback](https://docs.microsoft.com/python/api/azure-storage-common/azure.storage.common.storageclient.storageclient?view=azure-python) y la función [response_callback](https://docs.microsoft.com/python/api/azure-storage-common/azure.storage.common.storageclient.storageclient?view=azure-python). Estas funciones definen los controladores de eventos que se activan cuando una descarga se completa correctamente o si se produce un error de descarga y se vuelve a intentar.

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

Para ejecutar el ejemplo, abra un símbolo del sistema, vaya a la carpeta de ejemplo y escriba `node index.js`.

El ejemplo crea un contenedor en la cuenta de almacenamiento de blobs, carga **HelloWorld.png** en el contenedor y comprueba periódicamente si el contenedor y la imagen se han replicado en la región secundaria. Después de la replicación, se le pide que escriba **D** o **Q** (seguido de ENTRAR) para descargar o salir. La salida debería similar al siguiente ejemplo:

```
Created container successfully: newcontainer1550799840726
Uploaded blob: HelloWorld.png
Checking to see if container and blob have replicated to secondary region.
[0] Container has not replicated to secondary region yet: newcontainer1550799840726 : ContainerNotFound
[1] Container has not replicated to secondary region yet: newcontainer1550799840726 : ContainerNotFound
...
[31] Container has not replicated to secondary region yet: newcontainer1550799840726 : ContainerNotFound
[32] Container found, but blob has not replicated to secondary region yet.
...
[67] Container found, but blob has not replicated to secondary region yet.
[68] Blob has replicated to secondary region.
Ready for blob download. Enter (D) to download or (Q) to quit, followed by ENTER.
> D
Attempting to download blob...
Blob downloaded from primary endpoint.
> Q
Exiting...
Deleted container newcontainer1550799840726
```

---

## <a name="understand-the-sample-code"></a>Descripción del código de ejemplo

### <a name="net"></a>[.NET](#tab/dotnet)

### <a name="retry-event-handler"></a>Controlador de eventos de reintento

El controlador de eventos `OperationContextRetrying` se llama cuando se produce un error de descarga de la imagen y se ha establecido el reintento. Si se alcanza el máximo de reintentos definido en la aplicación, el valor [LocationMode](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.locationmode) de la solicitud cambia a `SecondaryOnly`. Esta opción obliga a la aplicación a intentar la descarga de la imagen desde el punto de conexión secundario. Esta configuración reduce el tiempo necesario para solicitar la imagen, ya que no se reintenta siempre con el punto de conexión principal.

```csharp
private static void OperationContextRetrying(object sender, RequestEventArgs e)
{
    retryCount++;
    Console.WriteLine("Retrying event because of failure reading the primary. RetryCount = " + retryCount);

    // Check if we have had more than n retries in which case switch to secondary.
    if (retryCount >= retryThreshold)
    {

        // Check to see if we can fail over to secondary.
        if (blobClient.DefaultRequestOptions.LocationMode != LocationMode.SecondaryOnly)
        {
            blobClient.DefaultRequestOptions.LocationMode = LocationMode.SecondaryOnly;
            retryCount = 0;
        }
        else
        {
            throw new ApplicationException("Both primary and secondary are unreachable. Check your application's network connection. ");
        }
    }
}
```

### <a name="request-completed-event-handler"></a>Controlador de eventos de solicitud completada

El controlador de eventos `OperationContextRequestCompleted` se llama cuando la descarga de la imagen es correcta. Si la aplicación utiliza el punto de conexión secundario, continuará usándolo hasta 20 veces. Después de 20 veces, la aplicación establece [LocationMode](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.locationmode) en `PrimaryThenSecondary` de nuevo y vuelve a probar con el punto de conexión principal. Si la solicitud se realiza correctamente, la aplicación continúa leyendo desde el punto de conexión principal.

```csharp
private static void OperationContextRequestCompleted(object sender, RequestEventArgs e)
{
    if (blobClient.DefaultRequestOptions.LocationMode == LocationMode.SecondaryOnly)
    {
        // You're reading the secondary. Let it read the secondary [secondaryThreshold] times,
        //    then switch back to the primary and see if it's available now.
        secondaryReadCount++;
        if (secondaryReadCount >= secondaryThreshold)
        {
            blobClient.DefaultRequestOptions.LocationMode = LocationMode.PrimaryThenSecondary;
            secondaryReadCount = 0;
        }
    }
}
```

### <a name="python"></a>[Python](#tab/python)

### <a name="retry-event-handler"></a>Controlador de eventos de reintento

El controlador de eventos `retry_callback` se llama cuando se produce un error de descarga de la imagen y se ha establecido el reintento. Si se alcanza el máximo de reintentos definido en la aplicación, el valor [LocationMode](https://docs.microsoft.com/python/api/azure-storage-common/azure.storage.common.models.locationmode?view=azure-python) de la solicitud cambia a `SECONDARY`. Esta opción obliga a la aplicación a intentar la descarga de la imagen desde el punto de conexión secundario. Esta configuración reduce el tiempo necesario para solicitar la imagen, ya que no se reintenta siempre con el punto de conexión principal.

```python
def retry_callback(retry_context):
    global retry_count
    retry_count = retry_context.count
    sys.stdout.write(
        "\nRetrying event because of failure reading the primary. RetryCount= {0}".format(retry_count))
    sys.stdout.flush()

    # Check if we have more than n-retries in which case switch to secondary
    if retry_count >= retry_threshold:

        # Check to see if we can fail over to secondary.
        if blob_client.location_mode != LocationMode.SECONDARY:
            blob_client.location_mode = LocationMode.SECONDARY
            retry_count = 0
        else:
            raise Exception("Both primary and secondary are unreachable. "
                            "Check your application's network connection.")
```

### <a name="request-completed-event-handler"></a>Controlador de eventos de solicitud completada

El controlador de eventos `response_callback` se llama cuando la descarga de la imagen es correcta. Si la aplicación utiliza el punto de conexión secundario, continuará usándolo hasta 20 veces. Después de 20 veces, la aplicación establece [LocationMode](https://docs.microsoft.com/python/api/azure-storage-common/azure.storage.common.models.locationmode?view=azure-python) en `PRIMARY` de nuevo y vuelve a probar con el punto de conexión principal. Si la solicitud se realiza correctamente, la aplicación continúa leyendo desde el punto de conexión principal.

```python
def response_callback(response):
    global secondary_read_count
    if blob_client.location_mode == LocationMode.SECONDARY:

        # You're reading the secondary. Let it read the secondary [secondaryThreshold] times,
        # then switch back to the primary and see if it is available now.
        secondary_read_count += 1
        if secondary_read_count >= secondary_threshold:
            blob_client.location_mode = LocationMode.PRIMARY
            secondary_read_count = 0
```

### <a name="nodejs"></a>[Node.js](#tab/nodejs)

Con el SDK de Node.js V10, los controladores de devolución de llamada son innecesarios. En su lugar, el ejemplo crea una canalización que se configura con opciones de reintento y un punto de conexión secundario. Esto permite que la aplicación cambie automáticamente a la canalización secundaria si se produce un error al acceder a los datos mediante la canalización principal.

```javascript
const accountName = process.env.AZURE_STORAGE_ACCOUNT_NAME;
const storageAccessKey = process.env.AZURE_STORAGE_ACCOUNT_ACCESS_KEY;
const sharedKeyCredential = new SharedKeyCredential(accountName, storageAccessKey);

const primaryAccountURL = `https://${accountName}.blob.core.windows.net`;
const secondaryAccountURL = `https://${accountName}-secondary.blob.core.windows.net`;

const pipeline = StorageURL.newPipeline(sharedKeyCredential, {
  retryOptions: {
    maxTries: 3,
    tryTimeoutInMs: 10000,
    retryDelayInMs: 500,
    maxRetryDelayInMs: 1000,
    secondaryHost: secondaryAccountURL
  }
});
```

---

## <a name="next-steps"></a>Pasos siguientes

En la primera parte de la serie, ha aprendido a hacer que una aplicación proporcione una alta disponibilidad mediante las cuentas de almacenamiento RA-GZRS.

Vaya a la segunda parte de la serie para aprender a simular un error y forzar la aplicación a utilizar el punto de conexión secundario con RA-GZRS.

> [!div class="nextstepaction"]
> [Simulación de un error al leer desde la región principal](simulate-primary-region-failure.md)
