---
title: "Creación de una máquina virtual Linux en Azure mediante la CLI | Microsoft Docs"
description: "Creación de una máquina virtual Linux en Azure mediante la CLI."
services: virtual-machines-linux
documentationcenter: 
author: vlivech
manager: timlt
editor: 
ms.assetid: facb1115-2b4e-4ef3-9905-330e42beb686
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: hero-article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 10/27/2016
ms.author: v-livech
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: fd75ab9a37dfc75679427a16c3ecb36adb1c9925


---
# <a name="create-a-linux-vm-on-azure-by-using-the-cli"></En este artículo se muestra cómo implementar rápidamente una máquina virtual Linux en Azure mediante el comando `azure vm quick-create` en la interfaz de la línea de comandos (CLI) de Azure. El comando `quick-create` implementa una máquina virtual dentro de una infraestructura básica y segura que puede usar para crear un prototipo o hacer una prueba de concepto rápidamente. Este artículo requiere:
a>Creación de una máquina virtual Linux en Azure mediante la CLI

* una cuenta de Azure ([obtenga aquí una prueba gratuita](https://azure.microsoft.com/pricing/free-trial/));
* la [CLI de Azure](../xplat-cli-install.md) en la que se inició sesión con `azure login`;
* la CLI de Azure *debe estar en* el modo Azure Resource Manager `azure config mode arm`.

También puede implementar rápidamente una máquina virtual Linux mediante [Azure Portal](virtual-machines-linux-quick-create-portal.md).

## <a name="quick-commands"></a>Comandos rápidos
En el ejemplo siguiente se muestra cómo implementar una máquina virtual CoreOS y asociar su clave de Shell seguro (SSH) (los argumentos pueden ser diferentes):

```azurecli
azure vm quick-create -M ~/.ssh/id_rsa.pub -Q CoreOS
```

## <a name="detailed-walkthrough"></a>Tutorial detallado
En el tutorial siguiente, se implementa, paso a paso, una máquina virtual UbuntuLTS, con explicaciones de lo que se hace en cada paso.

## <a name="vm-quickcreate-aliases"></a>Alias de creación rápida para máquinas virtuales
Una forma rápida de elegir una distribución es usar los alias de la CLI de Azure asignados a las distribuciones de sistemas operativos más comunes. En la siguiente tabla se enumeran los alias (desde la versión 0.10 de la CLI de Azure). Todas las implementaciones que usan `quick-create` utilizan de forma predeterminada las máquinas virtuales que están respaldadas por el almacenamiento en una unidad de estado sólido (SSD), lo que proporciona un aprovisionamiento más rápido y un acceso al disco de alto rendimiento. (Estos alias representan una mínima parte de las distribuciones disponibles en Azure. Para encontrar más imágenes en Azure Marketplace, [busque una imagen en PowerShell](virtual-machines-linux-cli-ps-findimage.md), búsquela [en la Web](https://azure.microsoft.com/marketplace/virtual-machines/) o [cargue su propia imagen personalizada](virtual-machines-linux-create-upload-generic.md)).

| Alias | Publicador | Oferta | SKU | Versión |
|:--- |:--- |:--- |:--- |:--- |
| CentOS |OpenLogic |CentOS |7,2 |más reciente |
| CoreOS |CoreOS |CoreOS |Stable |más reciente |
| Debian |credativ |Debian |8 |más reciente |
| openSUSE |SUSE |openSUSE |13.2 |más reciente |
| RHEL |Red Hat |RHEL |7,2 |más reciente |
| UbuntuLTS |Canonical |Ubuntu Server |14.04.4-LTS |más reciente |

En las siguientes secciones, se usa el alias `UbuntuLTS` de la opción **ImageURN** (`-Q`) para implementar una instancia de Ubuntu 14.04.4 LTS Server.

En el anterior ejemplo `quick-create` solo se llama a la marca `-M` para identificar la clave pública de SSH que se cargará, al tiempo que se deshabilitan las contraseñas de SSH, por lo que se le solicitan los siguientes argumentos:

* El nombre del grupo de recursos (normalmente cualquier cadena vale para el primer grupo de recursos de Azure).
* Nombre de la máquina virtual
* Ubicación (`westus` o `westeurope` son valores predeterminados adecuados).
* Linux (para informar a Azure qué sistema operativo desea).
* nombre de usuario

En el ejemplo siguiente se especifican todos los valores para que no sea necesaria ninguna otra solicitud. Siempre que tenga un archivo de clave pública `~/.ssh/id_rsa.pub` con formato ssh-rsa, funciona tal cual:

```azurecli
azure vm quick-create \
  --resource-group myResourceGroup \
  --name myVM \
  --location westus \
  --os-type Linux \
  --admin-username myAdminUser \
  --ssh-public-file ~/.ssh/id_rsa.pub \
  --image-urn UbuntuLTS
```

La salida debe tener un aspecto similar al siguiente bloque de salida:

```azurecli
info:    Executing command vm quick-create
+ Listing virtual machine sizes available in the location "westus"
+ Looking up the VM "myVM"
info:    Verifying the public key SSH file: /Users/ahmet/.ssh/id_rsa.pub
info:    Using the VM Size "Standard_DS1"
info:    The [OS, Data] Disk or image configuration requires storage account
+ Looking up the storage account cli16330708391032639673
+ Looking up the NIC "examp-westu-1633070839-nic"
info:    An nic with given name "examp-westu-1633070839-nic" not found, creating a new one
+ Looking up the virtual network "examp-westu-1633070839-vnet"
info:    Preparing to create new virtual network and subnet
/ Creating a new virtual network "examp-westu-1633070839-vnet" [address prefix: "10.0.0.0/16"] with subnet "examp-westu-1633070839-snet" [address prefix: "10.+.1.0/24"]
+ Looking up the virtual network "examp-westu-1633070839-vnet"
+ Looking up the subnet "examp-westu-1633070839-snet" under the virtual network "examp-westu-1633070839-vnet"
info:    Found public ip parameters, trying to setup PublicIP profile
+ Looking up the public ip "examp-westu-1633070839-pip"
info:    PublicIP with given name "examp-westu-1633070839-pip" not found, creating a new one
+ Creating public ip "examp-westu-1633070839-pip"
+ Looking up the public ip "examp-westu-1633070839-pip"
+ Creating NIC "examp-westu-1633070839-nic"
+ Looking up the NIC "examp-westu-1633070839-nic"
+ Looking up the storage account clisto1710997031examplev
+ Creating VM "myVM"
+ Looking up the VM "myVM"
+ Looking up the NIC "examp-westu-1633070839-nic"
+ Looking up the public ip "examp-westu-1633070839-pip"
data:    Id                              :/subscriptions/2<--snip-->d/resourceGroups/exampleResourceGroup/providers/Microsoft.Compute/virtualMachines/exampleVMName
data:    ProvisioningState               :Succeeded
data:    Name                            :exampleVMName
data:    Location                        :westus
data:    Type                            :Microsoft.Compute/virtualMachines
data:
data:    Hardware Profile:
data:      Size                          :Standard_DS1
data:
data:    Storage Profile:
data:      Image reference:
data:        Publisher                   :Canonical
data:        Offer                       :UbuntuServer
data:        Sku                         :14.04.4-LTS
data:        Version                     :latest
data:
data:      OS Disk:
data:        OSType                      :Linux
data:        Name                        :clic7fadb847357e9cf-os-1473374894359
data:        Caching                     :ReadWrite
data:        CreateOption                :FromImage
data:        Vhd:
data:          Uri                       :https://cli16330708391032639673.blob.core.windows.net/vhds/clic7fadb847357e9cf-os-1473374894359.vhd
data:
data:    OS Profile:
data:      Computer Name                 :myVM
data:      User Name                     :myAdminUser
data:      Linux Configuration:
data:        Disable Password Auth       :true
data:
data:    Network Profile:
data:      Network Interfaces:
data:        Network Interface #1:
data:          Primary                   :true
data:          MAC Address               :00-0D-3A-33-42-FB
data:          Provisioning State        :Succeeded
data:          Name                      :examp-westu-1633070839-nic
data:          Location                  :westus
data:            Public IP address       :138.91.247.29
data:            FQDN                    :examp-westu-1633070839-pip.westus.cloudapp.azure.com
data:
data:    Diagnostics Profile:
data:      BootDiagnostics Enabled       :true
data:      BootDiagnostics StorageUri    :https://clisto1710997031examplev.blob.core.windows.net/
data:
data:      Diagnostics Instance View:
info:    vm quick-create command OK
```

## <a name="log-in-to-the-new-vm"></a>Inicio de sesión en la nueva máquina virtual
Inicie sesión en la máquina virtual con la dirección IP pública indicada en la salida. También puede utilizar el nombre de dominio completo (FQDN) que aparece:

```bash
ssh -i ~/.ssh/id_rsa.pub ahmet@138.91.247.29
```

El proceso de inicio de sesión debe tener un aspecto similar al siguiente bloque de salida:

```bash
Warning: Permanently added '138.91.247.29' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 14.04.4 LTS (GNU/Linux 3.19.0-65-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Thu Sep  8 22:50:57 UTC 2016

  System load: 0.63              Memory usage: 2%   Processes:       81
  Usage of /:  39.6% of 1.94GB   Swap usage:   0%   Users logged in: 0

  Graph this data and manage this system at:
    https://landscape.canonical.com/

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

myAdminUser@myVM:~$
```

## <a name="next-steps"></a>Pasos siguientes
El comando `azure vm quick-create` es una forma rápida de implementar una máquina virtual para poder iniciar sesión en un shell de Bash y empezar a trabajar. Sin embargo, el uso de `vm quick-create` no ofrece un control amplio ni permite crear un entorno más complejo.  Para implementar una máquina virtual Linux personalizada para su infraestructura, puede seguir cualquiera de estos artículos.

* [Uso de una plantilla de Azure Resource Manager para crear una implementación específica](virtual-machines-linux-cli-deploy-templates.md)
* [Creación de un entorno personalizado para una máquina virtual Linux mediante el uso de comandos de la CLI de Azure directamente](virtual-machines-linux-create-cli-complete.md)
* [Creación de una máquina virtual Linux protegida con SSH en Azure mediante plantillas](virtual-machines-linux-create-ssh-secured-vm-from-template.md)

También puede [usar el controlador `docker-machine` de Azure con diversos comandos para crear rápidamente una máquina virtual Linux como host de docker](virtual-machines-linux-docker-machine.md).




<!--HONumber=Nov16_HO2-->


