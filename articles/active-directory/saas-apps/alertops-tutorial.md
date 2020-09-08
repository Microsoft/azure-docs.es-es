---
title: 'Tutorial: Integración de Azure Active Directory con AlertOps | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y AlertOps.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/20/2019
ms.author: jeedes
ms.openlocfilehash: d820a3fc9b12a36a76e2eb887b32d27acc156c76
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88522364"
---
# <a name="tutorial-integrate-alertops-with-azure-active-directory"></a>Tutorial: Integración de AlertOps con Azure Active Directory

En este tutorial, obtendrá información sobre cómo integrar AlertOps con Azure Active Directory (Azure AD). Al integrar AlertOps con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a AlertOps.
* Permitir que los usuarios puedan iniciar sesión automáticamente en AlertOps con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción de AlertOps con el inicio de sesión único habilitado.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. AlertOps admite el inicio de sesión único iniciado por **SP e IDP**.

## <a name="adding-alertops-from-the-gallery"></a>Adición de AlertOps desde la galería

Para configurar la integración de AlertOps en Azure AD, será preciso que añada AlertOps desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **AlertOps** en el cuadro de búsqueda.
1. Seleccione **AlertOps** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

Configure y pruebe el inicio de sesión único de Azure AD con AlertOps utilizando un usuario de prueba llamado **Britta Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de AlertOps.

Para configurar y probar el inicio de sesión único de Azure AD con AlertOps, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
2. **[Configuración de AlertOps](#configure-alertops)** , para configurar las opciones del SSO en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba en AlertOps](#create-alertops-test-user)** : para tener un homólogo de Britta Simon en AlertOps que esté vinculado a la representación del usuario en Azure AD.
6. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **AlertOps**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice los siguientes pasos:

    1. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<SUBDOMAIN>.alertops.com`

    1. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.alertops.com/login.aspx`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.alertops.com/login.aspx`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte de cliente de AlertOps](mailto:support@alertops.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

   ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up AlertOps** (Configurar AlertOps), copie las direcciones URL adecuadas según sus necesidades.

   ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="configure-alertops"></a>Configuración de AlertOps

1. Para automatizar la configuración en AlertOps, debe instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Configurar AlertOps** para ir a la aplicación AlertOps. Desde allí, proporcione las credenciales de administrador para iniciar sesión en AlertOps. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 5.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar AlertOps manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de AlertOps como administrador y haga lo siguiente:

4. Haga clic en **Configuración de cuenta** en el panel de navegación izquierdo.

    ![Configuración de AlertOps](./media/alertops-tutorial/configure1.png)

5. En la página **Subscription Settings** (Configuración de la suscripción), seleccione **SSO** y realice los pasos siguientes:

    ![Configuración de AlertOps](./media/alertops-tutorial/configure2.png)

    a. Active la casilla **Utilizar Inicio de sesión único (SSO)** .

    b. Seleccione **Azure Active Directory** como **proveedor de SSO** en la lista desplegable.

    c. En el cuadro de texto **URL del emisor**, use el identificador de valor que ha utilizado en la sección **Configuración básica de SAML** en Azure Portal.

    d. En el cuadro de texto **SAML endpoint URL** (Dirección URL del punto de conexión de SAML), pegue el valor de **Login URL** (Dirección URL de inicio de sesión) que ha copiado de Azure Portal.

    e. En el cuadro de texto **SLO endpoint URL** (Dirección URL del punto de conexión de SLO), pegue el valor de **Login URL** (Dirección URL de inicio de sesión) que ha copiado de Azure Portal.

    f. Seleccione **SHA256** como **algoritmo de firma de SAML** en el menú desplegable.

    g. Abra el archivo Certificado (Base64) que ha descargado en el Bloc de notas. Copie el contenido del archivo en el portapapeles y, después, péguelo en el cuadro de texto Certificado X.509.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado Britta Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `Britta Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `BrittaSimon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a AlertOps.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **AlertOps**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En la lista de usuarios del cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-alertops-test-user"></a>Creación de usuario de prueba de AlertOps

1. En otra ventana del explorador, inicie sesión en su sitio de la compañía de AlertOps como administrador.

2. Haga clic en **Usuarios** en el panel de navegación izquierdo.

    ![Configuración de AlertOps](./media/alertops-tutorial/user1.png)

3. Seleccione **Agregar usuario**.

    ![Configuración de AlertOps](./media/alertops-tutorial/user2.png)

4. En el cuadro de diálogo **Agregar usuario** , realice los pasos siguientes:

    ![Configuración de AlertOps](./media/alertops-tutorial/user3.png)

    a. En el cuadro de texto **Login User Name** (Nombre de usuario de inicio de sesión), escriba el nombre completo del usuario, por ejemplo, **BrittaSimon**.

    b. En el cuadro de texto **Official Email** (Correo electrónico oficial), escriba la dirección de correo electrónico del usuario, como **Brittasimon\@contoso.com**.

    c. En el cuadro de texto **First Name** (Nombre), escriba el nombre de usuario, en este caso **Britta**.

    d. En el cuadro de texto **Apellidos**, escriba el apellido del usuario, como **Simon**.

    e. Seleccione el valor **Tipo** en el menú desplegable según su organización.

    f. Seleccione el valor **Rol** del usuario en el menú desplegable según su organización.

    g. Seleccione **Agregar**.

### <a name="test-sso"></a>Prueba de SSO

Al seleccionar el icono de AlertOps en el panel de acceso, debería iniciar sesión automáticamente en la versión de AlertOps para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)