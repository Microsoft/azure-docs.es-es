---
title: 'Tutorial: Integración de Azure Active Directory con Clear Review | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Clear Review.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 8264159a-11a2-4a8c-8285-4efea0adac8c
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 04/16/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: ed7900373c5d8897fd360c708434d96e5dfbc73d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87034693"
---
# <a name="tutorial-azure-active-directory-integration-with-clear-review"></a>Tutorial: Integración de Azure Active Directory con Clear Review

En este tutorial, aprenderá a integrar Clear Review con Azure Active Directory (Azure AD).
Integrar Clear Review con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Clear Review.
* Puede permitir que los usuarios inicien sesión automáticamente en Clear Review (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Clear Review, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción habilitada para el inicio de sesión único en Clear Review

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Clear Review admite el inicio de sesión único iniciado por **SP e IDP**

## <a name="adding-clear-review-from-the-gallery"></a>Incorporación de Clear Review desde la galería

Para configurar la integración de Clear Review en Azure AD, es preciso agregar Clear Review desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Clear Review desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Clear Review**, seleccione **Clear Review** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![Clear Review en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Clear Review con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Clear Review.

Para configurar y probar el inicio de sesión único de Azure AD con Clear Review, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único en Clear Review](#configure-clear-review-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Clear Review](#create-clear-review-test-user)** : para tener un homólogo de Britta Simon en Clear Review que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Clear Review, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Clear Review**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice los siguientes pasos:

    ![Información sobre dominio y direcciones URL de inicio de sesión único de Clear Review](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<customer name>.clearreview.com/sso/metadata/`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<customer name>.clearreview.com/sso/acs/`

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    ![Información sobre dominio y direcciones URL de inicio de sesión único de Clear Review](common/metadata-upload-additional-signon.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<customer name>.clearreview.com`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico al cliente de Clear Review](https://clearreview.com/contact/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

6. La aplicación Clear Review espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de pantalla muestra la lista de atributos predeterminados, donde **nameidentifier** se asigna con **user.userprincipalname**. La aplicación Clear Review espera que **nameidentifier** se asigne con **user.mail**, por lo que debe editar la asignación de atributos haciendo clic en el icono **Editar** y cambiar esa asignación.

    ![imagen](common/edit-attribute.png)

7. En el cuadro de diálogo **Atributos y notificaciones de usuario**, realice estos pasos:

    a. Haga clic en el **icono Editar** situado a la derecha del **valor de identificador de nombre**.

    ![imagen](./media/clearreview-tutorial/attribute02.png)

    ![imagen](./media/clearreview-tutorial/attribute01.png)

    b. En la lista **Atributo de origen**, seleccione el valor de atributo **user.mail** de esa fila.

    c. Haga clic en **Save**(Guardar).

8. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

9. En la sección **Set up Clear Review** (Configurar Clear Review), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-clear-review-single-sign-on"></a>Configuración del inicio de sesión único en Clear Review

1. Para configurar el inicio de sesión único en el lado de **Clear Review** lateral, abra el portal de **Clear Review** con credenciales de administrador.

2. Seleccione **Administrador** en el panel de navegación izquierdo.

    ![Botón Configurar inicio de sesión único](./media/clearreview-tutorial/tutorial_clearreview_app_admin1.png)

3. En la sección **Integraciones** situada en la parte inferior de la página, haga clic en el botón **Cambiar** situado a la derecha de **Configuración de inicio de sesión único**.

    ![Botón Configurar inicio de sesión único](./media/clearreview-tutorial/tutorial_clearreview_app_admin2.png)

4. Siga estos pasos en la página **Configuración de inicio de sesión único**:

    ![Botón Configurar inicio de sesión único](./media/clearreview-tutorial/tutorial_clearreview_app_admin3.png)

    a. En el cuadro de texto **URL del emisor**, pegue el valor de **Identificador de Azure AD**  que ha copiado de Azure Portal.

    b. En el cuadro de texto **SAML Endpoint** (Punto de conexión SAML), pegue el valor de la **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.  

    c. En el cuadro de texto **SLO Endpoint** (Punto de conexión de SLO), pegue el valor de la **Dirección URL de cierre de sesión** que ha copiado de Azure Portal.  

    d. Abra el certificado descargado en el Bloc de notas y pegue el contenido en el cuadro de texto **X.509 Certificate** (Certificado X.509).   

    e. Haga clic en **Save**(Guardar).

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Clear Review.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Clear Review**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Clear Review**.

    ![Vínculo a Clear Review en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-clear-review-test-user"></a>Creación de un usuario de prueba de Clear Review

En esta sección, se crea un usuario denominado Britta Simon en Clear Review. Trabaje con el [equipo de soporte técnico de Clear Review](https://clearreview.com/contact/) para agregar los usuarios a la plataforma de Clear Review.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Clear Review en el panel de acceso, debería iniciar sesión automáticamente en la versión de Clear Review para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

