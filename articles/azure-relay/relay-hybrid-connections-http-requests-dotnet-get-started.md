---
title: Conexiones híbridas de Azure Relay - Solicitudes HTTP en .NET
description: Escriba una aplicación de consola en C# para las solicitudes HTTP de Conexiones híbridas de Azure Relay en .NET.
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 06/23/2020
ms.openlocfilehash: 7a11abb984da3601a4d6aa921224e01f94d0871c
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88922589"
---
# <a name="get-started-with-relay-hybrid-connections-http-requests-in-net"></a>Introducción a las solicitudes HTTP de Conexiones híbridas de Relay en .NET
[!INCLUDE [relay-selector-hybrid-connections](../../includes/relay-selector-hybrid-connections.md)]

En esta guía de inicio rápido, creará aplicaciones de remitente y receptor de .NET que envían y reciben mensajes mediante el protocolo HTTP. Las aplicaciones usan la característica Conexiones híbridas de Azure Relay. Para información acerca de Azure Relay en general, consulte [Azure Relay](relay-what-is-it.md). 

En esta guía de inicio rápido, realizará los siguientes pasos:

1. Creación de un espacio de nombres de Relay mediante Azure Portal.
2. Creación de una conexión híbrida en dicho espacio de nombres mediante Azure Portal.
3. Escritura de una aplicación de consola de servidor (de escucha) para recibir mensajes.
4. Escritura de una aplicación de consola de cliente (remitente) para enviar mensajes.
5. Ejecución de aplicaciones. 

## <a name="prerequisites"></a>Prerequisites

Para completar este tutorial, debe cumplir los siguientes requisitos previos:

* [Visual Studio 2015 o posterior](https://www.visualstudio.com). En los ejemplos de este tutorial se usa Visual Studio 2017.
* Suscripción a Azure. Si no tiene una, [cree una cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="create-a-namespace"></a>Creación de un espacio de nombres
[!INCLUDE [relay-create-namespace-portal](../../includes/relay-create-namespace-portal.md)]

## <a name="create-a-hybrid-connection"></a>Create a hybrid connection
[!INCLUDE [relay-create-hybrid-connection-portal](../../includes/relay-create-hybrid-connection-portal.md)]

## <a name="create-a-server-application-listener"></a>Creación de una aplicación de servidor (agente de escucha)
En Visual Studio, escriba una aplicación de consola en C# para escuchar y recibir mensajes de Relay.

[!INCLUDE [relay-hybrid-connections-http-requests-dotnet-get-started-server](../../includes/relay-hybrid-connections-http-requests-dotnet-get-started-server.md)]

## <a name="create-a-client-application-sender"></a>Creación de una aplicación de cliente (remitente)
En Visual Studio, escriba una aplicación de consola en C# para enviar mensajes a Relay.

[!INCLUDE [relay-hybrid-connections-http-requests-dotnet-get-started-client](../../includes/relay-hybrid-connections-http-requests-dotnet-get-started-client.md)]

## <a name="run-the-applications"></a>Ejecución de las aplicaciones
1. Ejecute la aplicación de servidor. Verá el texto siguiente en la ventana de la consola:

    ```
    Online
    Server listening
    ```
1. Ejecute la aplicación cliente. Verá `hello!` en la ventana cliente. El cliente envió una solicitud HTTP al servidor y este respondió con `hello!`. 
3. Ahora, para cerrar las ventanas de la consola, presione **ENTRAR** en ambas ventanas de la consola. 

Enhorabuena, ha creado una aplicación de Conexiones híbridas completa.

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado aplicaciones de cliente y servidor de .NET que han usado HTTP para enviar y recibir mensajes. La característica Conexiones híbridas de Azure Relay también admite el uso de WebSockets para enviar y recibir mensajes. Para aprender a usar WebSockets con Conexiones híbridas de Azure Relay, consulte la [Guía de inicio rápido de WebSockets](relay-hybrid-connections-dotnet-get-started.md).

En esta guía de inicio rápido ha usado .NET Framework para crear aplicaciones cliente y servidor. Para aprender a escribir aplicaciones cliente y servidor con Node.js, consulte la [Guía de inicio rápido de WebSockets en Node.js](relay-hybrid-connections-node-get-started.md) o la [Guía de inicio rápido de HTTP para Node.js](relay-hybrid-connections-http-requests-dotnet-get-started.md).
