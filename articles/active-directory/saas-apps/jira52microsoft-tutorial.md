---
title: 'Tutorial: Integración de Azure Active Directory con JIRA SAML SSO by Microsoft (versión 5.2) | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y JIRA SAML SSO by Microsoft (versión 5.2).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/22/2019
ms.author: jeedes
ms.openlocfilehash: e0198fdcfea1656e3aec5179358e69fb6fb55723
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88547581"
---
# <a name="tutorial-azure-active-directory-integration-with-jira-saml-sso-by-microsoft-v52"></a>Tutorial: Integración de Azure Active Directory con JIRA SAML SSO by Microsoft (versión 5.2)

En este tutorial, obtendrá información sobre cómo integrar JIRA SAML SSO by Microsoft (versión 5.2) con Azure Active Directory (Azure AD).
La integración de JIRA SAML SSO by Microsoft (versión 5.2) con Azure AD le proporciona las ventajas siguientes:

* Puede controlar en Azure AD quién tiene acceso a JIRA SAML SSO by Microsoft (versión 5.2).
* Puede permitir que los usuarios inicien sesión automáticamente en JIRA SAML SSO by Microsoft (versión 5.2) (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="description"></a>Descripción

Use la cuenta de Microsoft Azure Active Directory con el servidor JIRA de Atlassian para habilitar el inicio de sesión único. De esta forma, todos los usuarios de la organización pueden usar las credenciales de Azure AD para iniciar sesión en la aplicación JIRA. Este complemento usa SAML 2.0 para la federación.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con JIRA SAML SSO by Microsoft (versión 5.2), se necesitan los elementos siguientes:

- Una suscripción de Azure AD
- JIRA Core y Software 5.2 se deben instalar y configurar en una versión de 64 bits de Windows
- El servidor JIRA es compatible con HTTPS
- Tenga en cuenta que las versiones admitidas para el complemento JIRA se mencionan en la sección siguiente.
- El servidor JIRA es accesible en Internet, especialmente a la página de inicio de sesión de Azure AD para la autenticación y debe poder recibir el token de Azure AD
- Las credenciales de administrador se configuran en JIRA
- WebSudo está deshabilitado en JIRA
- Usuario de prueba creado en la aplicación de servidor JIRA

> [!NOTE]
> Para probar los pasos de este tutorial, no se recomienda el uso de un entorno de producción de JIRA. Pruebe primero la integración en el entorno de desarrollo o de ensayo de la aplicación y, después, use el entorno de producción.

Para probar los pasos de este tutorial, debe seguir estas recomendaciones:

- No use el entorno de producción, salvo que sea necesario.
- Si no dispone de un entorno de prueba de Azure AD, aquí puede obtener una versión de evaluación de un mes: [Oferta de prueba](https://azure.microsoft.com/pricing/free-trial/).

## <a name="supported-versions-of-jira"></a>Versiones compatibles de JIRA

* JIRA Core y Software: 5.2
* JIRA también admite desde 6.0 hasta 7.12. Para más información, haga clic en [JIRA SAML SSO by Microsoft](jiramicrosoft-tutorial.md)

> [!NOTE]
> Tenga en cuenta que nuestro complemento de JIRA también funciona con Ubuntu 16.04.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* JIRA SAML SSO by Microsoft (versión 5.2) admite el inicio de sesión único iniciado por **SP**

## <a name="adding-jira-saml-sso-by-microsoft-v52-from-the-gallery"></a>Agregar JIRA SAML SSO by Microsoft (versión 5.2) desde la galería

Para configurar la integración de JIRA SAML SSO by Microsoft (versión 5.2) en Azure AD, deberá agregar JIRA SAML SSO by Microsoft (versión 5.2) desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar JIRA SAML SSO by Microsoft (versión 5.2) desde la galería, siga estos pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **JIRA SAML SSO by Microsoft (versión 5.2)** , seleccione **JIRA SAML SSO by Microsoft (versión 5.2)** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

    ![JIRA SAML SSO by Microsoft (versión 5.2) se encuentra en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, va a configurar y probar el inicio de sesión único de Azure AD con JIRA SAML SSO by Microsoft (versión 5.2) con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de JIRA SAML SSO by Microsoft (versión 5.2).

Para configurar y probar el inicio de sesión único de Azure AD con JIRA SAML SSO by Microsoft (versión 5.2), es necesario completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de JIRA SAML SSO by Microsoft (versión 5.2)](#configure-jira-saml-sso-by-microsoft-v52-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de JIRA SAML SSO by Microsoft (versión 5.2)](#create-jira-saml-sso-by-microsoft-v52-test-user)** : para tener un homólogo de Britta Simon en JIRA SAML SSO by Microsoft (versión 5.2) que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con JIRA SAML SSO by Microsoft (versión 5.2), siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **JIRA SAML SSO by Microsoft (versión 5.2)** , haga clic en **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de JIRA SAML SSO by Microsoft (versión 5.2)](common/sp-identifier-reply.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<domain:port>/plugins/servlet/saml/auth`

    b. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón: `https://<domain:port>/`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<domain:port>/plugins/servlet/saml/auth`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. En caso de que sea una dirección URL con nombre, el puerto es opcional. Estos valores se reciben durante la configuración del complemento de Jira, que se explica más adelante en el tutorial.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

### <a name="configure-jira-saml-sso-by-microsoft-v52-single-sign-on"></a>Configuración del inicio de sesión único de JIRA SAML SSO by Microsoft (versión 5.2)

1. En otra ventana del explorador web, inicie sesión en la instancia de JIRA como administrador.

2. Mantenga el mouse encima del icono de engranaje y haga clic en **Complementos**.

    ![Configurar inicio de sesión único](./media/jira52microsoft-tutorial/addon1.png)

3. En la sección de la pestaña Complementos, haga clic en **Administrar complementos**.

    ![Configurar inicio de sesión único](./media/jira52microsoft-tutorial/addon7.png)

4. Descargue el complemento del [Centro de descarga de Microsoft](https://www.microsoft.com/download/details.aspx?id=56521). Cargue manualmente el complemento proporcionado por Microsoft mediante el menú **Cargar complemento**. La descarga del complemento está contemplada en el [Acuerdo de servicio de Microsoft](https://www.microsoft.com/servicesagreement/).

    ![Configurar inicio de sesión único](./media/jira52microsoft-tutorial/addon12.png)

5. Una vez instalado el complemento, aparece en la sección de complementos **Instalados por el usuario**. Haga clic en **Configurar** para configurar el nuevo complemento.

    ![Configurar inicio de sesión único](./media/jira52microsoft-tutorial/addon13.png)

6. Siga estos pasos en la página de configuración:

    ![Configurar inicio de sesión único](./media/jira52microsoft-tutorial/addon52.png)

    > [!TIP]
    > Asegúrese de que hay un solo certificado asignado a la aplicación, de forma que no se produzca ningún error en la resolución de los metadatos. Si hay varios certificados, después de resolver los metadatos, el administrador recibe un error.

    a. En el cuadro de texto **URL de metadatos**, pegue la **dirección URL de metadatos de federación de aplicación** que copió en Azure Portal y haga clic en el botón **Resolver**. Se lee la dirección URL de metadatos de IdP y se rellena toda la información de campos.

    b. Copie los valores **Identificador, Dirección URL de respuesta y Dirección URL de inicio de sesión**, y péguelos en los cuadros de texto **Identificador, Dirección URL de respuesta y Dirección URL de inicio de sesión** respectivamente en la sección **Configuración básica de SAML** de Azure Portal.

    c. En **Nombre del botón de inicio de sesión** escriba el nombre del botón que la organización quiere que los usuarios vean en la pantalla de inicio de sesión.

    d. En **SAML User ID Locations** (Ubicaciones de Id. de usuario de SAML) seleccione **User ID is in the NameIdentifier element of the Subject statement** (El Id. de usuario está en el elemento NameIdentifier de la instrucción Subject) o **User ID is in an Attribute element** (El Id. de usuario está en un elemento Attribute).  Este identificador debe ser el identificador de usuario de JIRA. Si el identificador de usuario no coincide, el sistema no permitirá que los usuarios inicien sesión.

    > [!Note]
    > La ubicación del Id. de usuario de SAML predeterminada es el identificador de nombre. Puede cambiarlo a una opción de atributo y escribir el nombre de atributo adecuado.

    e. Si selecciona la opción **User ID is in an Attribute element** (El Id. de usuario está en un elemento Attribute), escriba el nombre del atributo cuando se espera el id. de usuario en el cuadro de texto **Nombre del atributo**. 

    f. Si se usa el dominio federado (por ejemplo, ADFS, etc.) con Azure AD, haga clic en la opción **Habilitar detección de dominio principal** y configure el **nombre de dominio**.

    g. En **Nombre de dominio**, escriba el nombre del dominio en el caso de inicios de sesión basados en ADFS.

    h. Active **Enable Single Sign out** (Habilitar cierre de sesión único) si quiere que se cierre la sesión de Azure AD cuando un usuario cierre la sesión de JIRA. 

    i. Haga clic en el botón **Save** (Guardar) para guardar la configuración.

    > [!NOTE]
    > Para más información sobre la instalación y la solución de problemas, visite [MS JIRA SSO Connector Admin Guide](../ms-confluence-jira-plugin-adminguide.md) (Guía de administración del conector SSO de MS JIRA), que contiene una sección de [preguntas frecuentes](../ms-confluence-jira-plugin-faq.md) para ayudarle.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba `brittasimon\@yourcompanydomain.extension`. Por ejemplo, BrittaSimon@contoso.com.

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a JIRA SAML SSO by Microsoft (versión 5.2).

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y  **JIRA SAML SSO by Microsoft (versión 5.2)** .

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **JIRA SAML SSO by Microsoft (versión 5.2)** .

    ![El enlace de JIRA SAML SSO by Microsoft (versión 5.2) se encuentra en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-jira-saml-sso-by-microsoft-v52-test-user"></a>Creación de un usuario de prueba de JIRA SAML SSO by Microsoft (versión 5.2)

Para permitir que los usuarios de Azure AD inicien sesión en el servidor local de JIRA, se deben aprovisionar en el servidor local de JIRA.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión como administrador en el servidor local de JIRA.

2. Mantenga el mouse encima del icono de engranaje y haga clic en **Administración de usuarios**.

    ![Agregar empleado](./media/jira52microsoft-tutorial/user1.png)

3. Se le redirigirá a la página de acceso de administrador para especificar la **contraseña** y haga clic en el botón **Confirmar**.

    ![Agregar empleado](./media/jira52microsoft-tutorial/user2.png)

4. En la sección de la pestaña **Administración de usuarios**, haga clic en **Crear usuario**.

    ![Agregar empleado](./media/jira52microsoft-tutorial/user3.png) 

5. En la página del cuadro de diálogo **"Create New User"** (Crear nuevo usuario), realice los pasos siguientes:

    ![Agregar empleado](./media/jira52microsoft-tutorial/user4.png)

    a. En el cuadro de texto **Dirección de correo electrónico**, escriba la dirección de correo electrónico de un usuario, por ejemplo, Brittasimon@contoso.com.

    b. En el cuadro de texto **Nombre completo**, escriba el nombre completo de un usuario, por ejemplo, Britta Simon.

    c. En el cuadro de texto **Nombre de usuario**, escriba el correo electrónico de un usuario, por ejemplo, Brittasimon@contoso.com.

    d. En el cuadro de texto **Contraseña**, escriba la contraseña del usuario.

    e. Haga clic en **Crear usuario**.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de JIRA SAML SSO by Microsoft (versión 5.2) del panel de acceso, debería iniciar sesión automáticamente en la aplicación JIRA SAML SSO by Microsoft (versión 5.2) para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
