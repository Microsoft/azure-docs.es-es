---
title: Diseño de servicios disponibles de forma global
description: Obtenga información sobre el diseño de aplicaciones para servicios de alta disponibilidad con Azure SQL Database.
keywords: recuperación ante desastres en la nube, soluciones de recuperación ante desastres, copia de seguridad de datos de aplicación, planificación de continuidad del negocio
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: carlrab
ms.date: 07/28/2020
ms.openlocfilehash: a23330bb00fb06a3ed9d3dfe28666e8f27dae4fa
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87405048"
---
# <a name="designing-globally-available-services-using-azure-sql-database"></a>Diseño de servicios disponibles globalmente con Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Al crear e implementar servicios en la nube con Azure SQL Database, use los [grupos de conmutación por error automática](active-geo-replication-overview.md) y la [replicación geográfica activa](auto-failover-group-overview.md) para proporcionar resistencia frente a interrupciones regionales y errores graves. La misma característica permite crear aplicaciones distribuidas globalmente optimizadas para el acceso local a los datos. En este artículo se describen patrones de aplicación comunes, incluidas las ventajas y desventajas de cada opción.

> [!NOTE]
> Si usa bases de datos y grupos elásticos de nivel Premium o Crítico para la empresa, puede hacerlos resistentes a las interrupciones regionales mediante su conversión a una configuración de implementación con redundancia de zona. Consulte [Bases de datos con redundancia de zona](high-availability-sla.md).  

## <a name="scenario-1-using-two-azure-regions-for-business-continuity-with-minimal-downtime"></a>Escenario 1: Utilizar dos regiones de Azure para la continuidad empresarial con un tiempo de inactividad mínimo

En este escenario, las aplicaciones tienen las siguientes características:

* La aplicación está activa en una región de Azure.
* Todas las sesiones de base de datos requieren acceso de lectura y escritura (RW) a los datos.
* El nivel web y la capa de datos deben colocarse para reducir la latencia y el costo de tráfico.
* Básicamente, el tiempo de inactividad es un riesgo empresarial más alto para estas aplicaciones que la pérdida de datos.

En este caso, la topología de implementación de la aplicación está optimizada para el tratamiento de desastres regionales cuando tiene que realizarse la conmutación por error conjunta de todos los componentes de aplicación. En el diagrama siguiente se muestra esta topología. Para la redundancia geográfica, los recursos de la aplicación se implementan en la región A y B. Sin embargo, no se utilizan los recursos de la región B hasta que se produce un error en la región A. Se configura un grupo de conmutación por error entre las dos regiones para administrar la conmutación por error, la replicación y la conectividad de base de datos. El servicio web de ambas regiones está configurado para tener acceso a la base de datos mediante el agente de escucha de lectura y escritura **&lt;nombre del grupo de conmutación por error&gt;. database.windows.net** (1). Azure Traffic Manager se configura para usar el [método de enrutamiento de prioridad](../../traffic-manager/traffic-manager-configure-priority-routing-method.md) (2).  

> [!NOTE]
> [Azure Traffic Manager](../../traffic-manager/traffic-manager-overview.md) se usa en este artículo únicamente con fines ilustrativos. Puede usar cualquier solución de equilibrio de carga que admita el método de enrutamiento de prioridad.

El diagrama siguiente muestra esta configuración antes de una interrupción:

![Escenario 1. Configuración antes de la interrupción.](./media/designing-cloud-solutions-for-disaster-recovery/scenario1-a.png)

Después de una interrupción en la región primaria, SQL Database detectará que no se puede acceder a la base de datos principal y desencadenará una conmutación por error a la región secundaria en función de los parámetros de la directiva de conmutación por error automática (1). Dependiendo de su Acuerdo de Nivel de Servicio de aplicación, puede configurar un período de gracia que controle el tiempo entre la detección de la interrupción y la propia conmutación por error. Es posible que Azure Traffic Manager inicie la conmutación por error del punto de conexión antes de que el grupo de conmutación por error desencadene la conmutación por error de la base de datos. En ese caso, la aplicación web no se puede volver a conectar de inmediato a la base de datos. Sin embargo, las reconexiones se realizarán de forma correcta automáticamente tan pronto como se complete conmutación por error de la base de datos. Cuando se restaure la región errónea y vuelva a estar conectada, se volverá a conectar automáticamente con la primaria anterior como una nueva secundaria. El diagrama siguiente muestra la configuración después de la conmutación por error.

