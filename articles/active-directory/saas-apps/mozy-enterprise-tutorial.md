---
title: 'Tutorial: Integración de Azure Active Directory con Mozy Enterprise | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Mozy Enterprise.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/01/2019
ms.author: jeedes
ms.openlocfilehash: c9c4cef07bf1de64bcb14bc4487e0f56d2a40dfc
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88518649"
---
# <a name="tutorial-azure-active-directory-integration-with-mozy-enterprise"></a>Tutorial: Integración de Azure Active Directory con Mozy Enterprise

En este tutorial, aprenderá a integrar Mozy Enterprise con Azure Active Directory (Azure AD).
La integración de Mozy Enterprise con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Mozy Enterprise.
* Puede permitir que los usuarios inicien sesión automáticamente en Mozy Enterprise (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Mozy Enterprise, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en Mozy Enterprise

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Mozy Enterprise admite el inicio de sesión único iniciado por **SP**

## <a name="adding-mozy-enterprise-from-the-gallery"></a>Adición de Mozy Enterprise desde la galería

Para configurar la integración de Mozy Enterprise en Azure AD, es preciso agregar dicha solución desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Mozy Enterprise desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Mozy Enterprise**, seleccione **Mozy Enterprise** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![Mozy Enterprise en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Mozy Enterprise con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Mozy Enterprise.

Para configurar y probar el inicio de sesión único de Azure AD con Mozy Enterprise, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Mozy Enterprise](#configure-mozy-enterprise-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Mozy Enterprise](#create-mozy-enterprise-test-user)** : el objetivo es tener un homólogo de Britta Simon en Mozy Enterprise que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Mozy Enterprise, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Mozy Enterprise**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Mozy Enterprise](common/sp-signonurl.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<tenantname>.Mozyenterprise.com`

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de atención al cliente de Mozy Enterprise](https://support.mozy.com/) para obtener el valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Mozy Enterprise** (Configurar Mozy Enterprise), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-mozy-enterprise-single-sign-on"></a>Configuración de inicio de sesión único de Mozy Enterprise

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la compañía de Mozy Enterprise.

2. En la sección **Configuración**, haga clic en **Directiva de autenticación**.
   
    ![Authentication policy (Directiva de autenticación)](./media/mozy-enterprise-tutorial/ic777314.png "Authentication Policy")

3. En la sección **Authentication Policy** (Directiva de autenticación), realice estos pasos:
   
    ![Authentication policy (Directiva de autenticación)](./media/mozy-enterprise-tutorial/ic777315.png "Authentication Policy")
   
    a. Seleccione **Servicio de directorio** como **Proveedor**.
   
    b. Seleccione **Use LDAP Push**(Usar inserción de LDAP).
   
    c. Haga clic en la pestaña **Autenticación SAML** .
   
    d. Pegue la **URL de inicio de sesión** que ha copiado de Azure Portal en el cuadro de texto **URL de autenticación**.
   
    e. Pegue el valor de **Identificador de Azure AD** que copió de Azure Portal en el cuadro de texto **Punto de conexión de SAML**.
   
    f. Abra el certificado codificado en base 64 descargado en el Bloc de notas, copie su contenido en el Portapapeles y, a continuación, pegue todo el certificado en el cuadro de texto **Certificado SAML**.
   
    g. Seleccione **Habilitar SSO para que los administradores inicien sesión con sus credenciales de red**.
   
    h. Haga clic en **Guardar cambios**.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba **brittasimon\@yourcompanydomain.extension**.  
    Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Mozy Enterprise.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Mozy Enterprise**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Mozy Enterprise**.

    ![Vínculo a Mozy Enterprise en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-mozy-enterprise-test-user"></a>Creación de un usuario de prueba de Mozy Enterprise

Para permitir que los usuarios de Azure AD inicien sesión en Mozy Enterprise, deben aprovisionarse en Mozy Enterprise. En el caso de Mozy Enterprise, el aprovisionamiento es una tarea manual.

>[!NOTE]
>Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de Mozy Enterprise ofrecida por Mozy Enterprise para aprovisionar cuentas de usuario de Azure AD.

**Para aprovisionar cuentas de usuario, realice estos pasos:**

1. Inicie sesión en su inquilino de **Mozy Enterprise** .

2. Haga clic en **Usuarios** y, luego, en **Agregar nuevo usuario**.
   
    ![Usuarios](./media/mozy-enterprise-tutorial/ic777317.png "Usuarios")
   
    >[!NOTE]
    >La opción **Agregar nuevo usuario** solo se muestra si **Mozy** está seleccionado como proveedor en **Directiva de autenticación**. Si la autenticación SAML está configurada, los usuarios se agregan automáticamente al iniciar sesión por primera vez con inicio de sesión único.
    
3. En el cuadro de diálogo Nuevo usuario, realice los pasos siguientes:
   
    ![Incorporación de usuarios](./media/mozy-enterprise-tutorial/ic777318.png "Agregar usuarios")
   
    a. En la lista **Choose a Group** (Elija un grupo), seleccione un grupo.
   
    b. En la lista **Tipo de usuario** , seleccione un tipo.
   
    c. En el cuadro de texto **Username** (Nombre de usuario), escriba el nombre de usuario de Azure AD.
   
    d. En el cuadro de texto **Email** (Correo electrónico), escriba la dirección de correo electrónico del usuario de Azure AD.
   
    e. Seleccione **Send user instruction email**(Enviar correo electrónico al usuario con instrucciones).
   
    f. Haga clic en **Add User(s)** (Agregar usuarios).

     >[!NOTE]
     > Después de crear el usuario, se enviará un correo electrónico al usuario de Azure AD que incluye un vínculo para confirmar la cuenta antes de que se active.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Mozy Enterprise del panel de acceso, debería iniciar sesión automáticamente en la versión de Mozy Enterprise para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

