---
title: 'Protección de registros y zonas DNS: Azure DNS'
description: En esta ruta de aprendizaje, comience a proteger los conjuntos de registros y zonas DNS en Microsoft Azure DNS.
services: dns
author: asudbring
ms.service: dns
ms.topic: how-to
ms.date: 2/20/2020
ms.author: allensu
ms.openlocfilehash: fbf996a66bf76520002a22169903c7961080e38b
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87131538"
---
# <a name="how-to-protect-dns-zones-and-records"></a>Cómo proteger registros y zonas DNS

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Los registros y las zonas DNS son recursos críticos. La eliminación de una zona DNS o incluso de un solo un registro DNS puede dar lugar a una interrupción total del servicio. Es importante proteger las zonas y los registros DNS contra cambios accidentales o no autorizados.

En este artículo se explica cómo Azure DNS le permite proteger las zonas y los registros DNS privados de dichos cambios.  Se aplican dos características de seguridad eficaces que proporciona Azure Resource Manager: [control de acceso basado en rol](../role-based-access-control/overview.md) y [bloqueos de recursos](../azure-resource-manager/management/lock-resources.md).

## <a name="role-based-access-control"></a>Control de acceso basado en rol

El control de acceso basado en rol de Azure (Azure RBAC) permite realizar una administración detallada del acceso de usuarios, grupos y recursos de Azure. Con RBAC, puede conceder el nivel de acceso que necesitan los usuarios. Para más información sobre cómo RBAC ayuda a administrar el acceso, vea [¿Qué es el control de acceso basado en rol de Azure (RBAC)?](../role-based-access-control/overview.md).

### <a name="the-dns-zone-contributor-role"></a>Rol Colaborador de zona DNS

El rol de colaborador de zona DNS es un rol integrado para administrar recursos DNS privados. Este rol aplicado a un usuario o un grupo les permite administrar recursos DNS.

El grupo de recursos *myResourceGroup* contiene cinco zonas de Contoso Corporation. Cuando se conceden permisos de Colaborador de zona DNS de administrador de DNS a ese grupo de recursos, se permite el control total sobre esas zonas DNS. Se evita la concesión de permisos innecesarios. El administrador de DNS no puede crear ni detener máquinas virtuales.

La manera más sencilla de asignar permisos RBAC es [a través de Azure Portal](../role-based-access-control/role-assignments-portal.md).  

Abra **Control de acceso (IAM)** del grupo de recursos, seleccione **Agregar** y, luego, seleccione el rol **Colaborador de zona DNS**. Seleccione los usuarios o grupos necesarios para conceder permisos.

![RBAC de nivel de grupo de recursos a través de Azure Portal](./media/dns-protect-zones-recordsets/rbac1.png)

Los permisos también se pueden [conceder mediante Azure PowerShell](../role-based-access-control/role-assignments-powershell.md):

```azurepowershell
# Grant 'DNS Zone Contributor' permissions to all zones in a resource group

$usr = "<user email address>"
$rol = "DNS Zone Contributor"
$rsg = "<resource group name>"

New-AzRoleAssignment -SignInName $usr -RoleDefinitionName $rol -ResourceGroupName $rsg
```

El comando equivalente también está [disponible a través de la CLI de Azure](../role-based-access-control/role-assignments-cli.md):

```azurecli
# Grant 'DNS Zone Contributor' permissions to all zones in a resource group

az role assignment create \
--assignee "<user email address>" \
--role "DNS Zone Contributor" \
--resource-group "<resource group name>"
```

### <a name="zone-level-rbac"></a>RBAC de nivel de zona

Las reglas de RBAC de Azure pueden aplicarse a una suscripción, a un grupo de recursos o a un recurso individual. Ese recurso puede ser una zona DNS individual o un conjunto de registros individual.

Por ejemplo, el grupo de recursos *myResourceGroup* contiene la zona *contoso.com* y una subzona *customers.contoso.com*. Para cada cuenta de cliente se crean registros CNAME. La cuenta de administrador que se usa para administrar los registros CNAME recibe permisos para crear registros en la zona *customers.contoso.com*. La cuenta solo puede administrar *customers.contoso.com*.

