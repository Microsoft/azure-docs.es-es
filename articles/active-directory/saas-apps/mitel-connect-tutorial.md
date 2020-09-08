---
title: 'Tutorial: Integración de Azure Active Directory con Mitel Connect | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Mitel Connect.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/31/2020
ms.author: jeedes
ms.openlocfilehash: 63f1d1d40d8aff21641f3fa4ee10a289de40800d
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88552618"
---
# <a name="tutorial-azure-active-directory-integration-with-mitel-micloud-connect-or-cloudlink-platform"></a>Tutorial: Integración de Azure Active Directory con Mitel MiCloud Connect o CloudLink Platform

En este tutorial, aprenderá a usar la aplicación Mitel Connect para integrar Azure Active Directory (Azure AD) con Mitel MiCloud Connect o CloudLink Platform. La aplicación Mitel Connect está disponible en la galería de Azure. La integración de Azure AD con MiCloud Connect o CloudLink Platform le proporciona las siguientes ventajas:

* Puede controlar el acceso de los usuarios a las aplicaciones de MiCloud Connect y de CloudLink en Azure AD con sus credenciales de empresa.
* Puede permitir que los usuarios de su cuenta inicien sesión automáticamente en MiCloud Connect o en CloudLink Platform (inicio de sesión único) con sus cuentas de Azure AD.

Para obtener información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

