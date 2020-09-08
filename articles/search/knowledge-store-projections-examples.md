---
title: Definición de proyecciones en un almacén de conocimiento
titleSuffix: Azure Cognitive Search
description: Ejemplos de patrones comunes sobre cómo proyectar documentos enriquecidos en el almacén de información para usarse con Power BI o Azure ML.
manager: nitinme
author: vkurpad
ms.author: vikurpad
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: 18646fc92dda7cebb7a8c1030285956d8fe4df74
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88924170"
---
# <a name="how-to-shape-and-export-enrichments"></a>Dar forma y exportar enriquecimientos

Las proyecciones son la expresión física de los documentos enriquecidos en un almacén de conocimiento. El uso eficaz de los documentos enriquecidos requiere una estructura. En este artículo, explorará la estructura y las relaciones; aprenderá a crear propiedades de proyección y a establecer relaciones entre los datos de los distintos tipos de proyección que cree. 

Para crear una proyección, a los datos se dan forma mediante la [aptitud Conformador](cognitive-search-skill-shaper.md) para crear un objeto personalizado, o bien usar la sintaxis de moldeado en línea en la definición de un proyecto. 

Las formas de datos contienen todos los datos que se quieren proyectar en forma de jerarquía de nodos. En este artículo se muestran varias técnicas para dar forma a los datos, con el fin de que se puedan proyectar en estructuras físicas que favorecen la creación de informes, los análisis o el procesamiento de bajada. 

