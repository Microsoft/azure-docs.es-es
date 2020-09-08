---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Firmex VDR | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Firmex VDR.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/21/2020
ms.author: jeedes
ms.openlocfilehash: ab480dd8dae00cd48811473a7b335291fb6300eb
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88555026"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-firmex-vdr"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Firmex VDR

En este tutorial aprenderá a integrar Firmex VDR con Azure Active Directory (Azure AD). Al integrar Firmex VDR con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Firmex VDR.
* Permitir que los usuarios inicien sesión automáticamente en Firmex VDR con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Firmex VDR.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Firmex VDR admite SSO iniciado por **SP e IDP**

* Una vez configurado Firmex VDR, puede aplicar controles de sesión que protejan la filtración e infiltración de la información confidencial de la organización en tiempo real. Los controles de sesión proceden del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-firmex-vdr-from-the-gallery"></a>Adición de Firmex VDR desde la galería

Para configurar la integración de Firmex VDR en Azure AD, será preciso que agregue Firmex VDR desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Firmex VDR** en el cuadro de búsqueda.
1. Seleccione **Firmex VDR** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-single-sign-on-for-firmex-vdr"></a>Configuración y prueba del inicio de sesión único de Azure AD para Firmex VDR

Configure y pruebe el inicio de sesión único de Azure AD con Firmex VDR con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Firmex VDR.

Para configurar y probar el inicio de sesión único de Azure AD con Firmex VDR, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Firmex VDR](#configure-firmex-vdr-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba de Firmex VDR](#create-firmex-vdr-test-user)** : para tener un homólogo de B.Simon en Firmex VDR que esté vinculado con la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Firmex VDR**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, el usuario no tiene que realizar ningún paso porque la aplicación ya se ha integrado previamente con Azure.

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL: `https://login.firmex.com`

1. Haga clic en **Save**(Guardar).

1. La aplicación Firmex VDR espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación Firmex VDR espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

    | Nombre | Atributo de origen|
    | ------------ | --------- |
    | email | user.mail |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar Firmex VDR**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a conceder a B.Simon acceso a Firmex VDR mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Firmex VDR**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-firmex-vdr-sso"></a>Configuración del inicio de sesión único de Firmex VDR

### <a name="before-you-get-started"></a>Antes de comenzar

#### <a name="what-youll-need"></a>Qué necesita

-   Una suscripción de Firmex activa.
-   Azure AD como servicio de inicio de sesión único.
-   Su administrador de TI para configurar el inicio de sesión único.
-   Una vez habilitado el inicio de sesión único, todos los usuarios de la empresa deben iniciar sesión en Firmex con el inicio de sesión único y no usar un nombre de usuario y contraseña.

#### <a name="how-long-will-this-take"></a>¿Cuánto tardará esto?

La implementación del inicio de sesión único tarda unos minutos. No hay prácticamente ningún tiempo de inactividad entre el momento en que se habilita el inicio de sesión único en Firmex para su sitio y el momento en que los usuarios de la empresa se autentican mediante inicio de sesión único. Siga los pasos que se indican a continuación:

### <a name="step-1---identify-your-companys-domains"></a>Paso 1: Identificación de los dominios de la empresa

Identifique los dominios con los que los usuarios de su empresa inician sesión.

Por ejemplo:

- @firmex.com
- @firmex.ca

### <a name="step-2---contact-firmex-support-with-your-domains"></a>Paso 2: Contacto con el soporte técnico de Firmex con sus dominios

Envíe un correo electrónico al [equipo de soporte técnico de Firmex](mailto:support@firmex.com) o llámelos al 1888 688 4042 x.11. Páseles la información de su dominio. El equipo de soporte técnico de Firmex agregará los dominios a su VDR como **dominios notificados**. El administrador debe configurar ahora el inicio de sesión único.

Advertencia: Hasta que el administrador del sitio configure los dominios notificados, los usuarios de la empresa no podrán iniciar sesión en la aplicación VDR. Los usuarios ajenos a la empresa (es decir, los usuarios invitados) pueden iniciar sesión con su correo electrónico y contraseña. La configuración tardará unos minutos.

### <a name="step-3---configure-the-claimed-domains"></a>Paso 3: Configuración de los dominios notificados

1. Inicie sesión en Firmex como administrador del sitio.
1. En la esquina superior izquierda, haga clic en el logotipo de su empresa.
1. Seleccione la pestaña **SSO**. A continuación, seleccione **SSO Configuration** (Configuración de SSO). Haga clic en el dominio que desea configurar.

    ![Dominios notificados](./media/firmex-vdr-tutorial/edit-sso.png)  

1. Haga que el administrador de TI rellene los campos siguientes. Los valores de los campos se deben tomar del proveedor de identidades:  

    ![SSO Configuration](./media/firmex-vdr-tutorial/SSO-config.png)

    a. En el cuadro de texto **Identificador de entidad**, pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    b. En el cuadro de texto **Identity Provider URL** (Dirección URL del proveedor de identidades), pegue el valor de **Login URL** (Dirección URL de inicio de sesión) que ha copiado de Azure Portal.

    c. **Public Key Certificate** (Certificado de clave pública): el emisor puede firmar digitalmente un mensaje SAML con fines de autenticación. Para comprobar la firma del mensaje, el receptor del mensaje utiliza una clave pública que se sabe que pertenece al emisor. Del mismo modo, para cifrar un mensaje, el emisor debe conocer una clave de cifrado pública que pertenezca al receptor final. En ambos casos (la firma y el cifrado), las claves públicas de confianza deben compartirse de antemano.  Este es el **certificado X509** de **XML de metadatos de federación**.

    d. Haga clic en **Guardar** para completar la configuración de SSO. Los cambios surten efecto inmediatamente.

1. En este momento, SSO está habilitado para el sitio.

### <a name="create-firmex-vdr-test-user"></a>Creación de un usuario de prueba de Firmex VDR

En esta sección, creará un usuario llamado B.Simon en Firmex VDR. Colabore con el [equipo de soporte técnico de Firmex](mailto:support@firmex.com) para agregar los usuarios a la plataforma de Firmex. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Firmex VDR en el panel de acceso, iniciará sesión automáticamente en la instancia de Firmex VDR para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de Firmex VDR con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Firmex con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
