---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 08/04/2020
ms.author: alkohli
ms.openlocfilehash: 8d6c3125d0109ae9005414add27f9aa08e932e49
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89082547"
---
Conéctese a la máquina virtual Windows mediante el Protocolo de escritorio remoto (RDP) a través de la dirección IP que pasó durante la creación de la máquina virtual.

1. En el cliente, inicie RDP. Vaya a **Inicio** y escriba **mstcs**.
1. Escriba la dirección IP de la máquina virtual y las credenciales de acceso que usó en el archivo de parámetros de plantilla de la máquina virtual.

    ![Conexión a través de RDP a una máquina virtual Windows](media/azure-stack-edge-gateway-connect-vm-windows/connect-vm-rdp-1.png)
1. Es posible que tenga que aprobar la conexión a una máquina que no sea de confianza. 
1. Ahora tiene iniciada la sesión en la máquina virtual que se ejecuta en el dispositivo. 