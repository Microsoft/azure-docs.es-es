---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con BeyondTrust Remote Support | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y BeyondTrust Remote Support.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/30/2019
ms.author: jeedes
ms.openlocfilehash: ecaeea625bc4891c95614292a44d31b616d39eb2
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88549303"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-beyondtrust-remote-support"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con BeyondTrust Remote Support

En este tutorial aprenderá a integrar BeyondTrust Remote Support con Azure Active Directory (Azure AD). Al integrar BeyondTrust Remote Support con Azure AD, podrá:

* Controlar en Azure AD quién tiene acceso a BeyondTrust Remote Support.
* Permitir que los usuarios inicien sesión automáticamente en BeyondTrust Remote Support con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en BeyondTrust Remote Support.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* BeyondTrust Remote Support admite el inicio de sesión único iniciado por **SP**
* BeyondTrust Remote Support admite el aprovisionamiento de usuarios **Just-In-Time**

## <a name="adding-beyondtrust-remote-support-from-the-gallery"></a>Incorporación de BeyondTrust Remote Support desde la galería

Para configurar la integración de BeyondTrust Remote Support en Azure AD, es preciso agregarla desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **BeyondTrust Remote Support** en el cuadro de búsqueda.
1. Seleccione **BeyondTrust Remote Support** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-beyondtrust-remote-support"></a>Configuración y prueba del inicio de sesión único de Azure AD para BeyondTrust Remote Support

Configure y pruebe el inicio de sesión único de Azure AD con BeyondTrust Remote Support mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de BeyondTrust Remote Support.

