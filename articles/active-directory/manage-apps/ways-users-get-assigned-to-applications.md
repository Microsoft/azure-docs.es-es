---
title: Explicación del proceso de asignación de usuarios a aplicaciones en Azure Active Directory
description: Comprenda cómo se asignan los usuarios a una aplicación que usa Azure Active Directory para la administración de identidades.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 07/11/2017
ms.author: kenwith
ms.openlocfilehash: 243e5ece0d0a14cb7e3ade409ee68510cef64a9c
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89397107"
---
# <a name="understand-how-users-are-assigned-to-apps-in-azure-active-directory"></a>Explicación del proceso de asignación de usuarios a aplicaciones en Azure Active Directory
Este artículo le ayudará a comprender cómo se asignan los usuarios a una aplicación del inquilino.

## <a name="how-do-users-get-assigned-to-an-application-in-azure-ad"></a>¿Cómo se asignan los usuarios a una aplicación de Azure AD?
Para que un usuario tenga acceso a una aplicación, se le debe primero asignar a ella de alguna forma. Un administrador, un delegado de la empresa o, en ocasiones, el usuario mismo puede realizar esta asignación. A continuación se describen las maneras en las que se pueden asignar los usuarios a las aplicaciones:

*  Un administrador [asigna un usuario](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal) a la aplicación directamente
*  Un administrador [asigna un grupo](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal), del que el usuario forma parte, a la aplicación, incluidos:
    * Un grupo que se ha sincronizado de forma local
    * Un grupo de seguridad estático creado en la nube
    * Un [grupo de seguridad dinámico](https://docs.microsoft.com/azure/active-directory/active-directory-groups-dynamic-membership-azure-portal) creado en la nube
    * Un grupo de Office 365 creado en la nube
    * El grupo [Todos los usuarios](https://docs.microsoft.com/azure/active-directory/active-directory-accessmanagement-dedicated-groups)
*  Un administrador habilita [Acceso de autoservicio a las aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-self-service-application-access) para permitir que un usuario agregue una aplicación mediante la característica **Agregar aplicación** de la página [Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction) **sin aprobación de la empresa**.
*  Un administrador habilita [Acceso de autoservicio a las aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-self-service-application-access) para permitir que un usuario agregue una aplicación mediante la característica **Agregar aplicación** de la página [Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction) pero solo **con la aprobación previa de un conjunto seleccionado de aprobadores de la empresa**.
*  Un administrador habilita [Administración de grupos de autoservicio](https://docs.microsoft.com/azure/active-directory/active-directory-accessmanagement-self-service-group-management) para permitir que un usuario se una a un grupo al que se ha asignado una aplicación **sin la aprobación de la empresa**
*  Un administrador habilita [Administración de grupos de autoservicio](https://docs.microsoft.com/azure/active-directory/active-directory-accessmanagement-self-service-group-management) para permitir que un usuario se una a un grupo al que se ha asignado una aplicación pero solo **con la aprobación anterior de un conjunto seleccionado de aprobadores de la empresa**
*  Un administrador asigna una licencia a un usuario directamente para una aplicación original de Microsoft, como [Microsoft Office 365](https://products.office.com/)
*  Un administrador asigna una licencia a un grupo del que el usuario forma parte para una aplicación original de Microsoft, como [Microsoft Office 365](https://products.office.com/)
*  Un [administrador da su consentimiento a una aplicación](https://docs.microsoft.com/azure/active-directory/develop/active-directory-devhowto-multi-tenant-overview) para que la usen todos los usuarios y, a continuación, un usuario inicia sesión en ella
* Un usuario [da su consentimiento a una aplicación](https://docs.microsoft.com/azure/active-directory/develop/active-directory-devhowto-multi-tenant-overview) por sí mismo iniciando sesión en ella directamente

## <a name="next-steps"></a>Pasos siguientes
* [Serie de guías de inicio rápido sobre la administración de aplicaciones](view-applications-portal.md)
* [¿Qué es la administración de aplicaciones?](what-is-application-management.md)
* [¿Qué es el inicio de sesión único?](what-is-single-sign-on.md)
