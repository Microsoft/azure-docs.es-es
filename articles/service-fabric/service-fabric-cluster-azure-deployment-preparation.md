---
title: Planeación de una implementación de clúster de Azure Service Fabric
description: Obtenga información sobre la planificación y preparación de la implementación de un clúster de producción de Service Fabric.
ms.topic: conceptual
ms.date: 03/20/2019
ms.openlocfilehash: 1064e59491b7144aafade24bd50131478fe025eb
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87281334"
---
# <a name="plan-and-prepare-for-a-cluster-deployment"></a>Planeamiento y preparación de la implementación de un clúster

La planeación y preparación para la implementación de un clúster de producción son muy importantes.  Existen muchos factores a tener en cuenta.  Este artículo le guiará por los pasos de preparación de la implementación del clúster.

## <a name="read-the-best-practices-information"></a>Lectura de información de procedimientos recomendados
Para administrar clústeres y aplicaciones de Azure Service Fabric correctamente, existen una serie de procedimientos que le recomendamos realizar para optimizar la fiabilidad de su entorno de producción.  Para obtener más información, consulte [Procedimientos recomendados para aplicaciones y clústeres de Service Fabric](service-fabric-best-practices-overview.md).

## <a name="select-the-os-for-the-cluster"></a>Selección del sistema operativo del clúster
Service Fabric permite la creación de clústeres de Service Fabric en cualquier máquina virtual o equipo con Windows Server o Linux.  Antes de implementar el clúster, debe elegir el sistema operativo:  Windows o Linux.  Todos los nodos (máquina virtual) del clúster utilizan el mismo sistema operativo. No puede combinar máquinas virtuales de Windows y Linux en el mismo clúster.

## <a name="capacity-planning"></a>Planificación de capacidad
En cualquier implementación de producción, la planeación de capacidad es un paso importante. Esto es algo que hay que tener en cuenta como parte de ese proceso.

* Número inicial de tipos de nodo para el clúster 
* Propiedades de cada tipo de nodo (tamaño, número de instancias, principal, accesible desde Internet, número de máquinas virtuales, etc.)
* Características de confiabilidad y durabilidad del clúster

### <a name="select-the-initial-number-of-node-types"></a>Selección del número inicial de tipos de nodo
En primer lugar, debe decidir para qué va a servir el clúster que va a crear. ¿Qué tipos de aplicaciones planea que se van a implementar en este clúster? ¿Tiene la aplicación varios servicios y alguno de ellos ha de ser público o accesible desde Internet? ¿Tienen los servicios (que componen la aplicación) distintos requisitos de infraestructura, como, por ejemplo, una RAM mayor o un número más alto de ciclos de CPU? Un clúster de Service Fabric puede constar de más de un tipo de nodo: un tipo de nodo principal o más tipos de nodo no principales. Cada tipo de nodo se asigna a un conjunto de escalado de máquinas virtuales. Cada tipo de nodo se puede escalar o reducir verticalmente de forma independiente. Cada uno tiene diferentes conjuntos de puertos abiertos y puede tener distintas métricas de capacidad. Se pueden configurar [restricciones de ubicación y propiedades de nodo][placementconstraints] para limitar servicios específicos para tipos de nodo determinados.  Para más información, consulte [planeamiento de capacidad del clúster de Service Fabric](service-fabric-cluster-capacity.md).

### <a name="select-node-properties-for-each-node-type"></a>Selección de propiedades de nodo para cada tipo de nodo
Los tipos de nodo definen la SKU de máquina virtual el número y las propiedades de las máquinas virtuales en el conjunto de escalado asociado.

El tamaño mínimo de las máquinas virtuales para cada tipo de nodo está determinado por el [nivel de durabilidad][durability] que elija para el tipo de nodo.

El número mínimo de máquinas virtuales del tipo de nodo principal se determina mediante el [nivel de confiabilidad][reliability] que se elija.

