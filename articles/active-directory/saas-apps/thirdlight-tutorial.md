---
title: 'Tutorial: Integración de Azure Active Directory con ThirdLight | Microsoft Docs'
description: En este tutorial, aprenderá a configurar el inicio de sesión único entre Azure Active Directory y ThirdLight.
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
ms.openlocfilehash: 3193df0848c0d67935faa0020465e79bd50e2151
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88544985"
---
# <a name="tutorial-azure-active-directory-integration-with-thirdlight"></a>Tutorial: Integración de Azure Active Directory con ThirdLight

En este tutorial, aprenderá a integrar ThirdLight con Azure Active Directory (Azure AD). Esta integración ofrece las siguientes ventajas:

* Puede usar Azure AD para controlar quién tiene acceso a ThirdLight.
* Puede permitir que los usuarios inicien sesión automáticamente en ThirdLight (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea más información acerca de la integración de aplicaciones SaaS con Azure AD, consulte [Inicio de sesión único en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con ThirdLight, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción de ThirdLight que tenga habilitado el inicio de sesión único.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial configurará y probará el inicio de sesión único de Azure AD en un entorno de prueba.

* ThirdLight admite el inicio de sesión único iniciado por SP.

## <a name="add-thirdlight-from-the-gallery"></a>Incorporación de ThirdLight desde la galería

Para configurar la integración de ThirdLight en Azure AD, es preciso agregar ThirdLight desde la galería a la lista de aplicaciones SaaS administradas.

1. En [Azure Portal](https://portal.azure.com), en el panel izquierdo, seleccione **Azure Active Directory**:

    ![Seleccione Azure Active Directory.](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** > **Todas las aplicaciones**:

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una aplicación, seleccione **Nueva aplicación** en la parte superior de la ventana:

    ![Seleccionar Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **ThirdLight**. Seleccione **ThirdLight** en los resultados de la búsqueda y, después, seleccione **Agregar**.

     ![Search Results](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con ThirdLight con un usuario de prueba llamado Britta Simon.
Para habilitar el inicio de sesión único, tendrá que establecer una relación entre un usuario de Azure AD y el usuario correspondiente de ThirdLight.

Para configurar y probar el inicio de sesión único de Azure AD con ThirdLight, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** para permitir que los usuarios utilicen esta característica.
2. **[Configuración del inicio de sesión único en ThirdLight](#configure-thirdlight-single-sign-on)** en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** para probar el inicio de sesión único de Azure AD.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** para permitir el inicio de sesión único de Azure AD para el usuario.
5. **[Creación de un usuario de prueba en ThirdLight](#create-a-thirdlight-test-user)** que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** para comprobar que la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con ThirdLight, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación ThirdLight, seleccione **Inicio de sesión único**:

    ![Selección de inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccione un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Selección de un método de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, seleccione el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**:

    ![Icono Editar](common/edit-urls.png)

4. En el cuadro de diálogo **Configuración básica de SAML**, siga los pasos que se indican a continuación.

    ![Cuadro de diálogo Configuración básica de SAML](common/sp-identifier.png)

    1. En el cuadro **URL de inicio de sesión**, escriba una dirección URL con el siguiente formato:
    
          `https://<subdomain>.thirdlight.com/`

    1. En el cuadro **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente formato:

       `https://<subdomain>.thirdlight.com/saml/sp`

       > [!NOTE]
       > Estos valores son marcadores de posición. Debe utilizar la dirección URL y el identificador reales de inicio de sesión. Para obtener los valores, póngase en contacto con el [equipo de soporte técnico de ThirdLight](https://www.thirdlight.com/support). También puede consultar los patrones que se muestran en el cuadro de diálogo **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione el vínculo **Descargar** situado junto a **XML de metadatos de federación**, según sus requisitos, y guarde el archivo en el equipo:

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up ThirdLight** (Configurar ThirdLight), copie las direcciones URL adecuadas según sus necesidades:

    ![Copia de las direcciones URL de configuración](common/copy-configuration-urls.png)

    1. **Dirección URL de inicio de sesión**

    1. **Identificador de Azure AD**.

    1. **Dirección URL de cierre de sesión**

### <a name="configure-thirdlight-single-sign-on"></a>Configuración del inicio de sesión único en ThirdLight

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la compañía de ThirdLight.

1. Vaya a **Configuration** > **System Administration** > **SAML2** (Configuración > Administración del sistema > SAML2):

    ![System Administration (Administración del sistema)](./media/thirdlight-tutorial/ic805843.png "Administración del sistema")

1. En la sección de configuración de SAML2, siga estos pasos.
  
    ![Sección de configuración de SAML2](./media/thirdlight-tutorial/ic805844.png "Sección de configuración de SAML2")

    1. Seleccione **Habilitar inicio de sesión único de SAML2**.

    1. En **Source for IdP Metadata** (Origen de los metadatos de IdP), seleccione **Load IdP Metadata from XML** (Cargar metadatos de IdP de XML).

    1. Abra el archivo de metadatos que descargó de Azure Portal en la sección anterior. Copie el contenido del archivo y péguelo en el cuadro **IdP Metadata XML** (XML de metadatos de IdP).

    1. Seleccione **Save SAML2 settings** (Guardar configuración de SAML2).

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección creará un usuario de prueba llamado Britta Simon en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**:

    ![Selección de Todos los usuarios](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la ventana:

    ![Selección de Nuevo usuario](common/new-user.png)

3. En el cuadro de diálogo **Usuario**, siga los pasos que se indican a continuación.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    1. En el cuadro **Nombre**, escriba **BrittaSimon**.
  
    1. En el cuadro **Nombre de usuario**, escriba **BrittaSimon@\<yourcompanydomain>.\<extension>** (Por ejemplo, BrittaSimon@contoso.com).

    1. Seleccione **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro **Contraseña**.

    1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure, para lo que le concederá acceso a ThirdLight.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **ThirdLight**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **ThirdLight**.

    ![Lista de aplicaciones](common/all-applications.png)

3. En el panel izquierdo, seleccione **Usuarios y grupos**:

    ![Seleccionar Usuarios y grupos](common/users-groups-blade.png)

4. Seleccione **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Selección de Agregar usuario](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la ventana.

6. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. Haga clic en el botón **Seleccionar** de la parte inferior de la ventana.

7. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

### <a name="create-a-thirdlight-test-user"></a>Creación de un usuario de prueba en ThirdLight

Para que los usuarios de Azure AD puedan iniciar sesión en ThirdLight, es preciso que los agregue a este. Y los debe agregar manualmente.

Para crear una cuenta de usuario, siga estos pasos:

1. Inicie sesión en el sitio de la compañía ThirdLight como administrador.

1. Vaya a la pestaña **Usuarios**.

1. Seleccione **Usuarios y grupos**.

1. Seleccione **Agregar nuevo usuario**.

1. Escriba el nombre de usuario, un nombre o una descripción y la dirección de correo electrónico de una cuenta de Azure AD válida que desee aprovisionar. Elija un valor preestablecido o un grupo de nuevos miembros.

1. Seleccione **Crear**.

> [!NOTE]
> Puede usar cualquier API o herramienta de creación de cuentas de usuario que proporcione ThirdLight para aprovisionar cuentas de usuario de Azure AD.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

Ahora, debe probar la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al seleccionar el icono de ThirdLight en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de ThirdLight para la que configuró el inicio de sesión único. Para más información acerca del Panel de acceso, consulte [Access and use apps on the My Apps portal](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction) (Uso de aplicaciones y acceso a ellas en el portal Aplicaciones).

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
