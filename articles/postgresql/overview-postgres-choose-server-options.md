---
title: Elección de la opción de servidor de PostgreSQL adecuada en Azure
description: Proporciona directrices para elegir la opción de servidor de PostgreSQL adecuada para las implementaciones.
author: sunilagarwal
ms.author: sunila
ms.custom: mvc
ms.service: postgresql
ms.topic: overview
ms.date: 09/21/2020
ms.openlocfilehash: f81a44af4a90ccda1875c0a9f2bfaa8dc0b73441
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945810"
---
# <a name="choose-the-right-postgresql-server-option-in-azure"></a>Elección de la opción de servidor de PostgreSQL adecuada en Azure

Con Azure, las cargas de trabajo del servidor PostgreSQL se pueden ejecutar en una infraestructura como servicio (IaaS) de máquina virtual hospedada o como una plataforma como servicio (PaaS) hospedada. PaaS tiene varias opciones de implementación, cada una con varios niveles de servicio. Al decidir entre PaaS o IaaS, debe decidir si quiere administrar la base de datos, aplicar revisiones y realizar copias de seguridad, o si quiere delegar estas operaciones en Azure.

Al tomar la decisión, tenga en cuenta las tres opciones siguientes en PaaS o en máquinas virtuales de Azure (IaaS) como alternativa.
- [Azure Database for PostgreSQL con la opción Servidor único](./overview-single-server.md)
- [Azure Database for PostgreSQL con la opción Servidor flexible](./flexible-server/overview.md)
- [Azure database for PostgreSQL con la opción Hiperescala (Citus)]()

La opción **PostgreSQL on Azure VMs** (PostgreSQL en máquinas virtuales de Azure) pertenece a la categoría del sector de IaaS. Con este servicio, puede ejecutar el servidor PostgreSQL en una máquina virtual totalmente administrada en la plataforma en la nube de Azure. Todas las versiones y ediciones recientes de PostgreSQL se pueden instalar en una máquina virtual IaaS. La diferencia más importante con Azure Database for PostgreSQL es que PostgreSQL en máquinas virtuales de Azure permite el control sobre el motor de base de datos. Sin embargo, este control conlleva el costo de la responsabilidad de administrar las máquinas virtuales y muchas tareas de administración de bases de datos (DBA). Estas tareas incluyen el mantenimiento y la revisión de los servidores de las bases de datos, la recuperación de las bases de datos y el diseño de la alta disponibilidad.

Las principales diferencias entre estas opciones se mencionan en la siguiente tabla:

| **Atributo** | **Postgres en máquinas virtuales de Azure** | **PostgreSQL como plataforma como servicio** |
| ----- | ----- | ----- |
| <B> Acuerdo de Nivel de Servicio de disponibilidad |-99,99 % con conjuntos de disponibilidad <br> -99,95 % con máquinas virtuales únicas | - Servidor único: 99,99 % <br> -Servidor flexible: no disponible durante la versión preliminar <br> -Hiperescala (Citus): 99,95% (cuando está habilitada la alta disponibilidad)|
| <B> Aplicación de revisión de SO y PostgreSQL | - Administrado por el cliente | - Servidor único: automático <br> -Servidor flexible: automático con una ventana opcional administrada por el cliente <br> - Hiperescala (Citus): automático |
| <B> Alta disponibilidad | - Los clientes diseñan, implementan, prueban y mantienen una alta disponibilidad. Las funcionalidades pueden incluir la agrupación en clústeres, la replicación, etc. | - Servidor único: integrado <br> - Servidor flexible: integrado <br> - Hiperescala (Citus): creada con modo de espera |
| <B> Redundancia de zona | - Las máquinas virtuales de Azure se pueden configurar para que se ejecuten en distintas zonas de disponibilidad. En el caso de una solución local, los clientes deben crear, administrar y mantener su propio centro de datos secundario. | - Servidor único: No <br> - Servidor flexible: Sí <br> - Hiperescala (Citus) No |
| <B> Escenario híbrido | - Administrado por el cliente |- Servidor único: Réplica de lectura <br> - Servidor flexible: No disponible en la versión preliminar <br> - Hiperescala (Citus) No |
| <B> Copias de seguridad y restauración | - Administrado por el cliente | -Servidor único: integrado con la configuración de usuario para el área local y geográfica <br> -Servidor flexible: integrado con la configuración de usuario en el almacenamiento con redundancia de zona <br> - Hiperescala (Citus): integrada |
| <B> Supervisión de las operaciones de base de datos | - Administrado por el cliente | - Servidor único, Servidor flexible e Hiperescala (Citus): Ofrecen a los clientes la capacidad de definir alertas en la operación de base de datos y realizar acciones al alcanzar los umbrales. |
| <B> Microsoft Defender for Identity | - Los clientes deben compilar esta protección por sí mismos. |- Servidor único: Sí <br> - Servidor flexible: No disponible en la versión preliminar <br> - Hiperescala (Citus) No |
| <B> Recuperación ante desastres | - Administrado por el cliente | - Servidor único: Copias de seguridad con redundancia geográfica y réplica de lectura geográfica <br> - Servidor flexible: No disponible en la versión preliminar <br> - Hiperescala (Citus) No |
| <B> Rendimiento inteligente | - Administrado por el cliente | - Servidor único: Sí <br> - Servidor flexible: No disponible en la versión preliminar <br> - Hiperescala (Citus) No |

