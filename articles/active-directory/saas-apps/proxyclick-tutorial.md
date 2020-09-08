---
title: 'Tutorial: Integración de Azure Active Directory con Proxyclick | Microsoft Docs'
description: En este tutorial, aprenderá a configurar el inicio de sesión único entre Azure Active Directory y Proxyclick.
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
ms.openlocfilehash: b3974c5e2a46ede6ef3f92d74759f58ed55b8497
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88553459"
---
# <a name="tutorial-azure-active-directory-integration-with-proxyclick"></a>Tutorial: Integración de Azure Active Directory con Proxyclick

En este tutorial aprenderá a integrar Proxyclick con Azure Active Directory (Azure AD).
Esta integración ofrece las siguientes ventajas:

* Puede usar Azure AD para controlar quién tiene acceso a Proxyclick.
* Puede permitir que los usuarios inicien sesión automáticamente en Proxyclick (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Para más información acerca de la integración de aplicaciones SaaS con Azure AD, consulte [Inicio de sesión único en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Proxyclick, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una evaluación gratuita de un mes](https://azure.microsoft.com/pricing/free-trial/).
* Una suscripción de Proxyclick que tenga habilitado el inicio de sesión único.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial configurará y probará el inicio de sesión único de Azure AD en un entorno de prueba.

* Proxyclick admite el inicio de sesión único iniciado por SP e IDP.

## <a name="add-proxyclick-from-the-gallery"></a>Incorporación de Proxyclick desde la galería

Para configurar la integración de Proxyclick en Azure AD, es preciso agregar Proxyclick desde la galería a la lista de aplicaciones SaaS administradas.

1. En [Azure Portal](https://portal.azure.com), en el panel izquierdo, seleccione **Azure Active Directory**:

    ![Seleccione Azure Active Directory.](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** > **Todas las aplicaciones**:

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una aplicación, seleccione **Nueva aplicación** en la parte superior de la ventana:

    ![Seleccionar Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Proxyclick**. Seleccione **Proxyclick** en los resultados de la búsqueda y, después, seleccione **Agregar**.

     ![Search Results](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Proxyclick con un usuario de prueba llamado Britta Simon.
Para habilitar el inicio de sesión único, tendrá que establecer una relación entre un usuario de Azure AD y el usuario correspondiente de Proxyclick.

Para configurar y probar el inicio de sesión único de Azure AD con Proxyclick, es preciso completar los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** para permitir que los usuarios utilicen esta característica.
2. **[Configuración del inicio de sesión único en Proxyclick](#configure-proxyclick-single-sign-on)** en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** para probar el inicio de sesión único de Azure AD.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** para permitir el inicio de sesión único de Azure AD para el usuario.
5. **[Creación de un usuario de prueba en Proxyclick](#create-a-proxyclick-test-user)** vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** para comprobar que la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Proxyclick, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación Proxyclick, seleccione **Inicio de sesión único**:

    ![Selección de inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccione un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Selección de un método de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, seleccione el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**:

    ![Icono Editar](common/edit-urls.png)

4. En el cuadro de diálogo **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por IdP, siga estos pasos.

    ![Cuadro de diálogo Configuración básica de SAML](common/idp-intiated.png)

    1. En el cuadro **Identificador**, escriba una dirección URL con este patrón:
   
       `https://saml.proxyclick.com/init/<companyId>`

    1. En el cuadro **Dirección URL de respuesta**, escriba una dirección URL con este patrón:

       `https://saml.proxyclick.com/consume/<companyId>`

5. Seleccione **Establecer direcciones URL adicionales** si desea configurar la aplicación en el modo iniciado por SP. En el cuadro **URL de inicio de sesión**, escriba una dirección URL con el siguiente formato:
   
   `https://saml.proxyclick.com/init/<companyId>`

    ![Información de inicio de sesión único de Dominio y direcciones URL de Proxyclick](common/metadata-upload-additional-signon.png)

    

    > [!NOTE]
    > Estos valores son marcadores de posición. Debe usar los valores reales de identificador, dirección URL de respuesta o dirección URL de inicio de sesión. Los pasos para obtener estos valores se describen más adelante.

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione el vínculo **Descargar** situado junto a **Certificado (Base64)** , según sus requisitos, y guarde el certificado en el equipo:

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

7. En la sección **Set up Proxyclick** (Configurar Proxyclick), copie las direcciones URL adecuadas en función de sus requisitos:

    ![Copia de las direcciones URL de configuración](common/copy-configuration-urls.png)

    1. **Dirección URL de inicio de sesión**

    1. **Identificador de Azure AD**.

    1. **Dirección URL de cierre de sesión**

### <a name="configure-proxyclick-single-sign-on"></a>Configuración del inicio de sesión único de Proxyclick

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la compañía de Proxyclick.

2. Seleccione **Account & Settings** (Cuenta y configuración):

    ![Seleccionar Account & Settings (Cuenta y configuración)](./media/proxyclick-tutorial/configure1.png)

3. Desplácese hacia abajo hasta la sección **Integrations** (Integraciones) y seleccione **SAML**:

    ![Seleccionar SAML](./media/proxyclick-tutorial/configure2.png)

4. En la sección **SAML**, siga estos pasos.

    ![Sección SAML](./media/proxyclick-tutorial/configure3.png)

    1. Copie el valor de **SAML Consumer URL** (URL de consumidor de SAML) y péguelo en el cuadro **URL de respuesta** del cuadro de diálogo **Configuración básica de SAML** de Azure Portal.

    1. Copie el valor de **SAML SSO Redirect URL** (URL de redireccionamiento de SSO de SAML) y péguelo en los cuadros **URL de inicio de sesión**  e **Identificador** del cuadro de diálogo **Configuración básica de SAML** de Azure Portal.

    1. En la lista **SAML Request Method** (Método de solicitud SAML), seleccione como **HTTP Redirect** (Redireccionamiento de HTTP).

    1. En el cuadro **Issuer** (Emisor), pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    1. En el cuadro **SAML 2.0 Endpoint URL** (URL de punto de conexión de SAML 2.0), pegue el valor de **Dirección URL de inicio de sesión** que copió de Azure Portal.

    1. En el Bloc de notas, abra el archivo de certificado que descargó desde Azure Portal. Pegue el contenido de dicho archivo en el cuadro **Certificate** (Certificado).

    1. Seleccione **Save changes** (Guardar los cambios).

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección creará un usuario de prueba llamado Britta Simon en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**:

    ![Selección de Todos los usuarios](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Selección de Nuevo usuario](common/new-user.png)

3. En el cuadro de diálogo **Usuario**, siga los pasos que se indican a continuación.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    1. En el cuadro **Nombre**, escriba **BrittaSimon**.
  
    1. En el cuadro **Nombre de usuario**, escriba **BrittaSimon@\<yourcompanydomain>.\<extension>** (Por ejemplo, BrittaSimon@contoso.com).

    1. Seleccione **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro **Contraseña**.

    1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure, para lo que le concederá acceso a Proxyclick.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Proxyclick**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Proxyclick**.

    ![Lista de aplicaciones](common/all-applications.png)

3. En el panel izquierdo, seleccione **Usuarios y grupos**:

    ![Seleccionar Usuarios y grupos](common/users-groups-blade.png)

4. Seleccione **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Selección de Agregar usuario](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la ventana.

6. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. Haga clic en el botón **Seleccionar** de la parte inferior de la ventana.

7. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

### <a name="create-a-proxyclick-test-user"></a>Crear un usuario de prueba de Proxyclick

Para que los usuarios de Azure AD puedan iniciar sesión en Proxyclick, es preciso que los agregue a este. Y los debe agregar manualmente.

Para crear una cuenta de usuario, siga estos pasos:

1. Inicie sesión en el sitio de la empresa de Proxyclick como administrador.

1. Seleccione **Colleagues** (Compañeros) en la parte superior de la ventana:

    ![Seleccionar Colleagues (Compañeros)](./media/proxyclick-tutorial/user1.png)

1. Seleccione **Add Colleague** (Agregar compañero):

    ![Seleccionar Add Colleague (Agregar compañero)](./media/proxyclick-tutorial/user2.png)

1. En la sección **Add a colleague** (Agregar un compañero), siga estos pasos.

    ![Sección Add a colleague (Agregar un compañero)](./media/proxyclick-tutorial/user3.png)

    1. En el cuadro **Email** (Correo electrónico), escriba la dirección de correo electrónico del usuario. En este caso, **brittasimon\@contoso.com**.

    1. En el cuadro **First Name** (Nombre), escriba el nombre del usuario. En este caso, **Britta**.

    1. En el cuadro **Last Name** (Apellidos), escriba los apellidos del usuario. En este caso, **Simon**.

    1. Seleccione **Agregar usuario**.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

Ahora, debe probar la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al seleccionar el icono de Proxyclick en el panel de acceso, debería iniciar sesión automáticamente en la instancia de Proxyclick para la que configuró el inicio de sesión único. Para más información acerca del Panel de acceso, consulte [Access and use apps on the My Apps portal](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction) (Uso de aplicaciones y acceso a ellas en el portal Aplicaciones).

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

