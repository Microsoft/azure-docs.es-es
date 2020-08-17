---
title: Idiomas compatibles
titleSuffix: Azure Data Science Virtual Machine
description: Lenguajes de programación admitidos y herramientas relacionadas que se instalaron previamente en Data Science Virtual Machine.
keywords: herramientas de ciencia de datos, máquina virtual de ciencia de datos, herramientas para la ciencia de datos, ciencia de datos de linux
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
ms.custom: devx-track-python
author: lobrien
ms.author: laobri
ms.topic: conceptual
ms.date: 12/12/2019
ms.openlocfilehash: 8a73336ba0a324c65ee0de764d81a8e69ab6079c
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87843273"
---
# <a name="languages-supported-on-the-data-science-virtual-machine"></a>Lenguajes admitidos en Data Science Virtual Machine 

Data Science Virtual Machine (DSVM) incorpora varios lenguajes predefinidos y herramientas de desarrollo para compilar sus aplicaciones de inteligencia artificial (IA). Estos son algunos de los más importantes.

## <a name="python-windows-server-2016-edition"></a>Python (edición de Windows Server 2016)

| Category | Value |
| ------------- | ------------- |
| Versiones de lenguajes admitidas | Python 2.7 y 3.7 |
| Ediciones de DSVM admitidas      | Windows Server 2016     |
| ¿Cómo se configura/instala en DSVM?  | Se crean dos entornos de `conda` globales. <br /> * El entorno `root` ubicado en `/anaconda/` es Python 3.7. <br/> * El entorno `python2` ubicado en `/anaconda/envs/python2` es Python 2.7.       |
| Vínculos a ejemplos      | Se cuadernos de Jupyter de ejemplo para Python.     |
| Herramientas relacionadas en DSVM      | PySpark, R y Julia.      |

> [!NOTE]
> Las compilaciones de Windows Server 2016 que se crearon antes de marzo de 2018 contienen Python 3.5 y Python 2.7. Python 2.7 es el entorno **raíz** de Conda y **py37** es el entorno de Python 3.7.

### <a name="how-to-use-and-run-it"></a>Cómo usarla y ejecutarla    

* En un símbolo del sistema:

  Abra un símbolo del sistema y use uno de los métodos siguientes, en función de la versión de Python que quiera ejecutar:

    ```
    # To run Python 2.7
    activate python2
    python --version
    
    # To run Python 3.7
    activate 
    python --version 
    ```
    
* Uso en un IDE:

  Use Herramientas de Python para Visual Studio (PTVS), que viene instalado en la edición de Visual Studio Community. De forma predeterminada, el único entorno que se configura automáticamente en PTVS es Python 3.6. 

    > [!NOTE]
    > Para señalar PTVS en Python 2.7, debe crear un entorno personalizado en PTVS. Para establecer estas rutas de acceso de entorno en Visual Studio Community Edition, vaya a **Herramientas** -> **Herramientas de Python** -> **Entornos de Python** y seleccione **+ Personalizar**. Después, establezca la ubicación en **c:\anaconda\envs\python2** y seleccione **Detección automática**.

* Uso en Jupyter:

  Abra Jupyter y seleccione **Nuevo** para crear un cuaderno. Puede establecer el tipo de kernel como _Python [Conda Root]_ para el entorno Python 3.7 y _Python [Conda env:python2]_ para Python 2.7.

* Instalación de paquetes de Python:

  Los entornos predeterminados de Python en DSVM son entornos globales que pueden leer todos los usuarios. Solo los administradores pueden escribir e instalar paquetes globales. Para instalar los paquetes en el entorno global, actívelos en el entorno raíz o en python2 mediante el comando `activate` como administrador. Después podrá usar un administrador de paquetes como `conda` o `pip` para instalar o actualizar paquetes.

## <a name="python-linux-edition"></a>Python (edición de Linux)

