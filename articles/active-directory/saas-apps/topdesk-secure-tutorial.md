---
title: 'Tutorial: integración de Azure Active Directory con TOPdesk - Secure | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y TOPdesk - Secure.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/27/2018
ms.author: jeedes
ms.openlocfilehash: db7f2afa7993c8f8d2fe9fe30e5ecd4f2388b44b
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88524273"
---
# <a name="tutorial-azure-active-directory-integration-with-topdesk---secure"></a>Tutorial: integración de Azure Active Directory con TOPdesk - Secure

En este tutorial, obtendrá información sobre cómo integrar TOPdesk - Secure con Azure Active Directory (Azure AD).
La integración de TOPdesk - Secure con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a TOPdesk - Secure.
* Puede permitir que los usuarios inicien sesión automáticamente en TOPdesk - Secure (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con TOPdesk - Secure, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Suscripción habilitada para el inicio de sesión único en TOPdesk - Secure

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* TOPdesk - Secure admite el inicio de sesión único iniciado por **SP**

## <a name="adding-topdesk---secure-from-the-gallery"></a>Agregar TOPdesk - Secure desde la galería

Para configurar la integración de TOPdesk - Secure en Azure AD, deberá agregar TOPdesk - Secure desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar TOPdesk - Secure desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **TOPdesk - Secure**, seleccione **TOPdesk - Secure** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![TOPdesk - Secure en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con TOPdesk - Secure con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de TOPdesk - Secure.

Para configurar y probar el inicio de sesión único de Azure AD con TOPdesk - Secure, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de TOPdesk - Secure](#configure-topdesk---secure-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de TOPdesk - Secure](#create-topdesk---secure-test-user)** : para tener un homólogo de Britta Simon en TOPdesk - Secure que esté vinculado a su representación en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con TOPdesk - Secure, realice los pasos siguientes:

1. En la página de integración aplicaciones de [TOPdesk - Secure](https://portal.azure.com/) de **Azure Portal**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de TOPdesk - Secure](common/sp-identifier-reply.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.topdesk.net`

    b. En el cuadro **URL del identificador**, rellene la dirección URL de metadatos de TOPdesk que puede recuperar desde la configuración de esta aplicación. Debe usar el patrón siguiente: `https://<companyname>.topdesk.net/saml-metadata/<identifier>`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.topdesk.net/tas/secure/login/verify`

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de inicio de sesión, el identificador y la dirección URL de respuesta reales. Póngase en contacto con el [equipo de soporte técnico de TOPdesk - Secure](https://www.topdesk.com/us/support/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up TOPdesk - Secure**  (Configurar TOPdesk - Secure), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-topdesk---secure-single-sign-on"></a>Configuración del inicio de sesión único de TOPdesk - Secure

1. Inicie sesión en su sitio de la compañía de **TOPdesk - Secure** como administrador.

2. En el menú **TOPdesk**, haga clic en **Configuración**.

    ![Configuración](./media/topdesk-secure-tutorial/ic790598.png "Configuración")

3. Haga clic en **Login Settings**(Configuración de inicio de sesión).

    ![Login Settings (Configuración de inicio de sesión)](./media/topdesk-secure-tutorial/ic790599.png "Login Settings")

4. Expanda el menú **Login Settings** (Configuración de inicio de sesión) y luego haga clic en **General**.

    ![General](./media/topdesk-secure-tutorial/ic790600.png "General")

5. En la sección **Secure** (Seguro) de la sección de configuración **SAML login** (Inicio de sesión SAML), realice los pasos siguientes:

    ![Technical Settings (Configuración técnica)](./media/topdesk-secure-tutorial/ic790855.png "Configuración técnica")

    a. Haga clic en **Download** (Descargar) para descargar el archivo de metadatos público y luego guárdelo localmente en el equipo.

    b. Abra el archivo de metadatos y luego busque el nodo **AssertionConsumerService** .

    ![Servicio del consumidor de aserción](./media/topdesk-secure-tutorial/ic790856.png "Servicio del consumidor de aserción")

    c. Copia el valor de **AssertionConsumerService**, péguelo en el cuadro de texto Dirección URL de respuesta de la sección **Dominio y direcciones URL de TOPdesk - Secure**.

6. Lleve a cabo los siguientes pasos para crear un archivo de certificado:

    ![Certificate](./media/topdesk-secure-tutorial/ic790606.png "Certificado")

    a. Abra el archivo de metadatos descargado de Azure Portal.

    b. Expanda el nodo **RoleDescriptor** cuyo **xsi:type** es **fed:ApplicationServiceType**.

    c. Copie el valor del nodo **X509Certificate** .

    d. Guarde el valor de **X509Certificate** copiado localmente en el equipo en un archivo.

7. En la sección **Public** (Público), haga clic en **Add** (Agregar).

    ![Add (Agregar)](./media/topdesk-secure-tutorial/ic790607.png "Sumar")

8. En la página de diálogo del **SAML configuration assistant** (Asistente de configuración de SAML), realice los siguientes pasos:

    ![SAML Configuration Assistant (Asistente de configuración de SAML)](./media/topdesk-secure-tutorial/ic790608.png "SAML configuration assistant")

    a. Para cargar el archivo de metadatos descargado de Azure Portal, en **Federation Metadata** (Metadatos de federación) haga clic en **Browse** (Examinar).

    b. Para cargar el archivo del certificado, en **Certificate (RSA)** (Certificado [RSA]), haga clic en **Browse** (Examinar).

    c. Para **Clave privada (RSA, PKCS8, DER)** , puede cargar su propia clave privada o puede ponerse en contacto con el [equipo de soporte técnico de TOPdesk](https://www.topdesk.com/us/support) para obtener la clave privada.

    d. Para cargar el archivo de logotipo que obtuvo del equipo de soporte técnico de TOPdesk, en **Logo icon** (Icono de logotipo), haga clic en **Browse** (Examinar).

    e. En el cuadro de texto **User name attribute** (Atributo de nombre de usuario), escriba `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`.

    f. En el cuadro de texto **Display name** (Nombre para mostrar), escriba un nombre para su configuración.

    g. Haga clic en **Save**(Guardar).

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a TOPdesk - Secure.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **TOPdesk - Secure**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, escriba **TOPdesk - Secure**.

    ![Vínculo a TOPdesk - Secure en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-topdesk---secure-test-user"></a>Creación de un usuario de prueba de TOPdesk - Secure

Para permitir que los usuarios de Azure AD inicien sesión en TOPdesk - Secure, deben aprovisionarse en TOPdesk - Secure.  
En el caso de TOPdesk - Secure, el aprovisionamiento es una tarea manual.

### <a name="to-configure-user-provisioning-perform-the-following-steps"></a>Siga estos pasos para configurar el aprovisionamiento de usuario:

1. Inicie sesión en su sitio de la compañía de **TOPdesk - Secure** como administrador.

2. En el menú de la parte superior, haga clic en **TOPdesk \> New \> Support Files \> Operator** (TOPdesk > Nuevo > Archivos de soporte > Operador).

    ![Operador](./media/topdesk-secure-tutorial/ic790610.png "Operator")

3. En el cuadro de diálogo **Nuevo operador** , realice los pasos siguientes:

    ![New Operator (Nuevo operador)](./media/topdesk-secure-tutorial/ic790611.png "Nuevo operador")

    a. Haga clic en la pestaña **General**.

    b. En el cuadro de texto **Apellidos**, escriba los apellidos del usuario, en este caso, **Simon**.

    c. Seleccione un **sitio** para la cuenta en la sección **Location** (Ubicación).

    d. En el cuadro de texto **Login Name** (Nombre de inicio de sesión) de la sección **TOPdesk Login** (Inicio de sesión de TOPdesk), escriba el nombre de inicio de sesión del usuario.

    e. Haga clic en **Save**(Guardar).

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de TOPdesk - Secure ofrecida por TOPdesk - Secure para aprovisionar cuentas de usuario de Azure AD.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de TOPdesk - Secure en el panel de acceso y debería iniciar sesión automáticamente en la versión de TOPdesk - Secure para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

