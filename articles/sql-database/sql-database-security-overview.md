---
title: Introducción a la seguridad de Azure SQL Database | Microsoft Docs
description: Aprenda sobre la seguridad de Azure SQL Database y SQL Server, incluidas las diferencias entre la nube y un servidor de SQL Server local.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: aliceku
ms.author: aliceku
ms.reviewer: vanto, carlrab, emlisa
manager: craigg
ms.date: 05/14/2019
ms.openlocfilehash: 7916e9493a5d572f844bca23a1dd7806e5fbe572
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65790159"
---
# <a name="an-overview-of-azure-sql-database-security-capabilities"></a>Información general sobre las funcionalidades de seguridad de Azure SQL Database

En este artículo se detallan los fundamentos de la protección de la capa de datos de una aplicación con Azure SQL Database. La estrategia de seguridad descrita sigue el enfoque por capas de defensa en profundidad, como se muestra en la siguiente imagen, y se mueve desde el exterior hacia el centro:

![sql-security-layer.png](media/sql-database-security-overview/sql-security-layer.png)

## <a name="network-security"></a>Seguridad de las redes

Microsoft Azure SQL Database ofrece un servicio de base de datos relacional para la nube y aplicaciones empresariales. Para ayudar a proteger los datos del cliente, los firewalls impiden el acceso de red al servidor de base de datos hasta que se concede explícitamente el acceso según la dirección IP o el origen del tráfico de red virtual de Azure.

### <a name="ip-firewall-rules"></a>Reglas de firewall de IP

Las reglas de firewall de IP otorgan acceso a las bases de datos según la dirección IP de origen de cada solicitud. Para más información, consulte [Reglas de firewall de Azure SQL Database y SQL Data Warehouse](sql-database-firewall-configure.md).

### <a name="virtual-network-firewall-rules"></a>Reglas de firewall de red virtual

Los [puntos de conexión del servicio de redes virtuales](../virtual-network/virtual-network-service-endpoints-overview.md) amplía la conectividad de red virtual a través de la red troncal de Azure y permite que Azure SQL Database identifique la subred de la red virtual desde la que se origina el tráfico. Para permitir que el tráfico llegue a Azure SQL Database, use las [etiquetas de servicio](../virtual-network/security-overview.md) de SQL para permitir el tráfico saliente a través de grupos de seguridad de red.

Las [reglas de red virtual](sql-database-vnet-service-endpoint-rule-overview.md) permiten que Azure SQL Database solo acepte comunicaciones que se envían desde subredes seleccionadas en una red virtual.

> [!NOTE]
> El control de acceso con reglas de firewall *no* se aplica a **una instancia administrada**. Para más información sobre la configuración de red necesaria, consulte [conexión a una instancia administrada](sql-database-managed-instance-connect-app.md)

## <a name="access-management"></a>administración de acceso

> [!IMPORTANT]
> La administración de bases de datos y servidores de bases de datos en Azure se controla mediante las asignaciones de roles de su cuenta de usuario del portal. Para obtener más información sobre este artículo, consulte [Introducción al control de acceso basado en roles en Azure Portal](../role-based-access-control/overview.md).

### <a name="authentication"></a>Authentication

La autenticación es el proceso por el cual se demuestra que el usuario es quien dice ser. Azure SQL Database admite dos tipos de autenticación:

- **Autenticación de SQL**:

    La autenticación de SQL Database hace referencia a la autenticación de un usuario al conectarse a [Azure SQL Database](sql-database-technical-overview.md) con el nombre de usuario y la contraseña. Durante la creación del servidor de base de datos para la base de datos, se debe especificar un inicio de sesión de "administrador del servidor" con un nombre de usuario y contraseña. Con estas credenciales, un "administrador del servidor" puede autenticarse en cualquier base de datos en ese servidor de base de datos como propietario de la base de datos. Después de eso, pueden crearse inicios de sesión SQL y usuarios adicionales mediante el administrador del servidor, lo que permite a los usuarios conectarse usando el nombre de usuario y contraseña.

