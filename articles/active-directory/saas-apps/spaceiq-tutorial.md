---
title: 'Tutorial: Integración de Azure Active Directory con SpaceIQ | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SpaceIQ.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/25/2019
ms.author: jeedes
ms.openlocfilehash: 643d2481c171ae58a9d105d3dd7c53c251c2c41f
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88545036"
---
# <a name="tutorial-azure-active-directory-integration-with-spaceiq"></a>Tutorial: Integración de Azure Active Directory con SpaceIQ

En este tutorial, aprenderá a integrar SpaceIQ con Azure Active Directory (Azure AD).
La integración de SpaceIQ con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a SpaceIQ.
* Puede permitir que los usuarios inicien sesión automáticamente en SpaceIQ (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con SpaceIQ, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Suscripción de SpaceIQ con el inicio de sesión único habilitado

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SpaceIQ admite el inicio de sesión único iniciado por **IDP**

## <a name="adding-spaceiq-from-the-gallery"></a>Incorporación de SpaceIQ desde la galería

Para configurar la integración de SpaceIQ en Azure AD, tendrá que agregar SpaceIQ desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar SpaceIQ desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **SpaceIQ**, seleccione **SpaceIQ** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

    ![SpaceIQ en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con SpaceIQ con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de SpaceIQ.

Para configurar y probar el inicio de sesión único de Azure AD con SpaceIQ, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de SpaceIQ](#configure-spaceiq-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de SpaceIQ](#create-spaceiq-test-user)** : para tener un homólogo de Britta Simon en SpaceIQ que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con SpaceIQ, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **SpaceIQ**, haga clic en **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    ![Información acerca del inicio de sesión único de dominio y direcciones URL de SpaceIQ](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba la dirección URL: `https://api.spaceiq.com`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://api.spaceiq.com/saml/<instanceid>/callback`

    > [!NOTE]
    > Actualice estos valores con la dirección URL de respuesta y el identificador reales, que se explican más adelante en el tutorial.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up SpaceIQ** (Configurar SpaceIQ), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-spaceiq-single-sign-on"></a>Configuración del inicio de sesión único de SpaceIQ

1. Abra una nueva ventana del explorador e inicie sesión en el entorno de SpaceIQ como administrador.

1. Una vez que haya iniciado sesión, haga clic en la pieza de puzzle en la esquina superior derecha, y luego haga clic en **Integraciones**.

    ![Configuración de cuenta](./media/spaceiq-tutorial/setting1.png) 

1. En **All PROVISIONING & SSO** (Todos los aprovisionamientos y SSO), haga clic en el icono de **Azure** para agregar una instancia de Azure como IDP.

    ![Icono de SAML](./media/spaceiq-tutorial/setting2.png)

1. En el cuadro de diálogo **SSO**, realice los pasos siguientes:

    ![Configuración de la autenticación SAML](./media/spaceiq-tutorial/setting3.png)

    a. En el cuadro **SAML Issuer URL** (Dirección URL del emisor de SAML), pegue el valor del **Identificador de Azure AD**  que copió de la ventana de configuración de la aplicación Azure AD.

    b. Copie el valor de **dirección URL de punto de conexión de devolución de llamada SAML (solo lectura)** y péguelo en el cuadro **URL de respuesta** en la sección **Configuración básica de SAML** de Azure Portal.

    c. Copie el valor de **URI de audiencia de SAML (solo lectura)** y péguelo en el cuadro **Identificador** de la sección **Configuración básica de SAML** de Azure Portal.

    d. Abra el certificado descargado en el Bloc de notas, copie el contenido y péguelo en el cuadro **Certificado X.509**.

    e. Haga clic en **Save**(Guardar).

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba `brittasimon@yourcompanydomain.extension`.  
    Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a SpaceIQ.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y, luego, **SpaceIQ**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **SpaceIQ**.

    ![Vínculo a SpaceIQ en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-spaceiq-test-user"></a>Creación de un usuario de prueba de SpaceIQ

En esta sección, creará un usuario llamado Britta Simon en SpaceIQ. Trabaje con el [equipo de soporte técnico de SpaceIQ](mailto:eng@spaceiq.com) para agregar usuarios en la plataforma de SpaceIQ. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de SpaceIQ en el panel de acceso, debería iniciar sesión automáticamente en la versión de SpaceIQ para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

