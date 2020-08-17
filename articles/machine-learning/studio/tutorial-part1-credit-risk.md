---
title: 'Tutorial de ML Studio (clásico): Predicción del riesgo crediticio: Azure'
description: Tutorial detallado que muestra cómo crear una solución de análisis predictivo para la evaluación del riesgo de crédito en Azure Machine Learning Studio (clásico). Este tutorial es el primero de una serie de tres partes.  Muestra cómo crear un área de trabajo, cargar datos y crear un experimento.
keywords: riesgo de crédito, solución de análisis predictivo, evaluación de riesgos
author: sdgilley
ms.author: sgilley
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: tutorial
ms.date: 02/11/2019
ms.openlocfilehash: 556dc89d3f1d0fd42ab6d161a6d5ccb462f555db
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87431849"
---
# <a name="tutorial-1-predict-credit-risk---azure-machine-learning-studio-classic"></a>Tutorial 1: Predicción del riesgo crediticio en Azure Machine Learning Studio (clásico)

**SE APLICA A:**  ![yes](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico)   ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)


[!INCLUDE [Designer notice](../../../includes/designer-notice.md)]

En este tutorial se explica con detalle el proceso de desarrollo de una solución de análisis predictivo. Va a desarrollar un modelo sencillo en Machine Learning Studio (clásico).  Después puede implementar el modelo como un servicio web de Azure Machine Learning.  Este modelo implementado puede hacer predicciones con datos nuevos. Este tutorial es el **primero de una serie de tres partes**.

Suponga que necesita predecir el riesgo de crédito de un individuo en función de la información que se proporcionó en una solicitud de crédito.  

La evaluación de riesgos crediticios es un problema complejo, pero en este tutorial se simplificará un poco. Se utilizará como ejemplo de cómo puede crear una solución de análisis predictivo con Microsoft Azure Machine Learning Studio (clásico). En esta solución se usará Azure Machine Learning Studio (clásico) y un servicio web Machine Learning.  

En este tutorial de tres partes, vamos a comenzar con los datos de riesgo crediticio disponibles públicamente.  Después, desarrollaremos y entrenaremos un modelo predictivo.  Finalmente, vamos a implementar el modelo como servicio web.

En esta parte del tutorial, se va a ver lo siguiente: 
 
> [!div class="checklist"]
> * Creación de un área de trabajo de Machine Learning Studio (clásico)
> * Carga de datos existentes
> * Creación de un experimento

Después, puede usar este experimento para [entrenar modelos en la parte 2](tutorial-part2-credit-risk-train.md) y, finalmente, [implementarlos en la parte 3](tutorial-part3-credit-risk-deploy.md).

## <a name="prerequisites"></a>Requisitos previos

En este tutorial, se presupone que usó Machine Learning Studio (clásico) con anterioridad al menos una vez y que tiene ciertos conocimientos sobre los conceptos de aprendizaje automático. Pero no se asume de que sea un experto.

Si nunca ha utilizado **Azure Machine Learning Studio (clásico)** , sería conveniente que realizara primero el inicio rápido [Creación del primer experimento de ciencia de datos en Azure Machine Learning Studio (clásico)](create-experiment.md). Este inicio rápido lo guiará por primera vez por Machine Learning Studio (clásico). Aquí se muestran los conceptos básicos de cómo arrastrar y colocar módulos en el experimento, conectarlos, ejecutar el experimento y examinar los resultados.


