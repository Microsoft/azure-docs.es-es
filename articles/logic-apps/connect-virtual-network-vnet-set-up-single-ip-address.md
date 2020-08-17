---
title: Configuración de una dirección IP de salida pública para ISE
description: Aprenda a configurar una única dirección IP de salida pública para entornos del servicio de integración (ISE) en Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: jonfan, logicappspm
ms.topic: conceptual
ms.date: 05/06/2020
ms.openlocfilehash: 9ee804f7bed01ca0c7f365a04e6108afd9598157
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87066388"
---
# <a name="set-up-a-single-ip-address-for-one-or-more-integration-service-environments-in-azure-logic-apps"></a>Configuración de una única dirección IP para uno o varios entornos de servicio de integración en Azure Logic Apps

Cuando trabaja con Azure Logic Apps, puede configurar un [*entorno de servicio de integración* (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) para hospedar aplicaciones lógicas que necesitan acceso a recursos en una [red virtual de Azure](../virtual-network/virtual-networks-overview.md). Si tiene varias instancias de ISE que necesitan acceso a otros puntos de conexión con restricciones de IP, implemente [Azure Firewall](../firewall/overview.md) o una [aplicación virtual de red](../virtual-network/virtual-networks-overview.md#filter-network-traffic) en la red virtual y enrute el tráfico saliente a través de ese firewall o aplicación virtual de red. Luego, puede hacer que todas las instancias de ISE de la red virtual usen una única dirección IP pública, estática y predecible para comunicarse con los sistemas de destino que quiera. De ese modo, no es necesario configurar aperturas adicionales del firewall en los sistemas de destino para cada ISE.

En este tema se muestra cómo enrutar el tráfico saliente a través de Azure Firewall, pero puede aplicar conceptos similares a una aplicación virtual de red, como un firewall de terceros, desde Azure Marketplace. Aunque este tema se centra en la configuración de varias instancias de ISE, también puede usar este enfoque para un solo ISE si el escenario requiere limitar el número de direcciones IP que necesitan acceso. Considere si los costos adicionales del firewall o el dispositivo de red virtual tienen sentido para su escenario. Más información acerca de los [precios de Azure Firewall](https://azure.microsoft.com/pricing/details/azure-firewall/).

## <a name="prerequisites"></a>Prerrequisitos

* Una instancia de Azure Firewall que se ejecuta en la misma red virtual que el ISE. Si no tiene un firewall, primero [agregue una subred](../virtual-network/virtual-network-manage-subnet.md#add-a-subnet) denominada `AzureFirewallSubnet` a la red virtual. Luego puede [crear e implementar un firewall](../firewall/tutorial-firewall-deploy-portal.md#deploy-the-firewall) en la red virtual.

* Una [tabla de rutas](../virtual-network/manage-route-table.md) de Azure. Si no tiene una tabla de rutas, primero [debe crearla](../virtual-network/manage-route-table.md#create-a-route-table). Para más información sobre el enrutamiento, consulte [Enrutamiento del tráfico de redes virtuales](../virtual-network/virtual-networks-udr-overview.md).

## <a name="set-up-route-table"></a>Configuración de una tabla de rutas

1. En [Azure Portal](https://portal.azure.com), seleccione la tabla de rutas, por ejemplo:

   ![Selección de tabla de rutas con una regla para dirigir el tráfico saliente](./media/connect-virtual-network-vnet-set-up-single-ip-address/select-route-table-for-virtual-network.png)

1. Para [agregar una ruta nueva](../virtual-network/manage-route-table.md#create-a-route), en el menú de la tabla de rutas, seleccione **Rutas** > **Agregar**.

   ![Agregar una ruta para dirigir el tráfico saliente](./media/connect-virtual-network-vnet-set-up-single-ip-address/add-route-to-route-table.png)

1. En el panel **Agregar ruta**, [configure la ruta nueva](../virtual-network/manage-route-table.md#create-a-route) con una regla que especifica que todo el tráfico saliente al sistema de destino sigue este comportamiento:

   * Usa la [**aplicación virtual**](../virtual-network/virtual-networks-udr-overview.md#user-defined) como el tipo del próximo salto.

   * Va a la dirección IP privada para la instancia de firewall como la dirección del próximo salto.

     Para encontrar esta dirección IP, en el menú del firewall, seleccione **Información general** y busque la **Dirección IP privada**, por ejemplo:

     ![Búsqueda de la dirección IP privada del firewall](./media/connect-virtual-network-vnet-set-up-single-ip-address/find-firewall-private-ip-address.png)

   Este es un ejemplo en el que se muestra cómo podría ser tal regla:

   ![Configuración de una regla para dirigir el tráfico saliente](./media/connect-virtual-network-vnet-set-up-single-ip-address/add-rule-to-route-table.png)

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | **Nombre de ruta** | <*unique-route-name*> | Un nombre único para la ruta en la tabla de rutas. |
   | **Prefijo de dirección** | <*destination-address*> | El prefijo de la dirección del sistema de destino adonde quiere que vaya el tráfico saliente. Asegúrese de usar la [notación de enrutamiento de interdominios sin clases (CIDR)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) para esta dirección. En este ejemplo, el prefijo de dirección es para un servidor SFTP, que se describe en la sección [Configuración de la regla de red](#set-up-network-rule). |
   | **Tipo del próximo salto** | **Aplicación virtual** | El [tipo de salto](../virtual-network/virtual-networks-udr-overview.md#next-hop-types-across-azure-tools) que usa el tráfico saliente. |
   | **Dirección del próximo salto** | <*firewall-private-IP-address*> | La dirección IP privada del firewall. |
   |||

<a name="set-up-network-rule"></a>

## <a name="set-up-network-rule"></a>Configuración de la regla de red

1. En Azure Portal, busque y seleccione el firewall. En el menú del firewall, en **Configuración**, seleccione **Reglas**. En el panel de las reglas, seleccione **Colección de reglas de red** > **Agregar una colección de reglas de red**.

   ![Agregar una colección de reglas de red al firewall](./media/connect-virtual-network-vnet-set-up-single-ip-address/add-network-rule-collection.png)

1. En la colección, agregue una regla que permita el tráfico al sistema de destino.

   Por ejemplo, supongamos que tiene una aplicación lógica que se ejecuta en un ISE y necesita comunicarse con un servidor SFTP. Cree una colección de reglas de red denominada `LogicApp_ISE_SFTP_Outbound` que contenga una regla de red denominada `ISE_SFTP_Outbound`. Esta regla permite el tráfico desde la dirección IP de cualquier subred en la que el ISE se ejecuta en la red virtual al servidor SFTP de destino mediante el uso de la dirección IP privada del firewall.

   ![Configuración de una regla de red para el firewall](./media/connect-virtual-network-vnet-set-up-single-ip-address/set-up-network-rule-for-firewall.png)

   **Propiedades de una colección de reglas de red**

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | **Nombre** | <*network-rule-collection-name*> | El nombre de la colección de reglas de red. |
   | **Prioridad** | <*priority-level*> | El orden de prioridad que se va a usar para ejecutar la colección de reglas. Para más información, consulte [¿Cuáles son algunos de los conceptos de Azure Firewall](../firewall/firewall-faq.md#what-are-some-azure-firewall-concepts)? |
   | **Acción** | **Permitir** | El tipo de acción que se va a realizar para esta regla. |
   |||

   **Propiedades de una regla de red**

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | **Nombre** | <*network-rule-name*> | El nombre de la regla de red. |
   | **Protocolo** | <*connection-protocols*> | Los protocolos de conexión que se van a usar. Por ejemplo, si usa reglas de NSG, seleccione tanto **TCP** como **UDP**, no solo **TCP**. |
   | **Direcciones de origen** | <*ISE-subnet-addresses*> | Las direcciones IP de subred donde se ejecuta el ISE y donde se origina el tráfico desde la aplicación lógica. |
   | **Direcciones de destino** | <*destination-IP-address*> | Las direcciones IP del sistema de destino adonde quiere que vaya el tráfico saliente. En este ejemplo, esta dirección IP es para el servidor SFTP. |
   | **Puertos de destino** | <*destination-ports*> | Todo puerto que el sistema de destino usa para la comunicación entrante. |
   |||

   Para más observación sobre las reglas de red, consulte estos artículos:

   * [Configuración de una regla de red](../firewall/tutorial-firewall-deploy-portal.md#configure-a-network-rule).
   * [Lógica de procesamiento de reglas de Azure Firewall](../firewall/rule-processing.md#network-rules-and-applications-rules).
   * [Preguntas frecuentes sobre Azure Firewall](../firewall/firewall-faq.md).
   * [Azure PowerShell: New-AzFirewallNetworkRule](/powershell/module/az.network/new-azfirewallnetworkrule)
   * [CLI de Azure: az network firewall network-rule](/cli/azure/ext/azure-firewall/network/firewall/network-rule?view=azure-cli-latest#ext-azure-firewall-az-network-firewall-network-rule-create)

## <a name="next-steps"></a>Pasos siguientes

* [Conexión a redes virtuales de Azure desde Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment.md).
