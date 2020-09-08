---
title: Actualización a la versión 3.0 de Computer Vision API
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo actualizar a la API de lectura Computer Vision v3.0 desde v2.0/v2.1.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: sample
ms.date: 08/11/2020
ms.author: pafarley
ROBOTS: NOINDEX
ms.openlocfilehash: 6e695fcfacac19ca82273d84d049bdb2afe14b54
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88214187"
---
# <a name="upgrade-to-computer-vision-v30-read-api-from-v20v21"></a>Actualización a la API de lectura Computer Vision v3.0 desde v2.0/v2.1

En esta guía se muestra cómo actualizar el código existente de las operaciones de lectura de la API REST Computer Vision v2.0 o v2.1 a v3.0. 

## <a name="upgrade-batch-read-file-to-read"></a>Actualización de `Batch Read File` a `Read`


1. Cambie la ruta de acceso de la API para `Batch Read File` versión 2.x de la siguiente manera:


    |Read 2.x |Read 3.0  |
    |----------|-----------|
    |https://{endpoint}/vision/**v2.0/read/core/asyncBatchAnalyze**     |https://{endpoint}/vision/**v3.0/read/analyze**[?language]|
    
    Está disponible el nuevo parámetro opcional _language_. Si no conoce el idioma del documento o puede que sea multilingüe, no lo incluya. 

2. Cambie la ruta de acceso de la API para `Get Read Results` en la versión 2.x de la siguiente manera:

    |Read 2.x |Read 3.0  |
    |----------|-----------|
    |https://{endpoint}/vision/**v2.0**/read/**operations**/{operationId}     |https://{endpoint}/vision/**v3.0**/read/**analyzeResults**/{operationId}|

3. Cambie el código para comprobar los resultados en formato JSON desde `Get Read Operation Result`. Cuando la llamada a `Get Read Operation Result` se realiza correctamente, devuelve un campo de cadena de estado en el cuerpo JSON. Los siguientes valores de la versión 2.0 se han cambiado para un mejor alineamiento con las otras API REST de Cognitive Services. 
 
    |Read 2.x |Read 3.0  |
    |----------|-----------|
    |`"NotStarted"` |   `"notStarted"`|
    |`"Running"` | `"running"`|
    |`"Failed"` | `"failed"`|
    |`"Succeeded"` | `"succeeded"`|
    
