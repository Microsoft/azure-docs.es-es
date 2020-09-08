---
title: 'Tutorial: Integración de Azure Active Directory con TOPdesk - Public | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y TOPdesk - Public.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/02/2019
ms.author: jeedes
ms.openlocfilehash: a18cb583b246c3cfbf09f43b9921497de37a3613
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88524296"
---
# <a name="tutorial-azure-active-directory-integration-with-topdesk---public"></a>Tutorial: Integración de Azure Active Directory con TOPdesk - Public

En este tutorial, obtendrá información sobre cómo integrar TOPdesk - Public con Azure Active Directory (Azure AD).
La integración de TOPdesk - Public con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a TOPdesk - Public.
* Puede permitir que los usuarios inicien sesión automáticamente en TOPdesk - Public (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con TOPdesk - Public, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en TOPdesk - Public

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* TOPdesk - Public admite el inicio de sesión único iniciado por **SP**

## <a name="adding-topdesk---public-from-the-gallery"></a>Agregar TOPdesk - Public desde la galería

Para configurar la integración de TOPdesk - Public en Azure AD, deberá agregar TOPdesk - Public desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar TOPdesk - Public desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **TOPdesk - Public**, seleccione **TOPdesk - Public** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![TOPdesk - Public en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con TOPdesk - Public con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de TOPdesk - Public.

Para configurar y probar el inicio de sesión único de Azure AD con TOPdesk - Public, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de TOPdesk - Public](#configure-topdesk---public-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de TOPdesk - Public](#create-topdesk---public-test-user)** : para tener un homólogo de Britta Simon en TOPdesk - Public que esté vinculado a su representación en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con TOPdesk - Public, realice los pasos siguientes:

1. En la página de integración de la aplicación **TOPdesk - Public** de [Azure Portal](https://portal.azure.com/), seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4.  En la sección **Configuración básica de SAML**, si tiene el **archivo de metadatos del proveedor de servicios**, lleve a cabo los siguientes pasos:

    >[!NOTE]
    >Obtendrá el **archivo de metadatos del proveedor de servicios**, como se explica más adelante en la sección **Configuración del inicio de sesión único de TOPdesk - Public** del tutorial.

    a. Haga clic en **Cargar el archivo de metadatos**.
    
    ![Carga del archivo de metadatos](common/upload-metadata.png)

    b. Haga clic en el **logotipo de la carpeta** para seleccionar el archivo de metadatos y luego en **Cargar**.

    ![Elección del archivo de metadatos](common/browse-upload-metadata.png)

    c. Una vez que se haya cargado correctamente el archivo de metadatos, el valor de **Identificador** y **URL de respuesta** se rellena automáticamente en la sección Configuración básica de SAML.

    ![Información de dominio y direcciones URL de inicio de sesión único de TOPdesk - Public](common/sp-identifier-reply.png)

    d. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.topdesk.net`

    e. En el cuadro de texto **URL del identificador**, rellene la dirección URL de metadatos de TOPdesk que puede recuperar de la configuración de esta aplicación. Debe usar el patrón siguiente: `https://<companyname>.topdesk.net/saml-metadata/<identifier>`
    
    f. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.topdesk.net/tas/public/login/verify`.
    
    > [!NOTE] 
    > Si los valores de **Identificador** y de **Dirección URL de respuesta** no se rellenan automáticamente, debe escribirlos manualmente. En Identificador, siga el patrón como se mencionó anteriormente y obtenga el valor de Dirección URL de respuesta en la sección **Configuración del inicio de sesión único de TOPdesk - Public**, que se explica más adelante en el tutorial. El valor de **Dirección URL de inicio de sesión** no es real de modo que debe actualizarla con esta. Póngase en contacto con el [equipo de soporte técnico de TOPdesk - Public](https://help.topdesk.com/saas/enterprise/user/) para obtener el valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Configurar TOPdesk - Public**, copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-topdesk---public-single-sign-on"></a>Configuración del inicio de sesión único de TOPdesk - Public

1. Inicie sesión en el sitio de la compañía de **TOPdesk - Public** como administrador.

2. En el menú **TOPdesk**, haga clic en **Configuración**.
   
    ![Configuración](./media/topdesk-public-tutorial/ic790598.png "Configuración")

3. Haga clic en **Login Settings**(Configuración de inicio de sesión).
   
    ![Login Settings (Configuración de inicio de sesión)](./media/topdesk-public-tutorial/ic790599.png "Login Settings")

4. Expanda el menú **Login Settings** (Configuración de inicio de sesión) y luego haga clic en **General**.
   
    ![General](./media/topdesk-public-tutorial/ic790600.png "General")

5. En la sección **Público** de la sección de configuración **Inicio de sesión SAML**, siga estos pasos:
   
    ![Technical Settings (Configuración técnica)](./media/topdesk-public-tutorial/ic790601.png "Configuración técnica")
   
    a. Haga clic en **Download** (Descargar) para descargar el archivo de metadatos público y luego guárdelo localmente en el equipo.
   
    b. Abra el archivo de metadatos descargado y luego busque el nodo **AssertionConsumerService**.

    ![AssertionConsumerService](./media/topdesk-public-tutorial/ic790619.png "AssertionConsumerService")
   
    c. Copia el valor de **AssertionConsumerService**, péguelo en el cuadro de texto **Dirección URL de respuesta** de la sección **Configuración básica de SAML**.      
   
6. Lleve a cabo los siguientes pasos para crear un archivo de certificado:
    
    ![Certificate](./media/topdesk-public-tutorial/ic790606.png "Certificado")
    
    a. Abra el archivo de metadatos descargado de Azure Portal.
    
    b. Expanda el nodo **RoleDescriptor** cuyo **xsi:type** es **fed:ApplicationServiceType**.
    
    c. Copie el valor del nodo **X509Certificate** .
    
    d. Guarde el valor de **X509Certificate** copiado localmente en el equipo en un archivo.

7. En la sección **Public** (Público), haga clic en **Add** (Agregar).
    
    ![SAML Login (Inicio de sesión de SAML)](./media/topdesk-public-tutorial/ic790625.png "Inicio de sesión SAML")

8. En la página de diálogo del **SAML configuration assistant** (Asistente de configuración de SAML), realice los siguientes pasos:
    
    ![SAML Configuration Assistant (Asistente de configuración de SAML)](./media/topdesk-public-tutorial/ic790608.png "SAML configuration assistant")
    
    a. Para cargar el archivo de metadatos descargado de Azure Portal, en **Federation Metadata** (Metadatos de federación) haga clic en **Browse** (Examinar).

    b. Para cargar el archivo del certificado, en **Certificate (RSA)** (Certificado [RSA]), haga clic en **Browse** (Examinar).

    c. Para cargar el archivo de logotipo que obtuvo del equipo de soporte técnico de TOPdesk, en **Logo icon** (Icono de logotipo), haga clic en **Browse** (Examinar).

    d. En el cuadro de texto **User name attribute** (Atributo de nombre de usuario), escriba `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`.

    e. En el cuadro de texto **Display name** (Nombre para mostrar), escriba un nombre para su configuración.

    f. Haga clic en **Save**(Guardar).

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba brittasimon@yourcompanydomain.extension. Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a TOPdesk - Public.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **TOPdesk - Public**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **TOPdesk - Public**.

    ![Vínculo a TOPdesk - Public en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-topdesk---public-test-user"></a>Creación de un usuario de prueba de TOPdesk - Public

Para permitir que los usuarios de Azure AD inicien sesión en TOPdesk - Public, deben aprovisionarse en TOPdesk - Public. En el caso de TOPdesk - Public, el aprovisionamiento es una tarea manual.

### <a name="to-configure-user-provisioning-perform-the-following-steps"></a>Siga estos pasos para configurar el aprovisionamiento de usuario:

1. Inicie sesión en su sitio de la compañía de **TOPdesk - Public** como administrador.

2. En el menú de la parte superior, haga clic en **TOPdesk \> New \> Support Files \> Person** (TOPdesk > Nuevo > Archivos de soporte > Persona).
   
    ![Person](./media/topdesk-public-tutorial/ic790628.png "Person")

3. En el cuadro de diálogo Nueva persona, realice los pasos siguientes:
   
    ![Nueva persona](./media/topdesk-public-tutorial/ic790629.png "Nueva persona")
   
    a. Haga clic en la pestaña General.

    b. En el cuadro de texto **Apellidos**, escriba los apellidos del usuario, en este caso, Simon.
 
    c. Seleccione un **sitio** para la cuenta.
 
    d. Haga clic en **Save**(Guardar).

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de TOPdesk - Public ofrecida por TOPdesk - Public para aprovisionar cuentas de usuario de Azure AD.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de TOPdesk - Public en el panel de acceso, debería iniciar sesión automáticamente en la versión de TOPdesk - Public para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
