---
title: 'Protección del acceso a un almacén de claves: Azure Key Vault | Microsoft Docs'
description: Administración de los permisos de acceso para Azure Key Vault, claves y secretos. Trata sobre el modelo de autenticación y autorización de Key Vault y cómo proteger un almacén de claves.
services: key-vault
author: ShaneBala-keyvault
manager: ravijan
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 05/11/2020
ms.author: sudbalas
ms.openlocfilehash: f9995b82c1dc437cdaa2f9f987abba3e9681454a
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87926763"
---
# <a name="secure-access-to-a-key-vault"></a>Protección del acceso a un almacén de claves

Azure Key Vault es un servicio en la nube que protege las claves de cifrado y los secretos, como certificados, cadenas de conexión y contraseñas. Dado que estos datos son confidenciales y críticos para la empresa, debe proteger el acceso a los almacenes de claves, de modo que solo se admita el acceso de las aplicaciones y los usuarios autorizados. En este artículo se proporciona información general del modelo de acceso de Key Vault. Se explican la autenticación y la autorización, y se describe cómo proteger el acceso a los almacenes de claves.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="access-model-overview"></a>Introducción al modelo acceso

El acceso a un almacén de claves se controla mediante dos interfaces: el **plano de administración** y el **plano de datos**. El plano de administración es donde puede administrar el propio almacén de claves. Las operaciones en este plano incluyen crear y eliminar los almacenes de claves, recuperar las propiedades de un almacén de claves y actualizar las directivas de acceso. El plano de datos es donde se trabaja con los datos almacenados en un almacén de claves. Puede agregar, eliminar y modificar claves, secretos y certificados.

Para obtener acceso a un almacén de claves en cualquier plano, todos los llamadores (usuarios o aplicaciones) deben tener una autorización y autenticación correctas. La autenticación establece la identidad del llamador. La autorización determina las operaciones que puede ejecutar el llamador.

Ambos planos usan Azure Active Directory (Azure AD) para la autenticación. Para la autorización, el plano de administración usa el control de acceso basado en rol (RBAC) y el plano de datos utiliza la directiva de acceso de Key Vault.

## <a name="active-directory-authentication"></a>Autenticación de Active Directory

Cuando se crea un almacén de claves en una suscripción de Azure, se asocia automáticamente al inquilino de Azure AD de dicha suscripción. En ambos planos, todos los llamadores deben registrarse en este inquilino y autenticarse para acceder al almacén de claves. En ambos casos, las aplicaciones pueden acceder a Key Vault de dos maneras:

- **Acceso de usuario y aplicación**: la aplicación accede a Key Vault en nombre de un usuario que ha iniciado sesión. Los ejemplos de este tipo de acceso incluyen Azure PowerShell y Azure Portal. Se concede acceso de usuario de dos maneras. Los usuarios pueden acceder a Key Vault desde cualquier aplicación o deben usar una aplicación específica (que se conoce como _identidad compuesta_).
- **Acceso de la aplicación solamente**: la aplicación se ejecuta como un servicio de demonio o un trabajo en segundo plano. A la identidad de la aplicación se le concede acceso al almacén de claves.

Para ambos tipos de acceso, la aplicación se autentica con Azure AD. La aplicación utiliza cualquiera [método de autenticación compatible](../../active-directory/develop/authentication-scenarios.md) según el tipo de aplicación. La aplicación adquiere un token para un recurso del plano para conceder acceso. El recurso es un punto de conexión en el plano de administración o de datos, según el entorno de Azure. La aplicación usa el token y envía la solicitud de una API de REST a Key Vault. Para más información, revise [todo el flujo de autenticación](../../active-directory/develop/v2-oauth2-auth-code-flow.md).

El modelo de un único mecanismo de autenticación para ambos planos tiene varias ventajas:

- Las organizaciones pueden controlar el acceso de forma centralizada a todos los almacenes de claves de su organización.
- Si un usuario abandona la organización, al instante pierde el acceso a todos los almacenes de claves de la organización.
- Las organizaciones pueden personalizar la autenticación mediante las opciones de Azure AD, como la habilitación de Multi-Factor Authentication para aumentar la seguridad.

