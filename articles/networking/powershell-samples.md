---
title: Ejemplos de Azure PowerShell - Redes
description: Obtenga información sobre ejemplos de Azure PowerShell para las redes, incluido un ejemplo para crear una red virtual para aplicaciones de niveles múltiples.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
tags: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 05/24/2017
ms.author: gwallace
ms.openlocfilehash: aa3304cbabb183368f0e47415e64120854f43535
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87282082"
---
# <a name="azure-powershell-samples-for-networking"></a>Ejemplos de Azure PowerShell para redes

En la tabla siguiente se incluyen vínculos a scripts creados con Azure PowerShell.

| Script | Descripción |
|-|-|
|**Conectividad entre recursos de Azure**||
| [Creación de una red virtual para aplicaciones de niveles múltiples](./scripts/virtual-network-powershell-sample-multi-tier-application.md?toc=%2fazure%2fnetworking%2ftoc.json) | Creación de una red virtual con subredes de front-end y back-end. El tráfico a la subred de front-end está limitado a HTTP, mientras que el tráfico a la subred de back-end está limitado a SQL, al puerto 1433. |
| [Emparejamiento de dos redes virtuales](./scripts/virtual-network-powershell-sample-peer-two-virtual-networks.md?toc=%2fazure%2fnetworking%2ftoc.json) | Creación y conexión de dos redes virtuales en la misma región. |
| [Enrutamiento del tráfico por una aplicación virtual de red](./scripts/virtual-network-powershell-sample-route-traffic-through-nva.md?toc=%2fazure%2fnetworking%2ftoc.json) | Creación de una red virtual con subredes de front-end y back-end y una máquina virtual que pueda enrutar el tráfico entre dos subredes. |
| [Filtrado del tráfico de red entrante y saliente de máquina virtual](./scripts/virtual-network-powershell-filter-network-traffic.md?toc=%2fazure%2fnetworking%2ftoc.json) | Creación de una red virtual con subredes de front-end y back-end. El tráfico de red entrante a la subred de front-end está limitado a HTTP y HTTPS. No se permite el tráfico saliente a Internet desde la subred de back-end. |
|**Equilibrio de carga y dirección del tráfico**||
| [Equilibrio de carga del tráfico a las máquinas virtuales para conseguir alta disponibilidad](./scripts/load-balancer-windows-powershell-sample-nlb.md?toc=%2fazure%2fnetworking%2ftoc.json) | Crea varias máquinas virtuales de alta disponibilidad y una configuración de equilibrio de carga. |
| [Dirección del tráfico entre varias regiones para conseguir alta disponibilidad de las aplicaciones](./scripts/traffic-manager-powershell-websites-high-availability.md?toc=%2fazure%2fnetworking%2ftoc.json) |  Creación de dos planes de App Service, dos aplicaciones web, un perfil de Traffic Manager y dos puntos de conexión de Traffic Manager. |
| | |
