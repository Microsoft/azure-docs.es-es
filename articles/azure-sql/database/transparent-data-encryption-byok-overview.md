---
title: Cifrado de datos transparente (TDE) administrado por el cliente
description: Compatibilidad de Bring Your Own Key (BYOK) con el cifrado de datos transparente (TDE) con Azure Key Vault para SQL Database y Azure Synapse Analytics. Información general de TDE con BYOK, ventajas, funcionamiento, consideraciones y recomendaciones.
titleSuffix: Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.custom: seo-lt-2019, azure-synapse
ms.devlang: ''
ms.topic: conceptual
author: jaszymas
ms.author: jaszymas
ms.reviewer: vanto
ms.date: 03/18/2020
ms.openlocfilehash: cf0fec1f081a232abc88941e3dd785fb7617fb57
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87387122"
---
# <a name="azure-sql-transparent-data-encryption-with-customer-managed-key"></a>Cifrado de datos transparente de Azure SQL con una clave administrada por el cliente
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

El [Cifrado de datos transparente (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption) de Azure SQL con una clave administrada por el cliente habilita el escenario Bring Your Own Key (BYOK) para la protección de datos en reposo y permite a las organizaciones implementar la separación de tareas en la administración de claves y datos. Con el cifrado de datos transparente administrado por el cliente, el cliente es responsable y tiene el control total de la administración del ciclo de vida de una clave (creación, carga, rotación y eliminación), de los permisos de uso de claves y de la auditoría de operaciones con clave.

En este escenario, la clave que se usa para el cifrado de la clave de cifrado de base de datos (DEK), denominada protector de TDE, es una clave asimétrica administrada por el cliente que se almacena en una instancia propiedad del cliente y administrada por él de [Azure Key Vault (AKV)](../../key-vault/general/secure-your-key-vault.md), un sistema de administración de claves externas basado en la nube. Key Vault ofrece un almacenamiento seguro de alta disponibilidad y escalabilidad para claves criptográficas RSA respaldado opcionalmente por módulos de seguridad de hardware (HSM) con certificación FIPS 140-2 nivel 2. No permite el acceso directo a una clave almacenada, sino que proporciona servicios de cifrado y descifrado mediante el uso de la clave en las entidades autorizadas. La clave puede generarse desde el almacén de claves, así como importarse o [transferirse al almacén de claves desde un dispositivo HSM local](../../key-vault/keys/hsm-protected-keys.md).

En Azure SQL Database y Azure Synapse Analytics, el protector de TDE se establece en el nivel de servidor y todas las bases de datos cifradas asociadas a dicho servidor lo heredan. En el caso de la Instancia administrada de Azure SQL Database, el protector de TDE se establece en el nivel de instancia y lo heredan todas las bases de datos cifradas que se encuentren en dicha instancia. A menos que se indique lo contrario, a lo largo de este documento el término *servidor* hace referencia tanto a un servidor en SQL Database y Azure Synapse, como a una instancia administrada en Instancia administrada de SQL.

> [!IMPORTANT]
> Para aquellos que usan TDE administrado por el servicio y que quieren empezar a usar TDE administrado por el cliente, los datos permanecen cifrados durante el proceso de cambio y no hay tiempo de inactividad ni se vuelven a cifrar los archivos de la base de datos. El cambio de una clave administrada por el servicio a una clave administrada por el cliente solo requiere el nuevo cifrado de la DEK, que es una operación rápida y en línea.

> [!NOTE]
> Para proporcionar a los clientes de Azure SQL dos capas de cifrado de datos en reposo, se está implementando el cifrado de infraestructura (mediante el algoritmo de cifrado AES-256) con claves administradas por la plataforma. Esto proporciona una capa adicional de cifrado en reposo junto con TDE con claves administradas por el cliente, que ya está disponible. En este momento, los clientes deben solicitar el acceso a esta funcionalidad. Si está interesado en esta funcionalidad, póngase en contacto con AzureSQLDoubleEncryptionAtRest@service.microsoft.com.

## <a name="benefits-of-the-customer-managed-tde"></a>Ventajas del TDE administrado por el cliente

El Cifrado de datos transparente (TDE) administrado por el cliente le proporciona las siguientes ventajas:

- Control completo y pormenorizado del uso y de la administración del protector de TDE.

- Transparencia del uso del protector de TDE.

- Capacidad de implementar la separación de tareas para la administración de claves y datos en la organización.

- El administrador de Key Vault puede revocar los permisos de acceso de las claves para hacer que la base de datos cifrada sea inaccesible.

- Administración central de claves en AKV.

- Mayor confianza de los clientes finales, ya que AKV está diseñado para que Microsoft no pueda ver ni extraer las claves de cifrado.

## <a name="how-customer-managed-tde-works"></a>Funcionamiento del TDE administrado por el cliente

![Configuración y funcionamiento del TDE administrado por el cliente](./media/transparent-data-encryption-byok-overview/customer-managed-tde-with-roles.PNG)

Para que el servidor pueda usar el protector de TDE almacenado en AKV para cifrar la DEK, el administrador del almacén de claves debe conceder los siguientes derechos de acceso al servidor mediante su identidad única de Azure Active Directory (Azure AD):

- **get**: para recuperar la parte pública y las propiedades de la clave en Key Vault

- **wrapKey**: para poder proteger (cifrar) la DEK

- **unwrapKey**: para poder desproteger (descifrar) la DEK

El administrador de Key Vault también puede [habilitar el registro de eventos de auditoría del almacén de claves](../../azure-monitor/insights/key-vault-insights-overview.md), por lo que se pueden auditar más adelante.

Cuando se configura el servidor para usar un protector de TDE de AKV, el servidor envía la DEK de cada base de datos habilitada para TDE al almacén de claves para su cifrado. Key Vault devuelve la DEK cifrada, la cual se almacena en la base de datos del usuario.

Cuando es necesario, el servidor envía la DEK protegida al almacén de claves para que se descifre.

Los auditores pueden usar Azure Monitor para revisar los registros de los objetos AuditEvent del almacén de claves, si está habilitado el registro.

[!INCLUDE [sql-database-akv-permission-delay](../includes/sql-database-akv-permission-delay.md)]

## <a name="requirements-for-configuring-customer-managed-tde"></a>Requisitos de configuración de TDE administrado por el cliente

### <a name="requirements-for-configuring-akv"></a>Requisitos de configuración de AKV

- Key Vault y SQL Database (o una instancia administrada) deben pertenecer al mismo inquilino de Azure Active Directory. No se admiten las interacciones de servidor y almacén de claves entre inquilinos. Para mover los recursos después, se tendrá que volver a configurar el TDE con AKV. Obtenga más información sobre el [movimiento de recursos](../../azure-resource-manager/management/move-resource-group-and-subscription.md).

- La característica de [eliminación temporal](../../key-vault/general/soft-delete-overview.md) debe estar habilitada en el almacén de claves para protegerlo contra la pérdida de datos en caso de que se eliminen las claves (o el almacén de claves) de manera involuntaria. Los recursos eliminados temporalmente se conservan durante 90 días, a menos que el cliente los recupere o los purgue dentro de ese plazo. Las acciones de *recuperación* y *purga* tienen permisos propios asociados a una directiva de acceso al almacén de claves. La característica de eliminación temporal está desactivada de forma predeterminada y se puede habilitar mediante [PowerShell](../../key-vault/general/soft-delete-powershell.md#enabling-soft-delete) o la [CLI](../../key-vault/general/soft-delete-cli.md#enabling-soft-delete). No se puede habilitar mediante Azure Portal.  

- Conceda al servidor o a la instancia administrada acceso al almacén de claves (get, wrapKey o unwrapKey) con su identidad de Azure Active Directory. Cuando usa Azure Portal, la identidad de Azure AD se crea automáticamente. Al usar PowerShell o la CLI, se debe crear explícitamente la identidad de Azure AD y se debe comprobar la finalización. Consulte [Configuración de TDE con BYOK](transparent-data-encryption-byok-configure.md) y [Configuración de TDE con BYOK para SQL Managed Instance](../managed-instance/scripts/transparent-data-encryption-byok-powershell.md) para obtener instrucciones detalladas paso a paso cuando se usa PowerShell.

- Al usar un firewall con AKV, debe habilitar la opción *Permitir que los servicios de confianza de Microsoft omitan el firewall*.

### <a name="requirements-for-configuring-tde-protector"></a>Requisitos para configurar el protector de TDE

- El protector de TDE solo puede ser una clave asimétrica, RSA o RSA HSM. Las longitudes de clave admitidas son 2048 y 3072 bytes.

- La fecha de activación de la clave (si se establece) debe ser una fecha y hora del pasado. La fecha de expiración (si se establece) debe ser una fecha y hora del futuro.

- El estado de la clave debe ser *Habilitada*.

- Si va a importar una clave existente en el almacén de claves, asegúrese de proporcionarla en uno de los formatos de archivo compatibles (.pfx, .byok o .backup).

## <a name="recommendations-when-configuring-customer-managed-tde"></a>Recomendaciones para la configuración de TDE administrado por el cliente

### <a name="recommendations-when-configuring-akv"></a>Recomendaciones para la configuración de AKV

- Asocie como máximo un total de 500 bases de datos de uso general o 200 bases de datos críticas para la empresa mediante un almacén de claves y en una sola suscripción; de este modo, garantizará una alta disponibilidad cuando el servidor acceda al protector de TDE en el almacén de claves. Estas cifras se basan en la experiencia y están documentadas en los [límites de servicio para Azure Key Vault](../../key-vault/general/service-limits.md). El objetivo es evitar problemas después de la conmutación por error del servidor, ya que desencadenará tantas operaciones de clave en el almacén como bases de datos haya en ese servidor.

- Configure un bloqueo de recursos en el almacén de claves para controlar quién puede eliminar este recurso crítico y para evitar la eliminación accidental o no autorizada. Obtenga más información acerca de los [bloqueos de recursos](../../azure-resource-manager/management/lock-resources.md).

- Habilite la auditoría y la generación de informes en todas las claves de cifrado: El almacén de claves proporciona registros que son fáciles de insertar en otras herramientas de administración de eventos e información de seguridad. [Log Analytics](../../azure-monitor/insights/azure-key-vault.md) de Operations Management Suite es un ejemplo de un servicio que ya está integrado.

- Vincule cada servidor con dos almacenes de claves que residan en regiones diferentes y que conserven el mismo material de clave para garantizar una alta disponibilidad de bases de datos cifradas. Marque solo la clave del almacén de claves que está en la misma región que un protector de TDE. El sistema cambiará automáticamente al almacén de claves de la región remota si se produce una interrupción que afecte al almacén de claves en la misma región.

### <a name="recommendations-when-configuring-tde-protector"></a>Recomendaciones para la configuración del protector de TDE

- Almacene una copia del protector de TDE en un lugar seguro o en el servicio de custodia.

- Si la clave se genera en el almacén de claves, cree una copia de seguridad de esta antes de usarla en AKV por primera vez. La copia de seguridad solo se puede restaurar en Azure Key Vault. Obtenga más información sobre el comando [Backup-AzKeyVaultKey](/powershell/module/az.keyvault/backup-azkeyvaultkey).

- Cree una nueva copia de seguridad cada vez que se realicen cambios en la clave (por ejemplo, atributos de clave, etiquetas o ACL).

- **Mantenga versiones anteriores** de la clave en el almacén de claves al rotar las claves, de manera que se puedan restaurar las copias de seguridad de bases de datos más antiguas. Cuando se cambia el protector de TDE para una base de datos, las copias de seguridad antiguas de la base de datos **no se actualizan** para usar el protector de TDE más reciente. En el momento de la restauración, cada copia de seguridad necesita el protector de TDE con el que se cifró durante su creación. Las rotaciones de claves se pueden realizar según las instrucciones de [Rotate the Transparent Data Encryption Protector Using PowerShell](transparent-data-encryption-byok-key-rotation.md) (Rotación del protector de cifrado de datos transparente mediante PowerShell).

- Guarde todas las claves usadas previamente en AKV incluso después de cambiar a claves administradas por el servicio. Esto garantiza que las copias de seguridad de las bases de datos se puedan restaurar con los protectores de TDE almacenados en AKV.  Los protectores de TDE creados con Azure Key Vault deben conservarse hasta que se hayan creado todas las copias de seguridad almacenadas restantes con claves administradas por el servicio. Realice copias de seguridad recuperables de estas claves mediante [Backup-AzKeyVaultKey](/powershell/module/az.keyvault/backup-azkeyvaultkey).

- Para quitar una clave potencialmente en peligro durante un incidente de seguridad sin el riesgo de pérdida de datos, siga los pasos descritos en [Eliminación de una clave potencialmente en peligro](transparent-data-encryption-byok-remove-tde-protector.md).

## <a name="inaccessible-tde-protector"></a>Protector de TDE inaccesible

Cuando se configura el cifrado de datos transparente para usar una clave administrada por el cliente, se necesita acceso continuo al protector de TDE para que la base de datos permanezca en línea. Si el servidor pierde el acceso al protector de TDE administrado por el cliente en AKV, pasados máximo 10 minutos, una base de datos comenzará a denegar todas las conexiones con el mensaje de error correspondiente y cambiará su estado a *Inaccesible*. La única acción que se puede realizar en una base de datos con el estado Inaccesible es eliminarla.

> [!NOTE]
> Si no se puede acceder a la base de datos debido a una interrupción intermitente de la red, no se requiere ninguna acción y las bases de datos se volverán a conectar automáticamente.

Una vez restaurado el acceso a la clave, se necesita tiempo para volver a poner en línea la base de datos y se deben realizar pasos adicionales, los cuales pueden variar en función del tiempo transcurrido sin tener acceso a la clave y según el tamaño de los datos de la base de datos:

- Si se restaura el acceso a la clave en un plazo de ocho horas, la base de datos se restablecerá automáticamente durante la próxima hora.

- Si se restaura el acceso a la clave transcurridas más de ocho horas, no será posible la recuperación automática de la base de datos y será necesario realizar pasos adicionales en el portal para recuperarla manualmente. Esto que puede llevar una cantidad considerable de tiempo en función del tamaño de la base de datos. Una vez que la base de datos vuelva a estar en línea, **se perderán** los ajustes de nivel de servidor configurados previamente, como el [grupo de conmutación por error](auto-failover-group-overview.md), el historial de restauración a un momento dado y las etiquetas. Por lo tanto, se recomienda implementar un sistema de notificación que le permita identificar y resolver los problemas subyacentes de acceso de las claves en un plazo de ocho horas.

### <a name="accidental-tde-protector-access-revocation"></a>Revocación accidental del acceso al protector de TDE

Es posible que alguien con los derechos de acceso suficientes al almacén de claves deshabilite accidentalmente el acceso del servidor a la clave al realizar alguna de las siguientes acciones:

- Revocación de los permisos *get*, *wrapKey* o *unwrapKey* del almacén de claves desde el servidor.

- Eliminar la clave.

- Eliminar el almacén de claves.

- Cambio de las reglas de firewall del almacén de claves.

- Eliminar la identidad administrada del servidor en Azure Active Directory.

Obtenga más información sobre los [errores comunes que hacen que las bases de datos dejen de estar accesibles](/sql/relational-databases/security/encryption/troubleshoot-tde?view=azuresqldb-current#common-errors-causing-databases-to-become-inaccessible).

## <a name="monitoring-of-the-customer-managed-tde"></a>Supervisión del TDE administrado por el cliente

Para supervisar el estado de la base de datos y habilitar las alertas para la pérdida de acceso al protector de TDE, configure las siguientes características de Azure:

- [Azure Resource Health](../../service-health/resource-health-overview.md). Una base de datos inaccesible que haya perdido el acceso al protector de TDE aparecerá como "No disponible" después de que se haya denegado la primera conexión a la base de datos.
- [Registro de actividad](../../service-health/alerts-activity-log-service-notifications.md) cuando se produce un error de acceso al protector de TDE en el almacén de claves administrado por el cliente, las entradas se agregan al registro de actividad.  La creación de alertas para estos eventos le permitirá restablecer el acceso lo antes posible.
- Los [grupos de acciones](../../azure-monitor/platform/action-groups.md) se pueden definir para que envíen notificaciones y alertas en función de las preferencias, por ejemplo, correo electrónico/SMS/Inserción/Voz, aplicación lógica, webhook, ITSM o Runbook de Automation.

## <a name="database-backup-and-restore-with-customer-managed-tde"></a>Realización de copias de seguridad y restauración de bases de datos con TDE administrado por el cliente

Una vez que una base de datos se haya cifrado con TDE mediante una clave de Key Vault, las nuevas copias de seguridad generadas también se cifran con el mismo protector de TDE. Cuando se cambia el protector de TDE, las copias de seguridad antiguas de la base de datos **no se actualizan** para usar el protector de TDE más reciente.

Para restaurar una copia de seguridad cifrada con un protector de TDE de Key Vault, asegúrese de que el material de clave está disponible en el servidor de destino. Por lo tanto, se recomienda mantener todas las versiones antiguas del protector de TDE en el almacén de claves, de manera que se puedan restaurar las copias de seguridad de la base de datos.

> [!IMPORTANT]
> Nunca puede haber más de un protector de TDE establecido para un servidor. Es la clave marcada con "Hacer que la clave seleccionada sea el protector de TDE predeterminado" en la hoja de Azure Portal. Sin embargo, se pueden vincular varias claves adicionales a un servidor sin marcarlas como protector de TDE. Estas claves no se usan para proteger la DEK, pero se pueden usar durante la restauración desde una copia de seguridad si el archivo de copia de seguridad está cifrado con la clave con la huella digital correspondiente.

Si la clave necesaria para restaurar una copia de seguridad ya no está disponible para el servidor de destino, se devuelve el siguiente mensaje de error tras el intento de restauración: "Target server `<Servername>` does not have access to all AKV URIs created between \<Timestamp #1> and \<Timestamp #2>. Please retry operation after restoring all AKV URIs" (El servidor destino no tiene acceso a todos los URI de AKV creados entre <marca de tiempo #1> y <marca de tiempo #2>. Vuelva a intentar la operación después de restaurar a todos los URI de AKV).

Para mitigarlo, ejecute el cmdlet [Get-AzSqlServerKeyVaultKey](/powershell/module/az.sql/get-azsqlserverkeyvaultkey) para el servidor de destino o [Get-AzSqlInstanceKeyVaultKey](/powershell/module/az.sql/get-azsqlinstancekeyvaultkey) para la instancia administrada de destino para devolver la lista de claves disponibles e identificar las que faltan. Para asegurarse de que se pueden restaurar las copias de seguridad, asegúrese de que el servidor de destino para la restauración tiene acceso a todas las claves necesarias. No es necesario que estas claves estén marcadas como protector de TDE.

Para obtener más información sobre la recuperación de copia de seguridad de SQL Database, consulte [Recuperación de una base de datos de SQL Database](recovery-using-backups.md). Para más información sobre la recuperación de copia de seguridad del grupo de SQL, consulte [Recuperación de un grupo de SQL](../../synapse-analytics/sql-data-warehouse/backup-and-restore.md). Para realizar una copia de seguridad o una restauración nativa de SQL Server con Instancia administrada de SQL, consulte [Inicio rápido: Restauración de una base de datos en SQL Managed Instance](../managed-instance/restore-sample-database-quickstart.md)

Una consideración adicional para los archivos de registro: las copias de seguridad de archivos de registro permanecen cifradas con el protector de TDE original, incluso si este se ha rotado y la base de datos usa ahora un nuevo protector de TDE.  Durante la restauración, se necesitarán ambas claves para restaurar la base de datos.  Si el archivo de registro está usando un protector de TDE almacenado en Azure Key Vault, se necesitará esta clave durante la restauración, incluso si la base de datos se ha cambiado para usar TDE administrado por el servicio durante el proceso.

## <a name="high-availability-with-customer-managed-tde"></a>Alta disponibilidad con TDE administrado por el cliente

Incluso en los casos en los que no hay ninguna redundancia geográfica configurada para el servidor, se recomienda encarecidamente configurar el servidor para usar dos almacenes de claves distintos en dos regiones diferentes con el mismo material de clave. Esto puede realizarse mediante la creación de un protector de TDE que use el almacén de claves ubicado en la misma región que el servidor y la clonación de la clave en un almacén de claves de otra región de Azure, para que el servidor tenga acceso a un segundo almacén de claves por si el almacén de claves principal experimenta una interrupción mientras la base de datos se está ejecutando.

Use el cmdlet Backup-AzKeyVaultKey para recuperar la clave en formato cifrado desde el almacén de claves principal y, a continuación, use el cmdlet Restore-AzKeyVaultKey y especifique un almacén de claves en la segunda región para clonar la clave. También puede usar Azure Portal para hacer una copia de seguridad de la clave y restaurarla. La clave del almacén de claves secundario en la otra región no se debe marcar como protector de TDE, no está admitido.

Solo en el caso de que se produzca una interrupción en el almacén de claves principal, el sistema pasará automáticamente a la otra clave vinculada con la misma huella digital en el almacén de claves secundario, si existe. Tenga en cuenta que este cambio no se realizará si se han revocado los derechos de acceso y no se puede acceder al protector de TDE, o si se ha eliminado la clave o el almacén de claves, ya que esto podría indicar que el cliente quiere restringir el acceso del servidor a la clave de forma intencionada.

![Alta disponibilidad de un solo servidor](./media/transparent-data-encryption-byok-overview/customer-managed-tde-with-ha.png)

## <a name="geo-dr-and-customer-managed-tde"></a>Recuperación ante desastres con localización geográfica y TDE administrado por el cliente

En los escenarios de [replicación geográfica activa](active-geo-replication-overview.md) y de [grupos de conmutación por error](auto-failover-group-overview.md), cada servidor implicado requiere un almacén de claves independiente que debe colocarse junto con el servidor en la misma región de Azure. El cliente es el responsable de mantener la coherencia del material de clave en los almacenes de claves para que la base de datos secundaria con replicación geográfica esté sincronizada y pueda asumir el uso de la misma clave desde su almacén de claves local, en caso de que la base de datos principal deje de estar accesible por una interrupción en la región y se desencadene una conmutación por error. Se pueden configurar hasta cuatro bases de datos secundarias y no se admite el encadenamiento (bases de datos secundarias de otras bases de datos secundarias).

Para evitar problemas durante la replicación geográfica debido a la falta de material de clave, es importante seguir estas reglas a la hora de configurar el TDE administrado por el cliente:

- Todos los almacenes de claves implicados deben tener las mismas propiedades y los mismos derechos de acceso para los servidores respectivos.

- Todos los almacenes de claves implicados deben contener el mismo material de clave. Esto no solo se aplica al protector de TDE actual, sino a todos los protectores de TDE anteriores que se pueden usar en los archivos de copia de seguridad.

- Tanto la configuración inicial como la rotación del protector de TDE se deben realizar primero en el servidor secundario y, a continuación, en el principal.

![Grupos de conmutación por error y recuperación ante desastres con localización geográfica](./media/transparent-data-encryption-byok-overview/customer-managed-tde-with-bcdr.png)

Para probar una conmutación por error, siga los pasos descritos en la [información general sobre la replicación geográfica activa](active-geo-replication-overview.md). La prueba de la conmutación por error se debe realizar de forma periódica para validar que SQL Database ha mantenido el permiso de acceso a ambos almacenes de claves.

## <a name="next-steps"></a>Pasos siguientes

También puede consultar los siguientes scripts de ejemplo de PowerShell para las operaciones comunes con TDE administrado por el cliente:

- [Rotación del protector de Cifrado de datos transparente para SQL Database mediante PowerShell](transparent-data-encryption-byok-key-rotation.md)

- [Eliminación de un protector de Cifrado de datos transparente (TDE) para SQL Database mediante PowerShell](transparent-data-encryption-byok-remove-tde-protector.md)

- [Administración del Cifrado de datos transparente en SQL Managed Instance con su propia clave mediante PowerShell](../managed-instance/scripts/transparent-data-encryption-byok-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)
