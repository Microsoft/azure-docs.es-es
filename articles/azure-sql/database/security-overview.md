---
title: Información general sobre seguridad
titleSuffix: Azure SQL Database & Azure SQL Managed Instance
description: Obtenga información sobre la seguridad de Azure SQL Database e Instancia administrada de Azure SQL, lo que incluye cómo difiere de SQL Server.
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: jaszymas
ms.author: jaszymas
ms.reviewer: vanto, emlisa
ms.date: 09/21/2020
ms.openlocfilehash: f3ae5e1ef4dc2968724daeafb32f26cf445b0d2f
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90885303"
---
# <a name="an-overview-of-azure-sql-database-and-sql-managed-instance-security-capabilities"></a>Información general sobre las capacidades de seguridad de Azure SQL Database e Instancia administrada de SQL
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

En este artículo se detallan los fundamentos de la protección de la capa de datos de una aplicación con [Azure SQL Database](sql-database-paas-overview.md) e [Instancia administrada de Azure SQL](../managed-instance/sql-managed-instance-paas-overview.md). La estrategia de seguridad descrita sigue el enfoque por capas de defensa en profundidad, como se muestra en la siguiente imagen, y se mueve desde el exterior hacia el centro:

![sql security layer.png](./media/security-overview/sql-security-layer.png)

## <a name="network-security"></a>Seguridad de las redes

Microsoft Azure SQL Database e Instancia administrada de SQL ofrecen un servicio de base de datos relacional para aplicaciones empresariales y en la nube. Para ayudar a proteger los datos del cliente, los firewalls evitan el acceso de red al servidor hasta que se concede acceso explícitamente según la dirección IP o el origen del tráfico de red virtual de Azure.

### <a name="ip-firewall-rules"></a>Reglas de firewall de IP

Las reglas de firewall de IP otorgan acceso a las bases de datos según la dirección IP de origen de cada solicitud. Para más información, consulte [Introducción a las reglas de firewall de Azure SQL Database y Azure Synapse Analytics](firewall-configure.md).

### <a name="virtual-network-firewall-rules"></a>Reglas de firewall de red virtual

Los [puntos de conexión del servicio de redes virtuales](../../virtual-network/virtual-network-service-endpoints-overview.md) amplía la conectividad de red virtual a través de la red troncal de Azure y permite que Azure SQL Database identifique la subred de la red virtual desde la que se origina el tráfico. Para permitir que el tráfico llegue a Azure SQL Database, use las [etiquetas de servicio](../../virtual-network/security-overview.md) de SQL para permitir el tráfico saliente a través de grupos de seguridad de red.

Las [reglas de red virtual](vnet-service-endpoint-rule-overview.md) permiten que Azure SQL Database solo acepte comunicaciones que se envían desde subredes seleccionadas en una red virtual.

> [!NOTE]
> El control de acceso con reglas de firewall *no* se aplica a **Instancia administrada de SQL**. Para más información sobre la configuración de red necesaria, consulte [Conexión a una instancia administrada](../managed-instance/connect-application-instance.md).

## <a name="access-management"></a>Administración de acceso

> [!IMPORTANT]
> La administración de bases de datos y servidores en Azure se controla mediante las asignaciones de roles de su cuenta de usuario del portal. Para obtener más información sobre este artículo, consulte [Control de acceso basado en rol en Azure Portal](../../role-based-access-control/overview.md).

### <a name="authentication"></a>Authentication

La autenticación es el proceso por el cual se demuestra que el usuario es quien dice ser. Azure SQL Database e Instancia administrada de SQL admiten dos tipos de autenticación:

- **Autenticación de SQL**:

    La autenticación de SQL hace referencia a la autenticación de un usuario al conectarse a Azure SQL Database o Instancia administrada de Azure SQL con el nombre de usuario y la contraseña. Cuando se crea el servidor, se debe especificar un inicio de sesión de **administrador de servidor** con un nombre de usuario y una contraseña. Con estas credenciales, un **administrador de servidor** puede autenticarse en cualquier base de datos en ese servidor o instancia como propietario de la base de datos. Después de eso, pueden crearse inicios de sesión SQL y usuarios adicionales mediante el administrador del servidor, lo que permite a los usuarios conectarse usando el nombre de usuario y contraseña.

