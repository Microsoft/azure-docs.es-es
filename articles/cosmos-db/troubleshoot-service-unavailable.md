---
title: Solución de problemas de excepción de servicio no disponible de Azure Cosmos DB
description: Diagnóstico y corrección de excepción de servicio no disponible de Cosmos DB
author: j82w
ms.service: cosmos-db
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: 12ecec7cf8f406ed53fb5e054fc304bf672cbbb0
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87293852"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-service-unavailable"></a>Diagnóstico y solución de problemas de servicio no disponible de Azure Cosmos DB
El SDK no ha podido conectarse con el servicio de Azure Cosmos DB.

## <a name="troubleshooting-steps"></a>Pasos para solucionar problemas
La lista siguiente contiene las causas conocidas y las soluciones para las excepciones de servicio no disponible.

### <a name="1-the-required-ports-are-not-enabled"></a>1. Los puertos necesarios no están habilitados
Compruebe que todos los [puertos necesarios](performance-tips-dotnet-sdk-v3-sql.md#networking) estén habilitados.

## <a name="if-an-existing-application-or-service-started-getting-503"></a>Si una aplicación o un servicio existente ha recibido un error 503

### <a name="1-there-is-an-outage"></a>1. Hay una interrupción
Compruebe el [estado de Azure](https://status.azure.com/status) para ver si hay algún problema pendiente.

### <a name="2-snat-port-exhaustion"></a>2. Agotamiento de puertos SNAT
Siga la [guía de agotamiento de puertos SNAT](troubleshoot-dot-net-sdk.md#snat).

### <a name="3-the-required-ports-are-being-blocked"></a>3. Los puertos necesarios se están bloqueando
Compruebe que todos los [puertos necesarios](performance-tips-dotnet-sdk-v3-sql.md#networking) estén habilitados.

## <a name="next-steps"></a>Pasos siguientes
* [Diagnóstico y solución de problemas](troubleshoot-dot-net-sdk.md) al usar el SDK de .NET de Azure Cosmos DB
* Más información sobre las directrices de rendimiento para [.NET V3](performance-tips-dotnet-sdk-v3-sql.md) y [.NET V2](performance-tips.md)