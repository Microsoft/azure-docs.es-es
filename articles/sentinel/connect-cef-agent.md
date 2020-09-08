---
title: Implementación del reenviador de registros para conectar CEF a Azure Sentinel | Microsoft Docs
description: Aprenda a implementar el agente para conectar datos CEF a Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/19/2020
ms.author: yelevin
ms.openlocfilehash: a7d7c7b7236841835866ccb7786e7e4eab767c1f
ms.sourcegitcommit: 37afde27ac137ab2e675b2b0492559287822fded
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88565594"
---
# <a name="step-1-deploy-the-log-forwarder"></a>Paso 1: Implementación del reenviador de registros


En este paso, designará y configurará la máquina Linux que reenviará los registros desde la solución de seguridad al área de trabajo de Azure Sentinel. Esta máquina puede ser una máquina física o virtual del entorno local, una máquina virtual de Azure o una máquina virtual en otra nube. Con el vínculo proporcionado, ejecutará un script en la máquina designada que realiza las tareas siguientes:
- Instala el agente de Log Analytics para Linux (también denominado agente de OMS) y lo configura para los siguientes fines:
    - escucha de mensajes CEF desde el demonio de Syslog de Linux integrado en el puerto TCP 25226
    - envío de mensajes de forma segura a través de TLS a su área de trabajo de Azure Sentinel, donde se analizan y enriquecen

- Configura el demonio de Syslog de Linux integrado (rsyslog.d/syslog-ng) para los siguientes fines:
    - escucha de mensajes de Syslog desde las soluciones de seguridad en el puerto TCP 514
    - reenvío de solo los mensajes que identifica como CEF al agente de Log Analytics en localhost mediante el puerto TCP 25226
 
## <a name="prerequisites"></a>Prerrequisitos

- Debe tener permisos elevados (sudo) en la máquina Linux designada.
- Debe tener Python instalado en la máquina Linux.<br>Use el comando `python -version` para comprobarlo.
- La máquina Linux no debe estar conectada a ninguna área de trabajo de Azure antes de instalar el agente de Log Analytics.

## <a name="run-the-deployment-script"></a>Ejecutar el script de implementación
 
1. En el menú de navegación de Azure Sentinel, haga clic en **Conectores de datos**. En la lista de conectores, haga clic en el icono de **Common Event Format (CEF)** y, a continuación, en el botón **Open connector page** (Abrir página del conector) en la parte inferior derecha. 

1. En **1.2 Install the CEF collector on the Linux machine** (1.2 Instalación del recopilador de CEF en la máquina Linux), copie el vínculo proporcionado en **Run the following script to install and apply the CEF collector** (Ejecutar el siguiente script para instalar y aplicar el recopilador de CEF) o en el texto siguiente:

     `sudo wget https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/DataConnectors/CEF/cef_installer.py&&sudo python cef_installer.py [WorkspaceID] [Workspace Primary Key]`

1. Mientras se ejecuta el script, asegúrese de que no recibe ningún mensaje de error o de advertencia.

