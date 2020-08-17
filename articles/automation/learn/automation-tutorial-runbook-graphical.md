---
title: Creación de un runbook gráfico en Azure Automation
description: Este artículo le enseña a crear, probar y publicar un sencillo runbook gráfico en Azure Automation.
keywords: runbook, plantilla de runbook, automatización de runbooks, runbook de azure
services: automation
ms.subservice: process-automation
ms.date: 04/19/2020
ms.topic: tutorial
ms.openlocfilehash: fa92cd4f4cba63eee09714813954af9fa9c9c4ea
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87290954"
---
# <a name="tutorial-create-a-graphical-runbook"></a>Tutorial: Creación de un runbook gráfico

Este tutorial le guiará por la creación de un [runbook gráfico](../automation-runbook-types.md#graphical-runbooks) en Azure Automation. Puede crear y editar runbooks gráficos y runbooks gráficos de flujo de trabajo de PowerShell mediante el editor gráfico de Azure Portal. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un runbook gráfico simple
> * Prueba y publicación del runbook
> * Ejecutar y seguir el estado del trabajo de runbook
> * Actualización del runbook para iniciar una máquina virtual de Azure, con parámetros de runbook y vínculos condicionales

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesitará lo siguiente:

* Suscripción de Azure. Si aún no tiene ninguna, puede [activar las ventajas de la suscripción a MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) o suscribirse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Cuenta de Automation](../index.yml) para contener el Runbook y autenticarse en recursos de Azure. Esta cuenta debe tener permiso para iniciar y detener la máquina virtual.
* Una máquina virtual de Azure. Dado que va a detener e iniciar esta máquina, no debería ser una máquina virtual de producción.
* Si es necesario, [importe módulos de Azure](../shared-resources/modules.md) o [actualice los módulos](../automation-update-azure-modules.md) en función de los cmdlets que utilice.

## <a name="step-1---create-runbook"></a>Paso 1: Creación del runbook

Empiece por crear un runbook simple que genere el texto `Hello World`.

1. En Azure Portal, abra su cuenta de Automation. 

    La página de la cuenta de Automation proporciona una vista rápida de los recursos que hay en esa cuenta. Ya debería tener algunos recursos. La mayoría son los módulos que se incluyen automáticamente en una cuenta nueva de Automation. También debe tener el recurso de credencial asociado a su suscripción.

2. En **Automatización de procesos**, haga clic en **Runbooks** para abrir la lista de runbooks.

3. Seleccione **Crear un Runbook** para crear uno.

4. Asigne al runbook el nombre **MyFirstRunbook-Graphical**.

5. En este caso, va a crear un [runbook gráfico](../automation-graphical-authoring-intro.md). Seleccione **Gráfico** en **Tipo de Runbook**.<br> ![Nuevo runbook](../media/automation-tutorial-runbook-graphical/create-new-runbook.png)<br>

6. Haga clic en **Crear** para crear el runbook y abrir el editor gráfico.

## <a name="step-2---add-activities"></a>Paso 2: Incorporación de actividades

El control Biblioteca en la parte izquierda del editor permite seleccionar actividades para agregarlas al Runbook. Va a agregar un cmdlet `Write-Output` para generar el texto desde el runbook.

1. En el control Biblioteca, haga clic en el campo de búsqueda y escriba `write-output`. Los resultados de la búsqueda se muestran en la siguiente imagen. <br> ![Microsoft.PowerShell.Utility](../media/automation-tutorial-runbook-graphical/search-powershell-cmdlet-writeoutput.png)

2. Desplácese hasta la parte inferior de la lista. Haga clic con el botón derecho en **Write-Output** y seleccione **Agregar al lienzo**. También puede hacer clic en los puntos suspensivos (...) junto al nombre del cmdlet y seleccionar **Agregar al lienzo**.

3. Haga clic en la actividad **Write-Output** en el lienzo. Esta acción abre la página Control de configuración, que le permite configurar la actividad.

4. El campo **Etiqueta** tiene como valor predeterminado el nombre del cmdlet, pero puede cambiarlo a algo más descriptivo. Cámbielo a `Write Hello World to output`.

5. Haga clic en **Parámetros** para especificar los valores de los parámetros del cmdlet.

   Algunos cmdlets tienen varios conjuntos de parámetros; debe seleccionar el que usará. En este caso, `Write-Output` solo tiene un conjunto de parámetros.

