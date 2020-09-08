---
title: 'Tutorial: Integración de Azure Active Directory con AnswerHub | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y AnswerHub.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/17/2019
ms.author: jeedes
ms.openlocfilehash: d453cc5300dc658e4b33bb0591100deae5f34aef
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88544747"
---
# <a name="tutorial-azure-active-directory-integration-with-answerhub"></a>Tutorial: Integración de Azure Active Directory con AnswerHub

En este tutorial, obtendrá información sobre cómo integrar AnswerHub con Azure Active Directory (Azure AD).
La integración de AnswerHub con Azure AD proporciona estas ventajas:

* Puede usar Azure AD para controlar quién tiene acceso a AnswerHub.
* Puede permitir que los usuarios inicien sesión automáticamente en AnswerHub con sus cuentas de Azure AD (inicio de sesión único).
* Puede administrar sus cuentas desde una ubicación central: Azure Portal.

Para más información acerca de la integración de aplicaciones SaaS con Azure AD, consulte [Inicio de sesión único en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con AnswerHub, se necesitan lo siguiente:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede empezar por una [versión de evaluación durante un mes](https://azure.microsoft.com/pricing/free-trial/).
* Una suscripción de AnswerHub que tenga el inicio de sesión único habilitado.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* AnswerHub admite el inicio de sesión único iniciado por SP.

## <a name="add-answerhub-from-the-gallery"></a>Incorporación de AnswerHub desde la galería

Para configurar la integración de AnswerHub en Azure AD, será preciso que agregue AnswerHub desde la galería a sus aplicaciones SaaS administradas.

**Incorporación de AnswerHub desde la galería:**

1. En el panel izquierdo de [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una aplicación, en la parte superior de la ventana, seleccione **Nueva aplicación**.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **AnswerHub**. En la lista de resultados, seleccione **AnswerHub**  y, a continuación, seleccione **Agregar**.

     ![AnswerHub en la lista de resultados](common/search-new-app.png)

## <a name="set-up-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con AnswerHub con un usuario de prueba llamado Britta Simon.
Para el inicio de sesión único, tendrá que establecer un vínculo entre un usuario de Azure AD y el usuario correspondiente de AnswerHub.

Para configurar y probar el inicio de sesión único de Azure AD con AnswerHub, es preciso completar estas tareas:

1. [Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on), para permitir que los usuarios utilicen esta característica.
2. [Configuración del inicio de sesión único de AnswerHub](#configure-answerhub-single-sign-on): para configurar los valores de inicio de sesión único en la aplicación.
3. [Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user) llamado Britta Simon.
4. [Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user), para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. Creación de un usuario de prueba de AnswerHub que se corresponde y se vincula con un usuario de prueba de Azure AD.
6. [Prueba del inicio de sesión único](#test-single-sign-on) para comprobar que la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, configura el inicio de sesión único de Azure AD en Azure Portal.

**Para configurar el inicio de sesión único de Azure AD con AnswerHub:**

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **AnswerHub**, seleccione **Inicio de sesión único**.

    ![Botón de inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Cuadro de diálogo Seleccionar un método de inicio de sesión único](common/select-saml-option.png)

3. En la página **Set-up Single Sign-On with SAML** (Configurar inicio de sesión único con SAML), seleccione el icono Edit (Editar) para abrir el cuadro de diálogo **Basic SAML Configuration** (Configuración básica de SAML).

    ![Página Configurar el inicio de sesión único con SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Sección Configuración básica de SAML](common/sp-identifier.png)

    a. En el cuadro **URL de inicio de sesión**, escriba una dirección URL con el siguiente formato: `https://<company>.answerhub.com`

    b. En el cuadro **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente formato: `https://<company>.answerhub.com`

    > [!NOTE]
    > Estos valores no son reales. Debe actualizarlos con la dirección URL de inicio de sesión y el identificador reales. Póngase en contacto con el [equipo de soporte técnico de AnswerHub](mailto:success@answerhub.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione el vínculo **Descargar** situado junto a **Certificado (Base64)** , según sus requisitos, y guarde el certificado en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up AnswerHub** (Configurar AnswerHub), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

   Puede copiar estas direcciones URL:
    - URL de inicio de sesión

    - Identificador de Azure AD

    - URL de cierre de sesión

### <a name="configure-answerhub-single-sign-on"></a>Configuración del inicio de sesión único de AnswerHub

En esta sección puede configurar el inicio de sesión único de AnswerHub.  

**Para configurar el inicio de sesión único de AnswerHub:**

1. En otra ventana del explorador web, inicie sesión en su sitio de la compañía AnswerHub como administrador.

    > [!NOTE]
    > Si necesita ayuda para configurar AnswerHub, póngase en contacto con el [equipo de soporte técnico de AnswerHub](mailto:success@answerhub.com.).

2. Vaya a **Administración**.

3. En la pestaña **Usuario y grupos**, en el panel de la izquierda, en la sección **Social Settings** (Configuración social), seleccione **SAML Setup** (Configuración de SAML).

4. En la pestaña **IDP Config**, complete estas tareas:

    ![Pestaña Users & Groups (Usuarios y grupos)](./media/answerhub-tutorial/ic785172.png "Configuración de SAML")  
  
    a. En el cuadro **IDP Login URL** (URL de inicio de sesión de IDP), pegue la **dirección URL de inicio de sesión** que copió de Azure Portal.
  
    b. En el cuadro **IDP Logout URL** (URL de cierre de sesión de IDP), pegue la **dirección URL de cierre de sesión** que copió de Azure Portal.

    c. En el cuadro de texto **IDP Name Identifier Format** (Formato del identificador de nombre de IDP), escriba el valor de **identificador** que seleccionó en Azure Portal en la sección **Atributos de usuario**.
  
    d. Seleccione **Claves y certificados**.

5. En la sección **Claves y certificados** sección, siga estos pasos:

    ![Sección Keys and Certificates (Claves y certificados)](./media/answerhub-tutorial/ic785173.png "Claves y certificados")  

    a. En el Bloc de notas, abra el certificado codificado en Base 64 que descargó de Azure Portal, copie el contenido y, a continuación, péguelo en el cuadro **IDP Public Key (x509 Format)** [Clave pública de IDP (formato x509)].
  
    b. Seleccione **Guardar**.

6. En la pestaña **IDP Config** (Configuración de IDP), seleccione **Save** (Guardar) de nuevo.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, creará un usuario de prueba llamado Britta Simon en Azure Portal.

**Creación de un usuario de prueba de Azure AD:**

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Seleccione Azure Active Directory, Usuarios, Todos los usuarios.](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades de usuario, realice estos pasos.

    ![Propiedades de usuario](common/user-properties.png)

    a. En el cuadro **Nombre**, escriba **BrittaSimon**.
  
    b. En el cuadro **Nombre de usuario**, escriba **brittasimon\@<yourcompanydomain.extension>** .  
    Por ejemplo, BrittaSimon@contoso.com.

    c. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.

    d. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a configurar al usuario Britta Simon para que use el inicio de sesión único de Azure AD concediéndole acceso a AnswerHub.

**Asignación del usuario de prueba de Azure AD:**

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **AnswerHub**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **AnswerHub**.

    ![Lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Seleccionar Usuarios y grupos](common/users-groups-blade.png)

4. Seleccione **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel "Agregar asignación"](common/add-assign-user.png)

5. En el cuadro de diálogo **Users and Groups** (Usuarios y grupos), seleccione **Britta Simon** en la lista **Users** (Usuarios) y, luego, pulse el botón **Select** (Seleccionar) en la parte inferior de la pantalla.

6. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. 

7. Haga clic en el botón **Seleccionar** situado en la parte inferior de la pantalla.

8. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

### <a name="create-an-answerhub-test-user"></a>Creación de un usuario de prueba de AnswerHub

Para permitir que los usuarios de Azure AD inicien sesión en AnswerHub, deberá agregarlos a este. En AnswerHub, esta tarea se realiza manualmente.

**Para configurar una cuenta de usuario:**

1. Inicie sesión en el sitio de la compañía **AnswerHub** como administrador.

2. Vaya a **Administración**.

3. Seleccione la pestaña **Users & Groups** (Usuarios y grupos).

4. En el panel izquierdo, en la sección **Manage Users** (Administrar usuarios), seleccione **Create or import users** (Crear o importar usuarios) y, a continuación, seleccione **Users & Groups** (Usuarios y grupos).

   ![Pestaña Users & Groups (Usuarios y grupos)](./media/answerhub-tutorial/ic785175.png "Usuarios y grupos")

5. En los cuadros correspondientes, escriba la **dirección de correo electrónico**, el **nombre de usuario** y la **contraseña** de una cuenta de Azure AD válida que desee agregar y seleccione **Save** (Guardar).

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de AnswerHub para configurar cuentas de usuario de Azure AD.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el panel de acceso.

Al seleccionar el icono de AnswerHub en el panel de acceso, debería iniciar sesión automáticamente en la versión de AnswerHub para la que configuró el inicio de sesión único. Para más información sobre el panel de acceso, consulte [Introducción al panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales acerca de la integración de aplicaciones SaaS en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

