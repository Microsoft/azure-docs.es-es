---
title: Solucionar problemas relativos a errores con Update Management
description: Obtenga información acerca de la solución de problemas relacionados con Update Management.
services: automation
author: georgewallace
ms.author: gwallace
ms.date: 05/07/2019
ms.topic: conceptual
ms.service: automation
manager: carmonm
ms.openlocfilehash: f286877c6a9e787c06a8a846efaf94668c04fc4e
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65787705"
---
# <a name="troubleshooting-issues-with-update-management"></a>Solución de problemas relacionados con Update Management

En este artículo se describen soluciones para resolver problemas que pueden surgir al usar Update Management.

Existe un solucionador de problemas de agente para que el agente de Hybrid Worker determine el problema subyacente. Para más información sobre el solucionador de problemas, consulte el artículo sobre [cómo solucionar problemas con el agente de actualización](update-agent-issues.md). Para todos los demás problemas, consulte la información detallada que aparece a continuación sobre posibles problemas.

## <a name="general"></a>General

### <a name="components-enabled-not-working"></a>Escenario: Se han habilitado los componentes de la solución "Update Management" y ahora se está configurando esta máquina virtual

#### <a name="issue"></a>Problema

Continúa recibiendo el mensaje siguiente en una máquina virtual 15 minutos después de la incorporación:

```error
The components for the 'Update Management' solution have been enabled, and now this virtual machine is being configured. Please be patient, as this can sometimes take up to 15 minutes.
```

#### <a name="cause"></a>Causa

Este error puede deberse a las siguientes razones:

1. Está bloqueada la comunicación a la cuenta de Automation.
2. La máquina virtual que se incorpora puede provenir de una máquina clonada que no estaba preparada con sysprep con Microsoft Monitoring Agent instalado.

#### <a name="resolution"></a>Resolución