Consulte las recomendaciones mínimas para [tipos de nodo principales](service-fabric-cluster-capacity.md#primary-node-type), [cargas de trabajo con estado en tipos de nodo no principales](service-fabric-cluster-capacity.md#stateful-workloads) y [cargas de trabajo sin estado en tipos de nodo no principales](service-fabric-cluster-capacity.md#stateless-workloads).

Los números de nodos superiores al mínimo deben basarse en el número de réplicas de la aplicación o los servicios que desee ejecutar en ese tipo de nodo.  [Planeamiento de capacidad para aplicaciones de Service Fabric](service-fabric-capacity-planning.md) le ayuda a estimar los recursos que necesita para ejecutar las aplicaciones. Siempre puede escalar o reducir verticalmente el clúster más tarde para ajustarlo a una carga de trabajo de aplicaciones cambiante. 

#### <a name="use-ephemeral-os-disks-for-virtual-machine-scale-sets"></a>Uso de discos de sistema operativo efímeros para conjuntos de escalado de máquinas virtuales

Los *discos del sistema operativo efímeros* son almacenamiento creado en la máquina virtual local y no se guardan en la instancia remota de Azure Storage. Se recomiendan para todos los tipos de nodos de Service Fabric (principales y secundarios) ya que, comparados con los discos del SO persistentes tradicionales, los discos efímeros:

* Reducen la latencia de lectura y escritura en el disco del sistema operativo;
* Permiten realizar con mayor rapidez las operaciones de administración para restablecimiento y restablecimiento de imagen inicial de los nodos;
* Reducen los costos generales (los discos son gratuitos y no hay ningún costo de almacenamiento adicional).

Los discos del sistema operativo efímeros no son una característica específica de Service Fabric, sino una característica de los *conjuntos de escalado de máquinas virtuales* de Azure que se asignan a los tipos de nodo de Service Fabric. Para usarlos con Service Fabric necesita realizar lo siguiente en la plantilla de Azure Resource Manager de su clúster:

1. Asegúrese de que los tipos de nodo tienen [tamaños de máquina virtual de Azure compatibles](../virtual-machines/ephemeral-os-disks.md) con los discos del sistema operativo efímeros, y que el tamaño de la máquina virtual tiene un tamaño de caché adecuado para admitir el tamaño del disco del sistema operativo (consulte la *Nota* a continuación). Por ejemplo:

    ```xml
    "vmNodeType1Size": {
        "type": "string",
        "defaultValue": "Standard_DS3_v2"
    ```

    > [!NOTE]
    > Asegúrese de seleccionar un tamaño de máquina virtual con un tamaño de caché igual o mayor que el tamaño de disco del sistema operativo de la propia máquina virtual; en caso contrario, la implementación de Azure podría generar un error (incluso si acepta el disco inicialmente).

2. Especifique una versión de conjunto de escalado de máquinas virtuales (`vmssApiVersion`) con un valor de `2018-06-01` o posterior:

    ```xml
    "variables": {
        "vmssApiVersion": "2018-06-01",
    ```

3. En la sección Conjunto de escalado de máquinas virtuales de la plantilla de implementación, especifique la opción `Local` para `diffDiskSettings`:

    ```xml
    "apiVersion": "[variables('vmssApiVersion')]",
    "type": "Microsoft.Compute/virtualMachineScaleSets",
        "virtualMachineProfile": {
            "storageProfile": {
                "osDisk": {
                        "caching": "ReadOnly",
                        "createOption": "FromImage",
                        "diffDiskSettings": {
                            "option": "Local"
                        },
                }
            }
        }
    ```

> [!NOTE]
> Las aplicaciones de usuario no deben tener ninguna dependencia, archivo o artefacto en el disco del sistema operativo, ya que este se perdería en caso de una actualización del sistema operativo.
> Por lo tanto, no se recomienda usar [PatchOrchestrationApplication](https://github.com/microsoft/Service-Fabric-POA) con discos efímeros.
>

> [!NOTE]
> Los VMSS no efímeros existentes no se pueden actualizar en contexto para usar discos efímeros.
> Para migrarlos, los usuarios tendrán que [agregar](./virtual-machine-scale-set-scale-node-type-scale-out.md) un nuevo nodeType con discos efímeros, trasladar las cargas de trabajo al nuevo nodeType y [quitar](./service-fabric-how-to-remove-node-type.md) el nodeType existente.
>

Para obtener más información y opciones de configuración adicionales, consulte [Discos de SO efímeros para máquinas virtuales de Azure](../virtual-machines/ephemeral-os-disks.md). 


### <a name="select-the-durability-and-reliability-levels-for-the-cluster"></a>Selección de los niveles de durabilidad y confiabilidad para el clúster
El nivel de durabilidad se usa para indicar al sistema los privilegios que tienen las máquinas virtuales con la infraestructura subyacente de Azure. En el tipo de nodo principal, este privilegio permite que Service Fabric pause cualquier solicitud de la infraestructura de nivel de máquina virtual (por ejemplo, reinicio de máquina virtual, restablecimiento de la imagen inicial de máquina virtual o migración de máquina virtual) que afecte a los requisitos de quórum de los servicios del sistema y los servicios con estado. En los tipos de nodo distintos del principal, este privilegio permite que Service Fabric pause cualquier solicitud de la infraestructura de nivel de máquina virtual (por ejemplo, el reinicio de máquina virtual, el restablecimiento de la imagen inicial de máquina virtual o la migración de máquinas virtuales) que afecte a los requisitos de cuórum de los servicios con estado.  Para conocer las ventajas de los diferentes niveles y obtener recomendaciones sobre qué nivel usar y cuándo, vea [Características de durabilidad del clúster][durability].

El nivel de confiabilidad se usa para establecer el número de réplicas de los servicios del sistema que se desea ejecutar en el tipo de nodo principal de este clúster. Cuanto mayor sea el número de réplicas, más confiables son los servicios del sistema en el clúster.  Para conocer las ventajas de los diferentes niveles y obtener recomendaciones sobre qué nivel usar y cuándo, vea [Características de confiabilidad del clúster][reliability]. 

## <a name="enable-reverse-proxy-andor-dns"></a>Habilitación del proxy inverso o DNS
Los servicios que se conectan entre sí dentro de un clúster pueden acceder, por lo general, directamente a los puntos de conexión de otros servicios ya que los nodos de un clúster se encuentran en la misma red local. Para facilitar la conexión entre servicios, Service Fabric ofrece servicios adicionales: Un [servicio DNS](service-fabric-dnsservice.md) y un [servicio de proxy inverso](service-fabric-reverseproxy.md).  Se pueden habilitar ambos servicios cuando se implementa un clúster.

Debido a que muchos servicios, sobre todo los servicios en contenedores, pueden tener un nombre de dirección URL existente, es conveniente tener la capacidad de resolverlo mediante el protocolo DNS estándar (en lugar del protocolo Naming Service), en particular en escenarios "lift-and-shift" de la aplicación. Esto es exactamente lo que hace el servicio DNS. Permite asignar nombres DNS a un nombre de servicio y, por tanto, resolver direcciones IP del punto de conexión.

Los servicios de direcciones de proxy inverso del clúster que expone puntos de conexión HTTP (incluidos HTTPS). El proxy inverso simplifica ampliamente la llamada a otros servicios mediante un formato de URI específico.  El proxy inverso también administra los pasos de resolución, conexión y reintento necesarios para que un servicio se comunique con otro.

## <a name="prepare-for-disaster-recovery"></a>Preparación para la recuperación ante desastres
Una parte fundamental de la entrega de alta disponibilidad es garantizar que los servicios sobreviven a cualquier tipo de error. Esto es especialmente importante para los errores imprevistos y que se encuentran fuera de control. [Preparación para la recuperación ante desastres](service-fabric-disaster-recovery.md) describe algunos modos de error frecuentes que podrían convertirse en desastres si no se modelan y administran correctamente. También se explican las mitigaciones y las acciones que deben realizarse en caso de desastre.

## <a name="production-readiness-checklist"></a>Lista de comprobación sobre la preparación de producción
¿La aplicación y el clúster están preparados para asumir el tráfico de producción? Antes de realizar la implementación del clúster en la producción, ejecute la [lista de comprobación sobre la preparación de producción](service-fabric-production-readiness-checklist.md). Revise los elementos de la siguiente lista de comprobación para que la aplicación y el clúster se ejecuten sin problemas. Se recomienda encarecidamente comprobar todos estos elementos antes de pasar a la producción.

## <a name="next-steps"></a>Pasos siguientes
* [Creación de un clúster de Service Fabric que ejecuta Windows](service-fabric-best-practices-overview.md)
* [Creación de un clúster de Service Fabric que ejecuta Linux](service-fabric-tutorial-create-vnet-and-linux-cluster.md)

[placementconstraints]: service-fabric-cluster-resource-manager-cluster-description.md#node-properties-and-placement-constraints
[durability]: service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster
[reliability]: service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster