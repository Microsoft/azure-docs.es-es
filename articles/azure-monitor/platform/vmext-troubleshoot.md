---
title: Solución de problemas de la extensión de máquina virtual de Azure Log Analytics
description: Aquí se describen los síntomas, causas y soluciones de los problemas más comunes que surgen con la extensión de máquinas virtuales de Log Analytics para máquinas virtuales de Azure que se ejecutan en Windows y Linux.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 06/06/2019
ms.openlocfilehash: 873f6beaa88e1631397827a94161ce4427b5f0bb
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323390"
---
# <a name="troubleshooting-the-log-analytics-vm-extension-in-azure-monitor"></a>Solución de problemas de la extensión de máquina virtual de Log Analytics en Azure Monitor
En este artículo se proporciona ayuda y posibles soluciones para resolver los errores que puedan surgir con la extensión de máquinas virtuales de Log Analytics para aquellas máquinas virtuales que se ejecutan en Microsoft Azure para Windows y Linux.

Para comprobar el estado de la extensión, realice los pasos siguientes en Azure Portal.

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com).
2. En Azure Portal, haga clic en **Todos los servicios**. En la lista de recursos, escriba **Máquinas virtuales**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Máquinas virtuales**.
3. En la lista de máquinas virtuales, busque y seleccione la que le interesa.
3. En la máquina virtual, haga clic en **Extensiones**.
4. En la lista, compruebe si la extensión de Log Analytics está habilitada.  Para Linux, el agente aparece como **OMSAgentforLinux** y para Windows, el agente aparece como **MicrosoftMonitoringAgent**.

   ![Vista de la extensión de máquina virtual](./media/vmext-troubleshoot/log-analytics-vmview-extensions.png)

4. Haga clic en la extensión para ver los detalles. 

   ![Detalles de la extensión de máquina virtual](./media/vmext-troubleshoot/log-analytics-vmview-extensiondetails.png)

## <a name="troubleshooting-azure-windows-vm-extension"></a>Solución de problemas con la extensión de máquinas virtuales de Azure en Windows.

Si la extensión de la máquina virtual *Microsoft Monitoring Agent* no se instala o no envía informes, puede realizar los pasos siguientes para solucionar el problema.

1. Siga los pasos de [KB 2965986](https://support.microsoft.com/kb/2965986#mt1) para comprobar que el agente de máquina virtual de Azure está instalado y funciona correctamente.
   * También puede revisar el archivo de registro del agente de máquina virtual `C:\WindowsAzure\logs\WaAppAgent.log`
   * Si el registro no existe, el agente de máquina virtual no estará instalado.
   * [Instale el agente de máquina virtual de Azure.](../learn/quick-collect-azurevm.md#enable-the-log-analytics-vm-extension)
2. Revise los archivos de registro de la extensión de máquina virtual de Microsoft Monitoring Agent en `C:\Packages\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent`.
3. Asegúrese de que la máquina virtual puede ejecutar scripts de PowerShell.
4. Asegúrese de que no se hayan modificado los permisos en C:\Windows\temp.
5. Consulte el estado de Microsoft Monitoring Agent, para lo que debe escribir lo siguiente en una ventana de PowerShell con privilegios elevados en la máquina virtual `(New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg').GetCloudWorkspaces() | Format-List`.
6. Revise los archivos de registro de instalación de Microsoft Monitoring Agent en `C:\Windows\System32\config\systemprofile\AppData\Local\SCOM\Logs`.

Para más información, consulte [Solución de problemas de la extensión de máquina virtual de Microsoft Azure](../../virtual-machines/extensions/oms-windows.md).

## <a name="troubleshooting-linux-vm-extension"></a>Solución de problemas con la extensión de máquinas virtuales en Linux
[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)] 
Si la extensión de la máquina virtual del *agente de Log Analytics para Linux* no se instala o no envía informes, puede realizar los pasos siguientes para solucionar el problema.

1. Si el estado de la extensión es *desconocido*, compruebe si el agente de máquina virtual de Azure está instalado y funciona correctamente revisando el archivo de registro del agente de máquina virtual `/var/log/waagent.log`.
   * Si el registro no existe, el agente de máquina virtual no estará instalado.
   * [Instale el agente de máquina virtual de Azure en máquinas virtuales Linux.](../learn/quick-collect-azurevm.md#enable-the-log-analytics-vm-extension)
2. Para otros estados incorrectos, revise los archivos de registro de extensión de máquina virtual del agente de Log Analytics para Linux en `/var/log/azure/Microsoft.EnterpriseCloud.Monitoring.OmsAgentForLinux/*/extension.log` y `/var/log/azure/Microsoft.EnterpriseCloud.Monitoring.OmsAgentForLinux/*/CommandExecution.log`.
3. Si el estado de la extensión es correcto, pero no se están cargando datos, revise los archivos de registro del agente de Log Analytics para Linux en `/var/opt/microsoft/omsagent/log/omsagent.log`.

Para más información, consulte [Solución de problemas de la extensión de máquina virtual de Linux Azure](../../virtual-machines/extensions/oms-linux.md).

## <a name="next-steps"></a>Pasos siguientes

Para obtener instrucciones adicionales para solucionar problemas relacionados con el agente de Log Analytics para Linux que se hospeda en equipos externos a Azure, vea [Troubleshoot Azure Log Analytics Linux Agent](agent-linux-troubleshoot.md) (Solución de problemas del agente de Linux para Azure Log Analytics).  

