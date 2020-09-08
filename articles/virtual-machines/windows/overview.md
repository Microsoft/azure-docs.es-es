---
title: Introducción a las máquinas virtuales Windows en Azure
description: Información general sobre las máquinas virtuales Windows en Azure
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: overview
ms.date: 11/14/2019
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 4e955a51906f160264fc4e81f263fe7677be91de
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88589629"
---
# <a name="windows-virtual-machines-in-azure"></a>Máquinas virtuales Windows en Azure

Azure Virtual Machines (VM) es uno de los diversos tipos de [recursos informáticos a petición y escalables](/azure/architecture/guide/technology-choices/compute-decision-tree) que ofrece Azure. Por lo general, elegirá una máquina virtual cuando necesite más control sobre su entorno informático del que ofrecen las otras opciones. En este artículo se proporciona información sobre lo que debe considerar antes de crear una máquina virtual, cómo crearla y cómo administrarla.

Una máquina virtual de Azure le ofrece la flexibilidad de la virtualización sin necesidad de adquirir y mantener el hardware físico que la ejecuta. Sin embargo, aún necesita mantener la máquina virtual con tareas como configurar, aplicar revisiones e instalar el software que se ejecuta en ella.

Las máquinas virtuales de Azure se pueden usar de diversas maneras. Ejemplos:

* **Desarrollo y pruebas**: las máquinas virtuales de Azure ofrecen una manera rápida y sencilla de crear un equipo con configuraciones específicas necesarias para codificar y probar una aplicación.
* **Aplicaciones en la nube**: como la demanda de la aplicación puede fluctuar, tendría sentido desde el punto de vista económico ejecutarla en una máquina virtual en Azure. Paga por las máquinas virtuales adicionales cuando las necesite y las desactiva cuando ya no sean necesarias.
* **Centro de datos ampliado**: las máquinas virtuales de una red virtual de Azure se pueden conectar fácilmente a la red de su organización.

El número de máquinas virtuales usadas por su aplicación se puede escalar vertical y horizontalmente a la cifra necesaria para satisfacer sus necesidades.

## <a name="what-do-i-need-to-think-about-before-creating-a-vm"></a>¿Qué hay que considerar antes de crear una máquina virtual?
Siempre hay gran cantidad de [consideraciones de diseño](/azure/architecture/reference-architectures/n-tier/windows-vm) cuando se crea una infraestructura de aplicaciones en Azure. Es importante pensar en estos aspectos de una máquina virtual antes de empezar:

* Los nombres de los recursos de la aplicación
* La ubicación donde se almacenan los recursos
* El tamaño de la máquina virtual
* El número máximo de máquinas virtuales que se pueden crear
* El sistema operativo que ejecuta la máquina virtual
* La configuración de la máquina virtual después de iniciarse
* Los recursos relacionados que necesita la máquina virtual

