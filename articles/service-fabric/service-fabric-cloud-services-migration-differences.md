---
title: Diferencias entre Cloud Services y Service Fabric
description: Introducción conceptual a la migración de aplicaciones desde Cloud Services a Service Fabric.
author: vturecek
ms.topic: conceptual
ms.date: 11/02/2017
ms.author: vturecek
ms.openlocfilehash: c98aeaff3ba39a28fad68454d76f6f4d33f44e5d
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836741"
---
# <a name="learn-about-the-differences-between-cloud-services-and-service-fabric-before-migrating-applications"></a>Obtenga información acerca de las diferencias entre Cloud Services y Service Fabric antes de migrar las aplicaciones.
Microsoft Azure Service Fabric es la plataforma de aplicaciones en la nube de última generación para aplicaciones distribuidas altamente escalables y altamente confiables. Presenta muchas características nuevas para empaquetar, implementar, actualizar y administrar aplicaciones distribuidas en la nube. 

Se trata de una guía de introducción a la migración de aplicaciones desde Cloud Services a Service Fabric. Se centra principalmente en las diferencias de diseño y arquitectura entre Cloud Services y Service Fabric.

## <a name="applications-and-infrastructure"></a>Infraestructura y aplicaciones
Una diferencia fundamental entre Cloud Services y Service Fabric es la relación entre las máquinas virtuales, las cargas de trabajo y aplicaciones. Aquí, una carga de trabajo se define como el código que escribe para realizar una tarea específica o proporcionar un servicio.

* **Cloud Services tiene que ver con la implementación de aplicaciones como máquinas virtuales.** El código que escriba está estrechamente asociado a una instancia de máquina virtual, como un rol de trabajo o un rol web. Implementar una carga de trabajo en Cloud Services es implementar una o más instancias de máquina virtual que ejecutan la carga de trabajo. No hay ninguna separación de las aplicaciones y las máquinas virtuales, por lo que no hay ninguna definición formal de una aplicación. Una aplicación puede considerarse un conjunto de instancias de rol de trabajo o web dentro de una implementación de Cloud Services o como una implementación completa de Cloud Services. En este ejemplo, se muestra una aplicación como un conjunto de instancias de rol.

![Aplicaciones y topología de Cloud Services][1]

* **Service Fabric se usa para implementar aplicaciones en máquinas virtuales existentes o máquinas que ejecutan Service Fabric en Windows o Linux.** Los servicios que escriba estarán completamente desacoplados de la infraestructura subyacente, que se abstrae mediante la plataforma de aplicaciones de Service Fabric, por lo que una aplicación se puede implementar en varios entornos. Una carga de trabajo en Service Fabric se denomina un "servicio", y uno o más servicios se agrupan en una aplicación definida formalmente, que se ejecuta en la plataforma de aplicaciones de Service Fabric. Varias aplicaciones se pueden implementar en un único clúster de Service Fabric.

![Aplicaciones y topología de Service Fabric][2]

Service Fabric en sí es una capa de la plataforma de aplicaciones que se ejecuta en Windows o Linux, mientras que Cloud Services es un sistema para la implementación de máquinas virtuales administradas de Azure con cargas de trabajo asociadas.
El modelo de aplicación de Service Fabric tiene varias ventajas:

* Tiempos de implementación rápidos. La creación de instancias de máquina virtual puede llevar mucho tiempo. En Service Fabric, las máquinas virtuales se implementan solo una vez para formar un clúster que hospeda la plataforma de aplicaciones Service Fabric. Desde ese momento, los paquetes de aplicación pueden implementarse rápidamente en el clúster.
* Hospedaje de alta densidad. En Cloud Services, una máquina virtual de rol de trabajo hospeda una carga de trabajo. En Service Fabric, las aplicaciones son independientes de las máquinas virtuales que las ejecutan, lo que significa que puede implementar un gran número de aplicaciones en un número pequeño de máquinas virtuales, y reducir el costo general en implementaciones de mayor tamaño.
* La plataforma Service Fabric puede ejecutarse en cualquier parte que tenga equipos de Windows Server o Linux, ya sea en Azure o localmente. La plataforma proporciona una capa de abstracción sobre la infraestructura subyacente, por lo que la aplicación puede ejecutarse en distintos entornos. 
* Administración de aplicaciones distribuidas. Service Fabric es una plataforma que no solo hospeda aplicaciones distribuidas, también ayuda a administrar su ciclo de vida independientemente de la máquina virtual host o del ciclo de vida de la máquina.

