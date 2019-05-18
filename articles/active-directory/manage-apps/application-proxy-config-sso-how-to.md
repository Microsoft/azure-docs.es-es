---
title: Configuración del inicio de sesión único en una aplicación de proxy de aplicación | Microsoft Docs
description: Cómo configurar rápidamente el inicio de sesión único en su aplicación de proxy de aplicación
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
ms.topic: conceptual
ms.date: 03/12/2019
ms.author: mimart
ms.reviewer: japere, asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: f26b28b34a569673b397fa4700c5332c3550500f
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65825864"
---
# <a name="how-to-configure-single-sign-on-to-an-application-proxy-application"></a>Configuración del inicio de sesión único en una aplicación de proxy de aplicación

El inicio de sesión único (SSO) permite que los usuarios accedan a una aplicación sin tener que autenticarse varias veces. Permite que se lleve a cabo la autenticación única en la nube, en Azure Active Directory, y que el servicio o el conector suplante al usuario para completar los desafíos de autenticación adicionales de la aplicación.

## <a name="how-to-configure-single-sign-on"></a>Configuración del inicio de sesión único
Para configurar SSO, asegúrese de que la aplicación esté configurada para la autenticación previa a través de Azure Active Directory. Para realizar esta configuración, vaya a **Azure Active Directory** -&gt; **Aplicaciones empresariales** -&gt; **Todas las aplicaciones** -&gt; Su aplicación **-&gt; Proxy de la aplicación**. En esta página, asegúrese de que el campo "Autenticación previa" esté establecido en "Azure Active Directory". 

Para más información sobre los métodos de autenticación previa, consulte el paso 4 del [documento sobre publicación de aplicaciones](application-proxy-add-on-premises-application.md).

   ![Método de autenticación previa en Azure Portal](./media/application-proxy-config-sso-how-to/app-proxy.png)

## <a name="configuring-single-sign-on-modes-for-application-proxy-applications"></a>Configuración de modos de inicio de sesión únicos para aplicaciones de proxy de aplicación
Configure el tipo específico de inicio de sesión único. Los métodos de inicio de sesión se clasifican en función del tipo de autenticación que la aplicación back-end use. Las aplicaciones de proxy de aplicación admiten tres tipos de inicio de sesión:

-   **Inicio de sesión con contraseña**: el inicio de sesión con contraseña sirve para cualquier aplicación que utilice los campos de nombre de usuario y contraseña para iniciar sesión. Los pasos de configuración están disponibles en [Configuración del inicio de sesión único con contraseña para una aplicación de la galería de Azure AD](configure-password-single-sign-on-gallery-applications.md).

-   **Autenticación integrada de Windows**: en el caso de las aplicaciones que utilizan la Autenticación integrada de Windows (IWA), el inicio de sesión único se habilita mediante de la delegación limitada de Kerberos (KCD). Este método concede permiso a los conectores de proxy de aplicación en Active Directory para suplantar a los usuarios, y para enviar y recibir tokens en su nombre. Puede ver más información sobre la configuración de KCD en la [documentación sobre el inicio de sesión único con KCD](application-proxy-configure-single-sign-on-with-kcd.md).

-   **Inicio de sesión basado en el encabezado**: se habilita mediante una asociación y requiere una configuración adicional. Para ver detalles sobre la asociación y las instrucciones paso a paso para configurar el inicio de sesión único para una aplicación que use encabezados para la autenticación, consulte la [documentación de PingAccess para Azure AD](application-proxy-configure-single-sign-on-with-ping-access.md).

-   **Inicio de sesión único en SAML**: Con SAML single sign-on, Azure AD autentica a la aplicación mediante el uso de la cuenta de Azure AD del usuario. Azure AD comunica la información de inicio de sesión a la aplicación a través de un protocolo de conexión. Con el inicio de sesión único basado en SAML puede asignar usuarios a roles de aplicación específicos según las reglas que defina en las notificaciones SAML. Para obtener información acerca de cómo configurar el inicio de sesión único de SAML, vea [SAML para inicio de sesión único con el Proxy de aplicación](application-proxy-configure-single-sign-on-on-premises-apps.md).

Puede encontrar todas estas opciones si va a "Aplicaciones empresariales" en la aplicación y abre la página **Inicio de sesión único** en el menú de la izquierda. Tenga en cuenta que si la aplicación se creó en el portal antiguo, es posible que no vea todas estas opciones.

En esta página, también verá una opción adicional para el inicio de sesión: Inicio de sesión vinculado. El proxy de aplicación también admite esta opción. Sin embargo, esta opción no agrega el inicio de sesión único a la aplicación. Dicho esto, es posible que ya se haya implementado el inicio de sesión único en la aplicación mediante otro servicio, como los Servicios de federación de Active Directory. 

Esta opción permite que un administrador cree un vínculo a una aplicación que los usuarios ven en primer lugar al acceder a la aplicación. Por ejemplo, si hay una aplicación que está configurada para autenticar a usuarios mediante Servicios de federación de Active Directory 2.0, el administrador puede usar la opción "Inicio de sesión vinculado" para crear un vínculo a ella en el panel de acceso.

## <a name="next-steps"></a>Pasos siguientes
- [Almacén de contraseñas para el inicio de sesión único con el proxy de aplicación](application-proxy-configure-single-sign-on-password-vaulting.md)
- [Delegación limitada de Kerberos para el inicio de sesión único con el proxy de aplicación](application-proxy-configure-single-sign-on-with-kcd.md)
- [Autenticación basada en el encabezado para el inicio de sesión único con el proxy de aplicación](application-proxy-configure-single-sign-on-with-ping-access.md) 
- [SAML de inicio de sesión único con el Proxy de aplicación](application-proxy-configure-single-sign-on-on-premises-apps.md).
