---
title: ¿Qué son el entrenamiento y los modelos? - Custom Translator
titleSuffix: Azure Cognitive Services
description: 'Un modelo es el sistema que proporciona una traducción para pares de idiomas específicos. El resultado de un entrenamiento correcto es un modelo. Cuando se entrena un modelo, se requieren tres conjuntos de datos mutuamente exclusivos: el conjunto de datos de aprendizaje, el de ajuste y el de pruebas.'
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: conceptual
ms.date: 08/17/2020
ms.author: swmachan
ms.openlocfilehash: d8713d20503dc788175a408f5ec652505090396e
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88510834"
---
# <a name="what-are-trainings-and-models"></a>¿Qué son los entrenamientos y los modelos?

Un modelo es el sistema que proporciona una traducción para pares de idiomas específicos.
El resultado de un entrenamiento correcto es un modelo. Cuando se entrena un modelo, se requieren tres tipos de documentos mutuamente exclusivos: aprendizaje, ajuste y pruebas. También se puede proporcionar el tipo de documento de diccionario. Vea [Alineación de frases](https://docs.microsoft.com/azure/cognitive-services/translator/custom-translator/sentence-alignment#suggested-minimum-number-of-sentences).

Si solo se proporcionan datos de aprendizaje al poner en cola un entrenamiento, Custom Translator reunirá automáticamente los datos de pruebas y de ajuste. Usará un subconjunto aleatorio de frases de los documentos de aprendizaje y excluirá estas frases de los propios datos de aprendizaje.

## <a name="training-document-type-for-custom-translator"></a>Tipo de documento de aprendizaje para Custom Translator

Custom Translator usa los documentos incluidos en el conjunto de aprendizaje como base para generar el modelo. Durante la ejecución del entrenamiento, se alinean (o emparejan) las oraciones que están presentes en estos documentos. Puede tomarse libertades durante la creación del conjunto de documentos de aprendizaje. Puede incluir documentos que crea que son de importancia tangencial en un modelo. Vuelva a incluirlos en otro entrenamiento para ver el efecto que esto tiene en la puntuación [BLEU (suplente de evaluación bilingüe)](what-is-bleu-score.md). Siempre que el conjunto de ajuste y el de pruebas sean constantes, no dude en experimentar con la creación del conjunto de aprendizaje. Este enfoque es una forma eficaz de modificar la calidad de su sistema de traducción.

Puede ejecutar varios entrenamientos dentro de un proyecto y comparar las [puntuaciones BLEU ](what-is-bleu-score.md) en todas las ejecuciones del entrenamiento. Cuando ejecute varios entrenamientos para compararlos, asegúrese de usar los mismos datos de prueba o de ajuste cada vez. También asegúrese de inspeccionar los resultados manualmente en la pestaña de [pruebas](how-to-view-system-test-results.md).

## <a name="tuning-document-type-for-custom-translator"></a>Tipo de documento de ajuste para Custom Translator

Custom Translator utiliza los documentos paralelos incluidos en este conjunto para optimizar el sistema de traducción y así obtener resultados óptimos.

Los datos de ajuste se usan durante el entrenamiento para ajustar todos los parámetros y las ponderaciones del sistema de traducción según los valores óptimos. Elija sus datos de ajuste cuidadosamente: deben ser representativos del contenido de los documentos que va a traducir en el futuro. Los datos de ajuste tienen una importante influencia en la calidad de las traducciones generadas. El ajuste permite al sistema de traducción proporcionar traducciones más cercanas a las muestras proporcionadas en los datos de ajuste. No se necesitan más que 2500 frases para los datos de ajuste. Para una calidad de traducción óptima, se recomienda que seleccione manualmente al conjunto de ajuste mediante la selección del grupo de oraciones más representativas.

Durante la creación del conjunto de ajuste, seleccione las oraciones que tengan una longitud representativa de las oraciones que espera traducir en el futuro. También debe elegir las oraciones que contengan palabras y frases que pretende traducir, con la distribución aproximada que espera encontrar en futuras traducciones. En la práctica, las oraciones con una longitud de entre 7 y 10 palabras generarán los mejores resultados, dado que estas oraciones contienen suficiente contexto para mostrar inflexiones y proporcionan una longitud de frase que es significativa sin ser demasiado compleja.

Una buena descripción del tipo de oraciones que debe usar en el conjunto de ajuste es la prosa: oraciones reales y fluidas. No celdas de una tabla, ni poemas, ni listas de cosas, ni solo puntuación o números: idioma común.

Si selecciona manualmente los datos de ajuste, en ellos no debe repetirse ninguna de las oraciones incluidas en los datos de aprendizaje o de pruebas. Los datos de ajuste tienen un impacto importante en la calidad de las traducciones: elija atentamente las oraciones.

Si no está seguro de qué opciones elegir para los datos de ajuste, simplemente, seleccione los datos de aprendizaje y permita que Custom Translator seleccione los datos de ajuste automáticamente. Cuando Custom Translator elige automáticamente los datos de ajuste, usa un subconjunto aleatorio de oraciones tomadas de los documentos bilingües de aprendizaje y excluye a las oraciones que están en el material de aprendizaje.

## <a name="testing-dataset-for-custom-translator"></a>Conjunto de datos de pruebas para Custom Translator

Los documentos paralelos que se incluyen en el conjunto de pruebas se utilizan para calcular la puntuación BLEU (suplente de evaluación bilingüe). Esta puntuación indica la calidad de su sistema de traducción. Realmente, esta puntuación le indica qué tanto se acercan las traducciones del sistema de traducción creado con este entrenamiento a las oraciones de referencia incluidas en el conjunto de datos de pruebas.

La puntuación BLEU es una medida de la delta entre la traducción automática y la traducción de referencia. Su valor varía entre 0 y 100. Una puntuación de 0 indica que en la traducción no aparece ni una sola palabra de la referencia. Una puntuación de 100 indica que la traducción automática coincide exactamente con la referencia: las mismas palabras están en exactamente la misma posición. La puntuación que recibe es la puntuación BLEU promedio de todas las oraciones en los datos de pruebas.

Los datos de prueba deben incluir documentos paralelos en los que las oraciones del idioma de destino sean las traducciones más deseables de las oraciones del idioma de origen correspondientes para el par de origen y destino. Es posible que quiera usar los mismos criterios que utilizó para crear los datos de ajuste. Sin embargo, los datos de prueba no influyen en la calidad del sistema de traducción. Se utiliza exclusivamente para generar la puntuación BLEU.

No se necesitan más de 2500 oraciones para los datos de pruebas. Cuando el sistema elige automáticamente al conjunto de pruebas, usa un subconjunto aleatorio de oraciones tomadas de los documentos bilingües de aprendizaje y excluye a las oraciones que están en el material de aprendizaje.

Puede ver las traducciones personalizadas del conjunto de pruebas y compararlas con las traducciones proporcionadas en el conjunto de pruebas. Para ello, vaya a la pestaña de pruebas dentro de un modelo.
