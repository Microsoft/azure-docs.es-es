---
title: 'Tutorial: Integración de Azure Active Directory con SAML SSO for Jira by resolution GmbH | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SAML SSO for Jira by resolution GmbH.
services: active-directory
documentationCenter: na
author: jeevansd
manager: daveba
ms.reviewer: barbkess
ms.assetid: 20e18819-e330-4e40-bd8d-2ff3b98e035f
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/03/2018
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 943131bc746b5d2a1fd95a26a6a6c9f3bb6b9e57
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2019
ms.locfileid: "65509953"
---
# <a name="tutorial-azure-active-directory-integration-with-saml-sso-for-jira-by-resolution-gmbh"></a>Tutorial: Integración de Azure Active Directory con SAML SSO for Jira by resolution GmbH

En este tutorial, aprenderá a configurar SAML SSO for Jira by resolution GmbH con Azure Active Directory (Azure AD).
La integración de SAML SSO for Jira by resolution GmbH con Azure AD proporciona las siguientes ventajas:

* Puedes controlar en Azure AD quién puede iniciar sesión en Jira con el complemento SAML SSO de resolution GmbH.
* Puede permitir que los usuarios inicien sesión automáticamente en Jira mediante SAML SSO for Jira by resolution GmbH (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD y SAML SSO for Jira by resolution GmbH, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en SAML SSO for Jira by resolution GmbH

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SAML SSO for Jira by resolution GmbH admite el inicio de sesión único iniciado por **SP** e **IDP**.

## <a name="adding-an-enterprise-application-for-single-sign-on"></a>Adición de una aplicación empresarial para inicio de sesión único

Para configurar un inicio de sesión único en Azure AD, deberá agregar una nueva aplicación empresarial. En la galería, hay una aplicación configurada previamente preestablecida para esto, **SAML SSO for Jira by resolution GmbH**.

**Para agregar SAML SSO for Jira by resolution GmbH desde la galería, siga estos pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)**, haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y, a continuación, haga clic en **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** en la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **SAML SSO for Jira by resolution GmbH**, seleccione **SAML SSO for Jira by resolution GmbH** en el panel de resultados y luego haga clic en el botón **Agregar** para agregar la aplicación. También puede cambiar el nombre de la aplicación empresarial.

     ![SAML SSO for Jira by resolution GmbH en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-single-sign-on-with-the-saml-sso-plugin-and-azure-ad"></a>Configuración y prueba del inicio de sesión único con el complemento SAML SSO y Azure AD

En esta sección, probará y configurará el inicio de sesión único en Jira para un usuario de Azure AD. Esto se realizará para un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de SAML SSO for Jira by resolution GmbH.

Para configurar y probar el inicio de sesión único, es preciso completar los pasos siguientes:

1. **[Configurar la aplicación empresarial de Azure AD para el inicio de sesión único](#configure-the-azure-ad-enterprise-application-for-single-sign-on)**: para configurar la aplicación empresarial de Azure AD para el inicio de sesión único.
2. **[Configurar el complemento SAML SSO de la instancia de Jira](#configure-the-saml-sso-plugin-of-your-jira-instance)**: para configurar las opciones del inicio de sesión único en la aplicación.
3. **[Crear un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)**: para crear un usuario de prueba en Azure AD.
1. **[Asignar el usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)**: para permitir que el usuario de prueba utilice el único inicio de sesión en Azure.
1. **[Crear el usuario de prueba en Jira](#create-the-test-user-also-in-jira)**: para crear un usuario de prueba homólogo en Jira para el usuario de prueba de Azure AD.
1. **[Prueba del inicio de sesión único](#test-single-sign-on)**: compruebe que la configuración funciona.

### <a name="configure-the-azure-ad-enterprise-application-for-single-sign-on"></a>Configuración de la aplicación empresarial de Azure AD para el inicio de sesión único

En esta sección, va a configurar el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único con SAML SSO for Jira by resolution GmbH, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la aplicación empresarial **SAML SSO for Jira by resolution GmbH** que acaba de crear, seleccione **Inicio de sesión único** en el panel izquierdo.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. Para **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. Después, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice los siguientes pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de SAML SSO for Jira by resolution GmbH](common/idp-intiated.png)

     a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<server-base-url>/plugins/servlet/samlsso`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<server-base-url>/plugins/servlet/samlsso`

    c. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    ![Información de dominio y direcciones URL de inicio de sesión único de SAML SSO for Jira by resolution GmbH](common/metadata-upload-additional-signon.png)

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<server-base-url>/plugins/servlet/samlsso`

    > [!NOTE]
    > Para los valores de identificador, dirección URL de respuesta y dirección URL de inicio de sesión, sustituya **\<server-base-url>** por la dirección URL base de la instancia de Jira. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal. Si tiene algún problema, póngase en contacto con el [equipo de soporte técnico de clientes de SAML SSO for Jira by resolution GmbH](https://www.resolution.de/go/support).

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, descargue el archivo **XML de metadatos de federación**  y guárdelo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

### <a name="configure-the-saml-sso-plugin-of-your-jira-instance"></a>Configuración del complemento SAML SSO de la instancia de Jira 

1. En otra ventana del explorador web, inicie sesión en la instancia de Jira como administrador.

2. Mantenga el puntero sobre el icono de engranaje en el lado derecho y haga clic en **Manage apps** (Administrar aplicaciones).
    
    ![Configurar inicio de sesión único](./media/samlssojira-tutorial/addon1.png)

3. Se le redirigirá a la página de acceso de administrador para especificar la **contraseña** y haga clic en el botón **Confirm** (Confirmar).

    ![Configurar inicio de sesión único](./media/samlssojira-tutorial/addon2.png)

4. Jira normalmente redirige a Atlassian Marketplace. En caso contrario, haga clic en **Find new apps** (Buscar nuevas aplicaciones) en el panel izquierdo. Busque **SAML Single Sign On (SSO) for JIRA** y haga clic en el botón **Install** (Instalar) para instalar el nuevo complemento SAML.

    ![Configurar inicio de sesión único](./media/samlssojira-tutorial/store.png)

5. Se iniciará la instalación del complemento. Cuando termine, haga clic en el botón **Close** (Cerrar).

    ![Configurar inicio de sesión único](./media/samlssojira-tutorial/store-2.png)

    ![Configurar inicio de sesión único](./media/samlssojira-tutorial/store-3.png)

6. A continuación, haga clic en **Manage** (Administrar).

    ![Configurar inicio de sesión único](./media/samlssojira-tutorial/store-4.png)
    
8. Después, haga clic en **Configure** (Configurar) para configurar el complemento que se acaba de instalar.

    ![Configurar inicio de sesión único](./media/samlssojira-tutorial/store-5.png)

9. En el asistente **SAML SingleSignOn Plugin Configuration** (Configuración del complemento SingleSignOn de SAML), haga clic en **Add new IdP** (Agregar IdP nuevo) para configurar a Azure AD como el nuevo proveedor de identidades.

    ![Configurar inicio de sesión único](./media/samlssojira-tutorial/addon4.png) 

10. En la página **Choose your SAML Identity Provider** (Elegir el proveedor de identidades SAML), siga estos pasos:

    ![Configurar inicio de sesión único](./media/samlssojira-tutorial/addon5a.png)
 
     a. Establezca **Azure AD** como el tipo de IdP.
    
    b. Agregue el **nombre** del proveedor de identidades (p. ej., Azure AD).
    
    c. Agregue la **descripción** (opcional) del proveedor de identidades (p. ej., Azure AD).
    
    d. Haga clic en **Next**.
    
11. En la página **Identity provider configuration** (Configuración del proveedor de identidades), haga clic en **Next** (Siguiente).
 
    ![Configurar inicio de sesión único](./media/samlssojira-tutorial/addon5b.png)

12. En la página **Import SAML IdP Metadata** (Importar los metadatos del IdP de SAML), siga los pasos siguientes:

    ![Configurar inicio de sesión único](./media/samlssojira-tutorial/addon5c.png)

     a. Haga clic en el botón **Select Metadata XML File** (Seleccionar archivo XML de metadatos) y elija el archivo **XML de metadatos de federación** que ha descargado antes.

    b. Haga clic en el botón **Import** (Importar).
     
    c. Espere un momento hasta que la importación se realice correctamente.  
     
    d. Haga clic en el botón **Next** (Siguiente).
    
13. En la página **User ID attribute and transformation** (Transformación y atributo del id. de usuario), haga clic en el botón **Next** (Siguiente).

    ![Configurar inicio de sesión único](./media/samlssojira-tutorial/addon5d.png)
    
14. En la página **User creation and update** (Creación y actualización de usuarios), haga clic en **Save & Next** (Guardar y siguiente) para guardar la configuración.
    
    ![Configurar inicio de sesión único](./media/samlssojira-tutorial/addon6a.png)
    
15. En la página **Test your settings** (Probar su configuración), haga clic en **Skip test & configure manually** (Omitir la prueba y configurar manualmente) para omitir la prueba de usuario por ahora. Esto se realizará en la sección siguiente y requiere la configuración de algunos valores en Azure Portal.
    
    ![Configurar inicio de sesión único](./media/samlssojira-tutorial/addon6b.png)
    
16. Haga clic en **OK** (Aceptar) para omitir la advertencia.
    
    ![Configurar inicio de sesión único](./media/samlssojira-tutorial/addon6c.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon". Con el usuario, probará el inicio de sesión único.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Elija **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las **propiedades de usuario**, siga estos pasos:

    ![Cuadro de diálogo Usuario](common/user-properties.png)

     a. En el campo **Nombre**, escriba **Britta Simon**.
  
    b. En el campo **Nombre de usuario**, escriba <b>BrittaSimon@contoso.com</b>.

    c. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro de contraseña.

    d. Haga clic en **Create**(Crear).

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, agregará a Britta Simon a la aplicación empresarial, que le permitirá utilizar inicio de sesión único.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**. 

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, busque la aplicación empresarial que ha creado al principio de este tutorial. Si está siguiendo los pasos del tutorial, se denomina **SAML SSO for Jira by resolution GmbH**. Si ha proporcionado otro nombre, busque por ese nombre.

    ![El vínculo de SAML SSO for Jira by resolution GmbH en la lista de aplicaciones](common/all-applications.png)

3. En el panel de la izquierda, haga clic en **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Seleccione **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario de la lista y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-the-test-user-also-in-jira"></a>Creación del usuario de prueba también en Jira

Para permitir que los usuarios de Azure AD inicien sesión en SAML SSO for Jira by resolution GmbH, deben estar aprovisionados en SAML SSO for Jira by resolution GmbH. En el caso de este tutorial, deberá realizar el aprovisionamiento de forma manual. Sin embargo, hay también otros modelos de aprovisionamiento disponibles para el complemento SAML SSO by resolution, por ejemplo **simplemente In Time** aprovisionamiento. Consulte la documentación en [SAML SSO by resolution GmbH](https://wiki.resolution.de/doc/saml-sso/latest/all). Si tiene alguna pregunta sobre ella, póngase en contacto con el soporte técnico para [su resolución](https://www.resolution.de/go/support).

**Para aprovisionar una cuenta de usuario manualmente, realice estos pasos:**

1. Inicie sesión como administrador en una instancia de Jira.

2. Mantenga el puntero sobre el icono de engranaje y seleccione **Administración de usuarios**.

   ![Agregar empleado](./media/samlssojira-tutorial/user1.png)

3. Se le redirigirá a la página de acceso de administrador para especificar la **contraseña** y haga clic en el botón **Confirm** (Confirmar).

    ![Agregar empleado](./media/samlssojira-tutorial/user2.png) 

4. En la sección de la pestaña **User management** (Administración de usuarios), haga clic en **Create user** (Crear usuario).

    ![Agregar empleado](./media/samlssojira-tutorial/user3-new.png) 

5. En la página del cuadro de diálogo **"Create New User"** (Crear nuevo usuario), realice los pasos siguientes. El usuario se crea exactamente igual que en Azure AD:

    ![Agregar empleado](./media/samlssojira-tutorial/user4-new.png) 

     a. En el cuadro de texto **Dirección de correo electrónico**, escriba la dirección de correo electrónico del usuario: <b>BrittaSimon@contoso.com</b>.

    b. En el cuadro de texto **Nombre completo**, escriba el nombre completo del usuario: **Britta Simon**.

    c. En el cuadro de texto **Username** (Nombre de usuario), escriba la dirección de correo electrónico del usuario: <b>BrittaSimon@contoso.com</b>. 

    d. En el cuadro de texto **Password** (Contraseña), escriba la contraseña del usuario.

    e. Haga clic en **Create user** (Crear usuario) para finalizar la creación del usuario.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de SAML SSO for Jira by resolution GmbH en el Panel de acceso, debería iniciar sesión automáticamente en la solución SAML SSO for Jira by resolution GmbH para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

También puede probar de sesión único, si navega a [https://\<base-server-url>/plugins/servlet/samlsso](https://\<server-base-url>/plugins/servlet/samlsso). Sustituya **\<server-base-url>** por la dirección URL base de la instancia de Jira.


## <a name="enable-single-sign-on-redirection-for-jira"></a>Habilitación de la redirección de inicio de sesión única para Jira

Como se ha indicado en la sección anterior, actualmente hay dos formas de desencadenar el inicio de sesión único. Ya sea mediante **Azure Portal** o mediante **un vínculo especial a la instancia de Jira**. El complemento SAML SSO by resolution GmbH también permite desencadenar un inicio de sesión único simplemente **mediante el acceso a cualquier dirección URL que apunte a la instancia de Jira**.

En esencia, todos los usuarios que tienen acceso a Jira se redirigirán al inicio de sesión único después de activar una opción en el complemento.

Para activar el redireccionamiento de inicio de sesión único, realice lo siguiente en **la instancia de Jira**:

1. Acceda a la página de configuración del complemento SAML SSO de Jira.
1. Haga clic en **Redirección** en el panel izquierdo.
![](./media/samlssojira-tutorial/ssore1.png)

1. Marque **Enable SSO Redirect** (Habilitar la redirección de inicio de sesión único).
![](./media/samlssojira-tutorial/ssore2.png) 

1. Presione el botón **Save Settings** (Guardar configuración) en la esquina superior derecha.

Después de activar la opción, todavía puede llegar a la solicitud de nombre de usuario y contraseña si la opción **Enable nosso** (Habilitar nosso) está activada al navegar a [https://\<server-base-url>/login.jsp?nosso](https://\<server-base-url>/login.jsp?nosso). Como siempre, sustituya **\<server-base-url>** por la dirección URL base.


## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

