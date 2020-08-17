---
title: Datos de Advisor en Azure Resource Graph
description: Creación de consultas para datos de Advisor en Azure Resource Graph
ms.topic: article
ms.date: 03/12/2020
ms.author: sagupt
ms.openlocfilehash: 66bb500d419d1f5537afafd7a2df543ded8cc7ce
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87057777"
---
# <a name="query-for-advisor-data-in-resource-graph-explorer-azure-resource-graph"></a>Consulta de datos de Advisor en Resource Graph Explorer (Azure Resource Graph)

Los recursos de Advisor ahora están incorporados en [Azure Resource Graph](https://azure.microsoft.com/features/resource-graph/). Esta es la base de muchos escenarios de clientes a escala para las recomendaciones de Advisor. Los siguientes son algunos de los escenarios que no eran posibles antes de hacerlo a escala y que ahora se pueden lograr con Resource Graph:
* Proporciona la capacidad de realizar consultas complejas para todas las suscripciones en Azure Portal.
* Recomendaciones resumidas por tipos de categoría (como alta confiabilidad, rendimiento) y tipos de impacto (alto, medio, bajo)
* Todas las recomendaciones para un tipo de recomendación concreto.
* Recuento de recursos afectados por categoría de recomendación.

![Advisor en Azure Resource Graph Explorer](./media/azure-resource-graph-1.png)  


## <a name="advisor-resource-types-in-azure-graph"></a>Tipos de recursos de Advisor en Azure Graph

Tipos de recursos de Advisor disponibles en [Resource Graph](../governance/resource-graph/index.yml): hay tres tipos de recursos disponibles para las consultas en recursos de Advisor. Esta es la lista de los recursos que ahora están disponibles para realizar consultas en Resource Graph.
* Microsoft.Advisor/configurations
* Microsoft.Advisor/recommendations
* Microsoft.Advisor/suppressions

Estos tipos de recursos se muestran en una nueva tabla denominada AdvisorResources, que también se puede consultar en Resource Graph Explorer en Azure Portal.


## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de las recomendaciones de Advisor, consulte:
* [Introducción a Azure Advisor](advisor-overview.md)
* [Introducción a Advisor](advisor-get-started.md)
* [Recomendaciones sobre el costo de Advisor](advisor-cost-recommendations.md)
* [Recomendaciones de confiabilidad de Advisor](advisor-high-availability-recommendations.md)
* [Recomendaciones sobre rendimiento de Advisor](advisor-performance-recommendations.md)
* [Recomendaciones sobre seguridad de Advisor](advisor-security-recommendations.md)
* [Recomendaciones de excelencia operativa de Advisor](advisor-operational-excellence-recommendations.md)
* [API REST de Advisor](/rest/api/advisor/)
