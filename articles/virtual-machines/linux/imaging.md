---
title: Introducción a la creación de imágenes de Linux para Azure
description: Información general sobre cómo traer imágenes de máquinas virtuales Linux o crear imágenes para usarlas en Azure.
author: danielsollondon
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.topic: overview
ms.workload: infrastructure
ms.date: 06/22/2020
ms.author: danis
ms.reviewer: cynthn
ms.openlocfilehash: aa372d4e1b377ecdcbeb49b47f0f9a3a217ee7ad
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86502187"
---
# <a name="bringing-and-creating-linux-images-in-azure"></a>Creación y uso de imágenes de Linux en Azure

En esta introducción se tratan los conceptos básicos de la creación de imágenes y cómo compilar y usar correctamente imágenes de Linux en Azure. Antes de incluir una imagen personalizada en Azure, es preciso conocer qué tipos de imágenes hay disponibles, así como las distintas opciones.

En este artículo se describirán los requisitos y los distintos aspectos de las imágenes, y se explican los conceptos clave no solo para poder seguirlo, sino también para poder crear sus propias imágenes personalizadas de acuerdo a sus especificaciones.

## <a name="difference-between-managed-disks-and-images"></a>Diferencia entre discos administrados e imágenes


Azure permite llevar un disco duro virtual a la plataforma, con el fin de usarlo como [disco administrado](../windows/faq-for-disks.md#managed-disks) o como origen de una imagen. 

Los discos administrados de Azure son discos duros virtuales únicos. Puede tomar un disco duro virtual existente y crear un disco administrado a partir de él, o bien crear un disco administrado vacío desde cero. Puede crear máquinas virtuales a partir de discos administrados. Para ello, debe conectar el disco a la máquina virtual, pero solo puede usar un disco duro virtual con una máquina virtual. No puede modificar las propiedades del sistema operativo, Azure solo intentará encender la máquina virtual y empezar a usar ese disco. 

Las imágenes de Azure pueden estar compuestas de varios discos de sistema operativo y discos de datos. Cuando se usa una imagen administrada para crear una máquina virtual, la plataforma realiza una copia de la imagen y la usa para crear la máquina virtual, por lo que las imágenes administradas permiten reutilizar la misma imagen para varias máquinas virtuales. Azure también proporciona funcionalidades de administración avanzadas para imágenes, como la replicación global y el control de versiones a través de [Shared Image Gallery](shared-image-galleries.md). 



## <a name="generalized-and-specialized"></a>Generalizadas y especializadas

Azure ofrece dos tipos principales de imágenes, generalizadas y especializadas. Los términos generalizadas y especializadas son originalmente términos de Windows que han migrado a Azure. Estos tipos definen la forma en que la plataforma va a controlar la máquina virtual cuando la active. Ambos tienen ventajas, desventajas y requisitos previos. Antes de comenzar, debe saber qué tipo de imagen va a necesitar. A continuación, se resumen los escenarios y el tipo que debe elegir:

| Escenario      | Tipo de imagen  | Opciones de almacenamiento |
| ------------- |:-------------:| :-------------:| 
| Creación de una imagen que pueda configurarse para que la usen varias máquinas virtuales y que se pueda establecer el nombre de host, agregar un usuario administrador y realizar otras tareas durante el primer arranque. | Generalizada | Shared Image Gallery o imágenes administradas independientes |
| Creación de una imagen a partir de una instantánea de máquina virtual o de una copia de seguridad | Especializada |Shared Image Gallery o un disco administrado |
| Creación rápida de una imagen que no necesite ninguna configuración para crear varias máquinas virtuales |Especializada |Galería de imágenes compartidas |


### <a name="generalized-images"></a>Imágenes generalizadas

Una imagen generalizada es una imagen que requiere que se complete la configuración en el primer arranque. Por ejemplo, la primera vez que arranque establezca el nombre de host, el usuario administrador y otras configuraciones específicas de la máquina virtual. Esto resulta útil no solo si desea reutilizar la imagen varias veces, sino también si desea usar parámetros durante la creación. Si la imagen generalizada contiene el agente de Azure, este procesará los parámetros y transmitirá a la plataforma la señal de que se ha completado la configuración inicial. Este proceso se denomina **aprovisionamiento**. 

El aprovisionamiento requiere que se incluya un aprovisionador en la imagen. Hay dos aprovisionadores:
- [Agente Linux de Azure](../extensions/agent-linux.md)
- [cloud-init](./using-cloud-init.md)

Estos son los [requisitos previos](./create-upload-generic.md) para crear una imagen.


### <a name="specialized-images"></a>Imágenes especializadas
Se trata de imágenes que están completamente configuradas y no requieren parámetros especiales ni de máquina virtual. La plataforma solo activará la máquina virtual, el usuario tiene que controlar la unicidad dentro de la máquina virtual como, por ejemplo, a la hora de establecer un nombre de host con el fin de evitar conflictos de DNS en la misma red virtual. 

Para estas imágenes no se necesitan agentes de aprovisionamiento; sin embargo, puede que desee tener funcionalidades de administración de extensiones. Puede instalar el agente de Linux, pero deshabilite la opción de aprovisionamiento. Aunque no necesite un agente de aprovisionamiento, la imagen debe cumplir los [requisitos previos](./create-upload-generic.md) de las imágenes de Azure.


## <a name="image-storage-options"></a>Opciones de almacenamiento de Azure
Al usar la imagen de Linux, tiene dos opciones:

- Imágenes administradas para la creación de una máquina virtual simple en un entorno de desarrollo y de prueba.
- [Shared Image Gallery](shared-image-galleries.md) para crear y compartir imágenes a escala.


### <a name="managed-images"></a>Imágenes administradas

Las imágenes administradas se pueden usar para crear varias máquinas virtuales, pero tienen muchas limitaciones. Las imágenes administradas solo se pueden crear a partir de un origen generalizado (máquina virtual o disco duro virtual). Solo se pueden usar para crear máquinas virtuales en la misma región y no se pueden compartir entre suscripciones e inquilinos.

Las imágenes administradas se pueden usar para los entornos de desarrollo y prueba cuando se necesiten dos imágenes generalizadas simples para usarlas en una sola región y suscripción. 

### <a name="azure-shared-image-gallery-sig"></a>Azure Shared Image Gallery (SIG)

Las [galerías de imágenes compartidas](shared-image-galleries.md) se recomiendan para crear, administrar y compartir imágenes a escala. Las galerías de imágenes compartidas le ayudan a crear cierta estructura y organización en torno a las imágenes.  

- Compatibilidad con imágenes generalizadas y especializadas.
- Compatibilidad con imágenes de generación 1 y 2.
- Replicación global de imágenes.
- Control de versiones y agrupación de las imágenes para facilitar la administración.
- Imágenes de alta disponibilidad con almacenamiento con redundancia de zona (ZRS), en las regiones en que esté disponible Availability Zones. ZRS ofrece mejor resistencia a errores de zona.
- Imágenes compartidas entre suscripciones e, incluso, entre inquilinos de Active Directory (AD) mediante RBAC.
- Escalado de las implementaciones con réplicas de imagen en cada región.

En un nivel alto, se crea un SIG, que consta de:
- Definiciones de imágenes: son contenedores que albergan grupos de imágenes.
- Versiones de imágenes: son las imágenes reales.



## <a name="hyper-v-generation"></a>Generación de Hyper-V

Azure admite Hyper-V Generación 1 (Gen1) y Generación 2 (Gen2), Gen2 es la generación más reciente y ofrece más funciones que Gen1. Por ejemplo: más memoria, Intel Software Guard Extensions (Intel SGX) y memoria persistente virtualizada (vPMEM). Las VM de generación 2 que se ejecutan en el entorno local también tienen algunas características que aún no se admiten en Azure. Para más información, consulte la sección Características y funcionalidades. Para más información, consulte [este](../windows/generation-2.md) artículo. Cree imágenes de Gen2 si necesita la funcionalidad adicional.

Si necesita crear su propia imagen, asegúrese de que cumple los [requisitos previos de las imágenes](./create-upload-generic.md) y cárguela en Azure. Requisitos específicos de la distribución:


- [Distribuciones basadas en CentOS](create-upload-centos.md)
- [Debian Linux](debian-create-upload-vhd.md)
- [Flatcar Container Linux](flatcar-create-upload-vhd.md)
- [Oracle Linux](oracle-create-upload-vhd.md)
- [Red Hat Enterprise Linux](redhat-create-upload-vhd.md)
- [SLES y openSUSE](suse-create-upload-vhd.md)
- [Ubuntu](create-upload-ubuntu.md)


## <a name="next-steps"></a>Pasos siguientes

Aprenda a crear una instancia de [Shared Image Gallery](tutorial-custom-images.md).
