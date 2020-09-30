---
title: Creación de una preferencia personalizada en Azure Automanage para máquinas virtuales
description: Obtenga información sobre cómo ajustar el perfil de configuración en Azure Automanage para máquinas virtuales y establezca sus propias preferencias.
author: ju-shim
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: how-to
ms.date: 09/04/2020
ms.author: jushiman
ms.openlocfilehash: cf8e7340ce3554bf27364335457a5b7eadcdc049
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931956"
---
# <a name="create-a-custom-preference-in-azure-automanage-for-vms"></a>Creación de una preferencia personalizada en Azure Automanage para máquinas virtuales

Azure Automanage para procedimientos recomendados para máquinas virtuales tiene perfiles de configuración predeterminados que se pueden ajustar en caso necesario. En este artículo se explicará cómo puede establecer sus propias preferencias de perfil de configuración al habilitar la administración automática en una máquina virtual nueva o existente.

Actualmente admitimos las personalizaciones en [Azure Backup](..\backup\backup-azure-arm-vms-prepare.md#create-a-custom-policy) y [Microsoft Antimalware](../security/fundamentals/antimalware.md#default-and-custom-antimalware-configuration).


> [!NOTE]
> No puede cambiar el perfil o la preferencia de configuración de la máquina virtual mientras está habilitado Automanage. Tendrá que deshabilitar Automanage para esa máquina virtual y, a continuación, volver a habilitarlo con el perfil y las preferencias de configuración deseados.


## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/) antes de empezar.

> [!NOTE]
> Las cuentas de evaluación gratuita no tienen acceso a las máquinas virtuales que se usan en este tutorial. Actualice a una suscripción de pago por uso.

> [!IMPORTANT]
> El siguiente permiso de RBAC es necesario para habilitar Automanage: Rol **Propietario** o **Colaborador** junto con roles **Administrador de acceso de usuario**.


## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com/).


## <a name="enable-automanage-for-vms-on-an-existing-vm"></a>Habilitación de Automanage para máquinas virtuales en una máquina virtual existente

1. En la barra de búsqueda, busque y seleccione **Automanage – Azure virtual machine best practices** (Automanage: procedimientos recomendados para máquinas virtuales de Azure).

2. Seleccione **Enable on existing VM** (Habilitación en una máquina virtual existente).

3. En la hoja **Seleccionar máquinas**:
    1. Filtre la lista de máquinas virtuales por su **Suscripción** y **Grupo de recursos**.
    1. Active la casilla de cada máquina virtual que desee incorporar.
    1. Haga clic en el botón **Seleccionar**.

    :::image type="content" source="media\virtual-machine-custom-preferences\existing-vm-select-machine.png" alt-text="Seleccione una máquina virtual existente en la lista de máquinas virtuales disponibles.":::

4. En **Perfil de configuración**, haga clic en **Browse and change profiles and preferences** (Examinar y cambiar perfiles y preferencias).

    :::image type="content" source="media\virtual-machine-custom-preferences\existing-vm-quick-create.png" alt-text="Examine y cambie perfiles y preferencias.":::

5. En la hoja **Selección del perfil y las preferencias de configuración**, seleccione un perfil en el lado izquierdo: *Desarrollo/pruebas* para pruebas, *Prod* para producción.

    :::image type="content" source="media\virtual-machine-custom-preferences\browse-production-profile.png" alt-text="Examine el perfil de configuración de producción.":::

6. En el perfil elegido, en **Preferencias de configuración**, hay una lista desplegable donde puede realizar ajustes para determinados servicios.
    1. Haga clic en **Create new preferences** (Crear nuevas preferencias).
    1. En la hoja **Create a configuration preference** (Crear una preferencia de configuración), rellene la pestaña Conceptos básicos:
        1. Subscription
        1. Resource group
        1. Nombre de preferencia
        1. Region

    :::image type="content" source="media\virtual-machine-custom-preferences\create-preference.png" alt-text="Rellene las preferencias de configuración.":::

7. Vaya a la pestaña Preferencias y ajuste las preferencias de configuración que desee.
        
    > [!NOTE]
    > Solo se permitirán los ajustes que se sigan ajustando a los límites superior e inferior de nuestros procedimientos recomendados al cambiar las configuraciones de perfil.

8. Revise el perfil de configuración.
9. Haga clic en el botón **Crear**.

10. Haga clic en el botón **Habilitar**.


## <a name="disable-automanage-for-vms"></a>Deshabilitación de Automanage para máquinas virtuales

Deje de usar rápidamente Azure Automanage para máquinas virtuales deshabilitando la administración automática.

:::image type="content" source="media\virtual-machine-custom-preferences\disable-step-1.png" alt-text="Deshabilite Automanage en una máquina virtual.":::

1. Vaya a la página **Automanage – Azure virtual machine best practices** (Automanage: procedimientos recomendados para máquinas virtuales de Azure) en la que se muestran todas las máquinas virtuales administradas automáticamente.
1. Active la casilla situada junto a la máquina virtual que desea deshabilitar.
1. Haga clic en el botón **Disable automanagement** (Deshabilitar administración automática).
1. Lea detenidamente los mensajes del elemento emergente resultante antes de aceptar **Deshabilitar**.


## <a name="clean-up-resources"></a>Limpieza de recursos

Si ha creado un nuevo grupo de recursos para probar Azure Automanage para máquinas virtuales y ya no lo necesita, puede eliminar el grupo de recursos. Al eliminar el grupo también se eliminan la máquina virtual y todos los recursos del grupo de recursos.

Azure Automanage crea grupos de recursos predeterminados en los que almacenar los recursos. Compruebe los grupos de recursos que tienen la convención de nomenclatura "DefaultResourceGroupRegionName" y "AzureBackupRGRegionName" para limpiar todos los recursos.

1. Seleccione el **Grupo de recursos**.
1. En la página del grupo de recursos, seleccione **Eliminar**.
1. Cuando se le solicite, escriba el nombre del grupo de recursos y, después, seleccione **Eliminar**.


## <a name="next-steps"></a>Pasos siguientes 

Consulte las preguntas más frecuentes respondidas en nuestra sección de preguntas más frecuentes. 

> [!div class="nextstepaction"]
> [Preguntas más frecuentes](faq.md)