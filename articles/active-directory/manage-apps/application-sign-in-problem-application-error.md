---
title: Error en la página de aplicación después de iniciar sesión | Microsoft Docs
description: Cómo resolver problemas con el inicio de sesión de Azure AD en cuando la propia aplicación genera un error
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
ms.date: 07/11/2017
ms.author: mimart
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 234fc94a0a2ad3d3bee49853cf1dfc6a805a0166
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65825397"
---
# <a name="error-on-an-applications-page-after-signing-in"></a>Error en la página de aplicación después de iniciar sesión

En este escenario, Azure AD ha iniciado la sesión del usuario, pero la aplicación muestra un error que no le permite finalizar correctamente el flujo de inicio de sesión. En este escenario, la aplicación no acepta el problema de respuesta de Azure AD.

Hay algunos posibles motivos de por qué la aplicación no aceptó la respuesta de Azure AD. Si el error de la aplicación no es lo suficientemente claro para saber lo que falta en la respuesta:

-   Si la aplicación es la Galería de Azure AD, compruebe que ha seguido todos los pasos del artículo [Cómo depurar el inicio de sesión único basado en SAML en aplicaciones de Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-saml-debugging).

-   Utilice una herramienta como [Fiddler](https://www.telerik.com/fiddler) para capturar la solicitud SAML, la respuesta de SAML y el token de SAML.

-   Comparta la respuesta de SAML con el proveedor de la aplicación para que sepan qué es lo que falta.

## <a name="missing-attributes-in-the-saml-response"></a>Faltan atributos en la respuesta SAML

Para agregar un atributo en la configuración de Azure AD para enviarlo en la respuesta de Azure AD, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6. Seleccione la aplicación que desea configurar para el inicio de sesión único.

7. Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.

8. Haga clic en **Ver y editar todos los demás atributos de usuario** en la sección **Atributos de usuario** para editar los atributos que se enviarán a la aplicación en el token SAML cuando los usuarios inicien sesión.

   Para agregar un atributo:

   * Haga clic en **Agregar atributo**. Escriba el **Nombre** y seleccione el **Valor** de la lista desplegable.

   * Haga clic en **Guardar**. En la tabla verá el nuevo atributo.

9. Guarde la configuración.

La próxima vez que el usuario inicie sesión en la aplicación, Azure AD enviará el nuevo atributo en la respuesta de SAML.

## <a name="the-application-expects-a-different-user-identifier-value-or-format"></a>La aplicación espera un formato o un valor de identificador de usuario diferente

El inicio de sesión en la aplicación falla porque la respuesta de SAML no encuentra los atributos, por ejemplo, los roles, o porque la aplicación espera un formato diferente para el atributo EntityID.

## <a name="add-an-attribute-in-the-azure-ad-application-configuration"></a>Incorporación de un atributo en la configuración de la aplicación de Azure AD:

Para cambiar el valor del identificador de usuario, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6. Seleccione la aplicación que desea configurar para el inicio de sesión único.

7. Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.

8. En **Atributos de usuario**, seleccione el identificador único para los usuarios de la lista desplegable **Identificador de usuario**.

## <a name="change-entityid-user-identifier-format"></a>Cambiar el formato de EntityID (Identificador de usuario)

Cuando la aplicación espera otro formato para el atributo EntityID. En este caso, no podrá seleccionar el formato de EntityID (Identificador de usuario) que Azure AD envía a la aplicación en la respuesta a la autenticación de usuario.

AD Azure selecciona el formato del atributo NameID (Identificador de usuario) en función del valor seleccionado o del formato que solicite la aplicación en el elemento AuthRequest de SAML. Para más información, visite la sección NameIDPolicy del artículo [Protocolo SAML de inicio de sesión único](https://docs.microsoft.com/azure/active-directory/develop/active-directory-single-sign-on-protocol-reference#authnrequest).

## <a name="the-application-expects-a-different-signature-method-for-the-saml-response"></a>La aplicación espera un método de firma diferente para la respuesta de SAML

Para cambiar las partes del token de SAML que Azure Active Directory firma digitalmente, Siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6. Seleccione la aplicación que desea configurar para el inicio de sesión único.

7. Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.

8. Haga clic en **Mostrar configuración avanzada de firma de certificados** en la sección **Certificado de firma de SAML**.

9. Seleccione la **Opción de firma** que espera la aplicación:

   * Firmar respuesta SAML

   * Firmar respuesta y aserción SAML

   * Firmar aserción SAML

La próxima vez que el usuario inicie sesión en la aplicación, Azure AD firmará la parte de la respuesta de SAML seleccionada.

## <a name="the-application-expects-the-signing-algorithm-to-be-sha-1"></a>La aplicación espera que el algoritmo de firma sea SHA-1

De forma predeterminada, Azure AD firma el token de SAML con el algoritmo de seguridad máxima. No se recomienda cambiar el algoritmo de firma a SHA-1, a menos que lo necesite la aplicación.

Para cambiar el algoritmo de firma, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6. Seleccione la aplicación que desea configurar para el inicio de sesión único.

7. Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.

8. Haga clic en **Mostrar configuración avanzada de firma de certificados** en la sección **Certificado de firma de SAML**.

9. En el **Algoritmo de firma**, seleccione SHA-1.

La próxima vez que el usuario inicie sesión en la aplicación, Azure AD firmará el token de SAML con el algoritmo SHA-1.

## <a name="next-steps"></a>Pasos siguientes
[Cómo depurar el inicio de sesión único basado en SAML en aplicaciones de Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-saml-debugging)
