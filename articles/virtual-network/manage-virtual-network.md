---
title: Creación, modificación o eliminación de una red virtual de Azure
titlesuffix: Azure Virtual Network
description: Cree y elimine una red virtual, y cambie ajustes de configuración, como los servidores DNS y los espacios de direcciones IP, en una red virtual existente.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/10/2019
ms.author: kumud
ms.openlocfilehash: 0a35576435780ee43d9f2aa99167b736f90799ab
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87265252"
---
# <a name="create-change-or-delete-a-virtual-network"></a>Crear, cambiar o eliminar una red virtual

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Aprenda a crear y eliminar una red virtual, y a cambiar ajustes de configuración, como los servidores DNS y los espacios de direcciones IP, en una red virtual existente. Si no está familiarizado con las redes virtuales, puede obtener más información sobre ellas en la [información general sobre redes virtuales](virtual-networks-overview.md) o siguiendo un [tutorial](quick-create-portal.md). Una red virtual contiene subredes. Para aprender a crear, cambiar y eliminar subredes, vea [Incorporación, cambio o eliminación de una subred de red virtual](virtual-network-manage-subnet.md).

## <a name="before-you-begin"></a>Antes de empezar

Complete las tareas siguientes antes de seguir los pasos de las secciones de este artículo:

- Si todavía no tiene una cuenta de Azure, regístrese para obtener una [cuenta de evaluación gratuita](https://azure.microsoft.com/free).
- Si usa el portal, abra https://portal.azure.com e inicie sesión con la cuenta de Azure.
- Si usa comandos de PowerShell para completar las tareas de este artículo, ejecute los comandos que se encuentran en [Azure Cloud Shell](https://shell.azure.com/powershell) o ejecute PowerShell en el equipo. Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. Para realizar este tutorial, se necesita la versión 1.0.0 del módulo de Azure PowerShell u otra posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.
- Si usa la interfaz de la línea de comandos (CLI) de Azure para completar las tareas de este artículo, ejecute los comandos que se encuentran en [Azure Cloud Shell](https://shell.azure.com/bash) o ejecute la CLI en el equipo. Para realizar este tutorial, es necesaria la versión 2.0.31 de la CLI de Azure o una versión posterior. Ejecute `az --version` para buscar la versión instalada. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Si ejecuta de forma local la CLI de Azure, también debe ejecutar `az login` para crear una conexión con Azure.
- La cuenta en la que inicia sesión o con la que se conecta a Azure debe tener asignado el rol de [colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o un [rol personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones apropiadas en [Permisos](#permissions).

## <a name="create-a-virtual-network"></a>Creación de una red virtual

1. Seleccione **+ Crear un recurso** > **Redes** > **Red virtual**.
2. Escriba o seleccione valores para las siguientes opciones y seleccione **Crear**:
   - **Name**: tiene que ser único en el [grupo de recursos](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#resource-group) dentro del que seleccione crear la red virtual. No se puede cambiar el nombre una vez creada la red virtual. Puede crear varias redes virtuales con el tiempo. Vea [Convenciones de nomenclatura](/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging#naming-and-tagging-resources) para obtener sugerencias de nombres. El uso de una convención de nomenclatura puede facilitar la administración de varias redes virtuales.
   - **Espacio de direcciones**: el espacio de direcciones de una red virtual se compone de uno o varios intervalos de direcciones no superpuestos que se especifican en la notación CIDR. El rango de direcciones que defina puede ser público o privado (RFC 1918). Tanto si se define el rango de direcciones como público o como privado, el rango de direcciones es accesible solo desde dentro de la red virtual, desde redes virtuales conectadas entre sí y desde las redes locales que se hayan conectado a la red virtual. No se pueden agregar los siguientes rangos de direcciones:
     - 224.0.0.0/4 (multidifusión)
     - 255.255.255.255/32 (difusión)
     - 127.0.0.0/8 (bucle invertido)
     - 169.254.0.0/16 (local de vínculo)
     - 168.63.129.16/32 ([sondeo de estado](../load-balancer/load-balancer-custom-probe-overview.md#probesource) de DNS, DHCP y Azure Load Balancer internos)

     Aunque solo se puede definir un único rango de direcciones cuando se crea la red virtual en el portal, puede agregar varios rangos de direcciones al espacio de direcciones una vez creada la red virtual. Para obtener información sobre cómo agregar un rango de direcciones a una red virtual existente, consulte [Agregar o quitar un rango de direcciones](#add-or-remove-an-address-range).

     >[!WARNING]
     >Si una red virtual tiene rangos de direcciones que se superponen con otra red virtual o red local, las dos redes no se pueden conectar. Antes de definir un rango de direcciones, tenga en cuenta si es posible que en el futuro quiera conectar la red virtual a otras redes virtuales o redes locales.
     >
     >

     - **Nombre de subred**: El nombre de la subred debe ser único dentro de la red virtual. El nombre de subred no se puede cambiar después de crear la subred. El portal requiere que se defina una subred al crear una red virtual, aunque una red virtual no necesita tener ninguna subred. En el portal, solo se puede definir una subred cuando se crea una red virtual. Puede agregar más subredes a la red virtual más adelante, una vez creada la red virtual. Para agregar una subred a una red virtual, consulte [Administrar subredes](virtual-network-manage-subnet.md). Puede crear una red virtual que tenga varias subredes mediante la CLI de Azure o PowerShell.

       >[!TIP]
       >A veces, los administradores crean diferentes subredes para filtrar o controlar el enrutamiento de tráfico entre ellas. Antes de definir subredes, considere cómo quiere filtrar y enrutar el tráfico entre ellas. Para obtener más información sobre el filtrado de tráfico entre subredes, vea [Grupos de seguridad de red](security-overview.md). Azure realiza el enrutamiento de tráfico entre subredes automáticamente, pero puede invalidar las rutas predeterminadas de Azure. Para obtener más información acerca de cómo enruta Azure el tráfico de subredes predeterminado, consulte [Introducción al enrutamiento](virtual-networks-udr-overview.md).
       >

     - **Rango de direcciones de subred**: el intervalo debe estar dentro del espacio de direcciones que especificó para la red virtual. El menor intervalo que se puede especificar es /29, lo que proporciona ocho direcciones IP de subred. De conformidad con el protocolo, Azure reserva la primera y la última dirección de cada subred. Otras tres direcciones están reservadas para el uso del servicio de Azure. Como resultado, una red virtual con un intervalo de direcciones de subred de /29 tiene solo tres direcciones IP utilizables. Si planea conectar una red virtual a una puerta de enlace VPN, debe crear una subred de puerta de enlace. Más información sobre las [consideraciones específicas del intervalo de direcciones de las subredes de puerta de enlace](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md?toc=%2fazure%2fvirtual-network%2ftoc.json#gwsub). En determinadas circunstancias se puede cambiar el intervalo de direcciones una vez creada la subred. Para obtener información sobre cómo cambiar un rango de direcciones de subred, consulte [Administrar subredes](virtual-network-manage-subnet.md).
     - **Suscripción**: Seleccione una [suscripción](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#subscription). No se puede usar la misma red virtual en más de una suscripción de Azure. Sin embargo, puede conectar la red virtual de una suscripción a las redes virtuales de otras suscripciones mediante el [emparejamiento de redes virtuales](virtual-network-peering-overview.md). Cualquier recurso de Azure que se conecte a la red virtual debe estar en la misma suscripción que la red virtual.
     - **Grupo de recursos**: Seleccione un [grupo de recursos](../azure-resource-manager/management/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json#resource-groups) existente, o bien cree uno. Un recurso de Azure que se conecta a la red virtual puede estar en el mismo grupo de recursos que la red virtual o en otro diferente.
     - **Ubicación**: seleccione una [ubicación](https://azure.microsoft.com/regions/) de Azure, también conocida como región. Una red virtual solo puede estar en una ubicación de Azure. Pero se puede conectar una red virtual en una ubicación a una red virtual en otra ubicación mediante el uso de VPN Gateway. Cualquier recurso de Azure que se conecte a la red virtual debe estar en la misma ubicación que la red virtual.

**Comandos**

- CLI de Azure: [az network vnet create](/cli/azure/network/vnet)
- PowerShell: [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork)

## <a name="view-virtual-networks-and-settings"></a>Ver las redes virtuales y su configuración

1. En el cuadro de búsqueda de la parte superior del portal, escriba *redes virtuales*. Cuando aparezca la opción **Redes virtuales** en los resultados de la búsqueda, selecciónela.
2. En la lista de redes virtuales, seleccione la red virtual de la cual quiera ver la configuración.
3. Los siguientes ajustes de configuración se muestran en la red virtual seleccionada:
   - **Información general**: proporciona información sobre la red virtual, incluido el espacio de direcciones y los servidores DNS. En la siguiente captura de pantalla se muestra la configuración de información general de una red virtual denominada **MyVNet**:

     ![Información general de interfaz de red](./media/manage-virtual-network/vnet-overview.png)

     Puede llevar una red virtual a una suscripción o grupo de recursos diferente si selecciona la opción **Cambiar** que se encuentra junto a **Grupo de recursos** o **Nombre de la suscripción**. Para obtener información sobre cómo mover una red virtual, vea [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../azure-resource-manager/management/move-resource-group-and-subscription.md?toc=%2fazure%2fvirtual-network%2ftoc.json). En el artículo se enumeran los requisitos previos y se indica cómo trasladar recursos mediante Azure Portal, PowerShell y la CLI de Azure. Todos los recursos que están conectados a la red virtual se deben mover con la red virtual.
   - **Espacio de direcciones**: se enumeran los espacios de direcciones asignados a la red virtual. Para obtener información sobre cómo agregar y quitar un rango de direcciones de un espacio de direcciones, complete los pasos de la sección [Agregar o quitar un rango de direcciones](#add-or-remove-an-address-range).
   - **Dispositivos conectados**: se muestran todos los recursos que están conectados a la red virtual. En la captura de pantalla anterior, hay tres interfaces de red y un equilibrador de carga conectados a la red virtual. Se muestran todos los nuevos recursos que se creen y conecten a la red virtual. Si elimina un recurso que estaba conectado a la red virtual, ya no aparecerá en la lista.
   - **Subredes**: se muestra una lista de las subredes que existen dentro de la red virtual. Para obtener información sobre cómo agregar y quitar una subred, consulte [Administrar subredes](virtual-network-manage-subnet.md).
   - **Servidores DNS**: puede especificar si el servidor DNS interno de Azure o un servidor DNS personalizado proporciona resolución de nombres para los dispositivos que están conectados a la red virtual. Al crear una red virtual mediante Azure Portal, los servidores DNS de Azure se usan para la resolución de nombres dentro de una red virtual, de forma predeterminada. Para modificar los servidores DNS, complete los pasos de la sección [Cambiar servidores DNS](#change-dns-servers) de este artículo.
   - **Emparejamientos**: si hay emparejamientos existentes en la suscripción, estos aparecerán aquí. Puede ver la configuración de emparejamientos existentes, o crear, cambiar o eliminar emparejamientos. Para obtener más información sobre emparejamiento, vea [Emparejamiento de redes virtuales de Azure](virtual-network-peering-overview.md).
   - **Propiedades**: muestra la configuración de la red virtual, incluidos el identificador de recurso de la red virtual y la suscripción de Azure en la que se encuentra.
   - **Diagrama**: el diagrama proporciona una representación visual de todos los dispositivos conectados a la red virtual. El diagrama muestra alguna información clave sobre los dispositivos. Para administrar un dispositivo en esta vista, vaya al diagrama y seleccione el dispositivo.
   - **Configuración común de Azure**: Para más información sobre la configuración común de Azure, consulte la información siguiente:
     - [Registro de actividad](../azure-monitor/platform/platform-logs-overview.md)
     - [Control de acceso (IAM)](../role-based-access-control/overview.md)
     - [Etiquetas](../azure-resource-manager/management/tag-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
     - [Bloqueos](../azure-resource-manager/management/lock-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
     - [Script de Automation](../azure-resource-manager/management/manage-resource-groups-portal.md#export-resource-groups-to-templates)

**Comandos**

- CLI de Azure: [az network vnet show](/cli/azure/network/vnet)
- PowerShell: [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork)

## <a name="add-or-remove-an-address-range"></a>Agregar o quitar un rango de direcciones

Puede agregar y quitar rangos de direcciones de una red virtual. Un rango de direcciones tiene que especificarse en una notación CIDR y no puede superponerse con otros rangos de direcciones dentro de la misma red virtual. Los rangos de direcciones que defina pueden ser públicos o privados (RFC 1918). Tanto si se define el rango de direcciones como público o como privado, el rango de direcciones es accesible solo desde dentro de la red virtual, desde redes virtuales conectadas entre sí y desde las redes locales que se hayan conectado a la red virtual. 

Puede disminuir el intervalo de direcciones de una red virtual siempre y cuando siga incluyendo los intervalos de las subredes asociadas. Además, puede extender el intervalo de direcciones, por ejemplo, cambiar de /16 a /8. 

<!-- the above statement has been edited to reflect the most recent comments on the reopened issue: https://github.com/MicrosoftDocs/azure-docs/issues/20572 -->

No se pueden agregar los siguientes rangos de direcciones:

- 224.0.0.0/4 (multidifusión)
- 255.255.255.255/32 (difusión)
- 127.0.0.0/8 (bucle invertido)
- 169.254.0.0/16 (local de vínculo)
- 168.63.129.16/32 ([sondeo de estado](../load-balancer/load-balancer-custom-probe-overview.md#probesource) de DNS, DHCP y Azure Load Balancer internos)

Para agregar o quitar un rango de direcciones:

1. En el cuadro de búsqueda de la parte superior del portal, escriba *redes virtuales*. Cuando aparezca la opción **Redes virtuales** en los resultados de la búsqueda, selecciónela.
2. En la lista de redes virtuales, seleccione la red virtual para la que quiera agregar o quitar un rango de direcciones.
3. En **CONFIGURACIÓN**, seleccione **Espacio de direcciones**.
4. Complete una de las siguientes opciones:
    - **Agregar un intervalo de direcciones**: Escriba el nuevo intervalo de direcciones. El rango de direcciones no puede superponerse a un rango de direcciones ya existente y que esté definido para la red virtual.
    - **Quitar un intervalo de direcciones**: a la derecha del intervalo de direcciones que quiera quitar, seleccione **...** y, a continuación, seleccione **Quitar**. Si existe una subred en el rango de direcciones, no se puede quitar ese rango de direcciones. Para quitar un rango de direcciones, es necesario eliminar primero las subredes que existen en ese rango de direcciones y todos los recursos conectados a las subredes.
5. Seleccione **Guardar**.

**Comandos**

- CLI de Azure: [az network vnet update](/cli/azure/network/vnet)
- PowerShell: [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork)

## <a name="change-dns-servers"></a>Cambio de los servidores DNS

Todas las máquinas virtuales que están conectadas a la red virtual se registran con los servidores DNS que especifique para la red virtual. También usan el servidor DNS especificado para la resolución de nombres. Cada interfaz de red (NIC) en una máquina virtual puede tener su propia configuración de servidor DNS. Si una NIC tiene su propia configuración de servidor DNS, esta invalida la configuración del servidor DNS para la red virtual. Para obtener más información sobre la configuración de DNS de NIC, vea [Configuración y tareas de la interfaz de red](virtual-network-network-interface.md#change-dns-servers). Para obtener más información sobre la resolución de nombres para las máquinas virtuales y las instancias de rol de Azure Cloud Services, vea [Resolución de nombres para las máquinas virtuales e instancias de rol](virtual-networks-name-resolution-for-vms-and-role-instances.md). Para agregar, cambiar o quitar un servidor DNS:

1. En el cuadro de búsqueda de la parte superior del portal, escriba *redes virtuales*. Cuando aparezca la opción **Redes virtuales** en los resultados de la búsqueda, selecciónela.
2. En la lista de redes virtuales, seleccione la red virtual en la cual quiera cambiar los servidores DNS.
3. Seleccione **Servidores DNS**, en **CONFIGURACIÓN**.
4. Seleccione una de las siguientes opciones:
   - **Predeterminado (proporcionado por Azure)** : todos los nombres de los recursos y las direcciones IP privadas se registran automáticamente en los servidores DNS de Azure. Puede resolver nombres entre los recursos conectados a la misma red virtual. No se puede usar esta opción para resolver nombres entre redes virtuales. Para resolver nombres de otras redes virtuales, tiene que usar un servidor DNS personalizado.
   - **Personalizado**: puede agregar uno o varios servidores, hasta el límite de Azure para una red virtual. Para obtener más información sobre los límites de servidor DNS, vea [Límites de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#virtual-networking-limits-classic). Tiene las siguientes opciones:
   - **Agregar una dirección**: agrega el servidor a la lista de servidores DNS de la red virtual. Esta opción también registra el servidor DNS con Azure. Si ya ha registrado un servidor DNS con Azure, puede seleccionarlo en la lista.
   - **Quitar una dirección**: junto al servidor que quiere quitar, seleccione **...** y, a continuación, **Quitar**. Eliminar el servidor solo lo quita de esta lista de redes virtuales. El servidor DNS seguirá registrado en Azure para que lo usen otras redes virtuales.
   - **Reordenar direcciones de servidor DNS**: es importante comprobar que se enumeran los servidores DNS en el orden correcto para su entorno. Las listas de servidores DNS se usan en el orden en que se especifican. No funcionan como una instalación round robin. Si se puede acceder al primer servidor DNS de la lista, el cliente usa ese servidor DNS, con independencia de si el servidor DNS funciona correctamente. Quite todos los servidores DNS que aparecen y vuelva a agregarlos en el orden que desee.
   - **Cambiar una dirección**: resalte el servidor DNS en la lista y después escriba el nuevo nombre.
5. Seleccione **Guardar**.
6. Reinicie las máquinas virtuales conectadas a la red virtual para que se les asigne la nueva configuración de servidor DNS. Hasta que se reinicien, las máquinas virtuales seguirán usando la configuración de DNS que consideran actual.

**Comandos**

- CLI de Azure: [az network vnet update](/cli/azure/network/vnet)
- PowerShell: [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork)

## <a name="delete-a-virtual-network"></a>Eliminar una red virtual

Solo se puede eliminar una red virtual si no tiene recursos conectados. Si hay recursos conectados a cualquier subred dentro de la red virtual, primero tiene que eliminarlos. Los pasos necesarios para eliminar un recurso varían según el recurso. Para obtener más información sobre cómo eliminar los recursos conectados a una subred, lea la documentación de cada tipo de recurso que quiera eliminar. Para eliminar una red virtual:

1. En el cuadro de búsqueda de la parte superior del portal, escriba *redes virtuales*. Cuando aparezca la opción **Redes virtuales** en los resultados de la búsqueda, selecciónela.
2. En la lista de redes virtuales, seleccione la red virtual que quiere eliminar.
3. En **CONFIGURACIÓN**, seleccione **Dispositivos conectados** para confirmar que no haya dispositivos conectados a la red virtual. Si hay dispositivos conectados, primero debe eliminarlos antes de poder eliminar la red virtual. Si no hay ningún dispositivo conectado, seleccione **Información general**.
4. Seleccione **Eliminar**.
5. Para confirmar la eliminación de la red virtual, seleccione **Sí**.

**Comandos**

- CLI de Azure: [azure network vnet delete](/cli/azure/network/vnet)
- PowerShell: [Remove-AzVirtualNetwork](/powershell/module/az.network/remove-azvirtualnetwork)

## <a name="permissions"></a>Permisos

Para realizar tareas en redes virtuales, su cuenta debe estar asignada al rol de [colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o a un rol [personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones adecuadas que se muestran en la tabla siguiente:

| Acción                                  |   Nombre                                |
|---------------------------------------- |   --------------------------------    |
|Microsoft.Network/virtualNetworks/read   |   Leer una red virtual              |
|Microsoft.Network/virtualNetworks/write  |   Crear o actualizar una red virtual  |
|Microsoft.Network/virtualNetworks/delete |   Eliminar una red virtual            |

## <a name="next-steps"></a>Pasos siguientes

- Crear una red virtual con scripts de ejemplo de [PowerShell](powershell-samples.md) o de la [CLI de Azure](cli-samples.md), o bien con [plantillas de Azure Resource Manager](template-samples.md)
- Crear y asignar [definiciones de Azure Policy](policy-samples.md) para redes virtuales
