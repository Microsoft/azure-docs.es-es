---
title: 'Terminación TLS mediante la CLI: Azure Application Gateway'
description: Aprenda a crear una puerta de enlace de aplicaciones y a agregar un certificado para la terminación TLS mediante la CLI de Azure.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/14/2019
ms.author: victorh
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: cf6cfdc00d38b72f292f2b3ed41a633fb9cf989d
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502739"
---
# <a name="create-an-application-gateway-with-tls-termination-using-the-azure-cli"></a>Creación de una puerta de enlace de aplicaciones con terminación TLS mediante la CLI de Azure

Puede usar la CLI de Azure para crear una [puerta de enlace de aplicaciones](overview.md) con un certificado para la [terminación TLS](ssl-overview.md). En el caso de los servidores back-end, puede usar un [conjunto de escalado de máquinas virtuales](../virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md). En este ejemplo, el conjunto de escalado contiene dos instancias de máquina virtual que se agregan al grupo de servidores back-end predeterminado de la puerta de enlace de aplicaciones.

En este artículo aprenderá a:

> [!div class="checklist"]
> * Creación de un certificado autofirmado
> * Configurar una red
> * Crear una puerta de enlace de aplicaciones con el certificado
> * Crear un conjunto de escalado de máquinas virtuales con el grupo de servidores back-end predeterminado

Si lo prefiere, puede realizar los pasos de este procedimiento mediante [Azure PowerShell](tutorial-ssl-powershell.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI de forma local, en este artículo necesitará la CLI de Azure versión 2.0.4 o posterior. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="create-a-self-signed-certificate"></a>Creación de un certificado autofirmado

Para su uso en producción, debe importar un certificado válido firmado por un proveedor de confianza. En este artículo, creará un certificado autofirmado y un archivo pfx con el comando openssl.

```console
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout privateKey.key -out appgwcert.crt
```

Escriba valores que tengan sentido para su certificado. Puede aceptar los valores predeterminados.

```console
openssl pkcs12 -export -out appgwcert.pfx -inkey privateKey.key -in appgwcert.crt
```

Escriba la contraseña del certificado. En este ejemplo, se usa *Azure123456!* .

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

## <a name="create-the-application-gateway"></a>Creación de la puerta de enlace de aplicaciones

Puede usar [az network application-gateway create](/cli/azure/network/application-gateway) para crear la puerta de enlace de aplicaciones. Cuando se crea una puerta de enlace de aplicaciones mediante la CLI de Azure, se especifica información de configuración, como capacidad, SKU y HTTP. 

La puerta de enlace de aplicaciones se asigna a los elementos *myAGSubnet* y *myAGPublicIPAddress* creados anteriormente. En este ejemplo, asociará el certificado que creó y su contraseña al crear la puerta de enlace de aplicaciones. 

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
  --frontend-port 443 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --public-ip-address myAGPublicIPAddress \
  --cert-file appgwcert.pfx \
  --cert-password "Azure123456!"

```

 La puerta de enlace de aplicaciones puede tardar varios minutos en crearse. Después de crear la puerta de enlace de aplicaciones, puede ver estas nuevas características de ella:

- *appGatewayBackendPool*: una puerta de enlace de aplicaciones debe tener al menos un grupo de direcciones de servidores back-end.
- *appGatewayBackendHttpSettings*: especifica que se use el puerto 80 y un protocolo HTTP para la comunicación.
- *appGatewayHttpListener*: agente de escucha predeterminado asociado con *appGatewayBackendPool*.
- *appGatewayFrontendIP*: asigna *myAGPublicIPAddress* a *appGatewayHttpListener*.
- *rule1*: la regla de enrutamiento predeterminada asociada a *appGatewayHttpListener*.

## <a name="create-a-virtual-machine-scale-set"></a>Crear un conjunto de escalado de máquinas virtuales

En este ejemplo, creará un conjunto de escalado de máquinas virtuales que proporcione servidores para el grupo de servidores back-end predeterminado en la puerta de enlace de aplicaciones. Las máquinas virtuales del conjunto de escalado están asociadas a *myBackendSubnet* y *appGatewayBackendPool*. Para crear el conjunto de escalado, puede usar [az vmss create](/cli/azure/vmss#az-vmss-create).

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

## <a name="test-the-application-gateway"></a>Prueba de la puerta de enlace de aplicaciones

Para obtener la dirección IP pública de la puerta de enlace de aplicaciones, puede usar [az network public-ip show](/cli/azure/network/public-ip).

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [ipAddress] \
  --output tsv
```

Copie la dirección IP pública y péguela en la barra de direcciones del explorador. En este ejemplo, la dirección URL es **https://52.170.203.149** .

![Advertencia de seguridad](./media/tutorial-ssl-cli/application-gateway-secure.png)

Para aceptar la advertencia de seguridad si usó un certificado autofirmado, seleccione **Detalles** y, a continuación, **Acceder a la página web**. A continuación, se muestra el sitio de NGINX protegido, como en el ejemplo siguiente:

![Prueba de la dirección URL base en la puerta de enlace de aplicaciones](./media/tutorial-ssl-cli/application-gateway-nginx.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, quite el grupo de recursos, la puerta de enlace de aplicaciones y todos los recursos relacionados.

```azurecli-interactive
az group delete --name myResourceGroupAG --location eastus
```

## <a name="next-steps"></a>Pasos siguientes

[Crear una puerta de enlace de aplicaciones que hospede varios sitios web](./tutorial-multiple-sites-cli.md)
