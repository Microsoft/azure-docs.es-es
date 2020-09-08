---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con TINFOIL SECURITY | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y TINFOIL SECURITY.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/16/2019
ms.author: jeedes
ms.openlocfilehash: 35ff822b4439a48a922fe5ccd4530dfb5d40ffae
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88552046"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-tinfoil-security"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con TINFOIL SECURITY

En este tutorial, aprenderá a integrar TINFOIL SECURITY con Azure Active Directory (Azure AD). Al integrar TINFOIL SECURITY con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a TINFOIL SECURITY.
* Permitir que los usuarios inicien sesión automáticamente en TINFOIL SECURITY con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en TINFOIL SECURITY.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* TINFOIL SECURITY admite el inicio de sesión único iniciado por **IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-tinfoil-security-from-the-gallery"></a>Adición de TINFOIL SECURITY desde la galería

Para configurar la integración de TINFOIL SECURITY en Azure AD, debe agregar TINFOIL SECURITY desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **TINFOIL SECURITY** en el cuadro de búsqueda.
1. Seleccione **TINFOIL SECURITY** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-tinfoil-security"></a>Configuración y prueba del inicio de sesión único de Azure AD para TINFOIL SECURITY

Configure y pruebe el inicio de sesión único de Azure AD con TINFOIL SECURITY con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de TINFOIL SECURITY.

Para configurar y probar el inicio de sesión único de Azure AD con TINFOIL SECURITY, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de TINFOIL SECURITY](#configure-tinfoil-security-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba de TINFOIL SECURITY](#create-tinfoil-security-test-user)** : para tener un homólogo de B.Simon en TINFOIL SECURITY que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **TINFOIL SECURITY**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, la aplicación está preconfigurada y las direcciones URL necesarias ya se han rellenado previamente con Azure. El usuario debe guardar la configuración, para lo que debe hacer clic en el botón **Guardar**.

1. La aplicación Visitly espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación Visitly espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

    | Nombre | Atributo de origen |
    | ------------------- | -------------|
    | accountid | UXXXXXXXXXXXXX |

    > [!NOTE]
    > El valor de accountid se explica más adelante en el tutorial.

1. En la sección **Certificado de firma de SAML**, haga clic en el botón **Editar** para abrir el cuadro de diálogo **Certificado de firma de SAML**.

    ![Edición del certificado de firma de SAML](common/edit-certificate.png)

1. En la sección **Certificado de firma de SAML**, copie el valor de **Huella digital** y guárdelo en el equipo.

    ![Copia del valor de la huella digital](common/copy-thumbprint.png)

1. En la sección **Configurar TINFOIL SECURITY**, copie las direcciones URL adecuadas según sus requisitos.

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

En esta sección, habilitará a B.Simon para que use el inicio de sesión único de Azure concediéndole acceso a TINFOIL SECURITY.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **TINFOIL SECURITY**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-tinfoil-security-sso"></a>Configuración del inicio de sesión único de TINFOIL SECURITY

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la compañía de TINFOIL SECURITY.

1. En la barra de herramientas de la parte superior, haga clic en el icono de **Mi cuenta**.

    ![Panel](./media/tinfoil-security-tutorial/ic798971.png "Panel")

1. Haga clic en **Seguridad**.

    ![Seguridad](./media/tinfoil-security-tutorial/ic798972.png "Seguridad")

1. Siga estos pasos en la página de configuración **Inicio de sesión único** :

    ![Inicio de sesión único](./media/tinfoil-security-tutorial/ic798973.png "Inicio de sesión único")

    a. Seleccione **Habilitar SAML**.

    b. Haga clic en **Configuración manual**.

    c. En el cuadro de texto **Dirección URL de publicación de SAML**, pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    d. En el cuadro de texto **SAML Certificate Fingerprint** (Huella digital de certificado de SAML), pegue el valor de **Huella digital** del certificado que haya copiado de la sección **Certificado de firma de SAML**.
  
    e. Copie el valor del **identificador de cuenta** y péguelo en el cuadro de texto **Atributo de origen** de la sección **Atributos y notificaciones de usuario** de Azure Portal.

    f. Haga clic en **Save**(Guardar).

### <a name="create-tinfoil-security-test-user"></a>Creación de un usuario de prueba de TINFOIL SECURITY

Para permitir que los usuarios de Azure AD inicien sesión en TINFOIL SECURITY, deben aprovisionarse en TINFOIL SECURITY. En el caso de TINFOIL SECURITY, el aprovisionamiento es una tarea manual.

**Siga estos pasos para que se aprovisione un usuario:**

1. Si el usuario forma parte de una cuenta de empresa, deberá [ponerse en contacto con el equipo de soporte técnico de TINFOIL SECURITY](https://www.tinfoilsecurity.com/contact) para que se cree la cuenta del usuario.

1. Si el usuario es usuario habitual de SaaS de TINFOIL SECURITY, puede agregar un colaborador a cualquiera de sus sitios. Esta acción desencadena un proceso para enviar una invitación al correo electrónico especificado y crear una cuenta de usuario de TINFOIL SECURITY.

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de TINFOIL SECURITY que ofrezca TINFOIL SECURITY para aprovisionar cuentas de usuario de Azure AD.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de TINFOIL SECURITY en el panel de acceso, debería iniciar sesión automáticamente en la instancia de TINFOIL SECURITY para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe TINFOIL SECURITY con Azure AD](https://aad.portal.azure.com/)