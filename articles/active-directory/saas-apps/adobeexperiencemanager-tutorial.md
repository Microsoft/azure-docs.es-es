---
title: 'Tutorial: Integración de Azure Active Directory con Adobe Experience Manager | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Adobe Experience Manager.
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
ms.openlocfilehash: 1088827ead320007c758d8e843a15328f108c035
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88543030"
---
# <a name="tutorial-azure-active-directory-integration-with-adobe-experience-manager"></a>Tutorial: Integración de Azure Active Directory con Adobe Experience Manager

En este tutorial, aprenderá a integrar Adobe Experience Manager con Azure Active Directory (Azure AD).
La integración de Adobe Experience Manager con Azure AD proporciona las siguientes ventajas:

* En Azure AD puede controlar quién tiene acceso a Adobe Experience Manager.
* Puede habilitar que los usuarios inicien sesión automáticamente en Adobe Experience Manager (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Adobe Experience Manager, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Suscripción habilitada para inicio de sesión único en Adobe Experience Manager

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Adobe Experience Manager admite el inicio de sesión único iniciado por **SP y IDP**

* Adobe Experience Manager admite el aprovisionamiento de usuarios **Just In Time**

## <a name="adding-adobe-experience-manager-from-the-gallery"></a>Incorporación de Adobe Experience Manager desde la galería

Para configurar la integración de Adobe Experience Manager en Azure AD, será preciso que lo agregue desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Adobe Experience Manager desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Adobe Experience Manager** , seleccione **Adobe Experience Manager**  en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![Adobe Experience Manager en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con [nombre de aplicación] utilizando usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de [nombre de aplicación].

Para configurar y probar el inicio de sesión único de Azure AD con [nombre de aplicación], es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Adobe](#configure-adobe-experience-manager-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba en Adobe Experience Manager](#create-adobe-experience-manager-test-user)** : para tener un homólogo de Britta Simon en Adobe Experience Manager que esté vinculado a su representación en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con [nombre de aplicación], siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración aplicaciones de **Adobe Experience Manager**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en el modo iniciado por **IDP** siga estos pasos:

    ![Información de direcciones URL de inicio de sesión único y dominio de Adobe Experience Manager](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba un valor único que se define también en el servidor AEM.

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<AEM Server Url>/saml_login`

    > [!NOTE]
    > El valor de dirección URL de respuesta no es real. Actualice la dirección URL de respuesta con la dirección URL de respuesta real. Para obtener este valor, póngase en contacto con el [equipo de soporte técnico de Adobe Experience Manager](https://helpx.adobe.com/support/experience-manager.html). También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    ![Información de direcciones URL de inicio de sesión único y dominio de Adobe Experience Manager](common/metadata-upload-additional-signon.png)

    En el cuadro de texto **Dirección URL de inicio de sesión**, escriba la dirección URL de servidor de Adobe Experience Manager.

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

7. En la sección **Configurar Adobe Experience Manager**, copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-adobe-experience-manager-single-sign-on"></a>Configuración del inicio de sesión único en Adobe Experience Manager

1. En otra ventana del explorador, abra el portal de administración de **Adobe Experience Manager**.

2. Seleccione **Settings** >  (Configuración) **Security** >  (Seguridad) **Users** (Usuarios).

    ![Botón Guardar de Configuración de inicio de sesión único](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_user.png)

3. Seleccione **Administrator** (Administrador) o cualquier otro usuario pertinente.

    ![Botón Configurar inicio de sesión único](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin6.png)

4. Seleccione **Account Settings** > **Manage TrustStore** (Configuración de la cuenta > Administrar TrustStore).

    ![Botón Configurar inicio de sesión único](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_managetrust.png)

5. En **Add Certificate from CER file** (Agregar certificado desde archivo CER), haga clic en **Select Certificate File** (Seleccionar archivo de certificado). Busque el archivo de certificado que ha descargado de Azure Portal y selecciónelo.

    ![Botón Guardar de Configuración de inicio de sesión único](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_user2.png)

6. El certificado se agrega a TrustStore. Anote el alias del certificado.

    ![Botón Configurar inicio de sesión único](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin7.png)

7. En la página **Users** (Usuarios), seleccione **authentication-service**.

    ![Botón Configurar inicio de sesión único](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin8.png)

8. Seleccione **Account Settings** >  (Configuración de la cuenta) **Create/Manage KeyStore** (Crear o administrar KeyStore). Proporcione una contraseña para crear KeyStore.

    ![Botón Configurar inicio de sesión único](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin9.png)

9. Vuelva a la pantalla de administración. Después, seleccione **Settings** > **Operations** > **Web Console** (Configuración > Operaciones > Consola web).

    ![Botón Configurar inicio de sesión único](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin1.png)

    Se abre la página de configuración.

    ![Botón Guardar de Configuración de inicio de sesión único](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin2.png)

10. Busque **Adobe Granite SAML 2.0 Authentication Handler** (Controlador de autenticación de Adobe Granite SAML 2.0). Después, seleccione el botón **Agregar**.

    ![Botón Configurar inicio de sesión único](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin3.png)

11. En esta página, realice las acciones siguientes.

    ![Botón Configurar inicio de sesión único](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin4.png)

    a. En el cuadro **Path** (Ruta de acceso), escriba **/** .

    b. En el cuadro **IDP URL** (URL de IDP), escriba el valor de **Login URL** (URL de inicio de sesión) que copió de Azure Portal.

    c. En el cuadro **IDP Certificate Alias** (Alias de certificado de IDP), introduzca el valor del **alias de certificado** que agregó en TrustStore.

    d. En el cuadro de texto **Security Provided Entity ID** (Identificador de entidad de seguridad proporcionado), escriba el valor único de **Azure AD Identifier** (Identificador de Azure AD) que se ha configurado en Azure Portal.

    e. En el cuadro **Assertion Consumer Service URL** (URL del Servicio de consumidor de aserciones), escriba el valor de **Reply URL** (URL de respuesta) que se ha configurado en Azure Portal.

    f. En el cuadro **Password of Key Store** (Contraseña del almacén de claves), escriba la **contraseña**, que ha establecido en el almacén de claves.

    g. En el cuadro **User Attribute ID** (Identificador de atributo de usuario), escriba **Name ID** (Identificador de nombre) u otro identificador de usuario que sean pertinentes en su caso.

    h. Seleccione **Autocreate CRX Users** (Crear automáticamente usuarios de CRX).

    i. En el cuadro **Logout URL** (URL de cierre de sesión), pegue la dirección de la **URL de cierre de sesión** que obtuvo de Azure Portal.

    j. Seleccione **Guardar**.

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

En esta sección, se habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Adobe Experience Manager.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Adobe Experience Manager**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Adobe Experience Manager**.

    ![Vínculo a Adobe Experience Manager en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-adobe-experience-manager-test-user"></a>Creación de un usuario de prueba de Adobe Experience Manager

En esta sección, se crea un usuario denominado Britta Simon en Adobe Experience Manager. Si ha seleccionado la opción **Autocreate CRX Users** (Crear automáticamente usuarios de CRX), los usuarios se crearán automáticamente tras la autenticación correcta.

Si desea crear los usuarios manualmente, trabaje con el [equipo de soporte técnico de Adobe Experience Manager](https://helpx.adobe.com/support/experience-manager.html)  para agregar los usuarios en la plataforma de Adobe Experience Manager.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Adobe Experience Manager en el panel de acceso, debería iniciar sesión automáticamente en la aplicación Adobe Experience Manager para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