- **Autenticación de Azure Active Directory**:

    La autenticación de Azure Active Directory es un mecanismo de conexión a [Azure SQL Database](sql-database-technical-overview.md) y [SQL Data Warehouse](../sql-data-warehouse/sql-data-warehouse-overview-what-is.md) mediante identidades de Azure Active Directory (AD). La autenticación de Azure AD permite a los administradores administrar centralmente las identidades y los permisos de los usuarios de la base de datos, junto con otros servicios de Microsoft, en una ubicación central. Esto incluye la minimización de almacenamiento de contraseñas y permite directivas centralizadas de rotación de contraseñas.

     Debe crearse un administrador del servidor denominado **Administrador de Active Directory** para usar la autenticación de Azure AD con SQL Database. Para más información, consulte [Usar la autenticación de Azure Active Directory para autenticación con SQL](sql-database-aad-authentication.md). La autenticación de Azure AD admite cuentas tanto administradas como federadas. Las cuentas federadas admiten usuarios y grupos de Windows para un dominio de cliente federado con Azure AD.

    Las opciones adicionales de autenticación de Azure AD disponibles son conexiones de [autenticación universal con SQL Server Management Studio](sql-database-ssms-mfa-authentication.md), incluidas [autenticación multifactor](../active-directory/authentication/concept-mfa-howitworks.md) y [acceso condicional](sql-database-conditional-access.md).

> [!IMPORTANT]
> La administración de bases de datos y servidores en Azure se controla mediante las asignaciones de roles de su cuenta de usuario del portal. Para obtener más información sobre este artículo, consulte [Introducción al control de acceso basado en roles en Azure Portal](../role-based-access-control/overview.md). El control de acceso con reglas de firewall *no* se aplica a **una instancia administrada**. Para más información acerca de la configuración de red necesaria, consulte el artículo siguiente sobre cómo [conectarse a una instancia administrada](sql-database-managed-instance-connect-app.md).

## <a name="authorization"></a>Autorización

Autorización hace referencia a los permisos asignados a un usuario dentro de una base de datos de Azure SQL, y determina qué puede hacer el usuario. Los permisos se controlan mediante la adición de cuentas de usuario [roles de base de datos](/sql/relational-databases/security/authentication-access/database-level-roles) y asignar permisos de nivel de base de datos a dichas funciones o concediendo al usuario determinado [permisos de nivel de objeto](/sql/relational-databases/security/permissions-database-engine). Para más información, consulte [Inicios de sesión y usuarios](sql-database-manage-logins.md).

Como práctica recomendada, crear funciones personalizadas cuando sea necesario. Agregar usuarios a la función con los privilegios mínimos necesarios para realizar su función de trabajo. No asigne permisos directamente a los usuarios. La cuenta de administrador del servidor es un miembro del rol db_owner integrados, que tiene amplios permisos y solo se debe conceder a algunos usuarios con derechos administrativos. Para las aplicaciones de Azure SQL Database, use el [EXECUTE AS](/sql/t-sql/statements/execute-as-clause-transact-sql) para especificar el contexto de ejecución del módulo llamado o usar [Roles de aplicación](/sql/relational-databases/security/authentication-access/application-roles) con permisos limitados. Esta práctica garantiza que la aplicación que se conecta a la base de datos tiene los privilegios mínimos necesarios para la aplicación. También se sigue estos procedimientos recomendados fomenta la separación de obligaciones.

### <a name="row-level-security"></a>Seguridad de nivel de fila

La seguridad de nivel de fila permite a los clientes controlar el acceso a las filas de una tabla de base de datos en función de las características del usuario que ejecuta una consulta (como, por ejemplo, la pertenencia a un grupo o el contexto de ejecución). Seguridad de nivel de fila también puede utilizarse para implementar los conceptos de seguridad personalizado basado en etiquetas. Para más información, consulte [Seguridad de nivel de fila](/sql/relational-databases/security/row-level-security).

![azure-database-rls.png](media/sql-database-security-overview/azure-database-rls.png)

## <a name="threat-protection"></a>Protección contra amenazas

SQL Database protege los datos de los clientes al ofrecer capacidades de auditoría y detección de amenazas.

### <a name="sql-auditing-in-azure-monitor-logs-and-event-hubs"></a>Auditoría de SQL en los registros de Azure Monitor y Event Hubs

SQL Database Auditing hace un seguimiento de las actividades de la base de datos y ayuda a mantener el cumplimiento de los estándares de seguridad mediante la grabación de eventos de la base de datos en un registro de auditoría en una cuenta de Azure Storage propiedad del cliente. La auditoría permite a los usuarios supervisan las actividades de la base de datos en curso, así como analizar e investigar la actividad histórica para identificar posibles amenazas o supuestas infracciones de seguridad y abusos. Para más información, consulte la introducción a la [auditoría de base de datos de SQL](sql-database-auditing.md).  

### <a name="advanced-threat-protection"></a>Protección contra amenazas avanzada

