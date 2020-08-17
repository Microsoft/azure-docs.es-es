---
title: Creación de un runbook de PowerShell en Azure Automation
description: Este artículo le enseña a crear, probar y publicar un sencillo runbook de PowerShell.
keywords: powershell de Azure, tutorial de scripts de powershell, automatización de powershell
services: automation
ms.subservice: process-automation
ms.date: 04/19/2020
ms.topic: tutorial
ms.openlocfilehash: a1b0dff9421f493958554c659043c49ff2874379
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87015007"
---
# <a name="tutorial-create-a-powershell-runbook"></a>Tutorial: Creación de un runbook de PowerShell

Este tutorial le guiará por la creación de un [Runbook de PowerShell](../automation-runbook-types.md#powershell-runbooks) en Azure Automation. Los runbooks de PowerShell están basados en Windows PowerShell. Puede modificar directamente el código del runbook con el editor de texto en Azure Portal.

> [!div class="checklist"]
> * Creación de un runbook de PowerShell simple
> * Prueba y publicación del runbook
> * Ejecución y seguimiento del estado del trabajo de runbook
> * Actualización del runbook para iniciar una máquina virtual de Azure con parámetros de runbook

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesitará lo siguiente:

* Suscripción de Azure. Si aún no tiene ninguna, puede [activar las ventajas de la suscripción a MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) o suscribirse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Cuenta de Automation](../automation-quickstart-create-account.md) para contener el Runbook y autenticarse en recursos de Azure. Esta cuenta debe tener permiso para iniciar y detener la máquina virtual.
* Una máquina virtual de Azure. Dado que va a detener e iniciar esta máquina, no debería ser una máquina virtual de producción.
* Si es necesario, [importe módulos de Azure](../shared-resources/modules.md) o [actualice los módulos](../automation-update-azure-modules.md) en función de los cmdlets que utilice.

## <a name="differences-from-powershell-workflow-runbooks"></a>Diferencias con los runbooks del flujo de trabajo de PowerShell

Los runbooks de PowerShell tienen el mismo ciclo de vida, las mismas funcionalidades y la misma administración que los runbooks de flujo de trabajo de PowerShell. Sin embargo, existen algunas diferencias y limitaciones.

| Característica  | Runbooks de PowerShell | Runbooks del flujo de trabajo de PowerShell |
| ------ | ----- | ----- |
| Velocidad | Se ejecutan rápidamente, ya que no usan un paso de compilación. | Se ejecutan más lentamente. |
| Puntos de control | No admiten puntos de control. Un runbook de PowerShell solo puede reanudar el funcionamiento desde el principio. | Use puntos de control, que permiten que un libro reanude el funcionamiento desde cualquier punto. |
| Ejecución de comando | Solo admite la ejecución en serie. | Admite la ejecución en serie y en paralelo.|
| Espacio de ejecución | Un espacio de ejecución único lo ejecuta todo en un script. | Se puede usar un espacio de ejecución independiente para una actividad, un comando o un bloque de script. |

Además de estas diferencias, los runbooks de PowerShell presentan algunas [diferencias sintácticas](/previous-versions/technet-magazine/dn151046(v=msdn.10)) con los runbooks del flujo de trabajo de PowerShell.

## <a name="step-1---create-runbook"></a>Paso 1: Creación del runbook

Empiece por crear un runbook simple que genere el texto `Hello World`.

1. En Azure Portal, abra su cuenta de Automation.

2. En **Automatización de procesos**, haga clic en **Runbooks** para abrir la lista de runbooks.

3. Seleccione **Crear un Runbook** para crear uno.

4. Asigne al Runbook el nombre **MyFirstRunbook-PowerShell**.

