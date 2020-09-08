---
title: Uso del Emulador de Azure Storage para desarrollo y pruebas
description: El Emulador de Azure Storage ofrece un entorno de desarrollo local gratuito para desarrollo y pruebas de las aplicaciones de Azure Storage.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 07/16/2020
ms.service: storage
ms.subservice: common
ms.topic: how-to
ms.openlocfilehash: 959a58a38861075c6509fe57136d8991eeb98ce6
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88588202"
---
# <a name="use-the-azure-storage-emulator-for-development-and-testing"></a>Uso del Emulador de Azure Storage para desarrollo y pruebas

El Emulador de Microsoft Azure Storage es una herramienta que emula los servicios Azure Blob, Queue y Table para fines de desarrollo local. Puede probar la aplicación en los servicios de almacenamiento local, sin crear una suscripción de Azure ni incurrir en ningún gasto. Cuando esté satisfecho con el funcionamiento de la aplicación en el emulador, cambie al uso de una cuenta de almacenamiento de Azure en la nube.

> [!IMPORTANT]
> El emulador de Azure Storage ya no se está desarrollando activamente. [**Azurite**](storage-use-azurite.md) es la futura plataforma del emulador de almacenamiento. Azurite reemplaza al emulador de Azure Storage. Azurite se seguirá actualizando para admitir las versiones más recientes de las API de Azure Storage. Para más información, consulte [**Uso del emulador de Azurite para desarrollo y pruebas locales de Azure Storage**](storage-use-azurite.md).

## <a name="get-the-storage-emulator"></a>Obtención del emulador de almacenamiento