4. Cambie el código para que interprete el código JSON del resultado de reconocimiento final desde `Get Read Operation Result`. 

    Tenga en cuenta los siguientes cambios en el código JSON:
    
    - En la versión 2.x, `"Get Read Operation Result"` devolverá el código JSON de reconocimiento de OCR cuando el estado sea `"Succeeded"`. En la versión 3.0, este campo es `"succeeded"`.
    - Para obtener la raíz de la matriz de páginas, cambie la jerarquía JSON de `"recognitionResults"` a `"analyzeResult"`/`"readResults"`. La jerarquía JSON de palabras y líneas por página permanece sin cambios, por lo que no se requiere ningún cambio en el código.
    -   Se ha cambiado el nombre del elemento `"clockwiseOrientation"` del ángulo de la página a `"angle"` y el intervalo de 0 a 360 grados pasa a ser de -180 a 180 grados. En función del código, es posible que tenga que realizar cambios o no, ya que la mayoría de las funciones matemáticas pueden controlar ambos intervalos.
    -   La versión 3.0 de la API también presenta las siguientes mejoras que puede aprovechar opcionalmente: se agregan `"createdDateTime"` y `"lastUpdatedDateTime"` para que pueda realizar un seguimiento de la duración del procesamiento. Consulte la documentación para más detalles. 
        - El elemento `"version"` indica la versión de la API usada para generar resultados.
        - Se ha agregado el valor de `"confidence"` (confianza) por palabra. Este valor se calibra, de modo que un valor de 0,95 significa que hay un 95 % de probabilidad de que el reconocimiento sea correcto. La puntuación de confianza se puede usar para seleccionar el texto que se va a enviar para revisión humana. 
    
    
        En la versión 2.X, el formato de salida es el siguiente: 
    
    ```json
    {
        {
                "status": "Succeeded",
                "recognitionResults": [
                    {
                    "page": 1,
                    "language": "en",
                    "clockwiseOrientation": 349.59,
                    "width": 2661,
                    "height": 1901,
                    "unit": "pixel",
                    "lines": [
                        {
                        "boundingBox": [
                            67,
                            646,
                            2582,
                            713,
                            2580,
                            876,
                            67,
                            821
                        ],
                        "text": "The quick brown fox jumps",
                        "words": [
                            {
                            "boundingBox": [
                                143,
                                650,
                                435,
                                661,
                                436,
                                823,
                                144,
                                824
                            ],
                            "text": "The",
                            },
            // The rest of result is omitted for brevity 
            
    }
    ```
    
    En la versión 3.0, se ha ajustado:
    
    ```json
    {
        {
            "status": "succeeded",
            "createdDateTime": "2020-05-28T05:13:21Z",
            "lastUpdatedDateTime": "2020-05-28T05:13:22Z",
            "analyzeResult": {
            "version": "3.0.0",
            "readResults": [
                {
                "page": 1,
                "language": "en",
                "angle": 0.8551,
                "width": 2661,
                "height": 1901,
                "unit": "pixel",
                "lines": [
                    {
                    "boundingBox": [
                        67,
                        646,
                        2582,
                        713,
                        2580,
                        876,
                        67,
                        821
                    ],
                    "text": "The quick brown fox jumps",
                    "words": [
                        {
                        "boundingBox": [
                            143,
                            650,
                            435,
                            661,
                            436,
                            823,
                            144,
                            824
                        ],
                        "text": "The",
                        "confidence": 0.958
                        },
        // The rest of result is omitted for brevity 
        
    }
    ```

## <a name="upgrade-from-recognize-text-to-read"></a>Actualización de `Recognize Text` a `Read`
`Recognize Text` es una operación en *versión preliminar* que está *en desuso en todas las versiones de Computer Vision API*. Debe migrar desde `Recognize Text` a `Read` (v3.0) o `Batch Read File` (v2.0, v2.1). La versión 3.0 de `Read` incluye modelos más recientes y mejores para el reconocimiento de texto y características adicionales, por lo que se recomienda. Para actualizar de `Recognize Text` a `Read`:

1. Cambie la ruta de acceso de la API para la versión 2.x de `Recognize Text` de la siguiente manera:


    |Recognize Text 2.x |Read 3.0  |
    |----------|-----------|
    |https://{endpoint}/vision/**v2.0/recognizeText[?mode]**|https://{endpoint}/vision/**v3.0/read/analyze**[?language]|
    
    El parámetro _mode_ no se admite en `Read`. Se admitirá automáticamente el texto escrito a mano e impreso.
    
    En la versión 3.0 está disponible el nuevo parámetro opcional _language_. Si no conoce el idioma del documento o puede que sea multilingüe, no lo incluya. 

2. Cambie la ruta de acceso de la API para la versión 2.x de `Get Recognize Text Operation Result` de la siguiente manera:

    |Recognize Text 2.x |Read 3.0  |
    |----------|-----------|
    |https://{endpoint}/vision/**v2.0/textOperations/** {operationId}|https://{endpoint}/vision/**v3.0/read/analyzeResults**/{operationId}|

3. Cambie el código para comprobar los resultados en formato JSON desde `Get Recognize Text Operation Result`. Cuando la llamada a `Get Read Operation Result` se realiza correctamente, devuelve un campo de cadena de estado en el cuerpo JSON. 
 
    |Recognize Text 2.x |Read 3.0  |
    |----------|-----------|
    |`"NotStarted"` |   `"notStarted"`|
    |`"Running"` | `"running"`|
    |`"Failed"` | `"failed"`|
    |`"Succeeded"` | `"succeeded"`|


