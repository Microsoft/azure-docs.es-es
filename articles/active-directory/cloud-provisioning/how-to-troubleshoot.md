---
title: Solución de problemas de aprovisionamiento en la nube de Azure AD Connect
description: En este artículo se describe cómo solucionar los problemas que pueden surgir con el agente de aprovisionamiento en la nube.
author: billmath
ms.author: billmath
manager: daveba
ms.date: 12/02/2019
ms.topic: how-to
ms.prod: windows-server-threshold
ms.technology: identity-adfs
ms.openlocfilehash: 34796a435536a48100b7434ed5267802cd2d549f
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89226954"
---
# <a name="cloud-provisioning-troubleshooting"></a>Solución de problemas de aprovisionamiento en la nube

El aprovisionamiento en la nube afecta a muchos aspectos distintos y tiene muchas dependencias diferentes. Este amplio ámbito puede dar lugar a varios problemas. Este artículo le ayuda a solucionar estos problemas. Presenta las áreas típicas en las que puede centrarse, cómo recopilar información adicional, así como las distintas técnicas que puede usar para realizar un seguimiento de los problemas.


## <a name="common-troubleshooting-areas"></a>Áreas comunes de solución de problemas

|Nombre|Descripción|
|-----|-----|
|[Problemas del agente](#agent-problems)|Compruebe que el agente se ha instalado correctamente y que se comunica con Azure Active Directory (Azure AD).|
|[Problemas de sincronización de objetos](#object-synchronization-problems)|Use registros de aprovisionamiento para solucionar problemas de sincronización de objetos.|
|[Problemas de aprovisionamiento en cuarentena](#provisioning-quarantined-problems)|Obtenga información sobre los problemas de aprovisionamiento en cuarentena y cómo corregirlos.|


## <a name="agent-problems"></a>Problemas del agente
Algunas de los primeros aspectos que debe comprobar con el agente son las siguientes:

-  ¿Está instalado?
-  ¿Se ejecuta localmente?
-  ¿Está en el portal?
-  ¿Está marcado como correcto?

Estos aspectos se pueden comprobar en Azure Portal y en el servidor local que ejecuta el agente.

### <a name="azure-portal-agent-verification"></a>Comprobación del agente en Azure Portal

Para comprobar que Azure ve el agente y que este es correcto, siga estos pasos.

1. Inicie sesión en Azure Portal.
1. En la parte izquierda, seleccione **Azure Active Directory** > **Azure AD Connect**. En el centro, seleccione **Administración del aprovisionamiento (versión preliminar)** .
1. En la pantalla **Aprovisionamiento de Azure AD (versión preliminar)** , seleccione **Revisar todos los agentes**.

   ![Revisar todos los agentes](media/how-to-install/install7.png)</br>
 
1. En la pantalla **Agentes de aprovisionamiento locales**, verá los agentes que ha instalado. Compruebe que el agente en cuestión está ahí y que se ha marcado como *Correcto*.

   ![Pantalla de agentes de aprovisionamiento local](media/how-to-install/install8.png)</br>

### <a name="verify-the-port"></a>Comprobación del puerto

Compruebe que Azure está escuchando en el puerto 443 y que el agente puede comunicarse con él. 

En esta prueba se comprobará que los agentes pueden comunicarse con Azure a través del puerto 443. Abra un explorador y vaya a la dirección URL anterior desde el servidor en el que está instalado el agente.

![Comprobación de disponibilidad de puertos](media/how-to-install/verify2.png)

### <a name="on-the-local-server"></a>En el servidor local

Para comprobar que el agente se ejecuta, siga estos pasos.

1. En el servidor con el agente instalado, abra **Servicios**. Para ello, vaya ahí o a **Inicio** > **Ejecutar** > **Services.msc**.
1. En **Servicios**, asegúrese de que tanto el **Actualizador del Agente de Microsoft Azure AD Connect** como el **Agente de aprovisionamiento de Microsoft Azure AD Connect** están ahí y que su estado es *En ejecución*.

   ![Pantalla de servicios](media/how-to-troubleshoot/troubleshoot1.png)

### <a name="common-agent-installation-problems"></a>Problemas comunes de instalación del agente

En las secciones siguientes se describen algunos problemas comunes de instalación de agentes y las soluciones típicas.

#### <a name="agent-failed-to-start"></a>El agente no se pudo iniciar

Puede recibir un mensaje de error que indique lo siguiente:

**No se pudo iniciar el servicio "Agente de aprovisionamiento de Microsoft Azure AD Connect". Compruebe que dispone de suficientes privilegios para iniciar servicios del sistema".** 

Este problema suele deberse a que una directiva de grupo impidió que se aplicaran permisos a la cuenta de inicio de sesión del servicio NT local creada por el instalador (NT SERVICE\AADConnectProvisioningAgent). Estos permisos son necesarios para iniciar el servicio.

Para solucionar este problema, siga estos pasos.

1. Inicie sesión en el servidor con una cuenta de administrador.
1. Abra **Servicios**. Para ello, vaya ahí o a **Inicio** > **Ejecutar** > **Services.msc**.
1. En **Servicios**, haga doble clic en **Agente de aprovisionamiento de Microsoft Azure AD Connect**.
1. En la pestaña **Iniciar sesión**, cambie **Esta cuenta** a un administrador de dominio. A continuación, reinicie el servicio. 

   ![Pestaña Inicio de sesión](media/how-to-troubleshoot/troubleshoot3.png)

#### <a name="agent-times-out-or-certificate-is-invalid"></a>El agente agota el tiempo de espera o el certificado no es válido

Al intentar registrar el agente, es posible que reciba el siguiente mensaje de error.

![Mensaje de error de tiempo de espera](media/how-to-troubleshoot/troubleshoot4.png)

Este problema suele deberse a que el agente no puede conectarse al servicio de identidad híbrida y requiere configurar un proxy HTTP. Para solucionar este problema, configure un proxy de salida. 

El agente de aprovisionamiento admite el uso de un proxy de salida. Puede configurarlo al editar el archivo de configuración del agente *C:\Archivos de programa\Microsoft Azure AD Connect Provisioning Agent\AADConnectProvisioningAgent.exe.config*. Agregue al archivo las siguientes líneas, hacia el final del archivo, justo antes de la etiqueta de cierre `</configuration>`.
Reemplace las variables `[proxy-server]` y `[proxy-port]` con los valores del puerto y el nombre del servidor proxy.

```xml
    <system.net>
        <defaultProxy enabled="true" useDefaultCredentials="true">
            <proxy
                usesystemdefault="true"
                proxyaddress="http://[proxy-server]:[proxy-port]"
                bypassonlocal="true"
            />
        </defaultProxy>
    </system.net>
```

#### <a name="agent-registration-fails-with-security-error"></a>Error de seguridad al registrar el agente

Es posible que reciba un mensaje de error al instalar el agente de aprovisionamiento en la nube.

Este problema suele deberse a que el agente no puede ejecutar los scripts de registro de PowerShell debido a las directivas de ejecución locales de PowerShell.

Para solucionar este problema, cambie las directivas de ejecución de PowerShell en el servidor. Debe establecer las directivas de máquina y usuario en *Undefined* o *RemoteSigned*. Si se establece como *Unrestricted*, verá este error. Para más información, consulte las [directivas de ejecución de PowerShell](/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-6). 

### <a name="log-files"></a>Archivos de registro

De manera predeterminada, el agente emite mensajes de error mínimos e información de seguimiento de la pila. Puede encontrar estos registros de seguimiento en la carpeta *C:\ProgramData\Microsoft\Azure AD Connect Provisioning Agent\Trace*.

Para recopilar información adicional para solucionar problemas relacionados con el agente, siga estos pasos.

1. Detenga el servicio **Agente de aprovisionamiento de Microsoft Azure AD Connect**.
1. Cree una copia del archivo de configuración original: *C:\Archivos de programa\Microsoft Azure AD Connect Provisioning Agent\AADConnectProvisioningAgent.exe.config*.
1. Reemplace la sección `<system.diagnostics>` existente por lo siguiente y todos los mensajes de seguimiento irán al archivo *ProvAgentTrace.log*.

   ```xml
     <system.diagnostics>
         <sources>
         <source name="AAD Connect Provisioning Agent">
             <listeners>
             <add name="console"/>
             <add name="etw"/>
             <add name="textWriterListener"/>
             </listeners>
         </source>
         </sources>
         <sharedListeners>
         <add name="console" type="System.Diagnostics.ConsoleTraceListener" initializeData="false"/>
         <add name="etw" type="System.Diagnostics.EventLogTraceListener" initializeData="Azure AD Connect Provisioning Agent">
             <filter type="System.Diagnostics.EventTypeFilter" initializeData="All"/>
         </add>
         <add name="textWriterListener" type="System.Diagnostics.TextWriterTraceListener" initializeData="C:/ProgramData/Microsoft/Azure AD Connect Provisioning Agent/Trace/ProvAgentTrace.log"/>
         </sharedListeners>
     </system.diagnostics>
    
   ```
1. Inicie el servicio **Agente de aprovisionamiento de Microsoft Azure AD Connect**.
1. Use el siguiente comando para terminar el archivo y depurar problemas. 
    ```
    Get-Content “C:/ProgramData/Microsoft/Azure AD Connect Provisioning Agent/Trace/ProvAgentTrace.log” -Wait
    ```
## <a name="object-synchronization-problems"></a>Problemas de sincronización de objetos

La siguiente sección contiene información sobre la solución de problemas de sincronización de objetos.

### <a name="provisioning-logs"></a>Registros de aprovisionamiento

En Azure Portal, se pueden usar registros de aprovisionamiento para ayudar a realizar un seguimiento de los problemas de sincronización de objetos y solucionarlos. Para ver los registros, seleccione **Registros**.

![Botón Registros](media/how-to-troubleshoot/log1.png)

Los registros de aprovisionamiento proporcionan una gran cantidad de información sobre el estado de los objetos que se sincronizan entre el entorno local de Active Directory y Azure.

![Pantalla de registros de aprovisionamiento](media/how-to-troubleshoot/log2.png)

Puede utilizar los cuadros desplegables de la parte superior de la página para filtrar la vista para centrarse en problemas específicos, como fechas. Haga doble clic en un evento individual para ver información adicional.

![Información del cuadro desplegable de registros de aprovisionamiento](media/how-to-troubleshoot/log3.png)

Esta información proporciona pasos detallados e indica dónde se está produciendo el problema de sincronización. De esta manera, puede identificar el punto exacto del problema.


## <a name="provisioning-quarantined-problems"></a>Problemas de aprovisionamiento en cuarentena

El aprovisionamiento en la nube supervisa el estado de la configuración y establece los objetos incorrectos en un estado de cuarentena. Si todas o la mayoría de las llamadas realizadas al sistema de destino no tienen éxito sistemáticamente debido a un error (como en el caso de credenciales de administrador no válidas), el trabajo de aprovisionamiento se establece en un estado de cuarentena.

![Estado de cuarentena](media/how-to-troubleshoot/quarantine1.png)

Al seleccionar el estado, puede ver información adicional acerca de la cuarentena. También puede obtener el código de error y el mensaje.

![Información del estado de cuarentena](media/how-to-troubleshoot/quarantine2.png)

### <a name="resolve-a-quarantine"></a>Resolución de una cuarentena

- Use Azure Portal para reiniciar el trabajo de aprovisionamiento. En la página de configuración del agente, seleccione **Reiniciar aprovisionamiento**.

  ![Reinicio del aprovisionamiento](media/how-to-troubleshoot/quarantine3.png)

- Use Microsoft Graph para [reiniciar el trabajo de aprovisionamiento](/graph/api/synchronization-synchronizationjob-restart?tabs=http&view=graph-rest-beta). Así tendrá control total sobre lo que reinicie. Puede elegir borrar:
  - Custodias, para reiniciar el contador de custodias que se acumula hacia el estado de cuarentena.
  - Cuarentena, para quitar la aplicación de la cuarentena.
  - Marcas de agua. 
  
  Use la siguiente solicitud:
 
  `POST /servicePrincipals/{id}/synchronization/jobs/{jobId}/restart`

## <a name="next-steps"></a>Pasos siguientes 

- [¿Qué es el aprovisionamiento?](what-is-provisioning.md)
- [¿Qué es el aprovisionamiento en la nube de Azure AD Connect?](what-is-cloud-provisioning.md)