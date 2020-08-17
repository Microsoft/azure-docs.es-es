---
title: Cifrado de datos transparente
titleSuffix: Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics
description: Información general sobre el cifrado de datos transparente para Azure SQL Database, Azure SQL Managed Instance y Azure Synapse Analytics. Este documento explica sus ventajas y las opciones de configuración, que incluye cifrado de datos transparente administrado por el servicio y Bring Your Own Key.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: seo-lt-2019 sqldbrb=3
ms.devlang: ''
ms.topic: conceptual
author: jaszymas
ms.author: jaszymas
ms.reviewer: vanto
ms.date: 06/15/2020
ms.openlocfilehash: d9bc5e91d45b75c47cee31c45b937f7d3f0118b8
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836690"
---
# <a name="transparent-data-encryption-for-sql-database-sql-managed-instance-and-azure-synapse-analytics"></a>Cifrado de datos transparente para SQL Database, SQL Managed Instance y Azure Synapse Analytics
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

El [cifrado de datos transparente (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption) ayuda a proteger Azure SQL Database, Azure SQL Managed Instance y Azure Synapse Analytics frente a la amenaza de actividad malintencionada sin conexión, ya que cifra los datos en reposo. También realiza cifrado y descifrado de la base de datos en tiempo real, copias de seguridad asociadas y archivos de registro de transacciones en reposo sin necesidad de efectuar cambios en la aplicación. El TDE se habilita de manera predeterminada para todas las instancias de SQL Database recién implementadas, y es preciso habilitarlo manualmente para las bases de datos anteriores de Azure SQL Database y Azure SQL Managed Instance. TDE debe habilitarse manualmente para Azure Synapse Analytics.

El TDE efectúa el cifrado y descifrado de E/S en tiempo real de los datos en el nivel de página. Todas las páginas se descifran cuando se leen en la memoria y, a continuación, se cifran antes de escribirse en el disco. El TDE cifra el almacenamiento de una base de datos completa mediante una clave simétrica denominada clave de cifrado de base de datos (DEK). Al iniciarse la base de datos, la DEK cifrada se descifra y luego se usa para descifrar y volver a cifrar los archivos de base de datos en el proceso del motor de base de datos de SQL Server. A la clave de cifrado se le aplica el protector de TDE. El protector de TDE es un certificado administrado por el servicio (cifrado de datos transparentes administrado por el servicio) o una clave asimétrica almacenada en [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault) (cifrado de datos transparentes administrado por el cliente).

En el caso de Azure SQL Database y Azure Synapse, el protector del TDE se establece en el nivel de [servidor](logical-servers.md) y lo heredan todas las bases de datos asociadas a ese servidor. En el caso de la Instancia administrada de Azure SQL Database, el protector de TDE se establece en el nivel de instancia y lo heredan todas las bases de datos cifradas que se encuentren en dicha instancia. El término *servidor* hace referencia tanto a servidor como a instancia a lo largo de este documento, a menos que se indique lo contrario.

> [!IMPORTANT]
> Todas las bases de datos recién creadas en SQL Database se cifran de forma predeterminada mediante el cifrado de datos transparente administrado por el servicio. Las bases de datos SQL existentes creadas antes de mayo de 2017 y las bases de datos SQL creadas mediante restauración, replicación geográfica y copia de base de datos no se cifran de forma predeterminada. Las bases de datos de SQL Managed Instance existentes que se crearon antes de febrero de 2019 no se cifran de forma predeterminada. Las bases de datos de SQL Managed Instance que se crearon mediante restauración heredan el estado de cifrado del origen.

> [!NOTE]
> TDE no se puede usar para cifrar la base de datos **maestra** en SQL Database.  La base de datos **maestra** contiene objetos que son necesarios para realizar las operaciones de TDE en las bases de datos de usuario.

## <a name="service-managed-transparent-data-encryption"></a>Cifrado de datos transparente administrado por el servicio

En Azure, la configuración predeterminada de TDE es que la clave de cifrado está protegida mediante un certificado de servidor integrado. El certificado de servidor integrado es único para cada servidor y el algoritmo de cifrado que se usa es AES 256. Si una base de datos está en una relación de replicación geográfica, tanto la base de datos principal como la secundaria con replicación geográfica están protegidas por la clave de servidor principal de la base de datos principal. Si hay dos bases de datos conectadas al mismo servidor, también comparten el mismo certificado integrado. Microsoft rota automáticamente estos certificados en cumplimiento de la directiva de seguridad interna y se protege la clave raíz mediante un almacén secreto interno de Microsoft. Los clientes pueden verificar el cumplimiento de SQL Database e Instancia administrada de SQL con las directivas de seguridad internas en los informes de auditoría de terceros independientes disponibles en el [Centro de confianza de Microsoft](https://servicetrust.microsoft.com/).

Microsoft también mueve y administra con total fluidez las claves según sea necesario para la replicación geográfica y las restauraciones.

## <a name="customer-managed-transparent-data-encryption---bring-your-own-key"></a>Cifrado de datos transparente administrado por el cliente (Bring Your Own Key)

La TDE administrada por el cliente también se conoce como compatibilidad de Bring Your Own Key (BYOK) con TDE. En este escenario, el protector de TDE que cifra la clave de cifrado es una clave asimétrica administrada por el cliente, que se almacena en una instancia de Azure Key Vault que es propiedad del cliente y que este administra (un sistema de administración de claves externas basado en la nube de Azure), y que nunca sale del almacén de claves. El protector del TDE lo puede [generar el almacén de claves, o bien se puede transferir a él](https://docs.microsoft.com/azure/key-vault/key-vault-hsm-protected-keys) desde un dispositivo del módulo de seguridad de hardware (HSM) local. SQL Database, Instancia administrada de SQL y Azure Synapse deben tener permisos para el almacén de claves propiedad del cliente para descifrar y cifrar el DEK. Si se revocan los permisos del servidor para el almacén de claves, no se podrá acceder a las bases de datos y se cifrarán todos los datos.

Gracias al cifrado de datos transparente con integración de Azure Key Vault, los usuarios pueden controlar las tareas de administración de claves, entre las que se incluyen las rotaciones de claves, los permisos del almacén de claves y la copia de seguridad de claves, así como la opción de llevar a cabo auditorías o crear informes sobre todos los protectores de TDE mediante la funcionalidad de Azure Key Vault. Key Vault ofrece una administración centralizada de claves, aprovecha los módulos de seguridad de hardware, a los que se les supervisa intensamente, y permite la separación de obligaciones entre la administración de las claves y de los datos, lo que ayudar a cumplir las directivas de seguridad.
Para más información sobre BYOK para Azure SQL Database y Azure Synapse, consulte el artículo acerca del [cifrado de datos transparente con la integración de Azure Key Vault](transparent-data-encryption-byok-overview.md).

Para empezar a usar el cifrado de datos transparente con integración de Azure Key Vault, consulte la guía paso a paso en la que se explica cómo [habilitar el cifrado de datos transparente mediante su propia clave desde Azure Key Vault](transparent-data-encryption-byok-configure.md).

## <a name="move-a-transparent-data-encryption-protected-database"></a>Traslado de una base de datos protegida por cifrado de datos transparente

No es necesario descifrar las bases de datos para las operaciones dentro de Azure. La configuración de TDE en la base de datos de origen o la base de datos principal se hereda de forma transparente en el destino. Entre las operaciones incluidas están las siguientes:

- Geo-restore
- Restauración a un momento específico de autoservicio
- Restauración de una base de datos eliminada
- Replicación geográfica activa
- Creación de una copia de base de datos
- Restauración de archivo de copia de seguridad a la Instancia administrada de Azure SQL Database

> [!IMPORTANT]
> En Azure SQL Managed Instance no se permite realizar copias de seguridad manuales de una base de datos cifrada por TDE administrada por el servicio, ya que el certificado que se usa para el cifrado no es accesible. Use la característica de restauración de punto en el tiempo para mover este tipo de base de datos a otra instancia de SQL Managed Instance o cambie a una clave administrada por el cliente.

Si exporta una base de datos protegida mediante TDE, el contenido exportado de la base de datos no se cifra. Este contenido exportado se almacena en archivos BACPAC no cifrados. Asegúrese de proteger adecuadamente los archivos BACPAC y de habilitar el TDE cuando haya finalizado la importación de la nueva base de datos.

Por ejemplo, si el archivo BACPAC se exporta desde una instancia de SQL Server, el contenido importado de la nueva base de datos no se cifra automáticamente. Del mismo modo, si el archivo BACPAC se importa a una instancia de SQL Server, la base de datos nueva tampoco se cifra automáticamente.

La única excepción es cuando se exporta una base de datos con una instancia de SQL Database como origen y destino. El TDE se habilita en la nueva base de datos, pero el propio archivo BACPAC sigue sin estar cifrado.

## <a name="manage-transparent-data-encryption"></a>Administración del cifrado de datos transparente

# <a name="the-azure-portal"></a>[Portal de Azure](#tab/azure-portal)

Administre TDE en Azure Portal.

Para configurar el TDE desde Azure Portal, es preciso estar conectado como propietario de Azure, colaborador o administrador de seguridad de SQL.

Habilite y deshabilite el TDE en el nivel de base de datos. En el caso de Instancia administrada de Azure SQL Database, use Transact-SQL (T-SQL) para activar y desactivar el TDE en las bases de datos. Para Azure SQL Database y Azure Synapse, puede administrar TDE para la base de datos en [Azure Portal](https://portal.azure.com) después de haber iniciado sesión con la cuenta de administrador o colaborador de Azure. Busque la configuración del TDE en la base de datos de usuario. De forma predeterminada, se usa el cifrado de datos transparente administrado por el servicio. Se genera automáticamente un certificado de cifrado de datos transparente para el servidor que contiene la base de datos.

![Cifrado de datos transparente administrado por el servicio](./media/transparent-data-encryption-tde-overview/service-managed-transparent-data-encryption.png)  

Establezca la clave maestra de TDE, conocida como protector del TDE, en el nivel de instancia o de servidor. Para usar el cifrado de datos transparente con compatibilidad con Bring Your Own Key y proteger las bases de datos con una clave de Key Vault, abra la configuración de TDE en su servidor.

![Cifrado de datos transparente con compatibilidad con Bring Your Own Key](./media/transparent-data-encryption-tde-overview/tde-byok-support.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Administre TDE mediante PowerShell.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> El módulo de Azure Resource Manager para PowerShell todavía es compatible, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. Para estos cmdlets, consulte [AzureRM.Sql](https://docs.microsoft.com/powershell/module/AzureRM.Sql/). Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos.

Para configurar el TDE mediante PowerShell, debe estar conectado como propietario de Azure, colaborador o administrador de seguridad de SQL.

### <a name="cmdlets-for-azure-sql-database-and-azure-synapse"></a>Cmdlets para Azure SQL Database y Azure Synapse

Use los siguientes cmdlets para Azure SQL Database y Azure Synapse:

| Cmdlet | Descripción |
| --- | --- |
| [Set-AzSqlDatabaseTransparentDataEncryption](https://docs.microsoft.com/powershell/module/az.sql/set-azsqldatabasetransparentdataencryption) |Habilita o deshabilita el cifrado de datos transparente para una base de datos.|
| [Get-AzSqlDatabaseTransparentDataEncryption](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldatabasetransparentdataencryption) |Obtiene el estado del cifrado de datos transparente para una base de datos. |
| [Get-AzSqlDatabaseTransparentDataEncryptionActivity](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldatabasetransparentdataencryptionactivity) |Comprueba el progreso de cifrado de una base de datos. |
| [Add-AzSqlServerKeyVaultKey](https://docs.microsoft.com/powershell/module/az.sql/add-azsqlserverkeyvaultkey) |Agrega una clave de Key Vault a un servidor Server. |
| [Get-AzSqlServerKeyVaultKey](https://docs.microsoft.com/powershell/module/az.sql/get-azsqlserverkeyvaultkey) |Obtiene las claves de Key Vault para un servidor.  |
| [Set-AzSqlServerTransparentDataEncryptionProtector](https://docs.microsoft.com/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector) |Establece el protector de cifrado de datos transparente para un servidor. |
| [Get-AzSqlServerTransparentDataEncryptionProtector](https://docs.microsoft.com/powershell/module/az.sql/get-azsqlservertransparentdataencryptionprotector) |Obtiene el protector de cifrado de datos transparente |
| [Remove-AzSqlServerKeyVaultKey](https://docs.microsoft.com/powershell/module/az.sql/remove-azsqlserverkeyvaultkey) |Quita una clave de Key Vault de un servidor. |
|  | |

> [!IMPORTANT]
> En el caso de Instancia administrada de SQL Database, use el comando [ALTER DATABASE](/sql/t-sql/statements/alter-database-azure-sql-database) de T-SQL para activar y desactivar el cifrado de datos transparente en el nivel de base de datos y consulte el [script de PowerShell de ejemplo](transparent-data-encryption-byok-configure.md) para administrar el cifrado de datos transparente en el nivel de instancia.

# <a name="transact-sql"></a>[Transact-SQL](#tab/azure-TransactSQL)

Administre TDE mediante Transact-SQL.

Conéctese a la base de datos mediante un inicio de sesión que sea un administrador o un miembro del rol **dbmanager** en la base de datos maestra.

| Get-Help | Descripción |
| --- | --- |
| [ALTER DATABASE (Azure SQL Database)](/sql/t-sql/statements/alter-database-azure-sql-database) | SET ENCRYPTION ON/OFF cifra o descifra una base de datos |
| [sys.dm_database_encryption_keys](/sql/relational-databases/system-dynamic-management-views/sys-dm-database-encryption-keys-transact-sql) |Devuelve información sobre el estado de cifrado de una base de datos y sus claves de cifrado de base de datos asociadas |
| [sys.dm_pdw_nodes_database_encryption_keys](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-nodes-database-encryption-keys-transact-sql) |Devuelve información sobre el estado de cifrado de cada nodo de Azure Synapse y sus claves de cifrado de base de datos asociadas |
|  | |

No se puede cambiar el protector de TDE para una clave de Key Vault mediante Transact-SQL. Use PowerShell o Azure Portal.

# <a name="rest-api"></a>[REST API](#tab/azure-RESTAPI)

Administre TDE mediante API REST.

Para configurar el cifrado de datos transparente mediante la API REST, debe estar conectado como propietario de Azure, colaborador o administrador de seguridad de SQL.
Use el siguiente conjunto de comandos para Azure SQL Database y Azure Synapse:

| Get-Help | Descripción |
| --- | --- |
|[Create Or Update Server](https://docs.microsoft.com/rest/api/sql/servers/createorupdate)|Agrega una identidad de Azure Active Directory a un servidor (se usa para conceder acceso a Key Vault).|
|[Create Or Update Server Key](https://docs.microsoft.com/rest/api/sql/serverkeys/createorupdate)|Agrega una clave de Key Vault a un servidor Server.|
|[Delete Server Key](https://docs.microsoft.com/rest/api/sql/serverkeys/delete)|Quita una clave de Key Vault de un servidor. |
|[Get Server Keys](https://docs.microsoft.com/rest/api/sql/serverkeys/get)|Obtiene una clave específica de Key Vault de un servidor.|
|[List Server Keys By Server](https://docs.microsoft.com/rest/api/sql/serverkeys/listbyserver)|Obtiene las claves de Key Vault para un servidor. |
|[Create Or Update Encryption Protector](https://docs.microsoft.com/rest/api/sql/encryptionprotectors/createorupdate)|Establece el protector del TDE para un servidor.|
|[Get Encryption Protector](https://docs.microsoft.com/rest/api/sql/encryptionprotectors/get)|Obtiene el protector del TDE para un servidor.|
|[List Encryption Protectors By Server](https://docs.microsoft.com/rest/api/sql/encryptionprotectors/listbyserver)|Obtiene los protectores del TDE para un servidor. |
|[Create Or Update Transparent Data Encryption Configuration](https://docs.microsoft.com/rest/api/sql/transparentdataencryptions/createorupdate)|Habilita o deshabilita el TDE para una base de datos.|
|[Get Transparent Data Encryption Configuration](https://docs.microsoft.com/rest/api/sql/transparentdataencryptions/get)|Obtiene la configuración del TDE para una base de datos.|
|[List Transparent Data Encryption Configuration Results](https://docs.microsoft.com/rest/api/sql/transparentdataencryptionactivities/listbyconfiguration)|Obtiene el resultado de cifrado para una base de datos.|

## <a name="see-also"></a>Consulte también

- Las instancias de SQL Server que se ejecutan en una máquina virtual de Azure también pueden usar una clave asimétrica desde Key Vault. Los pasos de configuración son diferentes del uso de una clave asimétrica en SQL Database y en la Instancia administrada de SQL Database. Para obtener más información, consulte [Administración extensible de claves con Azure Key Vault (SQL Server)](/sql/relational-databases/security/encryption/extensible-key-management-using-azure-key-vault-sql-server).
- Para ver una descripción general de TDE, consulte [Cifrado de datos transparente](/sql/relational-databases/security/encryption/transparent-data-encryption).
- Para más información sobre el cifrado de datos transparente con compatibilidad de Bring Your Own Key con Azure SQL Database, Instancia administrada de Azure SQL y Azure Synapse, consulte [Cifrado de datos transparente con compatibilidad con Bring Your Own Key](transparent-data-encryption-byok-overview.md).
- Para empezar a usar el cifrado de datos transparente con compatibilidad con Bring Your Own Key, consulte la guía paso a paso en la que se explica cómo [habilitar el cifrado de datos transparente mediante su propia clave desde Azure Key Vault](transparent-data-encryption-byok-configure.md).
- Para más información sobre Key Vault, consulte [Protección del acceso a un almacén de claves](https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault).
