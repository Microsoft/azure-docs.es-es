---
title: Lectura de registros de flujos de NSG | Microsoft Docs
description: Obtenga información sobre cómo usar Azure PowerShell para analizar los registros de flujo de grupos de seguridad de red, que se crean cada hora y se actualizan cada pocos minutos en Azure Network Watcher.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/13/2017
ms.author: damendo
ms.openlocfilehash: 18023f5a5b36dd971080d7a62fdd44698a205bb4
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88272542"
---
# <a name="read-nsg-flow-logs"></a>Lectura de registros de flujos de NSG

Aprenda a leer las entradas de los registros de flujos de NSG con PowerShell.

Los registros de flujos de NSG se almacenan en una cuenta de almacenamiento de [blobs en bloques](https://docs.microsoft.com/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs). Los blobs en bloques se componen de bloques más pequeños. Cada registro es un blob en bloques independiente que se genera cada hora. Cada hora se generan registros nuevos, que se actualizan con entradas nuevas cada pocos minutos con los datos más recientes. En este artículo aprenderá a leer las secciones de los registros de flujos.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="scenario"></a>Escenario

En el siguiente escenario tiene un registro de flujo de ejemplo que se almacena en una cuenta de almacenamiento. Aprenderá a leer de forma selectiva los eventos más recientes de los registros de los flujos de grupo de seguridad de red. En este artículo usaremos PowerShell, aunque los conceptos tratados en él no se limitan al lenguaje de programación y se pueden aplicar a todos los lenguajes admitidos por las API de Azure Storage.

## <a name="setup"></a>Configurar

Antes de empezar, tiene que tener el registro de flujo de grupo de seguridad de red habilitado en uno o más grupos de seguridad de red de su cuenta. Para obtener instrucciones sobre cómo habilitar los registros de flujo de grupo de seguridad de red, consulte el artículo siguiente: [Introducción al registro de flujo de grupos de seguridad de red](network-watcher-nsg-flow-logging-overview.md).

## <a name="retrieve-the-block-list"></a>Recuperación de la lista de bloques

En el siguiente PowerShell se configuran las variables necesarias para consultar el blob del registro de flujos de NSG y mostrar los bloques del blob en bloques [CloudBlockBlob](https://docs.microsoft.com/dotnet/api/microsoft.azure.storage.blob.cloudblockblob). Actualice el script para que contenga valores válidos para su entorno.

```powershell
function Get-NSGFlowLogCloudBlockBlob {
    [CmdletBinding()]
    param (
        [string] [Parameter(Mandatory=$true)] $subscriptionId,
        [string] [Parameter(Mandatory=$true)] $NSGResourceGroupName,
        [string] [Parameter(Mandatory=$true)] $NSGName,
        [string] [Parameter(Mandatory=$true)] $storageAccountName,
        [string] [Parameter(Mandatory=$true)] $storageAccountResourceGroup,
        [string] [Parameter(Mandatory=$true)] $macAddress,
        [datetime] [Parameter(Mandatory=$true)] $logTime
    )

    process {
        # Retrieve the primary storage account key to access the NSG logs
        $StorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $storageAccountResourceGroup -Name $storageAccountName).Value[0]

        # Setup a new storage context to be used to query the logs
        $ctx = New-AzStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

        # Container name used by NSG flow logs
        $ContainerName = "insights-logs-networksecuritygroupflowevent"

        # Name of the blob that contains the NSG flow log
        $BlobName = "resourceId=/SUBSCRIPTIONS/${subscriptionId}/RESOURCEGROUPS/${NSGResourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/${NSGName}/y=$($logTime.Year)/m=$(($logTime).ToString("MM"))/d=$(($logTime).ToString("dd"))/h=$(($logTime).ToString("HH"))/m=00/macAddress=$($macAddress)/PT1H.json"

        # Gets the storage blog
        $Blob = Get-AzStorageBlob -Context $ctx -Container $ContainerName -Blob $BlobName

        # Gets the block blog of type 'Microsoft.Azure.Storage.Blob.CloudBlob' from the storage blob
        $CloudBlockBlob = [Microsoft.Azure.Storage.Blob.CloudBlockBlob] $Blob.ICloudBlob

        #Return the Cloud Block Blob
        $CloudBlockBlob
    }
}

function Get-NSGFlowLogBlockList  {
    [CmdletBinding()]
    param (
        [Microsoft.Azure.Storage.Blob.CloudBlockBlob] [Parameter(Mandatory=$true)] $CloudBlockBlob
    )
    process {
        # Stores the block list in a variable from the block blob.
        $blockList = $CloudBlockBlob.DownloadBlockListAsync()

        # Return the Block List
        $blockList
    }
}


$CloudBlockBlob = Get-NSGFlowLogCloudBlockBlob -subscriptionId "yourSubscriptionId" -NSGResourceGroupName "FLOWLOGSVALIDATIONWESTCENTRALUS" -NSGName "V2VALIDATIONVM-NSG" -storageAccountName "yourStorageAccountName" -storageAccountResourceGroup "ml-rg" -macAddress "000D3AF87856" -logTime "11/11/2018 03:00" 

$blockList = Get-NSGFlowLogBlockList -CloudBlockBlob $CloudBlockBlob
```

La variable `$blockList` devuelve una lista de los bloques del blob. Cada blob en bloques contiene al menos dos bloques.  El primer bloque tiene una longitud de `12` bytes. Este bloque contiene los corchetes de apertura del registro de json. El otro bloque son los corchetes de cierre y tiene una longitud de `2` bytes.  Como puede ver, el siguiente registro de ejemplo contiene siete entradas, y cada una de ellas representa una entrada. Todas las entradas nuevas del registro se agregan al final, justo antes del bloque final.

```
Name                                         Length Committed
----                                         ------ ---------
ZDk5MTk5N2FkNGE0MmY5MTk5ZWViYjA0YmZhODRhYzY=     12      True
NzQxNDA5MTRhNDUzMGI2M2Y1MDMyOWZlN2QwNDZiYzQ=   2685      True
ODdjM2UyMWY3NzFhZTU3MmVlMmU5MDNlOWEwNWE3YWY=   2586      True
ZDU2MjA3OGQ2ZDU3MjczMWQ4MTRmYWNhYjAzOGJkMTg=   2688      True
ZmM3ZWJjMGQ0ZDA1ODJlOWMyODhlOWE3MDI1MGJhMTc=   2775      True
ZGVkYTc4MzQzNjEyMzlmZWE5MmRiNjc1OWE5OTc0OTQ=   2676      True
ZmY2MjUzYTIwYWIyOGU1OTA2ZDY1OWYzNmY2NmU4ZTY=   2777      True
Mzk1YzQwM2U0ZWY1ZDRhOWFlMTNhYjQ3OGVhYmUzNjk=   2675      True
ZjAyZTliYWE3OTI1YWZmYjFmMWI0MjJhNzMxZTI4MDM=      2      True
```

## <a name="read-the-block-blob"></a>Lectura del blob en bloques

A continuación tiene que leer la variable `$blocklist` para recuperar los datos. En este ejemplo se recorre en iteración la lista de bloques, se leen los bytes de cada bloque y se almacenan en una matriz. Para recuperar los datos se usa el método [DownloadRangeToByteArray](/dotnet/api/microsoft.azure.storage.blob.cloudblob.downloadrangetobytearray).

```powershell
function Get-NSGFlowLogReadBlock  {
    [CmdletBinding()]
    param (
        [System.Array] [Parameter(Mandatory=$true)] $blockList,
        [Microsoft.Azure.Storage.Blob.CloudBlockBlob] [Parameter(Mandatory=$true)] $CloudBlockBlob

    )
    # Set the size of the byte array to the largest block
    $maxvalue = ($blocklist | measure Length -Maximum).Maximum

    # Create an array to store values in
    $valuearray = @()

    # Define the starting index to track the current block being read
    $index = 0

    # Loop through each block in the block list
    for($i=0; $i -lt $blocklist.count; $i++)
    {
        # Create a byte array object to story the bytes from the block
        $downloadArray = New-Object -TypeName byte[] -ArgumentList $maxvalue

        # Download the data into the ByteArray, starting with the current index, for the number of bytes in the current block. Index is increased by 3 when reading to remove preceding comma.
        $CloudBlockBlob.DownloadRangeToByteArray($downloadArray,0,$index, $($blockList[$i].Length)) | Out-Null

        # Increment the index by adding the current block length to the previous index
        $index = $index + $blockList[$i].Length

        # Retrieve the string from the byte array

        $value = [System.Text.Encoding]::ASCII.GetString($downloadArray)

        # Add the log entry to the value array
        $valuearray += $value
    }
    #Return the Array
    $valuearray
}
$valuearray = Get-NSGFlowLogReadBlock -blockList $blockList -CloudBlockBlob $CloudBlockBlob
```

Ahora la matriz `$valuearray` contiene el valor de cadena de cada bloque. Para comprobar la entrada, ejecute `$valuearray[$valuearray.Length-2]` para obtener el penúltimo valor de la matriz. El último valor no sirve ya que es el corchete de cierre.

Los resultados de este valor se muestran en el ejemplo siguiente:

```json
        {
             "time": "2017-06-16T20:59:43.7340000Z",
             "systemId": "5f4d02d3-a7d0-4ed4-9ce8-c0ae9377951c",
             "category": "NetworkSecurityGroupFlowEvent",
             "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/CONTOSORG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/CONTOSONSG",
             "operationName": "NetworkSecurityGroupFlowEvents",
             "properties": {"Version":1,"flows":[{"rule":"DefaultRule_AllowInternetOutBound","flows":[{"mac":"000D3A18077E","flowTuples":["1497646722,10.0.0.4,168.62.32.14,44904,443,T,O,A","1497646722,10.0.0.4,52.240.48.24,45218,443,T,O,A","1497646725,10.
0.0.4,168.62.32.14,44910,443,T,O,A","1497646725,10.0.0.4,52.240.48.24,45224,443,T,O,A","1497646728,10.0.0.4,168.62.32.14,44916,443,T,O,A","1497646728,10.0.0.4,52.240.48.24,45230,443,T,O,A","1497646732,10.0.0.4,168.62.32.14,44922,443,T,O,A","14976
46732,10.0.0.4,52.240.48.24,45236,443,T,O,A","1497646735,10.0.0.4,168.62.32.14,44928,443,T,O,A","1497646735,10.0.0.4,52.240.48.24,45242,443,T,O,A","1497646738,10.0.0.4,168.62.32.14,44934,443,T,O,A","1497646738,10.0.0.4,52.240.48.24,45248,443,T,O,
A","1497646742,10.0.0.4,168.62.32.14,44942,443,T,O,A","1497646742,10.0.0.4,52.240.48.24,45256,443,T,O,A","1497646745,10.0.0.4,168.62.32.14,44948,443,T,O,A","1497646745,10.0.0.4,52.240.48.24,45262,443,T,O,A","1497646749,10.0.0.4,168.62.32.14,44954
,443,T,O,A","1497646749,10.0.0.4,52.240.48.24,45268,443,T,O,A","1497646753,10.0.0.4,168.62.32.14,44960,443,T,O,A","1497646753,10.0.0.4,52.240.48.24,45274,443,T,O,A","1497646756,10.0.0.4,168.62.32.14,44966,443,T,O,A","1497646756,10.0.0.4,52.240.48
.24,45280,443,T,O,A","1497646759,10.0.0.4,168.62.32.14,44972,443,T,O,A","1497646759,10.0.0.4,52.240.48.24,45286,443,T,O,A","1497646763,10.0.0.4,168.62.32.14,44978,443,T,O,A","1497646763,10.0.0.4,52.240.48.24,45292,443,T,O,A","1497646766,10.0.0.4,
168.62.32.14,44984,443,T,O,A","1497646766,10.0.0.4,52.240.48.24,45298,443,T,O,A","1497646769,10.0.0.4,168.62.32.14,44990,443,T,O,A","1497646769,10.0.0.4,52.240.48.24,45304,443,T,O,A","1497646773,10.0.0.4,168.62.32.14,44996,443,T,O,A","1497646773,
10.0.0.4,52.240.48.24,45310,443,T,O,A","1497646776,10.0.0.4,168.62.32.14,45002,443,T,O,A","1497646776,10.0.0.4,52.240.48.24,45316,443,T,O,A","1497646779,10.0.0.4,168.62.32.14,45008,443,T,O,A","1497646779,10.0.0.4,52.240.48.24,45322,443,T,O,A"]}]}
,{"rule":"DefaultRule_DenyAllInBound","flows":[]},{"rule":"UserRule_ssh-rule","flows":[]},{"rule":"UserRule_web-rule","flows":[{"mac":"000D3A18077E","flowTuples":["1497646738,13.82.225.93,10.0.0.4,1180,80,T,I,A","1497646750,13.82.225.93,10.0.0.4,
1184,80,T,I,A","1497646768,13.82.225.93,10.0.0.4,1181,80,T,I,A","1497646780,13.82.225.93,10.0.0.4,1336,80,T,I,A"]}]}]}
        }
```

Este escenario es un ejemplo de cómo leer las entradas de registros de flujos de NSG sin tener que analizar todo el registro. Puede leer las entradas nuevas del registro a medida que se escriben mediante el identificador de bloque o mediante el seguimiento de la longitud de los bloques almacenados en el blob en bloques. Esto le permite leer solo las entradas nuevas.

## <a name="next-steps"></a>Pasos siguientes


Visite [Uso de Elastic Stack](network-watcher-visualize-nsg-flow-logs-open-source-tools.md), [Uso de Grafana](network-watcher-nsg-grafana.md) y [Uso de Graylog](network-watcher-analyze-nsg-flow-logs-graylog.md) para obtener más información sobre cómo ver los registros de flujos de NSG. Puede encontrar un enfoque de la función de Azure de código abierto para consumir los blobs directamente y emitir a varios consumidores de Log Analytics aquí: [Conector de registros de flujo de NSG para Azure Network Watcher](https://github.com/Microsoft/AzureNetworkWatcherNSGFlowLogsConnector).

Puede usar [Análisis de tráfico de Azure](https://docs.microsoft.com/azure/network-watcher/traffic-analytics) para sacar conclusiones sobre los flujos de tráfico. Análisis de tráfico usa [Log Analytics](https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal) para permitir las consultas en el flujo de tráfico.

Para obtener más información sobre los blobs de almacenamiento, visite: [Enlaces de Azure Blob Storage para Azure Functions](../azure-functions/functions-bindings-storage-blob.md)
