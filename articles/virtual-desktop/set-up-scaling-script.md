---
title: 'Escalado de hosts de sesión con Azure Automation: Azure'
description: Escalado automático de hosts de sesión de Windows Virtual Desktop con Azure Automation
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 12a15ab1a4c7369c448e9f65862121b03ca05bba
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89078561"
---
# <a name="scale-session-hosts-using-azure-automation"></a>Escalado de hosts de sesión con Azure Automation

Puede reducir el costo total de implementación de Windows Virtual Desktop mediante el escalado de las máquinas virtuales. Esto significa que se cierran y desasignan las máquinas virtuales del host de sesión durante las horas de menos uso y, posteriormente, se vuelven a activar y se reasignan durante las horas punta.

En este artículo, obtendrá información sobre la herramienta de escalado creada con la cuenta de Azure Automation y Azure Logic Apps que escala automáticamente las máquinas virtuales del host de sesión en el entorno de Windows Virtual Desktop. Para más información sobre cómo usar la herramienta de escalado, consulte los [requisitos previos](#prerequisites).

## <a name="how-the-scaling-tool-works"></a>Funcionamiento de la herramienta de escalado

La herramienta de escalado proporciona una opción de automatización de bajo costo para aquellos clientes que deseen optimizar los costos de las máquinas virtuales del host de sesión.

Puede usar la herramienta de escalado para:

- Programar las máquinas virtuales para que se inicien y se detengan según el horario comercial de horas punta y horas de menos uso.
- Escalar horizontalmente las máquinas virtuales basadas en el número de sesiones por núcleo de CPU.
- Reducir horizontalmente las máquinas virtuales durante las horas de menos uso, lo que mantiene en ejecución el número mínimo de máquinas virtuales del host de sesión.

Para funcionar, la herramienta de escalado usa una combinación de cuentas de Azure Automation, runbooks de PowerShell, webhooks y Azure Logic Apps. Cuando se ejecuta la herramienta, la aplicación lógica de Azure llama a un webhook para iniciar el runbook de Azure Automation. A continuación, el runbook crea un trabajo.

Durante el tiempo de uso en horas punta, el trabajo comprueba el número actual de sesiones y la capacidad de la máquina virtual del host de sesión que se está ejecutando para cada grupo de hosts. Usa esta información para calcular si las máquinas virtuales del host de sesión pueden admitir las sesiones existentes en función del parámetro *SessionThresholdPerCPU* definido en el archivo **CreateOrUpdateAzLogicApp.ps1**. Si las máquinas virtuales del host de sesión no admiten las sesiones existentes, el trabajo inicia máquinas virtuales del host de sesión adicionales en el grupo de hosts.

>[!NOTE]
>*SessionThresholdPerCPU* no restringe el número de sesiones en la máquina virtual. Este parámetro solo determina cuándo es necesario iniciar nuevas máquinas virtuales para equilibrar la carga de las conexiones. Para restringir el número de sesiones, debe seguir las instrucciones de [Update-AzWvdHostPool](configure-host-pool-load-balancing.md#configure-breadth-first-load-balancing) para configurar el parámetro *MaxSessionLimit* en consecuencia.

En las horas de menos uso, el trabajo determina cuántas máquinas virtuales de los hosts de sesión deberían apagarse según el parámetro *MinimumNumberOfRDSH*. Si establece el parámetro *LimitSecondsToForceLogOffUser* en un valor positivo distinto de cero, el trabajo establecerá las máquinas virtuales del host de sesión en modo de purga para evitar que las sesiones nuevas se conecten a los hosts. Después, el trabajo enviará a los usuarios que tengan la sesión iniciada la notificación de que guarden su trabajo, esperará el periodo predeterminado y, después, obligará a los usuarios a cerrar la sesión. Cuando se hayan cerrado todas las sesiones de usuario en la máquina virtual del host de sesión, el trabajo apagará la máquina virtual. Una vez que la máquina virtual se apague, el trabajo restablecerá el modo de purga del host de sesión.

>[!NOTE]
>Si configura manualmente la máquina virtual del host de sesión en el modo de purga, el trabajo no administrará la máquina virtual del host de sesión. Si la máquina virtual del host de sesión está en ejecución y se establece en modo de purga, se tratará como no disponible, por lo que el trabajo iniciará máquinas virtuales adicionales para administrar la carga. Se recomienda etiquetar todas las máquinas virtuales de Azure antes de establecerlas manualmente en modo de purga. Puede asignar un nombre a la etiqueta con el parámetro *MaintenanceTagName* al crear la instancia de Azure Logic App Scheduler más adelante. Las etiquetas le ayudarán a distinguir estas máquinas virtuales de las que administra la herramienta de escalado. La configuración de la etiqueta de mantenimiento también evita que la herramienta de escalado realice cambios en la máquina virtual hasta que quite la etiqueta.

Si establece el parámetro *LimitSecondsToForceLogOffUser* en cero, el trabajo permite que el valor de configuración de la sesión en las directivas de grupo especificadas controle el cierre de las sesiones de los usuarios. Para ver estas directivas de grupo, vaya a **Configuración del equipo** > **Directivas** > **Plantillas administrativas** > **Componentes de Windows** > **Servicios de Escritorio remoto** > **Host de sesión de Escritorio remoto** > **Límites de tiempo de sesión**. Si hay sesiones activas en una máquina virtual del host de sesión, el trabajo dejará que se ejecute esa máquina. Si no hay ninguna sesión activa, el trabajo apagará la máquina virtual del host de sesión.

En cualquier momento, el trabajo también tiene en cuenta el elemento *MaxSessionLimit* del grupo de hosts para determinar si el número actual de sesiones es superior al 90 % de la capacidad máxima. Si es así, el trabajo iniciará máquinas virtuales de host de sesión adicionales.

El trabajo se ejecuta periódicamente según un intervalo de periodicidad establecido. Puede cambiar este intervalo según el tamaño del entorno de Windows Virtual Desktop, pero recuerde que el inicio y el apagado de las VM pueden tardar algún tiempo, por lo que debe tener en cuenta el retraso. Se recomienda establecer el intervalo de periodicidad en 15 minutos.

Sin embargo, la herramienta también tiene las siguientes limitaciones:

- Esta solución solo se aplica a las máquinas virtuales de host multisesión agrupadas.
- Esta solución administra máquinas virtuales de cualquier región, pero solo se puede usar en la misma suscripción que la cuenta de Azure Automation y Azure Logic Apps.
- El tiempo de ejecución máximo de un trabajo en el runbook es de tres horas. Si el inicio o la detención de las máquinas virtuales del grupo de hosts tarda más, se producirá un error en el trabajo. Para obtener más información, consulte [Recursos compartidos](../automation/automation-runbook-execution.md#fair-share).

>[!NOTE]
>La herramienta de escalado controla el modo de equilibrio de carga del grupo de hosts que se está escalando actualmente. La herramienta usa el modo de equilibrio de carga en amplitud tanto en las horas punta como en las horas de menos uso.

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar a configurar la herramienta de escalado, asegúrese de tener preparado lo siguiente:

- Un [grupo de hosts de Windows Virtual Desktop](create-host-pools-azure-marketplace.md)
- Máquinas virtuales del grupo de hosts de sesión configuradas y registradas en el servicio Windows Virtual Desktop
- Un usuario con [acceso de colaborador](../role-based-access-control/role-assignments-portal.md) en la suscripción de Azure

La máquina que se usa para implementar la herramienta debe tener:

- Windows PowerShell 5.1 o posterior
- El módulo Microsoft Az PowerShell

Si tiene todo preparado, vamos a empezar.

## <a name="create-or-update-an-azure-automation-account"></a>Creación o actualización de una cuenta de Azure Automation

>[!NOTE]
>Si ya tiene una cuenta de Azure Automation con un runbook que ejecuta una versión anterior del script de escalado, solo debe seguir las instrucciones a continuación para asegurarse de que está actualizado.

En primer lugar, necesitará una cuenta de Azure Automation para ejecutar el runbook de PowerShell. El proceso que se describe en esta sección es válido incluso si tiene una cuenta de Azure Automation existente que quiera usar para configurar el runbook de PowerShell. Así es como se configura:

1. Abra Windows PowerShell.

2. Ejecute el siguiente cmdlet para iniciar sesión en la cuenta de Azure.

    ```powershell
    Login-AzAccount
    ```

    >[!NOTE]
    >La cuenta debe tener derechos de colaborador en la suscripción a Azure en la que quiere implementar la herramienta de escalado.

3. Ejecute el siguiente cmdlet para descargar el script para crear la cuenta de Azure Automation:

    ```powershell
    New-Item -ItemType Directory -Path "C:\Temp" -Force
    Set-Location -Path "C:\Temp"
    $Uri = "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-scaling-script/CreateOrUpdateAzAutoAccount.ps1"
    # Download the script
    Invoke-WebRequest -Uri $Uri -OutFile ".\CreateOrUpdateAzAutoAccount.ps1"
    ```

4. Ejecute el siguiente cmdlet para ejecutar el script y crear la cuenta de Azure Automation. Puede rellenar los valores de los parámetros o marcarlos como comentario para que se usen los valores predeterminados.

    ```powershell
    $Params = @{
         "AADTenantId"           = "<Azure_Active_Directory_tenant_ID>"   # Optional. If not specified, it will use the current Azure context
         "SubscriptionId"        = "<Azure_subscription_ID>"              # Optional. If not specified, it will use the current Azure context
         "UseARMAPI"             = $true
         "ResourceGroupName"     = "<Resource_group_name>"                # Optional. Default: "WVDAutoScaleResourceGroup"
         "AutomationAccountName" = "<Automation_account_name>"            # Optional. Default: "WVDAutoScaleAutomationAccount"
         "Location"              = "<Azure_region_for_deployment>"
         "WorkspaceName"         = "<Log_analytics_workspace_name>"       # Optional. If specified, Log Analytics will be used to configure the custom log table that the runbook PowerShell script can send logs to
    }

    .\CreateOrUpdateAzAutoAccount.ps1 @Params
    ```

5. La salida del cmdlet incluirá un URI de webhook. Asegúrese de mantener un registro del URI porque lo usará como parámetro cuando configure la programación de ejecución para la aplicación lógica de Azure.

6. Si especificó el parámetro **WorkspaceName** para Log Analytics, la salida del cmdlet también incluirá el id. del área de trabajo de Log Analytics y su clave principal. Asegúrese de recordar el identificador URI, porque deberá usarlo más tarde como parámetro cuando configure la programación de ejecución para la aplicación lógica de Azure.

7. Después de configurar la cuenta de Azure Automation, inicie sesión en la suscripción de Azure y asegúrese de que la cuenta de Azure Automation y el runbook correspondiente han aparecido en el grupo de recursos especificado, tal como se muestra en la siguiente imagen:

    >[!div class="mx-imgBorder"]
    >![Una imagen de la página de información general de Azure que muestra la cuenta de Azure Automation y el runbook creados recientemente.](media/automation-account.png)

    Para comprobar si el webhook está donde debe estar, seleccione el nombre del runbook. A continuación, vaya a la sección Recursos del runbook y seleccione **Webhooks**.

## <a name="create-an-azure-automation-run-as-account"></a>Creación de una cuenta de ejecución de Azure Automation

Ahora que tiene una cuenta de Azure Automation, también deberá crear una cuenta de ejecución de Azure Automation si aún no tiene una. Esta cuenta permitirá a la herramienta acceder a los recursos de Azure.

La [cuenta de ejecución en Azure Automation](../automation/manage-runas-account.md) proporciona autenticación para administrar los recursos de Azure con los cmdlets de Azure. Cuando se crea una cuenta de ejecución, se crea un nuevo usuario de la entidad de servicio en Azure Active Directory y se asigna el rol Colaborador a este usuario de entidad de servicio en el nivel de suscripción. Una cuenta de ejecución de Azure es una excelente manera de autenticarse de forma segura con los certificados y un nombre de entidad de seguridad de servicio sin necesidad de almacenar un nombre de usuario y una contraseña en un objeto de credenciales. Para obtener más información sobre la autenticación de la cuenta de ejecución, consulte [Limitación de los permisos de las cuentas de ejecución](../automation/manage-runas-account.md#limit-run-as-account-permissions).

Cualquier usuario que sea miembro del rol de administradores de suscripciones y coadministrador de la suscripción puede crear una cuenta de ejecución.

Para crear una cuenta de ejecución en la cuenta de Azure Automation:

1. En Azure Portal, seleccione **Todos los servicios**. En la lista de recursos, escriba y seleccione **Cuentas de Automation**.

2. En la página **Cuentas de Automation**, seleccione su cuenta de Azure Automation.

3. En el panel de la izquierda de la ventana, seleccione **Cuentas de ejecución** en la sección **Configuración de la cuenta**.

4. Seleccione **Cuenta de ejecución de Azure**. Cuando se muestre el panel **Agregar cuenta de ejecución de Azure**, revise la información general y, después, seleccione **Crear** para iniciar el proceso de creación de la cuenta.

5. Espere unos minutos para que Azure cree la cuenta de ejecución. Puede realizar el seguimiento del progreso de la creación en el menú en Notificaciones.

6. Cuando el proceso finaliza, crea un recurso denominado **AzureRunAsConnection** en la cuenta de Azure Automation especificada. Seleccione **Cuenta de ejecución de Azure**. El recurso de conexión contiene el identificador de la aplicación, el identificador del inquilino, el identificador de la suscripción y la huella digital del certificado. También puede ver la misma información en la página **Conexiones**. Para ir a esta página, en el panel de la izquierda de la ventana, seleccione **Conexiones** en la sección **Recursos compartidos** y haga clic en el recurso de conexión denominado **AzureRunAsConnection**.

## <a name="create-the-azure-logic-app-and-execution-schedule"></a>Creación de la aplicación lógica de Azure y la programación de ejecución

Por último, deberá crear la aplicación lógica de Azure y configurar una programación de ejecución para la nueva herramienta de escalado. En primer lugar y, si aún no lo ha hecho, descargue e importe el [módulo de PowerShell para virtualización de Escritorio](powershell-module.md) que se usará en la sesión de PowerShell.

1. Abra Windows PowerShell.

2. Ejecute el siguiente cmdlet para iniciar sesión en la cuenta de Azure.

    ```powershell
    Login-AzAccount
    ```

3. Ejecute el siguiente cmdlet para descargar el script para crear la aplicación lógica de Azure.

    ```powershell
    New-Item -ItemType Directory -Path "C:\Temp" -Force
    Set-Location -Path "C:\Temp"
    $Uri = "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-scaling-script/CreateOrUpdateAzLogicApp.ps1"
    # Download the script
    Invoke-WebRequest -Uri $Uri -OutFile ".\CreateOrUpdateAzLogicApp.ps1"
    ```

4. Ejecute el siguiente script de PowerShell para crear la aplicación lógica de Azure y la programación de ejecución del grupo de hosts.

    >[!NOTE]
    >Tendrá que ejecutar este script en cada grupo de hosts que quiera escalar automáticamente, pero solo necesita una cuenta de Azure Automation.

    ```powershell
    $AADTenantId = (Get-AzContext).Tenant.Id

    $AzSubscription = Get-AzSubscription | Out-GridView -OutputMode:Single -Title "Select your Azure Subscription"
    Select-AzSubscription -Subscription $AzSubscription.Id

    $ResourceGroup = Get-AzResourceGroup | Out-GridView -OutputMode:Single -Title "Select the resource group for the new Azure Logic App"

    $WVDHostPool = Get-AzResource -ResourceType "Microsoft.DesktopVirtualization/hostpools" | Out-GridView -OutputMode:Single -Title "Select the host pool you'd like to scale"

    $LogAnalyticsWorkspaceId = Read-Host -Prompt "If you want to use Log Analytics, enter the Log Analytics Workspace ID returned by when you created the Azure Automation account, otherwise leave it blank"
    $LogAnalyticsPrimaryKey = Read-Host -Prompt "If you want to use Log Analytics, enter the Log Analytics Primary Key returned by when you created the Azure Automation account, otherwise leave it blank"
    $RecurrenceInterval = Read-Host -Prompt "Enter how often you'd like the job to run in minutes, e.g. '15'"
    $BeginPeakTime = Read-Host -Prompt "Enter the start time for peak hours in local time, e.g. 9:00"
    $EndPeakTime = Read-Host -Prompt "Enter the end time for peak hours in local time, e.g. 18:00"
    $TimeDifference = Read-Host -Prompt "Enter the time difference between local time and UTC in hours, e.g. +5:30"
    $SessionThresholdPerCPU = Read-Host -Prompt "Enter the maximum number of sessions per CPU that will be used as a threshold to determine when new session host VMs need to be started during peak hours"
    $MinimumNumberOfRDSH = Read-Host -Prompt "Enter the minimum number of session host VMs to keep running during off-peak hours"
    $MaintenanceTagName = Read-Host -Prompt "Enter the name of the Tag associated with VMs you don't want to be managed by this scaling tool"
    $LimitSecondsToForceLogOffUser = Read-Host -Prompt "Enter the number of seconds to wait before automatically signing out users. If set to 0, any session host VM that has user sessions, will be left untouched"
    $LogOffMessageTitle = Read-Host -Prompt "Enter the title of the message sent to the user before they are forced to sign out"
    $LogOffMessageBody = Read-Host -Prompt "Enter the body of the message sent to the user before they are forced to sign out"

    $AutoAccount = Get-AzAutomationAccount | Out-GridView -OutputMode:Single -Title "Select the Azure Automation account"
    $AutoAccountConnection = Get-AzAutomationConnection -ResourceGroupName $AutoAccount.ResourceGroupName -AutomationAccountName $AutoAccount.AutomationAccountName | Out-GridView -OutputMode:Single -Title "Select the Azure RunAs connection asset"

    $WebhookURIAutoVar = Get-AzAutomationVariable -Name 'WebhookURIARMBased' -ResourceGroupName $AutoAccount.ResourceGroupName -AutomationAccountName $AutoAccount.AutomationAccountName

    $Params = @{
         "AADTenantId"                   = $AADTenantId                             # Optional. If not specified, it will use the current Azure context
         "SubscriptionID"                = $AzSubscription.Id                       # Optional. If not specified, it will use the current Azure context
         "ResourceGroupName"             = $ResourceGroup.ResourceGroupName         # Optional. Default: "WVDAutoScaleResourceGroup"
         "Location"                      = $ResourceGroup.Location                  # Optional. Default: "West US2"
         "UseARMAPI"                     = $true
         "HostPoolName"                  = $WVDHostPool.Name
         "HostPoolResourceGroupName"     = $WVDHostPool.ResourceGroupName           # Optional. Default: same as ResourceGroupName param value
         "LogAnalyticsWorkspaceId"       = $LogAnalyticsWorkspaceId                 # Optional. If not specified, script will not log to the Log Analytics
         "LogAnalyticsPrimaryKey"        = $LogAnalyticsPrimaryKey                  # Optional. If not specified, script will not log to the Log Analytics
         "ConnectionAssetName"           = $AutoAccountConnection.Name              # Optional. Default: "AzureRunAsConnection"
         "RecurrenceInterval"            = $RecurrenceInterval                      # Optional. Default: 15
         "BeginPeakTime"                 = $BeginPeakTime                           # Optional. Default: "09:00"
         "EndPeakTime"                   = $EndPeakTime                             # Optional. Default: "17:00"
         "TimeDifference"                = $TimeDifference                          # Optional. Default: "-7:00"
         "SessionThresholdPerCPU"        = $SessionThresholdPerCPU                  # Optional. Default: 1
         "MinimumNumberOfRDSH"           = $MinimumNumberOfRDSH                     # Optional. Default: 1
         "MaintenanceTagName"            = $MaintenanceTagName                      # Optional.
         "LimitSecondsToForceLogOffUser" = $LimitSecondsToForceLogOffUser           # Optional. Default: 1
         "LogOffMessageTitle"            = $LogOffMessageTitle                      # Optional. Default: "Machine is about to shutdown."
         "LogOffMessageBody"             = $LogOffMessageBody                       # Optional. Default: "Your session will be logged off. Please save and close everything."
         "WebhookURI"                    = $WebhookURIAutoVar.Value
    }

    .\CreateOrUpdateAzLogicApp.ps1 @Params
    ```

    Después de ejecutar el script, la aplicación lógica de Azure debe aparecer en un grupo de recursos, como se muestra en la siguiente imagen.

    >[!div class="mx-imgBorder"]
    >![Imagen de la página de información general para ver una aplicación lógica de Azure de ejemplo](media/logic-app.png)

    Para realizar cambios en la programación de ejecución, como cambiar el intervalo de periodicidad o la zona horaria, vaya al programador de escalabilidad automática de la aplicación lógica de Azure y seleccione **Editar** para ir al diseñador de aplicaciones lógicas de Azure.

    >[!div class="mx-imgBorder"]
    >![Imagen del diseñador de aplicaciones lógicas de Azure. Los menús Periodicidad y webhook que permiten al usuario editar los tiempos de periodicidad y el archivo de webhook están abiertos.](media/logic-apps-designer.png)

## <a name="manage-your-scaling-tool"></a>Administración de la herramienta de escalado

Ahora que ha creado la herramienta de escalado, puede acceder a su salida. En esta sección se describen algunas características que pueden resultarle útiles.

### <a name="view-job-status"></a>Ver estado del trabajo

Puede ver un resumen del estado de todos los trabajos del runbook o profundizar en el estado de un trabajo de runbook específico en Azure Portal.

A la derecha de la cuenta de Azure Automation seleccionada, en "Estadísticas del trabajo" puede ver una lista de los resúmenes de todos los trabajos de runbook. Al abrir la página **Trabajos** en el lado izquierdo de la ventana, se muestran los estados de trabajo actuales, las horas de inicio y las horas de finalización.

>[!div class="mx-imgBorder"]
>![Captura de pantalla de la página de estado del trabajo](media/jobs-status.png)

### <a name="view-logs-and-scaling-tool-output"></a>Visualización de registros y salida de la herramienta de escalado

Para ver los registros de las operaciones de escalabilidad y reducción horizontales, abra el runbook y seleccione el trabajo.

Vaya al runbook en el grupo de recursos que hospeda la cuenta de Azure Automation y seleccione **Información general**. En la página Información general, seleccione un trabajo en **Trabajos recientes** para ver la salida de la herramienta de escalado, tal como se muestra en la siguiente imagen.

>[!div class="mx-imgBorder"]
>![Imagen de la ventana de salida de la herramienta de escalado](media/tool-output.png)

### <a name="check-the-runbook-script-version-number"></a>Comprobación del número de versión del script de runbook

Para comprobar qué versión del script de runbook está usando, abra el archivo de runbook en la cuenta de Azure Automation y seleccione **Ver**. Aparecerá un script para el runbook en el lado derecho de la pantalla. En el script, aparece el número de versión con el formato `v#.#.#` bajo la sección `SYNOPSIS`. Podrá encontrar el número de versión más reciente [aquí](https://github.com/Azure/RDS-Templates/blob/master/wvd-templates/wvd-scaling-script/ARM_based/basicScale.ps1#L1). Si no ve un número de versión en el script del runbook, significará que está ejecutando una versión anterior del script y debe actualizarlo de inmediato. Si necesita actualizar el script de runbook, siga las instrucciones de la sección [Creación o actualización de una cuenta de Azure Automation](#create-or-update-an-azure-automation-account).

### <a name="reporting-issues"></a>Información sobre los problemas

Al notificar un problema, deberá proporcionar la siguiente información para ayudarnos a solucionarlo:

- Un registro completo de la pestaña **Todos los registros** del trabajo que causó el problema. Para obtener información sobre cómo obtener el registro, siga las instrucciones de la sección [Visualización de registros y salida de la herramienta de escalado](#view-logs-and-scaling-tool-output). Si hay información confidencial o privada en el registro, puede quitarla antes de informarnos sobre el problema.

- La versión del script de runbook que está usando. Para saber cómo obtener el número de versión, consulte la sección [Comprobación del número de versión del script de runbook](#check-the-runbook-script-version-number).

- El número de versión de cada uno de los siguientes módulos de PowerShell que están instalados en su cuenta de Azure Automation. Para buscar estos módulos, abra la cuenta de Azure Automation, seleccione **Módulos** en la sección **Recursos compartidos** en el panel del lado izquierdo de la ventana y, a continuación, busque el nombre del módulo.
    - Az.Accounts
    - Az.Compute
    - Az.Resources
    - Az.Automation
    - OMSIngestionAPI
    - Az.DesktopVirtualization

- La fecha de expiración de la [cuenta de ejecución](#create-an-azure-automation-run-as-account). Para buscar este dato, abra la cuenta de Azure Automation y, a continuación, seleccione **Cuentas de ejecución** en **Configuración de cuenta** en el panel de la izquierda de la ventana. La fecha de expiración debe encontrarse debajo de **Cuenta de ejecución de Azure**.

### <a name="log-analytics"></a>Log Analytics

Si decidió usar Log Analytics, puede ver todos los datos de registros en un registro personalizado denominado **WVDTenantScale_CL** debajo de **Registros personalizados** en la vista **Registros** de su área de trabajo de Log Analytics. Hemos enumerado algunas consultas de ejemplo que pueden resultarle útiles.

- Para ver todos los registros de un grupo hosts, escriba la siguiente consulta.

    ```Kusto
    WVDTenantScale_CL
    | where hostpoolName_s == "<host_pool_name>"
    | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
    ```

- Para ver el número total de máquinas virtuales de host de sesión que se están ejecutando actualmente y las sesiones de usuario activas en el grupo de hosts, escriba la siguiente consulta.

    ```Kusto
    WVDTenantScale_CL
    | where logmessage_s contains "Number of running session hosts:"
         or logmessage_s contains "Number of user sessions:"
         or logmessage_s contains "Number of user sessions per Core:"
    | where hostpoolName_s == "<host_pool_name>"
    | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
    ```

- Para ver el estado de todas las máquinas virtuales de host de sesión en un grupo de hosts, escriba la siguiente consulta.

    ```Kusto
    WVDTenantScale_CL
    | where logmessage_s contains "Session host:"
    | where hostpoolName_s == "<host_pool_name>"
    | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
    ```

- Para ver los errores y advertencias, escriba la siguiente consulta.

    ```Kusto
    WVDTenantScale_CL
    | where logmessage_s contains "ERROR:" or logmessage_s contains "WARN:"
    | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
    ```

## <a name="report-issues"></a>Informar de problemas

En la actualidad, el servicio de soporte técnico de Microsoft administra los informes de problemas de la herramienta de escalado. Cuando cree un informe de problemas, asegúrese de seguir las instrucciones descritas en [Informes de problemas](#reporting-issues). Si tiene comentarios sobre la herramienta o desea solicitar nuevas características, abra una incidencia de GitHub con la etiqueta "4-WVD-scaling-Tool" en la [página RDS de GitHub](https://github.com/Azure/RDS-Templates/issues?q=is%3Aissue+is%3Aopen+label%3A4-WVD-scaling-tool).
