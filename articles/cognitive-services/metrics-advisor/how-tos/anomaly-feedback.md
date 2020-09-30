---
title: Envío de comentarios sobre anomalías al servicio Metrics Advisor
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo enviar comentarios sobre las anomalías que encuentre la instancia de Metrics Advisor y ajustar los resultados.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 09/10/2020
ms.author: aahi
ms.openlocfilehash: 50d422edf1a4b45132d0b86eac9d4947cef5e5bf
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932023"
---
# <a name="adjust-anomaly-detection-using-feedback"></a>Ajuste de la detección de anomalías mediante comentarios

Si no queda conforme con algunos de los resultados de la detección de anomalías proporcionados por Metrics Monitor, puede agregar manualmente comentarios que afecten al modelo aplicado a los datos. 

Use los botones de la esquina superior derecha de la página para activar el modo de anotación de comentarios.

:::image type="content" source="../media/feedback/annotation-mode.png" alt-text="Modo de anotación de comentarios":::

Después de activar el modo de anotación de comentarios, puede enviar comentarios sobre un punto o varios puntos continuos.

## <a name="give-feedback-for-one-point"></a>Envío de comentarios sobre un punto 

Con el modo de anotación de comentarios activado, haga clic en un punto para abrir un **panel para agregar comentarios**. Puede establecer el tipo de comentarios que desea aplicar. Estos comentarios se incorporarán en la detección de puntos futuros.  

* Seleccione **Anomaly** (Anomalía) si cree que Metrics Monitor etiquetó incorrectamente el punto. Puede especificar si un punto debe o no ser una anomalía. 
* Seleccione **ChangePoint** si cree que el punto indica el inicio de un cambio de tendencia.
* Seleccione **Period** (Período) para indicar estacionalidad. Metrics Monitor puede detectar automáticamente intervalos de estacionalidad, o bien puede especificarlos manualmente. 

Plantéese dejar un comentario en el cuadro de texto **Comment** (Comentario) y haga clic en **Save** (Guardar) para guardar los comentarios.

:::image type="content" source="../media/feedback/feedback-menu.png" alt-text="Menú de comentarios":::

## <a name="give-feedback-for-multiple-continuous-points"></a>Envío de comentarios sobre varios puntos continuos

Puede enviar comentarios sobre varios puntos continuos a la vez haciendo clic en abajo y arrastrando el mouse sobre los puntos que desea anotar. Verá el mismo menú de comentarios que antes. Los mismos comentarios se aplicarán a todos los puntos elegidos después de hacer clic en **Save** (Guardar).

:::image type="content" source="../media/feedback/continuous-points.png" alt-text="Elección de varios puntos":::

## <a name="how-to-view-my-feedback"></a>Cómo ver comentarios

Para ver si ha cambiado la detección de anomalías de un punto, mantenga el mouse sobre el punto. La información sobre herramientas mostrará **Affected by feedback: true** (Afectado por comentarios: verdadero) si se cambió la detección. Si se muestra **False** (Falso), significa que se realizó el cálculo de comentarios en el punto, pero no se cambió el resultado de la detección de anomalías.

:::image type="content" source="../media/feedback/affected-point.png" alt-text="Punto afectado por comentarios":::

## <a name="when-should-i-annotate-an-anomaly-as-normal"></a>¿Cuándo debo anotar una anomalía como "normal"?

Hay muchas razones por las que podría considerar que una anomalía es una falsa alarma. Plantéese usar las siguientes características de Metrics Advisor si se da uno de los siguientes escenarios:


|Escenario  |Recomendación |
|---------|---------|
|La anomalía se debe a un cambio en el origen de datos conocido; por ejemplo, un cambio del sistema.     | No anote esta anomalía si no se espera que este escenario se produzca con regularidad.        |
|La anomalía está causada por festividades.     | Use [Preset events](configure-metrics.md#preset-events) (Eventos preestablecidos) para marcar la detección de anomalías en las horas especificadas.       |
|Hay un patrón normal para detectar anomalías (por ejemplo, los fines de semana) y no deben ser anomalías.      |Use la característica de comentarios o los eventos preestablecidos.        |

## <a name="next-steps"></a>Pasos siguientes
- [Diagnóstico de incidentes](diagnose-incident.md)
- [Configuración de métricas y ajuste de la configuración de la detección](configure-metrics.md)
- [Configuración de alertas y obtención de notificaciones mediante enlaces](../how-tos/alerts.md)