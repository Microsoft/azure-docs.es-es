---
title: 'Azure ExpressRoute: Configuración del emparejamiento: CLI'
description: Este artículo le ayudará a crear y aprovisionar las configuraciones entre pares privados, públicos y de Microsoft de un circuito ExpressRoute. Este artículo también muestra cómo comprobar el estado, actualizar, o eliminar configuraciones entre pares en el circuito.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 04/24/2019
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: c20b0741f12ee2cab40d8f5b53c51ad537801bf2
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89393180"
---
# <a name="create-and-modify-peering-for-an-expressroute-circuit-using-cli"></a>Creación y modificación del emparejamiento de un circuito ExpressRoute mediante la CLI

Este artículo le ayuda a crear y administrar la configuración de enrutamiento o el emparejamiento de un circuito ExpressRoute en el modelo de implementación de Resource Manager mediante la CLI. También puede comprobar el estado de los emparejamientos de un circuito ExpressRoute, así como el modo de actualizarlos o eliminarlos y desaprovisionarlos. Si quiere usar un método diferente para trabajar con el circuito, seleccione un artículo de la lista siguiente:

> [!div class="op_single_selector"]
> * [Azure Portal](expressroute-howto-routing-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-routing-arm.md)
> * [CLI de Azure](howto-routing-cli.md)
> * [Emparejamiento público](about-public-peering.md)
> * [Vídeo: pares privados](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-private-peering-for-your-expressroute-circuit)
> * [Vídeo: emparejamiento de Microsoft](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-microsoft-peering-for-your-expressroute-circuit)
> * [PowerShell (clásico)](expressroute-howto-routing-classic.md)
> 

## <a name="configuration-prerequisites"></a>Requisitos previos de configuración

* Antes de empezar, instale la versión más reciente de los comandos de la CLI (2.0 o posteriores). Para obtener más información sobre la instalación de los comandos de la CLI, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).
* Asegúrese de haber revisado los [requisitos previos](expressroute-prerequisites.md), los [requisitos de enrutamiento](expressroute-routing.md) y las páginas del [flujo de trabajo](expressroute-workflows.md) antes de comenzar la configuración.
* Tiene que tener un circuito ExpressRoute activo. Antes de continuar, siga las instrucciones para [crear un circuito ExpressRoute](howto-circuit-cli.md) y para que el proveedor de conectividad habilite el circuito. El circuito ExpressRoute debe estar en un estado habilitado y aprovisionado para poder ejecutar los comandos que se describen en este artículo.

Estas instrucciones se aplican solo a los circuitos creados con proveedores de servicios que ofrecen servicios de conectividad de capa 2. Si usa un proveedor de servicios que ofrece servicios administrados de nivel 3 (normalmente VPN IP, como MPLS), el mismo proveedor de conectividad configurará y administrará el enrutamiento.

Puede configurar el emparejamiento privado de Azure y de Microsoft para un circuito ExpressRoute (el emparejamiento público de Azure está en desuso para los nuevos circuitos). El emparejamiento se puede configurar en el orden que elija. Pero tiene que asegurarse de que completa cada configuración entre pares de una en una. Para más información sobre los emparejamientos y dominios de enrutamiento, vea [Dominios de enrutamiento de ExpressRoute](expressroute-circuit-peerings.md). Para obtener información sobre el emparejamiento público, consulte [Emparejamiento público de ExpressRoute](about-public-peering.md).

## <a name="microsoft-peering"></a><a name="msft"></a>Emparejamiento de Microsoft

Esta sección le ayuda a crear, obtener, actualizar y eliminar la configuración de emparejamiento de Microsoft para un circuito ExpressRoute.

> [!IMPORTANT]
> Se anunciarán todos los prefijos de servicio para el emparejamiento de Microsoft de los circuitos ExpressRoute que se configuraron antes del 1 de agosto de 2017, incluso si no se definen filtros de ruta. No se anunciará ningún prefijo para el emparejamiento de Microsoft de los circuitos ExpressRoute que se configuraron el 1 de agosto de 2017 o con posterioridad, hasta que se asocie un filtro de ruta al circuito. Para más información, consulte [Configuración de un filtro de ruta para el emparejamiento de Microsoft](how-to-routefilter-powershell.md).
> 
> 

### <a name="to-create-microsoft-peering"></a>Creación del emparejamiento de Microsoft

