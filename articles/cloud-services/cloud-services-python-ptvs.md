---
title: Introducción a Python y a Azure Cloud Services | Microsoft Docs
description: Información general sobre el uso de Python Tools para Visual Studio para crear servicios en la nube de Azure, incluidos roles web y roles de trabajo.
services: cloud-services
documentationcenter: python
author: tgore03
ms.service: cloud-services
ms.devlang: python
ms.topic: conceptual
ms.date: 07/18/2017
ms.author: tagore
ms.custom: devx-track-python
ms.openlocfilehash: 7025d042fa021c6ee5311566d66bfd090dbd06cd
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87851586"
---
# <a name="python-web-and-worker-roles-with-python-tools-for-visual-studio"></a>Roles web y de trabajo de Python con herramientas de Python para Visual Studio

En este artículo se ofrece información general sobre el uso de roles web y de trabajo de Python con [herramientas de Python para Visual Studio][Python Tools for Visual Studio]. Obtenga información acerca de cómo usar Visual Studio para crear e implementar un servicio en la nube básico que usa Python.

## <a name="prerequisites"></a>Prerrequisitos
* [Visual Studio 2013, 2015 o 2017](https://www.visualstudio.com/)
* [Herramientas de Python para Visual Studio][Python Tools for Visual Studio] (PTVS)
* [Herramientas de Azure SDK para VS 2013][Azure SDK Tools for VS 2013] o  
[Herramientas de Azure SDK para VS 2015][Azure SDK Tools for VS 2015] o  
[Herramientas de Azure SDK para VS 2017][Azure SDK Tools for VS 2017]
* [Python 2.7 de 32 bits][Python 2.7 32-bit] o [Python 3.5 de 32 bits][Python 3.5 32-bit]

[!INCLUDE [create-account-and-websites-note](../../includes/create-account-and-websites-note.md)]

## <a name="what-are-python-web-and-worker-roles"></a>¿Qué son los roles web y de trabajo de Python?
Azure proporciona tres modelos de proceso para ejecutar aplicaciones: [La característica Web Apps en Azure App Service][execution model-web sites], [Azure Virtual Machines][execution model-vms] y [Azure Cloud Services][execution model-cloud services]. Los tres modelos admiten Python. Cloud Services, que incluye roles web y de trabajo, proporciona una *Plataforma como servicio (PaaS)* . En un servicio en la nube, un rol web ofrece un servidor web dedicado de Internet Information Services (IIS) para hospedar aplicaciones web front-end, mientras que un rol de trabajo puede ejecutar tareas asincrónicas, de ejecución prolongada o tareas perpetuas independientes de la entrada o la interacción del usuario.

Para más información, consulte [¿Qué es un servicio en la nube?]

> [!NOTE]
> *¿Desea compilar un sitio web sencillo?*
> Si el escenario solo requiere un sitio web de front-end sencillo, considere la posibilidad de usar la característica Web Apps ligera en Azure App Service. Puede actualizar a un Servicio en la nube más adelante, cuando su sitio web sea más grande y sus requisitos cambien. Consulte el [Centro para desarrolladores de Python](https://azure.microsoft.com/develop/python/) para obtener artículos sobre el desarrollo de la característica Web Apps en Azure App Service.
> <br />
> 
> 

## <a name="project-creation"></a>Creación de un proyecto
En Visual Studio, puede seleccionar **Servicio en la nube de Azure** en el cuadro de diálogo **Nuevo proyecto**, en **Python**.

![Cuadro de diálogo Nuevo proyecto](./media/cloud-services-python-ptvs/new-project-cloud-service.png)

En el asistente Servicio en la nube de Azure, puede crear roles web y de trabajo nuevos.

![Cuadro de diálogo Servicio en la nube de Azure](./media/cloud-services-python-ptvs/new-service-wizard.png)

La plantilla de rol de trabajo incluye código reutilizable que conecta a una cuenta de Azure Storage o Azure Service Bus.

![Solución de servicio en la nube](./media/cloud-services-python-ptvs/worker.png)

Puede agregar roles web o de trabajo a un servicio en la nube que ya existe en cualquier momento.  Puede optar por agregar proyectos existentes a su solución o por crear otros nuevos.

![Comando Agregar rol](./media/cloud-services-python-ptvs/add-new-or-existing-role.png)

Su servicio en la nube puede contener roles implementados en diferentes lenguajes.  Por ejemplo, puede tener un rol web de Python implementado con Django, con Python o con roles de trabajo de C#.  Puede comunicarse fácilmente entre sus roles usando colas de Service Bus o colas de almacenamiento.

## <a name="install-python-on-the-cloud-service"></a>Instalación de Python en el servicio en la nube
> [!WARNING]
> No funcionan los scripts de configuración que se instalaron con Visual Studio (en el momento en que se actualizó por última vez este artículo). En esta sección se describe una solución alternativa.
> 
> 

El problema principal con los scripts de configuración consiste en que instalan Python. En primer lugar, defina dos [tareas de inicio](cloud-services-startup-tasks.md) en el archivo [ServiceDefinition.csdef](cloud-services-model-and-package.md#servicedefinitioncsdef). La primera tarea (**PrepPython.ps1**) descarga e instala el entorno de tiempo de ejecución de Python. La segunda tarea (**PipInstaller.ps1**) ejecuta pip para instalar todas las dependencias que pueda tener.

Los siguientes scripts se escribieron para Python 3.5. Si desea usar la versión 2.x de Python, establezca el archivo de variables **PYTHON2** en **activado** para las dos tareas de inicio y la tarea en tiempo de ejecución: `<Variable name="PYTHON2" value="<mark>on</mark>" />`.

```xml
<Startup>

  <Task executionContext="elevated" taskType="simple" commandLine="bin\ps.cmd PrepPython.ps1">
    <Environment>
      <Variable name="EMULATED">
        <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
      </Variable>
      <Variable name="PYTHON2" value="off" />
    </Environment>
  </Task>

  <Task executionContext="elevated" taskType="simple" commandLine="bin\ps.cmd PipInstaller.ps1">
    <Environment>
      <Variable name="EMULATED">
        <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
      </Variable>
      <Variable name="PYTHON2" value="off" />
    </Environment>

  </Task>

</Startup>
```

Se deben agregar las variables **PYTHON2** y **PYPATH** a la tarea de inicio del trabajo. La variable **PYPATH** solo se usa si la variable **PYTHON2** se establece en **activado**.

```xml
<Runtime>
  <Environment>
    <Variable name="EMULATED">
      <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
    </Variable>
    <Variable name="PYTHON2" value="off" />
    <Variable name="PYPATH" value="%SystemDrive%\Python27" />
  </Environment>
  <EntryPoint>
    <ProgramEntryPoint commandLine="bin\ps.cmd LaunchWorker.ps1" setReadyOnProcessStart="true" />
  </EntryPoint>
</Runtime>
```

#### <a name="sample-servicedefinitioncsdef"></a>Archivo ServiceDefinition.csdef de ejemplo
```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceDefinition name="AzureCloudServicePython" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2015-04.2.6">
  <WorkerRole name="WorkerRole1" vmsize="Small">
    <ConfigurationSettings>
      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" />
      <Setting name="Python2" />
    </ConfigurationSettings>
    <Startup>
      <Task executionContext="elevated" taskType="simple" commandLine="bin\ps.cmd PrepPython.ps1">
        <Environment>
          <Variable name="EMULATED">
            <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
          </Variable>
          <Variable name="PYTHON2" value="off" />
        </Environment>
      </Task>
      <Task executionContext="elevated" taskType="simple" commandLine="bin\ps.cmd PipInstaller.ps1">
        <Environment>
          <Variable name="EMULATED">
            <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
          </Variable>
          <Variable name="PYTHON2" value="off" />
        </Environment>
      </Task>
    </Startup>
    <Runtime>
      <Environment>
        <Variable name="EMULATED">
          <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
        </Variable>
        <Variable name="PYTHON2" value="off" />
        <Variable name="PYPATH" value="%SystemDrive%\Python27" />
      </Environment>
      <EntryPoint>
        <ProgramEntryPoint commandLine="bin\ps.cmd LaunchWorker.ps1" setReadyOnProcessStart="true" />
      </EntryPoint>
    </Runtime>
    <Imports>
      <Import moduleName="RemoteAccess" />
      <Import moduleName="RemoteForwarder" />
    </Imports>
  </WorkerRole>
</ServiceDefinition>
```



A continuación, cree los archivos **PrepPython.ps1** y **PipInstaller.ps1** en la carpeta **./bin** del rol.

#### <a name="preppythonps1"></a>PrepPython.ps1
Este script instala Python. Si la variable de entorno **PYTHON2** se establece en **on,** (activado) se instala Python 2.7 o, en caso contrario, se instala Python 3.5.

```powershell
[Net.ServicePointManager]::SecurityProtocol = "tls12, tls11, tls"
$is_emulated = $env:EMULATED -eq "true"
$is_python2 = $env:PYTHON2 -eq "on"
$nl = [Environment]::NewLine

if (-not $is_emulated){
    Write-Output "Checking if python is installed...$nl"
    if ($is_python2) {
        & "${env:SystemDrive}\Python27\python.exe"  -V | Out-Null
    }
    else {
        py -V | Out-Null
    }

    if (-not $?) {

        $url = "https://www.python.org/ftp/python/3.5.2/python-3.5.2-amd64.exe"
        $outFile = "${env:TEMP}\python-3.5.2-amd64.exe"

        if ($is_python2) {
            $url = "https://www.python.org/ftp/python/2.7.12/python-2.7.12.amd64.msi"
            $outFile = "${env:TEMP}\python-2.7.12.amd64.msi"
        }

        Write-Output "Not found, downloading $url to $outFile$nl"
        Invoke-WebRequest $url -OutFile $outFile
        Write-Output "Installing$nl"

        if ($is_python2) {
            Start-Process msiexec.exe -ArgumentList "/q", "/i", "$outFile", "ALLUSERS=1" -Wait
        }
        else {
            Start-Process "$outFile" -ArgumentList "/quiet", "InstallAllUsers=1" -Wait
        }

        Write-Output "Done$nl"
    }
    else {
        Write-Output "Already installed"
    }
}
```

#### <a name="pipinstallerps1"></a>PipInstaller.ps1
Este script se llama pip e instala todas las dependencias en el archivo **requirements.txt**. Si la variable de entorno **PYTHON2** se establece en **on,** (activado) se usa Python 2.7 o, en caso contrario, se usa Python 3.5.

```powershell
$is_emulated = $env:EMULATED -eq "true"
$is_python2 = $env:PYTHON2 -eq "on"
$nl = [Environment]::NewLine

if (-not $is_emulated){
    Write-Output "Checking if requirements.txt exists$nl"
    if (Test-Path ..\requirements.txt) {
        Write-Output "Found. Processing pip$nl"

        if ($is_python2) {
            & "${env:SystemDrive}\Python27\python.exe" -m pip install -r ..\requirements.txt
        }
        else {
            py -m pip install -r ..\requirements.txt
        }

        Write-Output "Done$nl"
    }
    else {
        Write-Output "Not found$nl"
    }
}
```

#### <a name="modify-launchworkerps1"></a>Modificación del archivo LaunchWorker.ps1
> [!NOTE]
> En caso de un proyecto de **rol de trabajo**, se necesita el archivo **LauncherWorker.ps1** para ejecutar el archivo de inicio. En un proyecto de **rol web**, el archivo de inicio se define por el contrario en las propiedades del proyecto.
> 
> 

El archivo **bin\LaunchWorker.ps1** se creó originalmente para hacer una gran cantidad de trabajo de preparación pero realmente no funciona. Reemplace el contenido del archivo por el script siguiente.

Este script llama al archivo **worker.py** desde el proyecto de Python. Si la variable de entorno **PYTHON2** se establece en **on,** (activado) se usa Python 2.7 o, en caso contrario, se usa Python 3.5.

```powershell
$is_emulated = $env:EMULATED -eq "true"
$is_python2 = $env:PYTHON2 -eq "on"
$nl = [Environment]::NewLine

if (-not $is_emulated)
{
    Write-Output "Running worker.py$nl"

    if ($is_python2) {
        cd..
        iex "$env:PYPATH\python.exe worker.py"
    }
    else {
        cd..
        iex "py worker.py"
    }
}
else
{
    Write-Output "Running (EMULATED) worker.py$nl"

    # Customize to your local dev environment

    if ($is_python2) {
        cd..
        iex "$env:PYPATH\python.exe worker.py"
    }
    else {
        cd..
        iex "py worker.py"
    }
}
```

#### <a name="pscmd"></a>ps.cmd
Las plantillas de Visual Studio deben haber creado un archivo **ps.cmd** en la carpeta **./bin**. Este script de shell llama a los scripts anteriores del contenedor de PowerShell y proporciona un registro basado en el nombre del contenedor de PowerShell que se ha llamado. Si no se ha creado este archivo, esto es lo que debería haber en él. 

```bat
@echo off

cd /D %~dp0

if not exist "%DiagnosticStore%\LogFiles" mkdir "%DiagnosticStore%\LogFiles"
%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe -ExecutionPolicy Unrestricted -File %* >> "%DiagnosticStore%\LogFiles\%~n1.txt" 2>> "%DiagnosticStore%\LogFiles\%~n1.err.txt"
```



## <a name="run-locally"></a>Ejecución en modo local
Si establece su proyecto de servicio en la nube como proyecto de inicio y presiona F5, el servicio en la nube se ejecuta en el emulador de Azure local.

Aunque PTVS admite el inicio en el emulador, la depuración (por ejemplo, puntos de interrupción) no funciona.

Para depurar roles web y de trabajo, puede establecer el proyecto de rol como proyecto de inicio y depurarlo.  También puede establecer varios proyectos de inicio.  Haga clic con el botón derecho en la solución y luego seleccione **Establecer proyectos de inicio**.

![Propiedades del proyecto de inicio de la solución](./media/cloud-services-python-ptvs/startup.png)

## <a name="publish-to-azure"></a>Publicación en Azure
Para publicar, haga clic con el botón derecho en el proyecto del servicio en la nube de la solución y luego seleccione **Publicar**.

![Inicio de sesión para publicación de Microsoft Azure](./media/cloud-services-python-ptvs/publish-sign-in.png)

Siga las instrucciones del asistente. Si es necesario, habilite el Escritorio remoto. El Escritorio remoto es útil cuando necesita depurar algo.

Cuando haya terminado la configuración, haga clic en **Publicar**.

En la ventana de salida se ve cierto progreso y después se verá la ventana Registro de actividad de Microsoft Azure.

![Ventana Registro de actividad de Microsoft Azure](./media/cloud-services-python-ptvs/publish-activity-log.png)

La implementación tarda varios minutos y después sus roles web y/o de trabajo estarán ejecutándose en Azure.

### <a name="investigate-logs"></a>Investigación de registros
Después de que la máquina virtual del servicio en la nube se inicie e instale Python, puede mirar los registros para buscar mensajes de error. Estos registros se encuentran en la carpeta **C:\Resources\Directory\\{role}\LogFiles**. **PrepPython.err.txt** tiene al menos un error cuando el script intente detectar si está instalado Python y **PipInstaller.err.txt** puede informar acerca de una versión obsoleta de pip.

## <a name="next-steps"></a>Pasos siguientes
Para obtener información detallada sobre el trabajo con roles web y de trabajo en Python Tools para Visual Studio, consulte la documentación de PTVS:

* [Proyectos de servicio en la nube][Cloud Service Projects]

Para más información sobre el uso de servicios de Azure desde roles web y de trabajo, como el uso de Azure Storage o Service Bus, consulte los siguientes artículos:

* [Blob Service][Blob Service]
* [Table service][Table Service]
* [Queue Service][Queue Service]
* [Colas de Service Bus][Service Bus Queues]
* [Temas de Service Bus][Service Bus Topics]

<!--Link references-->

[¿Qué es un servicio en la nube?]: cloud-services-choose-me.md
[execution model-web sites]: ../app-service/overview.md
[execution model-vms]:../virtual-machines/windows/overview.md
[execution model-cloud services]: cloud-services-choose-me.md
[Python Developer Center]: /develop/python/

[Blob Service]:../storage/blobs/storage-python-how-to-use-blob-storage.md
[Queue Service]: ../storage/queues/storage-python-how-to-use-queue-storage.md
[Table Service]:../cosmos-db/table-storage-how-to-use-python.md
[Service Bus Queues]: ../service-bus-messaging/service-bus-python-how-to-use-queues.md
[Service Bus Topics]: ../service-bus-messaging/service-bus-python-how-to-use-topics-subscriptions.md


<!--External Link references-->

[Python Tools for Visual Studio]: https://aka.ms/ptvs
[Python Tools for Visual Studio Documentation]: https://aka.ms/ptvsdocs
[Cloud Service Projects]: https://docs.microsoft.com/visualstudio/python/python-azure-cloud-service-project-template
[Azure SDK Tools for VS 2013]: https://go.microsoft.com/fwlink/?LinkId=746482
[Azure SDK Tools for VS 2015]: https://go.microsoft.com/fwlink/?LinkId=746481
[Azure SDK Tools for VS 2017]: https://go.microsoft.com/fwlink/?LinkId=746483
[Python 2.7 32-bit]: https://www.python.org/downloads/
[Python 3.5 32-bit]: https://www.python.org/downloads/



