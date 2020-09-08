---
title: Introducción de las máquinas virtuales Linux en Azure
description: Información general sobre las máquinas virtuales Linux en Azure.
author: cynthn
ms.service: virtual-machines-linux
ms.topic: overview
ms.workload: infrastructure
ms.date: 11/14/2019
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: b205665a0e5fc06fdc784efa91036f26da5d3cde
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88654351"
---
# <a name="linux-virtual-machines-in-azure"></a>Máquinas virtuales Linux en Azure

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
El [tamaño](../sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) de la máquina virtual que use depende de la carga de trabajo que vaya a ejecutar. El tamaño que elija determina factores tales como la capacidad de almacenamiento, la memoria y la capacidad de procesamiento. Azure ofrece una amplia variedad de tamaños para admitir muchos tipos de usos.

Azure cobra un [precio por hora](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) en función del tamaño y el sistema operativo de la máquina virtual. Para las fracciones de hora, solo cobra los minutos usados. El precio del almacenamiento se calcula y se cobra por separado.

## <a name="vm-limits"></a>Límites de máquina virtual
Su suscripción tiene [límites de cuota](../../azure-resource-manager/management/azure-subscription-service-limits.md) predeterminados que pueden afectar a la implementación de numerosas máquinas virtuales en su proyecto. El límite actual por suscripción es 20 máquinas virtuales por región. Para aumentar estos límites, [cree una incidencia de soporte técnico y solicite un aumento](../../azure-portal/supportability/resource-manager-core-quotas-request.md)

## <a name="managed-disks"></a>Managed Disks

El servicio Managed Disks controla la creación y administración de las cuentas de almacenamiento de Azure Storage en segundo plano y se asegura de que no tenga que preocuparse de los límites de escalabilidad de la cuenta de almacenamiento. Especifique el tamaño del disco y el nivel de rendimiento (Estándar o Premium), y Azure crea y administra el disco. Al agregar discos o escale y reduzca la máquina virtual no tendrá que preocuparse por el almacenamiento que se va a usar. Si está creando nuevas máquinas virtuales, [use la CLI de Azure](quick-create-cli.md) o Azure Portal para crear máquinas virtuales con sistema operativo administrado y discos de datos. Si tiene máquinas virtuales con discos no administrados, puede [convertir las máquinas virtuales para realizar copias de seguridad con Managed Disks](convert-unmanaged-to-managed-disks.md).

También puede administrar sus imágenes personalizadas en una cuenta de almacenamiento por región de Azure y utilizarlas para crear cientos de máquinas virtuales en la misma suscripción. Para más información acerca de Managed Disks, consulte [Introducción a Azure Managed Disks](../managed-disks-overview.md).

## <a name="distributions"></a>Distribuciones 
Microsoft Azure permite ejecutar varias de las distribuciones de Linux más populares proporcionadas y mantenidas por diversos asociados.  Entre otras cosas, puede encontrar distribuciones como Red Hat Enterprise, CentOS, SUSE Linux Enterprise, Debian, Ubuntu, CoreOS, RancherOS y FreeBSD en Azure Marketplace. Microsoft trabaja activamente con distintas comunidades de Linux para agregar aún más tipos a la lista de [distribuciones de Linux aprobadas para Azure](endorsed-distros.md).

Si su distribución de Linux favorita no está en la galería, puede usar su propia máquina virtual Linux [creando y actualizando un VHD de Linux en Azure](create-upload-generic.md).

En Microsoft trabajamos estrechamente con los asociados para garantizar que las imágenes disponibles están actualizadas y optimizadas para los entornos de tiempo de ejecución de Azure.  Para más información acerca de los asociados de Azure, consulte los vínculos siguientes:

* Linux en Azure: [distribuciones aprobadas](endorsed-distros.md)
* SUSE - [Azure Marketplace - SUSE Linux Enterprise Server](https://azuremarketplace.microsoft.com/marketplace/apps?search=suse%20sles&page=1)
* Red Hat - [Azure Marketplace - Red Hat Enterprise Linux 8.1](https://azuremarketplace.microsoft.com/marketplace/apps/RedHat.RedHatEnterpriseLinux81-ARM)
* Canonical - [Azure Marketplace - Ubuntu Server](https://azuremarketplace.microsoft.com/marketplace/apps/Canonical.UbuntuServer)
* Debian: [Azure Marketplace - Debian 8 "Jessie"](https://azuremarketplace.microsoft.com/marketplace/apps/credativ.debian)
* FreeBSD: [Azure Marketplace - FreeBSD 10.4](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.FreeBSD104)
* CoreOS - [Azure Marketplace - Container Linux by CoreOS](https://azuremarketplace.microsoft.com/marketplace/apps/CoreOS.CoreOS)
* RancherOS: [Azure Marketplace - RancherOS](https://azuremarketplace.microsoft.com/marketplace/apps/rancher.rancheros)
* Bitnami: [Bitnami Library for Azure](https://azure.bitnami.com/)
* Mesosphere: [Azure Marketplace - Mesosphere DC/OS on Azure](https://azure.microsoft.com/services/kubernetes-service/mesosphere/)
* Docker - [Azure Marketplace - Docker images](https://azuremarketplace.microsoft.com/marketplace/apps?search=docker&page=1&filters=virtual-machine-images)
* Jenkins: [Azure Marketplace - CloudBees Jenkins Platform](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/cloudbees.cloudbees-core-contact)


## <a name="cloud-init"></a>Cloud-Init 

Para instaurar una cultura de DevOps adecuada, la infraestructura al completa debe ser código.  Cuando toda la infraestructura reside en el código, puede volver a crearse con facilidad.  Azure funciona con todas las principales herramientas de automatización, como Ansible, Chef, SaltStack y Puppet.  Asimismo, tiene sus propias herramientas de automatización:

* [Plantillas de Azure](create-ssh-secured-vm-from-template.md)
* [VMAccess de Azure](../extensions/vmaccess.md)

Azure admite [cloud-init](https://cloud-init.io/) en la mayoría de las distribuciones de Linux que admiten este paquete.  Estamos trabajando activamente con nuestros asociados de distribuciones de Linux certificadas para disponer de imágenes con cloud-init habilitado en Azure Marketplace. Estas imágenes harán que las implementaciones y configuraciones de cloud-init funcionen perfectamente con las máquinas virtuales y los conjuntos de escalado de máquinas virtuales.

* [Uso de cloud-init en máquinas virtuales Linux](using-cloud-init.md)

## <a name="storage"></a>Storage
* [Introducción a Microsoft Azure Storage](../../storage/common/storage-introduction.md)
* [Incorporación de un disco a una máquina virtual con Linux mediante la CLI de Azure](add-disk.md)
* [Incorporación de un disco de datos a una máquina virtual con Linux en Azure Portal](attach-disk-portal.md)

## <a name="networking"></a>Redes
* [Información general sobre redes virtuales](../../virtual-network/virtual-networks-overview.md)
* [Direcciones IP en Azure](../../virtual-network/public-ip-addresses.md)
* [Apertura de puertos para una máquina virtual con Linux en Azure](nsg-quickstart.md)
* [Crear un nombre de dominio completo en el Portal de Azure](portal-create-fqdn.md)


## <a name="next-steps"></a>Pasos siguientes

Creación de la primera máquina virtual

- [Portal](quick-create-portal.md)
- [CLI de Azure](quick-create-cli.md)
- [PowerShell](quick-create-powershell.md)
