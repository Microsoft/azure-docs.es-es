---
title: Novedades de Azure Migrate
description: Obtenga información sobre las novedades y actualizaciones recientes del servicio Azure Migrate.
ms.topic: overview
ms.date: 04/19/2020
ms.custom: mvc
ms.openlocfilehash: 019c04c5820a82b7910b6755ff69e5dfbe7eb66c
ms.sourcegitcommit: 9c262672c388440810464bb7f8bcc9a5c48fa326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89421618"
---
# <a name="whats-new-in-azure-migrate"></a>Novedades de Azure Migrate

[Azure Migrate](migrate-services-overview.md) le ayuda a detectar, evaluar y migrar servidores, aplicaciones y datos locales a la nube de Microsoft Azure. En este artículo se resumen las nuevas versiones y características de Azure Migrate.
## <a name="update-september-2020"></a>Actualización (septiembre de 2020)
- Azure Migrate ahora permite migrar servidores a Availability Zones

## <a name="update-august-2020"></a>Actualización (agosto de 2020)

- Mejor experiencia de incorporación, en la que se genera una clave de proyecto de Azure Migrate desde el portal y se usa para completar el registro del dispositivo.
- Opción para descargar archivos OVA/VHD o los scripts del instalador desde el portal para configurar los dispositivos de VMware y Hyper-V, respectivamente.
- Se ha actualizado el administrador de configuración de aplicaciones con una experiencia de usuario mejorada.
- Compatibilidad con múltiples credenciales para la detección de máquinas virtuales con Hyper-V.
- Se han mejorado las funcionalidades de búsqueda, ordenación y filtrado para las credenciales agregadas y los orígenes de detección.
- Entrada de un solo elemento, entrada de varios elementos e importación de opciones de CSV para que el usuario agregue orígenes de detección para hosts o clústeres de Hyper-V y servidores físicos.
- Se ha mejorado la gestión de los errores mejorada con actualizaciones del estado para las operaciones de validación y detección en cada origen agregado de la tabla. 

## <a name="update-june-2020"></a>Actualización (junio de 2020)