6. Seleccione el parámetro `InputObject`. Este es el parámetro que utilizará para especificar el texto que se va a enviar al flujo de salida.

7. El menú desplegable **Origen de datos** proporciona orígenes que se usan para rellenar un valor de parámetro. En este menú, seleccione **Expresión de PowerShell**. 

   Puede usar la salida de esos orígenes como otra actividad, un recurso de Automation o una expresión de PowerShell. En este caso, la salida es simplemente `Hello World`. Puede usar una expresión de PowerShell y especificar una cadena.<br>

8. En el cuadro **Expresión**, escriba `Hello World` y haga clic en **Aceptar** dos veces para volver al lienzo.

9. Guarde el Runbook, para lo que debe hacer clic en **Guardar**.

## <a name="step-3---test-the-runbook"></a>Paso 3: probar el runbook

Antes de publicar el runbook para que esté disponible en producción, debe probarlo para asegurarse de que funciona correctamente. La prueba de un runbook ejecuta su versión de borrador y permite ver la salida de forma interactiva.

1. Seleccione **Panel de prueba** para abrir el panel de prueba.

2. Haga clic en **Iniciar** para iniciar la prueba. Esta debe ser la única opción habilitada.

3. Observe que se crea un [trabajo de runbook](../automation-runbook-execution.md) y su estado se muestra en el panel.

   El estado del trabajo se inicia como `Queued`, que indica que el trabajo está esperando a que haya disponible un trabajo de runbook en la nube. El estado cambia a `Starting` cuando un rol de trabajo solicita el trabajo. Por último, el estado pasa a `Running` cuando el runbook comienza a ejecutarse.

4. Cuando se completa el trabajo del runbook, el panel de prueba muestra su salida. En este caso, verá `Hello World`.

    ![Hola mundo](../media/automation-tutorial-runbook-graphical/runbook-test-results.png)

5. Cierre el panel Prueba para volver al lienzo.

## <a name="step-4---publish-and-start-the-runbook"></a>Paso 4: publicar e iniciar el runbook

El runbook que ha creado aún está en modo de borrador. Debe publicarse antes de ejecutarlo en producción. Al publicar un runbook, se sobrescribe la versión publicada existente con la versión de borrador. En este caso, no tiene una versión publicada aún porque acaba de crear el runbook.

1. Seleccione **Publicar** para publicar el runbook y **Sí** cuando se le solicite.

2. Desplácese hacia la izquierda para ver el runbook en la página Runbooks y observe que el valor de **Estado de creación** está establecido en **Publicado**.

3. Desplácese de nuevo a la derecha para ver la página de **MyFirstRunbook-Graphical**.

   Las opciones que se encuentran en la parte superior permiten iniciar el runbook ahora, programar una hora de inicio futura o crear un [webhook](../automation-webhooks.md) para que el runbook pueda iniciarse mediante una llamada HTTP.

4. Para iniciar el runbook, seleccione **Iniciar** y **Sí** cuando se le solicite.

5. Se abre un panel Trabajo para el trabajo de runbook que se ha creado. Compruebe que en **Estado del trabajo** aparece **Completado**.

6. Haga clic en **Salida** para abrir la página de salida, donde puede ver `Hello World`.

7. Cierre la página Salida.

8. Haga clic en **Todos los registros** para abrir el panel Transmisiones para el trabajo de Runbook. Solo debería ver `Hello World` en el flujo de salida. 

    Tenga en cuenta que el panel Transmisiones puede mostrar otras transmisiones de un trabajo de runbook, como las de error y detalladas, si el runbook escribe en ellas.

9. Cierre el panel Transmisiones y el panel Trabajo para volver a la página de MyFirstRunbook-Graphical.

10. Para ver todos los trabajos del runbook, seleccione **Trabajos** en **Recursos**. La página Trabajos muestra todos los trabajos creados por el runbook. Solo debería ver un trabajo en la lista, ya que solo ejecutó el trabajo una vez.

11. Haga clic en el nombre del trabajo para abrir el mismo panel Trabajo que vio cuando inició el runbook. Use este panel para ver los detalles de cualquier trabajo creado para el runbook.

## <a name="step-5---create-variable-assets"></a>Paso 5: Creación de recursos de variables

