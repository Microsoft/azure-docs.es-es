---
title: 'Tutorial: Integración de Azure Active Directory con webMethods Integration Suite | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y webMethods Integration Suite.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 97261535-7a2d-4d73-94c8-38116b8a776e
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 02/15/2019
ms.author: jeedes
ms.openlocfilehash: 8dead16c60a26e2fc53953ed65337195c3b2aa67
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2019
ms.locfileid: "65470805"
---
# <a name="tutorial-azure-active-directory-integration-with-webmethods-integration-suite"></a>Tutorial: Integración de Azure Active Directory con webMethods Integration Suite

En este tutorial, aprenderá a integrar webMethods Integration Suite con Azure Active Directory (Azure AD).
La integración de webMethods Integration Suite con Azure AD ofrece las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a webMethods Integration Suite.
* Puede permitir que los usuarios inicien sesión automáticamente en webMethods Integration Suite (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con webMethods Integration Suite se necesitan los elementos siguientes:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de evaluación de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/).
* Una suscripción habilitada para el inicio de sesión único en webMethods Integration Suite

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* webMethods Integration Suite admite el inicio de sesión único iniciado por **SP** e **IDP**

* webMethods Integration Suite admite el aprovisionamiento de usuarios **Just-In-Time**

## <a name="adding-webmethods-integration-suite-from-the-gallery"></a>Incorporación de webMethods Integration Suite desde la galería

Para configurar la integración de webMethods Integration Suite en Azure AD, es preciso agregar webMethods Integration Suite desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar webMethods Integration Suite desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)**, haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **webMethods Integration Suite**, seleccione **webMethods Integration Suite** en el panel de resultados y haga clic en **Agregar** para agregar la aplicación.

     ![webMethods Integration Suite en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con webMethods Integration Suite con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de webMethods Integration Suite.

Para configurar y probar el inicio de sesión único de Azure AD con webMethods Integration Suite, es necesario completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)**: para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único en webMethods Integration Suite](#configure-webmethods-integration-suite-single-sign-on)**: para configurar el inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)**, para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)**, para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba en webMethods Integration Suite](#create-webmethods-integration-suite-test-user)**: para tener un homólogo de Britta Simon en webMethods Integration Suite que esté vinculado a la representación de ella en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)**: para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con webMethods Integration Suite, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **webMethods Integration Suite**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. Para configurar **webMethods Integration Cloud**, en la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de webMethods Integration Suite](common/idp-intiated.png)

     a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente:

    | |
    |--|
    | `<SUBDOMAIN>.webmethodscloud.com` |
    | `<SUBDOMAIN>.webmethodscloud.eu` |
    | `<SUBDOMAIN>.webmethodscloud.de` |

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón:

    | |
    |--|
    | `https://<SUBDOMAIN>.webmethodscloud.com/integration/live/saml/ssoResponse` |
    | `https://<SUBDOMAIN>.webmethodscloud.eu/integration/live/saml/ssoResponse` |
    | `https://<SUBDOMAIN>.webmethodscloud.de/integration/live/saml/ssoResponse` |

    c. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    ![Información de dominio y direcciones URL de inicio de sesión único de webMethods Integration Suite](common/metadata-upload-additional-signon.png)

    d. En el cuadro de texto **Dirección URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón:

    | |
    |--|
    | `https://<SUBDOMAIN>.webmethodscloud.com/integration/live/saml/ssoRequest` |
    | `https://<SUBDOMAIN>.webmethodscloud.eu/integration/live/saml/ssoRequest` |
    | `https://<SUBDOMAIN>.webmethodscloud.de/integration/live/saml/ssoRequest` |

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de webMethods Integration Suite](https://empower.softwareag.com/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. Para configurar **webMethods API Cloud**, en la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de webMethods Integration Suite](common/idp-intiated.png)

     a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente:

    | |
    |--|
    | `<SUBDOMAIN>.webmethodscloud.com` |
    | `<SUBDOMAIN>.webmethodscloud.eu` |
    | `<SUBDOMAIN>.webmethodscloud.de` |

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón:

    | |
    |--|
    | `https://<SUBDOMAIN>.webmethodscloud.com/umc/rest/saml/initsso` |
    | `https://<SUBDOMAIN>.webmethodscloud.eu/umc/rest/saml/initsso` |
    | `https://<SUBDOMAIN>.webmethodscloud.de/umc/rest/saml/initsso` |

    c. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    ![Información de dominio y direcciones URL de inicio de sesión único de webMethods Integration Suite](common/metadata-upload-additional-signon.png)

    d. En el cuadro de texto **Dirección URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón:

    | |
    |--|
    | `https://api.webmethodscloud.com/umc/rest/saml/initsso/?tenant=<TENANTID>` |
    | `https://api.webmethodscloud.eu/umc/rest/saml/initsso/?tenant=<TENANTID>` |
    | `https://api.webmethodscloud.de/umc/rest/saml/initsso/?tenant=<TENANTID>` |

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de webMethods Integration Suite](https://empower.softwareag.com/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

7. En la sección **Set up webMethods Integration Suite** (Configurar webMethods Integration Suite), copie las direcciones URL adecuadas en función de sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-webmethods-integration-suite-single-sign-on"></a>Configuración del inicio de sesión único en webMethods Integration Suite

Para configurar el inicio de sesión único en **webMethods Integration Suite**, es preciso enviar el **XML de metadatos de federación** descargado y las direcciones URL correspondientes copiadas de Azure Portal al [equipo de soporte técnico de webMethods Integration Suite](https://empower.softwareag.com/). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

     a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba **brittasimon\@yourcompanydomain.extension**  
    Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Create**(Crear).

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, concederá acceso a Britta Simon a webMethods Integration Suite para que use el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **webMethods Integration Suite**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **webMethods Integration Suite**.

    ![Vínculo de webMethods Integration Suite en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-webmethods-integration-suite-test-user"></a>Creación de un usuario de prueba en webMethods Integration Suite

En esta sección, se crea un usuario llamado Britta Simon en webMethods Integration Suite. webMethods Integration Suite admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe en webMethods Integration Suite, se crea después de la autenticación.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de webMethods Integration Suite en el Panel de acceso, debería iniciar sesión automáticamente en la versión de webMethods Integration Suite para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

