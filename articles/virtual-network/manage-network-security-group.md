---
title: Creación, modificación o eliminación de un grupo de seguridad de red de Azure
titlesuffix: Azure Virtual Network
description: Obtenga información sobre crear, modificar o eliminar un grupo de seguridad de red.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/05/2018
ms.author: kumud
ms.openlocfilehash: f1353165954021cd949d6e46357d10514ee26b3c
ms.sourcegitcommit: 179918af242d52664d3274370c6fdaec6c783eb6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65560944"
---
# <a name="create-change-or-delete-a-network-security-group"></a>Crear, modificar o eliminar un grupo de seguridad de red

Las reglas de seguridad de grupos de seguridad de red permiten filtrar el tipo de tráfico de red que puede fluir dentro y fuera de las interfaces de red y las subredes de redes virtuales. Si no está familiarizado con los grupos de seguridad de red, consulte la [introducción a la seguridad de red](security-overview.md) para obtener más información sobre estos y complete el tutorial [Filtrado del tráfico de red](tutorial-filter-network-traffic.md) para adquirir experiencia en el tema.

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Complete las tareas siguientes antes de seguir los pasos de las secciones de este artículo:

- Si todavía no tiene una cuenta de Azure, regístrese para obtener una [cuenta de evaluación gratuita](https://azure.microsoft.com/free).
- Si usa el portal, abra https://portal.azure.com e inicie sesión con la cuenta de Azure.
- Si usa comandos de PowerShell para completar las tareas de este artículo, ejecute los comandos que se encuentran en [Azure Cloud Shell](https://shell.azure.com/powershell) o ejecute PowerShell en el equipo. Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. Para realizar este tutorial, se necesita la versión 1.0.0 del módulo de Azure PowerShell u otra posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.
- Si usa la interfaz de la línea de comandos (CLI) de Azure para completar las tareas de este artículo, ejecute los comandos que se encuentran en [Azure Cloud Shell](https://shell.azure.com/bash) o ejecute la CLI en el equipo. Para realizar este tutorial es necesaria la versión 2.0.28 de la CLI de Azure o una versión posterior. Ejecute `az --version` para buscar la versión instalada. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Si ejecuta de forma local la CLI de Azure, también debe ejecutar `az login` para crear una conexión con Azure.

La cuenta en la que inicia sesión o con la que se conecta a Azure debe tener asignado el rol de [colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o un [rol personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones apropiadas en [Permisos](#permissions).

## <a name="work-with-network-security-groups"></a>Trabajar con grupos de seguridad de red

Los grupos de seguridad de red se pueden crear, [ver todos](#view-all-network-security-groups), [ver sus detalles](#view-details-of-a-network-security-group), [cambiar](#change-a-network-security-group) y [eliminar](#delete-a-network-security-group). También puede [asociar o desasociar](#associate-or-dissociate-a-network-security-group-to-or-from-a-subnet-or-network-interface) un grupo de seguridad de red de una interfaz de red o subred.

### <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

Existe un límite para la cantidad de grupos de seguridad de red que puede crear por suscripción y ubicación de Azure. Para más información, consulte el artículo acerca de los [límites de Azure](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

1. En la esquina superior izquierda del portal, seleccione **+ Crear un recurso**.
2. Seleccione **Redes** y, a continuación, **grupo de seguridad de red**.
3. Escriba un **nombre** para el grupo de seguridad de red, seleccione la **suscripción**, cree un **grupo de recursos** nuevo o seleccione uno existe, seleccione una **ubicación** y, luego, elija **Crear**.

**Comandos**

- CLI de Azure: [az network nsg create](/cli/azure/network/nsg#az-network-nsg-create)
- PowerShell: [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup)

### <a name="view-all-network-security-groups"></a>Ver todos los grupos de seguridad de red

En el cuadro de búsqueda que aparece en la parte superior del portal, escriba *grupos de seguridad de red*. Cuando aparezca **grupos de seguridad de red** en los resultados de la búsqueda, selecciónelo. Se mostrará una lista de los grupos de seguridad de red que existen en la suscripción.

**Comandos**

- CLI de Azure: [az network nsg list](/cli/azure/network/nsg#az-network-nsg-list)
- PowerShell: [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup)

### <a name="view-details-of-a-network-security-group"></a>Ver detalles de un grupo de seguridad de red

1. En el cuadro de búsqueda que aparece en la parte superior del portal, escriba *grupos de seguridad de red*. Cuando aparezca **grupos de seguridad de red** en los resultados de la búsqueda, selecciónelo.
2. En la lista, seleccione el grupo de seguridad de red cuyos detalles quiere ver. En **CONFIGURACIÓN** puede ver las **reglas de seguridad de entrada** y las **reglas de seguridad de salida**, las **interfaces de red** y las **subredes** a las que está asociado el grupo de seguridad de red. También puede habilitar o deshabilitar los **registros de diagnóstico** y ver las **reglas de seguridad efectivas**. Para más información, consulte los temas sobre [registros de diagnóstico](virtual-network-nsg-manage-log.md) y [Visualización de reglas de seguridad efectivas](diagnose-network-traffic-filter-problem.md).
3. Para más información sobre la configuración común de Azure que se muestra, consulte los artículos siguientes:
    *   [Registro de actividad](../azure-monitor/platform/activity-logs-overview.md)
    *   [Control de acceso (IAM)](../role-based-access-control/overview.md)
    *   [Etiquetas](../azure-resource-manager/resource-group-using-tags.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
    *   [Bloqueos](../azure-resource-manager/resource-group-lock-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
    *   [Script de Automation](../azure-resource-manager/manage-resource-groups-portal.md#export-resource-groups-to-templates)

**Comandos**

- CLI de Azure: [az network nsg show](/cli/azure/network/nsg#az-network-nsg-show)
- PowerShell: [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup)

### <a name="change-a-network-security-group"></a>Cambiar un grupo de seguridad de red

1. En el cuadro de búsqueda que aparece en la parte superior del portal, escriba *grupos de seguridad de red*. Cuando aparezca **grupos de seguridad de red** en los resultados de la búsqueda, selecciónelo.
2. Seleccione el grupo de seguridad de red que quiere modificar. Los cambios más comunes son [agregar](#create-a-security-rule) o [quitar](#delete-a-security-rule) de reglas de seguridad y [asociar o desasociar un grupo de seguridad de red a o de una subred o una interfaz de red](#associate-or-dissociate-a-network-security-group-to-or-from-a-subnet-or-network-interface).

**Comandos**

- CLI de Azure: [az network nsg update](/cli/azure/network/nsg#az-network-nsg-update)
- PowerShell: [Set-AzNetworkSecurityGroup](/powershell/module/az.network/set-aznetworksecuritygroup)

### <a name="associate-or-dissociate-a-network-security-group-to-or-from-a-subnet-or-network-interface"></a>Asociar o desasociar un grupo de seguridad de red a o de una subred o una interfaz de red

Para asociar un grupo de seguridad de red a una interfaz de red o desasociarlo de esta, consulte [Asociar un grupo de seguridad de red o una interfaz de red o desasociarlo de esta](virtual-network-network-interface.md#associate-or-dissociate-a-network-security-group). Para asociar un grupo de seguridad de red a una subred o desasociarlo de esta, consulte [Modificación de la configuración de subred](virtual-network-manage-subnet.md#change-subnet-settings).

### <a name="delete-a-network-security-group"></a>Eliminar un grupo de seguridad de red

Si un grupo de seguridad de red está asociado a subredes o interfaces de red cualesquiera, no se puede eliminar. Desasocie un grupo de seguridad de red de todas las subredes e interfaces de red antes de intentar eliminarlo.

1. En el cuadro de búsqueda que aparece en la parte superior del portal, escriba *grupos de seguridad de red*. Cuando aparezca **grupos de seguridad de red** en los resultados de la búsqueda, selecciónelo.
2. Seleccione el grupo de seguridad de red que quiere eliminar de la lista.
3. Seleccione **Eliminar** y, luego, seleccione **Sí**.

**Comandos**

- CLI de Azure: [az network nsg delete](/cli/azure/network/nsg#az-network-nsg-delete)
- PowerShell: [Remove-AzNetworkSecurityGroup](/powershell/module/az.network/remove-aznetworksecuritygroup)

## <a name="work-with-security-rules"></a>Trabajar con reglas de seguridad

Un grupo de seguridad de red puede contener una regla de seguridad, varias o ninguna. Las reglas de seguridad se pueden crear, [ver todas](#view-all-security-rules), [ver sus detalles](#view-details-of-a-security-rule), [cambiar](#change-a-security-rule) y [eliminar](#delete-a-security-rule).

### <a name="create-a-security-rule"></a>Creación de una regla de seguridad

Existe un límite para la cantidad de reglas por grupo de seguridad de red que se pueden crear por suscripción y ubicación de Azure. Para más información, consulte el artículo acerca de los [límites de Azure](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

1. En el cuadro de búsqueda que aparece en la parte superior del portal, escriba *grupos de seguridad de red*. Cuando aparezca **grupos de seguridad de red** en los resultados de la búsqueda, selecciónelo.
2. En la lista, seleccione el grupo de seguridad de red al que quiere agregar una regla de seguridad.
3. En **CONFIGURACIÓN**, seleccione **Reglas de seguridad de entrada**. Se muestran varias reglas existentes. Es posible que algunas de las reglas no las haya agregado usted. Cuando se crea un grupo de seguridad de red, se crean en este varias reglas de seguridad de forma predeterminada. Para más información, consulte las [reglas de seguridad predeterminadas](security-overview.md#default-security-rules).  Las reglas de seguridad predeterminadas no se pueden eliminar, pero pueden reemplazarse por reglas de prioridad más alta.
4. <a name = "security-rule-settings"></a>Seleccione **+ Agregar**.  Seleccione o agregue valores para las siguientes opciones y, continuación, seleccione **Aceptar**:
    
    |Configuración  |Valor  |Detalles  |
    |---------|---------|---------|
    |Origen     | Seleccione **Cualquiera**, **Grupo de seguridad de aplicaciones**, **Direcciones IP** o **Etiqueta de servicio** para las reglas de seguridad de entrada. Si va a crear una regla de seguridad de salida, las opciones son las mismas que las indicadas para **Destino**.       | Si selecciona **Grupo de seguridad de aplicaciones**, seleccione después uno o varios grupos de seguridad de aplicaciones existentes en la misma región que la interfaz de red. Obtenga información sobre cómo [crear un grupo de seguridad de aplicaciones](#create-an-application-security-group). Si selecciona **Grupo de seguridad de aplicaciones** para **Origen** y **Destino**, las interfaces de red de ambos grupos de seguridad de aplicaciones deben estar en la misma red virtual. Si selecciona **Direcciones IP**, especifique después **Intervalos de direcciones IP de origen y CIDR**. Puede especificar un valor único o una lista de varios valores separados por comas. Un ejemplo de varios valores es 10.0.0.0/16, 192.188.1.1. Se aplican límites al número de valores que puede especificar. Para más información, consulte los [límites de Azure](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits). Si selecciona **Etiqueta de servicio**, seleccione después una etiqueta de servicio. Una etiqueta de servicio es un identificador predefinido para una categoría de direcciones IP. Para más información sobre las etiquetas de servicio disponibles y qué representa cada etiqueta, consulte [Etiquetas de servicio](security-overview.md#service-tags). Si la dirección IP que especifica está asignada a una máquina virtual de Azure, asegúrese de que especifica la dirección IP privada, no la dirección IP pública asignada a la máquina virtual. Las reglas de seguridad se procesan después de que Azure traduce la dirección IP pública a una dirección IP privada para las reglas de seguridad de entrada, y antes de que Azure traduzca una dirección IP privada a una dirección IP pública para las reglas de salida. Para más información sobre las direcciones IP públicas y privadas de Azure, consulte [Tipos de direcciones IP](virtual-network-ip-addresses-overview-arm.md).        |
    |Intervalos de puertos de origen     | Especifique un único puerto (por ejemplo, 80), un intervalo de puertos (como 1024-65535) o una lista de puertos únicos y/o intervalos de puertos separados por comas (como 80, 1024-65535). Escriba un asterisco para permitir el tráfico en cualquier puerto. | Los puertos e intervalos especifican qué tráfico de puertos permitirá o denegará la regla. Se aplican límites al número de puertos que puede especificar. Para más información, consulte los [límites de Azure](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).  |
    |Destino     | Seleccione **cualquier**, **grupo de seguridad de la aplicación**, **direcciones IP**, o **red Virtual** para las reglas de seguridad de salida. Si va a crear una regla de seguridad de entrada, las opciones son las mismas que para **origen**.        | Si selecciona **Grupo de seguridad de aplicaciones**, debe seleccionar después uno o varios grupos de seguridad de aplicaciones existentes en la misma región que la interfaz de red. Obtenga información sobre cómo [crear un grupo de seguridad de aplicaciones](#create-an-application-security-group). Si selecciona **Grupo de seguridad de aplicaciones**, seleccione después un grupo de seguridad de aplicaciones que exista en la misma región que la interfaz de red. Si selecciona **Direcciones IP**, especifique después **Intervalos de direcciones IP de destino y CIDR**. De manera similar a **Origen** e **Intervalos de direcciones IP de origen y CIDR**, puede especificar uno o varios intervalos o direcciones y se aplican límites a la cantidad que puede especificar. Si se selecciona **Red virtual**, que es una etiqueta de servicio, significa que se permite el tráfico a todas las direcciones IP dentro del espacio de direcciones de la red virtual. Si la dirección IP que especifica está asignada a una máquina virtual de Azure, asegúrese de que especifica la dirección IP privada, no la dirección IP pública asignada a la máquina virtual. Las reglas de seguridad se procesan después de que Azure traduce la dirección IP pública a una dirección IP privada para las reglas de seguridad de entrada, y antes de que Azure traduzca una dirección IP privada a una dirección IP pública para las reglas de salida. Para más información sobre las direcciones IP públicas y privadas de Azure, consulte [Tipos de direcciones IP](virtual-network-ip-addresses-overview-arm.md).        |
    |Intervalos de puertos de destino     | Especifique un valor único o una lista de varios valores separados por comas. | De manera similar a **Intervalos de puertos de origen**, puede especificar uno o varios intervalos o puertos y se aplican límites a la cantidad que puede especificar. |
    |Protocol     | Seleccione **Cualquiera**, **TCP** o **UDP**.        |         |
    |.     | Seleccione **Permitir** o **Denegar**.        |         |
    |Prioridad     | Escriba un valor de 100-4096 que sea único para todas las reglas de seguridad del grupo de seguridad de red. |Las reglas se procesan en el orden de prioridad. Cuanto menor sea el número, mayor será la prioridad. Se recomienda dejar un espacio entre los números de prioridad al crear reglas, como 100, 200, 300. Dejar espacios facilita la adición de reglas en el futuro que pueda necesitar por encima o debajo de las reglas existentes.         |
    |NOMBRE     | Nombre único para la regla en el grupo de seguridad de red.        |  Puede tener hasta 80 caracteres. Debe empezar con una letra o un número y acabar con una letra, un número o un guion bajo, y solo debe contener letras, números, guiones bajos, puntos o guiones.       |
    |DESCRIPCIÓN     | Descripción opcional.        |         |

**Comandos**

- CLI de Azure: [az network nsg rule create](/cli/azure/network/nsg/rule#az-network-nsg-rule-create)
- PowerShell: [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig)

### <a name="view-all-security-rules"></a>Ver todas las reglas de seguridad

Un grupo de seguridad de red contiene varias reglas de seguridad o ninguna. Para más detalles sobre la información que aparece al ver las rutas, consulte la [introducción a la seguridad de red](security-overview.md).

1. En el cuadro de búsqueda que aparece en la parte superior del portal, escriba *grupos de seguridad de red*. Cuando aparezca **grupos de seguridad de red** en los resultados de la búsqueda, selecciónelo.
2. En la lista, seleccione el grupo de seguridad de red cuyas reglas quiere ver.
3. En **CONFIGURACIÓN**, seleccione **Reglas de seguridad de entrada** o **Reglas de seguridad de salida**.

La lista contiene todas las reglas que ha creado y las [reglas de seguridad predeterminadas](security-overview.md#default-security-rules) del grupo de seguridad de red.

**Comandos**

- CLI de Azure: [az network nsg rule list](/cli/azure/network/nsg/rule#az-network-nsg-rule-list)
- PowerShell: [Get-AzNetworkSecurityRuleConfig](/powershell/module/az.network/get-aznetworksecurityruleconfig)

### <a name="view-details-of-a-security-rule"></a>Ver detalles de una regla de seguridad

1. En el cuadro de búsqueda que aparece en la parte superior del portal, escriba *grupos de seguridad de red*. Cuando aparezca **grupos de seguridad de red** en los resultados de la búsqueda, selecciónelo.
2. Seleccione el grupo de seguridad de red cuyos detalles de una regla de seguridad quiere ver.
3. En **CONFIGURACIÓN**, seleccione **Reglas de seguridad de entrada** o **Reglas de seguridad de salida**.
4. Seleccione la regla cuyos detalles quiere ver. Para obtener una explicación detallada de todas las opciones, consulte la [configuración de las reglas de seguridad](#security-rule-settings).

**Comandos**

- CLI de Azure: [az network nsg rule show](/cli/azure/network/nsg/rule#az-network-nsg-rule-show)
- PowerShell: [Get-AzNetworkSecurityRuleConfig](/powershell/module/az.network/get-aznetworksecurityruleconfig)

### <a name="change-a-security-rule"></a>Cambiar una regla de seguridad

1. Complete los pasos de [Ver detalles de una regla de seguridad](#view-details-of-a-security-rule).
2. Cambie la configuración según desee y, a continuación, seleccione **Guardar**. Para obtener una explicación detallada de todas las opciones, consulte la [configuración de las reglas de seguridad](#security-rule-settings).

**Comandos**

- CLI de Azure: [az network nsg rule update](/cli/azure/network/nsg/rule#az-network-nsg-rule-update)
- PowerShell: [Set-AzNetworkSecurityRuleConfig](/powershell/module/az.network/set-aznetworksecurityruleconfig)

### <a name="delete-a-security-rule"></a>Eliminar una regla de seguridad

1. Complete los pasos de [Ver detalles de una regla de seguridad](#view-details-of-a-security-rule).
2. Seleccione **Eliminar** y, luego, seleccione **Sí**.

**Comandos**

- CLI de Azure: [az network nsg rule delete](/cli/azure/network/nsg/rule#az-network-nsg-rule-delete)
- PowerShell: [Remove-AzNetworkSecurityRuleConfig](/powershell/module/az.network/remove-aznetworksecurityruleconfig)

## <a name="work-with-application-security-groups"></a>Trabajar con grupos de seguridad de aplicaciones

Un grupo de seguridad de aplicaciones contiene una interfaz de red, varias o ninguna. Para más información, consulte [grupos de seguridad de aplicaciones](security-overview.md#application-security-groups). Todas las interfaces de red de un grupo de seguridad de aplicaciones deben existir en la misma red virtual. Para obtener información sobre cómo agregar una interfaz de red a un grupo de seguridad de aplicaciones, consulte el tema sobre cómo [agregar una interfaz de red a un grupo de seguridad de aplicaciones](virtual-network-network-interface.md#add-to-or-remove-from-application-security-groups).

### <a name="create-an-application-security-group"></a>Crear un grupo de seguridad de aplicación

1. Seleccione **+ Crear un recurso** en la esquina superior izquierda de Azure Portal.
2. En el cuadro **Buscar en Marketplace**, escriba *Grupo de seguridad de aplicaciones*. Cuando **Grupo de seguridad de aplicaciones** aparezca en los resultados de la búsqueda, selecciónelo, vuelva a seleccionar **Grupo de seguridad de aplicaciones** en **Todo** y, después, haga clic en **Crear**.
3. Especifique o seleccione los siguientes datos y haga clic en **Crear**:

    | Configuración        | Valor                                                   |
    | ---            | ---                                                     |
    | NOMBRE           | El nombre debe ser único dentro de un grupo de recursos.        |
    | Subscription   | Seleccione su suscripción.                               |
    | Grupos de recursos | Seleccione un grupo de recursos existente, o bien cree uno. |
    | Ubicación       | Seleccionar una ubicación                                       |

**Comandos**

- CLI de Azure: [az network asg create](/cli/azure/network/asg#az-network-asg-create)
- PowerShell: [New-AzApplicationSecurityGroup](/powershell/module/az.network/new-azapplicationsecuritygroup)

### <a name="view-all-application-security-groups"></a>Ver todos los grupos de seguridad de aplicaciones

1. En la esquina superior izquierda de Azure Portal, seleccione **Todos los servicios**.
2. Escriba *grupos de seguridad de aplicaciones* en el cuadro **de filtro Todos los servicios** y, después, seleccione **Grupos de seguridad de aplicaciones** cuando aparezca en los resultados de la búsqueda.

**Comandos**

- CLI de Azure: [az network asg list](/cli/azure/network/asg#az-network-asg-list)
- PowerShell: [Get-AzApplicationSecurityGroup](/powershell/module/az.network/get-azapplicationsecuritygroup)

### <a name="view-details-of-a-specific-application-security-group"></a>Ver detalles de un grupo de seguridad de aplicaciones específico

1. En la esquina superior izquierda de Azure Portal, seleccione **Todos los servicios**.
2. Escriba *grupos de seguridad de aplicaciones* en el cuadro **de filtro Todos los servicios** y, después, seleccione **Grupos de seguridad de aplicaciones** cuando aparezca en los resultados de la búsqueda.
3. Seleccione el grupo de seguridad de aplicaciones del que quiera ver los detalles.

**Comandos**

- CLI de Azure: [az network asg show](/cli/azure/network/asg#az-network-asg-show)
- PowerShell: [Get-AzApplicationSecurityGroup](/powershell/module/az.network/get-azapplicationsecuritygroup)

### <a name="change-an-application-security-group"></a>Cambiar un grupo de seguridad de aplicaciones

1. En la esquina superior izquierda de Azure Portal, seleccione **Todos los servicios**.
2. Escriba *grupos de seguridad de aplicaciones* en el cuadro **de filtro Todos los servicios** y, después, seleccione **Grupos de seguridad de aplicaciones** cuando aparezca en los resultados de la búsqueda.
3. Seleccione el grupo de seguridad de aplicaciones para el que quiera cambiar la configuración. Puede agregar o quitar etiquetas, o bien asignar o quitar permisos para el grupo de seguridad de aplicaciones.

- CLI de Azure: [az network asg update](/cli/azure/network/asg#az-network-asg-update)
- PowerShell: Ningún cmdlet de PowerShell.

### <a name="delete-an-application-security-group"></a>Eliminar un grupo de seguridad de aplicaciones

No se puede eliminar un grupo de seguridad de aplicaciones si tiene alguna interfaz de red. Quite todas las interfaces de red del grupo de seguridad de aplicaciones. Para ello, cambie la configuración de la interfaz de red o elimine las interfaces de red. Para obtener más información, consulte los temas sobre cómo [agregar a o quitar una interfaz de red de grupos de seguridad de aplicaciones](virtual-network-network-interface.md#add-to-or-remove-from-application-security-groups) o [eliminar una interfaz de red](virtual-network-network-interface.md#delete-a-network-interface).

1. En la esquina superior izquierda de Azure Portal, seleccione **Todos los servicios**.
2. Escriba *grupos de seguridad de aplicaciones* en el cuadro **de filtro Todos los servicios** y, después, seleccione **Grupos de seguridad de aplicaciones** cuando aparezca en los resultados de la búsqueda.
3. Seleccione el grupo de seguridad de aplicaciones que quiera eliminar.
4. Haga clic en **Eliminar** y después en **Sí** para eliminar el grupo de seguridad de aplicaciones.

**Comandos**

- CLI de Azure: [az network asg delete](/cli/azure/network/asg#az-network-asg-delete)
- PowerShell: [Remove-AzApplicationSecurityGroup](/powershell/module/az.network/remove-azapplicationsecuritygroup)

## <a name="permissions"></a>Permisos

Para realizar tareas en grupos de seguridad de red, reglas de seguridad y grupos de seguridad de aplicaciones, la cuenta debe estar asignada al rol de [colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o a un rol [personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignados los permisos adecuados que se muestran en las tablas siguientes:

### <a name="network-security-group"></a>Grupo de seguridad de red

| .                                                        |   NOMBRE                                                                |
|-------------------------------------------------------------- |   -------------------------------------------                         |
| Microsoft.Network/networkSecurityGroups/read                  |   Obtener grupo de seguridad de red                                          |
| Microsoft.Network/networkSecurityGroups/write                 |   Crear o actualizar grupo de seguridad de red                             |
| Microsoft.Network/networkSecurityGroups/delete                |   Eliminar el grupo de seguridad de red                                       |
| Microsoft.Network/networkSecurityGroups/join/action           |   Asociar un grupo de seguridad de red a una subred o a una interfaz de red 


### <a name="network-security-group-rule"></a>Regla de grupo de seguridad de red

| .                                                        |   NOMBRE                                                                |
|-------------------------------------------------------------- |   -------------------------------------------                         |
| Microsoft.Network/networkSecurityGroups/rules/read            |   Obtener regla                                                            |
| Microsoft.Network/networkSecurityGroups/rules/write           |   Crear o actualizar regla                                               |
| Microsoft.Network/networkSecurityGroups/rules/delete          |   Eliminar regla                                                         |

### <a name="application-security-group"></a>Grupo de seguridad de aplicaciones

| .                                                                     | NOMBRE                                                     |
| --------------------------------------------------------------             | -------------------------------------------              |
| Microsoft.Network/applicationSecurityGroups/joinIpConfiguration/action     | Unir una configuración IP a un grupo de seguridad de aplicaciones|
| Microsoft.Network/applicationSecurityGroups/joinNetworkSecurityRule/action | Unir una regla de seguridad a un grupo de seguridad de aplicaciones    |
| Microsoft.Network/applicationSecurityGroups/read                           | Obtener un grupo de seguridad de aplicaciones                        |
| Microsoft.Network/applicationSecurityGroups/write                          | Crear o actualizar un grupo de seguridad de aplicaciones           |
| Microsoft.Network/applicationSecurityGroups/delete                         | Eliminar un grupo de seguridad de aplicaciones                     |

## <a name="next-steps"></a>Pasos siguientes

- Crear una red o un grupo de seguridad de aplicaciones con scripts de ejemplo de [PowerShell](powershell-samples.md) o de la [CLI de Azure](cli-samples.md), o bien con [plantillas de Azure Resource Manager](template-samples.md)
- Crear y aplicar una [directiva de Azure](policy-samples.md) para redes virtuales
