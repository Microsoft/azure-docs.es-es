---
title: Configuración del inicio de sesión vinculado en Azure Active Directory
description: Configure el inicio de sesión vinculado en Azure AD.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: how-to
ms.workload: identity
ms.date: 07/30/2020
ms.author: kenwith
ms.reviewer: arvinh,luleon
ms.openlocfilehash: 2269a8f7f58d35fee5e2ca30a77af5e8cba83678
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87459341"
---
# <a name="configure-linked-sign-on"></a>Configuración del inicio de sesión vinculado

En la [serie de guías de inicio rápido](view-applications-portal.md) sobre la administración de aplicaciones, aprendió a usar Azure AD como proveedor de identidades (IdP) para una aplicación. En la guía de inicio rápido, configura el inicio de sesión único basado en SAML. Otra opción es **Vinculado**. Este artículo explica con más detalle la opción Vinculado.

La opción **Vinculado** permite configurar la ubicación de destino cuando un usuario selecciona la aplicación en el portal de Office 365 o [Aplicaciones](https://myapplications.microsoft.com/) de la organización.

Estos son algunos de los escenarios comunes en los que la opción de vínculo resulta de gran utilidad:
- Incorporación de un vínculo a una aplicación web personalizada que actualmente use la federación, como Servicios de federación de Active Directory (AD FS).
- Incorporación de vínculos profundos a páginas específicas de SharePoint o a otras páginas web que desee que aparezcan en los paneles de acceso de su usuario.
- Incorporación de un vínculo a una aplicación que no requiera autenticación. 
 
 La opción **Vinculado** no proporciona la funcionalidad de inicio de sesión a través de las credenciales de Azure AD. Sin embargo, puede seguir usando algunas de las otras características de las **aplicaciones empresariales**. Por ejemplo, puede usar registros de auditoría y agregar un logotipo personalizado y un nombre de la aplicación.

## <a name="before-you-begin"></a>Antes de empezar

Para empezar rápidamente, consulte la [serie de guías de inicio rápido](view-applications-portal.md) sobre la administración de aplicaciones. En el inicio rápido, donde configura el inicio de sesión único, también encontrará la opción **Vinculado**. 

La opción **Vinculado** no proporciona la funcionalidad de inicio de sesión a través de Azure AD. La opción solo establece la ubicación a la que se enviarán los usuarios cuando seleccionen la aplicación en [Aplicaciones](https://myapplications.microsoft.com/) o el iniciador de aplicaciones de Microsoft 365.

> [!IMPORTANT] 
> Hay escenarios en los que la opción **Inicio de sesión único** no estará al desplazarse por una aplicación en las **aplicaciones empresariales**. 
>
> Por ejemplo, si la aplicación se registró mediante **Registros de aplicaciones**, la característica de inicio de sesión único estará configurada para usar OAuth de OIDC de forma predeterminada. En este caso, la opción de **Inicio de sesión único** no se mostrará en la navegación en **Aplicaciones empresariales**. Cuando use **Registros de aplicaciones** para agregar su aplicación personalizada, configure las opciones en el archivo de manifiesto. Para obtener más información sobre el archivo de manifiesto, consulte [Manifiesto de aplicación de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/reference-app-manifest). Para obtener más información sobre los estándares de SSO, consulte [Autenticación y autorización mediante la Plataforma de identidad de Microsoft](https://docs.microsoft.com/azure/active-directory/develop/authentication-vs-authorization#authentication-and-authorization-using-microsoft-identity-platform). 
>
> Otros escenarios en los que falta **inicio de sesión único** en la navegación incluyen cuándo se hospeda una aplicación en otro inquilino o si su cuenta no tiene los permisos necesarios (Administrador global, Administrador de aplicaciones en la nube, Administrador de la aplicación o propietario de la entidad de servicio). Los permisos también pueden provocar un escenario en el que puede abrir **Inicio de sesión único** pero no podrá guardar. Para más información acerca de los roles administrativos de Azure AD, consulte (https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles).

### <a name="configure-link"></a>Configuración de un vínculo

A fin de establecer un vínculo para una aplicación, seleccione **Vinculado** en la página **Inicio de sesión único**. A continuación, escriba el vínculo y seleccione **Guardar**. ¿Necesita un recordatorio de dónde encontrar estas opciones? Consulte la [serie de guías de inicio rápido](view-applications-portal.md).
 
Una vez que configure una aplicación, asigne usuarios y grupos a la misma. Al asignar usuarios, puede controlar cuándo aparecerá la aplicación en [Aplicaciones](https://myapplications.microsoft.com/) o en el iniciador de aplicaciones de Microsoft 365.

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de usuarios y grupos a la aplicación](methods-for-assigning-users-and-groups.md)
- [Configuración del aprovisionamiento automático de cuentas de usuario](../app-provisioning/configure-automatic-user-provisioning-portal.md)
