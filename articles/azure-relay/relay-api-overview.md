---
title: Información general sobre API de Azure Relay | Microsoft Docs
description: En este artículo se proporciona información general sobre las API de Azure Relay disponibles (.NET Standard, .NET Framework, Node.js, etc.).
ms.topic: article
ms.custom: devx-track-dotnet
ms.date: 06/23/2020
ms.openlocfilehash: 98bbb1ecc7e870ff9b7687284e7087d44375d275
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935728"
---
# <a name="available-relay-apis"></a>API de Relay disponibles

## <a name="runtime-apis"></a>API de tiempo de ejecución

La siguiente table enumera todos los clientes del runtime de Relay disponibles.

La sección de [información adicional](#additional-information) contiene más información sobre el estado de cada biblioteca de runtime.

| Lenguaje/plataforma | Característica disponible | Paquete del cliente | Repositorio |
| --- | --- | --- | --- |
| .NET Standard | conexiones híbridas | [Microsoft.Azure.Relay](https://www.nuget.org/packages/Microsoft.Azure.Relay/) | [GitHub](https://github.com/azure/azure-relay-dotnet) |
| .NET Framework | Retransmisión de WCF | [WindowsAzure.ServiceBus](https://www.nuget.org/packages/WindowsAzure.ServiceBus/) | N/D |
| Nodo | conexiones híbridas | [WebSockets: `hyco-ws`](https://www.npmjs.com/package/hyco-ws)<br/>[WebSockets: `hyco-websocket`](https://www.npmjs.com/package/hyco-websocket)<br/>[Solicitudes HTTP: `hyco-https`](https://www.npmjs.com/package/hyco-https) | [GitHub](https://github.com/Azure/azure-relay-node) |

### <a name="additional-information"></a>Información adicional

#### <a name="net"></a>.NET

El ecosistema de .NET tiene varios entornos de ejecución y, por tanto, hay varias bibliotecas de .NET para el servicio de retransmisión. La biblioteca de .NET Standard se puede ejecutar mediante .NET Core o .NET Framework, mientras que la biblioteca de .NET Framework solo puede ejecutarse en un entorno de .NET Framework. Para ampliar la información sobre las versiones de .NET Framework, consulte [Versiones de marcos de trabajo](/dotnet/articles/standard/frameworks).

La biblioteca de .NET Standard solo admite el modelo de programación de WCF y se basa en un protocolo binario propio basado en el Transporte `net.tcp` de WCF. Este protocolo y biblioteca se conservan para la compatibilidad con versiones anteriores de las aplicaciones existentes.

La biblioteca de .NET Standard se basa en la definición de protocolo abierto para el servicio de retransmisión de Conexiones híbridas que se basa en HTTP y WebSockets. La biblioteca admite una abstracción de secuencia sobre WebSockets y un gesto de la API de solicitud-respuesta simple para responder a las solicitudes HTTP. El ejemplo de [API web](https://github.com/Azure/azure-relay-dotnet) muestra cómo integrar Conexiones híbridas con ASP.NET Core para servicios web.

#### <a name="nodejs"></a>Node.js

Los módulos de Conexiones híbridas que aparecen en la tabla anterior reemplazan o modificar los módulos existentes de Node.js con implementaciones alternativas que escuchan en el servicio de Azure Relay en lugar de la pila de red local.

El módulo `hyco-https` modifica e invalida parcialmente los módulos Node.js centrales `http` y `https`, lo que proporciona una implementación de agente de escucha HTTPS compatible con muchas aplicaciones y módulos Node.js existentes que se basan en estos módulos centrales.

Los módulos `hyco-ws` y `hyco-websocket` modifican los populares módulos `ws` y `websocket` para Node.js, lo que proporciona implementaciones de agente de escucha alternativos que permiten que los módulos y las aplicaciones que se basan en cualquiera de los módulos trabajen detrás del servicio de retransmisión de Conexiones híbridas.

Puede encontrar detalles sobre esos módulos en el repositorio GitHub [azure-relay-node](https://github.com/Azure/azure-relay-node).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre Azure Relay, visite estos vínculos:
* [¿Qué es Relay de Azure?](relay-what-is-it.md)
* [Preguntas más frecuentes acerca de Relay](relay-faq.md)
