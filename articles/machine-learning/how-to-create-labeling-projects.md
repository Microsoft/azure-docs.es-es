---
title: Creación de un proyecto de etiquetado de datos
titleSuffix: Azure Machine Learning
description: Aprenda a crear y ejecutar proyectos para etiquetar los datos del aprendizaje automático.  Las herramientas incluyen el etiquetado con asistencia de ML o el etiquetado con intervención humana para ayudar con la tarea.
author: sdgilley
ms.author: sgilley
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.date: 07/27/2020
ms.openlocfilehash: e74d22d3d45079a6568f6fca35dc5d84e2d7469f
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90897973"
---
# <a name="create-a-data-labeling-project-and-export-labels"></a>Creación de un proyecto de etiquetado de datos y exportación de etiquetas 



Etiquetar datos voluminosos en proyectos de Machine Learning suele ser una tarea compleja. Los proyectos que tienen un componente de Computer Vision (como la clasificación de imágenes o la detección de objetos) normalmente requieren etiquetar miles de imágenes.
 
El etiquetado de datos de [Azure Machine Learning](https://ml.azure.com/) proporciona una ubicación central para crear, administrar y supervisar proyectos de etiquetado. Úselo para coordinar los datos, las etiquetas y los miembros del equipo para administrar de forma eficaz las tareas de etiquetado. Machine Learning permite la clasificación de imágenes (de varias etiquetas y varias clases) y la identificación de objetos mediante rectángulos de selección.

El etiquetado de datos realiza un seguimiento del progreso y el mantenimiento de la cola de tareas de etiquetado incompletas.

Puede iniciar y detener el proyecto y controlar el progreso de etiquetado. Puede revisar los datos etiquetados y exportarlos en formato COCO o como un conjunto de datos de Azure Machine Learning.

> [!Important]
> Actualmente solo se admiten proyectos de etiquetado de clasificación de imágenes e identificación de objetos. Además, las imágenes de datos deben estar disponibles en un almacén de datos de blobs de Azure. (Si no tiene un almacén de datos existente, puede cargar las imágenes durante la creación del proyecto).

En este artículo, aprenderá a:

> [!div class="checklist"]
> * Crear un proyecto
> * Especificar los datos y la estructura del proyecto
> * Ejecutar y supervisar el proyecto
> * Exportar las etiquetas


## <a name="prerequisites"></a>Requisitos previos

* Los datos que quiere etiquetar, ya sea en archivos locales o en el almacenamiento de blobs de Azure.
* Conjunto de etiquetas que quiere aplicar.
* Instrucciones para el etiquetado.
* Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://aka.ms/AMLFree) antes de empezar.
* Un área de trabajo de Machine Learning. Consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

## <a name="create-a-labeling-project"></a>Crear un proyecto de etiquetado

Los proyectos de etiquetado se administran desde Azure Machine Learning. Use la página **Proyectos de etiquetado** para administrar sus proyectos.

Si los datos ya están en el almacenamiento de blobs de Azure, debe hacer que estén disponibles como un almacén de datos antes de crear el proyecto de etiquetado. Para obtener un ejemplo del uso de un almacén de datos, consulte [Tutorial: Creación de un proyecto de etiquetado para la clasificación de imágenes](tutorial-labeling.md).

Para crear un proyecto, seleccione **Agregar proyecto**. Asigne un nombre adecuado al proyecto y seleccione **Tipo de tarea de etiquetado**.

:::image type="content" source="media/how-to-create-labeling-projects/labeling-creation-wizard.png" alt-text="Asistente para la creación de proyectos de etiquetado":::

* Elija un proyecto de tipo **Clasificación de imágenes con varias clases** cuando quiera aplicar una *sola etiqueta* de un conjunto de etiquetas a una imagen.
* Elija un proyecto de tipo **Clasificación de imágenes con varias etiquetas** cuando quiera aplicar *una o varias* etiquetas de un conjunto de etiquetas a una imagen. Por ejemplo, una fotografía de un perro podría etiquetarse como *perro* y *diurno*.
* Elija un proyecto de tipo **Identificación del objeto (rectángulo de selección)** cuando quiera asignar una etiqueta y un rectángulo de selección a cada objeto de una imagen.

Seleccione **Siguiente** cuando esté listo para continuar.

## <a name="specify-the-data-to-label"></a>Especificación de los datos que se van a etiquetar

Si ya ha creado un conjunto de datos que contiene los datos, selecciónelo en la lista desplegable **Seleccione un conjunto de datos existente**. O bien, seleccione **Crear un conjunto de datos** para usar un almacén de información de Azure existente o cargar archivos locales.

> [!NOTE]
> Un proyecto no puede contener más de 500 000 imágenes.  Si el conjunto de datos las tiene, solo se cargarán las primeras 500 000.  

### <a name="create-a-dataset-from-an-azure-datastore"></a>Creación de un conjunto de datos a partir de un almacén de datos de Azure

En muchos casos, basta con cargar archivos locales. Sin embargo, [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) es una forma más rápida y eficaz de transferir una gran cantidad de datos. Se recomienda usar Explorador de Storage de forma predeterminada para migrar archivos.

Para crear un conjunto de datos a partir de los datos que ya ha almacenado en el almacenamiento de blobs de Azure:

1. Seleccione **Crear un conjunto de datos** > **De almacén de datos**.
1. En **Nombre**, asigne un nombre al conjunto de datos.
1. Elija **Archivo** como **Tipo de conjunto de datos**.  Solo se admiten tipos de conjunto de datos de archivo.
1. Seleccione el almacén de datos.
1. Si los datos están en una subcarpeta del almacenamiento de blobs, elija **Examinar** para seleccionar la ruta de acceso.
    * Anexe "/**" a la ruta de acceso para incluir todos los archivos que haya en las subcarpetas de la ruta de acceso seleccionada.
    * Anexe "* */* .*" para incluir todos los datos que haya en el contenedor actual y sus subcarpetas.
1. Proporcione una descripción para el conjunto de datos.
1. Seleccione **Next** (Siguiente).
1. Confirme los detalles. Seleccione **Atrás** para modificar la configuración o **Crear** para crear el conjunto de datos.


### <a name="create-a-dataset-from-uploaded-data"></a>Creación de un conjunto de datos a partir de los datos cargados

Para cargar los datos directamente:

1. Seleccione **Crear un conjunto de datos** > **De archivos locales**.
1. En **Nombre**, asigne un nombre al conjunto de datos.
1. Elija "Archivo" como **Tipo de conjunto de datos**.
1. *Opcional:* Seleccione **Configuración avanzada** para personalizar el almacén de datos, el contenedor y la ruta de acceso a los datos.
1. Seleccione **Examinar** para seleccionar los archivos locales que se van a cargar.
1. Proporcione una descripción para el conjunto de datos.
1. Seleccione **Next** (Siguiente).
1. Confirme los detalles. Seleccione **Atrás** para modificar la configuración o **Crear** para crear el conjunto de datos.

Los datos se cargan en el almacén de blobs predeterminado ("workspaceblobstore") del área de trabajo de Machine Learning.

## <a name="configure-incremental-refresh"></a><a name="incremental-refresh"> </a>Configuración de la actualización incremental

Si planea agregar nuevas imágenes al conjunto de datos, utilice la actualización incremental para agregar estas imágenes al proyecto.   Cuando la opción de **actualización incremental**  está habilitada, se comprueba periódicamente si hay nuevas imágenes el conjunto de datos que puedan agregarse a un proyecto en función de la tasa de finalización de la etiqueta.   La comprobación de los nuevos datos se detiene cuando el proyecto alcanza el número máximo de 500 000 imágenes.

Para agregar más imágenes al proyecto, use el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) para cargarlas en la carpeta adecuada del almacenamiento de blobs. 

