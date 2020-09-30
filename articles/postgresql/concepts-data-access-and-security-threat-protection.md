---
title: 'Advanced Threat Protection: Azure Database for PostgreSQL con un único servidor'
description: Obtenga más información acerca del uso de Advanced Threat Protection para detectar actividades anómalas en la base de datos que indican posibles amenazas de seguridad.
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: fbb6a7dbe73444ac9049573705e4d04bd868a806
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90884556"
---
# <a name="advanced-threat-protection-in-azure-database-for-postgresql---single-server"></a>Advanced Threat Protection en Azure Database for PostgreSQL con un único servidor

Advanced Threat Protection para Azure Database for PostgreSQL detecta actividades anómalas que indican intentos inusuales y potencialmente dañinos de acceso o ataque a las bases de datos.

> [!NOTE]
> Advanced Threat Protection está en versión preliminar pública.

Threat Protection forma parte de la oferta Advanced Threat Protection (ATP), que es un paquete unificado para funcionalidades avanzadas de seguridad. Se puede obtener acceso a Advanced Threat Protection y administrarlo por medio de [Azure Portal](https://portal.azure.com) y la [API REST](/rest/api/postgresql). La característica está disponible para los servidores de uso general y para los optimizados para memoria.

> [!NOTE]
> La característica Advanced Threat Protection **no** está disponible en las siguientes regiones de nube soberana y Azure Government: US Gov Texas, US Gov Arizona, US Gov Iowa, US Gov Virginia, US DoD (este), US DoD (centro), Centro de Alemania, Norte de Alemania, Este de China y Este de China 2. Visite [productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/) para obtener información de disponibilidad general del producto.

## <a name="what-is-advanced-threat-protection"></a>¿Qué es Advanced Threat Protection?

Advanced Threat Protection para Azure Database for PostgreSQL proporciona una nueva capa de seguridad, que permite a los clientes detectar posibles amenazas y responder a ellas cuando se producen, gracias a que proporciona alertas de seguridad sobre actividades anómalas. Los usuarios reciben una alerta sobre actividades sospechosas en las bases de datos, posibles puntos vulnerables, así como sobre patrones anómalos de consulta y acceso a las bases de datos. Advanced Threat Protection para Azure Database for PostgreSQL integra las alertas con [Azure Security Center](https://azure.microsoft.com/services/security-center/), que incluye detalles de actividades sospechosas y recomienda acciones sobre cómo investigar y mitigar la amenaza. Advanced Threat Protection para Azure Database for PostgreSQL facilita la solución de posibles amenazas a la base de datos sin necesidad de ser un experto en seguridad ni tener que administrar sistemas de supervisión de seguridad avanzada. 

:::image type="content" source="media/concepts-data-access-and-security-threat-protection/advanced-threat-protection-concept.png" alt-text="Concepto de Advanced Threat Protection":::

## <a name="advanced-threat-protection-alerts"></a>Alertas de Advanced Threat Protection 
Advanced Threat Protection para Azure Database for PostgreSQL detecta actividades anómalas que indican intentos inusuales y potencialmente peligrosos de acceder a bases de datos, o de vulnerar su seguridad, y puede desencadenar las siguientes alertas:
- **Acceso desde una ubicación inusual**: esta alerta se desencadena cuando se produce un cambio en el patrón de acceso al servidor de Azure Database for PostgreSQL, donde alguien ha iniciado sesión en el servidor de Azure Database for PostgreSQL desde una ubicación geográfica inusual. En algunos casos, la alerta detecta una acción legítima (una nueva aplicación o el mantenimiento de un desarrollador). En otros casos, la alerta detecta una acción malintencionada (por ejemplo, un antiguo empleado o un atacante externo).
- **Acceso desde un centro de datos de Azure inusual**: esta alerta se desencadena cuando se produce un cambio en el patrón de acceso al servidor de Azure Database for PostgreSQL, donde alguien ha iniciado sesión en el servidor desde un centro de datos de Azure inusual que se vio en este servidor recientemente. En algunos casos, la alerta detecta una acción legítima (una aplicación nueva en Azure, Power BI, el editor de consultas de Azure Database for PostgreSQL). En otros casos, la alerta detecta una acción malintencionada procedente de un recurso o servicio de Azure (por ejemplo, un antiguo empleado o un atacante externo).
- **Acceso desde una entidad de seguridad desconocida**: esta alerta se desencadena cuando se produce un cambio en el patrón de acceso al servidor de Azure Database for PostgreSQL, donde alguien ha iniciado sesión en el servidor mediante una entidad de seguridad inusual (un usuario de Azure Database for PostgreSQL). En algunos casos, la alerta detecta una acción legítima (una nueva aplicación o el mantenimiento de un desarrollador). En otros casos, la alerta detecta una acción malintencionada (por ejemplo, un antiguo empleado o un atacante externo).
- **Acceso desde una aplicación potencialmente dañina**: esta alerta se desencadena cuando una aplicación potencialmente dañina se utiliza para tener acceso a la base de datos. En algunos casos, la alerta detecta la realización de pruebas de seguridad. En otros casos, la alerta detecta un ataque que se realiza con herramientas de ataque comunes.
- **Ataques por fuerza bruta a las credenciales de Azure Database for PostgreSQL**: esta alerta se desencadena cuando hay un número anormalmente elevado de inicios de sesión infructuosos con distintas credenciales. En algunos casos, la alerta detecta la realización de pruebas de seguridad. En otros casos, la alerta detecta ataques por fuerza bruta.

## <a name="next-steps"></a>Pasos siguientes

* Más información acerca de [Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-intro)
* Para obtener más información sobre los precios, vea la [página Precios de Azure Database for PostgreSQL](https://azure.microsoft.com/pricing/details/postgresql/) 
* Configuración de [Advanced Threat Protection de Azure Database for PostgreSQL](howto-database-threat-protection-portal.md) mediante Azure Portal  
