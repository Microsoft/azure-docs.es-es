---
title: Restricciones de acceso de Azure App Service
description: Aprenda a proteger la aplicación en Azure App Service mediante la especificación de restricciones de acceso.
author: ccompy
ms.assetid: 3be1f4bd-8a81-4565-8a56-528c037b24bd
ms.topic: article
ms.date: 06/06/2019
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: ce5882f2621dc5b8c48bcf5be6d4ea3a2f723bfe
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962968"
---
# <a name="azure-app-service-access-restrictions"></a>Restricciones de acceso de Azure App Service

Las restricciones de acceso permiten definir una lista ordenada por prioridad de elementos permitidos o denegados con la que se controla el acceso de red a la aplicación. La lista puede incluir direcciones IP o subredes de Azure Virtual Network. Cuando hay una o varias entradas, hay una denegación implícita de todo lo que existe al final de la lista.

La funcionalidad de restricciones de acceso se aplica a todas las cargas de trabajo hospedadas en App Service, lo que incluye aplicaciones web, aplicaciones de API, aplicaciones de Linux, aplicaciones de contenedor de Linux e instancias de Functions.

Cuando se realiza una solicitud a la aplicación, la dirección de origen se evalúa con respecto a una serie de reglas de dirección IP de la lista de restricciones de acceso. Si la dirección de origen está en una subred configurada con puntos de conexión de servicio a Microsoft.Web, la subred de origen se compara con las reglas de red virtual de la lista de restricciones de acceso. Si la dirección no tiene permitido el acceso según las reglas de la lista, el servicio responde con un código de estado [HTTP 403](https://en.wikipedia.org/wiki/HTTP_403).

La funcionalidad de restricciones de acceso se implementa en los roles front-end de App Service, que son de nivel superior de los hosts de trabajo donde el código se ejecuta. Por lo tanto, las restricciones de acceso son listas de control de acceso de red de facto.

La capacidad de restringir el acceso a la aplicación web desde una instancia de Azure Virtual Network (red virtual) se denomina [puntos de conexión de servicio][serviceendpoints]. Los puntos de conexión de servicio permiten restringir el acceso de un servicio multiinquilino de las subredes seleccionadas. Se debe habilitar tanto en el lado de la red como en el servicio con el que se está habilitando. No funciona para restringir el tráfico a aplicaciones hospedadas en un App Service Environment. Si está en un App Service Environment, puede controlar el acceso a la aplicación con reglas de direcciones IP.

![Flujo de restricciones de acceso](media/app-service-ip-restrictions/access-restrictions-flow.png)

## <a name="adding-and-editing-access-restriction-rules-in-the-portal"></a>Incorporación y edición de reglas de restricciones de acceso en el portal ##

Para agregar una regla de restricción de acceso a la aplicación, use el menú para abrir **Red**>**Restricciones de acceso** y haga clic en **Configurar restricciones de acceso**.

![Opciones de red de App Service](media/app-service-ip-restrictions/access-restrictions.png)  

En la interfaz de las restricciones de acceso puede revisar la lista de reglas de restricción de acceso definidas en relación con la aplicación.

![Lista de restricciones de acceso](media/app-service-ip-restrictions/access-restrictions-browse.png)

La lista mostrará todas las restricciones actuales que hay en la aplicación. Si tiene una restricción de red virtual en la aplicación, la tabla mostrará si los puntos de conexión del servicio están habilitados para Microsoft.Web. Si no hay ninguna restricción definida en la aplicación, esta será accesible desde cualquier lugar.  

## <a name="adding-ip-address-rules"></a>Adición de reglas de direcciones IP

Puede hacer clic en **[+] Agregar regla** para agregar una nueva regla de restricciones de acceso. Una vez que agregue una regla, entrará en vigor de inmediato. Las reglas se aplican en orden de prioridad, empezando por el número más bajo y en ascenso. Hay una denegación implícita de todo lo que esté en vigor una vez que agregue incluso una sola regla.

Al crear una regla, debe seleccionar Permitir/Denegar y, también, el tipo de regla. También debe proporcionar el valor de prioridad y a qué se está restringiendo el acceso.  Opcionalmente, puede agregar un nombre y una descripción a la regla.  

![Adición de una regla de restricción de acceso de IP](media/app-service-ip-restrictions/access-restrictions-ip-add.png)

Para definir una regla basada en direcciones IP, seleccione el tipo IPv4 o IPv6. La notación de dirección IP debe ser CIDR tanto para direcciones IPv4 como IPv6. Para especificar una dirección exacta, puede usar un formato como 1.2.3.4/32, donde los cuatro primeros octetos representan la dirección IP y /32 es la máscara. La notación CIDR IPv4 para todas las direcciones es 0.0.0.0/0. Para más información acerca de la notación CIDR, puede leer [Classless Inter-Domain Routing](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) (Enrutamiento interdominios sin clases). 

## <a name="service-endpoints"></a>Puntos de conexión del servicio

Los puntos de conexión de servicio permiten restringir el acceso a las subredes de Azure Virtual Network. Para restringir el acceso a una subred específica, cree una regla de restricción de tipo Virtual Network. Puede elegir la suscripción, la red virtual y la subred en las que quiera permitir o denegar el acceso. Si los puntos de conexión de servicio no están habilitados aún con Microsoft.Web en relación con la subred seleccionada, se habilitarán automáticamente a menos que active la casilla que indica que no desea hacerlo. La situación en la que convendría habilitarlos en la aplicación, pero no en la subred, tiene mucho que ver con el hecho de si se tienen los permisos para habilitar puntos de conexión de servicio en la subred o no. Si necesita acudir a alguien más para habilitar los puntos de conexión de servicio en la subred, puede activar la casilla y configurar la aplicación para puntos de conexión de servicio en previsión de que vayan a habilitarse más adelante. 

![Adición de una regla de restricción de acceso a red virtual](media/app-service-ip-restrictions/access-restrictions-vnet-add.png)

Los puntos de conexión de servicio no se pueden usar para restringir el acceso a las aplicaciones que se ejecutan en un App Service Environment. Si está en un App Service Environment, puede controlar el acceso a la aplicación con reglas de acceso de IP. 

Si se usan puntos de conexión de servicio, la aplicación se puede configurar con Application Gateway u otros dispositivos de WAF. También se pueden configurar aplicaciones de varios niveles con back-ends seguros. Para más información sobre algunas de las posibilidades, lea [Características de redes de App Service](networking-features.md) e [Integración de Application Gateway con puntos de conexión de servicio](networking/app-gateway-with-service-endpoints.md).

> [!NOTE]
> Actualmente no se admiten puntos de conexión de servicio para las aplicaciones web que usan IP virtual (VIP) con SSL de IP. 
>

## <a name="managing-access-restriction-rules"></a>Administración de reglas de restricción de acceso

Puede hacer clic en cualquier fila para editar una regla de restricción de acceso existente. Las modificaciones son efectivas inmediatamente, incluidos los cambios en el orden de prioridad.

![Edición de una regla de restricción de acceso](media/app-service-ip-restrictions/access-restrictions-ip-edit.png)

Al editar una regla, no se puede cambiar el tipo entre una regla de dirección IP y una regla de Virtual Network. 

![Edición de una regla de restricción de acceso](media/app-service-ip-restrictions/access-restrictions-vnet-edit.png)

Para eliminar una regla, haga clic en los puntos suspensivos **...** en la regla y, a continuación, haga clic en **Quitar**.

![Eliminación de una regla de restricción de acceso](media/app-service-ip-restrictions/access-restrictions-delete.png)

## <a name="blocking-a-single-ip-address"></a>Bloqueo de una única dirección IP ##

Al agregar la primera regla de restricción de IP, el servicio agregará una regla **Denegar todo** explícita con una prioridad de 2147483647. En la práctica, la regla explícita **Denegar todo** será la última regla que se ejecute y bloqueará el acceso a cualquier dirección IP que no esté expresamente permitida mediante una regla **Permitir**.

En situaciones en las que los usuarios quieran bloquear expresamente una única dirección IP o un bloque de direcciones IP, pero permitir el acceso a todo lo demás, será necesario agregar la regla **Permitir todo** explícita.

![Bloqueo de dirección IP única](media/app-service-ip-restrictions/block-single-address.png)

## <a name="scm-site"></a>Sitio de SCM 

Aparte de controlar el acceso a la aplicación, también puede restringir el acceso al sitio de SCM utilizado por la aplicación. El sitio de SCM es el punto de conexión de Web Deploy y, también, la consola de Kudu. Puede asignar restricciones de acceso aisladas al sitio de SCM desde la aplicación o usar el mismo conjunto para la aplicación y el sitio de SCM. Si se activa la casilla para tener las mismas restricciones que la aplicación, todo estará en blanco. Si se desactiva, se aplicarán las configuraciones que hubiera anteriormente en el sitio de SCM. 

![Lista de restricciones de acceso](media/app-service-ip-restrictions/access-restrictions-scm-browse.png)

## <a name="programmatic-manipulation-of-access-restriction-rules"></a>Manipulación mediante programación de reglas de restricción de acceso ##

La [CLI de Azure](/cli/azure/webapp/config/access-restriction?view=azure-cli-latest) y [Azure PowerShell](/powershell/module/Az.Websites/Add-AzWebAppAccessRestrictionRule?view=azps-3.1.0) admiten la edición de restricciones de acceso. Ejemplo de incorporación de una restricción de acceso mediante la CLI de Azure:

```azurecli-interactive
az webapp config access-restriction add --resource-group ResourceGroup --name AppName \
    --rule-name 'IP example rule' --action Allow --ip-address 122.133.144.0/24 --priority 100
```
Ejemplo de incorporación de una restricción de acceso mediante Azure PowerShell:

```azurepowershell-interactive
Add-AzWebAppAccessRestrictionRule -ResourceGroupName "ResourceGroup" -WebAppName "AppName"
    -Name "Ip example rule" -Priority 100 -Action Allow -IpAddress 122.133.144.0/24
```

Los valores también se pueden establecer manualmente con una operación PUT de la [API REST de Azure](/rest/api/azure/) en la configuración de la aplicaciones de Resource Manager o mediante una plantilla de Azure Resource Manager. Por ejemplo, puede usar resources.azure.com y editar el bloque ipSecurityRestrictions para agregar el código JSON requerido.

La ubicación de esta información en Resource Manager es:

management.azure.com/subscriptions/**Id. de suscripción**/resourceGroups/**resource groups**/providers/Microsoft.Web/sites/**nombre de aplicación web**/config/web?api-version=2018-02-01

La sintaxis JSON para el ejemplo anterior es:
```json
{
  "properties": {
    "ipSecurityRestrictions": [
      {
        "ipAddress": "122.133.144.0/24",
        "action": "Allow",
        "priority": 100,
        "name": "IP example rule"
      }
    ]
  }
}
```

## <a name="azure-functions-access-restrictions"></a>Restricciones de acceso de Azure Functions

También hay restricciones de acceso disponibles para las aplicaciones de funciones con la misma funcionalidad que los planes de App Service. Si se habilitan restricciones de acceso, se deshabilitará el editor de código del portal en las direcciones IP no permitidas.

## <a name="next-steps"></a>Pasos siguientes
[Restricciones de acceso para Azure Functions](../azure-functions/functions-networking-options.md#inbound-ip-restrictions)

[Integración de Application Gateway con puntos de conexión de servicio](networking/app-gateway-with-service-endpoints.md)

<!--Links-->
[serviceendpoints]: ../virtual-network/virtual-network-service-endpoints-overview.md