Active la casilla **Habilitar la actualización incremental** cuando desee que el proyecto supervise continuamente los nuevos datos en el almacén de datos.

Anule la selección de esta casilla si no desea que las imágenes nuevas que aparecen en el almacén de datos se agreguen a su proyecto.

Puede encontrar la marca de tiempo de la última actualización en la sección **Actualización incremental** de la pestaña **Detalles** del proyecto.


## <a name="specify-label-classes"></a>Especificación de clases de etiquetas

En la página **Clases de etiquetas**, especifique el conjunto de clases para clasificar los datos. Hágalo con cuidado, porque la precisión y la velocidad de los etiquetadores depende de su capacidad para elegir entre las clases. Por ejemplo, en lugar de deletrear el género y la especie completos para plantas o animales, use códigos de campo o abrevie el género.

Escriba una etiqueta por fila. Use el botón **+** para agregar una nueva fila. Si tiene más de 3 o 4 etiquetas pero menos de 10, quizás quiera añadir a los nombres un prefijo de número ("1:", "2:") para que los etiquetadores puedan usar las teclas numéricas para acelerar su trabajo.

## <a name="describe-the-labeling-task"></a>Descripción de la tarea de etiquetado

Es importante explicar claramente la tarea de etiquetado. En la página **Instrucciones de etiquetado**, puede agregar un vínculo a un sitio externo que contenga las instrucciones de etiquetado o bien incluir instrucciones en el cuadro de edición de la página. Mantenga las instrucciones orientadas a tareas y adecuadas para el público. Tenga en cuenta estas preguntas:

