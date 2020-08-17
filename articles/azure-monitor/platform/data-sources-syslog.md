---
title: Recopilación y análisis de mensajes de Syslog en Azure Monitor | Microsoft Docs
description: Syslog es un protocolo de registro de eventos que es común a Linux. En este artículo se describe cómo configurar la recopilación de mensajes de Syslog en Log Analytics y detalles de los registros que se crean.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/22/2019
ms.openlocfilehash: d9efdb11ffd30c68a0ac8ea8e8156fe707f188de
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322319"
---
# <a name="syslog-data-sources-in-azure-monitor"></a>Orígenes de datos de Syslog en Azure Monitor
Syslog es un protocolo de registro de eventos que es común a Linux. Las aplicaciones envían mensajes que pueden almacenarse en la máquina local o entregarse a un recopilador de Syslog. Al instalar el agente de Log Analytics para Linux, este configura el demonio Syslog local para que reenvíe mensajes al agente. En ese momento, el agente envía el mensaje a Azure Monitor, donde se crea un registro correspondiente.  

> [!NOTE]
> Azure Monitor admite la recopilación de mensajes enviados por rsyslog o syslog-ng, donde rsyslog es el demonio predeterminado. El demonio predeterminado de Syslog en la versión 5 de Red Hat Enterprise Linux, CentOS y Oracle Linux (Sysklog) no se admite para la recopilación de eventos de Syslog. Para recopilar datos de Syslog de esta versión de estas distribuciones, es necesario instalar el [demonio rsyslog](http://rsyslog.com) y configurarlo para reemplazar Sysklog.
>
>

![Recopilación de Syslog](media/data-sources-syslog/overview.png)

Los recursos siguientes son compatibles con el recopilador de Syslog:

* kern
* usuario
* mail
* daemon
* auth
* syslog
* lpr
* news
* uucp
* cron
* authpriv
* ftp
* local0-local7

Para cualquier otro recurso, [configure un origen de datos de registros personalizados](data-sources-custom-logs.md) en Azure Monitor.
 
## <a name="configuring-syslog"></a>Configuración de Syslog
El agente de Log Analytics para Linux solo recopilará los eventos con los recursos y los niveles de gravedad que se especifican en su configuración. Puede configurar Syslog a través de Azure Portal o mediante la administración de archivos de configuración de sus agentes de Linux.

### <a name="configure-syslog-in-the-azure-portal"></a>Configuración de Syslog en Azure Portal
Configure Syslog desde el [menú de datos en la configuración avanzada](agent-data-sources.md#configuring-data-sources). Esta configuración se entrega al archivo de configuración de cada agente de Linux.

Para agregar una nueva instalación, seleccione primero la opción **Aplicar la configuración siguiente a mis máquinas**, escriba su nombre y haga clic en **+** . Para cada recurso, solo se recopilarán los mensajes con los niveles de gravedad seleccionados.  Compruebe los niveles de gravedad del recurso determinado que desea recopilar. No puede proporcionar criterios adicionales para filtrar mensajes.

![Configuración de Syslog](media/data-sources-syslog/configure.png)

De forma predeterminada, todos los cambios realizados en la configuración se insertan automáticamente en todos los agentes. Si desea configurar Syslog manualmente en cada uno de los agentes de Linux, desactive la casilla *Aplicar la configuración siguiente a mis máquinas*.

### <a name="configure-syslog-on-linux-agent"></a>Configuración de Syslog en agente de Linux
Cuando el [agente de Log Analytics se instala en un cliente Linux](../learn/quick-collect-linux-computer.md), instala un archivo de configuración de Syslog predeterminado que define el recurso y la gravedad de los mensajes que se recopilan. Puede modificar este archivo para cambiar la configuración. El archivo de configuración es diferente según el demonio Syslog que ha instalado el cliente.

> [!NOTE]
> Si modifica la configuración de Syslog, tiene que reiniciar el demonio Syslog para que los cambios surtan efecto.
>
>

#### <a name="rsyslog"></a>rsyslog
El archivo de configuración de rsyslog se encuentra en **/etc/rsyslog.d/95-omsagent.conf**. A continuación se muestra su contenido predeterminado. Recopila mensajes de Syslog enviados desde el agente local para todos los recursos con, al menos, un nivel de advertencia.

```config
kern.warning       @127.0.0.1:25224
user.warning       @127.0.0.1:25224
daemon.warning     @127.0.0.1:25224
auth.warning       @127.0.0.1:25224
syslog.warning     @127.0.0.1:25224
uucp.warning       @127.0.0.1:25224
authpriv.warning   @127.0.0.1:25224
ftp.warning        @127.0.0.1:25224
cron.warning       @127.0.0.1:25224
local0.warning     @127.0.0.1:25224
local1.warning     @127.0.0.1:25224
local2.warning     @127.0.0.1:25224
local3.warning     @127.0.0.1:25224
local4.warning     @127.0.0.1:25224
local5.warning     @127.0.0.1:25224
local6.warning     @127.0.0.1:25224
local7.warning     @127.0.0.1:25224
```

Para quitar un recurso, elimine su sección del archivo de configuración. Puede limitar los niveles de gravedad que se recopilan para un recurso determinado mediante la modificación de la entrada de ese recurso. Por ejemplo, para limitar el recurso del usuario a mensajes con una gravedad de error o superior, debe modificar esa línea del archivo de configuración de la siguiente manera:

```config
user.error    @127.0.0.1:25224
```

#### <a name="syslog-ng"></a>syslog-ng
El archivo de configuración de syslog-ng se encuentra en **/etc/syslog-ng/syslog-ng.conf**.  A continuación se muestra su contenido predeterminado. Recopila mensajes de Syslog enviados desde el agente local para todos los recursos y todos los niveles de gravedad.   

```config
#
# Warnings (except iptables) in one file:
#
destination warn { file("/var/log/warn" fsync(yes)); };
log { source(src); filter(f_warn); destination(warn); };

#OMS_Destination
destination d_oms { udp("127.0.0.1" port(25224)); };

#OMS_facility = auth
filter f_auth_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(auth); };
log { source(src); filter(f_auth_oms); destination(d_oms); };

#OMS_facility = authpriv
filter f_authpriv_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(authpriv); };
log { source(src); filter(f_authpriv_oms); destination(d_oms); };

#OMS_facility = cron
filter f_cron_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(cron); };
log { source(src); filter(f_cron_oms); destination(d_oms); };

#OMS_facility = daemon
filter f_daemon_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(daemon); };
log { source(src); filter(f_daemon_oms); destination(d_oms); };

#OMS_facility = kern
filter f_kern_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(kern); };
log { source(src); filter(f_kern_oms); destination(d_oms); };

#OMS_facility = local0
filter f_local0_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(local0); };
log { source(src); filter(f_local0_oms); destination(d_oms); };

#OMS_facility = local1
filter f_local1_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(local1); };
log { source(src); filter(f_local1_oms); destination(d_oms); };

#OMS_facility = mail
filter f_mail_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(mail); };
log { source(src); filter(f_mail_oms); destination(d_oms); };

#OMS_facility = syslog
filter f_syslog_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(syslog); };
log { source(src); filter(f_syslog_oms); destination(d_oms); };

#OMS_facility = user
filter f_user_oms { level(alert,crit,debug,emerg,err,info,notice,warning) and facility(user); };
log { source(src); filter(f_user_oms); destination(d_oms); };
```

Para quitar un recurso, elimine su sección del archivo de configuración. Puede limitar los niveles de gravedad que se recopilan de un recurso determinado, para lo que debe eliminarlos de la lista.  Por ejemplo, para limitar el recurso del usuario solo a mensajes de alerta y críticos, debe modificar la sección del archivo de configuración de la siguiente manera:

```config
#OMS_facility = user
filter f_user_oms { level(alert,crit) and facility(user); };
log { source(src); filter(f_user_oms); destination(d_oms); };
```

### <a name="collecting-data-from-additional-syslog-ports"></a>Recopilación de datos de puertos de Syslog adicionales
El agente de Log Analytics escucha los mensajes de Syslog en el cliente local en el puerto 25224.  Cuando se instala el agente, se aplica una configuración de Syslog predeterminada, que se encuentra en la siguiente ubicación:

* Rsyslog: `/etc/rsyslog.d/95-omsagent.conf`
* Syslog-ng: `/etc/syslog-ng/syslog-ng.conf`

Puede cambiar el número de puerto si crea dos archivos de configuración: un archivo de configuración de FluentD y un archivo rsyslog-or-syslog-ng según el demonio Syslog que haya instalado.  

* El archivo de configuración de FluentD debe ser un nuevo archivo ubicado en: `/etc/opt/microsoft/omsagent/conf/omsagent.d`; reemplace el valor de la entrada **port** por el número de puerto personalizado.

    ```xml
    <source>
        type syslog
        port %SYSLOG_PORT%
        bind 127.0.0.1
        protocol_type udp
        tag oms.syslog
    </source>
    <filter oms.syslog.**>
        type filter_syslog
    ```

* Para rsyslog, debe crear un archivo de configuración ubicado en: `/etc/rsyslog.d/`; reemplace el valor %SYSLOG_PORT% por el número de puerto personalizado.  

    > [!NOTE]
    > Si modifica este valor en el archivo de configuración `95-omsagent.conf`, se sobrescribirá cuando el agente aplique una configuración predeterminada.
    >

    ```config
    # OMS Syslog collection for workspace %WORKSPACE_ID%
    kern.warning              @127.0.0.1:%SYSLOG_PORT%
    user.warning              @127.0.0.1:%SYSLOG_PORT%
    daemon.warning            @127.0.0.1:%SYSLOG_PORT%
    auth.warning              @127.0.0.1:%SYSLOG_PORT%
    ```

* Para modificar la configuración de syslog-ng, copie la configuración del ejemplo que se muestra a continuación y agregue la configuración modificada personalizada al final del archivo de configuración syslog-ng.conf ubicado en `/etc/syslog-ng/`. **No** use la etiqueta predeterminada **%WORKSPACE_ID%_oms** ni **%WORKSPACE_ID_OMS**; defina una etiqueta personalizada para ayudar a distinguir los cambios.  

    > [!NOTE]
    > Si modifica los valores predeterminados en el archivo de configuración, se sobrescribirán cuando el agente aplique una configuración predeterminada.
    >

    ```config
    filter f_custom_filter { level(warning) and facility(auth; };
    destination d_custom_dest { udp("127.0.0.1" port(%SYSLOG_PORT%)); };
    log { source(s_src); filter(f_custom_filter); destination(d_custom_dest); };
    ```

Después de completar los cambios, Syslog y el servicio de agente de Log Analytics deben reiniciarse para asegurarse de que los cambios de configuración surtan efecto.   

## <a name="syslog-record-properties"></a>Propiedades de registros de Syslog
Los registros de Syslog tienen un tipo **Syslog** y las propiedades que aparecen en la tabla siguiente.

| Propiedad | Descripción |
|:--- |:--- |
| Computer |Nombre del equipo desde el que se recopiló el evento. |
| Facility |Define la parte del sistema que ha generado el mensaje. |
| HostIP |Dirección IP del sistema que envía el mensaje. |
| HostName |Nombre del sistema que envía el mensaje. |
| SeverityLevel |Nivel de gravedad del evento. |
| SyslogMessage |Texto del mensaje. |
| ProcessID |Identificador del proceso que ha generado el mensaje. |
| EventTime |Fecha y hora en que se generó el evento. |

## <a name="log-queries-with-syslog-records"></a>Consultas de registro con registros de Syslog
La tabla siguiente proporciona ejemplos distintos de consultas de registro que recuperan registros de Syslog.

| Consultar | Descripción |
|:--- |:--- |
| syslog |Todos los registros de Syslog. |
| Syslog &#124; where SeverityLevel == "error" |Todos los registros de Syslog con gravedad de error. |
| Syslog &#124; summarize AggregatedValue = count() by Computer |Número de registros de Syslog por equipo. |
| Syslog &#124; summarize AggregatedValue = count() by Facility |Número de registros de Syslog por recurso. |

## <a name="next-steps"></a>Pasos siguientes
* Obtenga información acerca de las [consultas de registros](../log-query/log-query-overview.md) para analizar los datos recopilados de soluciones y orígenes de datos.
* Use [Campos personalizados](./custom-fields.md) para analizar datos de registros de Syslog en campos individuales.
* [Configure agentes de Linux](../learn/quick-collect-linux-computer.md) para recopilar otros tipos de datos.

