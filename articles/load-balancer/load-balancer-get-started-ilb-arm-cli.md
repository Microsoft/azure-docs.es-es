---
title: 'Creación de un equilibrador de carga interno: CLI de Azure'
titleSuffix: Azure Load Balancer
description: En este artículo, aprenderá a crear un equilibrador de carga interno mediante la CLI de Azure
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.custom: seodec18, devx-track-azurecli
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/02/2020
ms.author: allensu
ms.openlocfilehash: 9a1587efc07cf852fe9389cc9bf39e693373ef89
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502331"
---
# <a name="create-an-internal-load-balancer-to-load-balance-vms-using-azure-cli"></a>Creación de un equilibrador de carga interno para equilibrar la carga de las máquinas virtuales con la CLI de Azure

Este artículo muestra cómo crear un equilibrador de carga interno para equilibrar la carga de las máquinas virtuales. Para probar el equilibrador de carga, implemente dos máquinas virtuales (VM) que ejecuten un servidor Ubuntu a fin de equilibrar la carga de una aplicación web.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)] 

Si decide instalar y usar la CLI localmente, para este tutorial es preciso que ejecute la CLI de Azure versión 2.0.28 o versiones posteriores. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos con [az group create](https://docs.microsoft.com/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroupILB* en la ubicación *eastus*:

```azurecli-interactive
  az group create \
    --name myResourceGroupILB \
    --location eastus
```

## <a name="create-a-virtual-network"></a>Creación de una red virtual

Cree una red virtual llamada *myVnet* con una subred llamada *mySubnet* en *myResourceGroup* con el comando [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet).

```azurecli-interactive
  az network vnet create \
    --name myVnet \
    --resource-group myResourceGroupILB \
    --location eastus \
    --subnet-name mySubnet
```

## <a name="create-standard-load-balancer"></a>Creación de Standard Load Balancer

En esta sección se detalla cómo se pueden crear y configurar los componentes siguientes del equilibrador de carga:
  - La configuración de direcciones IP de front-end que recibe el tráfico de red entrante en el equilibrador de carga.
  - Un grupo de direcciones IP de back-end al que el grupo de servidores front-end envía el tráfico de red de carga equilibrada.
  - Un sondeo de estado que determina el estado de las instancias de máquina virtual de back-end.
  - Una regla de equilibrador de carga que define cómo se distribuye el tráfico a las máquinas virtuales.

### <a name="create-the-load-balancer"></a>Creación del equilibrador de carga

Cree un equilibrador de carga interno con [az network lb create](https://docs.microsoft.com/cli/azure/network/lb?view=azure-cli-latest) denominado **myLoadBalancer** que incluya una configuración de direcciones IP de front-end denominada **myFrontEnd**, un grupo de back-end denominado **myBackEndPool** que se asocia a una dirección IP privada **10.0.0.7**. 

Utilice `--sku basic` para crear una instancia de Load Balancer Básico. Microsoft recomienda la SKU estándar para cargas de trabajo de producción.

```azurecli-interactive
  az network lb create \
    --resource-group myResourceGroupILB \
    --name myLoadBalancer \
    --sku standard \
    --frontend-ip-name myFrontEnd \
    --private-ip-address 10.0.0.7 \
    --backend-pool-name myBackEndPool \
    --vnet-name myVnet \
    --subnet mySubnet      
```

### <a name="create-the-health-probe"></a>Creación del sondeo de estado

Un sondeo de estado comprueba todas las instancias de máquina virtual para asegurarse de que pueden recibir tráfico de red. La instancia de máquina virtual con comprobaciones de sondeo incorrectas se elimina del equilibrador de carga hasta que vuelve a estar en línea y una comprobación de sondeo determina que es correcta. Cree un sondeo de estado con [az network lb probe create](https://docs.microsoft.com/cli/azure/network/lb/probe?view=azure-cli-latest) para supervisar el estado de las máquinas virtuales. 

```azurecli-interactive
  az network lb probe create \
    --resource-group myResourceGroupILB \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80
```

### <a name="create-the-load-balancer-rule"></a>Creación de la regla de equilibrador de carga

Una regla de equilibrador de carga define la configuración de la dirección IP de front-end para el tráfico entrante y el grupo de direcciones IP de back-end para recibir el tráfico, junto con el puerto de origen y destino requeridos. Cree una regla de equilibrador de carga *myHTTPRule* con [az network lb rule create](https://docs.microsoft.com/cli/azure/network/lb/rule?view=azure-cli-latest) para escuchar el puerto 80 en el grupo de servidores front-end *myFrontEnd* y enviar tráfico de red con equilibrio de carga al conjunto de direcciones back-end *myBackEndPool*, que también usan el puerto 80. 

```azurecli-interactive
  az network lb rule create \
    --resource-group myResourceGroupILB \
    --lb-name myLoadBalancer \
    --name myHTTPRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe  
```

También puede crear una regla de equilibrador de carga de los [puertos de alta disponibilidad](load-balancer-ha-ports-overview.md) mediante la configuración siguiente con Standard Load Balancer.

```azurecli-interactive
az network lb rule create --resource-group myResourceGroupILB --lb-name myLoadBalancer --name haportsrule --protocol all --frontend-port 0 --backend-port 0 --frontend-ip-name myFrontEnd --backend-address-pool-name myBackEndPool
```

## <a name="create-servers-for-the-backend-address-pool"></a>Creación de servidores para el grupo de direcciones back-end

Antes de implementar algunas máquinas virtuales y poder probar el equilibrador de carga, cree los recursos de red virtual auxiliares.

### <a name="create-nics"></a>Creación de tarjetas NIC

Cree dos interfaces de red con [az network nic create](/cli/azure/network/nic#az-network-nic-create) y asócielas con la dirección IP privada. 

```azurecli-interactive
for i in `seq 1 2`; do
  az network nic create \
    --resource-group myResourceGroupILB \
    --name myNic$i \
    --vnet-name myVnet \
    --subnet mySubnet \
    --lb-name myLoadBalancer \
    --lb-address-pools myBackEndPool
done
```

## <a name="create-backend-servers"></a>Creación de servidores back-end

En este ejemplo, se crean dos máquinas virtuales que se usarán como servidores back-end para el equilibrador de carga. Para comprobar que el equilibrador de carga se ha creado correctamente, también se debe instalar NGINX en las máquinas virtuales.

### <a name="create-an-availability-set"></a>Creación de un conjunto de disponibilidad

Cree un conjunto de disponibilidad con [az vm availabilityset create](/cli/azure/network/nic)

```azurecli-interactive
  az vm availability-set create \
    --resource-group myResourceGroupILB \
    --name myAvailabilitySet
```

### <a name="create-two-virtual-machines"></a>Creación de dos máquinas virtuales

Puede usar un archivo de configuración cloud-init para instalar NGINX y ejecutar una aplicación Node.js "Hola mundo" en una máquina virtual Linux. En el shell actual, cree un archivo denominado cloud-init.txt, y copie y pegue la siguiente configuración en el shell. Asegúrese de copiar correctamente todo el archivo cloud-init, especialmente la primera línea:

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

Cree las máquinas virtuales con [az vm create](/cli/azure/vm#az-vm-create).

```azurecli-interactive
for i in `seq 1 2`; do
  az vm create \
    --resource-group myResourceGroupILB \
    --name myVM$i \
    --availability-set myAvailabilitySet \
    --nics myNic$i \
    --image UbuntuLTS \
    --generate-ssh-keys \
    --custom-data cloud-init.txt
    done
```

Es posible que la implementación de las máquinas virtuales tarde unos minutos.

### <a name="create-a-vm-for-testing-the-load-balancer"></a>Creación de una máquina virtual para probar el equilibrador de carga

Para probar el equilibrador de carga, cree una máquina virtual, *myVMTest* y asóciela a *myNic3*.

```azurecli-interactive
 az vm create \
    --resource-group myResourceGroupILB \
    --name myVMTest \
    --image win2016datacenter \
    --admin-username azureuser \
    --admin-password myPassword123456!
```

## <a name="test-the-internal-load-balancer"></a>Prueba del equilibrador de carga interno

Para probar el equilibrador de carga, primero debe obtener su dirección IP privada. A continuación, inicie sesión en myVMTest de la máquina virtual y escriba la dirección IP privada en la barra de direcciones del explorador web.

Para obtener la dirección IP privada del equilibrador de carga, use [az network lb show](/cli/azure/network/public-ip). Copie la dirección IP privada y, a continuación, péguela en la barra de direcciones de un explorador web de la máquina virtual: *myVMTest*.

```azurecli-interactive
  az network lb show \
    --name myLoadBalancer \
    --resource-group myResourceGroupILB
```

![Prueba del equilibrador de carga](./media/load-balancer-get-started-ilb-arm-cli/load-balancer-test.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede usar el comando [az group delete](/cli/azure/group#az-group-delete) para quitar el grupo de recursos, el equilibrador de carga y todos los recursos relacionados.

```azurecli-interactive
  az group delete --name myResourceGroupILB
```


## <a name="next-steps"></a>Pasos siguientes
En este artículo, ha creado un instancia de Load Balancer básico interno, le ha asociado las máquinas virtuales, ha configurado la regla de tráfico del equilibrador de carga, ha sondeado el estado y después ha probado el equilibrador de carga. Para más información acerca de los equilibradores de carga y sus recursos asociados, vaya a los artículos de procedimientos.
