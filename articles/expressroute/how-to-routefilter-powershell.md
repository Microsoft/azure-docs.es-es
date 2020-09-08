---
title: 'ExpressRoute: Filtros de ruta - emparejamiento de Microsoft: Azure PowerShell'
description: En este artículo se describe cómo configurar filtros de ruta para el emparejamiento de Microsoft mediante PowerShell
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 02/25/2019
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: aecc933b9ada1d1935d400809e06bb4dac06abb5
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89393316"
---
# <a name="configure-route-filters-for-microsoft-peering-powershell"></a>Configuración de filtros de ruta para el emparejamiento de Microsoft: PowerShell
> [!div class="op_single_selector"]
> * [Azure Portal](how-to-routefilter-portal.md)
> * [Azure PowerShell](how-to-routefilter-powershell.md)
> * [CLI de Azure](how-to-routefilter-cli.md)
> 

Los filtros de ruta son una forma de consumir un subconjunto de servicios admitidos mediante el emparejamiento de Microsoft. Los pasos que se describen en este artículo le ayudan a configurar y administrar filtros de ruta para circuitos ExpressRoute.

Mediante el emparejamiento de Microsoft se puede acceder a servicios de Office 365, como Exchange Online, SharePoint Online y Skype Empresarial, y a servicios públicos de Azure, como Storage y SQL DB. Los servicios públicos de Azure se pueden seleccionar región a región y no se pueden definir por servicio público.

