---
title: 'Inicio rápido: Envío y recepción de eventos mediante Go en Azure Event Hubs'
description: 'Inicio rápido: En este artículo se ofrece un tutorial para crear una aplicación de Go que envía eventos de Azure Event Hubs.'
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: 59cd0d757108e7579ce389d216b0ee4d569e12fd
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87002461"
---
# <a name="quickstart-send-events-to-or-receive-events-from-event-hubs-using-go"></a>Inicio rápido: Envío o recepción de eventos en Event Hubs mediante Go
Azure Event Hubs es una plataforma de streaming de macrodatos y servicio de ingesta de eventos de gran escalabilidad capaz de recibir y procesar millones de eventos por segundo. Event Hubs puede procesar y almacenar eventos, datos o telemetría generados por dispositivos y software distribuido. Los datos enviados a un centro de eventos se pueden transformar y almacenar con cualquier proveedor de análisis en tiempo real o adaptadores de procesamiento por lotes y almacenamiento. Para más información sobre Event Hubs, consulte [Introducción a Event Hubs](event-hubs-about.md) y [Características de Event Hubs](event-hubs-features.md).

En este tutorial se describe cómo escribir aplicaciones de Go para enviar o recibir eventos en un centro de eventos. 

> [!NOTE]
> Puede descargar esta guía de inicio rápido como un ejemplo desde [GitHub](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/eventhubs), reemplazar las cadenas `EventHubConnectionString` y `EventHubName` por los valores del centro de eventos, y ejecutarlo. También puede seguir los pasos de este tutorial para crear el suyo propio.

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, debe cumplir los siguientes requisitos previos:

