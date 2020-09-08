---
title: Administración de zonas DNS en Azure DNS - PowerShell | Microsoft Docs
description: Puede administrar zonas DNS con Azure Powershell. Este artículo describe cómo actualizar, eliminar y crear zonas DNS en Azure DNS
services: dns
documentationcenter: na
author: rohinkoul
ms.service: dns
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/19/2018
ms.author: rohink
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 347fde86b2a37aa0f82c09c94d6aa2f9e405da9d
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89075603"
---
# <a name="how-to-manage-dns-zones-using-powershell"></a>Cómo administrar zonas DNS con PowerShell

> [!div class="op_single_selector"]
> * [Portal](dns-operations-dnszones-portal.md)
> * [PowerShell](dns-operations-dnszones.md)
> * [CLI de Azure clásica](dns-operations-dnszones-cli-nodejs.md)
> * [CLI de Azure](dns-operations-dnszones-cli.md)

En este artículo se muestra cómo administrar sus zonas DNS mediante Azure PowerShell. También se pueden administrar las zonas DNS mediante la [CLI de Azure](dns-operations-dnszones-cli.md) multiplataforma o Azure Portal.

Esta guía trata específicamente con las zonas DNS públicas. Para obtener información acerca del uso de Azure PowerShell para administrar zonas privadas en Azure DNS, consulte [Get started with Azure DNS Private Zones using Azure CLI 2.0](private-dns-getstarted-powershell.md) (Introducción a Azure DNS Private Zones con Azure PowerShell).

[!INCLUDE [dns-create-zone-about](../../includes/dns-create-zone-about-include.md)]

[!INCLUDE [dns-powershell-setup](../../includes/dns-powershell-setup-include.md)]


## <a name="create-a-dns-zone"></a>Creación de una zona DNS

Una zona DNS se crea mediante el cmdlet `New-AzureRmDnsZone` .

En el ejemplo siguiente, se crea una zona DNS llamada *contoso.com* en el grupo de recursos *MyResourceGroup*:

```powershell
New-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup
```