Cuando se configura el emparejamiento de Microsoft en un circuito ExpressRoute y se anexa un filtro de ruta, todos los prefijos seleccionados para estos servicios se anuncian mediante las sesiones BGP establecidas. A cada prefijo se asocia un valor de la comunidad de BGP para identificar el servicio que se ofrece mediante el prefijo en cuestión. Para ver una lista de valores de la comunidad de BGP y los servicios a los que se asignan, consulte [Comunidades de BGP](expressroute-routing.md#bgp).

Si necesita conectividad con todos los servicios, se anuncia un gran número de prefijos mediante BGP. Como consecuencia, aumenta considerablemente el tamaño de las tablas de rutas que mantienen los enrutadores dentro de la red. Si tiene previsto consumir solo un subconjunto de los servicios ofrecidos mediante el emparejamiento de Microsoft, puede reducir el tamaño de las tablas de rutas de dos maneras. Puede:

- Filtrar los prefijos no deseados mediante la aplicación de filtros de ruta en las comunidades de BGP. Este es un procedimiento de red estándar y se usa normalmente en muchas redes.

- Definir filtros de ruta y aplicarlos al circuito ExpressRoute. Un filtro de ruta es un nuevo recurso que le permite seleccionar la lista de servicios que se va a consumir mediante el emparejamiento de Microsoft. Los enrutadores de ExpressRoute solo envían la lista de prefijos que pertenecen a los servicios identificados en el filtro de ruta.

### <a name="about-route-filters"></a><a name="about"></a>Acerca de los filtros de ruta

Cuando se configura el emparejamiento de Microsoft en el circuito ExpressRoute, los enrutadores perimetrales de red de Microsoft establecen un par de sesiones BGP con los enrutadores perimetrales suyos o del proveedor de conectividad. No se anuncia ningún enrutador en la red. Para habilitar los anuncios de rutas en la red, debe asociar un filtro de ruta.

Un filtro de ruta permite identificar los servicios que desea consumir mediante el emparejamiento de Microsoft del circuito ExpressRoute. Se trata básicamente de una lista de todos los valores permitidos de la comunidad de BGP. Una vez que se define un recurso de filtro de ruta y se asocia a un circuito ExpressRoute, todos los prefijos que se asignan a valores de la comunidad de BGP se anuncian en la red.

Para poder asociar filtros de ruta con servicios de Office 365 en ellos, debe tener autorización para consumir servicios de Office 365 mediante ExpressRoute. Si no está autorizado para consumir servicios de Office 365 mediante ExpressRoute, la operación para asociar filtros de ruta de produce un error. Para más información sobre el proceso de autorización, consulte [Azure ExpressRoute para Office 365](https://support.office.com/article/Azure-ExpressRoute-for-Office-365-6d2534a2-c19c-4a99-be5e-33a0cee5d3bd).

> [!IMPORTANT]
> Se anunciarán todos los prefijos de servicio para el emparejamiento de Microsoft de los circuitos ExpressRoute que se configuraron antes del 1 de agosto de 2017, incluso si no se definen filtros de ruta. No se anunciará ningún prefijo para el emparejamiento de Microsoft de los circuitos ExpressRoute que se configuraron el 1 de agosto de 2017 o con posterioridad, hasta que se asocie un filtro de ruta al circuito.
> 
> 

### <a name="workflow"></a><a name="workflow"></a>Flujo de trabajo

Para poder conectarse correctamente a los servicios mediante el emparejamiento de Microsoft, debe realizar los siguientes pasos de configuración:

- Debe tener un circuito ExpressRoute activo que tenga el emparejamiento de Microsoft aprovisionado. Para realizar estas tareas, puede usar las siguientes instrucciones:
  - [Cree un circuito ExpressRoute](expressroute-howto-circuit-arm.md) y pida a su proveedor de conectividad que se lo habilite antes de continuar. El circuito ExpressRoute debe estar en estado habilitado y aprovisionado.
  - Si administra la sesión BGP directamente, [cree el emparejamiento de Microsoft](expressroute-circuit-peerings.md). O bien, pida a su proveedor de conectividad que aprovisione el emparejamiento de Microsoft para su circuito.

-  Debe crear y configurar un filtro de ruta.
    - Identifique los servicios que quiere consumir mediante el emparejamiento de Microsoft.
    - Identifique la lista de valores de la comunidad de BGP asociados a los servicios.
    - Cree una regla para permitir que la lista de prefijos coincida con los valores de la comunidad de BGP

-  Debe asociar el filtro de ruta al circuito ExpressRoute.

## <a name="before-you-begin"></a>Antes de empezar

Antes de comenzar la configuración, asegúrese de que cumple los siguientes criterios:

 - Revise los [Requisitos previos y lista de comprobación de ExpressRoute](expressroute-prerequisites.md) y los [Flujos de trabajo de ExpressRoute para aprovisionamiento de circuitos y estados de circuitos de ExpressRoute](expressroute-workflows.md) antes de comenzar la configuración.

 - Tiene que tener un circuito ExpressRoute activo. Antes de continuar, siga las instrucciones para [crear un circuito ExpressRoute](expressroute-howto-circuit-arm.md) y para que el proveedor de conectividad habilite el circuito. El circuito ExpressRoute debe estar en estado habilitado y aprovisionado.

 - Debe tener un emparejamiento de Microsoft activo. Siga las instrucciones que encontrará en el artículo [Creación y modificación de la configuración de emparejamiento](expressroute-circuit-peerings.md).


### <a name="working-with-azure-powershell"></a>Trabajo con Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

[!INCLUDE [expressroute-cloudshell](../../includes/expressroute-cloudshell-powershell-about.md)]

### <a name="log-in-to-your-azure-account"></a>Iniciar sesión en su cuenta

Antes de comenzar esta configuración, debe iniciar sesión en su cuenta de Azure. El cmdlet pide las credenciales de inicio de sesión para la cuenta de Azure. Después de iniciar la sesión, se descarga la configuración de la cuenta a fin de que esté disponible para Azure PowerShell.

Abra la consola de PowerShell con privilegios elevados y conéctese a su cuenta. Use el siguiente ejemplo para conectarse. Si usa Azure Cloud Shell, no es necesario ejecutar este cmdlet, ya que la sesión se iniciará automáticamente.

```azurepowershell
Connect-AzAccount
```

Si tiene varias suscripciones de Azure, compruebe las suscripciones de la cuenta.

```azurepowershell-interactive
Get-AzSubscription
```

Especifique la suscripción que desea usar.

```azurepowershell-interactive
Select-AzSubscription -SubscriptionName "Replace_with_your_subscription_name"
```

## <a name="step-1-get-a-list-of-prefixes-and-bgp-community-values"></a><a name="prefixes"></a>Paso 1: Obtención de una lista de prefijos y valores de la comunidad de BGP

### <a name="1-get-a-list-of-bgp-community-values"></a>1. Obtención de una lista de valores de la comunidad de BGP

Use el siguiente cmdlet para obtener la lista de valores de la comunidad de BGP asociados con los servicios accesibles mediante emparejamiento de Microsoft, junto con la lista de prefijos asociados a ellos:

```azurepowershell-interactive
Get-AzBgpServiceCommunity
```
### <a name="2-make-a-list-of-the-values-that-you-want-to-use"></a>2. Creación de una lista de los valores que quiera usar

Cree una lista de valores de la comunidad de BGP que quiera usar en el filtro de ruta.

## <a name="step-2-create-a-route-filter-and-a-filter-rule"></a><a name="filter"></a>Paso 2: Creación de un filtro de ruta y una regla de filtro

Un filtro de ruta puede tener una única regla y la regla debe ser de tipo "Permitir". Esta regla puede tener una lista de valores de la comunidad de BGP asociados a ella.

### <a name="1-create-a-route-filter"></a>1. Creación de un filtro de ruta

En primer lugar, cree el filtro de ruta. El comando "New-AzRouteFilter" solamente crea un recurso de filtro de ruta. Después de crear el recurso, debe crear una regla y asociarla al objeto de filtro de ruta. Ejecute el comando siguiente para crear un recurso de filtro de ruta:

```azurepowershell-interactive
New-AzRouteFilter -Name "MyRouteFilter" -ResourceGroupName "MyResourceGroup" -Location "West US"
```

### <a name="2-create-a-filter-rule"></a>2. Creación de una regla de filtro

Puede especificar un conjunto de comunidades de BGP como una lista separada por comas, como se muestra en el ejemplo. Ejecute el siguiente comando para crear una nueva regla:
 
```azurepowershell-interactive
$rule = New-AzRouteFilterRuleConfig -Name "Allow-EXO-D365" -Access Allow -RouteFilterRuleType Community -CommunityList 12076:5010,12076:5040
```

### <a name="3-add-the-rule-to-the-route-filter"></a>3. Adición de la regla al filtro de ruta

Ejecute el siguiente comando para agregar la regla de filtro al filtro de ruta:
 
```azurepowershell-interactive
$routefilter = Get-AzRouteFilter -Name "RouteFilterName" -ResourceGroupName "ExpressRouteResourceGroupName"
$routefilter.Rules.Add($rule)
Set-AzRouteFilter -RouteFilter $routefilter
```

## <a name="step-3-attach-the-route-filter-to-an-expressroute-circuit"></a><a name="attach"></a>Paso 3: Asociación del filtro de ruta a un circuito ExpressRoute

Ejecute el siguiente comando para asociar el filtro de ruta al circuito ExpressRoute (se da por hecho que solo tiene emparejamiento de Microsoft):

```azurepowershell-interactive
$ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
$ckt.Peerings[0].RouteFilter = $routefilter 
Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="common-tasks"></a><a name="tasks"></a>Tareas comunes

### <a name="to-get-the-properties-of-a-route-filter"></a><a name="getproperties"></a>Obtención de las propiedades de un filtro de ruta

Para obtener las propiedades de un filtro de ruta, siga estos pasos:

1. Ejecute el comando siguiente para obtener el recurso de filtro de ruta:

   ```azurepowershell-interactive
   $routefilter = Get-AzRouteFilter -Name "RouteFilterName" -ResourceGroupName "ExpressRouteResourceGroupName"
   ```
2. Obtenga las reglas de filtro de la ruta para el recurso de filtro de ruta mediante la ejecución del comando siguiente:

   ```azurepowershell-interactive
   $routefilter = Get-AzRouteFilter -Name "RouteFilterName" -ResourceGroupName "ExpressRouteResourceGroupName"
   $rule = $routefilter.Rules[0]
   ```

### <a name="to-update-the-properties-of-a-route-filter"></a><a name="updateproperties"></a>Actualización de las propiedades de un filtro de ruta

Si el filtro de ruta ya está asociado a un circuito, las actualizaciones de la lista de la comunidad de BGP propagan automáticamente los cambios en los anuncios de prefijos adecuados mediante las sesiones BGP establecidas. Puede actualizar la lista de la comunidad de BGP de su filtro de ruta con el comando siguiente:

```azurepowershell-interactive
$routefilter = Get-AzRouteFilter -Name "RouteFilterName" -ResourceGroupName "ExpressRouteResourceGroupName"
$routefilter.rules[0].Communities = "12076:5030", "12076:5040"
Set-AzRouteFilter -RouteFilter $routefilter
```

### <a name="to-detach-a-route-filter-from-an-expressroute-circuit"></a><a name="detach"></a>Desasociación de un filtro de ruta de un circuito ExpressRoute

Una vez que un filtro de ruta se desasocia del circuito ExpressRoute, no se anuncia ningún prefijo mediante la sesión BGP. Puede desasociar un filtro de ruta de un circuito ExpressRoute con el comando siguiente:
  
```azurepowershell-interactive
$ckt.Peerings[0].RouteFilter = $null
Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

### <a name="to-delete-a-route-filter"></a><a name="delete"></a>Eliminación de un filtro de ruta

Solo se puede eliminar un filtro de ruta si no está asociado a ningún circuito. Asegúrese de que el filtro de ruta no esté asociado a ningún circuito antes de intentar eliminarlo. Puede eliminar un filtro de ruta con el comando siguiente:

```azurepowershell-interactive
Remove-AzRouteFilter -Name "MyRouteFilter" -ResourceGroupName "MyResourceGroup"
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de ExpressRoute, consulte [P+F de ExpressRoute](expressroute-faqs.md).
