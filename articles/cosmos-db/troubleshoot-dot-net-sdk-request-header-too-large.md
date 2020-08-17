---
title: Solución de problemas de encabezado de solicitud demasiado grande o solicitud incorrecta con código de error 400 en Azure Cosmos DB
description: Diagnóstico y corrección de excepción de encabezado de solicitud demasiado grande
author: j82w
ms.service: cosmos-db
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: 942af52c43a1d4526a42f20df40abf3f510e0ee6
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87293921"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-request-header-too-large-message"></a>Diagnóstico y solución de problemas de mensaje de encabezado de solicitud demasiado grande de Azure Cosmos DB
El mensaje de encabezado de solicitud demasiado grande se muestra con un código de error HTTP 400. Este error se produce cuando el tamaño del encabezado de solicitud es demasiado grande y supera el tamaño máximo permitido. Se recomienda usar la versión más reciente del SDK. Asegúrese de usar como mínimo la versión 3.x o 2.x, que agregan la supervisión del tamaño del encabezado al mensaje de excepción.

## <a name="troubleshooting-steps"></a>Pasos para solucionar problemas
El mensaje de encabezado de solicitud demasiado grande aparece si la sesión o el token de continuación es demasiado grande. En las secciones siguientes se describe la causa y la solución para el problema en cada categoría.

### <a name="1-session-token-too-large"></a>1. El token de sesión es demasiado grande

#### <a name="cause"></a>Causa:
Una solicitud incorrecta con código de error 400 es la causa más probable de que el token de sesión sea grande. Si se cumplen las siguientes condiciones, se confirmará que el token de sesión es demasiado grande.

* El error se produce en una operación de punto como crear, leer, actualizar, etc., donde no hay un token de continuación.
* La excepción se inició sin realizar cambios en la aplicación. El token de sesión crece a medida que aumenta el número de particiones en el contenedor. El número de particiones crece a medida que también lo hace la cantidad de datos o si se aumenta el rendimiento.

#### <a name="temporary-mitigation"></a>Mitigación temporal: 
Reinicie la aplicación cliente para restablecer todos los tokens de sesión. El token de sesión volverá al tamaño anterior que causó el problema. Por lo tanto, use la solución de la sección siguiente para evitar este problema por completo.

#### <a name="solution"></a>Solución:
1. Siga las instrucciones que se indican en el artículo de consejos de rendimiento de [.NET V3](performance-tips-dotnet-sdk-v3-sql.md) o [.NET V2](performance-tips.md) y convierta la aplicación para que use el modo de conexión directa con el protocolo TCP. El modo directo con el protocolo TCP no tiene la restricción de tamaño de encabezado como el protocolo HTTP, por lo que evita este problema. Asegúrese de usar la versión más reciente del SDK, que tiene una corrección para las operaciones de consulta cuando la interoperabilidad del servicio no está disponible.
2. Si el modo de conexión directa con el protocolo TCP no es una opción para su carga de trabajo, mitigue el problema cambiando el [nivel de coherencia del cliente](how-to-manage-consistency.md). El token de sesión solo se usa para la coherencia de la sesión, que es el nivel de coherencia predeterminado para Azure Cosmos DB. Cualquier otro nivel de coherencia no usará el token de sesión.

### <a name="2-continuation-token-too-large"></a>2. Token de continuación demasiado grande

#### <a name="cause"></a>Causa:
La solicitud incorrecta con código de error 400 se produce en operaciones de consulta en las que se usa el token de continuación. Esto ocurre si el token de continuación es demasiado grande o si hay diferentes consultas con tokens de continuación de distintos tamaños.
    
#### <a name="solution"></a>Solución:
1. Siga las instrucciones que se indican en el artículo de consejos de rendimiento de [.NET V3](performance-tips-dotnet-sdk-v3-sql.md) o [.NET V2](performance-tips.md) y convierta la aplicación para que use el modo de conexión directa con el protocolo TCP. El modo directo con el protocolo TCP no tiene la restricción de tamaño de encabezado como el protocolo HTTP, por lo que evita este problema. 
3. Si el modo de conexión directa con el protocolo TCP no es una opción para su carga de trabajo, pruebe a establecer la opción `ResponseContinuationTokenLimitInKb`. Puede encontrar esta opción en `FeedOptions` para V2 o en `QueryRequestOptions` para V3.

## <a name="next-steps"></a>Pasos siguientes
* [Diagnóstico y solución de problemas](troubleshoot-dot-net-sdk.md) al usar el SDK de .NET de Azure Cosmos DB
* Más información sobre las directrices de rendimiento para [.NET V3](performance-tips-dotnet-sdk-v3-sql.md) y [.NET V2](performance-tips.md)
