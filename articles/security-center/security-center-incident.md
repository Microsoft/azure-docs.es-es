---
title: Administración de incidentes de seguridad en Azure Security Center | Microsoft Docs
description: Este documento le ayuda a usar Azure Security Center para administrar incidentes de seguridad.
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: conceptual
ms.date: 06/15/2020
ms.author: memildin
ms.openlocfilehash: a6ddfa1f6db06c2ca06b94efec7f890c2a714b2a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90883864"
---
# <a name="manage-security-incidents-in-azure-security-center"></a>Administración de incidentes de seguridad en Azure Security Center

La clasificación e investigación de las alertas de seguridad puede ser un proceso largo incluso para los analistas de seguridad más cualificados. Para muchos, es difícil saber por dónde empezar. 

Security Center usa [análisis](security-center-detection-capabilities.md) para conectar la información entre [alertas de seguridad](security-center-managing-and-responding-alerts.md) diferentes. Con estas conexiones, Security Center puede proporcionar una vista única de una campaña de ataque y sus alertas relacionadas para ayudarle a comprender las acciones del atacante y los recursos afectados.

Esta página contiene información general sobre los incidentes de Security Center.

## <a name="what-is-a-security-incident"></a>¿Qué es un incidente de seguridad?

En Security Center, un incidente de seguridad es la suma de todas las alertas de un recurso que se alinean con patrones de [cadenas de eliminación](alerts-reference.md#intentions) . Los incidentes aparecen en la página [Alertas de seguridad](security-center-managing-and-responding-alerts.md). Seleccione un incidente para ver las alertas relacionadas y obtener más información.

## <a name="managing-security-incidents"></a>Administración de incidentes de seguridad

1. En la página información general de Security Center, seleccione el icono **Alertas de seguridad**. Se enumerarán los incidentes y alertas. Tenga en cuenta que los incidentes de seguridad tienen un icono diferente a las alertas de seguridad.

    ![Visualización de los incidentes de seguridad](./media/security-center-managing-and-responding-alerts/security-center-manage-alerts.png)

1. Para ver detalles, seleccione un incidente. En la página **Incidente de seguridad** se muestran más detalles. 

    [![Respuesta a incidentes de seguridad en Azure Security Center](media/security-center-incident/incident-details.png)](media/security-center-incident/incident-details.png#lightbox)

    En el panel izquierdo de la página Incidente de seguridad se muestra información de alto nivel sobre el incidente de seguridad: título, gravedad, estado, tiempo de actividad, descripción y el recurso afectado. Junto al recurso afectado, puede ver las etiquetas relevantes de Azure. Use estas etiquetas para deducir el contexto de la organización del recurso al investigar la alerta.

    En el panel derecho se incluye la pestaña **Alertas** con las alertas de seguridad que se correlacionan como parte de este incidente. 

    >[!TIP]
    > Para obtener más información acerca de una alerta específica, selecciónela. 

    [![Pestaña Realizar acción del incidente](media/security-center-incident/incident-take-action-tab.png)](media/security-center-incident/incident-take-action-tab.png#lightbox)

    Para cambiar a la pestaña **Realizar acción**, seleccione la pestaña o el botón en la parte inferior del panel derecho. Use esta pestaña para realizar acciones adicionales, como:
    - *Mitigar las amenazas:* proporciona pasos de corrección manual para este incidente de seguridad.
    - *Evitar ataques futuros:* proporciona recomendaciones de seguridad para ayudar a reducir la superficie expuesta a ataques, aumentar la postura de seguridad y evitar futuros ataques.
    - *Desencadenar respuesta automatizada:* ofrece la opción de desencadenar una aplicación lógica como respuesta a este incidente de seguridad.
    - *Suprimir alertas similares:* ofrece la opción de suprimir las alertas futuras con características similares si la alerta no es relevante para su organización. 

   > [!NOTE]
   > La misma alerta puede existir como parte de un incidente y estar visible como alerta independiente.

1. Para corregir las amenazas del incidente, siga los pasos de corrección que se proporcionan con cada alerta.


## <a name="next-steps"></a>Pasos siguientes

En esta página se han explicado las capacidades de incidentes de seguridad de Security Center. Para obtener información relacionada, consulte las páginas siguientes:

- [Alertas de seguridad en Security Center](security-center-alerts-overview.md)
- [Administración de las alertas de seguridad y respuesta a ellas](security-center-managing-and-responding-alerts.md)