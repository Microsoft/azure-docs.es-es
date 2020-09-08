---
title: 'Inicio rápido: Creación de un equilibrador de carga interno: CLI de Azure'
titleSuffix: Azure Load Balancer
description: En esta guía de inicio rápido se muestra cómo crear un equilibrador de carga interno mediante la CLI de Azure.
services: load-balancer
documentationcenter: na
author: asudbring
manager: KumudD
tags: azure-resource-manager
Customer intent: I want to create a load balancer so that I can load balance internal traffic to VMs.
ms.service: load-balancer
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/20/2020
ms.author: allensu
ms.custom: mvc, devx-track-javascript, devx-track-azurecli
ms.openlocfilehash: 3016825648ef45f8aa64b4228bac2b1f830004c3
ms.sourcegitcommit: e2b36c60a53904ecf3b99b3f1d36be00fbde24fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88763857"
---
# <a name="quickstart-create-an-internal-load-balancer-to-load-balance-vms-using-azure-cli"></a>Inicio rápido: Creación de un equilibrador de carga interno para equilibrar la carga de las máquinas virtuales con la CLI de Azure

Comience a usar Azure Load Balancer con la CLI de Azure para crear un equilibrador de carga público y tres máquinas virtuales.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- CLI de Azure instalada localmente o Azure Cloud Shell

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)] 

Si decide instalar y usar la CLI localmente, para esta guía de inicio rápido se necesita la versión 2.0.28 de la CLI de Azure o una versión posterior. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