En el ejemplo siguiente se muestra cómo crear una zona DNS con dos [etiquetas Azure Resource Manager](dns-zones-records.md#tags), *project = demo* y *env = test*:

```powershell
New-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup -Tag @{ project="demo"; env="test" }
```

Azure DNS también admite zonas DNS privadas.  Para más información acerca de las zonas DNS privadas, consulte [Using Azure DNS for private domains](private-dns-overview.md) (Uso de Azure DNS para dominios privados). Para obtener un ejemplo de cómo crear una zona DNS privada, vea [Introducción al uso de PowerShell con las zonas privadas de Azure DNS](./private-dns-getstarted-powershell.md).

## <a name="get-a-dns-zone"></a>Recuperación de una zona DNS

Para recuperar una zona DNS, use el cmdlet `Get-AzureRmDnsZone` . Esta operación devuelve un objeto de la zona DNS correspondiente a una zona existente en DNS de Azure. El objeto contiene datos sobre la zona (por ejemplo, el número de conjuntos de registros), pero no contiene los conjuntos de registros (vea `Get-AzureRmDnsRecordSet`).

```powershell
Get-AzureRmDnsZone -Name contoso.com –ResourceGroupName MyAzureResourceGroup

Name                  : contoso.com
ResourceGroupName     : myresourcegroup
Etag                  : 00000003-0000-0000-8ec2-f4879750d201
Tags                  : {project, env}
NameServers           : {ns1-01.azure-dns.com., ns2-01.azure-dns.net., ns3-01.azure-dns.org.,
                        ns4-01.azure-dns.info.}
NumberOfRecordSets    : 2
MaxNumberOfRecordSets : 5000
```

## <a name="list-dns-zones"></a>Enumeración de zonas DNS

Si se omite el nombre de la zona de `Get-AzureRmDnsZone`, puede enumerar todas las zonas en un grupo de recursos. Esta operación devuelve una matriz de objetos de la zona.

```powershell
$zoneList = Get-AzureRmDnsZone -ResourceGroupName MyAzureResourceGroup
```

Si se omite tanto el nombre de zona como el nombre del grupo de recursos de `Get-AzureRmDnsZone`, puede enumerar todas las zonas de la suscripción de Azure.

```powershell
$zoneList = Get-AzureRmDnsZone
```

## <a name="update-a-dns-zone"></a>Actualización de una zona DNS

Los cambios en los recursos de una zona DNS se pueden realizar mediante `Set-AzureRmDnsZone`. Este cmdlet no actualiza ninguno de los conjuntos de registros de DNS de la zona (consulte [Administración de registros DNS](dns-operations-recordsets.md)). Solo se utiliza para actualizar las propiedades de los recursos de la zona. Las propiedades de zona que se pueden escribir están actualmente limitadas a las ["etiquetas" Azure Resource Manager para el recurso de la zona](dns-zones-records.md#tags).

Utilice una de las dos opciones siguientes para actualizar una zona DNS:

### <a name="specify-the-zone-using-the-zone-name-and-resource-group"></a>Especificar la zona con la utilización del nombre de zona y el grupo de recursos

Este método reemplaza las etiquetas de zona existentes con los valores especificados.

```powershell
Set-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup -Tag @{ project="demo"; env="test" }
```

### <a name="specify-the-zone-using-a-zone-object"></a>Especificar la zona utilizando un objeto $zone

Este método recupera el objeto de zona existente, modifica las etiquetas y, a continuación, confirma los cambios. De este modo, se pueden conservar las etiquetas existentes.

```powershell
# Get the zone object
$zone = Get-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup

# Remove an existing tag
$zone.Tags.Remove("project")

# Add a new tag
$zone.Tags.Add("status","approved")

# Commit changes
Set-AzureRmDnsZone -Zone $zone
```

Al usar `Set-AzureRmDnsZone` con un objeto $zone, se utilizan las [comprobaciones de ETag](dns-zones-records.md#etags) para garantizar que no se sobrescriben los cambios simultáneos. Puede usar el modificador `-Overwrite` opcional para suprimir estas comprobaciones.

## <a name="delete-a-dns-zone"></a>Eliminación de una zona DNS

Las zonas DNS se pueden eliminar mediante el cmdlet `Remove-AzureRmDnsZone`.

> [!NOTE]
> Si se elimina una zona DNS, también se eliminan todos los registros DNS dentro de la zona. No se puede deshacer esta operación. Si la zona DNS está en uso, los servicios con la zona provocarán un error cuando se elimina la zona.
>
>Para protegerse contra la eliminación accidental de zona, consulte [Proteger registros y zonas DNS](dns-protect-zones-recordsets.md).


Utilice una de las dos opciones siguientes para eliminar una zona DNS:

### <a name="specify-the-zone-using-the-zone-name-and-resource-group-name"></a>Especificar la zona con la utilización del nombre de zona y el nombre del grupo de recursos

```powershell
Remove-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup
```

### <a name="specify-the-zone-using-a-zone-object"></a>Especificar la zona utilizando un objeto $zone

Puede especificar la zona que desee eliminar mediante un objeto `$zone` devuelto por `Get-AzureRmDnsZone`.

```powershell
$zone = Get-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup
Remove-AzureRmDnsZone -Zone $zone
```

También se puede canalizar el objeto de la zona en lugar de pasarlo como parámetro:

```powershell
Get-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup | Remove-AzureRmDnsZone

```

Como con `Set-AzureRmDnsZone`, especificar la zona utilizando un objeto `$zone` permite realizar las comprobaciones de ETag para asegurarse de que no se eliminan los cambios simultáneos. Use el modificador `-Overwrite` para suprimir estas comprobaciones.

## <a name="confirmation-prompts"></a>Mensajes de confirmación

Los cmdlets `New-AzureRmDnsZone`, `Set-AzureRmDnsZone` y `Remove-AzureRmDnsZone` todos admiten mensajes de confirmación.

Ambos `New-AzureRmDnsZone` y `Set-AzureRmDnsZone` piden confirmación si la variable de preferencia de PowerShell `$ConfirmPreference` tiene un valor de `Medium` o inferior. Debido al impacto potencialmente alto que puede tener la eliminación de una zona DNS, el cmdlet `Remove-AzureRmDnsZone` pide confirmación si la variable de PowerShell `$ConfirmPreference` tiene cualquier valor distinto de `None`.

Puesto que el valor predeterminado para `$ConfirmPreference` es `High`, solo `Remove-AzureRmDnsZone` pide confirmación de forma predeterminada.

Puede invalidar el valor actual de `$ConfirmPreference` mediante el parámetro `-Confirm`. Si especifica `-Confirm` o `-Confirm:$True`, el cmdlet solicita confirmación antes de ejecutarse. Si especifica `-Confirm:$False`, el cmdlet no solicita confirmación.

Para más información sobre `-Confirm` y `$ConfirmPreference`, consulte [About Preference Variables](/powershell/module/microsoft.powershell.core/about/about_preference_variables) (Acerca de las variables de preferencias).

## <a name="next-steps"></a>Pasos siguientes

En esta guía se explica cómo [administrar conjuntos de registros y registros](dns-operations-recordsets.md) en la zona DNS.
<br>
Aprenda a cómo [delegar el dominio a Azure DNS](dns-domain-delegation.md).
<br>
Revise la [documentación de referencia de PowerShell para Azure DNS](/powershell/module/azurerm.dns).

