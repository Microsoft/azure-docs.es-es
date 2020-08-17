---
title: 'Tutorial: Redireccionamiento basado en rutas de URL mediante la CLI'
titleSuffix: Azure Application Gateway
description: En este tutorial aprenderá a crear una puerta de enlace de aplicaciones con tráfico redirigido basado en rutas de URL con la CLI de Azure.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: tutorial
ms.date: 11/14/2019
ms.author: victorh
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: daee09de544e12d07b28d4e88a4847f71bd2e01f
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502722"
---
# <a name="tutorial-create-an-application-gateway-with-url-path-based-redirection-using-the-azure-cli"></a>Tutorial: Creación de una puerta de enlace de aplicaciones con redirección basada en rutas de dirección URL con la CLI de Azure

Puede usar la CLI de Azure para configurar [reglas de enrutamiento basadas en rutas de dirección URL](tutorial-url-route-cli.md) cuando se crea una [puerta de enlace de aplicaciones](application-gateway-introduction.md). En este tutorial, creará grupos de back-end mediante [conjuntos de escalado de máquinas virtuales](../virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md). A continuación, creará reglas de enrutamiento de direcciones URL que garanticen que el tráfico web se redirija a los grupos de back-end adecuados.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configuración de la red
> * Creación de una puerta de enlace de aplicaciones
> * Incorporación de agentes de escucha y reglas de enrutamiento
> * Creación de conjuntos de escalado de máquinas virtuales para grupos de back-end

En el ejemplo siguiente se muestra el tráfico del sitio procedente de los puertos 8080 y 8081, y que se dirige a los mismos grupos de back-end:

![Ejemplo de enrutamiento de direcciones URL](./media/tutorial-url-redirect-cli/scenario.png)

