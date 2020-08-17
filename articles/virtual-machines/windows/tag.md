---
title: Etiquetado de un recurso de máquina virtual Windows en Azure
description: Infórmese acerca del etiquetado de una máquina virtual Azure creada con el modelo de implementación de Resource Manager.
author: mmccrory
ms.service: virtual-machines-windows
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 07/05/2016
ms.author: memccror
ms.openlocfilehash: 84fc6e6fee0b4a85b909c9b43a17381b31f39abf
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87291982"
---
# <a name="how-to-tag-a-windows-virtual-machine-in-azure"></a>Etiquetado de una máquina virtual en Azure
En este artículo se describen diferentes maneras de etiquetar una máquina virtual Windows en Azure por medio del modelo de implementación de Resource Manager. Las etiquetas son pares clave-valor definidos por el usuario que se pueden colocar directamente en un recurso o un grupo de recursos. Actualmente, Azure admite un máximo de 50 etiquetas por recurso y grupo de recursos. Las etiquetas se pueden colocar en un recurso en el momento de su creación, o bien se pueden agregar a un recurso existente. Tenga en cuenta que las etiquetas solo son compatibles con los recursos creados mediante el modelo de implementación de Resource Manager. Si desea etiquetar una máquina virtual Linux, consulte [How to tag a Linux virtual machine in Azure](../linux/tag.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)(Etiquetado de una máquina virtual Linux en Azure).

[!INCLUDE [virtual-machines-common-tag](../../../includes/virtual-machines-common-tag.md)]

## <a name="tagging-with-powershell"></a>Etiquetado con PowerShell
Para crear, agregar y eliminar etiquetas a través de PowerShell, primero es preciso configurar el [entorno de PowerShell con Administrador de recursos de Azure][PowerShell environment with Azure Resource Manager]. Una vez completada la configuración, puede colocar etiquetas en los recursos de Compute, Network y Storage tanto al crear el recurso a través de PowerShell como después de haberlo creado. En este artículo se centrará en la visualización y edición de etiquetas en máquinas virtuales.

 

En primer lugar, navegue a una máquina virtual a través del cmdlet `Get-AzVM` .

```azurepowershell
PS C:\> Get-AzVM -ResourceGroupName "MyResourceGroup" -Name "MyTestVM"
```

Si la máquina virtual ya contiene etiquetas, verá todas ellas en el recurso:

```json
Tags : {
        "Application": "MyApp1",
        "Created By": "MyName",
        "Department": "MyDepartment",
        "Environment": "Production"
        }
```

Si desea agregar etiquetas a través de PowerShell, puede usar el comando `Set-AzResource` . Tenga en cuenta que si actualiza las etiquetas a través de PowerShell, se actualizan todas ellas en conjunto. Por tanto, si va a agregar una etiqueta a un recurso que ya tiene etiquetas, tendrá que incluir todas las etiquetas que desea colocar en el recurso. A continuación se muestra un ejemplo de cómo agregar etiquetas adicionales a un recurso mediante los cmdlets de PowerShell.

Este primer cmdlet establece todas las etiquetas colocadas en *MyTestVM* en la variable *$tags*, para lo que emplea las propiedades `Get-AzResource` y `Tags`.

```azurepowershell
PS C:\> $tags = (Get-AzResource -ResourceGroupName MyResourceGroup -Name MyTestVM).Tags
```

El segundo comando muestra las etiquetas de la variable especificada.

```azurepowershell
PS C:\> $tags

Key           Value
----          -----
Department    MyDepartment
Application   MyApp1
Created By    MyName
Environment   Production
```

El tercer comando agrega una etiqueta adicional a la variable *tags* . Observe el uso de **+=** para anexar el nuevo par de clave-valor a la lista de *tags* .

```azurepowershell
PS C:\> $tags += @{Location="MyLocation"}
```

El cuarto comando establece todas las etiquetas definidas en la variable *tags* en el recurso especificado. En este caso, es MyTestVM.

```azurepowershell
PS C:\> Set-AzResource -ResourceGroupName MyResourceGroup -Name MyTestVM -ResourceType "Microsoft.Compute/VirtualMachines" -Tag $tags
```

El quinto comando muestra todas las etiquetas en el recurso. Como puede ver, *Location* está definido como una etiqueta, con *MyLocation* como valor.

```azurepowershell
PS C:\> (Get-AzResource -ResourceGroupName MyResourceGroup -Name MyTestVM).Tags

Key           Value
----          -----
Department    MyDepartment
Application   MyApp1
Created By    MyName
Environment   Production
Location      MyLocation
```

Para más información sobre el etiquetado a través de PowerShell, consulte los [cmdlets de recursos de Azure][Azure Resource Cmdlets].

[!INCLUDE [virtual-machines-common-tag-usage](../../../includes/virtual-machines-common-tag-usage.md)]

## <a name="next-steps"></a>Pasos siguientes
* Para más información sobre el etiquetado de los recursos de Azure, consulte [Información general de Azure Resource Manager][Azure Resource Manager Overview] y [Uso de etiquetas para organizar los recursos de Azure][Using Tags to organize your Azure Resources].
* Para ver cómo las etiquetas pueden ayudarle a administrar el uso de los recursos de Azure, consulte [Comprender la factura de Microsoft Azure][Understanding your Azure Bill] y [Obtención de información sobre el consumo de recursos de Microsoft Azure][Gain insights into your Microsoft Azure resource consumption].

[PowerShell environment with Azure Resource Manager]: ../../azure-resource-manager/management/manage-resources-powershell.md
[Azure Resource Cmdlets]: /powershell/module/az.resources/
[Azure Resource Manager Overview]: ../../azure-resource-manager/management/overview.md
[Using Tags to organize your Azure Resources]: ../../azure-resource-manager/management/tag-resources.md
[Understanding your Azure Bill]:../../cost-management-billing/understand/review-individual-bill.md
[Gain insights into your Microsoft Azure resource consumption]:../../cost-management-billing/manage/usage-rate-card-overview.md
