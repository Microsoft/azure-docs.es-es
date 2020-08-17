---
title: 'Varias direcciones IP para máquinas virtuales de Azure: Portal | Microsoft Docs'
description: Obtenga información sobre cómo asignar varias direcciones IP a una máquina virtual con Azure Portal | Resource Manager.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/30/2016
ms.author: allensu
ms.openlocfilehash: 135504ad4956836614bfc0f2cb107e610de17c0c
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543493"
---
# <a name="assign-multiple-ip-addresses-to-virtual-machines-using-the-azure-portal"></a>Asignación de varias direcciones IP a máquinas virtuales con Azure Portal

> [!INCLUDE [virtual-network-multiple-ip-addresses-intro.md](../../includes/virtual-network-multiple-ip-addresses-intro.md)]
> 
> En este artículo se describe cómo crear una máquina virtual con el modelo de implementación de Azure Resource Manager mediante Azure Portal. No se pueden asignar varias direcciones IP a los recursos creados mediante el modelo de implementación clásica. Para información acerca de los modelos de implementación de Azure, lea el artículo [Understand deployment models](../resource-manager-deployment-model.md) (Descripción de los modelos de implementación).

[!INCLUDE [virtual-network-multiple-ip-addresses-scenario.md](../../includes/virtual-network-multiple-ip-addresses-scenario.md)]

## <a name="create-a-vm-with-multiple-ip-addresses"></a><a name = "create"></a>Creación de una máquina virtual con varias direcciones IP

