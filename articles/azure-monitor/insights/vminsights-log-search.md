---
title: Cómo consultar registros de Azure Monitor para VM (versión preliminar) | Microsoft Docs
description: Monitor para la solución de máquinas virtuales de Azure recopila métricas y datos de registro y en este artículo describe los registros e incluye las consultas de ejemplo.
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: tysonn
ms.assetid: ''
ms.service: azure-monitor
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/10/2019
ms.author: magoedte
ms.openlocfilehash: 38979aa5cbb7eff0a949dfb77d6a29b2cdb5c67b
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65602077"
---
# <a name="how-to-query-logs-from-azure-monitor-for-vms-preview"></a>Cómo consultar registros de Azure Monitor para VM (versión preliminar)
Monitor para las máquinas virtuales de Azure recopila información de estado de mantenimiento y métricas de conexión, equipo y procesar los datos de inventario y rendimiento y la reenvía al área de trabajo de Log Analytics en Azure Monitor.  Estos datos están disponibles para [consulta](../../azure-monitor/log-query/log-query-overview.md) en Azure Monitor. Estos datos se pueden aplicar a escenarios que incluyen la planeación de la migración, el análisis de la capacidad, la detección y la solución de problemas de rendimiento a petición.

## <a name="map-records"></a>Registros de asignación
Se genera un registro por hora para cada equipo y proceso únicos, además de los registros generados cuando un proceso o equipo se inicia o se integra en la característica de asignación de Azure Monitor para VM. Estos registros tienen las propiedades de las tablas siguientes. Los campos y valores de los eventos ServiceMapComputer_CL se asignan a los campos del recurso Equipo en la API ServiceMap de Azure Resource Manager. Los campos y valores de los eventos ServiceMapProcess_CL se asignan a los campos del recurso Proceso en la API ServiceMap de Azure Resource Manager. El campo ResourceName_s coincide con el campo de nombre del recurso correspondiente de Resource Manager. 

Hay propiedades generadas internamente que puede usar para identificar los equipos y procesos únicos:

- Equipo: Use *ResourceId* o *ResourceName_s* para identificar de forma exclusiva un equipo dentro de un área de trabajo de Log Analytics.
- Proceso: Use *ResourceId* para identificar de forma exclusiva un proceso dentro de un área de trabajo de Log Analytics. *ResourceName_s* es único dentro del contexto de la máquina en la que se está ejecutando el proceso (MachineResourceName_s) 

Puesto que pueden existir varios registros para un proceso y equipo especificados en un intervalo de tiempo concreto, las consultas pueden devolver más de un registro para el mismo proceso o equipo. Para incluir solo el registro más reciente agregue "| dedup ResourceId" a la consulta.

### <a name="connections-and-ports"></a>Puertos y conexiones
La característica de métricas de conexión presenta dos nuevas tablas en los registros de Azure Monitor - VMConnection y VMBoundPort. Estas tablas proporcionan información acerca de las conexiones para una máquina (entrante y saliente), así como el servidor de puertos que se abra/activo en ellos. ConnectionMetrics también se exponen a través de API que proporcionan los medios para obtener una métrica específica durante un período de tiempo. Las conexiones TCP resultantes de *aceptando* en un socket de escucha son entrante, mientras que los creados por *conexión* a una determinada dirección IP y puerto son de salida. La dirección de una conexión se representa mediante la propiedad Direction, que se puede definir como **inbound** u **outbound**. 

Se generan los registros en estas tablas de datos notificados por el agente de dependencia. Cada registro representa una observación a través de un intervalo de tiempo de 1 minuto. La propiedad TimeGenerated indica el inicio del intervalo de tiempo. Cada registro contiene información para identificar la entidad correspondiente; es decir, conexión o puerto, así como las métricas asociadas con esa entidad. Actualmente, solo se notifica la actividad de red que tiene lugar mediante TCP a través de IPv4. 

#### <a name="common-fields-and-conventions"></a>Los campos y las convenciones comunes 
Los campos y las convenciones siguientes se aplican a VMConnection y VMBoundPort: 

