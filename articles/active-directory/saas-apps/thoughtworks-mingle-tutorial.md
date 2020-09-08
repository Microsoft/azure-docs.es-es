---
title: 'Tutorial: integración de Azure Active Directory con Thoughtworks Mingle | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Thoughtworks Mingle.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: caae4251c8a7c7194b2010f1cba750ef34579934
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88546515"
---
# <a name="tutorial-azure-active-directory-integration-with-thoughtworks-mingle"></a>Tutorial: Integración de Azure Active Directory con Thoughtworks Mingle

En este tutorial, aprenderá a integrar Thoughtworks Mingle con Azure Active Directory (Azure AD).
La integración de Thoughtworks Mingle con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Thoughtworks Mingle.
* Puede permitir que los usuarios inicien sesión automáticamente en Thoughtworks Mingle (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Thoughtworks Mingle, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en Thoughtworks Mingle

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Thoughtworks Mingle admite el inicio de sesión único iniciado por **SP**

## <a name="adding-thoughtworks-mingle-from-the-gallery"></a>Agregación de Thoughtworks Mingle desde la galería

Para configurar la integración de Thoughtworks Mingle en Azure AD, deberá agregar Thoughtworks Mingle desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Thoughtworks Mingle desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Thoughtworks Mingle**, seleccione **Thoughtworks Mingle** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![Thoughtworks Mingle en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Thoughtworks Mingle con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Thoughtworks Mingle.

Para configurar y probar el inicio de sesión único de Azure AD con Thoughtworks Mingle, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único en Thoughtworks Mingle](#configure-thoughtworks-mingle-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Thoughtworks Mingle](#create-thoughtworks-mingle-test-user)** : para tener un homólogo de Britta Simon en Thoughtworks Mingle que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Thoughtworks Mingle, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Thoughtworks Mingle**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Thoughtworks Mingle](common/sp-signonurl.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.mingle.thoughtworks.com`

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico de Thoughtworks Mingle](https://support.thoughtworks.com/hc/categories/201743486-Mingle-Community-Support) para obtener este valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up Thoughtworks Mingle** (Configurar Thoughtworks Mingle), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-thoughtworks-mingle-single-sign-on"></a>Configuración del inicio de sesión único de Thoughtworks Mingle

1. Inicie sesión en su sitio de compañía de **Thoughtworks Mingle** como administrador.

2. Haga clic en la pestaña **Admin** (Administrador) y, luego, en **SSO Config** (Configuración de SSO).
   
    ![Pestaña Administrador](./media/thoughtworks-mingle-tutorial/ic785157.png "Configuración de SSO")

3. En la sección **Configuración de SSO** , lleve a cabo estos pasos:
   
    ![Configuración de SSO](./media/thoughtworks-mingle-tutorial/ic785158.png "Configuración de SSO")
    
    a. Para cargar el archivo de metadatos, haga clic en **Elegir archivo**. 

    b. Haga clic en **Guardar cambios**.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba brittasimon@yourcompanydomain.extension. Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Thoughtworks Mingle.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Thoughtworks Mingle**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Thoughtworks Mingle**.

    ![Vínculo a Thoughtworks Mingle en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-thoughtworks-mingle-test-user"></a>Creación de un usuario de prueba de Thoughtworks Mingle

Para que los usuarios de Azure AD puedan iniciar sesión, deben aprovisionarse en la aplicación Thoughtworks Mingle con sus nombres de usuario de Azure Active Directory. En el caso de Thoughtworks Mingle, el aprovisionamiento es una tarea manual.

**Siga estos pasos para configurar el aprovisionamiento de usuario:**

1. Inicie sesión en su sitio de compañía de Thoughtworks Mingle como administrador.

2. Haga clic en **Perfil**.
   
    ![Su primer proyecto](./media/thoughtworks-mingle-tutorial/ic785160.png "Su primer proyecto")

3. Haga clic en la pestaña **Admin** (Administrador) y, luego, en **Users** (Usuarios).
   
    ![Usuarios](./media/thoughtworks-mingle-tutorial/ic785161.png "Usuarios")

4. Haga clic en **Nuevo usuario**.
   
    ![New User](./media/thoughtworks-mingle-tutorial/ic785162.png "Nuevo usuario") (Usuario nuevo)

5. En la página del cuadro de diálogo **Nuevo usuario** , realice los pasos siguientes:
   
    ![Cuadro de diálogo Nuevo usuario](./media/thoughtworks-mingle-tutorial/ic785163.png "Nuevo usuario")  
 
    a. En los cuadros de texto **Sign-in name** (Nombre de inicio de sesión), **Display name** (Nombre para mostrar), **Choose password** (Elegir contraseña) y **Confirm password** (Confirmar contraseña), escriba la información pertinente de una cuenta de Azure AD válida que desee aprovisionar. 

    b. En **User type** (Tipo de usuario), seleccione **Full user** (Usuario completo).

    c. Haga clic en **Crear este perfil**.

>[!NOTE]
>Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de Thoughtworks Mingle ofrecida por Thoughtworks Mingle para aprovisionar cuentas de usuario de Azure AD.
> 

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Thoughtworks Mingle en el panel de acceso, debería iniciar sesión automáticamente en la versión de Thoughtworks Mingle para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

