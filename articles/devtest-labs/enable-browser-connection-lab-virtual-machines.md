---
title: Habilitación de una conexión de explorador en máquinas virtuales de Azure DevTest Labs
description: Ahora, DevTest Labs se integra con Azure Bastion, lo que le permite, como propietario del laboratorio, habilitar el acceso a todas las máquinas virtuales de laboratorio a través de un explorador.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 28a3c11f3df578265f9746a173fcb3029a132b26
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88870500"
---
# <a name="enable-browser-connection-on-azure-devtest-labs-virtual-machines"></a>Habilitación de una conexión de explorador en máquinas virtuales de Azure DevTest Labs 
DevTest Labs se integra con [Azure Bastion](../bastion/index.yml), lo que le permite conectarse a sus máquinas virtuales a través de un explorador. En primer lugar, hay que habilitar una conexión del explorador en las máquinas virtuales de laboratorio en cuestión.

Como propietario de un laboratorio, puede habilitar el acceso a todas las máquinas virtuales de laboratorio a través de un explorador. No es necesario un cliente, agente o componente de software adicional. Azure Bastion proporciona una conexión RDP/SSH segura e ininterrumpida a las máquinas virtuales directamente en Azure Portal a través de TLS. Cuando se conecta a través de Azure Bastion, las máquinas virtuales no necesitan una dirección IP pública. Para más información, vea [¿Qué es Azure Bastion?](../bastion/bastion-overview.md)


En este artículo se explica cómo habilitar una conexión del explorador en máquinas virtuales de laboratorio.

## <a name="prerequisites"></a>Prerrequisitos 
- Implemente un host de Bastion en la red virtual del laboratorio existente **(O)** conecte el laboratorio a una red virtual configurada con Bastion.
Para aprender cómo implementar un host de Bastion en una red virtual, consulte [Creación de un host de Azure Bastion](../bastion/bastion-create-host-portal.md). Al crear el host de Bastion, seleccione la red virtual del laboratorio. 
- El usuario del laboratorio debe tener el rol **Lector** en el host de Bastion y en la red virtual que lo tenga configurado. 

## <a name="create-a-second-sub-net-in-the-bastion-virtual-network"></a>Creación de una segunda subred en la red virtual de Bastion
En primer lugar, tiene que crear una segunda subred en la red virtual de Bastion porque AzureBastionSubnet no permite que se creen en ella recursos que no sean Bastion. Cree otra subred en la red virtual de Bastion tal como se muestra en la siguiente imagen:

![Segunda subred en la red virtual de Azure Bastion](./media/connect-virtual-machine-through-browser/second-subnet.png)

## <a name="enable-vm-creation-in-the-subnet"></a>Habilitación de la creación de máquinas virtuales en la subred
Ahora, habilite la creación de máquinas virtuales en esta subred siguiendo estos pasos: 

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione **Todos los servicios** en el menú de navegación izquierdo. 
1. Seleccione **DevTest Labs** en la lista. 
1. En la lista de laboratorios, seleccione el *suyo*. 

    > [!NOTE]
    > Azure Bastion ya está disponible con carácter general en las regiones siguientes: Oeste de EE. UU., Este de EE. UU., Oeste de Europa, Centro-sur de EE. UU., Este de Australia y Japón Oriental. Por lo tanto, cree un laboratorio en una de estas regiones si el suyo no está en una de ellas. 
    
1. Seleccione **Configuración y directivas** en la sección **Configuración** en el menú de la izquierda. 
1. Seleccione **Redes virtuales**.
1. Seleccione **Agregar** en la barra de herramientas. 
1. Seleccione la **red virtual** que tenga implementado el host de Bastion. 
1. Seleccione la subred para las máquinas virtuales, no **AzureBastionSubnet**, que fue la que creó primero. Si no ve la subred en la lista de la parte inferior, cierre la página y vuelva a abrirla. 

    ![Habilitación de la creación de máquinas virtuales en la subred](./media/connect-virtual-machine-through-browser/enable-vm-creation-subnet.png)
1. Seleccione la opción **Usar en la creación de máquinas virtuales**. 
1. Seleccione **Guardar** en la barra de herramientas. 
1. Si tiene una red virtual de laboratorio anterior, quítela seleccionando * *...* y, después, **Quitar**. 

## <a name="enable-browser-connection"></a>Habilitar una conexión del explorador 

Una vez que tenga configurada una red virtual de Bastion en el laboratorio, como propietario del laboratorio puede habilitar una conexión del explorador en las máquinas virtuales del laboratorio.

Haga lo siguiente para habilitar una conexión del explorador en las máquinas virtuales de laboratorio:

1. En Azure Portal, vaya a *su laboratorio*.
1. Seleccione **Configuration and policies** (Directivas y configuración).
1. En **Configuración**, seleccione **Conexión del explorador**. Si no ve esta opción, cierre la página **Directivas de configuración** y vuelva a abrirla. 

    ![Habilitar una conexión del explorador](./media/enable-browser-connection-lab-virtual-machines/browser-connect.png)

## <a name="next-steps"></a>Pasos siguientes
Vea el siguiente artículo para saber cómo conectarse a máquinas virtuales con un explorador: [Conexión a las máquinas virtuales a través de un explorador](connect-virtual-machine-through-browser.md)
