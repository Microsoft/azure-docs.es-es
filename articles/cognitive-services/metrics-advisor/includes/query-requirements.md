---
title: Requisitos de consulta
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 09/10/2020
ms.author: aahi
ms.openlocfilehash: 23da214f4ffc9e589523f7ada00a77494183eb15
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932413"
---
Dentro de la consulta, use el parámetro `@StartTime` para obtener datos de métricas para una marca de tiempo determinada. Se reemplazará por una cadena de formato `yyyy-MM-ddTHH:mm:ss`. 

> [!IMPORTANT]
> Asegúrese de que la consulta solo devolverá los datos de métricas de **una marca de tiempo única**. Metrics Advisor ejecutará la consulta en todas las marcas de tiempo para obtener los datos de métricas correspondientes. Por ejemplo, una consulta para una métrica con granularidad *diaria* solo debe contener una marca de tiempo única, como `2020-06-21T00:00:00Z` al ejecutar la consulta una vez. 
