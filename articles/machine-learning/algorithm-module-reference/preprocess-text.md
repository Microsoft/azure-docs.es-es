---
title: 'Preprocess Text (Preprocesamiento de texto): referencia para los módulos'
titleSuffix: Azure Machine Learning
description: Aprenda a usar el módulo Preprocesamiento de texto en Azure Machine Learning para limpiar y simplificar texto.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 09/01/2019
ms.openlocfilehash: 4112d26d6a21ac800e20bb67ce24a35ca9d09a13
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905232"
---
# <a name="preprocess-text"></a>preprocesamiento de texto

En este artículo se describe un módulo del diseñador de Azure Machine Learning.

Use el módulo **Preprocess Text** para limpiar y simplificar texto. Admite estas operaciones de procesamiento de texto habituales:

* Eliminación de palabras irrelevantes
* Uso de expresiones regulares para buscar y reemplazar cadenas de destino específicas
* Lematización: esta operación convierte varias palabras relacionadas en una única forma canónica
* Normalización de mayúsculas
* Eliminación de ciertas clases de caracteres, como números, caracteres especiales y secuencias de caracteres repetidos, como "aaaa"
* Identificación y eliminación de correos electrónicos y direcciones URL

El módulo **Preprocess Text** solo admite actualmente texto en inglés.

## <a name="configure-text-preprocessing"></a>Configuración de Preprocess Text  

1.  Agregue el módulo **Preprocesamiento de texto** al experimento en Azure Machine Learning. Puede encontrar este módulo en **Text Analytics**.

1. Conecte un conjunto de datos que incluya al menos una columna que contenga texto.

1. Seleccione el idioma en la lista desplegable **Language** (Idioma).

1. **Text column to clean** (Columna de texto que se va a limpiar): seleccione la columna que quiere preprocesar.

1. **Remove stop words** (Eliminar palabras irrelevantes): seleccione esta opción si desea aplicar una lista de palabras irrelevantes predefinida a la columna de texto. 

    Las listas de palabras irrelevantes dependen del idioma y son personalizables.

1. **Lemmatization** (Lematización): seleccione esta opción si desea que las palabras se representen en su forma canónica. Esta opción es útil para reducir el número de apariciones únicas de otros tokens de texto similares.

    El proceso de lematización depende en gran medida del idioma.

1. **Detect sentences** (Detectar frases): seleccione esta opción si desea que el módulo inserte una marca de límite de frase al realizar el análisis.

    Este módulo usa una serie de tres caracteres de barra vertical `|||` para representar el terminador de frase.

1. Realice operaciones opcionales de búsqueda y reemplazo mediante expresiones regulares.

    * **Custom regular expression** (Personalizar expresión regular): defina el texto que está buscando.
    * **Custom replacement string** (Personalizar cadena de sustitución): defina un único valor de sustitución.

1. **Normalize case to lowercase** (Normalización de mayúsculas en minúsculas): seleccione esta opción si desea convertir los caracteres ASCII en mayúsculas a sus correspondientes formas en minúsculas.

    Si no se normalizan los caracteres, la misma palabra en mayúsculas y minúsculas se considerarán como dos palabras diferentes.

1. También puede eliminar los siguientes tipos de caracteres o secuencias de caracteres del texto de salida procesado:

    * **Remove numbers** (Quitar números): seleccione esta opción para quitar todos los caracteres numéricos del idioma especificado. Los números de identificación dependen del dominio y dependen del idioma. Si los caracteres numéricos forman parte integral de una palabra conocida, es posible que no se quite el número.
    
    * **Remove special characters** (Quitar caracteres especiales): utilice esta opción para quitar los caracteres especiales no alfanuméricos.
    
    * **Remove duplicate characters** (Quitar caracteres duplicados): seleccione esta opción para quitar los caracteres adicionales de las secuencias que se repiten más de dos veces. Por ejemplo, una secuencia como "aaaaa" se reduciría a "aa".
    
    * **Remove email addresses** (Quitar direcciones de correo electrónico): seleccione esta opción para quitar cualquier secuencia del formato `<string>@<string>`.  
    * **Remove URLs** (Quitar direcciones URL): seleccione esta opción para quitar cualquier secuencia que incluya los siguientes prefijos de direcciones URL: `http`, `https`, `ftp`, `www`.
    
1. **Expand verb contractions** (Expandir contracciones verbales): esta opción solo se aplica a los idiomas que usan contracciones verbales. Actualmente, solo se aplica al inglés. 

    Por ejemplo, si selecciona esta opción, podría sustituir la expresión *"wouldn't stay there"* por *"would not stay there"* .

1. **Normalize backslashes to slashes** (Normalizar barras diagonales inversas en barras diagonales: seleccione esta opción para asignar todas las instancias de `\\` a `/`.

1. **Split tokens on special characters** (Dividir tokens en caracteres especiales): seleccione esta opción si desea dividir las palabras en caracteres como `&`, `-`, etc. Esta opción también puede reducir los caracteres especiales cuando se repite más de dos veces. 

    Por ejemplo, la cadena `MS---WORD` se dividiría en tres tokens `MS`, `-` y `WORD`.

1. Envíe la canalización.

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 