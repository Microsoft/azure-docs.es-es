---
title: Redireccionamiento interno mediante la CLI
titleSuffix: Azure Application Gateway
description: Obtenga información sobre cómo crear una puerta de enlace de aplicaciones que redirija el tráfico web interno al grupo adecuado mediante la CLI de Azure.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/14/2019
ms.author: victorh
ms.openlocfilehash: d0730148a3da41d0d72961ea521577fa370b183d
ms.sourcegitcommit: 1b320bc7863707a07e98644fbaed9faa0108da97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89596097"
---
# <a name="create-an-application-gateway-with-internal-redirection-using-the-azure-cli"></a>Creación de una puerta de enlace de aplicaciones con redireccionamiento interno mediante la CLI de Azure

Puede usar la CLI de Azure para configurar el [redireccionamiento del tráfico web](multiple-site-overview.md) al crear una [puerta de enlace de aplicaciones](overview.md). En este tutorial, definirá un grupo de back-end mediante un conjunto de escalado de máquinas virtuales. A continuación, configurará reglas y agentes de escucha basados en dominios de su propiedad para asegurarse de que el tráfico web llega al grupo adecuado. En este tutorial se da por supuesto que posee varios dominios y se usan ejemplos de *www\.contoso.com* y *www\.contoso.org*.

En este artículo aprenderá a:

* Configuración de la red
* Creación de una puerta de enlace de aplicaciones
* Incorporación de agentes de escucha y una regla de redireccionamiento
* Creación de un conjunto de escalado de máquinas virtuales con el grupo de back-end
* Creación de un registro CNAME en el dominio

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para esta guía de inicio rápido es preciso que ejecute la CLI de Azure versión 2.0.4 o posterior. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Para crear un grupo de recursos, use [az group create](/cli/azure/group).

En el ejemplo siguiente, se crea un grupo de recursos llamado *myResourceGroupAG* en la ubicación *eastus*.

```azurecli-interactive 
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>Crear recursos de red 

Cree la red virtual llamada *myVNet* y la subred llamada *myAGSubnet* mediante [az network vnet create](/cli/azure/network/vnet). Luego, puede agregar la subred llamada *myBackendSubnet* que necesita el grupo de back-end de servidores mediante [az network vnet subnet create](/cli/azure/network/vnet/subnet). Cree la dirección IP pública llamada *myAGPublicIPAddress* mediante [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create).

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
  --name myAGPublicIPAddress
```

## <a name="create-an-application-gateway"></a>Creación de una puerta de enlace de aplicaciones

Puede usar [az network application-gateway create](/cli/azure/network/application-gateway) para crear la puerta de enlace de aplicaciones llamada *myAppGateway*. Cuando se crea una puerta de enlace de aplicaciones mediante la CLI de Azure, se especifica información de configuración, como capacidad, SKU y HTTP. La puerta de enlace de aplicaciones se asigna a los elementos *myAGSubnet* y *myAGPublicIPAddress* creados anteriormente. 

```azurecli-interactive
az network application-gateway create \
  --name myAppGateway \
  --location eastus \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --subnet myAGsubnet \
  --capacity 2 \
  --sku Standard_Medium \
  --http-settings-cookie-based-affinity Disabled \
  --frontend-port 80 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --public-ip-address myAGPublicIPAddress
```

La puerta de enlace de aplicaciones puede tardar varios minutos en crearse. Después de crear la puerta de enlace de aplicaciones, puede ver estas nuevas características de ella:

- *appGatewayBackendPool*: una puerta de enlace de aplicaciones debe tener al menos un grupo de direcciones de servidores back-end.
- *appGatewayBackendHttpSettings*: especifica que se use el puerto 80 y un protocolo HTTP para la comunicación.
- *appGatewayHttpListener*: agente de escucha predeterminado asociado con *appGatewayBackendPool*.
- *appGatewayFrontendIP*: asigna *myAGPublicIPAddress* a *appGatewayHttpListener*.
- *rule1*: la regla de enrutamiento predeterminada asociada a *appGatewayHttpListener*.


## <a name="add-listeners-and-rules"></a>Incorporación de reglas y agentes de escucha 

Es necesario un agente de escucha para que la puerta de enlace de aplicaciones enrute el tráfico de forma adecuada al grupo de servidores back-end. En este tutorial, creará dos agentes de escucha para los dos dominios. En este ejemplo se crean clientes de escucha para los dominios de *www\.contoso.com* y *www\.contoso.org*.

