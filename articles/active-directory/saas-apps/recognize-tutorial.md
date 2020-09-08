---
title: 'Tutorial: integración de Azure Active Directory con Recognize | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Recognize.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: 86c2c7383f3850b0c51831f94e673c3941894a08
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88548903"
---
# <a name="tutorial-azure-active-directory-integration-with-recognize"></a>Tutorial: Integración de Azure Active Directory con Recognize

En este tutorial, obtendrá información sobre cómo integrar SCC LifeCycle con Azure Active Directory (Azure AD).
Integrar Recognize con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Recognize.
* Puede permitir que los usuarios inicien sesión automáticamente en Recognize (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Recognize, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en Recognize

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Recognize admite el inicio de sesión único iniciado por **SP**

## <a name="adding-recognize-from-the-gallery"></a>Adición de Recognize desde la galería

Para configurar la integración de Recognize en Azure AD, deberá agregar Recognize desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Recognize desde la galería, siga estos pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Recognize**, seleccione **Recognize** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![Recognize en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Recognize con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Recognize.

Para configurar y probar el inicio de sesión único de Azure AD con Recognize, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Recognize](#configure-recognize-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Recognize](#create-recognize-test-user)** : para tener un homólogo de Britta Simon en Recognize que esté vinculado a su representación en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Recognize, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Recognize**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si tiene el **archivo de metadatos del proveedor de servicios**, lleve a cabo los siguientes pasos:

    >[!NOTE]
    >Puede obtener el **archivo de metadatos del proveedor de servicios** en la sección **Configuración del inicio de sesión único de Recognize** del tutorial.

    a. Haga clic en **Cargar el archivo de metadatos**.

    ![Carga del archivo de metadatos](common/upload-metadata.png)

    b. Haga clic en el **logotipo de la carpeta** para seleccionar el archivo de metadatos y luego en **Cargar**.

    ![Elección del archivo de metadatos](common/browse-upload-metadata.png)

    c. Una vez que se haya cargado correctamente el archivo de metadatos, el valor de **Identificador** se rellena automáticamente en la sección Configuración básica de SAML.

    ![Información sobre dominio y direcciones URL de inicio de sesión único de Recognize](common/sp-identifier.png)

     En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://recognizeapp.com/<your-domain>/saml/sso`

    > [!Note]
    > Si el valor de **Identificador** no se rellena automáticamente, puede obtenerlo abriendo la dirección URL de metadatos del proveedor de servicios en la sección de configuración del inicio de sesión único como se explica más adelante en la sección **Configuración del inicio de sesión único de Recognize** de este tutorial. El valor de la dirección URL de inicio de sesión no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico de Recognize](mailto:support@recognizeapp.com) para obtener este valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Configurar Recognize**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-recognize-single-sign-on"></a>Configuración del inicio de sesión único de Recognize

1. En otra ventana del explorador web, inicie sesión en el inquilino de Recognize como administrador.

2. En la esquina superior derecha, haga clic en **Menú**. Vaya a **Company Admin** (Administrador de empresa).
   
    ![Configuración del inicio de sesión único en la aplicación](./media/recognize-tutorial/tutorial_recognize_000.png)

3. En la barra de navegación de la izquierda, haga clic en **Settings**(Configuración).
   
    ![Configuración del inicio de sesión único en la aplicación](./media/recognize-tutorial/tutorial_recognize_001.png)

4. Siga estos pasos en la sección **SSO Settings** (Configuración de SSO).
   
    ![Configuración del inicio de sesión único en la aplicación](./media/recognize-tutorial/tutorial_recognize_002.png)
    
    a. En **Enable SSO** (Habilitar SSO), seleccione **ON** (Activado).

    b. En el cuadro de texto **IDP Entity ID** (Identificador de la entidad de IdP), pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.
    
    c. En el cuadro de texto **SSO Target URL** (Dirección URL de destino de SSO), pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.
    
    d. En el cuadro de texto **SSO Target URL** (Dirección URL de destino de SSO), pegue el valor de **URL de cierre de sesión** que ha copiado de Azure Portal. 
    
    e. Abra el archivo descargado **Certificado (Base64)** en el Bloc de notas, copie su contenido en el Portapapeles y luego péguelo en el cuadro de texto **Certificado**.
    
    f. Haga clic en el botón **Guardar configuración**. 

5. Junto a la sección de **configuración de SSO**, copie la dirección URL en **Service Provider Metadata url** (Dirección URL de metadatos del proveedor de servicio).
   
    ![Configuración del inicio de sesión único en la aplicación](./media/recognize-tutorial/tutorial_recognize_003.png)

6. Abra el vínculo **Dirección URL de metadatos** en un explorador en blanco para descargar el documento de metadatos. Después, copie el valor de EntityDescriptor (entityID) del archivo y péguelo en el cuadro de texto **Identificador** en **Configuración básica de SAML** en Azure Portal.
    
    ![Configuración del inicio de sesión único en la aplicación](./media/recognize-tutorial/tutorial_recognize_004.png)

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

En esta sección, concederá acceso a Britta Simon a Recognize para que use el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **Recognize**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Recognize**.

    ![Vínculo a Recognize en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-recognize-test-user"></a>Creación de un usuario de prueba de Recognize

Para permitir que los usuarios de Azure AD inicien sesión en Recognize, deben aprovisionarse en Recognize. En el caso de Recognize, el aprovisionamiento es una tarea manual.

Esta aplicación no admite el aprovisionamiento SCIM, pero tiene una sincronización de usuario alternativa que aprovisiona a los usuarios. 

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en su sitio de la compañía de Recognize como administrador.

2. En la esquina superior derecha, haga clic en **Menú**. Vaya a **Company Admin** (Administrador de empresa).

3. En la barra de navegación de la izquierda, haga clic en **Settings**(Configuración).

4. En la sección **User Sync** (Sincronización de usuario), siga estos pasos.
   
    ![New User](./media/recognize-tutorial/tutorial_recognize_005.png "Nuevo usuario") (Usuario nuevo)
   
    a. En **Sync Enabled** (Sincronización habilitada), seleccione **ON** (Activado).
   
    b. En **Choose sync provider** (Elegir proveedor de sincronización), seleccione **Microsoft/Office 365**.
   
    c. Haga clic en **Run User Sync** (Ejecutar sincronización de usuario).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Recognize en el panel de acceso y debería iniciar sesión automáticamente en la versión de Recognize para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