Los permisos RBAC de nivel de zona se pueden conceder a través de Azure Portal.  Abra **Control de acceso (IAM)** para la zona, seleccione **Agregar** y, a continuación, seleccione el rol **Colaborador de zona DNS** y los usuarios o grupos a los que necesita conceder permisos.

![RBAC de nivel de zona DNS a través de Azure Portal](./media/dns-protect-zones-recordsets/rbac2.png)

Los permisos también se pueden [conceder mediante Azure PowerShell](../role-based-access-control/role-assignments-powershell.md):

```azurepowershell
# Grant 'DNS Zone Contributor' permissions to a specific zone

$usr = "<user email address>"
$rol = "DNS Zone Contributor"
$rsg = "<resource group name>"
$zon = "<zone name>"
$typ = "Microsoft.Network/DNSZones"

New-AzRoleAssignment -SignInName $usr -RoleDefinitionName $rol -ResourceGroupName $rsg -ResourceName $zon -ResourceType $typ
```

El comando equivalente también está [disponible a través de la CLI de Azure](../role-based-access-control/role-assignments-cli.md):

```azurecli
# Grant 'DNS Zone Contributor' permissions to a specific zone

az role assignment create \
--assignee <user email address> \
--role "DNS Zone Contributor" \
--scope "/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Network/DnsZones/<zone name>/"
```

### <a name="record-set-level-rbac"></a>RBAC de nivel de conjunto de registros

Los permisos se aplican en el nivel de conjunto de registros.  Se concede al usuario control sobre las entradas que necesita y no puede realizar ningún otro cambio.

Los permisos RBAC de nivel de conjunto de registros pueden configurarse mediante Azure Portal, con el botón **Control de acceso (IAM)** de la página del conjunto de registros:

![RBAC de nivel de conjunto de registros a través de Azure Portal](./media/dns-protect-zones-recordsets/rbac3.png)

Los permisos RBAC de nivel de conjunto de registros también se pueden [conceder mediante Azure PowerShell](../role-based-access-control/role-assignments-powershell.md):

```azurepowershell
# Grant permissions to a specific record set

$usr = "<user email address>"
$rol = "DNS Zone Contributor"
$sco = 
"/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Network/dnszones/<zone name>/<record type>/<record name>"

New-AzRoleAssignment -SignInName $usr -RoleDefinitionName $rol -Scope $sco
```

El comando equivalente también está [disponible a través de la CLI de Azure](../role-based-access-control/role-assignments-cli.md):

```azurecli
# Grant permissions to a specific record set

az role assignment create \
--assignee "<user email address>" \
--role "DNS Zone Contributor" \
--scope "/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Network/dnszones/<zone name>/<record type>/<record name>"
```

### <a name="custom-roles"></a>Roles personalizados

El rol Colaborador de zona DNS integrado permite el control total sobre un recurso DNS. También es posible crear roles de Azure de cliente propios, para proporcionar un control más detallado.

La cuenta que se use para administrar CNAME solo debe tener permiso para administrar registros CNAME. La cuenta no puede modificar registros de otros tipos. La cuenta no puede realizar operaciones de nivel de zona, como la eliminación de zonas.

En el ejemplo siguiente se muestra una definición de rol personalizado para administrar únicamente registros CNAME:

```json
{
    "Name": "DNS CNAME Contributor",
    "Id": "",
    "IsCustom": true,
    "Description": "Can manage DNS CNAME records only.",
    "Actions": [
        "Microsoft.Network/dnsZones/CNAME/*",
        "Microsoft.Network/dnsZones/read",
        "Microsoft.Authorization/*/read",
        "Microsoft.Insights/alertRules/*",
        "Microsoft.ResourceHealth/availabilityStatuses/read",
        "Microsoft.Resources/deployments/*",
        "Microsoft.Resources/subscriptions/resourceGroups/read",
        "Microsoft.Support/*"
    ],
    "NotActions": [
    ],
    "AssignableScopes": [
        "/subscriptions/c276fc76-9cd4-44c9-99a7-4fd71546436e"
    ]
}
```

