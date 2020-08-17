---
title: Información general sobre cargas de trabajo de Red Hat en Azure | Microsoft Docs
description: Más información sobre las ofertas de productos de Red Hat disponibles en Azure
services: virtual-machines-linux
author: asinn826
manager: borisb2015
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.topic: overview
ms.date: 02/10/2020
ms.author: alsin
ms.openlocfilehash: 7394cb50010bddddf8f8eff4b4f04eaf4d3231b6
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87052108"
---
# <a name="red-hat-workloads-on-azure"></a>Cargas de trabajo de Red Hat en Azure

Las cargas de trabajo de Red Hat se admiten mediante diversas ofertas de Azure. Las imágenes de Red Hat Enterprise Linux (RHEL) son el núcleo de las cargas de trabajo de RHEL, como Red Hat Update Infrastructure (RHUI).

## <a name="red-hat-enterprise-linux-images"></a>Imágenes de Red Hat Enterprise Linux

Azure ofrece una amplia oferta de imágenes de RHEL en Azure. Estas imágenes están disponibles mediante dos modelos de licencia diferentes: Pago por uso (PAYG) y "Bring Your Own Subscription" (BYOS). Las nuevas imágenes de RHEL en Azure se publican cuando se crean nuevas versiones de RHEL y se actualizan a lo largo de sus ciclos de vida según sea necesario.

### <a name="pay-as-you-go-images"></a>Imágenes de pago por uso

Azure ofrece una variedad de imágenes de pago por uso de RHEL. Estas imágenes están correctamente autorizadas para RHEL y se adjuntan a un origen de actualizaciones (Red Hat Update Infrastructure). Estas imágenes cobran un precio prémium por el derecho y las actualizaciones de RHEL. Las variantes de imagen de pago por uso de RHEL incluyen:

* RHEL estándar.
* RHEL for SAP.
* RHEL for SAP con alta disponibilidad y servicios de actualización.

Puede que desee usar las imágenes de pago por uso si no desea preocuparse por el pago por separado del número de suscripciones adecuado.

### <a name="red-hat-gold-images"></a>Imágenes de Red Hat Gold

Azure también ofrece imágenes de Red Hat Gold (`rhel-byos`). Estas imágenes pueden ser útiles para los clientes que tienen suscripciones de Red Hat existentes y desean usarlas en Azure. Debe habilitar las suscripciones de Red Hat existentes para Red Hat Cloud Access para poder usarlas en Azure. El acceso a estas imágenes se concede automáticamente cuando las suscripciones de Red Hat están habilitadas para el acceso a la nube y cumplen los requisitos de idoneidad. El uso de estas imágenes permite a un cliente evitar la facturación doble en la que se podría incurrir al usar las imágenes de Pago por uso.
* Obtenga información sobre cómo [habilitar las suscripciones de Red Hat para el acceso a la nube con Azure](https://access.redhat.com/documentation/en-us/red_hat_subscription_management/1/html/red_hat_cloud_access_reference_guide/con-enable-subs).
* Obtenga información sobre cómo [buscar imágenes de Red Hat Gold en Azure Portal, la CLI de Azure o el cmdlet de PowerShell](./byos.md).

> [!NOTE]
> La facturación doble se produce cuando un usuario paga dos veces para las suscripciones RHEL. Esto suele suceder cuando un cliente utiliza el administrador de suscripciones para adjuntar un derecho en una máquina virtual de Pago por uso de RHEL. Por ejemplo, a un cliente que utiliza el administrador de suscripciones para adjuntar un derecho a los paquetes de SAP en una imagen de pago por uso de RHEL se le factura indirectamente el doble porque paga dos veces por RHEL. Paga una vez con la cuota de pago por uso Prémium y otra con su suscripción de SAP. Este escenario no se produce para los usuarios de imágenes BYOS.

### <a name="generation-2-images"></a>Imágenes de segunda generación

Las máquinas virtuales de segunda generación proporcionan algunas características más recientes, en comparación con las de primera generación. Para más información, consulte la [documentación de Compatibilidad para máquinas virtuales de generación 2 en Azure](../../linux/generation-2.md). La diferencia clave desde la perspectiva de una imagen de RHEL es que las máquinas virtuales de segunda generación usan una UEFI en lugar de la interfaz de firmware del BIOS. También utilizan una tabla de particiones GUID (GPT) en lugar de un registro de arranque maestro (MBR) en el momento del arranque. El uso de una GPT permite, entre otras cosas, tamaños de disco de sistema operativo superiores a 2 TB. Además, las [máquinas virtuales de la serie Mv2](../../mv2-series.md) solo se ejecutan en imágenes de segunda generación.

Las imágenes de segunda generación de RHEL están disponibles en Azure Marketplace. Busque "gen2" en la SKU de la imagen en la lista de todas las imágenes que aparece cuando se usa la CLI de Azure. Vaya a la pestaña **Avanzada** en el proceso de implementación de máquinas virtuales para implementar una máquina virtual de segunda generación.

## <a name="red-hat-update-infrastructure"></a>Infraestructura de actualización de Red Hat

Azure proporciona la infraestructura de actualización de Red Hat solo para las máquinas virtuales de RHEL de pago por uso. RHUI es, en realidad, un reflejo de las redes CDN de Red Hat, pero solo es accesible para las máquinas virtuales de RHEL de Pago por uso de Azure. Tendrá acceso a los paquetes correspondientes en función de la imagen RHEL que haya implementado. Por ejemplo, una imagen de RHEL para SAP tiene acceso a los paquetes SAP además de a los paquetes base de RHEL.

### <a name="rhui-update-behavior"></a>Comportamiento de actualización de RHUI

De forma predeterminada, las imágenes de RHEL conectadas a RHUI se actualizan a la versión secundaria más reciente de RHEL cuando se ejecuta `yum update`. Este comportamiento significa que una máquina virtual RHEL 7.4 puede actualizarse a RHEL 7.7 si se ejecuta una operación `yum update`. Este comportamiento es así por diseño para RHUI. Para mitigar este comportamiento de actualización, cambie de repositorios de RHEL normales a [repositorios de compatibilidad de actualización extendida](./redhat-rhui.md#rhel-eus-and-version-locking-rhel-vms).

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre las [imágenes de RHEL en Azure](./redhat-images.md).
* Más información sobre la [Infraestructura de Red Hat Update](./redhat-rhui.md).
* Más información sobre la oferta de [imagen de Red Hat Gold (`rhel-byos`) ](./byos.md).
