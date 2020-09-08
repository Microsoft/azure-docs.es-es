---
title: Obtención de información sobre conversiones
description: Obtención de información sobre conversiones
author: malcolmtyrrell
ms.author: matyrr
ms.date: 03/05/2020
ms.topic: how-to
ms.openlocfilehash: 529bfb61b3af7040f3656c04071683841f5abe86
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88870296"
---
# <a name="get-information-about-conversions"></a>Obtención de información sobre conversiones

## <a name="information-about-a-conversion-the-result-file"></a>Información sobre una conversión: El archivo de resultados

Cuando el servicio de conversión convierte un recurso, escribe un resumen de los problemas en un "archivo de resultados". Por ejemplo, si se convierte un archivo `buggy.gltf`, el contenedor de salida incluirá un archivo llamado `buggy.result.json`.

El archivo de resultados muestra los errores y las advertencias que se produjeron durante la conversión y proporciona un resumen del resultado, que puede ser `succeeded`, `failed` o `succeeded with warnings`.
El archivo de resultados está estructurado como una matriz JSON de objetos, cada uno de los cuales tiene una propiedad de cadena que puede ser `warning`, `error`, `internal warning`, `internal error` o `result`. Habrá como máximo un error ( `error` o `internal error`) y siempre habrá un valor de `result`.

## <a name="example-result-file"></a>Ejemplo de archivo de *resultados*

En el siguiente ejemplo se describe una conversión que generó correctamente un archivo arrAsset. Sin embargo, puesto que faltaba una textura, el archivo arrAsset resultante podría no ser el previsto.

```JSON
[
  {"warning":"4004","title":"Missing texture","details":{"texture":"buggy_baseColor.png","material":"buggy_col"}},
  {"result":"succeeded with warnings"}
]
```

## <a name="information-about-a-converted-model-the-info-file"></a>Información sobre un modelo convertido: El archivo de información

El archivo arrAsset generado por el servicio de conversión está diseñado exclusivamente para el consumo por parte del servicio de representación. Sin embargo, puede haber ocasiones en las que quiera acceder a información sobre un modelo sin iniciar una sesión de representación. Para admitir este flujo de trabajo, el servicio de conversión coloca un archivo JSON junto al archivo arrAsset en el contenedor de salida. Por ejemplo, si se convierte un archivo `buggy.gltf`, el contenedor de salida incluirá un archivo denominado `buggy.info.json` junto al archivo convertido `buggy.arrAsset`. Este archivo contiene información sobre el modelo de origen, el modelo convertido y sobre la propia conversión.

## <a name="example-info-file"></a>Archivo de *información* de ejemplo

Este es un archivo de *información* de ejemplo generado al convertir un archivo llamado `buggy.gltf`:

```JSON
{
    "files": {
        "input": "Buggy.gltf"
    },
    "conversionSettings": {
        "recenterToOrigin": true
    },
    "inputInfo": {
        "sourceAssetExtension": ".gltf",
        "sourceAssetFormat": "glTF2 Importer",
        "sourceAssetFormatVersion": "2.0",
        "sourceAssetGenerator": "COLLADA2GLTF"
    },
    "inputStatistics": {
        "numMeshes": 148,
        "numFaces": 308306,
        "numVertices": 245673,
        "numMaterial": 149,
        "numFacesSmallestMesh": 2,
        "numFacesBiggestMesh": 8764,
        "numNodes": 206,
        "numMeshUsagesInScene": 236,
        "maxNodeDepth": 3
    },
    "outputInfo": {
        "conversionToolVersion": "3b28d840de9916f9d628342f474d38c3ab949590",
        "conversionHash": "CCDB1F7A4C09F565"
    },
    "outputStatistics": {
        "numMeshPartsCreated": 236,
        "numMeshPartsInstanced": 88,
        "recenteringOffset": [
            -24.1,
            -50.9,
            -16.5974
        ],
        "boundingBox": {
            "min": [
                -43.52,
                -61.775,
                -79.6416
            ],
            "max": [
                43.52,
                61.775,
                79.6416
            ]
        }
    }
}
```

