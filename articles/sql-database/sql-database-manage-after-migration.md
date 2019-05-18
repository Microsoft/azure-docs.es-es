---
title: 'Administración de bases de datos únicas y agrupadas después de la migración: Azure SQL Database | Microsoft Docs'
description: Obtenga información acerca de cómo administrar la base de datos tras la migración a Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: joesackmsft
ms.author: josack
ms.reviewer: sstein
manager: craigg
ms.date: 02/13/2019
ms.openlocfilehash: 4424e53689714625ebc791df250956463452c3cb
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65791495"
---
# <a name="new-dba-in-the-cloud--managing-your-single-and-pooled-databases-in-azure-sql-database"></a>Nuevo DBA en la nube: administración de las bases de datos únicas o agrupadas en Azure SQL Database

El cambio del entorno autocontrolado y autoadministrado tradicional a un entorno PaaS puede parecer un poco abrumador al principio. Como desarrollador de aplicaciones o DBA, deseará conocer las capacidades básicas de la plataforma que le ayudarán a mantener la disponibilidad, el rendimiento, la seguridad y la resistencia de la aplicación. Ese es precisamente el objetivo de este artículo. El artículo organiza brevemente los recursos y ofrece instrucciones sobre cómo usar mejor las funcionalidades clave de SQL Database con bases de datos únicas y agrupadas para administrar y mantener la aplicación, que se ejecute de forma eficaz y lograr mejores resultados en la nube. Por lo general, el público al que va dirigido este artículo estará formado por personas que:

- Evalúan la migración de sus aplicaciones a Azure SQL Database (y modernizan así las suyas).
- Estén en proceso de migrar sus aplicaciones (escenario de migración en curso).
- Hayan completado recientemente la migración a Azure SQL DB (nuevo DBA en la nube).

En este artículo se tratan algunas de las características básicas de Azure SQL Database como una plataforma que puede aprovechar fácilmente al trabajar con bases de datos únicas y agrupadas en grupos elásticos. Son las siguientes:

- Recuperación ante desastres y continuidad empresarial (BCDR)
- Seguridad y cumplimiento normativo
- Supervisión y mantenimiento inteligentes de la base de datos
- Movimiento de datos

> [!NOTE]
> Este artículo se aplica a las siguientes opciones de implementación en Azure SQL Database: bases de datos únicas y grupos elásticos. No es válido para la opción de implementación de instancias administradas en SQL Database.

## <a name="business-continuity-and-disaster-recovery-bcdr"></a>Recuperación ante desastres y continuidad empresarial (BCDR)

Las capacidades de recuperación ante desastres y continuidad empresarial le permiten continuar con su actividad empresarial del modo habitual en caso de desastre. El desastre podría ser un evento de nivel de base de datos (por ejemplo, alguien elimina erróneamente una tabla de vital importancia) o un evento de nivel de centro de datos (una catástrofe regional, como un tsunami).

### <a name="how-do-i-create-and-manage-backups-on-sql-database"></a>Cómo crear y administrar copias de seguridad en SQL Database

