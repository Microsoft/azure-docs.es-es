---
title: Residencia de datos en Azure Network Watcher | Microsoft Docs
description: Uso de la residencia de datos en el servicio Network Watcher
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
ms.date: 07/20/2020
ms.author: damendo
ms.openlocfilehash: dff5519c1b36a0a7738cb2529c2f4460ecf61e48
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88117842"
---
# <a name="data-residency-for-azure-network-watcher"></a>Residencia de datos en Azure Network Watcher
Azure Network Watcher no almacena los datos de los clientes, excepto para el servicio Connection Monitor (versión preliminar).


## <a name="connection-monitor-preview-data-residency"></a>Residencia de datos de Connection Monitor (versión preliminar)
El servicio *Connection Monitor (versión preliminar)* almacena datos de clientes. Network Watcher almacena automáticamente estos datos en una sola región. Por tanto, *Connection Monitor (versión preliminar)* satisface automáticamente los requisitos de residencia de datos en la región, incluidos los que se especifican en el [Centro de confianza](https://azuredatacentermap.azurewebsites.net/).

## <a name="singapore-data-residency"></a>Residencia de datos en Singapur

En Azure, la característica que permite almacenar los datos de clientes en una sola región solo está disponible actualmente en la región de Sudeste Asiático (Singapur) de la geoárea Asia Pacífico. En todas las demás regiones, los datos del cliente se almacenan en la geoárea. Para más información, consulte el [Centro de confianza](https://azuredatacentermap.azurewebsites.net/).

> [!Note]
> **Antes de la replicación** los clientes tienen la opción de almacenar las direcciones IP del usuario final con su instancia de Network Watcher para que Network Watcher puedan supervisar los cambios de disponibilidad, latencia y topología de red relacionados con las direcciones IP del usuario final. Estas direcciones IP de usuario final se pueden clasificar como datos de clientes. A partir del 15 de julio de 2020, Network Watcher almacena estos datos en una sola región. (Los datos de clientes ya no se replican en HK). Estos datos ya no se replican en HK. Estos datos están y fueron cifrados en reposo.
> 
> Si estos datos de clientes se pusieran a disposición de un tercero, el tercero podría conocer la dirección IP, pero no se le concedería acceso a la máquina o al dispositivo asociados con la dirección IP. Network Watcher considera que ningún tercero ha tenido acceso a estos datos de clientes en HK.

## <a name="next-steps"></a>Pasos siguientes

* Lea la información general de [Network Watcher](https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview).
