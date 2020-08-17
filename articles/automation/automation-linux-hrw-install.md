---
title: Implementación de una instancia de Hybrid Runbook Worker en Azure Automation
description: En este artículo se describe cómo instalar Hybrid Runbook Worker de Azure Automation para ejecutar runbooks en máquinas con Linux en su centro de datos local o en su entorno en la nube.
services: automation
ms.subservice: process-automation
ms.date: 06/24/2020
ms.topic: conceptual
ms.openlocfilehash: 7f19aec65ed2616d757718116ac948473dd4b0ed
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87448017"
---
# <a name="deploy-a-linux-hybrid-runbook-worker"></a>Implementación de Hybrid Runbook Worker en Linux

La característica Hybrid Runbook Worker de Azure Automation permite ejecutar runbooks directamente en la máquina que hospeda el rol y en los recursos del entorno para administrar dichos recursos locales. Hybrid Runbook Worker en Linux ejecuta runbooks como un usuario especial que puede tener permisos elevados para ejecutar comandos que necesitan permisos elevados. Azure Automation almacena y administra los runbooks y, después, los entrega a una o más máquinas designadas. En este artículo se describe cómo instalar la instancia de Hybrid Runbook Worker en una máquina Linux, cómo eliminar el trabajo y cómo eliminar un grupo de Hybrid Runbook Worker.

Después de implementar correctamente un trabajo de runbook, revise la [ejecución de runbooks en Hybrid Runbook Worker](automation-hrw-run-runbooks.md) para más información sobre cómo configurar los runbooks para automatizar los procesos del centro de datos local o en otro entorno de nube.

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar, asegúrese de que dispone de lo siguiente:

### <a name="a-log-analytics-workspace"></a>Un área de trabajo de Log Analytics

