---
title: Lenguaje de consulta
titleSuffix: Azure Digital Twins
description: Información sobre los aspectos básicos del lenguaje de consulta de Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 3/26/2020
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: 29e1fa603600e246031f2a86aae3b0876b4910ba
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87562485"
---
# <a name="about-the-query-language-for-azure-digital-twins"></a>Acerca del lenguaje de consulta para Azure Digital Twins

Recuerde que el centro de Azure Digital Twins es el [**grafo de gemelos**](concepts-twins-graph.md), que se crea partir de **gemelos digitales** y **relaciones**. Este grafo se puede consultar para obtener información sobre los gemelos digitales y las relaciones que contiene. Estas consultas se escriben en un lenguaje de consulta personalizado similar a SQL al que se conoce como **lenguaje de consulta de Azure Digital Twins**.

Para enviar una consulta al servicio desde una aplicación cliente, usará la [**API de consulta**](https://docs.microsoft.com/dotnet/api/azure.digitaltwins.core.digitaltwinsclient.query?view=azure-dotnet-preview) de Azure Digital Twins. Esto permite a los desarrolladores escribir consultas y aplicar filtros para buscar conjuntos de gemelos digitales en el grafo de gemelos y otra información sobre el escenario de Azure Digital Twins.

[!INCLUDE [digital-twins-query-operations.md](../../includes/digital-twins-query-operations.md)]

## <a name="next-steps"></a>Pasos siguientes

Aprenda a escribir consultas y ver ejemplos de código de cliente en [*Procedimientos: Consulta del grafo gemelo*](how-to-query-graph.md).
