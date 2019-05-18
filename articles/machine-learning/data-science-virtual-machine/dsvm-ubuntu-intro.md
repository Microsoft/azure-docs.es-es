---
title: Creación de una instancia de Data Science Virtual Machine de Ubuntu Linux
titleSuffix: Azure
description: Configure y cree una instancia de Data Science Virtual Machine para Linux (Ubuntu) en Azure para realizar análisis y aprendizaje automático.
services: machine-learning
documentationcenter: ''
author: gopitk
ms.author: gokuma
manager: cgronlun
ms.custom: seodec18
ms.assetid: 3bab0ab9-3ea5-41a6-a62a-8c44fdbae43b
ms.service: machine-learning
ms.subservice: data-science-vm
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/16/2018
ms.openlocfilehash: d693af5ddd0b94734dfa065bef8a1f3a5de09ccc
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65597126"
---
# <a name="provision-the-data-science-virtual-machine-for-linux-ubuntu"></a>Aprovisionamiento de Data Science Virtual Machine para Linux (Ubuntu)

Data Science Virtual Machine para Linux es una imagen de máquina virtual basada en Ubuntu que facilita comenzar a trabajar con aprendizaje automático, incluido el aprendizaje profundo, en Azure. Las herramientas de aprendizaje profundo incluyen las siguientes:

