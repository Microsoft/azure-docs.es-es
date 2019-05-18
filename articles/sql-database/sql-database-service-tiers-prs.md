---
title: Retirada de nivel de servicio de Azure SQL Database Premium RS | Microsoft Docs
description: El nivel de servicio Premium RS se está retirando y el soporte técnico para dicho nivel está finalizando (consulte las opciones de migración).
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: carlrab
manager: craigg
ms.date: 02/07/2019
ms.openlocfilehash: 7f184178343f69f522148777752c51afc5c5dcb6
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65790408"
---
# <a name="azure-sql-database-premium-rs-service-tier-preview-is-being-retired---options-for-migration"></a>El nivel de servicio Premium RS de Azure SQL Database (versión preliminar) se está retirando: opciones de migración

En febrero de 2018, Microsoft anunció que el nivel de servicio Premium RS de Azure SQL Database no estaría disponible de manera general y no contaría con soporte técnico después del 31 de enero de 2019. Esta fecha límite de fin de soporte técnico se ha ampliado al 30 de junio de 2019. En este artículo se explican las opciones para migrar del nivel de servicio Premium RS a otro nivel de servicio. Después del 30 de junio de 2019, Microsoft migrará automáticamente las bases de datos Premium RS al nivel de servicio disponible con carácter general que más se acerque a los requisitos de rendimiento de la base de datos Premium RS.

A continuación se indican los destinos de migración y las opciones de precios que pueden resultar adecuados para los clientes del nivel Premium RS:

- Niveles de servicio de núcleo virtual

  En el artículo se describen los niveles de servicio **Uso general** y **Crítico para la empresa** en el [modelo de compra basado en núcleo virtual](sql-database-service-tiers-vcore.md). Estos dos niveles de servicio están disponibles para el público general. El modelo de compra basado en núcleos virtuales también ofrece la **hiperescala** nivel de servicio que se adapta a petición para las necesidades de su carga de trabajo con escalado automático hasta 100 TB por base de datos. El nivel de servicio Hiperescala proporciona un rendimiento de E/S comparable al nivel de servicio Premium en el [modelo de compra basado en DTU](sql-database-service-tiers-dtu.md) a un precio más cercano al del nivel de servicio Premium RS.
