---
title: Instalación de .NET en roles de Azure Cloud Services | Microsoft Docs
description: En este artículo se describe cómo instalar manualmente .NET Framework en el rol de trabajo y el rol web del servicio en la nube
services: cloud-services
documentationcenter: .net
author: tgore03
manager: carmonm
ms.service: cloud-services
ms.devlang: dotnet
ms.custom: devx-track-dotnet
ms.topic: article
ms.date: 06/22/2018
ms.author: tagore
ms.openlocfilehash: 6de4b79560557fc86edb9e1a25e32a6a1983ceb0
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88932243"
---
# <a name="install-net-on-azure-cloud-services-roles"></a>Instalación de .NET en roles de Azure Cloud Services
En este artículo se describe cómo instalar versiones de .NET Framework que no viene con el SO invitado de Azure. Puede usar .NET en el SO invitado para configurar el rol de trabajo y el rol web del servicio en la nube.

Por ejemplo, puede instalar .NET Framework 4.6.2 en la familia 4 del SO invitado, que no viene con ninguna versión de .NET Framework 4.6. (La familia 5 del SO invitado sí viene con .NET Framework 4.6) Para la información más reciente sobre las versiones del SO invitado de Azure, consulte las [novedades de la versión del SO invitado de Azure](cloud-services-guestos-update-matrix.md). 

