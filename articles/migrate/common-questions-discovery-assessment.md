---
title: Preguntas sobre la detección, la valoración y el análisis de dependencias en Azure Migrate
description: Obtenga respuestas a preguntas comunes sobre detección, valoración y análisis de dependencias en Azure Migrate.
ms.topic: conceptual
ms.date: 06/09/2020
ms.openlocfilehash: d4f833926541bafbae211caff37e5974b57e1047
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019018"
---
# <a name="discovery-assessment-and-dependency-analysis---common-questions"></a>Detección, valoración y análisis de dependencias: preguntas comunes

En este artículo se responde a preguntas comunes sobre detección, valoración y análisis de dependencias en Azure Migrate. Si tiene otras preguntas, repase estos recursos:

- [Preguntas generales](resources-faq.md) sobre Azure Migrate
- Preguntas sobre el [dispositivo de Azure Migrate](common-questions-appliance.md)
- Preguntas sobre la [migración de servidores](common-questions-server-migration.md)
- Obtenga respuestas a sus preguntas en el [foro de Azure Migrate](https://aka.ms/AzureMigrateForum)


## <a name="what-geographies-are-supported-for-discovery-and-assessment-with-azure-migrate"></a>¿Qué zonas geográficas se admiten para la detección y evaluación con Azure Migrate?

Revise las zonas geográficas admitidas para [nubes públicas](migrate-support-matrix.md#supported-geographies-public-cloud) y [gubernamentales](migrate-support-matrix.md#supported-geographies-azure-government).


## <a name="how-many-vms-can-i-discover-with-an-appliance"></a>¿Cuántas máquinas virtuales puedo detectar con un dispositivo?

Puede detectar hasta 10 000 máquinas virtuales de VMware, 5000 máquinas virtuales de Hyper-V y 1000 servidores físicos con un único dispositivo. Si tiene más máquinas, lea acerca de [escalar una valoración de Hyper-V](scale-hyper-v-assessment.md), [escalar una valoración de VMware](scale-vmware-assessment.md) y [escalar una valoración de servidor físico](scale-physical-assessment.md).

## <a name="how-do-i-choose-the-assessment-type"></a>¿Cómo elegir el tipo de valoración?

- Utilice **las valoraciones de máquinas virtuales de Azure** cuando quiera evaluar [las máquinas virtuales VMware](how-to-set-up-appliance-vmware.md) locales, [las máquinas virtuales Hyper-V](how-to-set-up-appliance-hyper-v.md) y [los servidores físicos](how-to-set-up-appliance-physical.md) locales para la migración a máquinas virtuales de Azure. [Más información](concepts-assessment-calculation.md)

- Utilice las valoraciones de **Azure VMware Solution (AVS)** cuando quiera evaluar [las máquinas virtuales VMware](how-to-set-up-appliance-vmware.md) locales para la migración a [Azure VMware Solution (AVS)](../azure-vmware/introduction.md) con este tipo de valoración. [Más información](concepts-azure-vmware-solution-assessment-calculation.md)

- Puede usar un grupo común con máquinas de VMware solo para ejecutar ambos tipos de valoraciones. Tenga en cuenta que si está ejecutando valoraciones de AVS en Azure Migrate por primera vez, es aconsejable crear un nuevo grupo de máquinas de VMware.
 

## <a name="why-is-performance-data-missing-for-someall-vms-in-my-assessment-report"></a>¿Por qué faltan datos de rendimiento de algunas o todas las máquinas virtuales en el informe de evaluación?

En el caso de la evaluación "en función del rendimiento", la exportación del informe de valoración indica "PercentageOfCoresUtilizedMissing" o "PercentageOfMemoryUtilizedMissing" cuando el dispositivo de Azure Migrate no puede recopilar datos de rendimiento de las máquinas virtuales locales pertinentes. Compruebe:

- Si las máquinas virtuales están encendidas durante el tiempo que ha estado creando la evaluación
- Si solo faltan contadores de memoria y está intentando evaluar las máquinas virtuales de Hyper-V, compruebe si tiene habilitada la memoria dinámica en estas máquinas virtuales. Existe un problema conocido debido a que el dispositivo de Azure Migrate no puede recopilar el uso de memoria de dichas máquinas virtuales.
- Si faltan todos los contadores de rendimiento, asegúrese de que se permiten las conexiones de salida en los puertos 443 (HTTPS).

Nota: Si falta alguno de los contadores de rendimiento, Azure Migrate: Server Assessment vuelve a la memoria y los núcleos locales asignados, y recomienda un tamaño de máquina virtual acorde.

## <a name="why-is-the-confidence-rating-of-my-assessment-low"></a>¿Por qué la clasificación de confianza de mi valoración es baja?

La clasificación de confianza se calcula para las evaluaciones "en función del rendimiento" en función del porcentaje de [puntos de datos disponibles](https://docs.microsoft.com/azure/migrate/concepts-assessment-calculation#ratings) necesarios para calcular la evaluación. Estos son los motivos por los que una valoración puede obtener una clasificación de confianza baja:

- No generó un perfil de su entorno durante el tiempo que está creando la evaluación. Por ejemplo, si está creando una evaluación con la duración de rendimiento establecida en una semana, debe esperar al menos una semana después de iniciar la detección para que se recopilen todos los puntos de datos. Si no puede esperar a la duración, cambie la duración del rendimiento a un período más pequeño y "recalcule" la evaluación.
 
- La evaluación del servidor no puede recopilar los datos de rendimiento de algunas o de todas las máquinas virtuales en el período de evaluación. Compruebe si las máquinas virtuales se encendieron mientras dura la evaluación, se permiten las conexiones salientes en los puertos 443. En el caso de las máquinas virtuales Hyper-V, si la memoria dinámica está habilitada, los contadores de memoria no tendrán una clasificación de confianza baja. "Recalcule" la evaluación para reflejar los cambios más recientes en la clasificación de confianza. 

- Algunas máquinas virtuales se crearon después de iniciar la detección en Server Assessment. Por ejemplo, si va a crear una valoración para el historial de rendimiento del último mes, pero algunas máquinas virtuales se crearon en el entorno hace solo una semana. En este caso, los datos de rendimiento de las nuevas máquinas virtuales no estarán disponibles en ningún momento y la clasificación de confianza sería baja.

[Obtenga más información](https://docs.microsoft.com/azure/migrate/concepts-assessment-calculation#confidence-ratings-performance-based) sobre la clasificación de confianza.

## <a name="i-cant-see-some-groups-when-i-am-creating-an-azure-vmware-solution-avs-assessment"></a>No puedo ver algunos grupos cuando creo una valoración de Azure VMware Solution (AVS).

- La valoración de AVS se puede realizar en grupos que solo tienen máquinas de VMware. Quite cualquier máquina que no sea de VMware del grupo si pretende realizar una valoración de AVS.
- Si está ejecutando valoraciones de AVS en Azure Migrate por primera vez, es aconsejable crear un nuevo grupo de máquinas de VMware.

## <a name="i-cant-see-some-vm-types-in-azure-government"></a>No puedo ver algunos tipos de VM en Azure Government

Los tipos de VM que se admiten para la evaluación y la migración dependen de la disponibilidad en la ubicación de Azure Government. Puede [revisar y comparar](https://azure.microsoft.com/global-infrastructure/services/?regions=usgov-non-regional,us-dod-central,us-dod-east,usgov-arizona,usgov-iowa,usgov-texas,usgov-virginia&products=virtual-machines) los tipos de VM en Azure Government.

## <a name="the-size-of-my-vm-changed-can-i-run-an-assessment-again"></a>El tamaño de mi VM ha cambiado. ¿Puedo volver a ejecutar una valoración?

El dispositivo de Azure Migrate recopila continuamente información sobre el entorno local.  Una valoración es una instantánea de un momento dado de las VM locales. Si cambia la configuración de una máquina virtual que quiere evaluar, use la opción de recalcular para actualizar la evaluación con los últimos cambios.

## <a name="how-do-i-discover-vms-in-a-multitenant-environment"></a>¿Cómo puedo detectar máquinas virtuales en un entorno de varios inquilinos?

- **VMware**: Si un entorno se comparte entre los inquilinos y no quiere detectar las VM de un inquilino en la suscripción de otro inquilino, cree credenciales de VMware vCenter Server con acceso solo a las VM que quiere detectar. Después, use esas credenciales al iniciar la detección en el dispositivo de Azure Migrate.
- **Hyper-V**: la detección usa credenciales de host de Hyper-V. Si las máquinas virtuales comparten el mismo host de Hyper-V, no hay actualmente ninguna manera de separar la detección.  

## <a name="do-i-need-vcenter-server"></a>¿vCenter Server es necesario?

Sí, Azure Migrate necesita vCenter Server para detectar un entorno de VMware a fin de realizar la detección. Azure Migrate no admite la detección de hosts de ESXi que no administra vCenter Server.

## <a name="what-are-the-sizing-options-in-an-azure-vm-assessment"></a>¿Cuáles son las opciones de dimensionamiento en una evaluación de máquinas virtuales de Azure?

Con el ajuste como tamaño local, Azure Migrate no tiene en cuenta los datos de rendimiento de la máquina virtual para la evaluación. Azure Migrate evalúa los tamaños de las VM en función de la configuración local. Con el ajuste de tamaño basado en el rendimiento, el ajuste de tamaño se basa en los datos de uso.

Por ejemplo, si una VM local tiene 4 núcleos y 8 GB de memoria con un 50 % de uso de CPU y un 50 % de uso de memoria:
- El ajuste de tamaño como local recomienda una SKU de máquina virtual de Azure con cuatro núcleos y 8 GB de memoria.
- El ajuste de tamaño basado en el rendimiento recomienda una SKU de VM de 2 núcleos y 4 GB de memoria, ya que se tiene en cuenta el porcentaje de uso.

De forma similar, el ajuste de tamaño de disco depende del criterio de ajuste de tamaño y el tipo de almacenamiento:
- Si el criterio de ajuste de tamaño se basa en el rendimiento y el tipo de almacenamiento es automático, Azure Migrate tiene en cuenta los valores de rendimiento y de IOPS del disco al identificar el tipo de disco de destino (estándar o prémium).
- Si el criterio de ajuste de tamaño está basado en el rendimiento y el tipo de almacenamiento es prémium, Azure Migrate recomienda una SKU de disco premium, basada en el tamaño del disco local. Se aplica la misma lógica para ajustar el tamaño del disco cuando el criterio de ajuste de tamaño es como local y el tipo de almacenamiento es estándar o prémium.

## <a name="does-performance-history-and-utilization-affect-sizing-in-an-azure-vm-assessment"></a>¿El historial de rendimiento y el uso afectan al dimensionamiento en una valoración de máquinas virtuales de Azure?

Sí, el historial de rendimiento y el uso afectan al dimensionamiento de máquinas virtuales de Azure.

### <a name="performance-history"></a>Historial de rendimiento

Solo para el ajuste de tamaño basado en el rendimiento, Azure Migrate recopila el historial de rendimiento de las máquinas locales y lo usa para recomendar el tipo de disco y el tamaño de la VM en Azure:

1. El dispositivo realiza un perfil del entorno local continuamente para recopilar datos de uso en tiempo real cada 20 segundos.
2. El dispositivo acumula los ejemplos de 20 segundos recopilados y los usa para crear un único punto de datos cada 15 minutos.
3. Para crear el punto de datos, el dispositivo selecciona el valor máximo de todos los ejemplos de 20 segundos.
4. El dispositivo envía el punto de datos a Azure.

### <a name="utilization"></a>Utilización

Cuando se crea una evaluación en Azure, en función de la duración del rendimiento y el valor del percentil del historial de rendimiento, Azure Migrate calcula el valor del uso efectivo y lo usa para ajustar el tamaño.

Por ejemplo, si establece la duración del rendimiento en 1 día y el valor del percentil en el percentil 95, Azure Migrate ordena los puntos de ejemplo de 15 minutos que envía el recopilador para el último día de manera ascendente. Elige el valor del percentil 95 como el uso efectivo.

El uso del valor del percentil 95 garantiza la omisión de todos los valores atípicos. Los valores atípicos se pueden incluir si Azure Migrate usa el percentil 99. Para elegir el uso máximo para el período sin omitir ningún valor atípico, establezca Azure Migrate para que utilice el percentil 99.


## <a name="how-are-import-based-assessments-different-from-assessments-with-discovery-source-as-appliance"></a>¿En qué se diferencian las valoraciones basadas en la importación de las valoraciones con el origen de detección como dispositivo?

Las valoraciones de máquinas virtuales de Azure basadas en la importación son valoraciones creadas con máquinas que se importan en Azure Migrate mediante un archivo CSV. Solo cuatro campos son obligatorios para la importación: nombre del servidor, núcleos, memoria y sistema operativo. Estos son algunos aspectos que hay que tener en cuenta: 
 - Los criterios de preparación son menos rigurosos en las valoraciones basadas en la importación en el parámetro de tipo de arranque. Si no se proporciona el tipo de arranque, se supone que la máquina usa el tipo BIOS y que no está macada como **Condicionalmente preparada**. En las valoraciones con el origen de detección como dispositivo, la preparación se marca como **Condicionalmente preparada** si falta el tipo de arranque. Esta diferencia en el cálculo de la preparación se debe a la posibilidad de que los usuarios no tengan toda la información sobre las máquinas en las primeras fases del planeamiento de la migración cuando se realizan las valoraciones basadas en la importación. 
 - Las valoraciones de importación basadas en el rendimiento usan el valor de uso proporcionado por el usuario para el cálculo de tamaño correcto. Dado que el valor de uso lo proporciona el usuario, las opciones **Historial de rendimiento** y **Utilización del percentil** están deshabilitadas en las propiedades de la valoración. En las valoraciones con el origen de detección como dispositivo, el valor del percentil elegido se selecciona de los datos de rendimiento recopilados por el dispositivo.

## <a name="why-is-the-suggested-migration-tool-in-import-based-avs-assessment-marked-as-unknown"></a>¿Por qué la herramienta de migración sugerida en la evaluación de AVS basada en la importación está marcada como desconocida?

En el caso de las máquinas importadas mediante un archivo CSV, se desconoce la herramienta de migración predeterminada en una valoración de AVS. Sin embargo, para las máquinas de VMware, se recomienda usar la solución Hybrid Cloud Extension (HCX) de VMware. [Más información](../azure-vmware/hybrid-cloud-extension-installation.md).


## <a name="what-is-dependency-visualization"></a>¿Qué es la visualización de dependencias?

La visualización de dependencias puede ayudarle a evaluar grupos de VM para realizar la migración con mayor confianza. La visualización de dependencias comprueba las dependencias de las máquinas antes de llevar a cabo una valoración. Este proceso ayuda a asegurar que nada se queda atrás y evita interrupciones inesperadas al migrar a Azure. Azure Migrate usa la solución Service Map en Azure Monitor para habilitar la visualización de dependencias. [Más información](concepts-dependency-visualization.md).

> [!NOTE]
> El análisis de dependencias basado en agente no está disponible en Azure Government. Puede usar el análisis de dependencias sin agente

## <a name="whats-the-difference-between-agent-based-and-agentless"></a>¿Cuál es la diferencia entre basado en agente y sin agente?

La diferencia entre la visualización sin agente y la visualización basada en agente se resume en la siguiente tabla.

**Requisito** | **Sin agente** | **Basado en agente**
--- | --- | ---
Soporte técnico | Esta opción se encuentra actualmente en versión preliminar y solo está disponible para máquinas virtuales de VMware. [Revise](migrate-support-matrix-vmware.md#dependency-analysis-requirements-agentless) los sistemas operativos compatibles. | En disponibilidad general (GA).
Agente | No es necesario instalar agentes en las máquinas que quiere comprobar. | Agentes que debe instalar en las máquinas locales que quiere analizar: [Microsoft Monitoring Agent (MMA)](../azure-monitor/platform/agent-windows.md) y el [agente de dependencia](../azure-monitor/platform/agents-overview.md#dependency-agent). 
Prerrequisitos | [Revise](concepts-dependency-visualization.md#agentless-analysis) los requisitos previos y los requisitos de implementación. | [Revise](concepts-dependency-visualization.md#agent-based-analysis) los requisitos previos y los requisitos de implementación.
Log Analytics | No se requiere. | Azure Migrate utiliza la solución [Service Map](../azure-monitor/insights/service-map.md) de los [registros de Azure Monitor](../azure-monitor/log-query/log-query-overview.md) para la visualización de dependencias. [Más información](concepts-dependency-visualization.md#agent-based-analysis).
Funcionamiento | Captura los datos de conexión TCP en las máquinas habilitadas para la visualización de dependencias. Después de la detección, recopila datos en intervalos de cinco minutos. | Los agentes de Service Map instalados en una máquina recopilan datos acerca de los procesos de TCP, así como de las conexiones de entrada o salida para cada proceso.
data | Nombre de aplicación, proceso y nombre del servidor de la máquina de origen.<br/><br/> Puerto, nombre de aplicación, proceso y nombre del servidor de la máquina de destino. | Nombre de aplicación, proceso y nombre del servidor de la máquina de origen.<br/><br/> Puerto, nombre de aplicación, proceso y nombre del servidor de la máquina de destino.<br/><br/> Se recopila la información sobre el número de conexiones, la latencia y la transferencia de datos, y está disponible para las consultas de Log Analytics. 
Visualización | El mapa de dependencias de un solo servidor se puede ver durante un plazo de entre una hora y 30 días. | Mapa de dependencia de un solo servidor.<br/><br/> El mapa puede verse solo durante una hora.<br/><br/> Mapa de dependencias de un grupo de servidores.<br/><br/> Agregue y quite servidores de un grupo desde la vista de mapa.
Exportación de datos | Los datos de los últimos 30 días se pueden descargar en formato CSV. | Los datos se pueden consultar con Log Analytics.


## <a name="do-i-need-to-deploy-the-appliance-for-agentless-dependency-analysis"></a>¿Es necesario implementar el dispositivo para el análisis de dependencia sin agente?

Sí, se debe implementar el [dispositivo de Azure Migrate](migrate-appliance.md).

## <a name="do-i-pay-for-dependency-visualization"></a>¿Tengo que pagar por la visualización de dependencias?

No. Más información sobre los [precios de Azure Migrate](https://azure.microsoft.com/pricing/details/azure-migrate/).

## <a name="what-do-i-install-for-agent-based-dependency-visualization"></a>¿Qué tengo que instalar para la visualización de dependencias basada en agente?

Para usar la visualización de dependencias basada en agente, descargue e instale agentes en cada máquina local que vaya a evaluar:

- [Microsoft Monitoring Agent (MMA)](../azure-monitor/platform/agent-windows.md)
- [Dependency Agent](../azure-monitor/platform/agents-overview.md#dependency-agent)
- Si también tiene máquinas sin conectividad a Internet, descargue e instale en ellas la puerta de enlace de Log Analytics.

Solo necesita estos agentes si utiliza la visualización de dependencias basada en agente.

## <a name="can-i-use-an-existing-workspace"></a>¿Puedo usar un área de trabajo existente?

Sí, en la visualización de dependencias basada en agente puede asociar un área de trabajo existente al proyecto de migración y usarla para la visualización de dependencias. 

## <a name="can-i-export-the-dependency-visualization-report"></a>¿Puedo exportar el informe de visualización de dependencias?

No, no se puede exportar el informe de visualización de dependencias en la visualización basada en agente. Sin embargo, Azure Migrate usa Service Map, y puede usar la [API REST de Service Map](/rest/api/servicemap/machines/listconnections) para obtener las dependencias en formato JSON.

## <a name="can-i-automate-agent-installation"></a>¿Puedo automatizar la instalación de los agentes?

Para la visualización de dependencias basada en agente:

- Use un [script para instalar el agente de dependencias](../azure-monitor/insights/vminsights-enable-hybrid.md#dependency-agent).
- En el caso de MMA, [use la línea de comandos o la automatización](../azure-monitor/platform/log-analytics-agent.md#installation-options), o bien use un [script](https://gallery.technet.microsoft.com/scriptcenter/Install-OMS-Agent-with-2c9c99ab).
- Además de los scripts, puede usar herramientas de implementación como Microsoft Endpoint Configuration Manager e [Intigua](https://www.intigua.com/intigua-for-azure-migration) para implementar los agentes.

## <a name="what-operating-systems-does-mma-support"></a>¿Qué sistemas operativos admite MMA?

- Vea la lista de los [sistemas operativos Windows compatibles con MMA](../azure-monitor/platform/log-analytics-agent.md#installation-options).
- Vea la lista de los [sistemas operativos Linux compatibles con MMA](../azure-monitor/platform/log-analytics-agent.md#installation-options).

## <a name="can-i-visualize-dependencies-for-more-than-one-hour"></a>¿Puedo visualizar las dependencias durante más de una hora?

Para la visualización basada en agente, puede visualizar las dependencias durante un máximo de una hora. Puede retroceder hasta un mes a una fecha determinada del historial, pero la duración máxima de la visualización es de una hora. Por ejemplo, puede usar la característica de duración de tiempo en el mapa de dependencias para ver las dependencias de ayer, pero solo puede verlas durante un período de una hora. No obstante, puede usar los registros de Azure Monitor para [consultar los datos de dependencias](./how-to-create-group-machine-dependencies.md) durante un período más largo.

Para la visualización sin agente, puede ver el mapa de dependencias de un solo servidor durante un plazo de entre una hora y 30 días.

## <a name="can-i-visualize-dependencies-for-groups-of-more-than-10-vms"></a>¿Puedo visualizar dependencias de grupos de más de 10 VM?

Puede [visualizar las dependencias](./how-to-create-a-group.md#refine-a-group-with-dependency-mapping) de grupos que tengan un máximo de 10 VM. Si tiene un grupo con más de 10 VM, se recomienda dividirlo en grupos más pequeños y luego visualizar las dependencias.

## <a name="next-steps"></a>Pasos siguientes

Consulte la sección [Información general de Azure Migrate](migrate-services-overview.md).