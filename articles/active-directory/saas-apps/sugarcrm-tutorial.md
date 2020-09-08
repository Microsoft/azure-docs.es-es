---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Sugar CRM | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Sugar CRM.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/17/2019
ms.author: jeedes
ms.openlocfilehash: cd1be43bd8984df8fef690e595ec99f4b112c3c1
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88533719"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-sugar-crm"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Sugar CRM

En este tutorial, aprenderá a integrar Sugar CRM con Azure Active Directory (Azure AD). Al integrar Sugar CRM con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a Sugar CRM.
* Permitir a los usuarios iniciar sesión automáticamente en Sugar CRM con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Sugar CRM.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Sugar CRM admite el SSO iniciado por **SP**

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-sugar-crm-from-the-gallery"></a>Agregación de Sugar CRM desde la galería

Para configurar la integración de Sugar CRM en Azure AD, deberá agregar Sugar CRM desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Sugar CRM** en el cuadro de búsqueda.
1. Seleccione **Sugar CRM** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-sugar-crm"></a>Configuración y prueba del inicio de sesión único en Azure AD para Sugar CRM

Configure y pruebe el inicio de sesión único de Azure AD con Sugar CRM utilizando un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Sugar CRM.

Para configurar y probar el inicio de sesión único de Azure AD con Sugar CRM, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Sugar CRM](#configure-sugar-crm-sso)** : para configurar los valores del inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Sugar CRM](#create-sugar-crm-test-user)** : para tener un homólogo de B.Simon en Sugar CRM que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Sugar CRM**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **Dirección URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón:

    - `https://<companyname>.sugarondemand.com`
    - `https://<companyname>.trial.sugarcrm`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón:

    - `https://<companyname>.sugarondemand.com/<companyname>`
    - `https://<companyname>.trial.sugarcrm.com/<companyname>`
    - `https://<companyname>.trial.sugarcrm.eu/<companyname>`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de Sugar CRM](https://support.sugarcrm.com/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up Sugar CRM** (Configurar Sugar CRM), copie las direcciones URL adecuadas en función de sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Sugar CRM mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Sugar CRM**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-sugar-crm-sso"></a>Configuración del inicio de sesión único en Sugar CRM

1. En otra ventana del explorador web, inicie sesión en su sitio de la compañía de Sugar CRM como administrador.

1. Vaya a **Administración**.

    ![Administrador](./media/sugarcrm-tutorial/ic795888.png "Administración")

1. En la sección **Administración**, haga clic en **Administración de contraseñas**.

    ![Administración](./media/sugarcrm-tutorial/ic795889.png "Administración")

1. Seleccione **Habilitar autenticación SAML**.

    ![Administración](./media/sugarcrm-tutorial/ic795890.png "Administración")

1. En la sección **Autenticación SAML** , realice los pasos siguientes:

    ![SAML Authentication (Autenticación SAML)](./media/sugarcrm-tutorial/ic795891.png "Autenticación SAML")  

    a. En el cuadro de texto **Login URL** (Dirección URL de inicio de sesión), pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.
  
    b. En el cuadro de texto **SLO URL** (Dirección URL de SLO), pegue el valor de **Dirección URL de cierre de sesión** que ha copiado de Azure Portal.
  
    c. Abra el certificado codificado en base 64 en el Bloc de notas, copie su contenido en el Portapapeles y luego pegue todo el certificado en el cuadro de texto **Certificado X.509** .
  
    d. Haga clic en **Save**(Guardar).

### <a name="create-sugar-crm-test-user"></a>Creación de un usuario de prueba en Sugar CRM

Para permitir que los usuarios de Azure AD inicien sesión en Sugar CRM, deben aprovisionarse en Sugar CRM. En el caso de SugarCRM, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en su sitio de compañía de **Sugar CRM** como administrador.

1. Vaya a **Administración**.

    ![Administrador](./media/sugarcrm-tutorial/ic795888.png "Administración")

1. En la sección **Administración**, haga clic en **Administración de usuarios**.

    ![Administración](./media/sugarcrm-tutorial/ic795893.png "Administración")

1. Vaya a **Usuarios \> Crear nuevo usuario**.

    ![Create New User (Crear nuevo usuario)](./media/sugarcrm-tutorial/ic795894.png "Crear nuevo usuario")

1. En la pestaña **Perfil de usuario** , realice los siguientes pasos:

    ![New User](./media/sugarcrm-tutorial/ic795895.png "Nuevo usuario") (Usuario nuevo)

    * Escriba el **nombre de usuario**, **apellidos** y la **dirección de correo electrónico** de un usuario de Azure Active Directory válido en los cuadros de texto relacionados.
  
1. Como **Estado**, seleccione **Activo**.

1. En la pestaña Contraseña, realice los pasos siguientes:

    ![New User](./media/sugarcrm-tutorial/ic795896.png "Nuevo usuario") (Usuario nuevo)

    a. Escriba la contraseña en el cuadro de texto relacionado.

    b. Haga clic en **Save**(Guardar).

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de SugarCRM ofrecida por SugarCRM para aprovisionar cuentas de usuario de Azure AD.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Sugar CRM en el panel de acceso debería iniciar sesión automáticamente en la versión de Sugar CRM para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Probar Sugar CRM con Azure AD](https://aad.portal.azure.com/)

