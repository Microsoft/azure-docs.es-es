---
title: Reconocimiento óptico de caracteres (OCR) en Computer Vision
titleSuffix: Azure Cognitive Services
description: Conceptos relacionados con el reconocimiento óptico de caracteres (OCR) aplicado a imágenes y documentos que contienen texto impreso y manuscrito mediante la API de Computer Vision.
services: cognitive-services
author: msbbonsu
manager: netahw
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 06/23/2020
ms.author: t-bebon
ms.custom: seodec18
ms.openlocfilehash: 2b3f9b0a4bec76f1f5f9b1f42ec33fdf5e2678bf
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87760165"
---
# <a name="optical-character-recognition-ocr"></a>Reconocimiento óptico de caracteres (OCR)

La API Computer Vision de Microsoft incluye funcionalidades para el reconocimiento óptico de caracteres (OCR) que permiten extraer texto impreso o manuscrito de imágenes y documentos PDF. Las API de OCR extraen texto tanto de los documentos analógicos (imágenes y documentos escaneados) como de los digitalizados. Puede extraer texto de imágenes del mundo real, como fotos de matrículas o de contenedores con números de serie, así como de documentos; por ejemplo, facturas, recibos, informes financieros y artículos, entre otros. La nueva API de OCR de lectura está disponible en el servicio administrado en la nube o de forma local (contenedores). Además, es compatible con redes virtuales y puntos de conexión privados para satisfacer sus necesidades de privacidad y cumplimiento de nivel empresarial.

## <a name="read-api"></a>Read API 

