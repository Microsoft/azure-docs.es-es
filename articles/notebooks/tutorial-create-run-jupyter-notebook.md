---
title: 'Tutorial: Creación y ejecución de un cuaderno de Jupyter en la versión preliminar de Azure Notebooks'
description: Aprenda a crear y ejecutar un cuaderno de Jupyter en la versión preliminar de Azure Notebooks para demostrar el proceso de regresión lineal en ciencia de datos.
ms.topic: tutorial
ms.date: 01/11/2019
ms.custom: devx-track-python
ms.openlocfilehash: f6480a93610761061462be8e83ce3e719f3e8bdf
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87846919"
---
# <a name="tutorial-create-and-run-a-jupyter-notebook-with-python"></a>Tutorial: Creación y ejecución de un cuaderno de Jupyter con Python

[!INCLUDE [notebooks-status](../../includes/notebooks-status.md)]

Este tutorial le guiará en el proceso de uso de Azure Notebooks para crear un cuaderno de Jupyter completo que muestra la regresión lineal simple. En el transcurso de este tutorial, se familiarizará con la interfaz de usuario de Jupyter Notebook, que incluye la creación de distintas celdas, la ejecución de celdas y la visualización del cuaderno como una presentación.

Encontrará el cuaderno completo en [GitHub: Ejemplos de Azure Notebooks](https://github.com/Microsoft/AzureNotebooks/tree/master/Samples/Linear%20Regression%20-%20Cricket%20Chirps). Sin embargo, este tutorial empieza con un proyecto nuevo y un cuaderno vacío para que pueda experimentar su creación paso a paso.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un cuaderno de proyecto con algunos datos de ejemplo
> * Usar la interfaz de cuaderno para crear diversos tipos de celdas
> * Ejecución del cuaderno
> * Guardar el cuaderno
> * Depurar el cuaderno en Visual Studio Code

## <a name="create-the-project"></a>Creación del proyecto

1. Vaya a [Azure Notebooks](https://notebooks.azure.com) e inicie sesión. (Para más información, consulte [Quickstart - Sign in to Azure Notebooks](quickstart-sign-in-azure-notebooks.md) [Inicio rápido: Inicio de sesión en Azure Notebooks]).

1. En la página del perfil público, seleccione **My Projects** (Mis proyectos) en la parte superior de la página:

    ![Vínculo Mis proyectos en la parte superior de la ventana del explorador](media/quickstarts/my-projects-link.png)

1. En la página **My Projects** (Mis proyectos), seleccione **+New Project** (+Nuevo proyecto) [método abreviado de teclado: n]; el botón puede aparecer solo como **+** si la ventana del explorador es estrecha:

    ![Comando New Project (Nuevo proyecto) en la página My Projects (Mis proyectos)](media/quickstarts/new-project-command.png)

1. En la ventana emergente **Create New Project** (Crear nuevo proyecto) que aparece, escriba o establezca estos detalles y, luego, seleccione **Create** (Crear):

   - **Project name** (Nombre del proyecto): Linear Regression Example - Cricket Chirps
   - **Project ID** (Id. del proyecto): linear-regression-example
   - **Public project** (Proyecto público): desactivada
   - **Create a README.md** (Crear un Léame.md): desactivada

1. Después de unos instantes, Azure Notebooks lo lleva al proyecto nuevo.

## <a name="create-the-data-file"></a>Creación del archivo de datos

El modelo de regresión lineal que se crea en el cuaderno extrae datos de un archivo del proyecto llamado *cricket_chirps.csv*. Para crear este archivo, cópielo de [GitHub - Azure Notebooks Samples](https://github.com/Microsoft/AzureNotebooks/tree/master/Samples/Linear%20Regression%20-%20Cricket%20Chirps) o escriba los datos directamente. Estas dos opciones se describen en las secciones siguientes.

### <a name="upload-the-data-file"></a>Carga del archivo de datos

1. En el panel del proyecto en Azure Notebooks, seleccione **Upload** > **From URL** (Cargar > Desde URL).
1. En la ventana emergente, escriba la siguiente dirección URL en **File URL** (URL de archivo) y *cricket_chirps.csv* en **File Name** (Nombre de archivo); después, seleccione **Done** (Listo).

    ```url
    https://raw.githubusercontent.com/Microsoft/AzureNotebooks/master/Samples/Linear%20Regression%20-%20Cricket%20Chirps/cricket_chirps.csv
    ```

1. El archivo *cricket_chirps.csv* aparecerá en la lista de archivos del proyecto:

    ![Archivo CSV recién creado que se muestra en la lista de archivos de proyecto](media/tutorial/csv-file-in-project.png)

### <a name="create-a-file-from-scratch"></a>Creación de un archivo desde cero

1. En el panel del proyecto en Azure Notebooks, seleccione **+ New** > **Blank File** (+Nuevo > Archivo en blanco).
1. Aparece un campo en la lista de archivos del proyecto. Escriba *cricket_chirps.csv* y pulse Entrar.
1. Haga clic con el botón derecho en *cricket_chirps.csv* y seleccione **Edit File** (Editar archivo).
1. En el editor que aparece, escriba la siguiente información:

    ```csv
    Chirps/Minute,Temperature
    20,88.6
    16,71.6
    19.8,93.3
    18.4,84.3
    17.1,80.6
    15.5,75.2
    14.7,69.7
    17.1,82
    15.4,69.4
    16.2,83.3
    15,79.6
    17.2,82.6
    16,80.6
    17,83.5
    14.4,76.3
    ```

1. Seleccione **Save File** (Guardar archivo) para guardar el archivo y volver al panel del proyecto.

## <a name="install-project-level-packages"></a>Instalación de los paquetes de nivel de proyecto

Dentro de un cuaderno, siempre puede usar comandos como `!pip install` en una celda de código para instalar los paquetes necesarios. Sin embargo, estos comandos se ejecutan cada vez que ejecute las celdas de código del cuaderno, lo que puede llevar un tiempo considerable. Por eso, en su lugar, puede instalar paquetes en el nivel proyecto con un archivo `requirements.txt`.

1. Use el proceso descrito en [Creación de un archivo desde cero](#create-a-file-from-scratch) para crear un archivo llamado `requirements.txt` con el siguiente contenido:

    ```text
    matplotlib==3.0.0
    numpy==1.15.0
    pandas==0.23.4
    scikit-learn==0.20.0
    ```

    También puede cargar un archivo `requirements.txt` desde el equipo local si lo prefiere, tal y como se describe en [Carga del archivo de datos](#upload-the-data-file).

1. En el panel del proyecto, seleccione **Project Settings** (Configuración del proyecto).
1. En la ventana emergente que aparece, seleccione la pestaña **Environment** (Entorno) y, después, **+Add** (+Agregar).
1. En el primer control de lista desplegable (la operación), en **Environment Setup Steps** (Pasos de configuración del entorno), elija **Requirements.txt**.
1. En el segundo control de lista desplegable (el nombre de archivo), elija *requirements.txt* (el archivo que creó).
1. En el tercer control de lista desplegable (la versión de Python), elija **Python Version 3.6**.
1. Seleccione **Guardar**.

![Pestaña Project Settings Environment (Entorno de configuración del proyecto) con el archivo requirements.txt especificado](media/tutorial/tutorial-requirements-txt.png)

Una vez realizado este paso de configuración, cualquier cuaderno que se ejecute en el proyecto se ejecutará en un entorno donde estén instalados esos paquetes.

## <a name="create-and-run-a-notebook"></a>Creación y ejecución de un cuaderno

Con el archivo de datos listo y el entorno del proyecto establecido, ya puede crear y abrir el cuaderno.

1. En el panel del proyecto, seleccione **+New** > **Notebook** (+Nuevo > Cuaderno).
1. En la ventana emergente, escriba *Linear Regression Example - Cricket Chirps.ipynb* en **Item Name** (Nombre del elemento), elija **Python 3.6** como lenguaje y, después, seleccione **New** (Nuevo).
1. Cuando el nuevo cuaderno aparezca en la lista de archivos, selecciónelo para iniciarlo. Se abre una nueva pestaña del explorador automáticamente.
1. Como tiene un archivo *requirements.txt* en la configuración del entorno, verá el mensaje "Waiting for your container to finish being prepared" (Esperando que finalice la preparación del contenedor). Puede seleccionar **OK** (Aceptar) para cerrar el mensaje y continuar trabajando en el cuaderno; sin embargo; las celdas de código no se pueden ejecutar hasta que el entorno esté totalmente configurado.
1. Abre el cuaderno en la interfaz de Jupyter con una celda única de código vacío como el valor predeterminado.

    [![Vista inicial de un cuaderno nuevo en Azure Notebooks](media/tutorial/tutorial-new-notebook.png)](media/tutorial/tutorial-new-notebook.png#lightbox)

## <a name="tour-the-notebook-interface"></a>Paseo por la interfaz del cuaderno

Con el cuaderno en ejecución, puede agregar código y celdas de Markdown, ejecutar esas celdas y administrar el funcionamiento del cuaderno. Sin embargo, merece la pena dedicar unos minutos primero a familiarizarse con la interfaz. Para obtener la documentación completa, seleccione el comando de menú **Help** > **Notebook Help** (Ayuda > Ayuda del cuaderno).

En la parte superior de la ventana verá los siguientes elementos:

(A) El nombre del cuaderno; puede hacer clic en él para editarlo.
(B) Los botones para navegar al proyecto contenedor y el panel de proyectos, que abre nuevas pestañas en el explorador.
(C) Un menú con comandos para trabajar con el cuaderno.
(D) Una barra de herramientas con los métodos abreviados para las operaciones comunes.
(E) El lienzo de edición que contiene las celdas.
(F) Un indicador de si el cuaderno es de confianza; el valor predeterminado es **Not Trusted** (No es de confianza).
(G) El kernel usado para ejecutar el cuaderno, junto con un indicador de actividad.

[![Áreas principales de la interfaz de usuario de Jupyter](media/tutorial/tutorial-notebook-ui.png)](media/tutorial/tutorial-notebook-ui.png#lightbox)

Jupyter incorpora un paseo por los principales elementos de la interfaz de usuario. Para iniciar la visita, seleccione el comando **Help** > **User Interface Tour** (Ayuda > Paseo por la interfaz de usuario) y haga clic en los menús emergentes.

Los grupos de comandos de menú son los siguientes:

| Menú | Descripción |
| --- | --- |
| Archivo | Comandos para administrar el archivo de cuaderno, incluidos los comandos para crear y copiar cuadernos, mostrar una vista previa de impresión y descargar el cuaderno en diversos formatos. |
| Editar | Comandos típicos para cortar, copiar y pegar celdas, buscar y reemplazar valores, administrar datos adjuntos de celda e insertar imágenes.  |
| Ver | Comandos para controlar la visibilidad de las distintas partes de la UI de Jupyter. |
| Insertar | Comandos para insertar una nueva celda por encima o debajo de la celda actual. Estos comandos se utilizan con frecuencia para crear un cuaderno. |
| Cell | Los distintos comandos **Run** (Ejecutar) ejecutan una o varias celdas en diferentes combinaciones. Los comandos **Cell Type** (Tipo de celda) cambia el tipo de celda entre **Code** (Código), **Markdown** y **Raw NBConvert** (texto sin formato). Los comandos **Current Outputs** (Salidas actuales) y **All Outputs** (Todas las salidas) controlan cómo se muestra el resultado de la ejecución del código, e incluyen un comando para borrar todas las salidas. |
| Kernel | Comandos para administrar cómo se ejecuta el código en el kernel, más **Change kernel** (Cambiar kernel) para cambiar el lenguaje o la versión de Python que se usa para ejecutar el cuaderno. |
| data | Comandos para cargar y descargar archivos desde el proyecto o la sesión. Consulte [Trabajo con archivos de datos de proyecto](work-with-project-data-files.md). |
| Widgets | Comandos para administrar [Jupyter Widgets](https://ipywidgets.readthedocs.io/en/stable/examples/Widget%20Basics.html) (Widgets de Jupyter), que proporcionan funcionalidades adicionales para la visualización, la asignación y el trazado.|
| Ayuda | Comandos que ofrecen ayuda y documentación sobre la interfaz de Jupyter. |

La mayoría de los comandos de la barra de herramientas tiene comandos de menú equivalentes. Una excepción es **Enter/Edit RISE Slideshow** (Especificar/editar presentación de RISE), que se describe en [Uso compartido y presentación de cuadernos](present-jupyter-notebooks-slideshow.md).

Use estos comandos mientras rellena el cuaderno en las secciones siguientes.

## <a name="create-a-markdown-cell"></a>Creación de una celda de Markdown

1. Haga clic en la primera celda vacía que se muestra en el lienzo del cuaderno. De forma predeterminada, una celda es del tipo **Code**, lo que significa que se ha diseñado para contener código ejecutable para el kernel seleccionado (Python, R o F#). El tipo actual se muestra en la lista desplegable de tipos, en la barra de herramientas:

    ![Lista desplegable de tipo de celda en la lista desplegable](media/tutorial/tutorial-cell-type-drop-down.png)

1. Cambie el tipo de celda a **Markdown** utilizando la lista desplegable de la barra de herramientas; también puede usar los comandos de menú **Cell** > **Cell Type** > **Markdown** (Celda > Tipo de celda > Markdown):

    ![Comando de menú de tipo de celda](media/tutorial/tutorial-cell-type-menu.png)

1. Haga clic en la celda para comenzar a editarla y, luego, escriba el siguiente código Markdown:

    ```markdown
    # Example Linear Regression

    This notebook contains a walkthrough of a simple linear regression. The data, obtained from [college.cengage.com](https://college.cengage.com/mathematics/brase/understandable_statistics/7e/students/datasets/slr/frames/frame.html), relates the rate of cricket chirps to temperature from *The Song of Insects*, by Dr. G. W. Pierce, Harvard College Press.

    In this example we're using the count of chirps per minute as the independent varible to then predict the dependent variable, temperature. In short, we're using a little data science to make ourselves a cricket thermometer. (You could also reverse the data and use temperature to predict the number of chirps, but it's more fun to use crickets as the thermometer itself!)

    The methods shown in this example follow what's presented in the Udemy course, [Machine Learning A to Z](https://www.udemy.com/machinelearning/learn/v4/).

    A lovely aspect of Notebooks is that you can use Markdown cells to explain what the code is doing rather than code comments. There are several benefits to doing so:

    - Markdown allows for richer text formatting, like *italics*, **bold**, `inline code`, hyperlinks, and headers.
    - Markdown cells automatically word wrap whereas code cells do not. Code comments typically use explicit line breaks for formatting, but that's not necessary in Markdown.
    - Using Markdown cells makes it easier to run the Notebook as a slide show.
    - Markdown cells help you remove lengthy comments from the code, making the code easier to scan.

    When you run a code cell, Jupyter executes the code; when you run a Markdown cell, Jupyter renders all the formatting into text that's suitable for presentation.
    ```

1. Para representar el código Markdown como HTML para el explorador, seleccione el comando **Run** (Ejecutar) en la barra de herramientas, o use el comando **Cell** > **Run Cells** (Celda > Ejecutar celdas). Ahora, el código Markdown del formato y los vínculos aparece según lo esperado en un explorador.

1. Cuando se ejecuta la última celda del cuaderno, Jupyter crea automáticamente una nueva celda debajo de la que se ha ejecutado. Repita los pasos descritos en esta sección para agregar el siguiente código Markdown a esta celda:

    ```markdown
    ## Install packages using pip or conda

    Because the code in your notebook likely uses some Python packages, you need to make sure the Notebook environment contains those packages. You can do this directly within the notebook in a code block that contains the appropriate pip or conda commands prefixed by `!`:

    \```
    !pip install <pkg name>

    !conda install <pkg name> -y
    \```
    ```

1. Para volver a editar el código Markdown, haga doble clic en la celda representada. Para volver a representar HTML después de realizar cambios, ejecute la celda.

## <a name="create-a-code-cell-with-commands"></a>Creación de una celda de código con comandos

Tal y como se explicó para la celda Markdown anterior, puede incluir comandos directamente en el cuaderno. Puede usar comandos para instalar paquetes, ejecutar curl o wget para recuperar datos, u otras acciones. Los cuadernos de Jupyter se ejecutan bien en una máquina virtual Linux, lo que permite trabajar con el conjunto completo de comandos Linux.

1. Escriba los siguientes comandos en la celda de código que apareció después de usar **Run** (Ejecutar) en la celda Markdown anterior. Si no ve una nueva celda, cree una con **Insert** > **Insert Cell Below** (Insertar > Insertar celda debajo) o use el botón **+** de la barra de herramientas.

    ```bash
    !pip install numpy
    !pip install matplotlib
    !pip install pandas
    !pip install sklearn
    ```

1. Antes de ejecutar la celda, cree una nueva celda con el botón **+** de la barra de herramientas, cambie el tipo a Markdown y escriba la explicación siguiente:

    ```markdown
    Note that when you run a code block that contains install commands, and also those with `import` statements, it make take the notebooks a little time to complete the task. To the left of the code block you see `In [*]` to indicate that execution is happening. The Notebook's kernel on the upper right also shows a filled-in circle to indicate "busy."
    ```

1. Seleccione el comando **Cell** > **Run All** (Celda > Ejecutar todo) para ejecutar todas las celdas del cuaderno. Observe que las celdas Markdown se representan como HTML y el comando se ejecuta en el kernel, y observe el indicador de kernel tal y como se describe en el propio código Markdown:

    ![Indicador de ocupado del kernel del cuaderno](media/tutorial/tutorial-kernel-busy.png)

1. Todos los comandos `pip install` se ejecutan rápidamente y, como estos paquetes ya están instalados en el entorno del proyecto (y también están incluidos en Azure Notebooks de forma predeterminada), verá muchos mensajes del tipo "Requirement already satisfied" (Requisito ya cumplido). Como todas esas salidas pueden resultar una distracción a la vista, seleccione esa celda (con un clic) y use **Cell** > **Cell Outputs** > **Toggle** (Celda > Salidas de celda > Alternar) para ocultar la salida. También puede usar el comando **Clear** (Borrar) en ese mismo submenú para quitar todas las salidas.

    El comando **Toggle** (Alternar) oculta solo la salida más reciente de la celda; si vuelve a ejecutar la celda, la salida vuelve a aparecer.

1. Como los paquetes están instalados en el entorno del proyecto, marque como comentario los comandos `! pip install` mediante `#`; de esta forma, pueden permanecer en el cuaderno como material informativo, pero no ocuparán tiempo de ejecución y no producirán salidas innecesarias. En este caso, mantener los comandos como comentarios en el cuaderno también informa sobre las dependencias del cuaderno.

    ```bash
    # !pip install numpy
    # !pip install matplotlib
    # !pip install pandas
    # !pip install sklearn
    ```

## <a name="create-the-remaining-cells"></a>Creación de las celdas restantes

Para rellenar el resto del cuaderno, cree una serie de celdas del tipo Markdown y Code. Para cada una de las siguientes celdas, primero cree la nueva celda y, luego, establezca el tipo y pegue el contenido.

Aunque puede esperar para ejecutar el cuaderno después de crear cada celda, es interesante ejecutar cada celda al crearla. No todas las celdas muestran una salida; si no ve ningún error, puede dar por hecho que la celda se ha ejecutado con normalidad.

Cada celda de código depende del código que se han ejecutado en las celdas anteriores y, si no se ejecuta una de las celdas, las celdas siguientes pueden producir errores. Si se da cuenta de que ha olvidado ejecutar una celda, pruebe a usar **Cell** > **Run All Above** (Celda > Ejecutar todas las anteriores) antes de ejecutar la celda actual.

Si ve resultados inesperados (y probablemente los verá), compruebe que todas las celdas son del tipo "Code" o "Markdown" según sea necesario. Por ejemplo, un error "Invalid syntax" (Sintaxis no válida) normalmente se produce cuando se ha especificado Markdown en una celda de código.

1. Celda Markdown:

    ```markdown
    ## Import packages and prepare the dataset

    In this example we're using numpy, pandas, and matplotlib. Data is in the file cricket_chirps.csv. Because this file is in the same project as this present Notebook, we can just load it using a relative pathname.
    ```

1. Celda de código; cuando se ejecuta, muestra el contenido de la tabla como salida. Puede suprimir la salida marcando como comentario la instrucción `print`.

    ```python
    import numpy as np
    import pandas as pd

    dataset = pd.read_csv('cricket_chirps.csv')
    print(dataset)
    X = dataset.iloc[:, :-1].values  # Matrix of independent variables -- remove the last column in this data set
    y = dataset.iloc[:, 1].values    # Matrix of dependent variables -- just the last column (1 == 2nd column)
    ```

    > [!Note]
    > Quizás vea advertencias de este código como "numpy.dtype size changed"; estas advertencias pueden omitirse.

1. Celda Markdown:

    ```markdown
    Next, split the dataset into a Training set (2/3rds) and Test set (1/3rd). We don't need to do any feature scaling because there is only one column of independent variables, and packages typically do scaling for you.
    ```

1. Celda de código; cuando se ejecuta, esta celda no produce una salida.

    ```python
    from sklearn.model_selection import train_test_split

    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size = 1/3, random_state = 0)
    ```

1. Celda Markdown:

    ```markdown
    ## Fit the data to the training set

    "Fitting" the data to a training set means making the line that describes the relationship between the independent and the dependent variables. With a simple data set like we're using here, you can visualize the line on a simple x-y plot: the x-axis is the independent variable (chirp count in this example), and the y-axis is the independent variable (temperature). Fitting the data means plotting all the points in the training set, then drawing the best-fit line through that data.

    With two independent variables you can imagine a three-dimensional plot with a line fitted to the data. At three or more independent variables, however, it's no longer easy to visualize the fit, but you get the idea. In the end, it's all just mathematics, which a computer can handle easily without having to form a mental picture!

    The regressor's `fit` method here creates the line, which algebraically is of the form `y = x*b1 + b0`, where b1 is the coefficient or slope of the line (which you can get to through `regressor.coef_`), and b0 is the intercept of the line at x=0 (which you can get to through `regressor.intercept_`).
    ```

1. Celda de código; cuando se ejecuta, esta celda produce la salida `LinearRegression(copy_X=True, fit_intercept=True, n_jobs=None,normalize=False)`.

    ```python
    from sklearn.linear_model import LinearRegression

    regressor = LinearRegression()    # This object is the regressor, that does the regression
    regressor.fit(X_train, y_train)   # Provide training data so the machine can learn to predict using a learned model.
    ```

1. Celda Markdown:

    ```markdown
    ## Predict the results

    With the regressor in hand, we can predict the test set results using its `predict` method. That method takes a vector of independent variables for which you want predictions.

    Because the regressor is fit to the data by virtue of `coef_` and `intercept_`, a prediction is the result of `coef_ * x + intercept_`. (Indeed, `predict(0)` returns `intercept_` and `predict(1)` returns `intercept_ + coef_`.)

    In the code, the `y_test` matrix (from when we split the set) contains the real observations. `y_pred` assigned here contains the predictions for the same `X_test` inputs. It's not expected that the test or training points exactly fit the regression; the regression is trying to find the model that we can use to make predictions with new observations of the independent variables.
    ```

1. Celda de código; cuando se ejecuta, esta celda muestera resultados como `[79.49588055 75.98873911 77.87719989 80.03544077 75.17939878]`.

    ```python
    y_pred = regressor.predict(X_test)
    print(y_pred)
    ```

1. Celda Markdown:

    ```markdown
    It's interesting to think that all the "predictions" we use in daily life, like weather forecasts, are just regression models of some sort working with various data sets. Those models are much more complicated than what's shown here, but the idea is the same.

    Knowing how predictions work help us understand that the actual observations we would collect in the moment will always be somewhat off from the predictions: the predictions fit exactly to the model, whereas the observations typically won't.

    Of course, such systems feed new observations back into the dataset to continually improve the model, meaning that predictions should get more accurate over time.

    The challenge is determining what data to actually use. For example, with weather, how far back in time do you go? How have weather patterns been changing decade by decade? In any case, something like weather predictions will be doing things hour by hour, day by day, for things like temperature, precipitation, winds, cloud cover, etc. Radar and other observations are of course fed into the model and the predictions are reduced to mathematics.
    ```

1. Celda Markdown:

    ```markdown
    ## Visualize the results

    The following code generates a plot: green dots are training data, red dots are test data, blue dots are predictions. Gray line is the regression itself. You see that all the blue dots are exactly on the line, as they should be, because the predictions exactly fit the model (the line).
    ```

1. Celda de código; cuando se ejecuta, esta celda produce un trazado gráfico. Si no ve el trazado la primera vez (y en su lugar ve "Figure size 640x480 with 1 Axes"), vuelva a ejecuta la celda.

    ```python
    import matplotlib.pyplot as plt

    plt.scatter(X_train, y_train, color = 'green')
    plt.scatter(X_test, y_test, color = 'red')   
    plt.scatter(X_test, y_pred, color = 'blue')  # The predicted temperatures of the same X_test input.
    plt.plot(X_train, regressor.predict(X_train), color = 'gray')
    plt.title('Temperature based on chirp count')
    plt.xlabel('Chirps/minute')
    plt.ylabel('Temperature')
    plt.show()
    ```

    ![Salida de trazado del código de matplotlib](media/tutorial/tutorial-plot-output.png)

1. Celda Markdown:

    ```markdown
    ## Closing comments

    At the end of the day, when you create a model, you use training data. Then you start feeding test data (real observations) to see how well the model actually works. You may find that the model is a little inaccurate over time, in which case you retrain the model with some new data. Retraining the model means you're creating a new fit line that's used for predictions.

    This regression can be used to examine the variability of the relationship between temperature and chirp count. Of course, if the model proves too inaccurate (that is, the predictions aren't very good), then it suggests that we might need to introduce more independent variables like humidity, time of year, latitude, amount of moonlight, and so on. If you have such data, you can do separate lines regressions for each independent variable, and then do multiple regressions with combinations of independent variables. 

    Again, once you are working with more than one or two independent variables, it's much easier to use machine learning to crunch the numbers than to try to visualize it graphically.
    ```

## <a name="clear-outputs-and-rerun-all-cells"></a>Borrado de las salidas y nueva ejecución de todas las celdas

Después de seguir los pasos descritos en la sección anterior para rellenar todo el cuaderno, ha creado un fragmento de código en ejecución en el contexto de un tutorial completo sobre la regresión lineal. Esta combinación directa de código y texto es una de las grandes ventajas de los cuadernos.

Ahora, vuelva a ejecutar el cuaderno completo:

1. Borre todos los datos de la sesión del kernel y todas las salidas de las celdas; para ello, seleccione **Kernel** > **Restart & Clear Output** (Kernel > Reiniciar y borrar salida). Siempre es conveniente ejecutar este comando cuando se ha completado un cuaderno, simplemente para asegurarse de que no se han creado dependencias extrañas entre las celdas de código.

1. Vuelva a ejecutar el cuaderno con **Cell** > **Run All** (Celda > Ejecutar todo). Tenga en cuenta que el indicador de kernel se rellena mientras se ejecuta el código.

    Si tiene un código que se ejecuta durante demasiado tiempo o se queda bloqueado, puede detener el kernel con el comando **Kernel** > **Interrupt** (Kernel > Interrumpir).

1. Desplácese por el cuaderno para examinar los resultados. (De nuevo, si el trazado no aparece, vuelva a ejecutar esa celda).

## <a name="save-halt-and-close-the-notebook"></a>Guardado, detención y cierre del cuaderno

Durante el tiempo que está editando un cuaderno, puede guardar el estado actual con el comando **File** > **Save and Checkpoint** (Archivo > Guardar y punto de comprobación) o con el botón de guardado de la barra de herramientas. Un "punto de comprobación" crea una instantánea que puede revertir en cualquier momento durante la sesión. Los puntos de comprobación le permiten realizar una serie de cambios experimentales y, si esos cambios no funcionan, puede volver a un punto de comprobación con el comando **File** > **Revert to Checkpoint** (Archivo > Revertir al punto de comprobación). Otro método es crear celdas adicionales y marcar como comentario todo el código que no desea ejecutar; funciona de las dos maneras.

También puede usar el comando **File** > **Make a Copy** (Archivo > Crear una copia) en cualquier momento para realizar una copia del estado actual del cuaderno en un nuevo archivo del proyecto. Esa copia se abre automáticamente en una nueva pestaña del explorador.

Cuando haya terminado con un cuaderno, use el comando la **File** > **Close and halt** (Archivo > Cerrar y detener) para cerrar el cuaderno y apagar el kernel que lo ejecutaba. Después, Azure Notebooks cierra la pestaña del explorador automáticamente.

## <a name="debug-notebooks-using-visual-studio-code"></a>Depuración de cuadernos con Visual Studio Code

Si las celdas de código del cuaderno no se comportan de la manera esperada, quizás haya errores de código u otros defectos. Sin embargo, aparte de usar instrucciones `print` para mostrar el valor de las variables, un entorno típico de Jupyter no ofrece funcionalidades de depuración.

Afortunadamente, puede descargar el archivo *.ipynb* del cuaderno y, después, abrirlo en Visual Studio Code con la extensión de Python. La extensión importa directamente un cuaderno como un solo archivo de código, conservando las celdas de Markdown en los comentarios. Una vez importado el cuaderno, puede usar el depurador de Visual Studio Code para recorrer el código, establecer puntos de interrupción, examinar el estado, etc. Después de realizar correcciones en el código, exporte el archivo *.ipynb* de Visual Studio Code y cárguelo en Azure Notebooks.

Para más información, consulte el artículo [Debug a Jupyter notebook](https://code.visualstudio.com/docs/python/jupyter-support#debug-a-jupyter-notebook) (Depurar un cuaderno de Jupyter) en la documentación de Visual Studio Code.

Consulte también [Visual Studio Code - Jupyter support](https://code.visualstudio.com/docs/python/jupyter-support) (Visual Studio Code: compatibilidad con Jupyter) para conocer las características adicionales de Visual Studio Code para cuadernos de Jupyter.

## <a name="next-steps"></a>Pasos siguientes

- [Exploración de cuadernos de ejemplo](azure-notebooks-samples.md)

Artículos de procedimientos:

- [Creación y clonación de proyectos](create-clone-jupyter-notebooks.md)
- [Configuración y administración de proyectos](configure-manage-azure-notebooks-projects.md)
- [Instalación de paquetes en un cuaderno](install-packages-jupyter-notebook.md)
- [Presentación](present-jupyter-notebooks-slideshow.md)
- [Uso de archivos de datos](work-with-project-data-files.md)
- [Acceso a recursos de datos](access-data-resources-jupyter-notebooks.md)
- [Uso de Azure Machine Learning](use-machine-learning-services-jupyter-notebooks.md)