En Azure SQL DB no se crean copias de seguridad porque no es necesario. SQL Database realiza copias de seguridad de las bases de datos automáticamente por usted, por lo que ya no tiene que preocuparse de programar, realizar y administrar copias de seguridad. La plataforma realiza una copia de seguridad completa cada semana, una copia de seguridad diferencial cada pocas horas y una copia de seguridad de registro cada 5 minutos para garantizar que la recuperación ante desastres sea eficaz y la pérdida de datos mínima. La primera copia de seguridad completa se realiza cuando se crea una base de datos. Estas copias de seguridad están disponibles durante un período determinado denominado "período de retención", que varía según el nivel de servicio que elija. SQL Database proporciona la capacidad de restaurar a un momento dado dentro de este período de retención mediante la [recuperación a un momento dado (PITR)](sql-database-recovery-using-backups.md#point-in-time-restore).

|Nivel de servicio|Período de retención en días|
|---|:---:|
|Básica|7|
|Estándar|35|
|Premium|35|
|||

Además, la característica de [retención a largo plazo (LTR)](sql-database-long-term-retention.md) permite mantener archivos de copia de seguridad durante un período de tiempo mucho mayor (hasta 10 años) y restaurar los datos a partir de estas copias de seguridad en cualquier momento de dicho período. Además, las copias de seguridad de base de datos se mantienen en almacenamientos con replicación geográfica para garantizar la resistencia ante una catástrofe regional. También puede restaurar estas copias de seguridad en cualquier región de Azure en cualquier momento dentro del período de retención. Consulte [Introducción a la continuidad del negocio](sql-database-business-continuity.md).

### <a name="how-do-i-ensure-business-continuity-in-the-event-of-a-datacenter-level-disaster-or-regional-catastrophe"></a>Cómo se garantiza la continuidad empresarial en caso de un desastre en el nivel de centro de datos o una catástrofe regional

Porque las copias de seguridad de base de datos se almacenan en almacenamiento con replicación geográfica para garantizar que, en caso de un desastre regional, puede restaurar la copia de seguridad en otra región de Azure. Esto se denomina restauración geográfica. El RPO (objetivo de punto de recuperación) para hacerlo suele ser inferior a 1 hora y el ERT (tiempo estimado de recuperación), de algunos minutos a horas.

Para las bases de datos críticas, Azure SQL DB ofrece replicación geográfica activa. Lo que hace básicamente es crear una copia secundaria con replicación geográfica de la base de datos original en otra región. Por ejemplo, si la base de datos se hospeda inicialmente en la región Oeste de EE. UU. de Azure y quiere resistencia ante desastres regionales. Debe crear una réplica geográfica activa de la base de datos en Oeste de EE. UU. que diga Este de EE. UU. Cuando se produzca el desastre en la región Oeste de EE. UU., puede conmutar por error a la región Este de EE. UU. Su configuración en un grupo de conmutación por error automática es aún mejor, porque ello garantiza que la base de datos conmute por error automáticamente a la copia secundaria de la región Este de EE. UU. en caso de un desastre. El RPO para hacerlo es menos de 5 segundos y el ERT, más de 30 segundos.

Si no se configura un grupo de conmutación por error automática, la aplicación debe supervisar activamente la existencia de un desastre e iniciar una conmutación por error a la base de datos secundaria. Puede crear hasta cuatro réplicas geográficas activas de este tipo en diferentes regiones de Azure. Aún es mejor. También puede obtener acceso de solo lectura a estas réplicas geográficas activas secundarias. Esto resulta muy útil para reducir la latencia de un escenario de aplicación distribuida geográficamente.

### <a name="how-does-my-disaster-recovery-plan-change-from-on-premises-to-sql-database"></a>Cómo cambia mi plan de recuperación ante desastres de local a SQL Database

En resumen, la configuración de SQL Server local tradicional requería administrar activamente la disponibilidad mediante características tales como clústeres de conmutación por error, creación de reflejo de la base de datos, replicación de transacciones o trasvase de registros, así como mantener y administrar las copias de seguridad para garantizar la Continuidad empresarial. Con SQL Database, la plataforma administra estas características por usted, para que pueda centrarse en el desarrollo y la optimización de la aplicación de base de datos y no tenga que preocuparse tanto por la administración de desastres. Puede tener planes de recuperación ante desastres y de copia de seguridad configurados y en funcionamiento con unos pocos clics en Azure Portal (o algunos comandos con las API de PowerShell).

Para obtener más información sobre la recuperación ante desastres, vea: [Azure SQL Db Disaster Recovery 101](https://azure.microsoft.com/blog/azure-sql-databases-disaster-recovery-101/) (Recuperación ante desastres de Azure SQL DB 101)

## <a name="security-and-compliance"></a>Seguridad y cumplimiento normativo

SQL Database se toma la seguridad y la privacidad muy en serio. La seguridad en SQL Database está disponible en el nivel de base de datos y en el nivel de plataforma, y se entiende mejor cuando se clasifica por categorías en varias capas. Cada capa le proporciona seguridad óptima para su aplicación y le permite controlarla. Las capas son:

- Identidad y autenticación ([Autenticación de Windows/SQL y autenticación de Azure Active Directory [AAD]](sql-database-control-access.md)).
- Supervisión de la actividad ([Auditoría](sql-database-auditing.md) y [detección de amenazas](sql-database-threat-detection.md)).
- Protección de datos reales ([Cifrado de datos transparente [TDE]](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql) y [Always Encrypted [AE]](/sql/relational-databases/security/encryption/always-encrypted-database-engine)).
- Control del acceso a datos confidenciales y con privilegios ([seguridad de nivel de fila](/sql/relational-databases/security/row-level-security) y [Enmascaramiento dinámico de datos](/sql/relational-databases/security/dynamic-data-masking)).

[Azure Security Center](https://azure.microsoft.com/services/security-center/) ofrece administración de seguridad centralizada de todas las cargas de trabajo que se ejecutan en Azure, de forma local y en otras nubes. Puede ver si se han configurado en todos los recursos elementos de protección esenciales de SQL Database como la [auditoría](sql-database-auditing.md) y el [Cifrado de datos transparente [TDE]](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql), y crear directivas basadas en sus propios requisitos.

### <a name="what-user-authentication-methods-are-offered-in-sql-database"></a>Qué métodos de autenticación de usuario se ofrecen en SQL Database

En SQL Database, se ofrecen [dos métodos de autenticación](sql-database-control-access.md#authentication):

- [Autenticación con Azure Active Directory](sql-database-aad-authentication.md)
- Autenticación de SQL

No se admite la autenticación de Windows tradicional. Azure Active Directory (AD) es un servicio de administración de identidades y acceso centralizado. Con esto puede proporcionar cómodamente un acceso de inicio de sesión único (SSO) a todo el personal de su organización. Esto significa que las credenciales se comparten entre todos los servicios de Azure para simplificar la autenticación. AAD es compatible con [MFA (Multi-Factor Authentication)](sql-database-ssms-mfa-authentication.md) y con unos [pocos clics](../active-directory/hybrid/how-to-connect-install-express.md) AAD puede integrarse con Windows Server Active Directory. La autenticación de SQL funciona del mismo modo que la ha estado usando en el pasado. Debe proporcionar un nombre de usuario y una contraseña, y puede autenticar a los usuarios en cualquier base de datos de un servidor de SQL Database determinado. Esto también permite a SQL Database y SQL Data Warehouse ofrecer autenticación multifactor y cuentas de usuario de invitado dentro de un dominio de Azure AD. Si ya tiene una instancia de Active Directory local, puede federar el directorio con Azure Active Directory para extender su directorio a Azure.

|**Si usted...**|**SQL Database/SQL Data Warehouse**|
|---|---|
|No desea usar Azure Active Directory (AD) en Azure|Use [autenticación de SQL](sql-database-security-overview.md)|
|Ha utilizado AD en SQL Server local|[Federe AD con Azure AD](../active-directory/hybrid/whatis-hybrid-identity.md) y use la autenticación de Azure AD. De este modo, puede usar el inicio de sesión único.|
|Necesita aplicar autenticación multifactor (MFA)|Exija MFA como una directiva mediante el [acceso condicional de Microsoft](sql-database-conditional-access.md) y utilice la [autenticación universal de Azure AD con compatibilidad con MFA](sql-database-ssms-mfa-authentication.md).|
|Tiene cuentas de invitado procedentes de cuentas de Microsoft (live.com, outlook.com) o de otros dominios (gmail.com)|Use la [autenticación universal de Azure AD](sql-database-ssms-mfa-authentication.md) en SQL Database y Data Warehouse, que aprovecha la [colaboración B2B de Azure AD](../active-directory/active-directory-b2b-what-is-azure-ad-b2b.md).|
|Ha iniciado sesión en Windows con sus credenciales de Azure AD desde un dominio federado|Use la [autenticación integrada de Azure AD](sql-database-aad-authentication-configure.md).|
|Ha iniciado sesión en Windows con sus credenciales desde un dominio no federado con Azure|Use la [autenticación integrada de Azure AD](sql-database-aad-authentication-configure.md).|
|Tiene servicios de nivel intermedio que necesitan conectarse a SQL Database o SQL Data Warehouse|Use la [autenticación integrada de Azure AD](sql-database-aad-authentication-configure.md).|
|||

### <a name="how-do-i-limit-or-control-connectivity-access-to-my-database"></a>Cómo se limita o controla el acceso de conectividad a mi base de datos

Existen varias técnicas a su disposición que podría utilizar para alcanzar la organización óptima de la conectividad para su aplicación.

- Reglas de firewall
- Puntos de conexión de servicio de red virtual
- Direcciones IP reservadas

#### <a name="firewall"></a>Firewall

Un firewall impide el acceso al servidor desde una entidad externa al permitir que solo entidades específicas obtengan acceso al servidor de SQL Database. De forma predeterminada, todas las conexiones y bases de datos del servidor de SQL Database están prohibidas, excepto las conexiones que proceden de otros servicios de Azure. Con una regla de firewall puede abrir el acceso al servidor únicamente a las entidades (por ejemplo, una máquina de desarrollador) que apruebe al permitir la dirección IP de ese equipo a través del firewall. También permite especificar un intervalo de direcciones IP al que quiere permitir el acceso al servidor de SQL Database. Por ejemplo, las direcciones IP de una máquina de desarrollador de su organización pueden agregarse a la vez mediante la especificación de un intervalo en la página Configuración del firewall.

Puede crear reglas de firewall en el nivel de servidor o el nivel de base de datos. Las reglas de firewall de IP en el nivel de servidor pueden crearse mediante Azure Portal o con SSMS. Para más información acerca de cómo configurar una regla de firewall en el nivel de servidor y en el de base de datos, consulte el tema sobre la [creación de reglas de firewall de IP en SQL Database](sql-database-security-tutorial.md#create-firewall-rules).

#### <a name="service-endpoints"></a>Puntos de conexión de servicio

De forma predeterminada, su instancia de SQL Database está configurada para "Permitir que los servicios de Azure accedan al servidor", lo que significa que todas las máquinas virtuales de Azure pueden intentar conectarse a su base de datos. Estos intentos todavía tienen que autenticarse. Sin embargo, si no desea que la base de datos sea accesible para las direcciones IP de Azure, puede deshabilitar "Permitir que los servicios de Azure accedan al servidor". Asimismo, puede configurar [puntos de conexión de servicio de red virtual](sql-database-vnet-service-endpoint-rule-overview.md).

Los puntos de conexión de servicio (SE) le permiten exponer los recursos críticos de Azure solo a su propia red privada virtual en Azure. Al hacerlo, básicamente se elimina el acceso público a sus recursos. El tráfico entre su red virtual y Azure se mantiene en la red troncal de Azure. Sin SE obtendrá un enrutamiento de paquetes de tunelización forzada. La red virtual fuerza el tráfico de Internet a su organización y el tráfico del servicio de Azure para que sigan la misma ruta. Con los puntos de conexión de servicio, puede optimizar esto, ya que los paquetes fluyen directamente de la red virtual al servicio en la red troncal de Azure.

![Puntos de conexión de servicio de red virtual](./media/sql-database-manage-after-migration/vnet-service-endpoints.png)

#### <a name="reserved-ips"></a>Direcciones IP reservadas

Otra opción es aprovisionar [direcciones IP reservadas](../virtual-network/virtual-networks-reserved-public-ip.md) para las máquinas virtuales e incluir en la lista de permitidas esas direcciones IP específicas de las máquinas virtuales en la configuración de firewall del servidor. Mediante la asignación de direcciones IP reservadas, se evita el problema de tener que actualizar las reglas del firewall con el cambio de direcciones IP.

### <a name="what-port-do-i-connect-to-sql-database-on"></a>A qué puerto debo conectarme en SQL Database

Puerto 1433. SQL Database se comunica a través de este puerto. Para conectarse desde una red corporativa, tiene que agregar una regla de salida en la configuración del firewall de su organización. Como norma, evite exponer el puerto 1433 fuera del límite de Azure.

### <a name="how-can-i-monitor-and-regulate-activity-on-my-server-and-database-in-sql-database"></a>Cómo puedo supervisar y regular la actividad en mi servidor y mi base de datos en SQL Database

#### <a name="sql-database-auditing"></a>Auditoría de SQL Database

Con SQL Database, puede activar la auditoría para realizar el seguimiento de los eventos de base de datos. La [auditoría de SQL Database](sql-database-auditing.md) registra eventos de base de datos y los escribe en un archivo de registro de auditoría en su cuenta de Azure Storage. La auditoría es especialmente útil si desea obtener información sobre posibles infracciones de seguridad y directivas, mantener el cumplimiento normativo, etc. Permite definir y configurar determinadas categorías de eventos que cree que necesitan auditoría y, en función de ello, puede obtener informes preconfigurados y un panel para obtener una visión general de los eventos que se producen en la base de datos. Puede aplicar estas directivas de auditoría en el nivel de base de datos o en el nivel de servidor. Para obtener instrucciones sobre cómo activar la auditoría para el servidor o la base de datos, vea: [Habilitación de la auditoría de SQL Database](sql-database-security-tutorial.md#enable-security-features).

#### <a name="threat-detection"></a>Detección de amenazas

Con la [detección de amenazas](sql-database-threat-detection.md), obtendrá la capacidad de actuar fácilmente sobre las infracciones de seguridad o directivas que la auditoría detecte. No es necesario ser un experto en seguridad para resolver posibles amenazas o infracciones en el sistema. La detección de amenazas también tiene algunas capacidades integradas, como la detección de SQL Injection. SQL Injection es un intento de modificar o poner en peligro los datos y una manera bastante común de atacar una aplicación de base de datos en general. La detección de amenazas ejecuta varios conjuntos de algoritmos que detectan posibles vulnerabilidades y ataques de inyección de SQL, así como patrones de acceso de base de datos anómalos (por ejemplo, el acceso desde una ubicación inusual o una entidad de seguridad desconocida). Los responsables de seguridad u otros administradores designados reciben una notificación por correo electrónico si se detecta una amenaza en la base de datos. Cada notificación proporciona detalles de la actividad sospechosa y recomendaciones acerca de cómo investigar más y mitigar la amenaza. Para obtener información sobre cómo activar la detección de amenazas, consulte: [Habilitar la detección de amenazas](sql-database-security-tutorial.md#enable-security-features).

### <a name="how-do-i-protect-my-data-in-general-on-sql-database"></a>Cómo puedo proteger mis datos en general en SQL Database

El cifrado proporciona un mecanismo seguro para proteger los datos confidenciales de los intrusos. Los datos cifrados son inútiles para el intruso sin la clave de descifrado. Por lo tanto, agrega una capa de protección adicional sobre las capas de seguridad existentes integradas en SQL Database. Existen dos aspectos sobre la protección de datos en SQL Database:

- Sus datos en reposo en archivos de datos y de registro
- Sus datos en curso

De forma predeterminada, en SQL Database, los datos en reposo de los archivos de datos y de registro en el subsistema de almacenamiento siempre están completamente cifrados a través del [Cifrado de datos transparente [TDE]](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql). También se cifran las copias de seguridad. Con TDE, no existen cambios necesarios en la aplicación que accede a estos datos. El cifrado y el descifrado se producen de forma transparente, de ahí el nombre.
Para proteger los datos confidenciales en curso y en reposo, SQL Database proporciona una característica denominada [Always Encrypted (AE)](/sql/relational-databases/security/encryption/always-encrypted-database-engine). AE es una forma de cifrado en el cliente que cifra las columnas confidenciales de la base de datos (para lo que están en texto cifrado para los administradores de la base de datos y los usuarios no autorizados). El servidor recibe los datos cifrados con los que comenzar. La clave de Always Encrypted también se almacena en el lado del cliente, de modo que solo los clientes autorizados pueden descifrar las columnas confidenciales. Los administradores del servidor y de los datos no pueden ver los datos confidenciales, ya que las claves de cifrado se almacenan en el cliente. AE cifra las columnas confidenciales de la tabla de extremo a extremo, desde los clientes no autorizados hasta el disco físico. AE admite comparaciones de igualdad en la actualidad, por lo que los DBA pueden seguir consultando las columnas cifradas como parte de los comandos SQL. Always Encrypted puede utilizarse con diversos almacenes de claves, como [Azure Key Vault](sql-database-always-encrypted-azure-key-vault.md), el almacén de certificados de Windows y módulos de seguridad de hardware local.

|**Características**|**Always Encrypted**|**Cifrado de datos transparente**|
|---|---|---|
|**Cifrado**|Completa|Datos en reposo|
|**El servidor de bases de datos puede acceder a datos confidenciales**|Sin |Sí, ya que el cifrado es para los datos en reposo|
|**Operaciones permitidas de T-SQL**|Comparación de igualdad|Toda el área expuesta de T-SQL está disponible|
|**Cambios de aplicación necesarios para usar la característica**|Mínimo|Muy mínimo|
|**Granularidad de cifrado**|Nivel de columna|Nivel de base de datos|
||||

### <a name="how-can-i-limit-access-to-sensitive-data-in-my-database"></a>Cómo se limita el acceso a la información confidencial en mi base de datos

Cada aplicación tiene cierta información confidencial en la base de datos debe protegerse para que no sea visible para todos los usuarios. Determinados empleados de la organización necesitan ver estos datos, pero otros no deberían poder verlos. Un ejemplo es el salario de los empleados. Aunque un administrador debe poder acceder a la información de salario de sus subordinados directos, los miembros del equipo individuales no deben tener acceso a la información de salario de sus compañeros. Otro escenario es el de los desarrolladores de datos, que pueden interactuar con datos confidenciales durante las fases de desarrollo o pruebas, por ejemplo, los números de la seguridad social de los clientes. Una vez más, no es necesario exponer esta información a los desarrolladores. En estos casos, los datos confidenciales deben enmascararse o no exponerse en absoluto. SQL Database ofrece dos enfoques para impedir que usuarios no autorizados puedan ver datos confidenciales:

[Enmascaramiento dinámico de datos](sql-database-dynamic-data-masking-get-started.md) es una característica de enmascaramiento de datos que permite limitar la exposición de información confidencial ocultándola a los usuarios sin privilegios en el nivel de aplicación. Debe definir una regla de enmascaramiento que pueda crear un patrón de enmascaramiento (por ejemplo, mostrar solo los cuatro últimos dígitos de un número del seguro social de identificación nacional: XXX-XX-0000 y marcar la mayoría como Xs) e identificar qué usuarios se excluirán de la regla de enmascaramiento. El enmascaramiento ocurre sobre la marcha y hay varias funciones de enmascaramiento disponibles para las distintas categorías de datos. El enmascaramiento dinámico de datos permite detectar datos confidenciales en la base de datos y aplicarles el enmascaramiento automáticamente.

La [seguridad de nivel de fila](/sql/relational-databases/security/row-level-security) le permite controlar el acceso en el nivel de fila. Esto quiere decir que determinadas filas de una tabla de base de datos se ocultan en función del usuario que ejecuta la consulta (pertenencia a un grupo o contexto de ejecución). La restricción de acceso se realiza en el nivel de base de datos en lugar de en una capa de aplicación para simplificar la lógica de la aplicación. Para empezar, debe crear un predicado de filtro y filtrar las filas que no se muestran y la directiva de seguridad. A continuación, debe definir quién tiene acceso a estas filas. Por último, el usuario final ejecuta la consulta y, en función de los privilegios del usuario, ve esas filas restringidas o no puede verlas.

### <a name="how-do-i-manage-encryption-keys-in-the-cloud"></a>Cómo se administran las claves de cifrado en la nube

Hay opciones de administración de claves para Always Encrypted (cifrado de cliente) y el Cifrado de datos transparente (cifrado en reposo). Se recomienda la rotación periódica de las claves de cifrado. La frecuencia de rotación debe alinearse con la normativa y con los requisitos de cumplimiento internos de la organización.

#### <a name="transparent-data-encryption-tde"></a>Cifrado de datos transparente (TDE)

hay una jerarquía de dos claves en TDE: los datos de cada base de datos del usuario se cifran mediante una clave de cifrado única para la base de datos AES-256 simétrica (DEK), que a su vez se cifra con una clave maestra RSA 2048 asimétrica única para el servidor. La clave maestra se puede administrar de las siguientes maneras:

- Automáticamente a través de la plataforma: SQL Database.
- Mediante [Azure Key Vault](sql-database-always-encrypted-azure-key-vault.md) como el almacén de claves.

De forma predeterminada, el servicio SQL Database administra la clave maestra de Cifrado de datos transparente para su comodidad. Si la organización quiere tener control sobre la clave maestra, existe la opción de usar Azure Key Vault (sql-database-always-encrypted-azure-key-vault.md) como el almacén de claves. Al usar Azure Key Vault, su organización asume el control sobre el aprovisionamiento de claves, la rotación y el control de permisos. La [rotación o cambio del tipo de una clave maestra TDE](/sql/relational-databases/security/encryption/transparent-data-encryption-byok-azure-sql-key-rotation) es rápida, ya que solo se vuelve a cifrar la DEK. Para organizaciones con separación de roles entre la seguridad y la administración de datos, un administrador de seguridad puede aprovisionar el material de clave para la clave maestra TDE en Azure Key Vault y proporcionar un identificador de Azure Key Vault al administrador de base de datos que se usará para el cifrado en reposo en un servidor. Key Vault está diseñado de modo que Microsoft no pueda ver ni extraer ninguna de sus claves de cifrado. También obtiene una administración centralizada de claves para su organización.

#### <a name="always-encrypted"></a>Always Encrypted

También hay una [jerarquía de dos claves](/sql/relational-databases/security/encryption/overview-of-key-management-for-always-encrypted) en Always Encrypted; una columna de datos confidenciales se cifra mediante una clave de cifrado de 256 columnas (CEK) AES, que a su vez se cifra con una clave maestra de columna (CMK). Los controladores de cliente proporcionados para Always Encrypted no tienen limitaciones en la longitud de las claves CMK. El valor cifrado de la CEK se almacena en la base de datos y la CMK se almacena en un almacén de claves de confianza, como el almacén de certificados de Windows, Azure Key Vault o un módulo de seguridad de hardware.

- Tanto la [CEK como la CMK](/sql/relational-databases/security/encryption/rotate-always-encrypted-keys-using-powershell) se pueden rotar.
- La rotación de la CEK es una operación de tamaño de datos, que puede llevar mucho tiempo en función del tamaño de las tablas que contienen las columnas cifradas. Por lo tanto, es recomendable planear las rotaciones de la CEK en consecuencia.
- No obstante, la rotación de CMK no interfiere con el rendimiento de la base de datos y puede hacerse con roles separados.

El diagrama siguiente muestra las opciones de almacén de claves para las claves maestras de columna en Always Encrypted

![Proveedores de almacén de CMK de Always Encrypted](./media/sql-database-manage-after-migration/always-encrypted.png)

### <a name="how-can-i-optimize-and-secure-the-traffic-between-my-organization-and-sql-database"></a>Cómo puedo optimizar y proteger el tráfico entre mi organización y SQL Database

El tráfico de red entre su organización y SQL Database suele enrutarse a través de la red pública. Sin embargo, si decide optimizar esta ruta de acceso y hacerla más seguro, puede explorar ExpressRoute. ExpressRoute le permite esencialmente ampliar la red corporativa a la plataforma Azure a través de una conexión privada. Al hacerlo, no pasa por la red pública de Internet. También obtiene mayores niveles de seguridad, confiabilidad y optimización de enrutamiento, lo que se traduce en menores latencias de red y velocidades mucho más rápidas de lo que experimentaría normalmente a través de la red pública de Internet. Si piensa transferir un fragmento de datos importante entre su organización y Azure, el uso de ExpressRoute puede proporcionarle ventajas económicas. Puede elegir entre tres modelos de conectividad diferentes para la conexión de su organización a Azure:

- [Colocación de intercambio en la nube](../expressroute/expressroute-connectivity-models.md#CloudExchange)
- [Conexión universal](../expressroute/expressroute-connectivity-models.md#IPVPN)
- [Conexión de punto a punto](../expressroute/expressroute-connectivity-models.md#Ethernet)

ExpressRoute permite ampliar el límite de ancho de banda que adquiere hasta el doble sin ningún cargo adicional. También es posible configurar conectividad entre regiones mediante Express Route. Para ver una lista de proveedores de conectividad de ER, consulte: [Asociados de ExpressRoute y ubicaciones de emparejamiento](../expressroute/expressroute-locations.md). En los artículos siguientes se describe ExpressRoute con más detalle:

- [Introducción a Express Route](../expressroute/expressroute-introduction.md)
- [Requisitos previos](../expressroute/expressroute-prerequisites.md)
- [Flujos de trabajo](../expressroute/expressroute-workflows.md)

### <a name="is-sql-database-compliant-with-any-regulatory-requirements-and-how-does-that-help-with-my-own-organizations-compliance"></a>SQL Database cumple con los requisitos de regulación y cómo ayuda eso con el cumplimiento de mi propia organización

SQL Database cumple con una variedad de regulaciones de cumplimiento. Para ver el conjunto más reciente de cumplimientos que se hayan cumplido por base de datos SQL, visite la [Microsoft Trust Center](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942) y profundizar en los cumplimientos que son importantes para su organización para ver si la base de datos de SQL se incluye en el compatibles Servicios de Azure. Es importante tener en cuenta que aunque SQL Database puede estar certificada como un servicio conforme, contribuye al cumplimiento del servicio en la organización pero no lo garantiza automáticamente.

## <a name="intelligent-database-monitoring-and-maintenance-after-migration"></a>Supervisión y mantenimiento inteligentes de la base de datos después de la migración

Una vez que se ha migrado la base de datos a SQL Databse, su objetivo será supervisar la base de datos (por ejemplo, comprobar el uso de los recursos o realizar comprobaciones de DBCC) y realizar un mantenimiento regular (por ejemplo, volver a generar o reorganizar índices, estadísticas, etc.). Afortunadamente, SQL Database es inteligente en el sentido de que usa las tendencias históricas y las métricas y estadísticas registradas para ayudarle de manera proactiva a supervisar y mantener la base de datos, de modo que su aplicación siempre se ejecute de forma óptima. En algunos casos, Azure SQL DB puede realizar automáticamente las tareas de mantenimiento en función de la configuración. Existen tres facetas para la supervisión de la base de datos en SQL Database:

- Supervisión y optimización del rendimiento.
- Optimización de la seguridad.
- Optimización de los costos.

### <a name="performance-monitoring-and-optimization"></a>Supervisión y optimización del rendimiento

con la información de rendimiento de consultas, puede obtener recomendaciones adaptadas para la carga de trabajo de la base de datos, de modo que sus aplicaciones puedan seguir funcionando siempre con un nivel óptimo. También se puede configurar para que estas recomendaciones se apliquen automáticamente y no tenga que preocuparse de realizar tareas de mantenimiento. Con Index Advisor, puede implementar automáticamente las recomendaciones de índices en función de la carga de trabajo, lo que se conoce como ajuste automático. Las recomendaciones evolucionan junto con los cambios de la carga de trabajo de las aplicaciones para proporcionarle sugerencias más relevantes. También tienen la opción de revisar estas recomendaciones manualmente y aplicarlas a su entera discreción.  

### <a name="security-optimization"></a>Optimización de la seguridad

SQL Database proporciona recomendaciones de seguridad accionables para ayudarle a proteger sus datos, así como detección de amenazas para identificar e investigar actividades sospechosas de la base de datos que puedan suponer una amenaza para la base de datos. [Evaluación de vulnerabilidades](sql-vulnerability-assessment.md) es un servicio de examen e informes de la base de datos que le permite supervisar el estado de seguridad de las bases de datos a escala e identificar los riesgos de seguridad y desfase respecto a una línea de base de seguridad definida por el usuario. Después de cada examen, se proporciona una lista personalizada de pasos procesables y scripts de corrección, así como un informe de evaluación que puede usarse para ayudar a cumplir con los requisitos de cumplimiento.

Con Azure Security Center, debe identificar las recomendaciones de seguridad en todos los ámbitos y aplicarlas con un solo clic.

### <a name="cost-optimization"></a>Optimización de costos

la plataforma Azure SQL analiza el historial de uso en las bases de datos de un servidor para evaluar y recomendarle opciones de optimización de costos. Este análisis suele tardar unas dos semanas en analizar y generar recomendaciones accionables. Un grupo elástico es una de esas opciones. La recomendación aparece en el portal como un banner:

![recomendaciones de grupos elásticos](./media/sql-database-manage-after-migration/elastic-pool-recommendations.png)

También puede ver este análisis en la sección "Advisor":

![recomendaciones de grupo elástico (advisor)](./media/sql-database-manage-after-migration/advisor-section.png)

### <a name="how-do-i-monitor-the-performance-and-resource-utilization-in-sql-database"></a>Cómo se supervisa el rendimiento y la utilización de recursos en SQL Database

En SQL Database puede aprovechar la información inteligente de la plataforma para supervisar el rendimiento y ajustarlo en consecuencia. Puede supervisar el rendimiento y la utilización de recursos en SQL Database utilizando los métodos siguientes:

#### <a name="azure-portal"></a>Azure Portal

Azure Portal muestra el uso de una base de datos al seleccionar la base de datos y hacer clic en el gráfico del panel Información general. Puede modificar el gráfico para mostrar varias métricas, incluido el porcentaje de CPU, el porcentaje de DTU, el porcentaje de E/S de datos, el porcentaje de las sesiones y el porcentaje de tamaño de base de datos.

![Gráfico de supervisión](./media/sql-database-manage-after-migration/monitoring-chart.png)

![Gráfico de supervisión2](./media/sql-database-manage-after-migration/chart.png)

En este gráfico también puede configurar alertas por recurso. Estas alertas le permiten responder a condiciones de los recursos con un correo electrónico, escribir a un punto de conexión HTTP/HTTPS o realizar una acción. Para más información, consulte [Creación de alertas](sql-database-insights-alerts-portal.md).

#### <a name="dynamic-management-views"></a>Vistas de administración dinámica

puede consultar la vista de administración dinámica [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) para devolver el historial de estadísticas de consumo de recursos de la última hora y la vista de catálogo del sistema [sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database) para devolver el historial de los catorce últimos días.

#### <a name="query-performance-insight"></a>Información de rendimiento de consultas

[Información de rendimiento de consultas](sql-database-query-performance.md) le permite ver un historial de las consultas de consumo de recursos principales y las consultas de larga ejecución para una base de datos específica. Puede identificar rápidamente las principales consultas por la utilización de recursos, la duración y la frecuencia de ejecución. Puede realizar un seguimiento de las consultas y detectar la regresión. Esta característica requiere que [Almacén de consultas](/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store) esté habilitado y activo para la base de datos.

![Información de rendimiento de consultas](./media/sql-database-manage-after-migration/query-performance-insight.png)

#### <a name="azure-sql-analytics-preview-in-azure-monitor-logs"></a>Los registros de Azure SQL Analytics (versión preliminar) en Azure Monitor

[Registros de Azure Monitor](../azure-monitor/insights/azure-sql.md) le permite recopilar y visualizar las métricas clave de rendimiento de Azure SQL Azure, permitiendo hasta 150.000 bases de datos de SQL y 5.000 grupos elásticos de SQL por área de trabajo. Puede utilizarlo para supervisar y recibir notificaciones. Puede supervisar SQL Database y las métricas de los grupos elásticos de varias suscripciones de Azure y puede utilizar los grupos elásticos para identificar problemas en cada capa de un pila de aplicación.

### <a name="i-am-noticing-performance-issues-how-does-my-sql-database-troubleshooting-methodology-differ-from-sql-server"></a>Estoy observando problemas de rendimiento: En qué se diferencia la metodología para solucionar problemas de SQL Database de la de SQL Server

Una parte importante de las técnicas de solución de problemas que usaría para el diagnóstico de problemas de rendimiento de la base de datos y de las consultas siguen siendo la misma. Después de todo, el mismo motor de SQL Server alimenta la nube. Sin embargo, la plataforma (Azure SQL DB) cuenta con "inteligencia" integrada. Puede ayudarle a solucionar problemas y a diagnosticar problemas de rendimiento aún más fácilmente. Puede realizar algunas de estas acciones correctivas en su nombre y, en algunos casos, corregirlas de forma proactiva automáticamente.

El enfoque para solucionar problemas de rendimiento puede suponer una ventaja considerable si se usan características inteligentes, como [Información de rendimiento de consultas (QPI)](sql-database-query-performance.md) y [Database Advisor](sql-database-advisor.md), juntas de modo que la diferencia de la metodología varíe en ese aspecto (ya no necesita hacer el trabajo manual de pulir los detalles esenciales que pueden ayudarle a solucionar el problema en cuestión). La plataforma hace el trabajo duro por usted. Un ejemplo de ello es QPI. Con QPI, puede profundizar hasta el nivel de consulta y examinar las tendencias históricas, así cómo averiguar cuándo se produjo exactamente la regresión de la consulta. Database Advisor le ofrece recomendaciones sobre aspectos que pueden ayudarle a mejorar el rendimiento general, como la falta de índices, la pérdida de índices, la parametrización de las consultas, etc.

Con la solución de problemas de rendimiento, es importante identificar si es solo la aplicación o la base de datos que la respalda, lo que influye en el rendimiento de la aplicación. A menudo el problema de rendimiento se encuentra en el nivel de aplicación. Puede ser la arquitectura o el patrón de acceso a datos. Por ejemplo, supongamos que tiene una aplicación con un exceso de intercambio de información que es sensible a la latencia de red. En este caso, la aplicación sufre porque habrá muchas solicitudes devueltas y enviadas ("exceso de intercambio de información") entre la aplicación y el servidor y, en una red congestionada, estas idas y venidas aumentan muy rápido. Para mejorar el rendimiento en este caso, puede usar [consultas por lotes](sql-database-performance-guidance.md#batch-queries). El uso de lotes resulta de gran ayuda porque ahora sus solicitudes se procesan en un lote; ello le ayuda a reducir la latencia de ida y vuelta, y mejora el rendimiento de la aplicación.

Asimismo, si observa una degradación del rendimiento general de la base de datos, puede supervisar las vistas de administración dinámica [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) y [sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database) para comprender el consumo de CPU, E/S y memoria. El rendimiento puede verse afectado porque la base de datos tiene escasez de recursos. Es posible que necesite cambiar el tamaño de proceso o el nivel de servicio en función de las peticiones de carga de trabajo que se expanden o contraen.

Para obtener un conjunto completo de recomendaciones para ajustar los problemas de rendimiento, consulte el tema sobre el [ajuste de la base de datos](sql-database-performance-guidance.md#tune-your-database).

### <a name="how-do-i-ensure-i-am-using-the-appropriate-service-tier-and-compute-size"></a>¿Cómo me aseguro de que estoy usando el nivel de servicio y el tamaño de proceso adecuados?

SQL Database ofrece varios niveles de servicio: Básico, Estándar y Premium. Con cada nivel de servicio obtiene un rendimiento predecible garantizado asociado a ese nivel de servicio. En función de la carga de trabajo, puede que tenga ráfagas de actividad donde el uso de recursos podría alcanzar el techo del tamaño de proceso actual en el que se encuentra. En tales casos, resulta útil empezar evaluando si los ajustes pueden ser de ayuda (por ejemplo, agregar o modificar un índice). Si sigue teniendo problemas de límite, considere la posibilidad de cambiar a un nivel de servicio o tamaño de proceso superior.

|**Nivel de servicio**|**Escenarios de casos de uso comunes**|
|---|---|
|**Básico**|Aplicaciones con pocos usuarios y una base de datos que no tienen grandes requisitos de simultaneidad, escalabilidad y rendimiento. |
|**Estándar**|Aplicaciones con requisitos de simultaneidad, escalabilidad y rendimiento considerables, y una demanda de E/S de baja a media. |
|**Premium**|Aplicaciones con una gran cantidad de usuarios simultáneos, y altas demandas de CPU/memoria y E/S. Las aplicaciones confidenciales con una alta latencia, un alto rendimiento y una alta simultaneidad pueden usar el nivel Premium. |
|||

Para asegurarse de que se encuentra en el tamaño de proceso adecuado, puede supervisar el consumo de recursos de consulta y base de datos a través de uno de los métodos mencionados anteriormente en "¿Cómo se supervisa el rendimiento y la utilización de recursos en SQL Database?". Si encuentra que sus consultas o bases de datos se ejecutan constantemente con un alto consumo de CPU/memoria, etc., puede considerar la posibilidad de escalar verticalmente a un tamaño de proceso superior. De forma similar, si observa que, incluso durante las horas de máxima actividad, no usa tantos recursos, considere la posibilidad de reducir verticalmente desde el tamaño de proceso actual.

Si tiene un patrón de aplicación de SaaS o un escenario de consolidación de bases de datos, considere la posibilidad de usar un grupo elástico para optimizar los costos. Un grupo elástico es una excelente manera de lograr la consolidación de bases de datos y la optimización de costos. Para obtener más información sobre cómo administrar varias bases de datos mediante un grupo elástico, consulte: [administración de grupos y bases de datos](sql-database-elastic-pool-manage.md#azure-portal-manage-elastic-pools-and-pooled-databases).

### <a name="how-often-do-i-need-to-run-database-integrity-checks-for-my-database"></a>Con qué frecuencia es necesario ejecutar comprobaciones de integridad de base de datos en mi base de datos

SQL Database utiliza algunas técnicas inteligentes que permiten controlar ciertas clases de datos dañados automáticamente y sin pérdida de datos. Estas técnicas se han integrado en el servicio, que las usa cuando es necesario. De forma regular, se prueban las copias de seguridad de la base de datos en el servicio, para lo cual se restauran y se ejecuta DBCC CHECKDB en ellas. Si hay problemas, SQL Database los resuelve proactivamente. [Reparación de página automática](/sql/sql-server/failover-clusters/automatic-page-repair-availability-groups-database-mirroring) se usa para corregir páginas que están dañadas o que tienen problemas de integridad de datos. Las páginas de base de datos siempre se comprueban con el valor de SUMA DE COMPROBACIÓN predeterminado que comprueba la integridad de la página. SQL Database supervisa y revisa proactivamente la integridad de los datos de la base de datos y, si surgen problemas, los aborda con la prioridad más alta. Además, puede ejecutar sus propias comprobaciones de integridad como desee.  Para más información, consulte [Data Integrity in SQL Database](https://azure.microsoft.com/blog/data-integrity-in-azure-sql-database/) (Integridad de datos en SQL Database).

## <a name="data-movement-after-migration"></a>Movimiento de datos después de la migración

### <a name="how-do-i-export-and-import-data-as-bacpac-files-from-sql-database"></a>Cómo se exportan e importan datos como archivos BACPAC desde SQL Database

- **Exportar**: puede exportar su base de datos de Azure SQL como un archivo BACPAC desde Azure Portal.

   ![exportación de base de datos](./media/sql-database-export/database-export1.png)

- **Importar**: también puede importar datos como un archivo BACPAC en la base de datos mediante Azure Portal.

   ![importación de base de datos](./media/sql-database-import/import1.png)

### <a name="how-do-i-synchronize-data-between-sql-database-and-sql-server"></a>Cómo se pueden sincronizar los datos entre SQL Database y SQL Server

Hay varias maneras de conseguir este objetivo:

- **[Sincronización de datos](sql-database-sync-data.md)**: esta característica le ayuda a sincronizar los datos bidireccionalmente entre varias bases de datos de SQL Server locales y SQL Database. Para la sincronización con bases de datos de SQL Server locales, debe instalar y configurar el agente de sincronización en un equipo local y abrir el puerto TCP de salida 1433.
- **[Replicación de transacciones](https://azure.microsoft.com/blog/transactional-replication-to-azure-sql-database-is-now-generally-available/)**: con la replicación de transacciones puede sincronizar sus datos de la base de datos local a Azure SQL DB, donde la base de datos local es el anunciante y Azure SQL DB es el suscriptor. Por ahora, solo se admite esta configuración. Para obtener más información sobre cómo migrar sus datos de la base de datos local a Azure SQL con el tiempo de inactividad mínimo, consulte el tema sobre el [uso de la replicación de transacciones](sql-database-single-database-migrate.md#method-2-use-transactional-replication).

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre [SQL Database](sql-database-technical-overview.md).