5. En este caso, va a crear un [runbook de PowerShell](../automation-runbook-types.md#powershell-runbooks). Seleccione **PowerShell** en **Tipo de Runbook**.

6. Haga clic en **Crear** para crear el runbook y abra el editor de texto.

## <a name="step-2---add-code-to-the-runbook"></a>Paso 2: Incorporación de código al runbook

Puede escribir el código directamente en el runbook o seleccionar los cmdlets, runbooks y recursos desde el control Biblioteca y agregarlos al runbook con los parámetros relacionados. En este tutorial, escribirá el código directamente en el runbook.

1. Su runbook está vacío actualmente. Escriba `Write-Output "Hello World"` en el cuerpo del script.

   ![Hola mundo](../media/automation-tutorial-runbook-textual-powershell/automation-helloworld.png)

2. Guarde el Runbook, para lo que debe hacer clic en **Guardar**.

## <a name="step-3---test-the-runbook"></a><a name="step-3---test-the-runbook"> </a> Paso 3: Prueba del runbook

Antes de publicar el runbook para que esté disponible en producción, debe probarlo para asegurarse de que funciona correctamente. La prueba de un runbook ejecuta su versión de borrador y permite ver la salida de forma interactiva.

1. Haga clic en **Panel Prueba** para abrir el panel de prueba.

2. Haga clic en **Iniciar** para iniciar la prueba. Esta debe ser la única opción habilitada.

3. Observe que se crea un [trabajo de runbook](../automation-runbook-execution.md) y su estado se muestra en el panel.

   El estado del trabajo se inicia como En cola, que indica que está esperando a que haya disponible un trabajo de runbook en la nube. El estado cambia a Iniciando cuando un trabajo lo solicita. Por último, el estado pasa a En ejecución cuando el runbook comienza a ejecutarse.

4. Cuando se completa el trabajo del runbook, el panel de prueba muestra su salida. En este caso, verá `Hello World`.

   ![Salida del panel de prueba](../media/automation-tutorial-runbook-textual-powershell/automation-testpane-output.png)

5. Cierre el panel Prueba para volver al lienzo.

## <a name="step-4---publish-and-start-the-runbook"></a>Paso 4: publicar e iniciar el runbook

El runbook que ha creado aún está en modo de borrador. Debe publicarse antes de ejecutarlo en producción. Al publicar un runbook, se sobrescribe la versión publicada existente con la versión de borrador. En este caso, no tiene una versión publicada aún porque acaba de crear el runbook.

1. Haga clic en **Publicar** para publicar el runbook y en **Sí** cuando se le solicite.

2. Desplácese hacia la izquierda para ver el runbook en la página Runbooks y observe que el valor de **Estado de creación** está establecido en **Publicado**.

3. Desplácese de nuevo a la derecha para ver la página de **MyFirstRunbook-PowerShell**.
   
   Las opciones que se encuentran en la parte superior permiten iniciar el runbook ahora, programar una hora de inicio futura o crear un [webhook](../automation-webhooks.md) para que el runbook pueda iniciarse mediante una llamada HTTP.

4. Para iniciar el runbook, seleccione **Iniciar** y **Sí** cuando se le solicite. 

5. Se abre un panel Trabajo para el trabajo de runbook que se ha creado. Aunque puede cerrar este panel, déjelo abierto para poder ver el progreso del trabajo. El estado del trabajo se muestra en **Resumen del trabajo**, y los posibles estados son los que se describen para probar el runbook.

   ![Resumen del trabajo](../media/automation-tutorial-runbook-textual-powershell/job-pane-status-blade-jobsummary.png)

6. Cuando el estado del runbook sea Completado, haga clic en **Salida** para abrir la página de resultados, donde podrá ver `Hello World`.

   ![Salida del trabajo](../media/automation-tutorial-runbook-textual-powershell/job-pane-status-blade-outputtile.png)

7. Cierre la página Salida.

8. Haga clic en **Todos los registros** para abrir el panel Transmisiones para el trabajo de Runbook. Solo debería ver `Hello World` en el flujo de salida.

    Tenga en cuenta que el panel Transmisiones puede mostrar otras transmisiones de un trabajo de runbook, como las de error y detalladas, si el runbook escribe en ellas.

   ![Todos los registros](../media/automation-tutorial-runbook-textual-powershell/job-pane-status-blade-alllogstile.png)

9. Cierre el panel Transmisiones y el panel Trabajo para volver a la página MyFirstRunbook-PowerShell.

10. En **Detalles**, haga clic en **Trabajos** para abrir la página Trabajos de este runbook. Esta página enumera todos los trabajos creados por el runbook. Solo debería ver un trabajo en la lista ya que solo ejecutó el trabajo una vez.

   ![Lista de trabajos](../media/automation-tutorial-runbook-textual-powershell/runbook-control-job-tile.png)

11. Haga clic en el nombre del trabajo para abrir el mismo panel Trabajo que vio cuando inició el runbook. Use este panel para ver los detalles de cualquier trabajo creado para el runbook.

## <a name="step-5---add-authentication-to-manage-azure-resources"></a>Paso 5: agregar autenticación para administrar recursos de Azure

Ha probado y publicado su runbook, pero hasta ahora no hace nada útil. Quiere que administre recursos de Azure. Para ello, el runbook debe poder autenticarse con la cuenta de ejecución que se creó automáticamente al crear la cuenta de Automation.

Como se muestra en el ejemplo siguiente, la conexión de ejecución se realiza con el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount?view=azps-3.5.0). Si administra recursos en varias suscripciones, debe usar el parámetro `AzContext` con [Get-AzContext](/powershell/module/Az.Accounts/Get-AzContext?view=azps-3.5.0).