Si desea crear una máquina virtual con varias direcciones IP, o una privada estática, debe crearla mediante PowerShell o la CLI de Azure. Para saber cómo, haga clic en las opciones de PowerShell o de la CLI en la parte superior de este artículo. Puede crear una máquina virtual con una única dirección IP privada dinámica y (opcionalmente) una única dirección IP pública. En el portal, siga los pasos de los artículos [Crear una máquina virtual Windows](../virtual-machines/windows/quick-create-portal.md) o [Crear una máquina virtual Linux](../virtual-machines/linux/quick-create-portal.md). Después de crear la máquina virtual, puede cambiar los tipos de direcciones IP de dinámicas a estáticas y agregar más mediante el portal siguiendo los pasos de la sección [Incorporación de direcciones IP a una VM](#add) de este artículo.

## <a name="add-ip-addresses-to-a-vm"></a><a name="add"></a>Incorporación de direcciones IP a una VM

Puede agregar direcciones IP públicas y privadas a una interfaz de red de Azure completando los pasos siguientes. En los ejemplos de las secciones siguientes se da por sentado que ya tiene una máquina virtual con las tres configuraciones de IP descritas en el [escenario](#scenario), pero no es un requisito indispensable.

### <a name="core-steps"></a><a name="coreadd"></a>Pasos principales

1. Vaya a Azure Portal en https://portal.azure.com e inicie sesión, si es necesario.
2. En el portal, haga clic en **Más servicios** > tipo *Máquinas virtuales* en el cuadro de filtro y luego haga clic en **Máquinas virtuales**.
3. En el panel **Máquinas virtuales**, haga clic en la máquina virtual a la que quiere agregar direcciones IP. Vaya a la pestaña **Redes**. Haga clic en **Interfaz de red** en la página. Como se muestra en la figura siguiente: 


    ![Adición de una dirección IP pública a una máquina virtual](./media/virtual-network-multiple-ip-addresses-portal/figure200319.png)
4. En la hoja **Interfaz de red**, seleccione **Configuraciones de IP**.

5. En el panel que aparece para la NIC que ha seleccionado, haga clic en **Configuraciones de IP**. Haga clic en **Agregar**, complete los pasos de una de las secciones siguientes según el tipo de dirección IP que desea agregar y, después, haga clic en **Aceptar**. 

### <a name="add-a-private-ip-address"></a>Incorporación de una dirección IP privada

Complete los pasos siguientes para agregar una nueva dirección IP privada:

1. Complete los pasos descritos en la sección [Pasos básicos](#coreadd) de este artículo y asegúrese de que se encuentre en la sección **Configuraciones de IP** de la interfaz de red de la máquina virtual.  Revise la subred que se muestra como predeterminada (por ejemplo, 10.0.0.0/24).
2. Haga clic en **Agregar**. En el panel **Agregar configuración de IP** que aparece, cree una configuración de IP denominada *IPConfig-4* con una nueva dirección IP privada *estática*. Para ello, seleccione un nuevo número para el octeto final y haga clic en **Aceptar**.  (Para la subred 10.0.0.0/24, una dirección IP de ejemplo sería *10.0.0.7*).

    > [!NOTE]
    > Al agregar una dirección IP estática, debe especificar una dirección válida no utilizada en la subred a la que la está conectada la NIC. Si la dirección que seleccionó no está disponible, el portal muestra una X para la dirección IP y debe seleccionar otra.

3. Al hacer clic en Aceptar, el panel se cierra y puede ver que aparece la nueva configuración de IP. Haga clic en **Aceptar** para cerrar el panel **Agregar configuración de IP**.
4. Puede hacer clic en **Agregar** agregue más configuraciones IP o cerrar todas las hojas abiertas para terminar de agregar direcciones IP.
5. Agregue la dirección IP privada al sistema operativo de la máquina virtual mediante los pasos de la sección [Incorporación de direcciones IP a un sistema operativo de la VM](#os-config) de este artículo.

### <a name="add-a-public-ip-address"></a>Incorporación de una dirección IP pública

Se agrega una dirección IP pública mediante la asociación de un recurso de dirección IP pública a una nueva configuración de IP o una configuración de IP existente.

> [!NOTE]
> Las direcciones IP públicas tienen un precio simbólico. Para más información sobre los precios de las direcciones IP, lea la página [Precios de las direcciones IP](https://azure.microsoft.com/pricing/details/ip-addresses) . Existe un límite para el número de direcciones IP públicas que pueden usarse dentro de una suscripción. Para más información sobre los límites, lea el artículo sobre los [límites de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).
> 

### <a name="create-a-public-ip-address-resource"></a><a name="create-public-ip"></a>Creación de un recurso de dirección IP pública

Una dirección IP pública es una configuración para un recurso de dirección IP pública. Si tiene un recurso de dirección IP pública que no está asociado actualmente a una configuración de IP que desea asociar a una configuración de IP, omita los pasos siguientes y complete los pasos de una de las secciones siguientes, según sea preciso. Si no tiene un recurso de dirección IP pública disponible, complete los pasos siguientes para crear uno:

1. Vaya a Azure Portal en https://portal.azure.com e inicie sesión, si es necesario.
3. En el portal, haga clic en **Crear un recurso** > **Redes** > **Dirección IP pública**.
4. En el panel **Crear dirección IP pública** que aparece, escriba un nombre en **Nombre**, seleccione un tipo en **Asignación de dirección IP**, una suscripción en **Suscripción**, un grupo de recursos en **Grupo de recursos** y una ubicación en **Ubicación**. Luego, haga clic en **Crear** tal y como se muestra en la siguiente imagen:

    ![Creación de un recurso de dirección IP pública](./media/virtual-network-multiple-ip-addresses-portal/figure5.png)

5. Complete los pasos de una de las secciones siguientes para asociar el recurso de dirección IP pública a una configuración de IP.

#### <a name="associate-the-public-ip-address-resource-to-a-new-ip-configuration"></a>Asociación del recurso de dirección IP pública a una nueva configuración de IP

1. Complete los pasos de la sección [Pasos principales](#coreadd) de este artículo.
2. Haga clic en **Agregar**. En el panel **Agregar configuración de IP** que aparece, cree una configuración de IP denominada *IPConfig 4*. Habilite la opción **Dirección IP pública** y seleccione un recurso de dirección IP pública existente que esté disponible en el panel **Elegir dirección IP pública** que aparece.

    Cuando haya seleccionado el recurso de dirección IP pública, haga clic en **Aceptar** y el panel se cierra. Si no tiene una dirección IP pública existente, puede crear una completando los pasos descritos en la sección [Creación de un recurso de dirección IP pública](#create-public-ip) de este artículo. 

3. Revise la nueva configuración de IP. Aunque no se asignara explícitamente una dirección IP privada, se asigna una automáticamente a la configuración de IP, dado que todas las configuraciones IP deben tener una dirección IP privada.
4. Puede hacer clic en **Agregar** agregue más configuraciones IP o cerrar todas las hojas abiertas para terminar de agregar direcciones IP.
5. Agregue al sistema operativo de la máquina virtual la dirección IP privada siguiendo las instrucciones de la sección [Incorporación de direcciones IP a un sistema operativo de la VM](#os-config) de este artículo. No agregue la dirección IP pública al sistema operativo.vo.

#### <a name="associate-the-public-ip-address-resource-to-an-existing-ip-configuration"></a>Asociación del recurso de dirección IP pública a una configuración de IP existente

1. Complete los pasos de la sección [Pasos principales](#coreadd) de este artículo.
2. Haga clic en la configuración de IP a la que desee asociar el recurso de dirección IP pública.
3. En el panel de configuración de IP que aparece, haga clic en **Dirección IP**.
4. Haga clic en el panel **Elegir dirección IP pública** que aparece y seleccione una dirección IP pública.
5. Haga clic en **Guardar**, el panel se cierra. Si no tiene una dirección IP pública existente, puede crear una completando los pasos descritos en la sección [Creación de un recurso de dirección IP pública](#create-public-ip) de este artículo.
3. Revise la nueva configuración de IP.
4. Puede hacer clic en **Agregar** agregue más configuraciones IP o cerrar todas las hojas abiertas para terminar de agregar direcciones IP. No agregue la dirección IP pública al sistema operativo.vo.


[!INCLUDE [virtual-network-multiple-ip-addresses-os-config.md](../../includes/virtual-network-multiple-ip-addresses-os-config.md)]
