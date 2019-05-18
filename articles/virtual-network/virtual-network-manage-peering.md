---
title: Crear, cambiar o eliminar un emparejamiento de red virtual de Azure | Microsoft Docs
description: Aprenda a crear, cambiar o eliminar un emparejamiento de red virtual.
services: virtual-network
documentationcenter: na
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/01/2019
ms.author: anavin
ms.openlocfilehash: ee7ffba182cdbc2a77df5edf112e29c49cd610ad
ms.sourcegitcommit: 17411cbf03c3fa3602e624e641099196769d718b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65519652"
---
# <a name="create-change-or-delete-a-virtual-network-peering"></a>Crear, cambiar o eliminar un emparejamiento de red virtual

Aprenda a crear, cambiar o eliminar un emparejamiento de red virtual. El emparejamiento de red virtual permite conectar redes virtuales de la misma región y entre regiones (lo que se conoce también como Emparejamiento de VNET global) mediante la red troncal de Azure. Una vez emparejadas, las redes virtuales se siguen administrando como recursos separados. Si no está familiarizado con el emparejamiento de redes virtuales, puede obtener más información sobre él en la [información general sobre el emparejamiento de redes virtuales](virtual-network-peering-overview.md) o si sigue un [tutorial](tutorial-connect-virtual-networks-portal.md).

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Complete las tareas siguientes antes de seguir los pasos de las secciones de este artículo:

