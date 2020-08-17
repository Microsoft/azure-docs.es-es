---
title: Hospedaje de varios sitios web con la CLI
titleSuffix: Azure Application Gateway
description: Aprenda a crear una puerta de enlace de aplicaciones que hospede varios sitios web mediante la CLI de Azure.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/13/2019
ms.author: victorh
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 40761b8b187d864c7b93b8aa4ee49233683fcad7
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502770"
---
# <a name="create-an-application-gateway-that-hosts-multiple-web-sites-using-the-azure-cli"></a>Creación de una puerta de enlace de aplicaciones que hospede varios sitios web mediante la CLI de Azure

Puede usar la CLI de Azure para [configurar el hospedaje de varios sitios web](multiple-site-overview.md) cuando se crea una [puerta de enlace de aplicaciones](overview.md). En este artículo se definen grupos de direcciones de back-end mediante conjuntos de escalado de máquinas virtuales. Después, configurará agentes de escucha y reglas basados en los dominios que posee para asegurarse de que el tráfico web llega a los servidores adecuados en los grupos. En este artículo se da por supuesto que posee varios dominios y se van a utilizar los ejemplos de *www\.contoso.com* y *www\.fabrikam.com*.

En este artículo aprenderá a:

> [!div class="checklist"]
> * Configuración de la red
> * Creación de una puerta de enlace de aplicaciones
> * Crear un agente de escucha de back-end
> * Crear reglas de enrutamiento
> * Crear conjuntos de escalado de máquinas virtuales con los grupos de back-end
> * Creación de un registro CNAME en el dominio

:::image type="content" source="./media/tutorial-multiple-sites-cli/scenario.png" alt-text="Instancia de Application Gateway multisitio":::

