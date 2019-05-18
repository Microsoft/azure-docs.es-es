---
title: Deshabilitación de los inicios de sesión de usuario de una aplicación empresarial en Azure Active Directory | Microsoft Docs
description: Cómo deshabilitar una aplicación empresarial para que ningún usuario pueda iniciar sesión en ella en Azure Active Directory
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/12/2019
ms.author: mimart
ms.reviewer: asteen
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 90000f34ff247fdd5939dc19971c170aa4b70386
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65824658"
---
# <a name="disable-user-sign-ins-for-an-enterprise-app-in-azure-active-directory"></a>Deshabilitación de los inicios de sesión de usuario de una aplicación empresarial en Azure Active Directory
Es fácil deshabilitar una aplicación empresarial, por lo que ningún usuario puede iniciar sesión en ella en Azure Active Directory (Azure AD). Necesita los permisos adecuados para administrar la aplicación empresarial. Y debe ser administrador global en el directorio.

## <a name="how-do-i-disable-user-sign-ins"></a>¿Cómo puedo deshabilitar los inicios de sesión de usuario?
1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta que tenga el rol de administrador global en el directorio.
1. Seleccione **Todos los servicios**, escriba **Azure Active Directory** en el cuadro de texto y, después, seleccione **Entrar**.
1. En el **Azure Active Directory** -  ***nombredeldirectorio*** panel (es decir, el panel de Azure AD para el directorio que se está administrando), seleccione **aplicaciones empresariales**.
1. En el **aplicaciones empresariales - todas las aplicaciones** panel, verá una lista de las aplicaciones que puede administrar. Seleccione una aplicación.
1. En el panel ***nombreDeLaAplicación*** (es decir, el panel con el nombre de la aplicación seleccionada en el título), seleccione **Propiedades**.
1. En el panel ***nombreDeLaAplicación*** - **Propiedades**, en **¿Habilitado para que los usuarios inicien sesión?**, seleccione **No**.
1. Haga clic en el comando **Guardar** .

## <a name="next-steps"></a>Pasos siguientes
* [Ver todos mis grupos](../fundamentals/active-directory-groups-view-azure-portal.md)
* [Asignar un usuario o grupo a una aplicación empresarial](assign-user-or-group-access-portal.md)
* [Quitar una asignación de usuario o grupo de una aplicación empresarial](remove-user-or-group-access-portal.md)
* [Cambio del nombre o el logotipo de una aplicación empresarial](change-name-or-logo-portal.md)
