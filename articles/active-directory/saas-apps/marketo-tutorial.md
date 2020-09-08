---
title: 'Tutorial: Integración de Azure Active Directory con Marketo | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Marketo.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/19/2019
ms.author: jeedes
ms.openlocfilehash: c1b8874d8813d6200c915778841c26e77b02e434
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88554930"
---
# <a name="tutorial-azure-active-directory-integration-with-marketo"></a>Tutorial: integración de Azure Active Directory con Marketo

En este tutorial, aprenderá a integrar Marketo con Azure Active Directory (Azure AD).
Integrar Marketo con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Marketo.
* Puede permitir que los usuarios inicien sesión automáticamente en Marketo (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Marketo, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para inicio de sesión único en Marketo

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Marketo admite SSO iniciado por **IDP**

## <a name="adding-marketo-from-the-gallery"></a>Incorporación de Marketo desde la galería

Para configurar la integración de Marketo en Azure AD, deberá agregar Marketo desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Marketo desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Marketo**, seleccione **Marketo** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![Marketo en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Marketo con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Marketo.

Para configurar y probar el inicio de sesión único de Azure AD con Marketo, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único en Marketo](#configure-marketo-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba en Marketo](#create-marketo-test-user)** : el objetivo es tener un homólogo de Britta Simon en Marketo que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Marketo, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Marketo**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Marketo](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://saml.marketo.com/sp`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://login.marketo.com/saml/assertion/\<munchkinid\>`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Póngase en contacto con el [equipo de soporte técnico para clientes de Marketo](https://investors.marketo.com/contactus.cfm) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Azure Portal,**  (Configurar Azure Portal, ), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-marketo-single-sign-on"></a>Configuración del inicio de sesión único en Marketo

1. Para obtener el identificador de Munchkin de la aplicación, inicie sesión con credenciales de administrador de Marketo y realizar las siguientes acciones:
   
    a. Inicie sesión en la aplicación Marketo con credenciales de administrador.
   
    b. Haga clic en el botón **Administrador** en el panel de navegación superior.
   
    ![Configurar inicio de sesión único](./media/marketo-tutorial/tutorial_marketo_06.png) 
   
    c. Vaya al menú Integración y haga clic en el **vínculo de Munchkin**.
   
    ![Configurar inicio de sesión único](./media/marketo-tutorial/tutorial_marketo_11.png)
   
    d. Copie el identificador de Munchkin mostrado en la pantalla y complete la dirección URL de respuesta en el asistente para configuración de Azure AD.
   
    ![Configurar inicio de sesión único](./media/marketo-tutorial/tutorial_marketo_12.png) 

2. Para configurar el inicio de sesión único en la aplicación, siga estos pasos:
   
    a. Inicie sesión en la aplicación Marketo con credenciales de administrador.
   
    b. Haga clic en el botón **Administrador** en el panel de navegación superior.
   
    ![Configurar inicio de sesión único](./media/marketo-tutorial/tutorial_marketo_06.png) 
   
    c. Vaya al menú Integración y haga clic en **Inicio de sesión único**.
   
    ![Configurar inicio de sesión único](./media/marketo-tutorial/tutorial_marketo_07.png) 
   
    d. Para habilitar la configuración de SAML, haga clic en el botón **Editar**.
   
    ![Configurar inicio de sesión único](./media/marketo-tutorial/tutorial_marketo_08.png) 
   
    e. Ya está la configuración de inicio de sesión único **habilitada**.
   
    f. Pegue el valor de **Identificador de Azure AD**  en el cuadro de texto **Issuer ID** (Identificador del emisor).
   
    g. En el cuadro de texto **Entity ID** (Id. de entidad), escriba la dirección URL como `http://saml.marketo.com/sp`.
   
    h. Seleccione la ubicación del identificador de usuario como **elemento Identificador de nombre**.
   
    ![Configurar inicio de sesión único](./media/marketo-tutorial/tutorial_marketo_09.png)
   
    > [!NOTE]
    > Si su identificador de usuario no es el valor UPN, cambie el valor en la pestaña Atributo.
   
    i. Cargue el certificado que ha descargado del Asistente para la configuración de Azure AD. **Guarde** la configuración.
   
    j. Edite la configuración delas páginas de redireccionamiento.
   
    k. Pegue el valor de **Dirección URL de inicio de sesión** en el cuadro de texto **Login URL** (URL de inicio de sesión).
   
    l. Pegue el valor de **URL de cierre de sesión** en el cuadro de texto **Logout URL** (URL de cierre de sesión).
   
    m. En **URL del error**, copie la **dirección URL de la instancia de Marketo** y haga clic en el botón **Guardar** para guardar la configuración.
   
    ![Configurar inicio de sesión único](./media/marketo-tutorial/tutorial_marketo_10.png)

3. Para habilitar el SSO para los usuarios, complete las siguientes acciones:
   
    a. Inicie sesión en la aplicación Marketo con credenciales de administrador.
   
    b. Haga clic en el botón **Administrador** en el panel de navegación superior.
   
    ![Configurar inicio de sesión único](./media/marketo-tutorial/tutorial_marketo_06.png) 
   
    c. Desplácese hasta el menú **Seguridad** y haga clic en **Login Settings** (Configuración de inicio de sesión).
   
    ![Configurar inicio de sesión único](./media/marketo-tutorial/tutorial_marketo_13.png)
   
    d. Active la opción **Require SSO** (Requerir SSO) y **guarde** la configuración.
   
    ![Configurar inicio de sesión único](./media/marketo-tutorial/tutorial_marketo_14.png)

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Marketo.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **Marketo**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Marketo**.

    ![Vínculo a Marketo en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-marketo-test-user"></a>Creación de un usuario de prueba en Marketo

En esta sección, creará un usuario llamado Britta Simon en Marketo. Siga estos pasos para crear un usuario en la plataforma Marketo.

1. Inicie sesión en la aplicación Marketo con credenciales de administrador.

2. Haga clic en el botón **Administrador** en el panel de navegación superior.
   
    ![Configurar inicio de sesión único](./media/marketo-tutorial/tutorial_marketo_06.png) 

3. Desplácese hasta el menú **Security** (Seguridad) y haga clic en **Users & Roles** (Usuarios y roles).
   
    ![Configurar inicio de sesión único](./media/marketo-tutorial/tutorial_marketo_19.png)  

4. Haga clic en el vínculo **Invite New User** (Invitar a un usuario nuevo) en la pestaña Users (Usuarios).
   
    ![Configurar inicio de sesión único](./media/marketo-tutorial/tutorial_marketo_15.png) 

5. En el asistente Invitar a un usuario nuevo, rellene la siguiente información.
   
    a. Especifique la dirección de **Correo electrónico** del usuario en el cuadro de texto.
   
    ![Configurar inicio de sesión único](./media/marketo-tutorial/tutorial_marketo_16.png)
   
    b. Escriba el **Nombre** en el cuadro de texto.
   
    c. Escriba los **Apellidos** en el cuadro de texto.
   
    d. Haga clic en **Siguiente**.

6. En la pestaña **Permissions** (Permisos) seleccione los **roles de usuario** y haga clic en **Next** (Siguiente).
   
    ![Configurar inicio de sesión único](./media/marketo-tutorial/tutorial_marketo_17.png)
7. Haga clic en el botón **Send** (Enviar) para enviar la invitación del usuario.
   
    ![Configurar inicio de sesión único](./media/marketo-tutorial/tutorial_marketo_18.png)

8. El usuario recibe la notificación de correo electrónico y tiene que hacer clic en el vínculo y cambiar la contraseña para activar la cuenta. 

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Marketo en el Panel de acceso, debería iniciar sesión automáticamente en la versión de Marketo para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

