---
title: Escalado de un clúster de Service Fabric en Azure
description: En este tutorial, aprenderá a escalar y reducir horizontalmente un clúster de Service Fabric en Azure y a limpiar los recursos sobrantes.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc
ms.openlocfilehash: d9699103f5e13301cce408d2e54f0e15780e0a35
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88716901"
---
# <a name="tutorial-scale-a-service-fabric-cluster-in-azure"></a>Tutorial: Escalado de un clúster de Service Fabric en Azure

Este tutorial es la tercera parte de una serie y en él se describe cómo reducir y escalar horizontalmente un clúster existente. Cuando haya terminado, habrá aprendido cómo escalar el clúster y cómo limpiar los recursos restantes.  Para más información sobre cómo escalar un clúster que se ejecuta en Azure, lea [Escalado de clústeres de Service Fabric](service-fabric-cluster-scaling.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Agregar y quitar nodos (escalado horizontal y reducción horizontal)
> * Agregar y quitar tipos de nodos (escalado horizontal y reducción horizontal)
> * Aumentar los recursos de nodo (escalabilidad vertical)

En esta serie de tutoriales, se aprende a:
> [!div class="checklist"]
> * Creación de un [clúster de Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) en Azure mediante una plantilla
> * [Supervisión de un clúster](service-fabric-tutorial-monitor-cluster.md)
> * Escalar o reducir un clúster horizontalmente
> * [Actualización del entorno en tiempo de ejecución de un clúster](service-fabric-tutorial-upgrade-cluster.md)
> * [Eliminación de un clúster](service-fabric-tutorial-delete-cluster.md)


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar este tutorial:

* Si no tiene ninguna suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* Instale [Azure Powershell](/powershell/azure/install-az-ps) o la [CLI de Azure](/cli/azure/install-azure-cli).
* Creación de un [clúster de Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) seguro en Azure

## <a name="important-considerations-and-guidelines"></a>Consideraciones e instrucciones importantes

Las cargas de trabajo de la aplicación cambian con el tiempo, ¿los servicios existentes necesitan más o menos recursos más ?  [Agregue o quite nodos](#add-nodes-to-or-remove-nodes-from-a-node-type) desde un tipo de nodo para aumentar o disminuir los recursos de clúster.

¿Necesita agregar más de 100 nodos al clúster?  Un único conjunto de escalado o tipo de nodo de Service Fabric no puede contener más de 100 nodos o máquinas virtuales.  Para escalar un clúster más allá de 100 nodos, [agregue tipos de nodo adicionales](#add-nodes-to-or-remove-nodes-from-a-node-type).

¿Tiene la aplicación varios servicios y alguno de ellos ha de ser público o accesible desde Internet?  Las aplicaciones típicas contienen un servicio front-end de puerta de enlace que recibe entradas de un cliente, así como uno o varios servicios back-end que se comunican con los servicios front-end. En este caso, se recomienda [agregar al menos dos tipos de nodos](#add-nodes-to-or-remove-nodes-from-a-node-type) al clúster.  

¿Tienen los servicios distintos requisitos de infraestructura, como, por ejemplo, una RAM mayor o un número más alto de ciclos de CPU? Por ejemplo, la aplicación contiene un servicio front-end y un servicio back-end. El servicio front-end se puede ejecutar en máquinas virtuales más pequeñas (tamaños de VM como D2) que tienen puertos abiertos a Internet. Pero el servicio back-end es de cálculo intensivo y tiene que ejecutarse en máquinas virtuales más grandes (con tamaños de VM como D4, D6 o D15) que no son accesibles desde Internet. En este caso, se recomienda [agregar dos o más tipos de nodo](#add-nodes-to-or-remove-nodes-from-a-node-type) a su clúster. Esto permite que cada tipo de nodo pueda tener distintas propiedades, por ejemplo, conectividad a Internet o tamaño de máquina virtual. El número de máquinas virtuales también se puede escalar de forma independiente.

Al cambiar la escala de un clúster de Azure, tenga en cuenta las directrices siguientes:

* Un único conjunto de escalado o tipo de nodo de Service Fabric no puede contener más de 100 nodos o máquinas virtuales.  Para escalar un clúster más allá de 100 nodos, agregue tipos de nodo adicionales.
* Los tipos de nodo principal que ejecutan cargas de trabajo de producción siempre deben tener un [nivel de durabilidad][durability] de Gold o Silver, y siempre cinco nodos como mínimo.
* Los tipos de nodo no principales que ejecutan cargas de trabajo de producción con estado siempre deben tener cinco o más nodos.
* Los tipos de nodo no principales que ejecutan cargas de trabajo de producción sin estado siempre deben tener dos o más nodos.
* Cualquier tipo de nodo del [nivel de durabilidad][durability] Gold o Silver siempre debe tener cinco o más nodos.
* Si reduce horizontalmente un tipo de nodo principal (eliminar nodos del nodo), nunca debe reducir el número de instancias a menos de lo que el [nivel de confiabilidad][reliability] requiera.

Para más información, consulte la [guía de capacidad del clúster](service-fabric-cluster-capacity.md).

## <a name="export-the-template-for-the-resource-group"></a>Exportación de la plantilla para el grupo de recursos

Después de crear un [clúster Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) seguro y configurar el grupo de recursos correctamente, exporte la plantilla de Resource Manager para el grupo de recursos. La exportación de la plantilla permite automatizar las futuras implementaciones del clúster y sus recursos, ya que la plantilla contiene la infraestructura completa.  Para más información sobre la exportación de plantillas, consulte [Administrar grupos de recursos de Azure Resource Manager mediante Azure Portal](../azure-resource-manager/management/manage-resource-groups-portal.md).

1. En [Azure Portal](https://portal.azure.com), vaya al grupo de recursos que contiene el clúster (**sfclustertutorialgroup**, si está siguiendo este tutorial). 

2. En el panel izquierdo, seleccione **Implementaciones** o seleccione el vínculo situado bajo **Implementaciones**. 

3. Seleccione la implementación correcta más reciente de la lista.

4. En el panel izquierdo, seleccione **Plantilla** y, a continuación, seleccione **Descargar** para exportar la plantilla como un archivo ZIP.  Guarde la plantilla y los parámetros en el equipo local.

## <a name="add-nodes-to-or-remove-nodes-from-a-node-type"></a>Incorporación o eliminación de nodos de un tipo de nodo

El escalado vertical u horizontal cambia el número de nodos del clúster. Al escalar o reducir horizontalmente, agrega más instancias de máquina virtual al conjunto de escalado. Estas instancias se convierten en los nodos que usa Service Fabric. Service Fabric sabe cuándo el conjunto de escalado tiene varias instancias agregadas (mediante la escalabilidad horizontal) y reacciona automáticamente. Puede escalar el clúster en cualquier momento, incluso con cargas de trabajo en ejecución en el clúster.

### <a name="update-the-template"></a>Actualización de la plantilla

[Exporte un archivo de plantilla y parámetros](#export-the-template-for-the-resource-group) del grupo de recursos para la implementación más reciente.  Abra el archivo *parameters.json*.  Si ha implementado el clúster con la [plantilla de muestra][template] en este tutorial, hay tres tipos de nodos en el clúster y tres parámetros que establecen el número de nodos para cada tipo de nodo: *nt0InstanceCount*, *nt1InstanceCount* y *nt2InstanceCount*.  El parámetro *nt1InstanceCount*, por ejemplo, establece el recuento de instancias para el segundo tipo de nodo y el número de máquinas virtuales en el conjunto de escalado de máquinas virtuales asociado.

Por ello, si actualiza el valor de *nt1InstanceCount*, se cambia el número de nodos en el segundo tipo de nodo.  Recuerde que no se puede escalar horizontalmente un tipo de nodo a más de 100 nodos.  Los tipos de nodo no principales que ejecutan cargas de trabajo de producción con estado siempre deben tener cinco o más nodos. Los tipos de nodo no principales que ejecutan cargas de trabajo de producción sin estado siempre deben tener dos o más nodos.

Si va a escalar verticalmente, al quitar nodos de un tipo de nodo de [nivel de durabilidad][durability] Bronze, tiene que [quitar manualmente el estado de esos nodos](service-fabric-cluster-scale-in-out.md#manually-remove-vms-from-a-node-typevirtual-machine-scale-set).  Para el nivel de durabilidad Silver y Gold, la plataforma realiza automáticamente estos pasos.

### <a name="deploy-the-updated-template"></a>Implementar la plantilla actualizada
Guarde los cambios realizados en los archivos *template.json* y *parameters.json*.  Para implementar la plantilla actualizada, ejecute el comando siguiente:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName sfclustertutorialgroup -TemplateFile c:\temp\template.json -TemplateParameterFile c:\temp\parameters.json -Name "ChangingInstanceCount"
```
O bien, el siguiente comando de la CLI de Azure:
```azurecli
az group deployment create --resource-group sfclustertutorialgroup --template-file c:\temp\template.json --parameters c:\temp\parameters.json
```

## <a name="add-a-node-type-to-the-cluster"></a>Incorporación de un tipo de nodo al clúster

Cada tipo de nodo definido en un clúster de Service Fabric que se ejecuta en Azure está configurado como un [conjunto de escalado de máquinas virtuales independiente](service-fabric-cluster-nodetypes.md). Cada tipo de nodo, a continuación, se puede administrar por separado. Cada tipo de nodo se puede escalar o reducir verticalmente de forma independiente, tiene diferentes conjuntos de puertos abiertos y puede usar distintas métricas de capacidad. También puede cambiar de forma independiente la SKU del sistema operativo que se ejecuta en cada nodo de clúster, pero tenga en cuenta que no puede tener una combinación de Windows y Linux en ejecución en el clúster de muestra. Un conjunto de escalado o de tipo de nodo único no puede contener más de 100 nodos.  Puede escalar un clúster horizontalmente a más de 100 nodos mediante la adición de tipos de nodos y conjuntos de escalado adicionales. Puede escalar el clúster en cualquier momento, incluso con cargas de trabajo en ejecución en el clúster.

### <a name="update-the-template"></a>Actualización de la plantilla

[Exporte un archivo de plantilla y parámetros](#export-the-template-for-the-resource-group) del grupo de recursos para la implementación más reciente.  Abra el archivo *parameters.json*.  Si ha implementado el clúster con la [plantilla de muestra][template] en este tutorial, hay tres tipos de nodo en el clúster.  En esta sección, se agregará un cuarto tipo de nodo; para ello, se actualiza e implementa una plantilla de Resource Manager. 

Además del nuevo tipo de nodo, también agregará el conjunto de escalado de máquinas virtuales asociado (que se ejecuta en una subred independiente de la red virtual) y el grupo de seguridad de red.  Puede agregar la dirección IP pública nueva o existente, y los recursos del equilibrador de carga de Azure para el nuevo conjunto de escalado.  El nuevo tipo de nodo tiene un [nivel de durabilidad][durability] Silver y el tamaño "Standard_D2_V2".

En el archivo *template.json*, agregue los siguientes parámetros nuevos:
```json
"nt3InstanceCount": {
    "defaultValue": 5,
    "type": "Int",
    "metadata": {
        "description": "Instance count for node type"
    }
},
"vmNodeType3Size": {
    "defaultValue": "Standard_D2_V2",
    "type": "String"
},
```

En el archivo *template.json*, agregue las siguientes variables nuevas:
```json
"lbID3": "[resourceId('Microsoft.Network/loadBalancers',concat('LB','-', parameters('clusterName'),'-',variables('vmNodeType3Name')))]",
"lbIPConfig3": "[concat(variables('lbID3'),'/frontendIPConfigurations/LoadBalancerIPConfig')]",
"lbPoolID3": "[concat(variables('lbID3'),'/backendAddressPools/LoadBalancerBEAddressPool')]",
"lbProbeID3": "[concat(variables('lbID3'),'/probes/FabricGatewayProbe')]",
"lbHttpProbeID3": "[concat(variables('lbID3'),'/probes/FabricHttpGatewayProbe')]",
"lbNatPoolID3": "[concat(variables('lbID3'),'/inboundNatPools/LoadBalancerBEAddressNatPool')]",
"vmNodeType3Name": "[toLower(concat('NT4', variables('vmName')))]",
"vmStorageAccountName3": "[toLower(concat(uniqueString(resourceGroup().id), '1', '3' ))]",
"nt3applicationStartPort": "20000",
"nt3applicationEndPort": "30000",
"nt3ephemeralStartPort": "49152",
"nt3ephemeralEndPort": "65534",
"nt3fabricTcpGatewayPort": "19000",
"nt3fabricHttpGatewayPort": "19080",
"nt3reverseProxyEndpointPort": "19081",
"subnet3Name": "Subnet-3",
"subnet3Prefix": "10.0.3.0/24",
"subnet3Ref": "[concat(variables('vnetID'),'/subnets/',variables('subnet3Name'))]",
```

En el archivo *template.json*, agregue una nueva subred al recurso de red virtual:
```json
{
    "type": "Microsoft.Network/virtualNetworks",
    "name": "[variables('virtualNetworkName')]",
    "apiVersion": "2018-08-01",
    "location": "[variables('computeLocation')]",
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    },
    "properties": {
        "addressSpace": {
            "addressPrefixes": [
                "[variables('addressPrefix')]"
            ]
        },
        "subnets": [
            ...
            {
                "name": "[variables('subnet3Name')]",
                "properties": {
                    "addressPrefix": "[variables('subnet3Prefix')]",
                    "networkSecurityGroup": {
                        "id": "[resourceId('Microsoft.Network/networkSecurityGroups', concat('nsg', variables('subnet3Name')))]"
                    }
                }
            }
        ]
    },
    "dependsOn": [
        ...
        "[concat('Microsoft.Network/networkSecurityGroups/', concat('nsg', variables('subnet3Name')))]"
    ]
},
```

En el archivo *template.json*, agregue los recursos de la nueva dirección IP pública y del equilibrador de carga:
```json
{
    "type": "Microsoft.Network/publicIPAddresses",
    "name": "[concat(variables('lbIPName'),'-',variables('vmNodeType3Name'))]",
    "apiVersion": "2018-08-01",
    "location": "[variables('computeLocation')]",
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    },
    "properties": {
        "dnsSettings": {
            "domainNameLabel": "[concat(variables('dnsName'),'-','nt4')]"
        },
        "publicIPAllocationMethod": "Dynamic"
    }
},
        {
    "type": "Microsoft.Network/loadBalancers",
    "name": "[concat('LB','-', parameters('clusterName'),'-',variables('vmNodeType3Name'))]",
    "apiVersion": "2018-08-01",
    "location": "[variables('computeLocation')]",
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    },
    "properties": {
        "frontendIPConfigurations": [
            {
                "name": "LoadBalancerIPConfig",
                "properties": {
                    "publicIPAddress": {
                        "id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(variables('lbIPName'),'-',variables('vmNodeType3Name')))]"
                    }
                }
            }
        ],
        "backendAddressPools": [
            {
                "name": "LoadBalancerBEAddressPool",
                "properties": {}
            }
        ],
        "loadBalancingRules": [
            {
                "name": "LBRule",
                "properties": {
                    "backendAddressPool": {
                        "id": "[variables('lbPoolID3')]"
                    },
                    "backendPort": "[variables('nt3fabricTcpGatewayPort')]",
                    "enableFloatingIP": "false",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig3')]"
                    },
                    "frontendPort": "[variables('nt3fabricTcpGatewayPort')]",
                    "idleTimeoutInMinutes": "5",
                    "probe": {
                        "id": "[variables('lbProbeID3')]"
                    },
                    "protocol": "tcp"
                }
            },
            {
                "name": "LBHttpRule",
                "properties": {
                    "backendAddressPool": {
                        "id": "[variables('lbPoolID3')]"
                    },
                    "backendPort": "[variables('nt3fabricHttpGatewayPort')]",
                    "enableFloatingIP": "false",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig3')]"
                    },
                    "frontendPort": "[variables('nt3fabricHttpGatewayPort')]",
                    "idleTimeoutInMinutes": "5",
                    "probe": {
                        "id": "[variables('lbHttpProbeID3')]"
                    },
                    "protocol": "tcp"
                }
            },
            {
                "name": "AppPortLBRule1",
                "properties": {
                    "backendAddressPool": {
                        "id": "[variables('lbPoolID3')]"
                    },
                    "backendPort": "[parameters('loadBalancedAppPort1')]",
                    "enableFloatingIP": "false",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig3')]"
                    },
                    "frontendPort": "[parameters('loadBalancedAppPort1')]",
                    "idleTimeoutInMinutes": "5",
                    "probe": {
                        "id": "[concat(variables('lbID3'),'/probes/AppPortProbe1')]"
                    },
                    "protocol": "tcp"
                }
            },
            {
                "name": "AppPortLBRule2",
                "properties": {
                    "backendAddressPool": {
                        "id": "[variables('lbPoolID3')]"
                    },
                    "backendPort": "[parameters('loadBalancedAppPort2')]",
                    "enableFloatingIP": "false",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig3')]"
                    },
                    "frontendPort": "[parameters('loadBalancedAppPort2')]",
                    "idleTimeoutInMinutes": "5",
                    "probe": {
                        "id": "[concat(variables('lbID3'),'/probes/AppPortProbe2')]"
                    },
                    "protocol": "tcp"
                }
            }
        ],
        "probes": [
            {
                "name": "FabricGatewayProbe",
                "properties": {
                    "intervalInSeconds": 5,
                    "numberOfProbes": 2,
                    "port": "[variables('nt3fabricTcpGatewayPort')]",
                    "protocol": "tcp"
                }
            },
            {
                "name": "FabricHttpGatewayProbe",
                "properties": {
                    "intervalInSeconds": 5,
                    "numberOfProbes": 2,
                    "port": "[variables('nt3fabricHttpGatewayPort')]",
                    "protocol": "tcp"
                }
            },
            {
                "name": "AppPortProbe1",
                "properties": {
                    "intervalInSeconds": 5,
                    "numberOfProbes": 2,
                    "port": "[parameters('loadBalancedAppPort1')]",
                    "protocol": "tcp"
                }
            },
            {
                "name": "AppPortProbe2",
                "properties": {
                    "intervalInSeconds": 5,
                    "numberOfProbes": 2,
                    "port": "[parameters('loadBalancedAppPort2')]",
                    "protocol": "tcp"
                }
            }
        ],
        "inboundNatPools": [
            {
                "name": "LoadBalancerBEAddressNatPool",
                "properties": {
                    "backendPort": "3389",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig3')]"
                    },
                    "frontendPortRangeEnd": "4500",
                    "frontendPortRangeStart": "3389",
                    "protocol": "tcp"
                }
            }
        ]
    },
    "dependsOn": [
        "[concat('Microsoft.Network/publicIPAddresses/',concat(variables('lbIPName'),'-',variables('vmNodeType3Name')))]"
    ]
},
```

En el archivo *template.json*, agregue los nuevos recursos de conjunto de escalado de máquinas virtuales y de grupo de seguridad de red.  La propiedad NodeTypeRef dentro de las propiedades de extensión de Service Fabric del conjunto de escalado de máquinas virtuales asigna el tipo de nodo especificado al conjunto de escalado.

```json
{
    "type": "Microsoft.Network/networkSecurityGroups",
    "name": "[concat('nsg', variables('subnet3Name'))]",
    "apiVersion": "2018-08-01",
    "location": "[resourceGroup().location]",
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    },
    "properties": {
        "securityRules": [
            {
                "name": "allowSvcFabSMB",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "445",
                    "direction": "Inbound",
                    "priority": 3950,
                    "protocol": "*",
                    "sourceAddressPrefix": "VirtualNetwork",
                    "sourcePortRange": "*",
                    "description": "allow SMB traffic within the net, used by fabric to move packages around"
                }
            },
            {
                "name": "allowSvcFabCluser",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "1025-1027",
                    "direction": "Inbound",
                    "priority": 3920,
                    "protocol": "*",
                    "sourceAddressPrefix": "VirtualNetwork",
                    "sourcePortRange": "*",
                    "description": "allow ports within vnet that are used by the fabric to talk between nodes"
                }
            },
            {
                "name": "allowSvcFabEphemeral",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[concat(variables('nt3ephemeralStartPort'), '-', variables('nt3ephemeralEndPort'))]",
                    "direction": "Inbound",
                    "priority": 3930,
                    "protocol": "*",
                    "sourceAddressPrefix": "VirtualNetwork",
                    "sourcePortRange": "*",
                    "description": "allow fabric ephemeral ports within the vnet"
                }
            },
            {
                "name": "allowSvcFabPortal",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[variables('nt3fabricHttpGatewayPort')]",
                    "direction": "Inbound",
                    "priority": 3900,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow port used to access the fabric cluster web portal"
                }
            },
            {
                "name": "allowSvcFabClient",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[variables('nt3fabricTcpGatewayPort')]",
                    "direction": "Inbound",
                    "priority": 3910,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow port used by the fabric client (includes powershell)"
                }
            },
            {
                "name": "allowSvcFabApplication",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[concat(variables('nt3applicationStartPort'), '-', variables('nt3applicationEndPort'))]",
                    "direction": "Inbound",
                    "priority": 3940,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow fabric application ports within the vnet"
                }
            },
            {
                "name": "blockAll",
                "properties": {
                    "access": "Deny",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "*",
                    "direction": "Inbound",
                    "priority": 4095,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "block all traffic except what we've explicitly allowed"
                }
            },
            {
                "name": "allowVNetRDP",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "3389",
                    "direction": "Inbound",
                    "priority": 3960,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow RDP within the net"
                }
            },
            {
                "name": "allowSvcFabReverseProxy",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[variables('nt3reverseProxyEndpointPort')]",
                    "direction": "Inbound",
                    "priority": 3980,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow port used to access the fabric cluster using reverse proxy"
                }
            },
            {
                "name": "allowAppPort1",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[parameters('loadBalancedAppPort1')]",
                    "direction": "Inbound",
                    "priority": 2001,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow public application port 1"
                }
            },
            {
                "name": "allowAppPort2",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[parameters('loadBalancedAppPort2')]",
                    "direction": "Inbound",
                    "priority": 2002,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow public application port 2"
                }
            }
        ]
    }
},
{
    "type": "Microsoft.Compute/virtualMachineScaleSets",
    "sku": {
        "name": "[parameters('vmNodeType3Size')]",
        "capacity": "[parameters('nt3InstanceCount')]",
        "tier": "Standard"
    },
    "name": "[variables('vmNodeType3Name')]",
    "apiVersion": "2018-10-01",
    "location": "[variables('computeLocation')]",
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    },
    "properties": {
        "overprovision": "[variables('overProvision')]",
        "upgradePolicy": {
            "mode": "Automatic"
        },
        "virtualMachineProfile": {
            "extensionProfile": {
                "extensions": [
                    {
                        "name": "[concat(variables('vmNodeType3Name'),'OMS')]",
                        "properties": {
                            "publisher": "Microsoft.EnterpriseCloud.Monitoring",
                            "type": "MicrosoftMonitoringAgent",
                            "typeHandlerVersion": "1.0",
                            "autoUpgradeMinorVersion": true,
                            "settings": {
                                "workspaceId": "[reference(resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename')), '2015-11-01-preview').customerId]"
                            },
                            "protectedSettings": {
                                "workspaceKey": "[listKeys(resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename')),'2015-11-01-preview').primarySharedKey]"
                            }
                        }
                    },
                    {
                        "name": "[concat('ServiceFabricNodeVmExt','_vmNodeType3Name')]",
                        "properties": {
                            "type": "ServiceFabricNode",
                            "autoUpgradeMinorVersion": true,
                            "protectedSettings": {
                                "StorageAccountKey1": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('supportLogStorageAccountName')),'2015-05-01-preview').key1]",
                                "StorageAccountKey2": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('supportLogStorageAccountName')),'2015-05-01-preview').key2]"
                            },
                            "publisher": "Microsoft.Azure.ServiceFabric",
                            "settings": {
                                "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
                                "nodeTypeRef": "[variables('vmNodeType3Name')]",
                                "dataPath": "D:\\SvcFab",
                                "durabilityLevel": "Silver",
                                "enableParallelJobs": true,
                                "nicPrefixOverride": "[variables('subnet3Prefix')]",
                                "certificate": {
                                    "thumbprint": "[parameters('certificateThumbprint')]",
                                    "x509StoreName": "[parameters('certificateStoreValue')]"
                                }
                            },
                            "typeHandlerVersion": "1.0"
                        }
                    },
                    {
                        "name": "[concat('VMDiagnosticsVmExt','_vmNodeType3Name')]",
                        "properties": {
                            "type": "IaaSDiagnostics",
                            "autoUpgradeMinorVersion": true,
                            "protectedSettings": {
                                "storageAccountName": "[variables('applicationDiagnosticsStorageAccountName')]",
                                "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('applicationDiagnosticsStorageAccountName')),'2015-05-01-preview').key1]",
                                "storageAccountEndPoint": "https://core.windows.net/"
                            },
                            "publisher": "Microsoft.Azure.Diagnostics",
                            "settings": {
                                "WadCfg": {
                                    "DiagnosticMonitorConfiguration": {
                                        "overallQuotaInMB": "50000",
                                        "EtwProviders": {
                                            "EtwEventSourceProviderConfiguration": [
                                                {
                                                    "provider": "Microsoft-ServiceFabric-Actors",
                                                    "scheduledTransferKeywordFilter": "1",
                                                    "scheduledTransferPeriod": "PT5M",
                                                    "DefaultEvents": {
                                                        "eventDestination": "ServiceFabricReliableActorEventTable"
                                                    }
                                                },
                                                {
                                                    "provider": "Microsoft-ServiceFabric-Services",
                                                    "scheduledTransferPeriod": "PT5M",
                                                    "DefaultEvents": {
                                                        "eventDestination": "ServiceFabricReliableServiceEventTable"
                                                    }
                                                }
                                            ],
                                            "EtwManifestProviderConfiguration": [
                                                {
                                                    "provider": "cbd93bc2-71e5-4566-b3a7-595d8eeca6e8",
                                                    "scheduledTransferLogLevelFilter": "Information",
                                                    "scheduledTransferKeywordFilter": "4611686018427387904",
                                                    "scheduledTransferPeriod": "PT5M",
                                                    "DefaultEvents": {
                                                        "eventDestination": "ServiceFabricSystemEventTable"
                                                    }
                                                }
                                            ]
                                        }
                                    }
                                },
                                "StorageAccount": "[variables('applicationDiagnosticsStorageAccountName')]"
                            },
                            "typeHandlerVersion": "1.5"
                        }
                    },
                    {
                        "name": "[concat('VMIaaSAntimalware','_vmNodeType3Name')]",
                        "properties": {
                            "publisher": "Microsoft.Azure.Security",
                            "type": "IaaSAntimalware",
                            "typeHandlerVersion": "1.5",
                            "settings": {
                                "AntimalwareEnabled": "true",
                                "Exclusions": {
                                    "Paths": "D:\\SvcFab;D:\\SvcFab\\Log;C:\\Program Files\\Microsoft Service Fabric",
                                    "Processes": "Fabric.exe;FabricHost.exe;FabricInstallerService.exe;FabricSetup.exe;FabricDeployer.exe;ImageBuilder.exe;FabricGateway.exe;FabricDCA.exe;FabricFAS.exe;FabricUOS.exe;FabricRM.exe;FileStoreService.exe"
                                },
                                "RealtimeProtectionEnabled": "true",
                                "ScheduledScanSettings": {
                                    "isEnabled": "true",
                                    "scanType": "Quick",
                                    "day": "7",
                                    "time": "120"
                                }
                            },
                            "protectedSettings": null
                        }
                    }
                ]
            },
            "networkProfile": {
                "networkInterfaceConfigurations": [
                    {
                        "name": "[concat(variables('nicName'), '-2')]",
                        "properties": {
                            "ipConfigurations": [
                                {
                                    "name": "[concat(variables('nicName'),'-',2)]",
                                    "properties": {
                                        "loadBalancerBackendAddressPools": [
                                            {
                                                "id": "[variables('lbPoolID3')]"
                                            }
                                        ],
                                        "loadBalancerInboundNatPools": [
                                            {
                                                "id": "[variables('lbNatPoolID3')]"
                                            }
                                        ],
                                        "subnet": {
                                            "id": "[variables('subnet3Ref')]"
                                        }
                                    }
                                }
                            ],
                            "primary": true
                        }
                    }
                ]
            },
            "osProfile": {
                "adminPassword": "[parameters('adminPassword')]",
                "adminUsername": "[parameters('adminUsername')]",
                "computernamePrefix": "[variables('vmNodeType3Name')]",
                "secrets": [
                    {
                        "sourceVault": {
                            "id": "[parameters('sourceVaultValue')]"
                        },
                        "vaultCertificates": [
                            {
                                "certificateStore": "[parameters('certificateStoreValue')]",
                                "certificateUrl": "[parameters('certificateUrlValue')]"
                            }
                        ]
                    }
                ]
            },
            "storageProfile": {
                "imageReference": {
                    "publisher": "[parameters('vmImagePublisher')]",
                    "offer": "[parameters('vmImageOffer')]",
                    "sku": "[parameters('vmImageSku')]",
                    "version": "[parameters('vmImageVersion')]"
                },
                "osDisk": {
                    "caching": "ReadOnly",
                    "createOption": "FromImage",
                    "managedDisk": {
                        "storageAccountType": "[parameters('storageAccountType')]"
                    }
                }
            }
        }
    },
    "dependsOn": [
        "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]",
        "[concat('Microsoft.Network/loadBalancers/', concat('LB','-', parameters('clusterName'),'-',variables('vmNodeType3Name')))]",
        "[concat('Microsoft.Storage/storageAccounts/', variables('supportLogStorageAccountName'))]",
        "[concat('Microsoft.Storage/storageAccounts/', variables('applicationDiagnosticsStorageAccountName'))]"
    ]
},
```

En el archivo *template.json*, actualice el recurso de clúster y agregue un nuevo tipo de nodo:
```json
{
    "type": "Microsoft.ServiceFabric/clusters",
    "name": "[parameters('clusterName')]",
    "apiVersion": "2018-02-01",
    "location": "[parameters('clusterLocation')]",
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    },
    "properties": {
        "nodeTypes": [
            ...
            {
                "name": "[variables('vmNodeType3Name')]",
                "applicationPorts": {
                    "endPort": "[variables('nt3applicationEndPort')]",
                    "startPort": "[variables('nt3applicationStartPort')]"
                },
                "clientConnectionEndpointPort": "[variables('nt3fabricTcpGatewayPort')]",
                "durabilityLevel": "Silver",
                "ephemeralPorts": {
                    "endPort": "[variables('nt3ephemeralEndPort')]",
                    "startPort": "[variables('nt3ephemeralStartPort')]"
                },
                "httpGatewayEndpointPort": "[variables('nt3fabricHttpGatewayPort')]",
                "isPrimary": false,
                "reverseProxyEndpointPort": "[variables('nt3reverseProxyEndpointPort')]",
                "vmInstanceCount": "[parameters('nt3InstanceCount')]"
            }
        ],    
    }
}                
```

En el archivo *parameters.json*, agregue los siguientes valores y parámetros nuevos:
```json
"nt3InstanceCount": {
    "Value": 5    
},
"vmNodeType3Size": {
    "Value": "Standard_D2_V2"
},
```

### <a name="deploy-the-updated-template"></a>Implementar la plantilla actualizada
Guarde los cambios realizados en los archivos *template.json* y *parameters.json*.  Para implementar la plantilla actualizada, ejecute el comando siguiente:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName sfclustertutorialgroup -TemplateFile c:\temp\template.json -TemplateParameterFile c:\temp\parameters.json -Name "AddingNodeType"
```
O bien, el siguiente comando de la CLI de Azure:
```azurecli
az group deployment create --resource-group sfclustertutorialgroup --template-file c:\temp\template.json --parameters c:\temp\parameters.json
```

