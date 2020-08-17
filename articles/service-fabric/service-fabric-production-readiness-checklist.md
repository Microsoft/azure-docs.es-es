---
title: Lista de comprobación sobre la preparación de producción de Azure Service Fabric
description: Prepare la producción del clúster y de la aplicación de Service Fabric mediante los procedimientos recomendados.
ms.topic: conceptual
ms.date: 6/05/2019
ms.openlocfilehash: a03df40a8ce213c5de9ed7017d47713c4de3449d
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87835874"
---
# <a name="production-readiness-checklist"></a>Lista de comprobación sobre la preparación de producción

¿La aplicación y el clúster están preparados para asumir el tráfico de producción? La ejecución y las pruebas de la aplicación y del clúster no significan necesariamente que todo esté listo para pasar a producción. Revise la siguiente lista de comprobación para que la aplicación y el clúster se ejecuten sin problemas. Se recomienda encarecidamente comprobar todos estos elementos. Obviamente, puede optar por usar soluciones alternativas para un elemento de línea específico (por ejemplo, sus propios marcos de diagnóstico).


## <a name="prerequisites-for-production"></a>Requisitos previos de producción
1. Procedimientos recomendados de Azure Service Fabric: [diseño de aplicaciones](./service-fabric-best-practices-applications.md), [seguridad](./service-fabric-best-practices-security.md), [redes](./service-fabric-best-practices-networking.md), [planeamiento de la capacidad y escalado](./service-fabric-best-practices-capacity-scaling.md), [infraestructura como código](./service-fabric-best-practices-infrastructure-as-code.md) y [supervisión y diagnóstico](./service-fabric-best-practices-monitoring.md). 
1. [Configure FabricTransport](./service-fabric-reliable-actors-fabrictransportsettings.md) si usa el modelo de programación Reliable Actors y precisa de una comunicación segura entre servicios.
1. Para los clústeres con más de 20 núcleos o 10 nodos, cree un tipo de nodo principal dedicado para los servicios del sistema. Agregue [restricciones de colocación](service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies.md) para reservar el tipo de nodo principal para los servicios del sistema.
1. Use una SKU D2v2 o una versión superior para el tipo de nodo principal. Se recomienda seleccionar una SKU con 50 GB de capacidad de disco duro como mínimo.
1. Los clústeres de producción deben ser [seguros](service-fabric-cluster-security.md). Para obtener un ejemplo de cómo configurar un clúster seguro, consulte esta [plantilla del clúster](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/7-VM-Windows-3-NodeTypes-Secure-NSG). Use nombres comunes para los certificados y evite el uso de certificados autofirmados.
1. Agregue [restricciones de recursos en contenedores y servicios](service-fabric-resource-governance.md), de modo que no consuman más del 75 % de los recursos del nodo. 
1. Conozca y establezca el [nivel de durabilidad](service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster). Se recomienda el nivel de durabilidad Silver o superior para los tipos de nodos que ejecutan cargas de trabajo con estado.
1. Comprenda y seleccione el [nivel de confiabilidad](service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster) del tipo de nodo. Se recomienda el nivel de confiabilidad Silver o superior.
1. Cargue las cargas de trabajo y pruebe de escalarlas para identificar [los requisitos de capacidad](service-fabric-cluster-capacity.md) para el clúster. 
1. Mediante las alertas, se supervisan los servicios y las aplicaciones y se generan y almacenan registros de aplicaciones. Por ejemplo, consulte [Adición del registro a la aplicación de Service Fabric](service-fabric-how-to-diagnostics-log.md) y [Supervisión de contenedores con los registros de Azure Monitor](service-fabric-diagnostics-oms-containers.md).
1. El clúster se supervisa con alertas (por ejemplo, con [registros de Azure Monitor](service-fabric-diagnostics-event-analysis-oms.md)). 
1. La infraestructura del conjunto de escalado de máquinas virtuales subyacente se supervisa mediante alertas (por ejemplo, con [registros de Azure Monitor](service-fabric-diagnostics-oms-agent.md)).
1. El clúster siempre tiene [certificados principales y secundarios](service-fabric-cluster-security-update-certs-azure.md) (para que no experimente bloqueos).
1. Mantenga clústeres independientes para el desarrollo, el ensayo y la producción. 
1. Las [actualizaciones de la aplicación](service-fabric-application-upgrade.md) y las [actualizaciones del clúster](service-fabric-tutorial-upgrade-cluster.md) primero se prueban en clústeres de desarrollo y ensayo. 
1. Desactive las actualizaciones automáticas en los clústeres de producción y actívelas para los clústeres de desarrollo y ensayo (reviértalo según sea necesario). 
1. Establezca un objetivo de punto de recuperación (RPO) para el servicio y configure un [proceso de recuperación ante desastres](service-fabric-disaster-recovery.md) y pruébelo.
1. Planee el [escalado](service-fabric-cluster-scaling.md) del clúster manualmente o mediante programación.
1. Planee la [revisión](service-fabric-patch-orchestration-application.md) de los nodos de clúster. 
1. Establezca una canalización de CI/CD para que los cambios más recientes se prueben continuamente. Por ejemplo, con [Azure DevOps](service-fabric-tutorial-deploy-app-with-cicd-vsts.md) o [Jenkins](/azure/developer/jenkins/deploy-to-service-fabric-cluster)
1. Pruebe los clústeres de desarrollo y ensayo con carga con el [servicio de análisis de errores](service-fabric-testability-overview.md) e inducza [caos](service-fabric-controlled-chaos.md) controlado. 
1. Planee el [escalado](service-fabric-concepts-scalability.md) de las aplicaciones. 


