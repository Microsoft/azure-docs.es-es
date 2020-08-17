---
title: Introducción a Hybrid Runbook Worker de Azure Automation
description: En este artículo se ofrece información general de Hybrid Runbook Worker, que puede usar para ejecutar runbooks en máquinas de su centro de datos local o proveedor de nube.
services: automation
ms.subservice: process-automation
ms.date: 07/16/2020
ms.topic: conceptual
ms.openlocfilehash: 4d29979e28140b728478d405db934cb41783f4b0
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87448075"
---
# <a name="hybrid-runbook-worker-overview"></a>Introducción a Hybrid Runbook Worker

Es posible que los runbooks de Azure Automation no tengan acceso a los recursos de otras nubes o del entorno local, porque se ejecutan en plataforma de nube de Azure. La característica Hybrid Runbook Worker de Azure Automation permite ejecutar runbooks directamente en la máquina que hospeda el rol y en los recursos del entorno para administrar dichos recursos locales. Los runbooks se almacenan y administran en Azure Automation y después se entregan a una o más máquinas asignadas.

En la imagen siguiente se muestra esta funcionalidad:

![Introducción a Hybrid Runbook Worker](media/automation-hybrid-runbook-worker/automation.png)

Una instancia de Hybrid Runbook Worker puede ejecutarse en los sistemas operativos Windows o Linux. Depende del [agente de Log Analytics](../azure-monitor/platform/log-analytics-agent.md) que informa a un [área de trabajo de Log Analytics](../azure-monitor/platform/design-logs-deployment.md) de Azure Monitor. El área de trabajo no sirve solo para supervisar el sistema operativo compatible de la máquina, sino también para descargar los componentes necesarios para Hybrid Runbook Worker.

Cada Trabajo híbrido de runbook es miembro de un grupo de Trabajos híbridos de runbook que especifica cuando instala el agente. Un grupo puede incluir solo un agente, pero puede instalar varios agentes en un grupo para contar con alta disponibilidad. Cada máquina puede hospedar una instancia de Hybrid Worker que informe a una cuenta de Automation.

Cuando se inicia un runbook en Hybrid Runbook Worker, se especifica el grupo en el que se ejecuta. Cada rol de trabajo del grupo sondea Azure Automation para ver si hay trabajos disponibles. Si un trabajo está disponible, el primer rol trabajo en obtener el trabajo lo toma. El tiempo de procesamiento de la cola de trabajos depende de la carga y del perfil de hardware de Hybrid Worker. No se puede especificar un trabajo determinado.