Agregue los agentes de escucha que sean necesarios para enrutar el tráfico mediante [az network application-gateway http-listener create](/cli/azure/network/application-gateway/http-listener#az-network-application-gateway-http-listener-create).

```azurecli-interactive
az network application-gateway http-listener create \
  --name contosoComListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port appGatewayFrontendPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway \
  --host-name www.contoso.com
az network application-gateway http-listener create \
  --name contosoOrgListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port appGatewayFrontendPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway \
  --host-name www.contoso.org   
  ```

### <a name="add-the-redirection-configuration"></a>Adición de la configuración de redireccionamiento

Agregue la configuración de redireccionamiento que envía tráfico desde *www\.consoto.org* al cliente de escucha de *www\.contoso.com* en la puerta de enlace de aplicaciones mediante [az network application-gateway redirect-config create](/cli/azure/network/application-gateway/redirect-config#az-network-application-gateway-redirect-config-create).

```azurecli-interactive
az network application-gateway redirect-config create \
  --name orgToCom \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --type Permanent \
  --target-listener contosoListener \
  --include-path true \
  --include-query-string true
```

### <a name="add-routing-rules"></a>Incorporación de reglas de enrutamiento

Las reglas se procesan en el orden en el que se crean y el tráfico se dirige mediante la primera regla que coincida con la dirección URL que se envía a la puerta de enlace de aplicaciones. Por ejemplo, si tiene una regla que usa un agente de escucha básico y una regla que usa un agente de escucha multisitio en el mismo puerto, la regla con el agente de escucha multisitio debe aparecer antes que la regla con el agente de escucha básico para que la regla multisitio funcione según lo previsto. 

En este ejemplo se crean dos reglas nuevas y se elimina la regla predeterminada que se creó.  Puede agregar la regla mediante [az network application-gateway rule create](/cli/azure/network/application-gateway/rule#az-network-application-gateway-rule-create).

```azurecli-interactive
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name contosoComRule \
  --resource-group myResourceGroupAG \
  --http-listener contosoComListener \
  --rule-type Basic \
  --address-pool appGatewayBackendPool
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name contosoOrgRule \
  --resource-group myResourceGroupAG \
  --http-listener contosoOrgListener \
  --rule-type Basic \
  --redirect-config orgToCom
az network application-gateway rule delete \
  --gateway-name myAppGateway \
  --name rule1 \
  --resource-group myResourceGroupAG
```

## <a name="create-virtual-machine-scale-sets"></a>Creación de conjuntos de escalado de máquinas virtuales

En este ejemplo, creará un conjunto de escalado de máquinas virtuales que admite el grupo de back-end que ha creado. El conjunto de escalado que cree se llamará *myvmss* y contiene dos instancias de máquina virtual en las que se instala NGINX.

```azurecli-interactive
az vmss create \
  --name myvmss \
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
  --backend-pool-name appGatewayBackendPool
```

### <a name="install-nginx"></a>Instalación de NGINX

Ejecute este comando en la ventana de shell:

```azurecli-interactive
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroupAG \
  --vmss-name myvmss \
  --settings '{ "fileUris": ["https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/install_nginx.sh"],
  "commandToExecute": "./install_nginx.sh" }'
```

## <a name="create-cname-record-in-your-domain"></a>Creación de un registro CNAME en el dominio

Después de crear la puerta de enlace de aplicaciones con la dirección IP pública, puede obtener la dirección DNS y usarla para crear un registro CNAME en el dominio. Para obtener la dirección DNS de la puerta de enlace de aplicaciones, puede usar [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show). Copie el valor de *fqdn* de DNSSettings y úselo como valor del registro CNAME que creó. No se recomienda el uso de registros A, ya que la IP virtual puede cambiar al reiniciarse la puerta de enlace de aplicaciones.

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [dnsSettings.fqdn] \
  --output tsv
```

## <a name="test-the-application-gateway"></a>Prueba de la puerta de enlace de aplicaciones

Escriba el nombre de dominio en la barra de direcciones del explorador. Por ejemplo, http:\//www.contoso.com.

![Prueba del sitio de contoso en la puerta de enlace de aplicaciones](./media/redirect-internal-site-cli/application-gateway-nginxtest.png)

Cambie la dirección para su otro dominio, por ejemplo http:\//www.contoso.org, y verá que se ha redirigido el tráfico hacia el agente de escucha para www\.contoso.com.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> * Configuración de la red
> * Creación de una puerta de enlace de aplicaciones
> * Incorporación de agentes de escucha y una regla de redireccionamiento
> * Creación de un conjunto de escalado de máquinas virtuales con el grupo de back-end
> * Creación de un registro CNAME en el dominio
