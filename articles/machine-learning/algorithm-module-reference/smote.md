---
title: SMOTE
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar el módulo SMOTE en Azure Machine Learning para aumentar el número de ejemplos de baja incidencia en un conjunto de datos mediante el sobremuestreo.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 10/16/2019
ms.openlocfilehash: 501f3e8946023d28d67a33fbbfca661afbc6306d
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90898260"
---
# <a name="smote"></a>SMOTE

En este artículo se describe cómo usar el módulo SMOTE en el diseñador de Azure Machine Learning para aumentar el número de casos subrepresentados de un conjunto de datos que se usa para el aprendizaje automático. SMOTE es una mejor manera para aumentar el número de casos poco frecuentes en lugar de simplemente duplicar los casos existentes.  

El módulo SMOTE se conecta a un conjunto de datos *con desequilibrios*. Hay muchas razones por las que un conjunto de datos puede tener desequilibrios. Por ejemplo, la categoría que es su destino podría ser poco frecuente en la población o los datos podrían ser difíciles de recopilar. Normalmente, SMOTE se utiliza cuando la *clase* que se desea analizar está subrepresentada. 
  
El módulo devuelve un conjunto de datos que contiene los ejemplos originales. También devuelve varios ejemplos de minorías sintéticas, en función del porcentaje que se especifique.  
  
## <a name="more-about-smote"></a>Más información sobre SMOTE

SMOTE es una técnica estadística de sobremuestreo de minorías sintéticas para aumentar el número de casos de un conjunto de datos de forma equilibrada. El módulo funciona cuando genera nuevas instancias a partir de casos minoritarios existentes que se proporcionan como entrada. Esta implementación de SMOTE *no* cambia el número de casos de mayoría.

Las instancias nuevas no son meras copias de los casos minoritarios existentes. En su lugar, el algoritmo toma muestras del *espacio de características* de cada clase de destino y de sus vecinos más próximos. Luego, el algoritmo genera nuevos ejemplos que combinan las características del caso que nos ocupa con características de sus vecinos. Este enfoque aumenta las características disponibles para cada clase y hace que las muestras sean más generales.
  
SMOTE toma todo el conjunto de datos como una entrada, pero solo aumenta el porcentaje de los casos minoritarios. Por ejemplo, suponga que tiene un conjunto de datos con desequilibrios en el que solo el 1 % de los casos tiene el valor de destino A (la clase minoritaria), mientras que el 99 % restante tienen el valor B. Para aumentar al doble el porcentaje de los casos minoritarios, es preciso especificar **200**como**porcentaje de SMOTE** en las propiedades del módulo.  
  
## <a name="examples"></a>Ejemplos  

Se recomienda intentar usar SMOTE con un conjunto de datos pequeño para ver cómo funciona. El ejemplo siguiente usa el conjunto de datos sobre donación sangre disponible en el diseñador de Azure Machine Learning.
  
Si agrega el conjunto de datos a una canalización y selecciona **Visualizar** en la salida del conjunto de datos, puede ver que, de las 748 filas o casos del conjunto de datos, hay 570 (76 %) de la clase 0 y 178 (24 %) de la clase 1. Aunque este resultados no tiene tantos desequilibrios, la clase 1 representa a las personas que donaron sangre, por lo que estas filas contienen el *espacio de características* que desea modelar.
 
Para aumentar el número de casos, puede usar múltiplos de 100 para establecer el valor del **porcentaje de SMOTE**, tal como se muestra a continuación:

||Clase 0|Clase 1|total|  
|-|-------------|-------------|-----------|  
|Conjunto de datos original<br /><br /> (equivalente a **porcentaje SMOTE** = **0**)|570<br /><br /> 76%|178<br /><br /> 24%|748|  
|**Porcentaje SMOTE** = **100**|570<br /><br /> 62%|356<br /><br /> 38%|926|  
|**Porcentaje SMOTE** = **200**|570<br /><br /> 52%|534<br /><br /> 48%|1104|  
|**Porcentaje SMOTE** = **300**|570<br /><br /> 44%|712<br /><br /> 56%|1\.282|  
  
> [!WARNING]
> El aumento del número de casos mediante SMOTE no garantiza la producción de modelos más precisos. Pruebe la canalización con diferentes porcentajes, conjuntos de características y números de vecinos más próximos para ver de qué forma la adición de casos afecta al modelo.  
  
## <a name="how-to-configure-smote"></a>Cómo configurar SMOTE
  
1.  Agregue el módulo SMOTE a la canalización. Puede encontrarlo en los **módulos de Transformación de datos**, en la categoría **Manipulación**.

