---
title: Aptitud cognitiva de análisis de imágenes
titleSuffix: Azure Cognitive Search
description: Extraiga texto semántico a través del análisis de imágenes mediante la aptitud cognitiva de análisis de imágenes en una canalización de enriquecimiento con inteligencia artificial de Búsqueda cognitiva de Azure.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/17/2020
ms.openlocfilehash: 63a1f8e30be2983c0df93ff5a7229460f8f39214
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936051"
---
# <a name="image-analysis-cognitive-skill"></a>Aptitud cognitiva de análisis de imágenes

La aptitud de **Análisis de imágenes** extrae un amplio conjunto de características visuales en función del contenido de la imagen. Por ejemplo, puede generar una leyenda a partir de una imagen, generar etiquetas o identificar celebridades y lugares de referencia. Esta aptitud utiliza los modelos de aprendizaje automático proporcionados por [Computer Vision](../cognitive-services/computer-vision/home.md) en Cognitive Services. 

> [!NOTE]
> Los volúmenes pequeños (menos de 20 transacciones) se pueden ejecutar gratis en Búsqueda cognitiva de Azure, pero las cargas de trabajo más grandes requieren la [asociación de un recurso de Cognitive Services facturable](cognitive-search-attach-cognitive-services.md). Los cargos se acumulan cuando se llama a las API de Cognitive Services y por la extracción de imágenes como parte de la fase de descifrado de documentos de Azure Cognitive Search. No hay ningún cargo por la extracción de texto de documentos.
>
> La ejecución de aptitudes integradas se cobra según los [precios de pago por uso de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/) existentes. Los precios de la extracción de imágenes se describen en la [página de precios de Búsqueda cognitiva de Azure](https://azure.microsoft.com/pricing/details/search/).


## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Vision.ImageAnalysisSkill 

## <a name="skill-parameters"></a>Parámetros de la aptitud

Los parámetros distinguen mayúsculas de minúsculas.

| Nombre de parámetro     | Descripción |
|--------------------|-------------|
| `defaultLanguageCode` |  Cadena que indica el idioma que se devolverá. Este servicio devuelve los resultados de reconocimiento en un idioma concreto. Si esta no se especifica este parámetro, se usa el valor predeterminado "en". <br/><br/>Estos son los idiomas admitidos: <br/>*en*: inglés (predeterminado) <br/> *es*: español <br/> *ja*: japonés <br/> *pt*: portugués <br/> *zh*: chino simplificado|
| `visualFeatures` |    Matriz de cadenas que indica los tipos de características visual que se devolverán. Los tipos de características visuales válidos incluyen:  <ul><li>*adult*: detecta si la imagen es de naturaleza pornográfica (representa desnudez o un acto sexual) o cruenta (muestra sangre o violencia extrema). También se detecta contenido provocativo (esto es, contenido subido de tono).</li><li>*brands*: detecta varias marcas en una imagen, incluida la ubicación aproximada. La característica visual *brands* solo está disponible en inglés.</li><li> *categories*: clasifica el contenido de la imagen según una taxonomía definida en la [documentación de Computer Vision](../cognitive-services/computer-vision/category-taxonomy.md) de Cognitive Services. </li><li>*description*: describe el contenido de la imagen con una oración completa en los idiomas que se admiten.</li><li>*faces*: detecta si hay caras presentes. Si las hay, indica las coordenadas, el género y la edad.</li><li>  *objects*: detecta varios objetos en una imagen, incluida la ubicación aproximada. La característica visual *objects* solo está disponible en inglés.</li><li> *Etiquetas*: etiquetas de la imagen con una lista detallada de palabras relacionadas con el contenido de la imagen.</li></ul> Los nombres de las características visuales distinguen entre mayúsculas y minúsculas. Tenga en cuenta que las características visuales *color* y *imageType* han quedado en desuso, pero todavía se puede acceder a esta funcionalidad mediante una [aptitud personalizada](./cognitive-search-custom-skill-interface.md).|
| `details` | Matriz de cadenas que indica qué detalles específicos del dominio que se devolverán. Los tipos de características visuales válidos incluyen: <ul><li>*celebrities*: identifica celebridades si se detectan en la imagen.</li><li>*landmarks*: identifica puntos de referencia si se detectan en la imagen. </li></ul> |

## <a name="skill-inputs"></a>Entradas de la aptitud

| Nombre de entrada      | Descripción                                          |
|---------------|------------------------------------------------------|
| `image`         | Tipo complejo. Actualmente, solo funciona con el campo "/document/normalized_images", que crea el indexador de Azure Blob cuando ```imageAction``` está establecido en un valor diferente a ```none```. Para obtener más información, consulte este [ejemplo](#sample-output).|



##  <a name="sample-skill-definition"></a>Ejemplo de definición de la aptitud

```json
        {
            "description": "Extract image analysis.",
            "@odata.type": "#Microsoft.Skills.Vision.ImageAnalysisSkill",
            "context": "/document/normalized_images/*",
            "defaultLanguageCode": "en",
            "visualFeatures": [
                "tags",
                "categories",
                "description",
                "faces",
                "brands"
            ],
            "inputs": [
                {
                    "name": "image",
                    "source": "/document/normalized_images/*"
                }
            ],
            "outputs": [
                {
                    "name": "categories"
                },
                {
                    "name": "tags"
                },
                {
                    "name": "description"
                },
                {
                    "name": "faces"
                },
                {
                    "name": "brands"
                }
            ]
        }
```
### <a name="sample-index-for-only-the-categories-description-faces-and-tags-fields"></a>Índice de ejemplo (solo para los campos de categorías, descripción, etiquetas y caras)
```json
{
    "fields": [
        {
            "name": "id",
            "type": "Edm.String",
            "key": true,
            "searchable": true,
            "filterable": false,
            "facetable": false,
            "sortable": true
        },
        {
            "name": "blob_uri",
            "type": "Edm.String",
            "searchable": true,
            "filterable": false,
            "facetable": false,
            "sortable": true
        },
        {
            "name": "content",
            "type": "Edm.String",
            "sortable": false,
            "searchable": true,
            "filterable": false,
            "facetable": false
        },
        {
            "name": "categories",
            "type": "Collection(Edm.ComplexType)",
            "fields": [
                {
                    "name": "name",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": false,
                    "facetable": false
                },
                {
                    "name": "score",
                    "type": "Edm.Double",
                    "searchable": false,
                    "filterable": false,
                    "facetable": false
                },
                {
                    "name": "detail",
                    "type": "Edm.ComplexType",
                    "fields": [
                        {
                            "name": "celebrities",
                            "type": "Collection(Edm.ComplexType)",
                            "fields": [
                                {
                                    "name": "name",
                                    "type": "Edm.String",
                                    "searchable": true,
                                    "filterable": false,
                                    "facetable": false
                                },
                                {
                                    "name": "faceBoundingBox",
                                    "type": "Collection(Edm.ComplexType)",
                                    "fields": [
                                        {
                                            "name": "x",
                                            "type": "Edm.Int32",
                                            "searchable": false,
                                            "filterable": false,
                                            "facetable": false
                                        },
                                        {
                                            "name": "y",
                                            "type": "Edm.Int32",
                                            "searchable": false,
                                            "filterable": false,
                                            "facetable": false
                                        }
                                    ]
                                },
                                {
                                    "name": "confidence",
                                    "type": "Edm.Double",
                                    "searchable": false,
                                    "filterable": false,
                                    "facetable": false
                                }
                            ]
                        },
                        {
                            "name": "landmarks",
                            "type": "Collection(Edm.ComplexType)",
                            "fields": [
                                {
                                    "name": "name",
                                    "type": "Edm.String",
                                    "searchable": true,
                                    "filterable": false,
                                    "facetable": false
                                },
                                {
                                    "name": "confidence",
                                    "type": "Edm.Double",
                                    "searchable": false,
                                    "filterable": false,
                                    "facetable": false
                                }
                            ]
                        }
                    ]
                }
            ]
        },
        {
            "name": "description",
            "type": "Collection(Edm.ComplexType)",
            "fields": [
                {
                    "name": "tags",
                    "type": "Collection(Edm.String)",
                    "searchable": true,
                    "filterable": false,
                    "facetable": false
                },
                {
                    "name": "captions",
                    "type": "Collection(Edm.ComplexType)",
                    "fields": [
                        {
                            "name": "text",
                            "type": "Edm.String",
                            "searchable": true,
                            "filterable": false,
                            "facetable": false
                        },
                        {
                            "name": "confidence",
                            "type": "Edm.Double",
                            "searchable": false,
                            "filterable": false,
                            "facetable": false
                        }
                    ]
                }
            ]
        },
        {
            "name": "faces",
            "type": "Collection(Edm.ComplexType)",
            "fields": [
                {
                    "name": "age",
                    "type": "Edm.Int32",
                    "searchable": false,
                    "filterable": false,
                    "facetable": false
                },
                {
                    "name": "gender",
                    "type": "Edm.String",
                    "searchable": false,
                    "filterable": false,
                    "facetable": false
                },
                {
                    "name": "faceBoundingBox",
                    "type": "Collection(Edm.ComplexType)",
                    "fields": [
                        {
                            "name": "x",
                            "type": "Edm.Int32",
                            "searchable": false,
                            "filterable": false,
                            "facetable": false
                        },
                        {
                            "name": "y",
                            "type": "Edm.Int32",
                            "searchable": false,
                            "filterable": false,
                            "facetable": false
                        }
                    ]
                }
            ]
        },
        {
            "name": "tags",
            "type": "Collection(Edm.ComplexType)",
            "fields": [
                {
                    "name": "name",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": false,
                    "facetable": false
                },
                {
                    "name": "confidence",
                    "type": "Edm.Double",
                    "searchable": false,
                    "filterable": false,
                    "facetable": false
                }
            ]
        }
    ]
}

```
### <a name="sample-output-field-mapping-for-the-above-index"></a>Asignación de campos de salida de ejemplo (para el índice anterior)
```json
    "outputFieldMappings": [
        {
            "sourceFieldName": "/document/normalized_images/*/categories/*",
            "targetFieldName": "categories"
        },
        {
            "sourceFieldName": "/document/normalized_images/*/tags/*",
            "targetFieldName": "tags"
        },
        {
            "sourceFieldName": "/document/normalized_images/*/description",
            "targetFieldName": "description"
        },
        {
            "sourceFieldName": "/document/normalized_images/*/faces/*",
            "targetFieldName": "faces"
        },
        {
            "sourceFieldName": "/document/normalized_images/*/brands/*/name",
            "targetFieldName": "brands"
        }
```
### <a name="variation-on-output-field-mappings-nested-properties"></a>Variación en las asignaciones de campos de salida (propiedades anidadas)

Puede definir asignaciones de campos de salida para propiedades de nivel inferior, como puntos de referencia o celebridades. En este caso, asegúrese de que el esquema de índice tiene un campo para contener puntos de referencia específicamente.

```json
    "outputFieldMappings": [
        {
            "sourceFieldName": "/document/normalized_images/*/categories/detail/celebrities/*",
            "targetFieldName": "celebrities"
        }
```
##  <a name="sample-input"></a>Entrada de ejemplo

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "image": {
                    "data": "BASE64 ENCODED STRING OF A JPEG IMAGE",
                    "width": 500,
                    "height": 300,
                    "originalWidth": 5000,
                    "originalHeight": 3000,
                    "rotationFromOriginal": 90,
                    "contentOffset": 500,
                    "pageNumber": 2
                }
            }
        }
    ]
}
```

##  <a name="sample-output"></a>Salida de ejemplo

```json
{
  "values": [
    {
      "recordId": "1",
      "data": {
        "categories": [
          {
            "name": "abstract_",
            "score": 0.00390625
          },
          {
            "name": "people_",
            "score": 0.83984375,
            "detail": {
              "celebrities": [
                {
                  "name": "Satya Nadella",
                  "faceBoundingBox": [
                        {
                            "x": 273,
                            "y": 309
                        },
                        {
                            "x": 395,
                            "y": 309
                        },
                        {
                            "x": 395,
                            "y": 431
                        },
                        {
                            "x": 273,
                            "y": 431
                        }
                    ],
                  "confidence": 0.999028444
                }
              ],
              "landmarks": [
                {
                  "name": "Forbidden City",
                  "confidence": 0.9978346
                }
              ]
            }
          }
        ],
        "adult": {
          "isAdultContent": false,
          "isRacyContent": false,
          "isGoryContent": false,
          "adultScore": 0.0934349000453949,
          "racyScore": 0.068613491952419281,
          "goreScore": 0.08928389008070282
        },
        "tags": [
          {
            "name": "person",
            "confidence": 0.98979085683822632
          },
          {
            "name": "man",
            "confidence": 0.94493889808654785
          },
          {
            "name": "outdoor",
            "confidence": 0.938492476940155
          },
          {
            "name": "window",
            "confidence": 0.89513939619064331
          }
        ],
        "description": {
          "tags": [
            "person",
            "man",
            "outdoor",
            "window",
            "glasses"
          ],
          "captions": [
            {
              "text": "Satya Nadella sitting on a bench",
              "confidence": 0.48293603002174407
            }
          ]
        },
        "requestId": "0dbec5ad-a3d3-4f7e-96b4-dfd57efe967d",
        "metadata": {
          "width": 1500,
          "height": 1000,
          "format": "Jpeg"
        },
        "faces": [
          {
            "age": 44,
            "gender": "Male",
            "faceBoundingBox": [
                {
                    "x": 1601,
                    "y": 395
                },
                {
                    "x": 1653,
                    "y": 395
                },
                {
                    "x": 1653,
                    "y": 447
                },
                {
                    "x": 1601,
                    "y": 447
                }
            ]
          }
        ],
        "objects": [
          {
            "rectangle": {
              "x": 25,
              "y": 43,
              "w": 172,
              "h": 140
            },
            "object": "person",
            "confidence": 0.931
          }
        ],
        "brands":[  
           {  
              "name":"Microsoft",
              "confidence": 0.903,
              "rectangle":{  
                 "x":20,
                 "y":97,
                 "w":62,
                 "h":52
              }
           }
        ]
      }
    }
  ]
}
```


## <a name="error-cases"></a>Casos de error
En los siguientes casos de error, no se extrae ningún elemento.

| Código de error | Descripción |
|------------|-------------|
| `NotSupportedLanguage` | El idioma proporcionado no se admite. |
| `InvalidImageUrl` | La dirección URL de la imagen tiene un formato que no se admite o no es accesible.|
| `InvalidImageFormat` | Los datos de entrada no son una imagen válida. |
| `InvalidImageSize` | La imagen de entrada es demasiado grande. |
| `NotSupportedVisualFeature`  | El tipo de característica especificado no es válido. |
| `NotSupportedImage` | La imagen no se admite como, por ejemplo, pornografía infantil. |
| `InvalidDetails` | El modelo específico del dominio no se admite. |

Si recibe el error similar a `"One or more skills are invalid. Details: Error in skill #<num>: Outputs are not supported by skill: Landmarks"`, compruebe la ruta de acceso. Tanto las celebridades como los puntos de referencia son propiedades en `detail`.

```json
"categories":[  
      {  
         "name":"building_",
         "score":0.97265625,
         "detail":{  
            "landmarks":[  
               {  
                  "name":"Forbidden City",
                  "confidence":0.92013400793075562
               }
            ]
```

## <a name="see-also"></a>Consulte también

+ [Aptitudes integradas](cognitive-search-predefined-skills.md)
+ [Definición de un conjunto de aptitudes](cognitive-search-defining-skillset.md)
+ [Create Indexer (REST)](/rest/api/searchservice/create-indexer)