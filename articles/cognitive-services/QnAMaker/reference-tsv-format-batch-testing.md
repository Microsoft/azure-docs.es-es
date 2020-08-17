---
title: 'Pruebas por lotes del formato TSV: QnA Maker'
titleSuffix: Azure Cognitive Services
description: Descripción del formato TSV para las pruebas por lotes
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: reference
ms.date: 10/24/2019
ms.openlocfilehash: 0ed948b4bf69fc672a59a7825279a12868d10521
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132133"
---
# <a name="batch-testing-tsv-format"></a>Pruebas por lotes de formato TSV

Las pruebas por lotes están disponibles desde [código fuente](https://github.com/Azure-Samples/cognitive-services-qnamaker-csharp/tree/master/documentation-samples/batchtesting) o como un [ejecutable descargable comprimido](https://aka.ms/qna_btzip). El formato del comando para ejecutar la prueba por lotes es:

```console
batchtesting.exe input.tsv https://YOUR-HOST.azurewebsites.net ENDPOINT-KEY out.tsv
```

|Parámetro|Valor esperado|
|--|--|
|1|Nombre del archivo TSV formateado con [campos de entrada TSV](#tsv-input-fields).|
|2|URI del punto de conexión, con YOUR-HOST de la página de publicación del portal de QnA Maker.|
|3|ENDPOINT-KEY, que se encuentra en la página de publicación del portal de QnA Maker.|
|4|Nombre del archivo TSV creado por la prueba por lotes para los resultados.|

Utilice la siguiente información para comprender e implementar el formato TSV para las pruebas por lotes. 

## <a name="tsv-input-fields"></a>Campos de entrada TSV

|Campos del archivo de entrada TSV|Notas|
|--|--|
|KBID|Identificador de KB que se encuentra en la página de publicación.|
|Pregunta|Pregunta que un usuario escribiría.|
|Etiquetas de metadatos|opcional|
|Parámetro principal|opcional| 
|Identificador de respuesta esperado|opcional|

![Formato de entrada del archivo TSV para las pruebas por lotes.](media/batch-test/input-tsv-format-batch-test.png)

## <a name="tsv-output-fields"></a>Campos de salida TSV 

|Parámetros del archivo de salida TSV|Notas|
|--|--|
|KBID|Identificador de KB que se encuentra en la página de publicación.|
|Pregunta|Pregunta tal como se especificó en el archivo de entrada.|
|Respuesta|Respuesta principal de Knowledge Base.|
|Identificador de la respuesta.|Identificador de la respuesta.|
|Score|Puntuación de predicción para la respuesta. |
|Etiquetas de metadatos|Asociadas a la respuesta devuelta.|
|Identificador de respuesta esperado|Opcional (solo cuando se proporciona el identificador de respuesta esperado).|
|Etiqueta de resolución|Opcional; los valores pueden ser: correcto o incorrecto (solo cuando se proporciona la respuesta esperada).|
