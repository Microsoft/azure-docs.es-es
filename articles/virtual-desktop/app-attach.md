---
title: 'Asociación de aplicaciones en formato MSIX de Windows Virtual Desktop: Azure'
description: Configuración de la asociación de aplicaciones en formato MSIX para Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 06/16/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: e461bbf8c3a6cd845744fc0e17b5d1f0eb9bef58
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88010164"
---
# <a name="set-up-msix-app-attach"></a>Configuración de la asociación de aplicaciones en formato .MSIX

> [!IMPORTANT]
> La asociación de aplicaciones en formato MSIX está actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin un Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Este tema lo guiará a través de la configuración de la asociación de aplicaciones en formato MSIX en un entorno de Windows Virtual Desktop.

## <a name="requirements"></a>Requisitos

Antes de empezar, esto es lo que necesita para configurar la asociación de aplicaciones en formato MSIX:

- Acceso al portal de Windows Insider para obtener la versión de Windows 10 con compatibilidad con las API de asociación de aplicaciones con formato MSIX.
- Una implementación de Windows Virtual Desktop en funcionamiento. Para obtener información sobre cómo implementar Windows Virtual Desktop (clásico), consulte [Creación de un inquilino en Windows Virtual Desktop](./virtual-desktop-fall-2019/tenant-setup-azure-active-directory.md). Para obtener información sobre cómo implementar Windows Virtual Desktop con la integración de Azure Resource Manager, consulte [Creación de un grupo de hosts con Azure Portal](./create-host-pools-azure-marketplace.md).
- La herramienta de empaquetado MSIX.
- Un recurso compartido de red en la implementación de Windows Virtual Desktop donde se almacenará el paquete MSIX.

## <a name="get-the-os-image"></a>Obtención de la imagen del sistema operativo

En primer lugar, debe obtener la imagen del sistema operativo. Puede obtener la imagen del sistema operativo mediante Azure Portal. Sin embargo, si es miembro del programa Windows Insider, puede usar el portal de Windows Insider en su lugar.

### <a name="get-the-os-image-from-the-azure-portal"></a>Obtención de la imagen de sistema operativo desde Azure Portal

Para obtener la imagen de sistema operativo desde Azure Portal:

