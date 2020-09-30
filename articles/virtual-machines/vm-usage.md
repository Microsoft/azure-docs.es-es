---
title: Descripción de uso de máquinas virtuales de Azure
description: Descripción de los detalles de uso de máquinas virtuales
services: virtual-machines
documentationcenter: ''
author: mimckitt
ms.author: mimckitt
ms.service: virtual-machines-linux
ms.topic: how-to
ms.tgt_pltfrm: vm
ms.workload: infrastructure-services
ms.date: 07/28/2020
ms.openlocfilehash: d43f94d3555a660d6b7c8f755eebfec253d31dc2
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89322835"
---
# <a name="understanding-azure-virtual-machine-usage"></a>Descripción de uso de máquinas virtuales de Azure
Mediante el análisis de los datos de uso de Azure, es posible obtener información importante sobre el consumo, es decir, información que puede permitirle mejorar la asignación y administración de los costos en toda la organización. En este documento se profundiza en los detalles de consumo de Azure Compute. Para más detalles sobre el uso general de Azure, navegue a [Descripción de la factura](../cost-management-billing/understand/review-individual-bill.md).

## <a name="download-your-usage-details"></a>Descarga de los detalles de uso
Para comenzar, [descargue los detalles de uso](../cost-management-billing/manage/download-azure-invoice-daily-usage-date.md#download-usage-in-azure-portal). En la tabla siguiente se proporciona la definición y valores de ejemplo de uso de las máquinas virtuales implementadas a través de Azure Resource Manager. Este documento no contiene información detallada de las máquinas virtuales implementadas a través del modelo clásico.


| Campo | Significado | Valores de ejemplo | 
|---|---|---|
| Fecha de uso | La fecha en que se usó el recurso. | `11/23/2017` |
| Id. de medidor | Identifica el servicio de nivel superior al que pertenece este uso.| `Virtual Machines`|
| Subcategoría de medidor | El identificador del medidor facturado. <br><br> Si desea conocer el uso de las horas de proceso, existe un medidor para cada tamaño de máquina virtual + SO (Windows, no Windows) + región. <br><br> Si desea conocer el uso de software Premium, existe un medidor para cada tipo de software. La mayoría de las imágenes de software Premium tienen distintos medidores para cada tamaño de núcleo. Para más información, visite la [página de precios de Compute](https://azure.microsoft.com/pricing/details/virtual-machines/).</li></ul>| `2005544f-659d-49c9-9094-8e0aea1be3a5`|
| Nombre de medidor| Es específico para cada servicio de Azure. En el caso de Compute, siempre es "Horas de proceso".| `Compute Hours`|
| Medidor de la región| Identifica la ubicación del centro de datos para ciertos servicios cuyos precios se establecen según la ubicación del centro de datos.|  `JA East`|
| Unidad| Identifica la unidad en que se cobra el servicio. Los recursos de Compute se facturan por hora.| `Hours`|
| Consumida| La cantidad del recurso consumida durante ese día. Para Compute, se facturará por cada minuto de ejecución de la máquina virtual durante una hora determinada (hasta seis decimales de precisión).| `1, 0.5`|
| Ubicación del recurso  | Identifica el centro de datos donde se está ejecutando el recurso.| `JA East`|
| Servicio consumido | El servicio de la plataforma Azure que ha usado.| `Microsoft.Compute`|
| Grupo de recursos | El grupo de recursos en el que se ejecuta el recurso implementado. Para más información, consulte [Información general de Azure Resource Manager](../azure-resource-manager/management/overview.md).|`MyRG`|
| Id. de instancia | Identificador del recurso. El identificador contiene el nombre especificado para el recurso cuando se creó En el caso de las máquinas virtuales, el identificador de instancia contendrá los valores SubscriptionId, ResourceGroupName y VMName (o el nombre del conjunto de escalado para el uso de conjunto de escalado).| `/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/ resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachines/MyVM1`<br><br>or<br><br>`/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/ resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachineScaleSets/MyVMSS1`|
| Etiquetas| Etiqueta asignada al recurso. Use etiquetas para agrupar los registros de facturación. Obtenga información sobre cómo etiquetar las máquinas virtuales mediante la [CLI](./linux/tag.md) o [PowerShell](./windows/tag.md). Solo está disponible para máquinas virtuales de Resource Manager.| `{"myDepartment":"RD","myUser":"myName"}`|
| Información adicional | Metadatos específicos del servicio. En el caso de las VM, rellenamos los siguientes datos en el campo de información adicional: <br><br> Tipo de imagen: la imagen específica que ejecutó. Encuentre la lista completa de cadenas compatibles a continuación, en Tipos de imagen.<br><br> Tipo de servicio: el tamaño que implementó.<br><br> VMName: el nombre de la máquina virtual. Este campo solo se rellena para las VM del conjunto de escalado. Si necesita el nombre de la máquina virtual para las máquinas virtuales del conjunto de escalado, puede buscarlo en la cadena del identificador de instancia anterior.<br><br> UsageType: especifica el tipo de uso que representa.<br><br> ComputeHR es el uso de Horas de proceso de la máquina virtual subyacente, como Standard_D1_v2.<br><br> ComputeHR_SW es el cambio de software Premium si la máquina virtual usa software Premium, como Microsoft R Server. | Virtual Machines<br>`{"ImageType":"Canonical","ServiceType":"Standard_DS1_v2","VMName":"", "UsageType":"ComputeHR"}`<br><br>Virtual Machine Scale Sets<br> `{"ImageType":"Canonical","ServiceType":"Standard_DS1_v2","VMName":"myVM1", "UsageType":"ComputeHR"}`<br><br>Software Premium<br> `{"ImageType":"","ServiceType":"Standard_DS1_v2","VMName":"", "UsageType":"ComputeHR_SW"}` |

## <a name="image-type"></a>Tipo de imagen
Para algunas imágenes de la Galería de Azure, el tipo de imagen se rellena en el campo de información adicional. Esto permite que los usuarios comprendan y hagan un seguimiento de lo que implementaron en su máquina virtual. Los siguientes valores que se rellenan en este campo según la imagen que implementó son los siguientes:
- BitRock 
- Canonical FreeBSD 
- Open Logic 
- Oracle 
- SLES para SAP 
- SQL Server 14 Preview en Windows Server 2012 R2 Preview 
- SUSE
- SUSE Premium
- StorSimple Cloud Appliance 
- Red Hat
- Red Hat for SAP Business Applications     
- Red Hat for SAP HANA 
- BYOL de cliente Windows 
- Windows Server BYOL 
- Versión preliminar de Windows Server 

## <a name="service-type"></a>Tipo de servicio
El campo de tipo de servicio en el campo Información adicional corresponde al tamaño exacto de la máquina virtual que implementó. Las máquinas virtuales de almacenamiento Premium (basadas en SSD) y las máquinas virtuales de almacenamiento no Premium (basadas en HDD) tienen el mismo precio. Si implementa un tamaño basado en SSD, como Standard\_DS2\_v2, verá el tamaño no SSD (`Standard\_D2\_v2 VM`) en la columna Subcategoría de medidor y el tamaño SSD (`Standard\_DS2\_v2`) en el campo Información adicional.

## <a name="region-names"></a>Nombres de región
El nombre de región que se rellenó en el campo Ubicación de recurso en los detalles de uso es distinto del nombre de la región que se usa en Azure Resource Manager. A continuación se muestra una asignación entre los valores de región:

| **Nombre de región de Resource Manager** | **Ubicación de los recursos en los detalles de uso** |
|---|---|
| australiaeast |Este de Australia|
| australiasoutheast | Sudeste de Australia|
| brazilsouth | Sur de Brasil|
| CanadaCentral | Centro de Canadá|
| CanadaEast | Este de Canadá|
| CentralIndia | Centro de la India|
| centralus | Centro de EE. UU.|
| chinaeast | Este de China|
| chinanorth | Norte de China|
| eastasia | Este de Asia|
| estado | Este de EE. UU.|
| eastus2 | Este de EE. UU. 2|
| GermanyCentral | Centro de Alemania|
| GermanyNortheast | Norte de Alemania|
| japaneast | Este de Japón|
| japanwest | Oeste de Japón|
| KoreaCentral | Centro de Corea del Sur|
| KoreaSouth | Sur de Corea del Sur|
| northcentralus | Centro-Norte de EE. UU|
| northeurope | Norte de Europa|
| southcentralus | Centro-sur de EE. UU.|
| southeastasia | Sudeste de Asia|
| SouthIndia | India meridional|
| UKNorth | Norte de Reino Unido|
| uksouth | Sur de Reino Unido 2|
| UKSouth2 | Sur de Reino Unido 2|
| ukwest | Oeste de Reino Unido|
| USDoDCentral | US DoD (centro)|
| USDoDEast | US DoD (este)|
| USGovArizona | USGov: Arizona|
| usgoviowa | USGov Iowa|
| USGovTexas | USGov: Texas|
| usgovvirginia | USGov Virginia|
| westcentralus | Centro-oeste de EE. UU.|
| westeurope | Oeste de Europa|
| WestIndia | India occidental|
| westus | Oeste de EE. UU.|
| westus2 | Oeste de EE. UU. 2|


## <a name="virtual-machine-usage-faq"></a>Preguntas más frecuentes sobre el uso de las máquinas virtuales
### <a name="what-resources-are-charged-when-deploying-a-vm"></a>¿Qué recursos se cobran cuando se implementa una máquina virtual?    
Las máquinas virtuales adquieren los costos de la misma máquina virtual, de cualquier software Premium que se ejecute en la máquina virtual, del disco administrado o la cuenta de almacenamiento asociados con la máquina virtual y las transferencias de ancho de banda de red desde la máquina virtual.
### <a name="how-can-i-tell-if-a-vm-is-using-azure-hybrid-benefit-in-the-usage-csv"></a>¿Cómo se puede saber si una máquina virtual usa Ventaja híbrida de Azure en el CSV de uso?
Si realiza la implementación con [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/), se le cobra la tarifa de máquina virtual no de Windows porque trae su propia licencia a la nube. En la factura, puede distinguir qué máquinas virtuales de Resource Manager ejecutan Ventaja híbrida de Azure porque en la columna ImageType aparece "BYOL de Windows\_Server" o "BYOL de cliente\_Windows".
### <a name="how-are-basic-vs-standard-vm-types-differentiated-in-the-usage-csv"></a>¿Cómo se diferencian los tipos de máquina virtual básica y estándar en el CSV de uso?
Se ofrecen máquinas virtuales básica y estándar de la serie A. Si implementa una máquina virtual básica, tiene la cadena "Basic" en la subcategoría de medidor. Si implementa una máquina virtual estándar de la serie A, el tamaño de máquina virtual aparece como "A1 VM" porque Estándar es el valor predeterminado. Para más información sobre las diferencias entre las máquinas virtuales básica y estándar, consulte la [página de precios](https://azure.microsoft.com/pricing/details/virtual-machines/).
### <a name="what-are-extrasmall-small-medium-large-and-extralarge-sizes"></a>¿Qué son los tamaños ExtraSmall, Small, Medium, Large y ExtraLarge?
ExtraSmall y ExtraLarge son los nombres heredados para Estándar\_A0 – Estándar\_A4. En los registros de uso de máquinas virtuales clásicas, puede ver que esta convención se usa si implementó estos tamaños.
### <a name="what-is-the-difference-between-meter-region-and-resource-location"></a>¿Cuál es la diferencia entre la región de medidor y la ubicación de recursos?
La región de medidor está asociada con el medidor. En algunos servicios de Azure que usan un precio para todas las regiones, el campo Región de medidor podría estar en blanco. Sin embargo, como las máquinas virtuales tienen precios dedicados por región para las máquinas virtuales, este campo se rellena. Del mismo modo, la ubicación de recursos de las máquinas virtuales es la ubicación donde se implementa la máquina virtual. Las regiones de Azure de ambos campos son las mismas, si bien pueden tener una convención de cadena distinta para el nombre de la región.
### <a name="why-is-the-imagetype-value-blank-in-the-additional-info-field"></a>¿Por qué el valor ImageType está en blanco en el campo de información adicional?
El campo ImageType se rellena solo para un subconjunto de imágenes. Si no implementó una de las imágenes anteriores, el valor de ImageType está en blanco.
### <a name="why-is-the-vmname-blank-in-the-additional-info"></a>¿Por qué el valor VMName está en blanco en el campo de información adicional?
El valor VMName solo se rellena en el campo de información adicional de las máquinas virtuales de un conjunto de escalado. El campo InstanceID contiene el nombre de máquina virtual de las máquinas virtuales que no son de un conjunto de escalado.
### <a name="what-does-computehr-mean-in-the-usagetype-field-in-the-additional-info"></a>¿Qué significa ComputeHR en el campo UsageType en el campo de información adicional?
ComputeHR se refiere a la hora de proceso, que representa el evento de uso para el costo de infraestructura subyacente. Si UsageType es ComputeHR\_SW, el evento de uso representa el cargo de software Premium de la máquina virtual.
### <a name="how-do-i-know-if-i-am-charged-for-premium-software"></a>¿Cómo puedo saber si me cobran por software Premium?
Cuando explore qué imagen de máquina virtual se adapta mejor a sus necesidades, asegúrese de revisar [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/compute). La imagen tiene la tarifa del plan de software. Si la tarifa aparece como "Gratis", no hay costo adicional por el software. 
### <a name="what-is-the-difference-between-microsoftclassiccompute-and-microsoftcompute-in-the-consumed-service"></a>¿Cuál es la diferencia entre Microsoft.ClassicCompute y Microsoft.Compute en el servicio consumido?
Microsoft.ClassicCompute representa los recursos clásicos implementados a través de Azure Service Manager. Si realiza la implementación a través de Resource Manager, Microsoft.Compute se rellena en el servicio consumido. Obtenga más información sobre los [modelos de implementación de Azure](../azure-resource-manager/management/deployment-models.md).
### <a name="why-is-the-instanceid-field-blank-for-my-virtual-machine-usage"></a>¿Por qué el campo InstanceID está en blanco en el uso de mi máquina virtual?
Si implementa a través del modelo de implementación clásica, la cadena InstanceID no está disponible.
### <a name="why-are-the-tags-for-my-vms-not-flowing-to-the-usage-details"></a>¿Por qué las etiquetas de mis máquinas virtuales no se transmiten a los detalles de uso?
Las etiquetas solo se transmiten al CSV de uso para las máquinas virtuales de Resource Manager. Las etiquetas de recursos clásicos no están disponibles en los detalles de uso.
### <a name="how-can-the-consumed-quantity-be-more-than-24-hours-one-day"></a>¿Cómo la cantidad consumida puede ser más de 24 horas al día?
En el modelo clásico, la facturación de recursos se agrega en el nivel de servicio en la nube. Si tiene más de una máquina virtual en un servicio en la nube que usa el mismo medidor de facturación, el uso se agrega en conjunto. Las máquinas virtuales que se implementan a través de Resource Manager se facturan en el nivel de máquina virtual, por lo que no se aplicará esta agregación.
### <a name="why-is-pricing-not-available-for-dsfsgsls-sizes-on-the-pricing-page"></a>¿Por qué en la página de precios no aparecen los precios de los tamaños DS/FS/GS/LS?
Las máquinas virtuales compatibles con el almacenamiento Premium se facturan con la misma tarifa que las máquinas virtuales compatibles con el almacenamiento que no es Premium. Solo difieren los costos de almacenamiento. Visite la [página de precios de almacenamiento](https://azure.microsoft.com/pricing/details/storage/unmanaged-disks/) para más información.

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre los detalles de uso, consulte [Comprender la factura de Microsoft Azure](../cost-management-billing/understand/review-individual-bill.md).
