---
title: 'Inicio rápido: Tráfico web directo mediante CLI'
titleSuffix: Azure Application Gateway
description: En esta guía de inicio rápido, obtendrá información sobre cómo utilizar la CLI de Azure para crear una instancia de Azure Application Gateway que dirija el tráfico web a las máquinas virtuales de un grupo de back-end.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: quickstart
ms.date: 08/27/2020
ms.author: victorh
ms.custom: mvc, devx-track-javascript, devx-track-azurecli
ms.openlocfilehash: b6b2077a71870d619c023b0d66c37ebc043aa3f3
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88961761"
---
# <a name="quickstart-direct-web-traffic-with-azure-application-gateway---azure-cli"></a>Inicio rápido: Dirección del tráfico web con Azure Application Gateway: CLI de Azure

En este inicio rápido, usará la CLI de Azure para crear una puerta de enlace de aplicaciones. Luego, lo probará para asegurarse de que funciona correctamente. 

La puerta de enlace de aplicaciones dirige el tráfico web de la aplicación a recursos específicos de un grupo de back-end. Se asignan escuchas a los puertos, se crean reglas y se agregan recursos a un grupo de back-end. Para simplificar, en este artículo se usa una configuración sencilla con una dirección IP de front-end pública, una escucha básica que hospede un único sitio en la puerta de enlace de aplicaciones, una regla de enrutamiento de solicitudes básica y dos máquinas virtuales que se usan con el grupo de back-end.

Este inicio rápido también puede completar mediante [Azure PowerShell](quick-create-powershell.md) o [Azure Portal](quick-create-portal.md).

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [CLI de Azure, versión 2.0.4 o posterior](/cli/azure/install-azure-cli) (si la CLI de Azure se ejecuta localmente).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]



## <a name="create-resource-group"></a>Creación de un grupo de recursos

En Azure, puede asignar recursos relacionados a un grupo de recursos. Cree un grupo de recursos mediante `az group create`. 

En el ejemplo siguiente, se crea un grupo de recursos llamado *myResourceGroupAG* en la ubicación *eastus*.

```azurecli-interactive 
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>Crear recursos de red 

Para que Azure se comunique entre los recursos que se crean, se necesita una red virtual.  La subred de la puerta de enlace de aplicaciones solo puede contener puertas de enlace de aplicaciones. No se permite ningún otro recurso.  Puede crear una subred para Application Gateway o usar una existente. En este ejemplo se crean dos subredes: una para la puerta de enlace de aplicaciones y la otra para los servidores back-end. Puede configurar la dirección IP de front-end de Application Gateway para que sea pública o privada, según el caso de uso. En este ejemplo, elegirá una dirección IP de front-end pública.

Para crear la red y la subred virtuales, use `az network vnet create`. Ejecute `az network public-ip create` para crear la dirección IP pública.

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
  --vnet-name myVNet   \
  --address-prefix 10.0.2.0/24
az network public-ip create \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --allocation-method Static \
  --sku Standard
```

## <a name="create-the-backend-servers"></a>Creación de servidores back-end

Un servidor back-end puede constar de NIC, conjuntos de escalado de máquinas virtuales, direcciones IP públicas e internas, nombres de dominio completos (FQDN) y servidores back-end multiinquilino, como Azure App Service. En este ejemplo, se crearán dos máquinas virtuales que se usarán como servidores back-end para la puerta de enlace de aplicaciones. También puede instalar IIS en las máquinas virtuales para probar la puerta de enlace de aplicaciones.

#### <a name="create-two-virtual-machines"></a>Creación de dos máquinas virtuales

Instale el servidor web NGINX en las máquinas virtuales para comprobar que la puerta de enlace de aplicaciones se ha creado correctamente. Puede usar un archivo de configuración cloud-init para instalar NGINX y ejecutar una aplicación Node.js "Hola mundo" en una máquina virtual Linux. Para más información sobre cloud-init, consulte [Compatibilidad con cloud-init para máquinas virtuales en Azure](../virtual-machines/linux/using-cloud-init.md).

En Azure Cloud Shell, copie y pegue la siguiente configuración en un archivo llamado *cloud-init.txt*. Escriba *editor cloud-init.txt* para crear el archivo.

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - nodejs
  - npm
