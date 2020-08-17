---
title: Ocultación o eliminación de un paquete de acceso en la administración de derechos - Azure AD
description: Aprenda a ocultar o eliminar un paquete de acceso en la administración de derechos de Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 22aa97c6e627c2072636ca2e079877ff0f608b68
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87798807"
---
# <a name="hide-or-delete-an-access-package-in-azure-ad-entitlement-management"></a>Ocultación o eliminación de un paquete de acceso en la administración de derechos de Azure AD

De forma predeterminada, los paquetes de acceso son detectables. Esto significa que si una directiva permite que un usuario solicite el paquete de acceso, verá automáticamente que el paquete de acceso se muestra en el portal Mi acceso. Sin embargo, puede cambiar la configuración de **Oculto** de forma que el paquete de acceso no aparezca en el portal Mi acceso del usuario.

En este artículo se describe cómo ocultar o eliminar un paquete de acceso.

## <a name="change-the-hidden-setting"></a>Cambiar el valor Oculto

Siga estos pasos para cambiar la configuración de **Oculto** de un paquete de acceso.

**Rol necesario:** Administrador global, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. En el menú de la izquierda, haga clic en **Paquetes de acceso** y luego abra el paquete de acceso.

1. En la página Información general, haga clic en **Editar**.

1. Configure el valor **Oculto**.

    Si está establecido en **No**, el paquete de acceso se mostrará en el portal Mi acceso del usuario.

    Si está establecido en **Sí**, el paquete de acceso no se mostrará en el portal Mi acceso del usuario. La única manera que un usuario puede ver el paquete de acceso es si tiene el **vínculo directo del portal Mi acceso** al paquete acceso. Para más información, consulte [Uso compartido de un vínculo para solicitar un paquete de acceso](entitlement-management-access-package-settings.md).

## <a name="delete-an-access-package"></a>Eliminación de un paquete de acceso

Solo se puede eliminar un paquete de acceso si no tiene ninguna asignación de usuarios activa. Siga estos pasos para eliminar un paquete de acceso.

**Rol necesario:** Administrador global, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. En el menú de la izquierda, haga clic en **Paquetes de acceso** y luego abra el paquete de acceso.

1. En el menú de la izquierda, haga clic en **Asignaciones** y quite el acceso para todos los usuarios.

1. En el menú de la izquierda, haga clic en **Información general** y luego en **Eliminar**.

1. En el mensaje de eliminación que aparece, haga clic en **Sí**.

## <a name="next-steps"></a>Pasos siguientes

- [Visualización, incorporación y eliminación de asignaciones en un paquete de acceso](entitlement-management-access-package-assignments.md)
- [Visualización de informes y registros](entitlement-management-reports.md)
