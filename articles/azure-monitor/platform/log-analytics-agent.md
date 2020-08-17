---
title: Introducción al agente de Log Analytics
description: En este tema se le ayudará a comprender cómo recopilar datos y supervisar equipos hospedados en Azure, localmente o en entornos en la nube con Log Analytics.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 02/04/2020
ms.openlocfilehash: 36b94f53d3a9113c3980c94c3b8eff0713f11814
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87446536"
---
# <a name="log-analytics-agent-overview"></a>Introducción al agente de Log Analytics
El agente de Log Analytics se desarrolló para administrar de forma integral máquinas virtuales de cualquier nube, máquinas del entorno local y máquinas supervisadas por [System Center Operations Manager](/system-center/scom/). Los agentes de Windows y Linux envían los datos recopilados de diferentes orígenes al área de trabajo de Log Analytics de Azure Monitor, así como cualquier registro o métrica únicos que se hayan definido en una solución de supervisión. El agente de Log Analytics también permite utilizar información detallada y otros servicios de Azure Monitor, como [Azure Monitor para VM](../insights/vminsights-enable-overview.md), [Azure Security Center](../../security-center/index.yml) y [Azure Automation](../../automation/automation-intro.md).

En este artículo se proporciona una descripción detallada del agente, de los requisitos del sistema y de la red, y de los diferentes métodos de implementación.

> [!NOTE]
> Debe saber que el agente de Log Analytics también se denomina a veces Microsoft Monitoring Agent (MMA) o agente de OMS para Linux.

> [!NOTE]
> Azure Diagnostics Extension es uno de los agentes disponibles para recopilar datos de supervisión del sistema operativo invitado y de los recursos de proceso de Azure. Consulte [Introducción a los agentes de Azure Monitor](agents-overview.md) para ver una descripción de los distintos agentes e instrucciones sobre cómo seleccionar los que necesite.

## <a name="comparison-to-azure-diagnostics-extension"></a>Comparación con Azure Diagnostics Extension
El agente [Azure Diagnostics Extension](diagnostics-extension-overview.md) de Azure Monitor también puede utilizarse para recopilar datos de supervisión del sistema operativo invitado de las máquinas virtuales de Azure. Puede optar por usar uno o ambos, según sus necesidades. Consulte [Introducción a los agentes de Azure Monitor](agents-overview.md) para ver una comparación detallada entre los agentes de Azure Monitor. 

Las principales diferencias que debe tener en cuenta son:

