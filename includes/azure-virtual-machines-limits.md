---
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 11/09/2018
ms.author: cynthn
ms.openlocfilehash: 994a7726adec07f2f6533d460d05469a0f3c7bf3
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87103009"
---
| Resource | Límite |
| --- | --- |
| Máquinas virtuales por servicio en la nube<sup>1</sup> |50 |
| Punto de conexión de entrada por servicio en la nube<sup>2</sup> |150 |

<sup>1</sup>Las máquinas virtuales creadas mediante el modelo de implementación clásico, en lugar de mediante Azure Resource Manager, se almacenan automáticamente en un servicio en la nube. Puede agregar más máquinas virtuales a ese servicio en la nube para usarlas para el equilibrio de carga y la disponibilidad. 

<sup>2</sup>Los puntos de conexión de entrada permiten las comunicaciones con una máquina virtual desde fuera del servicio en la nube de la máquina virtual. Las máquinas virtuales en el mismo servicio en la nube o red virtual pueden comunicarse automáticamente entre sí.  
