---
title: Configuración de la conversión de modelos
description: Descripción de todos los parámetros de conversión de modelos
author: florianborn71
ms.author: flborn
ms.date: 03/06/2020
ms.topic: how-to
ms.openlocfilehash: b4881ee52b39539bfc29f62d7c6773da371a3ea5
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88067178"
---
# <a name="configure-the-model-conversion"></a>Configuración de la conversión de modelos

En este capítulo se documentan las opciones para la conversión de modelos.

## <a name="settings-file"></a>Archivo de configuración

Si se encuentra un archivo denominado `<modelName>.ConversionSettings.json` en el contenedor de entrada junto al modelo de entrada `<modelName>.<ext>`, se usará para proporcionar una configuración adicional para el proceso de conversión de modelos.
Por ejemplo, `box.ConversionSettings.json` se utilizaría al convertir `box.gltf`.

El contenido del archivo debe cumplir el esquema JSON siguiente:

```json
{
    "$schema" : "http://json-schema.org/schema#",
    "description" : "ARR ConversionSettings Schema",
    "type" : "object",
    "properties" :
    {
        "scaling" : { "type" : "number", "exclusiveMinimum" : 0, "default" : 1.0 },
        "recenterToOrigin" : { "type" : "boolean", "default" : false },
        "opaqueMaterialDefaultSidedness" : { "type" : "string", "enum" : [ "SingleSided", "DoubleSided" ], "default" : "DoubleSided" },
        "material-override" : { "type" : "string", "default" : "" },
        "gammaToLinearMaterial" : { "type" : "boolean", "default" : false },
        "gammaToLinearVertex" : { "type" : "boolean", "default" : false },
        "sceneGraphMode": { "type" : "string", "enum" : [ "none", "static", "dynamic" ], "default" : "dynamic" },
        "generateCollisionMesh" : { "type" : "boolean", "default" : true },
        "unlitMaterials" : { "type" : "boolean", "default" : false },
        "fbxAssumeMetallic" : { "type" : "boolean", "default" : true },
        "deduplicateMaterials" : { "type" : "boolean", "default" : true },
        "axis" : {
            "type" : "array",
            "items" : {
                "type" : "string",
                "enum" : ["default", "+x", "-x", "+y", "-y", "+z", "-z"]
            },
            "minItems": 3,
            "maxItems": 3
        },
        "metadataKeys": {
            "type": "array",
            "items": {
                "type": "string"
            }
        }
    },
    "additionalProperties" : false
}
```

Un archivo `box.ConversionSettings.json` de ejemplo podría ser:

```json
{
    "scaling" : 0.01,
    "recenterToOrigin" : true,
    "material-override" : "box_materials_override.json"
}
```

### <a name="geometry-parameters"></a>Parámetros de geometría

* `scaling`: este parámetro escala uniformemente un modelo. El escalado se puede usar para aumentar o reducir un modelo, por ejemplo, para mostrar un modelo de compilación en la parte superior de una tabla.
El escalado también es importante cuando un modelo se define en unidades distintas a los medidores, ya que el motor de representación espera medidores.
Por ejemplo, si un modelo se define en centímetros, la aplicación de una escala de 0,01 debe representar el modelo con el tamaño correcto.
Algunos formatos de datos de origen (por ejemplo, .fbx) proporcionan una sugerencia de escalado de unidades, en cuyo caso la conversión escala implícitamente el modelo a unidades de medidor. El escalado implícito proporcionado por el formato de origen se aplicará en la parte superior del parámetro de escalado.
El factor de escalado final se aplica a los vértices de geometría y las transformaciones locales de los nodos de gráfico de escena. El escalado para la transformación de la entidad raíz permanece sin modificar.

* `recenterToOrigin`: indica que un modelo debe convertirse para que su rectángulo de selección se centre en el origen.
Si el modelo de origen se desplaza lejos del origen, las incidencias de precisión del número de punto flotante pueden dar lugar a artefactos de representación.
Centrar el modelo puede ayudar en esta situación.

* `opaqueMaterialDefaultSidedness`: el motor de representación da por sentado que los materiales opacos son de doble cara.
Si esa suposición no es verdadera para un modelo determinado, este parámetro debe establecerse en "SingleSided". Para más información, consulte [:::no-loc text="single sided":::Representación en una sola cara](../../overview/features/single-sided-rendering.md).

### <a name="material-overrides"></a>Invalidaciones de materiales