Ha probado y publicado el runbook, pero hasta ahora no resulta útil para gestionar recursos de Azure. Antes de configurar el runbook para la autenticación, debe crear una variable que contenga el identificador de suscripción, configurar una actividad para la autenticación y, después, hacer referencia a la variable. Incluir una referencia al contexto de suscripción permite trabajar fácilmente con varias suscripciones.

1. Copie el identificador de la suscripción de la opción **Suscripciones** en el panel Navegación.

2. En la página Cuentas de Automation, seleccione **Variables** en **Recursos compartidos**.

3. Seleccione **Agregar una variable**.

4. En la página Nueva variable, realice la siguiente configuración en los campos indicados.

    * **Nombre**: escriba `AzureSubscriptionId`.
    * **Valor**: escriba el identificador de la suscripción. 
    * **Tipo**: mantenga string seleccionado.
    * **Cifrado**: use el valor predeterminado.

5. Haga clic en **Crear** para crear la variable.

## <a name="step-6---add-authentication"></a>Paso 6: Incorporación de la autenticación

Ahora que tiene una variable para incluir el identificador de suscripción, puede configurar el runbook para que se autentique con las credenciales de ejecución de la suscripción. Para ello, agregue la conexión de ejecución de Azure como un recurso. También debe agregar los cmdlets [Connect-AzAccount](/powershell/module/az.accounts/Connect-AzAccount?view=azps-3.5.0) y [Set-AzContext](/powershell/module/az.accounts/Set-AzContext?view=azps-3.5.0) al lienzo.

>[!NOTE]
>En el caso de los runbooks de PowerShell, `Add-AzAccount` y `Add-AzureRMAccount` son alias de `Connect-AzAccount`. Tenga en cuenta que estos alias no están disponibles para los runbooks gráficos. Un runbook gráfico solo puede usar `Connect-AzAccount`.

1. Vaya al runbook y seleccione **Editar** en la página de MyFirstRunbook-Graphical.

2. La entrada `Write Hello World to output` ya no será necesaria. Haga clic en los puntos suspensivos y seleccione **Eliminar**.

3. En el control Biblioteca, expanda **RECURSOS** y, después, **Conexiones**. Agregue `AzureRunAsConnection` al lienzo seleccionando **Agregar al lienzo**.

4. Cambie el nombre de `AzureRunAsConnection` a `Get Run As Connection`.

5. En el control Biblioteca, escriba `Connect-AzAccount` en el campo de búsqueda.

6. Agregue `Connect-AzAccount` al lienzo.

7. Mantenga el puntero sobre `Get Run As Connection` hasta que aparezca un círculo en la parte inferior de la forma. Haga clic en el círculo y arrastre la flecha hasta `Connect-AzAccount` para crear un vínculo. El runbook se inicia con `Get Run As Connection` y, después, ejecuta `Connect-AzAccount`.<br> ![Crear vínculo entre actividades](../media/automation-tutorial-runbook-graphical/runbook-link-auth-activities.png)

8. En el lienzo, seleccione `Connect-AzAccount`. En el panel de control de configuración, escriba **Login to Azure** (Iniciar sesión en Azure) en el campo **Etiqueta**.

9. Haga clic en **Parámetros**; aparece la página Configuración de parámetros de la actividad.

10. El cmdlet `Connect-AzAccount` tiene varios conjuntos de parámetros; deberá seleccionar uno antes de proporcionar los valores de los parámetros. Haga clic en **Conjunto de parámetros** y seleccione **ServicePrincipalCertificateWithSubscriptionId**.

11. Los parámetros de este conjunto se muestran en la página Configuración de parámetros de la actividad. Haga clic en **APPLICATIONID**.<br> ![Agregar parámetros de cuenta de Azure](../media/automation-tutorial-runbook-graphical/Add-AzureRmAccount-params.png)

12. En la página Valor de parámetro, realice la siguiente configuración y, a continuación, haga clic en **Aceptar**.

   * **Origen de datos**: seleccione **Resultado de la actividad**.
   * Lista de orígenes de datos: seleccione **Get Run As Connection** (Obtener conexión de ejecución).
   * **Ruta de campo**: escriba `ApplicationId`. Vamos a especificar el nombre de la propiedad para la ruta de acceso del campo porque la actividad genera un objeto con varias propiedades.

