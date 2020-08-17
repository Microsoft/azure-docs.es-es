---
title: Reemplazo de materiales durante la conversión de modelos
description: Explica el flujo de trabajo de reemplazo de material en el momento de la conversión.
author: florianborn71
ms.author: flborn
ms.date: 02/13/2020
ms.topic: how-to
ms.openlocfilehash: 2e9cb216c100f1732230a90572284bd3f8462584
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87433138"
---
# <a name="override-materials-during-model-conversion"></a>Reemplazo de materiales durante la conversión de modelos

La configuración de materiales del modelo de origen se usa para definir los [materiales de PBR](../../overview/features/pbr-materials.md) que usa el representador.
A veces, la [conversión predeterminada](../../reference/material-mapping.md) no proporciona los resultados deseados y es necesario realizar cambios.
Cuando se convierte un modelo para su uso en Azure Remote Rendering, se puede proporcionar un archivo de invalidación de materiales para personalizar cómo se realiza la conversión de cada material.
En la sección sobre la [configuración de la conversión de modelos](configure-model-conversion.md) se incluyen instrucciones para declarar el nombre de archivo de invalidación de material.

## <a name="the-override-file-used-during-conversion"></a>Archivo de invalidación usado durante la conversión

Como ejemplo sencillo, supongamos que un modelo de cuadros tiene un único material, denominado "Default".
Además, digamos que el color albedo debe ajustarse para usarse en ARR.
En este caso, se puede crear un archivo `box_materials_override.json` de la siguiente manera:

```json
[
    {
        "name": "Default",
        "albedoColor": {
            "r": 0.33,
            "g": 0.33,
            "b": 0.33,
            "a": 1.0
        }
    }
]
```

El archivo `box_materials_override.json` se coloca en el contenedor de entrada y se agrega un elemento `box.ConversionSettings.json` junto a `box.fbx`, lo que indica a la conversión dónde encontrar el archivo de invalidación (consulte [Configuración de la conversión de modelos](configure-model-conversion.md)):

```json
{
    "material-override" : "box_materials_override.json"
}
```

Cuando se convierta el modelo, se aplicará la nueva configuración.

### <a name="color-materials"></a>Materiales de color

El modelo de [material de color](../../overview/features/color-materials.md) describe una superficie sombreada constantemente que es independiente de la iluminación.
Los materiales de color son útiles para los recursos que se crean con los algoritmos de fotogrametría, por ejemplo.
En los archivos de invalidación de materiales, un material se puede declarar como material de color estableciendo `unlit` en `true`.

```json
[
    {
        "name": "Photogrametry_mat1",
        "unlit" : true
    },
    {
        "name": "Photogrametry_mat2",
        "unlit" : true
    }
]
```

### <a name="ignore-specific-texture-maps"></a>Omitir mapas de texturas específicas

En ocasiones, es posible que quiera que el proceso de conversión ignore determinados mapas de texturas. Este puede ser el caso cuando el modelo se generó mediante una herramienta que genera mapas especiales que el representador no entiende correctamente. Por ejemplo, un elemento "OpacityMap" que se usa para definir un valor distinto de la opacidad o un modelo donde "NormalMap" se almacena como "BumpMap". (En este último caso, es recomendable omitir "NormalMap", lo que hará que el convertidor use "BumpMap" como "NormalMap").

El principio es sencillo. Simplemente, agregue una propiedad llamada `ignoreTextureMaps` y cualquier mapa de texturas que quiera omitir:

```json
[
    {
        "name": "Default",
        "ignoreTextureMaps": ["OpacityMap", "NormalMap"]
    }
]
```

Para obtener la lista completa de mapas de texturas que puede ignorar, vea el siguiente esquema JSON.

## <a name="json-schema"></a>Esquema JSON

Aquí se proporciona el esquema JSON completo para los archivos de materiales. A excepción de `unlit` y `ignoreTextureMaps`, las propiedades disponibles son un subconjunto de las propiedades descritas en las secciones de los modelos de [material de color](../../overview/features/color-materials.md) y [material de PBR](../../overview/features/pbr-materials.md).

```json
{
    "definitions" :
    {
        "color":
        {
            "type" : "object",
            "description" : "Color as 4 components vector",
            "properties":
            {
                "r": {"type":"number"},
                "g": {"type":"number"},
                "b": {"type":"number"},
                "a": {"type":"number"}
            },
            "required": ["r", "g", "b"]
        },
        "alpha":
        {
            "type" : "object",
            "description" : "Alpha channel for color",
            "properties":
            {
                "a": {"type":"number"}
            },
            "required": ["a"]
        },
        "colorOrAlpha":
        {
            "anyOf": [
                {"$ref": "#/definitions/color"},
                {"$ref": "#/definitions/alpha"}
            ]
        },
        "listOfMaps":
        {
            "type": "array",
            "items": {
                "type": "string",
                "enum": ["AlbedoMap",
                            "EmissiveMap",
                            "NormalMap",
                            "OcclusionMap",
                            "RoughnessMap",
                            "MetalnessMap",
                            "ReflectivityMap",
                            "BumpMap",
                            "OpacityMap",
                            "DiffuseMap",
                            "SpecularMap",
                            "ShininessMap",
                            "MetallicRoughnessMap",
                            "SpecularGlossinessMap"]
            }
        }
    },
    "type" : "array",
    "description" : "List of materials to override",
    "items":
    {
        "type" : "object",
        "description" : "List of parameters to override",
        "properties":
        {
            "name": { "type" : "string"},
            "unlit": { "type" : "boolean" },
            "albedoColor": { "$ref": "#/definitions/colorOrAlpha" },
            "roughness": { "type": "number" },
            "metalness": { "type": "number" },
            "transparent": { "type" : "boolean" },
            "alphaClipEnabled": { "type" : "boolean" },
            "alphaClipThreshold": { "type": "number" },
            "useVertexColor": { "type" : "boolean" },
            "isDoubleSided": { "type" : "boolean" },
            "ignoreTextureMaps": { "$ref" : "#/definitions/listOfMaps" }
        },
        "required": ["name"],
        "additionalProperties" : false
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

* [Materiales de color](../../overview/features/color-materials.md)
* [Materiales de PBR](../../overview/features/pbr-materials.md)