---
title: Reglas de recopilación de datos en Azure Monitor (versión preliminar)
description: Información general sobre las reglas de recopilación de datos (DCR) en Azure Monitor incluido su contenido y estructura, y cómo puede crearlas y trabajar con ellas.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/19/2020
ms.openlocfilehash: 177b79e0a33f4d43d07da9d0dea26df40e2ef11e
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88723867"
---
# <a name="data-collection-rules-in-azure-monitor-preview"></a>Reglas de recopilación de datos en Azure Monitor (versión preliminar)
Las reglas de recopilación de datos (DCR) definen los datos que entran en Azure Monitor y especifican dónde se deben enviar los datos o almacenarlos. En este artículo se proporciona información general sobre las reglas de recopilación de datos, incluido su contenido y su estructura, y cómo puede crearlas y trabajar con ellas.

## <a name="input-sources"></a>Orígenes de entrada
Las reglas de recopilación de datos admiten actualmente los siguientes orígenes de entrada:

- Máquina virtuales de Azure con el agente de Azure Monitor. Consulte [Configuración de la recopilación de datos para el agente de Azure Monitor (versión preliminar)](data-collection-rule-azure-monitor-agent.md).



## <a name="components-of-a-data-collection-rule"></a>Componentes de una regla de recopilación de datos
Una regla de recopilación de datos incluye los siguientes componentes.

| Componente | Descripción |
|:---|:---|
| Orígenes de datos | Origen único de los datos de supervisión con su propio formato y método para exponer los datos. Entre los ejemplos de origen de datos se incluyen el registro de eventos de Windows, los contadores de rendimiento y syslog. Cada origen de datos coincide con un tipo de origen de datos determinado tal y como se describe a continuación. |
| Secuencias | Identificador único que describe un conjunto de orígenes de datos que se transformarán y se esquematizarán como un tipo. Cada origen de datos requiere una o varias secuencias; varios orígenes de datos pueden utilizar una secuencia. Todos los orígenes de datos de una secuencia comparten un esquema común. Por ejemplo, puede usar varias secuencias si desea enviar un origen de datos determinado a varias tablas en el mismo área de trabajo de Log Analytics. |
| Destinations | Conjunto de destinos donde deben enviarse los datos. Algunos ejemplos son el área de trabajo de Log Analytics, Métricas de Azure Monitor y Event Hubs de Azure. | 
| Flujos de datos | Definición de las secuencias que se deben enviar a los destinos. | 

En el diagrama siguiente se muestran los componentes de una regla de recopilación de datos y su relación

