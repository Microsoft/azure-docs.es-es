---
title: Solución de problemas con el agente invitado de Microsoft Azure
description: Solución del problema de no funcionamiento del agente invitado de Microsoft Azure
services: virtual-machines-windows
ms.service: virtual-machines-windows
author: dkdiksha
manager: dcscontentpm
editor: ''
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 09/15/2020
ms.author: genli
ms.openlocfilehash: 597ea6e7ff7dbcfcb8a99d4e4de3c1b82915ee07
ms.sourcegitcommit: 6e1124fc25c3ddb3053b482b0ed33900f46464b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90561268"
---
# <a name="troubleshooting-windows-azure-guest-agent"></a>Solución de problemas con el agente invitado de Microsoft Azure

El agente invitado de Microsoft Azure es un agente de máquina virtual (VM). Permite que la máquina virtual se comunique con el controlador de tejido (el servidor físico subyacente en el que se hospeda la máquina virtual) en la dirección IP 168.63.129.16. Se trata de una dirección IP pública virtual que facilita la comunicación. Para más información, vea [¿Qué es la dirección IP 168.63.129.16?](https://docs.microsoft.com/azure/virtual-network/what-is-ip-address-168-63-129-16)

 La máquina virtual que se migra a Azure desde el entorno local o que se crea mediante una imagen personalizada no tiene instalado el agente invitado de Microsoft Azure. En estos casos, tendrá que instalar manualmente el agente de máquina virtual. Para más información sobre cómo instalar el agente de VM, vea [Información general del agente de máquina virtual de Azure](https://docs.microsoft.com/azure/virtual-machines/extensions/agent-windows).

Una vez instalado correctamente el agente invitado de Microsoft Azure, puede ver los siguientes servicios en services.msc en la máquina virtual:
 
- Servicio del agente invitado de Microsoft Azure
- Servicio de telemetría
- Servicio RDAgent

**Servicio del agente invitado de Microsoft Azure**: este servicio es el que es responsable de todo el registro en WAppAgent.log. Este servicio es responsable de configurar varias extensiones y la comunicación entre el invitado y el host. 

**Servicio de telemetría**: este servicio es responsable de enviar los datos de telemetría de la máquina virtual al servidor backend.

**Servicio RDAgent**: este servicio es responsable de la instalación del agente invitado. El instalador transparente también es un componente de RDAgent que ayuda a actualizar otros componentes y servicios del agente invitado. RDAgent también es responsable de enviar latidos de la máquina virtual invitada al agente de host en el servidor físico.

> [!NOTE]
> A partir de la versión 2.7.41491.971 del agente invitado de la máquina virtual, el componente de telemetría se incluye en el servicio RDAgent, por lo que es posible este servicio de telemetría no aparezca en las máquinas virtuales recién creadas.

## <a name="checking-agent-status-and-version"></a>Comprobación de la versión y el estado del agente

Vaya a la página de propiedades de la máquina virtual en Azure Portal y compruebe el **estado del agente**. Si el agente invitado de Microsoft Azure funciona correctamente, el estado se muestra como **Listo**. Si el agente de máquina virtual tiene el estado **No está listo**, las extensiones y el **comando Ejecutar** de Azure Portal no funcionarán.

## <a name="troubleshooting-vm-agent-that-is-in-not-ready-status"></a>Solución de problemas del agente de máquina virtual con el estado No está listo

### <a name="step-1-check-whether-the-windows-azure-guest-agent-service-is-installed"></a>Paso 1: Comprobar si el servicio del agente invitado de Microsoft Azure está instalado

- Comprobar la página

    Busque la carpeta C:\WindowsAzure. Si ve la carpeta GuestAgent que muestra el número de versión, significa que el agente invitado de Microsoft Azure se ha instalado en la máquina virtual. También puede buscar el paquete instalado.  Si el agente invitado de Microsoft Azure está instalado en la máquina virtual, el paquete se guardará en la siguiente ubicación: `C:\windows\OEM\GuestAgent\VMAgentPackage.zip`.
    
    Puede ejecutar el siguiente comando de PowerShell para comprobar si el agente de máquina virtual se ha implementado en la máquina virtual:
    
    `Get-AzVM -ResourceGroupName "RGNAME" -Name "VMNAME" -DisplayHint expand`
    
    En la salida, busque la propiedad **ProvisionVMAgent** y compruebe si el valor está establecido en **True**. Si es así, significa que el agente está instalado en la máquina virtual.
    
- Comprobar los servicios y procesos

   Vaya a la consola de Servicios (services.msc) y compruebe el estado de los siguientes servicios: Servicio del agente invitado de Microsoft Azure, servicio RDAgent, servicio de telemetría de Microsoft Azure y servicio del agente de red de Microsoft Azure.
 
    También puede comprobar si estos servicios se están ejecutando examinando el administrador de tareas para los siguientes procesos:
       
    - WindowsAzureGuestAgent.exe: servicio del agente invitado de Microsoft Azure
    - WaAppAgent.exe: servicio RDAgent
    - WindowsAzureNetAgent.exe: servicio del agente de red de Microsoft Azure
    - WindowsAzureTelemetryService.exe: servicio de telemetría de Microsoft Azure

   Si no puede encontrar estos procesos y servicios, significa que no tiene instalado el agente invitado de Microsoft Azure.

- Comprobar Programas y características

    En el Panel de control, vaya a **Programas y características** para determinar si está instalado el servicio del agente invitado de Microsoft Azure.

Si no encuentra ningún paquete, servicio y proceso en ejecución y no ve siquiera el agente invitado de Microsoft Azure instalado en Programas y características, intente [instalar el servicio del agente invitado de Microsoft Azure](https://docs.microsoft.com/azure/virtual-machines/extensions/agent-windows). Si el agente invitado no se instala correctamente, puede [instalar el agente de máquina virtual sin conexión](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/install-vm-agent-offline).

Si puede ver los servicios y se están ejecutando, reinicie el servicio para ver si se ha resuelto el problema. Si los servicios se detienen, inícielos y espere unos minutos. Después, compruebe si el **Estado del agente** se muestra como **Listo**. Si observa que estos servicios se bloquean, algunos procesos de terceros pueden estar provocando el bloqueo de estos servicios. Para solucionar estos problemas, póngase en contacto con [Soporte técnico de Microsoft](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

### <a name="step-2-check-whether-auto-update-is-working"></a>Paso 2: Comprobar si la actualización automática funciona

El agente invitado de Microsoft Azure tiene una característica de actualización automática. Comprobará automáticamente si hay nuevas actualizaciones y las instalará. Si la característica de actualización automática no funciona correctamente, intente desinstalar e instalar el agente invitado de Microsoft Azure mediante los pasos siguientes:

1. Si el agente invitado de Microsoft Azure aparece en **Programas y características**, desinstálelo.

2. Abra una ventana del símbolo del sistema con privilegios de administrador.

3. Detenga los servicios del agente invitado. Si los servicios no se detienen, debe establecerlos en **inicio manual** y, después, reiniciar la máquina virtual.
    ```
    net stop rdagent
    net stop WindowsAzureGuestAgent
    net stop WindowsAzureTelemetryService
    ```
1. Elimine los servicios del agente invitado:
    ```
    sc delete rdagent
    sc delete WindowsAzureGuestAgent
    sc delete WindowsAzureTelemetryService
    ```
1. En `C:\WindowsAzure`, cree una carpeta denominada OLD.

1. Mueva todas las carpetas denominadas Packages o GuestAgent a la carpeta OLD.

1. Descargue e instale la última versión del paquete de instalación del agente desde [aquí](https://go.microsoft.com/fwlink/?linkid=394789&clcid=0x409). Debe tener derechos de administrador para completar la instalación.

1. Instale el agente invitado con el siguiente comando:

    ```
    msiexec.exe /i c:\VMAgentMSI\WindowsAzureVmAgent.2.7.<version>.fre.msi /quiet /L*v c:\VMAgentMSI\msiexec.log
    ```
    A continuación, compruebe si los servicios del agente invitado se inician correctamente.
 
    En los casos excepcionales en los que el agente invitado no se instala correctamente, puede [instalar el agente de máquina virtual sin conexión](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/install-vm-agent-offline).
    

### <a name="step-3-check-whether-the-vm-can-connect-to-the-fabric-controller"></a>Paso 3: Comprobar si la máquina virtual se puede conectar al controlador de tejido

Use una herramienta como PsPing para probar si la máquina virtual puede conectarse a 168.63.129.16 en los puertos 80, 32526 y 443. Si la máquina virtual no se conecta según lo previsto, compruebe si la comunicación saliente a través de los puertos 80, 443 y 32526 está abierta en el firewall local de la máquina virtual. Si esta dirección IP se bloquea, el agente de máquina virtual puede mostrar un comportamiento inesperado en una variedad de escenarios.

## <a name="advanced-troubleshooting"></a>Pasos detallados para solucionar problemas de conexión a Escritorio remoto a máquinas virtuales Windows en Azure

Los eventos para solucionar problemas del agente invitado de Microsoft Azure se registran en los siguientes archivos de registro:

- C:\WindowsAzure\Logs\WaAppAgent.log
- C:\WindowsAzure\Logs\TransparentInstaller.log

A continuación, se muestran algunos escenarios comunes en los que el agente invitado de Microsoft Azure puede adoptar el estado **No está listo** o dejar de funcionar según lo previsto.

### <a name="agent-stuck-on-starting"></a>El agente se atasca al "Iniciar"

En el registro WaAppAgent, puede ver que el agente está bloqueado en el proceso de inicio y no puede iniciarse.

**Información de registro**
 
[00000007] [28/05/2019 12:58:50.90] [INFO] Inicio de WindowsAzureGuestAgent. Versión 2.7.41491.901

**Análisis**
 
La máquina virtual sigue ejecutando la versión anterior del agente invitado de Microsoft Azure. En la carpeta C:\WindowsAzure, puede observar que hay instaladas varias instancias del agente invitado de Microsoft Azure, incluidas varias de la misma versión. Dado que hay varias instancias del agente instaladas, la máquina virtual no inicia la última versión del agente invitado de Microsoft Azure.

**Solución**

Desinstale manualmente el agente invitado de Microsoft Azure y, a continuación, vuelva a instalarlo siguiendo estos pasos:

1. Abra el panel de control > Programas y características y, después, desinstale el agente invitado de Microsoft Azure.
1. Abra el administrador de tareas y detenga los siguientes servicios: Servicio del agente invitado de Microsoft Azure, servicio RDAgent, servicio de telemetría de Microsoft Azure y servicio del agente de red de Microsoft Azure.
1. En C:\WindowsAzure, cree una carpeta denominada OLD.
1. Mueva todas las carpetas denominadas Packages o GuestAgent a la carpeta OLD. Además, mueva cualquiera de las carpetas GuestAgent de C:\WindowsAzure\logs que comienzan como GuestAgent_x.x.xxxxx a la carpeta OLD.
1. Descargue e instale la última versión del agente de MSI. Debe tener derechos de administrador para completar la instalación.
1. Instale el agente invitado con el siguiente comando MSI:
    ```
    msiexec.exe /i c:\VMAgentMSI\WindowsAzureVmAgent.2.7.<version>.fre.msi /quiet /L*v c:\VMAgentMSI\msiexec.log`
    ```
1. Compruebe que RDAgent, el agente invitado de Microsoft Azure y los servicios de telemetría de Microsoft Azure se están ejecutando ahora.
 
1. Compruebe WaAppAgent.log para asegurarse de que se está ejecutando la última versión del agente invitado de Microsoft Azure.
 
1. Elimine la carpeta OLD de C:\WindowsAzure.
  
### <a name="unable-to-connect-to-wireserver-ip-host-ip"></a>No se puede conectar con la dirección IP de WireServer (IP de host) 

Observa las siguientes entradas de error en WaAppAgent.log y Telemetry.log:

**Información de registro**

```Log sample
[ERROR] GetVersions() failed with exception: Microsoft.ServiceModel.Web.WebProtocolException: Server Error: Service Unavailable (ServiceUnavailable) ---> 
System.Net.WebException: The remote server returned an error: (503) Server Unavailable.
```
```Log sample
[00000011] [12/11/2018 06:27:55.66] [WARN]  (Ignoring) Exception while fetching supported versions from HostGAPlugin: System.Net.WebException: Unable to connect to the remote server 
---> System.Net.Sockets.SocketException: An attempt was made to access a socket in a way forbidden by its access permissions 168.63.129.16:32526
at System.Net.Sockets.Socket.DoConnect(EndPoint endPointSnapshot, SocketAddress socketAddress)
at System.Net.ServicePoint.ConnectSocketInternal(Boolean connectFailure, Socket s4, Socket s6, Socket& socket, IPAddress& address, ConnectSocketState status, IAsyncResult asyncResult, Exception& exception)
--- End of inner exception stack trace ---
at System.Net.WebClient.DownloadDataInternal(Uri address, WebRequest& request)
at System.Net.WebClient.DownloadString(Uri address)
at Microsoft.GuestAgentHostPlugin.Client.GuestInformationServiceClient.GetVersions()
at Microsoft.WindowsAzure.GuestAgent.ContainerStateMachine.HostGAPluginUtility.UpdateHostGAPluginAvailability()`
```
**Análisis**

La máquina virtual no puede acceder al servidor host de WireServer.

**Solución**

1. Dado que no se puede acceder a WireServer, conéctese a la máquina virtual mediante Escritorio remoto y, a continuación, intente acceder a la siguiente dirección URL desde un explorador de Internet: http://168.63.129.16/?comp=versions. 
1. Si no puede acceder a la dirección URL del paso 1, compruebe la interfaz de red para determinar si está establecida como habilitada para DHCP y si tiene DNS. Para comprobar el estado de DHCP de la interfaz de red, ejecute el siguiente comando: `netsh interface ip show config`.
1. Si DHCP está deshabilitado, ejecute lo siguiente asegurándose de cambiar el valor en amarillo por el nombre de la interfaz: `netsh interface ip set address name="Name of the interface" source=dhcp`.
1. Compruebe si hay algún problema provocado por un firewall, un proxy u otro origen que pueda estar bloqueando el acceso a la dirección IP 168.63.129.16.
1. Compruebe si el Firewall de Windows o un firewall de terceros está bloqueando el acceso a los puertos 80, 443 y 32526. Para más información sobre por qué no se debe bloquear esta dirección, vea [¿Qué es la dirección IP 168.63.129.16?](https://docs.microsoft.com/azure/virtual-network/what-is-ip-address-168-63-129-16)

### <a name="guest-agent-is-stuck-stopping"></a>El agente invitado está bloqueado "deteniéndose"  

Observa las siguientes entradas de error en WaAppAgent.log:

**Información de registro** 

```Log sample
[00000007] [07/18/2019 14:46:28.87] [WARN] WindowsAzureGuestAgent stopping.
[00000007] [07/18/2019 14:46:28.89] [INFO] Uninitializing StateExecutor with WaitForTerminalStateReachedOnEnd : True
[00000004] [07/18/2019 14:46:28.89] [WARN] WindowsAzureGuestAgent could not be stopped. Exception: System.NullReferenceException: Object reference not set to an instance of an object.
at Microsoft.WindowsAzure.GuestAgent.ContainerStateMachine.GoalStateExecutorBase.WaitForExtensionWorkflowComplete(Boolean WaitForTerminalStateReachedOnEnd)
at Microsoft.WindowsAzure.GuestAgent.ContainerStateMachine.GoalStateExecutorBase.Uninitialize(Boolean WaitForTerminalStateReachedOnEnd)
at Microsoft.WindowsAzure.GuestAgent.ContainerStateMachine.GoalStateExecutorForCloud.Uninitialize(Boolean WaitForTerminalStateReachedOnEnd)
at Microsoft.WindowsAzure.GuestAgent.AgentCore.AgentCore.Stop(Boolean waitForTerminalState)
at Microsoft.WindowsAzure.GuestAgent.AgentCore.AgentService.DoStopService()
at Microsoft.WindowsAzure.GuestAgent.AgentCore.AgentService.<>c__DisplayClass2.<OnStopProcessing>b__1()
```
**Análisis**

El agente invitado de Microsoft Azure está bloqueado en el proceso de detención.

**Solución**

1. Asegúrese de que WaAppAgent.exe se ejecuta en la máquina virtual. Si no se está ejecutando, reinicie el servicio rdgagent y espere cinco minutos. Cuando WaAppAgent.exe está en ejecución, finalice el proceso WindowsAzureGuest.exe.
2. Si el paso 1 no resuelve el problema, quite la versión instalada actualmente e instale manualmente la última versión del agente.

### <a name="npcap-loopback-adapter"></a>Adaptador de bucle invertido Npcap 

Observa las siguientes entradas de error en WaAppAgent.log:
 
```Log sample
[00000006] [06/20/2019 07:44:28.93] [INFO]  Attempting to discover fabric address on interface Npcap Loopback Adapter.
[00000024] [06/20/2019 07:44:28.93] [WARN]  Empty DHCP option data returned
[00000006] [06/20/2019 07:44:28.93] [ERROR] Did not discover fabric address on interface Npcap Loopback Adapter
```
**Análisis**

Wireshark instala el adaptador de bucle invertido Npcap en la máquina virtual. Wireshark es una herramienta de código abierto para la generación de perfiles del tráfico de red y el análisis de paquetes. Esta herramienta suele denominarse analizador de red, analizador de protocolos de red o analizador de protocolos (sniffer).

**Solución**

WireShark probablemente instala el adaptador de bucle invertido Npcap. Pruebe a deshabilitarlo y, después, compruebe si se ha resuelto el problema.

## <a name="next-steps"></a>Pasos siguientes

Para solucionar el problema "El agente invitado de Microsoft Azure no funciona", [póngase en contacto con Soporte técnico de Microsoft](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).