2. Conecte el conjunto de datos que desee potenciar. Si desea especificar el espacio de características para compilar los nuevos casos, ya sea mediante el uso exclusivo de columnas concretas o mediante la exclusión de algunas, use el módulo [Select Columns in Dataset](select-columns-in-dataset.md) (Seleccionar columnas del conjunto de datos). A continuación, puede aislar las columnas que desea usar antes de utilizar SMOTE.
  
    De lo contrario, la creación de nuevos casos en SMOTE se basa en *todas* las columnas que se proporcionan como entradas. Al menos una de las columnas de características es numérica.
  
3.  Asegúrese que la columna que contiene la etiqueta o la clase de destino está seleccionada. SMOTE solo acepta etiquetas binarias.
  
4.  El módulo SMOTE identifica automáticamente la clase minoritaria en la columna de la etiqueta y luego obtiene todos los ejemplos de la clase minoritaria. Todas las columnas no pueden tener valores NaN.
  
5.  En la opción **SMOTE percentage** (Porcentaje de SMOTE), escriba un número entero que indique el porcentaje destino de los casos minoritarios en el conjunto de datos de salida. Por ejemplo:  
  
    - Escriba **0**. El módulo SMOTE devuelve exactamente el mismo conjunto de datos que proporcionó como entrada. No agrega ningún caso minoritario nuevo. En este conjunto de datos, la proporción de la clase no ha cambiado.  
  
    - Escriba **100**. El módulo SMOTE genera nuevos casos minoritarios. Agrega el mismo número de casos minoritarios que había en el conjunto de datos original. Dado que SMOTE no aumenta el número de casos mayoritarios, la proporción de casos de cada clase ha cambiado.  
  
    - Escriba **200**. El módulo duplica el porcentaje de casos minoritarios en comparación con el conjunto de datos original. Esto *no significa* que haya el doble de casos minoritarios que antes. En su lugar, el tamaño del conjunto de datos aumenta de tal forma que el número de casos mayoritarios no cambia. El número de casos minoritarios aumenta hasta que llega al valor porcentual deseado.  
  
    > [!NOTE]
    > Use solo múltiplos de 100 para el porcentaje SMOTE.

6.  Use la opción **Number of nearest neighbors** (Número de vecinos más próximos) para determinar el tamaño del espacio de características que utiliza el algoritmo SMOTE al compilar nuevos casos. Un vecino más próximo es una fila de datos (un caso) que es similar a un caso de destino. La distancia entre dos casos cualesquiera se mide con la combinación de los vectores ponderados de todas las características.  
  
    + Al aumentar el número de vecinos más próximos, se obtienen características en más casos.
    + Al mantener bajo el número de vecinos más próximos, se usan características que son más similares a las de la muestra original.  
  
7. Escriba un valor en el cuadro de texto **Random seed** (Valor de inicialización aleatorio) si desea garantizar los mismos resultados en las ejecuciones de la misma canalización, con los mismos datos. De lo contrario, el módulo genera un valor de inicialización aleatorio basada en los valores del reloj del procesador cuando se implementa la canalización. La generación de un valor de inicialización aleatorio puede producir resultados ligeramente diferentes en las distintas ejecuciones.

8. Envíe la canalización.  
  
   La salida del módulo es un conjunto de datos que contiene las filas originales, más un número de filas agregadas con casos minoritarios.  

## <a name="technical-notes"></a>Notas técnicas

+ Cuando publique un modelo que utilice el módulo **SMOTE**, elimine **SMOTE** del canal de predicción antes de que se publique como un servicio web. El motivo de dicha eliminación es que SMOTE está diseñado para mejorar un modelo durante el entrenamiento, no para la puntuación. Es posible que se produzca un error si una canalización predictiva publicada contiene el módulo SMOTE.

+ A menudo se pueden obtener mejores resultados si se borran los valores que faltan o se aplican otras transformaciones para corregir los datos antes de aplicar SMOTE. 

+ Se han realizado investigaciones para saber si SMOTE es efectivo en datos de gran dimensión o dispersos, como los que se utilizan en la clasificación de textos o en los conjuntos de datos de genómica. Este documento tiene un buen resumen de los efectos y de la validez teórica de la aplicación de SMOTE en estos casos: [Blagus y Lusa: SMOTE para datos de clase desequilibrada en su clase de gran dimensión](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-14-106).

+ Si SMOTE no es eficaz en su conjunto de datos, estos son otros enfoques que pueden resultarle útiles:
  + Métodos de sobremuestreo de casos minoritarios o de inframuestreo de casos mayoritarios.
  + Técnicas de ensamblaje que ayuda al estudiante directamente mediante el uso de la agrupación en clústeres, la agregación o la potenciación adaptable.


## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 

