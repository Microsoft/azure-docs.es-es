---
title: Asignación de materiales para formatos de modelos
description: Describe la conversión predeterminada de los formatos de origen del modelo al material de PBR
author: jakrams
ms.author: jakras
ms.date: 02/11/2020
ms.topic: reference
ms.openlocfilehash: 8313243bf680ea1a1d63f2719b647149a04935a9
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88893108"
---
# <a name="material-mapping-for-model-formats"></a>Asignación de materiales para formatos de modelos

Cuando un recurso de origen se [convierte en modelo](../how-tos/conversion/model-conversion.md), el convertidor crea [materiales](../concepts/materials.md) para cada [malla](../concepts/meshes.md). La forma en que los materiales se crean se puede [modificar](../how-tos/conversion/override-materials.md). Sin embargo, de forma predeterminada, la conversión creará [materiales de PBR](../overview/features/pbr-materials.md). Dado que todos los formatos de archivo de origen, como FBX, usan sus propias convenciones para definir materiales, esas convenciones deben asignarse a los parámetros de material de PBR de Azure Remote Rendering. 

En este artículo se enumeran las asignaciones exactas que se usan para convertir materiales de recursos de origen a materiales de entorno de ejecución.

## <a name="gltf"></a>glTF

Casi todo lo que se encuentra en la especificación glTF 2.0 es compatible con Azure Remote Rendering, excepto *EmissiveFactor* y *EmissiveTexture*.

En la tabla siguiente se muestra la asignación:

| glTF | Azure Remote Rendering |
|:-------------------|:--------------------------|
|   baseColorFactor   |   albedoColor              |
|   baseColorTexture  |   albedoMap                |
|   metallicFactor    |   metalness                |
|   metallicTexture   |   metalnessMap             |
|   roughnessFactor   |   roughness                |
|   roughnessTexture  |   roughnessMap             |
|   occlusionFactor   |   occlusion                |
|   occlusionTexture  |   occlusionMap             |
|   normalTexture     |   normalMap                |
|   alphaCutoff       |   alphaClipThreshold       |
|   alphaMode.OPAQUE  |   alphaClipEnabled = false, isTransparent = false |
|   alphaMode.MASK    |   alphaClipEnabled = true, isTransparent = false  |
|   alphaMode.BLEND   |   isTransparent = true     |
|   doubleSided       |   isDoubleSided            |
|   emissiveFactor    |   -                        |
|   emissiveTexture   |   -                        |

Cada textura de glTF puede tener un valor `texCoord`, que también se admite en los materiales de Azure Remote Rendering.

### <a name="embedded-textures"></a>Texturas incrustadas

Se admiten las texturas incrustadas en archivos *\*.bin* o *\*.glb*.

### <a name="supported-gltf-extension"></a>Extensión de glTF compatible

Además del conjunto de características básicas, Azure Remote Rendering admite las siguientes extensiones de glTF:

* [MSFT_packing_occlusionRoughnessMetallic](https://github.com/KhronosGroup/glTF/blob/master/extensions/2.0/Vendor/MSFT_packing_occlusionRoughnessMetallic/README.md)
* [KHR_materials_unlit](https://github.com/KhronosGroup/glTF/blob/master/extensions/2.0/Khronos/KHR_materials_unlit/README.md): corresponde a [materiales de color](../overview/features/color-materials.md). Como materiales *emisores*, se recomienda usar esta extensión.
* [KHR_materials_pbrSpecularGlossiness](https://github.com/KhronosGroup/glTF/blob/master/extensions/2.0/Khronos/KHR_materials_pbrSpecularGlossiness/README.md): en lugar de texturas de rugosidad metálica, puede proporcionar texturas de brillo, especular o difusas. La implementación de Azure Remote Rendering sigue directamente las fórmulas de conversión de la extensión.

## <a name="fbx"></a>FBX

El formato FBX es de código cerrado y los materiales de FBX no son compatibles con los materiales de PBR en general. FBX usa una descripción compleja de las superficies con muchos parámetros y propiedades únicos, y **no todos se usan en la canalización de Azure Remote Rendering**.

> [!IMPORTANT]
> La canalización de conversión del modelo de Azure Remote Rendering solo admite **FBX 2011 y versiones posteriores**.

El formato FBX define un enfoque conservador para los materiales, solo hay dos tipos en la especificación oficial de FBX:

* *Lambert*: no se usa habitualmente desde hace ya bastante tiempo, pero se sigue admitiendo mediante la conversión a Phong en el momento de la conversión.
* *Phong*: casi todos los materiales y la mayoría de las herramientas de contenido usan este tipo.

El modelo Phong es más preciso y se usa *solo* como modelo para los materiales de FBX. Más abajo se hace referencia a este modelo como *material de FBX*.

> Maya usa dos extensiones personalizadas para FBX mediante la definición de propiedades personalizadas para los tipos PBR y Stingray de un material. Estos detalles no se incluyen en la especificación de FBX, por lo que actualmente no se admite en Azure Remote Rendering.

Los materiales de FBX usan el concepto Difusión-Especular-NivelEspecular, por lo que, para convertir una textura difusa en un mapa de albedo, tenemos que calcular los demás parámetros para restarlos de la difusión.

> Todos los colores y las texturas de FBX están en el espacio sRGB (también conocido como espacio Gamma), pero Azure Remote Rendering funciona con el espacio lineal durante la visualización y al final del fotograma vuelve a convertir todo al espacio sRGB. La canalización de recursos de Azure Remote Rendering convierte todo al espacio lineal para enviarlo en forma de datos preparados al representador.

En esta tabla se muestra cómo se asignan las texturas de materiales de FBX a materiales de Azure Remote Rendering. Algunas no se usan directamente, sino en combinación con otras que participan en las fórmulas (por ejemplo, la textura difusa):

| FBX | Azure Remote Rendering |
|:-----|:----|
| AmbientColor | Mapa de oclusión   |
| DiffuseColor | *Se usa para Albedo, Metalness* |
| TransparentColor | *Se usa para el canal alfa de Albedo* |
| TransparencyFactor | *Se usa para el canal alfa de Albedo* |
| Opacidad | *Se usa para el canal alfa de Albedo* |
| SpecularColor | *Se usa para Albedo, Metalness, Roughness* |
| SpecularFactor| *Se usa para Albedo, Metalness, Roughness* |
| ShininessExponent | *Se usa para Albedo, Metalness, Roughness* |
| NormalMap | NormalMap |
| Bump | *Se convierte a NormalMap* |
| EmissiveColor | - |
| EmissiveFactor | - |
| ReflectionColor | - |
| DisplacementColor | - |

La asignación anterior es la parte más compleja de la conversión de materiales, debido a la cantidad de suposiciones que se deben realizar. A continuación se indican estas suposiciones.

Algunas de las definiciones que se usan a continuación:

* `Specular` =  `SpecularColor` * `SpecularFactor`
* `SpecularIntensity` = `Specular`.Rojo ∗ 0,2125 +  `Specular`.Verde ∗ 0,7154 + `Specular`.Azul ∗ 0,0721
* `DiffuseBrightness` = 0,299 * `Diffuse`.Rojo<sup>2</sup> + 0,587 * `Diffuse`.Verde<sup>2</sup> + 0,114 * `Diffuse`.Azul<sup>2</sup>
* `SpecularBrightness` = 0,299 * `Specular`.Rojo<sup>2</sup> + 0,587 * `Specular`.Verde<sup>2</sup> + 0,114 * `Specular`.Azul<sup>2</sup>
* `SpecularStrength` = max(`Specular`.Rojo, `Specular`.Verde, `Specular`.Azul)

La fórmula SpecularIntensity se obtiene de [aquí](https://en.wikipedia.org/wiki/Luma_(video)).
La fórmula de brillo se describe en esta [especificación](http://www.itu.int/dms_pubrec/itu-r/rec/bt/R-REC-BT.601-7-201103-I!!PDF-E.pdf).

### <a name="roughness"></a>Roughness

`Roughness` se calcula a partir de `Specular` y `ShininessExponent` mediante [esta fórmula](https://www.cs.cornell.edu/~srm/publications/EGSR07-btdf.pdf). La fórmula es una aproximación de la rugosidad del exponente especular Phong:

```cpp
Roughness = sqrt(2 / (ShininessExponent * SpecularIntensity + 2))
```

### <a name="metalness"></a>Metalness

`Metalness` se calcula a partir de `Diffuse` y `Specular` mediante esta [fórmula de especificación glTF](https://github.com/bghgary/glTF/blob/gh-pages/convert-between-workflows-bjs/js/babylon.pbrUtilities.js).

La idea es que se solucione la ecuación: Ax<sup>2</sup> + Bx + C = 0.
Básicamente, las superficies dieléctricas reflejan aproximadamente el 4 % de la luz de forma especular y el resto se difumina. Las superficies metálicas no reflejan ninguna luz de forma difusa, sino todas de forma especular.
Esta fórmula tiene algunas desventajas, ya que no hay ninguna manera de distinguir entre las superficies metálicas satinadas y las de plástico brillante. Suponemos la mayor parte del tiempo que la superficie tiene propiedades metálicas y, por tanto, las superficies de plástico y de caucho brillante podrían no tener la apariencia esperada.

```cpp
dielectricSpecularReflectance = 0.04
oneMinusSpecularStrength = 1 - SpecularStrength

A = dielectricSpecularReflectance
B = (DiffuseBrightness * (oneMinusSpecularStrength / (1 - A)) + SpecularBrightness) - 2 * A
C = A - SpecularBrightness
squareRoot = sqrt(max(0.0, B * B - 4 * A * C))
value = (-B + squareRoot) / (2 * A)
Metalness = clamp(value, 0.0, 1.0);
```

### <a name="albedo"></a>Albedo

`Albedo` se calcula a partir de `Diffuse`, `Specular` y `Metalness`.

Tal y como se describe en la sección Metalness, las superficies dieléctricas reflejan aproximadamente el 4 % de la luz.  
Aquí, la idea es interpolar linealmente entre los colores `Dielectric` y `Metal` con el valor `Metalness` como un factor. Si Metalness es `0.0`, en función del valor de Specular, será un color oscuro (si el valor es alto) o, si no, se cambiará el valor de Diffuse (si no hay ningún valor de Specular). Si el valor es grande, el color difuso desaparecerá en favor del color especular.

```cpp
dielectricSpecularReflectance = 0.04
oneMinusSpecularStrength = 1 - SpecularStrength

dielectricColor = diffuseColor * (oneMinusSpecularStrength / (1.0f - dielectricSpecularReflectance) / max(1e-4, 1.0 - metalness))
metalColor = (Specular - dielectricSpecularReflectance * (1.0 - metalness)) * (1.0 / max(1e-4, metalness))
albedoRawColor = lerpColors(dielectricColor, metalColor, metalness * metalness)
AlbedoRGB = clamp(albedoRawColor, 0.0, 1.0);
```

`AlbedoRGB` se ha calculado con la fórmula anterior, pero el canal alfa requiere cálculos adicionales. El formato FBX es impreciso en cuanto a la transparencia y tiene muchas maneras de definirla. Las distintas herramientas de contenido usan métodos diferentes. La idea es unificarlos en una fórmula. Sin embargo, hace que algunos recursos se muestren incorrectamente como transparentes, si no se crean de la forma habitual.

Se calcula a partir de `TransparentColor`, `TransparencyFactor` `Opacity`:

Si se define `Opacity`, úselo directamente: `AlbedoAlpha` = `Opacity` de lo contrario,  
si se define `TransparencyColor`, entonces `AlbedoAlpha` = 1,0 - ((`TransparentColor`.Rojo + `TransparentColor`.Verde + `TransparentColor`.Blue) / 3,0) de lo contrario  
si `TransparencyFactor`, entonces `AlbedoAlpha` = 1,0 - `TransparencyFactor`

El color de `Albedo` final tiene cuatro canales, que combina el valor `AlbedoRGB` con el de `AlbedoAlpha`.

### <a name="summary"></a>Resumen

Para resumir aquí, `Albedo` estará muy cerca del valor `Diffuse` original, si `Specular` está cerca de cero. De lo contrario, la superficie parecerá metálica y perderá el color difuso. La superficie tendrá un aspecto más pulido y reflectante si `ShininessExponent` es lo suficientemente grande y `Specular` es brillante. De lo contrario, la superficie será más rugosa y apenas reflejará el entorno.

### <a name="known-issues"></a>Problemas conocidos

* La fórmula actual no funciona bien para la geometría de color simple. Si `Specular` es suficientemente brillante, todas las geometrías se convierten en superficies metálicas reflectantes sin ningún color. La solución alternativa aquí es reducir `Specular` al 30 % del original o usar la configuración de conversión [fbxAssumeMetallic](../how-tos/conversion/configure-model-conversion.md#converting-from-older-fbx-formats-with-a-phong-material-model).
* Los materiales de PBR se agregaron recientemente a las herramientas de creación de contenido `Maya` y `3DS Max`. Usan propiedades de caja negra personalizadas definidas por el usuario para pasarlos a FBX. Azure Remote Rendering no lee estas propiedades adicionales porque no están documentadas y el formato es de código cerrado.

## <a name="next-steps"></a>Pasos siguientes

* [Conversión de modelos](../how-tos/conversion/model-conversion.md)
* [Reemplazo de materiales durante la conversión de modelos](../how-tos/conversion/override-materials.md)
