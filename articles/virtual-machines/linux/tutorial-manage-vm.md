---
title: 'Tutorial: Creación y administración de máquinas virtuales Linux con la CLI de Azure | Microsoft Docs'
description: En este tutorial, aprenderá a usar la CLI de Azure para crear y administrar máquinas virtuales Linux en Azure
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 03/23/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 270479061ad40fdda9db06571ad4ef24b00d6c4d
ms.sourcegitcommit: 039263ff6271f318b471c4bf3dbc4b72659658ec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/06/2019
ms.locfileid: "55749149"
---
# <a name="tutorial-create-and-manage-linux-vms-with-the-azure-cli"></a>Tutorial: Creación y administración de máquinas virtuales Linux con la CLI de Azure

Las máquinas virtuales de Azure proporcionan un entorno informático completamente configurable y flexible. En este tutorial se tratan elementos básicos de la implementación de máquinas virtuales de Azure, como la selección de su tamaño, la selección de una imagen de máquina virtual y la implementación de una máquina virtual. Aprenderá a:

> [!div class="checklist"]
> * Crear y conectar elementos a una máquina virtual.
> * Seleccionar y usar imágenes de máquinas virtuales.
> * Ver y usar tamaños de una máquina virtual específicos.
> * Cambiar el tamaño de una máquina virtual.
> * Ver y entender el estado de las máquinas virtuales.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, en este tutorial es preciso que ejecute la CLI de Azure de la versión 2.0.30, u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="create-resource-group"></a>Creación de un grupo de recursos

