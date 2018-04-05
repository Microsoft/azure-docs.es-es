---
title: Información general sobre las máquinas virtuales Windows | Microsoft Docs
description: Aprenda a crear y administrar máquinas virtuales con Windows en Azure.
services: virtual-machines-windows
documentationcenter: ''
author: iainfoulds
manager: jeconnoc
editor: ''
tags: azure-resource-manager,azure-service-management
ms.assetid: fbae9c8e-2341-4ed0-bb20-fd4debb2f9ca
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: get-started-article
ms.date: 07/17/2017
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: 6ab964760ea06c2e82d0c9f69f89fe62dec8ceac
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/23/2018
---
# <a name="overview-of-windows-virtual-machines-in-azure"></a>Información general sobre las máquinas virtuales Windows en Azure

Azure Virtual Machines (VM) es uno de los diversos tipos de [recursos informáticos a petición y escalables](../../app-service/choose-web-site-cloud-service-vm.md) que ofrece Azure. Por lo general, elegirá una máquina virtual cuando necesite más control sobre su entorno informático del que ofrecen las otras opciones. En este artículo se proporciona información sobre lo que debe considerar antes de crear una máquina virtual, cómo crearla y cómo administrarla.

Una máquina virtual de Azure le ofrece la flexibilidad de la virtualización sin necesidad de adquirir y mantener el hardware físico que la ejecuta. Sin embargo, aún necesita mantener la máquina virtual con tareas como configurar, aplicar revisiones e instalar el software que se ejecuta en ella.

Las máquinas virtuales de Azure se pueden usar de diversas maneras. A continuación, se indican algunos ejemplos:

* **Desarrollo y pruebas**: las máquinas virtuales de Azure ofrecen una manera rápida y sencilla de crear un equipo con configuraciones específicas necesarias para codificar y probar una aplicación.
* **Aplicaciones en la nube**: como la demanda de la aplicación puede fluctuar, tendría sentido desde el punto de vista económico ejecutarla en una máquina virtual en Azure. Paga por las máquinas virtuales adicionales cuando las necesite y las desactiva cuando ya no sean necesarias.
* **Centro de datos ampliado**: las máquinas virtuales de una red virtual de Azure se pueden conectar fácilmente a la red de su organización.

El número de máquinas virtuales usadas por su aplicación se puede escalar vertical y horizontalmente a la cifra necesaria para satisfacer sus necesidades.

## <a name="what-do-i-need-to-think-about-before-creating-a-vm"></a>¿Qué hay que considerar antes de crear una máquina virtual?
Siempre hay gran cantidad de [consideraciones de diseño](/azure/architecture/reference-architectures/virtual-machines-windows?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) cuando se crea una infraestructura de aplicaciones en Azure. Es importante pensar en estos aspectos de una máquina virtual antes de empezar:

* Los nombres de los recursos de la aplicación
* La ubicación donde se almacenan los recursos
* El tamaño de la máquina virtual
* El número máximo de máquinas virtuales que se pueden crear
* El sistema operativo que ejecuta la máquina virtual
* La configuración de la máquina virtual después de iniciarse
* Los recursos relacionados que necesita la máquina virtual