write_files:
  - owner: www-data:www-data
  - path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 80;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
      }
  - owner: azureuser:azureuser
  - path: /home/azureuser/myapp/index.js
    content: |
      var express = require('express')
      var app = express()
      var os = require('os');
      app.get('/', function (req, res) {
        res.send('Hello World from host ' + os.hostname() + '!')
      })
      app.listen(3000, function () {
        console.log('Hello world app listening on port 3000!')
      })
runcmd:
  - service nginx restart
  - cd "/home/azureuser/myapp"
  - npm init
  - npm install express -y
  - nodejs index.js
```

Cree las interfaces de red con `az network nic create`. Para crear las máquinas virtuales, use `az vm create`.

```azurecli-interactive
for i in `seq 1 2`; do
  az network nic create \
    --resource-group myResourceGroupAG \
    --name myNic$i \
    --vnet-name myVNet \
    --subnet myBackendSubnet
  az vm create \
    --resource-group myResourceGroupAG \
    --name myVM$i \
    --nics myNic$i \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt
done
```

## <a name="create-the-application-gateway"></a>Creación de la puerta de enlace de aplicaciones

Cree una puerta de enlace de aplicaciones mediante `az network application-gateway create`. Cuando se crea una puerta de enlace de aplicaciones mediante la CLI de Azure, se especifica información de configuración, como capacidad, SKU y HTTP. Posteriormente, Azure agrega las direcciones IP privadas de las interfaces de red como servidores en el grupo de servidores back-end de la puerta de enlace de aplicaciones.

```azurecli-interactive
address1=$(az network nic show --name myNic1 --resource-group myResourceGroupAG | grep "\"privateIpAddress\":" | grep -oE '[^ ]+$' | tr -d '",')
address2=$(az network nic show --name myNic2 --resource-group myResourceGroupAG | grep "\"privateIpAddress\":" | grep -oE '[^ ]+$' | tr -d '",')
az network application-gateway create \
  --name myAppGateway \
  --location eastus \
  --resource-group myResourceGroupAG \
  --capacity 2 \
  --sku Standard_v2 \
  --http-settings-cookie-based-affinity Enabled \
  --public-ip-address myAGPublicIPAddress \
  --vnet-name myVNet \
  --subnet myAGSubnet \
  --servers "$address1" "$address2"
```

La puerta de enlace de aplicaciones puede tardar hasta 30 minutos en crearse. Una vez creada, puede ver los siguientes valores en la sección **Configuración** de la página **Puerta de enlace de aplicaciones**:

- **appGatewayBackendPool**: Se encuentra en la página **Grupos de back-end**. Especifica el grupo de back-end necesario.
- **appGatewayBackendHttpSettings**: Se encuentra en la página **Configuración HTTP**. Este especifica que la puerta de enlace de aplicaciones usa el puerto 80 y el protocolo HTTP para la comunicación.
- **appGatewayHttpListener**: Se encuentra en la **página de Agentes de escucha**. Este valor, especifica el agente de escucha predeterminado asociado con **appGatewayBackendPool**.
- **appGatewayFrontendIP**: Se encuentra en la página **Configuraciones de IP de front-end**. Asigna *myAGPublicIPAddress* a **appGatewayHttpListener**.
- **rule1**: Se encuentra en la página **Reglas**. Especifica la regla de enrutamiento predeterminada asociada a **appGatewayHttpListener**.

## <a name="test-the-application-gateway"></a>Prueba de la puerta de enlace de aplicaciones

Aunque Azure no necesita un servidor web NGINX para crear la puerta de enlace de aplicaciones, lo instaló en este inicio rápido para comprobar si Azure la creaba correctamente. Para obtener la dirección IP pública de la nueva puerta de enlace de aplicaciones, use `az network public-ip show`. 

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [ipAddress] \
  --output tsv
```

Copie la dirección IP pública y péguela en la barra de direcciones del explorador.
    
![Prueba de la puerta de enlace de aplicaciones](./media/quick-create-cli/application-gateway-nginxtest.png)

Al actualizar el explorador, verá aparecer el nombre de la segunda máquina virtual. Esto indica que la puerta de enlace de aplicaciones se ha creado correctamente y se puede conectar con el servidor back-end.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que creó con la puerta de enlace de aplicaciones, use el comando `az group delete` para eliminar el grupo de recursos. Al hacerlo, también elimina la puerta de enlace de aplicaciones y todos sus recursos relacionados.

```azurecli-interactive 
az group delete --name myResourceGroupAG
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Administrar el tráfico web con Application Gateway mediante la CLI de Azure](./tutorial-manage-web-traffic-cli.md)