* ¿Cuáles son las etiquetas que verán y cómo las elegirán? ¿Hay un texto de referencia que puedan consultar?
* ¿Qué debe hacer si ninguna etiqueta parece adecuada?
* ¿Qué debe hacer si varias etiquetas parecen adecuadas?
* ¿Qué umbral de confianza debe aplicarse a una etiqueta? ¿Quiere que use una conjetura si no está seguro?
* ¿Qué debe hacer con los objetos de interés ocluidos o superpuestos?
* ¿Qué debe hacer si un objeto de interés se corta con el borde de la imagen?
* ¿Qué deben hacer después de enviar una etiqueta si creen que han cometido un error?

En el caso de los rectángulos de selección, estas son algunas preguntas importantes:

* ¿Cómo se define el cuadro de límite para esta tarea? ¿Debe estar totalmente en el interior del objeto o en el exterior? ¿Debe recortarse lo más cerca posible o hay algo de margen?
* ¿Qué nivel de cuidado y coherencia espera que los etiquetadores apliquen para definir los rectángulos de selección?
* ¿Cómo se etiqueta el objeto que se muestra parcialmente en la imagen? 
* ¿Cómo se etiqueta el objeto parcialmente cubierto por otro objeto?

>[!NOTE]
> Recuerde que los etiquetadores podrán seleccionar las 9 primeras etiquetas usando las claves numéricas de 1 a 9.

## <a name="use-ml-assisted-labeling"></a>Uso del etiquetado con asistencia de ML

La página **Etiquetado con asistencia de ML** permite desencadenar modelos de Machine Learning automáticos para acelerar la tarea de etiquetado. Al principio del proyecto de etiquetado, las imágenes se presentan en orden aleatorio para reducir el posible sesgo. Sin embargo, los sesgos presentes en el conjunto de datos se reflejarán en el modelo entrenado. Por ejemplo, si el 80 % de las imágenes son de una sola clase, aproximadamente el 80 % de los datos usados para entrenar el modelo serán de esa clase. Este entrenamiento no incluye el aprendizaje activo.

Seleccione *Habilitar el etiquetado con asistencia de ML* y especifique una GPU para habilitar el etiquetado con asistencia, que consta de dos fases:
* Agrupación en clústeres
* Etiquetado previo

El número exacto de imágenes con etiqueta necesarias para iniciar el etiquetado con asistencia no es un número fijo.  Puede variar significativamente de un proyecto de etiquetado a otro. En algunos proyectos, a veces es posible ver tareas de etiquetado previo o de agrupación en clústeres después de que se hayan etiquetado manualmente 300 imágenes. El etiquetado con asistencia de ML usa una técnica llamada *transferencia de aprendizaje*, que usa un modelo entrenado previamente para iniciar el proceso de entrenamiento. Si las clases del conjunto de datos son similares a las del modelo entrenado previamente, puede haber etiquetas previas disponibles después de unos pocos cientos de imágenes etiquetadas manualmente. Si el conjunto de datos es significativamente diferente de los datos utilizados para entrenar previamente el modelo, puede tardar mucho más.

Como las etiquetas finales se siguen basando en la entrada del etiquetador, a veces esta tecnología se denomina etiquetado *con intervención humana*.

### <a name="clustering"></a>Agrupación en clústeres

Después de que haber enviado un determinado número de etiquetas, el modelo de Machine Learning empezará a agrupar imágenes similares.  Estas imágenes similares se presentan a los etiquetadores en la misma pantalla, para acelerar el etiquetado manual. La agrupación en clústeres es especialmente útil cuando el etiquetador está viendo una cuadrícula de 4, 6 o 9 imágenes. 

Una vez que se ha entrenado un modelo de Machine Learning con los datos etiquetados manualmente, el modelo se trunca a su última capa totalmente conectada. A continuación, las imágenes sin etiquetar pasan a través del modelo truncado en un proceso comúnmente conocido como "incrustación" o "caracterización". El proceso incrusta cada imagen en un espacio de alta dimensión definido por esta capa de modelo. Las imágenes que son vecinos más próximos en el espacio se usan para las tareas de agrupación en clústeres. 

