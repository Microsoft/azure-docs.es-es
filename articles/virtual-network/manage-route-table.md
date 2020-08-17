---
title: Creación, modificación o eliminación de una tabla de rutas de Azure
titlesuffix: Azure Virtual Network
description: Aprenda dónde encontrar información sobre el enrutamiento del tráfico de red virtual y cómo crear, cambiar o eliminar una tabla de rutas.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/19/2020
ms.author: kumud
ms.openlocfilehash: 664e3851370be812cd7a0e58cf9beb1fddb5d991
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87290996"
---
# <a name="create-change-or-delete-a-route-table"></a>Creación, modificación o eliminación de una tabla de rutas

Azure enruta automáticamente el tráfico entre redes locales, las redes virtuales y las subredes de Azure. Si desea cambiar algún enrutamiento predeterminado de Azure, debe crear una tabla de rutas. Si no está familiarizado con el enrutamiento en redes virtuales, puede obtener más información al respecto en [Enrutamiento del tráfico de redes virtuales](virtual-networks-udr-overview.md) o siguiendo un [tutorial](tutorial-create-route-table-portal.md).

## <a name="before-you-begin"></a>Antes de empezar

Si no tiene ninguna, configure una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). Después, complete una de estas tareas antes de iniciar los pasos de cualquiera de las secciones de este artículo:

- **Usuarios de Portal**: Inicie sesión en [Azure Portal](https://portal.azure.com) con su cuenta de Azure.

- **Usuarios de PowerShell**: ejecute los comandos en [Azure Cloud Shell](https://shell.azure.com/powershell), o bien ejecute PowerShell en el equipo. Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. En la pestaña del explorador Azure Cloud Shell, busque la lista desplegable **Seleccionar entorno** y, después, elija **PowerShell** si aún no está seleccionado.

    Si ejecuta PowerShell en el entorno local, use la versión del módulo de Azure PowerShell 1.0.0 o una posterior. Ejecute `Get-Module -ListAvailable Az.Network` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Ejecute también `Connect-AzAccount` para crear una conexión con Azure.

- **Usuarios de la interfaz de la línea de comandos (CLI) de Azure**: ejecute los comandos en [Azure Cloud Shell](https://shell.azure.com/bash), o bien ejecute la CLI en el equipo. Use la versión 2.0.31 de la CLI de Azure o una posterior si ejecuta la CLI de Azure en el entorno local. Ejecute `az --version` para buscar la versión instalada. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Ejecute también `az login` para crear una conexión con Azure.

La cuenta en la que inicia sesión o con la que se conecta a Azure debe tener asignado el [rol Colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o un [rol personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones apropiadas en [Permisos](#permissions).

## <a name="create-a-route-table"></a>Creación de una tabla de rutas

Existe un límite en cuanto a la cantidad de tablas de rutas que se pueden crear por suscripción y ubicación de Azure. Para más información, vea [Límites de redes - Azure Resource Manager](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

1. En el menú [Azure Portal](https://portal.azure.com) o en la página **Inicio**, seleccione **Crear un recurso**.

1. En el cuadro de búsqueda, escriba *Tabla de enrutamiento*. Cuando **Tabla de enrutamiento** aparezca en los resultados de la búsqueda, selecciónelo.

1. En la página **Tabla de enrutamiento**, seleccione **Crear**.

1. En el cuadro de diálogo **Crear tabla de rutas**:

    1. Escriba un **Nombre** para la tabla.
    1. Elija la **suscripción**.
    1. Seleccione un **Grupo de recursos** existente o seleccione **Crear nuevo** para crear uno desde cero.
    1. Elija una **ubicación**.
    1. Si va a asociar la tabla de rutas a una subred de una red virtual conectada a la red local por medio de una puerta de enlace VPN, y no quiere que las rutas locales se propaguen a las interfaces de red de la subred, establezca **Propagación de rutas de puerta de enlace de red virtual** en **Deshabilitado**.

1. Seleccione **Crear** para crear la tabla de rutas.

### <a name="create-route-table---commands"></a>Creación de una tabla de rutas: comandos

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network route-table create](/cli/azure/network/route-table#az-network-route-table-create) |
| PowerShell | [New-AzRouteTable](/powershell/module/az.network/new-azroutetable) |

## <a name="view-route-tables"></a>Vista de las tablas de rutas

Vaya a [Azure Portal](https://portal.azure.com) para administrar la red virtual. Busque y seleccione **Tablas de rutas**. Aparecen las tablas de rutas que existen en la suscripción.

### <a name="view-route-table---commands"></a>Vista de una tabla de rutas: comandos

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network route-table list](/cli/azure/network/route-table#az-network-route-table-list) |
| PowerShell | [Get-AzRouteTable](/powershell/module/az.network/get-azroutetable) |

## <a name="view-details-of-a-route-table"></a>Vista de los detalles de una tabla de rutas

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar la red virtual. Busque y seleccione **Tablas de rutas**.

1. En la lista de tablas de rutas, elija aquella cuyos detalles quiera ver.

1. En la página de la tabla de rutas, en **Configuración**, vea las **Rutas** de esa tabla de rutas y las **subredes** con las que dicha la tabla de rutas está asociada.

Para más información sobre la configuración común de Azure, consulte la información siguiente:

- [Registro de actividad](../azure-monitor/platform/platform-logs-overview.md)
- [Control de acceso (IAM)](../role-based-access-control/overview.md)
- [Etiquetas](../azure-resource-manager/management/tag-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Bloqueos](../azure-resource-manager/management/lock-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Script de Automation](../azure-resource-manager/templates/export-template-portal.md)

### <a name="view-details-of-route-table---commands"></a>Vista de los detalles de una tabla de rutas: comandos

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network route-table show](/cli/azure/network/route-table#az-network-route-table-show) |
| PowerShell | [Get-AzRouteTable](/powershell/module/az.network/get-azroutetable) |

## <a name="change-a-route-table"></a>Modificación de una tabla de rutas

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar la red virtual. Busque y seleccione **Tablas de rutas**.

1. En la lista de tablas de rutas, elija aquella que quiera cambiar.

Los cambios más habituales son [agregar](#create-a-route) rutas, [quitar](#delete-a-route) rutas, [asociar](#associate-a-route-table-to-a-subnet) tablas de rutas a subredes o [desasociar](#dissociate-a-route-table-from-a-subnet) tablas de rutas de subredes.

### <a name="change-a-route-table---commands"></a>Modificación de una tabla de rutas: comandos

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network route-table update](/cli/azure/network/route-table#az-network-route-table-update) |
| PowerShell | [Set-AzRouteTable](/powershell/module/az.network/set-azroutetable) |

## <a name="associate-a-route-table-to-a-subnet"></a>Asociación de una tabla de rutas a una subred

Opcionalmente, puede asociar una tabla de rutas a una subred. Una tabla de rutas se puede asociar a varias subredes o a ninguna. Como las tablas de rutas no se asocian a redes virtuales, debe asociar una tabla de rutas a cada subred con la que quiera asociarla. Azure enruta todo el tráfico que sale de la subred según las rutas que se crearon dentro de las tablas de rutas, las [rutas predeterminadas](virtual-networks-udr-overview.md#default) y las rutas propagadas desde una red local, si la red virtual está conectada a una puerta de enlace de red virtual de Azure (ExpressRoute o VPN). Solo puede asociar una tabla de rutas a las subredes de las redes virtuales que existen en la misma suscripción y ubicación de Azure de la tabla de rutas.

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar la red virtual. Busque y seleccione **Redes virtuales**.

1. En la lista de redes virtuales, seleccione la red virtual que contiene la subred a la que quiera asociar una tabla de rutas.

1. En la barra de menús de la red virtual, elija **Subredes**.

1. Seleccione la subred a la que desea asociar la tabla de rutas.

1. En **Tabla de rutas**, seleccione la tabla de rutas a la que quiera asociar la subred.

1. Seleccione **Guardar**.

Si la red virtual está conectada a una instancia de Azure VPN Gateway, no asocie una tabla de rutas a la [subred de la puerta de enlace](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md?toc=%2fazure%2fvirtual-network%2ftoc.json#gwsub) que incluya una ruta con un destino *0.0.0.0/0*. Si lo hace, puede que la puerta de enlace no funcione correctamente. Para más información sobre cómo usar *0.0.0.0/0* en una ruta, vea [Enrutamiento del tráfico de redes virtuales](virtual-networks-udr-overview.md#default-route).

### <a name="associate-a-route-table---commands"></a>Asociación de una tabla de rutas: comandos

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network vnet subnet update](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-update) |
| PowerShell | [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) |

## <a name="dissociate-a-route-table-from-a-subnet"></a>Desasociación de una tabla de rutas de una subred

Cuando desasocia una tabla de rutas de una subred, Azure enruta el tráfico según sus [rutas predeterminadas](virtual-networks-udr-overview.md#default).

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar la red virtual. Busque y seleccione **Redes virtuales**.

1. En la lista de redes virtuales, seleccione la red virtual que contiene la subred de la que quiera desasociar una tabla de rutas.

1. En la barra de menús de la red virtual, elija **Subredes**.

1. Seleccione la subred de la que desea desasociar la tabla de rutas.

1. En **Tabla de rutas**, seleccione **Ninguno**.

1. Seleccione **Guardar**.

### <a name="dissociate-a-route-table---commands"></a>Desasociación de una tabla de rutas: comandos

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network vnet subnet update](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-update) |
| PowerShell | [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) |

## <a name="delete-a-route-table"></a>Eliminación de una tabla de rutas

Una tabla de rutas que está asociada a una subred no se puede eliminar. [Desasocie](#dissociate-a-route-table-from-a-subnet) una tabla de rutas de todas las subredes antes de intentar eliminarla.

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar la tabla de rutas. Busque y seleccione **Tablas de rutas**.

1. En la lista de tablas de rutas, elija aquella que quiera eliminar.

1. Seleccione **Eliminar** y, después, seleccione **Sí** en el cuadro de diálogo de confirmación.

### <a name="delete-a-route-table---commands"></a>Eliminación de una tabla de rutas: comandos

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network route-table delete](/cli/azure/network/route-table#az-network-route-table-delete) |
| PowerShell | [Remove-AzRouteTable](/powershell/module/az.network/remove-azroutetable) |

## <a name="create-a-route"></a>Creación de una ruta

Existe un límite en cuanto a la cantidad de rutas por tabla de rutas que se puede crear por suscripción y ubicación de Azure. Para más información, vea [Límites de redes - Azure Resource Manager](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar la tabla de rutas. Busque y seleccione **Tablas de rutas**.

1. En la lista de tablas de rutas, elija aquella a la que quiera agregar una ruta.

1. En la barra de menús de la tabla de rutas, seleccione **rutas** > **Agregar**.

1. Escriba un **Nombre de ruta** único para la ruta dentro de la tabla de rutas.

1. Escriba el **Prefijo de dirección**, en la notación de Enrutamiento de interdominios sin clases (CIDR), al que quiera enrutar el tráfico. El prefijo no se puede duplicar en más de una ruta dentro de la tabla de rutas, aunque puede estar dentro de otro prefijo. Por ejemplo, si definió *10.0.0.0/16* como prefijo en una ruta, puede seguir definiendo otra ruta con el prefijo de dirección *10.0.0.0/22*. Azure selecciona una ruta para el tráfico en función de la coincidencia de prefijo más larga. Para más información, vea [Selección de rutas por parte de Azure](virtual-networks-udr-overview.md#how-azure-selects-a-route).

1. Seleccione un **Tipo del próximo salto**. Para más información sobre los tipos de próximo salto, vea [Enrutamiento del tráfico de redes virtuales](virtual-networks-udr-overview.md).

1. Si seleccionó un **Tipo del próximo salto** de **Aplicación virtual**, escriba una dirección IP en **Dirección del próximo salto**.

1. Seleccione **Aceptar**.

### <a name="create-a-route---commands"></a>Creación de una ruta: comandos

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network route-table route create](/cli/azure/network/route-table/route#az-network-route-table-route-create) |
| PowerShell | [New-AzRouteConfig](/powershell/module/az.network/new-azrouteconfig) |

## <a name="view-routes"></a>Vista de las rutas

Una tabla de rutas puede contener varias rutas o ninguna. Para más detalles sobre la información que aparece al ver las rutas, vea [Enrutamiento del tráfico de redes virtuales](virtual-networks-udr-overview.md).

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar la tabla de rutas. Busque y seleccione **Tablas de rutas**.

1. En la lista de tablas de rutas, elija aquella cuyas rutas quiera ver.

1. En la barra de menús de la tabla de rutas, seleccione **Rutas** para ver la lista de rutas.

### <a name="view-routes---commands"></a>Vista de las rutas: comandos

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network route-table route list](/cli/azure/network/route-table/route#az-network-route-table-route-list) |
| PowerShell | [Get-AzRouteConfig](/powershell/module/az.network/get-azrouteconfig) |

## <a name="view-details-of-a-route"></a>Vista de los detalles de una ruta

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar la tabla de rutas. Busque y seleccione **Tablas de rutas**.

1. En la lista de tablas de rutas, elija aquella que contenga la ruta cuyos detalles quiera ver.

1. En la barra de menús de la tabla de rutas, seleccione **Rutas** para ver la lista de rutas.

1. Seleccione la ruta cuyos detalles quiere ver.

### <a name="view-details-of-a-route---commands"></a>Vista de los detalles de una ruta: comandos

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network route-table route show](/cli/azure/network/route-table/route#az-network-route-table-route-show) |
| PowerShell | [Get-AzRouteConfig](/powershell/module/az.network/get-azrouteconfig) |

## <a name="change-a-route"></a>Modificación de una ruta

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar la tabla de rutas. Busque y seleccione **Tablas de rutas**.

1. En la lista de tablas de rutas, elija aquella que contenga la ruta que quiera cambiar.

1. En la barra de menús de la tabla de rutas, seleccione **Rutas** para ver la lista de rutas.

1. Seleccione la ruta que quiera cambiar.

1. Cambie la configuración existente a la configuración nueva y, luego, seleccione **Guardar**.

### <a name="change-a-route---commands"></a>Modificación de una ruta: comandos

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network route-table route update](/cli/azure/network/route-table/route#az-network-route-table-route-update) |
| PowerShell | [Set-AzRouteConfig](/powershell/module/az.network/set-azrouteconfig) |

## <a name="delete-a-route"></a>Eliminación de una ruta

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar la tabla de rutas. Busque y seleccione **Tablas de rutas**.

1. En la lista de tablas de rutas, elija aquella que contenga la ruta que quiera eliminar.

1. En la barra de menús de la tabla de rutas, seleccione **Rutas** para ver la lista de rutas.

1. Seleccione la ruta que quiera eliminar.

1. Seleccione **Eliminar** y, después, seleccione **Sí** en el cuadro de diálogo de confirmación.

### <a name="delete-a-route---commands"></a>Eliminación de una ruta: comandos

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network route-table route delete](/cli/azure/network/route-table/route#az-network-route-table-route-delete) |
| PowerShell | [Remove-AzRouteConfig](/powershell/module/az.network/remove-azrouteconfig) |

## <a name="view-effective-routes"></a>Vista de las rutas eficaces

Las rutas eficaces de cada interfaz de red conectada a una máquina virtual son una combinación de las tablas de rutas que creó, las rutas predeterminadas de Azure y cualquier ruta propagada desde las redes locales por medio del Protocolo de puerta de enlace de borde (BGP) a través de una puerta de enlace de red virtual de Azure. Conocer las rutas eficaces de una interfaz de red resulta útil para solucionar los problemas con el enrutamiento. Puede ver las rutas eficaces de cualquier interfaz de red conectada a una máquina virtual en ejecución.

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar sus máquinas virtuales. Busque y seleccione **Máquinas virtuales**.

1. En la lista de máquinas virtuales, seleccione aquella cuyas rutas efectivas quiera ver.

1. En la barra de menús de la máquina virtual, elija **Redes**.

1. Seleccione el nombre de una interfaz de red.

1. En la barra de menús de la interfaz de red, seleccione **Rutas eficaces**.

1. Revise la lista de las rutas eficaces para ver si existe la ruta correcta a la que quiere enrutar el tráfico. En [Enrutamiento del tráfico de redes virtuales](virtual-networks-udr-overview.md) encontrará más información sobre los tipos de próximo salto que figuran en esta lista.

### <a name="view-effective-routes---commands"></a>Vista de las rutas efectivas: comandos

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network nic show-effective-route-table](/cli/azure/network/nic#az-network-nic-show-effective-route-table) |
| PowerShell | [Get-AzEffectiveRouteTable](/powershell/module/az.network/get-azeffectiveroutetable) |

## <a name="validate-routing-between-two-endpoints"></a>Validación del enrutamiento entre dos puntos de conexión

Puede determinar el tipo de próximo paso entre una máquina virtual y la dirección IP de otro recurso de Azure, un recurso local o un recurso de Internet. Determinar el enrutamiento de Azure resulta útil para solucionar los problemas con el enrutamiento. Para completar esta tarea, debe haber una instancia de Network Watcher. Si no tiene una instancia de Network Watcher, complete los pasos descritos en [Creación de una instancia de Network Watcher](../network-watcher/network-watcher-create.md?toc=%2fazure%2fvirtual-network%2ftoc.json) para crear una.

1. Vaya a [Azure Portal](https://portal.azure.com) para sus instancias de Network Watcher. Busque y seleccione **Network Watcher**.

1. En la barra de menús de Network Watcher, seleccione **Próximo salto**.

1. En la página **Network Watcher | Próximo salto**:

    1. Seleccione su **Suscripción** y el **Grupo de recursos** de la máquina virtual de origen desde la que quiere validar el enrutamiento.

    1. Seleccione la **Máquina virtual** y la **Interfaz de red** que está conectada a la máquina virtual.
    
    1. Escriba una **Dirección IP de origen** asignada a la interfaz de red desde la que quiera validar el enrutamiento.

    1. Escriba una **Dirección IP de destino** hacia la que quiera validar el enrutamiento.

1. Seleccione **Próximo paso**.

Tras una breve espera, Azure le indica el tipo de próximo paso y el identificador de la ruta que enrutó el tráfico. En [Enrutamiento del tráfico de redes virtuales](virtual-networks-udr-overview.md) encontrará más información sobre los tipos de próximo salto que se devuelven.

### <a name="validate-routing-between-two-endpoints---commands"></a>Validación del enrutamiento entre dos puntos de conexión: comandos

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network watcher show-next-hop](/cli/azure/network/watcher#az-network-watcher-show-next-hop) |
| PowerShell | [Get-AzNetworkWatcherNextHop](/powershell/module/az.network/get-aznetworkwatchernexthop) |

## <a name="permissions"></a>Permisos

Para realizar tareas en rutas y en tablas de rutas, la cuenta debe tener asignado el [rol Colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o un [rol personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones adecuadas que figuran en la siguiente tabla:

| Acción                                                          |   Nombre                                                  |
|--------------------------------------------------------------   |   -------------------------------------------           |
| Microsoft.Network/routeTables/read                              |   Lectura de una tabla de rutas                                    |
| Microsoft.Network/routeTables/write                             |   Creación o actualización de una tabla de rutas                        |
| Microsoft.Network/routeTables/delete                            |   Eliminación de una tabla de rutas                                  |
| Microsoft.Network/routeTables/join/action                       |   Asociación de una tabla de rutas a una subred                   |
| Microsoft.Network/routeTables/routes/read                       |   Lectura de una ruta                                          |
| Microsoft.Network/routeTables/routes/write                      |   Creación o actualización de una ruta                              |
| Microsoft.Network/routeTables/routes/delete                     |   Eliminación de una ruta                                        |
| Microsoft.Network/networkInterfaces/effectiveRouteTable/action  |   Obtención de la tabla de rutas eficaces de una interfaz de red |
| Microsoft.Network/networkWatchers/nextHop/action                |   Obtención del próximo salto de una VM                           |

## <a name="next-steps"></a>Pasos siguientes

- Crear una tabla de rutas usando scripts de ejemplo de [PowerShell](powershell-samples.md) o de la [CLI de Azure](cli-samples.md) o [plantillas de Azure Resource Manager](template-samples.md)
- Crear y asignar [definiciones de Azure Policy](policy-samples.md) para redes virtuales