* `material-override`: este parámetro permite que el procesamiento de materiales se [personalice durante la conversión](override-materials.md).

### <a name="material-de-duplication"></a>Desduplicación de materiales

* `deduplicateMaterials`: este parámetro habilita o deshabilita la desduplicación automática de materiales que comparten las mismas propiedades y texturas. La desduplicación se produce una vez procesados los reemplazos de material. Esto está habilitada de manera predeterminada.

### <a name="color-space-parameters"></a>Parámetros de espacio de colores

El motor de representación espera que los valores del color estén en el espacio lineal.
Si un modelo se define mediante el espacio gamma, estas opciones deben establecerse en true.

* `gammaToLinearMaterial`: conversión de colores del material del espacio gamma al espacio lineal
* `gammaToLinearVertex`: conversión:::no-loc text="vertex"::: de colores del vértice del espacio gamma al espacio lineal

> [!NOTE]
> En el caso de los archivos FBX, esta configuración se establece en `true` de forma predeterminada. En el caso del resto de los tipos de archivo, el valor predeterminado es `false`.

### <a name="scene-parameters"></a>Parámetros de escena

* `sceneGraphMode`: define cómo se convierte el gráfico de escena en el archivo de origen:
  * `dynamic` (predeterminado): todos los objetos del archivo se exponen como [entidades](../../concepts/entities.md) en la API y se pueden transformar de manera independiente. La jerarquía de nodos en tiempo de ejecución es idéntica a la estructura del archivo de origen.
  * `static`: todos los objetos se exponen en la API, pero no se pueden transformar de manera independiente.
  * `none`: el gráfico de escena se contrae en un objeto.

Cada modo tiene un rendimiento en tiempo de ejecución diferente. En el modo `dynamic`, el costo de rendimiento se escala linealmente con el número de [entidades](../../concepts/entities.md) del gráfico, incluso cuando no se mueve ninguna parte. Utilice el modo `dynamic` solo cuando sea necesario trasladar partes individualmente, por ejemplo, para una animación de "vista explosionada".

El modo de `static` exporta el gráfico de escena completo, pero las partes que contiene este gráfico tienen una transformación constante relativa a su parte raíz. El nodo raíz del objeto, sin embargo, puede seguir moviéndose, girándose o escalándose sin ningún costo de rendimiento importante. Además, las [consultas espaciales](../../overview/features/spatial-queries.md) devolverán partes individuales, pudiendo modificarse cada una a través de [invalidaciones de estado](../../overview/features/override-hierarchical-state.md). Con este modo, la sobrecarga en tiempo de ejecución por objeto es insignificante. Es ideal para escenas grandes en las que aún es necesaria una inspección por objeto, pero no cambia ninguna transformación por objeto.

El modo `none` tiene la sobrecarga en tiempo de ejecución más baja, así como tiempos de carga ligeramente mejores. La inspección o transformación de objetos individuales no es posible en este modo. Los casos de uso son, por ejemplo, modelos de fotogrametría que no tienen un gráfico de escena significativo en primer lugar.

> [!TIP]
> Muchas aplicaciones cargarán varios modelos. Debe optimizar los parámetros de conversión de cada modelo en función de cómo se vaya a utilizar. Por ejemplo, si desea mostrar el modelo de un automóvil para que el usuario lo desmonte e inspeccione con detalle, deberá convertirlo con el modo `dynamic`. Sin embargo, si además desea colocar el automóvil en un entorno de sala de exposiciones, ese modelo puede convertirse con `sceneGraphMode` establecido en `static` o incluso `none`.

### <a name="physics-parameters"></a>Parámetros de física

* `generateCollisionMesh`: si necesita compatibilidad con [consultas espaciales](../../overview/features/spatial-queries.md) en un modelo, debe habilitarse esta opción. En el peor de los casos, la creación de una malla de colisión puede duplicar el tiempo de conversión. Los modelos con mallas de colisión tardan más tiempo en cargarse y, al usar un gráfico de escena `dynamic`, también tienen una mayor sobrecarga de rendimiento en tiempo de ejecución. Para obtener un rendimiento óptimo general, debe deshabilitar esta opción en todos los modelos en los que no son necesarias las consultas espaciales.

### <a name="unlit-materials"></a>Materiales sin iluminación

