---
title: Exploración de los recursos de Azure
description: Aprenda a usar el lenguaje de consulta de Resource Graph para explorar los recursos y descubrir cómo se conectan.
ms.date: 08/10/2020
ms.topic: conceptual
ms.openlocfilehash: 2dcd27380cb67213c3c2c7a5776243b5e9a2e37f
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88056591"
---
# <a name="explore-your-azure-resources-with-resource-graph"></a>Exploración de recursos de Azure con el gráfico de recursos

El gráfico de recursos de Azure proporciona la capacidad de explorar y detectar los recursos de Azure rápidamente y a escala. Diseñado para respuestas rápidas, es una excelente manera de aprender sobre el entorno y también sobre las propiedades que componen los recursos de Azure.

## <a name="explore-virtual-machines"></a>Exploración de máquinas virtuales

Un recurso común en Azure es una máquina virtual. Como tipo de recurso, las máquinas virtuales tienen muchas propiedades que se pueden consultar. Cada propiedad ofrece una opción para filtrar o buscar exactamente el recurso que está buscando.

### <a name="virtual-machine-discovery"></a>Detección de la máquina virtual

Comencemos con una simple consulta para obtener una única máquina virtual de nuestro entorno y ver las propiedades devueltas.

```kusto
Resources
| where type =~ 'Microsoft.Compute/virtualMachines'
| limit 1
```

```azurecli-interactive
az graph query -q "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | limit 1"
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | limit 1" | ConvertTo-Json -Depth 100
```

> [!NOTE]
> El cmdlet `Search-AzGraph` de Azure PowerShell devuelve un objeto **PSCustomObject** de forma predeterminada. Para que la salida sea parecida a lo que devuelve la CLI de Azure, se usa el cmdlet `ConvertTo-Json`. El valor predeterminado de **Depth** es _2_. Si se establece en _100_ Se deberían convertir todos los niveles devueltos.

Los resultados de JSON tienen una estructura similar a la del ejemplo siguiente:

```json
[
  {
    "id": "/subscriptions/<subscriptionId>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/virtualMachines/ContosoVM1",
    "kind": "",
    "location": "westus2",
    "managedBy": "",
    "name": "ContosoVM1",
    "plan": {},
    "properties": {
      "hardwareProfile": {
        "vmSize": "Standard_B2s"
      },
      "networkProfile": {
        "networkInterfaces": [
          {
            "id": "/subscriptions/<subscriptionId>/MyResourceGroup/providers/Microsoft.Network/networkInterfaces/contosovm1535",
            "resourceGroup": "MyResourceGroup"
          }
        ]
      },
      "osProfile": {
        "adminUsername": "localAdmin",
        "computerName": "ContosoVM1",
        "secrets": [],
        "windowsConfiguration": {
          "enableAutomaticUpdates": true,
          "provisionVMAgent": true
        }
      },
      "provisioningState": "Succeeded",
      "storageProfile": {
        "dataDisks": [],
        "imageReference": {
          "offer": "WindowsServer",
          "publisher": "MicrosoftWindowsServer",
          "sku": "2016-Datacenter",
          "version": "latest"
        },
        "osDisk": {
          "caching": "ReadWrite",
          "createOption": "FromImage",
          "diskSizeGB": 127,
          "managedDisk": {
            "id": "/subscriptions/<subscriptionId>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/disks/ContosoVM1_OsDisk_1_9676b7e1b3c44e2cb672338ebe6f5166",
            "resourceGroup": "MyResourceGroup",
            "storageAccountType": "Premium_LRS"
          },
          "name": "ContosoVM1_OsDisk_1_9676b7e1b3c44e2cb672338ebe6f5166",
          "osType": "Windows"
        }
      },
      "vmId": "bbb9b451-6dc7-4117-bec5-c971eb1118c6"
    },
    "resourceGroup": "MyResourceGroup",
    "sku": {},
    "subscriptionId": "<subscriptionId>",
    "tags": {},
    "type": "microsoft.compute/virtualmachines"
  }
]
```

Las propiedades nos proporcionan información adicional sobre el propio recurso de la máquina virtual, desde SKU, sistema operativo, discos, etiquetas y el grupo de recursos y la suscripción a la que pertenece.

### <a name="virtual-machines-by-location"></a>Máquinas virtuales por ubicación

Tomando lo que hemos aprendido sobre el recurso de máquinas virtuales, vamos a usar la propiedad **location** para contar todas las máquinas virtuales por ubicación. Para actualizar la consulta, vamos a quitar el límite y resumir el número de valores de ubicación.

```kusto
Resources
| where type =~ 'Microsoft.Compute/virtualMachines'
| summarize count() by location
```

```azurecli-interactive
az graph query -q "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | summarize count() by location"
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | summarize count() by location"
```

Los resultados de JSON tienen una estructura similar a la del ejemplo siguiente:

