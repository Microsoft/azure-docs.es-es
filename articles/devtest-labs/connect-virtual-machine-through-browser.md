---
title: Conexión a las máquinas virtuales a través de un explorador - Azure | Microsoft Docs
description: Sepa cómo conectar sus máquinas virtuales a través de un explorador.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 352fc5fd3ff53a00d9f62966ecf21417ad898706
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288065"
---
# <a name="connect-to-your-virtual-machines-through-a-browser"></a>Conexión a las máquinas virtuales a través de un explorador 

DevTest Labs se integra con [Azure Bastion](../bastion/index.yml), lo que le permite conectarse a sus máquinas virtuales a través de un explorador. Para más información sobre cómo habilitar esta característica en DevTest Labs, vea [Habilitar una conexión del explorador en máquinas virtuales de laboratorio](enable-browser-connection-lab-virtual-machines.md).

Una vez habilitada la *conexión del explorador*, los usuarios de laboratorio pueden acceder a las máquinas virtuales a través de un explorador.  

## <a name="create-a-lab-virtual-machine"></a>Creación de una máquina virtual de laboratorio

En primer lugar, debe crear la máquina virtual de laboratorio en una red virtual donde Bastion esté configurado. Seleccione la segunda **subred** que creó, no la AzureBastionSubnet. Puede seleccionar una red virtual mientras crea la máquina virtual; para ello, vaya a la pestaña **Configuración avanzada**.

![Crear máquina virtual](./media/connect-virtual-machine-through-browser/create-virtual-machine.png)

## <a name="launch-virtual-machine-in-a-browser"></a>Inicio de una máquina virtual en un explorador

Una vez creada la máquina virtual, puede iniciarla en un explorador; para ello, haga clic en el botón *Browser connect* (Conexión del explorador) y escriba el nombre de usuario y la contraseña de la máquina.  

![Inicio en un explorador](./media/connect-virtual-machine-through-browser/browser-connect.png)

## <a name="next-steps"></a>Pasos siguientes

[Incorporación de una máquina virtual a un laboratorio de Azure DevTest Labs](devtest-lab-add-vm.md)
