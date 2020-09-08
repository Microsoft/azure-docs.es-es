---
title: Solución de problemas del mensaje "Encabezado de solicitud demasiado grande" o solicitud incorrecta con código de error 400 en Azure Cosmos DB
description: Obtenga información sobre cómo diagnosticar y corregir la excepción de encabezado de solicitud demasiado grande.
author: j82w
ms.service: cosmos-db
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.custom: devx-track-dotnet
ms.openlocfilehash: d5d66ca05390af5f6fef91ca959f1db3d547d3fb
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89014054"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-request-header-too-large-message"></a>Diagnóstico y solución de problemas del mensaje "Encabezado de solicitud demasiado grande" de Azure Cosmos DB
El mensaje "Encabezado de solicitud demasiado grande" se muestra con un código de error HTTP 400. Este error se produce cuando el tamaño del encabezado de solicitud ha crecido tanto que supera el tamaño máximo permitido. Se recomienda usar la versión más reciente del SDK. Utilice como mínimo la versión 3.x o 2.x, ya que estas versiones agregan un seguimiento del tamaño del encabezado al mensaje de excepción.

## <a name="troubleshooting-steps"></a>Pasos para solucionar problemas
El mensaje "Encabezado de solicitud demasiado grande" aparece si el token de sesión o de continuación es demasiado grande. En las secciones siguientes se describe la causa del problema y su solución en cada categoría.

### <a name="session-token-is-too-large"></a>El token de sesión es demasiado grande

#### <a name="cause"></a>Causa:
Es más probable que se produzca una solicitud incorrecta 400 porque el token de sesión es demasiado grande. Si se cumplen las siguientes condiciones, el token de sesión es demasiado grande:

* El error se produce en operaciones puntuales como crear, leer y actualizar, en las que no hay un token de continuación.
* La excepción se inició sin realizar cambios en la aplicación. El token de sesión crece a medida que aumenta el número de particiones en el contenedor. El número de particiones crece a medida que también lo hace la cantidad de datos o si se aumenta el rendimiento.

#### <a name="temporary-mitigation"></a>Mitigación temporal: 
Reinicie la aplicación cliente para restablecer todos los tokens de sesión. Es posible que el token de sesión vuelva a crecer hasta el tamaño anterior que causó el problema. Para evitar este problema por completo, use la solución de la sección siguiente.

#### <a name="solution"></a>Solución:
1. Siga las instrucciones de los artículos sobre las sugerencias de rendimiento para [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) o [.NET v2](performance-tips.md). Convierta la aplicación para que use el modo de conexión directa con el Protocolo de control de transmisión (TCP). El modo de conexión directa con el protocolo TCP no tiene la restricción de tamaño de encabezado como el protocolo HTTP, por lo que se evita este problema. Asegúrese de usar la versión más reciente del SDK, que tiene una corrección para las operaciones de consulta cuando la interoperabilidad del servicio no está disponible.
1. Si el modo de conexión directa con el protocolo TCP no es una opción para su carga de trabajo, mitigue el problema cambiando el [nivel de coherencia del cliente](how-to-manage-consistency.md). El token de sesión solo se usa para la coherencia de la sesión, que es el nivel de coherencia predeterminado para Azure Cosmos DB. Los otros niveles de coherencia no usan el token de sesión.

### <a name="continuation-token-is-too-large"></a>El token de continuación es demasiado grande

#### <a name="cause"></a>Causa:
El error 400 Solicitud incorrecta se produce en operaciones de consulta en las que se usa el token de continuación si el token de continuación ha crecido demasiado o si hay consultas diferentes con tamaños de tokens de continuación diferentes.
    
#### <a name="solution"></a>Solución:
1. Siga las instrucciones de los artículos sobre las sugerencias de rendimiento para [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) o [.NET v2](performance-tips.md). Convierta la aplicación para que use el modo de conexión directa con el protocolo TCP. El modo de conexión directa con el protocolo TCP no tiene la restricción de tamaño de encabezado como el protocolo HTTP, por lo que se evita este problema. 
1. Si el modo de conexión directa con el protocolo TCP no es una opción para su carga de trabajo, establezca la opción `ResponseContinuationTokenLimitInKb`. Puede encontrar esta opción en `FeedOptions` en v2 o en `QueryRequestOptions` en v3.

## <a name="next-steps"></a>Pasos siguientes
* [Diagnóstico y solución de problemas](troubleshoot-dot-net-sdk.md) al utilizar el SDK de Azure Cosmos DB para .NET.
* Más información sobre las directrices de rendimiento de [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) y [.NET v2](performance-tips.md).
