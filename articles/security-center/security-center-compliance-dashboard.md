---
title: 'Tutorial: Comprobaciones de cumplimiento normativo: Azure Security Center'
description: 'Tutorial: Aprenda a mejorar el cumplimiento de reglamentaciones mediante Azure Security Center.'
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 5f50c4dc-ea42-418d-9ea8-158ffeb93706
ms.service: security-center
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/12/2019
ms.author: memildin
ms.openlocfilehash: 7f235f9255df49c096e224e1754ad2bbdf664423
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90895822"
---
# <a name="tutorial-improve-your-regulatory-compliance"></a>Tutorial: Mejora del cumplimiento normativo
---

Azure Security Center resulta de gran ayuda para simplificar el proceso necesario para cumplir los requisitos de cumplimiento normativo, para lo que se usa el **panel de cumplimiento normativo.** . En el panel, Security Center proporciona información detallada acerca de su estado de cumplimiento de la norma, gracias a la valoración continua de su entorno de Azure. Security Center analiza los factores de riesgo en su entorno de nube híbrida según los procedimientos recomendados de seguridad. Estas valoraciones se asignan a los controles de cumplimiento desde un conjunto de estándares compatible. En el panel de cumplimiento normativo, se puede ver el estado de todas estas valoraciones dentro de su entorno en el contexto de un estándar o reglamento concreto. Al actuar sobre las recomendaciones y reducir los factores de riesgo en su entorno, su estado de cumplimiento normativo mejora.

En este tutorial, aprenderá a:

-   Evaluar su cumplimiento de reglamentaciones mediante el panel de cumplimiento de reglamentaciones

-   Mejorar su estado de cumplimiento normativo mediante la realización de acciones sobre las recomendaciones

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para recorrer las características descritas en este tutorial, [Azure Defender](azure-defender.md) debe estar habilitado. Puede probar Azure Defender gratis durante 30 días.

##  <a name="assess-your-regulatory-compliance"></a>Mejora del cumplimiento de reglamentaciones

Security Center evalúa constantemente la configuración de los recursos para identificar problemas de seguridad y vulnerabilidades. Estas valoraciones se presentan como recomendaciones, que se centran en mejorar su higiene de seguridad. En el panel de cumplimiento de reglamentaciones, puede ver un conjunto de estándares de cumplimiento con todos sus requisitos en el que los requisitos compatibles se asignan a las valoraciones de seguridad aplicables. Esto le permite ver su estado de cumplimiento de la norma, en función del estado de estas valoraciones.

La vista del panel de cumplimiento de reglamentaciones puede ayudarle a centrar la atención en los espacios que hay en el cumplimiento de un estándar o reglamentación que es importante para usted. También le permite supervisar continuamente su puntuación de cumplimiento a lo largo del tiempo en entornos híbrido y de nube dinámicos.

>[!NOTE]
> De forma predeterminada, Security Center admite los siguientes estándares: Azure CIS, PCI DSS 3.2, ISO 27001 y SOC TSP. 
>
> La característica de [paquetes de cumplimiento dinámicos (versión preliminar)](update-regulatory-compliance-packages.md) permite actualizar los estándares que se muestran en el panel de cumplimiento normativo a los nuevos paquetes *dinámicos*. También puede usar la misma característica en vista previa para agregar nuevos paquetes de cumplimiento y supervisar el cumplimiento de estándares adicionales. 

1.  En el menú principal de Security Center, en **POLICY & COMPLIANCE** (DIRECTIVA Y CUMPLIMIENTO), seleccione **Regulatory compliance** (Cumplimiento de reglamentaciones). <br>
En la parte superior de la pantalla se ve un panel con información general acerca del estado de cumplimiento con el conjunto de reglamentaciones de cumplimiento admitidas. Puede ver la puntuación global de cumplimiento y el número de valoraciones aprobadas y suspendidas asociadas a cada estándar.

    ![confianza alta en descripción de equipo](./media/security-center-compliance-dashboard/compliance-dashboard.png)

2.  Seleccione la pestaña del estándar de cumplimiento que le interese. Verá la lista de todos los controles de dicho estándar. Para sabe qué controles se pueden aplicar, puede ver los detalles de las valoraciones aprobadas y suspendidas asociadas a dicho control. Algunos controles aparecen atenuados, no tienen valoraciones de Security Center asociadas. Compruebe los requisitos de estos y valórelos en su entorno. Es posible que algunos de ellos estén relacionados con los procesos y no sean técnicos.

    ![Pestaña de cumplimiento](./media/security-center-compliance-dashboard/compliance-pci.png)

