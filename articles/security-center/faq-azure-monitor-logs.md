---
title: 'Preguntas frecuentes sobre Azure Security Center: preguntas sobre agentes de Log Analytics existentes'
description: En este artículo de preguntas frecuentes se responde a preguntas de los clientes que ya usan el agente de Log Analytics y que se están planteando el empleo de Azure Security Center, un producto que ayuda a evitar amenazas, a detectarlas y a responder a ellas.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: be2ab6d5-72a8-411f-878e-98dac21bc5cb
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/25/2020
ms.author: memildin
ms.openlocfilehash: 0f4552d6488ecd083b6ee5d4cae2ef2bd660efc7
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906382"
---
# <a name="faq-for-customers-already-using-azure-monitor-logs"></a>Preguntas frecuentes para clientes que ya usan registros de Azure Monitor<a name="existingloganalyticscust"></a>

## <a name="does-security-center-override-any-existing-connections-between-vms-and-workspaces"></a>¿Invalida Security Center las conexiones existentes entre VM y áreas de trabajo?

Si una VM ya tiene el agente de Log Analytics instalado como una extensión de Azure, Security Center no invalida la conexión de área de trabajo existente. En su lugar, Security Center usa el área de trabajo existente. La máquina virtual se protege siempre que se haya instalado la solución "Security" o "SecurityCenterFree" en el área de trabajo a la que está notificando. 

Una solución de Security Center se instala en el área de trabajo seleccionada en la pantalla Colección de datos si todavía no está presente y la solución solo se aplica a las máquinas virtuales pertinentes. Cuando se agrega una solución, se implementa automáticamente de forma predeterminada en todos los agentes de Windows y Linux conectados al área de trabajo de Log Analytics. La [selección de destino de solución](../operations-management-suite/operations-management-suite-solution-targeting.md) le permite aplicar un ámbito a sus soluciones.

> [!TIP]
> Si el agente de Log Analytics se instala directamente en la máquina virtual (no como una extensión de Azure), Security Center no lo instala y la supervisión de seguridad estará limitada.

## <a name="does-security-center-install-solutions-on-my-existing-log-analytics-workspaces-what-are-the-billing-implications"></a>¿Instala Security Center soluciones en mis áreas de trabajo de Log Analytics existentes? ¿Qué implica desde el punto de vista de la facturación?
Cuando Security Center identifica que una VM ya está conectada a un área de trabajo creada por el usuario, habilita las soluciones en esta área de trabajo según la configuración de precios. Las soluciones se aplican solo a las VM de Azure pertinentes a través de la [selección de destino de solución](../operations-management-suite/operations-management-suite-solution-targeting.md), por lo que la facturación sigue siendo la misma.

- **Azure Defender desactivado**: Security Center instala la solución “SecurityCenterFree” en el área de trabajo. No se le facturará.
- 
- **Azure Defender activado**: Security Center instala la solución “Security” en el área de trabajo.

   ![Soluciones del área de trabajo predeterminada](./media/security-center-platform-migration-faq/solutions.png)

## <a name="i-already-have-workspaces-in-my-environment-can-i-use-them-to-collect-security-data"></a>Ya tengo áreas de trabajo en mi entorno, ¿puedo utilizarlas para recopilar datos de seguridad?
Si una VM ya tiene el agente de Log Analytics instalado como una extensión de Azure, Security Center usa el área de trabajo conectada existente. Una solución de Security Center se instala en el área de trabajo si todavía no está presente y la solución solo se aplica a VM relevantes mediante la [selección de destino de solución](../operations-management-suite/operations-management-suite-solution-targeting.md).

Cuando Security Center instala el agente de Log Analytics en las máquinas virtuales, usa los espacios de trabajo predeterminados creados por Security Center si este no apunta a un área de trabajo ya existente.

## <a name="i-already-have-security-solution-on-my-workspaces-what-are-the-billing-implications"></a>Ya tengo una solución de seguridad en mis áreas de trabajo. ¿Qué implica desde el punto de vista de la facturación?
La solución Seguridad y auditoría se usa para habilitar **Azure defender para servidores**. Si la solución Seguridad y auditoría ya está instalada en un área de trabajo, Security Center utiliza la solución existente. No hay ningún cambio en la facturación.
