---
title: 'Uso de del aprendizaje activo con la base de conocimiento: QnA Maker'
description: Mejore la calidad de su base de conocimientos con el aprendizaje activo. Revise, acepte o rechace, o agregue sin quitar ni cambiar las preguntas existentes.
ms.topic: conceptual
ms.date: 03/18/2020
ms.openlocfilehash: 004ee9d4cced0116831718b1601d5d510dea5a4d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87090933"
---
# <a name="use-active-learning-to-improve-your-knowledge-base"></a>Uso del aprendizaje activo para mejorar la base de conocimiento

El [aprendizaje activo](../Concepts/active-learning-suggestions.md) le permite mejorar la calidad de la base de conocimiento, ya que sugiere preguntas alternativas. Los envíos de usuarios se tienen en cuenta y se muestran como sugerencias en la lista de preguntas alternativas. Tiene la flexibilidad de agregar esas sugerencias como preguntas alternativas o rechazarlas.

La base de conocimiento no cambia automáticamente. Debe aceptar las sugerencias para que cualquier cambio surta efecto. Estas sugerencias agregan preguntas, pero no cambian ni quitan preguntas existentes.


## <a name="upgrade-runtime-version-to-use-active-learning"></a>Actualización de la versión en tiempo de ejecución para usar el aprendizaje activo

El aprendizaje activo se admite en el tiempo de ejecución versión 4.4.0 y superior. Si la base de conocimiento se creó en una versión anterior, [actualice el tiempo de ejecución](set-up-qnamaker-service-azure.md#get-the-latest-runtime-updates) para usar esta característica.

## <a name="turn-on-active-learning-for-alternate-questions"></a>Activación del aprendizaje activo para preguntas alternativas

El aprendizaje activo está desactivado de forma predeterminada. Actívelo para ver preguntas sugeridas. Una vez activado el aprendizaje activo, deberá enviar información desde la aplicación cliente a QnA Maker. Para obtener más información, consulte [Flujo arquitectónico para usar GenerateAnswer y Train API desde un bot](improve-knowledge-base.md#architectural-flow-for-using-generateanswer-and-train-apis-from-a-bot).

1. Seleccione **Publicar** para publicar la base de conocimiento. Las consultas de aprendizaje activo se recopilan solo desde el punto de conexión de predicción de GenerateAnswer API. Las consultas en el panel de prueba del portal de QnA Maker no afectarán al aprendizaje activo.

1. Para activar el aprendizaje activo en el portal de QnA Maker, vaya a la esquina superior derecha, seleccione su **Nombre** y vaya a [**Configuración del servicio**](https://www.qnamaker.ai/UserSettings).

    ![Active las preguntas sugeridas alternativas de aprendizaje activo desde la página Configuración del servicio. Seleccione su nombre de usuario en el menú superior derecho y seleccione Continuación del servicio.](../media/improve-knowledge-base/Endpoint-Keys.png)


1. Busque el servicio QnA Maker, y active **Aprendizaje activo**.

    > [!div class="mx-imgBorder"]
    > [![En la página Configuración del servicio, active la característica Aprendizaje activo. Si no es capaz de activar o desactivar la característica, deberá actualizar el servicio.](../media/improve-knowledge-base/turn-active-learning-on-at-service-setting.png)](../media/improve-knowledge-base/turn-active-learning-on-at-service-setting.png#lightbox)

    > [!Note]
    > La versión exacta de la imagen anterior se muestra solo como un ejemplo. Su versión puede ser diferente.

    Una vez que **Aprendizaje activo** está habilitado, la base de conocimiento sugiere nuevas preguntas a intervalos regulares según las preguntas enviadas por el usuario. Para deshabilitar **Aprendizaje activo**, vuelva a cambiar la configuración.

## <a name="review-suggested-alternate-questions"></a>Revisión de preguntas alternativas sugeridas

[Revise las preguntas sugeridas alternativas](improve-knowledge-base.md) en la página **Editar** de cada base de conocimiento.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una base de conocimientos](./manage-knowledge-bases.md)