> [!NOTE]
> **Uso de la misma máquina para reenviar Syslog sin formato *y* mensajes de CEF**
>
> Si tiene previsto usar esta máquina de reenviador de registros para reenviar [mensajes de Syslog](connect-syslog.md), así como de CEF, haga lo siguiente para evitar la duplicación de eventos en las tablas Syslog y CommonSecurityLog:
>
> 1. En cada máquina de origen que envíe registros al reenviador en formato CEF, debe editar el archivo de configuración de Syslog para quitar las funciones que se usan para enviar mensajes de CEF. De este modo, las instalaciones que se envían en CEF no se enviarán también en Syslog. Consulte [Configuración de Syslog en agente de Linux](../azure-monitor/platform/data-sources-syslog.md#configure-syslog-on-linux-agent) para obtener instrucciones detalladas sobre cómo hacerlo.
>
> 1. Debe ejecutar el siguiente comando en esas máquinas para deshabilitar la sincronización del agente con la configuración de Syslog en Azure Sentinel. Esto garantiza que el cambio de configuración que ha realizado en el paso anterior no se sobrescriba.<br>
> `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/OMS_MetaConfigHelper.py --disable'`

Diríjase al [PASO 2: Configurar la solución de seguridad para reenviar mensajes CEF](connect-cef-solution-config.md).

## <a name="deployment-script-explained"></a>Explicación del script de implementación

A continuación, se presenta una descripción, comando por comando, de las acciones del script de implementación.

Elija un demonio de syslog para ver la descripción adecuada.

# <a name="rsyslog-daemon"></a>[demonio rsyslog](#tab/rsyslog)

1. **Descarga e instalación del agente de Log Analytics:**

    - Descarga el script de instalación para el agente de Linux de Log Analytics (OMS)<br>
        `wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh`

    - Instala el agente de Log Analytics<br>
        `sh onboard_agent.sh -w [workspaceID] -s [Primary Key] -d opinsights.azure.com`

1. **Configuración del demonio de Syslog:**

    1. Abre el puerto 514 para la comunicación TCP mediante el archivo de configuración de syslog `/etc/rsyslog.conf`.

    1. Configura el demonio para reenviar los mensajes de CEF al agente de Log Analytics en el puerto TCP 25226, insertando un archivo de configuración especial `security-config-omsagent.conf` en el directorio del demonio de syslog `/etc/rsyslog.d/`.

        Contenido del archivo `security-config-omsagent.conf`:

        ```console
        :rawmsg, regex, "CEF"|"ASA"
        *.* @@127.0.0.1:25226
        ```

1. **Reinicio del demonio de Syslog**

    `service rsyslog restart`

1. **Establecimiento de la configuración del agente de Log Analytics para escuchar en el puerto 25226 y reenviar mensajes de CEF a Azure Sentinel**

    1. Descarga la configuración del repositorio GitHub del agente de Log Analytics<br>
        `wget -o /etc/opt/microsoft/omsagent/[workspaceID]/conf/omsagent.d/security_events.conf https://raw.githubusercontent.com/microsoft/OMS-Agent-for-Linux/master/installer/conf/omsagent.d/security_events.conf`


    1. Reinicia el agente de Log Analytics<br>
        `/opt/microsoft/omsagent/bin/service_control restart [workspaceID]`

# <a name="syslog-ng-daemon"></a>[demonio syslog-ng](#tab/syslogng)

1. **Descarga e instalación del agente de Log Analytics:**

    - Descarga el script de instalación para el agente de Linux de Log Analytics (OMS)<br>`wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh`

    - Instala el agente de Log Analytics<br>`sh onboard_agent.sh -w [workspaceID] -s [Primary Key] -d opinsights.azure.com`

1. **Configuración del demonio de Syslog:**

    1. Abre el puerto 514 para la comunicación TCP mediante el archivo de configuración de syslog `/etc/syslog-ng/syslog-ng.conf`.

    1. Configura el demonio para reenviar los mensajes de CEF al agente de Log Analytics en el puerto TCP 25226, insertando un archivo de configuración especial `security-config-omsagent.conf` en el directorio del demonio de syslog `/etc/syslog-ng/conf.d/`.

        Contenido del archivo `security-config-omsagent.conf`:

        ```console
        filter f_oms_filter {match(\"CEF\|ASA\" ) ;};
        destination oms_destination {tcp(\"127.0.0.1\" port("25226"));};
        log {source(s_src);filter(f_oms_filter);destination(oms_destination);};
        ```

1. **Reinicio del demonio de Syslog**

    `service syslog-ng restart`

1. **Establecimiento de la configuración del agente de Log Analytics para escuchar en el puerto 25226 y reenviar mensajes de CEF a Azure Sentinel**

    1. Descarga la configuración del repositorio GitHub del agente de Log Analytics<br>
        `wget -o /etc/opt/microsoft/omsagent/[workspaceID]/conf/omsagent.d/security_events.conf https://raw.githubusercontent.com/microsoft/OMS-Agent-for-Linux/master/installer/conf/omsagent.d/security_events.conf`


    1. Reinicia el agente de Log Analytics<br>
        `/opt/microsoft/omsagent/bin/service_control restart [workspaceID]`

---

## <a name="next-steps"></a>Pasos siguientes
En este documento, ha aprendido a implementar el agente de Log Analytics para conectar dispositivos CEF a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats.md).