## <a name="resource-endpoints"></a>Puntos de conexión de recursos

Las aplicaciones acceden a los planos a través de puntos de conexión. Los controles de acceso para los dos planos funcionan de forma independiente. Para conceder acceso a un aplicación para usar las claves de un almacén de claves, debe conceder acceso al plano de datos mediante una directiva de acceso de Key Vault. Para conceder a un usuario acceso de lectura a las propiedades y etiquetas de Key Vault, pero sin que este pueda acceder a los datos (claves, secretos o certificados), debe conceder acceso al plano de administración con RBAC.

En la siguiente tabla se muestran los puntos de conexión para los planos de administración y datos.

| Plano de&nbsp;acceso | Puntos de conexión de acceso | Operaciones | Mecanismo de&nbsp;control de acceso |
| --- | --- | --- | --- |
| Plano de administración | **Global:**<br> management.azure.com:443<br><br> **Azure China 21Vianet:**<br> management.chinacloudapi.cn:443<br><br> **Azure US Gov:**<br> management.usgovcloudapi.net:443<br><br> **Azure Alemania:**<br> management.microsoftazure.de:443 | Crear, leer, actualizar y eliminar almacenes de claves<br><br>Establecer directivas de acceso de Key Vault<br><br>Establecer etiquetas de Key Vault | Azure RBAC |
| Plano de datos | **Global:**<br> &lt;vault-name&gt;.vault.azure.net:443<br><br> **Azure China 21Vianet:**<br> &lt;vault-name&gt;.vault.azure.cn:443<br><br> **Azure US Gov:**<br> &lt;vault-name&gt;.vault.usgovcloudapi.net:443<br><br> **Azure Alemania:**<br> &lt;vault-name&gt;.vault.microsoftazure.de:443 | Claves: descifrar, cifrar,<br> desencapsular, encapsular, verificar, firmar,<br> obtener, enumerar, actualizar, crear,<br> importar, eliminar, hacer copia de seguridad y restaurar<br><br> Secretos: obtener, enumerar, establecer y eliminar | Directiva de acceso de Key Vault |

## <a name="management-plane-and-rbac"></a>Plano de administración y RBAC

