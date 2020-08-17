---
title: Solución de problemas de runbook de Azure Automation
description: En este artículo se describe cómo solucionar y resolver problemas con runbooks de Azure Automation.
services: automation
ms.date: 07/28/2020
ms.topic: conceptual
ms.service: automation
ms.custom: has-adal-ref
ms.openlocfilehash: 1cbb5be8c1a4045b218c0e6bf5ac7ed0b901aa80
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87904809"
---
# <a name="troubleshoot-runbook-issues"></a>Solución de incidencias de runbooks

 En este artículo se describen los errores de runbook que pueden producirse y cómo resolverlos. Para obtener información general, consulte [Ejecución de un runbook en Azure Automation](../automation-runbook-execution.md).

## <a name="diagnose-runbook-issues"></a>Diagnóstico de problemas de runbooks

Si se producen errores durante la ejecución del runbook en Azure Automation, puede usar los pasos siguientes para ayudar a diagnosticar los problemas:

1. Asegúrese de que el script de runbook se ejecuta correctamente en la máquina local.

    Para información sobre los módulos de referencia de lenguaje y aprendizaje, consulte la [documentación de PowerShell](/powershell/scripting/overview) o la [documentación de Python](https://docs.python.org/3/). La ejecución local del script puede detectar y resolver errores comunes, como, por ejemplo:

      * Módulos que faltan
      * Errores de sintaxis
      * Errores lógicos

1. Investigue los [flujos de errores](../automation-runbook-output-and-messages.md#runbook-output) del runbook.

    Examine estos flujos en busca de mensajes específicos y compárelos con los errores documentados en este artículo.

1. Asegúrese de que los nodos y el área de trabajo de Automation tengan los módulos necesarios.

    Si el runbook importa algún módulo, compruebe que está disponible para la cuenta de Automation mediante los pasos indicados en [Importación de módulos](../shared-resources/modules.md#import-modules). Actualice los módulos de PowerShell a la versión más reciente siguiendo las instrucciones del artículo [Actualización de módulos de Azure PowerShell en Azure Automation](../automation-update-azure-modules.md). Para más información sobre solución de problemas, consulte [Solución de problemas de módulos](shared-resources.md#modules).

1. Haga lo siguiente si el runbook se suspende o registra un error inesperado:

    * [Renueve el certificado](../manage-runas-account.md#cert-renewal) si la cuenta de ejecución ha expirado.
    * [Renueve el webhook](../automation-webhooks.md#renew-a-webhook) si está intentando usar un webhook expirado para iniciar el runbook.
    * [Compruebe los estados de los trabajos](../automation-runbook-execution.md#job-statuses) para determinar los estados actuales del runbook y algunas posibles causas del problema.
    * [Agregue datos de salida adicionales](../automation-runbook-output-and-messages.md#monitor-message-streams) al runbook para identificar lo que sucede antes de que se suspenda el runbook.
    * [Controle las excepciones](../automation-runbook-execution.md#exceptions) producidas por su trabajo.

1. Realice este paso si el trabajo del runbook o el entorno de Hybrid Runbook Worker no responden.

    Si ejecuta los runbooks en una instancia de Hybrid Runbook Worker en lugar de Azure Automation, puede que necesite [solucionar los propios problemas de Hybrid Worker](hybrid-runbook-worker.md).

## <a name="scenario-runbook-fails-with-a-no-permission-or-forbidden-403-error"></a><a name="runbook-fails-no-permission"></a>Escenario: Se produce un error en el runbook del tipo Sin permisos o Prohibido (403).

### <a name="issue"></a>Incidencia

Se produce un error en el runbook del tipo Sin permisos, Prohibido (403) o equivalente.

### <a name="cause"></a>Causa

Es posible que las cuentas de ejecución no tengan los mismos permisos en los recursos de Azure que su cuenta de Automation actual. 

### <a name="resolution"></a>Resolución

Asegúrese de que su cuenta de ejecución tiene [permisos para acceder a todos los recursos](../../role-based-access-control/role-assignments-portal.md) que se usan en el script.

## <a name="scenario-sign-in-to-azure-account-failed"></a><a name="sign-in-failed"></a>Escenario: Error al iniciar sesión en la cuenta de Azure

### <a name="issue"></a>Incidencia

Se produce uno de los siguientes errores al trabajar con el cmdlet `Connect-AzAccount`:

```error
Unknown_user_type: Unknown User Type
```

```error
No certificate was found in the certificate store with thumbprint
```

### <a name="cause"></a>Causa

Estos errores se producen si el nombre de recurso de credencial no es válido. También pueden producirse si el nombre de usuario y la contraseña que usó para configurar el recurso de credencial de Automation no son válidos.

### <a name="resolution"></a>Resolución

Para determinar cuál es el problema, siga estos pasos:

1. Asegúrese de no haber incluido ningún carácter especial. Entre estos se incluye el carácter `\@` en el nombre del recurso de credencial de Automation que esté usando para conectarse a Azure.
1. Compruebe que puede usar el nombre de usuario y la contraseña que están almacenados en la credencial de Azure Automation en su editor de ISE de PowerShell local. Ejecute los siguientes cmdlets en el ISE de PowerShell.

   ```powershell
   $Cred = Get-Credential
   #Using Azure Service Management
   Add-AzureAccount –Credential $Cred
   #Using Azure Resource Manager
   Connect-AzAccount –Credential $Cred
   ```

1. Si la autenticación produce un error a nivel local, significa que no ha configurado correctamente las credenciales de Azure Active Directory (Azure AD). Para configurar la cuenta de Azure AD correctamente, consulte el artículo [Autenticación en Azure mediante Azure Active Directory](../automation-use-azure-ad.md).

1. Si el error parece ser transitorio, intente agregar lógica de reintento a su rutina de autenticación para hacer de la autenticación un proceso más sólido.

   ```powershell
   # Get the connection "AzureRunAsConnection"
   $connectionName = "AzureRunAsConnection"
   $servicePrincipalConnection = Get-AutomationConnection -Name $connectionName

   $logonAttempt = 0
   $logonResult = $False

   while(!($connectionResult) -And ($logonAttempt -le 10))
   {
       $LogonAttempt++
       #Logging in to Azure...
       $connectionResult = Connect-AzAccount `
                              -ServicePrincipal `
                              -Tenant $servicePrincipalConnection.TenantId `
                              -ApplicationId $servicePrincipalConnection.ApplicationId `
                              -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint

       Start-Sleep -Seconds 30
   }
   ```

## <a name="scenario-run-login-azurermaccount-to-log-in"></a><a name="login-azurerm"></a>Escenario: Ejecución de Login-AzureRMAccount para iniciar sesión

### <a name="issue"></a>Incidencia

Recibe el error siguiente al ejecutar un runbook:

```error
Run Login-AzureRMAccount to login.
```

### <a name="cause"></a>Causa

Este error puede producirse cuando no usa una cuenta de ejecución o esta ha expirado. Para más información, consulte [Administración de cuentas de ejecución en Azure Automation](../manage-runas-account.md).

Este error tiene dos causas principales:

* Hay versiones diferentes del módulo de AzureRM o Az.
* Intenta acceder a los recursos de otra suscripción.

### <a name="resolution"></a>Resolución

Si recibe este error después de actualizar un módulo de AzureRM o Az, actualice todos los módulos a la misma versión.

Si intenta acceder a los recursos de otra suscripción, siga estos pasos para configurar los permisos:

1. Vaya a la cuenta de ejecución de Automation y copie los valores de **Application ID** (Id. de aplicación) y **Thumbprint** (Huella digital).

    ![Copia del identificador de la aplicación y la huella digital](../media/troubleshoot-runbooks/collect-app-id.png)

1. Vaya a la opción **Access control** (Control de acceso) de la suscripción donde la cuenta de Automation *no* está hospedada y agregue una asignación de roles nueva.

    ![Control de acceso](../media/troubleshoot-runbooks/access-control.png)

1. Agregue el valor de **Application ID** (Id. de aplicación) recopilado anteriormente. Seleccione los permisos de **Contributor** (Colaborador).

    ![Agregar asignación de roles](../media/troubleshoot-runbooks/add-role-assignment.png)

1. Copie el nombre de la suscripción.

1. Ahora puede usar el código del runbook siguiente para probar los permisos de la cuenta de Automation en la otra suscripción. Reemplace `"\<CertificateThumbprint\>"` por el valor que copió en el paso 1. Reemplace `"\<SubscriptionName\>"` por el valor que copió en el paso 4.

    ```powershell
    $Conn = Get-AutomationConnection -Name AzureRunAsConnection
    Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint "<CertificateThumbprint>"
    #Select the subscription you want to work with
    Select-AzSubscription -SubscriptionName '<YourSubscriptionNameGoesHere>'

    #Test and get outputs of the subscriptions you granted access.
    $subscriptions = Get-AzSubscription
    foreach($subscription in $subscriptions)
    {
        Set-AzContext $subscription
        Write-Output $subscription.Name
    }
    ```

## <a name="scenario-unable-to-find-the-azure-subscription"></a><a name="unable-to-find-subscription"></a>Escenario: No se encuentra la suscripción de Azure

### <a name="issue"></a>Incidencia

Recibirá el siguiente error al trabajar con el cmdlet `Select-AzureSubscription`, `Select-AzureRMSubscription` o `Select-AzSubscription`:

```error
The subscription named <subscription name> cannot be found.
```

### <a name="error"></a>Error

Este error se puede generar si:

* El nombre de la suscripción no es válido.
* El usuario de Azure AD que intenta obtener los detalles de la suscripción no está configurado como administrador de la suscripción.
* El cmdlet no está disponible.

### <a name="resolution"></a>Resolución

Siga estos pasos para determinar si se ha autenticado en Azure y tiene acceso a la suscripción que intenta seleccionar:

1. Para asegurarse de que el script funciona de forma independiente, pruébelo fuera de Azure Automation.
1. Asegúrese de que el script ejecute el cmdlet [Connect-AzAccount](/powershell/module/Az.Accounts/Connect-AzAccount?view=azps-3.7.0) antes que el cmdlet `Select-*`.
1. Agregue `Disable-AzContextAutosave –Scope Process` al principio del runbook. Este cmdlet garantiza que las credenciales solo se aplican a la ejecución del runbook actual.
1. Si continúa recibiendo el mensaje de error, modifique el código. Para ello, agregue el parámetro `AzContext` a `Connect-AzAccount` y luego ejecute el código.

   ```powershell
   Disable-AzContextAutosave –Scope Process

   $Conn = Get-AutomationConnection -Name AzureRunAsConnection
   Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

   $context = Get-AzContext

   Get-AzVM -ResourceGroupName myResourceGroup -AzContext $context
    ```

## <a name="scenario-runbooks-fail-when-dealing-with-multiple-subscriptions"></a><a name="runbook-auth-failure"></a>Escenario: se produce un error en los runbooks cuando se trabaja con varias suscripciones

### <a name="issue"></a>Incidencia

Cuando se ejecuta un runbook, este no puede administrar los recursos de Azure.

### <a name="cause"></a>Causa

El runbook no está utilizando el contexto correcto cuando se ejecuta.

### <a name="resolution"></a>Resolución

El contexto de la suscripción podría perderse cuando un runbook invoca varios runbooks. Para asegurarse de que el contexto de la suscripción se pasa a los runbooks, haga que el runbook cliente pase el contexto al cmdlet `Start-AzureRmAutomationRunbook` en el parámetro `AzureRmContext`. Use el cmdlet `Disable-AzureRmContextAutosave` con el parámetro `Scope` establecido en `Process` para garantizar que las credenciales especificadas solo se usan para el runbook actual. Para más información, consulte [Suscripciones](../automation-runbook-execution.md#subscriptions).

```azurepowershell-interactive
# Ensures that any credentials apply only to the execution of this runbook
Disable-AzContextAutosave –Scope Process

# Connect to Azure with Run As account
$ServicePrincipalConnection = Get-AutomationConnection -Name 'AzureRunAsConnection'

Connect-AzAccount `
    -ServicePrincipal `
    -Tenant $ServicePrincipalConnection.TenantId `
    -ApplicationId $ServicePrincipalConnection.ApplicationId `
    -CertificateThumbprint $ServicePrincipalConnection.CertificateThumbprint

$AzContext = Select-AzSubscription -SubscriptionId $ServicePrincipalConnection.SubscriptionID

$params = @{"VMName"="MyVM";"RepeatCount"=2;"Restart"=$true}

Start-AzAutomationRunbook `
    –AutomationAccountName 'MyAutomationAccount' `
    –Name 'Test-ChildRunbook' `
    -ResourceGroupName 'LabRG' `
    -AzContext $AzureContext `
    –Parameters $params –wait
```

## <a name="scenario-authentication-to-azure-fails-because-multifactor-authentication-is-enabled"></a><a name="auth-failed-mfa"></a>Escenario: Error de autenticación en Azure porque está habilitada la autenticación multifactor

### <a name="issue"></a>Incidencia

Recibe el siguiente error al autenticarse en Azure con el nombre de usuario y la contraseña de Azure:

```error
Add-AzureAccount: AADSTS50079: Strong authentication enrollment (proof-up) is required
```

### <a name="cause"></a>Causa

Si dispone de autenticación multifactor en su cuenta de Azure, no puede usar un usuario de Azure Active Directory para autenticarse en Azure. En su lugar, tiene que usar un certificado o una entidad de servicio para autenticarse.

### <a name="resolution"></a>Resolución

Para usar una cuenta de ejecución clásica con los cmdlets del modelo de implementación clásica de Azure, consulte el artículo sobre cómo [crear una cuenta de ejecución clásica para administrar los servicios de Azure](../automation-create-standalone-account.md#create-a-classic-run-as-account). Para usar una entidad de servicio con los cmdlets de Azure Resource Manager, consulte [Creación de una entidad de servicio mediante Azure Portal](../../active-directory/develop/howto-create-service-principal-portal.md) y [Autenticación de una entidad de servicio con Azure Resource Manager](../../active-directory/develop/howto-authenticate-service-principal-powershell.md).

## <a name="scenario-runbook-fails-with-a-task-was-canceled-error-message"></a><a name="task-was-cancelled"></a>Escenario: Se produce un error en un runbook con el mensaje "Se canceló una tarea"

### <a name="issue"></a>Incidencia

El runbook genera un error similar al ejemplo siguiente:

```error
Exception: A task was canceled.
```

### <a name="cause"></a>Causa

Este error puede deberse al uso de módulos de Azure obsoletos.

### <a name="resolution"></a>Resolución

Puede solucionar este error con la actualización de los módulos de Azure a la versión más reciente:

1. En la cuenta de Automation, seleccione **Modules** (Módulos) y, luego, elija **Update Azure modules** (Actualizar módulos de Azure).
1. La actualización tarda aproximadamente 15 minutos. Una vez finalizada, vuelva a ejecutar el runbook en el que se produjo el error.

Para más información acerca de cómo actualizar los módulos, consulte [Actualización de módulos de Azure en Azure Automation](../automation-update-azure-modules.md).

## <a name="scenario-term-not-recognized-as-the-name-of-a-cmdlet-function-or-script"></a><a name="not-recognized-as-cmdlet"></a>Escenario: Término no reconocido como nombre de un cmdlet, función o script

### <a name="issue"></a>Incidencia

El runbook genera un error similar al ejemplo siguiente:

```error
The term 'Connect-AzAccount' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of the name, or if the path was included verify that the path is correct and try again.
```

### <a name="cause"></a>Causa

Este error puede deberse a los siguientes motivos:

* El módulo que contiene el cmdlet no está importado en la cuenta de Automation.
* El módulo que contiene el cmdlet está importado, pero no está actualizado.

### <a name="resolution"></a>Resolución

Realice una de las siguientes tareas para solucionar este error:

* En el caso de un módulo de Azure, consulte [Actualización de módulos de Azure PowerShell en Azure Automation](../automation-update-azure-modules.md) para obtener información sobre cómo actualizar los módulos en su cuenta de Automation.
* En el caso de un módulo que no es de Azure, asegúrese de que el módulo se importa en su cuenta de Automation.

## <a name="scenario-cmdlet-fails-in-pnp-powershell-runbook-on-azure-automation"></a>Escenario: Error del cmdlet en el runbook de PowerShell de PnP en Azure Automation

### <a name="issue"></a>Incidencia

Cuando un runbook escribe un objeto PnP generado por PowerShell directamente en la salida de Azure Automation, la salida del cmdlet no puede volver a transmitirse a Automation.

### <a name="cause"></a>Causa

Este problema normalmente se produce cuando Azure Automation procesa runbooks que invocan cmdlets de PowerShell de PnP; por ejemplo, `add-pnplistitem`, sin detectar los objetos devueltos.

### <a name="resolution"></a>Resolución

Edite los scripts para asignar los valores devueltos a variables de forma que los cmdlets no intenten escribir objetos completos en la salida estándar. Un script puede redirigir el flujo de salida a un cmdlet, como se muestra a continuación.

```azurecli
  $null = add-pnplistitem
```

Si el script analiza la salida del cmdlet, el script debe almacenar la salida en una variable y manipular la variable en lugar de simplemente transmitir la salida.

```azurecli
$SomeVariable = add-pnplistitem ....
if ($SomeVariable.someproperty -eq ....
```

## <a name="scenario-cmdlet-not-recognized-when-executing-a-runbook"></a><a name="cmdlet-not-recognized"></a>Escenario: No se reconoce el cmdlet cuando se ejecuta un runbook

### <a name="issue"></a>Incidencia

Se produce un error en el trabajo de runbook con el error:

```error
<cmdlet name>: The term <cmdlet name> is not recognized as the name of a cmdlet, function, script file, or operable program.
```

### <a name="cause"></a>Causa

Este error se produce cuando el motor de PowerShell no puede encontrar el cmdlet que está usando en su runbook. Es posible que el módulo que contiene el cmdlet no esté presente en la cuenta, que haya un conflicto de nombres con un nombre de runbook o que el cmdlet también exista en otro módulo y Automation no pueda resolver el nombre.

### <a name="resolution"></a>Resolución

Use cualquiera de las siguientes soluciones para solucionar el problema:

* Asegúrese de que ha especificado correctamente el nombre del cmdlet.
* Asegúrese de que el cmdlet exista en su cuenta de Automation y de que no haya ningún conflicto. Para comprobar si está presente el cmdlet, abra un runbook en modo de edición y busque el cmdlet que quiere encontrar en la biblioteca o ejecute `Get-Command <CommandName>`. Una vez que haya validado que el cmdlet está disponible para la cuenta y que no hay conflictos de nombres con otros cmdlets o runbooks, agregue el cmdlet al lienzo. Asegúrese de que usa un conjunto válido de parámetros en el runbook.
* Si tiene un conflicto de nombres y el cmdlet está disponible en dos módulos diferentes, use el nombre completo del cmdlet para resolver este problema. Por ejemplo, puede usar `ModuleName\CmdletName`.
* Si está ejecutando el runbook en el entorno local de un grupo de Hybrid Worker, asegúrese de que el cmdlet o el módulo están instalados en la máquina que hospeda la instancia de Hybrid Worker.

## <a name="scenario-incorrect-object-reference-on-call-to-add-azaccount"></a><a name="object-reference-not-set"></a>Escenario: Referencia de objeto incorrecta en la llamada a Add-AzAccount

### <a name="issue"></a>Incidencia

Este error aparece cuando se trabaja con `Add-AzAccount`, que es un alias del cmdlet `Connect-AzAccount`:

```error
Add-AzAccount : Object reference not set to an instance of an object
```

### <a name="cause"></a>Causa

Este error puede producirse si el runbook no realiza los pasos adecuados antes de llamar a `Add-AzAccount` para agregar la cuenta de Automation. Un ejemplo de uno de los pasos necesarios es iniciar sesión con una cuenta de ejecución. Para ver las operaciones correctas que se deben usar en el runbook, consulte [Ejecución de un runbook en Azure Automation](../automation-runbook-execution.md).

## <a name="scenario-object-reference-not-set-to-an-instance-of-an-object"></a><a name="child-runbook-object"></a>Escenario: Referencia a objeto no establecida como instancia de un objeto

### <a name="issue"></a>Incidencia

Recibe el siguiente error cuando invoca un runbook secundario con el parámetro `Wait` y el flujo de salida contiene un objeto:

```error
Object reference not set to an instance of an object
```

### <a name="cause"></a>Causa

Si el flujo contiene objetos, significa que `Start-AzAutomationRunbook` no controla el flujo de salida correctamente.

### <a name="resolution"></a>Resolución

Implemente una lógica de sondeo y use el cmdlet [Get-AzAutomationJobOutput](/powershell/module/Az.Automation/Get-AzAutomationJobOutput?view=azps-3.7.0) para recuperar la salida. Aquí se define un ejemplo de esta lógica:

```powershell
$automationAccountName = "ContosoAutomationAccount"
$runbookName = "ChildRunbookExample"
$resourceGroupName = "ContosoRG"

function IsJobTerminalState([string] $status) {
    return $status -eq "Completed" -or $status -eq "Failed" -or $status -eq "Stopped" -or $status -eq "Suspended"
}

$job = Start-AzAutomationRunbook -AutomationAccountName $automationAccountName -Name $runbookName -ResourceGroupName $resourceGroupName
$pollingSeconds = 5
$maxTimeout = 10800
$waitTime = 0
while($false -eq (IsJobTerminalState $job.Status) -and $waitTime -lt $maxTimeout) {
   Start-Sleep -Seconds $pollingSeconds
   $waitTime += $pollingSeconds
   $job = $job | Get-AzAutomationJob
}

$job | Get-AzAutomationJobOutput | Get-AzAutomationJobOutputRecord | Select-Object -ExpandProperty Value
```

## <a name="scenario-runbook-fails-because-of-deserialized-object"></a><a name="fails-deserialized-object"></a>Escenario: Error en runbook debido a un objeto deserializado

### <a name="issue"></a>Incidencia

Se produce un error en el runbook con el error:

```error
Cannot bind parameter <ParameterName>.

Cannot convert the <ParameterType> value of type Deserialized <ParameterType> to type <ParameterType>.
```

### <a name="cause"></a>Causa

Si el runbook es un flujo de trabajo de PowerShell, almacena objetos complejos en un formato deserializado para conservar el estado del Runbook si se suspende el flujo de trabajo.

### <a name="resolution"></a>Resolución

Use cualquiera de las siguientes soluciones para corregir este problema:

* Si canaliza objetos complejos de un cmdlet a otro, encapsule dichos cmdlets en una actividad `InlineScript`.
* En lugar de pasar el objeto complejo entero, pase solamente el nombre o valor del mismo que necesite.
* Use un runbook de PowerShell en lugar de un runbook de flujo de trabajo de PowerShell.

## <a name="scenario-400-bad-request-status-when-calling-a-webhook"></a><a name="expired webhook"></a>Escenario: Estado de solicitud incorrecta 400 al llamar a un webhook

### <a name="issue"></a>Incidencia

Cuando intenta invocar un webhook para un runbook de Azure Automation, recibe el siguiente error:

```error
400 Bad Request : This webhook has expired or is disabled
```

### <a name="cause"></a>Causa

El webhook al que intenta llamar está deshabilitado o ha expirado. 

### <a name="resolution"></a>Resolución

Si el webhook está deshabilitado, puede volver a habilitarlo mediante Azure Portal. Si el webhook expiró, debe eliminarlo y volver a crearlo. Solo puede [renovar un webhook](../automation-webhooks.md#renew-a-webhook) si aún no ha expirado. 

## <a name="scenario-429-the-request-rate-is-currently-too-large"></a><a name="429"></a>Escenario: 429: Actualmente la tasa de solicitud es demasiado grande

### <a name="issue"></a>Incidencia

Recibe el siguiente mensaje de error al ejecutar el cmdlet `Get-AzAutomationJobOutput`:

```error
429: The request rate is currently too large. Please try again
```

### <a name="cause"></a>Causa

Este error puede producirse al recuperar la salida de trabajo de un runbook que tiene muchos [flujos detallados](../automation-runbook-output-and-messages.md#monitor-verbose-stream).

### <a name="resolution"></a>Resolución

Realice una de las siguientes acciones para solucionar este error:

* Edite el runbook y reduzca el número de flujos de trabajo que emite.
* Reduzca el número de flujos para recuperar cuando se ejecuta el cmdlet. Para ello, puede establecer el valor del parámetro `Stream` del cmdlet [Get-AzAutomationJobOutput](/powershell/module/Az.Automation/Get-AzAutomationJobOutput?view=azps-3.7.0) para que recupere solo los flujos de salida. 

## <a name="scenario-runbook-job-fails-because-allocated-quota-was-exceeded"></a><a name="quota-exceeded"></a>Escenario: Error en el trabajo del runbook porque se superó la cuota asignada

### <a name="issue"></a>Incidencia

Se produce un error en el trabajo de runbook con el error:

```error
The quota for the monthly total job run time has been reached for this subscription
```

### <a name="cause"></a>Causa

Este error se produce cuando la ejecución del trabajo supera la cuota gratuita de 500 minutos para su cuenta. Esta cuota se aplica a todos los tipos de tareas de ejecución de trabajos. Algunas de estas tareas son probar un trabajo, iniciar un trabajo desde el portal, ejecutar un trabajo mediante webhooks o programar un trabajo para que se ejecute mediante Azure Portal o el centro de datos. Para obtener más información sobre precios para, consulte [Precios de Automation](https://azure.microsoft.com/pricing/details/automation/).

### <a name="resolution"></a>Resolución

Si quiere usar más de 500 minutos de procesamiento por mes cambie la suscripción del nivel Gratis al nivel Básico:

1. Inicie sesión en la suscripción de Azure.
1. Seleccione la cuenta de Automation que quiera actualizar.
1. Seleccione **Settings** (Configuración) y luego **Pricing** (Precio).
1. Haga clic en **Enable** (Habilitar) en la parte inferior de la página para actualizar la cuenta al nivel Basic (Básico).

## <a name="scenario-runbook-output-stream-greater-than-1-mb"></a><a name="output-stream-greater-1mb"></a>Escenario: Flujo de salida de runbook superior a 1 MB

### <a name="issue"></a>Incidencia

Se produce el siguiente error en el runbook que se ejecuta en el espacio aislado de Azure:

```error
The runbook job failed due to a job stream being larger than 1MB, this is the limit supported by an Azure Automation sandbox.
```

### <a name="cause"></a>Causa

Este error se produce porque el runbook intentó escribir demasiados datos de excepción en el flujo de salida.

### <a name="resolution"></a>Resolución

Hay un límite de 1 MB en el flujo de salida del trabajo. Asegúrese de que el runbook incluye las llamadas a un archivo ejecutable o a un subproceso mediante bloques `try` y `catch`. Si las operaciones producen una excepción, haga que el código escriba el mensaje de la excepción en una variable de Automation. Esta técnica impedirá que el mensaje se escriba en el flujo de salida del trabajo. En el caso de trabajos ejecutados de Hybrid Runbook Worker, el flujo de salida truncado en 1 MB se muestra sin ningún mensaje de error.

## <a name="scenario-runbook-job-start-attempted-three-times-but-fails-to-start-each-time"></a><a name="job-attempted-3-times"></a>Escenario: Se intentó iniciar el trabajo del runbook tres veces, pero en las tres se produjo un error

### <a name="issue"></a>Incidencia

Se produce el siguiente error en el runbook:

```error
The job was tried three times but it failed
```

### <a name="cause"></a>Causa

Este error se produce debido a uno de los siguientes problemas:

* **Límite de memoria.** Un trabajo puede producir un error si emplea más de 400 MB de memoria. Los límites documentados sobre la memoria que se asigna a un espacio aislado se encuentran en [Límites del servicio Automation](../../azure-resource-manager/management/azure-subscription-service-limits.md#automation-limits). 

* **Sockets de red.** Los espacios aislados de Azure están limitados a 1000 sockets de red simultáneos. Para más información, consulte [Límites de servicio de Automation](../../azure-resource-manager/management/azure-subscription-service-limits.md#automation-limits).

* **Módulo incompatible.** Es posible que las dependencias del módulo no sean correctas. En este caso, el runbook suele devolver el mensaje `Command not found` o `Cannot bind parameter`.

* **No se realizó ninguna autenticación con Active Directory de los espacios aislados.** El runbook intentó llamar a un archivo ejecutable o a un subproceso que se ejecuta en un espacio aislado de Azure. No se admite la configuración de runbooks para autenticarse con Azure AD mediante la Biblioteca de autenticación de Azure Active Directory (ADAL).

### <a name="resolution"></a>Resolución

* **Límite de memoria, sockets de red.** Algunas formas sugeridas para trabajar dentro del límite de memoria son dividir la carga de trabajo entre varios runbooks, procesar menos datos en la memoria, evitar escribir resultados innecesarios de los runbooks y considerar cuántos puntos de control se escriben en los runbooks de flujo de trabajo de PowerShell. Use el método clear, como `$myVar.clear`, para borrar las variables y utilice `[GC]::Collect` para ejecutar inmediatamente la recolección de elementos no utilizados. Estas acciones reducen la superficie de memoria de su runbook en tiempo de ejecución.

* **Módulo incompatible.** Actualice los módulos de Azure siguiendo los pasos en [Actualización de módulos de Azure PowerShell en Azure Automation](../automation-update-azure-modules.md).

* **No se realizó ninguna autenticación con Active Directory de los espacios aislados.** Al autenticarse en Azure AD con un runbook, asegúrese de que el módulo de Azure AD está disponible en la cuenta de Automation. Asegúrese de conceder a la cuenta de ejecución los permisos necesarios para realizar las tareas que el runbook automatiza.

  Si el runbook no puede llamar a un archivo ejecutable o a un subproceso que se ejecuta en un espacio aislado de Azure, use el runbook en una instancia de [Hybrid Runbook Worker](../automation-hrw-run-runbooks.md). Los roles de Hybrid Worker no están limitados por los límites de memoria y de red como lo están los espacios aislados de Azure.

## <a name="scenario-powershell-job-fails-with-cannot-invoke-method-error-message"></a><a name="cannot-invoke-method"></a>Escenario: Error en el trabajo de PowerShell con el mensaje de error "No se puede invocar el método"

### <a name="issue"></a>Incidencia

Al iniciar un trabajo de PowerShell en un runbook que se ejecuta en Azure, recibe el siguiente mensaje de error:

```error
Exception was thrown - Cannot invoke method. Method invocation is supported only on core types in this language mode.
```

### <a name="cause"></a>Causa

Este error puede indicar que los runbooks que se ejecutan en un espacio aislado de Azure no se pueden ejecutar en el [modo de lenguaje completo](/powershell/module/microsoft.powershell.core/about/about_language_modes).

### <a name="resolution"></a>Resolución

Hay dos maneras de resolver este error:

* En lugar de usar [Start-Job](/powershell/module/microsoft.powershell.core/start-job?view=powershell-7), use [Start-AzAutomationRunbook](/powershell/module/az.automation/start-azautomationrunbook?view=azps-3.7.0) para iniciar el runbook.
* Intente ejecutar el runbook en una instancia de Hybrid Runbook Worker.

Para más información sobre este comportamiento y otros comportamientos de los runbooks de Azure Automation, consulte [Ejecución de un runbook en Azure Automation](../automation-runbook-execution.md).

## <a name="scenario-a-long-running-runbook-fails-to-complete"></a><a name="long-running-runbook"></a>Escenario: Un runbook de larga ejecución no se puede completar

### <a name="issue"></a>Incidencia

El runbook muestra un estado Detenido después de ejecutarse durante tres horas. También se muestra este error:

```error
The job was evicted and subsequently reached a Stopped state. The job cannot continue running.
```

Este comportamiento es así por naturaleza en los espacios aislados de Azure debido a la supervisión de [distribución equilibrada](../automation-runbook-execution.md#fair-share) de los procesos en Azure Automation. Si un proceso se ejecuta durante más de tres horas, la distribución equilibrada detiene automáticamente el runbook. El estado de un runbook que va más allá del límite de tiempo de distribución equilibrada varía según el tipo de runbook. Runbooks de PowerShell y Python se establecen en un estado Detenido. Los runbooks del flujo de trabajo de PowerShell se establecen en Error.

### <a name="cause"></a>Causa

El runbook se ejecutó por encima del límite de tres horas permitido por la distribución equilibrada en un espacio aislado de Azure.

### <a name="resolution"></a>Resolución

La solución recomendada consiste en ejecutar el runbook en un [Hybrid Runbook Worker](../automation-hrw-run-runbooks.md). Los roles Hybrid Worker no tienen el límite de tres horas de los runbooks de distribución equilibrada que tienen los espacios aislados de Azure. Los runbooks que se ejecutan en instancias de Hybrid Runbook Worker se deben desarrollar para admitir comportamientos de reinicio si existen problemas inesperados con la infraestructura local.

Otra solución consiste en optimizar el runbook mediante la creación de [runbooks secundarios](../automation-child-runbooks.md). Si el runbook recorre en bucle la misma función en varios recursos (por ejemplo, una operación de base de datos en varias bases de datos), la función se puede mover a un runbook secundario. Cada runbook secundario se ejecuta en paralelo en un proceso independiente. Este comportamiento reduce la cantidad total de tiempo que tarda en completarse el runbook primario.

Los cmdlets de PowerShell que habilitan el escenario de runbook secundario son:

* [Start-AzAutomationRunbook](/powershell/module/Az.Automation/Start-AzAutomationRunbook?view=azps-3.7.0). este cmdlet permite iniciar un runbook y pasar parámetros al mismo.
* [Get-AzAutomationJob](/powershell/module/Az.Automation/Get-AzAutomationJob?view=azps-3.7.0). Si hay operaciones que deben realizarse después de que se complete el runbook secundario, este cmdlet permite comprobar el estado del trabajo de cada elemento secundario.

## <a name="scenario-error-in-job-streams-about-the-get_serializationsettings-method"></a><a name="get-serializationsettings"></a>Escenario: Error en los flujos de trabajos sobre el método get_SerializationSettings

### <a name="issue"></a>Incidencia

Se muestra el siguiente error en sus flujos de trabajo para un runbook:

```error
Connect-AzAccount : Method 'get_SerializationSettings' in type
'Microsoft.Azure.Management.Internal.Resources.ResourceManagementClient' from assembly
'Microsoft.Azure.Commands.ResourceManager.Common, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35'
does not have an implementation.
At line:16 char:1
+ Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -Appl ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Connect-AzAccount], TypeLoadException
    + FullyQualifiedErrorId : System.TypeLoadException,Microsoft.Azure.Commands.Profile.ConnectAzAccountCommand
```

### <a name="cause"></a>Causa

Este error probablemente se deba a una migración incompleta de los módulos AzureRM a los módulos Az en el runbook. Esta situación puede hacer que Azure Automation inicie un trabajo de runbook solo con los módulos de AzureRM y, luego, otro trabajo solo con los módulos Az, lo que lleva a un bloqueo del espacio aislado.

### <a name="resolution"></a>Resolución

No se recomienda el uso de los cmdlets Az y AzureRM en el mismo runbook. Para obtener más información sobre el uso correcto de los módulos, consulte [Migración a los módulos Az](../shared-resources/modules.md#migrate-to-az-modules).

## <a name="scenario-access-denied-when-using-azure-sandbox-for-runbook-or-application"></a><a name="access-denied-azure-sandbox"></a>Escenario: Acceso denegado al usar el espacio aislado de Azure para un runbook o una aplicación

### <a name="issue"></a>Incidencia

Cuando el runbook o la aplicación intentan ejecutarse en un espacio aislado de Azure, el entorno deniega el acceso.

### <a name="cause"></a>Causa

Este problema puede producirse porque los espacios aislados de Azure impiden el acceso a todos los servidores COM fuera de proceso. Por ejemplo, una aplicación en espacio aislado o un runbook no pueden llamar a Instrumental de administración de Windows (WMI) o al servicio Windows Installer (msiserver.exe). 

### <a name="resolution"></a>Resolución

Para obtener detalles sobre el uso de espacios aislados de Azure, consulte [Entorno de ejecución de un runbook](../automation-runbook-execution.md#runbook-execution-environment).

## <a name="scenario-invalid-forbidden-status-code-when-using-key-vault-inside-a-runbook"></a>Escenario: Código de estado no válido "Prohibido" al usar Key Vault dentro de un runbook

### <a name="issue"></a>Incidencia

Al intentar acceder a Azure Key Vault mediante un runbook de Azure Automation, se produce el siguiente error:

```error
Operation returned an invalid status code 'Forbidden'
```

### <a name="cause"></a>Causa

Las posibles causas de este problema son:

* No usar una cuenta de ejecución.
* Permisos insuficientes.

### <a name="resolution"></a>Resolución

#### <a name="not-using-a-run-as-account"></a>No usar una cuenta de ejecución.

Siga el [Paso 5: Agregar autenticación para administrar recursos de Azure](../learn/automation-tutorial-runbook-textual-powershell.md#step-5---add-authentication-to-manage-azure-resources) para asegurarse de que usa una cuenta de ejecución para acceder a Key Vault.

#### <a name="insufficient-permissions"></a>Permisos insuficientes

[Agregue permisos a Key Vault](../manage-runas-account.md#add-permissions-to-key-vault) para asegurarse de que la cuenta de ejecución tenga suficientes permisos para acceder a Key Vault.

## <a name="recommended-documents"></a>Documentos recomendados

* [Ejecución de un runbook en Azure Automation](../automation-runbook-execution.md)
* [Inicio de un runbook en Azure Automation](../start-runbooks.md)

## <a name="next-steps"></a>Pasos siguientes

Si su problema no aparece o es incapaz de resolverlo, pruebe uno de los siguientes canales para obtener soporte técnico adicional:

* Obtenga respuestas de expertos de Azure en los [foros de Azure](https://azure.microsoft.com/support/forums/).
* Póngase en contacto con [@AzureSupport](https://twitter.com/azuresupport), la cuenta oficial de Microsoft Azure para mejorar la experiencia del cliente. El servicio de soporte técnico de Azure le pone en contacto con la comunidad de Azure donde encontrará respuestas, soporte técnico y expertos.
* Si necesita más ayuda, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte técnico**.
