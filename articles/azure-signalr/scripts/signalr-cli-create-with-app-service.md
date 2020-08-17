---
title: Creación de una instancia de SignalR Service con App Service mediante la CLI de Azure
description: Use la CLI de Azure para crear una instancia de SignalR Service con App Service. Obtenga información sobre todos los comandos de la CLI para Azure SignalR Service.
author: sffamily
ms.service: signalr
ms.devlang: azurecli
ms.topic: sample
ms.date: 11/13/2018
ms.author: zhshang
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 3b0d88b548f68bc365adfa3fdb3149db9c9f43f0
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87494747"
---
# <a name="create-a-signalr-service-with-an-app-service"></a>Creación de una instancia de SignalR Service con una instancia de App Service

Este script de ejemplo crea un nuevo recurso de Azure SignalR Service, que se utiliza para insertar las actualizaciones de contenido en tiempo real en los clientes. Este script también agrega una nueva aplicación web y un nuevo plan de App Service para hospedar la aplicación web de ASP.NET que usa la instancia de SignalR Service. La aplicación web se configura con una configuración de aplicación denominada *AzureSignalRConnectionString* para conectar con el nuevo recurso de SignalR Service.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este artículo es preciso que ejecute la versión 2.0 o posterior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli). 

## <a name="sample-script"></a>Script de ejemplo

Este script usa la extensión *signalr* con la CLI de Azure. Ejecute el siguiente comando para instalar la extensión *signalr* para la CLI de Azure antes de usar este script de ejemplo:

```azurecli-interactive
#!/bin/bash

# Generate a unique suffix for the service name
let randomNum=$RANDOM*$RANDOM

# Generate unique names for the SignalR service, resource group, 
# app service, and app service plan
SignalRName=SignalRTestSvc$randomNum
#resource name must be lowercase
mySignalRSvcName=${SignalRName,,}
myResourceGroupName=$SignalRName"Group"
myWebAppName=SignalRTestWebApp$randomNum
myAppSvcPlanName=$myAppSvcName"Plan"

# Create resource group 
az group create --name $myResourceGroupName --location eastus

# Create the Azure SignalR Service resource
az signalr create \
  --name $mySignalRSvcName \
  --resource-group $myResourceGroupName \
  --sku Standard_S1 \
  --unit-count 1 \
  --service-mode Default

# Create an App Service plan.
az appservice plan create --name $myAppSvcPlanName --resource-group $myResourceGroupName --sku FREE

# Create the Web App
az webapp create --name $myWebAppName --resource-group $myResourceGroupName --plan $myAppSvcPlanName  

# Get the SignalR primary connection string
primaryConnectionString=$(az signalr key list --name $mySignalRSvcName \
  --resource-group $myResourceGroupName --query primaryConnectionString -o tsv)

#Add an app setting to the web app for the SignalR connection
az webapp config appsettings set --name $myWebAppName --resource-group $myResourceGroupName \
  --settings "AzureSignalRConnectionString=$primaryConnectionString"
```

Anote el nombre real generado para el nuevo grupo de recursos. Se mostrará en la salida. Usará ese nombre de grupo de recursos cuando quiera eliminar todos los recursos del grupo.

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Cada comando de la tabla crea un vínculo a documentación específica del comando. Este script usa los siguientes comandos:

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az signalr create](/cli/azure/signalr#az-signalr-create) | Crea un recurso de Azure SignalR Service. |
| [az signalr key list](/cli/azure/signalr/key#az-signalr-key-list) | Muestra las claves, que usará la aplicación al insertar actualizaciones de contenido en tiempo real con SignalR. |
| [az appservice plan create](/cli/azure/appservice/plan#az-appservice-plan-create) | Crea un plan de Azure App Service para hospedar aplicaciones web. |
| [az webapp create](/cli/azure/webapp#az-webapp-create) | Crea una aplicación web de Azure mediante el plan de hospedaje de App Service. |
| [az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set) | Agrega una nueva configuración de aplicación para la aplicación web. Esta configuración de aplicación se usa para almacenar la cadena de conexión de SignalR. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Se pueden encontrar ejemplos de scripts adicionales de la CLI de Azure SignalR Service en la [documentación de Azure SignalR Service](../signalr-reference-cli.md).
