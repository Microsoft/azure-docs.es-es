---
title: 'Creación de una instancia de Connection Monitor (versión preliminar): ARMClient'
titleSuffix: Azure Network Watcher
description: Aprenda a crear una instancia de Connection Monitor (versión preliminar) mediante ARMClient.
services: network-watcher
documentationcenter: na
author: vinigam
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/30/2020
ms.author: vinigam
ms.openlocfilehash: 7d35799cd73ff4d065cb58189f2325dc4dac6840
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87567834"
---
# <a name="create-a-connection-monitor-preview-using-the-armclient"></a>Creación de una instancia de Connection Monitor (versión preliminar) mediante ARMClient

Aprenda a crear una instancia de Connection Monitor (versión preliminar) para supervisar la comunicación entre los recursos mediante ARMClient. Admite implementaciones híbridas y en la nube de Azure.

## <a name="before-you-begin"></a>Antes de empezar 

En los monitores de conexión que crea en el Monitor de conexión (versión preliminar), puede agregar máquinas locales y máquinas virtuales de Azure como orígenes. Estos monitores de conexión también pueden supervisar la conectividad con los puntos de conexión. Los puntos de conexión pueden estar en Azure o en cualquier otra dirección URL o IP.

El Monitor de conexión (versión preliminar) incluye estas entidades:

* **Recurso de monitor de conexión**: un recurso de Azure específico de la región. Todas las entidades siguientes son propiedades de un recurso de monitor de conexión.
* **Punto de conexión**: un origen o destino que participa en las comprobaciones de conectividad. Algunos ejemplos de puntos de conexión son las máquinas virtuales de Azure, los agentes locales, las direcciones URL y las direcciones IP.
* **Configuración de prueba**: la configuración de una prueba específica para el protocolo. En función del protocolo elegido, puede definir el puerto, los umbrales, la frecuencia de las pruebas y otros parámetros.
* **Grupo de prueba**: el grupo que contiene puntos de conexión de origen, puntos de conexión de destino y configuraciones de prueba. Un monitor de conexión puede contener más de un grupo de prueba.
* **Prueba**: la combinación de un punto de conexión de origen, un punto de conexión de destino y una configuración de prueba. Una prueba es el nivel más granular en el que están disponibles los datos de supervisión. Los datos de supervisión incluyen el porcentaje de comprobaciones con errores y el tiempo de ida y vuelta (RTT).

    ![Diagrama que muestra un monitor de conexión y donde se define la relación entre los grupos de prueba y las pruebas](./media/connection-monitor-2-preview/cm-tg-2.png)

## <a name="steps-to-create-with-sample-arm-template"></a>Pasos para crear una plantilla de ARM de ejemplo

Use el código siguiente para crear un monitor de conexión mediante ARMClient.

