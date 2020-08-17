---
title: Compatibilidad para la evaluación de servidores físicos en Azure Migrate
description: Obtenga información sobre la compatibilidad para la evaluación de servidores físicos con la herramienta Server Assessment de Azure Migrate.
ms.topic: conceptual
ms.date: 06/03/2020
ms.openlocfilehash: 97da09fa88cc3e69965237cb5b4326b8b59739bd
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423786"
---
# <a name="support-matrix-for-physical-server-assessment"></a>Matriz de compatibilidad para la evaluación de servidores físicos 

En este artículo se resumen los requisitos previos y los requisitos de soporte técnico a la hora de evaluar servidores físicos para migrarlos a Azure mediante la herramienta [Server Assessment de Azure Migrate](migrate-services-overview.md#azure-migrate-server-assessment-tool). Si quiere migrar servidores físicos a Azure, revise la [matriz de compatibilidad para la migración](migrate-support-matrix-physical-migration.md).


Para evaluar los servidores físicos, cree un proyecto de Azure Migrate y, a continuación, agregue la herramienta Server Assessment al proyecto. Después de agregarla, implemente el [dispositivo de Azure Migrate](migrate-appliance.md). El dispositivo detecta las máquinas locales de forma continuada y envía metadatos de las máquinas y datos rendimiento a Azure. Una vez completada la detección, recopile las máquinas detectadas en grupos y ejecute una evaluación de un grupo.


## <a name="limitations"></a>Limitaciones

**Soporte técnico** | **Detalles**
--- | ---
**Límites de evaluación** | Puede detectar y evaluar hasta 35 000 servidores físicos en un solo [proyecto de Azure Migrate](migrate-support-matrix.md#azure-migrate-projects).
**Límites del proyecto** | Puede crear varios proyectos en una suscripción a Azure. Además de los servidores físicos, un proyecto puede incluir máquinas virtuales de VMware y máquinas virtuales de Hyper-V, hasta sus límites de evaluación.
**Detección** | El dispositivo de Azure Migrate puede detectar hasta 1000 servidores físicos.
**Valoración** | Puede agregar hasta 35 000 máquinas en un solo grupo.<br/><br/> Puede evaluar hasta 35 000 máquinas en una sola evaluación.

[Más información](concepts-assessment-calculation.md) sobre las evaluaciones.

## <a name="physical-server-requirements"></a>Requisitos del servidor físico

| **Soporte técnico**                | **Detalles**               
| :-------------------       | :------------------- |
| **Implementación de servidor físico**       | El servidor físico puede ser independiente o implementarse en un clúster. |
| **Permisos**           | **Windows:** Use una cuenta de dominio para máquinas unidas a un dominio y una cuenta local para las máquinas no unidas a un dominio. Debe agregar la cuenta de usuario a estos grupos: Usuarios de administración remota, Usuarios de Monitor de rendimiento y Usuarios del registro de rendimiento. <br/><br/> **Linux:** Necesita una cuenta raíz en los servidores Linux que desee detectar. |
| **Sistema operativo** | Todos los sistemas operativos excepto Windows Server 2003 y SUSE Linux se pueden evaluar para la migración. |


## <a name="azure-migrate-appliance-requirements"></a>Requisitos del dispositivo de Azure Migrate

Azure Migrate usa el [dispositivo de Azure Migrate](migrate-appliance.md) para la detección y la evaluación. El dispositivo para los servidores físicos se puede ejecutar en una máquina virtual o en una máquina física. 

- Obtenga información sobre los [requisitos del dispositivo](migrate-appliance.md#appliance---physical) para los servidores físicos.
- Obtenga información sobre las direcciones URL a las que el dispositivo necesita acceder en nubes [públicas](migrate-appliance.md#public-cloud-urls) y [gubernamentales](migrate-appliance.md#government-cloud-urls).
- Además, este dispositivo se configura mediante un [script de PowerShell](how-to-set-up-appliance-physical.md) que se descarga desde Azure Portal.
En Azure Government, debe implementar el dispositivo [mediante este script](deploy-appliance-script-government.md).

## <a name="port-access"></a>Acceso a puertos

En la tabla siguiente se resumen los requisitos de los puertos para la evaluación.

**Dispositivo** | **Connection**
--- | ---
**Dispositivo** | Conexiones entrantes en el puerto TCP 3389 para permitir las conexiones del Escritorio remoto al dispositivo.<br/><br/> Conexiones entrantes en el puerto 44368 para tener acceso de forma remota a la aplicación de administración del dispositivo mediante la dirección URL: ``` https://<appliance-ip-or-name>:44368 ```.<br/><br/> Conexiones salientes en el puerto 443 (HTTPS) para enviar metadatos de detección y rendimiento a Azure Migrate.
**Servidores físicos** | **Windows:** conexiones entrantes en el puerto 5985 (HTTP) de WinRM para extraer los metadatos de configuración y rendimiento de los servidores Windows. <br/><br/> **Linux:**  conexiones entrantes en el puerto 22 (TCP) para extraer los metadatos de configuración y rendimiento de los servidores Linux. |

## <a name="agent-based-dependency-analysis-requirements"></a>Requisitos para el análisis de dependencia con agentes

El [análisis de dependencias](concepts-dependency-visualization.md) le ayuda a identificar las dependencias entre las máquinas locales que quiere evaluar y migrar a Azure. En esta tabla se resumen los requisitos para configurar el análisis de dependencias basadas en agentes. Actualmente solo se admite el análisis de dependencias basadas en agentes para los servidores físicos.

**Requisito** | **Detalles** 
--- | --- 
**Antes de la implementación** | Debe tener un proyecto de Azure Migrate activo con la herramienta Server Assessment agregada.<br/><br/>  La visualización de dependencias se implementa después de configurar un dispositivo Azure Migrate para detectar las máquinas locales.<br/><br/> [Aprenda](create-manage-projects.md) a crear un proyecto por primera vez.<br/> [Aprenda](how-to-assess.md) a agregar una herramienta de evaluación a un proyecto existente.<br/> Aprenda a configurar un dispositivo de Azure Migrate para la evaluación de [Hyper-V](how-to-set-up-appliance-hyper-v.md), [VMware](how-to-set-up-appliance-vmware.md) o de servidores físicos.
**Azure Government** | La visualización de dependencias no está disponible en Azure Government.
**Log Analytics** | Azure Migrate utiliza la solución [Service Map](../azure-monitor/insights/service-map.md) de los [registros de Azure Monitor](../azure-monitor/log-query/log-query-overview.md) para la visualización de dependencias.<br/><br/> Puede asociar un área de trabajo de Log Analytics nueva o existente con un proyecto de Azure Migrate. El área de trabajo de un proyecto de Azure Migrate no se puede modificar una vez que se ha agregado. <br/><br/> El área de trabajo debe estar en la misma suscripción que el proyecto de Azure Migrate.<br/><br/> El área de trabajo debe residir en las regiones Este de EE. UU., Sudeste Asiático u Oeste de Europa. Las áreas de trabajo de otras regiones no se pueden asociar a un proyecto.<br/><br/> El área de trabajo debe estar en una región en la que [se admita Service Map](../azure-monitor/insights/vminsights-configure-workspace.md#supported-regions).<br/><br/> En Log Analytics, el área de trabajo asociada con Azure Migrate se etiqueta con la clave del proyecto de migración y el nombre del proyecto.
**Agentes necesarios** | En cada máquina que desee analizar, instale los siguientes agentes:<br/><br/> [Microsoft Monitoring agent (MMA)](../azure-monitor/platform/agent-windows.md).<br/> [Dependency Agent](../azure-monitor/platform/agents-overview.md#dependency-agent).<br/><br/> Si las máquinas locales no están conectadas a Internet, tiene que descargar e instalar la puerta de enlace de Log Analytics en ellas.<br/><br/> Conozca más información sobre la instalación de [Dependency Agent](how-to-create-group-machine-dependencies.md#install-the-dependency-agent) y [MMA](how-to-create-group-machine-dependencies.md#install-the-mma).
**Área de trabajo de Log Analytics** | El área de trabajo debe estar en la misma suscripción que el proyecto de Azure Migrate.<br/><br/> Azure Migrate admite áreas de trabajo que residen en las regiones Este de EE. UU., Sudeste Asiático y Oeste de Europa.<br/><br/>  El área de trabajo debe estar en una región en la que [se admita Service Map](../azure-monitor/insights/vminsights-configure-workspace.md#supported-regions).<br/><br/> El área de trabajo de un proyecto de Azure Migrate no se puede modificar una vez que se ha agregado.
**Costos** | La solución Service Map no supone ningún gasto durante los primeros 180 días (a partir del día que asocie el área de trabajo de Log Analytics con el proyecto de Azure Migrate).<br/><br/> Transcurridos los 180 días, se aplicarán las tarifas normales de Log Analytics.<br/><br/> Si se usa una solución que no sea Service Map en el área de trabajo de Log Analytics asociada generará los [gastos estándar](https://azure.microsoft.com/pricing/details/log-analytics/) de Log Analytics.<br/><br/> Al eliminar el proyecto de Azure Migrate, el área de trabajo no se elimina con él. Tras la eliminación del proyecto, el uso de Service Map no será gratuito y cada nodo se facturará conforme al nivel de pago del área de trabajo de Log Analytics.<br/><br/>Si tiene proyectos creados antes de que Azure Migrate estuviera disponible con carácter general (el 28 de febrero de 2018), puede que haya incurrido en cargos de Service Map adicionales. Para garantizar el pago solo después de 180 días, es recomendable que cree un nuevo proyecto, ya que las áreas de trabajo existentes antes de la disponibilidad general siguen siendo facturables.
**Administración** | Al registrar agentes en el área de trabajo, se usa el identificador y la clave proporcionados por el proyecto de Azure Migrate.<br/><br/> Puede usar el área de trabajo de Log Analytics fuera de Azure Migrate.<br/><br/> Si elimina el proyecto de Azure Migrate asociado, el área de trabajo no se elimina automáticamente. [Elimínela manualmente](../azure-monitor/platform/manage-access.md).<br/><br/> No elimine el área de trabajo creado por Azure Migrate, a menos que elimine el proyecto de Azure Migrate. Si lo hace, la característica de visualización de dependencias no funcionará según lo previsto.
**Conectividad de Internet** | Si las máquinas no están conectadas a Internet, necesita instalar la puerta de enlace de Log Analytics en ellas.
**Azure Government** | El análisis de dependencias basado en agente no se admite.

## <a name="next-steps"></a>Pasos siguientes

[Preparación de la valoración de servidores físicos](tutorial-prepare-physical.md).
