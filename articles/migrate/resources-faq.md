---
title: Preguntas más frecuentes sobre Azure Migrate
description: Obtenga respuestas a preguntas comunes sobre el servicio Azure Migrate.
ms.topic: conceptual
ms.date: 04/15/2020
ms.openlocfilehash: 95fd2ea9ecf5e4bcdf3fb2291e4cf7bf5b111c5f
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87847472"
---
# <a name="azure-migrate-common-questions"></a>Azure Migrate: Preguntas frecuentes

En este artículo se responden a preguntas habituales sobre Azure Migrate. Si después de leer este artículo tiene alguna pregunta, puede publicarla en el [ foro de Azure Migrate](https://aka.ms/AzureMigrateForum). También puede revisar estos artículos:

- Preguntas sobre el [dispositivo de Azure Migrate](common-questions-appliance.md)
- Preguntas sobre la [detección, valoración y visualización de dependencias](common-questions-discovery-assessment.md)

## <a name="what-is-azure-migrate"></a>¿Qué es Azure Migrate?

Azure Migrate proporciona un centro de conectividad para realizar la detección, valoración y migración a Azure de las aplicaciones y cargas de trabajo locales, así como de las máquinas virtuales en las nubes privadas o públicas. El centro proporciona herramientas de Azure Migrate para la evaluación y la migración, así como ofertas de ISV de terceros. [Más información](migrate-services-overview.md).

## <a name="what-can-i-do-with-azure-migrate"></a>¿Qué puedo hacer con Azure Migrate?

Use Azure Migrate para descubrir, valorar y migrar la infraestructura, las aplicaciones y los datos locales a Azure. Azure Migrate admite la valoración y migración de máquinas virtuales de VMware locales, máquinas virtuales de Hyper-V, servidores físicos, otras máquinas virtuales virtualizadas, bases de datos, aplicaciones web y escritorios virtuales. 

## <a name="whats-the-difference-between-azure-migrate-and-azure-site-recovery"></a>¿Cuál es la diferencia entre Azure Migrate y Azure Site Recovery?

[Azure Migrate](migrate-services-overview.md) ofrece un centro principal para la valoración y la migración a Azure. 

- El uso de Azure Migrate proporciona interoperabilidad y extensibilidad futura con herramientas de Azure Migrate, otros servicios de Azure y herramientas de terceros.
- La herramienta Azure Migrate:Server Migration está diseñada específicamente para la migración de servidores a Azure. Está optimizada para la migración. No es necesario obtener información sobre los conceptos y escenarios que no son directamente pertinentes para la migración. 
- No hay cargos por uso de herramientas para la migración durante 180 días a partir del momento en que se inicia la replicación de una máquina virtual. Esto le proporciona tiempo para completar la migración. Solo paga por los recursos de almacenamiento y de red que se usan en la replicación, y por los cargos de proceso consumidos durante las migraciones de prueba.
- Azure Migrate admite todos los escenarios de migración que admite Site Recovery. Además, en el caso de máquinas virtuales de VMware, Azure Migrate proporciona una opción de migración sin agente.
- Estamos dando prioridad a las nuevas características de migración solo de la herramienta Azure Migrate:Server Migration. Estas características no están dirigidas a Site Recovery.

[Azure Site Recovery](../site-recovery/site-recovery-overview.md) debe usarse solo para la recuperación ante desastres.

Azure Migrate: La herramienta Server Migration usa cierta funcionalidad de back-end de Site Recovery para la migración mediante lift-and-shift de algunas máquinas locales.

## <a name="i-have-a-project-with-the-previous-classic-experience-of-azure-migrate-how-do-i-start-using-the-new-version"></a>Tengo un proyecto con la experiencia clásica anterior de Azure Migrate. ¿Cómo empiezo a usar la nueva versión?

No se pueden actualizar proyectos ni componentes de la versión anterior a la nueva versión. Tiene que [crear un proyecto de Azure Migrate](create-manage-projects.md) y [agregarle herramientas de valoración y migración](how-to-add-tool-first-time.md). Use los tutoriales para aprender a usar las herramientas de valoración y migración disponibles. Si tiene un área de trabajo Log Analytics asociada a un proyecto clásico, puede adjuntarla a un proyecto de la versión actual después de eliminar el proyecto clásico.

## <a name="whats-the-difference-between-azure-migrate-server-assessment-and-the-map-toolkit"></a>¿Cuál es la diferencia entre Azure Migrate: Server Assessment y MAP Toolkit?

Server Assessment proporciona una valoración para ayudar con la preparación de la migración y la evaluación de las cargas de trabajo para la migración a Azure. [Microsoft Assessment and Planning (MAP) Toolkit](https://www.microsoft.com/download/details.aspx?id=7826) ayuda con otras tareas, como el planeamiento de la migración para versiones más recientes de los sistemas operativos cliente y servidor de Windows y el seguimiento del uso de software. Para esos escenarios, siga usando MAP Toolkit.

## <a name="whats-the-difference-between-server-assessment-and-the-site-recovery-deployment-planner"></a>¿Cuál es la diferencia entre Server Assessment y Site Recovery Deployment Planner?

Server Assessment es una herramienta de planeamiento de la migración. Site Recovery Deployment Planner es una herramienta de planeamiento de recuperación ante desastres.

Elija la herramienta en función de lo que desee hacer:

- **Planeamiento de la migración del entorno local a Azure**: Si planea migrar los servidores locales a Azure, use Server Assessment para planear la migración. Server Assessment evalúa las cargas de trabajo locales y proporciona orientación y herramientas que le ayudarán a migrar. Una vez implementado el plan de migración, puede usar herramientas como Azure Migrate: Server Migration, para migrar las máquinas a Azure.
- **Planeamiento de recuperación ante desastres en Azure**: Si planea configurar la recuperación ante desastres de una ubicación local a Azure con Site Recovery, use Site Recovery Deployment Planner. Deployment Planner realiza una evaluación profunda y específica de Site Recovery de su entorno local con el fin de la recuperación ante desastres. Proporciona recomendaciones relacionadas con la recuperación ante desastres, como la replicación y la conmutación por error.

## <a name="how-does-server-migration-work-with-site-recovery"></a>¿Cómo funciona Server Migration con Site Recovery?

- Si usa Azure Migrate: Server Migration para realizar una migración *sin agente* de máquinas virtuales de VMware locales, la migración es nativa de Azure Migrate y no se utiliza Site Recovery.
- Si usa Azure Migrate: Server Migration para realizar una migración *basada en agentes* de máquinas virtuales de VMware, o bien, si migra máquinas virtuales de Hyper-V o servidores físicos, Azure Migrate: Server Migration usa el motor de replicación de Azure Site Recovery.

## <a name="which-geographies-are-supported"></a>¿Qué zonas geográficas se admiten?

Revise las zonas geográficas admitidas para [nubes públicas](migrate-support-matrix.md#supported-geographies-public-cloud) y [gubernamentales](migrate-support-matrix.md#supported-geographies-azure-government).

## <a name="how-do-i-get-started"></a>¿Cómo empiezo?

Identifique la herramienta que necesita y agréguela a un proyecto de Azure Migrate. 

Para agregar una herramienta de ISV o Movere:

1. Comience con la obtención de una licencia o regístrese para obtener una evaluación gratuita de acuerdo con la directiva de la herramienta. La licencia para las herramientas se conceden de conformidad con el modelo de licencias de la herramienta o de ISV.
2. En cada herramienta, existe una opción para conectarse a Azure Migrate. Siga las instrucciones y la documentación de la herramienta para conectarla con Azure Migrate.

Realice un seguimiento del recorrido de la migración desde el proyecto de Azure Migrate, en Azure y en otras herramientas.

## <a name="how-do-i-delete-a-project"></a>¿Cómo puedo eliminar un proyecto?

Aprenda a [eliminar un proyecto](how-to-delete-project.md). 

## <a name="next-steps"></a>Pasos siguientes

Consulte la sección [Información general de Azure Migrate](migrate-services-overview.md).
