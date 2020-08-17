---
title: Grupos de IP en la directiva de Azure Firewall
description: Los grupos IP permiten agrupar y administrar direcciones IP en las reglas de Azure Firewall.
services: firewall-manager
author: vhorne
ms.service: firewall-manager
ms.topic: conceptual
ms.date: 07/30/2020
ms.author: victorh
ms.openlocfilehash: 5192ecb31c71364bdf1301b13da0b0742625d44f
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87460140"
---
# <a name="ip-groups-in-azure-firewall-policy"></a>Grupos de IP en Azure Firewall

Los grupos de IP permiten agrupar y administrar direcciones IP en una directiva de Azure Firewall de las siguientes maneras:

- Como un tipo de origen en las reglas de DNAT
- Como una dirección de origen o destino en las reglas de red
- Como un tipo de origen en las reglas de aplicación


Un grupo IP puede tener una sola dirección IP, varias direcciones IP o uno o varios intervalos de direcciones IP.

Los grupos de IP se puede reutilizar en reglas de DNAT, red y aplicación de Azure Firewall para varios firewalls entre regiones y suscripciones de Azure. Los nombres de grupo deben ser únicos. Puede configurar un grupo de IP en Azure Portal, la CLI de Azure o la API REST. Para ayudarle a comenzar, se proporciona una plantilla de ejemplo.

## <a name="sample-format"></a>Formato de ejemplo

Los siguientes ejemplos de formato de dirección IPv4 son válidos para su uso en IP Groups:

- Dirección única: 10.0.0.0
- Notación CIDR: 10.1.0.0/32
- Intervalo de direcciones: 10.2.0.0-10.2.0.31

## <a name="create-an-ip-group"></a>Creación de un grupo de IP

Un grupo de IP se puede crear mediante Azure Portal, la CLI de Azure o la API REST. Para obtener más información, consulte [Creación de grupos de IP](../firewall/create-ip-group.md).

## <a name="browse-ip-groups"></a>Acceso a IP Groups
1. En la barra de búsqueda de Azure Portal, escriba **IP Groups** y selecciónelo. Puede ver la lista de los grupos de IP o puede seleccionar **Agregar** para crear uno.
2. Seleccione un grupo de IP para abrir la página de información general. Puede editar, agregar o eliminar direcciones IP o grupos de IP.

   ![Información general de grupos de IP](media/ip-groups/overview.png)

## <a name="manage-an-ip-group"></a>Administración de un grupo de IP

Puede ver todas las direcciones IP del grupo de IP y las reglas o los recursos a él asociados. Para eliminar un grupo de IP, primero debe desasociar el grupo de IP del recurso que lo usa.

1. Para ver o editar las direcciones IP, seleccione **Direcciones IP** en **Configuración** en el panel izquierdo.
2. Para agregar una o varias direcciones IP, seleccione **Agregar direcciones IP**. Se abre la página **Drag or Browse** (Arrastrar o examinar) para que las descargue; también puede escribir las direcciones manualmente.
3.    Seleccione los puntos suspensivos ( **…** ) de la derecha para editar o eliminar las direcciones IP. Para editar o eliminar varias direcciones IP, active las casillas y seleccione **Editar** o **Eliminar** en la parte superior.
4. Por último, puede exportar el archivo en formato CSV.

> [!NOTE]
> Si elimina todas las direcciones IP de un grupo de IP mientras todavía se está usando en una regla, esa regla se omite.


## <a name="use-an-ip-group"></a>Uso de un grupo de IP

Ahora puede seleccionar **Grupo de IP** como **Tipo de origen** o **Tipo de destino** para las direcciones IP al crear una directiva con reglas DNAT, de aplicación o red.

:::image type="content" source="media/ip-groups/firewall-ip-group.png" alt-text="Grupos de IP en Firewall":::

## <a name="ip-address-limits"></a>Límites de direcciones IP

Puede tener un máximo de 100 grupos de IP por firewall con un máximo de 5000 direcciones IP individuales o prefijos IP por cada grupo de IP.

## <a name="related-azure-powershell-cmdlets"></a>Cmdlets de Azure PowerShell relacionados

Se pueden usar los siguientes cmdlets de Azure PowerShell para crear y administrar grupos de IP:

- [New-AzIpGroup](https://docs.microsoft.com/powershell/module/az.network/new-azipgroup?view=azps-3.4.0)
- [Remove-AzIPGroup](https://docs.microsoft.com/powershell/module/az.network/remove-azipgroup?view=azps-3.4.0)
- [Get-AzIpGroup](https://docs.microsoft.com/powershell/module/az.network/get-azipgroup?view=azps-3.4.0)
- [Set-AzIpGroup](https://docs.microsoft.com/powershell/module/az.network/set-azipgroup?view=azps-3.4.0)
- [New-AzFirewallPolicyNetworkRule](https://docs.microsoft.com/powershell/module/az.network/new-azfirewallpolicynetworkrule?view=azps-3.4.0)
- [New-AzFirewallPolicyApplicationRule](https://docs.microsoft.com/powershell/module/az.network/new-azfirewallpolicyapplicationrule?view=azps-3.4.0)
- [New-AzFirewallPolicyNatRule](https://docs.microsoft.com/powershell/module/az.network/new-azfirewallpolicynatrule?view=azps-3.4.0)

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Protección de una red WAN virtual mediante Azure Firewall Manager](secure-cloud-network.md)