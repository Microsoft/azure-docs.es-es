---
title: Problemas de latido de Apache Ambari en Azure HDInsight
description: Revisión de varias causas para los problemas de latido de Apache Ambari en Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 02/06/2020
ms.openlocfilehash: 5eebde42098d74f533565d274b693c4a06f2f60d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/19/2021
ms.locfileid: "98946737"
---
# <a name="apache-ambari-heartbeat-issues-in-azure-hdinsight"></a>Problemas de latido de Apache Ambari en Azure HDInsight

En este artículo se describen los pasos de solución de problemas y las posibles soluciones para los problemas que se producen al usar clústeres de Azure HDInsight.

## <a name="scenario-high-cpu-utilization"></a>Escenario: alto uso de CPU

### <a name="issue"></a>Problema

El agente de Ambari tiene un uso elevado de CPU. Como resultado, se generan alertas de la interfaz de usuario de Ambari informando que se perdió el latido del agente de Ambari para algunos nodos. La alerta de pérdida de latido suele ser transitoria.

### <a name="cause"></a>Causa

Debido a varios errores de ambari-agent, en raras ocasiones, ambari-agent puede tener un uso elevado de CPU (cercano al 100).

### <a name="resolution"></a>Solución

1. Identifique el ID. de proceso (PID) de ambari-agent:

    ```bash
    ps -ef | grep ambari_agent
    ```

1. A continuación, ejecute el comando siguiente para mostrar el uso de la CPU:

    ```bash
    top -p <ambari-agent-pid>
    ```

1. Reinicie ambari-agent para mitigar el problema:

    ```bash
    service ambari-agent restart
    ```

1. Si el reinicio no funciona, termine el proceso ambari-agent y, a continuación, inícielo:

    ```bash
    kill -9 <ambari-agent-pid>
    service ambari-agent start
    ```

---

## <a name="scenario-ambari-agent-not-started"></a>Escenario: El agente de Ambari no se ha iniciado

### <a name="issue"></a>Problema

El agente de Ambari no se ha iniciado. Como resultado, se generan alertas de la interfaz de usuario de Ambari que informan de que se ha perdido el latido del agente de Ambari para algunos nodos.

### <a name="cause"></a>Causa

Las alertas se deben a que el agente de Ambari no se está ejecutando.

### <a name="resolution"></a>Solución

1. Confirme el estado de ambari-agent:

    ```bash
    service ambari-agent status
    ```

1. Confirme si se están ejecutando los servicios de controlador de conmutación por error:

    ```bash
    ps -ef | grep failover
    ```

    Si los servicios de controlador de conmutación por error no se están ejecutando, probablemente se deba a un problema que impide que hdinsight-agent inicie el controlador de conmutación por error. Compruebe el registro de hdinsight-agent desde el archivo `/var/log/hdinsight-agent/hdinsight-agent.out`.

## <a name="scenario-heartbeat-lost-for-ambari"></a>Escenario: Pérdida de latido para Ambari

### <a name="issue"></a>Problema

Se perdió el agente de latido de Ambari.

### <a name="cause"></a>Causa

Los registros de OMS están causando un uso elevado de la CPU.

### <a name="resolution"></a>Solución

* Deshabilite el registro de Azure Monitor mediante el cmdlet [Disable-AzHDInsightMonitoring](/powershell/module/az.hdinsight/disable-azhdinsightmonitoring) de PowerShell.
* Elimine el archivo de registro `mdsd.warn`.

---

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]