---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con CloudPassage | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y CloudPassage.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/22/2019
ms.author: jeedes
ms.openlocfilehash: 34e36be8ee9444b5612901439aa5dca4771f2dda
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88551694"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-cloudpassage"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con CloudPassage

En este tutorial, aprenderá a integrar CloudPassage con Azure Active Directory (Azure AD). Al integrar CloudPassage con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a CloudPassage.
* Permitir que los usuarios inicien sesión automáticamente en CloudPassage con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en CloudPassage.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* CloudPassage admite el inicio de sesión único iniciado por **SP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-cloudpassage-from-the-gallery"></a>Adición de CloudPassage desde la galería

Para configurar la integración de CloudPassage en Azure AD, deberá agregar CloudPassage desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **CloudPassage** en el cuadro de búsqueda.
1. Seleccione **CloudPassage** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-single-sign-on-for-cloudpassage"></a>Configuración y prueba del inicio de sesión único de Azure AD para CloudPassage

Configure y pruebe el inicio de sesión único de Azure AD con CloudPassage mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de CloudPassage.

Para configurar y probar el inicio de sesión único de Azure AD con CloudPassage, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en CloudPassage](#configure-cloudpassage-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de CloudPassage](#create-cloudpassage-test-user)** : para tener un homólogo de B.Simon en CloudPassage que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **CloudPassage**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

     a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://portal.cloudpassage.com/saml/init/accountid`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://portal.cloudpassage.com/saml/consume/accountid`. Para obtener el valor de este atributo, haga clic en **SSO Setup documentation** (Documentación de instalación de inicio de sesión única) en la sección **Single Sign-on Settings** (Configuración de inicio de sesión único) del portal de CloudPassage.

    ![Configurar inicio de sesión único](./media/cloudpassage-tutorial/tutorial_cloudpassage_05.png)

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte de cliente de CloudPassage](https://www.cloudpassage.com/company/contact/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación CloudPassage espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/edit-attribute.png)

1. Además de lo anterior, la aplicación CloudPassage espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

    | Nombre | Atributo de origen|
    | ---------------| --------------- |
    | firstname |user.givenname |
    | lastname |user.surname |
    | email |user.mail |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar CloudPassage**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a conceder a B.Simon acceso a CloudPassage mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **CloudPassage**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-cloudpassage-sso"></a>Configuración del inicio de sesión único de CloudPassage

1. En otra ventana del explorador, inicie sesión en su sitio de la compañía de CloudPassage como administrador.

1. En el menú de la parte superior, haga clic en **Settings** (Configuración) y en **Site Administration** (Administración del sitio). 
   
    ![Configurar inicio de sesión único][12]

1. Haga clic en la pestaña **Authentication Settings** (Configuración de autenticación). 
   
    ![Configurar inicio de sesión único][13]

1. En la sección **Single Sign-on Settings** (Configuración del inicio de sesión único), siga estos pasos: 
   
    ![Configurar inicio de sesión único][14]

    a. Seleccione la casilla **Enable Single sign-on(SSO)(SSO Setup Documentation)** (Habilitar el inicio de sesión único (SSO) (Documentación de configuración de SSO)).
    
    b. Pegue el valor de **Identificador de Azure AD** en el cuadro de texto **SAML issuer URL** (Dirección URL del emisor de SAML).
  
    c. Pegue el valor de **Login URL** (Dirección URL de inicio de sesión) en el cuadro de texto **SAML endpoint URL** (Dirección URL del punto de conexión de SAML).
  
    d. Pegue el valor de **Logout URL** (Dirección URL de cierre de sesión) en el cuadro de texto **Logout landing page** (Página de aterrizaje de cierre de sesión).
  
    e. Abra el certificado descargado en el Bloc de notas, copie el contenido del mismo en el Portapapeles y luego péguelo en el cuadro de texto **Certificado x 509**.
  
    f. Haga clic en **Save**(Guardar).

### <a name="create-cloudpassage-test-user"></a>Creación de un usuario de prueba de CloudPassage

El objetivo de esta sección es crear un usuario llamado B.Simon en CloudPassage.

**Para crear un usuario llamado B.Simon en CloudPassage, realice los pasos siguientes:**

1. Inicie sesión en su sitio de la compañía de **CloudPassage** como administrador. 

1. En la barra de herramientas de la parte superior, haga clic en **Settings** (Configuración) y en **Site Administration** (Administración del sitio). 
   
    ![Creación de un usuario de prueba de CloudPassage][22] 

1. Haga clic en la pestaña **Users** (Usuarios) y, a continuación, en **Add New User** (Agregar nuevo usuario). 
   
    ![Creación de un usuario de prueba de CloudPassage][23]

1. En la sección **Agregar nuevo usuario** , lleve a cabo estos pasos: 
   
    ![Creación de un usuario de prueba de CloudPassage][24]
    
    a. En el cuadro de texto **First Name** (Nombre), escriba Britta. 
  
    b. En el cuadro de texto **Last Name** (Apellidos), escriba Simon.
  
    c. En el cuadro de texto **Username** (Nombre de usuario), el cuadro de texto **Email** (Correo electrónico) y el cuadro de texto **Retype Email** (Volver a escribir correo electrónico), escriba el nombre de usuario de Britta en Azure AD.
  
    d. En **Access Type** (Tipo de acceso), seleccione **Enable Halo Portal Access** (Habilitar el acceso de Portal de Halo).
  
    e. Haga clic en **Agregar**.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de CloudPassage en el Panel de acceso, debería iniciar sesión automáticamente en la versión de CloudPassage para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe CloudPassage con Azure AD](https://aad.portal.azure.com/).

<!--Image references-->

[12]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_07.png
[13]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_08.png
[14]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_09.png
[15]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_10.png
[22]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_15.png
[23]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_16.png
[24]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_17.png

