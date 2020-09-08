---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con iProva | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory e iProva.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/19/2020
ms.author: jeedes
ms.openlocfilehash: 187ec5c42c733837689de95c8111b557519f8b7b
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88552850"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-iprova"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con iProva

En este tutorial, aprenderá a integrar iProva con Azure Active Directory (Azure AD). Al integrar iProva con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a iProva.
* Permitir que los usuarios puedan iniciar sesión automáticamente en iProva con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en iProva.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* iProva admite el inicio de sesión único iniciado por **SP**

* Una vez configurado iProva, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-iprova-from-the-gallery"></a>Incorporación de iProva desde la galería

Para configurar la integración de iProva en Azure AD, tendrá que agregar iProva desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **iProva** en el cuadro de búsqueda.
1. Seleccione **iProva** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-iprova"></a>Configuración y prueba del inicio de sesión único de Azure AD para iProva

Configure y pruebe el inicio de sesión único (SSO) de Azure AD con iProva mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de iProva.

Para configurar y probar el inicio de sesión único de Azure AD con iProva, es preciso completar los siguientes bloques de creación:

1. **[Recuperación de la información de configuración desde iProva](#retrieve-configuration-information-from-iprova)** : como preparación para los pasos siguientes.
1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Creación de un usuario de prueba de iProva](#create-iprova-test-user)** : para tener un homólogo de B.Simon en iProva que esté vinculado a la representación del usuario en Azure AD.
1. **[Configuración del inicio de sesión único en iProva](#configure-iprova-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="retrieve-configuration-information-from-iprova"></a>Recuperación de la información de configuración desde iProva

En esta sección, va a recuperar información de iProva para configurar el inicio de sesión único en Azure AD.

1. Abra un explorador web y vaya a la **página de información de SAML2** en iProva, con el siguiente patrón de dirección URL:

    ```https
    https://SUBDOMAIN.iprova.nl/saml2info
    https://SUBDOMAIN.iprova.be/saml2info
    ```

    ![Visualización de la página de información de SAML2 de iProva](media/iprova-tutorial/iprova-saml2-info.png)

1. Deje abierta la pestaña del explorador mientras continúa con los pasos siguientes en otra pestaña.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **iProva**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. Rellene el campo **URL de inicio de sesión** con el valor que se muestra detrás de la etiqueta **URL de inicio de sesión** de la página de **información de SAML2 de iProva**. Esta página sigue abierta en la otra pestaña del explorador.

    b. Rellene el campo **Identifier** (Identificador) con el valor que se muestra detrás de la etiqueta **EntityID** (Identificador de la entidad) de la **página de información de SAML2 de iProva**. Esta página sigue abierta en la otra pestaña del explorador.

    c. Rellene el campo **URL de respuesta** con el valor que se muestra detrás de la etiqueta **URL de respuesta** de la página de **información de SAML2 de iProva**. Esta página sigue abierta en la otra pestaña del explorador.

1. La aplicación iProva espera las aserciones de SAML en un formato específico que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación iProva espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

    | Nombre | Atributo de origen| Espacio de nombres  |
    | ---------------| -------- | -----|
    | `samaccountname` | `user.onpremisessamaccountname`| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims`|

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar la **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

## <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

## <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir que B.Simon acceda a iProva mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **iProva**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="create-iprova-test-user"></a>Creación de un usuario de prueba para iProva

1. Inicie sesión en iProva con la cuenta de **Administrador**.

2. Abra el menú **Ir a**.

3. Seleccione **Administración de aplicaciones**.

4. Seleccione **Usuarios** en el panel **Usuarios y grupos de usuarios**.

5. Seleccione **Agregar**.

6. En el cuadro **Nombre de usuario**, escriba el nombre de un usuario; por ejemplo, `B.Simon@contoso.com`.

7. En el cuadro **Nombre completo**, escriba el nombre completo del usuario, como **B.Simon**.

8. Seleccione la opción **No password (use single sign-on)** (Sin contraseña [ usar inicio de sesión único]).

9. En el cuadro **Email address** (Dirección de correo electrónico), escriba la dirección de correo electrónico del usuario, como `B.Simon@contoso.com`.

10. Desplácese al final de la página y seleccione **Finalizar**.

## <a name="configure-iprova-sso"></a>Configuración del inicio de sesión único de iProva

1. Inicie sesión en iProva con la cuenta de **Administrador**.

2. Abra el menú **Ir a**.

3. Seleccione **Administración de aplicaciones**.

4. Seleccione **General** en el panel **Configuración del sistema**.

5. Seleccione **Editar**.

6. Desplácese hacia abajo hasta **Control de acceso**.

    ![Configuración del control de acceso de iProva](media/iprova-tutorial/iprova-accesscontrol.png)

7. Busque la opción **Users are automatically logged on with their network accounts** (Los usuarios inician sesión automáticamente con sus cuentas de red) y cámbiela a **Yes, authentication via SAML** (Sí, autenticación vía SAML). Ahora aparecen otras opciones.

8. Seleccione **Configurar**.

9. Seleccione **Next** (Siguiente).

10. iProva le pregunta si quiere descargar datos de federación desde una dirección URL o cargarlos desde un archivo. Seleccione la opción **From URL** (Desde dirección URL).

    ![Descarga de metadatos de Azure AD](media/iprova-tutorial/iprova-download-metadata.png)

11. Pegue la dirección URL de metadatos que guardó en el último paso de la sección "Configuración del inicio de sesión único de Azure AD".

12. Seleccione el botón con forma de flecha para descargar los metadatos desde Azure AD.

13. Cuando finaliza la descarga, aparece el mensaje de confirmación **Valid Federation Data file downloaded** (Se descargó un archivo de datos de federación válido).

14. Seleccione **Next** (Siguiente).

15. Omita por ahora la opción **Test login** (Probar inicio de sesión) y seleccione **Siguiente**.

16. En el cuadro desplegable **Claim to use** (Notificación que usar), seleccione **windowsaccountname**.

17. Seleccione **Finalizar**.

18. Ahora vuelva a la pantalla **Edit general settings** (Editar la configuración general). Desplácese hasta el final de la página y seleccione **Aceptar** para guardar la configuración.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de iProva en el panel de acceso y debería iniciar sesión automáticamente en la versión de iProva para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de iProva con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de iProva con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)