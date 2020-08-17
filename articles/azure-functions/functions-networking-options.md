---
title: Opciones de redes de Azure Functions
description: Introducción a todas las opciones de redes disponibles en Azure Functions.
ms.topic: conceptual
ms.date: 4/11/2019
ms.openlocfilehash: 60258ef4aa3bbbbab69acd4f5106c774caa6f46f
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87385949"
---
# <a name="azure-functions-networking-options"></a>Opciones de redes de Azure Functions

En este artículo se describen las características de redes disponibles en las opciones de hospedaje de Azure Functions. Todas las opciones de redes siguientes ofrecen la posibilidad de acceder a recursos sin usar direcciones enrutables de Internet o de restringir el acceso a Internet a una aplicación de funciones.

Los modelos de hospedaje tienen diferentes niveles de aislamiento de red disponibles. Elegir el correcto le ayuda a cumplir los requisitos de aislamiento de red que tenga.

Puede hospedar aplicaciones de funciones de dos formas:

* Puede elegir entre un conjunto de opciones de plan que se ejecutan en una infraestructura multiinquilino, con distintos niveles de conectividad de red virtual y opciones de escalado:
    * El [plan de consumo](functions-scale.md#consumption-plan) se escala dinámicamente como respuesta a la carga y ofrece opciones de aislamiento de red mínimo.
    * El [plan Premium](functions-scale.md#premium-plan) también se escala dinámicamente y ofrece un aislamiento de red más completo.
    * El [plan de Azure App Service](functions-scale.md#app-service-plan) funciona en una escala fija y ofrece un aislamiento de red similar al del plan Premium.
* Puede ejecutar funciones en un [App Service Environment](../app-service/environment/intro.md). Este método implementa la función en la red virtual y ofrece aislamiento y control de la red completos.

## <a name="matrix-of-networking-features"></a>Matriz de las características de redes

[!INCLUDE [functions-networking-features](../../includes/functions-networking-features.md)]

## <a name="inbound-ip-restrictions"></a>Restricciones de IP de entrada

Puede usar restricciones de IP para definir una lista de direcciones IP ordenadas por prioridad a las que se les permite o deniega el acceso a la aplicación. La lista puede incluir direcciones IPv4 e IPv6. Si hay una o varias entradas, existe un "denegar a todos" implícito al final de la lista. Las restricciones de IP funcionan con todas las opciones de hospedaje de funciones.

> [!NOTE]
> Con las restricciones de red vigentes, solo puede usar el editor del portal desde la red virtual o si ha incluido en la lista de destinatarios seguros la dirección IP del equipo que usa para acceder a Azure Portal. Sin embargo, todavía puede acceder a todas las características de la pestaña **Características de la plataforma** desde cualquier máquina.

Para obtener más información, consulte [Restricciones de acceso estático de Azure App Service](../app-service/app-service-ip-restrictions.md).

## <a name="private-site-access"></a>El acceso privado a sitios

[!INCLUDE [functions-private-site-access](../../includes/functions-private-site-access.md)]

## <a name="virtual-network-integration"></a>Integración de la red virtual

La integración de red virtual permite que la aplicación de funciones acceda a recursos dentro de una red virtual.
Azure Functions admite dos tipos de integración de redes virtuales:

[!INCLUDE [app-service-web-vnet-types](../../includes/app-service-web-vnet-types.md)]

La integración de red virtual de Azure Functions usa una infraestructura compartida con las aplicaciones web de App Service. Para obtener más información sobre los dos tipos de integración de red virtual, vea:

* [Integración de red virtual regional](../app-service/web-sites-integrate-with-vnet.md#regional-vnet-integration)
* [Integración de red virtual con requisito de puerta de enlace](../app-service/web-sites-integrate-with-vnet.md#gateway-required-vnet-integration)

Para aprender a configurar la integración de red virtual, consulte [Tutorial: integración de Functions con una red virtual de Azure](functions-create-vnet.md).

## <a name="regional-virtual-network-integration"></a>Integración de red virtual regional

[!INCLUDE [app-service-web-vnet-types](../../includes/app-service-web-vnet-regional.md)]

## <a name="connect-to-service-endpoint-secured-resources"></a>Conexión a recursos protegidos del punto de conexión de servicio

Con el fin de proporcionar un mayor nivel de seguridad, puede restringir una serie de servicios de Azure a una red virtual mediante puntos de conexión de servicio. Después debe integrar la aplicación de funciones con esa red virtual para acceder al recurso. Esta configuración es compatible con todos los planes que admiten la integración de redes virtuales.

Para más información, consulte [Puntos de conexión de servicio de red virtual](../virtual-network/virtual-network-service-endpoints-overview.md).

## <a name="restrict-your-storage-account-to-a-virtual-network"></a>Restricción de la cuenta de almacenamiento a una red virtual

Al crear una aplicación de funciones, debe crear una cuenta de Azure Storage de uso general compatible con Blob, Queue y Table Storage, o vincular a una. En la actualidad, no es posible usar ninguna restricción de red virtual en esta cuenta. Si configura un punto de conexión de servicio de red virtual en la cuenta de almacenamiento que usa para la aplicación de funciones, esta configuración interrumpirá la aplicación.

Para más información, consulte [Requisitos de la cuenta de almacenamiento](./functions-create-function-app-portal.md#storage-account-requirements).

## <a name="use-key-vault-references"></a>Uso de referencias de Key Vault

Puede usar las referencias de Azure Key Vault para utilizar secretos de esta solución en la aplicación Azure Functions sin necesidad de realizar cambios en el código. Azure Key Vault es un servicio que proporciona administración centralizada de los secretos, con control total sobre las directivas de acceso y el historial de auditoría.

Actualmente las [referencias de Key Vault](../app-service/app-service-key-vault-references.md) no funcionan si el almacén de claves está protegido con puntos de conexión de servicio. Para conectarse a un almacén de claves mediante la integración de la red virtual, deberá llamar a Key Vault en el código de la aplicación.

## <a name="virtual-network-triggers-non-http"></a>Desencadenadores de red virtual (no HTTP)

Actualmente, puede usar funciones de desencadenador no HTTP desde una red virtual de una de estas dos formas:

+ Ejecute la aplicación de funciones en un plan Premium y habilite la compatibilidad con el desencadenador de red virtual.
+ Ejecute una aplicación de funciones en un plan de App Service o App Service Environment.

### <a name="premium-plan-with-virtual-network-triggers"></a>Plan Premium con desencadenadores de red virtual

Cuando cuenta con un plan Premium, puede conectar funciones de desencadenador no HTTP a los servicios que se ejecutan en una red virtual. Para ello, debe habilitar la compatibilidad con el desencadenador de red virtual para la aplicación de funciones. La configuración de la **compatibilidad con el desencadenador de red virtual** se encuentra en [Azure Portal](https://portal.azure.com), en **Configuración** > **Configuración del runtime de la función**.

:::image type="content" source="media/functions-networking-options/virtual-network-trigger-toggle.png" alt-text="VNETToggle":::

También puede habilitar los desencadenadores de red virtual mediante el siguiente comando de la CLI de Azure:

```azurecli-interactive
az resource update -g <resource_group> -n <function_app_name>/config/web --set properties.functionsRuntimeScaleMonitoringEnabled=1 --resource-type Microsoft.Web/sites
```

Los desencadenadores de red virtual se admiten en la versión 2.x y superior del tiempo de ejecución de Functions. Se admiten los siguientes tipos de desencadenador no HTTP.

| Extensión | Versión mínima |
|-----------|---------| 
|[Microsoft.Azure.WebJobs.Extensions.Storage](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage/) | 3.0.10 o superior |
|[Microsoft.Azure.WebJobs.Extensions.EventHubs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.EventHubs)| 4.1.0 o superior|
|[Microsoft.Azure.WebJobs.Extensions.ServiceBus](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.ServiceBus)| 3.2.0 o superior|
|[Microsoft.Azure.WebJobs.Extensions.CosmosDB](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.CosmosDB)| 3.0.5 o superior|
|[Microsoft.Azure.WebJobs.Extensions.DurableTask](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.DurableTask)| 2.0.0 o superior|

> [!IMPORTANT]
> Cuando se habilita la compatibilidad con desencadenadores de red virtual, solo los tipos de desencadenador incluidos en la tabla anterior se escalan dinámicamente con la aplicación. Podrá seguir usando desencadenadores que no figuran en la tabla, pero no se escalarán más allá de su recuento de instancias activadas previamente. Consulte la lista completa de los desencadenadores en [Enlaces admitidos](./functions-triggers-bindings.md#supported-bindings).

### <a name="app-service-plan-and-app-service-environment-with-virtual-network-triggers"></a>Plan de App Service y App Service Environment con desencadenadores de red virtual

Cuando la aplicación de funciones se ejecuta en un plan de App Service o en App Service Environment, puede usar funciones de desencadenador no HTTP. Para que las funciones se desencadenen correctamente, debe estar conectado a una red virtual con acceso al recurso definido en la conexión del desencadenador.

Por ejemplo, imagine que quiere configurar Azure Cosmos DB para aceptar el tráfico solo desde una red virtual. En este caso, debe implementar la aplicación de funciones en un plan de App Service que proporcione integración de red virtual con esa red virtual. La integración permite que un recurso de Azure Cosmos DB desencadene una función.

## <a name="hybrid-connections"></a>conexiones híbridas

[Conexiones híbridas](../azure-relay/relay-hybrid-connections-protocol.md) es una característica de Azure Relay que se puede usar para acceder a recursos de la aplicación en otras redes. Proporciona acceso desde la aplicación a un punto de conexión de la aplicación. No se puede usar para acceder a la aplicación. Las conexiones híbridas están disponibles en funciones que se ejecutan Windows con todos los planes, excepto el plan Consumo.

Dado que se usa en Azure Functions, cada conexión híbrida se correlaciona con una combinación única de host y puerto TCP. Esto significa que el punto de conexión de la conexión híbrida puede estar en cualquier sistema operativo y en cualquier aplicación, siempre que se acceda a un puerto de escucha TCP. La característica Conexiones híbridas no sabe lo que es el protocolo de aplicación ni a qué se accede. Simplemente ofrece acceso a la red.

Para obtener más información, vea la [documentación de App Service para Conexiones híbridas](../app-service/app-service-hybrid-connections.md). Estos mismos pasos de configuración sirven para Azure Functions.

>[!IMPORTANT]
> Las conexiones híbridas solo se admiten en los planes de Windows. Linux no se admite.

## <a name="outbound-ip-restrictions"></a>Restricciones de IP de salida

Las restricciones de IP de salida solo están disponibles para el plan Premium, el plan de App Service o App Service Environment. Puede configurar las restricciones de salida de la red virtual en la que está implementado el App Service Environment.

Cuando se integra una aplicación de funciones de un plan Premium o un plan de App Service con una red virtual, la aplicación todavía puede realizar llamadas salientes a Internet de forma predeterminada. Al agregar la configuración de aplicación `WEBSITE_VNET_ROUTE_ALL=1`, se fuerza el envío de todo el tráfico de salida a la red virtual, donde se pueden usar reglas de grupo de seguridad de red para restringir el tráfico.

## <a name="automation"></a>Automation
Las siguientes API permiten administrar mediante programación las integraciones de redes virtuales regionales:

+ **CLI de Azure**: Use los comandos [`az functionapp vnet-integration`](/cli/azure/functionapp/vnet-integration) para agregar, enumerar o quitar integraciones de redes virtuales regionales.  
+ **Plantillas ARM**: La integración de la red virtual regional se puede habilitar mediante una plantilla de Azure Resource Manager. Para obtener un ejemplo completo, consulte [esta plantilla de inicio rápido de Functions](https://azure.microsoft.com/resources/templates/101-function-premium-vnet-integration/).

## <a name="troubleshooting"></a>Solución de problemas

[!INCLUDE [app-service-web-vnet-troubleshooting](../../includes/app-service-web-vnet-troubleshooting.md)]

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre las redes y Azure Functions:

* [Siga el tutorial sobre cómo empezar con la integración de red virtual](./functions-create-vnet.md)
* [Lea las preguntas más frecuentes de las redes de Functions](./functions-networking-faq.md)
* [Obtenga más información sobre la integración de red virtual con App Service o Functions](../app-service/web-sites-integrate-with-vnet.md)
* [Obtenga más información sobre las redes virtuales en Azure](../virtual-network/virtual-networks-overview.md)
* [Habilite más características de redes y control con App Service Environment](../app-service/environment/intro.md)
* [Conéctese a recursos locales individuales mediante Conexiones híbridas sin modificar el firewall](../app-service/app-service-hybrid-connections.md)
