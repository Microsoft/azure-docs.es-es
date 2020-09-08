---
title: 'CLI: Implementación de un punto de conexión privado para una aplicación web con la CLI de Azure'
description: Aprenda a usar la CLI de Azure para implementar un punto de conexión privado para la aplicación web.
author: ericgre
ms.assetid: a56faf72-7237-41e7-85ce-da8346f2bcaa
ms.devlang: azurecli
ms.topic: sample
ms.date: 07/06/2020
ms.author: ericg
ms.service: app-service
ms.workload: web
ms.openlocfilehash: 08e1878adab680329a33bc44020b49dce38de4d6
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88959738"
---
# <a name="create-an-app-service-app-and-deploy-private-endpoint-using-azure-cli"></a>Creación de una aplicación de App Service e implementación de un punto de conexión privado mediante la CLI de Azure

Este script de ejemplo crea una aplicación en App Service con sus recursos relacionados y, después, implementa un punto de conexión privado.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si, en su lugar, decide instalar y usar la CLI de Azure en un entorno local, para esta guía de inicio rápido se necesita la versión 2.0.28 de la CLI de Azure o una versión posterior. Ejecute `az --version` para buscar la versión instalada. Consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli) para obtener información sobre la instalación o actualización.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para poder crear un recurso, debe crear un grupo de recursos que hospede la aplicación web, la red virtual y otros componentes de red. Cree un grupo de recursos con [az group create](/cli/azure/group). En el ejemplo, se crea un grupo de recursos denominado *myResourceGroup*, en la ubicación *francecentral*:

```azurecli-interactive
az group create --name myResourceGroup --location francecentral 
```

## <a name="create-an-app-service-plan"></a>Creación de un plan de App Service

Debe crear un plan de App Service para hospedar la aplicación web.
Hágalo con el comando [az appservice plan create](/cli/azure/appservice/plan?view=azure-cli-latest#az-appservice-plan-create).
En este ejemplo se crea un plan de App Service denominado *myAppServicePlan* en la ubicación *francecentral* con el plan de tarifa *P1V2* y un solo rol de trabajo: 

```azurecli-interactive
az appservice plan create \
--name myAppServicePlan \
--resource-group myResourceGroup \
--location francecentral \
--sku P1V2 \
--number-of-workers 1
```

## <a name="create-a-web-app"></a>Creación de una aplicación web

Ahora que tiene un plan de App Service puede implementar una aplicación web.
Cree una aplicación web con [az appservice plan create](/cli/azure/webapp?view=azure-cli-latest#az-webapp-create.
En este ejemplo se crea una aplicación web denominada *mySiteName* en el plan denominado *myAppServicePlan*.

```azurecli-interactive
az webapp create \
--name mySiteName \
--resource-group myResourceGroup \
--plan myAppServicePlan
```

## <a name="create-a-vnet"></a>Creación de una red virtual

Cree la red virtual con [az network vnet create](/cli/azure/network/vnet). En este ejemplo se crea una red virtual predeterminada denominada *myVNet* con una subred denominada *mySubnet*:

```azurecli-interactive
az network vnet create \
--name myVNet \
--resource-group myResourceGroup \
--location francecentral \
--address-prefixes 10.8.0.0/16 \
--subnet-name mySubnet \
--subnet-prefixes 10.8.100.0/24
```

## <a name="configure-the-subnet"></a>Configuración de la subred 

Debe actualizar la subred para deshabilitar las directivas de red del punto de conexión privado. Actualice una configuración de subred denominada *mySubnet* con [az network vnet subnet update](https://docs.microsoft.com/cli/azure/network/vnet/subnet?view=azure-cli-latest#az-network-vnet-subnet-update):

```azurecli-interactive
az network vnet subnet update \
--name mySubnet \
--resource-group myResourceGroup \
--vnet-name myVNet \
--disable-private-endpoint-network-policies true
```

## <a name="create-the-private-endpoint"></a>Creación del punto de conexión privado

Cree el punto de conexión privado para la aplicación web con [az network private-endpoint create](/cli/azure/network/private-endpoint). En este ejemplo se crea un punto de conexión privado denominado *myPrivateEndpoint* en la subred *mySubnet* de la red virtual *myVNet* con una conexión denominada *myConnectionName* al identificador de recurso de la aplicación web /subscriptions/SubscriptionID/resourceGroups/myResourceGroup/providers/Microsoft.Web/sites/myWebApp; el parámetro de grupo es *sites* para el tipo de aplicación web. 

```azurecli-interactive
az network private-endpoint create \
--name myPrivateEndpoint \
--resource-group myResourceGroup \
--vnet-name myVNet \
--subnet mySubnet \
--connection-name myConnectionName \
--private-connection-resource-id /subscriptions/SubscriptionID/resourceGroups/myResourceGroup/providers/Microsoft.Web/sites/myWebApp \
--group-id sites
```

## <a name="configure-the-private-zone"></a>Configuración de la zona privada

Al final, debe crear una zona DNS privada llamada *privatelink.azurewebsites.net* vinculada a la red virtual para resolver el nombre DNS de la aplicación web.


```azurecli-interactive
az network private-dns zone create \
--name privatelink.azurewebsites.net \
--resource-group myResourceGroup

az network private-dns link vnet create \
--name myDNSLink \
--resource-group myResourceGroup \
--registration-enabled false \
--virtual-network myVNet \
--zone-name privatelink.azurewebsites.net

az network private-endpoint dns-zone-group create \
--name myZoneGroup \
--resource-group myResourceGroup \
--endpoint-name myPrivateEndpoint \
--private-dns-zone privatelink.azurewebsites.net \
--zone-name privatelink.azurewebsites.net
```






[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]


## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).
- Puede encontrar ejemplos de script adicionales de la CLI de App Service en la [documentación de Azure App Service](../samples-cli.md).