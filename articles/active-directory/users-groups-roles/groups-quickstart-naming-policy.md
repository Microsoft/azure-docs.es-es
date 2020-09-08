---
title: Inicio rápido de directiva de nomenclatura de grupos - Azure Active Directory | Microsoft Docs
description: Describe como agregar usuarios nuevos o eliminar usuarios existentes en Azure Active Directory.
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: quickstart
ms.date: 08/13/2020
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: e5b54af39401ba8c3fede4c9ad80ec8edc963b1d
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88213674"
---
# <a name="quickstart-naming-policy-for-groups-in-azure-active-directory"></a>Inicio rápido: Directiva de nomenclatura para grupos de Azure Active Directory

En este inicio rápido, configurará una directiva de nomenclatura en la organización de Azure Active Directory (Azure AD) para los grupos de Microsoft 365 creados por el usuario, a fin de ayudarle a ordenar y buscar los grupos de la organización. Por ejemplo, podría usar la directiva de nomenclatura para:

* Comunicar la función de un grupo, la pertenencia, la región geográfica o quién creó el grupo.
* Ayudar a clasificar los grupos en la libreta de direcciones.
* Impedir que determinadas palabras se usen en los nombres y alias de grupo.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="configure-the-group-naming-policy-in-the-azure-portal"></a>Configuración de la directiva de nomenclatura de grupos en Azure Portal

1. Inicie sesión en el [Centro de administración de Azure AD](https://aad.portal.azure.com) con una cuenta de administrador de usuarios.
1. Seleccione **Grupos**, a continuación, seleccione **Directiva de nomenclatura** para abrir la página de la directiva de nomenclatura.

    ![apertura de la página de la directiva de nomenclatura en el centro de administración](./media/groups-naming-policy/policy.png)

### <a name="view-or-edit-the-prefix-suffix-naming-policy"></a>Visualización o edición de la directiva de nomenclatura de prefijo-sufijo

1. En la página **Directiva de nomenclatura**, seleccione **Directiva de nomenclatura de grupos**.
1. Puede ver o editar las directivas de nomenclatura de prefijo o sufijo actuales individualmente seleccionando los atributos o cadenas que desea aplicar como parte de la directiva de nomenclatura.
1. Para quitar un prefijo o sufijo de la lista, seleccione el prefijo o sufijo y luego seleccione **Eliminar**. Se pueden eliminar varios elementos al mismo tiempo.
1. Seleccione **Guardar** para que los cambios en la directiva entren en vigor.

### <a name="view-or-edit-the-custom-blocked-words"></a>Visualización o edición de las palabras bloqueadas personalizadas

1. En la página **Directiva de nomenclatura**, seleccione **Palabras bloqueadas**.

    ![edición y carga de la lista de palabras bloqueadas para la directiva de nomenclatura](./media/groups-naming-policy/blockedwords.png)

1. Vea o edite la lista actual de palabras bloqueadas personalizadas seleccionando **Descargar**.
1. Cargue la nueva lista de palabras bloqueadas personalizadas seleccionando el icono del archivo.
1. Seleccione **Guardar** para que los cambios en la directiva entren en vigor.

Eso es todo. Ha establecido la directiva de nomenclatura y ha agregado las palabras bloqueadas personalizadas.

## <a name="clean-up-resources"></a>Limpieza de recursos

### <a name="remove-the-naming-policy-using-azure-portal"></a>Eliminación de la directiva de nomenclatura mediante Azure Portal

1. En la página **Directiva de nomenclatura**, seleccione **Eliminar directiva**.
1. Después de confirmar la eliminación, se quita la directiva de nomenclatura, incluida toda la directiva de nomenclatura de prefijos y sufijos y cualquier palabra bloqueada personalizada.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a establecer la directiva de nomenclatura para la organización de Azure AD mediante Azure Portal.

Avance al artículo siguiente para obtener más información acerca de los cmdlets de PowerShell para la directiva de nomenclatura, las limitaciones técnicas, la adición de una lista de palabras bloqueadas personalizadas y las experiencias de usuario final en las aplicaciones de Office 365.
> [!div class="nextstepaction"]
> [Naming policy PowerShell](groups-naming-policy.md) (Directiva de nomenclatura de PowerShell)