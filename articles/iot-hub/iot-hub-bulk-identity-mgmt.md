---
title: Importación y exportación de identidades de dispositivo de IoT Hub de Azure | Microsoft Docs
description: Describe cómo usar el SDK de servicio IoT de Azure para realizar operaciones masivas en el registro de identidad con el fin de importar y exportar identidades de dispositivo. Las operaciones de importación permiten crear, actualizar y eliminar las identidades de dispositivo de forma masiva.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 05/11/2019
ms.author: robinsh
ms.openlocfilehash: 5dd93af7deec2b0c8c90f6a8586de905207ad0a6
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65796362"
---
# <a name="import-and-export-iot-hub-device-identities-in-bulk"></a>Importar y exportar identidades de dispositivo de IoT Hub de forma masiva

Cada Centro de IoT tiene un registro de identidad que se puede usar para crear recursos por dispositivo en el servicio. El registro de identidad también permite controlar el acceso a los puntos de conexión accesibles desde los dispositivos. En este artículo se describe cómo importar y exportar identidades de dispositivo de forma masiva hacia y desde un Registro de identidad.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Las operaciones de importación y exportación tienen lugar en el contexto de *Trabajos* que permiten ejecutar operaciones de servicio de forma masiva en un IoT Hub.

La clase **RegistryManager** incluye los métodos **ExportDevicesAsync** y **ImportDevicesAsync**, que usan el marco **Trabajo**. Estos métodos le permiten exportar, importar y sincronizar la totalidad de un Registro de identidad de IoT Hub.

En este tema se describe el uso de la clase **RegistryManager** y del sistema de **trabajo** para realizar importaciones y exportaciones en bloque de dispositivos a y desde un registro de identidad de un centro de IoT. También puede usar el servicio Azure IoT Hub Device Provisioning para habilitar el aprovisionamiento Just-In-Time sin intervención del usuario de uno o varios centros de IoT sin necesidad de ninguna intervención humana. Para más información, consulte la [documentación del servicio de aprovisionamiento](/azure/iot-dps).


## <a name="what-are-jobs"></a>¿Qué son los trabajos?

Las operaciones de Registro de identidad usan el sistema de **trabajo** cuando la operación:

* Tiene un tiempo de ejecución potencialmente largo en comparación con las operaciones en tiempo de ejecución estándar.

* Devuelve una gran cantidad de datos al usuario.

En estos casos, en lugar de una única API en espera o que bloquea el resultado de la operación, esta crea de forma asincrónica un **Trabajo** para ese Centro de IoT. La operación después devuelve inmediatamente un objeto **JobProperties**.

El siguiente fragmento de código de C# muestra cómo crear un trabajo de exportación:

```csharp
// Call an export job on the IoT Hub to retrieve all devices
JobProperties exportJob = await 
  registryManager.ExportDevicesAsync(containerSasUri, false);
```

> [!NOTE]
> Para usar la clase **RegistryManager** en el código de C#, agregue el paquete de NuGet **Microsoft.Azure.Devices** al proyecto. La clase **RegistryManager** está en el espacio de nombres **Microsoft.Azure.Devices**.

Puede usar la clase **RegistryManager** para consultar el estado de **Trabajo** con los metadatos de **JobProperties** devueltos. Para crear una instancia de la clase **RegistryManager**, use el método **CreateFromConnectionString**.

```csharp
RegistryManager registryManager =
  RegistryManager.CreateFromConnectionString("{your IoT Hub connection string}");
```

Para buscar la cadena de conexión de su IoT Hub en Azure Portal:

- Vaya a su instancia de IoT Hub.

- Seleccione **Directivas de acceso compartido**.

- Seleccione una directiva teniendo en cuenta los permisos que necesita.

- Copie la cadena de conexión desde el panel del lado derecho de la pantalla.

El siguiente fragmento de código de C# muestra cómo sondear cada cinco segundos para ver si el trabajo ha terminado de ejecutarse:

```csharp
// Wait until job is finished
while(true)
{
  exportJob = await registryManager.GetJobAsync(exportJob.JobId);
  if (exportJob.Status == JobStatus.Completed || 
      exportJob.Status == JobStatus.Failed ||
      exportJob.Status == JobStatus.Cancelled)
  {
    // Job has finished executing
    break;
  }

  await Task.Delay(TimeSpan.FromSeconds(5));
}
```

## <a name="device-importexport-job-limits"></a>Límites de trabajo de importación/exportación de dispositivos

