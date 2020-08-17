---
title: Incorporación, modificación o eliminación de una subred de red virtual de Azure
titlesuffix: Azure Virtual Network
description: Aprenda dónde encontrar información acerca de redes virtuales y cómo agregar, cambiar o eliminar una subred de red virtual en Azure.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/20/2020
ms.author: kumud
ms.openlocfilehash: 15fe5d6d16948875253d65e70d9d440214a4a2e8
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87286111"
---
# <a name="add-change-or-delete-a-virtual-network-subnet"></a>Incorporación, cambio o eliminación de una subred de red virtual

Obtenga información sobre la incorporación, la modificación o la eliminación de una subred de red virtual. Todos los recursos de Azure implementados en una red virtual se implementan en una subred de esa red virtual. Si no está familiarizado con las redes virtuales, puede obtener más información sobre ellas en la [información general sobre redes virtuales](virtual-networks-overview.md) o siguiendo un [inicio rápido](quick-create-portal.md). Para obtener más información sobre la administración de una red virtual, consulte [Crear, cambiar o eliminar una red virtual](manage-virtual-network.md).

## <a name="before-you-begin"></a>Antes de empezar

Si no tiene ninguna, configure una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). Después, complete una de estas tareas antes de iniciar los pasos de cualquiera de las secciones de este artículo: 

