---
title: Azure Standard Load Balancer y Virtual Machine Scale Sets
titleSuffix: Azure Standard Load Balancer and Virtual Machine Scale Sets
description: Con esta ruta de aprendizaje, empiece a usar Azure Standard Load Balancer y Virtual Machine Scale Sets.
services: load-balancer
documentationcenter: na
author: irenehua
ms.custom: seodec18
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/17/2020
ms.author: irenehua
ms.openlocfilehash: f5a8453f84854108facb4da4616a7d362e0fb38c
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86528305"
---
# <a name="azure-load-balancer-with-azure-virtual-machine-scale-sets"></a>Azure Load Balancer con el conjunto de escalado de máquinas virtuales de Azure

Al trabajar con conjuntos de escalado de máquinas virtuales y el equilibrador de carga, debe tener en cuenta las pautas siguientes:

## <a name="multiple-virtual-machine-scale-sets-cant-use-the-same-load-balancer"></a>Varios conjuntos de escalado de máquinas virtuales no pueden usar el mismo equilibrador de carga.
## <a name="port-forwarding-and-inbound-nat-rules"></a>Enrutamiento de puerto y reglas NAT de entrada:
  * Después de crear el conjunto de escalado, no se puede modificar el puerto de back-end para una regla de equilibrio de carga que se usa en un sondeo de estado del equilibrador de carga. Para cambiar el puerto, puede quitar el sondeo de estado mediante la actualización del conjunto de escalado de máquinas virtuales de Azure, actualizar el puerto y, a continuación, volver a configurar el sondeo de estado.
  * Al usar el conjunto de escalado de máquinas virtuales en el grupo de back-end del equilibrador de carga, las reglas NAT de entrada predeterminadas se crean automáticamente.
## <a name="inbound-nat-pool"></a>Grupo NAT de entrada:
  * Cada conjunto de escalado de máquinas virtuales debe tener al menos un grupo NAT de entrada. 
  * El grupo NAT de entrada es una colección de reglas NAT de entrada. Un grupo NAT de entrada no puede admitir varios conjuntos de escalado de máquinas virtuales.
  
## <a name="load-balancing-rules"></a>Reglas de equilibrio de carga:
  * Al usar el conjunto de escalado de máquinas virtuales en el grupo de back-end del equilibrador de carga, la regla de equilibrio de carga predeterminada se crea automáticamente.
## <a name="outbound-rules"></a>Reglas de salida:
  *  A fin de crear una regla de salida para un grupo de back-end al que ya se hace referencia mediante una regla de equilibrio de carga, primero debe marcar **"Crear reglas de salida implícitas"** como **No** en el portal cuando se crea la regla de equilibrio de carga de entrada.

  :::image type="content" source="./media/vm-scale-sets/load-balancer-and-vm-scale-sets.png" alt-text="Creación de una regla de equilibrio de carga" border="true":::

Los siguientes métodos se pueden usar para implementar un conjunto de escalado de máquinas virtuales con una instancia existente de Azure Load Balancer.

* [Configuración de un conjunto de escalado de máquinas virtuales con una instancia existente de Azure Load Balancer mediante Azure Portal](https://docs.microsoft.com/azure/load-balancer/configure-vm-scale-set-portal)
* [Configuración de un conjunto de escalado de máquinas virtuales con una instancia existente de Azure Load Balancer mediante Azure PowerShell](https://docs.microsoft.com/azure/load-balancer/configure-vm-scale-set-powershell).
* [Configuración de un conjunto de escalado de máquinas virtuales con una instancia existente de Azure Load Balancer mediante la CLI de Azure](https://docs.microsoft.com/azure/load-balancer/configure-vm-scale-set-cli).
