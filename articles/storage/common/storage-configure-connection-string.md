---
title: Configuración de la cadena de conexión
titleSuffix: Azure Storage
description: Configure una cadena de conexión para una cuenta de Azure Storage. Una cadena de conexión contiene la información necesaria para autorizar el acceso a una cuenta de almacenamiento desde una aplicación en tiempo de ejecución mediante la autorización de clave compartida.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 07/16/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: 30e8bd7fda08795ecc1aa78b0d2507e7c915ad0d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87087261"
---
# <a name="configure-azure-storage-connection-strings"></a>Configuración de las cadenas de conexión de Azure Storage

Una cadena de conexión incluye la información de autorización que requiere la aplicación para acceder a los datos de una cuenta de Azure Storage en tiempo de ejecución mediante autorización de clave compartida. Las cadenas de conexión se pueden configurar para:

* Conectarse al emulador de almacenamiento Azurite.
* Acceder a la cuenta de Azure Storage.
* Acceder a recursos especificados en Azure a través de una firma de acceso compartido (SAS).

Para información sobre cómo ver las claves de acceso a la cuenta y copiar una cadena de conexión, consulte [Administración de las claves de acceso de la cuenta de almacenamiento](storage-account-keys-manage.md).

[!INCLUDE [storage-account-key-note-include](../../../includes/storage-account-key-note-include.md)]

## <a name="store-a-connection-string"></a>Almacenamiento de una cadena de conexión

La aplicación necesita acceder a la cadena de conexión en tiempo de ejecución para autorizar las solicitudes realizadas a Azure Storage. Tiene varias opciones para almacenar una cadena de conexión:

* Puede almacenar la cadena de conexión en una variable de entorno.
* Una aplicación que se ejecuta en el escritorio o en un dispositivo puede almacenar la cadena de conexión en un archivo **app.config** o **web.config**. Agregue la cadena de conexión a la sección **AppSettings** en estos archivos.
* Una aplicación que se ejecuta en un servicio en la nube de Azure puede almacenar la cadena de conexión en el [archivo de esquema de configuración de servicio de Azure (.cscfg)](https://msdn.microsoft.com/library/ee758710.aspx). Agregue la cadena de conexión a la sección **ConfigurationSettings** del archivo de configuración del servicio.

El almacenamiento de la cadena de conexión en un archivo de configuración facilita la actualización de la cadena de conexión para que alterne entre el [emulador de almacenamiento Azurite](../common/storage-use-azurite.md) y una cuenta de Azure Storage en la nube. Solo necesitará editar la cadena de conexión para apuntar al entorno de destino.

Puede usar el [Administrador de configuración de Microsoft Azure](https://www.nuget.org/packages/Microsoft.Azure.ConfigurationManager/) para acceder a la cadena de conexión en tiempo de ejecución independientemente de dónde se ejecute la aplicación.

## <a name="configure-a-connection-string-for-azurite"></a>Configuración de una cadena de conexión para Azurite

[!INCLUDE [storage-emulator-connection-string-include](../../../includes/storage-emulator-connection-string-include.md)]

Para obtener más información sobre Azurite, consulte [Uso del emulador de Azurite para el desarrollo local de Azure Storage](../common/storage-use-azurite.md).

## <a name="configure-a-connection-string-for-an-azure-storage-account"></a>Configuración de una cadena de conexión para una cuenta de Azure Storage

Para crear una cadena de conexión para una cuenta de Azure Storage, use el siguiente formato. Indique si desea conectarse a la cuenta de almacenamiento a través de HTTPS (recomendado) o HTTP, reemplace `myAccountName` por el nombre de la cuenta de almacenamiento y reemplace `myAccountKey` por la clave de acceso a la cuenta:

`DefaultEndpointsProtocol=[http|https];AccountName=myAccountName;AccountKey=myAccountKey`

Por ejemplo, la cadena de conexión podría ser similar a la siguiente:

`DefaultEndpointsProtocol=https;AccountName=storagesample;AccountKey=<account-key>`

Aunque Azure Storage admite HTTP y HTTPS en una cadena de conexión, *se recomienda encarecidamente utilizar HTTPS*.

> [!TIP]
> Las cadenas de conexión de la cuenta de almacenamiento se pueden encontrar en [Azure Portal](https://portal.azure.com). Navegue a **CONFIGURACIÓN** > **Claves de acceso** en la hoja del menú de la cuenta de almacenamiento para ver las cadenas de conexión de las claves de acceso principal y secundaria.
>

## <a name="create-a-connection-string-using-a-shared-access-signature"></a>Creación de una cadena de conexión con una firma de acceso compartido

[!INCLUDE [storage-use-sas-in-connection-string-include](../../../includes/storage-use-sas-in-connection-string-include.md)]

## <a name="create-a-connection-string-for-an-explicit-storage-endpoint"></a>Creación de una cadena de conexión para un punto de conexión de Storage explícito

En una cadena de conexión se pueden especificar puntos de conexión de servicio explícitos, en lugar de usar los predeterminados. Para crear una cadena de conexión que especifique un punto de conexión explícito, especifique el punto de conexión de servicio completo para cada servicio, incluida la especificación de protocolo (HTTPS (recomendado) o HTTP) con el formato siguiente:

```
DefaultEndpointsProtocol=[http|https];
BlobEndpoint=myBlobEndpoint;
FileEndpoint=myFileEndpoint;
QueueEndpoint=myQueueEndpoint;
TableEndpoint=myTableEndpoint;
AccountName=myAccountName;
AccountKey=myAccountKey
```

Un escenario en el que puede desear especificar un punto de conexión explícito es cuando ha asignado un punto de conexión de Blob Storage a un [dominio personalizado](../blobs/storage-custom-domain-name.md). En ese caso, puede especificar un punto de conexión personalizado para Blob Storage en la cadena de conexión. También puede especificar los puntos de conexión predeterminados para los demás servicios, en caso de que la aplicación los use.

Este es un ejemplo de una cadena de conexión que especifica un punto de conexión explícito para Blob service:

```
# Blob endpoint only
DefaultEndpointsProtocol=https;
BlobEndpoint=http://www.mydomain.com;
AccountName=storagesample;
AccountKey=<account-key>
```

Este ejemplo especifica puntos de conexión explícitos para todos los servicios, lo que incluye un dominio personalizado para Blob service:

```
# All service endpoints
DefaultEndpointsProtocol=https;
BlobEndpoint=http://www.mydomain.com;
FileEndpoint=https://myaccount.file.core.windows.net;
QueueEndpoint=https://myaccount.queue.core.windows.net;
TableEndpoint=https://myaccount.table.core.windows.net;
AccountName=storagesample;
AccountKey=<account-key>
```

Los valores del punto de conexión de una cadena de conexión se usan para construir los identificadores URI de solicitud para los servicios de Storage e indicar el formato de los identificadores URI que se devuelven al código.

Si ha asignado un punto de conexión de Storage a un dominio personalizado y omite dicho punto en una cadena de conexión, no podrá usarla para acceder a los datos de dicho servicio desde el código.

> [!IMPORTANT]
> Los valores del punto de conexión de servicio de las cadenas de conexión deben ser identificadores URI con el formato correcto, entre los que se incluyen `https://` (recomendado) o `http://`. Dado que Azure Storage aún no admite HTTPS en dominios personalizados, *debe* especificar `http://` para los identificadores URI de todos los puntos de conexión que apunten a un dominio personalizado.
>

### <a name="create-a-connection-string-with-an-endpoint-suffix"></a>Creación de una cadena de conexión con el sufijo de un punto de conexión

Para crear una cadena de conexión para un servicio de Storage en regiones o instancias con sufijos de punto de conexión diferentes, como Azure China 21Vianet o Azure Government, utilice el siguiente formato de cadena de conexión. Indique si desea conectarse a la cuenta de almacenamiento a través de HTTP (recomendado) o HTTPS, reemplace `myAccountName` por el nombre de la cuenta de almacenamiento, reemplace `myAccountKey` por la clave de acceso a la cuenta y reemplace `mySuffix` por el sufijo del identificador URI:

```
DefaultEndpointsProtocol=[http|https];
AccountName=myAccountName;
AccountKey=myAccountKey;
EndpointSuffix=mySuffix;
```

Este es un ejemplo de cadena de conexión para los servicios de Storage en Azure China 21Vianet:

```
DefaultEndpointsProtocol=https;
AccountName=storagesample;
AccountKey=<account-key>;
EndpointSuffix=core.chinacloudapi.cn;
```

## <a name="parsing-a-connection-string"></a>Análisis de una cadena de conexión

[!INCLUDE [storage-cloud-configuration-manager-include](../../../includes/storage-cloud-configuration-manager-include.md)]

## <a name="next-steps"></a>Pasos siguientes

* [Uso del emulador Azurite para el desarrollo local de Azure Storage](../common/storage-use-azurite.md)
* [Exploradores de Azure Storage](storage-explorers.md)
* [Uso de firmas de acceso compartido (SAS)](storage-sas-overview.md)
