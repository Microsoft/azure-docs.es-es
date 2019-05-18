---
title: 'Tutorial: Integración de Azure Active Directory con Everbridge | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Everbridge.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 58d7cd22-98c0-4606-9ce5-8bdb22ee8b3e
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 04/18/2019
ms.author: jeedes
ms.openlocfilehash: f8dd11e7fb0b9fda0e0f1c7d3f794f6bfd766cdf
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65231454"
---
# <a name="tutorial-azure-active-directory-integration-with-everbridge"></a>Tutorial: Integración de Azure Active Directory con Everbridge

En este tutorial, aprenderá a integrar Everbridge con Azure Active Directory (Azure AD).
Al integrar Everbridge con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Everbridge.
* Permitir que los usuarios inicien sesión automáticamente en Everbridge con sus cuentas de Azure AD. Este control de acceso se llama inicio de sesión único (SSO).
* Administre sus cuentas en una ubicación central mediante Azure Portal.
Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con Everbridge, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción Everbridge que usa el inicio de sesión único.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Everbridge admite el inicio de sesión único basado en IDP.

## <a name="add-everbridge-from-the-azure-marketplace"></a>Adición de Everbridge desde Azure Marketplace

Para configurar la integración de Everbridge en Azure AD, agregue Everbridge desde Azure Marketplace a su lista de aplicaciones SaaS administradas.

Para agregar Everbridge desde Azure Marketplace, siga estos pasos.