[![Diagrama de DCR](media/data-collection-rule/data-collection-rule-components.png)](media/data-collection-rule/data-collection-rule-components.png#lightbox)

### <a name="data-source-types"></a>Tipos de origen de datos
Cada origen de datos tiene un tipo de origen de datos. Cada tipo define un conjunto único de propiedades que se deben especificar para cada origen de datos. Los tipos de orígenes de datos disponibles actualmente se muestran en la tabla siguiente.

| Tipo de origen de datos | Descripción | 
|:---|:---|
| extensión | Origen de datos basado en la extensión de máquina virtual |
| performanceCounters | Contadores de rendimiento para Windows y Linux |
| syslog | Eventos de Syslog en Linux |
| windowsEventLogs | Registro de eventos de Windows |


## <a name="limits"></a>Límites
En la tabla siguiente se enumeran los límites que se aplican actualmente a cada regla de recopilación de datos.

| Límite | Value |
|:---|:---|
| Número máximo de orígenes de datos | 10 |
| Número máximo de especificadores de contador en el contador de rendimiento | 100 |
| Número máximo de nombres de utilidades en Syslog | 20 |
| Número máximo de consultas XPath en el registro de eventos | 100 |
| Número máximo de flujos de datos | 10 |
| Número máximo de flujos de datos | 10 |
| Número máximo de extensiones | 10 |
| Tamaño máximo de la configuración de la extensión | 32 Kb |
| Número máximo de áreas de trabajo de Log Analytics | 10 |


## <a name="create-a-dcr"></a>Creación de una regla de recopilación de datos
Actualmente hay dos métodos disponibles para crear una regla de recopilación de datos:

- [Use Azure Portal](data-collection-rule-azure-monitor-agent.md) para crear una regla de recopilación de datos y asociarla a una o varias máquinas virtuales.
- Editar directamente la regla de recopilación de datos en el archivo JSON y [enviarla mediante la API de REST](https://docs.microsoft.com/rest/api/monitor/datacollectionrules).

## <a name="sample-data-collection-rule"></a>Eliminación de una regla de recopilación de datos
La siguiente regla de recopilación de datos de ejemplo es para las máquinas virtuales con el agente de administración de Azure y tiene los detalles siguientes:

- Datos de rendimiento
  - Recopila los contadores de procesador, memoria, disco lógico y disco físico específicos cada 15 segundos, y se carga cada minuto.
  - Recopila los contadores de proceso específicos cada 30 segundos y los carga cada 5 minutos.
- Eventos de Windows
  - Recopila eventos de seguridad de Windows y los carga cada minuto.
  - Recopila eventos del sistema y de aplicación de Windows, y los carga cada 5 minutos.
- syslog
  - Recopila eventos de depuración, críticos y de emergencia desde la utilidad cron.
  - Recopila eventos de alerta, críticos y de emergencia desde la utilidad syslog.
- Destinations
  - Envía todos los datos a un área de trabajo de Log Analytics denominada centralWorkspace.

```json
{
    "location": "eastus",
    "properties": {
      "dataSources": {
        "performanceCounters": [
          {
            "name": "cloudTeamCoreCounters",
            "streams": [
              "Microsoft-Perf"
            ],
            "scheduledTransferPeriod": "PT1M",
            "samplingFrequencyInSeconds": 15,
            "counterSpecifiers": [
              "\\Processor(_Total)\\% Processor Time",
              "\\Memory\\Committed Bytes",
              "\\LogicalDisk(_Total)\\Free Megabytes",
              "\\PhysicalDisk(_Total)\\Avg. Disk Queue Length"
            ]
          },
          {
            "name": "appTeamExtraCounters",
            "streams": [
              "Microsoft-Perf"
            ],
            "scheduledTransferPeriod": "PT5M",
            "samplingFrequencyInSeconds": 30,
            "counterSpecifiers": [
              "\\Process(_Total)\\Thread Count"
            ]
          }
        ],
        "windowsEventLogs": [
          {
            "name": "cloudSecurityTeamEvents",
            "streams": [
              "Microsoft-WindowsEvent"
            ],
            "scheduledTransferPeriod": "PT1M",
            "xPathQueries": [
              "Security!*"
            ]
          },
          {
            "name": "appTeam1AppEvents",
            "streams": [
              "Microsoft-WindowsEvent"
            ],
            "scheduledTransferPeriod": "PT5M",
            "xPathQueries": [
              "System!*[System[(Level = 1 or Level = 2 or Level = 3)]]",
              "Application!*[System[(Level = 1 or Level = 2 or Level = 3)]]"
            ]
          }
        ],
        "syslog": [
          {
            "name": "cronSyslog",
            "streams": [
              "Microsoft-Syslog"
            ],
            "facilityNames": [
              "cron"
            ],
            "logLevels": [
              "Debug",
              "Critical",
              "Emergency"
            ]
          },
          {
            "name": "syslogBase",
            "streams": [
              "Microsoft-Syslog"
            ],
            "facilityNames": [
              "syslog"
            ],
            "logLevels": [
              "Alert",
              "Critical",
              "Emergency"
            ]
          }
        ]
      },
      "destinations": {
        "logAnalytics": [
          {
            "workspaceResourceId": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.OperationalInsights/workspaces/my-workspace",
            "name": "centralWorkspace"
          }
        ]
      },
      "dataFlows": [
        {
          "streams": [
            "Microsoft-Perf",
            "Microsoft-Syslog",
            "Microsoft-WindowsEvent"
          ],
          "destinations": [
            "centralWorkspace"
          ]
        }
      ]
    }
  }
```


## <a name="next-steps"></a>Pasos siguientes

- [Cree una regla de recopilación de datos](data-collection-rule-azure-monitor-agent.md) y una asociación a ella desde una máquina virtual mediante el agente de Azure Monitor.
