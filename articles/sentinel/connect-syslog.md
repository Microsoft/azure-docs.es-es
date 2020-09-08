---
title: Conexión de datos de Syslog a Azure Sentinel | Microsoft Docs
description: Conecte cualquier máquina o dispositivo que admita Syslog en Azure Sentinel mediante un agente en una máquina Linux entre el dispositivo y Sentinel. 
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
ms.date: 07/17/2020
ms.author: yelevin
ms.openlocfilehash: 7670d00a2dd25961a51d18c50c102e0f92b30975
ms.sourcegitcommit: 37afde27ac137ab2e675b2b0492559287822fded
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88566155"
---
# <a name="collect-data-from-linux-based-sources-using-syslog"></a>Recopilación de datos de orígenes basados en Linux mediante Syslog

Puede transmitir eventos de máquinas o dispositivos compatibles con Syslog basados en Linux a Azure Sentinel mediante el agente de Log Analytics para Linux (anteriormente conocido como agente de OMS). Puede hacerlo en cualquier máquina que permita instalar el agente de Log Analytics directamente en la máquina. El demonio de Syslog nativo de la máquina recopilará los eventos locales de los tipos especificados y los reenviará localmente al agente, que los transmitirá al área de trabajo Log Analytics.

> [!NOTE]
> - Si el dispositivo admite **Common Event Format (CEF) en Syslog**, se recopila un conjunto de datos más completo y los datos se analizan en la colección. Debe elegir esta opción y seguir las instrucciones en [Conexión de su solución externa con CEF](connect-common-event-format.md).
>
> - Log Analytics admite la recopilación de mensajes enviados por los demonios **rsyslog** o **syslog-ng**, donde rsyslog es el predeterminado. El demonio predeterminado de Syslog en la versión 5 de Red Hat Enterprise Linux (RHEL), CentOS y Oracle Linux (**sysklog**) no se admite para la recopilación de eventos de Syslog. Para recopilar datos de Syslog de esta versión de estas distribuciones, es necesario instalar configurar el demonio de Rsyslog para reemplazar Sysklog.

## <a name="how-it-works"></a>Funcionamiento

**Syslog** es un protocolo de registro de eventos que es común a Linux. Al instalar el **agente de Log Analytics para Linux** en su VM o dispositivo, la rutina de instalación configura el demonio Syslog local para que reenvíe mensajes al agente en el puerto TCP 25224. Después, el agente envía el mensaje al área de trabajo de Log Analytics a través de HTTPS, donde se analiza en una entrada del registro de eventos de la tabla Syslog en **Azure Sentinel > Registros**.

Para más información, consulte [Orígenes de datos de Syslog en Azure Monitor](../azure-monitor/platform/data-sources-syslog.md).

## <a name="configure-syslog-collection"></a>Configuración de recopilaciones de Syslog

### <a name="configure-your-linux-machine-or-appliance"></a>Configuración de la máquina o el dispositivo Linux

1. En Azure Sentinel, seleccione **Data connectors** (Conectores de datos) y, después, seleccione el conector de **Syslog**.

1. En la hoja **Syslog**, seleccione **Open connector page** (Abrir página del conector).

1. Instale el agente de Linux. En **Elija dónde quiere instalar al agente:**
    
    **Para una VM Linux de Azure:**
      
    1. Seleccione **Instalación del agente en una máquina virtual Linux de Azure**.
    
    1. Haga clic en el vínculo **Download & install agent for Azure Linux Virtual machines >** (Descargar e instalar el agente para máquinas virtuales Linux de Azure >). 
    
    1. En la hoja **Máquinas virtuales**, haga clic en la máquina virtual en la que quiera instalar el agente y, a continuación, haga clic en **Conectar**. Repita este paso para cada VM que quiera conectar.
    
    **Para cualquier otra máquina Linux:**

    1. Seleccione **Instalación del agente en una máquina Linux que no sea de Azure**.

    1. Haga clic en el vínculo **Download & install agent for non-Azure Linux machines >** (Descargar e instalar el agente para máquinas Linux que no sean de Azure >). 

    1. En la hoja **Agents Management** (Administración de agentes), haga clic en la pestaña **Servidores Linux** y, a continuación, copie el comando para **descargar e incorporar el agente para Linux** y ejecútelo en la máquina Linux. 
    
   > [!NOTE]
   > Asegúrese de que configura los valores de seguridad para estos equipos de acuerdo con la directiva de seguridad de su organización. Por ejemplo, puede configurar los valores de red para que se alineen con la directiva de seguridad de red de la organización y cambiar los puertos y protocolos del demonio para que se adapten a sus requisitos de seguridad.

### <a name="configure-the-log-analytics-agent"></a>Configuración del agente de Log Analytics

1. En la parte inferior de la hoja del conector de Syslog, haga clic en el vínculo **Open your workspace advanced settings configuration >** (Abrir la configuración avanzada del área de trabajo >).

