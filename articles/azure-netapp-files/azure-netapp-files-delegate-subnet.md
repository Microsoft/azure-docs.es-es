---
title: Delegación de una subred en Azure NetApp Files | Microsoft Docs
description: Aprenda a delegar una subred en Azure NetApp Files. Cuando se crea un volumen, especifique la subred delegada.
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
ms.date: 07/28/2020
ms.author: b-juche
ms.openlocfilehash: 9bb3e93b99ce8d5a61501d417a71e5e38753f5ff
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87513033"
---
# <a name="delegate-a-subnet-to-azure-netapp-files"></a>Delegación de una subred en Azure NetApp Files 

Debe delegar una subred en Azure NetApp Files.   Cuando se crea un volumen, debe especificar la subred delegada.

## <a name="considerations"></a>Consideraciones
* El asistente para crear una nueva subred se establece de manera predeterminada en una máscara de red /24, que ofrece 251 direcciones IP disponibles. Con una máscara de red /28, que ofrece 11 direcciones IP utilizables, es suficiente para el servicio.
* En cada red virtual de Azure (VNet), solo puede delegarse una subred a Azure NetApp Files.   
   Azure permite crear varias subredes delegadas en una red virtual.  Sin embargo, si usa más de una subred delegada, todos los intentos de crear un nuevo volumen darán error.  
   Solo puede tener una única subred delegada en una red virtual. Una cuenta de NetApp puede implementar volúmenes en varias redes virtuales, cada una de las cuales tiene su propia subred delegada.  
* No se puede designar un grupo de seguridad de red o punto de conexión de servicio en la subred delegada. Si lo hace, se producirá un error en la delegación de subred.
* Actualmente no se admite el acceso a un volumen desde una red virtual emparejada de forma global.
* No se admite la creación de [rutas personalizadas definidas por el usuario](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview#custom-routes) en subredes de máquina virtual con un prefijo de dirección (destino) dirigido a una subred delegada a Azure NetApp Files. Si las crea, se verá afectada la conectividad de la máquina virtual.

## <a name="steps"></a>Pasos 
1.  Vaya a la hoja **Redes virtuales** de Azure Portal y seleccione la red virtual que desea usar para Azure NetApp Files.    

1. Seleccione **Subredes** en la hoja de la red virtual y haga clic en el botón **+ Subred**. 

1. Cree una nueva subred que se usará para Azure NetApp Files completando los siguientes campos obligatorios en la página Agregar subred:
    * **Name**: Especifique el nombre de la subred.
    * **Intervalo de direcciones**: Especifique el intervalo de direcciones IP.
    * **Delegación de subred**: Seleccione **Microsoft.NetApp/volumes**. 

      ![Delegación de subred](../media/azure-netapp-files/azure-netapp-files-subnet-delegation.png)
    
También puede crear y delegar una subred al [crear un volumen de Azure NetApp Files](azure-netapp-files-create-volumes.md). 

## <a name="next-steps"></a>Pasos siguientes  
* [Creación de un volumen de Azure NetApp Files](azure-netapp-files-create-volumes.md)
* [Obtenga información sobre la integración de redes virtuales para los servicios de Azure](https://docs.microsoft.com/azure/virtual-network/virtual-network-for-azure-services)


