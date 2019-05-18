---
title: Mejora del cumplimiento de reglamentaciones mediante Azure Security Center | Microsoft Docs
description: 'Tutorial: Aprenda a mejorar el cumplimiento de reglamentaciones mediante Azure Security Center.'
services: security-center
documentationcenter: na
author: monhaber
manager: barbkess
editor: ''
ms.assetid: 5f50c4dc-ea42-418d-9ea8-158ffeb93706
ms.service: security-center
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 4/30/2019
ms.author: monhaber
ms.openlocfilehash: 111a32983472d18c1435feaf2456e5b9416b5112
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65442321"
---
# <a name="tutorial-improve-your-regulatory-compliance"></a>Tutorial: Mejora del cumplimiento normativo
---

Azure Security Center resulta de gran ayuda para simplificar el proceso necesario para cumplir los requisitos de cumplimiento de reglamentaciones, para lo que se usa el panel de cumplimiento de reglamentaciones. En el panel, Security Center proporciona información detallada acerca de su estado de cumplimiento de la norma, gracias a la valoración continua de su entorno de Azure. Las valoraciones que realiza Security Center analizan los factores de riesgo que hay en su entorno de nube híbrida con arreglo a los procedimientos recomendados de seguridad. Estas valoraciones se asignan a los controles de cumplimiento desde un conjunto de estándares compatible. En el panel de cumplimiento de reglamentaciones, se puede ver con claridad el estado de todas estas valoraciones dentro de su entorno en el contexto de un estándar o reglamento concreto. Al actuar sobre las recomendaciones y reducir los factores de riesgo en su entorno, puede ver que su estado de cumplimiento normativo mejora.

En este tutorial, aprenderá a:

-   Evaluar su cumplimiento de reglamentaciones mediante el panel de cumplimiento de reglamentaciones