> [!NOTE]
> En el caso de los runbooks de PowerShell, `Add-AzAccount` y `Add-AzureRMAccount` son alias de `Connect-AzAccount`. Puede usar estos cmdlets o bien [actualizar los módulos](../automation-update-azure-modules.md) de la cuenta de Automation a las versiones más recientes. Es posible que deba actualizar los módulos incluso si acaba de crear una nueva cuenta de Automation.

   ```powershell
   # Ensures you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave –Scope Process

   $connection = Get-AutomationConnection -Name AzureRunAsConnection

   # Wrap authentication in retry logic for transient network failures
   $logonAttempt = 0
   while(!($connectionResult) -and ($logonAttempt -le 10))
   {
       $LogonAttempt++
       # Logging in to Azure...
       $connectionResult = Connect-AzAccount `
                               -ServicePrincipal `
                               -Tenant $connection.TenantID `
                               -ApplicationId $connection.ApplicationID `
                               -CertificateThumbprint $connection.CertificateThumbprint

       Start-Sleep -Seconds 30
   }

   $AzureContext = Get-AzSubscription -SubscriptionId $connection.SubscriptionID

   Get-AzVM -ResourceGroupName myResourceGroup -AzContext $AzureContext
   ```

1. Haga clic en **Editar** en la página MyFirstRunbook-PowerShell para abrir el editor de texto.

2. Ya no necesita la línea `Write-Output`. Simplemente continúe y elimínela.

3. Escriba o copie y pegue el siguiente código, que controla la autenticación con la cuenta de ejecución de Automation.

   ```powershell
   # Ensures you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave –Scope Process

   $connection = Get-AutomationConnection -Name AzureRunAsConnection

   while(!($connectionResult) -and ($logonAttempt -le 10))
   {
       $LogonAttempt++
       # Logging in to Azure...
       $connectionResult = Connect-AzAccount `
                               -ServicePrincipal `
                               -Tenant $connection.TenantID `
                               -ApplicationId $connection.ApplicationID `
                               -CertificateThumbprint $connection.CertificateThumbprint

       Start-Sleep -Seconds 30
   }
   ```

4. Haga clic en el **panel de prueba** para poder probar el runbook.

5. Haga clic en **Iniciar** para iniciar la prueba. Cuando termine, verá una salida similar a la siguiente, que muestra información básica de su cuenta. Esta salida confirma que la cuenta de ejecución es válida.

   ![Authenticate](../media/automation-tutorial-runbook-textual-powershell/runbook-auth-output.png)

## <a name="step-6---add-code-to-start-a-virtual-machine"></a>Paso 6: Incorporación de una actividad para iniciar una máquina virtual

Ahora que el runbook está autenticado en la suscripción a Azure, puede administrar los recursos. Agreguemos un comando para iniciar una máquina virtual. Puede seleccionar cualquier máquina virtual de la suscripción de Azure y, por ahora, limitarse a codificar ese nombre de forma rígida en el runbook.

