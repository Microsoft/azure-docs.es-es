---
title: Preguntas más frecuentes sobre el dispositivo de Azure Migrate
description: Respuestas a preguntas comunes sobre el dispositivo de Azure Migrate.
ms.topic: conceptual
ms.date: 06/03/2020
ms.openlocfilehash: de34bba40b9200c198f3c07262bd6b7a00b62060
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89050682"
---
# <a name="azure-migrate-appliance-common-questions"></a>Dispositivo de Azure Migrate: Preguntas frecuentes

Este artículo contiene respuestas a algunas preguntas frecuentes sobre el dispositivo de Azure Migrate. Si tiene otras preguntas, repase estos recursos:

- [Preguntas generales](resources-faq.md) sobre Azure Migrate
- Preguntas sobre [detección, valoración y visualización de dependencias](common-questions-discovery-assessment.md)
- Preguntas sobre la [migración de servidores](common-questions-server-migration.md)
- Obtenga respuestas a sus preguntas en el [foro de Azure Migrate](https://aka.ms/AzureMigrateForum)

## <a name="what-is-the-azure-migrate-appliance"></a>¿Qué es el dispositivo de Azure Migrate?

El dispositivo de Azure Migrate es un dispositivo ligero que Azure Migrate: herramienta Server Assessment usa para detectar y evaluar los servidores locales. Azure Migrate: La herramienta Server Assessment también usa el dispositivo para realizar la migración sin agente de máquinas virtuales de VMware locales.

Aquí encontrará más información sobre el dispositivo Azure Migrate:

- El dispositivo se implementa de forma local como una máquina virtual o física.
- El dispositivo detecta máquinas locales y envía continuamente metadatos y datos de rendimiento de las máquinas a Azure Migrate.
- La detección del dispositivo es sin agente. No se instala nada en las máquinas detectadas.

[Obtenga más información](migrate-appliance.md) sobre el dispositivo.

## <a name="how-can-i-deploy-the-appliance"></a>¿Cómo se puede implementar el dispositivo?

El dispositivo se puede implementar así:

- Mediante una plantilla para máquinas virtuales de VMware y máquinas virtuales de Hyper-V (plantilla OVA en el caso de VMware o VHD en el de Hyper-V).
- Si no desea usar una plantilla o está en Azure Government, puede implementar el dispositivo para VMware o Hyper-V mediante un script de PowerShell.
- En el caso de los servidores físicos, el dispositivo siempre se implementa mediante un script.

## <a name="how-does-the-appliance-connect-to-azure"></a>¿Cómo se conecta el dispositivo a Azure?

El dispositivo puede conectarse a través de Internet, o mediante Azure ExpressRoute.

- Para usar Azure ExpressRoute para el tráfico de replicación de Azure Migrate, se requiere del emparejamiento de Microsoft, o de un emparejamiento público existente (el emparejamiento público está en desuso para las creaciones nuevas de ExpressRoute).
- No se admite la replicación a través de Azure ExpressRoute con (solo) el emparejamiento privado habilitado.

Azure ExpressRoute con el emparejamiento de Microsoft configurado es el dominio de enrutamiento recomendado para el tráfico de replicación.

## <a name="does-appliance-analysis-affect-performance"></a>¿Afecta el análisis del dispositivo al rendimiento?

El dispositivo de Azure Migrate genera perfiles de máquinas locales continuamente para medir los datos de rendimiento de la VM. Esta generación de perfiles apenas afecta al rendimiento de las máquinas en las que se realiza.

## <a name="can-i-harden-the-appliance-vm"></a>¿Se puede proteger la máquina virtual del dispositivo?

Cuando usa la plantilla descargada para crear la máquina virtual del dispositivo, puede agregar componentes (por ejemplo, un antivirus) a la plantilla, siempre que las reglas de firewall y de comunicación necesarias para el dispositivo de Azure Migrate no se modifiquen.

## <a name="what-network-connectivity-is-required"></a>¿Qué conectividad de red se necesita?

El dispositivo necesita acceso a las direcciones URL de Azure. [Examine](migrate-appliance.md#url-access) la lista de direcciones URL.

## <a name="what-data-does-the-appliance-collect"></a>¿Qué datos recopila el dispositivo?

Consulte los artículos siguientes para obtener información sobre los datos que el dispositivo Azure Migrate recopila en las máquinas virtuales:

- **Máquina virtual de VMware**: [Revisión](migrate-appliance.md#collected-data---vmware) de los datos recopilados.
- **Máquina virtual de Hyper-V**: [Revisión](migrate-appliance.md#collected-data---hyper-v) de los datos recopilados.

## <a name="how-is-data-stored"></a>¿Cómo se almacenan los datos?

Los datos que recopila el dispositivo de Azure Migrate se almacenan en la ubicación de Azure en la que creó el proyecto de Azure Migrate.

Aquí encontrará más información sobre cómo se almacenan los datos:

- Los datos recopilados se almacenan de forma segura en CosmosDB en una suscripción de Microsoft. Los datos se eliminan cuando se elimina el proyecto de Azure Migrate. Azure Migrate controla el almacenamiento. No se puede elegir una cuenta de almacenamiento especifica para los datos recopilados.
- Si usa la [visualización de dependencias](concepts-dependency-visualization.md), los datos recopilados se almacenan en los Estados Unidos, en un área de trabajo de Log Analytics creada en su suscripción de Azure. Los datos se eliminan al eliminar el área de trabajo de Log Analytics en su suscripción.

## <a name="how-much-data-is-uploaded-during-continuous-profiling"></a>¿Qué cantidad de datos se carga en la generación de perfiles continua?

El volumen de datos que se envía a Azure Migrate depende de varios parámetros. Por ejemplo, un proyecto Azure Migrate que tiene 10 máquinas (cada una con un disco y una NIC) envía aproximadamente 50 MB de datos al día. Este valor es aproximado; el valor real varía en función del número de puntos de datos para los discos y las NIC. Si aumenta el número de máquinas, discos o NIC, el aumento de los datos que se envían es no lineal.

## <a name="is-data-encrypted-at-rest-and-in-transit"></a>¿Los datos se cifran en reposo y en tránsito?

Sí, en ambos:

- Los metadatos se envían de manera segura al servicio Azure Migrate por Internet a través de HTTPS.
- Los metadatos se almacenan en una base de datos de [Azure Cosmos](../cosmos-db/database-encryption-at-rest.md) y en [Azure Blob Storage](../storage/common/storage-service-encryption.md), en una suscripción de Microsoft. Los metadatos se cifran en reposo en el caso del almacenamiento.
- Los datos para el análisis de dependencia se cifran en tránsito (con HTTPS seguro). Se almacenan en un área de trabajo de Log Analytics de la suscripción. Los datos para el análisis de dependencias se cifran en reposo.

## <a name="how-does-the-appliance-connect-to-vcenter-server"></a>¿Cómo se conecta el dispositivo a vCenter Server?

En estos pasos se describe cómo se conecta el dispositivo a VMware vCenter Server:

1. El dispositivo se conecta a vCenter Server (puerto 443) mediante las credenciales que proporcionó al configurar el dispositivo.
2. El dispositivo usa VMware PowerCLI para realizar consultas en vCenter Server, para recopilar metadatos sobre las máquinas virtuales que administra vCenter Server.
3. El dispositivo recopila los datos de configuración sobre máquinas virtuales (núcleos, memoria, discos, NIC) y el historial de rendimiento de cada máquina virtual del último mes.
4. Los metadatos recopilados se envían a Azure Migrate: La herramienta Server Assessment (mediante Internet a través de HTTPS) para la valoración.

## <a name="can-the-azure-migrate-appliance-connect-to-multiple-vcenter-servers"></a>¿Puede conectarse el dispositivo Azure Migrate a varias instancias de vCenter Server?

No. Hay una asignación unívoca entre un [dispositivo Azure Migrate](migrate-appliance.md) y vCenter Server. Para detectar máquinas virtuales en varias instancias de vCenter Server, tiene que implementar varios dispositivos. 

## <a name="can-an-azure-migrate-project-have-multiple-appliances"></a>¿Puede un proyecto Azure Migrate tener varios dispositivos?

Un proyecto puede tener varios dispositivos conectados. Sin embargo, un dispositivo solo se puede asociar a un proyecto. 

## <a name="can-the-azure-migrate-appliancereplication-appliance-connect-to-the-same-vcenter"></a>¿El dispositivo de Azure Migrate o el dispositivo de replicación puede conectarse al mismo vCenter?

Sí. Puede agregar el dispositivo de Azure Migrate (que se usa para la evaluación y la migración de VMware sin agente) y el dispositivo de replicación (que se usa para la migración basada en agente de máquinas virtuales de VMware) al mismo servidor vCenter.


## <a name="how-many-vms-or-servers-can-i-discover-with-an-appliance"></a>¿Cuántas máquinas virtuales o servidores puedo detectar con un dispositivo?

Puede detectar hasta 10 000 VM de VMware, 5000 de Hyper-V y 1000 servidores físicos con un único dispositivo. Si tiene más máquinas en su entorno local, lea acerca de [escalar una valoración de Hyper-V](scale-hyper-v-assessment.md), [escalar una valoración de VMware](scale-vmware-assessment.md) y [escalar una evaluación de un servidor físico](scale-physical-assessment.md).

## <a name="can-i-delete-an-appliance"></a>¿Puedo eliminar un dispositivo?

Actualmente no se admite la eliminación de un dispositivo del proyecto.

La única manera de eliminar el dispositivo es eliminar el grupo de recursos que contiene el proyecto de Azure Migrate que está asociado con el dispositivo.

Si bien, al eliminar el grupo de recursos también se eliminarán otros dispositivos registrados, el inventario detectado, las valoraciones y todos los demás componentes de Azure en el grupo de recursos que estén asociados al proyecto.

## <a name="can-i-use-the-appliance-with-a-different-subscription-or-project"></a>¿Puedo usar el dispositivo con una suscripción o un proyecto distintos?

Después de usar el dispositivo para iniciar la detección, no puede volver a configurarlo con otra suscripción de Azure, ni puede usarlo en otro proyecto de Azure Migrate. Tampoco puede detectar máquinas virtuales en otra instancia de vCenter Server. Configure un dispositivo nuevo para estas tareas.

## <a name="can-i-set-up-the-appliance-on-an-azure-vm"></a>¿Se puede configurar el dispositivo en una máquina virtual de Azure?

No. Actualmente, esta opción no es compatible. 

## <a name="can-i-discover-on-an-esxi-host"></a>¿Se puede detectar en un host ESXi?

No. Para detectar máquinas virtuales de VMware, tiene que tener vCenter Server.

## <a name="how-do-i-update-the-appliance"></a>¿Cómo se actualiza el dispositivo?

De forma predeterminada, el dispositivo y sus agentes instalados se actualizan automáticamente. El dispositivo busca actualizaciones cada 24 horas. Las actualizaciones que resultan en error se vuelven a intentar. 

Estas actualizaciones automáticas solo actualizan los agentes del dispositivo y el dispositivo. Las actualizaciones automáticas de Azure Migrate no actualizan el sistema operativo. Use las actualizaciones de Windows para mantener actualizado el sistema operativo.

## <a name="can-i-check-agent-health"></a>¿Puedo comprobar el estado del agente?

Sí. En el portal, vaya a la página **Agent Health** de Azure Migrate: Server Assessment o Azure Migrate: Herramienta de migración del servidor. Allí puede comprobar el estado de la conexión de los agentes de detección y valoración del dispositivo con Azure.

## <a name="next-steps"></a>Pasos siguientes

Consulte la sección [Información general de Azure Migrate](migrate-services-overview.md).