13. Haga clic en **CERTIFICATETHUMBPRINT** y, en la página Valor de parámetro, realice la configuración siguiente; después, haga clic en **Aceptar**.

    * **Origen de datos**: seleccione **Resultado de la actividad**.
    * Lista de orígenes de datos: seleccione **Get Run As Connection** (Obtener conexión de ejecución).
    * **Ruta de campo**: escriba `CertificateThumbprint`.

14. Haga clic en **SERVICEPRINCIPAL** y, en la página Valor de parámetro, seleccione **ConstantValue** en el campo **Origen de datos**; haga clic en la opción **True** y después en **Aceptar**.

15. Haga clic en **TENANTID** y realice la siguiente configuración en la página Valor de parámetro. Cuando termine, haga clic en **Aceptar** dos veces.

    * **Origen de datos**: seleccione **Resultado de la actividad**. 
    * Lista de orígenes de datos: seleccione **Get Run As Connection** (Obtener conexión de ejecución).
    * **Ruta de campo**: escriba `TenantId`. 

16. En el control Biblioteca, escriba `Set-AzContext` en el campo de búsqueda.

17. Agregue `Set-AzContext` al lienzo.

18. Seleccione `Set-AzContext` en el lienzo. En el panel de control de configuración, escriba `Specify Subscription Id` en el campo **Etiqueta**.

19. Haga clic en **Parámetros**; aparece la página Configuración de parámetros de la actividad.

20. El cmdlet `Set-AzContext` tiene varios conjuntos de parámetros; deberá seleccionar uno antes de proporcionar los valores de los parámetros. Haga clic en **Conjunto de parámetros** y seleccione **SubscriptionId**.

21. Los parámetros de este conjunto se muestran en la página Configuración de parámetros de la actividad. Haga clic en **SubscriptionID**.

22. En la página Valor de parámetro, seleccione **Recurso de variable** en **Origen de datos** y seleccione **AzureSubscriptionId** en la lista de origen. Cuando termine, haga clic en **Aceptar** dos veces.

23. Mantenga el puntero sobre `Login to Azure` hasta que aparezca un círculo en la parte inferior de la forma. Haga clic en el círculo y arrastre la flecha hasta `Specify Subscription Id`. En este punto, el runbook debe tener el siguiente aspecto.

    ![Configuración de autenticación de Runbook](../media/automation-tutorial-runbook-graphical/runbook-auth-config.png)

## <a name="step-7---add-activity-to-start-a-virtual-machine"></a>Paso 7: Adición de una actividad para iniciar una máquina virtual

Ahora debe agregar una actividad `Start-AzVM` para iniciar una máquina virtual. Puede seleccionar cualquier máquina virtual de su suscripción de Azure. Por ahora, codificará ese nombre en el cmdlet [Start-AzVM](/powershell/module/az.compute/start-azvm?view=azps-3.5.0).

1. En el control Biblioteca, escriba `Start-Az` en el campo de búsqueda.

2. Agregue `Start-AzVM` al lienzo y, después, haga clic en él y arrástrelo debajo de `Specify Subscription Id`.

3. Mantenga el puntero sobre `Specify Subscription Id` hasta que aparezca un círculo en la parte inferior de la forma. Haga clic en el círculo y arrastre la flecha hasta `Start-AzVM`.

4. Seleccione `Start-AzVM`. Haga clic en **Parámetros** y después en **Conjunto de parámetros** para ver los conjuntos de la actividad.

5. Seleccione **ResourceGroupNameParameterSetName** para el conjunto de parámetros. Los campos **ResourceGroupName** y **Name** tienen signos de exclamación junto a ellos para indicar que son parámetros necesarios. Tenga en cuenta que ambos campos esperan valores de cadena.

6. Seleccione **Name**. En el campo **Origen de datos**, seleccione **Expresión de PowerShell**. Escriba, delimitado por comillas dobles, el nombre de la máquina virtual que usará para iniciar este runbook. Haga clic en **OK**.

7. Seleccione **ResourceGroupName**. Utilice el valor **Expresión de PowerShell** en el campo **Origen de datos** y escriba el nombre del grupo de recursos entre comillas dobles. Haga clic en **OK**.

8. Haga clic en el **panel de prueba** para poder probar el runbook.

9. Haga clic en **Iniciar** para iniciar la prueba. Una vez que se complete, asegúrese de que la máquina virtual se ha iniciado. En este punto, el runbook debe tener el siguiente aspecto.

    ![Configuración de autenticación de Runbook](../media/automation-tutorial-runbook-graphical/runbook-startvm.png)