- Azure Diagnostics Extension solo se puede usar con máquinas virtuales de Azure. El agente de Log Analytics se puede usar con máquinas virtuales de Azure, de otras nubes y del entorno local.
- Azure Diagnostics Extension envía datos a Azure Storage, a las [métricas de Azure Monitor](data-platform-metrics.md) (solo en Windows) y a Event Hubs. El agente de Log Analytics recopila los datos en los [registros en Azure Monitor](data-platform-logs.md).
- El agente de Log Analytics es necesario para las [soluciones](../monitor-reference.md#insights-and-core-solutions), [Azure Monitor para VM](../insights/vminsights-overview.md) y otros servicios, como [Azure Security Center](../../security-center/index.yml).

## <a name="costs"></a>Costos
El agente de Log Analytics no cuesta nada, pero puede generar gastos por los datos ingeridos. Consulte [Administrar el uso y los costos con los registros de Azure Monitor](manage-cost-storage.md) para más información sobre los precios de los datos recopilados en el área de trabajo de Log Analytics.

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
El agente de Log Analytics envía datos al área de trabajo de Log Analytics de Azure Monitor. El agente de Windows puede hospedarse en diferentes sitios para enviar datos a varias áreas de trabajo y grupos de administración de System Center Operations Manager. El agente de Linux solo puede enviar datos a un único destino.

## <a name="other-services"></a>Otros servicios
El agente para Windows y Linux no es solo para conectarse a Azure Monitor, sino que también admite Azure Automation para hospedar el rol de trabajo Hybrid Runbook Worker y otros servicios como [Change Tracking](../../automation/change-tracking.md), [Update Management](../../automation/update-management/update-mgmt-overview.md) y [Azure Security Center](../../security-center/security-center-intro.md). Para obtener más información acerca de la función Hybrid Runbook Worker, consulte [Hybrid Runbook Worker de Azure Automation](../../automation/automation-hybrid-runbook-worker.md).  

## <a name="installation-and-configuration"></a>Instalación y configuración

Si se usan los agentes de Log Analytics para recopilar datos, es preciso conocer lo siguiente para planear la implementación de los agentes:

* Para recopilar datos de los agentes de Windows, puede [configurar cada agente para que informe a una o varias áreas de trabajo](agent-windows.md), aun cuando informen a un grupo de administración de System Center Operations Manager. El agente de Windows puede notificar hasta cuatro áreas de trabajo.
* El agente de Linux no admite el hospedaje múltiple y solo puede informar a un área de trabajo.
* El agente de Windows admite el [estándar FIPS 140](/windows/security/threat-protection/fips-140-validation), mientras que el agente de Linux no lo admite.  

Si usa System Center Operations Manager 2012 R2, o cualquier versión posterior:

* Cada grupo de administración de Operations Manager se puede [conectar a una sola área de trabajo](om-agents.md).
* Los equipos Linux que informan a un grupo de administración deben estar configurados para informar directamente a un área de trabajo Log Analytics. Si los equipos Linux ya están notificando directamente a un área de trabajo y desea supervisarlos con Operations Manager, siga estos para [notificar a un grupo de administración de Operations Manager](agent-manage.md#configure-agent-to-report-to-an-operations-manager-management-group).
* El agente de Windows de Log Analytics se puede instalar en el equipo Windows y se le puede indicar que informe tanto a Operations Manager integrado con un área de trabajo como a otra área de trabajo.


Existen diferentes métodos para instalar el agente de Log Analytics y conectar la máquina a Azure Monitor en función de los requisitos. En la tabla siguiente se resalta cada método para determinar cuál funciona mejor en su organización.

|Source | Método | Descripción|
|-------|-------------|-------------|
|Azure VM| [Manualmente desde Azure Portal](../learn/quick-collect-azurevm.md?toc=%2fazure%2fazure-monitor%2ftoc.json) | Especifique las máquinas virtuales que desea implementar desde el área de trabajo de Log Analytics. |
| | Extensión de máquina virtual de Log Analytics para [Windows](../../virtual-machines/extensions/oms-windows.md) o [Linux](../../virtual-machines/extensions/oms-linux.md) con la CLI de Azure o con una plantilla de Azure Resource Manager | La extensión instala el agente de Log Analytics en máquinas virtuales de Azure y las inscribe en un área de trabajo de Azure Monitor existente. |
| | [Azure Monitor para VM](../insights/vminsights-enable-overview.md) | Cuando se habilita la supervisión con Azure Monitor para VM, esta instala la extensión y el agente de Log Analytics. |
| | [Aprovisionamiento automático de Azure Security Center](../../security-center/security-center-enable-data-collection.md) | Azure Security Center puede aprovisionar el agente de Log Analytics en todas las máquinas virtuales de Azure admitidas y en las nuevas máquinas virtuales que se creen si se habilita para supervisar las amenazas y vulnerabilidades de la seguridad. Si está habilitado, se aprovisionarán todas las máquinas virtuales nuevas o existentes sin un agente instalado. |
| Equipo híbrido con Windows| [Instalación manual](agent-windows.md) | Instale Microsoft Monitoring Agent desde la línea de comandos. |
| | [DSC de Azure Automation](agent-windows.md#install-the-agent-using-dsc-in-azure-automation) | Automatice la instalación con Azure Automation DSC. |
| | [Plantilla de Resource Manager con Azure Stack](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/MicrosoftMonitoringAgent-ext-win) | Utilice una plantilla de Resource Manager si tiene Microsoft Azure Stack implementado en el centro de datos.| 
| Equipo híbrido con Linux| [Instalación manual](../learn/quick-collect-linux-computer.md)|Instale al agente para Linux llamando a un script contenedor hospedado en GitHub. | 
| System Center Operations Manager|[Integrar Operations Manager con Log Analytics](./om-agents.md) | Configure la integración entre Operations Manager y los registros de Azure Monitor para reenviar datos recopilados de equipos Windows que informan a un grupo de administración.|  


## <a name="supported-windows-operating-systems"></a>Sistemas operativos Windows compatibles

Las siguientes versiones del sistema operativo Windows son compatibles oficialmente con el agente de Windows:

* Windows Server 2019
* Windows Server 2016, versión 1709 y 1803
* Windows Server 2012, 2012 R2
* Windows Server 2008 SP2 (x64), 2008 R2
* Windows 10 Enterprise (incluida la sesión múltiple) y Pro
* Windows 8 Enterprise y Pro 
* Windows 7 SP1

>[!NOTE]
>Aunque el agente de Log Analytics para Windows se diseñó para admitir escenarios de supervisión de servidor, se puede ejecutar el cliente de Windows para admitir las cargas de trabajo configuradas y optimizadas para el sistema operativo del servidor. El agente es compatible con el cliente de Windows, pero nuestras soluciones de supervisión no se centran en escenarios de supervisión de cliente, salvo se indique explícitamente.

## <a name="supported-linux-operating-systems"></a>Sistemas operativos Linux compatibles

En esta sección se ofrecen detalles sobre las distribuciones de Linux compatibles.

A partir de las versiones publicadas después de agosto de 2018, hemos realizado los siguientes cambios en nuestro modelo de soporte técnico:  

* Solo se admiten las versiones de servidor, no de cliente.  
* La compatibilidad se centra en cualquiera de las [distribuciones de Linux aprobadas en Azure](../../virtual-machines/linux/endorsed-distros.md). Tenga en cuenta que puede haber cierto retraso entre la aprobación de una distribución o versión de Linux en Azure y su compatibilidad con el agente de Linux de Log Analytics.
* Se admiten todas las versiones secundarias de cada versión principal de la lista.
* No se admiten las versiones que han superado la fecha de finalización de soporte técnico de su fabricante.  
* No se admiten las nuevas versiones de AMI.  
* Solo se admiten las versiones que ejecutan SSL 1.x de forma predeterminada.

>[!NOTE]
>Si utiliza una distribución o versión que no se admite actualmente y no se alinea con nuestro modelo de soporte técnico, se recomienda bifurcar este repositorio, confirmar que el soporte técnico de Microsoft no prestará asistencia con versiones de agente con bifurcación.


### <a name="python-2-requirement"></a>Requisito de Python 2
 El agente de Log Analytics requiere Python 2. Si la máquina virtual usa un distribución que no incluye Python 2 de forma predeterminada, debe instalarlo. Los siguientes comandos de ejemplo instalarán Python 2 en diferentes distribuciones.

 - Red Hat, CentOS, Oracle: `yum install -y python2`
 - Ubuntu, Debian: `apt-get install -y python2`
 - SUSE: `zypper install -y python2`

Se debe crear el alias "python" para el ejecutable python2 mediante el comando siguiente:

```
alternatives --set python `which python2`
```

### <a name="supported-distros"></a>Distribuciones admitidas

Las siguientes versiones del sistema operativo Linux son compatibles oficialmente con el agente de Linux:

* Amazon Linux 2017.09 (x64)
* CentOS Linux 6 (x64) y 7 (x64)  
* Oracle Linux 6 y 7 (x64) 
* Red Hat Enterprise Linux Server 6 (x64), 7 (x64) y 8 (x64)
* Debian GNU/Linux 8 y 9 (x64)
* Ubuntu 14.04 LTS (x86/x64), 16.04 LTS (x64) y 18.04 LTS (x64)
* SUSE Linux Enterprise Server 12 (x64) y 15 (x64)

>[!NOTE]
>OpenSSL 1.1.0 solo se admite en las plataformas x86_x64 (64 bits) y OpenSSL anterior a 1.x no se admite en ninguna plataforma.
>

### <a name="agent-prerequisites"></a>Requisitos del agente

En la tabla siguiente se indican los paquetes necesarios para las distribuciones de Linux compatibles en las que se instalará el agente.

|Paquete necesario |Descripción |Versión mínima |
|-----------------|------------|----------------|
|Glibc |    Biblioteca GNU C | 2.5-12 
|Openssl    | Bibliotecas OpenSSL | 1.0.x o 1.1.x |
|Curl | Cliente web de cURL | 7.15.5 |
|Python | | 2.6+ o 3.3+
|Python ctypes | | 
|PAM | Módulos de autenticación conectables | | 

>[!NOTE]
>Rsyslog o Syslog son necesarios para recopilar mensajes de Syslog. El demonio predeterminado de Syslog en la versión 5 de Red Hat Enterprise Linux, CentOS y Oracle Linux (Sysklog) no se admite para la recopilación de eventos de Syslog. Para recopilar datos de Syslog de esta versión de estas distribuciones, es necesario instalar configurar el demonio de Rsyslog para reemplazar Sysklog.

## <a name="tls-12-protocol"></a>Protocolo TLS 1.2

Para garantizar la seguridad de los datos en tránsito hacia los registros de Azure Monitor, se recomienda encarecidamente configurar el agente para que use al menos Seguridad de la capa de transporte (TLS) 1.2. Las versiones anteriores de TLS/Capa de sockets seguros (SSL) han demostrado ser vulnerables y, si bien todavía funcionan para permitir la compatibilidad con versiones anteriores, **no se recomiendan**.  Para información adicional, revise [Sending data securely using TLS 1.2](data-security.md#sending-data-securely-using-tls-12) (Envío de datos de forma segura mediante TLS 1.2). 


## <a name="sha-2-code-signing-support-requirement-for-windows"></a>Requisito de compatibilidad con la firma de código SHA-2 para Windows
El agente de Windows empezará a usar exclusivamente la firma SHA-2 el 17 de agosto de 2020. Este cambio afectará a los clientes que usan el agente de Log Analytics en un sistema operativo heredado como parte de cualquier servicio de Azure (Azure Monitor, Azure Automation, Azure Update Management, Azure Change Tracking, Azure Security Center, Azure Sentinel, ATP de Windows Defender). El cambio no requiere ninguna acción del cliente a menos que esté ejecutando el agente en una versión de sistema operativo heredada (Windows 7, Windows Server 2008 R2 y Windows Server 2008). Los clientes que ejecutan una versión de sistema operativo heredada deben realizar las siguientes acciones en sus máquinas antes del 17 de agosto de 2020 o los agentes dejarán de enviar datos a las áreas de trabajo de Log Analytics:

1. Instale el Service Pack más reciente para su sistema operativo. Las versiones de Service Pack necesarias son:
    - Windows 7 SP1
    - Windows Server 2008 SP2
    - Windows Server 2008 R2 SP1

2. Instale las actualizaciones de firma SHA-2 de Windows para su sistema operativo, como se describe en [Requisito de compatibilidad con la firma de código SHA-2 para Windows y WSUS de 2019](https://support.microsoft.com/help/4472027/2019-sha-2-code-signing-support-requirement-for-windows-and-wsus).
3. Actualice el agente de Windows a la versión más reciente (versión 10.20.18029).
4. Se recomienda para configurar el agente para [usar TLS 1.2](agent-windows.md#configure-agent-to-use-tls-12). 


## <a name="network-requirements"></a>Requisitos de red
El agente para Linux y Windows se comunica con el servicio Azure Monitor en el puerto TCP 443. Si la máquina se conecta mediante un servidor proxy o firewall para comunicarse a través de Internet, consulte los requisitos siguientes para comprender qué configuración de red debe aplicarse. Si las directivas de seguridad de TI no permiten que los equipos de la red se conecten a Internet, puede configurar una [puerta de enlace de Log Analytics](gateway.md) y, luego, configurar el agente para conectarse a registros de Azure Monitor a través de la puerta de enlace. El agente, a continuación, puede recibir información de configuración y enviar los datos recopilados según qué reglas de recopilación de datos y soluciones de supervisión haya habilitado en el área de trabajo.

![Diagrama de comunicación del agente de Log Analytics](./media/log-analytics-agent/log-analytics-agent-01.png)

En la tabla siguiente se muestra la información de configuración de proxy y firewall requerida para que el agente de Windows y Linux se comunique con los registros de Azure Monitor.

### <a name="firewall-requirements"></a>Requisitos de firewall

|Recurso del agente|Puertos |Dirección |Omitir inspección de HTTPS|
|------|---------|--------|--------|   
|*.ods.opinsights.azure.com |Puerto 443 |Salida|Sí |  
|*.oms.opinsights.azure.com |Puerto 443 |Salida|Sí |  
|*.blob.core.windows.net |Puerto 443 |Salida|Sí |
|*.azure-automation.net |Puerto 443 |Salida|Sí |

Para obtener información sobre el firewall necesaria para Azure Government, vea [Administración de Azure Government](../../azure-government/compare-azure-government-global-azure.md#azure-monitor-logs). 

Si tiene previsto usar Hybrid Runbook Worker de Azure Automation para conectarse al servicio Automation y registrarse en él para usar runbooks o soluciones de administración en el entorno, debe tener acceso al número de puerto y las direcciones URL descritos en [Configuración de la red para Hybrid Runbook Worker](../../automation/automation-hybrid-runbook-worker.md#network-planning). 

### <a name="proxy-configuration"></a>Configuración de proxy

El agente de Linux y Windows admite la comunicación a través de un servidor proxy o la puerta de enlace de Log Analytics con Azure Monitor mediante el protocolo HTTPS.  Se admite la autenticación anónima y básica (nombre de usuario/contraseña).  Si el agente de Windows está conectado directamente al servicio, la configuración de proxy se especifica durante la instalación o [después de la implementación](agent-manage.md#update-proxy-settings) desde el Panel de control o con PowerShell.  

Con el agente de Linux, el servidor proxy se especifica durante o [después de la instalación](agent-manage.md#update-proxy-settings) mediante la modificación del archivo de configuración proxy.conf.  El valor de configuración del proxy del agente de Linux tiene la siguiente sintaxis:

`[protocol://][user:password@]proxyhost[:port]`

> [!NOTE]
> Aunque el servidor proxy no requiera que se autentique, el agente de Linux sí que requerirá que proporcione un pseudousuario y contraseña. Puede ser cualquier nombre de usuario o contraseña.

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