En el plano de administración, utilice el control de acceso basado en roles de Azure (Azure RBAC) para autorizar las operaciones que el autor de la llamada puede ejecutar. En el modelo de RBAC, cada suscripción de Azure tiene una instancia de Azure AD. Puede conceder acceso a usuarios, grupos y aplicaciones desde este directorio. El acceso se concede para administrar recursos de la suscripción de Azure que usan el modelo de implementación de Azure Resource Manager. Para conceder acceso, use [Azure Portal](https://portal.azure.com/), la [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest), [Azure PowerShell](/powershell/azure/) o las [API de REST de Azure Resource Manager](https://msdn.microsoft.com/library/azure/dn906885.aspx).

Puede crear un almacén de claves en un grupo de recursos y administrar el acceso mediante Azure AD. Puede conceder a usuarios o grupos la capacidad de administrar los almacenes de claves en un grupo de recursos. Puede conceder acceso a un nivel de ámbito específico mediante la asignación de roles de Azure apropiados. Para conceder acceso a un usuario para administrar almacén de claves, debe asignar un rol `key vault Contributor` predefinido al usuario en un ámbito específico. Los siguientes niveles de ámbitos se pueden asignar a un rol de Azure:

- **Suscripción**: Un rol de Azure asignado al nivel de suscripción se aplica a todos los recursos y grupos de recursos de esa suscripción.
- **Grupo de recursos**: Un rol de Azure asignado al nivel de grupo de recursos se aplica a todos los recursos de dicho grupo de recursos.
- **Recurso específico**: un rol de Azure asignado a un recurso concreto se aplica a dicho recurso. En este caso, el recurso es un almacén de claves específico.

Existen varios roles predefinidos. Si un rol predefinido no se ajusta a sus necesidades, puede definir uno propio. Para más información, vea [RBAC: roles integrados](../../role-based-access-control/built-in-roles.md).

> [!IMPORTANT]
> Si un usuario tiene permisos `Contributor` en un plano de administración de Key Vault, se puede conceder a sí mismo acceso al plano de datos estableciendo una directiva de acceso de Key Vault. Debe controlar estrechamente quién tiene el rol de acceso `Contributor` a los almacenes de claves. Asegúrese de que solo las personas autorizadas pueden acceder a los almacenes de claves, las claves, los secretos y los certificados y administrarlos.
>

<a id="data-plane-access-control"></a>
## <a name="data-plane-and-access-policies"></a>Directivas de acceso y plano de datos

El acceso al plano de datos se concede mediante el establecimiento de directivas de acceso de Key Vault para un almacén de claves. Para establecer estas directivas de acceso, un usuario, grupo o aplicación deben tener permisos `Contributor` para el plano de administración de dicho almacén de claves.

Puede conceder acceso a un usuario, grupo o aplicación para ejecutar operaciones concretas en las claves o los secretos de un almacén de claves. Key Vault admite hasta 1024 entradas de directivas de acceso para un almacén de claves. Para conceder acceso al plano de datos a varios usuarios, cree un grupo de seguridad de Azure AD y agregue usuarios a dicho grupo.

Puede ver la lista completa de operaciones de almacén y secreto, y comprender las operaciones que se permiten al configurar las directivas de acceso de Key Vault mediante la consulta de la siguiente referencia. [Referencia de operación de Key Vault](https://docs.microsoft.com/rest/api/keyvault/#vault-operations)

<a id="key-vault-access-policies"></a> Las directivas de acceso de Key Vault conceden permisos a las claves, los secretos y los certificados por separado. Puede conceder acceso a un usuario solo a las claves y no a los secretos. Los permisos para acceder a las claves, los secretos y los certificados se encuentran en el nivel del almacén. Las directivas de acceso de Key Vault no admiten permisos de nivel de objeto pormenorizados, como una clave, un secreto o un certificado específicos. Para establecer directivas de acceso de un almacén de claves, use [Azure Portal](https://portal.azure.com/), la [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest), [Azure PowerShell](/powershell/azure/) o las [API de REST de administración de Key Vault](https://msdn.microsoft.com/library/azure/mt620024.aspx).

> [!IMPORTANT]
> Las directivas de acceso de Key Vault se aplican en el nivel de almacén. Cuando se concede a un usuario permiso para crear y eliminar claves, este puede realizar dichas operaciones en todas las claves de dicho almacén de claves.
>

Puede restringir el acceso al plano de datos mediante el uso de [puntos de conexión de servicio de red virtual para Azure Key Vault](overview-vnet-service-endpoints.md). Puede configurar [firewall y reglas de red virtual](network-security.md) para una capa adicional de seguridad.

## <a name="example"></a>Ejemplo

En este ejemplo, se desarrolla una aplicación que usa un certificado para TLS/SSL, Azure Storage para almacenar los datos y una clave RSA de 2048 bits para las operaciones de firma. La aplicación se ejecuta en una máquina virtual (VM) de Azure o en un conjunto de escalado de máquinas virtuales. Podemos usar un almacén de claves para almacenar los secretos de la aplicación. Podemos almacenar el certificado de arranque que usa la aplicación para autenticarse con Azure AD.

Se necesita acceso a los siguientes secretos y claves almacenados:
- **Certificado TLS/SSL**: Se usa para TLS/SSL.
- **Clave de almacenamiento**: se usa para acceder a la cuenta de almacenamiento.
- **Clave RSA de 2048 bits**: se usa para las operaciones de firma.
- **Certificado de arranque**: se usa para la autenticación con Azure AD. Una vez que se concede acceso, se puede recuperar la clave de almacenamiento y usar la clave RSA para la firma.

Es necesario definir los siguientes roles para especificar quién puede administrar, implementar y auditar la aplicación:
- **Equipo de seguridad**: el personal de TI de la oficina del CSO (director de seguridad) o colaboradores equivalentes. El equipo de seguridad es responsable de la protección adecuada de los secretos. Los secretos pueden incluir los certificados TLS/SSL, las claves RSA usadas para la firma, las cadenas de conexión y las claves de la cuenta de almacenamiento.
- **Desarrolladores y operadores**: personal que desarrolla la aplicación y la implementa en Azure. Los miembros de este equipo no forman parte del personal de seguridad. No deben tener acceso a información confidencial, como los certificados TLS/SSL y las claves RSA. Solo la aplicación que implementan debe tener acceso a información confidencial.
- **Auditores**: este rol es para los colaboradores que no sean miembros del personal de TI general o de desarrollo. Deben revisar el uso y mantenimiento de los certificados, las claves y los secretos para garantizar el cumplimiento de los estándares de seguridad.

Hay otro rol que está fuera del ámbito de la aplicación: el administrador de la suscripción o del grupo de recursos. El administrador de la suscripción configura los permisos de acceso iniciales del equipo de seguridad. Conceden acceso al equipo de seguridad mediante el uso de un grupo de recursos que tiene los recursos requeridos por la aplicación.

Es necesario autorizar las siguientes operaciones para los roles:

**Equipo de seguridad**
- Crear instancias de Key Vault.
- Activar el registro de Key Vault.
- Agregar claves y secretos.
- Crear copias de seguridad de las claves para la recuperación ante desastres.
- Establecer directivas de acceso de Key Vault para conceder permisos a usuarios y aplicaciones para operaciones concretas.
- Rotar periódicamente las claves y los secretos.

**Desarrolladores y operadores**
- Obtener referencias del equipo de seguridad para los certificados de arranque y TLS/SSL (huellas digitales), la clave de almacenamiento (identificador URI de secreto) y la clave RSA (identificador URI de clave) para la firma.
- Desarrollar e implementar una aplicación que acceda a las claves y los secretos mediante programación.

**Auditores**
- Revisar los registros de Key Vault para confirmar el uso adecuado de las claves y los secretos, así como el cumplimiento de los estándares de seguridad de datos.

En la tabla siguiente se resumen los permisos de acceso para los roles y la aplicación.

| Role | Permisos del plano de administración | Permisos del plano de datos |
| --- | --- | --- |
| Equipo de seguridad | Colaborador de almacén de claves | Claves: copia de seguridad, creación, eliminación, obtención, importación, lista y restauración<br>Secretos: todas las operaciones |
| Desarrolladores y&nbsp;operadores | Permiso de implementación en Key Vault<br><br> **Nota**: Este permiso permite que las máquinas virtuales implementadas recuperen los secretos de un almacén de claves. | None |
| Auditores | None | Claves: enumeración<br>Secretos: enumeración<br><br> **Nota**: Este permiso permite a los auditores inspeccionar los atributos (etiquetas, fechas de activación y fechas de expiración) para las claves y los secretos que no se emiten en los registros. |
| Application | None | Claves: firma<br>Secretos: obtención |

Los tres roles de equipo necesitan tener acceso a otros recursos junto con los permisos de Key Vault. Para implementar las máquinas virtuales o la característica Web Apps de Azure App Service, los desarrolladores y operadores también necesitan acceso `Contributor` a dichos tipos de recursos. Los auditores necesitan acceso de lectura a la cuenta de almacenamiento donde se almacenan los registros de Key Vault.

Para más información sobre cómo implementar certificados, claves de acceso y secretos mediante programación, vea estos recursos:
- Obtenga más información sobre cómo [implementar certificados en máquinas virtuales desde una instancia de Key Vault administrada por un cliente](https://blogs.technet.microsoft.com/kv/2016/09/14/updated-deploy-certificates-to-vms-from-customer-managed-key-vault/) (entrada de blog).
- Vea los [ejemplos de cliente de Azure Key Vault](https://docs.microsoft.com/samples/browse/?term=Key%20Vault). En este contenido se ilustra cómo usar un certificado de arranque para autenticarse en Azure AD con la finalidad de acceder a un almacén de claves.

Puede conceder la mayoría de los permisos de acceso mediante Azure Portal. Para conceder permisos pormenorizados, puede usar Azure PowerShell o la CLI de Azure.

Los fragmentos de código de PowerShell de esta sección se crean con los siguientes supuestos:
- El administrador de Azure AD ha creado grupos de seguridad para representar los tres roles: Contoso Security Team, Contoso App DevOps y Contoso App Auditors. El administrador ha agregado usuarios a sus respectivos grupos.
- Todos los recursos se encuentran en el grupo de recursos **ContosoAppRG**.
- Los registros de Key Vault se almacenan en la cuenta de almacenamiento **contosologstorage**.
- La instancia de Key Vault **ContosoKeyVault** y la cuenta de almacenamiento **contosologstorage** están en la misma ubicación de Azure.

El administrador de la suscripción asigna los roles `key vault Contributor` y `User Access Administrator` al equipo de seguridad. Estos roles permiten al equipo de seguridad administrar el acceso a otros recursos y los almacenes de claves, y ambos se encuentran en el grupo de recursos **ContosoAppRG**.

```powershell
New-AzRoleAssignment -ObjectId (Get-AzADGroup -SearchString 'Contoso Security Team')[0].Id -RoleDefinitionName "key vault Contributor" -ResourceGroupName ContosoAppRG
New-AzRoleAssignment -ObjectId (Get-AzADGroup -SearchString 'Contoso Security Team')[0].Id -RoleDefinitionName "User Access Administrator" -ResourceGroupName ContosoAppRG
```

El equipo de seguridad crea un almacén de claves y configura los permisos de acceso y el registro.

```powershell
# Create a key vault and enable logging
$sa = Get-AzStorageAccount -ResourceGroup ContosoAppRG -Name contosologstorage
$kv = New-AzKeyVault -Name ContosoKeyVault -ResourceGroup ContosoAppRG -SKU premium -Location 'westus' -EnabledForDeployment
Set-AzDiagnosticSetting -ResourceId $kv.ResourceId -StorageAccountId $sa.Id -Enabled $true -Category AuditEvent

# Set up data plane permissions for the Contoso Security Team role
Set-AzKeyVaultAccessPolicy -VaultName ContosoKeyVault -ObjectId (Get-AzADGroup -SearchString 'Contoso Security Team')[0].Id -PermissionsToKeys backup,create,delete,get,import,list,restore -PermissionsToSecrets get,list,set,delete,backup,restore,recover,purge

# Set up management plane permissions for the Contoso App DevOps role
# Create the new role from an existing role
$devopsrole = Get-AzRoleDefinition -Name "Virtual Machine Contributor"
$devopsrole.Id = $null
$devopsrole.Name = "Contoso App DevOps"
$devopsrole.Description = "Can deploy VMs that need secrets from a key vault"
$devopsrole.AssignableScopes = @("/subscriptions/<SUBSCRIPTION-GUID>")

# Add permissions for the Contoso App DevOps role so members can deploy VMs with secrets deployed from key vaults
$devopsrole.Actions.Add("Microsoft.KeyVault/vaults/deploy/action")
New-AzRoleDefinition -Role $devopsrole

# Assign the new role to the Contoso App DevOps security group
New-AzRoleAssignment -ObjectId (Get-AzADGroup -SearchString 'Contoso App Devops')[0].Id -RoleDefinitionName "Contoso App Devops" -ResourceGroupName ContosoAppRG

# Set up data plane permissions for the Contoso App Auditors role
Set-AzKeyVaultAccessPolicy -VaultName ContosoKeyVault -ObjectId (Get-AzADGroup -SearchString 'Contoso App Auditors')[0].Id -PermissionsToKeys list -PermissionsToSecrets list
```

Los roles personalizados definidos solo se pueden asignar a la suscripción en la que se crea el grupo de recursos **ContosoAppRG**. Para usar un rol personalizado para otros proyectos de otras suscripciones, agregue otras suscripciones al ámbito del rol.

Para el personal de DevOps, la asignación de roles personalizados para el permiso `deploy/action` del almacén de claves se realiza en el ámbito del grupo de recursos. Solo a las máquinas virtuales creadas en el grupo de recursos **ContosoAppRG** se les permite acceder a los secretos (certificados de arranque y TLS/SSL). Las máquinas virtuales creadas en otros grupos de recursos por un miembro de DevOps no pueden acceder a estos secretos, incluso si la máquina virtual tiene los identificadores URI de los secretos.

En el ejemplo se describe un escenario sencillo. Los escenarios reales pueden ser más complejos. Puede ajustar los permisos para el almacén de claves según sus necesidades. Se supone que el equipo de seguridad proporciona las referencias de clave y secreto (identificadores URI y huellas digitales), que el personal de DevOps utiliza en sus aplicaciones. Los desarrolladores y operadores no requieren ningún acceso al plano de datos. Nos centramos en cómo proteger el almacén de claves. Debe darse una consideración parecida a la protección de [las máquinas virtuales](https://azure.microsoft.com/services/virtual-machines/security/), [las cuentas de almacenamiento](../../storage/blobs/security-recommendations.md) y otros recursos de Azure.

> [!NOTE]
> En este ejemplo se muestra cómo se bloquea el acceso a Key Vault en producción. Los desarrolladores deben tener su propia suscripción o grupo de recursos con permisos completos para administrar sus almacenes, máquinas virtuales y la cuenta de almacenamiento donde desarrollan la aplicación.

Se recomienda configurar una protección de acceso adicional al almacén de claves mediante la [configuración de los firewalls y las redes virtuales de Key Vault](network-security.md).

## <a name="resources"></a>Recursos

* [Azure RBAC](../../role-based-access-control/role-assignments-portal.md)

* [RBAC: roles integrados](../../role-based-access-control/built-in-roles.md)

* [Descripción de la implementación de Resource Manager y la implementación clásica](../../azure-resource-manager/management/deployment-models.md)

* [Administración de RBAC con Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md)

* [Administración de RBAC con la API de REST](../../role-based-access-control/role-assignments-rest.md)

* [RBAC para Microsoft Azure](https://channel9.msdn.com/events/Ignite/2015/BRK2707)

    En este vídeo de la conferencia Microsoft Ignite de 2015 se tratan las funcionalidades de administración del acceso y la generación de informes en Azure. También se exploran los procedimientos recomendados para proteger el acceso a suscripciones de Azure con Azure AD.

* [Autorización del acceso a aplicaciones web mediante OAuth 2.0 y Azure AD](../../active-directory/develop/v2-oauth2-auth-code-flow.md)

* [API de REST de administración de Key Vault](https://msdn.microsoft.com/library/azure/mt620024.aspx)

    La referencia para que las API de REST administren un almacén de claves mediante programación, incluido el establecimiento de la directiva de acceso de Key Vault.

* [API de REST de Key Vault](https://msdn.microsoft.com/library/azure/dn903609.aspx)

* [Control de acceso de claves](https://msdn.microsoft.com/library/azure/dn903623.aspx#BKMK_KeyAccessControl)

* [Control de acceso de secretos](https://msdn.microsoft.com/library/azure/dn903623.aspx#BKMK_SecretAccessControl)

* [Establecer](/powershell/module/az.keyvault/Set-azKeyVaultAccessPolicy) y [quitar](/powershell/module/az.keyvault/Remove-azKeyVaultAccessPolicy) una directiva de acceso de Key Vault mediante PowerShell

## <a name="next-steps"></a>Pasos siguientes

Configure [firewalls y redes virtuales de Key Vault](network-security.md).

Para consultar un tutorial de introducción para un administrador, consulte [¿Qué es Azure Key Vault?](overview.md)

Para más información acerca del registro de uso para Key Vault, consulte [Registro de Azure Key Vault](logging.md).

Para más información acerca del uso de claves y secretos con Azure Key Vault, consulte [Información acerca de claves, secretos y certificados](https://msdn.microsoft.com/library/azure/dn903623.aspx).

Si le queda alguna duda acerca de Key Vault, visite la [Página de preguntas y respuestas de Microsoft](https://docs.microsoft.com/answers/topics/azure-key-vault.html).