Si usa el modelo de programación de Reliable Services o Reliable Actors de Service Fabric, deben comprobarse los elementos siguientes:
1. Actualice las aplicaciones durante el desarrollo local para comprobar que el código de servicio respeta el token de cancelación en el método `RunAsync` y cierra los agentes de escucha de comunicación personalizada.
1. Evite [errores comunes](service-fabric-work-with-reliable-collections.md) al utilizar Reliable Collections.
1. Supervise los contadores de rendimiento de memoria de CLR de .NET al ejecutar pruebas de carga y busque tasas altas de crecimiento descontrolado en el montón o de recolección de elementos no utilizados.
1. Mantenga la copia de seguridad sin conexión de [Reliable Services y Reliable Actors](service-fabric-reliable-services-backup-restore.md) y pruebe el proceso de restauración.
1. Idealmente, el número de instancias de máquina virtual de NodeType principal debe ser igual al mínimo para el nivel de confiabilidad de clústeres; las condiciones cuando es adecuado exceder el mínimo del nivel incluyen las siguientes: temporalmente cuando se escala de manera vertical la SKU del conjunto de escalado de máquinas virtuales de NodeTypes principal.

## <a name="optional-best-practices"></a>Procedimientos recomendados opcionales

Aunque las listas anteriores son requisitos previos para pasar a la producción, también deben considerarse los siguientes elementos:
1. Conectar el [modelo de estado de Service Fabric](service-fabric-health-introduction.md) para ampliar los informes y la evaluación de estado integrada.
1. Implementar un guardián personalizado que supervise la [carga](service-fabric-cluster-resource-manager-metrics.md) de la aplicación y de los informes para el [equilibrio de recursos](service-fabric-cluster-resource-manager-balancing.md). 


## <a name="next-steps"></a>Pasos siguientes
* [Implementación de un clúster Windows de Service Fabric](service-fabric-tutorial-create-vnet-and-windows-cluster.md)
* [Implementación de un clúster de Service Fabric de Linux](service-fabric-tutorial-create-vnet-and-linux-cluster.md)
* Más información acerca del [ciclo de vida de aplicaciones](service-fabric-application-lifecycle.md) de Service Fabric.