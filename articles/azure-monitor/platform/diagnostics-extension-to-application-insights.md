---
title: Envío de datos de Azure Diagnostics a Application Insights
description: Actualice la configuración pública de Diagnósticos de Azure para enviar datos a Application Insights.
ms.subservice: diagnostic-extension
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/19/2016
ms.openlocfilehash: 5af0eb20f9766369caa7351719b63b213c394e5d
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87305812"
---
# <a name="send-cloud-service-virtual-machine-or-service-fabric-diagnostic-data-to-application-insights"></a>Envío de datos de diagnóstico de Cloud Services, Virtual Machines o Service Fabric a Application Insights
Cloud Services, Virtual Machines, los conjuntos de escalado de máquinas virtuales y Service Fabric usan la extensión Azure Diagnostics para recopilar datos.  Esta extensión envía los datos a las tablas de Azure Storage.  Sin embargo, también puede canalizar todos los datos o un subconjunto de ellos a otras ubicaciones mediante la versión 1.5 o una posterior de la extensión Azure Diagnostics.

En este artículo se describe cómo enviar datos de la extensión Azure Diagnostics a Application Insights.

## <a name="diagnostics-configuration-explained"></a>Explicación de la configuración de Diagnostics
La versión 1.5 de la extensión Azure Diagnostics introdujo los receptores, que son más ubicaciones donde puede enviar datos de diagnóstico.

Ejemplo de configuración de un receptor para Application Insights:

```XML
<SinksConfig>
    <Sink name="ApplicationInsights">
      <ApplicationInsights>{Insert InstrumentationKey}</ApplicationInsights>
      <Channels>
        <Channel logLevel="Error" name="MyTopDiagData"  />
        <Channel logLevel="Verbose" name="MyLogData"  />
      </Channels>
    </Sink>
</SinksConfig>
```
```JSON
"SinksConfig": {
    "Sink": [
        {
            "name": "ApplicationInsights",
            "ApplicationInsights": "{Insert InstrumentationKey}",
            "Channels": {
                "Channel": [
                    {
                        "logLevel": "Error",
                        "name": "MyTopDiagData"
                    },
                    {
                        "logLevel": "Error",
                        "name": "MyLogData"
                    }
                ]
            }
        }
    ]
}
```
- El atributo *name* del **receptor** es un valor de cadena que identifica de forma única el receptor.

- El elemento **ApplicationInsights** especifica la clave de instrumentación del recurso de Application Insights donde se enviarán los datos de Azure Diagnostics.
    - Si no tiene un recurso existente de Application Insights, consulte [Creación de recursos en Application Insights](../app/create-new-resource.md) para más información sobre cómo crear un recurso y obtener la clave de instrumentación.
    - Si está desarrollando un servicio en la nube con Azure SDK 2.8 y versiones posteriores, se rellena automáticamente esta clave de instrumentación. El valor se basa en la configuración de servicio **APPINSIGHTS_INSTRUMENTATIONKEY** al empaquetar el proyecto de Cloud Services. Consulte [Usar Application Insights con Cloud Services](../app/cloudservices.md).

- El elemento **Channels** contiene uno o varios elementos **Channel**.
    - El atributo *name* identifica de forma única el atributo a ese canal.
    - Con el atributo *loglevel* se puede especificar el nivel de registro que permitirá el canal. Estos son los niveles de registro disponibles, en orden de más a menos información:
        - Verbose
        - Information
        - Advertencia
        - Error
        - Crítico

El canal actúa como filtro y permite seleccionar los niveles de registro específicos que enviar al receptor. Por ejemplo, podría recopilar registros detallados y enviarlos al almacenamiento, pero solo se envían los errores al receptor.

En el gráfico siguiente se muestra esta relación.

![Configuración pública de diagnósticos](media/diagnostics-extension-to-application-insights/AzDiag_Channels_App_Insights.png)

El gráfico siguiente resume los valores de configuración y cómo funcionan. Puede incluir varios receptores en la configuración en distintos niveles de la jerarquía. El receptor del nivel superior de la jerarquía actúa como una configuración global y el especificado en los elementos individuales como una invalidación de esa configuración global.

![Receptores de diagnósticos: configuración con Application Insights](media/diagnostics-extension-to-application-insights/Azure_Diagnostics_Sinks.png)

## <a name="complete-sink-configuration-example"></a>Ejemplo completo de configuración de receptor
Este es un ejemplo completo del archivo de configuración público que
1. envía todos los errores a Application Insights (se especifica en el nodo **DiagnosticMonitorConfiguration**).
2. Además, envía los registros de aplicación detallados (se especifica en el nodo **Logs**).

