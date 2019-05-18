---
title: 'Tutorial: Integración de Azure Active Directory con Picturepark | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y Picturepark.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 31c21cd4-9c00-4cad-9538-a13996dc872f
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 04/18/2019
ms.author: jeedes
ms.openlocfilehash: 617c75024b45dab7ff2466b99bfb71c18cdd778a
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230034"
---
# <a name="tutorial-azure-active-directory-integration-with-picturepark"></a>Tutorial: Integración de Azure Active Directory con Picturepark

En este tutorial, obtendrá información sobre cómo integrar Picturepark con Azure Active Directory (Azure AD).
La integración de Picturepark con Azure AD proporciona las siguientes ventajas:

* En Azure AD, puede controlar quién tiene acceso a Picturepark.
* Puede permitir que los usuarios inicien sesión automáticamente en Picturepark (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con Picturepark, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción habilitada para el inicio de sesión único en Picturepark

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Picturepark admite el inicio de sesión único iniciado por **SP**

## <a name="adding-picturepark-from-the-gallery"></a>Incorporación de Picturepark desde la galería

Para configurar la integración de Picturepark en Azure AD, será preciso que agregue Picturepark desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Picturepark desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)**, haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Picturepark**, seleccione **Picturepark** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![Picturepark en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Picturepark con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Picturepark.

Para configurar y probar el inicio de sesión único de Azure AD con Picturepark, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)**: para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Picturepark](#configure-picturepark-single-sign-on)**: para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)**, para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)**, para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Picturepark](#create-picturepark-test-user)**: el objetivo es tener un homólogo de Britta Simon en Picturepark que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)**: para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Picturepark, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Picturepark**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información sobre dominio y direcciones URL de inicio de sesión único de Picturepark](common/sp-identifier.png)

     a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.picturepark.com`

    b. En el cuadro de texto **Identificador (id. de entidad)**, escriba una dirección URL con el siguiente patrón:

    |  |
    |--|
    | `https://<companyname>.current-picturepark.com`|
    | `https://<companyname>.picturepark.com`|
    | `https://<companyname>.next-picturepark.com`|
    | |

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de cliente de Picturepark](https://picturepark.com/about/contact/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la sección **Certificado de firma de SAML**, haga clic en el botón **Editar** para abrir el cuadro de diálogo **Certificado de firma de SAML**.

    ![Edición del certificado de firma de SAML](common/edit-certificate.png)

6. En la sección **Certificado de firma de SAML**, copie el valor de **Huella digital** en el equipo.

    ![Copia del valor de la huella digital](common/copy-thumbprint.png)

7. En la sección **Configurar Picturepark**, copie las direcciones URL que necesite. Para **Dirección URL de inicio de sesión**, use un valor con el siguiente patrón: `https://login.microsoftonline.com/_my_directory_id_/wsfed`

    > [!Note]
    > _my_directory_id_ es el identificador de inquilino de la suscripción de Azure AD.

    ![Copiar direcciones URL de configuración](./media/picturepark-tutorial/configurls.png)

     a. Identificador de Azure AD

    b. URL de cierre de sesión

### <a name="configure-picturepark-single-sign-on"></a>Configuración del inicio de sesión único de Picturepark

1. En otra ventana del explorador web, inicie sesión en el sitio de la compañía Picturepark como administrador.

2. En la barra de herramientas de la parte superior, haga clic en **Herramientas administrativas** y en **Consola de administración**.
   
    ![Consola de administración](./media/picturepark-tutorial/ic795062.png "Consola de administración")

3. Haga clic en **Autenticación** y en **Proveedores de identidades**.
   
    ![Autenticación](./media/picturepark-tutorial/ic795063.png "Autenticación")

4. En la sección de **Configuración del proveedor de identidades** , realice los pasos siguientes:
   
    ![Configuración del proveedor de identidades](./media/picturepark-tutorial/ic795064.png "Configuración del proveedor de identidades")
   
     a. Haga clic en **Agregar**.
  
    b. Escriba un nombre para su configuración.
   
    c. Seleccione **Establecer como predeterminado**.
   
    d. En el cuadro de texto **Issuer URI**, (Identificador URI del emisor) pegue el valor de **Dirección URL de inicio de sesión**, que ha copiado de Azure Portal.
   
    e. En el cuadro de texto **Trusted Issuer Thumb Print** (Huella digital del emisor de confianza), pegue el valor de **huella digital** que copió de la sección **Certificado de firma SAML**. 

5. Haga clic en **JoinDefaultUsersGroup**.

6. Para establecer el atributo **Emailaddress** en el cuadro de texto **Notificación**, escriba `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` y haga clic en **Guardar**.

      ![Configuración](./media/picturepark-tutorial/ic795065.png "Configuración")

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

    d. Haga clic en **Create**(Crear).

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Picturepark.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y luego **Picturepark**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Picturepark**.

    ![Vínculo a Picturepark en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-picturepark-test-user"></a>Creación de un usuario de prueba de Picturepark

Para permitir que los usuarios de Azure AD inicien sesión en Picturepark, deben aprovisionarse en Picturepark. En el caso de Picturepark, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en el inquilino de **Picturepark**.

1. En la barra de herramientas de la parte superior, haga clic en **Herramientas administrativas** y en **Usuarios**.
   
    ![Usuarios](./media/picturepark-tutorial/ic795067.png "Usuarios")

1. En la pestaña de **información general de los usuarios**, haga clic **Nuevo**.
   
    ![Administración de usuarios](./media/picturepark-tutorial/ic795068.png "Administración de usuarios")

1. En el cuadro de diálogo **Crear usuario**, siga estos pasos para un usuario válido de Azure Active Directory que desea aprovisionar:
   
    ![Creación de usuarios](./media/picturepark-tutorial/ic795069.png "Creación de usuarios")
   
     a. En el cuadro de texto **Dirección de correo electrónico**, escriba la **dirección de correo electrónico** del usuario `BrittaSimon@contoso.com`.  
   
    b. En los cuadros de texto **Contraseña** y **Confirmar contraseña**, escriba la **contraseña** de BrittaSimon. 
   
    c. En el cuadro de texto **Nombre**, escriba el **nombre** de la usuaria **Britta**. 
   
    d. En el cuadro de texto **Apellido**, escriba el **apellido** de la usuaria **Simon**.
   
    e. En el cuadro de texto **Empresa**, escriba el **nombre de la empresa** del usuario. 
   
    f. En el cuadro de texto **País**, seleccione el **país o región** del usuario.
  
    g. En el cuadro de texto **ZIP** (Código postal), escriba el **código postal** de la ciudad.
   
    h. En el cuadro de texto **Ciudad**, escriba el **nombre de la ciudad** del usuario.

    i. Seleccione un valor en **Language**(Idioma).
   
    j. Haga clic en **Create**(Crear).

>[!NOTE]
>Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de Picturepark ofrecida por Picturepark para aprovisionar cuentas de usuario de Azure AD.
>

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Picturepark en el panel de acceso, se iniciará sesión automáticamente en la versión de Picturepark para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

