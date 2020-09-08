---
title: 'Tutorial: Integración de Azure Active Directory con Jitbit Helpdesk | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Jitbit Helpdesk.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/14/2019
ms.author: jeedes
ms.openlocfilehash: 5d4900713cd8d96180bf74b300a738a8b676421e
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88547161"
---
# <a name="tutorial-azure-active-directory-integration-with-jitbit-helpdesk"></a>Tutorial: Integración de Azure Active Directory con Jitbit Helpdesk

En este tutorial, obtendrá información sobre cómo integrar Jitbit Helpdesk con Azure Active Directory (Azure AD).
La integración de Jitbit Helpdesk con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Jitbit Helpdesk.
* Puede permitir que los usuarios inicien sesión automáticamente en Jitbit Helpdesk (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Jitbit Helpdesk, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en Jitbit Helpdesk

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Jitbit Helpdesk admite el inicio de sesión único iniciado por **SP**

## <a name="adding-jitbit-helpdesk-from-the-gallery"></a>Incorporación de Jitbit Helpdesk desde la Galería

Para configurar la integración de Jitbit Helpdesk en Azure AD, será preciso que agregue Jitbit Helpdesk desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Jitbit Helpdesk desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Jitbit Helpdesk**, seleccione **Jitbit Helpdesk** en el panel de resultados y, a continuación, haga clic en el botón **Agregar** para agregar la aplicación.

     ![Jitbit Helpdesk en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Jitbit Helpdesk con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Jitbit Helpdesk.

Para configurar y probar el inicio de sesión único de Azure AD con Jitbit Helpdesk, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Jitbit Helpdesk](#configure-jitbit-helpdesk-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Jitbit Helpdesk](#create-jitbit-helpdesk-test-user)** : el objetivo es tener un homólogo de Britta Simon en Jitbit Helpdesk que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Jitbit Helpdesk, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Jitbit Helpdesk**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información acerca del inicio de sesión único de dominio y direcciones URL de Jitbit Helpdesk](common/sp-identifier.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón:
    | |
    | ----------------------------------------|
    | `https://<hostname>/helpdesk/User/Login`|
    | `https://<tenant-name>.Jitbit.com`|
    | |
    
    > [!NOTE] 
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico de Jitbit Helpdesk](https://www.jitbit.com/support/) para obtener este valor.

    b. En el cuadro de texto **Identificador (Id. de entidad)** , escriba una dirección URL como sigue: `https://www.jitbit.com/web-helpdesk/`

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Jitbit Helpdesk** (Configuración de Jitbit Helpdesk), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-jitbit-helpdesk-single-sign-on"></a>Configuración del inicio de sesión único en Jitbit Helpdesk

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la compañía de Jitbit Helpdesk.

1. En la barra de herramientas de la parte superior, haga clic en **Administración**.

    ![Administración](./media/jitbit-helpdesk-tutorial/ic777681.png "Administración")

1. Haga clic en **Configuración general**.

    ![Usuarios, compañías y permisos](./media/jitbit-helpdesk-tutorial/ic777680.png "Usuarios, compañías y permisos")

1. En la sección **Configuración de autenticación** , realice estos pasos:

    ![Configuración de la autenticación](./media/jitbit-helpdesk-tutorial/ic777683.png "Configuración de autenticación")

    a. Seleccione **Enable SAML 2.0 single sign on** (Habilitar inicio de sesión único de SAML 2.0) para iniciar sesión mediante inicio de sesión único (SSO) con **OneLogin**.

    b. En el cuadro de texto **EndPoint URL** (Dirección URL de punto de conexión), pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    c. Abra el certificado codificado en **base-64** en el Bloc de notas, copie el contenido del mismo en el Portapapeles y péguelo en el cuadro de texto **Certificado X.509**.

    d. Haga clic en **Guardar cambios**.

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

En esta sección, concederá acceso a Britta Simon a Jitbit Helpdesk para que use el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Jitbit Helpdesk**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Jitbit Helpdesk**.

    ![Vínculo de Jitbit Helpdesk en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-jitbit-helpdesk-test-user"></a>Creación de un usuario de prueba de Jitbit Helpdesk

Para permitir que los usuarios de Azure AD inicien sesión en Jitbit Helpdesk, deben aprovisionarse en Jitbit Helpdesk. En el caso de Jitbit Helpdesk, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en su inquilino de **Jitbit Helpdesk**.

1. En el menú de la parte superior, haga clic en **Administration**(Administración).

    ![Administración](./media/jitbit-helpdesk-tutorial/ic777681.png "Administración")

1. Haga clic en **Usuarios, compañías y permisos**.

    ![Usuarios, compañías y permisos](./media/jitbit-helpdesk-tutorial/ic777682.png "Usuarios, compañías y permisos")

1. Haga clic en **Agregar usuario**.

    ![Agregar usuario](./media/jitbit-helpdesk-tutorial/ic777685.png "Agregar usuario")

1. En la sección Crear, escriba los datos de la cuenta de Azure AD que desee aprovisionar como sigue:

    ![Creación](./media/jitbit-helpdesk-tutorial/ic777686.png "Crear")

   a. En el cuadro de texto **Nombre de usuario**, escriba el nombre de un usuario, por ejemplo, **Britta Simon**.

   b. En el cuadro de texto **Correo electrónico**, escriba el correo electrónico del usuario, en el ejemplo **BrittaSimon@contoso.com** .

   c. En el cuadro de texto **Nombre**, escriba el nombre del usuario, en este caso **Britta**.

   d. En el cuadro de texto **Apellidos**, escriba el apellido del usuario, en este caso **Simon**.

   e. Haga clic en **Crear**.

> [!NOTE]
> Puede usar cualquier herramienta de creación de cuentas de usuario de Jitbit Helpdesk u otras API proporcionadas por Jitbit Helpdesk para aprovisionar cuentas de usuario de Azure AD.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Jitbit Helpdesk en el panel de acceso, debería iniciar sesión automáticamente en la versión de Jitbit Helpdesk para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