> [!TIP] 
> Puede encontrar una copia de trabajo del experimento que se ha desarrollado en este tutorial en [Azure AI Gallery](https://gallery.azure.ai). Vaya **[Tutorial: predicción de riesgos de crédito](https://gallery.azure.ai/Experiment/Walkthrough-Credit-risk-prediction-1)** y haga clic en **Abrir en Studio** para descargar una copia del experimento en el área de trabajo de Machine Learning Studio (clásico).
> 


## <a name="create-a-machine-learning-studio-classic-workspace"></a>Creación de un área de trabajo de Machine Learning Studio (clásico)

Para usar Machine Learning Studio (clásico), debe tener un área de trabajo de Microsoft Azure Machine Learning Studio. Esta área de trabajo contiene las herramientas que necesita para crear, administrar y publicar experimentos.  

Para crear un área de trabajo, consulte [Creación y uso compartido de un área de trabajo de Azure Machine Learning Studio (clásico)](create-workspace.md).

Una vez haya creado el área de trabajo, abra Machine Learning Studio (clásico) ([https://studio.azureml.net/Home](https://studio.azureml.net/Home)). Si tiene más de un área de trabajo, puede seleccionar la que desee en la barra de herramientas de la esquina superior derecha de la ventana.

![Selección del área de trabajo en Studio (clásico)](./media/tutorial-part1-credit-risk/open-workspace.png)

> [!TIP]
> Si es el propietario del área de trabajo, puede compartir los experimentos en los que esté trabajando invitando a otros al área. Para ello, en Machine Learning Studio (clásico), vaya a la página **SETTINGS** (CONFIGURACIÓN). Solo necesita la cuenta Microsoft o la cuenta de organización de cada usuario.
> 
> En la página **CONFIGURACIÓN**, haga clic en **USUARIOS** y, después, haga clic en **INVITE MORE USERS** (INVITAR A MÁS USUARIOS) en la parte inferior de la ventana.
> 

## <a name="upload-existing-data"></a><a name="upload"></a>Carga de datos existentes

Para desarrollar un modelo de predicción de riesgo de crédito, se necesitan datos para entrenar y probar el modelo. Para este tutorial, se usará el conjunto de datos "UCI Statlog (German Credit Data)" del repositorio de Machine Learning de UC Irvine. Puede encontrarlo aquí:  
<a href="https://archive.ics.uci.edu/ml/datasets/Statlog+(German+Credit+Data)">https://archive.ics.uci.edu/ml/datasets/Statlog+(German+Credit+Data)</a>

Usaremos el archivo llamado **german.data**. Descargue este archivo en la unidad de disco duro local.  

El conjunto de datos **german.data** contiene filas de 20 variables para 1000 solicitantes de crédito. Estas 20 variables representan el conjunto de características (el *vector de características*) del conjunto de datos que proporciona características de identificación de cada solicitante de crédito. Una columna adicional en cada fila representa el riesgo de crédito calculado del solicitante, donde 700 solicitantes se identificaron como de bajo riesgo y 300 como de alto riesgo.

El sitio web de UCI proporciona una descripción de los atributos del vector de características de estos datos. Entre estos datos figuran la información financiera, el historial de crédito, el estado de empleo y la información personal. A cada solicitante se le ha dado una calificación binaria para indicar si son de riesgo de crédito alto o bajo. 

Estos datos se usarán para entrenar un modelo de análisis predictivo. Cuando se haya terminado, el modelo debe poder aceptar un vector de características para una nueva persona y predecir si esta presenta un alto o bajo riesgo de crédito.  

Aquí hay un giro interesante.

La descripción del conjunto de datos en el sitio web de UCI menciona lo que cuesta si se clasifica erróneamente el riesgo de crédito de una persona.
Si el modelo predice un riesgo de crédito alto para un usuario que realmente tiene un riesgo de crédito bajo, el modelo ha realizado una clasificación incorrecta.

Pero las clasificaciones inversas incorrectas son cinco veces más costosas para la institución financiera: si el modelo predice un riesgo de crédito bajo para un usuario que realmente tiene un riesgo de crédito alto.

Por lo tanto, es deseable entrenar el modelo para que el costo de este último tipo de clasificación incorrecta sea cinco veces mayor que clasificar erróneamente de la otra forma.

Una forma sencilla de hacerlo al entrenar el modelo en el experimento es duplicar (cinco veces) esas entradas que representan a alguien con un riesgo de crédito alto. 

A continuación, si el modelo clasifica erróneamente a una persona como de riesgo de crédito bajo cuando realmente tiene un riesgo alto, el modelo realiza esa misma clasificación incorrecta cinco veces, una vez para cada duplicado. Esto aumentará el coste de este error en los resultados del entrenamiento.


### <a name="convert-the-dataset-format"></a>Conversión del formato del conjunto de datos

El conjunto de datos original utiliza un formato separado por espacios en blanco. Machine Learning Studio (clásico) funciona mejor con un archivo de valores separados por comas (CSV), así que va a convertir el conjunto de datos y reemplazará los espacios por comas.  

Hay muchas maneras de convertir los datos. En primer lugar, es posible convertirlos con el siguiente comando de Windows PowerShell:   

```powershell
cat german.data | %{$_ -replace " ",","} | sc german.csv  
```

También se puede hacer con el comando sed de Unix:  

```console
sed 's/ /,/g' german.data > german.csv
```

En cualquier caso, se creará una versión separada por comas de los datos en un archivo denominado **german.csv** que se puede usar en el experimento.

### <a name="upload-the-dataset-to-machine-learning-studio-classic"></a>Carga del conjunto de datos en Machine Learning Studio (clásico)

Una vez que los datos se han convertido al formato CSV, hay que cargarlos en Machine Learning Studio (clásico). 

1. Abra la página principal de Machine Learning Studio (clásico) ([https://studio.azureml.net](https://studio.azureml.net)). 

2. Haga clic en el ![menú](./media/tutorial-part1-credit-risk/menu.png) de la esquina superior izquierda de la ventana, haga clic en **Azure Machine Learning**, seleccione **Studio** e inicie sesión.

3. Haga clic en **+NUEVO** en la parte inferior de la página.

4. Seleccione **CONJUNTO DE DATOS**.

5. Seleccione **DE ARCHIVO LOCAL**.

    ![Adición de un conjunto de datos desde un archivo local](./media/tutorial-part1-credit-risk/add-dataset.png)

6. En el diálogo **Cargar un nuevo conjunto de datos**, haga clic en Examinar y busque el archivo **german.csv** que ha creado.

7. Escriba un nombre para el conjunto de datos. En este tutorial, se denominará "UCI German Credit Card Data".

8. Para el tipo de datos, seleccione **Archivo CSV genérico sin encabezado (.nh.csv)** .

9. Agregue una descripción si lo desea.

10. Haga clic en la marca de verificación **Aceptar**.  

    ![Carga del conjunto de datos](./media/tutorial-part1-credit-risk/upload-dataset.png)

De esta manera los datos se cargan en un módulo de conjunto de datos que se pueden usar en un experimento.

Para administrar los conjuntos de datos que cargó en Studio (clásico), haga clic en la pestaña **CONJUNTOS DE DATOS** a la izquierda de la ventana de Studio (clásico).

![Administración de conjuntos de datos](./media/tutorial-part1-credit-risk/dataset-list.png)

Para más información sobre la importación de diversos tipos de datos a un experimento, consulte [Importación de datos de entrenamiento en Azure Machine Learning Studio (clásico)](import-data.md).

## <a name="create-an-experiment"></a>Creación de un experimento

El siguiente paso de este tutorial es crear un experimento en Machine Learning Studio (clásico) que use el conjunto de datos cargado.  

1. En Studio (clásico), haga clic en **+NUEVO** en la parte inferior de la ventana.
1. Seleccione **EXPERIMENTO** y luego ''Experimento en blanco''. 

    ![Creación de un nuevo experimento](./media/tutorial-part1-credit-risk/create-new-experiment.png)


1. Seleccione el nombre del experimento predeterminado en la parte superior del lienzo y cámbielo por uno significativo.

    ![Renombrar el experimento](./media/tutorial-part1-credit-risk/rename-experiment.png)

   > [!TIP]
   > Se recomienda rellenar los campos **Resumen** y **Descripción** del experimento en el panel **Propiedades**. Estas propiedades ofrecen la oportunidad para documentar el experimento para que cualquier persona que lo vea posteriormente entienda sus objetivos y la metodología.
   > 
   > ![Propiedades del experimento](./media/tutorial-part1-credit-risk/experiment-properties.png)
   > 

1. En la paleta de módulos, a la izquierda del lienzo de experimentos, expanda **Conjuntos de datos guardados**.
1. Busque el conjunto de datos que ha creado en **Mis conjuntos de datos** y arrástrelo al lienzo. También puede buscar el conjunto de datos escribiendo su nombre en el cuadro **Buscar** que está encima de la paleta.  

    ![Agregar el conjunto de datos al experimento](./media/tutorial-part1-credit-risk/add-dataset-to-experiment.png)


### <a name="prepare-the-data"></a>Preparación de los datos

Para ver las primeras 100 filas de datos y alguna información estadística de todo el conjunto de datos: haga clic en el puerto de salida del conjunto de datos (el círculo pequeño de la parte inferior) y seleccione **Visualizar**.  

Dado que el archivo de datos no incluye encabezados de columna, Studio (clásico) ha proporcionado encabezados genéricos (Col1, Col2, *etc.* ). No es esencial que los encabezados sean perfectos para crear un modelo, pero facilitan el trabajo con los datos del experimento. Además, cuando finalmente se publique este modelo en un servicio web, los encabezados ayudarán al usuario del servicio a identificar las columnas.  

Se pueden agregar encabezados de columna mediante el módulo [Edit Metadata][edit-metadata] (Editar metadatos).

Use el módulo [Edit Metadata][edit-metadata] (Editar metadatos) para cambiar los metadatos asociados a un conjunto de datos. En este caso, se usa para proporcionar nombres más descriptivos para los encabezados de las columnas. 

Para usar [Edit Metadata][edit-metadata] (Editar metadatos), especifique primero las columnas que desea modificar (en este caso, todas). A continuación, especifique la acción que desea realizar en esas columnas (en este caso, cambiar los encabezados de las columnas).

1. En la paleta de módulos, escriba "metadatos" en el cuadro **Buscar** . El módulo [Edit Metadata][edit-metadata] (Editar metadatos) aparece en la lista de módulos.

1. Haga clic en el módulo [Edit Metadata][edit-metadata] (Editar metadatos), arrástrelo al lienzo y colóquelo bajo el conjunto de datos agregado anteriormente.

1. Conecte el conjunto de datos al módulo [Edit Metadata][edit-metadata] (Editar metadatos): haga clic en el puerto de salida del conjunto de datos (el círculo pequeño de la parte inferior del conjunto de datos), arrástrelo al puerto de entrada del módulo [Edit Metadata][edit-metadata] (Editar metadatos) (el círculo pequeño de la parte superior del módulo) y luego suelte el botón del ratón. El conjunto de datos y el módulo permanecen conectados aunque se desplace por el lienzo.
 
    El experimento debería tener ahora un aspecto similar al siguiente:  

    ![Agregar metadatos de edición](./media/tutorial-part1-credit-risk/experiment-with-edit-metadata-module.png)

    El signo de exclamación rojo indica que no se han configurado aún las propiedades de este módulo. Se hará a continuación.

    > [!TIP]
    > Puede agregar un comentario a un módulo; para ello, haga doble clic en el módulo y escriba algún texto. Esto puede ayudarle a ver de un vistazo lo que el módulo hace en el experimento. En este caso, haga doble clic en el módulo [Edit Metadata][edit-metadata] (Editar metadatos) y escriba el comentario "Agregar encabezados de columna". Haga clic en cualquier lugar del lienzo para cerrar el cuadro de texto. Para mostrar el comentario, haga clic en la flecha abajo en el módulo.
    > 
    > ![Modificación del módulo de metadatos con el comentario agregado](./media/tutorial-part1-credit-risk/edit-metadata-with-comment.png)
    > 

1. Seleccione [Edit Metadata][edit-metadata] (Editar metadatos) y, en el panel **Propiedades** a la derecha del lienzo, haga clic en **Launch column selector** (Iniciar el selector de columnas).

1. En el cuadro de diálogo **Seleccionar columnas**, elija todas las filas de **Columnas disponibles** y haga clic en > para moverlas a **Columnas seleccionadas**.
   El cuadro de diálogo debe ser similar al siguiente:

   ![Selector de columnas con todas las columnas seleccionadas](./media/tutorial-part1-credit-risk/select-columns.png)


1. Haga clic en la marca de verificación **Aceptar**.

1. En el panel **Propiedades**, busque el parámetro **Nuevo nombre de columna**. En este campo, escriba la lista de nombres de las 21 columnas del conjunto de datos, separadas por comas y en el orden de las columnas. Puede obtener los nombres de las columnas en la documentación del conjunto de datos en el sitio web de UCI o, para mayor comodidad, puede copiar y pegar la siguiente lista:  

   ```   
   Status of checking account, Duration in months, Credit history, Purpose, Credit amount, Savings account/bond, Present employment since, Installment rate in percentage of disposable income, Personal status and sex, Other debtors, Present residence since, Property, Age in years, Other installment plans, Housing, Number of existing credits, Job, Number of people providing maintenance for, Telephone, Foreign worker, Credit risk  
   ```

   El panel Propiedades tiene un aspecto similar al siguiente:

   ![Propiedades de los metadatos de edición](./media/tutorial-part1-credit-risk/edit-metadata-properties.png)

   > [!TIP]
   > Si desea comprobar los encabezados de columna, ejecute el experimento (haga clic en **EJECUTAR** debajo del lienzo del experimento). Cuando termine de ejecutarse —aparece una marca de verificación verde en [Edit Metadata][edit-metadata] (Editar metadatos)—, haga clic en el puerto de salida del módulo [Edit Metadata][edit-metadata] (Editar metadatos) y seleccione **Visualizar**. Puede ver el resultado de cualquier módulo de la misma manera, para visualizar el progreso de los datos a lo largo del experimento.
   > 
   > 

### <a name="create-training-and-test-datasets"></a>Creación de conjuntos de datos de entrenamiento y prueba

Se necesitan algunos datos para entrenar el modelo y otros tantos para probarlo.
De este modo, en el siguiente paso del experimento, se divide el conjunto de datos en dos conjuntos de datos independientes: uno para el entrenamiento de nuestro modelo y el otro para probarlo.

Para ello, se usa el módulo [Split Data][split] (Dividir datos).  

1. Busque el módulo [Split Data][split] (Dividir datos), arrástrelo al lienzo y conéctelo al módulo [Edit Metadata][edit-metadata] (Editar metadatos).

1. De manera predeterminada, la proporción de división es 0,5 y se establece el parámetro **División aleatoria** . Esto significa que una mitad aleatoria de los datos sale por un puerto del módulo [Split Data][split] (Dividir datos) y la otra mitad, por el otro. Puede cambiar estos parámetros, así como el parámetro **Valor de inicialización aleatorio**, para cambiar la división entre datos de entrenamiento y de prueba. En este ejemplo, se dejan tal cual.
   
   > [!TIP]
   > La propiedad **Fraction of rows in the first output dataset** (Fracción de filas del primer conjunto de datos de salida) determina la cantidad de datos que salen a través del puerto de salida de la *izquierda*. Por ejemplo, si establece la proporción en 0,7, el 70 % de los datos sale por el puerto de la izquierda y el 30 % por el puerto de la derecha.  
   > 
   > 

1. Haga doble clic en el módulo [Split Data][split] (Dividir datos) y escriba el comentario "Dividir 50 % de los datos de entrenamiento y pruebas". 

Se pueden utilizar las salidas del módulo [Split Data][split] (Dividir datos) como se quiera, pero se va a optar por utilizar la salida de la izquierda como datos de entrenamiento y la salida de la derecha como datos de pruebas.  

Como se menciona en el [paso anterior](tutorial-part1-credit-risk.md#upload), el costo de clasificar erróneamente un riesgo de crédito alto como bajo es cinco veces más alto que el costo de clasificar erróneamente un riesgo de crédito bajo como alto. Para tener esto en cuenta, se debe generar un nuevo conjunto de datos que refleje esta función de costo. En el nuevo conjunto de datos, cada ejemplo de alto riesgo se replica cinco veces, mientras que los ejemplos de bajo riesgo no se replican.   

Podemos conseguir esta replicación mediante el código R:  

1. Busque el módulo [Execute R Script][execute-r-script] (Ejecutar script R) y arrástrelo al lienzo del experimento. 

1. Conecte el puerto de salida de la izquierda del módulo [Split Data][split](Dividir datos) al primer puerto de entrada ("Dataset1") del módulo [Execute R Script][execute-r-script] (Ejecutar script R).

1. Haga doble clic en el módulo [Execute R Script][execute-r-script] (Ejecutar script R) y escriba el comentario "Establecer ajuste de costos".

1. En el panel **Propiedades**, elimine el texto predeterminado del parámetro **Script R** y escriba este script:
   
    ```r
    dataset1 <- maml.mapInputPort(1)
    data.set<-dataset1[dataset1[,21]==1,]
    pos<-dataset1[dataset1[,21]==2,]
    for (i in 1:5) data.set<-rbind(data.set,pos)
    maml.mapOutputPort("data.set")
    ```

    ![Script R en el módulo Execute R Script (Ejecutar script R)](./media/tutorial-part1-credit-risk/execute-r-script.png)

Hay que hacer esta misma operación de replicación para cada salida del módulo [Split Data][split] (Dividir datos) para que los datos de entrenamiento y prueba tengan el mismo ajuste con relación al costo. La forma más sencilla de hacerlo consiste en duplicar el módulo [Execute R Script][execute-r-script] (Ejecutar script R) que se acaba de crear y conectarlo al otro puerto de salida del módulo [Split Data][split] (Dividir datos).

1. Haga clic con el botón derecho en el módulo [Execute R Script][execute-r-script] (Ejecutar script R) y seleccione **Copy** (Copiar).

1. Haga clic con el botón derecho en el lienzo del experimento y seleccione **Pegar**.

1. Arrastre el nuevo módulo a la posición correspondiente y luego conecte el puerto de salida de la derecha del módulo [Split Data][split] (Dividir datos) al primer puerto de entrada de este nuevo módulo [Execute R Script][execute-r-script] (Ejecutar script R). 

1. En la parte inferior del lienzo, haga clic en **Ejecutar**. 

> [!TIP]
> La copia del módulo Ejecutar script R contiene el mismo script que el módulo original. Al copiar y pegar un módulo en el lienzo, la copia retiene todas las propiedades del original.  
> 
>

Nuestro experimento tiene ahora un aspecto similar al siguiente:

![Adding Split module and R scripts](./media/tutorial-part1-credit-risk/experiment.png)

Para obtener más información sobre cómo usar los scripts de R en sus experimentos, consulte [Extender el experimento con R](extend-your-experiment-with-r.md).


## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [machine-learning-studio-clean-up](../../../includes/machine-learning-studio-clean-up.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha completado estos pasos: 
 
> [!div class="checklist"]
> * Creación de un área de trabajo de Machine Learning Studio (clásico)
> * Carga de datos existentes en el área de trabajo
> * Creación de un experimento

Ahora está preparado para entrenar y evaluar modelos para estos datos.

> [!div class="nextstepaction"]
> [Tutorial 2: Entrenamiento y evaluación de los modelos](tutorial-part2-credit-risk-train.md)

<!-- Module References -->
[execute-r-script]: https://docs.microsoft.com/azure/machine-learning/studio-module-reference/execute-r-script
[edit-metadata]: https://docs.microsoft.com/azure/machine-learning/studio-module-reference/edit-metadata
[split]: https://docs.microsoft.com/azure/machine-learning/studio-module-reference/split-data
