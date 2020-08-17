---
title: Obtenga más información sobre los modos de orquestación para los conjuntos de escalado de máquinas virtuales en Azure
description: Obtenga más información sobre los modos de orquestación para los conjuntos de escalado de máquinas virtuales en Azure.
author: mimckitt
ms.author: mimckitt
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 10/23/2019
ms.reviewer: jushiman
ms.custom: mimckitt
ms.openlocfilehash: eb7d4d8a6f1c1ee55601cdd839e330147e60bcc7
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87011080"
---
# <a name="orchestration-modes-preview"></a>Modo de orquestación (versión preliminar)

> [!CAUTION]
> Gracias a todos los usuarios que participaron en esta versión preliminar pública. Pudimos recopilar comentarios valiosos de nuestra comunidad. Esta versión preliminar está ahora **cerrada** a nuevos participantes, con el objetivo de integrar los comentarios. Este espacio se actualizará con cualquier nueva información que tengamos.

Los conjuntos de escalado de máquinas virtuales proporcionan una agrupación lógica de máquinas virtuales administradas por la plataforma. Con los conjuntos de escalado, se crea un modelo de configuración de máquina virtual, se agregan o quitan automáticamente instancias adicionales en función de la carga de CPU o de memoria y se actualiza automáticamente a la versión más reciente del sistema operativo. Tradicionalmente, los conjuntos de escalado permiten crear máquinas virtuales con un modelo de configuración de máquina virtual proporcionado en el momento de la creación del conjunto de escalado. Además, el conjunto de escalado solo puede administrar máquinas virtuales que se crean de forma implícita según el modelo de configuración.

Con el modo de orquestación del conjunto de escalado (versión preliminar), ahora puede elegir si el conjunto de escalado debe orquestar las máquinas virtuales que se crean explícitamente fuera de un modelo de configuración de conjunto de escalado o las instancias de máquina virtual creadas de manera implícita en función del modelo de configuración. El modo de orquestación del conjunto de escalado también le ayuda a diseñar la infraestructura de máquinas virtuales para lograr una alta disponibilidad a través de la implementación de estas máquinas virtuales en dominios de error y Availability Zones.


Los conjuntos de escalado de máquinas virtuales admitirán dos modos de orquestación distintos:

- ScaleSetVM: las instancias de máquina virtual que se agregan al conjunto de escalado se basan en el modelo de configuración del conjunto de escalado. Ciclo de vida de la instancia de máquina virtual: la creación, actualización y eliminación se administran mediante el conjunto de escalado.
- VM (máquinas virtuales): las máquinas virtuales creadas fuera del conjunto de escalado se pueden agregar de forma explícita al conjunto de escalado. 
 

> [!IMPORTANT]
> El modo de orquestación se define al crear el conjunto de escalado y no se puede cambiar ni actualizar más adelante. 
> 
> Esta característica de los conjuntos de escalado de máquinas virtuales se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. 
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


## <a name="orchestration-modes"></a>Modos de orquestación

| Característica                     | "orchestrationMode”: "VM" (VirtualMachine) | "orchestrationMode": "ScaleSetVM" (VirtualMachineScaleSetVM) |
|-----------------------------|--------------------------------------------|--------------------------------------------------------------|
| Modelo de configuración de máquina virtual      | None                                       | Obligatorio |
| Incorporación de una máquina virtual nueva a un conjunto de escalado  | Las máquinas virtuales se agregan de manera explícita al conjunto de escalado cuando se crea la máquina virtual. | Las máquinas virtuales se crean de manera implícita y se agregan al conjunto de escalado según el modelo de configuración de la máquina virtual, el recuento de instancias y las reglas de escalado automático. | |
| Eliminación de la máquina virtual                   | Las máquinas virtuales deben eliminarse de forma individual, el conjunto de escalado no se eliminará si contiene máquinas virtuales. | Las máquinas virtuales se pueden eliminar de forma individual. Si elimina el conjunto de escalado, se eliminarán todas las instancias de máquina virtual.  |
| Asociación y disociación de máquinas virtuales           | No compatible                              | No compatible |
| Ciclo de vida de la instancia (creación mediante eliminación) | Las máquinas virtuales y sus artefactos (como los discos y adaptadores de red) se pueden administrar de forma independiente. | Las instancias y sus artefactos (como los discos y NIC) son implícitos en las instancias del conjunto de escalado que los crean. No se pueden separar ni administrar de forma independiente fuera del conjunto de escalado. |
| Dominios de error               | Puede definir los dominios de error. 2 o 3 según la compatibilidad regional y 5 para la zona de disponibilidad. | Puede definir dominios de error que van del 1 al 5 |
| Dominios de actualización              | Los dominios de actualización se asignan automáticamente a los dominios de error | Los dominios de actualización se asignan automáticamente a los dominios de error |
| Zonas de disponibilidad          | Admite la implementación regional o las máquinas virtuales en una zona de disponibilidad | Admite la implementación regional o varias zonas de disponibilidad. Puede definir la estrategia de equilibrio de zona. |
| Escalado automático                   | No compatible                              | Compatible |
| Actualización de sistema operativo                  | No compatible                              | Compatible |
| Actualizaciones del modelo               | No compatible                              | Compatible |
| Control de instancias            | Control completo de la máquina virtual. Las máquinas virtuales tienen un URI completo que admite toda la gama de funcionalidades de administración de máquinas virtuales de Azure (como Azure Policy, Azure Backup y Azure Site Recovery) | Las máquinas virtuales son recursos dependientes del conjunto de escalado. Solo se puede tener acceso a las instancias para administrarlas a través del conjunto de escalado. |
| Modelo de instancias              | Definición de modelo Microsoft.Compute/VirtualMachines. | Definición de modelo Microsoft.Compute/VirtualMachineScaleSets/VirtualMachines. |
| Capacity                    | Se puede crear un conjunto de escalado vacío; se pueden agregar hasta 200 máquinas virtuales al conjunto de escalado. | Los conjuntos de escalado se pueden definir con un recuento de entre 0 y 1000 instancias. |
| Move                        | Compatible                                  | Compatible |
| Grupo de selección de ubicación único == false | No compatible                          | Compatible |


## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte [Información general sobre las opciones de disponibilidad](../virtual-machines/availability.md?toc=%2fazure%2fvirtual-machine-scale-sets%2ftoc.json).
