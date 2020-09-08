---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con NS1 SSO for Azure | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y NS1 SSO for Azure.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/12/2020
ms.author: jeedes
ms.openlocfilehash: d0fee044506a9a19e09478ef8d70b3719ecc167a
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88554267"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-ns1-sso-for-azure"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con NS1 SSO for Azure

En este tutorial, aprenderá a integrar NS1 SSO for Azure con Azure Active Directory (Azure AD). Al integrar NS1 SSO for Azure con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a NS1 SSO for Azure.
* Permitir que los usuarios inicien sesión automáticamente en NS1 SSO for Azure con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central, Azure Portal.

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en NS1 SSO for Azure.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* NS1 SSO for Azure admite el inicio de sesión único iniciado por SP e IDP.
* Después de configurar NS1 SSO for Azure, puede aplicar el control de sesión. Esto protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).


## <a name="add-ns1-sso-for-azure-from-the-gallery"></a>Adición de NS1 SSO for Azure desde la galería

Para configurar la integración de NS1 SSO for Azure en Azure AD, es preciso agregar NS1 SSO for Azure desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **NS1 SSO for Azure** en el cuadro de búsqueda.
1. Seleccione **NS1 SSO for Azure** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-single-sign-on-for-ns1-sso-for-azure"></a>Configuración y prueba del inicio de sesión único de Azure AD para NS1 SSO for Azure

Configure y pruebe el inicio de sesión único de Azure AD con NS1 SSO for Azure mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de NS1 SSO for Azure.

Estos son los pasos generales para configurar y probar el inicio de sesión único de Azure AD con NS1 SSO for Azure:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.

    a. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.

    b. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en NS1 SSO for Azure](#configure-ns1-sso-for-azure-sso)** para configurar los valores de inicio de sesión único en la aplicación.

    a. **[Creación de un usuario de prueba de NS1 SSO for Azure](#create-an-ns1-sso-for-azure-test-user)** para tener un homólogo de B.Simon en NS1 SSO for Azure. Este homólogo está vinculado a la representación del usuario en Azure AD.
1. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **NS1 SSO for Azure**, busque la sección **Administrar**. Seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, seleccione el icono con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Captura de pantalla de la página Configurar el inicio de sesión único con SAML, con el icono de lápiz resaltado](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si quiere configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba la siguiente dirección URL: `https://api.nsone.net/saml/metadata`.

    b. En el cuadro de texto **Dirección URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://api.nsone.net/saml/sso/<ssoid>`.

1. Seleccione **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **Dirección URL de inicio de sesión**, escriba la siguiente dirección URL: `https://my.nsone.net/#/login/sso`.

    > [!NOTE]
    > El valor de la dirección URL de respuesta no es real. Actualice la dirección URL de respuesta con la dirección URL de respuesta real. Póngase en contacto con el [equipo de soporte técnico al cliente de NS1 SSO for Azure](mailto:techops@nsone.net) para obtener el valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación NS1 SSO for Azure espera las aserciones de SAML en un formato específico. Configure las siguientes notificaciones para esta aplicación. Puede administrar los valores de estos atributos en la sección **Atributos y notificaciones del usuario** de la página de integración de aplicaciones. En la página **Configurar inicio de sesión único con SAML**, seleccione el icono de lápiz para abrir el cuadro de diálogo **Atributos de usuario**.

    ![Captura de pantalla de la sección Atributos y notificaciones del usuario, con el icono de lápiz resaltado](./media/ns1-sso-for-azure-tutorial/attribute-edit-option.png)

1. Seleccione el nombre del atributo para editar la notificación.

    ![Captura de pantalla de la sección Atributos y notificaciones del usuario, con el nombre de atributo resaltado](./media/ns1-sso-for-azure-tutorial/attribute-claim-edit.png)

1. Seleccione **Transformación**.

    ![Captura de pantalla de la sección Administrar notificación, con la opción Transformación resaltada](./media/ns1-sso-for-azure-tutorial/prefix-edit.png)

1. En la sección **Administrar transformación**, lleve a cabo los pasos siguientes:

    ![Captura de pantalla de la sección Administrar transformación, con varios campos resaltados](./media/ns1-sso-for-azure-tutorial/prefix-added.png)

    1. Seleccione **ExactMailPrefix ()** en **Transformación**.

    1. Seleccione **user.userprincipalname** en **Parámetro 1**.

    1. Seleccione **Agregar**.

    1. Seleccione **Guardar**.

1. En la página **Configuración del inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione el botón de copia. Esto copia la **Dirección URL de metadatos de federación de la aplicación** y la guarda en el equipo.

    ![Captura de pantalla de la sección Certificado de firma de SAML, con el botón de copia resaltado](common/copy-metadataurl.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory** > **Usuarios** > **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:

   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Seleccione la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el campo **Contraseña**.
   1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección va a permitir que B.Simon acceda a NS1 SSO for Azure mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **NS1 SSO for Azure**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Captura de pantalla de la sección Administrar, con la opción Usuarios y grupos resaltada](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. En el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Captura de pantalla de la página Usuarios y grupos, con Agregar usuario resaltado](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** en la lista de usuarios. A continuación, elija el botón **Seleccionar** situado en la parte inferior de la pantalla.
1. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. A continuación, elija el botón **Seleccionar** situado en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

## <a name="configure-ns1-sso-for-azure-sso"></a>Configuración del inicio de sesión único en NS1 SSO for Azure

Para configurar el inicio de sesión único en NS1 SSO for Azure, es preciso enviar la dirección URL de metadatos de federación de la aplicación al [equipo de soporte técnico de NS1 SSO for Azure](mailto:techops@nsone.net). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-an-ns1-sso-for-azure-test-user"></a>Creación de un usuario de prueba en NS1 SSO for Azure

En esta sección, va a crear un usuario llamado B.Simon en NS1 SSO for Azure. Colabore con el equipo de soporte técnico de NS1 SSO for Azure para agregar los usuarios en la plataforma de NS1 SSO for Azure. No puede usar el inicio de sesión único hasta que cree y active los usuarios.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al seleccionar el icono de NS1 SSO for Azure en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de NS1 SSO for Azure para la que configuró el inicio de sesión único. Para más información, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe NS1 SSO for Azure con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)