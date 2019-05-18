---
title: 'Tutorial: Integración de Azure Active Directory con Zscaler Beta | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Zscaler Beta.
services: active-directory
documentationCenter: na
author: jeevansd
manager: daveba
ms.reviewer: barbkess
ms.assetid: 56b846ae-a1e7-45ae-a79d-992a87f075ba
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 04/24/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: f31361dc3d7e24092677f1a78b2c405ae84578ed
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230061"
---
# <a name="tutorial-azure-active-directory-integration-with-zscaler-beta"></a>Tutorial: Integración de Azure Active Directory con Zscaler Beta

En este tutorial, aprenderá a integrar Zscaler Beta con Azure Active Directory (Azure AD).
Al integrar Zscaler Beta con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Zscaler Beta.
* Permitir que los usuarios inicien sesión automáticamente en Zscaler Beta con sus cuentas de Azure AD. Este control de acceso se llama inicio de sesión único (SSO).
* Administrar las cuentas en una ubicación central mediante Azure Portal.

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con Zscaler Beta, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción de Zscaler Beta que use el inicio de sesión único.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Zscaler Beta admite el inicio de sesión único iniciado por SP.
* Zscaler Beta admite el aprovisionamiento de usuarios Just-In-Time.

## <a name="add-zscaler-beta-from-the-azure-marketplace"></a>Adición de Zscaler Beta desde Azure Marketplace

Para configurar la integración de Zscaler Beta en Azure AD, deberá agregar Zscaler Beta desde Azure Marketplace a la lista de aplicaciones SaaS administradas.

Para agregar Zscaler Beta desde Azure Marketplace, siga estos pasos.

