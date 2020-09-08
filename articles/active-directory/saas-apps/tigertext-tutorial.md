---
title: 'Tutorial: Integración de Azure Active Directory con TigerConnect Secure Messenger | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y TigerConnect Secure Messenger.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/21/2020
ms.author: jeedes
ms.openlocfilehash: 995cd8470d4fbbf3dc340139a86275593a3d5d28
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88815260"
---
# <a name="tutorial-azure-active-directory-integration-with-tigerconnect-secure-messenger"></a>Tutorial: Integración de Azure Active Directory con TigerConnect Secure Messenger

En este tutorial, obtendrá información sobre cómo integrar TigerConnect Secure Messenger con Azure Active Directory (Azure AD).

La integración de TigerConnect Secure Messenger con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a TigerConnect Secure Messenger.
* Puede habilitar a los usuarios para que inicien sesión automáticamente en TigerConnect Secure Messenger (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Para más información sobre la integración de aplicaciones de software como servicio (SaaS) con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con TigerConnect Secure Messenger, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Una suscripción habilitada para el inicio de sesión único en TigerConnect Secure Messenger.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba e integrar TigerConnect Secure Messenger con Azure AD.

* TigerConnect Secure Messenger admite el inicio de sesión único iniciado por el **proveedor de servicios**.
* Una vez configurado TigerConnect Secure Messenger, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-tigerconnect-secure-messenger-from-the-gallery"></a>Adición de TigerConnect Secure Messenger desde la galería

Para configurar la integración de TigerConnect Secure Messenger en Azure AD, será preciso que lo agregue desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **TigerConnect Secure Messenger**  en el cuadro de búsqueda.
1. Seleccione **TigerConnect Secure Messenger** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso"></a>Configuración y prueba del inicio de sesión único de Azure AD

En esta sección, puede configurar y probar el inicio de sesión único de Azure AD con TigerConnect Secure Messenger con un usuario de prueba llamado **Britta Simon**. Para que el inicio de sesión único funcione, es preciso establecer un vínculo entre un usuario de Azure AD y el usuario correspondiente de TigerConnect Secure Messenger.

Para configurar y probar el inicio de sesión único de Azure AD con TigerConnect Secure Messenger, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en TigerConnect Secure Messenger](#configure-tigerconnect-secure-messenger-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba de TigerConnect Secure Messenger](#create-a-tigerconnect-secure-messenger-test-user)** , para tener un usuario llamado Britta Simon en TigerConnect Secure Messenger que esté vinculado con el usuario de Azure AD llamado Britta Simon.
1. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con TigerConnect Secure Messenger, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **TigerConnect Secure Messenger**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    1. En el cuadro **URL de inicio de sesión**, escriba una dirección URL:

       `https://home.tigertext.com`

    1. En el cuadro de texto **Identificador (Id. de entidad)**, escriba una dirección URL con el siguiente patrón:

       `https://saml-lb.tigertext.me/v1/organization/<instance ID>`

    > [!NOTE]
    > El valor **Identificador (Id. de entidad)** no es real. Actualícelo con el identificador real. Para obtener este valor póngase en contacto con el [equipo de soporte técnico de TigerConnect Secure Messenger](mailto:prosupport@tigertext.com). También puede hacer referencia a los patrones que se muestran en el panel **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas y guárdelo en el equipo.

    ![Opción de descarga del XML de metadatos de federación](common/metadataxml.png)

1. En la sección **Configurar TigerConnect Secure Messenger**, copie las direcciones URL según sus necesidades.

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

En esta sección, podrá habilitar a B.Simon para que use el inicio de sesión único de Azure concediéndole acceso a TigerConnect Secure Messenger.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **TigerConnect Secure Messenger**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-tigerconnect-secure-messenger-sso"></a>Configuración del inicio de sesión único de TigerConnect Secure Messenger

Para configurar el inicio de sesión único en TigerConnect Secure Messenger, tiene que enviar el XML de metadatos de federación descargado y las direcciones URL apropiadas copiadas de Azure Portal al [equipo de soporte técnico de TigerConnect Secure Messenger](mailto:prosupport@tigertext.com). El equipo de TigerConnect Secure Messenger se asegurará de que la conexión de inicio de sesión único de SAML esté establecida correctamente en ambos lados.

## <a name="create-a-tigerconnect-secure-messenger-test-user"></a>Creación de un usuario de prueba de TigerConnect Secure Messenger

En esta sección, creará un usuario llamado Britta Simon en TigerConnect Secure Messenger. Colabore con el [equipo de soporte técnico de TigerConnect Secure Messenger](mailto:prosupport@tigertext.com) para agregar a Britta Simon como usuario en TigerConnect Secure Messenger. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el portal Aplicaciones.

Al seleccionar **TigerConnect Secure Messenger** en el portal Aplicaciones, debería iniciar sesión automáticamente en la suscripción de TigerConnect Secure Messenger para la que configuró el inicio de sesión único. Para más información acerca del portal Aplicaciones, consulte [Access and use apps on the My Apps portal](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction) (Uso de aplicaciones y acceso a ellas en el portal Aplicaciones).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe TigerConnect Secure Messenger con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)