- Go instalado de forma local. Siga [estas instrucciones](https://golang.org/doc/install) si es necesario.
- Una cuenta de Azure activa. Si no tiene una suscripción a Azure, cree una [cuenta gratuita][] antes de empezar.
- **Creación de un espacio de nombres de Event Hubs y un centro de eventos**. Use [Azure Portal](https://portal.azure.com) para crear un espacio de nombres de tipo Event Hubs y obtener las credenciales de administración que la aplicación necesita para comunicarse con el centro de eventos. Para crear un espacio de nombres y un centro de eventos, siga el procedimiento que se indica en [este artículo](event-hubs-create.md).

## <a name="send-events"></a>Envío de eventos
En esta sección se muestra cómo crear una aplicación de Go para enviar eventos a un centro de eventos. 

### <a name="install-go-package"></a>Instalación del paquete Go

Consiga el paquete Go para Event Hubs con `go get` o `dep`. Por ejemplo:

```bash
go get -u github.com/Azure/azure-event-hubs-go
go get -u github.com/Azure/azure-amqp-common-go/...

# or

dep ensure -add github.com/Azure/azure-event-hubs-go
dep ensure -add github.com/Azure/azure-amqp-common-go
```

### <a name="import-packages-in-your-code-file"></a>Importación de paquetes en el archivo de código

Para importar paquetes de Go, utilice el siguiente ejemplo de código:

```go
import (
    aad "github.com/Azure/azure-amqp-common-go/aad"
    eventhubs "github.com/Azure/azure-event-hubs-go"
)
```

### <a name="create-service-principal"></a>Creación de una entidad de servicio

Cree una nueva entidad de servicio siguiendo las instrucciones que se indican en [Creación de una entidad de servicio de Azure con la CLI de Azure 2.0](/cli/azure/create-an-azure-service-principal-azure-cli). Guarde las credenciales proporcionadas en su entorno con los siguientes nombres. Los paquetes de Azure SDK para Go y de Event Hubs se han configurado previamente para buscar estos nombres de variables:

```bash
export AZURE_CLIENT_ID=
export AZURE_CLIENT_SECRET=
export AZURE_TENANT_ID=
export AZURE_SUBSCRIPTION_ID= 
```

Ahora, cree un proveedor de autorización para el cliente de Event Hubs que usa estas credenciales:

```go
tokenProvider, err := aad.NewJWTProvider(aad.JWTProviderWithEnvironmentVars())
if err != nil {
    log.Fatalf("failed to configure AAD JWT provider: %s\n", err)
}
```

### <a name="create-event-hubs-client"></a>Creación de cliente de Event Hubs

El siguiente código crea un cliente de Event Hubs:

```go
hub, err := eventhubs.NewHub("namespaceName", "hubName", tokenProvider)
ctx := context.WithTimeout(context.Background(), 10 * time.Second)
defer hub.Close(ctx)
if err != nil {
    log.Fatalf("failed to get hub %s\n", err)
}
```

### <a name="write-code-to-send-messages"></a>Escritura de código para enviar mensajes

En el siguiente fragmento de código, utilice (1) para enviar mensajes de forma interactiva desde un terminal, o (2) para enviar mensajes desde el programa:

```go
// 1. send messages at the terminal
ctx = context.Background()
reader := bufio.NewReader(os.Stdin)
for {
    fmt.Printf("Input a message to send: ")
    text, _ := reader.ReadString('\n')
    hub.Send(ctx, eventhubs.NewEventFromString(text))
}

// 2. send messages within program
ctx = context.Background()
hub.Send(ctx, eventhubs.NewEventFromString("hello Azure!"))
```

### <a name="extras"></a>Extras

Obtenga los identificadores de las particiones del centro de eventos:

```go
info, err := hub.GetRuntimeInformation(ctx)
if err != nil {
    log.Fatalf("failed to get runtime info: %s\n", err)
}
log.Printf("got partition IDs: %s\n", info.PartitionIDs)
```

Ejecute la aplicación para enviar eventos al centro de eventos. 

Felicidades. Ha enviado mensajes a un centro de eventos.

## <a name="receive-events"></a>Recepción de eventos

### <a name="create-a-storage-account-and-container"></a>Creación de una cuenta de almacenamiento y un contenedor

Los estados como, por ejemplo, las concesiones sobre particiones y puntos de comprobación del flujo de eventos se comparten entre receptores mediante un contenedor de Azure Storage. Puede crear una cuenta de almacenamiento y un contenedor con el SDK para Go, pero también puede crearlos siguiendo las instrucciones de [Acerca de las cuentas de almacenamiento de Azure](../storage/common/storage-account-create.md).

Hay ejemplos disponibles para la creación de artefactos de Storage con el SDK para Go en el [repositorio de ejemplos de Go](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/storage) y en el ejemplo correspondiente de este tutorial.

### <a name="go-packages"></a>Paquetes de Go

Para recibir los mensajes, obtenga los paquetes de Go para Event Hubs con `go get` o `dep`:

```bash
go get -u github.com/Azure/azure-event-hubs-go/...
go get -u github.com/Azure/azure-amqp-common-go/...
go get -u github.com/Azure/go-autorest/...

# or

dep ensure -add github.com/Azure/azure-event-hubs-go
dep ensure -add github.com/Azure/azure-amqp-common-go
dep ensure -add github.com/Azure/go-autorest
```

### <a name="import-packages-in-your-code-file"></a>Importación de paquetes en el archivo de código

Para importar paquetes de Go, utilice el siguiente ejemplo de código:

```go
import (
    aad "github.com/Azure/azure-amqp-common-go/aad"
    eventhubs "github.com/Azure/azure-event-hubs-go"
    eph "github.com/Azure/azure-event-hubs-go/eph"
    storageLeaser "github.com/Azure/azure-event-hubs-go/storage"
    azure "github.com/Azure/go-autorest/autorest/azure"
)
```

### <a name="create-service-principal"></a>Creación de una entidad de servicio

Cree una nueva entidad de servicio siguiendo las instrucciones que se indican en [Creación de una entidad de servicio de Azure con la CLI de Azure 2.0](/cli/azure/create-an-azure-service-principal-azure-cli). Guarde las credenciales proporcionadas en su entorno con los siguientes nombres: Los paquetes de Azure SDK para Go y de Event Hubs se han configurado previamente para buscar estos nombres de variables.

```bash
export AZURE_CLIENT_ID=
export AZURE_CLIENT_SECRET=
export AZURE_TENANT_ID=
export AZURE_SUBSCRIPTION_ID= 
```

A continuación, cree un proveedor de autorización para el cliente de Event Hubs que usa estas credenciales:

```go
tokenProvider, err := aad.NewJWTProvider(aad.JWTProviderWithEnvironmentVars())
if err != nil {
    log.Fatalf("failed to configure AAD JWT provider: %s\n", err)
}
```

### <a name="get-metadata-struct"></a>Obtención de la estructura de metadatos

Obtenga una estructura con metadatos sobre el entorno de Azure mediante Azure SDK para Go. Las operaciones posteriores usarán esta estructura para buscar los puntos de conexión correctos.

```go
azureEnv, err := azure.EnvironmentFromName("AzurePublicCloud")
if err != nil {
    log.Fatalf("could not get azure.Environment struct: %s\n", err)
}
```

### <a name="create-credential-helper"></a>Creación de un asistente de credenciales 

Cree un asistente de credenciales que use las credenciales anteriores de Azure Active Directory (AAD) para crear una credencial de firma de acceso compartido (SAS) para Storage. El último parámetro le indica a este constructor que debe usar las mismas variables de entorno que se usaron anteriormente:

```go
cred, err := storageLeaser.NewAADSASCredential(
    subscriptionID,
    resourceGroupName,
    storageAccountName,
    storageContainerName,
    storageLeaser.AADSASCredentialWithEnvironmentVars())
if err != nil {
    log.Fatalf("could not prepare a storage credential: %s\n", err)
}
```

### <a name="create-a-check-pointer-and-a-leaser"></a>Creación de un generador de puntos de comprobación y un generador de concesiones 

Cree un **generador de concesiones**, que se encargue de conceder una partición a un receptor determinado, y un **generador de puntos de comprobación**, encargado de escribir puntos de comprobación para el flujo de mensajes para que otros receptores puedan empezar a leerlos desde la posición de desplazamiento correcta.

Actualmente, solo hay una instancia de **StorageLeaserCheckpointer** disponible que usa el mismo contenedor de Storage para administrar las concesiones y los puntos de comprobación. Además de la cuenta de almacenamiento y de los nombres de contenedor, **StorageLeaserCheckpointer** necesita las credenciales que se crearon en el paso anterior y la estructura del entorno de Azure para acceder correctamente al contenedor.

```go
leaserCheckpointer, err := storageLeaser.NewStorageLeaserCheckpointer(
    cred,
    storageAccountName,
    storageContainerName,
    azureEnv)
if err != nil {
    log.Fatalf("could not prepare a storage leaserCheckpointer: %s\n", err)
}
```

### <a name="construct-event-processor-host"></a>Construcción del host del procesador de eventos

Ahora tiene los elementos necesarios para construir un host EventProcessorHost, de la forma que se indica a continuación. Se usa la misma instancia de **StorageLeaserCheckpointer** para el generador de concesiones y de puntos de comprobación, como se ha descrito anteriormente:

```go
ctx := context.Background()
p, err := eph.New(
    ctx,
    nsName,
    hubName,
    tokenProvider,
    leaserCheckpointer,
    leaserCheckpointer)
if err != nil {
    log.Fatalf("failed to create EPH: %s\n", err)
}
defer p.Close(context.Background())
```

### <a name="create-handler"></a>Creación de un controlador 

Cree un controlador y regístrelo con el host del procesador de eventos. Cuando se inicie el host, se aplicará este y todos los demás controladores especificados a los mensajes entrantes:

```go
handler := func(ctx context.Context, event *eventhubs.Event) error {
    fmt.Printf("received: %s\n", string(event.Data))
    return nil
}

// register the handler with the EPH
_, err := p.RegisterHandler(ctx, handler)
if err != nil {
    log.Fatalf("failed to register handler: %s\n", err)
}
```

### <a name="write-code-to-receive-messages"></a>Escritura de código para recibir mensajes

Una vez configurado todo, puede iniciar el host del procesador de eventos con `Start(context)` para mantenerlo constantemente en ejecución, o con `StartNonBlocking(context)` para que se ejecute solo si los mensajes están disponibles.

Este tutorial se inicia y se ejecuta de la siguiente manera. Consulte el ejemplo mediante `StartNonBlocking`:

```go
ctx := context.Background()
err = p.Start()
if err != nil {
    log.Fatalf("failed to start EPH: %s\n", err)
}
```

## <a name="next-steps"></a>Pasos siguientes
Lea los siguientes artículos:

- [EventProcessorHost](event-hubs-event-processor-host.md)
- [Características y terminología de Azure Event Hubs](event-hubs-features.md)
- [Preguntas más frecuentes sobre Event Hubs](event-hubs-faq.md)


<!-- Links -->
[Event Hubs overview]: event-hubs-about.md
[cuenta gratuita]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
