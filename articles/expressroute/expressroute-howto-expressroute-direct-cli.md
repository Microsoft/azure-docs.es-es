---
title: 'Azure ExpressRoute: Configuración de ExpressRoute Direct: CLI'
description: En este artículo aprenderá a configurar ExpressRoute Direct mediante la CLI de Azure.
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: how-to
ms.date: 05/20/2019
ms.author: cherylmc
ms.custom: devx-track-azurecli
ms.openlocfilehash: 4cbacf997413c4eba1e8a142925cc6cde58fb0ec
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87488013"
---
# <a name="configure-expressroute-direct-by-using-the-azure-cli"></a>Configuración de ExpressRoute Direct mediante la CLI de Azure

Puede usar Azure ExpressRoute Direct para conectarse directamente a la red global de Microsoft en ubicaciones de emparejamiento distribuidas estratégicamente por todo el mundo. Para obtener más información, consulte [About ExpressRoute Direct Connect](expressroute-erdirect-about.md) (Acerca de ExpressRoute Direct Connect).

## <a name="create-the-resource"></a><a name="resources"></a>Crear el recurso

1. Inicie sesión en Azure y seleccione la suscripción que contiene ExpressRoute. El recurso de ExpressRoute Direct y sus circuitos de ExpressRoute deben estar en la misma suscripción. En la CLI de Azure, ejecute los siguientes comandos:

   ```azurecli
   az login
   ```

   Compruebe las suscripciones para la cuenta: 

   ```azurecli
   az account list 
   ```

   Seleccione la suscripción para la que desea crear un circuito de ExpressRoute:

   ```azurecli
   az account set --subscription "<subscription ID>"
   ```

2. Vuelva a registrar la suscripción a Microsoft.Network para acceder a las API expressrouteportslocation y expressrouteport.

   ```azurecli
   az provider register --namespace Microsoft.Network
   ```
3. Enumere todas las ubicaciones donde se admita ExpressRoute Direct:
    
   ```azurecli
   az network express-route port location list
   ```

   **Salida del ejemplo**
  
   ```output
   [
   {
    "address": "21715 Filigree Court, DC2, Building F, Ashburn, VA 20147",
    "availableBandwidths": [],
    "contact": "support@equinix.com",
    "id": "/subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-Ashburn-DC2",
    "location": null,
    "name": "Equinix-Ashburn-DC2",
    "provisioningState": "Succeeded",
    "tags": null,
    "type": "Microsoft.Network/expressRoutePortsLocations"
   },
   {
    "address": "1950 N. Stemmons Freeway, Suite 1039A, DA3, Dallas, TX 75207",
    "availableBandwidths": [],
    "contact": "support@equinix.com",
    "id": "/subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-Dallas-DA3",
    "location": null,
    "name": "Equinix-Dallas-DA3",
    "provisioningState": "Succeeded",
    "tags": null,
    "type": "Microsoft.Network/expressRoutePortsLocations"
   },
   {
    "address": "111 8th Avenue, New York, NY 10011",
    "availableBandwidths": [],
    "contact": "support@equinix.com",
    "id": "/subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-New-York-NY5",
    "location": null,
    "name": "Equinix-New-York-NY5",
    "provisioningState": "Succeeded",
    "tags": null,
    "type": "Microsoft.Network/expressRoutePortsLocations"
   },
   {
    "address": "11 Great Oaks Blvd, SV1, San Jose, CA 95119",
    "availableBandwidths": [],
    "contact": "support@equinix.com",
    "id": "/subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-San-Jose-SV1",
    "location": null,
    "name": "Equinix-San-Jose-SV1",
    "provisioningState": "Succeeded",
    "tags": null,
    "type": "Microsoft.Network/expressRoutePortsLocations"
   },
   {
    "address": "2001 Sixth Ave., Suite 350, SE2, Seattle, WA 98121",
    "availableBandwidths": [],
    "contact": "support@equinix.com",
    "id": "/subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-Seattle-SE2",
    "location": null,
    "name": "Equinix-Seattle-SE2",
    "provisioningState": "Succeeded",
    "tags": null,
    "type": "Microsoft.Network/expressRoutePortsLocations"
   }
   ]
   ```
