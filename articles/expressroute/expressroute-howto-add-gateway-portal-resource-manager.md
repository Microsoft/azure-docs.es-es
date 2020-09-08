---
title: 'Azure ExpressRoute: Incorporación de una puerta de enlace a una red virtual: portal'
description: Este artículo le guiará para agregar una puerta de enlace de red virtual a una red virtual de Administrador de recursos ya creada para ExpressRoute mediante Azure Portal.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 12/06/2018
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 06f7e5d28017ee618adfeeec52c6f1226e1ae82c
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89396371"
---
# <a name="configure-a-virtual-network-gateway-for-expressroute-using-the-azure-portal"></a>Configuración de una puerta de enlace de red virtual para ExpressRoute con Azure Portal
> [!div class="op_single_selector"]
> * [Resource Manager: Azure Portal](expressroute-howto-add-gateway-portal-resource-manager.md)
> * [Resource Manager - PowerShell](expressroute-howto-add-gateway-resource-manager.md)
> * [Clásico: PowerShell](expressroute-howto-add-gateway-classic.md)
> * [Vídeo: Azure Portal](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-vpn-gateway-for-your-virtual-network)
> 
> 

Este artículo lo guía por los pasos para agregar una puerta de enlace de red virtual para una red virtual existente. Este artículo lo guía por los pasos para agregar, cambiar el tamaño y quitar una puerta de enlace de red virtual (VNet) para una red virtual existente. Los pasos de esta configuración son específicos para las redes virtuales que se crearon con el modelo de implementación de Resource Manager que se utilizarán en una configuración ExpressRoute. Para obtener más información acerca de las puertas de enlace de red virtual y la configuración de puerta de enlace para ExpressRoute, consulte [Acerca de las puertas de enlace de red virtual para ExpressRoute](expressroute-about-virtual-network-gateways.md). 


## <a name="before-beginning"></a>Antes de comenzar

Los pasos de esta tarea usan una red virtual que se basa en los valores de la siguiente lista de referencia de configuración. Usamos esta lista en nuestros pasos de ejemplo. Puede copiar la lista para utilizarla como referencia y reemplazar los valores por los suyos propios.

**Lista de referencia de configuración**

* Nombre de red virtual = "TestVNet"
* Espacio de direcciones de red virtual = 192.168.0.0/16
* Nombre de subred = "FrontEnd" 
    * Espacio de direcciones de subred = "192.168.1.0/24"
* Grupo de recursos: "TestRG"
* Ubicación: = "Este de EE. UU."
* Nombre de subred de puerta de enlace: "GatewaySubnet" (siempre debe asignar a las subredes de puerta de enlace el nombre *GatewaySubnet*).
    * Espacio de direcciones de subred de puerta de enlace = "192.168.200.0/26"
* Nombre de puerta de enlace = "ERGW"
* Nombre de IP pública de puerta de enlace = "MyERGWVIP"
* Tipo de puerta de enlace = "ExpressRoute" (Este tipo es obligatorio para una configuración de ExpressRoute).

Puede ver un [vídeo](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-vpn-gateway-for-your-virtual-network) de estos pasos antes de comenzar la configuración.

## <a name="create-the-gateway-subnet"></a>Creación de la subred de la puerta de enlace

1. En el [portal](https://portal.azure.com), navegue a la red virtual de Resource Manager para la que desea crear una puerta de enlace de red virtual.
2. En la sección **Configuración** de la hoja de redes virtuales, haga clic en **Subredes** para expandir la hoja Subredes.
3. En la hoja **Subredes**, haga clic en **+Subred de puerta de enlace** para abrir la hoja **Agregar subred**. 
   
    ![Agregue la subred de puerta de enlace](./media/expressroute-howto-add-gateway-portal-resource-manager/addgwsubnet.png "Adición de la subred de puerta de enlace")


4. El **nombre** de la subred se rellena automáticamente con el valor "GatewaySubnet". Este valor es necesario para que Azure reconozca que se trata de subred de puerta de enlace. Modifique los valores de **Intervalo de direcciones** rellenados automáticamente para ajustarlos a sus requisitos de configuración. Se recomienda crear una subred de puerta de enlace con un /27 o superior (/ 26, / 25, etc.). Luego haga clic en **Aceptar** para guardar los valores y crear la subred de puerta de enlace.

    ![Adición de la subred](./media/expressroute-howto-add-gateway-portal-resource-manager/addsubnetgw.png "Adición de la subred")

## <a name="create-the-virtual-network-gateway"></a>Creación de la puerta de enlace de red virtual

1. En el portal, a la izquierda, haga clic en **+** y escriba "Puerta de enlace de red virtual" en el cuadro de búsqueda. Busque **Puerta de enlace de red virtual** en los resultados de la búsqueda y haga clic en la entrada. En la hoja **Puerta de enlace de red virtual**, haga clic en **Crear** en la parte inferior de la hoja. Se abrirá la hoja **Crear puerta de enlace de red virtual**.
2. En la hoja **Create virtual network gateway** (Crear puerta de enlace de red virtual), rellene los valores de la puerta de enlace de red virtual.

    ![Campos de la hoja crear puerta de enlace de red virtual](./media/expressroute-howto-add-gateway-portal-resource-manager/gw.png "Campos de la hoja Crear puerta de enlace de red virtual")
3. **Name**: Asigne un nombre a la puerta de enlace. Esta acción no es igual a la de asignación de un nombre a una subred de puerta de enlace. Este es el nombre del objeto de puerta de enlace que va a crear.
4. **Tipo de puerta de enlace**: seleccione **ExpressRoute**.
5. **SKU**: seleccione la SKU de puerta de enlace en la lista desplegable.
6. **Ubicación**: Ajuste el campo **Ubicación** para que apunte a la ubicación en la que se encuentra la red virtual. Si la ubicación no apunta a la región en que reside la red virtual, esta no aparece en la lista desplegable "Elegir una red virtual".
7. Elija la red virtual a la que quiera agregar esta puerta de enlace. Haga clic en **Virtual network** para abrir la hoja **Elegir una red virtual**. Seleccione la red virtual. Si no se muestra la red virtual, asegúrese de que el campo **Ubicación** apunta a la región en la que se encuentra la red virtual.
9. Elija una dirección IP pública. Haga clic en **Dirección de IP pública** para abrir la hoja **Elegir dirección IP pública**. Haga clic en **+Crear nueva** para abrir la hoja **Crear dirección IP pública**. Escriba un nombre para la dirección IP pública. Esta hoja crea un objeto de dirección IP pública al que se le asignará dinámicamente una dirección IP pública. Haga clic en **Aceptar** para guardar los cambios en esta hoja.
10. **Suscripción**: compruebe que se selecciona la suscripción correcta.
11. **Grupo de recursos**: este ajuste vendrá determinado por la red virtual que seleccione.
12. No ajuste la **ubicación** después de especificar la configuración anterior.
13. Compruebe la configuración. Si desea que la puerta de enlace aparezca en el panel, puede seleccionar **Anclar al panel** en la parte inferior de la hoja.
14. Haga clic en **Crear** para comenzar a crear la puerta de enlace. Se valida la configuración y se implementa la puerta de enlace. La creación de una puerta de enlace de red virtual puede tardar en completarse hasta 45 minutos.

## <a name="next-steps"></a>Pasos siguientes
Después de crear la puerta de enlace de red virtual, puede vincular la red virtual a un circuito ExpressRoute. Consulte el artículo [Vinculación de la red virtual a circuitos ExpressRoute](expressroute-howto-linkvnet-portal-resource-manager.md).