## <a name="application-architecture"></a>Arquitectura de la aplicación
La arquitectura de una aplicación de Servicios de nube generalmente incluye numerosas dependencias de servicios externos, como Service Bus, Azure Table y Blob Storage, SQL, Redis y otros, para administrar el estado y los datos de una aplicación y la comunicación entre los roles web y los roles de trabajo en una implementación de Cloud Services. Un ejemplo de una aplicación de Cloud Services completa podría tener este aspecto:  

![Arquitectura de Cloud Services][9]

Las aplicaciones de Service Fabric también pueden usar los mismos servicios externos en una aplicación completa. Con este ejemplo de arquitectura de Cloud Services, la ruta de migración más sencilla desde Cloud Services a Service Fabric es reemplazar solo la implementación de Cloud Services por una aplicación de Service Fabric y mantener la arquitectura general de la misma. Los roles web y de trabajo se pueden portar a servicios sin estado de Service Fabric con cambios mínimos en el código.

![Arquitectura de Service Fabric después de una migración simple][10]

En esta etapa, el sistema funcionará igual que antes. Aprovechando las características con estado de Service Fabric, los almacenes de estado externos se pueden internalizar como servicios con estado, cuando corresponda. Esto es más complicado que una migración simple de roles web y de trabajo a servicios sin estado de Service Fabric, ya que requiere escribir servicios personalizados que proporcionen la aplicación una funcionalidad equivalente a la que ofrecían los servicios externos. Las ventajas de hacerlo son: 

* Eliminación de dependencias externas 
* Unificación de los modelos de implementación, administración y actualización. 

Una arquitectura de ejemplo resultante de internalizar estos servicios podría tener este aspecto:

![Arquitectura de Service Fabric después de una migración completa][11]

## <a name="communication-and-workflow"></a>Comunicación y flujo de trabajo
La mayoría de las aplicaciones del Servicio en la nube constan de más de un nivel. De forma similar, una aplicación de Service Fabric consta de más de un servicio (normalmente muchos servicios). Dos modelos de comunicación comunes son la comunicación directa y a través de un almacenamiento externo duradero.

### <a name="direct-communication"></a>Comunicación directa
Con la comunicación directa, los niveles pueden comunicarse directamente a través del punto de conexión expuesto por cada nivel. En los entornos sin estado tales como Cloud Services, esto significa seleccionar una instancia de un rol de VM, al azar o por turnos, para equilibrar la carga y conectarse directamente a su punto de conexión.

![Comunicación directa de Cloud Services][5]

 La comunicación directa es un modelo de comunicación común en Service Fabric. La principal diferencia entre Service Fabric y Cloud Services es que en Cloud Services se conecta a una máquina virtual, mientras que en Service Fabric se conecta a un servicio. Esta distinción es importante por dos motivos:

* En Service Fabric, los servicios no están enlazados a las máquinas virtuales que los hospedan y pueden moverse por el clúster y; de hecho, se espera que lo hagan por varias razones: equilibrio de recursos, conmutación por error, actualizaciones de aplicaciones y de infraestructura, y restricciones de colocación o carga. Esto significa que la dirección de una instancia de servicio se puede cambiar en cualquier momento. 
* En Service Fabric, una máquina virtual puede hospedar varios servicios, cada uno con puntos de conexión únicos.

Service Fabric proporciona un mecanismo de detección de servicios, llamado Servicio de nombres, que puede usarse para resolver direcciones de puntos de conexión. 

![Comunicación directa de Service Fabric][6]

### <a name="queues"></a>Colas
Un mecanismo de comunicación común entre los niveles en entornos sin estado, como Cloud Services, es usar una cola de almacenamiento externo para almacenar a largo plazo tareas de trabajo de un nivel a otro. Un escenario común es un nivel web que envía trabajos a una cola de Azure o de Service Bus, y las instancias de rol Worker Role pueden quitar de la cola los trabajos y procesarlos.

