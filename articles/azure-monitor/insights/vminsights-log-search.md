---
title: Cómo consultar registros de Azure Monitor para VM
description: La solución Azure Monitor para VM recopila datos de registro y métricas. En este artículo se describen los registros y se incluyen consultas de ejemplo.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/12/2020
ms.openlocfilehash: 64884f07bc59e5ff2b29eac645ddb469ef3db465
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87325192"
---
# <a name="how-to-query-logs-from-azure-monitor-for-vms"></a>Cómo consultar registros de Azure Monitor para VM

Azure Monitor para VM recopila métricas de rendimiento y conexión, datos de inventario de proceso y equipo, e información sobre el estado, y reenvía estos datos al área de trabajo de Log Analytics en Azure Monitor.  Estos datos están disponibles para [consulta](../log-query/log-query-overview.md) en Azure Monitor. Estos datos se pueden aplicar a escenarios que incluyen la planeación de la migración, el análisis de la capacidad, la detección y la solución de problemas de rendimiento a petición.

## <a name="map-records"></a>Registros de asignación

Se genera un registro por hora para cada equipo y proceso únicos, además de los registros generados cuando un proceso o equipo se inicia o se integra en la característica de asignación de Azure Monitor para VM. Estos registros tienen las propiedades de las tablas siguientes. Los campos y valores de los eventos ServiceMapComputer_CL se asignan a los campos del recurso Equipo en la API ServiceMap de Azure Resource Manager. Los campos y valores de los eventos ServiceMapProcess_CL se asignan a los campos del recurso Proceso en la API ServiceMap de Azure Resource Manager. El campo ResourceName_s coincide con el campo de nombre del recurso correspondiente de Resource Manager. 

Hay propiedades generadas internamente que puede usar para identificar los equipos y procesos únicos:

- Equipo: Use *ResourceId* o *ResourceName_s* para identificar de forma exclusiva un equipo dentro de un área de trabajo de Log Analytics.
- Proceso: Use *ResourceId* para identificar de forma exclusiva un proceso dentro de un área de trabajo de Log Analytics. *ResourceName_s* es único dentro del contexto de la máquina en la que se está ejecutando el proceso (MachineResourceName_s) 

Puesto que pueden existir varios registros para un proceso y equipo especificados en un intervalo de tiempo concreto, las consultas pueden devolver más de un registro para el mismo proceso o equipo. Para incluir solo el registro más reciente, agregue `| summarize arg_max(TimeGenerated, *) by ResourceId` a la consulta.

### <a name="connections-and-ports"></a>Conexiones y puertos

La característica de métricas de conexión presenta dos nuevas tablas en los registros de Azure Monitor: VMConnection y VMBoundPort. Estas tablas proporcionan información acerca de las conexiones para una máquina (entrantes y salientes), así como los puertos del servidor que están abiertos o activos en estas. Las métricas de conexión también se exponen a través de API que proporcionan los medios para obtener una métrica específica durante un período de tiempo. Las conexiones TCP resultantes de *aceptar* en un socket de escucha son de entrada, mientras que las creadas al *conectarse* a un puerto y una IP concretos son de salida. La dirección de una conexión se representa mediante la propiedad Direction, que se puede definir como **inbound** u **outbound**. 

Los registros de estas tablas se generan a partir de los datos que notifica Dependency Agent. Cada registro representa una observación en un intervalo de tiempo de un minuto. La propiedad TimeGenerated indica el inicio del intervalo de tiempo. Cada registro contiene información para identificar la entidad correspondiente; es decir, conexión o puerto, así como las métricas asociadas con esa entidad. Actualmente, solo se notifica la actividad de red que tiene lugar mediante TCP a través de IPv4. 

#### <a name="common-fields-and-conventions"></a>Campos y convenciones comunes 

Los campos y las convenciones siguientes se aplican a VMConnection y VMBoundPort: 

- Equipo: nombre de dominio completo de la máquina que genera el informe. 
- AgentId: identificador único de una máquina con el agente de Log Analytics.  
- Máquina: nombre del recurso de Azure Resource Manager para la máquina expuesta por ServiceMap. Tiene el formato *m-{GUID}* , donde *GUID* coincide con el GUID de AgentId.  
- Proceso: nombre del recurso de Azure Resource Manager para el proceso expuesto por ServiceMap. Tiene el formato *p-{cadena hexadecimal}* . El proceso es único dentro de un ámbito de la máquina y, para generar un identificador de proceso único entre máquinas, combina los valores de los campos Máquina y Proceso. 
- ProcessName: nombre del archivo ejecutable del proceso de informes.
- Todas las direcciones IP son cadenas en formato canónico IPv4, como *13.107.3.160*. 

