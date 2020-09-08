---
title: Elección entre Azure Cost Management y Cloudyn
description: 'Este artículo le ayuda a determinar cuál es la mejor opción para sus necesidades de administración de costos: Azure Cost Management o Cloudyn.'
author: bandersmsft
ms.author: banders
ms.date: 03/20/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: adwise
ms.openlocfilehash: a46e02bfa9590b5ef34affa45f3c468fba6999c2
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88684973"
---
# <a name="choose-between-azure-cost-management-and-cloudyn"></a>Elección entre Azure Cost Management y Cloudyn

Cloudyn pasará a estar en desuso a finales de 2020. Las características de Cloudyn existentes se están integrando directamente en Azure Portal siempre que es posible. A excepción de los clientes de CSP, en este momento no se pueden incorporar nuevos clientes. La compatibilidad con el producto existente se mantendrá hasta que esté completamente en desuso.

Microsoft adquirió Cloudyn y está migrando sus características de administración de costos desde el portal de Cloudyn de manera nativa a Azure. Para usar las nuevas características, inicie sesión en Azure Portal y vaya a [Administración de costos + facturación](https://ms.portal.azure.com/#blade/Microsoft_Azure_CostManagement/Menu/overview) en la lista de servicios de Azure. En comparación con Cloudyn, la experiencia nativa ofrece un mejor rendimiento y una menor latencia de los datos de unas ocho horas.

La migración de las características clave de Contrato Enterprise, Pago por uso y las categorías de oferta de MSDN a Azure Cost Management se completó. Las suscripciones de CSP están en proceso de migración a Azure Cost Management.

Si hay una categoría de oferta que todavía no se migra, debe seguir usando el portal de Cloudyn. Todos los demás pueden usar Azure Cost Management.

## <a name="recommended-services-by-offer"></a>Servicios recomendados por oferta

| Ofertas de Microsoft Azure | Servicio recomendado de administración de costos |
| --- | --- |
| Contrato Enterprise de Azure | [Azure Cost Management](https://ms.portal.azure.com/#blade/Microsoft_Azure_CostManagement/Menu/overview) |
| Azure Web Direct (Pago por uso/MSDN) | [Azure Cost Management](https://ms.portal.azure.com/#blade/Microsoft_Azure_CostManagement/Menu/overview) |
| Azure Government | [Azure Cost Management](https://ms.portal.azure.com/#blade/Microsoft_Azure_CostManagement/Menu/overview) |
| Contrato de cliente de Microsoft | [Azure Cost Management](https://ms.portal.azure.com/#blade/Microsoft_Azure_CostManagement/Menu/overview)|
| Contrato de cliente de Microsoft admitido por los asociados | [Azure Cost Management](https://ms.portal.azure.com/#blade/Microsoft_Azure_CostManagement/Menu/overview)|
| Azure CSP | [Cloudyn](https://azure.cloudyn.com) |


## <a name="available-cost-management-features"></a>Características de Cost Management disponibles

Algunas de las características siguientes están disponibles en Cloudyn, pero la totalidad de ellas están disponibles ahora en Azure Cost Management.

- API existentes
- Recomendaciones de optimización de costos de Azure, entre ellas:
    - Recomendaciones para el tamaño adecuado y el cierre de la instancia de Azure
    - Recomendaciones de reservas de Azure
- Presupuestos
- Análisis de costos
- Exportación de datos a una cuenta de almacenamiento de Azure
- Menor latencia
- Aplicación de plantilla de Power BI
- Compatibilidad con etiquetas de recurso
- Compatibilidad del análisis de costos en toda la nube con AWS

## <a name="next-steps"></a>Pasos siguientes
- Más información sobre [Azure Cost Management](../cost-management-billing-overview.md).