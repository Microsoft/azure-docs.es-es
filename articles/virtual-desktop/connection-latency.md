---
title: 'Latencia de la conexión para usuarios de Windows Virtual Desktop: Azure'
description: Latencia de la conexión para usuarios de Windows Virtual Desktop.
author: Heidilohr
ms.topic: conceptual
ms.date: 10/30/2019
ms.author: helohr
manager: lizross
ms.openlocfilehash: 403cf584c79bc0a166054ae36c9d2ea50e4b9d9e
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88008736"
---
# <a name="determine-user-connection-latency-in-windows-virtual-desktop"></a>Determinación de la latencia de la conexión de los usuarios en Windows Virtual Desktop

Windows Virtual Desktop está disponible a nivel mundial. Los administradores pueden crear máquinas virtuales (VM) en cualquier región de Azure que deseen. La latencia de la conexión variará en función de la ubicación de los usuarios y las máquinas virtuales. Los servicios de Windows Virtual Desktop se implementarán continuamente en nuevas zonas geográficas para mejorar la latencia.

La [herramienta Estimador de experiencia de Windows Virtual Desktop](https://azure.microsoft.com/services/virtual-desktop/assessment/) puede ayudarle a determinar la mejor ubicación para optimizar la latencia de las máquinas virtuales. Se recomienda usar la herramienta cada dos o tres meses para asegurarse de que la ubicación óptima no ha cambiado a medida que Windows Virtual Desktop se implementa en nuevas áreas.

## <a name="azure-traffic-manager"></a>Administrador de tráfico de Azure

Windows Virtual Desktop emplea Azure Traffic Manager, que comprueba la ubicación del servidor DNS del usuario para encontrar la instancia más cercana de este servicio. Se recomienda a los administradores que revisen la ubicación del servidor DNS del usuario antes de elegir la ubicación de las máquinas virtuales.

## <a name="next-steps"></a>Pasos siguientes

- Para comprobar cuál es la mejor ubicación de cara a una latencia óptima, consulte la [herramienta Estimador de experiencia de Windows Virtual Desktop](https://azure.microsoft.com/services/virtual-desktop/assessment/).
- Para información sobre los planes de precios, consulte [Precios de Windows Virtual Desktop](https://azure.microsoft.com/pricing/details/virtual-desktop/).
- Para empezar a trabajar con la implementación de Windows Virtual Desktop, consulte [nuestro tutorial](./virtual-desktop-fall-2019/tenant-setup-azure-active-directory.md).