Para administrar el costo y la complejidad, los registros de conexión no representan conexiones de red físicas individuales. Varias conexiones de red físicas se agrupan en una conexión lógica, que, a continuación, se refleja en la tabla correspondiente.  Lo que significa que los registros de la tabla *VMConnection* representan una agrupación lógica, y no las conexiones físicas individuales que se observan. Las conexiones de red físicas que comparten el mismo valor para los siguientes atributos durante un intervalo determinado de un minuto se agregan en un registro lógico único en *VMConnection*. 

| Propiedad | Descripción |
|:--|:--|
|Dirección |Dirección de la conexión; el valor es *inbound* u *outbound* |
|Máquina |FQDN del equipo |
|Proceso |Identidad de proceso o grupos de procesos; iniciar/aceptar la conexión |
|SourceIp |Dirección IP de origen |
|DestinationIp |Dirección IP de destino. |
|DestinationPort |Número de puerto de destino |
|Protocolo |Protocolo utilizado para la conexión.  Los valores son *tcp*. |

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
4. Los puertos que están enlazados solo en una interfaz específica tienen IsWildcardBind definido como *False*.

#### <a name="naming-and-classification"></a>Nomenclatura y clasificación

Para mayor comodidad, la dirección IP del extremo remoto de una conexión se incluye en la propiedad RemoteIp. Para las conexiones entrantes, RemoteIp coincide con SourceIp, mientras que, para las conexiones salientes, coincide con DestinationIp. La propiedad RemoteDnsCanonicalNames representa los nombres canónicos DNS que notifica la máquina para RemoteIp. Las propiedades RemoteDnsQuestions y RemoteClassification están reservadas para uso futuro. 

#### <a name="geolocation"></a>Geolocalización

*VMConnection* también incluye información de ubicación geográfica para el extremo remoto de cada registro de conexión en las siguientes propiedades del registro: 

| Propiedad | Descripción |
|:--|:--|
|RemoteCountry |Nombre del país o región que hospeda la dirección IP de RemoteIp.  Por ejemplo: *Estados Unidos* |
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
|Confianza |Los valores válidos se encuentran entre *0 y 100*. |
|severity |Los valores se encuentran entre *0 y 5*, donde *5* es el más grave y *0* no es grave en absoluto. El valor predeterminado es *3*.  |
|FirstReportedDateTime |La primera vez que el proveedor informó sobre el indicador. |
|LastReportedDateTime |La última vez que Interflow ha visto el indicador. |
|IsActive |Indica que los indicadores se desactivan con el valor *True* o *False*. |
|ReportReferenceLink |Vincula a informes relacionados con un objeto observable especificado. |
|AdditionalInformation |Proporciona información adicional, si procede, sobre la amenaza observada. |

### <a name="ports"></a>Puertos 

En una máquina, los puertos que aceptan activamente el tráfico entrante o que pueden aceptar tráfico potencialmente, pero que están inactivas durante el período de tiempo de informes, se escriben en la tabla VMBoundPort.  

Todos los registros de VMBoundPort se identifican mediante los campos siguientes: 

| Propiedad | Descripción |
|:--|:--|
|Proceso | Identidad del proceso (o grupos de procesos) a la que está asociado el puerto.|
|Ip | Dirección IP del puerto (puede ser la dirección IP comodín, *0.0.0.0*). |
|Port |Número del puerto. |
|Protocolo | Protocolo.  Ejemplo, *tcp* o *udp* (solo se admite *tcp* actualmente).|
 
Identidad de la que se deriva un puerto de los cinco campos anteriores. Se almacena en la propiedad PortId. Esta propiedad se puede usar para buscar rápidamente los registros de un puerto específico a través del tiempo. 

#### <a name="metrics"></a>Métricas 

Los registros de puerto incluyen métricas que representen las conexiones asociadas. Actualmente, se notifican las métricas siguientes (los detalles de cada métrica se describen en la sección anterior): 

- BytesSent y BytesReceived 
- LinksEstablished, LinksTerminated y LinksLive 
- ResposeTime, ResponseTimeMin, ResponseTimeMax y ResponseTimeSum 

