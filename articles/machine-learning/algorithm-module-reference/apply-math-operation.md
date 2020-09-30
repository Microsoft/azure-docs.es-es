---
title: Aplicación de operación matemática
titleSuffix: Azure Machine Learning
description: Aprenda a usar el módulo Aplicación de operación matemática en Azure Machine Learning para aplicar una operación matemática a los valores de columna de una canalización.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 09/09/2019
ms.openlocfilehash: 654b10a283c4dcf5a1a1866ec51799aad45a8893
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90898902"
---
# <a name="apply-math-operation"></a>Aplicación de operación matemática

En este artículo se describe un módulo del diseñador de Azure Machine Learning.

Use el módulo Aplicación de operación matemática para crear cálculos que se aplican a columnas numéricas del conjunto de datos de entrada. 

Las operaciones matemáticas admitidas incluyen funciones aritméticas comunes, como la multiplicación y la división, funciones trigonométricas, una variedad de funciones de redondeo y funciones especiales que se usan en ciencia de datos, como funciones gamma y de error.  

Después de definir una operación y ejecutar la canalización, los valores se agregan al conjunto de datos. En función de cómo configure el módulo, puede:

+ Anexar los resultados al conjunto de datos. Esto es especialmente útil cuando se comprueba el resultado de la operación.
+ Reemplazar valores de columna por los nuevos valores calculados.
+ Generar una nueva columna para los resultados y no mostrar los datos originales. 

Busque la operación que necesita en estas categorías:  

