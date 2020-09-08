---
title: 'Tutorial: Integración de Azure Active Directory con HighGear | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y HighGear.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/16/2019
ms.author: jeedes
ms.openlocfilehash: af2630b977c1ce34712a582175674277799f105b
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88550626"
---
# <a name="tutorial-azure-active-directory-integration-with-highgear"></a>Tutorial: Integración de Azure Active Directory con HighGear

En este tutorial, aprenderá a integrar HighGear con Azure Active Directory (Azure AD).
La integración de HighGear con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a HighGear.
* Puede permitir que los usuarios inicien sesión automáticamente en HighGear (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con HighGear, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Un sistema HighGear con una licencia Enterprise o Unlimited

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* HighGear admite SSO iniciado por **SP e IDP**

## <a name="adding-highgear-from-the-gallery"></a>Adición de HighGear desde la galería

Para configurar la integración de HighGear en Azure AD, será preciso que agregue HighGear desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar HighGear desde la galería, realice los pasos siguientes:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Haga clic en el botón **Nueva aplicación** en la parte superior del cuadro de diálogo para agregar la nueva aplicación.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **HighGear**, seleccione **HighGear** en el panel de resultados y, a continuación, haga clic en el botón **Agregar** para agregar la aplicación.

     ![HighGear en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con el sistema HighGear con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado del sistema HighGear.

Para configurar y probar el inicio de sesión único de Azure AD con el sistema HighGear, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de HighGear](#configure-highgear-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación HighGear.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba en HighGear](#create-highgear-test-user)** : para tener un homólogo de Britta Simon en HighGear que esté vinculado a su representación en Azure AD. 
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con el sistema HighGear, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **HighGear**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Set-up Single Sign-On with SAML** (Configurar inicio de sesión único con SAML), haga clic en el icono **Edit** (Editar) para abrir el cuadro de diálogo **Basic SAML Configuration** (Configuración básica de SAML).

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de HighGear](common/idp-intiated.png)

    a. En el cuadro de texto **Identifier** (Identificador), pegue el valor del campo **Service Provider Entity ID** (Id. de entidad de servicio de proveedor) que se encuentra en la página de configuración del inicio de sesión único en el sistema HighGear.

    ![Campo del identificador de entidad del proveedor de servicio](media/highgear-tutorial/service-provider-entity-id-field.png)
    
    > [!NOTE]
    > Deberá iniciar sesión en el sistema HighGear para acceder a la página de configuración de inicio de sesión único. Cuando haya iniciado sesión, pase el cursor sobre la pestaña Administration (Administración) en HighGear y haga clic en el elemento de menú Single Sign-On Settings (Configuración del inicio de sesión único).
    
    ![Elemento de menú Single Sign-On Settings (Configuración del inicio de sesión único)](media/highgear-tutorial/single-sign-on-settings-menu-item.png)

    b. En el cuadro de texto **Reply URL** (Dirección URL de respuesta), pegue el valor de la **dirección URL del servicio de consumidor de aserciones (ACS)** de la página de configuración del inicio de sesión único del sistema HighGear.

    ![Campo de la dirección URL del servicio de consumidor de aserciones (ACS)](media/highgear-tutorial/assertion-consumer-service-url-field.png)

    c. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

     ![Información de dominio y direcciones URL de inicio de sesión único de HighGear](common/metadata-upload-additional-signon.png)

     En el cuadro de texto **Sign-on URL** (URL de inicio de sesión), pegue el valor del campo **Service Provider Entity ID** (Id. de entidad de servicio de proveedor) que se encuentra en la página de configuración del inicio de sesión único en el sistema HighGear. (Este identificador de entidad también es la dirección URL base del sistema HighGear que se va a usar para el inicio de sesión iniciado por SP).

    ![Campo del identificador de entidad del proveedor de servicio](media/highgear-tutorial/service-provider-entity-id-field.png)

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador real, la dirección URL de respuesta y la dirección URL de inicio de sesión desde la página **Single Sign-On Settings** (Configuración del inicio de sesión único) del sistema HighGear. Si necesita ayuda, póngase en contacto con el [equipo de soporte técnico de HighGear](mailto:support@highgear.com).

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** y guárdelo en su equipo. Lo necesitará en un paso posterior de la configuración del inicio de sesión único.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up HighGear** (Configuración de HighGear), tenga en cuenta la ubicación de las siguientes direcciones URL.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión. Necesitará este valor en el paso 2 en **Configuración del inicio de sesión único de HighGear** más abajo.

    b. Identificador de Azure AD. Necesitará este valor en el paso 3 en **Configuración del inicio de sesión único de HighGear** más abajo.

    c. URL de cierre de sesión. Necesitará este valor en el paso 4 en **Configuración del inicio de sesión único de HighGear** más abajo.

### <a name="configure-highgear-single-sign-on"></a>Configuración del inicio de sesión único de HighGear

Para configurar HighGear para el inicio de sesión único, inicie sesión en el sistema HighGear. Cuando haya iniciado sesión, pase el cursor sobre la pestaña Administration (Administración) en HighGear y haga clic en el elemento de menú Single Sign-On Settings (Configuración del inicio de sesión único).

![Elemento de menú Single Sign-On Settings (Configuración del inicio de sesión único)](media/highgear-tutorial/single-sign-on-settings-menu-item.png)

1. En el campo **Identity Provider Name** (Nombre del proveedor de identidades), escriba una breve descripción que aparecerá en el botón de inicio de sesión único de HighGear en la página de inicio de sesión. Por ejemplo: Azure AD

2. En el campo **Single Sign-On (SSO) URL** (URL de inicio de sesión único [SSO]) de HighGear, pegue el valor del campo **Login URL** (Dirección URL de inicio de sesión) que se encuentra en la sección **Set up HighGear** (Configuración de HighGear) en Azure.

3. En el campo **Identity Provider Entity ID** (Id. de entidad del proveedor de identidades) de HighGear, pegue el valor del campo **Azure AD Identifier** (Identificador de Azure AD) que se encuentra en la sección **Set up HighGear** (Configuración de HighGear) en Azure.

4. En el campo **Single Logout (SLO) URL** (URL de cierre de sesión único [SLO]) de HighGear, pegue el valor del campo **Logout URL** (Dirección URL de cierre de sesión) que se encuentra en la sección **Set up HighGear** (Configuración de HighGear) en Azure.

5. Utilice el Bloc de notas para abrir el certificado que ha descargado en la sección **Certificado de firma de SAML** en Azure. Ha descargado el formato **Certificado (Base64)** . Copie el contenido del certificado en el Bloc de notas y péguelo en el campo **Identity Provider Certificate** (Certificado del proveedor de identidades) en HighGear.

6. Envíe un correo electrónico al [equipo de soporte técnico de HighGear](mailto:support@highgear.com) para solicitar el certificado de HighGear. Siga las instrucciones que reciba de ellos para rellenar los campos **HighGear Certificate** (Certificado de HighGear) y **HighGear Certificate Password** (Contraseña del certificado de HighGear).

7. Haga clic en el botón **Save** (Guardar) la configuración de inicio de sesión único de HighGear.

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

    c. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro de contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure, para lo que le concederá acceso a HighGear.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales**, **Todas las aplicaciones**, **HighGear**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **HighGear**.

    ![Vínculo a HighGear en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-highgear-test-user"></a>Creación del usuario de prueba de HighGear

Para crear un usuario de prueba de HighGear para probar la configuración de inicio de sesión único, inicie sesión en el sistema HighGear.

1. Haga clic en el botón **Create New Contact** (Crear nuevo contacto).

    ![Botón Create New Contact (Crear nuevo contacto)](media/highgear-tutorial/create-new-contact-button.png)

    Aparecerá un menú que le permitirá elegir el tipo de contacto que desea crear.

2. Haga clic en el elemento de menú **Individual** para crear un usuario de HighGear.

    Un panel se deslizará a la derecha para que pueda escribir la información del nuevo usuario.  
    ![Formulario New Contact (Nuevo contacto)](media/highgear-tutorial/new-contact-form.png)

3. En el campo **Name** (Nombre), escriba un nombre para el contenedor. Por ejemplo: Britta Simon

4. Haga clic en el menú **More Options** (Más opciones) y seleccione el elemento de menú **Account Info** (Información de cuenta).

    ![Hacer clic en el elemento de menú Account Info (Información de cuenta)](media/highgear-tutorial/account-info-menu-item.png)

5. Establezca el campo **Can Log In** (Puede iniciar sesión) en Sí.

    El campo **Enable Single Sign-On** (Habilitar inicio de sesión único) se establecerá automáticamente en Sí también.

6. En elñ campo **Single Sign-On User Id** (Id. de usuario de inicio de sesión único), escriba el identificador del usuario. Por ejemplo: BrittaSimon@contoso.com

    La sección Información de cuenta debería tener un aspecto parecido a este:  
    ![Sección de información de cuenta finalizada](media/highgear-tutorial/finished-account-info-section.png)

7. Para guardar el contacto, haga clic en el botón **Guardar** de la parte inferior de la pantalla.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de HighGear en el panel de acceso y debería iniciar sesión automáticamente en la versión de HighGear para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

