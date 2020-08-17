---
title: 'Tutorial: Equilibrio de carga de máquinas virtuales Linux en Azure'
description: En este tutorial, aprenderá a utilizar la CLI de Azure para crear un equilibrador de carga para una aplicación segura y de alta disponibilidad entre tres máquinas virtuales Linux
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: gwallace
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 11/13/2017
ms.author: cynthn
ms.custom: mvc, devx-track-javascript, devx-track-azurecli
ms.openlocfilehash: 82f1a803935c69835906155cf0d522e74d3d2fd6
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502501"
---
# <a name="tutorial-load-balance-linux-virtual-machines-in-azure-to-create-a-highly-available-application-with-the-azure-cli"></a>Tutorial: Equilibrio de carga de máquinas virtuales Linux en Azure para crear una aplicación de alta disponibilidad con la CLI de Azure

El equilibrio de carga proporciona un mayor nivel de disponibilidad al distribuir las solicitudes entrantes entre varias máquinas virtuales. En este tutorial, aprenderá sobre los distintos componentes de Azure Load Balancer que distribuyen el tráfico y proporcionan una alta disponibilidad. Aprenderá a:

> [!div class="checklist"]
> * Creación de un equilibrador de carga de Azure
> * Creación del sondeo de estado de un equilibrador de carga
> * Crear reglas de tráfico del equilibrador de carga
> * Usar cloud-init para crear una aplicación básica de Node.js
> * Crear máquinas virtuales y conectarlas a un equilibrador de carga
> * Ver un equilibrador de carga en acción
> * Agregar y quitar las máquinas virtuales de un equilibrador de carga

En este tutorial se usa la CLI dentro de [Azure Cloud Shell](../../cloud-shell/overview.md), que se actualiza constantemente a la versión más reciente. Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior de cualquier bloque de código.

Si decide instalar y usar la CLI localmente, en este tutorial es preciso que ejecute la CLI de Azure de la versión 2.0.30, u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="azure-load-balancer-overview"></a>Información general sobre Azure Load Balancer
Un equilibrador de carga de Azure es un equilibrador de carga de nivel 4 (TCP, UDP) que proporciona una alta disponibilidad mediante la distribución del tráfico entrante entre máquinas virtuales con un estado correcto. Un sondeo de estado de equilibrador de carga supervisa un puerto determinado en cada máquina virtual y solo distribuye tráfico a una máquina virtual operativa.

Se define una configuración de IP de front-end que contiene una o varias direcciones IP públicas. Esta configuración de IP de front-end permite que el equilibrador de carga y las aplicaciones sean accesibles a través de Internet. 

Las máquinas virtuales se conectarán a un equilibrador de carga mediante su tarjeta de interfaz de red (NIC) virtual. Para distribuir el tráfico a las máquinas virtuales, un grupo de direcciones de back-end contiene las direcciones IP de las tarjetas de interfaz de red (NIC) virtual conectadas al equilibrador de carga.

Para controlar el flujo de tráfico, se definen reglas de equilibrador de carga para determinados puertos y protocolos que se asignan a las máquinas virtuales.

Si siguió el tutorial anterior para [crear un conjunto de escalado de máquinas virtuales](tutorial-create-vmss.md), se creó un equilibrador de carga automáticamente. Todos estos componentes se configuraron automáticamente como parte del conjunto de escalado.


## <a name="create-azure-load-balancer"></a>Creación del equilibrador de carga de Azure
En esta sección se detalla cómo se puede crear y configurar cada componente del equilibrador de carga. Antes de poder crear el equilibrador de carga, cree un grupo de recursos con [az group create](/cli/azure/group). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroupLoadBalancer* en la ubicación *eastus*:

```azurecli-interactive
az group create --name myResourceGroupLoadBalancer --location eastus
```

### <a name="create-a-public-ip-address"></a>Crear una dirección IP pública
Para obtener acceso a la aplicación en Internet, necesita una dirección IP pública para el equilibrador de carga. Cree una dirección IP pública con [az network public-ip create](/cli/azure/network/public-ip). En el ejemplo siguiente se crea una dirección IP pública denominada *myPublicIP* en el grupo de recursos *myResourceGroupLoadBalancer*:

```azurecli-interactive
az network public-ip create \
    --resource-group myResourceGroupLoadBalancer \
    --name myPublicIP
```

