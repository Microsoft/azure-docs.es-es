---
title: 'ML Studio (clásico): conversión de un modelo a un servicio web (Azure)'
description: Información general de cómo progresa el modelo de Machine Learning Studio (clásico) de un experimento de desarrollo a un servicio web.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: conceptual
author: likebupt
ms.author: keli19
ms.custom: previous-ms.author=yahajiza, previous-author=YasinMSFT
ms.date: 03/20/2017
ms.openlocfilehash: 1b659335d09ed26b6556ccb7937094d90e1c5f1e
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87428422"
---
# <a name="how-a-machine-learning-studio-classic-model-progresses-from-an-experiment-to-a-web-service"></a>Progreso de un modelo de Machine Learning Studio (clásico) de un experimento a un servicio web

**SE APLICA A:**  ![yes](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico)   ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)


Azure Machine Learning Studio (clásico) ofrece un lienzo interactivo que permite desarrollar, ejecutar, probar e iterar un ***experimento*** que representa un modelo de análisis predictivo. Hay una gran variedad de módulos que le permiten realizar las acciones siguientes:

* Introducir datos en el experimento
* Manipulación de los datos.
* Entrenar un modelo con algoritmos de aprendizaje automático
* Puntuación del modelo
* Evaluar los resultados
* Generar los resultados finales

Cuando esté satisfecho con el experimento, puede implementarlo como un ***servicio web de Azure Machine Learning clásico*** o un ***nuevo servicio web de Azure Machine Learning*** para que los usuarios puedan enviar nuevos datos y recibir los resultados de vuelta.

En este artículo ofrecemos una información general de cómo progresa el modelo de Machine Learning desde un experimento de desarrollo hasta un servicio web aplicado.

