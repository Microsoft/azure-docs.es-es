---
title: 'Tutorial: Integración de Azure Active Directory con Pega Systems | Microsoft Docs'
description: En este tutorial, aprenderá a configurar el inicio de sesión único entre Azure Active Directory y Pega Systems.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/26/2019
ms.author: jeedes
ms.openlocfilehash: 3dd09770ef5f747f008090368c35b1f274374c5f
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88549082"
---
# <a name="tutorial-azure-active-directory-integration-with-pega-systems"></a>Tutorial: integración de Azure Active Directory con Pega Systems

En este tutorial, aprenderá a integrar Pega Systems con Azure Active Directory (Azure AD).

Esta integración ofrece las siguientes ventajas:

* Puede usar Azure AD para controlar quién tiene acceso a Pega Systems.
* Puede permitir que los usuarios inicien sesión automáticamente en Pega Systems (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Para más información acerca de la integración de aplicaciones SaaS con Azure AD, consulte [Inicio de sesión único en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Pega Systems, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una evaluación gratuita de un mes](https://azure.microsoft.com/pricing/free-trial/).
* Una suscripción a Pega Systems que tenga habilitado el inicio de sesión único.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial configurará y probará el inicio de sesión único de Azure AD en un entorno de prueba.

* Pega Systems admite el inicio de sesión único iniciado por SP e IDP.

## <a name="add-pega-systems-from-the-gallery"></a>Adición de Pega Systems desde la galería

Para configurar la integración de Pega Systems en Azure AD, será preciso que agregue Pega Systems desde la galería a la lista de aplicaciones SaaS administradas.

1. En [Azure Portal](https://portal.azure.com), en el panel izquierdo, seleccione **Azure Active Directory**:

    ![Seleccione Azure Active Directory.](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** > **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una aplicación, seleccione **Nueva aplicación** en la parte superior de la ventana:

    ![Seleccionar Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Pega Systems**. Seleccione **Pega Systems** en los resultados de búsqueda y, luego, seleccione **Agregar**.

     ![Search Results](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Pega Systems con un usuario de prueba llamado Britta Simon.
Para habilitar el inicio de sesión único, deberá establecer un vínculo entre un usuario de Azure AD y el usuario correspondiente de Pega Systems.

Para configurar y probar el inicio de sesión único de Azure AD con Pega Systems, debe hacer lo siguientes:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** para permitir que los usuarios utilicen esta característica.
2. **[Configure el inicio de sesión único en Pega Systems](#configure-pega-systems-single-sign-on)** en el lado de la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** para probar el inicio de sesión único de Azure AD.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** para permitir el inicio de sesión único de Azure AD para el usuario.
5. **[Cree un usuario de prueba de Pega Systems](#create-a-pega-systems-test-user)** que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** para comprobar que la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Pega Systems, haga lo siguiente:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Pega Systems**, seleccione **Inicio de sesión único**:

    ![Seleccionar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccione un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Selección de un método de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, seleccione el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**:

    ![Icono Editar](common/edit-urls.png)

4. En el cuadro de diálogo **Configuración básica de SAML**, si quiere configurar la aplicación en modo iniciado por IdP, haga lo siguiente:

    ![Cuadro de diálogo Configuración básica de SAML](common/idp-intiated.png)

    1. En el cuadro **Identificador**, escriba una dirección URL con este patrón:

       `https://<customername>.pegacloud.io:443/prweb/sp/<instanceID>`

    1. En el cuadro **Dirección URL de respuesta**, escriba una dirección URL con este patrón:

       `https://<customername>.pegacloud.io:443/prweb/PRRestService/WebSSO/SAML/AssertionConsumerService`

5. Si quiere configurar la aplicación en modo iniciado por SP, seleccione **Establecer direcciones URL adicionales** y haga lo siguiente.

    ![Información de dominio y direcciones URL de inicio de sesión único de Pega Systems](common/both-advanced-urls.png)

    1. En **Dirección URL de inicio de sesión**, escriba el valor de URL de inicio de sesión.

    1. En el cuadro **Estado de la retransmisión**, escriba una dirección URL con este patrón: `https://<customername>.pegacloud.io/prweb/sso`

    > [!NOTE]
    > Los valores proporcionados aquí son marcadores de posición. Deberá actualizarlos con los valores reales. Puede obtener los valores de identificador y dirección URL de respuesta de una aplicación de Pega, tal como se explica más adelante en este tutorial. Para obtener el valor de estado de la retransmisión, póngase en contacto con el [equipo de soporte técnico de Pega Systems](https://www.pega.com/contact-us). También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

6. La aplicación de Pega Systems necesita las aserciones de SAML en un formato específico. Para obtenerlas en el formato correcto, deberá agregar asignaciones de atributo personalizadas a la configuración de atributos del token SAML. En la siguiente captura se muestra la lista de atributos predeterminados. Seleccione el icono **Editar** para abrir el cuadro de diálogo **Atributos de usuario**.

    ![Atributos de usuario](common/edit-attribute.png)

7. Además de los atributos que se muestran en la captura de pantalla anterior, la aplicación Pega Systems requiere que se pasen algunos atributos más en la respuesta de SAML. En la sección **Notificaciones del usuario** del cuadro de diálogo **Atributos de usuario**, haga lo siguiente para agregar estos atributos del token SAML:

    
   - `uid`
   - `cn`
   - `mail`
   - `accessgroup`  
   - `organization`  
   - `orgdivision`
   - `orgunit`
   - `workgroup`  
   - `Phone`

    > [!NOTE]
    > Estos valores son específicos de su organización. Proporcione los valores adecuados.

    1. Seleccione **Agregar nueva notificación** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**:

    ![Selección de Agregar nueva notificación](common/new-save-attribute.png)

    ![Cuadro de diálogo Administrar las notificaciones del usuario](common/new-attribute-details.png)

    1. En el cuadro **Nombre**, escriba el nombre de atributo que se muestra para la fila.

    1. Deje **Espacio de nombres** vacío.

    1. En **Origen**, seleccione **Atributo**.

    1. En la lista **Atributo de origen**, escriba el valor de atributo que se muestra para esa fila.

    1. Seleccione **Aceptar**.

    1. Seleccione **Guardar**.

8. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione el vínculo **Descargar** situado junto a **XML de metadatos de federación**, según sus requisitos, y guarde el certificado en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

9. En la sección **Configurar Pega Systems**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copia de las direcciones URL de configuración](common/copy-configuration-urls.png)

    1. **Dirección URL de inicio de sesión**

    1. **Identificador de Azure AD**.

    1. **Dirección URL de cierre de sesión**

### <a name="configure-pega-systems-single-sign-on"></a>Configuración del inicio de sesión único en Pega Systems

1. Para configurar el inicio de sesión único en **Pega Systems**, inicie sesión en el portal de Pega con cuenta de administrador en otra ventana del explorador.

2. Seleccione **Create** > **SysAdmin** > **Authentication Service** (Crear > SysAdmin > Servicio de autenticación):

    ![Selección del servicio de autenticación](./media/pegasystems-tutorial/tutorial_pegasystems_admin.png)
    
3. Realice lo siguiente en la pantalla **Create Authentication Service** (Crear servicio de autenticación).

    ![Pantalla de creación del servicio de autenticación](./media/pegasystems-tutorial/tutorial_pegasystems_admin1.png)

    1. En la lista **Type** (Tipo), seleccione **SAML 2.0**.

    1. En el cuadro **Name** (Nombre), escriba cualquier nombre (por ejemplo, **Inicio de sesión único de Azure AD**).

    1. En el cuadro **Short description** (Descripción corta) escriba una descripción.  

    1. Seleccione **Create and open** (Crear y abrir).
    
4. En la sección **Identity Provider (IdP) information** (Información de proveedor de identidades [IdP]), haga clic en **Import IdP metadata** (Importar metadatos de IdP) y busque el archivo de metadatos que descargó de Azure Portal. Haga clic en **Submit** (Enviar) para cargar los metadatos:

    ![Sección de información del proveedor de identidades (IdP)](./media/pegasystems-tutorial/tutorial_pegasystems_admin2.png)
    
    La importación rellenará los datos del IdP, como se muestra a continuación:

    ![Datos del IdP importados](./media/pegasystems-tutorial/tutorial_pegasystems_admin3.png)
    
6. En la sección **Service Provider (SP) settings** (Configuración del proveedor de servicios [SP]), haga lo siguiente:

    ![Configuración del proveedor de servicio](./media/pegasystems-tutorial/tutorial_pegasystems_admin4.png)

    1. Copie el valor de **Entity Identification** (Identificación de la entidad) y péguelo en el cuadro **Identificador** de **Configuración básica de SAML** en Azure Portal.

    1. Copie el valor de **Assertion Consumer Service (ACS) location** (Ubicación del servicio de consumidor de aserciones [ACS]) y péguelo en el cuadro **Dirección URL de respuesta** de **Configuración básica de SAML** en Azure Portal.

    1. Seleccione **Disable request signing** (Deshabilitar firma de solicitudes).

7. Seleccione **Guardar**.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección creará un usuario de prueba llamado Britta Simon en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**:

    ![Selección de Todos los usuarios](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Selección de Nuevo usuario](common/new-user.png)

3. En el cuadro de diálogo **Usuario**, haga lo siguiente:

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el cuadro **Nombre**, escriba **BrittaSimon**.
  
    b. En el cuadro **Nombre de usuario**, escriba **brittasimon@\<yourcompanydomain>.\<extension>** (Por ejemplo, BrittaSimon@contoso.com).

    c. Seleccione **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro **Contraseña**.

    d. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure, para lo cual le concederá acceso a Pega Systems.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y, luego, **Pega Systems**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Pega Systems**.

    ![Lista de aplicaciones](common/all-applications.png)

3. En el panel izquierdo, seleccione **Usuarios y grupos**:

    ![Seleccionar Usuarios y grupos](common/users-groups-blade.png)

4. Seleccione **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Selección de Agregar usuario](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. Haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

### <a name="create-a-pega-systems-test-user"></a>Creación de un usuario de prueba de Pega Systems

A continuación, deberá crear un usuario llamado a Britta Simon en Pega Systems. Trabaje con el [equipo de soporte técnico de Pega Systems](https://www.pega.com/contact-us) para crear usuarios.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

Ahora, debe probar la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al seleccionar el icono de Pega Systems en el Panel de acceso, debería iniciar sesión automáticamente en la versión de Pega Systems para la que configuró el inicio de sesión único. Para más información, consulte [Acceso y uso del aplicaciones en el portal Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)