---
title: ¿Qué es el servicio Azure SQL Database?
description: 'Obtenga una introducción a SQL Database: detalles técnicos y funcionalidades del sistema de administración de bases de datos relacionales (RDBMS) de Microsoft en la nube.'
keywords: introducción a sql,introducción a sql,qué es base de datos sql
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: sqldbrb=3
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 09/21/2020
ms.openlocfilehash: 38f52178ec9c736f3ee51839678401753365d48d
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907058"
---
# <a name="what-is-azure-sql-database"></a>¿Qué es Azure SQL Database?
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Azure SQL Database es un motor de base de datos de plataforma como servicio (PaaS) totalmente administrado que se encarga de la mayoría de las funciones de administración de bases de datos, como actualizar, aplicar revisiones, crear copias de seguridad y supervisar sin intervención del usuario. Azure SQL Database se ejecuta siempre en la última versión estable del motor de base de datos de SQL Server y en un sistema operativo revisado con el 99,99 % de disponibilidad. Las capacidades de PaaS que están integradas en Azure SQL Database permiten centrarse en las actividades de administración y optimización de bases de datos específicas del dominio que son críticas para el negocio.

Con Azure SQL Database, puede crear una capa de almacenamiento de datos de gran rendimiento y disponibilidad para las aplicaciones y las soluciones de Azure. SQL Database puede ser la opción adecuada para una variedad de aplicaciones modernas en la nube, porque le permite procesar tanto datos relacionales como [estructuras no relacionales](../multi-model-features.md), por ejemplo, grafos, JSON, elementos espaciales y XML.