| Category | Value |
| ------------- | ------------- |
| Versiones de lenguajes admitidas | Python 2.7 y 3.5 |
| Ediciones de DSVM admitidas      | Linux   |
| ¿Cómo se configura/instala en DSVM?  | Se crean dos entornos de `conda` globales. <br /> El entorno * `root` ubicado en `/anaconda/` es Python 2.7. <br/> El entorno * `py35` ubicado en `/anaconda/envs/py35` es Python 3.5.       |
| Vínculos a ejemplos      | Se cuadernos de Jupyter de ejemplo para Python.     |
| Herramientas relacionadas en DSVM      | PySpark, R y Julia      |

### <a name="how-to-use-and-run-it"></a>Cómo usarla y ejecutarla    

* Ejecución en un terminal:

  Abra un terminal y realice lo siguiente, según la versión de Python que quiera ejecutar:

    ```
    # To run Python 2.7
    source activate 
    python --version
    
    # To run Python 3.5
    source activate py35
    python --version
    
    ```
* Uso en un IDE:

  Use PyCharm, que está instalado en la edición Visual Studio Community. 

* Uso en Jupyter:

  Abra Jupyter y seleccione **Nuevo** para crear un cuaderno. Puede establecer el tipo de kernel como **Python [Conda Root]** para Python 2.7 y **Python [Conda env:py35]** para el entorno Python 3.5. 

* Instalación de paquetes de Python:

  Los entornos predeterminados de Python en DSVM son entornos globales que pueden leer todos los usuarios, Solo los administradores pueden escribir e instalar paquetes globales. Para instalar el paquete en el entorno global, actívelo en el entorno raíz o en py35 mediante el comando `source activate` como administrador o como un usuario que tenga los permisos sudo. Después podrá usar un administrador de paquetes como `conda` o `pip` para instalar o actualizar paquetes.


## <a name="r"></a>R

| Category | Value |
| ------------- | ------------- |
| Versiones de lenguajes admitidas | Microsoft R Open 3.x (100 % compatible con CRAN-R)<br /> Edición Microsoft R Server 9.x Developer (una plataforma de R escalable y preparada para empresas)|
| Ediciones de DSVM admitidas      | Linux y Windows     |
| ¿Cómo se configura/instala en DSVM?  | Windows: `C:\Program Files\Microsoft\ML Server\R_SERVER` <br />Linux: `/usr/lib64/microsoft-r/3.3/lib64/R`    |
| Vínculos a ejemplos      | Se incluyen cuadernos de Jupyter de ejemplo para R.     |
| Herramientas relacionadas en DSVM      | SparkR, Python y Julia      |

### <a name="how-to-use-and-run-it"></a>Cómo usarla y ejecutarla    

**Windows**:

* En un símbolo del sistema:

  Abra un símbolo del sistema y escriba `R`.

* Uso en un IDE:

  Use Herramientas de R para Visual Studio (RTVS), que viene instalado en la edición de Visual Studio Community o RStudio. Están disponibles en el menú Inicio o en forma de icono del escritorio. 

* Uso en Jupyter

  Abra Jupyter y seleccione **Nuevo** para crear un cuaderno. Puede establecer el tipo de kernel como **R** para usar el kernel de Jupyter R (IRKernel).

* Instalación de paquetes de R:

  R se instala en DSVM en un entorno global que todos los usuarios pueden leer, Solo los administradores pueden escribir e instalar paquetes globales. Para instalar paquetes en el entorno global, ejecute R con uno de los métodos anteriores. Después podrá ejecutar el administrador de paquetes de R `install.packages()` para instalar o actualizar paquetes.

**Linux**:

* Ejecución en el terminal:

  Abra el terminal y ejecute `R`.  

* Uso en un IDE:

  Use RStudio, que viene instalado en DSVM para Linux.  

* Uso en Jupyter:

  Abra Jupyter y seleccione **Nuevo** para crear un cuaderno. Puede establecer el tipo de kernel como **R** para usar el kernel de Jupyter R (IRKernel). 

