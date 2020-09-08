---
title: Configuración del firewall de IP para el espacio de nombres de Azure Relay
description: En este artículo se describe el uso de las reglas de firewall para permitir las conexiones desde direcciones IP específicas a espacios de nombres de Azure Relay.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: e47c5071a5fc7207d4eabc162fcb24ab6ad57d28
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88141863"
---
# <a name="configure-ip-firewall-for-an-azure-relay-namespace"></a>Configuración del firewall de IP para un espacio de nombres de Azure Relay
De forma predeterminada, se puede acceder en Internet a los espacios de nombres de Relay, siempre que la solicitud tenga una autenticación y una autorización válidas. Con el firewall de IP, puede restringirlo aún más a solo un conjunto de direcciones o intervalos de direcciones IPv4 en notación [CIDR (Enrutamiento de interdominios sin clases)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing).

Esta característica es útil en escenarios en los que Azure Relay debe ser accesible únicamente desde ciertos sitios conocidos. Las reglas de firewall permiten configurar reglas para aceptar el tráfico procedente de direcciones IPv4 concretas. Por ejemplo, si usa Relay con [Azure Express Route](../expressroute/expressroute-faqs.md#supported-services), puede crear una **regla de firewall** para permitir el tráfico procedente única y exclusivamente de las direcciones IP de la infraestructura local. 


> [!IMPORTANT]
> Esta funcionalidad actualmente está en su versión preliminar. 


## <a name="enable-ip-firewall-rules"></a>Habilitación de las reglas de firewall de IP
Las reglas de firewall de IP se aplican en el nivel del espacio de nombres. Por lo tanto, las reglas se aplican a todas las conexiones de clientes que usan cualquier protocolo admitido. Cualquier intento de conexión desde una dirección IP que no coincida con una regla IP admitida en el espacio de nombres se rechaza como no autorizado. La respuesta no menciona la regla IP. Las reglas de filtro IP se aplican en orden y la primera regla que coincida con la dirección IP determina la acción de aceptar o rechazar.

### <a name="use-azure-portal"></a>Usar Azure Portal
En esta sección se muestra cómo usar Azure Portal para crear reglas de firewall de IP para un espacio de nombres. 

1. Vaya a su **espacio de nombres de Relay** en [Azure Portal](https://portal.azure.com).
2. En el menú de la izquierda, seleccione la opción **Redes**. Si selecciona la opción **Todas las redes** en la sección **Permitir el acceso desde**, el espacio de nombres de Relay acepta conexiones de cualquier dirección IP. Esta configuración equivale a una regla que acepta el intervalo de direcciones IP 0.0.0.0/0. 

    ![Firewall: opción Todas las redes seleccionada](./media/ip-firewall/all-networks-selected.png)
1. Para restringir el acceso a únicamente algunas redes y direcciones IP, seleccione la opción **Redes seleccionadas**. En la sección **Firewall**, haga lo siguiente:
    1. Seleccione la opción **Agregar la dirección IP del cliente** para dar acceso a esa IP de cliente actual al espacio de nombres. 
    2. En **Intervalo de direcciones**, escriba una dirección IPv4 específica o un intervalo de direcciones IPv4 en notación CIDR. 

        ![Firewall: opción Todas las redes seleccionada](./media/ip-firewall/selected-networks-trusted-access-disabled.png)
3. Seleccione **Guardar** en la barra de herramientas para guardar la configuración. Espere unos minutos hasta que la confirmación se muestre en las notificaciones de Azure Portal.


### <a name="use-resource-manager-template"></a>Uso de plantillas de Resource Manager
La siguiente plantilla de Resource Manager permite agregar una regla de filtro de IP a un espacio de nombres de Relay existente.

La plantilla toma un parámetro: **ipMask**, que es una única dirección IPv4 o un bloque de direcciones IP en la notación CIDR. Por ejemplo, en notación CIDR 70.37.104.0/24 representa las 256 direcciones IPv4 de 70.37.104.0 a 70.37.104.255, donde 24 indica el número de bits de prefijo significativos para el intervalo.

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
      "relayNamespaceName": {
        "type": "string",
        "metadata": {
          "description": "Name of the Relay namespace"
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
      "namespaceNetworkRuleSetName": "[concat(parameters('relayNamespaceName'), concat('/', 'default'))]",
    },
    "resources": [
      {
        "apiVersion": "2018-01-01-preview",
        "name": "[parameters('relayNamespaceName')]",
        "type": "Microsoft.Relay/namespaces",
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
        "type": "Microsoft.Relay/namespaces/networkruleset",
        "dependsOn": [
          "[concat('Microsoft.Relay/namespaces/', parameters('relayNamespaceName'))]"
        ],
        "properties": {
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

Para implementar la plantilla, siga las instrucciones para [Azure Resource Manager](../azure-resource-manager/templates/deploy-powershell.md).



## <a name="next-steps"></a>Pasos siguientes
Para información sobre otras características relacionadas con la seguridad de red, consulte [Seguridad de red](network-security.md).


<!-- Links -->

[express-route]:  ../expressroute/expressroute-faqs.md#supported-services