A continuación se incluyen puntos importantes que debe tener en cuenta:

- Si un proceso acepta conexiones en la misma dirección IP, pero a través de varias interfaces de red, se notificará un registro independiente para cada interfaz.  
- Los registros con IP comodín no contendrán ninguna actividad. Se incluyen para representar el hecho de que un puerto del equipo está abierto al tráfico de entrada. 
- Para reducir el nivel de detalle y el volumen de datos, los registros con dirección IP de carácter comodín se omitirán cuando haya un registro coincidente (para el mismo proceso, puerto y protocolo) con una dirección IP específica. Si un registro de dirección IP comodín se omite, la propiedad *IsWildcardBind* del registro con la dirección IP específica se establecerá en *True*.  Esto indica que el puerto se expone a través de todas las interfaces de la máquina del informe. 
- Los puertos que están enlazados solo en una interfaz específica tienen IsWildcardBind definido como *False*. 

### <a name="vmcomputer-records"></a>Registros de VMComputer

Los registros con un tipo de *VMComputer* tienen datos de inventario para servidores con Dependency Agent. Estos registros tienen las propiedades de la tabla siguiente:

| Propiedad | Descripción |
|:--|:--|
|TenantId | Identificador único del área de trabajo |
|SourceSystem | *Insights* | 
|TimeGenerated | Marca de tiempo del registro (UTC) |
|Computer | FQDN del equipo | 
|AgentId | Identificador único del agente de Log Analytics |
|Máquina | nombre del recurso de Azure Resource Manager para la máquina expuesta por ServiceMap. Tiene el formato *m-{GUID}* , donde *GUID* coincide con el GUID de AgentId. | 
|DisplayName | Nombre para mostrar | 
|FullDisplayName | Nombre para mostrar del plan | 
|HostName | Nombre de la máquina sin el nombre de dominio |
|BootTime | Hora de arranque de la máquina (UTC) |
|TimeZone | La zona horaria normalizada |
|VirtualizationState | *virtual*, *hypervisor*, *physical* |
|Ipv4Addresses | Matriz de direcciones IPv4 | 
|Ipv4SubnetMasks | Matriz de máscaras de subred IPv4 (en el mismo orden que Ipv4Addresses) |
|Ipv4DefaultGateways | Matriz de puertas de enlace IPv4 | 
|Ipv6Addresses | Matriz de direcciones IPv6 | 
|MacAddresses | Matriz de direcciones MAC | 
|DnsNames | Matriz de nombres DNS asociados a la máquina |
|DependencyAgentVersion | Versión del agente de Dependency Agent que se ejecuta en la máquina | 
|OperatingSystemFamily | *Linux*, *Windows* |
|OperatingSystemFullName | Nombre completo del sistema operativo | 
|PhysicalMemoryMB | Memoria física en MB | 
|Cpus | Número de procesadores | 
|CpuSpeed | Velocidad de la CPU en MHz | 
|VirtualMachineType | *hyperv*, *vmware*, *xen* |
|VirtualMachineNativeId | Identificador de máquina virtual asignado por su hipervisor | 
|VirtualMachineNativeName | Nombre de la máquina virtual |
|VirtualMachineHypervisorId | Identificador único del hipervisor que hospeda la máquina virtual |
|HypervisorType | *hyperv* |
|HypervisorId | Identificador único del hipervisor | 
|HostingProvider | *azure* |
|_ResourceId | Identificador único de un recurso de Azure |
|AzureSubscriptionId | Identificador único global que identifica la suscripción | 
|AzureResourceGroup | Nombre del grupo de recursos de Azure del que es miembro la máquina |
|AzureResourceName | Nombre del recurso de Azure |
|AzureLocation | Ubicación del grupo de recursos de Azure |
|AzureUpdateDomain | Nombre del dominio de actualización de Azure |
|AzureFaultDomain | Nombre del dominio de error de Azure |
|AzureVmId | Identificador único de la máquina virtual de Azure |
|AzureSize | Tamaño de la máquina virtual de Azure |
|AzureImagePublisher | Nombre del publicador de máquinas virtuales de Azure |
|AzureImageOffering | Nombre del tipo de oferta de máquina virtual de Azure | 
|AzureImageSku | SKU de la imagen de máquina virtual de Azure | 
|AzureImageVersion | Versión de la imagen de máquina virtual de Azure | 
|AzureCloudServiceName | Nombre del servicio en la nube de Azure |
|AzureCloudServiceDeployment | Identificador de implementación del servicio en la nube |
|AzureCloudServiceRoleName | Nombre del rol del servicio en la nube |
|AzureCloudServiceRoleType | Tipo de rol del servicio en la nube: *worker* o *web* |
|AzureCloudServiceInstanceId | Identificador de instancia del rol del servicio en la nube |
|AzureVmScaleSetName | Nombre de conjunto de escalado de máquinas virtuales |
|AzureVmScaleSetDeployment | Identificador de la implementación del conjunto de escalado de máquinas virtuales |
|AzureVmScaleSetResourceId | Identificador único del recurso del conjunto de escalado de máquinas virtuales|
|AzureVmScaleSetInstanceId | Identificador único del conjunto de escalado de máquinas virtuales |
|AzureServiceFabricClusterId | Identificador único del clúster de Azure Service Fabric | 
|AzureServiceFabricClusterName | Nombre del clúster de Azure Service Fabric |