- [Basic](#basic-math-operations)  
  
     Las funciones de la categoría **Básicas** se pueden usar para manipular un solo valor o una columna de valores. Por ejemplo, puede obtener el valor absoluto de todos los números de una columna o calcular la raíz cuadrada de cada valor de una columna.  
  
-   [Comparar](#comparison-operations)  
  
      Todas las funciones de la categoría **Comparación** se usan para establecer comparaciones: puede realizar una comparación por pares de los valores de dos columnas o puede comparar cada valor de una columna con una constante especificada. Por ejemplo, puede comparar columnas para determinar si los valores eran los mismos en dos conjuntos de datos. También puede usar una constante, como un valor máximo permitido, para buscar valores atípicos en una columna numérica.  
  
-   [Operaciones](#arithmetic-operations)  
  
     La categoría **Operaciones** incluye las funciones matemáticas básicas: suma, resta, multiplicación y división. Puede trabajar con columnas o con constantes. Por ejemplo, puede sumar el valor de la columna A al valor de la columna B. También puede restar una constante, como una media calculada previamente, de cada valor de la columna A.  
  
-   [Redondeo](#rounding-operations)  
  
     La categoría **Redondeo** incluye una variedad de funciones para realizar operaciones como el redondeo, el valor superior, el valor inferior y el truncamiento a varios niveles de precisión. Puede especificar el nivel de precisión para números enteros y decimales.  
  
-   [Especiales](#special-math-functions)  
  
     La categoría **Especiales** incluye funciones matemáticas que se utilizan especialmente en ciencia de datos, como las integrales elípticas y la función de error gaussiana.  
  
-   [Trigonométricas](#trigonometric-functions)  
  
     La categoría **Trigonométricas** incluye todas las funciones trigonométricas estándar. Por ejemplo, puede convertir radianes en grados o funciones de cálculo como una tangente en radianes o grados.
     Estas funciones son unarias, lo que significa que toman una sola columna de valores como entrada, aplican la función trigonométrica y devuelven una columna de valores como resultado.  Por lo tanto, debe asegurarse de que la columna de entrada sea del tipo adecuado y contenga el tipo correcto de valores para la operación especificada.   

## <a name="how-to-configure-apply-math-operation"></a>Configuración del módulo Aplicación de operación matemática  

El módulo **Aplicación de operación matemática** requiere un conjunto de datos que tenga al menos una columna que solo contenga números. Los números pueden ser discretos o continuos, pero deben ser de tipo de datos numérico, no una cadena.

Puede aplicar la misma operación a varias columnas numéricas, pero todas las columnas deben estar en el mismo conjunto de datos. 

Cada instancia de este módulo puede realizar un solo tipo de operación cada vez. Para realizar operaciones matemáticas complejas, es posible que tenga que encadenar varias instancias del módulo **Aplicación de operación matemática**.  
  
1.  Agregue el módulo **Aplicación de operación matemática** a la canalización.

1. Conecte un conjunto de datos que contenga al menos una columna numérica.  

1.  Seleccione una o varias columnas de origen en las que realizar el cálculo.   
  
    - Cualquier columna que elija debe ser de tipo de datos numérico. 
    - El intervalo de datos debe ser válido para la operación matemática seleccionada. De lo contrario, puede producirse un error o un resultado NaN (no es un número). Por ejemplo, Ln(-1,0) es una operación no válida y da como resultado un valor de `NaN`.
  
1.  Haga clic en **Category** (Categoría) para seleccionar el **tipo** de operación matemática que realizar.
    
1. Elija una operación específica de la lista de esa categoría.
  
1.  Establezca los parámetros adicionales que requiera cada tipo de operación.  
  
1.  Utilice la opción **Output mode** (Modo de salida) para indicar cómo desea que se genere la operación matemática: 

    - **Append** (Anexión). Todas las columnas que se usan como entradas se incluyen en el conjunto de datos de salida, además de anexarse una columna adicional que contiene los resultados de la operación matemática.
    - **Inplace** (Reemplazo). Los valores de las columnas que se usan como entradas se reemplazan por los nuevos valores calculados. 
    - **ResultOnly** (Solo resultado). Se devuelve una sola columna que contiene los resultados de la operación matemática.
  
1.  Envíe la canalización.  
  
## <a name="results"></a>Results

Si genera los resultados con las opciones **Append** (Anexión) o **ResultOnly** (Solo resultado), los encabezados de columna del conjunto de datos devuelto indican la operación y las columnas que se usaron. Por ejemplo, si compara dos columnas mediante el operador **Equals** (Es igual a), los resultados serían similares a los siguientes:  
  
-   **Equals(Col2_Col1)** , que indica que ha contrastado la columna Col2 con Col1.  
-   **Equals(Col2_$10)** , que indica que ha comparado la columna 2 con la constante 10.  

Aunque use la opción **Inplace**, los datos de origen no se eliminan ni se cambian; la columna del conjunto de datos original sigue estando disponible en el diseñador. Para ver los datos originales, puede conectar el módulo [Agregar columnas](add-columns.md) y combinarlo con la salida de **Aplicación de operación matemática**.  
    
## <a name="basic-math-operations"></a>Operaciones matemáticas básicas 

Las funciones de la categoría **Básicas** normalmente toman un único valor de una columna, realizan la operación predefinida y devuelven un único valor. Para algunas funciones, puede especificar una constante o un conjunto de columnas como segundo argumento.  
  
 Azure Machine Learning admite las siguientes funciones en la categoría **Básicas**:  

### <a name="abs"></a>Abs

Devuelve el valor absoluto de las columnas seleccionadas.  
  
### <a name="atan2"></a>Atan2

Devuelve una tangente inversa de cuatro cuadrantes.  

Seleccione las columnas que contienen las coordenadas de puntos. Para el segundo argumento, que corresponde a la coordenada x, también puede especificar una constante.  

Corresponde a la función ATAN2 en Matlab.  

### <a name="conj"></a>Conj

Devuelve el conjugado de los valores de la columna seleccionada.  

### <a name="cuberoot"></a>CubeRoot

Calcula la raíz cúbica de los valores de la columna seleccionada.  

### <a name="doublefactorial"></a>DoubleFactorial  
 Calcula el doble factorial de los valores de la columna seleccionada. El doble factorial es una extensión de la función factorial normal y se expresa como x!!.  

### <a name="eps"></a>Eps

Devuelve el tamaño de la diferencia entre el valor actual y el siguiente número de doble precisión más alto. Corresponde a la función EPS en Matlab.  
  
### <a name="exp"></a>Exp

Devuelve e elevado a la potencia del valor de la columna seleccionada. Es igual que la función Excel EXP.  

### <a name="exp2"></a>Exp2

Devuelve el valor exponencial de base 2 de los argumentos, resolviendo para y = x * 2<sup>t</sup>, donde t es una columna de valores que contienen exponentes.  

En **Column set** (Conjunto de columnas), seleccione la columna que contiene los valores de exponente t.

Para **Exp2** puede especificar un segundo argumento x, que puede ser una constante u otra columna de valores. En **Second argument type** (Tipo de segundo argumento), indique si va a proporcionar el multiplicador x como constante o un valor en una columna.  

Por ejemplo, si selecciona una columna con los valores {0,1,2,3,4,5} para el multiplicador y el exponente, la función devuelve {0, 2, 8, 24, 64 160).  

### <a name="expminus1"></a>ExpMinus1 

Devuelve el exponente negativo para los valores de la columna seleccionada.  

### <a name="factorial"></a>Factorial
Devuelve el factorial de los valores de la columna seleccionada.  

### <a name="hypotenuse"></a>Hypotenuse
Calcula la hipotenusa de un triángulo en el que la longitud de un lado se especifica como una columna de valores y la longitud del segundo lado se especifica como una constante o como dos columnas.  

### <a name="ln"></a>Ln

Devuelve el logaritmo natural de los valores de la columna seleccionada.  

### <a name="lnplus1"></a>LnPlus1

Devuelve el logaritmo natural más uno de los valores de la columna seleccionada.  

### <a name="log"></a>Log

Devuelve el logaritmo de los valores de la columna seleccionada, dada la base especificada.  

Puede especificar la base (el segundo argumento) como una constante o seleccionando otra columna de valores.  

### <a name="log10"></a>Log10

Devuelve los valores del logaritmo de base 10 de la columna seleccionada.  

### <a name="log2"></a>Log2

Devuelve los valores del logaritmo de base 2 de la columna seleccionada.  

### <a name="nthroot"></a>NthRoot
Devuelve la radiación del valor con un número n que especifique.  

Seleccione las columnas para las que desea calcular la raíz mediante la opción **ColumnSet** (Conjunto de columnas).  

En **Second argument type** (Tipo de segundo argumento), seleccione otra columna que contenga la raíz o especifique una constante para usarla como raíz.  

Si el segundo argumento es una columna, cada valor de la columna se usa como el valor de n para la fila correspondiente. Si el segundo argumento es una constante, escriba el valor de n en el cuadro de texto **Second argument** (Segundo argumento).
### <a name="pow"></a>Pow

Calcula X elevado a la potencia de Y para cada uno de los valores de la columna seleccionada.  

En primer lugar, seleccione las columnas que contengan la **base**, que debe ser un valor flotante, mediante la opción **ColumnSet** (Conjunto de columnas).  

En **Second argument type** (Tipo de segundo argumento), seleccione la columna que contenga el exponente o especifique una constante para usarla como exponente.  

Si el segundo argumento es una columna, cada valor de la columna se usa como el exponente para la fila correspondiente. Si el segundo argumento es una constante, escriba el valor del exponente en el cuadro de texto **Second argument** (Segundo argumento).  

### <a name="sqrt"></a>Sqrt

Devuelve la raíz cuadrada de los valores de la columna seleccionada.  

### <a name="sqrtpi"></a>SqrtPi

Para cada valor de la columna seleccionada, multiplica el valor por pi y, a continuación, devuelve la raíz cuadrada del resultado.  

### <a name="square"></a>Square

Eleva al cuadrado los valores de la columna seleccionada.  

## <a name="comparison-operations"></a>Operaciones de comparación  

Use las funciones de comparación del diseñador de Azure Machine Learning siempre que necesite contrastar dos conjuntos de valores. Por ejemplo, en una canalización podría necesitar realizar las siguientes operaciones de comparación:  

- Evaluar una columna de modelo de puntuaciones de probabilidad con un valor de umbral.
- Determinar si dos conjuntos de resultados son iguales. Para cada fila que sea diferente, se agrega una marca FALSE, que puede resultar útil para posteriores operaciones de procesamiento o de filtrado.  

### <a name="equalto"></a>EqualTo

Devuelve True si los valores son iguales.  

### <a name="greaterthan"></a>GreaterThan

Devuelve True si los valores de **Column set** (Conjunto de columnas) son mayores que la constante especificada o mayores que los valores correspondientes de la columna de comparación.  

### <a name="greaterthanorequalto"></a>GreaterThanOrEqualTo

Devuelve True si los valores de **Column set** (Conjunto de columnas) son mayores o iguales que la constante especificada o mayores o iguales que los valores correspondientes de la columna de comparación.  

### <a name="lessthan"></a>LessThan

Devuelve True si los valores de **Column set** (Conjunto de columnas) son menores que la constante especificada o menores que los valores correspondientes de la columna de comparación.  
  
### <a name="lessthanorequalto"></a>LessThanOrEqualTo

Devuelve True si los valores de **Column set** (Conjunto de columnas) son menores o iguales que la constante especificada o menores o iguales que los valores correspondientes de la columna de comparación.  

### <a name="notequalto"></a>NotEqualTo

Devuelve True si los valores de **Column set** (Conjunto de columnas) no son iguales a la constante o la columna de comparación, y devuelve False si son iguales.  

### <a name="pairmax"></a>PairMax

Devuelve el valor que sea mayor: el valor de **Column set** (Conjunto de columnas) o el valor de la constante o la columna de comparación.  

### <a name="pairmin"></a>PairMin

Devuelve el valor que sea menor: el valor de **Column set** (Conjunto de columnas) o el valor de la constante o la columna de comparación.  
  
##  <a name="arithmetic-operations"></a>Operaciones aritméticas   

Incluye las operaciones aritméticas básicas: suma, resta, división y multiplicación.  Dado que la mayoría de las operaciones son binarias, lo que requiere dos números, primero seleccionará la operación y, a continuación, elegirá la columna o los números que usar en el primer y el segundo argumento.

El orden en que se eligen las columnas para la división y la resta puede parecer poco intuitivo; sin embargo, para facilitar la comprensión de los resultados, el encabezado de columna indica el nombre de la operación y el orden en el que se usaron las columnas.

Operación|Num1|Num2|Columna de resultados|Valor del resultado|
----|----|----|----|----
|Suma|1|5|Add(Num2_Num1)| 4|
|Multiplicación|1|5|Multiple(Num2_Num1)|5|
|Resta|1|5|Subtract(Num2_Num1)|4|
|Resta|0|1|Subtract(Num2_Num1)|0|
|División|1|5|Divide(Num2_Num1)|5|
|División|0|1|Divide(Num2_Num1)|Infinity|

### <a name="add"></a>Sumar

Especifique las columnas de origen mediante **Column set** (Conjunto de columnas) y, a continuación, sume a esos valores un número especificado en **Second argument** (Segundo argumento).  

Para sumar los valores de dos columnas, elija una columna o columnas mediante **Column set** (Conjunto de columnas) y, a continuación, seleccione una segunda columna mediante **Second argument** (Segundo argumento).  

### <a name="divide"></a>Divide

Divide los valores de **Column set** (Conjunto de columnas) entre una constante o entre los valores de columna definidos en **Second argument** (Segundo argumento).  En otras palabras, primero seleccionará el divisor y después el dividendo. El valor de salida es el cociente.

### <a name="multiply"></a>Multiplicar

Multiplica los valores de **Column set** (Conjunto de columnas) por la constante o los valores de columna especificados.  

### <a name="subtract"></a>Restar

Especifique la columna de valores en la que operar (el *minuendo*); para ello, elija una columna diferente mediante la opción **Column set** (Conjunto de columnas). A continuación, especifique el número que restar (el *sustraendo*) mediante la lista desplegable **Second argument** (Segundo argumento). Puede elegir una constante o una columna de valores.

##  <a name="rounding-operations"></a>Operaciones de redondeo 

El diseñador de Azure Machine Learning admite diversas operaciones de redondeo. Para muchas operaciones, debe especificar la cantidad de precisión que se va a utilizar al redondear. Puede usar un nivel de precisión estática, especificado como una constante, o puede aplicar un valor de precisión dinámica obtenido a partir de una columna de valores.  

- Si usa una constante, establezca **Precision Type** (Tipo de precisión) en **Constant** (Constante) y, a continuación, escriba el número de dígitos como un entero en el cuadro de texto **Constant Precision** (Precisión de constante). Si escribe un valor no entero, el módulo no genera un error, pero los resultados pueden ser inesperados.  

- Para usar un valor de precisión diferente para cada fila del conjunto de datos, establezca **Precision Type** en **ColumnSet** (Conjunto de columnas) y, a continuación, elija la columna que contiene los valores de precisión adecuados.  

### <a name="ceiling"></a>Ceiling

Devuelve el valor superior de los valores de **Column set** (Conjunto de columnas).  

### <a name="ceilingpower2"></a>CeilingPower2

Devuelve el valor superior al cuadrado de los valores de **Column set** (Conjunto de columnas).  

### <a name="floor"></a>Floor

Devuelve el valor inferior de los valores de **Column set** (Conjunto de columnas), a la precisión especificada.  

### <a name="mod"></a>Mod

Devuelve la parte fraccionaria de los valores de **Column set** (Conjunto de columnas), a la precisión especificada.  

### <a name="quotient"></a>Quotient

Devuelve la parte fraccionaria de los valores de **Column set** (Conjunto de columnas), a la precisión especificada.  

### <a name="remainder"></a>Remainder

Devuelve el resto de los valores de **Column set** (Conjunto de columnas).  

### <a name="rounddigits"></a>RoundDigits

Devuelve los valores de **Column set** (Conjunto de columnas), redondeados por la regla 4/5 al número de dígitos especificado.  

### <a name="rounddown"></a>RoundDown

Devuelve los valores de **Column set** (Conjunto de columnas), redondeados hacia abajo al número de dígitos especificado.  

### <a name="roundup"></a>RoundUp

Devuelve los valores de **Column set** (Conjunto de columnas), redondeados hacia arriba al número de dígitos especificado.  

### <a name="toeven"></a>ToEven

Devuelve los valores de **Column set** (Conjunto de columnas), redondeado al número par entero más cercano.  

### <a name="toodd"></a>ToOdd

Devuelve los valores de **Column set** (Conjunto de columnas), redondeado al número impar entero más cercano.  

### <a name="truncate"></a>Truncate

Trunca los valores de **Column set** (Conjunto de columnas) quitando todos los dígitos no permitidos por la precisión especificada.  
  
## <a name="special-math-functions"></a>Funciones matemáticas especiales

Esta categoría incluye funciones matemáticas especializadas que se suelen usar en ciencia de datos. A menos que se indique lo contrario, la función es unaria y devuelve el cálculo especificado para cada valor de la columna o columnas seleccionadas.  

### <a name="beta"></a>Beta

Devuelve el valor de la función beta de Euler.  

### <a name="ellipticintegrale"></a>EllipticIntegralE
Devuelve el valor de la integral elíptica incompleta.  
  

### <a name="ellipticintegralk"></a>EllipticIntegralK

Devuelve el valor de la integral elíptica completa (K).  

### <a name="erf"></a>Erf

Devuelve el valor de la función de error.  

La función de error (también denominada función de error de Gauss) es una función especial de forma sigmoide que se usa en probabilidad para describir la difusión.  

### <a name="erfc"></a>Erfc

Devuelve el valor de la función de error complementaria.  

Erfc se define como 1 – erf(x).  

### <a name="erfscaled"></a>ErfScaled

Devuelve el valor de la función de error escalada.  

La versión escalada de la función de error se puede usar para evitar el desbordamiento aritmético.  

### <a name="erfinverse"></a>ErfInverse

Devuelve el valor de la función erf inversa.  

### <a name="exponentialintegralein"></a>ExponentialIntegralEin

Devuelve el valor de la integral exponencial Ei.  

### <a name="gamma"></a>Gamma

Devuelve el valor de la función gamma.  

### <a name="gammaln"></a>GammaLn

Devuelve el logaritmo natural de la función gamma.  

### <a name="gammaregularizedp"></a>GammaRegularizedP

Devuelve el valor de la función gamma incompleta regularizada.  

Esta función toma un segundo argumento, que se puede proporcionar como una constante o como una columna de valores.  

### <a name="gammaregularizedpinverse"></a>GammaRegularizedPInverse

Devuelve el valor de la función gamma incompleta regularizada inversa.  

Esta función toma un segundo argumento, que se puede proporcionar como una constante o como una columna de valores.  

### <a name="gammaregularizedq"></a>GammaRegularizedQ  

Devuelve el valor de la función gamma incompleta regularizada.  

Esta función toma un segundo argumento, que se puede proporcionar como una constante o como una columna de valores.  

### <a name="gammaregularizedqinverse"></a>GammaRegularizedQInverse

Devuelve el valor de la función gamma incompleta regularizada generalizada inversa.

Esta función toma un segundo argumento, que se puede proporcionar como una constante o como una columna de valores.  

### <a name="polygamma"></a>PolyGamma

Devuelve el valor de la función poligamma.  

Esta función toma un segundo argumento, que se puede proporcionar como una constante o como una columna de valores.

##  <a name="trigonometric-functions"></a>Funciones trigonométricas 

Esta categoría incluye la mayoría de las funciones trigonométricas y trigonométricas inversas importantes. Todas las funciones trigonométricas son unarias y no requieren ningún argumento adicional.  

### <a name="acos"></a>Acos

Calcula el arcocoseno de los valores de la columna.  

### <a name="acosdegree"></a>AcosDegree

Calcula el arcocoseno de los valores de la columna en grados.  

### <a name="acosh"></a>Acosh

Calcula el arcocoseno hiperbólico de los valores de la columna.  

### <a name="acot"></a>Acot

Calcula la arcocotangente de los valores de la columna.  

### <a name="acotdegrees"></a>AcotDegrees

Calcula la arcocotangente de los valores de la columna en grados.  

### <a name="acoth"></a>Acoth

Calcula la arcocotangente hiperbólica de los valores de la columna.  

### <a name="acsc"></a>Acsc

Calcula la arcocosecante de los valores de la columna.  

### <a name="acscdegrees"></a>AcscDegrees

Calcula la arcocosecante de los valores de la columna en grados.  
### <a name="asec"></a>Asec

Calcula la arcosecante de los valores de la columna.  

### <a name="asecdegrees"></a>AsecDegrees

Calcula la arcosecante de los valores de la columna en grados.  

### <a name="asech"></a>Asech

Calcula la arcosecante hiperbólica de los valores de la columna.  

### <a name="asin"></a>Asin

Calcula el arcoseno de los valores de la columna.  

### <a name="asindegrees"></a>AsinDegrees

Calcula el arcoseno de los valores de la columna en grados.  

### <a name="asinh"></a>Asinh

Calcula el arcoseno hiperbólico de los valores de la columna.  

### <a name="atan"></a>Atan

Calcula la arcotangente de los valores de la columna.  

### <a name="atandegrees"></a>AtanDegrees

Calcula la arcotangente de los valores de la columna en grados.  

### <a name="atanh"></a>Atanh

Calcula la arcotangente hiperbólica de los valores de la columna.  

### <a name="cos"></a>Cos

Calcula el coseno de los valores de la columna.  

### <a name="cosdegrees"></a>CosDegrees

Calcula el coseno de los valores de la columna en grados.  

### <a name="cosh"></a>Cosh

Calcula el coseno hiperbólico de los valores de la columna.  

### <a name="cot"></a>Cot

Calcula la cotangente de los valores de la columna.  

### <a name="cotdegrees"></a>CotDegrees

Calcula la cotangente de los valores de la columna en grados.  

### <a name="coth"></a>Coth
Calcula la cotangente hiperbólica de los valores de la columna.  

### <a name="csc"></a>Csc

Calcula la cosecante de los valores de la columna.  

### <a name="cscdegrees"></a>CscDegrees

Calcula la cosecante de los valores de la columna en grados.  

### <a name="csch"></a>Csch

Calcula la cosecante hiperbólica de los valores de la columna.  

### <a name="degreestoradians"></a>DegreesToRadians

Convierte los grados en radianes.  

### <a name="sec"></a>Sec

Calcula la secante de los valores de la columna.  

### <a name="asecdegrees"></a>aSecDegrees

Calcula la secante de los valores de la columna en grados.  

### <a name="asech"></a>aSech

Calcula la secante hiperbólica de los valores de la columna.  

### <a name="sign"></a>Firma

Devuelve el signo de los valores de la columna.  

### <a name="sin"></a>Sin

Calcula el seno de los valores de la columna.  

### <a name="sinc"></a>Sinc

Calcula el valor seno-coseno de los valores de la columna.  

### <a name="sindegrees"></a>SinDegrees

Calcula el seno de los valores de la columna en grados.  

### <a name="sinh"></a>Sinh

Calcula el seno hiperbólico de los valores de la columna.  

### <a name="tan"></a>Tan

Calcula la tangente de los valores de la columna.  

### <a name="tandegrees"></a>TanDegrees

Calcula la tangente del argumento en grados.  

### <a name="tanh"></a>Tanh

Calcula la tangente hiperbólica de los valores de la columna.  
  
## <a name="technical-notes"></a>Notas técnicas

Tenga cuidado al seleccionar más de una columna como segundo operador. Los resultados son fáciles de entender si la operación es sencilla, como la suma de una constante a todas las columnas. 

Supongamos que el conjunto de datos tiene varias columnas y que suma el conjunto de datos a sí mismo. En los resultados, cada columna se suma a sí misma, de la siguiente manera:  
  
|Num1|Num2|Num3|Add(Num1_Num1)|Add(Num2_Num2)|Add(Num3_Num3)|
|----|----|----|----|----|----|
|1|5|2|2|10|4|
|2|3|-1|4|6|-2|
|0|1|-1|0|2|-2|

Si necesita realizar cálculos más complejos, puede encadenar varias instancias del módulo **Aplicación de operación matemática**. Por ejemplo, podría sumar dos columnas mediante una instancia de **Aplicación de operación matemática** y, a continuación, usar otra instancia de **Aplicación de operación matemática** para dividir la suma entre una constante para obtener la media.  
  
También puede usar uno de los siguientes módulos para realizar todos los cálculos a la vez, mediante un script SQL, R o Python:
 
+ [Ejecución script de R](execute-r-script.md)
+ [Ejecución de script de Python](execute-python-script.md)
+ [Aplicación de transformaciones de SQL](apply-sql-transformation.md)   
  
## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 
