---
title: 'CLI de Azure: restricción del acceso de importación y exportación a Managed Disks con vínculos privados (versión preliminar)'
description: Habilite vínculos privados (versión preliminar) para los discos administrados con la CLI de Azure. Permite exportar e importar discos de forma segura solo dentro de la red virtual.
author: roygara
ms.service: virtual-machines
ms.topic: overview
ms.date: 07/15/2020
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: 5df11e704987098d61ced7afbff5e6234d4d5f04
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420298"
---
# <a name="azure-cli---restrict-importexport-access-for-managed-disks-with-private-links-preview"></a>CLI de Azure: restricción del acceso de importación y exportación a Managed Disks con vínculos privados (versión preliminar)

Puede usar [puntos de conexión privados](../../private-link/private-endpoint-overview.md) (versión preliminar) para restringir la exportación e importación de Managed Disks y el acceso seguro a los datos mediante un [vínculo privado](../../private-link/private-link-overview.md) desde los clientes a la red virtual de Azure. El punto de conexión privado usa una dirección IP del espacio de direcciones de la red virtual para el servicio Managed Disks. El tráfico de red entre los clientes de la red virtual y Managed Disks atraviesa la red virtual y un vínculo privado de la red troncal de Microsoft, lo que elimina la exposición a la red pública de Internet. 

Para usar vínculos privados para exportar e importar de Managed Disks, primero debe crear un recurso de acceso a disco y vincularlo a una red virtual en la misma suscripción mediante la creación de un punto de conexión privado. A continuación, asocie un disco o una instantánea con una instancia de acceso a disco. Finalmente, establezca la propiedad NetworkAccessPolicy del disco o la instantánea en `AllowPrivate`. Esto limitará el acceso a la red virtual. 

Puede establecer la propiedad NetworkAccessPolicy en `DenyAll` para evitar que alguien exporte los datos de un disco o una instantánea. El valor predeterminado de la propiedad NetworkAccessPolicy es `AllowAll`.

## <a name="limitations"></a>Limitaciones

[!INCLUDE [virtual-machines-disks-private-links-limitations](../../../includes/virtual-machines-disks-private-links-limitations.md)]

## <a name="regional-availability"></a>Disponibilidad regional

[!INCLUDE [virtual-machines-disks-private-links-regions](../../../includes/virtual-machines-disks-private-links-regions.md)]

## <a name="log-in-into-your-subscription-and-set-your-variables"></a>Inicio de sesión en la suscripción y establecimiento de las variables

```azurecli-interactive

subscriptionId=yourSubscriptionId
resourceGroupName=yourResourceGroupName
region=northcentralus
diskAccessName=yourDiskAccessForPrivateLinks
vnetName=yourVNETForPrivateLinks
subnetName=yourSubnetForPrivateLinks
privateEndPointName=yourPrivateLinkForSecureMDExportImport
privateEndPointConnectionName=yourPrivateLinkConnection

#The name of an existing disk which is the source of the snapshot
sourceDiskName=yourSourceDiskForSnapshot

#The name of the new snapshot which will be secured via Private Links
snapshotNameSecuredWithPL=yourSnapshotNameSecuredWithPL

az login

az account set --subscription $subscriptionId

```

## <a name="create-a-disk-access-using-azure-cli"></a>Creación de un acceso a disco mediante la CLI de Azure
```azurecli-interactive
az group deployment create -g $resourceGroupName \
--template-uri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/privatelinks/CreateDiskAccess.json" \
--parameters "region=$region" "diskAccessName=$diskAccessName"

diskAccessId=$(az resource show -n $diskAccessName -g $resourceGroupName --namespace Microsoft.Compute --resource-type diskAccesses --query [id] -o tsv)
```

## <a name="create-a-virtual-network"></a>Creación de una red virtual

Las directivas de red como, por ejemplo, los grupos de seguridad de red (NSG) no se admiten para los puntos de conexión privados. Para implementar un punto de conexión privado en una subred especificada, es necesario un valor de deshabilitación explícito en dicha subred. 

```azurecli-interactive
az network vnet create --resource-group $resourceGroupName \
    --name $vnetName \
    --subnet-name $subnetName
```
## <a name="disable-subnet-private-endpoint-policies"></a>Deshabilitación de las directivas de punto de conexión privado

