---
title: Registro de recursos de diagnóstico de un grupo de seguridad de red
titlesuffix: Azure Virtual Network
description: Aprenda a habilitar los registros de recursos de diagnóstico del contador de reglas y de eventos de un grupo de seguridad de red de Azure.
services: virtual-network
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 06/04/2018
ms.author: kumud
ms.openlocfilehash: 221f7577b3181b1535ab9f544073dac4d031fe66
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89319452"
---
# <a name="resource-logging-for-a-network-security-group"></a>Registro de recursos de un grupo de seguridad de red

Los grupos de seguridad de red (NSG) incluyen reglas que permiten o deniegan el tráfico a una subred de red virtual, a una interfaz de red, o a ambas. 

Al habilitar el registro para un grupo de seguridad de red, puede recopilar los siguientes tipos de información del registro de recursos:

* **Evento:** se registran las entradas en las que se aplican reglas de NSG a máquinas virtuales en función de la dirección MAC.
* **Contador de regla:** contiene entradas para el número de veces que se aplica cada regla NSG para denegar o permitir el tráfico. El estado de estas reglas se recopila cada 300 segundos.

Los registros de diagnóstico solo están disponibles para los grupos de seguridad de red implementados con el modelo de implementación de Azure Resource Manager. No se puede habilitar el registro de diagnóstico para los grupos de seguridad de red implementados con el modelo de implementación clásica. Para entender mejor los dos modelos, consulte [Descripción de los modelos de implementación de Azure](../resource-manager-deployment-model.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

El registro de recursos se habilita por separado para *cada* grupo de seguridad de red para el que desee recopilar datos. Si lo que le interesan son los registros de actividades (operaciones), consulte el [registro de actividades](../azure-monitor/platform/platform-logs-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) de Azure.

## <a name="enable-logging"></a>Habilitación del registro

Para habilitar el registro de recursos se pueden usar [Azure Portal](#azure-portal), [PowerShell](#powershell) o la [CLI de Azure](#azure-cli).

### <a name="azure-portal"></a>Azure Portal

1. Inicie sesión en el [portal](https://portal.azure.com).
2. Seleccione **Todos los servicios** y escriba *grupos de seguridad de red*. Cuando aparezca **Grupos de seguridad de red** en los resultados de la búsqueda, selecciónelo.
3. Seleccione el NSG para el que desea habilitar el registro.
4. En **SUPERVISIÓN**, seleccione **Registros de diagnósticos**y, después, seleccione **Turn on diagnostics** (Activar diagnóstico), como se muestra en la siguiente imagen:

   ![Activación del diagnóstico](./media/virtual-network-nsg-manage-log/turn-on-diagnostics.png)

5. En **Configuración de diagnóstico**, especifique o seleccione los siguientes datos y, después, seleccione **Guardar**:

    | Configuración                                                                                     | Value                                                          |
    | ---------                                                                                   |---------                                                       |
    | Nombre                                                                                        | El nombre que prefiera.  Por ejemplo: *myNsgDiagnostics*      |
    | **Archivar en una cuenta de almacenamiento**, **Transmitir en secuencias a un centro de eventos** y **Enviar a Log Analytics** | Puede seleccionar tantos destinos como quiera. Para más información acerca de cada uno de ellos, consulte [Destinos de registro](#log-destinations).                                                                                                                                           |
    | REGISTRO                                                                                         | Seleccione una de las categorías de registro, o incluso ambas. Para más información acerca de los datos que se registran en cada categoría, consulte [Categorías de registro](#log-categories).                                                                                                                                             |
6. Vea y analice los registros. Para más información, consulte [Visualización y análisis de los registros](#view-and-analyze-logs).

### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Puede ejecutar los comandos siguientes en [Azure Cloud Shell](https://shell.azure.com/powershell), o mediante la ejecución de PowerShell en el equipo. Azure Cloud Shell es un shell interactivo gratuito. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. Si ejecuta PowerShell desde el equipo, necesita el módulo Azure PowerShell versión 1.0.0 o posterior. Ejecute `Get-Module -ListAvailable Az` en el equipo para encontrar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si ejecuta PowerShell localmente, también debe ejecutar `Connect-AzAccount` para iniciar sesión en Azure con una cuenta que tenga los [permisos necesarios](virtual-network-network-interface.md#permissions).

Para habilitar el registro de recursos, se necesita el identificador de un grupo de seguridad de red existente. Si no tiene ninguno, puede crearlo con [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup).

Recupere el grupo de seguridad de red para el que desea habilitar el registro de recursos con [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup). Por ejemplo, para recuperar un grupo de seguridad de red denominado *myNsg* que existe en un grupo de recursos denominado *myResourceGroup*, escriba el siguiente comando:

```azurepowershell-interactive
$Nsg=Get-AzNetworkSecurityGroup `
  -Name myNsg `
  -ResourceGroupName myResourceGroup
```

Puede escribir registros de recursos para tres tipos de destino. Para más información, consulte [Destinos del registro](#log-destinations). En este artículo, los registros se envían al destino *Log Analytics*, pero no es más que un ejemplo. Recupere un área de trabajo de Log Analytics existente con [Get-AzOperationalInsightsWorkspace](/powershell/module/az.operationalinsights/get-azoperationalinsightsworkspace). Por ejemplo, para recuperar un área de trabajo denominado *myWorkspace* en un grupo de recursos denominado *myWorkspaces*, escriba el siguiente comando:

```azurepowershell-interactive
$Oms=Get-AzOperationalInsightsWorkspace `
  -ResourceGroupName myWorkspaces `
  -Name myWorkspace
```

Si no tiene un área de trabajo, puede crearla con [New-AzOperationalInsightsWorkspace](/powershell/module/az.operationalinsights/new-azoperationalinsightsworkspace).

Existen dos categorías de registro para las que se pueden habilitar registros. Para más información, consulte [Categorías de registro](#log-categories). Habilite el registro de recursos para el grupo de seguridad de red con [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting). En el ejemplo siguiente se registran en el área de trabajo de un grupo de seguridad de red los datos de la categoría tanto del evento como del contador, para lo que se usan los identificadores del grupo de seguridad de red como del área de trabajo que recuperó antes:

```azurepowershell-interactive
Set-AzDiagnosticSetting `
  -ResourceId $Nsg.Id `
  -WorkspaceId $Oms.ResourceId `
  -Enabled $true
```

Si solo desea registrar los datos en una de las dos categorías, en lugar de en ambas, agregue la opción `-Categories` al comando anterior, seguida de *NetworkSecurityGroupEvent* o *NetworkSecurityGroupRuleCounter*. Si desea realizar el registro en un [destino](#log-destinations) que no sea un área de trabajo de Log Analytics, utilice los parámetros adecuados para una [cuenta de almacenamiento](../azure-monitor/platform/archive-diagnostic-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o un [centro de eventos](../azure-monitor/platform/resource-logs-stream-event-hubs.md?toc=%2fazure%2fvirtual-network%2ftoc.json) de Azure.

Vea y analice los registros. Para más información, consulte [Visualización y análisis de los registros](#view-and-analyze-logs).

### <a name="azure-cli"></a>Azure CLI

Puede ejecutar los comandos siguientes en [Azure Cloud Shell](https://shell.azure.com/bash), o mediante la ejecución de la CLI de Azure en el equipo. Azure Cloud Shell es un shell interactivo gratuito. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. Si ejecuta la CLI desde el equipo, necesita la versión 2.0.38 o posterior. Ejecute `az --version` en el equipo para encontrar la versión instalada. Si debe actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest). Si ejecuta la CLI localmente, también debe ejecutar `az login` para iniciar sesión en Azure con una cuenta que tenga los [permisos necesarios](virtual-network-network-interface.md#permissions).

Para habilitar el registro de recursos se necesita el identificador de un grupo de seguridad de red existente. Si no tiene ninguno, puede crear uno con [az network nsg create](/cli/azure/network/nsg#az-network-nsg-create).

Recupere el grupo de seguridad de red para el que desea habilitar el registro de recursos con [az network nsg show](/cli/azure/network/nsg#az-network-nsg-show). Por ejemplo, para recuperar un grupo de seguridad de red denominado *myNsg* que existe en un grupo de recursos denominado *myResourceGroup*, escriba el siguiente comando:

```azurecli-interactive
nsgId=$(az network nsg show \
  --name myNsg \
  --resource-group myResourceGroup \
  --query id \
  --output tsv)
```

Puede escribir registros de recursos para tres tipos de destino. Para más información, consulte [Destinos del registro](#log-destinations). En este artículo, los registros se envían al destino *Log Analytics*, pero no es más que un ejemplo. Para más información, consulte [Categorías de registro](#log-categories).

Habilite el registro de recursos para el grupo de seguridad de red con [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings#az-monitor-diagnostic-settings-create). En el ejemplo siguiente se registran los datos de categoría de eventos y de contadores en un área de trabajo denominada *myWorkspace*, que se encuentra en un grupo de recursos denominado *myWorkspaces*, y el identificador del grupo de seguridad de red que recuperó anteriormente:

```azurecli-interactive
az monitor diagnostic-settings create \
  --name myNsgDiagnostics \
  --resource $nsgId \
  --logs '[ { "category": "NetworkSecurityGroupEvent", "enabled": true, "retentionPolicy": { "days": 30, "enabled": true } }, { "category": "NetworkSecurityGroupRuleCounter", "enabled": true, "retentionPolicy": { "days": 30, "enabled": true } } ]' \
  --workspace myWorkspace \
  --resource-group myWorkspaces
```

Si no tiene ningún área de trabajo existente, puede crear una mediante [Azure Portal](../azure-monitor/learn/quick-create-workspace.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o [PowerShell](/powershell/module/az.operationalinsights/new-azoperationalinsightsworkspace). Existen dos categorías de registro para las que se pueden habilitar registros.

Si solo desea registrar los datos de una categoría o la otra, quite la categoría para la que no desea registrar los datos en el comando anterior. Si desea realizar el registro en un [destino](#log-destinations) que no sea un área de trabajo de Log Analytics, utilice los parámetros adecuados para una [cuenta de almacenamiento](../azure-monitor/platform/archive-diagnostic-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o un [centro de eventos](../azure-monitor/platform/resource-logs-stream-event-hubs.md?toc=%2fazure%2fvirtual-network%2ftoc.json) de Azure.

Vea y analice los registros. Para más información, consulte [Visualización y análisis de los registros](#view-and-analyze-logs).

## <a name="log-destinations"></a>Destinos de registro

Los datos de diagnóstico se pueden:
- [Escribir en una cuenta de Azure Storage](../azure-monitor/platform/archive-diagnostic-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json), para auditarlos o para comprobarlos manualmente. El tiempo de retención (en días) se puede especificar en la configuración del diagnóstico de recursos.
- [Transmitir en secuencias a un centro de eventos](../azure-monitor/platform/resource-logs-stream-event-hubs.md?toc=%2fazure%2fvirtual-network%2ftoc.json) para que los ingiera un servicio de terceros o una solución de análisis personalizada, como PowerBI.
- [Escritura en los registros de Azure Monitor](../azure-monitor/platform/resource-logs-collect-storage.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="log-categories"></a>Categorías de registro

Se escriben datos con formato JSON para las siguientes categorías de registro:

### <a name="event"></a>Evento

El registro de eventos contiene información acerca de las reglas de NSG que se aplican a las máquinas virtuales en función de su dirección MAC. Los siguientes datos se registran para cada evento. En el ejemplo siguiente, los datos se registran para una máquina virtual con la dirección IP 192.168.1.4 y una dirección MAC de 00-0D-3A-92-6A-7C:

```json
{
    "time": "[DATE-TIME]",
    "systemId": "[ID]",
    "category": "NetworkSecurityGroupEvent",
    "resourceId": "/SUBSCRIPTIONS/[SUBSCRIPTION-ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG-NAME]",
    "operationName": "NetworkSecurityGroupEvents",
    "properties": {
        "vnetResourceGuid":"[ID]",
        "subnetPrefix":"192.168.1.0/24",
        "macAddress":"00-0D-3A-92-6A-7C",
        "primaryIPv4Address":"192.168.1.4",
        "ruleName":"[SECURITY-RULE-NAME]",
        "direction":"[DIRECTION-SPECIFIED-IN-RULE]",
        "priority":"[PRIORITY-SPECIFIED-IN-RULE]",
        "type":"[ALLOW-OR-DENY-AS-SPECIFIED-IN-RULE]",
        "conditions":{
            "protocols":"[PROTOCOLS-SPECIFIED-IN-RULE]",
            "destinationPortRange":"[PORT-RANGE-SPECIFIED-IN-RULE]",
            "sourcePortRange":"[PORT-RANGE-SPECIFIED-IN-RULE]",
            "sourceIP":"[SOURCE-IP-OR-RANGE-SPECIFIED-IN-RULE]",
            "destinationIP":"[DESTINATION-IP-OR-RANGE-SPECIFIED-IN-RULE]"
            }
        }
}
```

### <a name="rule-counter"></a>Contador de reglas

El contador de reglas contiene información acerca de todas las reglas que se aplican a los recursos. Los datos de ejemplo siguientes se registran cada vez que se aplica una regla. En el ejemplo siguiente, los datos se registran para una máquina virtual con la dirección IP 192.168.1.4 y una dirección MAC de 00-0D-3A-92-6A-7C:

```json
{
    "time": "[DATE-TIME]",
    "systemId": "[ID]",
    "category": "NetworkSecurityGroupRuleCounter",
    "resourceId": "/SUBSCRIPTIONS/[SUBSCRIPTION ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG-NAME]",
    "operationName": "NetworkSecurityGroupCounters",
    "properties": {
        "vnetResourceGuid":"[ID]",
        "subnetPrefix":"192.168.1.0/24",
        "macAddress":"00-0D-3A-92-6A-7C",
        "primaryIPv4Address":"192.168.1.4",
        "ruleName":"[SECURITY-RULE-NAME]",
        "direction":"[DIRECTION-SPECIFIED-IN-RULE]",
        "type":"[ALLOW-OR-DENY-AS-SPECIFIED-IN-RULE]",
        "matchedConnections":125
        }
}
```

> [!NOTE]
> La dirección IP de origen de la comunicación no se ha registrado. Sin embargo, puede habilitar el [registro de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md) para un NSG, lo que registra toda la información del contador de reglas, así como la dirección IP de origen que inició la comunicación. Los datos del registro de flujos de NSG se escriben en una cuenta de Azure Storage. Puede analizar los datos con la funcionalidad de [análisis del tráfico](../network-watcher/traffic-analytics.md) Azure Network Watcher.

## <a name="view-and-analyze-logs"></a>Visualización y análisis de los registros

Para aprender a ver los datos del registro de recursos, consulte [Introducción a los registros de plataforma Azure](../azure-monitor/platform/platform-logs-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Si envía datos de diagnóstico a:
- **Registros de Azure Monitor**: puede usar la solución de [análisis de grupos de seguridad de red](../azure-monitor/insights/azure-networking-analytics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-network-security-group-analytics-solution-in-azure-monitor
) para obtener una información más detallada. La solución ofrece visualizaciones para de las reglas de NSG que permiten o deniegan el tráfico, por dirección MAC, de la interfaz de red de una máquina virtual.
- **Cuenta de Azure Storage**: los datos se escriben en el archivo PT1H.json. Puede encontrar el:
  - Registro de eventos en la siguiente ruta de acceso: `insights-logs-networksecuritygroupevent/resourceId=/SUBSCRIPTIONS/[ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME-FOR-NSG]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG NAME]/y=[YEAR]/m=[MONTH/d=[DAY]/h=[HOUR]/m=[MINUTE]`
  - Registro del contador de reglas en la siguiente ruta de acceso: `insights-logs-networksecuritygrouprulecounter/resourceId=/SUBSCRIPTIONS/[ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME-FOR-NSG]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG NAME]/y=[YEAR]/m=[MONTH/d=[DAY]/h=[HOUR]/m=[MINUTE]`

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre el [registro de actividad](../azure-monitor/platform/platform-logs-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). El registro de actividades está habilitado de forma predeterminada para los grupo de seguridad de red creados a través de cualquier modelo de implementación de Azure. Para determinar qué operaciones se completaron en los NSG del registro de actividad, busque las entradas que contengan los siguientes tipos de recursos:
  - Microsoft.ClassicNetwork/networkSecurityGroups
  - Microsoft.ClassicNetwork/networkSecurityGroups/securityRules
  - Microsoft.Network/networkSecurityGroups
  - Microsoft.Network/networkSecurityGroups/securityRules
- Para aprender a registrar información de diagnóstico e incluir la dirección IP de origen para cada flujo, consulte [Registro de flujos de NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