- Precios para desarrollo/pruebas

  [Precios para desarrollo/pruebas](https://azure.microsoft.com/pricing/dev-test/) proporciona un ahorro de hasta el 55 % en comparación con las tarifas de licencia incluida con su suscripción a Visual Studio.
- Ventaja híbrida de Azure y precios de capacidad reservada

  [Ventaja híbrida de Azure y precios de capacidad reservada](https://azure.microsoft.com/pricing/details/sql-database/) proporciona un ahorro de hasta el 80 % en comparación con las tarifas de licencia incluida. Para obtener más información sobre estas opciones, consulte los artículos sobre la [Ventaja híbrida de Azure para SQL Server](https://azure.microsoft.com/pricing/hybrid-benefit/) y la [capacidad reservada de Azure SQL Database](sql-database-reserved-capacity.md).

## <a name="act-now-to-migrate-your-premium-rs-databases-to-alternative-sql-database-service-tiers"></a>Actúe ahora para migrar las bases de datos Premium RS a niveles de servicio de SQL Database alternativos

Revise la guía de este artículo, junto con nuestros precios y nuestra documentación, para determinar los destinos de migración adecuados para sus cargas de trabajo Premium RS.

## <a name="migrate-compute-intensive-workloads-and-save"></a>Migrar cargas de trabajo de proceso intensivo y guardar

Para las cargas de trabajo de proceso intensivo Premium RS, se recomienda migrar a nuestro nivel de servicio de uso general basado en núcleos virtuales disponible con carácter general y ahorrar más en comparación con las tarifas de licencia incluida que ofrecen Ventaja híbrida de Azure para SQL Server y la capacidad reservada. Si prefiere quedarse con la opción de compra basada en DTU, puede migrar las bases de datos de proceso intensivo Premium RS a un nivel de servicio Estándar y seguir ahorrando en comparación con los precios de disponibilidad general de Premium RS (si está disponible con carácter general).

> [!WARNING]
> La migración de cargas de trabajo de Premium RES a niveles de servicio Premium basados en DTU pueden aumentar los costos mensuales frente a los precios actuales de Premium RS. Se recomienda tener en cuenta los niveles Hiperescala o Crítico para la empresa con los precios de Ventaja híbrida de Azure y capacidad reservada para conseguir unos costos similares o inferiores que los de Premium RS.

### <a name="premium-rs-databases"></a>Bases de datos Premium RS

|**Si actualmente está en el nivel…**|**Migre a un nivel basado en núcleos virtuales comparable…**|**Migre a un nivel basado en DTU comparable…**|
|---|---|---|
|Premium RS 1|1 núcleo virtual de uso general (generación 4)|Estándar 3|
|Premium RS 2|2 núcleos virtuales de uso general (generación 4)|Estándar 4|
|Premium RS 4|4 núcleos virtuales de uso general (generación 4)|Estándar 6|
|Premium RS 6|6 núcleos virtuales de uso general (generación 4)|Estándar 7|

### <a name="premium-rs-pools"></a>Grupos Premium RS

|**Si actualmente está en el nivel…**|**Migre a un nivel basado en núcleos virtuales comparable…**|**Migre a un nivel basado en DTU comparable…**|
|---|---|---|
|125 DTU del grupo Premium RS|1 núcleo virtual de uso general (generación 4)|100 eDTU del grupo Estándar|
|250 DTU del grupo Premium RS|2 núcleos virtuales de uso general (generación 4)|250 eDTU del grupo Estándar|
|500 DTU del grupo Premium RS|4 núcleos virtuales de uso general (generación 4)|500 eDTU del grupo Estándar|
|1000 DTU del grupo Premium RS|8 núcleos virtuales de uso general (generación 4)|1000 eDTU del grupo Estándar|

## <a name="optimize-savings-and-performance-for-your-io-intensive-workloads"></a>Optimizar el ahorro y el rendimiento para las cargas de trabajo intensivas de E/S

Se recomienda migrar las bases de datos únicas con un tráfico intensivo de E/S a nuestro nivel Hiperescala basado en núcleos virtuales, actualmente en versión preliminar, y los grupos de bases de datos con un tráfico intensivo de E/S a nuestro nivel Crítico para la empresa disponible con carácter general, para conseguir una combinación óptima de costo y rendimiento.  Las siguientes opciones basadas en núcleos virtuales mantendrán o mejorarán el rendimiento actual y pueden ayudarle a ahorrar dinero si se combinan con los precios de Ventaja híbrida de Azure y capacidad reservada.

|**Si actualmente está en el nivel…**|**Migre a un nivel basado en núcleos virtuales comparable…**|**Migre a un nivel basado en DTU comparable…**|
|---|---|---|
|Premium RS 1| Núcleo virtual de 1 a gran escala (Gen4) o empresariales críticos de 1 núcleo virtual (Gen4)|Premium 1|
|Premium RS 2| Hiperescala 2 núcleos virtuales (Gen4) o negocio críticos 2 núcleos virtuales (Gen4|Premium 2|
|Premium RS 4| Hiperescala 4 núcleos virtuales (Gen4) o negocio críticas 4 núcleos virtuales (Gen4)|Premium 4
|Premium RS 6| 6 de hiperescala núcleos virtuales (Gen4) o empresariales críticas 6 núcleos virtuales (Gen4)|Premium 6|

|**Si actualmente está en el nivel…**|**Migre a un nivel basado en núcleos virtuales comparable…**|**Migre a un nivel basado en DTU comparable…**|
|---|---|---|
|125 DTU del grupo Premium RS|2 núcleos virtuales críticos para la empresa (generación 4)|125 eDTU del grupo Premium|
|250 DTU del grupo Premium RS|2 núcleos virtuales críticos para la empresa (generación 4)|250 eDTU del grupo Premium|
|500 DTU del grupo Premium RS|4 núcleos virtuales críticos para la empresa (generación 4)|500 eDTU del grupo Premium|
|1000 DTU del grupo Premium RS|8 núcleos virtuales críticos para la empresa (generación 4)|1000 eDTU del grupo Premium|

## <a name="take-advantage-of-our-new-offers"></a>Aprovechar las ventajas de las nuevas ofertas

Nuestros niveles de servicio del modelo de compra basado en núcleo virtual son aptos para las ofertas especiales que le pueden ayudar a ahorrar hasta un 80 % en comparación con los precios de licencia incluida. Use sus licencias de SQL Server Standard o Enterprise Edition con Software Assurance activo para ahorrar hasta un 55 % en comparación con los precios de licencia incluida con [Ventaja híbrida de Azure para SQL Server](https://azure.microsoft.com/pricing/hybrid-benefit/). Puede combinar el precio de la ventaja híbrida con el de la [capacidad reservada de Azure SQL Database](sql-database-reserved-capacity.md) y ahorrar hasta un 80 % al confirmar por adelantado un período de uno o tres años.  Active ambas ventajas hoy mismo desde Azure Portal.

Si tiene alguna pregunta o preocupación respecto a este cambio o si necesita ayuda con la migración, póngase en contacto con [Microsoft](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).

## <a name="migration-from-a-premium-rs-service-tier-to-a-service-tier-in-either-the-dtu-or-the-vcore-model"></a>Migración de un nivel de servicio Premium RS a un nivel de servicio del modelo basado en DTU o núcleos virtuales

### <a name="migration-of-a-database"></a>Migración de una base de datos

La migración de una base de datos de un nivel de servicio Premium RS a un nivel de servicio del modelo basado en DTU o en núcleos virtuales es similar a la actualización o degradación entre los niveles de servicio en el nivel de servicio Premium RS.

### <a name="using-database-copy-to-convert-a-premium-rs-database-to-a-dtu-based-or-vcore-based-database"></a>Uso de una copia de base de datos para convertir una base de datos Premium RS a una base de datos basada en DTU o en núcleos virtuales

Puede copiar cualquier base de datos con un tamaño de proceso de Premium RS en una base de datos que tenga un tamaño de proceso basado en DTU o en núcleos virtuales sin restricciones o secuencias especiales, siempre y cuando el tamaño de proceso de destino admita el tamaño máximo de base de datos de la base de datos de origen. La copia de base de datos crea una instantánea de los datos a partir de la hora de inicio de la operación de copia y no realiza la sincronización de datos entre el origen y el destino.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre las opciones de tamaños de proceso y de tamaños de almacenamiento específicas que hay disponibles para las bases de datos únicas, consulte [Límites de recursos basados en núcleos virtuales de SQL Database para bases de datos únicas](sql-database-vcore-resource-limits-single-databases.md).
- Para más información sobre las opciones de tamaño de proceso y de tamaño de almacenamiento específicas que hay disponibles para los grupos elásticos, consulte [SQL Database vCore-based resource limits for elastic pools](sql-database-vcore-resource-limits-elastic-pools.md) (Límites de recursos basados en núcleos virtuales de Azure SQL Database para grupos elásticos).