[Read API](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005) de Computer Vision es la tecnología de OCR más reciente de Microsoft que permite extraer texto impreso en varios idiomas, texto manuscrito (solo en inglés), dígitos y símbolos de divisa de imágenes, y documentos PDF con varias páginas. Esta tecnología está optimizada para extraer texto de imágenes del mundo real con mucho texto y de documentos PDF con varias páginas y una mezcla de idiomas. Es compatible con la detección de texto impreso y manuscrito (solo en inglés) en un mismo documento o una misma imagen. Consulte la lista completa de la página [Idiomas admitidos por OCR](https://docs.microsoft.com/azure/cognitive-services/computer-vision/language-support#optical-character-recognition-ocr).

### <a name="how-ocr-works"></a>Funcionamiento de OCR

[Read API](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005) admite documentos con mucho texto, de hasta 2000 páginas; por lo tanto, se ejecuta de forma asincrónica. El primer paso es llamar a la operación de lectura. Dicha operación toma como entrada una imagen o un documento PDF y devuelve un id. de operación. 

El segundo paso es llamar a la operación de [obtención de resultados](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750). Esta operación toma el id. de operación que la operación de lectura ha creado. A continuación, devuelve el contenido del texto extraído de la imagen o del documento en formato JSON. La respuesta en JSON mantiene las agrupaciones de líneas originales de palabras reconocidas. Incluye las líneas de texto extraídas y las coordenadas del cuadro de límite correspondientes. Cada línea de texto incluye todas las palabras extraídas, con las coordenadas y las puntuaciones de confianza respectivas.

Si es necesario, Read corrige el giro de la página reconocida devolviendo el desplazamiento de rotación en grados alrededor del eje horizontal de la imagen, tal como se muestra en la ilustración que hay a continuación.

![Cómo OCR convierte imágenes y documentos en una salida estructurada con texto extraído](./Images/how-ocr-works.svg)

### <a name="sample-ocr-output"></a>Salida de OCR de ejemplo

Se devuelve una respuesta correcta en formato JSON, como se muestra en el siguiente ejemplo:

```json
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
              }
            ]
          }
        ]
      }
    ]
  }
}
```

Complete la guía de inicio rápido sobre [extracción de texto impreso y manuscrito](./QuickStarts/CSharp-hand-text.md) para implementar OCR mediante C# y la API de REST.

### <a name="input-requirements"></a>Requisitos de entrada

Read API permite trabajar con las entradas siguientes:
* Formatos de archivos admitidos: JPEG, PNG, BMP, PDF y TIFF.
* En el caso de los formatos PDF y TIFF, se procesan 2000 páginas como máximo. En el caso de los suscriptores del nivel Gratis, solo se procesan las dos primeras páginas.
* El tamaño de archivo debe ser inferior a 50 MB; y sus dimensiones, de al menos 50x50 píxeles y, como máximo, de 10 000x10 000 píxeles.
* Los archivos PDF deben tener unas dimensiones de 17x17 pulgadas como máximo, lo que se corresponde con los tamaños de papel Legal o A3 y más pequeños.

### <a name="text-from-images"></a>Texto en imágenes

En el siguiente resultado de Read API se muestran las líneas de texto y las palabras extraídas de una imagen que contiene texto con varios ángulos, colores y fuentes.

![Una imagen que se gira y su texto siendo leído y definido](./Images/text-from-images-example.png)

### <a name="text-from-documents"></a>Texto en documentos

Además de con imágenes, Read API también trabaja con documentos PDF como entrada.

![Una imagen que se gira y su texto siendo leído y definido](./Images/text-from-documents-example.png)


### <a name="handwritten-text-in-english"></a>Texto manuscrito en inglés

En estos momentos, la operación de lectura admite la extracción de texto manuscrito solo en inglés.

![Una imagen que se gira y su texto siendo leído y definido](./Images/handwritten-example.png)

### <a name="printed-text-in-supported-languages"></a>Texto impreso en idiomas admitidos

Read API 3.0 admite la extracción de texto impreso en inglés, español, alemán, francés, italiano, portugués y neerlandés. La [versión preliminar pública de la API de Read 3.1](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-preview-1/operations/5d986960601faab4bf452005) agrega compatibilidad con chino simplificado. Si para su escenario se requieren más idiomas, consulte la descripción general sobre la API de OCR que hay en este documento. Consulte la lista de todos los [idiomas admitidos por OCR](https://docs.microsoft.com/azure/cognitive-services/computer-vision/language-support#optical-character-recognition-ocr).

![Una imagen que se gira y su texto siendo leído y definido](./Images/supported-languages-example.png)

### <a name="mixed-languages-support"></a>Compatibilidad con una mezcla de idiomas

Read API admite imágenes y documentos escritos en varios idiomas; normalmente, dichos documentos se conocen como "documentos con una mezcla de idiomas". Para poder procesarlos, clasifica cada línea de texto que hay en un documento según el idioma detectado antes de extraer el contenido de texto.

![Una imagen que se gira y su texto siendo leído y definido](./Images/mixed-language-example.png)

### <a name="data-privacy-and-security"></a>Seguridad y privacidad de datos

Al igual que sucede con todas las instancias de Cognitive Services, los desarrolladores que usan el servicio Read deben estar al tanto de las directivas de Microsoft sobre los datos de los clientes. Para más información, consulte la página de Cognitive Services en [Microsoft Trust Center](https://www.microsoft.com/en-us/trust-center/product-overview).

### <a name="containers-for-on-premise-deployment"></a>Contenedores para la implementación local

Read también está disponible como contenedor de Docker (versión preliminar) para que usted pueda implementar las nuevas funcionalidades de OCR en su entorno. Los contenedores son excelentes para requisitos específicos de control de datos y seguridad. Vea [cómo instalar y ejecutar contenedores de Read](https://docs.microsoft.com/azure/cognitive-services/computer-vision/computer-vision-how-to-install-containers).


## <a name="ocr-api"></a>API de OCR

La [API de OCR](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fc) usa un modelo de reconocimiento anterior, solo admite imágenes y se ejecuta de forma sincrónica, así que devuelve el texto detectado inmediatamente. Consulte los [idiomas admitidos por OCR](https://docs.microsoft.com/azure/cognitive-services/computer-vision/language-support#optical-character-recognition-ocr) y Read API.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre la [API de REST Read 3.0](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005).
- Obtenga información sobre la [API REST de versión preliminar pública de Read 3.1](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-preview-1/operations/5d986960601faab4bf452005) con compatibilidad agregada con chino simplificado.
- Siga la guía de inicio rápido sobre [extracción de texto](./QuickStarts/CSharp-hand-text.md) para implementar OCR mediante C#, Java, JavaScript o Python junto con la API de REST.