### <a name="create-a-load-balancer"></a>Creación de un equilibrador de carga
Cree un equilibrador de carga con [az network lb create](/cli/azure/network/lb). En el ejemplo siguiente se crea un equilibrador de carga llamado *myLoadBalancer* y se asigna la dirección *myPublicIP* a la configuración de IP de front-end:

```azurecli-interactive
az network lb create \
    --resource-group myResourceGroupLoadBalancer \
    --name myLoadBalancer \
    --frontend-ip-name myFrontEndPool \
    --backend-pool-name myBackEndPool \
    --public-ip-address myPublicIP
```

### <a name="create-a-health-probe"></a>Creación de un sondeo de estado
Para permitir que el equilibrador de carga supervise el estado de la aplicación, utilice un sondeo de estado. El sondeo de estado agrega o quita de forma dinámica las máquinas virtuales de la rotación del equilibrador de carga en base a su respuesta a las comprobaciones de estado. De forma predeterminada, una máquina virtual se quita de la distribución del equilibrador de carga después de dos errores consecutivos en un intervalo de 15 segundos. Cree un sondeo de estado en función de un protocolo o una página de comprobación de mantenimiento específica para la aplicación. 

En el ejemplo siguiente se crea un sondeo de TCP. También se pueden crear sondeos HTTP personalizados para comprobaciones de estado más específicas. Al usar un sondeo HTTP personalizado, debe crear la página de comprobación de estado, por ejemplo *healthcheck.js*. El sondeo debe devolver una respuesta **HTTP 200 OK** para que el equilibrador de carga mantenga el host en rotación.

Para crear un sondeo de estado TCP, debe usar el comando [az network lb probe create](/cli/azure/network/lb/probe). En el ejemplo siguiente se crea un sondeo de TCP denominado *myHealthProbe*:

```azurecli-interactive
az network lb probe create \
    --resource-group myResourceGroupLoadBalancer \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80
```

### <a name="create-a-load-balancer-rule"></a>Creación de una regla de equilibrador de carga
Las reglas de equilibrador de carga se utilizan para definir cómo se distribuye el tráfico a las máquinas virtuales. Se define la configuración de IP front-end para el tráfico entrante y el grupo IP de back-end para recibir el tráfico, junto con el puerto de origen y destino requeridos. Para asegurarse de que solo las máquinas virtuales correctas reciban tráfico, también hay que definir el sondeo de estado que se va usar.

Cree una regla de equilibrador de carga con [az network lb rule create](/cli/azure/network/lb/rule). En el ejemplo siguiente se crea una regla denominada *myLoadBalancerRule*, se usa el sondeo de estado *myHealthProbe* y se equilibra el tráfico en el puerto *80*:

```azurecli-interactive
az network lb rule create \
    --resource-group myResourceGroupLoadBalancer \
    --lb-name myLoadBalancer \
    --name myLoadBalancerRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEndPool \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe
```


## <a name="configure-virtual-network"></a>Configurar la red virtual
Antes de implementar algunas máquinas virtuales y poder probar el equilibrador, cree los recursos de red virtual auxiliares. Para más información sobre las redes virtuales, consulte el tutorial [Administración de Azure Virtual Networks](tutorial-virtual-network.md).

### <a name="create-network-resources"></a>Crear recursos de red
Cree la red virtual con el comando [az network vnet create](/cli/azure/network/vnet). En el ejemplo siguiente se crea una red virtual denominada *myVnet* con una subred *mySubnet*:

```azurecli-interactive
az network vnet create \
    --resource-group myResourceGroupLoadBalancer \
    --name myVnet \
    --subnet-name mySubnet
```

Para agregar un grupo de seguridad de red, use el comando [az network nsg create](/cli/azure/network/nsg). En el ejemplo siguiente se crea un grupo de seguridad de red denominado *myNetworkSecurityGroup*:

```azurecli-interactive
az network nsg create \
    --resource-group myResourceGroupLoadBalancer \
    --name myNetworkSecurityGroup
```

Cree una regla de grupo de seguridad de red con el comando [az network nsg rule create](/cli/azure/network/nsg/rule). En el ejemplo siguiente se crea una regla grupo de seguridad de red denominada *myNetworkSecurityGroupRule*:

```azurecli-interactive
az network nsg rule create \
    --resource-group myResourceGroupLoadBalancer \
    --nsg-name myNetworkSecurityGroup \
    --name myNetworkSecurityGroupRule \
    --priority 1001 \
    --protocol tcp \
    --destination-port-range 80
```