- Equipo: Nombre de dominio completo del equipo de reporting 
- AgentID: El identificador único para un equipo con el agente de Log Analytics  
- Máquina: Nombre del recurso de Azure Resource Manager para la máquina expuesta por Service Map. Es el formato *m-{GUID}*, donde *GUID* es el mismo GUID como Id. de agente  
- Proceso: Nombre del recurso de Azure Resource Manager para el proceso expuesto por Service Map. Es el formato *p-{cadena hexadecimal}*. Proceso es único dentro de un ámbito de la máquina y para generar un identificador único del proceso entre máquinas, combinar campos de equipo y proceso. 
- Nombre del proceso: Nombre del archivo ejecutable del proceso de generación de informes.
- Todas las direcciones IP son cadenas en formato canónico de IPv4, por ejemplo *13.107.3.160* 

Para administrar el costo y la complejidad, los registros de conexión no representan conexiones de red físicas individuales. Varias conexiones de red físicas se agrupan en una conexión lógica, que, a continuación, se refleja en la tabla correspondiente.  Lo que significa que los registros de la tabla *VMConnection* representan una agrupación lógica, y no las conexiones físicas individuales que se observan. Las conexiones de red físicas que comparten el mismo valor para los siguientes atributos durante un intervalo determinado de un minuto se agregan en un registro lógico único en *VMConnection*. 

| Propiedad | Descripción |
|:--|:--|
|Direction |Dirección de la conexión; el valor es *inbound* u *outbound* |
|Machine |FQDN del equipo |
|Process |Identidad de proceso o grupos de procesos; iniciar/aceptar la conexión |
|SourceIp |Dirección IP de origen |
|DestinationIp |Dirección IP de destino. |
|DestinationPort |Número de puerto de destino |
|Protocol |Protocolo utilizado para la conexión.  Los valores son *tcp*. |

Para tener en cuenta el impacto de la agrupación, se proporciona información sobre el número de conexiones físicas agrupadas en las siguientes propiedades del registro:

| Propiedad | Descripción |
|:--|:--|
|LinksEstablished |Número de conexiones de red físicas que se han establecido durante el período de tiempo de generación de informes |
|LinksTerminated |Número de conexiones de red físicas que han finalizado durante el período de tiempo de generación de informes |
|LinksFailed |Número de conexiones de red físicas que han generado errores durante el período de tiempo de generación de informes Actualmente, esta información está disponible solo para las conexiones salientes. |
|LinksLive |Número de conexiones de red físicas que estaban abiertas al final del período de tiempo de generación de informes|

#### <a name="metrics"></a>Métricas

Además de las métricas de recuento de conexión, también se incluye información sobre el volumen de datos enviado y recibido en una conexión lógica o puerto de red concreto en las siguientes propiedades del registro:

| Propiedad | Descripción |
|:--|:--|
|BytesSent |Número total de bytes enviados durante el período de tiempo de generación de informes |
|BytesReceived |Número total de bytes recibidos durante el período de tiempo de generación de informes |
|Respuestas |Número de respuestas observado durante el período de tiempo de generación de informes. 
|ResponseTimeMax |Mayor tiempo de respuesta (milisegundos) observado durante el período de tiempo de generación de informes. Si no hay ningún valor, la propiedad está en blanco.|
|ResponseTimeMin |Menor tiempo de respuesta (milisegundos) observado durante el período de tiempo de generación de informes. Si no hay ningún valor, la propiedad está en blanco.|
|ResponseTimeMin |Suma de todos los tiempos de respuesta (milisegundos) observados durante el período de tiempo de generación de informes. Si no hay ningún valor, la propiedad está en blanco.|

