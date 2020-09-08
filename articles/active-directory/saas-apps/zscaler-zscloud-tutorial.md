---
title: 'Tutorial: integración de Azure Active Directory con ZScaler ZSCloud | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y ZScaler ZSCloud.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/24/2019
ms.author: jeedes
ms.openlocfilehash: 21a6d6f728b11c3dee166921538bb844d1e72f70
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88545829"
---
# <a name="tutorial-azure-active-directory-integration-with-zscaler-zscloud"></a>Tutorial: integración de Azure Active Directory con ZScaler ZSCloud

En este tutorial, aprenderá a integrar ZScaler ZSCloud con Azure Active Directory (Azure AD).
La integración de ZScaler ZSCloud con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Zscaler ZSCloud.
* Puede permitir que los usuarios inicien sesión automáticamente en Zscaler ZSCloud (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con ZScaler ZSCloud, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción habilitada para el inicio de sesión único en Zscaler ZSCloud

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Zscaler ZSCloud admite el inicio de sesión único iniciado por **SP**.

* Zscaler ZSCloud admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="adding-zscaler-zscloud-from-the-gallery"></a>Incorporación de ZScaler ZSCloud desde la galería

Para configurar la integración de ZScaler ZSCloud en Azure AD, será preciso agregar ZScaler ZSCloud desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar ZScaler ZSCloud desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Zscaler ZSCloud**, seleccione **Zscaler ZSCloud** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![Zscaler ZSCloud en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Zscaler ZSCloud con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Zscaler ZSCloud.

Para configurar y probar el inicio de sesión único de Azure AD con ZScaler ZSCloud, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único en Zscaler ZSCloud](#configure-zscaler-zscloud-single-sign-on)** : para configurar el inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de ZScaler ZSCloud](#create-zscaler-zscloud-test-user)** : para tener un homólogo de Britta Simon en ZScaler ZSCloud vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con ZScaler ZSCloud, haga lo siguiente:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **ZScaler ZSCloud**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Zscaler ZSCloud](common/sp-signonurl.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL con la que los usuarios inician sesión en la aplicación ZScaler ZSCloud.

    > [!NOTE]
    > Tiene que actualizar el valor con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico de ZScaler ZSCloud](https://help.zscaler.com/) para obtenerlo. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. La aplicación Zscaler ZSCloud espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token SAML. La siguiente captura de muestra la lista de atributos predeterminados. Haga clic en el icono **Editar** para abrir el cuadro de diálogo **Atributos de usuario**.

    ![imagen](common/edit-attribute.png)

6. Además de lo anterior, la aplicación Zscaler ZSCloud espera que se usen algunos atributos más en la respuesta de SAML. En la sección **Notificaciones del usuario** del cuadro de diálogo **Atributos de usuario**, realice los siguientes pasos para agregar el atributo Token SAML como se muestra en la tabla siguientes:
    
    | Nombre | Atributo de origen |
    | ---------| ------------ |
    | memberOf     | user.assignedroles |

    a. Haga clic en **Agregar nueva notificación** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

    ![imagen](common/new-save-attribute.png)

    ![imagen](common/new-attribute-details.png)

    b. En el cuadro de texto **Nombre**, escriba el nombre que se muestra para la fila.

    c. Deje **Espacio de nombres** en blanco.

    d. Seleccione **Atributo** como origen.

    e. En la lista **Atributo de origen**, escriba el valor de atributo que se muestra para esa fila.
    
    f. Haga clic en **Save**(Guardar).

    > [!NOTE]
    > Haga clic [aquí](https://docs.microsoft.com/azure/active-directory/active-directory-enterprise-app-role-management) para saber cómo configurar el valor Role en Azure AD.

7. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

8. En la sección **Configurar Zscaler ZSCloud**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-zscaler-zscloud-single-sign-on"></a>Configuración del inicio de sesión único en Zscaler ZSCloud

1. Para automatizar la configuración en Zscaler ZSCloud, debe instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Configurar Zscaler ZSCloud** para ir a la aplicación Zscaler ZSCloud. Desde allí, proporcione las credenciales del administrador para iniciar sesión en Zscaler ZSCloud. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 6.

    ![Configuración del inicio de sesión único](common/setup-sso.png)

3. Si quiere configurar Zscaler ZSCloud manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Zscaler ZSCloud como administrador y haga lo siguiente:

4. Vaya a **Administración > Autenticación > Configuración de autenticación** y realice los siguientes pasos:
   
    ![Administración](./media/zscaler-zscloud-tutorial/ic800206.png "Administración")

    a. En Tipo de autenticación, elija **SAML**.

    b. Haga clic en **Configurar SAML**.

5. En la ventana **Editar SAML**, realice los pasos siguientes y haga clic en Guardar.  
            
    ![Administración de usuarios y autenticación](./media/zscaler-zscloud-tutorial/ic800208.png "Manage Users & Authentication")
    
    a. En el cuadro de texto **Dirección URL del portal de SAML**, pegue la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    b. En el cuadro de texto **Atributo de nombre de inicio de sesión**, escriba **NameID**.

    c. Haga clic en **Cargar** para cargar el certificado de firma de SAML de Azure que ha descargado desde Azure Portal en el **Certificado SSL público**.

    d. Alterne **Habilitar aprovisionamiento automático de SAML**.

    e. En el cuadro de texto **Atributo de nombre para mostrar del usuario**, escriba **displayName** si desea habilitar el aprovisionamiento automático de SAML para atributos displayName.

    f. En el cuadro de texto **Atributo de nombre del grupo**, escriba **memberOf** si desea habilitar el aprovisionamiento automático de SAML para atributos memberOf.

    g. En el cuadro de texto **Atributo de nombre del departamento**, escriba **department** si desea habilitar el aprovisionamiento automático de SAML para atributos department.

    h. Haga clic en **Save**(Guardar).

6. En la página del cuadro de diálogo **Configurar autenticación de usuario** , realice los pasos siguientes:

    ![Administración](./media/zscaler-zscloud-tutorial/ic800207.png)

    a. Mantenga el puntero sobre el menú **Activación** situado cerca de la parte inferior izquierda.

    b. Haga clic en **Activar**.

## <a name="configuring-proxy-settings"></a>Configuración de los valores de proxy
### <a name="to-configure-the-proxy-settings-in-internet-explorer"></a>Para definir la configuración de proxy en Internet Explorer

1. Inicie **Internet Explorer**.

2. Seleccione **Opciones de Internet** en el menú **Herramientas** para abrir el diálogo **Opciones de Internet**.   
    
     ![Opciones de Internet](./media/zscaler-zscloud-tutorial/ic769492.png "Opciones de Internet")

3. Haga clic en la pestaña **Conexiones** .   
  
     ![Conexiones](./media/zscaler-zscloud-tutorial/ic769493.png "Conexiones")

4. Haga clic en **Configuración de LAN** para abrir el diálogo **Configuración de LAN**.

5. En la sección del servidor proxy, lleve a cabo estos pasos:   
   
    ![Servidor proxy](./media/zscaler-zscloud-tutorial/ic769494.png "Servidor proxy")

    a. Seleccione **Usar un servidor proxy para la LAN**.

    b. En el cuadro de texto Dirección, escriba **gateway.Zscaler ZSCloud.net**.

    c. En el cuadro de texto Puerto, escriba **80**.

    d. Seleccione **No usar servidor proxy para direcciones locales**.

    e. Haga clic en **Aceptar** para cerrar el diálogo **Configuración de red de área local (LAN)** .

6. Haga clic en **Aceptar** para cerrar el diálogo **Opciones de Internet**.

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a ZScaler ZSCloud.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y, luego, **Zscaler ZSCloud**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **ZScaler ZSCloud**.

    ![Vínculo de Zscaler ZSCloud en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione el usuario, como **Britta Simon**, en la lista y luego haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

    ![imagen](./media/zscaler-zscloud-tutorial/tutorial_zscalerzscloud_users.png)

6. En el cuadro de diálogo **Seleccionar rol**, elija el rol de usuario adecuado de la lista y, luego, haga clic en el botón **Seleccionar** situado en la parte inferior de la pantalla.

    ![imagen](./media/zscaler-zscloud-tutorial/tutorial_zscalerzscloud_roles.png)

7. En el cuadro de diálogo **Agregar asignación**, seleccione el botón **Asignar**.

    ![imagen](./media/zscaler-zscloud-tutorial/tutorial_zscalerzscloud_assign.png)

    >[!NOTE]
    >El rol de acceso predeterminado no se admite porque interrumpirá el aprovisionamiento, por lo que no se puede seleccionar al asignar el usuario.

### <a name="create-zscaler-zscloud-test-user"></a>Creación de un usuario de prueba de Zscaler ZSCloud

En esta sección, se crea un usuario llamado Britta Simon en Zscaler ZSCloud. Zscaler ZSCloud admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en Zscaler ZSCloud, se crea uno después de la autenticación.

>[!Note]
>Si tiene que crear un usuario manualmente, póngase en contacto con el [equipo de soporte técnico de Zscaler ZSCloud](https://help.zscaler.com/).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Zscaler ZSCloud en el Panel de acceso, debería iniciar sesión automáticamente en la versión de Zscaler ZSCloud para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

