---
title: 'Tutorial: Integración de Azure Active Directory con Wdesk | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Wdesk.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/02/2020
ms.author: jeedes
ms.openlocfilehash: 9e64d76e29ba98181aac12e1e3167351a9c1aa95
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88524024"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-wdesk"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Wdesk

En este tutorial, aprenderá a integrar Wdesk con Azure Active Directory (Azure AD). Al integrar Wdesk con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Wdesk.
* Permitir que los usuarios inicien sesión automáticamente en Wdesk con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Wdesk.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Wdesk admite SSO iniciado por **SP** e **IDP**
* Una vez configurado Wdesk, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-aad).

## <a name="adding-wdesk-from-the-gallery"></a>Adición de Wdesk desde la galería

Para configurar la integración de Wdesk en Azure AD, es preciso agregar Wdesk desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Wdesk** en el cuadro de búsqueda.
1. Seleccione **Wdesk** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Wdesk con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Wdesk.

Para configurar y probar el inicio de sesión único de Azure AD con Wdesk, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Wdesk](#configure-wdesk-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Wdesk](#create-wdesk-test-user)** , para tener un homólogo de B.Simon en Wdesk que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Wdesk, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Wdesk**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice los siguientes pasos:

    ![Información acerca del inicio de sesión único de dominio y direcciones URL de Wdesk](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<subdomain>.wdesk.com/auth/saml/sp/metadata/<instancename>`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<subdomain>.wdesk.com/auth/saml/sp/consumer/<instancename>`

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    ![Información acerca del inicio de sesión único de dominio y direcciones URL de Wdesk](common/metadata-upload-additional-signon.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<subdomain>.wdesk.com/auth/login/saml/<instancename>`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Estos valores se obtienen de portal de WDesk al configurar el SSO.

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up Wdesk** (Configurar Wdesk), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba brittasimon@yourcompanydomain.extension. Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Wdesk.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **Wdesk**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Wdesk**.

    ![Vínculo a Wdesk en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-wdesk-sso"></a>Configuración del inicio de sesión único de Wdesk

1. En otra ventana del explorador web, inicie sesión en Wdesk como administrador de seguridad.

2. En la parte inferior izquierda, haga clic en **Admin** (Administración) y elija **Account Admin** (Administrador de cuenta):
 
     ![Configurar inicio de sesión único](./media/wdesk-tutorial/tutorial_wdesk_ssoconfig1.png)

3. En Wdesk Admin, navegue hasta **Security** (Seguridad) y, después, **SAML** > **SAML Settings** (Configuración de SAML):

    ![Configurar inicio de sesión único](./media/wdesk-tutorial/tutorial_wdesk_ssoconfig2.png)

1. En **SAML User ID Settings** (Configuración de Id. de usuario de SAML), marque **SAML User ID is Wdesk Username** (Id. de usuario de SAML es nombreDeUsuario de Wdesk).

    ![Configurar inicio de sesión único](./media/wdesk-tutorial/wdesk-username.png)

4. En **General Settings** (Configuración general), seleccione la casilla **Enable SAML Single Sign On** (Habilitar inicio de sesión único de SAML):

    ![Configurar inicio de sesión único](./media/wdesk-tutorial/tutorial_wdesk_ssoconfig3.png)

5. En **Service Provider Details** (Detalles del proveedor de servicios), siga estos pasos:

    ![Configurar inicio de sesión único](./media/wdesk-tutorial/tutorial_wdesk_ssoconfig4.png)

      a. Copia el valor de **Login URL** (Dirección URL de inicio de sesión) y péguelo en el cuadro de texto **Dirección URL de inicio de sesión** de Azure Portal.
   
      b. Copia el valor de **Metadata Url** (Dirección URL de metadatos) y péguelo en el cuadro de texto **Identificador** de Azure Portal.
       
      c. Copia el valor de **Consumer url** (Dirección URL del consumidor ) y péguelo en el cuadro de texto **URL de respuesta** de Azure Portal.
   
      d. Haga clic en **Guardar** en Azure Portal para guardar los cambios.      

6. Haga clic en **Configure IdP Settings** (Configurar valores de IdP) para abrir el cuadro de diálogo **Edit IdP Settings** (Editar valores de IdP). Haga clic en **Choose File** (Elegir archivo) para buscar el archivo **Metadata.xml** que guardó en Azure Portal y cárguelo.
    
    ![Configurar inicio de sesión único](./media/wdesk-tutorial/tutorial_wdesk_ssoconfig5.png)
  
7. Haga clic en **Guardar cambios**.

    ![Configurar inicio de sesión único](./media/wdesk-tutorial/tutorial_wdesk_ssoconfigsavebutton.png)

### <a name="create-wdesk-test-user"></a>Creación de usuario de prueba de Wdesk

Para permitir que los usuarios de Azure AD inicien sesión en Wdesk, tienen que aprovisionarse en Wdesk. En el caso de Wdesk, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en Wdesk como administrador de seguridad.

2. Navegue hasta **Admin** (Administración)  > **Account Admin** (Administrador de cuenta).

     ![Configurar inicio de sesión único](./media/wdesk-tutorial/tutorial_wdesk_ssoconfig1.png)

3. Haga clic en **Members** (Miembros) en **People** (Contactos).

4. A continuación, haga clic en **Add Member** (Agregar miembro) para abrir el cuadro de diálogo **(Agregar miembro)** . 
   
    ![Creación de un usuario de prueba de Azure AD](./media/wdesk-tutorial/createuser1.png)  

5. En el cuadro de texto **User** (Usuario), escriba el nombre del usuario así b.simon@contoso.com y haga clic en el botón **Continue** (Continuar).

    ![Creación de un usuario de prueba de Azure AD](./media/wdesk-tutorial/createuser3.png)

6.  Escriba los detalles que se muestran a continuación:
  
    ![Creación de un usuario de prueba de Azure AD](./media/wdesk-tutorial/createuser4.png)
 
    a. En el cuadro de texto **E-mail** (Correo electrónico), escriba el correo electrónico del usuario con el siguiente formato b.simon@contoso.com.

    b. En el cuadro de texto **First Name** (Nombre), escriba el nombre del usuario, en este caso **B**.

    c. En el cuadro de texto **Last Name** (Apellidos), escriba el nombre de usuario, en este caso **Simon**.

7. Haga clic en el botón **Save Member** (Guardar miembro).  

    ![Creación de un usuario de prueba de Azure AD](./media/wdesk-tutorial/createuser5.png)

### <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Wdesk en el panel de acceso y debería iniciar sesión automáticamente en la versión de Wdesk para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)