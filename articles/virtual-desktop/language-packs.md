---
title: 'Instalación de paquetes de idioma en VM con Windows 10 en Windows Virtual Desktop: Azure'
description: Procedimiento para instalar paquetes de idioma para VM multisesión con Windows 10 en Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 08/21/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: de495d18220500e5aa5653e89776c2634d5b1c85
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88719159"
---
# <a name="add-language-packs-to-a-windows-10-multi-session-image"></a>Adición de paquetes de idioma a una imagen multisesión de Windows 10

Windows Virtual Desktop es un servicio que los usuarios pueden implementar en cualquier momento y lugar. Por eso es importante que los usuarios puedan personalizar el idioma en el que se muestra la imagen multisesión de Windows 10 Enterprise.

Hay dos maneras de adaptarse a las necesidades de idioma de los usuarios:

- Crear grupos de hosts dedicados con una imagen personalizada para cada idioma.
- Colocar a los usuarios con requisitos de idioma y localización diferentes en el mismo grupo de hosts, pero personalizar sus imágenes para asegurarse de que pueden seleccionar el idioma que necesiten.

El último método es mucho más eficaz y rentable. Sin embargo, usted decide cuál es el método que mejor se adapta a sus necesidades. En este artículo se muestra cómo personalizar los idiomas para las imágenes.

## <a name="prerequisites"></a>Requisitos previos

Necesita lo siguiente para personalizar las imágenes multisesión de Windows 10 Enterprise con varios idiomas:

- Una máquina virtual (VM) de Azure con Windows 10 Enterprise multisesión, versión 1903 o posterior.

