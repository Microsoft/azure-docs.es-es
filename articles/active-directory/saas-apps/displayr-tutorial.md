---
title: 'Tutorial: Integración de Azure Active Directory con Displayr | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Displayr.
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
ms.openlocfilehash: a69b4a068e62c6376660a79b3a7c8ab5402b5704
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88536264"
---
# <a name="tutorial-integrate-displayr-with-azure-active-directory"></a>Tutorial: Integración de Displayr con Azure Active Directory

En este tutorial, aprenderá a integrar Displayr con Azure Active Directory (Azure AD). Al integrar Displayr con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Displayr.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Displayr con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Displayr.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. Displayr admite el inicio de sesión único iniciado por **SP**.

## <a name="adding-displayr-from-the-gallery"></a>Adición de Displayr desde la galería

Para configurar la integración de Displayr en Azure AD, será preciso que agregue Displayr desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Displayr** en el cuadro de búsqueda.
1. Seleccione **Displayr** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

Configure y pruebe el inicio de sesión único de Azure AD con Displayr con un usuario de prueba llamado **Britta Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Displayr.

Para configurar y probar el inicio de sesión único de Azure AD con Displayr, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
2. **[Configuración de Displayr](#configure-displayr)** , para configurar las opciones del SSO en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba en Displayr](#create-displayr-test-user)** para tener un homólogo de Britta Simon en Displayr que esté vinculado a su representación en Azure AD.
6. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Displayr**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<YOURDOMAIN>.displayr.com`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón:`<YOURDOMAIN>.displayr.com`

    >[!NOTE]
    >Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de Displayr](mailto:support@displayr.com) para obtener estos valores. También puede consultar los patrones que se muestran en la sección Configuración básica de SAML de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

   ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. La aplicación Displayr espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados. Haga clic en el icono **Editar** para abrir el cuadro de diálogo Atributos de usuario.

    ![imagen](common/edit-attribute.png)

1. Además de lo anterior, la aplicación Displayr espera que se usen algunos atributos más en la respuesta de SAML. En la sección **User Attributes & Claims** (Atributos y notificaciones del usuario) del cuadro de diálogo **Group Claims (Preview)** (Notificaciones de grupo [versión preliminar]), siga estos pasos:

    a. Haga clic en el **lápiz** junto a **Groups returned in claim** (Grupos devueltos en la notificación).

    ![imagen](./media/displayr-tutorial/config04.png)

    ![imagen](./media/displayr-tutorial/config05.png)

    b. Seleccione **Todos los grupos** en la lista de selección.

    c. Seleccione **Atributo de origen** de **Id. de grupo**.

    d. Marque **Personalizar nombre de la notificación del grupo**.

    e. Marque **Emitir grupos como notificaciones de roles**.

    f. Haga clic en **Save**(Guardar).

1. En la sección **Configurar Displayr**, copie las direcciones URL adecuadas según sus necesidades.

   ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="configure-displayr"></a>Configuración de Displayr

1. Para automatizar la configuración en Displayr, es preciso instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Configurar Displayr** para ir a la aplicación Displayr. En ella, escriba las credenciales de administrador para iniciar sesión en Displayr. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 6.

    ![Configuración](common/setup-sso.png)

3. Si desea configurar Displayr manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Displayr como administrador y haga lo siguiente:

4. Haga clic en **Settings** (Configuración) y, después, vaya a **Account** (Cuenta).

    ![Configuración](./media/displayr-tutorial/config01.png)

5. Cambie a **Settings** (Configuración) en el menú superior y desplácese hacia abajo en la página para hacer clic en **Configure Single Sign On (SAML)** [Configurar inicio de sesión único (SAML)].

    ![Configuración](./media/displayr-tutorial/config02.png)

6. Siga estos pasos en la página **Single Sign On (SAML)** [Inicio de sesión único (SAML)]:

    ![Configuración](./media/displayr-tutorial/config03.png)

    a. Seleccione la casilla **Enable Single Sign On (SAML)** [Habilitar inicio de sesión único (SAML)].

    b. Copie el valor real de **Identificador** de la sección **Configuración básica de SAML** de Azure AD y péguelo en el cuadro de texto **Issuer** (Emisor).

    c. En el cuadro de texto **Login URL** (Dirección URL de inicio de sesión), pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    d. En el cuadro de texto **Logout URL** (Dirección URL de cierre de sesión), pegue el valor de la **dirección URL de cierre de sesión** que ha copiado de Azure Portal.

    e. Abra el certificado (Base64) en el Bloc de notas, copie su contenido y péguelo en el cuadro de texto **Certificate** (Certificado).

    f. El campo **Group mappings** (Asignaciones de grupo) es opcional.

    g. Haga clic en **Save**(Guardar).  

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

En esta sección, se habilita a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Displayr.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Displayr**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En la lista de usuarios del cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-displayr-test-user"></a>Creación de un usuario de prueba de Displayr

Para permitir que los usuarios de Azure AD inicien sesión en Displayr, deben aprovisionarse en Displayr. En Displayr, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión como administrador en Displayr.

2. Haga clic en **Settings** (Configuración) y, después, vaya a **Account** (Cuenta).

    ![Configuración de Displayr](./media/displayr-tutorial/config01.png)

3. Cambie a **Settings** (Configuración) en el menú superior y desplácese hacia abajo en la página hasta la sección **Users** (Usuarios) y después haga clic en **New User** (Nuevo usuario).

    ![Configuración de Displayr](./media/displayr-tutorial/config07.png)

4. En la página **New User** (Nuevo usuario), realice estos pasos:

    ![Configuración de Displayr](./media/displayr-tutorial/config06.png)

    a. En el cuadro de texto **Name** (Nombre), escriba el nombre de usuario **Brittasimon**.

    b. En el cuadro de texto **E-mail** (Correo electrónico), escriba el correo electrónico del usuario, por ejemplo, `Brittasimon@contoso.com`.

    c. Seleccione la opción apropiada en **Group membership** (Pertenencia a grupos).

    d. Haga clic en **Save**(Guardar).

### <a name="test-sso"></a>Prueba de SSO

Al seleccionar el icono de Displayr en el panel de acceso, debería iniciar sesión automáticamente en la versión de Displayr para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