![Comunicación mediante la cola de Cloud Services][7]

Puede usarse el mismo modelo de comunicación en Service Fabric. Esto puede ser útil al migrar una aplicación existente de Cloud Services a Service Fabric. 

![Comunicación directa de Service Fabric][8]

## <a name="parity"></a>Parity
[Cloud Services es similar a Service Fabric en lo que se refiere al grado de control frente a la facilidad de uso, pero es un servicio heredado y Service Fabric se recomienda para los desarrollos nuevos](/azure/architecture/guide/technology-choices/compute-decision-tree). A continuación se muestra una comparación de la API:


| **API de Cloud Services** | **API de Service Fabric** | **Notas** |
| --- | --- | --- |
| RoleInstance.GetID | FabricRuntime.GetNodeContext.NodeId o .NodeName | ID es una propiedad de NodeName. |
| RoleInstance.GetFaultDomain | FabricClient.QueryManager.GetNodeList | Filtrar por NodeName y usar la propiedad FD. |
| RoleInstance.GetUpgradeDomain | FabricClient.QueryManager.GetNodeList | Filtrar por NodeName y usar la propiedad Upgrade. |
| RoleInstance.GetInstanceEndpoints | FabricRuntime.GetActivationContext o Naming (ResolveService) | CodePackageActivationContext que se proporciona mediante FabricRuntime.GetActivationContext y dentro de las réplicas mediante ServiceInitializationParameters.CodePackageActivationContext proporcionado durante .Initialize. |
| RoleEnvironment.GetRoles | FabricClient.QueryManager.GetNodeList | Si desea realizar el mismo tipo de filtrado por tipo, puede obtener la lista tipos de nodos del manifiesto de clúster mediante FabricClient.ClusterManager.GetClusterManifest y seleccionar los tipos de rol/nodo de ahí. |
| RoleEnvironment.GetIsAvailable | Connect-WindowsFabricCluster o crear FabricRuntime que apunta a un nodo determinado | * |
| RoleEnvironment.GetLocalResource | CodePackageActivationContext.Log/Temp/Work | * |
| RoleEnvironment.GetCurrentRoleInstance | CodePackageActivationContext.Log/Temp/Work | * |
| LocalResource.GetRootPath | CodePackageActivationContext.Log/Temp/Work | * |
| Role.GetInstances | FabricClient.QueryManager.GetNodeList o ResolveService | * |
| RoleInstanceEndpoint.GetIPEndpoint | FabricRuntime.GetActivationContext o Naming (ResolveService) | * |

## <a name="next-steps"></a>Pasos siguientes
La ruta de migración más sencilla desde Cloud Services a Service Fabric es reemplazar solo la implementación de Cloud Services por una aplicación de Service Fabric y mantener la arquitectura general de la misma. El siguiente artículo proporciona una guía para ayudar a convertir un rol de trabajo o web en un servicio sin estado de Service Fabric.

* [Migración sencilla: convertir un rol de trabajo o web en un servicio sin estado de Service Fabric](service-fabric-cloud-services-migration-worker-role-stateless-service.md)

<!--Image references-->
[1]: ./media/service-fabric-cloud-services-migration-differences/topology-cloud-services.png
[2]: ./media/service-fabric-cloud-services-migration-differences/topology-service-fabric.png
[5]: ./media/service-fabric-cloud-services-migration-differences/cloud-service-communication-direct.png
[6]: ./media/service-fabric-cloud-services-migration-differences/service-fabric-communication-direct.png
[7]: ./media/service-fabric-cloud-services-migration-differences/cloud-service-communication-queues.png
[8]: ./media/service-fabric-cloud-services-migration-differences/service-fabric-communication-queues.png
[9]: ./media/service-fabric-cloud-services-migration-differences/cloud-services-architecture.png
[10]: ./media/service-fabric-cloud-services-migration-differences/service-fabric-architecture-simple.png
[11]: ./media/service-fabric-cloud-services-migration-differences/service-fabric-architecture-full.png