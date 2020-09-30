---
title: 'Guía de inicio rápido: habilitación de la administración automática de Azure para máquinas virtuales en Azure Portal'
description: Obtenga información sobre cómo habilitar rápidamente la administración automática para máquinas virtuales en una VM nueva o existente en Azure Portal.
author: ju-shim
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: quickstart
ms.date: 09/04/2020
ms.author: jushiman
ms.openlocfilehash: 43e0805a828a2ed32d23984b6ffef95a7e710953
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945403"
---
# <a name="quickstart-enable-azure-automanage-for-virtual-machines-in-the-azure-portal"></a>Inicio rápido: Habilitación de la administración automática para máquinas virtuales en Azure Portal

Introducción a la administración automática para máquinas virtuales mediante Azure Portal para habilitar la administración automática en una VM nueva o existente.


## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/) antes de empezar.

> [!NOTE]
> Las cuentas de evaluación gratuita no tienen acceso a las máquinas virtuales que se usan en este tutorial. Actualice a una suscripción de pago por uso.

> [!IMPORTANT]
> El siguiente permiso de RBAC es necesario para habilitar la administración automática: Rol **Propietario** o **Colaborador** junto con roles **Administrador de acceso de usuario**.


## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com/).


## <a name="enable-automanage-for-vms-on-an-existing-vm"></a>Habilitación de la administración automática para máquinas virtuales en una máquina virtual existente

1. En la barra de búsqueda, busque y seleccione **Administración automática: procedimientos recomendados para máquinas virtuales de Azure**.

2. Seleccione **Enable on existing VM** (Habilitar en una máquina virtual existente).

    :::image type="content" source="media\quick-create-virtual-machine-portal\zero-vm-list-view.png" alt-text="Habilitación en una máquina virtual existente.":::

3. En la hoja **Seleccionar máquinas**:
    1. Filtre la lista de máquinas virtuales por su **Suscripción** y **Grupo de recursos**.
    1. Active la casilla de cada máquina virtual que desee incorporar.
    1. Haga clic en el botón **Seleccionar**.

    :::image type="content" source="media\quick-create-virtual-machine-portal\existing-vm-select-machine.png" alt-text="Seleccione una máquina virtual existente en la lista de máquinas virtuales disponibles.":::

4. En **Perfil de configuración**, haga clic en **Browse and change profiles and preferences** (Examinar y cambiar perfiles y preferencias).

    :::image type="content" source="media\quick-create-virtual-machine-portal\existing-vm-quick-create.png" alt-text="Examine y cambie perfiles y preferencias.":::

5. En la hoja **Selección del perfil y las preferencias de configuración**:
    1. Seleccione un perfil a la izquierda: *Desarrollo/pruebas* para pruebas, *Prod* para producción.
    1. Haga clic en el botón **Seleccionar**.

    :::image type="content" source="media\quick-create-virtual-machine-portal\browse-production-profile.png" alt-text="Examine el perfil de configuración de producción.":::

6. Haga clic en el botón **Habilitar**.


## <a name="enable-automanage-for-vms-on-a-new-vm"></a>Habilitación de la administración automática para máquinas virtuales en una máquina virtual nueva

1. Siga los pasos de creación de la [Guía de inicio rápido: creación de una máquina virtual Windows en Azure Portal](..\virtual-machines\windows\quick-create-portal.md).

2. Una vez implementada la máquina virtual, se mostrará la página de estado de la implementación con los **Pasos siguientes** recomendados en la parte inferior.

    :::image type="content" source="media\quick-create-virtual-machine-portal\create-next-steps.png" alt-text="Sección Pasos siguientes ubicada en la parte inferior de la página de la implementación.":::

3. En **Pasos siguientes**, seleccione **Enable Automanage virtual machine best practices** (Habilitar los procedimientos recomendados de la administración automática de máquinas virtuales).

4. En la página **Administración automática: procedimientos recomendados para máquinas virtuales de Azure**, la información de **Máquinas** se rellenará automáticamente con la máquina virtual recién creada.

    :::image type="content" source="media\quick-create-virtual-machine-portal\create-new-enable-overview.png" alt-text="La máquina virtual recién creada se mostrará como la máquina seleccionada.":::

5. En **Perfil de configuración**, haga clic en **Browse and change profiles and preferences** (Examinar y cambiar perfiles y preferencias).

6. En la hoja **Selección del perfil y las preferencias de configuración**:
    1. Seleccione un perfil a la izquierda: *Desarrollo/pruebas* para pruebas, *Prod* para producción.
    1. Haga clic en el botón **Seleccionar**.

    :::image type="content" source="media\quick-create-virtual-machine-portal\browse-production-profile.png" alt-text="Examine el perfil de configuración de producción.":::

7. Haga clic en el botón **Habilitar**.


## <a name="disable-automanage-for-vms"></a>Deshabilitación de la administración automática para máquinas virtuales

Deje de usar rápidamente la administración automática de Azure para máquinas virtuales deshabilitando la administración automática.

:::image type="content" source="media\automanage-virtual-machines\disable-step-1.png" alt-text="Deshabilite la administración automática en una máquina virtual.":::

1. Vaya a la página **Administración automática: procedimientos recomendados para máquinas virtuales de Azure** en la que se muestran todas las máquinas virtuales administradas automáticamente.
1. Active la casilla situada junto a la máquina virtual que desea deshabilitar.
1. Haga clic en el botón **Disable automanagement** (Deshabilitar administración automática).
1. Lea detenidamente los mensajes del elemento emergente resultante antes de aceptar la opción **Deshabilitar**.


## <a name="clean-up-resources"></a>Limpieza de recursos

Si ha creado un nuevo grupo de recursos para probar la administración automática de Azure para máquinas virtuales y ya no lo necesita, puede eliminar el grupo de recursos. Al eliminar el grupo también se eliminan la máquina virtual y todos los recursos del grupo de recursos.

La administración automática de Azure crea grupos de recursos predeterminados en los que almacenar los recursos. Compruebe los grupos de recursos que tienen la convención de nomenclatura "DefaultResourceGroupRegionName" y "AzureBackupRGRegionName" para limpiar todos los recursos.

1. Seleccione el **Grupo de recursos**.
1. En la página del grupo de recursos, seleccione **Eliminar**.
1. Cuando se le solicite, escriba el nombre del grupo de recursos y, después, seleccione **Eliminar**.


## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, habilitó la administración automática de Azure para máquinas virtuales. 

Descubra cómo puede crear y aplicar preferencias personalizadas al habilitar la administración automática en su máquina virtual. 

> [!div class="nextstepaction"]
> [Administración automática de Azure para máquinas virtuales: perfil de configuración personalizado](virtual-machines-custom-preferences.md)