La propiedad Actions define los siguientes permisos específicos de DNS:

* `Microsoft.Network/dnsZones/CNAME/*` concede control total sobre registros CNAME
* `Microsoft.Network/dnsZones/read` concede permiso para leer zonas DNS, pero no para modificarlas, lo que permite ver la zona en la que se crea el registro CNAME.

Las acciones restantes se copian desde el [Rol de colaborador de zona DNS integrado](../role-based-access-control/built-in-roles.md#dns-zone-contributor).

> [!NOTE]
> El uso de un rol personalizado de Azure para evitar que se eliminen conjuntos de registros mientras se permite modificarlos no es un control eficaz. Evita que los conjuntos de registros se eliminen, pero no que se modifiquen.  Entre las modificaciones permitidas figuran la adición y eliminación de registros desde el conjunto de registros, incluida la eliminación de todos los registros para dejar un conjunto de registros empty. Esto tiene el mismo efecto que eliminar el conjunto de registros desde un punto de vista de la resolución DNS.

Actualmente no pueden realizarse definiciones de roles personalizados a través de Azure Portal. Puede crearse un rol personalizado basado en esta definición de rol mediante Azure PowerShell:

```azurepowershell
# Create new role definition based on input file
New-AzRoleDefinition -InputFile <file path>
```

También puede crearse a través de la CLI de Azure:

```azurecli
# Create new role definition based on input file
az role create -inputfile <file path>
```

El rol se puede asignar después del mismo modo que los roles integrados, tal cual se describe anteriormente en este artículo.

Para obtener más información sobre cómo crear, administrar y asignar roles personalizados, vea [Roles personalizados en RBAC de Azure](../role-based-access-control/custom-roles.md).

## <a name="resource-locks"></a>Bloqueos de recursos

Azure Resource Manager admite otro tipo de control de seguridad: la posibilidad de bloquear recursos. Los bloqueos de recursos se aplican al recurso y están en vigor en todos los usuarios y roles. Para obtener más información, consulte [Bloqueo de recursos con el Administrador de recursos de Azure](../azure-resource-manager/management/lock-resources.md).

Existen dos tipos de bloqueos de recursos: **CanNotDelete** y **ReadOnly**. Estos tipos de bloqueos pueden aplicarse a una zona DNS privada o a un conjunto de registros individual. En las secciones siguientes se describen varios escenarios comunes y cómo mantenerlos con bloqueos de recursos.

### <a name="protecting-against-all-changes"></a>Protección contra todos los cambios

Para evitar que se realicen cambios, aplique un bloqueo ReadOnly a la zona. Este bloqueo impide que se creen otros conjuntos de registros y que se modifiquen o eliminen conjuntos de registros existentes.

Pueden crearse bloqueos de recursos de nivel de zona a través de Azure Portal.  En la página de la zona DNS, seleccione **Bloqueos** y después seleccione **+ Agregar**:

![Bloqueos de recursos de nivel de zona a través de Azure Portal](./media/dns-protect-zones-recordsets/locks1.png)

También pueden crearse bloqueos de recursos de nivel de zona a través de [Azure PowerShell](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcelock?view=latest):

```azurepowershell
# Lock a DNS zone

$lvl = "<lock level>"
$lnm = "<lock name>"
$rsc = "<zone name>"
$rty = "Microsoft.Network/DNSZones"
$rsg = "<resource group name>"

New-AzResourceLock -LockLevel $lvl -LockName $lnm -ResourceName $rsc -ResourceType $rty -ResourceGroupName $rsg
```

El comando equivalente también está [disponible a través de la CLI de Azure](https://docs.microsoft.com/cli/azure/lock?view=azure-cli-latest#az-lock-create):

```azurecli
# Lock a DNS zone

az lock create \
--lock-type "<lock level>" \
--name "<lock name>" \
--resource-name "<zone name>" \
--namespace "Microsoft.Network" \
--resource-type "DnsZones" \
--resource-group "<resource group name>"
```

### <a name="protecting-individual-records"></a>Protección de registros individuales

Para evitar que se modifique un conjunto de registros de DNS existente, aplique un bloqueo ReadOnly al conjunto de registros.

> [!NOTE]
> La aplicación de un bloqueo CanNotDelete a un conjunto de registros no es un control eficaz. Evita que el conjunto de registros se elimine, pero no que se modifique.  Entre las modificaciones permitidas figuran la adición y eliminación de registros desde el conjunto de registros, incluida la eliminación de todos los registros para dejar un conjunto de registros empty. Esto tiene el mismo efecto que eliminar el conjunto de registros desde un punto de vista de la resolución DNS.

Actualmente, los bloqueos de recursos de nivel de conjunto de recursos solo pueden configurarse mediante Azure PowerShell.  No se admiten en Azure Portal ni en la CLI de Azure.

```azurepowershell
# Lock a DNS record set

$lvl = "<lock level>"
$lnm = "<lock name>"
$rsc = "<zone name>/<record set name>"
$rty = "Microsoft.Network/DNSZones/<record type>"
$rsg = "<resource group name>"

New-AzResourceLock -LockLevel $lvl -LockName $lnm -ResourceName $rsc -ResourceType $rty -ResourceGroupName $rsg
```

### <a name="protecting-against-zone-deletion"></a>Protección contra la eliminación de zonas

Cuando se elimina una zona en Azure DNS, se eliminan también todos los conjuntos de registros de la zona.  Esta operación no se puede deshacer. La eliminación accidental de una zona crítica tiene la posibilidad de tener un significativo impacto de negocio.  Es importante protegerse contra la eliminación accidental de zonas.

La aplicación de un bloqueo CanNotDelete a una zona impide que se elimine la zona. Los recursos secundarios heredan los bloqueos. Un bloqueo impide que se eliminen los conjuntos de registros de la zona. Como se describe en la nota anterior, no resulta eficaz puesto que todavía se pueden quitar registros de los conjuntos de registros existentes.

Como alternativa, aplique un bloqueo CanNotDelete a un conjunto de registros de la zona, como el conjunto de registros SOA. La zona no se elimina sin eliminar también los conjuntos de registros. Este bloqueo protege contra la eliminación de la zona, a la vez que permite que se modifiquen libremente los conjuntos de registros dentro de la zona. Si se realiza un intento de eliminar la zona, Azure Resource Manager detecta esta eliminación. La eliminación también eliminaría el conjunto de registros SOA; Azure Resource Manager bloquea la llamada porque SOA está bloqueada.  No se elimina ningún conjunto de registros.

El comando de PowerShell siguiente crea un bloqueo CanNotDelete contra el registro SOA de la zona especificada:

```azurepowershell
# Protect against zone delete with CanNotDelete lock on the record set

$lvl = "CanNotDelete"
$lnm = "<lock name>"
$rsc = "<zone name>/@"
$rty = "Microsoft.Network/DNSZones/SOA"
$rsg = "<resource group name>"

New-AzResourceLock -LockLevel $lvl -LockName $lnm -ResourceName $rsc -ResourceType $rty -ResourceGroupName $rsg
```

Otra opción para evitar la eliminación accidental de la zona es usar un rol personalizado. Este rol garantiza que las cuentas que se usan para administrar las zonas no tienen permisos de eliminación de zonas. 

Cuando sea necesario eliminar una zona, puede aplicar una eliminación de dos pasos:

 - En primer lugar, conceda permisos de eliminación de zonas.
 - En segundo lugar, conceda permisos para eliminar la zona.

El rol personalizado funciona para todas las zonas a las que se accede con esas cuentas. Las cuentas con permisos de eliminación de zonas, como el propietario de la suscripción, todavía pueden eliminar una zona por accidente.

Es posible usar ambos enfoques (bloqueos de recursos y roles personalizados) al mismo tiempo, como un método de defensa en profundidad para la protección de zonas DNS.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre cómo trabajar con RBAC, vea [Introducción a la administración de acceso en Azure Portal](../role-based-access-control/overview.md).
* Para más información sobre cómo trabajar con bloqueos de recursos, vea [Bloqueo de recursos con Azure Resource Manager](../azure-resource-manager/management/lock-resources.md).