Los ejemplos que se incluyen en este artículo se pueden encontrar en este [ejemplo de API REST](https://github.com/Azure-Samples/azure-search-postman-samples/blob/master/projections/Projections%20Docs.postman_collection.json) que puede descargar y ejecutar en un cliente HTTP.

## <a name="introduction-to-projection-examples"></a>Introducción a los ejemplos de proyección

Hay tres tipos de [proyecciones](knowledge-store-projection-overview.md):

+ Tablas
+ Objetos
+ Archivos

Las proyecciones de tabla se almacenan en Azure Table Storage. Tanto las proyecciones de objeto como las de archivo se escriben en el almacenamiento de blobs, donde las proyecciones de objeto se guardan como archivos JSON, y pueden incluir contenido del documento de origen, así como las salidas o características enriquecidas de cualquier aptitud. La canalización de enriquecimiento también puede extraer archivos binarios, como las imágenes. Estos archivos binarios se proyectan como proyecciones de archivos. Cuando un objeto binario se proyecta como proyección de objeto, solo se guardan como blob JSON los metadatos asociados a él. 

Para comprender la relación entre el modelado de datos y las proyecciones, usaremos el siguiente conjunto de aptitudes como base para explorar diversas configuraciones. Este conjunto de aptitudes procesa contenido de texto e imágenes sin procesar. Las proyecciones se definirán a partir del contenido del documento y de los resultados de las aptitudes para los escenarios deseados.

> [!IMPORTANT] 
> Al experimentar con las proyecciones, resulta útil [establecer la propiedad de la caché del indizador](search-howto-incremental-index.md) para asegurarse de controlar los costos. La edición de proyecciones dará lugar a que todo el documento se vuelva a enriquecer si no se establece la memoria caché del indizador. Cuando se establece la memoria caché y solo se actualizan las proyecciones, las ejecuciones de los conjuntos de aptitudes en documentos enriquecidos previamente no generan nuevos costos de Cognitive Services.

```json
{
    "name": "azureblob-skillset",
    "description": "Skillset created from the portal. skillsetName: azureblob-skillset; contentField: merged_content; enrichmentGranularity: document; knowledgeStoreStorageAccount: confdemo;",
    "skills": [
        {
            "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
            "name": "#1",
            "description": null,
            "context": "/document/merged_content",
            "categories": [
                "Person",
                "Quantity",
                "Organization",
                "URL",
                "Email",
                "Location",
                "DateTime"
            ],
            "defaultLanguageCode": "en",
            "minimumPrecision": null,
            "includeTypelessEntities": null,
            "inputs": [
                {
                    "name": "text",
                    "source": "/document/merged_content"
                },
                {
                    "name": "languageCode",
                    "source": "/document/language"
                }
            ],
            "outputs": [
                {
                    "name": "persons",
                    "targetName": "people"
                },
                {
                    "name": "organizations",
                    "targetName": "organizations"
                },
                {
                    "name": "locations",
                    "targetName": "locations"
                },
                {
                    "name": "entities",
                    "targetName": "entities"
                }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.KeyPhraseExtractionSkill",
            "name": "#2",
            "description": null,
            "context": "/document/merged_content",
            "defaultLanguageCode": "en",
            "maxKeyPhraseCount": null,
            "inputs": [
                {
                    "name": "text",
                    "source": "/document/merged_content"
                },
                {
                    "name": "languageCode",
                    "source": "/document/language"
                }
            ],
            "outputs": [
                {
                    "name": "keyPhrases",
                    "targetName": "keyphrases"
                }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.LanguageDetectionSkill",
            "name": "#3",
            "description": null,
            "context": "/document",
            "inputs": [
                {
                    "name": "text",
                    "source": "/document/merged_content"
                }
            ],
            "outputs": [
                {
                    "name": "languageCode",
                    "targetName": "language"
                }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.MergeSkill",
            "name": "#4",
            "description": null,
            "context": "/document",
            "insertPreTag": " ",
            "insertPostTag": " ",
            "inputs": [
                {
                    "name": "text",
                    "source": "/document/content"
                },
                {
                    "name": "itemsToInsert",
                    "source": "/document/normalized_images/*/text"
                },
                {
                    "name": "offsets",
                    "source": "/document/normalized_images/*/contentOffset"
                }
            ],
            "outputs": [
                {
                    "name": "mergedText",
                    "targetName": "merged_content"
                }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Vision.OcrSkill",
            "name": "#5",
            "description": null,
            "context": "/document/normalized_images/*",
            "textExtractionAlgorithm": "printed",
            "lineEnding": "Space",
            "defaultLanguageCode": "en",
            "detectOrientation": true,
            "inputs": [
                {
                    "name": "image",
                    "source": "/document/normalized_images/*"
                }
            ],
            "outputs": [
                {
                    "name": "text",
                    "targetName": "text"
                },
                {
                    "name": "layoutText",
                    "targetName": "layoutText"
                }
            ]
        }
    ],
    "cognitiveServices": {
        "@odata.type": "#Microsoft.Azure.Search.CognitiveServicesByKey",
        "description": "DemosCS",
        "key": "<COGNITIVE SERVICES KEY>"
    },
    "knowledgeStore": null
}
```

Con este conjunto de aptitudes, con su valor `knowledgeStore` establecido en null como base, en el primer ejemplo se rellena el objeto `knowledgeStore`, configurado con proyecciones que crean estructuras de datos tabulares que se pueden usar en otros escenarios. 

## <a name="projecting-to-tables"></a>Proyección en tablas

En Azure Storage, la proyección en tablas es útil para la generación de informes y análisis mediante herramientas como Power BI. Power BI puede leer de tablas y detectar relaciones en función de las claves que se generen durante la proyección. Si intenta compilar un panel, tener datos relacionados simplificará la tarea. 

Vamos a crear un panel para visualizar las frases clave extraídas de los documentos como una nube de palabras. Para crear la estructura de datos correcta, agregamos la aptitud Conformador al conjunto de aptitudes para crear una forma personalizada que tenga los detalles específicos del documento y frases clave. La forma personalizada se llamará `pbiShape` en el nodo raíz `document`.

> [!NOTE] 
> Las proyecciones de tabla son tablas de Azure Storage, reguladas por los límites de almacenamiento que impone Azure Storage. Para más información, consulte el artículo acerca de los [límites del almacenamiento en tablas](/rest/api/storageservices/understanding-the-table-service-data-model). Debe saber que el tamaño de la entidad no puede superar 1 MB, y que una sola propiedad no puede ser mayor que 64 KB. Estas restricciones hacen que las tablas sean una buena solución para almacenar un gran número de entidades pequeñas.

### <a name="using-a-shaper-skill-to-create-a-custom-shape"></a>Uso de una aptitud modeladora para crear una forma personalizada

Cree una forma personalizada que se pueda proyectar en el almacenamiento de tablas. Sin una forma personalizada, una proyección solo puede hacer referencia a un solo nodo (una proyección por salida). La creación de una forma personalizada agrega varios elementos a un nuevo conjunto lógico que se puede proyectar como una sola tabla, o que se puede segmentar y distribuir en una colección de tablas. 

En este ejemplo, la forma personalizada combina los metadatos con las entidades y frases clave identificadas. El objeto se denomina `pbiShape` y tiene como elemento primario `/document`. 

> [!IMPORTANT] 
> Un propósito de la creación de formas es asegurarse de que todos los nodos de enriquecimiento se expresan en código JSON bien formado, lo cual es necesario para proyectar en el almacén de conocimiento. Esto sucede, sobre todo cuando un árbol de enriquecimiento contiene nodos que no son código JSON bien formado (por ejemplo, cuando un enriquecimiento tiene como elemento primario un tipo primitivo como una cadena).
>
> Observe los dos últimos nodos, `KeyPhrases` y `Entities`. Estos se encapsulan en un objeto JSON válido con `sourceContext`. Este formato es obligatorio, ya que `keyphrases` y `entities` son enriquecimientos de primitivos y deben convertirse en código JSON válido antes de proyectarse.
>


```json
{
    "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
    "name": "ShaperForTables",
    "description": null,
    "context": "/document",
    "inputs": [
        {
            "name": "metadata_storage_content_type",
            "source": "/document/metadata_storage_content_type",
            "sourceContext": null,
            "inputs": []
        },
        {
            "name": "metadata_storage_name",
            "source": "/document/metadata_storage_name",
            "sourceContext": null,
            "inputs": []
        },
        {
            "name": "metadata_storage_path",
            "source": "/document/metadata_storage_path",
            "sourceContext": null,
            "inputs": []
        },
        {
            "name": "metadata_content_type",
            "source": "/document/metadata_content_type",
            "sourceContext": null,
            "inputs": []
        },
        {
            "name": "keyPhrases",
            "source": null,
            "sourceContext": "/document/merged_content/keyphrases/*",
            "inputs": [
                {
                    "name": "KeyPhrases",
                    "source": "/document/merged_content/keyphrases/*"
                }

            ]
        },
        {
            "name": "Entities",
            "source": null,
            "sourceContext": "/document/merged_content/entities/*",
            "inputs": [
                {
                    "name": "Entities",
                    "source": "/document/merged_content/entities/*/name"
                }

            ]
        }
    ],
    "outputs": [
        {
            "name": "output",
            "targetName": "pbiShape"
        }
    ]
}
```

Agregue la aptitud Conformador anterior al conjunto de aptitudes. 

```json
    "name": "azureblob-skillset",
    "description": "A friendly description of the skillset goes here.",
    "skills": [
        {
            Shaper skill goes here
            }
        ],
    "cognitiveServices":  "A key goes here",
    "knowledgeStore": []
}  
```

Ahora que tenemos todos los datos necesarios para proyectar en tablas, actualice el objeto knowledgeStore con las definiciones de tabla. En este ejemplo tenemos tres tablas, para cuya definición se establecen las propiedades `tableName`, `source` y `generatedKeyName`.

```json
"knowledgeStore" : {
    "storageConnectionString": "DefaultEndpointsProtocol=https;AccountName=<Acct Name>;AccountKey=<Acct Key>;",
    "projections": [
        {
            "tables": [
                {
                    "tableName": "pbiDocument",
                    "generatedKeyName": "Documentid",
                    "source": "/document/pbiShape"
                },
                {
                    "tableName": "pbiKeyPhrases",
                    "generatedKeyName": "KeyPhraseid",
                    "source": "/document/pbiShape/keyPhrases/*"
                },
                {
                    "tableName": "pbiEntities",
                    "generatedKeyName": "Entityid",
                    "source": "/document/pbiShape/Entities/*"
                }
            ],
            "objects": [],
            "files": []
        }
    ]
}
```

Puede procesar su trabajo mediante estos pasos:

1. Establezca la propiedad ```storageConnectionString``` en una cadena de conexión válida de una cuenta de almacenamiento de uso general V2.  

1. Actualice el conjunto de aptitudes. Para ello, emita la solicitud PUT.

1. Después de actualizar el conjunto de aptitudes, ejecute el indexador. 

Ya tiene una proyección activa con tres tablas. La importación de estas tablas en Power BI debería permitir que este detecte automáticamente las relaciones.

Antes de pasar al siguiente ejemplo, vamos a volver a los distintos aspectos de la proyección de tabla para conocer la mecánica de la segmentación y relación de datos.

### <a name="slicing"></a>Segmentación 

La segmentación es una técnica que subdivide una forma completa consolidada en las partes que la conforman. El resultado consta de tablas independientes, pero relacionadas, con las que se puede trabajar de forma individual.

En el ejemplo, `pbiShape` es la forma consolidada (o el nodo de enriquecimiento). En la definición de la proyección, `pbiShape` se segmenta en tablas adicionales, lo que le permite extraer las partes de la forma, ```keyPhrases``` y ```Entities```. En Power BI, esto es útil, ya que hay varias entidades y keyPhrases asociadas con cada documento, y obtendrá más información si puede ver entidades y keyPhrases como datos con categorías.

La segmentación genera de forma implícita una relación entre las tablas primaria y secundaria, y usa ```generatedKeyName``` de la tabla primaria para crear una columna con el mismo nombre en la tabla secundaria. 

### <a name="naming-relationships"></a>Nomenclatura de relaciones

Las propiedades ```generatedKeyName``` y ```referenceKeyName``` se usan para establecer relaciones entre los datos de distintas tablas o incluso tipos de proyección. Cada fila de la tabla secundaria o proyección tiene una propiedad que apunta de vuelta al elemento primario. El nombre de la columna o propiedad en el elemento secundario es el ```referenceKeyName``` del elemento primario. Cuando no se proporciona el ```referenceKeyName```, el servicio usa como valor predeterminado el ```generatedKeyName``` del elemento primario. 

Power BI usa estas claves generadas para detectar relaciones en las tablas. Si necesita que la columna de la tabla secundaria tenga un nombre diferente, establezca la propiedad ```referenceKeyName``` en la tabla primaria. Como ejemplo, podría establecer ```generatedKeyName``` como id. en la tabla pbiDocument y ```referenceKeyName``` como DocumentID. Como resultado, la columna de las tablas pbiEntities y pbiKeyPhrases que contiene el id. del documento tendría por nombre DocumentID.

## <a name="projecting-to-objects"></a>Proyección en objetos

Las proyecciones de objeto no tienen las mismas limitaciones que las proyecciones de tabla y resultan más adecuadas para proyectar documentos grandes. En este ejemplo, el documento completo se envía como una proyección de objeto. Las proyecciones de objeto están limitadas a una sola proyección en un contenedor y no se pueden segmentar.

Para definir una proyección de objeto, se usa la matriz ```objects``` en las proyecciones. Puede generar una nueva forma mediante la aptitud Conformador o usar el modelado insertado de la proyección de objeto. Aunque en el ejemplo de tablas se mostró el método para crear una forma y segmentarla, en este ejemplo se muestra el uso del modelado insertado. 

El modelado insertado es la capacidad de crear una forma en la definición de las entradas para una proyección. El modelado insertado crea un objeto anónimo que es idéntico a lo que produciría una aptitud Conformador (en nuestro caso, `pbiShape`). El modelado insertado resulta útil si está definiendo una forma que no se va a reutilizar.

La propiedad projections es una matriz. En este ejemplo se agrega una nueva instancia de proyección a la matriz, donde la definición de knowledgeStore contiene proyecciones insertadas. Cuando use proyecciones insertadas, puede omitir la aptitud Conformador.

```json
"knowledgeStore" : {
        "storageConnectionString": "DefaultEndpointsProtocol=https;AccountName=<Acct Name>;AccountKey=<Acct Key>;",
        "projections": [
             {
                "tables": [ ],
                "objects": [
                    {
                        "storageContainer": "sampleobject",
                        "source": null,
                        "generatedKeyName": "myobject",
                        "sourceContext": "/document",
                        "inputs": [
                            {
                                "name": "metadata_storage_name",
                                "source": "/document/metadata_storage_name"
                            },
                            {
                                "name": "metadata_storage_path",
                                "source": "/document/metadata_storage_path"
                            },
                            {
                                "name": "content",
                                "source": "/document/content"
                            },
                            {
                                "name": "keyPhrases",
                                "source": "/document/merged_content/keyphrases/*"
                            },
                            {
                                "name": "entities",
                                "source": "/document/merged_content/entities/*/name"
                            },
                            {
                                "name": "ocrText",
                                "source": "/document/normalized_images/*/text"
                            },
                            {
                                "name": "ocrLayoutText",
                                "source": "/document/normalized_images/*/layoutText"
                            }
                        ]

                    }
                ],
                "files": []
            }
        ]
    }
```

## <a name="projecting-to-file"></a>Proyección en archivo

Las proyecciones de archivo son imágenes que se extraen del documento de origen o salidas de enriquecimiento que se pueden proyectar fuera del proceso de enriquecimiento. Las proyecciones de archivo, de forma similar a las proyecciones de objeto, se implementan como blobs en Azure Storage y contienen la imagen. 

Para generar una proyección de archivo, se usa la matriz `files` en el objeto projection. En este ejemplo, se proyectan todas las imágenes extraídas del documento en un contenedor denominado `samplefile`.

```json
"knowledgeStore" : {
        "storageConnectionString": "DefaultEndpointsProtocol=https;AccountName=<Acct Name>;AccountKey=<Acct Key>;",
        "projections": [
            {
                "tables": [ ],
                "objects": [ ],
                "files": [
                    {
                        "storageContainer": "samplefile",
                        "source": "/document/normalized_images/*"
                    }
                ]
            }
        ]
    }
```

## <a name="projecting-to-multiple-types"></a>Proyección en varios tipos

Un escenario más complejo podría requerir que se proyecte contenido en distintos tipos de proyección. Por ejemplo, si necesita proyectar datos como frases clave y entidades en tablas, guarde los resultados de OCR del texto y el texto de diseño como objetos y, a continuación, proyecte las imágenes como archivos. 

En este ejemplo se actualizan conjuntos de aptitudes con los siguientes cambios:

1. Crear una tabla con una fila para cada documento.
1. Crear una tabla relacionada con la tabla de documento, en la que cada frase clave se identificará como una fila en la tabla.
1. Crear una tabla relacionada con la tabla de documento, en la que cada entidad se identificará como una fila en la tabla.
1. Crear una proyección de objeto con el texto de diseño de cada imagen.
1. Crear una proyección de archivo en la que se proyecta cada imagen extraída.
1. Crear una tabla de referencias cruzadas que contenga referencias a la tabla de documentos, la proyección de objeto con el texto de diseño y la proyección de archivo.

Estos cambios se reflejan en la definición de knowledgeStore, que encontrará a continuación. 

### <a name="shape-data-for-cross-projection"></a>Dar forma a datos para la proyección cruzada

Para obtener las formas que necesitamos para estas proyecciones, empiece por agregar una nueva aptitud Conformador que cree un objeto con forma llamado `crossProjection`. 

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
    "name": "ShaperForCross",
    "description": null,
    "context": "/document",
    "inputs": [
        {
            "name": "metadata_storage_name",
            "source": "/document/metadata_storage_name",
            "sourceContext": null,
            "inputs": []
        },
        {
            "name": "keyPhrases",
            "source": null,
            "sourceContext": "/document/merged_content/keyphrases/*",
            "inputs": [
                {
                    "name": "KeyPhrases",
                    "source": "/document/merged_content/keyphrases/*"
                }

            ]
        },
        {
            "name": "entities",
            "source": null,
            "sourceContext": "/document/merged_content/entities/*",
            "inputs": [
                {
                    "name": "Entities",
                    "source": "/document/merged_content/entities/*/name"
                }

            ]
        },
        {
            "name": "images",
            "source": null,
            "sourceContext": "/document/normalized_images/*",
            "inputs": [
                {
                    "name": "image",
                    "source": "/document/normalized_images/*"
                },
                {
                    "name": "layoutText",
                    "source": "/document/normalized_images/*/layoutText"
                },
                {
                    "name": "ocrText",
                    "source": "/document/normalized_images/*/text"
                }
                ]
        }
 
    ],
    "outputs": [
        {
            "name": "output",
            "targetName": "crossProjection"
        }
    ]
}
```

### <a name="define-table-object-and-file-projections"></a>Definición de proyecciones de tabla, objeto y archivo

Desde el objeto crossProjection consolidado, segmente el objeto en varias tablas, capture la salida de OCR como blobs y, después, guarde la imagen como archivos (también en Blob Storage).

```json
"knowledgeStore" : {
        "storageConnectionString": "DefaultEndpointsProtocol=https;AccountName=<Acct Name>;AccountKey=<Acct Key>;",
        "projections": [
             {
                "tables": [
                    {
                        "tableName": "crossDocument",
                        "generatedKeyName": "Id",
                        "source": "/document/crossProjection"
                    },
                    {
                        "tableName": "crossEntities",
                        "generatedKeyName": "EntityId",
                        "source": "/document/crossProjection/entities/*"
                    },
                    {
                        "tableName": "crossKeyPhrases",
                        "generatedKeyName": "KeyPhraseId",
                        "source": "/document/crossProjection/keyPhrases/*"
                    },
                    {
                        "tableName": "crossReference",
                        "generatedKeyName": "CrossId",
                        "source": "/document/crossProjection/images/*"
                    }
                     
                ],
                "objects": [
                    {
                        "storageContainer": "crossobject",
                        "generatedKeyName": "crosslayout",
                        "source": null,
                        "sourceContext": "/document/crossProjection/images/*/layoutText",
                        "inputs": [
                            {
                                "name": "OcrLayoutText",
                                "source": "/document/crossProjection/images/*/layoutText"
                            }
                        ]
                    }
                ],
                "files": [
                     {
                        "storageContainer": "crossimages",
                        "generatedKeyName": "crossimages",
                        "source": "/document/crossProjection/images/*/image"
                    }
                    ]
                
            }
        ]
    }