## <a name="information-in-the-info-file"></a>Información en el archivo de información

### <a name="the-files-section"></a>La sección *files*

Esta sección contiene los nombres de archivo proporcionados.

* `input`: nombre del archivo de origen.
* `output`: nombre del archivo de salida, cuando el usuario ha especificado un nombre no predeterminado.

### <a name="the-conversionsettings-section"></a>La sección *conversionSettings*

En esta sección se incluye una copia de la sección [ConversionSettings](configure-model-conversion.md#settings-file) especificada cuando se convirtió el modelo.

### <a name="the-inputinfo-section"></a>La sección *inputInfo*

En esta sección se registra información sobre el formato de archivo de origen.

* `sourceAssetExtension`: la extensión de archivo del archivo de origen.
* `sourceAssetFormat`: descripción del formato del archivo de origen.
* `sourceAssetFormatVersion`: versión del formato del archivo de origen.
* `sourceAssetGenerator`: nombre de la herramienta que generó el archivo de origen, si está disponible.

### <a name="the-inputstatistics-section"></a>La sección *inputStatistics*

Esta sección proporciona información sobre la escena de origen. A menudo habrá discrepancias entre los valores de esta sección y los valores equivalentes de la herramienta que creó el modelo de origen. Tales diferencias están previstas, ya que el modelo se modifica durante los pasos de exportación y conversión.

* `numMeshes`: número de partes de la malla, donde cada parte puede hacer referencia a un único material.
* `numFaces`: número total de _triángulos_ en todo el modelo. Tenga en cuenta que la malla se triangula durante la conversión. Este número contribuye al límite de polígonos en el [tamaño del servidor de representación estándar](../../reference/vm-sizes.md#how-the-renderer-evaluates-the-number-of-polygons).
* `numVertices`: número total de vértices en todo el modelo.
* `numMaterial`: número total de materiales en todo el modelo.
* `numFacesSmallestMesh`: número de triángulos en la malla más pequeña del modelo.
* `numFacesBiggestMesh`: número de triángulos en la malla más grande del modelo.
* `numNodes`: número de nodos del gráfico de escenas del modelo.
* `numMeshUsagesInScene`: número de veces que los nodos hacen referencia a las mallas. Más de un nodo puede hacer referencia a la misma malla.
* `maxNodeDepth`: profundidad máxima de los nodos dentro del gráfico de escenas.

### <a name="the-outputinfo-section"></a>La sección *outputInfo*

En esta sección se registra información general sobre la salida generada.

* `conversionToolVersion`: versión del convertidor de modelos.
* `conversionHash`: hash de los datos dentro de arrAsset que pueden contribuir a la representación. Se puede usar para saber si el servicio de conversión ha producido un resultado diferente cuando se vuelve a ejecutar en el mismo archivo.

### <a name="the-outputstatistics-section"></a>La sección *outputStatistics*

En esta sección se registra la información calculada a partir del recurso convertido.

* `numMeshPartsCreated`: número de mallas en el arrAsset. Puede variar con respecto a `numMeshes` de la sección `inputStatistics`, porque la creación de instancias se ve afectada por el proceso de conversión.
* `numMeshPartsInstanced`: número de mallas que se reutilizan en arrAsset.
* `recenteringOffset`: cuando está habilitada la opción `recenterToOrigin` en [ConversionSettings](configure-model-conversion.md), este valor es el traslado que movería el modelo convertido de vuelta a su posición original.
* `boundingBox`: límites del modelo.

## <a name="deprecated-features"></a>Características en desuso

El servicio de conversión escribe los archivos `stdout.txt` y `stderr.txt` en el contenedor de salida, y estos eran el único origen de advertencias y errores.
Estos archivos ahora están en desuso. Para este fin se usan ahora [archivos de resultados](#information-about-a-conversion-the-result-file).

## <a name="next-steps"></a>Pasos siguientes

* [Conversión de datos](model-conversion.md)
* [Configuración de la conversión de modelos](configure-model-conversion.md)
