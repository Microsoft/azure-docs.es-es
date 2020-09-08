---
title: 'Tutorial: Integración de Azure Active Directory con Predictix Assortment Planning | Microsoft Docs'
description: Con este tutorial aprenderá a configurar el inicio de sesión único entre Azure Active Directory y Predictix Assortment Planning.
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
ms.openlocfilehash: 5b7ec1843d279e8c0477ed1c7754746d4f3044ad
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88553673"
---
# <a name="tutorial-azure-active-directory-integration-with-predictix-assortment-planning"></a>Tutorial: Integración de Azure Active Directory con Predictix Assortment Planning

En este tutorial aprenderá a integrar Predictix Assortment Planning con Azure Active Directory (Azure AD).
Esta integración ofrece las siguientes ventajas:

* En Azure AD puede controlar quién tiene acceso a Predictix Assortment Planning.
* Puede permitir que los usuarios inicien sesión automáticamente en Predictix Assortment Planning (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Para más información acerca de la integración de aplicaciones SaaS con Azure AD, consulte [Inicio de sesión único en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con Predictix Assortment Planning, necesita:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/).
* Una suscripción habilitada para el inicio de sesión único en Predictix Assortment Planning.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial configurará y probará el inicio de sesión único de Azure AD en un entorno de prueba.

* Predictix Assortment Planning admite el inicio de sesión único iniciado por SP.

## <a name="add-predictix-assortment-planning-from-the-gallery"></a>Adición de Predictix Assortment Planning desde la galería

Para configurar la integración de Predictix Assortment Planning en Azure AD, deberá agregar la aplicación desde la galería a la lista de aplicaciones SaaS administradas.

1. En [Azure Portal](https://portal.azure.com), en el panel izquierdo, seleccione **Azure Active Directory**:

    ![Seleccione Azure Active Directory.](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** > **Todas las aplicaciones**:

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una aplicación, seleccione **Nueva aplicación** en la parte superior de la ventana:

    ![Seleccionar Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Predictix Assortment Planning**. Seleccione **Predictix Assortment Planning** en los resultados de búsqueda y, después, **Agregar**.

     ![Search Results](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección podrá configurar y probar el inicio de sesión único de Azure AD con Predictix Assortment Planning con un usuario de prueba llamado Britta Simon.
Para habilitar el inicio de sesión único, tendrá que establecer una relación entre un usuario de Azure AD y el usuario correspondiente de Predictix Assortment Planning.

Para configurar y probar el inicio de sesión único de Azure AD con Predictix Assortment Planning, es preciso completar los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** para permitir que los usuarios utilicen esta característica.
2. **[Configuración del inicio de sesión único en Predictix Assortment Planning](#configure-predictix-assortment-planning-single-sign-on)** en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** para probar el inicio de sesión único de Azure AD.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** para permitir el inicio de sesión único de Azure AD para el usuario.
5. **[Creación de un usuario de prueba en Predictix Assortment Planning](#create-a-predictix-assortment-planning-test-user)** vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** para comprobar que la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Predictix Assortment Planning, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Predictix Assortment Planning**, seleccione **Inicio de sesión único**:

    ![Seleccionar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccione un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Selección de un método de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, seleccione el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**:

    ![Icono Editar](common/edit-urls.png)

4. En el cuadro de diálogo **Configuración básica de SAML**, siga los pasos que se indican a continuación.

    ![Cuadro de diálogo Configuración básica de SAML](common/sp-identifier.png)

    1. En el cuadro **URL de inicio de sesión**, escriba una dirección URL con el siguiente formato:

        ```https
        https://<sub-domain>.ap.predictix.com/sso/request
        https://<sub-domain>.dev.ap.predictix.com/
        ```

    1. En el cuadro **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente formato:

        ```https
        https://<sub-domain>.ap.predictix.com
        https://<sub-domain>.dev.ap.predictix.com
        ```

    > [!NOTE]
    > Estos valores son marcadores de posición. Debe utilizar la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de Predictix Assortment Planning](https://www.infor.com/support) para obtener estos valores. También puede consultar los patrones que se muestran en el cuadro de diálogo **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione el vínculo **Descargar** situado junto a **Certificado (Base64)** , según sus requisitos, y guarde el certificado en el equipo:

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Predictix Assortment Planning** (Configurar Predictix Assortment Planning), copie las direcciones URL que necesite:

    ![Copia de las direcciones URL de configuración](common/copy-configuration-urls.png)

    1. **Dirección URL de inicio de sesión**

    1. **Identificador de Azure AD**.

    1. **Dirección URL de cierre de sesión**

### <a name="configure-predictix-assortment-planning-single-sign-on"></a>Configuración del inicio de sesión único en Predictix Assortment Planning

Para configurar el inicio de sesión único en Predictix Assortment Planning, es preciso enviar el certificado descargado y las direcciones URL copiadas de Azure Portal al [equipo de soporte técnico de Predictix Assortment Planning](https://www.infor.com/support). El equipo se asegurará de que la conexión de inicio de sesión único de SAML esté establecida correctamente en ambos lados.

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

En esta sección permitirá que Britta Simon use el inicio de sesión único de Azure AD concediéndole acceso a Predictix Assortment Planning.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Predictix Assortment Planning**.

    ![Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Predictix Assortment Planning**.

    ![Lista de aplicaciones](common/all-applications.png)

3. En el panel izquierdo, seleccione **Usuarios y grupos**:

    ![Seleccionar Usuarios y grupos](common/users-groups-blade.png)

4. Seleccione **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Selección de Agregar usuario](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. Haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

### <a name="create-a-predictix-assortment-planning-test-user"></a>Creación de un usuario de prueba de Predictix Assortment Planning

A continuación debe crear un usuario llamado Britta Simon en Predictix Assortment Planning. Póngase en contacto con el [equipo de soporte técnico de Predictix Assortment Planning](https://www.infor.com/support) para agregar usuarios. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

> [!NOTE]
> El titular de la cuenta de Azure AD recibirá un mensaje de correo electrónico y seleccionará un vínculo para confirmar la cuenta antes de que se active.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

Ahora, debe probar la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al seleccionar el icono de Predictix Assortment Planning en el panel de acceso, debería iniciar sesión automáticamente en la instancia de Predictix Assortment Planning para la que configurara el inicio de sesión único. Para más información, consulte [Acceso y uso del aplicaciones en el portal Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)