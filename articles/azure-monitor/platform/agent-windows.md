---
title: Instalación del agente de Log Analytics en equipos Windows
description: En este artículo se describe la manera de conectar equipos Windows hospedados en otras nubes o de forma local en Azure Monitor mediante el agente de Log Analytics de Windows.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/03/2020
ms.openlocfilehash: d283c2b2cdbbeb3ef4bc4e25f4288dfd95158552
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89003378"
---
# <a name="install-log-analytics-agent-on-windows-computers"></a>Instalación del agente de Log Analytics en equipos Windows
En este artículo se proporcionan detalles sobre cómo instalar el agente de Log Analytics en equipos Windows mediante los siguientes métodos:

* Instalación manual mediante el [asistente para la instalación](#install-agent-using-setup-wizard) o la [línea de comandos](#install-agent-using-command-line).
* [Desired State Configuration (DSC) de Azure Automation](#install-agent-using-dsc-in-azure-automation) 

>[!IMPORTANT]
> Los métodos de instalación que se describen en este artículo se usan normalmente para máquinas virtuales locales o en otras nubes. Consulte [Opciones de instalación](log-analytics-agent.md#installation-options) para ver las opciones más eficaces que puede usar para máquinas virtuales de Azure.

> [!NOTE]
> Si tiene que configurar el agente para que informe a varias áreas de trabajo, esto no puede realizarse durante la instalación inicial, solo después actualizando la configuración del Panel de control o PowerShell, tal como se describe en [Adición o eliminación de un área de trabajo](agent-manage.md#adding-or-removing-a-workspace).  

## <a name="supported-operating-systems"></a>Sistemas operativos admitidos

Consulte [Información general de los agentes de Azure Monitor](agents-overview.md#supported-operating-systems) para obtener una lista de las versiones de Windows admitidas por el agente de Log Analytics.

### <a name="sha-2-code-signing-support-requirement"></a>Requisito de compatibilidad con la firma de código SHA-2 para Windows 
El agente de Windows empezará a usar exclusivamente la firma SHA-2 el 17 de agosto de 2020. Este cambio afectará a los clientes que usan el agente de Log Analytics en un sistema operativo heredado como parte de cualquier servicio de Azure (Azure Monitor, Azure Automation, Azure Update Management, Azure Change Tracking, Azure Security Center, Azure Sentinel, ATP de Windows Defender). El cambio no requiere ninguna acción del cliente a menos que esté ejecutando el agente en una versión de sistema operativo heredada (Windows 7, Windows Server 2008 R2 y Windows Server 2008). Los clientes que ejecutan una versión de sistema operativo heredada deben realizar las siguientes acciones en sus máquinas antes del 17 de agosto de 2020 o los agentes dejarán de enviar datos a las áreas de trabajo de Log Analytics:

1. Instale el Service Pack más reciente para su sistema operativo. Las versiones de Service Pack necesarias son:
    - Windows 7 SP1
    - Windows Server 2008 SP2
    - Windows Server 2008 R2 SP1

2. Instale las actualizaciones de firma SHA-2 de Windows para su sistema operativo, como se describe en [Requisito de compatibilidad con la firma de código SHA-2 para Windows y WSUS de 2019](https://support.microsoft.com/help/4472027/2019-sha-2-code-signing-support-requirement-for-windows-and-wsus).
3. Actualice el agente de Windows a la versión más reciente (versión 10.20.18029).
4. Se recomienda para configurar el agente para [usar TLS 1.2](agent-windows.md#configure-agent-to-use-tls-12). 

## <a name="network-requirements"></a>Requisitos de red
Consulte [Introducción al agente de Log Analytics](log-analytics-agent.md#network-requirements) para conocer los requisitos de red para el agente de Windows.


   
## <a name="configure-agent-to-use-tls-12"></a>Configuración del agente para usar TLS 1.2
El protocolo [TLS 1.2](/windows-server/security/tls/tls-registry-settings#tls-12) garantiza la seguridad de los datos en tránsito para la comunicación entre el agente de Windows y el servicio de Log Analytics. Si va a realizar la instalación en un [sistema operativo sin TLS 1.2 habilitado de forma predeterminada](data-security.md#sending-data-securely-using-tls-12), debe configurar TLS 1.2 mediante los pasos siguientes.

1. Busque la siguiente subclave del Registro: **HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols**
2. Cree una subclave en **Protocolos** para TLS 1.2 **HKLM\System\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2**
3. Cree una subclave **Cliente** debajo de la subclave de la versión del protocolo TLS 1.2 que ha creado anteriormente. Por ejemplo, **HKLM\System\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client**.
4. Cree los siguientes valores DWORD en **HKLM\System\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client**:

    * **Enabled** [Value = 1]
    * **DisabledByDefault** [Value = 0]  

Configure .NET Framework 4.6 o posterior para que sea compatible con la criptografía segura, ya que, de forma predeterminada, está desactivada. La [criptografía fuerte](/dotnet/framework/network-programming/tls#schusestrongcrypto) utiliza protocolos de red más seguros como TLS 1.2, y bloquea los protocolos que no son seguros. 

1. Busque la siguiente subclave del Registro: **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\\.NETFramework\v4.0.30319**.  
2. Cree el valor DWORD **SchUseStrongCrypto** bajo esta subclave con un valor de **1**.  
3. Busque la siguiente subclave del Registro: **HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\\.NETFramework\v4.0.30319**.  
4. Cree el valor DWORD **SchUseStrongCrypto** bajo esta subclave con un valor de **1**. 
5. Reinicie el sistema para que la configuración surta efecto. 

## <a name="install-agent-using-setup-wizard"></a>Instalación del agente mediante el asistente para instalación
Los siguientes pasos permiten la instalación y configuración del agente de Log Analytics en Azure y en la nube de Azure Government mediante el asistente para la configuración del agente en su equipo. Si desea aprender a configurar el agente para que también informe a un grupo de administración de System Center Operations Manager, consulte [Para implementar el agente de Operations Manager con el Asistente para instalación del agente](/system-center/scom/manage-deploy-windows-agent-manually#to-deploy-the-operations-manager-agent-with-the-agent-setup-wizard).

1. En el área de trabajo de Log Analytics, en la página **Servidores Windows** a la que navegó anteriormente, seleccione la versión **Descargar el agente de Windows** que desee descargar según la arquitectura del procesador del sistema operativo Windows.   
2. Ejecute el programa de instalación para instalar al agente en el equipo.
2. En la página **principal**, haga clic en **Siguiente**.
3. En la página **Términos de licencia**, lea la licencia y haga clic en **Acepto**.
4. En la página e **Carpeta de destino**, cambie o mantenga la carpeta de instalación predeterminada y haga clic en **Siguiente**.
5. En la página **Opciones de instalación del agente**, elija la opción para conectar el agente a Azure Log Analytics y luego haga clic en **Siguiente**.   
6. En la página **Azure Log Analytics**, realice lo siguiente:
   1. Pegue el **Id. del área de trabajo** y la **clave del área de trabajo (clave principal)** que copió anteriormente.  Si el equipo tiene que notificar a un área de trabajo de Log Analytics en Azure Government Cloud, seleccione **Azure US Gov** desde la lista desplegable **Azure Cloud**.  
   2. Si el equipo necesita comunicarse a través de un servidor proxy con el servicio de Log Analytics, haga clic en **Avanzado** y proporcione la dirección URL y el número de puerto del servidor proxy.  Si el servidor proxy requiere autenticación, escriba el nombre de usuario y la contraseña para autenticar con el servidor proxy y, luego, haga clic en **Siguiente**.  
7. Haga clic en **Siguiente** cuando haya terminado de proporcionar las opciones de configuración necesarias.<br><br> ![pegar identificador del área de trabajo y clave principal](media/agent-windows/log-analytics-mma-setup-laworkspace.png)<br><br>
8. En la página **Preparado para instalar**, revise las opciones seleccionadas y haga clic en **Instalar**.
9. En la página **La configuración finalizó correctamente**, haga clic en **Finalizar**.

Una vez completado el proceso, el **Agente de administración de Microsoft** aparece en el **Panel de control**. Para confirmar que está informando a Log Analytics, revise [Comprobar la conectividad del agente con Log Analytics](#verify-agent-connectivity-to-azure-monitor). 

## <a name="install-agent-using-command-line"></a>Instalación del agente mediante la línea de comandos
El archivo descargado del agente es un paquete de instalación independiente.  El programa de instalación del agente y los archivos auxiliares se encuentran en el paquete y deben extraerse para realizar la instalación correctamente mediante la línea de comandos que se muestra en los ejemplos siguientes.    

>[!NOTE]
>Si desea actualizar a un agente, debe utilizar la API de scripting de Log Analytics. Si quiere obtener más información, consulte [Administrar y mantener el agente de Log Analytics para Windows y Linux](agent-manage.md).

En la tabla siguiente se resaltan los parámetros específicos que admite la configuración del agente, aunque la implementación se haya realizado mediante DSC de Automatización.

|Opciones específicas de MMA                   |Notas         |
|---------------------------------------|--------------|
| NOAPM=1                               | Parámetro opcional. Instala al agente sin la supervisión de rendimiento de aplicaciones .NET.|   
|ADD_OPINSIGHTS_WORKSPACE               | 1 = Configurar el agente para informar a un área de trabajo                |
|OPINSIGHTS_WORKSPACE_ID                | Identificador de área de trabajo (guid) para el área de trabajo que se agregará                    |
|OPINSIGHTS_WORKSPACE_KEY               | Clave del área de trabajo que se usa para autenticar inicialmente con el área de trabajo |
|OPINSIGHTS_WORKSPACE_AZURE_CLOUD_TYPE  | Especificar el entorno en la nube donde se encuentra el área de trabajo <br> 0 = nube comercial de Azure (valor predeterminado) <br> 1= Azure Government |
|OPINSIGHTS_PROXY_URL               | Identificador URI del proxy que se va a usar |
|OPINSIGHTS_PROXY_USERNAME               | Nombre de usuario para acceder a un proxy autenticado |
|OPINSIGHTS_PROXY_PASSWORD               | Contraseña para acceder a un proxy autenticado |

1. Para extraer los archivos de instalación del agente, desde un símbolo del sistema con privilegios elevados ejecute `MMASetup-<platform>.exe /c` y se le solicitará la ruta en la que extraer los archivos.  Como alternativa, puede especificar la ruta de acceso pasando los argumentos `MMASetup-<platform>.exe /c /t:<Full Path>`.  
2. Para instalar el agente de forma silenciosa y configurarlo para que informe a un área de trabajo en la nube comercial de Azure, vaya a la carpeta en la que extrajo los archivos de instalación y escriba: 
   
     ```dos
    setup.exe /qn NOAPM=1 ADD_OPINSIGHTS_WORKSPACE=1 OPINSIGHTS_WORKSPACE_AZURE_CLOUD_TYPE=0 OPINSIGHTS_WORKSPACE_ID="<your workspace ID>" OPINSIGHTS_WORKSPACE_KEY="<your workspace key>" AcceptEndUserLicenseAgreement=1
    ```

   o bien, para configurar el agente para que informe a la nube de la administración pública de EE. UU., escriba: 

     ```dos
    setup.exe /qn NOAPM=1 ADD_OPINSIGHTS_WORKSPACE=1 OPINSIGHTS_WORKSPACE_AZURE_CLOUD_TYPE=1 OPINSIGHTS_WORKSPACE_ID="<your workspace ID>" OPINSIGHTS_WORKSPACE_KEY="<your workspace key>" AcceptEndUserLicenseAgreement=1
    ```
    >[!NOTE]
    >Los valores de cadena para los parámetros *OPINSIGHTS_WORKSPACE_ID* y *OPINSIGHTS_WORKSPACE_KEY* se deben encapsular en comillas dobles para indicar a Windows Installer que los interprete como opciones válidas para el paquete. 

## <a name="install-agent-using-dsc-in-azure-automation"></a>Instalación del agente mediante DSC en Azure Automation

Puede usar el siguiente script de ejemplo para instalar el agente mediante DSC de Azure Automation.   Si no tiene una cuenta de Automation, consulte [Introducción a Azure Automation](../../automation/index.yml) para comprobar los requisitos y los pasos necesarios para crear una cuenta de Automation antes de poder usar DSC de Automatización.  Si no está familiarizado con DSC de Automatización, consulte [Introducción a DSC de Automatización](../../automation/automation-dsc-getting-started.md).

En el ejemplo siguiente se instala el agente de 64 bits, identificado mediante el valor `URI`. También puede utilizar la versión de 32 bits si se reemplaza el valor del identificador URI. Estos son los identificadores URI para ambas versiones:

- Agente de Windows de 64 bits: https://go.microsoft.com/fwlink/?LinkId=828603
- Agente de Windows de 32 bits: https://go.microsoft.com/fwlink/?LinkId=828604


>[!NOTE]
>Este procedimiento y el script de ejemplo no admiten la actualización del agente que ya está implementado en un equipo Windows.

Las versiones de 32 y 64 bits del paquete de agente tienen códigos de producto diferentes y las nuevas versiones publicadas también tienen un valor único.  El código de producto es un GUID, que es la identificación principal de una aplicación o un producto, y se representa mediante la propiedad **ProductCode** de Windows Installer.  El valor `ProductId` del script **MMAgent.ps1** tiene que coincidir con el código de producto del paquete del instalador del agente de 32 o 64 bits.

Para recuperar el código de producto del paquete para instalar el agente directamente, puede usar Orca.exe desde la página [Windows SDK Components for Windows Installer Developers](/windows/win32/msi/platform-sdk-components-for-windows-installer-developers) (Componentes del SDK de Windows para desarrolladores de Windows Installer), donde encontrará un componente del Kit de desarrollo de software de Windows, o mediante PowerShell y un [script de ejemplo](https://www.scconfigmgr.com/2014/08/22/how-to-get-msi-file-information-with-powershell/) que haya escrito un MVP (profesional más valioso) de Microsoft.  Para cualquier enfoque, primero debe extraer el archivo **MOMagent.msi** del paquete de instalación MMASetup.  Esto se explicó anteriormente en el primero paso de la sección [Instalación del agente a través de la línea de comandos](#install-agent-using-command-line).  

1. Importe el módulo DSC xPSDesiredStateConfiguration de [https://www.powershellgallery.com/packages/xPSDesiredStateConfiguration](https://www.powershellgallery.com/packages/xPSDesiredStateConfiguration) en Azure Automation.  
2.    En Azure Automation, cree los recursos de variable *OPSINSIGHTS_WS_ID* y *OPSINSIGHTS_WS_KEY*. Establezca *OPSINSIGHTS_WS_ID* en el identificador del área de trabajo de Log Analytics y *OPSINSIGHTS_WS_KEY* en la clave principal del área de trabajo.
3.    Copie el script y guárdelo como MMAgent.ps1.

```powershell
Configuration MMAgent
{
    $OIPackageLocalPath = "C:\Deploy\MMASetup-AMD64.exe"
    $OPSINSIGHTS_WS_ID = Get-AutomationVariable -Name "OPSINSIGHTS_WS_ID"
    $OPSINSIGHTS_WS_KEY = Get-AutomationVariable -Name "OPSINSIGHTS_WS_KEY"

    Import-DscResource -ModuleName xPSDesiredStateConfiguration
    Import-DscResource -ModuleName PSDesiredStateConfiguration

    Node OMSnode {
        Service OIService
        {
            Name = "HealthService"
            State = "Running"
            DependsOn = "[Package]OI"
        }

        xRemoteFile OIPackage {
            Uri = "https://go.microsoft.com/fwlink/?LinkId=828603"
            DestinationPath = $OIPackageLocalPath
        }

        Package OI {
            Ensure = "Present"
            Path  = $OIPackageLocalPath
            Name = "Microsoft Monitoring Agent"
            ProductId = "8A7F2C51-4C7D-4BFD-9014-91D11F24AAE2"
            Arguments = '/C:"setup.exe /qn NOAPM=1 ADD_OPINSIGHTS_WORKSPACE=1 OPINSIGHTS_WORKSPACE_ID=' + $OPSINSIGHTS_WS_ID + ' OPINSIGHTS_WORKSPACE_KEY=' + $OPSINSIGHTS_WS_KEY + ' AcceptEndUserLicenseAgreement=1"'
            DependsOn = "[xRemoteFile]OIPackage"
        }
    }
}

```

4. Actualice el valor `ProductId` en el script con el código de producto extraído de la versión más reciente del paquete de instalación del agente mediante los métodos recomendados anteriormente. 
5. [Importe el script de configuración MMAgent.ps1](../../automation/automation-dsc-getting-started.md#import-a-configuration-into-azure-automation) en su cuenta de Automation. 
6. [Asigne un equipo Windows o un nodo](../../automation/automation-dsc-getting-started.md#enable-an-azure-resource-manager-vm-for-management-with-state-configuration) a la configuración. Durante los 15 minutos siguientes, el nodo comprueba la configuración y el agente se inserta en el nodo.

## <a name="verify-agent-connectivity-to-azure-monitor"></a>Comprobación de la conectividad del agente a Azure Monitor

Una vez completada la instalación del agente, compruebe que se ha conectado correctamente y que la creación de informes puede realizarse desde ambos lados.  

En el **Panel de control** del equipo, busque el elemento **Microsoft Monitoring Agent**.  Selecciónelo y, en la pestaña **Azure Log Analytics**, el agente debe mostrar un mensaje que indica: **The Microsoft Monitoring Agent has successfully connected to the Microsoft Operations Management Suite service** (Microsoft Monitoring Agent se ha conectado correctamente al servicio Microsoft Operations Management Suite).<br><br> ![Estado de la conexión de MMA en Log Analytics](media/agent-windows/log-analytics-mma-laworkspace-status.png)

También puede realizar una consulta de registros sencilla en Azure Portal.  

1. En Azure Portal, busque y seleccione **Monitor**.
1. Seleccione **Registros** en el menú.
1. En el panel de**Registros**, en el campo tipo de consulta:  

    ```
    Heartbeat 
    | where Category == "Direct Agent" 
    | where TimeGenerated > ago(30m)  
    ```

En los resultados de la búsqueda devueltos, verá los registros de latido del equipo que indican que está conectado y que está creando informes para el servicio.

## <a name="cache-information"></a>Información de caché

Los datos del agente de Log Analytics se almacenan en la memoria caché del equipo local en *C:\Program Files\Microsoft Monitoring Agent\Agent\Health Service State* antes de enviarse a Azure Monitor. El agente intenta realizar la carga cada 20 segundos. En caso de que se produzca un error, esperará una cantidad de tiempo que aumenta exponencialmente hasta que se ejecute correctamente. Esperará 30 segundos antes del segundo intento, 60 segundos antes del siguiente, 120 segundos y así sucesivamente hasta un máximo de 8,5 horas entre los reintentos hasta que se vuelva a conectar correctamente. Este tiempo de espera es ligeramente aleatorio para evitar que todos los agentes intenten conectarse simultáneamente. Los datos más antiguos se descartan cuando se alcanza el búfer máximo.

El tamaño de caché predeterminado es 50 MB, pero se puede configurar entre un mínimo de 5 MB y un máximo de 1,5 GB. Se almacena en la clave del Registro *HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\HealthService\Parameters\Persistence Cache Maximum*. El valor representa el número de páginas, con 8 KB por página.


## <a name="next-steps"></a>Pasos siguientes

- Revise [Administrar y mantener el agente de Log Analytics para Windows y Linux](agent-manage.md) para obtener información sobre cómo volver a configurar, actualizar o quitar el agente de la máquina virtual.

- Consulte [Solución de problemas del agente de Windows](agent-windows-troubleshoot.md) si encuentra incidencias durante la instalación o la administración del agente.