### <a name="vmprocess-records"></a>Registro de VMProcess

Los registros con un tipo *VMProcess* tienen datos de inventario para procesos conectados mediante TCP en servidores con Dependency Agent. Estos registros tienen las propiedades de la tabla siguiente:

| Propiedad | Descripción |
|:--|:--|
|TenantId | Identificador único del área de trabajo |
|SourceSystem | *Insights* | 
|TimeGenerated | Marca de tiempo del registro (UTC) |
|Computer | FQDN del equipo | 
|AgentId | Identificador único del agente de Log Analytics |
|Máquina | nombre del recurso de Azure Resource Manager para la máquina expuesta por ServiceMap. Tiene el formato *m-{GUID}* , donde *GUID* coincide con el GUID de AgentId. | 
|Proceso | Identificador único del proceso de Service Map. Tiene el formato *p-{GUID}* . 
|ExecutableName | Nombre del archivo ejecutable del proceso | 
|DisplayName | Nombre para mostrar del proceso |
|Role | Rol de proceso: *webserver*, *appServer*, *databaseServer*, *ldapServer*, *smbServer* |
|Grupo | Nombre del grupo de procesos Los procesos del mismo grupo están relacionados de manera lógica; por ejemplo, parte del mismo producto o componente del sistema. |
|StartTime | Hora de inicio del grupo de procesos |
|FirstPid | Primer PID del grupo de procesos |
|Descripción | Descripción del proceso |
|CompanyName | Nombre de la compañía |
|InternalName | Nombre interno |
|ProductName | Nombre del producto |
|ProductVersion | Versión del producto |
|FileVersion | Versión del archivo |
|ExecutablePath _s |Ruta de acceso del archivo ejecutable |
|CommandLine | Línea de comandos |
|WorkingDirectory | Directorio de trabajo |
|Servicios | Matriz de servicios en la que se ejecuta el proceso |
|UserName | Cuenta en la que se está ejecutando el proceso |
|UserDomain | Dominio en el que se está ejecutando el proceso |
|_ResourceId | Identificador único de un proceso en el área de trabajo |


## <a name="sample-map-queries"></a>Consultas de mapa de ejemplo

### <a name="list-all-known-machines"></a>Enumerar todas las máquinas conocidas

```kusto
VMComputer | summarize arg_max(TimeGenerated, *) by _ResourceId
```

### <a name="when-was-the-vm-last-rebooted"></a>Cuando se reinicie la VM por última vez

```kusto
let Today = now(); VMComputer | extend DaysSinceBoot = Today - BootTime | summarize by Computer, DaysSinceBoot, BootTime | sort by BootTime asc
```

### <a name="summary-of-azure-vms-by-image-location-and-sku"></a>Resumen de las VM de Azure por imagen, ubicación y SKU

```kusto
VMComputer | where AzureLocation != "" | summarize by Computer, AzureImageOffering, AzureLocation, AzureImageSku
```

### <a name="list-the-physical-memory-capacity-of-all-managed-computers"></a>Enumeración de la capacidad de memoria física de todos los equipos administrados

```kusto
VMComputer | summarize arg_max(TimeGenerated, *) by _ResourceId | project PhysicalMemoryMB, Computer
```

### <a name="list-computer-name-dns-ip-and-os"></a>Enumeración del nombre de equipo, DNS, IP y SO

```kusto
VMComputer | summarize arg_max(TimeGenerated, *) by _ResourceId | project Computer, OperatingSystemFullName, DnsNames, Ipv4Addresses
```

