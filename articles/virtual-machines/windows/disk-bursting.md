---
title: Seguridad de disco administrado
description: Aprenda sobre la seguridad de disco para discos de Azure y para máquinas virtuales de Azure
author: albecker1
ms.author: albecker
ms.date: 09/22/2020
ms.topic: conceptual
ms.service: virtual-machines
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: ab54b68ab3f7984ee18a39cf3a81fa663af54dee
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90889097"
---
# <a name="disk-bursting"></a>Seguridad de disco
[!INCLUDE [managed-disks-bursting](../../../includes/managed-disks-bursting.md)]

## <a name="virtual-machine-level-bursting"></a>Seguridad en el nivel de máquina virtual
La compatibilidad con la seguridad de nivel de máquina virtual está habilitada en todas las regiones de la nube pública en estos tamaños admitidos: 
- [Serie Lsv2](../lsv2-series.md)

La expansión en el nivel de máquina virtual también está disponible en la región Centro-oeste de EE. UU. para los siguientes tamaños admitidos:
- [Serie Dsv3](../dv3-dsv3-series.md)
- [Serie Esv3](../ev3-esv3-series.md)

La seguridad está habilitada de forma predeterminada para las máquinas virtuales que la admiten.

## <a name="disk-level-bursting"></a>Seguridad en el nivel de disco
La seguridad también está disponible en las unidades [SSD Premium](disks-types.md#premium-ssd) para tamaños de disco P20 e inferiores en todas las regiones. La ampliación del disco está habilitada de forma predeterminada en las nuevas implementaciones de los tamaños del disco que la admiten. Los tamaños del disco existentes, si admiten la ampliación del disco, pueden habilitar la seguridad a través de cualquiera de los métodos siguientes: 
- **Reinicio de la máquina virtual** 
- **Desasociación y reasociación del disco**


[!INCLUDE [managed-disks-bursting](../../../includes/managed-disks-bursting-2.md)]
