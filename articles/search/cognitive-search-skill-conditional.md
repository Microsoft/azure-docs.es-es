---
title: Habilidades de búsqueda cognitiva condicional (Azure Search) | Microsoft Docs
description: La aptitud condicional permite el filtrado, crear valores predeterminados y combinar los valores.
services: search
manager: pablocas
author: luiscabrer
ms.service: search
ms.devlang: NA
ms.workload: search
ms.topic: conceptual
ms.date: 05/01/2019
ms.author: luisca
ms.openlocfilehash: 149b701d4a1700787656448e2bdd0d92d2a93844
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65791513"
---
#   <a name="conditional-skill"></a>Aptitudes condicional

El *aptitud condicional* habilita escenarios de Azure Search que requieren una operación booleana para determinar los datos que se va a asignar a una salida. Estos escenarios incluyen el filtrado, asignar un valor predeterminado y combinar datos según una condición.

El siguiente pseudocódigo muestra lo que lleva a cabo la aptitud condicional:

```
if (condition) 
    { output = whenTrue } 
else 
    { output = whenFalse } 
```

> [!NOTE]
> Esta habilidad no está enlazada a una API de Azure Cognitive Services y no se cobra de uso. Sin embargo, todavía debe [adjuntar un recurso de Cognitive Services](cognitive-search-attach-cognitive-services.md) para invalidar la opción de recurso "Gratis" que tiene un límite en un número pequeño de enriquecimientos al día.

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Util.ConditionalSkill


## <a name="evaluated-fields"></a>Campos evaluados

Esta habilidad es especial porque sus entradas son campos evaluados.

Los elementos siguientes son los valores válidos de una expresión:

-   Las rutas de acceso de anotación (rutas de acceso en las expresiones deben delimitarse con "$(" and ")")
 <br/>
    Ejemplos:
    ```
        "= $(/document)"
        "= $(/document/content)"
    ```

-  Literales (cadenas, números, true, false, null) <br/>
    Ejemplos:
    ```
       "= 'this is a string'"   // string (note the single quotation marks)
       "= 34"                   // number
       "= true"                 // Boolean
       "= null"                 // null value
    ```

-  Expresiones que utilizan operadores de comparación (==,! =, > =, >, < =, <) <br/>
    Ejemplos:
    ```
        "= $(/document/language) == 'en'"
        "= $(/document/sentiment) >= 0.5"
    ```

-   Las expresiones que usan los operadores booleanos (& &, ||,!, ^) <br/>
    Ejemplos:
    ```
        "= $(/document/language) == 'en' && $(/document/sentiment) > 0.5"
        "= !true"
    ```

-   Expresiones que utilizan operadores numéricos (+, -, \*, /, %) <br/>
    Ejemplos: 
    ```
        "= $(/document/sentiment) + 0.5"         // addition
        "= $(/document/totalValue) * 1.10"       // multiplication
        "= $(/document/lengthInMeters) / 0.3049" // division
    ```

