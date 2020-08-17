---
title: Esquema de Windows Diagnostics Extension
description: Referencia del esquema de configuración de Windows Diagnostics Extension (WAD) en Azure Monitor.
ms.subservice: diagnostic-extension
ms.topic: reference
author: bwren
ms.author: bwren
ms.date: 01/20/2020
ms.openlocfilehash: d2b1afea746410e966b43bef01a039a8471d4ae7
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87007935"
---
# <a name="windows-diagnostics-extension-schema"></a>Esquema de Windows Diagnostics Extension
Azure Diagnostics Extension es un agente de Azure Monitor que recopila datos de supervisión del sistema operativo invitado y de las cargas de trabajo de los recursos de proceso. En este artículo, se describe el esquema que se usa para configurar Diagnostics Extension en máquinas virtuales Windows y otros recursos de proceso.

> [!NOTE]
> El esquema de este artículo es válido para la versión 1.3 y posteriores (SDK 2.4 de Azure y versiones posteriores). Las secciones de configuración más recientes incluyen comentarios para mostrar en qué versión se agregaron. Las versiones 1.0 y 1.2 del esquema se han archivado y ya no está disponibles. 

## <a name="public-configuration-file-schema"></a>Esquema del archivo de configuración pública

Descargue la definición del esquema del archivo de configuración público ejecutando el comando de PowerShell siguiente:  

```powershell  
(Get-AzureServiceAvailableExtension -ExtensionName 'PaaSDiagnostics' -ProviderNamespace 'Microsoft.Azure.Diagnostics').PublicConfigurationSchema | Out-File –Encoding utf8 -FilePath 'C:\temp\WadConfig.xsd'  
```  


