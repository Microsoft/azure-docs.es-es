---
title: Máquinas virtuales de generación 2 (versión preliminar) en Azure | Microsoft Docs
description: Información general de máquinas virtuales de Azure de generación 2
services: virtual-machines-linux
documentationcenter: ''
author: laurenhughes
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 05/13/2019
ms.author: lahugh
ms.openlocfilehash: 777486a64464c29e9014fdc3dbf94957b2e93aa5
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65596333"
---
# <a name="generation-2-vms-preview-on-azure"></a>Máquinas virtuales de generación 2 (versión preliminar) en Azure

> [!IMPORTANT]
> Máquinas virtuales de generación 2 están actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Compatibilidad para las máquinas virtuales de generación 2 (VM) ahora está disponible en versión preliminar pública en Azure. No se puede cambiar la generación de una máquina virtual después de que haya creado. Por lo tanto, se recomienda que revise las consideraciones [aquí](https://docs.microsoft.com/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v) , así como la información de esta página antes de elegir una generación.

Características clave de soporte técnico de las máquinas virtuales de generación 2, como: aumento de memoria, Intel® Software Guard Extensions (SGX) y virtual memoria persistente (vPMEM), que no se admiten en máquinas virtuales de generación 1. Máquinas virtuales de generación 2 tienen algunas características que aún no se admiten en Azure. Para obtener más información, consulte el [características y capacidades](#features-and-capabilities) sección. 

Máquinas virtuales de generación 2 usan la nueva vs arquitectura de arranque basados en UEFI la arquitectura basados en BIOS utilizada por máquinas virtuales de generación 1. En comparación con las máquinas virtuales de generación 1, las máquinas virtuales de generación 2 pueden haber mejorado tiempos de arranque e instalación. Para obtener una visión general de las máquinas virtuales de generación 2 y algunas de las diferencias claves entre la generación 1 y generación 2, consulte [debo crear una máquina virtual de generación 1 o 2 en Hyper-V?](https://docs.microsoft.com/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v).

## <a name="generation-2-vm-sizes"></a>Tamaños de máquina virtual de generación 2

Máquinas virtuales de generación 1 son compatibles con todos los tamaños de máquina virtual en Azure. Azure ofrece ahora compatibilidad de generación 2 con la siguiente serie de máquina virtual seleccionada en versión preliminar pública:

* [Dsv2](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-general#dsv2-series) y [serie Dsv3](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-general#dsv3-series-1)
* [Serie Esv3](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-memory#esv3-series)
* [Serie Fsv2](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-compute#fsv2-series-1)
* [Serie GS](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-memory#gs-series)
* [La serie ls](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-previous-gen#ls-series) y [Lsv2-series](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-storage#lsv2-series)
* [Mv2-series](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-memory#mv2-series)

## <a name="generation-2-vm-images-in-azure-marketplace"></a>Imágenes de máquina virtual de generación 2 en Azure Marketplace

Máquinas virtuales de generación 2 admiten las siguientes imágenes de Marketplace de Azure:

* Servidor de Windows de 2019 centro de datos
* Servidor de Windows 2016 Datacenter
* Servidor Windows 2012 R2 Datacenter
* Servidor Windows 2012 Datacenter

## <a name="on-premises-vs-azure-generation-2-vms"></a>En el entorno local frente a máquinas virtuales de Azure de generación 2

Azure no admite actualmente algunas de las características que on-premises Hyper-V admite para las máquinas virtuales de generación 2.

| Característica de generación 2                | En el entorno local Hyper-V | Azure |
|-------------------------------------|---------------------|-------|
| Arranque seguro                         | :heavy_check_mark:  | :x:   |
| Máquina virtual blindada                         | :heavy_check_mark:  | :x:   |
| vTPM                                | :heavy_check_mark:  | :x:   |
| Seguridad basada en virtualización (VBS) | :heavy_check_mark:  | :x:   |
| Formato VHDX                         | :heavy_check_mark:  | :x:   |

## <a name="features-and-capabilities"></a>Características y funcionalidades

### <a name="generation-1-vs-generation-2-features"></a>Características de generación 2 de vs de generación 1

| Característica | Generación 1 | Generación 2 |
|---------|--------------|--------------|
| Arranque             | PCAT                      | UEFI                               |
| Controladores de disco | IDE                       | SCSI                               |
| Tamaños de máquina virtual         | Disponible en todos los tamaños de máquina virtual | Premium storage admitida solo para máquinas virtuales |

### <a name="generation-1-vs-generation-2-capabilities"></a>Capacidades de generación 2 de vs de generación 1

| Funcionalidad | Generación 1 | Generación 2 |
|------------|--------------|--------------|
| > 2 TB de disco del sistema operativo                    | :x:                        | :heavy_check_mark: |
| SO de disco/imágenes/Swap personalizado         | :heavy_check_mark:         | :heavy_check_mark: |
| Compatibilidad con juego de escalado de máquinas virtuales | :heavy_check_mark:         | :heavy_check_mark: |
| Copia de seguridad/ASR                        | :heavy_check_mark:         | :x:                |
| Galería de imágenes compartidas              | :heavy_check_mark:         | :x:                |
| Azure Disk Encryption             | :heavy_check_mark:         | :x:                |

## <a name="creating-a-generation-2-vm"></a>Creación de una generación 2 VM

### <a name="marketplace-image"></a>Imagen de Marketplace

Máquinas virtuales de generación 2 pueden crearse desde una imagen de marketplace (que admite el arranque UEFI) a través del portal de Azure o la CLI de Azure.

El `windowsserver-gen2preview` oferta contiene solo las imágenes de generación 2 de Windows. Esto evita la confusión con respecto a las imágenes de generación 2 de vs de generación 1. Para crear generación 2 máquinas virtuales, seleccione **imágenes** de esta oferta y siga el proceso de creación de máquina virtual estándar.

Actualmente, la siguiente generación de Windows 2 imágenes están publicadas en Azure Marketplace:

* 2019-datacenter-gen2
* 2016-datacenter-gen2
* 2012-r2-datacenter-gen2
* 2012-datacenter-gen2

Consulte la sección de capacidades para obtener una lista de imágenes de marketplace compatibles como seguiremos agregando imágenes adicionales que admiten la generación 2.

### <a name="managed-image-or-managed-disk"></a>Imagen administrada o un disco administrado

Generación 2 máquinas virtuales se pueden crear desde una imagen administrada o un disco administrado de la misma manera que crearía una generación 1 máquina virtual.

### <a name="virtual-machine-scale-sets"></a>Conjuntos de escalas de máquina virtual

Generación que 2 máquinas virtuales también pueden crearse mediante conjuntos de escalado de máquinas virtuales. Puede crear la generación 2 máquinas virtuales mediante conjuntos de escalado de máquina virtual de Azure mediante la CLI de Azure.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

* **¿Las máquinas virtuales de generación 2 admiten Accelerated Networking?**  
    Sí, la compatibilidad de las máquinas virtuales de generación 2 [Accelerated Networking](../../virtual-network/create-vm-accelerated-networking-cli.md).

* **¿.vhdx es compatible en la generación 2?**  
    No, .vhd solo se admite en máquinas virtuales de generación 2.

* **¿Las máquinas virtuales de generación 2 admiten unidades Ultra de estado sólido (SSD)?**  
    Sí, las máquinas virtuales de generación 2 admiten Ultra SSD.

* **¿Puedo migrar desde la generación 1 a las máquinas virtuales de generación 2?**  
    No, no se puede cambiar la generación de una máquina virtual después de que haya creado. Si tiene que cambiar entre varias generaciones de máquina virtual, deberá crear una nueva máquina virtual de una generación.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre [las máquinas virtuales de generación 2 en Hyper-V](https://docs.microsoft.com/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v).