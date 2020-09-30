---
title: Diagnóstico y solución de problemas de disponibilidad de los SDK de Azure Cosmos en entornos de varias regiones
description: Obtenga información sobre el comportamiento de la disponibilidad del SDK de Azure Cosmos cuando se trabaja en entornos de varias regiones.
author: ealsur
ms.service: cosmos-db
ms.date: 09/16/2020
ms.author: maquaran
ms.subservice: cosmosdb-sql
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: 0c717aca88095df05fc7927f3c3d6e2d481925d2
ms.sourcegitcommit: 7374b41bb1469f2e3ef119ffaf735f03f5fad484
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90708471"
---
# <a name="diagnose-and-troubleshoot-the-availability-of-azure-cosmos-sdks-in-multiregional-environments"></a>Diagnóstico y solución de problemas de disponibilidad de los SDK de Azure Cosmos en entornos de varias regiones

En este artículo se describe el comportamiento de la versión más reciente de los SDK de Azure Cosmos cuando se tiene un problema de conectividad con una región determinada o cuando se produce una conmutación por error de una región.

Todos los SDK de Azure Cosmos ofrecen la opción de personalizar las preferencias regionales. Las siguientes propiedades se utilizan en diferentes SDK:

* La propiedad [ConnectionPolicy.PreferredLocations](/dotnet/api/microsoft.azure.documents.client.connectionpolicy.preferredlocations) en el SDK de .NET V2.
* Las propiedades [CosmosClientOptions.ApplicationRegion](/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationregion) o [CosmosClientOptions.ApplicationPreferredRegions](/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions) en el SDK de .NET V3.
* El método [CosmosClientBuilder.preferredRegions](/java/api/com.azure.cosmos.cosmosclientbuilder.preferredregions) en el SDK de Java V4.
* El [parámetro CosmosClient.preferred_locations](/python/api/azure-cosmos/azure.cosmos.cosmos_client.cosmosclient) en el SDK de Python.

En el caso de las cuentas de una sola región de escritura, todas las operaciones de escritura irán siempre a la región de escritura, por lo que la lista de regiones preferidas solo se aplicará a las operaciones de lectura. En el caso de las cuentas de varias regiones de escritura, la lista de preferencias afecta a las operaciones de lectura y escritura.

Si no establece una región preferida, el orden de preferencias regionales se define mediante el [orden de la lista de regiones de Azure Cosmos DB](distribute-data-globally.md).

Cuando se produce cualquiera de los siguientes escenarios, el cliente que usa el SDK de Azure Cosmos expone los registros e incluye la información de reintentos como parte de la **información de diagnóstico de la operación**.

## <a name="removing-a-region-from-the-account"></a><a id="remove region">Eliminación de una región de la cuenta</a>

Al quitar una región de una cuenta de Azure Cosmos, cualquier cliente SDK que use activamente la cuenta detectará la eliminación de la región a través de un código de respuesta de back-end. Después, el cliente marca el punto de conexión regional como no disponible. El cliente reintentará la operación actual y todas las operaciones futuras se enrutarán permanentemente a la siguiente región en orden de preferencia.

## <a name="adding-a-region-to-an-account"></a>Adición de una región a una cuenta

Cada 5 minutos, el cliente SDK de Azure Cosmos lee la configuración de la cuenta y actualiza las regiones que conoce.

Cuando se quita una región y más adelante se vuelve a agregar a la cuenta, si la región agregada tiene una preferencia más alta, el SDK volverá a usar esta región de forma permanente. Una vez detectada la región agregada, todas las solicitudes futuras se dirigirán a ella.

Si configura el cliente para que se conecte preferiblemente a una región que no tiene la cuenta de Azure Cosmos, se omitirá la región preferida. Si agrega esa región más adelante, el cliente la detectará y cambiará de forma permanente a esa región.

## <a name="failover-the-write-region-in-a-single-write-region-account"></a><a id="manual-failover-single-region"></a>Conmutación por error de la región de escritura en una cuenta de una sola región de escritura

Si inicia una conmutación por error de la región de escritura actual, la siguiente solicitud de escritura producirá un error con una respuesta de back-end conocida. Cuando se detecta esta respuesta, el cliente consultará la cuenta para obtener información sobre la nueva región de escritura y volverá a intentar la operación actual y enrutará permanentemente todas las operaciones de escritura futuras a la nueva región.

## <a name="regional-outage"></a>Interrupción regional

Si la cuenta es de una sola región de escritura y la interrupción regional se produce durante una operación de escritura, el comportamiento es similar a una [conmutación por error manual](#manual-failover-single-region). En el caso de las solicitudes de lectura o de cuentas de varias regiones de escritura, el comportamiento es similar a [quitar una región](#remove region).

## <a name="session-consistency-guarantees"></a>Garantías de coherencia de la sesión

Al utilizar la [coherencia de la sesión](consistency-levels.md#guarantees-associated-with-consistency-levels), el cliente debe garantizar que puede leer sus propias escrituras. En las cuentas de una sola región de escritura en las que la preferencia de región de lectura es diferente de la región de escritura, podría haber casos en los que el usuario emita una escritura y al realizar una lectura desde una región local, esta no haya recibido todavía la replicación de datos (restricción de velocidad de la luz). En tales casos, el SDK detecta el error específico en la operación de lectura y reintenta la lectura en la región del centro para garantizar la coherencia de la sesión.

## <a name="transient-connectivity-issues-on-tcp-protocol"></a>Problemas de conectividad transitorios en el protocolo TCP

En los escenarios en los que el cliente SDK de Azure Cosmos está configurado para usar el protocolo TCP, pueden darse situaciones para una solicitud determinada en las que las condiciones de red afecten temporalmente a la comunicación con un punto de conexión determinado. Estas condiciones de red temporales se pueden exponer como tiempos de expiración de TCP. El cliente reintentará la solicitud localmente en el mismo punto de conexión durante algunos segundos.

Si el usuario ha configurado una lista de regiones preferidas con más de una región y la cuenta de Azure Cosmos es de varias regiones de escritura o de una sola región de escritura y la operación es una solicitud de lectura, el cliente volverá a intentar esa única operación en la siguiente región de la lista de preferencias.

## <a name="next-steps"></a>Pasos siguientes

* Utilización del [SDK de .NET](sql-api-sdk-dotnet-standard.md) más reciente
* Utilización del [SDK de Java](sql-api-sdk-java-v4.md) más reciente
* Utilización del [SDK de Python](sql-api-sdk-python.md) más reciente
* Utilización del [SDK de Node](sql-api-sdk-node.md) más reciente