## <a name="common-attribute-types"></a>Tipos de atributos comunes  
 El atributo **scheduledTransferPeriod** aparece en varios elementos. Es el intervalo entre las transferencias programadas en el almacenamiento, redondeado al minuto más cercano. El valor es un [“tipo de datos de duración” XML](https://www.w3schools.com/xml/schema_dtypes_date.asp).


## <a name="diagnosticsconfiguration-element"></a>Elemento DiagnosticsConfiguration  
 *Tree: Root - DiagnosticsConfiguration*

Agregado en la versión 1.3.  

Elemento de nivel superior del archivo de configuración de diagnóstico.  

**Atributo**: xmlns - El espacio de nombres XML del archivo de configuración de diagnóstico es el siguiente:  
`http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration`


|Elementos secundarios|Descripción|  
|--------------------|-----------------|  
|**PublicConfig**|Necesario. Consulte la descripción en cualquier parte de esta página.|  
|**PrivateConfig**|Opcional. Consulte la descripción en cualquier parte de esta página.|  
|**IsEnabled**|booleano. Consulte la descripción en cualquier parte de esta página.|  

## <a name="publicconfig-element"></a>Elemento PublicConfig  
 *Tree: Root - DiagnosticsConfiguration - PublicConfig*

 Describe la configuración de diagnóstico pública.  

|Elementos secundarios|Descripción|  
|--------------------|-----------------|  
|**WadCfg**|Necesario. Consulte la descripción en cualquier parte de esta página.|  
|**StorageAccount**|El nombre de la cuenta de Azure Storage en la que se van a almacenar los datos. También se puede especificar como un parámetro al ejecutar el cmdlet Set-AzureServiceDiagnosticsExtension.|  
|**StorageType**|Puede ser *Table*, *Blob* o *TableAndBlob*. Table es el valor predeterminado. Cuando se elige TableAndBlob, los datos de diagnóstico se escriben dos veces: una vez en cada tipo.|  
|**LocalResourceDirectory**|El directorio en la máquina virtual donde Monitoring Agent almacena los datos del evento. Si no se establece, se usa el directorio predeterminado:<br /><br /> Para un rol de trabajo o web: `C:\Resources\<guid>\directory\<guid>.<RoleName.DiagnosticStore\`<br /><br /> Para una máquina virtual: `C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<WADVersion>\WAD<WADVersion>`<br /><br /> Los atributos necesarios son:<br /><br /> - **path**: el directorio del sistema que va a usar Diagnósticos de Azure.<br /><br /> - **expandEnvironment**: controla si se expanden las variables de entorno en el nombre de ruta de acceso.|  

## <a name="wadcfg-element"></a>Elemento WadCFG  
 *Tree: Root - DiagnosticsConfiguration - PublicConfig - WadCFG*

 Identifica y configura los datos de telemetría que se van a recopilar.  


## <a name="diagnosticmonitorconfiguration-element"></a>Elemento DiagnosticMonitorConfiguration
 *Tree: Root - DiagnosticsConfiguration - PublicConfig - WadCFG - DiagnosticMonitorConfiguration*

 Obligatorio

|Atributos|Descripción|  
|----------------|-----------------|  
| **overallQuotaInMB** | La cantidad máxima de espacio en disco local que se puede utilizar en los distintos tipos de datos de diagnóstico que recopila Azure Diagnostics. La configuración predeterminada es 4096 MB.<br />
|**useProxyServer** | Configure Azure Diagnostics para utilizar la configuración del servidor proxy tal como se estableció en la configuración de Internet Explorer.|
|**sinks** | Agregado en 1.5. Opcional. Apunta a una ubicación de receptor para enviar datos de diagnóstico de todos los elementos secundarios que admiten receptores. Ejemplo de receptor es Application Insights o Event Hubs. Tenga en cuenta que debe agregar la propiedad *resourceId* en el elemento *Metrics* si desea que los eventos cargados en Event Hubs tengan un identificador de recurso. |  


<br /> <br />

|Elementos secundarios|Descripción|  
|--------------------|-----------------|  
|**CrashDumps**|Consulte la descripción en cualquier parte de esta página.|  
|**DiagnosticInfrastructureLogs**|Habilite la recopilación de registros generados por Diagnósticos de Azure. Los registros de infraestructura de diagnóstico son útiles para solucionar problemas del mismo sistema de diagnóstico. Los atributos opcionales son:<br /><br /> - **scheduledTransferLogLevelFilter**: configura el nivel de gravedad mínimo de los registros recopilados.<br /><br /> - **scheduledTransferPeriod**: el intervalo existente entre las transferencias programadas en el almacenamiento, redondeado al minuto más cercano. El valor es un [“tipo de datos de duración” XML](https://www.w3schools.com/xml/schema_dtypes_date.asp). |  
|**Directorios**|Consulte la descripción en cualquier parte de esta página.|  
|**EtwProviders**|Consulte la descripción en cualquier parte de esta página.|  
|**Métricas**|Consulte la descripción en cualquier parte de esta página.|  
|**PerformanceCounters**|Consulte la descripción en cualquier parte de esta página.|  
|**WindowsEventLog**|Consulte la descripción en cualquier parte de esta página.|
|**DockerSources**|Consulte la descripción en cualquier parte de esta página. |



## <a name="crashdumps-element"></a>Elemento CrashDumps  
 *Tree: Root - DiagnosticsConfiguration - PublicConfig - WadCFG - DiagnosticMonitorConfiguration - CrashDumps*

 Habilita la recopilación de volcados de memoria.  

|Atributos|Descripción|  
|----------------|-----------------|  
|**containerName**|Opcional. El nombre del contenedor de blobs en la cuenta de Azure Storage que se usará para almacenar los volcados de memoria.|  
|**crashDumpType**|Opcional.  Configura Diagnósticos de Azure para recopilar volcados de memoria parciales o completos.|  
|**directoryQuotaPercentage**|Opcional.  Configura el porcentaje de **overallQuotaInMB** que se va a reservar para los volcados de memoria en la máquina virtual.|  

|Elementos secundarios|Descripción|  
|--------------------|-----------------|  
|**CrashDumpConfiguration**|Necesario. Define los valores de configuración para cada proceso.<br /><br /> También se necesita el atributo siguiente:<br /><br /> **processName**: el nombre del proceso para el que desea que Diagnósticos de Azure recopile un volcado de memoria.|  

## <a name="directories-element"></a>Elemento Directories
 *Tree: Root - DiagnosticsConfiguration - PublicConfig - WadCFG - DiagnosticMonitorConfiguration -  Directories*

 Habilita la recopilación del contenido de un directorio, los registros de solicitud de acceso de error de IIS o los registros de IIS.  

 Atributo **scheduledTransferPeriod** opcional. Consulte la explicación anterior.  

|Elementos secundarios|Descripción|  
|--------------------|-----------------|  
|**IISLogs**|La inclusión de este elemento en la configuración habilita la recopilación de registros de IIS:<br /><br /> **containerName**: el nombre del contenedor de blobs en la cuenta de Azure Storage que se usará para almacenar los registros de IIS.|   
|**FailedRequestLogs**|La inclusión de este elemento en la configuración habilita la recopilación de registros sobre las solicitudes erróneas en un sitio o aplicación de IIS. También debe habilitar las opciones de seguimiento en **system.WebServer** de **Web.config**.|  
|**DataSources**|Una lista de directorios que se van a supervisar.|




## <a name="datasources-element"></a>Elemento DataSources  
 *Tree: Root - DiagnosticsConfiguration - PublicConfig - WadCFG - DiagnosticMonitorConfiguration - Directories - DataSources*

 Una lista de directorios que se van a supervisar.  

|Elementos secundarios|Descripción|  
|--------------------|-----------------|  
|**DirectoryConfiguration**|Necesario. Atributo necesario:<br /><br /> **containerName**: el nombre del contenedor de blobs en la cuenta de Azure Storage que se usará para almacenar los archivos de registro.|  





## <a name="directoryconfiguration-element"></a>Elemento DirectoryConfiguration  
 *Tree: Root - DiagnosticsConfiguration - PublicConfig - WadCFG - DiagnosticMonitorConfiguration - Directories - DataSources - DirectoryConfiguration*

 Puede incluir el elemento **Absolute** o el elemento **LocalResource**, pero no ambos.  

|Elementos secundarios|Descripción|  
|--------------------|-----------------|  
|**Absolute**|La ruta de acceso absoluta al directorio que se va a supervisar. Los atributos siguientes son necesarios:<br /><br /> - **Path**: la ruta de acceso absoluta al directorio que se va a supervisar.<br /><br /> - **expandEnvironment**: configura si se expanden las variables de entorno en Path.|  
|**LocalResource**|La ruta de acceso relativa a un recurso local que se va a supervisar. Los atributos necesarios son:<br /><br /> - **Name**: el recurso local que contiene el directorio que se va a supervisar<br /><br /> - **relativePath**: la ruta de acceso relativa al nombre que contiene el directorio que se va a supervisar|  



## <a name="etwproviders-element"></a>Elemento EtwProviders  
 *Tree: Root - DiagnosticsConfiguration - PublicConfig - WadCFG - DiagnosticMonitorConfiguration - EtwProviders*

 Configura la recopilación de eventos ETW en EventSource o el manifiesto de ETW en función de los proveedores.  

|Elementos secundarios|Descripción|  
|--------------------|-----------------|  
|**EtwEventSourceProviderConfiguration**|Configura la recopilación de eventos generados a partir de la [clase EventSource](/dotnet/api/system.diagnostics.tracing.eventsource?view=netcore-3.1). Atributo necesario:<br /><br /> **provider**: el nombre de clase del evento EventSource.<br /><br /> Los atributos opcionales son:<br /><br /> - **scheduledTransferLogLevelFilter**: el nivel de gravedad mínimo para transferir a la cuenta de almacenamiento.<br /><br /> - **scheduledTransferPeriod**: el intervalo existente entre las transferencias programadas en el almacenamiento, redondeado al minuto más cercano. El valor es un [“tipo de datos de duración” XML](https://www.w3schools.com/xml/schema_dtypes_date.asp). |  
|**EtwManifestProviderConfiguration**|Atributo necesario:<br /><br /> **provider**: el GUID del proveedor de eventos<br /><br /> Los atributos opcionales son:<br /><br /> - **scheduledTransferLogLevelFilter**: el nivel de gravedad mínimo para transferir a la cuenta de almacenamiento.<br /><br /> - **scheduledTransferPeriod**: el intervalo existente entre las transferencias programadas en el almacenamiento, redondeado al minuto más cercano. El valor es un [“tipo de datos de duración” XML](https://www.w3schools.com/xml/schema_dtypes_date.asp). |  



## <a name="etweventsourceproviderconfiguration-element"></a>Elemento EtwEventSourceProviderConfiguration  
 *Tree: Root - DiagnosticsConfiguration - PublicConfig - WadCFG - DiagnosticMonitorConfiguration - EtwProviders- EtwEventSourceProviderConfiguration*

 Configura la recopilación de eventos generados a partir de la [clase EventSource](/dotnet/api/system.diagnostics.tracing.eventsource?view=netcore-3.1).  

|Elementos secundarios|Descripción|  
|--------------------|-----------------|  
|**DefaultEvents**|Atributo opcional:<br/><br/> **eventDestination**: el nombre de la tabla en la que se van a almacenar los eventos|  
|**Evento**|Atributo necesario:<br /><br /> **id**: el identificador del evento.<br /><br /> Atributo opcional:<br /><br /> **eventDestination**: el nombre de la tabla en la que se van a almacenar los eventos|  



## <a name="etwmanifestproviderconfiguration-element"></a>Elemento EtwManifestProviderConfiguration  
 *Tree: Root - DiagnosticsConfiguration - PublicConfig - WadCFG - DiagnosticMonitorConfiguration - EtwProviders - EtwManifestProviderConfiguration*

|Elementos secundarios|Descripción|  
|--------------------|-----------------|  
|**DefaultEvents**|Atributo opcional:<br /><br /> **eventDestination**: el nombre de la tabla en la que se van a almacenar los eventos|  
|**Evento**|Atributo necesario:<br /><br /> **id**: el identificador del evento.<br /><br /> Atributo opcional:<br /><br /> **eventDestination**: el nombre de la tabla en la que se van a almacenar los eventos|  



## <a name="metrics-element"></a>Elemento Metrics  
 *Tree: Root - DiagnosticsConfiguration - PublicConfig - WadCFG - DiagnosticMonitorConfiguration - Metrics*

 Le permite generar una tabla de contadores de rendimiento optimizada para las consultas rápidas. Cada contador de rendimiento que se define en el elemento **PerformanceCounters** se almacena en la tabla de métricas además de la tabla de contadores de rendimiento.  

 El atributo **resourceId** es necesario.  El identificador de recurso de la máquina virtual o conjunto de escalado de máquinas virtuales en donde se va a implementar Azure Diagnostics. Obtenga el valor de **resourceID** en [Azure Portal](https://portal.azure.com). Seleccione **Examinar** -> **Grupos de recursos** ->  **<Nombre\>** . Haga clic en el icono **Propiedades** y copie el valor del campo **ID**.  Esta propiedad resourceID se usa para enviar métricas personalizadas y para agregar una propiedad resourceID a los datos enviados a Event Hubs. Tenga en cuenta que debe agregar la propiedad *resourceId* en el elemento *Metrics* si desea que los eventos cargados en Event Hubs tengan un identificador de recurso.

|Elementos secundarios|Descripción|  
|--------------------|-----------------|  
|**MetricAggregation**|Atributo necesario:<br /><br /> **scheduledTransferPeriod**: el intervalo existente entre las transferencias programadas en el almacenamiento, redondeado al minuto más cercano. El valor es un [“tipo de datos de duración” XML](https://www.w3schools.com/xml/schema_dtypes_date.asp). |  



## <a name="performancecounters-element"></a>Elemento PerformanceCounters  
 *Tree: Root - DiagnosticsConfiguration - PublicConfig - WadCFG - DiagnosticMonitorConfiguration - PerformanceCounters*

 Habilita la recopilación de contadores de rendimiento.  

 Atributo opcional:  

 Atributo **scheduledTransferPeriod** opcional. Consulte la explicación anterior.

|Elemento secundario|Descripción|  
|-------------------|-----------------|  
|**PerformanceCounterConfiguration**|Los atributos siguientes son necesarios:<br /><br /> - **counterSpecifier**: el nombre del contador de rendimiento. Por ejemplo, `\Processor(_Total)\% Processor Time`. Para obtener una lista de contadores de rendimiento en el host, ejecute el comando `typeperf`.<br /><br /> - **sampleRate**: la frecuencia de muestreo del contador.<br /><br /> Atributo opcional:<br /><br /> **unit**: la unidad de medida del contador. Los valores están disponibles en la [clase UnitType](/dotnet/api/microsoft.azure.management.sql.models.unittype?view=azure-dotnet) |
|**sinks** | Agregado en 1.5. Opcional. Apunta a una ubicación de receptor para enviar datos de diagnóstico. Por ejemplo, Azure Monitor o Event Hubs. Tenga en cuenta que debe agregar la propiedad *resourceId* en el elemento *Metrics* si desea que los eventos cargados en Event Hubs tengan un identificador de recurso.|    




## <a name="windowseventlog-element"></a>Elemento WindowsEventLog
 *Tree: Root - DiagnosticsConfiguration - PublicConfig - WadCFG - DiagnosticMonitorConfiguration - WindowsEventLog*

 Habilita la recopilación de registros de eventos de Windows.  

 Atributo **scheduledTransferPeriod** opcional. Consulte la explicación anterior.  

|Elemento secundario|Descripción|  
|-------------------|-----------------|  
|**DataSource**|Los registros de eventos de Windows que se van a recopilar. Atributo necesario:<br /><br /> **name**: la consulta de XPath que describe los eventos de Windows que se van a recopilar. Por ejemplo:<br /><br /> `Application!*[System[(Level <=3)]], System!*[System[(Level <=3)]], System!*[System[Provider[@Name='Microsoft Antimalware']]], Security!*[System[(Level <= 3)]`<br /><br /> Para recopilar todos los eventos, especifique "*". |
|**sinks** | Agregado en 1.5. Opcional. Apunta a una ubicación de receptor para enviar datos de diagnóstico de todos los elementos secundarios que admiten receptores. Ejemplo de receptor es Application Insights o Event Hubs.|  


## <a name="logs-element"></a>Elemento Logs  
 *Tree: Root - DiagnosticsConfiguration - PublicConfig - WadCFG - DiagnosticMonitorConfiguration - Logs*

 Se presenta en la versión 1.0 y 1.1. Falta en 1.2. Se ha agregado en 1.3.  

 Define la configuración del búfer para los registros básicos de Azure.  

|Atributo|Tipo|Descripción|  
|---------------|----------|-----------------|  
|**bufferQuotaInMB**|**unsignedInt**|Opcional. Especifica la cantidad máxima de almacenamiento del sistema de archivos que está disponible para los datos especificados.<br /><br /> El valor predeterminado es 0.|  
|**scheduledTransferLogLevelFilter**|**string**|Opcional. Especifica el nivel de gravedad mínimo para las entradas de registro que se van a transferir. El valor predeterminado es **Undefined**, que transfiere todos los registros. Otros valores posibles (en orden de mayor a menor información) son **Verbose**, **Information**, **Warning**, **Error** y **Critical**.|  
|**scheduledTransferPeriod**|**duration**|Opcional. Especifica el intervalo existente entre las transferencias programadas de datos, redondeado al minuto más cercano.<br /><br /> El valor predeterminado es PT0S.|  
|**sinks** |**string**| Agregado en 1.5. Opcional. Apunta a una ubicación de receptor para enviar datos de diagnóstico. Por ejemplo, Application Insights o Event Hubs. Tenga en cuenta que debe agregar la propiedad *resourceId* en el elemento *Metrics* si desea que los eventos cargados en Event Hubs tengan un identificador de recurso.|  

## <a name="dockersources"></a>DockerSources
 *Tree: Root - DiagnosticsConfiguration - PublicConfig - WadCFG - DiagnosticMonitorConfiguration - DockerSources*

 Se agregó en la versión 1.9.

|Nombre del elemento|Descripción|  
|------------------|-----------------|  
|**Stats**|Indica al sistema para recopilar estadísticas de los contenedores de Docker.|  

## <a name="sinksconfig-element"></a>Elemento SinksConfig  
 *Tree: Root - DiagnosticsConfiguration - PublicConfig - WadCFG - SinksConfig*

 Una lista de ubicaciones donde enviar datos de diagnóstico y la configuración asociada a estas ubicaciones.  

|Nombre del elemento|Descripción|  
|------------------|-----------------|  
|**Sink**|Consulte la descripción en cualquier parte de esta página.|  

## <a name="sink-element"></a>Elemento Sink
 *Tree: Root - DiagnosticsConfiguration - PublicConfig - WadCFG - SinksConfig - Sink*

 Agregado en la versión 1.5.  

 Define las ubicaciones donde se van a enviar datos de diagnóstico. Por ejemplo, el servicio Application Insights.  

|Atributo|Tipo|Descripción|  
|---------------|----------|-----------------|  
|**name**|string|Cadena que identifica el nombre de receptor.|  

|Elemento|Tipo|Descripción|  
|-------------|----------|-----------------|  
|**Application Insights**|string|Se usa solo al enviar datos a Application Insights. Contiene la clave de instrumentación para una cuenta activa de Application Insights a la que tiene acceso.|  
|**Channels**|string|Uno para cada filtrado adicional transmitido|  

## <a name="channels-element"></a>Elemento Channels  
 *Tree: Root - DiagnosticsConfiguration - PublicConfig - WadCFG - SinksConfig - Sink - Channels*

 Agregado en la versión 1.5.  

 Define los filtros para los flujos de datos de registro que se pasan a través de un receptor.  

|Elemento|Tipo|Descripción|  
|-------------|----------|-----------------|  
|**Channel**|string|Consulte la descripción en cualquier parte de esta página.|  

## <a name="channel-element"></a>Elemento Channel
 *Tree: Root - DiagnosticsConfiguration - PublicConfig - WadCFG - SinksConfig - Sink - Channels - Channel*

 Agregado en la versión 1.5.  

 Define las ubicaciones donde se van a enviar datos de diagnóstico. Por ejemplo, el servicio Application Insights.  

|Atributos|Tipo|Descripción|  
|----------------|----------|-----------------|  
|**logLevel**|**string**|Especifica el nivel de gravedad mínimo para las entradas de registro que se van a transferir. El valor predeterminado es **Undefined**, que transfiere todos los registros. Otros valores posibles (en orden de mayor a menor información) son **Verbose**, **Information**, **Warning**, **Error** y **Critical**.|  
|**name**|**string**|Un nombre único del canal al que se hace referencia|  


## <a name="privateconfig-element"></a>Elemento PrivateConfig
 *Tree: Root - DiagnosticsConfiguration - PrivateConfig*

 Agregado en la versión 1.3.  

 Opcional  

 Almacena los detalles privados de la cuenta de almacenamiento (nombre, clave y punto de conexión). Esta información se envía a la máquina virtual, pero no se puede recuperar de ella.  

|Elementos secundarios|Descripción|  
|--------------------|-----------------|  
|**StorageAccount**|La cuenta de almacenamiento que se va a usar. Los atributos siguientes son necesarios<br /><br /> - **name**: el nombre de la cuenta de almacenamiento.<br /><br /> - **key**: la clave de la cuenta de almacenamiento.<br /><br /> - **endpoint**: el punto de conexión para acceder a la cuenta de almacenamiento. <br /><br /> -**sasToken** (se agregó en la versión 1.8.1): puede especificar un token de SAS en lugar de una clave de cuenta de almacenamiento en la configuración privada. Si se proporciona, se omite la clave de cuenta de almacenamiento. <br />Requisitos del token de SAS: <br />- Solo admite el token de SAS de la cuenta. <br />Se requieren los tipos de servicio - *b* y *t*. <br /> Se requieren los permisos - *a*, *c*, *u* y *w*. <br /> Se requieren los tipos de recursos - *c* y *o*. <br /> - Solo admite únicamente el protocolo HTTPS. <br /> - La hora de inicio y de expiración debe ser válida.|  


## <a name="isenabled-element"></a>Elemento IsEnabled  
 *Tree: Root - DiagnosticsConfiguration - IsEnabled*

 booleano. Use `true` para habilitar los diagnósticos o `false` para deshabilitarlos.

## <a name="example-configuration"></a>Ejemplo de configuración
 A continuación, se muestra una configuración de ejemplo completa de Windows Diagnostics Extension en formato JSON y XML.

 
### <a name="json"></a>JSON

Los elementos *PublicConfig* y *PrivateConfig* están separados porque, en la mayoría de los casos de uso de JSON, se pasan utilizando variables diferentes. Entre estos casos, se incluyen las plantillas de Resource Manager, PowerShell y Visual Studio.

> [!NOTE]
> En la configuración pública, la definición del receptor de datos de Azure Monitor tiene dos propiedades: *resourceId* y *region*. Solo se necesitan para las máquinas virtuales clásicas y los servicios en la nube clásica. La propiedad *region* no debe usarse para otros recursos, la propiedad *resourceId* se usa en máquinas virtuales ARM para rellenar el campo resourceID en los registros cargados en Event Hubs.

```json
"PublicConfig" {
    "WadCfg": {
        "DiagnosticMonitorConfiguration": {
            "overallQuotaInMB": 10000,
            "DiagnosticInfrastructureLogs": {
                "scheduledTransferLogLevelFilter": "Error"
            },
            "PerformanceCounters": {
                "scheduledTransferPeriod": "PT1M",
                "sinks": "AzureMonitorSink",
                "PerformanceCounterConfiguration": [
                    {
                        "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
                        "sampleRate": "PT1M",
                        "unit": "percent"
                    }
                ]
            },
            "Directories": {
                "scheduledTransferPeriod": "PT5M",
                "IISLogs": {
                    "containerName": "iislogs"
                },
                "FailedRequestLogs": {
                    "containerName": "iisfailed"
                },
                "DataSources": [
                    {
                        "containerName": "mynewprocess",
                        "Absolute": {
                            "path": "C:\\MyNewProcess",
                            "expandEnvironment": false
                        }
                    },
                    {
                        "containerName": "badapp",
                        "Absolute": {
                            "path": "%SYSTEMDRIVE%\\BadApp",
                            "expandEnvironment": true
                        }
                    },
                    {
                        "containerName": "goodapp",
                        "LocalResource": {
                            "relativePath": "..\\PeanutButter",
                            "name": "Skippy"
                        }
                    }
                ]
            },
            "EtwProviders": {
                "sinks": "",
                "EtwEventSourceProviderConfiguration": [
                    {
                        "scheduledTransferPeriod": "PT5M",
                        "provider": "MyProviderClass",
                        "Event": [
                            {
                                "id": 0
                            },
                            {
                                "id": 1,
                                "eventDestination": "errorTable"
                            }
                        ],
                        "DefaultEvents": {
                        }
                    }
                ],
                "EtwManifestProviderConfiguration": [
                    {
                        "scheduledTransferPeriod": "PT2M",
                        "scheduledTransferLogLevelFilter": "Information",
                        "provider": "5974b00b-84c2-44bc-9e58-3a2451b4e3ad",
                        "Event": [
                            {
                                "id": 0
                            }
                        ],
                        "DefaultEvents": {
                        }
                    }
                ]
            },
            "WindowsEventLog": {
                "scheduledTransferPeriod": "PT5M",
                "DataSource": [
                    {
                        "name": "System!*[System[Provider[@Name='Microsoft Antimalware']]]"
                    },
                    {
                        "name": "System!*[System[Provider[@Name='NTFS'] and (EventID=55)]]"
                    },
                    {
                        "name": "System!*[System[Provider[@Name='disk'] and (EventID=7 or EventID=52 or EventID=55)]]"
                    }
                ]
            },
            "Logs": {
                "scheduledTransferPeriod": "PT1M",
                "scheduledTransferLogLevelFilter": "Verbose",
                "sinks": "ApplicationInsights.AppLogs"
            },
            "CrashDumps": {
                "directoryQuotaPercentage": 30,
                "dumpType": "Mini",
                "containerName": "wad-crashdumps",
                "CrashDumpConfiguration": [
                    {
                        "processName": "mynewprocess.exe"
                    },
                    {
                        "processName": "badapp.exe"
                    }
                ]
            }
        },
        "SinksConfig": {
            "Sink": [
                {
                    "name": "AzureMonitorSink",
                    "AzureMonitor":
                    {
                        "ResourceId": "{insert resourceId if a classic VM or cloud service, else property not needed}",
                        "Region": "{insert Azure region of resource if a classic VM or cloud service, else property not needed}"
                    }
                },
                {
                    "name": "ApplicationInsights",
                    "ApplicationInsights": "{Insert InstrumentationKey}",
                    "Channels": {
                        "Channel": [
                            {
                                "logLevel": "Error",
                                "name": "Errors"
                            },
                            {
                                "logLevel": "Verbose",
                                "name": "AppLogs"
                            }
                        ]
                    }
                },
                {
                    "name": "EventHub",
                    "EventHub": {
                        "Url": "https://myeventhub-ns.servicebus.windows.net/diageventhub",
                        "SharedAccessKeyName": "SendRule",
                        "usePublisherId": false
                    }
                },
                {
                    "name": "secondaryEventHub",
                    "EventHub": {
                        "Url": "https://myeventhub-ns.servicebus.windows.net/secondarydiageventhub",
                        "SharedAccessKeyName": "SendRule",
                        "usePublisherId": false
                    }
                },
                {
                    "name": "secondaryStorageAccount",
                    "StorageAccount": {
                        "name": "secondarydiagstorageaccount",
                        "endpoint": "https://core.windows.net"
                    }
                }
            ]
        }
    },
    "StorageAccount": "diagstorageaccount",
    "StorageType": "TableAndBlob"
}
```

> [!NOTE]
> En la configuración privada, la definición del receptor de datos de Azure Monitor tiene dos propiedades: *PrincipalId* y *Secret*. Solo se necesitan para las máquinas virtuales clásicas y los servicios en la nube clásica. Estas propiedades no deben usarse con otros recursos.


```json
"PrivateConfig" {
    "storageAccountName": "diagstorageaccount",
    "storageAccountKey": "{base64 encoded key}",
    "storageAccountEndPoint": "https://core.windows.net",
    "storageAccountSasToken": "{sas token}",
    "EventHub": {
        "Url": "https://myeventhub-ns.servicebus.windows.net/diageventhub",
        "SharedAccessKeyName": "SendRule",
        "SharedAccessKey": "{base64 encoded key}"
    },
    "AzureMonitorAccount": {
        "ServicePrincipalMeta": {
            "PrincipalId": "{Insert service principal client Id}",
            "Secret": "{Insert service principal client secret}"
        }
    },
    "SecondaryStorageAccounts": {
        "StorageAccount": [
            {
                "name": "secondarydiagstorageaccount",
                "key": "{base64 encoded key}",
                "endpoint": "https://core.windows.net",
                "sasToken": "{sas token}"
            }
        ]
    },
    "SecondaryEventHubs": {
        "EventHub": [
            {
                "Url": "https://myeventhub-ns.servicebus.windows.net/secondarydiageventhub",
                "SharedAccessKeyName": "SendRule",
                "SharedAccessKey": "{base64 encoded key}"
            }
        ]
    }
}

```

### <a name="xml"></a>XML

```xml  
<?xml version="1.0" encoding="utf-8"?>  
<DiagnosticsConfiguration  xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">   
  <PublicConfig>  
    <WadCfg>  
      <DiagnosticMonitorConfiguration overallQuotaInMB="10000">  

        <PerformanceCounters scheduledTransferPeriod="PT1M", sinks="AzureMonitorSink">  
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT1M" unit="percent" />  
        </PerformanceCounters>  

        <Directories scheduledTransferPeriod="PT5M">  
          <IISLogs containerName="iislogs" />  
          <FailedRequestLogs containerName="iisfailed" />  

          <DataSources>  
            <DirectoryConfiguration containerName="mynewprocess">  
              <Absolute path="C:\MyNewProcess" expandEnvironment="false" />  
            </DirectoryConfiguration>  
            <DirectoryConfiguration containerName="badapp">  
              <Absolute path="%SYSTEMDRIVE%\BadApp" expandEnvironment="true" />  
            </DirectoryConfiguration>  
            <DirectoryConfiguration containerName="goodapp">  
              <LocalResource name="Skippy" relativePath="..\PeanutButter"/>  
            </DirectoryConfiguration>  
          </DataSources>  

        </Directories>  

        <EtwProviders>  
          <EtwEventSourceProviderConfiguration   
                       provider="MyProviderClass"   
                       scheduledTransferPeriod="PT5M">  
            <Event id="0"/>  
            <Event id="1" eventDestination="errorTable"/>  
            <DefaultEvents />  
          </EtwEventSourceProviderConfiguration>  
          <EtwManifestProviderConfiguration provider="5974b00b-84c2-44bc-9e58-3a2451b4e3ad" scheduledTransferLogLevelFilter="Information" scheduledTransferPeriod="PT2M">  
            <Event id="0"/>  
            <DefaultEvents eventDestination="defaultTable"/>  
          </EtwManifestProviderConfiguration>  
        </EtwProviders>  

        <WindowsEventLog scheduledTransferPeriod="PT5M">  
          <DataSource name="System!*[System[Provider[@Name='Microsoft Antimalware']]]"/>  
          <DataSource name="System!*[System[Provider[@Name='NTFS'] and (EventID=55)]]" />  
          <DataSource name="System!*[System[Provider[@Name='disk'] and (EventID=7 or EventID=52 or EventID=55)]]" />  
        </WindowsEventLog>  

        <Logs  bufferQuotaInMB="1024"   
             scheduledTransferPeriod="PT1M"   
             scheduledTransferLogLevelFilter="Verbose"   
             sinks="ApplicationInsights.AppLogs"/>  <!-- sinks attribute added in 1.5 -->  

        <CrashDumps containerName="wad-crashdumps" directoryQuotaPercentage="30" dumpType="Mini">  
          <CrashDumpConfiguration processName="mynewprocess.exe" />  
          <CrashDumpConfiguration processName="badapp.exe"/>  
        </CrashDumps>  

        <DockerSources> <!-- Added in 1.9 -->
          <Stats enabled="true" sampleRate="PT1M" scheduledTransferPeriod="PT1M" />
        </DockerSources>

      </DiagnosticMonitorConfiguration>  

      <SinksConfig>   <!-- Added in 1.5 -->  
        <Sink name="AzureMonitorSink">
            <AzureMonitor> <!-- Added in 1.11 -->
                <resourceId>{insert resourceId}</ResourceId> <!-- Parameter only needed for classic VMs and Classic Cloud Services, exclude VMSS and Resource Manager VMs-->
                <Region>{insert Azure region of resource}</Region> <!-- Parameter only needed for classic VMs and Classic Cloud Services, exclude VMSS and Resource Manager VMs -->
            </AzureMonitor>
        </Sink>
        <Sink name="ApplicationInsights">   
          <ApplicationInsights>{Insert InstrumentationKey}</ApplicationInsights>   
          <Channels>   
            <Channel logLevel="Error" name="Errors"  />   
            <Channel logLevel="Verbose" name="AppLogs"  />   
          </Channels>   
        </Sink>   
        <Sink name="EventHub"> <!-- Added in 1.7 -->
          <EventHub Url="https://myeventhub-ns.servicebus.windows.net/diageventhub" SharedAccessKeyName="SendRule" usePublisherId="false" />
        </Sink>
        <Sink name="secondaryEventHub"> <!-- Added in 1.7 -->
          <EventHub Url="https://myeventhub-ns.servicebus.windows.net/secondarydiageventhub" SharedAccessKeyName="SendRule" usePublisherId="false" />
        </Sink>
        <Sink name="secondaryStorageAccount"> <!-- Added in 1.7 -->
          <StorageAccount name="secondarydiagstorageaccount" endpoint="https://core.windows.net" />
        </Sink>
   </SinksConfig>

  </WadCfg>  

  <StorageAccount>diagstorageaccount</StorageAccount>
  <StorageType>TableAndBlob</StorageType> <!-- Added in 1.8 -->  
  </PublicConfig>  

  <PrivateConfig>  <!-- Added in 1.3 -->  
    <StorageAccount name="" key="" endpoint="" sasToken="{sas token}"  />  <!-- sasToken in Private config added in 1.8.1 -->  
    <EventHub Url="https://myeventhub-ns.servicebus.windows.net/diageventhub" SharedAccessKeyName="SendRule" SharedAccessKey="{base64 encoded key}" />

    <AzureMonitorAccount>
        <ServicePrincipalMeta> <!-- Added in 1.11; only needed for classic VMs and Classic cloud services -->
            <PrincipalId>{Insert service principal clientId}</PrincipalId>
            <Secret>{Insert service principal client secret}</Secret>
        </ServicePrincipalMeta>
    </AzureMonitorAccount>

    <SecondaryStorageAccounts>
       <StorageAccount name="secondarydiagstorageaccount" key="{base64 encoded key}" endpoint="https://core.windows.net" sasToken="{sas token}" />
    </SecondaryStorageAccounts>

    <SecondaryEventHubs>
       <EventHub Url="https://myeventhub-ns.servicebus.windows.net/secondarydiageventhub" SharedAccessKeyName="SendRule" SharedAccessKey="{base64 encoded key}" />
    </SecondaryEventHubs>

  </PrivateConfig>  
  <IsEnabled>true</IsEnabled>  
</DiagnosticsConfiguration>  

```  
> [!NOTE]
> La configuración pública de la definición de receptor de Azure Monitor tiene dos propiedades: resourceId y region. Solo se necesitan para las máquinas virtuales clásicas y los servicios en la nube clásica. Estas propiedades no deben usarse para las máquinas virtuales de Resource Manager ni para Virtual Machine Scale Sets.
> También hay un elemento de configuración privada adicional para el receptor de Azure Monitor, que pasa un identificador de entidad de seguridad y el secreto. Esto solo es necesario para las máquinas virtuales clásicas y los servicios en la nube clásica. Para las máquinas virtuales de Resource Manager y VMSS, la definición de Azure Monitor se puede excluir del elemento de configuración privada.
>
