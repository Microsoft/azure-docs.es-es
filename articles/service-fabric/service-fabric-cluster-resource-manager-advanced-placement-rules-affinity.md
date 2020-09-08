---
title: 'Resource Manager para el clúster de Service Fabric: afinidad'
description: Introducción a la afinidad de servicio para servicios de Azure Service Fabric e instrucciones sobre la configuración de afinidad de servicio.
services: service-fabric
documentationcenter: .net
author: masnider
ms.topic: conceptual
ms.date: 08/18/2017
ms.author: masnider
ms.custom: devx-track-csharp
ms.openlocfilehash: 9c141cd96877fd140b858d0aaed9197f2de80eca
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89005741"
---
# <a name="configuring-and-using-service-affinity-in-service-fabric"></a>Configuración y uso de la afinidad de servicio en Service Fabric
La afinidad es un control que se proporciona principalmente para facilitar la transición de aplicaciones grandes y monolíticas al mundo de la nube y los microservicios. También se usa como optimización para mejorar el rendimiento de los servicios, aunque en este caso podrían producirse efectos secundarios.

Supongamos que quiere migrar una aplicación grande, o una que se acaba de diseñar teniendo en mente los microservicios, a Service Fabric (o a algún entorno distribuido). Este tipo de transición es habitual. Para empezar, traslade toda la aplicación al entorno, empaquétela y asegúrese de que se ejecute sin problemas. A continuación, divídala en servicios más pequeños que se comuniquen entre sí.

A la larga puede encontrar que la aplicación experimenta algunos problemas. Los problemas pertenecen por lo general a una de estas categorías:

1. Uno de los componentes X de la aplicación monolítica tenía una dependencia sin documentar del componente Y, y acaba de convertir estos componentes en servicios independientes. Como ahora estos servicios se ejecutan en diferentes nodos del clúster, están separados.
2. Estos componentes se comunican a través de (canalizaciones con nombre locales | memoria compartida | archivos del disco) y necesitan realmente poder escribir ahora mismo en un recurso local compartido por motivos de rendimiento. Quizás, esa dependencia física se quite más adelante.
3. Todo es correcto, pero resulta que estos dos componentes están realmente fragmentados o son sensibles al rendimiento. Cuando se mueven a servicios independientes, el rendimiento general de la aplicación se reduce y aumenta la latencia. Como resultado, la aplicación en su conjunto no cumple las expectativas.

En estos casos, no queremos perder nuestro trabajo de refactorización y no queremos volver a lo monolítico. La última condición incluso puede ser deseable como una optimización sin formato. Sin embargo, hasta que podamos rediseñar los componentes para que funcionen de manera natural como servicios o hasta que podamos resolver las expectativas de rendimiento de alguna otra forma, vamos a seguir necesitando algún sentido de localidad.

¿Qué debe hacer? Podemos intentar activar la afinidad.

## <a name="how-to-configure-affinity"></a>Configuración de la afinidad
Para configurar la afinidad, debe definir una relación de afinidad entre dos servicios diferentes. Es como si hiciera que un servicio "apuntara" a otro y dijera "Este servicio solo se puede ejecutar donde se esté ejecutando ese otro". A veces se denominan relaciones de tipo primario-secundario (en las que el elemento secundario apunta al elemento primario). La afinidad garantiza que las réplicas o las instancias de un servicio se colocan en los mismos nodos que las réplicas o las instancias de otro.

