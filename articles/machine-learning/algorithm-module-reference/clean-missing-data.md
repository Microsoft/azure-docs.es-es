---
title: 'Limpiar datos que faltan: referencia para los módulos'
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar el módulo Limpiar datos que faltan en Azure Machine Learning para quitar, reemplazar o deducir los valores que faltan.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 02/11/2020
ms.openlocfilehash: 5b7943b2026d640ae7e5d119e165bd752ae2fe7f
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90898823"
---
# <a name="clean-missing-data-module"></a>Módulo Limpiar datos que faltan

En este artículo se describe un módulo del diseñador de Azure Machine Learning.

Utilice este módulo para quitar, reemplazar o deducir los valores que faltan. 

Los científicos de datos a menudo comprueban los datos para ver si faltan valores y, a continuación, realizan varias operaciones para corregir los datos o insertar nuevos valores. El objetivo de estas operaciones de limpieza es evitar los problemas causados por falta de datos, que pueden surgir al entrenar un modelo. 

Este módulo admite varios tipos de operaciones para "limpiar" los valores que faltan, entre los que se incluyen:

+ Reemplazar los valores que faltan por un marcador de posición, una media u otro valor
+ Eliminar completamente las filas y columnas que tienen valores que faltan
+ Deducir valores en función de métodos estadísticos


El uso de este módulo no cambia el conjunto de datos de origen. En su lugar, crea un nuevo conjunto de datos en el área de trabajo que puede usar en el flujo de trabajo posterior. También puede guardar el conjunto de datos nuevo y limpio para reutilizarlo.

Este módulo también genera una definición de la transformación usada para limpiar los valores que faltan. Puede volver a usar esta transformación en otros conjuntos de datos que tengan el mismo esquema, mediante el módulo [Aplicar transformación](./apply-transformation.md).  

## <a name="how-to-use-clean-missing-data"></a>Procedimiento para usar Limpiar datos que faltan

Este módulo permite definir una operación de limpieza. También puede guardar la operación de limpieza para aplicarla más adelante a nuevos datos. Vea las secciones siguientes para obtener una descripción de cómo crear y guardar un proceso de limpieza: 
 
