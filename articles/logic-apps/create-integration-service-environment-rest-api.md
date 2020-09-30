---
title: Creación de entornos del servicio de integración (ISE) en la API REST de Logic Apps
description: Creación de un entorno del servicio de integración (ISE) mediante la API REST de Logic Apps para acceder a las redes virtuales de Azure desde Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: rarayudu, logicappspm
ms.topic: conceptual
ms.date: 05/29/2020
ms.openlocfilehash: 427b488fe6673bef505fccdaa7185d69437bceaf
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89231323"
---
# <a name="create-an-integration-service-environment-ise-by-using-the-logic-apps-rest-api"></a>Creación de un entorno del servicio de integración (ISE) mediante la API REST de Logic Apps

En este artículo se muestra cómo crear un [*entorno del servicio de integración* (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) mediante la API REST de Logic Apps para escenarios donde sus cuentas de integración y las aplicaciones lógicas necesitan tener acceso a una [red virtual de Azure](../virtual-network/virtual-networks-overview.md). Un ISE es un entorno dedicado que usa almacenamiento dedicado y otros recursos que existen de forma independiente del servicio Logic Apps multiinquilino "global". Esta separación también reduce los posibles efectos que podrían tener otros inquilinos de Azure en el rendimiento de la aplicación. Un ISE también le proporciona sus propias direcciones IP estáticas. Estas direcciones IP son independientes de las direcciones IP estáticas que comparten las aplicaciones lógicas en el servicio multiinquilino público.

También puede crear un ISE mediante el [ejemplo de plantilla de inicio rápido de Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/201-integration-service-environment) o con [Azure Portal](../logic-apps/connect-virtual-network-vnet-isolated-environment.md).

> [!IMPORTANT]
> Las aplicaciones lógicas, los desencadenadores integrados, las acciones integradas y los conectores que se ejecutan en el ISE usan un plan de tarifa diferente al plan de tarifa basado en el consumo. Para saber cómo funcionan los precios y la facturación para los ISE, consulte [Modelo de precios de Logic Apps](../logic-apps/logic-apps-pricing.md#fixed-pricing). Para ver las tarifas de precios, consulte los [precios de Logic Apps](../logic-apps/logic-apps-pricing.md).

## <a name="prerequisites"></a>Requisitos previos

* Los mismos [requisitos previos](../logic-apps/connect-virtual-network-vnet-isolated-environment.md#prerequisites) y [requisitos para habilitar el acceso a su ISE](../logic-apps/connect-virtual-network-vnet-isolated-environment.md#enable-access) que cuando se crea un ISE en Azure Portal

* Una herramienta que puede usar para crear su ISE mediante una llamada a la API de REST de Logic Apps con una solicitud HTTPS PUT. Por ejemplo, puede usar [Postman](https://www.getpostman.com/downloads/) o puede compilar una aplicación lógica que realice esta tarea.

## <a name="send-the-request"></a>Envío de la solicitud

Para crear su ISE mediante una llamada a la API de REST de Logic Apps, realice esta solicitud HTTPS PUT.

`PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/integrationServiceEnvironments/{integrationServiceEnvironmentName}?api-version=2019-05-01`

> [!IMPORTANT]
> La versión 2019-05-01 de la API de REST de Logic Apps requiere que realice su propia solicitud HTTP PUT para conectores ISE.

Normalmente, la implementación tarda un máximo de dos horas en completarse. En ocasiones, la implementación puede tardar hasta cuatro horas. Para comprobar el estado de implementación, en [Azure Portal](https://portal.azure.com), en la barra de herramientas de Azure, seleccione el icono de notificaciones, que abre el panel de notificaciones.

> [!NOTE]
> Si se produce un error en la implementación o se elimina el ISE, Azure podría tardar hasta una hora en liberar las subredes. Debido a este retraso, es posible que deba esperar antes de volver a usar esas subredes en otro ISE.
>
> Si elimina su red virtual, Azure generalmente tarda hasta dos horas antes de liberar las subredes, pero esta operación puede llevar más tiempo. 
> Cuando elimine redes virtuales, asegúrese de que no haya recursos conectados. 
> Consulte [Eliminar red virtual](../virtual-network/manage-virtual-network.md#delete-a-virtual-network).

## <a name="request-header"></a>Encabezado de solicitud

En el encabezado de solicitud, incluya estas propiedades:

* `Content-type`: Establezca el valor de propiedad en `application/json`.

* `Authorization`: Establezca este valor de propiedad en el token de portador para el cliente que tiene acceso al grupo de recursos o la suscripción de Azure que desea usar.

<a name="request-body"></a>

## <a name="request-body"></a>Cuerpo de la solicitud

Esta es la sintaxis del cuerpo de la solicitud, que describe las propiedades que se deben usar al crear el ISE. Para crear un ISE que permita el uso de un certificado autofirmado que está instalado en la ubicación `TrustedRoot`, incluya el objeto `certificates` dentro de la sección `properties` de la definición del ISE. Para un ISE existente, puede enviar una solicitud PATCH solo para el objeto `certificates`. Para más información sobre el uso de certificados autofirmados, consulte [Protección del acceso y los datos: acceso para las llamadas salientes a otros servicios y sistemas](../logic-apps/logic-apps-securing-a-logic-app.md#secure-outbound-requests).

```json
{
   "id": "/subscriptions/{Azure-subscription-ID/resourceGroups/{Azure-resource-group}/providers/Microsoft.Logic/integrationServiceEnvironments/{ISE-name}",
   "name": "{ISE-name}",
   "type": "Microsoft.Logic/integrationServiceEnvironments",
   "location": "{Azure-region}",
   "sku": {
      "name": "Premium",
      "capacity": 1
   },
   "properties": {
      "networkConfiguration": {
         "accessEndpoint": {
            // Your ISE can use the "External" or "Internal" endpoint. This example uses "External".
            "type": "External"
         },
         "subnets": [
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-1}",
            },
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-2}",
            },
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-3}",
            },
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-4}",
            }
         ]
      },
      // Include `certificates` object to enable self-signed certificate support
      "certificates": {
         "testCertificate": {
            "publicCertificate": "{base64-encoded-certificate}",
            "kind": "TrustedRoot"
         }
      }
   }
}
```

### <a name="request-body-example"></a>Ejemplo de cuerpo de solicitud

En este cuerpo de solicitud de ejemplo se muestran los valores de ejemplo:

```json
{
   "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Logic/integrationServiceEnvironments/Fabrikam-ISE",
   "name": "Fabrikam-ISE",
   "type": "Microsoft.Logic/integrationServiceEnvironments",
   "location": "WestUS2",
   "sku": {
      "name": "Premium",
      "capacity": 1
   },
   "properties": {
      "networkConfiguration": {
         "accessEndpoint": {
            // Your ISE can use the "External" or "Internal" endpoint. This example uses "External".
            "type": "External"
         },
         "subnets": [
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-1",
            },
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-2",
            },
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-3",
            },
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-4",
            }
         ]
      },
      "certificates": {
         "testCertificate": {
            "publicCertificate": "LS0tLS1CRUdJTiBDRV...",
            "kind": "TrustedRoot"
         }
   }
}
```

## <a name="next-steps"></a>Pasos siguientes

* [Incorporación de recursos a entornos del servicio de integración](../logic-apps/add-artifacts-integration-service-environment-ise.md)
* [Administración de entornos del servicio de integración](../logic-apps/ise-manage-integration-service-environment.md#check-network-health)

