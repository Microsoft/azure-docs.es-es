---
title: 'Referencia: Data Science Virtual Machine para Ubuntu'
titleSuffix: Azure Data Science Virtual Machine
description: Detalles de las herramientas incluidas en la máquina Data Science Virtual Machine de Ubuntu
author: gvashishtha
ms.service: machine-learning
ms.subservice: data-science-vm
ms.custom: devx-track-python
ms.author: gopalv
ms.date: 09/11/2019
ms.topic: reference
ms.openlocfilehash: c919e8abc55dd5cb36325ae060340c1ee147a1a0
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88816310"
---
# <a name="reference-ubuntu-linux-data-science-virtual-machine"></a>Referencia: Data Science Virtual Machine de Ubuntu

A continuación encontrará una lista de las herramientas disponibles en la máquina Data Science Virtual Machine de Ubuntu. 

## <a name="deep-learning-libraries"></a>Bibliotecas de aprendizaje profundo

### <a name="cntk"></a>CNTK

Microsoft Cognitive Toolkit es un kit de herramientas de aprendizaje profundo de código abierto. Los enlaces de Python están disponibles en los entornos raíz y py35 Conda. También tiene una herramienta de línea de comandos (CNTK) que ya se encuentra en la ruta de acceso.

En JupyterHub hay disponibles cuadernos Python de ejemplo. Para ejecutar un ejemplo básico en la línea de comandos, ejecute los comandos siguientes en el shell:

```bash
cd /home/[USERNAME]/notebooks/CNTK/HelloWorld-LogisticRegression
cntk configFile=lr_bs.cntk makeMode=false command=Train
```

