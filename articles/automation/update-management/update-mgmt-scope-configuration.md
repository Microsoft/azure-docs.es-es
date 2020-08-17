---
title: Limitación del ámbito de implementación de Update Management de Azure Automation
description: En este artículo se indica cómo usar las configuraciones de ámbito para limitar el ámbito de implementación de Update Management.
services: automation
ms.date: 03/04/2020
ms.topic: conceptual
ms.custom: mvc
ms.openlocfilehash: aafd284122ca61ba2b668186095b88003be2775c
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87449639"
---
# <a name="limit-update-management-deployment-scope"></a>Limitación del ámbito de implementación de Update Management

En este artículo se describe cómo trabajar con configuraciones de ámbito cuando se utiliza la característica [Update Management](update-mgmt-overview.md) para implementar actualizaciones y revisiones en las máquinas virtuales. Para más información, consulte [Soluciones de supervisión como destino en Azure Monitor (versión preliminar)](../../azure-monitor/insights/solution-targeting.md).

## <a name="about-scope-configurations"></a>Acerca de las configuraciones de ámbito

Una configuración de ámbito es un grupo de una o varias búsquedas (consultas) utilizadas para limitar el ámbito de Update Management a equipos concretos. La configuración de ámbito se usa dentro del área de trabajo de Log Analytics para seleccionar como destino los equipos que se van a habilitar. Cuando agrega un equipo para recibir actualizaciones de Update Management, el equipo también se agrega a una búsqueda guardada en el área de trabajo.

## <a name="set-the-scope-limit"></a>Definición del límite de ámbito

Para limitar el ámbito de implementación de Update Management:

1. En la cuenta de Automation, seleccione **Área de trabajo vinculada**, en **Recursos relacionados**.

2. Seleccione **Ir al área de trabajo**.

3. Seleccione **Configuraciones de ámbito (versión preliminar)** en **Orígenes de datos del área de trabajo**.

4. Seleccione los puntos suspensivos a la derecha de la configuración del ámbito `MicrosoftDefaultScopeConfig-Updates` y seleccione **Editar**.

5. En el panel de edición, expanda **Seleccionar grupos de equipos**. En el panel Grupos de equipos se muestran las búsquedas guardadas que se usan para crear la configuración de ámbito. La búsqueda guardada que usa Update Management es la siguiente:

    |Nombre     |Category  |Alias  |
    |---------|---------|---------|
    |MicrosoftDefaultComputerGroup     | Actualizaciones        | Updates__MicrosoftDefaultComputerGroup         |

6. Seleccione la búsqueda guardada para ver y editar la consulta usada para rellenar el grupo. La siguiente imagen muestra la consulta y sus resultados:

    [ ![Búsquedas guardadas](./media/update-mgmt-scope-configuration/logsearch.png)](./media/update-mgmt-scope-configuration/logsearch-expanded.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes

Puede [consultar los registros de Azure Monitor](update-mgmt-query-logs.md) para analizar las evaluaciones de las actualizaciones, las implementaciones y otras tareas de administración relacionadas. Incluye consultas predefinidas para ayudarle a empezar.