El emulador de almacenamiento se incluye como parte del [SDK de Microsoft Azure](https://azure.microsoft.com/downloads/). También puede instalar el emulador de almacenamiento mediante el [instalador independiente](https://go.microsoft.com/fwlink/?linkid=717179&clcid=0x409) (descarga directa). Para instalar el emulador de almacenamiento, debe tener privilegios de administrador en el equipo.

El emulador de almacenamiento solo se ejecuta actualmente en Windows. Si necesita uno para Linux, una opción es [Azurite](https://github.com/azure/azurite), el emulador de almacenamiento de código abierto mantenido por la comunidad.

> [!NOTE]
> No se garantiza que los datos que se crean en una versión del emulador de almacenamiento estén disponibles cuando se utilice una versión diferente. Si necesita conservar los datos a largo plazo, es recomendable que almacene esos datos en una cuenta de almacenamiento de Azure y no en el mismo emulador de almacenamiento.
> 
> El emulador de almacenamiento depende de versiones específicas de las bibliotecas de OData. No se admite la sustitución de las DLL de OData usadas por el emulador de almacenamiento con otras versiones y, de ser así, puede provocar un comportamiento inesperado. Sin embargo, cualquier versión de OData admitida por el servicio de almacenamiento puede utilizarse para enviar las solicitudes al emulador.

## <a name="how-the-storage-emulator-works"></a>Cómo funciona el emulador de almacenamiento.

El emulador de almacenamiento usa una instancia de Microsoft SQL Server 2012 Express LocalDB para emular los servicios de almacenamiento de Azure. Puede configurar el emulador de almacenamiento para acceder a una instancia local de SQL Server en lugar de una de LocalDB. Consulte la sección [Iniciar e inicializar el emulador de almacenamiento](#start-and-initialize-the-storage-emulator) más adelante en este artículo para más información.

El emulador de almacenamiento se conecta a SQL Server o LocalDB mediante la autenticación de Windows.

Existen algunas diferencias de funcionalidad entre el emulador de almacenamiento y los servicios de almacenamiento de Azure. Para más información sobre estas diferencias, consulte la sección [Diferencias entre el emulador de almacenamiento y Azure Storage](#differences-between-the-storage-emulator-and-azure-storage).

## <a name="start-and-initialize-the-storage-emulator"></a>Inicio e inicialización del emulador de almacenamiento

Para iniciar el Emulador de Azure Storage:

1. Haga clic en el botón **Inicio** o pulse la tecla **Windows**.
2. Comience a escribir `Azure Storage Emulator`.
3. Seleccione el emulador de la lista de aplicaciones mostradas.

Cuando se inicie el emulador de almacenamiento, aparecerá una ventana de símbolo del sistema. Puede usar esta ventana de la consola para iniciar y detener el emulador de almacenamiento. También puede borrar datos, obtener el estado e inicializar el emulador desde el símbolo del sistema. Para más información, consulte la sección [Referencia de la herramienta de la línea de comandos del emulador de almacenamiento](#storage-emulator-command-line-tool-reference) más adelante en este artículo.

> [!NOTE]
> Es posible que el Emulador de Azure Storage no se inicie correctamente si otro emulador de almacenamiento, como Azurite, se está ejecutando en el sistema.

Cuando se ejecuta el emulador, verá un icono en el área de notificación de la barra de tareas de Windows.

Al cerrar la ventana de símbolo del sistema del emulador de almacenamiento, el emulador de almacenamiento continuará funcionando. Para abrir de nuevo la ventana de la consola del emulador de almacenamiento, siga los pasos anteriores como si fuera a iniciar este.

Al ejecutar por primera vez el emulador de almacenamiento, el entorno de almacenamiento local se inicia automáticamente. El proceso de inicialización crea una base de datos en LocalDB y se reserva puertos HTTP para cada servicio de almacenamiento local.

El emulador de almacenamiento está instalado de forma predeterminada en `C:\Program Files (x86)\Microsoft SDKs\Azure\Storage Emulator`.

> [!TIP]
> Puede usar el [Explorador de Microsoft Azure Storage](https://storageexplorer.com) para trabajar con recursos locales del emulador de almacenamiento. Después de haber instalado e iniciado el emulador de almacenamiento, busque "(Emulador: Puertos predeterminados) (Clave)" en "Local &Attached" (Local y acoplado), en el árbol de recursos del Explorador de Storage.
>

### <a name="initialize-the-storage-emulator-to-use-a-different-sql-database"></a>Inicialización del emulador de almacenamiento para usar otra base de datos SQL

La herramienta de la línea de comandos del emulador de almacenamiento se puede usar para inicializar el emulador de almacenamiento para que señale a una instancia de base de datos SQL distinta a la instancia LocalDB predeterminada:

1. Abra la ventana de la consola del emulador de almacenamiento como se describe en la sección [Inicio e inicialización del emulador de almacenamiento](#start-and-initialize-the-storage-emulator).
1. En la ventana de consola, escriba el siguiente comando, donde `<SQLServerInstance>` es el nombre de la instancia de SQL Server. Para usar LocalDb, especifique `(localdb)\MSSQLLocalDb` como instancia de SQL Server.

   `AzureStorageEmulator.exe init /server <SQLServerInstance>`

   También puede usar el siguiente comando, el cual indicará al emulador que utilice la instancia predeterminada de SQL Server:

   `AzureStorageEmulator.exe init /server .`

   O bien, puede usar el comando siguiente, que inicializa la base de datos a la instancia de LocalDB predeterminada:

   `AzureStorageEmulator.exe init /forceCreate`

Para más información sobre estos comandos, consulte la sección [Referencia de la herramienta de la línea de comandos del emulador de almacenamiento](#storage-emulator-command-line-tool-reference).

> [!TIP]
> Puede usar [Microsoft SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS) para administrar las instancias de SQL Server, incluida la instalación de LocalDB. En el cuadro de diálogo **Conectar al servidor** de SMSS, especifique `(localdb)\MSSQLLocalDb` en el campo **Nombre del servidor:** para conectarse a la instancia de LocalDB.

## <a name="authenticating-requests-against-the-storage-emulator"></a>Autenticación de solicitudes en el emulador de almacenamiento

Cuando haya instalado e iniciado el emulador de almacenamiento, puede probar el código en él. Todas las solicitudes que realice en el emulador de almacenamiento se deben autorizar, a menos que sea una solicitud anónima. Puede autorizar solicitudes en el emulador de almacenamiento mediante la autenticación de clave compartida o con una firma de acceso compartido (SAS).

### <a name="authorize-with-shared-key-credentials"></a>Autorización con credenciales de clave compartida

[!INCLUDE [storage-emulator-connection-string-include](../../../includes/storage-emulator-connection-string-include.md)]

Para más información sobre las cadenas de conexión, consulte [Configuración de las cadenas de conexión de Azure Storage](../storage-configure-connection-string.md).

### <a name="authorize-with-a-shared-access-signature"></a>Autorización con una firma de acceso compartido

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Algunas bibliotecas de cliente de almacenamiento de Azure, como la biblioteca Xamarin, solo admiten la autenticación con un token de firma de acceso compartido (SAS). Puede crear el token de SAS mediante el [Explorador de Storage](https://storageexplorer.com/) u otra aplicación que admita la autenticación de clave compartida.

También puede generar un token de SAS mediante Azure PowerShell. En el ejemplo siguiente se genera un token de SAS con permisos completos a un contenedor de blobs:

1. Instale Azure PowerShell si no lo ha hecho ya (se recomienda el uso de la versión más reciente de los cmdlets de Azure PowerShell). Para ver las instrucciones de instalación, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/install-Az-ps).
2. Abra Azure PowerShell y ejecute los siguientes comandos, sustituyendo `CONTAINER_NAME` por un nombre de su elección:

```powershell
$context = New-AzStorageContext -Local

New-AzStorageContainer CONTAINER_NAME -Permission Off -Context $context

$now = Get-Date

New-AzStorageContainerSASToken -Name CONTAINER_NAME -Permission rwdl -ExpiryTime $now.AddDays(1.0) -Context $context -FullUri
```

El identificador URI de la firma de acceso compartido resultante para el nuevo contenedor debe ser similar a:

```
http://127.0.0.1:10000/devstoreaccount1/sascontainer?sv=2012-02-12&se=2015-07-08T00%3A12%3A08Z&sr=c&sp=wl&sig=t%2BbzU9%2B7ry4okULN9S0wst%2F8MCUhTjrHyV9rDNLSe8g%3Dsss
```

La firma de acceso compartido creada con este ejemplo es válida durante un día. La firma concede acceso completo (lectura, escritura, eliminación, enumeración) a blobs situados dentro del contenedor.

Para obtener más información sobre las firmas de acceso compartido, consulte [Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido (SAS)](storage-sas-overview.md).

## <a name="addressing-resources-in-the-storage-emulator"></a>Direccionamiento de los recursos en el emulador de almacenamiento

Los puntos de conexión de servicio del emulador de almacenamiento son diferentes de los de una cuenta de almacenamiento de Azure. El equipo local no realiza la resolución de nombres de dominio, lo que requiere que los puntos de conexión del emulador de almacenamiento sean direcciones locales.

Al direccionar un recurso en una cuenta de Azure Storage, usará el siguiente esquema. El nombre de cuenta forma parte del nombre de host del identificador URI y el recurso que se redirige forma parte de la ruta de acceso del identificador URI:

`<http|https>://<account-name>.<service-name>.core.windows.net/<resource-path>`

Por ejemplo, el siguiente URI es una dirección válida para un blob en una cuenta de almacenamiento de Azure:

`https://myaccount.blob.core.windows.net/mycontainer/myblob.txt`

Como el equipo local no realiza la resolución de nombres de dominio, el nombre de la cuenta forma parte de la ruta de acceso del URI en lugar del nombre de host. Use el siguiente formato de URI en un recurso del emulador de almacenamiento:

`http://<local-machine-address>:<port>/<account-name>/<resource-path>`

Por ejemplo, la siguiente dirección se puede usar para obtener acceso a un blob en el emulador de almacenamiento:

`http://127.0.0.1:10000/myaccount/mycontainer/myblob.txt`

Los puntos de conexión de servicio del emulador de almacenamiento son:

* Blob service: `http://127.0.0.1:10000/<account-name>/<resource-path>`
* Queue service: `http://127.0.0.1:10001/<account-name>/<resource-path>`
* Table service: `http://127.0.0.1:10002/<account-name>/<resource-path>`

### <a name="addressing-the-account-secondary-with-ra-grs"></a>Direccionar la cuenta secundaria con RA-GRS

A partir de la versión 3.1, el emulador de almacenamiento admite la replicación con redundancia geográfica con acceso de lectura (RA-GRS). Puede acceder a la ubicación secundaria si anexa -secondary al nombre de la cuenta. Por ejemplo, la siguiente dirección se puede usar para obtener acceso a un blob usando la cuenta secundaria de solo lectura del emulador de almacenamiento:

`http://127.0.0.1:10000/myaccount-secondary/mycontainer/myblob.txt`

> [!NOTE]
> Para el acceso mediante programación a la cuenta secundaria con el emulador de almacenamiento, usa la biblioteca de clientes de almacenamiento para la versión 3.2 de .NET o una versión posterior. Consulte la [biblioteca del cliente de Microsoft Azure Storage para .NET](https://msdn.microsoft.com/library/azure/dn261237.aspx) si desea obtener más información.
>
>

## <a name="storage-emulator-command-line-tool-reference"></a>Referencia de la herramienta de la línea de comandos del emulador de almacenamiento

A partir de la versión 3.0, se muestra una ventana de consola cuando se inicia el emulador de almacenamiento. Use la línea de comandos en la ventana de la consola para iniciar y detener el emulador. También puede consultar el estado y realizar otras operaciones desde la línea de comandos.

> [!NOTE]
> Si tiene instalado el emulador de proceso de Microsoft Azure, aparece un icono de la bandeja del sistema al iniciar el emulador de almacenamiento. Haga clic con el botón derecho en el icono para abrir un menú que ofrece en forma de gráficos las opciones de iniciar y detener el emulador de almacenamiento.
>
>

### <a name="command-line-syntax"></a>Sintaxis de línea de comandos

`AzureStorageEmulator.exe [start] [stop] [status] [clear] [init] [help]`

### <a name="options"></a>Opciones

Para ver la lista de opciones, escriba `/help` en el símbolo del sistema.

| Opción | Descripción | Comando | Argumentos |
| --- | --- | --- | --- |
| **Iniciar** |Inicia el emulador de almacenamiento. |`AzureStorageEmulator.exe start [-inprocess]` |*-Reprocess*: inicia el emulador en el proceso actual en lugar de crear un nuevo proceso. |
| **Detención** |Detiene el emulador de almacenamiento. |`AzureStorageEmulator.exe stop` | |
| **Estado** |Imprime el estado del emulador de almacenamiento. |`AzureStorageEmulator.exe status` | |
| **Borrar** |Borra los datos de todos los servicios especificados en la línea de comandos. |`AzureStorageEmulator.exe clear [blob] [table] [queue] [all]` |*blob*: borra datos del blob. <br/>*queue*: borra datos de la cola. <br/>*table*: borra datos de la tabla. <br/>*all*: borra todos los datos de todos los servicios. |
| **Init** |Realiza una inicialización única para configurar el emulador. |<code>AzureStorageEmulator.exe init [-server serverName] [-sqlinstance instanceName] [-forcecreate&#124;-skipcreate] [-reserveports&#124;-unreserveports] [-inprocess]</code> |*-server serverName\instanceName*: especifica el servidor que hospeda la instancia de SQL. <br/>*-sqlinstance instanceName*: especifica el nombre de la instancia de SQL que se usará en la instancia de servidor predeterminada. <br/>*-forcecreate*: fuerza la creación de la instancia de SQL Database, aunque ya exista. <br/>*-skipcreate*: mite la creación de la instancia de SQL Database. Esto tiene prioridad sobre -forcecreate.<br/>*-reserveports*: intenta reservar los puertos HTTP asociados con los servicios.<br/>*-unreserveports*: intenta quitar las reservas de los puertos HTTP asociados con los servicios. Esto tiene prioridad sobre -reserveports.<br/>*-inprocess*: realiza la inicialización en el proceso actual, en lugar de generar un proceso nuevo. Tiene que iniciar el proceso actual con permisos elevados si se cambian las reservas de puerto. |

## <a name="differences-between-the-storage-emulator-and-azure-storage"></a>Diferencias entre el emulador de almacenamiento y Azure Storage

Dado que el emulador de almacenamiento es un entorno emulado local, existen diferencias entre el uso del emulador y una cuenta de almacenamiento de Azure en la nube:

* El emulador de almacenamiento es compatible con una sola cuenta fija y una clave de autenticación ya conocida.
* El emulador de almacenamiento no es un servicio de almacenamiento escalable y no es compatible con un gran número de clientes simultáneos.
* Como se describe en [Direccionamiento de los recursos en el emulador de almacenamiento](#addressing-resources-in-the-storage-emulator), los recursos se tratan de forma diferente en el emulador de almacenamiento que en una cuenta de almacenamiento de Azure. La diferencia se debe a que la resolución de nombres de dominio está disponible en la nube, pero no en el equipo local.
* A partir de la versión 3.1, la cuenta del emulador de almacenamiento admite la replicación con redundancia geográfica con acceso de lectura (RA-GRS). En el emulador, todas las cuentas tienen RA-GRS habilitado y no hay ningún retraso entre las réplicas principal y secundaria. Las operaciones Get Blob Service Stats, Get Queue Service Stats y Get Table Service Stats son compatibles con la cuenta secundaria y siempre devolverán el valor del elemento de respuesta `LastSyncTime` como la hora actual según la base de datos SQL subyacente.
* El servicio de archivo y los puntos de conexión de servicio de protocolo SMB no se admiten actualmente en el emulador de almacenamiento.
* Si usa una versión de los servicios de almacenamiento que no es compatible con el emulador, este devuelve un error VersionNotSupportedByEmulator (código de estado HTTP 400 - Solicitud incorrecta).

### <a name="differences-for-blob-storage"></a>Diferencias en el almacenamiento de blobs

Las siguientes diferencias se aplican al almacenamiento de blobs en el emulador:

* El emulador de almacenamiento solo admite tamaños de blobs de hasta 2 GB.
* La longitud máxima de un nombre de blob en el emulador de almacenamiento es de 256 caracteres, mientras que en Azure Storage es de 1024.
* La copia incremental permite copiar instantáneas desde los blobs sobrescritos, lo que devuelve un error en el servicio.
* Get Page Ranges Diff no funciona entre instantáneas copiadas con el blob de copia incremental.
* Una operación Put Blob puede funcionar en un blob que existe en el emulador de almacenamiento con una concesión activa, incluso si el identificador de concesión no se ha especificado en la solicitud.
* El emulador no admite operaciones de blob en anexos. Intentando realizar una operación en un blob en anexos, devuelve un error FeatureNotSupportedByEmulator (código de estado HTTP 400 - Solicitud incorrecta).

### <a name="differences-for-table-storage"></a>Diferencias en el almacenamiento de tabla

Las siguientes diferencias se aplican al almacenamiento de tablas en el emulador:

* Las propiedades de fecha en Table service del emulador de almacenamiento solo admiten el intervalo admitido por SQL Server 2005 (es decir, tienen que ser posteriores al 1 de enero de 1753). Todas las fechas anteriores al 1 de enero de 1753 se cambiarán a este valor. La precisión de las fechas se limita a la precisión de SQL Server 2005, lo que significa que las fechas son precisas hasta 1/300 de un segundo.
* El emulador de almacenamiento admite valores de propiedad de clave de fila y de clave de partición de menos de 512 bytes. El tamaño total del nombre de la cuenta, el nombre de la tabla y los nombres de las propiedades de clave no pueden superar en conjunto los 900 bytes.
* El tamaño total de una fila de una tabla en el emulador de almacenamiento se limita a menos de 1 MB.
* En el emulador de almacenamiento, las propiedades de tipo de datos `Edm.Guid` o `Edm.Binary` solo admiten los operadores de comparación `Equal (eq)` y `NotEqual (ne)` en las cadenas de filtro de consultas.

### <a name="differences-for-queue-storage"></a>Diferencias en el almacenamiento de cola

No hay ninguna diferencia específica del almacenamiento en cola en el emulador.

## <a name="storage-emulator-release-notes"></a>Notas de la versión del emulador de almacenamiento

### <a name="version-510"></a>Versión 5.10

* El emulador de almacenamiento no rechazará la versión 2019-07-07 de los servicios de almacenamiento en los puntos de conexión de servicio de Blob, Queue y Table.

### <a name="version-59"></a>Versión 5.9

* El emulador de almacenamiento no rechazará la versión 2019-02-02 de los servicios de almacenamiento en los puntos de conexión de servicio de Blob, Queue y Table.

### <a name="version-58"></a>Versión 5.8

* El emulador de almacenamiento no rechazará la versión 2018-11-09 de los servicios de almacenamiento en los puntos de conexión de servicio de Blob, Queue y Table.

### <a name="version-57"></a>Versión 5.7

* Se ha corregido un error que podría ocasionar un bloqueo si el registro estaba habilitado.

### <a name="version-56"></a>Versión 5.6

* El emulador de almacenamiento admite ahora la versión 2018-03-28 de los servicios de almacenamiento en los puntos de conexión de Blob, Queue y Table.

### <a name="version-55"></a>Versión 5.5

* El emulador de almacenamiento admite ahora la versión 2017-11-09 de los servicios de almacenamiento en los puntos de conexión de Blob, Queue y Table.
* Se ha agregado compatibilidad para la propiedad **Created** del blob, que devuelve la hora de creación de este.

### <a name="version-54"></a>Versión 5.4

* Para mejorar la estabilidad de la instalación, el emulador ya no intenta reservar puertos en el momento de la instalación. Si quiere realizar las reservas de puertos, use la opción *-reserveports* del comando **init** para especificarlos.

### <a name="version-53"></a>Versión 5.3

* El emulador de almacenamiento admite ahora la versión 2017-07-29 de los servicios de almacenamiento en los puntos de conexión de servicio de Blob, Queue y Table.

### <a name="version-52"></a>Versión 5.2

* El emulador de almacenamiento admite ahora la versión 2017-04-17 de los servicios de almacenamiento en los puntos de conexión de servicio de Blob, Queue y Table.
* Se ha corregido un error por el que no se codificaban correctamente los valores de propiedad de tabla.

### <a name="version-51"></a>Versión 5.1

* Se ha corregido un error que hacía que el emulador de almacenamiento devolviera el encabezado `DataServiceVersion` en algunas respuestas en las que el servicio no lo hacía.

### <a name="version-50"></a>Versión 5.0

* El instalador del emulador de almacenamiento ya no comprueba instalaciones existentes de MSSQL y .NET Framework.
* El instalador del emulador de almacenamiento ya no crea la base de datos como parte de la instalación. Se seguirá creando la base de datos si se necesita como parte del proceso de inicio.
* La creación de la base de datos ya no requiere elevación.
* Las reservas de puertos ya no son necesarias para el inicio.
* Agrega las siguientes opciones a `init`: `-reserveports` (requiere elevación), `-unreserveports` (requiere elevación), `-skipcreate`.
* La opción de interfaz de usuario Emulador de almacenamiento en el icono de la bandeja del sistema ahora inicia la interfaz de línea de comandos. La interfaz gráfica de usuario anterior ya no está disponible.
* Algunos archivos DLL se han quitado o se han cambiado de nombre.

### <a name="version-46"></a>Versión 4.6

* El emulador de almacenamiento admite ahora la versión 2016-05-31 de los servicios de almacenamiento en los puntos de conexión de servicio de Blob, Queue y Table.

### <a name="version-45"></a>Versión 4.5

* Se corrigió un error que provocaba un error en la instalación e inicialización cuando se cambia el nombre de la base de datos auxiliar.

### <a name="version-44"></a>Version 4.4

* El emulador de almacenamiento admite ahora la versión 2015-12-11 de los servicios de almacenamiento en los puntos de conexión de servicio de Blob, Queue y Table.
* La recolección de elementos no utilizados de datos de blobs del emulador de almacenamiento ahora es más eficaz cuando se trabaja con grandes cantidades de blobs.
* Se ha corregido un error que provocaba que el XML de ACL del contenedor se validara de forma algo diferente de cómo lo hace el servicio de almacenamiento.
* Se ha corregido un error que a veces causaba que los valores máx. y mín. de DateTime se notificarán en una zona horaria incorrecta.

### <a name="version-43"></a>Versión 4.3

* El emulador de almacenamiento admite ahora la versión 2015-07-08 de los servicios de almacenamiento en los puntos de conexión de servicio de Blob, Queue y Table.

### <a name="version-42"></a>Versión 4.2

* El emulador de almacenamiento admite ahora la versión 2015-04-05 de los servicios de almacenamiento en los puntos de conexión de servicio de Blob, Queue y Table.

### <a name="version-41"></a>Versión 4.1

* El emulador de almacenamiento admite ahora la versión 2015-02-21 de los servicios de almacenamiento en los puntos de conexión de servicio de Blob, Queue y Table. No es compatible con las nuevas características de blobs en anexos.
* El emulador ahora devuelve un mensaje de error significativo para las versiones no admitidas de los servicios de almacenamiento. Se recomienda usar la versión más reciente del emulador. Si se encuentra con un error VersionNotSupportedByEmulator (código de estado HTTP 400 - Solicitud incorrecta), descargue la versión más reciente del emulador.
* Se ha corregido un error en el que una condición de carrera hacía que los datos de entidad de tabla fuesen incorrectos durante las operaciones de mezcla simultáneas.

### <a name="version-40"></a>Versión 4.0

* Se ha cambiado el nombre del ejecutable del emulador de almacenamiento por *AzureStorageEmulator.exe*.

### <a name="version-32"></a>Versión 3.2

* El emulador de almacenamiento admite ahora la versión 2014-02-14 de los servicios de almacenamiento en los puntos de conexión de servicio de Blob, Queue y Table. Los puntos de conexión de servicio de archivos no se admiten actualmente en el emulador de almacenamiento. Consulte [Versiones de los servicios de Azure Storage](/rest/api/storageservices/Versioning-for-the-Azure-Storage-Services) para obtener información acerca de la versión 2014-02-14.

### <a name="version-31"></a>Versión 3.1

* Ahora se admite el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) en el emulador de almacenamiento. Las API `Get Blob Service Stats`, `Get Queue Service Stats` y `Get Table Service Stats` se admiten para la cuenta secundaria y siempre devolverán el valor del elemento de respuesta LastSyncTime como la hora actual según la base de datos SQL subyacente. Para el acceso mediante programación a la cuenta secundaria con el emulador de almacenamiento, usa la biblioteca de clientes de almacenamiento para la versión 3.2 de .NET o una versión posterior. Consulte la Documentación de referencia de la biblioteca cliente de Microsoft Azure Storage para .NET para obtener más información.

### <a name="version-30"></a>Versión 3.0

* El Emulador de Azure Storage ya no se incluye en el mismo paquete que el emulador de proceso.
* La interfaz gráfica de usuario del emulador de almacenamiento está en desuso. Se reemplazó por una interfaz de línea de comandos que admite scripts. Para obtener más información sobre la interfaz de la línea de comandos, vea Referencia de la herramienta de la línea de comandos del emulador de almacenamiento. La interfaz gráfica seguirá estando presente en la versión 3.0, pero solo se puede acceder a ella cuando se instala el emulador de proceso con el botón secundario en el icono de la bandeja del sistema y se selecciona la opción de mostrar IU del emulador de almacenamiento.
* La versión 2013-08-15 de los servicios de almacenamiento de Azure ahora es totalmente compatible. (Anteriormente esta versión solo era compatible con versión la versión 2.2.1 Preview del emulador de almacenamiento.)

## <a name="next-steps"></a>Pasos siguientes

* Evalúe [Azurite](https://github.com/azure/azurite), el emulador de almacenamiento de código abierto multiplataforma mantenido por la comunidad. 
* [Ejemplos de Azure Storage mediante .NET](../storage-samples-dotnet.md) contiene vínculos a varios ejemplos de código que puede usar al desarrollar su aplicación.
* Puede usar el [Explorador de Microsoft Azure Storage](https://storageexplorer.com) para trabajar con recursos de su cuenta de almacenamiento en la nube y del emulador de almacenamiento.

## <a name="see-also"></a>Consulte también

* [Desarrollo de Azure Storage local con Azurite, SDK de Azure y Explorador de Azure Storage](https://blog.jongallant.com/2020/04/local-azure-storage-development-with-azurite-azuresdks-storage-explorer/)