Si lo prefiere, puede seguir los pasos de este tutorial mediante [Azure PowerShell](tutorial-url-redirect-powershell.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI de forma local, en este tutorial necesitará la CLI de Azure versión 2.0.4 o posterior. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Para crear un grupo de recursos, use [az group create](/cli/azure/group).

En el ejemplo siguiente, se crea un grupo de recursos llamado *myResourceGroupAG* en la ubicación *eastus*.

```azurecli-interactive 
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>Crear recursos de red 

Cree la red virtual llamada *myVNet* y la subred llamada *myAGSubnet* mediante [az network vnet create](/cli/azure/network/vnet). Luego, puede agregar la subred llamada *myBackendSubnet* que necesitan los servidores back-end mediante [az network vnet subnet create](/cli/azure/network/vnet/subnet). Cree la dirección IP pública llamada *myAGPublicIPAddress* mediante [az network public-ip create](/cli/azure/network/public-ip).

```azurecli-interactive
az network vnet create \
  --name myVNet \
  --resource-group myResourceGroupAG \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name myAGSubnet \
  --subnet-prefix 10.0.1.0/24

az network vnet subnet create \
  --name myBackendSubnet \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --address-prefix 10.0.2.0/24

az network public-ip create \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --allocation-method Static \
  --sku Standard
```

## <a name="create-an-application-gateway"></a>Creación de una puerta de enlace de aplicaciones

Use [az network application-gateway create](/cli/azure/network/application-gateway) para crear la puerta de enlace de aplicaciones llamada myAppGateway. Cuando se crea una puerta de enlace de aplicaciones mediante la CLI de Azure, se especifica información de configuración, como capacidad, SKU y HTTP. La puerta de enlace de aplicaciones se asigna a los elementos *myAGSubnet* y *myPublicIPAddress* que se crearon anteriormente.

```azurecli-interactive
az network application-gateway create \
  --name myAppGateway \
  --location eastus \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --subnet myAGsubnet \
  --capacity 2 \
  --sku Standard_v2 \
  --http-settings-cookie-based-affinity Disabled \
  --frontend-port 80 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --public-ip-address myAGPublicIPAddress
```

 La puerta de enlace de aplicaciones puede tardar varios minutos en crearse. Después de crear la puerta de enlace de aplicaciones, podrá ver estas nuevas características:

- *appGatewayBackendPool*: una puerta de enlace de aplicaciones debe tener al menos un grupo de direcciones de servidores back-end.
- *appGatewayBackendHttpSettings*: especifica que se use el puerto 80 y un protocolo HTTP para la comunicación.
- *appGatewayHttpListener*: agente de escucha predeterminado asociado con *appGatewayBackendPool*.
- *appGatewayFrontendIP*: asigna *myAGPublicIPAddress* a *appGatewayHttpListener*.
- *rule1*: la regla de enrutamiento predeterminada asociada a *appGatewayHttpListener*.


### <a name="add-backend-pools-and-ports"></a>Incorporación de grupos de back-end y puertos

Puede agregar los grupos de direcciones de back-end llamados *imagesBackendPool* y *videoBackendPool* a la puerta de enlace de aplicaciones mediante [az network application-gateway address-pool create](/cli/azure/network/application-gateway/address-pool). Los puertos de front-end de los grupos se agregan mediante [az network application-gateway frontend-port create](/cli/azure/network/application-gateway/frontend-port). 

```azurecli-interactive
az network application-gateway address-pool create \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name imagesBackendPool

az network application-gateway address-pool create \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name videoBackendPool

az network application-gateway frontend-port create \
  --port 8080 \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name bport

az network application-gateway frontend-port create \
  --port 8081 \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name rport
```

## <a name="add-listeners-and-rules"></a>Incorporación de reglas y agentes de escucha

### <a name="add-listeners"></a>Incorporación de agentes de escucha

Agregue los agentes de escucha de backend llamados *backendListener* y *redirectedListener* que son necesarios para enrutar el tráfico con [az network application-gateway http-listener create](/cli/azure/network/application-gateway/http-listener).


```azurecli-interactive
az network application-gateway http-listener create \
  --name backendListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port bport \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway

az network application-gateway http-listener create \
  --name redirectedListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port rport \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway
```

### <a name="add-the-default-url-path-map"></a>Incorporación de la asignación de ruta de URL predeterminada

Las asignaciones de ruta de URL se aseguran de que las direcciones URL específicas se enrutan a grupos de back-end específicos. Puede crear las asignaciones de ruta de URL llamadas *imagePathRule* y *videoPathRule* mediante [az network application-gateway url-path-map create](/cli/azure/network/application-gateway/url-path-map) y [az network application-gateway url-path-map rule create](/cli/azure/network/application-gateway/url-path-map/rule)

```azurecli-interactive
az network application-gateway url-path-map create \
  --gateway-name myAppGateway \
  --name urlpathmap \
  --paths /images/* \
  --resource-group myResourceGroupAG \
  --address-pool imagesBackendPool \
  --default-address-pool appGatewayBackendPool \
  --default-http-settings appGatewayBackendHttpSettings \
  --http-settings appGatewayBackendHttpSettings \
  --rule-name imagePathRule

az network application-gateway url-path-map rule create \
  --gateway-name myAppGateway \
  --name videoPathRule \
  --resource-group myResourceGroupAG \
  --path-map-name urlpathmap \
  --paths /video/* \
  --address-pool videoBackendPool
```

### <a name="add-redirection-configuration"></a>Incorporación de la configuración de redireccionamiento

Puede configurar la redirección del agente de escucha mediante [az network application-gateway redirect-config create](/cli/azure/network/application-gateway/redirect-config).

```azurecli-interactive
az network application-gateway redirect-config create \
  --gateway-name myAppGateway \
  --name redirectConfig \
  --resource-group myResourceGroupAG \
  --type Found \
  --include-path true \
  --include-query-string true \
  --target-listener backendListener
```

### <a name="add-the-redirection-url-path-map"></a>Incorporación de la asignación de ruta de URL de redireccionamiento

```azurecli-interactive
az network application-gateway url-path-map create \
  --gateway-name myAppGateway \
  --name redirectpathmap \
  --paths /images/* \
  --resource-group myResourceGroupAG \
  --redirect-config redirectConfig \
  --rule-name redirectPathRule
```

### <a name="add-routing-rules"></a>Incorporación de reglas de enrutamiento

Las reglas de enrutamiento asocian las asignaciones de ruta de URL con los agentes de escucha que ha creado. Puede agregar las reglas llamadas *defaultRule* y *redirectedRule* mediante [az network application-gateway rule create](/cli/azure/network/application-gateway/rule).

```azurecli-interactive
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name defaultRule \
  --resource-group myResourceGroupAG \
  --http-listener backendListener \
  --rule-type PathBasedRouting \
  --url-path-map urlpathmap \
  --address-pool appGatewayBackendPool

az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name redirectedRule \
  --resource-group myResourceGroupAG \
  --http-listener redirectedListener \
  --rule-type PathBasedRouting \
  --url-path-map redirectpathmap \
  --address-pool appGatewayBackendPool
```

## <a name="create-virtual-machine-scale-sets"></a>Creación de conjuntos de escalado de máquinas virtuales

En este ejemplo, creará tres conjuntos de escalado de máquinas virtuales que admiten los tres grupos de back-end que ha creado. Los conjuntos de escalado que crea se llaman *myvmss1*, *myvmss2* y *myvmss3*. Cada conjunto de escalado contiene dos instancias de máquina virtual en las que se instalará NGINX.

```azurecli-interactive
for i in `seq 1 3`; do
  if [ $i -eq 1 ]
  then
    poolName="appGatewayBackendPool" 
  fi
  if [ $i -eq 2 ]
  then
    poolName="imagesBackendPool"
  fi
  if [ $i -eq 3 ]
  then
    poolName="videoBackendPool"
  fi

  az vmss create \
    --name myvmss$i \
    --resource-group myResourceGroupAG \
    --image UbuntuLTS \
    --admin-username azureuser \
    --admin-password Azure123456! \
    --instance-count 2 \
    --vnet-name myVNet \
    --subnet myBackendSubnet \
    --vm-sku Standard_DS2 \
    --upgrade-policy-mode Automatic \
    --app-gateway myAppGateway \
    --backend-pool-name $poolName
done
```

### <a name="install-nginx"></a>Instalación de NGINX

```azurecli-interactive
for i in `seq 1 3`; do
  az vmss extension set \
    --publisher Microsoft.Azure.Extensions \
    --version 2.0 \
    --name CustomScript \
    --resource-group myResourceGroupAG \
    --vmss-name myvmss$i \
    --settings '{ "fileUris": ["https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/install_nginx.sh"], "commandToExecute": "./install_nginx.sh" }'

done
```

## <a name="test-the-application-gateway"></a>Prueba de la puerta de enlace de aplicaciones

Para obtener la dirección IP pública de la puerta de enlace de aplicaciones, use [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show). Copie la dirección IP pública y péguela en la barra de direcciones del explorador. Como, por ejemplo, `http://40.121.222.19`, `http://40.121.222.19:8080/images/test.htm`, `http://40.121.222.19:8080/video/test.htm` o `http://40.121.222.19:8081/images/test.htm`.

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [ipAddress] \
  --output tsv
```

![Prueba de la dirección URL base en la puerta de enlace de aplicaciones](./media/tutorial-url-redirect-cli/application-gateway-nginx.png)

Cambie la dirección URL por http://&lt;dirección-ip&gt;:8080/images/test.html, sustituyendo &lt;dirección-ip&gt; por su dirección IP y verá algo similar al ejemplo siguiente:

![Prueba de la dirección URL de imágenes en la puerta de enlace de aplicaciones](./media/tutorial-url-redirect-cli/application-gateway-nginx-images.png)

Cambie la dirección URL por http://&lt;ip-address&gt;:8080/video/test.html, sustituyendo &lt;ip-address&gt; por su dirección IP y verá algo similar al ejemplo siguiente:

![Prueba de la dirección URL de vídeo en la puerta de enlace de aplicaciones](./media/tutorial-url-redirect-cli/application-gateway-nginx-video.png)

Ahora, cambie la dirección URL por http://&lt;ip-address&gt;:8081/images/test.htm, sustituyendo &lt;ip-address&gt; por su dirección IP, y debería ver el tráfico redirigido al grupo de back-end de imágenes en http://&lt;ip-address&gt;:8080/images.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, quite el grupo de recursos, la puerta de enlace de aplicaciones y todos los recursos relacionados.

```azurecli-interactive
az group delete --name myResourceGroupAG
```
## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Obtenga más información sobre lo que puede hacer con la puerta de enlace de aplicaciones](application-gateway-introduction.md).