* `unlitMaterials`: de forma predeterminada, la conversión preferirá crear [materiales de PBR](../../overview/features/pbr-materials.md). Esta opción indica al convertidor que trate todos los materiales como [materiales de color](../../overview/features/color-materials.md) en su lugar. Si tiene datos que ya incorporan iluminación, como, por ejemplo, los modelos creados a través de fotogrametría, esta opción le permite aplicar rápidamente la conversión correcta para todos los materiales, sin necesidad de [invalidar cada material](override-materials.md) de forma individual.

### <a name="converting-from-older-fbx-formats-with-a-phong-material-model"></a>Conversión a partir de formatos de FBX anteriores, con un modelo de material de Phong

* `fbxAssumeMetallic`: las versiones anteriores del formato de FBX definen sus materiales mediante un modelo de material de Phong. El proceso de conversión debe deducir cómo se asignan estos materiales al [modelo de PBR](../../overview/features/pbr-materials.md) del representador. Normalmente esto funciona bien, pero puede surgir una ambigüedad cuando un material no tiene texturas, valores especulares altos ni colores albedo no grises. En este caso, la conversión debe decidir entre priorizar los valores especulares altos, definir un material metálico altamente reflectante en el que el color albedo se disuelve o priorizar el color albedo, lo que define algo similar a un plástico de colores brillantes. De forma predeterminada, en el proceso de conversión se da por sentado que los valores altamente especulares implican un material metálico en los casos en los que se aplica ambigüedad. Este parámetro se puede establecer en `false` para cambiar al contrario.

### <a name="coordinate-system-overriding"></a>Invalidación del sistema de coordenadas

* `axis`: para invalidar los vectores de unidad del sistema de coordenadas. Los valores predeterminados son `["+x", "+y", "+z"]`. En teoría, el formato de FBX tiene un encabezado en el que se definen esos vectores y la conversión usa esa información para transformar la escena. El formato glTF también define un sistema de coordenadas fijo. En la práctica, algunos recursos tienen información incorrecta en su encabezado o se han guardado con una convención diferente del sistema de coordenadas. Esta opción permite invalidar el sistema de coordenadas que se va a compensar. Por ejemplo: `"axis" : ["+x", "+z", "-y"]` intercambiará el eje Z y el eje Y y mantendrá la mano del sistema de coordenadas invirtiendo la dirección del eje Y.

### <a name="node-meta-data"></a>Metadatos de nodo

