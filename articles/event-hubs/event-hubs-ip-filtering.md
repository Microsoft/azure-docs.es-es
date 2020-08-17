---
title: Reglas de firewall de Azure Event Hubs | Microsoft Docs
description: Use las reglas de firewall para permitir las conexiones desde direcciones IP específicas a Azure Event Hubs.
ms.topic: article
ms.date: 07/16/2020
ms.openlocfilehash: 2b886aaaf40e5c82d9c7ac3ce5abeda8f54cad3b
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288048"
---
# <a name="configure-ip-firewall-rules-for-an-azure-event-hubs-namespace"></a>Configuración de reglas de firewall de IP para un espacio de nombres de Azure Event Hubs
Los espacios de nombres de Azure Event Hubs son accesibles de forma predeterminada desde Internet, siempre que la solicitud venga con una autenticación y una autorización válidas. Con el firewall de IP, puede restringirlo aún más a solo un conjunto de direcciones o intervalos de direcciones IPv4 en notación [CIDR (Enrutamiento de interdominios sin clases)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing).

Esta característica es útil en escenarios en los que Azure Event Hubs debe ser accesible únicamente desde ciertos sitios conocidos. Las reglas de firewall permiten configurar reglas para aceptar el tráfico procedente de direcciones IPv4 concretas. Por ejemplo, si usa Event Hubs con [Azure ExpressRoute][express-route], puede crear una **regla de firewall** para permitir el tráfico procedente única y exclusivamente de las direcciones IP de la infraestructura local. 

>[!WARNING]
> La habilitación del filtrado de IP puede evitar que otros servicios de Azure interactúen con Event Hubs.
>
> Los servicios de confianza de Microsoft no se admiten cuando se implementan instancias de Virtual Network.
>
> Estos son los escenarios comunes de Azure que no funcionan con instancias de Virtual Network (tenga en cuenta que la lista **NO** está completa).
> - Azure Stream Analytics
> - Enrutamientos de Azure IoT Hub
> - Azure IoT Device Explorer
>
> Los siguientes servicios de Microsoft deben estar en una red virtual
> - Azure Web Apps
> - Azure Functions


## <a name="ip-firewall-rules"></a>Reglas de firewall de IP
Las reglas de firewall de IP se aplican en el nivel del espacio de nombres de Event Hubs. Por lo tanto, las reglas se aplican a todas las conexiones de clientes que usan cualquier protocolo admitido. Cualquier intento de conexión desde una dirección IP que no coincida con una regla IP admitida en el espacio de nombres de Event Hubs se rechaza como no autorizado. La respuesta no menciona la regla IP. Las reglas de filtro IP se aplican en orden y la primera regla que coincida con la dirección IP determina la acción de aceptar o rechazar.

## <a name="use-azure-portal"></a>Usar Azure Portal
En esta sección se muestra cómo usar Azure Portal para crear reglas de firewall de IP para un espacio de nombres de Event Hubs. 

