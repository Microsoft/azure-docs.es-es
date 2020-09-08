---
title: 'ExpressRoute: Filtros de ruta: emparejamiento de Microsoft: Azure Portal'
description: En este artículo se describe cómo configurar los filtros de ruta para el emparejamiento de Microsoft mediante Azure Portal.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 07/01/2019
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: f65538d7946090229f163680f21c13031e323445
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89393418"
---
# <a name="configure-route-filters-for-microsoft-peering-azure-portal"></a>Configuración de filtros de ruta para el emparejamiento de Microsoft: Portal de Azure
> [!div class="op_single_selector"]
> * [Azure Portal](how-to-routefilter-portal.md)
> * [Azure PowerShell](how-to-routefilter-powershell.md)
> * [CLI de Azure](how-to-routefilter-cli.md)
> 

Los filtros de ruta son una forma de consumir un subconjunto de servicios admitidos mediante el emparejamiento de Microsoft. Los pasos que se describen en este artículo le ayudan a configurar y administrar filtros de ruta para circuitos ExpressRoute.

Mediante el emparejamiento de Microsoft se puede acceder a servicios de Office 365, como Exchange Online, SharePoint Online y Skype Empresarial, y a servicios de Azure, como Storage y SQL DB. Cuando se configura el emparejamiento de Microsoft en un circuito ExpressRoute, todos los prefijos relacionados con estos servicios se anuncian mediante las sesiones BGP establecidas. A cada prefijo se asocia un valor de la comunidad de BGP para identificar el servicio que se ofrece mediante el prefijo en cuestión. Para ver una lista de valores de la comunidad de BGP y los servicios a los que se asignan, consulte [Comunidades de BGP](expressroute-routing.md#bgp).

Si necesita conectividad con todos los servicios, se anuncia un gran número de prefijos mediante BGP. Como consecuencia, aumenta considerablemente el tamaño de las tablas de rutas que mantienen los enrutadores dentro de la red. Si tiene previsto consumir solo un subconjunto de los servicios ofrecidos mediante el emparejamiento de Microsoft, puede reducir el tamaño de las tablas de rutas de dos maneras. Puede:

- Filtrar los prefijos no deseados mediante la aplicación de filtros de ruta en las comunidades de BGP. Este es un procedimiento de red estándar y se usa normalmente en muchas redes.

- Definir filtros de ruta y aplicarlos al circuito ExpressRoute. Un filtro de ruta es un nuevo recurso que le permite seleccionar la lista de servicios que se va a consumir mediante el emparejamiento de Microsoft. Los enrutadores de ExpressRoute solo envían la lista de prefijos que pertenecen a los servicios identificados en el filtro de ruta.

### <a name="about-route-filters"></a><a name="about"></a>Acerca de los filtros de ruta

Cuando se configura el emparejamiento de Microsoft en el circuito ExpressRoute, los enrutadores perimetrales de Microsoft establecen un par de sesiones BGP con los enrutadores perimetrales suyos o del proveedor de conectividad. No se anuncia ningún enrutador en la red. Para habilitar los anuncios de rutas en la red, debe asociar un filtro de ruta.

Un filtro de ruta permite identificar los servicios que desea consumir mediante el emparejamiento de Microsoft del circuito ExpressRoute. Se trata básicamente de una lista de todos los valores de la comunidad de BGP que desea permitir. Una vez que se define un recurso de filtro de ruta y se asocia a un circuito ExpressRoute, todos los prefijos que se asignan a valores de la comunidad de BGP se anuncian en la red.

Para poder asociar filtros de ruta con servicios de Office 365 en ellos, debe tener autorización para consumir servicios de Office 365 mediante ExpressRoute. Si no está autorizado para consumir servicios de Office 365 mediante ExpressRoute, la operación para asociar filtros de ruta de produce un error. Para más información sobre el proceso de autorización, consulte [Azure ExpressRoute para Office 365](https://support.office.com/article/Azure-ExpressRoute-for-Office-365-6d2534a2-c19c-4a99-be5e-33a0cee5d3bd).

> [!IMPORTANT]
> Se anunciarán todos los prefijos de servicio para el emparejamiento de Microsoft de los circuitos ExpressRoute que se configuraron antes del 1 de agosto de 2017, incluso si no se definen filtros de ruta. No se anunciará ningún prefijo para el emparejamiento de Microsoft de los circuitos ExpressRoute que se configuraron el 1 de agosto de 2017 o con posterioridad, hasta que se asocie un filtro de ruta al circuito.
> 
> 

### <a name="workflow"></a><a name="workflow"></a>Flujo de trabajo

Para poder conectarse correctamente a los servicios mediante el emparejamiento de Microsoft, debe realizar los siguientes pasos de configuración:

- Debe tener un circuito ExpressRoute activo que tenga el emparejamiento de Microsoft aprovisionado. Para realizar estas tareas, puede usar las siguientes instrucciones:
  - [Cree un circuito ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md) y pida a su proveedor de conectividad que se lo habilite antes de continuar. El circuito ExpressRoute debe estar en estado habilitado y aprovisionado.
  - Si administra la sesión BGP directamente, [cree el emparejamiento de Microsoft](expressroute-howto-routing-portal-resource-manager.md). O bien, pida a su proveedor de conectividad que aprovisione el emparejamiento de Microsoft para su circuito.

-  Debe crear y configurar un filtro de ruta.
    - Identifique los servicios que quiere consumir mediante el emparejamiento de Microsoft.
    - Identifique la lista de valores de la comunidad de BGP asociados a los servicios.
    - Cree una regla para permitir que la lista de prefijos coincida con los valores de la comunidad de BGP

-  Debe asociar el filtro de ruta al circuito ExpressRoute.

## <a name="before-you-begin"></a>Antes de empezar

Antes de comenzar la configuración, asegúrese de que cumple los siguientes criterios:

 - Revise los [Requisitos previos y lista de comprobación de ExpressRoute](expressroute-prerequisites.md) y los [Flujos de trabajo de ExpressRoute para aprovisionamiento de circuitos y estados de circuitos de ExpressRoute](expressroute-workflows.md) antes de comenzar la configuración.

 - Tiene que tener un circuito ExpressRoute activo. Antes de continuar, siga las instrucciones para [crear un circuito ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md) y para que el proveedor de conectividad habilite el circuito. El circuito ExpressRoute debe estar en estado habilitado y aprovisionado.

 - Debe tener un emparejamiento de Microsoft activo. Siga las instrucciones que encontrará en [Creación y modificación de la configuración de emparejamiento](expressroute-howto-routing-portal-resource-manager.md).


## <a name="step-1-get-a-list-of-prefixes-and-bgp-community-values"></a><a name="prefixes"></a>Paso 1: Obtención de una lista de prefijos y valores de la comunidad de BGP

### <a name="1-get-a-list-of-bgp-community-values"></a>1. Obtención de una lista de valores de la comunidad de BGP

Los valores de la comunidad de BGP asociados con los servicios accesibles a través de emparejamiento de Microsoft están disponibles en la página [Requisitos de enrutamiento de ExpressRoute](expressroute-routing.md).

### <a name="2-make-a-list-of-the-values-that-you-want-to-use"></a>2. Creación de una lista de los valores que quiera usar

Cree una lista de [valores de la comunidad de BGP](expressroute-routing.md#bgp) que quiera usar en el filtro de ruta. 

## <a name="step-2-create-a-route-filter-and-a-filter-rule"></a><a name="filter"></a>Paso 2: Creación de un filtro de ruta y una regla de filtro

Un filtro de ruta puede tener una única regla y la regla debe ser de tipo "Permitir". Esta regla puede tener una lista de valores de la comunidad de BGP asociados a ella.

### <a name="1-create-a-route-filter"></a>1. Creación de un filtro de ruta
Puede crear un filtro de ruta seleccionando la opción de creación de un nuevo recurso. Haga clic en **Crear un recurso** > **Redes** > **RouteFilter**, tal y como se muestra en la siguiente imagen:

![Creación de un filtro de ruta](./media/how-to-routefilter-portal/CreateRouteFilter1.png)

Debe colocar el filtro de ruta en un grupo de recursos. 

![Creación de un filtro de ruta](./media/how-to-routefilter-portal/CreateRouteFilter.png)

### <a name="2-create-a-filter-rule"></a>2. Creación de una regla de filtro

Para agregar y actualizar las reglas, seleccione la pestaña de administración de reglas del filtro de ruta.

![Creación de un filtro de ruta](./media/how-to-routefilter-portal/ManageRouteFilter.png)


Puede seleccionar los servicios a los que desea conectarse desde la lista desplegable y guardar la regla cuando haya finalizado.

![Creación de un filtro de ruta](./media/how-to-routefilter-portal/AddRouteFilterRule.png)


## <a name="step-3-attach-the-route-filter-to-an-expressroute-circuit"></a><a name="attach"></a>Paso 3: Asociación del filtro de ruta a un circuito ExpressRoute

Puede adjuntar el filtro de ruta a un circuito seleccionando el botón "Agregar circuito" y seleccionando el circuito de ExpressRoute en la lista desplegable.

![Creación de un filtro de ruta](./media/how-to-routefilter-portal/AddCktToRouteFilter.png)

Si el proveedor de conectividad configura emparejamiento para su circuito de ExpressRoute, actualice el circuito desde la hoja de circuito de ExpressRoute antes de seleccionar el botón "Agregar circuito".

![Creación de un filtro de ruta](./media/how-to-routefilter-portal/RefreshExpressRouteCircuit.png)

## <a name="common-tasks"></a><a name="tasks"></a>Tareas comunes

### <a name="to-get-the-properties-of-a-route-filter"></a><a name="getproperties"></a>Obtención de las propiedades de un filtro de ruta

Puede ver propiedades de un filtro de ruta cuando abra el recurso en el portal.

![Creación de un filtro de ruta](./media/how-to-routefilter-portal/ViewRouteFilter.png)


### <a name="to-update-the-properties-of-a-route-filter"></a><a name="updateproperties"></a>Actualización de las propiedades de un filtro de ruta

Puede actualizar la lista de valores de la comunidad de BGP adjunta a un circuito seleccionando el botón "Administrar regla".


![Creación de un filtro de ruta](./media/how-to-routefilter-portal/ManageRouteFilter.png)

![Creación de un filtro de ruta](./media/how-to-routefilter-portal/AddRouteFilterRule.png) 


### <a name="to-detach-a-route-filter-from-an-expressroute-circuit"></a><a name="detach"></a>Desasociación de un filtro de ruta de un circuito ExpressRoute

Para desasociar un circuito del filtro de ruta, haga clic con el botón derecho en el circuito y haga clic en "desasociar".

![Creación de un filtro de ruta](./media/how-to-routefilter-portal/DetachRouteFilter.png) 


### <a name="to-delete-a-route-filter"></a><a name="delete"></a>Eliminación de un filtro de ruta

Para eliminar un filtro de ruta, seleccione el botón Eliminar. 

![Creación de un filtro de ruta](./media/how-to-routefilter-portal/DeleteRouteFilter.png) 

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información acerca de ExpressRoute, consulte [P+F de ExpressRoute](expressroute-faqs.md).

* Para más información acerca de los ejemplos de configuración del enrutador, consulte [Ejemplos de configuración de enrutadores para configurar y administrar enrutamiento](expressroute-config-samples-routing.md). 