Si lo prefiere, puede realizar los pasos de este procedimiento mediante [Azure PowerShell](tutorial-multiple-sites-powershell.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este artículo es preciso que ejecute la versión 2.0.4 o posterior de la CLI de Azure. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Para crear un grupo de recursos, use [az group create](/cli/azure/group).

En el ejemplo siguiente, se crea un grupo de recursos llamado *myResourceGroupAG* en la ubicación *eastus*.

```azurecli-interactive
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>Crear recursos de red

Cree la red virtual y la subred denominada *myAGSubnet* mediante [az network vnet create](/cli/azure/network/vnet). A continuación, puede agregar la subred que necesitan los servidores de back-end mediante [az network vnet subnet create](/cli/azure/network/vnet/subnet). Cree la dirección IP pública llamada *myAGPublicIPAddress* mediante [az network public-ip create](/cli/azure/network/public-ip).

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

## <a name="create-the-application-gateway"></a>Creación de la puerta de enlace de aplicaciones

Puede usar [az network application-gateway create](/cli/azure/network/application-gateway#az-network-application-gateway-create) para crear la puerta de enlace de aplicaciones. Cuando se crea una puerta de enlace de aplicaciones mediante la CLI de Azure, se especifica información de configuración, como capacidad, SKU y HTTP. La puerta de enlace de aplicaciones se asigna a los elementos *myAGSubnet* y *myAGPublicIPAddress* creados anteriormente. 

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

La puerta de enlace de aplicaciones puede tardar varios minutos en crearse. Después de crear la puerta de enlace de aplicaciones, puede ver estas nuevas características de ella:

- *appGatewayBackendPool*: una puerta de enlace de aplicaciones debe tener al menos un grupo de direcciones de servidores back-end.
- *appGatewayBackendHttpSettings*: especifica que se use el puerto 80 y un protocolo HTTP para la comunicación.
- *appGatewayHttpListener*: agente de escucha predeterminado asociado con *appGatewayBackendPool*.
- *appGatewayFrontendIP*: asigna *myAGPublicIPAddress* a *appGatewayHttpListener*.
- *rule1*: la regla de enrutamiento predeterminada asociada a *appGatewayHttpListener*.

### <a name="add-the-backend-pools"></a>Adición de grupos de back-end

Agregue los grupos de back-end necesarios para contener los servidores de back-end mediante [az network application-gateway address-pool create](/cli/azure/network/application-gateway/address-pool#az-network-application-gateway-address-pool-create).
```azurecli-interactive
az network application-gateway address-pool create \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name contosoPool

az network application-gateway address-pool create \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name fabrikamPool
```

### <a name="add-listeners"></a>Incorporación de agentes de escucha

Agregue los clientes de escucha que sean necesarios para enrutar el tráfico mediante [az network application-gateway http-listener create](/cli/azure/network/application-gateway/http-listener#az-network-application-gateway-http-listener-create).

>[!NOTE]
> Con la SKU de Application Gateway o WAF v2, también puede configurar hasta cinco nombres de host por cliente de escucha y puede usar caracteres comodín en el nombre de host. Consulte los [nombres de host comodín en el cliente de escucha](multiple-site-overview.md#wildcard-host-names-in-listener-preview) para obtener más información.
>Para usar varios nombres de host y caracteres comodín en un cliente de escucha mediante la CLI de Azure, debe usar `--host-names` en lugar de `--host-name`. Con los nombres de host, puede mencionar hasta cinco nombres de host como valores separados por comas. Por ejemplo: `--host-names "*.contoso.com,*.fabrikam.com"`

```azurecli-interactive
az network application-gateway http-listener create \
  --name contosoListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port appGatewayFrontendPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway \
  --host-name www.contoso.com

az network application-gateway http-listener create \
  --name fabrikamListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port appGatewayFrontendPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway \
  --host-name www.fabrikam.com   
  ```

### <a name="add-routing-rules"></a>Incorporación de reglas de enrutamiento

Las reglas se procesan en el orden en que aparecen. El tráfico se dirige mediante la primera regla que coincide, con independencia de la especificidad. Por ejemplo, si tiene una regla que usa un agente de escucha básico y una regla que usa un agente de escucha multisitio en el mismo puerto, la regla con el agente de escucha multisitio debe aparecer antes que la regla con el agente de escucha básico para que la regla multisitio funcione según lo previsto. 

En este ejemplo, va a crear dos reglas nuevas y a eliminar la regla predeterminada creada cuando implementó la puerta de enlace de aplicaciones. Puede agregar la regla mediante [az network application-gateway rule create](/cli/azure/network/application-gateway/rule#az-network-application-gateway-rule-create).

```azurecli-interactive
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name contosoRule \
  --resource-group myResourceGroupAG \
  --http-listener contosoListener \
  --rule-type Basic \
  --address-pool contosoPool

az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name fabrikamRule \
  --resource-group myResourceGroupAG \
  --http-listener fabrikamListener \
  --rule-type Basic \
  --address-pool fabrikamPool

az network application-gateway rule delete \
  --gateway-name myAppGateway \
  --name rule1 \
  --resource-group myResourceGroupAG
```

## <a name="create-virtual-machine-scale-sets"></a>Creación de conjuntos de escalado de máquinas virtuales

En este ejemplo, creará tres conjuntos de escalado de máquinas virtuales que admiten los tres grupos de back-end en la puerta de enlace de aplicaciones. Los conjuntos de escalado que crea se llaman *myvmss1*, *myvmss2* y *myvmss3*. Cada conjunto de escalado contiene dos instancias de máquina virtual en las que se instala IIS.

```azurecli-interactive
for i in `seq 1 2`; do

  if [ $i -eq 1 ]
  then
    poolName="contosoPool"
  fi

  if [ $i -eq 2 ]
  then
    poolName="fabrikamPool"
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
for i in `seq 1 2`; do

  az vmss extension set \
    --publisher Microsoft.Azure.Extensions \
    --version 2.0 \
    --name CustomScript \
    --resource-group myResourceGroupAG \
    --vmss-name myvmss$i \
    --settings '{ "fileUris": ["https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/install_nginx.sh"],
  "commandToExecute": "./install_nginx.sh" }'

done
```

## <a name="create-a-cname-record-in-your-domain"></a>Creación de un registro CNAME en el dominio

Después de crear la puerta de enlace de aplicaciones con la dirección IP pública, puede obtener la dirección DNS y usarla para crear un registro CNAME en el dominio. Para obtener la dirección DNS de la puerta de enlace de aplicaciones, puede usar [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show). Copie el valor de *fqdn* de DNSSettings y úselo como valor del registro CNAME que creó. 

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [dnsSettings.fqdn] \
  --output tsv
```

No se recomienda el uso de registros A, ya que la IP virtual puede cambiar al reiniciarse la puerta de enlace de aplicaciones.

## <a name="test-the-application-gateway"></a>Prueba de la puerta de enlace de aplicaciones

Escriba el nombre de dominio en la barra de direcciones del explorador. Por ejemplo, http:\//www.contoso.com.

![Prueba del sitio de contoso en la puerta de enlace de aplicaciones](./media/tutorial-multiple-sites-cli/application-gateway-nginxtest1.png)

Cambie la dirección al otro dominio. Debería ver algo parecido al ejemplo siguiente:

![Prueba del sitio de fabrikam en la puerta de enlace de aplicaciones](./media/tutorial-multiple-sites-cli/application-gateway-nginxtest2.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, quite el grupo de recursos, la puerta de enlace de aplicaciones y todos los recursos relacionados.

```azurecli-interactive
az group delete --name myResourceGroupAG
```

## <a name="next-steps"></a>Pasos siguientes

[Crear una puerta de enlace de aplicaciones con reglas de enrutamiento basadas en rutas de direcciones URL](./tutorial-url-route-cli.md)
