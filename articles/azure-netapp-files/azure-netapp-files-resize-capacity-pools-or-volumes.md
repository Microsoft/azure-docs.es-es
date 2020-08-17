---
title: Cambiar el tamaño de un grupo de capacidad o de un volumen de Azure NetApp Files  | Microsoft Docs
description: Aprenda a cambiar el tamaño de un grupo de capacidad o de un volumen. Al cambiar el tamaño del grupo de capacidad, cambia la capacidad adquirida de Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/14/2019
ms.author: b-juche
ms.openlocfilehash: 7d1c54d05430600e9fffa2659dbaaf6db0b086b5
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87512880"
---
# <a name="resize-a-capacity-pool-or-a-volume"></a>Cambiar el tamaño de un grupo de capacidad o de un volumen
Puede cambiar el tamaño de un grupo de capacidad o de un volumen según sea necesario. 

## <a name="resize-the-capacity-pool"></a>Cambiar el tamaño del grupo de capacidad 

Puede cambiar el tamaño del grupo de capacidad en incrementos o decrementos de 1 TiB. Aun así, el tamaño del grupo de capacidad no puede ser inferior a 4 TiB. Al cambiar el tamaño del grupo de capacidad, cambia la capacidad adquirida de Azure NetApp Files.

1. En la hoja de administración de cuenta de NetApp, haga clic en el grupo de capacidad cuyo tamaño desee cambiar. 
2. Haga clic con el botón derecho en el nombre del grupo de capacidad o haga clic en el icono "…" al final de la fila del grupo de capacidad para mostrar el menú contextual. 
3. Use las opciones del menú contextual para cambiar el tamaño del grupo de capacidad o eliminarlo.

## <a name="resize-a-volume"></a>Cambiar el tamaño de un volumen

Puede cambiar el tamaño de un volumen según sea necesario. El consumo de la capacidad de un volumen se descuenta de la capacidad aprovisionada de su grupo.

1. En la hoja de administración de cuenta de NetApp, haga clic en **Volúmenes**. 
2. Haga clic con el botón derecho en el nombre del volumen cuyo tamaño desee cambiar o haga clic en el icono "…" al final de la fila del volumen para mostrar el menú contextual.
3. Use las opciones del menú contextual para cambiar el tamaño del volumen o eliminarlo.