>[!IMPORTANT]
>Azure SDK 2.9 contiene una restricción sobre la implementación de .NET Framework 4.6 en la familia 4 o anterior del SO invitado. Hay disponible una corrección para la restricción en el sitio [Microsoft Docs](https://github.com/MicrosoftDocs/azure-cloud-services-files/tree/master/Azure%20Targets%20SDK%202.9).

Para instalar .NET en el rol de trabajo y el rol web, incluya el instalador web de .NET como parte del proyecto de servicios en la nube. Inicie el instalador como parte de las tareas de inicio del rol. 

## <a name="add-the-net-installer-to-your-project"></a>Agregar el instalador de .NET al proyecto
Para descargar el instalador web de .NET Framework, elija la versión que desea instalar:

* [Instalador web de .NET Framework 4.8](https://dotnet.microsoft.com/download/thank-you/net48)
* [Instalador web de .NET Framework 4.7.2](https://go.microsoft.com/fwlink/?LinkId=863262)
* [Instalador web de .NET Framework 4.6.2](https://www.microsoft.com/download/details.aspx?id=53345)

Para agregar el instalador para un rol *web*:
  1. En el **Explorador de soluciones**, en **Roles** del proyecto de servicio en la nube, haga clic con el botón derecho en el rol *web* y seleccione **Agregar** > **Nueva carpeta**. Cree una carpeta llamada **bin**.
  2. Haga clic con el botón derecho en la carpeta bin y seleccione **Agregar** > **Elemento existente**. Seleccione el instalador de .NET y agréguelo a la carpeta bin.
  
Para agregar el instalador para un rol *trabajo*:
* Haga clic con el botón derecho en el rol de *trabajo* y seleccione **Agregar** > **Elemento existente**. Seleccione el instalador de .NET y agréguelo al rol. 

Cuando se agregan de esta manera archivos a la carpeta de contenido de rol, se agregan automáticamente al paquete del servicio en la nube. Los archivos luego se implementan en una ubicación coherente en la máquina virtual. Repita este proceso para cada rol web y rol de trabajo en el servicio en la nube, de manera que todos los roles tengan una copia del instalador.

> [!NOTE]
> Debe instalar .NET Framework 4.6.2 en su rol de servicio en la nube, aunque la aplicación tenga como destino .NET Framework 4.6. El SO invitado incluye la [actualización 3098779](https://support.microsoft.com/kb/3098779) y la [actualización 3097997](https://support.microsoft.com/kb/3097997) de Knowledge Base. Pueden producirse problemas cuando se ejecutan las aplicaciones .NET si .NET Framework 4.6 está instalado sobre las actualizaciones de Knowledge Base. Para evitar estos problemas, instale .NET Framework 4.6.2 en lugar de la versión 4.6. Para más información, consulte los artículos [3118750](https://support.microsoft.com/kb/3118750) y [4340191](https://support.microsoft.com/kb/4340191) de Knowledge Base.
> 
> 

![Contenidos de rol con archivos de instalador][1]

## <a name="define-startup-tasks-for-your-roles"></a>Definir las tareas de inicio para los roles
Puede usar las tareas de inicio para realizar operaciones antes de que se inicie un rol. Instalar .NET Framework como parte de la tarea de inicio garantiza que el marco se instala antes de que se ejecute cualquier código de la aplicación. Para más información sobre de las tareas de inicio, consulte [Ejecución de tareas de inicio en Azure](cloud-services-startup-tasks.md). 

1. Agregue el contenido siguiente al archivo ServiceDefinition.csdef en el nodo **WebRole** o **WorkerRole** para todos los roles:
   
    ```xml
    <LocalResources>
      <LocalStorage name="NETFXInstall" sizeInMB="1024" cleanOnRoleRecycle="false" />
    </LocalResources>    
    <Startup>
      <Task commandLine="install.cmd" executionContext="elevated" taskType="simple">
        <Environment>
          <Variable name="PathToNETFXInstall">
            <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='NETFXInstall']/@path" />
          </Variable>
          <Variable name="ComputeEmulatorRunning">
            <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
          </Variable>
        </Environment>
      </Task>
    </Startup>
    ```
   
    La configuración anterior ejecuta el comando `install.cmd` de la consola con privilegios de administrador para instalar .NET Framework. La configuración también crea un elemento **LocalStorage** denominado **NETFXInstall**. El script de inicio establece la carpeta temporal para utilizar este recurso de almacenamiento local. 
    
    > [!IMPORTANT]
    > Para garantizar la instalación correcta de .NET Framework, establezca el tamaño de este recurso al menos en 1024 MB.
    
    Para más información sobre las tareas de inicio, consulte las [tareas de inicio comunes de Azure Cloud Services](cloud-services-startup-tasks-common.md).

2. Cree un archivo llamado **install.cmd** y agréguele el script de instalación siguiente.

   El script consulta el registro para comprobar si la versión especificada de .NET Framework ya esta instalada en la máquina. Si la versión de .NET Framework no está instalada, se abre el instalador web de .NET Framework. Para ayudar a solucionar cualquier problema, el script registra toda la actividad en el archivo startuptasklog-(fecha y hora actual).txt que se almacena en el almacenamiento local **InstallLogs**.
   
   > [!IMPORTANT]
   > Use un editor de texto básico, como el Bloc de notas de Windows, para crear el archivo the install.cmd. Si usa Visual Studio para crear un archivo de texto y cambia la extensión a .cmd, es posible que el archivo siga teniendo una marca BOM UTF-8. Esta marca puede provocar un error cuando se ejecuta la primera línea del script. Para evitarlo, haga que la primera línea del script sea una instrucción REM que el procesamiento de orden de bytes pueda omitir. 
   > 
   >
   
   ```cmd
   REM Set the value of netfx to install appropriate .NET Framework. 
   REM ***** To install .NET 4.5.2 set the variable netfx to "NDP452" *****
   REM ***** To install .NET 4.6 set the variable netfx to "NDP46" *****
   REM ***** To install .NET 4.6.1 set the variable netfx to "NDP461" ***** https://go.microsoft.com/fwlink/?LinkId=671729
   REM ***** To install .NET 4.6.2 set the variable netfx to "NDP462" ***** https://www.microsoft.com/download/details.aspx?id=53345
   REM ***** To install .NET 4.7 set the variable netfx to "NDP47" ***** 
   REM ***** To install .NET 4.7.1 set the variable netfx to "NDP471" ***** https://go.microsoft.com/fwlink/?LinkId=852095
   REM ***** To install .NET 4.7.2 set the variable netfx to "NDP472" ***** https://go.microsoft.com/fwlink/?LinkId=863262
   set netfx="NDP472"
   REM ***** To install .NET 4.8 set the variable netfx to "NDP48" ***** https://dotnet.microsoft.com/download/thank-you/net48
      
   REM ***** Set script start timestamp *****
   set timehour=%time:~0,2%
   set timestamp=%date:~-4,4%%date:~-10,2%%date:~-7,2%-%timehour: =0%%time:~3,2%
   set "log=install.cmd started %timestamp%."
   
   REM ***** Exit script if running in Emulator *****
   if "%ComputeEmulatorRunning%"=="true" goto exit
   
   REM ***** Needed to correctly install .NET 4.6.1, otherwise you may see an out of disk space error *****
   set TMP=%PathToNETFXInstall%
   set TEMP=%PathToNETFXInstall%
   
   REM ***** Setup .NET filenames and registry keys *****
   if %netfx%=="NDP472" goto NDP472
   if %netfx%=="NDP471" goto NDP471
   if %netfx%=="NDP47" goto NDP47
   if %netfx%=="NDP462" goto NDP462
   if %netfx%=="NDP461" goto NDP461
   if %netfx%=="NDP46" goto NDP46
   
   set "netfxinstallfile=NDP452-KB2901954-Web.exe"
   set netfxregkey="0x5cbf5"
   goto logtimestamp
   
   :NDP46
   set "netfxinstallfile=NDP46-KB3045560-Web.exe"
   set netfxregkey="0x6004f"
   goto logtimestamp
   
   :NDP461
   set "netfxinstallfile=NDP461-KB3102438-Web.exe"
   set netfxregkey="0x6040e"
   goto logtimestamp
   
   :NDP462
   set "netfxinstallfile=NDP462-KB3151802-Web.exe"
   set netfxregkey="0x60632"
   goto logtimestamp
   
   :NDP47
   set "netfxinstallfile=NDP47-KB3186500-Web.exe"
   set netfxregkey="0x707FE"
   goto logtimestamp
   
   :NDP471
   set "netfxinstallfile=NDP471-KB4033344-Web.exe"
   set netfxregkey="0x709fc"
   goto logtimestamp
   
   :NDP472
   set "netfxinstallfile=NDP472-KB4054531-Web.exe"
   set netfxregkey="0x70BF6"
   goto logtimestamp
   
   :logtimestamp
   REM ***** Setup LogFile with timestamp *****
   md "%PathToNETFXInstall%\log"
   set startuptasklog="%PathToNETFXInstall%log\startuptasklog-%timestamp%.txt"
   set netfxinstallerlog="%PathToNETFXInstall%log\NetFXInstallerLog-%timestamp%"
   echo %log% >> %startuptasklog%
   echo Logfile generated at: %startuptasklog% >> %startuptasklog%
   echo TMP set to: %TMP% >> %startuptasklog%
   echo TEMP set to: %TEMP% >> %startuptasklog%
   
   REM ***** Check if .NET is installed *****
   echo Checking if .NET (%netfx%) is installed >> %startuptasklog%
   set /A netfxregkeydecimal=%netfxregkey%
   set foundkey=0
   FOR /F "usebackq skip=2 tokens=1,2*" %%A in (`reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full" /v Release 2^>nul`) do @set /A foundkey=%%C
   echo Minimum required key: %netfxregkeydecimal% -- found key: %foundkey% >> %startuptasklog%
   if %foundkey% GEQ %netfxregkeydecimal% goto installed
   
   REM ***** Installing .NET *****
   echo Installing .NET with commandline: start /wait %~dp0%netfxinstallfile% /q /serialdownload /log %netfxinstallerlog%  /chainingpackage "CloudService Startup Task" >> %startuptasklog%
   start /wait %~dp0%netfxinstallfile% /q /serialdownload /log %netfxinstallerlog% /chainingpackage "CloudService Startup Task" >> %startuptasklog% 2>>&1
   if %ERRORLEVEL%== 0 goto installed
       echo .NET installer exited with code %ERRORLEVEL% >> %startuptasklog%    
       if %ERRORLEVEL%== 3010 goto restart
       if %ERRORLEVEL%== 1641 goto restart
       echo .NET (%netfx%) install failed with Error Code %ERRORLEVEL%. Further logs can be found in %netfxinstallerlog% >> %startuptasklog%
       goto exit
       
   :restart
   echo Restarting to complete .NET (%netfx%) installation >> %startuptasklog%
   shutdown.exe /r /t 5 /c "Installed .NET framework" /f /d p:2:4
   
   :installed
   echo .NET (%netfx%) is installed >> %startuptasklog%
   
   :end
   echo install.cmd completed: %date:~-4,4%%date:~-10,2%%date:~-7,2%-%timehour: =0%%time:~3,2% >> %startuptasklog%
   
   :exit
   EXIT /B 0
   ```

3. Agregue el archivo install.cmd a cada rol en **Agregar** > **Elemento existente** en **Explorador de soluciones**, como se describió anteriormente en este tema. 

    Una vez que complete este paso, todos los roles deberían tener el archivo de instalador de .NET y el archivo install.cmd.

   ![Contenidos de rol con todos los archivos][2]

## <a name="configure-diagnostics-to-transfer-startup-logs-to-blob-storage"></a>Configuración de Diagnostics para transferir registros de inicio a Blob Storage
Para simplificar la resolución de problemas de instalación, puede configurar Azure Diagnostics para transferir cualquier archivo de registro generado por el script de inicio o el instalador de .NET a Azure Blob Storage. Mediante este enfoque, es posible ver los registros si descarga los archivos de registro desde Blob Storage en lugar de tener que establecer una conexión de Escritorio remoto con el rol.


Para configurar Diagnostics, abra el archivo diagnostics.wadcfgx y agregue el contenido siguiente en el nodo **Directorios**: 

```xml 
<DataSources>
 <DirectoryConfiguration containerName="netfx-install">
  <LocalResource name="NETFXInstall" relativePath="log"/>
 </DirectoryConfiguration>
</DataSources>
```

Este XML configura Diagnostics para transferir los archivos del directorio de registro en el recurso **NETFXInstall** a la cuenta de almacenamiento de Diagnostics en el contenedor de blobs **netfx-install**.

## <a name="deploy-your-cloud-service"></a>Implementación del servicio en la nube
Cuando implementa el servicio en la nube, las tareas de inicio instala .NET Framework si todavía no está instalado. Los roles del servicio en la nube quedan en estado *ocupado* mientras se instala .NET Framework. Si la instalación de .NET Framework requiere que se reinicie, es posible que los roles de servicio también lo hagan. 

## <a name="additional-resources"></a>Recursos adicionales
* [Instalar .NET Framework][Installing the .NET Framework]
* [Determinación de las versiones instaladas de .NET Framework][How to: Determine Which .NET Framework Versions Are Installed]
* [Solución de problemas en instalaciones de .NET Framework][Troubleshooting .NET Framework Installations]

[How to: Determine Which .NET Framework Versions Are Installed]: /dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed
[Installing the .NET Framework]: /dotnet/framework/install/guide-for-developers
[Troubleshooting .NET Framework Installations]: /dotnet/framework/install/troubleshoot-blocked-installations-and-uninstallations

<!--Image references-->
[1]: ./media/cloud-services-dotnet-install-dotnet/rolecontentwithinstallerfiles.png
[2]: ./media/cloud-services-dotnet-install-dotnet/rolecontentwithallfiles.png



