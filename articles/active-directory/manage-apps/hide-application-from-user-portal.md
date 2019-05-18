---
title: Ocultación de una aplicación de la experiencia del usuario en Azure Active Directory | Microsoft Docs
description: Cómo ocultar una aplicación de la experiencia del usuario en los paneles de acceso de Azure Active Directory o los iniciadores de Office 365.
services: active-directory
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 11/12/2018
ms.author: mimart
ms.reviewer: kasimpso
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3dd98aa974f2adcd363c04c10b7a10cef6ca8ce7
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65824539"
---
# <a name="hide-applications-from-end-users-in-azure-active-directory"></a>Ocultar aplicaciones de usuarios finales en Azure Active Directory

Instrucciones sobre cómo ocultar las aplicaciones del panel MyApps o del iniciador de Office 365 de los usuarios finales. Aunque una aplicación se oculte, los usuarios seguirán teniendo los permisos para ella. 

## <a name="prerequisites"></a>Requisitos previos

Se requieren privilegios de administrador de aplicaciones para ocultar una aplicación del panel MyApps y del iniciador de Office 365.

Se requieren privilegios de administrador global para ocultar todas las aplicaciones de Office 365.


## <a name="hide-an-application-from-the-end-user"></a>Ocultar una aplicación al usuario final
Siga estos pasos para ocultar una aplicación del panel MyApps y del iniciador de aplicaciones de Office 365.

1.  Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global de su directorio.
2.  Seleccione **Azure Active Directory**.
3.  Seleccione **Aplicaciones empresariales**. Se abrirá la hoja **Aplicaciones empresariales - Todas las aplicaciones**.
4.  En **Tipo de aplicación**, seleccione **Aplicaciones empresariales**, si aún no está seleccionado.
5.  Busque la aplicación que quiere ocultar y haga clic en la aplicación.  Se abrirá la información general de la aplicación.
6.  Haga clic en **Propiedades**. 
7.  Para la pregunta **¿Es visible para los usuarios?**, haga clic en **No**.
8.  Haga clic en **Save**(Guardar).


## <a name="hide-office-365-applications-from-the-myapps-panel"></a>Ocultar las aplicaciones de Office 365 del panel MyApps

Siga estos pasos para ocultar todas las aplicaciones de Office 365 del panel MyApps. Las aplicaciones siguen siendo visibles en el portal de Office 365.

1.  Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global de su directorio.
2.  Seleccione **Azure Active Directory**.
3.  Seleccione **Configuración de usuario**.
4.  En **Aplicaciones empresariales**, haga clic en **Administrar cómo los usuarios finales inician y ven sus aplicaciones**.
5.  En **Los usuarios solo pueden ver las aplicaciones de Office 365 en el Portal de Office 365**, haga clic en **Sí**.
6.  Haga clic en **Save**(Guardar).


## <a name="next-steps"></a>Pasos siguientes
* [Ver todos mis grupos](../fundamentals/active-directory-groups-view-azure-portal.md)
* [Asignar un usuario o grupo a una aplicación empresarial](assign-user-or-group-access-portal.md)
* [Quitar una asignación de usuario o grupo de una aplicación empresarial](remove-user-or-group-access-portal.md)
* [Cambio del nombre o el logotipo de una aplicación empresarial](change-name-or-logo-portal.md)

