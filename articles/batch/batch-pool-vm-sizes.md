---
title: Selección de los tamaños de máquina virtual para grupos
description: Cómo elegir uno de los tamaños de máquina virtual disponibles para los nodos de proceso en grupos de Azure Batch
ms.topic: conceptual
ms.date: 08/07/2020
ms.custom: seodec18
ms.openlocfilehash: 9aef1fc21120401252d188b7373c6ce4139c71c4
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88005140"
---
# <a name="choose-a-vm-size-for-compute-nodes-in-an-azure-batch-pool"></a>Selección de un tamaño de máquina virtual para nodos de proceso en un grupo de Azure Batch

Cuando seleccione un tamaño de nodo para un grupo de Azure Batch, puede elegir entre casi todas las familias y tamaños de máquinas virtuales disponibles en Azure. Azure ofrece una variedad de tamaños para máquinas virtuales Windows y Linux para diferentes cargas de trabajo.

Hay algunas excepciones y limitaciones a la hora de elegir un tamaño de máquina virtual:

* Algunas series o tamaños de máquina virtual no se admiten en Batch.
* Algunos tamaños de máquina virtual están restringidos y tienen que habilitarse específicamente antes de poder asignarlos.

## <a name="supported-vm-series-and-sizes"></a>Series y tamaños de maquina virtual compatibles

### <a name="pools-in-virtual-machine-configuration"></a>Grupos en la configuración de máquina virtual

Los grupos de Batch en la configuración de máquina virtual son compatibles con casi todos los tamaños de máquina virtual ([Linux](../virtual-machines/linux/sizes.md), [Windows](../virtual-machines/windows/sizes.md)). Consulte la tabla siguiente para más información sobre los tamaños admitidos y las restricciones.

| Series de máquinas virtuales  | Tamaños admitidos |
|------------|---------|
| A básico | Todos los tamaños *excepto* Basic_A0 (A0) |
| A | Todos los tamaños *excepto* Standard_A0 |
| Av2 | Todos los tamaños |
| B | None |
| DC | None |
| Dv2, DSv2 | Todos los tamaños |
| Dv3, Dsv3 | Todos los tamaños |
| Dav4<sup>1</sup> | Todos los tamaños |
| Dav4<sup>1</sup> | Todos los tamaños |
| Ddv4, Ddsv4 |  Ninguno: no disponible todavía |
| Ev3, Esv3 | Todos los tamaños, excepto E64is_v3 y E64i_v3 |
| Eav4<sup>1</sup> | Todos los tamaños |
| Easv4<sup>1</sup> | Todos los tamaños |
| Edv4, Edsv4 |  Ninguno: no disponible todavía |
| F, Fs | Todos los tamaños |
| Fsv2 | Todos los tamaños |
| G, Gs | Todos los tamaños |
| H | Todos los tamaños |
| HB<sup>1</sup> | Todos los tamaños |
| HBv2<sup>1</sup> | Todos los tamaños |
| HC<sup>1</sup> | Todos los tamaños |
| LS | Todos los tamaños |
| Lsv2<sup>1</sup> | Todos los tamaños |
| M<sup>1</sup> | Todos los tamaños |
| Mv2 | Ninguno: no disponible todavía |
| NC | Todos los tamaños |
| NCv2<sup>1</sup> | Todos los tamaños |
| NCv3<sup>1</sup> | Todos los tamaños |
| ND<sup>1</sup> | Todos los tamaños |
| NDv2<sup>1</sup> | Ninguno: no disponible todavía |
| NV | Todos los tamaños |
| NVv3<sup>1</sup> | Todos los tamaños |
| NVv4 | None |
| SAP HANA | None |

<sup>1</sup> Estos tamaños de máquina virtual se pueden asignar en grupos de Batch en la configuración de la máquina virtual, pero debe crear una nueva cuenta de Batch y solicitar un [aumento de cuota](batch-quota-limit.md#increase-a-quota) específico. Esta limitación se eliminará una vez que la cuota de vCPU por serie de máquinas virtuales sea totalmente compatible con las cuentas de Batch.

### <a name="pools-in-cloud-service-configuration"></a>Grupos en la configuración de Cloud Service

Los grupos de Batch en la configuración de Cloud Services son compatibles con todos los [tamaños de máquina virtual para Cloud Services](../cloud-services/cloud-services-sizes-specs.md)**excepto** en los siguientes casos:

| Series de máquinas virtuales  | Tamaños no compatibles |
|------------|-------------------|
| Serie A   | Extra pequeño       |
| Serie Av2 | Standard_A1_v2, Standard_A2_v2, Standard_A2m_v2 |

## <a name="size-considerations"></a>Consideraciones de tamaño

* **Requisitos de aplicación**: tenga en cuenta las características y los requisitos de la aplicación que se va a ejecutar en los nodos. Aspectos tales como si la aplicación es multiproceso y cuánta memoria consume pueden ayudar a determinar el tamaño de nodo más adecuado y rentable. Para varias instancias de [cargas de trabajo MPI](batch-mpi.md) o aplicaciones CUDA, considere la posibilidad de tamaños de máquina virtual especializados [HPC](../virtual-machines/sizes-hpc.md) o [habilitado GPU](../virtual-machines/sizes-gpu.md), respectivamente. (Consulte [Uso de instancias compatibles con RDMA o habilitadas para GPU en grupos de Batch](batch-pool-compute-intensive-sizes.md)).

* **Tareas por nodo**: normalmente, se selecciona un tamaño de nodo bajo el supuesto de que no se ejecutará más que una sola tarea a la vez en un nodo. No obstante, puede tener ventajas [ejecutar en paralelo](batch-parallel-node-tasks.md) varias tareas y, por tanto, varias instancias de la aplicación, en varios nodos de proceso durante la ejecución del trabajo. En este caso, es habitual elegir un tamaño de nodo de varios núcleos para acomodar el aumento de la demanda por la ejecución de tareas en paralelo.

* **Niveles de carga para diferentes tareas**: todos los nodos en un grupo tienen el mismo tamaño. Si va a ejecutar aplicaciones con requisitos del sistema o niveles de carga diferentes, es recomendable usar grupos separados.

* **Disponibilidad por regiones**: una serie o tamaño de máquina virtual, podría no estar disponible en las regiones en las que cree las cuentas de Batch. Para comprobar que un tamaño está disponible, vea [Productos disponibles por región](https://azure.microsoft.com/regions/services/).

* **Cuotas**: la [cuota de núcleos](batch-quota-limit.md#resource-quotas) en su cuenta de Batch puede limitar el número de nodos de un tamaño específico que se puede agregar a un grupo de Batch. Para solicitar un aumento de la cuota, consulte [este artículo](batch-quota-limit.md#increase-a-quota). 

* **Configuración de grupo**: por lo general, tiene más opciones de tamaño de máquina virtual cuando crea un grupo en la configuración de máquina virtual, en comparación con la configuración de Cloud Service.

## <a name="next-steps"></a>Pasos siguientes

* Conozca el [flujo de trabajo y los recursos principales del servicio Batch](batch-service-workflow-features.md), como grupos, nodos, trabajos y tareas.
* Para más información acerca del uso de tamaños de máquinas virtuales de proceso intensivo, consulte [Uso de instancias compatibles con RDMA o habilitadas para GPU en grupos de Batch](batch-pool-compute-intensive-sizes.md).