1. Instale la versión más reciente de la CLI de Azure. Use la versión más reciente de la interfaz de la línea de comandos (CLI) de Azure. Revise los [Requisitos previos y lista de comprobación de ExpressRoute](expressroute-prerequisites.md) y los [Flujos de trabajo de ExpressRoute para aprovisionamiento de circuitos y estados de circuitos de ExpressRoute](expressroute-workflows.md) antes de comenzar la configuración.

   ```azurecli
   az login
   ```

   Seleccione la suscripción para la que desea crear un circuito ExpressRoute.

   ```azurecli
   az account set --subscription "<subscription ID>"
   ```
2. Crear un circuito ExpressRoute. Siga las instrucciones para crear un [circuito ExpressRoute](howto-circuit-cli.md) y habilite su aprovisionamiento a través del proveedor de conectividad. Si el proveedor de conectividad ofrece servicios administrados de nivel 3, puede solicitar a su proveedor de conectividad que habilite para usted la configuración entre pares de Microsoft. En ese caso, no necesita seguir las instrucciones que aparecen en las secciones siguientes. Sin embargo, si el proveedor de conectividad no administra el enrutamiento por usted, después de crear el circuito, continúe con la configuración mediante los pasos siguientes. 

3. Compruebe el circuito ExpressRoute para asegurarse de que está aprovisionado y también habilitado. Utilice el ejemplo siguiente:

   ```azurecli
   az network express-route list
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
   "serviceProviderProvisioningState": "Provisioned",
   "sku": {
    "family": "UnlimitedData",
    "name": "Standard_MeteredData",
    "tier": "Standard"
   },
   "tags": null,
   "type": "Microsoft.Network/expressRouteCircuits]
   ```

4. Establezca la configuración del emparejamiento de Microsoft para el circuito. Asegúrese de que tiene la siguiente información antes de empezar:

   * Una subred /30 para el vínculo principal. Debe ser un prefijo de IPv4 público válido que sea de su propiedad y esté registrado en un Registro regional de Internet (RIR) o un Registro de enrutamiento de Internet (IRR).
   * Una subred /30 para el vínculo secundario. Debe ser un prefijo de IPv4 público válido que sea de su propiedad y esté registrado en un Registro regional de Internet (RIR) o un Registro de enrutamiento de Internet (IRR).
   * Un identificador VLAN válido para establecer esta configuración entre pares. Asegúrese de que ninguna otra configuración entre pares en el circuito usa el mismo identificador de VLAN.
   * Número de sistema autónomo (AS) para la configuración entre pares. Puede usar 2 bytes o 4 bytes como números AS.
   * Prefijos anunciados: tiene que proporcionar una lista de todos los prefijos que planea anunciar en la sesión BGP. Se aceptan solo prefijos de direcciones IP públicas. Si tiene pensado enviar un conjunto de prefijos, puede enviar una lista separada por comas. Estos prefijos tienen que estar registrados a su nombre en un Registro regional de Internet (RIR) o un Registro de enrutamiento de Internet (IRR).
   * **Opcional -** ASN de cliente: si anuncia prefijos que no están registrados en el número de AS de emparejamiento, puede especificar el número de AS en el que están registrados.
   * Nombre del enrutamiento del Registro: puede especificar el RIR o TIR en el que están registrados el número AS y los prefijos.
   * **Opcional:** un hash MD5 si elige usar uno.

   Ejecute el ejemplo siguiente para configurar el emparejamiento de Microsoft para el circuito:

   ```azurecli
   az network express-route peering create --circuit-name MyCircuit --peer-asn 100 --primary-peer-subnet 123.0.0.0/30 -g ExpressRouteResourceGroup --secondary-peer-subnet 123.0.0.4/30 --vlan-id 300 --peering-type MicrosoftPeering --advertised-public-prefixes 123.1.0.0/24
   ```

### <a name="to-view-microsoft-peering-details"></a><a name="getmsft"></a>Visualización de detalles del emparejamiento de Microsoft

Puede obtener detalles sobre la configuración mediante el ejemplo siguiente:

```azurecli
az network express-route peering show -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzureMicrosoftPeering
```
> [!IMPORTANT]
> Microsoft comprueba si los prefijos públicos anunciados y ASN del mismo nivel (o ASN de cliente) especificados se le han asignado en el registro de enrutamiento de Internet. Si obtiene los prefijos públicos de otra entidad y la asignación no se registra con el registro de enrutamiento, la validación automática no se completará y requerirá la validación manual. Si se produce un error en la validación automática, el valor de "AdvertisedPublicPrefixesState" que aparecerá en la salida del comando anterior será "Se necesita validación". 
> 
> Si ve el mensaje "Se necesita validación", recopile los documentos que muestren los prefijos públicos asignados a la organización por la entidad que aparece como propietaria de los prefijos en el registro de enrutamiento, y envíe estos documentos para su validación manual. Abra una incidencia de soporte técnico como se muestra a continuación. 
> 
>