- **Usuarios de Portal**: Inicie sesión en [Azure Portal](https://portal.azure.com) con su cuenta de Azure.

- **Usuarios de PowerShell**: ejecute los comandos en [Azure Cloud Shell](https://shell.azure.com/powershell), o bien ejecute PowerShell en el equipo. Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. En la pestaña del explorador Azure Cloud Shell, busque la lista desplegable **Seleccionar entorno** y, después, elija **PowerShell** si aún no está seleccionado.

    Si ejecuta PowerShell en el entorno local, use la versión del módulo de Azure PowerShell 1.0.0 o una posterior. Ejecute `Get-Module -ListAvailable Az.Network` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Ejecute también `Connect-AzAccount` para crear una conexión con Azure.

- **Usuarios de la interfaz de la línea de comandos (CLI) de Azure**: ejecute los comandos en [Azure Cloud Shell](https://shell.azure.com/bash), o bien ejecute la CLI en el equipo. Use la versión 2.0.31 de la CLI de Azure o una posterior si ejecuta la CLI de Azure en el entorno local. Ejecute `az --version` para buscar la versión instalada. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Además, ejecute `az login` para crear una conexión con Azure.

La cuenta en la que inicia sesión o con la que se conecta a Azure, debe tener asignado el rol de [colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o un [rol personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones apropiadas en [Permisos](#permissions).

## <a name="add-a-subnet"></a>Incorporación de una subred

1. Vaya a [Azure Portal](https://portal.azure.com) para ver las redes virtuales. Busque y seleccione **Redes virtuales**.

2. Seleccione el nombre de la red virtual en la que quiera agregar una subred.

3. En **Configuración**, seleccione **Subredes** > **Subred**.

4. En el cuadro de diálogo **Agregar subred**, especifique valores para la configuración siguiente:

    | Configuración | Descripción |
    | --- | --- |
    | **Nombre** | El nombre debe ser único dentro de la red virtual. Para lograr la máxima compatibilidad con otros servicios de Azure, se recomienda usar una letra como primer carácter del nombre. Por ejemplo, Azure Application Gateway no se implementará en una subred que tiene un nombre que comienza con un número. |
    | **Intervalo de direcciones** | <p>El intervalo debe ser único dentro del espacio de direcciones de la red virtual. Este intervalo no puede superponerse con otros intervalos de direcciones de subred dentro de la red virtual. El espacio de direcciones debe especificarse mediante notación de Enrutamiento de interdominios sin clases (CIDR).</p><p>Por ejemplo, en una red virtual con el espacio de direcciones *10.0.0.0/16*, podría definir el espacio de direcciones de subred *10.0.0.0/22*. El menor intervalo que se puede especificar es */29*, lo que proporciona ocho direcciones IP de subred. De conformidad con el protocolo, Azure reserva la primera y la última dirección de cada subred. Otras tres direcciones están reservadas para el uso del servicio de Azure. Como resultado, al definir una subred con el intervalo de direcciones */29*, se generan tres direcciones IP utilizables en la subred.</p><p>Si planea conectar una red virtual a una puerta de enlace VPN, debe crear una subred de puerta de enlace. Más información sobre las [consideraciones específicas del intervalo de direcciones de las subredes de puerta de enlace](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md?toc=%2fazure%2fvirtual-network%2ftoc.json#gwsub). En determinadas circunstancias se puede cambiar el intervalo de direcciones una vez agregada la subred. Para información sobre cómo cambiar un intervalo de direcciones de subred, consulte [Cambio de configuración de subred](#change-subnet-settings).</p> |
    | **Grupo de seguridad de red** | Para filtrar el tráfico de red entrante y saliente de la subred, puede asociar un grupo de seguridad de red existente a una subred. El grupo de seguridad de red debe existir en la misma suscripción y la misma ubicación que la red virtual. Obtenga más información sobre los [grupos de seguridad de red](security-overview.md) y [cómo crear un grupo de seguridad de red](tutorial-filter-network-traffic.md). |
    | **Tabla de rutas** | Para controlar el enrutamiento del tráfico de red a otras redes, puede asociar una tabla de rutas existente a una subred. La tabla de rutas debe existir en la misma suscripción y la misma ubicación que la red virtual. Obtenga más información sobre el [enrutamiento de Azure](virtual-networks-udr-overview.md) y [cómo crear una tabla de rutas](tutorial-create-route-table-portal.md). |
    | **Puntos de conexión de servicio** | <p>Una subred puede tener uno o más puntos de conexión de servicio habilitados. Para habilitar un punto de conexión de servicio, seleccione el servicio o los servicios para los que desea habilitar los puntos de conexión en la lista de **servicios**. Azure configura la ubicación automáticamente para un punto de conexión. De forma predeterminada, Azure configura los puntos de conexión de servicio de la región de la red virtual. En el caso de Azure Storage, para admitir escenarios de conmutación por error regionales, Azure configura automáticamente los puntos de conexión para las [regiones emparejadas de Azure](../best-practices-availability-paired-regions.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-paired-regions).</p><p>Para quitar un punto de conexión de servicio, anule la selección del servicio para el que desea quitar el punto de conexión. Para obtener más información sobre los puntos de conexión de servicio y sobre los servicios para los que se pueden habilitar, vea [Puntos de conexión de servicio de red virtual](virtual-network-service-endpoints-overview.md). Una vez que habilita un punto de conexión para un servicio, también debe habilitar el acceso de red para la subred de un recurso creado con el servicio. Por ejemplo, si habilita el punto de conexión de servicio para **Microsoft.Storage**, también debe habilitar el acceso de red a todas las cuentas de Azure Storage a las que desea conceder acceso de red. Para habilitar el acceso de red a las subredes para las que está habilitado un punto de conexión de servicio, consulte la documentación correspondiente al servicio individual para el que habilitó el punto de conexión de servicio.</p><p>Para validar que un punto de conexión de servicio está habilitado para una subred, consulte las [rutas efectivas](diagnose-network-routing-problem.md) para cualquier interfaz de red de la subred. Cuando configure un punto de conexión, verá una ruta *predeterminada* con los prefijos de dirección del servicio y el tipo de próximo salto **VirtualNetworkServiceEndpoint**. Para obtener más información sobre el enrutamiento, consulte [Enrutamiento del tráfico de redes virtuales](virtual-networks-udr-overview.md).</p> |
    | **Delegación de subred** | Una subred puede tener una o más delegaciones habilitadas. La delegación de subred proporciona permisos explícitos al servicio para crear los recursos específicos del servicio en la subred con un identificador único durante la implementación del servicio. Para delegar a un servicio, seleccione el servicio al que desea delegar en la lista **Servicios**. |

5. Para agregar la subred a la red virtual que seleccionó, seleccione **Aceptar**.

### <a name="commands"></a>Comandos:

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network vnet subnet create](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-create) |
| PowerShell | [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) |

## <a name="change-subnet-settings"></a>Modificación de la configuración de subred

1. Vaya a [Azure Portal](https://portal.azure.com) para ver las redes virtuales. Busque y seleccione **Redes virtuales**.

2. Seleccione el nombre de la red virtual que contenga la subred que quiere cambiar.

3. En **Configuración**, seleccione **Subredes**.

4. En la lista de subredes, seleccione la subred para la que desea modificar la configuración.

5. En la página de subred, cambie cualquiera de las opciones de configuración siguientes:

    | Configuración | Descripción |
    | --- | --- |
    | **Intervalo de direcciones** | si no hay recursos implementados dentro de la subred, es posible modificar el intervalo de direcciones. Si existe algún recurso en la subred, primero deberá moverlos a otra subred o eliminarlos. Los pasos necesarios para mover o eliminar un recurso varían según el recurso. Para obtener información sobre cómo mover o eliminar los recursos que se encuentran en las subredes, lea la documentación de cada uno de estos tipos de recursos. Consulte cuáles son las restricciones de la configuración **Intervalo de direcciones** en el paso 4 de la sección [Adición de una subred](#add-a-subnet). |
    | **Usuarios** | puede controlar el acceso a la subred con los roles integrados u otros personalizados. Para obtener más información sobre la asignación de roles y usuarios para el acceso a la subred, consulte [Adición de una asignación de roles](../role-based-access-control/role-assignments-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json#add-a-role-assignment). |
    | **Grupo de seguridad de red** y **Tabla de rutas** | Vea el paso 4 de la sección [Adición de una subred](#add-a-subnet). |
    | **Puntos de conexión de servicio** | <p>Consulte los puntos de conexión de servicio del paso 4 de la sección [Adición de una subred](#add-a-subnet). A la hora de habilitar un punto de conexión de servicio para una red existente, asegúrese de que no se esté ejecutando ninguna tarea crítica en ningún recurso de la subred. Los puntos de conexión de servicio cambian las rutas en cada interfaz de red de la subred. Los puntos de conexión de servicio pasan de usar la ruta predeterminada con el prefijo de dirección *0.0.0.0/0* y el tipo de próximo salto *Internet* a usar una ruta nueva con los prefijos de dirección del servicio y el tipo de próximo salto *VirtualNetworkServiceEndpoint*.</p><p>Durante el cambio se puede terminar cualquier conexión TCP que esté abierta. El punto de conexión de servicio no se habilita hasta que los flujos de tráfico al servicio de todas las interfaces de red se actualicen con la nueva ruta. Para obtener más información sobre el enrutamiento, consulte [Enrutamiento del tráfico de redes virtuales](virtual-networks-udr-overview.md).</p> |
    | **Delegación de subred** | Consulte los puntos de conexión de servicio del paso 4 de la sección [Adición de una subred](#add-a-subnet). La delegación de subred se puede modificar a varias delegaciones habilitadas para él o a ninguna. Si un recurso de un servicio ya está implementado en la subred, la delegación de la subred no se puede agregar ni quitar hasta que se quiten todos los recursos del servicio. Para delegar a otro servicio, seleccione el servicio al que desea delegar en la lista **Servicios**. |

6. Seleccione **Guardar**.

### <a name="commands"></a>Comandos:

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network vnet subnet update](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-update) |
| PowerShell | [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) |

## <a name="delete-a-subnet"></a>Eliminación de una subred

Solo se puede eliminar una subred si no tiene recursos. Si hay recursos en la subred, debe eliminarlos para poder eliminar la subred. Los pasos necesarios para eliminar un recurso varían según el recurso. Para obtener información sobre cómo eliminar los recursos que se encuentran en las subredes, lea la documentación de cada uno de estos tipos de recursos.

1. Vaya a [Azure Portal](https://portal.azure.com) para ver las redes virtuales. Busque y seleccione **Redes virtuales**.

2. Seleccione el nombre de la red virtual que contenga la subred que quiere eliminar.

3. En **Configuración**, seleccione **Subredes**.

4. En la lista de subredes, seleccione la subred que quiere eliminar.

5. Seleccione **Eliminar** y, a continuación, seleccione **Sí** en el cuadro de diálogo de confirmación.

### <a name="commands"></a>Comandos:

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network vnet subnet delete](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-delete) |
| PowerShell | [Remove-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/remove-azvirtualnetworksubnetconfig?toc=%2fazure%2fvirtual-network%2ftoc.json) |

## <a name="permissions"></a>Permisos

Para llevar a cabo tareas en subredes, su cuenta debe estar asignada al [rol de colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o a un [rol personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones adecuadas que se muestran en la tabla siguiente:

|Acción                                                                   |   Nombre                                       |
|-----------------------------------------------------------------------  |   -----------------------------------------  |
|Microsoft.Network/virtualNetworks/subnets/read                           |   Leer una subred de red virtual              |
|Microsoft.Network/virtualNetworks/subnets/write                          |   Crear o actualizar una subred de red virtual  |
|Microsoft.Network/virtualNetworks/subnets/delete                         |   Eliminar una subred de red virtual            |
|Microsoft.Network/virtualNetworks/subnets/join/action                    |   Unirse a una red virtual                     |
|Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/action  |   Habilitar un punto de conexión de servicio para una subred     |
|Microsoft.Network/virtualNetworks/subnets/virtualMachines/read           |   Obtener las máquinas virtuales de una subred       |

## <a name="next-steps"></a>Pasos siguientes

- Crear una red virtual y subredes con scripts de ejemplo de [PowerShell](powershell-samples.md) o de la [CLI de Azure](cli-samples.md) o con [plantillas de Azure Resource Manager](template-samples.md)
- Crear y asignar [definiciones de Azure Policy](policy-samples.md) para redes virtuales