## <a name="step-8---add-additional-input-parameters"></a>Paso 8: Incorporación de parámetros de entrada adicionales

Actualmente, el runbook inicia la máquina virtual en el grupo de recursos que especificó para el cmdlet `Start-AzVM`. El runbook será más útil si especifica el nombre y el grupo de recursos cuando se inicie el runbook. Agreguemos parámetros de entrada al runbook para incluir esta funcionalidad.

1. Abra el editor gráfico; para ello, haga clic en **Editar** en la página MyFirstRunbook-Graphical.

2. Seleccione **Entrada y salida**, y en **Agregar entrada** para abrir el panel Parámetros de entrada de Runbook.

3. Realice la siguiente configuración en los campos indicados y, a continuación, haga clic en **Aceptar**.
   * **Nombre**: especifique `VMName`.
   * **Tipo**: mantenga el valor String.
   * **Obligatorio**: cambie el valor a **Sí**.

4. Cree un segundo parámetro de entrada obligatorio denominado `ResourceGroupName` y haga clic en **Aceptar** para cerrar el panel Entrada y salida.<br> ![Parámetros de entrada de runbook](../media/automation-tutorial-runbook-graphical/start-azurermvm-params-outputs.png)

5. Seleccione la actividad `Start-AzVM` y haga clic en **Parámetros**.

6. Cambie el campo **Origen de datos** de **Name** a **Entrada de Runbook**. A continuación, seleccione **VMName**.

7. Cambie el campo **Origen de datos** de **ResourceGroupName** a **Entrada de Runbook** y seleccione **ResourceGroupName**.<br> ![Parámetros de Start-AzVM](../media/automation-tutorial-runbook-graphical/start-azurermvm-params-runbookinput.png)

8. Guarde el runbook y abra el panel Prueba. Ahora puede proporcionar valores para las dos variables de entrada que se usan en la prueba.

9. Cierre el panel Prueba.

10. Haga clic en **Publicar** para publicar la nueva versión del runbook.

11. Detenga la máquina virtual que inició anteriormente.

12. Haga clic en **Iniciar** para iniciar el runbook. Escriba los valores de `VMName` y de `ResourceGroupName` relativos a la máquina virtual que va a iniciar.

13. Cuando se complete el runbook, asegúrese de que se ha iniciado la máquina virtual.

## <a name="step-9---create-a-conditional-link"></a>Paso 9: Creación de un vínculo condicional

Ahora puede modificar el runbook para que solo intente iniciar la máquina virtual si aún no se ha iniciado. Para ello, agregará un cmdlet [Get-AzVM](/powershell/module/Az.Compute/Get-AzVM?view=azps-3.5.0) que recupere el estado de nivel de instancia de la máquina virtual. Tras ello, podrá agregar un módulo de código de flujo de trabajo de PowerShell denominado `Get Status` con un fragmento de código de PowerShell, para determinar si la máquina virtual está en ejecución o detenida. Un vínculo condicional desde el módulo `Get Status` solo ejecutará `Start-AzVM` si el estado de ejecución actual es detenido. Al final de este procedimiento, el runbook usará el cmdlet `Write-Output` para generar un mensaje que le informará de si la máquina virtual se ha iniciado correctamente.

1. Abra **MyFirstRunbook-Graphical** en el editor gráfico.

2. Para quitar el vínculo entre `Specify Subscription Id` y `Start-AzVM`, haga clic en él y presione **Eliminar**.

3. En el control Biblioteca, escriba `Get-Az` en el campo de búsqueda.

4. Agregue `Get-AzVM` al lienzo.

5. Seleccione `Get-AzVM` y, después, **Conjunto de parámetros** para ver los conjuntos del cmdlet. 

6. Seleccione el conjunto de parámetros **GetVirtualMachineInResourceGroupNameParamSet** . Los campos **ResourceGroupName** y **Name** tienen signos de exclamación junto a ellos, lo que indica que especifican parámetros necesarios. Tenga en cuenta que ambos campos esperan valores de cadena.

7. En **Origen de datos** de **Name**, seleccione **Entrada de Runbook** y después **VMName**. Haga clic en **OK**.

8. En **Origen de datos** de **ResourceGroupName**, seleccione **Entrada de Runbook** y después **ResourceGroupName**. Haga clic en **OK**.

