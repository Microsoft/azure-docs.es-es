---
title: 'Creación y prueba de una puerta de enlace de NAT: CLI de Azure'
titlesuffix: Azure Virtual Network NAT
description: En este tutorial se muestra cómo crear una puerta de enlace de NAT mediante la CLI de Azure y probar el servicio NAT.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
Customer intent: I want to test a NAT gateway for outbound connectivity for my virtual network.
ms.service: virtual-network
ms.subservice: nat
ms.devlang: na
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 06/11/2020
ms.author: allensu
ms.custom: devx-track-azurecli
ms.openlocfilehash: 7d4467e557105100fc32940c05fa349722689867
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88054381"
---
# <a name="tutorial-create-a-nat-gateway-using-azure-cli-and-test-the-nat-service"></a>Tutorial: Creación de una puerta de enlace de NAT mediante la CLI de Azure y prueba del servicio NAT

En este tutorial, creará una puerta de enlace de NAT para proporcionar conectividad saliente a las máquinas virtuales de Azure. Para probar la puerta de enlace de NAT, implemente una máquina virtual de origen y destino. Luego, realizará conexiones salientes a una dirección IP pública. Estas conexiones irán desde la máquina virtual de origen a la de destino. Para simplificar las cosas, en este tutorial se implementan el origen y el destino en dos redes virtuales diferentes del mismo grupo de recursos.


