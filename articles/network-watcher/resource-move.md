---
title: Traslado de recursos de Azure Network Watcher | Microsoft Docs
description: Traslado de recursos de Azure Network Watcher entre regiones
services: network-watcher
documentationcenter: na
author: damendo
manager: ''
editor: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/27/2020
ms.author: damendo
ms.openlocfilehash: 97349071fee6a95623e5b5efdc0c9818cfe7b811
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87388339"
---
# <a name="moving-azure-network-watcher-resources-across-regions"></a>Traslado de recursos de Azure Network Watcher entre regiones

## <a name="moving-the-network-watcher-resource"></a>Traslado del recurso de Network Watcher
El recurso de Network Watcher representa el servicio back-end para Network Watcher y está totalmente administrado por Azure. Los clientes no tienen que administrarlo. La operación de traslado no se admite en este recurso.

## <a name="moving-child-resources-of-network-watcher"></a>Mover recursos secundarios de Network Watcher
Actualmente no se admite el traslado de recursos entre regiones para ningún recurso secundario del tipo de recurso `*networkWatcher*`.

## <a name="next-steps"></a>Pasos siguientes
* Lea la [introducción a Network Watcher](https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview).
* Consulte las [Preguntas frecuentes de Network Watcher](https://docs.microsoft.com/azure/network-watcher/frequently-asked-questions).
