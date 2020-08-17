---
title: 'Inicio rápido: Configuración del inicio de sesión único (SSO) de una aplicación en el inquilino de Azure Active Directory (Azure AD)'
description: En este inicio rápido se describe el proceso de configuración del inicio de sesión único (SSO) de una aplicación en el inquilino de Azure Active Directory (Azure AD).
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: quickstart
ms.workload: identity
ms.date: 07/01/2020
ms.author: kenwith
ms.collection: M365-identity-device-management
ms.openlocfilehash: 461a424f23161e1567c6b32a38db0225efc56b1e
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87808395"
---
# <a name="quickstart-set-up-single-sign-on-sso-for-an-application-in-your-azure-active-directory-azure-ad-tenant"></a>Inicio rápido: Configuración del inicio de sesión único (SSO) de una aplicación en el inquilino de Azure Active Directory (Azure AD)

Comience a usar inicios de sesión de usuario simplificados mediante la configuración del inicio de sesión único (SSO) para una aplicación que haya agregado al inquilino de Azure Active Directory (Azure AD). Después de configurar el inicio de sesión único, los usuarios podrán iniciar sesión en una aplicación con sus credenciales de Azure AD. El inicio de sesión único se incluye en la edición gratuita de Azure AD.

## <a name="prerequisites"></a>Requisitos previos

Para configurar el inicio de sesión único de una aplicación que haya agregado a su inquilino de Azure AD, necesita:

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Uno de los siguientes roles: Administrador global, Administrador de aplicaciones en la nube, Administrador de aplicaciones o Propietario de la entidad de servicio.
- Una aplicación que admita el inicio de sesión único y que se haya configurado previamente y agregado ya a la galería de Azure AD. La mayoría de las aplicaciones pueden usar Azure AD para el inicio de sesión único. Las aplicaciones de la galería de Azure AD están preconfiguradas. Aunque la aplicación no aparezca en la lista, o se trate de una aplicación desarrollada de forma personalizada, podrá usarla de todas formas con Azure AD. Consulte los tutoriales y otra documentación adicional en la tabla de contenido. Este inicio rápido se centra en las aplicaciones que se configuraron previamente para el inicio de sesión único, y que los desarrolladores de aplicaciones han agregado a la galería de Azure AD.
- Opcional: Haber finalizado el inicio rápido de [Visualización de las aplicaciones](view-applications-portal.md).
- Opcional: Haber finalizado el inicio rápido de [Incorporación de una aplicación](add-application-portal.md).
- Opcional: Haber finalizado el inicio rápido de [Configuración de una aplicación](add-application-portal-configure.md).


>[!IMPORTANT]
>Para probar los pasos de este inicio rápido, use un entorno que no sea de producción.


## <a name="enable-single-sign-on-for-an-app"></a>Habilitar el inicio de sesión único de una aplicación

Cuando termine de agregar una aplicación al inquilino de Azure AD, aparecerá la página información general. Si está configurando una aplicación que ya se había agregado, consulte el primer inicio rápido. Este le guía a través de la visualización de las aplicaciones agregadas a su inquilino. 

Para configurar el inicio de sesión único de una aplicación:

1. En el portal de Azure AD, seleccione **Aplicaciones empresariales**. A continuación, busque y seleccione la aplicación que desea configurar para el inicio de sesión único.
1. En la sección **Administrar**, seleccione **Inicio de sesión único** para abrir el panel **Inicio de sesión único**.

    :::image type="content" source="media/add-application-portal-setup-sso/configure-sso.png" alt-text="Captura de pantalla de la página de configuración del inicio de sesión único en el portal de Azure AD":::.

1. Seleccione **SAML** para abrir la página de configuración del inicio de sesión único. En este ejemplo, la aplicación que se está configurando para el inicio de sesión único es GitHub. Después de configurar GitHub, los usuarios podrán iniciar sesión en GitHub con sus credenciales desde su inquilino de Azure AD.

    :::image type="content" source="media/add-application-portal-setup-sso/github-sso.png" alt-text="Captura de pantalla de la página de configuración del inicio de sesión único en GitHub":::.

1. El proceso para configurar una aplicación para que use Azure AD para el inicio de sesión único basado en SAML varía en función de la aplicación. Hay un vínculo a la guía de GitHub. Para buscar guías para otras aplicaciones, consulte [Tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/saas-apps/).
1. Siga la guía para configurar el inicio de sesión único de la aplicación. Muchas aplicaciones tienen requisitos de suscripción específicos para la funcionalidad de inicio de sesión único. Por ejemplo, GitHub requiere una suscripción Enterprise.
    > [!TIP]
    > Para más información sobre las opciones de configuración de SAML, consulte [Configuración del inicio de sesión único basado en SAML](configure-saml-single-sign-on.md).

    :::image type="content" source="media/add-application-portal-setup-sso/github-pricing.png" alt-text="Captura de pantalla que muestra la opción de inicio de sesión único en la suscripción Enterprise de la página de precios de GitHub":::.


> [!TIP]
> La administración de aplicaciones se puede automatizar mediante Graph API, consulte el artículo sobre la [automatización de la administración de aplicaciones con Microsoft Graph API](https://docs.microsoft.com/graph/application-saml-sso-configure-api).


## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con esta serie de inicios rápidos, considere la posibilidad de eliminar la aplicación para limpiar el inquilino de prueba. La eliminación de la aplicación se trata en el último inicio rápido de esta serie, consulte [Eliminación de una aplicación](delete-application-portal.md).

## <a name="next-steps"></a>Pasos siguientes

En el siguiente artículo aprenderá a eliminar una aplicación.
> [!div class="nextstepaction"]
> [Eliminación de una aplicación](delete-application-portal.md)
