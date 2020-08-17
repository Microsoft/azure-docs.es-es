---
title: Introducción a la vista de reglas de seguridad vigentes de Azure Network Watcher | Microsoft Docs
description: En esta página se proporciona información general sobre las funcionalidades de la vista de reglas de seguridad vigentes de Network Watcher
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/26/2017
ms.author: damendo
ms.openlocfilehash: f4c601184a060c3dfc4f033bcf983bf773f7167f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87022657"
---
# <a name="introduction-to-effective-security-rules-view-in-azure-network-watcher"></a>Introducción a la vista de reglas de seguridad vigentes de Azure Network Watcher

Los grupos de seguridad de red pueden asociarse a un nivel de subred o a un nivel de NIC. Cuando se asocia a un nivel de subred, se aplica a todas las instancias de la máquina virtual en la subred. La vista de reglas de seguridad vigentes devuelve todos los grupos de seguridad de red así como las reglas que están asociadas a un nivel de subred y a la NIC para una máquina virtual que proporciona una perspectiva sobre la configuración. Además, se devuelven las reglas de seguridad vigentes para cada una de las NIC en una máquina virtual. Con la vista de reglas de seguridad vigentes, se puede evaluar una VM en busca de vulnerabilidades de red, como puertos abiertos. También se puede validar si el grupo de seguridad de red está funcionando según lo previsto en función de una [comparación entre las reglas de seguridad aprobadas y configuradas](network-watcher-nsg-auditing-powershell.md).

Un caso de uso más extendido se encuentra en el cumplimiento de seguridad y auditoría. Puede definir un conjunto prescriptivo de reglas de seguridad como modelo para la gobernanza de la seguridad de su organización. Una auditoría periódica del cumplimiento puede implementarse de forma programática al comparar las reglas preceptivas con las reglas vigentes para cada una de las máquinas virtuales en la red.

En el portal, las reglas se muestran para cada interfaz de red y se agrupan por entrada y salida. Esto proporciona una vista sencilla de las reglas que se aplican a una máquina virtual. Se incluye un botón de descarga para descargar fácilmente todas las reglas de seguridad con independencia de la pestaña en un archivo CSV.

![vista de grupos de seguridad][1]

Se pueden seleccionar las reglas y se abre una nueva hoja para mostrar el grupo de seguridad de red y los prefijos de origen y de destino. En esta hoja puede navegar directamente al recurso del grupo de seguridad de red.

![obtención de detalles][2]

### <a name="next-steps"></a>Pasos siguientes

También puede usar la característica *Grupos de seguridad en vigor* a través de otros métodos que se enumeran a continuación:
* [REST API](https://docs.microsoft.com/rest/api/virtualnetwork/NetworkInterfaces/ListEffectiveNetworkSecurityGroups)
* [PowerShell](https://docs.microsoft.com/powershell/module/az.network/get-azeffectivenetworksecuritygroup?view=azps-4.4.0)
* [CLI de Azure](https://docs.microsoft.com/cli/azure/network/nic?view=azure-cli-latest#az-network-nic-list-effective-nsg)

Aprenda a auditar la configuración del grupo de seguridad de red consultando [Auditoría de la configuración del grupo de seguridad de red con PowerShell](network-watcher-nsg-auditing-powershell.md).

[1]: ./media/network-watcher-security-group-view-overview/securitygroupview.png
[2]: ./media/network-watcher-security-group-view-overview/figure1.png









