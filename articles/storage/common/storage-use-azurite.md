---
title: Uso del emulador de Azurite para el desarrollo local de Azure Storage
description: El emulador de código abierto Azurite proporciona un entorno local gratuito para probar las aplicaciones de Azure Storage.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 07/15/2020
ms.service: storage
ms.subservice: common
ms.topic: how-to
ms.openlocfilehash: c850fccf5a86df4c35ce4db53b5b40d5e8588210
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87089420"
---
# <a name="use-the-azurite-emulator-for-local-azure-storage-development"></a>Uso del emulador Azurite para el desarrollo local de Azure Storage

El emulador de código abierto Azurite proporciona un entorno local gratuito para probar las aplicaciones de almacenamiento de colas y blobs de Azure. Cuando esté satisfecho con el funcionamiento de la aplicación a nivel local, puede empezar a usarla en una cuenta de almacenamiento de Azure Storage en la nube. El emulador proporciona compatibilidad multiplataforma en Windows, Linux y MacOS.

Azurite es la plataforma del emulador de almacenamiento futura. Azurite reemplaza al [emulador de Azure Storage](storage-use-emulator.md). Azurite se seguirá actualizando para admitir las versiones más recientes de las API de Azure Storage.

Hay varias maneras de instalar y ejecutar Azurite en el sistema local:

  1. [Instalación y ejecución de la extensión de Visual Studio Code para Azurite](#install-and-run-the-azurite-visual-studio-code-extension)
  1. [Instalación y ejecución de Azurite con NPM](#install-and-run-azurite-by-using-npm)
  1. [Instalación y ejecución de la imagen de Docker de Azurite](#install-and-run-the-azurite-docker-image)
  1. [Clonación, compilación y ejecución de Azurite desde el repositorio de GitHub](#clone-build-and-run-azurite-from-the-github-repository)

## <a name="install-and-run-the-azurite-visual-studio-code-extension"></a>Instalación y ejecución de la extensión de Visual Studio Code para Azurite

En Visual Studio Code, seleccione el panel **EXTENSIONES** y busque *Azurite* en **EXTENSIONES: MARKETPLACE**.

![Marketplace de extensiones de Visual Studio Code](media/storage-use-azurite/azurite-vs-code-extension.png)

También puede ir a la opción de [comercialización de la extensión de Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=Azurite.azurite) en el explorador. Seleccione el botón **Instalar** para abrir Visual Studio Code y vaya directamente a la página de la extensión de Azurite.

La extensión admite los siguientes comandos de Visual Studio Code. Presione F1 en Visual Studio Code para abrir la paleta de comandos. 

   - **Azurite: Clean** (Azurite: Limpiar). Restablece todos los datos de persistencia de los servicios de Azurite
   - **Azurite: Limpiar Blob Service**. Limpia Blob Service
   - **Azurite: Limpiar Queue Service**. Limpia Queue Service
   - **Azurite: Close**(Azurite: Cerrar). Cierra todos los servicios de Azurite
   - **Azurite: Cerrar Blob Service**. Cierra Blob Service
   - **Azurite: Cerrar Queue Service**. Cierra Queue Service
   - **Azurite: Start** (Azurite: Iniciar). Inicia todos los servicios de Azurite
   - **Azurite: Iniciar Blob Service**. Inicia Blob Service
   - **Azurite: Iniciar Queue Service**. Inicia Queue Service

Para configurar Azurite en Visual Studio Code, seleccione el panel de extensiones. Seleccione el icono de **Administración** (engranaje) de **Azurite**. Seleccione la **configuración de extensiones**.

![Configuración de la extensión de configuración de Azurite](media/storage-use-azurite/azurite-configure-extension-settings.png)

Se admite los siguientes valores de configuración:

   - **Azurite: Blob Host** (Azurite: host de blob). El punto de conexión de escucha de Blob service. El valor predeterminado es 127.0.0.1.
   - **Azurite: Blob Port** (Azurite: puerto de blob). El puerto de escucha de Blob service. El puerto predeterminado es 10000.
   - **Azurite: Cert**: ruta de acceso a un archivo de certificado PEM o PFX de confianza local para habilitar el modo HTTPS.
   - **Azurite: Debug** (Azurite: depurar). Salida del registro de depuración al canal de Azurite. El valor predeterminado es **false**.
   - **Azurite: Key**: ruta de acceso a un archivo de clave PEM de confianza local, necesario cuando **Azurite: Cert** apunta a un archivo PEM.
   - **Azurite: Location** (Ubicación de Azurite). La ruta de acceso de la ubicación del área de trabajo. El valor predeterminado es la carpeta de trabajo de Visual Studio Code.
   - **Azurite: Loose**: habilita el modo flexible, que omite los encabezados y parámetros no compatibles.
   - **Azurite: OAuth**: nivel de OAuth opcional.
   - **Azurite: Pwd**: contraseña para el archivo PFX. Es necesaria cuando **Azurite: Cert** apunta a un archivo PFX.
   - **Azurite: Host de cola**. El punto de conexión de escucha del Queue service. El valor predeterminado es 127.0.0.1.
   - **Azurite: Puerto de cola**. El puerto de escucha del Queue service. El puerto predeterminado es 10001.
   - **Azurite: Silent** (Azurite: modo silencioso). El modo silencioso deshabilita el registro de acceso. El valor predeterminado es **false**.
   - **Azurite: Skip Api Version Check** (Azurite: Omitir comprobación de la versión de la API): se omite la comprobación de la versión de la API solicitada. El valor predeterminado es **false**.

## <a name="install-and-run-azurite-by-using-npm"></a>Instalación y ejecución de Azurite con NPM

Este método de instalación requiere que tenga instalada [la versión 8.0 de Node.js o una posterior](https://nodejs.org). El Administrador de paquetes de nodos (npm) es la herramienta de administración de paquetes que se incluye con cada instalación de Node.js. Después de instalar Node.js, ejecute el siguiente comando `npm` para instalar Azurite.

```console
npm install -g azurite
```

Después de instalar Azurite, consulte [Ejecución de Azurite desde una línea de comandos](#run-azurite-from-a-command-line).

## <a name="install-and-run-the-azurite-docker-image"></a>Instalación y ejecución de la imagen de Docker de Azurite

Use [DockerHub](https://hub.docker.com/) para extraer la [imagen de Azurite más reciente](https://hub.docker.com/_/microsoft-azure-storage-azurite) con el siguiente comando:

```console
docker pull mcr.microsoft.com/azure-storage/azurite
```

**Ejecución de la imagen de Docker de Azurite**:

El siguiente comando ejecuta la imagen de Docker de Azurite. El parámetro `-p 10000:10000` redirige las solicitudes del puerto 10000 de la máquina host a la instancia de Docker.

```console
docker run -p 10000:10000 -p 10001:10001 \
    mcr.microsoft.com/azure-storage/azurite
```

**Especifique la ubicación del área de trabajo**:

En el ejemplo siguiente, el `-v c:/azurite:/data`parámetro especifica *c:/azurite* como la ubicación de los datos persistentes de Azurite. El directorio *c:/Azurite*debe crearse antes de ejecutar el comando Docker.

```console
docker run -p 10000:10000 -p 10001:10001 \
    -v c:/azurite:/data mcr.microsoft.com/azure-storage/azurite
```

**Ejecutar solo Blob service**

```console
docker run -p 10000:10000 mcr.microsoft.com/azure-storage/azurite \
    azurite-blob --blobHost 0.0.0.0 --blobPort 10000
```

Para obtener más información sobre la configuración de Azurite al inicio, consulte las [opciones de la línea de comandos](#command-line-options).

## <a name="clone-build-and-run-azurite-from-the-github-repository"></a>Clonación, compilación y ejecución de Azurite desde el repositorio de GitHub

Este método de instalación requiere que tenga instalado [Git](https://git-scm.com/). Clone el [repositorio de GitHub](https://github.com/azure/azurite) para el proyecto de Azurite con el siguiente comando de la consola.

```console
git clone https://github.com/Azure/Azurite.git
```

Después de clonar el código fuente, ejecute los siguientes comandos desde la raíz del repositorio clonado para compilar e instalar Azurite.

```console
npm install
npm run build
npm install -g
```

Después de instalar y compilar Azurite, consulte [Ejecución de Azurite desde una línea de comandos](#run-azurite-from-a-command-line).

## <a name="run-azurite-from-a-command-line"></a>Ejecución de Azurite desde una línea de comandos

> [!NOTE]
> No se puede ejecutar Azurite desde la línea de comandos si solo se instaló la extensión Visual Studio Code. En su lugar, use la paleta de comandos de Visual Studio Code. Para más información, consulte [Instalación y ejecución de la extensión de Visual Studio Code para Azurite](#install-and-run-the-azurite-visual-studio-code-extension).

Para empezar inmediatamente con la línea de comandos, cree un directorio denominado *c:\azurite* y, a continuación, inicie Azurite emitiendo el siguiente comando:

```console
azurite --silent --location c:\azurite --debug c:\azurite\debug.log
```

Este comando indica a Azurite que almacene todos los datos en un directorio determinado, *c:\azurite*. Si se omite la opción `--location`, se usará el directorio de trabajo actual.

## <a name="command-line-options"></a>Opciones de línea de comandos

En esta sección se detallan los modificadores de línea de comandos disponibles al iniciar Azurite.

### <a name="help"></a>Ayuda

**Opcional:** obtenga ayuda de la línea de comandos mediante el modificador `-h` o `--help`.

```console
azurite -h
azurite --help
```

### <a name="blob-listening-host"></a>Host de escucha de Blob

**Opcional**: de forma predeterminada, Azurite escuchará a 127.0.0.1 como el servidor local. Use el modificador `--blobHost` para establecer la dirección de acuerdo a sus requisitos.

Acepte solicitudes solo en la máquina local:

```console
azurite --blobHost 127.0.0.1
```

Permita las solicitudes remotas:

```console
azurite --blobHost 0.0.0.0
```

> [!CAUTION]
> Permitir las solicitudes remotas puede hacer que el sistema sea vulnerable a ataques externos.

### <a name="blob-listening-port-configuration"></a>Configuración del puerto de escucha de Blob

**Opcional**: de forma predeterminada, Azurite escuchará a la instancia de Blob service en el puerto 10000. Use el modificador `--blobPort` para especificar el puerto de escucha que necesite.

> [!NOTE]
> Después de utilizar un puerto personalizado, tiene que actualizar la cadena de conexión o la configuración correspondiente en los SDK o las herramientas de Azure Storage.

Personalice el puerto de escucha de Blob service:

```console
azurite --blobPort 8888
```

Permitir al sistema seleccionar automáticamente un puerto disponible:

```console
azurite --blobPort 0
```

El puerto en uso se muestra durante el inicio de Azurite.

### <a name="queue-listening-host"></a>Host de escucha de Queue

**Opcional**: de forma predeterminada, Azurite escuchará a 127.0.0.1 como el servidor local. Use el modificador `--queueHost` para establecer la dirección de acuerdo a sus requisitos.

Acepte solicitudes solo en la máquina local:

```console
azurite --queueHost 127.0.0.1
```

Permita las solicitudes remotas:

```console
azurite --queueHost 0.0.0.0
```

> [!CAUTION]
> Permitir las solicitudes remotas puede hacer que el sistema sea vulnerable a ataques externos.

### <a name="queue-listening-port-configuration"></a>Configuración del puerto de escucha de Queue

**Opcional**: de forma predeterminada, Azurite escuchará a la instancia de Queue service en el puerto 10001. Use el modificador `--queuePort` para especificar el puerto de escucha que necesite.

> [!NOTE]
> Después de utilizar un puerto personalizado, tiene que actualizar la cadena de conexión o la configuración correspondiente en los SDK o las herramientas de Azure Storage.

Personalice el puerto de escucha de Queue service:

```console
azurite --queuePort 8888
```

Permitir al sistema seleccionar automáticamente un puerto disponible:

```console
azurite --queuePort 0
```

El puerto en uso se muestra durante el inicio de Azurite.

### <a name="workspace-path"></a>Ruta de acceso del área de trabajo

**Opcional**: Azurite almacena los datos en el disco local durante la ejecución. Use el modificador `-l` o `--location` para especificar una ruta de acceso como ubicación del área de trabajo. De forma predeterminada, se usará el directorio de trabajo del proceso actual. Tenga en cuenta la "l" minúscula.

```console
azurite -l c:\azurite
azurite --location c:\azurite
```

### <a name="access-log"></a>Registro de acceso

**Opcional**: de forma predeterminada, el registro de acceso se muestra en la ventana de la consola. Deshabilite la visualización del registro de acceso con el modificador `-s` o `--silent`.

```console
azurite -s
azurite --silent
```
### <a name="debug-log"></a>Registro de depuración

**Opcional**: el registro de depuración incluye información detallada sobre cada solicitud y seguimiento de la pila de excepciones. Habilite el registro de depuración proporcionando una ruta de acceso local válida de archivo al modificador `-d` o `--debug`.

```console
azurite -d path/debug.log
azurite --debug path/debug.log
```

### <a name="loose-mode"></a>Modo flexible

**Opcional**: de forma predeterminada, Azurite aplica el modo strict para bloquear parámetros y encabezados de solicitud no compatibles. Deshabilite el modo strict mediante el modificador `-L` o `--loose`. Tenga en cuenta la "L"' mayúscula.

```console
azurite -L
azurite --loose
```
### <a name="version"></a>Versión

**Opcional**: muestra el número de versión de Azurite instalado mediante el modificador `-v` o `--version`.

```console
azurite -v
azurite --version
```

### <a name="certificate-configuration-https"></a>Configuración de certificados (HTTPS)

**Opcional**: de forma predeterminada, Azurite usa el protocolo HTTP. Habilite el modo HTTPS; para ello, proporcione una ruta de acceso a un correo con privacidad mejorada (.pem) o el archivo de certificado de [Personal Information Exchange (.pfx)](https://docs.microsoft.com/windows-hardware/drivers/install/personal-information-exchange---pfx--files) al modificador `--cert`.

Cuando se proporciona `--cert` para un archivo PEM, debe proporcionar un modificador de `--key` correspondiente.

```console
azurite --cert path/server.pem --key path/key.pem
```

Cuando se proporciona `--cert` para un archivo PFX, debe proporcionar un modificador de `--pwd` correspondiente.

```console
azurite --cert path/server.pfx --pwd pfxpassword
```

Para obtener información detallada sobre la creación de archivos PEM y PFX, consulte la [configuración de HTTPS](https://github.com/Azure/Azurite/blob/master/README.md#https-setup).

### <a name="oauth-configuration"></a>Configuración de OAuth

**Opcional**: habilite la autenticación de OAuth para Azurite mediante el modificador `--oauth`.

```console
azurite --oauth basic --cert path/server.pem --key path/key.pem
```

> [!NOTE]
> OAuth necesita un punto de conexión HTTPS. Asegúrese de que HTTPS está habilitado proporcionando para ello el modificador `--cert` junto con el modificador `--oauth`.

Azurite admite la autenticación básica mediante la especificación del parámetro `basic` en el modificador `--oauth`. Azurite realizará la autenticación básica, como la validación del token de portador entrante, la comprobación del emisor, la audiencia y la expiración. Asimismo, Azurite no comprobará la firma o los permisos del token.

### <a name="skip-api-version-check"></a>Omisión de la comprobación de la versión de la API

**Opcional**: al inicio, Azurite comprueba que la versión de la API solicitada sea válida. El siguiente comando omite la comprobación de la versión de la API:

```console
azurite --skipApiVersionCheck
```


## <a name="authorization-for-tools-and-sdks"></a>Autorización para herramientas y SDK

Conéctese a Azurite desde los SDK o las herramientas de Azure Storage, como el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/), mediante cualquier estrategia de autenticación. Es necesaria la autenticación. Azurite admite la autorización con OAuth, claves compartidas y firmas de acceso compartido (SAS). Azurite también admite el acceso anónimo a contenedores públicos.

Si usa los SDK de Azure, inicie Azurite con las opciones de `--oauth basic and --cert --key/--pwd`.

### <a name="well-known-storage-account-and-key"></a>Cuenta de almacenamiento y clave conocidas

Azurite acepta la misma cuenta y clave conocidas que usa el emulador de Azure Storage heredado.

- Nombre de cuenta: `devstoreaccount1`
- Clave de cuenta: `Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==`

### <a name="custom-storage-accounts-and-keys"></a>Cuentas y claves de almacenamiento personalizadas

Azurite admite las claves y los nombres de cuentas de almacenamiento personalizados estableciendo la variable de entorno `AZURITE_ACCOUNTS` en el siguiente formato: `account1:key1[:key2];account2:key1[:key2];...`.

Por ejemplo, use una cuenta de almacenamiento personalizada que tenga una clave:

```cmd
set AZURITE_ACCOUNTS="account1:key1"
```

```bash
export AZURITE_ACCOUNTS="account1:key1"
```

O bien, use varias cuentas de almacenamiento con dos claves cada una:

```cmd
set AZURITE_ACCOUNTS="account1:key1:key2;account2:key1:key2"
```

```bash
export AZURITE_ACCOUNTS="account1:key1:key2;account2:key1:key2"
```

Azurite actualiza las claves y los nombres de cuentas personalizados de la variable de entorno cada minuto de forma predeterminada. Con esta característica, puede girar de forma dinámica la clave de cuenta, o bien agregar nuevas cuentas de almacenamiento sin reiniciar Azurite.

> [!NOTE]
> La cuenta de almacenamiento `devstoreaccount1` predeterminada se deshabilita cuando establece cuentas de almacenamiento personalizadas.

### <a name="connection-strings"></a>Cadenas de conexión

La manera más fácil de conectarse a Azurite desde su aplicación consiste en configurar, dentro del archivo de configuración de la aplicación, una cadena de conexión que haga referencia al acceso directo *UseDevelopmentStorage=true*. Este es un ejemplo de una cadena de conexión un archivo *app.config*:

```xml
<appSettings>
  <add key="StorageConnectionString" value="UseDevelopmentStorage=true" />
</appSettings>
```

#### <a name="http-connection-strings"></a>Cadenas de conexión de HTTP

Puede pasar las siguientes cadenas de conexión a los [SDK de Azure](https://aka.ms/azsdk) o a herramientas como la CLI de Azure 2.0 o el Explorador de Storage.

La cadena de conexión completa es:

`DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;QueueEndpoint=http://127.0.0.1:10001/devstoreaccount1;`

Para conectarse solo a Blob service, la cadena de conexión es:

`DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;`

Para conectarse solo a Queue service, la cadena de conexión es:

`DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;QueueEndpoint=http://127.0.0.1:10001/devstoreaccount1;`

#### <a name="https-connection-strings"></a>Cadenas de conexión de HTTP

La cadena de conexión completa de HTTPS es:

`DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=https://127.0.0.1:10000/devstoreaccount1;QueueEndpoint=https://127.0.0.1:10001/devstoreaccount1;`

Para usar solo Blob service, la cadena de conexión HTTPS es:

`DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=https://127.0.0.1:10000/devstoreaccount1;`

Para usar solo Queue service, la cadena de conexión HTTPS es:

`DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;QueueEndpoint=https://127.0.0.1:10001/devstoreaccount1;`

Si usó `dotnet dev-certs` para generar el certificado autofirmado, utilice la siguiente cadena de conexión.

`DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=https://localhost:10000/devstoreaccount1;QueueEndpoint=https://localhost:10001/devstoreaccount1;`

Actualice la cadena de conexión al usar [claves y cuentas de almacenamiento personalizadas](#custom-storage-accounts-and-keys).

Para obtener más información sobre las cadenas de conexión, consulte [Configuración de las cadenas de conexión de Azure Storage](storage-configure-connection-string.md).

### <a name="azure-sdks"></a>SDK de Azure

Para usar Azurite con los [SDK de Azure](https://aka.ms/azsdk), use las opciones de OAuth y HTTPS:

```console
azurite --oauth basic --cert certname.pem --key certname-key.pem
```

#### <a name="azure-blob-storage"></a>Azure Blob Storage

A continuación, puede crear una instancia de BlobContainerClient, BlobServiceClient o BlobClient.

```csharp
// With container URL and DefaultAzureCredential
var client = new BlobContainerClient(
    new Uri("https://127.0.0.1:10000/devstoreaccount1/container-name"), new DefaultAzureCredential()
  );

// With connection string
var client = new BlobContainerClient(
    "DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=https://127.0.0.1:10000/devstoreaccount1;QueueEndpoint=https://127.0.0.1:10001/devstoreaccount1;", "container-name"
  );

// With account name and key
var client = new BlobContainerClient(
    new Uri("https://127.0.0.1:10000/devstoreaccount1/container-name"),
    new StorageSharedKeyCredential("devstoreaccount1", "Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==")
  );
```

#### <a name="azure-queue-storage"></a>Azure Queue Storage

También puede crear una instancia de QueueClient o QueueServiceClient.

```csharp
// With queue URL and DefaultAzureCredential
var client = new QueueClient(
    new Uri("https://127.0.0.1:10001/devstoreaccount1/queue-name"), new DefaultAzureCredential()
  );

// With connection string
var client = new QueueClient(
    "DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=https://127.0.0.1:10000/devstoreaccount1;QueueEndpoint=https://127.0.0.1:10001/devstoreaccount1;", "queue-name"
  );

// With account name and key
var client = new QueueClient(
    new Uri("https://127.0.0.1:10001/devstoreaccount1/queue-name"),
    new StorageSharedKeyCredential("devstoreaccount1", "Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==")
  );
```

### <a name="microsoft-azure-storage-explorer"></a>Explorador de Microsoft Azure Storage

Puede usar el Explorador de Storage para ver los datos almacenados en Azurite.

#### <a name="connect-to-azurite-using-http"></a>Conexión a Azurite mediante HTTP

En el Explorador de Storage, conéctese a Azurite siguiendo estos pasos:

 1. Seleccione el icono para **administrar cuentas**.
 1. Haga clic en la opción para **agregar una cuenta**.
 1. Seleccione la opción **Asociar a un emulador local**.
 1. Seleccione **Siguiente**.
 1. Edite el campo del **nombre para mostrar** con un nombre de su elección.
 1. Seleccione **Siguiente** de nuevo.
 1. Seleccione **Conectar**.

#### <a name="connect-to-azurite-using-https"></a>Conexión a Azurite mediante HTTPS

De forma predeterminada, el Explorador de Storage no abrirá un punto de conexión HTTPS que use un certificado autofirmado. Si está ejecutando Azurite con HTTPS, es probable que ya use un certificado autofirmado. En el Explorador de Storage, importe los certificados SSL a través del cuadro de diálogo **Editar** -> **Certificados SSL** -> **Importar certificados**.

##### <a name="import-certificate-to-storage-explorer"></a>Importación del certificado en el Explorador de Storage

1. Busque el certificado en el equipo local.
1. En el Explorador de Storage, vaya a **Editar** -> **Certificados SSL** -> **Importar certificados** e importe el certificado.

Si no importa un certificado, se producirá un error:

`unable to verify the first certificate` o `self signed certificate in chain`

##### <a name="add-azurite-via-https-connection-string"></a>Inclusión de Azurite a través de la cadena de conexión HTTPS

Siga estos pasos para agregar Azurite HTTPS al Explorador de Storage:

1. Seleccione **Alternar explorador**.
1. Seleccione **Local y conectado**.
1. Haga clic con el botón derecho en las **cuentas de almacenamiento** y seleccione la opción para **conectarse a Azure Storage**.
1. Seleccione **Usar una cadena de conexión**.
1. Seleccione **Next** (Siguiente).
1. En el campo del **nombre para mostrar**, especifique un valor.
1. Escriba la [cadena de conexión HTTPS](#https-connection-strings) de la sección anterior de este documento
1. Seleccione **Siguiente**.
1. Seleccione **Conectar**.

## <a name="workspace-structure"></a>Estructura del área de trabajo

Los siguientes archivos y carpetas se pueden crear en la ubicación del área de trabajo al inicializar Azurite.

- `__blobstorage__` - Directorio que contiene datos binarios persistentes de Blob service de Azurite
- `__queuestorage__` - Directorio que contiene datos binarios persistentes de Queue service de Azurite
- `__azurite_db_blob__.json` - Archivo de metadatos de Blob service de Azurite
- `__azurite_db_blob_extent__.json` - Archivo de metadatos de la extensión de Blob service de Azurite
- `__azurite_db_queue__.json` - Archivo de metadatos de Queue service de Azurite
- `__azurite_db_queue_extent__.json` - Archivo de metadatos de la extensión de Queue service de Azurite

Para limpiar Azurite, elimine los archivos y las carpetas anteriores y reinicie el emulador.

## <a name="differences-between-azurite-and-azure-storage"></a>Diferencias entre Azurite y Azure Storage

Existen diferencias funcionales entre una instancia local de Azurite y una cuenta de Azure Storage en la nube.

### <a name="endpoint-and-connection-url"></a>Punto de conexión y URL de conexión

Los puntos de conexión de servicio para Azurite son diferentes de los de una cuenta de Azure Storage. El equipo local no realiza la resolución del nombre de dominio, lo que requiere que los puntos de conexión de Azurite sean direcciones locales.

Al direccionar un recurso en una cuenta de Azure Storage, el nombre de la cuenta forma parte del nombre de host del URI. El recurso que se va a direccionar es parte de la ruta de acceso del URI:

`<http|https>://<account-name>.<service-name>.core.windows.net/<resource-path>`

El siguiente URI es una dirección válida para un blob en una cuenta de Azure Storage:

`https://myaccount.blob.core.windows.net/mycontainer/myblob.txt`

Dado que el equipo local no realiza la resolución de nombres de dominio, el nombre de la cuenta forma parte de la ruta de acceso del URI en lugar del nombre de host. Use el siguiente formato de URI para un recurso en Azurite:

`http://<local-machine-address>:<port>/<account-name>/<resource-path>`

La siguiente dirección se puede usar para obtener acceso a un blob en Azurite:

`http://127.0.0.1:10000/myaccount/mycontainer/myblob.txt`

### <a name="scaling-and-performance"></a>Escalado y rendimiento

Azurite no admite un gran número de clientes conectados. No hay garantía de rendimiento. Azurite se ha pensado para su uso con fines de desarrollo y pruebas.

### <a name="error-handling"></a>Control de errores

Azurite se alinea con la lógica de control de errores de Azure Storage, pero existen diferencias. Por ejemplo, los mensajes de error pueden ser diferentes, mientras que los códigos de estado de error están alineados.

### <a name="ra-grs"></a>RA-GRS

Azurite admite la replicación con redundancia geográfica con acceso de lectura (RA-GRS). Si quiere ver los recursos de almacenamiento, puede obtener acceso a la ubicación secundaria si anexa `-secondary` al nombre de la cuenta. Por ejemplo, la siguiente dirección se puede usar para obtener acceso a un blob usando la cuenta secundaria de solo lectura en Azurite:

`http://127.0.0.1:10000/devstoreaccount1-secondary/mycontainer/myblob.txt`

## <a name="azurite-is-open-source"></a>Azurite es de código abierto

Las contribuciones y las sugerencias para Azurite son bienvenidas. Vaya a la página del [proyecto GitHub](https://github.com/Azure/Azurite/projects) o de [incidencias de Github](https://github.com/Azure/Azurite/issues) para ver los hitos y los elementos de trabajo de los que estamos haciendo un seguimiento para futuras características y correcciones de errores. También se realiza un seguimiento de los elementos de trabajo detallados en GitHub.

## <a name="next-steps"></a>Pasos siguientes

- [Uso del emulador de Azure Storage para el desarrollo y pruebas](storage-use-emulator.md) documenta el antiguo emulador de almacenamiento de Azure que Azurite sustituye.
- En [Configuración de las cadenas de conexión de Azure Storage](storage-configure-connection-string.md) se explica cómo ensamblar una cadena de conexión de Azure Storage válida.
