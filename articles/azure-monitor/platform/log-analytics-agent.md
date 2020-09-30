---
title: Introducción al agente de Log Analytics
description: En este tema se le ayudará a comprender cómo recopilar datos y supervisar equipos hospedados en Azure, localmente o en entornos en la nube con Log Analytics.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/21/2020
ms.openlocfilehash: fb58728e005ad70ac5392aa9e3e6a254ed317276
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89016346"
---
# <a name="log-analytics-agent-overview"></a>Introducción al agente de Log Analytics
El agente de Azure Log Analytics recopila la telemetría de las máquinas virtuales Windows y Linux en cualquier nube, en máquinas locales y en aquellas supervisadas por [System Center Operations Manager](/system-center/scom/) y envía los datos recopilados al área de trabajo de Log Analytics en Azure Monitor. El agente de Log Analytics también permite utilizar información detallada y otros servicios de Azure Monitor, como [Azure Monitor para VM](../insights/vminsights-enable-overview.md), [Azure Security Center](../../security-center/index.yml) y [Azure Automation](../../automation/automation-intro.md). En este artículo se proporciona una descripción detallada del agente, de los requisitos del sistema y de la red, y de los métodos de implementación.

> [!NOTE]
> Debe saber que el agente de Log Analytics también se denomina a veces Microsoft Monitoring Agent (MMA) o agente de OMS para Linux.

## <a name="comparison-to-azure-diagnostics-extension"></a>Comparación con Azure Diagnostics Extension
El agente [Azure Diagnostics Extension](diagnostics-extension-overview.md) de Azure Monitor también puede utilizarse para recopilar datos de supervisión del sistema operativo invitado de las máquinas virtuales de Azure. Puede optar por usar uno o ambos, según sus necesidades. Consulte [Introducción a los agentes de Azure Monitor](agents-overview.md) para ver una comparación detallada entre los agentes de Azure Monitor. 

Las principales diferencias que debe tener en cuenta son:

- Azure Diagnostics Extension solo se puede usar con máquinas virtuales de Azure. El agente de Log Analytics se puede usar con máquinas virtuales de Azure, de otras nubes y del entorno local.
- Azure Diagnostics Extension envía datos a Azure Storage, a las [métricas de Azure Monitor](data-platform-metrics.md) (solo en Windows) y a Event Hubs. El agente de Log Analytics envía los datos a los [registros de Azure Monitor](data-platform-logs.md).
- El agente de Log Analytics es necesario para las [soluciones](../monitor-reference.md#insights-and-core-solutions), [Azure Monitor para VM](../insights/vminsights-overview.md) y otros servicios, como [Azure Security Center](../../security-center/index.yml).

## <a name="costs"></a>Costos
El agente de Log Analytics no cuesta nada, pero puede generar gastos por los datos ingeridos. Consulte [Administrar el uso y los costos con los registros de Azure Monitor](manage-cost-storage.md) para más información sobre los precios de los datos recopilados en el área de trabajo de Log Analytics.

## <a name="supported-operating-systems"></a>Sistemas operativos admitidos

 Consulte los [sistemas operativos admitidos](agents-overview.md#supported-operating-systems) para obtener una lista de las versiones del sistema operativo Windows y Linux compatibles con el agente de Log Analytics. 


## <a name="data-collected"></a>Datos recopilados
En la tabla siguiente, se muestran los tipos de datos que puede configurar en un área de trabajo de Log Analytics para recopilar los datos de todos los agentes conectados. Consulte [¿Qué supervisa Azure Monitor?](../monitor-reference.md) para ver una lista con información detallada, soluciones y otros servicios que usan el agente de Log Analytics para recopilar otros tipos de datos.

| Origen de datos | Descripción |
| --- | --- |
| [Registros de eventos de Windows](data-sources-windows-events.md) | Información enviada al sistema de registro de eventos de Windows. |
| [Syslog](data-sources-syslog.md)                     | Información enviada al sistema de registro de eventos de Windows. |
| [Rendimiento](data-sources-performance-counters.md)  | Valores numéricos que miden el rendimiento de diferentes aspectos del sistema operativo y las cargas de trabajo. |
| [Registros de IIS](data-sources-iis-logs.md)                 | Información sobre el uso de los sitios web de IIS que se ejecutan en el sistema operativo invitado. |
| [Registros personalizados](data-sources-custom-logs.md)           | Eventos de archivos de texto en equipos Windows y Linux. |

## <a name="data-destinations"></a>Destinos de datos
El agente de Log Analytics envía datos al área de trabajo de Log Analytics de Azure Monitor. El agente de Windows puede hospedarse en diferentes sitios para enviar datos a varias áreas de trabajo y grupos de administración de System Center Operations Manager. El agente de Linux puede enviar datos a un solo destino, ya sea un área de trabajo o un grupo de administración.

## <a name="other-services"></a>Otros servicios
El agente para Linux y Windows no sirve solo para conectarse a Azure Monitor. Otros servicios, como Azure Security Center y Azure Sentinel, también usan este agente y su conexión con el área de trabajo de Log Analytics. El agente también admite Azure Automation para hospedar el rol de trabajo Hybrid Runbook Worker y otros servicios como [Change Tracking](../../automation/change-tracking.md), [Update Management](../../automation/update-management/update-mgmt-overview.md) y [Azure Security Center](../../security-center/security-center-intro.md). Para obtener más información acerca de la función Hybrid Runbook Worker, consulte [Hybrid Runbook Worker de Azure Automation](../../automation/automation-hybrid-runbook-worker.md).  

## <a name="workspace-and-management-group-limitations"></a>Limitaciones del área de trabajo y el grupo de administración

Consulte [Configurar el agente para que envíe notificaciones a un grupo de administración de Operations Manager](agent-manage.md#configure-agent-to-report-to-an-operations-manager-management-group) para obtener más información sobre cómo conectar un agente a un grupo de administración de Operations Manager.

* Los agentes de Windows pueden conectarse con hasta cuatro áreas de trabajo, incluso si estas están conectadas a un grupo de administración de System Center Operations Manager.
* El agente de Linux no admite el hospedaje múltiple y solo puede conectarse con un área de trabajo o grupo de administración.
  

## <a name="security-limitations"></a>Limitaciones de seguridad

* El agente de Windows admite el [estándar FIPS 140](/windows/security/threat-protection/fips-140-validation), mientras que el agente de Linux no lo admite.  


## <a name="installation-options"></a>Opción de instalación

Existen diferentes métodos para instalar el agente de Log Analytics y conectar la máquina a Azure Monitor en función de los requisitos. En las secciones siguientes se enumeran los posibles métodos para distintos tipos de máquinas virtuales.

### <a name="azure-virtual-machine"></a>Máquina virtual de Azure

- [Azure Monitor para VM](../insights/vminsights-enable-overview.md) proporciona varios métodos para habilitar agentes a escala. Esto incluye la instalación del agente de Log Analytics y Dependency Agent. 
- [Azure Security Center puede aprovisionar el agente de Log Analytics](../../security-center/security-center-enable-data-collection.md) en todas las máquinas virtuales de Azure admitidas y en las nuevas máquinas virtuales que se creen si se habilita para supervisar las amenazas y vulnerabilidades de la seguridad.
- La extensión de Log Analytics para máquinas virtuales [Windows](../../virtual-machines/extensions/oms-windows.md) o [Linux](../../virtual-machines/extensions/oms-linux.md) se puede instalar mediante Azure Portal, la CLI de Azure, Azure PowerShell o una plantilla de Azure Resource Manager.
- Puede instalarla para máquinas virtuales individuales de Azure [manualmente desde Azure Portal](../learn/quick-collect-azurevm.md?toc=%2fazure%2fazure-monitor%2ftoc.json).


### <a name="windows-virtual-machine-on-premises-or-in-another-cloud"></a>Máquina virtual de Windows en el entorno local o en otra nube 

- [Instale manualmente](agent-windows.md) el agente desde la línea de comandos.
- Automatice la instalación con [DSC de Automatización de Azure](agent-windows.md#install-agent-using-dsc-in-azure-automation).
- Use una [plantilla de Resource Manager con Azure Stack](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/MicrosoftMonitoringAgent-ext-win)

### <a name="linux-virtual-machine-on-premises-or-in-another-cloud"></a>Máquina virtual Linux en el entorno local o en otra nube

- [Instale manualmente](../learn/quick-collect-linux-computer.md) el agente mediante una llamada a un script contenedor hospedado en GitHub.
- System Center Operations Manager|[Integre Operations Manager con Azure Monitor](./om-agents.md) para reenviar datos recopilados de equipos Windows que informan a un grupo de administración.

## <a name="workspace-id-and-key"></a>Clave e id. del área de trabajo
Independientemente del método de instalación que haya usado, necesitará la clave y el id. de área de trabajo para área de trabajo de Log Analytics a la que se conectará el agente. Seleccione el área de trabajo desde el menú **Áreas de trabajo de Log Analytics** en Azure Portal. A continuación, seleccione **Administración de agentes** en la sección **Configuración**. 

[![Detalles del área de trabajo](media/log-analytics-agent/workspace-details.png)](media/log-analytics-agent/workspace-details.png#lightbox)

## <a name="tls-12-protocol"></a>Protocolo TLS 1.2

Para garantizar la seguridad de los datos en tránsito hacia los registros de Azure Monitor, se recomienda encarecidamente configurar el agente para que use al menos Seguridad de la capa de transporte (TLS) 1.2. Las versiones anteriores de TLS/Capa de sockets seguros (SSL) han demostrado ser vulnerables y, si bien todavía funcionan para permitir la compatibilidad con versiones anteriores, **no se recomiendan**.  Para información adicional, revise [Sending data securely using TLS 1.2](data-security.md#sending-data-securely-using-tls-12) (Envío de datos de forma segura mediante TLS 1.2). 

## <a name="network-requirements"></a>Requisitos de red
El agente para Linux y Windows se comunica con el servicio Azure Monitor a través del puerto TCP 443. Si la máquina se conecta mediante un servidor proxy o firewall para comunicarse a través de Internet, consulte los requisitos siguientes para comprender qué configuración de red es necesaria. Si las directivas de seguridad de TI no permiten que los equipos de la red se conecten a Internet, puede configurar una [puerta de enlace de Log Analytics](gateway.md) y, luego, configurar el agente para conectarse a Azure Monitor a través de la puerta de enlace. Después, el agente puede recibir información de configuración y enviar los datos recopilados.

![Diagrama de comunicación del agente de Log Analytics](./media/log-analytics-agent/log-analytics-agent-01.png)

En la tabla siguiente se muestra la información de configuración de proxy y firewall necesaria para que los agentes de Windows y Linux se comuniquen con los registros de Azure Monitor.

### <a name="firewall-requirements"></a>Requisitos de firewall

|Recurso del agente|Puertos |Dirección |Omitir inspección de HTTPS|
|------|---------|--------|--------|   
|*.ods.opinsights.azure.com |Puerto 443 |Salida|Sí |  
|*.oms.opinsights.azure.com |Puerto 443 |Salida|Sí |  
|*.blob.core.windows.net |Puerto 443 |Salida|Sí |
|*.azure-automation.net |Puerto 443 |Salida|Sí |

Para obtener información sobre el firewall necesaria para Azure Government, vea [Administración de Azure Government](../../azure-government/compare-azure-government-global-azure.md#azure-monitor). 

Si tiene previsto usar Hybrid Runbook Worker de Azure Automation para conectarse al servicio Automation y registrarse en él para usar runbooks o soluciones de administración en el entorno, debe tener acceso al número de puerto y las direcciones URL descritos en [Configuración de la red para Hybrid Runbook Worker](../../automation/automation-hybrid-runbook-worker.md#network-planning). 

### <a name="proxy-configuration"></a>Configuración de proxy

El agente de Linux y Windows admite la comunicación a través de un servidor proxy o la puerta de enlace de Log Analytics con Azure Monitor mediante el protocolo HTTPS.  Se admite la autenticación anónima y básica (nombre de usuario/contraseña).  Si el agente de Windows está conectado directamente al servicio, la configuración de proxy se especifica durante la instalación o [después de la implementación](agent-manage.md#update-proxy-settings) desde el Panel de control o con PowerShell.  

Con el agente de Linux, el servidor proxy se especifica durante o [después de la instalación](agent-manage.md#update-proxy-settings) mediante la modificación del archivo de configuración proxy.conf.  El valor de configuración del proxy del agente de Linux tiene la siguiente sintaxis:

`[protocol://][user:password@]proxyhost[:port]`

|Propiedad| Descripción |
|--------|-------------|
|Protocolo | https |
|usuario | Nombre de usuario opcional para la autenticación de proxy |
|password | Contraseña opcional para la autenticación de proxy |
|proxyhost | Dirección o nombre de dominio completo (FQDN) del servidor proxy o la puerta de enlace de Log Analytics |
|port | Número de puerto opcional para el servidor proxy o la puerta de enlace de Log Analytics |

Por ejemplo: `https://user01:password@proxy01.contoso.com:30443`

> [!NOTE]
> Si usa caracteres especiales como "\@" en la contraseña, recibirá un error de conexión de proxy porque el valor no se analiza correctamente.  Para solucionar este problema, codifique la contraseña en la dirección URL con una herramienta como [URLDecode](https://www.urldecoder.org/).  



## <a name="next-steps"></a>Pasos siguientes

* Revise los [orígenes de datos](agent-data-sources.md) para saber qué orígenes de datos hay disponibles para recopilar datos de su sistema Windows o Linux. 
* Obtenga información acerca de las [consultas de registros](../log-query/log-query-overview.md) para analizar los datos recopilados de soluciones y orígenes de datos. 
* Conozca las [soluciones de supervisión](../insights/solutions.md) que agregan funcionalidad a Azure Monitor y que también recopilan datos en el área de trabajo de Log Analytics.

