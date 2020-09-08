---
title: Cuotas de vCPU de Azure
description: Obtenga información acerca de las cuotas de vCPU para Azure Virtual Machines.
author: cynthn
ms.service: virtual-machines
ms.subservice: sizes
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 05/31/2018
ms.author: cynthn
ms.openlocfilehash: b00d84e1c2dc8f104046fe24360d79e6952dfacc
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88950296"
---
# <a name="check-vcpu-quotas-using-azure-powershell"></a>Comprobación de cuotas de vCPU mediante Azure PowerShell

Las cuotas de vCPU para máquinas virtuales y conjuntos de escalado de máquinas virtuales se organizan en dos niveles para cada suscripción en cada región. El primer nivel es el número de vCPU regionales totales y el segundo son los núcleos de las diversas familias de tamaños de máquina virtual, como las vCPU de la serie D estándar. Siempre que se implemente una máquina virtual nueva, las vCPU de dicha máquina virtual no deben exceder la cuota de vCPU de la familia de tamaños de máquina virtual o el total de la cuota de vCPU regional. Si se supera cualquiera de esas dos cuotas, no se permitirá la implementación de la máquina virtual. También hay una cuota para el número total de máquinas virtuales en la región. Se pueden ver los detalles de cada una de estas cuotas en la sección **Uso y cuotas** de la página **Suscripción** en [Azure Portal](https://portal.azure.com), o bien puede consultar los valores mediante PowerShell.

> [!NOTE]
> La cuota se calcula en función del número total de núcleos en uso tanto los asignados como los desasignados. Si necesita más núcleos, [solicite un aumento de la cuota](../../azure-portal/supportability/resource-manager-core-quotas-request.md) o elimine las máquinas virtuales que ya no sean necesarias. 
 
## <a name="check-usage"></a>Comprobación del uso

Puede usar el cmdlet [Get-AzVMUsage](/powershell/module/az.compute/get-azvmusage) para comprobar el uso de la cuota.

```azurepowershell-interactive
Get-AzVMUsage -Location "East US"
```

El resultado será similar al siguiente:

```
Name                             Current Value Limit  Unit
----                             ------------- -----  ----
Availability Sets                            0  2000 Count
Total Regional vCPUs                         4   260 Count
Virtual Machines                             4 10000 Count
Virtual Machine Scale Sets                   1  2000 Count
Standard B Family vCPUs                      1    10 Count
Standard DSv2 Family vCPUs                   1   100 Count
Standard Dv2 Family vCPUs                    2   100 Count
Basic A Family vCPUs                         0   100 Count
Standard A0-A7 Family vCPUs                  0   250 Count
Standard A8-A11 Family vCPUs                 0   100 Count
Standard D Family vCPUs                      0   100 Count
Standard G Family vCPUs                      0   100 Count
Standard DS Family vCPUs                     0   100 Count
Standard GS Family vCPUs                     0   100 Count
Standard F Family vCPUs                      0   100 Count
Standard FS Family vCPUs                     0   100 Count
Standard NV Family vCPUs                     0    24 Count
Standard NC Family vCPUs                     0    48 Count
Standard H Family vCPUs                      0     8 Count
Standard Av2 Family vCPUs                    0   100 Count
Standard LS Family vCPUs                     0   100 Count
Standard Dv2 Promo Family vCPUs              0   100 Count
Standard DSv2 Promo Family vCPUs             0   100 Count
Standard MS Family vCPUs                     0     0 Count
Standard Dv3 Family vCPUs                    0   100 Count
Standard DSv3 Family vCPUs                   0   100 Count
Standard Ev3 Family vCPUs                    0   100 Count
Standard ESv3 Family vCPUs                   0   100 Count
Standard FSv2 Family vCPUs                   0   100 Count
Standard ND Family vCPUs                     0     0 Count
Standard NCv2 Family vCPUs                   0     0 Count
Standard NCv3 Family vCPUs                   0     0 Count
Standard LSv2 Family vCPUs                   0     0 Count
Standard Storage Managed Disks               2 10000 Count
Premium Storage Managed Disks                1 10000 Count
```


## <a name="reserved-vm-instances"></a>Instancias reservadas de máquina virtual
Las instancias reservadas de máquina virtual, cuyo ámbito es una sola suscripción sin tamaño de máquina virtual flexible, agregarán un aspecto nuevo a las cuotas de vCPU. Estos valores describen el número de instancias del tamaño indicado que deben poderse implementar en la suscripción. Actúan como un marcador de posición en el sistema de cuotas para asegurarse de que esa cuota esté reservada y que las instancias reservadas de máquina virtual puedan implementarse en la suscripción. Por ejemplo, si una suscripción específica tiene diez instancias reservadas de máquina virtual Standard_D1, el límite de uso para las instancias reservadas de máquina virtual Standard_D1 será de diez. Esto hará que Azure se asegure de que siempre hay al menos 10 vCPU disponibles en el total de la cuota de vCPU regional que se utilizará para instancias de Standard_D1 y que hay al menos 10 vCPU disponibles en la cuota de vCPU de la Familia D estándar que se utilizará para las instancias de Standard_D1.

Si es necesario un incremento de cuota para adquirir una instancia reservada de suscripción única, puede [solicitarlo](../../azure-portal/supportability/resource-manager-core-quotas-request.md) en su suscripción.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre facturación y cuotas, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md?toc=/azure/billing/TOC.json).