1. En el panel de navegación izquierdo de [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una aplicación nueva, en la parte superior del cuadro de diálogo, seleccione **Nueva aplicación**.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Everbridge**. Seleccione **Everbridge** en el panel de resultados y luego **Agregar**.

     ![Everbridge en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, va a configurar y probar el inicio de sesión único de Azure AD con Everbridge con el usuario de prueba Britta Simon.
Para que el inicio de sesión único funcione, establezca una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Everbridge.

Para configurar y probar el inicio de sesión único de Azure AD con EverBridge, complete los siguientes bloques de creación:

- [Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on), para permitir que los usuarios utilicen esta característica.
- [Configuración de Everbridge como inicio de sesión único del portal de administrador de Everbridge](#configure-everbridge-as-everbridge-manager-portal-single-sign-on) para configurar los valores de inicio de sesión único en la aplicación.
- [Configuración de Everbridge como inicio de sesión único del portal de miembro de Everbridge](#configure-everbridge-as-everbridge-member-portal-single-sign-on) para configurar los valores de inicio de sesión único en la aplicación.
- [Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user), para probar el inicio de sesión único de Azure AD con Britta Simon.
- [Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user), para permitir que Britta Simon use el inicio de sesión único de Azure AD.
- [Creación de un usuario de prueba para Everbridge](#create-an-everbridge-test-user) para tener un homólogo de Britta Simon en Everbridge que esté vinculado a su representación en Azure AD.
- [Prueba del inicio de sesión único](#test-single-sign-on), para comprobar si funciona la configuración.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Everbridge, realice los pasos siguientes.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Everbridge**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, seleccione **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

    >[!NOTE]
    >Configure la aplicación como portal administrador *o* como portal miembro tanto en Azure Portal como en el portal Everbridge.

4. Para configurar la aplicación **Everbridge** como **portal administrador de Everbridge**, en la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información sobre dominio y direcciones URL de inicio de sesión único de Everbridge](common/idp-intiated.png)

     a. En el cuadro **Identificador**, escriba una dirección URL con el formato `https://sso.everbridge.net/<API_Name>`.

    b. En el cuadro **URL de respuesta**, escriba una dirección URL con el formato `https://manager.everbridge.net/saml/SSO/<API_Name>/alias/defaultAlias`.

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de identificador y URL de respuesta. Póngase en contacto con el [equipo de soporte técnico de Everbridge](mailto:support@everbridge.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. Para configurar la aplicación **Everbridge** como **portal miembro de Everbridge**, en la sección **Configuración básica de SAML**, siga estos pasos:

  * Después, realice los siguientes pasos si quiere configurar la aplicación en modo iniciado por IDP:

     ![Información sobre dominio y direcciones URL de inicio de sesión único de Everbridge para el modo iniciado por IDP](common/idp-intiated.png)

     a. En el cuadro **Identificador**, escriba una dirección URL con el formato `https://sso.everbridge.net/<API_Name>/<Organization_ID>`.

    b. En el cuadro **URL de respuesta**, escriba una dirección URL con el formato `https://member.everbridge.net/saml/SSO/<API_Name>/<Organization_ID>/alias/defaultAlias`.

   * Seleccione **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por SP:

     ![Información sobre dominio y direcciones URL de inicio de sesión único de Everbridge para modo iniciado por SP](common/both-signonurl.png)

      a. En el cuadro **URL de inicio de sesión**, escriba una dirección URL con el formato `https://member.everbridge.net/saml/login/<API_Name>/<Organization_ID>/alias/defaultAlias?disco=true`.

     > [!NOTE]
     > Estos valores no son reales. Actualice estos valores con los valores de identificador y de direcciones URL de inicio de sesión y de respuesta reales. Póngase en contacto con el [equipo de soporte técnico de Everbridge](mailto:support@everbridge.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione **Descargar** para descargar el archivo **XML de metadatos de federación** . Guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

7. En la sección **Configurar Everbridge**, copie las direcciones URL adecuadas según sus necesidades:

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    - URL de inicio de sesión
    - Identificador de Azure AD
    - URL de cierre de sesión

### <a name="configure-everbridge-as-everbridge-manager-portal-single-sign-on"></a>Configuración de Everbridge como inicio de sesión único del portal de administrador de Everbridge

Para configurar el inicio de sesión único en **Everbridge** como **portal administrador de Everbridge**, siga estos pasos.
 
1. En otra ventana del explorador web, inicie sesión en Everbridge como administrador.

1. En el menú de la parte superior, seleccione "**Settings**" (Configuración). En **Security** (Security), seleccione **Single Sign-On** (Inicio de sesión único).
   
     ![Configurar inicio de sesión único](./media/everbridge-tutorial/tutorial_everbridge_002.png)
   
      a. En el cuadro **Name** (Nombre), escriba el nombre del proveedor de identificador. Un ejemplo es el nombre de su empresa.
   
     b. En el cuadro **API Name** (Nombre de la API), escriba el nombre de la API.
   
     c. Seleccione **Choose File** (Elegir archivo) para cargar el archivo de metadatos que descargó de Azure Portal.
   
     d. En **SAML Identity Location** (Ubicación de identidad de SAML), seleccione **Identity is in the NameIdentifier element of the Subject statement** (La identidad está en el elemento NameIdentifier de la instrucción de asunto).
   
     e. En el cuadro **Identity provider Login URL** (URL de inicio de sesión del proveedor de identidades), pegue la **dirección URL de inicio de sesión** que copió de Azure Portal.
   
     f. En **Service Provider Initiated Request Binding** (Vinculación de solicitud iniciada del proveedor de servicios), seleccione **HTTP Redirect** (Redirección HTTP).

     g. Seleccione **Guardar**.

### <a name="configure-everbridge-as-everbridge-member-portal-single-sign-on"></a>Configuración de Everbridge como inicio de sesión único del portal de miembro de Everbridge

Para configurar el inicio de sesión único en **Everbridge** como **portal miembro de Everbridge**, tiene que enviar el **archivo XML de metadatos de federación** descargado al [equipo de soporte técnico de Everbridge](mailto:support@everbridge.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

Para crear el usuario de prueba Britta Simon en Azure Portal, siga estos pasos.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory** > **Usuarios** > **Todos los usuarios**.

    ![Vínculos Usuarios y Todos los usuarios](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En el cuadro de diálogo **Usuario**, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

     a. En el cuadro **Nombre**, escriba **BrittaSimon**.
  
    b. En el cuadro **Nombre de usuario**, escriba `brittasimon@yourcompanydomain.extension`. Un ejemplo es BrittaSimon@contoso.com.

    c. Active la casilla **Show Password** (Mostrar contraseña). Anote el valor que se muestra en el cuadro **Contraseña**.

    d. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

Habilite a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Everbridge.

1. En Azure Portal, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones** >**Everbridge**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Everbridge**.

    ![Vínculo a Everbridge en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo Usuarios y grupos](common/users-groups-blade.png)

4. Seleccione **Agregar usuario**. En el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Cuadro de diálogo Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon**. Elija **Seleccionar** en la parte inferior de la pantalla.

6. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. Elija **Seleccionar** en la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

### <a name="create-an-everbridge-test-user"></a>Creación de un usuario de prueba para EverBridge

En esta sección, creará un usuario de prueba llamado Britta Simon en Everbridge. Para agregar usuarios en la plataforma Everbridge, colabore con el [equipo de soporte técnico de Everbridge](mailto:support@everbridge.com). Los usuarios se tienen que crear y activar en Everbridge antes de usar el inicio de sesión único. 

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

Pruebe la configuración de inicio de sesión único de Azure AD mediante el panel de acceso.

Al seleccionar el icono de Everbridge en el panel de acceso, debería iniciar sesión automáticamente en la cuenta de Everbridge para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)
- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)
- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

