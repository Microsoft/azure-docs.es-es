---
title: 'Tutorial: Integración de Azure Active Directory con Knowledge Anywhere LMS | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Knowledge Anywhere LMS.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/22/2019
ms.author: jeedes
ms.openlocfilehash: 507298f323fe14e3b114afebed5f2f258145a76e
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88549949"
---
# <a name="tutorial-integrate-knowledge-anywhere-lms-with-azure-active-directory"></a>Tutorial: Integración de Knowledge Anywhere LMS con Azure Active Directory

En este tutorial, aprenderá a integrar Knowledge Anywhere LMS.con Azure Active Directory (Azure AD). Al integrar Knowledge Anywhere LMS con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Knowledge Anywhere LMS.
* Permitir que los usuarios inicien sesión automáticamente en Knowledge Anywhere LMS con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para inicio de sesión único en Knowledge Anywhere LMS.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. Knowledge Anywhere LMS admite el SSO iniciado por **SP**, así como el aprovisionamiento de usuarios **Just In Time**.

## <a name="adding-knowledge-anywhere-lms-from-the-gallery"></a>Adición de Knowledge Anywhere LMS desde la galería

Para configurar la integración de Knowledge Anywhere LMS en Azure AD, será preciso que lo agregue desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Knowledge Anywhere LMS** en el cuadro de búsqueda.
1. Seleccione **Knowledge Anywhere LMS** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

Configure y pruebe el inicio de sesión único de Azure AD con Knowledge Anywhere LMS utilizando un usuario de prueba llamado **B. Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Knowledge Anywhere LMS.

Para configurar y probar el inicio de sesión único de Azure AD con Knowledge Anywhere LMS, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
2. **[Configuración de Knowledge Anywhere LMS](#configure-knowledge-anywhere-lms)** , para configurar las opciones del SSO en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B. Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B. Simon para que use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Knowledge Anywhere LMS](#create-knowledge-anywhere-lms-test-user)** , para tener un homólogo de B. Simon en Knowledge Anywhere LMS que esté vinculado a la representación del usuario en Azure AD.
6. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Knowledge Anywhere LMS**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice los siguientes pasos:

    1. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<CLIENTNAME>.knowledgeanywhere.com/`

    1. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<CLIENTNAME>.knowledgeanywhere.com/SSO/SAML/Response.aspx?<IDPNAME>`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL de respuesta y el identificador reales, que se explican más adelante en el tutorial.

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<CLIENTNAME>.knowledgeanywhere.com/`

    > [!NOTE]
    > El valor de la dirección URL de inicio de sesión no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico al cliente de Knowledge Anywhere LMS](https://knowany.zendesk.com/hc/en-us/articles/360000469034-SAML-2-0-Single-Sign-On-SSO-Set-Up-Guide) para obtener este valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

   ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Knowledge Anywhere LMS**, copie las direcciones URL adecuadas según sus necesidades.

   ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="configure-knowledge-anywhere-lms"></a>Configuración de Knowledge Anywhere LMS

1. Para automatizar la configuración en Knowledge Anywhere LMS, debe instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Configurar Knowledge Anywhere LMS** para ir a la aplicación Knowledge Anywhere LMS. En ella, escriba las credenciales de administrador para iniciar sesión en Knowledge Anywhere LMS. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 7.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar Knowledge Anywhere LMS manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Knowledge Anywhere LMS como administrador y haga lo siguiente:

4. Seleccione en la pestaña **Sitio**.

    ![Configuración de Knowledge Anywhere LMS](./media/knowledge-anywhere-lms-tutorial/configure1.png)

5. Seleccione en la pestaña **Configuración de SAML**.

    ![Configuración de Knowledge Anywhere LMS](./media/knowledge-anywhere-lms-tutorial/configure2.png)

6. Haga clic en **Agregar nuevo**.

    ![Configuración de Knowledge Anywhere LMS](./media/knowledge-anywhere-lms-tutorial/configure3.png)

7. En la página **Add/Update SAML Settings** (Agregar o actualizar la configuración de SAML), realice los pasos siguientes:

    ![Configuración de Knowledge Anywhere LMS](./media/knowledge-anywhere-lms-tutorial/configure4.png)

    a. Escriba el nombre de IDP según su organización. Por ejemplo, `Azure`.

    b. En el cuadro de texto **IDP Entity ID** (Identificador de entidad IDP), pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    c. En el cuadro de texto **IDP URL** (URL de IDP), pegue la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    d. Abra en el Bloc de notas el archivo de certificado que descargó de Azure Portal, copie el contenido y, luego, péguelo en el cuadro de texto **Certificado**.

    e. En el cuadro de texto **Logout URL** (Dirección URL de cierre de sesión), pegue la **dirección URL de cierre de sesión** que ha copiado de Azure Portal.

    f. Seleccione **Sitio principal** en la lista desplegable para el **dominio**.

    g. Copie el valor **Id. de entidad de SP** y péguelo en el cuadro de texto **Identificador** de la sección **Configuración básica de SAML** en Azure Portal.

    h. Copie el valor **SP Response(ACS) URL** [URL de respuesta (ACS) de SP] y péguelo en el cuadro de texto **URL de respuesta** de la sección **Configuración básica de SAML** en Azure Portal.

    i. Haga clic en **Save**(Guardar).

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B. Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B. Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `BrittaSimon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a B. Simon para que use el inicio de sesión único de Azure concediéndole acceso a Knowledge Anywhere LMS.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Knowledge Anywhere LMS**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B. Simon** en la lista de usuarios y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-knowledge-anywhere-lms-test-user"></a>Creación de un usuario de prueba de Knowledge Anywhere LMS

En esta sección, se crea un usuario llamado B. Simon en Knowledge Anywhere LMS. Knowledge Anywhere LMS admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en Knowledge Anywhere LMS, se crea uno nuevo después de la autenticación.

### <a name="test-sso"></a>Prueba de SSO

Al seleccionar el icono de Knowledge Anywhere LMS en el Panel de acceso, debería iniciar sesión automáticamente en la versión de Knowledge Anywhere LMS para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)