1. En la hoja **Configuración avanzada**, seleccione **Datos** > **Syslog**. A continuación, agregue los recursos del conector que se van a recopilar.
    
    - Agregue los recursos que su dispositivo de Syslog incluye en sus encabezados de registro. 
    
    - Si desea usar la detección de inicio de sesión de SSH anómalo con los datos que recopila, agregue **auth** y **authpriv**. Consulte la [siguiente sección](#configure-the-syslog-connector-for-anomalous-ssh-login-detection) para conocer más detalles.

1. Cuando haya agregado todos los recursos que desea supervisar y ajustado las opciones de gravedad de cada uno, active la casilla **Aplicar la configuración siguiente a mis máquinas**.

1. Seleccione **Guardar**. 

1. En la VM o dispositivo, asegúrese de que está enviando los recursos que ha especificado.

1. Para consultar los datos de registro de Syslog en **Registros**, escriba `Syslog` en la ventana de consulta.

1. Puede usar los parámetros de consulta que se describen en [Uso de funciones en consultas de registros de Azure Monitor](../azure-monitor/log-query/functions.md) para analizar los mensajes de Syslog. Después, puede guardar la consulta como una nueva función de Log Analytics y usarla como un nuevo tipo de datos.

> [!NOTE]
> **Uso del mismo equipo para reenviar Syslog sin formato *y* mensajes de CEF**
>
>
> Puede usar la [máquina del reenviador de registros de CEF](connect-cef-agent.md) existente para recopilar y reenviar registros también desde orígenes de Syslog sin formato. Sin embargo, debe realizar los pasos siguientes para evitar el envío de eventos en ambos formatos a Azure Sentinel, ya que esto provocará la duplicación de eventos.
>
>    Cuando ya haya configurado la [recopilación de datos de los orígenes de CEF](connect-common-event-format.md) y haya configurado el agente de Log Analytics como se ha indicado anteriormente:
>
> 1. En cada máquina que envíe registros en formato CEF, debe editar el archivo de configuración de Syslog para quitar las funciones que se usan para enviar mensajes CEF. De este modo, las instalaciones que se envían en CEF no se enviarán también en Syslog. Consulte [Configuración de Syslog en agente de Linux](../azure-monitor/platform/data-sources-syslog.md#configure-syslog-on-linux-agent) para obtener instrucciones detalladas sobre cómo hacerlo.
>
> 1. Debe ejecutar el siguiente comando en esas máquinas para deshabilitar la sincronización del agente con la configuración de Syslog en Azure Sentinel. Esto garantiza que el cambio de configuración que ha realizado en el paso anterior no se sobrescriba.<br>
> `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/OMS_MetaConfigHelper.py --disable'`


### <a name="configure-the-syslog-connector-for-anomalous-ssh-login-detection"></a>Configuración del conector de Syslog para la detección de inicios de sesión de SSH anómalos

> [!IMPORTANT]
> La detección de inicios de sesión de SSH anómalos se encuentra actualmente en versión preliminar pública.
> Esta característica se ofrece sin contrato de nivel de servicio y no se recomienda para cargas de trabajo de producción.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Azure Sentinel puede aplicar aprendizaje automático (ML) a los datos de Syslog para identificar una actividad de inicio de sesión de Secure Shell anómala (SSH). Los escenarios incluyen:

- Viaje imposible: cuando se producen dos eventos de inicio de sesión correctos desde dos ubicaciones que son imposibles de alcanzar en el período de tiempo de los dos eventos de inicio de sesión.
- Ubicación inesperada: la ubicación desde donde se produjo un evento de inicio de sesión correcto es sospechosa. Por ejemplo, la ubicación no se ha visualizado recientemente.
 
Esta detección requiere una configuración específica del conector de datos de Syslog: 

1. En el paso 5 del procedimiento anterior, asegúrese de que **auth** y **authpriv** estén seleccionados como recursos para supervisar. Mantenga la configuración predeterminada de las opciones de gravedad, de modo que todas estén seleccionadas. Por ejemplo:
    
    > [!div class="mx-imgBorder"]
    > ![Recursos necesarios para la detección de inicios de sesión SSH anómalos](./media/connect-syslog/facilities-ssh-detection.png)

2. Deje tiempo suficiente para que se recopile la información de Syslog. A continuación, vaya a **Azure Sentinel: Registros** y copie y pegue la siguiente consulta:
    
    ```console
    Syslog |  where Facility in ("authpriv","auth")| extend c = extract( "Accepted\\s(publickey|password|keyboard-interactive/pam)\\sfor ([^\\s]+)",1,SyslogMessage)| where isnotempty(c) | count 
    ```
    
    Cambie el **intervalo de tiempo** si es necesario y seleccione **Ejecutar**.
    
    Si el recuento resultante es cero, confirme la configuración del conector y que los equipos supervisados tienen una actividad de inicio de sesión correcta durante el período de tiempo que especificó para la consulta.
    
    Si el recuento resultante es mayor que cero, los datos de Syslog son adecuados para la detección de inicios de sesión de SSH anómalos. Puede habilitar esta detección en **Análisis** >  **Rule templates (Plantillas de reglas)**  >  **(Versión preliminar) Anomalous SSH Login Detection** (Detección de inicios de sesión de SSH anómalos).

## <a name="next-steps"></a>Pasos siguientes
En este documento, ha aprendido a conectar dispositivos locales de Syslog a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Use libros](tutorial-monitor-your-data.md) para supervisar los datos.

