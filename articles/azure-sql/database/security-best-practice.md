---
title: Cuaderno de estrategias para abordar requisitos de seguridad comunes
titleSuffix: Azure SQL Database & Azure SQL Managed Instance
description: En este artículo se proporcionan requisitos de seguridad comunes y procedimientos recomendados de seguridad en Azure SQL Database y en Azure SQL Managed Instance.
ms.service: sql-db-mi
ms.subservice: security
ms.custom: sqldbrb=2
author: VanMSFT
ms.author: vanto
ms.topic: article
ms.date: 02/20/2020
ms.reviewer: ''
ms.openlocfilehash: 6630b924decacc5ff59611c657e1d7e38b1813a7
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87541726"
---
# <a name="playbook-for-addressing-common-security-requirements-with-azure-sql-database-and-azure-sql-managed-instance"></a>Cuaderno de estrategias para abordar requisitos de seguridad comunes con Azure SQL Database y Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

En este documento se proporcionan los procedimientos recomendados para resolver los requisitos de seguridad comunes. No todos los requisitos se pueden aplicar a todos los entornos y debe consultar la base de datos y al equipo de seguridad qué características implementar.

## <a name="solving-common-security-requirements"></a>Solución de requisitos de seguridad comunes

En este documento se proporcionan instrucciones sobre cómo resolver los requisitos de seguridad comunes para aplicaciones nuevas o existentes mediante Azure SQL Database e Instancia administrada de Azure SQL. Está organizado por áreas de seguridad de nivel alto. Para abordar amenazas específicas, consulte la sección [Amenazas de seguridad comunes y posibles mitigaciones](#common-security-threats-and-potential-mitigations). Aunque algunas de las recomendaciones presentadas se pueden aplicar al migrar aplicaciones desde el entorno local a Azure, los escenarios de migración no son el foco de este documento.

### <a name="azure-sql-database-deployment-offers-covered-in-this-guide"></a>Ofertas de implementación de Azure SQL Database que se describen en esta guía

- [Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-single-index): [bases de datos únicas](single-database-overview.md) y [grupos elásticos](elastic-pool-overview.md) en [servidores](logical-servers.md)
- [Instancia administrada de Azure SQL](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-index)

### <a name="deployment-offers-not-covered-in-this-guide"></a>Ofertas de implementación que no se describen en esta guía

- Azure SQL Data Warehouse
- Máquinas virtuales de Azure SQL (IaaS)
- SQL Server

### <a name="audience"></a>Público

Esta guía está pensada para clientes que se preguntan cómo proteger Azure SQL Database. Los roles interesados en este artículo de procedimientos recomendados incluyen, entre otros:

- Arquitectos de seguridad
- Administradores de seguridad
- Responsables de cumplimiento normativo
- Responsables de privacidad
- Ingenieros de seguridad

### <a name="using-this-guide"></a><a id="using"></a> Uso de esta guía

Este documento está pensado como complemento de la documentación de [seguridad de Azure SQL Database](security-overview.md).

A menos que se indique lo contrario, se recomienda seguir todos los procedimientos recomendados que aparecen en cada sección para lograr el objetivo o requisito respectivo. Para cumplir con los procedimientos recomendados o los estándares de cumplimiento de seguridad específicos, los controles de cumplimiento normativo importantes se muestran en la sección Requisitos u Objetivos, siempre que corresponda. Estos son los estándares y reglamentaciones de seguridad a las que se hace referencia en este documento:

- [FedRAMP](https://www.fedramp.gov/documents/): AC-04, AC-06
- [SOC](https://www.aicpa.org/interestareas/frc/assuranceadvisoryservices/sorhome.html): CM-3, SDL-3
- [ISO/IEC 27001](https://www.iso27001security.com/html/27001.html): Control de acceso, Criptografía
- [Procedimientos de Garantía de seguridad operacional (OSA) de Microsoft](https://www.microsoft.com/securityengineering/osa/practices): Procedimiento n.° 1 a 6 y n.° 9
- [Controles de seguridad NIST Special Publication 800-53](https://nvd.nist.gov/800-53): AC-5, AC-6
- [PCI DSS](https://www.pcisecuritystandards.org/document_library): 6.3.2, 6.4.2

Tenemos previsto seguir actualizando las recomendaciones y los procedimientos recomendados que se describen aquí. Para proporcionar entradas o correcciones en este documento, use el vínculo **Comentarios** que se encuentra en la parte inferior de este artículo.

## <a name="authentication"></a>Authentication

La autenticación es el proceso por el cual se demuestra que el usuario es quien dice ser. Azure SQL Database e Instancia administrada de SQL admiten dos tipos de autenticación:

- Autenticación SQL
- Autenticación con Azure Active Directory

> [!NOTE]
> Es posible que la autenticación con Azure Active Directory no se admita en todas las herramientas y aplicaciones de terceros.

### <a name="central-management-for-identities"></a>Administración central de identidades

La administración de identidades central ofrece estas ventajas:

- Administre cuentas de grupo y controle los permisos de usuario sin duplicar los inicios de sesión en servidores, bases de datos e instancias administradas.
- Administración de permisos simplificada y flexible.
- Administración de aplicaciones a gran escala.

**Implementación**:

- Use la autenticación de Azure Active Directory (Azure AD) para la administración de identidades centralizada.

**Procedimientos recomendados**:

- Cree un inquilino de Azure AD y [cree usuarios](../../active-directory/fundamentals/add-users-azure-active-directory.md) para representar a los usuarios humanos y cree [entidades de servicio](../../active-directory/develop/app-objects-and-service-principals.md) para representar aplicaciones, servicios y herramientas de automatización. Las entidades de servicio son equivalentes a las cuentas de servicio de Windows y Linux.

- Asigne derechos de acceso a los recursos a entidades de seguridad de Azure AD a través de la asignación de grupos: Cree grupos de Azure AD, conceda acceso a los grupos y agregue miembros individuales a los grupos. En la base de datos, cree usuarios de bases de datos independientes que asignen los grupos de Azure AD. Para asignar permisos dentro de la base de datos, coloque los usuarios que están asociados con los grupos de Azure AD en los roles de base de datos con los permisos adecuados.
  - Consulte los artículos [Configuración y administración de la autenticación de Azure Active Directory con SQL](authentication-aad-configure.md) y [Usar la autenticación de Azure Active Directory para autenticación con SQL](authentication-aad-overview.md).
  > [!NOTE]
  > En Instancia administrada de SQL, también puede crear inicios de sesión que se asignen a entidades de seguridad de Azure AD en la base de datos maestra. Consulte [CREATE LOGIN (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current).

- El uso de grupos de Azure AD simplifica la administración de permisos y tanto el propietario del grupo como el propietario del recurso puede agregar o quitar miembros del grupo.

- Cree un grupo independiente para los administradores de Azure AD para cada servidor o instancia administrada.

  - Consulte el artículo [Aprovisionamiento de un administrador de Azure Active Directory para su servidor](authentication-aad-configure.md#provision-azure-ad-admin-sql-database).

- Supervise los cambios en la pertenencia a los grupos de Azure AD con los informes de actividad de auditoría de Azure AD.

- En el caso de una instancia administrada, se requiere un paso aparte para crear un administrador de Azure AD.
  - Consulte el artículo [Aprovisionamiento de un administrador de Azure Active Directory para su instancia administrada](authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance).

> [!NOTE]
>
> - La autenticación de Azure AD se registra en los registros de auditoría de Azure SQL, pero no en los registros de inicio de sesión de Azure AD.
> - Los permisos de RBAC concedidos en Azure no se aplican a los permisos de Azure SQL Database ni de Instancia administrada de SQL. Estos permisos se deben crear o asignar manualmente con los permisos de SQL existentes.
> - En el lado cliente, la autenticación de Azure AD necesita acceso a Internet o, a través de una ruta definida por el usuario (UDR), a una red virtual.
> - El token de acceso de Azure AD se almacena en caché en el lado cliente y su vigencia depende de la configuración del token. Consulte el artículo [Vigencia de tokens configurable en Azure Active Directory](../../active-directory/develop/active-directory-configurable-token-lifetimes.md).
> - Para encontrar instrucciones de solución de problemas con la autenticación de Azure AD, consulte el siguiente blog: [Solución de problemas de Azure AD](https://techcommunity.microsoft.com/t5/azure-sql-database/troubleshooting-problems-related-to-azure-ad-authentication-with/ba-p/1062991).

### <a name="azure-multi-factor-authentication"></a>Azure Multi-Factor Authentication

> Se menciona en: Procedimiento n.° 2 de OSA, Control de acceso (CA) de ISO

Azure Multi-Factor Authentication ayuda a proporcionar más seguridad al requerir más de una forma de autenticación.

**Implementación**:

- [Habilite Multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md) en Azure AD mediante el acceso condicional y use la autenticación interactiva.

- La alternativa es habilitar Multi-Factor Authentication para todo el dominio de AD o Azure AD.

**Procedimientos recomendados**:

- Active el acceso condicional en Azure AD (requiere una suscripción Premium).
  - Consulte el artículo [Acceso condicional en Azure°AD](../../active-directory/conditional-access/overview.md).  

- Cree grupos de Azure AD y habilite la directiva de Multi-Factor Authentication para los grupos seleccionados con el acceso condicional de Azure AD.
  - Consulte el artículo [Planeamiento de la implementación del Acceso condicional](../../active-directory/conditional-access/plan-conditional-access.md).

- Multi-Factor Authentication se puede habilitar para todo Azure AD o para todo Active Directory federado con Azure AD.

- Use el modo de autenticación interactiva de Azure AD para Azure SQL Database y Azure SQL Managed Instance, donde se solicita una contraseña de manera interactiva, seguida de la autenticación de Multi-Factor Authentication:
  - Use la autenticación universal en SSMS. Consulte el artículo [Uso de la autenticación multifactor de Azure AD con Azure SQL Database, SQL Managed Instance y Azure Synapse (compatibilidad de SSMS con Multi-Factor Authentication)](authentication-mfa-ssms-overview.md).
  - Use la autenticación interactiva admitida en SQL Server Data Tools (SSDT). Consulte el artículo [Compatibilidad de Azure Active Directory con SQL Server Data Tools (SSDT)](https://docs.microsoft.com/sql/ssdt/azure-active-directory?view=azuresqldb-current).
  - Use otras herramientas de SQL que admitan Multi-Factor Authentication.
    - Compatibilidad con el Asistente de SSMS para exportar, extraer e implementar una base de datos  
    - [sqlpackage.exe](https://docs.microsoft.com/sql/tools/sqlpackage): opción "/ua"
    - [Utilidad sqlcmd ](https://docs.microsoft.com/sql/tools/sqlcmd-utility): opción -G (interactiva)
    - [Utilidad bcp](https://docs.microsoft.com/sql/tools/bcp-utility): opción -G (interactiva)

- Implemente las aplicaciones para conectarse a Azure SQL Database o Azure SQL Managed Instance mediante la autenticación interactiva con compatibilidad con Multi-Factor Authentication.
  - Consulte el artículo [Conexión a Azure SQL Database con Azure Multi-factor Authentication](active-directory-interactive-connect-azure-sql-db.md).
  > [!NOTE]
  > Este modo de autenticación requiere identidades basadas en el usuario. En los casos en los que se usa un modelo de identidad de confianza que omite la autenticación de usuario individual de Azure AD (por ejemplo, mediante la identidad administrada para recursos de Azure), Multi-Factor Authentication no se aplica.

### <a name="minimize-the-use-of-password-based-authentication-for-users"></a>Minimización del uso de la autenticación de usuarios basada en contraseñas

> Se menciona en: Procedimiento n.° 4 de OSA, Control de acceso (CA) de ISO

Los métodos de autenticación basados en contraseña son una forma menos segura de autenticación. Las credenciales pueden verse comprometidas o se pueden entregar por error.

**Implementación**:

- Use una autenticación integrada de Azure AD que elimine el uso de contraseñas.

**Procedimientos recomendados**:

- Use la autenticación de inicio de sesión único con credenciales de Windows. Federe el dominio de AD local con Azure AD y use la autenticación integrada de Windows (para máquinas unidas a un dominio con Azure AD).
  - Consulte el artículo [Compatibilidad de SSMS con la autenticación integrada de Azure AD](authentication-aad-configure.md#active-directory-integrated-authentication).

### <a name="minimize-the-use-of-password-based-authentication-for-applications"></a>Minimización del uso de la autenticación de aplicaciones basada en contraseñas

> Se menciona en: Procedimiento n.° 4 de OSA, Control de acceso (CA) de ISO

**Implementación**:

- Habilite la identidad administrada de Azure. También puede usar la autenticación integrada o basada en certificados.

**Procedimientos recomendados**:

- Use [identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md).
  - [Identidad administrada asignada por el sistema](../../active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-sql.md)
  - [Identidad administrada asignada por el usuario](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md)
  - [Uso de Azure SQL Database desde Azure App Service con identidad administrada (sin cambios de código)](https://github.com/Azure-Samples/app-service-msi-entityframework-dotnet)

- Use la autenticación basada en certificados para una aplicación.
  - Consulte este [ejemplo de código](https://github.com/Microsoft/sql-server-samples/tree/master/samples/features/security/azure-active-directory-auth/token).

- Use la autenticación de Azure AD para una máquina unida a un dominio y un dominio federado integrado (consulte la sección anterior).
  - Consulte la [aplicación de ejemplo para la autenticación integrada](https://github.com/Microsoft/sql-server-samples/tree/master/samples/features/security/azure-active-directory-auth/integrated).

### <a name="protect-passwords-and-secrets"></a>Protección de contraseñas y secretos

En caso de que no se puedan evitar las contraseñas, asegúrese de que estén protegidas.

**Implementación**:

- Use Azure Key Vault para almacenar contraseñas y secretos. Siempre que sea aplicable, use Multi-Factor Authentication para Azure SQL Database con usuarios de Azure AD.

**Procedimientos recomendados**:

- Si no es posible evitar contraseñas o secretos, almacene las contraseñas de usuario y los secretos de aplicación en Azure Key Vault y administre el acceso a través de las directivas de acceso de Key Vault.

- Varios marcos de desarrollo de aplicaciones también pueden ofrecer mecanismos específicos del marco para proteger los secretos de la aplicación. Por ejemplo: [Aplicación de ASP.NET Core](https://docs.microsoft.com/aspnet/core/security/app-secrets?view=aspnetcore-2.1&tabs=windows).

### <a name="use-sql-authentication-for-legacy-applications"></a>Uso de la autenticación de SQL para las aplicaciones heredadas

La autenticación de SQL hace referencia a la autenticación de un usuario al conectarse a Azure SQL Database o Instancia administrada de SQL con el nombre de usuario y la contraseña. Se debe crear un inicio de sesión en cada servidor o instancia administrada y se debe crear un usuario en cada base de datos.

**Implementación**:

- Use la autenticación de SQL.

**Procedimientos recomendados**:

- Como administrador del servidor o la instancia, cree inicios de sesión y usuarios. A menos que use usuarios de bases de datos independientes con contraseñas, todas las contraseñas se almacenan en la base de datos maestra.
  - Consulte el artículo [Control y concesión de acceso de la base de datos a SQL Database, Instancia administrada de SQL y Azure Synapse Analytics](logins-create-manage.md).

## <a name="access-management"></a>Administración de acceso

La administración de acceso (también denominada Autorización) es el proceso de controlar y administrar los privilegios y el acceso de los usuarios autorizados a Azure SQL Database o Instancia administrada de SQL.

### <a name="implement-principle-of-least-privilege"></a>Implementación del principio de privilegio mínimo

> Se menciona en: FedRamp controla AC-06, NIST: AC-6, Procedimiento n.° 3 de OSA

El principio de privilegio mínimo indica que los usuarios no deben tener más privilegios de los necesarios para completar sus tareas. Para más información, consulte el artículo [Just Enough Administration](https://docs.microsoft.com/powershell/scripting/learn/remoting/jea/overview).

**Implementación**:

Asigne solo los permisos [necesarios](https://docs.microsoft.com/sql/relational-databases/security/permissions-database-engine) para completar las tareas necesarias:

- En SQL Database:
  - Use permisos granulares y roles de bases de datos definidos por el usuario (o roles de servidor en Instancia administrada):
    1. Cree los roles necesarios.
       - [CREATE ROLE](https://docs.microsoft.com/sql/t-sql/statements/create-role-transact-sql)
       - [CREATE SERVER ROLE](https://docs.microsoft.com/sql/t-sql/statements/create-server-role-transact-sql)
    1. Cree los usuarios necesarios.
       - [CREATE USER](https://docs.microsoft.com/sql/t-sql/statements/create-user-transact-sql)
    1. Agregue usuarios como miembros a los roles.
       - [ALTER ROLE](https://docs.microsoft.com/sql/t-sql/statements/alter-role-transact-sql)
       - [ALTER SERVER ROLE](https://docs.microsoft.com/sql/t-sql/statements/alter-server-role-transact-sql)
    1. Luego, asigne permisos a los roles.
       - [GRANT](https://docs.microsoft.com/sql/t-sql/statements/grant-transact-sql)
  - Asegúrese de no asignar usuarios a roles innecesarios.

- En Azure Resource Manager:
  - Use los roles integrados, si están disponibles, o roles personalizados de Azure y asigne los permisos necesarios.
    - [Roles integrados en los recursos de Azure](../../role-based-access-control/built-in-roles.md)
    - [Roles personalizados en los recursos de Azure](../../role-based-access-control/custom-roles.md)

**Procedimientos recomendados**:

Los procedimientos recomendados siguientes son opcionales, pero darán lugar a una mejor capacidad de administración y compatibilidad con la estrategia de seguridad:

- Si es factible, empiece con el menor conjunto de permisos posible y comience a agregar permisos de uno en uno si es realmente necesario (y se justifica), en lugar del enfoque opuesto: eliminar los permisos paso a paso.

- Evite asignar permisos a usuarios individuales. En su lugar, use roles (roles de base de datos o de servidor) de manera coherente. Los roles ayudan en gran medida a los informes y a la solución de problemas. (Azure RBAC solo admite la asignación de permisos a través de roles).

- Cree y use roles personalizados con los permisos específicos necesarios. Roles típicos que se usan en la práctica:
  - Implementación de seguridad
  - Administrador
  - Desarrollador
  - Personal de soporte técnico
  - Auditor
  - Procesos automatizados
  - Usuario final
  
- Use roles integrados solo cuando los permisos de los roles coincidan exactamente con los que necesite el usuario. Puede asignar usuarios a varios roles.

- Recuerde que los permisos del motor de base de datos se pueden aplicar dentro de los siguientes ámbitos (cuanto más pequeño sea el ámbito, menor será el impacto de los permisos concedidos):
  - Servidor (roles especiales en la base de datos maestra) en Azure
  - Base de datos
  - Schema
    - El uso de esquemas para conceder permisos en una base de datos es un procedimiento recomendado (consulte también el artículo en el que se explica el [Schema-design: Recommendations for Schema design with security in mind](http://andreas-wolter.com/en/schema-design-for-sql-server-recommendations-for-schema-design-with-security-in-mind/) [Diseño de esquemas: recomendaciones para el diseño de esquemas teniendo en cuenta la seguridad])
  - Objeto (tabla, vista, procedimiento, etc.)

  > [!NOTE]
  > No se recomienda aplicar permisos en el nivel de objeto porque este nivel agrega complejidad innecesaria a la implementación global. Si decide usar permisos en el nivel de objeto, se deben documentar claramente. Lo mismo se aplica a los permisos en el nivel de columna, que se recomiendan incluso menos por las mismas razones. Tenga también en cuenta también que, de forma predeterminada, una instrucción [DENY](https://docs.microsoft.com/sql/t-sql/statements/deny-object-permissions-transact-sql) de nivel de tabla no invalida una instrucción GRANT de nivel de columna. Esto requeriría la activación de la [configuración de un servidor que cumpla criterios comunes](https://docs.microsoft.com/sql/database-engine/configure-windows/common-criteria-compliance-enabled-server-configuration-option).

- Lleve a cabo comprobaciones periódicas con [Evaluación de vulnerabilidades (VA)](https://docs.microsoft.com/sql/relational-databases/security/sql-vulnerability-assessment) para probar si hay demasiados permisos.

### <a name="implement-separation-of-duties"></a>Implementación de la separación de tareas

> Se menciona en: FedRamp: AC-04, NIST: AC-5, ISO: A.6.1.2, PCI 6.4.2, SOC: CM-3, SDL-3

La separación de tareas, que también se denomina "separación de obligaciones", describe el requisito de dividir las tareas confidenciales en varias tareas que se asignan a distintos usuarios. La separación de tareas ayuda a evitar infracciones de datos.

**Implementación**:

- Identifique el nivel necesario de separación de tareas. Ejemplos:
  - Entre entornos de desarrollo/prueba y entornos de producción
  - Tareas confidenciales de seguridad frente a tareas en el nivel de administración del Administrador de base de datos (DBA) frente a tareas del desarrollador.
    - Ejemplos: Auditor, creación de la directiva de seguridad para la seguridad de nivel de fila (RLS), implementación de objetos de SQL Database con permisos de DDL.

- Identifique una jerarquía completa de los usuarios (y procesos automatizados) que tienen acceso al sistema.

- Cree roles según los grupos de usuarios necesarios y asigne permisos a los roles.
  - En el caso de las tareas en el nivel de administración en Azure Portal o a través de la automatización de PowerShell, use los roles de Azure. Busque un rol integrado que coincida con el requisito o cree un rol personalizado de Azure con los permisos disponibles.
  - Cree roles de servidor para las tareas de servidor (crear inicios de sesión y bases de datos) en una instancia administrada.
  - Cree roles de base de datos para las tareas en el nivel de base de datos.

- En el caso de ciertas tareas confidenciales, considere la posibilidad de crear procedimientos almacenados especiales firmados por un certificado para ejecutar las tareas en nombre de los usuarios. Una ventaja importante de los procedimientos almacenados firmados digitalmente es que, si se cambia el procedimiento, se quitan inmediatamente los permisos concedidos a la versión anterior del procedimiento.
  - Ejemplo: [Tutorial: Firmar procedimientos almacenados con un certificado](https://docs.microsoft.com/sql/relational-databases/tutorial-signing-stored-procedures-with-a-certificate)

- Implemente el Cifrado de datos transparente (TDE) con claves administradas por el cliente en Azure Key Vault para habilitar la separación de tareas entre el propietario de los datos y el propietario de la seguridad.
  - Consulte el artículo [Configuración de las claves administradas del cliente para el cifrado de Azure Storage desde Azure Portal](../../storage/common/storage-encryption-keys-portal.md).

- Para asegurarse de que un DBA no pueda ver los datos que se consideran muy sensibles y pueda seguir realizando las tareas de DBA, puede usar Always Encrypted con separación de roles.
  - Consulte los artículos [Información general de administración de claves de Always Encrypted](https://docs.microsoft.com/sql/relational-databases/security/encryption/overview-of-key-management-for-always-encrypted), [Aprovisionamiento de claves con separación de roles](https://docs.microsoft.com/sql/relational-databases/security/encryption/configure-always-encrypted-keys-using-powershell#KeyProvisionWithRoles) y [Rotación de claves maestras de columna con separación de roles](https://docs.microsoft.com/sql/relational-databases/security/encryption/rotate-always-encrypted-keys-using-powershell#column-master-key-rotation-with-role-separation).

- En los casos en los que no sea factible el uso de Always Encrypted, al menos sin costos importantes y esfuerzos que puedan hacer que el sistema quede casi inutilizable, es posible llegar a consensos y mitigar los riesgos mediante el uso de controles de compensación como:
  - Intervención humana en los procesos.
  - Pistas de auditoría: para más información sobre la auditoría, consulte [Auditoría de eventos de seguridad críticos](#audit-critical-security-events).

**Procedimientos recomendados**:

- Asegúrese de que se usan cuentas distintas para los entornos de desarrollo/pruebas y de producción. Las distintas cuentas ayudan a cumplir con la separación de los sistemas de prueba y producción.

- Evite asignar permisos a usuarios individuales. En su lugar, use roles (roles de base de datos o de servidor) de manera coherente. Tener roles ayuda en gran medida a los informes y a la solución de problemas.

- Use roles integrados cuando los permisos coincidan exactamente con los permisos necesarios: si la unión de todos los permisos de varios roles integrados genera una coincidencia del 100 %, puede asignar también varios roles de manera simultánea.

- Cree y use roles definidos por el usuario cuando los roles integrados concedan demasiados permisos o permisos insuficientes.

- Las asignaciones de roles también se pueden realizar de manera temporal, algo que también se conoce como "separación dinámica de tareas" (DSD), ya sea dentro de los pasos de trabajo del Agente SQL en T-SQL o mediante Azure PIM para los roles de Azure.

- Asegúrese de que los administradores de bases de datos no tengan acceso a las claves de cifrado o los almacenes de claves y que los administradores de seguridad con acceso a las claves no tengan, a su vez, acceso a la base de datos. El uso de la [Administración extensible de claves (EKM)](https://docs.microsoft.com/sql/relational-databases/security/encryption/extensible-key-management-ekm) puede facilitar esta separación. [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) se puede usar para implementar EKM.

- Asegúrese siempre de tener una pista de auditoría para las acciones relacionadas con la seguridad.

- Puede recuperar la definición de los roles integrados de Azure para ver los permisos usados y crear un rol personalizado basado en extractos y acumulaciones de estos a través de PowerShell.

- Puesto que cualquier miembro del rol de base de datos db_owner puede cambiar una configuración de seguridad como Cifrado de datos transparente (TDE) o cambiar el SLO, es necesario tener cuidado al conceder esta pertenencia. No obstante, existen muchas tareas que requieren privilegios db_owner. Tareas como cambiar cualquier valor de base de datos (por ejemplo, opciones de la base de datos). La auditoría desempeña un papel clave en cualquier solución.

- No es posible restringir los permisos de db_owner ni, por consiguiente, impedir que una cuenta administrativa vea datos de usuario. Si hay datos altamente confidenciales en una base de datos, Always Encrypted se pueden usar para impedir que db_owners o cualquier otro DBA puedan verlos.

> [!NOTE]
> Lograr la separación de tareas (SoD) es desafiante para las tareas relacionadas con la seguridad o la solución de problemas. Otras áreas como el desarrollo y los roles de usuario final son más fáciles de separar. La mayoría de los controles relacionados con el cumplimiento permiten el uso de funciones de control alternativas, como la auditoría, cuando otras soluciones no resultan prácticas.

En el caso de los lectores que quieran profundizar en SoD, se recomiendan los recursos siguientes:

- Para Azure SQL Database y SQL Managed Instance:  
  - [Control y concesión de acceso a bases de datos](logins-create-manage.md)
  - [Motor Separación de tareas para el desarrollador de aplicaciones](https://docs.microsoft.com/previous-versions/sql/sql-server-2008/cc974525(v=sql.100))
  - [Separación de obligaciones](https://www.microsoft.com/download/details.aspx?id=39269)
  - [Firmar procedimientos almacenados](https://docs.microsoft.com/dotnet/framework/data/adonet/sql/signing-stored-procedures-in-sql-server)

- Para la administración de recursos de Azure:
  - [Roles integrados en los recursos de Azure](../../role-based-access-control/built-in-roles.md)
  - [Roles personalizados en los recursos de Azure](../../role-based-access-control/custom-roles.md)
  - [Uso de Azure AD Privileged Identity Management para acceso con privilegios elevados](https://www.microsoft.com/itshowcase/using-azure-ad-privileged-identity-management-for-elevated-access)

### <a name="perform-regular-code-reviews"></a>Realización de revisiones de código periódicas

> Se menciona en: PCI: 6.3.2, SOC: SDL-3

La separación de tareas no se limita a los datos de una base de datos, sino que incluye el código de la aplicación. El código malintencionado puede eludir los controles de seguridad. Antes de implementar el código personalizado en el entorno de producción, es esencial revisar lo que se está implementando.

**Implementación**:

- Use una herramienta de base de datos como Azure Data Studio que admita el control de código fuente.

- Implemente un proceso de implementación de código separado.

- Antes de confirmar en la rama principal, una persona (que no sea el creador del código mismo) tiene que inspeccionar el código para detectar posibles riesgos de elevación de privilegios, así como modificaciones de datos malintencionados como protección contra fraudes y accesos no autorizados. Esto se puede hacer mediante el uso de mecanismos de control de código fuente.

**Procedimientos recomendados**:

- Normalización: ayuda a implementar un procedimiento estándar que se va a seguir para cualquier actualización de código.

- La Evaluación de vulnerabilidades contiene reglas que comprueban si hay permisos excesivos, el uso de algoritmos de cifrado antiguos y otros problemas de seguridad dentro de un esquema de base de datos.

- Se pueden realizar más comprobaciones en un entorno de control de calidad o de prueba mediante Advanced Threat Protection que examina el código que es vulnerable a la inyección de código SQL.

- Ejemplos de lo que se debe observar:
  - La creación de un usuario o el cambio de la configuración de seguridad desde una implementación automatizada de actualización de código SQL.
  - Un procedimiento almacenado que, en función de los parámetros proporcionados, actualiza un valor monetario de una celda de una manera no conforme.

- Asegúrese de que la persona que realiza la revisión es un individuo distinto del autor del código de origen y experto en revisiones de código y codificación segura.

- Asegúrese de conocer todos los orígenes de los cambios de código. El código puede estar en scripts T-SQL. Puede tratarse de comandos ad-hoc que se ejecutarán o implementarán en forma de vistas, funciones, desencadenadores y procedimientos almacenados. Puede formar parte de las definiciones del trabajo (pasos) del Agente SQL. También se puede ejecutar desde paquetes SSIS, Azure Data Factory y otros servicios.

## <a name="data-protection"></a>Protección de los datos

La protección de datos es un conjunto de funcionalidades para proteger la información importante del riesgo mediante cifrado o ofuscación.

> [!NOTE]
> Microsoft certifica que Azure SQL Database e Instancia administrada de SQL cumplen con el nivel 1 de FIPS 140-2. Esto se hace después de comprobar el uso estricto de los algoritmos aceptables de nivel 1 de FIPS 140-2 y las instancias validadas de nivel 1 de FIPS 140-2 de esos algoritmos, incluida la coherencia con las longitudes de clave, la administración de claves, la generación de claves y el almacenamiento de claves necesarios. Esta atestación está pensada para permitir que nuestros clientes respondan a la necesidad o al requisito de usar las instancias validadas de nivel 1 de FIPS 140-2 en el procesamiento de datos o la entrega de sistemas o aplicaciones. Definimos los términos "Conforme con el nivel 1 de FIPS 140-2" y "Cumplimiento con el nivel 1 de FIPS 140-2" que se usan en la afirmación anterior para demostrar su aplicabilidad prevista según el uso del término "Validado para el nivel 1 de FIPS 140-2" de la administración pública de los Estados Unidos y Canadá.

### <a name="encrypt-data-in-transit"></a>Cifrado de los datos en tránsito

> Se menciona en: Procedimiento n.° 6 de OSA, familia de control de ISO: Cryptography

Protege los datos mientras se mueven entre el cliente y el servidor. Consulte [Seguridad de las redes](#network-security).

### <a name="encrypt-data-at-rest"></a>Cifrado de datos en reposo

> Se menciona en: Procedimiento n.° 6 de OSA, familia de control de ISO: Cryptography

El cifrado en reposo es la protección criptográfica de los datos cuando persisten en archivos de copias de seguridad, registros y bases de datos.

**Implementación**:

- El [Cifrado de datos transparente (TDE)](transparent-data-encryption-tde-overview.md) con claves administradas por el servicio están habilitadas de manera predeterminada para todas las bases de datos creadas después de 2017 en Azure SQL Database e Instancia administrada de SQL.
- En una instancia administrada, si la base de datos se crea a partir de una operación de restauración mediante un servidor local, se respetará la configuración de TDE de la base de datos original. Si la base de datos original no tiene habilitado TDE, se recomienda que TDE se active manualmente para la instancia administrada.

**Procedimientos recomendados**:

- No almacene datos que requieran cifrado en reposo en la base de datos maestra. La base de datos maestra no se puede cifrar con TDE.

- Use claves administradas por el cliente en Azure Key Vault si necesita mayor transparencia y control granular sobre la protección de TDE. Azure Key Vault permite revocar los permisos en cualquier momento para representar la base de datos como inaccesible. Puede administrar de forma centralizada los protectores de TDE junto con otras claves, o bien rotar el protector de TDE según su propia programación mediante Azure Key Vault.

- Si usa claves administradas por el cliente en Azure Key Vault, siga los artículos [Directrices para configurar TDE con Azure Key Vault](transparent-data-encryption-byok-overview.md#recommendations-when-configuring-akv) y [Configuración de la recuperación ante desastres con localización geográfica con Azure Key Vault](transparent-data-encryption-byok-overview.md#geo-dr-and-customer-managed-tde).

### <a name="protect-sensitive-data-in-use-from-high-privileged-unauthorized-users"></a>Protección de datos confidenciales en uso de usuarios no autorizados con privilegios elevados

Los datos en uso son los datos almacenados en memoria del sistema de la base de datos durante la ejecución de consultas SQL. Si la base de datos almacena información confidencial, es posible que la organización deba asegurarse de que los usuarios con privilegios elevados no puedan ver datos confidenciales en la base de datos. Los usuarios con privilegios elevados, como los operadores de Microsoft o los administradores de bases de datos de su organización, deben ser capaces de administrar la base de datos, pero no de ver ni extraer datos confidenciales de la memoria del proceso de SQL o mediante una consulta a la base de datos.

Las directivas que determinan qué datos son confidenciales y si estos datos se deben cifrar en la memoria y si los administradores no deben tener acceso a ellos en texto sin formato, son específicas tanto de la organización como de las regulaciones de cumplimiento que se deben cumplir. Consulte el requisito relacionado: [Identificación y etiquetado de datos confidenciales](#identify-and-tag-sensitive-data).

**Implementación**:

- Use [Always Encrypted](https://docs.microsoft.com/sql/relational-databases/security/encryption/always-encrypted-database-engine) para asegurarse de que los datos confidenciales no se exponen en texto sin formato en Azure SQL Database o Instancia administrada de SQL, incluso en la memoria o en uso. Always Encrypted protege los datos de los administradores de bases de datos (DBA) y los administradores de la nube (o actores no válidos que pueden suplantar a usuarios con privilegios elevados pero no autorizados) y proporciona más control sobre quién puede acceder a los datos.

**Procedimientos recomendados**:

- Always Encrypted no es un sustituto para cifrar los datos en reposo (TDE) o en tránsito (SSL/TLS). Always Encrypted no deben usarse para datos no confidenciales con el fin de minimizar el rendimiento y el impacto de la funcionalidad. Se recomienda usar Always Encrypted junto con TDE y Seguridad de la capa de transporte (TLS) para una protección integral de los datos en reposo, en tránsito y en uso.

- Evalúe el impacto de cifrar las columnas de datos confidenciales identificadas antes de implementar Always Encrypted en una base de datos de producción. En general, Always Encrypted reduce la funcionalidad de las consultas en columnas cifradas y tiene otras limitaciones, que se enumeran en [Always Encrypted: detalles de las características](https://docs.microsoft.com/sql/relational-databases/security/encryption/always-encrypted-database-engine#feature-details). Por consiguiente, es posible que tenga que rediseñar la aplicación para volver a implementar la funcionalidad (que una consulta no admite) en el lado del cliente o refactorizar el esquema de la base de datos, lo que incluye las definiciones de los procedimientos, funciones, vistas y desencadenadores almacenados. Si no cumplen las restricciones y limitaciones de Always Encrypted, es posible que las aplicaciones existentes no funcionen con columnas cifradas. Aunque el ecosistema de herramientas, productos y servicios de Microsoft que admiten Always Encrypted está creciendo, hay una serie de ellos que no funciona con columnas cifradas. El cifrado de una columna también puede afectar al rendimiento de las consultas, en función de las características de la carga de trabajo.

- Administre las claves de Always Encrypted con la separación de roles si usa Always Encrypted para proteger los datos de DBA malintencionados. Con la separación de roles, un administrador de seguridad crea las claves físicas. El DBA crea objetos de metadatos de clave maestra de columna y de cifrado de columna, que describen las claves físicas en la base de datos. Durante este proceso, el administrador de seguridad no necesita tener acceso a la base de datos y el DBA no necesita tener acceso a las claves físicas en texto sin formato.
  - Consulte el artículo [Administración de claves con separación de roles](https://docs.microsoft.com/sql/relational-databases/security/encryption/overview-of-key-management-for-always-encrypted#managing-keys-with-role-separation) para detalles.

- Almacene las claves maestras de columna en Azure Key Vault para facilitar la administración. Evite usar el Almacén de certificados de Windows (y, en general, las soluciones de almacén de claves distribuidas, a diferencia de las soluciones de administración de claves centrales) que dificultan la administración de claves.

- Piense detenidamente en las ventajas y desventajas de usar varias claves (clave maestra de columna o claves de cifrado de columna). Mantenga reducido el número de claves para reducir el costo de administración de claves. Una clave maestra de columna y una clave de cifrado de columna por base de datos suele ser suficiente en entornos de estado estable (no en medio de una rotación de claves). Es posible que necesite claves adicionales si tiene grupos de usuarios diferentes, que usan claves diferentes y acceden a datos diferentes.  

- Rote las claves maestras de columna según los requisitos de cumplimiento. Si también necesita rotar las claves de cifrado de columna, considere la posibilidad de usar el cifrado en línea para minimizar el tiempo de inactividad de la aplicación.
  - Consulte el artículo [Consideraciones sobre rendimiento y disponibilidad](https://docs.microsoft.com/sql/relational-databases/security/encryption/configure-column-encryption-using-powershell#performance-and-availability-considerations).

- Use el cifrado determinista si es necesario admitir cálculos (igualdad) en los datos. De lo contrario, use el cifrado aleatorio. Evite usar el cifrado determinista para conjuntos de datos de baja entropía o conjuntos de datos con distribución conocida públicamente.

- Si le preocupa el acceso de terceros a los datos de manera legal y sin su consentimiento, asegúrese de que todas las aplicaciones y herramientas que tienen acceso a las claves y los datos en texto no cifrado se ejecuten fuera de la nube de Microsoft Azure. Sin acceso a las claves, el tercero no tendrá forma de descifrar los datos a menos que eludan el cifrado.

- Always Encrypted no admite fácilmente la concesión de acceso temporal a las claves (y a los datos protegidos). Por ejemplo, si necesita compartir las claves con un DBA para permitir que el DBA realice algunas operaciones de limpieza en datos confidenciales y cifrados. La única forma de revocar de forma confiable el acceso a los datos del DBA será girar las claves de cifrado de columnas y las claves maestras de columna que protegen los datos, lo que es una operación costosa.

- Para acceder a los valores de texto no cifrado de las columnas cifradas, un usuario debe tener acceso a la Clave maestra de columna (CMK) que protege las columnas, que están configuradas en el almacén de claves que contiene la CMK. El usuario también debe tener los permisos de base de datos **VIEW ANY COLUMN MASTER KEY DEFINITION** y **VIEW ANY COLUMN ENCRYPTION KEY DEFINITION**.

### <a name="control-access-of-application-users-to-sensitive-data-through-encryption"></a>Controle el acceso de los usuarios de la aplicación a los datos confidenciales a través del cifrado

El cifrado se puede usar como una manera de asegurarse de que solo usuarios específicos de la aplicación que tengan acceso a las claves criptográficas puedan ver o actualizar los datos.

**Implementación**:

- Use Cifrado de nivel de celda (CLE). Para detalles, consulte el artículo [Cifrar una columna de datos](https://docs.microsoft.com/sql/relational-databases/security/encryption/encrypt-a-column-of-data).
- Use Always Encrypted, pero tenga en cuenta sus limitaciones. Las limitaciones se indican a continuación.

**procedimientos recomendados**

Al usar CLE:

- Controle el acceso a las claves a través de permisos y roles de SQL.

- Use AES (se recomienda AES 256) para el cifrado de datos. Los algoritmos, como RC4, DES y TripleDES, están en desuso y no se deben utilizar debido a vulnerabilidades conocidas.

- Proteja las claves simétricas con certificados o claves asimétricos (no contraseñas) para evitar el uso de 3DES.

- Tenga cuidado al migrar una base de datos mediante el Cifrado de nivel de celda a través de Export/Import (archivos bacpac).
  - Consulte el artículo de [recomendaciones para usar el cifrado de nivel de celda en Azure SQL Database](https://blogs.msdn.microsoft.com/sqlsecurity/2015/05/12/recommendations-for-using-cell-level-encryption-in-azure-sql-database/) para saber cómo evitar perder claves al migrar datos y otras instrucciones de procedimientos recomendados.

Tenga en cuenta que Always Encrypted está diseñado principalmente para proteger los datos confidenciales en uso de los usuarios con privilegios elevados de Azure SQL Database (operadores de nube, DBA). Consulte [Protección de datos confidenciales en uso de usuarios no autorizados con privilegios elevados](#protect-sensitive-data-in-use-from-high-privileged-unauthorized-users). Tenga en cuenta los siguientes desafíos al usar Always Encrypted para proteger los datos de los usuarios de la aplicación:

- De manera predeterminada, todos los controladores cliente de Microsoft que admiten Always Encrypted mantienen una caché global (una por aplicación) de claves de cifrado de columnas. Una vez que un controlador cliente adquiere una clave de cifrado de columna de texto no cifrado poniéndose en contacto con un almacén de claves que contiene una clave maestra de columna, la clave de cifrado de columna de texto no cifrado se almacena en caché. Esto dificulta el aislamiento de los datos de los usuarios de una aplicación de varios usuarios. Si la aplicación suplanta a los usuarios finales al interactuar con un almacén de claves (como Azure Key Vault), una vez que la consulta de un usuario rellena la memoria caché con una clave de cifrado de columna, una consulta posterior que requiera la misma clave, pero que la desencadene otro usuario, utilizará el clave almacenada en caché. El controlador no llamará al almacén de claves y no comprobará si el segundo usuario tiene permiso de acceso a la clave de cifrado de columna. Como resultado, el usuario puede ver los datos cifrados aunque no tenga acceso a las claves. Para lograr el aislamiento de usuarios en una aplicación de varios usuarios, puede deshabilitar el almacenamiento en caché de claves de cifrado de columna. Al deshabilitar el almacenamiento en caché, se producirán sobrecargas de rendimiento adicionales, ya que el controlador deberá ponerse en contacto con el almacén de claves para cada operación de cifrado o descifrado de datos.

### <a name="protect-data-against-unauthorized-viewing-by-application-users-while-preserving-data-format"></a>Protección de datos contra la visualización no autorizada por parte de los usuarios de la aplicación al tiempo que se conserva el formato de los datos

Otra técnica para impedir que usuarios no autorizados vean datos es ofuscar o enmascarar los datos al tiempo que se conservan los tipos de datos y los formatos para asegurarse de que las aplicaciones de usuario puedan seguir controlando y mostrando los datos.

**Implementación**:

- Use el [Enmascaramiento dinámico de datos](https://docs.microsoft.com/sql/relational-databases/security/dynamic-data-masking) para ofuscar las columnas de las tablas.

> [!NOTE]
> Always Encrypted no funciona con el Enmascaramiento dinámico de datos. No es posible cifrar y enmascarar la misma columna, lo que implica que debe dar prioridad a la protección de los datos en uso en comparación con enmascarar los datos de los usuarios de la aplicación a través del Enmascaramiento dinámico de datos.

**Procedimientos recomendados**:

> [!NOTE]
> El Enmascaramiento dinámico de datos no se puede usar para proteger los datos de usuarios con privilegios elevados. Las directivas de enmascaramiento no se aplican a los usuarios con acceso administrativo como db_owner.

- No permita a los usuarios de la aplicación ejecutar consultas ad-hoc (ya que es posible que encuentren una solución alternativa para el Enmascaramiento dinámico de datos).  
  - Para detalles, consulte el artículo [Omisión del enmascaramiento con técnicas de fuerza bruta o inferencia](https://docs.microsoft.com/sql/relational-databases/security/dynamic-data-masking#security-note-bypassing-masking-using-inference-or-brute-force-techniques).  

- Use una directiva de control de acceso adecuada (a través de RLS, roles y permisos de SQL) para limitar los permisos de usuario para hacer actualizaciones en las columnas enmascaradas. La creación de una máscara en una columna no impide que se efectúen actualizaciones en ella. Los usuarios que reciban datos enmascarados cuando realicen una consulta en una columna enmascarada podrán actualizar los datos si cuentan con permisos de escritura.

- La función Enmascaramiento dinámico de datos no conserva las propiedades estadísticas de los valores enmascarados. Esto puede afectar a los resultados de la consulta (por ejemplo, las consultas que contienen predicados de filtrado o combinaciones en los datos enmascarados).

## <a name="network-security"></a>Seguridad de las redes

La seguridad de las redes hace referencia a los controles de acceso y a los procedimientos recomendados para proteger los datos en tránsito a Azure SQL Database.

### <a name="configure-my-client-to-connect-securely-to-sql-databasesql-managed-instance"></a>Configuración de mi cliente para una conexión segura a SQL Database o Instancia administrada de SQL

Procedimientos recomendados para impedir que las máquinas cliente y las aplicaciones con vulnerabilidades conocidas (por ejemplo, que utilizan protocolos y conjuntos de cifrado TLS) se conecten a Azure SQL Database e Instancia administrada de SQL.

**Implementación**:

- Asegúrese de que las máquinas cliente que se conectan a Azure SQL Database e Instancia administrada de SQL usan la [Seguridad de la capa de transporte (TLS)](security-overview.md#transport-layer-security-encryption-in-transit).

**Procedimientos recomendados**:

- Configure todas las aplicaciones y herramientas para conectarse a SQL Database con cifrado habilitado
  - Encrypt = On, TrustServerCertificate = Off (o equivalente con controladores que no son de Microsoft).

- Si la aplicación usa un controlador que no admite TLS o que admite una versión anterior de TLS, reemplace el controlador, si es posible. Si no es posible, evalúe detenidamente los riesgos de seguridad.

- Para reducir los vectores de ataque a través de vulnerabilidades en SSL 2.0, SSL 3.0, TLS 1.0 y TLS 1.1, deshabilítelos en las máquinas cliente que se conectan a Azure SQL Database por [configuración del Registro de Seguridad de la capa de transporte (TLS)](https://docs.microsoft.com/windows-server/security/tls/tls-registry-settings#tls-10).

- Compruebe los conjuntos de cifrado disponibles en el cliente: [Conjuntos de cifrado en TLS/SSL (Schannel SSP)](https://docs.microsoft.com/windows/desktop/SecAuthN/cipher-suites-in-schannel). En concreto, deshabilite 3DES para [configurar el orden de los conjuntos de cifrado TLS](https://docs.microsoft.com/windows-server/security/tls/manage-tls#configuring-tls-cipher-suite-order).

- En el caso de Azure SQL Database e Instancia administrada de SQL, el cifrado se aplica tanto a los tipos de conexión Proxy como Redireccionamiento. Para Instancia administrada de Azure SQL, utilice el tipo de conexión **Proxy** (valor predeterminado), ya que aplica el cifrado del lado servidor. El tipo de conexión **Redireccionamiento** actualmente no admite la aplicación del cifrado y solo está disponible en las conexiones de direcciones IP privadas.

- Para más información, consulte [Arquitectura de conectividad de Azure SQL Database - Directiva de conexión](connectivity-architecture.md#connection-policy).

### <a name="minimize-attack-surface"></a>Minimización de la superficie expuesta a ataques

Minimice el número de características susceptibles de sufrir el ataque de un usuario malintencionado. Implemente controles de acceso a la red para Azure SQL Database.

> Se menciona en: Procedimiento n.° 5 de OSA

**Implementación**:

En SQL Database:

- Establezca Permitir el acceso a servicios de Azure en OFF en el nivel de servidor.
- Utilice puntos de conexión de servicio de red virtual y reglas de firewall de red virtual.
- Use Private Link (versión preliminar).

En SQL Managed Instance:

- Siga las instrucciones que aparece en [Requisitos de red](../managed-instance/connectivity-architecture-overview.md#network-requirements).

**Procedimientos recomendados**:

- Restringir el acceso a Azure SQL Database e Instancia administrada de SQL mediante la conexión en un punto de conexión privado (por ejemplo, mediante una ruta de acceso a datos privada):
  - Una instancia administrada se puede aislar dentro de una red virtual para evitar el acceso externo. Las aplicaciones y herramientas que se encuentran en la misma red virtual o en una red virtual emparejada en la misma región podrían acceder a ella directamente. Las aplicaciones y herramientas que se encuentran en una región distinta podrían usar la conexión de red virtual a red virtual o el emparejamiento de circuito ExpressRoute para establecer la conexión. El cliente debe usar los grupos de seguridad de red (NSG) para restringir el acceso a través del puerto 1433 solo a los recursos que requieren acceso a una instancia administrada.
  - Para una instancia de SQL Database, use la característica [Private Link](../../private-link/private-endpoint-overview.md), que proporciona una dirección IP privada dedicada para el servidor dentro de la red virtual. También puede usar [puntos de conexión de servicio de red virtual con reglas de firewall de red virtual](vnet-service-endpoint-rule-overview.md) para restringir el acceso a los servidores.
  - Los usuarios móviles deben usar conexiones VPN de punto a sitio para conectarse a través de la ruta de acceso a los datos.
  - Los usuarios conectados a su red local deben usar la conexión VPN de sitio a sitio o ExpressRoute para conectarse a través de la ruta de acceso a los datos.

- Puede acceder a Azure SQL Database e Instancia administrada de SQL mediante la conexión a un punto de conexión público (por ejemplo, mediante una ruta de acceso a datos pública). Se deben tener en cuenta los procedimientos recomendados siguientes:
  - Para un servidor de SQL Database, use las [reglas de firewall de IP](firewall-configure.md) para restringir el acceso solo a direcciones IP autorizadas.
  - En el caso de SQL Managed Instance, use los grupos de seguridad de red (NSG) para restringir el acceso a través del puerto 3342 solo a los recursos necesarios. Para obtener más información, consulte [Uso de una instancia administrada de forma segura con puntos de conexión públicos](../managed-instance/public-endpoint-overview.md).

> [!NOTE]
> El punto de conexión público de SQL Managed Instance no está habilitado de forma predeterminada y debe habilitarse explícitamente. Si la directiva de la empresa no permite usar puntos de conexión públicos, use [Azure Policy](../../governance/policy/overview.md) para impedir que se habiliten los puntos de conexión públicos en primer lugar.

- Configure los componentes de Redes:
  - Siga los [Procedimientos recomendados de seguridad de la red de Azure](../../security/fundamentals/network-best-practices.md).
  - Planee la configuración de las redes virtuales según los procedimientos recomendados que se describen en las [Preguntas más frecuentes (P+F) acerca de Azure Virtual Network](../../virtual-network/virtual-networks-faq.md).
  - Segmente una red virtual en varias subredes y asigne recursos para un rol similar a la misma subred (por ejemplo, recursos front-end frente a recursos back-end).
  - Use los [grupos de seguridad de red (NSG)](../../virtual-network/security-overview.md) para controlar el tráfico entre las subredes dentro del límite de la red virtual de Azure.
  - Habilite [Azure Network Watcher](../../network-watcher/network-watcher-monitoring-overview.md) de su suscripción para supervisar el tráfico de red entrante y saliente.

### <a name="configure-power-bi-for-secure-connections-to-sql-databasesql-managed-instance"></a>Configuración de Power BI para proteger las conexiones a Azure SQL Database o Instancia administrada de SQL

**Procedimientos recomendados**:

- Para Power BI Desktop, utilice la ruta de acceso a datos privada siempre que sea posible.

- Asegúrese de que Power BI Desktop se está conectando mediante TLS 1.2 estableciendo la clave del Registro en la máquina equipo cliente según la configuración del Registro de [Seguridad de la capa de transporte (TLS)](https://docs.microsoft.com/windows-server/security/tls/tls-registry-settings).

- Restrinja el acceso a los datos para usuarios específicos a través de la [Seguridad de nivel de fila (RLS) con Power BI](https://docs.microsoft.com/power-bi/service-admin-rls).

- Para el servicio Power BI, use la [puerta de enlace de datos local](https://docs.microsoft.com/power-bi/service-gateway-onprem), teniendo en cuenta las [limitaciones y consideraciones](https://docs.microsoft.com/power-bi/service-gateway-deployment-guidance#installation-considerations-for-the-on-premises-data-gateway).

### <a name="configure-app-service-for-secure-connections-to-sql-databasesql-managed-instance"></a>Configuración de App Service para proteger las conexiones a Azure SQL Database o Instancia administrada de SQL

**Procedimientos recomendados**:

- En el caso de una aplicación web simple, la conexión a través de un punto de conexión público requiere que la opción **Allow Azure Services** (Permitir servicios de Azure) esté establecida en ON.

- [Integre su aplicación con una instancia de Azure Virtual Network](../../app-service/web-sites-integrate-with-vnet.md) para la conectividad de la ruta de acceso a datos privada con una instancia administrada. De manera opcional, también puede implementar una aplicación web con instancias de [App Service Environment (ASE)](../../app-service/environment/intro.md).

- En el caso de la aplicación web con ASE o la aplicación web integrada de red virtual con conexión a una base de datos en SQL Database, puede usar los [puntos de conexión de servicio de red virtual y las reglas de firewall de red virtual](vnet-service-endpoint-rule-overview.md) para limitar el acceso desde una red virtual y una subred específicas. A continuación, establezca **Allow Azure Services** (Permitir servicios de Azure) en OFF. También puede conectar ASE a una instancia administrada en Instancia administrada de SQL a través de una ruta de acceso a datos privada.  

- Asegúrese de que la aplicación web está configurada según el artículo [Procedimientos recomendados para proteger aplicaciones web y móviles de plataforma como servicio (PaaS) con Azure App Service](../../security/fundamentals/paas-applications-using-app-services.md).

- Instale [Firewall de aplicaciones web (WAF)](../../web-application-firewall/ag/ag-overview.md) para proteger la aplicación web de vulnerabilidades y aprovechamientos comunes.

### <a name="configure-azure-virtual-machine-hosting-for-secure-connections-to-sql-databasesql-managed-instance"></a>Configuración del hospedaje de máquina virtual de Azure para proteger las conexiones a SQL Database o SQL Managed Instance

**Procedimientos recomendados**:

- Use una combinación de reglas de permiso y denegación en los grupos de seguridad de red de las máquinas virtuales de Azure para controlar a qué regiones se puede acceder desde la máquina virtual.

- Asegúrese de que la máquina virtual está configurada según el artículo [Procedimientos de seguridad recomendados para cargas de trabajo de IaaS de Azure](../../security/fundamentals/iaas.md).

- Asegúrese de que todas las máquinas virtuales están asociadas con una red virtual y una subred específicas.

- Evalúe si necesita la ruta predeterminada 0.0.0.0/Internet según las instrucciones que se encuentran en [Información acerca de la tunelización forzada](../../vpn-gateway/vpn-gateway-forced-tunneling-rm.md#about-forced-tunneling).
  - Si es así (por ejemplo, subred de front-end), mantenga la ruta predeterminada.
  - Si no (por ejemplo, subred de back-end o nivel intermedio), habilite la tunelización forzada para que ningún tráfico pase a través de Internet para llegar a un entorno local (que también se conoce como entre locales).

- Implemente [Rutas predeterminadas opcionales](../../virtual-network/virtual-networks-udr-overview.md#optional-default-routes) si usa el emparejamiento o la conexión al entorno local.

- Implemente [rutas definidas por el usuario](../../virtual-network/virtual-networks-udr-overview.md#user-defined) si necesita enviar todo el tráfico de la red virtual a una aplicación virtual de red para la inspección de paquetes.

- Use [puntos de conexión de servicio de red virtual](vnet-service-endpoint-rule-overview.md) para el acceso seguro a los servicios de PaaS, como Azure Storage, a través de la red troncal de Azure.

### <a name="protect-against-distributed-denial-of-service-ddos-attacks"></a>Protección contra ataques de denegación de servicio distribuido (DDoS)

Los ataques de denegación de servicio distribuido (DDoS) son intentos por parte de un usuario malintencionado de enviar una avalancha de tráfico de red a Azure SQL Database con el objetivo de sobrecargar la infraestructura de Azure y hacer que rechace la carga de trabajo y los inicios de sesión válidos.

> Se menciona en: Procedimiento n.° 9 de OSA

**Implementación**:

La protección contra DDoS está habilitada de manera automática como parte de la plataforma de Azure. Incluye la supervisión del tráfico siempre activa y la mitigación en tiempo real de los ataques en el nivel de red en puntos de conexión públicos.

- Use [Azure DDoS Protection](../../virtual-network/ddos-protection-overview.md) para supervisar las direcciones IP públicas asociadas a los recursos implementados en redes virtuales.

- Utilice [Advanced Threat Protection en Azure SQL Database](threat-detection-overview.md) para detectar ataques de denegación de servicio (DoS) contra las bases de datos.

**Procedimientos recomendados**:

- Siga los procedimientos que se describen en [Minimización de la superficie expuesta a ataques](#minimize-attack-surface) para minimizar las amenazas de ataques DDoS.

- La alerta **Credenciales de SQL por fuerza bruta** de Advanced Threat Protection ayuda a detectar los ataques por fuerza bruta. En algunos casos, la alerta puede incluso distinguir las cargas de trabajo de pruebas de penetración.

- En el caso de las máquinas virtuales de Azure que hospedan aplicaciones que se conectan a SQL Database:
  - Siga las recomendaciones para restringir el acceso a través de puntos de conexión accesibles desde Internet en Azure Security Center.
  - Use los conjuntos de escalado de máquinas virtuales para ejecutar varias instancias de la aplicación en máquinas virtuales de Azure.
  - Deshabilite RDP y SSH desde Internet para evitar los ataques por fuerza bruta.

## <a name="monitoring-logging-and-auditing"></a>Supervisión, registro y auditoría

En esta sección se hace referencia a las funcionalidades que lo ayudan a detectar actividades anómalas que indican intentos inusuales y potencialmente dañinos para acceder o vulnerar las bases de datos. También se describen los procedimientos recomendados para configurar la auditoría de base de datos a fin de realizar un seguimiento de los eventos de base de datos y capturarlos.

### <a name="protect-databases-against-attacks"></a>Protección de las bases de datos frente a ataques

La protección de amenazas avanzada le permite detectar posibles amenazas y responder a ellas si fuera necesario, gracias a las alertas de seguridad sobre actividades anómalas que se proporcionan.

**Implementación**:

- Use [Advanced Threat Protection para SQL](threat-detection-overview.md#alerts) para detectar intentos inusuales y potencialmente dañinos para acceder o vulnerar las bases de datos, entre los que se incluyen:
  - Ataque por inyección de código SQL.
  - Robo o pérdida de credenciales.
  - Abuso de privilegios.
  - Filtración de datos.

**Procedimientos recomendados**:

- Configure [Advanced Data Security (ADS)](advanced-data-security.md#getting-started-with-ads) para un servidor específico o una instancia administrada. También puede configurar ADS para todos los servidores y las instancias administradas de una suscripción. Para ello, cambie al [nivel estándar de Azure Security Center](../../security-center/security-center-pricing.md).

- Para obtener una experiencia de investigación completa, se recomienda habilitar  [SQL Database Auditing](../../azure-sql/database/auditing-overview.md). Con la auditoría, puede realizar un seguimiento de los eventos de base de datos y escribirlos en un registro de auditoría en una cuenta de Azure Storage o en el área de trabajo de Azure Log Analytics.

### <a name="audit-critical-security-events"></a>Auditoría de eventos de seguridad críticos

El seguimiento de los eventos de base de datos ayuda a comprender la actividad de las bases de datos. Puede obtener información sobre discrepancias y anomalías que pueden indicar problemas en el negocio o infracciones de seguridad sospechosas. También habilita y facilita el cumplimiento de los estándares de cumplimiento.

**Implementación**:

- Habilite  [Auditoría de SQL Database](../../azure-sql/database/auditing-overview.md) o [Auditoría de Instancia administrada](../managed-instance/auditing-configure.md) para realizar un seguimiento de los eventos de base de datos y escribirlos en un registro de auditoría en una cuenta de Azure Storage, en un área de trabajo de Log Analytics (versión preliminar) o en Event Hubs (versión preliminar).

- Los registros de auditoría se pueden escribir en una cuenta de Azure Storage, en un área de trabajo de Log Analytics para su consumo en registros de Azure Monitor o en un centro de eventos para consumirlos mediante el centro de eventos. Puede configurar cualquier combinación de estas opciones, y los registros de auditoría se escribirán en cada una.

**Procedimientos recomendados**:

- Al configurar [Auditoría de SQL Database](../../azure-sql/database/auditing-overview.md) en el servidor o [Auditoría de Instancia administrada](../managed-instance/auditing-configure.md) para auditar los eventos, se auditarán todas las bases de datos existentes y recién creadas en ese servidor.
- De manera predeterminada, la directiva de auditoría incluye todas las acciones (consultas, procedimientos almacenados e inicios de sesión correctos y erróneos) en las bases de datos, lo que puede dar lugar a un gran volumen de registros de auditoría. Se recomienda a los clientes [configurar la auditoría para diferentes tipos de acciones y grupos de acciones mediante PowerShell](../../sql-database/sql-database-auditing.md#manage-auditing). Configurarla ayudará a controlar el número de acciones auditadas y a minimizar el riesgo de pérdida de eventos. Las configuraciones de auditoría personalizadas permiten a los clientes capturar solo los datos de auditoría necesarios.
- Los registros de auditoría se pueden consumir directamente en [Azure Portal](https://portal.azure.com/) o desde la ubicación de almacenamiento que se configuró.

> [!NOTE]
> La habilitación de la auditoría en Log Analytics incurrirá en costos en función de las tarifas de ingesta. Tenga en cuenta el costo asociado al uso de esta [opción](https://azure.microsoft.com/pricing/details/monitor/), o considere la posibilidad de almacenar los registros de auditoría en una cuenta de almacenamiento de Azure.

**Recursos adicionales**:

- [Auditoría de base de datos SQL](../../azure-sql/database/auditing-overview.md)
- [Auditoría de SQL Server](https://docs.microsoft.com/sql/relational-databases/security/auditing/sql-server-audit-database-engine)

### <a name="secure-audit-logs"></a>Protección de los registros de auditoría

Restrinja el acceso a la cuenta de almacenamiento para permitir la separación de tareas y separar a los DBA de los auditores.

**Implementación**:

- Al guardar los registros de auditoría en Azure Storage, asegúrese de que el acceso a la cuenta de almacenamiento está restringido a los principios de seguridad mínimos. Controle quién tiene acceso a la cuenta de almacenamiento.
- Para más información, consulte [Autorización de acceso a Azure Storage](../../storage/common/storage-auth.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

**Procedimientos recomendados**:

- Controlar el acceso al destino de auditoría es un concepto clave en la separación de los DBA de los auditores.

- Al auditar el acceso a datos confidenciales, considere la posibilidad de proteger los datos con el cifrado de datos para evitar la fuga de información al auditor. Para más información, consulte la sección [Protección de datos confidenciales en uso de usuarios no autorizados con privilegios elevados](#protect-sensitive-data-in-use-from-high-privileged-unauthorized-users).

## <a name="security-management"></a>Administración de seguridad

En esta sección se describen los distintos aspectos y procedimientos recomendados para administrar su posición de seguridad de las bases de datos. Incluye procedimientos recomendados para garantizar que las bases de datos están configuradas para cumplir con los estándares de seguridad, para detectar y para clasificar y realizar el seguimiento del acceso a datos potencialmente confidenciales en las bases de datos.

### <a name="ensure-that-the-databases-are-configured-to-meet-security-best-practices"></a>Comprobación de que las bases de datos están configuradas para cumplir con los procedimientos recomendados de seguridad

Mejore de manera proactiva la seguridad de la base de datos mediante la detección y corrección de posibles vulnerabilidades de la base de datos.

**Implementación**:

- Habilite la [Evaluación de vulnerabilidades de SQL](https://docs.microsoft.com/sql/relational-databases/security/sql-vulnerability-assessment) (VA) para examinar la base de datos en busca de problemas de seguridad y para que se ejecute periódicamente en las bases de datos.

**Procedimientos recomendados**:

- Para empezar, ejecute VA en sus bases de datos y realice una iteración mediante la corrección de las comprobaciones con errores que se oponen a los procedimientos recomendados de seguridad. Configure líneas de base para las configuraciones aceptables hasta obtener un examen _limpio_ o superar todas las comprobaciones.  

- Configure exámenes periódicos recurrentes para que se ejecuten una vez a la semana y configure al usuario correspondiente para que reciba los correos electrónicos de resumen.

- Revise el resumen de VA después de cada examen semanal. En el caso de las vulnerabilidades detectadas, evalúe el desfase del resultado del examen anterior y determine si se debe resolver la comprobación. Revise si hay una razón legítima para el cambio en la configuración.

- Resuelva las comprobaciones y actualice las líneas de base cuando proceda. Cree elementos de vale para resolver acciones y realizar un seguimiento de estos hasta que se resuelvan.

**Recursos adicionales**:

- [Evaluación de vulnerabilidad de SQL](https://docs.microsoft.com/sql/relational-databases/security/sql-vulnerability-assessment)
- [El servicio de evaluación de vulnerabilidad de SQL le ayuda a identificar los puntos vulnerables de la base de datos](sql-vulnerability-assessment.md)

### <a name="identify-and-tag-sensitive-data"></a>Identificación y etiquetado de datos confidenciales

Detecte las columnas que posiblemente contengan datos confidenciales. Lo que se considera datos confidenciales depende en gran medida del cliente, del reglamento de cumplimiento, etc. y los deben evaluar los usuarios a cargo de esos datos. Clasifique las columnas para utilizar escenarios avanzados de protección y auditoría basadas en la confidencialidad.

**Implementación**:

- Use [Clasificación y detección de datos de SQL](data-discovery-and-classification-overview.md) para detectar, clasificar, etiquetar y proteger los datos confidenciales en las bases de datos.
  - Vea las recomendaciones de clasificación creadas por la detección automatizada en el panel Clasificación y detección de datos SQL. Acepte las clasificaciones pertinentes, de modo que los datos confidenciales se etiqueten de forma persistente con las etiquetas de clasificación.
  - Agregue manualmente las clasificaciones para los campos de datos confidenciales adicionales que no haya descubierto el mecanismo automatizado.
- Para obtener más información, consulte [Clasificación y detección de datos de SQL](https://docs.microsoft.com/sql/relational-databases/security/sql-data-discovery-and-classification).

**Procedimientos recomendados**:

- Supervise periódicamente el panel de clasificación para obtener una evaluación precisa del estado de la clasificación de la base de datos. Un informe sobre el estado de la clasificación de la base de datos se puede exportar o imprimir para compartirlo con fines de cumplimiento y auditoría.

- Supervise continuamente el estado de los datos confidenciales recomendados en Evaluación de vulnerabilidad de SQL. Realice un seguimiento de la regla de detección de datos confidenciales e identifique cualquier desfase en las columnas recomendadas para la clasificación.  

- Use la clasificación de una manera que esté adaptada a las necesidades específicas de su organización. Personalice la directiva Information Protection (etiquetas de confidencialidad, tipos de información, lógica de detección) en la directiva [SQL Information Protection](../../security-center/security-center-info-protection-policy.md) de Azure Security Center.

### <a name="track-access-to-sensitive-data"></a>Seguimiento del acceso a datos confidenciales

Supervise quién tiene acceso a los datos confidenciales y capture consultas sobre los datos confidenciales en los registros de auditoría.

**Implementación**:

- Use la auditoría de SQL y la clasificación de datos en combinación.
  - En el registro de [auditoría de SQL Database](../../azure-sql/database/auditing-overview.md), puede realizar un seguimiento del acceso a datos confidenciales específicamente. También puede ver información, como los datos a los que se accedió, así como su etiqueta de confidencialidad. Para obtener más información, consulte [Clasificación y detección de datos](data-discovery-and-classification-overview.md) y [Auditoría del acceso a datos confidenciales](data-discovery-and-classification-overview.md#audit-sensitive-data).

**Procedimientos recomendados**:

- Consulte los procedimientos recomendados para las secciones de Auditoría y Clasificación de datos:
  - [Auditoría de eventos de seguridad críticos](#audit-critical-security-events)
  - [Identificación y etiquetado de datos confidenciales](#identify-and-tag-sensitive-data)

### <a name="visualize-security-and-compliance-status"></a>Visualización del estado de la seguridad y del cumplimiento

Use un sistema de administración de la seguridad de infraestructura unificada que refuerce la posición de seguridad de los centros de datos (incluidas las bases de datos de SQL Database). Vea una lista de recomendaciones sobre la seguridad de las bases de datos y el estado de cumplimiento.

**Implementación**:

- Supervise las recomendaciones de seguridad relacionadas con SQL y las amenazas activas en [Azure Security Center](https://azure.microsoft.com/documentation/services/security-center/).

## <a name="common-security-threats-and-potential-mitigations"></a>Amenazas de seguridad comunes y posibles mitigaciones

Esta sección lo ayuda a encontrar medidas de seguridad para protegerse contra determinados vectores de ataque. Se espera que la mayoría de las mitigaciones se puedan lograr siguiendo una o varias de las directrices de seguridad mencionadas.

### <a name="security-threat-data-exfiltration"></a>Amenaza de seguridad: Filtración de datos

La filtración de datos es la copia, transferencia o recuperación no autorizada de los datos de un equipo o servidor. Consulte una definición de [filtración de datos](https://en.wikipedia.org/wiki/Data_exfiltration) en Wikipedia.

La conexión al servidor a través de un punto de conexión público presenta un riesgo de filtración de datos, ya que requiere que los clientes abran sus firewalls a direcciones IP públicas.  

**Escenario 1**: una aplicación en una VM de Azure se conecta a una base de datos de una instancia de Azure SQL Database. Un actor no autorizado obtiene acceso a la máquina virtual y la pone en peligro. En este escenario, la filtración de datos significa que una entidad externa que usa la VM no autorizada se conecta a la base de datos, copia la información personal y la almacena en un almacenamiento de blobs o en una base de datos SQL Database diferente en una suscripción distinta.

**Escenario 2**: un DBA no autorizado. A menudo, este escenario lo generan los clientes confidenciales de sectores regulados. En este escenario, un usuario con privilegios elevados podría copiar datos de Azure SQL Database a otra suscripción no controlada por el propietario de los datos.

**Posibles mitigaciones**:

En la actualidad, Azure SQL Database e Instancia administrada de SQL ofrecen las técnicas siguientes para mitigar las amenazas de filtración de datos:

- Use una combinación de reglas de permiso y denegación en los grupos de seguridad de red de las máquinas virtuales de Azure para controlar a qué regiones se puede tener acceso desde la máquina virtual.
- Si usa un servidor de SQL Database, establezca las siguientes opciones:
  - Establezca Allow Azure Services (Permitir servicios de Azure) en OFF.
  - Permita solo el tráfico desde la subred que contiene la máquina virtual de Azure mediante la configuración de una regla de firewall de red virtual.
  - Use [Private Link](../../private-link/private-endpoint-overview.md).
- En el caso de SQL Managed Instance, el uso del acceso IP privado de manera predeterminada aborda la primera preocupación relacionada con la filtración de datos de una máquina virtual no autorizada. Active la característica de delegación de subred en una subred para establecer automáticamente la directiva más restrictiva en una subred de Instancia administrada de SQL.
- El problema de DBA no autorizado se expone más con SQL Managed Instance, ya que tiene un área expuesta mayor y los requisitos de red son visibles para los clientes. La mejor mitigación para esto es aplicar todos los procedimientos de esta guía de seguridad para evitar en primer lugar el escenario de un DBA no autorizado (no solo para la filtración de datos). Always Encrypted es un método para proteger los datos confidenciales mediante su cifrado e impidiendo que el DBA pueda acceder a la clave.

## <a name="security-aspects-of-business-continuity-and-availability"></a>Aspectos de seguridad de la disponibilidad y continuidad empresarial

La mayoría de los estándares de seguridad abordan la disponibilidad de los datos en términos de la continuidad operativa, gracias a la implementación de funcionalidades de redundancia y conmutación por error para evitar únicos puntos de error. En escenarios de desastre, un procedimiento común es mantener copias de seguridad de los archivos de datos y de registro. En la sección siguiente se proporciona información general de alto nivel sobre las funcionalidades integradas en Azure. También se proporcionan opciones adicionales que se pueden configurar para satisfacer necesidades específicas:

- Azure ofrece alta disponibilidad integrada: [Alta disponibilidad con SQL Database y SQL Managed Instance](high-availability-sla.md)

- El nivel Crítico para la empresa incluye grupos de conmutación por error, zonas de disponibilidad múltiple, copias de seguridad de registros completas y diferenciales y copias de seguridad de restauración a un momento dado habilitadas de forma predeterminada:  
  - [Alta disponibilidad: configuración de redundancia de zona](high-availability-sla.md#zone-redundant-configuration)
  - [Copias de seguridad automatizadas](automated-backups-overview.md)
  - [Recuperación de una base de datos mediante copias de seguridad automatizadas de base de datos - Restauración a un momento dado](recovery-using-backups.md#point-in-time-restore)

- Se pueden configurar otras características de continuidad empresarial, como los grupos de conmutación por error automática en diferentes zonas geográficas de Azure, tal como se describe aquí: [Información general sobre la continuidad empresarial](business-continuity-high-availability-disaster-recover-hadr-overview.md)

## <a name="next-steps"></a>Pasos siguientes

- Consulte [Información general sobre las funcionalidades de seguridad de Azure SQL Database](security-overview.md).