4. Determine si alguna de las ubicaciones enumeradas en el paso anterior tiene ancho de banda disponible:

   ```azurecli
   az network express-route port location show -l "Equinix-Ashburn-DC2"
   ```

   **Salida del ejemplo**

   ```output
   {
   "address": "21715 Filigree Court, DC2, Building F, Ashburn, VA 20147",
   "availableBandwidths": [
    {
      "offerName": "100 Gbps",
      "valueInGbps": 100
    }
   ],
   "contact": "support@equinix.com",
   "id": "/subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-Ashburn-DC2",
   "location": null,
   "name": "Equinix-Ashburn-DC2",
   "provisioningState": "Succeeded",
   "tags": null,
   "type": "Microsoft.Network/expressRoutePortsLocations"
   }
   ```
5. Cree un recurso de ExpressRoute Direct situado en la ubicación que eligió en los pasos anteriores.

   ExpressRoute Direct admite la encapsulación de tipo QinQ y Dot1Q. Si selecciona QinQ, a cada circuito de ExpressRoute se le asigna dinámicamente una S-Tag y es única en todo el recurso de ExpressRoute Direct. Cada C-Tag del circuito debe ser única dentro del circuito, pero no en el recurso de ExpressRoute Direct.  

   Si selecciona la encapsulación Dot1Q, debe administrar la exclusividad de C-Tag (VLAN) en todo el recurso de ExpressRoute Direct.  

   > [!IMPORTANT]
   > ExpressRoute Direct solo puede tener un tipo de encapsulación. No se puede cambiar el tipo de encapsulación después de crear el recurso de ExpressRoute Direct.
   > 
 
   ```azurecli
   az network express-route port create -n $name -g $RGName --bandwidth 100 gbps  --encapsulation QinQ | Dot1Q --peering-location $PeeringLocationName -l $AzureRegion 
   ```

   > [!NOTE]
   > También puede establecer el atributo **Encapsulación** en **Dot1Q**. 
   >

   **Salida del ejemplo**

   ```output
   {
   "allocationDate": "Wednesday, October 17, 2018",
   "bandwidthInGbps": 100,
   "circuits": null,
   "encapsulation": "Dot1Q",
   "etag": "W/\"<etagnumber>\"",
   "etherType": "0x8100",
   "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct",
   "links": [
    {
      "adminState": "Disabled",
      "connectorType": "LC",
      "etag": "W/\"<etagnumber>\"",
      "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct/links/link1",
      "interfaceName": "HundredGigE2/2/2",
      "name": "link1",
      "patchPanelId": "PPID",
      "provisioningState": "Succeeded",
      "rackId": "RackID",
      "resourceGroup": "Contoso-Direct-rg",
      "routerName": "tst-09xgmr-cis-1",
      "type": "Microsoft.Network/expressRoutePorts/links"
    },
    {
      "adminState": "Disabled",
      "connectorType": "LC",
      "etag": "W/\"<etagnumber>\"",
      "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct/links/link2",
      "interfaceName": "HundredGigE2/2/2",
      "name": "link2",
      "patchPanelId": "PPID",
      "provisioningState": "Succeeded",
      "rackId": "RackID",
      "resourceGroup": "Contoso-Direct-rg",
      "routerName": "tst-09xgmr-cis-2",
      "type": "Microsoft.Network/expressRoutePorts/links"
    }
   ],
   "location": "westus",
   "mtu": "1500",
   "name": "Contoso-Direct",
   "peeringLocation": "Equinix-Ashburn-DC2",
   "provisionedBandwidthInGbps": 0.0,
   "provisioningState": "Succeeded",
   "resourceGroup": "Contoso-Direct-rg",
   "resourceGuid": "02ee21fe-4223-4942-a6bc-8d81daabc94f",
   "tags": null,
   "type": "Microsoft.Network/expressRoutePorts"
   }  
   ```

## <a name="change-adminstate-for-links"></a><a name="state"></a>Cambio de AdminState para vínculos

Utilice este proceso para llevar a cabo una prueba de capa 1. Asegúrese de que cada conexión cruzada se ha revisado correctamente en cada enrutador de los puertos principales y secundarios.

