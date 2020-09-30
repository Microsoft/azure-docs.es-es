---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: dademath
manager: nimag
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 07/28/2020
ms.topic: include
ms.custom: include file
ms.author: dademath
ms.openlocfilehash: b4e085a4c81b44f721ddc8385d2dcd0f4a56f42a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945995"
---
Para empezar a usar Azure Communication Services, utilice la biblioteca cliente de SMS de C# de Communication Services para enviar mensajes SMS.

Este inicio rápido supone un pequeño costo en su cuenta de Azure.

<!--**TODO: update all these reference links as the resources go live**

[API reference documentation](../../../references/overview.md) | [Library source code](https://github.com/Azure/azure-sdk-for-net-pr/tree/feature/communication/sdk/communication/Azure.Communication.Sms#todo-update-to-public) | [Package (NuGet)](#todo-nuget) | [Samples](#todo-samples)-->

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- La versión más reciente de la [biblioteca cliente de .NET Core](https://dotnet.microsoft.com/download/dotnet-core) para su sistema operativo.
- Un recurso activo de Communication Services y una cadena de conexión. [Cree un recurso de Communication Services](../../create-communication-resource.md).
- Un número de teléfono habilitado para SMS. [Obtenga un número de teléfono](../get-phone-number.md).

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

- En una ventana de terminal o de comandos, ejecute `dotnet` para comprobar que la biblioteca cliente de .NET esté instalada.
- Para ver los números de teléfono asociados a su recurso de Communication Services, inicie sesión en [Azure Portal](https://portal.azure.com/), busque el recurso de Communication Services y abra la pestaña **números de teléfono** en el panel de navegación izquierdo.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `SmsQuickstart`. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: **Program.cs**.

```console
dotnet new console -o SmsQuickstart
```

Cambie el directorio a la carpeta de la aplicación recién creada y use el comando `dotnet build` para compilar la aplicación.

```console
cd SmsQuickstart
dotnet build
```

### <a name="install-the-package"></a>Instalar el paquete

Mientras sigue en el directorio de aplicaciones, instale el paquete de la biblioteca cliente de SMS de Azure Communication Services para .NET con el comando `dotnet add package`.

```console
dotnet add package Azure.Communication.Sms
```

Agregue la directiva `using` a la parte superior de **Program.cs** para incluir el espacio de nombres `Azure.Communication`.

```csharp

using Azure.Communication.Sms;

```

## <a name="object-model"></a>Modelo de objetos

Las clases e interfaces siguientes controlan algunas de las características principales de la biblioteca cliente de SMS de Azure Communication Services para C#.

| Nombre                                       | Descripción                                                                                                                                                       |
| ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SmsClient     | Esta clase es necesaria para la funcionalidad de los SMS. Cree una instancia de esta con la información de suscripción y úsela para enviar mensajes SMS.                           |
| SendSmsOptions | Esta clase proporciona opciones para configurar los informes de entrega. Si enable_delivery_report se establece en True, se emitirá un evento cuando la entrega se realice correctamente. |

## <a name="authenticate-the-client"></a>Autenticar el cliente

 Abra **Program.cs** en un editor de texto y reemplace el cuerpo del método `Main` por el código para inicializar `SmsClient` con la cadena de conexión. El código siguiente recupera la cadena de conexión para el recurso de una variable de entorno denominada `COMMUNICATION_SERVICES_CONNECTION_STRING`. Obtenga información sobre cómo [administrar la cadena de conexión del recurso](../../create-communication-resource.md#store-your-connection-string).


```csharp
// This code demonstrates how to fetch your connection string
// from an environment variable.
string connectionString = Environment.GetEnvironmentVariable("COMMUNICATION_SERVICES_CONNECTION_STRING");

SmsClient smsClient = new SmsClient(connectionString);
```

## <a name="send-an-sms-message"></a>Enviar un mensaje SMS

Envíe un mensaje SMS con una llamada al método Enviar. Agregue este código al final del método `Main` en **Program.cs**:

```csharp
smsClient.Send(
    from: new PhoneNumber("<leased-phone-number>"),
    to: new PhoneNumber("<to-phone-number>"),
    message: "Hello World via SMS",
    new SendSmsOptions { EnableDeliveryReport = true } // optional
);
```

Debe reemplazar `<leased-phone-number>` por un número de teléfono habilitado para SMS asociado al recurso de Communication Services y `<to-phone-number>` por el número de teléfono al que quiere enviar un mensaje. Todos los parámetros de número de teléfono deben cumplir el [estándar E.164](../../../concepts/telephony-sms/plan-solution.md#optional-reading-international-public-telecommunication-numbering-plan-e164).

El parámetro `EnableDeliveryReport` es un parámetro opcional que puede usar para configurar los informes de entrega. Resulta útil para los escenarios en los que quiere emitir eventos cuando se entregan mensajes SMS. Consulte el inicio rápido [Control de eventos SMS](../handle-sms-events.md) a fin de configurar los informes de entrega para los mensajes SMS.

## <a name="run-the-code"></a>Ejecución del código

Ejecute la aplicación desde el directorio de la aplicación con el comando `dotnet run`.

```console
dotnet run
```
