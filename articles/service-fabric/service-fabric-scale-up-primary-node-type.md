---
title: Escalado vertical de un tipo de nodo principal de Azure Service Fabric
description: Aprenda a escalar un clúster de Service Fabric mediante la adición de un tipo de nodo.
ms.topic: article
ms.date: 08/06/2020
ms.author: pepogors
ms.openlocfilehash: a18a40cc9e467b089ea9d6be3d0ca81a21d2c474
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89228722"
---
# <a name="scale-up-a-service-fabric-cluster-primary-node-type-by-adding-a-node-type"></a>Escalado vertical del tipo de nodo principal de un clúster de Service Fabric mediante la adición de un tipo de nodo
En este artículo, se explica cómo escalar verticalmente un tipo de nodo principal de un clúster Service Fabric agregando un tipo de nodo adicional al clúster. Un clúster de Service Fabric es un conjunto de máquinas físicas o virtuales conectadas a la red, en las que se implementan y administran los microservicios. Un equipo o máquina virtual que forma parte de un clúster se denomina nodo. Los conjuntos de escalado de máquinas virtuales son un recurso de proceso de Azure que se puede usar para implementar y administrar una colección de máquinas virtuales de forma conjunta. Cada tipo de nodo que se define en un clúster de Azure está [configurado como un conjunto de escalado independiente](service-fabric-cluster-nodetypes.md). Cada tipo de nodo, a continuación, se puede administrar por separado.

Aquí se pueden encontrar las plantillas de ejemplo del siguiente tutorial: [Ejemplos de escalado del tipo de nodo principal de Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/Primary-NodeType-Scaling-Sample)

> [!WARNING]
> No intente un procedimiento de escalado vertical de tipo de nodo principal si el estado del clúster es incorrecto, ya que esto solo desestabilizará el clúster.
>

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="process-to-upgrade-the-size-and-operating-system-of-the-primary-node-type"></a>Proceso de actualización del tamaño y sistema operativo del tipo de nodo principal
Este es el proceso para actualizar el tamaño de máquina virtual y el sistema operativo de las máquinas virtuales del tipo de nodo principal.  Después de la actualización, el tamaño de las máquinas virtuales del tipo de nodo principal es D4_V2 Estándar y en ellas se ejecuta Windows Server 2019 Datacenter con Containers.

> [!WARNING]
> Antes de intentar este procedimiento en un clúster de producción, se recomienda que revise atentamente las plantillas de ejemplo y compruebe el proceso en un clúster de prueba. También es posible que el clúster no esté disponible durante un breve período. 

### <a name="deploy-the-initial-service-fabric-cluster"></a>Implementación inicial del clúster de Service Fabric 
Si desea seguir con el ejemplo, implemente el clúster inicial con un solo tipo de nodo principal y un conjunto de escalado único con la plantilla de [clúster inicial de Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/Primary-NodeType-Scaling-Sample/AzureDeploy-1.json). Este paso se puede omitir este paso si hay un clúster de Service Fabric existente implementado. 

1. Inicio de sesión en la cuenta de Azure 
```powershell
# sign in to your Azure account and select your subscription
Login-AzAccount -SubscriptionId "<your subscription ID>"
```
2. Cree un nuevo grupo de recursos. 
```powershell
# create a resource group for your cluster deployment
$resourceGroupName = "myResourceGroup"
$location = "WestUS"

New-AzResourceGroup `
    -Name $resourceGroupName `
    -Location $location
```
3. Rellene los valores de los parámetros en los archivos de plantilla. 
4. Implemente el clúster en el grupo de recursos creado en el paso 2. 
```powershell
# deploy the template files to the resource group created above
$templateFilePath = "C:\AzureDeploy-1.json"
$parameterFilePath = "C:\AzureDeploy.Parameters.json"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $templateFilePath `
    -TemplateParameterFile $parameterFilePath