1. Vaya a su **espacio de nombres de Event Hubs** en [Azure Portal](https://portal.azure.com).
2. En el menú de la izquierda, seleccione la opción **Redes**. Si selecciona la opción **Todas las redes**, el centro de eventos aceptará conexiones procedentes de cualquier dirección IP. Esta configuración equivale a una regla que acepta el intervalo de direcciones IP 0.0.0.0/0. 

    ![Firewall: opción Todas las redes seleccionada](./media/event-hubs-firewall/firewall-all-networks-selected.png)
1. Para restringir el acceso a únicamente algunas redes y direcciones IP, seleccione la opción **Redes seleccionadas**. En la sección **Firewall**, haga lo siguiente:
    1. Seleccione la opción **Agregar la dirección IP del cliente** para dar acceso a esa IP de cliente actual al espacio de nombres. 
    2. En **Intervalo de direcciones**, escriba una dirección IPv4 específica o un intervalo de direcciones IPv4 en notación CIDR. 
    3. Especifique si quiere **permitir que los servicios de confianza de Microsoft omitan este firewall**. 

        > [!WARNING]
        > Si elige la opción **Redes seleccionadas** y no especifica ninguna dirección IP o un intervalo de direcciones, el servicio permitirá el tráfico desde todas las redes. 

        ![Firewall: opción Todas las redes seleccionada](./media/event-hubs-firewall/firewall-selected-networks-trusted-access-disabled.png)
3. Seleccione **Guardar** en la barra de herramientas para guardar la configuración. Espere unos minutos hasta que la confirmación se muestre en las notificaciones de Azure Portal.


## <a name="use-resource-manager-template"></a>Uso de plantillas de Resource Manager

> [!IMPORTANT]
> Las reglas de firewall se admiten en los niveles **estándar** y **dedicado** de Event Hubs. No se admiten en el nivel básico.

La siguiente plantilla de Resource Manager permite agregar una regla de filtro de IP a un espacio de nombres de Event Hubs.

Parámetros de plantilla:

- **ipMask** es una única dirección IPv4 o un bloque de direcciones IP en la notación CIDR. Por ejemplo, en notación CIDR 70.37.104.0/24 representa las 256 direcciones IPv4 de 70.37.104.0 a 70.37.104.255, donde 24 indica el número de bits de prefijo significativos para el intervalo.

> [!NOTE]
> Si bien no hay reglas de denegación posibles, la plantilla de Azure Resource Manager tiene la acción predeterminada establecida en **"Permitir"** , que no restringe las conexiones.
> Cuando se realizan las reglas de Virtual Network o de firewall, es necesario cambiar el valor ***"defaultAction"***
> 
> desde
> ```json
> "defaultAction": "Allow"
> ```
> to
> ```json
> "defaultAction": "Deny"
> ```
>

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "eventhubNamespaceName": {
        "type": "string",
        "metadata": {
          "description": "Name of the Event Hubs namespace"
        }
      },
      "location": {
        "type": "string",
        "metadata": {
          "description": "Location for Namespace"
        }
      }
    },
    "variables": {
      "namespaceNetworkRuleSetName": "[concat(parameters('eventhubNamespaceName'), concat('/', 'default'))]",
    },
    "resources": [
      {
        "apiVersion": "2018-01-01-preview",
        "name": "[parameters('eventhubNamespaceName')]",
        "type": "Microsoft.EventHub/namespaces",
        "location": "[parameters('location')]",
        "sku": {
          "name": "Standard",
          "tier": "Standard"
        },
        "properties": { }
      },
      {
        "apiVersion": "2018-01-01-preview",
        "name": "[variables('namespaceNetworkRuleSetName')]",
        "type": "Microsoft.EventHub/namespaces/networkruleset",
        "dependsOn": [
          "[concat('Microsoft.EventHub/namespaces/', parameters('eventhubNamespaceName'))]"
        ],
        "properties": {
          "virtualNetworkRules": [<YOUR EXISTING VIRTUAL NETWORK RULES>],
          "ipRules": 
          [
            {
                "ipMask":"10.1.1.1",
                "action":"Allow"
            },
            {
                "ipMask":"11.0.0.0/24",
                "action":"Allow"
            }
          ],
          "trustedServiceAccessEnabled": false,
          "defaultAction": "Deny"
        }
      }
    ],
    "outputs": { }
  }
```

Para implementar la plantilla, siga las instrucciones para [Azure Resource Manager][lnk-deploy].

## <a name="next-steps"></a>Pasos siguientes

Para restringir el acceso a Event Hubs a las redes virtuales de Azure, visite el siguiente vínculo:

- [Puntos de conexión de servicio de red virtual para Event Hubs][lnk-vnet]

<!-- Links -->

[express-route]:  /azure/expressroute/expressroute-faqs#supported-services
[lnk-deploy]: ../azure-resource-manager/templates/deploy-powershell.md
[lnk-vnet]: event-hubs-service-endpoints.md