```json
[
  {
    "count_": 386,
    "location": "eastus"
  },
  {
    "count_": 215,
    "location": "southcentralus"
  },
  {
    "count_": 59,
    "location": "westus"
  }
]
```

Ahora podemos ver cuántas máquinas virtuales tenemos en cada región de Azure.

### <a name="virtual-machines-by-sku"></a>Máquinas virtuales por SKU

Volviendo a las propiedades originales de la máquina virtual, vamos a intentar encontrar todas las máquinas virtuales que tengan un tamaño de SKU de **Standard_B2s**. Mirando el JSON devuelto, vemos que está almacenado en **properties.hardwareprofile.vmsize**. Actualizaremos la consulta para buscar todas las máquinas virtuales que coincidan con este tamaño y se devolverá solo el nombre de la máquina virtual y la región.

```kusto
Resources
| where type =~ 'Microsoft.Compute/virtualMachines' and properties.hardwareProfile.vmSize == 'Standard_B2s'
| project name, resourceGroup
```

```azurecli-interactive
az graph query -q "Resources | where type =~ 'Microsoft.Compute/virtualMachines' and properties.hardwareProfile.vmSize == 'Standard_B2s' | project name, resourceGroup"
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Compute/virtualMachines' and properties.hardwareProfile.vmSize == 'Standard_B2s' | project name, resourceGroup"
```

### <a name="virtual-machines-connected-to-premium-managed-disks"></a>Máquinas virtuales conectadas a discos administrados premium

Si quisiéramos obtener los detalles de los discos administrados premium que están conectados a estas máquinas virtuales **Standard_B2s**, podríamos expandir la consulta para que nos proporcione el identificador de recurso de esos discos administrados.

```kusto
Resources
| where type =~ 'Microsoft.Compute/virtualmachines' and properties.hardwareProfile.vmSize == 'Standard_B2s'
| extend disk = properties.storageProfile.osDisk.managedDisk
| where disk.storageAccountType == 'Premium_LRS'
| project disk.id
```