## <a name="remove-a-node-type-from-the-cluster"></a>Eliminación de un tipo de nodo del clúster
Después de crear un clúster de Service Fabric, para escalarlo horizontalmente no tiene más que quitar un tipo de nodo (conjunto de escalado de máquinas virtuales) y todos sus nodos. Puede escalar el clúster en cualquier momento, incluso con cargas de trabajo en ejecución en el clúster. Según se escala el clúster, las aplicaciones se escalan automáticamente.

> [!WARNING]
> No es aconsejable usar Remove-AzServiceFabricNodeType para quitar un tipo de nodo de un clúster de producción de forma frecuente. Se trata de un comando muy peligroso, ya que elimina el recurso del conjunto de escalado de máquinas virtuales que hay detrás del tipo de nodo. 

Para quitar el tipo de nodo, ejecute el cmdlet [Remove-AzServiceFabricNodeType](/powershell/module/az.servicefabric/remove-azservicefabricnodetype).  El tipo de nodo tiene que ser de [nivel de durabilidad][durability] Silver o Gold. El cmdlet elimina el conjunto de escalado asociado con el tipo de nodo y tarda algún tiempo en completarse.  A continuación, ejecute el cmdlet [Remove-ServiceFabricNodeState](/powershell/module/servicefabric/remove-servicefabricnodestate?view=azureservicefabricps) en cada uno de los nodos que se van a quitar, lo que elimina el estado de nodo y quita los nodos del clúster. Si hay servicios en los nodos, los servicios primero se moverán a otro nodo. Si el administrador de clústeres no puede encontrar un nodo para la réplica o servicio, la operación se retrasará o bloqueará.

