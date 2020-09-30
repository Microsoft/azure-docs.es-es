---
title: Regiones y Availability Zones en Azure
description: Obtenga información sobre las regiones y Availability Zones en Azure para satisfacer sus requisitos técnicos y normativos.
author: cynthn
ms.service: azure
ms.topic: article
ms.date: 08/27/2020
ms.author: cynthn
ms.custom: fasttrack-edit, mvc
ms.openlocfilehash: b9a400862280cbbc219e87695e363c0ec2e75ee6
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018678"
---
# <a name="regions-and-availability-zones-in-azure"></a>Regiones y Availability Zones en Azure

Los servicios de Microsoft Azure están disponibles globalmente para impulsar las operaciones en la nube a un nivel óptimo. Puede elegir la mejor región para sus necesidades en función de las consideraciones técnicas y normativas: capacidades de servicio, residencia de datos, requisitos de cumplimiento y latencia.

## <a name="terminology"></a>Terminología

Para conocer mejor las regiones y Availability Zones en Azure, le resultará más fácil comprender los términos o conceptos clave.

| Término o concepto | Descripción |
| --- | --- |
| region | Un conjunto de centros de datos implementados dentro de un perímetro que define la latencia y conectados a través de una red regional dedicada de baja latencia. |
| geography | Un área del mundo que contiene al menos una región de Azure. Las zonas geográficas definen un mercado discreto que conserva los límites de residencia de datos y cumplimiento. Las zonas geográficas permiten a los clientes con necesidades específicas de residencia de datos y cumplimiento normativo mantener sus datos y aplicaciones cerca. Las zonas geográficas son tolerantes a errores hasta el punto de resistir una interrupción total del funcionamiento de una región gracias a su conexión con nuestra infraestructura de red dedicada de alta capacidad. |
| Zona de disponibilidad | Ubicaciones físicas exclusivas dentro de una región. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. |
| Región recomendada | Una región que proporciona la gama más amplia de capacidades de servicio que está diseñada para admitir Availability Zones ahora o en el futuro. Se designan en Azure Portal como **Recomendado**. |
| Región alternativa (otra) | Una región que extiende la superficie de Azure dentro de un límite de residencia de datos donde también existe una región recomendada. Las regiones alternativas ayudan a optimizar la latencia y proporcionan una segunda región para las necesidades de recuperación ante desastres. No están diseñadas para admitir Availability Zones (aunque Azure realiza una evaluación periódica de estas regiones para determinar si deben convertirse en regiones recomendadas). Se designan en Azure Portal como **Otras**. |
| Servicio fundamental | Un servicio básico de Azure que está disponible en todas las regiones cuando la región está disponible con carácter general. |
| Servicio estándar | Un servicio de Azure que está disponible en todas las regiones recomendadas en un plazo de 12 meses a partir de la disponibilidad general de la región o del servicio o la disponibilidad basada en la demanda en regiones alternativas. |
| Servicio especializado | Un servicio de Azure que ofrece una disponibilidad basada en la demanda entre regiones respaldadas por hardware personalizado o especializado. |
| Servicio regional | Un servicio de Azure que se implementa de forma regional y permite al cliente especificar la región en la que se implementará el servicio. Para obtener una lista completa, vea [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=all). |
| Servicio no regional | Un servicio de Azure para el que no hay ninguna dependencia en una región específica de Azure. Los servicios no regionales se implementan en dos o más regiones y, si hay un error regional, la instancia del servicio en otra región sigue atendiendo a los clientes. Para obtener una lista completa, vea [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=all). |

## <a name="regions"></a>Regions