La fase de agrupación en clústeres no aparece para los modelos de detección de objetos.

### <a name="prelabeling"></a>Etiquetado previo

Después de haber enviado un número suficiente de etiquetas de imagen, se usará un modelo de clasificación para predecir las etiquetas de imagen. O bien, se utilizará un modelo de detección de objetos para predecir los rectángulos de selección. Ahora el etiquetador ve las páginas que contienen etiquetas previstas ya presentes en cada imagen. También se muestran cuadros de predicción para la detección de objetos. La tarea siguiente consiste en revisar estas predicciones y corregir cualquier imagen que se haya etiquetado incorrectamente antes de enviar la página.  

Una vez que se ha entrenado un modelo de Machine Learning con los datos etiquetados manualmente, el modelo se evalúa en un conjunto de pruebas de imágenes etiquetadas manualmente para determinar su precisión en una variedad de distintos umbrales de confianza. Este proceso de evaluación se usa para determinar un umbral de confianza por encima del cual el modelo es lo suficientemente preciso como para mostrar las etiquetas previas. A continuación, el modelo se evalúa con datos sin etiquetar. Las imágenes con predicciones más confiables que este umbral se usan para la etiquetado previo.

## <a name="initialize-the-labeling-project"></a>Inicialización del proyecto de etiquetado

Una vez inicializado el proyecto de etiquetado, algunos aspectos del proyecto son inmutables. No se puede cambiar el tipo de tarea ni el conjunto de datos. Se *pueden* modificar las etiquetas y la dirección URL de la descripción de la tarea. Repase atentamente la configuración antes de crear el proyecto. Después de enviar el proyecto, volverá a la página principal **Data Labelling** (Etiquetado de datos), que muestra el proyecto como **Initializing** (Inicializando).

> [!NOTE]
> Puede que esta página no se actualice automáticamente. Espere unos momentos y actualice la página manualmente para ver el estado del proyecto como **Creado**.

## <a name="run-and-monitor-the-project"></a>Ejecutar y supervisar el proyecto
Después de inicializar el proyecto, Azure comenzará a ejecutarlo. Seleccione el proyecto en la página **Etiquetado de datos** para ver los detalles del proyecto.

Para pausar o reiniciar el proyecto, alterne el estado **En ejecución** en la parte superior derecha. Solo se pueden etiquetar datos cuando el proyecto se está ejecutando.

### <a name="dashboard"></a>Panel

En la pestaña **Panel** se muestra el progreso de la tarea de etiquetado.

:::image type="content" source="media/how-to-create-labeling-projects/labeling-dashboard.png" alt-text="Panel de etiquetado de datos":::

El gráfico de progreso muestra cuántos elementos se han etiquetado y cuántos no todavía.  Es posible que los elementos pendientes:

* Todavía no se hayan agregado a una tarea.
* Se hayan incluido en una tarea que está asignada a un etiquetador, pero que aún no se hayan completado. 
* Se encuentren en la cola de tareas que todavía se deben asignar.

En la sección central se muestra la cola de tareas que todavía se deben asignar. Cuando el etiquetado con asistencia de ML esté desactivado, en esta sección se mostrará el número de tareas manuales que se van a asignar. Cuando el etiquetado con asistencia de ML esté activado, también se mostrará:

* Tareas que contienen elementos agrupados en la cola
* Tareas que contienen elementos etiquetados previamente en la cola

Además, cuando el etiquetado con asistencia de ML está habilitado, se muestra una pequeña barra de progreso cuando se produce la siguiente ejecución de entrenamiento.  En la sección Experimentos se proporcionan vínculos para cada una de las ejecuciones de aprendizaje automático.

* Entrenamiento: Entrena un modelo para predecir las etiquetas.
* Validación: Determina si se usará la predicción de este modelo para etiquetar previamente los elementos. 
* Inferencia: Ejecución de predicción para nuevos elementos.
* Caracterización: Elementos de clúster (solo para proyectos de clasificación de imágenes).

En el lado derecho, se muestra una distribución de las etiquetas de las tareas que se han completado.  Recuerde que en algunos tipos de proyecto, un elemento puede tener varias etiquetas, en cuyo caso, el número total de etiquetas puede ser mayor que el número total de elementos.

### <a name="data-tab"></a>Pestaña Datos

En la pestaña **Datos**, puede ver el conjunto de datos y revisar los datos etiquetados. Si ve datos etiquetados incorrectamente, selecciónelos y elija **Rechazar**; se quitarán las etiquetas y los datos se volverán a colocar en la cola sin etiquetar.