1. Abra [Azure Portal](https://portal.azure.com) e inicie sesión.

2. Vaya a **Crear una máquina virtual**.

3. En la pestaña **Básico**, seleccione **Sesión múltiple de Windows 10 Enterprise, versión 2004**.

4. Siga el resto de las instrucciones para terminar de crear la máquina virtual.

     >[!NOTE]
     >Puede usar esta máquina virtual para probar directamente la aplicación MSIX adjunta. Para más información, vaya a [Generación de un paquete VHD o VHDX para MSIX](#generate-a-vhd-or-vhdx-package-for-msix). De lo contrario, siga leyendo esta sección.

### <a name="get-the-os-image-from-the-windows-insider-portal"></a>Obtención de la imagen del sistema operativo desde el portal de Windows Insider

Para obtener la imagen de sistema operativo desde el portal de Windows Insider:

1. Abra el [portal de Windows Insider](https://www.microsoft.com/software-download/windowsinsiderpreviewadvanced?wa=wsignin1.0) e inicie sesión.

     >[!NOTE]
     >Para acceder al portal de Windows Insider, debe ser miembro del programa Windows Insider. Para más información sobre el programa Windows Insider, revise la [documentación sobre Windows Insider](/windows-insider/at-home/).

2. Desplácese hasta la sección **Seleccionar edición** y seleccione **Windows 10 Insider Preview Enterprise (FAST) – Build 19041** [Windows 10 Insider Preview Enterprise (FAST), compilación 19041] o posterior.

3. Seleccione **Confirmar**, el idioma que quiere usar y, luego, vuelva a seleccionar **Confirmar**.

     >[!NOTE]
     >Por el momento, solo se probó el inglés con la característica. Puede seleccionar otros idiomas, pero es posible que no se muestren según lo previsto.

4. Cuando se genere el vínculo de descarga, seleccione la **descarga para 64 bits** y guárdela en el disco duro local.

## <a name="prepare-the-vhd-image-for-azure"></a>Preparación de la imagen de disco duro virtual para Azure

A continuación, va a crear una imagen de VHD maestro. Si todavía no crea esta imagen, vaya a [Preparación y personalización de una imagen de disco duro virtual maestro](set-up-customize-master-image.md) y siga las instrucciones.

Después de crear la imagen de disco duro virtual maestro, debe deshabilitar las actualizaciones automáticas para las aplicaciones de asociación de aplicaciones en formato MSIX. Para deshabilitar las actualizaciones automáticas, deberá ejecutar estos comandos en un símbolo del sistema con privilegios elevados:

```cmd
rem Disable Store auto update:

reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Automatic app update" /Disable
Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable

rem Disable Content Delivery auto download apps that they want to promote to users:

reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f

reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f

rem Disable Windows Update:

sc config wuauserv start=disabled
```

Después de deshabilitar las actualizaciones automáticas, debe habilitar Hyper-V porque va a usar el comando Mount-VHD para agregar al "stage" y Dismount-VHD para quitar del "stage".

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```
>[!NOTE]
>Este cambio requerirá que se reinicie la máquina virtual.

A continuación, prepare el disco duro virtual de la máquina virtual para Azure y cargue el disco duro virtual resultante en Azure. Para más información, consulte [Preparación y personalización de una imagen de disco duro virtual maestro](set-up-customize-master-image.md).

Una vez que haya cargado el disco duro virtual en Azure, siga las instrucciones que aparecen en el tutorial [Creación de un grupo host con Azure Marketplace](create-host-pools-azure-marketplace.md) para crear un grupo host basado en esta imagen nueva.

## <a name="prepare-the-application-for-msix-app-attach"></a>Preparación de la aplicación para la asociación de aplicaciones en formato MSIX

Si ya tiene un paquete de MSIX, vaya directamente a [Configuración de la infraestructura de Windows Virtual Desktop](#configure-windows-virtual-desktop-infrastructure). Si quiere probar las aplicaciones heredadas, siga las instrucciones que aparecen en [Creación de un paquete de MSIX a partir de un instalador de escritorio en una máquina virtual](/windows/msix/packaging-tool/create-app-package-msi-vm/) para convertir la aplicación heredada en un paquete de MSIX.

## <a name="generate-a-vhd-or-vhdx-package-for-msix"></a>Generación de un paquete VHD o VHDX para MSIX

Los paquetes están en formato VHD o VHDX para optimizar el rendimiento. MSIX requiere que los paquetes VHD o VHDX funcionen correctamente.

Para generar un paquete VHD o VHDX para MSIX:

1. [Descargue la herramienta msixmgr](https://aka.ms/msixmgr) y guarde la carpeta ZIP en una carpeta dentro de una VM del host de sesión.

2. Descomprima la carpeta ZIP de la herramienta msixmgr.

3. Ponga el paquete de MSIX de origen en la misma carpeta donde descomprimió la herramienta msixmgr.

4. Ejecute el cmdlet siguiente en PowerShell para crear un VHD:

    ```powershell
    New-VHD -SizeBytes <size>MB -Path c:\temp\<name>.vhd -Dynamic -Confirm:$false
    ```

    >[!NOTE]
    >Asegúrese de que el tamaño del disco duro virtual es lo suficientemente grande para contener el MSIX expandido*.

5. Ejecute este cmdlet para montar el disco duro virtual recién creado:

    ```powershell
    $vhdObject = Mount-VHD c:\temp\<name>.vhd -Passthru
    ```

6. Ejecute este cmdlet para inicializar el disco duro virtual:

    ```powershell
    $disk = Initialize-Disk -Passthru -Number $vhdObject.Number
    ```

7. Ejecute este cmdlet para crear una partición nueva:

    ```powershell
    $partition = New-Partition -AssignDriveLetter -UseMaximumSize -DiskNumber $disk.Number
    ```

8. Ejecute este cmdlet para dar formato a la partición:

    ```powershell
    Format-Volume -FileSystem NTFS -Confirm:$false -DriveLetter $partition.DriveLetter -Force
    ```

9. Cree una carpeta principal en el disco duro virtual montado. Este paso es obligatorio, porque la asociación de aplicaciones en formato MSIX requiere una carpeta principal. Puede asignar a la carpeta principal el nombre que quiera.

### <a name="expand-msix"></a>Expansión de MSIX

Después de eso, deberá descomprimir la imagen de MSIX para "expandirla". Para desempaquetar la imagen de MSIX:

1. Abra un símbolo del sistema como administrador y vaya a la carpeta en la que descargó y descomprimió la herramienta msixmgr.

2. Ejecute el cmdlet siguiente para desempaquetar MSIX en el disco duro virtual que creó y montó en la sección anterior.

    ```powershell
    msixmgr.exe -Unpack -packagePath <package>.msix -destination "f:\<name of folder you created earlier>" -applyacls
    ```

    El mensaje siguiente debería aparecer una vez que termine el desempaquetado:

    `Successfully unpacked and applied ACLs for package: <package name>.msix`

    >[!NOTE]
    > Si usa paquetes de Microsoft Store para Empresas (o Educación) dentro de la red o en dispositivos que no están conectados a Internet, deberá obtener las licencias de paquete en Store e instalarlas para ejecutar correctamente la aplicación. Consulte [Uso de paquetes sin conexión](#use-packages-offline).

3. Vaya al disco duro virtual montado y abra la carpeta de la aplicación y confirme que el contenido del paquete existe.

4. Desmonte el disco duro virtual.

## <a name="configure-windows-virtual-desktop-infrastructure"></a>Configuración de la infraestructura de Windows Virtual Desktop

Por diseño, un paquete expandido de MSIX (el disco duro virtual que creó en la sección anterior) se puede compartir entre varias máquinas virtuales de host de sesión, porque los discos duros virtuales están adjuntos en modo de solo lectura.

Antes de empezar, asegúrese de que el recurso compartido de red cumple estos requisitos:

- El recurso compartido es compatible con SMB.
- Las máquinas virtuales que forman parte del grupo de hosts de sesión tienen permisos de NTFS para el recurso compartido.

### <a name="set-up-an-msix-app-attach-share"></a>Configuración de un recurso compartido de asociación de aplicaciones en formato MSIX

En el entorno de Windows Virtual Desktop, cree un recurso compartido de red y mueva ahí el paquete.

>[!NOTE]
> El procedimiento recomendado para crear recursos compartidos de red de MSIX es configurar el recurso compartido de red con permisos de NTFS de solo lectura.

## <a name="install-certificates"></a>Instalación de certificados

Si la aplicación usa un certificado que no es de confianza pública o que fue autofirmado, puede instalarlo de esta manera:

1. Haga clic con el botón derecho en el paquete y seleccione **Propiedades**.
2. En la ventana que aparece, seleccione la pestaña **Firmas digitales**. Solo habrá un elemento en la lista en la pestaña, tal como se muestra en la imagen siguiente. Seleccione ese elemento para resaltar el elemento y, luego, seleccione **Detalles**.
3. Cuando aparezca la ventana de detalles de firma digital, seleccione la pestaña **General**, luego, **Ver certificado** y, por último, **Instalar certificado**.
4. Cuando se abra el instalador, seleccione **máquina local** como su ubicación de almacenamiento y, luego, seleccione **Siguiente**.
5. Si el instalador le pregunta si quiere permitir que la aplicación haga cambios en el dispositivo, seleccione **Sí**.
6. Seleccione **Colocar todos los certificados en el siguiente almacén** y, después, **Examinar**.
7. Cuando aparezca la ventana del almacén de certificados, seleccione **Trusted people** (Personas de confianza) y, luego, **Aceptar**.
8. Seleccione **Siguiente** y **Finalizar**.

## <a name="prepare-powershell-scripts-for-msix-app-attach"></a>Preparación de scripts para PowerShell para la asociación de aplicaciones en formato MSIX

La asociación de aplicaciones en formato MSIX tiene cuatro fases distintas que se deben realizar en este orden:

1. Fase
2. Register
3. Eliminar registro
4. Quitar del "stage"

En cada fase se crea un script de PowerShell. Los scripts de ejemplo para fase están disponibles [aquí](https://github.com/Azure/RDS-Templates/tree/master/msix-app-attach).

### <a name="stage-powershell-script"></a>Script de PowerShell de fase

Antes de actualizar los scripts de PowerShell, asegúrese de que tiene el GUID de volumen del volumen en el disco duro virtual. Para obtener el GUID del volumen:

1.  Abra el recurso compartido de red donde se encuentra el disco duro virtual dentro de la máquina virtual donde se ejecutará el script.

2.  Haga clic con el botón derecho y seleccione **Montar**. De este modo, el disco duro virtual se montará en una letra de unidad.

3.  Después de montar el disco duro virtual, se abrirá la ventana **Explorador de archivos**. Capture la carpeta principal y actualice la variable **$parentFolder**.

    >[!NOTE]
    >Si no ve una carpeta principal, significa que MSIX no se expandió de manera correcta. Rehaga la sección anterior y vuelva a intentarlo.

4.  Abra la carpeta principal. Si se expandió correctamente, verá una carpeta con el mismo nombre del paquete. Actualice la variable **$packageName** para que coincida con el nombre de esta carpeta.

    Por ejemplo, `VSCodeUserSetup-x64-1.38.1_1.38.1.0_x64__8wekyb3d8bbwe`.

5.  Abra un símbolo del sistema y escriba **mountvol**. Este comando mostrará una lista de volúmenes y sus GUID. Copie el GUID del volumen en el que la letra de unidad coincide con la unidad que montó en el disco duro virtual del paso 2.

    Por ejemplo, en esta salida de ejemplo del comando mountvol, si montó el disco duro virtual en la unidad C, querrá copiar el valor anterior `C:\`:

    ```cmd
    Possible values for VolumeName along with current mount points are:

    \\?\Volume{a12b3456-0000-0000-0000-10000000000}\
    *** NO MOUNT POINTS ***

    \\?\Volume{c78d9012-0000-0000-0000-20000000000}\
        E:\

    \\?\Volume{d34e5678-0000-0000-0000-30000000000}\
        C:\

    ```


6.  Actualice la variable **$volumeGuid** con el GUID del volumen que acaba de copiar.

7. Abra un símbolo del sistema de administrador de PowerShell y actualice el script de PowerShell siguiente con las variables que se aplican al entorno.

    ```powershell
    #MSIX app attach staging sample

    #region variables
    $vhdSrc="<path to vhd>"
    $packageName = "<package name>"
    $parentFolder = "<package parent folder>"
    $parentFolder = "\" + $parentFolder + "\"
    $volumeGuid = "<vol guid>"
    $msixJunction = "C:\temp\AppAttach\"
    #endregion

    #region mountvhd
    try
    {
          Mount-Diskimage -ImagePath $vhdSrc -NoDriveLetter -Access ReadOnly
          Write-Host ("Mounting of " + $vhdSrc + " was completed!") -BackgroundColor Green
    }
    catch
    {
          Write-Host ("Mounting of " + $vhdSrc + " has failed!") -BackgroundColor Red
    }
    #endregion

    #region makelink
    $msixDest = "\\?\Volume{" + $volumeGuid + "}\"
    if (!(Test-Path $msixJunction))
    {
         md $msixJunction
    }

    $msixJunction = $msixJunction + $packageName
    cmd.exe /c mklink /j $msixJunction $msixDest
    #endregion

    #region stage
    [Windows.Management.Deployment.PackageManager,Windows.Management.Deployment,ContentType=WindowsRuntime] | Out-Null
    Add-Type -AssemblyName System.Runtime.WindowsRuntime
    $asTask = ([System.WindowsRuntimeSystemExtensions].GetMethods() | Where { $_.ToString() -eq 'System.Threading.Tasks.Task`1[TResult] AsTask[TResult,TProgress](Windows.Foundation.IAsyncOperationWithProgress`2[TResult,TProgress])'})[0]
    $asTaskAsyncOperation = $asTask.MakeGenericMethod([Windows.Management.Deployment.DeploymentResult], [Windows.Management.Deployment.DeploymentProgress])
    $packageManager = [Windows.Management.Deployment.PackageManager]::new()
    $path = $msixJunction + $parentFolder + $packageName # needed if we do the pbisigned.vhd
    $path = ([System.Uri]$path).AbsoluteUri
    $asyncOperation = $packageManager.StagePackageAsync($path, $null, "StageInPlace")
    $task = $asTaskAsyncOperation.Invoke($null, @($asyncOperation))
    $task
    #endregion
    ```

### <a name="register-powershell-script"></a>Registrar el script de PowerShell

Para ejecutar el script de registro, ejecute estos cmdlets de PowerShell con los valores de marcador de posición reemplazados por los valores que se aplican al entorno.

```powershell
#MSIX app attach registration sample

#region variables
$packageName = "<package name>"
$path = "C:\Program Files\WindowsApps\" + $packageName + "\AppxManifest.xml"
#endregion

#region register
Add-AppxPackage -Path $path -DisableDevelopmentMode -Register
#endregion
```

### <a name="deregister-powershell-script"></a>Eliminar el registro del script de PowerShell

Para este script, reemplace el marcador de posición de **$packageName** por el nombre del paquete que está probando.

```powershell
#MSIX app attach deregistration sample

#region variables
$packageName = "<package name>"
#endregion

#region deregister
Remove-AppxPackage -PreserveRoamableApplicationData $packageName
#endregion
```

### <a name="destage-powershell-script"></a>Quitar el script de PowerShell del "stage"

Para este script, reemplace el marcador de posición de **$packageName** por el nombre del paquete que está probando.

```powershell
#MSIX app attach de staging sample

#region variables
$packageName = "<package name>"
$msixJunction = "C:\temp\AppAttach\"
#endregion

#region deregister
Remove-AppxPackage -AllUsers -Package $packageName
cd $msixJunction
rmdir $packageName -Force -Verbose
#endregion
```

## <a name="set-up-simulation-scripts-for-the-msix-app-attach-agent"></a>Configuración de scripts de simulación para el agente de asociación de aplicaciones en formato MSIX

Después de crear los scripts, los usuarios pueden ejecutarlos manualmente o configurarlos para que se ejecuten de manera automática en como scripts de inicio, inicio de sesión, cierre de sesión y apagado. Para más información sobre estos tipos de scripts, consulte el artículo sobre cómo [usar los scripts de inicio, apagado, inicio de sesión y cierre de sesión en una directiva de grupo](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn789196(v=ws.11)/).

Cada uno de estos scripts automáticos ejecuta una fase de los scripts de asociación de aplicaciones:

- El script de inicio ejecuta el script para agregar al "stage".
- El script de inicio de sesión ejecuta el script para registrar.
- El script de cierre de sesión ejecuta el script para eliminar del registro.
- El script de apagado ejecuta el script para quitar del "stage".

## <a name="use-packages-offline"></a>Uso de paquetes sin conexión

Si usa paquetes de [Microsoft Store para Empresas](https://businessstore.microsoft.com/) o de [Microsoft Store para Educación](https://educationstore.microsoft.com/) dentro de la red o en dispositivos que no está conectados a Internet, debe obtener las licencias de paquete en Microsoft Store e instalarlas en el dispositivo para ejecutar correctamente la aplicación. Si el dispositivo está en línea y puede conectarse a Microsoft Store para Empresas, las licencias necesarias se deben descargar automáticamente pero, si está sin conexión, deberá configurar las licencias de manera manual.

Para instalar los archivos de licencia, deberá usar un script de PowerShell que llama a la clase MDM_EnterpriseModernAppManagement_StoreLicenses02_01 en el proveedor de puentes de WMI.

Aquí se muestra cómo configurar las licencias para usarlas sin conexión:

1. Descargue el paquete de la aplicación, las licencias y los marcos de trabajo requeridos de Microsoft Store para Empresas. Necesita los archivos de licencias codificados y sin codificar. [Aquí](/microsoft-store/distribute-offline-apps#download-an-offline-licensed-app) puede encontrar instrucciones detalladas sobre la descarga.
2. Actualice estas variables en el script del paso 3:
      1. `$contentID` es el valor ContentID del archivo de licencia no codificado (.xml). Puede abrir el archivo de licencia en el editor de texto que prefiera.
      2. `$licenseBlob` es la cadena entera para el blob de licencia en el archivo de licencia codificado (.bin). Puede abrir el archivo de licencia codificado en el editor de texto que prefiera.
3. Ejecute el script siguiente desde un símbolo del sistema de administrador de PowerShell. Un buen lugar para instalar la licencia es al final del [script para agregar al "stage"](#stage-powershell-script) que también se tiene que ejecutar desde un símbolo del sistema de administrador.

```powershell
$namespaceName = "root\cimv2\mdm\dmmap"
$className = "MDM_EnterpriseModernAppManagement_StoreLicenses02_01"
$methodName = "AddLicenseMethod"
$parentID = "./Vendor/MSFT/EnterpriseModernAppManagement/AppLicenses/StoreLicenses"

#TODO - Update $contentID with the ContentID value from the unencoded license file (.xml)
$contentID = "{'ContentID'_in_unencoded_license_file}"

#TODO - Update $licenseBlob with the entire String in the encoded license file (.bin)
$licenseBlob = "{Entire_String_in_encoded_license_file}"

$session = New-CimSession

#The final string passed into the AddLicenseMethod should be of the form <License Content="encoded license blob" />
$licenseString = '<License Content='+ '"' + $licenseBlob +'"' + ' />'

$params = New-Object Microsoft.Management.Infrastructure.CimMethodParametersCollection
$param = [Microsoft.Management.Infrastructure.CimMethodParameter]::Create("param",$licenseString ,"String", "In")
$params.Add($param)


try
{
   $instance = New-CimInstance -Namespace $namespaceName -ClassName $className -Property @{ParentID=$parentID;InstanceID=$contentID}
   $session.InvokeMethod($namespaceName, $instance, $methodName, $params)

}
catch [Exception]
{
     write-host $_ | out-string
}
```

## <a name="next-steps"></a>Pasos siguientes

Esta característica no se admite actualmente, pero puede preguntarle a la comunidad en [TechCommunity sobre Windows Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop).

También puede dejar comentarios sobre Windows Virtual Desktop en el [Concentrador de comentarios de Windows Virtual Desktop](https://support.microsoft.com/help/4021566/windows-10-send-feedback-to-microsoft-with-feedback-hub-app).
