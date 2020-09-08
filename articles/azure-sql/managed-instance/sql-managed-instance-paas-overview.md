---
title: ¿Qué es Instancia administrada de Azure SQL?
description: Obtenga información sobre cómo Azure SQL Managed Instance proporciona una compatibilidad cercana al 100 % con el motor de base de datos de SQL Server (Enterprise Edition) más reciente
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: bonova
ms.author: bonova
ms.reviewer: sstein, carlrab, vanto
ms.date: 08/14/2020
ms.openlocfilehash: 72d0745e5a885ddbc57a9a849a7537a40e0b1215
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88590071"
---
# <a name="what-is-azure-sql-managed-instance"></a>¿Qué es Instancia administrada de Azure SQL?
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Azure SQL Managed Instance es un servicio de base de datos en la nube inteligente y escalable que combina la mayor compatibilidad con el motor de base de datos de SQL Server y todas las ventajas de una plataforma como servicio totalmente administrada y permanente. SQL Managed Instance tiene casi un 100 % de compatibilidad con el motor de base de datos más reciente de SQL Server (Enterprise Edition), lo que proporciona una implementación nativa de la [red virtual (VNet)](../../virtual-network/virtual-networks-overview.md) que permite solucionar problemas de seguridad habituales, y un [modelo de negocio](https://azure.microsoft.com/pricing/details/sql-database/) favorable para los clientes existentes de SQL Server. Instancia administrada de SQL permite a los clientes existentes de SQL Server migrar mediante lift-and-shift sus aplicaciones locales a la nube con cambios mínimos en la aplicación y la base de datos. Al mismo tiempo, SQL Managed Instance conserva todas las funcionalidades de PaaS (aplicación de revisiones y actualizaciones de versión automáticas, [copias de seguridad automáticas](../database/automated-backups-overview.md), [alta disponibilidad](../database/high-availability-sla.md)), que reducen enormemente la sobrecarga de administración y el costo total de propiedad.

> [!IMPORTANT]
> Para una lista de regiones en las que SQL Managed Instance está actualmente disponible, consulte las [regiones admitidas](resource-limits.md#supported-regions).

En el diagrama siguiente se describen las características principales de SQL Managed Instance:

![Principales características](./media/sql-managed-instance-paas-overview/key-features.png)

Azure SQL Managed Instance está diseñada para aquellos clientes que quieran migrar un gran número de aplicaciones desde un entorno local o de IaaS, compilado automáticamente o que hayan proporcionado los fabricantes de software independientes, a un entorno en la nube de PaaS totalmente administrado, con el menor esfuerzo de migración posible. Mediante el uso del [Azure Data Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md#create-an-azure-database-migration-service-instance), los clientes pueden migrar mediante lift-and-shift su instancia de SQL Server existente a SQL Managed Instance, que ofrece compatibilidad con SQL Server y aislamiento completo de las instancias de cliente con compatibilidad nativa con redes virtuales.  Con Software Assurance, puede intercambiar sus licencias existentes para obtener descuentos en SQL Managed Instance mediante la [Ventaja híbrida de Azure para SQL Server](https://azure.microsoft.com/pricing/hybrid-benefit/). SQL Managed Instance es el mejor destino de migración en la nube para instancias de SQL Server que requieren alta seguridad y una completa superficie de programación.

## <a name="key-features-and-capabilities"></a>Funcionalidades y características clave

SQL Managed Instance combina las mejores características que están disponibles tanto en Azure SQL Database como en el motor de base de datos de SQL Server.

> [!IMPORTANT]
> Instancia administrada de SQL se ejecuta con todas las características de la última versión de SQL Server, incluidas las operaciones en línea, las correcciones automáticas del plan y otras mejoras de rendimiento empresarial. Se explica una comparación de las características disponibles en [Comparación de características: Instancia administrada de Azure SQL frente a SQL Server](../database/features-comparison.md).

| **Ventajas de PaaS** | **Continuidad del negocio** |
| --- | --- |
|No hay compras de hardware ni administración <br>Ninguna sobrecarga de administración a la hora de administrar la infraestructura subyacente <br>Aprovisionamiento y escalado de servicio rápidos <br>Aplicación de revisiones y actualización de versiones automatizadas <br>Integración con otros servicios de datos de PaaS |Acuerdo de Nivel de Servicio de tiempo de actividad del 99,99 %  <br>[Alta disponibilidad](../database/high-availability-sla.md) integrada <br>Datos protegidos con [copias de seguridad automatizadas](../database/automated-backups-overview.md) <br>Período de retención de copia de seguridad configurable por el cliente <br>[Copias de seguridad](https://docs.microsoft.com/sql/t-sql/statements/backup-transact-sql?view=azuresqldb-mi-current) que haya iniciado el usuario <br>Funcionalidad de [restauración de base de datos a un momento dado](../database/recovery-using-backups.md#point-in-time-restore) |
|**Seguridad y cumplimiento normativo** | **Administración**|
|Entorno aislado ([integración con red virtual](connectivity-architecture-overview.md), servicio de inquilino único y procesos y almacenamiento dedicados) <br>[Cifrado de datos transparente (TDE)](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)<br>[Autenticación de Azure Active Directory (Azure AD)](../database/authentication-aad-overview.md), compatibilidad con el inicio de sesión único <br> <a href="/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current">Entidades de seguridad (inicios de sesión) de un servidor de Azure AD</a>  <br>Se adhiere a las mismas normas de cumplimiento estándar que Azure SQL Database <br>[Auditoría de SQL](auditing-configure.md) <br>[Advanced Threat Protection](threat-detection-configure.md) |API de Azure Resource Manager para automatizar el aprovisionamiento y escalado del servicio <br>Funcionalidad de Azure Portal para el aprovisionamiento y escalado manuales del servicio <br>Data Migration Service

> [!IMPORTANT]
> Instancia administrada de Azure SQL ha obtenido la certificación de diversas normas de cumplimiento. Para más información, consulte las [Ofertas de cumplimiento de Microsoft Azure](https://servicetrust.microsoft.com/ViewPage/MSComplianceGuideV3?command=Download&downloadType=Document&downloadId=44bbae63-bf4d-4e3b-9d3d-c96fb25ec363&tab=7027ead0-3d6b-11e9-b9e1-290b1eb4cdeb&docTab=7027ead0-3d6b-11e9-b9e1-290b1eb4cdeb_FAQ_and_White_Papers), donde encontrará la lista más reciente de certificaciones de cumplimiento de SQL Managed Instance, en **SQL Database**.

Las características principales de Instancia administrada de SQL se muestran en la tabla siguiente:

|Característica | Descripción|
|---|---|
| Versión/compilación de SQL Server | Motor de base de datos de SQL Server (versión estable más reciente) |
| Administración de copias de seguridad automatizadas | Sí |
| Supervisión y métricas integradas de instancias y bases de datos | Sí |
| Aplicación automática de revisiones de software | Sí |
| Características del motor de base de datos más recientes | Sí |
| Número de archivos de datos (ROWS) por base de datos | Múltiple |
| Número de archivos de registro (LOG) por base de datos | 1 |
| Redes virtuales: implementación de Azure Resource Manager | Sí |
| Redes virtuales: modelo de implementación clásica de Azure | No |
| Soporte técnico del portal | Sí|
| Integration Service (SSIS) integrado | No: SSIS es una parte de [PaaS de Azure Data Factory](https://docs.microsoft.com/azure/data-factory/tutorial-deploy-ssis-packages-azure) |
| Analysis Service (SSAS) integrado | No: SSAS es un servicio [PaaS](https://docs.microsoft.com/azure/analysis-services/analysis-services-overview) independiente. |
| Reporting Service (SSRS) integrado | No: use [informes paginados de Power BI](https://docs.microsoft.com/power-bi/paginated-reports/paginated-reports-report-builder-power-bi) en su lugar u hospede SSRS en una máquina virtual de Azure. Si bien SQL Managed Instance no puede ejecutar SSRS como servicio, sí puede hospedar las [bases de datos del catálogo de SSRS ](https://docs.microsoft.com/sql/reporting-services/install-windows/ssrs-report-server-create-a-report-server-database#database-server-version-requirements) para un servidor de informes instalado en una máquina virtual de Azure, mediante la autenticación de SQL Server. |
|||

## <a name="vcore-based-purchasing-model"></a>Modelo de compra basado en núcleo virtual

El [modelo de compra basado en núcleo virtual](../database/service-tiers-vcore.md) de Instancia administrada de SQL le ofrece flexibilidad, control, transparencia y facilidad para trasladar sus necesidades de carga de trabajo del entorno local a la nube. Este modelo le permite cambiar los recursos de proceso, memoria y almacenamiento en función de las necesidades de la carga de trabajo. El modelo de núcleos virtuales también permite disfrutar de hasta un 55 % de ahorro con la [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/) para SQL Server.

En el modelo de núcleos virtuales, puede elegir entre distintas generaciones de hardware.

- Las CPU lógicas **Gen4** se basan en procesadores Intel E5-2673 v3 (Haswell) de 2,4 GHz, SSD conectado, núcleos físicos, 7 GB de RAM por núcleo y tamaños de proceso de entre 8 y 24 núcleos virtuales.
- Las CPU lógicas **Gen5** se basan en los procesadores Intel® E5-2673 v4 (Broadwell) de 2,3 GHz, y en los procesadores Intel® SP-8160 (Skylake) e Intel® 8272CL (Cascade Lake) de 2,5 GHz, SSD NVMe rápida, núcleo lógico con Hyper-Threading y tamaños de proceso de entre 4 y 80 núcleos.

Encontrará más información sobre la diferencia entre las generaciones de hardware en el artículo sobre [límites de recursos de Instancia administrada de SQL](resource-limits.md#hardware-generation-characteristics).

## <a name="service-tiers"></a>Niveles de servicio

Instancia administrada de SQL está disponible en dos niveles de servicio:

- **Uso general**: diseñada para aplicaciones con rendimiento y requisitos de latencia de E/S comunes.
- **Crítico para la empresa**: diseñada para aplicaciones con requisitos de latencia baja de E/S y un impacto mínimo subyacente de operaciones de mantenimiento en la carga de trabajo.

Ambos niveles de servicio garantizan una disponibilidad del 99,99 % y le permiten seleccionar el tamaño de almacenamiento y la capacidad de proceso de forma independiente. Para obtener más información sobre la arquitectura de alta disponibilidad de Instancia administrada de Azure SQL, vea [Alta disponibilidad e Instancia administrada de Azure SQL](../database/high-availability-sla.md).

### <a name="general-purpose-service-tier"></a>Nivel de servicio Uso general

En la lista siguiente se describen las características principales del nivel de servicio de uso general:

- Diseñado para la mayoría de las aplicaciones empresariales con requisitos típicos de alto rendimiento
- Azure Blob Storage de alto rendimiento (8 TB)
- [Alta disponibilidad](../database/high-availability-sla.md#basic-standard-and-general-purpose-service-tier-availability) integrada basada en los servicios confiables Azure Blob Storage y [Azure Service Fabric](../../service-fabric/service-fabric-overview.md)

Para más información, consulte [Capa de almacenamiento en el nivel de uso general](https://medium.com/azure-sqldb-managed-instance/file-layout-in-general-purpose-azure-sql-managed-instance-cf21fff9c76c) y [Procedimientos recomendados y consideraciones de rendimiento de almacenamiento para SQL Managed Instance (de uso general)](https://blogs.msdn.microsoft.com/sqlcat/2018/07/20/storage-performance-best-practices-and-considerations-for-azure-sql-db-managed-instance-general-purpose/).

Encontrará más información sobre la diferencia entre los niveles de servicio en el artículo sobre los [límites de recursos de Instancia administrada de SQL](resource-limits.md#service-tier-characteristics).

### <a name="business-critical-service-tier"></a>Nivel de servicio Crítico para la empresa

El nivel de servicio Crítico para la empresa se ha creado para las aplicaciones con elevados requisitos de E/S. Ofrece la máxima resistencia a los errores gracias al uso de varias réplicas aisladas.

La siguiente lista describe las principales características del nivel de servicio Crítico para la empresa:

- Diseñado para aplicaciones empresariales con mayor rendimiento y requisitos de alta disponibilidad
- Incluye almacenamiento local SSD extremadamente rápido (hasta 1 TB en Gen 4 y hasta 4 TB en Gen 5)
- [Alta disponibilidad](../database/high-availability-sla.md#premium-and-business-critical-service-tier-availability) integrada basada en los [Grupos de disponibilidad Always On](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server) y [Azure Service Fabric](../../service-fabric/service-fabric-overview.md)
- [Réplica de base de datos de solo lectura](../database/read-scale-out.md) adicional integrada que se puede usar para informes y otras cargas de trabajo de solo lectura.
- [OLTP en memoria](../in-memory-oltp-overview.md), que se puede usar para la carga de trabajo con requisitos de alto rendimiento.  

Encontrará más información sobre la diferencia entre los niveles de servicio en el artículo sobre los [límites de recursos de SQL Managed Instance](resource-limits.md#service-tier-characteristics).

## <a name="management-operations"></a>Operaciones de administración

Instancia administrada de SQL proporciona operaciones de administración que puede usar para implementar automáticamente instancias administradas nuevas, actualizar las propiedades de una instancia y eliminar instancias que ya no son necesarias. Puede encontrar una explicación detallada de las operaciones de administración en la página [Información general sobre operaciones de administración de instancias administradas](management-operations-overview.md).

## <a name="advanced-security-and-compliance"></a>Conformidad y seguridad avanzada

SQL Managed Instance incluye características de seguridad avanzadas que proporciona la plataforma Azure y el motor de base de datos de SQL Server.

### <a name="security-isolation"></a>Aislamiento de seguridad

SQL Managed Instance proporciona un aislamiento de seguridad adicional a partir de otros inquilinos en la plataforma Azure. El aislamiento de seguridad incluye:

- [Implementación nativa de redes virtuales](connectivity-architecture-overview.md) y conectividad al entorno local mediante Azure ExpressRoute o VPN Gateway.
- En una implementación predeterminada, el punto de conexión de SQL se expone solo mediante una dirección IP privada, lo que permite una conectividad segura desde una nube privada de Azure o desde redes híbridas.
- Inquilino único con infraestructura subyacente dedicada (proceso, almacenamiento).

El diagrama siguiente describe distintas opciones de conectividad para las aplicaciones:

![Alta disponibilidad](./media/sql-managed-instance-paas-overview/application-deployment-topologies.png)  

Para más información sobre la integración con redes virtuales y las exigencias de la directiva de redes en el nivel de subred, vea [arquitectura de red virtual para instancias administradas](connectivity-architecture-overview.md) y [Conexión de la aplicación a una instancia administrada](connect-application-instance.md).

> [!IMPORTANT]
> Coloque varias instancias administradas en la misma subred, dondequiera que lo permitan sus requisitos de seguridad, ya que le proporcionará ventajas adicionales. Colocar instancias en la misma subred simplificará significativamente el mantenimiento de la infraestructura de red y reducirá el tiempo de aprovisionamiento de instancias, ya que el aprovisionamiento de larga duración está asociado con el costo de implementar la primera instancia administrada en una subred.

### <a name="security-features"></a>Características de seguridad

Instancia administrada de Azure SQL proporciona un conjunto de características de seguridad avanzadas que se pueden usar para proteger los datos.

- La [auditoría de Instancia administrada de SQL](auditing-configure.md) realiza un seguimiento de los eventos de bases de datos y los escribe en un archivo de registro de auditoría de su cuenta de Azure Storage. La auditoría puede ayudarle a mantener el cumplimiento de normativas, comprender la actividad de las bases de datos y conocer las discrepancias y anomalías que pueden indicar problemas en el negocio o infracciones de seguridad sospechosas.
- Cifrado de datos en movimiento: SQL Managed Instance protege los datos gracias al cifrado de datos en movimiento mediante la Seguridad de la capa de transporte. Además de la Seguridad de la capa de transporte, SQL Managed Instance ofrece la protección de la información confidencial en tránsito, en reposo y durante el procesamiento de consultas con [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine). Always Encrypted ofrece seguridad de los datos frente a las infracciones que implican el robo de datos críticos. Por ejemplo, con Always Encrypted, los números de las tarjetas de crédito siempre se almacenan cifrados en la base de datos, incluso durante el procesamiento de las consultas, lo que permite que solo los descifren personal autorizado o las aplicaciones que los necesitan para procesar los datos en el lugar en que se van a usar.
- [Advanced Threat Protection](threat-detection-configure.md) complementa la [auditoría](auditing-configure.md), ya que proporciona una capa adicional de inteligencia de seguridad integrada en el servicio que detecta intentos inusuales y potencialmente dañinos para obtener acceso a las bases de datos o vulnerarlas. Recibirá alertas de actividades sospechosas, vulnerabilidades potenciales y ataques por inyección de código SQL, así como patrones anómalos de acceso a bases de datos. Las alertas de Advanced Threat Protection se pueden ver en [Azure Security Center](https://azure.microsoft.com/services/security-center/). Proporcionan detalles de cualquier actividad sospechosa y recomiendan las acciones necesarias para investigar y mitigar la amenaza.  
- El [enmascaramiento dinámico de datos](/sql/relational-databases/security/dynamic-data-masking) limita la exposición de información confidencial mediante su enmascaramiento a los usuarios sin privilegios. El enmascaramiento dinámico de datos ayuda a impedir el acceso no autorizado a datos confidenciales permitiéndole designar la cantidad de los datos confidenciales que se revelarán con un impacto mínimo en el nivel de aplicación. Se trata de una característica de seguridad basada en directivas que oculta la información confidencial del conjunto de resultados de una consulta de campos designados de una base de datos, sin modificar los datos de esta última.
- La [seguridad de nivel de fila](/sql/relational-databases/security/row-level-security) (RLS) le permite controlar el acceso a las filas de una tabla de base de datos en función de las características del usuario que ejecuta una consulta (por ejemplo, la pertenencia a un grupo o el contexto de ejecución). RLS simplifica el diseño y la codificación de la seguridad de la aplicación. RLS permite implementar restricciones de acceso a filas de datos. Por ejemplo, garantiza que los empleados únicamente puedan acceder a aquellas filas de datos necesarios para su departamento o restringe el acceso solo a los datos relevantes.
- [Cifrado de datos transparente (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql) cifra los archivos de datos de Instancia administrada de SQL, lo que se conoce como cifrado de datos en reposo. TDE realiza el cifrado y descifrado de E/S en tiempo real de los archivos de datos y de registro. El cifrado usa una clave de cifrado de base de datos (DEK), que se almacena en el registro de arranque de la base de datos de disponibilidad durante la recuperación. Puede proteger todas las bases de datos en una instancia administrada con cifrado de datos transparente. TDE es la probada tecnología de cifrado en reposo de SQL Server que requieren muchos estándares de cumplimiento normativo para proteger contra el robo de soportes de almacenamiento.

Se admite la migración de una base de datos cifrada a SQL Managed Instance a través de Azure Database Migration Service o la restauración nativa. Si va a migrar una base de datos cifrada mediante la restauración nativa, la migración del certificado TDE existente desde la instancia de SQL Server a SQL Managed Instance es un paso necesario. Para obtener más información sobre las opciones de migración, vea [Migración de SQL Server a Instancia administrada de SQL](migrate-to-instance-from-sql-server.md).

## <a name="azure-active-directory-integration"></a>Integración de Azure Active Directory

SQL Managed Instance admite los inicios de sesión tradicionales del motor de base de datos de SQL Server, así como los inicios de sesión integrados con Azure AD. Las entidades de seguridad (inicios de sesión) de un servidor de Azure AD (**versión preliminar pública**) son una versión en la nube de Azure de los inicios de sesión de la base de datos local que está utilizando en su entorno local. Las entidades de seguridad (inicios de sesión) de un servidor de Azure AD le permiten especificar usuarios y grupos del inquilino de Azure AD como entidades de seguridad de la instancia con capacidad para llevar a cabo operaciones dentro del ámbito de la misma, incluidas consultas entre bases de datos dentro de la misma instancia administrada.

Se ha incluido una nueva sintaxis para crear entidades de seguridad (inicios de sesión) de un servidor de Azure AD, **FROM EXTERNAL PROVIDER**. Para más información sobre la sintaxis, consulte <a href="/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current">CREATE LOGIN</a> y revise el artículo [Aprovisionamiento de un administrador de Azure Active Directory para SQL Managed Instance](../database/authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance).

### <a name="azure-active-directory-integration-and-multi-factor-authentication"></a>Integración de Azure Active Directory y autenticación multifactor

SQL Managed Instance permite administrar de forma centralizada las identidades de los usuarios de base de datos y otros servicios de Microsoft con la [integración de Azure Active Directory](../database/authentication-aad-overview.md). Esta funcionalidad simplifica la administración de permisos y mejora la seguridad. Azure Active Directory admite [autenticación multifactor](../database/authentication-mfa-ssms-configure.md) para aumentar la seguridad de los datos y de la aplicación, al tiempo que admite un proceso de inicio de sesión único.

### <a name="authentication"></a>Authentication

La autenticación de Instancia administrada de SQL indica la forma en que los usuarios prueban su identidad al conectarse a la base de datos. Instancia administrada de SQL admite dos tipos de autenticación:  

- **Autenticación de SQL**:

  Este método de autenticación utiliza un nombre de usuario y una contraseña.
- **Autenticación con Azure Active Directory**:

  Este método de autenticación usa las identidades administradas por Azure Active Directory y es compatible con dominios administrados e integrados. Use la autenticación de Active Directory (seguridad integrada) [siempre que sea posible](https://docs.microsoft.com/sql/relational-databases/security/choose-an-authentication-mode).

### <a name="authorization"></a>Authorization

La autorización indica las acciones que pueden llevar a cabo los usuarios en una base de datos de Instancia administrada de Azure SQL, algo que controlan los permisos de nivel de objeto y las pertenencias a roles de bases de datos de la cuenta de usuario. SQL Managed Instance tiene las mismas funcionalidades de autorización que SQL Server 2017.

## <a name="database-migration"></a>Migración de bases de datos

Instancia administrada de SQL está diseñada para escenarios de usuario con migración masiva de bases de datos desde implementaciones locales o de base de datos de IaaS. Instancia administrada de SQL admite varias opciones de migración de base de datos:

### <a name="backup-and-restore"></a>Copia de seguridad y restauración  

El enfoque de migración aprovecha las copias de seguridad de SQL en Azure Blob Storage. Las copias de seguridad almacenadas en una instancia de Azure Storage Blob se pueden restaurar directamente en una instancia administrada mediante el [comando T-SQL RESTORE](https://docs.microsoft.com/sql/t-sql/statements/restore-statements-transact-sql?view=azuresqldb-mi-current).

- Para ver una guía rápida que muestra cómo restaurar el archivo de copia de seguridad de base de datos de Wide World Importers - Standard, vea [Restore a backup file to a Managed Instance](restore-sample-database-quickstart.md) (Restauración de un archivo de copia de seguridad a una instancia administrada). En esta guía rápida se muestra cómo cargar un archivo de copia de seguridad en una instancia de Azure Blob Storage y protegerlo mediante una clave de Firma de acceso compartido (SAS).
- Para obtener información sobre la restauración desde una URL, vea [Restauración de copias de seguridad nativas desde la dirección URL](migrate-to-instance-from-sql-server.md#native-restore-from-url).

> [!IMPORTANT]
> Solo se pueden restaurar las copias de seguridad de una instancia administrada en otra instancia administrada. No se pueden restaurar en una instancia de SQL Server o en Azure SQL Database.

### <a name="database-migration-service"></a>Database Migration Service

Azure Database Migration Service es un servicio totalmente administrado diseñado para permitir migraciones completas desde varios orígenes de base de datos hasta las plataformas de datos de Azure con un tiempo de inactividad mínimo. Este servicio simplifica las tareas necesarias para mover bases de datos existentes de SQL Server y de terceros a Azure SQL Database, a Azure SQL Managed Instance y a SQL Server en las máquinas virtuales de Azure. Vea cómo [migrar su base de datos local a Instancia administrada de SQL mediante Database Migration Service](https://aka.ms/migratetoMIusingDMS).

## <a name="sql-features-supported"></a>Características de SQL admitidas

Instancia administrada de SQL quiere proporcionar una compatibilidad de área expuesta cercana al 100 % con la última versión de SQL Server mediante un plan de lanzamiento gradual. Para obtener una lista de características y comparaciones, vea [Comparación de características de Instancia administrada de SQL](../database/features-comparison.md). Para obtener una lista de diferencias de T-SQL comparando Instancia administrada de SQL y SQL Server, vea [Diferencias de T-SQL entre Instancia administrada de SQL y SQL Server](transact-sql-tsql-differences-sql-server.md).

SQL Managed Instance admite versiones anteriores hasta bases de datos de SQL Server 2008. Se admite la migración directa de servidores de base de datos de SQL Server 2005 y el nivel de compatibilidad con las bases de datos de SQL Server 2005 migradas se actualiza a SQL Server 2008.
  
El siguiente diagrama describe la compatibilidad de área expuesta en Instancia administrada de SQL:  

![Migración](./media/sql-managed-instance-paas-overview/migration.png)

### <a name="key-differences-between-sql-server-on-premises-and-sql-managed-instance"></a>Diferencias principales entre SQL Server local y SQL Managed Instance

SQL Managed Instance tiene la ventaja de estar siempre actualizada en la nube, lo cual significa que algunas características de SQL Server pueden ser obsoletas, estar retiradas o presentar alternativas. Hay casos concretos en los que las herramientas necesitan reconocer que una característica determinada funciona de forma ligeramente diferente o que el servicio se ejecuta en un entorno que no se controla totalmente.

Algunas diferencias clave:

- La alta disponibilidad está integrada y preconfigurada con tecnología similar a los [Grupos de disponibilidad Always On](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server).
- Solo hay copias de seguridad automatizadas y restauración a un momento dado. Los clientes pueden iniciar copias de seguridad `copy-only` que no interfieren con la cadena de copias de seguridad automáticas.
- No se permite especificar rutas de acceso físicas completas, por lo que todos los escenarios correspondientes tienen que ser compatibles de manera diferente: RESTORE DB no es compatible con WITH MOVE, CREATE DB no permite rutas de acceso físicas, BULK INSERT funciona solo con Azure Blobs, etc.
- SQL Managed Instance admite la [autenticación de Azure AD](../database/authentication-aad-overview.md) como alternativa en la nube a la autenticación de Windows.
- SQL Managed Instance administra automáticamente el grupo de archivos XTP y los archivos de bases de datos que contienen objetos OLTP en memoria.
- SQL Managed Instance es compatible con SQL Server Integration Services (SSIS) y puede hospedar un catálogo de SSIS (SSISDB) que almacena paquetes SSIS, pero se ejecutan en una instancia administrada de Azure-SSIS Integration Runtime (IR) en Azure Data Factory (ADF). Consulte [Creación de una instancia de Azure-SSIS IR en Azure Data Factory](https://docs.microsoft.com/azure/data-factory/create-azure-ssis-integration-runtime). Para comparar las características de SSIS, vea una [comparación entre SQL Database con Instancia administrada de SQL](../../data-factory/create-azure-ssis-integration-runtime.md#comparison-of-sql-database-and-sql-managed-instance).

### <a name="administration-features"></a>Características de administración

SQL Managed Instance permite al administrador del sistema dedicar menos tiempo a las tareas administrativas, ya que el servicio las realiza automáticamente o las simplifica en gran medida. Por ejemplo, la [instalación y aplicación de revisiones de SO o RDBMS](../database/high-availability-sla.md), las [opciones dinámicas de cambio de tamaño y configuración de instancias](../database/single-database-scale.md), las [copias de seguridad](../database/automated-backups-overview.md), la [replicación de bases de datos](replication-between-two-instances-configure-tutorial.md) (incluidas las bases de datos del sistema), la [configuración de la alta disponibilidad](../database/high-availability-sla.md) y los flujos de datos de configuración del estado y [supervisión del rendimiento](../../azure-monitor/insights/azure-sql.md).

Para más información, consulte una [lista de características admitidas y no admitidas de SQL Managed Instance](../database/features-comparison.md) y las [diferencias de T-SQL entre SQL Managed Instance y SQL Server](transact-sql-tsql-differences-sql-server.md).

### <a name="programmatically-identify-a-managed-instance"></a>Identificación mediante programación de una instancia administrada

En la tabla siguiente se muestran varias propiedades, accesibles mediante Transact-SQL, que se pueden usar para detectar si la aplicación funciona con Instancia administrada de SQL y recuperar propiedades importantes.

|Propiedad|Value|Comentario|
|---|---|---|
|`@@VERSION`|Microsoft SQL Azure (RTM) - 12.0.2000.8 2018-03-07 Copyright (C) 2018 Microsoft Corporation.|Este valor es igual que en SQL Database. Esto **no** indica la versión 12 del motor de SQL (SQL Server 2014). Instancia administrada de SQL siempre ejecuta la última versión estable del motor de SQL, que es igual o mayor que la última versión de RTM disponible de SQL Server.  |
|`SERVERPROPERTY ('Edition')`|SQL Azure|Este valor es igual que en SQL Database.|
|`SERVERPROPERTY('EngineEdition')`|8|Este valor identifica la instancia administrada de forma única.|
|`@@SERVERNAME`, `SERVERPROPERTY ('ServerName')`|Nombre DNS completo de la instancia en el siguiente formato:`<instanceName>`.`<dnsPrefix>`.database.windows.net, donde `<instanceName>` es el nombre que proporciona el cliente, mientras que `<dnsPrefix>` es la parte que se genera automáticamente del nombre, lo cual permite garantizar la exclusividad del nombre DNS global (por ejemplo, "wcus17662feb9ce98").|Por ejemplo: my-managed-instance.wcus17662feb9ce98.database.windows.net|

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre cómo crear su primera instancia administrada, vea la [Guía de inicio rápido](instance-create-quickstart.md).
- Para obtener una lista de características y una comparación, consulte [Características comunes de SQL](../database/features-comparison.md).
- Para obtener más información sobre la configuración de redes virtuales, vea [Configuración de una red virtual de Instancia administrada de SQL](connectivity-architecture-overview.md).
- Para ver un inicio rápido en el que se crea una instancia administrada y se restaura una base de datos desde un archivo de copia de seguridad, consulte [Creación de una instancia administrada](instance-create-quickstart.md).
- Para consultar un tutorial sobre el uso de Azure Database Migration Service para la migración, consulte [Migración de Instancia administrada de SQL mediante Database Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md).
- Para obtener información sobre la supervisión avanzada del rendimiento de las bases de datos de Instancia administrada de SQL con inteligencia integrada para la solución de problemas, consulte [Supervisión de instancias de Azure SQL Database con Azure SQL Analytics](../../azure-monitor/insights/azure-sql.md).
- Para información sobre los precios, consulte [Precios de SQL Database](https://azure.microsoft.com/pricing/details/sql-database/managed/).
