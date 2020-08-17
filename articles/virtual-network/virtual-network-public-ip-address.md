---
title: Creación, modificación o eliminación de una dirección IP pública de Azure | Microsoft Docs
description: Cree, modifique o elimine una dirección IP pública. Además, aprenda de qué forma una dirección IP pública es un recurso con sus propios valores de configuración.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
editor: ''
tags: azure-resource-manager
ms.assetid: bb71abaf-b2d9-4147-b607-38067a10caf6
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/06/2019
ms.author: kumud
ms.openlocfilehash: 4c0766dc063932c5fdd41a4e21ac11befd84a0e5
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87265133"
---
# <a name="create-change-or-delete-a-public-ip-address"></a>Creación, modificación o eliminación de una dirección IP pública

Obtenga información sobre una dirección IP pública y cómo crearla, modificarla y eliminarla. Una dirección IP pública es un recurso con sus propios valores de configuración. Asignar una dirección IP pública a un recurso de Azure que admita direcciones IP públicas permite:
- La comunicación entrante desde Internet a los recursos, como Azure Virtual Machines, Azure Application Gateway, Azure Load Balancer, Azure VPN Gateway y otros. Todavía puede comunicarse con recursos como máquinas virtuales desde Internet, si una máquina virtual no tiene asignada una dirección IP pública, y siempre que la máquina virtual forme parte de un grupo de back-end de un equilibrador de carga, y el equilibrador de carga tenga asignada una dirección IP pública. Para determinar si puede asignarse una dirección IP pública a un recurso para un servicio específico de Azure, o si es posible comunicarse con él a través de la dirección IP pública de otro recurso de Azure, consulte la documentación del servicio.
- La conectividad saliente a Internet usa una dirección IP predecible. Por ejemplo, una máquina virtual puede establecer una comunicación saliente a Internet sin tener asignada una dirección IP pública, pero su dirección es una dirección de red que Azure traduce a una dirección pública impredecible, de manera predeterminada. Asignar una dirección IP pública a un recurso le permite saber cuál es la dirección IP que se usa para la conexión saliente. Si bien la dirección es predecible, puede cambiar en función del método de asignación que se elija. Para más información, consulte [Creación de una dirección IP pública](#create-a-public-ip-address). Para obtener más información sobre las conexiones salientes de recursos de Azure, consulte [Conexiones salientes](../load-balancer/load-balancer-outbound-connections.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Complete las tareas siguientes antes de seguir los pasos de las secciones de este artículo:

- Si todavía no tiene una cuenta de Azure, regístrese para obtener una [cuenta de evaluación gratuita](https://azure.microsoft.com/free).
- Si usa el portal, abra https://portal.azure.com e inicie sesión con la cuenta de Azure.
- Si usa comandos de PowerShell para completar las tareas de este artículo, ejecute los comandos que se encuentran en [Azure Cloud Shell](https://shell.azure.com/powershell) o ejecute PowerShell en el equipo. Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. Para realizar este tutorial, se necesita la versión 1.0.0 del módulo de Azure PowerShell u otra posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.
- Si usa la interfaz de la línea de comandos (CLI) de Azure para completar las tareas de este artículo, ejecute los comandos que se encuentran en [Azure Cloud Shell](https://shell.azure.com/bash) o ejecute la CLI en el equipo. Para realizar este tutorial, es necesaria la versión 2.0.31 de la CLI de Azure o una versión posterior. Ejecute `az --version` para buscar la versión instalada. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Si ejecuta de forma local la CLI de Azure, también debe ejecutar `az login` para crear una conexión con Azure.

La cuenta en la que inicia sesión o con la que se conecta a Azure debe tener asignado el rol de [colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o un [rol personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones apropiadas en [Permisos](#permissions).

Las direcciones IP públicas tienen un precio simbólico. Para ver los precios, consulte la página [Precios de las direcciones IP](https://azure.microsoft.com/pricing/details/ip-addresses).

## <a name="create-a-public-ip-address"></a>Crear una dirección IP pública

1. En el menú de Azure Portal o en la **página principal**, seleccione **Crear un recurso**.
2. Escriba *dirección ip pública* en el cuadro de *búsqueda de Marketplace*. Seleccione **Dirección IP pública** cuando aparezca en los resultados de búsqueda.
3. En **Dirección IP pública**, seleccione **Crear**.
4. Escriba o seleccione valores para las siguientes opciones, en **Crear dirección IP pública** y seleccione **Crear**:

   |Configuración|¿Necesario?|Detalles|
   |---|---|---|
   |Versión de la dirección IP|Sí| Seleccione IPv4 o IPv6 o ambas. Seleccionar ambas se traducirá en la creación de dos direcciones IP públicas: una dirección IPv4 y una dirección IPv6. Obtenga más información sobre [IPv6 en las redes virtuales de Azure](../virtual-network/ipv6-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json).|
   |SKU|Sí|Todas las direcciones IP públicas creadas antes de la introducción de SKU son direcciones IP públicas de SKU **básica**. No se puede cambiar la SKU después de crear la dirección IP pública. Una máquina virtual independiente, las máquinas virtuales dentro de un conjunto de disponibilidad o los conjuntos de escalado de máquinas virtuales pueden usar SKU básicas o estándar. No se permite la mezcla de SKU entre máquinas virtuales dentro de conjuntos de disponibilidad, conjuntos de escalado o VM independientes. SKU **básico**: si va a crear una dirección IP pública en una región que admite zonas de disponibilidad, el ajuste **Zona disponibilidad** se establece en *Ninguno* de forma predeterminada. Las IP públicas básicas no admiten zonas de disponibilidad. SKU **estándar**: una dirección IP pública de SKU estándar se puede asociar a una máquina virtual o una front-end de equilibrador de carga. Si va a crear una dirección IP pública en una región que admite zonas de disponibilidad, el ajuste **Zona disponibilidad** se establece en *Con redundancia de zona* de forma predeterminada. Para obtener información sobre las zonas de disponibilidad, consulte el ajuste **Zona de disponibilidad**. La SKU estándar es necesaria si se asocia la dirección a un equilibrador de carga estándar. Para obtener más información sobre equilibradores de carga estándar, consulte [Azure load balancer standard SKU](../load-balancer/load-balancer-standard-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (SLU estándar para Azure Load Balancer). Cuando asigna una dirección IP pública de SKU estándar a una interfaz de red de una máquina virtual, debe permitir explícitamente el tráfico previsto con un [grupo de seguridad de red](security-overview.md#network-security-groups). Para evitar que se produzca un error en la comunicación con el recurso, debe crear un grupo de seguridad de red, asociarlo y permitir explícitamente el tráfico deseado.|
   |Nombre|Sí|El nombre debe ser único dentro del grupo de recursos que seleccione.|
   |Asignación de dirección IP|Sí|**Dinámica:** las direcciones dinámicas se asignan solo después de que una dirección IP pública se asocia a un recurso de Azure y el recurso se inicia por primera vez. Las direcciones dinámicas pueden cambiar si se asignan a un recurso, como una máquina virtual, y la máquina virtual se detiene (desasigna) y luego se reinicia. La dirección sigue siendo la misma si la máquina virtual se reinicia o se detiene (pero no se desasigna). Las direcciones dinámicas se liberan cuando un recurso de dirección IP pública se desasocia de un recurso o se asocia a él. **Estática:** las direcciones estáticas se asignan cuando se crea la dirección IP pública. Las direcciones estáticas no se liberan hasta que se elimina un recurso de dirección IP pública. Si la dirección no está asociada a un recurso, puede cambiar el método de asignación una vez creada la dirección. Si la dirección está asociada a un recurso, es posible que no pueda cambiar el método de asignación. Si selecciona *IPv6* como la **versión de dirección IP**, el método de asignación debe ser *Dinámica* para el SKU básico.  Las direcciones de SKU estándar son *estáticas* tanto para IPv4 como para IPv6. |
   |Tiempo de espera de inactividad (minutos)|No|Cantidad de minutos para mantener una conexión TCP o HTTP abierta sin que dependa del envío de mensajes de mantenimiento de los clientes. Si selecciona IPv6 como la **versión de dirección IP**, no se puede cambiar este valor. |
   |Etiqueta de nombre DNS|No|Debe ser único dentro de la ubicación de Azure en la que cree el nombre (entre todas las suscripciones y todos los clientes). Azure registra automáticamente el nombre y la dirección IP en el DNS para que pueda conectarse a un recurso con el nombre. Azure anexa una subred predeterminada, como *location.cloudapp.azure.com* (donde location es la ubicación seleccionada), al nombre que proporcione para crear el nombre DNS completo. Si elige crear ambas versiones de direcciones, el mismo nombre DNS se asigna tanto a la dirección IPv4 como a la IPv6. El DNS predeterminado de Azure contiene registros de nombres AAAA tanto de IPv4 como de IPv6 y responde con ambos registros cuando se busca el nombre DNS. El cliente elige con qué dirección (IPv4 o IPv6) comunicarse. En lugar de usar la etiqueta de nombre DNS con el sufijo predeterminado, o además de ello, puede usar el servicio Azure DNS para configurar un nombre DNS con un sufijo personalizado que se resuelva en la dirección IP pública. Para obtener más información, vea los detalles relativos al [uso de Azure DNS con una dirección IP pública de Azure](../dns/dns-custom-domain.md?toc=%2fazure%2fvirtual-network%2ftoc.json#public-ip-address).|
   |Nombre (solo visible si selecciona la versión de IP de **ambas**)|Sí, si selecciona la versión de IP de **ambas**|El nombre debe ser distinto del nombre que escribe para el **nombre** de esta lista. Si elige crear tanto una dirección IPv4 como IPv6, el portal crea dos recursos de direcciones IP públicas independientes, uno con cada versión de dirección IP asignada.|
   |Asignación de direcciones IP (solo visible si selecciona la versión de IP de **ambas**)|Sí, si selecciona la versión de IP de **ambas**|Las mismas restricciones que la asignación de direcciones IP anterior|
   |Subscription|Sí|Debe existir en la misma [suscripción](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#subscription) que el recurso al cual asociará las direcciones IP públicas.|
   |Resource group|Sí|Puede existir en el mismo [grupo de recursos](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#resource-group) o en otro diferente como recurso al que asociará las direcciones IP públicas.|
   |Location|Sí|Debe existir en la misma [ubicación](https://azure.microsoft.com/regions), a la que también se hace referencia como región, que el recurso al que asociará las direcciones públicas.|
   |Zona de disponibilidad| No | Este ajuste solo aparece si selecciona una ubicación admitida. Para una lista de ubicaciones admitidas, consulte [Introducción a las zonas de disponibilidad](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Si seleccionó la SKU **básica**, la opción *Ninguna* se selecciona automáticamente. Si prefiere garantizar una zona específica, puede seleccionarla. Cualquiera de las opciones no es con redundancia de zona. Si seleccionó la SKU **estándar**: la opción con redundancia de zona se selecciona automáticamente y hace que la ruta de acceso de datos sea resistente a errores de zona. Si prefiere garantizar una zona específica, que no sea resistente a errores de zona, puede seleccionarla.

**Comandos**

Si bien el portal proporciona la opción de crear dos recursos de direcciones IP públicas (una IPv4 y una IPv6), los comandos de PowerShell y la CLI siguientes crean un recurso con una dirección para una versión de dirección IP o la otra. Si desea dos recursos de direcciones IP públicas, uno para cada versión de dirección IP, debe ejecutar dos veces el comando y especificar distintos nombres y versiones de dirección IP para los recursos de direcciones IP públicas.

|Herramienta|Get-Help|
|---|---|
|CLI|[az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create)|
|PowerShell|[New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress)|

## <a name="view-change-settings-for-or-delete-a-public-ip-address"></a>Visualización, cambio de la configuración o eliminación de una dirección IP pública

1. En el cuadro que contiene el texto *Buscar recursos*, en la parte superior de Azure Portal, escriba *Dirección IP pública*. Seleccione **Direcciones IP públicas** cuando aparezca en los resultados de búsqueda.
2. Seleccione de la lista el nombre de la dirección IP pública cuya configuración desee ver, cambiar o eliminar.
3. Complete una de las siguientes opciones en función de si desea ver, eliminar o cambiar la dirección IP pública.
   - **Ver**: la sección **Información general** muestra la configuración clave de la dirección IP pública, como la interfaz de red a la que está asociada (si la dirección está asociada a una interfaz de red). El portal no muestra la versión de la dirección (IPv4 o IPv6). Para ver información sobre la versión, use el comando de la CLI o PowerShell para ver la dirección IP pública. Si la versión de la dirección IP es IPv6, ni el portal, ni PowerShell ni la CLI muestran la dirección asignada.
   - **Eliminar**: para eliminar la dirección IP pública, seleccione **Eliminar** en la sección **Información general**. Si la dirección está actualmente asociada a una configuración de IP, no se puede eliminar. Si la dirección está asociada actualmente con una configuración, seleccione **Desasociar** para desasociar la dirección de la configuración de IP.
   - **Cambiar**: seleccione **Configuración**. Cambie la configuración según se indica en el paso 4 de [Crear una dirección IP pública](#create-a-public-ip-address). Para cambiar la asignación de una dirección IPv4 de estática a dinámica, primero debe desasociar la dirección IPv4 pública de la configuración de IP a que está asociada. A continuación, puede cambiar el método de asignación a dinámico y seleccionar **Asociar** para asociar la dirección IP a la misma configuración de IP, una diferente o dejarla desasociada. Para desasociar una dirección IP pública, en la sección **Información general**, seleccione **Desasociar**.

   >[!WARNING]
   >Al cambiar el método de asignación de estático a dinámico, perderá la dirección IP que se asignó a la dirección IP pública. Aunque los servidores DNS públicos de Azure mantienen una asignación entre las direcciones estáticas o dinámicas y cualquier etiqueta de nombre DNS (si se han definido), las direcciones IP dinámicas pueden cambiar cuando se inicia la máquina virtual desde el estado detenido (desasignado). Para evitar que la dirección cambie, asigne una dirección IP estática.

**Comandos**

|Herramienta|Get-Help|
|---|---|
|CLI|[az network public-ip list](/cli/azure/network/public-ip#az-network-public-ip-list) para mostrar las direcciones IP públicas, [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show) para mostrar la configuración, [az network public-ip update](/cli/azure/network/public-ip#az-network-public-ip-update) para actualizar, [az network public-ip delete](/cli/azure/network/public-ip#az-network-public-ip-delete) para eliminar|
|PowerShell|[Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) para recuperar un objeto de dirección IP pública y ver su configuración, [Set-AzPublicIpAddress](/powershell/module/az.network/set-azpublicipaddress) para actualizar la configuración, [Remove-AzPublicIpAddress](/powershell/module/az.network/remove-azpublicipaddress) para eliminar|

## <a name="assign-a-public-ip-address"></a>Asignación de una dirección IP pública

Aprenda a asignar una dirección IP pública a los siguientes recursos:

- Una máquina virtual [Windows](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o [Linux](../virtual-machines/linux/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (durante la creación), o una [máquina virtual ya existente](virtual-network-network-interface-addresses.md#add-ip-addresses)
- [Equilibrador de carga accesible desde Internet](../load-balancer/load-balancer-get-started-internet-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Introducción a Puerta de enlace de aplicaciones](../application-gateway/application-gateway-create-gateway-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Conexión de sitio a sitio con Azure VPN Gateway](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Conjunto de escalado de máquinas virtuales de Azure](../virtual-machine-scale-sets/virtual-machine-scale-sets-portal-create.md?toc=%2fazure%2fvirtual-network%2ftoc.json)

## <a name="permissions"></a>Permisos

Para realizar tareas en direcciones IP públicas, su cuenta debe estar asignada al rol de [colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o a un rol [personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones adecuadas que se muestran en la tabla siguiente:

| Acción                                                             | Nombre                                                           |
| ---------                                                          | -------------                                                  |
| Microsoft.Network/publicIPAddresses/read                           | Leer una dirección IP pública                                          |
| Microsoft.Network/publicIPAddresses/write                          | Crear o actualizar una dirección IP pública                           |
| Microsoft.Network/publicIPAddresses/delete                         | Eliminar una dirección IP pública                                     |
| Microsoft.Network/publicIPAddresses/join/action                    | Asociar una dirección IP pública a un recurso                    |

## <a name="next-steps"></a>Pasos siguientes

- Crear una dirección IP pública con scripts de ejemplo de [PowerShell](powershell-samples.md) o [CLI de Azure](cli-samples.md) o con [plantillas de Azure Resource Manager](template-samples.md)
- Crear y asignar [definiciones de Azure Policy](policy-samples.md) para direcciones IP públicas
