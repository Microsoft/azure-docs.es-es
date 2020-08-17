---
title: Procedimientos para solucionar problemas relacionados con el agente de Log Analytics para Windows
description: Se describen los síntomas, las causas y las soluciones de los problemas más comunes que surgen con el agente de Log Analytics para Windows en Azure Monitor.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 11/21/2019
ms.openlocfilehash: 3d99293ea83c883f8d0870d78dfbec58f74c9bd1
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87927324"
---
# <a name="how-to-troubleshoot-issues-with-the-log-analytics-agent-for-windows"></a>Procedimientos para solucionar problemas relacionados con el agente de Log Analytics para Windows 

En este artículo se proporciona información sobre los errores que es posible que experimente con el agente de Log Analytics para Windows en Azure Monitor y se sugieren posibles soluciones para resolverlos.

Si ninguno de estos pasos funciona, también están disponibles los siguientes canales de soporte técnico:

* Los clientes con ventajas de soporte técnico Premier pueden abrir una solicitud de soporte técnico en [Premier](https://premier.microsoft.com/).
* Los clientes con contratos de soporte técnico de Azure pueden abrir una solicitud de soporte técnico [en Azure Portal](https://manage.windowsazure.com/?getsupport=true).
* Visite la página de comentarios de Log Analytics para revisar los errores e ideas enviadas [https://aka.ms/opinsightsfeedback](https://aka.ms/opinsightsfeedback) o registre uno nuevo. 

## <a name="important-troubleshooting-sources"></a>Orígenes de solución de problemas importantes

 Para ayudar a solucionar los problemas relacionados con el agente de Log Analytics para Windows, el agente registra los eventos en el registro de eventos de Windows, concretamente en *Aplicaciones y servicios\Operations Manager*.  

## <a name="connectivity-issues"></a>Problemas de conectividad

Si el agente se comunica a través de un servidor proxy o un firewall, es posible que haya restricciones vigentes que impidan la comunicación desde el equipo de origen y el servicio Azure Monitor. En caso de que se bloquee la comunicación, debido a un error de configuración, se puede producir un error en el registro con un área de trabajo al intentar instalar el agente o configurarlo después de la instalación para que notifique a un área de trabajo adicional. Se puede producir un error de comunicación del agente después de un registro correcto. En esta sección se describen los métodos para solucionar este tipo de problema con el agente de Windows.

Compruebe que el firewall o proxy está configurado para permitir los puertos y las direcciones URL descritas en la tabla siguiente. Confirme también que la inspección de HTTP no está habilitada para el tráfico web, ya que puede evitar un canal TLS seguro entre el agente y Azure Monitor.  

|Recurso del agente|Puertos |Dirección |Omitir inspección de HTTPS|
|------|---------|--------|--------|   
|*.ods.opinsights.azure.com |Puerto 443 |Salida|Sí |  
|*.oms.opinsights.azure.com |Puerto 443 |Salida|Sí |  
|*.blob.core.windows.net |Puerto 443 |Salida|Sí |  
|*.agentsvc.azure-automation.net |Puerto 443 |Salida|Sí |  

Para obtener información sobre el firewall necesaria para Azure Government, vea [Administración de Azure Government](../../azure-government/compare-azure-government-global-azure.md#azure-monitor). Si tiene previsto usar Hybrid Runbook Worker de Azure Automation para conectarse al servicio Automation y registrarse en él para usar runbooks o soluciones de administración en el entorno, debe tener acceso al número de puerto y las direcciones URL descritos en [Configuración de la red para Hybrid Runbook Worker](../../automation/automation-hybrid-runbook-worker.md#network-planning). 

Hay varias formas de comprobar si el agente se comunica de forma correcta con Azure Monitor.

- Habilite la [Evaluación de estado del agente de Azure Log Analytics](../insights/solution-agenthealth.md) en el área de trabajo. En el panel de Agent Health, examine la columna **Count of unresponsive agents** (Número de agentes que no responden) para ver rápidamente si aparece el agente.  

- Ejecute la consulta siguiente para confirmar que el agente envía un latido al área de trabajo que se ha configurado para que le envíe notificaciones. Reemplace `<ComputerName>` con el nombre real del equipo.

    ```
    Heartbeat 
    | where Computer like "<ComputerName>"
    | summarize arg_max(TimeGenerated, * ) by Computer 
    ```

    Si el equipo se comunica correctamente con el servicio, la consulta debe devolver un resultado. Si la consulta no ha devuelto un resultado, compruebe primero que el agente está configurado para notificar al área de trabajo correcta. Si está configurado correctamente, vaya al paso 3 y busque en el registro de eventos de Windows para identificar si el agente registra el problema que podría estar impidiendo la comunicación con Azure Monitor.

- Otro método para identificar un problema de conectividad consiste en ejecutar la herramienta **TestCloudConnectivity**. La herramienta se instala de forma predeterminada con el agente en la carpeta *%SystemRoot%\Archivos de programa\Microsoft Monitoring Agent\Agent*. Desde un símbolo del sistema con privilegios elevados, vaya a la carpeta y ejecute la herramienta. La herramienta devuelve los resultados y resalta dónde se ha producido el error en la prueba (por ejemplo, si estaba relacionado con un puerto o una dirección URL determinada que se ha bloqueado). 

    ![Resultados de la ejecución de la herramienta TestCloudConnection](./media/agent-windows-troubleshoot/output-testcloudconnection-tool-01.png)

- Filtre el registro de eventos de *Operations Manager* por **Orígenes de eventos** - *Módulos de servicio de mantenimiento*, *HealthService* y *Conector de servicio*, y fíltrelo por **Nivel de evento** *Advertencia* y *Error* para confirmar si ha escrito eventos de la tabla siguiente. Si se han escrito, revise los pasos de resolución incluidos para cada evento posible.

    |Id. de evento |Source |Descripción |Resolución |
    |---------|-------|------------|-----------|
    |2133 y 2129 |Servicio de mantenimiento |Error de conexión con el servicio desde el agente |Este error se puede producir cuando el agente no se puede comunicar directamente o a través de un servidor proxy o firewall con el servicio Azure Monitor. Compruebe la configuración de proxy de agente o que el firewall o proxy de red permite el tráfico TCP desde el equipo al servicio.|
    |2138 |Módulos de servicio de mantenimiento |Se requiere autenticación del proxy |Configure las opciones del proxy de agente y especifique el nombre de usuario y la contraseña necesarios para autenticarse con el servidor proxy. |
    |2129 |Módulos de servicio de mantenimiento |Error de conexión o error de negociación TLS |Compruebe la configuración TCP/IP del adaptador de red y del proxy de agente.|
    |2127 |Módulos de servicio de mantenimiento |Error al enviar datos de código de error recibido |Si solo se produce de forma periódica durante el día, podría ser simplemente una anomalía aleatoria que se puede ignorar. Supervise para comprender la frecuencia con que sucede. Si se produce con frecuencia a lo largo del día, compruebe la configuración de red y de proxy. Si en la descripción se incluye el código de error HTTP 404 y es la primera vez que el agente intenta enviar datos al servicio, incluirá un error 500 con un código de error 404 interno. 404 significa que no se ha encontrado, lo que indica que el área de almacenamiento para la nueva área de trabajo todavía se está aprovisionando. En el siguiente reintento, los datos se escribirán correctamente en el área de trabajo según lo previsto. Un error HTTP 403 podría indicar un problema de credenciales o permisos. El error 403 incluye más información para ayudar a solucionar el problema.|
    |4000 |Conector de servicio |Error de resolución de nombre DNS |El equipo no pudo resolver la dirección de Internet que se ha usado al enviar datos al servicio. Esto podría deberse a la configuración de resolución DNS en el equipo, la configuración incorrecta del proxy o un problema de DNS temporal con el proveedor. Si se produce de forma periódica, se podría deber a un problema transitorio relacionado con la red.|
    |4001 |Conector de servicio |Error de conexión al servicio. |Este error se puede producir cuando el agente no se puede comunicar directamente o a través de un servidor proxy o firewall con el servicio Azure Monitor. Compruebe la configuración de proxy de agente o que el firewall o proxy de red permite el tráfico TCP desde el equipo al servicio.|
    |4002 |Conector de servicio |El servicio ha devuelto el código de estado HTTP 403 en respuesta a una consulta. Póngase en contacto con el administrador de servicios para el estado del servicio. La consulta se reintentará más tarde. |Este error se escribe durante la fase de registro inicial del agente y verá una dirección URL similar a la siguiente: *https://\<workspaceID>.oms.opinsights.azure.com/AgentService.svc/AgentTopologyRequest*. Un código de error 403 significa prohibido y se puede deber a que la clave o el identificador del área de trabajo se hayan escrito incorrectamente, o bien a que la fecha y hora en el equipo sean incorrectas. Si la hora es +/-15 minutos de la hora actual, la incorporación produce un error. Para corregir este problema, actualice la fecha o la zona horaria del equipo Windows.|

## <a name="data-collection-issues"></a>Problemas recopilación de datos

Después de instalar el agente y de que notifique a sus áreas de trabajo configuradas, es posible que deje de recibir configuración o de recopilar o reenviar rendimiento, registros u otros datos para el servicio en función de lo que se haya habilitado y destinado al equipo. Es necesario determinar si:

- En el área de trabajo no está disponible un tipo de datos determinado o se trata de todos los datos.
- El tipo de datos lo especifica una solución o se especifica como parte de la configuración de recopilación de datos del área de trabajo.
- Cuántos equipos se ven afectados. Cuántos equipos notifican al área de trabajo: uno o varios.
- Funcionaba y se ha detenido a una hora concreta del día o nunca se ha recopilado. 
- La consulta de búsqueda de registros que se usa es sintácticamente correcta. 
- El agente ha recibido alguna vez su configuración de Azure Monitor.

El primer paso para solucionar problemas consiste en determinar si el equipo envía un evento de latido.

```
Heartbeat 
    | where Computer like "<ComputerName>"
    | summarize arg_max(TimeGenerated, * ) by Computer
```

Si la consulta devuelve resultados, tendrá que determinar si no se ha recopilado un tipo de datos determinado y se ha reenviado al servicio. La causa podría ser que el agente no reciba la configuración actualizada desde el servicio, o bien algún otro síntoma que impida el funcionamiento normal del agente. Para solucionar problemas adicionales, realice los pasos siguientes.

1. Abra un símbolo del sistema con privilegios elevados en el equipo y escriba `net stop healthservice && net start healthservice` para reiniciar el servicio del agente.
2. Abra el registro de eventos de *Operations Manager* y busque los **identificadores de evento** *7023, 7024, 7025, 7028* y *1210* en **Origen del evento** *HealthService*.  Estos eventos indican que el agente recibe correctamente la configuración de Azure Monitor y que supervisan el equipo de forma activa. La descripción del evento con el identificador 1210 también especificará en la última línea todas las soluciones y la información que se incluyen en el ámbito de supervisión del agente.  

    ![Descripción del identificador de evento 1210](./media/agent-windows-troubleshoot/event-id-1210-healthservice-01.png)

3. Si después de unos minutos no ve los datos esperados en la visualización o los resultados de la consulta, en función de si los ve desde una solución o desde Insight, desde el registro de eventos de *Operations Manager*, busque **Orígenes de eventos** *HealthService* y *Módulos de servicio de mantenimiento*, y filtre por **Nivel de evento** *Advertencia* y *Error* para confirmar si ha escrito eventos de la tabla siguiente.

    |Id. de evento |Source |Descripción |Resolución |
    |---------|-------|------------|
    |8000 |HealthService |Este evento especificará si un flujo de trabajo relacionado con el rendimiento, los eventos u otro tipo de datos recopilado no puede reenviar al servicio para la ingesta en el área de trabajo. | El id. de evento 2136 de la instancia de HealthService de origen se escribe junto con este evento y puede indicar que el agente no se puede comunicar con el servicio, posiblemente debido a una configuración incorrecta de las opciones de proxy y autenticación, a la interrupción de la red, o bien a que el firewall de red o el proxy no permite el tráfico TCP desde el equipo al servicio.| 
    |10102 y 10103 |Módulos de servicio de mantenimiento |El flujo de trabajo no ha podido resolver el origen de datos. |Esto puede ocurrir si el contador de rendimiento especificado o la instancia no existen en el equipo o se han definido incorrectamente en la configuración de datos del área de trabajo. Si se trata de un [contador de rendimiento](data-sources-performance-counters.md#configuring-performance-counters) especificado por el usuario, compruebe que la información especificada sigue el formato correcto y que existe en los equipos de destino. |
    |26002 |Módulos de servicio de mantenimiento |El flujo de trabajo no ha podido resolver el origen de datos. |Esto puede ocurrir si el registro de eventos de Windows especificado no existe en el equipo. Este error se puede omitir sin problemas si el equipo no espera que se registre este registro de eventos; en caso contrario, si se trata de un [registro de eventos](data-sources-windows-events.md#configuring-windows-event-logs) especificado por el usuario, compruebe que la información especificada sea correcta. |