1. Establezca el estado de los vínculos en **Habilitado**. Repita este paso para establecer el estado de cada vínculo como **Habilitado**.

   Links[0] es el puerto principal, y Links[1] es el puerto secundario.

   ```azurecli
   az network express-route port update -n Contoso-Direct -g Contoso-Direct-rg --set links[0].adminState="Enabled"
   ```
   ```azurecli
   az network express-route port update -n Contoso-Direct -g Contoso-Direct-rg --set links[1].adminState="Enabled"
   ```
   **Salida del ejemplo**

   ```output
   {
   "allocationDate": "Wednesday, October 17, 2018",
   "bandwidthInGbps": 100,
   "circuits": null,
   "encapsulation": "Dot1Q",
   "etag": "W/\"<etagnumber>\"",
   "etherType": "0x8100",
   "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct",
   "links": [
    {
      "adminState": "Enabled",
      "connectorType": "LC",
      "etag": "W/\"<etagnumber>\"",
      "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct/links/link1",
      "interfaceName": "HundredGigE2/2/2",
      "name": "link1",
      "patchPanelId": "PPID",
      "provisioningState": "Succeeded",
      "rackId": "RackID",
      "resourceGroup": "Contoso-Direct-rg",
      "routerName": "tst-09xgmr-cis-1",
      "type": "Microsoft.Network/expressRoutePorts/links"
    },
    {
      "adminState": "Enabled",
      "connectorType": "LC",
      "etag": "W/\"<etagnumber>\"",
      "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct/links/link2",
      "interfaceName": "HundredGigE2/2/2",
      "name": "link2",
      "patchPanelId": "PPID",
      "provisioningState": "Succeeded",
      "rackId": "RackID",
      "resourceGroup": "Contoso-Direct-rg",
      "routerName": "tst-09xgmr-cis-2",
      "type": "Microsoft.Network/expressRoutePorts/links"
    }
   ],
   "location": "westus",
   "mtu": "1500",
   "name": "Contoso-Direct",
   "peeringLocation": "Equinix-Ashburn-DC2",
   "provisionedBandwidthInGbps": 0.0,
   "provisioningState": "Succeeded",
   "resourceGroup": "Contoso-Direct-rg",
   "resourceGuid": "<resourceGUID>",
   "tags": null,
   "type": "Microsoft.Network/expressRoutePorts"
   }
   ```

   Use el mismo procedimiento mediante `AdminState = "Disabled"` para deshabilitar los puertos.

## <a name="create-a-circuit"></a><a name="circuit"></a>Crear un circuito

De forma predeterminada, puede crear 10 circuitos en la suscripción que contiene el recurso ExpressRoute Direct. El Soporte técnico de Microsoft puede aumentar el límite predeterminado. Tenga en cuenta que debe realizar usted mismo el seguimiento tanto del ancho de banda aprovisionado como el del utilizado. El ancho de banda aprovisionado es la suma del ancho de banda de todos los circuitos en el recurso de ExpressRoute Direct. El ancho de banda utilizado es el uso físico de las interfaces físicas subyacentes.

Puede utilizar anchos de banda de circuito adicionales en ExpressRoute Direct solo para admitir los escenarios aquí descritos. Los anchos de banda son 40 Gbps y 100 Gbps.

**SkuTier** puede ser Local, Estándar o Premium.

**SkuFamily** deben ser datos limitados solo, ya los ilimitados no son compatibles con ExpressRoute Direct.
Cree un circuito en el recurso de ExpressRoute Direct:

  ```azurecli
  az network express-route create --express-route-port "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct" -n "Contoso-Direct-ckt" -g "Contoso-Direct-rg" --sku-family MeteredData --sku-tier Standard --bandwidth 100 Gbps
  ```

  Otros anchos de banda pueden ser 5 Gbps, 10 Gbps y 40 Gbps.

  **Salida del ejemplo**

  ```output
  {
  "allowClassicOperations": false,
  "allowGlobalReach": false,
  "authorizations": [],
  "bandwidthInGbps": 100.0,
  "circuitProvisioningState": "Enabled",
  "etag": "W/\"<etagnumber>\"",
  "expressRoutePort": {
    "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct",
    "resourceGroup": "Contoso-Direct-rg"
  },
  "gatewayManagerEtag": "",
  "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRouteCircuits/ERDirect-ckt-cli",
  "location": "westus",
  "name": "ERDirect-ckt-cli",
  "peerings": [],
  "provisioningState": "Succeeded",
  "resourceGroup": "Contoso-Direct-rg",
  "serviceKey": "<serviceKey>",
  "serviceProviderNotes": null,
  "serviceProviderProperties": null,
  "serviceProviderProvisioningState": "Provisioned",
  "sku": {
    "family": "MeteredData",
    "name": "Standard_MeteredData",
    "tier": "Standard"
  },
  "stag": null,
  "tags": null,
  "type": "Microsoft.Network/expressRouteCircuits"
  }  
  ```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre ExpressRoute Direct, consulte la [información general](expressroute-erdirect-about.md).