La salida es similar a la del ejemplo siguiente:

```output
{
  "azureAsn": 12076,
  "etag": "W/\"2e97be83-a684-4f29-bf3c-96191e270666\"",
  "gatewayManagerEtag": "18",
  "id": "/subscriptions/9a0c2943-e0c2-4608-876c-e0ddffd1211b/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit/peerings/AzureMicrosoftPeering",
  "lastModifiedBy": "Customer",
  "microsoftPeeringConfig": {
    "advertisedPublicPrefixes": [
        ""
      ],
     "advertisedPublicPrefixesState": "",
     "customerASN": ,
     "routingRegistryName": ""
  }
  "name": "AzureMicrosoftPeering",
  "peerAsn": ,
  "peeringType": "AzureMicrosoftPeering",
  "primaryAzurePort": "",
  "primaryPeerAddressPrefix": "",
  "provisioningState": "Succeeded",
  "resourceGroup": "ExpressRouteResourceGroup",
  "routeFilter": null,
  "secondaryAzurePort": "",
  "secondaryPeerAddressPrefix": "",
  "sharedKey": null,
  "state": "Enabled",
  "stats": null,
  "vlanId": 100
}
```

### <a name="to-update-microsoft-peering-configuration"></a><a name="updatemsft"></a>Actualización de la configuración de emparejamiento de Microsoft

Puede actualizar cualquier parte de la configuración. En el siguiente ejemplo, los prefijos anunciados del circuito se están actualizando de 123.1.0.0/24 a 124.1.0.0/24:

```azurecli
az network express-route peering update --circuit-name MyCircuit -g ExpressRouteResourceGroup --peering-type MicrosoftPeering --advertised-public-prefixes 124.1.0.0/24
```

### <a name="to-add-ipv6-microsoft-peering-settings-to-an-existing-ipv4-configuration"></a><a name="addIPv6msft"></a>Adición de la configuración de IPv6 para el emparejamiento de Microsoft a una configuración existente de IPv4

```azurecli
az network express-route peering update -g ExpressRouteResourceGroup --circuit-name MyCircuit --peering-type MicrosoftPeering --ip-version ipv6 --primary-peer-subnet 2002:db00::/126 --secondary-peer-subnet 2003:db00::/126 --advertised-public-prefixes 2002:db00::/126
```

### <a name="to-delete-microsoft-peering"></a><a name="deletemsft"></a>Eliminación del emparejamiento de Microsoft

Puede quitar la configuración de emparejamiento mediante la ejecución del ejemplo siguiente:

```azurecli
az network express-route peering delete -g ExpressRouteResourceGroup --circuit-name MyCircuit --name MicrosoftPeering
```

## <a name="azure-private-peering"></a><a name="private"></a>Configuración entre pares privados de Azure

Esta sección le ayuda a crear, obtener, actualizar y eliminar la configuración de emparejamiento privado de Azure para un circuito ExpressRoute.

### <a name="to-create-azure-private-peering"></a>Creación de un emparejamiento privado de Azure

1. Instale la versión más reciente de la CLI de Azure. Debe usar la versión más reciente de la Interfaz de la línea de comandos de Azure (CLI)*. Revise los [requisitos previos](expressroute-prerequisites.md) y los [flujos de trabajo](expressroute-workflows.md) antes de comenzar la configuración.

   ```azurecli
   az login
   ```

   Seleccione la suscripción en la que desea crear un circuito ExpressRoute.

   ```azurecli
   az account set --subscription "<subscription ID>"
   ```
2. Crear un circuito ExpressRoute. Siga las instrucciones para crear un [circuito ExpressRoute](howto-circuit-cli.md) y habilite su aprovisionamiento a través del proveedor de conectividad. Si su proveedor de conectividad ofrece servicios administrados de nivel 3, puede solicitarle que habilite la configuración entre pares privados de Azure. En ese caso, no necesita seguir las instrucciones que aparecen en las secciones siguientes. Sin embargo, si el proveedor de conectividad no administra el enrutamiento por usted, después de crear el circuito, continúe con la configuración mediante los pasos siguientes.

3. Compruebe el circuito ExpressRoute para asegurarse de que está aprovisionado y también habilitado. Utilice el ejemplo siguiente:

   ```azurecli
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
   "serviceProviderProvisioningState": "Provisioned",
   "sku": {
    "family": "UnlimitedData",
    "name": "Standard_MeteredData",
    "tier": "Standard"
   },
   "tags": null,
   "type": "Microsoft.Network/expressRouteCircuits]
   ```