1. Para generar y descargar un informe en PDF que resuma su estado actual de cumplimiento de un estándar concreto, haga clic en **Download report** (Descargar informe).

    El informe proporciona un resumen de alto nivel del estado de cumplimiento del estándar seleccionado, para lo que usa los datos de las evaluaciones de Security Center, y se organiza según los controles de dicho estándar concreto. El informe se puede compartir con las partes interesadas competentes y puede servir para proporcionar evidencia a los auditores internos y externos.

    ![descarga](./media/security-center-compliance-dashboard/download-report.png)

## <a name="improve-your-compliance-posture"></a>Mejora de su estado de cumplimiento de la norma

Dada la información del panel de cumplimiento de reglamentaciones, puede mejorar su estado de cumplimiento normativo mediante la resolución de recomendaciones del panel.

1.  Haga clic en cualquiera de las valoraciones suspensas que aparecen en el panel para ver los detalles de dicha recomendación. Cada recomendación incluye un conjunto de pasos de corrección que se deben seguir si se desea resolver el problema.

1.  Puede seleccionar un recurso concreto para ver más detalles y resolver la recomendación del mismo. <br>Por ejemplo, en la pestaña **Azure CIS standard** (Estándar Azure CIS), puede hacer clic en la recomendación **Require secure transfer to storage account** (Requerir transferencia segura a la cuenta de almacenamiento).

    ![Selección de la recomendación **Requerir transferencia segura a la cuenta de almacenamiento**](./media/security-center-compliance-dashboard/compliance-recommendation.png)

1. Al hacer clic en la información de la recomendación y seleccionar un recurso incorrecto, le lleva directamente a la habilitación de la **transferencia segura de almacenamiento** en Azure Portal.

    Para más información sobre cómo aplicar las recomendaciones, consulte [Implementación de recomendaciones de seguridad en Azure Security Center](security-center-recommendations.md).

    ![Configuración de la cuenta de almacenamiento](./media/security-center-compliance-dashboard/compliance-remediate-recommendation.png)

1.  Después de realizar las acciones necesarias para resolver las recomendaciones, verá su repercusión en informe del panel de cumplimiento, ya que la puntuación de cumplimiento mejora.

    > [!NOTE]
    > Las valoraciones se ejecutan aproximadamente cada 12 horas, por lo que este es el tiempo que tardará en verse el impacto en los datos de cumplimiento.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a utilizar el panel de cumplimiento de reglamentaciones de Security Center para:

-   Ver y supervisar su estado de cumplimiento de la norma, en relación con los estándares y regulaciones que le resultan importantes.

-   Mejorar el estado de cumplimiento mediante la resolución de las recomendaciones pertinentes y la observación de la mejora en la puntuación del cumplimiento.

El panel de cumplimiento de reglamentaciones puede simplificar considerablemente el proceso de cumplimiento y reducir notablemente el tiempo necesario para la recopilación de evidencias del cumplimiento en su entorno de Azure e híbrido.

Para más información, consulte:

-   [Actualización de los paquetes de cumplimiento dinámicos en el panel de cumplimiento normativo (vista previa)](update-regulatory-compliance-packages.md): obtenga información sobre esta característica en vista previa que le permite actualizar los estándares que se muestran en el panel de cumplimiento normativo a los nuevos paquetes *dinámicos*. También puede usar la misma característica en vista previa para agregar nuevos paquetes de cumplimiento y supervisar el cumplimiento de estándares adicionales. 

-   [Supervisión del estado de seguridad en Azure Security Center](security-center-monitoring.md): obtenga información sobre cómo supervisar el mantenimiento de los recursos de Azure.

-   [Administración de recomendaciones de seguridad en Azure Security Center](security-center-recommendations.md): aprenda a usar las recomendaciones de Azure Security Center como ayuda para la protección de los recursos de Azure.

-   [Mejora de la puntuación de seguridad de Azure Security Center](secure-score-security-controls.md): aprenda a dar prioridad a los puntos vulnerables y las recomendaciones de seguridad para mejorar al máximo su postura ante la seguridad.