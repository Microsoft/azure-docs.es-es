---
title: 'Tutorial: Integración de Azure Active Directory con PageDNA | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y PageDNA.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/03/2019
ms.author: jeedes
ms.openlocfilehash: 543085eaea993f6e464a501ffc93befd7589a009
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88554148"
---
# <a name="tutorial-azure-active-directory-integration-with-pagedna"></a>Tutorial: Integración de Azure Active Directory con PageDNA

En este tutorial, obtendrá información sobre cómo integrar PageDNA con Azure Active Directory (Azure AD).

La integración de PageDNA con Azure AD le proporciona las siguientes ventajas:

* En Azure AD, puede controlar quién tiene acceso a PageDNA.
* Puede permitir que los usuarios inicien sesión automáticamente en PageDNA (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Para más información sobre la integración de aplicaciones de software como servicio (SaaS) con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con PageDNA, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Suscripción de PageDNA con el inicio de sesión único habilitado.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba e integrar PageDNA con Azure AD.

PageDNA admite las siguientes características:

* Inicio de sesión único (SSO) iniciado por SP.

* Aprovisionamiento de usuarios Just-In-Time.

## <a name="add-pagedna-from-the-azure-marketplace"></a>Adición de PageDNA desde Azure Marketplace

Para configurar la integración de PageDNA en Azure AD, deberá agregar PageDNA desde Azure Marketplace a la lista de aplicaciones SaaS administradas:

1. Inicie sesión en [Azure Portal](https://portal.azure.com?azure-portal=true).
1. En el panel izquierdo, seleccione **Azure Active Directory**.

    ![Opción de Azure Active Directory](common/select-azuread.png)

1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Panel Aplicaciones empresariales](common/enterprise-applications.png)

1. Para agregar una aplicación nueva, en la parte superior del panel, seleccione **+ Nueva aplicación**.

    ![Opción Nueva aplicación](common/add-new-app.png)

1. En el cuadro de búsqueda, escriba **PageDNA**. En los resultados de búsqueda, seleccione **PageDNA** y, a continuación, seleccione **Agregar** para agregar la aplicación.

    ![PageDNA en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con PageDNA con un usuario de prueba llamado **Britta Simon**. Para que el inicio de sesión único funcione, es preciso establecer un vínculo entre un usuario de Azure AD y el usuario relacionado de PageDNA.

Para configurar y probar el inicio de sesión único de Azure AD con PageDNA, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** , para permitir que los usuarios utilicen esta característica.
1. **[Configuración del inicio de sesión único de PageDNA](#configure-pagedna-single-sign-on)** : para configurar los valores de inicio de sesión único en la aplicación.
1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
1. **[Creación de un usuario de prueba de PageDNA](#create-a-pagedna-test-user)** para que haya un usuario llamado Britta Simon en PageDNA que esté vinculado a ese mismo nombre de usuario de Azure AD.
1. **[Prueba del inicio de sesión único](#test-single-sign-on)** , para comprobar si funciona la configuración.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con PageDNA, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **PageDNA**, seleccione **Inicio de sesión único**.

    ![Configuración de la opción de inicio de sesión único](common/select-sso.png)

1. En el panel **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

1. En la página **Configurar el inicio de sesión único con SAML**, seleccione **Editar** (icono de lápiz) para abrir el panel **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En el panel **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de PageDNA](common/sp-identifier.png)

    1. En el cuadro de texto **Dirección URL de inicio de sesión**, escriba una dirección URL con alguno de los siguientes patrones:

        ```https
        https://stores.pagedna.com/<your site>
        https://<your domain>
        https://<your domain>/<your site>
        https://www.nationsprint.com/<your site>
        ```

    1. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con alguno de los siguientes patrones:

        ```https
        https://stores.pagedna.com/<your site>/saml2ep.cgi
        https://www.nationsprint.com/<your site>/saml2ep.cgi
        ```

    > [!NOTE]
    > Estos valores no son reales. Debe actualizarlos con la dirección URL de inicio de sesión y el identificador reales. Póngase en contacto con el [equipo de soporte técnico de PageDNA](mailto:success@pagedna.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en el panel **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione **Descargar** para descargar el **Certificado (sin procesar)** de las opciones proporcionadas y guárdelo en el equipo.

    ![Opción de descarga del certificado (sin procesar)](common/certificateraw.png)

1. En la sección **Configurar PageDNA**, copie la dirección o direcciones URL que necesita:

   * **Dirección URL de inicio de sesión**
   * **Identificador de Azure AD**
   * **Dirección URL de cierre de sesión**

    ![Copia de las direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="configure-pagedna-single-sign-on"></a>Configuración del inicio de sesión único de PageDNA

Para configurar el inicio de sesión único en PageDNA, envíe el certificado (sin procesar) descargado y las direcciones URL apropiadas copiadas de Azure Portal al [equipo de soporte técnico de PageDNA](mailto:success@pagedna.com). El equipo de PageDNA se asegurará de que la conexión de inicio de sesión único de SAML esté establecida correctamente en ambos lados.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, creará un usuario de prueba en Azure Portal llamado Britta Simon.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**   > **Usuarios** > **Todos los usuarios**.

    ![Opciones "Usuarios" y "Todos los usuarios"](common/users.png)

1. En la parte superior de la pantalla, seleccione **+ Nuevo usuario**.

    ![Nueva opción de usuario](common/new-user.png)

1. En el panel **Usuario**, realice los pasos siguientes:

    ![Panel Usuario](common/user-properties.png)

    1. En el cuadro **Nombre**, escriba **BrittaSimon**.
  
    1. En el cuadro **Nombre de usuario**, escriba **BrittaSimon\@\<yourcompanydomain>.\<extension>** . Por ejemplo, **BrittaSimon\@contoso.com**.

    1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.

    1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, para permitir a Britta Simon usar el inicio de sesión único de Azure, le concederá acceso a PageDNA.

1. En Azure Portal, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones** > **PageDNA**.

    ![Panel Aplicaciones empresariales](common/enterprise-applications.png)

1. En la lista de aplicaciones, seleccione **PageDNA**.

    ![PageDNA en la lista de aplicaciones](common/all-applications.png)

1. En el panel izquierdo, seleccione **ADMINISTRAR** y **Usuarios y grupos**.

    ![Opción "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **+ Agregar usuario** y, después, seleccione **Usuarios y grupos** en el panel **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

1. En el panel **Usuarios y grupos**, seleccione **Britta Simon** en la lista **Usuarios** y, luego, elija **Seleccionar** en la parte inferior del panel.

1. Si espera algún valor de rol en la aserción de SAML, en el panel **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. Elija **Seleccionar** en la parte inferior del panel.

1. En el panel **Agregar asignación**, seleccione **Asignar**.

### <a name="create-a-pagedna-test-user"></a>Creación de un usuario de prueba en PageDNA

Ahora se crea un usuario llamado Britta Simon en PageDNA. No tiene que hacer nada para crear este usuario. PageDNA admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. Si el usuario llamado Britta Simon no existe en PageDNA, se crea uno después de la autenticación.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el portal Aplicaciones.

Al seleccionar **PageDNA** en el portal Aplicaciones, debería iniciar sesión automáticamente en la suscripción de PageDNA para la que configuró el inicio de sesión único. Para más información acerca del portal Aplicaciones, consulte [Access and use apps on the My Apps portal](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction) (Uso de aplicaciones y acceso a ellas en el portal Aplicaciones).

## <a name="additional-resources"></a>Recursos adicionales

* [Lista de tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

* [Inicio de sesión único en aplicaciones en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

* [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

