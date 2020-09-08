---
title: Explorar Linux
titleSuffix: Azure Data Science Virtual Machine
description: Aprenda a completar varias tareas comunes de ciencia de datos mediante Data Science Virtual Machine de Linux.
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
author: vijetajo
ms.author: vijetaj
ms.topic: conceptual
ms.date: 07/17/2020
ms.openlocfilehash: ca3cfa44bd4f757c6fbb0dd2c84d7a843f9bff36
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88816225"
---
# <a name="data-science-with-an-ubuntu-data-science-virtual-machine-in-azure"></a>Ciencia de datos con una instancia de Data Science Virtual Machine de Ubuntu en Azure

En este tutorial se muestra cómo realizar varias tareas comunes de ciencia de datos mediante Data Science Virtual Machine (DSMV) de Ubuntu. DSVM de Ubuntu es una imagen de máquina virtual disponible en Azure que viene preinstalada con una colección de herramientas usadas normalmente en el análisis de datos y el aprendizaje automático. Los componentes de software principales se detallan en el tema [Aprovisionamiento de una instancia de Data Science Virtual Machine de Ubuntu](./dsvm-ubuntu-intro.md). La imagen de DSMV permite comenzar a trabajar fácilmente con la ciencia de los datos en cuestión de minutos, sin tener que instalar ni configurar cada una de las herramientas de forma individual. Fácilmente puede escalar verticalmente DSVM si es necesario y puede detenerlo cuando no esté en uso. El recurso de DSVM es tanto elástico como rentable.

