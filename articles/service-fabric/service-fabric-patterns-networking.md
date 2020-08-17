---
title: Patrones de redes para Azure Service Fabric
description: En este artículo se describen los patrones de redes comunes de Service Fabric y cómo crear un clúster con las características de red de Azure.
ms.topic: conceptual
ms.date: 01/19/2018
ms.openlocfilehash: 20bd5e931307725016c3e2ad69dae91214b2caab
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421474"
---
# <a name="service-fabric-networking-patterns"></a>Patrones de redes de Service Fabric
Puede integrar el clúster de Azure Service Fabric con otras características de red de Azure. En este artículo se muestra cómo crear clústeres que usan las siguientes características:

- [Subred y red virtual existentes](#existingvnet)
- [Dirección IP pública estática](#staticpublicip)
- [Equilibrador de carga solo interno](#internallb)
- [Equilibrador de carga interno y externo](#internalexternallb)

Service Fabric se ejecuta en un conjunto de escalado de máquinas virtuales estándar. Cualquier funcionalidad que pueda usar en un conjunto de escalado de máquinas virtuales, puede usarse con un clúster de Service Fabric. Las secciones de red de las plantillas de Azure Resource Manager para los conjuntos de escalado de máquinas virtuales y Service Fabric son idénticas. Después de implementar en una red virtual existente, es fácil incorporar otras características de red como Azure ExpressRoute, Azure VPN Gateway, un grupo de seguridad de red y emparejamiento de redes virtuales.

### <a name="allowing-the-service-fabric-resource-provider-to-query-your-cluster"></a>Permitir que el proveedor de recursos de Service Fabric consulte el clúster

Service Fabric es único en relación con otras características de red en un aspecto. [Azure Portal](https://portal.azure.com) usa internamente el proveedor de recursos de Service Fabric para llamar a un clúster con el fin de obtener información de los nodos y las aplicaciones. El proveedor de recursos de Service Fabric requiere acceso de entrada público al puerto de la puerta de enlace HTTP (puerto 19080 de forma predeterminada) en el punto de conexión de administración. [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) utiliza el punto de conexión de administración para administrar el clúster. Este puerto también lo emplea el proveedor de recursos de Service Fabric para consultar información sobre el clúster con el fin de que se muestre en Azure Portal. 

Si el puerto 19080 no es accesible desde el proveedor de recursos de Service Fabric, aparecerá un mensaje del tipo *No se encontraron los nodos* en el portal, y la lista de nodos y aplicaciones aparecerá vacía. Si desea ver el clúster en Azure Portal, el equilibrador de carga debe exponer una dirección IP pública y el grupo de seguridad de red debe permitir el tráfico entrante en el puerto 19080. Si no se cumplen estos requisitos de configuración, Azure Portal no mostrará el estado del clúster.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="templates"></a>Plantillas

Todas las plantillas de Service Fabric están en [GitHub](https://github.com/Azure/service-fabric-scripts-and-templates/tree/master/templates/networking). Debe poder implementar las plantillas tal como están con los siguientes comandos de PowerShell. Si va a implementar la plantilla existente de Azure Virtual Network o la de dirección IP pública estática, lea primero la sección [Configuración inicial](#initialsetup) de este artículo.

<a id="initialsetup"></a>
## <a name="initial-setup"></a>Configuración inicial

### <a name="existing-virtual-network"></a>Red virtual existente

En el ejemplo siguiente, vamos a empezar con una red virtual existente denominada "ExistingRG-vnet" en el grupo de recursos **ExistingRG**. La subred recibe un nombre predeterminado. Estos recursos predeterminados son los que se crearon al usar Azure Portal para crear una máquina virtual estándar. Se podría crear la red y subred virtual sin necesidad de crear la máquina virtual, pero el objetivo principal de agregar un clúster a una red virtual existente es proporcionar conectividad de red a otras máquinas virtuales. La creación de la máquina virtual proporciona un buen ejemplo de cómo se usa normalmente una red virtual existente. Si el clúster de Service Fabric solo usa un equilibrador de carga interno, sin una dirección IP pública, podrá usar la máquina virtual con su dirección IP pública como *equipo de inicio de sesión seguro*.

### <a name="static-public-ip-address"></a>Dirección IP pública estática

Una dirección IP pública estática es normalmente un recurso dedicado que se administra de forma independiente de las máquinas virtuales a las que está asignada. Se aprovisiona en un grupo de recursos de red dedicado (en contraposición al aprovisionamiento en el grupo de recursos del clúster de Service Fabric en sí). Cree una dirección IP pública estática con el nombre "staticIP1" en el mismo grupo de recursos ExistingRG, ya sea a través de Azure Portal o mediante PowerShell:

```powershell
PS C:\Users\user> New-AzPublicIpAddress -Name staticIP1 -ResourceGroupName ExistingRG -Location westus -AllocationMethod Static -DomainNameLabel sfnetworking

Name                     : staticIP1
ResourceGroupName        : ExistingRG
Location                 : westus
Id                       : /subscriptions/1237f4d2-3dce-1236-ad95-123f764e7123/resourceGroups/ExistingRG/providers/Microsoft.Network/publicIPAddresses/staticIP1
Etag                     : W/"fc8b0c77-1f84-455d-9930-0404ebba1b64"
ResourceGuid             : 77c26c06-c0ae-496c-9231-b1a114e08824
ProvisioningState        : Succeeded
Tags                     :
PublicIpAllocationMethod : Static
IpAddress                : 40.83.182.110
PublicIpAddressVersion   : IPv4
IdleTimeoutInMinutes     : 4
IpConfiguration          : null
DnsSettings              : {
                             "DomainNameLabel": "sfnetworking",
                             "Fqdn": "sfnetworking.westus.cloudapp.azure.com"
                           }
```

### <a name="service-fabric-template"></a>Plantilla de Service Fabric

En los ejemplos de este artículo, usamos el archivo template.json de Service Fabric. Puede usar el asistente estándar del portal para descargar la plantilla desde el portal antes de crear un clúster. También puede usar una de las [plantillas de muestra](https://github.com/Azure-Samples/service-fabric-cluster-templates), como la del [clúster de cinco nodos de Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure).

<a id="existingvnet"></a>
## <a name="existing-virtual-network-or-subnet"></a>Subred y red virtual existentes

1. Cambie el parámetro de subred al nombre de la subred existente y agregue dos nuevos parámetros para hacer referencia a la red virtual existente:

    ```json
        "subnet0Name": {
                "type": "string",
                "defaultValue": "default"
            },
            "existingVNetRGName": {
                "type": "string",
                "defaultValue": "ExistingRG"
            },

            "existingVNetName": {
                "type": "string",
                "defaultValue": "ExistingRG-vnet"
            },
            /*
            "subnet0Name": {
                "type": "string",
                "defaultValue": "Subnet-0"
            },
            "subnet0Prefix": {
                "type": "string",
                "defaultValue": "10.0.0.0/24"
            },*/
    ```

   También puede comentar el parámetro con el nombre "virtualNetworkName" para que no le solicite escribir el nombre de la red virtual dos veces en la hoja de implementación del clúster en Azure Portal.

2. Marque como comentario el atributo `nicPrefixOverride` de `Microsoft.Compute/virtualMachineScaleSets`, ya que está usando la subred existente y se ha deshabilitado esta variable en el paso 1.

    ```json
            /*"nicPrefixOverride": "[parameters('subnet0Prefix')]",*/
    ```

3. Cambie la variable `vnetID` para que apunte a la red virtual existente:

    ```json
            /*old "vnetID": "[resourceId('Microsoft.Network/virtualNetworks',parameters('virtualNetworkName'))]",*/
            "vnetID": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', parameters('existingVNetRGName'), '/providers/Microsoft.Network/virtualNetworks/', parameters('existingVNetName'))]",
    ```

4. Quite `Microsoft.Network/virtualNetworks` de los recursos, para que Azure no cree una nueva red virtual:

    ```json
    /*{
    "apiVersion": "[variables('vNetApiVersion')]",
    "type": "Microsoft.Network/virtualNetworks",
    "name": "[parameters('virtualNetworkName')]",
    "location": "[parameters('computeLocation')]",
    "properties": {
        "addressSpace": {
            "addressPrefixes": [
                "[parameters('addressPrefix')]"
            ]
        },
        "subnets": [
            {
                "name": "[parameters('subnet0Name')]",
                "properties": {
                    "addressPrefix": "[parameters('subnet0Prefix')]"
                }
            }
        ]
    },
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    }
    },*/
    ```

5. Comente la red virtual del atributo `dependsOn` de `Microsoft.Compute/virtualMachineScaleSets`, para que no dependa de crear una nueva red virtual:

    ```json
    "apiVersion": "[variables('vmssApiVersion')]",
    "type": "Microsoft.Computer/virtualMachineScaleSets",
    "name": "[parameters('vmNodeType0Name')]",
    "location": "[parameters('computeLocation')]",
    "dependsOn": [
        /*"[concat('Microsoft.Network/virtualNetworks/', parameters('virtualNetworkName'))]",
        */
        "[Concat('Microsoft.Storage/storageAccounts/', variables('uniqueStringArray0')[0])]",

    ```

6. Implemente la plantilla:

    ```powershell
    New-AzResourceGroup -Name sfnetworkingexistingvnet -Location westus
    New-AzResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkingexistingvnet -TemplateFile C:\SFSamples\Final\template\_existingvnet.json
    ```

    Después de la implementación, la red virtual debe incluir las nuevas máquinas virtuales del conjunto de escalado. El tipo de nodo del conjunto de escalado de máquinas virtuales debe mostrar la red virtual y la subred existentes. También puede usar el protocolo de escritorio remoto (RDP) para tener acceso a la máquina virtual que se encontraba ya en la red virtual y hacer ping en las nuevas máquinas virtuales del conjunto de escalado:

    ```
    C:>\Users\users>ping 10.0.0.5 -n 1
    C:>\Users\users>ping NOde1000000 -n 1
    ```

Para obtener otro ejemplo, consulte [uno que no es específico de Service Fabric](https://github.com/gbowerman/azure-myriad/tree/main/existing-vnet).


<a id="staticpublicip"></a>
## <a name="static-public-ip-address"></a>Dirección IP pública estática

1. Agregue parámetros para el nombre del grupo de recursos de direcciones IP estáticas existente, el nombre y el nombre de dominio completo (FQDN):

    ```json
    "existingStaticIPResourceGroup": {
                "type": "string"
            },
            "existingStaticIPName": {
                "type": "string"
            },
            "existingStaticIPDnsFQDN": {
                "type": "string"
    }
    ```

2. Elimine el parámetro `dnsName`. (La dirección IP estática ya tiene uno).

    ```json
    /*
    "dnsName": {
        "type": "string"
    },
    */
    ```

3. Agregue una variable para hacer referencia a la dirección IP estática existente:

    ```json
    "existingStaticIP": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', parameters('existingStaticIPResourceGroup'), '/providers/Microsoft.Network/publicIPAddresses/', parameters('existingStaticIPName'))]",
    ```

4. Quite `Microsoft.Network/publicIPAddresses` de los recursos, para que Azure no cree una nueva dirección IP:

    ```json
    /*
    {
        "apiVersion": "[variables('publicIPApiVersion')]",
        "type": "Microsoft.Network/publicIPAddresses",
        "name": "[concat(parameters('lbIPName'),)'-', '0')]",
        "location": "[parameters('computeLocation')]",
        "properties": {
            "dnsSettings": {
                "domainNameLabel": "[parameters('dnsName')]"
            },
            "publicIPAllocationMethod": "Dynamic"        
        },
        "tags": {
            "resourceType": "Service Fabric",
            "clusterName": "[parameters('clusterName')]"
        }
    }, */
    ```

5. Comente la dirección IP del atributo `dependsOn` de `Microsoft.Network/loadBalancers`, para que no dependa de crear una nueva dirección IP:

    ```json
    "apiVersion": "[variables('lbIPApiVersion')]",
    "type": "Microsoft.Network/loadBalancers",
    "name": "[concat('LB', '-', parameters('clusterName'), '-', parameters('vmNodeType0Name'))]",
    "location": "[parameters('computeLocation')]",
    /*
    "dependsOn": [
        "[concat('Microsoft.Network/publicIPAddresses/', concat(parameters('lbIPName'), '-', '0'))]"
    ], */
    "properties": {
    ```

6. En el recurso `Microsoft.Network/loadBalancers`, cambie el elemento `publicIPAddress` de `frontendIPConfigurations` para que haga referencia a la dirección IP estática existente en lugar de a una recién creada:

    ```json
                "frontendIPConfigurations": [
                        {
                            "name": "LoadBalancerIPConfig",
                            "properties": {
                                "publicIPAddress": {
                                    /*"id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(parameters('lbIPName'),'-','0'))]"*/
                                    "id": "[variables('existingStaticIP')]"
                                }
                            }
                        }
                    ],
    ```

7. En el recurso `Microsoft.ServiceFabric/clusters`, cambie `managementEndpoint` al FQDN de DNS de la dirección IP estática. Si está usando un clúster seguro, asegúrese de cambiar *http://* a *https://* . (Tenga en cuenta que este paso se aplica solo a los clústeres de Service Fabric. Si está empleando un conjunto de escalado de máquinas virtuales, omita este paso).

    ```json
                    "fabricSettings": [],
                    /*"managementEndpoint": "[concat('http://',reference(concat(parameters('lbIPName'),'-','0')).dnsSettings.fqdn,':',parameters('nt0fabricHttpGatewayPort'))]",*/
                    "managementEndpoint": "[concat('http://',parameters('existingStaticIPDnsFQDN'),':',parameters('nt0fabricHttpGatewayPort'))]",
    ```

8. Implemente la plantilla:

    ```powershell
    New-AzResourceGroup -Name sfnetworkingstaticip -Location westus

    $staticip = Get-AzPublicIpAddress -Name staticIP1 -ResourceGroupName ExistingRG

    $staticip

    New-AzResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkingstaticip -TemplateFile C:\SFSamples\Final\template\_staticip.json -existingStaticIPResourceGroup $staticip.ResourceGroupName -existingStaticIPName $staticip.Name -existingStaticIPDnsFQDN $staticip.DnsSettings.Fqdn
    ```

Después de la implementación, puede ver que el equilibrador de carga está enlazado a la dirección IP estática pública del otro grupo de recursos. El punto de conexión del cliente de Service Fabric y [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) apuntan al FQDN de DNS de la dirección IP estática.

<a id="internallb"></a>
## <a name="internal-only-load-balancer"></a>Equilibrador de carga solo interno

Este escenario reemplaza el equilibrador de carga externo en la plantilla predeterminada de Service Fabric por un equilibrador de carga solo interno. Consulte [anteriormente en este artículo](#allowing-the-service-fabric-resource-provider-to-query-your-cluster) para conocer las implicaciones en Azure Portal y en el proveedor de recursos de Service Fabric.

1. Elimine el parámetro `dnsName`. (No es necesario).

    ```json
    /*
    "dnsName": {
        "type": "string"
    },
    */
    ```

2. Si lo desea, puede agregar un parámetro de dirección IP estática, si usa el método de asignación estático. Si usa un método de asignación dinámico, no es necesario realizar este paso.

    ```json
            "internalLBAddress": {
                "type": "string",
                "defaultValue": "10.0.0.250"
            }
    ```

3. Quite `Microsoft.Network/publicIPAddresses` de los recursos, para que Azure no cree una nueva dirección IP:

    ```json
    /*
    {
        "apiVersion": "[variables('publicIPApiVersion')]",
        "type": "Microsoft.Network/publicIPAddresses",
        "name": "[concat(parameters('lbIPName'),)'-', '0')]",
        "location": "[parameters('computeLocation')]",
        "properties": {
            "dnsSettings": {
                "domainNameLabel": "[parameters('dnsName')]"
            },
            "publicIPAllocationMethod": "Dynamic"        
        },
        "tags": {
            "resourceType": "Service Fabric",
            "clusterName": "[parameters('clusterName')]"
        }
    }, */
    ```

4. Elimine la dirección IP del atributo `dependsOn` de `Microsoft.Network/loadBalancers`, para que no dependa de crear una nueva dirección IP. Agregue el atributo `dependsOn` de la red virtual porque el equilibrador de carga ahora depende de la subred de la red virtual:

    ```json
                "apiVersion": "[variables('lbApiVersion')]",
                "type": "Microsoft.Network/loadBalancers",
                "name": "[concat('LB','-', parameters('clusterName'),'-',parameters('vmNodeType0Name'))]",
                "location": "[parameters('computeLocation')]",
                "dependsOn": [
                    /*"[concat('Microsoft.Network/publicIPAddresses/',concat(parameters('lbIPName'),'-','0'))]"*/
                    "[concat('Microsoft.Network/virtualNetworks/',parameters('virtualNetworkName'))]"
                ],
    ```

5. Cambie la configuración `frontendIPConfigurations` del equilibrador de carga para que pase de usar un `publicIPAddress` a usar una subred y `privateIPAddress`. `privateIPAddress` usa una dirección IP interna estática predefinida. Para utilizar una dirección IP dinámica, quite el elemento `privateIPAddress` y, a continuación, cambie `privateIPAllocationMethod` a **Dinámico**.

    ```json
                "frontendIPConfigurations": [
                        {
                            "name": "LoadBalancerIPConfig",
                            "properties": {
                                /*
                                "publicIPAddress": {
                                    "id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(parameters('lbIPName'),'-','0'))]"
                                } */
                                "subnet" :{
                                    "id": "[variables('subnet0Ref')]"
                                },
                                "privateIPAddress": "[parameters('internalLBAddress')]",
                                "privateIPAllocationMethod": "Static"
                            }
                        }
                    ],
    ```

6. En el recurso `Microsoft.ServiceFabric/clusters`, cambie `managementEndpoint` para que apunte a la dirección del equilibrador de carga interno. Si usa un clúster seguro, asegúrese de cambiar *http://* a *https://* . (Tenga en cuenta que este paso se aplica solo a los clústeres de Service Fabric. Si está empleando un conjunto de escalado de máquinas virtuales, omita este paso).

    ```json
                    "fabricSettings": [],
                    /*"managementEndpoint": "[concat('http://',reference(concat(parameters('lbIPName'),'-','0')).dnsSettings.fqdn,':',parameters('nt0fabricHttpGatewayPort'))]",*/
                    "managementEndpoint": "[concat('http://',reference(variables('lbID0')).frontEndIPConfigurations[0].properties.privateIPAddress,':',parameters('nt0fabricHttpGatewayPort'))]",
    ```

7. Implemente la plantilla:

    ```powershell
    New-AzResourceGroup -Name sfnetworkinginternallb -Location westus

    New-AzResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkinginternallb -TemplateFile C:\SFSamples\Final\template\_internalonlyLB.json
    ```

Después de la implementación, el equilibrador de carga usará la dirección IP privada estática 10.0.0.250. Si tiene otra máquina en esa misma red virtual puede ir al punto de conexión interno de [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md). Tenga en cuenta que se conecta a uno de los nodos detrás del equilibrador de carga.

<a id="internalexternallb"></a>
## <a name="internal-and-external-load-balancer"></a>Equilibrador de carga interno y externo

En este escenario, comienza por el equilibrador de carga externo de un solo tipo de nodo y se agrega un equilibrador de carga interno adicional al mismo tipo de nodo. Un puerto de back-end conectado a un grupo de direcciones de back-end solo se puede asignar a un único equilibrador de carga. Elija qué equilibrador de carga tendrá los puertos de la aplicación y cuál tendrá los puntos de conexión de administración (puertos 19000 y 19080). Si pone los puntos de conexión de administración en el equilibrador de carga interno, tenga en cuenta las restricciones del proveedor de recursos de Service Fabric descritas [anteriormente en este artículo](#allowing-the-service-fabric-resource-provider-to-query-your-cluster). En el ejemplo que estamos utilizando, los puntos de conexión de administración permanecen en el equilibrador de carga externo. También puede agregar un puerto de aplicación 80 y colocarlo en el equilibrador de carga interno.

En un clúster de dos tipos de nodo, un tipo de nodo está en el equilibrador de carga externo. El otro estará en el equilibrador de carga interno. Para usar un clúster de dos tipos de nodo, en la plantilla de dos tipos de nodo creada en el portal (que incluye dos equilibradores de carga), cambie el segundo equilibrador de carga a un equilibrador de carga interno. Para más información, consulte la sección [Equilibrador de carga solo interno](#internallb).

1. Agregue el parámetro de dirección IP del equilibrador de carga interno estático. (Para ver las notas relativas al uso de una dirección IP dinámica, consulte las secciones anteriores de este artículo).

    ```json
            "internalLBAddress": {
                "type": "string",
                "defaultValue": "10.0.0.250"
            }
    ```

2. Agregue un parámetro de puerto 80 de aplicación.

3. Para agregar las versiones internas de las variables de red existentes, cópielas y péguelas y agregue "-Int" al nombre:

    ```json
    /* Add internal load balancer networking variables */
            "lbID0-Int": "[resourceId('Microsoft.Network/loadBalancers', concat('LB','-', parameters('clusterName'),'-',parameters('vmNodeType0Name'), '-Internal'))]",
            "lbIPConfig0-Int": "[concat(variables('lbID0-Int'),'/frontendIPConfigurations/LoadBalancerIPConfig')]",
            "lbPoolID0-Int": "[concat(variables('lbID0-Int'),'/backendAddressPools/LoadBalancerBEAddressPool')]",
            "lbProbeID0-Int": "[concat(variables('lbID0-Int'),'/probes/FabricGatewayProbe')]",
            "lbHttpProbeID0-Int": "[concat(variables('lbID0-Int'),'/probes/FabricHttpGatewayProbe')]",
            "lbNatPoolID0-Int": "[concat(variables('lbID0-Int'),'/inboundNatPools/LoadBalancerBEAddressNatPool')]",
            /* Internal load balancer networking variables end */
    ```

4. Si ha empezado con la plantilla generada mediante el portal que usa un puerto 80 de aplicación, la plantilla del portal predeterminada agrega AppPort1 (puerto 80) en el equilibrador de carga externo. En este caso, elimine AppPort1 de los sondeos y del elemento `loadBalancingRules` del equilibrador de carga externo para que pueda agregarlo al equilibrador de carga interno:

    ```json
    "loadBalancingRules": [
        {
            "name": "LBHttpRule",
            "properties":{
                "backendAddressPool": {
                    "id": "[variables('lbPoolID0')]"
                },
                "backendPort": "[parameters('nt0fabricHttpGatewayPort')]",
                "enableFloatingIP": "false",
                "frontendIPConfiguration": {
                    "id": "[variables('lbIPConfig0')]"            
                },
                "frontendPort": "[parameters('nt0fabricHttpGatewayPort')]",
                "idleTimeoutInMinutes": "5",
                "probe": {
                    "id": "[variables('lbHttpProbeID0')]"
                },
                "protocol": "tcp"
            }
        } /* Remove AppPort1 from the external load balancer.
        {
            "name": "AppPortLBRule1",
            "properties": {
                "backendAddressPool": {
                    "id": "[variables('lbPoolID0')]"
                },
                "backendPort": "[parameters('loadBalancedAppPort1')]",
                "enableFloatingIP": "false",
                "frontendIPConfiguration": {
                    "id": "[variables('lbIPConfig0')]"            
                },
                "frontendPort": "[parameters('loadBalancedAppPort1')]",
                "idleTimeoutInMinutes": "5",
                "probe": {
                    "id": "[concate(variables('lbID0'), '/probes/AppPortProbe1')]"
                },
                "protocol": "tcp"
            }
        }*/

    ],
    "probes": [
        {
            "name": "FabricGatewayProbe",
            "properties": {
                "intervalInSeconds": 5,
                "numberOfProbes": 2,
                "port": "[parameters('nt0fabricTcpGatewayPort')]",
                "protocol": "tcp"
            }
        },
        {
            "name": "FabricHttpGatewayProbe",
            "properties": {
                "intervalInSeconds": 5,
                "numberOfProbes": 2,
                "port": "[parameters('nt0fabricHttpGatewayPort')]",
                "protocol": "tcp"
            }
        } /* Remove AppPort1 from the external load balancer.
        {
            "name": "AppPortProbe1",
            "properties": {
                "intervalInSeconds": 5,
                "numberOfProbes": 2,
                "port": "[parameters('loadBalancedAppPort1')]",
                "protocol": "tcp"
            }
        } */

    ],
    "inboundNatPools": [
    ```

5. Agregue un segundo recurso `Microsoft.Network/loadBalancers`. Se parece al equilibrador de carga interno creado en la sección [Equilibrador de carga solo interno](#internallb), pero usa las variables "-Int" del equilibrador de carga e implementa únicamente el puerto 80 de aplicación. Este procedimiento quita también `inboundNatPools`, para conservar los puntos de conexión RDP en el equilibrador de carga público. Si desea acceder mediante RDP en el equilibrador de carga interno, mueva `inboundNatPools` del equilibrador de carga externo a este interno:

    ```json
            /* Add a second load balancer, configured with a static privateIPAddress and the "-Int" load balancer variables. */
            {
                "apiVersion": "[variables('lbApiVersion')]",
                "type": "Microsoft.Network/loadBalancers",
                /* Add "-Internal" to the name. */
                "name": "[concat('LB','-', parameters('clusterName'),'-',parameters('vmNodeType0Name'), '-Internal')]",
                "location": "[parameters('computeLocation')]",
                "dependsOn": [
                    /* Remove public IP dependsOn, add vnet dependsOn
                    "[concat('Microsoft.Network/publicIPAddresses/',concat(parameters('lbIPName'),'-','0'))]"
                    */
                    "[concat('Microsoft.Network/virtualNetworks/',parameters('virtualNetworkName'))]"
                ],
                "properties": {
                    "frontendIPConfigurations": [
                        {
                            "name": "LoadBalancerIPConfig",
                            "properties": {
                                /* Switch from Public to Private IP address
                                */
                                "publicIPAddress": {
                                    "id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(parameters('lbIPName'),'-','0'))]"
                                }
                                */
                                "subnet" :{
                                    "id": "[variables('subnet0Ref')]"
                                },
                                "privateIPAddress": "[parameters('internalLBAddress')]",
                                "privateIPAllocationMethod": "Static"
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
                        /* Add the AppPort rule. Be sure to reference the "-Int" versions of backendAddressPool, frontendIPConfiguration, and the probe variables. */
                        {
                            "name": "AppPortLBRule1",
                            "properties": {
                                "backendAddressPool": {
                                    "id": "[variables('lbPoolID0-Int')]"
                                },
                                "backendPort": "[parameters('loadBalancedAppPort1')]",
                                "enableFloatingIP": "false",
                                "frontendIPConfiguration": {
                                    "id": "[variables('lbIPConfig0-Int')]"
                                },
                                "frontendPort": "[parameters('loadBalancedAppPort1')]",
                                "idleTimeoutInMinutes": "5",
                                "probe": {
                                    "id": "[concat(variables('lbID0-Int'),'/probes/AppPortProbe1')]"
                                },
                                "protocol": "tcp"
                            }
                        }
                    ],
                    "probes": [
                    /* Add the probe for the app port. */
                    {
                            "name": "AppPortProbe1",
                            "properties": {
                                "intervalInSeconds": 5,
                                "numberOfProbes": 2,
                                "port": "[parameters('loadBalancedAppPort1')]",
                                "protocol": "tcp"
                            }
                        }
                    ],
                    "inboundNatPools": [
                    ]
                },
                "tags": {
                    "resourceType": "Service Fabric",
                    "clusterName": "[parameters('clusterName')]"
                }
            },
    ```

6. En `networkProfile` para el recurso `Microsoft.Compute/virtualMachineScaleSets`, agregue el grupo de direcciones de back-end interno:

    ```json
    "loadBalancerBackendAddressPools": [
                                                        {
                                                            "id": "[variables('lbPoolID0')]"
                                                        },
                                                        {
                                                            /* Add internal BE pool */
                                                            "id": "[variables('lbPoolID0-Int')]"
                                                        }
    ],
    ```

7. Implemente la plantilla:

    ```powershell
    New-AzResourceGroup -Name sfnetworkinginternalexternallb -Location westus

    New-AzResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkinginternalexternallb -TemplateFile C:\SFSamples\Final\template\_internalexternalLB.json
    ```

Después de la implementación, puede ver dos de equilibradores de carga en el grupo de recursos. Si examina los equilibradores de carga, puede ver la dirección IP pública y los puntos de conexión de administración (puertos 19000 y 19080) asignados a la dirección IP pública. También puede ver la dirección IP interna estática y el punto de conexión de la aplicación (puerto 80) asignados al equilibrador de carga interno. Ambos equilibradores de carga usan el mismo grupo de back-end de conjunto de escalado de máquinas virtuales.

## <a name="notes-for-production-workloads"></a>Notas para cargas de trabajo de producción

Las plantillas de GitHub anteriores se han diseñado para trabajar con la SKU predeterminada para Azure Standard Load Balancer (SLB), la SKU básica. Este SLB no dispone de Acuerdo de Nivel de Servicio, de modo que la SKU estándar debe usarse para las cargas de trabajo de producción. Para obtener más información al respecto, consulte la [introducción a Azure Standard Load Balancer](../load-balancer/load-balancer-overview.md). Cualquier clúster de Service Fabric que use la SKU estándar para SLB deberá asegurarse de que cada tipo de nodo tiene una regla que permite el tráfico saliente en el puerto 443. Esto es necesario para completar la configuración del clúster, produciéndose un error en cualquier implementación sin esta regla. En el ejemplo anterior de un equilibrador de carga "solo externo", debe agregarse un equilibrador de carga externo adicional a la plantilla con una regla que permite el tráfico saliente para el puerto 443.

## <a name="next-steps"></a>Pasos siguientes
[Creación de un clúster](service-fabric-cluster-creation-via-arm.md)

Después de la implementación, puede ver dos de equilibradores de carga en el grupo de recursos. Si examina los equilibradores de carga, puede ver la dirección IP pública y los puntos de conexión de administración (puertos 19000 y 19080) asignados a la dirección IP pública. También puede ver la dirección IP interna estática y el punto de conexión de la aplicación (puerto 80) asignados al equilibrador de carga interno. Ambos equilibradores de carga usan el mismo grupo de back-end de conjunto de escalado de máquinas virtuales.