El rol Hybrid Runbook Worker depende de un área de trabajo de Log Analytics de Azure Monitor para instalar y configurar el rol. Puede usar [Azure Resource Manager](../azure-monitor/samples/resource-manager-workspace.md#create-a-log-analytics-workspace), mediante [PowerShell](../azure-monitor/scripts/powershell-sample-create-workspace.md?toc=/powershell/module/toc.json) o [Azure Portal](../azure-monitor/learn/quick-create-workspace.md), para crearla.

Si no tiene ningún área de trabajo de Log Analytics de Azure Monitor, revise la [guía de diseño de registros de Azure Monitor](../azure-monitor/platform/design-logs-deployment.md) antes de crear el área de trabajo.

Si tiene un área de trabajo, pero no está vinculada a su cuenta de Automation, la habilitación de una característica de Automation agrega funcionalidad para Azure Automation, incluida la compatibilidad con Hybrid Runbook Worker. Cuando habilita una de las características de Azure Automation en el área de trabajo de Log Analytics, en concreto, en [Update Management](update-management/update-mgmt-overview.md) o [Change Tracking e Inventario](change-tracking.md), los componentes de trabajo se insertan automáticamente en la máquina del agente.

Para agregar la característica Administración de actualizaciones al área de trabajo, ejecute el cmdlet de PowerShell siguiente:

```powershell-interactive
    Set-AzOperationalInsightsIntelligencePack -ResourceGroupName <logAnalyticsResourceGroup> -WorkspaceName <logAnalyticsWorkspaceName> -IntelligencePackName "Updates" -Enabled $true
```

Para agregar la característica Change Tracking e Inventario al área de trabajo, ejecute el cmdlet de PowerShell siguiente:

```powershell-interactive
    Set-AzOperationalInsightsIntelligencePack -ResourceGroupName <logAnalyticsResourceGroup> -WorkspaceName <logAnalyticsWorkspaceName> -IntelligencePackName "ChangeTracking" -Enabled $true
```

### <a name="log-analytics-agent"></a>Agente de Log Analytics

El rol de Hybrid Runbook Worker requiere el [agente de Log Analytics](../azure-monitor/platform/log-analytics-agent.md) para el sistema operativo Linux compatible.

### <a name="supported-linux-operating-systems"></a>Sistemas operativos Linux compatibles

La característica Hybrid Runbook Worker admite las siguientes distribuciones:

* Amazon Linux 2012.09 a 2015.09 (x86/x64)
* CentOS Linux 5, 6 y 7 (x86/x64)
* Oracle Linux 5, 6 y 7 (x86/x64)
* Red Hat Enterprise Linux Server 5, 6 y 7 (x86/x64)
* Debian GNU/Linux 6, 7 y 8 (x86/x64)
* Ubuntu 12.04 LTS, 14.04 LTS, 16.04 LTS y 18.04 (x86/x64)
* SUSE Linux Enterprise Server 11 y 12 (x86/x64)

### <a name="minimum-requirements"></a>Requisitos mínimos

Estos son los requisitos mínimos de Hybrid Runbook Worker en Linux:

* Dos núcleos
* 4 GB de RAM
* Puerto 443 (saliente)

| **Paquetes necesarios** | **Descripción** | **Versión mínima**|
|--------------------- | --------------------- | -------------------|
|Glibc |Biblioteca GNU C| 2.5-12 |
|Openssl| Bibliotecas OpenSSL | 1.0 (se admiten TLS 1.1 y TLS 1.2)|
|Curl | Cliente web de cURL | 7.15.5|
|Python ctypes | Python 2.x es obligatorio |
|PAM | Módulos de autenticación conectables|
| **Paquete opcional** | **Descripción** | **Versión mínima**|
| PowerShell Core | Para ejecutar runbooks de PowerShell, es necesario instalar PowerShell Core. Consulte [Instalación de PowerShell Core en Linux](/powershell/scripting/install/installing-powershell-core-on-linux) para obtener información sobre cómo instalarlo. | 6.0.0 |

## <a name="supported-runbook-types"></a>Tipos de runbook admitidos

Las instancias de Hybrid Runbook Worker admiten un conjunto limitado de tipos de runbook en Azure Automation y se describen en la tabla siguiente.

|Tipo de runbook | Compatible |
|-------------|-----------|
|Python 2 |Sí |
|PowerShell |Sí<sup>1</sup> |
|Flujo de trabajo de PowerShell |No |
|Gráfico |No |
|Flujo de trabajo gráfico de PowerShell |No |

<sup>1</sup>Los runbooks de PowerShell requieren que PowerShell Core esté instalado en la máquina con Linux. Consulte [Instalación de PowerShell Core en Linux](/powershell/scripting/install/installing-powershell-core-on-linux) para obtener información sobre cómo instalarlo.

## <a name="install-a-linux-hybrid-runbook-worker"></a>Instalación de Hybrid Runbook Worker en Linux

Para instalar y configurar una instancia de Hybrid Runbook Worker para Linux, siga los pasos que se indican a continuación.

1. Implemente el agente de Log Analytics en la máquina de destino.

    * En el caso de las VM de Azure, instale el agente de Log Analytics para Linux mediante la [extensión de máquina virtual para Linux](../virtual-machines/extensions/oms-linux.md). La extensión instala el agente de Log Analytics en Azure Virtual Machines e inscribe las máquinas virtuales en un área de trabajo de Log Analytics existente mediante una plantilla de Azure Resource Manager o la CLI de Azure. Una vez instalado el agente, la máquina virtual se puede agregar a un grupo de Hybrid Runbook Worker de su cuenta de Automation.

    * En el caso de VM que no sean de Azure, instale el agente de Log Analytics para Linux con las opciones de implementación que se describen en el artículo [Conexión de equipos Linux a Azure Monitor](../azure-monitor/platform/agent-linux.md). Puede repetir este proceso para varias máquinas para agregar varios trabajos a su entorno. Una vez instalado el agente, las VM se pueden agregar a un grupo de Hybrid Runbook Worker de su cuenta de Automation.

    > [!NOTE]
    > Para administrar la configuración de las máquinas que admiten el rol de Hybrid Runbook Worker con Desired State Configuration (DSC), debe agregar dichas máquinas como nodos de DSC.

    > [!NOTE]
    > La [cuenta nxautomation](automation-runbook-execution.md#log-analytics-agent-for-linux) con los permisos sudo correspondientes debe estar presente durante la instalación de la instancia de Hybrid Worker en Linux. Si intenta instalar el rol de trabajo y la cuenta no está presente o no tiene los permisos adecuados, no se realizará la instalación.

2. Compruebe que el agente informa al área de trabajo.

    El agente de Log Analytics para Linux conecta las máquinas a un área de trabajo de Log Analytics de Azure Monitor. Cuando instala el agente en la máquina y lo conecta al área de trabajo, se descargan automáticamente los componentes necesarios para la instancia de Hybrid Runbook Worker.

    Una vez que el agente se ha conectado correctamente al área de trabajo de Log Analytics, transcurridos unos minutos, puede ejecutar la consulta siguiente para comprobar que envía datos de latido al área de trabajo.

    ```kusto
    Heartbeat 
    | where Category == "Direct Agent"
    | where TimeGenerated > ago(30m)
    ```

    En los resultados de la búsqueda, verá los registros de latido de la máquina que indican que está conectada y que está creando informes para el servicio. De forma predeterminada, cada agente reenvía un registro de latido a su área de trabajo asignada.

3. Ejecute el siguiente comando para agregar la máquina a un grupo de Hybrid Runbook Worker y especifique los valores para los parámetros `-w`, `-k`, `-g` y `-e`.

    Puede obtener la información necesaria para los parámetros `-k` y `-e` de la página **Claves** de la cuenta de Automation. Seleccione **Claves** en la sección **Configuración de la cuenta** en el lado izquierdo de la página.

    ![Página Administrar claves](media/automation-hybrid-runbook-worker/elements-panel-keys.png)

    * Para el parámetro `-e`, copie el valor para **URL**.

    * Para el parámetro `-k`, copie el valor para **CLAVE DE ACCESO PRIMARIA**.

    * Para el parámetro `-g`, especifique el nombre del grupo de Hybrid Runbook Worker al que se debe unir la nueva instancia de Hybrid Runbook Worker para Linux. Si este grupo ya existe en la cuenta de Automation, se le agregará la máquina actual. Si no existe, se crea con ese nombre.

    * Para el parámetro `-w`, especifique el identificador del área de trabajo de Log Analytics.

   ```bash
   sudo python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/scripts/onboarding.py --register -w <logAnalyticsworkspaceId> -k <automationSharedKey> -g <hybridGroupName> -e <automationEndpoint>
   ```

4. Una vez completado el comando, la página Grupos de Hybrid Worker de la cuenta de Automation muestra el nuevo grupo y el número de miembros. Si se trata de un grupo existente, se incrementa el número de miembros. Puede seleccionar el grupo de la lista en la página Grupos de Hybrid Worker y seleccionar el icono **Hybrid Workers**. En la página Hybrid Workers, verá que aparece cada miembro del grupo.

    > [!NOTE]
    > Si usa la extensión de máquina virtual de Log Analytics para una VM de Azure para Linux, se recomienda establecer `autoUpgradeMinorVersion` en `false`, ya que las versiones de actualización automática pueden provocar problemas con Hybrid Runbook Worker. Para obtener información sobre cómo actualizar manualmente la extensión, vea [Implementación de la CLI de Azure](../virtual-machines/extensions/oms-linux.md#azure-cli-deployment).

## <a name="turn-off-signature-validation"></a>Desactivación de la validación de firma

De forma predeterminada, las instancias de Hybrid Runbook Worker de Linux requieren la validación de la firma. Si ejecuta un runbook sin firmar en un trabajo, verá un error `Signature validation failed`. Para desactivar la validación de la firma, ejecute el siguiente comando. Reemplace el segundo parámetro por el identificador de área de trabajo de Log Analytics.

 ```bash
 sudo python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/scripts/require_runbook_signature.py --false <logAnalyticsworkspaceId>
 ```

## <a name="remove-the-hybrid-runbook-worker-from-an-on-premises-linux-machine"></a><a name="remove-linux-hybrid-runbook-worker"></a>Eliminación de la instancia de Hybrid Runbook Worker de una máquina con Linux local

Puede usar el comando `ls /var/opt/microsoft/omsagent` en Hybrid Runbook Worker para obtener el identificador del área de trabajo. Se creará una carpeta cuyo nombre será el identificador del área de trabajo.

```bash
sudo python onboarding.py --deregister --endpoint="<URL>" --key="<PrimaryAccessKey>" --groupname="Example" --workspaceid="<workspaceId>"
```

> [!NOTE]
> Este script no quita el agente de Log Analytics para Linux de la máquina. Solo quita la funcionalidad y la configuración del rol de Hybrid Runbook Worker.

## <a name="remove-a-hybrid-worker-group"></a>Eliminación de un grupo de Hybrid Worker

Para eliminar un grupo de Hybrid Runbook Worker de máquinas con Linux, use los mismos pasos que para un grupo de Hybrid Worker de Windows. Consulte [Eliminación de un grupo de Hybrid Worker](automation-windows-hrw-install.md#remove-a-hybrid-worker-group).

## <a name="next-steps"></a>Pasos siguientes

* Consulte [Ejecución de runbooks en Hybrid Runbook Worker](automation-hrw-run-runbooks.md) para más información sobre cómo configurar los runbooks para automatizar los procesos del centro de datos local o de otros entornos de nube.

* Para aprender a solucionar problemas de las instancias de Hybrid Runbook Worker, consulte [Solución de problemas de Hybrid Runbook Worker: Linux](troubleshoot/hybrid-runbook-worker.md#linux).
