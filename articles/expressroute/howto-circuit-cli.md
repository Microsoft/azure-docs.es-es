---
title: 'Creación y modificación de un circuito ExpressRoute: Azure CLI'
description: En este artículo se muestra cómo crear, aprovisionar, comprobar, actualizar, eliminar y desaprovisionar un circuito ExpressRoute con la CLI.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 11/13/2019
ms.author: duau
ms.custom: devx-track-azurecli
ms.openlocfilehash: 31d3c95ffe94d6ff4ea6deee2fe87d3132e4550e
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89393231"
---
# <a name="create-and-modify-an-expressroute-circuit-using-cli"></a>Creación y modificación de un circuito ExpressRoute mediante la CLI


En este artículo se describe cómo crear un circuito Azure ExpressRoute con la Interfaz de la línea de comandos (CLI). También se muestra cómo comprobar el estado, actualizar, o eliminar y desaprovisionar un circuito. Si quiere usar otro método para trabajar con los circuitos ExpressRoute, puede seleccionar el artículo en la lista siguiente:

> [!div class="op_single_selector"]
> * [Azure Portal](expressroute-howto-circuit-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-circuit-arm.md)
> * [CLI de Azure](howto-circuit-cli.md)
> * [Plantilla de Azure Resource Manager](expressroute-howto-circuit-resource-manager-template.md)
> * [Vídeo: Azure Portal](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-an-expressroute-circuit)
> * [PowerShell (clásico)](expressroute-howto-circuit-classic.md)
>

## <a name="before-you-begin"></a>Antes de empezar

* Antes de empezar, instale la versión más reciente de los comandos de la CLI (2.0 o posteriores). Para más información acerca de la instalación de los comandos de la CLI, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli) y [Introducción a la CLI de Azure](/cli/azure/get-started-with-azure-cli).
* Revise los [Requisitos previos y lista de comprobación de ExpressRoute](expressroute-prerequisites.md) y los [Flujos de trabajo de ExpressRoute para aprovisionamiento de circuitos y estados de circuitos de ExpressRoute](expressroute-workflows.md) antes de comenzar la configuración.

## <a name="create-and-provision-an-expressroute-circuit"></a><a name="create"></a>Crear y aprovisionar un circuito ExpressRoute

### <a name="1-sign-in-to-your-azure-account-and-select-your-subscription"></a>1. Iniciar sesión en la cuenta de Azure y seleccione la suscripción

Para empezar la configuración, inicie sesión en la cuenta de Azure. Si usa la funcionalidad "Try It" de CloudShell, iniciará sesión de manera automática. Use los siguientes ejemplos para conectarse:

```azurecli-interactive
az login
```

Compruebe las suscripciones para la cuenta.

```azurecli-interactive
az account list
```

Seleccione la suscripción para la que desea crear un circuito ExpressRoute.

```azurecli-interactive
az account set --subscription "<subscription ID>"
```

### <a name="2-get-the-list-of-supported-providers-locations-and-bandwidths"></a>2. Obtención de la lista de proveedores, ubicaciones y anchos de banda admitidos

Para crear un circuito ExpressRoute, necesita la lista de proveedores de conectividad, ubicaciones y opciones de ancho de banda admitidas. El comando de la CLI `az network express-route list-service-providers` devuelve esta información, que se usará en pasos posteriores:

```azurecli-interactive
az network express-route list-service-providers
```

La respuesta es similar al siguiente ejemplo:

```output
[
  {
    "bandwidthsOffered": [
      {
        "offerName": "50Mbps",
        "valueInMbps": 50
      },
      {
        "offerName": "100Mbps",
        "valueInMbps": 100
      },
      {
        "offerName": "200Mbps",
        "valueInMbps": 200
      },
      {
        "offerName": "500Mbps",
        "valueInMbps": 500
      },
      {
        "offerName": "1Gbps",
        "valueInMbps": 1000
      },
      {
        "offerName": "2Gbps",
        "valueInMbps": 2000
      },
      {
        "offerName": "5Gbps",
        "valueInMbps": 5000
      },
      {
        "offerName": "10Gbps",
        "valueInMbps": 10000
      }
    ],
    "id": "/subscriptions//resourceGroups//providers/Microsoft.Network/expressRouteServiceProviders/",
    "location": null,
    "name": "AARNet",
    "peeringLocations": [
      "Melbourne",
      "Sydney"
    ],
    "provisioningState": "Succeeded",
    "resourceGroup": "",
    "tags": null,
    "type": "Microsoft.Network/expressRouteServiceProviders"
  },
```

Revise la respuesta para ver si aparece su proveedor de conectividad. Tome nota de la información siguiente, la que necesitará cuando cree un circuito:

* Nombre
* PeeringLocations
* BandwidthsOffered

Ahora está listo para crear un circuito ExpressRoute.

### <a name="3-create-an-expressroute-circuit"></a>3. Creación de un circuito ExpressRoute

> [!IMPORTANT]
> El circuito ExpressRoute se factura a partir del momento en que se emite una clave de servicio. Haga esta operación cuando el proveedor de conectividad esté listo para aprovisionar el circuito.
>
>

Si todavía no tiene un grupo de recursos, debe crear uno antes de crear ExpressRoute. Puede crear uno si ejecuta el comando siguiente:

```azurecli-interactive
az group create -n ExpressRouteResourceGroup -l "West US"
```

En el ejemplo siguiente se muestra cómo crear un circuito ExpressRoute de 200 Mbps a través de Equinix en Silicon Valley. Si usa otro proveedor y otra configuración, sustituya esa información al realizar la solicitud.

Asegúrese de que especifica el nivel y la familia correctos de SKU.

* El nivel de SKU determina si un circuito ExpressRoute es [Local](expressroute-faqs.md#expressroute-local), Estándar o [Premium](expressroute-faqs.md#expressroute-premium). Puede especificar *Local*, *Estándar* o *Premium*.
* La familia de SKU determina el tipo de facturación. Puede seleccionar *Metereddata* para el plan de datos limitado y *Unlimiteddata* para el plan de datos ilimitado. Puede cambiar el tipo de facturación de *Metereddata* a *Unlimiteddata*, pero no se puede cambiar el tipo de *Unlimiteddata* a *Metereddata*. Un circuito *Local* solo es *Unlimiteddata*.


El circuito ExpressRoute se factura a partir del momento en que se emite una clave de servicio. El ejemplo siguiente es una solicitud para una nueva clave de servicio:

```azurecli-interactive
az network express-route create --bandwidth 200 -n MyCircuit --peering-location "Silicon Valley" -g ExpressRouteResourceGroup --provider "Equinix" -l "West US" --sku-family MeteredData --sku-tier Standard
```

La respuesta contiene la clave del servicio.

### <a name="4-list-all-expressroute-circuits"></a>4. Lista de todos los circuitos ExpressRoute

Para obtener una lista de todos los circuitos ExpressRoute que haya creado, ejecute el comando `az network express-route list`. Puede recuperar esta información en cualquier momento con este comando. Para mostrar todos los circuitos, realice la llamada sin parámetros.

```azurecli-interactive
az network express-route list
```

La clave de servicio aparece en el campo *ServiceKey* de la respuesta.

```output
"allowClassicOperations": false,
"authorizations": [],
"circuitProvisioningState": "Enabled",
"etag": "W/\"1262c492-ffef-4a63-95a8-a6002736b8c4\"",
"gatewayManagerEtag": null,
"id": "/subscriptions/81ab786c-56eb-4a4d-bb5f-f60329772466/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit",
"location": "westus",
"name": "MyCircuit",
"peerings": [],
"provisioningState": "Succeeded",
"resourceGroup": "ExpressRouteResourceGroup",
"serviceKey": "1d05cf70-1db5-419f-ad86-1ca62c3c125b",
"serviceProviderNotes": null,
"serviceProviderProperties": {
  "bandwidthInMbps": 200,
  "peeringLocation": "Silicon Valley",
  "serviceProviderName": "Equinix"
},
"serviceProviderProvisioningState": "NotProvisioned",
"sku": {
  "family": "UnlimitedData",
  "name": "Standard_MeteredData",
  "tier": "Standard"
},
"tags": null,
"type": "Microsoft.Network/expressRouteCircuits]
```

Puede ejecutar el comando con el parámetro "-h" para obtener descripciones detalladas de todos los parámetros.

```azurecli-interactive
az network express-route list -h
```

### <a name="5-send-the-service-key-to-your-connectivity-provider-for-provisioning"></a>5. Envío de la clave de servicio al proveedor de conectividad para el aprovisionamiento

"ServiceProviderProvisioningState" da información sobre el estado actual del aprovisionamiento en el lado del proveedor de servicios. El estado muestra el estado en el lado de Microsoft. Para más información, consulte el [artículo sobre flujos de trabajo](expressroute-workflows.md#expressroute-circuit-provisioning-states).

Cuando se crea un nuevo circuito ExpressRoute, dicho circuito tiene el siguiente estado:

```output
"serviceProviderProvisioningState": "NotProvisioned"
"circuitProvisioningState": "Enabled"
```

El circuito pasa al estado siguiente cuando el proveedor de conectividad se encuentra en el proceso de habilitarlo:

```output
"serviceProviderProvisioningState": "Provisioning"
"circuitProvisioningState": "Enabled"
```

Para poder usar un circuito ExpressRoute, dicho circuito tiene que estar en el siguiente estado.

```output
"serviceProviderProvisioningState": "Provisioned"
"circuitProvisioningState": "Enabled
```

### <a name="6-periodically-check-the-status-and-the-state-of-the-circuit-key"></a>6. Comprobación periódica del estado y la condición de la clave del circuito

La comprobación del estado y la condición de la clave de circuito le informa cuando el proveedor ha habilitado el circuito. Después de configurar el circuito, "ServiceProviderProvisioningState" aparece como "Provisioned", tal como se muestra en el ejemplo siguiente:

```azurecli-interactive
az network express-route show --resource-group ExpressRouteResourceGroup --name MyCircuit
```

La respuesta es similar al siguiente ejemplo:

```output
"allowClassicOperations": false,
"authorizations": [],
"circuitProvisioningState": "Enabled",
"etag": "W/\"1262c492-ffef-4a63-95a8-a6002736b8c4\"",
"gatewayManagerEtag": null,
"id": "/subscriptions/81ab786c-56eb-4a4d-bb5f-f60329772466/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit",
"location": "westus",
"name": "MyCircuit",
"peerings": [],
"provisioningState": "Succeeded",
"resourceGroup": "ExpressRouteResourceGroup",
"serviceKey": "1d05cf70-1db5-419f-ad86-1ca62c3c125b",
"serviceProviderNotes": null,
"serviceProviderProperties": {
  "bandwidthInMbps": 200,
  "peeringLocation": "Silicon Valley",
  "serviceProviderName": "Equinix"
},
"serviceProviderProvisioningState": "NotProvisioned",
"sku": {
  "family": "UnlimitedData",
  "name": "Standard_MeteredData",
  "tier": "Standard"
},
"tags": null,
"type": "Microsoft.Network/expressRouteCircuits]
```

### <a name="7-create-your-routing-configuration"></a>7. Creación de la configuración de enrutamiento

Consulte [Creación y modificación del enrutamiento de un circuito ExpressRoute mediante PowerShell](howto-routing-cli.md) para ver las instrucciones paso a paso.

> [!IMPORTANT]
> Estas instrucciones se aplican solo a los circuitos creados con proveedores de servicios que ofrecen servicios de conectividad de nivel 2. Si usa un proveedor de servicios que ofrece servicios administrados de nivel 3 (normalmente VPN IP, como MPLS), el mismo proveedor de conectividad configura y administra el enrutamiento.
>
>

### <a name="8-link-a-virtual-network-to-an-expressroute-circuit"></a>8. Vinculación de una red virtual a un circuito ExpressRoute

A continuación, vincule una red virtual a su circuito ExpressRoute. Use el artículo [Vinculación de redes virtuales a circuitos ExpressRoute](howto-linkvnet-cli.md).

## <a name="modifying-an-expressroute-circuit"></a><a name="modify"></a>Modificación de un circuito ExpressRoute

Puede modificar determinadas propiedades de un circuito ExpressRoute sin afectar a la conectividad. Puede realizar los cambios siguientes sin tiempo de inactividad:

* Puede habilitar o deshabilitar el complemento ExpressRoute Premium en su circuito ExpressRoute.
* Puede aumentar el ancho de banda del circuito ExpressRoute, siempre que haya capacidad disponible en el puerto. Sin embargo, no se admite la degradación del ancho de banda de un circuito.
* Puede cambiar el plan de medición de datos limitados a datos ilimitados. Sin embargo, no se admite cambiar el plan de medición de datos ilimitados a datos limitados.
* Puede habilitar y deshabilitar *Allow Classic Operations*(Permitir operaciones clásicas).

Para más información sobre los límites y las limitaciones, consulte las [preguntas más frecuentes de ExpressRoute](expressroute-faqs.md).

### <a name="to-enable-the-expressroute-premium-add-on"></a>Para habilitar el complemento ExpressRoute Premium

Puede habilitar el complemento ExpressRoute Premium en el circuito existente mediante el comando siguiente:

```azurecli-interactive
az network express-route update -n MyCircuit -g ExpressRouteResourceGroup --sku-tier Premium
```

El circuito ahora tiene habilitadas las características del complemento ExpressRoute Premium. La facturación de la capacidad del complemento Premium comenzará en cuanto el comando se ejecute correctamente.

### <a name="to-disable-the-expressroute-premium-add-on"></a>Para deshabilitar el complemento ExpressRoute Premium

> [!IMPORTANT]
> Esta operación puede producir un error si usa recursos que son más grandes de lo que está permitido para el circuito estándar.
>
>

Antes de deshabilitar el complemento premium de ExpressRoute, debe comprender los criterios siguientes:

* Antes de cambiar de premium a estándar, debe asegurarse de tener menos de diez redes virtuales vinculadas al circuito. Si tiene más de diez, se produce una solicitud de actualización y se le factura con las tarifas de nivel Premium.
* Tiene que desvincular todas las redes virtuales en otras regiones geopolíticas. Si no lo hace, se produce un error en la solicitud de actualización y se le factura con las tarifas de nivel Premium.
* La tabla de enrutamiento tiene que tener menos de 4.000 rutas para el emparejamiento entre pares privados. Si la tabla de rutas tiene más de 4000 rutas, se quita la sesión de BGP. La sesión no se volverá a habilitar hasta que el número de prefijos anunciados sea inferior a 4000.

Puede deshabilitar el complemento premium de ExpressRoute en el circuito existente mediante el ejemplo siguiente:

```azurecli-interactive
az network express-route update -n MyCircuit -g ExpressRouteResourceGroup --sku-tier Standard
```

### <a name="to-update-the-expressroute-circuit-bandwidth"></a>Para actualizar el ancho de banda del circuito ExpressRoute

Consulte la página de [preguntas más frecuentes de ExpressRoute](expressroute-faqs.md) para conocer las opciones de ancho de banda compatibles con su proveedor. Puede elegir cualquier tamaño mayor que el tamaño de su circuito existente.

> [!IMPORTANT]
> Si el puerto existente no tiene la capacidad adecuada, es posible que tenga que volver a crear el circuito ExpressRoute. El circuito no se puede actualizar si no hay más capacidad disponible en la ubicación.
>
> No podrá reducir el ancho de banda de un circuito ExpressRoute sin interrupciones. Para degradar un ancho de banda, es necesario desaprovisionar el circuito ExpressRoute y luego volver a aprovisionar un nuevo circuito ExpressRoute.
>

Cuando haya decidido el tamaño que necesita, use el comando siguiente para cambiar el tamaño del circuito:

```azurecli-interactive
az network express-route update -n MyCircuit -g ExpressRouteResourceGroup --bandwidth 1000
```

El circuito se cambia de tamaño en el lado de Microsoft. Luego, debe ponerse en contacto con su proveedor de conectividad para actualizar las configuraciones de su parte para que coincidan con este cambio. Después de realizar esta notificación, se le empieza a facturar por la opción de ancho de banda actualizada.

### <a name="to-move-the-sku-from-metered-to-unlimited"></a>Para pasar la SKU de limitada a ilimitada

Puede cambiar la SKU de un circuito ExpressRoute mediante el ejemplo siguiente:

```azurecli-interactive
az network express-route update -n MyCircuit -g ExpressRouteResourceGroup --sku-family UnlimitedData
```

### <a name="to-control-access-to-the-classic-and-resource-manager-environments"></a>Para controlar el acceso a los entornos de implementación clásica y del Resource Manager

Revise las instrucciones que se ofrecen en [Transición de los circuitos ExpressRoute desde el modelo de implementación clásica al modelo de implementación de Resource Manager](expressroute-howto-move-arm.md).

## <a name="deprovisioning-and-deleting-an-expressroute-circuit"></a><a name="delete"></a>Desaprovisionar y eliminar un circuito ExpressRoute

Para desaprovisionar y eliminar un circuito ExpressRoute, asegúrese de comprender los criterios siguientes:

* Tiene que desvincular todas las redes virtuales del circuito ExpressRoute. Si se produce un error en esta operación, compruebe si hay alguna red virtual vinculada al circuito.
* Si el estado de aprovisionamiento del proveedor de servicios del circuito ExpressRoute es **Aprovisionando** o **Aprovisionado**, debe colaborar con su proveedor de servicios para que desaprovisionen el circuito. Se le siguen reservando recursos y facturándole por ello hasta que el proveedor de servicios complete el desaprovisionamiento del circuito y nos lo notifique.
* Puede eliminar el circuito si el proveedor de servicios lo desaprovisionó. Cuando se desaprovisiona un circuito, el estado de aprovisionamiento del proveedor de servicios está establecido en **No aprovisionado**. Esto detiene la facturación del circuito.

Puede eliminar el circuito ExpressRout con la ejecución del siguiente comando:

```azurecli-interactive
az network express-route delete  -n MyCircuit -g ExpressRouteResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

Después de crear el circuito, asegúrese de hacer las tareas siguientes:

* [Crear y modificar el enrutamiento para el circuito ExpressRoute](howto-routing-cli.md)
* [Vincular la red virtual a su circuito ExpressRoute](howto-linkvnet-cli.md)
