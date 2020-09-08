---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Workday | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Workday.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/31/2020
ms.author: jeedes
ms.openlocfilehash: 4867a1735f091085f64bbe7010969bd086f820a1
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88527181"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-workday"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Workday

En este tutorial, obtendrá información sobre cómo integrar Workday con Azure Active Directory (Azure AD). Al integrar Workday con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Workday.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Workday con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Workday.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Workday admite el inicio de sesión único iniciado por **SP**.

* Una vez configurado Workday, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-aad).

## <a name="adding-workday-from-the-gallery"></a>Incorporación de Workday desde la galería

Para configurar la integración de Workday en Azure AD, hay que agregar Workday desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Workday** en el cuadro de búsqueda.
1. Seleccione **Workday** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-workday"></a>Configuración y prueba del inicio de sesión único de Azure AD para Workday

Configure y pruebe el inicio de sesión único de Azure AD con Workday mediante un usuario de prueba llamado **B.Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Workday.

Para configurar y probar el inicio de sesión único de Azure AD con Workday, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
2. **[Configuración de Workday](#configure-workday)** , para configurar las opciones del SSO en la aplicación.
    1. **[Creación de un usuario de prueba de Workday](#create-workday-test-user)** : para tener un homólogo de B. Simon en Workday que esté vinculado a la representación del usuario en Azure AD.
3. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Workday**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://impl.workday.com/<tenant>/login-saml2.flex`

    b. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `http://www.workday.com`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://impl.workday.com/<tenant>/login-saml.htmld`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de URL de respuesta y URL de inicio de sesión. La URL de respuesta debe tener un subdominio (por ejemplo, www, wd2, wd3, wd3-impl, wd5 y wd5-impl).
    > Algo como `http://www.myworkday.com` funciona, pero `http://myworkday.com` no. Póngase en contacto con el [equipo de soporte técnico de Workday](https://www.workday.com/en-us/partners-services/services/support.html) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

6. La aplicación Workday espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de pantalla muestra la lista de atributos predeterminados, donde **nameidentifier** se asigna con **user.userprincipalname**. La aplicación Workday espera que **nameidentifier** se asigne con **user.mail**, **UPN** etc., por lo que debe editar la asignación de atributos haciendo clic en el icono **Editar** y cambiar dicha asignación.

    ![imagen](common/edit-attribute.png)

    > [!NOTE]
    > Aquí hemos asignado el Identificador de nombre con el UPN (user.userprincipalname) como valor predeterminado. Tiene que asignar el Identificador de nombre con el Identificador de usuario real en su cuenta de Workday (su correo electrónico, UPN etc.) para el correcto funcionamiento del inicio de sesión único.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

   ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. Para modificar las opciones de **Firma** según sus propios requisitos, haga clic en el botón **Editar** para abrir el cuadro de diálogo **Certificado de firma de SAML**.

    ![imagen](common/edit-certificate.png) 

    ![imagen](./media/workday-tutorial/signing-option.png)

    a. Seleccione **Opción de firma** como **Firmar respuesta y aserción SAML**.

    b. Haga clic en **Guardar**

1. En la sección **Set up Workday** (Configurar Workday), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B. Simon acceda a Workday mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Workday**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-workday"></a>Configuración de Workday

1. En otra ventana del explorador web, inicie sesión en el sitio de la compañía de Workday como administrador.

2. En el **cuadro de búsqueda**, escriba el nombre **Edit Tenant Setup – Security** (Editar configuración de inquilino – Seguridad) en la parte superior izquierda de la página principal.

    ![Edición de la seguridad del inquilino](./media/workday-tutorial/IC782925.png "Editar seguridad del inquilino")

3. En la sección **URL de redireccionamiento** , siga estos pasos:

    ![Direcciones URL de redireccionamiento](./media/workday-tutorial/IC7829581.png "URL de redireccionamiento")

    a. Haga clic en **Add Row**(Agregar fila).

    b. En los cuadros de texto **Dirección URL de redireccionamiento de inicio de sesión**, **Dirección URL de redireccionamiento de tiempo de espera** y **Dirección URL de redireccionamiento móvil**, pegue la **Dirección URL de inicio de sesión** que ha copiado de la sección **Set up Workday** (Configurar Workday) de Azure Portal.

    c. En el cuadro de texto **Logout Redirect URL** (URL de redireccionamiento de cierre de sesión), pegue la **URL de cierre de sesión** que copió de la sección **Set up Workday** (Configurar Workday) de Azure Portal.

    d. En el cuadro de texto **Used for Environments** (Se usa en entornos), seleccione el nombre del entorno.  

   > [!NOTE]
   > El valor del atributo Entorno está vinculado con el valor de la URL del inquilino:  
   > Si el nombre de dominio de la dirección URL de inquilino de Workday empieza por impl (por ejemplo, *https://www.myworkday.com/"tenant"/login-saml2.htmld*), el atributo **Environment** (Entorno) tiene que establecerse en Implementation (Implementación).  
   > Si el nombre de dominio empieza por otra cosa, deberá ponerse en contacto con el [equipo de soporte técnico de Workday](https://www.workday.com/en-us/partners-services/services/support.html) para obtener el valor de **Entorno** coincidente.

4. En la sección **Configuración de SAML** , realice los pasos siguientes:

    ![Configuración de SAML](./media/workday-tutorial/IC782926.png "Configuración de SAML")

    a.  Seleccione **Habilitar autenticación SAML**.

    b.  Haga clic en **Add Row**(Agregar fila).

5. En la sección **Proveedores de identidades SAML**, realice los pasos siguientes:

    ![Proveedores de identidades de SAML](./media/workday-tutorial/IC7829271.png "Proveedores de identidades SAML")

    a. En el cuadro de texto **Identity Provider Name** (Nombre del proveedor de identidades), escriba un nombre de proveedor (por ejemplo: *SPInitiatedSSO*).

    b. En Azure Portal, en la sección **Set up Workday** (Configurar Workday), copie el valor de **Identificador de Azure AD** y péguelo en el cuadro de texto **Emisor**.

    ![Proveedores de identidades de SAML](./media/workday-tutorial/IC7829272.png "Proveedores de identidades SAML")

    c. En Azure Portal, en la sección **Set up Workday** (Configurar Workday), copie el valor de **URL de cierre de sesión** y péguelo en el cuadro de texto **Logout Request URL** (URL de solicitud de cierre de sesión).

    d. En Azure Portal, en la sección **Set up Workday** (Configurar Workday), copie el valor de **URL de inicio de sesión** y péguelo en el cuadro de texto **IdP SSO Service URL** (URL del servicio SSO de IdP).

    e. En el cuadro de texto **Used for Environments** (Se usa en entornos), seleccione el nombre del entorno.

    f. Haga clic en **Identity Provider Public Key Certificate** (Certificado de clave pública de proveedor de identidades) y, después, en **Crear**.

    ![Creación](./media/workday-tutorial/IC782928.png "Crear")

    g. Haga clic en **Create x509 Public Key**(Crear clave pública x509).

    ![Creación](./media/workday-tutorial/IC782929.png "Crear")

6. En la sección **View x509 Public Key** (Ver clave pública x509), siga estos pasos:

    ![View x509 Public Key](./media/workday-tutorial/IC782930.png "View x509 Public Key") (Ver clave pública x509)

    a. En el cuadro de texto **Name** (Nombre), escriba un nombre para el certificado (por ejemplo: *PPE\_SP*).

    b. En el cuadro de texto **Válido desde** , escriba el valor del atributo Válido desde del certificado.

    c.  En el cuadro de texto **Válido hasta** , escriba el valor del atributo Válido hasta del certificado.

    > [!NOTE]
    > Puede obtener la fecha válido desde y la fecha válido hasta del certificado descargado haciendo doble clic en él.  Las fechas se muestran bajo la pestaña **Detalles** .
    >
    >

    d.  Abra el certificado codificado en base 64 en el Bloc de notas y luego copie el contenido del mismo.

    e.  En el cuadro de texto **Certificado** , pegue el contenido del portapapeles.

    f.  Haga clic en **OK**.

7. Lleve a cabo los siguiente pasos:

    ![Configuración del inicio de sesión único](./media/workday-tutorial/WorkdaySSOConfiguratio.png "Configuración de SSO")

    a.  En el cuadro de texto **Service Provider ID** (Id. de proveedor de servicios), escriba **http://www.workday.com** .

    b. Seleccione **No desinflar la solicitud de autenticación iniciada por el SP**.

    c. Como **Método de firma de solicitud de autenticación**, seleccione **SHA256**.

    ![Método de firma de solicitud de autenticación](./media/workday-tutorial/WorkdaySSOConfiguration.png "Método de firmas de solicitudes de autenticación")

    d. Haga clic en **OK**.

    ![OK (CORRECTO)](./media/workday-tutorial/IC782933.png "Aceptar")

    > [!NOTE]
    > Asegúrese de configurar el inicio de sesión único correctamente. Si habilita el inicio de sesión único con una configuración incorrecta, es posible que no pueda entrar en la aplicación con sus credenciales y quede bloqueado fuera de esta. En esta situación, Workday proporciona una URL de inicio de sesión de respaldo donde los usuarios pueden iniciar sesión con su nombre de usuario y contraseña normal en el siguiente formato: [Su URL de Workday]/login.flex?redirect=n

### <a name="create-workday-test-user"></a>Creación de un usuario de prueba de Workday

En esta sección, creará un usuario llamado B. Simon en Workday. Trabaje con el [equipo de soporte técnico al cliente de Workday](https://www.workday.com/en-us/partners-services/services/support.html) para agregar los usuarios a la plataforma de Workday. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

Cuando selecciona el icono de Workday en el panel de acceso, debería iniciar sesión automáticamente en la versión de Workday para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Probar WorkDay con Azure AD](https://aad.portal.azure.com)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Workday con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/protect-workday)