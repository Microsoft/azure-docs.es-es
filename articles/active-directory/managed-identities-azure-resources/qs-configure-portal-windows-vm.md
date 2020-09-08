---
title: 'Configuración de identidades administradas mediante Azure Portal: Azure AD'
description: Instrucciones detalladas para configurar identidades administradas para recursos de Azure en una VM de Azure mediante Azure Portal.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/10/2018
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2ed19b9dae179c3e4fe89a9ce28df4ad0cd080af
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89263219"
---
# <a name="configure-managed-identities-for-azure-resources-on-a-vm-using-the-azure-portal"></a>Configurar identidades administradas para recursos de Azure en una VM mediante Azure Portal

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Las identidades administradas para los recursos de Azure proporcionan a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory. Puede usar esta identidad para autenticar a cualquier servicio que admita la autenticación de Azure AD, sin necesidad de tener credenciales en el código. 

En este artículo, aprenderá a habilitar y deshabilitar las identidades administradas asignadas por el usuario y el sistema para una máquina virtual (VM) de Azure mediante Azure Portal. 

## <a name="prerequisites"></a>Prerrequisitos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md).
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.

## <a name="system-assigned-managed-identity"></a>Identidad administrada asignada por el sistema

En esta sección, aprenderá a habilitar y deshabilitar la identidad administrada asignada por el sistema para una VM mediante Azure Portal.

### <a name="enable-system-assigned-managed-identity-during-creation-of-a-vm"></a>Habilitación de una identidad administrada asignada por el sistema durante la creación de una VM

Para habilitar una identidad administrada asignada por el sistema en una máquina virtual durante su creación, la cuenta necesita la asignación de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  No se requiere ninguna otra asignación de roles de directorio de Azure AD.

- En el pestaña **Administración** de la sección **Identidad** cambie **Managed Service Identity** a **Activado**.  

![Habilitación de una identidad asignada por el sistema durante la creación de una máquina virtual](./media/msi-qs-configure-portal-windows-vm/enable-system-assigned-identity-vm-creation.png)

Consulte las siguientes guías de inicio rápido para crear una máquina virtual: 

- [Creación de una máquina virtual Windows desde Azure Portal](../../virtual-machines/windows/quick-create-portal.md#create-virtual-machine) 
- [Creación de una máquina virtuales Linux desde Azure Portal](../../virtual-machines/linux/quick-create-portal.md#create-virtual-machine)


### <a name="enable-system-assigned-managed-identity-on-an-existing-vm"></a>Habilitación de una identidad administrada asignada por el sistema en una VM existente

Para habilitar una identidad administrada asignada por el sistema en una máquina virtual que en un principio se aprovisionó sin dicha identidad, la cuenta necesita la asignación de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  No se requiere ninguna otra asignación de roles de directorio de Azure AD.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta asociada a la suscripción de Azure que contiene la máquina virtual.

2. Vaya a la máquina virtual que desee y seleccione la página **Identidad**.

3. En **Asignado por el sistema**, en **Estado**, seleccione **Activado** y haga clic en **Guardar**:

   ![Captura de pantalla de la página de configuración](./media/msi-qs-configure-portal-windows-vm/create-windows-vm-portal-configuration-blade.png)  

### <a name="remove-system-assigned-managed-identity-from-a-vm"></a>Quitar una identidad administrada asignada por el sistema de una VM

Para eliminar una identidad administrada asignada por el sistema de una máquina virtual, la cuenta necesita la asignación de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  No se requiere ninguna otra asignación de roles de directorio de Azure AD.

Si tiene una máquina virtual que ya no necesita la identidad administrada asignada por el sistema:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta asociada a la suscripción de Azure que contiene la máquina virtual. 

2. Vaya a la máquina virtual que desee y seleccione la página **Identidad**.

3. En **Asignado por el sistema**, en **Estado**, seleccione **Desactivado** y haga clic en **Guardar**:

   ![Captura de pantalla de la página de configuración](./media/msi-qs-configure-portal-windows-vm/create-windows-vm-portal-configuration-blade-disable.png)

## <a name="user-assigned-managed-identity"></a>Identidad administrada asignada por el usuario

 En esta sección, aprenderá a agregar y quitar una identidad administrada asignada por el usuario desde una VM mediante Azure Portal.

### <a name="assign-a-user-assigned-identity-during-the-creation-of-a-vm"></a>Asignación de una identidad asignada por el usuario durante la creación de una VM

Para asignar una identidad asignada por un usuario a una máquina virtual, la cuenta debe tener las asignaciones de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) y [Operador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-operator). No se requiere ninguna otra asignación de roles de directorio de Azure AD.

En la actualidad, Azure Portal no permite asignar una identidad administrada asignada por el usuario durante la creación de una VM. En su lugar, consulte uno de los artículos de inicio rápido sobre la creación de VM para crear primero una VM y continúe en la siguiente sección, donde encontrará información acerca de cómo asignar la identidad administrada asignada por el usuario a la VM:

- [Creación de una máquina virtual Windows desde Azure Portal](../../virtual-machines/windows/quick-create-portal.md#create-virtual-machine)
- [Creación de una máquina virtuales Linux desde Azure Portal](../../virtual-machines/linux/quick-create-portal.md#create-virtual-machine)

### <a name="assign-a-user-assigned-managed-identity-to-an-existing-vm"></a>Asignación de una identidad administrada asignada por el usuario a una VM existente

Para asignar una identidad asignada por un usuario a una máquina virtual, la cuenta debe tener las asignaciones de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) y [Operador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-operator). No se requiere ninguna otra asignación de roles de directorio de Azure AD.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta asociada a la suscripción de Azure que contiene la máquina virtual.
2. Vaya a la máquina virtual que desee y haga clic en **Identidad**, **Asignado por el usuario** y en **\+Agregar**.

   ![Agregar identidad administrada asignada por el usuario a una VM](./media/msi-qs-configure-portal-windows-vm/add-user-assigned-identity-vm-screenshot1.png)

3. Haga clic en la identidad asignada por el usuario que quiera agregar a la VM y, a continuación, en **Agregar**.

    ![Agregar identidad administrada asignada por el usuario a una VM](./media/msi-qs-configure-portal-windows-vm/add-user-assigned-identity-vm-screenshot2.png)

### <a name="remove-a-user-assigned-managed-identity-from-a-vm"></a>Eliminación de una identidad administrada asignada por el usuario de una VM

Para eliminar una identidad asignada por el usuario de una máquina virtual, la cuenta necesita la asignación de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor). No se requiere ninguna otra asignación de roles de directorio de Azure AD.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta asociada a la suscripción de Azure que contiene la máquina virtual.
2. Vaya a la VM que quiera y haga clic en **Identidad**, en **Asignado por el usuario**, en el nombre de la identidad administrada asignada por el usuario que quiera eliminar y en **Quitar** (haga clic en **Sí** en el panel de confirmación).

   ![Eliminación de una identidad asignada por el usuario de una VM](./media/msi-qs-configure-portal-windows-vm/remove-user-assigned-identity-vm-screenshot.png)

## <a name="next-steps"></a>Pasos siguientes

- Con Azure Portal, dé a la identidad administrada de una VM de Azure [acceso a otro recurso de Azure](howto-assign-access-portal.md).