---
title: ¿Qué es Azure SQL?
titleSuffix: " "
description: 'Obtenga información sobre las diferentes opciones de la familia de servicios de Azure SQL: Azure SQL Database, Instancia administrada de Azure SQL y SQL Server en una máquina virtual de Azure.'
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: sqldbrb=4
ms.devlang: ''
ms.topic: conceptual
keywords: Nube de SQL Server, SQL Server en la nube, base de datos PaaS, DBaaS, IaaS
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 07/27/2020
ms.openlocfilehash: ff8dd9d10bf5970a5a96080beb51e2c111d481e3
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87280858"
---
# <a name="what-is-azure-sql"></a>¿Qué es Azure SQL? 
[!INCLUDE[appliesto-asf](includes/appliesto-asf.md)]

Azure SQL es una familia de productos administrados, seguros e inteligentes que usan el motor de base de datos de SQL Server de la nube de Azure.

- **Azure SQL Database**: admita aplicaciones modernas en la nube en un servicio de base de datos inteligente y administrado que incluye procesos sin servidor. 
- **Instancia administrada de Azure SQL**: modernice sus aplicaciones de SQL Server existentes a escala con una instancia totalmente administrada inteligente como servicio, que tiene una paridad de características casi del 100 % con el motor de base de datos de SQL Server. la mejor opción para la mayoría de las migraciones a la nube.
- **SQL Server en Máquinas virtuales de Azure**: migre mediante lift-and-shift sus cargas de trabajo de SQL Server con facilidad y conserve la una compatibilidad del 100 % con SQL Server, así como acceso de nivel de sistema operativo. 
 
Azure SQL se basa en el conocido motor de SQL Server, por lo que puede migrar aplicaciones con facilidad y seguir usando las herramientas, lenguajes y recursos con los que está familiarizado. Sus aptitudes y experiencia se trasladan a la nube, por lo que puede ´realizar más acciones con lo que ya tiene. 

Obtenga información acerca de qué producto se ajusta a la plataforma de datos de Microsoft Azure SQL para encontrar la opción más adecuada para sus requisitos empresariales. Si asigna mayor prioridad al ahorro o bien antepone la mínima administración, este artículo puede ayudarle a decidir el enfoque correcto, en función de los requisitos empresariales que más le preocupan.


## <a name="overview"></a>Información general

En el mundo controlado por datos de la actualidad, impulsar la transformación digital depende cada vez más de nuestra capacidad para administrar grandes cantidades de datos y aprovechar su potencial. Sin embargo, los datos actualmente son cada vez más complejos, con datos hospedados de forma local, en la nube o en el perímetro de la red. Los desarrolladores que crean aplicaciones inteligentes y envolventes pueden verse limitados de formas que, en última instancia, pueden afectar su experiencia. Las limitaciones derivadas de las plataformas incompatibles, la seguridad de datos inadecuada, los recursos insuficientes y las barreras de precio-rendimiento crean una complejidad que puede impedir el desarrollo y la modernización de las aplicaciones. 

Una de las primeras cosas que hay que comprender al comparar Azure con bases de datos SQL Server locales es que puede usarlas todas. La plataforma de datos de Microsoft aprovecha la tecnología de SQL Server y la pone a disposición de los usuarios en máquinas físicas locales, entornos en nubes privadas, entornos en nubes privadas hospedados por terceros y la nube pública. 


### <a name="fully-managed-and-always-up-to-date"></a>Totalmente administrado y siempre actualizado 

Dedique más tiempo a innovar y menos tiempo a revisar, actualizar y realizar copias de seguridad de las bases de datos. Azure es la única nube con SQL permanente que aplica automáticamente las actualizaciones y revisiones más recientes para que las bases de datos estén siempre actualizadas. De este modo, se eliminan las complicaciones del fin del soporte técnico. Incluso las tareas complejas como la optimización del rendimiento, la alta disponibilidad, la recuperación ante desastres y las copias de seguridad están automatizadas, lo que le permite centrarse en las aplicaciones.  

### <a name="protect-your-data-with-built-in-intelligent-security"></a>Protección de los datos con seguridad inteligente integrada 

Azure supervisa constantemente los datos en busca de amenazas. Con Azure SQL, puede hacer lo siguiente:

