---
title: Opciones avanzadas de firma de certificados en el token SAML para aplicaciones previamente integradas en Azure Active Directory | Microsoft Docs
description: Aprenda a usar opciones avanzadas de firma de certificados en el token SAML para aplicaciones previamente integradas en Azure Active Directory
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
ms.date: 03/25/2019
ms.author: mimart
ms.reviewer: jeedes
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1baf4af41fd5dcb6723b6ee2827ae91b43b072d9
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65780960"
---
# <a name="advanced-certificate-signing-options-in-the-saml-token-for-gallery-apps-in-azure-active-directory"></a>Opciones avanzadas de firma de certificados en el token SAML para aplicaciones previamente integradas en Azure Active Directory

En la actualidad Azure Active Directory (Azure AD) es compatible con miles de aplicaciones previamente integradas de la galería de aplicaciones de Azure Active Directory. Admite más de 500 de las aplicaciones de inicio de sesión único mediante el [lenguaje de marcado de aserción de seguridad](https://wikipedia.org/wiki/Security_Assertion_Markup_Language) (SAML) 2.0 del protocolo, como el [NetSuite](https://azuremarketplace.microsoft.com/marketplace/apps/aad.netsuite) aplicación. Cuando un cliente se autentica en una aplicación a través de Azure AD con SAML, Azure AD envía un token a la aplicación (a través de una solicitud HTTP POST). A continuación, la aplicación valida y usa el token para iniciar sesión en el cliente en lugar de solicitar un nombre de usuario y una contraseña. Estos tokens SAML se firman con el certificado único que se genera en Azure AD y mediante algoritmos estándar específicos.

Azure AD usa algunos de los valores predeterminados para las aplicaciones de la galería. Los valores predeterminados se configuran según los requisitos de la aplicación.

En Azure AD, puede configurar opciones de firma y el algoritmo de firma de certificado.

## <a name="certificate-signing-options"></a>Opciones de firma de certificado

Azure AD admite tres opciones avanzadas de firma de certificado:

* **Firmar aserción SAML** Esta opción predeterminada se establece para la mayoría de las aplicaciones de la galería. Si selecciona esta opción, Azure AD como proveedor de identidades (IdP) firma la aserción de SAML y el certificado con la [X.509](https://wikipedia.org/wiki/X.509) certificado de la aplicación.

* **Firmar respuesta SAML** Si selecciona esta opción, Azure AD como IdP firma la respuesta SAML con el certificado X.509 de la aplicación.

* **Firmar respuesta y aserción SAML** Si selecciona esta opción, Azure AD como IdP firma todo el token SAML con el certificado X.509 de la aplicación.

## <a name="certificate-signing-algorithms"></a>Algoritmos de firma de certificado

Azure AD admite dos algoritmos de firmas o algoritmos de hash seguro (SHA), para firmar la respuesta SAML:

* **SHA-256**. Azure AD utiliza este algoritmo predeterminado para firmar la respuesta SAML. Es el más reciente y es más seguro que SHA-1. La mayoría de las aplicaciones admiten el algoritmo SHA-256. Si una aplicación solo admite SHA-1 como algoritmo de firma, puede cambiarlo. En caso contrario, se recomienda utilizar el algoritmo SHA-256 para firmar la respuesta SAML.

* **SHA-1**. Este algoritmo es más antigua y se considera como menos seguro que SHA-256. Si la aplicación admite solo este algoritmo de firma, puede seleccionar esta opción en la lista desplegable **Algoritmo de firma**. Azure AD firma entonces la respuesta SAML con el algoritmo SHA-1.

## <a name="change-certificate-signing-options-and-signing-algorithm"></a>Cambiar opciones de firma y algoritmo de firma de certificado

Para cambiar el certificado SAML de la aplicación opciones de firma y el algoritmo de firma, seleccione la aplicación en cuestión:

1. En el [portal de Azure Active Directory](https://aad.portal.azure.com/), inicie sesión su cuenta. El **centro de administración de Azure Active Directory** aparece la página.
1. En el panel izquierdo, seleccione **Aplicaciones empresariales**. Aparece una lista de las aplicaciones empresariales en su cuenta.
1. Seleccione una aplicación. Aparece una página de información general de la aplicación.

   ![Página de información general de la aplicación](./media/certificate-signing-options/application-overview-page.png)

A continuación, cambie el opciones en el token SAML para esa aplicación de firma de certificado:

1. En el panel izquierdo de la página de información general de la aplicación, seleccione **inicio de sesión único**.

2. Si el **establecer seguridad de sesión único con SAML - versión preliminar** aparece en la página, vaya al paso 5.

3. Si el **seleccionar un método de inicio de sesión único** página no aparece, seleccione **cambiar los modos de inicio de sesión únicos** para mostrar esa página.

4. En el **seleccionar un método de inicio de sesión único** , seleccione **SAML** si está disponible. (Si **SAML** no está disponible, la aplicación no es compatible con SAML, y puede omitir el resto de este procedimiento y el artículo.)

5. En el **establecer seguridad de sesión único con SAML - versión preliminar** página, busque el **certificado de firma SAML** encabezado y seleccione el **editar** icono (un lápiz). El **certificado de firma SAML** aparece la página.

   ![Página de firma de SAML](./media/certificate-signing-options/saml-signing-page.png)

6. En el **opción firma** lista desplegable, elija **respuesta de SAML de inicio de sesión**, **firmar aserción SAML**, o **respuesta y aserción SAML de inicio de sesión**. Descripciones de estas opciones aparecen anteriormente en este artículo en el [opciones de firma de certificado](#certificate-signing-options).

7. En el **algoritmo de firma** lista desplegable, elija **SHA-1** o **SHA-256**. Descripciones de estas opciones aparecen anteriormente en este artículo en el [algoritmos de firma de certificado](#certificate-signing-algorithms) sección.

8. Si está satisfecho con las opciones seleccionadas, seleccione **guardar** para aplicar la configuración de certificados de firma de SAML nuevo. En caso contrario, seleccione el **X** para descartar los cambios.

## <a name="next-steps"></a>Pasos siguientes

* [Configuración del inicio de sesión único en aplicaciones que no están en la galería de aplicaciones de Azure Active Directory](configure-federated-single-sign-on-non-gallery-applications.md)
* [Solución de problemas del inicio de sesión único basado en SAML](../develop/howto-v1-debug-saml-sso-issues.md)