### <a name="locations"></a>Ubicaciones
Todos los recursos creados en Azure se distribuyen entre diversas [regiones geográficas](https://azure.microsoft.com/regions/) de todo el mundo. Por lo general, se llama a la región **ubicación** cuando se crea una máquina virtual. Para una máquina virtual, la ubicación especifica dónde se almacenan los discos duros virtuales.

En esta tabla se muestran algunas de las formas en que puede obtener una lista de ubicaciones disponibles.

| Método | Descripción |
| --- | --- |
| Azure portal |Seleccione una ubicación en la lista cuando cree una máquina virtual. |
| Azure PowerShell |Use el comando [Get-AzLocation](/powershell/module/az.resources/get-azlocation). |
| API DE REST |Use la operación para [mostrar la lista de ubicaciones](/rest/api/resources/subscriptions). |
| Azure CLI |Use la operación[az account list-locations](/cli/azure/account?view=azure-cli-latest). |

### <a name="singapore-data-residency"></a>Residencia de datos en Singapur

En Azure, la característica que permite almacenar los datos de clientes en una sola región solo está disponible actualmente en la región de Sudeste Asiático (Singapur) de la geoárea Asia Pacífico. En todas las demás regiones, los datos del cliente se almacenan en la geoárea. Para más información, consulte el [Centro de confianza](https://azuredatacentermap.azurewebsites.net/).

## <a name="availability"></a>Disponibilidad
Azure anunció un Acuerdo de Nivel de Servicio líder de la industria de máquinas virtuales de una sola instancia del 99,9 % siempre y cuando la máquina virtual se implemente con Premium Storage en todos los discos.  Para que su implementación pueda optar al Acuerdo de Nivel de Servicio estándar de máquina virtual del 99,95 %, debe implementar dos o más máquinas virtuales que ejecuten la carga de trabajo dentro de un conjunto de disponibilidad. Un conjunto de disponibilidad garantiza que las máquinas virtuales se distribuyen en varios dominios de error de los centros de datos de Azure y que se implementan en hosts con diferentes ventanas de mantenimiento. En el [SLA de Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/) completo se explica la disponibilidad garantizada de Azure como un conjunto.


## <a name="vm-size"></a>Tamaño de VM
El [tamaño](../sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) de la máquina virtual que use depende de la carga de trabajo que vaya a ejecutar. El tamaño que elija determina factores tales como la capacidad de almacenamiento, la memoria y la capacidad de procesamiento. Azure ofrece una amplia variedad de tamaños para admitir muchos tipos de usos.

Azure cobra un [precio por hora](https://azure.microsoft.com/pricing/details/virtual-machines/windows/) en función del tamaño y el sistema operativo de la máquina virtual. Para las fracciones de hora, solo cobra los minutos usados. El precio del almacenamiento se calcula y se cobra por separado.

## <a name="vm-limits"></a>Límites de máquina virtual
Su suscripción tiene [límites de cuota](../../azure-resource-manager/management/azure-subscription-service-limits.md) predeterminados que pueden afectar a la implementación de numerosas máquinas virtuales en su proyecto. El límite actual por suscripción es 20 máquinas virtuales por región. Para aumentar estos límites, [cree una incidencia de soporte técnico y solicite un aumento](../../azure-portal/supportability/resource-manager-core-quotas-request.md)

### <a name="operating-system-disks-and-images"></a>Imágenes y discos del sistema operativo
Las máquinas virtuales usan [discos duros virtuales (VHD)](managed-disks-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) para almacenar el sistema operativo y los datos. Estos discos también se usan para las imágenes entre las que se puede elegir para instalar un sistema operativo. 

Azure proporciona muchas [imágenes de Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps?filters=virtual-machine-images%3Bwindows&page=1) que se pueden usar con diversas versiones y tipos de sistemas operativos Windows Server. Las imágenes de Marketplace se identifican mediante el publicador de la imagen, la oferta, la SKU y la versión (normalmente, la versión se especifica como la más reciente). Solo se admiten los sistemas operativos de 64 bits. Para más información sobre los sistemas operativos invitados admitidos, roles y características, consulte [Soporte de software de servidor de Microsoft para las máquinas virtuales de Microsoft Azure](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines).

En esta tabla se muestran algunas maneras de encontrar la información sobre una imagen.

| Método | Descripción |
| --- | --- |
| Azure portal |Los valores se especifican automáticamente cuando se selecciona una imagen para usarla. |
| Azure PowerShell |[Get-AzVMImagePublisher](/powershell/module/az.compute/get-azvmimagepublisher) -Location *location*<BR>[Get-AzVMImageOffer](/powershell/module/az.compute/get-azvmimageoffer) -Location *location* -Publisher *publisherName*<BR>[Get-AzVMImageSku](/powershell/module/az.compute/get-azvmimagesku) -Location *location* -Publisher *publisherName* -Offer *offerName* |
| API de REST |[List image publishers](/rest/api/compute/platformimages/platformimages-list-publishers) (Lista de publicadores de imágenes)<BR>[List image offers](/rest/api/compute/platformimages/platformimages-list-publisher-offers) (Lista de ofertas de imágenes)<BR>[List image skus](/rest/api/compute/platformimages/platformimages-list-publisher-offer-skus) (Lista de SKU de imágenes) |
| Azure CLI |[az vm image list-publishers](/cli/azure/vm/image?view=azure-cli-latest) --location *ubicación*<BR>[az vm image list-offers](/cli/azure/vm/image?view=azure-cli-latest) --location *location* --publisher *publisherName*<BR>[az vm image list-skus](/cli/azure/vm?view=azure-cli-latest) --location *location* --publisher *publisherName* --offer *offerName*|

Puede elegir [cargar y usar su propia imagen](upload-generalized-managed.md) y, cuando lo haga, no se usan el nombre del publicador, la oferta ni la SKU.

### <a name="extensions"></a>Extensiones
Las [extensiones](../extensions/features-windows.md?toc=/azure/virtual-machines/windows/toc.json) de máquina virtual ofrecen funcionalidades adicionales de máquina virtual por medio de la configuración posterior a la implementación y tareas automatizadas.

Pueden llevarse a cabo estas tareas comunes mediante las extensiones:

* **Ejecutar scripts personalizados**: la [extensión de script personalizado](../extensions/custom-script-windows.md?toc=/azure/virtual-machines/windows/toc.json) ayuda a configurar cargas de trabajo en la máquina virtual al ejecutar su script cuando se aprovisiona la máquina virtual.
* **Implementar y administrar configuraciones**: la [extensión de configuración de estado deseado (DSC) de PowerShell](../extensions/dsc-overview.md?toc=/azure/virtual-machines/windows/toc.json) ayuda a configurar DSC en una máquina virtual para administrar entornos y configuraciones.
* **Recopilar datos de diagnóstico**: la [extensión Diagnósticos de Azure](../extensions/diagnostics-template.md?toc=/azure/virtual-machines/windows/toc.json) ayuda a configurar la máquina virtual para que recopile datos de diagnóstico que sirvan para supervisar el estado de la aplicación.

### <a name="related-resources"></a>Recursos relacionados
Los recursos de esta tabla se usan en la máquina virtual y deben ya existir o crearse al tiempo que la máquina virtual.

| Resource | Obligatorio | Descripción |
| --- | --- | --- |
| [Grupos de recursos](../../azure-resource-manager/management/overview.md) |Sí |La máquina virtual debe encontrarse en un grupo de recursos. |
| [Cuenta de almacenamiento](../../storage/common/storage-account-create.md) |Sí |La máquina virtual necesita la cuenta de almacenamiento para almacenar sus discos duros virtuales. |
| [Red virtual](../../virtual-network/virtual-networks-overview.md) |Sí |La máquina virtual debe ser miembro de una red virtual. |
| [Dirección IP pública](../../virtual-network/public-ip-addresses.md) |No |La máquina virtual puede tener una dirección IP pública asignada para acceder remotamente a ella. |
| [Interfaz de red](../../virtual-network/virtual-network-network-interface.md) |Sí |La máquina virtual necesita la interfaz de red para comunicarse en la red. |
| [Discos de datos](attach-managed-disk-portal.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) |No |La máquina virtual puede incluir discos de datos para ampliar las funcionalidades de almacenamiento. |

## <a name="next-steps"></a>Pasos siguientes

Creación de la primera máquina virtual

- [Portal](quick-create-portal.md)
- [PowerShell](quick-create-powershell.md)
- [CLI de Azure](quick-create-cli.md)