1. En el panel de navegación izquierdo de [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una aplicación nueva, en la parte superior del cuadro de diálogo, seleccione **Nueva aplicación**.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Zscaler Beta**. Seleccione **Zscaler Beta** en el panel de resultados y, a continuación, seleccione **Agregar**.

     ![Zscaler Beta en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Zscaler Beta con un usuario de prueba llamado Britta Simon.
Para que el inicio de sesión único funcione, establezca una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Zscaler Beta.

Para configurar y probar el inicio de sesión único de Azure AD con Zscaler Beta, debe completar los siguientes bloques de creación:

- [Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on), para permitir que los usuarios utilicen esta característica.
- [Configuración del inicio de sesión único de Zscaler Beta:](#configure-zscaler-beta-single-sign-on) para configurar el inicio de sesión único en la aplicación.
- [Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user), para probar el inicio de sesión único de Azure AD con Britta Simon.
- [Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user), para permitir que Britta Simon use el inicio de sesión único de Azure AD.
- [Creación de un usuario de prueba de Zscaler Beta](#create-a-zscaler-beta-test-user): para tener un homólogo de Britta Simon en Zscaler Beta vinculado a la representación del usuario en Azure AD.
- [Prueba del inicio de sesión único](#test-single-sign-on), para comprobar si funciona la configuración.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Zscaler Beta, realice los pasos siguientes.

1. En la página de integración de aplicaciones de **Zscaler Beta** de [Azure Portal](https://portal.azure.com/), seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, seleccione **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Zscaler Beta](common/sp-intiated.png)

    - En el cuadro de texto **Dirección URL de inicio de sesión**, escriba la dirección URL con la que los usuarios inician sesión en la aplicación Zscaler Beta.

    > [!NOTE]
    > Este valor no es real. Actualice el valor con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico de Zscaler Beta](https://www.zscaler.com/company/contact) para obtener este valor.

5. La aplicación Zscaler Beta espera las aserciones de SAML en un formato específico. Debe agregar asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados. Seleccione  **Editar** para abrir el cuadro de diálogo **Atributos de usuario** .

    ![Cuadro de diálogo Atributos de usuario](common/edit-attribute.png)

6. La aplicación Zscaler Beta espera que se usen algunos atributos más en la respuesta de SAML. En la sección **Notificaciones del usuario** del cuadro de diálogo **Atributos de usuario**, realice estos pasos para agregar el atributo del token de SAML como se muestra en la tabla siguiente.
    
    | NOMBRE | Atributo de origen | 
    | ---------------| --------------- |
    | memberOf  | user.assignedroles |

     a. Seleccione **Agregar nueva notificación** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

    ![Cuadro de diálogo Notificaciones del usuario](common/new-save-attribute.png)

    ![Cuadro de diálogo Administrar las notificaciones del usuario](common/new-attribute-details.png)

    b. En el cuadro **Nombre**, escriba el nombre de atributo que se muestra para la fila.

    c. Deje el cuadro **Espacio de nombres** en blanco.

    d. En **Origen**, seleccione **Atributo**.

    e. En la lista **Atributo de origen**, escriba el valor de atributo que se muestra para esa fila.

    f. Seleccione **Aceptar**.

    g. Seleccione **Guardar**.

    > [!NOTE]
    > Para obtener información sobre cómo configurar roles en Azure AD, consulte [Configuración de la notificación de rol](https://docs.microsoft.com/azure/active-directory/active-directory-enterprise-app-role-management).

7. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione **Descargar** para descargar el **Certificado (base 64)**. Guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

8. En la sección **Configurar Zscaler Beta**, copie las direcciones URL adecuadas según sus necesidades:

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    - URL de inicio de sesión
    - Identificador de Azure AD
    - URL de cierre de sesión

### <a name="configure-zscaler-beta-single-sign-on"></a>Configuración del inicio de sesión único de Zscaler Beta

1. Para automatizar la configuración de Zscaler Beta, instale la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, seleccione **Instale la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, al seleccionar **Configurar Zscaler Beta** se abre la aplicación Zscaler Beta. Desde allí, proporcione las credenciales del administrador para iniciar sesión en Zscaler Beta. La extensión del explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 6.

    ![Configuración](common/setup-sso.png)

3. Para configurar Zscaler Beta manualmente, abra una nueva ventana del explorador web. Inicie sesión como administrador en el sitio de la empresa de Zscaler Beta y siga estos pasos.

4. Vaya a **Administración** > **Autenticación** > **Configuración de autenticación** y siga estos pasos.
   
    ![Administración](./media/zscaler-beta-tutorial/ic800206.png "Administración")

     a. En **Tipo de autenticación**, seleccione **SAML**.

    b. Seleccione **Configurar SAML**.

5. En la ventana **Editar SAML**, siga estos pasos: 
            
    ![Administración de usuarios y autenticación](./media/zscaler-beta-tutorial/ic800208.png "Administración de usuarios y autenticación")
    
     a. En el cuadro **Dirección URL del portal de SAML**, pegue la **dirección URL de inicio de sesión** que copió de Azure Portal.

    b. En el cuadro de texto **Atributo de nombre de inicio de sesión**, escriba **NameID**.

    c. En el cuadro de texto **Certificado SSL público**, seleccione **Cargar** para cargar el certificado de firma de SAML que descargó de Azure Portal.

    d. Seleccione **Habilitar aprovisionamiento automático de SAML**.

    e. En el cuadro de texto **Atributo de nombre para mostrar del usuario**, escriba **displayName** si desea habilitar el aprovisionamiento automático de SAML para los atributos displayName.

    f. En el cuadro de texto **Atributo de nombre del grupo**, escriba **memberOf** si desea habilitar el aprovisionamiento automático de SAML para los atributos memberOf.

    g. En el cuadro de texto **Atributo de nombre del departamento**, escriba **department** si desea habilitar el aprovisionamiento automático de SAML para los atributos department.

    h. Seleccione **Guardar**.

6. En la página del cuadro de diálogo **Configurar autenticación de usuario**, siga estos pasos:

    ![Menú de activación y botón Activar](./media/zscaler-beta-tutorial/ic800207.png)

     a. Mantenga el puntero sobre el menú **Activación** situado en la parte inferior izquierda.

    b. Seleccione **Activar**.

## <a name="configure-proxy-settings"></a>Configuración de los valores de proxy
Para definir la configuración de proxy en Internet Explorer, siga estos pasos.

1. Inicie **Internet Explorer**.

2. Seleccione **Opciones de Internet** en el menú **Herramientas** para abrir el cuadro de diálogo **Opciones de Internet**. 
    
     ![Cuadro de diálogo Opciones de Internet](./media/zscaler-beta-tutorial/ic769492.png "Opciones de Internet")

3. Seleccione la pestaña **Conexiones**. 
  
     ![Pestaña conexiones](./media/zscaler-beta-tutorial/ic769493.png "Conexiones")

4. Seleccione **Configuración de LAN** para abrir el cuadro de diálogo **Configuración de red de área local (LAN)**.

5. En la sección **Servidor proxy**, siga estos pasos: 
   
    ![Sección Servidor proxy](./media/zscaler-beta-tutorial/ic769494.png "Servidor proxy")

     a. Seleccione la casilla de verificación **Usar un servidor proxy para la LAN**.

    b. En el cuadro de texto **Dirección**, escriba **gateway.Zscaler Beta.net**.

    c. En **Puerto**, escriba **80**.

    d. Seleccione la casilla de verificación **No usar servidor proxy para direcciones locales**.

    e. Seleccione **Aceptar** para cerrar el cuadro de diálogo **Configuración de red de área local (LAN)**.

6. Seleccione **Aceptar** para cerrar el cuadro de diálogo **Opciones de Internet**.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

En Azure Portal, cree un usuario de prueba llamado Britta Simon.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory** > **Usuarios** > **Todos los usuarios**.

    ![Vínculos Usuarios y Todos los usuarios](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En el cuadro de diálogo **Usuario**, siga estos pasos:

    ![Cuadro de diálogo Usuario](common/user-properties.png)

     a. En el cuadro **Nombre**, escriba **BrittaSimon**.
  
    b. En el cuadro **Nombre de usuario**, escriba `brittasimon@yourcompanydomain.extension`. Un ejemplo es BrittaSimon@contoso.com.

    c. Active la casilla de verificación **Mostrar contraseña**. Anote el valor que se muestra en el cuadro de texto **Contraseña**.

    d. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

Habilite a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Zscaler Beta.

1. En Azure Portal, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones** > **Zscaler Beta**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, escriba y seleccione **Zscaler Beta**.

    ![Vínculo de Zscaler Beta en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo Usuarios y grupos](common/users-groups-blade.png)

4. Seleccione **Agregar usuario**. En el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Botón Agregar usuario](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista de usuarios. A continuación, elija **Seleccionar** en la parte inferior de la pantalla.

    ![Cuadro de diálogo Usuarios y grupos](./media/zscaler-beta-tutorial/tutorial_zscalerbeta_users.png)

6. En el cuadro de diálogo **Seleccionar rol**, seleccione el rol de usuario adecuado en la lista. A continuación, elija **Seleccionar** en la parte inferior de la pantalla.

    ![Cuadro de diálogo Seleccionar rol](./media/zscaler-beta-tutorial/tutorial_zscalerbeta_roles.png)

7. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

    ![Cuadro de diálogo Agregar asignación](./media/zscaler-beta-tutorial/tutorial_zscalerbeta_assign.png)

### <a name="create-a-zscaler-beta-test-user"></a>Creación de un usuario de prueba de Zscaler Beta

En esta sección, se crea el usuario Britta Simon en Zscaler Beta. Zscaler Beta admite el **aprovisionamiento de usuarios Just-In-Time**, que está habilitado de forma predeterminada. No hay nada para hacer en esta sección. Si un usuario deja de existir en Zscaler Beta, se crea uno después de la autenticación.

>[!Note]
>Para crear un usuario manualmente, póngase en contacto con el [equipo de soporte técnico de Zscaler Beta](https://www.zscaler.com/company/contact).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

Pruebe la configuración de inicio de sesión único de Azure AD mediante el panel de acceso.

Al seleccionar el icono de Zscaler Beta en el Panel de acceso, debería iniciar sesión automáticamente en la versión de Zscaler Beta para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)
- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)
- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

