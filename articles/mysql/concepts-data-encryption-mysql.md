---
title: 'Cifrado de datos con claves administradas por el cliente: Azure Database for MySQL'
description: El cifrado de datos de Azure Database for MySQL con una clave administrada por el cliente le permite usar el método Bring Your Own Key (BYOK) para la protección de datos en reposo. También permite a las organizaciones implementar la separación de tareas en la administración de claves y datos.
author: kummanish
ms.author: manishku
ms.service: mysql
ms.topic: conceptual
ms.date: 01/13/2020
ms.openlocfilehash: 8fca0195c2941e4ed1a859c3201adfc2a4a0a2ed
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88067450"
---
# <a name="azure-database-for-mysql-data-encryption-with-a-customer-managed-key"></a>Cifrado de datos de Azure Database for MySQL con una clave administrada por el cliente

El cifrado de datos con claves administradas por el cliente para Azure Database for MySQL le permite traer su propia clave (BYOK) para la protección de datos en reposo. También permite a las organizaciones implementar la separación de tareas en la administración de claves y datos. Con el cifrado administrado por el cliente, el usuario es responsable y tiene el control total del ciclo de vida de una clave, los permisos de uso de la clave y la auditoría de operaciones con claves.

El cifrado de datos con claves administradas por el cliente para Azure Database for MySQL se establece en el nivel de servidor. En un servidor determinado, se usa una clave administrada por el cliente, denominada clave de cifrado de claves (KEK), para cifrar la clave de cifrado de datos (DEK) que usa el servicio. KEK es una clave asimétrica almacenada en una instancia de [Azure Key Vault](../key-vault/key-Vault-secure-your-key-Vault.md) administrada por el cliente y de su propiedad. La clave de cifrado de claves (KEK) y la clave de cifrado de datos (DEK) se describen con más detalle más adelante en este artículo.

Key Vault es un sistema de administración de claves externas basado en la nube. Tiene una gran disponibilidad y ofrece almacenamiento seguro escalable para claves criptográficas RSA respaldado opcionalmente por módulos de seguridad de hardware (HSM) con certificación FIPS 140-2 nivel 2. No permite acceso directo a una clave almacenada, pero proporciona servicios de cifrado y descifrado a entidades autorizadas. Key Vault puede generar la clave, importarla o [hacer que se transfiera desde un dispositivo HSM local](../key-vault/key-Vault-hsm-protected-keys.md).