Para más información, consulte la sección sobre CNTK de [GitHub](https://github.com/Microsoft/CNTK) y la [wiki de CNTK](https://github.com/Microsoft/CNTK/wiki).

### <a name="caffe"></a>Caffe

Caffe es un entorno de aprendizaje profundo de Berkeley Vision and Learning Center. Está disponible en /opt/caffe. Puede encontrar ejemplos en /opt/caffe/examples.

### <a name="caffe2"></a>Caffe2

Caffe2 es un marco de aprendizaje profundo de Facebook basado en Caffe. Está disponible en Python 2.7 en el entorno root Conda. Para activarlo, ejecute el siguiente comando desde el shell:

```bash
source /anaconda/bin/activate root
```

En JupyterHub hay disponibles algunos cuadernos de ejemplo.

### <a name="h2o"></a>H2O

H2O es una plataforma de análisis predictivo y aprendizaje automático rápida, distribuida y en memoria. Se instala un paquete Python en el entorno raíz y en el entorno py35 Anaconda. También se instala un paquete de R. 

Para abrir H2O desde la línea de comandos, ejecute `java -jar /dsvm/tools/h2o/current/h2o.jar`. Cuenta con varias [opciones de línea de comandos](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/starting-h2o.html#from-the-command-line) que puede configurar. Puede acceder a la interfaz de usuario web de Flow desde `http://localhost:54321` para empezar. En JupyterHub también hay disponibles cuadernos de ejemplo.

### <a name="keras"></a>Keras

Keras es una API de red neuronal de alto nivel en Python. Se puede ejecutar a partir de TensorFlow, Microsoft Cognitive Toolkit o Theano. Está disponible en los entornos root y py35 Python.

### <a name="mxnet"></a>MXNet

MXNet es un entorno de aprendizaje profundo diseñado para lograr eficiencia y flexibilidad. Tiene enlaces de R y Python incluidos en DSVM. Se incluyen cuadernos de ejemplo en JupyterHub y hay código de ejemplo disponible en /dsvm/samples/mxnet.

### <a name="nvidia-digits"></a>NVIDIA DIGITS

El sistema de entrenamiento de GPU de aprendizaje profundo de NVIDIA, conocido como DIGITS, es un sistema que permite simplificar las tareas comunes de aprendizaje profundo. Entre estas tareas se incluyen la administración de datos, el diseño y el entrenamiento de redes neuronal en sistemas GPU, y la supervisión del rendimiento en tiempo real con visualización avanzada.

DIGITS está disponible como un servicio llamado *digits*. Inicie el servicio y vaya a `http://localhost:5000` para comenzar.

DIGITS también está instalado como módulo Python en el entorno raíz Conda.

### <a name="tensorflow"></a>TensorFlow

TensorFlow es la biblioteca de aprendizaje profundo de Google. Es una biblioteca de software de código abierto para cálculo numérico que usa gráficos de flujo de datos. TensorFlow está disponible en el entorno py35 Python y JupyterHub incluye algunos cuadernos de ejemplo.

### <a name="theano"></a>Theano

Theano es una biblioteca Python para lograr un cálculo numérico eficaz. Está disponible en los entornos root y py35 Python. 

### <a name="torch"></a>Torch

Torch es un entorno informático científico con amplia compatibilidad con algoritmos de aprendizaje automático. Está disponible en /dsvm/tools/torch. La sesión interactiva **th** y el administrador de paquetes LuaRocks están disponibles en la línea de comandos. Hay ejemplos disponibles en /dsvm/samples/torch.

PyTorch también está disponible en el entorno raíz Anaconda. Puede encontrar ejemplos en /dsvm/samples/pytorch.

## <a name="microsoft-machine-learning-server"></a>Servidor de Microsoft Machine Learning

R es uno de los lenguajes más conocidos para el análisis de datos y el aprendizaje automático. Si quiere usar R para los análisis, la máquina virtual incluye Microsoft Machine Learning Server con Microsoft R Open y Math Kernel Library. Math Kernel Library optimiza las operaciones matemáticas comunes de los algoritmos analíticos. Microsoft R Open es totalmente compatible con CRAN R; cualquiera de las bibliotecas de R publicadas en CRAN puede instalarse en Microsoft R Open. 

Machine Learning Server ofrece el escalado y la operacionalización de los modelos de R en servicios web. Puede editar los programas de R en uno de los editores predeterminados, como RStudio, vi o Emacs. Si prefiere usar el editor Emacs, ya está instalado. El paquete ESS (Emacs Speaks Statistics) de Emacs simplifica el trabajo con los archivos de R en el editor Emacs.

Para abrir la consola de R, escriba **R** en el shell. Este comando lo dirigirá a un entorno interactivo. Para desarrollar el programa de R, se suele usar un editor como Emacs o vi, y después se ejecutan los scripts en R. Con RStudio tendrá un entorno IDE gráfico completo para desarrollar el programa de R.

También hay un script de R para que pueda instalar los [20 paquetes principales de R](https://www.kdnuggets.com/2015/06/top-20-r-packages.html) si lo desea. Puede ejecutar este script cuando esté en la interfaz interactiva de R. Como se mencionó anteriormente, puede abrir esta interfaz escribiendo **R** en el shell.  

## <a name="python"></a>Python

Anaconda Python se instala con los entornos Python 2.7 y 3.5. El entorno 2.7 se denomina _root_, y el entorno 3.5 se denomina _py35_. Esta distribución contiene Python base, junto con aproximadamente 300 de los paquetes de matemáticas, ingeniería y análisis de datos más populares.

El entorno py35 es el predeterminado. Para activar el entorno root (2.7), use este comando:

```bash
source activate root
```

Para volver a activar el entorno py35, use este comando:

```bash
source activate py35
```

Para invocar una sesión interactiva de Python, escriba **python** en el shell. 

Instale bibliotecas de Python adicionales mediante Conda o pip. Para pip, active primero el entorno correcto si no quiere el predeterminado:

```bash
source activate root
pip install <package>
```

O bien, especifique la ruta de acceso completa a pip:

```bash
/anaconda/bin/pip install <package>
```

En el caso de Conda, siempre debe especificar el nombre del entorno (py35 o root):

```bash
conda install <package> -n py35
```

Si se encuentra en una interfaz gráfica o tiene la configuración de reenvío de X11, puede escribir **pycharm** para abrir el IDE de PyCharm Python. Puede usar los editores de texto predeterminados. Además, puede usar Spyder, un IDE de Python que se incluye con las distribuciones de Anaconda Python. Spyder necesita un escritorio gráfico o el reenvío de X11. El escritorio gráfico tiene un acceso directo a Spyder.

## <a name="jupyter-notebook"></a>Jupyter Notebook

La distribución de Anaconda también incluye un cuaderno de Jupyter Notebook, un entorno para compartir código y análisis. Se accede a Jupyter Notebook mediante JupyterHub. Inicie sesión con su nombre de usuario y contraseña de Linux local.

Se ha configurado previamente el servidor Jupyter Notebook con kernels de Python 2, Python 3 y R. Use el icono de escritorio de **Jupyter Notebook** para abrir el explorador y acceder al servidor de Jupyter Notebook. Si se encuentra en la máquina virtual a través de SSH o del cliente X2Go, también puede acceder al servidor de Jupyter Notebook en `https://localhost:8000/`.

> [!NOTE]
> Continúe aunque aparezca una advertencia de certificado.

Puede acceder al servidor de Jupyter Notebook desde cualquier host. Escriba **https://\<VM DNS name or IP address\>:8000/** .

> [!NOTE]
> El puerto 8000 se abre en el firewall de forma predeterminada cuando se aprovisiona la VM. 

Hemos empaquetado algunos cuadernos de ejemplo (uno en Python y otro en R). Puede ver el vínculo a los ejemplos en la página principal del cuaderno después de que se autentique en Jupyter Notebook con el nombre de usuario y la contraseña de Linux local. Puede crear un nuevo cuaderno seleccionando **New** (Nuevo) y, después, el kernel de lenguaje apropiado. Si no ve el botón **New** (Nuevo), seleccione el icono **Jupyter** en la parte superior izquierda para ir a la página principal del servidor de cuadernos.

## <a name="apache-spark-standalone"></a>Apache Spark independiente

En la instancia de DSVM de Linux hay instalada una instancia independiente de Apache Spark, para ayudarle a desarrollar aplicaciones de Spark localmente antes de probarlas e implementarlas en clústeres de gran tamaño. 

Puede ejecutar programas PySpark mediante el kernel de Jupyter. Al abrir Jupyter, seleccione el botón **New** (Nuevo); verá una lista de los kernels disponibles. **Spark - Python** es el kernel de PySpark que le permite compilar aplicaciones Spark mediante el lenguaje Python. También puede usar un IDE de Python, como PyCharm o Spyder, para compilar el programa Spark. 

En esta instancia independiente, la pila de Spark se ejecuta en el programa cliente que realiza la llamada. Esta característica permite simplificar y acelerar la solución de problemas en comparación con el desarrollo en un clúster de Spark.

Jupyter proporciona un cuaderno de PySpark de ejemplo. Puede encontrarlo en el directorio SparkML en el directorio principal de Jupyter ($HOME/notebooks/SparkML/pySpark). 

Si va a programar en R para Spark, puede usar Microsoft Machine Learning Server, SparkR o sparklyr. 

Antes de ejecutar en un contexto de Spark en Microsoft Machine Learning Server, debe llevar a cabo un paso de configuración una sola vez a fin de habilitar un HDFS de Hadoop de un solo nodo y una instancia de Yarn. De manera predeterminada, los servicios de Hadoop están instalados pero deshabilitados en la DSVM. Para habilitarlos, debe ejecutar los comandos siguientes como raíz la primera vez:

```bash
echo -e 'y\n' | ssh-keygen -t rsa -P '' -f ~hadoop/.ssh/id_rsa
cat ~hadoop/.ssh/id_rsa.pub >> ~hadoop/.ssh/authorized_keys
chmod 0600 ~hadoop/.ssh/authorized_keys
chown hadoop:hadoop ~hadoop/.ssh/id_rsa
chown hadoop:hadoop ~hadoop/.ssh/id_rsa.pub
chown hadoop:hadoop ~hadoop/.ssh/authorized_keys
systemctl start hadoop-namenode hadoop-datanode hadoop-yarn
```

Puede detener los servicios relacionados con Hadoop cuando no los necesite si ejecuta ```systemctl stop hadoop-namenode hadoop-datanode hadoop-yarn```.

El directorio /dsvm/samples/MRS proporciona un ejemplo que muestra cómo desarrollar y probar Microsoft Machine Learning Server en un contexto de Spark remoto (la instancia de Spark independiente en DSVM).

## <a name="ides-and-editors"></a>IDE y editores

Tiene la opción de usar varios editores de código como, por ejemplo, vi/Vim, Emacs, PyCharm, RStudio e IntelliJ. 

PyCharm, RStudio e IntelliJ son editores gráficos. Para usarlos, debe haber iniciado sesión en un escritorio gráfico. Se abren mediante accesos directos de menú de escritorio y aplicación.

Vim y Emacs son editores basados en texto. En Emacs, el paquete de complementos de ESS facilita el trabajo con R en el editor Emacs. Puede encontrar más información en el [sitio web de ESS](https://ess.r-project.org/).

LaTex se instala mediante el paquete texlive, junto con un paquete de complementos de Emacs llamado [AUCTeX](https://www.gnu.org/software/auctex/manual/auctex/auctex.html). Este paquete simplifica la creación de documentos de LaTex en Emacs.  

## <a name="databases"></a>Bases de datos

### <a name="graphical-sql-client"></a>Cliente SQL gráfico

SQuirrel SQL, un cliente SQL gráfico, se puede conectar a diversas bases de datos (como Microsoft SQL Server y MySQL) y ejecutar consultas SQL. Puede ejecutar SQuirrel SQL desde una sesión de escritorio gráfico (a través del cliente de X2Go, por ejemplo) mediante un icono de escritorio. O bien, puede ejecutar el cliente mediante el comando siguiente en el shell:

```bash
/usr/local/squirrel-sql-3.7/squirrel-sql.sh
```

Antes de usarlo por primera vez, configure los controladores y los alias de las bases de datos. Los controladores JDBC se encuentran en /usr/share/java/jdbcdrivers.

Para obtener más información, consulte [SQuirrel SQL](http://squirrel-sql.sourceforge.net/index.php?page=screenshots).

### <a name="command-line-tools-for-accessing-microsoft-sql-server"></a>Herramientas de línea de comandos para tener acceso a Microsoft SQL Server

El paquete de controladores ODBC para SQL Server también incluye dos herramientas de línea de comandos:

- **bcp**: la herramienta bcp copia datos de forma masiva entre una instancia de Microsoft SQL Server y un archivo de datos en un formato especificado por el usuario. Puede usarla para importar grandes cantidades de filas nuevas en tablas de SQL Server o para exportar datos de tablas a archivos de datos. Para importar datos en una tabla, debe usar un archivo de formato creado para esa tabla. O bien, debe comprender la estructura de la tabla y los tipos de datos que son válidos para sus columnas.

  Puede encontrar más información en [Conexión con bcp](https://msdn.microsoft.com/library/hh568446.aspx).

- **sqlcmd**: esta herramienta le permite escribir instrucciones Transact-SQL. También puede especificar procedimientos del sistema y archivos de script en el símbolo del sistema. Esta herramienta usa ODBC para ejecutar lotes de Transact-SQL.

  Puede encontrar más información en [Conexión con sqlcmd](https://msdn.microsoft.com/library/hh568447.aspx).

  > [!NOTE]
  > Existen algunas diferencias en esta herramienta entre las plataformas Windows y Linux. Consulte la documentación de para obtener más información.

### <a name="database-access-libraries"></a>Bibliotecas de acceso a las bases de datos

En R y Python dispone de bibliotecas de acceso a las bases de datos:

* En R, puede usar el paquete RODBC o el paquete dplyr para consultar o ejecutar instrucciones SQL en el servidor de bases de datos.
* En Python, la biblioteca pyodbc proporciona acceso a las bases de datos con ODBC como capa subyacente.  

## <a name="azure-tools"></a>Herramientas de Azure

En la VM se instalan las siguientes herramientas de Azure:

* **CLI de Azure**: puede usar la interfaz de la línea de comandos de Azure para crear y administrar recursos de Azure mediante comandos de shell. Para abrir las herramientas de Azure, escriba **azure help**. Para obtener más información, consulte la [página de documentación de la CLI de Azure](https://docs.microsoft.com/cli/azure/get-started-with-az-cli2).
* **Explorador de Azure Storage** el Explorador de Azure Storage es una herramienta gráfica que se usa para examinar los objetos que se han almacenado en la cuenta de Azure Storage y para cargar y descargar datos con los blobs de Azure como origen y destino. Puede acceder al Explorador de Storage desde el icono de acceso directo del escritorio. También puede abrirlo desde un símbolo del sistema del shell si escribe **StorageExplorer**. Debe haber iniciado sesión desde un cliente X2Go o tener la configuración de reenvío de X11.
* **Bibliotecas de Azure**: a continuación, figuran algunas de las bibliotecas preinstaladas.
  
  * **Python**: las bibliotecas relacionadas con Azure en Python son *azure*, *azureml*, *pydocumentdb* y *pyodbc*. Las tres primeras bibliotecas permiten acceder a los servicios de Azure Storage, a Azure Machine Learning y a Azure Cosmos DB (una base de datos NoSQL en Azure). La cuarta biblioteca, pyodbc (junto con el controlador ODBC de Microsoft para SQL Server), permite el acceso a SQL Server, Azure SQL Database y Azure SQL Data Warehouse desde Python mediante una interfaz ODBC. Escriba **pip list** para ver todas las bibliotecas enumeradas. Asegúrese de ejecutar este comando en los entornos de Python 2.7 y 3.5.
  * **R**: las bibliotecas relacionadas con Azure en R son AzureML y RODBC.
  * **Java**: la lista de bibliotecas de Java para Azure se puede encontrar en el directorio /dsvm/sdk/AzureSDKJava de la máquina virtual. Las bibliotecas principales son Azure Storage y las API de administración, Azure Cosmos DB y los controladores JDBC para SQL Server.  

Puede acceder a [Azure Portal](https://portal.azure.com) desde el explorador Firefox instalado previamente. En Azure Portal, puede crear, administrar y supervisar los recursos de Azure.

## <a name="azure-machine-learning"></a>Azure Machine Learning

Azure Machine Learning es un servicio en la nube totalmente administrado que permite crear, implementar y compartir soluciones de análisis predictivo. Puede crear sus experimentos y modelos desde Azure Machine Learning Studio (versión preliminar). Puede acceder a él desde un explorador web en la instancia de Data Science Virtual Machine si visita la página de [Microsoft Azure Machine Learning](https://ml.azure.com).

Después de iniciar sesión en Azure Machine Learning Studio, puede usar un lienzo de experimentación para crear un flujo lógico para los algoritmos de aprendizaje automático. También tiene acceso a un cuaderno de Jupyter Notebook hospedado en Azure Machine Learning, que puede funcionar perfectamente con los experimentos de Machine Learning Studio. 

Haga que los modelos de aprendizaje automático que ha creado estén operativos encapsulándolos en una interfaz de servicio web. El funcionamiento de modelos de aprendizaje automático permite a los clientes escritos en cualquier lenguaje invocar predicciones a partir de esos modelos. Para obtener más información, consulte la [documentación sobre Machine Learning](https://azure.microsoft.com/documentation/services/machine-learning/).

También puede crear los modelos en R o en Python en la máquina virtual y, después, implementarlos en producción en Azure Machine Learning. Hemos instalado bibliotecas en R (**AzureML**) y en Python (**azureml**) para habilitar esta funcionalidad.

> [!NOTE]
> Estas instrucciones se han escrito para la versión de Windows de Data Science Virtual Machine. Sin embargo, la información proporcionada sobre la implementación de modelos en Azure Machine Learning es aplicable a la máquina virtual Linux.

## <a name="machine-learning-tools"></a>Herramientas de aprendizaje automático

La máquina virtual incluye herramientas y algoritmos de aprendizaje automático que ya se han compilado e instalado de forma local. Entre ellas se incluyen las siguientes:

* **Vowpal Wabbit**: algoritmo de aprendizaje rápido en línea.
* **xgboost**: herramienta que proporciona los algoritmos de árbol ampliados y optimizados.
* **Rattle**: herramienta gráfica basada en R para facilitar el modelado y la exploración de datos.
* **Python**: Anaconda Python integra algoritmos de aprendizaje automático con bibliotecas como Scikit-learn. Puede instalar otras bibliotecas con el comando `pip install` .
* **LightGBM**: entorno de potenciación de gradientes rápido, distribuido y de alto rendimiento basado en algoritmos de árbol de decisión.
* **R**: una amplia biblioteca de funciones de aprendizaje automático está disponible para R. Las bibliotecas preinstaladas incluyen lm, glm, randomForest y rpart. Puede instalar otras bibliotecas mediante la ejecución de este comando:

    ```r
    install.packages(<lib name>)
    ```

A continuación, proporcionamos información adicional sobre las tres primeras herramientas de aprendizaje automático de la lista.

### <a name="vowpal-wabbit"></a>Vowpal Wabbit

Vowpal Wabbit es un sistema de aprendizaje automático rápido que emplea varias técnicas, como el aprendizaje en línea, el uso de hash, la clase AllReduce, las reducciones, learning2search y el aprendizaje activo e interactivo.

Para ejecutar la herramienta en un ejemplo básico, use los siguientes comandos:

```bash
cp -r /dsvm/tools/VowpalWabbit/demo vwdemo
cd vwdemo
vw house_dataset
```

Puede encontrar otras demostraciones más grandes en ese directorio. Para más información sobre Vowpal Wabbit, consulte [esta sección de GitHub](https://github.com/JohnLangford/vowpal_wabbit) y la [wiki de Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki).

### <a name="xgboost"></a>xgboost

Se trata de una biblioteca que está diseñada y optimizada para algoritmos potenciados (de árbol). El objetivo de esta biblioteca es llevar los límites de cálculo de las máquinas a los puntos de conexión necesarios para proporcionar una ampliación de árboles a gran escala que sea escalable, portátil y precisa.

Se proporciona como una línea de comandos y como una biblioteca de R. Para usar esta biblioteca en R, puede iniciar una sesión interactiva de R (escribiendo **R** en el shell) y cargar la biblioteca.

Este es un ejemplo sencillo que puede ejecutar en un símbolo del sistema de R:

```R
library(xgboost)

data(agaricus.train, package='xgboost')
data(agaricus.test, package='xgboost')
train <- agaricus.train
test <- agaricus.test
bst <- xgboost(data = train$data, label = train$label, max.depth = 2,
                eta = 1, nthread = 2, nround = 2, objective = "binary:logistic")
pred <- predict(bst, test$data)
```

Aquí se muestran los comandos que se deben ejecutar en el shell para ejecutar la línea de comandos xgboost:

```bash
cp -r /dsvm/tools/xgboost/demo/binary_classification/ xgboostdemo
cd xgboostdemo
xgboost mushroom.conf
```

En el directorio especificado se escribe un archivo .model. Puede encontrar información sobre este ejemplo de demostración [en GitHub](https://github.com/dmlc/xgboost/tree/master/demo/binary_classification).

Para más información sobre xgboost, consulte la [página de documentación de xgboost](https://xgboost.readthedocs.org/en/latest/) y su [repositorio de GitHub](https://github.com/dmlc/xgboost).

### <a name="rattle"></a>Rattle

Rattle (siglas del inglés **R** **A**nalytical **T**ool **T**o **L**earn **E**asily, la herramienta de análisis de R para aprender fácilmente) utiliza el modelado y la exploración de datos basados en GUI. Presenta resúmenes visuales y estadísticos de datos, transforma datos que pueden modelarse fácilmente, crea modelos supervisados y no supervisados a partir de los datos, presenta el rendimiento de los modelos gráficamente y puntúa los conjuntos de datos nuevos. También genera un código R replicando las operaciones en la interfaz de usuario que pueden ejecutarse directamente en R o usarse como un punto de inicio para un análisis posterior.

Para ejecutar Rattle, debe haber iniciado sesión en un escritorio gráfico. En el terminal, escriba **R** para abrir el entorno de R. En el símbolo del sistema de R, escriba los siguientes comandos:

```R
library(rattle)
rattle()
```

Se abrirá una interfaz gráfica con un conjunto de pestañas. Siga estos pasos de inicio rápido en Rattle para usar un conjunto de datos meteorológicos de ejemplo y crear un modelo. En algunos de los pasos, se le solicitará que instale y cargue automáticamente algún paquete de R necesario que todavía no esté en el sistema.

> [!NOTE]
> Si no tiene acceso para instalar el paquete en el directorio del sistema (valor predeterminado), puede que aparezca un aviso en la ventana de la consola de R para instalar los paquetes en su biblioteca personal. Si ve estos avisos, responda **y** .

1. Seleccione **Execute**(Ejecutar).
1. Aparece un cuadro de diálogo que le pregunta si desea utilizar el conjunto de datos meteorológicos de ejemplo. Haga clic en **Yes** (Sí) para cargar el ejemplo.
1. Seleccione la pestaña **Model** (Modelo).
1. Seleccione **Execute** (Ejecutar) para crear un árbol de decisión.
1. Seleccione **Draw** (Dibujar) para mostrar el árbol de decisión.
1. Seleccione la opción **Forest** (Bosque) y, luego, seleccione **Execute** (Ejecutar) para crear un bosque aleatorio.
1. Seleccione la pestaña **Evaluate** (Evaluar).
1. Seleccione la opción **Risk** (Riesgo) y, luego, **Execute** (Ejecutar) para mostrar dos trazados de rendimiento de **Risk (Cumulative)** (Riesgo [acumulativo]).
1. Seleccione la pestaña **Log** (Registro) para mostrar el código R generado con las operaciones anteriores.
   Debido a un error de la versión actual de Rattle, debe insertar un carácter **#** delante de **Export this log** (Exportar este registro) en el texto del registro.
1. Seleccione el botón **Export** (Exportar) para guardar el archivo de script de R llamado *weather_script.R* en la carpeta principal.

Puede salir de Rattle y R. Ahora puede modificar el script de R generado. O bien, use el script tal y como está y ejecútelo en cualquier momento para repetir todo lo que se hizo en la interfaz de usuario de Rattle. Esta es una forma sencilla, especialmente para los principiantes en R, de realizar análisis y aprendizaje automático rápidamente en una interfaz gráfica sencilla, al mismo tiempo que se genera automáticamente código en R para modificar o aprender.

## <a name="next-steps"></a>Pasos siguientes

¿Tiene más preguntas? Considere la posibilidad de crear [una incidencia de soporte técnico](https://azure.microsoft.com/support/create-ticket/).