> [!NOTE]
> Otra forma de obtener la SKU habría sido utilizando la propiedad **aliases** **Microsoft.Compute/virtualMachines/sku.name**. Consulte los ejemplos [Mostrar alias](../samples/starter.md#show-aliases) y [Mostrar valores para distintos alias](../samples/starter.md#distinct-alias-values).

```azurecli-interactive
az graph query -q "Resources | where type =~ 'Microsoft.Compute/virtualmachines' and properties.hardwareProfile.vmSize == 'Standard_B2s' | extend disk = properties.storageProfile.osDisk.managedDisk | where disk.storageAccountType == 'Premium_LRS' | project disk.id"
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Compute/virtualmachines' and properties.hardwareProfile.vmSize == 'Standard_B2s' | extend disk = properties.storageProfile.osDisk.managedDisk | where disk.storageAccountType == 'Premium_LRS' | project disk.id"
```

El resultado es una lista de identificadores de disco.

### <a name="managed-disk-discovery"></a>Detección de disco administrado

Con el primer registro de la consulta anterior, vamos a explorar las propiedades que existen en el disco administrado que se asoció a la primera máquina virtual. La consulta actualizada utiliza el identificador del disco y cambia el tipo.

Ejemplo de salida de la consulta anterior, por ejemplo:

```json
[
  {
    "disk_id": "/subscriptions/<subscriptionId>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/disks/ContosoVM1_OsDisk_1_9676b7e1b3c44e2cb672338ebe6f5166"
  }
]
```

```kusto
Resources
| where type =~ 'Microsoft.Compute/disks' and id == '/subscriptions/<subscriptionId>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/disks/ContosoVM1_OsDisk_1_9676b7e1b3c44e2cb672338ebe6f5166'
```

Antes de ejecutar la consulta, ¿cómo sabíamos que el **tipo** debería ser ahora **Microsoft.Compute/discs**? Si se observa el identificador completo, podrá ver **/providers/Microsoft.Compute/disks/** como parte de la cadena.
Este fragmento de cadena nos da una sugerencia sobre qué tipo buscar. Un método alternativo sería quitar el límite por tipo y en su lugar buscar solo por el campo del identificador. Como el identificador es único, solo se devolverá un registro y la propiedad **type** que contiene proporciona ese detalle.

> [!NOTE]
> Para que este ejemplo funcione, debe reemplazar el campo del identificador por un resultado de su propio entorno.

```azurecli-interactive
az graph query -q "Resources | where type =~ 'Microsoft.Compute/disks' and id == '/subscriptions/<subscriptionId>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/disks/ContosoVM1_OsDisk_1_9676b7e1b3c44e2cb672338ebe6f5166'"
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Compute/disks' and id == '/subscriptions/<subscriptionId>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/disks/ContosoVM1_OsDisk_1_9676b7e1b3c44e2cb672338ebe6f5166'"
```

Los resultados de JSON tienen una estructura similar a la del ejemplo siguiente:

```json
[
  {
    "id": "/subscriptions/<subscriptionId>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/disks/ContosoVM1_OsDisk_1_9676b7e1b3c44e2cb672338ebe6f5166",
    "kind": "",
    "location": "westus2",
    "managedBy": "",
    "name": "ContosoVM1_OsDisk_1_9676b7e1b3c44e2cb672338ebe6f5166",
    "plan": {},
    "properties": {
      "creationData": {
        "createOption": "Empty"
      },
      "diskSizeGB": 127,
      "diskState": "ActiveSAS",
      "provisioningState": "Succeeded",
      "timeCreated": "2018-09-14T12:17:32.2570000Z"
    },
    "resourceGroup": "MyResourceGroup",
    "sku": {
      "name": "Premium_LRS",
      "tier": "Premium"
    },
    "subscriptionId": "<subscriptionId>",
    "tags": {
      "environment": "prod"
    },
    "type": "microsoft.compute/disks"
  }
]
```

## <a name="explore-virtual-machines-to-find-public-ip-addresses"></a>Exploración de máquinas virtuales para buscar direcciones IP públicas

Este conjunto de consultas primero busca y almacena todos los recursos de interfaces de red (NIC) conectados a las máquinas virtuales. Después, las consultas utilizan la lista de NIC para buscar cada uno de los recursos que es una dirección IP pública y almacenar estos valores. Por último, las consultas proporcionan una lista de las direcciones IP públicas.

```azurecli-interactive
# Use Resource Graph to get all NICs and store in the 'nics.txt' file
az graph query -q "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | project nic = tostring(properties['networkProfile']['networkInterfaces'][0]['id']) | where isnotempty(nic) | distinct nic | limit 20" --output table | tail -n +3 > nics.txt

# Review the output of the query stored in 'nics.txt'
cat nics.txt
```

```azurepowershell-interactive
# Use Resource Graph to get all NICs and store in the $nics variable
$nics = Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | project nic = tostring(properties['networkProfile']['networkInterfaces'][0]['id']) | where isnotempty(nic) | distinct nic | limit 20"

# Review the output of the query stored in the variable
$nics.nic
```

Utilice el archivo (CLI de Azure) o variable (Azure PowerShell) en la siguiente consulta para obtener los detalles de los recursos de interfaz de red relacionados en los que hay una dirección IP pública asociada a la NIC.

```azurecli-interactive
# Use Resource Graph with the 'nics.txt' file to get all related public IP addresses and store in 'publicIp.txt' file
az graph query -q="Resources | where type =~ 'Microsoft.Network/networkInterfaces' | where id in ('$(awk -vORS="','" '{print $0}' nics.txt | sed 's/,$//')') | project publicIp = tostring(properties['ipConfigurations'][0]['properties']['publicIPAddress']['id']) | where isnotempty(publicIp) | distinct publicIp" --output table | tail -n +3 > ips.txt

# Review the output of the query stored in 'ips.txt'
cat ips.txt
```

```azurepowershell-interactive
# Use Resource Graph  with the $nics variable to get all related public IP addresses and store in $ips variable
$ips = Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Network/networkInterfaces' | where id in ('$($nics.nic -join "','")') | project publicIp = tostring(properties['ipConfigurations'][0]['properties']['publicIPAddress']['id']) | where isnotempty(publicIp) | distinct publicIp"

# Review the output of the query stored in the variable
$ips.publicIp
```

Por último, utilice la lista de recursos de direcciones IP públicas almacenadas el archivo (CLI de Azure) o variable (Azure PowerShell) para obtener la dirección IP pública real y mostrarla.

```azurecli-interactive
# Use Resource Graph with the 'ips.txt' file to get the IP address of the public IP address resources
az graph query -q="Resources | where type =~ 'Microsoft.Network/publicIPAddresses' | where id in ('$(awk -vORS="','" '{print $0}' ips.txt | sed 's/,$//')') | project ip = tostring(properties['ipAddress']) | where isnotempty(ip) | distinct ip" --output table
```

```azurepowershell-interactive
# Use Resource Graph with the $ips variable to get the IP address of the public IP address resources
Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Network/publicIPAddresses' | where id in ('$($ips.publicIp -join "','")') | project ip = tostring(properties['ipAddress']) | where isnotempty(ip) | distinct ip"
```

Para ver cómo llevar a cabo estos pasos en una sola consulta con el operador `join`, consulte el ejemplo [Enumeración de máquinas virtuales con su interfaz de red y dirección IP pública](../samples/advanced.md#join-vmpip).

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre el [lenguaje de consulta](query-language.md).
- Consulte el lenguaje en uso en[Consultas básicas](../samples/starter.md).
- Consulte los usos avanzados en [Consultas avanzadas](../samples/advanced.md).
