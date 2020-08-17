---
title: 'Uso de R con Machine Learning Studio (clásico): Azure'
description: Use este tutorial de programación R para empezar a usar Azure Machine Learning Studio (clásico) en R para crear una solución de previsión.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: previous-author=heatherbshapiro, previous-ms.author=hshapiro
ms.date: 03/01/2019
ms.openlocfilehash: 719b96c9186d463ca3ee41c6fb401a8f22c4c11c
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87431970"
---
# <a name="get-started-with-azure-machine-learning-studio-classic-in-r"></a>Introducción a Azure Machine Learning Studio (clásico) en R

**SE APLICA A:**  ![sí](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico)   ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)


<!-- Stephen F Elston, Ph.D. -->
En este tutorial, aprenderá a usar ML Studio (clásico) para crear, probar y ejecutar código de R. Al final, tendrá una solución completa de previsión.  

> [!div class="checklist"]
> * Cree código para la limpieza y la transformación de los datos.
> * Analice las correlaciones entre muchas de las variables en el conjunto de datos.
> * Cree un modelo de previsión de serie temporal estacional para la producción de leche.


Azure Machine Learning Studio (clásico) contiene muchos módulos versátiles de manipulación de datos y aprendizaje automático. Además, con el lenguaje de programación R, esta combinación une la escalabilidad y sencillez de la implementación de Studio (clásico) con la flexibilidad y el análisis profundo de R.

La previsión es un método de análisis ampliamente utilizado y bastante útil. La previsión suele utilizarse para crear predicciones de ventas de productos estacionales, para determinar los niveles de inventario óptimos o para predecir las variables macroeconómicas. La previsión suele realizarse con modelos de serie temporal. Los datos de series temporales son datos en los que los valores tienen un índice temporal. El índice temporal puede ser normal, es decir, cada mes o cada minuto; o bien puede ser irregular. Los modelos de serie temporal se basan en datos de series temporales. El lenguaje de programación R contiene un marco de trabajo flexible y amplias características de análisis de datos de serie temporales.

## <a name="get-the-data"></a>Obtener los datos

En este tutorial, usará datos sobre la producción y precios de lácteos en California, que incluyen información mensual acerca de la producción de varios de los productos lácteos, así como sobre el precio de la grasa láctea, materia prima de referencia.

