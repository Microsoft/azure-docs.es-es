---
title: Requisitos del esquema de datos
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 09/10/2020
ms.author: aahi
ms.openlocfilehash: a64bb5b28a06d9a013d59e022047f5e2841126ab
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932412"
---
Metrics Monitor es un servicio de detección, diagnóstico y análisis de anomalías de serie temporal. Como servicio basado en IA, utiliza los datos para entrenar el modelo usado. El servicio acepta tablas de datos agregados con las columnas siguientes:

* **Measure** (Medida) (obligatorio): una o varias columnas que contienen valores numéricos.
* **Timestamp** (Marca de tiempo) (opcional): cero o una columna del tipo `DateTime` o `String`. Cuando esta columna no está establecida, la marca de tiempo se establece como la hora de inicio de cada período de ingesta. Dé formato a la marca de tiempo como `yyyy-MM-ddTHH:mm:ssZ`. 
  * **La marca de tiempo debe coincidir con la granularidad de la métrica. Por ejemplo, una métrica diaria debe garantizar la hora, el minuto y el segundo de la marca de tiempo etiquetada como `00:00:00`** .
* **Dimension** (Dimensión) (opcional): las columnas pueden ser de cualquier tipo de datos. Tenga cuidado al trabajar con grandes volúmenes de columnas y valores si desea evitar que se procese un número excesivo de dimensiones.

> [!Note]
> Para cada métrica, solo debería haber una marca de tiempo por medida, que corresponde a una combinación de dimensión. Agregue los datos antes de la incorporación o use la consulta para especificar los que se vayan a ingerir.