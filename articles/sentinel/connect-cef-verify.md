---
title: Validación de la conectividad a Azure Sentinel | Microsoft Docs
description: Valide la conectividad de la solución de seguridad para asegurarse de que los mensajes de CEF se reenvían a Azure Sentinel.
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
ms.openlocfilehash: f6892f4ebb250290a0faad546fd000530baf4479
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87038178"
---
# <a name="step-3-validate-connectivity"></a>PASO 3: Validar conectividad

Una vez que haya implementado el reenviador de registros (en el paso 1) y configurado la solución de seguridad para enviar mensajes de CEF (en el paso 2), siga estas instrucciones para comprobar la conectividad entre la solución de seguridad y Azure Sentinel. 

## <a name="prerequisites"></a>Prerrequisitos

- Debe tener permisos elevados (sudo) en la máquina del reenviador de registros.

- Python debe estar instalado en la máquina del reenviador de registros.<br>
Use el comando `python –version` para comprobarlo.

## <a name="how-to-validate-connectivity"></a>Procedimientos para validar la conectividad

1. En el menú de navegación de Azure Sentinel, abra **Registros**. Ejecute una consulta con el esquema **CommonSecurityLog** para ver si recibe registros de la solución de seguridad.<br>
Tenga en cuenta que pueden transcurrir unos 20 minutos hasta que los registros empiecen a aparecer en **Log Analytics**. 

1. Si no ve ningún resultado de la consulta, compruebe que se generan eventos en la solución de seguridad o intente generar algunos, y compruebe que se reenvían a la máquina del reenviador de Syslog que ha designado. 

1. Ejecute el siguiente script en el reenviador de registros para comprobar la conectividad entre la solución de seguridad, el reenviador de registros y Azure Sentinel. Este script comprueba que el demonio escucha en los puertos correctos, que el reenvío del demonio está configurado correctamente y que nada bloquee la comunicación entre el demonio y el agente de Log Analytics. También envía mensajes ficticios "TestCommonEventFormat" para comprobar la conectividad de un extremo a otro. <br>
 `sudo wget https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/DataConnectors/CEF/cef_troubleshoot.py&&sudo python cef_troubleshoot.py [WorkspaceID]`

## <a name="validation-script-explained"></a>Explicación del script de validación

El script de validación realiza las siguientes comprobaciones:

# <a name="rsyslog-daemon"></a>[demonio rsyslog](#tab/rsyslog)

1. Comprueba que el archivo<br>
    `/etc/opt/microsoft/omsagent/[WorkspaceID]/conf/omsagent.d/security_events.conf`<br>
    existe y es válido.

1. Comprueba que el archivo incluye el texto siguiente:

    ```console
    <source>
        type syslog
        port 25226
        bind 127.0.0.1
        protocol_type tcp
        tag oms.security
        format /(?<time>(?:\w+ +){2,3}(?:\d+:){2}\d+|\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}.[\w\-\:\+]{3,12}):?\s*(?:(?<host>[^: ]+) ?:?)?\s*(?<ident>.*CEF.+?(?=0\|)|%ASA[0-9\-]{8,10})\s*:?(?<message>0\|.*|.*)/
        <parse>
            message_format auto
        </parse>
    </source>

    <filter oms.security.**>
        type filter_syslog_security
    </filter>
    ```

1. Comprueba si hay mejoras de seguridad en el equipo que podrían estar bloqueando el tráfico de red (por ejemplo, un firewall de host).

1. Comprueba que el demonio de syslog (rsyslog) esté configurado correctamente para enviar mensajes que identifica como CEF (mediante regex) al agente de Log Analytics en el puerto TCP 25226:

    - Archivo de configuración: `/etc/rsyslog.d/security-config-omsagent.conf`

        ```console
        :rawmsg, regex, "CEF"|"ASA"
        *.* @@127.0.0.1:25226
        ```
  
1. Comprueba que el demonio de syslog esté recibiendo datos en el puerto 514.

1. Comprueba que se han establecido las conexiones necesarias: TCP 514 para recibir datos, TCP 25226 para la comunicación interna entre el demonio de syslog y el agente de Log Analytics.

1. Envía datos ficticios al puerto 514 en localhost. Estos datos deben poderse observar en el área de trabajo de Azure Sentinel ejecutando la siguiente consulta:

    ```console
    CommonSecurityLog
    | where DeviceProduct == "MOCK"
    ```

# <a name="syslog-ng-daemon"></a>[demonio syslog-ng](#tab/syslogng)

1. Comprueba que el archivo<br>
    `/etc/opt/microsoft/omsagent/[WorkspaceID]/conf/omsagent.d/security_events.conf`<br>
    existe y es válido.

1. Comprueba que el archivo incluye el texto siguiente:

    ```console
    <source>
        type syslog
        port 25226
        bind 127.0.0.1
        protocol_type tcp
        tag oms.security
        format /(?<time>(?:\w+ +){2,3}(?:\d+:){2}\d+|\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}.[\w\-\:\+]{3,12}):?\s*(?:(?<host>[^: ]+) ?:?)?\s*(?<ident>.*CEF.+?(?=0\|)|%ASA[0-9\-]{8,10})\s*:?(?<message>0\|.*|.*)/
        <parse>
            message_format auto
        </parse>
    </source>

    <filter oms.security.**>
        type filter_syslog_security
    </filter>
    ```

1. Comprueba si hay mejoras de seguridad en el equipo que podrían estar bloqueando el tráfico de red (por ejemplo, un firewall de host).

1. Comprueba que el demonio de syslog (syslog-ng) esté configurado correctamente para enviar mensajes que identifica como CEF (mediante regex) al agente de Log Analytics en el puerto TCP 25226:

    - Archivo de configuración: `/etc/syslog-ng/conf.d/security-config-omsagent.conf`

        ```console
        filter f_oms_filter {match(\"CEF\|ASA\" ) ;};
        destination oms_destination {tcp(\"127.0.0.1\" port("25226"));};
        log {source(s_src);filter(f_oms_filter);destination(oms_destination);};
        ```

1. Comprueba que el demonio de syslog esté recibiendo datos en el puerto 514.

1. Comprueba que se han establecido las conexiones necesarias: TCP 514 para recibir datos, TCP 25226 para la comunicación interna entre el demonio de syslog y el agente de Log Analytics.

1. Envía datos ficticios al puerto 514 en localhost. Estos datos deben poderse observar en el área de trabajo de Azure Sentinel ejecutando la siguiente consulta:

    ```console
    CommonSecurityLog
    | where DeviceProduct == "MOCK"
    ```
---

## <a name="next-steps"></a>Pasos siguientes
En este documento, ha aprendido a conectar dispositivos CEF a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats.md).
- [Use libros](tutorial-monitor-your-data.md) para supervisar los datos.