## <a name="total-cost-of-ownership-tco"></a>Costo total de propiedad (TCO)

El costo total de propiedad suele ser la consideración principal que determina la mejor solución para hospedar las bases de datos. Esto suele darse en el caso de las startups con poca disponibilidad de efectivo o el equipo de una empresa bien establecida que opera con presupuestos ajustados. En esta sección se describen los conceptos básicos de facturación y licencias en Azure tal como se aplican a Azure Database for PostgreSQL y PostgreSQL en las máquinas virtuales de Azure.

## <a name="billing"></a>Facturación

Azure Database for PostgreSQL está disponible como servicio en varios niveles con recursos de distintos precios. Todos los recursos se facturan por hora a una tarifa fija. Para obtener la información más reciente sobre los niveles de servicio, los tamaños de proceso y las cantidades de almacenamiento que se admiten actualmente, consulte la [página de precios](https://azure.microsoft.com/pricing/details/postgresql/server/). Puede ajustar de forma dinámica los niveles de servicio y los tamaños de proceso para satisfacer las necesidades de rendimiento variables de la aplicación. Se le factura por el tráfico saliente de Internet según las [tarifas de transferencia de datos](https://azure.microsoft.com/pricing/details/data-transfers/) habituales.

Con Azure Database for PostgreSQL, Microsoft configura el software de base de datos, aplica revisiones en él y lo actualiza de forma automática. Estas acciones automatizadas reducen los costos de administración. Además, Azure Database for PostgreSQL tiene funcionalidades de [vínculos de copia de seguridad automatizados](). Estas funcionalidades le ayudan a obtener un ahorro significativo, sobre todo, cuando se tiene gran cantidad de base de datos. Por el contrario, con PostgreSQL en las máquinas virtuales de Azure puede elegir y ejecutar cualquiera de las versiones de PostgreSQL. Sin embargo, debe pagar la máquina virtual aprovisionada, el coste de almacenamiento asociado a los datos, la copia de seguridad, la supervisión de los datos y el almacenamiento de registros, y los costes del tipo de licencia de PostgreSQL específico que se utilice (si existe).

Azure Database for PostgreSQL proporciona alta disponibilidad integrada para cualquier tipo de interrupción de nivel de nodo y mantiene al mismo tiempo la garantía del 99,99 % establecida en el Acuerdo de Nivel de Servicio para el servicio. No obstante, para la alta disponibilidad de la base de datos en las máquinas virtuales, se usan las opciones de alta disponibilidad que están disponibles en una base de datos de PostgreSQL, como la [replicación de streaming](https://www.postgresql.org/docs/12/warm-standby.html#STREAMING-REPLICATION). El uso de una opción de alta disponibilidad compatible no proporciona un SLA adicional, pero permite lograr una disponibilidad de base de datos del 99,99 % a cambio de un costo adicional y sobrecarga administrativa.

Para más información sobre los precios, consulte los siguientes artículos:
- [Precios de Azure Database for PostgreSQL](https://azure.microsoft.com/pricing/details/postgresql/server/)
- [Precios de máquinas virtuales](https://azure.microsoft.com/pricing/details/virtual-machines/)
- [Calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/)

## <a name="administration"></a>Administración

En muchas empresas, la decisión de pasar a un servicio en la nube está tan relacionada con la posibilidad de reducir la carga de complejidad de administración como con el costo.

Con IaaS, Microsoft:

- Administra la infraestructura subyacente.
- Proporciona una aplicación automatizada de revisiones para el hardware y el sistema operativo subyacentes.

Con PaaS, Microsoft:

- Administra la infraestructura subyacente.
- Proporciona una aplicación automatizada de revisiones para el hardware, el sistema operativo y el motor de base de datos subyacentes.
- Administra la alta disponibilidad de la base de datos.
- Realiza copias de seguridad y replicaciones automáticas de todos los datos para proporcionar recuperación ante desastres.
- Cifra los datos en reposo y en movimiento de forma predeterminada.
- Supervisa el servidor y proporciona características para obtener información de rendimiento de consultas y recomendaciones de rendimiento.

Con Azure Database for PostgreSQL, puede seguir administrando la base de datos. Sin embargo, ya no necesita administrar el motor de base de datos, el sistema operativo o el hardware. Entre los ejemplos de elementos que puede seguir administrando se incluyen:

- Bases de datos
- Inicio de sesión
- Ajuste del índice
- Ajuste de las consultas
- Auditoría
- Seguridad

Además, la configuración de alta disponibilidad en otro centro de datos no requiere configuración y administración o, si acaso, ajustes mínimos.

- Con PostgreSQL en las máquinas virtuales de Azure tiene un control completo sobre la configuración del sistema operativo y la instancia de servidor de PostgreSQL. Con una máquina virtual, puede decidir cuándo actualizar o renovar el sistema operativo y el software de la base de datos, y qué revisiones aplicar. También puede decidir cuándo instalar cualquier software adicional, como una aplicación antivirus. Se proporcionan algunas características automatizadas para simplificar considerablemente la aplicación de revisiones, las copias de seguridad y la alta disponibilidad. Puede controlar el tamaño de la máquina virtual, el número de discos y sus configuraciones de almacenamiento. Para más información, consulte [Tamaños de las máquinas virtuales y los servicios en la nube de Azure](https://docs.microsoft.com/azure/virtual-machines/windows/sizes).

## <a name="time-to-move-to-azure-postgresql-service-paas"></a>Tiempo de traslado al servicio Azure PostgreSQL (PaaS)

- Azure Database for PostgreSQL es la solución adecuada para las aplicaciones diseñadas para la nube cuando la productividad del desarrollador y un plazo de comercialización rápido de las nuevas soluciones son factores críticos. Con una funcionalidad mediante programación de tipo DBA, el servicio es idóneo para arquitectos y desarrolladores de la nube, puesto que reduce la necesidad de administrar el sistema operativo y la base de datos subyacentes.

- Si quiere evitar el tiempo y los gastos de adquirir nuevo hardware local, PostgreSQL en máquinas virtuales de Azure es la solución adecuada para las aplicaciones que necesitan un control granular y la personalización del motor de PostgreSQL que no admite el servicio o que requiere acceso al sistema operativo subyacente.

## <a name="next-steps"></a>Pasos siguientes

- Consulte los [precios de Azure Database for PostgreSQL](https://azure.microsoft.com/pricing/details/postgresql/server/).
- Comience por crear su primer servidor.