> [!NOTE]
> Durante la reconexión se pierden todas las transacciones confirmadas después de la conmutación por error. Una vez completada la conmutación por error, la aplicación de la región B puede volver a conectarse y reiniciar el procesamiento que solicita el usuario. La aplicación web y la base de datos principal se encuentran ahora en la región B y permanecen colocadas.

![Escenario 1. Configuración después de la conmutación por error](./media/designing-cloud-solutions-for-disaster-recovery/scenario1-b.png)

Si se produce una interrupción en la región B, se suspende el proceso de replicación entre la base de datos principal y la secundaria, pero el vínculo entre las dos permanece intacto (1). Traffic Manager detecta que esa conectividad con la región B se ha interrumpido y marca la aplicación web del punto de conexión 2 como Degradada (2). El rendimiento de la aplicación no se ve afectado en este caso, pero la base de datos pasa a estar expuesta y, por tanto, existe un mayor riesgo de pérdida de datos en el caso de que se produzca un error en la región A sucesivamente.

> [!NOTE]
> Para recuperación ante desastres, se recomienda la configuración con la implementación de aplicaciones limitada a dos regiones. Esto es porque la mayoría de las ubicaciones geográficas de Azure tienen solo dos regiones. Esta configuración no protegerá la aplicación de un error grave simultáneo de ambas regiones. En el caso poco probable de que este error se produjese, podría recuperar sus bases de datos en una tercera región mediante una [operación de restauración geográfica](disaster-recovery-guidance.md#recover-using-geo-restore).
>

 Una vez que se reduce la interrupción, la base de datos secundaria se resincroniza automáticamente con la principal. Durante la sincronización, el rendimiento de la principal puede verse afectado. El impacto específico depende de la cantidad de datos que adquirió la nueva principal desde la conmutación por error. 

> [!NOTE]
> Una vez mitigada la interrupción, Azure Traffic Manager comenzará a enrutar las conexiones a la aplicación de la región A como un punto de conexión con una prioridad más alta. Si pretende mantener la base de datos principal en la región B durante algún tiempo, debería cambiar la tabla de prioridades en el perfil de Azure Trafic Manager como corresponda. 
>
 
 El siguiente diagrama ilustra una interrupción en la región secundaria.

![Escenario 1. Configuración después de una interrupción en la región secundaria.](./media/designing-cloud-solutions-for-disaster-recovery/scenario1-c.png)

Las **ventajas** clave de este patrón de diseño son:

* La misma aplicación web se implementa en ambas regiones sin ninguna configuración específica de la región y no requiere una lógica adicional para administrar la conmutación por error.
* El rendimiento de la aplicación no se ve afectado por la conmutación por error, ya que la aplicación web y la base de datos siempre están colocadas.

La **contrapartida** principal es que los recursos de la aplicación de la región B están infrautilizados la mayoría del tiempo.

## <a name="scenario-2-azure-regions-for-business-continuity-with-maximum-data-preservation"></a>Escenario 2: Regiones de Azure para la continuidad empresarial con conservación máxima de datos

Esta opción es mejor para las aplicaciones con las siguientes características:

* Cualquier pérdida de datos supone un alto riesgo para la empresa. La conmutación por error de la base de datos solo puede usarse como último recurso si la interrupción se debe a un error muy grave.
* La aplicación admite los modos de solo lectura y lectura y escritura de operaciones y puede funcionar en "modo de solo lectura" durante un período de tiempo.

En este modelo, la aplicación cambia al modo de solo lectura cuando las conexiones de lectura y escritura empiezan a tener errores de tiempo de espera agotado. La aplicación web se implementa en ambas regiones e incluyen una conexión con el punto de conexión del agente de escucha de lectura y escritura y otra conexión con el punto de conexión del agente de escucha de solo lectura (1). El perfil de Traffic Manager debe usar el [enrutamiento de prioridad](../../traffic-manager/traffic-manager-configure-priority-routing-method.md). Debe habilitarse la [supervisión de punto de conexión](../../traffic-manager/traffic-manager-monitoring.md) para el punto de conexión de la aplicación en cada región (2).

El siguiente diagrama muestra esta configuración antes de una interrupción:

![Escenario 2. Configuración antes de la interrupción.](./media/designing-cloud-solutions-for-disaster-recovery/scenario2-a.png)

Cuando Traffic Manager detecta un error de conectividad con la región A, cambia automáticamente el tráfico de usuario a la instancia de aplicación de la región B. Con este modelo, es importante establecer el período de gracia con pérdida de datos en un valor suficientemente alto, por ejemplo, 24 horas. Esto asegurará que se evita la pérdida de datos si la interrupción se mitiga durante ese período. Cuando la aplicación web de la región B se active, las operaciones de lectura y escritura comenzarán a generar errores. En ese momento, debe cambiar al modo de solo lectura (1). En este modo, las solicitudes se enrutarán automáticamente a la base de datos secundaria. Si la interrupción se debe a un error muy grave, lo más probable es que no pueda mitigarse dentro del período de gracia. Cuando expira, el grupo de conmutación por error activa la conmutación por error. Después de eso, el agente de escucha de lectura y escritura pasará a estar disponible y las conexiones a él dejarán de dar error (2). El siguiente diagrama muestra las dos fases del proceso de recuperación.

> [!NOTE]
> Si la interrupción de la región primaria se soluciona dentro del período de gracia, Traffic Manager detecta la restauración de la conectividad en la región principal y cambia el tráfico de usuario de nuevo a la instancia de aplicación de la región A. Esa instancia de aplicación se reanuda y funciona en modo de lectura y escritura usando la base de datos principal de la región A, tal y como se mostró en el diagrama anterior.

![Escenario 2. Fases de recuperación ante desastres.](./media/designing-cloud-solutions-for-disaster-recovery/scenario2-b.png)

Si se produce una interrupción en la región B, Traffic Manager detecta el error de la aplicación web del punto de conexión 2 en la región B y lo marca como Degradado (1). Mientras tanto, en el grupo de conmutación por error cambia el agente de escucha de solo lectura a la región A (2). Esta interrupción no afecta a la experiencia del usuario final, pero la base de datos principal queda expuesta mientras tanto. El siguiente diagrama muestra un error en la región secundaria:

![Escenario 2. Interrupción de la región secundaria.](./media/designing-cloud-solutions-for-disaster-recovery/scenario2-c.png)

Una vez mitigada la interrupción, la base de datos secundaria se sincroniza inmediatamente con la principal y el agente de escucha de solo lectura se vuelve a cambiar a la base de datos secundaria de la región B. Durante la sincronización, el rendimiento de la principal podría verse ligeramente afectado dependiendo de la cantidad de datos que se necesite sincronizarse.

Este patrón de diseño tiene varias **ventajas**:

* Evita la pérdida de datos durante las interrupciones temporales.
* El tiempo de inactividad depende solo de la rapidez de Traffic Manager para detectar el error de conectividad, que se puede configurar.

La **contrapartida** es que la aplicación debe poder funcionada en modo de solo lectura.

## <a name="scenario-3-application-relocation-to-a-different-geography-without-data-loss-and-near-zero-downtime"></a>Escenario 3: Recolocación de la aplicación en una ubicación geográfica diferente sin que se produzca una pérdida de datos y prácticamente sin tiempo de inactividad

En este escenario, la aplicación tiene las siguientes características:

* Los usuarios finales tienen acceso a la aplicación desde diferentes ubicaciones geográficas.
* La aplicación incluye cargas de trabajo de solo lectura que no dependen de la sincronización completa con las actualizaciones más recientes.
* El acceso de escritura a los datos debe ser compatible en la misma ubicación geográfica para la mayoría de los usuarios.
* La latencia de lectura es fundamental para la experiencia del usuario final.

Con el fin de cumplir estos requisitos, tiene que asegurarse de que el dispositivo del usuario **siempre** se conecta a la aplicación implementada en la misma ubicación geográfica para las operaciones de solo lectura, como la exploración de datos, análisis, etc. Por otra parte, las operaciones OLTP se procesan en la misma geografía la **mayoría de las veces**. Por ejemplo, por el día, las operaciones OLTP se procesan en la misma ubicación geográfica, pero durante las horas de inactividad podrían procesarse en una ubicación geográfica diferente. Si la actividad del usuario final se produce principalmente durante las horas de trabajo, puede garantizar un rendimiento óptimo para la mayoría de los usuarios la mayor parte del tiempo. El siguiente diagrama muestra esta topología.

Los recursos de la aplicación deben implementase en cada ubicación geográfica en la que haya una demanda de uso considerable. Por ejemplo, si la aplicación se utiliza activamente en Estados Unidos, la Unión Europea y Sudeste de Asia, la aplicación debe implementarse en todas estas ubicaciones geográficas. La base de datos principal debe cambiarse dinámicamente de una ubicación geográfica a la siguiente al final de las horas laborables. A este método se le conoce como "seguimiento del sol". La carga de trabajo OLTP siempre se conecta a la base de datos mediante el agente de escucha de lectura y escritura **&lt;nombre de grupo de conmutación por error&gt;.database.windows.net** (1). La carga de trabajo de solo lectura se conecta a la base de datos local directamente mediante el punto de conexión del servidor de bases de datos **&lt;nombre del servidor&gt;.database.windows.net** (2). Traffic Manager está configurado con el [método de enrutamiento de rendimiento](../../traffic-manager/traffic-manager-configure-performance-routing-method.md). Eso garantiza que el dispositivo del usuario final se conecte al servicio web de la región más cercana. Traffic Manager debe configurarse con la supervisión de punto de conexión habilitada para cada punto de conexión de servicio web (3).

> [!NOTE]
> La configuración del grupo de conmutación por error define qué región se utiliza para la conmutación por error. Puesto que la región primaria nueva se encuentra en una ubicación geográfica diferente, la conmutación por error provoca una latencia mayor para las cargas de trabajo de solo lectura y OLTP hasta que la región afectada vuelva a estar conectada.

![Escenario 3. Configuración con la región primaria en el Este de EE. UU.](./media/designing-cloud-solutions-for-disaster-recovery/scenario3-a.png)

Al final del día (por ejemplo, a las 11 p.m., hora local), las bases de datos activas deben cambiarse a la siguiente región (Norte de Europa). Esta tarea puede se puede automatizar por completo mediante el uso de [Azure Logic Apps](../../logic-apps/logic-apps-overview.md). La tarea implica los pasos siguientes:

* Cambiar el servidor principal en el grupo de conmutación por error a Norte de Europa con una conmutación por error fácil de usar (1).
* Quitar el grupo de conmutación por error entre el Este de EE. UU. y Norte de Europa.
* Crear un nuevo grupo de conmutación por error con el mismo nombre, pero entre Norte de Europa y Este de Asia (2).
* Agregar la región primaria en Norte de Europa y la secundaria en Este de Asia para este grupo de conmutación por error (3).

El siguiente diagrama ilustra la nueva configuración después de la conmutación por error planeada:

![Escenario 3. Cambio de la región principal a Norte de Europa.](./media/designing-cloud-solutions-for-disaster-recovery/scenario3-b.png)

Si se produce una interrupción en Norte de Europa, por ejemplo, el grupo de conmutación por error inicia la conmutación por error de base de datos automática, lo que resulta eficaz para trasladar la aplicación a la siguiente región antes de lo previsto (1).  En ese caso, Este de EE. UU. es la única región secundaria que queda hasta que Norte de Europa vuelva a estar activa. Las dos regiones restantes sirven a los clientes en las otras tres regiones geográficas al intercambiar los roles. Azure Logic Apps debe ajustarse según corresponda. Puesto que el resto de regiones obtienen tráfico de usuario adicional desde Europa, el rendimiento de la aplicación se ve afectado no solo por latencia adicional, sino también por un mayor número de conexiones de usuario final. Una vez que se solucione la interrupción en Norte de Europa, la base de datos secundaria se sincronizará inmediatamente con la principal actual. El siguiente diagrama ilustra una interrupción en Norte de Europa:

![Escenario 3. Interrupción en Norte de Europa.](./media/designing-cloud-solutions-for-disaster-recovery/scenario3-c.png)

> [!NOTE]
> Puede reducir el tiempo cuando la experiencia del usuario final en Europa sea inferior debido a una mayor latencia. Para ello, debe implementar proactivamente una copia de la aplicación y crear bases de datos secundarias en otra región local (Oeste de Europa) como sustitución de la instancia de la aplicación sin conexión en Norte de Europa. Cuando esta última vuelva a estar activa, puede decidir si quiere seguir usando Oeste de Europa o quitar la copia de la aplicación y volver a usar Norte de Europa.

Las principales **ventajas** de este diseño son las siguientes:

* La carga de trabajo de la aplicación de solo lectura tiene acceso a datos en la región más cercana en todo momento.
* La carga de trabajo de aplicación de lectura y escritura aplicación tiene acceso a datos en la región más cercana durante el período de máxima actividad en cada región geográfica.
* Puesto que la aplicación se implementa en varias regiones, puede sobrevivir a una pérdida de una de las regiones sin que se produzca un tiempo de inactividad significativo.

Sin embargo, hay algunas **contraprestaciones**:

* Una interrupción regional da como resultado que una ubicación geográfica se vea afectada por una mayor latencia. La aplicación ofrece cargas de trabajo de solo lectura y de lectura y escritura en una ubicación geográfica diferente.
* Las cargas de trabajo de solo lectura deben conectarse a un punto de conexión distinto en cada región.

## <a name="business-continuity-planning-choose-an-application-design-for-cloud-disaster-recovery"></a>Planificación de la continuidad del negocio: elección del diseño de una aplicación para la recuperación ante desastres en la nube

Su estrategia de recuperación ante desastres en la nube puede combinar o ampliar estos patrones para satisfacer mejor las necesidades de su aplicación.  Como se mencionó anteriormente, la estrategia que elija se basa en el acuerdo de nivel de servicio que desee ofrecer a sus clientes y en la topología de implementación de la aplicación. Para guiarle en su decisión, la siguiente tabla compara las opciones en función del objetivo de punto de recuperación (RPO) y el tiempo de recuperación calculado (ERT).

| Patrón | RPO | ERT |
|:--- |:--- |:--- |
| Implementación activa-pasiva para la recuperación ante desastres con acceso a base de datos colocalizada |Acceso de lectura y escritura < 5 s |Tiempo de detección de errores + TTL de DNS |
| Implementación activa-activa para el equilibrio de carga de aplicación |Acceso de lectura y escritura < 5 s |Tiempo de detección de errores + TTL de DNS |
| Implementación activa-pasiva para la conservación de datos |Acceso de solo lectura < 5 s | Acceso de solo lectura = 0 |
||Acceso de lectura y escritura = cero | Acceso de lectura y escritura = Tiempo de detección de errores + Período de gracia con pérdida de datos |
|||

## <a name="next-steps"></a>Pasos siguientes

* Para obtener una descripción general y los escenarios de la continuidad empresarial, consulte [Continuidad empresarial con Base de datos SQL de Azure](business-continuity-high-availability-disaster-recover-hadr-overview.md)
* Para obtener más información sobre la replicación geográfica activa, consulte [Replicación geográfica activa](active-geo-replication-overview.md).
* Para obtener información acerca de los grupos de conmutación por error automática, consulte [Grupos de conmutación por error automática](auto-failover-group-overview.md).
* Para información acerca de cómo utilizar la replicación geográfica activa con los grupos elásticos, consulte [Estrategias de recuperación ante desastres para aplicaciones que usan grupos elásticos de SQL Database](disaster-recovery-strategies-for-applications-with-elastic-pool.md).
