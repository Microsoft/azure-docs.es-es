---
title: Creación, modificación o eliminación del prefijo de una dirección IP pública de Azure
titlesuffix: Azure Virtual Network
description: Obtenga información sobre cómo crear, cambiar o eliminar el prefijo de una dirección IP pública.
services: virtual-network
documentationcenter: na
author: anavinahar
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/13/2019
ms.author: anavin
ms.openlocfilehash: 82ee9d04785fc0f6ac534428bf411ca0fe3204ad
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65601513"
---
# <a name="create-change-or-delete-a-public-ip-address-prefix"></a>Creación, modificación o eliminación del prefijo de una dirección IP pública

Obtenga información sobre el prefijo de una dirección IP pública y cómo crearlo, modificarlo y eliminarlo. El prefijo de una dirección IP pública es un intervalo contiguo de direcciones basado en el número de direcciones IP públicas que especifique. Las direcciones se asignan a su suscripción. Cuando se crea un recurso de dirección IP pública, puede asignar una dirección IP pública estática desde el prefijo y asociar la dirección de las máquinas virtuales, equilibradores de carga u otros recursos, para habilitar la conectividad a internet. Si no está familiarizado con los prefijos de las direcciones IP públicas, consulte [Información general sobre el prefijo de una dirección IP pública](public-ip-address-prefix.md)

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Complete las tareas siguientes antes de seguir los pasos de las secciones de este artículo:

- Si todavía no tiene una cuenta de Azure, regístrese para obtener una [cuenta de evaluación gratuita](https://azure.microsoft.com/free).
- Si usa el portal, abra https://portal.azure.com e inicie sesión con la cuenta de Azure.
- Si usa comandos de PowerShell para completar las tareas de este artículo, ejecute los comandos que se encuentran en [Azure Cloud Shell](https://shell.azure.com/powershell) o ejecute PowerShell en el equipo. Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. Para realizar este tutorial, se necesita la versión 1.0.0 del módulo de Azure PowerShell u otra posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.
- Si usa la interfaz de la línea de comandos (CLI) de Azure para completar las tareas de este artículo, ejecute los comandos que se encuentran en [Azure Cloud Shell](https://shell.azure.com/bash) o ejecute la CLI en el equipo. Para realizar este tutorial, es necesaria la versión 2.0.41 de la CLI de Azure o una versión posterior. Ejecute `az --version` para buscar la versión instalada. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure 2.0](/cli/azure/install-azure-cli). Si ejecuta de forma local la CLI de Azure, también debe ejecutar `az login` para crear una conexión con Azure.

La cuenta en la que inicia sesión o con la que se conecta a Azure debe tener asignado el rol de [colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o un [rol personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones apropiadas en [Permisos](#permissions).

Los prefijos de las direcciones IP públicas tienen un cargo. Para obtener información detallada, consulte [Precios](https://azure.microsoft.com/pricing/details/ip-addresses).

## <a name="create-a-public-ip-address-prefix"></a>Creación del prefijo de una dirección IP pública

1. En la esquina superior izquierda del portal, seleccione **+ Crear un recurso**.
2. Escriba *Prefijo de dirección IP pública* en el cuadro *Búsqueda en Marketplace*. Seleccione **Prefijo de dirección IP pública** cuando aparezca en los resultados de búsqueda.
3. En **Prefijo de dirección IP pública**, seleccione **Crear**.
4. Escriba o seleccione valores para las siguientes opciones, en **Crear prefijo de dirección IP pública**, y seleccione **Crear**:

   |Configuración|¿Necesario?|Detalles|
   |---|---|---|
   |Subscription|Sí|Debe existir en la misma [suscripción](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#subscription) que el recurso al cual desee asociar la dirección IP pública.|
   |Grupos de recursos|Sí|Puede existir en la misma suscripción que el [grupo de recursos](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#resource-group) al cual desee asociar la dirección IP pública o en otra diferente.|
   |NOMBRE|Sí|El nombre debe ser único dentro del grupo de recursos que seleccione.|
   |Área|Sí|Debe existir en la misma [región](https://azure.microsoft.com/regions) que las direcciones IP públicas que asignará a direcciones del intervalo. El prefijo está actualmente en versión preliminar en centro-oeste de EE. UU., oeste de EE. UU., oeste de EE. UU. 2, centro de EE. UU., Europa del norte, Europa occidental y sudeste asiático.|
   |Tamaño del prefijo|Sí| El tamaño del prefijo que necesita. A/28 o 16 direcciones IP es el valor predeterminado.

**Comandos**

|Tool|Get-Help|
|---|---|
|CLI|[az network public-ip prefix create](/cli/azure/network/public-ip/prefix#az-network-public-ip-prefix-create)|
|PowerShell|[New-AzPublicIpPrefix](/powershell/module/az.network/new-azpublicipprefix)|

## <a name="create-a-static-public-ip-address-from-a-prefix"></a>Creación de una dirección IP pública estática a partir de un prefijo
Una vez que cree un prefijo, debe crear las direcciones IP estáticas a partir del prefijo. Para ello, siga estos pasos.

1. En el cuadro que contiene el texto *Buscar recursos*, en la parte superior de Azure Portal, escriba *Prefijo de dirección IP pública*. Seleccione **Prefijos de direcciones IP públicas** cuando aparezca en los resultados de búsqueda.
2. Seleccione el prefijo a partir del cual quiere crear direcciones IP públicas.
3. Cuando aparezca en los resultados de búsqueda, selecciónelo y haga clic en **+Agregar dirección IP** en la sección Información general.
4. Escriba o seleccione valores para las siguientes opciones, en **Crear dirección IP pública**. Puesto que un prefijo es para SKU estándar, IPv4 y estático, solo deberá proporcionar la información siguiente:

   |Configuración|¿Necesario?|Detalles|
   |---|---|---|
   |NOMBRE|Sí|El nombre de la dirección IP pública debe ser único dentro del grupo de recursos que seleccione.|
   |Tiempo de espera de inactividad (minutos)|No|Cantidad de minutos para mantener una conexión TCP o HTTP abierta sin que dependa del envío de mensajes de mantenimiento de los clientes. |
   |Etiqueta de nombre DNS|No|Debe ser único dentro de la región de Azure en la que cree el nombre (entre todas las suscripciones y todos los clientes). Azure registra automáticamente el nombre y la dirección IP en el DNS para que pueda conectarse a un recurso con el nombre. Azure anexa una subred predeterminada, como *location.cloudapp.azure.com* (en que location es la ubicación seleccionada), al nombre que proporcione para crear el nombre DNS cualificado completo. Para obtener más información, consulte [Utilizar Azure DNS con una dirección IP pública de Azure](../dns/dns-custom-domain.md?toc=%2fazure%2fvirtual-network%2ftoc.json#public-ip-address).|

## <a name="view-or-delete-a-prefix"></a>Ver o eliminar un prefijo

1. En el cuadro que contiene el texto *Buscar recursos*, en la parte superior de Azure Portal, escriba *Prefijo de dirección IP pública*. Seleccione **Prefijos de direcciones IP públicas** cuando aparezca en los resultados de búsqueda.
2. Seleccione el nombre del prefijo de la dirección IP pública que quiera ver, cambiar la configuración o eliminar de la lista.
3. Complete una de las siguientes opciones en función de si desea ver, eliminar o cambiar el prefijo de la dirección IP pública.
   - **Ver**: en la sección **Información general** se muestra la configuración clave del prefijo de una dirección IP pública, como prefijo.
   - **Eliminar**: para eliminar el prefijo de una dirección IP pública, seleccione **Eliminar** en la sección **Información general**. Si las direcciones dentro del prefijo están asociadas a recursos de dirección IP pública, primero debe eliminar dichos recursos de dirección IP pública. Consulte [Eliminar una dirección IP pública](virtual-network-public-ip-address.md#view-change-settings-for-or-delete-a-public-ip-address).

**Comandos**

|Tool|Get-Help|
|---|---|
|CLI|[az network public-ip prefix list](/cli/azure/network/public-ip/prefix#az-network-public-ip-prefix-list) para mostrar las direcciones IP públicas, [az network public-ip prefix show](/cli/azure/network/public-ip/prefix#az-network-public-ip-prefix-show) para mostrar la configuración, [az network public-ip prefix update](/cli/azure/network/public-ip/prefix#az-network-public-ip-prefix-update) para actualizar, [az network public-ip prefix delete](/cli/azure/network/public-ip/prefix#az-network-public-ip-prefix-delete) para eliminar|
|PowerShell|[Get-AzPublicIpPrefix](/powershell/module/az.network/get-azpublicipprefix) para recuperar un objeto de dirección IP pública y ver su configuración, [conjunto AzPublicIpPrefix](/powershell/module/az.network/set-azpublicipprefix) para actualizar la configuración; [Remove-AzPublicIpPrefix](/powershell/module/az.network/remove-azpublicipprefix) para eliminar|

## <a name="permissions"></a>Permisos

Para realizar tareas en prefijos de direcciones IP públicas, su cuenta debe estar asignada al rol de [colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o a un rol [personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones adecuadas que se muestran en la tabla siguiente:

| .                                                            | NOMBRE                                                           |
| ---------                                                         | -------------                                                  |
| Microsoft.Network/publicIPPrefixes/read                           | Leer el prefijo de una dirección IP pública                                |
| Microsoft.Network/publicIPPrefixes/write                          | Crear o actualizar el prefijo de una dirección IP pública                    |
| Microsoft.Network/publicIPPrefixes/delete                         | Eliminación del prefijo de una dirección IP pública                              |
|Microsoft.Network/publicIPPrefixes/join/action                     | Creación de una dirección IP pública a partir de un prefijo |

## <a name="next-steps"></a>Pasos siguientes

- Obtener información sobre los escenarios y las ventajas de usar un [prefijo de IP pública](public-ip-address-prefix.md)