-   Mejorar su estado de cumplimiento normativo mediante la realización de acciones sobre las recomendaciones

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para recorrer todas las características que se tratan en este tutorial debe tener el plan de tarifa Estándar de Security Center. Dicho plan se puede probar de forma gratuita.
Para más información, consulte la [página de precios](https://azure.microsoft.com/pricing/details/security-center/). En [Guía de inicio rápido de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-get-started) le explicamos cómo realizar la actualización al plan de tarifa Estándar.

##  <a name="assess-your-regulatory-compliance"></a>Mejora del cumplimiento de reglamentaciones

Security Center evalúa constantemente la configuración de los recursos para identificar problemas de seguridad y vulnerabilidades. Estas valoraciones se presentan como recomendaciones, que se centran en mejorar su higiene de seguridad. En el panel de cumplimiento de reglamentaciones, puede ver un conjunto de estándares de cumplimiento con todos sus requisitos en el que los requisitos compatibles se asignan a las valoraciones de seguridad aplicables. Esto le permite ver su estado de cumplimiento de la norma, en función del estado de estas valoraciones.

La vista del panel de cumplimiento de reglamentaciones puede ayudarle a centrar la atención en los espacios que hay en el cumplimiento de un estándar o reglamentación que es importante para usted. También le permite supervisar continuamente su puntuación de cumplimiento a lo largo del tiempo en entornos híbrido y de nube dinámicos.

>[!NOTE]
> Los estándares de regulaciones que se admiten en la actualidad son: Azure CIS, PCI DSS 3.2, ISO 27001 y SOC TSP. Los estándares adicionales se reflejarán en el panel conforme se desarrolla.
1.  En el menú principal de Security Center, en **POLICY & COMPLIANCE** (DIRECTIVA Y CUMPLIMIENTO), seleccione **Regulatory compliance** (Cumplimiento de reglamentaciones). <br>
En la parte superior de la pantalla se ve un panel con información general acerca del estado de cumplimiento con el conjunto de reglamentaciones de cumplimiento admitidas. Puede ver la puntuación global de cumplimiento y el número de valoraciones aprobadas y suspendidas asociadas a cada estándar.

    ![confianza alta en descripción de equipo](./media/security-center-compliance-dashboard/compliance-dashboard.png)


2.  Seleccione la pestaña del estándar de cumplimiento que le interese. Verá la lista de todos los controles de dicho estándar. Para sabe qué controles se pueden aplicar, puede ver los detalles de las valoraciones aprobadas y suspendidas asociadas a dicho control. Algunos controles aparecen atenuados, ya que no tienen valoraciones de Security Center asociadas. Por consiguiente, es el propio usuario quien debe analizar los requisitos de estos y valorarlos en su entorno. Es posible que algunos de ellos estén relacionados con los procesos y no sean técnicos.

    ![Pestaña de cumplimiento](./media/security-center-compliance-dashboard/compliance-pci.png)

3. Seleccione la pestaña **All** (Todas) para ver una vista de todas las recomendaciones de Security Center pertinentes y sus estándares asociados. Esta vista puede ser útil para identificar los diferentes estándares a los que afecta una recomendación determinada <br> y se puede usar para dar prioridad a las recomendaciones que necesita resolver. Por ejemplo, si ve que la recomendación **Enable MFA for accounts with owner permissions on your subscription** (Habilitar MFA en las cuentas con permisos de propietario de su suscripción) no funciona en varios recursos y está asociado a varios estándares, la resolución de esta recomendación tendrá un impacto elevado en la puntuación total de cumplimiento.

    ![impacto de la puntuación de cumplimiento](./media/security-center-compliance-dashboard/compliance-all-tabs.png)

1. Para generar y descargar un informe en PDF que resuma su estado actual de cumplimiento de un estándar concreto, haga clic en **Download report** (Descargar informe).

    El informe proporciona un resumen de alto nivel del estado de cumplimiento del estándar seleccionado, para lo que usa los datos de las evaluaciones de Security Center, y se organiza según los controles de dicho estándar concreto. El informe se puede compartir con las partes interesadas competentes y puede servir para proporcionar evidencia a los auditores internos y externos.

    ![descargar](./media/security-center-compliance-dashboard/download-report.png)

## <a name="improve-your-compliance-posture"></a>Mejora de su estado de cumplimiento de la norma

Dada la información del panel de cumplimiento de reglamentaciones, puede mejorar su estado de cumplimiento normativo mediante la resolución de recomendaciones del panel.

1.  Haga clic en cualquiera de las valoraciones suspensas que aparecen en el panel para ver los detalles de dicha recomendación. Cada recomendación incluye un conjunto de pasos de corrección que se deben seguir si se desea resolver el problema.

2.  Puede seleccionar un recurso concreto para ver más detalles y resolver la recomendación del mismo. <br>Por ejemplo, en la pestaña **Azure CIS standard** (Estándar Azure CIS), puede hacer clic en la recomendación **Require secure transfer to storage account** (Requerir transferencia segura a la cuenta de almacenamiento).

    ![recomendación de cumplimiento](./media/security-center-compliance-dashboard/compliance-recommendation.png)

3. Al hacer clic en la información de la recomendación y seleccionar un recurso incorrecto, le lleva directamente a la habilitación de la **transferencia segura de almacenamiento** en Azure Portal.<br>Para más información sobre cómo aplicar las recomendaciones, consulte [Implementación de recomendaciones de seguridad en Azure Security Center](security-center-recommendations.md).

    ![recomendación de cumplimiento](./media/security-center-compliance-dashboard/compliance-remediate-recommendation.png)

4.  Después de realizar las acciones necesarias para resolver las recomendaciones, verá su repercusión en informe del panel de cumplimiento, ya que la puntuación de cumplimiento mejora.

    > [!NOTE]
    > Las valoraciones se ejecutan aproximadamente cada 12 horas, por lo que este es el tiempo que tardará en verse el impacto en los datos de cumplimiento.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a utilizar el panel de cumplimiento de reglamentaciones de Security Center para:

-   Ver y supervisar su estado de cumplimiento de la norma, en relación con los estándares y regulaciones que le resultan importantes.

-   Mejorar el estado de cumplimiento mediante la resolución de las recomendaciones pertinentes y la observación de la mejora en la puntuación del cumplimiento.

El panel de cumplimiento de reglamentaciones puede simplificar considerablemente el proceso de cumplimiento y reducir notablemente el tiempo necesario para la recopilación de evidencias del cumplimiento en su entorno de Azure e híbrido.

Para más información sobre Security Center, consulte:

-   [Supervisión del estado de seguridad en Azure Security Center](security-center-monitoring.md): aprenda a supervisar el estado de los recursos de Azure.

-   [Administración de recomendaciones de seguridad en Azure Security Center](security-center-recommendations.md): aprenda a usar las recomendaciones de Azure Security Center como ayuda para la protección de los recursos de Azure.

-   [Mejora de la puntuación de seguridad en Azure Security Center](security-center-secure-score.md): aprenda a dar prioridad a las vulnerabilidades y recomendaciones de seguridad para mejorar su posición ante la seguridad.

-   [Preguntas más frecuentes sobre Azure Security Center](security-center-faq.md): encuentre las preguntas más frecuentes sobre el uso del servicio.
