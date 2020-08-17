---
title: 'Tutorial: Supervisión de máquinas virtuales Windows en Azure'
description: En este tutorial aprenderá a supervisar el rendimiento y los componentes de la aplicación detectados que se ejecutan en las máquinas virtuales Windows.
author: mgoedtel
manager: carmonm
ms.service: virtual-machines-windows
ms.subservice: monitoring
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 09/27/2018
ms.author: magoedte
ms.custom: mvc
ms.openlocfilehash: 154e4c9421d1c0a54b3d9b5f53424e7640fe62d8
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323577"
---
# <a name="tutorial-monitor-a-windows-virtual-machine-in-azure"></a>Tutorial: Supervisión de una máquina virtual Windows en Azure

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

## <a name="create-virtual-machine"></a>Crear máquina virtual

Para configurar la supervisión y la administración de actualizaciones de Azure en este tutorial, necesita una máquina virtual Windows en Azure. En primer lugar, establezca un nombre de usuario de administrador y una contraseña para la máquina virtual con [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1):

```azurepowershell-interactive
$cred = Get-Credential
```

Ahora cree la máquina virtual con [New-AzVM](/powershell/module/az.compute/new-azvm). En el ejemplo siguiente se crea una máquina virtual denominada *myVM* en la ubicación *EastUS*. Si aún no existen, se crean el grupo de recursos *myResourceGroupMonitorMonitor* y los recursos de red auxiliares.

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroupMonitor" `
    -Name "myVM" `
    -Location "East US" `
    -Credential $cred
```

Los recursos y la máquina virtual tardan unos minutos en crearse.

## <a name="view-boot-diagnostics"></a>Ver los diagnósticos de arranque

Cuando las máquinas virtuales Windows arrancan, el agente de diagnóstico de arranque captura una salida de pantalla que se puede usar para solucionar problemas. Esta funcionalidad está habilitada de forma predeterminada. Las capturas de pantalla se almacenan en una cuenta de Azure Storage, que también se crea de forma predeterminada.

Puede obtener los datos de diagnóstico de arranque con el comando [Get-AzureRmVMBootDiagnosticsData](/powershell/module/az.compute/get-azvmbootdiagnosticsdata). En el ejemplo siguiente, el diagnóstico de arranque se descarga en la raíz de la unidad *c:\*.

```powershell
Get-AzVMBootDiagnosticsData -ResourceGroupName "myResourceGroupMonitor" -Name "myVM" -Windows -LocalPath "c:\"
```

## <a name="view-host-metrics"></a>Visualización de las métricas del host

Una máquina virtual Windows tiene una máquina virtual host dedicada en Azure con la que interactúa. Las métricas del host se recopilan automáticamente y se pueden ver en Azure Portal.

1. En Azure Portal, haga clic en **Grupos de recursos**, seleccione **myResourceGroupMonitor** y, después, seleccione **myVM** en la lista de recursos.
2. Haga clic en **Métricas** en la hoja de la máquina virtual y seleccione cualquiera de las métricas del host en **Métricas disponibles** para ver el funcionamiento de la máquina virtual host.

    ![Visualización de las métricas del host](./media/tutorial-monitoring/tutorial-monitor-host-metrics.png)

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
> [Administración de la seguridad de máquinas virtuales](../../security/fundamentals/overview.md)