Los datos usados en este artículo, junto con los scripts de R, pueden descargarse desde [MachineLearningSamples-Notebooks/studio-samples](https://github.com/Azure-Samples/MachineLearningSamples-Notebooks/tree/master/studio-samples). Los datos del archivo `cadairydata.csv` se resumieron a partir de la información disponible gracias a la Universidad de Wisconsin en [https://dairymarkets.com](https://dairymarkets.com).



## <a name="interact-with-r-language-in-machine-learning-studio-classic"></a><a id="mlstudio"></a>Interactuación con el lenguaje R en Machine Learning Studio (clásico)

En esta sección se guía al usuario por algunos aspectos básicos de la interacción con el lenguaje de programación R en el entorno de Machine Learning Studio (clásico). El lenguaje R proporciona una herramienta eficaz para crear módulos de manipulación de datos y de análisis personalizado en el entorno de Azure Machine Learning Studio (clásico).

Utilizaremos RStudio para desarrollar, probar y depurar el código R a escala reducida. A continuación, este código se corta y pega en un módulo [Ejecutar script R][execute-r-script] listo para ejecutarse en Azure Machine Learning Studio (clásico).  

### <a name="the-execute-r-script-module"></a>Módulo Ejecutar script de R

En Machine Learning Studio (clásico), los scripts R se ejecutan en el módulo [Ejecutar script R][execute-r-script]. En la ilustración 1 se muestra un ejemplo del módulo [Ejecutar script R][execute-r-script] en Machine Learning Studio (clásico).

 ![Lenguaje de programación R: el módulo Ejecutar script de R seleccionado en Machine Learning Studio (clásico)](./media/r-quickstart/fig1.png)

*Ilustración 1. Entorno de Machine Learning Studio (clásico) que muestra el módulo Ejecutar script de R seleccionado.*

Tomando como referencia la figura 1, veamos algunas de las partes principales del entorno de Machine Learning Studio (clásico) para trabajar con el módulo [Ejecutar script R][execute-r-script].

* Los módulos utilizados en el experimento se muestran en el panel central.
* La parte superior del panel derecho contiene una ventana para ver y editar los scripts de código R.  
* La parte inferior del panel derecho muestra algunas propiedades del módulo [Ejecutar script R][execute-r-script]. Para ver los registros de error y de salida, seleccione los puntos correspondientes de este panel.

Por supuesto, analizaremos el módulo [Ejecutar script R][execute-r-script] con mayor detalle en el resto de este artículo.

Cuando se utilicen funciones complejas de R, es recomendable editar, probar y depurar el código en RStudio. Al igual que con cualquier desarrollo de software, amplíe el código de forma incremental y pruébelo en casos de prueba más sencillos. Luego, corte y pegue las funciones en la ventana de scripts R del módulo [Ejecutar script R][execute-r-script]. Este enfoque le permite aprovechar el entorno de desarrollo integrado (IDE) de RStudio y la eficacia de Azure Machine Learning Studio (clásico).  

#### <a name="execute-r-code"></a>Ejecución del código R

Todo el código R del módulo [Ejecutar script R][execute-r-script] se ejecutará cuando ejecute el experimento al seleccionar el botón **Ejecutar**. Cuando haya finalizado la ejecución, aparecerá una marca de verificación en el icono [Ejecutar script R][execute-r-script].

#### <a name="defensive-r-coding-for-azure-machine-learning"></a>Codificación R defensiva para Azure Machine Learning

Si va a desarrollar código R, por ejemplo para un servicio web mediante Azure Machine Learning Studio (clásico), deberá planear la forma en que dicho código va a tratar las entradas de datos inesperadas y las excepciones. Para mantener la claridad, he preferido no entrar en muchos detalles en cuanto al método de comprobación o control de excepciones en la mayoría de los ejemplos de código mostrados. Sin embargo, conforme avancemos compartiré varios ejemplos de funciones mediante la capacidad de control de excepciones de R.  

Si necesita una explicación más completa del control de excepciones de R, se recomienda que lea las secciones correspondientes del manual de Wickham que se detallan en [Lecturas adicionales](#appendixb).

#### <a name="debug-and-test-r-in-machine-learning-studio-classic"></a>Depuración y prueba de R en Machine Learning Studio (clásico)

Nuevamente, recomiendo probar y depurar el código de R a escala reducida en RStudio. Sin embargo, hay casos en los que necesitará rastrear problemas de código R en el propio módulo [Ejecutar script R][execute-r-script]. Además, se recomienda comprobar los resultados en Machine Learning Studio (clásico).

Los resultados de la ejecución del código R y de la plataforma Azure Machine Learning Studio (clásico) se encuentran principalmente en el archivo output.log. Se mostrará información adicional en el archivo error.log.  

Si se produce un error en Machine Learning Studio (clásico) durante la ejecución del código R, lo primero que debe hacer es consultar el archivo error.log. Este archivo puede contener mensajes de error útiles que le ayudarán a comprender y a corregir el error. Para ver el archivo error.log, seleccione **Ver registro de errores** en el **panel de propiedades** del módulo [Ejecutar script R][execute-r-script] que contiene el error.

Por ejemplo, ejecuté el siguiente código R con una variable y sin definir en un módulo [Ejecutar script R][execute-r-script]:

```r
x <- 1.0
z <- x + y
```

Este código no se puede ejecutar, lo que da lugar a un error. Cuando selecciona **Ver registro de errores** en el **panel de propiedades**, se muestra la pantalla de la Ilustración 2.

  ![Mensaje de error emergente](./media/r-quickstart/fig2.png)

*Ilustración 2. Mensaje de error emergente.*

Parece que tenemos que consultar el archivo output.log para poder ver el mensaje de error R. Seleccione el módulo [Ejecutar script R][execute-r-script] y, luego, el elemento **Ver output.log** del **panel de propiedades** que se encuentra a la derecha. Se abrirá una nueva ventana del explorador y podrá ver lo siguiente.

```output
[Critical]     Error: Error 0063: The following error occurred during evaluation of R script:
---------- Start of error message from R ----------
object 'y' not found


object 'y' not found
----------- End of error message from R -----------
```

Este mensaje de error no contiene sorpresas e identifica claramente el problema.

Imprima estos valores en el archivo output.log para comprobar el valor de los objetos en R. Las reglas para examinar los valores de objeto son las mismas que las que se usan en sesiones interactivas de código R. Por ejemplo, si escribe un nombre de variable en una línea, el valor del objeto se imprimirá al archivo output.log.  

#### <a name="packages-in-machine-learning-studio-classic"></a>Paquetes de Machine Learning Studio (clásico)

Studio incluye más de 350 paquetes del lenguaje R preinstalados. Puede usar el código siguiente en el módulo [Ejecutar script R][execute-r-script] para recuperar una lista de paquetes preinstalados.

```r
data.set <- data.frame(installed.packages())
maml.mapOutputPort("data.set")
```

Siga leyendo si no comprende la última línea de este código. En el resto del artículo se describe con detalle el uso de R en el entorno de Studio (clásico).

### <a name="introduction-to-rstudio"></a>Introducción a RStudio

RStudio es un IDE ampliamente usado para R, que se emplea para editar, probar y depurar el código R utilizado en esta guía. Una vez que el código R se pruebe y esté listo, simplemente deberá cortarlo y pegarlo desde el editor de RStudio en un módulo [Ejecutar script R][execute-r-script] de Machine Learning Studio (clásico).  

Si no tiene instalado el lenguaje de programación R en su equipo de sobremesa, es recomendable que lo instale ahora. Encontrará descargas gratuitas del lenguaje R de código abierto en la red completa de archivos de R (CRAN) en [https://www.r-project.org/](https://www.r-project.org/). Hay descargas disponibles para Windows, Mac OS y Linux/UNIX. Elija el espejo más cercano a su ubicación y siga las instrucciones de descarga. Además, CRAN contiene una gran cantidad de paquetes de manipulación de datos y análisis de utilidad.

Si no está familiarizado con RStudio, descargue e instale la versión de escritorio. Encontrará descargas de RStudio para Windows, Mac OS y Linux/UNIX en http://www.rstudio.com/products/RStudio/. Siga las instrucciones proporcionadas para instalar RStudio en su equipo.  

Existe un tutorial de introducción a RStudio disponible en [Using the RStudio IDE](https://support.rstudio.com/hc/sections/200107586-Using-RStudio) (Uso del IDE de RStudio).

En la [Guía de la documentación de RStudio](#appendixa) más adelante, encontrará información adicional sobre el uso de RStudio.  

## <a name="get-data-in-and-out-of-the-execute-r-script-module"></a><a id="scriptmodule"></a>Obtención de datos dentro y fuera del módulo Ejecutar script de R

En esta sección, veremos cómo introducir y extraer datos del módulo [Ejecutar script R][execute-r-script]. Revisaremos cómo controlar los distintos tipos de datos leídos dentro y fuera del módulo [Ejecutar script R][execute-r-script].

El código completo de esta sección se encuentra en [MachineLearningSamples-Notebooks/studio-samples](https://github.com/Azure-Samples/MachineLearningSamples-Notebooks/tree/master/studio-samples).

### <a name="load-and-check-data"></a>Carga y comprobación de datos 

#### <a name="load-the-dataset"></a><a id="loading"></a>Carga del conjunto de datos

En primer lugar, cargaremos el archivo **csdairydata.csv** en Azure Machine Learning Studio (clásico).

1. Inicie su entorno de Azure Machine Learning Studio (clásico).
1. Seleccione **+ NUEVO** en la parte inferior izquierda de la pantalla y seleccione **Conjunto de datos**.
1. Seleccione **From Local File** (De archivo local) y luego **Examinar** para seleccionar el archivo.
1. Asegúrese de haber seleccionado la opción **Archivo CSV genérico con encabezado (.csv)** como el tipo de conjunto de datos.
1. Seleccione la marca de verificación.
1. Una vez cargado el conjunto de datos, debería ver el nuevo conjunto de datos cuando seleccione la pestaña **Conjuntos de datos**.  

#### <a name="create-an-experiment"></a>Creación de un experimento

Ahora que tenemos algunos datos en Machine Learning Studio (clásico), debemos crear un experimento para realizar el análisis.  

1. Seleccione **+ NUEVO** en la parte inferior izquierda, **Experiment** (Experimento) y luego **Blank Experiment** (Experimento en blanco).
1. Para asignar un nombre al experimento, seleccione y modifique el título **Experimento creado el...** en la parte superior de la página. Por ejemplo, cámbielo a **Análisis de productos lácteos de CA**.
1. A la izquierda de la página del experimento, expanda **Saved Datasets** (Conjuntos de datos guardados) y luego **My Datasets** (Mis conjuntos de datos). Debería ver **cadairydata.csv** que ha cargado anteriormente.
1. Arrastre y suelte el **conjunto de datos csdairydata.csv** en el experimento.
1. En el cuadro **Search experiment items** (Búsqueda de elementos de experimento) ubicado en la parte superior del panel izquierdo, escriba [Ejecutar script R][execute-r-script]. El módulo aparecerá en la lista de búsqueda.
1. Arrastre y suelte el módulo [Ejecutar script R][execute-r-script] en el pallet.  
1. Conecte la salida del **conjunto de datos csdairydata.csv** a la entrada ubicada más a la izquierda (**Dataset1**) del módulo [Ejecutar script R][execute-r-script].
1. **No olvide seleccionar "Guardar".**  

En este punto el experimento debería tener un aspecto similar al de la ilustración 3.

![Experimento Análisis de productos lácteos de CA con el conjunto de datos y el módulo Ejecutar script de R.](./media/r-quickstart/fig3.png)

*Ilustración 3. Experimento Análisis de productos lácteos de CA con el conjunto de datos y el módulo Ejecutar script de R.*

#### <a name="check-on-the-data"></a>Comprobación de los datos

Echemos un vistazo a los datos que se han cargado en nuestro experimento. En el experimento, seleccione el resultado del **conjunto de datos cadairydata.csv** y seleccione **Visualizar**. Debería ver algo parecido a lo que se muestra en la ilustración 4.  

![Resumen del conjunto de datos cadairydata.csv](./media/r-quickstart/fig4.png)

*Ilustración 4. Resumen del conjunto de datos cadairydata.csv.*

En esta vista hay una gran cantidad de información útil. Pueden verse las primeras filas de dicho conjunto de datos. Si se selecciona una columna, la sección de estadísticas muestra más información sobre ella. Por ejemplo, la fila de tipo de característica muestra los tipos de datos que Azure Machine Learning Studio (clásico) asignó a la columna. Echar un vistazo rápido de este modo es una buena práctica de comprobación antes de empezar a realizar cualquier trabajo más importante.

### <a name="first-r-script"></a>Primer script de R

A continuación, vamos a crear nuestro primer script de código R para experimentar con él en Azure Machine Learning Studio (clásico). Para ello, he creado y probado el siguiente script en RStudio.  

```r
## Only one of the following two lines should be used
## If running in Machine Learning Studio (classic), use the first line with maml.mapInputPort()
## If in RStudio, use the second line with read.csv()
cadairydata <- maml.mapInputPort(1)
# cadairydata  <- read.csv("cadairydata.csv", header = TRUE, stringsAsFactors = FALSE)
str(cadairydata)
pairs(~ Cotagecheese.Prod + Icecream.Prod + Milk.Prod + N.CA.Fat.Price, data = cadairydata)
## The following line should be executed only when running in
## Azure Machine Learning Studio (classic)
maml.mapOutputPort('cadairydata')
```

Ahora necesito transferir este script a Azure Machine Learning Studio (clásico). Basta con cortar y pegar. Sin embargo, en este caso, transferiré el script de código R mediante un archivo zip.

### <a name="data-input-to-the-execute-r-script-module"></a>Introducción de datos en el módulo de script de ejecución de código R

Echemos un vistazo a las entradas del módulo [Ejecutar script R][execute-r-script]. En este ejemplo se leerán los datos de productos lácteos de California en el módulo [Ejecutar script R][execute-r-script].  

Existen tres entradas posibles para el módulo [Ejecutar script R][execute-r-script]. Puede usar cualquiera de estas entradas o todas ellas, dependiendo de la aplicación. También se puede utilizar un script de código R que no tome ninguna entrada.  

Echemos un vistazo a cada una de estas entradas de izquierda a derecha. Para ver los nombres de cada una de las entradas, coloque el cursor sobre la entrada y lea la información sobre herramientas.  

#### <a name="script-bundle"></a>Paquete de script

La entrada del paquete de script permite pasar el contenido de un archivo zip al módulo [Ejecutar script R][execute-r-script]. Puede utilizar uno de los siguientes comandos para leer el contenido del archivo zip en su código R.

```r
source("src/yourfile.R") # Reads a zipped R script
load("src/yourData.rdata") # Reads a zipped R data file
```

> [!NOTE]
> Azure Machine Learning Studio (clásico) trata los archivos del archivo ZIP como si se encontrasen en el directorio src/, por lo que será preciso que agregue el nombre del directorio como prefijo a los nombres de archivo. Por ejemplo, si el archivo zip contiene los archivos `yourfile.R` y `yourData.rdata` en la raíz del archivo zip, los tratará como `src/yourfile.R` y `src/yourData.rdata` al utilizar `source` y `load`.

Ya se ha explicado el proceso de carga de conjuntos de datos en [Carga del conjunto de datos](#loading). Una vez creado y probado el script de código R que se muestra en la sección anterior, haga lo siguiente:

1. Guarde el script de código R en un archivo .R. Llamaré a mi archivo de script "simpleplot.R". Este es el contenido.

   ```r
   ## Only one of the following two lines should be used
   ## If running in Machine Learning Studio (classic), use the first line with maml.mapInputPort()
   ## If in RStudio, use the second line with read.csv()
   cadairydata <- maml.mapInputPort(1)
   # cadairydata  <- read.csv("cadairydata.csv", header = TRUE, stringsAsFactors = FALSE)
   str(cadairydata)
   pairs(~ Cotagecheese.Prod + Icecream.Prod + Milk.Prod + N.CA.Fat.Price, data = cadairydata)
   ## The following line should be executed only when running in
   ## Azure Machine Learning Studio (classic)
   maml.mapOutputPort('cadairydata')
   ```

1. Cree un archivo zip y copie el script en el archivo zip. En Windows, puede hacer clic con el botón derecho en el archivo, seleccionar **Enviar a** y luego **Carpeta comprimida**. Esto creará un nuevo archivo zip que contiene el archivo "simpleplot.R".

1. Agregue el archivo a los **conjuntos de datos** de Azure Machine Learning Studio (clásico) y especifique el tipo como **ZIP**. Ahora debería ver el archivo zip en los conjuntos de datos.

1. Arrastre y suelte el archivo zip de los **conjuntos de datos** al **lienzo de ML Studio (clásico)** .

1. Conecte la salida del icono de **datos del zip** a la entrada del **paquete de script** del módulo [Ejecutar código de R][execute-r-script].

1. Escriba la función `source()` con el nombre del archivo zip en la ventana de código del módulo [Ejecutar script R][execute-r-script]. En mi caso escribí `source("src/simpleplot.R")`.  

1. No olvide seleccionar **Guardar**.

Una vez completados estos pasos, el módulo [Ejecutar script R][execute-r-script] ejecutará el script R en el archivo zip cuando se ejecute el experimento. En este punto el experimento debería tener un aspecto similar al que se muestra en la ilustración 5.

![Experimento con el script de código R comprimido en un archivo zip](./media/r-quickstart/fig6.png)

*Ilustración 5. Experimento con el script de código R comprimido en un archivo zip.*

#### <a name="dataset1"></a>DataSet1

Puede pasar una tabla de datos rectangular al código R mediante la entrada Dataset1. En nuestro script sencillo, la función `maml.mapInputPort(1)` lee los datos del puerto 1. Estos datos se asignan a un nombre de variable de la trama de datos en el código. En nuestro script sencillo, la primera línea de código realiza la asignación.

```r
cadairydata <- maml.mapInputPort(1)
```

Para ejecutar el experimento, seleccione el botón **Ejecutar**. Cuando finalice la ejecución, seleccione el módulo [Ejecutar script R][execute-r-script] y, luego, seleccione **View output log** (Ver registro de salida) en la página de propiedades. Debería aparecer una nueva página en el explorador que muestre el contenido del archivo output.log. Al desplazarse hacia abajo, debería ver algo similar a lo siguiente.

```output
[ModuleOutput] InputDataStructure
[ModuleOutput]
[ModuleOutput] {
[ModuleOutput]  "InputName":Dataset1
[ModuleOutput]  "Rows":228
[ModuleOutput]  "Cols":9
[ModuleOutput]  "ColumnTypes":System.Int32,3,System.Double,5,System.String,1
[ModuleOutput] }
```

Más abajo en la página se especifica información más detallada sobre las columnas, con un aspecto similar al siguiente.

```output
[ModuleOutput] [1] "Loading variable port1..."
[ModuleOutput]
[ModuleOutput] 'data.frame':    228 obs. of  9 variables:
[ModuleOutput]
[ModuleOutput]  $ Column 0         : int  1 2 3 4 5 6 7 8 9 10 ...
[ModuleOutput]
[ModuleOutput]  $ Year.Month       : num  1995 1995 1995 1995 1995 ...
[ModuleOutput]
[ModuleOutput]  $ Month.Number     : int  1 2 3 4 5 6 7 8 9 10 ...
[ModuleOutput]
[ModuleOutput]  $ Year             : int  1995 1995 1995 1995 1995 1995 1995 1995 1995 1995 ...
[ModuleOutput]
[ModuleOutput]  $ Month            : chr  "Jan" "Feb" "Mar" "Apr" ...
[ModuleOutput]
[ModuleOutput]  $ Cotagecheese.Prod: num  4.37 3.69 4.54 4.28 4.47 ...
[ModuleOutput]
[ModuleOutput]  $ Icecream.Prod    : num  51.6 56.1 68.5 65.7 73.7 ...
[ModuleOutput]
[ModuleOutput]  $ Milk.Prod        : num  2.11 1.93 2.16 2.13 2.23 ...
[ModuleOutput]
[ModuleOutput]  $ N.CA.Fat.Price   : num  0.98 0.892 0.892 0.897 0.897 ...
```

Estos resultados son los esperados e incluyen 228 observaciones y 9 columnas en el marco de datos. Pueden verse los nombres de columna, el tipo de datos de código R y un ejemplo de cada columna.

> [!NOTE]
> Esta misma salida impresa está disponible desde el resultado del dispositivo R del módulo [Ejecutar script R][execute-r-script]. Abordaremos las salidas del módulo [Ejecutar script R][execute-r-script] en la sección siguiente.  

#### <a name="dataset2"></a>Dataset2

El comportamiento de la entrada Dataset2 es idéntico al de la entrada Dataset1. Con esta entrada, se puede pasar una segunda tabla rectangular de datos al código R. La función `maml.mapInputPort(2)`con el argumento 2, se utiliza para pasar estos datos.  

### <a name="execute-r-script-outputs"></a>Ejecución de resultados de script de código R

#### <a name="output-a-dataframe"></a>Generación de tramas de datos

Es posible generar el contenido de un marco de datos R como tabla rectangular a través del puerto Dataset1 de resultado mediante la función `maml.mapOutputPort()` . En nuestro script de código R sencillo, esto se realiza con la siguiente línea.

```r
maml.mapOutputPort('cadairydata')
```

Después de ejecutar el experimento, seleccione el puerto de salida Dataset1 y, a continuación, seleccione **Visualizar**. Debería ver algo parecido a lo que se muestra en la ilustración 6.

![Visualización de la salida de los datos de productos lácteos de California](./media/r-quickstart/fig7.png)

*Ilustración 6. Visualización de la salida de los datos de productos lácteos de California.*

Este resultado es idéntico a la entrada, justo como se esperaba.  

### <a name="r-device-output"></a>Resultados del dispositivo R

La salida del dispositivo del módulo [Ejecutar script R][execute-r-script] contiene la salida de mensajes y gráficos. Los mensajes de error y de resultados estándar de R se envían al puerto de salida del dispositivo R.  

Para ver la salida del dispositivo R, seleccione el puerto y, a continuación, **Visualizar**. En la ilustración 7 puede verse el resultado estándar y el error estándar del script de código R.

![Salida estándar y errores estándar desde el puerto del dispositivo R](./media/r-quickstart/fig8.png)

*Ilustración 7. Salida estándar y errores estándar desde el puerto del dispositivo R.*

Si nos desplazamos hacia abajo, se puede ver el resultado de gráficos de nuestro script de código R en la ilustración 8.  

![Salida gráfica desde el puerto de dispositivo R](./media/r-quickstart/fig9.png)

*Ilustración 8. Salida gráfica desde el puerto de dispositivo R.*  

## <a name="data-filtering-and-transformation"></a><a id="filtering"></a>Transformación y filtrado de datos

En esta sección se realizarán operaciones básicas de filtrado y transformación de datos de productos lácteos de California. Al final de esta sección, dispondrá de datos en formato adecuado para la creación de un modelo analítico.  

En esta sección se realizarán varias tareas de transformación y limpieza de datos comunes: transformación de tipos, filtrado de tramas de datos, adición de nuevas columnas de cálculo y transformaciones de valor. Esta información general le ayudará a tratar con muchas de las variaciones que encontrará cuando se enfrente a problemas reales.

El código R completo de esta sección está disponible en [MachineLearningSamples-Notebooks/studio-samples](https://github.com/Azure-Samples/MachineLearningSamples-Notebooks/tree/master/studio-samples).

### <a name="type-transformations"></a>Transformaciones de tipo

Ahora que podemos leer los datos de productos lácteos de California en código R en el módulo [Ejecutar script R][execute-r-script], es necesario asegurarse de que los datos de las columnas tienen el tipo y el formato deseados.  

R es un lenguaje con tipos dinámicos, lo que significa que los tipos de datos se convierten de un tipo a otro según sea necesario. Los tipos de datos atómicos en R incluyen caracteres, números y operaciones lógicas. El tipo de factor se utiliza para almacenar de manera compacta datos categóricos. Encontrará mucha más información sobre los tipos de datos en las referencias de las [Lecturas adicionales](#appendixb).

Cuando se lean datos tabulares en código R desde un origen externo, se recomienda comprobar siempre los tipos resultantes de las columnas. Es posible que desee una columna de caracteres de tipo, pero en muchos casos esto se mostrará como factor o viceversa. En otros casos, la columna que piensa que debe ser numérica se mostrará con datos de caracteres como, por ejemplo, '1,23' en lugar de 1,23 como número de punto flotante.  

Afortunadamente, es fácil realizar la conversión de un tipo a otro, siempre que sea posible la asignación. Por ejemplo, no se puede convertir 'Nevada' a un valor numérico, pero se puede convertir en un factor (variable de categoría). Como otro ejemplo, puede convertir el valor numérico 1 en un carácter '1' o en un factor.  

La sintaxis para cualquiera de estas conversiones es simple: `as.datatype()`. Estas funciones de conversión de tipos incluyen lo siguiente.

* `as.numeric()`
* `as.character()`
* `as.logical()`
* `as.factor()`

Viendo los tipos de datos de las columnas introducidas en la sección anterior: todas las columnas son de tipo numérico, excepto la columna con la etiqueta 'Month', que es de carácter de tipo. A continuación, vamos a convertirla en un factor y a probar los resultados.  

He eliminado la línea que creaba la matriz de trazado de dispersión y he agregado una línea para convertir la columna 'Month' en un factor. En mi experimento solo cortaré y pegaré el código R en la ventana de código del módulo [Ejecutar script R][execute-r-script]. También puede actualizar el archivo ZIP y cargarlo en Azure Machine Learning Studio (clásico); aunque esto implica varios pasos.  

```r
## Only one of the following two lines should be used
## If running in Machine Learning Studio (classic), use the first line with maml.mapInputPort()
## If in RStudio, use the second line with read.csv()
cadairydata <- maml.mapInputPort(1)
# cadairydata  <- read.csv("cadairydata.csv", header = TRUE, stringsAsFactors = FALSE)
## Ensure the coding is consistent and convert column to a factor
cadairydata$Month <- as.factor(cadairydata$Month)
str(cadairydata) # Check the result
## The following line should be executed only when running in
## Azure Machine Learning Studio (classic)
maml.mapOutputPort('cadairydata')
```

Vamos a ejecutar este código y a examinar el registro de salida del script de R. Los datos pertinentes del registro se muestran en la ilustración 9.

```output
[ModuleOutput] [1] "Loading variable port1..."
[ModuleOutput] 
[ModuleOutput] 'data.frame':    228 obs. of  9 variables:
[ModuleOutput] 
[ModuleOutput]  $ Column 0         : int  1 2 3 4 5 6 7 8 9 10 ...
[ModuleOutput] 
[ModuleOutput]  $ Year.Month       : num  1995 1995 1995 1995 1995 ...
[ModuleOutput] 
[ModuleOutput]  $ Month.Number     : int  1 2 3 4 5 6 7 8 9 10 ...
[ModuleOutput] 
[ModuleOutput]  $ Year             : int  1995 1995 1995 1995 1995 1995 1995 1995 1995 1995 ...
[ModuleOutput] 
[ModuleOutput]  $ Month            : Factor w/ 14 levels "Apr","April",..: 6 5 9 1 11 8 7 3 14 13 ...
[ModuleOutput] 
[ModuleOutput]  $ Cotagecheese.Prod: num  4.37 3.69 4.54 4.28 4.47 ...
[ModuleOutput] 
[ModuleOutput]  $ Icecream.Prod    : num  51.6 56.1 68.5 65.7 73.7 ...
[ModuleOutput] 
[ModuleOutput]  $ Milk.Prod        : num  2.11 1.93 2.16 2.13 2.23 ...
[ModuleOutput] 
[ModuleOutput]  $ N.CA.Fat.Price   : num  0.98 0.892 0.892 0.897 0.897 ...
[ModuleOutput] 
[ModuleOutput] [1] "Saving variable  cadairydata  ..."
[ModuleOutput] 
[ModuleOutput] [1] "Saving the following item(s):  .maml.oport1"
```

*Ilustración 9. Resumen del marco de datos con una variable de factor.*

El tipo de Month debe ser ahora '**Factor con 14 niveles**'. Esto es un problema, puesto que un año solo tiene 12 meses. También puede realizar comprobaciones para ver que el tipo de **Visualizar** del puerto Result Dataset (Conjunto de datos de resultados) es **Categorical**.

El problema es que la columna 'Month' no se ha codificado de forma sistemática. En algunos casos, un mes se denomina Abril y en otros se abrevia como Abr. Podemos resolver este problema recortando la cadena a 3 caracteres. La línea de código tendrá el aspecto siguiente:

```r
## Ensure the coding is consistent and convert column to a factor
cadairydata$Month <- as.factor(substr(cadairydata$Month, 1, 3))
```

Vuelva a ejecutar el experimento y vea el registro de salida. Los resultados esperados se muestran en la ilustración 10.  

```output
[ModuleOutput] [1] "Loading variable port1..."
[ModuleOutput] 
[ModuleOutput] 'data.frame':    228 obs. of  9 variables:
[ModuleOutput] 
[ModuleOutput]  $ Column 0         : int  1 2 3 4 5 6 7 8 9 10 ...
[ModuleOutput] 
[ModuleOutput]  $ Year.Month       : num  1995 1995 1995 1995 1995 ...
[ModuleOutput] 
[ModuleOutput]  $ Month.Number     : int  1 2 3 4 5 6 7 8 9 10 ...
[ModuleOutput] 
[ModuleOutput]  $ Year             : int  1995 1995 1995 1995 1995 1995 1995 1995 1995 1995 ...
[ModuleOutput] 
[ModuleOutput]  $ Month            : Factor w/ 12 levels "Apr","Aug","Dec",..: 5 4 8 1 9 7 6 2 12 11 ...
[ModuleOutput] 
[ModuleOutput]  $ Cotagecheese.Prod: num  4.37 3.69 4.54 4.28 4.47 ...
[ModuleOutput] 
[ModuleOutput]  $ Icecream.Prod    : num  51.6 56.1 68.5 65.7 73.7 ...
[ModuleOutput] 
[ModuleOutput]  $ Milk.Prod        : num  2.11 1.93 2.16 2.13 2.23 ...
[ModuleOutput] 
[ModuleOutput]  $ N.CA.Fat.Price   : num  0.98 0.892 0.892 0.897 0.897 ...
[ModuleOutput] 
[ModuleOutput] [1] "Saving variable  cadairydata  ..."
[ModuleOutput] 
[ModuleOutput] [1] "Saving the following item(s):  .maml.oport1"
```

*Ilustración 10. Resumen del marco de datos con el número correcto de niveles de factor.*

Nuestra variable factor tiene ahora los 12 niveles deseados.

### <a name="basic-data-frame-filtering"></a>Filtrado del marco de datos básico

Las tramas de datos R incluyen capacidades de filtrado eficaces. Es posible obtener subconjuntos de los conjuntos de datos mediante el uso de filtros lógicos en filas o columnas. En muchos casos, serán necesarios criterios de filtro complejos. Las referencias de las [Lecturas adicionales](#appendixb) contienen ejemplos extensos del filtrado de tramas de datos.  

En nuestro conjunto de datos, es necesario crear un bit de filtrado. Si observamos las columnas de la trama de datos cadairydata, podemos ver que hay dos columnas innecesarias. La primera columna contiene solo un número de fila, que no es muy útil. La segunda columna, Year.Month, contiene información redundante. Estas dos columnas se pueden excluir fácilmente mediante el código R siguiente.

> [!NOTE]
> De ahora en adelante en esta sección, solo mostraré el código adicional que voy a agregar en el módulo [Ejecutar script R][execute-r-script]. Voy a agregar cada nueva línea **antes** de la función `str()`. Esta función se utiliza para comprobar los resultados en Azure Machine Learning Studio (clásico).

Voy a agregar la línea siguiente a mi código R en el módulo [Ejecutar script R][execute-r-script].

```r
# Remove two columns we do not need
cadairydata <- cadairydata[, c(-1, -2)]
```

Ejecute este código en su experimento y compruebe el resultado del registro de salida. Estos resultados se muestran en la ilustración 11.

```output
[ModuleOutput] [1] "Loading variable port1..."
[ModuleOutput] 
[ModuleOutput] 'data.frame':    228 obs. of  7 variables:
[ModuleOutput] 
[ModuleOutput]  $ Month.Number     : int  1 2 3 4 5 6 7 8 9 10 ...
[ModuleOutput] 
[ModuleOutput]  $ Year             : int  1995 1995 1995 1995 1995 1995 1995 1995 1995 1995 ...
[ModuleOutput] 
[ModuleOutput]  $ Month            : Factor w/ 12 levels "Apr","Aug","Dec",..: 5 4 8 1 9 7 6 2 12 11 ...
[ModuleOutput] 
[ModuleOutput]  $ Cotagecheese.Prod: num  4.37 3.69 4.54 4.28 4.47 ...
[ModuleOutput] 
[ModuleOutput]  $ Icecream.Prod    : num  51.6 56.1 68.5 65.7 73.7 ...
[ModuleOutput] 
[ModuleOutput]  $ Milk.Prod        : num  2.11 1.93 2.16 2.13 2.23 ...
[ModuleOutput] 
[ModuleOutput]  $ N.CA.Fat.Price   : num  0.98 0.892 0.892 0.897 0.897 ...
[ModuleOutput] 
[ModuleOutput] [1] "Saving variable  cadairydata  ..."
[ModuleOutput] 
[ModuleOutput] [1] "Saving the following item(s):  .maml.oport1"
```

*Ilustración 11. Resumen de la trama de datos con dos columnas quitadas.*

¡Buenas noticias! Se obtienen los resultados esperados.

### <a name="add-a-new-column"></a>Adición de una columna nueva

Para crear modelos de serie temporal será conveniente tener una columna que contenga los meses desde el inicio de la serie temporal. Por ello, crearemos una nueva columna 'Month.Count'.

Para facilitar la organización del código, crearemos la primera función simple, `num.month()`. A continuación, aplicaremos esta función para crear una nueva columna en la trama de datos. El nuevo código es el siguiente:

```r
## Create a new column with the month count
## Function to find the number of months from the first
## month of the time series
num.month <- function(Year, Month) {
  ## Find the starting year
  min.year  <- min(Year)

  ## Compute the number of months from the start of the time series
  12 * (Year - min.year) + Month - 1
}

## Compute the new column for the dataframe
cadairydata$Month.Count <- num.month(cadairydata$Year, cadairydata$Month.Number)
```

Ahora ejecute el experimento actualizado y use el registro de salida para ver los resultados. Estos resultados se muestran en la ilustración 12.

```output
[ModuleOutput] [1] "Loading variable port1..."
[ModuleOutput] 
[ModuleOutput] 'data.frame':    228 obs. of  8 variables:
[ModuleOutput] 
[ModuleOutput]  $ Month.Number     : int  1 2 3 4 5 6 7 8 9 10 ...
[ModuleOutput] 
[ModuleOutput]  $ Year             : int  1995 1995 1995 1995 1995 1995 1995 1995 1995 1995 ...
[ModuleOutput] 
[ModuleOutput]  $ Month            : Factor w/ 12 levels "Apr","Aug","Dec",..: 5 4 8 1 9 7 6 2 12 11 ...
[ModuleOutput] 
[ModuleOutput]  $ Cotagecheese.Prod: num  4.37 3.69 4.54 4.28 4.47 ...
[ModuleOutput] 
[ModuleOutput]  $ Icecream.Prod    : num  51.6 56.1 68.5 65.7 73.7 ...
[ModuleOutput] 
[ModuleOutput]  $ Milk.Prod        : num  2.11 1.93 2.16 2.13 2.23 ...
[ModuleOutput] 
[ModuleOutput]  $ N.CA.Fat.Price   : num  0.98 0.892 0.892 0.897 0.897 ...
[ModuleOutput] 
[ModuleOutput]  $ Month.Count      : num  0 1 2 3 4 5 6 7 8 9 ...
[ModuleOutput] 
[ModuleOutput] [1] "Saving variable  cadairydata  ..."
[ModuleOutput] 
[ModuleOutput] [1] "Saving the following item(s):  .maml.oport1"
```

*Ilustración 12. Resumen de la trama de datos con la columna adicional.*

Parece que todo funciona correctamente. Tenemos la nueva columna con los valores esperados en nuestra trama de datos.

### <a name="value-transformations"></a>Transformaciones de valor

En esta sección se realizarán algunas transformaciones simples en los valores de algunas de las columnas de nuestra trama de datos. El lenguaje R admite las transformaciones de valores casi arbitrarias. Las referencias de las [Lecturas adicionales](#appendixb) contienen ejemplos extensos.

Si examinamos los valores de los resúmenes de nuestra trama de datos deberíamos ver algo extraño. ¿Se produce más de helado que leche en California? Por supuesto que no. Esto no tiene sentido, por desgracia para los amantes del helado. Las unidades son diferentes. El precio se especifica en unidades de libras, la leche se especifica en unidades de millones de libras y el helado en unidades de mil galones. Asimismo, el requesón se proporciona en unidades de miles de libras. Suponiendo que el peso del helado sea de 6,5 libras por galón, podemos realizar fácilmente la multiplicación para convertir estos valores para que se expresen en las mismas unidades de miles de libras.

Para nuestro modelo de pronóstico, utilizaremos un modelo de multiplicación de tendencia y de ajuste de temporada de estos datos. Una transformación de registro permite usar un modelo lineal, lo que simplifica este proceso. Podemos aplicar la transformación de registro en la misma función en la que se aplica el multiplicador.

En el código siguiente, voy a definir una nueva función, `log.transform()`, y la aplicaré a las filas que contienen los valores numéricos. La función R `Map()` se utiliza para aplicar la función `log.transform()` a las columnas seleccionadas de la trama de datos. `Map()` es similar a `apply()`, pero permite más de una lista de argumentos de la función. Tenga en cuenta que una lista de multiplicadores proporciona el segundo argumento de la función `log.transform()` . La función `na.omit()` se utiliza para realizar limpieza y garantizar que no haya valores no definidos o que falten en la trama de datos.

```r
log.transform <- function(invec, multiplier = 1) {
  ## Function for the transformation, which is the log
  ## of the input value times a multiplier

  warningmessages <- c("ERROR: Non-numeric argument encountered in function log.transform",
                       "ERROR: Arguments to function log.transform must be greate than zero",
                       "ERROR: Aggurment multiplier to funcition log.transform must be a scaler",
                       "ERROR: Invalid time seies value encountered in function log.transform"
                       )

  ## Check the input arguments
  if(!is.numeric(invec) | !is.numeric(multiplier)) {warning(warningmessages[1]); return(NA)}  
  if(any(invec < 0.0) | any(multiplier < 0.0)) {warning(warningmessages[2]); return(NA)}
  if(length(multiplier) != 1) {{warning(warningmessages[3]); return(NA)}}

  ## Wrap the multiplication in tryCatch
  ## If there is an exception, print the warningmessage to
  ## standard error and return NA
  tryCatch(log(multiplier * invec),
           error = function(e){warning(warningmessages[4]); NA})
}


## Apply the transformation function to the 4 columns
## of the dataframe with production data
multipliers  <- list(1.0, 6.5, 1000.0, 1000.0)
cadairydata[, 4:7] <- Map(log.transform, cadairydata[, 4:7], multipliers)

## Get rid of any rows with NA values
cadairydata <- na.omit(cadairydata)  
```

La función `log.transform()` realiza bastantes acciones. La mayor parte de este código comprueba si hay posibles problemas con los argumentos o aborda las excepciones que pueden producirse durante los cálculos. En realidad, solo unas pocas líneas de este código realizan los cálculos.

El objetivo de la programación defensiva es evitar que y¡un error en una función impida que el procesamiento continúe. Un error repentino en un análisis cuya ejecución lleva mucho tiempo puede resultar muy frustrante para los usuarios. Para evitar esta situación, se deben elegir valores devueltos predeterminados para limitar los daños en el procesamiento de nivel inferior. También se genera un mensaje para avisar a los usuarios de que se ha producido un error.

Si no está familiarizado con la programación defensiva en R, es posible que este código pueda parecerle complejo. Por ello, lo guiaré a través de los pasos principales:

1. Se define un vector de cuatro mensajes. Estos mensajes se usan para comunicar información acerca de algunos de los posibles errores y excepciones que pueden producirse con este código.
2. Devolveré un valor de NA para cada caso. Hay muchas otras posibilidades que pueden tener menos efectos secundarios. Podría devolver un vector de ceros o el vector de entrada original, por ejemplo.
3. Las comprobaciones se ejecutan en los argumentos de la función. En cada caso, si se detecta un error, se devuelve un valor predeterminado y la función `warning()` produce un mensaje. Utilizaremos la función `warning()` en lugar de `stop()`, ya que esta finaliza la ejecución, que es justo lo que estamos tratando de evitar. He escrito este código en un estilo de procedimiento, ya que, en este caso, un enfoque funcional sería demasiado complejo.
4. Los cálculos de registros se encapsulan en `tryCatch()` para que las excepciones no detengan el procesamiento de forma abrupta. Sin `tryCatch()` , la mayoría de los errores que generan las funciones de R dan como resultado una señal de detención, que hace justamente eso.

Ejecute este código R en el experimento y preste atención al resultado impreso en el archivo output.log. Ahora verá los valores transformados de las cuatro columnas en el registro, tal como se muestra en la ilustración 13.

```output
[ModuleOutput] [1] "Loading variable port1..."
[ModuleOutput] 
[ModuleOutput] 'data.frame':    228 obs. of  8 variables:
[ModuleOutput] 
[ModuleOutput]  $ Month.Number     : int  1 2 3 4 5 6 7 8 9 10 ...
[ModuleOutput] 
[ModuleOutput]  $ Year             : int  1995 1995 1995 1995 1995 1995 1995 1995 1995 1995 ...
[ModuleOutput] 
[ModuleOutput]  $ Month            : Factor w/ 12 levels "Apr","Aug","Dec",..: 5 4 8 1 9 7 6 2 12 11 ...
[ModuleOutput] 
[ModuleOutput]  $ Cotagecheese.Prod: num  1.47 1.31 1.51 1.45 1.5 ...
[ModuleOutput] 
[ModuleOutput]  $ Icecream.Prod    : num  5.82 5.9 6.1 6.06 6.17 ...
[ModuleOutput] 
[ModuleOutput]  $ Milk.Prod        : num  7.66 7.57 7.68 7.66 7.71 ...
[ModuleOutput] 
[ModuleOutput]  $ N.CA.Fat.Price   : num  6.89 6.79 6.79 6.8 6.8 ...
[ModuleOutput] 
[ModuleOutput]  $ Month.Count      : num  0 1 2 3 4 5 6 7 8 9 ...
[ModuleOutput] 
[ModuleOutput] [1] "Saving variable  cadairydata  ..."
[ModuleOutput] 
[ModuleOutput] [1] "Saving the following item(s):  .maml.oport1"
```

*Ilustración 13. Resumen de los valores transformados en la trama de datos.*

Como puede, observar, los valores se han transformado. La producción de leche ahora supera con creces la producción de otros productos lácteos, recordando que ahora estamos examinando una escala logarítmica.

En este momento nuestros datos se limpian y estamos preparados el modelado. Según el resumen de visualización de la salida del conjunto de datos de resultado del módulo [Ejecutar script R][execute-r-script], verá que la columna "Month" es "Categorical" con 12 valores únicos, nuevamente tal como deseaba.

## <a name="time-series-objects-and-correlation-analysis"></a><a id="timeseries"></a>Análisis de correlación y objetos de series temporales

En esta sección se explorarán objetos básicos de series temporales R y se analizarán las correlaciones entre algunas de las variables. Nuestro objetivo es producir una trama de datos que contiene la información de correlación en pares en varios intervalos de salida.

El código R completo de esta sección se encuentra en [MachineLearningSamples-Notebooks/studio-samples](https://github.com/Azure-Samples/MachineLearningSamples-Notebooks/tree/master/studio-samples).

### <a name="time-series-objects-in-r"></a>Objetos de series de temporales de R

Como ya se ha mencionado, las series temporales son series de valores de datos indexados por tiempo. Los objetos de series temporales de R se utilizan para crear y administrar el índice de tiempo. La utilización de objetos de series temporales ofrece varias ventajas. Los objetos de series temporales le liberan de los numerosos detalles de la administración de los valores de índice de series temporales que se encapsulan en el objeto. Además, los objetos de series temporales permiten utilizar los distintos métodos de la serie temporal para realizar tareas de trazado, impresión, modelado, etc.

La clase de serie temporal POSIXct es la que se utiliza con más frecuencia y es relativamente sencilla. Esta serie temporal mide el tiempo desde el inicio del periodo, el 1 de enero de 1970. En este ejemplo se utilizarán los objetos de serie temporal POSIXct. Otras clases de objeto de serie temporal R ampliamente utilizadas son las clases zoo y xts, así como las series temporales extensibles.

### <a name="time-series-object-example"></a>Ejemplo de objeto de serie temporal

Comencemos con el ejemplo. Arrastre y suelte un **nuevo** módulo [Ejecutar script R][execute-r-script] en el experimento. Conecte el puerto de salida de Dataset1 de resultados del módulo [Ejecutar script R][execute-r-script] al puerto de entrada de Dataset1 del nuevo módulo [Ejecutar script R][execute-r-script].

Como hicimos en los primeros ejemplos, a medida que progresamos en el ejemplo, solo mostraré en algunos puntos las líneas adicionales incrementales de código R en cada paso.  

#### <a name="reading-the-dataframe"></a>Lectura de la trama de datos

Como primer paso, vamos leer una trama de datos y nos aseguraremos de que se obtienen los resultados esperados. El código siguiente debe funcionar.

```r
# Comment the following if using RStudio
cadairydata <- maml.mapInputPort(1)
str(cadairydata) # Check the results
```

Ahora, vamos a ejecutar el experimento. El registro de la nueva forma de ejecutar script de R debería parecerse a la ilustración 14.

```output
[ModuleOutput] [1] "Loading variable port1..."
[ModuleOutput] 
[ModuleOutput] 'data.frame':    228 obs. of  8 variables:
[ModuleOutput] 
[ModuleOutput]  $ Month.Number     : int  1 2 3 4 5 6 7 8 9 10 ...
[ModuleOutput] 
[ModuleOutput]  $ Year             : int  1995 1995 1995 1995 1995 1995 1995 1995 1995 1995 ...
[ModuleOutput] 
[ModuleOutput]  $ Month            : Factor w/ 12 levels "Apr","Aug","Dec",..: 5 4 8 1 9 7 6 2 12 11 ...
[ModuleOutput] 
[ModuleOutput]  $ Cotagecheese.Prod: num  1.47 1.31 1.51 1.45 1.5 ...
[ModuleOutput] 
[ModuleOutput]  $ Icecream.Prod    : num  5.82 5.9 6.1 6.06 6.17 ...
[ModuleOutput] 
[ModuleOutput]  $ Milk.Prod        : num  7.66 7.57 7.68 7.66 7.71 ...
[ModuleOutput] 
[ModuleOutput]  $ N.CA.Fat.Price   : num  6.89 6.79 6.79 6.8 6.8 ...
[ModuleOutput] 
[ModuleOutput]  $ Month.Count      : num  0 1 2 3 4 5 6 7 8 9 ...
```

*Ilustración 14. Resumen de la trama de datos en el módulo Ejecutar script de R.*

Estos datos tienen el tipo y el formato esperados. Tenga en cuenta que la columna 'Month' es de factor de tipo y tiene el número de niveles esperado.

#### <a name="creating-a-time-series-object"></a>Creación de un objeto de serie temporal

Necesitamos agregar un objeto de serie temporal a nuestra trama de datos. Reemplace el código actual por el siguiente, que agrega una nueva columna de la clase POSIXct.

```r
# Comment the following if using RStudio
cadairydata <- maml.mapInputPort(1)

## Create a new column as a POSIXct object
Sys.setenv(TZ = "PST8PDT")
cadairydata$Time <- as.POSIXct(strptime(paste(as.character(cadairydata$Year), "-", as.character(cadairydata$Month.Number), "-01 00:00:00", sep = ""), "%Y-%m-%d %H:%M:%S"))

str(cadairydata) # Check the results
```

Ahora, compruebe el registro. Debería ser similar al que se muestra en la ilustración 15.

```output
[ModuleOutput] [1] "Loading variable port1..."
[ModuleOutput] 
[ModuleOutput] 'data.frame':    228 obs. of  9 variables:
[ModuleOutput] 
[ModuleOutput]  $ Month.Number     : int  1 2 3 4 5 6 7 8 9 10 ...
[ModuleOutput] 
[ModuleOutput]  $ Year             : int  1995 1995 1995 1995 1995 1995 1995 1995 1995 1995 ...
[ModuleOutput] 
[ModuleOutput]  $ Month            : Factor w/ 12 levels "Apr","Aug","Dec",..: 5 4 8 1 9 7 6 2 12 11 ...
[ModuleOutput] 
[ModuleOutput]  $ Cotagecheese.Prod: num  1.47 1.31 1.51 1.45 1.5 ...
[ModuleOutput] 
[ModuleOutput]  $ Icecream.Prod    : num  5.82 5.9 6.1 6.06 6.17 ...
[ModuleOutput] 
[ModuleOutput]  $ Milk.Prod        : num  7.66 7.57 7.68 7.66 7.71 ...
[ModuleOutput] 
[ModuleOutput]  $ N.CA.Fat.Price   : num  6.89 6.79 6.79 6.8 6.8 ...
[ModuleOutput] 
[ModuleOutput]  $ Month.Count      : num  0 1 2 3 4 5 6 7 8 9 ...
[ModuleOutput] 
[ModuleOutput]  $ Time             : POSIXct, format: "1995-01-01" "1995-02-01" ...
```

*Ilustración 15. Resumen de la trama de datos con un objeto de serie temporal.*

Podemos ver en el resumen que la nueva columna es en realidad de la clase POSIXct.

### <a name="exploring-and-transforming-the-data"></a>Exploración y transformación de los datos

Exploremos algunas de las variables de este conjunto de datos. Una matriz de trazado de dispersión es una buena manera de generar un vistazo rápido. Voy a reemplazar la función `str()` del código R anterior por la siguiente línea.

```r
pairs(~ Cotagecheese.Prod + Icecream.Prod + Milk.Prod + N.CA.Fat.Price, data = cadairydata, main = "Pairwise Scatterplots of dairy time series")
```

Ejecute este código y observe qué sucede. El trazado producido en el puerto del dispositivo R debería parecerse al que se muestra en la ilustración 16.

![Matriz de trazado de dispersión de las variables seleccionadas](./media/r-quickstart/fig17.png)

*Ilustración 16. Matriz de trazado de dispersión de las variables seleccionadas.*

Como se puede observar, hay una estructura extraña en las relaciones entre estas variables. Quizás esto se debe a las tendencias en los datos y al hecho de que no hemos estandarizado las variables.

### <a name="correlation-analysis"></a>Análisis de correlación

Para realizar el análisis de correlación, necesitamos anular las tendencias y estandarizar las variables. Para ello, basta con usar la función `scale()` de R que permite centrar y escalar las variables. Esta función también puede ejecutarse más rápido. Sin embargo, voy a mostrar un ejemplo de programación defensiva en R.

La función `ts.detrend()` que se muestra a continuación realiza estas dos operaciones. Las siguientes dos líneas de código permiten anular las tendencias de datos y estandarizar los valores.

```r
ts.detrend <- function(ts, Time, min.length = 3){
  ## Function to de-trend and standardize a time series

  ## Define some messages if they are NULL  
  messages <- c('ERROR: ts.detrend requires arguments ts and Time to have the same length',
                'ERROR: ts.detrend requires argument ts to be of type numeric',
                paste('WARNING: ts.detrend has encountered a time series with length less than', as.character(min.length)),
                'ERROR: ts.detrend has encountered a Time argument not of class POSIXct',
                'ERROR: Detrend regression has failed in ts.detrend',
                'ERROR: Exception occurred in ts.detrend while standardizing time series in function ts.detrend'
  )
  # Create a vector of zeros to return as a default in some cases
  zerovec  <- rep(length(ts), 0.0)

  # The input arguments are not of the same length, return ts and quit
  if(length(Time) != length(ts)) {warning(messages[1]); return(ts)}

  # If the ts is not numeric, just return a zero vector and quit
  if(!is.numeric(ts)) {warning(messages[2]); return(zerovec)}

  # If the ts is too short, just return it and quit
  if((ts.length <- length(ts)) < min.length) {warning(messages[3]); return(ts)}

  ## Check that the Time variable is of class POSIXct
  if(class(cadairydata$Time)[[1]] != "POSIXct") {warning(messages[4]); return(ts)}

  ## De-trend the time series by using a linear model
  ts.frame  <- data.frame(ts = ts, Time = Time)
  tryCatch({ts <- ts - fitted(lm(ts ~ Time, data = ts.frame))},
           error = function(e){warning(messages[5]); zerovec})

  tryCatch( {stdev <- sqrt(sum((ts - mean(ts))^2))/(ts.length - 1)
             ts <- ts/stdev},
            error = function(e){warning(messages[6]); zerovec})

  ts
}  
## Apply the detrend.ts function to the variables of interest
df.detrend <- data.frame(lapply(cadairydata[, 4:7], ts.detrend, cadairydata$Time))

## Plot the results to look at the relationships
pairs(~ Cotagecheese.Prod + Icecream.Prod + Milk.Prod + N.CA.Fat.Price, data = df.detrend, main = "Pairwise Scatterplots of detrended standardized time series")
```

La función `ts.detrend()` realiza bastantes acciones. La mayor parte de este código comprueba si hay posibles problemas con los argumentos o aborda las excepciones que pueden producirse durante los cálculos. En realidad, solo unas pocas líneas de este código realizan los cálculos.

Ya hemos analizado un ejemplo de programación defensiva en Transformaciones de valor. Ambos bloques de cálculo se incluyen en `tryCatch()`. Para algunos errores, es recomendable devolver el vector de entrada original, mientras que en otros casos es preferible devolver un vector de ceros.  

Tenga en cuenta que la regresión lineal usada para anular las tendencias es una regresión de la serie temporal. La variable de predicción es un objeto de la serie temporal.  

Una vez definida la función `ts.detrend()` , se aplica a las variables de interés en nuestra trama de datos. Es necesario forzar la lista resultante creada con la función `lapply()` en los datos de la trama de datos mediante la función `as.data.frame()`. Debido a los aspectos defensivos de la función `ts.detrend()`, si no se procesa alguna de las variables, no se impedirá el correcto procesamiento de las demás.  

La última línea de código crea un trazado de dispersión en pares. Después de ejecutar el código R, se mostrarán los resultados del trazado de dispersión en la ilustración 17.

![Trazado de dispersión en pares de series temporales estandarizadas y con las tendencias anuladas](./media/r-quickstart/fig18.png)

*Ilustración 17. Trazado de dispersión en pares de series temporales estandarizadas y con las tendencias anuladas.*

Puede comparar estos resultados con los que se muestran en la ilustración 16. Con la tendencia quitad y las variables estandarizadas, podemos ver que la estructura en las relaciones entre estas variables es menor.

El código necesario para calcular las correlaciones como objetos ccf R es el siguiente.

```r
## A function to compute pairwise correlations from a
## list of time series value vectors
pair.cor <- function(pair.ind, ts.list, lag.max = 1, plot = FALSE){
  ccf(ts.list[[pair.ind[1]]], ts.list[[pair.ind[2]]], lag.max = lag.max, plot = plot)
}

## A list of the pairwise indices
corpairs <- list(c(1,2), c(1,3), c(1,4), c(2,3), c(2,4), c(3,4))

## Compute the list of ccf objects
cadairycorrelations <- lapply(corpairs, pair.cor, df.detrend)  

cadairycorrelations
```

La ejecución de este código genera el registro mostrado en la ilustración 18.

```output
[ModuleOutput] Loading objects:
[ModuleOutput]   port1
[ModuleOutput] [1] "Loading variable port1..."
[ModuleOutput] [[1]]
[ModuleOutput] 
[ModuleOutput] 
[ModuleOutput] Autocorrelations of series 'X', by lag
[ModuleOutput] 
[ModuleOutput] 
[ModuleOutput]    -1     0     1 
[ModuleOutput] 0.148 0.358 0.317 
[ModuleOutput] 
[ModuleOutput] 
[ModuleOutput] [[2]]
[ModuleOutput] 
[ModuleOutput] 
[ModuleOutput] Autocorrelations of series 'X', by lag
[ModuleOutput] 
[ModuleOutput] 
[ModuleOutput]     -1      0      1 
[ModuleOutput] -0.395 -0.186 -0.238 
[ModuleOutput] 
[ModuleOutput] 
[ModuleOutput] [[3]]
[ModuleOutput] 
[ModuleOutput] 
[ModuleOutput] Autocorrelations of series 'X', by lag
[ModuleOutput] 
[ModuleOutput] 
[ModuleOutput]     -1      0      1 
[ModuleOutput] -0.059 -0.089 -0.127 
[ModuleOutput] 
[ModuleOutput] 
[ModuleOutput] [[4]]
[ModuleOutput] 
[ModuleOutput] 
[ModuleOutput] Autocorrelations of series 'X', by lag
[ModuleOutput] 
[ModuleOutput] 
[ModuleOutput]    -1     0     1 
[ModuleOutput] 0.140 0.294 0.293 
[ModuleOutput] 
[ModuleOutput] 
[ModuleOutput] [[5]]
[ModuleOutput] 
[ModuleOutput] 
[ModuleOutput] Autocorrelations of series 'X', by lag
[ModuleOutput] 
[ModuleOutput] 
[ModuleOutput]     -1      0      1 
[ModuleOutput] -0.002 -0.074 -0.124 
```

*Ilustración 18. Lista de objetos ccf del análisis de correlación en pares.*

Hay un valor de correlación para cada intervalo. Ninguno de estos valores de correlación es lo suficientemente grande como para que tenga algún interés. Por lo tanto, podemos concluir que podemos modelar cada variable de forma independiente.

### <a name="output-a-dataframe"></a>Generación de tramas de datos
Hemos calculado las correlaciones en pares como una lista de objetos ccf de R. Esto supone un problema, ya que el puerto de salida del conjunto de resultados resultante requiere una trama de datos. Además, el objeto ccf es en sí mismo una lista y solo necesitamos los valores del primer elemento de esta lista, es decir, las correlaciones de los distintos intervalos.

El código siguiente permite extraer los valores de los intervalos de la lista de objetos ccf, que son listas en sí mismos.

```r
df.correlations <- data.frame(do.call(rbind, lapply(cadairycorrelations, '[[', 1)))

c.names <- c("correlation pair", "-1 lag", "0 lag", "+1 lag")
r.names  <- c("Corr Cot Cheese - Ice Cream",
              "Corr Cot Cheese - Milk Prod",
              "Corr Cot Cheese - Fat Price",
              "Corr Ice Cream - Mik Prod",
              "Corr Ice Cream - Fat Price",
              "Corr Milk Prod - Fat Price")

## Build a dataframe with the row names column and the
## correlation data frame and assign the column names
outframe <- cbind(r.names, df.correlations)
colnames(outframe) <- c.names
outframe


## WARNING!
## The following line works only in Azure Machine Learning Studio (classic)
## When running in RStudio, this code will result in an error
#maml.mapOutputPort('outframe')
```

La primera línea de código puede parecer compleja, por lo que es posible que necesite algunas explicaciones para comprenderla. Desde dentro hacia fuera, tenemos lo siguiente:

1. El operador " **[[** " con el argumento "**1**" permite seleccionar el vector de correlaciones en los intervalos desde el primer elemento de la lista de objetos ccf.
2. La función `do.call()` se aplica a la función `rbind()` sobre los elementos de las devoluciones de la lista mediante `lapply()`.
3. La función `data.frame()` fuerza el resultado producido por `do.call()` en una trama de datos.

Tenga en cuenta que los nombres de fila están en una columna de la trama de datos. Esto permite conservar los nombres de fila cuando son la salida del módulo [Ejecutar script R][execute-r-script].

La ejecución del código genera la salida que se muestra en la ilustración 19 cuando se usa **Visualizar** para la salida en el puerto del conjunto de datos de resultados. Los nombres de fila están en la primera columna, según lo previsto.

![Resultados del análisis de correlación](./media/r-quickstart/fig20.png)

*Ilustración 19. Resultados del análisis de correlación.*

## <a name="time-series-example-seasonal-forecasting"></a><a id="seasonalforecasting"></a>Ejemplo de serie temporal: previsión estacional

Nuestros datos están ahora en un formato adecuado para el análisis y hemos determinado que no hay correlaciones significativas entre las variables. Vamos a continuar y a crear un modelo de previsión de serie temporal. Mediante este modelo, realizaremos una previsión de la producción de leche para California para los 12 meses de 2013.

Nuestro modelo de pronóstico tendrá dos componentes, un componente de tendencia y un componente de temporada. La previsión completa es el producto de estos dos componentes. Este tipo de modelo se conoce como un modelo de multiplicación. La alternativa es un modelo de suma. Ya hemos aplicado una transformación de registro a las variables de interés, que hace que este análisis sea manejable.

El código R completo de esta sección se encuentra en [MachineLearningSamples-Notebooks/studio-samples](https://github.com/Azure-Samples/MachineLearningSamples-Notebooks/tree/master/studio-samples).

### <a name="creating-the-dataframe-for-analysis"></a>Creación de la trama de datos para el análisis

Para empezar, agregue un **nuevo** módulo [Ejecutar script R][execute-r-script] al experimento. Conecte la salida de **Conjunto de datos de resultados** del módulo [Ejecutar script R][execute-r-script] existente a la entrada **Dataset1** del módulo nuevo. El resultado debería ser similar al que se muestra en la ilustración 20.

![Experimento con el nuevo módulo Ejecutar script de R agregado](./media/r-quickstart/fig21.png)

*Ilustración 20. Experimento con el nuevo módulo Ejecutar script de R agregado.*

Al igual que el análisis de correlación que acabamos de finalizar, tenemos que agregar una columna con un objeto de serie temporal POSIXct. El código siguiente se encargará de realizar esta operación.

```r
# If running in Machine Learning Studio (classic), uncomment the first line with maml.mapInputPort()
cadairydata <- maml.mapInputPort(1)

## Create a new column as a POSIXct object
Sys.setenv(TZ = "PST8PDT")
cadairydata$Time <- as.POSIXct(strptime(paste(as.character(cadairydata$Year), "-", as.character(cadairydata$Month.Number), "-01 00:00:00", sep = ""), "%Y-%m-%d %H:%M:%S"))

str(cadairydata)
```

Ejecute este código y examine el registro. El resultado deberá ser similar al que se muestra en la ilustración 21.

```output
[ModuleOutput] [1] "Loading variable port1..."
[ModuleOutput] 
[ModuleOutput] 'data.frame':    228 obs. of  9 variables:
[ModuleOutput] 
[ModuleOutput]  $ Month.Number     : int  1 2 3 4 5 6 7 8 9 10 ...
[ModuleOutput] 
[ModuleOutput]  $ Year             : int  1995 1995 1995 1995 1995 1995 1995 1995 1995 1995 ...
[ModuleOutput] 
[ModuleOutput]  $ Month            : Factor w/ 12 levels "Apr","Aug","Dec",..: 5 4 8 1 9 7 6 2 12 11 ...
[ModuleOutput] 
[ModuleOutput]  $ Cotagecheese.Prod: num  1.47 1.31 1.51 1.45 1.5 ...
[ModuleOutput] 
[ModuleOutput]  $ Icecream.Prod    : num  5.82 5.9 6.1 6.06 6.17 ...
[ModuleOutput] 
[ModuleOutput]  $ Milk.Prod        : num  7.66 7.57 7.68 7.66 7.71 ...
[ModuleOutput] 
[ModuleOutput]  $ N.CA.Fat.Price   : num  6.89 6.79 6.79 6.8 6.8 ...
[ModuleOutput] 
[ModuleOutput]  $ Month.Count      : num  0 1 2 3 4 5 6 7 8 9 ...
[ModuleOutput] 
[ModuleOutput]  $ Time             : POSIXct, format: "1995-01-01" "1995-02-01" ...
```

*Ilustración 21. Resumen de la trama de datos.*

Con este resultado, estamos preparados para iniciar nuestro análisis.

### <a name="create-a-training-dataset"></a>Creación de un conjunto de datos de entrenamiento

Con la trama de datos construida, necesitamos crear un conjunto de datos de entrenamiento. Estos datos incluirán todas las observaciones, excepto las últimas 12, que corresponden al año 2013 y que forman nuestro conjunto de datos de prueba. El siguiente código crea subconjuntos de la trama de datos y gráficas de las variables de producción junto con las variables de precios. A continuación, crearé gráficos de las cuatro producciones y de las variables de precios. Se utilizará una función anónima para definir algunos argumentos para el trazado y para, a continuación, iterar la lista de los otros dos argumentos con `Map()`. Si piensa que un bucle for podría haber funcionado para esta operación, está en lo cierto. Sin embargo, puesto que R es un lenguaje funcional, he preferido ofrecer un enfoque funcional.

```r
cadairytrain <- cadairydata[1:216, ]

Ylabs  <- list("Log CA Cotage Cheese Production, 1000s lb",
               "Log CA Ice Cream Production, 1000s lb",
               "Log CA Milk Production 1000s lb",
               "Log North CA Milk Milk Fat Price per 1000 lb")

Map(function(y, Ylabs){plot(cadairytrain$Time, y, xlab = "Time", ylab = Ylabs, type = "l")}, cadairytrain[, 4:7], Ylabs)
```

La ejecución del código genera series de gráficos temporales desde el resultado del dispositivo R que se muestra en la ilustración 22. Tenga en cuenta que el eje temporal se muestra en unidades de fechas. Esta es una de las ventajas del método de gráfico de series temporales.

![Primero de los trazados de series temporales de la producción de productos lácteos de California y de los datos de precios](./media/r-quickstart/unnamed-chunk-161.png)

![Segundos de los trazados de series temporales de la producción de productos lácteos de California y de los datos de precios](./media/r-quickstart/unnamed-chunk-162.png)

![Tercero de los trazados de series temporales de la producción de productos lácteos de California y de los datos de precios](./media/r-quickstart/unnamed-chunk-163.png)

![Cuarto de los trazados de series temporales de la producción de productos lácteos de California y de los datos de precios](./media/r-quickstart/unnamed-chunk-164.png)

*Ilustración 22. Trazados de series temporales de la producción de productos lácteos de California y de los datos de precios.*

### <a name="a-trend-model"></a>Modelo de tendencia

Una vez creado el objeto de la serie temporal y tras haber analizado los datos, procederemos con la creación de un modelo de tendencias para los datos de producción de leche de California. Para ello, utilizaremos una regresión de serie temporal. Sin embargo, resulta evidente en el trazado, que necesitaremos algo más una pendiente y una intercepción para modelar con precisión la tendencia observada en los datos de entrenamiento.

Dada la pequeña escala de los datos, crearemos el modelo de la tendencia en RStudio para, a continuación, cortar y pegar el modelo resultante en Azure Machine Learning Studio (clásico). RStudio proporciona un entorno interactivo para este tipo de análisis interactivo.

Como primer intento, probaré con una regresión polinómica con potencia de hasta 3. Existe un claro riesgo de sobreajuste con estos modelos. Por lo tanto, es mejor evitar los términos de alto nivel. La función `I()` impide la interpretación del contenido (interpreta el contenido "tal cual") y permite escribir una función interpretada de manera literal en una ecuación de regresión.

```r
milk.lm <- lm(Milk.Prod ~ Time + I(Month.Count^2) + I(Month.Count^3), data = cadairytrain)
summary(milk.lm)
```

Esto genera lo siguiente.

```output
##
## Call:
## lm(formula = Milk.Prod ~ Time + I(Month.Count^2) + I(Month.Count^3),
##     data = cadairytrain)
##
## Residuals:
##      Min       1Q   Median       3Q      Max
## -0.12667 -0.02730  0.00236  0.02943  0.10586
##
## Coefficients:
##                   Estimate Std. Error t value Pr(>|t|)
## (Intercept)       6.33e+00   1.45e-01   43.60   <2e-16 ***
## Time              1.63e-09   1.72e-10    9.47   <2e-16 ***
## I(Month.Count^2) -1.71e-06   4.89e-06   -0.35    0.726
## I(Month.Count^3) -3.24e-08   1.49e-08   -2.17    0.031 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
##
## Residual standard error: 0.0418 on 212 degrees of freedom
## Multiple R-squared:  0.941,    Adjusted R-squared:  0.94
## F-statistic: 1.12e+03 on 3 and 212 DF,  p-value: <2e-16
```

A partir de los valores de P (`Pr(>|t|)`) obtenidos en este resultado, podemos ver que el término al cuadrado puede que no sea significativo. Voy a utilizar la función `update()` para modificar este modelo quitando el término al cuadrado.

```r
milk.lm <- update(milk.lm, . ~ . - I(Month.Count^2))
summary(milk.lm)
```

Esto genera lo siguiente.

```output
##
## Call:
## lm(formula = Milk.Prod ~ Time + I(Month.Count^3), data = cadairytrain)
##
## Residuals:
##      Min       1Q   Median       3Q      Max
## -0.12597 -0.02659  0.00185  0.02963  0.10696
##
## Coefficients:
##                   Estimate Std. Error t value Pr(>|t|)
## (Intercept)       6.38e+00   4.07e-02   156.6   <2e-16 ***
## Time              1.57e-09   4.32e-11    36.3   <2e-16 ***
## I(Month.Count^3) -3.76e-08   2.50e-09   -15.1   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
##
## Residual standard error: 0.0417 on 213 degrees of freedom
## Multiple R-squared:  0.941,  Adjusted R-squared:  0.94
## F-statistic: 1.69e+03 on 2 and 213 DF,  p-value: <2e-16
```

Mucho mejor ahora. Todos los términos son significativos. Sin embargo, el valor 2e-16 es un valor predeterminado y no debe tomarse muy en serio.  

Como prueba de validez, vamos a crear un gráfico de serie temporal de los datos de producción de productos lácteos de California con la curva de tendencias. Hemos agregado el código siguiente al modelo [Ejecutar script de R][execute-r-script] (no RStudio) de Azure Machine Learning Studio (clásico) para crear el modelo y hacer un gráfico. El resultado aparece en la ilustración 23.

```r
milk.lm <- lm(Milk.Prod ~ Time + I(Month.Count^3), data = cadairytrain)

plot(cadairytrain$Time, cadairytrain$Milk.Prod, xlab = "Time", ylab = "Log CA Milk Production 1000s lb", type = "l")
lines(cadairytrain$Time, predict(milk.lm, cadairytrain), lty = 2, col = 2)
```

![Datos de producción de leche de California con modelo de tendencias](./media/r-quickstart/unnamed-chunk-18.png)

*Ilustración 23. Datos de producción de leche de California con modelo de tendencias.*

Parece que el modelo de tendencias se ajusta bastante bien a los datos. Además, no parece que haya signos de sobreajuste como, por ejemplo formas extrañas en la curva de modelo.  

### <a name="seasonal-model"></a>Modelo de temporada

Con un modelo de tendencias en mano, necesitamos ir más allá e incluir los efectos estacionales. Usaremos el mes del año como una variable ficticia en el modelo lineal para capturar el efecto de cada mes. Tenga en cuenta que al introducir variables factor en un modelo, la intercepción no se debe calcular. Si no lo hace, la fórmula tendrá un exceso de especificación y R quitará uno de los factores deseados pero conservará el término de intercepción.

Puesto que tenemos un modelo de tendencias satisfactorio, podemos utilizar la función `update()` para agregar los nuevos términos al modelo existente. El valor -1 en la fórmula de actualización quita el término de intercepción. Si continuamos en RStudio:

```r
milk.lm2 <- update(milk.lm, . ~ . + Month - 1)
summary(milk.lm2)
```

Esto genera lo siguiente.

```output
##
## Call:
## lm(formula = Milk.Prod ~ Time + I(Month.Count^3) + Month - 1,
##     data = cadairytrain)
##
## Residuals:
##      Min       1Q   Median       3Q      Max
## -0.06879 -0.01693  0.00346  0.01543  0.08726
##
## Coefficients:
##                   Estimate Std. Error t value Pr(>|t|)
## Time              1.57e-09   2.72e-11    57.7   <2e-16 ***
## I(Month.Count^3) -3.74e-08   1.57e-09   -23.8   <2e-16 ***
## MonthApr          6.40e+00   2.63e-02   243.3   <2e-16 ***
## MonthAug          6.38e+00   2.63e-02   242.2   <2e-16 ***
## MonthDec          6.38e+00   2.64e-02   241.9   <2e-16 ***
## MonthFeb          6.31e+00   2.63e-02   240.1   <2e-16 ***
## MonthJan          6.39e+00   2.63e-02   243.1   <2e-16 ***
## MonthJul          6.39e+00   2.63e-02   242.6   <2e-16 ***
## MonthJun          6.38e+00   2.63e-02   242.4   <2e-16 ***
## MonthMar          6.42e+00   2.63e-02   244.2   <2e-16 ***
## MonthMay          6.43e+00   2.63e-02   244.3   <2e-16 ***
## MonthNov          6.34e+00   2.63e-02   240.6   <2e-16 ***
## MonthOct          6.37e+00   2.63e-02   241.8   <2e-16 ***
## MonthSep          6.34e+00   2.63e-02   240.6   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
##
## Residual standard error: 0.0263 on 202 degrees of freedom
## Multiple R-squared:     1,    Adjusted R-squared:     1
## F-statistic: 1.42e+06 on 14 and 202 DF,  p-value: <2e-16
```

Vemos que el modelo ya no tiene ningún término de intercepción y que cuenta con 12 factores de mes. Esto es exactamente lo que queríamos ver.

A continuación, crearemos otro gráfico de serie temporal con los datos de producción de productos lácteos de California para ver cómo funciona el modelo de temporada. Hemos agregado el código siguiente al módulo [Ejecutar script R][execute-r-script] de Azure Machine Learning Studio (clásico) para crear el modelo y hacer un gráfico.

```r
milk.lm2 <- lm(Milk.Prod ~ Time + I(Month.Count^3) + Month - 1, data = cadairytrain)

plot(cadairytrain$Time, cadairytrain$Milk.Prod, xlab = "Time", ylab = "Log CA Milk Production 1000s lb", type = "l")
lines(cadairytrain$Time, predict(milk.lm2, cadairytrain), lty = 2, col = 2)
```

La ejecución de este código en Azure Machine Learning Studio (clásico) genera el gráfico que se muestra en la figura 24.

![Producción de leche de California con modelo que incluye los efectos de temporada](./media/r-quickstart/unnamed-chunk-20.png)

*Ilustración 24. Producción de leche de California con modelo que incluye los efectos de temporada.*

El ajuste a los datos que se muestran en la ilustración 24 es bastante alentador. Tanto la tendencia como el efecto estacional (variación mensual) parecen ser razonables.

Con el fin de realizar una comprobación adicional en nuestro modelo, veamos los valores residuales. El código siguiente calcula los valores de predicción de los dos modelos, calcula los valores residuales para el modelo de temporada y, a continuación, crea un trazado de estos valores residuales para los datos de entrenamiento.

```r
## Compute predictions from our models
predict1  <- predict(milk.lm, cadairydata)
predict2  <- predict(milk.lm2, cadairydata)

## Compute and plot the residuals
residuals <- cadairydata$Milk.Prod - predict2
plot(cadairytrain$Time, residuals[1:216], xlab = "Time", ylab ="Residuals of Seasonal Model")
```

El gráfico de valores residuales se muestra en la ilustración 25.

![Valores residuales del modelo estacional para los datos de entrenamiento](./media/r-quickstart/unnamed-chunk-21.png)

*Ilustración 25. Valores residuales del modelo estacional para los datos de entrenamiento.*

Estos valores residuales parecen ser razonables. No hay ninguna estructura determinada, excepto el efecto de la recesión de 2008 y 2009 que nuestro modelo no tiene especialmente en cuenta.

El gráfico que se muestra en la ilustración 25 es útil para detectar los patrones que dependen del tiempo en los valores residuales. El enfoque explícito de cálculo y gráficos de los valores residuales que he utilizado coloca los valores residuales por orden cronológico en el gráfico. Por otro lado, si hubiese creado el trazado con la  `milk.lm$residuals`, este no estaría ordenado de forma cronológica.

También puede usar la función `plot.lm()` para generar una serie de trazados de diagnóstico:

```r
## Show the diagnostic plots for the model
plot(milk.lm2, ask = FALSE)
```

Este código genera una serie de trazados de diagnósticos que se muestran en la ilustración 26.

![Primero de los trazados de diagnóstico para el modelo estacional](./media/r-quickstart/unnamed-chunk-221.png)

![Segundo de los trazados de diagnóstico para el modelo estacional](./media/r-quickstart/unnamed-chunk-222.png)

![Tercero de los trazados de diagnóstico para el modelo estacional](./media/r-quickstart/unnamed-chunk-223.png)

![Cuarto de los trazados de diagnóstico para el modelo estacional](./media/r-quickstart/unnamed-chunk-224.png)

*Ilustración 26. Trazados de diagnóstico para el modelo estacional.*

Existen varios puntos influyentes identificados en estos gráficos, pero nada por lo que debamos preocuparnos. Además, podemos ver en el gráfico Normal Q-Q que los valores residuales son próximos a los de distribución normal, una suposición importante para los modelos lineales.

### <a name="forecasting-and-model-evaluation"></a>Evaluación del modelo y predicción

Hay solo una cosa más que nos falta por hacer para completar nuestro ejemplo. Tenemos que calcular las previsiones y evaluar el error con respecto a los datos reales. Nuestra previsión será para los 12 meses de 2013. Podemos calcular una evaluación de error para esta previsión de los datos reales que no forman parte de nuestro conjunto de datos de entrenamiento. Además, podemos comparar el rendimiento de los 18 años de datos de entrenamiento con los 12 meses de los datos de prueba.  

Se utiliza una gran cantidad de métricas para medir el rendimiento de los modelos de serie temporal. En nuestro caso, usaremos el error cuadrático medio (RMS). La función siguiente calcula el error RMS entre dos series.  

```r
RMS.error <- function(series1, series2, is.log = TRUE, min.length = 2){
  ## Function to compute the RMS error or difference between two
  ## series or vectors

  messages <- c("ERROR: Input arguments to function RMS.error of wrong type encountered",
                "ERROR: Input vector to function RMS.error is too short",
                "ERROR: Input vectors to function RMS.error must be of same length",
                "WARNING: Funtion rms.error has received invald input time series.")

  ## Check the arguments
  if(!is.numeric(series1) | !is.numeric(series2) | !is.logical(is.log) | !is.numeric(min.length)) {
    warning(messages[1])
    return(NA)}

  if(length(series1) < min.length) {
    warning(messages[2])
    return(NA)}

  if((length(series1) != length(series2))) {
       warning(messages[3])
    return(NA)}

  ## If is.log is TRUE exponentiate the values, else just copy
  if(is.log) {
    tryCatch( {
      temp1 <- exp(series1)
      temp2 <- exp(series2) },
      error = function(e){warning(messages[4]); NA}
    )
  } else {
    temp1 <- series1
    temp2 <- series2
  }

 ## Compute predictions from our models
predict1  <- predict(milk.lm, cadairydata)
predict2  <- predict(milk.lm2, cadairydata)

## Compute the RMS error in a dataframe
  tryCatch( {
    sqrt(sum((temp1 - temp2)^2) / length(temp1))},
    error = function(e){warning(messages[4]); NA})
}
```

Como ocurre con la función `log.transform()` explicada en la sección "Transformaciones de valor", en esta función hay una gran cantidad de código de recuperación de excepciones y de comprobación de errores. Los principios empleados son los mismos. El trabajo se realiza en dos puntos en la función `tryCatch()`. En primer lugar, se potencia la serie temporal, ya que hemos estado trabajando con los registros de los valores. En segundo lugar, se calcula el error RMS real.  

Equipados con una función para medir el error RMS, vamos a compilar y a crear una trama de datos que contenga los errores RMS. Incluiremos términos para el modelo de tendencias únicamente y para el modelo completo con factores de temporada. El siguiente código realiza el trabajo con los dos modelos lineales que hemos creado.

```r
## Compute the RMS error in a dataframe
## Include the row names in the first column so they will
## appear in the output of the Execute R Script
RMS.df  <-  data.frame(
rowNames = c("Trend Model", "Seasonal Model"),
  Traing = c(
  RMS.error(predict1[1:216], cadairydata$Milk.Prod[1:216]),
  RMS.error(predict2[1:216], cadairydata$Milk.Prod[1:216])),
  Forecast = c(
    RMS.error(predict1[217:228], cadairydata$Milk.Prod[217:228]),
    RMS.error(predict2[217:228], cadairydata$Milk.Prod[217:228]))
)
RMS.df

## The following line should be executed only when running in
## Azure Machine Learning Studio (classic)
maml.mapOutputPort('RMS.df')
```

La ejecución de este código genera el resultado que se muestra en la ilustración 27 en el puerto de salida del conjunto de datos de resultado.

![Comparación de errores RMS para los modelos](./media/r-quickstart/fig26.png)

*Ilustración 27. Comparación de errores RMS para los modelos.*

Según estos resultados, podemos ver que el hecho de agregar factores estacionales al modelo reduce significativamente el error RMS. No es sorprendente que el error RMS de los datos de entrenamiento sea menor que el del pronóstico.

## <a name="guide-to-rstudio-documentation"></a><a id="appendixa"></a>Guía de la documentación de RStudio

RStudio cuenta con una documentación bastante extensa. Estos son algunos vínculos a secciones claves de la documentación de RStudio para ayudarle a comenzar.

* **Creación de proyectos**: puede organizar y administrar el código R en proyectos mediante RStudio. Consulte [Using Projects](https://support.rstudio.com/hc/articles/200526207-Using-Projects) (Uso de proyectos) para obtener más información. Se recomienda que siga estas instrucciones y cree un proyecto para los ejemplos de código R de este artículo.  
* **Edición y ejecución de código R**: RStudio proporciona un entorno integrado para editar y ejecutar código R. Consulte [Editing and Executing Code](https://support.rstudio.com/hc/articles/200484448-Editing-and-Executing-Code) (Edición y ejecución de código) para obtener más información.
* **Depuración**: RStudio incluye eficaces capacidades de depuración. Consulte [Debugging with RStudio](https://support.rstudio.com/hc/articles/200713843-Debugging-with-RStudio) (Depuración con RStudio) para obtener más información acerca de estas características. Para obtener información acerca de las características de solución de problemas en el punto de interrupción, consulte [Breakpoint Troubleshooting](https://support.rstudio.com/hc/articles/200534337-Breakpoint-Troubleshooting) (Solución de problemas de punto de interrupción).

## <a name="further-reading"></a><a id="appendixb"></a>Lecturas adicionales

Este tutorial de programación de R cubre los aspectos básicos de lo que necesita para usar el lenguaje R con Azure Machine Learning Studio (clásico). Si no está familiarizado con el código R, encontrará dos introducciones disponibles en CRAN:

* [R for Beginners](https://cran.r-project.org/doc/contrib/Paradis-rdebuts_en.pdf) (R para principiantes) de Emmanuel Paradis es un buen lugar para comenzar.  
* [An Introduction to R](https://cran.r-project.org/doc/manuals/R-intro.html) (Introducción a R) de W. N. Venables et. al. profundiza un poco más en el tema.

Existen muchas obras sobre el código R que pueden servirle como punto de partida. Estas son algunas que considero más útiles:

* The **Art of R Programming: A Tour of Statistical Software Design** (El arte de la programación: un recorrido por el diseño de software estadístico) de Norman Matloff ofrece una excelente introducción a la programación en código R.  
* **R Cookbook** (Guía paso a paso de R) de Paul Teetor ofrece un enfoque del uso del código R basado en problemas y soluciones.  
* **R in Action** (R en acción) de Robert Kabacoff es otro libro que puede resultarle muy útil. El [sitio web Quick R](https://www.statmethods.net/) complementario es un recurso que le será de gran utilidad.
* **R Inferno** (Infierno de R), de Patrick Burns, es un libro sorprendentemente divertido que le ayudará a abordar numerosos temas complejos con los que puede encontrarse a la hora de programar en R. Esta obra está disponible gratis en [este enlace](https://www.burns-stat.com/documents/books/the-r-inferno/).
* Si desea obtener información más detallada sobre temas avanzados de R, recomendamos el título **Advanced R** (R avanzado) de Hadley Wickham. La versión en línea de este libro está disponible de forma gratuita en [http://adv-r.had.co.nz/](http://adv-r.had.co.nz/).

Podrá encontrar un catálogo de series temporales de R en [CRAN Task View: Time Series Analysis](https://cran.r-project.org/web/views/TimeSeries.html) (Vista de tareas de CRAN: análisis de series temporales). Para obtener información sobre paquetes de objetos de series temporales, debe hacer referencia a la documentación de ese paquete.

El libro **Introductory Time Series with R** (Series temporales básicas con R) de Paul Cowpertwait y Andrew Metcalfe ofrece una introducción al uso de R para el análisis de series temporales. No obstante, existen muchos más textos teóricos que proporcionan ejemplos de R.

Los siguientes algunos recursos excelentes en Internet:

* DataCamp enseña R desde la comodidad del explorador con lecciones en vídeo y ejercicios de codificación. Existen tutoriales interactivos sobre los paquetes y las técnicas más recientes de R. Siga el [tutorial interactivo de R](https://www.datacamp.com/courses/introduction-to-r) gratuito.
* [Learn R Programming, The Definitive Guide](https://www.programiz.com/r-programming) (La guía definitiva para aprender programación en R) de Programiz.
* Un [tutorial rápido de R](https://www.cyclismo.org/tutorial/R/) de Kelly Black de la Universidad de Clarkson.
* Hay más de 60 recursos de R en la lista [Top R language resources to improve your data skills](https://www.computerworld.com/article/2497464/business-intelligence-60-r-resources-to-improve-your-data-skills.html) (Los mejores recursos del lenguaje R para mejorar sus habilidades de datos).

<!-- Module References -->
[execute-r-script]: /azure/machine-learning/studio-module-reference/execute-r-script
