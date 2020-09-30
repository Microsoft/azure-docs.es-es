---
title: 'Azure Defender para SQL: ventajas y características'
description: Obtenga información sobre las ventajas y características de Azure Defender para SQL.
author: memildin
ms.author: memildin
ms.date: 9/12/2020
ms.topic: conceptual
ms.service: security-center
ms.custom: references_regions
manager: rkarlin
ms.openlocfilehash: e76fd33ccca73394c4ee48b3737db8b93a990659
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932444"
---
# <a name="introduction-to-azure-defender-for-sql"></a>Introducción a Azure Defender para SQL

Azure Defender para SQL incluye dos planes de Azure Defender que amplían el [paquete de seguridad de datos](../azure-sql/database/advanced-data-security.md) de Azure Security Center para proteger las bases de datos y sus datos dondequiera que se encuentren. 

## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|**Azure Defender para servidores de base de datos de Azure SQL**: disponible con carácter general (GA)<br>**Azure Defender para servidores SQL Server en máquinas**: versión preliminar|
|Precios:|Los dos planes que forman **Azure Defender para SQL** se facturan como se muestra en [la página de precios](security-center-pricing.md).|
|Versiones de SQL protegidas:|Azure SQL Database <br>Instancia administrada de Azure SQL<br>Azure Synapse Analytics (anteriormente SQL DW)<br>SQL Server (todas las versiones compatibles)|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![Sí](./media/icons/yes-icon.png) US Gov<br>![No](./media/icons/no-icon.png) China Gov, otros gobiernos|
|||

## <a name="what-does-azure-defender-for-sql-protect"></a>¿Qué protege Azure Defender para SQL?

**Azure Defender para SQL** consta de dos planes de Azure Defender independientes:

- **Azure Defender para servidores de base de datos de Azure SQL** protege:
  - [Azure SQL Database](../azure-sql/database/sql-database-paas-overview.md)
  - [Instancia administrada de Azure SQL](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md)
  - [Azure Synapse Analytics](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md)

- **Azure Defender para servidores SQL Server en máquinas (versión preliminar)** amplía las protecciones de los servidores SQL Server nativos de Azure para admitir totalmente entornos híbridos y proteger servidores SQL Server (todas las versiones compatibles) hospedados en Azure, otros entornos en la nube e incluso en máquinas locales.


## <a name="what-are-the-benefits-of-azure-defender-for-sql"></a>¿Cuáles son las ventajas de Azure Defender para SQL?

Estos dos planes incluyen funcionalidad para identificar y mitigar posibles vulnerabilidades de la base de datos, además de detectar actividades anómalas que puedan suponer alguna amenaza para las bases de datos:

- [Evaluación de vulnerabilidades](../azure-sql/database/sql-vulnerability-assessment.md): servicio de exploración que puede detectar, seguir y corregir posibles vulnerabilidades en las bases de datos. Los exámenes de evaluación proporcionan información general sobre el estado de seguridad de las máquinas SQL y detalles de las conclusiones de seguridad.

- [Advanced Threat Protection](../azure-sql/database/threat-detection-overview.md): servicio de detección que supervisa continuamente los servidores SQL Server para detectar amenazas tales como inyección de código SQL, ataques por fuerza bruta y abuso de privilegios. Este servicio proporciona alertas de seguridad orientadas a acciones en Azure Security Center con detalles de la actividad sospechosa, instrucciones sobre cómo mitigar las amenazas y opciones para continuar las investigaciones con Azure Sentinel.


## <a name="what-kind-of-alerts-does-azure-defender-for-sql-provide"></a>¿Qué tipos de alertas ofrece Azure Defender para SQL?

Las alertas de seguridad se desencadenan en los escenarios siguientes:

- **Posibles ataques por inyección de código SQL**, incluidas las vulnerabilidades detectadas cuando las aplicaciones generan una instrucción SQL defectuosa en la base de datos.
- **Acceso a bases de datos y patrones de consulta anómalos**: por ejemplo, un número anormalmente alto de intentos de inicio de sesión incorrectos con credenciales diferentes (un intento por fuerza bruta).
- **Actividad de base de datos sospechosa**: por ejemplo, un cambio en el destino de almacenamiento de exportación para una operación de importación y exportación de SQL.

Las alertas incluyen detalles acerca del incidente que las desencadenó, así como recomendaciones acerca de cómo investigar y corregir las amenazas.



## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha obtenido información sobre Azure Defender para SQL.

Para obtener material relacionado, consulte los siguientes artículos: 

- [Habilitación de Azure Defender para servidores SQL Server en máquinas virtuales](defender-for-sql-usage.md)
- [Habilitación de Azure Defender para servidores de bases de datos de SQL](../azure-sql/database/advanced-data-security.md)
- [La lista de alertas de Azure Defender para SQL](alerts-reference.md#alerts-sql-db-and-warehouse)