4. Cambie el código para que interprete el código JSON del resultado de reconocimiento final desde `Get Recognize Text Operation Result` para admitir `Get Read Operation Result`.

    Tenga en cuenta los siguientes cambios en el código JSON:    

    - En la versión 2.x, `"Get Read Operation Result"` devolverá el código JSON de reconocimiento de OCR cuando el estado sea `"Succeeded"`. En la versión 3.0, este campo es `"succeeded"`.
    - Para obtener la raíz de la matriz de páginas, cambie la jerarquía JSON de `"recognitionResult"` a `"analyzeResult"`/`"readResults"`. La jerarquía JSON de palabras y líneas por página permanece sin cambios, por lo que no se requiere ningún cambio en el código.
    -   La versión 3.0 de la API también presenta las siguientes mejoras que puede aprovechar opcionalmente. Consulte la referencia de la API para más detalles: se han agregado `"createdDateTime"` y `"lastUpdatedDateTime"` para que pueda realizar un seguimiento de la duración del procesamiento. Consulte la documentación para más detalles. 
        - El elemento `"version"` indica la versión de la API usada para generar resultados.
        - Se ha agregado el valor de `"confidence"` (confianza) por palabra. Este valor se calibra, de modo que un valor de 0,95 significa que hay un 95 % de probabilidad de que el reconocimiento sea correcto. La puntuación de confianza se puede usar para seleccionar el texto que se va a enviar para revisión humana. 
        - La orientación general (`"angle"`) del texto va en el sentido de las agujas del reloj, medido en grados entre -180 y 180.
        -  `"width"` y `"height"` proporcionan las dimensiones del documento y `"unit"` proporciona la unidad de esas dimensiones (píxeles o pulgadas, en función del tipo de documento).
        - Con `"page"` se admiten documentos de varias páginas.
        - `"language"` es el idioma de entrada del documento (del parámetro opcional _language_).


    En la versión 2.X, el formato de salida es el siguiente: 
    
    ```json
    {
        {
                "status": "Succeeded",
                "recognitionResult": [
                    {
                    "lines": [
                        {
                        "boundingBox": [
                            67,
                            646,
                            2582,
                            713,
                            2580,
                            876,
                            67,
                            821
                        ],
                        "text": "The quick brown fox jumps",
                        "words": [
                            {
                            "boundingBox": [
                                143,
                                650,
                                435,
                                661,
                                436,
                                823,
                                144,
                                824
                            ],
                            "text": "The",
                            },
            // The rest of result is omitted for brevity 
            
    }
    ```
    
    En la versión 3.0, se ha ajustado:
    
    ```json
    {
        {
            "status": "succeeded",
            "createdDateTime": "2020-05-28T05:13:21Z",
            "lastUpdatedDateTime": "2020-05-28T05:13:22Z",
            "analyzeResult": {
            "version": "3.0.0",
            "readResults": [
                {
                "page": 1,
                "angle": 0.8551,
                "width": 2661,
                "height": 1901,
                "unit": "pixel",
                "lines": [
                    {
                    "boundingBox": [
                        67,
                        646,
                        2582,
                        713,
                        2580,
                        876,
                        67,
                        821
                    ],
                    "text": "The quick brown fox jumps",
                    "words": [
                        {
                        "boundingBox": [
                            143,
                            650,
                            435,
                            661,
                            436,
                            823,
                            144,
                            824
                        ],
                        "text": "The",
                        "confidence": 0.958
                        },
        // The rest of result is omitted for brevity 
        
    }
    ```
    
## <a name="all-other-operations"></a>Todas las demás operaciones

No hay ningún otro cambio importante entre la versión 2.X y la versión 3.0 de Computer Vision API. Simplemente puede modificar la ruta de acceso de la API para reemplazar `v2.0` por `v3.0`.
