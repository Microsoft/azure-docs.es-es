---
title: Migración a Connection Monitor (versión preliminar) de Network Performance Monitor
titleSuffix: Azure Network Watcher
description: Aprenda a migrar a Connection Monitor (versión preliminar) de Network Performance Monitor.
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
ms.openlocfilehash: 69dbb1dd4017c5acf9c195f5104741caee38c2b7
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88701347"
---
# <a name="migrate-to-connection-monitor-preview-from-network-performance-monitor"></a>Migración a Connection Monitor (versión preliminar) de Network Performance Monitor

Puede migrar las pruebas de Network Performance Monitor al nuevo y mejorado Connection Monitor (versión preliminar) con un solo clic y sin tiempo de inactividad. Para más información sobre las ventajas, puede consultar [Connection Monitor (versión preliminar)](https://docs.microsoft.com/azure/network-watcher/connection-monitor-preview).

>[!NOTE]
> Solo las pruebas del Monitor de conectividad de servicio se pueden migrar a Connection Monitor (version preliminar).
>

## <a name="key-points-to-note"></a>Puntos clave a tener en cuenta

* Los agentes locales y la configuración de firewall funcionarán tal cual. No se requieren cambios. Los agentes de Log Analytics instalados en Azure Virtual Machines deben reemplazarse por la extensión Network Watcher.
* Las pruebas existentes se asignarán a Connection Monitor (versión preliminar) -> Grupo de pruebas -> Formato de prueba. Los usuarios pueden hacer clic en *Editar* para ver y modificar las propiedades del Connection Monitor nuevo y descargar la plantilla para hacer cambios en Connection Monitor y enviarlo a través de Azure Resource Manager.
* Los agentes envían datos tanto al área de trabajo de Log Analytics y a las métricas.
* Supervisión de datos
    * Datos en Log Analytics: todos los datos previos a la migración continúan en el área de trabajo en la que está configurado NPM en la tabla NetworkMonitoring. Después de la migración, los datos van a la tabla NetworkMonitoring y a la tabla ConnectionMonitor_CL en la misma área de trabajo. Una vez que se deshabilitan las pruebas desde NPM, los datos solo se almacenan en la tabla ConnectionMonitor_CL.
    * Integraciones, paneles y alertas que se basan en registros: deberá editar manualmente las consultas en función de la tabla ConnectionMonitor_CL nueva. También puede volver a crear las alertas en las métricas con este vínculo. Pronto estará disponible la capacidad de migrar alertas basadas en registros en la tabla NetworkMonitoring a alertas basadas en métricas como parte de la migración.
    
## <a name="prerequisites"></a>Requisitos previos

*   Asegúrese de que Network Watcher esté habilitado en la suscripción y la región del área de trabajo de Log Analytics.
*   Las máquinas virtuales de Azure con agentes de Log Analytics instalados deberán estar habilitadas con la extensión Network Watcher.

## <a name="steps-to-migrate-tests-from-network-performance-monitor-to-connection-monitor-preview"></a>Pasos para migrar pruebas de Network Performance Monitor a Connection Monitor (versión preliminar)

1.  Haga clic en "Connection Monitor", vaya a "Migrate tests from NPM" (Migrar pruebas desde NPM) para migrar las pruebas a Connection Monitor (versión preliminar).

    ![Captura de pantalla que muestra la migración de pruebas de NPM a Connection Monitor (versión preliminar)](./media/connection-monitor-2-preview/migrate-npm-to-cm-preview.png)
    
1.  Seleccione suscripción, área de trabajo y la característica de NPM que quiere migrar. Actualmente solo se pueden migrar pruebas del Monitor de conectividad de servicio.  
1.  Haga clic en "Importar" para migrar las pruebas.
1.  Una vez que se inicie la migración, ocurrirán los cambios siguientes: 
    1. Se crea un recurso de monitor de conexión nuevo.
        1. Se crea un monitor de conexión por región y suscripción. En el caso de las pruebas con agentes locales, el nombre del monitor de conexión nuevo tiene el formato <workspaceName> _"local". En el caso de las pruebas con agentes de Azure, el nombre del monitor de conexión nuevo tiene el formato <workspaceName>_ <nombre_de_la_región_de_Azure>.
        1. Los datos de supervisión ahora se almacenan en la misma área de trabajo de Log Analytics donde está habilitado NPM, en una tabla nueva llamada Connectionmonitor_CL. 
        1. El nombre de la prueba se mantiene como el nombre del grupo de pruebas. La descripción de la prueba no se migrará.
        1. Los puntos de conexión de origen y destino se crean y usan en el grupo de pruebas creado. En el caso de los agentes locales, el nombre de los puntos de conexión tiene el formato <workspaceName> _"punto de conexión"_ <FQDN of on-premises machine>. En el caso de Azure, si las pruebas de migración contienen agentes que no están en ejecución, deberá habilitar a los agentes y volver a realizar la migración.
        1. El puerto de destino y el intervalo de sondeo se trasladan a la configuración de prueba, es decir, "TC" _<testname>" y "TC"_ <testname>_"AppThresholds". Según los valores de puerto, se establece el protocolo. Los umbrales de éxito y otras propiedades opcionales se dejan en blanco.
    1. NPM no se deshabilita. Por lo tanto, las pruebas migradas siguen enviando datos a la tabla NetworkMonitoring, así como a la tabla ConnectionMonitor_CL. Este paso garantiza que las integraciones y las alertas basadas en registros no se vean afectadas. Pronto estará disponible la migración de alertas basadas en registros en la tabla NetworkMonitoring a alertas basadas en métricas como parte de la migración.
    1. El monitor de conexión recién creado estará visible en Connection Monitor (versión preliminar).
1.  Después de la migración, deberá deshabilitar manualmente las pruebas en NPM. Hasta que lo haga, se le seguirán cobrando. Al deshabilitar NPM, asegúrese de volver a crear las alertas en la tabla ConnectionMonitor_CL o de usar las métricas. Asegúrese también de que todas las integraciones externas, como los paneles de Power BI, Grafana, integraciones con sistemas SIEM, se deberán migrar a la tabla ConnectionMonitor_CL.


## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [migrar de Connection Monitor a Connection Monitor (versión preliminar)](migrate-to-connection-monitor-preview-from-connection-monitor.md).
* Aprenda a [crear Connection Monitor (versión preliminar) con Azure Portal](https://docs.microsoft.com/azure/network-watcher/connection-monitor-preview-create-using-portal).