Use una instancia de Hybrid Runbook Worker en lugar de un [espacio aislado de Azure](automation-runbook-execution.md#runbook-execution-environment), ya que no tiene muchos de los [límites](../azure-resource-manager/management/azure-subscription-service-limits.md#automation-limits) del espacio aislado en el espacio en disco, la memoria o los sockets de red. Los límites de una instancia de Hybrid Worker solo están relacionados con los propios recursos del rol de trabajo.

> [!NOTE]
> Las instancias de Hybrid Runbook Worker no están restringidas por el límite tiempo de [distribución equilibrada](automation-runbook-execution.md#fair-share) que tienen los espacios aislados de Azure.

## <a name="hybrid-runbook-worker-installation"></a>Instalación de una instancia de Hybrid Runbook Worker

El proceso para instalar una instancia de Hybrid Runbook Worker depende del sistema operativo. En la tabla siguiente, se definen los tipos de implementación.

|Sistema operativo  |Tipos de implementación  |
|---------|---------|
|Windows     | [Automatizado](automation-windows-hrw-install.md#automated-deployment)<br>[Manual](automation-windows-hrw-install.md#manual-deployment)        |
|Linux     | [Python](automation-linux-hrw-install.md#install-a-linux-hybrid-runbook-worker)        |

El método de instalación recomendado consiste en utilizar un runbook de Azure Automation para automatizar por completo el proceso de configuración de las máquinas Windows. Si no es posible, puede seguir un procedimiento paso a paso para instalar y configurar el rol manualmente. En el caso de las máquinas de Linux, debe ejecutar un script de Python para instalar el agente en la máquina.

## <a name="network-planning"></a><a name="network-planning"></a>Planeamiento de red

Para que Hybrid Runbook Worker se conecte y se registre con Azure Automation, debe tener acceso al número de puerto y a las direcciones URL que se describen en esta sección. El rol de trabajo también ha de tener acceso a los [puertos y las direcciones URL necesarios para que el agente de Log Analytics](../azure-monitor/platform/agent-windows.md) se conecte al área de trabajo Log Analytics de Azure Monitor.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

Los siguientes puertos y direcciones URL son necesarios para la instancia de Hybrid Runbook Worker:

* Puerto: solo se requiere el puerto TCP 443 para el acceso a Internet
* Dirección URL global: `*.azure-automation.net`
* Dirección URL global de US Gov Virginia: `*.azure-automation.us`
* Servicio del agente: `https://<workspaceId>.agentsvc.azure-automation.net`

Si tiene una cuenta de Automation definida para una región específica, puede restringir la comunicación de Hybrid Runbook Worker con ese centro de datos regional. Revise los registros DNS necesarios en los [registros DNS usados por Azure Automation](how-to/automation-region-dns-records.md).

### <a name="proxy-server-use"></a>Uso del servidor proxy

Si se usa un servidor proxy para la comunicación entre Azure Automation y las máquinas que ejecutan el agente de Log Analytics, asegúrese de que sea posible acceder a los recursos adecuados. El tiempo de expiración para las solicitudes de Hybrid Runbook Worker y los servicios de Automation es de 30 segundos. Después de tres intentos, se produce un error en una solicitud.

### <a name="firewall-use"></a>Uso del firewall

Si usa un firewall para restringir el acceso a Internet, tendrá que configurarlo para que permita el acceso. Si usa la puerta de enlace de Log Analytics como proxy, asegúrese de que está configurada para instancias de Hybrid Runbook Worker. Consulte el artículo [Conexión de equipos sin acceso a Internet mediante la puerta de enlace de Log Analytics en Azure Monitor](../azure-monitor/platform/gateway.md).

### <a name="service-tags"></a>Etiquetas de servicio

Azure Automation admite etiquetas de servicio de red virtual de Azure, empezando por la etiqueta de servicio [GuestAndHybridManagement](../virtual-network/service-tags-overview.md). Puede usar etiquetas de servicio para definir controles de acceso a la red en [grupos de seguridad de red](../virtual-network/security-overview.md#security-rules) o [Azure Firewall](../firewall/service-tags.md). Las etiquetas de servicio se pueden usar en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio **GuestAndHybridManagement** en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico para el servicio Automation. Esta etiqueta de servicio no permite un control más pormenorizado mediante la restricción de los intervalos de IP a una región específica.

La etiqueta de servicio de Azure Automation solo proporciona direcciones IP que se usan para los siguientes escenarios:

* Desencadenar webhooks desde dentro de la red virtual
* Permitir que los agentes de Hybrid Runbook Worker o State Configuration de la red virtual se comuniquen con el servicio Automation

>[!NOTE]
>La etiqueta de servicio **GuestAndHybridManagement** no admite actualmente la ejecución de trabajos de runbook en un espacio aislado de Azure, solo directamente en una instancia de Hybrid Runbook Worker.

## <a name="support-for-impact-level-5-il5"></a>Compatibilidad con el nivel 5 de impacto (IL5)

Hybrid Runbook Worker de Azure Automation se puede usar en Azure Government para admitir cargas de trabajo de nivel 5 de impacto en cualquiera de las dos configuraciones siguientes:

* [Máquina virtual aislada](../azure-government/documentation-government-impact-level-5.md#isolated-virtual-machines). Cuando se implementa, consume todo el host físico de esa VM, lo que proporciona el nivel necesario de aislamiento para admitir las cargas de trabajo IL5.

* [Azure Dedicated Host](../azure-government/documentation-government-impact-level-5.md#azure-dedicated-hosts), proporciona servidores físicos capaces de hospedar una o varias máquinas virtuales, dedicados a una suscripción de Azure.

>[!NOTE]
>El aislamiento de proceso mediante el rol Hybrid Runbook Worker está disponible para las nubes de Azure Commercial y US Government. 

## <a name="update-management-on-hybrid-runbook-worker"></a>Update Management en Hybrid Runbook Worker

Si habilita [Update Management](update-management/update-mgmt-overview.md) de Azure Automation, las máquinas conectadas al área de trabajo de Log Analytics se configurarán automáticamente como Hybrid Runbook Worker. Cada rol de trabajo puede admitir runbooks destinados a Update Management.

Una máquina configurada de esta manera no se registra en ningún grupo de Hybrid Runbook Worker ya definido en la cuenta de Automation. Puede agregar la máquina a un grupo de Hybrid Runbook Worker, pero debe usar la misma cuenta para Update Management y la pertenencia al grupo de Hybrid Runbook Worker. Esta funcionalidad se agregó a la versión 7.2.12024.0 de Hybrid Runbook Worker.

### <a name="update-management-addresses-for-hybrid-runbook-worker"></a>Direcciones de Update Management para Hybrid Runbook Worker

Además de las direcciones y puertos estándar necesarios para Hybrid Runbook Worker, Update Management tiene requisitos de configuración de red adicionales que se describen en la sección [planeamiento de red](update-management/update-mgmt-overview.md#ports).

## <a name="azure-automation-state-configuration-on-a-hybrid-runbook-worker"></a>State Configuration de Azure Automation en una instancia de Hybrid Runbook Worker

Puede ejecutar [State Configuration de Azure Automation](automation-dsc-overview.md) en una instancia de Hybrid Runbook Worker. Para administrar la configuración de los servidores que admiten la instancia de Hybrid Runbook Worker, debe agregar dichos servidores como nodos de DSC. Consulte el artículo [Incorporación de máquinas para su administración mediante Azure Automation State Configuration](automation-dsc-onboarding.md).

## <a name="runbooks-on-a-hybrid-runbook-worker"></a>Runbooks en Hybrid Runbook Worker

Podría tener runbooks que administren recursos en la máquina local o se ejecuten con recursos en el entorno local en el que se implementa una instancia de Hybrid Runbook Worker. En este caso, puede elegir ejecutar los runbooks en la instancia de Hybrid Worker en lugar de en una cuenta de Automation. La estructura de los runbooks que se ejecutan en una instancia de Hybrid Runbook Worker es idéntica a la de los que ejecuta en la cuenta de Automation. Consulte [Ejecución de runbooks en Hybrid Runbook Worker](automation-hrw-run-runbooks.md).

### <a name="hybrid-runbook-worker-jobs"></a>Trabajos de Hybrid Runbook Worker

Los trabajos de Hybrid Runbook Worker se ejecutan en la cuenta del **sistema** local en Windows o en la [cuenta nxautomation](automation-runbook-execution.md#log-analytics-agent-for-linux) en Linux. Azure Automation controla los trabajos en Hybrid Runbook Worker de forma ligeramente diferente a los trabajos que se ejecutan en espacios aislados de Azure. Consulte [Dónde ejecutar los runbooks](automation-runbook-execution.md#runbook-execution-environment).

Si la máquina host de Hybrid Runbook Worker se reinicia, cualquier trabajo de runbook en ejecución se reinicia desde el principio o desde el último punto de comprobación para runbooks de Flujo de trabajo de PowerShell. Si un trabajo de runbook se reinicia más de tres veces, se suspende.

### <a name="runbook-permissions-for-a-hybrid-runbook-worker"></a>Permisos del runbook para una instancia de Hybrid Runbook Worker

Puesto que acceden a recursos que no son de Azure, los runbooks que se ejecutan en una instancia de Hybrid Runbook Worker no pueden usar el mecanismo de autenticación que usan normalmente los runbooks que se autentican en los recursos de Azure. Un runbook proporciona su propia autenticación a los recursos locales o configura la autenticación mediante [identidades administradas para los recursos de Azure](../active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-arm.md#grant-your-vm-access-to-a-resource-group-in-resource-manager). También puede especificar una cuenta de ejecución para proporcionar un contexto de usuario para todos los runbooks.

## <a name="next-steps"></a>Pasos siguientes

* Consulte [Ejecución de runbooks en Hybrid Runbook Worker](automation-hrw-run-runbooks.md) para más información sobre cómo configurar los runbooks para automatizar los procesos del centro de datos local o de otros entornos de nube.

* Para aprender a solucionar problemas con las instancias de Hybrid Runbook Worker, consulte [Solución de incidencias de Hybrid Runbook Worker](troubleshoot/hybrid-runbook-worker.md#general).