[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Puede realizar este tutorial con Azure Cloud Shell o ejecutar los comandos respectivos de forma local.  Si no ha usado Azure Cloud Shell, debe [iniciar sesión ahora](https://shell.azure.com).

Si elige ejecutar estos comandos de forma local, debe instalar la CLI.  En este tutorial es necesario ejecutar la versión 2.0.71 de la CLI de Azure o cualquier versión posterior. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).


## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos con [az group create](https://docs.microsoft.com/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

En el ejemplo siguiente se crea un grupo de recursos llamado **myResourceGroupNAT** en la ubicación **eastus2**:

```azurecli-interactive
  az group create \
    --name myResourceGroupNAT \
    --location eastus2
    
```

## <a name="create-the-nat-gateway"></a>Creación de la puerta de enlace de NAT

### <a name="create-a-public-ip-address"></a>Crear una dirección IP pública

Para acceder a la red pública de Internet, necesita una o varias direcciones IP públicas para la puerta de enlace de NAT. Use [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip) para crear un recurso de dirección IP pública llamado **myPublicIPsource** en **myResourceGroupNAT**.

```azurecli-interactive
  az network public-ip create \
  --resource-group myResourceGroupNAT \
  --name myPublicIPsource \
  --sku standard
  
```

### <a name="create-a-public-ip-prefix"></a>Creación de un prefijo de dirección IP pública

Puede usar uno o varios recursos de dirección IP pública, prefijos de dirección IP pública, o ambos, con la puerta de enlace de NAT. Para demostrarlo, vamos a agregar un recurso de prefijo de dirección IP pública a este escenario.   Use [az network public-ip prefix create](/cli/azure/network/public-ip/prefix?view=azure-cli-latest#az-network-public-ip-prefix-create) para crear un recurso de prefijo de dirección IP pública llamado **myPublicIPprefixsource** en **myResourceGroupNAT**.

```azurecli-interactive
  az network public-ip prefix create \
  --resource-group myResourceGroupNAT \
  --name myPublicIPprefixsource \
  --length 31
  
```

### <a name="create-a-nat-gateway-resource"></a>Creación de un recurso de puerta de enlace de NAT

En esta sección se detalla cómo crear y configurar los componentes siguientes del servicio NAT mediante el recurso de puerta de enlace de NAT:
  - Un grupo de direcciones IP públicas y un prefijo de dirección IP pública que se usarán en los flujos salientes traducidos por el recurso de puerta de enlace de NAT.
  - Cambie el tiempo de espera de inactividad de 4 (el valor predeterminado) a 10 minutos.

Cree una puerta de enlace de NAT de Azure global con [az network nat gateway create](https://docs.microsoft.com/cli/azure/network/nat?view=azure-cli-latest) llamada **myNATgateway**. El comando usa la dirección IP pública **myPublicIP** y el prefijo de dirección IP pública **myPublicIPprefix**. El comando también cambia el tiempo de espera de inactividad a 10 minutos.

```azurecli-interactive
  az network nat gateway create \
    --resource-group myResourceGroupNAT \
    --name myNATgateway \
    --public-ip-addresses myPublicIPsource \
    --public-ip-prefixes myPublicIPprefixsource \
    --idle-timeout 10       
    
  ```

En este momento, la puerta de enlace de NAT es funcional y todo lo que queda es configurar qué subredes de una red virtual deben usarla.

## <a name="prepare-the-source-for-outbound-traffic"></a>Preparación del origen para el tráfico saliente

Se le va a guiar por la configuración de un entorno de prueba completo. Configurará una prueba mediante herramientas de código abierto para comprobar la puerta de enlace de NAT. Se comenzará con el origen, que usará la puerta de enlace de NAT que creamos anteriormente.

### <a name="configure-virtual-network-for-source"></a>Configuración de una red virtual para el origen

Antes de implementar una máquina virtual y probar la puerta de enlace de NAT, es necesario crear la red virtual.

Cree una red virtual llamada **myVnetsource** con una subred llamada **mySubnetsource** en el grupo de recursos **myResourceGroupNAT** con [az network Microsoft Azure Virtual Network create](https://docs.microsoft.com/cli/azure/network/vnet).  El espacio de direcciones IP de la red virtual es **192.168.0.0/16**. La subred de la red virtual es **192.168.0.0/24**.

```azurecli-interactive
  az network vnet create \
    --resource-group myResourceGroupNAT \
    --name myVnetsource \
    --address-prefix 192.168.0.0/16 \
    --subnet-name mySubnetsource \
    --subnet-prefix 192.168.0.0/24
    
```

### <a name="configure-nat-service-for-source-subnet"></a>Configuración del servicio NAT para la subred de origen

Configure la subred de origen **mySubnetsource** de la red virtual **myVnetsource** para usar el recurso de puerta de enlace de NAT específico **myNATgateway** con [az network Microsoft Azure Virtual Network subnet update](https://docs.microsoft.com/cli/azure/network/vnet/subnet). Este comando activará el servicio NAT en la subred especificada.

```azurecli-interactive
    az network vnet subnet update \
    --resource-group myResourceGroupNAT \
    --vnet-name myVnetsource \
    --name mySubnetsource \
    --nat-gateway myNATgateway
    
```

Todo el tráfico saliente a destinos de Internet usa ahora el servicio NAT.  No es necesario configurar una UDR.

Antes de poder probar la puerta de enlace de NAT, es necesario crear una máquina virtual de origen.  Se va a asignar un recurso de dirección IP pública como una dirección IP pública de nivel de instancia para acceder a esta máquina virtual desde el exterior. Esta dirección solo se usa para el acceso de prueba.  Se va a demostrar cómo el servicio NAT tiene prioridad sobre otras opciones de salida.

Como ejercicio, también puede crear esta máquina virtual sin una dirección IP pública y crear otra máquina virtual para usarla como JumpBox sin una dirección IP pública.

### <a name="create-public-ip-for-source-vm"></a>Creación de una dirección IP pública para la máquina virtual de origen

Se va a crear una dirección IP pública que se usará para acceder a la máquina virtual de origen. Use [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip) para crear un recurso de dirección IP pública llamado **myPublicIPsourceVM** en **myResourceGroupNAT**.

```azurecli-interactive
  az network public-ip create \
    --resource-group myResourceGroupNAT \
    --name myPublicIPsourceVM \
    --sku standard
    
```

### <a name="create-an-nsg-for-source-vm"></a>Creación de un grupo de seguridad de red para la máquina virtual de origen

Dado que las direcciones IP públicas estándar son "seguras de forma predeterminada", es necesario crear un grupo de seguridad de red para permitir el acceso de entrada mediante SSH.  El servicio Azure NAT depende de la dirección del flujo. Este grupo de seguridad de red no se usará para la salida una vez que la puerta de enlace de NAT esté configurada en la misma subred. Use [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg?view=azure-cli-latest#az-network-nsg-create) para crear un recurso de grupo de seguridad de red llamado **myNSGsource** en **myResourceGroupNAT**.

```azurecli-interactive
  az network nsg create \
    --resource-group myResourceGroupNAT \
    --name myNSGsource 
    
```

### <a name="expose-ssh-endpoint-on-source-vm"></a>Exposición del punto de conexión SSH en la máquina virtual de origen

Se va a crear una regla en el grupo de seguridad de red para el acceso mediante SSH a la máquina virtual de origen. Use [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule?view=azure-cli-latest#az-network-nsg-rule-create) para crear una regla de grupo de seguridad de red llamada **ssh**. Esta regla se creará en el grupo de seguridad de red llamado **myNSGsource** en el grupo de recursos **myResourceGroupNAT**.

```azurecli-interactive
  az network nsg rule create \
    --resource-group myResourceGroupNAT \
    --nsg-name myNSGsource \
    --priority 100 \
    --name ssh \
    --description "SSH access" \
    --access allow \
    --protocol tcp \
    --direction inbound \
    --destination-port-ranges 22
    
```

### <a name="create-nic-for-source-vm"></a>Creación de una NIC para la máquina virtual de origen

Cree una interfaz de red con el comando [az network nic create](/cli/azure/network/nic#az-network-nic-create) y asóciela a la dirección IP pública y al grupo de seguridad de red. 

```azurecli-interactive
  az network nic create \
    --resource-group myResourceGroupNAT \
    --name myNicsource \
    --vnet-name myVnetsource \
    --subnet mySubnetsource \
    --public-ip-address myPublicIPSourceVM \
    --network-security-group myNSGsource
    
```

### <a name="create-a-source-vm"></a>Creación de una máquina virtual de origen

Cree la máquina virtual con [az vm create](/cli/azure/vm#az-vm-create).  Se van a generar claves SSH para esta máquina virtual y se va a almacenar la clave privada para usarla más adelante.

```azurecli-interactive
  az vm create \
    --resource-group myResourceGroupNAT \
    --name myVMsource \
    --nics myNicsource \
    --image UbuntuLTS \
    --generate-ssh-keys \
    --no-wait
    
```

Aunque el comando devolverá resultados inmediatamente, la máquina virtual puede tardar unos minutos en implementarse.

## <a name="prepare-destination-for-outbound-traffic"></a>Preparación del destino para el tráfico saliente

Ahora se va a crear un destino para el tráfico saliente traducido por el servicio NAT para que pueda probarlo.

### <a name="configure-virtual-network-for-destination"></a>Configuración de una red virtual para el destino

 Es necesario crear una red virtual donde estará la máquina virtual de destino.  Estos comandos son los mismos que para la máquina virtual de origen con pequeños cambios para exponer el punto de conexión de destino.

Cree una red virtual llamada **myVnetdestination** con una subred llamada **mySubnetdestination** en el grupo de recursos **myResourceGroupNAT** con [az network Microsoft Azure Virtual Network create](https://docs.microsoft.com/cli/azure/network/vnet).  El espacio de direcciones IP de la red virtual es **192.168.0.0/16**. La subred de la red virtual es **192.168.0.0/24**.

```azurecli-interactive
  az network vnet create \
    --resource-group myResourceGroupNAT \
    --name myVnetdestination \
    --address-prefix 192.168.0.0/16 \
    --subnet-name mySubnetdestination \
    --subnet-prefix 192.168.0.0/24
    
```

### <a name="create-public-ip-for-destination-vm"></a>Creación de una dirección IP pública para la máquina virtual de destino

Se va a crear una dirección IP pública que se usará para acceder a la máquina virtual de origen. Use [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip) para crear un recurso de dirección IP pública llamado **myPublicIPdestinationVM** en **myResourceGroupNAT**. 

```azurecli-interactive
  az network public-ip create \
  --resource-group myResourceGroupNAT \
  --name myPublicIPdestinationVM \
  --sku standard
  
```

### <a name="create-an-nsg-for-destination-vm"></a>Creación de un grupo de seguridad de red para la máquina virtual de destino

Las direcciones IP públicas estándar son "seguras de forma predeterminada", por lo que deberá crear un grupo de seguridad de red para permitir el acceso de entrada mediante SSH. El servicio Azure NAT depende de la dirección del flujo. Este grupo de seguridad de red no se usará para la salida una vez que la puerta de enlace de NAT esté configurada en la misma subred. Use [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg?view=azure-cli-latest#az-network-nsg-create) para crear un recurso de grupo de seguridad de red llamado **myNSGdestination** en **myResourceGroupNAT**.

```azurecli-interactive
    az network nsg create \
    --resource-group myResourceGroupNAT \
    --name myNSGdestination
    
```

### <a name="expose-ssh-endpoint-on-destination-vm"></a>Exposición del punto de conexión SSH en la máquina virtual de destino

Se va a crear una regla en el grupo de seguridad de red para el acceso mediante SSH a la máquina virtual de destino. Use [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule?view=azure-cli-latest#az-network-nsg-rule-create) para crear una regla de grupo de seguridad de red llamada **ssh**. Esta regla se creará en el grupo de seguridad de red llamado **myNSGdestination** en el grupo de recursos **myResourceGroupNAT**.

```azurecli-interactive
    az network nsg rule create \
    --resource-group myResourceGroupNAT \
    --nsg-name myNSGdestination \
    --priority 100 \
    --name ssh \
    --description "SSH access" \
    --access allow \
    --protocol tcp \
    --direction inbound \
    --destination-port-ranges 22
    
```

### <a name="expose-http-endpoint-on-destination-vm"></a>Exposición de un punto de conexión HTTP en la máquina virtual de destino

Se va a crear una regla en el grupo de seguridad de red para el acceso mediante HTTP a la máquina virtual de destino. Use [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule?view=azure-cli-latest#az-network-nsg-rule-create) para crear una regla de grupo de seguridad de red llamada **http** en el NSG llamado **myNSGdestination** en **myResourceGroupNAT**.

```azurecli-interactive
    az network nsg rule create \
    --resource-group myResourceGroupNAT \
    --nsg-name myNSGdestination \
    --priority 101 \
    --name http \
    --description "HTTP access" \
    --access allow \
    --protocol tcp \
    --direction inbound \
    --destination-port-ranges 80
    
```

### <a name="create-nic-for-destination-vm"></a>Creación de una NIC para la máquina virtual de destino

Cree una interfaz de red con [az network nic create](/cli/azure/network/nic#az-network-nic-create) y asóciela a la dirección IP pública **myPublicIPdestinationVM** y al grupo de seguridad de red **myNSGdestination**. 

```azurecli-interactive
    az network nic create \
    --resource-group myResourceGroupNAT \
    --name myNicdestination \
    --vnet-name myVnetdestination \
    --subnet mySubnetdestination \
    --public-ip-address myPublicIPdestinationVM \
    --network-security-group myNSGdestination
    
```

### <a name="create-a-destination-vm"></a>Creación de una máquina virtual de destino

Cree la máquina virtual con [az vm create](/cli/azure/vm#az-vm-create).  Se van a generar claves SSH para esta máquina virtual y se va a almacenar la clave privada para usarla más adelante.

 ```azurecli-interactive
    az vm create \
    --resource-group myResourceGroupNAT \
    --name myVMdestination \
    --nics myNicdestination \
    --image UbuntuLTS \
    --generate-ssh-keys \
    --no-wait
    
```
Aunque el comando devolverá resultados inmediatamente, la máquina virtual puede tardar unos minutos en implementarse.

## <a name="prepare-a-web-server-and-test-payload-on-destination-vm"></a>Preparación de un servidor web y prueba de la carga en la máquina virtual de destino

En primer lugar, es necesario detectar la dirección IP de la máquina virtual de destino.  Para obtener la dirección IP pública de la máquina virtual de destino, use [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show). 

```azurecli-interactive
  az network public-ip show \
    --resource-group myResourceGroupNAT \
    --name myPublicIPdestinationVM \
    --query [ipAddress] \
    --output tsv
    
``` 

>[!IMPORTANT]
>Copie la dirección IP pública y, luego, péguela en un bloc de notas para que pueda usarla en pasos posteriores. Indique que esta es la máquina virtual de destino.

### <a name="sign-in-to-destination-vm"></a>Inicio de sesión en la máquina virtual de destino

Las credenciales de SSH deben almacenarse en la instancia de Cloud Shell de la operación anterior.  Abra [Azure Cloud Shell](https://shell.azure.com) en el explorador. Use la dirección IP recuperada en el paso anterior para conectarse a la máquina virtual mediante SSH. 

```bash
ssh <ip-address-destination>
```

Una vez que haya iniciado sesión, copie y pegue los siguientes comandos.  

```bash
sudo apt -y update && \
sudo apt -y upgrade && \
sudo apt -y install nginx && \
sudo ln -sf /dev/null /var/log/nginx/access.log && \
sudo touch /var/www/html/index.html && \
sudo rm /var/www/html/index.nginx-debian.html && \
sudo dd if=/dev/zero of=/var/www/html/100k bs=1024 count=100

```

Estos comandos actualizarán la máquina virtual, instalarán Nginx y crearán un archivo de 100 Kbytes. Este archivo se recuperará de la máquina virtual de origen mediante el servicio NAT.

Cierre la sesión SSH con la máquina virtual de destino.

## <a name="prepare-test-on-source-vm"></a>Preparación de la prueba en la máquina virtual de origen

En primer lugar, es necesario detectar la dirección IP de la máquina virtual de origen.  Para obtener la dirección IP pública de la máquina virtual de origen, use [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show). 

```azurecli-interactive
  az network public-ip show \
    --resource-group myResourceGroupNAT \
    --name myPublicIPsourceVM \
    --query [ipAddress] \
    --output tsv
    
``` 

>[!IMPORTANT]
>Copie la dirección IP pública y, luego, péguela en un bloc de notas para que pueda usarla en pasos posteriores. Indique que esta es la máquina virtual de origen.

### <a name="sign-in-to-source-vm"></a>Inicio de sesión en la máquina virtual de origen

De nuevo, las credenciales de SSH se almacenan en Cloud Shell. Abra una nueva pestaña de [Azure Cloud Shell](https://shell.azure.com) en el explorador.  Use la dirección IP recuperada en el paso anterior para conectarse a la máquina virtual mediante SSH. 

```bash
ssh <ip-address-source>
```

Copie y pegue los siguientes comandos para preparar la prueba del servicio NAT.

```bash
sudo apt -y update && \
sudo apt -y upgrade && \
sudo apt install -y nload golang && \
echo 'export GOPATH=${HOME}/go' >> .bashrc && \
echo 'export PATH=${PATH}:${GOPATH}/bin' >> .bashrc && \
. ~/.bashrc &&
go get -u github.com/rakyll/hey

```

Este comando actualizará la máquina virtual, instalará Go, instalará [Hey](https://github.com/rakyll/hey) de GitHub y actualizará el entorno de Shell.

Ahora está listo para probar el servicio NAT.

## <a name="validate-nat-service"></a>Validación del servicio NAT

Mientras ha iniciado sesión en la máquina virtual de origen, puede usar **cURL** y **Hey** para generar solicitudes a la dirección IP de destino.

Use cURL para recuperar el archivo de 100 Kbytes.  Reemplace **\<ip-address-destination>** en el ejemplo siguiente por la dirección IP de destino que copió anteriormente.  El parámetro **--output** indica que se descartará el archivo recuperado.

```bash
curl http://<ip-address-destination>/100k --output /dev/null
```

También puede generar una serie de solicitudes mediante **Hey**. Una vez más, reemplace **\<ip-address-destination>** por la dirección IP de destino que copió anteriormente.

```bash
hey -n 100 -c 10 -t 30 --disable-keepalive http://<ip-address-destination>/100k
```

Este comando generará 100 solicitudes, 10 de ellas a la vez, con un tiempo de espera de 30 segundos. La conexión TCP no se volverá a usar.  Cada solicitud recuperará 100 Kbytes.  Al final de la ejecución, **Hey** notificará algunas estadísticas sobre el rendimiento del servicio NAT.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, puede usar el comando [az group delete](/cli/azure/group#az-group-delete) para quitar el grupo de recursos y todos los recursos que contiene.

```azurecli-interactive 
  az group delete --name myResourceGroupNAT
  
```

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha creado una puerta de enlace de NAT y una máquina virtual de origen y destino y, luego, ha probado la puerta de enlace de NAT.

Revise las métricas de Azure Monitor para ver el funcionamiento del servicio NAT. Diagnostique problemas, como el agotamiento de recursos de los puertos SNAT disponibles.  El agotamiento de los recursos de los puertos SNAT se soluciona fácilmente agregando recursos de dirección IP pública o recursos de prefijo de dirección IP pública adicionales (o ambos).

- Obtenga más información sobre [Virtual Network NAT](./nat-overview.md).
- Obtenga más información sobre [recursos de puerta de enlace de NAT](./nat-gateway-resource.md).
- Inicio rápido para la implementación de [recursos de puerta de enlace de NAT con la CLI de Azure](./quickstart-create-nat-gateway-cli.md).
- Inicio rápido para la implementación de [recursos de puerta de enlace de NAT con PowerShell](./quickstart-create-nat-gateway-powershell.md).
- Inicio rápido para la implementación de [recursos de puerta de enlace de NAT con Azure Portal](./quickstart-create-nat-gateway-portal.md).

> [!div class="nextstepaction"]