### <a name="find-all-processes-with-sql-in-the-command-line"></a>Buscar todos los procesos con "sql" en la línea de comandos

```kusto
VMProcess | where CommandLine contains_cs "sql" | summarize arg_max(TimeGenerated, *) by _ResourceId
```

### <a name="find-a-machine-most-recent-record-by-resource-name"></a>Buscar una máquina (registro más reciente) por el nombre de recurso

```kusto
search in (VMComputer) "m-4b9c93f9-bc37-46df-b43c-899ba829e07b" | summarize arg_max(TimeGenerated, *) by _ResourceId
```

### <a name="find-a-machine-most-recent-record-by-ip-address"></a>Buscar un equipo (registro más reciente) por dirección IP

```kusto
search in (VMComputer) "10.229.243.232" | summarize arg_max(TimeGenerated, *) by _ResourceId
```

### <a name="list-all-known-processes-on-a-specified-machine"></a>Enumerar todos los procesos conocidos en un equipo determinado

```kusto
VMProcess | where Machine == "m-559dbcd8-3130-454d-8d1d-f624e57961bc" | summarize arg_max(TimeGenerated, *) by _ResourceId
```

### <a name="list-all-computers-running-sql-server"></a>Enumerar todos los equipos que ejecutan SQL Server

```kusto
VMComputer | where AzureResourceName in ((search in (VMProcess) "*sql*" | distinct Machine)) | distinct Computer
```

### <a name="list-all-unique-product-versions-of-curl-in-my-datacenter"></a>Enumerar todas las versiones de producto únicas de curl en mi centro de datos

```kusto
VMProcess | where ExecutableName == "curl" | distinct ProductVersion
```

### <a name="create-a-computer-group-of-all-computers-running-centos"></a>Crear un grupo de equipos de todos los equipos con CentOS

```kusto
VMComputer | where OperatingSystemFullName contains_cs "CentOS" | distinct Computer
```

### <a name="bytes-sent-and-received-trends"></a>Tendencias de bytes enviados y recibidos

```kusto
VMConnection | summarize sum(BytesSent), sum(BytesReceived) by bin(TimeGenerated,1hr), Computer | order by Computer desc | render timechart
```

### <a name="which-azure-vms-are-transmitting-the-most-bytes"></a>VM de Azure que transmiten más bytes

```kusto
VMConnection | join kind=fullouter(VMComputer) on $left.Computer == $right.Computer | summarize count(BytesSent) by Computer, AzureVMSize | sort by count_BytesSent desc
```

### <a name="link-status-trends"></a>Tendencias del estado del vínculo

```kusto
VMConnection | where TimeGenerated >= ago(24hr) | where Computer == "acme-demo" | summarize dcount(LinksEstablished), dcount(LinksLive), dcount(LinksFailed), dcount(LinksTerminated) by bin(TimeGenerated, 1h) | render timechart
```

### <a name="connection-failures-trend"></a>Tendencia de errores de conexión

```kusto
VMConnection | where Computer == "acme-demo" | extend bythehour = datetime_part("hour", TimeGenerated) | project bythehour, LinksFailed | summarize failCount = count() by bythehour | sort by bythehour asc | render timechart
```

### <a name="bound-ports"></a>Puertos enlazados

```kusto
VMBoundPort
| where TimeGenerated >= ago(24hr)
| where Computer == 'admdemo-appsvr'
| distinct Port, ProcessName
```

### <a name="number-of-open-ports-across-machines"></a>Número de puertos abiertos en las máquinas

```kusto
VMBoundPort
| where Ip != "127.0.0.1"
| summarize by Computer, Machine, Port, Protocol
| summarize OpenPorts=count() by Computer, Machine
| order by OpenPorts desc
```

### <a name="score-processes-in-your-workspace-by-the-number-of-ports-they-have-open"></a>Puntuación de procesos en el área de trabajo por el número de puertos que tienen abiertos

```kusto
VMBoundPort
| where Ip != "127.0.0.1"
| summarize by ProcessName, Port, Protocol
| summarize OpenPorts=count() by ProcessName
| order by OpenPorts desc
```

### <a name="aggregate-behavior-for-each-port"></a>Comportamiento agregado de cada puerto