```powershell
$groupname = "sfclustertutorialgroup"
$nodetype = "nt4vm"
$clustername = "mysfcluster123"

Remove-AzServiceFabricNodeType -Name $clustername  -NodeType $nodetype -ResourceGroupName $groupname

Connect-ServiceFabricCluster -ConnectionEndpoint mysfcluster123.eastus.cloudapp.azure.com:19000 `
          -KeepAliveIntervalInSec 10 `
          -X509Credential -ServerCertThumbprint <thumbprint> `
          -FindType FindByThumbprint -FindValue <thumbprint> `
          -StoreLocation CurrentUser -StoreName My

$nodes = Get-ServiceFabricNode | Where-Object {$_.NodeType -eq $nodetype} | Sort-Object { $_.NodeName.Substring($_.NodeName.LastIndexOf('_') + 1) } -Descending

Foreach($node in $nodes)
{
    Remove-ServiceFabricNodeState -NodeName $node.NodeName -TimeoutSec 300 -Force 
}
```

## <a name="increase-node-resources"></a>Aumento de los recursos de nodo 
Después de crear un clúster de Service Fabric, puede escalar el tipo de nodo del clúster verticalmente (cambiar los recursos de los nodos) o actualizar el sistema operativo de las máquinas virtuales del tipo de nodo reemplazando el tipo de nodo original por uno nuevo (con la SKU de la máquina virtual o la imagen del sistema operativo actualizadas). Para más información, consulte [Escalado vertical de un tipo de nodo de Azure Service Fabric](service-fabric-scale-up-primary-node-type.md).

