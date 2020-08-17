---
title: Uso de Docker Compose
description: Instalación y uso de Docker y Compose en máquinas virtuales Linux mediante la CLI de Azure.
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 02/14/2019
ms.author: cynthn
ms.custom: devx-track-azurecli
ms.openlocfilehash: e9f824a3b7e1e5e482bbc04c448ca4930bf88678
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500953"
---
# <a name="get-started-with-docker-and-compose-to-define-and-run-a-multi-container-application-in-azure"></a>Introducción a Docker y Compose para definir y ejecutar una aplicación de contenedores múltiples en Azure
Con [Compose](https://github.com/docker/compose), usará un archivo de texto simple para definir una aplicación compuesta de varios contenedores de Docker. Después, puede poner en marcha la aplicación con un solo comando que realiza todos los pasos para implementarla en el entorno definido. Como ejemplo, en este artículo se muestra cómo configurar rápidamente un blog de WordPress con una base de datos SQL MariaDB de back-end en una máquina virtual Ubuntu. También puede utilizar Compose para configurar aplicaciones más complejas.

Este artículo se probó por última vez el 14/2/2019 mediante [Azure Cloud Shell](https://shell.azure.com/bash) y la versión 2.0.58 de la [CLI de Azure](/cli/azure/install-azure-cli).

## <a name="create-docker-host-with-azure-cli"></a>Creación de un host de Docker con la CLI de Azure
Instale la última versión de la [CLI de Azure](/cli/azure/install-az-cli2) e inicie sesión en una cuenta de Azure con [az login](/cli/azure/reference-index).

En primer lugar, cree un grupo de recursos para su entorno de Docker con [az group create](/cli/azure/group). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli-interactive
az group create --name myDockerGroup --location eastus
```

Cree un archivo denominado *cloud-init.txt* y pegue la siguiente configuración. Escriba `sensible-editor cloud-init.txt` para crear el archivo y ver una lista de editores disponibles. 

```yaml
#include https://get.docker.com
```

Ahora cree una máquina virtual con el comando [az vm create](/cli/azure/vm#az-vm-create). Use el parámetro `--custom-data` para pasar su archivo de configuración cloud-init. Proporcione la ruta de acceso completa a la configuración de *cloud-init.txt* si guardó el archivo fuera de su directorio de trabajo actual. En el ejemplo siguiente se crea una máquina virtual denominada *myDockerVM* y abre el puerto 80 para el tráfico web.

```azurecli-interactive
az vm create \
    --resource-group myDockerGroup \
    --name myDockerVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt
az vm open-port --port 80 \
    --resource-group myDockerGroup \
    --name myDockerVM
```

Transcurren unos minutos hasta que la máquina virtual se crea, los paquetes se instalan y la aplicación se inicia. Hay tareas en segundo plano que continúan ejecutándose después de que la CLI de Azure vuelve a abrir el símbolo del sistema. Cuando se haya creado la máquina virtual, anote el valor `publicIpAddress` mostrado por la CLI de Azure. 

                 

## <a name="install-compose"></a>Instalación de Compose


SSH a la nueva máquina virtual de host Docker. Proporcione su propia dirección IP.

```bash
ssh azureuser@10.10.111.11
```

Instale Compose en la máquina virtual.

```bash
sudo apt install docker-compose
```


## <a name="create-a-docker-composeyml-configuration-file"></a>Creación de un archivo de configuración docker-compose.yml
Cree un archivo de configuración `docker-compose.yml` para definir los contenedores de Docker que ejecutar la máquina virtual. El archivo especifica la imagen que se ejecutará en cada contenedor, variables de entorno y dependencias necesarias, puertos y vínculos entre contenedores. Para más información sobre la sintaxis del archivo YML, consulte la [referencia del archivo de Compose](https://docs.docker.com/compose/compose-file/).

Cree un archivo *docker-compose.yml*. Use el editor de texto que prefiera para agregar algunos datos al archivo. El ejemplo siguiente crea el archivo con un mensaje `sensible-editor` para elegir el editor que desea usar.

```bash
sensible-editor docker-compose.yml
```

Pegue el ejemplo siguiente en el archivo de Docker Compose. Esta configuración usa imágenes del [Registro de DockerHub](https://registry.hub.docker.com/_/wordpress/) para instalar WordPress (el sistema de administración de contenido y blogs de código abierto) y una base de datos SQL MariaDB de back-end vinculada. Escriba su propia *MYSQL_ROOT_PASSWORD*.

```yml
wordpress:
  image: wordpress
  links:
    - db:mysql
  ports:
    - 80:80

db:
  image: mariadb
  environment:
    MYSQL_ROOT_PASSWORD: <your password>
```

## <a name="start-the-containers-with-compose"></a>inicio de los contenedores con Compose
En el mismo directorio que el archivo *docker-compose.yml*, ejecute el comando siguiente (dependiendo de su entorno, quizás tenga que ejecutar `docker-compose` mediante `sudo`):

```bash
sudo docker-compose up -d
```

Este comando inicia los contenedores de Docker especificados en *docker-compose.yml*. Este paso tarda uno o dos minutos en completarse. Verá un resultado similar al siguiente:

```
Creating wordpress_db_1...
Creating wordpress_wordpress_1...
...
```


Para comprobar que los contenedores están activos, escriba `sudo docker-compose ps`. Debería ver algo parecido a lo siguiente:

```
        Name                       Command               State         Ports
-----------------------------------------------------------------------------------
azureuser_db_1          docker-entrypoint.sh mysqld      Up      3306/tcp
azureuser_wordpress_1   docker-entrypoint.sh apach ...   Up      0.0.0.0:80->80/tcp
```

Ahora puede conectarse a WordPress directamente en la máquina virtual a través del puerto 80. Abra un explorador web y escriba el nombre de la dirección IP de la máquina virtual. Ahora debería ver la pantalla de inicio de WordPress, donde se puede completar la instalación e iniciar la aplicación.

![Pantalla de inicio de WordPress](./media/docker-compose-quickstart/wordpressstart.png)

## <a name="next-steps"></a>Pasos siguientes
* Consulte la [referencia de línea de comandos de Compose](https://docs.docker.com/compose/reference/) y el [manual del usuario](https://docs.docker.com/compose/) para obtener más ejemplos de compilación e implementación de aplicaciones con varios contenedores.
* Use una plantilla del Administrador de recursos de Azure, o bien una propia o una proporcionada por la [comunidad](https://azure.microsoft.com/documentation/templates/), para implementar una VM de Azure con Docker y una aplicación configurada con Compose. Por ejemplo, la plantilla [Implementación de un blog de WordPress con Docker](https://github.com/Azure/azure-quickstart-templates/tree/master/docker-wordpress-mysql) usa Docker y Compose para implementar rápidamente WordPress con un back-end de MySQL en una máquina virtual de Ubuntu.
* Pruebe a integrar Docker Compose con un clúster de Docker Swarm. Consulte [Using Compose with Swarm](https://docs.docker.com/compose/swarm/) (Uso de Compose con Swarm) para conocer distintos escenarios.