Importar solo 1 dispositivos activos o trabajo de exportación se permite en un momento para todos los niveles de IoT Hub. IoT Hub también tiene los límites de velocidad de las operaciones de trabajos. Para obtener más información, consulte [referencia: cuotas y limitación IoT Hub](iot-hub-devguide-quotas-throttling.md).

## <a name="export-devices"></a>Exportación de dispositivos

Use el método **ExportDevicesAsync** para exportar la totalidad de un Registro de identidad de IoT Hub a un contenedor de blobs de [Azure Storage](../storage/index.yml) mediante una [firma de acceso compartido](../storage/common/storage-security-guide.md#data-plane-security).

Este método le permite crear copias de seguridad confiables de la información del dispositivo en un contenedor de blobs que usted controle.

El método **ExportDevicesAsync** requiere dos parámetros:

* Una *cadena* que contiene un identificador URI de un contenedor de blobs. Este identificador URI debe contener un token SAS que conceda acceso de escritura al contenedor. El trabajo crea un blob en bloques en este contenedor para almacenar los datos serializados de exportación del dispositivo. El token de SAS debe incluir estos permisos:

   ```csharp
   SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.Read 
     | SharedAccessBlobPermissions.Delete
   ```

* Un valor *booleano* que indica si desea excluir las claves de autenticación de los datos de exportación. Si es **false**, las claves de autenticación se incluyen en la exportación. De lo contrario, las claves se exportan como **null**.

El siguiente fragmento de código de C# muestra cómo iniciar un trabajo de exportación que incluye las claves de autenticación de dispositivo en los datos de exportación y luego sondea la finalización:

```csharp
// Call an export job on the IoT Hub to retrieve all devices
JobProperties exportJob = 
  await registryManager.ExportDevicesAsync(containerSasUri, false);

// Wait until job is finished
while(true)
{
    exportJob = await registryManager.GetJobAsync(exportJob.JobId);
    if (exportJob.Status == JobStatus.Completed || 
        exportJob.Status == JobStatus.Failed ||
        exportJob.Status == JobStatus.Cancelled)
    {
    // Job has finished executing
    break;
    }

    await Task.Delay(TimeSpan.FromSeconds(5));
}
```

El trabajo almacena su salida en el contenedor de blobs proporcionado como un blob en bloques con el nombre **devices.txt**. Los datos de salida se componen de datos de dispositivos serializados de JSON, con un dispositivo por línea.

En el siguiente ejemplo se muestran los datos de salida:

```json
{"id":"Device1","eTag":"MA==","status":"enabled","authentication":{"symmetricKey":{"primaryKey":"abc=","secondaryKey":"def="}}}
{"id":"Device2","eTag":"MA==","status":"enabled","authentication":{"symmetricKey":{"primaryKey":"abc=","secondaryKey":"def="}}}
{"id":"Device3","eTag":"MA==","status":"disabled","authentication":{"symmetricKey":{"primaryKey":"abc=","secondaryKey":"def="}}}
{"id":"Device4","eTag":"MA==","status":"disabled","authentication":{"symmetricKey":{"primaryKey":"abc=","secondaryKey":"def="}}}
{"id":"Device5","eTag":"MA==","status":"enabled","authentication":{"symmetricKey":{"primaryKey":"abc=","secondaryKey":"def="}}}
```

Si un dispositivo tiene datos gemelos, estos también se exportan junto con los datos del dispositivo. En el siguiente ejemplo se muestra este formato. Todos los datos de la línea "twinETag" hasta el final son datos gemelos.

```json
{
   "id":"export-6d84f075-0",
   "eTag":"MQ==",
   "status":"enabled",
   "statusReason":"firstUpdate",
   "authentication":null,
   "twinETag":"AAAAAAAAAAI=",
   "tags":{
      "Location":"LivingRoom"
   },
   "properties":{
      "desired":{
         "Thermostat":{
            "Temperature":75.1,
            "Unit":"F"
         },
         "$metadata":{
            "$lastUpdated":"2017-03-09T18:30:52.3167248Z",
            "$lastUpdatedVersion":2,
            "Thermostat":{
               "$lastUpdated":"2017-03-09T18:30:52.3167248Z",
               "$lastUpdatedVersion":2,
               "Temperature":{
                  "$lastUpdated":"2017-03-09T18:30:52.3167248Z",
                  "$lastUpdatedVersion":2
               },
               "Unit":{
                  "$lastUpdated":"2017-03-09T18:30:52.3167248Z",
                  "$lastUpdatedVersion":2
               }
            }
         },
         "$version":2
      },
      "reported":{
         "$metadata":{
            "$lastUpdated":"2017-03-09T18:30:51.1309437Z"
         },
         "$version":1
      }
   }
}
```

Si necesita acceso a los datos en el código, puede deserializar fácilmente estos datos mediante la clase **ExportImportDevice** . El siguiente fragmento de código de C# muestra cómo leer la información de dispositivo que se ha exportado previamente a un blob en bloques:

```csharp
var exportedDevices = new List<ExportImportDevice>();

using (var streamReader = new StreamReader(await blob.OpenReadAsync(AccessCondition.GenerateIfExistsCondition(), null, null), Encoding.UTF8))
{
  while (streamReader.Peek() != -1)
  {
    string line = await streamReader.ReadLineAsync();
    var device = JsonConvert.DeserializeObject<ExportImportDevice>(line);
    exportedDevices.Add(device);
  }
}
```

## <a name="import-devices"></a>Importación de dispositivos

El método **ImportDevicesAsync** de la clase **RegistryManager** le permite realizar operaciones de sincronización e importación masiva en un Registro de identidad de IoT Hub. Al igual que el método **ExportDevicesAsync**, el método **ImportDevicesAsync** usa el marco **Trabajo**.

Tenga cuidado con el método **ImportDevicesAsync** porque además del aprovisionamiento de nuevos dispositivos en el Registro de identidad, también puede actualizar y eliminar dispositivos existentes.

> [!WARNING]
> Una operación de importación no se puede deshacer. Realice siempre una copia de seguridad de los datos existentes mediante el método **ExportDevicesAsync** en otro contenedor de blobs antes de realizar cambios masivos en el Registro de identidad.

El método **ImportDevicesAsync** requiere dos parámetros:

* Una *cadena* que contiene un identificador URI de un contenedor de blobs de [Azure Storage](../storage/index.yml) para usar como *entrada* para el trabajo. Este identificador URI debe contener un token SAS que conceda acceso de lectura al contenedor. Este contenedor debe incluir un blob con el nombre **devices.txt** que contenga los datos de dispositivo serializados para importar en el Registro de identidad. Los datos de importación deben contener la información del dispositivo en el mismo formato JSON que usa el trabajo **ExportImportDevice** cuando crea un blob **devices.txt**. El token de SAS debe incluir estos permisos:

   ```csharp
   SharedAccessBlobPermissions.Read
   ```

* Una *cadena* que contenga un identificador URI de un contenedor de blobs de [Azure Storage](https://azure.microsoft.com/documentation/services/storage/) para usar como *salida* del trabajo. El trabajo crea un blob en bloques en este contenedor para almacenar cualquier información de error del **Trabajo**de importación finalizado. El token de SAS debe incluir estos permisos:

   ```csharp
   SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.Read 
     | SharedAccessBlobPermissions.Delete
   ```

> [!NOTE]
> Los dos parámetros pueden apuntar al mismo contenedor de blobs. Los parámetros independientes simplemente habilitan más control sobre sus datos, ya que el contenedor de salida requiere permisos adicionales.

El siguiente fragmento de código de C# muestra cómo iniciar un trabajo de importación:

```csharp
JobProperties importJob = 
   await registryManager.ImportDevicesAsync(containerSasUri, containerSasUri);
```

Este método también se puede usar para importar los datos para el dispositivo gemelo. El formato de la entrada de datos es el mismo que se muestra en la sección **ExportDevicesAsync**. De esta manera, se pueden volver a importar los datos exportados. El valor de **$metadata** es opcional.

## <a name="import-behavior"></a>Comportamiento de la importación

Puede usar el método **ImportDevicesAsync** para realizar las siguientes operaciones de forma masiva en el Registro de identidad:

* Registro masivo de nuevos dispositivos
* Eliminación masiva de dispositivos existentes
* Cambios masivos de estado (habilitar o deshabilitar dispositivos)
* Asignación masiva de nuevas claves de autenticación de dispositivos
* Regeneración automática masiva de claves de autenticación de dispositivos
* Actualización masiva de datos gemelos

Puede realizar cualquier combinación de las operaciones precedentes en una única llamada **ImportDevicesAsync**. Por ejemplo, puede registrar nuevos dispositivos y eliminar o actualizar los dispositivos existentes al mismo tiempo. Cuando se utiliza junto con el método **ExportDevicesAsync** , puede migrar completamente todos los dispositivos de un Centro de IoT a otro.

Si el archivo de importación especifica metadatos gemelos, estos sobrescriben los existentes. Si el archivo de importación no incluye metadatos gemelos, solo se actualizan los metadatos de `lastUpdateTime` que usan la hora actual.

Use la propiedad opcional **importMode** en los datos de serialización de importación para cada dispositivo para controlar el proceso de importación por dispositivo. La propiedad **importMode** tiene las siguientes opciones:

| importMode | DESCRIPCIÓN |
| --- | --- |
| **createOrUpdate** |Si no existe un dispositivo con los valores especificados **ID**, se registra por primera vez. <br/>Si el dispositivo ya existe, la información existente se sobrescribe con los datos de entrada proporcionados con independencia del valor **ETag** . <br> El usuario puede especificar opcionalmente datos gemelos junto con los datos del dispositivo. El valor etag del gemelo, si se especifica, se procesa por separado del etag del dispositivo. Si no coincide con el etag del gemelo existente, se escribe un error en el archivo de registro. |
| **create** |Si no existe un dispositivo con los valores especificados **ID**, se registra por primera vez. <br/>Si el dispositivo ya existe, se escribe un error en el archivo de registro. <br> El usuario puede especificar opcionalmente datos gemelos junto con los datos del dispositivo. El valor etag del gemelo, si se especifica, se procesa por separado del etag del dispositivo. Si no coincide con el etag del gemelo existente, se escribe un error en el archivo de registro. |
| **update** |Si ya existe un dispositivo con los valores especificados **ID**, información existente se sobrescribe con los datos de entrada proporcionados sin respecto a la **ETag** valor. <br/>Si el dispositivo no existe, se escribe un error en el archivo de registro. |
| **updateIfMatchETag** |Si ya existe un dispositivo con los valores especificados **ID**, información existente se sobrescribe con los datos de entrada proporcionados solo si hay un **ETag** coinciden. <br/>Si el dispositivo no existe, se escribe un error en el archivo de registro. <br/>Si no existe la coincidencia con **ETag** , se escribe un error en el archivo de registro. |
| **createOrUpdateIfMatchETag** |Si no existe un dispositivo con los valores especificados **ID**, se registra por primera vez. <br/>Si el dispositivo ya existe, la información existente se sobrescribe con los datos de entrada proporcionados solo si hay una coincidencia con **ETag** . <br/>Si no existe la coincidencia con **ETag** , se escribe un error en el archivo de registro. <br> El usuario puede especificar opcionalmente datos gemelos junto con los datos del dispositivo. El valor etag del gemelo, si se especifica, se procesa por separado del etag del dispositivo. Si no coincide con el etag del gemelo existente, se escribe un error en el archivo de registro. |
| **delete** |Si ya existe un dispositivo con los valores especificados **ID**, se elimina sin respecto a la **ETag** valor. <br/>Si el dispositivo no existe, se escribe un error en el archivo de registro. |
| **deleteIfMatchETag** |Si ya existe un dispositivo con los valores especificados **ID**, se elimina solo si hay un **ETag** coinciden. Si el dispositivo no existe, se escribe un error en el archivo de registro. <br/>Si no existe una coincidencia con ETag, se escribe un error en el archivo de registro. |

> [!NOTE]
> Si los datos de serialización no definen explícitamente una marca **importMode** para un dispositivo, se establece de manera predeterminada en **createOrUpdate** durante la operación de importación.

## <a name="import-devices-example--bulk-device-provisioning"></a>Ejemplo de importación de dispositivos: aprovisionamiento masivo de dispositivos

El siguiente ejemplo de código de C# muestra cómo generar varias identidades de dispositivo que:

* Incluyen claves de autenticación.
* Escriben la información del dispositivo en un blob en bloques.
* Importan los dispositivos en el Registro de identidad.

```csharp
// Provision 1,000 more devices
var serializedDevices = new List<string>();

for (var i = 0; i < 1000; i++)
{
  // Create a new ExportImportDevice
  // CryptoKeyGenerator is in the Microsoft.Azure.Devices.Common namespace
  var deviceToAdd = new ExportImportDevice()
  {
    Id = Guid.NewGuid().ToString(),
    Status = DeviceStatus.Enabled,
    Authentication = new AuthenticationMechanism()
    {
      SymmetricKey = new SymmetricKey()
      {
        PrimaryKey = CryptoKeyGenerator.GenerateKey(32),
        SecondaryKey = CryptoKeyGenerator.GenerateKey(32)
      }
    },
    ImportMode = ImportMode.Create
  };

  // Add device to the list
  serializedDevices.Add(JsonConvert.SerializeObject(deviceToAdd));
}

// Write the list to the blob
var sb = new StringBuilder();
serializedDevices.ForEach(serializedDevice => sb.AppendLine(serializedDevice));
await blob.DeleteIfExistsAsync();

using (CloudBlobStream stream = await blob.OpenWriteAsync())
{
  byte[] bytes = Encoding.UTF8.GetBytes(sb.ToString());
  for (var i = 0; i < bytes.Length; i += 500)
  {
    int length = Math.Min(bytes.Length - i, 500);
    await stream.WriteAsync(bytes, i, length);
  }
}

// Call import using the blob to add new devices
// Log information related to the job is written to the same container
// This normally takes 1 minute per 100 devices
JobProperties importJob =
   await registryManager.ImportDevicesAsync(containerSasUri, containerSasUri);

// Wait until job is finished
while(true)
{
  importJob = await registryManager.GetJobAsync(importJob.JobId);
  if (importJob.Status == JobStatus.Completed || 
      importJob.Status == JobStatus.Failed ||
      importJob.Status == JobStatus.Cancelled)
  {
    // Job has finished executing
    break;
  }

  await Task.Delay(TimeSpan.FromSeconds(5));
}
```

## <a name="import-devices-example--bulk-deletion"></a>Ejemplo de importación de dispositivos: eliminación masiva

El ejemplo de código siguiente muestra cómo eliminar los dispositivos que ha agregado con el ejemplo de código anterior:

```csharp
// Step 1: Update each device's ImportMode to be Delete
sb = new StringBuilder();
serializedDevices.ForEach(serializedDevice =>
{
  // Deserialize back to an ExportImportDevice
  var device = JsonConvert.DeserializeObject<ExportImportDevice>(serializedDevice);

  // Update property
  device.ImportMode = ImportMode.Delete;

  // Re-serialize
  sb.AppendLine(JsonConvert.SerializeObject(device));
});

// Step 2: Write the new import data back to the block blob
await blob.DeleteIfExistsAsync();
using (CloudBlobStream stream = await blob.OpenWriteAsync())
{
  byte[] bytes = Encoding.UTF8.GetBytes(sb.ToString());
  for (var i = 0; i < bytes.Length; i += 500)
  {
    int length = Math.Min(bytes.Length - i, 500);
    await stream.WriteAsync(bytes, i, length);
  }
}

// Step 3: Call import using the same blob to delete all devices
importJob = await registryManager.ImportDevicesAsync(containerSasUri, containerSasUri);

// Wait until job is finished
while(true)
{
  importJob = await registryManager.GetJobAsync(importJob.JobId);
  if (importJob.Status == JobStatus.Completed || 
      importJob.Status == JobStatus.Failed ||
      importJob.Status == JobStatus.Cancelled)
  {
    // Job has finished executing
    break;
  }

  await Task.Delay(TimeSpan.FromSeconds(5));
}
```

## <a name="get-the-container-sas-uri"></a>Obtención del identificador URI de SAS del contenedor

El ejemplo de código siguiente muestra cómo generar un [identificador URI de SAS](../storage/common/storage-dotnet-shared-access-signature-part-1.md) con permisos de lectura, escritura y eliminación para un contenedor de blobs:

```csharp
static string GetContainerSasUri(CloudBlobContainer container)
{
  // Set the expiry time and permissions for the container.
  // In this case no start time is specified, so the
  // shared access signature becomes valid immediately.
  var sasConstraints = new SharedAccessBlobPolicy();
  sasConstraints.SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24);
  sasConstraints.Permissions = 
    SharedAccessBlobPermissions.Write | 
    SharedAccessBlobPermissions.Read | 
    SharedAccessBlobPermissions.Delete;

  // Generate the shared access signature on the container,
  // setting the constraints directly on the signature.
  string sasContainerToken = container.GetSharedAccessSignature(sasConstraints);

  // Return the URI string for the container,
  // including the SAS token.
  return container.Uri + sasContainerToken;
}
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió a realizar operaciones de forma masiva en el Registro de identidad en un centro de IoT. Siga estos vínculos para más información sobre la administración de Azure IoT Hub:

* [Métricas de IoT Hub](iot-hub-metrics.md)
* [Registros del centro de IoT](iot-hub-monitor-resource-health.md)

Para explorar aún más las funcionalidades de IoT Hub, consulte:

* [Guía para desarrolladores de IoT Hub](iot-hub-devguide.md)
* [Implementación de IA en dispositivos perimetrales con Azure IoT Edge](../iot-edge/tutorial-simulate-device-linux.md)

Para explorar el uso del servicio IoT Hub Device Provisioning para habilitar el aprovisionamiento Just-In-Time sin intervención del usuario, vea: 

* [Servicio Azure IoT Hub Device Provisioning](/azure/iot-dps)