> [!NOTE]
> Hay otras maneras de desarrollar e implementar modelos de Machine Learning, pero este artículo se centra en cómo usar Machine Learning Studio (clásico). Por ejemplo, para leer una descripción sobre cómo crear un servicio web predictivo clásico con R, consulte la publicación del blog [Build & Deploy Predictive Web Apps Using RStudio and Azure Machine Learning studio](https://docs.microsoft.com/archive/blogs/machinelearning/build-deploy-predictive-web-apps-using-rstudio-and-azure-ml) (Creación e implementación de aplicaciones web predictivas con RStudio y Azure Machine Learning Studio).
>
>

Aunque Azure Machine Learning Studio (clásico) está diseñado para ayudarle a desarrollar e implementar un *modelo de análisis predictivo*, Studio (clásico) se puede usar para desarrollar un experimento que no incluya ningún modelo de este tipo. Por ejemplo, un experimento podría simplemente insertar datos, manipularlos y, después, producir los resultados. Al igual que un experimento de análisis predictivo, puede implementar este experimento no predictivo como un servicio web, pero es un proceso más sencillo porque el experimento no entrena ni puntúa un modelo de aprendizaje automático. Aunque este no es el uso típico de Studio (clásico), se debe incluir en la explicación siguiente para que podamos darle una explicación completa de su funcionamiento.

## <a name="developing-and-deploying-a-predictive-web-service"></a>Desarrollo e implementación de un servicio web predictivo
Estas son las fases que sigue una solución típica cuando la desarrolla e implementa mediante Machine Learning Studio (clásico):

![Flujo de implementación](./media/model-progression-experiment-to-web-service/model-stages-from-experiment-to-web-service.png)

*Ilustración 1 - Fases de un modelo de análisis predictivo típico*

### <a name="the-training-experiment"></a>El experimento de entrenamiento
El ***experimento de entrenamiento*** es la fase inicial del proceso de desarrollo de un servicio web en Machine Learning Studio (clásico). El propósito del experimento de entrenamiento es brindarle un lugar para desarrollar, probar, iterar y finalmente entrenar un modelo de aprendizaje automático. Incluso puede entrenar varios modelos a la vez mientras busca la mejor solución, pero una vez que haya terminado de experimentar, puede seleccionar un único modelo entrenado y eliminar el resto del experimento. Para ver un ejemplo de desarrollo de un experimento de análisis predictivo, consulte [Desarrollo de una solución de análisis predictivo para la evaluación del riesgo de crédito en Azure Machine Learning Studio (clásico)](tutorial-part1-credit-risk.md).

### <a name="the-predictive-experiment"></a>El experimento predictivo
Cuando ya tenga un modelo entrenado en el experimento de entrenamiento, haga clic en **Set Up Web Service** (Configurar servicio web) y seleccione **Predictive Web Service** (Servicio web predictivo) en Machine Learning Studio (clásico) para iniciar el proceso de conversión del experimento de entrenamiento en un ***experimento predictivo***. El propósito del experimento predictivo es usar el modelo entrenado para puntuar nuevos datos, con el fin de ponerlo en marcha en algún momento como un servicio web de Azure.

Esta conversión se realiza a través de los pasos siguientes:

* Conversión del conjunto de módulos usados para el entrenamiento en un único módulo y su almacenamiento como un modelo entrenado
* Eliminación de los módulos extraños no relacionados con la puntuación
* Adición de puertos de entrada y salida que usará el servicio web final

Es posible que desee hacer más cambios para que el experimento predictivo esté listo para implementarse como un servicio web. Por ejemplo, si desea que el servicio web genere solo un subconjunto de resultados, puede agregar un módulo de filtrado antes del puerto de salida.

En este proceso de conversión no se descarta el experimento de entrenamiento. Cuando se complete el proceso, tendrá dos pestañas en Studio (clásico): una para el experimento de entrenamiento y otra para el experimento predictivo. De este modo, puede realizar cambios en el experimento de entrenamiento antes de implementar el servicio web y volver a compilar el experimento predictivo. O bien, puede guardar una copia del experimento de entrenamiento para iniciar otra línea de experimentación.

> [!NOTE]
> Al hacer clic en **Predictive Web Service** (Servicio web predictivo), se inicia un proceso automático para convertir el experimento de entrenamiento en un experimento predictivo, lo que funciona correctamente en la mayoría de los casos. Si el experimento de entrenamiento es complejo (por ejemplo, tiene varias rutas de acceso para el entrenamiento que se unen), es posible que prefiera realizar esta conversión manualmente. Para obtener más información, consulte [Preparación del modelo de implementación en Azure Machine Learning Studio (clásico)](convert-training-experiment-to-scoring-experiment.md).
>
>

### <a name="the-web-service"></a>El servicio web
Cuando esté plenamente convencido de que el experimento predictivo está listo, puede implementar el servicio como servicio web clásico o nuevo basado en Azure Resource Manager. Para aplicar el modelo implementándolo como *servicio web Machine Learning clásico*, haga clic en **Deploy Web Service** (Implementar servicio web) y seleccione **Deploy Web Service [Classic]** (Implementar servicio web [clásico]). Para implementarlo como *servicio web Machine Learning nuevo*, haga clic en **Deploy Web Service** (Implementar servicio web) y seleccione **Deploy Web Service [New]** (Implementar servicio web [nuevo]). Hecho esto, los usuarios ya pueden enviar datos al modelo mediante la API de REST del servicio web y recibir los resultados de vuelta. Para más información, consulte [Consumo de servicios web de Azure Machine Learning](consume-web-services.md).

## <a name="the-non-typical-case-creating-a-non-predictive-web-service"></a>El caso no típico: creación de un servicio web no predictivo
Si el experimento no entrena un modelo de análisis predictivo, no tendrá que crear un experimento de entrenamiento y un experimento de puntuación, sino solo un experimento que pueda implementar como un servicio web. Machine Learning Studio (clásico) detecta si el experimento contiene un modelo predictivo mediante el análisis de los módulos que se han usado.

Después de que haya recorrido en iteración el experimento y esté satisfecho con él:

1. Haga clic en **Set Up Web Service** (Configurar servicio web) y seleccione **Retraining Web Service** (Servicio web de reentrenamiento): los nodos de entrada y salida se agregan automáticamente.
2. Haga clic en **Ejecutar**
3. Haga clic en **Deploy Web Service** (Implementar servicio web) y seleccione **Deploy Web Service [Classic]** (Implementar servicio web [clásico]) o **Deploy Web Service [New]** (Implementar servicio web [nuevo]), en función del entorno en el que quiera realizar la implementación.

Hecho esto, el servicio web se habrá implementado, y puede acceder a él y a administrarlo como si fuera un servicio web predictivo.

## <a name="updating-your-web-service"></a>Actualización del servicio web
Una vez implementado el experimento como un servicio web, ¿qué ocurre si tiene que actualizarlo?

Depende de lo que necesite actualizar:

**Quiere cambiar la entrada o salida, o quiere modificar la forma en que el servicio web manipula los datos**

Si no modifica el modelo, pero solo quiere cambiar la forma en que el servicio web administra los datos, puede editar el experimento predictivo y después volver a hacer clic en **Deploy Web Service** (Implementar servicio web) y seleccionar **Deploy Web Service [Classic]** (Implementar servicio web [clásico]) o **Deploy Web Service [New]** (Implementar servicio web [nuevo]). Se detiene el servicio web, se implementa el experimento predictivo actualizado y se reinicia el servicio web.

Este es un ejemplo: Imagine que el experimento predictivo devuelve toda la fila de datos de entrada con el resultado de la predicción. Puede decidir que quiere que el servicio web solo devuelva el resultado. Por tanto agrega un módulo de **columnas del proyecto** en el experimento predictivo, justo antes del puerto de salida, para excluir otras columnas que no sean el resultado. Al hacer clic en **Deploy Web Service** (Implementar servicio web) y volver a seleccionar **Deploy Web Service [Classic]** (Implementar servicio web [clásico]) o **Deploy Web Service [New]** (Implementar servicio web [nuevo]), el servicio web se actualiza.

**Quiere reciclar el modelo con nuevos datos**

Si quiere mantener su modelo de aprendizaje automático, pero le gustaría reciclarlo con nuevos datos, tiene dos opciones:

1. **Reentrenar el modelo mientras se ejecuta el servicio web**: si desea reentrenar el modelo mientras se ejecuta el servicio web predictivo, puede hacerlo realizando un par de modificaciones en el experimento de entrenamiento para convertirlo en un ***experimento de reentrenamiento*** y después puede implementarlo como un ***servicio web* de reentrenamiento**. Para obtener más información sobre cómo hacerlo, consulte [Reciclar modelos de Machine Learning mediante programación](/azure/machine-learning/studio/retrain-machine-learning-model).
2. **Volver al experimento de entrenamiento original y usar distintos datos de entrenamiento para desarrollar el modelo**: el experimento predictivo está vinculado al servicio web, pero el experimento de entrenamiento no está vinculado directamente de esta manera. Si modifica el experimento de entrenamiento original y hace clic en **Set Up Web Service** (Configurar servicio web), se creará un *nuevo* experimento predictivo que, cuando se implemente, creará un *nuevo* servicio web. Esta acción no solo actualiza el servicio web original.

   Si tiene que modificar el experimento de entrenamiento, ábralo y haga clic en **Guardar como** para realizar una copia. De este modo, el experimento de entrenamiento original, el experimento predictivo y el servicio web quedan intactos. Ahora puede crear un nuevo servicio web con los cambios. Después de implementar el nuevo servicio web, puede decidir si quiere detener el servicio web anterior o dejar que siga ejecutándose junto con el nuevo.

**Quiere entrenar un modelo diferente**

Si desea realizar cambios en el experimento predictivo original, como seleccionar un algoritmo de Aprendizaje automático diferente o probar un método de entrenamiento distinto, debe seguir el segundo procedimiento descrito anteriormente para reciclar el modelo: abra el experimento de entrenamiento, haga clic en **Guardar como** para hacer una copia e inicie la nueva ruta de acceso del desarrollo del modelo, de la creación del experimento predictivo y de la implementación del servicio web. Esto creará un nuevo servicio web no relacionado con el original (puede decidir que se siga ejecutando solo uno de ellos o ambos).

## <a name="next-steps"></a>Pasos siguientes
Para obtener más detalles sobre el proceso de desarrollo y el experimento, consulte los artículos siguientes:

* Conversión del experimento: [Preparación del modelo de implementación en Azure Machine Learning Studio (clásico)](convert-training-experiment-to-scoring-experiment.md)
* implementación del servicio web: [Implementación de un servicio web Azure Machine Learning](deploy-a-machine-learning-web-service.md)
* reciclaje del modelo - [Reciclar modelos de Machine Learning mediante programación](/azure/machine-learning/studio/retrain-machine-learning-model)

Para ver ejemplos de todo el proceso, consulte:

* [Tutorial sobre Machine Learning: Creación del primer experimento en Azure Machine Learning Studio (clásico)](create-experiment.md)
* [Tutorial: Desarrollar una solución de análisis predictivo para la evaluación del riesgo de crédito en Azure Machine Learning Studio](tutorial-part1-credit-risk.md)

