---
title: Uso de runbooks de Azure Automation para administrar dispositivos de StorSimple
description: Aprenda a usar el runbook de Azure Automation para administrar el dispositivo de la serie 8000 de StorSimple en Azure Portal.
author: alkohli
ms.service: storsimple
ms.topic: how-to
ms.date: 10/23/2017
ms.author: alkohli
ms.openlocfilehash: fee0837c1196adf35a956cd13df584528d46bfc0
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88183486"
---
# <a name="use-azure-automation-runbooks-to-manage-storsimple-devices"></a>Uso de runbooks de Azure Automation para administrar dispositivos de StorSimple

En este artículo se describe cómo se pueden usar los runbooks de Azure Automation para administrar un dispositivo de la serie StorSimple 8000 en Azure Portal. También se incluye un runbook de ejemplo para guiarle por los pasos necesarios para configurar un entorno para ejecutar este runbook.


## <a name="configure-add-and-run-azure-runbook"></a>Configurar, agregar y ejecutar un runbook de Azure

En esta sección se toma como ejemplo un script de Windows PowerShell para StorSimple y se detallan los pasos necesarios para importar el script en un runbook y, después, publicar y ejecutar el runbook.

### <a name="prerequisites"></a>Prerequisites

Antes de comenzar, asegúrese de que dispone de:

* una suscripción activa de Azure asociada con el servicio StorSimple Device Manager registrado con un dispositivo de la serie StorSimple 8000.

* Windows PowerShell 5.0 instalado en el equipo (o bien, el host de Windows Server para una instancia de StorSimple, si usa una).

### <a name="create-automation-runbook-module-in-windows-powershell"></a>Crear un módulo de runbook de automatización en Windows PowerShell

Para crear un módulo de automatización para la administración de dispositivos de la serie StorSimple 8000, realice los pasos siguientes:

1. Inicie Windows PowerShell. Cree una carpeta nueva y cambie de directorio a la carpeta nueva.

    ```powershell
        mkdir C:\scripts\StorSimpleSDKTools
        cd C:\scripts\StorSimpleSDKTools
    ```

