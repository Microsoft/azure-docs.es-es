---
title: WebSockets de Conexiones híbridas de Azure Relay en Node
description: Escritura de una aplicación de consola en Node.js para Websockets de Conexiones híbridas de Azure Relay
ms.topic: conceptual
ms.date: 06/23/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: 8dc52c93d4b5cd6d50b66338350a41cf0e24daa0
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87386306"
---
# <a name="get-started-with-relay-hybrid-connections-websockets-in-nodejs"></a>Introducción a WebSockets de Conexiones híbridas de Relay en Node.js

[!INCLUDE [relay-selector-hybrid-connections](../../includes/relay-selector-hybrid-connections.md)]

En esta guía de inicio rápido, va a crear aplicaciones de remitente y receptor de Node.js que envían y reciben mensajes mediante WebSockets de Conexiones híbridas en Azure Relay. Para información acerca de Azure Relay en general, consulte [Azure Relay](relay-what-is-it.md). 

En esta guía de inicio rápido, realizará los siguientes pasos: 

1. Creación de un espacio de nombres de Relay mediante Azure Portal.
2. Creación de una conexión híbrida en dicho espacio de nombres mediante Azure Portal.
3. Escritura de una aplicación de consola de servidor (de escucha) para recibir mensajes.
4. Escritura de una aplicación de consola de cliente (remitente) para enviar mensajes.
5. Ejecución de aplicaciones. 

## <a name="prerequisites"></a>Prerequisites

- [Node.js](https://nodejs.org/en/).
- Suscripción a Azure. Si no tiene una, [cree una cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="create-a-namespace"></a>Creación de un espacio de nombres
[!INCLUDE [relay-create-namespace-portal](../../includes/relay-create-namespace-portal.md)]

## <a name="create-a-hybrid-connection"></a>Create a hybrid connection
[!INCLUDE [relay-create-hybrid-connection-portal](../../includes/relay-create-hybrid-connection-portal.md)]

## <a name="create-a-server-application-listener"></a>Creación de una aplicación de servidor (agente de escucha)
Para escuchar y recibir mensajes desde Relay, escriba una aplicación de consola en Node.js.

[!INCLUDE [relay-hybrid-connections-node-get-started-server](../../includes/relay-hybrid-connections-node-get-started-server.md)]

## <a name="create-a-client-application-sender"></a>Creación de una aplicación de cliente (remitente)
Para enviar mensajes a Relay, escriba una aplicación de consola en Node.js.

[!INCLUDE [relay-hybrid-connections-node-get-started-client](../../includes/relay-hybrid-connections-node-get-started-client.md)]

## <a name="run-the-applications"></a>Ejecución de las aplicaciones

1. Ejecute la aplicación de servidor: en un símbolo del sistema de Node.js escriba `node listener.js`.
2. Ejecute la aplicación de cliente: en un símbolo del sistema de Node.js escriba `node sender.js` y escriba texto.
3. Asegúrese de que la consola de aplicación de servidor genera el texto que escribió en la aplicación de cliente.

    ![running-applications](./media/relay-hybrid-connections-node-get-started/running-applications.png)

Ya ha creado una aplicación de Conexiones híbridas de un extremo a otro.

## <a name="next-steps"></a>Pasos siguientes
En esta guía de inicio rápido, ha creado aplicaciones de cliente y servidor de Node.js que han usado WebSockets para enviar y recibir mensajes. La característica Conexiones híbridas de Azure Relay también admite el uso de HTTP para enviar y recibir mensajes. Para aprender a usar HTTP con Conexiones híbridas de Azure Relay, consulte la [Guía de inicio rápido de HTTP para Node.js](relay-hybrid-connections-http-requests-node-get-started.md).

En esta guía de inicio rápido, ha usado Node.js para crear aplicaciones cliente y servidor. Para aprender a escribir aplicaciones cliente y servidor con .NET Framework, consulte la [Guía de inicio rápido de WebSockets para .NET](relay-hybrid-connections-dotnet-get-started.md) o la [Guía de inicio rápido de HTTP para .NET](relay-hybrid-connections-http-requests-dotnet-get-started.md).