```csharp
ServiceCorrelationDescription affinityDescription = new ServiceCorrelationDescription();
affinityDescription.Scheme = ServiceCorrelationScheme.Affinity;
affinityDescription.ServiceName = new Uri("fabric:/otherApplication/parentService");
serviceDescription.Correlations.Add(affinityDescription);
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

> [!NOTE]
> Un servicio secundario solo puede participar en una relación de afinidad sencilla. Si quisiera que el elemento secundario participara en una relación de afinidad con dos servicios principales a la vez, tendría dos opciones:
> - Invertir las relaciones (hacer que parentService1 y parentService2 apunten al servicio secundario actual), o
> - designar uno de los elementos primarios como centro por convención y hacer que todos los servicios apunten a ese servicio. 
>
> El comportamiento de colocación resultante en el clúster debe ser el mismo.
>

## <a name="different-affinity-options"></a>Diferentes opciones de afinidad
La afinidad se representa mediante uno de varios esquemas de correlación y tiene dos modos diferentes. El modo de afinidad más común es el denominado NonAlignedAffinity. En NonAlignedAffinity, las réplicas o las instancias de los diferentes servicios se colocan en los mismos nodos. El otro modo es AlignedAffinity. AlignedAffinity resulta de utilidad con servicios con estado. Configurar dos servicios con estado para tener afinidad alineada garantiza que los elementos principales de esos servicios se colocan en los mismos nodos. También hace que cada par de elementos secundarios de esos servicios se coloquen en los mismos nodos. También es posible (aunque menos frecuente) configurar NonAlignedAffinity para servicios con estado. Para NonAlignedAffinity, las diferentes réplicas de los dos servicios con estado se ejecutarían en los mismos nodos, pero sus elementos primarios podrían acabar en diferentes nodos.

<center>

![Modos de afinidad y consecuencias][Image1]
</center>

### <a name="best-effort-desired-state"></a>Estado de mejor esfuerzo deseado
Una relación de afinidad es la mejor opción. No proporciona las mismas garantías de colocación o confiabilidad que la ejecución en el mismo proceso ejecutable. Los servicios en una relación de afinidad son entidades básicamente diferentes que pueden producir error y moverse de forma independiente. También se puede interrumpir una relación de afinidad, aunque estas interrupciones son temporales. Por ejemplo, las limitaciones de capacidad pueden significar que solo algunos de los objetos de servicio de la relación de afinidad pueden caber en un determinado nodo. En estos casos, aunque exista una relación de afinidad, no se puede aplicar debido a las otras restricciones. Si es posible hacerlo, la infracción se corrige automáticamente más adelante.

### <a name="chains-vs-stars"></a>Cadenas frente a estrellas
Actualmente, Cluster Resource Manager no puede crear cadenas de relaciones de afinidad. Esto significa que un servicio que es un elemento secundario de una relación de afinidad no puede ser un elemento primario de otra relación de afinidad. Si desea modelar este tipo de relación, tendrá que crear una estrella, en lugar de una cadena. Para pasar de una cadena a una estrella, el elemento secundario de la parte inferior se convertiría en el elemento primario del primer elemento secundario. En función de la disposición de los servicios, puede que tenga que repetir esto varias veces. Si no hay un servicio principal natural, puede que tenga que crear uno que actúe como marcador de posición. En función de los requisitos, puede que también desee observar los [grupos de aplicaciones](service-fabric-cluster-resource-manager-application-groups.md).

<center>

![Cadenas frente a estrellas en el contexto de las relaciones de afinidad][Image2]
</center>

Otra cuestión que debe tener en cuenta sobre las relaciones de afinidad en la actualidad es que son direccionales de manera predeterminada. Esto significa que la regla de afinidad solo se aplica si el elemento secundario está donde el elemento primario. No garantiza que el elemento primario se coloque con el secundario. Por lo tanto, si existe una infracción de afinidad y, por algún motivo, para corregirla no es viable mover el elemento secundario al nodo primario (aunque el traslado del elemento primario al nodo secundario hubiese corregido la infracción), el elemento primario no se moverá al nodo secundario. Al establecer la configuración [MoveParentToFixAffinityViolation](service-fabric-cluster-fabric-settings.md) en true, se eliminaría la direccionalidad. También es importante tener en cuenta que la relación de afinidad no puede ser perfecta ni aplicarse instantáneamente dado que diferentes servicios tienen ciclos de vida diferentes y pueden producir error y moverse de forma independiente. Por ejemplo, supongamos que el elemento primario conmuta por error de repente a otro nodo porque se ha bloqueado. Cluster Resource Manager y el Administrador de conmutación por error gestionan primero la conmutación por error dado que lo importante es mantener los servicios en funcionamiento, coherentes y disponibles. Una vez finalizada la conmutación por error, la relación de afinidad se interrumpe, pero Cluster Resource Manager cree que todo va bien hasta que advierte que el elemento secundario no está colocado con el primario. Estos tipos de comprobaciones se realizan periódicamente. Puede encontrar más información sobre cómo Cluster Resource Manager evalúa las restricciones en [este artículo](service-fabric-cluster-resource-manager-management-integration.md#constraint-types), y en [este otro](service-fabric-cluster-resource-manager-balancing.md) se habla sobre cómo configurar la cadencia con la que se evalúan estos componentes.   


### <a name="partitioning-support"></a>Compatibilidad con particiones
Lo último que debe tener en cuenta sobre la afinidad es que no se admiten relaciones de afinidad en caso de que el elemento primario esté particionado. A la larga los servicios principales particionados se admitirán, pero por ahora no se permiten.

## <a name="next-steps"></a>Pasos siguientes
- Para más información sobre la configuración de servicios, vaya a [este vínculo](service-fabric-cluster-resource-manager-configure-services.md).
- Para limitar los servicios a un conjunto pequeño de máquinas o agregar la carga de servicios, use [Grupos de aplicaciones](service-fabric-cluster-resource-manager-application-groups.md).

[Image1]:./media/service-fabric-cluster-resource-manager-advanced-placement-rules-affinity/cluster-resrouce-manager-affinity-modes.png
[Image2]:./media/service-fabric-cluster-resource-manager-advanced-placement-rules-affinity/cluster-resource-manager-chains-vs-stars.png