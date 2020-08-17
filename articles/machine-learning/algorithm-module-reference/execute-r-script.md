---
title: 'Ejecutar script R: Referencia del módulo'
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar el módulo Ejecutar script R en Azure Machine Learning para ejecutar código R.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 07/27/2020
ms.openlocfilehash: 873f0d7d2aa4493e77a10f62b0646f4f8233f6b9
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87337847"
---
# <a name="execute-r-script-module"></a>Módulo Execute R Script

En este artículo se describe cómo usar el módulo Execute R Script (Ejecutar script R) para ejecutar código R en la canalización de Azure Machine Learning Designer (versión preliminar).

Con R, puede realizar tareas que no son compatibles actualmente con los módulos existentes, como: 
- Crear transformaciones de datos personalizadas
- Usar sus propias métricas para evaluar las predicciones
- Generar modelos mediante algoritmos que no se implementan como módulos independientes en el diseñador

## <a name="r-version-support"></a>Compatibilidad con la versión de R

El diseñador de Azure Machine Learning usa la distribución CRAN (red de archivo de R completa) de R. La versión que se usa actualmente es CRAN 3.5.1.

## <a name="supported-r-packages"></a>Paquetes de R admitidos

El entorno de R está preinstalado con más de 100 paquetes. Para obtener una lista completa, consulte la sección [Paquetes preinstalados de R](#preinstalled-r-packages).

También puede agregar el código siguiente a cualquier módulo Execute R Script (Ejecutar script R) para ver los paquetes instalados.

```R
azureml_main <- function(dataframe1, dataframe2){
  print("R script run.")
  dataframe1 <- data.frame(installed.packages())
  return(list(dataset1=dataframe1, dataset2=dataframe2))
}
```
> [!NOTE]
> Si la canalización contiene varios módulos Execute R Script (Ejecutar script R) que necesitan paquetes que no están en la lista de elementos preinstalados, instale los paquetes en cada módulo. 

## <a name="installing-r-packages"></a>Instalación de paquetes de R
Para instalar paquetes de R adicionales, use el método `install.packages()`. Los paquetes se instalan para cada módulo Execute R Script (Ejecutar script R). No se comparten con otros módulos Execute R Script (Ejecutar script R).

> [!NOTE]
> Especifique el repositorio de CRAN al instalar paquetes, como `install.packages("zoo",repos = "http://cran.us.r-project.org")`.

En este ejemplo se muestra cómo instalar Zoo:
```R
# R version: 3.5.1
# The script MUST contain a function named azureml_main,
# which is the entry point for this module.

# Note that functions dependent on the X11 library,
# such as "View," are not supported because the X11 library
# is not preinstalled.

# The entry point function MUST have two input arguments.
# If the input port is not connected, the corresponding
# dataframe argument will be null.
#   Param<dataframe1>: a R DataFrame
#   Param<dataframe2>: a R DataFrame
azureml_main <- function(dataframe1, dataframe2){
  print("R script run.")
  
  if(!require(zoo)) install.packages("zoo",repos = "http://cran.us.r-project.org")
  library(zoo)
  # Return datasets as a Named List
  return(list(dataset1=dataframe1, dataset2=dataframe2))
}
```
 > [!NOTE]
 > Antes de instalar un paquete, compruebe si ya existe para no repetir una instalación. La repetición de instalaciones pueden hacer que se agote el tiempo de espera de las solicitudes de servicio web.     

## <a name="uploading-files"></a>Carga de archivos
El módulo Execute R Script (Ejecutar script R) admite la carga de archivos mediante el SDK de R de Azure Machine Learning.

En el ejemplo siguiente se muestra cómo cargar un archivo de imagen en Execute R Script (Ejecutar script R):
```R

# R version: 3.5.1
# The script MUST contain a function named azureml_main,
# which is the entry point for this module.

# Note that functions dependent on the X11 library,
# such as "View," are not supported because the X11 library
# is not preinstalled.

# The entry point function MUST have two input arguments.
# If the input port is not connected, the corresponding
# dataframe argument will be null.
#   Param<dataframe1>: a R DataFrame
#   Param<dataframe2>: a R DataFrame
azureml_main <- function(dataframe1, dataframe2){
  print("R script run.")

  # Generate a jpeg graph
  img_file_name <- "rect.jpg"
  jpeg(file=img_file_name)
  example(rect)
  dev.off()

  upload_files_to_run(names = list(file.path("graphic", img_file_name)), paths=list(img_file_name))


  # Return datasets as a Named List
  return(list(dataset1=dataframe1, dataset2=dataframe2))
}
```

Una vez que termina la ejecución de la canalización, se puede obtener una vista previa de la imagen en el panel derecho del módulo.

> [!div class="mx-imgBorder"]
> ![Vista previa de la imagen cargada](media/module/upload-image-in-r-script.png)

## <a name="access-to-registered-dataset"></a>Acceso al conjunto de datos registrado

Puede consultar el siguiente código de ejemplo para [acceder a los conjuntos de datos registrados](https://docs.microsoft.com/azure/machine-learning/how-to-create-register-datasets#access-datasets-in-your-script) en el área de trabajo:

```R
        azureml_main <- function(dataframe1, dataframe2){
  print("R script run.")
  run = get_current_run()
  ws = run$experiment$workspace
  dataset = azureml$core$dataset$Dataset$get_by_name(ws, "YOUR DATASET NAME")
  dataframe2 <- dataset$to_pandas_dataframe()
  # Return datasets as a Named List
  return(list(dataset1=dataframe1, dataset2=dataframe2))
}
```

## <a name="how-to-configure-execute-r-script"></a>Procedimiento para configurar Ejecutar script R

El módulo Execute R Script (Ejecutar script R) contiene código de ejemplo que puede usar como punto de partida. Para configurar el módulo Execute R Script (Ejecutar script R), proporcione un conjunto de entradas y código para ejecutarlo.

![Diagrama de entradas para un módulo de R](media/module/execute-r-script.png)

Los conjuntos de datos almacenados en el diseñador se convierten automáticamente en una trama de datos R cuando se cargan con este módulo.

1.  Agregue el módulo **Execute R Script** (Ejecutar script de R) a la canalización.  

1. Conecte cualquier entrada que necesite el script. Las entradas son opcionales y pueden incluir datos y código de R adicional.

    * **Dataset1**: haga referencia a la primera entrada como `dataframe1`. El conjunto de datos de entrada tiene que tener el formato de un archivo CSV, TSV o ARFF. O bien, puede conectar un conjunto de datos de Azure Machine Learning.

    * **Dataset2**: haga referencia a la segunda entrada como `dataframe2`. Este conjunto de datos también tiene que tener un formato de archivo CSV, TSV o ARFF, o de conjunto de datos de Azure Machine Learning.

    * **Conjunto de scripts**: la tercera entrada acepta archivos ZIP. Un archivo ZIP puede contener varios archivos y varios tipos de archivo.

1. En el cuadro de texto **Script de R**, escriba o pegue el script de R válido.

    > [!NOTE]
    > Escriba el script con cuidado. Asegúrese de que no hay errores de sintaxis, como el uso de variables no declaradas o de funciones o módulos no importados. Preste especial atención a la lista de paquetes preinstalados al final de este artículo. Para usar paquetes que no aparecen en la lista, instálelos en el script. Un ejemplo es `install.packages("zoo",repos = "http://cran.us.r-project.org")`.
    
    Para ayudarle a empezar, el cuadro de texto **Script de R** se muestra rellenado previamente con el código de ejemplo, que puede editar o reemplazar.
    
    ```R
    # R version: 3.5.1
    # The script MUST contain a function named azureml_main,
    # which is the entry point for this module.

    # Note that functions dependent on the X11 library,
    # such as "View," are not supported because the X11 library
    # is not preinstalled.
    
    # The entry point function MUST have two input arguments.
    # If the input port is not connected, the corresponding
    # dataframe argument will be null.
    #   Param<dataframe1>: a R DataFrame
    #   Param<dataframe2>: a R DataFrame
    azureml_main <- function(dataframe1, dataframe2){
    print("R script run.")

    # If a .zip file is connected to the third input port, it's
    # unzipped under "./Script Bundle". This directory is added
    # to sys.path.

    # Return datasets as a Named List
    return(list(dataset1=dataframe1, dataset2=dataframe2))
    }
    ```

    La función de punto de entrada tiene que tener los argumentos de entrada (`Param<dataframe1>` y `Param<dataframe2>`), aunque estos argumentos no se usen en la función.

    > [!NOTE]
    > Se hace referencia a los datos pasados al módulo Execute R Script (Ejecutar script R) como `dataframe1` y `dataframe2`, lo que es distinto de Azure Machine Learning Designer, en este la referencia es `dataset1` y `dataset2`. Asegúrese de que la referencia de los datos de entrada es correcta en el script.  
 
    > [!NOTE]
    > Puede que el código R existente necesite pequeños cambios para ejecutarse en una canalización de diseñador. Por ejemplo, los datos de entrada que se proporcionan en formato CSV deben convertirse explícitamente en un conjunto de datos para que pueda usarlos en su código. Los tipos de datos y columnas que se usan en el lenguaje R también difieren en algunos aspectos de los tipos de columnas y datos que se usan en el diseñador.

    Si el script tiene más de 16 KB, use el puerto **Conjunto de scripts** para evitar errores parecidos a *CommandLine supera el límite de 16597 caracteres*. 
    
    Agrupe el script y otros recursos personalizados en un archivo zip, y cargue el archivo zip como un **conjunto de datos de archivo** en el estudio. Luego puede arrastrar el módulo del conjunto de datos de la lista *My datasets* (Mis conjuntos de datos) en el panel de módulos de la izquierda a la página de creación del diseñador. Conecte el módulo de conjunto de datos al puerto **Conjunto de scripts** del módulo **Ejecutar script R**.
    
    A continuación, se muestra el código de ejemplo para consumir el script en el conjunto de scripts:

    ```R
    azureml_main <- function(dataframe1, dataframe2){
    # Source the custom R script: my_script.R
    source("./Script Bundle/my_script.R")

    # Use the function that defined in my_script.R
    dataframe1 <- my_func(dataframe1)

    sample <- readLines("./Script Bundle/my_sample.txt")
    return (list(dataset1=dataframe1, dataset2=data.frame("Sample"=sample)))
    }
    ```

1.  Para **Random Seed** (Valor de inicialización aleatorio) escriba un valor para usarlo en el entorno de R como valor de inicialización aleatorio. Este parámetro equivale a llamar a `set.seed(value)` en el código de R.  

1. Envíe la canalización.  

## <a name="results"></a>Results

Los módulos Execute R Script (Ejecutar script R) pueden devolver varias salidas, pero se tienen que proporcionar como tramas de datos de R. Las tramas de datos se convierten automáticamente en conjuntos de datos en el diseñador para su compatibilidad con otros módulos.

Los mensajes estándar y los errores de R se devuelven al registro del módulo.

Si necesita imprimir los resultados del script de R, puede buscar los resultados impresos en **70_driver_log**, en la pestaña **Outputs+logs** (Salidas y registros) del panel derecho del módulo.

## <a name="sample-scripts"></a>Muestras de scripts

Hay muchas formas de ampliar una canalización mediante los scripts de R personalizados. En esta sección se proporciona el código de ejemplo para las tareas comunes.


### <a name="add-an-r-script-as-an-input"></a>Incorporación de un script de R como entrada

El módulo Execute R Script (Ejecutar script R) admite archivos de script de R arbitrarios como entradas. Para usarlos, tiene que cargarlos en el área de trabajo como parte del archivo. zip.

1. Para cargar un archivo ZIP que contenga código R en el área de trabajo, vaya a la página de recurso **Datasets** (Conjuntos de datos). Seleccione **Create dataset** (Crear conjunto de datos) y, después, seleccione **From local file** (Desde archivo local) y la opción de tipo de conjunto de datos **File** (Archivo).  

1. Compruebe que el archivo comprimido está disponible en la lista **My Datasets** (Mis conjuntos de datos) en la categoría **Datasets** (Conjuntos de datos) del árbol de módulos de la izquierda.

1.  Conecte el conjunto de datos al puerto de entrada del **conjunto de scripts**.

1. Todos los archivos del archivo ZIP están disponibles durante el tiempo de ejecución de la canalización. 

    Si el archivo del conjunto de scripts contiene una estructura de directorios, esta se conserva. Sin embargo, tiene que modificar el código para anteponer el directorio **./Script Bundle** a la ruta de acceso.

### <a name="process-data"></a>Datos de proceso

En el ejemplo siguiente se muestra cómo escalar y normalizar los datos de entrada:

```R
# R version: 3.5.1
# The script MUST contain a function named azureml_main,
# which is the entry point for this module.

# Note that functions dependent on the X11 library,
# such as "View," are not supported because the X11 library
# is not preinstalled.

# The entry point function MUST have two input arguments.
# If the input port is not connected, the corresponding
# dataframe argument will be null.
#   Param<dataframe1>: a R DataFrame
#   Param<dataframe2>: a R DataFrame
azureml_main <- function(dataframe1, dataframe2){
  print("R script run.")
  # If a .zip file is connected to the third input port, it's
  # unzipped under "./Script Bundle". This directory is added
  # to sys.path.
  series <- dataframe1$width
  # Find the maximum and minimum values of the width column in dataframe1
  max_v <- max(series)
  min_v <- min(series)
  # Calculate the scale and bias
  scale <- max_v - min_v
  bias <- min_v / dis
  # Apply min-max normalizing
  dataframe1$width <- dataframe1$width / scale - bias
  dataframe2$width <- dataframe2$width / scale - bias
  # Return datasets as a Named List
  return(list(dataset1=dataframe1, dataset2=dataframe2))
}
 ```

### <a name="read-a-zip-file-as-input"></a>Lectura de un archivo ZIP como entrada

En este ejemplo se muestra cómo usar un conjunto de datos en un archivo ZIP como entrada para el módulo Execute R Script (Ejecutar script R).

1. Cree el archivo de datos en formato CSV y asígnele el nombre **mydatafile.csv**.
1. Cree un archivo ZIP y agregue el archivo CSV al archivo.
1. Cargue el archivo comprimido en su área de trabajo de Azure Machine Learning. 
1. Conecte el conjunto de datos resultante en la entrada **ScriptBundle** del módulo **Ejecutar script R**.
1. Use el código siguiente para leer los datos en formato CSV del archivo comprimido.

```R
azureml_main <- function(dataframe1, dataframe2){
  print("R script run.")
  mydataset<-read.csv("./Script Bundle/mydatafile.csv",encoding="UTF-8");  
  # Return datasets as a Named List
  return(list(dataset1=mydataset, dataset2=dataframe2))
}
```

### <a name="replicate-rows"></a>Replicar filas

En este ejemplo se muestra cómo replicar registros positivos en un conjunto de datos para equilibrar la muestra:

```R
azureml_main <- function(dataframe1, dataframe2){
  data.set <- dataframe1[dataframe1[,1]==-1,]  
  # positions of the positive samples
  pos <- dataframe1[dataframe1[,1]==1,]
  # replicate the positive samples to balance the sample  
  for (i in 1:20) data.set <- rbind(data.set,pos)  
  row.names(data.set) <- NULL
  # Return datasets as a Named List
  return(list(dataset1=data.set, dataset2=dataframe2))
}
```

### <a name="pass-r-objects-between-execute-r-script-modules"></a>Pasar objetos R entre módulos Ejecutar script R

Puede pasar objetos R entre instancias del módulo Execute R Script (Ejecutar script R) mediante el mecanismo de serialización interno. En este ejemplo se da por supuesto que quiere mover el objeto de R denominado `A` entre dos módulos Execute R Script (Ejecutar script R).

1. Agregue el primer módulo **Execute R Script** (Ejecutar script de R) a la canalización. Después, especifique el código siguiente en el cuadro de texto **Script de R** para crear un objeto serializado `A` como columna en la tabla de datos de salida del módulo:  
  
    ```R
    azureml_main <- function(dataframe1, dataframe2){
      print("R script run.")
      # some codes generated A
      
      serialized <- as.integer(serialize(A,NULL))  
      data.set <- data.frame(serialized,stringsAsFactors=FALSE)

      return(list(dataset1=data.set, dataset2=dataframe2))
    }
    ```

    La conversión explícita al tipo entero se realiza porque la función de serialización devuelve los datos en el formato de R `Raw`, lo que el diseñador no admite.

1. Agregue una segunda instancia del módulo **Ejecutar script R** y conéctelo al puerto de salida del módulo anterior.

1. Escriba el siguiente código en el cuadro de texto **Script de R** para extraer el objeto `A` de la tabla de datos de entrada. 

    ```R
    azureml_main <- function(dataframe1, dataframe2){
      print("R script run.")
      A <- unserialize(as.raw(dataframe1$serialized))  
      # Return datasets as a Named List
      return(list(dataset1=dataframe1, dataset2=dataframe2))
    }
    ```

## <a name="preinstalled-r-packages"></a>Paquetes preinstalados de R

Los siguientes paquetes preinstalados de R están disponibles actualmente:

| Paquete      | Versión    | 
|--------------|------------| 
| askpass      | 1.1        | 
| assertthat   | 0.2.1      | 
| backports    | 1.1.4      | 
| base         | 3.5.1      | 
| base64enc    | 0.1-3      | 
| BH           | 1.69.0-1   | 
| bindr        | 0.1.1      | 
| bindrcpp     | 0.2.2      | 
| bitops       | 1.0-6      | 
| boot         | 1.3-22     | 
| broom        | 0.5.2      | 
| callr        | 3.2.0      | 
| caret        | 6.0-84     | 
| caTools      | 1.17.1.2   | 
| cellranger   | 1.1.0      | 
| class        | 7.3-15     | 
| cli          | 1.1.0      | 
| clipr        | 0.6.0      | 
| cluster      | 2.0.7-1    | 
| codetools    | 0.2-16     | 
| colorspace   | 1.4-1      | 
| compiler     | 3.5.1      | 
| crayon       | 1.3.4      | 
| curl         | 3.3        | 
| data.table   | 1.12.2     | 
| conjuntos de datos     | 3.5.1      | 
| DBI          | 1.0.0      | 
| dbplyr       | 1.4.1      | 
| digest       | 0.6.19     | 
| dplyr        | 0.7.6      | 
| e1071        | 1.7-2      | 
| evaluate     | 0.14       | 
| fansi        | 0.4.0      | 
| forcats      | 0.3.0      | 
| foreach      | 1.4.4      | 
| foreign      | 0.8-71     | 
| fs           | 1.3.1      | 
| gdata        | 2.18.0     | 
| generics     | 0.0.2      | 
| ggplot2      | 3.2.0      | 
| glmnet       | 2.0-18     | 
| glue         | 1.3.1      | 
| gower        | 0.2.1      | 
| gplots       | 3.0.1.1    | 
| graphics     | 3.5.1      | 
| grDevices    | 3.5.1      | 
| grid         | 3.5.1      | 
| gtable       | 0.3.0      | 
| gtools       | 3.8.1      | 
| haven        | 2.1.0      | 
| highr        | 0.8        | 
| hms          | 0.4.2      | 
| htmltools    | 0.3.6      | 
| httr         | 1.4.0      | 
| ipred        | 0.9-9      | 
| iterators    | 1.0.10     | 
| jsonlite     | 1.6        | 
| KernSmooth   | 2.23-15    | 
| knitr        | 1.23       | 
| labeling     | 0,3        | 
| lattice      | 0.20-38    | 
| lava         | 1.6.5      | 
| lazyeval     | 0.2.2      | 
| lubridate    | 1.7.4      | 
| magrittr     | 1.5        | 
| markdown     | 1          | 
| MASS         | 7.3-51.4   | 
| Matrix       | 1.2-17     | 
| methods      | 3.5.1      | 
| mgcv         | 1.8-28     | 
| mime         | 0,7        | 
| ModelMetrics | 1.2.2      | 
| modelr       | 0.1.4      | 
| munsell      | 0.5.0      | 
| nlme         | 3.1-140    | 
| nnet         | 7.3-12     | 
| numDeriv     | 2016.8-1.1 | 
| openssl      | 1.4        | 
| parallel     | 3.5.1      | 
| pillar       | 1.4.1      | 
| pkgconfig    | 2.0.2      | 
| plogr        | 0.2.0      | 
| plyr         | 1.8.4      | 
| prettyunits  | 1.0.2      | 
| processx     | 3.3.1      | 
| prodlim      | 2018.04.18 | 
| progreso     | 1.2.2      | 
| ps           | 1.3.0      | 
| purrr        | 0.3.2      | 
| quadprog     | 1.5-7      | 
| quantmod     | 0.4-15     | 
| R6           | 2.4.0      | 
| randomForest | 4.6-14     | 
| RColorBrewer | 1.1-2      | 
| Rcpp         | 1.0.1      | 
| RcppRoll     | 0.3.0      | 
| readr        | 1.3.1      | 
| readxl       | 1.3.1      | 
| recipes      | 0.1.5      | 
| rematch      | 1.0.1      | 
| reprex       | 0.3.0      | 
| reshape2     | 1.4.3      | 
| reticulate   | 1.12       | 
| rlang        | 0.4.0      | 
| rmarkdown    | 1.13       | 
| ROCR         | 1.0-7      | 
| rpart        | 4.1-15     | 
| rstudioapi   | 0,1        | 
| rvest        | 0.3.4      | 
| scales       | 1.0.0      | 
| selectr      | 0.4-1      | 
| spatial      | 7.3-11     | 
| splines      | 3.5.1      | 
| SQUAREM      | 2017.10-1  | 
| stats        | 3.5.1      | 
| stats4       | 3.5.1      | 
| stringi      | 1.4.3      | 
| stringr      | 1.3.1      | 
| survival     | 2.44-1.1   | 
| sys          | 3.2        | 
| tcltk        | 3.5.1      | 
| tibble       | 2.1.3      | 
| tidyr        | 0.8.3      | 
| tidyselect   | 0.2.5      | 
| tidyverse    | 1.2.1      | 
| timeDate     | 3043.102   | 
| tinytex      | 0,13       | 
| herramientas        | 3.5.1      | 
| tseries      | 0.10-47    | 
| TTR          | 0.23-4     | 
| utf8         | 1.1.4      | 
| utils        | 3.5.1      | 
| vctrs        | 0.1.0      | 
| viridisLite  | 0.3.0      | 
| whisker      | 0.3-2      | 
| withr        | 2.1.2      | 
| xfun         | 0.8        | 
| xml2         | 1.2.0      | 
| xts          | 0.11-2     | 
| yaml         | 2.2.0      | 
| zeallot      | 0.1.0      | 
| zoo          | 1.8-6      | 

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 
