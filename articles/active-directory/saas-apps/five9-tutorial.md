---
title: 'Tutorial: integración de Azure Active Directory con Five9 Plus Adapter (CTI, Contact Center Agents) | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Five9 Plus Adapter (CTI, Contact Center Agents).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/04/2019
ms.author: jeedes
ms.openlocfilehash: 2ab03aecf8226662eeb03a5bb3f8adc85013375d
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88555066"
---
# <a name="tutorial-azure-active-directory-integration-with-five9-plus-adapter-cti-contact-center-agents"></a>Tutorial: integración de Azure Active Directory con Five9 Plus Adapter (CTI, Contact Center Agents)

En este tutorial aprenderá a integrar Five9 Plus Adapter (CTI, Contact Center Agents) con Azure Active Directory (Azure AD).
La integración de Five9 Plus Adapter (CTI, Contact Center Agents) con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Five9 Plus Adapter (CTI, Contact Center Agents).
* Puede permitir que los usuarios inicien sesión automáticamente en Five9 Plus Adapter (CTI, Contact Center Agents) (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Five9 Plus Adapter (CTI, Contact Center Agents), necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único de Five9 Plus Adapter (CTI, Contact Center Agents)

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Five9 Plus Adapter (CTI, Contact Center Agents) admite el inicio de sesión único iniciado por **IDP**

## <a name="adding-five9-plus-adapter-cti-contact-center-agents-from-the-gallery"></a>Agregar Five9 Plus Adapter (CTI, Contact Center Agents) desde la galería

Para configurar la integración de Five9 Plus Adapter (CTI, Contact Center Agents) en Azure AD, deberá agregar Five9 Plus Adapter (CTI, Contact Center Agents) desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Five9 Plus Adapter (CTI, Contact Center Agents) desde la galería, lleve a cabo los siguientes pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Five9 Plus Adapter (CTI, Contact Center Agents)** , seleccione **Five9 Plus Adapter (CTI, Contact Center Agents)** del panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![Five9 Plus Adapter (CTI, Contact Center Agents) en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección puede configurar y probar el inicio de sesión único de Azure AD con Five9 Plus Adapter (CTI, Contact Center Agents) con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Five9 Plus Adapter (CTI, Contact Center Agents).

Para configurar y probar el inicio de sesión único de Azure AD con Five9 Plus Adapter (CTI, Contact Center Agents), es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único en Five9 Plus Adapter (CTI, Contact Center Agents)](#configure-five9-plus-adapter-cti-contact-center-agents-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación del usuario de prueba en Five9 Plus Adapter (CTI, Contact Center Agents)](#create-five9-plus-adapter-cti-contact-center-agents-test-user)** : para disponer de un usuario homólogo de Britta Simon en Five9 Plus Adapter (CTI, Contact Center Agents) vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Five9 Plus Adapter (CTI, Contact Center Agents), siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Five9 Plus Adapter (CTI, Contact Center Agents)** , seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    ![Información sobre dominio y direcciones URL de inicio de sesión único de Five9 Plus Adapter (CTI, Contact Center Agents)](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente:
    
    |    Entorno      |       URL      |
    | :-- | :-- |
    | Para "Five9 Plus Adapter for Microsoft Dynamics CRM" | `https://app.five9.com/appsvcs/saml/metadata/alias/msdc` |
    | Para "Five9 Plus Adapter for Zendesk" | `https://app.five9.com/appsvcs/saml/metadata/alias/zd` |
    | Para "Five9 Plus Adapter for Agent Desktop Toolkit" | `https://app.five9.com/appsvcs/saml/metadata/alias/adt` |

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón:

    |      Entorno     |      URL      |
    | :--                  | :--           |
    | Para "Five9 Plus Adapter for Microsoft Dynamics CRM" | `https://app.five9.com/appsvcs/saml/SSO/alias/msdc` |
    | Para "Five9 Plus Adapter for Zendesk" | `https://app.five9.com/appsvcs/saml/SSO/alias/zd` |
    | Para "Five9 Plus Adapter for Agent Desktop Toolkit" | `https://app.five9.com/appsvcs/saml/SSO/alias/adt` |

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

7. En la sección **Set up Five9 Plus Adapter (CTI, Contact Center Agents)** (Configurar Five9 Plus Adapter [CTI, Contact Center Agents]), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-five9-plus-adapter-cti-contact-center-agents-single-sign-on"></a>Configuración del inicio de sesión único en Five9 Plus Adapter (CTI, Contact Center Agents)

1. Para configurar el inicio de sesión único en **Five9 Plus Adapter (CTI, Contact Center Agents)** , debe enviar el **certificado descargado (Base64)** y las direcciones URL adecuadas que copiara al [equipo de soporte técnico de Five9 Plus Adapter (CTI, Contact Center Agents)](https://www.five9.com/about/contact). Además, para configurar aún más el inicio de sesión único, puede seguir los pasos descritos a continuación en función del adaptador:

    a. Guía del administrador de "Five9 Plus Adapter for Agent Desktop Toolkit": [https://webapps.five9.com/assets/files/for_customers/documentation/integrations/agent-desktop-toolkit/plus-agent-desktop-toolkit-administrators-guide.pdf](https://webapps.five9.com/assets/files/for_customers/documentation/integrations/agent-desktop-toolkit/plus-agent-desktop-toolkit-administrators-guide.pdf)
    
    b. Guía del administrador de "Five9 Plus Adapter for Microsoft Dynamics CRM": [https://webapps.five9.com/assets/files/for_customers/documentation/integrations/microsoft/microsoft-administrators-guide.pdf](https://webapps.five9.com/assets/files/for_customers/documentation/integrations/microsoft/microsoft-administrators-guide.pdf)
    
    c. Guía del administrador de "Five9 Plus Adapter for Zendesk": [https://webapps.five9.com/assets/files/for_customers/documentation/integrations/zendesk/zendesk-plus-administrators-guide.pdf](https://webapps.five9.com/assets/files/for_customers/documentation/integrations/zendesk/zendesk-plus-administrators-guide.pdf)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba `brittasimon@yourcompanydomain.extension`. Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Five9 Plus Adapter (CTI, Contact Center Agents).

1. En Azure Portal, seleccione **Aplicaciones empresariales**, seleccione **Todas las aplicaciones** y **Five9 Plus Adapter (CTI, Contact Center Agents)** .

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Five9 Plus Adapter (CTI, Contact Center Agents)** .

    ![Vínculo a Five9 Plus Adapter (CTI, Contact Center Agents) en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-five9-plus-adapter-cti-contact-center-agents-test-user"></a>Creación del usuario de prueba en Five9 Plus Adapter (CTI, Contact Center Agents)

En esta sección creará un usuario llamado Britta Simon en Five9 Plus Adapter (CTI, Contact Center Agents). Colabore con el [equipo de soporte técnico de Five9 Plus Adapter (CTI, Contact Center Agents)](https://www.five9.com/about/contact) para agregar los usuarios a la plataforma de Five9 Plus Adapter (CTI, Contact Center Agents). Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único. 

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Five9 Plus Adapter (CTI, Contact Center Agents) del panel de acceso, debería iniciar sesión automáticamente en la versión de Five9 Plus Adapter (CTI, Contact Center Agents) para la que configurara el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