```XML
<WadCfg>
  <DiagnosticMonitorConfiguration overallQuotaInMB="4096"
       sinks="ApplicationInsights.MyTopDiagData"> <!-- All info below sent to this channel -->
    <DiagnosticInfrastructureLogs />
    <PerformanceCounters>
      <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT3M" />
      <PerformanceCounterConfiguration counterSpecifier="\Memory\Available MBytes" sampleRate="PT3M" />
    </PerformanceCounters>
    <WindowsEventLog scheduledTransferPeriod="PT1M">
      <DataSource name="Application!*" />
    </WindowsEventLog>
    <Logs scheduledTransferPeriod="PT1M" scheduledTransferLogLevelFilter="Verbose"
            sinks="ApplicationInsights.MyLogData"/> <!-- This specific info sent to this channel -->
  </DiagnosticMonitorConfiguration>

<SinksConfig>
    <Sink name="ApplicationInsights">
      <ApplicationInsights>{Insert InstrumentationKey}</ApplicationInsights>
      <Channels>
        <Channel logLevel="Error" name="MyTopDiagData"  />
        <Channel logLevel="Verbose" name="MyLogData"  />
      </Channels>
    </Sink>
  </SinksConfig>
</WadCfg>
```
```JSON
"WadCfg": {
    "DiagnosticMonitorConfiguration": {
        "overallQuotaInMB": 4096,
        "sinks": "ApplicationInsights.MyTopDiagData", "_comment": "All info below sent to this channel",
        "DiagnosticInfrastructureLogs": {
        },
        "PerformanceCounters": {
            "PerformanceCounterConfiguration": [
                {
                    "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
                    "sampleRate": "PT3M"
                },
                {
                    "counterSpecifier": "\\Memory\\Available MBytes",
                    "sampleRate": "PT3M"
                }
            ]
        },
        "WindowsEventLog": {
            "scheduledTransferPeriod": "PT1M",
            "DataSource": [
                {
                    "name": "Application!*"
                }
            ]
        },
        "Logs": {
            "scheduledTransferPeriod": "PT1M",
            "scheduledTransferLogLevelFilter": "Verbose",
            "sinks": "ApplicationInsights.MyLogData", "_comment": "This specific info sent to this channel"
        }
    },
    "SinksConfig": {
        "Sink": [
            {
                "name": "ApplicationInsights",
                "ApplicationInsights": "{Insert InstrumentationKey}",
                "Channels": {
                    "Channel": [
                        {
                            "logLevel": "Error",
                            "name": "MyTopDiagData"
                        },
                        {
                            "logLevel": "Verbose",
                            "name": "MyLogData"
                        }
                    ]
                }
            }
        ]
    }
}
```
En la configuración anterior, las líneas siguientes tienen estos significados:

### <a name="send-all-the-data-that-is-being-collected-by-azure-diagnostics"></a>Envía todos los datos que recopila Azure Diagnostics.

```XML
<DiagnosticMonitorConfiguration overallQuotaInMB="4096" sinks="ApplicationInsights">
```
```JSON
"DiagnosticMonitorConfiguration": {
    "overallQuotaInMB": 4096,
    "sinks": "ApplicationInsights",
}
```

### <a name="send-only-error-logs-to-the-application-insights-sink"></a>Envía solo los registros de errores al receptor de Application Insights.

```XML
<DiagnosticMonitorConfiguration overallQuotaInMB="4096" sinks="ApplicationInsights.MyTopDiagdata">
```
```JSON
"DiagnosticMonitorConfiguration": {
    "overallQuotaInMB": 4096,
    "sinks": "ApplicationInsights.MyTopDiagData",
}
```

### <a name="send-verbose-application-logs-to-application-insights"></a>Envía los registros de aplicación detallados a Application Insights.

```XML
<Logs scheduledTransferPeriod="PT1M" scheduledTransferLogLevelFilter="Verbose" sinks="ApplicationInsights.MyLogData"/>
```
```JSON
"DiagnosticMonitorConfiguration": {
    "overallQuotaInMB": 4096,
    "sinks": "ApplicationInsights.MyLogData",
}
```

## <a name="limitations"></a>Limitaciones

- **Channels solo registra el tipo de registro y no los contadores de rendimiento.** Si especifica un canal con un elemento de contador de rendimiento, se pasará por alto.
- **El nivel de registro de un canal no puede superar el nivel de registro de lo que va a recopilar con Azure Diagnostics.** Por ejemplo, no puede recopilar errores de registro de aplicación del elemento Logs y tratar de enviar registros detallados al receptor de Application Insights. El atributo *scheduledTransferLogLevelFilter* debe recopilar siempre un número de registros igual o superior a los registros que intenta enviar a un receptor.
- **No puede enviar datos de blob recopilados mediante la extensión Azure Diagnostics a Application Insights.** Por ejemplo, nada de lo especificado en el nodo *Directories*. En el caso de volcados de memoria, el volcado real se sigue enviando al almacenamiento de blobs y solo se transmitirá a Application Insights una notificación de que el volcado de memoria se ha generado.

## <a name="next-steps"></a>Pasos siguientes
* Obtenga información sobre cómo [ver información de Azure Diagnostics](../app/cloudservices.md) en Application Insights.
* Use [PowerShell](../../cloud-services/cloud-services-diagnostics-powershell.md) para habilitar la extensión de diagnósticos de Azure en su aplicación.
* Use [Visual Studio](/visualstudio/azure/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines) para habilitar la extensión de Diagnósticos de Azure en su aplicación

