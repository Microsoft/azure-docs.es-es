---
title: Investigación de alertas con Azure Sentinel | Microsoft Docs
description: Aprenda a usar las plantillas de detección de amenazas de Azure integradas y listas para usar, que le avisan cuando ocurre algo sospechoso.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/06/2020
ms.author: yelevin
ms.openlocfilehash: 5d73337c25c812363b7a542bf42372ca3baa10e8
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88605447"
---
# <a name="tutorial-detect-threats-out-of-the-box"></a>Tutorial: Detección de amenazas integrada


> [!IMPORTANT]
> La detección de amenazas integrada se encuentra actualmente en versión preliminar pública.
> Esta característica se ofrece sin contrato de nivel de servicio y no se recomienda para cargas de trabajo de producción.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Después de [conectar los orígenes de datos](quickstart-onboard.md)  a Azure Sentinel, querrá recibir una notificación cuando suceda algo sospechoso. Por este motivo, Azure Sentinel proporciona plantillas integradas predefinidas para ayudarle a crear reglas de detección de amenazas. Estas plantillas fueron diseñadas por un equipo de expertos y analistas en seguridad de Microsoft basándose en amenazas conocidas, vectores de ataque habituales y cadenas de escalado de actividades sospechosas. Las reglas creadas a partir de estas plantillas buscarán automáticamente en el entorno las actividades que parezcan sospechosas. Muchas de las plantillas se pueden personalizar para buscar o filtrar actividades, según sus necesidades. Las alertas generadas por estas reglas crearán incidentes que puede asignar e investigar en su entorno.

Este tutorial ayuda a detectar amenazas con Azure Sentinel:

> [!div class="checklist"]
> * Uso de detecciones de amenazas predefinidas
> * Automatizar las respuestas frente a amenazas

## <a name="about-out-of-the-box-detections"></a>Acerca de las detecciones integradas

Para ver todas las detecciones estándar, vaya a **Análisis** y, después, a **Rule templates** (Plantillas de reglas). Esta pestaña contiene todas las reglas integradas de Azure Sentinel.

   :::image type="content" source="media/tutorial-detect-built-in/view-oob-detections.png" alt-text="Usar las detecciones integradas para encontrar amenazas con Azure Sentinel":::

Están disponibles los siguientes tipos de plantilla:

- **Seguridad de Microsoft**
   
   Las plantillas de seguridad de Microsoft crean automáticamente incidentes de Azure Sentinel a partir de las alertas generadas en otras soluciones de seguridad de Microsoft, en tiempo real. Puede usar las reglas de seguridad de Microsoft como plantilla para crear nuevas reglas con una lógica parecida. Para más información sobre las reglas de seguridad, consulte [Creación automática de incidentes a partir de alertas de seguridad de Microsoft](create-incidents-from-alerts.md).

- **Fusión** 

    Basada en la tecnología de fusión, la detección avanzada de ataques de varias fases de Azure Sentinel emplea algoritmos de aprendizaje automático escalables que pueden correlacionar muchas alertas y eventos de baja fidelidad de varios productos con incidentes útiles y de alta fidelidad. La fusión está habilitada de manera predeterminada. Dado que la lógica está oculta y, por lo tanto, no se puede personalizar, solo puede crear una regla con esta plantilla.

- **Análisis de comportamiento de aprendizaje automático**

    Estas plantillas se basan en algoritmos de aprendizaje automático de Microsoft, por lo que no se puede ver la lógica interna de cómo funcionan y cuándo se ejecutan. Dado que la lógica está oculta y, por lo tanto, no se puede personalizar, solo puede crear una regla con cada plantilla de este tipo.

- **Programado**

    Las reglas de análisis programado se basan en consultas integradas escritas por expertos de seguridad de Microsoft. Puede ver la lógica de consulta y realizar cambios en ella. Puede usar la plantilla de reglas programadas y personalizar la lógica de consulta y la configuración de programación para crear otras reglas.

## <a name="use-out-of-the-box-detections"></a>Usar detecciones integradas

1. Para usar una plantilla integrada, haga clic en el nombre de la plantilla y, luego, haga clic en el botón **Crear regla** en el panel de detalles para crear una regla activa basada en esa plantilla. Cada plantilla tiene una lista de orígenes de datos necesarios. Al abrir la plantilla, se comprueba automáticamente la disponibilidad de los orígenes de datos. Si hay un problema de disponibilidad, es posible que el botón **Crear regla** esté deshabilitado, o puede que vea una advertencia a tal efecto.
  
    :::image type="content" source="media/tutorial-detect-built-in/use-built-in-template.png" alt-text="Panel de vista previa de reglas de detección":::
 
1. Al hacer clic en el botón **Crear regla**, se abre el Asistente para la creación de reglas basado en la plantilla seleccionada. Todos los detalles se rellenan automáticamente y, con las plantillas **Programado** o **Microsoft security** (Seguridad de Microsoft), puede personalizar la lógica y otras configuraciones de reglas para satisfacer mejor sus necesidades específicas. Este proceso puede repetirse para crear reglas adicionales en función de la plantilla integrada. Después de seguir los pasos del Asistente para la creación de reglas hasta el final, habrá terminado de crear una regla basada en la plantilla. Las nuevas reglas aparecerán en la pestaña **Active rules** (Reglas activas).

    Para más información sobre cómo personalizar las reglas en el Asistente para la creación de reglas, consulte [Tutorial: Creación de reglas de análisis personalizadas para detectar amenazas](tutorial-detect-threats-custom.md).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a detectar amenazas casos mediante Azure Sentinel. 

Para aprender a automatizar las respuestas a las amenazas, consulte [Configuración de respuestas automatizadas frente a amenazas en Azure Sentinel](tutorial-respond-threats-playbook.md).

