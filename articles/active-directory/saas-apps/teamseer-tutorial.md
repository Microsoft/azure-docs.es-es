---
title: 'Tutorial: Integración de Azure Active Directory con TeamSeer | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y TeamSeer.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/07/2019
ms.author: jeedes
ms.openlocfilehash: 815c738f1680c12f94aa97e91cd123237bf1b4f1
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88542508"
---
# <a name="tutorial-azure-active-directory-integration-with-teamseer"></a>Tutorial: Integración de Azure Active Directory con TeamSeer

En este tutorial, aprenderá a integrar TeamSeer con Azure Active Directory (Azure AD).
La integración de TeamSeer con Azure AD proporciona las siguientes ventajas:

* En Azure AD se puede controlar quién tiene acceso a TeamSeer.
* Puede permitir que los usuarios inicien sesión automáticamente en TeamSeer (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con TeamSeer, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción habilitada para el inicio de sesión único en TeamSeer

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* TeamSeer admite SSO iniciado por **SP**

## <a name="adding-teamseer-from-the-gallery"></a>Incorporación de TeamSeer desde la galería

Para configurar la integración de TeamSeer en Azure AD, será preciso que agregue TeamSeer desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar TeamSeer desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **TeamSeer**, seleccione **TeamSeer** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![TeamSeer en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con TeamSeer con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de TeamSeer.

Para configurar y probar el inicio de sesión único de Azure AD con TeamSeer, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de TeamSeer](#configure-teamseer-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de TeamSeer](#create-teamseer-test-user)** : para tener un homólogo de Britta Simon en TeamSeer vinculado a la representación del usuario de Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con TeamSeer, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **TeamSeer**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de TeamSeer](common/sp-signonurl.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://www.teamseer.com/<companyid>`

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico de TeamSeer](https://pages.theaccessgroup.com/solutions_business-suite_absence-management_contact.html) para obtener este valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up TeamSeer** (Configurar TeamSeer), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-teamseer-single-sign-on"></a>Configuración del inicio de sesión único en TeamSeer

1. En otra ventana del explorador web, inicie sesión en su sitio de la compañía de TeamSeer como administrador.

1. Vaya a **Administrador de RR. HH**.

    ![Administrador de RR. HH.](./media/teamseer-tutorial/ic789634.png "Administrador de RR. HH.")

1. Haga clic en **Configuración**.

    ![Configuración](./media/teamseer-tutorial/ic789635.png "Configurar")

1. Haga clic en **Configurar detalles del proveedor SAML**.

    ![Configuración de SAML](./media/teamseer-tutorial/ic789636.png "Configuración de SAML")

1. En la sección de detalles del proveedor SAML, lleve a cabo estos pasos:

    ![Configuración de SAML](./media/teamseer-tutorial/ic789637.png "Configuración de SAML")

    a. En el cuadro de texto **URL**, pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    b. Abra el certificado codificado en base 64 en el bloc de notas, copie su contenido en el portapapeles y péguelo en el cuadro de texto **IdP Public Certificate** (Certificado público del proveedor de identidades).

1. Para completar la configuración del proveedor SAML, lleve a cabo estos pasos:

    ![Configuración de SAML](./media/teamseer-tutorial/ic789638.png "Configuración de SAML")

    a. En las **Direcciones de correo electrónico de prueba**, escriba la dirección de correo electrónico del usuario de prueba.
  
    b. En el cuadro de texto **Emisor** , escriba la dirección URL de emisor del proveedor de servicios.
  
    c. Haga clic en **Save**(Guardar).

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba **brittasimon@yourcompanydomain.extension**  
    Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a TeamSeer.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **TeamSeer**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **TeamSeer**.

    ![Vínculo a TeamSeer en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-teamseer-test-user"></a>Creación de un usuario de prueba en TeamSeer

Para permitir que los usuarios de Azure AD inicien sesión en TeamSeer, deben aprovisionarse en ShiftPlanning. En el caso de TeamSeer, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en su sitio de la compañía de **TeamSeer** como administrador.

1. Vaya a **HR Admin \> Users** (Administración de RR.HH. > Usuarios) y haga clic en **Run the New User wizard** (Ejecutar el Asistente para usuario nuevo).

    ![Administrador de RR. HH.](./media/teamseer-tutorial/ic789640.png "Administrador de RR. HH.")

1. En la sección **Detalles del usuario** , lleve a cabo estos pasos:

    ![User Details (Detalles del usuario)](./media/teamseer-tutorial/ic789641.png "Detalles del usuario")

    a. Escriba el **nombre**, **apellidos** y **nombre de usuario (dirección de correo electrónico)** de una cuenta válida de Azure AD que desee aprovisionar en los cuadros de texto correspondientes.
  
    b. Haga clic en **Next**.

1. Siga las instrucciones en pantalla para agregar un nuevo usuario y haga clic en **Finalizar**.

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de TeamSeer ofrecida por TeamSeer para aprovisionar cuentas de usuario de Azure AD.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de TeamSeer en el Panel de acceso, debería iniciar sesión automáticamente en la versión de TeamSeer para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
