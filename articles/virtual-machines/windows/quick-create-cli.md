---
title: 'Guía de inicio rápido: creación de una máquina virtual Windows mediante la CLI de Azure'
description: En este inicio rápido aprenderá a usar la CLI de Azure para crear una máquina virtual Windows
author: cynthn
ms.service: virtual-machines-windows
ms.topic: quickstart
ms.workload: infrastructure
ms.date: 07/02/2019
ms.author: cynthn
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: bcfbcc4f78e14edc40abccf4a697b00bc7a3f014
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496192"
---
# <a name="quickstart-create-a-windows-virtual-machine-with-the-azure-cli"></a>Inicio rápido: Creación de una máquina virtual Windows con la CLI de Azure

La CLI de Azure se usa para crear y administrar recursos de Azure desde la línea de comandos o en scripts. En esta guía de inicio rápido se muestra como usar la CLI de Azure para implementar una máquina virtual en Azure que ejecute Windows Server 2016. Para ver la máquina virtual en acción, conéctese a la máquina virtual mediante RDP e instale al servidor web IIS.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/bash](https://shell.azure.com/bash) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione **Entrar** para ejecutarlos.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-virtual-machine"></a>Crear máquina virtual

Cree la máquina virtual con [az vm create](/cli/azure/vm). En el ejemplo siguiente se crea una máquina virtual denominada *myVM*. En este ejemplo se usa *azureuser* como nombre de un usuario administrativo. 

Necesitará establecer una contraseña que cumpla los [requisitos de contraseña de las máquinas virtuales de Azure](./faq.md#what-are-the-password-requirements-when-creating-a-vm
). Con el ejemplo siguiente, se le pedirá que escriba una contraseña en la línea de comandos. También puede agregar el parámetro `--admin-password` con un valor para la contraseña. El nombre de usuario y la contraseña se usarán más adelante cuando se conecte a la máquina virtual.

```azurecli-interactive
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image win2016datacenter \
    --admin-username azureuser 
```

La creación de la máquina virtual y los recursos auxiliares tarda unos minutos en realizarse. En la salida de ejemplo siguiente se muestra que la operación de creación de la máquina virtual se realizó correctamente.

```output
{
  "fqdns": "",
  "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.174.34.95",
  "resourceGroup": "myResourceGroup"
}
```

Tenga en cuenta sus propios valores de `publicIpAddress` en la salida de la máquina virtual. Esta dirección se usa para acceder a la máquina virtual en los siguientes pasos.

## <a name="open-port-80-for-web-traffic"></a>Apertura del puerto 80 para el tráfico web

De forma predeterminada, solo las conexiones de RDP se abren al crease una máquina virtual Windows en Azure. Use [az vm open-port](/cli/azure/vm) para abrir el puerto TCP 80 que se usará con el servidor web de IIS:

```azurecli-interactive
az vm open-port --port 80 --resource-group myResourceGroup --name myVM
```

## <a name="connect-to-virtual-machine"></a>Conexión a la máquina virtual

Use el comando siguiente para crear una sesión de Escritorio remoto desde el equipo local. Reemplace la dirección IP por la dirección IP pública de la máquina virtual. Cuando se le solicite, escriba las credenciales usadas al crear la máquina virtual:

```powershell
mstsc /v:publicIpAddress
```

## <a name="install-web-server"></a>Instalación del servidor web

Para ver la máquina virtual en acción, instale el servidor web IIS. Abra un símbolo del sistema de PowerShell en la máquina virtual y ejecute el siguiente comando:

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

Cuando haya terminado, cierre la conexión RDP con la máquina virtual.

## <a name="view-the-web-server-in-action"></a>Visualización del servidor web en acción

Con IIS instalado y el puerto 80 abierto en la máquina virtual desde Internet, use el explorador web que prefiera para ver la página principal predeterminada de IIS. Use la dirección IP pública de la máquina virtual que obtuvo en el paso anterior. En el ejemplo siguiente se muestra el sitio web de IIS predeterminado:

![Sitio predeterminado de IIS](./media/quick-create-powershell/default-iis-website.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede usar el comando [az group delete](/cli/azure/group) para eliminar el grupo de recursos, la máquina virtual y todos los recursos relacionados:

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha implementado una máquina virtual sencilla, ha abierto un puerto de red para el tráfico web y ha instalado un servidor web básico. Para más información acerca de las máquinas virtuales de Azure, continúe con el tutorial de máquinas virtuales Windows.

> [!div class="nextstepaction"]
> [Tutoriales de máquinas virtuales Windows de Azure](./tutorial-manage-vm.md)
