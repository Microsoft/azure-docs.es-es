---
title: 'Conversión de palabra en vector: Referencia del módulo'
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar tres modelos de Word2Vec proporcionados para extraer un vocabulario y sus incrustaciones de palabras correspondientes de una corpus de texto.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 05/19/2020
ms.openlocfilehash: 1c2aa9023a7081387d38b9f7c6cfe8323300ad6e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90898601"
---
# <a name="convert-word-to-vector-module"></a>Módulo Convert Word to Vector

En este artículo se describe cómo usar el módulo Convert Word to Vector (Convertir palabra en vector) en el diseñador de Azure Machine Learning para realizar las siguientes tareas:

- Aplicar varios modelos de Word2Vec (Word2Vec, FastText, modelo previamente entrenado de GloVe) en el corpus de texto que haya especificado como entrada.
- Generar un vocabulario con inserciones de palabras.

En este módulo se usa la biblioteca Gensim. Para más información sobre Gensim, consulte su [sitio web oficial](https://radimrehurek.com/gensim/apiref.html) que incluye tutoriales y una explicación de los algoritmos.

### <a name="more-about-converting-words-to-vectors"></a>Más información acerca de la conversión de palabras en vectores

En términos generales, la vectorización de palabras, o lo que es lo mismo, convertir palabras en vectores, es un procedimiento de procesamiento de lenguaje natural (NLP). El proceso utiliza modelos de lenguaje o técnicas para asignar palabras al espacio vectorial, es decir, para representar cada palabra por un vector de números reales. Mientras tanto, permite que las palabras con significados similares tengan representaciones similares.

Las inserciones de palabras se pueden usar como entrada inicial para las tareas de nivel inferior de NLP, como la clasificación de texto y el análisis de opiniones.

Entre las distintas tecnologías de inserción de palabras, en este módulo hemos implementado tres métodos ampliamente utilizados. Dos son modelos de aprendizaje en línea: Word2Vec y FastText. El otro es un modelo entrenado previamente, glove-wiki-gigaword-100. 

Los modelos de aprendizaje en línea están entrenados en los datos de entrada. Los modelos entrenados previamente están entrenados sin conexión en un corpus de texto más grande (por ejemplo, Wikipedia, Google News) que normalmente contiene un número aproximado de 100 mil millones palabras. La inserción de palabras permanece constante durante la vectorización de palabras. Los modelos de palabras entrenados previamente proporcionan ventajas como un tiempo de entrenamiento reducido, mejores vectores de palabra codificados y un rendimiento general mejorado.

Aquí tiene alguna información sobre los métodos:

+ Word2Vec es una de las técnicas más populares para aprender inserciones de palabras mediante una red neuronal superficial. La teoría se trata en este documento, disponible como descarga en PDF: [Estimación eficaz de las representaciones de palabras en el Espacio vectorial de Tomas Mikolov y otros](https://arxiv.org/pdf/1301.3781.pdf). La implementación de este módulo se basa en la [biblioteca de Gensim para Word2Vec](https://radimrehurek.com/gensim/models/word2vec.html).

+ La teoría del modelo FastText se trata en este documento, disponible como descarga en PDF: [Enriquecimiento de vectores de palabra con información de subpalabra de Piotr Bojanowski y otros](https://arxiv.org/pdf/1607.04606.pdf). La implementación de este módulo se basa en la [biblioteca de Gensim para FastText](https://radimrehurek.com/gensim/models/fasttext.html).

+ El modelo GloVe entrenado previamente es glove-wiki-gigaword-100. Se trata de una colección de vectores entrenados previamente, basados en un corpus de texto de Wikipedia, que contiene 5600 millones de tokens y 400 000 palabras de vocabulario sin distinción de mayúsculas o minúsculas. Hay disponible una descarga en PDF: [GloVe: Vectores globales para la representación de palabras](https://nlp.stanford.edu/pubs/glove.pdf).

## <a name="how-to-configure-convert-word-to-vector"></a>Procedimiento para configurar Convertir palabra en vector

Este módulo requiere un conjunto de datos que contenga una columna de texto. Es mejor que el texto sea preprocesado.

1. Agregue el módulo **Convertir palabra en vector** a una canalización.

2. Como entrada para el módulo, proporcione un conjunto de datos que contenga una o varias columnas de texto.

3. Para **Target column** (Columna de destino), elija una sola columna que contengan el texto para procesar.

    Dado que este módulo crea un vocabulario a partir de texto, el contenido de las distintas columnas difiere, lo que conduce a un contenido de vocabulario diferente. Por eso el módulo solo acepta una columna de destino.

4. Para **Word2Vec strategy** (Estrategia Word2Vec), elija entre el **modelo inglés entrenado previamente de GloVe**, **Gensim Word2Vec** y **Gensim FastText**.

5. Si **Word2Vec strategy** es **Gensim Word2Vec** o **Gensim FastText**:

    + Para **Word2Vec Training Algorithm** (Algoritmo de entrenamiento), elija entre **Skip_gram** y **CBOW**. La diferencia se presenta en el [artículo original (PDF)](https://arxiv.org/pdf/1301.3781.pdf).

        El método predeterminado es **Skip_gram**.

    + Para **Length of word embedding** (Longitud de la inserción de palabras), especifique la dimensionalidad de los vectores de palabra. Este valor corresponde al parámetro `size` de Gensim.

        El tamaño predeterminado de inserción es 100.

    + Para **Context window size** (Tamaño de ventana de contexto) especifique la distancia máxima entre la palabra que se va a predecir y la palabra actual. Este valor corresponde al parámetro `window` de Gensim.

        El tamaño predeterminado de la ventana es 5.

    + Para **Number of epochs** (Número de tiempos base), especifique el número de tiempos base (iteraciones) en el corpus. Este valor corresponde al parámetro `iter` de Gensim.

        El número de tiempos base predeterminado es 5.

6. Para **Maximum vocabulary size** (Tamaño máximo de vocabulario), especifique el número máximo de palabras en el vocabulario generado.

    Si hay más palabras únicas, elimine las poco frecuentes.

    El tamaño predeterminado del vocabulario es 10 000.

7. Para **Minimum word count** (Recuento mínimo de palabras), proporcione un recuento mínimo de palabras. El módulo prescindirá de todas las palabras que tengan una frecuencia inferior a este valor.

    El valor predeterminado es 5.

8. Envíe la canalización.

## <a name="examples"></a>Ejemplos

El módulo tiene una salida:

+ **Vocabulario con incrustaciones**: Contiene el vocabulario generado, junto con la inserción de cada palabra. Una dimensión ocupa una columna.

En el ejemplo siguiente se muestra cómo funciona el módulo Convert Word to Vector (Convertir palabra en vector). Aplica este módulo con la configuración predeterminada al conjunto de datos preprocesado Wikipedia SP 500 que se proporciona en Azure Machine Learning.

### <a name="source-dataset"></a>Conjunto de datos de origen

El conjunto de datos contiene una columna de categoría, junto con el texto completo capturado de Wikipedia. En esta tabla se muestran solo algunos ejemplos representativos.

|Texto|
|----------|
|NASDAQ 100 componente s p 500 componente fundación fundador ubicación ciudad campus de Apple 1 bucle infinito calle bucle infinito Cupertino California Cupertino California ubicación país Estados Unidos...|
|br nasdaq 100 nasdaq 100 componente br s p 500 s p 500 componente industry ordenador software fundación br fundador Charles Geschke br John Warnock ubicación sistemas de Adobe...|
|s p 500 s p 500 componente industria automoción industria automoción predecesor general motores corporación 1908 2009 sucesor...|
|s p 500 s p 500 componente industria conglomerado empresa conglomerado fundación fundador ubicación ciudad Fairfield Connecticut Fairfield Connecticut ubicación área país EE. UU...|
|br s p 500 s p 500 componente fundación 1903 fundador William s Harley br Arthur Davidson Harley Davidson fundador Arthur Davidson br Walter Davidson br William a Davidson ubicación...|

### <a name="output-vocabulary-with-embeddings"></a>Vocabulario de salida con incrustaciones

La tabla siguiente contiene la salida de este módulo que toma como entrada el conjunto de datos Wikipedia SP 500. La columna situada más a la izquierda indica el vocabulario. Su vector de inserción se representa mediante valores de las columnas restantes de la misma fila.

|Vocabulario|Incrustación de dim 0|Incrustación de dim 1|Incrustación de dim 2|Incrustación de dim 3|Incrustación de dim 4|Incrustación de dim 5|…|Incrustación de dim 99|
|-------------|-------------|-------------|-------------|-------------|-------------|-------------|-------------|-------------|
|NASDAQ|-0.375865|0.609234|0.812797|-0.002236|0.319071|-0.591986|…|0.364276
|component|0.081302|0.40001|0.121803|0.108181|0.043651|-0.091452|…|0.636587
|s|-0.34355|-0.037092|-0.012167|0.151542|0.601019|0.084501|…|0.149419
|p|-0.133407|0.073244|0.170396|0.326706|0.213463|-0.700355|…|0.530901
fundación|-0.166819|0.10883|-0.07933|-0.073753|0.262137|0.045725|…|0.27487
fundador|0\.297408|0.493067|0.316709|-0.031651|0.455416|-0.284208|…|0.22798
ubicación|-0.375213|0.461229|0.310698|0.213465|0.200092|0.314288|…|0.14228
city|-0.460828|0.505516|-0.074294|-0.00639|0.116545|0.494368|…|-0.2403
Apple|0.05779|0.672657|0.597267|-0.898889|0.099901|0.11833|…|0.4636
campus|-0.281835|0.29312|0.106966|-0.031385|0.100777|-0.061452|…|0.05978
infinito|-0.263074|0.245753|0.07058|-0.164666|0.162857|0\.027345|…|-0.0525
bucle|-0.391421|0.52366|0.141503|-0.105423|0.084503|-0.018424|…|-0.0521

En este ejemplo usamos **Gensim Word2Vec** para **estrategia de Word2Vec**, y el **Algoritmo de entrenamiento** es **Skip-gram**. **La longitud de la inserción de palabras** es 100, por lo que tenemos 100 columnas de inserción.

## <a name="technical-notes"></a>Notas técnicas

Esta sección contiene sugerencias y respuestas a las preguntas más frecuentes.

+ Diferencia entre el modelo de entrenamiento en línea y el modelo entrenado previamente:

    En este módulo de conversión de palabra en vector, hemos proporcionado tres estrategias diferentes: dos modelos de aprendizaje en línea y un modelo entrenado previamente. Los modelos de aprendizaje en línea utilizan su conjunto de datos de entrada como datos de entrenamiento, y generan vectores de palabras y vocabulario durante el entrenamiento. El modelo entrenado previamente ya está entrenado por un corpus de texto mucho mayor, como texto proveniente de Wikipedia o Twitter. El modelo entrenado previamente es realmente una colección de pares de palabra/inserción.  

    Si el modelo entrenado previamente de GloVe se elige como la estrategia de vectorización de palabras, resume un vocabulario del conjunto de datos de entrada y genera un vector de inserción para cada palabra del modelo previamente entrenado. Sin aprendizaje en línea, el uso de un modelo entrenado previamente puede ahorrar tiempo de entrenamiento. Tiene un mejor rendimiento, especialmente cuando el tamaño del conjunto de datos de entrada es relativamente pequeño.

+ Tamaño de inserción:

    En general, para lograr un buen rendimiento la longitud de la inserción de palabras se establece en unas pocas centenas (por ejemplo, 100, 200, 300). El motivo es que un pequeño tamaño de inserción significa un pequeño espacio vectorial, lo que puede provocar colisiones de inserción de palabras.  

    En el caso de los modelos entrenados previamente, la longitud de las inserciones de palabras es fija. En esta implementación, el tamaño de la inserción de glove-wiki-gigaword-100 es 100.


## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 

Para obtener una lista de los errores específicos de los módulos del diseñador, consulte [Códigos de error de Machine Learning](designer-error-codes.md).
