---
title: Uso de Docker Machine para crear hosts de Linux
description: Se describe el uso Docker Machine para crear hosts de Docker en Azure.
author: cynthn
ms.service: virtual-machines-linux
ms.devlang: multiple
ms.topic: how-to
ms.date: 12/15/2017
ms.author: cynthn
ms.openlocfilehash: 83799940f8c98952a435f582a9160585e9dbdfd4
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289790"
---
# <a name="how-to-use-docker-machine-to-create-hosts-in-azure"></a>Uso de Docker Machine para crear hosts en Azure
En este artículo se detalla cómo usar [Docker Machine](https://docs.docker.com/machine/) para crear hosts en Azure. El comando `docker-machine` crea una máquina virtual (VM) Linux en Azure para, a continuación, instalar Docker. De este modo, puede administrar los hosts de Docker en Azure con las mismas herramientas y flujos de trabajo locales. Para usar la máquina Docker en Windows 10, debe usar bash de Linux.

## <a name="create-vms-with-docker-machine"></a>Creación de VM con la máquina de Docker
En primer lugar, obtenga el identificador de suscripción de Azure con el comando [az account show](/cli/azure/account) de la manera siguiente:

```azurecli
sub=$(az account show --query "id" -o tsv)
```

Las máquinas virtuales de host de Docker se crean en Azure con `docker-machine create`, y se especifica *azure* como controlador. Para más información, consulte la [documentación del controlador de Azure para Docker](https://docs.docker.com/machine/drivers/azure/).

En el ejemplo siguiente se crea una máquina virtual llamada *myVM*, basada en el plan "Estándar D2 v2", se crea una cuenta de usuario llamada *azureuser* y se abre el puerto *80* en la máquina virtual host. Siga las indicaciones para iniciar sesión en su cuenta de Azure y conceder permisos a Docker Machine para crear y administrar recursos.

```bash
docker-machine create -d azure \
    --azure-subscription-id $sub \
    --azure-ssh-user azureuser \
    --azure-open-port 80 \
    --azure-size "Standard_DS2_v2" \
    myvm
```

La salida es similar a la siguiente:

```bash
Creating CA: /Users/user/.docker/machine/certs/ca.pem
Creating client certificate: /Users/user/.docker/machine/certs/cert.pem
Running pre-create checks...
(myvm) Completed machine pre-create checks.
Creating machine...
(myvm) Querying existing resource group.  name="docker-machine"
(myvm) Creating resource group.  name="docker-machine" location="westus"
(myvm) Configuring availability set.  name="docker-machine"
(myvm) Configuring network security group.  name="myvm-firewall" location="westus"
(myvm) Querying if virtual network already exists.  rg="docker-machine" location="westus" name="docker-machine-vnet"
(myvm) Creating virtual network.  name="docker-machine-vnet" rg="docker-machine" location="westus"
(myvm) Configuring subnet.  name="docker-machine" vnet="docker-machine-vnet" cidr="192.168.0.0/16"
(myvm) Creating public IP address.  name="myvm-ip" static=false
(myvm) Creating network interface.  name="myvm-nic"
(myvm) Creating storage account.  sku=Standard_LRS name="vhdski0hvfazyd8mn991cg50" location="westus"
(myvm) Creating virtual machine.  location="westus" size="Standard_A2" username="azureuser" osImage="canonical:UbuntuServer:16.04.0-LTS:latest" name="myvm
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with ubuntu(systemd)...
Installing Docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env myvm
```

## <a name="configure-your-docker-shell"></a>Configuración del shell de Docker
Para conectarse al host de Docker en Azure, defina la configuración de conexión adecuada. Como se indicó al final de la salida, vea la información de conexión del host de Docker como se indica a continuación: 

```bash
docker-machine env myvm
```

La salida es similar a la del ejemplo siguiente:

```bash
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://40.68.254.142:2376"
export DOCKER_CERT_PATH="/Users/user/.docker/machine/machines/machine"
export DOCKER_MACHINE_NAME="machine"
# Run this command to configure your shell:
# eval $(docker-machine env myvm)
```

Para definir la configuración de conexión, puede ejecuta el comando de configuración sugerido (`eval $(docker-machine env myvm)`), o bien puede establecer las variables de entorno manualmente. 

## <a name="run-a-container"></a>Ejecución de un contenedor
Para ver un contenedor en acción, vamos a ejecutar un servidor web básico de NGINX. Cree un contenedor con `docker run` y exponga el puerto 80 al tráfico web de la manera siguiente:

```bash
docker run -d -p 80:80 --restart=always nginx
```

La salida es similar a la del ejemplo siguiente:

```bash
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
ff3d52d8f55f: Pull complete
226f4ec56ba3: Pull complete
53d7dd52b97d: Pull complete
Digest: sha256:41ad9967ea448d7c2b203c699b429abe1ed5af331cd92533900c6d77490e0268
Status: Downloaded newer image for nginx:latest
675e6056cb81167fe38ab98bf397164b01b998346d24e567f9eb7a7e94fba14a
```

Vea los contenedores en funcionamiento con `docker ps`. La salida del ejemplo siguiente muestra el contenedor NGINX que se ejecuta con el puerto 80 expuesto:

```bash
CONTAINER ID    IMAGE    COMMAND                   CREATED          STATUS          PORTS                          NAMES
d5b78f27b335    nginx    "nginx -g 'daemon off"    5 minutes ago    Up 5 minutes    0.0.0.0:80->80/tcp, 443/tcp    festive_mirzakhani
```

## <a name="test-the-container"></a>Prueba del contenedor
Obtenga la dirección IP pública del host de Docker de la manera siguiente:


```bash
docker-machine ip myvm
```

Para ver el contenedor en acción, abra un explorador web y escriba la dirección IP pública observada en la salida del comando anterior:

![Ejecución del contenedor ngnix](./media/docker-machine/nginx.png)

## <a name="next-steps"></a>Pasos siguientes
Para ver ejemplos sobre el uso de Docker Compose, consulte [Introducción a Docker y Compose en Azure](docker-compose-quickstart.md).