Azure implementa los recursos en una subred de una red virtual, por lo que debe actualizar la subred para deshabilitar las directivas de red del punto de conexión privado. 

```azurecli-interactive
az network vnet subnet update --resource-group $resourceGroupName \
    --name $subnetName  \
    --vnet-name $vnetName \
    --disable-private-endpoint-network-policies true
```
## <a name="create-a-private-endpoint-for-the-disk-access-object"></a>Creación de un punto de conexión privado para el objeto de acceso a disco

```azurecli-interactive
az network private-endpoint create --resource-group $resourceGroupName \
    --name $privateEndPointName \
    --vnet-name $vnetName  \
    --subnet $subnetName \
    --private-connection-resource-id $diskAccessId \
    --group-ids disks \
    --connection-name $privateEndPointConnectionName
```

## <a name="configure-the-private-dns-zone"></a>Configuración de la zona DNS privada

Cree una zona DNS privada para el dominio del blob de almacenamiento, cree un vínculo de asociación con la red virtual y un grupo de zona DNS para asociar el punto de conexión privado con la zona DNS privada. 

```azurecli-interactive
az network private-dns zone create --resource-group $resourceGroupName \ 
    --name "privatelink.blob.core.windows.net"

az network private-dns link vnet create --resource-group $resourceGroupName \
    --zone-name "privatelink.blob.core.windows.net" \
    --name yourDNSLink \
    --virtual-network $vnetName \
    --registration-enabled false 

az network private-endpoint dns-zone-group create \
   --resource-group $resourceGroupName \
   --endpoint-name $privateEndPointName \
   --name yourZoneGroup \
   --private-dns-zone "privatelink.blob.core.windows.net" \
   --zone-name disks
```

## <a name="create-a-disk-protected-with-private-links"></a>Creación de un disco protegido con vínculos privados
  ```cli
    resourceGroupName=yourResourceGroupName
    region=northcentralus
    diskAccessName=yourDiskAccessName
    diskName=yourDiskName
    diskSkuName=Standard_LRS
    diskSizeGB=128

    diskAccessId=$(az resource show -n $diskAccessName -g $resourceGroupName --namespace Microsoft.Compute --resource-type diskAccesses --query [id] -o tsv)

    az group deployment create -g $resourceGroupName \
       --template-uri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/privatelinks/CreateDisksWithExportViaPrivateLink.json" \
       --parameters "diskName=$diskName" \
       "diskSkuName=$diskSkuName" \
       "diskSizeGB=$diskSizeGB" \
       "diskAccessId=$diskAccessId" \
       "region=$region" \
       "networkAccessPolicy=AllowPrivate"
```

## <a name="create-a-snapshot-of-a-disk-protected-with-private-links"></a>Creación de una instantánea de un disco protegido con vínculos privados
   ```cli
    resourceGroupName=yourResourceGroupName
    region=northcentralus
    diskAccessName=yourDiskAccessName
    sourceDiskName=yourSourceDiskForSnapshot
    snapshotNameSecuredWithPL=yourSnapshotName

    diskId=$(az disk show -n $sourceDiskName -g $resourceGroupName --query [id] -o tsv)
   
    diskAccessId=$(az resource show -n $diskAccessName -g $resourceGroupName --namespace Microsoft.Compute --resource-type diskAccesses --query [id] -o tsv)
   
    az group deployment create -g $resourceGroupName \
      --template-uri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/privatelinks/CreateSnapshotWithExportViaPrivateLink.json" \
      --parameters "snapshotName=$snapshotNameSecuredWithPL" \
      "sourceResourceId=$diskId" \
      "diskAccessId=$diskAccessId" \
      "region=$region" \
      "networkAccessPolicy=AllowPrivate" 
```

## <a name="next-steps"></a>Pasos siguientes

- [Preguntas más frecuentes sobre vínculos privados](faq-for-disks.md#private-links-for-securely-exporting-and-importing-managed-disks)
- [Exportación o copia de instantáneas administradas como VHD a una cuenta de almacenamiento de otra región con CLI](../scripts/virtual-machines-linux-cli-sample-copy-managed-disks-vhd.md)