En este tutorial analizamos el conjunto de datos [spambase](https://archive.ics.uci.edu/ml/datasets/spambase). Spambase es un conjunto de correos electrónicos que se marcan como correo no deseado o ham (no es correo no deseado). Spambase también contiene algunas estadísticas sobre el contenido de los correos electrónicos. Hablaremos sobre las estadísticas más adelante en el tutorial.

## <a name="prerequisites"></a>Requisitos previos

Antes de usar una instancia de DSVM de Linux, debe cumplir los siguientes requisitos previos:

* **Suscripción de Azure**. Para obtener una suscripción a Azure, consulte [Cree su cuenta gratuita de Azure hoy mismo](https://azure.microsoft.com/free/).

* [**Data Science Virtual Machine para Ubuntu**](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-dsvm.ubuntu-1804) Para más información sobre el aprovisionamiento de la máquina virtual, consulte [Aprovisionamiento de una instancia de Data Science Virtual Machine de Ubuntu](linux-dsvm-intro.md).
* [**X2Go**](https://wiki.x2go.org/doku.php) instalado en su equipo con una sesión abierta de XFCE. Para más información, consulte [Instalación y configuración del cliente X2Go](linux-dsvm-intro.md#x2go).
* Para obtener una experiencia de desplazamiento más fluida, en el explorador web de Firefox de DSVM, alterne la marca `gfx.xrender.enabled` en `about:config`. [Más información](https://www.reddit.com/r/firefox/comments/4nfmvp/ff_47_unbearable_slow_over_remote_x11/). Considere también la posibilidad de establecer `mousewheel.enable_pixel_scrolling` en `False`. [Más información](https://support.mozilla.org/questions/981140).

## <a name="download-the-spambase-dataset"></a>Descarga del conjunto de datos spambase

El conjunto de datos [spambase](https://archive.ics.uci.edu/ml/datasets/spambase) es un conjunto de datos relativamente pequeño que contiene 4601 ejemplos. Este conjunto de datos es adecuado para demostrar algunas de las características principales de DSVM, ya que mantiene los requisitos de recursos en un nivel modesto.

> [!NOTE]
> Este tutorial se ha creado con una versión de DSVM de Linux de tamaño v2 de D2 (Ubuntu 18.04 Edition). Puede usar una instancia de DSVM de este tamaño para completar los procedimientos que se muestran en este tutorial.

Si necesita más espacio de almacenamiento, puede crear discos adicionales y conectarlos a la instancia de DSVM. Los discos usan almacenamiento de Azure persistente, por lo que sus datos se conservan incluso cuando el servidor se reaprovisiona debido a un cambio de tamaño o se apaga. Para agregar un disco y conectarlo a DSVM, complete los pasos que se describen en [Adición de un disco a una máquina virtual de Linux](../../virtual-machines/linux/add-disk.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). En los pasos para agregar un disco se usa la CLI de Azure, que ya está instalada en DSVM. Puede completar los pasos completamente desde la propia instancia de DSVM. Otra opción para aumentar el almacenamiento es usar [Azure Files](../../storage/files/storage-how-to-use-files-linux.md).

Para descargar los datos, abra una ventana de terminal y, a continuación, ejecute este comando:

```bash
wget --no-check-certificate https://archive.ics.uci.edu/ml/machine-learning-databases/spambase/spambase.data
```

El archivo descargado no tiene ninguna fila de encabezado. Vamos a crear otro archivo que tenga un encabezado. Ejecute este comando para crear un archivo con los encabezados adecuados:

```bash
echo 'word_freq_make, word_freq_address, word_freq_all, word_freq_3d,word_freq_our, word_freq_over, word_freq_remove, word_freq_internet,word_freq_order, word_freq_mail, word_freq_receive, word_freq_will,word_freq_people, word_freq_report, word_freq_addresses, word_freq_free,word_freq_business, word_freq_email, word_freq_you, word_freq_credit,word_freq_your, word_freq_font, word_freq_000, word_freq_money,word_freq_hp, word_freq_hpl, word_freq_george, word_freq_650, word_freq_lab,word_freq_labs, word_freq_telnet, word_freq_857, word_freq_data,word_freq_415, word_freq_85, word_freq_technology, word_freq_1999,word_freq_parts, word_freq_pm, word_freq_direct, word_freq_cs, word_freq_meeting,word_freq_original, word_freq_project, word_freq_re, word_freq_edu,word_freq_table, word_freq_conference, char_freq_semicolon, char_freq_leftParen,char_freq_leftBracket, char_freq_exclamation, char_freq_dollar, char_freq_pound, capital_run_length_average,capital_run_length_longest, capital_run_length_total, spam' > headers
```

A continuación, concatene los dos archivos juntos:

```bash
cat spambase.data >> headers
mv headers spambaseHeaders.data
```

El conjunto de datos tiene varios tipos de estadísticas para cada correo electrónico:

* Las columnas, como **word\_freq\__WORD_** indican el porcentaje de palabras en el correo electrónico que coinciden con *WORD*. Por ejemplo, si **word\_freq\_make** es **1**, el 1 % de todas las palabras en el correo electrónico son *make*.
* Las columnas, como **char\_freq\__CHAR_** indican el porcentaje de todos los caracteres en el correo electrónico que son *CHAR*.
* **capital\_run\_length\_longest** es la mayor longitud de una secuencia de letras mayúsculas.
* **capital\_run\_length\_average** es la longitud media de todas las secuencias de letras mayúsculas.
* **capital\_run\_length\_total** es la longitud media de todas las secuencias de letras mayúsculas.
* **spam** indica si el correo electrónico se considera correo no deseado o no (1 = es correo no deseado, 0 = no es correo no deseado).

## <a name="explore-the-dataset-by-using-r-open"></a>Exploración del conjunto de datos mediante R Open

Vamos a examinar los datos y a realizar algunas tareas básicas de aprendizaje automático con R. DSVM viene preinstalado con [Microsoft R Open](https://mran.revolutionanalytics.com/open/). Las bibliotecas matemáticas multiproceso de esta versión preinstalada de R ofrecen un rendimiento mejor que las versiones de un único subproceso. R Open proporciona también reproducibilidad a través de una instantánea del repositorio de paquetes CRAN.

Para obtener copias de los ejemplos de código usados en este tutorial, clone el repositorio Azure-Machine-Learning-Data-Science. Git está preinstalado en DSVM. En la línea de comandos de Git, ejecute:

```bash
git clone https://github.com/Azure/Azure-MachineLearning-DataScience.git
```

Abra una ventana de terminal e inicie una nueva sesión de R en la consola interactiva de R. También puede usar RStudio, que está preinstalado en DSVM.

Para importar los datos y configurar el entorno:

```R
data <- read.csv("spambaseHeaders.data")
set.seed(123)
```

Para ver estadísticas de resumen sobre cada columna:

```R
summary(data)
```

Para tener una vista diferente de los datos:

```R
str(data)
```

Esta vista muestra el tipo de cada variable y algunos de los primeros valores del conjunto de datos.

La columna **spam** se leyó como un entero, pero es en realidad una variable (o factor) categórica. Para establecer su tipo:

```R
data$spam <- as.factor(data$spam)
```

Para realizar un análisis de exploración, use el paquete [ggplot2](https://ggplot2.tidyverse.org/), una conocida biblioteca de gráficos para R que está instalada en DSVM. Según los datos de resumen que se mostraron anteriormente, tenemos estadísticas de resumen sobre la frecuencia del carácter de signo de exclamación. Vamos a trazar aquí esas frecuencias mediante la ejecución de los siguientes comandos:

```R
library(ggplot2)
ggplot(data) + geom_histogram(aes(x=char_freq_exclamation), binwidth=0.25)
```

Puesto que la barra de cero está sesgando el trazado, vamos a eliminarla:

```R
email_with_exclamation = data[data$char_freq_exclamation > 0, ]
ggplot(email_with_exclamation) + geom_histogram(aes(x=char_freq_exclamation), binwidth=0.25)
```

Hay una densidad no trivial por encima de 1 que parece interesante. Echemos un vistazo solo a esos datos:

```R
ggplot(data[data$char_freq_exclamation > 1, ]) + geom_histogram(aes(x=char_freq_exclamation), binwidth=0.25)
```

A continuación, los vamos a dividir en: es correo no deseado y no es correo no deseado:

```R
ggplot(data[data$char_freq_exclamation > 1, ], aes(x=char_freq_exclamation)) +
geom_density(lty=3) +
geom_density(aes(fill=spam, colour=spam), alpha=0.55) +
xlab("spam") +
ggtitle("Distribution of spam \nby frequency of !") +
labs(fill="spam", y="Density")
```

Estos ejemplos deben ayudarle a realizar trazados similares y explorar datos en las otras columnas.

## <a name="train-and-test-a-machine-learning-model"></a>Entrenamiento y prueba de un modelo de Machine Learning

Vamos a entrenar un par de modelos de Machine Learning para clasificar los correos electrónicos del conjunto de datos según si son correo no deseado o no es correo no deseado. En esta sección, entrenamos un modelo de árbol de decisión y un modelo de bosque aleatorio. A continuación, probamos la precisión de las predicciones.

> [!NOTE]
> El paquete *rpart* (árboles de regresión y partición recursiva) usado en el siguiente código ya está instalado en DSVM.

En primer lugar, vamos a dividir el conjunto de datos en conjuntos de entrenamiento y conjuntos de prueba:

```R
rnd <- runif(dim(data)[1])
trainSet = subset(data, rnd <= 0.7)
testSet = subset(data, rnd > 0.7)
```

Luego, crearemos un árbol de decisiones para clasificar los correos electrónicos:

```R
require(rpart)
model.rpart <- rpart(spam ~ ., method = "class", data = trainSet)
plot(model.rpart)
text(model.rpart)
```

Este es el resultado:

![Diagrama del árbol de decisión que se ha creado](./media/linux-dsvm-walkthrough/decision-tree.png)

Para determinar su rendimiento en el conjunto de entrenamiento, use el siguiente código:

```R
trainSetPred <- predict(model.rpart, newdata = trainSet, type = "class")
t <- table(`Actual Class` = trainSet$spam, `Predicted Class` = trainSetPred)
accuracy <- sum(diag(t))/sum(t)
accuracy
```

Para determinar su rendimiento en el conjunto de prueba:

```R
testSetPred <- predict(model.rpart, newdata = testSet, type = "class")
t <- table(`Actual Class` = testSet$spam, `Predicted Class` = testSetPred)
accuracy <- sum(diag(t))/sum(t)
accuracy
```

Vamos a probar un modelo de bosque aleatorio. Los bosques aleatorios entrenan multitud de árboles de decisiones y generan una clase que es el modo de las clasificaciones de todos los árboles de decisiones individuales. Proporcionan un enfoque de aprendizaje automático más poderoso ya que corrigen la tendencia de un modelo de árbol de decisiones a saturar en exceso un conjunto de datos de entrenamiento.

```R
require(randomForest)
trainVars <- setdiff(colnames(data), 'spam')
model.rf <- randomForest(x=trainSet[, trainVars], y=trainSet$spam)

trainSetPred <- predict(model.rf, newdata = trainSet[, trainVars], type = "class")
table(`Actual Class` = trainSet$spam, `Predicted Class` = trainSetPred)

testSetPred <- predict(model.rf, newdata = testSet[, trainVars], type = "class")
t <- table(`Actual Class` = testSet$spam, `Predicted Class` = testSetPred)
accuracy <- sum(diag(t))/sum(t)
accuracy
```

<a name="deep-learning"></a>

## <a name="deep-learning-tutorials-and-walkthroughs"></a>Tutoriales del aprendizaje profundo

Además de los ejemplos basados en marcos, también se proporciona un conjunto de tutoriales completos. con los que puede iniciar el desarrollo de aplicaciones de aprendizaje profundo en ámbitos como la comprensión de lenguajes o de imágenes y texto.

- [Ejecución de redes neuronales en distintos marcos](https://github.com/ilkarman/DeepLearningFrameworks): un completo tutorial que muestra cómo migrar código de un marco a otro. También se muestra cómo comparar el rendimiento del modelo y el entorno de ejecución en varios marcos. 

- [Una guía paso a paso para compilar una solución integral que detecte productos dentro de las imágenes](https://github.com/Azure/cortana-intelligence-product-detection-from-images): la detección de imágenes es una técnica que puede localizar y clasificar objetos dentro de las imágenes. La tecnología tiene el potencial de aportar grandes ventajas a muchos ámbitos comerciales de uso real. Por ejemplo, los minoristas pueden aplicar esta técnica para determinar qué producto ha seleccionado un cliente de una estantería. A su vez, esta información permite que las tiendas administren el inventario de productos. 

- [Aprendizaje profundo para audio](https://blogs.technet.microsoft.com/machinelearning/2018/01/30/hearing-ai-getting-started-with-deep-learning-for-audio-on-azure/): en este tutorial se muestra cómo entrenar un modelo de aprendizaje profundo para detectar eventos de audio en el [conjunto de datos de sonidos urbanos](https://urbansounddataset.weebly.com/). El tutorial proporciona información general sobre cómo trabajar con datos de audio.

- [Clasificación de documentos de texto](https://github.com/anargyri/lstm_han): en este tutorial se demuestra cómo crear y entrenar dos arquitecturas de redes neuronales distintas: red de atención jerárquica y memoria a corto y largo plazo (LSTM). Estas redes neurales usan la API de Keras de aprendizaje profundo para clasificar documentos de texto. Keras es un front-end para tres de los marcos de aprendizaje profundo más populares: Microsoft Cognitive Toolkit, TensorFlow y Theano.

## <a name="other-tools"></a>Otras herramientas

En las secciones restantes se muestra cómo usar algunas de las herramientas instaladas en DSVM de Linux. Trataremos las siguientes herramientas:

* XGBoost
* Python
* JupyterHub
* Rattle
* PostgreSQL y SQuirreL SQL
* Azure Synapse Analytics (anteriormente SQL DW)

### <a name="xgboost"></a>XGBoost

[XGBoost](https://xgboost.readthedocs.org/en/latest/) proporciona una implementación de árbol ampliada rápida y precisa.

```R
require(xgboost)
data <- read.csv("spambaseHeaders.data")
set.seed(123)

rnd <- runif(dim(data)[1])
trainSet = subset(data, rnd <= 0.7)
testSet = subset(data, rnd > 0.7)

bst <- xgboost(data = data.matrix(trainSet[,0:57]), label = trainSet$spam, nthread = 2, nrounds = 2, objective = "binary:logistic")

pred <- predict(bst, data.matrix(testSet[, 0:57]))
accuracy <- 1.0 - mean(as.numeric(pred > 0.5) != testSet$spam)
print(paste("test accuracy = ", accuracy))
```

XGBoost también se puede llamar desde Python o una línea de comandos.

### <a name="python"></a>Python

Para el desarrollo con Python, se han instalado las distribuciones 3.5 y 2.7 de Anaconda Python en DSVM.

> [!NOTE]
> La distribución de Anaconda incluye [Conda](https://conda.pydata.org/docs/index.html). Puede usar Conda para crear entornos de Python personalizados que tengan instalados diferentes paquetes o versiones.

Vamos a leer algunos de los conjuntos de datos spambase y a clasificar los correos electrónicos con máquinas vectoriales de apoyo en Scikit-learn:

```Python
import pandas
from sklearn import svm
data = pandas.read_csv("spambaseHeaders.data", sep = ',\s*')
X = data.ix[:, 0:57]
y = data.ix[:, 57]
clf = svm.SVC()
clf.fit(X, y)
```

Para realizar predicciones:

```Python
clf.predict(X.ix[0:20, :])
```

Para mostrar cómo publicar un punto de conexión de Azure Machine Learning, vamos a crear un modelo más básico. Usaremos las tres variables que usamos al publicar el modelo de R anteriormente:

```Python
X = data[["char_freq_dollar", "word_freq_remove", "word_freq_hp"]]
y = data.ix[:, 57]
clf = svm.SVC()
clf.fit(X, y)
```

### <a name="jupyterhub"></a>JupyterHub

La distribución Anaconda en DSVM viene con Jupyter Notebook, un entorno multiplataforma para compartir código y análisis de Python, R o Julia. Se accede a Jupyter Notebook mediante JupyterHub. Inicie sesión con el nombre de usuario y la contraseña de Linux local en https://\<DSVM DNS name or IP address\>:8000/. Todos los archivos de configuración de JupyterHub se encuentran en /etc/jupyterhub.

> [!NOTE]
> Para usar el Administrador de paquetes de Python (con el comando `pip`) de Jupyter Notebook en el kernel actual, use este comando en la celda de código:
>
>   ```Python
>    import sys
>    ! {sys.executable} -m pip install numpy -y
>   ```
> 
> Para usar el instalador de Conda (con el comando `conda`) de Jupyter Notebook en el kernel actual, use este comando en una celda de código:
>
>   ```Python
>    import sys
>    ! {sys.prefix}/bin/conda install --yes --prefix {sys.prefix} numpy
>   ```

Hay varios cuadernos de muestra ya instalados en DSVM:

* Cuadernos de Python de ejemplo:
  * [IntroToJupyterPython.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Data-Science-Virtual-Machine/Samples/Notebooks/IntroToJupyterPython.ipynb)
* Cuaderno de R de ejemplo:
  * [IntroTutorialinR](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Data-Science-Virtual-Machine/Samples/Notebooks/IntroTutorialinR.ipynb) 

> [!NOTE]
> El lenguaje Julia también está disponible desde la línea de comandos en DSVM de Linux.

### <a name="rattle"></a>Rattle

[Rattle](https://cran.r-project.org/web/packages/rattle/index.html) (*R* *A*nalytical *T*ool *T*o *L*earn *E*asily) es una herramienta gráfica de R para la minería de datos. Rattle presenta una interfaz intuitiva que permite cargar, explorar y transformar los datos y compilar y evaluar modelos de forma fácil. En [Rattle: A Data Mining GUI for R](https://journal.r-project.org/archive/2009-2/RJournal_2009-2_Williams.pdf) (Rattle: una GUI de minería de datos para R), se proporciona un tutorial que muestra sus características.

Ejecute estos comandos para instalar e iniciar Rattle:

```R
if(!require("rattle")) install.packages("rattle")
require(rattle)
rattle()
```

> [!NOTE]
> No es necesario instalar Rattle en DSVM. Sin embargo, es posible que se le pida que instale paquetes adicionales cuando se abra Rattle.

Rattle usa una interfaz de usuario basada en pestañas. La mayoría de las pestañas corresponden a pasos del [Proceso de ciencia de datos en equipo](https://docs.microsoft.com/azure/machine-learning/team-data-science-process/), como cargar los datos o explorarlos. El proceso de ciencia de los datos fluye de izquierda a derecha por las pestañas. La última pestaña contiene un registro de los comandos de R ejecutados por Rattle.

Para cargar y configurar el conjunto de datos:

1. Para cargar el archivo, seleccione la pestaña **Datos**.
1. Elija el selector junto a **Nombre de archivo** y, a continuación, seleccione **spambaseHeaders.data**.
1. Para cargar el archivo seleccione **Ejecutar**. Verá un resumen de cada columna, junto con su tipo de datos identificado, si es una entrada, un destino u otro tipo de variable, y el número de valores únicos.
1. Rattle ha identificado correctamente la columna **correo no deseado** como el destino. Seleccione la columna de **correo no deseado** y luego establezca el **tipo de datos de destino** en **Categoric** (Categórico).

Para explorar los datos:

1. Seleccione la pestaña **Explore** (Explorar).
1. Para ver alguna información sobre los tipos de variables y algunas estadísticas de resumen, seleccione **Resumen** > **Ejecutar**.
1. Para ver otros tipos de estadísticas sobre cada variable, seleccione otras opciones, como **Describir** o **Datos básicos**.

También puede usar la pestaña **Explorar** para generar trazados detallados. Para trazar un histograma de los datos:

1. Seleccione **Distributions**(Distribuciones).
1. Busque **word_freq_remove** y **word_freq_you** y seleccione **Histograma**.
1. Seleccione **Execute**(Ejecutar). Verá ambos trazados de densidad en una sola ventana gráfica, donde está claro que la palabra _you_ aparece con mucha más frecuencia en los correos electrónicos que _remove_.

Los trazados de **correlación** también son interesantes. Para crear un trazado:

1. En **Tipo**, seleccione **Correlación**.
1. Seleccione **Execute**(Ejecutar).
1. Rattle le avisa de que recomienda 40 variables como máximo. Seleccione **Yes** (Sí) para ver la trazado.

Surgen algunas correlaciones interesantes: por ejemplo, _technology_ está estrechamente relacionado con _HP_ y _labs_. También está estrechamente correlacionado con _650_, porque el código de área de los donantes del conjunto de datos es 650.

Los valores numéricos de las correlaciones entre palabras están disponibles en la ventana de **exploración**. Es interesante advertir, por ejemplo, que _technology_ está correlacionada negativamente con _your_ y _money_.

Rattle puede transformar el conjunto de datos para tratar algunos problemas comunes. Por ejemplo, puede volver a escalar características, imputar valores que faltan, administrar los valores atípicos y quitar variables u observaciones con datos que faltan. Rattle también puede identificar reglas de asociación entre observaciones y variables. Estas pestañas no se cubren este tutorial de introducción.

Rattle también puede ejecutar el análisis de clústeres. Vamos a excluir algunas características para que la salida sea más fácil de leer. En la pestaña **Datos**, seleccione **Ignorar** junto a cada una de las variables, excepto estos diez elementos:

* word_freq_hp
* word_freq_technology
* word_freq_george
* word_freq_remove
* word_freq_your
* word_freq_dollar
* word_freq_money
* capital_run_length_longest
* word_freq_business
* spam

Vuelva a la pestaña **Clúster**. Seleccione **KMeans** y, a continuación, establezca el **número de clústeres** en **4**. Seleccione **Execute**(Ejecutar). Los resultados se muestran en la ventana de salida. Un clúster tiene alta frecuencia de _george_ y _hp_ y es probablemente un correo electrónico comercial legítimo.

Para crear un modelo de aprendizaje automático de árbol de decisiones básico:

1. Seleccione la pestaña **Model** (Modelo).
1. En **Tipo**, seleccione **Árbol**.
1. Seleccione **Execute** (Ejecutar) para mostrar el árbol en forma de texto en la ventana de salida.
1. Seleccione el botón **Draw** (Dibujar) para ver una versión gráfica. El árbol de decisión se parece al árbol que obtuvimos anteriormente mediante rpart.

Una característica útil de Rattle es la posibilidad de ejecutar varios métodos de aprendizaje automático y evaluarlos rápidamente. Estos son los pasos que se deben seguir:

1. En **Tipo**, seleccione **Todos**.
1. Seleccione **Execute**(Ejecutar).
1. Cuando Rattle termine de ejecutarse, puede seleccionar cualquier valor de **Tipo**, como **SVM**, y ver los resultados.
1. También puede comparar el rendimiento de los modelos en el conjunto de validación mediante la pestaña **Evaluar**. Por ejemplo, la selección **Error Matrix** (Matriz de errores) muestra la matriz de confusiones, los errores generales y la media de errores de clase de cada modelo en el conjunto de validación. También puede trazar curvas ROC, ejecutar análisis de sensibilidad y llevar a cabo otros tipos de evaluaciones de modelos.

Cuando haya terminado de crear modelos, seleccione la pestaña **Registrar** para ver el código R ejecutado por Rattle durante la sesión. Puede seleccionar el botón **Export** (Exportar) para guardarlo.

> [!NOTE]
> La versión actual de Rattle contiene un error. Para modificar el script o usarlo para repetir los pasos más adelante, debe insertar un carácter **#** delante de *Export this log...* (Exportar este registro...) en el texto del registro.

### <a name="postgresql-and-squirrel-sql"></a>PostgreSQL y SQuirreL SQL

La DSVM viene con PostgreSQL instalado. PostgreSQL es una base de datos relacional sofisticada de código abierto. En esta sección se muestra cómo cargar el conjunto de datos de spambase en PostgreSQL y luego consultarlo.

Antes de cargar los datos, debe permitir la autenticación de contraseña desde el host local. En un símbolo del sistema, ejecute:

```Bash
sudo gedit /var/lib/pgsql/data/pg_hba.conf
```

Cerca de la parte inferior del archivo de configuración, hay varias líneas que detallan las conexiones permitidas:

```
# "local" is only for Unix domain socket connections:
local   all             all                                     trust
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident
```

Cambie la línea **IPv4 local connections** para usar **md5** en lugar de **ident**, así podremos iniciar sesión con un nombre de usuario y una contraseña:

```
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
```

A continuación, reinicie el servicio PostgreSQL:

```Bash
sudo systemctl restart postgresql
```

Para iniciar *psql* (un terminal interactivo para PostgreSQL) con el usuario de Postgres integrado, ejecute el siguiente comando:

```Bash
sudo -u postgres psql
```

Cree una nueva cuenta de usuario con el nombre de usuario de la cuenta de Linux que usó para iniciar sesión. Cree una contraseña:

```Bash
CREATE USER <username> WITH CREATEDB;
CREATE DATABASE <username>;
ALTER USER <username> password '<password>';
\quit
```

Inicie sesión en psql:

```Bash
psql
```

Importe los datos en una base de datos:

```SQL
CREATE DATABASE spam;
\c spam
CREATE TABLE data (word_freq_make real, word_freq_address real, word_freq_all real, word_freq_3d real,word_freq_our real, word_freq_over real, word_freq_remove real, word_freq_internet real,word_freq_order real, word_freq_mail real, word_freq_receive real, word_freq_will real,word_freq_people real, word_freq_report real, word_freq_addresses real, word_freq_free real,word_freq_business real, word_freq_email real, word_freq_you real, word_freq_credit real,word_freq_your real, word_freq_font real, word_freq_000 real, word_freq_money real,word_freq_hp real, word_freq_hpl real, word_freq_george real, word_freq_650 real, word_freq_lab real,word_freq_labs real, word_freq_telnet real, word_freq_857 real, word_freq_data real,word_freq_415 real, word_freq_85 real, word_freq_technology real, word_freq_1999 real,word_freq_parts real, word_freq_pm real, word_freq_direct real, word_freq_cs real, word_freq_meeting real,word_freq_original real, word_freq_project real, word_freq_re real, word_freq_edu real,word_freq_table real, word_freq_conference real, char_freq_semicolon real, char_freq_leftParen real,char_freq_leftBracket real, char_freq_exclamation real, char_freq_dollar real, char_freq_pound real, capital_run_length_average real, capital_run_length_longest real, capital_run_length_total real, spam integer);
\copy data FROM /home/<username>/spambase.data DELIMITER ',' CSV;
\quit
```

Ahora, vamos a explorar los datos y a ejecutar algunas consultas mediante Squirrel SQL, una herramienta gráfica que puede usar para interactuar con bases de datos mediante un controlador JDBC.

Para comenzar, abra SQuirreL SQL desde el menú de **Aplicaciones**. Para configurar el controlador:

1. Seleccione **Windows** > **View Drivers** (Ver controladores).
1. Haga clic con el botón derecho en **PostgreSQL** y seleccione **Modify Driver** (Modificar controlador).
1. Seleccione **Extra Class Path** (Ruta de clase adicional) > **Agregar**.
1. En **Nombre de archivo**, escriba **/usr/share/java/jdbcdrivers/postgresql-9.4.1208.jre6.jar**.
1. seleccione **Open**(Abrir).
1. Seleccione **Mostrar controladores**. En **Nombre de clase**, seleccione **org.postgresql.Driver** y, luego, **Aceptar**.

Para configurar la conexión al servidor local:

1. Seleccione **Windows** > **View Aliases** (Ver alias).
1. Seleccione el botón **+** para crear un nuevo alias. En el nuevo nombre de alias, escriba **base de datos de correo no deseado**. 
1. En **controlador**, seleccione **PostgreSQL**.
1. Establezca la dirección URL en **jdbc:postgresql://localhost/spam**.
1. Escriba su nombre de usuario y contraseña.
1. Seleccione **Aceptar**.
1. Para abrir la ventana **Connection** (Conexión), haga doble clic en el alias **Spam database** (Base de datos de correo no deseado).
1. Seleccione **Conectar**.

Para ejecutar algunas consultas:

1. Seleccione la pestaña **SQL** .
1. En el cuadro de consulta de la parte superior de la pestaña **SQL**, escriba una consulta básica, como `SELECT * from data;`.
1. Presione CTRL + ENTRAR para ejecutar la consulta. SQuirreL SQL devuelve de manera predeterminada las 100 primeras filas de la consulta.

Hay muchas más consultas que se pueden ejecutar para explorar estos datos. Por ejemplo, ¿de qué modo la frecuencia de la palabra *make* es diferente en es correo no deseado y no es correo no deseado?

```SQL
SELECT avg(word_freq_make), spam from data group by spam;
```

O, ¿cuáles son las características del correo electrónico que con frecuencia contiene *3d*?

```SQL
SELECT * from data order by word_freq_3d desc;
```

La mayoría de los mensajes de correo electrónico que tienen una gran repetición de *3d* parecen ser correo no deseado. Esta información puede ser útil para compilar un modelo predictivo para clasificar los correos electrónicos.

Si quiere realizar aprendizaje automático con datos almacenados en una base de datos de PostgreSQL, puede usar [MADlib](https://madlib.incubator.apache.org/).

### <a name="azure-synapse-analytics-formerly-sql-dw"></a>Azure Synapse Analytics (anteriormente SQL DW)

Azure Synapse Analytics es una base de datos de escalado horizontal y basada en la nube que puede procesar volúmenes masivos de datos, tanto relacionales como no relacionales. Para más información, consulte [¿Qué es Azure Synapse Analytics?](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md)

Para conectarse al almacén de datos y crear la tabla, ejecute el siguiente comando desde un símbolo del sistema:

```Bash
sqlcmd -S <server-name>.database.windows.net -d <database-name> -U <username> -P <password> -I
```

En el símbolo del sistema de sqlcmd, ejecute este comando:

```SQL
CREATE TABLE spam (word_freq_make real, word_freq_address real, word_freq_all real, word_freq_3d real,word_freq_our real, word_freq_over real, word_freq_remove real, word_freq_internet real,word_freq_order real, word_freq_mail real, word_freq_receive real, word_freq_will real,word_freq_people real, word_freq_report real, word_freq_addresses real, word_freq_free real,word_freq_business real, word_freq_email real, word_freq_you real, word_freq_credit real,word_freq_your real, word_freq_font real, word_freq_000 real, word_freq_money real,word_freq_hp real, word_freq_hpl real, word_freq_george real, word_freq_650 real, word_freq_lab real,word_freq_labs real, word_freq_telnet real, word_freq_857 real, word_freq_data real,word_freq_415 real, word_freq_85 real, word_freq_technology real, word_freq_1999 real,word_freq_parts real, word_freq_pm real, word_freq_direct real, word_freq_cs real, word_freq_meeting real,word_freq_original real, word_freq_project real, word_freq_re real, word_freq_edu real,word_freq_table real, word_freq_conference real, char_freq_semicolon real, char_freq_leftParen real,char_freq_leftBracket real, char_freq_exclamation real, char_freq_dollar real, char_freq_pound real, capital_run_length_average real, capital_run_length_longest real, capital_run_length_total real, spam integer) WITH (CLUSTERED COLUMNSTORE INDEX, DISTRIBUTION = ROUND_ROBIN);
GO
```

Copie los datos mediante bcp:

```bash
bcp spam in spambaseHeaders.data -q -c -t  ',' -S <server-name>.database.windows.net -d <database-name> -U <username> -P <password> -F 1 -r "\r\n"
```

> [!NOTE]
> El archivo descargado contiene los finales de línea de estilo Windows. La herramienta bcp espera finales de línea de estilo Unix. Use la marca -r para indicárselo a bcp.

A continuación, realice una consulta mediante sqlcmd:

```sql
select top 10 spam, char_freq_dollar from spam;
GO
```

También puede consultar mediante SQuirreL SQL. Siga los pasos similares a PostgreSQL mediante el uso del controlador JDBC de SQL Server. El controlador JDBC está en la carpeta /usr/share/java/jdbcdrivers/sqljdbc42.jar.


