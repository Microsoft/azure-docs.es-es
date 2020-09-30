---
title: Eliminación de replicaciones para la replicación entre regiones de Azure NetApp Files | Microsoft Docs
description: Describe cómo eliminar una conexión de replicación que ya no es necesaria entre los volúmenes de origen y de destino.
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
ms.date: 09/16/2020
ms.author: b-juche
ms.openlocfilehash: 0904ac36a9453e51dbb1efc50eee2b9bf3c669c7
ms.sourcegitcommit: 7374b41bb1469f2e3ef119ffaf735f03f5fad484
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90708446"
---
# <a name="delete-replications"></a>Eliminación de replicaciones

Puede finalizar la conexión de replicación entre los volúmenes de origen y de destino mediante la eliminación de la replicación de volúmenes. Puede realizar la operación de eliminación desde el volumen de origen o de destino. La operación de eliminación solo quita la autorización para la replicación; no quita el volumen de origen ni de destino. 

## <a name="steps"></a>Pasos

1. Para eliminar la replicación de volúmenes, seleccione **Replicación** desde el volumen de origen o de destino.  

2. Haga clic en **Eliminar**.    

3. Para confirmar la eliminación, escriba **Sí** y haga clic en **Eliminar**.   

    ![Eliminación de la replicación](../media/azure-netapp-files/cross-region-replication-delete-replication.png)

## <a name="next-steps"></a>Pasos siguientes  

* [Replicación entre regiones](cross-region-replication-introduction.md)
* [Requisitos y consideraciones del uso de la replicación entre regiones](cross-region-replication-requirements-considerations.md)
* [Visualización del estado de mantenimiento de la relación de la replicación](cross-region-replication-display-health-status.md)
* [Solución de problemas de replicación entre regiones](troubleshoot-cross-region-replication.md)