* [Caffe](https://caffe.berkeleyvision.org/): entorno de aprendizaje profundo diseñado para lograr velocidad, expresividad y modularidad
* [Caffe2](https://github.com/caffe2/caffe2): versión multiplataforma de Caffe
* [Microsoft Cognitive Toolkit](https://github.com/Microsoft/CNTK): kit de herramientas de software de Deep Learning de Microsoft Research
* [H2O](https://www.h2o.ai/): interfaz gráfica de usuario y plataforma de macrodatos de código abierto
* [Keras](https://keras.io/): API de red neuronal de alto nivel en Python para TensorFlow, Microsoft Cognitive Toolkit y Theano
* [MXNet](https://mxnet.io/): biblioteca de aprendizaje profundo flexible y eficaz con muchos enlaces de lenguaje
* [NVIDIA DIGITS](https://developer.nvidia.com/digits): sistema gráfico que simplifica las tareas comunes de aprendizaje profundo
* [PyTorch](https://pytorch.org/): una biblioteca de Python de alto nivel con compatibilidad para redes dinámicas
* [TensorFlow](https://www.tensorflow.org/): biblioteca de código abierto para inteligencia automática de Google
* [Theano](http://deeplearning.net/software/theano/): biblioteca de Python para definir, optimizar y evaluar de manera eficaz expresiones matemáticas que implican matrices multidimensionales
* [Torch](http://torch.ch/): entorno informático científico con amplia compatibilidad con algoritmos de aprendizaje automático
* CUDA, cuDNN y el controlador NVIDIA
* Muchos cuadernos de Jupyter Notebook de ejemplo

Todas las bibliotecas son las versiones GPU, aunque también se ejecutan en la CPU.

Data Science Virtual Machine para Linux también contiene herramientas conocidas para actividades de desarrollo y ciencia de datos, entre las que se incluyen:

* Microsoft R Server Developer Edition con Microsoft R Open
* Distribución de Anaconda Python (versiones 2.7 y 3.5), incluidas las bibliotecas de análisis de datos más conocidas
* JuliaPro : distribución protegida del lenguaje Julia con bibliotecas populares científicas y de análisis de datos
* Instancia independiente de Spark y Hadoop de un solo nodo (HDFS, Yarn)
* JupyterHub: servidor de Jupyter Notebook multiusuario compatible con kernels de R, Python, PySpark y Julia
* Explorador de Azure Storage
* Interfaz de la línea de comandos de Azure (CLI) para administrar recursos de Azure
* Herramientas de aprendizaje automático
  * [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit): sistema de aprendizaje automático rápido que admite varias técnicas, como el aprendizaje en línea, el uso de hash, la clase AllReduce, las reducciones, learning2search y los aprendizajes activo e interactivo
  * [XGBoost](https://xgboost.readthedocs.org/en/latest/): herramienta que proporciona una implementación de árbol ampliada, rápida y precisa
  * [Rattle](https://togaware.com/rattle/): herramienta gráfica que facilita comenzar a trabajar con análisis de datos y aprendizaje automático
  * [LightGBM](https://github.com/Microsoft/LightGBM): entorno de potenciación de gradientes rápido, distribuido y de alto rendimiento
* Azure SDK en Java, Python, node.js, Ruby, PHP
* Bibliotecas en R y Python para usarlas en Azure Machine Learning y en otros servicios de Azure
* Editores y herramientas de desarrollo (RStudio, PyCharm, IntelliJ, Emacs, vim)

La ciencia de datos implica la iteración de una secuencia de tareas:

1. Buscar, cargar y preprocesar datos
1. Compilar y probar modelos
1. Implementar los modelos para consumirse en aplicaciones inteligentes

Los científicos de datos usan varias herramientas para realizar estas tareas. Puede llevar bastante tiempo encontrar las versiones del software adecuadas y, después, descargarlas, compilarlas e instalarlas.

Data Science Virtual Machine para Linux puede facilitar considerablemente esta carga. Úsela para comenzar de inmediato su proyecto de análisis. Le permite trabajar en tareas en varios lenguajes, incluidos R, Python, SQL, Java y C++. El Azure SDK incluido en la máquina virtual permite compilar aplicaciones con varios servicios en Linux para la plataforma en la nube de Microsoft. Además, tiene acceso a otros lenguajes como Ruby, Perl, PHP y node.js, que también están instalados previamente.

No hay ningún cargo de software para esta imagen de VM de ciencia de datos. Solo se pagan las cuotas de uso del hardware de Azure, que se calculan en función del tamaño de la máquina virtual que aprovisiona. Puede encontrar más información sobre cuotas de proceso en la [página de listas de VM en Azure Marketplace](https://azure.microsoft.com/marketplace/partners/microsoft-ads/linux-data-science-vm/).

## <a name="other-versions-of-the-data-science-virtual-machine"></a>Otras versiones de Data Science Virtual Machine

También hay una imagen de [CentOS](linux-dsvm-intro.md) disponible, con la mayoría de las mismas herramientas que la imagen de Ubuntu. También hay una imagen de [Windows](provision-vm.md) disponible.

## <a name="prerequisites"></a>Requisitos previos

Antes de que pueda crear una instancia de Data Science Virtual Machine para Linux, debe tener una suscripción a Azure. Para obtener una, consulte [Obtención de una evaluación gratuita de Azure](https://azure.microsoft.com/free/).

## <a name="create-your-data-science-virtual-machine-for-linux"></a>Creación de la instancia de Data Science Virtual Machine para Linux

A continuación le indicamos los pasos para crear una instancia de Data Science Virtual Machine para Linux:

1. Navegue al listado de máquinas virtuales en el [Portal de Azure](https://portal.azure.com/#create/microsoft-dsvm.linux-data-science-vm-ubuntulinuxdsvmubuntu). Es posible que se le pida que inicie sesión en su cuenta de Azure si todavía no lo ha hecho. 
1. Haga clic en **Crear** (botón ubicado en la parte inferior) para acceder al asistente.![configure-data-science-vm](./media/dsvm-ubuntu-intro/configure-data-science-virtual-machine.png)
1. En las secciones siguientes se proporcionan las entradas para cada uno de los pasos del asistente (enumerados a la derecha de la figura anterior) que se siguieron para crear la Microsoft Data Science Virtual Machine. Estas son las entradas necesarias para configurar cada uno de estos pasos:

    a. **Aspectos básicos**:

   * **Nombre**: nombre del servidor de ciencia de datos que está creando.
   * **Tipo de disco de máquina virtual**: elija **SSD Premium** si prefiere una unidad de estado sólido (SSD). De lo contrario, elija **HDD estándar**. 
   * **Nombre de usuario**: identificador de acceso de la primera cuenta.
   * **Contraseña**: contraseña de la primera cuenta (puede usar una clave pública SSH en lugar de una contraseña).
   * **Suscripción**: Si tiene más de una suscripción, seleccione aquella en la que se creará y facturará la máquina. Debe tener privilegios de creación de recursos en esta suscripción.
   * **Grupos de recursos**: Puede crear uno nuevo o utilizar un grupo ya existente.
   * **Ubicación**: seleccione el centro de datos más adecuado. Normalmente, es el centro de datos que tenga la mayoría de los datos o el que esté más cerca de su ubicación física para disfrutar de un acceso más rápido a la red.

   b. **Tamaño**:

   * Seleccione uno de los tipos de servidor que cumpla sus requisitos funcionales y las restricciones de costo. Seleccione un NC o ND clase VM para las instancias de máquina virtual basada en GPU. La página [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/) muestra las regiones con GPU.

   c. **Configuración**:

   * En la mayoría de los casos, puede usar simplemente los valores predeterminados. Si se plantea utilizar valores no predeterminados, mueva el puntero sobre el vínculo informativo para obtener ayuda sobre los campos específicos.

   d. **Resumen**:

   * Compruebe que toda la información que ha especificado es correcta. Se proporciona un vínculo a las condiciones de uso. La máquina virtual no tiene ningún cargo adicional más allá del proceso para el tamaño del servidor que eligió en el paso **Tamaño** . Para iniciar el aprovisionamiento, haga clic en **Crear**. 

El aprovisionamiento tardará alrededor de 5 minutos. El estado del aprovisionamiento se muestra en el Portal de Azure.

## <a name="how-to-access-the-data-science-virtual-machine-for-linux"></a>Acceso a Data Science Virtual Machine para Linux

Puede tener acceso a la DSVM Ubuntu mediante tres métodos:

1. SSH para sesiones de terminal
1. X2Go para sesiones de gráficas
1. JupyterHub y JupyterLab para instancias de Jupyter Notebook

También puede adjuntar una máquina virtual de ciencia de datos a Azure Notebooks para ejecutar cuadernos de Jupyter en la máquina virtual y omitir las limitaciones del nivel de servicio gratuito. Para obtener más información, consulte [administrar y configurar proyectos de blocs de notas: nivel de proceso](/azure/notebooks/configure-manage-azure-notebooks-projects.md#compute-tier).

### <a name="ssh"></a>SSH

Después de crear la máquina virtual, puede iniciar sesión en ella mediante SSH. Utilice las credenciales de la cuenta que haya creado en la sección **Aspectos básicos** del paso 3 para la interfaz de shell de texto. En Windows, puede descargar una herramienta de cliente SSH como [Putty](https://www.putty.org). Si prefiere un escritorio gráfico (X Windows System), puede usar el reenvío de X11 en Putty o instalar el cliente X2Go.

> [!NOTE]
> El cliente X2Go ha tenido un mejor rendimiento que el reenvío de X11 durante las pruebas. Por lo tanto, se recomienda usar el cliente X2Go para la interfaz gráfica de escritorio.

### <a name="x2go"></a>X2Go

La VM de Linux ya está provista del servidor X2Go y está preparada para aceptar conexiones de cliente. Para conectarse al escritorio gráfico de la máquina virtual Linux, lleve a cabo el siguiente procedimiento en el cliente:

1. Descargue e instale el cliente X2Go para su plataforma cliente desde [aquí](https://wiki.x2go.org/doku.php/doc:installation:x2goclient).
1. Ejecute el cliente X2Go y seleccione **Nueva sesión**. Se abrirá una ventana de configuración con varias pestañas. Escriba los siguientes parámetros de configuración:
   * **Pestaña Sesión**:
     * **Host**: nombre de host o dirección IP de la Linux Data Science Virtual Machine.
     * **Inicio de sesión**: nombre de usuario en la máquina virtual Linux.
     * **Puerto SSH**: déjelo en 22, el valor predeterminado.
     * **Tipo de sesión**: cambie el valor a XFCE. Actualmente, la máquina virtual Linux solo admite el escritorio XFCE.
   * **Pestaña Multimedia**: puede desactivar la compatibilidad de sonido y la impresión en el cliente si no necesita usarlas.
   * **Carpetas compartidas**: si quiere que los directorios de las máquinas cliente se monten en la VM Linux, agregue en esta pestaña los directorios de máquina cliente que quiere compartir con la VM.

Una vez que inicie sesión en la máquina virtual mediante el cliente SSH o el escritorio gráfico XFCE a través del cliente X2Go, ya podrá empezar a usar las herramientas que están instaladas y configuradas en la máquina virtual. En XFCE, puede ver accesos directos del menú de aplicaciones e iconos de escritorio para muchas de las herramientas.

### <a name="jupyterhub-and-jupyterlab"></a>JupyterHub y JupyterLab

La DSVM DE Ubuntu ejecuta [JupyterHub](https://github.com/jupyterhub/jupyterhub), un servidor de Jupyter multiusuario. Para conectarse, vaya a https:\// su-vm-ip:8000 en su equipo portátil o escritorio, escriba el nombre de usuario y la contraseña que usó para crear la máquina virtual e inicie sesión. Puede elegir algún cuaderno de ejemplo (hay muchos disponibles) y probar.

JupyterLab, la siguiente generación de los cuadernos de Jupyter y JupyterHub, también está disponible. Para acceder a ella, inicie sesión en JupyterHub y, a continuación, vaya a la dirección URL https:\// su-vm-ip:8000/usuario/your-nombre de usuario/laboratorio. Puede establecer JupyterLab como servidor de cuadernos predeterminado agregando esta línea a */etc/jupyterhub/jupyterhub_config.py*:

```python
c.Spawner.default_url = '/lab'
```

## <a name="tools-installed-on-the-data-science-virtual-machine-for-linux"></a>Herramientas instaladas en Data Science Virtual Machine para Linux

### <a name="deep-learning-libraries"></a>Bibliotecas de aprendizaje profundo

#### <a name="cntk"></a>CNTK

Microsoft Cognitive Toolkit es un kit de herramientas de aprendizaje profundo de código abierto. Los enlaces de Python están disponibles en los entornos raíz y py35 Conda. También tiene una herramienta de línea de comandos (cntk) que ya se encuentra en la ruta PATH.

En JupyterHub hay disponibles cuadernos Python de ejemplo. Para ejecutar un ejemplo básico en la línea de comandos, ejecute los comandos siguientes en el shell:

```bash
cd /home/[USERNAME]/notebooks/CNTK/HelloWorld-LogisticRegression
cntk configFile=lr_bs.cntk makeMode=false command=Train
```

Para obtener más información, consulte la sección sobre CNTK de [GitHub](https://github.com/Microsoft/CNTK) y la [wiki de CNTK](https://github.com/Microsoft/CNTK/wiki).

#### <a name="caffe"></a>Caffe

Caffe es un entorno de aprendizaje profundo de Berkeley Vision and Learning Center. Está disponible en /opt/caffe. Puede encontrar ejemplos en /opt/caffe/examples.

#### <a name="caffe2"></a>Caffe2

Caffe2 es un marco de aprendizaje profundo de Facebook basado en Caffe. Está disponible en Python 2.7 en el entorno de raíz Conda. Para activarlo, ejecute el siguiente comando desde el shell:

```bash
source /anaconda/bin/activate root
```

En JupyterHub hay disponibles algunos cuadernos de ejemplo.

#### <a name="h2o"></a>H2O

H2O es una plataforma de análisis predictivo y aprendizaje automático rápida, distribuida y en memoria. Se instala un paquete Python en el entorno raíz y en el entorno py35 Anaconda. También se instala un paquete de R. Para iniciar H2O desde la línea de comandos, ejecute `java -jar /dsvm/tools/h2o/current/h2o.jar`; hay distintas [opciones de línea de comandos](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/starting-h2o.html#from-the-command-line) que podría configurar. Se puede acceder a la interfaz de usuario web de Flow desde http://localhost:54321 para empezar. En JupyterHub también hay disponibles cuadernos de ejemplo.

#### <a name="keras"></a>Keras

Keras es una API de red neuronal de alto nivel en Python capaz de ejecutarse a partir de TensorFlow, Microsoft Cognitive Toolkit o Theano. Está disponible en los entornos raíz y py35 Python.

#### <a name="mxnet"></a>MXNet

MXNet es un entorno de aprendizaje profundo diseñado para lograr eficiencia y flexibilidad. Tiene enlaces de R y Python incluidos en DSVM. Se incluyen cuadernos de ejemplo en JupyterHub y hay código de ejemplo disponible en /dsvm/samples/mxnet.

#### <a name="nvidia-digits"></a>NVIDIA DIGITS

NVIDIA Deep Learning GPU Training System, conocido como DIGITS, es un sistema que simplifica las tareas comunes de aprendizaje profundo como administrar datos, diseñar y entrenar redes neuronales en sistemas GPU y supervisar el rendimiento en tiempo real con visualización avanzada.

DIGITS está disponible como un servicio llamado digits. Inicie el servicio y vaya a http://localhost:5000 para comenzar.

DIGITS también está instalado como módulo Python en el entorno raíz Conda.

#### <a name="tensorflow"></a>TensorFlow

TensorFlow es la biblioteca de aprendizaje profundo de Google. Es una biblioteca de software de código abierto para cálculo numérico con gráficos de flujo de datos. TensorFlow está disponible en el entorno py35 Python y JupyterHub incluye algunos cuadernos de ejemplo.

#### <a name="theano"></a>Theano

Theano es una biblioteca Python para lograr un cálculo numérico eficaz. Está disponible en los entornos raíz y py35 Python. 

#### <a name="torch"></a>Torch

Torch es un entorno informático científico con amplia compatibilidad con algoritmos de aprendizaje automático. Está disponible en /dsvm/tools/torch y la sesión interactiva th y el administrador de paquetes luarocks están disponibles en la línea de comandos. Hay ejemplos disponibles en /dsvm/samples/torch.

PyTorch también está disponible en el entorno raíz Anaconda. Puede encontrar ejemplos en /dsvm/samples/pytorch.

### <a name="microsoft-r-server"></a>Microsoft R Server

R es uno de los lenguajes más conocidos para el análisis de datos y el aprendizaje automático. Si quiere usar R para el análisis, la máquina virtual incluye Microsoft R Server (MRS) con Microsoft R Open (MRO) y la biblioteca Math Kernel Library (MKL). MKL optimiza las operaciones matemáticas comunes en algoritmos de análisis. MRO es totalmente compatible con CRAN-R; cualquiera de las bibliotecas de R publicadas en CRAN puede instalarse en MRO. MRS ofrece el escalado y la operacionalización de los modelos de R en servicios web. Puede editar los programas de R en uno de los editores predeterminados, como RStudio, vi o Emacs. Si prefiere usar el editor Emacs, ya está instalado. El paquete de Emacs ESS (Emacs Speaks Statistics) simplifica el trabajo con los archivos de R en el editor Emacs.

Para iniciar la consola de R, simplemente escriba **R** en el shell. Este comando le lleva a un entorno interactivo. Para desarrollar el programa de R, se suele usar un editor como Emacs o vi, y se ejecutan los scripts en R. Con RStudio, tendrá un entorno de IDE gráfico completo para desarrollar su programa de R.

También hay un script de R para que pueda instalar los [20 paquetes principales de R](https://www.kdnuggets.com/2015/06/top-20-r-packages.html) si lo desea. Este script se puede ejecutar una vez que esté en la interfaz interactiva de R, a la que se puede acceder (como se mencionó anteriormente) escribiendo **R** en el shell.  

### <a name="python"></a>Python

Anaconda Python se instala con los entornos Python 2.7 y 3.5. El entorno 2.7 se denomina _root_, y el entorno 3.5 se denomina _py35_. Esta distribución contiene Python base, junto con aproximadamente 300 de los paquetes de matemáticas, ingeniería y análisis de datos más populares.

El entorno py35 es el predeterminado. Para activar el entorno de raíz (2.7):

```bash
source activate root
```

Para volver a activar el entorno py35:

```bash
source activate py35
```

Después, para invocar la sesión interactiva de Python, escriba **python** en el shell. 

Instale bibliotecas adicionales de Python con ```conda``` o ```pip```. Para pip, active el entorno correcto en primer lugar si no desea el valor predeterminado:

```bash
source activate root
pip install <package>
```

O bien, especifique la ruta de acceso completa a pip:

```bash
/anaconda/bin/pip install <package>
```

Para conda, siempre debe especificar el nombre del entorno (_py35_ o _root_):

```bash
conda install <package> -n py35
```

Si se encuentra en una interfaz gráfica o tiene la configuración de reenvío de X11, puede escribir **pycharm** para iniciar el IDE de PyCharm Python. Puede usar los editores de texto predeterminados. Además, puede usar Spyder, un IDE de Python que integra las distribuciones de Anaconda Python. Spyder necesita un escritorio gráfico o el reenvío de X11. Se proporciona un acceso directo a Spyder en el escritorio gráfico.

### <a name="jupyter-notebook"></a>Cuaderno de Jupyter

La distribución de Anaconda también incluye un cuaderno de Jupyter Notebook, un entorno para compartir código y análisis. Se accede a Jupyter Notebook mediante JupyterHub. Inicie sesión con su nombre de usuario y contraseña de Linux local.

Se ha configurado previamente el servidor Jupyter Notebook con kernels de Python 2, Python 3 y R. Hay un icono del escritorio llamado "Jupyter Notebook" para iniciar el explorador y acceder al servidor de Jupyter Notebook. Si se encuentra en la máquina virtual a través de SSH o del cliente X2Go, también puede visitar [https://localhost:8000/](https://localhost:8000/) para acceder al servidor de Jupyter Notebook.

> [!NOTE]
> Continúe aunque aparezca una advertencia de certificado.

Puede acceder al servidor de Jupyter Notebook desde cualquier host. Solo tiene que escribir *https://\<<Nombre DNS o dirección IP de la máquina virtual\>:8000/*

> [!NOTE]
> El puerto 8000 se abre en el firewall de forma predeterminada cuando se aprovisiona la VM. 

Hemos empaquetado algunos cuadernos de ejemplo (uno en Python y otro en R). Puede ver el vínculo a los ejemplos en la página principal del cuaderno después de que se autentique en Jupyter Notebook con el nombre de usuario y la contraseña de Linux local. Puede crear un nuevo cuaderno seleccionando **Nuevo** y, después, el kernel de lenguaje apropiado. Si no ve el botón **Nuevo**, haga clic en el icono de **Jupyter** en la parte superior izquierda para ir a la página principal del servidor de Notebook.

### <a name="apache-spark-standalone"></a>Apache Spark Standalone

Una instancia independiente de Apache Spark está preinstalada en la DSVM de Linux para ayudarlo a desarrollar aplicaciones de Spark localmente antes de probarlas e implementarlas en clústeres de gran tamaño. Puede ejecutar programas PySpark mediante el kernel de Jupyter. Al abrir Jupyter, haga clic en el botón **Nuevo**. Debería ver una lista de los kernels disponibles. "Spark - Python" es el kernel de PySpark que le permite compilar aplicaciones Spark mediante el lenguaje Python. También puede usar un IDE de Python, como PyCharm o Spyder para compilar el programa de Spark. En esta instancia independiente, la pila de Spark se ejecuta el programa cliente que realiza la llamada, que hace que sea más rápido y más fácil solucionar problemas en comparación con el desarrollo en un clúster de Spark.

Se proporciona un bloc de notas de PySpark de ejemplo en Jupyter que puede encontrar en el directorio "SparkML" dentro del directorio particular de Jupyter ($HOME/notebooks/SparkML/pySpark). 

Si está programando en R para Spark, puede usar Microsoft R Server, SparkR o sparklyr. 

Antes de ejecutar en el contexto de Spark en Microsoft R Server, debe llevar a cabo un paso de configuración por una sola vez a fin de habilitar un HDFS de Hadoop de un solo nodo y una instancia de Yarn. De manera predeterminada, los servicios de Hadoop están instalados pero deshabilitados en la DSVM. Para habilitarlos, debe ejecutar los comandos siguientes como raíz la primera vez:

```bash
echo -e 'y\n' | ssh-keygen -t rsa -P '' -f ~hadoop/.ssh/id_rsa
cat ~hadoop/.ssh/id_rsa.pub >> ~hadoop/.ssh/authorized_keys
chmod 0600 ~hadoop/.ssh/authorized_keys
chown hadoop:hadoop ~hadoop/.ssh/id_rsa
chown hadoop:hadoop ~hadoop/.ssh/id_rsa.pub
chown hadoop:hadoop ~hadoop/.ssh/authorized_keys
systemctl start hadoop-namenode hadoop-datanode hadoop-yarn
```

Puede detener el Hadoop relacionados con los servicios cuando no necesite mediante la ejecución ```systemctl stop hadoop-namenode hadoop-datanode hadoop-yarn```

Un ejemplo que demuestra cómo desarrollar y probar MRS en el contexto de Spark remoto (que es la instancia independiente de Spark en DSVM) se proporciona y disponibles en el */dsvm/samples/MRS* directory.

### <a name="ides-and-editors"></a>IDE y editores

Tiene la opción de varios editores de código incluidos vi/VIM, Emacs, PyCharm, RStudio e IntelliJ. IntelliJ, RStudio y PyCharm son editores gráficos y necesitará iniciar sesión en un escritorio gráfico para poder usarlos. Puede iniciar estos editores desde los accesos directos del menú de aplicaciones y el escritorio.

**VIM** y **Emacs** son editores basados en texto. En Emacs, hemos instalado un paquete de complementos denominado Emacs Speaks Statistics (ESS) que facilita el trabajo con R en el editor de Emacs. Puede obtener más información en [ESS](https://ess.r-project.org/).

**LaTex** se instala mediante el paquete de texlive junto con un paquete [auctex](https://www.gnu.org/software/auctex/manual/auctex/auctex.html) del complemento de Emacs, lo que simplifica la creación de los documentos LaTex con Emacs.  

### <a name="databases"></a>Bases de datos

#### <a name="graphical-sql-client"></a>Cliente SQL gráfico

**SQuirrel SQL**, un cliente SQL gráfico, se proporciona para conectarse a bases de datos diferentes (Microsoft SQL Server y MySQL) y ejecutar consultas SQL. Puede ejecutar SQL SQuirrel desde una sesión de escritorio gráfico (mediante el cliente X2Go, por ejemplo) mediante un icono del escritorio, o mediante el siguiente comando en el shell:

```bash
/usr/local/squirrel-sql-3.7/squirrel-sql.sh
```

Antes de usarlo por primera vez, configure los controladores y los alias de las bases de datos. Los controladores JDBC se ubican en:

*/usr/share/java/jdbcdrivers*

Para obtener más información, consulte [SQuirrel SQL](http://squirrel-sql.sourceforge.net/index.php?page=screenshots).

#### <a name="command-line-tools-for-accessing-microsoft-sql-server"></a>Herramientas de línea de comandos para tener acceso a Microsoft SQL Server

El paquete de controladores ODBC para SQL Server también incluye dos herramientas de línea de comandos:

**bcp**: la utilidad bcp copia datos de forma masiva entre una instancia de Microsoft SQL Server y un archivo de datos en un formato especificado por el usuario. Asimismo, puede usarse para importar grandes cantidades de filas nuevas en tablas de SQL Server o exportar datos de tablas en archivos de datos. Para importar datos en una tabla, debe usar un archivo de formato creado para esa tabla o entender la estructura de la tabla y los tipos de datos que son válidos para sus columnas.

Puede encontrar más información en [Conexión con bcp](https://msdn.microsoft.com/library/hh568446.aspx).

**sqlcmd**: la utilidad sqlcmd le permite introducir instrucciones Transact-SQL, procedimientos del sistema y archivos de script en el símbolo del sistema. Esta utilidad usa ODBC para ejecutar lotes de Transact-SQL.

Puede encontrar más información en [Conexión con sqlcmd](https://msdn.microsoft.com/library/hh568447.aspx).

> [!NOTE]
> Existen algunas diferencias en esta utilidad entre las plataformas de Windows y Linux. Consulte la documentación de para obtener más información.

#### <a name="database-access-libraries"></a>Bibliotecas de acceso a las bases de datos

Hay bibliotecas disponibles en Python y en R para acceder a bases de datos.

* En R, el paquete **RODBC** o el paquete **dplyr** le permiten consultar o ejecutar instrucciones SQL en el servidor de bases de datos.
* En Python, la biblioteca **pyodbc** proporciona acceso a la base de datos con ODBC como la capa subyacente.  

### <a name="azure-tools"></a>Herramientas de Azure

En la VM se instalan las siguientes herramientas de Azure:

* **Interfaz de línea de comandos de Azure**: la CLI de Azure permite crear y administrar recursos de Azure mediante comandos de shell. Para invocar las herramientas de Azure, escriba **azure help**. Para obtener más información, consulte la [página de documentación de la CLI de Azure](https://docs.microsoft.com/cli/azure/get-started-with-az-cli2).
* **Explorador de Microsoft Azure Storage**: el Explorador de Microsoft Azure Storage es una herramienta gráfica que se usa para examinar los objetos que haya almacenado en la cuenta de Azure Storage y cargar o descargar datos con los blobs de Azure como origen y destino. Puede acceder al Explorador de Storage desde el icono de acceso directo del escritorio. Puede invocarlo desde el símbolo del sistema del shell escribiendo **StorageExplorer**. Debe haber iniciado sesión desde un cliente X2Go o tener X11 configuración de reenvío.
* **Bibliotecas de Azure**: a continuación, figuran algunas de las bibliotecas preinstaladas.
  
  * **Python**: las bibliotecas relacionadas con Azure en Python que están instaladas son **azure**, **azureml**, **pydocumentdb** y **pyodbc**. Las tres primeras bibliotecas permiten acceder a los servicios de Azure Storage, a Azure Machine Learning y a Azure Cosmos DB (una base de datos NoSQL en Azure). La cuarta biblioteca, pyodbc (junto con el controlador ODBC de Microsoft para SQL Server), permite el acceso a SQL Server, Azure SQL Database y Azure SQL Data Warehouse desde Python mediante una interfaz ODBC. Escriba **pip list** para ver todas las bibliotecas enumeradas. Asegúrese de ejecutar este comando en los entornos de Python 2.7 y 3.5.
  * **R**: las bibliotecas relacionadas con Azure en R que están instaladas son **AzureML** y **RODBC**.
  * **Java**: la lista de bibliotecas de Java para Azure se puede encontrar en el directorio **/dsvm/sdk/AzureSDKJava** de la máquina virtual. Las bibliotecas principales son Azure Storage y las API de administración, Azure Cosmos DB y los controladores JDBC para SQL Server.  

Puede acceder a [Azure Portal](https://portal.azure.com) desde el explorador Firefox instalado previamente. En Azure Portal, puede crear, administrar y supervisar los recursos de Azure.

### <a name="azure-machine-learning"></a>Azure Machine Learning

Azure Machine Learning es un servicio en la nube totalmente administrado que permite crear, implementar y compartir soluciones de análisis predictivo. Puede crear sus experimentos y modelos desde Azure Machine Learning Studio. Puede acceder a él desde un explorador web en la máquina virtual de ciencia de datos si visita la página de [Microsoft Azure Machine Learning](https://studio.azureml.net).

Una vez que inicie sesión en Azure Machine Learning Studio, tendrá acceso a un lienzo de experimentación donde puede crear un flujo lógico para los algoritmos de aprendizaje automático. También tiene acceso a Jupyter Notebook hospedado en Azure Machine Learning, que puede funcionar perfectamente con los experimentos de Azure Machine Learning Studio. Haga que los modelos de aprendizaje automático que ha creado estén operativos encapsulándolos en una interfaz de servicio web. Operacionalización de modelos de aprendizaje automático permite a los clientes escritos en cualquier lenguaje invocar predicciones a partir de esos modelos. Para obtener más información, consulte la [documentación sobre Machine Learning](https://azure.microsoft.com/documentation/services/machine-learning/).

También puede crear los modelos en R o en Python en la máquina virtual y, después, implementarlos en la producción en Azure Machine Learning. Hemos instalado bibliotecas en R (**AzureML**) y en Python (**azureml**) para habilitar esta funcionalidad.

Para obtener información sobre cómo implementar modelos en R y en Python en Azure Machine Learning, consulte la sección [Diez cosas que puede hacer en Data Science Virtual Machine](vm-do-ten-things.md) (en concreto, la sección Compilación de modelos mediante R y Python, y hacer que estén operativos mediante Azure Machine Learning).

> [!NOTE]
> Estas instrucciones se han escrito para la versión de Windows de la VM de ciencia de datos. Sin embargo, la información proporcionada sobre la implementación de modelos en Azure Machine Learning es aplicable a la máquina virtual Linux.

### <a name="machine-learning-tools"></a>Herramientas de aprendizaje automático

La máquina virtual incluye algunas herramientas o algoritmos de aprendizaje automático que se han precompilado y preinstalado de forma local. Entre ellas se incluyen las siguientes:

* **Vowpal Wabbit**: algoritmo de aprendizaje rápido en línea.
* **xgboost**: herramienta que proporciona los algoritmos de árbol ampliados y optimizados.
* **Rattle**: herramienta gráfica basada en R para facilitar el modelado y la exploración de datos.
* **Python**: Anaconda Python integra algoritmos de aprendizaje automático con bibliotecas como Scikit-learn. Puede instalar otras bibliotecas con el comando `pip install` .
* **LightGBM**: entorno de potenciación de gradientes rápido, distribuido y de alto rendimiento basado en algoritmos de árbol de decisión.
* **R**: hay disponible una completa biblioteca de funciones de aprendizaje automático para R. Algunas de las bibliotecas preinstaladas son lm, glm, randomForest y rpart. Puede instalar otras bibliotecas si ejecuta el comando:
  
        install.packages(<lib name>)

A continuación, proporcionamos información adicional sobre las tres primeras herramientas de aprendizaje automático de la lista.

#### <a name="vowpal-wabbit"></a>Vowpal Wabbit

Vowpal Wabbit es un sistema de aprendizaje automático rápido que emplea varias técnicas, como el aprendizaje en línea, el uso de hash, la clase AllReduce, las reducciones, learning2search y el aprendizaje activo e interactivo.

Para ejecutar la herramienta en un ejemplo básico, use los siguientes comandos:

```bash
cp -r /dsvm/tools/VowpalWabbit/demo vwdemo
cd vwdemo
vw house_dataset
```

Puede encontrar otras demostraciones más grandes en ese directorio. Para obtener más información sobre VW, consulte [esta sección de GitHub](https://github.com/JohnLangford/vowpal_wabbit) y la [wiki de Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki).

#### <a name="xgboost"></a>XGBoost

Se trata de una biblioteca que está diseñada y optimizada para algoritmos incrementados (de árbol). El objetivo de esta biblioteca es llevar los límites de cálculo de las máquinas a los puntos de conexión necesarios para proporcionar una ampliación de árboles a gran escala que sea escalable, portátil y precisa.

Se proporciona como una línea de comandos, así como una biblioteca de R.

Para usar esta biblioteca en R, puede iniciar una sesión interactiva de R (escribiendo simplemente **R** en el shell) y cargando la biblioteca.

Este es un ejemplo sencillo que puede ejecutar en el símbolo del sistema de R:

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

Se escribe un archivo .model en el directorio especificado. Puede encontrar información sobre este ejemplo de demostración [en GitHub](https://github.com/dmlc/xgboost/tree/master/demo/binary_classification).

Puede obtener más información sobre xgboost en la [página de documentación de xgboost](https://xgboost.readthedocs.org/en/latest/) y en su [repositorio de GitHub](https://github.com/dmlc/xgboost).

#### <a name="rattle"></a>Rattle

Rattle (siglas del inglés **R** **A**nalytical **T**ool **T**o **L**earn **E**asily, la herramienta de análisis de R para aprender fácilmente) utiliza el modelado y la exploración de datos basados en GUI. Presenta resúmenes visuales y estadísticos de datos, transforma datos que pueden modelarse fácilmente, crea modelos supervisados y no supervisados a partir de los datos, presenta el rendimiento de los modelos gráficamente y puntúa los conjuntos de datos nuevos. También genera un código R replicando las operaciones en la interfaz de usuario que pueden ejecutarse directamente en R o usarse como un punto de inicio para un análisis posterior.

Para ejecutar Rattle, debe haber iniciado sesión en un escritorio gráfico. En el terminal, escriba ```R``` para entrar en el entorno de R. En el símbolo del sistema de R, escriba los siguientes comandos:

```R
library(rattle)
rattle()
```

Se abrirá una interfaz gráfica con un conjunto de pestañas. Estos son los pasos de inicio rápido en Rattle para usar un conjunto de datos meteorológicos de ejemplo y generar un modelo. En algunos de los pasos siguientes, se le solicitará que instale y cargue automáticamente cualquier paquete de R necesario que todavía no esté en el sistema.

> [!NOTE]
> Si no tiene acceso para instalar el paquete en el directorio del sistema (valor predeterminado), puede ver un símbolo del sistema en la ventana de la consola de R para instalar paquetes en su biblioteca personal. Si ve estos avisos, responda *y* .

1. Haga clic en **Ejecutar**.
1. Aparecerá un cuadro de diálogo que le preguntará si quiere usar el conjunto de datos de meteorología de ejemplo. Haga clic en **Yes** (Sí) para cargar el ejemplo.
1. Haga clic en la pestaña **Model** (Modelo).
1. Haga clic en **Execute** (Ejecutar) para crear un árbol de decisión.
1. Haga clic en **Draw** (Dibujar) para mostrar el árbol de decisión.
1. Haga clic en el botón de opción **Forest** (Bosque) y haga clic en **Execute** (Ejecutar) para crear un bosque aleatorio.
1. Haga clic en la pestaña **Evaluate** (Evaluar).
1. Haga clic en el botón de opción **Risk** (Riesgo) y haga clic en **Execute** (Ejecutar) para mostrar dos trazados de rendimiento de riesgo (acumulativo).
1. Haga clic en la pestaña **Log** (Registro) para mostrar el código en R generado para las operaciones anteriores
   (debido a un error de la versión actual de Rattle, debe insertar un carácter *#* delante de *Export this log* [Exportar este registro] en el texto del registro).
1. Haga clic en el botón **Export** (Exportar) para guardar el script de R en el archivo *weather_script.R* de la carpeta principal.

Puede salir de Rattle y R. Ahora puede modificar el script de R generado o usarlo tal como está y ejecutarlo en cualquier momento para repetir todo lo que se realizó en la interfaz de usuario de Rattle. Esta es una forma sencilla (especialmente para los principiantes en R) de realizar el análisis y el aprendizaje automático rápidamente en una interfaz gráfica sencilla mientras se genera automáticamente un código en R para modificar o aprender.

## <a name="next-steps"></a>Pasos siguientes

A continuación, mostramos cómo puede continuar con las tareas de aprendizaje y exploración:

* El tutorial sobre [Ciencia de datos en Data Science Virtual Machine para Linux](linux-dsvm-walkthrough.md) muestra cómo llevar a cabo varias tareas comunes de ciencia de datos con la máquina virtual de ciencia de datos Linux aprovisionada aquí. 
* Explore las diversas herramientas de ciencia de datos en la máquina virtual de ciencia de datos probando las herramientas descritas en este artículo. También puede ejecutar *dsvm-more-info* en el shell de la máquina virtual para obtener un introducción básica y referencias para consultar más información sobre las herramientas instaladas en la máquina virtual.  
* Aprenda a crear soluciones analíticas completas mediante el uso sistemático del [proceso de ciencia de datos en equipo](https://aka.ms/tdsp).
* Visite la [Azure AI Gallery](https://gallery.azure.ai/) para ver ejemplos de aprendizaje automático y análisis de datos donde se usan los servicios de Azure AI.