Protección contra amenazas avanzada está analizando los registros de SQL Server para detectar un comportamiento poco habitual e intentos potencialmente dañinos de acceso o ataque a las bases de datos. Las alertas se crean para detectar actividades sospechosas, como inyección de código SQL, infiltración de datos posibles y fuerza bruta forzar ataques o para detectar anomalías en el acceso de usar patrones para detectar las elevaciones de privilegios y credenciales ha puesto en peligro. Las alertas se ven desde el [Azure Security Center](https://azure.microsoft.com/services/security-center/), donde se proporcionan los detalles de las actividades sospechosas y recomendaciones para una investigación más minuciosa dado junto con las acciones para mitigar la amenaza. Protección contra amenazas avanzada se puede habilitar por servidor para una cuota adicional. Para obtener más información, consulte [empezar a trabajar con protección contra amenazas avanzada de base de datos del SQL](sql-database-threat-detection.md).

![azure-database-td.jpg](media/sql-database-security-overview/azure-database-td.jpg)

## <a name="information-protection-and-encryption"></a>Protección y cifrado de información

### <a name="transport-layer-security-tls-encryption-in-transit"></a>Seguridad de la capa de transporte (TLS) (cifrado en tránsito)

SQL Database protege los datos de los clientes mediante el cifrado de datos en movimiento con [Seguridad de la capa de transporte](https://support.microsoft.com/help/3135244/tls-1-2-support-for-microsoft-sql-server).

SQL Server aplica el cifrado (SSL/TLS) en todo momento para todas las conexiones. Esto garantiza que todos los datos se cifran "en tránsito" entre el cliente y servidor independientemente de la configuración de **Encrypt** o **TrustServerCertificate** en la cadena de conexión.

Como práctica recomendada, se recomienda que, en la conexión de la aplicación de cadena especifica una conexión cifrada y _**no**_ confiar en el certificado de servidor. Esto obliga la la aplicación para comprobar el certificado de servidor y, por tanto, impide que la aplicación sea vulnerable a ataques de tipo de medio de.

Por ejemplo, cuando se utiliza el controlador ADO.NET esto se logra a través de **Encrypt = True** y **TrustServerCertificate = False**. Si obtiene la cadena de conexión de Azure portal, tendrá la configuración correcta.

> [!IMPORTANT]
> Tenga en cuenta que algunos controladores no son de Microsoft no pueden usar TLS de forma predeterminada o se basan en una versión anterior de TLS (< 1.2) para poder funcionar. En este caso SQL Server permite conectarse a la base de datos. Sin embargo, recomendamos que evalúe los riesgos de seguridad de permitir que los controladores y aplicaciones para conectarse a SQL Database, especialmente si se almacenan datos confidenciales. 
>
> Para obtener más información acerca de TLS y conectividad, consulte [consideraciones de TLS](sql-database-connect-query.md#tls-considerations-for-sql-database-connectivity)

### <a name="transparent-data-encryption-encryption-at-rest"></a>Cifrado de datos transparente (cifrado en reposo)

[Cifrado de datos transparente (TDE) para Azure SQL Database](transparent-data-encryption-azure-sql.md) agrega una capa de seguridad para ayudar a proteger los datos en reposo frente al acceso no autorizado o sin conexión a archivos sin formato o copias de seguridad. Entre los escenarios habituales se incluyen el robo del centro de datos o la eliminación no segura de hardware o medios, como unidades de disco y cintas de copia de seguridad. TDE cifra toda la base de datos mediante un algoritmo de cifrado de AES, lo que no requiere que los desarrolladores de aplicaciones hagan cambios en las aplicaciones existentes.

En Azure, todas las bases de datos SQL recién creadas se cifran de forma predeterminada, y la clave de cifrado de la base de datos se protege mediante un certificado de servidor integrado.  El servicio administra el mantenimiento y la rotación de certificados, y no se requiere ninguna acción por parte del usuario. Los clientes que prefieren tomar el control de las claves de cifrado pueden administrar las claves en [Azure Key Vault](../key-vault/key-vault-secure-your-key-vault.md).

### <a name="key-management-with-azure-key-vault"></a>Administración de claves con Azure Key Vault

La compatibilidad de [Bring Your Own Key](transparent-data-encryption-byok-azure-sql.md) (BYOK) para  [Cifrado de datos transparente](/sql/relational-databases/security/encryption/transparent-data-encryption) (TDE) permite a los clientes apropiarse de la administración y la rotación de claves con  [Azure Key Vault](../key-vault/key-vault-secure-your-key-vault.md), el sistema externo de administración de claves basado en la nube de Azure. Si se revoca el acceso de la base de datos al almacén de claves, una base de datos no se puede descifrar y leer en la memoria. Azure Key Vault ofrece una plataforma de administración central de claves, aprovecha los módulos de seguridad de hardware (HSM) extremadamente supervisados y permite la separación de obligaciones entre la administración de claves y los datos para ayudar a cumplir los requisitos de cumplimiento de seguridad.

### <a name="always-encrypted-encryption-in-use"></a>Always Encrypted (cifrado en uso)

![azure-database-ae.png](media/sql-database-security-overview/azure-database-ae.png)

[Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) es una característica creada para proteger la información confidencial almacenada en columnas específicas de bases de datos (por ejemplo, números de tarjeta de crédito, números de identificación nacional o datos según la _necesidad de conocimiento_). Esto incluye a administradores de bases de datos u otros usuarios con privilegios que tengan autorización para acceder a la base de datos para realizar tareas de administración, pero que no tienen necesidades empresariales de acceder a datos específicos de las columnas cifradas. Los datos están siempre cifrados, lo que significa que los datos cifrados se descifran solo para el procesamiento por parte de las aplicaciones cliente con acceso a la clave de cifrado.  La clave de cifrado nunca se expone a SQL y se pueden almacenar en el [almacén de certificados de Windows](sql-database-always-encrypted.md) o en [Azure Key Vault](sql-database-always-encrypted-azure-key-vault.md).

### <a name="dynamic-data-masking"></a>Enmascaramiento de datos dinámicos

![azure-database-ddm.png](media/sql-database-security-overview/azure-database-ddm.png)

El enmascaramiento dinámico de datos de SQL Database limita la exposición de información confidencial mediante su enmascaramiento a los usuarios sin privilegios. La característica Enmascaramiento dinámico de datos detecta automáticamente información potencialmente confidencial en Azure SQL Database y proporciona recomendaciones accionables para enmascarar estos campos, con un impacto mínimo en el nivel de aplicación. Su funcionamiento consiste en ocultar los datos confidenciales del conjunto de resultados de una consulta en los campos designados de la base de datos, mientras que los datos de la base de datos no cambian. Para más información, consulte [Enmascaramiento dinámico de datos de SQL Database](sql-database-dynamic-data-masking-get-started.md).

## <a name="security-management"></a>Administración de la seguridad

### <a name="vulnerability-assessment"></a>Evaluación de vulnerabilidades

La [evaluación de vulnerabilidades](sql-vulnerability-assessment.md) es un servicio fácil de configurar que puede detectar, realizar un seguimiento y corregir posibles puntos vulnerables en la base de datos con el objetivo de mejorar de manera proactiva la seguridad general de las bases de datos. La evaluación de vulnerabilidades (VA) forma parte de la oferta de Advanced Data Security (ADS), que es un paquete unificado para funcionalidades avanzadas de seguridad de SQL. Puede acceder a la evaluación de vulnerabilidades y administrarla a través del portal central de ADS de SQL.

### <a name="data-discovery--classification"></a>Clasificación y detección de datos

La clasificación y detección de datos (actualmente en su versión preliminar) proporciona funcionalidades avanzadas integradas en Azure SQL Database para detectar, clasificar, etiquetar y proteger la información confidencial de las bases de datos. Las funciones de detección y clasificación de la información confidencial más importante (empresarial, financiera, médica, personal, etc.) desempeñan un rol fundamental en el modo en que se protege la información de su organización. Puede servir como infraestructura para:

- Varios escenarios de seguridad, como la supervisión (auditoría) y las alertas relacionadas con accesos anómalos a información confidencial.
- Controlar el acceso y mejorar la seguridad de las bases de datos que contienen información altamente confidencial.
- Ayudar a cumplir los requisitos de cumplimiento de normas y los estándares relacionados con la privacidad de datos.

Para más información, consulte [Clasificación y detección de datos de Azure SQL Database](sql-database-data-discovery-and-classification.md).

### <a name="compliance"></a>Cumplimiento normativo

Además de las anteriores características y funcionalidades que pueden ayudar a la aplicación a cumplir distintos requisitos de seguridad, Azure SQL Database también participa en las auditorías regulares y ha obtenido la certificación de una serie de normas de cumplimiento. Para obtener más información, consulte el [Microsoft Azure Trust Center](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942) donde puede encontrar la lista más actualizada de certificaciones de cumplimiento de la base de datos SQL.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información acerca del uso de las características de control de acceso en SQL Database, consulte [Control de acceso](sql-database-control-access.md).
- Para más información sobre la auditoría de bases de datos, consulte [SQL Database auditing](sql-database-auditing.md) (Auditoría de SQL Database).
- Para más información sobre la detección de amenaza, consulte [SQL Database threat detection](sql-database-threat-detection.md) (Detección de amenazas de SQL Database).