* Instalación de paquetes de R:

  R se instala en DSVM en un entorno global que todos los usuarios pueden leer, Solo los administradores pueden escribir e instalar paquetes globales. Para instalar paquetes en el entorno global, ejecute R con uno de los métodos anteriores. Después podrá ejecutar el administrador de paquetes de R `install.packages()` para instalar o actualizar paquetes.


## <a name="julia"></a>Julia

| Category | Value |
| ------------- | ------------- |
| Versiones de lenguajes admitidas | 0.6 |
| Ediciones de DSVM admitidas      | Linux y Windows     |
| ¿Cómo se configura/instala en DSVM?  | Windows: se instaló en `C:\JuliaPro-VERSION`<br /> Linux: se instaló en `/opt/JuliaPro-VERSION`    |
| Vínculos a ejemplos      | Se incluyen cuadernos de Jupyter de ejemplo para Julia.     |
| Herramientas relacionadas en DSVM      | Python, R      |

### <a name="how-to-use-and-run-it"></a>Cómo usarla y ejecutarla    

**Windows**:

* Ejecución en un símbolo del sistema

  Abra un símbolo del sistema y ejecute `julia`.
* Uso en un IDE:

  Use `Juno` con el IDE de Julia instalado en DSVM y disponible como acceso directo del escritorio.

* Uso en Jupyter:

  Abra Jupyter y seleccione **Nuevo** para crear un cuaderno. Puede establecer el tipo de kernel como **Julia VERSIÓN**.

* Instalación de paquetes de Julia:

  La ubicación predeterminada de Julia es un entorno global que todos los usuarios pueden leer, Solo los administradores pueden escribir e instalar paquetes globales. Para instalar paquetes en el entorno global, ejecute Julia con uno de los métodos anteriores. Después podrá ejecutar los comandos del administrador de paquetes de Julia, como `Pkg.add()`, para instalar o actualizar paquetes.


**Linux**:
* Ejecución en un terminal:

  Abra el terminal y ejecute `julia`.
* Uso en un IDE:

  Use `Juno`, con el IDE de Julia instalado en la DSVM y disponible como acceso directo del menú de **aplicación**.

* Uso en Jupyter:

  Abra Jupyter y seleccione **Nuevo** para crear un cuaderno. Puede establecer el tipo de kernel como **Julia VERSIÓN**.

* Instalación de paquetes de Julia:

  La ubicación predeterminada de Julia es un entorno global que todos los usuarios pueden leer, Solo los administradores pueden escribir e instalar paquetes globales. Para instalar paquetes en el entorno global, ejecute Julia con uno de los métodos anteriores. Después podrá ejecutar los comandos del administrador de paquetes de Julia, como `Pkg.add()`, para instalar o actualizar paquetes.

## <a name="other-languages"></a>Otros idiomas

**C#** : disponible en Windows y accesible mediante Visual Studio Community Edition o en `Developer Command Prompt for Visual Studio`, donde solo puede ejecutar el comando `csc`.

**Java**: OpenJDK está disponible en las ediciones para Windows y Linux de DSVM y se establece en la ruta de acceso. Para usar Java, escriba el comando `javac` o `java` en el símbolo del sistema en Windows o en el shell bash de Linux.

**Node.js**: Node.js está disponible en la edición para Windows y Linux de DSVM y se establece en la ruta de acceso. Para acceder a Node.js, escriba el comando `node` o `npm` en el símbolo del sistema en Windows o en el shell bash de Linux. En Windows, se instala la extensión de Visual Studio para las herramientas de Node.js para proporcionar un IDE gráfico con el fin de desarrollar la aplicación Node.js.

**F#** : disponible en Windows y accesible mediante Visual Studio Community Edition o en `Developer Command Prompt for Visual Studio`, donde solo puede ejecutar el comando `fsc`.
