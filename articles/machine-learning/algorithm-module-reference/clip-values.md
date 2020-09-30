---
title: Recorte de valores
titleSuffix: Azure Machine Learning
description: Aprenda a usar el módulo Recorte de valores en Azure Machine Learning para detectar valores atípicos y recortar o reemplazar sus valores.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 09/09/2019
ms.openlocfilehash: 99fb41542dff28997438881abad71da11e927a78
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90898813"
---
# <a name="clip-values"></a>Recorte de valores

En este artículo se describe un módulo del diseñador de Azure Machine Learning.

Use el módulo Clip Values (Recorte de valores) para identificar y, opcionalmente, reemplazar los valores de los datos que estén por encima o por debajo de un umbral especificado por una media, una constante u otro valor sustituto.  

Conecte el módulo a un conjunto de valores que tenga los números que desea recortar, elija las columnas con las que trabajar y establezca un umbral o un intervalo de valores y un método de reemplazo. El módulo puede generar solo los resultados o los valores modificados que se anexen al conjunto de datos original.

## <a name="how-to-configure-clip-values"></a>Configuración de Clip Values (Recorte de valores)

Antes de comenzar, identifique las columnas que desea recortar y el método que se va a usar. Se recomienda probar primero cualquier método de recorte en un pequeño subconjunto de datos.

El módulo aplica los mismos criterios y método de reemplazo a **todas** columnas que se incluyen en la selección. Por lo tanto, asegúrese de excluir las que no desee cambiar.

Si necesita aplicar métodos de recorte o criterios diferentes a algunas columnas, debe usar una instancia de **Clip Values** (Recorte de valores) nueva para cada conjunto de columnas similares.

1.  Agregue el módulo **Clip Values** (Recorte de valores) a la canalización y conéctelo al conjunto de datos que desee modificar. Puede encontrar este módulo en **Data Transformation** (Transformación de datos), en la categoría **Scale and Reduce** (Escalar y reducir). 
  
1.  En **List of columns** (Lista de columnas), use el selector de columnas para elegir las columnas a las que se aplicará **Clip Values** (Recorte de valores).  
  
1.  En **Set of thresholds** (Conjunto de umbrales), elija una de las siguientes opciones de la lista desplegable. Estas opciones determinan cómo se establecen los límites superior e inferior de los valores aceptables frente a los que se deben recortar.  
  
    - **ClipPeaks** (Recortar picos): al recortar los valores por picos, solo se especifica un límite superior. Los valores mayores que el límite se reemplazan.
  
    -  **ClipSubpeaks** (Recortar subpicos): al recortar los valores por subpicos, solo se especifica un límite inferior. Se reemplazan los valores que son menores que el límite.  
  
    - **ClipPeaksAndSubpeaks** (Recortar picos y subpicos): al recortar los valores por picos y subpicos, puede especificar los límites superior e inferior. Los valores que se encuentran fuera de ese intervalo se reemplazan. Los valores que coinciden con los valores de límite no cambian.
  
1.  En función de la selección realizada en el paso anterior, puede establecer los valores de umbral siguientes: 

    + **Lower threshold** (Umbral inferior): solo se muestra si elige **ClipSubPeaks**
    + **Upper threshold** (Umbral superior): solo se muestra si elige **ClipPeaks**
    + **Umbral**: solo se muestra si elige **ClipPeaksAndSubPeaks**

    Para cada tipo de umbral, elija **Constant** (Constante) o **Percentile**  (Percentil).

1. Si selecciona **Constant** (Constante), escriba el valor máximo o mínimo en el cuadro de texto. Por ejemplo, supongamos que sabe que el valor 999 se usó como valor de marcador de posición. Puede elegir **Constant** (Constante) para el umbral superior y escribir 999 en **Constant value for upper threshold** (Valor constante para umbral superior).
  