Cree un grupo de recursos con [az group create](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest#az-group-create):

* Denominado **myResourceGroupLB**. 
* En la ubicación **eastus**.

```azurecli-interactive
  az group create \
    --name myResourceGroupLB \
    --location eastus
```
---

# <a name="standard-sku"></a>[**SKU Estándar**](#tab/option-1-create-load-balancer-standard)

>[!NOTE]
>Se recomienda usar la SKU Estándar de Load Balancer para las cargas de trabajo de producción. Para más información sobre las SKU, consulte **[SKU de Azure Load Balancer](skus.md)** .

## <a name="configure-virtual-network"></a>Configurar la red virtual

Antes de implementar las máquinas virtuales y el equilibrador de carga, cree los recursos de red virtual auxiliares.

### <a name="create-a-virtual-network"></a>Creación de una red virtual

Cree una red virtual con [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet?view=azure-cli-latest#az-network-vnet-createt):

* Denominada **myVNet**.
* Con el prefijo de dirección **10.1.0.0/16**.
* Subred denominada **MyBackendSubnet**.
* Con el prefijo de subred **10.1.0.0/24**.
* En el grupo de recursos **myResourceGroupLB**.
* Ubicación de **eastus**.

```azurecli-interactive
  az network vnet create \
    --resource-group myResourceGroupLB \
    --location eastus \
    --name myVNet \
    --address-prefixes 10.1.0.0/16 \
    --subnet-name myBackendSubnet \
    --subnet-prefixes 10.1.0.0/24
```
### <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

En el caso de un equilibrador de carga estándar, las VM de la dirección de back-end deben tener interfaces de red que pertenezcan a un grupo de seguridad de red. 

Cree un grupo de seguridad de red con [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg?view=azure-cli-latest#az-network-nsg-create):

* Denominado **myNSG**.
* En el grupo de recursos **myResourceGroupLB**.

```azurecli-interactive
  az network nsg create \
    --resource-group myResourceGroupLB \
    --name myNSG
```

### <a name="create-a-network-security-group-rule"></a>Creación de una regla de grupo de seguridad de red

Cree una regla de grupo de seguridad de red con el [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule?view=azure-cli-latest#az-network-nsg-rule-create):

* Denominada **myNSGRuleHTTP**.
* En el grupo de seguridad de red que creó en el paso anterior, **myNSG**.
* En el grupo de recursos **myResourceGroupLB**.
* Protocolo **(*)** .
* Dirección **Inbound**.
* Origen **(*)** .
* Destino: **(*)** .
* Puerto de destino **80**.
* Acceso: **Allow**.
* Prioridad **200**.

```azurecli-interactive
  az network nsg rule create \
    --resource-group myResourceGroupLB \
    --nsg-name myNSG \
    --name myNSGRuleHTTP \
    --protocol '*' \
    --direction inbound \
    --source-address-prefix '*' \
    --source-port-range '*' \
    --destination-address-prefix '*' \
    --destination-port-range 80 \
    --access allow \
    --priority 200
```

### <a name="create-network-interfaces-for-the-virtual-machines"></a>Creación de interfaces de red para las máquinas virtuales

Cree dos interfaces de red con [az network nic create](https://docs.microsoft.com/cli/azure/network/nic?view=azure-cli-latest#az-network-nic-create):

#### <a name="vm1"></a>VM1

* Denominada **myNicVM1**.
* En el grupo de recursos **myResourceGroupLB**.
* En la red virtual **myVNet**.
* En la subred **myBackendSubnet**.
* En el grupo de seguridad de red **myNSG**.

```azurecli-interactive
  az network nic create \
    --resource-group myResourceGroupLB \
    --name myNicVM1 \
    --vnet-name myVNet \
    --subnet myBackEndSubnet \
    --network-security-group myNSG
```
#### <a name="vm2"></a>VM2

* Denominada **myNicVM2**.
* En el grupo de recursos **myResourceGroupLB**.
* En la red virtual **myVNet**.
* En la subred **myBackendSubnet**.
* En el grupo de seguridad de red **myNSG**.

```azurecli-interactive
  az network nic create \
    --resource-group myResourceGroupLB \
    --name myNicVM2 \
    --vnet-name myVnet \
    --subnet myBackEndSubnet \
    --network-security-group myNSG
```

## <a name="create-backend-servers"></a>Creación de servidores back-end

En esta sección, creará:

* Un archivo de configuración de nube denominado **cloud-init.txt** para la configuración del servidor.
* Dos máquinas virtuales que se usarán como servidores back-end para el equilibrador de carga.

### <a name="create-cloud-init-configuration-file"></a>Creación del archivo de configuración cloud-init

Use un archivo de configuración cloud-init para instalar NGINX y ejecutar una aplicación Node.js "Hola mundo" en una máquina virtual Linux. 

En el shell actual, cree un archivo denominado cloud-init.txt. Copie el siguiente fragmento de código y péguelo en el shell. Asegúrese de copiar correctamente todo el archivo cloud-init, especialmente la primera línea:

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

Cree las máquinas virtuales con [az vm create](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest#az-vm-create):

#### <a name="vm1"></a>VM1
* Denominada **myVM1**.
* En el grupo de recursos **myResourceGroupLB**.
* Conectada a la interfaz de red **myNicVM1**.
* Imagen de máquina virtual **UbuntuLTS**.
* Archivo de configuración **cloud-init.txt** creado en el paso anterior.
* En **Zona 1**.

```azurecli-interactive
  az vm create \
    --resource-group myResourceGroupLB \
    --name myVM1 \
    --nics myNicVM1 \
    --image UbuntuLTS \
    --admin-user azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt \
    --zone 1 \
    --no-wait
    
```
#### <a name="vm2"></a>VM2
* Denominada **myVM2**.
* En el grupo de recursos **myResourceGroupLB**.
* Conectada a la interfaz de red **myNicVM2**.
* Imagen de máquina virtual **UbuntuLTS**.
* Archivo de configuración **cloud-init.txt** creado en el paso anterior.
* En **Zona 2**.

```azurecli-interactive
  az vm create \
    --resource-group myResourceGroupLB \
    --name myVM2 \
    --nics myNicVM2 \
    --image UbuntuLTS \
    --admin-user azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt \
    --zone 2 \
    --no-wait
```

Puede que las VM tarden unos minutos en implementarse.

## <a name="create-standard-load-balancer"></a>Creación de un equilibrador de carga estándar

En esta sección se detalla cómo se pueden crear y configurar los componentes siguientes del equilibrador de carga:

  * Un grupo de direcciones IP de front-end que recibe el tráfico de red entrante en el equilibrador de carga.
  * Un grupo de direcciones IP de back-end al que el grupo de servidores front-end envía el tráfico de red de carga equilibrada.
  * Un sondeo de estado que determina el estado de las instancias de máquina virtual de back-end.
  * Una regla de equilibrador de carga que define cómo se distribuye el tráfico a las VM.

### <a name="create-the-load-balancer-resource"></a>Creación del recurso del equilibrador de carga

Cree un equilibrador de carga público con [az network lb create](https://docs.microsoft.com/cli/azure/network/lb?view=azure-cli-latest#az-network-lb-create):

* Denominado **myLoadBalancer**.
* Un grupo de front-end denominado **MyFrontEnd**.
* Un grupo de back-end denominado **myBackEndPool**
* Asociado a la red virtual **myVNet**.
* Asociado a la subred de back-end **myBackendSubnet**.

```azurecli-interactive
  az network lb create \
    --resource-group myResourceGroupLB \
    --name myLoadBalancer \
    --sku Standard \
    --vnet-name myVnet \
    --subnet myBackendSubnet \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool       
```

### <a name="create-the-health-probe"></a>Creación del sondeo de estado

Los sondeos de estado comprueban todas las instancias de máquina virtual para asegurarse de que pueden enviar tráfico de red. 

Una máquina virtual con una comprobación de sondeo con errores se quita del equilibrador de carga. La máquina virtual se agrega de nuevo al equilibrador de carga cuando se resuelve el error.

Cree un sondeo de estado con [az network lb probe create](https://docs.microsoft.com/cli/azure/network/lb/probe?view=azure-cli-latest#az-network-lb-probe-create):

* Supervisa el estado de las máquinas virtuales.
* Denominado **myHealthProbe**.
* Protocolo **TCP**.
* **Puerto 80** de supervisión.

```azurecli-interactive
  az network lb probe create \
    --resource-group myResourceGroupLB \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80   
```

### <a name="create-the-load-balancer-rule"></a>Creación de la regla de equilibrador de carga

Una regla de equilibrador de carga define:

* La configuración de IP del front-end para el tráfico entrante.
* El grupo de IP de back-end para recibir el tráfico.
* Los puertos de origen y de destino requeridos. 

Cree una regla de equilibrador de carga con [az network lb rule create](https://docs.microsoft.com/cli/azure/network/lb/rule?view=azure-cli-latest#az-network-lb-rule-create):

* Denominada **myHTTPRule**.
* Que escuche en el **puerto 80** en el grupo de front-end **myFrontEnd**.
* Que envíe el tráfico de red con equilibrio de carga al grupo de direcciones de back-end **myBackEndPool** a través del **Puerto 80**. 
* Mediante el sondeo de estado **myHealthProbe**.
* Protocolo **TCP**.
* Habilite la traducción de direcciones de red de origen (SNAT) de salida mediante la dirección IP de front-end.

```azurecli-interactive
  az network lb rule create \
    --resource-group myResourceGroupLB \
    --lb-name myLoadBalancer \
    --name myHTTPRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe \
    --disable-outbound-snat true 
```
>[!NOTE]
>Las máquinas virtuales del grupo de back-end no tendrán conectividad de salida a Internet con esta configuración. </br> Para más información acerca de cómo proporcionar conectividad de salida, consulte: </br> **[Conexiones salientes en Azure](load-balancer-outbound-connections.md)**</br> Opciones para proporcionar conectividad: </br> **[Configuración del equilibrador de carga solo de salida](egress-only.md)** </br> **[¿Qué es NAT de Virtual Network?](https://docs.microsoft.com/azure/virtual-network/nat-overview)**

### <a name="add-virtual-machines-to-load-balancer-backend-pool"></a>Adición de máquinas virtuales al grupo de back-end del equilibrador de carga

Agregue las máquinas virtuales al grupo de back-end con [az network nic ip-config address-pool add](https://docs.microsoft.com/cli/azure/network/nic/ip-config/address-pool?view=azure-cli-latest#az-network-nic-ip-config-address-pool-add):


#### <a name="vm1"></a>VM1
* En el grupo de direcciones de back-end **myBackEndPool**.
* En el grupo de recursos **myResourceGroupLB**.
* Asociada a la interfaz de red **myNicVM1** e **ipconfig1**.
* Asociada al equilibrador de carga **myLoadBalancer**.

```azurecli-interactive
  az network nic ip-config address-pool add \
   --address-pool myBackendPool \
   --ip-config-name ipconfig1 \
   --nic-name myNicVM1 \
   --resource-group myResourceGroupLB \
   --lb-name myLoadBalancer
```

#### <a name="vm2"></a>VM2
* En el grupo de direcciones de back-end **myBackEndPool**.
* En el grupo de recursos **myResourceGroupLB**.
* Asociada a la interfaz de red **myNicVM2** e **ipconfig1**.
* Asociada al equilibrador de carga **myLoadBalancer**.

```azurecli-interactive
  az network nic ip-config address-pool add \
   --address-pool myBackendPool \
   --ip-config-name ipconfig1 \
   --nic-name myNicVM2 \
   --resource-group myResourceGroupLB \
   --lb-name myLoadBalancer
```

# <a name="basic-sku"></a>[**SKU básica**](#tab/option-1-create-load-balancer-basic)

>[!NOTE]
>Se recomienda usar la SKU Estándar de Load Balancer para las cargas de trabajo de producción. Para más información sobre las SKU, consulte **[SKU de Azure Load Balancer](skus.md)** .

## <a name="configure-virtual-network"></a>Configurar la red virtual

Antes de implementar las máquinas virtuales y el equilibrador de carga, cree los recursos de red virtual auxiliares.

### <a name="create-a-virtual-network"></a>Creación de una red virtual

Cree una red virtual con [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet?view=azure-cli-latest#az-network-vnet-createt):

* Denominada **myVNet**.
* Con el prefijo de dirección **10.1.0.0/16**.
* Subred denominada **MyBackendSubnet**.
* Con el prefijo de subred **10.1.0.0/24**.
* En el grupo de recursos **myResourceGroupLB**.
* Ubicación de **eastus**.

```azurecli-interactive
  az network vnet create \
    --resource-group myResourceGroupLB \
    --location eastus \
    --name myVNet \
    --address-prefixes 10.1.0.0/16 \
    --subnet-name myBackendSubnet \
    --subnet-prefixes 10.1.0.0/24
```
### <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

En el caso de un equilibrador de carga estándar, las VM de la dirección de back-end deben tener interfaces de red que pertenezcan a un grupo de seguridad de red. 

Cree un grupo de seguridad de red con [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg?view=azure-cli-latest#az-network-nsg-create):

* Denominado **myNSG**.
* En el grupo de recursos **myResourceGroupLB**.

```azurecli-interactive
  az network nsg create \
    --resource-group myResourceGroupLB \
    --name myNSG
```

### <a name="create-a-network-security-group-rule"></a>Creación de una regla de grupo de seguridad de red

Cree una regla de grupo de seguridad de red con el [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule?view=azure-cli-latest#az-network-nsg-rule-create):

* Denominada **myNSGRuleHTTP**.
* En el grupo de seguridad de red que creó en el paso anterior, **myNSG**.
* En el grupo de recursos **myResourceGroupLB**.
* Protocolo **(*)** .
* Dirección **Inbound**.
* Origen **(*)** .
* Destino: **(*)** .
* Puerto de destino **80**.
* Acceso: **Allow**.
* Prioridad **200**.

```azurecli-interactive
  az network nsg rule create \
    --resource-group myResourceGroupLB \
    --nsg-name myNSG \
    --name myNSGRuleHTTP \
    --protocol '*' \
    --direction inbound \
    --source-address-prefix '*' \
    --source-port-range '*' \
    --destination-address-prefix '*' \
    --destination-port-range 80 \
    --access allow \
    --priority 200
```

### <a name="create-network-interfaces-for-the-virtual-machines"></a>Creación de interfaces de red para las máquinas virtuales

Cree dos interfaces de red con [az network nic create](https://docs.microsoft.com/cli/azure/network/nic?view=azure-cli-latest#az-network-nic-create):

#### <a name="vm1"></a>VM1

* Denominada **myNicVM1**.
* En el grupo de recursos **myResourceGroupLB**.
* En la red virtual **myVNet**.
* En la subred **myBackendSubnet**.
* En el grupo de seguridad de red **myNSG**.

```azurecli-interactive

  az network nic create \
    --resource-group myResourceGroupLB \
    --name myNicVM1 \
    --vnet-name myVNet \
    --subnet myBackEndSubnet \
    --network-security-group myNSG
```
#### <a name="vm2"></a>VM2

* Denominada **myNicVM2**.
* En el grupo de recursos **myResourceGroupLB**.
* En la red virtual **myVNet**.
* En la subred **myBackendSubnet**.

```azurecli-interactive
  az network nic create \
    --resource-group myResourceGroupLB \
    --name myNicVM2 \
    --vnet-name myVnet \
    --subnet myBackEndSubnet \
    --network-security-group myNSG
```

## <a name="create-backend-servers"></a>Creación de servidores back-end

En esta sección, creará:

* Un archivo de configuración de nube denominado **cloud-init.txt** para la configuración del servidor. 
* Conjunto de disponibilidad para máquinas virtuales
* Dos máquinas virtuales que se usarán como servidores back-end para el equilibrador de carga.

Para verificar que el equilibrador de carga se ha creado correctamente, instale NGINX en las máquinas virtuales.

### <a name="create-cloud-init-configuration-file"></a>Creación del archivo de configuración cloud-init

Use un archivo de configuración cloud-init para instalar NGINX y ejecutar una aplicación Node.js "Hola mundo" en una máquina virtual Linux. 

En el shell actual, cree un archivo denominado cloud-init.txt. Copie el siguiente fragmento de código y péguelo en el shell. Asegúrese de copiar correctamente todo el archivo cloud-init, especialmente la primera línea:

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

### <a name="create-availability-set-for-virtual-machines"></a>Creación de un conjunto de disponibilidad para máquinas virtuales

Cree el conjunto de disponibilidad con [az vm availability-set create](https://docs.microsoft.com/cli/azure/vm/availability-set?view=azure-cli-latest#az-vm-availability-set-create):

* Denominado **myAvSet**.
* En el grupo de recursos **myResourceGroupLB**.
* Ubicación **eastus**.

```azurecli-interactive
  az vm availability-set create \
    --name myAvSet \
    --resource-group myResourceGroupLB \
    --location eastus 
    
```

### <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Cree las máquinas virtuales con [az vm create](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest#az-vm-create):

#### <a name="vm1"></a>VM1
* Denominada **myVM1**.
* En el grupo de recursos **myResourceGroupLB**.
* Conectada a la interfaz de red **myNicVM1**.
* Imagen de máquina virtual **UbuntuLTS**.
* Archivo de configuración **cloud-init.txt** creado en el paso anterior.
* En el conjunto de disponibilidad **myAvSet**.

```azurecli-interactive
  az vm create \
    --resource-group myResourceGroupLB \
    --name myVM1 \
    --nics myNicVM1 \
    --image UbuntuLTS \
    --admin-user azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt \
    --availability-set myAvSet \
    --no-wait
    
```
#### <a name="vm2"></a>VM2
* Denominada **myVM2**.
* En el grupo de recursos **myResourceGroupLB**.
* Conectada a la interfaz de red **myNicVM2**.
* Imagen de máquina virtual **UbuntuLTS**.
* Archivo de configuración **cloud-init.txt** creado en el paso anterior.
* En **Zona 2**.

```azurecli-interactive
  az vm create \
    --resource-group myResourceGroupLB \
    --name myVM2 \
    --nics myNicVM2 \
    --image UbuntuLTS \
    --admin-user azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt \
    --availability-set myAvSet  \
    --no-wait
```

Puede que las VM tarden unos minutos en implementarse.

## <a name="create-basic-load-balancer"></a>Creación de un equilibrador de carga básico

En esta sección se detalla cómo se pueden crear y configurar los componentes siguientes del equilibrador de carga:

  * Un grupo de direcciones IP de front-end que recibe el tráfico de red entrante en el equilibrador de carga.
  * Un grupo de direcciones IP de back-end al que el grupo de servidores front-end envía el tráfico de red de carga equilibrada.
  * Un sondeo de estado que determina el estado de las instancias de máquina virtual de back-end.
  * Una regla de equilibrador de carga que define cómo se distribuye el tráfico a las VM.

### <a name="create-the-load-balancer-resource"></a>Creación del recurso del equilibrador de carga

Cree un equilibrador de carga público con [az network lb create](https://docs.microsoft.com/cli/azure/network/lb?view=azure-cli-latest#az-network-lb-create):

* Denominado **myLoadBalancer**.
* Un grupo de front-end denominado **MyFrontEnd**.
* Un grupo de back-end denominado **myBackEndPool**
* Asociado a la red virtual **myVNet**.
* Asociado a la subred de back-end **myBackendSubnet**.

```azurecli-interactive
  az network lb create \
    --resource-group myResourceGroupLB \
    --name myLoadBalancer \
    --sku Basic \
    --vnet-name myVNet \
    --subnet myBackendSubnet \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool       
```

### <a name="create-the-health-probe"></a>Creación del sondeo de estado

Los sondeos de estado comprueban todas las instancias de máquina virtual para asegurarse de que pueden enviar tráfico de red. 

Una máquina virtual con una comprobación de sondeo con errores se quita del equilibrador de carga. La máquina virtual se agrega de nuevo al equilibrador de carga cuando se resuelve el error.

Cree un sondeo de estado con [az network lb probe create](https://docs.microsoft.com/cli/azure/network/lb/probe?view=azure-cli-latest#az-network-lb-probe-create):

* Supervisa el estado de las máquinas virtuales.
* Denominado **myHealthProbe**.
* Protocolo **TCP**.
* **Puerto 80** de supervisión.

```azurecli-interactive
  az network lb probe create \
    --resource-group myResourceGroupLB \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80   
```

### <a name="create-the-load-balancer-rule"></a>Creación de la regla de equilibrador de carga

Una regla de equilibrador de carga define:

* La configuración de IP del front-end para el tráfico entrante.
* El grupo de IP de back-end para recibir el tráfico.
* Los puertos de origen y de destino requeridos. 

Cree una regla de equilibrador de carga con [az network lb rule create](https://docs.microsoft.com/cli/azure/network/lb/rule?view=azure-cli-latest#az-network-lb-rule-create):

* Denominada **myHTTPRule**.
* Que escuche en el **puerto 80** en el grupo de front-end **myFrontEnd**.
* Que envíe el tráfico de red con equilibrio de carga al grupo de direcciones de back-end **myBackEndPool** a través del **Puerto 80**. 
* Mediante el sondeo de estado **myHealthProbe**.
* Protocolo **TCP**.

```azurecli-interactive
  az network lb rule create \
    --resource-group myResourceGroupLB \
    --lb-name myLoadBalancer \
    --name myHTTPRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe
```
### <a name="add-virtual-machines-to-load-balancer-backend-pool"></a>Adición de máquinas virtuales al grupo de back-end del equilibrador de carga

Agregue las máquinas virtuales al grupo de back-end con [az network nic ip-config address-pool add](https://docs.microsoft.com/cli/azure/network/nic/ip-config/address-pool?view=azure-cli-latest#az-network-nic-ip-config-address-pool-add):


#### <a name="vm1"></a>VM1
* En el grupo de direcciones de back-end **myBackEndPool**.
* En el grupo de recursos **myResourceGroupLB**.
* Asociada a la interfaz de red **myNicVM1** e **ipconfig1**.
* Asociada al equilibrador de carga **myLoadBalancer**.

```azurecli-interactive
  az network nic ip-config address-pool add \
   --address-pool myBackendPool \
   --ip-config-name ipconfig1 \
   --nic-name myNicVM1 \
   --resource-group myResourceGroupLB \
   --lb-name myLoadBalancer
```

#### <a name="vm2"></a>VM2
* En el grupo de direcciones de back-end **myBackEndPool**.
* En el grupo de recursos **myResourceGroupLB**.
* Asociada a la interfaz de red **myNicVM2** e **ipconfig1**.
* Asociada al equilibrador de carga **myLoadBalancer**.

```azurecli-interactive
  az network nic ip-config address-pool add \
   --address-pool myBackendPool \
   --ip-config-name ipconfig1 \
   --nic-name myNicVM2 \
   --resource-group myResourceGroupLB \
   --lb-name myLoadBalancer
```

---

## <a name="test-the-load-balancer"></a>Prueba del equilibrador de carga

### <a name="create-azure-bastion-public-ip"></a>Creación de una dirección IP pública de Azure Bastion

Use [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip?view=azure-cli-latest#az-network-public-ip-create) para crear una dirección IP pública para el host bastión:

* Cree una dirección IP pública con redundancia de zona estándar llamada **myBastionIP**.
* En **myResourceGroupLB**.

```azurecli-interactive
  az network public-ip create \
    --resource-group myResourceGroupLB \
    --name myBastionIP \
    --sku Standard
```

### <a name="create-azure-bastion-subnet"></a>Creación de una subred de Azure Bastion

Use [az network vnet subnet create](https://docs.microsoft.com/cli/azure/network/vnet/subnet?view=azure-cli-latest#az-network-vnet-subnet-create) para crear una subred:

* Llamada **AzureBastionSubnet**.
* Con el prefijo de dirección **10.1.1.0/24**.
* En la red virtual **myVNet**.
* En el grupo de recursos **myResourceGroupLB**.

```azurecli-interactive
  az network vnet subnet create \
    --resource-group myResourceGroupLB \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.1.1.0/24
```

### <a name="create-azure-bastion-host"></a>Creación de un host de Azure Bastion
Use [az network bastion create](https://docs.microsoft.com/cli/azure/network/bastion?view=azure-cli-latest#az-network-bastion-create) para crear un host bastión:

* Llamado **myBastionHost**.
* En **myResourceGroupLB**.
* Asociado a la dirección IP pública **myBastionIP**.
* Asociado a la red virtual **myVNet**.
* En la ubicación **eastus**.

```azurecli-interactive
  az network bastion create \
    --resource-group myResourceGroupLB \
    --name myBastionHost \
    --public-ip-address myBastionIP \
    --vnet-name myVNet \
    --location eastus
```
El host bastión tardará unos minutos en implementarse.

### <a name="create-test-virtual-machine"></a>Creación de una máquina virtual de prueba

Cree la interfaz de red con [az network nic create](https://docs.microsoft.com/cli/azure/network/nic?view=azure-cli-latest#az-network-nic-create):

* Llamada **myNicTestVM**.
* En el grupo de recursos **myResourceGroupLB**.
* En la red virtual **myVNet**.
* En la subred **myBackendSubnet**.
* En el grupo de seguridad de red **myNSG**.

```azurecli-interactive
  az network nic create \
    --resource-group myResourceGroupLB \
    --name myNicTestVM \
    --vnet-name myVNet \
    --subnet myBackEndSubnet \
    --network-security-group myNSG
```
Cree la máquina virtual con [az vm create](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest#az-vm-create):

* Llamada **myTestVM**.
* En el grupo de recursos **myResourceGroupLB**.
* Conectada a la interfaz de red **myNicTestVM**.
* Con la imagen de máquina virtual **Win2019Datacenter**.
* Elija los valores de **\<adminpass>** y **\<adminuser>** .
  

```azurecli-interactive
  az vm create \
    --resource-group myResourceGroupLB \
    --name myTestVM \
    --nics myNicTestVM \
    --image Win2019Datacenter \
    --admin-username <adminuser> \
    --admin-password <adminpass> \
    --no-wait
```
La implementación de la máquina virtual puede tardar unos minutos.

### <a name="test"></a>Prueba

1. [Inicie sesión](https://portal.azure.com) en Azure Portal.

1. Busque la dirección IP privada del equilibrador de carga en la pantalla **Información general**. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, **myLoadBalancer**.

2. Tome nota o copie la dirección que encontrará junto a **Dirección IP privada**, en la pestaña **Información general** de **myLoadBalancer**.

3. Seleccione **Todos los servicios** en el menú de la izquierda, seleccione **Todos los recursos** y, después, en la lista de recursos, seleccione **myTestVM**, que se encuentra en el grupo de recursos **myResourceGroupLB**.

4. En la página **Introducción**, seleccione **Conectar** y después **Instancia de Bastion**.

6. Escriba el nombre de usuario y la contraseña especificados durante la creación de la máquina virtual.

7. Abra **Internet Explorer** en **myTestVM**.

8. Escriba la dirección IP del paso anterior en la barra de direcciones del explorador. La página predeterminada del servidor web de IIS se muestra en el explorador.

    :::image type="content" source="./media/quickstart-load-balancer-standard-internal-portal/load-balancer-test.png" alt-text="Crear un equilibrador de carga interno estándar" border="true":::
   
Para ver el tráfico distribuido por Load Balancer entre las tres máquinas virtuales, puede personalizar la página predeterminada de cada servidor web IIS de las máquinas virtuales y luego forzar una actualización del explorador web desde el equipo cliente.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, use el comando [az group delete](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest#az-group-delete) para quitar el grupo de recursos, el equilibrador de carga y todos los recursos relacionados.

```azurecli-interactive
  az group delete \
    --name myResourceGroupLB
```

## <a name="next-steps"></a>Pasos siguientes
En esta guía de inicio rápido

* Creó un equilibrador de carga estándar o público.
* Conectó máquinas virtuales. 
* Configuró la regla de tráfico del equilibrador de carga y el sondeo de estado.
* Probó el equilibrador de carga.

Para más información sobre Azure Load Balancer, consulte [¿Qué es Azure Load Balancer?](load-balancer-overview.md) y [Preguntas frecuentes de Load Balancer](load-balancer-faqs.md).

Más información sobre [Load Balancer y zonas de disponibilidad](load-balancer-standard-availability-zones.md).