Las NIC virtuales se crean con el comando [az network nic create](/cli/azure/network/nic). En el ejemplo siguiente se crean tres NIC. (Una NIC virtual para cada máquina virtual que cree para la aplicación en los pasos siguientes). Puede crear NIC virtuales y máquinas virtuales adicionales en cualquier momento y agregarlas al equilibrador de carga:

```azurecli
for i in `seq 1 3`; do
    az network nic create \
        --resource-group myResourceGroupLoadBalancer \
        --name myNic$i \
        --vnet-name myVnet \
        --subnet mySubnet \
        --network-security-group myNetworkSecurityGroup \
        --lb-name myLoadBalancer \
        --lb-address-pools myBackEndPool
done
```

Tras crear los tres NIC virtuales, continúe con el paso siguiente.


## <a name="create-virtual-machines"></a>Creación de máquinas virtuales

### <a name="create-cloud-init-config"></a>Creación de cloud-init config
En un tutorial anterior sobre [cómo personalizar una máquina virtual Linux en el primer arranque](tutorial-automate-vm-deployment.md), aprendió a automatizar la personalización de máquinas virtuales con cloud-init. El mismo archivo de configuración cloud-init puede usarse para instalar NGINX y ejecutar una aplicación sencilla Node.js "Hola mundo" en el siguiente paso. Para ver el equilibrador de carga en acción, al final del tutorial accederá a esta sencilla aplicación en un explorador web.

En el shell actual, cree un archivo denominado "*cloud-init.txt*" y pegue la siguiente configuración. Por ejemplo, cree el archivo en Cloud Shell, no en la máquina local. Escriba `sensible-editor cloud-init.txt` para crear el archivo y ver una lista de editores disponibles. Asegúrese de que todo el archivo cloud-init se copia correctamente, especialmente la primera línea:

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

### <a name="create-virtual-machines"></a>Creación de máquinas virtuales
Para mejorar la alta disponibilidad de la aplicación, coloque las máquinas virtuales en un conjunto de disponibilidad. Para más información sobre los conjuntos de disponibilidad, consulte el tutorial anterior [Creación de máquinas virtuales de alta disponibilidad](tutorial-availability-sets.md).

Cree el conjunto de disponibilidad con [az vm availability-set create](/cli/azure/vm/availability-set). En el ejemplo siguiente se crea un conjunto de disponibilidad denominado *myAvailabilitySet*:

```azurecli-interactive 
az vm availability-set create \
    --resource-group myResourceGroupLoadBalancer \
    --name myAvailabilitySet
```

Ahora puede crear las máquinas virtuales con el comando [az vm create](/cli/azure/vm). El siguiente ejemplo crea tres máquinas virtuales y genera claves SSH si aún no existen:

```azurecli
for i in `seq 1 3`; do
    az vm create \
        --resource-group myResourceGroupLoadBalancer \
        --name myVM$i \
        --availability-set myAvailabilitySet \
        --nics myNic$i \
        --image UbuntuLTS \
        --admin-username azureuser \
        --generate-ssh-keys \
        --custom-data cloud-init.txt \
        --no-wait
done
```

Hay tareas en segundo plano que continúan ejecutándose después de que la CLI de Azure vuelve a abrir el símbolo del sistema. El parámetro `--no-wait` no espera a que se completen todas las tareas. Es posible que tenga que esperar otros dos minutos antes de poder acceder a la aplicación. El sondeo de estado del equilibrador de carga detecta automáticamente cuándo la aplicación se está ejecutando en cada máquina virtual. Una vez que la aplicación se esté ejecutando, la regla del equilibrador de carga empieza a distribuir el tráfico.


## <a name="test-load-balancer"></a>Prueba del equilibrador de carga
Obtenga la dirección IP pública del equilibrador de carga con [az network public-ip show](/cli/azure/network/public-ip). En el ejemplo siguiente se obtiene la dirección IP de *myPublicIP* que se ha creado anteriormente:

```azurecli-interactive
az network public-ip show \
    --resource-group myResourceGroupLoadBalancer \
    --name myPublicIP \
    --query [ipAddress] \
    --output tsv
```

A continuación, puede escribir la dirección IP pública en un explorador web. Recuerde que tendrán que pasar unos minutos para que las máquinas virtuales estén listas antes de que el equilibrador de carga comience a distribuir el tráfico entre ellas. Se muestra la aplicación, incluido el nombre de host de la máquina virtual a la que el equilibrador de carga distribuye el tráfico como en el ejemplo siguiente:

![Ejecución de la aplicación Node.js](./media/tutorial-load-balancer/running-nodejs-app.png)

Para ver cómo el equilibrador de carga distribuye el tráfico entre las tres máquinas virtuales que ejecutan la aplicación, puede realizar una actualización forzada del explorador web.


## <a name="add-and-remove-vms"></a>Agregar y quitar máquinas virtuales
Puede que tenga que realizar labores de mantenimiento de las máquinas virtuales que ejecutan la aplicación, como la instalación de actualizaciones del sistema operativo. Para gestionar un aumento de tráfico a la aplicación, tiene que agregar más máquinas virtuales. Esta sección le muestra cómo quitar o agregar una máquina virtual desde el equilibrador de carga.

### <a name="remove-a-vm-from-the-load-balancer"></a>Eliminación de una máquina virtual del equilibrador de carga
Puede quitar una máquina virtual del grupo de direcciones de back-end con el comando [az network nic ip-config address-pool remove](/cli/azure/network/nic/ip-config/address-pool). En el ejemplo siguiente se quita la NIC virtual para **myVM2** de *myLoadBalancer*:

```azurecli-interactive
az network nic ip-config address-pool remove \
    --resource-group myResourceGroupLoadBalancer \
    --nic-name myNic2 \
    --ip-config-name ipConfig1 \
    --lb-name myLoadBalancer \
    --address-pool myBackEndPool 
```

Para ver cómo el equilibrador de carga distribuye el tráfico entre las dos máquinas virtuales que quedan ejecutando la aplicación, puede realizar una actualización forzada del explorador web. Ahora puede realizar tareas de mantenimiento en la máquina virtual, como instalar actualizaciones del sistema operativo o realizar un reinicio de máquina virtual.

Para ver una lista de las máquinas virtuales con NIC virtuales conectados al equilibrador de carga, use [az network lb address-pool show](/cli/azure/network/lb/address-pool). Consulte y filtre por identificador de NIC virtual de la siguiente manera:

```azurecli-interactive
az network lb address-pool show \
    --resource-group myResourceGroupLoadBalancer \
    --lb-name myLoadBalancer \
    --name myBackEndPool \
    --query backendIpConfigurations \
    --output tsv | cut -f4
```

El resultado es similar al ejemplo siguiente, que muestra que la NIC virtual de VM 2 ya no forma parte del grupo de direcciones de back-end:

```output
/subscriptions/<guid>/resourceGroups/myResourceGroupLoadBalancer/providers/Microsoft.Network/networkInterfaces/myNic1/ipConfigurations/ipconfig1
/subscriptions/<guid>/resourceGroups/myResourceGroupLoadBalancer/providers/Microsoft.Network/networkInterfaces/myNic3/ipConfigurations/ipconfig1
```

### <a name="add-a-vm-to-the-load-balancer"></a>Incorporación de una máquina virtual al equilibrador de carga
Después de realizar el mantenimiento en una máquina virtual, o si necesita expandir la capacidad, puede agregar una máquina virtual al grupo de direcciones de back-end con el comando [az network nic ip-config address-pool add](/cli/azure/network/nic/ip-config/address-pool). En el ejemplo siguiente se agrega la NIC virtual para **myVM2** a *myLoadBalancer*:

```azurecli-interactive
az network nic ip-config address-pool add \
    --resource-group myResourceGroupLoadBalancer \
    --nic-name myNic2 \
    --ip-config-name ipConfig1 \
    --lb-name myLoadBalancer \
    --address-pool myBackEndPool
```

Para comprobar que la NIC virtual está conectada al grupo de direcciones de back-end, vuelva a usar [az network lb address-pool show](/cli/azure/network/lb/address-pool) del paso anterior.


## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha creado un equilibrador de carga y conectó máquinas virtuales a él. Ha aprendido a:

> [!div class="checklist"]
> * Creación de un equilibrador de carga de Azure
> * Creación del sondeo de estado de un equilibrador de carga
> * Crear reglas de tráfico del equilibrador de carga
> * Usar cloud-init para crear una aplicación básica de Node.js
> * Crear máquinas virtuales y conectarlas a un equilibrador de carga
> * Ver un equilibrador de carga en acción
> * Agregar y quitar las máquinas virtuales de un equilibrador de carga

Prosiga con el siguiente tutorial para aprender más sobre los componentes de red virtual de Azure.

> [!div class="nextstepaction"]
> [Administración de máquinas y redes virtuales](tutorial-virtual-network.md)