```armclient
$connectionMonitorName = "sampleConnectionMonitor"

$ARM = "https://management.azure.com"

$SUB = "subscriptions/<subscription id 1>;"

$NW = "resourceGroups/NetworkWatcherRG/providers/Microsoft.Network/networkWatchers/NetworkWatcher\_<region>"

$body =

"{

location: '<region>',

properties: {

endpoints: [{

name: 'workspace',

resourceId: '/subscriptions/<subscription id>/resourcegroups/<resource group>/providers/Microsoft.OperationalInsights/workspaces/sampleWorkspace',

filter: {

 items: [{

type: 'AgentAddress',

address: '<FQDN of your on-premises agent>'

}]

}

          },

 {

name: 'vm1',

resourceId: '/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/virtualMachines/<vm-name>'

},

 {

name: 'vm2',

resourceId: '/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/virtualMachines/<vm-name>'

   },

{

name: 'azure portal'

address: '<URL>'

   },

 {

    name: 'ip',

     address: '<IP>'

 }

  ],

  testGroups: [{

    name: 'Connectivity to Azure Portal and Public IP',

    testConfigurations: ['http', 'https', 'tcpEnabled', 'icmpEnabled'],

    sources: ['vm1', 'workspace'],

    destinations: ['azure portal', 'ip']

   },

{

    name: 'Connectivty from Azure VM 1 to Azure VM 2',

   // Choose your protocol
   
    testConfigurations: ['http', 'https', 'tcpDisabled', 'icmpDisabled'],

    sources: ['vm1'],

    destinations: ['vm2'],

    disable: true

   }

  ],

  testConfigurations: [{

    name: 'http',

    testFrequencySec: <frequency>,

    protocol: 'HTTP',

    successThreshold: {

     checksFailedPercent: <threshold for checks failed %>,

     roundTripTimeMs: <threshold for RTT>

    }

   }, {

    name: 'https',

    testFrequencySec: <frequency>,

    protocol: 'HTTP',

    httpConfiguration: {

     preferHTTPS: true

    },

    successThreshold: {

     checksFailedPercent: <choose your checks failed threshold>,

     roundTripTimeMs: <choose your RTT threshold>

    }

   }, {

    name: 'tcpEnabled',

    testFrequencySec: <frequency>,

    protocol: 'TCP',

    tcpConfiguration: {

     port: 80

    },

    successThreshold: {

     checksFailedPercent: <choose your checks failed threshold>,

     roundTripTimeMs: <choose your RTT threshold>

    }

   }, {

    name: 'icmpEnabled',

    testFrequencySec: <frequency>,

    protocol: 'ICMP',

    successThreshold: {

     checksFailedPercent: <choose your checks failed threshold>,

     roundTripTimeMs: <choose your RTT threshold>

    }

   }, {

    name: 'icmpDisabled',

    testFrequencySec: <frequency>,

    protocol: 'ICMP',

    icmpConfiguration: {

     disableTraceRoute: true

    },

    successThreshold: {

     checksFailedPercent: <choose your checks failed threshold>,

     roundTripTimeMs: <choose your RTT threshold>

    }

   }, {

    name: 'tcpDisabled',

    testFrequencySec: <frequency>,

    protocol: 'TCP',

    tcpConfiguration: {

     port: 80,

     disableTraceRoute: true

    },

    successThreshold: {

     checksFailedPercent: <choose your checks failed threshold>,

     roundTripTimeMs: <choose your RTT threshold>

    }

   }

  ]

 }

} "
```

Este es el comando de implementación:
```
armclient PUT $ARM/$SUB/$NW/connectionMonitors/$connectionMonitorName/?api-version=2019-07-01 $body -verbose
```

## <a name="description-of-properties"></a>Descripción de las propiedades

* connectionMonitorName: nombre del recurso de Connection Monitor.

* SUB: identificador de la suscripción donde quiere crear el monitor de conexión.

* NW: identificador de recurso de Network Watcher en el que se creará el monitor de conexión. 

* location: región en la que se creará el monitor de conexión.

* Puntos de conexión
    * name: nombre único de cada punto de conexión.
    * resourceId: en el caso de los puntos de conexión de Azure, el identificador de recurso hace referencia al identificador de recurso de Azure Resource Manager (ARM) de las máquinas virtuales. En el caso de los puntos de conexión que no son de Azure, el identificador de recurso hace referencia al identificador de recurso de ARM del área de trabajo de Log Analytics vinculada a agentes que no son de Azure.
    * address: solo se aplica cuando no se especifica ningún identificador de recurso o si el identificador de recurso es el área de trabajo de Log Analytics. Si se usa con el identificador de recurso de Log Analytics, se refiere al FQDN del agente que se puede usar para la supervisión. Si se usa sin el identificador de recurso, puede ser la dirección URL o IP de cualquier punto de conexión público.
    * filter: en el caso de los puntos de conexión que no son de Azure, use este valor para seleccionar los agentes del área de trabajo de Log Analytics que se usarán para supervisar los recursos del monitor de conexión. Si no se establecen filtros, todos los agentes que pertenezcan al área de trabajo Log Analytics se pueden usar en la supervisión.
        * type: establezca el tipo como "dirección del agente".
        * address: establezca la dirección como el FQDN del agente local.