1. Visite [Planeamiento de red](../automation-hybrid-runbook-worker.md#network-planning) para obtener información acerca de qué direcciones y puertos deben permitirse para que Update Management funcione.
2. Si usa una imagen clonada:
   1. En el área de trabajo de Log Analytics, quite la máquina virtual de la búsqueda guardada con la configuración de ámbito `MicrosoftDefaultScopeConfig-Updates` si se muestra. Las búsquedas guardadas se pueden encontrar en la sección **General** del área de trabajo.
   2. Ejecute `Remove-Item -Path "HKLM:\software\microsoft\hybridrunbookworker" -Recurse -Force`
   3. Ejecute `Restart-Service HealthService` para reiniciar `HealthService`. Se volverá a crear la clave y se generará un nuevo UUID.
   4. Si esto no funciona, la imagen sysprep primero e instalar el agente MMA después del hecho.

### <a name="multi-tenant"></a>Escenario: Recibe un error de la suscripción vinculada al crear una implementación de actualización para las máquinas en otro inquilino de Azure.

#### <a name="issue"></a>Problema

Recibe el error siguiente al intentar crear una implementación de actualización para las máquinas en otro inquilino de Azure:

```error
The client has permission to perform action 'Microsoft.Compute/virtualMachines/write' on scope '/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/resourceGroupName/providers/Microsoft.Automation/automationAccounts/automationAccountName/softwareUpdateConfigurations/updateDeploymentName', however the current tenant '00000000-0000-0000-0000-000000000000' is not authorized to access linked subscription '00000000-0000-0000-0000-000000000000'.
```

#### <a name="cause"></a>Causa

Este error se produce cuando se crea una implementación de actualización que tiene máquinas virtuales de Azure en otro inquilino incluidas en una implementación de actualización.

#### <a name="resolution"></a>Resolución

Tendrá que usar la solución alternativa siguiente para programarlas. Puede usar el cmdlet [New-AzureRmAutomationSchedule](/powershell/module/azurerm.automation/new-azurermautomationschedule) con el modificador `-ForUpdate` para crear una programación y usar el cmdlet [New AzureRmAutomationSoftwareUpdateConfiguration](/powershell/module/azurerm.automation/new-azurermautomationsoftwareupdateconfiguration
) y pasar las máquinas del otro inquilino al parámetro `-NonAzureComputer`. En el ejemplo siguiente se muestra cómo hacerlo:

```azurepowershell-interactive
$nonAzurecomputers = @("server-01", "server-02")

$startTime = ([DateTime]::Now).AddMinutes(10)

$s = New-AzureRmAutomationSchedule -ResourceGroupName mygroup -AutomationAccountName myaccount -Name myupdateconfig -Description test-OneTime -OneTime -StartTime $startTime -ForUpdate

New-AzureRmAutomationSoftwareUpdateConfiguration  -ResourceGroupName $rg -AutomationAccountName $aa -Schedule $s -Windows -AzureVMResourceId $azureVMIdsW -NonAzureComputer $nonAzurecomputers -Duration (New-TimeSpan -Hours 2) -IncludedUpdateClassification Security,UpdateRollup -ExcludedKbNumber KB01,KB02 -IncludedKbNumber KB100
```

### <a name="nologs"></a>Escenario: Actualizar los datos de administración no se muestran en los registros de Azure Monitor para una máquina

#### <a name="issue"></a>Problema

Tiene máquinas que se muestran como **no evaluado** en **cumplimiento**, y ver datos de latido en los registros de Azure Monitor para el Hybrid Runbook Worker, pero no la administración de actualizaciones.

#### <a name="cause"></a>Causa

Es posible que sea necesario volver a registrar e instalar Hybrid Runbook Worker.

#### <a name="resolution"></a>Resolución

Siga los pasos descritos en [Implementación de Hybrid Runbook Worker en Windows](../automation-windows-hrw-install.md) para volver a instalar Hybrid Worker para Windows o los de [Implementación de Hybrid Runbook Worker en Linux](../automation-linux-hrw-install.md) para Linux.

## <a name="windows"></a> Windows

Si se producen problemas al intentar incorporar la solución en una máquina virtual, compruebe el registro de eventos de **Operations Manager** en **Registros de aplicaciones y servicios** en la máquina local en busca de eventos con el identificador de evento **4502** y el mensaje de evento que contenga **Microsoft.EnterpriseManagement.HealthService.AzureAutomation.HybridAgent**.

En la sección siguiente se destacan los mensajes de error específicos y una posible solución para cada uno. Para otros problemas sobre la incorporación, consulte el artículo sobre la [solución de problemas de incorporación de la solución](onboarding.md).

### <a name="machine-already-registered"></a>Escenario: La máquina ya está registrada en otra cuenta

#### <a name="issue"></a>Problema

Aparece el siguiente mensaje de error:

```error
Unable to Register Machine for Patch Management, Registration Failed with Exception System.InvalidOperationException: {"Message":"Machine is already registered to a different account."}
```

#### <a name="cause"></a>Causa

La máquina ya está incorporada a otra área de trabajo para Update Management.

#### <a name="resolution"></a>Resolución

Realice una limpieza de los artefactos antiguos en la máquina mediante la [eliminación del grupo Hybrid Runbook](../automation-hybrid-runbook-worker.md#remove-a-hybrid-worker-group) y vuelva a intentarlo.

### <a name="machine-unable-to-communicate"></a>Escenario: La máquina no se puede comunicar con el servicio

#### <a name="issue"></a>Problema

Aparece uno de los siguientes mensajes de error:

```error
Unable to Register Machine for Patch Management, Registration Failed with Exception System.Net.Http.HttpRequestException: An error occurred while sending the request. ---> System.Net.WebException: The underlying connection was closed: An unexpected error occurred on a receive. ---> System.ComponentModel.Win32Exception: The client and server can't communicate, because they do not possess a common algorithm
```

```error
Unable to Register Machine for Patch Management, Registration Failed with Exception Newtonsoft.Json.JsonReaderException: Error parsing positive infinity value.
```

```error
The certificate presented by the service <wsid>.oms.opinsights.azure.com was not issued by a certificate authority used for Microsoft services. Contact your network administrator to see if they are running a proxy that intercepts TLS/SSL communication.
```

#### <a name="cause"></a>Causa

Puede haber un servidor proxy, una puerta de enlace o un firewall bloqueando la comunicación de red.

#### <a name="resolution"></a>Resolución

Revise la red y asegúrese de que se permiten las direcciones y los puertos correspondientes. Consulte los [requisitos de red](../automation-hybrid-runbook-worker.md#network-planning), para obtener una lista de puertos y direcciones que necesitan Update Management y las instancias de Hybrid Runbook Worker.

### <a name="unable-to-create-selfsigned-cert"></a>Escenario: Error al crear el certificado autofirmado

#### <a name="issue"></a>Problema

Aparece uno de los siguientes mensajes de error:

```error
Unable to Register Machine for Patch Management, Registration Failed with Exception AgentService.HybridRegistration. PowerShell.Certificates.CertificateCreationException: Failed to create a self-signed certificate. ---> System.UnauthorizedAccessException: Access is denied.
```

#### <a name="cause"></a>Causa

Hybrid Runbook Worker no pudo generar un certificado autofirmado

#### <a name="resolution"></a>Resolución

Verifique que la cuenta del sistema tiene acceso de lectura a la carpeta **C:\ProgramData\Microsoft\Crypto\RSA** e inténtelo de nuevo.

### <a name="failed-to-start"></a>Escenario: Muestra de una máquina no se pudo iniciar en una implementación de actualizaciones

#### <a name="issue"></a>Problema

Una máquina tiene el estado **no se pudo iniciar** para una máquina. Al ver los detalles específicos de la máquina, verá el siguiente error:

```error
Failed to start the runbook. Check the parameters passed. RunbookName Patch-MicrosoftOMSComputer. Exception You have requested to create a runbook job on a hybrid worker group that does not exist.
```

#### <a name="cause"></a>Causa

Este error puede producirse debido a uno de los siguientes motivos:

* Ya no existe la máquina.
* La máquina está desactivada off e inaccesible.
* La máquina tiene un problema de conectividad de red y hybrid worker en la máquina está inaccesible.
* Se ha producido una actualización de Microsoft Monitoring Agent que ha cambiado el SourceComputerId
* La ejecución de actualización puede haber limitado si se alcanza el límite de 2.000 trabajos simultáneos en una cuenta de Automation. Cada implementación se considera un trabajo y cada máquina en un recuento de implementación de actualización como un trabajo. Cualquier otra automatización trabajo o actualización de implementación en ejecución actualmente en el número de cuenta de Automation para el límite de trabajos simultáneos.

#### <a name="resolution"></a>Resolución

Cuando uso aplicables [grupos dinámicos](../automation-update-management.md#using-dynamic-groups) para las implementaciones de actualización.

* Compruebe la máquina aún existe y es accesible. Si no existe, edite la implementación y quite la máquina.
* Consulte la sección sobre [planeamiento de red](../automation-update-management.md#ports) para obtener una lista de puertos y direcciones que son necesarios para la administración de actualizaciones y comprobación que el equipo cumple estos requisitos.
* Ejecute la siguiente consulta de Log Analytics buscar equipos en su entorno cuya `SourceComputerId` cambiado. Busque los equipos que tienen el mismo `Computer` valor, pero distintos `SourceComputerId` valor. Una vez que encuentre los equipos afectados, debe editar las implementaciones de actualizaciones que tener como destino esas máquinas, quitarán y volver a agregar las máquinas por lo que el `SourceComputerId` refleja el valor correcto.

   ```loganalytics
   Heartbeat | where TimeGenerated > ago(30d) | distinct SourceComputerId, Computer, ComputerIP
   ```

### <a name="hresult"></a>Escenario: La máquina aparece como No evaluado y se muestra una excepción HResult

#### <a name="issue"></a>Problema

Tiene máquinas que aparecen como **No evaluado** en **Cumplimiento**, y verá un mensaje de excepción debajo de él.

#### <a name="cause"></a>Causa

Windows Update o WSUS no están configurados correctamente en la máquina. Update Management se basa en Windows Update o WSUS para proporcionar las actualizaciones necesarias, el estado de la revisión y los resultados de las revisiones implementadas. Sin esta información, Update Management no puede informar correctamente de las revisiones que son necesarias o que están instaladas.

#### <a name="resolution"></a>Resolución

Haga doble clic en la excepción que se muestra en rojo para ver el mensaje de excepción completo. Revise la siguiente tabla para ver posibles soluciones o acciones que deben realizarse:

|Excepción  |Acción o resolución  |
|---------|---------|
|`Exception from HRESULT: 0x……C`     | Busque el código de error correspondiente en [Lista de códigos de error de Windows Update](https://support.microsoft.com/help/938205/windows-update-error-code-list) para buscar detalles adicionales sobre la causa de la excepción.        |
|`0x8024402C`</br>`0x8024401C`</br>`0x8024402F`      | Estos errores son problemas de conectividad de red. Asegúrese de que la máquina tenga la conectividad de red adecuada para Update Management. Consulte la sección sobre [planeamiento de red](../automation-update-management.md#ports) para obtener una lista de puertos y direcciones que se requieren.        |
|`0x8024001E`| No se completó la operación de actualización porque se estaba cerrando el servicio o el sistema.|
|`0x8024002E`| Servicio de Windows Update está deshabilitado.|
|`0x8024402C`     | Si usa un servidor WSUS, asegúrese de que los valores de registro de `WUServer` y `WUStatusServer` bajo la clave del registro `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate` tienen el servidor WSUS correcto.        |
|`The service cannot be started, either because it is disabled or because it has no enabled devices associated with it. (Exception from HRESULT: 0x80070422)`     | Asegúrese de que el servicio Windows Update (wuauserv) se está ejecutando y no está deshabilitado.        |
|Cualquier otra excepción genérica     | Realice una búsqueda en Internet para ver las soluciones posibles y colabore con el equipo de soporte técnico de TI local.         |

Revisar el `windowsupdate.log` puede ayudarle a probar determinar la posible causa. Para obtener más información sobre cómo leer el registro, consulte [cómo leer el archivo Windowsupdate.log](https://support.microsoft.com/en-ca/help/902093/how-to-read-the-windowsupdate-log-file).

Además, puede descargar y ejecutar el [solucionador de problemas de Windows Update](https://support.microsoft.com/help/4027322/windows-update-troubleshooter) para comprobar si hay algún problema con Windows Update en el equipo.

> [!NOTE]
> El [Solucionador de problemas de Windows Update](https://support.microsoft.com/help/4027322/windows-update-troubleshooter) indica que es para clientes de Windows, pero también funciona en Windows Server.

## <a name="linux"></a>Linux

### <a name="scenario-update-run-fails-to-start"></a>Escenario: No se puede iniciar la ejecución de actualización

#### <a name="issue"></a>Problema

No se pudo iniciar una ejecución de actualización en una máquina Linux.

#### <a name="cause"></a>Causa

La instancia de Hybrid Worker de Linux no es correcta.

#### <a name="resolution"></a>Resolución

Realice una copia del archivo de registro siguiente y consérvelo para la solución de problemas:

```bash
/var/opt/microsoft/omsagent/run/automationworker/worker.log
```

### <a name="scenario-update-run-starts-but-encounters-errors"></a>Escenario: La ejecución de actualización se inicia, pero encuentra errores

#### <a name="issue"></a>Problema

Se inicia una ejecución de la actualización, pero encuentra errores durante la ejecución.

#### <a name="cause"></a>Causa

Las posibles causas pueden ser:

* El estado del administrador de paquetes no es correcto
* Paquetes específicos pueden interferir con la aplicación de revisiones basada en la nube
* Otros motivos

#### <a name="resolution"></a>Resolución

Si se producen errores durante una ejecución de actualizaciones después de que se ha iniciado correctamente en Linux, compruebe el trabajo de salida desde la máquina afectada en la ejecución. Puede encontrar mensajes de error específicos procedentes del Administrador de paquetes de su máquina que puede investigar e intentar solucionar. Update Management requiere que el administrador de paquetes tenga un estado correcto para que las implementaciones de actualizaciones se realicen con éxito.

En algunos casos, las actualizaciones de paquetes pueden interferir con Update Management e impedir que finalice una implementación de actualizaciones. Si ese fuera el caso, deberá excluir estos paquetes de las futuras ejecuciones de actualizaciones o instalarlos manualmente por su cuenta.

Si no puede resolver un problema de aplicación de revisiones, realice una copia del archivo de registro siguiente y consérvela **antes** de que se inicie la siguiente implementación de actualizaciones para solucionar los problemas:

```bash
/var/opt/microsoft/omsagent/run/automationworker/omsupdatemgmt.log
```

## <a name="next-steps"></a>Pasos siguientes

Si su problema no aparece o es incapaz de resolverlo, visite uno de nuestros canales para obtener ayuda adicional:

* Obtenga respuestas de expertos de Azure en los [foros de Azure](https://azure.microsoft.com/support/forums/).
* Póngase en contacto con [@AzureSupport](https://twitter.com/azuresupport): la cuenta de Microsoft Azure oficial para mejorar la experiencia del cliente, que pone en contacto a la comunidad de Azure con los recursos adecuados: respuestas, soporte técnico y expertos.
* Si necesita más ayuda, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte**.
