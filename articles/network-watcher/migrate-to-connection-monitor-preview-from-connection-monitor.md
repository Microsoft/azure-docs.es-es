---
title: Migración a Connection Monitor (versión preliminar) de Connection Monitor
titleSuffix: Azure Network Watcher
description: Aprenda a migrar a Connection Monitor (versión preliminar) de Connection Monitor.
services: network-watcher
documentationcenter: na
author: vinigam
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/20/2020
ms.author: vinigam
ms.openlocfilehash: ddf6e326df876229d32ef15983f76879836f1fca
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88701348"
---
# <a name="migrate-to-connection-monitor-preview-from-connection-monitor"></a>Migración a Connection Monitor (versión preliminar) de Connection Monitor

Puede migrar los monitores de conexión existentes al nuevo y mejorado Connection Monitor (versión preliminar) con un solo clic y sin tiempo de inactividad. Para más información sobre las ventajas, puede consultar [Connection Monitor (versión preliminar)](https://docs.microsoft.com/azure/network-watcher/connection-monitor-preview).

## <a name="key-points-to-note"></a>Puntos clave a tener en cuenta

* Los agentes y la configuración de firewall funcionarán tal cual (no es necesario modificar nada). 
* Los monitores de conexión existentes se asignarán a Connection Monitor (versión preliminar) -> Grupo de pruebas -> Formato de prueba. Los usuarios pueden hacer clic en *Editar* para ver y modificar las propiedades del Connection Monitor nuevo y descargar la plantilla para hacer cambios en Connection Monitor y enviarlo a través de Azure Resource Manager. 
* Las máquinas virtuales de Azure con la extensión Network Watcher envían datos tanto al área de trabajo como a las métricas. Connection Monitors hará que los datos estén disponibles a través de las métricas nuevas (ChecksFailedPercent (versión preliminar) y RoundTripTimeMs (versión preliminar)) en lugar de las anterior métricas de detención (ProbesFailedPercent y AverageRoundtripMs). 
* Supervisión de datos
    * Alertas: se migrarán a métricas nuevas como parte de la migración.
    * Paneles e integraciones: tendrá que editar manualmente el conjunto de métricas. 
    
## <a name="prerequisites"></a>Requisitos previos

Si usa el área de trabajo actual, asegúrese de que Network Watcher esté habilitado en la suscripción y la región del área de trabajo de Log Analytics. 

## <a name="steps-to-migrate-from-connection-monitor-to-connection-monitor-preview"></a>Pasos para migrar de Connection Monitor a Connection Monitor (versión preliminar)

1. Haga clic en "Connection Monitor", vaya a "Migrar instancias de Connection Monitor" para migrar monitores de conexión de una solución antigua a una más reciente.

    ![Captura de pantalla que muestra la migración de monitores de conexión a Connection Monitor (versión preliminar)](./media/connection-monitor-2-preview/migrate-cm-to-cm-preview.png)
    
1. Seleccione la suscripción y los monitores de conexión y haga clic en "Migrar selección". Con un solo clic, migre los monitores de conexión existentes a Connection Monitor (versión preliminar). 
1. Puede personalizar las propiedades del monitor de conexión, cambiar el área de trabajo predeterminada, descargar la plantilla y comprobar el estado de la migración. 
1. Una vez que se inicie la migración, ocurrirán los cambios siguientes: 
    1. El recurso de Azure Resource Manager cambia al supervisor de conexión más reciente.
        1. El nombre, la región y la suscripción del monitor de conexión no se modifican. Por lo tanto, no hay ningún impacto en el identificador de recurso.
        1. A menos que esté personalizada, se crea un área de trabajo de Log Analytics predeterminada en la región y suscripción del monitor de conexión. En esta área de trabajo se almacenarán los datos de supervisión. Los datos del resultado de la prueba también se almacenarán en las métricas.
        1. Cada prueba se migra a un grupo de pruebas denominado * defaultTestGroup*.
        1.  Los puntos de conexión de origen y destino se crean y usan en el grupo de pruebas creado. Los nombres predeterminados son *defaultSourceEndpoint* y *defaultDestinationEndpoint*.
        1. El puerto de destino y el intervalo de sondeo se trasladan a la configuración de prueba denominada *defaultTestConfiguration*. Según los valores de puerto, se establece el protocolo. Los umbrales de éxito y otras propiedades opcionales se dejan en blanco.
    1. Las alertas de métricas se migran a las alertas de métricas de Connection Monitor (versión preliminar). Las métricas son distintas <link to metric section in the doc>, de ahí el cambio.
    1. Los monitores de conexión migrados no se mostrarán en la solución de monitor de conexión anterior. Ahora solo estarán disponibles para su uso en Connection Monitor (versión preliminar).
    1. El usuario deberá migrar directamente toda integración externa, como los paneles de Power BI, Grafana, integraciones con sistemas SIEM. Este es el único manual que el usuario debe realizar para migrar la configuración.

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [migrar de Network Performance Monitor a Connection Monitor (versión preliminar)](migrate-to-connection-monitor-preview-from-network-performance-monitor.md).
* Aprenda a [crear Connection Monitor (versión preliminar) con Azure Portal](https://docs.microsoft.com/azure/network-watcher/connection-monitor-preview-create-using-portal).