El tercer tipo de datos que se va a notificar es el tiempo de respuesta: ¿cuánto tiempo pasa el autor de la llamada esperando que una solicitud enviada a través de una conexión se procese y reciba respuesta del punto de conexión remoto? El tiempo de respuesta notificado es una estimación del tiempo de respuesta real del protocolo de aplicación subyacente. Se calcula utilizando la heurística basada en la observación del flujo de datos entre el origen y destino de una conexión de red física. Conceptualmente, es la diferencia entre el momento en que el último byte de una solicitud sale del remitente y el momento en que llega el último byte de la respuesta. Estas dos marcas de tiempo se utilizan para delinear los eventos de solicitud y respuesta de una conexión física concreta. La diferencia entre ellas representa el tiempo de respuesta de una única solicitud. 

En esta primera versión de esta característica, el algoritmo es una aproximación que puede funcionar con cierto grado de éxito según el protocolo de aplicación real usado para una conexión de red concreta. Por ejemplo, el enfoque actual funciona bien para los protocolos basados en solicitud-respuesta, como HTTP(S), pero no funciona con protocolos unidireccionales ni basados en la cola de mensajes.

A continuación se incluyen puntos importantes que debe tener en cuenta:

1. Si un proceso acepta conexiones en la misma dirección IP, pero a través de varias interfaces de red, se notificará un registro independiente para cada interfaz. 
2. Los registros con IP comodín no contendrán ninguna actividad. Se incluyen para representar el hecho de que un puerto del equipo está abierto al tráfico de entrada.
3. Para reducir el nivel de detalle y el volumen de datos, los registros con dirección IP de carácter comodín se omitirán cuando haya un registro coincidente (para el mismo proceso, puerto y protocolo) con una dirección IP específica. Cuando un registro de dirección IP comodín se omite, la propiedad del registro IsWildcardBind con la dirección IP específica se definirá como "True" para indicar que el puerto se expone a través de cada interfaz de la máquina de generación de informes.
4. Puertos que están enlazados solo en una interfaz específica tienen IsWildcardBind establecido en *False*.

#### <a name="naming-and-classification"></a>Nomenclatura y clasificación
Para mayor comodidad, la dirección IP del extremo remoto de una conexión se incluye en la propiedad RemoteIp. Para las conexiones entrantes, RemoteIp coincide con SourceIp, mientras que, para las conexiones salientes, coincide con DestinationIp. La propiedad RemoteDnsCanonicalNames representa los nombres canónicos DNS que notifica la máquina para RemoteIp. Las propiedades RemoteDnsQuestions y RemoteClassification están reservadas para uso futuro. 

#### <a name="geolocation"></a>Geolocalización
*VMConnection* también incluye información de ubicación geográfica para el extremo remoto de cada registro de conexión en las siguientes propiedades del registro: 

| Propiedad | Descripción |
|:--|:--|
|RemoteCountry |El nombre del país/región RemoteIp de hospedaje.  Por ejemplo: *United States* |
|RemoteLatitude |Latitud de geolocalización. Por ejemplo, *47.68* |
|RemoteLongitude |Longitud de geolocalización. Por ejemplo, *-122.12* |

#### <a name="malicious-ip"></a>Direcciones IP malintencionadas
Todas las propiedades de RemoteIp de la tabla *VMConnection* se comparan con un conjunto de direcciones IP con actividad malintencionada conocida. Si el valor de RemoteIp se identifica como malintencionado, las propiedades siguientes se completarán (si la IP no se considera malintencionada, están vacías) en las siguientes propiedades del registro:

| Propiedad | Descripción |
|:--|:--|
|MaliciousIP |Dirección RemoteIp |
|IndicatorThreadType |El indicador de amenazas detectado es uno de los siguientes valores: *Botnet*, *C2*, *CryptoMining*, *Darknet*, *DDos* , *MaliciousUrl*, *Malware*, *Phishing*, *Proxy*, *PUA*, *Watchlist*.   |
|Descripción |Descripción de la amenaza observada. |
|TLPLevel |Nivel de protocolo de semáforo (TLP) es uno de los valores definidos: *blanco*, *verde*, *ámbar*, *rojo*. |
|Confidence |Los valores válidos se encuentran entre *0 y 100*. |
|Severity |Los valores se encuentran entre *0 y 5*, donde *5* es el más grave y *0* no es grave en absoluto. El valor predeterminado es *3*.  |
|FirstReportedDateTime |La primera vez que el proveedor informó sobre el indicador. |
|LastReportedDateTime |La última vez que Interflow ha visto el indicador. |
|IsActive |Indica que los indicadores se desactivan con el valor *True* o *False*. |
|ReportReferenceLink |Vincula a informes relacionados con un objeto observable especificado. |
|AdditionalInformation |Proporciona información adicional, si procede, sobre la amenaza observada. |