> [!IMPORTANT]
> No intente nunca realizar un cambio en contexto de la SKU de la máquina virtual o de la imagen del sistema operativo ya que es una operación peligrosa y no se admite.

Si ello no es posible, puede crear un clúster nuevo y [restaurar el estado de la aplicación](service-fabric-reliable-services-backup-restore.md) (si procede) a partir del clúster anterior. No es preciso restaurar el estado de ningún servicio del sistema, ya que se vuelven a crear al implementar las aplicaciones en el clúster nuevo. Si ha ejecutado aplicaciones sin estado en el clúster, lo único que hace es implementar las aplicaciones en el clúster nuevo, no hay nada que para restaurar.

### <a name="update-the-template"></a>Actualización de la plantilla

[Exporte un archivo de plantilla y parámetros](#export-the-template-for-the-resource-group) del grupo de recursos para la implementación más reciente.  Abra el archivo *parameters.json*.  Si ha implementado el clúster con la [plantilla de muestra][template] en este tutorial, hay tres tipos de nodo en el clúster.  

El tamaño de las máquinas virtuales en el segundo tipo de nodo se establece en el parámetro *vmNodeType1Size*.  Cambie el valor del parámetro *vmNodeType1Size* de Standard_D2_V2 a [Standard_D3_V2](../virtual-machines/dv2-dsv2-series.md), lo que duplica los recursos de cada instancia de máquina virtual.

La SKU de VM para los tres tipos de nodos se establece en el parámetro *vmImageSku*.  De nuevo, el cambio de la SKU de VM de un tipo de nodo debe llevarse a cabo con precaución y no se recomienda para el tipo de nodo principal.

### <a name="deploy-the-updated-template"></a>Implementar la plantilla actualizada
Guarde los cambios realizados en los archivos *template.json* y *parameters.json*.  Para implementar la plantilla actualizada, ejecute el comando siguiente:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName sfclustertutorialgroup -TemplateFile c:\temp\template.json -TemplateParameterFile c:\temp\parameters.json -Name "ScaleUpNodeType"
```
O bien, el siguiente comando de la CLI de Azure:
```azurecli
az group deployment create --resource-group sfclustertutorialgroup --template-file c:\temp\template.json --parameters c:\temp\parameters.json
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Agregar y quitar nodos (escalado horizontal y reducción horizontal)
> * Agregar y quitar tipos de nodos (escalado horizontal y reducción horizontal)
> * Aumentar los recursos de nodo (escalabilidad vertical)

Luego, pase al siguiente tutorial para aprender a actualizar el sistema de tiempo de ejecución de un clúster.
> [!div class="nextstepaction"]
> [Actualización del entorno en tiempo de ejecución de un clúster](service-fabric-tutorial-upgrade-cluster.md)

[durability]: service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster
[reliability]: service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster
[template]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Windows-3-NodeTypes-Secure-NSG/AzureDeploy.json
[parameters]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Windows-3-NodeTypes-Secure-NSG/AzureDeploy.Parameters.json
