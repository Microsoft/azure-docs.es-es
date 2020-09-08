---
title: 'Cómo convertir formatos de token de sesión en el SDK de .NET: Azure Cosmos DB'
description: Aprenda a convertir formatos de token de sesión para garantizar la compatibilidad entre distintas versiones del SDK de .NET
author: vinhms
ms.service: cosmos-db
ms.topic: how-to
ms.date: 04/30/2020
ms.author: vitrinh
ms.custom: devx-track-csharp
ms.openlocfilehash: 787c39681d0e9aff25d205c7b195be00b8c0bc9c
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020021"
---
# <a name="convert-session-token-formats-in-net-sdk"></a>Conversión de formatos de token de sesión en el SDK de .NET

En este artículo se explica cómo convertir a otros formatos de token de sesión para garantizar la compatibilidad entre distintas versiones del SDK.

> [!NOTE]
> De forma predeterminada, el SDK realiza un seguimiento del token de sesión automáticamente y usará el token de sesión más reciente.  Para más información, visite [Uso de tokens de sesión](how-to-manage-consistency.md#utilize-session-tokens). Las instrucciones de este artículo solo se aplican con las siguientes condiciones:
> * Su cuenta de Azure Cosmos DB usa coherencia de sesión.
> * Administra los tokens de sesión manualmente.
> * Utiliza varias versiones del SDK al mismo tiempo.

## <a name="session-token-formats"></a>Formatos de token de sesión

Existen dos formatos de token de sesión: **simple** y **vector**.  Estos dos formatos no son intercambiables, por lo que debe convertirse el formato al pasar a la aplicación cliente con versiones diferentes.
- El SDK de .NET v1 (Microsoft.Azure.DocumentDB -versión 1.x) usa el formato de token de sesión **simple**.
- El SDK de .NET v2 (Microsoft.Azure.DocumentDB -versión 2.x) usa el formato de token de sesión **vector**.

### <a name="simple-session-token"></a>Token de sesión simple

Un token de sesión simple tiene este formato: `{pkrangeid}:{globalLSN}`

### <a name="vector-session-token"></a>Token de sesión vector

Un token de sesión vector tiene este formato: `{pkrangeid}:{Version}#{GlobalLSN}#{RegionId1}={LocalLsn1}#{RegionId2}={LocalLsn2}....#{RegionIdN}={LocalLsnN}`

## <a name="convert-to-simple-session-token"></a>Conversión a token de sesión simple

Para pasar un token de sesión al cliente mediante el SDK de .NET v1, use un formato de token de sesión **simple**.  Por ejemplo, use el siguiente código de ejemplo para convertirlo.

```csharp
private static readonly char[] SegmentSeparator = (new[] { '#' });
private static readonly char[] PkRangeSeparator = (new[] { ':' });

// sessionTokenToConvert = session token from previous response
string[] items = sessionTokenToConvert.Split(PkRangeSeparator, StringSplitOptions.RemoveEmptyEntries);
string[] sessionTokenSegments = items[1].Split(SessionTokenHelpers.SegmentSeparator, StringSplitOptions.RemoveEmptyEntries);

string sessionTokenInSimpleFormat;

if (sessionTokenSegments.Length == 1)
{
    // returning the same token since it already has the correct format
    sessionTokenInSimpleFormat = sessionTokenToConvert;
}
else
{
    long version = 0;
    long globalLSN = 0;

    if (!long.TryParse(sessionTokenSegments[0], out version)
        || !long.TryParse(sessionTokenSegments[1], out globalLSN))
    {
        throw new ArgumentException("Invalid session token format", sessionTokenToConvert);
    }

    sessionTokenInSimpleFormat = string.Format("{0}:{1}", items[0], globalLSN);
}
```

## <a name="convert-to-vector-session-token"></a>Conversión a token de sesión vector

Para pasar un token de sesión al cliente mediante el SDK de .NET v2, use un formato de token de sesión **vector**.  Por ejemplo, use el siguiente código de ejemplo para convertirlo.

```csharp

private static readonly char[] SegmentSeparator = (new[] { '#' });
private static readonly char[] PkRangeSeparator = (new[] { ':' });

// sessionTokenToConvert = session token from previous response
string[] items = sessionTokenToConvert.Split(PkRangeSeparator, StringSplitOptions.RemoveEmptyEntries);
string[] sessionTokenSegments = items[1].Split(SegmentSeparator, StringSplitOptions.RemoveEmptyEntries);

string sessionTokenInVectorFormat;

if (sessionTokenSegments.Length == 1)
{
    long globalLSN = 0;
    if (long.TryParse(sessionTokenSegments[0], out globalLSN))
    {
        sessionTokenInVectorFormat = string.Format("{0}:-2#{1}", items[0], globalLSN);
    }
    else
    {
        throw new ArgumentException("Invalid session token format", sessionTokenToConvert);
    }
}
else
{
    // returning the same token since it already has the correct format
    sessionTokenInVectorFormat = sessionTokenToConvert;
}
```

## <a name="next-steps"></a>Pasos siguientes

Lea los siguientes artículos:

* [Uso de tokens de sesión para administrar la coherencia en Azure Cosmos DB](how-to-manage-consistency.md#utilize-session-tokens)
* [Selección del nivel de coherencia adecuado en Azure Cosmos DB](consistency-levels-choosing.md)
* [Inconvenientes de la coherencia, disponibilidad y rendimiento en Azure Cosmos DB](consistency-levels-tradeoffs.md)
* [Availability and performance tradeoffs for various consistency levels](consistency-levels-tradeoffs.md) (Compromisos entre rendimiento y disponibilidad en los distintos niveles de coherencia)