A continuación, esta consulta se puede usar para puntuar los puertos por actividad (por ejemplo, puertos con más tráfico entrante y saliente o puertos con más conexiones).
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
let ips=materialize(VMComputer
| summarize ips=makeset(todynamic(Ipv4Addresses)) by MonitoredMachine=AzureResourceName
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

## <a name="performance-records"></a>Registros de rendimiento
Los registros con un tipo de *InsightsMetrics* tienen datos de rendimiento del sistema operativo invitado de la máquina virtual. Estos registros tienen las propiedades de la tabla siguiente:


| Propiedad | Descripción |
|:--|:--|
|TenantId | Identificador único del área de trabajo |
|SourceSystem | *Insights* | 
|TimeGenerated | Hora en que se recopiló el valor (UTC) |
|Computer | FQDN del equipo | 
|Origen | *vm.azm.ms* |
|Espacio de nombres | Categoría del contador de rendimiento | 
|Nombre | Nombre del contador de rendimiento. |
|Val | Valor recopilado | 
|Etiquetas | Detalles relacionados sobre el registro. Vea la tabla siguiente para consultar las etiquetas utilizadas con diferentes tipos de registros.  |
|AgentId | Identificador único de cada agente del equipo |
|Tipo | *InsightsMetrics* |
|_ResourceId_ | Identificador de recurso de la máquina virtual |

Los contadores de rendimiento recopilados actualmente en la tabla *InsightsMetrics* se enumeran en la tabla siguiente:

| Espacio de nombres | Nombre | Descripción | Unidad | Etiquetas |
|:---|:---|:---|:---|:---|
| Computer    | Latido             | Latido de equipo                        | | |
| Memoria      | AvailableMB           | Bytes disponibles en la memoria                    | Megabytes      | memorySizeMB: tamaño total de la memoria|
| Red     | WriteBytesPerSecond   | Bytes de escritura de red por segundo            | BytesPerSecond | NetworkDeviceId: identificador del dispositivo<br>bytes: total de bytes enviados |
| Red     | ReadBytesPerSecond    | Bytes de lectura de red por segundo             | BytesPerSecond | networkDeviceId: identificador del dispositivo<br>bytes: total de bytes recibidos |
| Procesador   | UtilizationPercentage | Porcentaje de uso del procesador          | Percent        | totalCpus: CPU totales |
| LogicalDisk | WritesPerSecond       | Escrituras en el disco lógico por segundo            | CountPerSecond | mountId: identificador de montaje del dispositivo |
| LogicalDisk | WriteLatencyMs        | Latencia de escritura en el disco lógico en milisegundos    | MilliSeconds   | mountId: identificador de montaje del dispositivo |
| LogicalDisk | WriteBytesPerSecond   | Bytes de escritura en el disco lógico por segundo       | BytesPerSecond | mountId: identificador de montaje del dispositivo |
| LogicalDisk | TransfersPerSecond    | Transferencias del disco lógico por segundo         | CountPerSecond | mountId: identificador de montaje del dispositivo |
| LogicalDisk | TransferLatencyMs     | Latencia de transferencia del disco lógico en milisegundos | MilliSeconds   | mountId: identificador de montaje del dispositivo |
| LogicalDisk | ReadsPerSecond        | Lecturas del disco lógico por segundo             | CountPerSecond | mountId: identificador de montaje del dispositivo |
| LogicalDisk | ReadLatencyMs         | Latencia de lectura del disco lógico en milisegundos     | MilliSeconds   | mountId: identificador de montaje del dispositivo |
| LogicalDisk | ReadBytesPerSecond    | Bytes de lectura de disco lógico por segundo        | BytesPerSecond | mountId: identificador de montaje del dispositivo |
| LogicalDisk | FreeSpacePercentage   | Porcentaje de espacio disponible en disco lógico        | Percent        | mountId: identificador de montaje del dispositivo |
| LogicalDisk | FreeSpaceMB           | Bytes de espacio disponible en el disco lógico             | Megabytes      | mountId: identificador de montaje del dispositivo<br>diskSizeMB: tamaño total del disco |
| LogicalDisk | BytesPerSecond        | Bytes del disco lógico por segundo             | BytesPerSecond | mountId: identificador de montaje del dispositivo |


## <a name="next-steps"></a>Pasos siguientes

* Si eres nuevo a la hora de escribir consultas en Azure Monitor, revisa el tema sobre el [uso de Log Analytics](../log-query/get-started-portal.md) en Azure Portal para escribir consultas de registro.

* Información acerca de la [escritura de consultas de búsqueda](../log-query/search-queries.md).