* `metadataKeys`: permite especificar las claves de las propiedades de metadatos de nodo que desea conservar en el resultado de la conversión. Puede especificar claves exactas o claves de caracteres comodín. Las claves de caracteres comodín tienen el formato "ABC*" y coinciden con cualquier clave que empiece por "ABC". Los tipos de valor de metadatos admitidos son `bool`, `int`, `float` y `string`.

    En el caso de los archivos GLTF, estos datos proceden del [objeto extras en los nodos](https://github.com/KhronosGroup/glTF/tree/master/specification/2.0#nodeextras). En el caso de los archivos FBX, estos datos proceden de los datos de `Properties70` de `Model nodes`. Consulte la documentación de la herramienta de recursos 3D para obtener más detalles.

### <a name="no-loc-textvertex-format"></a>Formato de :::no-loc text="Vertex":::

El formato de :::no-loc text="vertex"::: se puede ajustar para una malla para cambiar la precisión del ahorro de memoria. Una superficie de memoria más baja permite cargar modelos más grandes o conseguir un mejor rendimiento. Sin embargo, en función de los datos, el formato incorrecto puede afectar significativamente a la calidad de la representación.

> [!CAUTION]
> El cambio del formato de :::no-loc text="vertex"::: debería ser el último recurso si los modelos ya no caben en la memoria o cuando se realiza una optimización para lograr el mejor rendimiento posible. Los cambios pueden presentar fácilmente artefactos de representación, ambos obvios y sutiles. A menos que sepa qué buscar, no debe cambiar el valor predeterminado.

Estos ajustes son posibles:

* Los flujos de datos específicos pueden incluirse o excluirse de forma explícita.
* La precisión de los flujos de datos puede reducirse para, a su vez, reducir la superficie de memoria.

La sección `vertex` siguiente del archivo `.json` es opcional. En el caso de cada una de las partes no especificadas explícitamente, el servicio de conversión recurre a su configuración predeterminada.

```json
{
    ...
    "vertex" : {
        "position"  : "32_32_32_FLOAT",
        "color0"    : "NONE",
        "color1"    : "NONE",
        "normal"    : "NONE",
        "tangent"   : "NONE",
        "binormal"  : "NONE",
        "texcoord0" : "32_32_FLOAT",
        "texcoord1" : "NONE"
    },
    ...
```

Al forzar a un componente a `NONE`, se garantiza que la malla de salida no tenga el flujo correspondiente.

#### <a name="component-formats-per-no-loc-textvertex-stream"></a>Formatos de componentes por flujo de :::no-loc text="vertex":::

Estos formatos están permitidos para los componentes correspondientes:

| Componente de :::no-loc text="Vertex"::: | Formatos admitidos (negrita = valor predeterminado) |
|:-----------------|:------------------|
|position| **32_32_32_FLOAT**, 16_16_16_16_FLOAT |
|color0| **8_8_8_8_UNSIGNED_NORMALIZED**, NONE |
|color1| 8_8_8_8_UNSIGNED_NORMALIZED, **NONE**|
|normal| **8_8_8_8_SIGNED_NORMALIZED**, 16_16_16_16_FLOAT, NONE |
|tangente| **8_8_8_8_SIGNED_NORMALIZED**, 16_16_16_16_FLOAT, NONE |
|binormal| **8_8_8_8_SIGNED_NORMALIZED**, 16_16_16_16_FLOAT, NONE |
|texcoord0| **32_32_FLOAT**, 16_16_FLOAT, NONE |
|texcoord1| **32_32_FLOAT**, 16_16_FLOAT, NONE |

#### <a name="supported-component-formats"></a>Formatos de componentes compatibles

Las superficies de memoria de los formatos son las siguientes:

| Formato | Descripción | Bytes por :::no-loc text="vertex"::: |
|:-------|:------------|:---------------|
|32_32_FLOAT|precisión del número de punto flotante completa de dos componentes|8
|16_16_FLOAT|precisión del número de punto flotante media de dos componentes|4
|32_32_32_FLOAT|precisión del número de punto flotante completa de tres componentes|12
|16_16_16_16_FLOAT|precisión del número de punto flotante media de cuatro componentes|8
|8_8_8_8_UNSIGNED_NORMALIZED|byte de cuatro componentes, normalizado en el intervalo `[0; 1]`|4
|8_8_8_8_SIGNED_NORMALIZED|byte de cuatro componentes, normalizado en el intervalo `[-1; 1]`|4

#### <a name="best-practices-for-component-format-changes"></a>Procedimientos recomendados para los cambios del formato de componentes

* `position`: es poco habitual que la precisión reducida sea suficiente. **16_16_16_16_FLOAT** presenta artefactos de cuantificación perceptibles, incluso para modelos pequeños.
* `normal`, `tangent`, `binormal`: normalmente, estos valores se cambian juntos. A menos que haya artefactos de iluminación perceptibles derivados de la cuantificación normal, no hay ninguna razón para aumentar su precisión. En algunos casos, sin embargo, estos componentes se pueden establecer en **NONE**:
  * `normal`, `tangent` y `binormal` solo son necesarios si se debe iluminar al menos un material del modelo. En ARR, este es el caso al usarse un [material de PBR](../../overview/features/pbr-materials.md) en el modelo en cualquier momento.
  * `tangent` y `binormal` solo son necesarios si alguno de los materiales de iluminación usa una textura de mapa normal.
* `texcoord0`, `texcoord1`: Las coordenadas de textura pueden usar una precisión reducida (**16_16_FLOAT**) si sus valores permanecen en el intervalo `[0; 1]` y si las texturas tratadas tienen un tamaño máximo de 2048 x 2048 píxeles. Si se superan esos límites, la calidad de la asignación de textura se verá afectada.

#### <a name="example"></a>Ejemplo

Suponga que tiene un modelo de fotogrametría, cuya iluminación está preparada en las texturas. Todo lo que se necesita para representar el modelo son las posiciones de :::no-loc text="vertex"::: y las coordenadas de textura.

De forma predeterminada, el convertidor debe suponer que es posible que desee usar materiales de PBR en un modelo en algún momento, por lo que generará datos relativos a `normal`, `tangent` y `binormal` automáticamente. Por lo tanto, el uso de memoria por vértice es `position` (12 bytes) + `texcoord0` (8 bytes) + `normal` (4 bytes) + `tangent` (4 bytes) + `binormal` (4 byte) = 32 bytes. Los modelos más grandes de este tipo pueden tener fácilmente muchos millones de :::no-loc text="vertices":::, lo que da como resultado modelos que pueden aceptar varios gigabytes de memoria. Estas grandes cantidades de datos afectarán al rendimiento y es posible que incluso se quede sin memoria.

Al saber que nunca va a necesitar iluminación dinámica en el modelo y que todas las coordenadas de textura están en el intervalo `[0; 1]`, puede establecer `normal`, `tangent` y `binormal` en `NONE` y `texcoord0` en precisión media (`16_16_FLOAT`), lo que da como resultado solo 16 bytes por :::no-loc text="vertex":::. La reducción de los datos de malla a la mitad permite cargar modelos más grandes y mejora el rendimiento potencialmente.

## <a name="memory-optimizations"></a>Optimizaciones de memoria

El consumo de memoria de contenido cargado puede convertirse en un cuello de botella en el sistema de representación. Si la carga de memoria llega a ser demasiado grande, puede poner en peligro el rendimiento de la representación o hacer que el modelo no se cargue por completo. En este párrafo se describen algunas estrategias importantes para reducir la superficie de memoria.

### <a name="instancing"></a>Instanciación

La instanciación es un concepto en el que se reutilizan mallas para las partes con transformaciones espaciales distintas, en lugar de que cada una de las partes haga referencia a su propia geometría única. La instanciación tiene un impacto considerable en la superficie de memoria.
Los casos de uso de ejemplo para la instanciación son los tornillos de un modelo de motor o las sillas de un modelo arquitectónico.

> [!NOTE]
> La instanciación puede mejorar considerablemente el consumo de memoria (y, por tanto, los tiempos de carga), pero las mejoras en el rendimiento de la representación son insignificantes.

El servicio de conversión respeta la instanciación si las partes están marcadas en consecuencia en el archivo de origen. Sin embargo, la conversión no realiza un análisis profundo adicional de los datos de la malla para identificar las partes que se pueden volver a usar. Por consiguiente, la herramienta de creación de contenido y su canalización de exportación son los criterios decisivos para la correcta configuración de la instanciación.

Una manera sencilla de comprobar si la información de la instanciación se conserva durante la conversión es echar un vistazo a las [estadísticas de salida](get-information.md#example-info-file), en concreto al miembro `numMeshPartsInstanced`. Si el valor de `numMeshPartsInstanced` es mayor que cero, indica que las mallas se comparten entre las instancias.

#### <a name="example-instancing-setup-in-3ds-max"></a>Ejemplo: Configuración de la instanciación en 3ds Max

[Autodesk 3ds Max](https://www.autodesk.de/products/3ds-max) tiene distintos modos de clonación de objetos denominados **`Copy`** , **`Instance`** y **`Reference`** que se comportan de manera diferente en lo que respecta a la instanciación en el archivo `.fbx` exportado.

![Clonación en 3ds Max](./media/3dsmax-clone-object.png)

* **`Copy`** : En este modo, se clona la malla, por lo que no se usa la instanciación (`numMeshPartsInstanced` = 0).
* **`Instance`** : Los dos objetos comparten la misma malla, por lo que se usa la instanciación (`numMeshPartsInstanced` = 1).
* **`Reference`** : Se pueden aplicar distintos modificadores a las geometrías, por lo que el exportador elige un enfoque conservador y no utiliza la instanciación (`numMeshPartsInstanced` = 0).


### <a name="depth-based-composition-mode"></a>Modo de composición basado en profundidad

Si la memoria es un problema, configure el representador con el [modo de composición basado en profundidad](../../concepts/rendering-modes.md#depthbasedcomposition-mode). En este modo, la carga de GPU se distribuye entre varias GPU.

### <a name="decrease-vertex-size"></a>Reducir el tamaño del vértice

Como se describe en la sección de [procedimientos recomendados para realizar cambios en el formato de los componentes](configure-model-conversion.md#best-practices-for-component-format-changes), el ajuste del formato del vértice puede reducir la superficie de memoria. Sin embargo, esta opción debería ser el último recurso.

### <a name="texture-sizes"></a>Tamaños de texturas

Dependiendo del tipo de escenario, la cantidad de datos de textura puede ser mayor que la memoria usada para los datos de la malla. Los modelos de fotogrametría son candidatos.
La configuración de la conversión no proporciona una manera de reducir verticalmente las texturas de forma automática. Si fuera necesario, el escalado de textura debe realizarse como un paso del procesamiento previo del cliente. Sin embargo, el paso de conversión elige un [formato de compresión de textura](https://docs.microsoft.com/windows/win32/direct3d11/texture-block-compression-in-direct3d-11) adecuado:

* `BC1` para texturas de color opacas
* `BC7` para las texturas de color de origen con canal alfa

Como el formato `BC7` tiene el doble de superficie de memoria, en comparación con `BC1`, es importante asegurarse de que las texturas de entrada no proporcionan un canal alfa innecesariamente.

## <a name="typical-use-cases"></a>Casos de uso típicos

Buscar una buena configuración de importación para un caso de uso determinado puede ser un proceso tedioso. Por otro lado, la configuración de conversión puede afectar de forma significativa al rendimiento en tiempo de ejecución.

Existen ciertas clases de casos de uso que son aptas para optimizaciones específicas. A continuación se proporcionan algunos ejemplos.

### <a name="use-case-architectural-visualization--large-outdoor-maps"></a>Caso de uso: Visualización de arquitectura/grandes mapas exteriores

* Estos tipos de escenas tienden a ser estáticos, lo que significa que no necesitan partes movibles. En consecuencia, el modo `sceneGraphMode` se puede establecer en `static` o incluso `none`, que mejora el rendimiento en tiempo de ejecución. Con el modo `static`, el nodo raíz de la escena puede seguir moviéndose, girándose y escalándose, por ejemplo, para cambiar de forma dinámica entre la escala de 1:1 (para la vista de primera persona) y una vista principal de tabla.

* Si necesita mover partes, normalmente también significa que necesita compatibilidad con proyecciones de rayo u otras [consultas espaciales](../../overview/features/spatial-queries.md), de modo que pueda elegir esas partes en primer lugar. Por otro lado, si no tiene previsto mover nada, lo más probable es que tampoco lo necesite para participar en consultas espaciales y, por lo tanto, pueda desactivar la marca `generateCollisionMesh`. Este modificador tiene un impacto significativo en los tiempos de conversión, los tiempos de carga y también los costos de actualización por fotograma en tiempo de ejecución.

* Si la aplicación no usa [planos de corte](../../overview/features/cut-planes.md), la marca `opaqueMaterialDefaultSidedness` debe desactivarse. La ganancia de rendimiento suele situarse entre un 20 y un 30 %. Se pueden seguir usando los planos de corte, pero no habrá caras posteriores al fijarse en las partes internas de los objetos, lo que parece contradictorio. Para más información, consulte [:::no-loc text="single sided":::Representación en una sola cara](../../overview/features/single-sided-rendering.md).

### <a name="use-case-photogrammetry-models"></a>Caso de uso: Modelos de fotogrametría

Al representar los modelos de fotogrametría, no suele ser necesario un gráfico de escena, por lo que puede establecer el `sceneGraphMode` en `none`. Sin embargo, dado que esos modelos raramente contienen un gráfico de escena complejo para comenzar, el impacto de esta opción debería ser insignificante.

Dado que la iluminación ya recurre a las texturas, no se necesita iluminación dinámica. Por lo tanto:

* Establezca la marca `unlitMaterials` en `true` para convertir todos los materiales en [materiales de color](../../overview/features/color-materials.md) sin iluminación.
* Quite los datos innecesarios del formato de vértice. Consulte el [ejemplo](#example) anterior.

### <a name="use-case-visualization-of-compact-machines-etc"></a>Caso de uso: Visualización de máquinas compactas, etc.

En estos casos de uso, los modelos suelen ser muy detallados en un volumen pequeño. El representador está muy optimizado para controlar estos casos de manera adecuada. Sin embargo, la mayoría de las optimizaciones mencionadas en el caso de uso anterior no se aplican aquí:

* Las partes individuales deben poder seleccionarse y moverse, por lo que el `sceneGraphMode` debe dejarse en `dynamic`.
* Las conversiones de rayo suelen ser una parte integral de la aplicación, por lo que se deben generar mallas de colisión.
* Los planos de corte se ven mejor con la marca `opaqueMaterialDefaultSidedness` habilitada.

## <a name="deprecated-features"></a>Características en desuso

Se sigue admitiendo la configuración mediante el nombre de archivo `conversionSettings.json` no específico del modelo, pero en desuso.
En su lugar, use el nombre de archivo `<modelName>.ConversionSettings.json` específico del modelo.

## <a name="next-steps"></a>Pasos siguientes

* [Conversión de modelos](model-conversion.md)
* [Materiales de color](../../overview/features/color-materials.md)
* [Materiales de PBR](../../overview/features/pbr-materials.md)
* [Reemplazo de materiales durante la conversión de modelos](override-materials.md)
