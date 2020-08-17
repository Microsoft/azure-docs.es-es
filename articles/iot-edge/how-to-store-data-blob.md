---
title: 'Almacenamiento de blob en bloques en los dispositivos: Azure IoT Edge | Microsoft Docs'
description: Comprenda las características de niveles y período de vida, consulte las operaciones de Blob Storage compatibles y conéctese a su cuenta de Blob Storage.
author: kgremban
ms.author: kgremban
ms.reviewer: arduppal
ms.date: 12/13/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 07da9316ea76e609948eed586f776be33c91b4bb
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87287261"
---
# <a name="store-data-at-the-edge-with-azure-blob-storage-on-iot-edge"></a>Almacenamiento de datos en el perímetro con Azure Blob Storage en IoT Edge

Azure Blob Storage en IoT Edge proporciona una solución de almacenamiento de [blobs en bloques](https://docs.microsoft.com/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-block-blobs) y [blobs en anexos](https://docs.microsoft.com/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-append-blobs) en el perímetro. En los dispositivos IoT Edge, los módulos de almacenamiento de blobs se comportan como un servicio de blobs de Azure, excepto los blobs que se almacenan localmente en el dispositivo IoT Edge. Para acceder a los blobs puede usar los mismos métodos del SDK de Azure Storage o las llamadas a la API de blobs a las que está acostumbrado. En este artículo se explican los conceptos relacionados con Azure Blob Storage en el contenedor de IoT Edge que ejecuta un servicio de blob en su dispositivo de IoT Edge.

Este módulo es útil en los siguientes escenarios:

* Cuando es necesario almacenar los datos localmente hasta que se puedan procesar o transferir a la nube. Estos datos pueden ser vídeos, imágenes, datos económicos, información médica o cualquier otro dato no estructurado.
* Cuando los dispositivos están en un lugar con conectividad limitada.
* Cuando se quiere procesar los datos de un modo eficiente en el entorno local para poder acceder a ellos con una latencia baja y actuar ante una emergencia lo antes posible.
* Cuando se quieren reducir los costos de ancho de banda y evitar la transferencia a la nube de terabytes de datos. Puede procesar los datos en el entorno local y, después, enviar a la nube solo los datos procesados.

Consulte el vídeo para ver una introducción breve.
> [!VIDEO https://www.youtube.com/embed/xbwgMNGB_3Y]

En este módulo se proporcionan los elementos **deviceToCloudUpload** y **deviceAutoDelete**.

**deviceToCloudUpload** es una funcionalidad configurable. Esta función le permite cargar automáticamente los datos de la instancia local de Blob Storage a Azure con compatibilidad para una conectividad de Internet intermitente. Eso le permite lo siguiente:

* Activar y desactivar la característica deviceToCloudUpload.
* Elegir el orden en el que los datos se copian en Azure como NewestFirst o OldestFirst.
* Especificar la cuenta de Azure Storage en la que quiere que se carguen los datos.
* Especificar los contenedores que quiera cargar en Azure. Este módulo le permite especificar los nombres de los contenedores de origen y destino.
* Elegir la capacidad de eliminar los blobs inmediatamente, una vez finalizada la carga de almacenamiento en la nube.
* Realizar la carga completa del blob (mediante la operación `Put Blob`) y la carga en el nivel de bloque (mediante las operaciones `Put Block`, `Put Block List` y `Append Block`).

Este módulo usa la carga a nivel de bloque cuando su blob consiste en bloques. Éstos son algunos de los escenarios comunes:

* La aplicación actualiza algunos bloques de un blob en bloques cargado previamente o anexa nuevos bloques a un blob en anexos, ya que este módulo carga solo los bloques actualizados y no el blob completo.
* El módulo está cargando el blob y la conexión a Internet desaparece, así que cuando la conectividad vuelve a funcionar, solo carga los bloques restantes y no todo el blob.

Si el proceso finaliza de forma inesperada (por ejemplo, debido a un error eléctrico) durante la carga de un blob, todos los bloques de la carga se volverán a cargar, cuando el módulo vuelva a estar en línea.

**deviceAutoDelete** es una funcionalidad configurable. Esta función elimina automáticamente sus blobs del almacenamiento local cuando caduca la duración especificada (medida en minutos). Eso le permite lo siguiente:

* Activar y desactivar la función deviceAutoDelete.
* Especificar el tiempo en minutos (deleteAfterMinutes) después del cual los blobs se eliminarán automáticamente.
* Elija la capacidad de retener el blob mientras se está cargando, si el valor de deleteAfterMinutes expira.

## <a name="prerequisites"></a>Requisitos previos

Un dispositivo de Azure IoT Edge:

* Puede usar la máquina de desarrollo o una máquina virtual como el dispositivo de IoT Edge siguiendo los pasos que se indican en la guía de inicio rápido para dispositivos de [Linux](quickstart-linux.md) o de [Windows](quickstart.md).

* Para obtener una lista de las arquitecturas y sistemas operativos compatibles, consulte [Sistemas compatibles con Azure IoT Edge](support.md#operating-systems). El módulo de Azure Blob Storage en IoT Edge admite las siguientes arquitecturas:
  * Windows AMD64
  * Linux AMD64
  * Linux ARM32
  * Linux ARM64 (versión preliminar)

Recursos en la nube:

Una instancia de [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) de nivel estándar en Azure.

## <a name="devicetocloudupload-and-deviceautodelete-properties"></a>propiedades deviceToCloudUpload y deviceAutoDelete

Use las propiedades necesarias del módulo para establecer **deviceToCloudUploadProperties** y **deviceAutoDeleteProperties**. Las propiedades necesarias pueden configurarse durante la implementación o modificarse más adelante editando el módulo gemelo sin necesidad de volver a implementarlas. Le recomendamos seleccionar el "Módulo gemelo" para `reported configuration` y `configurationValidation`, y que así pueda asegurarse de que los valores se propagan correctamente.

### <a name="devicetoclouduploadproperties"></a>deviceToCloudUploadProperties

El nombre de este valor es `deviceToCloudUploadProperties`. Si usa el simulador de IoT Edge, establezca los valores en las variables de entorno relacionadas de estas propiedades, que puede encontrar en la sección de explicación.

| Propiedad | Valores posibles | Explicación |
| ----- | ----- | ---- |
| uploadOn | true, false | Se establece en `false` de forma predeterminada. Si quiere activar la característica, establezca este campo en `true`. <br><br> Variable de entorno: `deviceToCloudUploadProperties__uploadOn={false,true}` |
| uploadOrder | NewestFirst, OldestFirst | Le permite elegir el orden en que se copiarán los datos a Azure. Se establece en `OldestFirst` de forma predeterminada. El orden está determinado en función de la última hora de modificación del blob. <br><br> Variable de entorno: `deviceToCloudUploadProperties__uploadOrder={NewestFirst,OldestFirst}` |
| cloudStorageConnectionString |  | `"DefaultEndpointsProtocol=https;AccountName=<your Azure Storage Account Name>;AccountKey=<your Azure Storage Account Key>;EndpointSuffix=<your end point suffix>"` es una cadena de conexión que le permite especificar la cuenta de almacenamiento en la que desea cargar los datos. Especificar `Azure Storage Account Name`, `Azure Storage Account Key`, `End point suffix`. Agregue el valor de EndpointSuffix adecuado de Azure donde se cargarán los datos; recuerde que varía según Global Azure, Government Azure y Microsoft Azure Stack. <br><br> Aquí puede elegir especificar una cadena de conexión de SAS de Azure Storage. Pero tiene que actualizar esta propiedad cuando expire. <br><br> Variable de entorno: `deviceToCloudUploadProperties__cloudStorageConnectionString=<connection string>` |
| storageContainersForUpload | `"<source container name1>": {"target": "<target container name>"}`,<br><br> `"<source container name1>": {"target": "%h-%d-%m-%c"}`, <br><br> `"<source container name1>": {"target": "%d-%c"}` | Le permite especificar los nombres de los contenedores que desea cargar en Azure. Este módulo le permite especificar los nombres de los contenedores de origen y destino. Si no especifica el nombre del contenedor de destino, se asignará automáticamente el nombre del contenedor como `<IoTHubName>-<IotEdgeDeviceID>-<ModuleName>-<SourceContainerName>`. Puede crear cadenas de plantillas para el nombre del contenedor de destino, así que compruebe la columna de valores posibles. <br>* %h -> nombre de IoT Hub (entre 3 y 50 caracteres). <br>* %d -> id. de dispositivo de IoT Edge (entre 1 y 129 caracteres). <br>* %m -> nombre del módulo (entre 1 y 64 caracteres). <br>* %c -> nombre del contenedor de origen (entre 3 y 63 caracteres). <br><br>El tamaño máximo del nombre del contenedor es de 63 caracteres; se asigna automáticamente el nombre del contenedor de destino si el tamaño del contenedor supera los 63 caracteres y se recortará cada sección (IoTHubName, IotEdgeDeviceID, ModuleName, SourceContainerName) a 15 caracteres. <br><br> Variable de entorno: `deviceToCloudUploadProperties__storageContainersForUpload__<sourceName>__target=<targetName>` |
| deleteAfterUpload | true, false | Se establece en `false` de forma predeterminada. Cuando se establece en `true`, se eliminarán automáticamente los datos cuando finalice la carga al almacenamiento en la nube. <br><br> **PRECAUCIÓN**: Si usa blobs en anexos, esta configuración eliminará los blobs en anexos del almacenamiento local después de una carga correcta y se producirá un error en cualquier operación posterior de anexión de bloques en esos blobs. Use esta configuración con precaución, no habilite esta opción si la aplicación realiza operaciones de anexión con poca frecuencia o no admite operaciones de anexión continuas.<br><br> Variable de entorno: `deviceToCloudUploadProperties__deleteAfterUpload={false,true}`. |

### <a name="deviceautodeleteproperties"></a>deviceAutoDeleteProperties

El nombre de este valor es `deviceAutoDeleteProperties`. Si usa el simulador de IoT Edge, establezca los valores en las variables de entorno relacionadas de estas propiedades, que puede encontrar en la sección de explicación.

| Propiedad | Valores posibles | Explicación |
| ----- | ----- | ---- |
| deleteOn | true, false | Se establece en `false` de forma predeterminada. Si quiere activar la característica, establezca este campo en `true`. <br><br> Variable de entorno: `deviceAutoDeleteProperties__deleteOn={false,true}` |
| deleteAfterMinutes | `<minutes>` | Especifique el tiempo en minutos. El módulo eliminará automáticamente los blobs del almacenamiento local cuando este valor expire. <br><br> Variable de entorno: `deviceAutoDeleteProperties__ deleteAfterMinutes=<minutes>` |
| retainWhileUploading | true, false | De forma predeterminada, se establece en `true`, y mantendrá el blob mientras se está cargando en el almacenamiento en la nube si expira deleteAfterMinutes. Puede configurarlo en `false` y eliminará los datos tan pronto como expire deleteAfterMinutes. Nota: Para que esta propiedad funcione, uploadOn se debe establecer en true.  <br><br> **PRECAUCIÓN**: Si usa blobs en anexos, esta configuración eliminará los blobs en anexos del almacenamiento local cuando el valor expira y se producirá un error en cualquier operación posterior de anexión de bloques en esos blobs. Es posible que desee asegurarse de que el valor de expiración es lo suficientemente grande para la frecuencia esperada de las operaciones de anexión realizadas por la aplicación.<br><br> Variable de entorno: `deviceAutoDeleteProperties__retainWhileUploading={false,true}`|

## <a name="using-smb-share-as-your-local-storage"></a>Uso de un recurso compartido de SMB como almacenamiento local

Cuando implemente el contenedor de Windows de este módulo en el host de Windows puede proporcionar un recurso compartido de SMB como ruta de acceso de almacenamiento local.

Asegúrese de que el recurso compartido de SMB y el dispositivo de IoT se encuentran en dominios de confianza mutua.

Puede ejecutar el comando de PowerShell `New-SmbGlobalMapping` para asignar el recurso compartido de SMB localmente en el dispositivo IoT que ejecuta Windows.

Estos son los pasos de la configuración:

```PowerShell
$creds = Get-Credential
New-SmbGlobalMapping -RemotePath <remote SMB path> -Credential $creds -LocalPath <Any available drive letter>
```

Por ejemplo:

```powershell
$creds = Get-Credential
New-SmbGlobalMapping -RemotePath \\contosofileserver\share1 -Credential $creds -LocalPath G:
```

Este comando usará las credenciales para autenticarse con el servidor de SMB remoto. A continuación, asigne la ruta de acceso del recurso compartido remoto a la letra de unidad G: (puede ser cualquier otra letra de unidad disponible). El dispositivo IoT ahora tiene el volumen de datos asignado a una ruta de acceso en la unidad G:.

Asegúrese de que el usuario en el dispositivo IoT puede leer y escribir en el recurso compartido de SMB remoto.

Para su implementación, el valor de `<storage mount>` puede ser **G:/ContainerData: C:/BlobRoot**.

## <a name="granting-directory-access-to-container-user-on-linux"></a>Concesión de acceso al directorio al usuario del contenedor en Linux

Si ha usado el [montaje de volúmenes](https://docs.docker.com/storage/volumes/) para el almacenamiento en sus opciones de creación para contenedores de Linux, no tiene que realizar ningún paso adicional, pero si ha usado el [montaje de enlace](https://docs.docker.com/storage/bind-mounts/), estos pasos son necesarios para ejecutar el servicio correctamente.

Siguiendo el principio de privilegios mínimos para limitar los derechos de acceso de los usuarios a los permisos mínimos que necesitan para realizar su trabajo, este módulo incluye un usuario (nombre: absie, ID: 11000) y un grupo de usuarios (nombre: absie, ID: 11000). Si el contenedor se inicia como **raíz** (el usuario predeterminado es **raíz**), el servicio se iniciará como el usuario **absie** con privilegios bajos.

Este comportamiento hace que la configuración de los permisos en la ruta de acceso de host sea crucial para que el servicio funcione correctamente, ya que, de lo contrario, el servicio se bloqueará con errores de acceso denegado. La ruta de acceso que se usa en el enlace de directorio debe ser accesible para el usuario del contenedor (por ejemplo, absie 11000). Puede conceder al usuario del contenedor acceso al directorio ejecutando los comandos siguientes en el host:

```terminal
sudo chown -R 11000:11000 <blob-dir>
sudo chmod -R 700 <blob-dir>
```

Por ejemplo:

```terminal
sudo chown -R 11000:11000 /srv/containerdata
sudo chmod -R 700 /srv/containerdata
```

Si necesita ejecutar el servicio como un usuario distinto de **absie**, puede especificar el identificador de usuario personalizado en createOptions en la propiedad "User" del manifiesto de implementación. En tal caso, debe usar el identificador `0` predeterminado o de grupo raíz.

```json
"createOptions": {
  "User": "<custom user ID>:0"
}
```

Ahora, conceda al usuario del contenedor acceso al directorio.

```terminal
sudo chown -R <user ID>:<group ID> <blob-dir>
sudo chmod -R 700 <blob-dir>
```

## <a name="configure-log-files"></a>Configurar los archivos de registro

Para obtener información sobre la configuración de los archivos de registro para su módulo, consulte estos [procedimientos recomendados de producción](https://docs.microsoft.com/azure/iot-edge/production-checklist#set-up-logs-and-diagnostics).

## <a name="connect-to-your-blob-storage-module"></a>Conexión a un módulo de Blob Storage

Puede usar el nombre y la clave de cuenta que configuró para el módulo para acceder al almacenamiento de blobs en el dispositivo de IoT Edge.

Especifique el dispositivo de IoT Edge como el punto de conexión del blob para todas las solicitudes de almacenamiento que realice en él. Puede [crear una cadena de conexión para un punto de conexión de almacenamiento explícito](../storage/common/storage-configure-connection-string.md#create-a-connection-string-for-an-explicit-storage-endpoint) mediante la información del dispositivo de IoT Edge y el nombre de cuenta que configuró.

* Para los módulos que se implementan en el mismo dispositivo donde se ejecuta Azure Blob Storage en el módulo IoT Edge, el punto final de blob es: `http://<module name>:11002/<account name>`.
* En el caso de los módulos o aplicaciones que se ejecutan en otro dispositivo, tiene que elegir el punto de conexión adecuado para su red. En función de la configuración de la red, elija un formato de punto de conexión, de modo que el tráfico de datos de su módulo o aplicación externo pueda llegar al dispositivo que ejecuta Azure Blob Storage en el módulo de IoT Edge. El punto de conexión de blob para este escenario es uno de los siguientes:
  * `http://<device IP >:11002/<account name>`
  * `http://<IoT Edge device hostname>:11002/<account name>`
  * `http://<fully qualified domain name>:11002/<account name>`
 
 > [!IMPORTANT]
 > Azure IoT Edge distingue mayúsculas y minúsculas al realizar llamadas a los módulos y el SDK de Storage también toma como valor predeterminado la minúsculas. Aunque el nombre del módulo en [Azure Marketplace](how-to-deploy-modules-portal.md#deploy-modules-from-azure-marketplace) es **AzureBlobStorageonIoTEdge**, cambiar el nombre a minúsculas ayuda a garantizar que las conexiones al módulo Azure Blob Storage en IoT Edge no se interrumpan.
 
## <a name="azure-blob-storage-quickstart-samples"></a>Ejemplos de inicio rápido de Azure Blob Storage

La documentación de Azure Blob Storage incluye guías de inicio rápido que proporcionan ejemplos de código en varios idiomas. Puede ejecutar estos ejemplos para probar Azure Blob Storage en IoT Edge cambiando el punto de conexión del blob para que apunte al módulo local de Blob Storage.

En las siguientes guías de inicio rápido se usan lenguajes que también son compatibles con IoT Edge, por lo que se pueden implementar como módulos de IoT Edge, junto con el módulo de Blob Storage:

* [.NET](../storage/blobs/storage-quickstart-blobs-dotnet.md)
* [Python](../storage/blobs/storage-quickstart-blobs-python.md)
  * Las versiones anteriores a la versión 2.1 del SDK de Python tienen un problema conocido que hace que el modulo no devuelva la hora de creación del blob. Debido a ese problema, algunos métodos como la enumeración de blobs no funcionan. Como solución alternativa, establezca explícitamente la versión de la API en el cliente de blob en "2017-04-17". Ejemplo: `block_blob_service._X_MS_VERSION = '2017-04-17'`
  * [Ejemplo de blob en anexos](https://github.com/Azure/azure-storage-python/blob/master/samples/blob/append_blob_usage.py)
* [Node.js](../storage/blobs/storage-quickstart-blobs-nodejs-legacy.md)
* [JS/HTML](../storage/blobs/storage-quickstart-blobs-javascript-client-libraries-legacy.md)
* [Ruby](../storage/blobs/storage-quickstart-blobs-ruby.md)
* [Go](../storage/blobs/storage-quickstart-blobs-go.md)
* [PHP](../storage/blobs/storage-quickstart-blobs-php.md)

## <a name="connect-to-your-local-storage-with-azure-storage-explorer"></a>Conectarse al almacenamiento local con el Explorador de Azure Storage

Puede usar el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) para conectarse a su cuenta de almacenamiento local.

1. Descargue e instale el Explorador de Azure Storage.

1. Conéctese a Azure Storage mediante una cadena de conexión.

1. Proporcione la cadena de conexión: `DefaultEndpointsProtocol=http;BlobEndpoint=http://<host device name>:11002/<your local account name>;AccountName=<your local account name>;AccountKey=<your local account key>;`.

1. Siga los pasos para conectarse.

1. Cree el contenedor en su cuenta de almacenamiento local.

1. Inicie la carga de archivos como blobs en bloques o blobs en anexos.
   > [!NOTE]
   > Este módulo no es compatible con blobs en páginas.

1. También puede conectarse a las cuentas de Azure Storage en el Explorador de Storage. Esta configuración le proporcionará una vista única de la cuenta de almacenamiento local y la cuenta de Azure Storage.

## <a name="supported-storage-operations"></a>Operaciones de Blob Storage

Los módulos de Blob Storage en IoT Edge usan los SDK de Azure Storage y son coherentes con la versión 2017-04-17 de la API de Azure Storage para los puntos de conexión del blob en bloques.

No todas las operaciones de Azure Blob Storage son compatibles con Azure Blob Storage en IoT Edge, así que esta sección enumera el estado de cada una.

### <a name="account"></a>Cuenta

Compatible:

* Enumerar contenedores

No compatible:

* Obtener y definir las propiedades del servicio Blob
* Preparar solicitud de blob
* Obtener estadísticas del servicio Blob
* Obtención de información de la cuenta

### <a name="containers"></a>Contenedores

Compatible:

* Crear y eliminar contenedor
* Obtener de propiedades y metadatos del contenedor
* Enumeración de blobs
* Obtener y definir lista de control de acceso de contenedor
* Establecer metadatos de contenedor

No compatible:

* Conceder contenedor

### <a name="blobs"></a>Datos BLOB

Compatible:

* Poner, obtener y eliminar blob
* Obtener y definir propiedades de blob
* Obtener y definir metadatos de blob

No compatible:

* Conceder blob
* Instantánea de blob
* Copiar y anular blob de copia
* Recuperar blob
* Establecer nivel de blob

### <a name="block-blobs"></a>Blobs en bloques

Compatible:

* Colocar bloque
* Colocar y obtener lista de bloques

No compatible:

* Colocar bloque desde dirección URL

### <a name="append-blobs"></a>Blobs en anexos

Compatible:

* Anexar bloque

No compatible:

* Anexar bloque desde dirección URL

## <a name="event-grid-on-iot-edge-integration"></a>Integración de Event Grid en IoT Edge

> [!CAUTION]
> Integración de Event Grid en IoT Edge se encuentra en versión preliminar

Ahora el Azure Blob Storage en el módulo de IoT Edge proporciona integración con Event Grid en IoT Edge. Para obtener información detallada sobre esta integración, consulte el [tutorial para implementar los módulos, publicar eventos y comprobar la entrega de eventos](../event-grid/edge/react-blob-storage-events-locally.md).

## <a name="release-notes"></a>Notas de la versión

Aquí están las [notas de la versión de Docker Hub](https://hub.docker.com/_/microsoft-azure-blob-storage) para este módulo

## <a name="suggestions"></a>Sugerencias

Su opinión es muy importante para que este módulo y sus características sean útiles y fáciles de usar. Comparta sus comentarios y háganos saber cómo podemos mejorar.

Puede ponerse en contacto con nosotros en absiotfeedback@microsoft.com.

## <a name="next-steps"></a>Pasos siguientes

Aprenda a [implementar Azure Blob Storage en IoT Edge](how-to-deploy-blob.md)

Permanezca actualizado con los anuncios y actualizaciones recientes de [Azure Blob Storage en el blog de IoT Edge](https://aka.ms/abs-iot-blogpost).