Dado que la aptitud condicional es compatible con la evaluación, puede usarlo en escenarios de transformación de menores. Por ejemplo, vea [definición habilidades 4](#transformation-example).

## <a name="skill-inputs"></a>Entradas de la aptitud
Las entradas distinguen mayúsculas de minúsculas.

| Entrada   | DESCRIPCIÓN |
|-------------|-------------|
| condición   | Esta entrada es un [evalúa campo](#evaluated-fields) que representa la condición para evaluar. Esta condición debe evaluarse como un valor booleano (*true* o *false*).   <br/>  Ejemplos: <br/> "= true" <br/> "= $(/document/language) == 'fr'" <br/> "= $(/documentos/páginas/ \* /Language) == $(/ documentos/expectedLanguage)" <br/> |
| whenTrue    | Esta entrada es un [evalúa campo](#evaluated-fields) que representa el valor devuelto si la condición se evalúa como *true*. Se deben devolver las cadenas constantes entre comillas simples ('y'). <br/>Valores de ejemplo: <br/> "="contrato""<br/>"= $(/document/contractType)" <br/> "= $(/documentos/entidades/\*)" <br/> |
| whenFalse   | Esta entrada es un [evalúa campo](#evaluated-fields) que representa el valor devuelto si la condición se evalúa como *false*. <br/>Valores de ejemplo: <br/> "="contrato""<br/>"= $(/document/contractType)" <br/> "= $(/documentos/entidades/\*)" <br/>

## <a name="skill-outputs"></a>Salidas de la aptitud
Hay una única salida se denomina simplemente "salida". Devuelve el valor *whenFalse* si la condición es false o *whenTrue* si la condición es true.

## <a name="examples"></a>Ejemplos

### <a name="sample-skill-definition-1-filter-documents-to-return-only-french-documents"></a>Definición de la habilidad de ejemplo 1: Filtrar los documentos para devolver únicamente los documentos francés

La salida siguiente devuelve una matriz de las oraciones ("/ documentos/frenchSentences") si el idioma del documento es francés. Si el idioma no es francés, el valor se establece en *null*.

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document",
    "inputs": [
        { "name": "condition", "source": "= $(/document/language) == 'fr'" },
        { "name": "whenTrue", "source": "/document/sentences" },
        { "name": "whenFalse", "source": "= null" }
    ],
    "outputs": [ { "name": "output", "targetName": "frenchSentences" } ]
}
```
Si se usa "/ documentos/frenchSentences" como el *contexto* de otra habilidad, dicha aptitud se ejecuta solo si "/ documentos/frenchSentences" no está establecido en *null*.


### <a name="sample-skill-definition-2-set-a-default-value-for-a-value-that-doesnt-exist"></a>Definición de la habilidad de ejemplo 2: Establecer un valor predeterminado para un valor que no existe

La salida siguiente crea una anotación ("/ documentos/languageWithDefault") que se establece en el lenguaje del documento o a "es" Si no se establece el idioma.

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document",
    "inputs": [
        { "name": "condition", "source": "= $(/document/language) == null" },
        { "name": "whenTrue", "source": "= 'es'" },
        { "name": "whenFalse", "source": "= $(/document/language)" }
    ],
    "outputs": [ { "name": "output", "targetName": "languageWithDefault" } ]
}
```

### <a name="sample-skill-definition-3-merge-values-from-two-fields-into-one"></a>Definición de la habilidad de ejemplo 3: Combinar valores de dos campos en uno

En este ejemplo, tienen algunas oraciones un *frenchSentiment* propiedad. Cada vez que el *frenchSentiment* propiedad es null, queremos usar el *englishSentiment* valor. Asignamos a la salida a un miembro que se denomina *opiniones* ("/ documentos/opiniones / * / opinión").

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document/sentences/*",
    "inputs": [
        { "name": "condition", "source": "= $(/document/sentences/*/frenchSentiment) == null" },
        { "name": "whenTrue", "source": "/document/sentences/*/englishSentiment" },
        { "name": "whenFalse", "source": "/document/sentences/*/frenchSentiment" }
    ],
    "outputs": [ { "name": "output", "targetName": "sentiment" } ]
}
```

## <a name="transformation-example"></a>Ejemplo de transformación
### <a name="sample-skill-definition-4-data-transformation-on-a-single-field"></a>Definición de la habilidad de ejemplo 4: Transformación de datos en un solo campo

En este ejemplo, se recibe un *opiniones* que está entre 0 y 1. Queremos transformar para estar entre -1 y 1. Podemos usar la aptitud condicional para realizar esta transformación secundaria.

En este ejemplo, no usamos el aspecto de la aptitud condicional porque la condición es siempre *true*.

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document/sentences/*",
    "inputs": [
        { "name": "condition", "source": "= true" },
        { "name": "whenTrue", "source": "= $(/document/sentences/*/sentiment) * 2 - 1" },
        { "name": "whenFalse", "source": "= 0" }
    ],
    "outputs": [ { "name": "output", "targetName": "normalizedSentiment" } ]
}
```

## <a name="special-considerations"></a>Consideraciones especiales
Algunos parámetros se evalúan, por lo que deberá ser especialmente cuidadoso seguir el patrón documentado. Las expresiones deben comenzar con un signo igual. Una ruta de acceso debe estar delimitado por "$(" and ")". Asegúrese de que poner las cadenas entre comillas simples. Que permite que el evaluador de distinguir entre cadenas y rutas de acceso y los operadores. Además, asegúrese de colocar el espacio en blanco alrededor de los operadores (por ejemplo, un "*" en una ruta de acceso, significa algo distinto a multiply).


## <a name="next-steps"></a>Pasos siguientes

+ [Aptitudes predefinidas](cognitive-search-predefined-skills.md)
+ [Definición de un conjunto de aptitudes](cognitive-search-defining-skillset.md)
