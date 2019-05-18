---
title: Asignación de usuarios a las aplicaciones | Microsoft Docs
description: Descripción de cómo se asignan los usuarios a una aplicación del inquilino
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/11/2017
ms.author: mimart
ms.collection: M365-identity-device-management
ms.openlocfilehash: b818fe1d8b6bbc9d2d8c5b460b4d71dccdd39366
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65825987"
---
# <a name="how-to-assign-users-to-applications"></a>Asignación de usuarios a las aplicaciones

Este artículo le ayudará a comprender cómo se asignan los usuarios a una aplicación del inquilino.

## <a name="how-do-users-get-assigned-to-an-application-in-azure-ad"></a>¿Cómo se asignan los usuarios a una aplicación de Azure AD?

Para que un usuario tenga acceso a una aplicación, se le debe primero asignar a ella de alguna forma. Un administrador, un delegado de la empresa o, en ocasiones, el usuario mismo puede realizar esta asignación. A continuación se describen las maneras en las que se pueden asignar los usuarios a las aplicaciones:

1.  Un administrador [asigna un usuario](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal) a la aplicación directamente

2.  Un administrador [asigna un grupo](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal), del que el usuario forma parte, a la aplicación, incluidos:

    * Un grupo que se ha sincronizado de forma local

    * Un grupo de seguridad estático creado en la nube

    * Un [grupo de seguridad dinámico](https://docs.microsoft.com/azure/active-directory/active-directory-groups-dynamic-membership-azure-portal) creado en la nube

    * Un grupo de Office 365 creado en la nube

    * El grupo [Todos los usuarios](https://docs.microsoft.com/azure/active-directory/active-directory-accessmanagement-dedicated-groups)

3.  Un administrador habilita [Acceso de autoservicio a las aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-self-service-application-access) para permitir que un usuario pueda agregar una aplicación mediante la característica **Agregar aplicación** del [panel de acceso a las aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction) **sin la aprobación de la empresa**

4.  Un administrador habilita [Acceso de autoservicio a las aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-self-service-application-access) para permitir que un usuario pueda agregar una aplicación mediante la característica **Agregar aplicación** del [panel de acceso a las aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction) pero solo **con la aprobación anterior de un conjunto seleccionado de aprobadores de la empresa**

5.  Un administrador habilita [Administración de grupos de autoservicio](https://docs.microsoft.com/azure/active-directory/active-directory-accessmanagement-self-service-group-management) para permitir que un usuario se una a un grupo al que se ha asignado una aplicación **sin la aprobación de la empresa**

6.  Un administrador habilita [Administración de grupos de autoservicio](https://docs.microsoft.com/azure/active-directory/active-directory-accessmanagement-self-service-group-management) para permitir que un usuario se una a un grupo al que se ha asignado una aplicación pero solo **con la aprobación anterior de un conjunto seleccionado de aprobadores de la empresa**

7.  Un administrador asigna una licencia a un usuario directamente para una aplicación original de Microsoft, como [Microsoft Office 365](https://products.office.com/)

8.  Un administrador asigna una licencia a un grupo del que el usuario forma parte para una aplicación original de Microsoft, como [Microsoft Office 365](https://products.office.com/)

9.  Un [administrador da su consentimiento a una aplicación](https://docs.microsoft.com/azure/active-directory/develop/active-directory-devhowto-multi-tenant-overview) para que la usen todos los usuarios y, a continuación, un usuario inicia sesión en ella

10. Un usuario [da su consentimiento a una aplicación](https://docs.microsoft.com/azure/active-directory/develop/active-directory-devhowto-multi-tenant-overview) por sí mismo iniciando sesión en ella directamente

## <a name="next-steps"></a>Pasos siguientes
[Administración de aplicaciones con Azure Active Directory](what-is-application-management.md)
