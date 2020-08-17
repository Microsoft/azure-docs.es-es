---
title: Creación, modificación o eliminación de un grupo de seguridad de red de Azure
titlesuffix: Azure Virtual Network
description: Aprenda dónde encontrar información sobre las reglas de seguridad y cómo crear, cambiar o eliminar un grupo de seguridad de red.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/13/2020
ms.author: kumud
ms.openlocfilehash: dfb6426ec4e75f6484df37008522b966ebc3af6f
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87281266"
---
# <a name="create-change-or-delete-a-network-security-group"></a>Crear, modificar o eliminar un grupo de seguridad de red

Las reglas de seguridad de grupos de seguridad de red permiten filtrar el tipo de tráfico de red que puede fluir dentro y fuera de las interfaces de red y las subredes de redes virtuales. Para más información acerca de los grupos de seguridad de red, consulte [Seguridad de las redes](security-overview.md). Después, complete el tutorial [Filtrado del tráfico de red](tutorial-filter-network-traffic.md) para obtener experiencia con los grupos de seguridad de red.

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Si no tiene ninguna, configure una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). Complete una de estas tareas antes de continuar con este artículo:

- **Usuarios de Portal**: Inicie sesión en [Azure Portal](https://portal.azure.com) con su cuenta de Azure.

- **Usuarios de PowerShell**: ejecute los comandos en [Azure Cloud Shell](https://shell.azure.com/powershell), o bien ejecute PowerShell en el equipo. Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. En la pestaña del explorador Azure Cloud Shell, busque la lista desplegable **Seleccionar entorno** y, después, elija **PowerShell** si aún no está seleccionado.

    Si ejecuta PowerShell en el entorno local, use la versión del módulo de Azure PowerShell 1.0.0 o una posterior. Ejecute `Get-Module -ListAvailable Az.Network` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Ejecute `Connect-AzAccount` para crear una conexión con Azure.

- **Usuarios de la interfaz de la línea de comandos (CLI) de Azure**: ejecute los comandos en [Azure Cloud Shell](https://shell.azure.com/bash), o bien ejecute la CLI en el equipo. Use la versión 2.0.28 de la CLI de Azure o una posterior si ejecuta la CLI de Azure en el entorno local. Ejecute `az --version` para buscar la versión instalada. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Ejecute `az login` para crear una conexión con Azure.

La cuenta en la que inicia sesión o con la que se conecta a Azure debe tener asignado el [rol Colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o un [rol personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones apropiadas en [Permisos](#permissions).

## <a name="work-with-network-security-groups"></a>Trabajar con grupos de seguridad de red

Los grupos de seguridad de red se pueden crear, [ver todos](#view-all-network-security-groups), [ver sus detalles](#view-details-of-a-network-security-group), [cambiar](#change-a-network-security-group) y [eliminar](#delete-a-network-security-group). También puede [asociar o desasociar](#associate-or-dissociate-a-network-security-group-to-or-from-a-subnet-or-network-interface) un grupo de seguridad de red de una interfaz de red o subred.

### <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

Existe un límite para la cantidad de grupos de seguridad de red que puede crear por suscripción y ubicación de Azure. Para obtener más información, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

1. En el menú [Azure Portal](https://portal.azure.com) o en la página **Inicio**, seleccione **Crear un recurso**.

2. Seleccione **Redes** y, a continuación, **Grupo de seguridad de red**.

3. En la página **Crear grupo de seguridad de red**, en la pestaña **Aspectos básicos**, establezca los valores de configuración siguientes:

    | Configuración | Acción |
    | --- | --- |
    | **Suscripción** | Elija su suscripción. |
    | **Grupos de recursos** | Seleccione un grupo de recursos existente o seleccione **Crear nuevo** para crear uno nuevo. |
    | **Nombre** | Escriba una cadena de texto única en un grupo de recursos. |
    | **Región** | Elija la ubicación que desee. |

4. Seleccione **Revisar + crear**.

5. Cuando vea el mensaje **Validación superada**, seleccione **Crear**.

#### <a name="commands"></a>Comandos:

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network nsg create](/cli/azure/network/nsg#az-network-nsg-create) |
| PowerShell | [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) |

### <a name="view-all-network-security-groups"></a>Ver todos los grupos de seguridad de red

Diríjase a [Azure Portal](https://portal.azure.com) para ver sus grupos de seguridad de red. Busque y seleccione **Grupos de seguridad de red**. Aparecerá la lista de grupos de seguridad de red de su suscripción.

#### <a name="commands"></a>Comandos:

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network nsg list](/cli/azure/network/nsg#az-network-nsg-list) |
| PowerShell | [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup) |

### <a name="view-details-of-a-network-security-group"></a>Ver detalles de un grupo de seguridad de red

1. Diríjase a [Azure Portal](https://portal.azure.com) para ver sus grupos de seguridad de red. Busque y seleccione **Grupos de seguridad de red**.

2. Seleccione el nombre del grupo de seguridad de red.

En la barra de menús del grupo de seguridad de red, en **Configuración**, puede ver las opciones de **Reglas de seguridad de entrada**, **Reglas de seguridad de salida**, **Interfaces de red** y **Subredes** a las que está asociado el grupo de seguridad de red.

En **Supervisión**, puede habilitar o deshabilitar **Configuración de diagnóstico**. En **Soporte técnico y solución de problemas**, puede ver **Reglas de seguridad vigentes**. Para más información, consulte [Registros de diagnóstico de un grupo de seguridad de red](virtual-network-nsg-manage-log.md) y [Diagnóstico de problemas al filtrar el tráfico de red de una VM](diagnose-network-traffic-filter-problem.md).

Para más información sobre la configuración común de Azure que se muestra, consulte los artículos siguientes:

- [Registro de actividad](../azure-monitor/platform/platform-logs-overview.md)
- [Control de acceso (IAM)](../role-based-access-control/overview.md)
- [Etiquetas](../azure-resource-manager/management/tag-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Bloqueos](../azure-resource-manager/management/lock-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Script de Automation](../azure-resource-manager/templates/export-template-portal.md)

#### <a name="commands"></a>Comandos:

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network nsg show](/cli/azure/network/nsg#az-network-nsg-show) |
| PowerShell | [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup) |

### <a name="change-a-network-security-group"></a>Cambiar un grupo de seguridad de red

1. Diríjase a [Azure Portal](https://portal.azure.com) para ver sus grupos de seguridad de red. Busque y seleccione **Grupos de seguridad de red**.

2. Seleccione el nombre del grupo de seguridad de red que quiere modificar.

Los cambios más comunes son [agregar una regla de seguridad](#create-a-security-rule), [quitar una regla de seguridad](#delete-a-security-rule) y [asociar un grupo de seguridad de red a una subred o una interfaz de red, o desasociarlo](#associate-or-dissociate-a-network-security-group-to-or-from-a-subnet-or-network-interface).

#### <a name="commands"></a>Comandos:

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network nsg update](/cli/azure/network/nsg#az-network-nsg-update) |
| PowerShell | [Set-AzNetworkSecurityGroup](/powershell/module/az.network/set-aznetworksecuritygroup) |

### <a name="associate-or-dissociate-a-network-security-group-to-or-from-a-subnet-or-network-interface"></a>Asociar o desasociar un grupo de seguridad de red a o de una subred o una interfaz de red

Para asociar un grupo de seguridad de red a una interfaz de red o desasociarlo de esta, consulte [Asociar un grupo de seguridad de red o una interfaz de red o desasociarlo de esta](virtual-network-network-interface.md#associate-or-dissociate-a-network-security-group). Para asociar un grupo de seguridad de red a una subred o desasociarlo de esta, consulte [Modificación de la configuración de subred](virtual-network-manage-subnet.md#change-subnet-settings).

### <a name="delete-a-network-security-group"></a>Eliminar un grupo de seguridad de red

Si un grupo de seguridad de red está asociado a subredes o interfaces de red, no se puede eliminar. Desasocie un grupo de seguridad de red de todas las subredes e interfaces de red antes de intentar eliminarlo.

1. Diríjase a [Azure Portal](https://portal.azure.com) para ver sus grupos de seguridad de red. Busque y seleccione **Grupos de seguridad de red**.

2. Seleccione el nombre del grupo de seguridad de red que quiere eliminar.

3. En la barra de herramientas del grupo de seguridad de red, seleccione **Eliminar**. A continuación, seleccione **Sí** en el cuadro de diálogo de confirmación.

#### <a name="commands"></a>Comandos:

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network nsg delete](/cli/azure/network/nsg#az-network-nsg-delete) |
| PowerShell | [Remove-AzNetworkSecurityGroup](/powershell/module/az.network/remove-aznetworksecuritygroup) |

## <a name="work-with-security-rules"></a>Trabajar con reglas de seguridad

Un grupo de seguridad de red puede contener una regla de seguridad, varias o ninguna. Las reglas de seguridad se pueden crear, [ver todas](#view-all-security-rules), [ver sus detalles](#view-details-of-a-security-rule), [cambiar](#change-a-security-rule) y [eliminar](#delete-a-security-rule).

### <a name="create-a-security-rule"></a>Creación de una regla de seguridad

Existe un límite para la cantidad de reglas por grupo de seguridad de red que puede crear por suscripción y ubicación de Azure. Para obtener más información, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

1. Diríjase a [Azure Portal](https://portal.azure.com) para ver sus grupos de seguridad de red. Busque y seleccione **Grupos de seguridad de red**.

2. Seleccione el nombre del grupo de seguridad de red al que quiere agregar una regla de seguridad.

3. En la barra de menús del grupo de seguridad de red, elija **Reglas de seguridad de entrada** o **Reglas de seguridad de salida**.

    Se muestran varias reglas existentes, incluidas algunas que quizás no haya agregado. Cuando se crea un grupo de seguridad de red, se crean en este varias reglas de seguridad predeterminadas. Para más información, consulte las [reglas de seguridad predeterminadas](security-overview.md#default-security-rules).  Las reglas de seguridad predeterminadas no se pueden eliminar, pero pueden reemplazarse por reglas de prioridad más alta.

4. <a name="security-rule-settings"></a> Seleccione **Agregar**. Seleccione o agregue valores para las siguientes opciones y, continuación, seleccione **Aceptar**:

    | Configuración | Value | Detalles |
    | ------- | ----- | ------- |
    | **Origen** | Uno de los valores siguientes:<ul><li>**Cualquiera**</li><li>**Direcciones IP**</li><li>**Etiqueta de servicio** (regla de seguridad de entrada) o **VirtualNetwork** (regla de seguridad de salida)</li><li>**Grupo de&nbsp;seguridad de&nbsp;aplicaciones**</li></ul> | <p>Si elige **Direcciones IP**, también debe especificar **Intervalos de direcciones IP de origen y CIDR**.</p><p>Si elige **Etiqueta de servicio**, también puede seleccionar una opción de **Etiqueta de servicio de origen**.</p><p>Si elige **Grupo de seguridad de aplicaciones**, también debe seleccionar un grupo de seguridad de aplicaciones existente. Si elige **Grupo de seguridad de aplicaciones** en **Origen** y **Destino**, las interfaces de red de ambos grupos de seguridad de aplicaciones deben estar en la misma red virtual.</p> |
    | **Intervalos de direcciones IP de origen y CIDR** | Lista delimitada por comas de direcciones IP e intervalos de Enrutamiento de interdominios sin clases (CIDR) | <p>Esta opción aparece si cambia **Origen** a **Direcciones IP**. Debe especificar un valor único o una lista de varios valores separados por comas. Un ejemplo de varios valores es `10.0.0.0/16, 192.188.1.1`. Se aplican límites al número de valores que puede especificar. Para más información, consulte [Límites de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).</p><p>Si la dirección IP que especifica se asigna a una VM de Azure, especifique su dirección IP privada, no la pública. Azure procesa las reglas de seguridad después de cambiar la dirección IP pública por una dirección IP privada para las reglas de seguridad de entrada, pero antes de cambiar una dirección IP privada por una dirección IP pública para las reglas de salida. Para más información sobre las direcciones IP públicas y privadas de Azure, consulte [Tipos de direcciones IP](virtual-network-ip-addresses-overview-arm.md).</p> |
    | **Etiqueta de servicio de origen** | Una etiqueta de servicio de la lista desplegable | Esta opción de configuración opcional aparece si se establece **Origen** en **Etiqueta de servicio** para una regla de seguridad de entrada. Una etiqueta de servicio es un identificador predefinido para una categoría de direcciones IP. Para más información sobre las etiquetas de servicio disponibles y qué representa cada etiqueta, consulte [Etiquetas de servicio](security-overview.md#service-tags). |
    | **Grupo de seguridad de aplicación de origen** | Grupo de seguridad de aplicaciones existente | Esta opción de configuración aparece si se establece **Origen** en **Grupo de seguridad de aplicaciones**. Seleccione un grupo de seguridad de aplicaciones que exista en la misma región que la interfaz de red. Obtenga información sobre cómo [crear un grupo de seguridad de aplicaciones](#create-an-application-security-group). |
    | **Intervalos de puertos de origen** | Uno de los valores siguientes:<ul><li>Un puerto único, como `80`</li><li>Un intervalo de puertos, como `1024-65535`</li><li>Una lista de puertos únicos o intervalos de puertos separados por comas, como `80, 1024-65535`</li><li>Un asterisco (`*`) para permitir el tráfico en cualquier puerto</li></ul> | Esta opción de configuración especifica los puertos en los que la regla permite o deniega el tráfico. Se aplican límites al número de puertos que puede especificar. Para más información, consulte [Límites de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits). |
    | **Destino** | Uno de los valores siguientes:<ul><li>**Cualquiera**</li><li>**Direcciones IP**</li><li>**Etiqueta de servicio** (regla de seguridad de salida) o **VirtualNetwork** (regla de seguridad de entrada)</li><li>**Grupo de&nbsp;seguridad de&nbsp;aplicaciones**</li></ul> | <p>Si selecciona **Direcciones IP**, especifique también los **Intervalos de direcciones IP de destino y CIDR**.</p><p>Si elige **VirtualNetwork**, se permite el tráfico a todas las direcciones IP del espacio de direcciones de la red virtual. **VirtualNetwork** es una etiqueta de servicio.</p><p>Si selecciona **Grupo de seguridad de aplicaciones**, debe seleccionar un grupo de seguridad de aplicaciones existente. Obtenga información sobre cómo [crear un grupo de seguridad de aplicaciones](#create-an-application-security-group).</p> |
    | **Intervalos de direcciones IP de destino y CIDR** | Lista de direcciones IP e intervalos CIDR delimitados por comas | <p>Esta opción aparece si cambia **Destino** a **Direcciones IP**. De manera similar a **Origen** e **Intervalos de direcciones IP de origen y CIDR**, puede especificar uno o varios intervalos o direcciones. Se aplican límites al número que puede especificar. Para más información, consulte [Límites de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).</p><p>Si la dirección IP que especifica se asigna a una VM de Azure, asegúrese de especificar su dirección IP privada, no la pública. Azure procesa las reglas de seguridad después de cambiar la dirección IP pública por una dirección IP privada para las reglas de seguridad de entrada, pero antes de cambiar una dirección IP privada por una dirección IP pública para las reglas de salida. Para más información sobre las direcciones IP públicas y privadas de Azure, consulte [Tipos de direcciones IP](virtual-network-ip-addresses-overview-arm.md).</p> |
    | **Etiqueta de servicio de destino** | Una etiqueta de servicio de la lista desplegable | Esta opción de configuración opcional aparece si se establece **Destino** en **Etiqueta de servicio** para una regla de seguridad de salida. Una etiqueta de servicio es un identificador predefinido para una categoría de direcciones IP. Para más información sobre las etiquetas de servicio disponibles y qué representa cada etiqueta, consulte [Etiquetas de servicio](security-overview.md#service-tags). |
    | **Grupo de seguridad de aplicación de destino** | Grupo de seguridad de aplicaciones existente | Esta opción de configuración aparece si se establece **Destino** en **Grupo de seguridad de aplicaciones**. Seleccione un grupo de seguridad de aplicaciones que exista en la misma región que la interfaz de red. Obtenga información sobre cómo [crear un grupo de seguridad de aplicaciones](#create-an-application-security-group). |
    | **Intervalos de puertos de destino** | Uno de los valores siguientes:<ul><li>Un puerto único, como `80`</li><li>Un intervalo de puertos, como `1024-65535`</li><li>Una lista de puertos únicos o intervalos de puertos separados por comas, como `80, 1024-65535`</li><li>Un asterisco (`*`) para permitir el tráfico en cualquier puerto</li></ul> | Como con **Intervalos de puertos de origen**, puede especificar uno o varios puertos e intervalos. Se aplican límites al número que puede especificar. Para más información, consulte [Límites de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits). |
    | **Protocolo** | **Cualquiera**, **TCP**, **UDP** o **ICMP** | Puede restringir la regla al Protocolo de control de transmisión (TCP), el Protocolo de datagramas de usuario (UDP) y el Protocolo de mensajes de control de Internet (ICMP). La opción predeterminada es que la regla se aplique a todos los protocolos. |
    | **Acción** | **Permitir** o **Denegar** | Esta opción de configuración especifica si esta regla permite o deniega el acceso a la configuración de origen y de destino proporcionada. |
    | **Prioridad** | Escriba un valor de 100 a 4096 que sea único para todas las reglas de seguridad del grupo de seguridad de red | Azure procesa las reglas de seguridad en orden de prioridad. Cuanto menor sea el número, mayor será la prioridad. Se recomienda dejar un espacio entre los números de prioridad al crear reglas, como 100, 200 y 300. Dejar espacios facilita la adición de reglas en el futuro, de modo que pueda asignarles una prioridad por encima o por debajo de las reglas existentes. |
    | **Nombre** | Nombre único para la regla en el grupo de seguridad de red | Puede tener hasta 80 caracteres. Debe comenzar con una letra o un número y terminar con una letra, un número o un carácter de subrayado. El nombre solo puede contener letras, números, caracteres de subrayado, puntos o guiones. |
    | **Descripción** | Descripción de texto | Opcionalmente, puede especificar una descripción de texto para la regla de seguridad. |

#### <a name="commands"></a>Comandos:

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network nsg rule create](/cli/azure/network/nsg/rule#az-network-nsg-rule-create) |
| PowerShell | [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) |

### <a name="view-all-security-rules"></a>Ver todas las reglas de seguridad

Un grupo de seguridad de red contiene varias reglas o ninguna. Para más detalles sobre la información que aparece al ver las rutas, consulte la [introducción a la seguridad de red](security-overview.md).

1. Diríjase a [Azure Portal](https://portal.azure.com) para ver las reglas de un grupo de seguridad de red. Busque y seleccione **Grupos de seguridad de red**.

2. Seleccione el nombre del grupo de seguridad de red cuyas reglas quiere ver.

3. En la barra de menús del grupo de seguridad de red, elija **Reglas de seguridad de entrada** o **Reglas de seguridad de salida**.

La lista contiene todas las reglas que ha creado y las [reglas de seguridad predeterminadas](security-overview.md#default-security-rules) del grupo de seguridad de red.

#### <a name="commands"></a>Comandos:

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network nsg rule list](/cli/azure/network/nsg/rule#az-network-nsg-rule-list) |
| PowerShell | [Get-AzNetworkSecurityRuleConfig](/powershell/module/az.network/get-aznetworksecurityruleconfig) |

### <a name="view-details-of-a-security-rule"></a>Ver detalles de una regla de seguridad

1. Diríjase a [Azure Portal](https://portal.azure.com) para ver las reglas de un grupo de seguridad de red. Busque y seleccione **Grupos de seguridad de red**.

2. Seleccione el nombre del grupo de seguridad de red del que quiere ver los detalles de una de sus reglas.

3. En la barra de menús del grupo de seguridad de red, elija **Reglas de seguridad de entrada** o **Reglas de seguridad de salida**.

4. Seleccione la regla cuyos detalles quiere ver. Para obtener una explicación de todas las opciones, consulte [Configuración de reglas de seguridad](#security-rule-settings).

    > [!NOTE]
    > Este procedimiento solo se aplica a una regla de seguridad personalizada. No funciona si elige una regla de seguridad predeterminada.

#### <a name="commands"></a>Comandos:

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network nsg rule show](/cli/azure/network/nsg/rule#az-network-nsg-rule-show) |
| PowerShell | [Get-AzNetworkSecurityRuleConfig](/powershell/module/az.network/get-aznetworksecurityruleconfig) |

### <a name="change-a-security-rule"></a>Cambiar una regla de seguridad

1. Complete los pasos de [Ver detalles de una regla de seguridad](#view-details-of-a-security-rule).

2. Cambie la configuración según sea necesario y, a continuación, seleccione **Guardar**. Para obtener una explicación de todas las opciones, consulte [Configuración de reglas de seguridad](#security-rule-settings).

    > [!NOTE]
    > Este procedimiento solo se aplica a una regla de seguridad personalizada. No se permite cambiar una regla de seguridad predeterminada.

#### <a name="commands"></a>Comandos:

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network nsg rule update](/cli/azure/network/nsg/rule#az-network-nsg-rule-update) |
| PowerShell | [Set-AzNetworkSecurityRuleConfig](/powershell/module/az.network/set-aznetworksecurityruleconfig) |

### <a name="delete-a-security-rule"></a>Eliminar una regla de seguridad

1. Complete los pasos de [Ver detalles de una regla de seguridad](#view-details-of-a-security-rule).

2. Seleccione **Eliminar** y, luego, seleccione **Sí**.

    > [!NOTE]
    > Este procedimiento solo se aplica a una regla de seguridad personalizada. No se permite eliminar una regla de seguridad predeterminada.

#### <a name="commands"></a>Comandos:

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network nsg rule delete](/cli/azure/network/nsg/rule#az-network-nsg-rule-delete) |
| PowerShell | [Remove-AzNetworkSecurityRuleConfig](/powershell/module/az.network/remove-aznetworksecurityruleconfig) |

## <a name="work-with-application-security-groups"></a>Trabajar con grupos de seguridad de aplicaciones

Un grupo de seguridad de aplicaciones contiene una interfaz de red, varias o ninguna. Para más información, consulte [grupos de seguridad de aplicaciones](security-overview.md#application-security-groups). Todas las interfaces de red de un grupo de seguridad de aplicaciones deben existir en la misma red virtual. Para obtener información sobre cómo agregar una interfaz de red a un grupo de seguridad de aplicaciones, consulte el tema sobre cómo [agregar una interfaz de red a un grupo de seguridad de aplicaciones](virtual-network-network-interface.md#add-to-or-remove-from-application-security-groups).

### <a name="create-an-application-security-group"></a>Crear un grupo de seguridad de aplicaciones

1. En el menú [Azure Portal](https://portal.azure.com) o en la página **Inicio**, seleccione **Crear un recurso**.

2. En el cuadro de búsqueda, escriba *Grupo de seguridad de aplicaciones*.

3. En la página **Grupo de seguridad de aplicaciones**, seleccione **Crear**.

4. En la página **Crear un grupo de seguridad de aplicación**, en la pestaña **Aspectos básicos**, establezca los valores de configuración siguientes:

    | Configuración | Acción |
    | --- | --- |
    | **Suscripción** | Elija su suscripción. |
    | **Grupos de recursos** | Seleccione un grupo de recursos existente o seleccione **Crear nuevo** para crear uno nuevo. |
    | **Nombre** | Escriba una cadena de texto única en un grupo de recursos. |
    | **Región** | Elija la ubicación que desee. |

5. Seleccione **Revisar + crear**.

6. En la pestaña **Revisar y crear**, cuando aparezca el mensaje **Validación superada**, seleccione **Crear**.

#### <a name="commands"></a>Comandos:

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network asg create](/cli/azure/network/asg#az-network-asg-create) |
| PowerShell | [New-AzApplicationSecurityGroup](/powershell/module/az.network/new-azapplicationsecuritygroup) |

### <a name="view-all-application-security-groups"></a>Ver todos los grupos de seguridad de aplicaciones

Vaya a [Azure Portal](https://portal.azure.com) para ver sus grupos de seguridad de aplicaciones. Busque y seleccione **Grupos de seguridad de aplicaciones**. Azure Portal muestra una lista de sus grupos de seguridad de aplicaciones.

#### <a name="commands"></a>Comandos:

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network asg list](/cli/azure/network/asg#az-network-asg-list) |
| PowerShell | [Get-AzApplicationSecurityGroup](/powershell/module/az.network/get-azapplicationsecuritygroup) |

### <a name="view-details-of-a-specific-application-security-group"></a>Ver detalles de un grupo de seguridad de aplicaciones específico

1. Diríjase a [Azure Portal](https://portal.azure.com) para ver un grupo de seguridad de aplicaciones. Busque y seleccione **Grupos de seguridad de aplicaciones**.

2. Seleccione el nombre del grupo de seguridad de aplicaciones del que quiera ver los detalles.

#### <a name="commands"></a>Comandos:

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network asg show](/cli/azure/network/asg#az-network-asg-show) |
| PowerShell | [Get-AzApplicationSecurityGroup](/powershell/module/az.network/get-azapplicationsecuritygroup) |

### <a name="change-an-application-security-group"></a>Cambiar un grupo de seguridad de aplicaciones

1. Diríjase a [Azure Portal](https://portal.azure.com) para ver un grupo de seguridad de aplicaciones. Busque y seleccione **Grupos de seguridad de aplicaciones**.

2. Seleccione el nombre del grupo de seguridad de aplicaciones que quiere cambiar.

3. Seleccione **Cambiar** junto a la opción de configuración que desea modificar. Por ejemplo, puede agregar o quitar **Etiquetas**, o bien cambiar los valores de **Grupo de recursos** o **Suscripción**.

    > [!NOTE]
    > No puede cambiar la ubicación.

    En la barra de menús, también puede seleccionar **Control de acceso (IAM)** . En la página **Control de acceso (IAM)** , puede asignar o quitar permisos para el grupo de seguridad de aplicaciones.

#### <a name="commands"></a>Comandos:

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network asg update](/cli/azure/network/asg#az-network-asg-update) |
| PowerShell | Ningún cmdlet de PowerShell |

### <a name="delete-an-application-security-group"></a>Eliminar un grupo de seguridad de aplicaciones

No puede eliminar un grupo de seguridad de aplicaciones si contiene alguna interfaz de red. Para quitar todas las interfaces de red del grupo de seguridad de aplicaciones, cambie la configuración de la interfaz de red o elimine las interfaces de red. Para obtener más información, consulte [Adición o eliminación de grupos de seguridad de aplicaciones](virtual-network-network-interface.md#add-to-or-remove-from-application-security-groups) o [Eliminación de una interfaz de red](virtual-network-network-interface.md#delete-a-network-interface).

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar sus grupos de seguridad de aplicaciones. Busque y seleccione **Grupos de seguridad de aplicaciones**.

2. Seleccione el nombre del grupo de seguridad de aplicaciones que quiere eliminar.

3. Haga clic en **Eliminar** y después en **Sí** para eliminar el grupo de seguridad de aplicaciones.

#### <a name="commands"></a>Comandos:

| Herramienta | Get-Help |
| ---- | ------- |
| Azure CLI | [az network asg delete](/cli/azure/network/asg#az-network-asg-delete) |
| PowerShell | [Remove-AzApplicationSecurityGroup](/powershell/module/az.network/remove-azapplicationsecuritygroup) |

## <a name="permissions"></a>Permisos

Para realizar tareas en grupos de seguridad de red, reglas de seguridad y grupos de seguridad de aplicaciones, la cuenta debe estar asignada al rol de [colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o a un [rol personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignados los permisos adecuados que se indican en las tablas siguientes:

### <a name="network-security-group"></a>Grupo de seguridad de red

| Acción                                                        |   Nombre                                                                |
|-------------------------------------------------------------- |   -------------------------------------------                         |
| Microsoft.Network/networkSecurityGroups/read                  |   Obtener grupo de seguridad de red                                          |
| Microsoft.Network/networkSecurityGroups/write                 |   Crear o actualizar grupo de seguridad de red                             |
| Microsoft.Network/networkSecurityGroups/delete                |   Eliminar grupo de seguridad de red                                       |
| Microsoft.Network/networkSecurityGroups/join/action           |   Asociar un grupo de seguridad de red a una subred o a una interfaz de red 

### <a name="network-security-group-rule"></a>Regla de grupo de seguridad de red

| Acción                                                        |   Nombre                                                                |
|-------------------------------------------------------------- |   -------------------------------------------                         |
| Microsoft.Network/networkSecurityGroups/securityRules/read            |   Obtener regla                                                            |
| Microsoft.Network/networkSecurityGroups/securityRules/write           |   Crear o actualizar regla                                               |
| Microsoft.Network/networkSecurityGroups/securityRules/delete          |   Eliminar regla                                                         |

### <a name="application-security-group"></a>Grupo de seguridad de aplicaciones

| Acción                                                                     | Nombre                                                     |
| --------------------------------------------------------------             | -------------------------------------------              |
| Microsoft.Network/applicationSecurityGroups/joinIpConfiguration/action     | Unir una configuración IP a un grupo de seguridad de aplicaciones|
| Microsoft.Network/applicationSecurityGroups/joinNetworkSecurityRule/action | Unir una regla de seguridad a un grupo de seguridad de aplicaciones    |
| Microsoft.Network/applicationSecurityGroups/read                           | Obtener un grupo de seguridad de aplicaciones                        |
| Microsoft.Network/applicationSecurityGroups/write                          | Crear o actualizar un grupo de seguridad de aplicaciones           |
| Microsoft.Network/applicationSecurityGroups/delete                         | Eliminar un grupo de seguridad de aplicaciones                     |

## <a name="next-steps"></a>Pasos siguientes

- Crear una red o un grupo de seguridad de aplicaciones con scripts de ejemplo de [PowerShell](powershell-samples.md) o de la [CLI de Azure](cli-samples.md), o bien con [plantillas de Azure Resource Manager](template-samples.md)
- Crear y asignar [definiciones de Azure Policy](policy-samples.md) para redes virtuales