1. Si elige **Percentile** (Percentil), se restringen los valores de columna a un intervalo de percentiles. 

    Por ejemplo, supongamos que desea mantener solo los valores del intervalo de percentiles 10-80 y reemplazar todos los demás. Elija **Percentile** (Percentil) y escriba 10 para **Percentile value for lower threshold** (Valor de percentil para umbral inferior) y 80 para **Percentile value for upper threshold** (Valor de percentil para umbra superior). 

    Consulte la sección sobre [percentiles](#examples-for-clipping-using-percentiles) para algunos ejemplos de cómo usar intervalos de percentiles.  
  
1.  Defina un valor sustituto.

    Los números que coinciden exactamente con los límites especificados se consideran dentro del intervalo permitido de valores y, por tanto, no se reemplazan. Todos los números que se encuentran fuera del intervalo especificado se reemplazan por el valor sustituto. 
  
    + **Substitute value for peaks** (Valor de sustitución de picos): define el valor de sustitución en todos los valores de columna mayores que el umbral especificado.  
    + **Substitute value for subpeaks** (Valor de sustitución de subpicos): define el valor de sustitución en todos los valores de columna menores que el umbral especificado.  
    + Si usa la opción **ClipPeaksAndSubpeaks**, puede especificar valores de reemplazo independientes para los valores recortados superior e inferior.  

    Se admiten los siguientes valores de reemplazo:  
  
    -   **Umbral**: reemplaza los valores recortados por el valor de umbral especificado.  
  
    -   **Mean** (Promedio): reemplaza los valores recortados por el promedio de los valores de la columna. El promedio se calcula antes de que se recorten los valores.  
  
    -   **Median** (Mediana): reemplaza los valores recortados por la mediana de los valores de la columna. La mediana se calcula antes de que se recorten los valores.   
  
    -   **Missing** (Ausente): reemplaza los valores recortados por el valor que falta (vacío).  
  
1.  **Add indicator columns** (Agregar columnas indicativas): seleccione esta opción si desea generar una nueva columna que indique si la operación de recorte especificada se ha aplicado a los datos de esa fila. Esta opción es útil cuando se prueba un nuevo conjunto de valores de recorte y sustitución.  
  
1. **Overwrite flag** (Sobrescribir marca): indica cómo desea que se generen los nuevos valores. De forma predeterminada, **Clip Values** (Recorte de valores) crea una nueva columna con los valores máximos recortados al umbral deseado. Los nuevos valores sobrescriben la columna original.  
  
    Para mantener la columna original y agregar una nueva columna con los valores recortados, anule la selección de esta opción.  
  
1.  Envíe la canalización.  
  
    Haga clic con el botón derecho en el módulo **Clip values** (Recortar valores) y seleccione **Visualize** (Visualizar), o bien seleccione el módulo y cambie a la pestaña **Outputs** (Salidas) en el panel derecho, haga clic en el icono del histograma de **Port outputs** (Salidas del puerto), para revisar los valores y asegurarse de que la operación de recorte cumple sus expectativas.  
 
### <a name="examples-for-clipping-using-percentiles"></a>Ejemplos de recorte con percentiles

Para entender cómo funciona el recorte por percentiles, considere un conjunto de 10 filas, con una instancia de cada una de los valores 1-10.  
  
- Si usa los percentiles como umbral superior, en el valor del percentil 90, el 90 % de todos los valores del conjunto de valores debe ser inferior a ese valor.  
  
- Si usa los percentiles como umbral inferior, en el valor del percentil 10, el 10 % de todos los valores del conjunto de valores debe ser inferior a ese valor.  
  
1.  En **Set of thresholds** (Conjunto de umbrales) elija **ClipPeaksAndSubPeaks**.  
  
1.  En **Upper threshold** (Umbral superior) elija **Percentile** (Percentil) y, en **Percentile number** (Número de percentil), escriba 90.  
  
1.  En **Upper substitute value** (Valor de sustitución superior) elija **Missing Value** (Valor ausente).  
  
1.  En **Lower threshold** (Umbral inferior) elija **Percentile** (Percentil) y, en **Percentile number** (Número de percentil), escriba 10.  
  
1.  En **Lower substitute value** (Valor de sustitución inferior) elija **Missing Value** (Valor ausente).  
  
1.  Anule la selección de la opción **Overwrite flag** (Sobrescribir marca) y seleccione la opción **Add indicator columns** (Agregar columnas indicativas).  
  
Ahora pruebe la misma canalización con 60 como percentil del umbral superior y 30 como el inferior, y use el valor de umbral como valor de sustitución. En la tabla siguiente se comparan estos dos resultados:  
  
1.  Reemplazar por el ausente; umbral superior = 90; umbral inferior = 20  
  
1.  Reemplazar por el umbral; percentil superior = 60; percentil inferior = 40  
  
|Datos originales|Reemplazar por el ausente|Reemplazar por el umbral|  
|-------------------|--------------------------|----------------------------|  
|1<br /><br /> 2<br /><br /> 3<br /><br /> 4<br /><br /> 5<br /><br /> 6<br /><br /> 7<br /><br /> 8<br /><br /> 9<br /><br /> 10|TRUE<br /><br /> TRUE<br /><br /> 3, FALSE<br /><br /> 4, FALSE<br /><br /> 5, FALSE<br /><br /> 6, FALSE<br /><br /> 7, FALSE<br /><br /> 8, FALSE<br /><br /> 9, FALSE<br /><br /> TRUE|4, TRUE<br /><br /> 4, TRUE<br /><br /> 4, TRUE<br /><br /> 4, TRUE<br /><br /> 5, FALSE<br /><br /> 6, FALSE<br /><br /> 7, TRUE<br /><br /> 7, TRUE<br /><br /> 7, TRUE<br /><br /> 7, TRUE| 
 
## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 