- Si todavía no tiene una cuenta de Azure, regístrese para obtener una [cuenta de evaluación gratuita](https://azure.microsoft.com/free).
- Si utiliza el portal, abra https://portal.azure.com e inicie sesión con una cuenta que tenga los [permisos necesarios](#permissions) para trabajar con emparejamientos.
- Si usa comandos de PowerShell para completar las tareas de este artículo, ejecute los comandos que se encuentran en [Azure Cloud Shell](https://shell.azure.com/powershell) o ejecute PowerShell en el equipo. Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. Para realizar este tutorial, se necesita la versión 1.0.0 del módulo de Azure PowerShell u otra posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si está ejecutando PowerShell localmente, también debe ejecutar `Connect-AzAccount` con una cuenta que tenga los [permisos necesarios](#permissions) para trabajar con emparejamiento, para crear una conexión con Azure.
- Si usa la interfaz de la línea de comandos (CLI) de Azure para completar las tareas de este artículo, ejecute los comandos que se encuentran en [Azure Cloud Shell](https://shell.azure.com/bash) o ejecute la CLI en el equipo. Para realizar este tutorial, es necesaria la versión 2.0.31 de la CLI de Azure o una versión posterior. Ejecute `az --version` para buscar la versión instalada. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Si está ejecutando la CLI de Azure localmente, también debe ejecutar `az login` con una cuenta que tenga los [permisos necesarios](#permissions) para trabajar con emparejamiento, para crear una conexión con Azure.

La cuenta en la que inicia sesión o con la que se conecta a Azure debe tener asignado el rol de [colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o un [rol personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones apropiadas en [Permisos](#permissions).

## <a name="create-a-peering"></a>Creación de un emparejamiento

Antes de crear un emparejamiento, familiarícese con los requisitos y las restricciones, así como los [permisos necesarios](#permissions).

1. En el cuadro de búsqueda de la parte superior de Azure Portal, escriba *redes virtuales*. Cuando aparezca la opción **Redes virtuales** en los resultados de la búsqueda, selecciónela. No seleccione **Redes virtuales (clásico)** si aparece en la lista, ya que no se puede crear un emparejamiento de una red virtual implementada mediante el modelo de implementación clásica.
2. Seleccione en la lista la red virtual que quiera emparejar.
3. En **CONFIGURACIÓN**, seleccione **Emparejamientos**.
4. Seleccione **+ agregar**. 
5. <a name="add-peering"></a>Escriba o seleccione valores para la siguiente configuración:
    - **Nombre:** el nombre del emparejamiento debe ser único dentro de la red virtual.
    - **Modelo de implementación de red virtual:** seleccione con qué modelo de implementación se implementó la red virtual con la que quiere realizar el emparejamiento.
    - **Conozco mi Id. de recurso**: si tiene acceso de lectura a la red virtual con la que quiere realizar el emparejamiento, no active esta casilla. Si no tiene acceso de lectura a la red virtual o la suscripción con la que quiere realizar el emparejamiento, active esta casilla. Escriba el identificador de recurso completo de la red virtual con la que quiere realizar el emparejamiento en el cuadro **Id. de recurso** que aparece cuando se activa la casilla. El identificador de recurso que especifique debe ser para una red virtual que exista en la misma región de Azure, o en una [región](#requirements-and-constraints) [diferente admitida](https://azure.microsoft.com/regions), que esta red virtual. El recurso completo es similar a identificador `/subscriptions/<Id>/resourceGroups/<resource-group-name>/providers/Microsoft.Network/virtualNetworks/<virtual-network-name>`. Para obtener el identificador de recurso de una red virtual, vea las propiedades de la red virtual. Para obtener información sobre cómo ver las propiedades de una red virtual, consulte [Administración de redes virtuales](manage-virtual-network.md#view-virtual-networks-and-settings). Si la suscripción está asociada a un inquilino de Azure Active Directory que resulte ser diferente al de la suscripción que tiene la red virtual en la que está creando el emparejamiento, primero debe agregar un usuario de cada inquilino como [usuario invitado](../active-directory/b2b/add-users-administrator.md?toc=%2fazure%2fvirtual-network%2ftoc.json#add-guest-users-to-the-directory) en el inquilino opuesto.
    - **Subscription** (Suscripción): seleccione la [suscripción](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#subscription) de la red virtual con la que quiere realizar el emparejamiento. Se muestran una o varias suscripciones, en función del número de suscripciones al que tenga acceso de lectura su cuenta. Si ha activado la casilla **Id. de recurso**, esta opción no está disponible.
    - **Red virtual:** seleccione la red virtual con la que quiere realizar el emparejamiento. Puede seleccionar una red virtual creada a través de cualquier modelo de implementación de Azure. Si desea seleccionar una red virtual en otra región, debe seleccionar una red virtual en una [región admitida](#cross-region). Debe tener acceso de lectura a la red virtual para que sea visible en la lista. Si una red virtual aparece en gris, puede deberse a que el espacio de direcciones de la red virtual se superpone con el espacio de direcciones de esta red virtual. Si los espacios de direcciones de las redes virtuales se superponen, no se pueden emparejar. Si ha activado la casilla **Id. de recurso**, esta opción no está disponible.
    - **Permitir acceso a red virtual**: seleccione **Habilitado** (valor predeterminado) si quiere habilitar la comunicación entre las dos redes virtuales. Al habilitar la comunicación entre las redes virtuales, permite que los recursos conectados a cualquier red virtual se comuniquen entre sí con el mismo ancho de banda y latencia que si estuvieran conectados a la misma red virtual. Todas las comunicaciones entre los recursos de las dos redes virtuales se realizan a través de la red privada de Azure. La etiqueta de servicio **VirtualNetwork** para los grupos de seguridad de red abarca la red virtual y la red virtual emparejada. Para más información acerca de las etiquetas de servicio de los grupos de seguridad de red, consulte la [información general sobre los grupos de seguridad de red](security-overview.md#service-tags). Seleccione **Deshabilitado** si no quiere que el tráfico fluya a la red virtual emparejada. Por ejemplo, podría seleccionar **Deshabilitado** si ha emparejado una red virtual con otra red virtual pero quiere deshabilitar en ocasiones el flujo de tráfico entre ambas redes. Le resultará más cómoda la opción de habilitar y deshabilitar que eliminar y volver a crear emparejamientos. Si esta opción está deshabilitada, el tráfico no fluye entre las redes virtuales emparejadas.
    - **Permitir tráfico reenviado:** seleccione esta casilla para permitir que el tráfico que *reenvió* una aplicación virtual de red desde una red virtual (este tráfico no se origina en la red virtual) fluya a esta red virtual mediante un emparejamiento. Por ejemplo, supongamos que hay tres redes virtuales llamadas Radio1, Radio2 y Concentrador. Existe un emparejamiento entre cada red virtual de radio y la red virtual de concentrador, pero no existe ninguno entre las redes virtuales de radio. Se implementa una aplicación virtual de red en la red virtual de concentrador, y las rutas definidas por el usuario se aplican a cada red virtual de radio que redirige el tráfico entre las subredes a través de la aplicación virtual de red. Si no está activada esta casilla de verificación para el emparejamiento entre cada red virtual de radio y la red virtual del concentrador, el tráfico no fluye entre las redes virtuales de radio porque el concentrador no reenvía el tráfico entre las redes virtuales. Aunque el hecho de habilitar esta funcionalidad permite el tráfico reenviado a través del emparejamiento, no se crean rutas definidas por el usuario ni aplicaciones virtuales de red. Las rutas definidas por el usuario y las aplicaciones virtuales de red se crean por separado. Obtenga información sobre las [rutas definidas por el usuario](virtual-networks-udr-overview.md#user-defined). No es necesario comprobar esta configuración si se reenvía el tráfico entre redes virtuales a través de VPN Gateway de Azure.
    - **Permitir tránsito de puerta de enlace:** active esta casilla si tiene una puerta de enlace de red virtual asociada a esta red virtual y quiere permitir que el tráfico procedente de la red virtual emparejada fluya a través de la puerta de enlace. Por ejemplo, esta red virtual puede asociarse a una red local a través de una puerta de enlace de red virtual. La puerta de enlace puede ser una puerta de enlace de ExpressRoute o VPN. La selección de esta casilla permite que el tráfico procedente de la red virtual emparejada fluya a través de la puerta de enlace asociada a esta red virtual hacia la red local. Si activa esta casilla, la red virtual emparejada no puede tener una puerta de enlace configurada. La red virtual emparejada debe tener seleccionada la casilla **Usar puertas de enlace remotas** cuando se configura el emparejamiento de la otra red virtual a esta red virtual. Si deja esta casilla sin activar (valor predeterminado), el tráfico procedente de la red virtual emparejada seguirá fluyendo a esta red virtual, pero no podrá fluir a través de una puerta de enlace de red virtual asociada a esta red virtual. Si el emparejamiento es entre una red virtual (Resource Manager) y una red virtual (clásica), la puerta de enlace debe estar en la red virtual (Resource Manager). No se puede habilitar esta opción si empareja redes virtuales de regiones diferentes.

       Además de reenviar tráfico a una red local, una puerta de enlace VPN puede reenviar el tráfico de red entre redes virtuales emparejadas con la red virtual en que se encuentra la puerta de enlace, sin necesidad de que las redes virtuales se tengan que emparejar entre sí. Usar una puerta de enlace VPN para reenviar el tráfico es útil si quiere usar una puerta de enlace VPN en una red virtual de concentrador (vea el ejemplo de concentrador y radio descrito para **Permitir tráfico reenviado**) para redirigir el tráfico entre redes virtuales de radio que no están emparejadas entre sí. Para obtener más información sobre permitir el uso de una puerta de enlace de tránsito, consulte [Configuración del tránsito de la puerta de enlace de VPN para el emparejamiento de red virtual](../vpn-gateway/vpn-gateway-peering-gateway-transit.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Este escenario requiere la implementación de las rutas definidas por el usuario que especifican la puerta de enlace de red virtual como el tipo de próximo salto. Obtenga información sobre las [rutas definidas por el usuario](virtual-networks-udr-overview.md#user-defined). Solo puede especificar una puerta de enlace VPN como un tipo de próximo salto en una ruta definida por el usuario; no puede especificar una puerta de enlace de ExpressRoute como el tipo de próximo salto en una ruta definida por el usuario. No se puede habilitar esta opción si empareja redes virtuales de regiones diferentes.

    - **Usar puertas de enlace remotas:** active esta casilla para permitir que el tráfico procedente de esta red virtual fluya a través de la puerta de enlace de red virtual asociada a la red virtual con la que está realizando el emparejamiento. Por ejemplo, la red virtual con la que está realizando el emparejamiento tiene una puerta de enlace de VPN asociada que permite la comunicación a una red local.  La activación de esta casilla permite que el tráfico procedente de esta red virtual fluya a través de la puerta de enlace de VPN asociada a la red virtual emparejada. Si activa esta casilla, la red virtual emparejada debe tener asociada una puerta de enlace de red virtual y debe tener activada la casilla **Permitir tránsito de puerta de enlace**. Si deja esta casilla sin activar (valor predeterminado), el tráfico procedente de la red virtual emparejada todavía puede fluir a esta red virtual, pero no podrá fluir a través de una puerta de enlace de red virtual asociada a esta red virtual.
    Esta opción puede estar habilitada solo en un emparejamiento de esta red virtual.

        No puede usar las puertas de enlace remotas si ya tiene una puerta de enlace configurada en la red virtual. No se puede habilitar esta opción si empareja redes virtuales de regiones diferentes. Para obtener más información sobre el uso de una puerta de enlace de tránsito, consulte [Configuración del tránsito de la puerta de enlace de VPN para el emparejamiento de red virtual](../vpn-gateway/vpn-gateway-peering-gateway-transit.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

6. Para agregar el emparejamiento a la red virtual que ha seleccionado, elija **Aceptar**.

Para obtener instrucciones paso a paso para implementar el emparejamiento entre redes virtuales de distintas suscripciones y los modelos de implementación, consulte los [pasos siguientes](#next-steps).

### <a name="commands"></a>Comandos:

- **CLI de Azure**: [az network vnet peering create](/cli/azure/network/vnet/peering)
- **PowerShell**: [Add-AzVirtualNetworkPeering](/powershell/module/az.network/add-azvirtualnetworkpeering)

## <a name="view-or-change-peering-settings"></a>Visualización o cambio de la configuración de emparejamiento

Antes de cambiar un emparejamiento, familiarícese con los requisitos y las restricciones, así como con los [permisos necesarios](#permissions).

1. En el cuadro de búsqueda de la parte superior del portal, escriba *redes virtuales*. Cuando aparezca la opción **Redes virtuales** en los resultados de la búsqueda, selecciónela. No seleccione **Redes virtuales (clásico)** si aparece en la lista, ya que no se puede crear un emparejamiento de una red virtual implementada mediante el modelo de implementación clásica.
2. Seleccione en la lista la red virtual a la que quiera cambiar la configuración de emparejamiento.
3. En **CONFIGURACIÓN**, seleccione **Emparejamientos**.
4. Seleccione el emparejamiento cuya configuración quiere ver o cambiar.
5. Cambie los valores pertinentes. Obtenga información sobre las opciones para cada configuración en [paso 5](#add-peering) de crear un emparejamiento.
6. Seleccione **Guardar**.

**Comandos**

- **CLI de Azure**: [az network vnet peering list](/cli/azure/network/vnet/peering) para mostrar los emparejamientos de una red virtual, [az network vnet peering show](/cli/azure/network/vnet/peering) para mostrar la configuración de un emparejamiento específico y [az network vnet peering update](/cli/azure/network/vnet/peering) para cambiar la configuración de emparejamiento.|
- **PowerShell**: [Get-AzVirtualNetworkPeering](/powershell/module/az.network/get-azvirtualnetworkpeering) para recuperar la configuración de emparejamiento de vista y [conjunto AzVirtualNetworkPeering](/powershell/module/az.network/set-azvirtualnetworkpeering) para cambiar la configuración.

## <a name="delete-a-peering"></a>Eliminación de un emparejamiento

Antes de eliminar un emparejamiento, asegúrese de que la cuenta tiene los [permisos necesarios](#permissions).

Cuando se elimina un emparejamiento, el tráfico procedente de una red virtual ya no fluye a la red virtual emparejada. Cuando se emparejan redes virtuales implementadas mediante el Administrador de recursos, cada red virtual tiene un emparejamiento con la otra red virtual. Aunque el hecho de eliminar el emparejamiento de una red virtual deshabilita la comunicación entre las redes virtuales, no elimina el emparejamiento de la otra red virtual. El estado del emparejamiento que existe en la otra red virtual es **Desconectado**. Para volver a crear el emparejamiento, primero tendrá que volver a crear el emparejamiento en la primera red virtual y el estado de emparejamiento de ambas redes virtuales deberá haber cambiado a *Conectado*.

Si quiere que las redes virtuales se comuniquen algunas veces pero no siempre, en lugar de eliminar un emparejamiento, puede establecer el valor **Permitir acceso a red virtual** en **Deshabilitado**. Para obtener información sobre cómo hacerlo, vea el paso 6 de la creación de una sección de emparejamiento de este artículo. Probablemente le resulte más fácil deshabilitar y habilitar el acceso a la red que eliminar y volver a crear emparejamientos.

1. En el cuadro de búsqueda de la parte superior del portal, escriba *redes virtuales*. Cuando aparezca la opción **Redes virtuales** en los resultados de la búsqueda, selecciónela. No seleccione **Redes virtuales (clásico)** si aparece en la lista, ya que no se puede crear un emparejamiento de una red virtual implementada mediante el modelo de implementación clásica.
2. Seleccione en la lista la red virtual de la cual quiera eliminar el emparejamiento.
3. En **CONFIGURACIÓN**, seleccione **Emparejamientos**.
4. En el lado derecho del emparejamiento que quiere eliminar, seleccione **...**, **Eliminar** y, a continuación, seleccione **Sí** para eliminar el emparejamiento de la primera red virtual.
5. Complete los pasos anteriores para eliminar el emparejamiento de la otra red virtual del emparejamiento.

**Comandos**

- **CLI de Azure**: [az network vnet peering delete](/cli/azure/network/vnet/peering)
- **PowerShell**: [Remove-AzVirtualNetworkPeering](/powershell/module/az.network/remove-azvirtualnetworkpeering)

## <a name="requirements-and-constraints"></a>Requisitos y restricciones

- <a name="cross-region"></a>Puede emparejar redes virtuales de la misma región o de regiones diferentes. Emparejamiento de redes virtuales en diferentes regiones se conoce también como *emparejamiento de VNet Global*. 
- Al crear un emparejamiento global, las redes virtuales emparejadas pueden existir en cualquier región de la nube pública de Azure o las regiones en la nube de China o regiones en la nube de Government. No se pueden emparejar entre nubes. Por ejemplo, no se pueden emparejar una red virtual en la nube pública de Azure a una red virtual en la nube de China de Azure.
- Los recursos en una red virtual no pueden comunicarse con la dirección IP front-end de un equilibrador de carga interno básico en una red virtual emparejada globalmente. Compatibilidad con Load Balancer básico solo existe en la misma región. Compatibilidad con Load Balancer estándar existe para, el emparejamiento de VNet y emparejamiento de VNet Global. Servicios que usan un equilibrador de carga básico que no funcionará a través de emparejamiento de VNet Global se documentan [aquí.](virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers)
- Puede utilizar puertas de enlace remotas o permitir el tránsito de puerta de enlace en redes virtuales emparejadas globalmente y redes virtuales emparejadas localmente.
- Las redes virtuales pueden estar en la misma suscripción o en suscripciones distintas. Cuando empareja redes virtuales en distintas suscripciones, ambas suscripciones pueden estar asociadas al mismo inquilino de Azure Active Directory o a uno diferente. Si no dispone de un inquilino de AD, puede [crearla](../active-directory/develop/quickstart-create-new-tenant.md?toc=%2fazure%2fvirtual-network%2ftoc.json-a-new-azure-ad-tenant). El soporte técnico para realizar emparejamientos entre redes virtuales de las suscripciones asociadas a diferentes inquilinos de Azure Active Directory no está disponible en el portal. Puede usar la CLI, PowerShell o plantillas.
- Las redes virtuales que empareje deben tener espacios de direcciones IP que no se solapen.
- Una vez que una red virtual se ha emparejado con otra red virtual, no se pueden agregar rangos de direcciones al espacio de direcciones de una red virtual ni pueden eliminarse de este. Para agregar o quitar rangos de direcciones, elimine el emparejamiento, agregue o quite los rangos de direcciones y, a continuación, vuelva a crear el emparejamiento. Para agregar rangos de direcciones a las redes virtuales o quitarlos, consulte [Manage virtual networks](manage-virtual-network.md) (Administrar redes virtuales).
- Es posible emparejar dos redes virtuales implementadas mediante el Administrador de recursos, o bien una red virtual implementada mediante el Administrador de recursos con una red virtual implementada mediante el modelo de implementación clásica. No se pueden emparejar dos redes virtuales creadas mediante el modelo de implementación clásica. Si no está familiarizado con los modelos de implementación de Azure, vea el artículo de [descripción de los modelos de implementación de Azure](../azure-resource-manager/resource-manager-deployment-model.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Puede usar [VPN Gateway](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json#V2V) para conectar dos redes virtuales creadas mediante el modelo de implementación clásica.
- Cuando se emparejan dos redes virtuales creadas mediante el Administrador de recursos, debe configurarse un emparejamiento para cada red virtual en el emparejamiento. Vea uno de los tipos siguientes de estado de emparejamiento: 
  - *Iniciado:* Cuando se crea el emparejamiento a la segunda red virtual desde la primera red virtual, el estado de emparejamiento es *Iniciado*. 
  - *Conectado:* Cuando se crea el emparejamiento desde la segunda red virtual a la primera red virtual, el estado de emparejamiento es *Conectado*. Si consulta el estado de emparejamiento de la primera red virtual, verá que ha cambiado de *Iniciado* a *Conectado*. El emparejamiento no se habrá establecido correctamente mientras el estado de ambos emparejamientos de red virtual no sea *Conectado*.
- Al emparejar una red virtual creada mediante el Administrador de recursos con una red virtual creada mediante el modelo de implementación clásica, solo se configura un emparejamiento para la red virtual implementada mediante el Administrador de recursos. No se puede configurar el emparejamiento para una red virtual (clásica), o bien entre dos redes virtuales implementadas mediante el modelo de implementación clásica. Cuando se crea el emparejamiento de la red virtual (Administrador de recursos) a la red virtual (clásica), el estado de emparejamiento es *Actualizando*, pero en breve cambia a *Conectado*.
- El emparejamiento se establece entre dos redes virtuales. Los emparejamientos no son transitivos. Si crea emparejamientos entre:
  - VirtualNetwork1 y VirtualNetwork2
  - VirtualNetwork2 y VirtualNetwork3

  No hay ningún emparejamiento entre VirtualNetwork1 y VirtualNetwork3 a través de VirtualNetwork2. Si desea crear un emparejamiento de red virtual entre VirtualNetwork1 y VirtualNetwork3, debe crear un emparejamiento entre VirtualNetwork1 y VirtualNetwork3.
- No se pueden resolver nombres en redes virtuales emparejadas mediante la resolución de nombres predeterminada de Azure. Para resolver nombres de otras redes virtuales, tiene que usar [Azure DNS para dominios privados](../dns/private-dns-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o un servidor DNS personalizado. Para obtener información sobre cómo configurar el servidor DNS, consulte [Resolución de nombres mediante su propio servidor DNS](virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).
- Los recursos de ambas redes virtuales emparejadas en la misma región pueden comunicarse entre sí con el mismo ancho de banda y latencia que si estuvieran en la misma red virtual. A pesar de ello, el tamaño de cada máquina virtual tiene su propio ancho de banda de red máximo. Para más información sobre el ancho de banda de red máximo para los diferentes tamaños de máquina virtual, consulte los artículos sobre los tamaños de máquina virtual [Windows](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o [Linux](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
- Una red virtual puede emparejarse con otra red virtual y también conectarse a otra red virtual con una puerta de enlace de red virtual de Azure. Cuando las redes virtuales están conectadas mediante emparejamiento y una puerta de enlace, el tráfico entre las redes virtuales fluye a través de la configuración de emparejamiento, en lugar de la puerta de enlace.
- Para asegurarse de que se están descargando las nuevas rutas en el cliente, los clientes VPN de punto a sitio deben descargarse de nuevo después de que el emparejamiento de red virtual se haya configurado correctamente.
- Hay un cargo nominal para el tráfico de entrada y salida que utiliza un emparejamiento de red virtual. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/virtual-network)para obtener más información.

## <a name="permissions"></a>Permisos

Las cuentas que use para trabajar con el emparejamiento de redes virtuales deben asignarse a los siguientes roles:

- [Colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor): para una red virtual implementada mediante Resource Manager.
- [Colaborador de la red virtual clásica](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#classic-network-contributor): para una red virtual implementada a través del modelo de implementación clásico.

Si su cuenta no está asignada a uno de los roles anteriores, se debe asignar a un [rol personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones necesarias de la tabla siguiente:

| .                                                          | NOMBRE |
|---                                                              |---   |
| Microsoft.Network/virtualNetworks/virtualNetworkPeerings/write  | Necesaria para crear un emparejamiento desde la red virtual A hasta la red virtual B. La red virtual A debe ser una red virtual (Resource Manager)          |
| Microsoft.Network/virtualNetworks/peer/action                   | Necesaria para crear un emparejamiento desde la red virtual B (Resource Manager) a la red virtual A                                                       |
| Microsoft.ClassicNetwork/virtualNetworks/peer/action                   | Necesaria para crear un emparejamiento desde la red virtual B (clásica) a la red virtual A                                                                |
| Microsoft.Network/virtualNetworks/virtualNetworkPeerings/read   | Lectura de un emparejamiento de redes virtuales   |
| Microsoft.Network/virtualNetworks/virtualNetworkPeerings/delete | Eliminación de un emparejamiento de redes virtuales |

## <a name="next-steps"></a>Pasos siguientes

- Un emparejamiento de redes virtuales se crea entre redes virtuales creadas mediante el mismo modelo de implementación o mediante modelos distintos que existen en la misma suscripción o en cualquier suscripción diferente. Realice el tutorial para uno de los siguientes escenarios:

  |Modelo de implementación de Azure             | Subscription  |
  |---------                          |---------|
  |Ambas mediante Resource Manager              |[La misma](tutorial-connect-virtual-networks-portal.md)|
  |                                   |[Diferente](create-peering-different-subscriptions.md)|
  |Una mediante Resource Manager y la otra clásica  |[La misma](create-peering-different-deployment-models.md)|
  |                                   |[Diferente](create-peering-different-deployment-models-subscriptions.md)|

- Aprenda a crear una [topología de red de concentrador y radio](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?toc=%2fazure%2fvirtual-network%2ftoc.json)
- Crear un emparejamiento de redes virtuales con scripts de ejemplo de [PowerShell](powershell-samples.md) o de la [CLI de Azure](cli-samples.md), o bien con [plantillas de Resource Manager](template-samples.md)
- Crear y aplicar una [directiva de Azure](policy-samples.md) para redes virtuales