### <a name="ports"></a>Puertos 
Puertos en una máquina que activamente acepten el tráfico entrante o potencialmente podrían aceptar tráfico, pero están inactivas durante el período de tiempo de generación de informes, se escriben en la tabla VMBoundPort.  

>[!NOTE]
>Azure Monitor para las máquinas virtuales no admite recopilar y registrar datos de puerto en un área de trabajo de Log Analytics en las siguientes regiones:  
>- Este de EE. UU  
>- Europa occidental
>
> Recopilar estos datos está habilitada en el otro [regiones admitidas](vminsights-enable-overview.md#log-analytics) para Azure Monitor para las máquinas virtuales. 

Todos los registros de VMBoundPort se identifican mediante los siguientes campos: 

| Propiedad | Descripción |
|:--|:--|
|Process | Identidad de proceso (o grupos de procesos) que está asociado con el puerto.|
|Ip | Dirección IP de puerto (puede ser la dirección IP de carácter comodín, *0.0.0.0*) |
|Port |El número de puerto |
|Protocol | El protocolo.  Ejemplo, *tcp* o *udp* (sólo *tcp* actualmente se admite).|
 
La identidad de un puerto se deriva de los cinco campos anteriores y se almacena en la propiedad PortId. Esta propiedad se puede usar para buscar rápidamente los registros para un puerto específico a través del tiempo. 

#### <a name="metrics"></a>Métricas 
Registros de puerto incluyen métricas que representen las conexiones asociadas. Actualmente, se notifican las métricas siguientes (los detalles de cada métrica se describen en la sección anterior): 

- BytesSent y BytesReceived 
- LinksLive LinksEstablished, LinksTerminated, 
- ResposeTime, ResponseTimeMin, ResponseTimeMax, ResponseTimeSum 

A continuación se incluyen puntos importantes que debe tener en cuenta:

- Si un proceso acepta conexiones en la misma dirección IP, pero a través de varias interfaces de red, se notificará un registro independiente para cada interfaz.  
- Los registros con IP comodín no contendrán ninguna actividad. Se incluyen para representar el hecho de que un puerto del equipo está abierto al tráfico de entrada. 
- Para reducir el nivel de detalle y el volumen de datos, los registros con dirección IP de carácter comodín se omitirán cuando haya un registro coincidente (para el mismo proceso, puerto y protocolo) con una dirección IP específica. Cuando un registro de dirección IP de carácter comodín se omite, el *IsWildcardBind* propiedad para el registro con la dirección IP específica, se establecerá en *True*.  Esto indica que el puerto se expone a través de todas las interfaces de la máquina de creación de informes. 
- Puertos que están enlazados solo en una interfaz específica tienen IsWildcardBind establecido en *False*. 

### <a name="servicemapcomputercl-records"></a>Registros de ServiceMapComputer_CL
Los registros con un tipo de *ServiceMapComputer_CL* tienen datos de inventario para servidores con Dependency Agent. Estos registros tienen las propiedades de la tabla siguiente:

| Propiedad | Descripción |
|:--|:--|
| Type | *ServiceMapComputer_CL* |
| SourceSystem | *OpsManager* |
| ResourceId | Identificador único de una máquina en el área de trabajo |
| ResourceName_s | Identificador único de una máquina en el área de trabajo |
| ComputerName_s | FQDN del equipo |
| Ipv4Addresses_s | Lista de las direcciones IPv4 del servidor |
| Ipv6Addresses_s | Lista de las direcciones IPv6 del servidor |
| DnsNames_s | Matriz de nombres DNS |
| OperatingSystemFamily_s | Windows o Linux |
| OperatingSystemFullName_s | Nombre completo del sistema operativo  |
| Bitness_s | Valor de bits del equipo (32 o 64 bits)  |
| PhysicalMemory_d | Memoria física en MB |
| Cpus_d | Número de CPU |
| CpuSpeed_d | Velocidad de la CPU en MHz|
| VirtualizationState_s | *unknown*, *physical*, *virtual*, *hypervisor* |
| VirtualMachineType_s | *hyperv*, *vmware*, etc. |
| VirtualMachineNativeMachineId_g | Identificador de máquina virtual asignado por su hipervisor |
| VirtualMachineName_s | Nombre de la máquina virtual |
| BootTime_t | Tiempo de arranque |

### <a name="servicemapprocesscl-type-records"></a>Registros con un tipo ServiceMapProcess_CL
Los registros con un tipo *ServiceMapProcess_CL* tienen datos de inventario para procesos con conexión TCP en servidores con Dependency Agent. Estos registros tienen las propiedades de la tabla siguiente:

| Propiedad | Descripción |
|:--|:--|
| Type | *ServiceMapProcess_CL* |
| SourceSystem | *OpsManager* |
| ResourceId | Identificador único de un proceso en el área de trabajo |
| ResourceName_s | Identificador único de un proceso en el equipo en el que se está ejecutando|
| MachineResourceName_s | Nombre de recurso del equipo |
| ExecutableName_s | Nombre del archivo ejecutable del proceso |
| StartTime_t | Hora de inicio del grupo de procesos |
| FirstPid_d | Primer PID del grupo de procesos |
| Description_s | Descripción del proceso |
| CompanyName_s | Nombre de la compañía |
| InternalName_s | Nombre interno |
| ProductName_s | Nombre del producto |
| ProductVersion_s | Versión del producto |
| FileVersion_s | Versión del archivo |
| CommandLine_s | Línea de comandos |
| ExecutablePath_s | Ruta de acceso al archivo ejecutable |
| WorkingDirectory_s | Directorio de trabajo |
| Nombre de usuario | Cuenta en la que se está ejecutando el proceso |
| UserDomain | Dominio en el que se está ejecutando el proceso |

## <a name="sample-log-searches"></a>Búsquedas de registros de ejemplo

### <a name="list-all-known-machines"></a>Enumerar todas las máquinas conocidas
```kusto
ServiceMapComputer_CL | summarize arg_max(TimeGenerated, *) by ResourceId`
```

### <a name="when-was-the-vm-last-rebooted"></a>Cuando se reinicie la VM por última vez
```kusto
let Today = now(); ServiceMapComputer_CL | extend DaysSinceBoot = Today - BootTime_t | summarize by Computer, DaysSinceBoot, BootTime_t | sort by BootTime_t asc`
```

### <a name="summary-of-azure-vms-by-image-location-and-sku"></a>Resumen de las VM de Azure por imagen, ubicación y SKU
```kusto
ServiceMapComputer_CL | where AzureLocation_s != "" | summarize by ComputerName_s, AzureImageOffering_s, AzureLocation_s, AzureImageSku_s`
```

### <a name="list-the-physical-memory-capacity-of-all-managed-computers"></a>Enumerar la capacidad de memoria física de todos los equipos administrados.
```kusto
ServiceMapComputer_CL | summarize arg_max(TimeGenerated, *) by ResourceId | project PhysicalMemory_d, ComputerName_s`
```

### <a name="list-computer-name-dns-ip-and-os"></a>Enumerar el nombre de equipo, DNS, IP y SO.
```kusto
ServiceMapComputer_CL | summarize arg_max(TimeGenerated, *) by ResourceId | project ComputerName_s, OperatingSystemFullName_s, DnsNames_s, Ipv4Addresses_s`
```

### <a name="find-all-processes-with-sql-in-the-command-line"></a>Buscar todos los procesos con "sql" en la línea de comandos
```kusto
ServiceMapProcess_CL | where CommandLine_s contains_cs "sql" | summarize arg_max(TimeGenerated, *) by ResourceId`
```

### <a name="find-a-machine-most-recent-record-by-resource-name"></a>Buscar una máquina (registro más reciente) por el nombre de recurso
```kusto
search in (ServiceMapComputer_CL) "m-4b9c93f9-bc37-46df-b43c-899ba829e07b" | summarize arg_max(TimeGenerated, *) by ResourceId`
```

### <a name="find-a-machine-most-recent-record-by-ip-address"></a>Buscar un equipo (registro más reciente) por dirección IP
```kusto
search in (ServiceMapComputer_CL) "10.229.243.232" | summarize arg_max(TimeGenerated, *) by ResourceId`
```

### <a name="list-all-known-processes-on-a-specified-machine"></a>Enumerar todos los procesos conocidos en un equipo determinado
```kusto
ServiceMapProcess_CL | where MachineResourceName_s == "m-559dbcd8-3130-454d-8d1d-f624e57961bc" | summarize arg_max(TimeGenerated, *) by ResourceId`
```

### <a name="list-all-computers-running-sql-server"></a>Enumerar todos los equipos que ejecutan SQL Server
```kusto
ServiceMapComputer_CL | where ResourceName_s in ((search in (ServiceMapProcess_CL) "\*sql\*" | distinct MachineResourceName_s)) | distinct ComputerName_s`
```

### <a name="list-all-unique-product-versions-of-curl-in-my-datacenter"></a>Enumerar todas las versiones de producto únicas de curl en mi centro de datos
```kusto
ServiceMapProcess_CL | where ExecutableName_s == "curl" | distinct ProductVersion_s`
```

### <a name="create-a-computer-group-of-all-computers-running-centos"></a>Crear un grupo de equipos de todos los equipos con CentOS
```kusto
ServiceMapComputer_CL | where OperatingSystemFullName_s contains_cs "CentOS" | distinct ComputerName_s`
```

### <a name="bytes-sent-and-received-trends"></a>Tendencias de bytes enviados y recibidos
```kusto
VMConnection | summarize sum(BytesSent), sum(BytesReceived) by bin(TimeGenerated,1hr), Computer | order by Computer desc | render timechart`
```

### <a name="which-azure-vms-are-transmitting-the-most-bytes"></a>VM de Azure que transmiten más bytes
```kusto
VMConnection | join kind=fullouter(ServiceMapComputer_CL) on $left.Computer == $right.ComputerName_s | summarize count(BytesSent) by Computer, AzureVMSize_s | sort by count_BytesSent desc`
```

### <a name="link-status-trends"></a>Tendencias del estado del vínculo
```kusto
VMConnection | where TimeGenerated >= ago(24hr) | where Computer == "acme-demo" | summarize  dcount(LinksEstablished), dcount(LinksLive), dcount(LinksFailed), dcount(LinksTerminated) by bin(TimeGenerated, 1h) | render timechart`
```

### <a name="connection-failures-trend"></a>Tendencia de errores de conexión
```kusto
VMConnection | where Computer == "acme-demo" | extend bythehour = datetime_part("hour", TimeGenerated) | project bythehour, LinksFailed | summarize failCount = count() by bythehour | sort by bythehour asc | render timechart`
```

### <a name="bound-ports"></a>Puertos de enlace
```kusto
VMBoundPort
| where TimeGenerated >= ago(24hr)
| where Computer == 'admdemo-appsvr'
| distinct Port, ProcessName
```

### <a name="number-of-open-ports-across-machines"></a>Número de puertos abiertos a través de máquinas
```kusto
VMBoundPort
| where Ip != "127.0.0.1"
| summarize by Computer, Machine, Port, Protocol
| summarize OpenPorts=count() by Computer, Machine
| order by OpenPorts desc
```

### <a name="score-processes-in-your-workspace-by-the-number-of-ports-they-have-open"></a>Puntuación de procesos en el área de trabajo por el número de puertos que tienen abierto
```kusto
VMBoundPort
| where Ip != "127.0.0.1"
| summarize by ProcessName, Port, Protocol
| summarize OpenPorts=count() by ProcessName
| order by OpenPorts desc
```

### <a name="aggregate-behavior-for-each-port"></a>Comportamiento agregado para cada puerto
Esta consulta, a continuación, se puede usar para puntuar los puertos por actividad, por ejemplo, con la mayor parte del tráfico entrante y saliente, puertos con la mayoría de las conexiones
```kusto
// 
VMBoundPort
| where Ip != "127.0.0.1"
| summarize BytesSent=sum(BytesSent), BytesReceived=sum(BytesReceived), LinksEstablished=sum(LinksEstablished), LinksTerminated=sum(LinksTerminated), arg_max(TimeGenerated, LinksLive) by Machine, Computer, ProcessName, Ip, Port, IsWildcardBind
| project-away TimeGenerated
| order by Machine, Computer, Port, Ip, ProcessName
```

### <a name="summarize-the-outbound-connections-from-a-group-of-machines"></a>Resumir las conexiones salientes desde un grupo de máquinas
```kusto
// the machines of interest
let machines = datatable(m: string) ["m-82412a7a-6a32-45a9-a8d6-538354224a25"];
// map of ip to monitored machine in the environment
let ips=materialize(ServiceMapComputer_CL
| summarize ips=makeset(todynamic(Ipv4Addresses_s)) by MonitoredMachine=ResourceName_s
| mvexpand ips to typeof(string));
// all connections to/from the machines of interest
let out=materialize(VMConnection
| where Machine in (machines)
| summarize arg_max(TimeGenerated, *) by ConnectionId);
// connections to localhost augmented with RemoteMachine
let local=out
| where RemoteIp startswith "127."
| project ConnectionId, Direction, Machine, Process, ProcessName, SourceIp, DestinationIp, DestinationPort, Protocol, RemoteIp, RemoteMachine=Machine;
// connections not to localhost augmented with RemoteMachine
let remote=materialize(out
| where RemoteIp !startswith "127."
| join kind=leftouter (ips) on $left.RemoteIp == $right.ips
| summarize by ConnectionId, Direction, Machine, Process, ProcessName, SourceIp, DestinationIp, DestinationPort, Protocol, RemoteIp, RemoteMachine=MonitoredMachine);
// the remote machines to/from which we have connections
let remoteMachines = remote | summarize by RemoteMachine;
// all augmented connections
(local)
| union (remote)
//Take all outbound records but only inbound records that come from either //unmonitored machines or monitored machines not in the set for which we are computing dependencies.
| where Direction == 'outbound' or (Direction == 'inbound' and RemoteMachine !in (machines))
| summarize by ConnectionId, Direction, Machine, Process, ProcessName, SourceIp, DestinationIp, DestinationPort, Protocol, RemoteIp, RemoteMachine
// identify the remote port
| extend RemotePort=iff(Direction == 'outbound', DestinationPort, 0)
// construct the join key we'll use to find a matching port
| extend JoinKey=strcat_delim(':', RemoteMachine, RemoteIp, RemotePort, Protocol)
// find a matching port
| join kind=leftouter (VMBoundPort 
| where Machine in (remoteMachines) 
| summarize arg_max(TimeGenerated, *) by PortId 
| extend JoinKey=strcat_delim(':', Machine, Ip, Port, Protocol)) on JoinKey
// aggregate the remote information
| summarize Remote=makeset(iff(isempty(RemoteMachine), todynamic('{}'), pack('Machine', RemoteMachine, 'Process', Process1, 'ProcessName', ProcessName1))) by ConnectionId, Direction, Machine, Process, ProcessName, SourceIp, DestinationIp, DestinationPort, Protocol
```

## <a name="next-steps"></a>Pasos siguientes
* Si está familiarizado con la escritura de consultas de registro en Azure Monitor, revisar [cómo usar Log Analytics](../../azure-monitor/log-query/get-started-portal.md) en el portal de Azure para escribir consultas de registros.
* Información acerca de la [escritura de consultas de búsqueda](../../azure-monitor/log-query/search-queries.md).