+ [Para reemplazar los valores que faltan](#replace-missing-values)
  
+ [Para aplicar una transformación de limpieza a nuevos datos](#apply-a-saved-cleaning-operation-to-new-data)
 
> [!IMPORTANT]
> El método de limpieza que use para controlar los valores que faltan puede afectar drásticamente a los resultados. Se recomienda que experimente con diferentes métodos. Tenga en cuenta tanto la justificación para el uso de un método concreto como la calidad de los resultados.

### <a name="replace-missing-values"></a>Reemplazar los valores que faltan  

Cada vez que se aplica el módulo [Limpiar datos que faltan](./clean-missing-data.md) a un conjunto de datos, se aplica la misma operación de limpieza a todas las columnas que seleccione. Por lo tanto, si tiene que limpiar distintas columnas mediante diferentes métodos, utilice instancias independientes del módulo.

1.  Agregue el módulo [Limpiar los datos que faltan](./clean-missing-data.md) a la canalización y conecte el conjunto de datos al que le faltan valores.  
  
2.  Para **Columns to be cleaned** (Columnas que se deben limpiar), elija las columnas que contienen los valores que faltan que desee cambiar. Puede elegir varias columnas, pero debe usar el mismo método de reemplazo en todas las columnas seleccionadas. Por lo tanto, normalmente deberá limpiar las columnas de cadena y las columnas numéricas por separado.

    Por ejemplo, para comprobar si hay valores que faltan en todas las columnas numéricas:

    1. Seleccione el módulo **Limpiar los datos que faltan** y haga clic en **Editar columna** en el panel derecho del módulo.

    3. Para **Incluir**, seleccione **Tipos de columna** en la lista desplegable y, a continuación, seleccione **Numérico**. 
  
    Cualquier método de limpieza o de reemplazo que elija debe ser aplicable a **todas** las columnas de la selección. Si los datos de cualquier columna no son compatibles con la operación especificada, el módulo devuelve un error y detiene la canalización.
  
3.  Para **Minimum missing value ratio** (Índice mínimo de valores que faltan), especifique el número mínimo de valores que faltan necesarios para que realizar la operación.  
  
    Use esta opción en combinación con **Maximum missing value ratio** (Índice máximo de valores que faltan) para definir las condiciones en las que se realiza una operación de limpieza en el conjunto de datos. Si hay demasiadas o insuficientes filas que tienen valores que faltan, no se puede realizar la operación. 
  
    El número que escriba representa el **índice** de valores que faltan para todos los valores de la columna. De forma predeterminada, la propiedad **Minimum missing value ratio** (Índice mínimo de valores que faltan) está establecida en 0. Esto significa que los valores que faltan se limpian incluso si hay solo un valor que falta. 

    > [!WARNING]
    > Esta condición debe cumplirse para cada columna a fin de que se aplique la operación especificada. Por ejemplo, supongamos que ha seleccionado tres columnas y, a continuación, establece el índice mínimo de valores que faltan en ,2 (20 %), pero en realidad, solo una columna tiene el 20 % de valores que faltan. En este caso, la operación de limpieza se aplicaría solo a la columna con más del 20 % de valores que faltan. Por lo tanto, las demás columnas no cambiarían.
    > 
    > Si tiene alguna duda sobre si han cambiado los valores que faltan, seleccione la opción **Generate missing value indicator column** (Generar columna de indicador de valores que faltan). Se anexa una columna al conjunto de datos para indicar si cada la columna cumple los criterios especificados para los intervalos mínimos y máximo o no.  
  
4. Para **Maximum missing value ratio** (Índice máximo de valores que faltan), especifique el número máximo de valores que faltan necesarios para que realizar la operación.   
  
    Por ejemplo, si desea realizar la sustitución de valores que faltan solo si el 30 % o menos de las filas contienen valores que faltan, pero dejar los valores tal cual si más del 30 % de las filas tienen valores que faltan.  
  
    El número se define como el índice de valores que faltan para todos los valores de la columna. De forma predeterminada, la propiedad **Maximum missing value ratio** (Índice máximo de valores que faltan) está establecida en 1. Esto significa que los valores que faltan se limpian incluso si falta el 100 % de los valores de la columna.  
  
   
  
5. Para **Cleaning Mode** (Modo de limpieza), seleccione una de las siguientes opciones para reemplazar o quitar los valores que faltan:  
  
  
    + **Custom substitution value** (Valor de sustitución personalizado): Utilice esta opción para especificar un valor de marcador de posición (por ejemplo, un valor 0 o NA) que se aplica a todos los valores que faltan. El valor que especifique como reemplazo debe ser compatible con el tipo de datos de la columna.
  
    + **Replace with mean** (Reemplazar por media): Calcula la media de la columna y la utiliza como valor de reemplazo para cada valor que falta en la columna.  
  
        Solo se aplica a las columnas que tienen tipos de datos entero, doble o booleano.  
  
    + **Replace with median** (Reemplazar por mediana): Calcula el valor de mediana de la columna y lo utiliza como valor de reemplazo para cada valor que falta en la columna.  
  
        Solo se aplica a las columnas que tienen tipos de datos entero o doble. 
  
    + **Replace with mode** (Reemplazar por modo): Calcula el modo de la columna y lo utiliza como valor de reemplazo para cada valor que falta en la columna.  
  
        Se aplica a las columnas que tienen tipos de datos entero, doble, booleano o categorical. 
  
    + **Remove entire row** (Quitar toda la fila): Quita completamente cualquier fila del conjunto de datos que tiene uno o más valores que faltan. Es útil si el valor que falta puede considerarse que falta de forma aleatoria.  
  
    + **Remove entire column** (Quitar toda la columna): Quita completamente cualquier columna del conjunto de datos que tiene uno o más valores que faltan.  
  
    
  
6. La opción **Replacement value** está disponible si ha seleccionado la opción **Custom substitution value** (Valor de sustitución personalizado). Escriba un nuevo valor que se usará como el valor de reemplazo para todos los valores que faltan en la columna.  
  
    Tenga en cuenta que puede usar esta opción únicamente en las columnas que tienen los tipos datos entero, doble, booleano o cadena.
  
7. **Generate missing value indicator column** (Generar columna de indicador de valores que faltan): Seleccione esta opción si desea producir alguna indicación de si los valores de la columna cumplen los criterios para la limpieza de valores que faltan. Esta opción es especialmente útil cuando va a configurar una nueva operación de limpieza y desea asegurarse de que funciona según lo diseñado.
  
8. Envíe la canalización.

### <a name="results"></a>Results

El módulo devuelve dos salidas:  

-   **Conjunto de datos limpio**: Un conjunto de datos que consta de las columnas seleccionadas, cuyos valores que faltan se han tratado según lo especificado, junto con una columna de indicador, si selecciona dicha opción.  

    Las columnas no seleccionadas para la limpieza también se "pasan a través".  
  
-  **Transformación de limpieza**: Una transformación de datos utilizada para la limpieza, que se puede guardar en el área de trabajo y se aplica a datos nuevos más tarde.

### <a name="apply-a-saved-cleaning-operation-to-new-data"></a>Aplicar una operación de limpieza guardada a datos nuevos  

Si necesita repetir operaciones de limpieza a menudo, se recomienda que guarde la receta para la limpieza de datos como una *transformación*, para volverla a usar con el mismo conjunto de datos. Guardar una transformación de limpieza es especialmente útil si debe volver a importar con frecuencia y, a continuación, limpiar datos que tienen el mismo esquema.  
      
1.  Agregue el módulo [Apply Transformation](./apply-transformation.md) (Aplicar transformación) a la canalización.  
  
2.  Agregue el conjunto de datos que desea limpiar y conecte el conjunto de datos al puerto de entrada derecho.  
  
3.  Expanda el grupo **Transformaciones** en el panel izquierdo del diseñador. Localice la transformación guardada y arrástrela hasta la canalización.  

4.  Conecte la transformación guardada al puerto de entrada izquierdo de [Aplicar transformación](./apply-transformation.md). 

    Al aplicar una transformación guardada, no puede seleccionar las columnas a las que se aplica la transformación. Eso es porque la transformación ya se ha definido y se aplica automáticamente a las columnas especificadas en la operación original.

    Sin embargo, supongamos que ha creado una transformación en un subconjunto de columnas numéricas. Puede aplicar esta transformación a un conjunto de datos de tipos de columna mixtos sin provocar un error, porque se cambian los valores que faltan solo en las columnas numéricas coincidentes.

6.  Envíe la canalización.  

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 