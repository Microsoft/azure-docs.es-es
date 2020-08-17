---
title: 'Inicio rápido: Uso de la CLI de Azure para crear una máquina virtual Linux'
description: En esta guía de inicio rápido, aprenderá a utilizar la CLI de Azure para crear una máquina virtual Linux
author: cynthn
ms.service: virtual-machines-linux
ms.topic: quickstart
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 10/09/2018
ms.author: cynthn
ms.custom:
- mvc
- seo-javascript-september2019
- seo-javascript-october2019
- seo-python-october2019
- devx-track-azurecli
ms.openlocfilehash: 78ecd052c071835ef488882979a3bc5306c35365
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87488983"
---
# <a name="quickstart-create-a-linux-virtual-machine-with-the-azure-cli"></a>Inicio rápido: Creación de una máquina virtual Linux con la CLI de Azure

En esta guía de inicio rápido se muestra como usar la interfaz de la línea de comandos (CLI) de Azure para implementar una máquina virtual (VM) Linux en Azure. La CLI de Azure se usa para crear y administrar recursos de Azure desde la línea de comandos o en scripts.

En este tutorial, se instalará Ubuntu 16.04 LTS. Para mostrar la máquina virtual en acción, se conectará a ella mediante SSH e instalará al servidor web de NGINX.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede abrir Cloud Shell en una pestaña independiente acudiendo a [https://shell.azure.com/bash](https://shell.azure.com/bash). Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, después, seleccione **Entrar** para ejecutarlos.

Si prefiere instalar y usar la CLI en un entorno local, para esta guía de inicio rápido se requiere la versión 2.0.30 de la CLI de Azure o una versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-virtual-machine"></a>Crear máquina virtual

Cree la máquina virtual con el comando [az vm create](/cli/azure/vm).

En el ejemplo siguiente se crea una máquina virtual llamada *myVM* y se agrega una cuenta de usuario llamada *azureuser*. El parámetro `--generate-ssh-keys` se usa para generar automáticamente una clave SSH y colocarla en la ubicación de la clave predeterminada ( *~/.ssh*). Para utilizar un conjunto de claves concreto, utilice la opción `--ssh-key-value`.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
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
  "publicIpAddress": "40.68.254.142",
  "resourceGroup": "myResourceGroup"
}
```

Tenga en cuenta sus propios valores de `publicIpAddress` en la salida de la máquina virtual. Esta dirección se usa para acceder a la máquina virtual en los siguientes pasos.

## <a name="open-port-80-for-web-traffic"></a>Apertura del puerto 80 para el tráfico web

De forma predeterminada, solo las conexiones de SSH se abren cuando se crea una máquina virtual Linux en Azure. Use [az vm open-port](/cli/azure/vm) para abrir el puerto TCP 80 que se usará con el servidor web de NGINX:

```azurecli-interactive
az vm open-port --port 80 --resource-group myResourceGroup --name myVM
```

## <a name="connect-to-virtual-machine"></a>Conexión a la máquina virtual

Conéctese a la máquina virtual mediante SSH, como hace habitualmente. Reemplace **publicIpAddress** por la dirección IP pública de la máquina virtual como se indica en la salida anterior de la máquina virtual:

```bash
ssh azureuser@publicIpAddress
```

## <a name="install-web-server"></a>Instalación del servidor web

Para ver la máquina virtual en acción, instale al servidor de web de NGINX. Actualice los orígenes de paquete y, después, instale el paquete NGINX más reciente.

```bash
sudo apt-get -y update
sudo apt-get -y install nginx
```

Cuando haya terminado, escriba `exit` para salir de la sesión de SSH.

## <a name="view-the-web-server-in-action"></a>Visualización del servidor web en acción

Use un explorador web de su elección para ver la página de bienvenida predeterminada de NGINX. Use la dirección IP pública de la máquina virtual como dirección web. En el ejemplo siguiente se muestra el sitio web de NGINX predeterminado:

![Visualización de la página de bienvenida de NGINX](./media/quick-create-cli/view-the-nginx-welcome-page.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede usar el comando [az group delete](/cli/azure/group) para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados. 

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha implementado una máquina virtual sencilla, ha abierto un puerto de red para el tráfico web y ha instalado un servidor web básico. Para más información acerca de las máquinas virtuales de Azure, continúe con el tutorial de máquinas virtuales Linux.


> [!div class="nextstepaction"]
> [Tutoriales de máquinas virtuales Linux de Azure](./tutorial-manage-vm.md)
