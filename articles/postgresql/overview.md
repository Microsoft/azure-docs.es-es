---
title: Qué es Azure Database for PostgreSQL
description: Proporciona información general sobre el servicio de base de datos relacional de Azure Database for PostgreSQL en el contexto de un servidor flexible.
author: sunilagarwal
ms.author: sunila
ms.custom: mvc
ms.service: postgresql
ms.topic: overview
ms.date: 09/21/2020
ms.openlocfilehash: 5945f50ada9af6a8d117d3d773ebeae48d5f4085
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90903750"
---
# <a name="what-is-azure-database-for-postgresql"></a>¿Qué es Azure Database for PostgreSQL?

Azure Database for PostgreSQL es un servicio de base de datos relacional de la nube de Microsoft basado en el motor de base de datos de [PostgreSQL Community Edition](https://www.postgresql.org/) (disponible bajo la licencia GPLv2). Azure Database for PostgreSQL ofrece:

- Alta disponibilidad integrada
- Protección de datos mediante copias de seguridad automáticas y restauración a un momento dado durante un máximo de 35 días.
- Mantenimiento automatizado para el hardware subyacente, el sistema operativo y el motor de base de datos para mantener el servicio seguro y actualizado.
- Rendimiento predecible, con precios de pago por uso inclusivos.
- Escalado elástico en cuestión de segundos.
- Seguridad de nivel empresarial y cumplimiento normativo líder en el mercado para proteger la información confidencial en reposo y en movimiento.
- Supervisión y automatización para simplificar la administración y la supervisión de las implementaciones a gran escala.
- Experiencia de soporte técnico líder del sector.

 :::image type="content" source="./media/overview/overview-what-is-azure-postgres.png" alt-text="Azure Database para PostgreSQL":::

Estas funcionalidades no requieren casi ninguna tarea de administración y todas se proporcionan sin ningún costo adicional. Le permiten centrarse en el desarrollo rápido de aplicaciones y en reducir el plazo de acceso al mercado, en lugar de tener que dedicar tiempo y recursos a la administración tanto de máquinas virtuales como de infraestructura. Además, puede seguir desarrollando su aplicación con las herramientas y la plataforma de código abierto de su elección, y entregar con la velocidad y eficacia que exige su negocio sin tener que aprender nuevas habilidades.

## <a name="deployment-models"></a>Modelos de implementación

Azure Database for PostgreSQL, con tecnología de PostgreSQL Community Edition, está disponible en tres modos de implementación:

- Servidor único
- Servidor flexible (versión preliminar)
- Hiperescala (Citus)

### <a name="azure-database-for-postgresql---single-server"></a>Azure Database for PostgreSQL: Un solo servidor

Servidor único de Azure Database for PostgreSQL es un servicio de base de datos totalmente administrado con requisitos mínimos para las personalizaciones de base de datos. La plataforma de servidor único está diseñada para administrar la mayoría de las funciones de administración de bases de datos, como las de aplicación de revisión, copias de seguridad, alta disponibilidad y seguridad con una configuración y un control mínimos para el usuario. La arquitectura está optimizada para una alta disponibilidad integrada, ya que ofrece una disponibilidad del 99,99 % en una zona de disponibilidad única. Es compatible con la versión Community de PostgreSQL 9.5, 9.6, 10 y 11. El servicio está disponible con carácter general hoy en muchas [regiones de Azure](https://azure.microsoft.com/global-infrastructure/services/).

La opción de implementación Un solo servidor ofrece tres planes de tarifa: Básico, De uso general y Optimizado para memoria. Cada plan ofrece capacidades de recursos diferente para admitir sus cargas de trabajo de la base de datos. Puede compilar su primera aplicación en una base de datos pequeña por poco dinero al mes y, después, ajustar la escala para satisfacer las necesidades de la solución. La escalabilidad dinámica permite a la base de datos responder de manera transparente a los cambiantes requisitos de recursos. Solo paga por los recursos que necesite y cuando los necesite. Vea los [Planes de tarifa](https://docs.microsoft.com/azure/postgresql/concepts-pricing-tiers) para obtener más información.

Los servidores únicos son los más adecuados para las aplicaciones nativas en la nube diseñadas para controlar la aplicación de revisiones automatizadas sin necesidad de un control granular sobre la programación de aplicación de revisiones ni de una configuración personalizada de PostgreSQL.

Para obtener información general detallada sobre el modo de implementación de servidor único, consulte [Información general de servidor único](./overview-single-server.md).

### <a name="azure-database-for-postgresql---flexible-server-preview"></a>Servidor flexible (versión preliminar) de Azure Database for PostgreSQL

Servidor flexible de Azure Database for PostgreSQL es un servicio de base de datos totalmente administrado diseñado para proporcionar un control más granular y una mayor flexibilidad sobre las funciones de administración de bases de datos y las opciones de configuración. En general, el servicio proporciona mayor flexibilidad y personalizaciones en función de los requisitos del usuario. La arquitectura de servidor flexible permite a los usuarios optar por la alta disponibilidad dentro de una única zona de disponibilidad o entre varias. El servidor flexible proporciona mejores controles de optimización de costos con la capacidad de iniciar o detener el servidor y niveles de proceso flexibles, que son idóneos para cargas de trabajo que no necesitan una capacidad de proceso completa de forma continua. El servicio admite actualmente la versión Community de PostgreSQL 11 y 12 con planes para agregar versiones más recientes próximamente. El servicio está disponible actualmente en versión preliminar pública en muchas regiones de Azure.

Los servidores flexibles son idóneos para

- Desarrollo de aplicaciones que necesitan un mejor control y personalizaciones.
- Controles de optimización de costos con capacidad para detener o iniciar un servidor.
- Alta disponibilidad con redundancia de zona
- Ventanas de mantenimiento administradas
  
Para obtener información general detallada sobre el modo de implementación de servidor flexible, consulte la [Información general de servidor flexible](./flexible-server/overview.md).

### <a name="azure-database-for-postgresql--hyperscale-citus"></a>Hiperescala (Citus) de Azure Database for PostgreSQL

La opción Citus escala horizontalmente las consultas entre varias máquinas mediante particionamiento. Su motor de consultas paraleliza las consultas SQL entrantes en estos servidores para agilizar las respuestas en conjuntos de datos grandes. Sirve aplicaciones que requieren mayor escala y mejor rendimiento, por lo general cargas de trabajo que se aproximan a los 100 GB de datos (o que ya los superan).

La opción de implementación Citus ofrece:

- Escalado horizontal entre varias máquinas mediante particionamiento
- Paralelización de consultas entre estos servidores, lo que agiliza las respuestas en conjuntos de datos grandes
- Una excelente compatibilidad con aplicaciones de varios inquilinos, análisis operativos en tiempo real y cargas de trabajo transaccionales de alto rendimiento
  
Las aplicaciones compiladas para PostgreSQL puede ejecutar consultas distribuidas en Citus con las [bibliotecas de conexiones](https://docs.microsoft.com/azure/postgresql/concepts-connection-libraries) estándar y unos cambios mínimos.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre los tres modos de implementación de Azure Database for PostgreSQL y elija las opciones adecuadas en función de sus necesidades.

- [Servidor único](./overview-single-server.md)
- [Servidor flexible](./flexible-server/overview.md)
- Hiperescala (Citus)