Azure SQL Database se basa en la versión estable más reciente del [motor de base de datos de Microsoft SQL Server](https://docs.microsoft.com/sql/sql-server/sql-server-technical-documentation?toc=/azure/sql-database/toc.json). Puede usar características avanzadas de procesamiento de consultas, como las [tecnologías en memoria de alto rendimiento](../in-memory-oltp-overview.md) y el [procesamiento de consultas inteligente](https://docs.microsoft.com/sql/relational-databases/performance/intelligent-query-processing?toc=/azure/sql-database/toc.json). De hecho, las funcionalidades más recientes de SQL Server se publican primero en SQL Database y, después, en el propio SQL Server. Las funcionalidades de SQL Server más recientes se obtienen sin costo alguno mediante revisiones o actualizaciones, y se han probado en millones de bases de datos. 

SQL Database le permite definir y escalar fácilmente el rendimiento de dos modelos de compra diferentes: un [modelo de compra basado en el núcleo virtual](service-tiers-vcore.md) y un [modelo de compra basado en DTU](service-tiers-dtu.md). SQL Database es un servicio totalmente administrado que ofrece alta disponibilidad, copias de seguridad y otras operaciones de mantenimiento comunes. Microsoft controla todas las revisiones y actualizaciones del código de sistema operativo y SQL. No hace falta administrar la infraestructura subyacente.

Si no está familiarizado con Azure SQL Database, consulte el vídeo de *introducción a Azure SQL Database* en la detallada [serie de vídeos sobre Azure SQL](https://channel9.msdn.com/Series/Azure-SQL-for-Beginners?WT.mc_id=azuresql4beg_azuresql-ch9-niner):
> [!VIDEO https://channel9.msdn.com/Series/Azure-SQL-for-Beginners/Azure-SQL-Database-Overview-7-of-61/player]

## <a name="deployment-models"></a>Modelos de implementación

Azure SQL Database proporciona las opciones de implementación siguientes para una base de datos:

- Una [base de datos única](single-database-overview.md) representa una base de datos aislada totalmente administrada. Puede usar esta opción si tiene aplicaciones y microservicios modernos en la nube que necesitan un único origen de datos confiable. Una base de datos única es similar a una [base de datos independiente](https://docs.microsoft.com/sql/relational-databases/databases/contained-databases?toc=/azure/sql-database/toc.json) del [motor de base de datos de SQL Server](https://docs.microsoft.com/sql/sql-server/sql-server-technical-documentation?toc=/azure/sql-database/toc.json).
- El [grupo elástico](elastic-pool-overview.md) es una colección de [bases de datos únicas](single-database-overview.md) con un conjunto compartido de recursos, como la CPU o la memoria. Las bases de datos únicas se pueden mover dentro y fuera de un grupo elástico.

> [!IMPORTANT]
> Para comprender las diferencias en las características de SQL Database y SQL Server, así como las que existen entre distintas opciones de Azure SQL Database, vea las [características de SQL Database](features-comparison.md).

SQL Database ofrece un rendimiento predecible con varios tipos de recursos, niveles de servicio y tamaños de proceso. Proporciona escalabilidad dinámica sin tiempo de inactividad, optimización inteligente integrada, escalabilidad y disponibilidad globales, y opciones de seguridad avanzadas. Estas funcionalidades le permiten centrarse en el desarrollo rápido de aplicaciones y en reducir el plazo de acceso al mercado, y olvidarse de la administración de máquinas virtuales e infraestructuras. SQL Database se encuentra actualmente en 38 centros de datos de todo el mundo, por lo que la base de datos se puede ejecutar en el que se encuentre más cerca.

## <a name="scalable-performance-and-pools"></a>Grupos y rendimiento escalable

Puede definir la cantidad de recursos asignados. 
- Con las bases de datos únicas, cada base de datos está aislada de otras y es portátil. Cada una tiene su propia cantidad garantizada de recursos de proceso, memoria y almacenamiento. La cantidad de recursos asignados a la base de datos está dedicada a esa base de datos y no se comparte con otras bases de datos de Azure. Puede [escalar y reducir verticalmente lo recursos](single-database-scale.md) de base de datos de forma dinámica. La opción de base de datos única proporciona diferentes recursos de proceso, memoria y almacenamiento para diferentes necesidades. Por ejemplo, puede obtener de 1 a 80 núcleos virtuales, o de 32 GB a 4 TB. El [nivel de servicio de hiperescala](service-tier-hyperscale.md) para bases de datos únicas permite escalar a 100 TB, con capacidades rápidas de copia de seguridad y restauración.
- Gracias a los grupos elásticos, puede asignar recursos que comparten todas las bases de datos del grupo. Puede crear una base de datos o mover las bases de datos únicas existentes a un grupo de recursos para así maximizar el uso de recursos y ahorrar dinero. Esta opción también ofrece la posibilidad de [escalar y reducir verticalmente los recursos del grupo elástico](elastic-pool-scale.md) de forma dinámica.

Puede compilar su primera aplicación en una base de datos pequeña de bajo costo mensual, en el nivel de servicio de uso general. Después, puede cambiar en cualquier momento el nivel de servicio, manualmente o mediante programación, por el nivel de servicio crítico para la empresa, para satisfacer las necesidades de su solución. El rendimiento se puede ajustar sin que la aplicación o los clientes sufran ningún tipo de inactividad. La escalabilidad dinámica permite a la base de datos responder de manera transparente a los cambiantes requisitos de recursos. Solo paga por los recursos que necesite cuando los necesite.

La *escalabilidad dinámica* es diferente de la *escalabilidad automática*. El escalado automático se produce al escalarse un servicio automáticamente en función de determinados criterios, mientras la escalabilidad dinámica permite el escalado manual sin tiempo de inactividad. La opción de base de datos única admite la escalabilidad dinámica manual, pero no la escalabilidad automática. Para disfrutar de una experiencia más automatizada, considere el uso de grupos elásticos, que permiten que las bases de datos compartan recursos en un grupo en función de las necesidades individuales de las bases de datos. Otra opción es usar scripts que pueden ayudar a automatizar la escalabilidad de una base de datos única. Consulte [Uso de PowerShell para supervisar y escalar una sola base de datos SQL](scripts/monitor-and-scale-database-powershell.md) encontrará un ejemplo.

### <a name="purchasing-models"></a>Modelos de compra

SQL Database ofrece los siguientes modelos de compra:
- El [modelo de compra basado en núcleo virtual](service-tiers-vcore.md) le permite elegir el número de núcleos virtuales, la cantidad de memoria y la cantidad y velocidad del almacenamiento. El modelo de compra basado en núcleo virtual también le permite usar la [Ventaja híbrida de Azure para SQL Server](https://azure.microsoft.com/pricing/hybrid-benefit/) para ahorrar en los costos. Para más información sobre la Ventaja híbrida de Azure, consulte la sección "Preguntas frecuentes" más adelante en este artículo.
- El [modelo de compra basado en DTU](service-tiers-dtu.md) ofrece una combinación de recursos de proceso, memoria y E/S en tres niveles de servicio, para admitir cargas de trabajo de base de datos de ligeras a pesadas. Los tamaños de proceso de cada nivel ofrecen una combinación diferente de estos recursos, a los que puede agregar recursos de almacenamiento adicionales.
- El [modelo sin servidor](serverless-tier-overview.md) escala automáticamente el proceso en función de la demanda de la carga de trabajo y se factura según la cantidad de proceso usado por segundo. El nivel de proceso sin servidor también detiene automáticamente las bases de datos durante períodos de inactividad cuando solo se factura el almacenamiento, y reactiva automáticamente las bases de datos cuando se reanuda la actividad.

### <a name="service-tiers"></a>Niveles de servicio

Azure SQL Database ofrece tres niveles de servicio que están diseñados para diferentes tipos de aplicaciones:
- Nivel de servicio [De uso general/estándar](service-tier-general-purpose.md) diseñado para las cargas de trabajo comunes. Ofrece opciones de proceso y almacenamiento equilibradas basadas en el presupuesto.
- Nivel de servicio [Crítico para la empresa/Premium](service-tier-business-critical.md), diseñado para las aplicaciones de OLTP con un alto índice de transacciones y una latencia de E/S más baja. Ofrece la máxima resistencia a los errores gracias al uso de varias réplicas aisladas.
- Nivel de servicio [Hiperescala](service-tier-hyperscale.md), diseñado para bases de datos OLTP de gran tamaño y con la posibilidad de escalar automáticamente el almacenamiento y el proceso de manera fluida.    

### <a name="elastic-pools-to-maximize-resource-utilization"></a>Grupos elásticos para maximizar la utilización de los recursos

Para muchas empresas y aplicaciones, poder crear bases de datos individuales y aumentar o reducir el rendimiento a petición es suficiente, especialmente si los patrones de uso son relativamente predecibles. Pero si dichos patrones son impredecibles, pueden dificultar la administración de los costos y del modelo de negocio. Los [grupos elásticos](elastic-pool-overview.md) están diseñadas para solucionar este problema. Los recursos de rendimiento se asignan a un grupo en lugar de a una base de datos individual. Se paga por los recursos de rendimiento colectivos del grupo, no por el rendimiento de una sola base de datos.

   ![Gráfico que muestra grupos elásticos de las ediciones Básico, Estándar y Premium](./media/sql-database-paas-overview/sqldb_elastic_pools.png)

Con los grupos elásticos no es preciso centrarse en marcar el ascenso y la bajada de rendimiento de la base de datos a medida que varía la demanda de recursos. Las bases de datos agrupadas consumen los recursos de rendimiento del grupo elástico a medida que se necesiten. Las bases de datos agrupadas consumen los recursos, pero nunca superan los límites del grupo, por lo que el costo es en todo momento predecible, aunque el uso de las bases de datos individuales no lo sea.

Puede [agregar bases de datos al grupo y quitarlas de este](elastic-pool-overview.md), de modo que la aplicación se escala de unas pocas bases de datos a miles, y todo sin perder el control del presupuesto. También puede controlar el número mínimo y máximo de recursos disponibles para las bases de datos del grupo, con el fin de asegurarse de que ninguna de ellas usa todos los recursos del grupo y que todas las bases de datos agrupadas tienen un número mínimo garantizado de recursos. Para más información sobre los modelos de diseño de las aplicaciones de software como servicio (SaaS) que usan grupos elásticos, consulte [Modelos de diseño para las aplicaciones SaaS multiinquilino y SQL Database](saas-tenancy-app-design-patterns.md).

Los scripts pueden ayudarle con la supervisión y el escalado de grupos elásticos. Para obtener un ejemplo, vea [Uso de PowerShell para supervisar y escalar un grupo elástico en Azure SQL Database](scripts/monitor-and-scale-pool-powershell.md).


### <a name="blend-single-databases-with-pooled-databases"></a>Fusión de bases de datos únicas con bases de datos agrupadas

Puede fusionar bases de datos únicas con grupos elásticos y cambiar los niveles de servicio de ambos para adaptarlos a su situación. También puede combinar otros servicios de Azure con SQL Database para satisfacer sus necesidades únicas de diseño de aplicaciones, impulsar ahorros de los costos y los recursos y acceder a nuevas oportunidades de negocio.

## <a name="extensive-monitoring-and-alerting-capabilities"></a>Extensas funcionalidades de supervisión y alerta

Azure SQL Database proporciona características avanzadas de supervisión y solución de problemas que le ayudan a profundizar en las peculiaridades de la carga de trabajo. Estas características y herramientas incluyen:
 - Las capacidades de supervisión integradas que se proporcionan en la versión más reciente del motor de base de datos de SQL Server. Estas funcionalidades le permiten encontrar información en tiempo real sobre el rendimiento. 
 - Funcionalidades de supervisión PaaS que se proporcionan en Azure y que permiten supervisar y solucionar problemas de un gran número de instancias de base de datos.

El [Almacén de consultas](https://docs.microsoft.com/sql/relational-databases/performance/best-practice-with-the-query-store), una característica de supervisión integrada de SQL Server, registra el rendimiento de las consultas en tiempo real y permite identificar los posibles problemas de rendimiento y los principales consumidores de recursos. El ajuste automático y las recomendaciones le proporcionan consejos relativos a las consultas con respecto al rendimiento limitado y los índices que faltan o que están duplicados. El ajuste automático en SQL Database le permite aplicar manualmente los scripts que pueden corregir los problemas, o puede dejar que SQL Database aplique la corrección. SQL Database también puede probar y comprobar que la corrección proporciona algunas ventajas, y conservar o revertir el cambio en función del resultado. Además de las funcionalidades de Almacén de consultas y ajuste automático, también puede usar los elementos [DMV y XEvent](monitoring-with-dmvs.md) estándar para supervisar el rendimiento de la carga de trabajo.

Azure proporciona herramientas integradas de [supervisión del rendimiento](performance-guidance.md) y [alertas](alerts-insights-configure-portal.md), junto con clasificaciones de rendimiento, que le permiten supervisar fácilmente el estado de miles de bases de datos. Con estas herramientas, puede evaluar rápidamente la repercusión que tiene escalar o reducir verticalmente en función de sus necesidades de rendimiento actuales o proyectadas. Además, SQL Database puede [emitir métricas y registros de recurso](metrics-diagnostic-telemetry-logging-streaming-export-configure.md) para facilitar la supervisión. SQL Database se puede configurar para que almacene el uso de recursos, los trabajadores y sesiones, y la conectividad en uno de estos recursos de Azure:

- **Azure Storage**: para archivar grandes cantidades de datos de telemetría a un pequeño precio.
- **Azure Event Hubs**: para integrar la telemetría de SQL Database con una solución de supervisión personalizada o canalizaciones activas.
- **Registros de Azure Monitor**: para contar con una solución de supervisión integrada con funcionalidades de informes, alertas y mitigación.

![Diagrama de la arquitectura de supervisión de Azure](./media/sql-database-paas-overview/architecture.png)

## <a name="availability-capabilities"></a>Funcionalidades de disponibilidad

Azure SQL Database permite que su empresa siga funcionando durante las interrupciones. En un entorno de SQL Server tradicional, lo normal es que tenga al menos dos máquinas configuradas localmente. Estas máquinas tienen copias exactas de los datos, que se mantienen de manera sincrónica, como forma de protección frente a un error de una sola máquina o componente. Este entorno proporciona alta disponibilidad, pero no protege de la destrucción del centro de datos por un desastre natural.

La recuperación ante desastres da por supuesto que la localización geográfica de un evento catastrófico será lo suficientemente precisa para tener otra máquina u otro conjunto de máquinas con una copia alejada de los datos. En SQL Server, puede usar grupos de disponibilidad AlwaysOn que se ejecuten en modo asincrónico para obtener esta funcionalidad. Con frecuencia, la gente no quiere esperar a que se produzca la replicación tan lejos para confirmar una transacción, por lo que se pueden perder datos al realizarse conmutaciones por error no planeadas.

Las bases de datos de los niveles de servicio Premium y Crítico para la empresa ya [hacen algo parecido](high-availability-sla.md#premium-and-business-critical-service-tier-availability) a la sincronización de un grupo de disponibilidad. Las bases de datos de los niveles de servicio menores proporcionan redundancia mediante almacenamiento con un [mecanismo distinto pero equivalente](high-availability-sla.md#basic-standard-and-general-purpose-service-tier-availability). La lógica integrada ayuda a protegerse frente al error de una sola máquina. La característica de replicación geográfica activa proporciona la capacidad de protegerse frente a desastres cuando se destruye toda una región.

Azure Availability Zones intenta ofrecer protección contra la interrupción de un edificio con un solo centro de datos dentro de una región. Así, le ayuda a protegerse frente a la interrupción de alimentación eléctrica o de red en un edificio. En SQL Database, las diferentes réplicas se colocan en distintas zonas de disponibilidad (diferentes edificios, en realidad).

De hecho, el Acuerdo de Nivel de Servicio [(SLA)](https://azure.microsoft.com/support/legal/sla/) de Azure, con la tecnología de una red global de centros de datos administrados por Microsoft, ayuda a mantener las aplicaciones en funcionamiento de forma ininterrumpida. La plataforma Azure administra completamente cada base de datos y garantiza un alto porcentaje de disponibilidad de los datos sin pérdida de datos. Azure controla automáticamente la aplicación de revisiones, las copias de seguridad, la replicación, la detección de errores, los posibles errores de hardware, de software o de red subyacentes, la implementación de correcciones de errores, las conmutaciones por error, las actualizaciones de base de datos y otras tareas de mantenimiento. La disponibilidad Estándar se consigue mediante una separación de las capas de proceso y de almacenamiento. La disponibilidad Premium se logra mediante la integración de los recursos de proceso y almacenamiento en un único nodo para obtener un buen rendimiento y, después, mediante la implementación de tecnología similar a los grupos de disponibilidad AlwaysOn. Para obtener una explicación completa de las funcionalidades de alta disponibilidad de Azure SQL Database, consulte [Disponibilidad de SQL Database](high-availability-sla.md). 

Además, SQL Database proporciona características de [continuidad empresarial y escalabilidad global](business-continuity-high-availability-disaster-recover-hadr-overview.md) integradas. Entre ellas se incluyen las siguientes:

- [Copias de seguridad automáticas](automated-backups-overview.md):

  SQL Database realiza automáticamente copias de seguridad del registro de transacciones completas y diferenciales de las bases de datos para permitirle hacer una restauración a cualquier momento dado. En el caso de bases de datos únicas y bases de datos agrupadas, puede configurar SQL Database para almacenar copias de seguridad de bases de datos completas en Azure Storage para la retención de copias de seguridad a largo plazo. Para las instancias administradas, también puede realizar copias de seguridad solo de copia para la retención de copias de seguridad a largo plazo.

- [Restauraciones a un momento dado](recovery-using-backups.md):

  todas las opciones de implementación de SQL Database admiten la recuperación a un momento dado dentro del período de retención de copias de seguridad automáticas de cualquier base de datos.
- [Replicación geográfica activa](active-geo-replication-overview.md):

  las opciones de base de datos única y bases de datos agrupadas permiten configurar hasta cuatro bases de datos secundarias legibles en los mismos centros de datos de Azure o en centros de datos distribuidos globalmente. Por ejemplo, si tiene una aplicación SaaS con una base de datos de catálogos que tiene un alto volumen de transacciones simultáneas de solo lectura, use la replicación geográfica activa para habilitar el escalado horizontal de lectura global. Esto elimina los cuellos de botella en el servidor principal ocasionados por las cargas de trabajo de lectura. En el caso de las instancias administradas, use grupos de conmutación por error automática.
- [Grupos de conmutación por error automática](auto-failover-group-overview.md):

  todas las opciones de implementación de SQL Database permiten usar grupos de conmutación por error para permitir alta disponibilidad y equilibrio de carga a escala global. Esto incluye la replicación geográfica y la conmutación por error transparentes de grandes conjuntos de bases de datos, grupos elásticos e instancias administradas. Los grupos de conmutación por error permiten la creación de aplicaciones SaaS distribuidas globalmente, con una sobrecarga de administración mínima. Esto deja a SQL Database todas las tareas complejas de supervisión, enrutamiento y orquestación de conmutación por error.
- [Bases de datos con redundancia de zona](high-availability-sla.md):

  SQL Database le permite aprovisionar bases de datos de nivel Premium o Crítico para la empresa o grupos elásticos a través de varias zonas de disponibilidad. Dado que estas bases de datos y grupos elásticos tienen varias réplicas redundantes para lograr una alta disponibilidad, la colocación de estas réplicas en varias zonas de disponibilidad proporciona una mayor resistencia. Esto incluye la posibilidad de recuperarse automáticamente de los errores de escala del centro de datos, sin pérdida de datos.

## <a name="built-in-intelligence"></a>Inteligencia integrada

Con SQL Database, obtiene inteligencia integrada que le ayuda a reducir drásticamente los costos de ejecutar y administrar bases de datos y que maximiza el rendimiento y la seguridad de la aplicación. Gracias a la ejecución de millones de cargas de trabajo de clientes las 24 horas del día, SQL Database recopila y procesa una cantidad ingente de datos de telemetría y, al mismo tiempo, respeta totalmente la privacidad de los clientes. Varios algoritmos evalúan continuamente los datos de telemetría para que el servicio pueda aprender de la aplicación y adaptarse a ella.

### <a name="automatic-performance-monitoring-and-tuning"></a>Supervisión y ajuste del rendimiento automático

SQL Database proporciona información detallada de las consultas que necesita supervisar. SQL Database aprende sus patrones de base de datos y permite adaptar el esquema de la base de datos a su carga de trabajo. SQL Database proporciona [recomendaciones para el ajuste del rendimiento](database-advisor-implement-performance-recommendations.md), donde puede consultar las acciones de ajuste y aplicarlas.

Pero supervisar constantemente una base de datos es una tarea ardua y tediosa, sobre todo cuando se trabaja con muchas bases de datos. [Intelligent Insights](intelligent-insights-overview.md) realiza este trabajo automáticamente mediante la supervisión del rendimiento de SQL Database a escala. Comunica los problemas de degradación del rendimiento, identifica la causa principal de cada problema y proporciona recomendaciones para mejorar el rendimiento cuando sea posible.

La administración de un número ingente de bases de datos podría ser imposible de realizar eficazmente, ni siquiera con todas las herramientas e informes que proporcionan SQL Database y Azure. En lugar de supervisar y ajustar la base de datos manualmente, puede considerar la posibilidad de delegar algunas de estas acciones en SQL Database con el [ajuste automático](automatic-tuning-overview.md). SQL Database aplica automáticamente las recomendaciones y pruebas, y comprueba cada una de sus acciones de ajuste para garantizar que el rendimiento no deje de mejorar. De esta forma, SQL Database se adapta automáticamente a su carga de trabajo de una manera controlada y segura. El ajuste automático significa que el rendimiento de la base de datos se supervisa y se compara cuidadosamente antes y después de cada acción de ajuste. Si el rendimiento no mejora, se revierte la acción de ajuste.

Muchos de nuestros asociados que ejecutan [aplicaciones SaaS multiinquilino](saas-tenancy-app-design-patterns.md) en SQL Database confían en el ajuste automático del rendimiento para garantizar que sus aplicaciones siempre tienen un rendimiento estable y predecible. Para ellos, esta característica reduce enormemente el riesgo de que se produzca un incidente de rendimiento durante la noche. Además, puesto que una parte de su base de clientes también usa SQL Server, emplean las mismas recomendaciones de indexación que proporciona SQL Database para ayudar a sus clientes de SQL Server.

Hay dos aspectos del ajuste automático [disponibles en SQL Database](automatic-tuning-overview.md):

- **Administración automática de índices**: Identifica tanto los índices que se deben agregar a la base de datos como los que se deben quitar.
- **Corrección automática de planes**: identifica los planes problemáticos y corrige los problemas de rendimiento de los planes de SQL.

### <a name="adaptive-query-processing"></a>Procesamiento de consultas adaptable

Puede usar el [procesamiento adaptable de consultas](/sql/relational-databases/performance/intelligent-query-processing), lo que incluye la ejecución intercalada de funciones con valores de tabla de varias instrucciones, comentarios de concesión de memoria del modo por lotes y combinaciones adaptables del modo por lotes. Cada una de estas características del procesamiento adaptable de consultas aplica técnicas de "aprendizaje y adaptación" similares, lo que ayuda a solucionar cualquier incidencia de rendimiento relacionada con problemas de optimización de consultas históricamente intrincados.

## <a name="advanced-security-and-compliance"></a>Conformidad y seguridad avanzada

SQL Database proporciona varias [características integradas de seguridad y cumplimiento](../../active-directory/identity-protection/security-overview.md) que facilitan que su aplicación cumpla los distintos requisitos de seguridad y cumplimiento normativo.

> [!IMPORTANT]
> Azure SQL Database (todas las opciones de implementación) ha recibido la certificación de Microsoft de cumplimiento de diversos estándares. Para más información, visite el [Centro de confianza de Microsoft Azure](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942), donde encontrará la lista más reciente de certificaciones de cumplimiento de SQL Database.

### <a name="advance-threat-protection"></a>Protección contra amenazas avanzada

Azure Defender para SQL es un paquete unificado de funcionalidades avanzadas de seguridad de SQL. Incluye funcionalidad para administrar los puntos vulnerables de una base de datos y detectar actividades anómalas que puedan indicar una amenaza para dicha base de datos. Proporciona una ubicación única para habilitar y administrar estas funcionalidades.

- [Evaluación de vulnerabilidades](sql-vulnerability-assessment.md):

  Este servicio puede detectar y realizar un seguimiento de posibles vulnerabilidades de la base de datos, así como ayudarle a corregirlas. Permite ver el estado de la seguridad e incluye los pasos necesarios para resolver problemas de seguridad y mejorar las defensas de cualquier base de datos.
- [Detección de amenazas](threat-detection-configure.md):

  esta característica detecta actividades anómalas que indiquen intentos inusuales y potencialmente perjudiciales de acceder a una base de datos o de aprovechar sus vulnerabilidades. Supervisa constantemente una base de datos para detectar actividades sospechosas y proporciona de forma inmediata alertas de seguridad de posibles puntos vulnerables, ataques por inyección de código SQL y patrones anómalos de acceso a las bases de datos. Las alertas de detección de amenazas proporcionan detalles de la actividad sospechosa y recomiendan acciones para investigar y mitigar la amenaza.

### <a name="auditing-for-compliance-and-security"></a>Auditoría de seguridad y cumplimiento

La [auditoría](../../azure-sql/database/auditing-overview.md) realiza un seguimiento de eventos de bases de datos y los escribe en un registro de auditoría de su cuenta de Azure Storage. La auditoría puede ayudarle a mantener el cumplimiento de normativas, comprender la actividad de las bases de datos y conocer las discrepancias y anomalías que pueden indicar problemas en el negocio o infracciones de seguridad sospechosas.

### <a name="data-encryption"></a>Cifrado de datos

SQL Database le ayuda a proteger los datos mediante cifrado. Para los datos en movimiento, usa [seguridad de la capa de transporte](https://support.microsoft.com/kb/3135244). Para los datos en reposo, usa [cifrado de datos transparente](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql). Para los datos en uso, emplea [Always Encrypted](https://docs.microsoft.com/sql/relational-databases/security/encryption/always-encrypted-database-engine).

### <a name="data-discovery-and-classification"></a>Clasificación y detección de datos

La [clasificación y detección de datos](data-discovery-and-classification-overview.md) proporciona funcionalidades integradas en Azure SQL Database para detectar, clasificar, etiquetar y proteger la información confidencial de las bases de datos. Se puede usar para proporcionar visibilidad sobre el estado de clasificación de una base de datos y para realizar un seguimiento del acceso a información confidencial dentro de la base de datos y más allá de sus límites.

### <a name="azure-active-directory-integration-and-multi-factor-authentication"></a>Integración de Azure Active Directory y autenticación multifactor

SQL Database permite administrar centralmente las identidades de usuario de base de datos y otros servicios de Microsoft con la [integración de Azure Active Directory](authentication-aad-overview.md). Esta funcionalidad simplifica la administración de permisos y mejora la seguridad. Azure Active Directory admite [autenticación multifactor](authentication-mfa-ssms-overview.md) para aumentar la seguridad de los datos y de la aplicación, al tiempo que admite un proceso de inicio de sesión único.

## <a name="easy-to-use-tools"></a>Herramientas fáciles de usar

SQL Database facilita la creación y el mantenimiento de aplicaciones y aumenta su productividad. SQL Database le permite centrarse en lo que mejor hace: crear magníficas aplicaciones. En SQL Database, puede realizar labores de administración y desarrollo mediante las herramientas y los conocimientos que ya posee.

|Herramienta|Descripción|
|:---|:---|
|[Portal de Azure](https://portal.azure.com/)|aplicación web para administrar todos los servicios de Azure.|
|[Azure Data Studio](/sql/azure-data-studio/)|herramienta de base de datos multiplataforma que se ejecuta en Windows, macOS y Linux.|
|[SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)|aplicación cliente gratuita que se puede descargar para administrar cualquier infraestructura de SQL, desde SQL Server hasta SQL Database.|
|[Herramientas de datos de SQL Server](https://docs.microsoft.com/sql/ssdt/download-sql-server-data-tools-ssdt)|aplicación cliente gratuita que se puede descargar para desarrollar bases de datos relacionales de SQL Server, bases de datos de Azure SQL Database, paquetes de Integration Services, modelos de datos de Analysis Services e informes de Reporting Services.|
|[Visual Studio Code](https://code.visualstudio.com/docs)|editor de código abierto gratuito que se puede descargar para Windows, macOS y Linux. Admite extensiones, como la [extensión mssql](https://aka.ms/mssql-marketplace) para consultar Microsoft SQL Server, Azure SQL Database y Azure Synapse Analytics (antes Azure SQL Data Warehouse).|

SQL Database admite la compilación de aplicaciones con Python, Java, Node.js, PHP, Ruby y .NET en macOS, Linux y Windows. SQL Database admite las mismas [bibliotecas de conexiones](connect-query-content-reference-guide.md#libraries) como SQL Server.

[!INCLUDE [sql-database-create-manage-portal](../includes/sql-database-create-manage-portal.md)]

## <a name="sql-database-frequently-asked-questions"></a>Preguntas frecuentes sobre SQL Database

### <a name="can-i-control-when-patching-downtime-occurs"></a>¿Puedo controlar cuando se produce el tiempo de inactividad de la aplicación de revisiones?

No. El impacto de la aplicación de revisiones no suele ser perceptible si [usa una lógica de reintento](develop-overview.md#resiliency) en la aplicación. Para más información, consulte [Planeación de los eventos de mantenimiento en Azure SQL Database](planned-maintenance.md).



## <a name="engage-with-the-sql-server-engineering-team"></a>Contactar con el equipo de ingeniería de SQL Server

- [DBA Stack Exchange](https://dba.stackexchange.com/questions/tagged/sql-server): formule preguntas sobre administración de base de datos.
- [Stack Overflow](https://stackoverflow.com/questions/tagged/sql-server): formule preguntas sobre desarrollo.
- [Página de preguntas y respuestas de Microsoft](https://docs.microsoft.com/answers/topics/azure-synapse-analytics.html): realice preguntas técnicas.
- [Comentarios](https://aka.ms/sqlfeedback): comunique errores y solicite características.
- [Reddit](https://www.reddit.com/r/SQLServer/): debata sobre SQL Server.

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [página de precios](https://azure.microsoft.com/pricing/details/sql-database/) para ver comparativas y calculadoras de los costos tanto de las bases de datos únicas como de los grupos elásticos.
- Consulte estos inicios rápidos para comenzar:

  - [Creación de una base de datos en Azure Portal](single-database-create-quickstart.md)  
  - [Creación de una base de datos con la CLI de Azure](az-cli-script-samples-content-guide.md)
  - [Creación de una base de datos mediante PowerShell](powershell-script-content-guide.md)

- Para obtener ejemplos de la CLI de Azure y de PowerShell, consulte:
  - [Ejemplos de la CLI de Azure para SQL Database](az-cli-script-samples-content-guide.md)
  - [Ejemplos de Azure PowerShell para SQL Database](powershell-script-content-guide.md)

- Para información sobre las nuevas funcionalidades a medida que se anuncian, consulte la [Hoja de ruta de Azure para SQL Database](https://azure.microsoft.com/roadmap/?category=databases).
- Consulte el [blog de Azure SQL Database](https://azure.microsoft.com/blog/topics/database), donde los miembros del equipo de producto de SQL Server escriben entradas acerca de las noticias y características de SQL Database.