```

Las proyecciones de objeto requieren un nombre de contenedor para cada proyección; las proyecciones de objeto o archivo no pueden compartir un contenedor. 

### <a name="relationships-among-table-object-and-file-projections"></a>Relaciones entre las proyecciones de tabla, objeto y archivo

En este ejemplo también se resalta otra característica de las proyecciones. Al definir varios tipos de proyecciones en el mismo objeto de proyección, existe una relación expresada en y entre los distintos tipos (tablas, objetos, archivos). Esto le permite empezar con una fila de tabla para un documento y buscar todo el texto de OCR de las imágenes de ese documento en la proyección de objeto. 

Si no desea los datos relacionados, defina las proyecciones en distintos objetos de proyección. Por ejemplo, el siguiente fragmento provocará que las tablas estén relacionadas, pero sin relaciones entre las tablas y las proyecciones de objeto (texto de OCR). 

Los grupos de proyección son útiles cuando se quieren proyectar los mismos datos en diferentes formas con distintas necesidades. Por ejemplo, un grupo de proyección para el panel de Power BI y otro para capturar los datos que se usan para entrenar un modelo de Machine Learning encapsulado en una aptitud personalizada.

Cuando se crean proyecciones de distintos tipos, primero se generan las proyecciones de archivo y objeto, y se agregan rutas de acceso a las tablas.

```json
"knowledgeStore" : {
        "storageConnectionString": "DefaultEndpointsProtocol=https;AccountName=<Acct Name>;AccountKey=<Acct Key>;",
        "projections": [
            {
                "tables": [
                    {
                        "tableName": "unrelatedDocument",
                        "generatedKeyName": "Documentid",
                        "source": "/document/pbiShape"
                    },
                    {
                        "tableName": "unrelatedKeyPhrases",
                        "generatedKeyName": "KeyPhraseid",
                        "source": "/document/pbiShape/keyPhrases"
                    }
                ],
                "objects": [
                    
                ],
                "files": []
            }, 
            {
                "tables": [],
                "objects": [
                    {
                        "storageContainer": "unrelatedocrtext",
                        "source": null,
                        "sourceContext": "/document/normalized_images/*/text",
                        "inputs": [
                            {
                                "name": "ocrText",
                                "source": "/document/normalized_images/*/text"
                            }
                        ]
                    },
                    {
                        "storageContainer": "unrelatedocrlayout",
                        "source": null,
                        "sourceContext": "/document/normalized_images/*/layoutText",
                        "inputs": [
                            {
                                "name": "ocrLayoutText",
                                "source": "/document/normalized_images/*/layoutText"
                            }
                        ]
                    }
                ],
                "files": []
            }
        ]
    }
