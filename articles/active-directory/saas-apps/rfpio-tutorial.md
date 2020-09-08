---
title: 'Tutorial: Integración de Azure Active Directory con RFPIO | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y RFPIO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/14/2019
ms.author: jeedes
ms.openlocfilehash: 6732857e278285071c9d36c629920ad9e67368f2
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88528571"
---
# <a name="tutorial-azure-active-directory-integration-with-rfpio"></a>Tutorial: Integración de Azure Active Directory con RFPIO

En este tutorial, aprenderá a integrar RFPIO con Azure Active Directory (Azure AD).
La integración de RFPIO con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a RFPIO.
* Puede permitir que los usuarios inicien sesión automáticamente en RFPIO (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con RFPIO, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Suscripción de RFPIO con el inicio de sesión único habilitado

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* RFPIO admite el inicio de sesión único iniciado por **SP e IDP**.

## <a name="adding-rfpio-from-the-gallery"></a>Adición de RFPIO desde la galería

Para configurar la integración de RFPIO en Azure AD, es preciso agregar dicha solución desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar RFPIO desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **RFPIO**, seleccione **RFPIO** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

    ![RFPIO en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con RFPIO con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de RFPIO.

Para configurar y probar el inicio de sesión único de Azure AD con RFPIO, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de RFPIO](#configure-rfpio-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de RFPIO](#create-rfpio-test-user)** , para tener un homólogo de Britta Simon en RFPIO que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con RFPIO, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **RFPIO**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice el siguiente paso:

    ![Información de dominio y direcciones URL de inicio de sesión único de RFPIO](common/idp-identifier.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://www.rfpio.com`

    b. Haga clic en **Establecer direcciones URL adicionales**.

    c. En el cuadro de texto **Estado de la retransmisión**, escriba un valor de cadena. Póngase en contacto con el [equipo de soporte técnico de RFPIO](https://www.rfpio.com/contact/) para obtener este valor.

    ![Información de dominio y direcciones URL de inicio de sesión único de RFPIO](common/idp-preintegrated-relay.png)

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    ![imagen](common/both-preintegrated-signon.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://www.app.rfpio.com`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la dirección URL de inicio de sesión reales. Póngase en contacto con el [equipo de soporte al cliente de RFPIO](https://www.rfpio.com/contact/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

7. En la sección **Set up RFPIO** (Configurar RFPIO), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-rfpio-single-sign-on"></a>Configuración del inicio de sesión único de RFPIO

1. En otra ventana del explorador web, inicie sesión en el sitio web de **RFPIO** como administrador.

1. Haga clic en la lista desplegable de la esquina inferior izquierda.

    ![Configurar inicio de sesión único](./media/rfpio-tutorial/app1.png)

1. Haga clic en **Organization Settings** (Configuración de la organización). 

    ![Configurar inicio de sesión único](./media/rfpio-tutorial/app2.png)

1. Haga clic en **FEATURES & INTEGRATION** (CARACTERÍSTICAS E INTEGRACIÓN).

    ![Configurar inicio de sesión único](./media/rfpio-tutorial/app4.png)

1. En **SAML SSO Configuration** (Configuración del inicio de sesión único de SAML), haga clic en **Edit** (Editar).

    ![Configurar inicio de sesión único](./media/rfpio-tutorial/app3.png)

1. En esta sección, realice las siguientes acciones:

    ![Configurar inicio de sesión único](./media/rfpio-tutorial/app5.png)
    
    a. Copie el contenido del archivo **XML de metadatos descargado** y péguelo en el campo **identity configuration** (Configuración de identidad).

    > [!NOTE]
    >Para copiar el contenido del archivo **XML de metadatos de federación** descargado, use **Notepad++** o un **Editor XML** adecuado.

    b. Haga clic en **Validar**.

    c. Después de hacer clic en **Validate** (Validar), mueva **SAML(Enabled)** (SAML [Habilitado]) a activado.

    d. Haga clic en **Enviar**.

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a RFPIO.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y, a continuación, seleccione **RFPIO**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **RFPIO**.

    ![Vínculo a RFPIO en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-rfpio-test-user"></a>Creación de un usuario de prueba de RFPIO

1. Inicie sesión en el sitio de la compañía de RFPIO como administrador.

1. Haga clic en la lista desplegable de la esquina inferior izquierda.

    ![Configurar inicio de sesión único](./media/rfpio-tutorial/app1.png)

1. Haga clic en **Organization Settings** (Configuración de la organización). 

    ![Configurar inicio de sesión único](./media/rfpio-tutorial/app2.png)

1. Haga clic en **TEAM MEMBERS** (MIEMBROS DE EQUIPO).

    ![Configurar inicio de sesión único](./media/rfpio-tutorial/app6.png)

1. Haga clic en **ADD MEMBERS** (AGREGAR MIEMBROS).

    ![Configurar inicio de sesión único](./media/rfpio-tutorial/app7.png)

1. Vaya a la sección **Add New Members** (Agregar nuevos miembros). Realice las siguientes acciones:

    ![Configurar inicio de sesión único](./media/rfpio-tutorial/app8.png)

    a. Escriba la **dirección de correo electrónico** en el campo **Enter one email per line** (Escribir un correo electrónico por línea).

    b. Seleccione **Role** (Role) de acuerdo con sus requisitos.

    c. Haga clic en **ADD MEMBERS** (AGREGAR MIEMBROS).

    > [!NOTE]
    > El titular de la cuenta de Azure Active Directory recibirá un mensaje de correo y seguirá un vínculo para confirmar su cuenta antes de que se active.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de RFPIO en el panel de acceso, debería iniciar sesión automáticamente en la versión de RFPIO para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

