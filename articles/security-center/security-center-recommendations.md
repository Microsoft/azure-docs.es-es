---
title: Recomendaciones de seguridad en el Centro de seguridad de Azure
description: Este documento explica cómo las recomendaciones del Centro de seguridad de Azure ayudan a proteger los recursos de Azure y a cumplir con las directivas de seguridad.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 86c50c9f-eb6b-4d97-acb3-6d599c06133e
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/29/2019
ms.author: memildin
ms.openlocfilehash: 6363100c844d071a3bb47521cec6ff7e988f6af8
ms.sourcegitcommit: 64ad2c8effa70506591b88abaa8836d64621e166
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88263242"
---
# <a name="security-recommendations-in-azure-security-center"></a>Recomendaciones de seguridad en el Centro de seguridad de Azure 
En este tema se explica cómo ver y entender las recomendaciones de Azure Security Center para ayudar a proteger los recursos de Azure.

> [!NOTE]
> En este documento se presenta el servicio mediante una implementación de ejemplo.  Este documento no es una guía paso a paso.
>

## <a name="what-are-security-recommendations"></a>¿Cuáles son las recomendaciones de seguridad?

Las recomendaciones son acciones que se deben llevar a cabo para proteger los recursos.

Security Center analiza periódicamente el estado de seguridad de los recursos de Azure para identificar posibles puntos vulnerables de la seguridad. A continuación, se proporcionan recomendaciones sobre cómo corregir dichos puntos vulnerables.

Cada recomendación le proporciona:

- Descripción breve de la regla.
- Los pasos de corrección que se deben llevar a cabo para implementar la recomendación.
- Recursos afectados.

## <a name="monitor-recommendations"></a>Supervisión de recomendaciones<a name="monitor-recommendations"></a>

Security Center analiza el estado de seguridad de los recursos para identificar posibles vulnerabilidades. El icono **Recomendaciones** de **Información general** permite conocer el número total de recomendaciones que identifica Security Center.

![Introducción a Security Center](./media/security-center-recommendations/asc-overview.png)

1. Seleccione el **icono de Recomendaciones** en **Información general**. Se abre la lista **Recomendaciones**.

1. Las recomendaciones se agrupan en controles de seguridad.

      ![Recomendaciones agrupadas por control de seguridad](./media/security-center-recommendations/view-recommendations.png)

1. Expanda un control y seleccione una recomendación concreta para ver la página de recomendaciones.

    :::image type="content" source="./media/security-center-recommendations/recommendation-details-page.png" alt-text="Página de detalles de recomendación" lightbox="./media/security-center-recommendations/recommendation-details-page.png":::

    La página incluye:

    - **Indicador de gravedad**
    - **Intervalo de actualización** (si procede) 
    - **Descripción**: una breve descripción del problema
    - **Pasos de corrección**: una descripción de los pasos manuales necesarios para corregir el problema de seguridad en los recursos afectados Para obtener recomendaciones con "corrección rápida", puede seleccionar **Ver lógica de corrección** antes de aplicar la corrección sugerida a los recursos. 
    - **Recursos afectados**: los recursos se agrupan en pestañas:
        - **Recursos con estado correcto**: recursos relevantes que no se ven afectados o en los que ya se ha corregido el problema.
        - **Recursos con estado incorrecto**: recursos que todavía se ven afectados por el problema identificado.
        - **Recursos no aplicables**: recursos para los que la recomendación no puede dar una respuesta definitiva. La pestaña no aplicable también incluye las razones de cada recurso. 

            :::image type="content" source="./media/security-center-recommendations/recommendations-not-applicable-reasons.png" alt-text="Los recursos no aplicables con motivos":::.


 
## <a name="next-steps"></a>Pasos siguientes

En este documento, se han presentando las recomendaciones de seguridad del Centro de seguridad. Para obtener información sobre cómo corregir las recomendaciones:

* [Recomendaciones de corrección ](security-center-remediate-recommendations.md): aprenda a configurar directivas de seguridad para las suscripciones y los grupos de recursos de Azure.
