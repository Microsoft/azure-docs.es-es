---
title: Realización de la evaluación sin conexión - Personalizer
titleSuffix: Azure Cognitive Services
description: En este artículo se muestra cómo usar la evaluación sin conexión para medir la eficacia de la aplicación y analizar el bucle de aprendizaje.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: how-to
ms.date: 02/20/2020
ms.openlocfilehash: a473085f9c94ca42a75d01b342d60cc33836b096
ms.sourcegitcommit: c293217e2d829b752771dab52b96529a5442a190
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/15/2020
ms.locfileid: "88244846"
---
# <a name="analyze-your-learning-loop-with-an-offline-evaluation"></a>Análisis del bucle de aprendizaje con una evaluación sin conexión

Aprenda a realizar una evaluación sin conexión y entender los resultados.

Las evaluaciones sin conexión le permiten medir la eficacia de Personalizer en comparación con el comportamiento predeterminado de la aplicación, conocer las características que más contribuyen a la personalización y descubrir automáticamente los valores nuevos de aprendizaje automático.

Consulte [Evaluaciones sin conexión](concepts-offline-evaluation.md) para más información.

## <a name="prerequisites"></a>Prerequisites

* Un bucle de Personalizer configurado.
* El bucle de Personalizer debe tener una cantidad de datos representativa: como valor aproximado, recomendamos al menos 50 000 eventos en sus registros para obtener resultados de evaluación significativos. Opcionalmente, también puede haber exportado previamente archivos de la  _directiva de aprendizaje_ que puede comparar y probar en la misma evaluación.

## <a name="run-an-offline-evaluation"></a>Ejecución de una evaluación sin conexión

1. En [Azure Portal](https://azure.microsoft.com/free/cognitive-services), busque el recurso de Personalizer.
1. En Azure Portal, vaya a la sección **Evaluaciones** y seleccione **Crear evaluación**.
    ![En Azure Portal, vaya a la sección **Evaluaciones** y seleccione **Crear evaluación**.](./media/offline-evaluation/create-new-offline-evaluation.png)
1. Configure los valores siguientes:

    * Un nombre de evaluación.
    * Fecha de inicio y finalización: son fechas que especifican el intervalo de datos que se va a utilizar en la evaluación. Estos datos deben estar presentes en los registros, como se especifica en el valor [Retención de datos](how-to-settings.md).
    * Detección de optimización establecida en **Sí**.

    > [!div class="mx-imgBorder"]
    > ![Elección de la configuración de evaluación sin conexión](./media/offline-evaluation/create-an-evaluation-form.png)

1. Seleccione **Aceptar** para iniciar la evaluación.

## <a name="review-the-evaluation-results"></a>Revisión de los resultados de la evaluación

Las evaluaciones pueden tardar mucho tiempo en ejecutarse, según la cantidad de datos que se vayan a procesar, el número de directivas de aprendizaje que se vayan a comparar y si se ha solicitado una optimización.

Una vez completada, puede seleccionar la evaluación en la lista de evaluaciones y, a continuación, seleccionar **Comparar la puntuación de la aplicación con otras configuraciones de aprendizaje potenciales**. Seleccione esta característica cuando desee ver cómo funciona la directiva de aprendizaje actual en comparación con una nueva directiva.

1. Revise el rendimiento de las [directivas de aprendizaje](concepts-offline-evaluation.md#discovering-the-optimized-learning-policy).

    > [!div class="mx-imgBorder"]
    > [![Revisión de los resultados de la evaluación](./media/offline-evaluation/evaluation-results.png)](./media/offline-evaluation/evaluation-results.png#lightbox)

1. Seleccione **Aplicar** para aplicar la directiva que mejore más el modelo según sus datos.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre [cómo funcionan las evaluaciones sin conexión](concepts-offline-evaluation.md).