1. En el script del runbook, agregue el cmdlet [Start-AzVM](/powershell/module/Az.Compute/Start-AzVM?view=azps-3.5.0) para iniciar la máquina virtual. Como se muestra a continuación, el cmdlet inicia una máquina virtual con el nombre `VMName` y con un grupo de recursos denominado `ResourceGroupName`.

   ```powershell
   # Ensures you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave –Scope Process

   $connection = Get-AutomationConnection -Name AzureRunAsConnection
   while(!($connectionResult) -and ($logonAttempt -le 10))
   {
       $LogonAttempt++
       # Logging in to Azure...
       $connectionResult = Connect-AzAccount `
                               -ServicePrincipal `
                               -Tenant $connection.TenantID `
                               -ApplicationId $connection.ApplicationID `
                               -CertificateThumbprint $connection.CertificateThumbprint

       Start-Sleep -Seconds 30
   }

   Start-AzVM -Name 'VMName' -ResourceGroupName 'ResourceGroupName'
   ```

2. Guarde el runbook y haga clic en **Panel de prueba** para probarlo.

3. Haga clic en **Iniciar** para iniciar la prueba. Una vez que se complete, asegúrese de que la máquina virtual se ha iniciado.

## <a name="step-7---add-an-input-parameter"></a>Paso 7: Incorporación de un parámetro de entrada

Actualmente, el runbook inicia la máquina virtual que ha codificado de forma rígida. El runbook será más útil si, al iniciarlo, especifica la máquina virtual. Agreguemos parámetros de entrada al runbook para incluir esta funcionalidad.

1. En el editor de texto, modifique el cmdlet `Start-AzVM` para usar variables de los parámetros `VMName` y `ResourceGroupName`. 

   ```powershell
   Param(
    [string]$VMName,
    [string]$ResourceGroupName
   )
   # Ensures you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave –Scope Process

   $connection = Get-AutomationConnection -Name AzureRunAsConnection
   while(!($connectionResult) -and ($logonAttempt -le 10))
   {
       $LogonAttempt++
       # Logging in to Azure...
       $connectionResult = Connect-AzAccount `
                               -ServicePrincipal `
                               -Tenant $connection.TenantID `
                               -ApplicationId $connection.ApplicationID `
                               -CertificateThumbprint $connection.CertificateThumbprint

       Start-Sleep -Seconds 30
   }

   Start-AzVM -Name $VMName -ResourceGroupName $ResourceGroupName
   ```

2. Guarde el runbook y abra el panel Prueba. Ahora puede proporcionar valores para las dos variables de entrada que se usan en la prueba.

3. Cierre el panel Prueba.

4. Haga clic en **Publicar** para publicar la nueva versión del runbook.

5. Detenga la máquina virtual que inició anteriormente.

6. Haga clic en **Iniciar** para iniciar el runbook. 

7. Escriba los valores de **VMNAME** y **RESOURCEGROUPNAME** para la máquina virtual que va a iniciar y, a continuación, haga clic en **Aceptar**.

    ![Pasar parámetro](../media/automation-tutorial-runbook-textual-powershell/automation-pass-params.png)

8. Cuando se complete el runbook, asegúrese de que se ha iniciado la máquina virtual.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre PowerShell, incluidos los módulos de referencia de lenguaje y aprendizaje, consulte la [documentación de PowerShell](/powershell/scripting/overview).
* Para ver una referencia de los cmdlets de PowerShell, consulte [Az.Automation](/powershell/module/az.automation/?view=azps-3.7.0#automation).
* Para empezar a trabajar con runbooks gráficos, consulte [Creación de un runbook gráfico](automation-tutorial-runbook-graphical.md).
* Para empezar a trabajar con runbooks de flujo de trabajo de PowerShell, consulte [Creación de un runbook de flujo de trabajo de PowerShell](automation-tutorial-runbook-textual.md).
* Para más información sobre los tipos de runbook, sus ventajas y sus limitaciones, consulte [Tipos de runbooks de Azure Automation](../automation-runbook-types.md).
* Para obtener más información sobre la característica de compatibilidad con scripts de PowerShell, consulte [Anuncio de la compatibilidad nativa con scripts de PowerShell en Azure Automation](https://azure.microsoft.com/blog/announcing-powershell-script-support-azure-automation-2/).
