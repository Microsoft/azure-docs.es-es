---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con ekarda | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y ekarda.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/15/2020
ms.author: jeedes
ms.openlocfilehash: cfd7a007c8f26a96d929026e231cb412cdc70279
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89440888"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-ekarda"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con ekarda

En este tutorial aprenderá a integrar ekarda con Azure Active Directory (Azure AD). Al integrar ekarda con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a ekarda.
* Permitir que los usuarios inicien sesión automáticamente en ekarda con sus cuentas de Azure AD.
* Administrar sus cuentas en una ubicación central: Azure Portal.

Para más información sobre la integración de aplicaciones de software como servicio (SaaS) con Azure AD, consulte [¿Qué es el inicio de sesión único (SSO)?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción de ekarda habilitada para el inicio de sesión único (SSO).

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* ekarda admite el inicio de sesión único iniciado por SP e IDP.
* ekarda admite el aprovisionamiento de usuarios Just-In-Time.
* Después de configurar ekarda, puede aplicar el control de sesión. Esta precaución protege frente a la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del control de aplicaciones de acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="add-ekarda-from-the-gallery"></a>Adición de ekarda desde la galería

Para configurar la integración de ekarda en Azure AD, agregue ekarda desde la galería a la lista de aplicaciones SaaS administradas:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.

1. En el panel izquierdo, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una aplicación nueva, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **ekarda** en el cuadro de búsqueda.
1. Seleccione **ekarda** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-ekarda"></a>Configuración y prueba del inicio de sesión único de Azure AD para ekarda

Configure y pruebe el inicio de sesión único de Azure AD con ekarda mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculo entre un usuario de Azure AD y el usuario relacionado de ekarda.

Para configurar y probar el inicio de sesión único de Azure AD con ekarda, es preciso completar los siguientes pasos:

1. [Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso), para permitir que los usuarios puedan utilizar esta característica.

    1. [Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user), para probar el inicio de sesión único de Azure AD con B. Simon.
    1. [Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user), para que B. Simon pueda usar el inicio de sesión único de Azure AD.
1. [Configuración del inicio de sesión único en ekarda](#configure-ekarda-sso), para configurar los valores de inicio de sesión único en la aplicación.
    * [Creación de un usuario de prueba de ekarda](#create-an-ekarda-test-user), para tener un homólogo de B.Simon en ekarda que esté vinculado a la representación del usuario en Azure AD.
1. [Comprobación del inicio de sesión único](#test-sso), para verificar que la configuración funciona correctamente.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos en Azure Portal para habilitar el inicio de sesión único de Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. En la página de integración de la aplicación **ekarda**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, seleccione el icono con forma de lápiz para editar los valores de **Configuración básica de SAML**.

   ![Captura de pantalla de la página Configuración del inicio de sesión único con SAML con el icono de lápiz resaltado.](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si tiene el **archivo de metadatos del proveedor de servicios**, siga estos pasos:
    1. Seleccione **Cargar el archivo de metadatos**.
    1. Para seleccionar el archivo de metadatos, seleccione el icono de carpeta y, a continuación, seleccione **Cargar**.
    1. Cuando se haya cargado correctamente el archivo de metadatos, los valores de **Identificador** y **Dirección URL de respuesta** se rellenarán automáticamente en los cuadros de texto de la sección de ekarda.

    > [!Note]
    > Si los valores de **Identificador** y **Dirección URL de respuesta** no se rellenan automáticamente, proporcione dichos valores manualmente según sus necesidades.

1. Si no tiene el **archivo de metadatos del proveedor de servicios** en la sección **Configuración básica de SAML** y desea configurar la aplicación en modo iniciado por IDP, escriba los valores de los siguientes campos:

    1. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón: `https://my.ekarda.com/users/saml_metadata/<COMPANY_ID>`
    1. En el cuadro de texto **Dirección URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://my.ekarda.com/users/saml_acs/<COMPANY_ID>`

1. Seleccione **Establecer direcciones URL adicionales** si desea configurar la aplicación en el modo iniciado por SP y haga lo siguiente:

    * En el cuadro de texto **Dirección URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://my.ekarda.com/users/saml_sso/<COMPANY_ID>`

    > [!NOTE]
    > Los valores de los dos pasos anteriores no son reales. Actualícelos con valores reales de identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de ekarda](mailto:contact@ekarda.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione **Descargar** para guardar el **certificado (Base 64)** en el equipo.

    ![Captura de pantalla de la sección Certificado de firma de SAML de la página Configuración del inicio de sesión único con SAML, con el vínculo de descarga del certificado Base 64 resaltado.](common/certificatebase64.png)

1. En la sección **Configurar ekarda**, copie las direcciones URL adecuadas según sus necesidades.

    ![Captura de pantalla de la sección Configurar ekarda de la página Configuración del inicio de sesión único con SAML, con los vínculos de copia de dirección URL resaltados.](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, usará Azure Portal para crear un usuario de prueba llamado B.Simon.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory** > **Usuarios** > **Todos los usuarios**.

1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, escriba `B.Simon@contoso.com`:
   1. Seleccione la casilla **Mostrar contraseña** y, anote el valor que aparece en el cuadro **Contraseña**.
   1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir que B.Simon acceda a ekarda mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **ekarda**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Captura de pantalla de la sección Administrar, con la opción Usuarios y grupos resaltada.](common/users-groups-blade.png)

1. Seleccione **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Captura de pantalla de la sección Usuarios y grupos, con la opción Agregar usuario resaltada.](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** en la lista de usuarios. A continuación, elija **Seleccionar** en la parte inferior de la pantalla.
1. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. A continuación, elija **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

## <a name="configure-ekarda-sso"></a>Configuración del inicio de sesión único de ekarda

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la compañía en ekarda.
1. Seleccione **Admin** > **My Account** (Administración > Mi cuenta).

    ![Captura de pantalla de la interfaz de usuario del sitio de ekarda con Mi cuenta resaltada en el menú Administración.](./media/ekarda-tutorial/ekarda.png)

1. En la parte inferior de la página, busque la sección **SAML SETTINGS** (Configuración de SAML). En esta sección se configura la integración de SAML.
1. En la sección **SAML SETTINGS** (Configuración de SAML), siga estos pasos:

    ![Captura de pantalla de la página de configuración de SAML de ekarda con los campos de configuración de SAML resaltados.](./media/ekarda-tutorial/ekarda1.png)

    1. Seleccione el vínculo **Service Provider metadata** (Metadatos del proveedor de servicios) y guárdelo como un archivo en el equipo.
    1. Active la casilla **Enable SAML** (Habilitar SAML).
    1. En el cuadro de texto **IDP Entity ID** (Identificador de entidad del IDP), pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.
    1. En el cuadro de texto **IDP Login URL** (Dirección URL de inicio de sesión del IDP), pegue el valor de **Dirección URL de inicio de sesión** que copió de Azure Portal.
    1. En el cuadro de texto **IDP Logout URL** (Dirección URL de cierre de sesión del IDP), pegue el valor de **URL de cierre de sesión** que copió de Azure Portal.
    1. Utilice el Bloc de notas para abrir el archivo del **certificado (Base 64)** que descargó de Azure Portal. Pegue el contenido en el cuadro de texto **IDP X509 Certificate** (Certificado X509 del IDP).
    1. Active la casilla **Enable SLO** (Habilitar SLO) en la sección **OPTIONS** (Opciones).
    1. Seleccione **Actualizar**.

### <a name="create-an-ekarda-test-user"></a>Creación de un usuario de prueba de ekarda

En esta sección se crea un usuario llamado B.Simon en ekarda. ekarda admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No es necesario realizar ninguna acción en esta sección. Si el usuario llamado B.Simon no existe en ekarda, se crea uno después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el portal Aplicaciones.

Al seleccionar el icono de ekarda en el portal Mis aplicaciones, debería iniciar sesión automáticamente en el sitio de ekarda para el que configuró el inicio de sesión único. Para más información acerca del portal Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

* [Lista de tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)
* [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
* [Pruebe ekarda con Azure AD](https://aad.portal.azure.com/).
* Use la [solución eCard empresarial de ekarda](https://ekarda.com/ecards-ecards-with-logo-for-business-corporate-enterprise) para aprovisionar a los empleados para que envíen eCards con la marca del logotipo de la empresa a sus clientes y compañeros. Más información sobre el [aprovisionamiento de ekarda como una solución de inicio de sesión único](https://support.ekarda.com/#SSO-Implementation).
* [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
* [Protección de ekarda con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