4. Establecimiento de la configuración entre pares privados de Azure para el circuito. Asegúrese de que tiene los elementos siguientes antes de continuar con los siguientes pasos:

   * Una subred /30 para el vínculo principal. La subred no debe ser parte de ningún espacio de direcciones reservado para redes virtuales.
   * Una subred /30 para el vínculo secundario. La subred no debe ser parte de ningún espacio de direcciones reservado para redes virtuales.
   * Un identificador VLAN válido para establecer esta configuración entre pares. Asegúrese de que ninguna otra configuración entre pares en el circuito usa el mismo identificador de VLAN.
   * Número de sistema autónomo (AS) para la configuración entre pares. Puede usar 2 bytes o 4 bytes como números AS. Puede usar un número AS privado para esta configuración entre pares. Asegúrese de que no usa 65515.
   * **Opcional:** un hash MD5 si elige usar uno.

   Use el ejemplo siguiente para configurar el emparejamiento privado de Azure para su circuito:

   ```azurecli
   az network express-route peering create --circuit-name MyCircuit --peer-asn 100 --primary-peer-subnet 10.0.0.0/30 -g ExpressRouteResourceGroup --secondary-peer-subnet 10.0.0.4/30 --vlan-id 200 --peering-type AzurePrivatePeering
   ```

   Si decide usar un hash MD5, use el ejemplo siguiente:

   ```azurecli
   az network express-route peering create --circuit-name MyCircuit --peer-asn 100 --primary-peer-subnet 10.0.0.0/30 -g ExpressRouteResourceGroup --secondary-peer-subnet 10.0.0.4/30 --vlan-id 200 --peering-type AzurePrivatePeering --SharedKey "A1B2C3D4"
   ```

   > [!IMPORTANT]
   > Asegúrese de especificar su número AS como ASN de configuración entre pares, no como cliente ASN.
   > 
   > 

### <a name="to-view-azure-private-peering-details"></a><a name="getprivate"></a>Visualización de los detalles del emparejamiento privado

Puede obtener detalles sobre la configuración mediante el ejemplo siguiente:

```azurecli
az network express-route peering show -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePrivatePeering
```

La salida es similar a la del ejemplo siguiente:

```output
{
  "azureAsn": 12076,
  "etag": "W/\"2e97be83-a684-4f29-bf3c-96191e270666\"",
  "gatewayManagerEtag": "18",
  "id": "/subscriptions/9a0c2943-e0c2-4608-876c-e0ddffd1211b/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit/peerings/AzurePrivatePeering",
  "ipv6PeeringConfig": null,
  "lastModifiedBy": "Customer",
  "microsoftPeeringConfig": null,
  "name": "AzurePrivatePeering",
  "peerAsn": 7671,
  "peeringType": "AzurePrivatePeering",
  "primaryAzurePort": "",
  "primaryPeerAddressPrefix": "",
  "provisioningState": "Succeeded",
  "resourceGroup": "ExpressRouteResourceGroup",
  "routeFilter": null,
  "secondaryAzurePort": "",
  "secondaryPeerAddressPrefix": "",
  "sharedKey": null,
  "state": "Enabled",
  "stats": null,
  "vlanId": 100
}
```

### <a name="to-update-azure-private-peering-configuration"></a><a name="updateprivate"></a>Actualización del establecimiento de configuración del emparejamiento privado de Azure

Puede actualizar cualquier parte de la configuración mediante el ejemplo siguiente. En este ejemplo, se va a actualizar el identificador de VLAN del circuito de 100 a 500.

```azurecli
az network express-route peering update --vlan-id 500 -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePrivatePeering
```

### <a name="to-delete-azure-private-peering"></a><a name="deleteprivate"></a>Eliminación del emparejamiento privado de Azure

Puede quitar la configuración de emparejamiento mediante la ejecución del ejemplo siguiente:

> [!WARNING]
> Debe asegurarse de que todas las conexiones de redes virtuales y de ExpressRoute Global Reach se eliminan antes de ejecutar este ejemplo. 
> 
> 

```azurecli
az network express-route peering delete -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePrivatePeering
```


## <a name="next-steps"></a>Pasos siguientes

Siguiente paso, [Vinculación de redes virtuales a circuitos ExpressRoute](howto-linkvnet-cli.md).

* Para más información sobre los flujos de trabajo de ExpressRoute, vea [Flujos de trabajo de ExpressRoute](expressroute-workflows.md).
* Para más información sobre el emparejamiento de circuitos, vea [Circuitos y dominios de enrutamiento de ExpressRoute](expressroute-circuit-peerings.md).
* Para más información sobre redes virtuales, vea [Información general sobre redes virtuales](../virtual-network/virtual-networks-overview.md).
