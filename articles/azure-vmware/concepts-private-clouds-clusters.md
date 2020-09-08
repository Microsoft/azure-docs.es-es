---
title: Conceptos sobre nubes privadas y clústeres
description: Obtenga información sobre las funcionalidades clave de los centros de datos definidos por software de VMware en Azure y los clústeres de vSphere en la solución de VMware en Azure.
ms.topic: conceptual
ms.date: 05/04/2020
ms.openlocfilehash: 06161d2ce95415ae3309d58ad18ad0d40b3782fb
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88752279"
---
# <a name="azure-vmware-solution-preview-private-cloud-and-cluster-concepts"></a>Conceptos de nubes privadas y clústeres de Azure VMware Solution en versión preliminar

Azure VMware Solution ofrece nubes privadas basadas en VMware en Azure. Las nubes privadas se crean a partir de clústeres de hosts dedicados sin sistema operativo, y se implementan y administran mediante Azure Portal. Los clústeres en las nubes privadas se aprovisionan con el software VMware vSphere, vCenter, vSAN y NSX. Las implementaciones de software y hardware en nubes privadas de Azure VMware Solution están totalmente integradas y automatizadas en Azure.

Existe una relación lógica entre las suscripciones a Azure, las nubes privadas de Azure VMware Solution, los clústeres de vSAN y los hosts. En el diagrama, se muestran dos nubes privadas en una sola suscripción a Azure. Las nubes privadas representan un entorno de desarrollo y de producción, cada uno con su propia nube privada. En cada una de esas nubes privadas hay dos clústeres. Para mostrar las menores necesidades potenciales de un entorno de desarrollo, se usan clústeres más pequeños con hosts de menor capacidad. Todos estos conceptos se describen en las siguientes secciones.

![Imagen de dos nubes privadas en la suscripción de un cliente](./media/hosts-clusters-private-clouds-final.png)

## <a name="private-clouds"></a>Nubes privadas

Las nubes privadas contienen clústeres de vSAN que se han creado con hosts dedicados sin sistema operativo de Azure. Cada nube privada puede tener varios clústeres, todos administrados por el mismo servidor vCenter y NSX-T Manager. Puede implementar y administrar nubes privadas en el portal, desde la CLI o con PowerShell. Al igual que con otros recursos, las nubes privadas se instalan y administran desde una suscripción a Azure.

El número de nubes privadas dentro de una suscripción es escalable. Inicialmente, hay un límite de una nube privada por suscripción.

## <a name="clusters"></a>Clústeres

Creará al menos un clúster de vSAN en cada nube privada. Cuando crea una nube privada, hay un clúster de forma predeterminada. Puede agregar clústeres adicionales a una nube privada mediante Azure Portal o a través de la API. Todos los clústeres tienen un tamaño predeterminado de tres hosts y se pueden escalar de 3 a 16 hosts. El tipo de host que se usa en un clúster debe ser del mismo tipo. En la siguiente sección se describen los tipos de host.

Los clústeres de prueba están disponibles para su evaluación y están limitados a tres hosts y un único clúster de prueba por nube privada. Puede escalar un clúster de prueba en un solo host durante el período de evaluación.

Los clústeres se crean, eliminan y escalan desde el portal o la API. Se siguen usando vSphere y NSX-T Manager para administrar la mayoría de los otros aspectos de la configuración o del funcionamiento de los clústeres. Todo el almacenamiento local de cada host de un clúster está bajo el control de vSAN.

## <a name="hosts"></a>Hosts

En los clústeres de nube privada de Azure VMware Solution se usan nodos de infraestructura hiperconvergidos sin sistema operativo. En la tabla siguiente se indican las capacidades de RAM, CPU y disco del host. 

| Tipo de host              |             CPU             |   RAM (GB)   |  Nivel de caché NVMe de vSAN (TB, sin procesar)  |  Nivel de capacidad SSD de vSAN (TB, sin procesar)  |
| :---                   |            :---:            |    :---:     |               :---:              |                :---:               |
| High-End (HE)          |  Intel doble de 18 núcleos, 2,3 GHz  |     576      |                3.2               |                15,20               |

Los hosts que se usan para crear o escalar clústeres se adquieren de un grupo de hosts aislado. Esos hosts han superado las pruebas de hardware, y se han eliminado todos los datos de los discos flash. Cuando se quita un host de un clúster, los discos internos se borran de forma segura, y los hosts se colocan en el grupo de hosts aislado. Al agregar un host a un clúster, se usa un host saneado del grupo aislado.

## <a name="vmware-software-versions"></a>Versiones de software de VMware

Las versiones actuales del software de VMware que se usan en los clústeres de nube privada de Azure VMware Solution son:

| Software              |    Versión   |
| :---                  |     :---:    |
| VCSA/vSphere/ESXi |    6.7 U2    | 
| ESXi                  |    6.7 U2    | 
| vSAN                  |    6.7 U2    |
| NSX-T                 |      2.5     |

En el caso de un clúster nuevo en una nube privada, la versión del software coincidirá con la que se ejecute actualmente en la nube privada. Para cualquier nube privada nueva en una suscripción de cliente, se instala la versión más reciente de la pila de software.

Las directivas y procesos de actualización generales del software de la plataforma Azure VMware Solution se describen en el documento sobre conceptos de actualización.

## <a name="host-maintenance-and-lifecycle-management"></a>Mantenimiento y administración del ciclo de vida de hosts

El mantenimiento y la administración del ciclo de vida de los hosts se realizan sin afectar a la capacidad ni al rendimiento de los clústeres de la nube privada. Entre los ejemplos de mantenimiento automatizado de los hosts se incluyen las actualizaciones de firmware y la reparación o sustitución de hardware.

Microsoft es responsable de la administración del ciclo de vida de los dispositivos NSX-T, como NSX-T Manager y NSX-T Edge. Microsoft también es responsable de la configuración de la red de arranque, como la creación de la puerta de enlace de nivel 0 y la habilitación del enrutamiento vertical de arriba abajo. Como administrador de la nube privada de Azure VMware Solution, usted es responsable de la configuración de SDN de NSX-T, como los segmentos de red, las reglas de firewall distribuidas, las puertas de enlace de nivel 1 y los equilibradores de carga.

> [!IMPORTANT]
> Un administrador de Azure VMware Solution no debe modificar la configuración de NSX-T Edge ni la puerta de enlace de nivel 0. Esto puede provocar una pérdida del servicio.

## <a name="backup-and-restoration"></a>Copia de seguridad y restauración

Se realiza una copia de seguridad de las configuraciones de vCenter y NSX-T de la nube privada cada hora. Las copias de seguridad se conservan durante tres días. La restauración a partir de una copia de seguridad se solicita a través de una solicitud de servicio en Azure Portal.

## <a name="next-steps"></a>Pasos siguientes

El siguiente paso es aprender los [conceptos de redes e interconectividad](concepts-networking.md).

<!-- LINKS - internal -->

<!-- LINKS - external-->
[VCSA versions]: https://kb.vmware.com/s/article/2143838
[ESXi versions]: https://kb.vmware.com/s/article/2143832
[vSAN versions]: https://kb.vmware.com/s/article/2150753

