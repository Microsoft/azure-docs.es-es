---
title: 'Tutorial: Integración de Azure Active Directory con Salesforce Sandbox | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Salesforce Sandbox.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/16/2020
ms.author: jeedes
ms.openlocfilehash: 277acbc84ab435ce1076c30a1e49f6ffdd2a0586
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88543727"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-salesforce-sandbox"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Salesforce Sandbox

En este tutorial, aprenderá a integrar Salesforce Sandbox con Azure Active Directory (Azure AD). Al integrar Salesforce Sandbox con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Salesforce Sandbox.
* Permitir que los usuarios inicien sesión automáticamente en Salesforce Sandbox con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Salesforce Sandbox.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Salesforce Sandbox admite el inicio de sesión único iniciado por **SP e IDP**
* Salesforce Sandbox admite el aprovisionamiento de usuarios **Just-In-Time**
* Salesforce Sandbox admite el aprovisionamiento [**Automatizado** de usuarios](salesforce-sandbox-provisioning-tutorial.md)
* Una vez configurado Salesforce Sandbox, puede aplicar controles de sesión, que protegen la filtración y la infiltración de la información confidencial de la organización en tiempo real. Los controles de sesión proceden del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-aad).

## <a name="adding-salesforce-sandbox-from-the-gallery"></a>Adición de Salesforce Sandbox desde la galería

Para configurar la integración de Salesforce Sandbox, deberá agregar Salesforce Sandbox desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Salesforce Sandbox** en el cuadro de búsqueda.
1. Seleccione **Salesforce Sandbox** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-single-sign-on-for-salesforce-sandbox"></a>Configuración y prueba del inicio de sesión único de Azure AD para Salesforce Sandbox

Configure y pruebe el inicio de sesión único de Azure AD con Salesforce Sandbox mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Salesforce Sandbox.

Para configurar y probar el inicio de sesión único de Azure AD con Salesforce Sandbox, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Salesforce Sandbox](#configure-salesforce-sandbox-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba de Salesforce Sandbox](#create-salesforce-sandbox-test-user)** , para tener un homólogo de B.Simon en Salesforce Sandbox vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Salesforce Sandbox**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si tiene el **archivo de metadatos del proveedor de servicios** y desea realizar la configuración en el modo iniciado por **IDP**, realice los pasos siguientes:

    a. Haga clic en **Cargar el archivo de metadatos**.

    ![Carga del archivo de metadatos](common/upload-metadata.png)

    b. Haga clic en el **logotipo de la carpeta** para seleccionar el archivo de metadatos y luego en **Cargar**.

    ![Elección del archivo de metadatos](common/browse-upload-metadata.png)

    > [!NOTE]
    > Obtendrá el archivo de metadatos del proveedor de servicios en el portal de Salesforce Sandbox como se explica más adelante en el tutorial.

    c. Una vez se ha cargado correctamente el archivo de metadatos, el valor de **Dirección URL de respuesta** se rellenará automáticamente en el cuadro de texto **Dirección URL de respuesta**.

    ![imagen](common/both-replyurl.png)

    > [!Note]
    > Si el valor **URL de respuesta** no se rellena automáticamente, hágalo manualmente según sus necesidades.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar **XML de metadatos** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Configurar Salesforce Sandbox**, copie la dirección o direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

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

En esta sección va a permitir que B.Simon acceda a Salesforce Sandbox mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Salesforce Sandbox**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-salesforce-sandbox-sso"></a>Configuración del inicio de sesión único de Salesforce Sandbox

1. Abra una nueva pestaña en el explorador e inicie sesión en su cuenta de administrador de Salesforce Sandbox.

2. Haga clic en **Setup** (Configuración) en el **icono de configuración** de la esquina superior derecha de la página.

    ![Configurar inicio de sesión único](./media/salesforce-sandbox-tutorial/configure1.png)

3. Desplácese hacia abajo hasta **SETTINGS** (CONFIGURACIÓN) en el panel de navegación y haga clic en **Identity** (Identidad) para expandir la sección relacionada. A continuación, haga clic en **Configuración de inicio de sesión único**.

    ![Configurar inicio de sesión único](./media/salesforce-sandbox-tutorial/sf-admin-sso.png)

4. En la página **Configuración de inicio de sesión único**, haga clic en el botón **Editar**.

    ![Configurar inicio de sesión único](./media/salesforce-sandbox-tutorial/configure3.png)

5. Seleccione **SAML habilitado** y haga clic en **Guardar**.

    ![Configurar inicio de sesión único](./media/salesforce-sandbox-tutorial/sf-enable-saml.png)

6. Para establecer la configuración de inicio de sesión único de SAML, haga clic en **New from Metadata File** (Nuevo archivo de metadatos).

    ![Configurar inicio de sesión único](./media/salesforce-sandbox-tutorial/sf-admin-sso-new.png)

7. Haga clic en **Choose File** (Elegir archivo) para cargar el archivo XML de metadatos que ha descargado desde Azure Portal y haga clic en **Create** (Crear).

    ![Configurar inicio de sesión único](./media/salesforce-sandbox-tutorial/xmlchoose.png)

8. En la página **SAML Single Sign-On Settings** (Configuración de inicio de sesión único de SAML), los campos se rellenan automáticamente. Haga clic en Save (Guardar).

    ![Configurar inicio de sesión único](./media/salesforce-sandbox-tutorial/salesforcexml.png)

9. En la página **Single Sign-On Settings** (Configuración del inicio de sesión único), haga clic en el botón **Download Metadata** (Descargar metadatos) para descargar el archivo de metadatos del proveedor de servicio. Use este archivo en la sección **Configuración básica de SAML** en Azure Portal para configurar las direcciones URL necesarias, como se explicó anteriormente.

    ![Configurar inicio de sesión único](./media/salesforce-sandbox-tutorial/configure4.png)

10. Si quiere configurar la aplicación en el modo iniciado por **SP**, estos son los requisitos previos:

    a. Debe tener un dominio comprobado.

    b. Se necesita configurar y habilitar su dominio en Salesforce Sandbox. Los pasos para hacerlo se explican más adelante en este tutorial.

    c. En Azure Portal, en la sección **Configuración básica de SAML**, haga clic en **Establecer direcciones URL adicionales** y realice el paso siguiente:
  
    ![Información de dominio y direcciones URL de inicio de sesión único de Salesforce Sandbox](common/both-signonurl.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba el valor con el siguiente patrón: `https://<instancename>--Sandbox.<entityid>.my.salesforce.com`

    > [!NOTE]
    > Este valor se debe copiar desde el portal de Salesforce Sandbox, una vez que se ha habilitado el dominio.

11. En la sección **Certificado de firma de SAML**, haga clic en **XML de metadatos de federación** y luego guarde el archivo XML en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

12. Abra una nueva pestaña en el explorador e inicie sesión en su cuenta de administrador de Salesforce Sandbox.

13. Haga clic en **Setup** (Configuración) en el **icono de configuración** de la esquina superior derecha de la página.

    ![Configurar inicio de sesión único](./media/salesforce-sandbox-tutorial/configure1.png)

14. Desplácese hacia abajo hasta **SETTINGS** (CONFIGURACIÓN) en el panel de navegación y haga clic en **Identity** (Identidad) para expandir la sección relacionada. A continuación, haga clic en **Configuración de inicio de sesión único**.

    ![Configurar inicio de sesión único](./media/salesforce-sandbox-tutorial/sf-admin-sso.png)

15. En la página **Configuración de inicio de sesión único**, haga clic en el botón **Editar**.

    ![Configurar inicio de sesión único](./media/salesforce-sandbox-tutorial/configure3.png)

16. Seleccione **SAML habilitado** y haga clic en **Guardar**.

    ![Configurar inicio de sesión único](./media/salesforce-sandbox-tutorial/sf-enable-saml.png)

17. Para establecer la configuración de inicio de sesión único de SAML, haga clic en **New from Metadata File** (Nuevo archivo de metadatos).

    ![Configurar inicio de sesión único](./media/salesforce-sandbox-tutorial/sf-admin-sso-new.png)

18. Haga clic en **Choose File** (Elegir archivo) para cargar el archivo XML de metadatos y haga clic en **Create** (Crear).

    ![Configurar inicio de sesión único](./media/salesforce-sandbox-tutorial/xmlchoose.png)

19. En la página **SAML Single Sign-On Settings** (Configuración de inicio de sesión único de SAML) los campos se rellenan automáticamente, escriba el nombre de la configuración (por ejemplo: *SPSSOWAAD_Test*) en el cuadro de texto **Name** (Nombre) y haga clic en Guardar.

    ![Configurar inicio de sesión único](./media/salesforce-sandbox-tutorial/sf-saml-config.png)

20. Para habilitar su dominio en Salesforce Sandbox, lleve a cabo los siguientes pasos:

    > [!NOTE]
    > Antes de habilitar el dominio debe crearlo en Salesforce Sandbox. Para más información, consulte [Defining Your Domain Name](https://help.salesforce.com/HTViewHelpDoc?id=domain_name_define.htm&language=en_US) (Definición del nombre de dominio). Una vez creado el dominio, asegúrese de que está configurado correctamente.

21. En el panel de navegación izquierdo de Salesforce, haga clic en **Company Settings** (Configuración de la empresa) para expandir la sección relacionada y haga clic en **My Domain** (Mi dominio).

    ![Configurar inicio de sesión único](./media/salesforce-sandbox-tutorial/sf-my-domain.png)

22. En la sección **Authentication Configuration** (Configuración de autenticación), haga clic en **Edit** (Editar).

    ![Configurar inicio de sesión único](./media/salesforce-sandbox-tutorial/sf-edit-auth-config.png)

23. En la sección **Authentication Configuration** (Configuración de autenticación), seleccione como el valor de **Servicio de autenticación** el nombre de la Configuración de inicio de sesión único SAML que ha establecido durante la configuración de SSO en Salesforce Sandbox. Finalmente, haga clic en **Save** (Guardar).

    ![Configurar inicio de sesión único](./media/salesforce-sandbox-tutorial/configure2.png)

### <a name="create-salesforce-sandbox-test-user"></a>Creación de un usuario de prueba de Salesforce Sandbox

En esta sección, creará un usuario llamado a Britta Simon en Salesforce Sandbox. Salesforce Sandbox admite el aprovisionamiento Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario ya no existe en Salesforce Sandbox, se crea uno nuevo cuando se intenta acceder a esta aplicación. El espacio aislado de Salesforce también admite el aprovisionamiento automático de usuarios. [Aquí](salesforce-sandbox-provisioning-tutorial.md) puede encontrar más detalles sobre cómo configurar el aprovisionamiento automático de usuarios.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Salesforce Sandbox en el panel de acceso, debería iniciar sesión automáticamente en la instancia de Salesforce Sandbox para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Salesforce Sandbox con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/protect-salesforce)

- [Configuración del aprovisionamiento de usuarios](salesforce-sandbox-provisioning-tutorial.md)

- [Protección de Salesforce Sandbox con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