```

### <a name="add-a-new-primary-node-type-to-the-cluster"></a>Incorporación de un nuevo tipo de nodo principal al clúster
> [!Note]
> Los recursos que se creen en los pasos siguientes se convertirán en el nuevo tipo de nodo principal en el clúster una vez que finalice la operación de escalado. Asegúrese de usar nombres únicos y diferentes de los recursos iniciales Subred, IP pública, Equilibrador de carga, Conjunto de escalado de máquinas virtuales y Tipo de nodo. 

Aquí puede encontrar una plantilla en la que se hayan dado todos los pasos: [Service Fabric: nuevo clúster de tipo de nodo](https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/Primary-NodeType-Scaling-Sample/AzureDeploy-2.json). Los siguientes pasos contienen fragmentos de código de recursos parciales que resaltan los cambios realizados en los nuevos recursos.  

1. Cree una subred en la red virtual existente.
```json
{
    "name": "[variables('subnet1Name')]",
    "properties": {
        "addressPrefix": "[variables('subnet1Prefix')]"
    }
}
```
2. Cree un recurso de IP pública con un valor de domainNameLabel único. 
```json
{
    "apiVersion": "[variables('publicIPApiVersion')]",
    "type": "Microsoft.Network/publicIPAddresses",
    "name": "[concat(variables('lbIPName'),'-',variables('vmNodeType1Name'))]",
    "location": "[variables('computeLocation')]",
    "properties": {
    "dnsSettings": {
        "domainNameLabel": "[concat(variables('dnsName'),'-','nt2')]"
    },
    "publicIPAllocationMethod": "Dynamic"
    },
    "tags": {
    "resourceType": "Service Fabric",
    "clusterName": "[parameters('clusterName')]"
    }
}
```
3. Cree un recurso Equilibrador de carga que dependa del recurso IP pública creado antes. 
```json
"dependsOn": [
    "[concat('Microsoft.Network/publicIPAddresses/',concat(variables('lbIPName'),'-',variables('vmNodeType1Name')))]"
]
```
4. Cree un conjunto de escalado de máquinas virtuales que use la nueva SKU de la máquina virtual y la SKU del sistema operativo que quiere escalar verticalmente. 

Referencia del tipo de nodo 
```json
"nodeTypeRef": "[variables('vmNodeType1Name')]"
```

SKU de la máquina virtual
```json
"sku": {
    "name": "[parameters('vmNodeType1Size')]",
    "capacity": "[parameters('nt1InstanceCount')]",
    "tier": "Standard"
}
```

SKU del sistema operativo 
```json
"imageReference": {
    "publisher": "[parameters('vmImagePublisher1')]",
    "offer": "[parameters('vmImageOffer1')]",
    "sku": "[parameters('vmImageSku1')]",
    "version": "[parameters('vmImageVersion1')]"
}
```

El siguiente fragmento de código es un ejemplo de un nuevo recurso de conjunto de escalado de máquinas virtuales que se usa para crear un nuevo tipo de nodo para un clúster de Service Fabric. Es aconsejable asegurarse de que incluye las extensiones adicionales necesarias para la carga de trabajo. 

```json
    {
      "apiVersion": "[variables('vmssApiVersion')]",
      "type": "Microsoft.Compute/virtualMachineScaleSets",
      "name": "[variables('vmNodeType1Name')]",
      "location": "[variables('computeLocation')]",
      "dependsOn": [
        "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]",
        "[concat('Microsoft.Network/loadBalancers/', concat('LB','-', parameters('clusterName'),'-',variables('vmNodeType1Name')))]",
        "[concat('Microsoft.Storage/storageAccounts/', variables('supportLogStorageAccountName'))]",
        "[concat('Microsoft.Storage/storageAccounts/', variables('applicationDiagnosticsStorageAccountName'))]"
      ],
      "properties": {
        "overprovision": "[variables('overProvision')]",
        "upgradePolicy": {
          "mode": "Automatic"
        },
        "virtualMachineProfile": {
          "extensionProfile": {
            "extensions": [
              {
                "name": "[concat('ServiceFabricNodeVmExt_',variables('vmNodeType1Name'))]",
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
                    "nodeTypeRef": "[variables('vmNodeType1Name')]",
                    "dataPath": "D:\\SvcFab",
                    "durabilityLevel": "Bronze",
                    "enableParallelJobs": true,
                    "nicPrefixOverride": "[variables('subnet1Prefix')]",
                    "certificate": {
                      "thumbprint": "[parameters('certificateThumbprint')]",
                      "x509StoreName": "[parameters('certificateStoreValue')]"
                    }
                  },
                  "typeHandlerVersion": "1.0"
                }
              }
            ]
          },
          "networkProfile": {
            "networkInterfaceConfigurations": [
              {
                "name": "[concat(variables('nicName'), '-1')]",
                "properties": {
                  "ipConfigurations": [
                    {
                      "name": "[concat(variables('nicName'),'-',1)]",
                      "properties": {
                        "loadBalancerBackendAddressPools": [
                          {
                            "id": "[variables('lbPoolID1')]"
                          }
                        ],
                        "loadBalancerInboundNatPools": [
                          {
                            "id": "[variables('lbNatPoolID1')]"
                          }
                        ],
                        "subnet": {
                          "id": "[variables('subnet1Ref')]"
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
            "computernamePrefix": "[variables('vmNodeType1Name')]",
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
              "publisher": "[parameters('vmImagePublisher1')]",
              "offer": "[parameters('vmImageOffer1')]",
              "sku": "[parameters('vmImageSku1')]",
              "version": "[parameters('vmImageVersion1')]"
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
      "sku": {
        "name": "[parameters('vmNodeType1Size')]",
        "capacity": "[parameters('nt1InstanceCount')]",
        "tier": "Standard"
      },
      "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
      }
    },

```

5. Agregue un nuevo tipo de nodo al clúster que haga referencia al conjunto de escalado de máquinas virtuales que se creó anteriormente. La propiedad **isPrimary** de este tipo de nodo se debe establecer en true. 
```json
"name": "[variables('vmNodeType1Name')]",
"applicationPorts": {
    "endPort": "[variables('nt0applicationEndPort')]",
    "startPort": "[variables('nt0applicationStartPort')]"
},
"clientConnectionEndpointPort": "[variables('nt0fabricTcpGatewayPort')]",
"durabilityLevel": "Bronze",
"ephemeralPorts": {
    "endPort": "[variables('nt0ephemeralEndPort')]",
    "startPort": "[variables('nt0ephemeralStartPort')]"
},
"httpGatewayEndpointPort": "[variables('nt0fabricHttpGatewayPort')]",
"isPrimary": true,
"reverseProxyEndpointPort": "[variables('nt0reverseProxyEndpointPort')]",
"vmInstanceCount": "[parameters('nt1InstanceCount')]"
```
6. Implemente la plantilla de Resource Manager actualizada. 
```powershell
# deploy the updated template files to the existing resource group
$templateFilePath = "C:\AzureDeploy-2.json"
$parameterFilePath = "C:\AzureDeploy.Parameters.json"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $templateFilePath `
    -TemplateParameterFile $parameterFilePath `
```

El clúster de Service Fabric tendrá dos tipos de nodos al finalizar la implementación. 

### <a name="remove-the-existing-node-type"></a>Eliminación del tipo de nodo existente 
Una vez que ha finalizado la implementación de los recursos, puede empezar a deshabilitar los nodos del tipo de nodo principal original. Como los nodos están deshabilitados, los servicios del sistema se migrarán al nuevo tipo de nodo principal que se había implementado en el paso anterior.

1. Establezca la propiedad del tipo de nodo principal del clúster de Service Fabric en false. 
```json
{
    "name": "[variables('vmNodeType0Name')]",
    "applicationPorts": {
        "endPort": "[variables('nt0applicationEndPort')]",
        "startPort": "[variables('nt0applicationStartPort')]"
    },
    "clientConnectionEndpointPort": "[variables('nt0fabricTcpGatewayPort')]",
    "durabilityLevel": "Bronze",
    "ephemeralPorts": {
        "endPort": "[variables('nt0ephemeralEndPort')]",
        "startPort": "[variables('nt0ephemeralStartPort')]"
    },
    "httpGatewayEndpointPort": "[variables('nt0fabricHttpGatewayPort')]",
    "isPrimary": false,
    "reverseProxyEndpointPort": "[variables('nt0reverseProxyEndpointPort')]",
    "vmInstanceCount": "[parameters('nt0InstanceCount')]"
}
```
2. Implemente la plantilla con la propiedad isPrimary actualizada en el tipo de nodo original. Aquí puede encontrar una plantilla con la marca principal establecida en false en el tipo de nodo original: [Service Fabric: tipo de nodo principal false](https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/Primary-NodeType-Scaling-Sample/AzureDeploy-3.json).

```powershell
# deploy the updated template files to the existing resource group
$templateFilePath = "C:\AzureDeploy-3.json"
$parameterFilePath = "C:\AzureDeploy.Parameters.json"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $templateFilePath `
    -TemplateParameterFile $parameterFilePath `
```

3. Deshabilite los nodos en el tipo de nodo 0. 
```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint $ClusterConnectionEndpoint `
    -KeepAliveIntervalInSec 10 `
    -X509Credential `
    -ServerCertThumbprint $thumb  `
    -FindType FindByThumbprint `
    -FindValue $thumb `
    -StoreLocation CurrentUser `
    -StoreName My 

Write-Host "Connected to cluster"


$nodeType = "nt1vm" # specify the name of node type
$nodes = Get-ServiceFabricNode 

Write-Host "Disabling nodes..."
foreach($node in $nodes)
{
  if ($node.NodeType -eq $nodeType)
  {
    $node.NodeName

    Disable-ServiceFabricNode -Intent RemoveNode -NodeName $node.NodeName -Force
  }
}
```
* Para lograr la durabilidad Bronze, espere a que todos los nodos lleguen al estado Deshabilitado.
* En el caso de la durabilidad Silver o Gold, algunos nodos pasarán a Deshabilitado y el resto estará en estado Deshabilitando. Compruebe la pestaña de detalles de los nodos en estado Deshabilitando. Si todos los nodos están detenidos para garantizar el cuórum de las particiones del servicio de infraestructura, es seguro continuar.

> [!Note]
> Este paso puede tardar un tiempo en completarse. 

4. Detenga los datos en el tipo de nodo 0. 
```powershell
foreach($node in $nodes)
{
  if ($node.NodeType -eq $nodeType)
  {
    $node.NodeName

    Start-ServiceFabricNodeTransition -Stop -OperationId (New-Guid) -NodeInstanceId $node.NodeInstanceId -NodeName $node.NodeName -StopDurationInSeconds 10000
  }
}
```
5. Desasignación de nodos en el conjunto de escalado de máquinas virtuales original 
```powershell
$scaleSetName="nt1vm"
$scaleSetResourceType="Microsoft.Compute/virtualMachineScaleSets"

Remove-AzResource -ResourceName $scaleSetName -ResourceType $scaleSetResourceType -ResourceGroupName $resourceGroupName -Force
```
> [!Note]
> Los pasos 6 y 7 son opcionales si ya usa una IP pública de la SKU Estándar y un equilibrador de carga de la SKU Estándar. En este caso puede tener varios conjuntos de escalado de máquinas virtuales o tipos de nodos en el mismo equilibrador de carga. 

6. Ahora puede eliminar los recursos Equilibrador de carga e IP originales. En este paso también actualizará el nombre DNS. 

```powershell
$lbname="LB-cluster-name-nt1vm"
$lbResourceType="Microsoft.Network/loadBalancers"
$ipResourceType="Microsoft.Network/publicIPAddresses"
$oldPublicIpName="PublicIP-LB-FE-nt1vm"
$newPublicIpName="PublicIP-LB-FE-nt2vm"

$oldprimaryPublicIP = Get-AzPublicIpAddress -Name $oldPublicIpName  -ResourceGroupName $resourceGroupName
$primaryDNSName = $oldprimaryPublicIP.DnsSettings.DomainNameLabel
$primaryDNSFqdn = $oldprimaryPublicIP.DnsSettings.Fqdn

Remove-AzResource -ResourceName $lbname -ResourceType $lbResourceType -ResourceGroupName $resourceGroupName -Force
Remove-AzResource -ResourceName $oldPublicIpName -ResourceType $ipResourceType -ResourceGroupName $resourceGroupName -Force

$PublicIP = Get-AzPublicIpAddress -Name $newPublicIpName  -ResourceGroupName $resourceGroupName
$PublicIP.DnsSettings.DomainNameLabel = $primaryDNSName
$PublicIP.DnsSettings.Fqdn = $primaryDNSFqdn
Set-AzPublicIpAddress -PublicIpAddress $PublicIP
``` 

7. Actualice el punto de conexión de administración en el clúster para que haga referencia a la nueva IP. 
```json
  "managementEndpoint": "[concat('https://',reference(concat(variables('lbIPName'),'-',variables('vmNodeType1Name'))).dnsSettings.fqdn,':',variables('nt0fabricHttpGatewayPort'))]",
```
8. Elimine el estado del nodo del tipo de nodo 0.
```powershell
foreach($node in $nodes)
{
  if ($node.NodeType -eq $nodeType)
  {
    $node.NodeName

    Remove-ServiceFabricNodeState -NodeName $node.NodeName -Force
  }
}
```
9. Elimine la referencia del tipo de nodo original del recurso Service Fabric en la plantilla de Resource Manager. 
```json
"name": "[variables('vmNodeType0Name')]",
"applicationPorts": {
    "endPort": "[variables('nt0applicationEndPort')]",
    "startPort": "[variables('nt0applicationStartPort')]"
},
"clientConnectionEndpointPort": "[variables('nt0fabricTcpGatewayPort')]",
"durabilityLevel": "Bronze",
"ephemeralPorts": {
    "endPort": "[variables('nt0ephemeralEndPort')]",
    "startPort": "[variables('nt0ephemeralStartPort')]"
},
"httpGatewayEndpointPort": "[variables('nt0fabricHttpGatewayPort')]",
"isPrimary": true,
"reverseProxyEndpointPort": "[variables('nt0reverseProxyEndpointPort')]",
"vmInstanceCount": "[parameters('nt0InstanceCount')]"
```
Solo en el caso de los clústeres de durabilidad Silver y superior, actualice el recurso de clúster en la plantilla y configure directivas de mantenimiento para omitir el estado de la aplicación fabric:/System; para ello, agregue applicationDeltaHealthPolicies en las propiedades del recurso de clúster como se indica a continuación. La directiva siguiente debe omitir los errores existentes, pero no permitir nuevos errores de estado.
```json
"upgradeDescription":  
{ 
 "forceRestart": false, 
 "upgradeReplicaSetCheckTimeout": "10675199.02:48:05.4775807", 
 "healthCheckWaitDuration": "00:05:00", 
 "healthCheckStableDuration": "00:05:00", 
 "healthCheckRetryTimeout": "00:45:00", 
 "upgradeTimeout": "12:00:00", 
 "upgradeDomainTimeout": "02:00:00", 
 "healthPolicy": { 
   "maxPercentUnhealthyNodes": 100, 
   "maxPercentUnhealthyApplications": 100 
 }, 
 "deltaHealthPolicy":  
 { 
   "maxPercentDeltaUnhealthyNodes": 0, 
   "maxPercentUpgradeDomainDeltaUnhealthyNodes": 0, 
   "maxPercentDeltaUnhealthyApplications": 0, 
   "applicationDeltaHealthPolicies":  
   { 
       "fabric:/System":  
       { 
           "defaultServiceTypeDeltaHealthPolicy":  
           { 
                   "maxPercentDeltaUnhealthyServices": 0 
           } 
       } 
   } 
 } 
}
```
10. Elimine de la plantilla de Resource Manager el resto de recursos que tengan relación con el tipo de nodo original. En [Service Fabric: nuevo clúster de tipo de nodo](https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/Primary-NodeType-Scaling-Sample/AzureDeploy-4.json), encontrará una plantilla de la que se han eliminado todos estos recursos originales.

11. Implemente la plantilla de Azure Resource Manager modificada. ** Este paso tardará un rato, habitualmente puede llegar a las dos horas. Esta actualización cambiará la configuración de InfrastructureService; por lo que es necesario reiniciar el nodo. En este caso, forceRestart se omite. El parámetro upgradeReplicaSetCheckTimeout especifica el tiempo máximo que Service Fabric espera a que una partición pase a un estado seguro, si aún no lo está. Una vez que se superan las comprobaciones de seguridad de todas las particiones de un nodo, Service Fabric continúa con la actualización en ese nodo. El valor del parámetro upgradeTimeout puede reducirse a 6 horas, pero debe usarse la seguridad máxima, 12 horas.
Luego, valide que el recurso Service Fabric aparece preparado en Azure Portal. 

```powershell
# deploy the updated template files to the existing resource group
$templateFilePath = "C:\AzureDeploy-4.json"
$parameterFilePath = "C:\AzureDeploy.Parameters.json"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $templateFilePath `
    -TemplateParameterFile $parameterFilePath `
```

Ahora se ha actualizado el tipo de nodo principal del clúster. Verifique que las aplicaciones implementadas funcionen correctamente y que el estado del clúster sea correcto.

## <a name="next-steps"></a>Pasos siguientes
* Obtenga información sobre la [incorporación de un tipo de nodo a un clúster](virtual-machine-scale-set-scale-node-type-scale-out.md)
* Obtenga información sobre [escalabilidad de aplicaciones](service-fabric-concepts-scalability.md).
* [Escalado horizontal de un clúster de Azure](service-fabric-tutorial-scale-cluster.md).
* [Escalado de un clúster de Azure mediante programación](service-fabric-cluster-programmatic-scaling.md) con el SDK de proceso de Azure.
* [Escalado o reducción horizontal de un clúster independiente](service-fabric-cluster-windows-server-add-remove-nodes.md)