9. En **Origen de datos** de **Status**, seleccione **Valor constante** y después **True**. Haga clic en **OK**.

10. Cree un vínculo desde `Specify Subscription Id` a `Get-AzVM`.

11. En el control Biblioteca, expanda **Control de Runbook** y agregue **Code** al lienzo.  

12. Cree un vínculo desde `Get-AzVM` a `Code`.  

13. Haga clic en `Code` y, en el panel de configuración, cambie la etiqueta a **Get Status**.

14. Seleccione `Code`; aparece la página Editor de código.  

15. Pegue el siguiente fragmento de código en la página del editor.

    ```powershell-interactive
     $StatusesJson = $ActivityOutput['Get-AzVM'].StatusesText
     $Statuses = ConvertFrom-Json $StatusesJson
     $StatusOut =""
     foreach ($Status in $Statuses){
     if($Status.Code -eq "Powerstate/running"){$StatusOut = "running"}
     elseif ($Status.Code -eq "Powerstate/deallocated") {$StatusOut = "stopped"}
     }
     $StatusOut
     ```

16. Cree un vínculo desde `Get Status` a `Start-AzVM`.

    ![Runbook con el módulo de código](../media/automation-tutorial-runbook-graphical/runbook-startvm-get-status.png)  

17. Seleccione el vínculo y, en el panel Configuración, cambie **Aplicar condición** a **Sí**. Observe que el vínculo se convierte en una línea discontinua, lo que indica que la actividad de destino solo se ejecuta si la condición se resuelve como verdadera.  

18. En **Expresión de condición**, escriba `$ActivityOutput['Get Status'] -eq "Stopped"`. Ahora, `Start-AzVM` solo se ejecuta si la máquina virtual está detenida.

19. En el control Biblioteca, expanda **Cmdlets** y **Microsoft.PowerShell.Utility**.

20. Agregue `Write-Output` al lienzo dos veces.

21. En el primer control `Write-Output`, haga clic en **Parámetros** y cambie el valor de **Etiqueta** a **Notify VM Started** (Notificar VM iniciada).

22. En **InputObject**, cambie **Origen de datos** a **Expresión de PowerShell** y escriba la expresión `$VMName successfully started.`.

23. En el segundo control `Write-Output`, haga clic en **Parámetros** y cambie el valor de **Etiqueta** a **Notify VM Start Failed** (Notificar error al iniciar VM).

24. En **InputObject**, cambie **Origen de datos** a **Expresión de PowerShell** y escriba la expresión `$VMName could not start`.

25. Cree vínculos desde `Start-AzVM` a `Notify VM Started` y `Notify VM Start Failed`.

26. Seleccione el vínculo a `Notify VM Started` y cambie **Aplicar condición** a True.

27. En **Expresión de condición**, escriba `$ActivityOutput['Start-AzVM'].IsSuccessStatusCode -eq $true`. Este control `Write-Output` ahora solo se ejecuta si la máquina virtual se inicia correctamente.

28. Seleccione el vínculo a `Notify VM Start Failed` y cambie **Aplicar condición** a True.

29. En el campo **Expresión de condición**, escriba `$ActivityOutput['Start-AzVM'].IsSuccessStatusCode -ne $true`. Ahora, este control `Write-Output` solo se ejecuta si la máquina virtual no se ha iniciado correctamente. El runbook debe tener un aspecto similar al de la siguiente imagen.

    ![Runbook con Write-Output](../media/automation-tutorial-runbook-graphical/runbook-startazurermvm-complete.png)

30. Guarde el runbook y abra el panel Prueba.

31. Inicie el runbook con la máquina virtual detenida; la máquina debe iniciarse.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre la creación de runbooks gráficos, consulte [Creación de un runbook gráfico en Azure Automation](../automation-graphical-authoring-intro.md).
* Para empezar a trabajar con Runbooks de PowerShell, consulte [Creación de un runbook de PowerShell](automation-tutorial-runbook-textual-powershell.md).
* Para empezar a trabajar con runbooks de flujo de trabajo de PowerShell, consulte [Creación de un runbook de flujo de trabajo de PowerShell](automation-tutorial-runbook-textual.md).
* Para ver una referencia de los cmdlets de PowerShell, consulte [Az.Automation](/powershell/module/az.automation/?view=azps-3.7.0#automation).