### <a name="naming"></a>Nomenclatura
Una máquina virtual tiene un [nombre](/azure/architecture/best-practices/naming-conventions#naming-rules-and-restrictions?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) asignado y un nombre de equipo configurado como parte del sistema operativo. El nombre de una máquina virtual puede tener hasta 15 caracteres.

Si usa Azure para crear el disco de sistema operativo, el nombre del equipo y el nombre de la máquina virtual son iguales. Si [carga y usa su propia imagen](upload-generalized-managed.md) que contenga un sistema operativo ya configurado y la usa para crear una máquina virtual, los nombres pueden ser diferentes. Se recomienda que, cuando cargue su propio archivo de imagen, haga que el nombre del equipo en el sistema operativo y el nombre de la máquina virtual sean iguales.

### <a name="locations"></a>Ubicaciones
Todos los recursos creados en Azure se distribuyen entre diversas [regiones geográficas](https://azure.microsoft.com/regions/) de todo el mundo. Por lo general, se llama a la región **ubicación** cuando se crea una máquina virtual. Para una máquina virtual, la ubicación especifica dónde se almacenan los discos duros virtuales.

En esta tabla se muestran algunas de las formas en que puede obtener una lista de ubicaciones disponibles.

| Método | DESCRIPCIÓN |
| --- | --- |
| Azure Portal |Seleccione una ubicación en la lista cuando cree una máquina virtual. |
| Azure PowerShell |Use el comando [Get-AzureRmLocation](/powershell/module/azurerm.resources/get-azurermlocation). |
| API DE REST |Use la operación para [mostrar la lista de ubicaciones](https://docs.microsoft.com/rest/api/resources/subscriptions#Subscriptions_ListLocations). |
| Azure CLI |Use la operación[az account list-locations](https://docs.microsoft.com/en-us/cli/azure/account?view=azure-cli-latest#az_account_list_locations). |

### <a name="vm-size"></a>Tamaño de VM
El [tamaño](sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) de la máquina virtual que use depende de la carga de trabajo que vaya a ejecutar. El tamaño que elija determina factores tales como la capacidad de almacenamiento, la memoria y la capacidad de procesamiento. Azure ofrece una amplia variedad de tamaños para admitir muchos tipos de usos.

Azure cobra un [precio por hora](https://azure.microsoft.com/pricing/details/virtual-machines/windows/) en función del tamaño y el sistema operativo de la máquina virtual. Para las fracciones de hora, solo cobra los minutos usados. El precio del almacenamiento se calcula y se cobra por separado.

### <a name="vm-limits"></a>Límites de máquina virtual
Su suscripción tiene [límites de cuota](../../azure-subscription-service-limits.md) predeterminados que pueden afectar a la implementación de numerosas máquinas virtuales en su proyecto. El límite actual por suscripción es 20 máquinas virtuales por región. Para aumentar estos límites, cree una incidencia de soporte técnico y solicite un aumento.

### <a name="operating-system-disks-and-images"></a>Imágenes y discos del sistema operativo
Las máquinas virtuales usan [discos duros virtuales (VHD)](about-disks-and-vhds.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) para almacenar el sistema operativo y los datos. Estos discos también se usan para las imágenes entre las que se puede elegir para instalar un sistema operativo. 

Azure proporciona muchas [imágenes de Marketplace](https://azure.microsoft.com/marketplace/virtual-machines/) que se pueden usar con diversas versiones y tipos de sistemas operativos Windows Server. Las imágenes de Marketplace se identifican mediante el publicador de la imagen, la oferta, la SKU y la versión (normalmente, la versión se especifica como la más reciente). 

En esta tabla se muestran algunas maneras de encontrar la información sobre una imagen.

| Método | DESCRIPCIÓN |
| --- | --- |
| Azure Portal |Los valores se especifican automáticamente cuando se selecciona una imagen para usarla. |
| Azure PowerShell |[Get-AzureRMVMImagePublisher](https://docs.microsoft.com/powershell/resourcemanager/azurerm.compute/v2.5.0/get-azurermvmimagepublisher) -Location "ubicación"<BR>[Get-AzureRMVMImageOffer](https://docs.microsoft.com/powershell/resourcemanager/azurerm.compute/v2.5.0/get-azurermvmimageoffer) -Location "ubicación" -Publisher "nombreDePublicador"<BR>[Get-AzureRMVMImageSku](/powershell/module/azurerm.compute/get-azurermvmimagesku) -Location "ubicación" -Publisher "nombreDePublicador" -Offer "nombreDeOferta" |
| API de REST |[List image publishers](https://docs.microsoft.com/rest/api/compute/platformimages/platformimages-list-publishers) (Lista de publicadores de imágenes)<BR>[List image offers](https://docs.microsoft.com/rest/api/compute/platformimages/platformimages-list-publisher-offers) (Lista de ofertas de imágenes)<BR>[List image skus](https://docs.microsoft.com/rest/api/compute/platformimages/platformimages-list-publisher-offer-skus) (Lista de SKU de imágenes) |
| Azure CLI |[az vm image list-publishers](https://docs.microsoft.com/en-us/cli/azure/vm/image?view=azure-cli-latest#az_vm_image_list_publishers) --location "location"<BR>[az vm image list-offers](https://docs.microsoft.com/en-us/cli/azure/vm/image?view=azure-cli-latest#az_vm_image_list_offers) --location "location" --publisher "publisherName"<BR>[az vm image list-skus](https://docs.microsoft.com/en-us/cli/azure/vm?view=azure-cli-latest#az_vm_list_skus) --location "location" --publisher "publisherName" --offer "offerName"|

Puede elegir [cargar y usar su propia imagen](upload-generalized-managed.md#upload-the-vhd-to-your-storage-account) y, cuando lo haga, no se usan el nombre del publicador, la oferta ni la SKU.

### <a name="extensions"></a>Extensiones
Las [extensiones](extensions-features.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) de máquina virtual ofrecen funcionalidades adicionales de máquina virtual por medio de la configuración posterior a la implementación y tareas automatizadas.

Pueden llevarse a cabo estas tareas comunes mediante las extensiones:

* **Ejecutar scripts personalizados**: la [extensión de script personalizado](extensions-customscript.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) ayuda a configurar cargas de trabajo en la máquina virtual al ejecutar su script cuando se aprovisiona la máquina virtual.
* **Implementar y administrar configuraciones**: la [extensión de configuración de estado deseado (DSC) de PowerShell](extensions-dsc-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) ayuda a configurar DSC en una máquina virtual para administrar entornos y configuraciones.
* **Recopilar datos de diagnóstico**: la [extensión Diagnósticos de Azure](extensions-diagnostics-template.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) ayuda a configurar la máquina virtual para que recopile datos de diagnóstico que sirvan para supervisar el estado de la aplicación.

### <a name="related-resources"></a>Recursos relacionados
Los recursos de esta tabla se usan en la máquina virtual y deben ya existir o crearse al tiempo que la máquina virtual.

| Recurso | Obligatorio | DESCRIPCIÓN |
| --- | --- | --- |
| [Grupos de recursos](../../azure-resource-manager/resource-group-overview.md) |Sí |La máquina virtual debe encontrarse en un grupo de recursos. |
| [Cuenta de almacenamiento](../../storage/common/storage-create-storage-account.md) |Sí |La máquina virtual necesita la cuenta de almacenamiento para almacenar sus discos duros virtuales. |
| [Red virtual](../../virtual-network/virtual-networks-overview.md) |Sí |La máquina virtual debe ser miembro de una red virtual. |
| [Dirección IP pública](../../virtual-network/virtual-network-ip-addresses-overview-arm.md) |Sin  |La máquina virtual puede tener una dirección IP pública asignada para acceder remotamente a ella. |
| [Interfaz de red](../../virtual-network/virtual-network-network-interface.md) |Sí |La máquina virtual necesita la interfaz de red para comunicarse en la red. |
| [Discos de datos](attach-managed-disk-portal.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) |Sin  |La máquina virtual puede incluir discos de datos para ampliar las funcionalidades de almacenamiento. |

## <a name="how-do-i-create-my-first-vm"></a>¿Cómo se crea la primera máquina virtual?
Hay varias opciones para crear la máquina virtual. La decisión que tome depende del entorno en que se encuentre. 

En esta tabla se proporciona información para comenzar a crear la máquina virtual.

| Método | Artículo |
| --- | --- |
| Azure Portal |[Creación de una máquina virtual que ejecuta Windows en el portal](../virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) |
| Plantillas |[Creación de una máquina virtual Windows con una plantilla del Administrador de recursos](ps-template.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) |
| Azure PowerShell |[Creación de una máquina virtual Windows con PowerShell](../virtual-machines-windows-ps-create.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) |
| SDK de cliente |[Implementación de recursos de Azure mediante C#](csharp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) |
| API de REST |[Create or update a VM](https://docs.microsoft.com/rest/api/compute/virtualmachines/virtualmachines-create-or-update) (Creación o actualización de una máquina virtual) |
| Azure CLI |[Creación de una máquina virtual con la CLI de Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/scripts/virtual-machines-windows-cli-sample-create-vm) |

Aunque se espera que nunca suceda, en ocasiones algo sale mal. Si se ve en esta situación, consulte la información en [Solución de problemas de implementación de Resource Manager con la creación de una máquina virtual Windows en Azure](troubleshoot-deployment-new-vm.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="how-do-i-manage-the-vm-that-i-created"></a>¿Cómo administro la máquina virtual que he creado?
Las máquinas virtuales pueden administrarse mediante un portal basado en el explorador, herramientas de línea de comandos con compatibilidad para scripts o directamente a través de API. Algunas tareas de administración habituales que puede realizar es obtener información sobre una máquina virtual, iniciar sesión en una máquina virtual, administrar la disponibilidad y realizar copias de seguridad.

### <a name="get-information-about-a-vm"></a>Obtención información acerca de una máquina virtual
En esta tabla se muestran algunas maneras en que puede obtener información sobre una máquina virtual.

| Método | DESCRIPCIÓN |
| --- | --- |
| Azure Portal |En el menú central, haga clic en **Máquinas virtuales** y seleccione la máquina virtual en la lista. En la hoja de la máquina virtual, puede ver información general, establecer valores y supervisar métricas. |
| Azure PowerShell |Para información sobre cómo usar PowerShell para administrar máquinas virtuales, consulte [Creación y administración de máquinas virtuales Windows con el módulo de Azure PowerShell](tutorial-manage-vm.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). |
| API DE REST |Use la operación para [obtener información sobre la máquina virtual](https://docs.microsoft.com/rest/api/compute/virtualmachines/virtualmachines-get) para conseguir información acerca de una máquina virtual. |
| SDK de cliente |Para información sobre cómo usar C# para administrar máquinas virtuales, consulte [Administración de Azure Virtual Machines con Azure Resource Manager y C#](csharp-manage.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). |
| Azure CLI |Para información acerca del uso de la CLI de Azure para administrar las máquinas virtuales, consulte la [referencia de la CLI de Azure](https://docs.microsoft.com/en-us/cli/azure/vm). |

### <a name="log-on-to-the-vm"></a>Inicio de sesión en la máquina virtual
Use el botón Conectar en Azure Portal para [iniciar una sesión de Escritorio remoto (RDP)](connect-logon.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). En ocasiones, algo sale mal cuando se intenta usar una conexión remota. Si se ve en esta situación, consulte la información de ayuda en [Solución de problemas de conexiones del Escritorio remoto a una máquina virtual de Azure con Windows](troubleshoot-rdp-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

### <a name="manage-availability"></a>Administración de la disponibilidad
Es importante que comprenda cómo [garantizar una alta disponibilidad](manage-availability.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) para la aplicación. Esta configuración supone crear varias máquinas virtuales para asegurarse de que haya al menos una ejecutándose.

Para que su implementación pueda optar a nuestro SLA de máquina virtual 99,95, debe implementar dos o más máquinas virtuales que ejecuten la carga de trabajo dentro de un [conjunto de disponibilidad](tutorial-availability-sets.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Esta configuración garantiza que las máquinas virtuales estén distribuidas entre varios dominios de error e implementadas en hosts con diferentes períodos de mantenimiento. En el [SLA de Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/) completo se explica la disponibilidad garantizada de Azure como un conjunto.

### <a name="back-up-the-vm"></a>Copia de seguridad de la máquina virtual
Un [almacén de Recovery Services](../../backup/backup-introduction-to-azure-backup.md) se usa para proteger datos y recursos en los servicios Azure Backup y Azure Site Recovery. Puede usar un almacén de Recovery Services para [implementar y administrar copias de seguridad para máquinas virtuales implementadas con Resource Manager mediante PowerShell](../../backup/backup-azure-vms-automation.md). 

## <a name="next-steps"></a>Pasos siguientes
* Si tiene intención de trabajar con máquinas virtuales Linux, consulte [Azure y Linux](../linux/overview.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
* Conozca las directrices para configurar la infraestructura en el [Tutorial de la infraestructura de Azure de ejemplo](infrastructure-example.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
