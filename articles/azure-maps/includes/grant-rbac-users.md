---
title: Concesión de acceso basado en roles para usuarios
titleSuffix: Azure Maps
description: Uso del control de acceso basado en roles para conceder a los usuarios autorización para Azure Maps
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/17/2020
ms.topic: include
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.openlocfilehash: 2b138dab2a97537a93b8d873f79b6ee9c00b4af4
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87126759"
---
## <a name="grant-role-based-access-for-users-to-azure-maps"></a>Concesión de acceso basado en roles para usuarios de Azure Maps

El *control de acceso basado en roles* (RBAC) se concede mediante la asignación de un grupo de Azure AD o entidades de seguridad a una o más definiciones de roles de control de acceso de Azure Maps. Para ver las definiciones de roles de RBAC que hay disponibles para Azure Maps, vaya a **Control de acceso (IAM)** . Seleccione **Roles** y busque los roles que comienzan con *Azure Maps*.

* Para administrar de forma eficaz una gran cantidad de acceso de usuarios a Azure Maps, consulte [Grupos de Azure AD](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-manage-groups).
* Para que los usuarios puedan autenticarse en la aplicación, se deben crear en Azure AD. Consulte [Incorporación o eliminación de usuarios mediante Azure AD](https://docs.microsoft.com/azure/active-directory/fundamentals/add-users-azure-active-directory).

Obtenga más información en [Azure AD](https://docs.microsoft.com/azure/active-directory/fundamentals/) para administrar de manera eficaz un directorio para los usuarios.

1. Vaya a su **cuenta de Azure Maps**. Seleccione **Control de acceso (IAM)**  > **Asignación de roles**.

    ![Concesión de RBAC](../media/how-to-manage-authentication/how-to-grant-rbac.png)

2. En la pestaña **Asignaciones de roles**, en **Rol**, seleccione una definición de roles de Azure Maps integrada, como **Lector de datos de Azure Maps** o **Colaborador de datos de Azure Maps**. En **Asignar acceso a**, seleccione **Usuario, grupo o entidad de servicio de Azure AD**. Seleccione la entidad de seguridad por nombre. Después, seleccione **Guardar**.

   * Consulte más información en [Adición y eliminación de asignaciones de roles](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal).

> [!WARNING]
> Las definiciones de roles de Azure Maps integradas proporcionan un acceso de autorización muy amplio a muchas API REST de Azure Maps. Para restringir al mínimo las API para los usuarios, consulte cómo [crear una definición de roles personalizada y asignar usuarios](https://docs.microsoft.com/azure/role-based-access-control/custom-roles) a la definición de roles personalizada. Esto proporcionará a los usuarios el privilegio mínimo necesario para la aplicación.