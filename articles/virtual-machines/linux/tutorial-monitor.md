---
title: 'Tutorial: Supervisión de máquinas virtuales Linux en Azure'
description: En este tutorial aprenderá a supervisar el rendimiento y los componentes de la aplicación detectados que se ejecutan en las máquinas virtuales Linux.
services: virtual-machines-linux
documentationcenter: virtual-machines
author: mgoedtel
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/30/2019
ms.author: magoedte
ms.custom: mvc
ms.openlocfilehash: c8d7b13f9e35a41a414a44c908997cfcc550af41
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89011742"
---
# <a name="tutorial-monitor-a-linux-virtual-machine-in-azure"></a>Tutorial: Supervisión de una máquina virtual Linux en Azure

La supervisión de Azure usa agentes para recopilar datos de arranque y de rendimiento de máquinas virtuales de Azure, almacenar estos datos en Azure Storage y permitir el acceso a ellos mediante el portal, el módulo de Azure PowerShell y la CLI de Azure. La supervisión avanzada se ofrece con Azure Monitor para VM mediante la recopilación de métricas de rendimiento, la detección de los componentes de aplicación instalados en la máquina virtual e incluye gráficos de rendimiento y asignaciones de dependencias.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Habilitar los diagnósticos de arranque en una máquina virtual
> * Ver los diagnósticos de arranque
> * Ver las métricas del host de la máquina virtual
> * Habilitar Azure Monitor para VM
> * Visualización de las métricas de rendimiento de la máquina virtual
> * Crear una alerta

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/powershell](https://shell.azure.com/powershell) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

Si decide instalar y usar la CLI localmente, en este tutorial es preciso que ejecute la CLI de Azure de la versión 2.0.30, u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="create-vm"></a>Creación de una máquina virtual

Para ver las métricas y los diagnósticos en acción, necesita una máquina virtual. En primer lugar, cree un grupo de recursos con [az group create](/cli/azure/group?view=azure-cli-latest#az-group-create). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroupMonitor* en la ubicación *eastus*.

```azurecli-interactive
az group create --name myResourceGroupMonitor --location eastus
```

Ahora cree una máquina virtual con el comando [az vm create](/cli/azure/vm?view=azure-cli-latest#az-vm-create). El siguiente ejemplo crea una máquina virtual llamada *myVM* y genera claves SSH, en caso de que no existan en *~/.ssh/* :

```azurecli-interactive
az vm create \
  --resource-group myResourceGroupMonitor \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

## <a name="enable-boot-diagnostics"></a>Habilitación de los diagnósticos de arranque

Cuando las máquinas virtuales Linux arrancan, la extensión de los diagnósticos de arranque captura la salida del arranque y la almacena en Azure Storage. Estos datos se pueden utilizar para solucionar los problemas de arranque de la máquina virtual. Los diagnósticos de arranque no se habilitan automáticamente al crear una máquina virtual Linux mediante la CLI de Azure.

Antes de habilitar los diagnósticos de arranque, es preciso crear una cuenta de almacenamiento para almacenar los registros de arranque. Las cuentas de almacenamiento deben tener un nombre único global, tener entre 3 y 24 caracteres, y deben contener solo números y letras en minúscula. Cree una cuenta de almacenamiento con el comando [az storage account create](/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-create). En este ejemplo, se utiliza una cadena aleatoria para crear un nombre de cuenta de almacenamiento único.

```azurecli-interactive
storageacct=mydiagdata$RANDOM

az storage account create \
  --resource-group myResourceGroupMonitor \
  --name $storageacct \
  --sku Standard_LRS \
  --location eastus
```

Al habilitar los diagnósticos de arranque, se necesita el identificador URI para el contenedor de Blob Storage. El siguiente comando de consulta la cuenta de almacenamiento a la que se devuelve este identificador URI. El valor del identificador URI se almacena en el nombre de variable *bloburi*, que se usa en el paso siguiente.

```azurecli-interactive
bloburi=$(az storage account show --resource-group myResourceGroupMonitor --name $storageacct --query 'primaryEndpoints.blob' -o tsv)
```

Habilite ahora los diagnósticos de arranque con [az vm boot-diagnostics enable](/cli/azure/vm/boot-diagnostics#az-vm-boot-diagnostics-enable). El valor `--storage` es el identificador URI del blob que se recopila en el paso anterior.

```azurecli-interactive
az vm boot-diagnostics enable \
  --resource-group myResourceGroupMonitor \
  --name myVM \
  --storage $bloburi
```

## <a name="view-boot-diagnostics"></a>Ver los diagnósticos de arranque

Cuando se habilitan los diagnósticos de arranque, cada vez que la máquina virtual se detiene y se inicia, la información acerca del proceso de arranque se escribe en un archivo de registro. En este ejemplo, en primer lugar desasigne la máquina virtual con el comando [az vm deallocate](/cli/azure/vm?view=azure-cli-latest#az-vm-deallocate) como se indica a continuación:

```azurecli-interactive
az vm deallocate --resource-group myResourceGroupMonitor --name myVM
```

Ahora inicie la máquina virtual con el comando [az vm start](/cli/azure/vm?view=azure-cli-latest#az-vm-start) como se indica a continuación:

```azurecli-interactive
az vm start --resource-group myResourceGroupMonitor --name myVM
```

Puede obtener los datos de los diagnósticos de arranque de *myVM* con el comando [az vm boot-diagnostics get-boot-log](/cli/azure/vm/boot-diagnostics#az-vm-boot-diagnostics-get-boot-log) como se indica a continuación:

```azurecli-interactive
az vm boot-diagnostics get-boot-log --resource-group myResourceGroupMonitor --name myVM
```

## <a name="view-host-metrics"></a>Visualización de las métricas del host

Una máquina virtual Linux tiene un host dedicado de Azure que interactúa con. Se recopilan automáticamente las métricas del host y se pueden ver en Azure Portal como se indica a continuación:

1. En Azure Portal, seleccione **Grupos de recursos**, elija **myResourceGroupMonitor** y seleccione **myVM** en la lista de recursos.
1. Para ver el funcionamiento de la máquina virtual host, seleccione **Métricas** en la ventana de la máquina virtual y, después, elija cualquiera de las métricas de *[Host]* en **Métricas disponibles**.

    ![Visualización de las métricas del host](./media/tutorial-monitoring/monitor-host-metrics.png)

## <a name="enable-advanced-monitoring"></a>Habilitación de la supervisión avanzada

Para habilitar la supervisión de la máquina virtual de Azure con Azure Monitor para VM:

1. En Azure Portal, haga clic en **Grupos de recursos**, seleccione **myResourceGroupMonitor** y, después, seleccione **myVM** en la lista de recursos.

2. En la página de la VM, en la sección **Supervisión**, seleccione **Insights (versión preliminar)** .

3. En la página **Insights (versión preliminar)** , seleccione **Probar ahora**.

    ![Habilitar Azure Monitor para VM para una máquina virtual](../../azure-monitor/insights/media/vminsights-enable-single-vm/enable-vminsights-vm-portal.png)

4. En la página **Azure Monitor Insights Onboarding** (Incorporación a Insights de Azure Monitor), si tiene un área de trabajo de Log Analytics existente en la misma suscripción, selecciónela en la lista desplegable.  

    La lista preselecciona el área de trabajo y la ubicación predeterminadas en las que se implementa la VM en la suscripción. 

    >[!NOTE]
    >Para crear un área de trabajo de Log Analytics nueva para almacenar los datos de supervisión de la VM, vea [Creación de un área de trabajo de Log Analytics en Azure Portal](../../azure-monitor/learn/quick-create-workspace.md). El área de trabajo debe pertenecer a una de las [regiones admitidas](../../azure-monitor/insights/vminsights-configure-workspace.md#supported-regions).

Después de habilitar la supervisión, puede que tenga que esperar unos minutos para ver las métricas de rendimiento de la máquina virtual.

![Habilitar el procesamiento de implementación de supervisión de Azure Monitor para VM](../../azure-monitor/insights/media/vminsights-enable-single-vm/onboard-vminsights-vm-portal-status.png)

## <a name="view-vm-performance-metrics"></a>Visualización de las métricas de rendimiento de la máquina virtual

Azure Monitor para VM incluye un conjunto de gráficos de rendimiento que tienen como destino varios indicadores clave de rendimiento (KPI) para ayudarle a determinar el rendimiento de una máquina virtual. Para acceder desde la máquina virtual, siga estos pasos.

1. En Azure Portal, haga clic en **Grupos de recursos**, seleccione **myResourceGroupMonitor** y, después, seleccione **myVM** en la lista de recursos.

2. En la página de la VM, en la sección **Supervisión**, seleccione **Insights (versión preliminar)** .

3. Seleccione la pestaña **Rendimiento**.

Esta página no solo incluye los gráficos de uso de rendimiento, sino también una tabla para cada disco lógico detectado, su capacidad, la utilización y la media total de cada medida.

## <a name="create-alerts"></a>Creación de alertas

Puede crear alertas basadas en métricas de rendimiento concretas. Se pueden usar alertas para enviar una notificación, por ejemplo, cuando el uso medio de la CPU supera un umbral concreto o cuando el espacio libre en disco disponible cae por debajo de una cantidad determinada. Las alertas se muestran en Azure Portal o se pueden enviar por correo electrónico. También puede desencadenar runbooks de Azure Automation o Azure Logic Apps en respuesta a las alertas que se generan.

En el siguiente ejemplo se crea una alerta para el uso medio de la CPU.

1. En Azure Portal, haga clic en **Grupos de recursos**, seleccione **myResourceGroupMonitor** y, después, seleccione **myVM** en la lista de recursos.

2. Haga clic en **Reglas de alertas** en la hoja de la máquina virtual y, después, en **Agregar alerta de métrica** en la parte superior de la hoja de alertas.

3. Especifique un **nombre** para la alerta, como *myAlertRule*.

4. Para desencadenar una alerta cuando el porcentaje de la CPU supera 1,0 durante cinco minutos, deje el resto de valores predeterminados seleccionados.

5. Opcionalmente, active la casilla *Enviar correo electrónico a propietarios, colaboradores y lectores* para enviar una notificación por correo electrónico. La acción predeterminada es presentar una notificación en el portal.

6. Haga clic en el botón **Aceptar** .

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha configurado y visto el rendimiento de la máquina virtual. Ha aprendido a:

> [!div class="checklist"]
> * Crear un grupo de recursos y una máquina virtual
> * Habilitar los diagnósticos de arranque en la máquina virtual
> * Ver los diagnósticos de arranque
> * Visualización de las métricas del host
> * Habilitar Azure Monitor para VM
> * Visualización de las métricas de la máquina virtual
> * Crear una alerta

Pase al siguiente tutorial para obtener información sobre Azure Security Center.

> [!div class="nextstepaction"]
> [Administración de la seguridad de máquinas virtuales](tutorial-azure-security.md)
