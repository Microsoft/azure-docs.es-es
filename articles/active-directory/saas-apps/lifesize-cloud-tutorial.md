---
title: 'Tutorial: Integración de Azure Active Directory con Lifesize Cloud | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y Lifesize Cloud.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 1/4/2019
ms.author: jeedes
ms.openlocfilehash: 0ff2464705a770e257010347f0f04e9b695bb6e4
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88540140"
---
# <a name="tutorial-azure-active-directory-integration-with-lifesize-cloud"></a>Tutorial: Integración de Azure Active Directory con Lifesize Cloud

En este tutorial, aprenderá a integrar Lifesize Cloud con Azure Active Directory (Azure AD).
La integración de Lifesize Cloud con Azure AD ofrece las ventajas siguientes:

* En Azure AD se puede controlar quién tiene acceso a Lifesize Cloud.
* Puede permitir que los usuarios inicien sesión automáticamente en Lifesize Cloud (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Lifesize Cloud se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en Lifesize Cloud

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Lifesize Cloud admite el inicio de sesión único iniciado por **SP**

* Lifesize Cloud admite el aprovisionamiento **automatizado** de usuarios

## <a name="adding-lifesize-cloud-from-the-gallery"></a>Agregar Lifesize Cloud desde la galería

Para configurar la integración de Lifesize Cloud en Azure AD, necesita agregar Lifesize Cloud desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Lifesize Cloud desde la galería, siga este procedimiento:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Lifesize Cloud**, seleccione **Lifesize Cloud** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![Lifesize Cloud en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Lifesize Cloud con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Lifesize Cloud.

Para configurar y probar el inicio de sesión único de Azure AD con Lifesize Cloud, es necesario completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Lifesize Cloud](#configure-lifesize-cloud-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación del usuario de prueba de Lifesize Cloud](#create-lifesize-cloud-test-user)** : el objetivo es tener un homólogo de Britta Simon en Lifesize Cloud vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Lifesize Cloud, siga este procedimiento:

1. En la página de integración de la aplicación [Lifesize Cloud](https://portal.azure.com/) de **Azure Portal**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Lifesize Cloud](common/sp-identifier-relay.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://login.lifesizecloud.com/ls/?acs`

    b. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://login.lifesizecloud.com/<companyname>`

    c. Haga clic en **Establecer direcciones URL adicionales**.

    d. En el cuadro de texto **Estado de la retransmisión**, escriba una dirección URL que siga este patrón: `https://webapp.lifesizecloud.com/?ent=<identifier>`

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con el identificador real, la dirección URL de inicio de sesión y el estado de la retransmisión. Póngase en contacto con el [equipo de soporte técnico de clientes de Lifesize Cloud Client](https://www.lifesize.com/en/support) para obtener los valores de la dirección URL de inicio de sesión y del identificador; el valor de estado de retransmisión lo puede obtener de la configuración del inicio de sesión único que se explica más adelante en este tutorial. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Lifesize Cloud** (Configurar Lifesize Cloud), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-lifesize-cloud-single-sign-on"></a>Configuración del inicio de sesión único de Lifesize Cloud

1. Para configurar SSO para su aplicación, inicie sesión en la aplicación de Lifesize Cloud con privilegios de administrador.

2. En la esquina superior derecha, haga clic en su nombre y, después, en **Configuración avanzada**.

    ![Configurar inicio de sesión único](./media/lifesize-cloud-tutorial/tutorial_lifesizecloud_06.png)

3. En Configuración avanzada, haga clic en el vínculo **Configuración de SSO**. Se abrirá la página de configuración de SSO de la instancia.

    ![Configurar inicio de sesión único](./media/lifesize-cloud-tutorial/tutorial_lifesizecloud_07.png)

4. Ahora, configure los valores siguientes en la interfaz de configuración de SSO.

    ![Configurar inicio de sesión único](./media/lifesize-cloud-tutorial/tutorial_lifesizecloud_08.png)

    a. En el cuadro de texto **Emisor de proveedor de identidades**, pegue el valor de **Identificador Azure AD** que ha copiado de Azure Portal.

    b.  En el cuadro de texto **Dirección URL de inicio de sesión**, pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    c. Abra el certificado codificado en base 64 descargado de Azure Portal en el Bloc de notas, copie su contenido en el Portapapeles y luego péguelo en el cuadro de texto **Certificado X.509**.
  
    d. En las asignaciones de SAML Attribute del cuadro de texto First Name (Nombre), escriba el valor `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`

    e. En las asignaciones de SAML Attribute del cuadro de texto **Last Name** (Apellidos), escriba el valor `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`

    f. En las asignaciones de SAML Attribute del cuadro de texto **Email** (Dirección de correo electrónico), escriba el valor `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`

5. Para comprobar la configuración, puede hacer clic en el botón **Probar**.

    >[!NOTE]
    >Para realizar una prueba correctamente, necesita completar el asistente para configuración en Azure AD y, además, proporcionar acceso los usuarios o grupos que pueden realizar la prueba.

6. Para habilitar SSO, seleccione el botón **Habilitar SSO**.

7. Ahora, haga clic en el botón **Actualizar** para guardar la configuración. Se generará el valor RelayState. Copie el valor de RelayState, que se genera en el cuadro de texto y péguelo en el cuadro de texto **Estado de la retransmisión** en la sección **Dominio y direcciones URL de Lifesize Cloud**.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba **brittasimon\@yourcompanydomain.extension**.  
    Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Lifesize Cloud.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Lifesize Cloud**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Lifesize Cloud**.

    ![Vínculo a Lifesize Cloud en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-lifesize-cloud-test-user"></a>Creación del usuario de prueba de Lifesize Cloud

En esta sección, creará el usuario Britta Simon en Lifesize Cloud. Lifesize Cloud no admite el aprovisionamiento de usuarios automático. Después de autenticarse correctamente en Azure AD, el usuario se aprovisionará automáticamente en la aplicación.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Lifesize Cloud en el Panel de acceso, debería llegar a la página de inicio de sesión de la aplicación Lifesize Cloud. Aquí debe especificar el nombre de usuario y se le redirigirá a la página principal de la aplicación.

Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