- Ahora se admiten las evaluaciones para migrar VM de VMware locales a [Azure VMware Solution (AVS)](https://go.microsoft.com/fwlink/?linkid=2132637). [Más información](how-to-create-azure-vmware-solution-assessment.md)
- Compatibilidad con varias credenciales en el dispositivo para la detección de servidores físicos.
- Compatibilidad para permitir el inicio de sesión de Azure desde el dispositivo para el inquilino en el que se ha configurado la restricción de inquilino.


## <a name="update-april-2020"></a>Actualización (abril de 2020)

Azure Migrate admite implementaciones en Azure Government. 

- Puede descubrir y evaluar máquinas virtuales de VMware, máquinas virtuales de Hyper-V y servidores físicos.
- Puede migrar máquinas virtuales de VMware locales, máquinas virtuales Hyper-V y servidores físicos a Azure.
- En el caso de VMware, puede realizar una migración con agente o sin agente. [Más información](server-migrate-overview.md).
- [Examine](migrate-support-matrix.md#supported-geographies-azure-government) regiones y zonas geográficas compatibles para obtener Azure Government.
- El [análisis de dependencias basado en agente](concepts-dependency-visualization.md#agent-based-analysis) no está disponible en Azure Government.
- Las características en versión preliminar se admiten en Azure Government, concretamente el [análisis de dependencias sin agente ](concepts-dependency-visualization.md#agentless-analysis) y la [detección de aplicaciones](how-to-discover-applications.md).


## <a name="update-march-2020"></a>Actualización (marzo de 2020)

Ahora hay disponible una instalación basada en scripts para configurar el [dispositivo de Azure Migrate](migrate-appliance.md):

- La instalación basada en script es una alternativa a la instalación .OVA (VMware)/VHD (Hyper-V) del dispositivo.
- Proporciona un script de instalador de PowerShell que se puede usar para configurar el dispositivo para VMware/Hyper-V en una máquina existente que ejecute Windows Server 2016.

## <a name="update-november-2019"></a>Actualización (noviembre de 2019)

Se han agregado varias características nuevas a Azure Migrate:

- **Evaluación de servidores físicos**. Ahora se admite la evaluación de servidores físicos locales, además de la migración de servidores físicos que ya se admite.
- **Evaluación basada en la importación**. Ahora se admite la evaluación de máquinas con metadatos y datos de rendimiento proporcionados en un archivo CSV.
- **Detección de aplicaciones**: Azure Migrate admite ahora la detección en el nivel de aplicación de aplicaciones, roles y características mediante el uso del dispositivo Azure Migrate. Actualmente solo se admite para máquinas virtuales de VMware y solo se limita a la detección (actualmente no se admite la evaluación). [Más información](how-to-discover-applications.md)
- **Visualización de dependencias sin agentes**: Ya no necesita instalar agentes explícitamente para la visualización de dependencias. Ahora se admiten tanto sin agente como basados en agente.
- **Virtual Desktop**: Use herramientas de ISV para evaluar y migrar la infraestructura de escritorio virtual (VDI) local a Windows Virtual Desktop en Azure.
- **Aplicación web**: Migration Assistant de Azure App Service, que se usa para evaluar y migrar aplicaciones web, ahora se integra en Azure Migrate.

Se han agregado a Azure Migrate nuevas herramientas para la evaluación y la migración:

- **Rackware**: Migración a la nube de ofertas.
- **Movere**: Evaluación de ofertas.

[Más información](migrate-services-overview.md) sobre el uso de herramientas y ofertas de ISV para la evaluación y la migración en Azure Migrate.

## <a name="azure-migrate-current-version"></a>Versión actual de Azure Migrate

La versión actual de Azure Migrate (publicada en julio de 2019) proporciona varias características nuevas:

- **Plataforma de migración unificada**: Ahora, Azure Migrate ofrece un único portal para centralizar, administrar y realizar el seguimiento del proceso de migración a Azure, con un flujo de implementación mejorado y una experiencia de portal.
- **Herramientas de evaluación y migración**: Azure Migrate proporciona herramientas nativas y se integra con otros servicios de Azure, así como con herramientas de fabricante de software independiente (ISV). [Más información](migrate-services-overview.md#isv-integration) acerca de la integración de ISV.
- **Evaluación de Azure Migrate**: Puede usar la herramienta de evaluación de servidores de Azure Migrate para evaluar VM de VMware y de Hyper-V para la migración a Azure. También puede evaluar la migración con otros servicios de Azure y herramientas de ISV.
- **Migración de Azure Migrate**: La herramienta de migración de servidores de Azure Migrate permite migrar VM de VMware locales y VM de Hyper-V a Azure, así como servidores físicos, otros servidores virtualizados y VM de nube privada o pública. Además, puede migrar a Azure mediante herramientas de ISV.
- **Dispositivo con Azure Migrate**: Azure Migrate implementa un dispositivo ligero para la detección y evaluación de VM de VMware locales y VM de Hyper-V.
    - La evaluación y la migración de servidores de Azure Migrate usan este dispositivo para realizar una migración sin agente.
    - El dispositivo detecta continuamente los metadatos de servidor y los datos de rendimiento con fines de evaluación y migración.  
- **Migración de VM de VMware**:  La migración de servidores de Azure Migrate proporciona un par de métodos para migrar VM de VMware locales a Azure.  Una migración sin agente mediante el dispositivo de Azure Migrate y una migración basada en agente que usa un dispositivo de replicación e implementa un agente en cada VM que quiere migrar. [Más información](server-migrate-overview.md)
 - **Valoración y migración de bases de datos**: desde Azure Migrate, puede evaluar las bases de datos locales para la migración a Azure mediante Azure Database Migration Assistant. Puede migrar bases de datos con Azure Database Migration Service.
- **Migración de aplicaciones web**: puede evaluar las aplicaciones web con una dirección URL de punto de conexión pública mediante Azure App Service. Para la migración de aplicaciones de .NET internas, puede descargar y ejecutar App Service Migration Assistant.
- **Data Box**: importe grandes cantidades de datos sin conexión a Azure mediante Azure Data Box en Azure Migrate.

## <a name="azure-migrate-previous-version"></a>Versión anterior de Azure Migrate

Si usaba la versión anterior de Azure Migrate (solo se admitía la valoración de máquinas virtuales de VMware locales), ahora debería usar la versión actual. En la versión anterior, ya no puede crear nuevos proyectos de Azure Migrate ni realizar nuevas detecciones. Todavía puede obtener acceso a los proyectos existentes. Para hacerlo, en Azure Portal > **Todos los servicios**, busque **Azure Migrate**. En las notificaciones de Azure Migrate, hay un vínculo para acceder a antiguos proyectos de Azure Migrate.



## <a name="next-steps"></a>Pasos siguientes

- [Más información](https://azure.microsoft.com/pricing/details/azure-migrate/) sobre los precios de Azure Migrate.
- [Repase las preguntas más frecuentes](resources-faq.md) sobre Azure Migrate.
- Pruebe nuestros tutoriales para evaluar las [VM de VMware](tutorial-assess-vmware.md) y [las VM de Hyper-V](tutorial-assess-hyper-v.md).