- Solucionar posibles amenazas en tiempo real con [detección de amenazas avanzada](https://docs.microsoft.com/azure/security/fundamentals/threat-detection#advanced-threat-detection-features-other-azure-services) inteligente y alertas proactivas de evaluación de vulnerabilidades. 
- Obtener la protección de varios niveles líder del sector con [controles de seguridad integrados](https://azure.microsoft.com/overview/security/) como T-SQL, autenticación, redes y administración de claves. 
- Aprovechar las ventajas de la cobertura de [cumplimiento](https://azure.microsoft.com/overview/trusted-cloud/compliance/) más completa de cualquier servicio de base de datos en la nube. 


### <a name="business-motivations"></a>Motivaciones empresariales

Hay varios factores que pueden influir a la hora de elegir entre las distintas ofertas de datos:

- [Costo](#cost): tanto PaaS como IaaS incluyen un precio base que abarca la infraestructura subyacente y la concesión de licencias. Sin embargo, con la opción IaaS deberá invertir más tiempo y recursos para administrar la base de datos, mientras que con PaaS disfruta de estas características de administración por el mismo precio. IaaS le permite apagar los recursos cuando no los esté usando para reducir el costo, mientras que PaaS los ejecuta siempre, a menos que quite los recursos y vuelva a crearlos según sea necesario.
- [Administración:](#administration): las opciones PaaS reducen la cantidad de tiempo que necesita invertir en administrar la base de datos. Sin embargo, también limita la gama de tareas de administración personalizadas y los scripts que se pueden realizar o ejecutar. Por ejemplo, CLR no es compatible con SQL Database, pero admite una instancia de SQL Managed Instance. Además, ninguna opción de implementación en PaaS admite el uso de marcas de seguimiento.
- [Contrato de nivel de servicio](#service-level-agreement-sla): IaaS y PaaS proporcionan un Acuerdo de Nivel de Servicio estándar del sector alto. La opción PaaS garantiza un SLA del 99,99 %, mientras que IaaS garantiza un SLA del 99,95 % para la infraestructura, lo que significa que deberá implementar mecanismos adicionales para garantizar la disponibilidad de las bases de datos. Puede alcanzar un contrato de nivel de servicio del 99,99 % si crea una máquina virtual de SQL adicional e implementa la solución de alta disponibilidad del grupo de disponibilidad Always On de SQL Server. 
- [Es hora de cambiar a Azure](#market): SQL Server en la máquina virtual de Azure es una copia exacta del entorno, por lo que la migración del entorno local a la máquina virtual de Azure es casi lo mismo que mover las bases de datos de un servidor local a otro. Instancia administrada de SQL también permite una migración sencilla. Sin embargo, puede que sea necesario aplicar algunos cambios antes de migrar. 


## <a name="service-comparison"></a>Comparación de servicios

   ![Opciones de SQL Server en la nube: SQL Server en IaaS o SQL Database de SaaS en la nube.](./media/azure-sql-iaas-vs-paas-what-is-overview/SQLIAAS_SQL_Server_Cloud_Continuum.png)

Como se ve en el diagrama, cada oferta de servicio puede caracterizarse por el nivel de administración que se tiene sobre la infraestructura y el grado de relación coste-eficacia.

En Azure, puede hacer que las cargas de trabajo de SQL Server se ejecuten en un servicio hospedado ([PaaS](https://azure.microsoft.com/overview/what-is-paas/)) o en una infraestructura hospedada ([IaaS](https://azure.microsoft.com/overview/what-is-iaas/)). En PaaS, tiene varias opciones de producto y niveles de servicio dentro de cada opción. La pregunta clave que debe hacerse al decidir entre PaaS o IaaS es si quiere administrar su base de datos, aplicar revisiones y realizar copias de seguridad, o bien si quiere delegar estas operaciones a Azure.

### <a name="azure-sql-database"></a>Azure SQL Database

[Azure SQL Database](database/sql-database-paas-overview.md) es una base de datos relacional como servicio (DBaaS), hospedada en Azure, que se engloba en la categoría del sector denominada *Plataforma como servicio (PaaS)* . 
- la mejor opción para las aplicaciones en la nube modernas que pretenden usar las características estables de SQL Server más recientes y que tienen restricciones de tiempo en las actividades de desarrollo y marketing. 
- Un motor de base de datos de SQL Server totalmente administrado, basado en la última versión de Enterprise Edition estable de SQL Server. SQL Database tiene dos opciones de implementación basadas en hardware y software estandarizados que Microsoft posee, hospeda y mantiene. 

Con SQL Server, puede usar la funcionalidad y las características integradas que requieren configuración extensiva (ya sea en el entorno local o en una máquina virtual de Azure). Al utilizar SQL Database, se emplea el método de pago por uso de opciones para escalar vertical u horizontalmente a fin de aumentar la potencia de forma ininterrumpida. SQL Database incluye algunas características adicionales que no están disponibles en SQL Server, como la alta disponibilidad, la administración y la inteligencia integradas.


Azure SQL Database ofrece las siguientes opciones de implementación:
  - Como una [*base de datos única*](database/single-database-overview.md) con su propio conjunto de recursos administrados mediante un [servidor SQL lógico](database/logical-servers.md). Una base de datos única es similar a una [base de datos independiente](https://docs.microsoft.com/sql/relational-databases/databases/contained-databases) en SQL Server. Esta opción está optimizada para el desarrollo de las nuevas aplicaciones modernas procedentes de la nube. Las opciones de [hiperescala](database/service-tier-hyperscale.md) y [sin servidor](database/serverless-tier-overview.md) están disponibles.
  - Un [*grupo elástico*](database/elastic-pool-overview.md), que es una colección de bases de datos con un conjunto compartido de recursos administrados a través de un [servidor SQL lógico](database/logical-servers.md). Las bases de datos únicas se pueden mover dentro y fuera de un grupo elástico. Esta opción está optimizada para el desarrollo de las nuevas aplicaciones modernas procedentes de la nube mediante el patrón de aplicación SaaS multiinquilino. Los grupos elásticos proporcionan una solución rentable para administrar el rendimiento de varias bases de datos que tienen patrones de utilización variables.

### <a name="azure-sql-managed-instance"></a>Instancia administrada de Azure SQL

[Azure SQL Managed Instance](managed-instance/sql-managed-instance-paas-overview.md) se engloba en la categoría del sector de *Plataforma como servicio (PaaS)* y es más conveniente para la mayoría de las migraciones a la nube. Instancia administrada de SQL es una colección de bases de datos de usuario y del sistema con un conjunto compartido de recursos que está listo para migraciones mediante lift-and-shift.  
- La mejor opción para aplicaciones nuevas o aplicaciones locales existentes que pretenden usar las características estables de SQL Server más recientes y que se han migrado a la nube con cambios mínimos. Una instancia de SQL Managed Instance es parecida a una instancia del [motor de base de datos de Microsoft SQL Server](https://docs.microsoft.com/sql/database-engine/sql-server-database-engine-overview), ya que ofrece recursos compartidos para las bases de datos y funciones de ámbito de instancia adicionales. 
- La instancia administrada de SQL admite la migración de base de datos desde el entorno local con cambios mínimos o nulos en las bases de datos. Esta opción proporciona todas las ventajas de PaaS de Azure SQL Database, pero incluye funcionalidades que anteriormente solo estaban disponibles en las máquinas virtuales con SQL Server. Por ejemplo, una red virtual nativa y un nivel próximo al 100 % de compatibilidad con la instancia local de SQL Server. Las instancias de SQL Managed Instance proporcionan acceso completo a SQL Server y compatibilidad con características para migrar servidores de SQL Server a Azure.

### <a name="sql-server-on-azure-vm"></a>SQL Server en máquina virtual de Azure

[SQL Server en máquinas virtuales de Azure](virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md) se engloba en la categoría del sector denominada *Infraestructura como servicio (IaaS)* y permite ejecutar SQL Server en una máquina virtual totalmente administrada en Azure. 
- la mejor opción para las migraciones y las aplicaciones que requieren acceso a nivel de sistema operativo. Las máquinas virtuales con SQL de Azure están listas para las migraciones mediante lift-and-shift para aplicaciones existentes que requieren una rápida migración a la nube con un número mínimo o nulo de cambios. Las máquinas virtuales SQL ofrecen un control administrativo total sobre la instancia de SQL Server y el sistema operativo subyacente para la migración a Azure. 
- Escenarios de desarrollo rápido y pruebas cuando no se desea comprar hardware de SQL Server de no producción local. Las máquinas virtuales SQL también se ejecutan en hardware estandarizado que Microsoft posee, hospeda y mantiene. Cuando se usan máquinas virtuales SQL, puede usar la licencia de pago por uso de SQL Server ya incluida en una imagen de SQL Server o usar fácilmente una licencia existente. También puede detener o reanudar la máquina virtual según sea necesario. 
- SQL Server instalado y hospedado en la nube se ejecuta en máquinas virtuales de Windows Server o Linux que se ejecutan en Azure, conocido también como infraestructura como servicio (IaaS). Las máquinas virtuales SQL son una buena opción para migrar aplicaciones y bases de datos de SQL Server locales sin realizar ningún cambio en las bases de datos. Todas las versiones y ediciones recientes de SQL Server están disponibles para la instalación en una máquina virtual de IaaS. 

    La diferencia más importante entre SQL Database y SQL Managed Instance es que SQL Server en Azure Virtual Machines permite un control total sobre el motor de base de datos. Puede elegir cuándo se iniciará el mantenimiento o la revisión, cambiar el modelo de recuperación de registro simple o masivo, pausar o iniciar el servicio cuando sea necesario y personalizar completamente el motor de base de datos de SQL Server. Este control adicional incluye la responsabilidad de administrar la máquina virtual.
- Optimizada para migrar las aplicaciones existentes a Azure o para llevar las aplicaciones locales existentes a la nube en implementaciones híbridas. Además, puede usar SQL Server en una máquina virtual para desarrollar y probar las aplicaciones tradicionales de SQL Server. Con las máquinas virtuales con SQL Server, tiene todos los derechos administrativos sobre una instancia dedicada de SQL Server y una máquina virtual basada en la nube. Es la elección perfecta cuando una organización ya dispone de recursos de TI para mantener las máquinas virtuales. Estas funcionalidades permiten crear un sistema muy personalizado para abordar los requisitos de rendimiento y disponibilidad específicos de la aplicación.

En la tabla siguiente se enumeran las diferencias adicionales, aunque *SQL Database e Instancia administrada de SQL están optimizados para reducir el costo total de administración al mínimo para el aprovisionamiento y la administración de muchas bases de datos.* Los costos recurrentes de administración se reducen, ya que no es necesario administrar las máquinas virtuales, los sistemas operativos ni el software de las bases de datos. No es necesario administrar actualizaciones, alta disponibilidad ni [copias de seguridad](database/automated-backups-overview.md). 

Habitualmente, SQL Database e Instancia administrada de SQL pueden aumentar considerablemente el número de bases de datos que administra un solo recurso de TI o de desarrollo. Los [grupos elásticos](database/elastic-pool-overview.md) también admiten arquitecturas de aplicaciones SaaS multiinquilino con características como el aislamiento de inquilinos y la capacidad de escalado para reducir costos al compartir recursos entre las bases de datos. [SQL Managed Instance](managed-instance/sql-managed-instance-paas-overview.md) proporciona compatibilidad con características de ámbito de instancia, lo que facilita la migración de las aplicaciones existentes, así como el uso compartido de recursos entre las bases de datos.

### <a name="comparison-table"></a>Tabla de comparación

| Azure SQL Database | Instancia administrada de Azure SQL | SQL Server en máquina virtual de Azure |
| :--- | :--- | :--- |
|Admite la mayoría de las funcionalidades de nivel de base de datos locales. Las características de SQL Server más utilizadas están disponibles.<br/>99,995 % de disponibilidad garantizada.<br/>Copias de seguridad, revisiones y acciones de recuperación integradas.<br/>Versión más reciente y estable del motor de base de datos.<br/>Capacidad para asignar los recursos necesarios (CPU y almacenamiento) a bases de datos individuales.<br/>Seguridad e inteligencia avanzadas e integradas.<br/>Cambio en línea de recursos (CPU y almacenamiento).| Admite casi todas las funcionalidades de nivel de instancia y nivel de base de datos locales. Alta compatibilidad con SQL Server.<br/>99,99 % de disponibilidad garantizada.<br/>Copias de seguridad, revisiones y acciones de recuperación integradas.<br/>Versión más reciente y estable del motor de base de datos.<br/>Migración sencilla desde SQL Server.<br/>Dirección IP privada dentro de Azure Virtual Network.<br/>Seguridad e inteligencia avanzadas e integradas.<br/>Cambio en línea de recursos (CPU y almacenamiento).| Tiene control total sobre el motor de SQL Server. Admite todas las funcionalidades locales.<br/>Disponibilidad de hasta el 99,99 %.<br/>Paridad completa con la versión correspondiente de una instancia local de SQL Server.<br/>Versión del motor de base de datos corregida y conocida.<br/>Migración sencilla desde SQL Server.<br/>Dirección IP privada dentro de Azure Virtual Network.<br/>Tiene la posibilidad de implementar aplicaciones o servicios en el host donde se sitúa la instancia de SQL Server.|
|La migración desde SQL Server puede ser un desafío.<br/>Algunas características de SQL Server no están disponibles.<br/>El tiempo exacto de mantenimiento no se garantiza (pero es casi transparente).<br/>La compatibilidad con la versión de SQL Server se puede lograr solo con los niveles de compatibilidad de la base de datos.<br/>Compatibilidad de direcciones IP privadas con [Azure Private Link](database/private-endpoint-overview.md).|Sigue habiendo un número mínimo de características de SQL Server que no están disponibles.<br/>El tiempo exacto de mantenimiento no se garantiza (pero es casi transparente).<br/>La compatibilidad con la versión de SQL Server se puede lograr solo con los niveles de compatibilidad de la base de datos.|Deberá administrar usted mismo las copias de seguridad y revisiones.<br>Deberá implementar su propia solución de alta disponibilidad.<br/>Existe un tiempo de inactividad mientras se cambian los recursos (CPU y almacenamiento)|
| Bases de datos de hasta 100 TB. | Hasta 8 TB. | Instancias de SQL Server con hasta 256 TB de almacenamiento. La instancia puede admitir tantas bases de datos como sea necesario. |
| La aplicación local puede obtener acceso a datos de Azure SQL Database. | [Implementación nativa de redes virtuales](managed-instance/vnet-existing-add-subnet.md) y conectividad al entorno local mediante Azure ExpressRoute o VPN Gateway. | Con las máquinas virtuales con SQL se pueden tener aplicaciones que se ejecuten parcialmente en la nube y parcialmente en la instalación local. Por ejemplo, se puede ampliar la red local y el Dominio de Active Directory a la nube mediante [Azure Virtual Network](../virtual-network/virtual-networks-overview.md). Para obtener más información sobre las soluciones de la nube híbrida, consulte [Ampliación de las soluciones de datos locales a la nube](https://docs.microsoft.com/azure/architecture/data-guide/scenarios/hybrid-on-premises-and-cloud). |


## <a name="cost"></a>Coste

Si forma parte de una startup con falta de medios o de un equipo de una compañía bien establecida que opera con restricciones presupuestarias, los fondos limitados suelen ser el factor principal a la hora de decidir cómo hospedar la base de datos. En esta sección, obtendrá información sobre los conceptos básicos de facturación y concesión de licencias de Azure asociados a la familia de servicios de Azure SQL.  También aprenderá a calcular el costo total de la aplicación.

### <a name="billing-and-licensing-basics"></a>Conceptos básicos sobre facturación y licencias

Actualmente, **SQL Database** e **Instancia administrada de SQL** se venden como servicios y están disponible con varias opciones y varios niveles de servicio con distintos precios para los recursos, que se facturan por hora a una tarifa fija en función del nivel de servicio y el tamaño de proceso que se elija. Para acceder a la información más reciente sobre los niveles de servicio, los tamaños de proceso y las cantidades de almacenamiento admitidos actualmente, consulte los artículos sobre el [modelo de compra basado en DTU para SQL Database](database/service-tiers-dtu.md) y el [modelo de compra basado en núcleo virtual para SQL Database e Instancia administrada de SQL](database/service-tiers-vcore.md).

- Con SQL Database, puede elegir un nivel de servicio que se adapte a sus necesidades entre una amplia gama de precios a partir de 5 dólares USD al mes para el nivel básico. Incluso, puede crear [grupos elásticos](database/elastic-pool-overview.md) para compartir recursos entre bases de datos con el fin de reducir los costos y dejar sitio para los picos de uso.
- También puede usar su propia licencia con Instancia administrada de SQL. Para obtener más información sobre la incorporación de licencias propias, consulte [Movilidad de licencias a través de Software Assurance en Azure](https://azure.microsoft.com/pricing/license-mobility/) o use la [calculadora de Ventaja híbrida de Azure ](https://azure.microsoft.com/pricing/hybrid-benefit/#sql-database) para saber cómo **ahorrar hasta un 40 %** .

Además, se le facturará el tráfico saliente de Internet en las [tasas de transferencia de datos](https://azure.microsoft.com/pricing/details/data-transfers/)habituales. Los niveles de servicio y los tamaños de proceso se pueden ajustar de forma dinámica para satisfacer las necesidades variables de rendimiento de la aplicación.

Con **SQL Database** e **Instancia administrada de SQL**, Azure configura, revisa y actualiza automáticamente el software de base de datos, lo que reduce los costos de administración. Además, sus capacidades de [copia de seguridad integrada](database/automated-backups-overview.md) ayudan a obtener un ahorro significativo, sobre todo, cuando se tiene gran cantidad de base de datos.

Con **SQL en máquinas virtuales de Azure**, también puede usar cualquiera de las imágenes de SQL Server que proporciona la plataforma (que incluyen una licencia) o incorporar su licencia de SQL Server. Están disponibles todas las versiones (2008 R2, 2012, 2014, 2016, 2017 y 2019) y ediciones (Developer, Express, Web, Standard y Enterprise) de SQL Server. Además, hay disponibles versiones “traiga su propia licencia” (BYOL) de las imágenes. Cuando se usan las imágenes suministradas por Azure, el costo operativo depende del tamaño de la máquina virtual, así como de la edición de SQL Server que elija. Independientemente del tamaño de la máquina virtual o la edición de SQL Server, se paga el costo de licencia por minuto de SQL Server y Windows o Linux Server, junto con el costo de Azure Storage para los discos de la máquina virtual. La opción de facturación por minuto permite utilizar SQL Server durante el tiempo que sea necesario sin comprar licencias adicionales de SQL Server. Si incorpora su propia licencia de SQL Server a Azure, solo se cobran los costos de servidor y de almacenamiento. Para obtener más información sobre la incorporación de licencias propias, consulte [Movilidad de Licencias a través de Software Assurance en Azure](https://azure.microsoft.com/pricing/license-mobility/). Además, se le facturará el tráfico saliente de Internet en las [tasas de transferencia de datos](https://azure.microsoft.com/pricing/details/data-transfers/)habituales.

#### <a name="calculating-the-total-application-cost"></a>Cálculo del coste total de la aplicación

Cuando se comienza a usar una plataforma en la nube, el coste de ejecución de la aplicación incluye los costos de nuevo desarrollo y de administración continua, además de los costos de servicio de la plataforma en la nube pública.

Para obtener más información sobre los precios, consulte los siguientes recursos:

- [Precios de SQL Database e Instancia administrada de SQL](https://azure.microsoft.com/pricing/details/sql-database/)
- [Precios de máquinas virtuales](https://azure.microsoft.com/pricing/details/virtual-machines/) para [SQL](https://azure.microsoft.com/pricing/details/virtual-machines/#sql) y [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/#windows)
- [Calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/)

## <a name="administration"></a>Administración

En muchas empresas, la decisión de pasar a un servicio en la nube está tan relacionada con la posibilidad de reducir la carga de complejidad de administración, como con el costo. Con IaaS y PaaS, Azure administra la infraestructura subyacente y replica automáticamente todos los datos para proporcionar recuperación ante desastres, configura y actualiza el software de base de datos, administra el equilibrio de carga y realiza una conmutación por error transparente en caso de error del servidor en un centro de datos.

- Con **SQL Database** e **Instancia administrada de SQL**, puede seguir administrando la base de datos, pero ya no necesita administrar el motor de la base de datos, el sistema operativo ni el hardware. Las bases de datos y los inicios de sesión, el ajuste de índices y consultas, así como la auditoría y la seguridad, son ejemplos de elementos que puede seguir administrando. Además, la configuración de la alta disponibilidad en otro centro de datos requiere tareas de configuración y administración mínimas.
- Con **SQL en máquinas virtuales de Azure**, tiene un control completo sobre la configuración del sistema operativo y de la instancia de SQL Server. Con una máquina virtual, el usuario decide cuándo actualizar el software del sistema operativo y de la base de datos, y cuándo instalar cualquier otro software adicional; por ejemplo, un antivirus. Se proporcionan algunas características automatizadas para simplificar considerablemente la aplicación de revisiones, las copias de seguridad y la alta disponibilidad. Además, se puede controlar el tamaño de la máquina virtual, el número de discos y sus configuraciones de almacenamiento. Azure permite cambiar el tamaño de una máquina virtual cuando sea necesario. Para obtener más información, consulte [Tamaños de máquinas virtuales](../virtual-machines/windows/sizes.md).

## <a name="service-level-agreement-sla"></a>Acuerdo de Nivel de Servicio (SLA)

Para algunos departamentos de TI, cumplir las obligaciones de tiempo de actividad de un contrato de nivel de servicio (SLA) es una prioridad máxima. En esta sección, analizaremos a qué se aplica un contrato de nivel de servicio para cada opción de hospedaje de base de datos.

En el caso de **Azure SQL Database** e **Instancia administrada de Azure SQL**, Microsoft proporciona un acuerdo de nivel de servicio de disponibilidad del 99,99 %. Para conocer la información más reciente, consulte [Contrato de nivel de servicio](https://azure.microsoft.com/support/legal/sla/sql-database/).

Para **SQL en máquinas virtuales de Azure**, Microsoft proporciona un contrato de nivel de servicio de disponibilidad del 99,95 % que incluye solo la máquina virtual. Este contrato no cubre los procesos (como SQL Server) que se ejecutan en la máquina virtual y requieren que se hospeden como mínimo dos instancias de máquina virtual en un conjunto de disponibilidad. Para obtener la información más reciente, consulte [Contrato de nivel de servicio para Máquinas virtuales](https://azure.microsoft.com/support/legal/sla/virtual-machines/). Para alta disponibilidad (HA) de base de datos en las máquinas virtuales, se debe configurar una de las opciones de alta disponibilidad admitidas en SQL Server, como [Grupos de disponibilidad AlwaysOn](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server). El uso de una opción de alta disponibilidad no proporciona un SLA adicional, pero le permite lograr una disponibilidad de base de datos del 99,99 %.

## <a name="time-to-move-to-azure"></a><a name="market"></a>Es hora de cambiar a Azure

**Azure SQL Database** es la solución adecuada para las aplicaciones diseñadas para la nube cuando la productividad del desarrollador y un plazo de comercialización rápido de las nuevas soluciones son factores críticos. Con una funcionalidad de tipo DBA mediante programación, resulta perfecto para arquitectos y desarrolladores en la nube puesto que reduce la necesidad de administrar el sistema operativo y la base de datos subyacentes.

**Instancia administrada de Azure SQL**  simplifica en gran medida la migración de aplicaciones existentes a Azure, lo que le permite comercializar aplicaciones de base de datos migradas rápidamente en Azure.

**SQL en máquinas virtuales de Azure** es perfecto si las aplicaciones nuevas o existentes requieren bases de datos grandes o acceso a todas las características de SQL Server o Windows/Linux, y quiere ahorrarse el tiempo y el dinero de adquirir nuevo hardware local. También es una buena opción si quiere migrar aplicaciones y bases de datos locales existentes a Azure tal cual, en casos en que SQL Database o Instancia administrada de SQL no sea una opción adecuada. Dado que no es necesario cambiar los niveles de presentación, aplicación y datos, se ahorra tiempo y presupuesto en renovar la arquitectura de la solución existente. En su lugar, puede centrarse en migrar todas las soluciones a Azure y realizar algunas optimizaciones del rendimiento que requiere la plataforma de Azure. Para obtener más información, consulte [Procedimientos recomendadas para mejorar el rendimiento para SQL Server en Azure Virtual Machines](virtual-machines/windows/performance-guidelines-best-practices.md).

[!INCLUDE [sql-database-create-manage-portal](includes/sql-database-create-manage-portal.md)]

## <a name="next-steps"></a>Pasos siguientes

- Consulte [cómo comenzar por primera vez con Azure SQL Database](database/single-database-create-quickstart.md).
- Consulte [Primera Instancia administrada de Azure SQL](managed-instance/instance-create-quickstart.md) para empezar a trabajar con Instancia administrada de SQL. 
- Consulte [Precio de SQL Database](https://azure.microsoft.com/pricing/details/sql-database/).
- Consulte [Aprovisionamiento de una máquina virtual de SQL Server en Azure](virtual-machines/windows/create-sql-vm-portal.md) para una introducción a SQL Server en máquinas virtuales de Azure.
- [Identificación de la SKU de instancia administrada de SQL o SQL Database adecuada para la base de datos local](/sql/dma/dma-sku-recommend-sql-db/).
