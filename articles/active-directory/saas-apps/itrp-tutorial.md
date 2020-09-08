---
title: 'Tutorial: Integración de Azure Active Directory con ITRP | Microsoft Docs'
description: En este tutorial, aprenderá a configurar el inicio de sesión único entre Azure Active Directory e ITRP.
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
ms.openlocfilehash: d7a7c4f7d9a9ba2ef574519a41aae3dd6e28e4fb
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88552805"
---
# <a name="tutorial-azure-active-directory-integration-with-itrp"></a>Tutorial: Integración de Azure Active Directory con ITRP

En este tutorial, obtendrá información sobre cómo integrar ITRP con Azure Active Directory (Azure AD).
Esta integración ofrece las siguientes ventajas:

* Puede usar Azure AD para controlar quién tiene acceso a ITRP.
* Puede permitir que los usuarios inicien sesión automáticamente en ITRP (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Para más información acerca de la integración de aplicaciones SaaS con Azure AD, consulte [Inicio de sesión único en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con ITRP, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción de ITRP que tenga habilitado el inicio de sesión único.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial configurará y probará el inicio de sesión único de Azure AD en un entorno de prueba.

* ITRP admite el inicio de sesión único iniciado por SP.

## <a name="add-itrp-from-the-gallery"></a>Incorporación de ITRP desde la galería

Para configurar la integración de ITRP en Azure AD, tiene que agregar ITRP desde la galería a la lista de aplicaciones SaaS administradas.

1. En [Azure Portal](https://portal.azure.com), en el panel izquierdo, seleccione **Azure Active Directory**:

    ![Seleccione Azure Active Directory.](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** > **Todas las aplicaciones**:

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una aplicación, seleccione **Nueva aplicación** en la parte superior de la ventana:

    ![Seleccionar Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **ITRP**. Seleccione **ITRP** en los resultados de búsqueda y, a continuación, seleccione **Agregar**.

     ![Search Results](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con ITRP con un usuario de prueba llamado Britta Simon.
Para habilitar el inicio de sesión único, tendrá que establecer una relación entre un usuario de Azure AD y el usuario correspondiente de ITRP.

Para configurar y probar el inicio de sesión único de Azure AD con ITRP, debe hacer lo siguiente:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** para permitir que los usuarios utilicen esta característica.
2. **[Configure el inicio de sesión único en ITRP](#configure-itrp-single-sign-on)** en el lado de la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** para probar el inicio de sesión único de Azure AD.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** para permitir el inicio de sesión único de Azure AD para el usuario.
5. **[Cree un usuario de prueba de ITRP](#create-an-itrp-test-user)** que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** para comprobar que la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con ITRP, haga lo siguiente:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación ITRP, seleccione **Inicio de sesión único**:

    ![Selección de inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccione un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Selección de un método de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, seleccione el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**:

    ![Icono Editar](common/edit-urls.png)

4. En el cuadro de diálogo **Configuración básica de SAML**, siga los pasos que se indican a continuación.

    ![Cuadro de diálogo Configuración básica de SAML](common/sp-identifier.png)

    1. En el cuadro **URL de inicio de sesión**, escriba una dirección URL con el siguiente formato:
    
       `https://<tenant-name>.itrp.com`

    1. En el cuadro **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente formato:

       `https://<tenant-name>.itrp.com`

    > [!NOTE]
    > Estos valores son marcadores de posición. Debe utilizar la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de ITRP](https://www.4me.com/support/) para obtener estos valores. También puede consultar los patrones que se muestran en el cuadro de diálogo **Configuración básica de SAML** de Azure Portal.

5. En la sección **Certificado de firma de SAML**, seleccione el icono **Editar** para abrir el cuadro de diálogo **Certificado de firma de SAML**:

    ![Icono Editar](common/edit-certificate.png)

6. En el cuadro de diálogo **Certificado de firma de SAML**, copie el valor de **Huella digital** y guárdelo:

    ![Copia del valor de huella digital](common/copy-thumbprint.png)

7. En la sección **Configurar ITRP**, copie las direcciones URL adecuadas según sus necesidades:

    ![Copia de las direcciones URL de configuración](common/copy-configuration-urls.png)

    1. **Dirección URL de inicio de sesión**

    1. **Identificador de Azure AD**.

    1. **Dirección URL de cierre de sesión**

### <a name="configure-itrp-single-sign-on"></a>Configuración del inicio de sesión único de ITRP

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la compañía de ITRP.

1. En la parte superior de la ventana, seleccione el icono **Settings** (Configuración):

    ![Icono Configuración](./media/itrp-tutorial/ic775570.png "Icono Configuración")

1. En el panel izquierdo, haga clic en **Single Sign-On** (Inicio de sesión único):

    ![Selección de Inicio de sesión único](./media/itrp-tutorial/ic775571.png "Seleccionar Inicio de sesión único")

1. Siga estos pasos en la sección de configuración de **Single Sign-On** (Inicio de sesión único).

    ![Sección Inicio de sesión único](./media/itrp-tutorial/ic775572.png "Sección Inicio de sesión único")

    ![Sección Inicio de sesión único](./media/itrp-tutorial/ic775573.png "Sección Inicio de sesión único")

    1. Seleccione **Habilitado**.

    1. En el cuadro **Remote Logout URL** (URL de cierre de sesión remota), pegue el valor de la **dirección URL de cierre de sesión** que copió de Azure Portal.

    1. En el cuadro **SAML SSO URL** (URL de inicio de sesión único de SAML), pegue el valor de la **dirección URL de inicio de sesión** que copió de Azure Portal.

    1. En el cuadro de texto **Certificate Fingerprint** (Huella digital de certificado), pegue el valor de **huella digital** del certificado que copió de Azure Portal.

    1. Seleccione **Guardar**.

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a ITRP.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales**, **Todas las aplicaciones**, **ITRP**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **ITRP**.

    ![Lista de aplicaciones](common/all-applications.png)

3. En el panel izquierdo, seleccione **Usuarios y grupos**:

    ![Seleccionar Usuarios y grupos](common/users-groups-blade.png)

4. Seleccione **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Selección de Agregar usuario](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la ventana.

6. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. Haga clic en el botón **Seleccionar** de la parte inferior de la ventana.

7. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

### <a name="create-an-itrp-test-user"></a>Creación de un usuario de prueba de ITRP

Para permitir que los usuarios de Azure AD inicien sesión en ITRP, deberá agregarlos a esta aplicación. Y los debe agregar manualmente.

Para crear una cuenta de usuario, siga estos pasos:

1. Inicie sesión en su inquilino de ITRP.

1. En la parte superior de la ventana, seleccione el icono **Records** (Registros):

    ![Icono Registros](./media/itrp-tutorial/ic775575.png "Icono Registros")

1. En el menú, seleccione **People** (Personas):

    ![Selección de personas](./media/itrp-tutorial/ic775587.png "Selección de personas")

1. Seleccione el signo de más ( **+** ) para agregar a una nueva persona:

    ![Selección del signo más](./media/itrp-tutorial/ic775576.png "Selección del signo más")

1. En el cuadro de diálogo **Add New Person** (Agregar nueva persona), realice los pasos siguientes.

    ![Cuadro de diálogo Agregar nueva persona](./media/itrp-tutorial/ic775577.png "Cuadro de diálogo Agregar nueva persona")

    1. Escriba el nombre y la dirección de correo electrónico de una cuenta de Azure AD válida que quiera agregar.

    1. Seleccione **Guardar**.

> [!NOTE]
> Puede usar cualquier API o herramienta de creación de cuentas de usuario que proporcione ITRP para aprovisionar cuentas de usuario de AAD.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

Ahora, debe probar la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al seleccionar el icono de ITRP en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de ITRP para la que configuró el inicio de sesión único. Para más información acerca del Panel de acceso, consulte [Access and use apps on the My Apps portal](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction) (Uso de aplicaciones y acceso a ellas en el portal Aplicaciones).

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