Una región es un conjunto de centros de datos implementados dentro de un perímetro definido por la latencia y conectados a través de una red regional dedicada de baja latencia. Azure ofrece la flexibilidad de implementar aplicaciones donde sea necesario, incluso en varias regiones para ofrecer una resistencia entre regiones. Para más información, vea [Información general sobre el pilar de resistencia](https://docs.microsoft.com/azure/architecture/framework/resiliency/overview).

## <a name="availability-zones"></a>Zonas de disponibilidad

Una zona de disponibilidad constituye una oferta de alta disponibilidad que protege las aplicaciones y los datos de los errores en el centro de datos. Las zonas de disponibilidad son ubicaciones físicas exclusivas dentro de una región de Azure. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. Para garantizar la resistencia, hay un mínimo de tres zonas independientes en todas las regiones habilitadas. La separación física de las zonas de disponibilidad dentro de una región protege las aplicaciones y los datos frente a los errores del centro de datos. Los servicios con redundancia de zona replican las aplicaciones y los datos entre zonas de disponibilidad para protegerlos frente a puntos de error únicos. Con las zonas de disponibilidad, Azure ofrece el mejor Acuerdo de Nivel de Servicio del sector de tiempo de actividad de máquina virtual, con un 99,99 %. En el [SLA de Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/) completo se explica la disponibilidad garantizada de Azure como un conjunto.

Una zona de disponibilidad de una región de Azure es una combinación de un dominio de error y un dominio de actualización. Por ejemplo, si crea tres o más máquinas virtuales en tres zonas de una región de Azure, las máquinas virtuales se distribuyen eficazmente en tres dominios de error y tres dominios de actualización. La plataforma Azure reconoce esta distribución entre dominios de actualización para asegurarse de que las máquinas virtuales de distintas zonas se programan para actualizarse a la vez.

Coloque sus recursos de proceso, almacenamiento, red y datos dentro de una zona y replíquelos en otras para conseguir una alta disponibilidad en la arquitectura de sus aplicaciones. Los servicios de Azure que admiten zonas de disponibilidad se dividen en dos categorías:

- **Servicios de zona**: donde un recurso se ancla a una zona específica (por ejemplo, máquinas virtuales, discos administrados o direcciones IP estándar) o
- **Servicios con redundancia de zona**: cuando la plataforma de Azure se replica automáticamente entre zonas (por ejemplo, almacenamiento con redundancia de zona y SQL Database).

Para lograr una continuidad del negocio integral en Azure, cree la arquitectura de aplicación mediante la combinación de zonas de disponibilidad y pares de regiones de Azure. Puede replicar de forma sincrónica las aplicaciones y los datos mediante zonas de disponibilidad dentro de una región de Azure para conseguir alta disponibilidad, y replicar de forma asincrónica entre regiones de Azure para la protección de recuperación ante desastres.
 
![vista conceptual de una zona fuera de servicio en una región](./media/az-overview/az-graphic-two.png)

> [!IMPORTANT]
> Los identificadores de la zona de disponibilidad (los números 1, 2 y 3 de la imagen anterior) se asignan lógicamente a las zonas físicas reales para cada suscripción de manera independiente. Esto significa que la zona de disponibilidad 1 en una suscripción determinada podría hacer referencia a una zona física diferente de la zona de disponibilidad 1 en una suscripción diferente. En consecuencia, se recomienda no confiar en identificadores de zona de disponibilidad de diferentes suscripciones para la selección de máquina virtual.

## <a name="region-and-service-categories"></a>Categorías de región y servicio

El enfoque de Azure sobre la disponibilidad de los servicios de Azure entre regiones se describe mejor al expresar los servicios disponibles en las regiones recomendadas y alternativas.

- **Región recomendada**: una región que proporciona la gama más amplia de capacidades de servicio que está diseñada para admitir Availability Zones ahora o en el futuro. Se designan en Azure Portal como **Recomendado**.
- **Región alternativa (otra)** : una región que extiende la superficie de Azure dentro de un límite de residencia de datos donde también existe una región recomendada. Las regiones alternativas ayudan a optimizar la latencia y proporcionan una segunda región para las necesidades de recuperación ante desastres. No están diseñadas para admitir Availability Zones (aunque Azure realiza una evaluación periódica de estas regiones para determinar si deben convertirse en regiones recomendadas). Se designan en Azure Portal como **Otras**.

### <a name="comparing-region-types"></a>Comparación de tipos de regiones

Los servicios de Azure se agrupan en tres categorías: servicios fundamentales, estándar y especializados. La directiva general de Azure sobre la implementación de servicios en una región determinada se basa principalmente en el tipo de región, las categorías de servicio y la demanda del cliente:

- **Fundamentales**: están disponibles en todas las regiones recomendadas y alternativas cuando la región está disponible con carácter general, o en un plazo de 12 meses después de que un nuevo servicio fundamental esté disponible con carácter general.
- **Estándar**: disponibles en todas las regiones recomendadas en un plazo de 12 meses a partir de la disponibilidad general de la región o del servicio; se basan en la demanda en las regiones alternativas (muchos ya están implementados en un gran subconjunto de regiones alternativas).
- **Especializados**: ofertas de servicio dirigidas, a menudo centradas en el sector o respaldadas por hardware personalizado o especializado. Disponibilidad basada en la demanda entre regiones (muchos ya están implementados en un gran subconjunto de regiones recomendadas).

Para ver los servicios que se implementan en una región determinada, así como la futura hoja de ruta de la versión preliminar o la disponibilidad general de los servicios de una región, vea [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=all).

Si una oferta de servicio no está disponible en una región específica, puede compartir su interés poniéndose en contacto con su representante de ventas de Microsoft.

| Tipo de región | No regional | Fundamental | Estándar | Especializada | Zonas de disponibilidad | Residencia de datos |
| --- | --- | --- | --- | --- | --- | --- |
| Recomendado | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | Basado en la demanda | :heavy_check_mark: | :heavy_check_mark: |
| Alternativa | :heavy_check_mark: | :heavy_check_mark: | Basado en la demanda | Basado en la demanda | N/D | :heavy_check_mark: |

### <a name="services-by-category"></a>Servicios por categoría

Como se mencionó anteriormente, Azure clasifica los servicios en tres categorías: fundamental, estándar y especializado. Las categorías de servicio se asignan en la disponibilidad general. A menudo, los servicios inician su ciclo de vida como un servicio especializado y, a medida que aumenta la demanda y el uso, se pueden promocionar a estándar o fundamental. En la tabla siguiente se muestra la categoría de los servicios como fundamental, estándar o especializado. Se debe tener en cuenta lo siguiente sobre la tabla:

- Algunos servicios no son regionales. Para obtener información y una lista de servicios no regionales, vea [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/).
- No se muestran las máquinas virtuales de las generaciones anteriores. Para más información, vea la documentación en [Generaciones anteriores de tamaños de máquina virtual](../virtual-machines/sizes-previous-gen.md).

> [!div class="mx-tableFixed"]
> | Fundamental | Estándar | Especializada |
> | --- | --- | --- |
> | Almacenamiento de cuenta | API Management | API de Azure para FHIR |
> | Application Gateway | Configuración de la aplicación | Azure Analysis Services |
> | Azure Backup | App Service | Servicio Azure Blockchain |
> | Azure Cosmos DB | Automation | Azure Blueprints |
> | Azure Data Lake Storage Gen2 | Azure Active Directory Domain Services | Azure Database for MariaDB |
> | Azure ExpressRoute | Azure Bastion | Azure Dedicated HSM |
> | Azure SQL Database | Azure Cache for Redis | Azure Dev Spaces |
> | Cloud Services | Azure Cognitive Search | Azure Digital Twins |
> | Cloud Services: Serie Av2 | Explorador de datos de Azure | Azure Lab Services |
> | Cloud Services: Serie Dv2 | Azure Data Share | Azure NetApp Files |
> | Cloud Services: Serie Dv3 | Azure Database for MySQL | Azure Quantum |
> | Cloud Services: Serie Ev3 | Azure Database for PostgreSQL | Servicio Azure Spring Cloud |
> | Cloud Services: Direcciones IP de nivel de instancia | Azure Database Migration Service | Azure Time Series Insights |
> | Cloud Services: IP reservada | Azure Databricks | Azure VMware Solution by CloudSimple |
> | Disk Storage | Azure DDoS Protection | Cloud Services: Serie G |
> | Event Hubs | Azure DevTest Labs | Cloud Services: Serie H |
> | Key Vault | Azure Firewall | Cognitive Services: Custom Vision |
> | Equilibrador de carga | Azure Firewall Manager | Cognitive Services: Speaker Recognition |
> | Azure Service Bus | Azure Functions | Data Box Heavy |
> | Service Fabric | Azure HPC Cache | Data Catalog |
> | Virtual Machine Scale Sets | Azure IoT Hub | Data Factory: Data Factory V1 |
> | Virtual Machines | Azure Kubernetes Service (AKS) | Data Lake Analytics |
> | Virtual Machines: Serie Av2 | Azure Machine Learning | Machine Learning Studio |
> | Virtual Machines: Serie Bs | Azure Private Link | Microsoft Genomics |
> | Virtual Machines: Serie DSv2 | Red Hat OpenShift en Azure | Remote Rendering |
> | Virtual Machines: Serie DSv3 | Servicio Azure SignalR | Spatial Anchors |
> | Virtual Machines: Serie Dv2 | Azure Site Recovery | StorSimple |
> | Virtual Machines: Serie Dv3 | Azure Stack Hub | Video Indexer |
> | Virtual Machines: Serie ESv3 | Azure Stream Analytics | Virtual Machines: Serie DASv4 |
> | Virtual Machines: Serie Ev3 | Azure Synapse Analytics | Virtual Machines: Serie DAv4 |
> | Virtual Machines: Serie F | Batch | Virtual Machines: Serie DCsv2 |
> | Virtual Machines: Serie FS | Cloud Services: Serie M | Virtual Machines: Serie EASv4 |
> | Virtual Machines: Direcciones IP de nivel de instancia | Cognitive Services | Virtual Machines: Serie EAv4 |
> | Virtual Machines: IP reservada | Cognitive Services: Computer Vision | Virtual Machines: Serie G |
> | Virtual Network | Cognitive Services: Content Moderator | Virtual Machines: Serie GS |
> | VPN Gateway | Cognitive Services: Caras | Virtual Machines: Serie HBv1 |
> |  | Cognitive Services: Form Recognizer | Virtual Machines: Serie HBv2 |
> |  | Cognitive Services: Language Understanding | Virtual Machines: Serie HCv1 |
> |  | Cognitive Services: QnA Maker | Virtual Machines: Serie H |
> |  | Cognitive Services: Speech Services | Virtual Machines: Serie LS |
> |  | Azure Container Instances | Virtual Machines: Serie LSv2 |
> |  | Container Registry | Virtual Machines: Serie Mv2 |
> |  | Data Factory | Virtual Machines: Serie NC |
> |  | Event Grid | Virtual Machines: Serie NCv2 |
> |  | HDInsight | Virtual Machines: Serie NCv3 |
> |  | Logic Apps | Virtual Machines: Serie NDs |
> |  | Media Services | Virtual Machines: Serie NDv2 |
> |  | Network Watcher | Virtual Machines: Serie NV |
> |  | Notification Hubs | Virtual Machines: Serie NVv3 |
> |  | Power BI Embedded | Virtual Machines: Serie NVv4 |
> |  | Premium Blob Storage | Virtual Machines: SAP HANA en Azure (instancias grandes) |
> |  | Premium Files Storage | Visual Studio App Center |
> |  | Almacenamiento: Archive Storage |  |
> |  | Almacenamiento en disco Ultra |  |
> |  | Virtual Machines: Serie Ddsv4 |  |
> |  | Virtual Machines: Serie Ddv4 |  |
> |  | Virtual Machines: Serie Dsv4 |  |
> |  | Virtual Machines: Serie Dv4 |  |
> |  | Virtual Machines: Serie Edsv4 |  |
> |  | Virtual Machines: Serie Edv4 |  |
> |  | Virtual Machines: Serie Esv4 |  |
> |  | Virtual Machines: Serie Ev4 |  |
> |  | Virtual Machines: Serie Fsv2 |  |
> |  | Virtual Machines: Serie M |  |
> |  | Red WAN virtual |  |

###  <a name="services-resiliency"></a>Resistencia de servicios

Todos los servicios de administración de Azure están diseñados para ser resistentes ante errores en el nivel de región. Dentro del espectro de errores, uno o varios errores de zona de disponibilidad en una región tienen un radio de error menor en comparación con un error de una región completa. Azure puede recuperarse de un error en un nivel de zona de los servicios de administración dentro de la región o desde otra región de Azure. Azure realiza el mantenimiento crítico zona por zona en una región, para evitar que los errores afecten a los recursos del cliente implementados en las zonas de disponibilidad de una región.

### <a name="pricing-for-vms-in-availability-zones"></a>Precios de máquinas virtuales en Availability Zones

No hay ningún costo adicional asociado a las máquinas virtuales implementadas en una zona de disponibilidad. Cuando dos o más máquinas virtuales se implementan en dos o más zonas de disponibilidad dentro de una región de Azure, se ofrece un Acuerdo de Nivel de Servicio de tiempo de actividad de máquina virtual del 99,99 %. Habrá gastos adicionales de transferencia de datos de máquina virtual a máquina virtual entre zonas de disponibilidad. Para más información, revise la página [Precios de ancho de banda](https://azure.microsoft.com/pricing/details/bandwidth/).

### <a name="get-started-with-availability-zones"></a>Introducción a las zonas de disponibilidad

- [Creación de una máquina virtual](../virtual-machines/windows/create-portal-availability-zone.md)
- [Agregación de un disco administrado mediante PowerShell](../virtual-machines/windows/attach-disk-ps.md#add-an-empty-data-disk-to-a-virtual-machine)
- [Creación de un conjunto de escalado de máquinas virtuales con redundancia de zona](../virtual-machine-scale-sets/virtual-machine-scale-sets-use-availability-zones.md)
- [Equilibrio de carga de máquinas virtuales en distintas zonas con un equilibrador de carga estándar con un front-end con redundancia de zona](../load-balancer/load-balancer-standard-public-zone-redundant-cli.md)
- [Equilibrio de carga de máquinas virtuales dentro de una zona con un equilibrador de carga estándar con un front-end de zona](../load-balancer/load-balancer-standard-public-zonal-cli.md)
- [Almacenamiento con redundancia de zona](../storage/common/storage-redundancy-zrs.md)
- [SQL Database](../azure-sql/database/high-availability-sla.md#zone-redundant-configuration)
- [Recuperación ante desastres geográfica de Event Hubs](../event-hubs/event-hubs-geo-dr.md#availability-zones)
- [Recuperación ante desastres geográfica de Service Bus](../service-bus-messaging/service-bus-geo-dr.md#availability-zones)
- [Crear una puerta de enlace de red virtual con redundancia de zona](../vpn-gateway/create-zone-redundant-vnet-gateway.md)
- [Adición de una región con redundancia de zona para Azure Cosmos DB](../cosmos-db/high-availability.md#availability-zone-support)
- [Introducción a las zonas de disponibilidad de Azure Cache for Redis](https://aka.ms/redis/az/getstarted)
- [Creación de una instancia de Azure Active Directory Domain Services](../active-directory-domain-services/tutorial-create-instance.md)
- [Creación de un clúster de Azure Kubernetes Service (AKS) que use zonas de disponibilidad](../aks/availability-zones.md)

## <a name="next-steps"></a>Pasos siguientes

- [Regiones que admiten Availability Zones en Azure](az-region.md)
- [Plantillas de inicio rápido](https://aka.ms/azqs)