2. [Descargue la CLI de NuGet](https://www.nuget.org/downloads) en la carpeta que se creó en el paso anterior. Existen varias versiones de _nuget.exe_. Elija la versión que corresponde a su SDK. Cada vínculo de descarga apunta directamente a un archivo _.exe_. Asegúrese de hacer clic con el botón derecho y de guardar el archivo en el equipo en lugar de ejecutarlo desde el explorador.

    También puede ejecutar el comando siguiente para descargar y almacenar el script en la misma carpeta que creó anteriormente.

    ```
        wget https://dist.nuget.org/win-x86-commandline/latest/nuget.exe -Out C:\scripts\StorSimpleSDKTools\nuget.exe
    ```

3. Descargue el SDK dependiente.

    ```
        C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Azure.Management.Storsimple8000series
        C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.IdentityModel.Clients.ActiveDirectory -Version 2.28.3
        C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Rest.ClientRuntime.Azure.Authentication -Version 2.2.9-preview
    ```

4. Descargue el script desde el proyecto GitHub de ejemplo.

    ```
        wget https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Monitor-Backups.ps1 -Out Monitor-Backups.ps1
    ```

5. Cree un módulo de Runbook de Azure Automation para la administración de dispositivos de StorSimple serie 8000. Escriba los comandos siguientes en la ventana de Windows PowerShell:

    ```powershell
        # set path variables
        $downloadDir = "C:\scripts\StorSimpleSDKTools"
        $moduleDir = "$downloadDir\AutomationModule\Microsoft.Azure.Management.StorSimple8000Series"

        #don't change the folder name "Microsoft.Azure.Management.StorSimple8000Series"
        mkdir "$moduleDir"
        Copy-Item "$downloadDir\Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3\lib\net45\Microsoft.IdentityModel.Clients.ActiveDirectory*.dll" $moduleDir
        Copy-Item "$downloadDir\Microsoft.Rest.ClientRuntime.Azure.3.3.7\lib\net452\Microsoft.Rest.ClientRuntime.Azure*.dll" $moduleDir
        Copy-Item "$downloadDir\Microsoft.Rest.ClientRuntime.2.3.8\lib\net452\Microsoft.Rest.ClientRuntime*.dll" $moduleDir
        Copy-Item "$downloadDir\Newtonsoft.Json.6.0.8\lib\net45\Newtonsoft.Json*.dll" $moduleDir
        Copy-Item "$downloadDir\Microsoft.Rest.ClientRuntime.Azure.Authentication.2.2.9-preview\lib\net45\Microsoft.Rest.ClientRuntime.Azure.Authentication*.dll" $moduleDir
        Copy-Item "$downloadDir\Microsoft.Azure.Management.Storsimple8000series.1.0.0\lib\net452\Microsoft.Azure.Management.Storsimple8000series*.dll" $moduleDir

        #Don't change the name of the Archive
        compress-Archive -Path "$moduleDir" -DestinationPath Microsoft.Azure.Management.StorSimple8000Series.zip
    ```

6. Compruebe que se crea un archivo ZIP del módulo de automatización en `C:\scripts\StorSimpleSDKTools`.

    ![verify-automation-module](./media/storsimple-8000-automation-azurerm-runbook/verify-automation-module.png)

7. El siguiente resultado se presenta cuando se crea el módulo de automatización mediante Windows PowerShell.

    ```powershell
    mkdir C:\scripts\StorSimpleSDKTools
    ```

    ```Output
        Directory: C:\scripts

    Mode                LastWriteTime         Length Name
    ----                -------------         ------ ----
    d-----       10/18/2017   8:43 AM                StorSimpleSDKTools
    ```

    ```powershell
    wget https://dist.nuget.org/win-x86-commandline/latest/nuget.exe -Out C:\scripts\StorSimpleSDKTools\nuget.exe
    ```

    ```powershell
    C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Azure.Management.Storsimple8000series
    ```

    ```Output
    -------------------------------------------
    CUT              CUT
    -------------------------------------------
    Successfully installed 'Microsoft.Azure.Management.Storsimple8000series 1.0.0' to C:\scripts\StorSimpleSDKTools
    Executing nuget actions took 1.77 sec
    ```

    ```powershell
    C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.IdentityModel.Clients.ActiveDirectory -Version 2.28.3
    ```

    ```Output
    -------------------------------------------
    CUT              CUT
    -------------------------------------------
    Successfully installed 'Microsoft.IdentityModel.Clients.ActiveDirectory 2.28.3' to C:\scripts\StorSimpleSDKTools
    Executing nuget actions took 927.64 ms
    ```

    ```powershell
    C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Rest.ClientRuntime.Azure.Authentication -Version 2.2.9-preview
    ```

    ```Output
    -------------------------------------------
    CUT              CUT
    -------------------------------------------
    Successfully installed 'Microsoft.Rest.ClientRuntime.Azure.Authentication 2.2.9-preview' to C:\scripts\StorSimpleSDKTools
    Executing nuget actions took 717.48 ms
    ```

    ```powershell
    wget https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Monitor-Backups.ps1 -Out Monitor-Backups.ps1
    # set path variables
    $downloadDir = "C:\scripts\StorSimpleSDKTools"
    $moduleDir = "$downloadDir\AutomationModule\Microsoft.Azure.Management.StorSimple8000Series"
    #don't change the folder name "Microsoft.Azure.Management.StorSimple8000Series"
    mkdir "$moduleDir"
    ```

    ```Output
        Directory: C:\scripts\StorSimpleSDKTools\AutomationModule

    Mode                LastWriteTime         Length Name
    ----                -------------         ------ ----
    d-----       10/18/2017   8:48 AM                Microsoft.Azure.Management.StorSimple8000Series
    ```

    ```powershell
    Copy-Item "$downloadDir\Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3\lib\net45\Microsoft.IdentityModel.Clients.ActiveDirectory*.dll" $moduleDir
    Copy-Item "$downloadDir\Microsoft.Rest.ClientRuntime.Azure.3.3.7\lib\net452\Microsoft.Rest.ClientRuntime.Azure*.dll" $moduleDir
    Copy-Item "$downloadDir\Microsoft.Rest.ClientRuntime.2.3.8\lib\net452\Microsoft.Rest.ClientRuntime*.dll" $moduleDir
    Copy-Item "$downloadDir\Newtonsoft.Json.6.0.8\lib\net45\Newtonsoft.Json*.dll" $moduleDir
    Copy-Item "$downloadDir\Microsoft.Rest.ClientRuntime.Azure.Authentication.2.2.9-preview\lib\net45\Microsoft.Rest.ClientRuntime.Azure.Authentication*.dll" $moduleDir
    Copy-Item "$downloadDir\Microsoft.Azure.Management.Storsimple8000series.1.0.0\lib\net452\Microsoft.Azure.Management.Storsimple8000series*.dll" $moduleDir
    #Don't change the name of the Archive
    compress-Archive -Path "$moduleDir" -DestinationPath Microsoft.Azure.Management.StorSimple8000Series.zip
    ```

### <a name="import-publish-and-run-automation-runbook"></a>Importar, publicar y ejecutar el runbook de Automation

1. Cree una cuenta de automatización de ejecución de Azure en Azure Portal. Para ello, vaya a **Azure Marketplace > Todo** y, después, busque **Automation**. Seleccione **Cuentas de Automation**.

    ![search-automation](./media/storsimple-8000-automation-azurerm-runbook/automation1.png)

2. En la hoja **Agregar cuenta de Automation**:

   1. Suministre el **nombre** de la cuenta de Automation.
   2. Seleccione la **suscripción** vinculada a su servicio StorSimple Device Manager.
   3. Cree un nuevo grupo de recursos o seleccione uno del grupo de recursos existente.
   4. Seleccione una **ubicación** (si es posible la misma en donde se está ejecutando el servicio).
   5. Deje la opción predeterminada **Crear cuenta de ejecución** seleccionada.
   6. Si quiere, seleccione **Anclar al panel**. Haga clic en **Crear**.

       ![create-automation-account](./media/storsimple-8000-automation-azurerm-runbook/create-automation-account.png)

      Recibirá una notificación cuando la cuenta de automatización se cree correctamente. Para obtener más información acerca de cómo crear una cuenta de Automation, vaya a [Crear una cuenta de ejecución](https://docs.microsoft.com/azure/automation/automation-create-runas-account).

3. Para asegurarse de que la cuenta de automatización que se creó puede tener acceso al servicio StorSimple Device Manager, debe asignar los permisos adecuados a la cuenta de automatización. En el servicio StorSimple Device Manager, vaya a **Control de acceso**. Haga clic en **+ Agregar** y proporcione el nombre de la cuenta de Azure Automation. **Guarde** la configuración.

    ![add-permissions-automation-account](./media/storsimple-8000-automation-azurerm-runbook/goto-add-roles.png)

4. En la cuenta recién creada, vaya a **Recursos compartidos > Módulos** y haga clic en **+ Agregar módulo**.

5. En la hoja **Agregar módulo**, vaya a la ubicación del módulo comprimido, y seleccione y abra el módulo. Haga clic en **OK**.

    ![add-module](./media/storsimple-8000-automation-azurerm-runbook/add-module.png)

6. Vaya a **Automatización de procesos > Runbooks y haga clic en + Agregar un runbook**. En la hoja **Agregar runbook**, haga clic en **Importar un runbook existente**. Seleccione el archivo de script de Windows PowerShell para el **Archivo de runbook**. El tipo de runbook se selecciona automáticamente. Proporcione un nombre y una descripción opcional del runbook. Haga clic en **Crear**.

    ![add-module](./media/storsimple-8000-automation-azurerm-runbook/import-runbook.png)

7. El runbook se agrega a la lista de runbooks. Seleccione y haga clic en este runbook.

    ![click-new-runbook](./media/storsimple-8000-automation-azurerm-runbook/verify-runbook-created.png)

8. Edite el runbook y haga clic en **Panel de prueba**. Proporcione los parámetros, como el nombre del servicio StorSimple Device Manager, el nombre del dispositivo de StorSimple y la suscripción. **Inicie** la prueba. Una vez completada la ejecución, se genera el informe. Para obtener más información, vaya a [cómo probar un runbook](../automation/automation-first-runbook-textual-powershell.md#step-3---test-the-runbook).

    ![test-runbook](./media/storsimple-8000-automation-azurerm-runbook/test-runbook.png)

9. Revise el resultado del runbook en el panel de prueba. Si le parece correcto, cierre el panel. Haga clic en **Publicar** y, cuando se le solicite una confirmación, confirme y publique el runbook.

    ![publish-runbook](./media/storsimple-8000-automation-azurerm-runbook/publish-runbook.png)

## <a name="next-steps"></a>Pasos siguientes

[Uso del servicio Administrador de dispositivos de StorSimple para administrar un dispositivo de StorSimple](storsimple-8000-manager-service-administration.md).
