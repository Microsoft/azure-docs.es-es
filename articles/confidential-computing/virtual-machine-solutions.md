---
title: Soluciones de computación confidencial de Azure en máquinas virtuales
description: Obtenga información sobre las soluciones de computación confidencial de Azure en máquinas virtuales.
author: JBCook
ms.service: virtual-machines
ms.subservice: workloads
ms.workload: infrastructure
ms.topic: conceptual
ms.date: 04/06/2020
ms.author: JenCook
ms.openlocfilehash: f9b73e0919d660947edd0417f7379b3f6e6140c0
ms.sourcegitcommit: c293217e2d829b752771dab52b96529a5442a190
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/15/2020
ms.locfileid: "88245859"
---
# <a name="solutions-on-azure-virtual-machines"></a>Soluciones en máquinas virtuales de Azure

En este artículo se ofrece información sobre la implementación de máquinas virtuales (VM) de computación confidencial de Azure que ejecutan procesadores Intel respaldados por [Intel Software Guard Extension](https://software.intel.com/sgx) (Intel SGX). 

## <a name="azure-confidential-computing-vm-sizes"></a>Tamaños de máquina virtual de computación confidencial de Azure

Las máquinas virtuales de computación confidencial de Azure están diseñadas para proteger la confidencialidad y la integridad de los datos y el código mientras se procesan en la nube. 

Las máquinas virtuales de la [serie DCsv2](../virtual-machines/dcv2-series.md) conforman la familia de tamaño de computación confidencial más reciente. Estas máquinas virtuales admiten una gama más amplia de capacidades de implementación, tienen el doble de Enclave Page Cache (EPC) y una mayor selección de tamaños en comparación con las máquinas virtuales de la serie DC. Las máquinas virtuales de la [serie DC](../virtual-machines/sizes-previous-gen.md#preview-dc-series) están actualmente en versión preliminar, pero se dejarán de usar y no se incluirán en la disponibilidad general.

Comience a implementar una máquina virtual de la serie DCsv2 a través del Marketplace comercial de Microsoft siguiendo el [tutorial de inicio rápido](quick-create-marketplace.md).

### <a name="current-available-sizes-and-regions"></a>Tamaños y regiones disponibles actualmente

Para obtener una lista de todos los tamaños de máquina virtual de proceso confidencial disponibles con carácter general en las regiones disponibles y en las zonas de disponibilidad, ejecute el siguiente comando en la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli-windows?view=azure-cli-latest):

```azurecli-interactive
az vm list-skus `
    --size dc `
    --query "[?family=='standardDCSv2Family'].{name:name,locations:locationInfo[0].location,AZ_a:locationInfo[0].zones[0],AZ_b:locationInfo[0].zones[1],AZ_c:locationInfo[0].zones[2]}" `
    --all `
    --output table
```

Para obtener una vista más detallada de los tamaños anteriores, ejecute el siguiente comando:

```azurecli-interactive
az vm list-skus `
    --size dc `
    --query "[?family=='standardDCSv2Family']"
```
### <a name="dedicated-host-requirements"></a>Requisitos de host dedicados
La implementación de un tamaño de máquina virtual **Standard_DC8_v2** en la familia de máquinas virtuales de la serie DCSv2 ocupará el host completo y no se compartirá con otros inquilinos o suscripciones. Esta familia de SKU de máquina virtual proporciona el aislamiento que puede necesitar para cumplir los requisitos de cumplimiento normativo y seguridad que normalmente se cumplen con un servicio de host dedicado. Al elegir la SKU **Standard_DC8_v2**, el servidor de host físico asignará todos los recursos de hardware disponibles, incluida la memoria EPC, solo a la máquina virtual. Tenga en cuenta que esta funcionalidad existe debido al diseño de la infraestructura y que se admitirán todas las características de **Standard_DC8_v2**. Esta implementación no es la misma que el servicio [Azure Dedicated Host](https://docs.microsoft.com/azure/virtual-machines/windows/dedicated-hosts) que proporcionan otras familias de máquinas virtuales de Azure.


## <a name="deployment-considerations"></a>Consideraciones de la implementación

Siga un tutorial de inicio rápido para implementar una máquina virtual de la serie DCsv2 en menos de 10 minutos. 

- **Suscripción de Azure**: para implementar una instancia de máquina virtual de computación confidencial, considere la posibilidad de usar una suscripción de pago por uso u otra opción de compra. Si usa una [cuenta gratuita de Azure](https://azure.microsoft.com/free/), no tendrá cuota para la cantidad adecuada de núcleos de proceso de Azure.

- **Precios y disponibilidad regional**: encuentre los precios de las máquinas virtuales de la serie DCsv2 en la [página de precios de máquinas virtuales](https://azure.microsoft.com/pricing/details/virtual-machines/linux/). Para ver la disponibilidad en las regiones de Azure, consulte [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines) .


- **Cuota de núcleos**: quizás tenga que aumentar la cuota de núcleos de su suscripción de Azure partiendo del valor predeterminado. La suscripción también podría limitar el número de núcleos que se pueden implementar en ciertas familias de tamaño de máquina virtual, como la serie DCsv2. Para solicitar un aumento de cuota, [abra una solicitud de soporte técnico al cliente en línea](https://docs.microsoft.com/azure/azure-portal/supportability/per-vm-quota-requests) sin cargo alguno. Tenga en cuenta que los límites predeterminados pueden variar según la categoría de suscripción.

  > [!NOTE]
  > Si tiene necesidades de capacidad a gran escala, póngase en contacto con el soporte técnico de Azure. Las cuotas de Azure son límites de crédito, no garantías de capacidad. Independientemente de la cuota, solamente se le cobrarán los núcleos que use.
  
- **Cambio de tamaño**: debido a su hardware especializado, solo se puede cambiar el tamaño de las instancias de computación confidencial dentro de la misma familia de tamaño. Por ejemplo, una máquina virtual de la serie DCsv2 solo se puede cambiar de un tamaño de serie DCsv2 a otro. No se admite el cambio de un tamaño de computación no confidencial a un tamaño de computación confidencial.  

- **Imagen**: para proporcionar compatibilidad con Intel Software Guard Extension (Intel SGX) en instancias de proceso confidenciales, todas las implementaciones deben ejecutarse en imágenes de generación 2. La computación confidencial de Azure admite cargas de trabajo que se ejecutan en Ubuntu 18.04 Gen 2, Ubuntu 16.04 Gen 2, Windows Server 2019 gen2 y Windows Server 2016 Gen 2. Lea acerca de la [compatibilidad de con las máquinas virtuales de generación 2 en Azure](../virtual-machines/linux/generation-2.md) para obtener más información sobre los escenarios admitidos y no admitidos. 

- **Almacenamiento**: los discos de datos de la máquina virtual de computación confidencial de Azure y nuestros discos del sistema operativo efímeros se encuentran en discos de NVMe. Las instancias solo admiten discos SSD Premium y SSD estándar, no SSD Ultra ni HDD estándar. El tamaño de máquina virtual **DC8_v2** no es compatible con Premium Storage. 

- **Cifrado de disco**: las instancias de proceso confidenciales no admiten el cifrado de disco en este momento. 

## <a name="high-availability-and-disaster-recovery-considerations"></a>Consideraciones sobre la alta disponibilidad y la recuperación ante desastres

Si usa máquinas virtuales en Azure, es responsable de implementar una solución de recuperación ante desastres y alta disponibilidad para evitar tiempo de inactividad. 

La computación confidencial de Azure no admite la redundancia de zona a través de Availability Zones en este momento. Para obtener la máxima disponibilidad y redundancia para la computación confidencial, use [conjuntos de disponibilidad](../virtual-machines/windows/manage-availability.md#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy). Debido a las restricciones de hardware, los conjuntos de disponibilidad para las instancias de computación confidencial solo pueden tener un máximo de 10 dominios de actualización. 

## <a name="deployment-with-azure-resource-manager-arm-template"></a>Implementación con plantilla de Azure Resource Manager (ARM)

Azure Resource Manager es el servicio de implementación y administración para Azure. Proporciona una capa de administración que le permite crear, actualizar y eliminar recursos de su suscripción de Azure. Puede usar las características de administración, como el control de acceso, la auditoría y las etiquetas, para proteger y organizar los recursos después de la implementación.

Para información sobre las plantillas de Resource Manager, consulte [Información general de Template Deployment](../azure-resource-manager/templates/overview.md).

Para implementar una máquina virtual de la serie DCsv2 en una plantilla de Resource Manager, utilizará el [recurso de máquina virtual](../virtual-machines/windows/template-description.md). Asegúrese de que especifica las propiedades correctas para **vmSize** y para **imageReference**.

### <a name="vm-size"></a>Tamaño de VM

Especifique uno de los siguientes tamaños en la plantilla de ARM del recurso de máquina virtual. Esta cadena se coloca como **vmSize** en las **propiedades**.

```json
  [
        "Standard_DC1s_v2",
        "Standard_DC2s_v2",
        "Standard_DC4s_v2",
        "Standard_DC8_v2"
      ],
```

### <a name="gen2-os-image"></a>Imagen de sistema operativo Gen2

En **Propiedades**, también tendrá que hacer referencia a una imagen en **storageProfile**. Use *solo una* de las siguientes imágenes para su valor de **imageReference**.

```json
      "2019-datacenter-gensecond": {
        "offer": "WindowsServer",
        "publisher": "MicrosoftWindowsServer",
        "sku": "2019-datacenter-gensecond",
        "version": "latest"
      },
      "2016-datacenter-gensecond": {
        "offer": "WindowsServer",
        "publisher": "MicrosoftWindowsServer",
        "sku": "2016-datacenter-gensecond",
        "version": "latest"
      },
        "18_04-lts-gen2": {
        "offer": "UbuntuServer",
        "publisher": "Canonical",
        "sku": "18_04-lts-gen2",
        "version": "latest"
      },
      "16_04-lts-gen2": {
        "offer": "UbuntuServer",
        "publisher": "Canonical",
        "sku": "16_04-lts-gen2",
        "version": "latest"
      }
```

## <a name="next-steps"></a>Pasos siguientes 

En este artículo ha aprendido sobre las calificaciones y las configuraciones necesarias para crear una máquina virtual de computación confidencial. Ahora puede visitar Microsoft Azure Marketplace para implementar una máquina virtual de la serie DCsv2.

> [!div class="nextstepaction"]
> [Implementación de una máquina virtual de la serie DCsv2 en Azure Marketplace](quick-create-marketplace.md)