Cree un grupo de recursos con el comando [az group create](https://docs.microsoft.com/cli/azure/group). 

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Se debe crear un grupo de recursos antes de una máquina virtual. En este ejemplo, se crea un grupo de recursos denominado *myResourceGroupVM* en la región *eastus*. 

```azurecli-interactive 
az group create --name myResourceGroupVM --location eastus
```

Se especifica el grupo de recursos al crear o modificar una máquina virtual, como se ve a lo largo de este tutorial.

## <a name="create-virtual-machine"></a>Crear máquina virtual

Cree la máquina virtual con el comando [az vm create](https://docs.microsoft.com/cli/azure/vm). 

Al crear una máquina virtual, hay varias opciones disponibles, como la imagen del sistema operativo, el tamaño del disco y las credenciales administrativas. En el ejemplo siguiente se crea una máquina virtual denominada *myVM* en la que se ejecuta Ubuntu Server. Se crea una cuenta de usuario denominada *azureuser* en la máquina virtual, y se generan claves SSH, si no existen en la ubicación de la clave predeterminada (*~/.ssh*):

```azurecli-interactive
az vm create \
    --resource-group myResourceGroupVM \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys
```

La creación de la máquina virtual puede tardar unos minutos. Una vez creada la máquina virtual, la CLI de Azure ofrece como salida información sobre la máquina virtual. Tome nota de `publicIpAddress`; una dirección que se puede usar para acceder a la máquina virtual. 

```azurecli-interactive 
{
  "fqdns": "",
  "id": "/subscriptions/d5b9d4b7-6fc1-0000-0000-000000000000/resourceGroups/myResourceGroupVM/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.174.34.95",
  "resourceGroup": "myResourceGroupVM"
}
```

## <a name="connect-to-vm"></a>Conexión con una máquina virtual

Ahora puede conectarse a la máquina virtual mediante SSH en Azure Cloud Shell o desde un equipo local. Reemplace la dirección IP de ejemplo por la dirección `publicIpAddress` que anotó en el paso anterior.

```bash
ssh azureuser@52.174.34.95
```

Una vez que inicia sesión en la máquina virtual, puede instalar y configurar las aplicaciones. Cuando haya terminado, cierre la sesión SSH como normal:

```bash
exit
```

## <a name="understand-vm-images"></a>Descripción de las imágenes de máquina virtual

Azure Marketplace incluye muchas imágenes que pueden usarse para crear VM. En los pasos anteriores, se creó una máquina virtual con una imagen de Ubuntu. En este paso, se usa la CLI de Azure para buscar en Marketplace una imagen de CentOS, que se usa para implementar una segunda máquina virtual. 

Para ver una lista de las imágenes usadas con más frecuencia, use el comando [az vm image list](/cli/azure/vm/image).

```azurecli-interactive 
az vm image list --output table
```

La salida del comando devuelve las imágenes de máquina virtual más populares en Azure.

```bash
Offer          Publisher               Sku                 Urn                                                             UrnAlias             Version
-------------  ----------------------  ------------------  --------------------------------------------------------------  -------------------  ---------
WindowsServer  MicrosoftWindowsServer  2016-Datacenter     MicrosoftWindowsServer:WindowsServer:2016-Datacenter:latest     Win2016Datacenter    latest
WindowsServer  MicrosoftWindowsServer  2012-R2-Datacenter  MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest  Win2012R2Datacenter  latest
WindowsServer  MicrosoftWindowsServer  2008-R2-SP1         MicrosoftWindowsServer:WindowsServer:2008-R2-SP1:latest         Win2008R2SP1         latest
WindowsServer  MicrosoftWindowsServer  2012-Datacenter     MicrosoftWindowsServer:WindowsServer:2012-Datacenter:latest     Win2012Datacenter    latest
UbuntuServer   Canonical               16.04-LTS           Canonical:UbuntuServer:16.04-LTS:latest                         UbuntuLTS            latest
CentOS         OpenLogic               7.3                 OpenLogic:CentOS:7.3:latest                                     CentOS               latest
openSUSE-Leap  SUSE                    42.2                SUSE:openSUSE-Leap:42.2:latest                                  openSUSE-Leap        latest
RHEL           RedHat                  7.3                 RedHat:RHEL:7.3:latest                                          RHEL                 latest
SLES           SUSE                    12-SP2              SUSE:SLES:12-SP2:latest                                         SLES                 latest
Debian         credativ                8                   credativ:Debian:8:latest                                        Debian               latest
CoreOS         CoreOS                  Stable              CoreOS:CoreOS:Stable:latest                                     CoreOS               latest
```

Para ver una lista completa, agregue el argumento `--all`. También puede filtrar la lista de imágenes por `--publisher` u `–-offer`. En este ejemplo, la lista se ha filtrado para todas las imágenes con una oferta que coincida con *CentOS*. 

```azurecli-interactive 
az vm image list --offer CentOS --all --output table
```

Salida parcial:

```azurecli-interactive 
Offer             Publisher         Sku   Urn                                     Version
----------------  ----------------  ----  --------------------------------------  -----------
CentOS            OpenLogic         6.5   OpenLogic:CentOS:6.5:6.5.201501         6.5.201501
CentOS            OpenLogic         6.5   OpenLogic:CentOS:6.5:6.5.201503         6.5.201503
CentOS            OpenLogic         6.5   OpenLogic:CentOS:6.5:6.5.201506         6.5.201506
CentOS            OpenLogic         6.5   OpenLogic:CentOS:6.5:6.5.20150904       6.5.20150904
CentOS            OpenLogic         6.5   OpenLogic:CentOS:6.5:6.5.20160309       6.5.20160309
CentOS            OpenLogic         6.5   OpenLogic:CentOS:6.5:6.5.20170207       6.5.20170207
```

Para implementar una máquina virtual mediante una imagen específica, anote el valor de la columna *Urn*, que consta del publicador, la oferta, la SKU y opcionalmente, un número de versión para [identificar](cli-ps-findimage.md#terminology) la imagen. Al especificar la imagen, se puede reemplazar el número de versión de la imagen por "latest", para que se seleccione la versión más reciente de la distribución. En este ejemplo, se emplea el argumento `--image` para especificar la versión más reciente de una imagen de CentOS 6.5.  

```azurecli-interactive 
az vm create --resource-group myResourceGroupVM --name myVM2 --image OpenLogic:CentOS:6.5:latest --generate-ssh-keys
```

## <a name="understand-vm-sizes"></a>Comprensión de los tamaños de máquina virtual

El tamaño de la máquina virtual determina la cantidad de recursos de proceso, como memoria, CPU y GPU, que están disponibles para la máquina virtual. Es necesario ajustar adecuadamente el tamaño de las máquinas virtuales para la carga de trabajo esperada. Si aumenta la carga de trabajo, se puede cambiar el tamaño de una máquina virtual existente.

### <a name="vm-sizes"></a>Tamaños de máquina virtual

En la tabla siguiente se clasifican los tamaños en casos de uso.  

| Type                     | Tamaños           |    DESCRIPCIÓN       |
|--------------------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [Uso general](sizes-general.md)         |Dsv3, Dv3, DSv2, Dv2, DS, D, Av2, A0-7| Uso equilibrado de CPU y memoria. Ideal para desarrollo/pruebas, así como soluciones de datos y aplicaciones de tamaño pequeño a mediano.  |
| [Proceso optimizado](sizes-compute.md)   | Fs, F             | Uso elevado de la CPU respecto a la memoria. Adecuado para aplicaciones, dispositivos de red y procesos por lotes con tráfico mediano.        |
| [Memoria optimizada](../virtual-machines-windows-sizes-memory.md)    | Esv3, Ev3, M, GS, G, DSv2, DS, Dv2, D   | Uso elevado de memoria respecto al núcleo. Excelente para bases de datos relacionales, memorias caché de capacidad de mediana a grande y análisis en memoria.                 |
| [Almacenamiento optimizado](../virtual-machines-windows-sizes-storage.md)      | LS                | Alto rendimiento de disco y E/S. Perfecto para bases de datos SQL y NoSQL y macrodatos.                                                         |
| [GPU](sizes-gpu.md)          | NV, NC            | Máquinas virtuales especializadas específicas para actividades intensas de representación de gráficos y edición de vídeo.       |
| [Alto rendimiento](sizes-hpc.md) | H, A8-11          | Nuestras máquinas virtuales con CPU más eficaces e interfaces de red de alto rendimiento (RDMA) opcionales. 


### <a name="find-available-vm-sizes"></a>Búsqueda de los tamaños de máquina virtual disponibles

Para ver una lista de tamaños de máquinas virtuales disponibles en una región determinada, use el comando [az vm list-sizes](/cli/azure/vm). 

```azurecli-interactive 
az vm list-sizes --location eastus --output table
```

Salida parcial:

```azurecli-interactive 
  MaxDataDiskCount    MemoryInMb  Name                      NumberOfCores    OsDiskSizeInMb    ResourceDiskSizeInMb
------------------  ------------  ----------------------  ---------------  ----------------  ----------------------
                 2          3584  Standard_DS1                          1           1047552                    7168
                 4          7168  Standard_DS2                          2           1047552                   14336
                 8         14336  Standard_DS3                          4           1047552                   28672
                16         28672  Standard_DS4                          8           1047552                   57344
                 4         14336  Standard_DS11                         2           1047552                   28672
                 8         28672  Standard_DS12                         4           1047552                   57344
                16         57344  Standard_DS13                         8           1047552                  114688
                32        114688  Standard_DS14                        16           1047552                  229376
                 1           768  Standard_A0                           1           1047552                   20480
                 2          1792  Standard_A1                           1           1047552                   71680
                 4          3584  Standard_A2                           2           1047552                  138240
                 8          7168  Standard_A3                           4           1047552                  291840
                 4         14336  Standard_A5                           2           1047552                  138240
                16         14336  Standard_A4                           8           1047552                  619520
                 8         28672  Standard_A6                           4           1047552                  291840
                16         57344  Standard_A7                           8           1047552                  619520
```

### <a name="create-vm-with-specific-size"></a>Creación de máquinas virtuales con un tamaño específico

En el anterior ejemplo de creación de máquinas virtuales, no se proporcionó ningún tamaño, lo que conlleva el uso de un tamaño predeterminado. Se puede seleccionar un tamaño para la máquina virtual al crearla con el comando [az vm create](/cli/azure/vm) y el argumento `--size`. 

```azurecli-interactive 
az vm create \
    --resource-group myResourceGroupVM \
    --name myVM3 \
    --image UbuntuLTS \
    --size Standard_F4s \
    --generate-ssh-keys
```

### <a name="resize-a-vm"></a>Cambiar el tamaño de una máquina virtual

Una vez implementada una máquina virtual, se puede cambiar su tamaño para aumentar o disminuir la asignación de recursos. El tamaño actual de una máquina virtual se puede ver con [az vm show](/cli/azure/vm):

```azurecli-interactive
az vm show --resource-group myResourceGroupVM --name myVM --query hardwareProfile.vmSize
```

Antes de cambiar el tamaño de una máquina virtual, compruebe si el tamaño deseado está disponible en el clúster de Azure actual. El comando [az vm list-vm-resize-options](/cli/azure/vm) devuelve la lista de tamaños. 

```azurecli-interactive 
az vm list-vm-resize-options --resource-group myResourceGroupVM --name myVM --query [].name
```
Si el tamaño deseado está disponible, puede cambiarlo con la máquina virtual encendida, aunque se reiniciará durante la operación. Use el comando [az vm resize]( /cli/azure/vm) para realizar el cambio de tamaño.

```azurecli-interactive 
az vm resize --resource-group myResourceGroupVM --name myVM --size Standard_DS4_v2
```

Si el tamaño deseado no está en el clúster actual, se debe desasignar la máquina virtual para que se pueda llevar a cabo la operación de cambio de tamaño. Use el comando [az vm deallocate]( /cli/azure/vm) para detener y desasignar la máquina virtual. Tenga en cuenta que, cuando se vuelve a encender la máquina virtual, es posible que se quiten todos los datos del disco temporal. Además, la dirección IP pública cambia a menos que se esté usando una estática. 

```azurecli-interactive 
az vm deallocate --resource-group myResourceGroupVM --name myVM
```

Una vez desasignada, puede realizarse el cambio de tamaño. 

```azurecli-interactive 
az vm resize --resource-group myResourceGroupVM --name myVM --size Standard_GS1
```

Tras el cambio de tamaño, se puede iniciar la máquina virtual.

```azurecli-interactive 
az vm start --resource-group myResourceGroupVM --name myVM
```

## <a name="vm-power-states"></a>Estados de una máquina virtual

Una máquina virtual de Azure puede tener uno de muchos estados de energía. Este estado representa el estado actual de la máquina virtual desde el punto de vista del hipervisor. 

### <a name="power-states"></a>Estados de energía

| Estado de energía | DESCRIPCIÓN
|----|----|
| Iniciando | Indica que se está iniciando la máquina virtual. |
| Ejecución | Indica que la máquina virtual se está ejecutando. |
| Deteniéndose | Indica que se está deteniendo la máquina virtual. | 
| Stopped | Indica que se ha detenido la máquina virtual. Las máquinas virtuales en el estado detenido siguen acumulando cargos por procesos.  |
| Desasignando | Indica que se está desasignando la máquina virtual. |
| Desasignado | Indica que la máquina virtual se quitó del hipervisor pero sigue estando disponible en el plano de control. Las máquinas virtuales en el estado Desasignado no incurren cargos por procesos. |
| - | Indica que se desconoce el estado de la máquina virtual. |

### <a name="find-the-power-state"></a>Busque el estado de energía de la máquina

Para recuperar el estado de una máquina virtual concreta, use el comando [az vm get instance-view](/cli/azure/vm). Asegúrese de especificar un nombre válido para la máquina virtual y el grupo de recursos. 

```azurecli-interactive 
az vm get-instance-view \
    --name myVM \
    --resource-group myResourceGroupVM \
    --query instanceView.statuses[1] --output table
```

Salida:

```azurecli-interactive 
ode                DisplayStatus    Level
------------------  ---------------  -------
PowerState/running  VM running       Info
```

## <a name="management-tasks"></a>Tareas de administración

Durante el ciclo de vida de una máquina virtual, es posible que desee ejecutar tareas de administración como iniciar, detener o eliminar una máquina virtual. Además, puede crear scripts para automatizar tareas repetitivas o complejas. Mediante la CLI de Azure, muchas tareas comunes de administración se pueden ejecutar desde la línea de comandos o en scripts. 

### <a name="get-ip-address"></a>Obtención de dirección IP

Este comando devuelve las direcciones IP públicas y privadas de una máquina virtual.  

```azurecli-interactive 
az vm list-ip-addresses --resource-group myResourceGroupVM --name myVM --output table
```

### <a name="stop-virtual-machine"></a>Detener una máquina virtual

```azurecli-interactive 
az vm stop --resource-group myResourceGroupVM --name myVM
```

### <a name="start-virtual-machine"></a>Iniciar una máquina virtual

```azurecli-interactive 
az vm start --resource-group myResourceGroupVM --name myVM
```

### <a name="delete-resource-group"></a>Eliminación de un grupo de recursos

Al eliminar un grupo de recursos, también se eliminan todos los recursos que contiene, como la máquina virtual, la red virtual y el disco. El parámetro `--no-wait` devuelve el control a la petición de confirmación sin esperar a que finalice la operación. El parámetro `--yes` confirma que desea eliminar los recursos sin pedir confirmación adicional.

```azurecli-interactive 
az group delete --name myResourceGroupVM --no-wait --yes
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido conceptos básicos sobre la creación y administración de máquinas virtuales. Por ejemplo:

> [!div class="checklist"]
> * Crear y conectar elementos a una máquina virtual.
> * Seleccionar y usar imágenes de máquinas virtuales.
> * Ver y usar tamaños de una máquina virtual específicos.
> * Cambiar el tamaño de una máquina virtual.
> * Ver y entender el estado de las máquinas virtuales.

Prosiga con el siguiente tutorial para aprender sobre los discos en máquinas virtuales de Azure.  

> [!div class="nextstepaction"]
> [Creación y administración de discos de máquinas virtuales](./tutorial-manage-disks.md)
