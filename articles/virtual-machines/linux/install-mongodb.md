---
title: Instalación de MongoDB en una máquina virtual Linux mediante la CLI de Azure
description: Aprenda cómo instalar y configurar MongoDB en una máquina virtual Linux con la CLI de Azure
author: cynthn
manager: gwallace
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 12/15/2017
ms.author: cynthn
ms.custom: devx-track-azurecli
ms.openlocfilehash: 49a0e48977393aeab7ff93b79e28acc55a87b51a
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87504626"
---
# <a name="how-to-install-and-configure-mongodb-on-a-linux-vm"></a>Procedimiento de instalación y configuración de MongoDB en una máquina virtual Linux

[MongoDB](https://www.mongodb.org) es una conocida base de datos NoSQL de código abierto y alto rendimiento. En este artículo se muestra cómo instalar y configurar MongoDB en una máquina virtual Linux con la CLI de Azure. Se muestran algunos ejemplos detallados de:

* [Instalación y configuración manuales de una instancia básica de MongoDB](#manually-install-and-configure-mongodb-on-a-vm)
* [Creación de una instancia básica de MongoDB mediante una plantilla de Resource Manager](#create-basic-mongodb-instance-on-centos-using-a-template)
* [Creación de un clúster complejo con particiones de MongoDB con conjuntos de réplicas mediante una plantilla de Resource Manager](#create-a-complex-mongodb-sharded-cluster-on-centos-using-a-template)


## <a name="manually-install-and-configure-mongodb-on-a-vm"></a>Instalación y configuración manuales de MongoDB en una máquina virtual
MongoDB [proporciona instrucciones de instalación](https://docs.mongodb.com/manual/administration/install-on-linux/) para distribuciones de Linux, incluidos Red Hat/CentOS, SUSE, Ubuntu y Debian. En el ejemplo siguiente se crea una máquina virtual *CentOS*. Para crear este entorno, necesita tener instalada la versión más reciente de la [CLI de Azure](/cli/azure/install-az-cli2) y haber iniciado sesión en una cuenta de Azure mediante [az login](/cli/azure/reference-index).

Cree un grupo de recursos con [az group create](/cli/azure/group). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli
az group create --name myResourceGroup --location eastus
```

Cree la máquina virtual con [az vm create](/cli/azure/vm). En el ejemplo siguiente se crea una máquina virtual llamada *myVM* con un usuario llamado *azureuser* mediante la autenticación de clave pública SSH

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image CentOS \
    --admin-username azureuser \
    --generate-ssh-keys
```

Conéctese mediante SSH a la máquina virtual con su propio nombre de usuario y la `publicIpAddress` que aparece en los resultados del paso anterior:

```bash
ssh azureuser@<publicIpAddress>
```

Para agregar los orígenes de instalación para MongoDB, cree un archivo de repositorio **yum** de la manera siguiente:

```bash
sudo touch /etc/yum.repos.d/mongodb-org-3.6.repo
```

Abra el archivo del repositorio de MongoDB para editarlo, por ejemplo, con `vi` o `nano`. Agregue las siguientes líneas:

```sh
[mongodb-org-3.6]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.6/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.6.asc
```

Instale MongoDB mediante **yum** como se indica a continuación:

```bash
sudo yum install -y mongodb-org
```

De forma predeterminada, SELinux está aplicado en las imágenes de CentOS, lo que impide el acceso de MongoDB. Instale herramientas de administración de directivas y configure SELinux para permitir que MongoDB funcione en su puerto TCP predeterminado, 27017, de la manera siguiente:

```bash
sudo yum install -y policycoreutils-python
sudo semanage port -a -t mongod_port_t -p tcp 27017
```

Inicie el servicio de MongoDB como se muestra a continuación:

```bash
sudo service mongod start
```

Para comprobar la instalación de MongoDB, conéctese por medio del cliente `mongo` local:

```bash
mongo
```

Ahora pruebe la instancia de MongoDB agregando algunos datos y después buscando:

```sh
> db
test
> db.foo.insert( { a : 1 } )  
> db.foo.find()  
{ "_id" : ObjectId("57ec477cd639891710b90727"), "a" : 1 }
> exit
```

Si lo desea, configure MongoDB para que se inicie automáticamente durante un reinicio del sistema:

```bash
sudo chkconfig mongod on
```


## <a name="create-basic-mongodb-instance-on-centos-using-a-template"></a>Creación de una instancia básica de MongoDB en CentOS mediante una plantilla
Puede crear una instancia básica de MongoDB una sola máquina virtual CentOS con la siguiente plantilla de inicio rápido de Azure en GitHub. Esta plantilla usa la extensión de script personalizada para Linux para agregar un repositorio **yum** a la máquina virtual CentOS recién creada y después instalar MongoDB.

* [Instancia básica de MongoDB en CentOS](https://github.com/Azure/azure-quickstart-templates/tree/master/mongodb-on-centos) - https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-on-centos/azuredeploy.json

Para crear este entorno, necesita tener instalada la versión más reciente de la [CLI de Azure](/cli/azure/install-az-cli2) y haber iniciado sesión en una cuenta de Azure mediante [az login](/cli/azure/reference-index). En primer lugar, cree un grupo de recursos con [az group create](/cli/azure/group). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli
az group create --name myResourceGroup --location eastus
```

Después, implemente la plantilla de MongoDB con [az group deployment create](/cli/azure/group/deployment). Cuando se le solicite, escriba sus propios valores únicos para *newStorageAccountName*, *dnsNameForPublicIP* y el nombre de usuario administrador y la contraseña:

```azurecli
az group deployment create --resource-group myResourceGroup \
  --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-on-centos/azuredeploy.json
```

Inicie sesión en la máquina virtual con la dirección DNS pública de la máquina virtual. Puede ver la dirección DNS pública con [az vm show](/cli/azure/vm):

```azurecli
az vm show -g myResourceGroup -n myLinuxVM -d --query [fqdns] -o tsv
```

Acceda mediante SSH a la máquina virtual con su propio nombre de usuario y la dirección DNS pública:

```bash
ssh azureuser@mypublicdns.eastus.cloudapp.azure.com
```

Para comprobar la instalación de MongoDB, conéctese por medio del cliente `mongo` local como sigue:

```bash
mongo
```

Ahora pruebe la instancia agregando algunos datos y después buscando:

```sh
> db
test
> db.foo.insert( { a : 1 } )  
> db.foo.find()  
{ "_id" : ObjectId("57ec477cd639891710b90727"), "a" : 1 }
> exit
```


## <a name="create-a-complex-mongodb-sharded-cluster-on-centos-using-a-template"></a>Creación de un clúster complejo con particiones de MongoDB en CentOS mediante una plantilla
Puede crear un clúster complejo con particiones de MongoDB con la siguiente plantilla de inicio rápido de Azure en GitHub. Esta plantilla sigue los [procedimientos recomendados para clústeres con particiones de MongoDB](https://docs.mongodb.com/manual/core/sharded-cluster-components/) para proporcionar alta disponibilidad y redundancia. La plantilla crea dos particiones, con tres nodos en cada conjunto de réplicas. También se crea un conjunto de réplicas de servidor de configuración con tres nodos, más dos servidores de ruta **mongos** para proporcionar coherencia a las aplicaciones entre las particiones.

* [Clúster de particionamiento de MongoDB en CentOS](https://github.com/Azure/azure-quickstart-templates/tree/master/mongodb-sharding-centos) - https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-sharding-centos/azuredeploy.json

> [!WARNING]
> Para implementar este clúster complejo con particiones de MongoDB, se requieren más de 20 núcleos, lo que suele ser el número predeterminado por región para una suscripción. Abra una solicitud de soporte técnico de Azure para aumentar el número de núcleos.

Para crear este entorno, necesita tener instalada la versión más reciente de la [CLI de Azure](/cli/azure/install-az-cli2) y haber iniciado sesión en una cuenta de Azure mediante [az login](/cli/azure/reference-index). En primer lugar, cree un grupo de recursos con [az group create](/cli/azure/group). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli
az group create --name myResourceGroup --location eastus
```

Después, implemente la plantilla de MongoDB con [az group deployment create](/cli/azure/group/deployment). Defina sus propios nombres de recursos y tamaños donde sea necesario; por ejemplo, para *mongoAdminUsername*, *sizeOfDataDiskInGB* y *configNodeVmSize*:

```azurecli
az group deployment create --resource-group myResourceGroup \
  --parameters '{"adminUsername": {"value": "azureuser"},
    "adminPassword": {"value": "P@ssw0rd!"},
    "mongoAdminUsername": {"value": "mongoadmin"},
    "mongoAdminPassword": {"value": "P@ssw0rd!"},
    "dnsNamePrefix": {"value": "mypublicdns"},
    "environment": {"value": "AzureCloud"},
    "numDataDisks": {"value": "4"},
    "sizeOfDataDiskInGB": {"value": 20},
    "centOsVersion": {"value": "7.0"},
    "routerNodeVmSize": {"value": "Standard_DS3_v2"},
    "configNodeVmSize": {"value": "Standard_DS3_v2"},
    "replicaNodeVmSize": {"value": "Standard_DS3_v2"},
    "zabbixServerIPAddress": {"value": "Null"}}' \
  --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-sharding-centos/azuredeploy.json \
  --name myMongoDBCluster \
  --no-wait
```

Esta implementación puede tardar una hora aproximadamente en realizarse y configurar todas las instancias de máquina virtual. La marca `--no-wait` se usa al final del comando anterior para devolver el control a la línea de comandos una vez que la plataforma de Azure acepte la implementación de la plantilla. Después, puede ver el estado de implementación con [az group deployment show](/cli/azure/group/deployment). En el ejemplo siguiente se visualiza el estado de la implementación de *myMongoDBCluster* en el grupo de recursos *myResourceGroup*:

```azurecli
az group deployment show \
    --resource-group myResourceGroup \
    --name myMongoDBCluster \
    --query [properties.provisioningState] \
    --output tsv
```

## <a name="next-steps"></a>Pasos siguientes
En estos ejemplos, puede conectarse a la instancia de MongoDB localmente desde la máquina virtual. Si desea conectarse a la instancia de MongoDB desde otra máquina virtual o una red, asegúrese de que [se creen las reglas de grupo de seguridad de red](nsg-quickstart.md) correspondientes.

Estos ejemplos implementan el entorno del núcleo de MongoDB para fines de desarrollo. Aplique las opciones de configuración de seguridad necesarias para su entorno. Para más información, consulte los [documentos de seguridad de MongoDB](https://docs.mongodb.com/manual/security/).

Para más información sobre la creación de plantillas, consulte [Información general de Azure Resource Manager](../../azure-resource-manager/management/overview.md).

Las plantillas de Azure Resource Manager usan la extensión de script personalizado para descargar y ejecutar scripts en las máquinas virtuales. Para más información, consulte [Uso de la extensión de script personalizado de Azure con máquinas virtuales Linux](../extensions/custom-script-linux.md).
