---
title: Experiencia Simple Logs en Azure Monitor (versión preliminar) | Microsoft Docs
description: La experiencia de Simple Logs le permite crear consultas básicas en Azure Monitor sin interactuar directamente con KQL.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 09/12/2019
ms.openlocfilehash: 4926e18aa6b00fe36608843ea5253903ace774e2
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90089115"
---
# <a name="simple-logs-experience-in-azure-monitor-preview"></a>Experiencia Simple Logs en Azure Monitor (versión preliminar)
Azure Monitor proporciona una [experiencia enriquecida](get-started-portal.md) para crear [consultas de registro](log-query-overview.md) mediante el lenguaje KQL. No obstante, es posible que no necesite toda la eficacia de KQL y prefiera una experiencia simplificada para los requisitos de consulta básicos. La experiencia de Simple Logs le permite crear consultas básicas sin interactuar directamente con KQL. También puede usar Simple Logs como herramienta de aprendizaje para KQL a medida que necesite consultas más sofisticadas.

> [!NOTE]
> Simple Logs actualmente está implementado como una prueba solo para Cosmos DB y los almacenes de claves. Comparta su experiencia con Microsoft a través de [User Voice](https://feedback.azure.com/forums/913690-azure-monitor) para ayudarnos a determinar si se ampliará y publicará esta característica.


## <a name="scope"></a>Ámbito
La experiencia de Simple Logs recupera datos de la tabla *AzureDiagnostics*, *AzureMetrics* y *AzureActivity* para el recurso seleccionado. 

## <a name="using-simple-logs"></a>Uso de Simple Logs
Desplácese a cualquier instancia de Cosmos DB o Key Vault de su suscripción a Azure con [opciones de diagnóstico configuradas para recopilar registros en un área de trabajo de Log Analytics](../platform/resource-logs.md#send-to-azure-storage). Haga clic en **Registros** en el menú **Supervisión** para abrir la experiencia de Simple Logs.

![Captura de pantalla que muestra el menú de supervisión con registros seleccionados.](media/simple-logs/menu.png)

Seleccione un **Campo** y un **Operador** y especifique un **Valor** para la comparación. Haga clic en **+** y especifique **Y/O** para agregar criterios adicionales.

![Captura de pantalla que muestra la búsqueda en el panel de registros con registros simples seleccionados.](media/simple-logs/criteria.png)

Haga clic en **Ejecutar** para ver los resultados de la consulta.

## <a name="view-and-edit-kql"></a>Visualización y edición de KQL
Seleccione **Editor de consultas** para abrir el KQL generado por la consulta de Simple Logs en la experiencia completa de Log Analytics. 

![Editor de consultas](media/simple-logs/query-editor.png)

Puede editar de forma directa el KQL y usar otras características de Log Analytics como filtros para refinar aún más los resultados.

![Edición de KQL](media/simple-logs/edit-kql.png)


## <a name="next-steps"></a>Pasos siguientes

- Complete un tutorial sobre el [uso de Log Analytics en Azure Portal](get-started-portal.md).
- Complete un tutorial sobre cómo [escribir consultas de registros](get-started-portal.md).
