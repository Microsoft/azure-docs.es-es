---
title: 'Redirección del tráfico web en base a la dirección URL: CLI de Azure'
description: En este artículo aprenderá a redirigir el tráfico web en base a la URL a grupos de servidores escalables específicos mediante la CLI de Azure.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 08/01/2019
ms.author: victorh
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 4aee41bc0188676ac0bd168474f1842199af9620
ms.sourcegitcommit: 1b320bc7863707a07e98644fbaed9faa0108da97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89595271"
---
# <a name="route-web-traffic-based-on-the-url-using-the-azure-cli"></a>Redirección del tráfico web en función de la dirección URL mediante la CLI de Azure

Como administrador de IT que administra el tráfico web, desea ayudar a los clientes o usuarios a obtener la información que necesitan lo más rápidamente posible. Una manera en la que puede optimizar su experiencia es mediante el enrutamiento de diferentes clases de tráfico web a diferentes recursos de servidor. Este artículo le muestra cómo usar la CLI de Azure para instalar y configurar el enrutamiento de Application Gateway para distintos tipos de tráfico de la aplicación. Posteriormente, el enrutamiento dirige el tráfico a diferentes granjas de servidores según la dirección URL.

![Ejemplo de enrutamiento de direcciones URL](./media/tutorial-url-route-cli/scenario.png)

En este artículo aprenderá a:

* Crear un grupo de recursos para los recursos de red que necesitará
* Crear los recursos de red
* Crear una puerta de enlace de aplicaciones para el tráfico que procede de la aplicación
* Especificar granjas de servidores y reglas de enrutamiento para los distintos tipos de tráfico
* Crear un conjunto de escalado para cada granja para que esta se pueda escalar de forma automática
* Ejecutar una prueba para comprobar que los distintos tipos de tráfico van a la granja correcta

