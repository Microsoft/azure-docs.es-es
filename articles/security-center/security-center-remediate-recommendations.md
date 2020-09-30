---
title: Recomendaciones de corrección de Azure Security Center | Microsoft Docs
description: En este artículo se explican las recomendaciones de corrección de Azure Security Center para proteger los recursos y cumplir con las directivas de seguridad.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 8be947cc-cc86-421d-87a6-b1e23077fd50
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/08/2020
ms.author: memildin
ms.openlocfilehash: 4bad3227e08c0fbe0d280967e45bbef9d477e1b3
ms.sourcegitcommit: d0541eccc35549db6381fa762cd17bc8e72b3423
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89569142"
---
# <a name="remediate-recommendations-in-azure-security-center"></a>Recomendaciones de corrección en Azure Security Center

Las recomendaciones ofrecen sugerencias sobre cómo proteger mejor los recursos. Para implementar una recomendación, siga los pasos de corrección proporcionados en la recomendación.

## <a name="remediation-steps"></a>Pasos de corrección <a name="remediation-steps"></a>

Después de revisar todas las recomendaciones, decida cuál se corrige primero. Se recomienda usar el [impacto de la puntuación segura](security-center-recommendations.md#monitor-recommendations) para ayudarle a priorizar qué hacer primero.

1. En la lista, haga clic en la recomendación.

1. Siga las instrucciones de la sección **Pasos para la corrección**. Cada recomendación tiene su propio conjunto de instrucciones. En la siguiente captura de pantalla se muestran los pasos de corrección para configurar aplicaciones, a fin de permitir solo el tráfico a través de HTTPS.

    ![Detalles de recomendación](./media/security-center-remediate-recommendations/security-center-remediate-recommendation.png)

1. Una vez completada, aparece una notificación que le informa si la corrección se ha realizado correctamente.

## <a name="quick-fix-remediation"></a>Solución por corrección rápida<a name="one-click"></a>

La corrección rápida le permite solucionar rápidamente una recomendación en varios recursos. Solo está disponible para recomendaciones específicas. La corrección rápida simplifica el proceso y permite mejorar rápidamente la puntuación de seguridad e incrementar la seguridad de su entorno.

Para implementar la solución por corrección rápida:

1. En la lista de recomendaciones que tienen la etiqueta **Corrección rápida**, haga clic en la recomendación.

    [![Seleccione Corrección rápida](media/security-center-remediate-recommendations/security-center-one-click-fix-select.png)](media/security-center-remediate-recommendations/security-center-one-click-fix-select.png#lightbox)

1. En la pestaña **Recursos con estado incorrecto**, seleccione los recursos en los que desea implementar la recomendación y haga clic en **Corregir**.

    > [!NOTE]
    > Algunos de los recursos enumerados podrían estar deshabilitados porque no tiene los permisos adecuados para modificarlos.

1. En el cuadro de confirmación, lea los detalles y las implicaciones de la corrección.

    ![Corrección rápida](./media/security-center-remediate-recommendations/security-center-one-click-fix-view.png)

    > [!NOTE]
    > Las implicaciones se muestran en el cuadro gris de la ventana **Corregir recursos** que se abre después de hacer clic en **Corregir**. Estas enumeran los cambios que se producen al continuar con la corrección rápida.

1. Inserte los parámetros pertinentes si es necesario y apruebe la corrección.

    > [!NOTE]
    > Una vez completada la corrección, pueden pasar varios minutos hasta que los recursos se vean en la pestaña **Recursos con estado correcto**. Para ver las acciones de la corrección, consulte el [registro de actividad](#activity-log).

1. Una vez completada, aparece una notificación que le informa si la corrección se ha realizado correctamente.

## <a name="quick-fix-remediation-logging-in-the-activity-log"></a>Registro de la corrección rápida en el registro de actividad <a name="activity-log"></a>

La operación de corrección usa una implementación de plantilla o una llamada API REST PATCH para aplicar la configuración en el recurso. Estas operaciones se registran en el [registro de actividad de Azure](../azure-resource-manager/management/view-activity-logs.md).


## <a name="next-steps"></a>Pasos siguientes

En este documento, se mostró cómo aplicar las recomendaciones de corrección de Security Center. Para más información sobre Security Center, consulte los temas:

* [Establecimiento de directivas de seguridad en Azure Security Center](tutorial-security-policy.md): aprenda a configurar directivas de seguridad para las suscripciones y los grupos de recursos de Azure.
* [Supervisión del estado de seguridad en Azure Security Center](security-center-monitoring.md): obtenga información sobre cómo supervisar el mantenimiento de los recursos de Azure.