- El archivo ISO del idioma y el disco 1 de características a petición para la versión del sistema operativo que usa la imagen. Puedes descargarlas aquí: 
     
     - ISO del idioma:
        - [Archivo ISO del paquete de idioma de Windows 10, versión 1903 o 1909](https://software-download.microsoft.com/download/pr/18362.1.190318-1202.19h1_release_CLIENTLANGPACKDVD_OEM_MULTI.iso)
        - [Archivo ISO del paquete de idioma de Windows 10, versión 2004](https://software-download.microsoft.com/download/pr/19041.1.191206-1406.vb_release_CLIENTLANGPACKDVD_OEM_MULTI.iso)

     - Archivo ISO del disco 1 de característica previa petición:
        - [Archivo ISO del disco 1 de característica previa petición de Windows 10, versión 1903 o 1909](https://software-download.microsoft.com/download/pr/18362.1.190318-1202.19h1_release_amd64fre_FOD-PACKAGES_OEM_PT1_amd64fre_MULTI.iso)
        - [Archivo ISO del disco 1 de característica previa petición de Windows 10, versión 2004](https://software-download.microsoft.com/download/pr/19041.1.191206-1406.vb_release_amd64fre_FOD-PACKAGES_OEM_PT1_amd64fre_MULTI.iso)

- Un recurso compartido de Azure Files o un recurso compartido de archivos en una máquina virtual del servidor de archivos de Windows

>[!NOTE]
>El recurso compartido de archivos (repositorio) debe ser accesible desde la máquina virtual de Azure que va a usar para crear la imagen personalizada.

## <a name="create-a-content-repository-for-language-packages-and-features-on-demand"></a>Creación de un repositorio de contenido para paquetes de idioma y características previa petición

Para crear el repositorio de contenido para los paquetes de idioma y las características previa petición:

1. En una máquina virtual de Azure, descargue el archivo ISO multilingüe de Windows 10 y las características previa petición para Windows 10 Enterprise multisesión, versión 1903, 1909 y 2004 de los vínculos en la sección [Requisitos previos](#prerequisites).

2. Abra y monte los archivos ISO en la máquina virtual.

3. Vaya al ISO del paquete de idioma y copie el contenido de las carpetas **LocalExperiencePacks** y **x64\\langpacks** y, a continuación, péguelo en el recurso compartido de archivos.

4. Vaya al **archivo ISO de característica previa petición**, copie todo su contenido y péguelo en el recurso compartido de archivos.

     >[!NOTE]
     > Si está trabajando con un almacenamiento limitado, debe copiar solo los archivos de los idiomas que sepa que los usuarios necesitan. Puede diferenciar los archivos al observar los códigos de idioma en los nombres de archivo. Por ejemplo, el archivo del francés tiene el código "fr-FR" en el nombre. Para obtener una lista completa de los códigos de idioma de todos los idiomas disponibles, consulte [Paquetes de idioma disponibles para Windows](/windows-hardware/manufacture/desktop/available-language-packs-for-windows).

     >[!IMPORTANT]
     > Algunos idiomas requieren fuentes adicionales incluidas en los paquetes satélite que siguen convenciones de nomenclatura distintas. Por ejemplo, los nombres de archivo de las fuentes del japonés incluyen "Jpan".
     >
     > [!div class="mx-imgBorder"]
     > ![Un ejemplo de los paquetes de idioma del japonés con la etiqueta de idioma "Jpan" en el nombre de los archivos.](media/language-pack-example.png)

5. Establezca los permisos en el recurso compartido del repositorio de contenido de idioma para que tenga acceso de lectura desde la máquina virtual que usará para crear la imagen personalizada.

## <a name="create-a-custom-windows-10-enterprise-multi-session-image-manually"></a>Creación manual de una imagen multisesión de Windows 10 Enterprise personalizada

Para crear manualmente una imagen multisesión de Windows 10 Enterprise personalizada:

1. Implemente una máquina virtual de Azure y, luego, vaya a la galería de Azure y seleccione la versión actual de Windows 10 Enterprise multisesión que está usando.
2. Una vez que haya implementado la máquina virtual, conéctese a ella mediante RDP como administrador local.
3. Asegúrese de que la máquina virtual tiene todas las actualizaciones de Windows más recientes. Descargue las actualizaciones y reinicie la máquina virtual, si es necesario.
4. Conéctese al repositorio del recurso compartido del paquete de idioma y característica previa anticipación y móntelo en una letra de unidad (por ejemplo, la unidad E).

## <a name="create-a-custom-windows-10-enterprise-multi-session-image-automatically"></a>Creación automática de una imagen multisesión de Windows 10 Enterprise personalizada

Si prefiere instalar los idiomas a través de un proceso automatizado, puede configurar un script en PowerShell. Puede usar el siguiente ejemplo de script para instalar los paquetes de idioma español (España), francés (Francia) y chino (RPC) y los paquetes satélite para Windows 10 Enterprise multisesión, versión 2004. El script integra el paquete de interfaz de idioma y todos los paquetes satélite necesarios en la imagen. No obstante, también puede modificar este script para instalar otros idiomas. Solo tiene que asegurarse de ejecutar el script desde una sesión de PowerShell con privilegios elevados; de lo contrario, no funcionará.

```powershell
########################################################
## Add Languages to running Windows Image for Capture##
########################################################

##Disable Language Pack Cleanup##
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\AppxDeploymentClient\" -TaskName "Pre-staged app cleanup"

##Set Language Pack Content Stores##
[string]$LIPContent = "E:"

##Spanish##
Add-AppProvisionedPackage -Online -PackagePath $LIPContent\es-es\LanguageExperiencePack.es-es.Neutral.appx -LicensePath $LIPContent\es-es\License.xml
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Client-Language-Pack_x64_es-es.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Basic-es-es-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Handwriting-es-es-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-OCR-es-es-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Speech-es-es-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-TextToSpeech-es-es-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-NetFx3-OnDemand-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-InternetExplorer-Optional-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-MSPaint-FoD-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Notepad-FoD-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-PowerShell-ISE-FOD-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Printing-WFS-FoD-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-StepsRecorder-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-WordPad-FoD-Package~31bf3856ad364e35~amd64~es-es~.cab
$LanguageList = Get-WinUserLanguageList
$LanguageList.Add("es-es")
Set-WinUserLanguageList $LanguageList -force

##French##
Add-AppProvisionedPackage -Online -PackagePath $LIPContent\fr-fr\LanguageExperiencePack.fr-fr.Neutral.appx -LicensePath $LIPContent\fr-fr\License.xml
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Client-Language-Pack_x64_fr-fr.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Basic-fr-fr-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Handwriting-fr-fr-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-OCR-fr-fr-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Speech-fr-fr-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-TextToSpeech-fr-fr-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-NetFx3-OnDemand-Package~31bf3856ad364e35~amd64~fr-fr~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-InternetExplorer-Optional-Package~31bf3856ad364e35~amd64~fr-FR~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-MSPaint-FoD-Package~31bf3856ad364e35~amd64~fr-FR~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Notepad-FoD-Package~31bf3856ad364e35~amd64~fr-FR~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-PowerShell-ISE-FOD-Package~31bf3856ad364e35~amd64~fr-FR~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Printing-WFS-FoD-Package~31bf3856ad364e35~amd64~fr-FR~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-StepsRecorder-Package~31bf3856ad364e35~amd64~fr-FR~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-WordPad-FoD-Package~31bf3856ad364e35~amd64~fr-FR~.cab
$LanguageList = Get-WinUserLanguageList
$LanguageList.Add("fr-fr")
Set-WinUserLanguageList $LanguageList -force

##Chinese(PRC)##
Add-AppProvisionedPackage -Online -PackagePath $LIPContent\zh-cn\LanguageExperiencePack.zh-cn.Neutral.appx -LicensePath $LIPContent\zh-cn\License.xml
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Client-Language-Pack_x64_zh-cn.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Basic-zh-cn-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Fonts-Hans-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Handwriting-zh-cn-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-OCR-zh-cn-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Speech-zh-cn-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-TextToSpeech-zh-cn-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-NetFx3-OnDemand-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-InternetExplorer-Optional-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-MSPaint-FoD-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Notepad-FoD-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-PowerShell-ISE-FOD-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Printing-WFS-FoD-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-StepsRecorder-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-WordPad-FoD-Package~31bf3856ad364e35~amd64~zh-cn~.cab
$LanguageList = Get-WinUserLanguageList
$LanguageList.Add("zh-cn")
Set-WinUserLanguageList $LanguageList -force
```

>[!IMPORTANT]
>Las versiones 1903 y 1909 de Windows 10 Enterprise no requieren el archivo del paquete `Microsoft-Windows-Client-Language-Pack_x64_<language-code>.cab`.

El script puede tardar unos minutos, en función del número de idiomas que necesite instalar.

Una vez finalizada la ejecución del script, asegúrese de que los paquetes de idioma se hayan instalado correctamente; para ello, vaya a **Inicio** > **Configuración** > **Hora e idioma** > **Idioma**. Si los archivos de idioma se encuentran ahí, todo está listo.

Cuando haya terminado, asegúrese de desconectar el recurso compartido.

## <a name="finish-customizing-your-image"></a>Finalización de la personalización de la imagen

Después de instalar los paquetes de idioma, puede instalar cualquier otro software que quiera agregar a la imagen personalizada.

Una vez que haya terminado de personalizar la imagen, deberá ejecutar la herramienta de preparación del sistema (sysprep).

Para ejecutar sysprep, haga lo siguiente:

1. Abra un símbolo del sistema con privilegios elevados y ejecute el siguiente comando para generalizar la imagen:  
   
     ```cmd
     C:\Windows\System32\Sysprep\sysprep.exe /oobe /generalize /shutdown
     ```

2. Apague la máquina virtual, captúrela en una imagen administrada siguiendo las instrucciones de [Captura de una imagen administrada de una máquina virtual generalizada en Azure](../virtual-machines/windows/capture-image-resource.md).

3. Ahora puede usar la imagen personalizada para implementar un grupo de hosts de Windows Virtual Desktop. Para aprender a implementar un grupo de hosts, consulte [Tutorial: Creación de un grupo de hosts con Azure Portal](create-host-pools-azure-marketplace.md).

## <a name="enable-languages-in-windows-settings-app"></a>Habilitación de idiomas en la aplicación de configuración de Windows

Por último, deberá agregar el idioma a la lista de idiomas de cada usuario para que pueda seleccionar su idioma preferido en el menú Configuración.

Para asegurarse de que los usuarios pueden seleccionar los idiomas que ha instalado, inicie sesión como un usuario y, después, ejecute el siguiente cmdlet de PowerShell para agregar los paquetes de idioma instalados al menú Idiomas. También puede configurar este script como una tarea automatizada que se activa cuando el usuario inicia sesión en su sesión.

```powershell
$LanguageList = Get-WinUserLanguageList
$LanguageList.Add("es-es")
$LanguageList.Add("fr-fr")
$LanguageList.Add("zh-cn")
Set-WinUserLanguageList $LanguageList -force
```

Una vez que un usuario ha cambiado la configuración de idioma, tendrá que cerrar la sesión de Windows Virtual Desktop e iniciar sesión de nuevo para que los cambios surtan efecto. 

## <a name="next-steps"></a>Pasos siguientes

Si tiene curiosidad sobre los problemas conocidos de los paquetes de idioma, consulte [Adición de paquetes de idioma en Windows 10, versión 1803 y versiones posteriores: Problemas conocidos](/windows-hardware/manufacture/desktop/language-packs-known-issue).

Si tiene alguna otra pregunta sobre Windows 10 Enterprise multisesión, consulte nuestras [preguntas más frecuentes](windows-10-multisession-faq.md).