Para configurar y probar el inicio de sesión único de Azure AD con BeyondTrust Remote Support, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en BeyondTrust Remote Support](#configure-beyondtrust-remote-support-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba en BeyondTrust Remote Support](#create-beyondtrust-remote-support-test-user)** : para tener un homólogo de B.Simon en BeyondTrust Remote Support vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **BeyondTrust Remote Support**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<HOSTNAME>.bomgar.com/saml`

    b. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón: `https://<HOSTNAME>.bomgar.com`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<HOSTNAME>.bomgar.com/saml/sso`

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de inicio de sesión, el identificador y la dirección URL de respuesta reales. Estos valores se explican más adelante en el tutorial.

1. La aplicación BeyondTrust Remote Support espera las aserciones de SAML en un formato específico, lo cual requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, BeyondTrust Remote Support espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

    | Nombre |  Atributo de origen|
    | ---------------| ----------|
    | Nombre de pila | user.givenname |
    | Dirección de correo electrónico | user.mail |
    | Nombre | user.userprincipalname |
    | Nombre de usuario | user.userprincipalname |
    | Grupos | user.groups |
    | Identificador de usuario único | user.userprincipalname |

    > [!NOTE]
    > Al asignar grupos de Azure AD para la aplicación BeyondTrust Remote Support, la opción "Grupos devueltos en la notificación" deberá modificarse de Ninguno a SecurityGroup. Los grupos se importarán en la aplicación con sus identificadores de objeto. El identificador de objeto del grupo de Azure AD se encuentra al comprobar las propiedades en la interfaz de Azure Active Directory. Esto será necesario para hacer referencia a los grupos de Azure AD y asignar estos a las directivas de grupo correctas.

1. Al establecer el identificador de usuario único, este valor debe establecerse en NameID-Format: **Persistente**. Es necesario que este sea un identificador persistente para identificar correctamente y asociar al usuario a las directivas de grupo correctas para los permisos. Haga clic en el icono de edición para abrir el cuadro de diálogo **Atributos y reclamaciones del usuario** para editar el valor de identificador de usuario único.

1. En la sección **Administración de la notificación**, haga clic en **Elija el formato del identificador de nombre**, establezca el valor en **Persistente** y haga clic en **Guardar**.

    ![Atributos y reclamaciones del usuario](./media/bomgarremotesupport-tutorial/attribute-unique-user-identifier.png)

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Set up BeyondTrust Remote Support** (Configurar BeyondTrust Remote Support), copie las direcciones URL que necesite.

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

En esta sección va a permitir que B.Simon acceda a BeyondTrust Remote Support mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **BeyondTrust Remote Support**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-beyondtrust-remote-support-sso"></a>Configuración del inicio de sesión único en BeyondTrust Remote Support

1. En otra ventana del explorador web, inicie sesión en el inquilino de BeyondTrust Remote Support como administrador.

1. Haga clic en el menú **STATUS** (ESTADO) y copie los valores de **identifier** (Identificador), **Reply URL** (URL de respuesta) y **Sign on URL** (URL de inicio de sesión), y úselos en la sección **Configuración básica de SAML** de Azure Portal.

    ![Configuración de BeyondTrust Remote Support](./media/bomgarremotesupport-tutorial/config-url-values.png)

1. Vaya a la interfaz /login de BeyondTrust Remote Support de `https://support.example.com/login`, donde **support.example.com** es el nombre de host principal de su dispositivo, y realice la autenticación con sus credenciales administrativas.

1. Vaya a **Users & Security** > **Security Providers** (Usuarios y seguridad > Proveedores de seguridad).

1. En el menú desplegable, seleccione **SAML** y haga clic en el botón **Create Provider** (Crear proveedor).

1. En la sección Identity Provider Settings (Configuración del proveedor de identidades) hay una opción para cargar los metadatos del proveedor de identidades. Busque el archivo XML de metadatos que ha descargado de Azure Portal y haga clic en el botón **Upload** (Cargar). Los valores de **Entity ID** (Id. de entidad), **Single Sign-On Service URL** (URL de servicio de inicio de sesión único) y el certificado se cargarán automáticamente y **Protocol Binding** (Enlace de protocolo) deberá cambiarse a **HTTP POST**. Consulte la captura de pantalla siguiente:

    ![Configuración de BeyondTrust Remote Support](./media/bomgarremotesupport-tutorial/config-uploadfile.png)

### <a name="create-beyondtrust-remote-support-test-user"></a>Creación de un usuario de prueba en BeyondTrust Remote Support

Aquí se configurarán las opciones de aprovisionamiento de usuarios. Se hará referencia a los valores de esta sección desde la sección **Atributos y reclamaciones del usuario** de Azure Portal. Los hemos configurado como los valores predeterminados que ya se han importado en el momento de la creación; no obstante, se pueden personalizar.

![Creación del usuario](./media/bomgarremotesupport-tutorial/config-user1.png)

> [!NOTE]
> Los grupos y el atributo e-mail no son necesarios para esta implementación. Si utiliza grupos de Azure AD y los asigna a las directivas de grupo de BeyondTrust Remote Support para los permisos, será necesario hacer referencia al identificador de objeto del grupo a través de sus propiedades en Azure Portal y colocarlo en la sección "Available Groups" (Grupos disponibles). Cuando haya terminado, el identificador de objeto o el grupo de AD estarán ya disponibles para su asignación a una directiva de grupo para los permisos.

![Creación del usuario](./media/bomgarremotesupport-tutorial/config-user2.png)

![Creación del usuario](./media/bomgarremotesupport-tutorial/config-user3.png)

> [!NOTE]
> Como alternativa, se puede establecer una directiva de grupo predeterminada en el proveedor de seguridad SAML2. Al definir esta opción, se asignarán los permisos especificados en la directiva de grupo a todos los usuarios que se autentiquen mediante SAML. La directiva General Members (Miembros generales) está incluida en BeyondTrust Remote Support/Privileged Remote Access (Acceso privilegiado remoto) con permisos limitados y se puede usar para probar la autenticación y asignar usuarios a las directivas correctas. Los usuarios no se rellenarán en la lista de usuarios de SAML2 a través de /login > Users & Security (Usuarios y seguridad) hasta el primer intento de autenticación correcto. Puede encontrar más información sobre las directivas de grupo en el vínculo siguiente: `https://www.beyondtrust.com/docs/remote-support/getting-started/admin/group-policies.htm`

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de BeyondTrust Remote Support en el panel de acceso, debería iniciar sesión automáticamente en la versión de BeyondTrust Remote Support en la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de BeyondTrust Remote Support con Azure AD](https://aad.portal.azure.com/)
