---
title: Bases de datos de Azure SQL únicas| Microsoft Docs
description: Obtenga información acerca de la base de datos única en Azure SQL Database
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
manager: craigg
ms.date: 04/08/2019
ms.openlocfilehash: 36b1fb96ac1dd89375588a65063ce729f6ac825d
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65794388"
---
# <a name="what-is-a-single-database-in-azure-sql-database"></a>Bases de datos únicas en Azure SQL Database

La opción de implementación de una base de datos única crea una base de datos en Azure SQL Database con su propio conjunto de recursos y se administra a través de un servidor de SQL Database. Con una base de datos única, cada base de datos está aislada de las demás y es portátil, ya que cada una tiene su propio nivel de servicio dentro del [modelo de compra basado en DTU](sql-database-service-tiers-dtu.md) o el [modelo de compra basado en núcleo virtual](sql-database-service-tiers-vcore.md) y un tamaño de proceso garantizado.

> [!IMPORTANT]
> La base de datos única es una de las tres opciones de implementación de Azure SQL Database. Las otras dos son los [grupos elásticos](sql-database-elastic-pool.md) y la [instancia administrada](sql-database-managed-instance.md).
> [!NOTE]
> Para obtener un glosario de términos en Azure SQL Database, consulte el [glosario de términos para SQL Database](sql-database-glossary-terms.md).

## <a name="dynamic-scalability"></a>Escalabilidad dinámica

Puede crear su primera aplicación en una base de datos pequeña a bajo costo en el nivel de proceso sin servidor (versión preliminar) o un tamaño de proceso pequeña en el nivel de proceso aprovisionada. Cambia el [nivel de servicio o proceso](sql-database-single-database-scale.md) manualmente o mediante programación en cualquier momento para satisfacer las necesidades de su solución. El rendimiento se puede ajustar sin que la aplicación o los clientes sufran ningún tipo de inactividad. La escalabilidad dinámica permite que una base de datos responda transparentemente a los requisitos de recursos, que cambian con rapidez, y le permite pagar solo por los recursos que necesite cuando los necesite.

## <a name="single-databases-and-elastic-pools"></a>Bases de datos únicas y grupos elásticos

Una base de datos única se puede mover dentro o fuera de un [grupo elástico](sql-database-elastic-pool.md) para que compartan recursos. Para muchas empresas y aplicaciones, poder crear bases de datos individuales y aumentar o reducir el rendimiento a petición es suficiente, especialmente si los patrones de uso son relativamente predecibles. Pero si dichos patrones son impredecibles, pueden dificultar la administración de los costos y del modelo de negocio. Los grupos elásticos están diseñados para solucionar este problema. El concepto es sencillo. Se asignan los recursos de rendimiento a un grupo, en lugar a una base de datos individual y se paga por los recursos de rendimiento colectivos del grupo, no por el rendimiento de la base de datos única.

## <a name="monitoring-and-alerting"></a>Supervisión y alertas

Utilice las herramientas integradas de [supervisión de rendimiento](sql-database-performance.md) y de [alertas](sql-database-insights-alerts-portal.md) en combinación con las clasificaciones del rendimiento. Uso de estas herramientas, puede evaluar rápidamente el impacto de escalar verticalmente en función de su suscripción actual o se proyecta necesidades de rendimiento. Además, SQL Database puede [emitir métricas y registros de diagnóstico](sql-database-metrics-diag-logging.md) para facilitar la supervisión.

## <a name="availability-capabilities"></a>Funcionalidades de disponibilidad

Bases de datos únicas, los grupos elásticos y las instancias administradas proporcionan muchas características de disponibilidad. Para obtener más información al respecto, consulte las [características de disponibilidad](sql-database-technical-overview.md#availability-capabilities).

## <a name="transact-sql-differences"></a>Diferencias de Transact-SQL

La mayoría de las características de Transact-SQL que usan las aplicaciones se admiten en Microsoft SQL Server y Azure SQL Database. Por ejemplo, los componentes principales de SQL, como tipos de datos, operadores, funciones de cadena, aritméticas, lógicas y de cursor, funcionan de la misma forma en SQL Server y SQL Database. Pero hay algunas diferencias de T-SQL en los elementos DDL (lenguaje de definición de datos) y DML (lenguaje de manipulación de datos) que generan instrucciones y consultas de T-SQL que solo se admiten parcialmente (como se describe más adelante en este artículo).
Además, hay algunas características y sintaxis que no se admiten en absoluto porque Azure SQL Database se diseñó para aislar las características de cualquier dependencia de la base de datos maestra y el sistema operativo. Por eso, muchas actividades de nivel de servidor no son apropiadas para SQL Database. Las instrucciones y opciones de T-SQL no están disponibles si configuran opciones de nivel de servidor y componentes del sistema operativo, o especifican la configuración del sistema de archivos. Cuando se necesitan estas capacidades, suele estar disponible una alternativa adecuada en alguna otra manera de SQL Database o de otra función o servicio de Azure.

Para obtener más información, consulte [Resolución de diferencias de Transact-SQL durante la migración a SQL Database](sql-database-transact-sql-information.md).

## <a name="security"></a>Seguridad

SQL Database proporciona varias [características integradas de seguridad y cumplimiento](sql-database-security-overview.md) que facilitan que su aplicación cumpla los distintos requisitos de seguridad y cumplimiento normativo.

> [!IMPORTANT]
> Azure SQL Database (todas las opciones de implementación), ha sido certificada con una serie de estándares de cumplimiento. Para obtener más información, consulte el [Microsoft Azure Trust Center](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942) donde puede encontrar la lista más actualizada de certificaciones de cumplimiento de la base de datos SQL.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener una introducción rápida a las bases de datos únicas, comience por la [guía de inicio rápido](sql-database-single-database-quickstart-guide.md).
- Para obtener información sobre cómo migrar una base de datos de SQL Server a Azure, vea [Migración a Azure SQL Database](sql-database-single-database-migrate.md).
- Para obtener información sobre las características admitidas, consulte el [artículo que trata sobre dicho tema](sql-database-features.md).