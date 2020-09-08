---
title: 'Tutorial: Integración de Azure Active Directory con Zscaler Internet Access Administrator | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Zscaler Internet Access Administrator.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/17/2019
ms.author: jeedes
ms.openlocfilehash: 0cbcfb65d7c7ac71b17e38dd72545a3d8fd2f162
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88545891"
---
# <a name="tutorial-azure-active-directory-integration-with-zscaler-internet-access-administrator"></a>Tutorial: Integración de Azure Active Directory con Zscaler Internet Access Administrator

En este tutorial, aprenderá a integrar Zscaler Internet Access Administrator con Azure Active Directory (Azure AD).
La integración de Zscaler Internet Access Administrator con Azure AD ofrece las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Zscaler Internet Access Administrator.
* Puede permitir que los usuarios inicien sesión automáticamente en Zscaler Internet Access Administrator (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Zscaler Internet Access Administrator, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Suscripción de Zscaler Internet Access Administrator

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Zscaler Internet Access Administrator admite el inicio de sesión único iniciado por el **punto de distribución de emisión**

## <a name="adding-zscaler-internet-access-administrator-from-the-gallery"></a>Adición de Zscaler Internet Access Administrator desde la galería

Para configurar la integración de Zscaler Internet Access Administrator con Azure AD, debe agregar Zscaler Internet Access Administrator desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Zscaler Internet Access Administrator desde la galería, realice los siguientes pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Zscaler Internet Access Administrator**, seleccione **Zscaler Internet Access Administrator** en el panel de resultados y luego haga clic en el botón **Agregar** para agregar la aplicación.

     ![Zscaler Internet Access Administrator en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Zscaler Internet Access Administrator con un usuario de prueba llamado **Britta Simon**.
Para que funcione el inicio de sesión único, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado en Zscaler Internet Access Administrator.

Para configurar y probar el inicio de sesión único de Azure AD con Zscaler Internet Access Administrator, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Zscaler Internet Access Administrator](#configure-zscaler-internet-access-administrator-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Zscaler Internet Access Administrator](#create-zscaler-internet-access-administrator-test-user)** : para tener un homólogo de Britta Simon en Zscaler Internet Access Administrator que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único en Azure AD con Zscaler Internet Access Administrator, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Zscaler Internet Access Administrator**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el botón **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Información de dominio y direcciones URL de inicio de sesión único de Zscaler Internet Access Administrator](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL según sus requisitos:

    | Identificador |
    |------------|
    | `https://admin.zscaler.net` |
    | `https://admin.zscalerone.net` |
    | `https://admin.zscalertwo.net` |
    | `https://admin.zscalerthree.net` |
    | `https://admin.zscloud.net` |
    | `https://admin.zscalerbeta.net` |

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL según sus requisitos:

    | URL de respuesta |
    |-----------|
    | `https://admin.zscaler.net/adminsso.do` |
    | `https://admin.zscalerone.net/adminsso.do` |
    | `https://admin.zscalertwo.net/adminsso.do` |
    | `https://admin.zscalerthree.net/adminsso.do` |
    | `https://admin.zscloud.net/adminsso.do` |
    | `https://admin.zscalerbeta.net/adminsso.do` |

5. La aplicación Zscaler Internet Access Administrator espera las aserciones de SAML en un formato específico. Configure las siguientes notificaciones para esta aplicación. Puede administrar los valores de estos atributos en la sección **User Attributes & Claims** (Atributos y notificaciones del usuario) de la página de integración de aplicaciones. En la **página Configurar el inicio de sesión único con SAML**, haga clic en el botón **Editar** para abrir el cuadro de diálogo **User Attributes & Claims** (Atributos y notificaciones del usuario).

    ![Vínculo de atributos](./media/zscaler-internet-access-administrator-tutorial/tutorial_zscaler-internet_attribute.png)

6. En la sección **Notificaciones del usuario** del cuadro de diálogo **Atributos de usuario**, configure el atributo Token SAML como muestra la imagen anterior y realice los siguientes pasos:

    | Nombre  | Atributo de origen  |
    | ---------| ------------ |
    | Role     | user.assignedroles |

    a. Haga clic en **Agregar nueva notificación** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

    ![imagen](./common/new-save-attribute.png)
    
    ![imagen](./common/new-attribute-details.png)

    b. En la lista **Atributo de origen**, seleccione el valor del atributo.

    c. Haga clic en **Aceptar**.

    d. Haga clic en **Save**(Guardar).

    > [!NOTE]
    > Haga clic [aquí](https://docs.microsoft.com/azure/active-directory/active-directory-enterprise-app-role-management) para saber cómo configurar el valor Role en Azure AD.

7. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

8. En la sección **Set up Zscaler Internet Access Administrator** (Configurar Zscaler Internet Access Administrator), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-zscaler-internet-access-administrator-single-sign-on"></a>Configuración del inicio de sesión único de Zscaler Internet Access Administrator

1. En otra ventana del explorador web, inicie sesión en la interfaz de usuario de Zscaler Internet Access Administrator.

2. Vaya a **Administración > Administración de administradores**, realice los pasos siguientes y haga clic en Guardar:

    ![Administración](./media/zscaler-internet-access-administrator-tutorial/AdminSSO.png "Administración")

    a. Marque **Enable SAML Authentication** (Habilitar autenticación SAML).

    b. Haga clic en **Cargar** para cargar el certificado de firma de SAML de Azure que ha descargado desde Azure Portal en el **Certificado SSL público**.

    c. Opcionalmente, para mayor seguridad, agregue los datos del **Emisor** para verificar el emisor de la respuesta SAML.

3. En la interfaz de usuario del administrador, realice los pasos siguientes:

    ![Administración](./media/zscaler-internet-access-administrator-tutorial/ic800207.png)

    a. Mantenga el puntero sobre el menú **Activación** situado cerca de la parte inferior izquierda.

    b. Haga clic en **Activar**.

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

En esta sección, permitirá que Britta Simon use el inicio de sesión único de Azure concediéndole acceso a Zscaler Internet Access Administrator.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Zscaler Internet Access Administrator**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, escriba y seleccione **Zscaler Internet Access Administrator**.

    ![Vínculo a Zscaler Internet Access Administrator en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-zscaler-internet-access-administrator-test-user"></a>Creación de un usuario de prueba en Zscaler Internet Access Administrator

El objetivo de esta sección es crear un usuario llamado Britta Simon en Zscaler Internet Access Administrator. Zscaler Internet Access no admite el aprovisionamiento Just-In-Time para el inicio de sesión único del administrador. Debe crear una cuenta de administrador manualmente.
Para instrucciones sobre cómo crear una cuenta de administrador, consulte la documentación de Zscaler:

https://help.zscaler.com/zia/adding-admins

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Zscaler Internet Access Administrator en el panel de acceso, debería iniciar sesión automáticamente en la interfaz de usuario de Zscaler Internet Access Administrator para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
