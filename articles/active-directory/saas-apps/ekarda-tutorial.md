---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Ekarda | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Ekarda.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 8e2945aa-46fc-41bc-a530-3807a5dcb76a
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 06/15/2020
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: aacaec5ff632385a1f1686610370bb92eb63c349
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87017575"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-ekarda"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Ekarda

En este tutorial, aprenderá a integrar Ekarda con Azure Active Directory (Azure AD). Al integrar Ekarda con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Ekarda.
* Permitir que los usuarios inicien sesión automáticamente en Ekarda con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Ekarda.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Ekarda admite el inicio de sesión único iniciado por **SP e IDP**.
* Ekarda admite el aprovisionamiento de usuarios **Just-In-Time**.

* Una vez configurado Ekarda, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-ekarda-from-the-gallery"></a>Adición de Ekarda desde la galería

Para configurar la integración de Ekarda en Azure AD, deberá agregar Ekarda desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Ekarda** en el cuadro de búsqueda.
1. Seleccione **Ekarda** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-single-sign-on-for-ekarda"></a>Configuración y prueba del inicio de sesión único de Azure AD para Ekarda

Configure y pruebe el inicio de sesión único de Azure AD con Ekarda mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Ekarda.

Para configurar y probar el inicio de sesión único de Azure AD con Ekarda, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Ekarda](#configure-ekarda-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Ekarda](#create-ekarda-test-user)** : para tener un homólogo de B.Simon en Ekarda que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Ekarda**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si tiene el **archivo de metadatos del proveedor de servicios**, lleve a cabo los siguientes pasos:
    
    a. Haga clic en **Cargar el archivo de metadatos**.

    b. Haga clic en el **logotipo de la carpeta** para seleccionar el archivo de metadatos y luego en **Cargar**.

    c. Cuando se haya cargado correctamente el archivo de metadatos, los valores de **Identificador** y **Dirección URL de respuesta** se rellenarán automáticamente en los cuadros de texto de la sección de Ekarda:

    > [!Note]
    > Si los valores **Identificador** y **Dirección URL de respuesta** no se rellenan automáticamente, hágalo manualmente según sus necesidades.

1. Si no tiene el **archivo de metadatos del proveedor de servicios** en la sección **Configuración básica de SAML** y desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://my.ekarda.com/users/saml_metadata/<COMPANY_ID>`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://my.ekarda.com/users/saml_acs/<COMPANY_ID>`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://my.ekarda.com/users/saml_sso/<COMPANY_ID>`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de Ekarda](mailto:contact@ekarda.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configuración del inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar el **certificado (Base 64)** y guárdelo en su equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Ekarda**, copie las direcciones URL según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Ekarda mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Ekarda**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-ekarda-sso"></a>Configuración del inicio de sesión único de Ekarda

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la compañía en Ekarda.

1. Haga clic en **Admin** -> **My Account** (Administración > Mi cuenta).

    ![Configuración de Ekarda](./media/ekarda-tutorial/ekarda.png)    

1. En la parte inferior de la página encontrará la sección **SAML SETTINGS** (Configuración de SAML), en la que se configurará la integración de SAML.
1. En la página siguiente, realice los pasos siguientes:

    ![Configuración de Ekarda](./media/ekarda-tutorial/ekarda1.png)

    a. Haga clic en el vínculo **Service Provider metadata** (Metadatos del proveedor de servicios) y guárdelo como un archivo en el equipo.

    b. Seleccione la casilla **Enable SAML** (Habilitar SAML).

    c. En el cuadro de texto **IDP Entity ID** (Identificador de entidad del IDP), pegue el valor de **Id. de entidad** que ha copiado desde Azure Portal.

    d. En el cuadro de texto **IDP Login URL** (Dirección URL de inicio de sesión del IDP), pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    e. En el cuadro de texto **IDP Logout URL** (Dirección URL de cierre de sesión del IDP), pegue el valor de **URL de cierre de sesión** que ha copiado de Azure Portal.

    f. Abra en el Bloc de notas el archivo del **certificado (Base 64)** que descargó de Azure Portal, copie el contenido y, luego, péguelo en el cuadro de texto **IDP x509 Certificate** (Certificado x509 del IDP).

    g. Seleccione **Enable SLO** (Habilitar SLO) en la sección **OPTIONS** (Opciones).

    h. Haga clic en **Update** (Actualizar).

### <a name="create-ekarda-test-user"></a>Creación de un usuario de prueba de Ekarda

En esta sección se crea un usuario llamado B.Simon en Ekarda. Ekarda admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe aún en Ekarda, se crea uno nuevo después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Ekarda en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de Ekarda para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Ekarda con Azure AD](https://aad.portal.azure.com/).

- Use la [solución eCard empresarial de Ekarda](https://ekarda.com/ecards-ecards-with-logo-for-business-corporate-enterprise) para aprovisionar a los empleados que envíen eCards con la marca del logotipo de la empresa a sus clientes y compañeros. Obtenga más información sobre el aprovisionamiento de [Ekarda como una solución de inicio de sesión único](https://support.ekarda.com/#SSO-Implementation).

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Ekarda con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
