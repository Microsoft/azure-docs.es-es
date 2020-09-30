---
title: Administración de alertas de seguridad en Azure Security Center | Microsoft Docs
description: Este documento le ayuda a usar las capacidades del Centro de seguridad de Azure para administrar y responder a las alertas de seguridad.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: b88a8df7-6979-479b-8039-04da1b8737a7
ms.service: security-center
ms.topic: conceptual
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/15/2020
ms.author: memildin
ms.openlocfilehash: 2551a3c860dbdf27e7c9473e88c3a3293f575faa
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90883950"
---
# <a name="manage-and-respond-to-security-alerts-in-azure-security-center"></a>Administración y respuesta a alertas de seguridad en Azure Security Center

En este tema se muestra cómo ver y procesar las alertas que ha recibido con el fin de proteger los recursos. 

* Para obtener información sobre los distintos tipos de alertas, vea [Tipos de alertas de seguridad](alerts-reference.md).
* Para obtener información general sobre la forma en que Security Center genera alertas, vea [¿Cómo detecta Security Center las amenazas?](security-center-alerts-overview.md)

> [!NOTE]
> Para habilitar las detecciones avanzadas, habilite Azure Defender. Hay una evaluación gratuita disponible. Para realizar la actualización, seleccione el plan de tarifa en la [directiva de seguridad](tutorial-security-policy.md). Consulte [Precios de Azure Security Center](security-center-pricing.md) para más información.

## <a name="what-are-security-alerts"></a>¿Qué son las alertas de seguridad?
Security Center recopila, analiza e integra automáticamente los datos de registro de los recursos de Azure, la red y las soluciones de asociados conectados, como firewalls y soluciones de protección de puntos de conexión, para detectar amenazas reales y reducir los falsos positivos. En Security Center, se muestra una lista de alertas de seguridad prioritarias, junto con la información que necesita para investigar rápidamente y recomendaciones para corregir un ataque.

> [!NOTE]
> Para obtener más información sobre la forma en que actúan las funcionalidades de detección de Security Center, vea [¿Cómo detecta Security Center las amenazas?](security-center-alerts-overview.md#detect-threats)

## <a name="manage-your-security-alerts"></a>Administración de las alertas de seguridad

1. En el panel de Security Center, consulte el icono **Protección contra amenazas** para ver y obtener información general de las alertas.

    ![Icono Alertas de seguridad en Security Center](./media/security-center-managing-and-responding-alerts/security-center-dashboard-alert.png)

1. Para ver más detalles sobre las alertas, haga clic en el icono.

   ![Las alertas de seguridad en Security Center](./media/security-center-managing-and-responding-alerts/security-center-manage-alerts.png)

1. Para filtrar las alertas que se muestran, haga clic en **Filtro** y, en la hoja **Filtro** que se abre, seleccione las opciones de filtro que quiere aplicar. La lista se actualiza según el filtro seleccionado. El filtrado puede resultar muy útil. Por ejemplo, podría comprobar las alertas de seguridad que se produjeron en las 24 horas anteriores, ya que se está investigando una posible brecha en el sistema.

    ![Filtrado de alertas en Security Center](./media/security-center-managing-and-responding-alerts/security-center-filter-alerts.png)

## <a name="respond-to-security-alerts"></a>Responder a alertas de seguridad

1. En la lista **Alertas de seguridad**, haga clic en una alerta de seguridad. Se muestran los recursos implicados y los pasos que debe seguir para solucionar las consecuencias de un ataque.

    ![Responder a alertas de seguridad](./media/security-center-managing-and-responding-alerts/security-center-alert.png)

1. Después de revisar esta información, haga clic en un recurso atacado.

    En el panel izquierdo de la página alerta de seguridad se muestra información de alto nivel sobre la alerta de seguridad: título, gravedad, estado, tiempo de actividad, descripción de la actividad sospechosa y el recurso afectado. Junto con el recurso afectado se encuentran las etiquetas de Azure relevantes para el recurso. Úselas para deducir el contexto de la organización del recurso al investigar la alerta.

    En el panel derecho se incluye la pestaña **Detalles de alerta** que contiene más detalles de la alerta para ayudarle a investigar el problema: Direcciones IP, archivos, procesos, etc.
     
    ![Sugerencias sobre qué hacer con las alertas de seguridad](./media/security-center-managing-and-responding-alerts/security-center-alert-remediate.png)

    Además, en el panel derecho se encuentra la pestaña **Realizar acción**. Use esta pestaña para realizar acciones adicionales con respecto a la alerta de seguridad. Acciones como:
    - *Mitigar las amenazas:* proporciona pasos de corrección manual para esta alerta de seguridad
    - *Evitar ataques futuros:* proporciona recomendaciones de seguridad para ayudar a reducir la superficie expuesta a ataques, aumentar la postura de seguridad y así evitar futuros ataques
    - *Desencadenar respuesta automatizada:* ofrece la opción de desencadenar una aplicación lógica como respuesta a esta alerta de seguridad
    - *Suprimir alertas similares:* ofrece la opción de suprimir las alertas futuras con características similares si la alerta no es relevante para su organización

    ![Pestaña Realizar acción](./media/security-center-managing-and-responding-alerts/alert-take-action.png)




## <a name="see-also"></a>Consulte también

En este documento, aprendió cómo ver alertas de seguridad. Vea las páginas siguientes para obtener material relacionado:

- [Configuración de reglas de eliminación de alertas](alerts-suppression-rules.md)
- [Automatización de respuestas a alertas y recomendaciones con automatización de flujos de trabajo](workflow-automation.md)