* Grupos de prueba
    * name: asigne un nombre al grupo de prueba.
    * testConfigurations: configuraciones de prueba basadas en los puntos de conexión de origen que se conectan a los puntos de conexión de destino.
    * sources: elija entre los puntos de conexión creados anteriormente. Los puntos de conexión de origen basados en Azure deben tener instalada la extensión de Azure Network Watcher y los que no se basan en Azure deben tener instalado el agente de Azure Log Analytics. Para instalar un agente para el origen, consulte [Instalación de los agentes de supervisión](https://docs.microsoft.com/azure/network-watcher/connection-monitor-preview#install-monitoring-agents).
    * destinations: elija entre los puntos de conexión creados anteriormente. Se puede supervisar la conectividad a las máquinas virtuales de Azure o a cualquier punto de conexión (una IP pública, una dirección URL o un FQDN) si se especifican como destinos. En un solo grupo de prueba, puede agregar máquinas virtuales de Azure, direcciones URL de Office 365, direcciones URL de Dynamics 365 y puntos de conexión personalizados.
    * disable: use este campo para deshabilitar la supervisión de todos los orígenes y destinos que especifica el grupo de prueba.

* Configuraciones de prueba
    * name: asigne un nombre a la configuración de prueba.
    * testFrequencySec: especifique la frecuencia con la que los orígenes harán ping a los destinos en el protocolo y el puerto especificados. Puede elegir 30 segundos, 1 minuto, 5 minutos, 15 minutos o 30 minutos. Los orígenes probarán la conectividad a los destinos según el valor que elija. Por ejemplo, si selecciona 30 segundos, los orígenes comprobarán la conectividad con el destino al menos una vez en un período de 30 segundos.
    * protocol: puede elegir TCP, ICMP, HTTP o HTTPS. Dependiendo del protocolo, puede realizar algunas configuraciones específicas del protocolo.
        * preferHTTPS: especifique si se usará HTTPS sobre HTTP.
        * port: especifique el puerto de destino de su elección.
        * disableTraceRoute: se aplica a los grupos de prueba cuyo protocolo es TCP o ICMP. Impide que los orígenes detecten la topología y el valor de RTT de salto a salto.
    * successThreshold: puede establecer los umbrales de estos parámetros de red:
        * checksFailedPercent: establezca el porcentaje de comprobaciones que pueden presentar errores cuando los orígenes comprueban la conectividad a los destinos mediante los criterios especificados. En el caso del protocolo TCP o ICMP, el porcentaje de comprobaciones con error puede ser igual al porcentaje de pérdida de paquetes. En el caso del protocolo HTTP, este campo representa el porcentaje de solicitudes HTTP que no recibieron respuesta.
        * roundTripTimeMs: establezca el RTT en milisegundos para indicar cuánto tiempo pueden tardar los orígenes en conectarse al destino a través de la configuración de prueba.

## <a name="scale-limits"></a>Límites de escalado

Los monitores de conexión tienen los límites de escala siguientes:

* Máximo de monitores de conexión por suscripción por región: 100
* Máximo de grupos de prueba por monitor de conexión: 20
* Máximo de orígenes y destinos por monitor de conexión: 100
* Máximo de configuraciones de prueba por monitor de conexión: 20 a través de ARMClient

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [analizar los datos de supervisión y a definir alertas](https://docs.microsoft.com/azure/network-watcher/connection-monitor-preview#analyze-monitoring-data-and-set-alerts).
* Aprenda a [diagnosticar los problemas de la red](https://docs.microsoft.com/azure/network-watcher/connection-monitor-preview#diagnose-issues-in-your-network).
