---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 07/17/2020
ms.author: tamram
ms.openlocfilehash: 85e7cb86217340e77a6f597a357c3de1f91fb8d0
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87070557"
---
Azurite es compatible con una sola cuenta fija y una clave de autenticación ya conocida para autenticación de clave compartida. Esta cuenta y clave son las únicas credenciales de clave compartida que se admiten para su uso con Azurite. Son las siguientes:

```
Account name: devstoreaccount1
Account key: Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==
```

> [!NOTE]
> La clave de autenticación compatible con Azurite está destinada únicamente a la realización de pruebas de funcionalidad del código de autenticación del cliente. No responde a ningún propósito de seguridad. No puede utilizar la cuenta de almacenamiento y la clave de producción con Azurite. Se debe tener en cuenta que no se puede utilizar la cuenta de desarrollo con datos de producción.
> 
> Azurite admite solo la conexión a través de HTTP. Sin embargo, HTTPS es el protocolo recomendado para obtener acceso a recursos en una cuenta de producción de Azure Storage.
> 

#### <a name="connect-to-the-emulator-account-using-a-shortcut"></a>Conexión a la cuenta del emulador mediante un acceso directo
La manera más fácil de conectarse a Azurite desde su aplicación consiste en configurar, dentro del archivo de configuración de la aplicación, una cadena de conexión que haga referencia al acceso directo `UseDevelopmentStorage=true`. Este es un ejemplo de una cadena de conexión con Azurite en un archivo *app.config*: 

```xml
<appSettings>
  <add key="StorageConnectionString" value="UseDevelopmentStorage=true" />
</appSettings>
```

#### <a name="connect-to-the-emulator-account-using-the-well-known-account-name-and-key"></a>Conexión a la cuenta del emulador con el nombre de cuenta y la clave conocidos
Para crear una cadena de conexión que hace referencia al nombre de la cuenta del emulador y la clave, debe especificar los puntos de conexión para cada uno de los servicios que desea usar desde el emulador en la cadena de conexión. Esto es necesario para que la cadena de conexión haga referencia a los extremos del emulador, que son diferentes de los de una cuenta de almacenamiento de producción. Por ejemplo, el valor de la cadena de conexión será similar al siguiente:

```
DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;
AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;
BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;
QueueEndpoint=http://127.0.0.1:10001/devstoreaccount1;
```

Este valor es idéntico al acceso directo mostrado anteriormente, `UseDevelopmentStorage=true`.
