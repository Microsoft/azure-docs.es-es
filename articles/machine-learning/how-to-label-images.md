---
title: Etiquetado de imágenes en un proyecto de etiquetado
title.suffix: Azure Machine Learning
description: Obtenga información sobre el uso de las herramientas de etiquetado de datos en un proyecto de etiquetado de Azure Machine Learning.
author: sdgilley
ms.author: sgilley
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.date: 07/27/2020
ms.openlocfilehash: e34fa4af08be898785acbc6f00aa735c1412ec47
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90897565"
---
# <a name="tag-images-in-a-labeling-project"></a>Etiquetado de imágenes en un proyecto de etiquetado 

Después de que el administrador del proyecto [cree un proyecto de etiquetado](https://docs.microsoft.com/azure/machine-learning/how-to-create-labeling-projects#create-a-labeling-project) en Azure Machine Learning, puede usar la herramienta de etiquetado (versión preliminar pública) para preparar rápidamente los datos para un proyecto de Machine Learning. En este artículo se describe:

> [!div class="checklist"]
> * Cómo acceder a proyectos de etiquetado
> * Herramientas de etiquetado
> * Cómo usar las herramientas para tareas específicas de etiquetado

## <a name="prerequisites"></a>Requisitos previos

* Una [cuenta Microsoft](https://account.microsoft.com/account) o una cuenta de Azure Active Directory para la organización y el proyecto
* Acceso de nivel de colaborador al área de trabajo que contiene el proyecto de etiquetado.

## <a name="sign-in-to-the-workspace"></a>Inicio de sesión en el área de trabajo

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com).

1. Seleccione la suscripción y el área de trabajo que contiene el proyecto de etiquetado.  Obtenga esta información del administrador del proyecto.

1. Seleccione **Data labeling** (Etiquetado de datos) en el lazo izquierdo para buscar el proyecto.  

## <a name="understand-the-labeling-task"></a>Descripción de la tarea de etiquetado

En la tabla de proyectos de etiquetado de datos, seleccione **Label link** (Vínculo de etiqueta) para el proyecto.

Verá instrucciones que son específicas del proyecto. Explican el tipo de datos a los que se enfrenta, cómo debe tomar sus decisiones y otra información pertinente. Tras leer esta información, seleccione **Tareas** en la parte superior de la página.  O bien, seleccione **Start labeling** (Iniciar etiquetado) en la parte inferior de la página.

## <a name="common-features-of-the-labeling-task"></a>Características comunes de la tarea de etiquetado

En todas las tareas de etiquetado de imágenes, elija una o varias etiquetas adecuadas del conjunto especificado por el administrador del proyecto. Puede seleccionar las nueve primeras etiquetas con las teclas numéricas del teclado.  

En las tareas de clasificación de imágenes, puede elegir ver varias imágenes simultáneamente. Use los iconos situados encima del área de imagen para seleccionar el diseño. 

Para seleccionar todas las imágenes mostradas al mismo tiempo, use **Select all** (Seleccionar todo). Para seleccionar imágenes individuales, use el botón de selección circular en la esquina superior derecha de la imagen. Debe seleccionar al menos una imagen para aplicar una etiqueta. Si selecciona varias imágenes, la etiqueta que seleccione se aplicará a todas las imágenes seleccionadas.

Aquí, hemos elegido un diseño de dos por dos y vamos a aplicar la etiqueta "Mammal" a las imágenes del oso y la orca. La imagen del tiburón ya se ha etiquetado como "Cartilaginous fish" y la iguana todavía no se ha etiquetado.

![Múltiples diseños de imagen y selección](./media/how-to-label-images/layouts.png)

> [!Important] 
> Cambie el diseño solo cuando tenga una página nueva de datos sin etiquetar. El cambio de diseño borra el trabajo de etiquetado en curso de la página.

Azure habilita el botón **Enviar** una vez etiquetadas todas las imágenes de la página. Seleccione **Submit** (Enviar) para guardar el trabajo.

Una vez que haya enviado etiquetas para los datos con los que está trabajando, Azure actualizará la página con un nuevo conjunto de imágenes de la cola de trabajo.

### <a name="assisted-machine-learning-preview"></a>Aprendizaje automático automatizado (versión preliminar) 

> [!IMPORTANT]
> El aprendizaje automático asistido se encuentra actualmente en versión preliminar pública.
> Se ofrece la versión preliminar sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Se pueden desencadenar algoritmos de aprendizaje automático. Si estos algoritmos se han habilitado en su proyecto, puede ver lo siguiente:

* Después de que se haya habilitado una cierta cantidad de imágenes, puede ver el mensaje **Tasks clustered** (Tareas agrupadas) en la parte superior de la pantalla al lado del nombre del proyecto,  lo cual significa que las imágenes se agrupan para mostrar las imágenes que sean similares en la misma página.  En ese caso, cambie a una de las distintas vistas de las imágenes para aprovechar la agrupación.  

* Más adelante puede ver **Tasks prelabeled** (Tareas preetiquetadas) junto al nombre del proyecto.  Luego aparecerán imágenes con una etiqueta sugerida que proviene de un modelo de clasificación de aprendizaje automático. Ninguno de los modelos de Machine Learning tiene una precisión del 100 %. Aunque solo usamos imágenes para las que el modelo sea seguro, existe la posibilidad de que estas etiquetas no estén preetiquetadas correctamente.  Cuando vea estas etiquetas, corrija las incorrectas antes de enviar la página.  

* En el caso de los modelos de detección de objetos, puede ver que los cuadros de límite y las etiquetas ya están presentes.  Corrija los que sean incorrectos antes de enviar la página.

En las primeras fases de un proyecto de etiquetado, es posible que el modelo de Machine Learning sea suficientemente preciso para preetiquetar un pequeño subconjunto de imágenes. Una vez que se etiqueten estas imágenes, el proyecto de etiquetado volverá al etiquetado manual para recopilar más datos para la siguiente ronda del entrenamiento del modelo. Con el paso del tiempo, el modelo pasará a ser más seguro en una mayor proporción de imágenes, lo cual dará como resultado posteriormente un mayor número de tareas preetiquetadas en el proyecto.

## <a name="tag-images-for-multi-class-classification"></a>Etiquetado de imágenes para la clasificación en varias clases

Si su proyecto es del tipo "multiclase de clasificación de imágenes", asignará una sola etiqueta a toda la imagen. Para revisar las instrucciones en cualquier momento, vaya a la página **Instructions** (Instrucciones) y seleccione **View detailed instructions** (Ver instrucciones detalladas).

Si ve que ha cometido un error después de asignar una etiqueta a una imagen, puede corregirla. Seleccione la cruz "**X**" en la etiqueta que se muestra debajo de la imagen para borrar la etiqueta. O bien, seleccione la imagen y elija otra clase. El valor recién seleccionado reemplazará a la etiqueta aplicada previamente.

## <a name="tag-images-for-multi-label-classification"></a>Etiquetado de imágenes para la clasificación en varias clases

Si está trabajando en un proyecto con clasificación con varias etiquetas, aplicará una *o más* etiquetas a una imagen. Para ver las instrucciones específicas del proyecto, seleccione **Instructions** (Instrucciones) y vaya a **View detailed instructions** (Ver instrucciones detalladas).

Seleccione la imagen que desea etiquetar y, a continuación, seleccione la etiqueta. La etiqueta se aplica a todas las imágenes seleccionadas y, a continuación, se anula la selección de las imágenes. Para aplicar más etiquetas, debe volver a seleccionar las imágenes. En la animación siguiente se muestra el etiquetado con varias etiquetas:

1. **Select all** (Seleccionar todo) se usa para aplicar la etiqueta "Ocean".
1. Se selecciona una sola imagen y se etiqueta como "Closeup".
1. Se seleccionan tres imágenes y se etiquetan como "Wide angle".

![La animación muestra el flujo multietiqueta](./media/how-to-label-images/multilabel.gif)

Para corregir un error, haga clic en "**X**" para borrar etiquetas individuales, o seleccione las imágenes y elija la etiqueta para borrar la etiqueta de todas las imágenes seleccionadas. Este escenario se muestra aquí. Al hacer clic en "Land", se borra la etiqueta de las dos imágenes seleccionadas.

![Una captura de pantalla muestra la anulación de varias selecciones](./media/how-to-label-images/multiple-deselection.png)

Azure solo habilitará el botón **Enviar** cuando haya aplicado al menos una etiqueta a cada imagen. Seleccione **Submit** (Enviar) para guardar el trabajo.


## <a name="tag-images-and-specify-bounding-boxes-for-object-detection"></a>Etiquetado de imágenes y especificación de rectángulos de selección para la detección de objetos

Si el proyecto es del tipo "Identificación del objeto (rectángulo de selección)", deberá especificar uno o varios rectángulos de selección en la imagen y aplicar una etiqueta a cada cuadro. Las imágenes pueden tener varios rectángulos de selección, cada uno con una sola etiqueta. Use **View detailed instructions** (Ver instrucciones detalladas) para determinar si se usan varios rectángulos de selección en el proyecto.

1. Seleccione una etiqueta para el rectángulo de selección que va a crear.
1. Seleccione la herramienta **Cuadro rectangular**![Herramienta Cuadro rectangular](./media/how-to-label-images/rectangular-box-tool.png) o presione "R".
3. Haga clic y arrastre diagonalmente en el destino para crear un rectángulo de selección aproximado. Para ajustar el rectángulo de selección, arrastre los bordes o las esquinas.

![Creación de un rectángulo delimitador](./media/how-to-label-images/bounding-box-sequence.png)

Para eliminar un rectángulo de selección, haga clic en la forma de X que aparece junto al rectángulo de selección una vez creado.

No se puede cambiar la etiqueta de un rectángulo de selección existente. Si asigna una etiqueta por error, tendrá que eliminar el rectángulo de selección y crear uno nuevo con la etiqueta correcta.

De forma predeterminada, puede editar los rectángulos de selección existentes. La herramienta **Bloquear/desbloquear regiones**![Bloquear/desbloquear regiones](./media/how-to-label-images/lock-bounding-boxes-tool.png) o "L" alterna ese comportamiento. Si las regiones están bloqueadas, solo puede cambiar la forma o la ubicación de un nuevo rectángulo de selección.

Use la herramienta **Manipulación de regiones**![Herramienta Manipulación de regiones](./media/how-to-label-images/regions-tool.png) o "M" para ajustar un cuadro de límite existente. Arrastre los bordes o esquinas para ajustar la forma. Haga clic en el interior para poder arrastrar todo el rectángulo de selección. Si no puede editar una región, es probable que haya activado la herramienta **Bloquear/desbloquear regiones**.

Use la herramienta **Cuadro basado en plantilla**![Herramienta Cuadro de plantilla](./media/how-to-label-images/template-box-tool.png) o "T" para crear varios cuadros de límite del mismo tamaño. Si la imagen no tiene rectángulos de selección y activa los recuadros basados en plantillas, la herramienta generará cuadros de 50 por 50 píxeles. Si crea un rectángulo de selección y, a continuación, activa los rectángulos basados en plantillas, los nuevos rectángulos de selección tendrán el tamaño del último rectángulo. Se puede cambiar el tamaño de los cuadros basados en plantillas después de la selección de ubicación. Al cambiar el tamaño de un rectángulo basado en una plantilla, solo se cambia el tamaño de ese rectángulo en concreto.

Para eliminar *todos* los rectángulos de selección de la imagen actual, seleccione la herramienta **Eliminar todas las regiones**![Herramienta Eliminar regiones](./media/how-to-label-images/delete-regions-tool.png).

Después de crear los rectángulos de selección de una imagen, seleccione **Submit** (Enviar) para guardar el trabajo; de lo contrario, no se guardará el trabajo en curso.

## <a name="finish-up"></a>Finalizar

Cuando se envía una página de datos etiquetados, Azure asigna datos nuevos sin etiquetar desde una cola de trabajo. Si no hay más datos sin etiquetar disponibles, verá un mensaje de aviso junto con un vínculo a la página principal del portal.

Cuando haya terminado de etiquetar, seleccione su nombre en la esquina superior derecha del portal de etiquetado y, a continuación, seleccione **sign-out** (Cerrar sesión). Si no lo hace, Azure acabará finalizando su "tiempo de espera" y asignará sus datos a otro etiquetador.

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [entrenar modelos de clasificación de imágenes en Azure](https://docs.microsoft.com/azure/machine-learning/tutorial-train-models-with-aml).