Si no tiene una suscripción a Azure, [cree una cuenta gratuita](https://azure.microsoft.com/free/) antes de comenzar la integración de Azure AD con Mitel MiCloud Connect o CloudLink Platform.

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con MiCloud Connect, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una cuenta de Mitel MiCloud Connect o Mitel CloudLink, en función de la aplicación que desee configurar.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial configurará y probará el inicio de sesión único (SSO) de Azure AD.

* Mitel Connect admite SSO iniciado por **SP**
* Una vez que haya configurado Mitel Connect, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="add-mitel-connect-from-the-gallery"></a>Adición de Mitel Connect desde la galería

Para configurar la integración de Mitel Connect en Azure AD, será preciso que agregue Mitel Connect desde la galería a la lista de aplicaciones SaaS administradas en Azure Portal.

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Seleccione **Aplicaciones empresariales** y, después, **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Seleccione **Nueva aplicación**.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. Escriba **Mitel Connect** en el campo de búsqueda, seleccione **Mitel Connect** en el panel de resultados y, a continuación, seleccione **Agregar**.

     ![Mitel Connect en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con MiCloud Connect o CloudLink Platform con un usuario de prueba llamado**_Britta Simon_**. Para que el inicio de sesión único funcione, debe establecerse un vínculo entre el usuario en el portal de Azure AD y el usuario correspondiente en la plataforma de Mitel. Consulte las siguientes secciones para obtener información sobre cómo configurar y probar el inicio de sesión único de Azure AD con MiCloud Connect o CloudLink Platform.
* Configuración y prueba del inicio de sesión único de Azure AD con MiCloud Connect
* Configuración y prueba del inicio de sesión único de Azure AD con CloudLink Platform

## <a name="configure-and-test-azure-ad-sso-with-micloud-connect"></a>Configuración y prueba del inicio de sesión único de Azure AD con MiCloud Connect

Para configurar y probar el inicio de sesión único en Azure AD con MiCloud Connect:

1. **[Configuración de MiCloud Connect para SSO con Azure AD](#configure-micloud-connect-for-sso-with-azure-ad)**, para permitir que los usuarios utilicen esta característica y configuren las opciones de inicio de sesión único en el lado de la aplicación.
2. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
3. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
4. **[Creación de un usuario de prueba de Mitel MiCloud Connect](#create-a-mitel-micloud-connect-test-user)**, para tener un homólogo de Britta Simon en la cuenta de MiCloud Connect que esté vinculado a su representación en Azure AD.
5. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

## <a name="configure-micloud-connect-for-sso-with-azure-ad"></a>Configuración de MiCloud Connect para SSO con Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD para MiCloud Connect en Azure Portal y configurará su cuenta de MiCloud Connect para permitir el inicio de sesión único con Azure AD.

Para configurar el SSO de MiCloud Connect para Azure AD, lo más fácil es abrir Azure Portal y el portal de Mitel Account en paralelo. Deberá copiar información desde Azure Portal al portal de Mitel Account y desde el portal de Mitel Account a Azure Portal.


1. Para abrir la página de configuración en [Azure Portal](https://portal.azure.com/):

    1. En la página de integración de aplicaciones de **Mitel Connect**, seleccione **Inicio de sesión único**.

       ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

    1. En el cuadro de diálogo **Seleccione un método de inicio de sesión único**, elija **SAML**.
    
       ![Modo de selección de inicio de sesión único](common/select-saml-option.png)
    
       Aparece la página de inicio de sesión basada en SAML.

2. Para abrir el cuadro de diálogo de configuración en el portal de Mitel Account:

    1. En el menú **Phone System** (Sistema telefónico), seleccione **Add-On Features** (Características complementarias).

    1. A la derecha de **Single Sign-On** (Inicio de sesión único), seleccione **Activate** (Activar) o **Settings** (Configuración).
    
    Aparece el cuadro de diálogo de configuración del inicio de sesión único de Connect.
    
3. Active la casilla **Enable Single Sign-On** (Habilitar el inicio de sesión único).
    
    ![Captura de pantalla que muestra la página de configuración del inicio de sesión único de Mitel Connect, con la casilla Enable Single Sign-On (Habilitar el inicio de sesión único) activada.](./media/mitel-connect-tutorial/mitel-connect-enable.png)

4. En Azure Portal, seleccione el icono **Editar** en la sección **Configuración básica de SAML**.
   
    ![imagen](common/edit-urls.png)

    Aparece el cuadro de diálogo de configuración básica de SAML.

5.  Copie la dirección URL del campo **Mitel Identifier (Entity ID)** (Identificador de Mitel [id. de entidad]) del portal de Mitel Account y péguelo en el campo **Identificador (Id. de entidad)** en Azure Portal.

6. Copie la dirección URL del campo **Reply URL (Assertion Consumer Service URL)** (URL de respuesta [URL del servicio de consumidor de aserciones]) del portal de Mitel Account y péguelo en el campo **URL de respuesta (URL del Servicio de consumidor de aserciones)** de Azure Portal.

   ![imagen](./media/mitel-connect-tutorial/mitel-azure-basic-configuration.png)

7. En el cuadro de texto **URL de inicio de sesión**, escriba una de las siguientes direcciones URL:

    1. **https://portal.shoretelsky.com** : para usar el portal de Mitel Account como aplicación Mitel predeterminada
    1. **https://teamwork.shoretel.com** : para usar Teamwork como aplicación Mitel predeterminada

    > [!NOTE]
    > Cuando un usuario selecciona el icono de Mitel Connect en el Panel de acceso, se accede a la aplicación de Mitel predeterminada. También es la aplicación a la que se accede al realizar una instalación de prueba desde Azure AD.

8. Seleccione **Guardar** en el cuadro de diálogo **Configuración básica de SAML** de Azure Portal.

9. En la sección **Certificado de firma de SAML** de la página **Inicio de sesión basado en SAML** de Azure Portal, seleccione **Descargar** junto a **Certificado (Base64)** para descargar el **Certificado de firma** y guardarlo en su equipo.

    ![imagen](./media/mitel-connect-tutorial/azure-signing-certificate.png)

10. Abra el archivo del certificado de firma en un editor de texto, copie todos los datos del archivo y, a continuación, pegue los datos en el campo **Signing Certificate** (Certificado de firma) del portal de Mitel Account. 

      ![imagen](./media/mitel-connect-tutorial/mitel-connect-signing-certificate.png)

11. En la sección **Configurar Mitel Connect** de la página **Inicio de sesión basado en SAML** de Azure Portal:

     1. Copie la dirección URL del campo **URL de inicio de sesión** y péguela en el campo **Sign-in URL** (URL de inicio de sesión) del portal de Mitel Account.

     1. Copie la dirección URL del campo **Identificador de Azure AD** y péguela en el campo **Entity ID** (Id. de entidad) del portal de Mitel Account.
         
         ![imagen](./media/mitel-connect-tutorial/mitel-azure-set-up-connect.png)

12. Seleccione **Save** (Guardar) en el cuadro de diálogo **Connect Single Sign-On Settings** (Configuración del inicio de sesión único de Connect) en el portal de Mitel Account.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

En esta sección creará un usuario de prueba llamado Britta Simon en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En el cuadro de diálogo Propiedades del usuario, siga estos pasos:

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    1. En el campo **Nombre**, escriba **BrittaSimon**.
  
    1. En el campo **Nombre de usuario**, escriba brittasimon@\<yourcompanydomain\>.\<extension\>.  Por ejemplo, BrittaSimon@contoso.com.

    1. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro **Contraseña**.

    1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Mitel Connect.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Mitel Connect**.

    ![Vínculo a Mitel Connect en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista  **Usuarios** y, luego, elija **Seleccionar** en la parte inferior de la pantalla.

6. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista y, después, elija **Seleccionar** en la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

### <a name="create-a-mitel-micloud-connect-test-user"></a>Creación de un usuario de prueba de Mitel MiCloud Connect

En esta sección, creará un usuario llamado Britta Simon en su cuenta de MiCloud Connect. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

Para obtener más información sobre cómo agregar usuarios en el portal de Mitel Account, consulte el artículo [Adding a user](https://oneview.mitel.com/s/article/Adding-a-User-092815) (Adición de un usuario en la base de conocimientos de Mitel).

Cree un usuario en su cuenta de MiCloud Connect con los detalles siguientes:

* **Nombre:** Britta Simon
* **Dirección de correo electrónico empresarial:** `brittasimon@<yourcompanydomain>.<extension>`   
  (Ejemplo: [brittasimon@contoso.com](mailto:brittasimon@contoso.com))
* **Nombre de usuario:** `brittasimon@<yourcompanydomain>.<extension>`  
  (Ejemplo: [brittasimon@contoso.com](mailto:brittasimon@contoso.com); el nombre de usuario suele ser el mismo que la dirección de correo electrónico empresarial del usuario)

> [!NOTE]
> El nombre del usuario de MiCloud Connect del usuario debe ser idéntico a la dirección de correo electrónico del usuario en Azure.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al seleccionar el icono de Mitel Connect en el Panel de acceso, se le redirigirá automáticamente para iniciar sesión en la aplicación de MiCloud Connect que configuró como predeterminada en el campo **Sign on URL** (Dirección URL de inicio de sesión). Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="configure-and-test-azure-ad-sso-with-cloudlink-platform"></a>Configuración y prueba del inicio de sesión único de Azure AD con CloudLink Platform

En esta sección se describe cómo habilitar el inicio de sesión único de Azure AD para la plataforma CloudLink en Azure Portal y cómo configurar la cuenta de la plataforma CloudLink para permitir el inicio de sesión único con Azure AD.

Para configurar la plataforma de CloudLink con el inicio de sesión único para Azure AD, se recomienda que abra Azure Portal y el portal de CloudLink Accounts en paralelo, ya que tendrá que copiar cierta información de Azure Portal al portal de CloudLink Accounts y viceversa.

1. Para abrir la página de configuración en [Azure Portal](https://portal.azure.com/):

    1. En la página de integración de aplicaciones de **Mitel Connect**, seleccione **Inicio de sesión único**.

       ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

    1. En el cuadro de diálogo **Seleccione un método de inicio de sesión único**, elija **SAML**.

       ![Modo de selección de inicio de sesión único](common/select-saml-option.png)
    
       Se abre la página **Inicio de sesión basado en SAML**, que muestra la sección **Configuración básica de SAML**.

       ![imagen](./media/mitel-connect-tutorial/mitel-azure-saml-settings.png)

2. Para acceder al panel de configuración de **Azure AD Single Sign On** (Inicio de sesión único de Azure AD) en el portal de CloudLink Accounts:

    1. Vaya a la página **Account Information** (Información de cuenta) de la cuenta de cliente con la que desea habilitar la integración.

    1. En la sección **Integrations** (Integraciones), seleccione **+ Add new** (+ Agregar nueva). En una pantalla emergente se muestra el panel **Integrations** (Integraciones).

    1. Seleccione la pestaña **3rd party** (Terceros). Se muestra una lista de aplicaciones de terceros admitidas. Seleccione el botón **Add** (Agregar) asociado a **Azure AD Single Sign On** (Inicio de sesión único de Azure AD) y seleccione **Done** (Hecho).

       ![imagen](./media/mitel-connect-tutorial/mitel-cloudlink-integrations.png)

       El **inicio de sesión único de Azure AD** se habilita para la cuenta de cliente y se agrega a la sección **Integrations** (Integraciones) de la página **Account Information** (Información de la cuenta).   

   1. Seleccione **Complete Setup** (Completar configuración).
    
      ![imagen](./media/mitel-connect-tutorial/mitel-cloudlink-complete-setup.png)
      
      Se abre el panel de configuración de **Azure AD Single Sign On** (Inicio de sesión de Azure AD).
      
       ![imagen](./media/mitel-connect-tutorial/mitel-cloudlink-sso-setup.png)
       
       Mitel recomienda que la casilla **Enable Mitel Credentials (Optional)** (Habilitar credenciales de Mitel [opcional]) de la sección **Optional Mitel credentials** (Credenciales de Mitel opcionales) no se active. Active esta casilla solo si desea que el usuario inicie sesión en la aplicación CloudLink con las credenciales de Mitel además de con la opción de inicio de sesión único.

3. En Azure Portal, en la página **Inicio de sesión basado en SAML**, seleccione el icono **Editar** en la sección **Configuración básica de SAML**. Se abre el panel **Configuración básica de SAML**.

    ![imagen](./media/mitel-connect-tutorial/mitel-azure-saml-basic.png)
 
 4. Copie la dirección URL del campo **Mitel Identifier (Entity ID)** (Identificador de Mitel [Id. de entidad]) del portal de CloudLink Accounts y péguelo en el campo **Identificador (Id. de entidad)** en Azure Portal.

 5. Copie la dirección URL del campo **Reply URL (Assertion Consumer Service URL)** (URL de respuesta [URL del servicio de consumidor de aserciones]) del portal de CloudLink Accounts y péguelo en el campo **URL de respuesta (URL del Servicio de consumidor de aserciones)** de Azure Portal.  
    
    ![imagen](./media/mitel-connect-tutorial/mitel-cloudlink-saml-mapping.png) 

 6. En el cuadro de texto **Sign-on URL** (Dirección URL de inicio de sesión), escriba la dirección URL `https://accounts.mitel.io` para usar el portal de CloudLink Accounts como aplicación predeterminada de Mitel.
     
     ![imagen](./media/mitel-connect-tutorial/mitel-cloudlink-sign-on-url.png)
  
     > [!NOTE]
     > La aplicación de Mitel predeterminada es la que se abre cuando un usuario hace clic en el icono de Mitel Connect en el Panel de acceso. También es la aplicación a la que se accede cuando el usuario configura una instalación de prueba desde Azure AD.

7. Seleccione **Guardar** en el cuadro de diálogo **Configuración básica de SAML**.

8. En la sección **Certificado de firma de SAML** de la página **Inicio de sesión basado en SAML** de Azure Portal, seleccione **Descargar** junto a **Certificado (Base64)** para descargar el **Certificado de firma**. Guarde el certificado en el equipo.
  
    ![imagen](./media/mitel-connect-tutorial/mitel-cloudlink-save-certificate.png)

9. Abra el archivo del certificado de firma en un editor de texto, copie todos los datos del archivo y, a continuación, pegue los datos en el campo **Signing Certificate** (Certificado de firma) del portal de CloudLink Accounts.  

    > [!NOTE]
    > Si tiene más de un certificado, se recomienda que los pegue de uno en uno. 
       
    ![imagen](./media/mitel-connect-tutorial/mitel-cloudlink-enter-certificate.png)

10. En la sección **Configurar Mitel Connect** de la página **Inicio de sesión basado en SAML** de Azure Portal:

     1. Copie la dirección URL del campo **URL de inicio de sesión** y péguela en el campo **Sign-in URL** (URL de inicio de sesión) del portal de CloudLink Accounts.

     1. Copie la dirección URL del campo **Identificador de Azure AD** y péguela en el campo **IDP Identifier (Entity ID)** (Identificador de IDP [Id. de entidad]) del portal de CloudLink Accounts.
     
        ![imagen](./media/mitel-connect-tutorial/mitel-cloudlink-copy-settings.png)

11. Seleccione **Save** (Guardar) en el panel **Azure AD Single Sign On** (Inicio de sesión único de Azure AD) en el portal de CloudLink Accounts.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

En esta sección creará un usuario de prueba llamado Britta Simon en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En el cuadro de diálogo Propiedades del usuario, siga estos pasos:

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    1. En el campo **Nombre**, escriba **BrittaSimon**.
  
    1. En el campo **Nombre de usuario**, escriba brittasimon@\<yourcompanydomain\>.\<extension\>.  Por ejemplo, BrittaSimon@contoso.com.

    1. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro **Contraseña**.

    1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Mitel Connect.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Mitel Connect**.

    ![Vínculo a Mitel Connect en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista  **Usuarios** y, luego, elija **Seleccionar** en la parte inferior de la pantalla.

6. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista y, después, elija **Seleccionar** en la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

### <a name="create-a-cloudlink-test-user"></a>Creación de un usuario de prueba de CloudLink

En esta sección se describe cómo crear una usuaria de prueba llamada **_Britta Simon_** en la plataforma de CloudLink. Los usuarios se deben crear y activar para que puedan usar el inicio de sesión único.

Para obtener más información sobre cómo agregar usuarios en el portal de CloudLink Accounts, consulte **_Administración de usuarios_** en la [documentación de CloudLink Accounts](https://www.mitel.com/document-center/technology/cloudlink/all-releases/en/cloudlink-accounts-html).

Cree un usuario en el portal de CloudLink Accounts con los detalles siguientes:

* Nombre: Britta Simon
* First name (Nombre): Britta
* Last name (Apellidos): Simon
* Correo electrónico: BrittaSimon@contoso.com

> [!NOTE]
> La dirección de correo electrónico de CloudLink del usuario debe ser idéntica al **nombre principal de usuario** en Azure Portal.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Mitel Connect del Panel de acceso, se le redirigirá automáticamente para iniciar sesión en la aplicación CloudLink que configuró como predeterminada en el campo **Sign on URL** (Dirección URL de inicio de sesión). Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)