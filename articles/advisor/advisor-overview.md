---
title: Introducción a Azure Advisor
description: Utilice Azure Advisor para optimizar las implementaciones de Azure.
ms.topic: article
ms.date: 02/01/2019
ms.openlocfilehash: 27e49fa28e7b02b873ca145d514920784689b407
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88654028"
---
# <a name="introduction-to-azure-advisor"></a>Introducción a Azure Advisor

Obtenga información sobre las funcionalidades clave de Azure Advisor y obtenga respuesta a las preguntas más frecuentes.

## <a name="what-is-advisor"></a>¿Qué es Advisor?
Advisor es un consultor personalizado en la nube que lo ayuda a seguir procedimientos recomendados para optimizar las implementaciones de Azure. Analiza la configuración de recursos y la telemetría de uso, y recomienda soluciones que pueden ayudar a mejorar la rentabilidad, el rendimiento, la confiabilidad (antes conocida como alta disponibilidad) y la seguridad de los recursos de Azure.

Con Advisor, puede:
* Obtener sugerencias de procedimientos recomendados proactivas, prácticas y personalizadas, 
* Mejorar el rendimiento, la seguridad y la confiabilidad de los recursos, al mismo tiempo que identifica oportunidades para reducir el gasto general de Azure, y
* Obtener recomendaciones con acciones propuestas en línea.

Puede acceder a Advisor mediante [Azure Portal](https://aka.ms/azureadvisordashboard). Inicie sesión en el [portal](https://portal.azure.com), busque **Asesor** en el menú de navegación o búsquelo en el menú **Todos los servicios**.

El panel de Advisor muestra recomendaciones personalizadas para todas las suscripciones.  Puede aplicar filtros para mostrar las recomendaciones para determinadas suscripciones y tipos de recursos.  Las recomendaciones se dividen en cinco categorías: 

* **Confiabilidad (anteriormente denominada alta disponibilidad)** : para garantizar y mejorar la continuidad de las aplicaciones empresariales críticas. Para obtener más información, consulte las [recomendaciones de confiabilidad de Advisor](advisor-high-availability-recommendations.md).
* **Seguridad**: ayuda a detectar amenazas y vulnerabilidades que podrían dar lugar a infracciones de seguridad. Para obtener más información, consulte las [recomendaciones sobre seguridad de Advisor](advisor-security-recommendations.md).
* **Rendimiento**: ayuda a mejorar la velocidad de las aplicaciones. Para obtener más información, consulte las [recomendaciones sobre rendimiento de Advisor](advisor-performance-recommendations.md).
* **Costo**: ayuda a optimizar y reducir el gasto general de Azure. Para obtener más información, consulte las [recomendaciones sobre el costo de Advisor](advisor-cost-recommendations.md).
* **Excelencia operativa**: ayuda a conseguir procedimientos recomendados de eficiencia en procesos y flujos de trabajo, manejabilidad de los recursos e implementación. . Para más información, consulte [Recomendaciones de excelencia operativa de Advisor](advisor-operational-excellence-recommendations.md).

  ![Tipos de recomendaciones de Advisor](./media/advisor-overview/advisor-dashboard.png)

Puede hacer clic en una categoría para ver la lista de recomendaciones dentro de esa categoría y seleccionar una recomendación para obtener más información sobre ella.  También puede aprender más sobre las acciones que puede llevar a cabo para aprovechar las ventajas de una oportunidad o resolver un problema.

![Categoría de recomendaciones de Advisor](./media/advisor-overview/advisor-ha-category-example.png) 

Seleccione la acción recomendada para implementar la recomendación.  Se abrirá una interfaz sencilla que le permitirá implementar la recomendación o consultar la documentación que le ayudará con la implementación.  Una vez que implemente una recomendación, Advisor puede tardar un día en reconocerla.

Si no desea realizar de inmediato una acción basada en una recomendación, puede posponerla durante un período de tiempo o descartarla.  SI no desea recibir recomendaciones para un grupo de recursos o una suscripción específicos, puede configurar Advisor para generar solo recomendaciones para grupos de recursos y suscripciones específicos.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="how-do-i-access-advisor"></a>¿Cómo se accede a Advisor?
Puede acceder a Advisor mediante [Azure Portal](https://aka.ms/azureadvisordashboard). Inicie sesión en el [portal](https://portal.azure.com), busque **Asesor** en el menú de navegación o búsquelo en el menú **Todos los servicios**.

También puede ver las recomendaciones de Advisor a través de la interfaz de recursos de la máquina virtual. Seleccione una máquina virtual y después desplácese a las recomendaciones de Advisor en el menú. 

### <a name="what-permissions-do-i-need-to-access-advisor"></a>¿Qué permisos son necesarios para acceder a Advisor?
 
Puede acceder a las recomendaciones de Advisor como *Propietario*, *Colaborador* o *Lector* de una suscripción, grupo de recursos o recurso.

### <a name="what-resources-does-advisor-provide-recommendations-for"></a>¿Para qué recursos Advisor ofrece recomendaciones?

Advisor proporciona recomendaciones sobre Application Gateway, App Services, conjuntos de disponibilidad, Azure Cache, Azure Data Factory, Azure Database for MySQL, Azure Database for PostgreSQL, Azure Database for MariaDB, Azure ExpressRoute, Azure Cosmos DB, direcciones IP públicas de Azure, Azure Synapse Analytics, servidores SQL Server, cuentas de almacenamiento, perfiles de Traffic Manager y máquinas virtuales.

Azure Advisor también incluye recomendaciones procedentes de [Azure Security Center](../security-center/security-center-recommendations.md), que pueden abarcar recomendaciones de tipos de recursos adicionales.

### <a name="can-i-postpone-or-dismiss-a-recommendation"></a>¿Se puede posponer o descartar una recomendación?

Para posponer o descartar una recomendación, haga clic en el vínculo **Postpone** (Posponer). Puede especificar un tiempo de posposición o seleccionar **Never** (Nunca) para descartar la recomendación.

## <a name="next-steps"></a>Pasos siguientes

Para aprender más sobre las recomendaciones de Advisor, consulte:

* [Introducción a Advisor](advisor-get-started.md)
* [Recomendaciones de confiabilidad de Advisor](advisor-high-availability-recommendations.md)
* [Recomendaciones sobre seguridad de Advisor](advisor-security-recommendations.md)
* [Recomendaciones sobre rendimiento de Advisor](advisor-performance-recommendations.md)
* [Recomendaciones sobre el costo de Advisor](advisor-cost-recommendations.md)
