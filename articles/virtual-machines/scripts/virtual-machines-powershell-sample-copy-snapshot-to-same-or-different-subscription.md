---
title: 'Copia de una instantánea de un disco administrado en una suscripción (Windows): PowerShell'
description: 'Ejemplo de script de Azure PowerShell: copiar (mover) instantánea de un disco administrado en la misma suscripción o en otra'
services: virtual-machines-windows
documentationcenter: storage
author: ramankumarlive
manager: kavithag
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-windows
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 02/28/2019
ms.author: ramankum
ms.openlocfilehash: a2fd6f8b5f2ba0771449178a2d8d3958329e83eb
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89651599"
---
# <a name="copy-snapshot-of-a-managed-disk-in-same-subscription-or-different-subscription-with-powershell-windows"></a>Copia de una instantánea de un disco administrado en la misma suscripción o en otra con PowerShell (Windows)

Este script copia una instantánea de un disco administrado en la misma suscripción o en otra. Use este script en los escenarios siguientes:

1. Migrar una instantánea de Premium Storage (Premium_LRS) a Standard Storage (Standard_LRS o Standard_ZRS) para reducir el costo.
1. Migrar una instantánea de almacenamiento con redundancia local (Premium_LRS, Standard_LRS) a almacenamiento con redundancia de zona (Standard_ZRS) para beneficiarse de la mayor confiabilidad del almacenamiento ZRS.
1. Mover una instantánea a otra suscripción de la misma región para alargar la retención.

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

 

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/virtual-machine/copy-snapshot-to-same-or-different-subscription/copy-snapshot-to-same-or-different-subscription.ps1 "Copy snapshot")]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear una instantánea en la suscripción de destino mediante el identificador de la instantánea de origen. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzSnapshotConfig](/powershell/module/az.compute/new-azsnapshotconfig) | Crea la configuración de instantánea que se usa para la creación de instantáneas. Incluye el identificador de recurso de la instantánea principal y una ubicación igual a la de la instantánea principal.  |
| [New-AzSnapshot](/powershell/module/az.compute/new-azsnapshot) | Crea una instantánea mediante la configuración de instantánea, el nombre de instantánea y el nombre de grupo de recursos pasados como parámetros. |

## <a name="next-steps"></a>Pasos siguientes

[Crear una máquina virtual a partir de una instantánea](./virtual-machines-windows-powershell-sample-create-vm-from-snapshot.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Encontrará más ejemplos de scripts de Azure PowerShell de máquina virtual en la [documentación sobre máquinas virtuales Windows de Azure](../windows/powershell-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