> [!NOTE]
> Esta característica está disponible en todas las regiones de Azure donde Azure Database for MySQL admita los planes de tarifa "Uso general" y "Optimizado para memoria". Para otras limitaciones, consulte la sección [Limitaciones](concepts-data-encryption-mysql.md#limitations).

## <a name="benefits"></a>Ventajas

El cifrado de datos con claves administradas por el cliente para Azure Database for MySQL proporciona las siguientes ventajas:

* El acceso a los datos está totalmente controlado por el usuario gracias a la posibilidad de quitar la clave y hacer que la base de datos sea inaccesible 
* Control total sobre el ciclo de vida de la clave, incluida la rotación de esta para cumplir con las directivas corporativas
* Organización y administración central de las claves en Azure Key Vault.
* Posibilidad de implementar la separación de tareas entre los responsables de seguridad y los administradores del sistema y de bases de datos


## <a name="terminology-and-description"></a>Terminología y descripción

**Clave de cifrado de datos (DEK)** : una clave simétrica AES256 que se emplea para cifrar una partición o un bloque de datos. Cifrar cada bloque de datos con una clave diferente dificulta los ataques de análisis criptográficos. Se necesita acceso a las DEK por la instancia de proveedor o aplicación de recursos que cifra y descifra un bloque específico. Cuando se reemplaza una DEK por una nueva clave, solo se deben volver a cifrar los datos de su bloque asociado con la nueva clave.

**Clave de cifrado de claves (KEK)** : una clave de cifrado usada para cifrar las DEK. Una KEK que nunca abandona Key Vault permite el cifrado y control de las DEK. La entidad que tiene acceso a la KEK puede ser diferente de aquella que necesita la DEK. Puesto que la KEK es necesaria para descrifrar la DEK, la KEK es de manera eficaz un punto único por el que se pueden eliminar de forma eficaz las DEK mediante la eliminación de la KEK.

Las DEK, cifradas con las KEK, se almacenan por separado. Solo una entidad con acceso a la KEK puede descifrar estas DEK. Para más información, consulte [Seguridad del cifrado en reposo](../security/fundamentals/encryption-atrest.md).

## <a name="how-data-encryption-with-a-customer-managed-key-work"></a>Funcionamiento del cifrado de datos con una clave administrada por el cliente

![Diagrama que muestra información general de Bring Your Own Key](media/concepts-data-access-and-security-data-encryption/mysqloverview.png)

Para que un servidor de MySQL pueda usar claves administradas por el cliente almacenadas en Key Vault para el cifrado de la DEK, un administrador de Key Vault debe conceder los siguientes derechos de acceso al servidor:

* **get**: para recuperar la parte pública y las propiedades de la clave del almacén de claves.
* **wrapKey**: para poder cifrar la DEK. La DEK cifrada se almacena en Azure Database for MySQL.
* **unwrapKey**: para poder descifrar la DEK. Azure Database for MySQL necesita la DEK descifrada para cifrar o descifrar los datos.

El administrador del almacén de claves también puede [habilitar el registro de eventos de auditoría de Key Vault](../azure-monitor/insights/key-vault-insights-overview.md), de forma que se puedan auditar más adelante.

Cuando el servidor está configurado para usar la clave administrada por el cliente que se almacena en el almacén de claves, el servidor envía a este la DEK para que la cifre. Key Vault devuelve las DEK cifradas, que se almacenan en la base de datos del usuario. De igual modo, cuando es necesario, el servidor envía la DEK protegida al almacén de claves para que la descifre. Los auditores pueden usar Azure Monitor para revisar los registros de eventos de auditoría de Key Vault, si está habilitado el registro.

## <a name="requirements-for-configuring-data-encryption-for-azure-database-for-mysql"></a>Requisitos de configuración del cifrado de datos para Azure Database for MySQL

A continuación se indican los requisitos para configurar Key Vault:

* Key Vault y Azure Database for MySQL deben pertenecer al mismo inquilino de Azure Active Directory (Azure AD). No se admiten las interacciones de servidor y Key Vault entre inquilinos. Para mover los recursos de Key Vault posteriormente, es necesario volver a configurar el cifrado de datos.
* Habilita la característica de eliminación temporal en el almacén de claves para protegerse frente a la pérdida de datos en caso de una eliminación accidental de claves (o de Key Vault). Los recursos eliminados temporalmente se conservan durante 90 días, a menos que el usuario los recupere o los purgue mientras tanto. Las acciones de recuperación y purga tienen permisos propios asociados a una directiva de acceso a Key Vault. La característica de eliminación temporal está desactivada de forma predeterminada, pero puede habilitarla mediante PowerShell o la CLI de Azure (tenga en cuenta que no puede habilitarla mediante Azure Portal).
* Conceda a Azure Database for MySQL acceso al almacén de claves con los permisos get, wrapKey y unwrapKey mediante su identidad administrada única. En Azure Portal, la identidad única "Servicio" se crea automáticamente cuando se habilita el cifrado de datos en MySQL. Consulte [Configuración del cifrado de datos para MySQL](howto-data-encryption-portal.md) para obtener instrucciones paso a paso al usar Azure Portal.

A continuación se indican los requisitos para configurar la clave administrada por el cliente:

* La clave administrada por el cliente que se va a usar para cifrar las DEK solo puede ser asimétrica, RSA 2048.
* La fecha de activación de la clave (si se establece) debe ser una fecha y hora del pasado. La fecha de expiración (si se establece) debe ser una fecha y hora del futuro.
* El estado de la clave debe ser *Habilitada*.
* Si va a [importar una clave existente](https://docs.microsoft.com/rest/api/keyvault/ImportKey/ImportKey) en el almacén de claves, asegúrese de proporcionarla en uno de los formatos de archivo compatibles (`.pfx`, `.byok`, `.backup`).

## <a name="recommendations"></a>Recomendaciones

Cuando vaya a usar el cifrado de datos mediante una clave administrada por el cliente, tenga en cuenta esta recomendaciones para configurar Key Vault:

* Establezca un bloqueo de recursos en Key Vault para controlar quién puede eliminar este recurso crítico e impedir la eliminación accidental o no autorizada.
* Habilite la auditoría y la generación de informes en todas las claves de cifrado. Key Vault proporciona registros que son fáciles de insertar en otras herramientas de administración de eventos e información de seguridad. Log Analytics de Azure Monitor es un ejemplo de un servicio que ya está integrado.
* Asegúrese de que Key Vault y Azure Database for MySQL residen en la misma región, a fin de garantizar un acceso más rápido para las operaciones de encapsulado y desencapsulado de DEK.
* Bloquee la KeyVault de Azure solo para **punto de conexión privado y las redes seleccionadas** y permita solamente *servicios de Microsoft* de confianza para proteger los recursos.

    ![servicio de confianza con AKV](media/concepts-data-access-and-security-data-encryption/keyvault-trusted-service.png)

A continuación se ofrecen recomendaciones para configurar una clave administrada por el cliente:

* Almacene una copia de la clave administrada por el cliente en un lugar seguro o en el servicio de custodia.

* Si Key Vault genera la clave, cree una copia de seguridad de ella antes de usarla por primera vez. Solo se puede restaurar la copia de seguridad en Key Vault. Para más información sobre del comando backup, consulte [Backup-AzKeyVaultKey](https://docs.microsoft.com/powershell/module/az.keyVault/backup-azkeyVaultkey).

## <a name="inaccessible-customer-managed-key-condition"></a>Condición de clave administrada por el cliente inaccesible

Cuando se configura el cifrado de datos con una clave administrada por el cliente en Key Vault, es necesario el acceso continuo a esta clave para que el servidor permanezca en línea. Si el servidor pierde el acceso a la clave administrada por el cliente en Key Vault, comienza a denegar todas las conexiones al cabo de 10 minutos. El servidor emite un mensaje de error correspondiente y cambia su estado a *Inaccesible*. Algunas de las razones por las que el servidor puede alcanzar este estado son:

* Si creamos un servidor de restauración a un momento dado para el Azure Database for MySQL, que tiene habilitado el cifrado de datos, el servidor recién creado estará en estado de *inaccesible*. Puede solucionar este paso a través de [Azure Portal](howto-data-encryption-portal.md#using-data-encryption-for-restore-or-replica-servers) o [CLI](howto-data-encryption-cli.md#using-data-encryption-for-restore-or-replica-servers).
* Si creamos una réplica de lectura para el Azure Database for MySQL, que tiene habilitado el cifrado de datos, el servidor de réplica estará en estado de *inaccesible*. Puede solucionar este paso a través de [Azure Portal](howto-data-encryption-portal.md#using-data-encryption-for-restore-or-replica-servers) o [CLI](howto-data-encryption-cli.md#using-data-encryption-for-restore-or-replica-servers).
* Si elimina el KeyVault, el Azure Database for MySQL no podrá tener acceso a la clave y se moverá a estado de *inaccesible*. Recupere el [Key Vault](../key-vault/general/soft-delete-cli.md#deleting-and-purging-key-vault-objects) y vuelva a validar el cifrado de datos para que el servidor esté *disponible*.
* Si elimina la clave del KeyVault, el Azure Database for MySQL no podrá tener acceso a la clave y se moverá a estado de *inaccesible*. Recupere la [clave](../key-vault/general/soft-delete-cli.md#deleting-and-purging-key-vault-objects) y vuelva a validar el cifrado de datos para que el servidor esté *disponible*.
* Si expira la clave almacenada en Azure KeyVault, la clave dejará de ser válida y el Azure Database for MySQL pasará a estado de *inaccesible*. Extienda la fecha de expiración de la clave mediante [CLI](https://docs.microsoft.com/cli/azure/keyvault/key?view=azure-cli-latest#az-keyvault-key-set-attributes) y, después, vuelva a validar el cifrado de datos para que el servidor esté *disponible*.

### <a name="accidental-key-access-revocation-from-key-vault"></a>Revocación accidental del acceso a la clave de Key Vault

Podría suceder que alguien con derechos de acceso suficientes a Key Vault deshabilitara por accidente el acceso del servidor a la clave de la siguiente forma:

* Revocación de los permisos get, wrapKey o unwrapKey de Key Vault desde el servidor.
* Eliminación de la clave.
* Eliminación del almacén de claves.
* Cambio de las reglas de firewall del almacén de claves.
* Eliminación de la identidad administrada del servidor en Azure AD.

## <a name="monitor-the-customer-managed-key-in-key-vault"></a>Supervisión de la clave administrada por el cliente en Key Vault

Para supervisar el estado de la base de datos y para habilitar las alertas cuando se produce la pérdida de acceso transparente al protector de cifrado de datos, configure las siguientes características de Azure:

* [Azure Resource Health](../service-health/resource-health-overview.md): una base de datos inaccesible que haya perdido acceso a la clave del cliente aparecerá como "Inaccesible" después de que se haya denegado la primera conexión a la base de datos.
* [Registro de actividades](../service-health/alerts-activity-log-service-notifications.md): cuando se produce un error de acceso a la clave de cliente en Key Vault administrado por el cliente, se agregan entradas al registro de actividad. Puede restablecer el acceso tan pronto como sea posible si crea alertas para estos eventos.

* [Grupos de acciones](../azure-monitor/platform/action-groups.md): Defina estos grupos para enviarle notificaciones y alertas en función de sus preferencias.

## <a name="restore-and-replicate-with-a-customers-managed-key-in-key-vault"></a>Restauración y réplica con la clave administrada del cliente en Key Vault

Después de cifrar Azure Database for MySQL con la clave administrada de un cliente almacenada en Key Vault, también se cifra cualquier copia recién creada del servidor. Puede realizar esta nueva copia mediante una operación local o de restauración geográfica, o por medio de réplicas de lectura. Sin embargo, la copia se puede cambiar para reflejar la nueva clave administrada por el cliente para el cifrado. Cuando se cambia la clave administrada por el cliente, las copias de seguridad antiguas del servidor comienzan a usar la clave más reciente.

Para evitar incidencias al configurar el cifrado de datos administrado por el cliente durante la restauración o la creación de réplicas de lectura, es importante seguir estos pasos en el servidor maestro o en el servidor de réplica o restaurado:

* Inicie el proceso de restauración o creación de réplica de lectura desde la Azure Database for MySQL maestra.
* Mantenga el servidor recién creado (de réplica o restaurado) en un estado inaccesible, ya que su identidad única todavía no tiene permisos para Key Vault.
* En el servidor restaurado/réplica, vuelva a validar la clave administrada por el cliente en la configuración de cifrado de datos para asegurarse de que el servidor recién creado tiene los permisos de ajuste y desajuste para la clave almacenada en Azure Key Vault.

## <a name="limitations"></a>Limitaciones

En el caso de Azure Database for MySQL, la compatibilidad con el cifrado de datos en reposo con la clave administrada de clientes (CMK) tiene algunas limitaciones:

* La compatibilidad con esta funcionalidad se limita a los planes de tarifa para **De uso general** y **Optimizados para memoria**.
* Esta característica solo se admite en regiones y servidores que admiten almacenamiento de hasta 16 TB. Para ver la lista de regiones de Azure que admiten almacenamiento de hasta 16 TB, consulte [aquí](concepts-pricing-tiers.md#storage) la sección de almacenamiento de la documentación.

    > [!NOTE]
    > - En todos los nuevos servidores MySQL creados en las regiones mencionadas anteriormente, el soporte para el cifrado con claves de administrador de clientes está **disponible**. El servidor de Restauración a un momento dado (PITR) o la réplica de lectura no calificarán aunque en teoría son "nuevos".
    > - Para validar si el servidor aprovisionado admite hasta 16 TB, puede ir a la hoja del plan de tarifa en el portal y ver el tamaño de almacenamiento máximo admitido por el servidor aprovisionado. Si puede subir el control deslizante hasta 4 TB, es posible que el servidor no admita el cifrado con claves administradas por el cliente. Pero los datos se cifran en todo momento con claves administradas por el servicio. Póngase en contacto con AskAzureDBforMySQL@service.microsoft.com si tiene alguna pregunta.

* El cifrado solo se admite con la clave criptográfica RSA 2048.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre cómo [configurar el cifrado de datos con una clave administrada por el cliente en Azure Database for MySQL mediante Azure Portal](howto-data-encryption-portal.md).