- **Autenticación de Azure Active Directory**:

    la autenticación de Azure Active Directory es un mecanismo de conexión a [Azure SQL Database](sql-database-paas-overview.md), [Instancia administrada de Azure SQL](../managed-instance/sql-managed-instance-paas-overview.md) y [Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) mediante identidades de Azure Active Directory (Azure AD). La autenticación de Azure AD permite a los administradores administrar centralmente las identidades y los permisos de los usuarios de la base de datos, junto con otros servicios de Azure, en una ubicación central. Esto incluye la minimización de almacenamiento de contraseñas y permite directivas centralizadas de rotación de contraseñas.

     Debe crearse un administrador del servidor denominado **Administrador de Active Directory** para usar la autenticación de Azure AD con SQL Database. Para más información, consulte [Usar la autenticación de Azure Active Directory para autenticación con SQL](authentication-aad-overview.md). La autenticación de Azure AD admite cuentas tanto administradas como federadas. Las cuentas federadas admiten usuarios y grupos de Windows para un dominio de cliente federado con Azure AD.

    Las opciones adicionales de autenticación de Azure AD disponibles son conexiones de [autenticación universal con SQL Server Management Studio](authentication-mfa-ssms-overview.md), incluidas [autenticación multifactor](../../active-directory/authentication/concept-mfa-howitworks.md) y [acceso condicional](conditional-access-configure.md).

> [!IMPORTANT]
> La administración de bases de datos y servidores en Azure se controla mediante las asignaciones de roles de su cuenta de usuario del portal. Para obtener más información sobre este artículo, consulte [Introducción al control de acceso basado en roles en Azure Portal](../../role-based-access-control/overview.md). El control de acceso con reglas de firewall *no* se aplica a **Instancia administrada de SQL**. Para más información acerca de la configuración de red necesaria, consulte el artículo siguiente sobre cómo [conectarse a una instancia administrada](../managed-instance/connect-application-instance.md).

## <a name="authorization"></a>Authorization

La autorización hace referencia a los permisos asignados a un usuario dentro de una base de datos de Azure SQL Database o Instancia administrada de Azure SQL, y determina qué puede hacer el usuario. Los permisos se controlan mediante la adición de cuentas de usuario a [roles de base de datos](/sql/relational-databases/security/authentication-access/database-level-roles) y la asignación de permisos de nivel de base de datos a estos roles o concediendo al usuario determinados [permisos de nivel de objeto](/sql/relational-databases/security/permissions-database-engine). Para más información, consulte [Inicios de sesión y usuarios](logins-create-manage.md).

Como procedimiento recomendado, cree roles personalizados cuando sea necesario. Agregue usuarios al rol con los privilegios mínimos necesarios para realizar su función de trabajo. No asigne permisos directamente a los usuarios. La cuenta de administrador del servidor es un miembro del rol db_owner integrado, que tiene amplios permisos y se debe conceder solo a pocos usuarios con responsabilidades administrativas. En las aplicaciones, use [EXECUTE AS](/sql/t-sql/statements/execute-as-clause-transact-sql) para especificar el contexto de ejecución del módulo llamado o use [Roles de la aplicación](/sql/relational-databases/security/authentication-access/application-roles) con permisos limitados. Esta práctica garantiza que la aplicación que se conecta a la base de datos tenga los privilegios mínimos necesarios para la aplicación. Seguir estos procedimientos recomendados también fomenta la separación de responsabilidades.

### <a name="row-level-security"></a>Seguridad de nivel de fila

La seguridad de nivel de fila permite a los clientes controlar el acceso a las filas de una tabla de base de datos según las características del usuario que ejecuta una consulta (por ejemplo, la pertenencia a grupos o el contexto de ejecución). La seguridad de nivel de fila también puede utilizarse para implementar los conceptos de seguridad basados en etiquetas personalizados. Para más información, consulte [Seguridad de nivel de fila](/sql/relational-databases/security/row-level-security).

![azure database rls.png](./media/security-overview/azure-database-rls.png)

## <a name="threat-protection"></a>Protección contra amenazas

SQL Database e Instancia administrada de SQL protegen los datos de los clientes al ofrecer capacidades de auditoría y detección de amenazas.

### <a name="sql-auditing-in-azure-monitor-logs-and-event-hubs"></a>Auditoría de SQL en los registros de Azure Monitor y Event Hubs

La auditoría de SQL Database e Instancia administrada de SQL hace un seguimiento de las actividades de la base de datos y ayuda a mantener el cumplimiento de los estándares de seguridad mediante la grabación de eventos de la base de datos en un registro de auditoría de una cuenta de almacenamiento de Azure propiedad del cliente. La auditoría permite a los usuarios supervisan las actividades de la base de datos en curso, así como analizar e investigar la actividad histórica para identificar posibles amenazas o supuestas infracciones de seguridad y abusos. Para más información, consulte la introducción a la [auditoría de base de datos de SQL](../../azure-sql/database/auditing-overview.md).  

### <a name="advanced-threat-protection"></a>Protección contra amenazas avanzada

