---
title: Actualización de nodos de clúster para usar discos administrados de Azure
description: Aquí se muestra cómo actualizar un clúster de Service Fabric existente para usar Azure Managed Disks con poco o ningún tiempo de inactividad del clúster.
ms.topic: how-to
ms.date: 4/07/2020
ms.openlocfilehash: 10863626945483e21aa264e2b05e94a6f08a22f6
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87542876"
---
# <a name="upgrade-cluster-nodes-to-use-azure-managed-disks"></a>Actualización de nodos de clúster para usar Azure Managed Disks

[Azure Managed Disks](../virtual-machines/windows/managed-disks-overview.md) es la nueva oferta de almacenamiento en disco recomendada que se puede usar con Azure Virtual Machines para almacenar datos de forma persistente. Puede mejorar la resistencia de las cargas de trabajo de Service Fabric actualizando los conjuntos de escalado de máquinas virtuales que subyacen a los tipos de nodo para usar Managed Disks. Aquí se muestra cómo actualizar un clúster de Service Fabric existente para usar Azure Managed Disks con poco o ningún tiempo de inactividad del clúster.

La estrategia general para actualizar un nodo de clúster de Service Fabric para usar discos administrados es:

1. Implemente otro conjunto de escalado de máquinas virtuales duplicado de ese tipo de nodo, pero con el objeto [managedDisk](/azure/templates/microsoft.compute/2019-07-01/virtualmachinescalesets/virtualmachines#ManagedDiskParameters) agregado a la sección `osDisk` de la plantilla de implementación del conjunto de escalado de máquinas virtuales. El nuevo conjunto de escalado debe enlazarse con el mismo equilibrador de carga o dirección IP que el original, de modo que los clientes no experimenten una interrupción del servicio durante la migración.

2. Una vez que los conjuntos de escalado original y actualizado se ejecuten en paralelo, deshabilite las instancias de nodo originales una a una para que los servicios del sistema (o las réplicas de servicios con estado) migren al nuevo conjunto de escalado.

3. Compruebe que el clúster y los nuevos nodos tengan un estado correcto y, a continuación, quite el conjunto de escalado original y el estado del nodo de los nodos eliminados.

Este artículo le guiará a lo largo de los pasos necesarios para actualizar el tipo de nodo principal de un clúster de ejemplo para usar discos administrados, al tiempo que evita cualquier tiempo de inactividad del clúster (vea la nota siguiente). El estado inicial del clúster de prueba de ejemplo consta de un tipo de nodo de [durabilidad Silver](service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster), respaldado por un solo conjunto de escalado con cinco nodos.

> [!CAUTION]
> Experimentará una interrupción con este procedimiento solo si tiene dependencias en el DNS del clúster (por ejemplo, al acceder a [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md)). El [procedimiento recomendado para los servicios de front-end](/azure/architecture/microservices/design/gateway) en el nivel de arquitectura es tener algún tipo de [equilibrador de carga](/azure/architecture/guide/technology-choices/load-balancing-overview) delante de los tipos de nodo para que el intercambio de nodos sea posible sin interrupción.

Estos son los [cmdlets y plantillas](https://github.com/microsoft/service-fabric-scripts-and-templates/tree/master/templates/nodetype-upgrade-no-outage) para Azure Resource Manager que usaremos para completar el escenario de actualización. Los cambios en la plantilla se explicarán en [Implementación de un conjunto de escalado actualizado para el tipo de nodo principal](#deploy-an-upgraded-scale-set-for-the-primary-node-type) a continuación.

## <a name="set-up-the-test-cluster"></a>Configuración del clúster de prueba

Configuremos el clúster de prueba inicial de Service Fabric. En primer lugar, [descargue](https://github.com/microsoft/service-fabric-scripts-and-templates/tree/master/templates/nodetype-upgrade-no-outage) las plantillas de muestra de Azure Resource Manager que usaremos para completar este escenario.

A continuación, inicie sesión en la cuenta de Azure.

```powershell
# Sign in to your Azure account
Login-AzAccount -SubscriptionId "<subscription ID>"
```

Los comandos siguientes le guiarán a lo largo de la generación de un nuevo certificado autofirmado y la implementación del clúster de prueba. Si ya tiene un certificado que le gustaría usar, vaya a [Uso de un certificado existente para implementar el clúster](#use-an-existing-certificate-to-deploy-the-cluster).

### <a name="generate-a-self-signed-certificate-and-deploy-the-cluster"></a>Generación de un certificado autofirmado e implementación del clúster

En primer lugar, asigne las variables que necesitará para la implementación del clúster de Service Fabric. Ajuste los valores de `resourceGroupName`, `certSubjectName`, `parameterFilePath` y `templateFilePath` para su entorno y cuenta específicos:

```powershell
# Assign deployment variables
$resourceGroupName = "sftestupgradegroup"
$certOutputFolder = "c:\certificates"
$certPassword = "Password!1" | ConvertTo-SecureString -AsPlainText -Force
$certSubjectName = "sftestupgrade.southcentralus.cloudapp.azure.com"
$templateFilePath = "C:\Initial-1NodeType-UnmanagedDisks.json"
$parameterFilePath = "C:\Initial-1NodeType-UnmanagedDisks.parameters.json"
```

> [!NOTE]
> Asegúrese de que la ubicación `certOutputFolder` existe en el equipo local antes de ejecutar el comando para implementar un nuevo clúster de Service Fabric.

A continuación, abra el archivo [*Initial-1NodeType-UnmanagedDisks.parameters.json*](https://github.com/erikadoyle/service-fabric-scripts-and-templates/blob/managed-disks/templates/nodetype-upgrade-no-outage/Initial-1NodeType-UnmanagedDisks.parameters.json) y ajuste los valores para `clusterName` y `dnsName` para que se correspondan con los valores dinámicos que establezca en PowerShell y guarde los cambios.

A continuación, implemente el clúster de prueba de Service Fabric:

```powershell
# Deploy the initial test cluster
New-AzServiceFabricCluster `
    -ResourceGroupName $resourceGroupName `
    -CertificateOutputFolder $certOutputFolder `
    -CertificatePassword $certPassword `
    -CertificateSubjectName $certSubjectName `
    -TemplateFile $templateFilePath `
    -ParameterFile $parameterFilePath
```

Una vez finalizada la implementación, busque el archivo *.pfx* (`$certPfx`) en la máquina local e impórtelo al almacén de certificados:

```powershell
cd c:\certificates
$certPfx = ".\sftestupgradegroup20200312121003.pfx"

Import-PfxCertificate `
     -FilePath $certPfx `
     -CertStoreLocation Cert:\CurrentUser\My `
     -Password (ConvertTo-SecureString Password!1 -AsPlainText -Force)
```

La operación devolverá la huella digital del certificado, que usará para [conectarse al nuevo clúster](#connect-to-the-new-cluster-and-check-health-status) y comprobar su estado de mantenimiento. (Omita la sección siguiente, que es un enfoque alternativo a la implementación del clúster).

### <a name="use-an-existing-certificate-to-deploy-the-cluster"></a>Uso de un certificado existente para implementar el clúster

También puede usar un certificado de Azure Key Vault existente para implementar el clúster de prueba. Para ello, necesitará [obtener referencias a su instancia de Key Vault](#obtain-your-key-vault-references) y huella digital de certificado.

```powershell
# Key Vault variables
$certUrlValue = "https://sftestupgradegroup.vault.azure.net/secrets/sftestupgradegroup20200309235308/dac0e7b7f9d4414984ccaa72bfb2ea39"
$sourceVaultValue = "/subscriptions/########-####-####-####-############/resourceGroups/sftestupgradegroup/providers/Microsoft.KeyVault/vaults/sftestupgradegroup"
$thumb = "BB796AA33BD9767E7DA27FE5182CF8FDEE714A70"
```

Abra el archivo [*Initial-1NodeType-UnmanagedDisks.parameters.json*](https://github.com/erikadoyle/service-fabric-scripts-and-templates/blob/managed-disks/templates/nodetype-upgrade-no-outage/Initial-1NodeType-UnmanagedDisks.parameters.json) y cambie los valores de `clusterName` y `dnsName` a un valor único.

Por último, designe un nombre de grupo de recursos para el clúster y establezca las ubicaciones `templateFilePath` y `parameterFilePath` de los archivos *Initial-1NodeType-UnmanagedDisks*:

> [!NOTE]
> El grupo de recursos designado ya debe existir y debe estar ubicado en la misma región que el Key Vault.

```powershell
# Deploy the new scale set (upgraded to use managed disks) into the primary node type.
$resourceGroupName = "sftestupgradegroup"
$templateFilePath = "C:\Upgrade-1NodeType-2ScaleSets-ManagedDisks.json"
$parameterFilePath = "C:\Upgrade-1NodeType-2ScaleSets-ManagedDisks.parameters.json"
```

Finalmente, ejecute el comando siguiente para implementar el clúster de prueba inicial:

```powershell
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $templateFilePath `
    -TemplateParameterFile $parameterFilePath `
    -CertificateThumbprint $thumb `
    -CertificateUrlValue $certUrlValue `
    -SourceVaultValue $sourceVaultValue `
    -Verbose
```

### <a name="connect-to-the-new-cluster-and-check-health-status"></a>Conexión al nuevo clúster y comprobación del estado de mantenimiento

Conéctese al clúster y asegúrese de que los cinco nodos tengan un estado correcto (reemplazando las variables `clusterName` y `thumb` del clúster):

```powershell
# Connect to the cluster
$clusterName = "sftestupgrade.southcentralus.cloudapp.azure.com:19000"
$thumb = "BB796AA33BD9767E7DA27FE5182CF8FDEE714A70"

Connect-ServiceFabricCluster `
    -ConnectionEndpoint $clusterName `
    -KeepAliveIntervalInSec 10 `
    -X509Credential `
    -ServerCertThumbprint $thumb  `
    -FindType FindByThumbprint `
    -FindValue $thumb `
    -StoreLocation CurrentUser `
    -StoreName My

# Check cluster health
Get-ServiceFabricClusterHealth
```

Con eso, estamos listos para comenzar el procedimiento de actualización.

## <a name="deploy-an-upgraded-scale-set-for-the-primary-node-type"></a>Implementación de un conjunto de escalado actualizado para el tipo de nodo principal

Para actualizar, o *escalar verticalmente*, un tipo de nodo, es necesario implementar una copia del conjunto de escalado de máquinas virtuales de ese tipo de nodo, que, por lo demás, es idéntico al conjunto de escalado original (incluida la referencia a los mismos elementos `nodeTypeRef`, `subnet` y `loadBalancerBackendAddressPools`), salvo que incluye la actualización o los cambios deseados y sus propios grupos de direcciones NAT de entrada y subred. Dado que estamos actualizando un tipo de nodo principal, el nuevo conjunto de escalado se marcará como principal (`isPrimary: true`), al igual que el conjunto de escalado original. (Para las actualizaciones de tipos de nodos distintos del principal, simplemente, omita este paso).

Para mayor comodidad, ya se han aplicado los cambios necesarios automáticamente en los archivos de [plantilla](https://github.com/erikadoyle/service-fabric-scripts-and-templates/blob/managed-disks/templates/nodetype-upgrade-no-outage/Upgrade-1NodeType-2ScaleSets-ManagedDisks.json) y [parámetros](https://github.com/erikadoyle/service-fabric-scripts-and-templates/blob/managed-disks/templates/nodetype-upgrade-no-outage/Upgrade-1NodeType-2ScaleSets-ManagedDisks.parameters.json) de *Upgrade-1NodeType-2ScaleSets-ManagedDisks*.

En las secciones siguientes se explican los cambios de plantilla con más detalle. Si lo prefiere, puede omitir la explicación y continuar con [el siguiente paso del procedimiento de actualización](#obtain-your-key-vault-references).

### <a name="update-the-cluster-template-with-the-upgraded-scale-set"></a>Actualización de la plantilla de clúster con el conjunto de escalado actualizado

Estas son las modificaciones sección por sección de la plantilla de implementación del clúster original para agregar un conjunto de escalado actualizado para el tipo de nodo principal.

#### <a name="parameters"></a>Parámetros

Agregue un parámetro para el nombre de instancia del nuevo conjunto de escalado. Tenga en cuenta que `vmNodeType1Name` es único para el nuevo conjunto de escalado, mientras que los valores de tamaño y recuento son idénticos a los del conjunto de escalado original.

**Archivo de plantilla**

```json
"vmNodeType1Name": {
    "type": "string",
    "defaultValue": "NTvm2",
    "maxLength": 9
}
```

**Archivo de parámetros**

```json
"vmNodeType1Name": {
    "value": "NTvm2"
}
```

### <a name="variables"></a>variables

En la sección `variables` de la plantilla de implementación, agregue una entrada para el grupo de direcciones de NAT de entrada del nuevo conjunto de escalado.

**Archivo de plantilla**

```json
"lbNatPoolID1": "[concat(variables('lbID0'),'/inboundNatPools/LoadBalancerBEAddressNatPool1')]", 
```

### <a name="resources"></a>Recursos

En la sección *resources* de la plantilla de implementación, agregue el nuevo conjunto de escalado de máquinas virtuales, teniendo en cuenta lo siguiente:

* El nuevo conjunto de escalado hace referencia al nuevo tipo de nodo:

    ```json
    "nodeTypeRef": "[parameters('vmNodeType1Name')]",
    ```

* El nuevo conjunto de escalado hace referencia a la misma dirección y subred de back-end del equilibrador de carga que el original, pero utiliza un grupo de NAT de entrada de equilibrador de carga diferente:

   ```json
    "loadBalancerBackendAddressPools": [
        {
            "id": "[variables('lbPoolID0')]"
        }
    ],
    "loadBalancerInboundNatPools": [
        {
            "id": "[variables('lbNatPoolID1')]"
        }
    ],
    "subnet": {
        "id": "[variables('subnet0Ref')]"
    }
   ```

* Al igual que el conjunto de escalado original, el nuevo conjunto de escalado se marca como tipo de nodo principal. (Al actualizar tipos de nodo no principales, omita este cambio).

    ```json
    "isPrimary": true,
    ```

* A diferencia del conjunto de escalado original, el nuevo conjunto de escalado se actualiza para usar discos administrados.

    ```json
    "managedDisk": {
        "storageAccountType": "[parameters('storageAccountType')]"
    }
    ```

A continuación, agregue una entrada a la lista `nodeTypes` del recurso *Microsoft.ServiceFabric/clusters*. Utilice los mismos valores que la entrada del tipo de nodo original, excepto el `name`, que debe hacer referencia al nuevo tipo de nodo (*vmNodeType1Name*).

```json
"nodeTypes": [
    {
        "name": "[parameters('vmNodeType0Name')]",
        ...
    },
    {
        "name": "[parameters('vmNodeType1Name')]",
        "applicationPorts": {
            "endPort": "[parameters('nt0applicationEndPort')]",
            "startPort": "[parameters('nt0applicationStartPort')]"
        },
        "clientConnectionEndpointPort": "[parameters('nt0fabricTcpGatewayPort')]",
        "durabilityLevel": "Silver",
        "ephemeralPorts": {
            "endPort": "[parameters('nt0ephemeralEndPort')]",
            "startPort": "[parameters('nt0ephemeralStartPort')]"
        },
        "httpGatewayEndpointPort": "[parameters('nt0fabricHttpGatewayPort')]",
        "isPrimary": true,
        "vmInstanceCount": "[parameters('nt0InstanceCount')]"
    }
],
```

Una vez que haya implementado todos los cambios en los archivos de plantilla y de parámetros, continúe con la sección siguiente para adquirir las referencias de Key Vault e implementar las actualizaciones en el clúster.

### <a name="obtain-your-key-vault-references"></a>Obtener las referencias de Key Vault

Para implementar la configuración actualizada, primero debe obtener varias referencias al certificado de clúster almacenado en Key Vault. La forma más fácil de encontrar estos valores es desde Azure Portal. Necesitará:

* **La dirección URL de Key del certificado del clúster.** En la instancia de Key Vault de Azure Portal, seleccione **Certificados** > *el certificado que quiera* > **Identificador secreto**:

    ```powershell
    $certUrlValue="https://sftestupgradegroup.vault.azure.net/secrets/sftestupgradegroup20200309235308/dac0e7b7f9d4414984ccaa72bfb2ea39"
    ```

* **La huella digital del certificado del clúster.** (Probablemente ya lo tenga si se [conectó al clúster original](#connect-to-the-new-cluster-and-check-health-status) para comprobar su estado de mantenimiento). En la misma hoja de certificado (**Certificados** > *el certificado que quiera*) en Azure Portal, copie la **huella digital X.509 SHA-1 (en formato hexadecimal)** :

    ```powershell
    $thumb = "BB796AA33BD9767E7DA27FE5182CF8FDEE714A70"
    ```

* **Identificador de recurso de Key Vault.** En la instancia de Key Vault en Azure Portal, seleccione **Propiedades** > **Id. de recurso**:

    ```powershell
    $sourceVaultValue = "/subscriptions/########-####-####-####-############/resourceGroups/sftestupgradegroup/providers/Microsoft.KeyVault/vaults/sftestupgradegroup"
    ```

### <a name="deploy-the-updated-template"></a>Implementar la plantilla actualizada

Ajuste `parameterFilePath` y `templateFilePath` según sea necesario y, a continuación, ejecute el comando siguiente:

```powershell
# Deploy the new scale set (upgraded to use managed disks) into the primary node type.
$templateFilePath = "C:\Upgrade-1NodeType-2ScaleSets-ManagedDisks.json"
$parameterFilePath = "C:\Upgrade-1NodeType-2ScaleSets-ManagedDisks.parameters.json"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $templateFilePath `
    -TemplateParameterFile $parameterFilePath `
    -CertificateThumbprint $thumb `
    -CertificateUrlValue $certUrlValue `
    -SourceVaultValue $sourceVaultValue `
    -Verbose
```

Una vez finalizada la implementación, vuelva a comprobar el estado del clúster y asegúrese de que los diez nodos (cinco del conjunto de escalado original y otros cinco del nuevo) tengan un estado correcto.

```powershell
Get-ServiceFabricClusterHealth
```

## <a name="migrate-seed-nodes-to-the-new-scale-set"></a>Migración de nodos de inicialización al nuevo conjunto de escalado

Estamos preparados para empezar a deshabilitar los nodos del conjunto de escalado original. A medida que estos nodos se inhabilitan, los servicios del sistema y los nodos de inicialización se migran a las VM del nuevo conjunto de escalado porque también está marcado como tipo de nodo principal.

```powershell
# Disable the nodes in the original scale set.
$nodeNames = @("_NTvm1_0","_NTvm1_1","_NTvm1_2","_NTvm1_3","_NTvm1_4")

Write-Host "Disabling nodes..."
foreach($name in $nodeNames){
    Disable-ServiceFabricNode -NodeName $name -Intent RemoveNode -Force
}
```

Use Service Fabric Explorer para supervisar la migración de los nodos de inicialización al nuevo conjunto de escalado y la progresión de los nodos del conjunto de escalado original del estado *Desactivando* a *Deshabilitado*.

![Service Fabric Explorer con los estados de los nodos deshabilitados](./media/upgrade-managed-disks/service-fabric-explorer-node-status.png)

> [!NOTE]
> La operación de deshabilitación puede tardar algún tiempo en completarse en todos los nodos del conjunto de escalado original. Para garantizar la coherencia de los datos, solo se puede cambiar un nodo de inicialización a la vez. Cada cambio de nodo de inicialización requiere una actualización de clúster. Por lo tanto, para reemplazar un nodo de inicialización, se requieren dos actualizaciones de clúster (para la adición y eliminación de nodos). La actualización de los cinco nodos de inicialización en este escenario de muestra dará como resultado diez actualizaciones de clúster.

## <a name="remove-the-original-scale-set"></a>Quitar el conjunto de escalado original

Una vez completada la operación de deshabilitación, quite el conjunto de escalado.

```powershell
# Remove the original scale set
$scaleSetName = "NTvm1"

Remove-AzVmss `
    -ResourceGroupName $resourceGroupName `
    -VMScaleSetName $scaleSetName `
    -Force

Write-Host "Removed scale set $scaleSetName"
```

En Service Fabric Explorer, los nodos quitados (y, por tanto, el *estado de mantenimiento del clúster*) presentarán ahora un estado de *error*.

![Service Fabric Explorer con nodos deshabilitados en estado de error](./media/upgrade-managed-disks/service-fabric-explorer-disabled-nodes-error-state.png)

Quite los nodos obsoletos del clúster de Service Fabric para restaurar el estado de mantenimiento del clúster a *Correcto*.

```powershell
# Remove node states for the deleted scale set
foreach($name in $nodeNames){
    Remove-ServiceFabricNodeState -NodeName $name -TimeoutSec 300 -Force
    Write-Host "Removed node state for node $name"
}
```

![Service Fabric Explorer con los nodos inactivos con estado de error quitados](./media/upgrade-managed-disks/service-fabric-explorer-healthy-cluster.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a actualizar los conjuntos de escalado de máquinas virtuales de un clúster de Service Fabric para usar discos administrados y, a la vez, evitar interrupciones del servicio durante el proceso. Para obtener más información sobre temas relacionados, consulte los siguientes recursos.

Obtenga información sobre cómo:

* [Escalado vertical del tipo de nodo principal de un clúster de Service Fabric](service-fabric-scale-up-node-type.md)

* [Conversión de una plantilla de conjunto de escalado para usar discos administrados](../virtual-machine-scale-sets/virtual-machine-scale-sets-convert-template-to-md.md)

* [Eliminación de un tipo de nodo de Service Fabric](service-fabric-how-to-remove-node-type.md)

Consulte también:

* [Ejemplo: Actualización de nodos de clúster para usar Azure Managed Disks](https://github.com/microsoft/service-fabric-scripts-and-templates/tree/master/templates/nodetype-upgrade-no-outage)

* [Consideraciones sobre escalado vertical](service-fabric-best-practices-capacity-scaling.md#vertical-scaling-considerations)