```

## <a name="common-issues"></a>Problemas comunes

Al definir una proyección, hay algunos problemas comunes que pueden generar resultados inesperados. Compruebe estas incidencias si la salida del almacén de conocimiento no es la que se espera.

+ Características enriquecidas de cadena sin modelar en un código JSON válido. Cuando las cadenas están enriquecidas; por ejemplo, `merged_content` se ha enriquecido con frases clave, la propiedad enriquecida se representa como un elemento secundario de `merged_content` en el árbol de enriquecimiento. La representación predeterminada no es código JSON bien formado. Por tanto, en el momento de la proyección, asegúrese de que transforma el enriquecimiento en un objeto JSON válido con un nombre y un valor.

+ Omisión de ```/*``` al final de una ruta de acceso de origen. Si el origen de una proyección es `/document/pbiShape/keyPhrases`, la matriz de frases clave se proyecta como un solo objeto o fila. En su lugar, establezca la ruta de acceso de origen en `/document/pbiShape/keyPhrases/*` para generar una sola fila u objeto para cada una de las frases clave.

+ Errores de sintaxis de la ruta de acceso. Los selectores de ruta de acceso distinguen mayúsculas de minúsculas y pueden generar advertencias por entradas que faltan si no se usan las mayúsculas y minúsculas exactas en el selector.

## <a name="next-steps"></a>Pasos siguientes

Los ejemplos de este artículo muestran patrones comunes sobre cómo crear proyecciones. Ahora que conoce los conceptos, está mejor preparado para compilar proyecciones para su escenario concreto.

Cuando explore nuevas características, considere el enriquecimiento incremental como el siguiente paso. El enriquecimiento incremental se basa en el almacenamiento en caché, lo cual le permite volver a usar todos los enriquecimientos que no se hayan visto afectados por una modificación del conjunto de aptitudes. Esto es especialmente útil para aquellas canalizaciones que incluyen OCR y análisis de imágenes.

> [!div class="nextstepaction"]
> [Introducción al enriquecimiento incremental y al almacenamiento en caché](cognitive-search-incremental-indexing-conceptual.md)

Para obtener información general sobre las proyecciones, obtenga más información sobre funcionalidades como los grupos y la segmentación, y cómo [definirlos en un conjunto de aptitudes](knowledge-store-projection-overview.md)

> [!div class="nextstepaction"]
> [Proyecciones en un almacén de conocimiento](knowledge-store-projection-overview.md)