### <a name="details-tab"></a>Pestaña Detalles

Vea los detalles del proyecto.  En esta pestaña, puede:

* Ver los detalles del proyecto y los conjuntos de datos de entrada
* Habilitar la actualización incremental
* Ver detalles del contenedor de almacenamiento usado para almacenar salidas etiquetadas en el proyecto
* Agregar etiquetas al proyecto
* Editar las instrucciones que proporcione a las etiquetas
* Editar los detalles del etiquetado con asistencia de ML, incluido habilitar o deshabilitar

### <a name="access-for-labelers"></a>Acceso para etiquetadores

Cualquier usuario que tenga acceso al área de trabajo puede etiquetar los datos del proyecto.  También puede personalizar los permisos de los etiquetadores para que puedan acceder al etiquetado, pero no a otras partes del área de trabajo o al proyecto de etiquetado.  Para obtener más información, consulte [Administración del acceso a un área de trabajo de Azure Machine Learning](how-to-assign-roles.md) y aprenda a crear el [rol personalizado de etiquetador](how-to-assign-roles.md#labeler).

## <a name="add-new-label-class-to-a-project"></a>Incorporación de una nueva clase de etiqueta a un proyecto

Durante el proceso de etiquetado es posible que se necesiten etiquetas adicionales para clasificar las imágenes.  Por ejemplo, puede que desee agregar una etiqueta "Desconocido" u "Otro" para indicar que las imágenes son confusas.

Siga estos pasos para agregar una o varias etiquetas a un proyecto:

1. Seleccione el proyecto en la página principal **Data Labeling** (Etiquetado de datos).
1. En la parte superior derecha de la página, cambie **En ejecución** a **En pausa** para detener la actividad de los etiquetadores.
1. Seleccione la pestaña **Details** (Detalles).
1. En la lista de la izquierda, seleccione **Label classes** (Clases de etiquetas).
1. En la parte superior de la lista, seleccione **+ Add Labels** (+ Agregar etiquetas) ![Add a label](media/how-to-create-labeling-projects/add-label.png) (Agregar una etiqueta).
1. En el formulario, agregue la nueva etiqueta y elija cómo continuar.  Como ha cambiado las etiquetas disponibles para una imagen, elija cómo tratar los datos ya etiquetados:
    * Vuelva a empezar y quite todas las etiquetas existentes.  Elija esta opción si desea empezar a etiquetar desde el principio con el nuevo conjunto de etiquetas completo. 
    * Vuelva a empezar y conserve todas las etiquetas existentes.  Elija esta opción para marcar todos los datos como sin etiquetar, pero mantener las etiquetas existentes como predeterminadas para las imágenes etiquetadas previamente.
    * Continúe y conserve todas las etiquetas existentes. Elija esta opción para mantener todos los datos ya etiquetados como están y empezar a usar la nueva etiqueta para los datos que aún no se hayan etiquetado.
1. Modifique la página de instrucciones según sea necesario para las nuevas etiquetas.
1. Una vez que haya agregado todas las etiquetas nuevas, en la parte superior derecha de la página, cambie **En pausa** a **En ejecución** para reiniciar el proyecto.  

## <a name="export-the-labels"></a>Exportar las etiquetas

En cualquier momento, puede exportar los datos de etiquetas para realizar experimentos de Machine Learning. Las etiquetas de imagen se pueden exportar en [formato COCO](http://cocodataset.org/#format-data) o como un conjunto de datos de Azure Machine Learning. Use el botón **Exportar** en la página **Detalles del proyecto** del proyecto de etiquetado.

El archivo COCO se crea en el almacén de blobs predeterminado del área de trabajo de Azure Machine Learning, en una carpeta dentro de *export/coco*. Puede acceder al conjunto de datos exportado de Azure Machine Learning en la sección **Conjuntos de datos** de Machine Learning. La página de detalles del conjunto de datos también proporciona código de ejemplo para acceder a las etiquetas desde Python.

![Conjunto de datos exportado](./media/how-to-create-labeling-projects/exported-dataset.png)

## <a name="next-steps"></a>Pasos siguientes

* [Tutorial: Creación de un proyecto de etiquetado para la clasificación de imágenes](tutorial-labeling.md).
* Etiquetado de imágenes para la [clasificación de imágenes o la detección de objetos](how-to-label-images.md)
* Obtenga más información sobre [Azure Machine Learning y Machine Learning Studio (clásico)](compare-azure-ml-to-studio-classic.md)
