---
title: 'Tutorial: Integración de Azure Active Directory con Meta Networks Connector | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Meta Networks Connector.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/21/2019
ms.author: jeedes
ms.openlocfilehash: 71aaaeec174611a86b037b693187cc4f49fe1a0c
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88528934"
---
# <a name="tutorial-azure-active-directory-integration-with-meta-networks-connector"></a>Tutorial: Integración de Azure Active Directory con Meta Networks Connector

En este tutorial, aprenderá a integrar Meta Networks Connector con Azure Active Directory (Azure AD).
La integración de Meta Networks Connector con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Meta Networks Connector.
* Puede habilitar que los usuarios inicien sesión automáticamente en Meta Networks Connector (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Meta Networks Connector se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en Meta Networks Connector

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Meta Networks Connector admite SSO iniciado por **SP** e **IDP**
 
* Meta Networks Connector admite el aprovisionamiento de usuarios **Just-In-Time**

## <a name="adding-meta-networks-connector-from-the-gallery"></a>Incorporación de Meta Networks Connector desde la galería

Para configurar la integración de Meta Networks Connector en Azure AD, deberá agregar Meta Networks Connector desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Meta Networks Connector desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Meta Networks Connector**, seleccione **Meta Networks Connector** en el panel de resultados y haga clic en el botón **Agregar** para agregar la aplicación.

     ![Meta Networks Connector en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Meta Networks Connector utilizando un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Meta Networks Connector.

Para configurar y probar el inicio de sesión único de Azure AD con Meta Networks Connector, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único en Meta Networks Connector](#configure-meta-networks-connector-single-sign-on)** : para configurar las opciones de inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba en Meta Networks Connector](#create-meta-networks-connector-test-user)** : para tener un homólogo de Britta Simon en Meta Networks Connector que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Meta Networks Connector, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Meta Networks Connector**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en el modo iniciado por **IDP** siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Meta Networks Connector](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://login.nsof.io/v1/<ORGANIZATION-SHORT-NAME>/saml/metadata`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://login.nsof.io/v1/<ORGANIZATION-SHORT-NAME>/sso/saml`

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    ![Información de dominio y direcciones URL de inicio de sesión único de Meta Networks Connector](common/both-advanced-urls.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<ORGANIZATION-SHORT-NAME>.metanetworks.com/login`

    b. En el cuadro de texto **Estado de la retransmisión**, escriba una dirección URL que siga este patrón: `https://<ORGANIZATION-SHORT-NAME>.metanetworks.com/#/`

    > [!NOTE]
    > Estos valores no son reales. La actualización de estos valores por el identificador, dirección URL de respuesta y dirección URL de inicio de sesión reales se explica más adelante en el tutorial.

6. La aplicación Meta Networks Connector espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados. Haga clic en el icono **Editar** para abrir el cuadro de diálogo **Atributos de usuario**.

    ![imagen](common/edit-attribute.png)
    
7. Además de lo anterior, la aplicación Meta Networks Connector espera que se usen algunos atributos más en la respuesta de SAML. En la sección **Notificaciones del usuario** del cuadro de diálogo **Atributos de usuario**, realice los siguientes pasos para agregar el atributo Token SAML como se muestra en la tabla siguientes:
    
    | Nombre | Atributo de origen | Espacio de nombres|
    | ---------------| --------------- | -------- |
    | firstname | user.givenname | |
    | lastname | user.surname | |
    | emailaddress| user.mail| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims` |
    | name | user.userprincipalname| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims` |
    | phone | user.telephonenumber | |

    a. Haga clic en **Agregar nueva notificación** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

    ![imagen](common/new-save-attribute.png)

    ![imagen](common/new-attribute-details.png)

    b. En el cuadro de texto **Nombre**, escriba el nombre que se muestra para la fila.

    c. Deje **Espacio de nombres** en blanco.

    d. Seleccione **Atributo** como origen.

    e. En la lista **Atributo de origen**, escriba el valor de atributo que se muestra para esa fila.

    f. Haga clic en **Aceptar**.

    g. Haga clic en **Save**(Guardar).

8. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

9. En la sección **Set up Meta Networks Connector** (Configurar Meta Networks Connector), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-meta-networks-connector-single-sign-on"></a>Configuración del inicio de sesión único en Meta Networks Connector

1. Abra una nueva pestaña en el explorador e inicie sesión en su cuenta de administrador de Meta Networks Connector.
    
    > [!NOTE]
    > Meta Networks Connector es un sistema seguro. Por lo que antes de acceder a su portal deberá hacer que su dirección IP pública se incluya en una lista de permitidos en su lado. Para obtener la dirección IP pública, siga el vínculo especificado [aquí](https://whatismyipaddress.com/). Envíe su dirección IP al [equipo de soporte técnico de Meta Networks Connector](mailto:support@metanetworks.com) para que su dirección IP se incluya en una lista de permitidos.
    
2. Vaya a **Administrador** y seleccione **Configuración**.
    
    ![Configurar inicio de sesión único](./media/metanetworksconnector-tutorial/configure3.png)
    
3. Asegúrese de que **Registro del tráfico de Internet** y **Forzar MFA de VPN** se establecen en Desactivado.
    
    ![Configurar inicio de sesión único](./media/metanetworksconnector-tutorial/configure1.png)
    
4. Vaya a **Administrador** y seleccione **SAML**.
    
    ![Configurar inicio de sesión único](./media/metanetworksconnector-tutorial/configure4.png)
    
5. En la página **Detalles**, lleve a cabo los pasos siguientes:
    
    ![Configurar inicio de sesión único](./media/metanetworksconnector-tutorial/configure2.png)
    
    a. Copie el valor de **Dirección URL de inicio de sesión único** y péguelo en el cuadro de texto **Dirección URL de inicio de sesión** en la sección **Dominio y direcciones URL de Meta Networks Connector**.
    
    b. Copie el valor de **Dirección URL del destinatario** y péguelo en el cuadro de texto **Dirección URL de respuesta** en la sección **Dominio y direcciones URL de Meta Networks Connector**.
    
    c. Copie el valor de **Identificador URI de audiencia** y péguelo en el cuadro de texto **Identificador (Identificador de entidad)** en la sección **Dominio y direcciones URL de Meta Networks Connector**.
    
    d. Habilitación de SAML
    
6. En la pestaña **General**, realice los pasos siguientes:

    ![Configurar inicio de sesión único](./media/metanetworksconnector-tutorial/configure5.png)

    a. En **Dirección URL del inicio de sesión único del proveedor de identidades**, pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    b. En **Emisor de proveedor de identidades**, pegue el valor de **Identificador de Azure AD** que ha copiado de Azure Portal.

    c. Abra el certificado descargado desde Azure Portal en el Bloc de notas y péguelo en el cuadro de texto **Certificado X.509**.

    d. Habilite el **Aprovisionamiento Just-in-Time**.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba **brittasimon\@yourcompanydomain.extension**  
    Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para usar el inicio de sesión único de Azure al concederle acceso a Meta Networks Connector.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Meta Networks Connector**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Meta Networks Connector**.

    ![Vínculo de Meta Networks Connector en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-meta-networks-connector-test-user"></a>Creación de un usuario de prueba en Meta Networks Connector

En esta sección, se crea un usuario llamado Britta Simon en Meta Networks Connector. Meta Networks Connector admite el aprovisionamiento Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si el usuario no existe en Meta Networks Connector, se crea uno cuando se intenta acceder a Meta Networks Connector.

>[!Note]
>Si necesita crear manualmente un usuario, es preciso que se ponga contacto con el [equipo de soporte técnico de Meta Networks Connector](mailto:support@metanetworks.com).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Meta Networks Connector en el Panel de acceso, debería iniciar sesión automáticamente en la aplicación Meta Networks Connector para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