Si lo prefiere, puede completar este procedimiento con [Azure PowerShell](tutorial-url-route-powershell.md) o [Azure Portal](create-url-route-portal.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI de forma local, en este artículo necesitará la CLI de Azure versión 2.0.4 o posterior. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Cree un grupo de recursos mediante `az group create`.

En el ejemplo siguiente, se crea un grupo de recursos llamado *myResourceGroupAG* en la ubicación *eastus*.

```azurecli-interactive
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>Crear recursos de red

Cree la red virtual llamada *myVNet* y la subred llamada *myAGSubnet* mediante `az network vnet create`. A continuación, agregue la subred llamada *myBackendSubnet* que necesitan los servidores de back-end mediante `az network vnet subnet create`. Cree la dirección IP pública llamada *myAGPublicIPAddress* mediante `az network public-ip create`.

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

## <a name="create-the-app-gateway-with-a-url-map"></a>Creación de la puerta de enlace de aplicaciones con una asignación de direcciones URL

Use `az network application-gateway create` para crear una puerta de enlace de aplicaciones denominada *myAppGateway*. Cuando se crea una puerta de enlace de aplicaciones mediante la CLI de Azure, se especifica información de configuración, como capacidad, SKU y HTTP. La puerta de enlace de aplicaciones se asigna a *myAGSubnet* y *myAGPublicIPAddress*.

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


|Característica  |Descripción  |
|---------|---------|
|appGatewayBackendPool     |Una puerta de enlace de aplicaciones debe tener al menos un grupo de direcciones de servidores back-end.|
|appGatewayBackendHttpSettings     |Especifica que se use el puerto 80 y un protocolo HTTP para la comunicación.|
|appGatewayHttpListener     |Agente de escucha predeterminado asociado con appGatewayBackendPool.|
|appGatewayFrontendIP     |Asigna myAGPublicIPAddress a appGatewayHttpListener.|
|rule1     |La regla de enrutamiento predeterminada asociada a appGatewayHttpListener.|

### <a name="add-image-and-video-backend-pools-and-a-port"></a>Adición de un puerto de back-end y grupos de back-end de imágenes y vídeo

Agregue los grupos de back-end denominados *imagesBackendPool* y *videoBackendPool* a la puerta de enlace de aplicaciones mediante `az network application-gateway address-pool create`. Agregue el puerto de front-end para los grupos mediante `az network application-gateway frontend-port create`.

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
  --name port8080
```

### <a name="add-a-backend-listener"></a>Adición de un agente de escucha de back-end

Agregue el agente de escucha de back-end llamado *backendListener* que es necesario para enrutar el tráfico mediante `az network application-gateway http-listener create`.


```azurecli-interactive
az network application-gateway http-listener create \
  --name backendListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port port8080 \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway
```

### <a name="add-a-url-path-map"></a>Adición de una asignación de ruta de URL

Las asignaciones correspondientes a la ruta de dirección URL garantizan que direcciones URL específicas se enruten hacia grupos de back-end específicos. Cree una asignación de rutas de URL llamada *imagePathRule* y *videoPathRule* mediante `az network application-gateway url-path-map create` y `az network application-gateway url-path-map rule create`.

```azurecli-interactive
az network application-gateway url-path-map create \
  --gateway-name myAppGateway \
  --name myPathMap \
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
  --path-map-name myPathMap \
  --paths /video/* \
  --address-pool videoBackendPool
```

### <a name="add-a-routing-rule"></a>Agregar una regla de enrutamiento

La regla de enrutamiento asocia las asignaciones de URL con el agente de escucha que ha creado. Agregue una regla denominada *rule2* mediante `az network application-gateway rule create`.

```azurecli-interactive
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name rule2 \
  --resource-group myResourceGroupAG \
  --http-listener backendListener \
  --rule-type PathBasedRouting \
  --url-path-map myPathMap \
  --address-pool appGatewayBackendPool
```

## <a name="create-virtual-machine-scale-sets"></a>Creación de conjuntos de escalado de máquinas virtuales

En este artículo, creará tres conjuntos de escalado de máquinas virtuales que admitan los tres grupos de back-end que ha creado. Cree los conjuntos de escalado denominados *myvmss1*, *myvmss2* y *myvmss3*. Cada conjunto de escalado contiene dos instancias de máquina virtual en las que se instalará NGINX.

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

Para obtener la dirección IP pública de la puerta de enlace de aplicaciones, use az network public-ip show. Copie la dirección IP pública y péguela en la barra de direcciones del explorador. Como, por ejemplo, `http://40.121.222.19`, `http://40.121.222.19:8080/images/test.htm` o `http://40.121.222.19:8080/video/test.htm`.

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [ipAddress] \
  --output tsv
```

![Prueba de la dirección URL base en la puerta de enlace de aplicaciones](./media/tutorial-url-route-cli/application-gateway-nginx.png)

Cambie la dirección URL por http://&lt;dirección-ip&gt;:8080/images/test.html, sustituyendo &lt;dirección-ip&gt; por su dirección IP y verá algo similar al ejemplo siguiente:

![Prueba de la dirección URL de imágenes en la puerta de enlace de aplicaciones](./media/tutorial-url-route-cli/application-gateway-nginx-images.png)

Cambie la dirección URL por http://&lt;dirección-ip&gt;:8080/video/test.html, sustituyendo &lt;dirección-ip&gt; por su dirección IP y verá algo similar al ejemplo siguiente.

![Prueba de la dirección URL de vídeo en la puerta de enlace de aplicaciones](./media/tutorial-url-route-cli/application-gateway-nginx-video.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, quite el grupo de recursos, la puerta de enlace de aplicaciones y todos los recursos relacionados.

```azurecli-interactive
az group delete --name myResourceGroupAG
```

## <a name="next-steps"></a>Pasos siguientes

[Crear una puerta de enlace de aplicaciones con redireccionamiento basado en rutas de dirección URL](./tutorial-url-redirect-cli.md)
