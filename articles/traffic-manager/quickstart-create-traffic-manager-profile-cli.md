---
title: 'Inicio rápido: Creación de un perfil de alta disponibilidad de aplicaciones: CLI de Azure (Azure Traffic Manager)'
description: En este artículo de inicio rápido se describe cómo crear un perfil de Traffic Manager para crear una aplicación web de alta disponibilidad.
services: traffic-manager
author: duongau
mnager: kumud
Customer intent: As an IT admin, I want to direct user traffic to ensure high availability of web applications.
ms.service: traffic-manager
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/26/2020
ms.author: duau
ms.custom: devx-track-azurecli
ms.openlocfilehash: 429d107a68c8631983bf791108f5091f31a480c3
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89178089"
---
# <a name="quickstart-create-a-traffic-manager-profile-for-a-highly-available-web-application-using-azure-cli"></a>Inicio rápido: Creación de un perfil de Traffic Manager para una aplicación web de alta disponibilidad mediante la CLI de Azure

En esta guía de inicio rápido se describe cómo crear un perfil de Traffic Manager que ofrece alta disponibilidad para la aplicación web.

En este inicio rápido, creará dos instancias de una aplicación web. Cada una de ellas se ejecuta en una región de Azure distinta. Creará un perfil de Traffic Manager según la [prioridad del punto de conexión](traffic-manager-routing-methods.md#priority-traffic-routing-method). El perfil dirige el tráfico de usuario al sitio principal que ejecuta la aplicación web. Traffic Manager supervisa continuamente la aplicación web. Si el sitio principal no está disponible, proporciona la conmutación automática por error al sitio de copia de seguridad.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) ahora.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este tutorial es preciso que ejecute la CLI de Azure versión 2.0.28 o versiones posteriores. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos
Cree un grupo de recursos con [az group create](https://docs.microsoft.com/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli-interactive

  az group create \
    --name myResourceGroup \
    --location eastus

```

## <a name="create-a-traffic-manager-profile"></a>Crear un perfil de Traffic Manager

Cree un perfil de Traffic Manager con [az network traffic-manager profile create](https://docs.microsoft.com/cli/azure/network/traffic-manager/profile?view=azure-cli-latest#az-network-traffic-manager-profile-create) que dirija el tráfico de los usuarios según la prioridad del punto de conexión.

En el siguiente ejemplo, reemplace **<profile_name>** por un nombre de perfil de Traffic Manager único.

```azurecli-interactive

az network traffic-manager profile create \
    --name <profile_name> \
    --resource-group myResourceGroup \
    --routing-method Priority \
    --path "/" \
    --protocol HTTP \
    --unique-dns-name <profile_name> \
    --ttl 30 \
    --port 80

```

## <a name="create-web-apps"></a>Creación de aplicaciones web

Esta guía de inicio rápido requiere que haya implementado dos instancias de una aplicación web en dos regiones de Azure distintas (*Este de EE. UU.* y *Oeste de Europa*). Cada una de ellas servirá como los puntos de conexión principal y de conmutación por error de Traffic Manager.

### <a name="create-web-app-service-plans"></a>Creación de planes de Web App Service
Cree planes de Web App Service mediante [az appservice plan create](https://docs.microsoft.com/cli/azure/appservice/plan?view=azure-cli-latest#az-appservice-plan-create) para las dos instancias de la aplicación web que implementará en dos regiones de Azure distintas.

En el siguiente ejemplo, reemplace **<appspname_eastus>** y **<appspname_westeurope>** por un nombre de plan de App Service único.

```azurecli-interactive

az appservice plan create \
    --name <appspname_eastus> \
    --resource-group myResourceGroup \
    --location eastus \
    --sku S1

az appservice plan create \
    --name <appspname_westeurope> \
    --resource-group myResourceGroup \
    --location westeurope \
    --sku S1

```

### <a name="create-a-web-app-in-the-app-service-plan"></a>Creación de una aplicación web en el plan de App Service
Cree dos instancias de la aplicación web con [az webapp create](https://docs.microsoft.com/cli/azure/webapp?view=azure-cli-latest#az-webapp-create) en los planes de App Service en las regiones de Azure *Este de EE. UU.* y *Oeste de Europa*.

En el siguiente ejemplo, reemplace **<app1name_eastus>** y **<app2name_westeurope>** por un nombre de aplicación único y **<appspname_eastus>** y **<appspname_westeurope>** , por el nombre que usó para crear los planes de App Service en la sección anterior.

```azurecli-interactive

az webapp create \
    --name <app1name_eastus> \
    --plan <appspname_eastus> \
    --resource-group myResourceGroup

az webapp create \
    --name <app2name_westeurope> \
    --plan <appspname_westeurope> \
    --resource-group myResourceGroup

```

## <a name="add-traffic-manager-endpoints"></a>Incorporación de puntos de conexión de Traffic Manager
Agregue las dos aplicaciones web como puntos de conexión de Traffic Manager mediante [az network traffic-manager endpoint create](https://docs.microsoft.com/cli/azure/network/traffic-manager/endpoint?view=azure-cli-latest#az-network-traffic-manager-endpoint-create) al perfil de Traffic Manager como se describe a continuación:

- Determine el identificador de la aplicación web y agregue la aplicación web ubicada e la región *Este de EE. UU.* de Azure como punto de conexión principal para enrutar todo el tráfico de usuarios. 
- Determine el identificador de la aplicación web y agregue la aplicación web que se encuentra en la región de Azure *Oeste de Europa* como punto de conexión de conmutación por error. 

Cuando el punto de conexión principal no está disponible, el tráfico se enruta automáticamente al punto de conexión de conmutación por error.

En el ejemplo siguiente, reemplace **< app1name_eastus>** y **< app2name_westeurope>** por los nombres de aplicación creados para cada región en la sección anterior. Luego, reemplace **<profile_name>** por el nombre del perfil usado en la sección anterior. 

**Punto de conexión del Este de EE. UU.**

```azurecli-interactive

az webapp show \
    --name <app1name_eastus> \
    --resource-group myResourceGroup \
    --query id

```

Tome nota del identificador que se muestra en la salida y úselo en el siguiente comando para agregar el punto de conexión:

```azurecli-interactive

az network traffic-manager endpoint create \
    --name <app1name_eastus> \
    --resource-group myResourceGroup \
    --profile-name <profile_name> \
    --type azureEndpoints \
    --target-resource-id <ID from az webapp show> \
    --priority 1 \
    --endpoint-status Enabled
```

**Punto de conexión del Oeste de Europa**

```azurecli-interactive

az webapp show \
    --name <app2name_westeurope> \
    --resource-group myResourceGroup \
    --query id

```

Tome nota del identificador que se muestra en la salida y úselo en el siguiente comando para agregar el punto de conexión:

```azurecli-interactive

az network traffic-manager endpoint create \
    --name <app1name_westeurope> \
    --resource-group myResourceGroup \
    --profile-name <profile_name> \
    --type azureEndpoints \
    --target-resource-id <ID from az webapp show> \
    --priority 2 \
    --endpoint-status Enabled

```

## <a name="test-your-traffic-manager-profile"></a>Prueba del perfil de Traffic Manager

En esta sección, comprobará el nombre de dominio del perfil de Traffic Manager. También configurará el punto de conexión principal para que no esté disponible. Por último, podrá ver que la aplicación web sigue estando disponible. Se debe a que Traffic Manager envía el tráfico al punto de conexión de conmutación por error.

En el ejemplo siguiente, reemplace **<app1name_eastus>** y **<app2name_westeurope>** por los nombres de aplicación creados para cada región en la sección anterior. Luego, reemplace **<profile_name>** por el nombre del perfil usado en la sección anterior.

### <a name="determine-the-dns-name"></a>Determinación del nombre DNS

Determine el nombre DNS del perfil de Traffic Manager mediante [az network traffic-manager profile show](https://docs.microsoft.com/cli/azure/network/traffic-manager/profile?view=azure-cli-latest#az-network-traffic-manager-profile-show).

```azurecli-interactive

az network traffic-manager profile show \
    --name <profile_name> \
    --resource-group myResourceGroup \
    --query dnsConfig.fqdn

```

Copie el valor **RelativeDnsName**. El nombre DNS del perfil de Traffic Manager es *http://<* relativednsname *>.trafficmanager.net*. 

### <a name="view-traffic-manager-in-action"></a>Ver a Traffic Manager en acción
1. En un explorador web, escriba el nombre DNS del perfil de Traffic Manager (*http://<* relativednsname *>.trafficmanager.net*) para ver el sitio web predeterminado de la aplicación web.

    > [!NOTE]
    > En el escenario de esta guía de inicio rápido, todas las solicitudes se enrutan al punto de conexión principal. Se ha establecido en **Prioridad 1**.
2. Para ver la conmutación por error de Traffic Manager en acción, deshabilite el sitio principal mediante [az network traffic-manager endpoint update](https://docs.microsoft.com/cli/azure/network/traffic-manager/endpoint?view=azure-cli-latest#az-network-traffic-manager-endpoint-update).

   ```azurecli-interactive

    az network traffic-manager endpoint update \
        --name <app1name_eastus> \
        --resource-group myResourceGroup \
        --profile-name <profile_name> \
        --type azureEndpoints \
        --endpoint-status Disabled
    
   ```

3. Copie el nombre DNS del perfil de Traffic Manager (*http://<* relativednsname *>.trafficmanager.net*) para ver el sitio web en una nueva sesión del explorador web.
4. Compruebe que la aplicación web sigue estando disponible.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, elimine los grupos de recursos, las aplicaciones web y todos los recursos relacionados mediante [az group delete](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest#az-group-delete).

```azurecli-interactive

az group delete \
    --resource-group myResourceGroup

```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un perfil de Traffic Manager que ofrece alta disponibilidad para la aplicación web. Para más información sobre el enrutamiento del tráfico, continúe con los tutoriales de Traffic Manager.

> [!div class="nextstepaction"]
> [Tutoriales de Traffic Manager](tutorial-traffic-manager-improve-website-response.md)