Advanced Threat Protection analiza los registros para detectar un comportamiento poco habitual e intentos potencialmente peligrosos de acceder o aprovechar las bases de datos. Las alertas se crean para detectar actividades sospechosas, como inyección de código SQL, infiltración potencial de datos y ataques de fuerza bruta, o anomalías en los patrones de acceso para detectar elevaciones de privilegios y uso de credenciales vulneradas. Las alertas se ven desde [Azure Security Center](https://azure.microsoft.com/services/security-center/), donde se proporcionan detalles de las actividades sospechosas y se dan recomendaciones para una investigación más minuciosa, junto con las acciones para mitigar la amenaza. La protección contra amenazas avanzada se puede habilitar por servidor, bajo una cuota adicional. Para más información, vea [Introducción a la protección de amenazas avanzadas de SQL Database](threat-detection-configure.md).

![azure database td.jpg](./media/security-overview/azure-database-td.jpg)

## <a name="information-protection-and-encryption"></a>Protección y cifrado de información

### <a name="transport-layer-security-encryption-in-transit"></a>Seguridad de la capa de transporte (cifrado en tránsito)

SQL Database e Instancia administrada de SQL protegen los datos de los clientes mediante el cifrado de datos en movimiento con [Seguridad de la capa de transporte (TLS)](https://support.microsoft.com/help/3135244/tls-1-2-support-for-microsoft-sql-server).

SQL Database e Instancia administrada de SQL aplican el cifrado (SSL/TLS) en todo momento para todas las conexiones. Esto garantiza que todos los datos se cifran "en tránsito" entre el cliente y el servidor independientemente de la configuración de **Encrypt** o **TrustServerCertificate** en la cadena de conexión.

Como procedimiento recomendado, en la cadena de conexión usada por la aplicación, especifique una conexión cifrada y _**no**_ confíe en el certificado de servidor. Esto obliga a la aplicación a comprobar el certificado de servidor y, por tanto, impide que la aplicación sea vulnerable a ataques de tipo "Man in the middle".

Por ejemplo, cuando se utiliza el controlador ADO.NET, esto se logra a través de **Encrypt=True** y **TrustServerCertificate=False**. Si obtiene la cadena de conexión en Azure Portal, tendrá la configuración correcta.

> [!IMPORTANT]
> Tenga en cuenta que algunos controladores que no son de Microsoft pueden no usar TLS de forma predeterminada o basarse en una versión anterior de TLS (<1.2) para poder funcionar. En este caso, el servidor sigue permitiendo conectarse a la base de datos. Pero recomendamos que evalúe los riesgos de seguridad de permitir que estos controladores y aplicaciones se conecten a SQL Database, especialmente si almacena datos confidenciales.
>
> Para obtener información adicional sobre TLS y la conectividad, vea [Consideraciones de TLS](connect-query-content-reference-guide.md#tls-considerations-for-database-connectivity)

### <a name="transparent-data-encryption-encryption-at-rest"></a>Cifrado de datos transparente (cifrado en reposo)

[Cifrado de datos transparente (TDE) para Azure SQL Database e Instancia administrada de SQL](transparent-data-encryption-tde-overview.md) agrega una capa de seguridad para ayudar a proteger los datos en reposo frente al acceso no autorizado o sin conexión a archivos sin formato o copias de seguridad. Los escenarios habituales incluyen el robo del centro de datos o la eliminación no segura de hardware o medios como unidades de disco y cintas de copia de seguridad. TDE cifra toda la base de datos mediante un algoritmo de cifrado de AES, lo que no requiere que los desarrolladores de aplicaciones hagan cambios en las aplicaciones existentes.

En Azure, todas las bases de datos recién creadas se cifran de forma predeterminada y la clave de cifrado de la base de datos se protege mediante un certificado de servidor integrado.  El servicio administra el mantenimiento y la rotación de certificados, y no se requiere ninguna acción por parte del usuario. Los clientes que prefieren tomar el control de las claves de cifrado pueden administrar las claves en [Azure Key Vault](../../key-vault/general/secure-your-key-vault.md).

### <a name="key-management-with-azure-key-vault"></a>Administración de claves con Azure Key Vault

La compatibilidad de [Bring Your Own Key](transparent-data-encryption-byok-overview.md) (BYOK) con el  [Cifrado de datos transparente](/sql/relational-databases/security/encryption/transparent-data-encryption) (TDE) permite a los clientes asumir la propiedad de la administración y la rotación de claves mediante  [Azure Key Vault](../../key-vault/general/secure-your-key-vault.md), el sistema externo de administración de claves basado en la nube de Azure. Si se revoca el acceso de la base de datos al almacén de claves, una base de datos no se puede descifrar y leer en la memoria. Azure Key Vault ofrece una plataforma de administración central de claves, aprovecha los módulos de seguridad de hardware (HSM) extremadamente supervisados y permite la separación de obligaciones entre la administración de claves y los datos para ayudar a cumplir los requisitos de cumplimiento de seguridad.

### <a name="always-encrypted-encryption-in-use"></a>Always Encrypted (cifrado en uso)

![azure database ae.png](./media/security-overview/azure-database-ae.png)

[Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) es una característica creada para proteger la información confidencial almacenada en columnas específicas de bases de datos (por ejemplo, números de tarjeta de crédito, números de identificación nacional o datos según la _necesidad de conocimiento_). Esto incluye a administradores de bases de datos u otros usuarios con privilegios que tengan autorización para acceder a la base de datos para realizar tareas de administración, pero que no tienen necesidades empresariales de acceder a datos específicos de las columnas cifradas. Los datos están siempre cifrados, lo que significa que los datos cifrados se descifran solo para el procesamiento por parte de las aplicaciones cliente con acceso a la clave de cifrado. La clave de cifrado nunca se expone a SQL Database ni a Instancia administrada de SQL, y se puede almacenar en el [almacén de certificados de Windows](always-encrypted-certificate-store-configure.md) o en [Azure Key Vault](always-encrypted-azure-key-vault-configure.md).

### <a name="dynamic-data-masking"></a>Enmascaramiento de datos dinámicos

![azure database ddm.png](./media/security-overview/azure-database-ddm.png)

El enmascaramiento dinámico de datos limita la exposición de información confidencial ocultándolos a los usuarios sin privilegios. La característica Enmascaramiento dinámico de datos detecta automáticamente datos posiblemente confidenciales en Azure SQL Database e Instancia administrada de SQL y proporciona recomendaciones accionables para enmascarar estos campos, con un impacto mínimo en la capa de aplicación. Su funcionamiento consiste en ocultar los datos confidenciales del conjunto de resultados de una consulta en los campos designados de la base de datos, mientras que los datos de la base de datos no cambian. Para obtener más información, vea [Introducción al enmascaramiento dinámico de datos de SQL Database e Instancia administrada de SQL](dynamic-data-masking-overview.md).

## <a name="security-management"></a>Administración de la seguridad

### <a name="vulnerability-assessment"></a>Evaluación de vulnerabilidades

La [evaluación de vulnerabilidades](sql-vulnerability-assessment.md) es un servicio fácil de configurar que puede detectar, realizar un seguimiento y corregir posibles puntos vulnerables en la base de datos con el objetivo de mejorar de manera proactiva la seguridad general de las bases de datos. La evaluación de vulnerabilidades (VA) forma parte de la oferta Azure Defender for SQL, que es un paquete unificado de funcionalidades de seguridad avanzadas de SQL. Se puede acceder a la evaluación de vulnerabilidades y administrarla a través del portal central de Azure Defender for SQL.

### <a name="data-discovery-and-classification"></a>Clasificación y detección de datos

La clasificación y detección de datos (actualmente en versión preliminar) proporciona capacidades avanzadas integradas en Azure SQL Database e Instancia administrada de SQL para detectar, clasificar, etiquetar y proteger los datos confidenciales de las bases de datos. Las funciones de detección y clasificación de la información confidencial más importante (empresarial, financiera, médica, personal, etc.) desempeñan un rol fundamental en el modo en que se protege la información de su organización. Puede servir como infraestructura para lo siguiente:

- Varios escenarios de seguridad, como la supervisión (auditoría) y las alertas relacionadas con accesos anómalos a información confidencial.
- Controlar el acceso y mejorar la seguridad de las bases de datos que contienen información altamente confidencial.
- Ayudar a cumplir los requisitos de cumplimiento de normas y los estándares relacionados con la privacidad de datos.

Para más información, consulte [Clasificación y detección de datos](data-discovery-and-classification-overview.md).

### <a name="compliance"></a>Cumplimiento normativo

Además de las anteriores características y funcionalidades que pueden ayudar a la aplicación a cumplir distintos requisitos de seguridad, Azure SQL Database también participa en las auditorías regulares y ha obtenido la certificación de una serie de normas de cumplimiento. Para obtener más información, vea el [Centro de confianza de Microsoft Azure](https://www.microsoft.com/trust-center/compliance/compliance-overview), donde encontrará la lista más reciente de certificaciones de cumplimiento de SQL Database.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre el uso de inicios de sesión, cuentas de usuario, roles de base de datos y permisos en SQL Database e Instancia administrada de SQL, vea [Administración de inicios de sesión y cuentas de usuario](logins-create-manage.md).
- Para obtener más información sobre la auditoría de bases de datos, vea [Auditoría](../../azure-sql/database/auditing-overview.md).
- Para obtener más información sobre la detección de amenazas, vea [Detección de amenazas](threat-detection-configure.md).
