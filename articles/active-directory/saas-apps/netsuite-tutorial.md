---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con NetSuite | Microsoft Docs'
description: Más información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y NetSuite.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/28/2020
ms.author: jeedes
ms.openlocfilehash: c5be6ef39a3e980852aa3373dbc54da63aa3983e
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88554522"
---
# <a name="tutorial-integrate-azure-ad-single-sign-on-sso-with-netsuite"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure AD con NetSuite

En este tutorial, aprenderá a integrar NetSuite con Azure Active Directory (Azure AD). Al integrar NetSuite con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a NetSuite.
* Permitir que los usuarios inicien sesión automáticamente en NetSuite con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central, Azure Portal.

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en NetSuite.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. 

NetSuite admite:

* El inicio de sesión único iniciado por IDP.
* El aprovisionamiento de usuarios Just-In-Time.
* El [aprovisionamiento automático de usuarios](NetSuite-provisioning-tutorial.md).
* Una vez configurado NetSuite, puede aplicar controles de sesión, que protegen la filtración y la infiltración de la información confidencial de la organización en tiempo real. Los controles de sesión proceden del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-aad).

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-netsuite-from-the-gallery"></a>Incorporación de NetSuite desde la galería

Para configurar la integración de NetSuite en Azure AD, agregue NetSuite desde la galería a la lista de aplicaciones SaaS administradas; para ello, haga lo siguiente:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta Microsoft personal, profesional o educativa.
1. En el panel izquierdo, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **NetSuite** en el cuadro de búsqueda.
1. En el panel de resultados, seleccione **NetSuite** y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-netsuite"></a>Configuración y prueba del inicio de sesión único de Azure AD para NetSuite

Configure y pruebe el inicio de sesión único de Azure AD con NetSuite con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de NetSuite.

Para configurar y probar el inicio de sesión único de Azure AD con NetSuite, es preciso completar los siguientes bloques de creación:

1. [Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso), para permitir que los usuarios puedan utilizar esta característica.
    * [Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user) para probar el inicio de sesión único de Azure AD con el usuario B.Simon.  
    * [Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user) para que el usuario B.Simon pueda usar el inicio de sesión único de Azure AD.
1. [Configuración del inicio de sesión único en NetSuite](#configure-netsuite-sso) para configurar los valores de inicio de sesión único en la aplicación.
    * [Creación de un usuario de prueba en NetSuite](#create-the-netsuite-test-user) para tener un homólogo de B.Simon en NetSuite vinculado a la representación del usuario en Azure AD.
1. [Pruebe el inicio de sesión único](#test-sso) para comprobar que la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Para habilitar el inicio de sesión único de Azure AD en Azure Portal, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **NetSuite**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En el panel **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, seleccione el icono **Editar** (lápiz) junto a **Configuración básica de SAML**.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, en el cuadro de texto **URL de respuesta**, escriba una dirección URL en uno de los formatos siguientes:

    ```https
    https://<Instance ID>.NetSuite.com/saml2/acs
    https://<Instance ID>.na1.NetSuite.com/saml2/acs
    https://<Instance ID>.na2.NetSuite.com/saml2/acs
    https://<Instance ID>.sandbox.NetSuite.com/saml2/acs
    https://<Instance ID>.na1.sandbox.NetSuite.com/saml2/acs
    https://<Instance ID>.na2.sandbox.NetSuite.com/saml2/acs
    ```

    * Obtendrá el valor **<`Instance ID`>** en la sección de configuración NetSuite, que se explica más adelante en el paso 8 del tutorial, en Configuración de NetSuite. Encontrará el dominio exacto (en este caso, system.na0.netsuite.com).

        ![Configurar inicio de sesión único](./media/NetSuite-tutorial/domain-value.png)

        > [!NOTE]
        > Los valores de las direcciones URL anteriores no son reales. Actualícelos con la dirección URL de respuesta real. Para obtener el valor, póngase en contacto con el [equipo de soporte técnico de cliente de NetSuite](http://www.netsuite.com/portal/services/support-services/suitesupport.shtml). También puede consultar los formatos que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación NetSuite espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación NetSuite espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

    | Nombre | Atributo de origen |
    | ---------------| --------------- |
    | account  | `account id` |

    > [!NOTE]
    > El valor del atributo account no es real. Actualizará este valor como se explica más adelante en este tutorial.

1. En la página Configurar el inicio de sesión único con SAML, en la sección Certificado de firma de SAML, busque XML de metadatos de federación y seleccione Descargar para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Set up NetSuite** (Configurar NetSuite), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En el panel izquierdo en Azure Portal, seleccione **Azure Active Directory** > **Usuarios** > **Todos los usuarios**.

1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

1. En el panel de propiedades **Usuario**, siga estos pasos:

   a. En el cuadro **Nombre**, escriba **B.Simon**.  
   b. En el cuadro **Nombre de usuario**, escriba username@companydomain.extension (por ejemplo, B.Simon@contoso.com).  
   c. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.  
   d. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección va a permitir que B.Simon acceda a NetSuite mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **NetSuite**.
1. En el panel de información general, busque la sección **Administrar** y seleccione el vínculo **usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario** y, en el panel **Agregar asignación**, **Usuarios y grupos**.

    ![Botón "Agregar usuario"](common/add-assign-user.png)

1. En el panel **Usuarios y grupos**, seleccione **B.Simon** en la lista **Usuarios** y el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera algún valor de rol en la aserción de SAML, haga lo siguiente:

   a. En la lista desplegable del panel **Seleccionar rol**, seleccione el rol adecuado para el usuario.  
   b. Seleccione el botón **Seleccionar** de la parte inferior de la pantalla.
1. En el panel **Agregar asignación**, seleccione el botón **Asignar**.

## <a name="configure-netsuite-sso"></a>Configuración del inicio de sesión único de NetSuite

1. Abra una nueva pestaña en el explorador e inicie sesión en el sitio de la empresa de NetSuite como administrador.

2. En la barra de navegación superior, seleccione **Setup**  (Configurar) y**Company**  > **Enable Features** (Empresa > Habilitar características).

    ![Configurar inicio de sesión único](./media/NetSuite-tutorial/ns-setupsaml.png)

3. En la barra de herramientas de la parte central de la página, seleccione **SuiteCloud**.

    ![Configurar inicio de sesión único](./media/NetSuite-tutorial/ns-suitecloud.png)

4. En **Manage Authentication** (Administrar autenticación), seleccione la casilla **SAML Single Sign-on** (Inicio de sesión único de SAML) para habilitar la opción de inicio de sesión único de SAML en NetSuite.

    ![Configurar inicio de sesión único](./media/NetSuite-tutorial/ns-ticksaml.png)

5. En la barra de navegación superior, seleccione **Setup** (Configurar).

    ![Configurar inicio de sesión único](./media/NetSuite-tutorial/ns-setup.png)

6. En la lista **Setup Tasks** (Tareas de configuración), seleccione **Integration** (Integración).

    ![Configurar inicio de sesión único](./media/NetSuite-tutorial/ns-integration.png)

7. En **Manage Authentication** (Administrar autenticación), seleccione **SAML Single Sign-on** (Inicio de sesión único de SAML).

    ![Configurar inicio de sesión único](./media/NetSuite-tutorial/ns-saml.png)

8. En la página **SAML Setup** (Configuración de SAML), **NetSuite Configuration** (Configuración de NetSuite), realice los pasos siguientes:

    ![Configurar inicio de sesión único](./media/NetSuite-tutorial/ns-saml-setup.png)
  
    a. Seleccione la casilla **Primary Authentication Method** (Método de autenticación principal).

    b. En **SAMLV2 Identity Provider Metadata** (Metadatos del proveedor de identidades de SAMLV2), seleccione **Upload IDP Metadata File** (Cargar archivo de metadatos de IDP) y, luego, **Browse** (Examinar) para cargar el archivo de metadatos que descargó de Azure Portal.

    c. Seleccione **Submit** (Enviar).

9. En la barra de navegación superior de NetSuite, seleccione **Setup**  (Configuración) y, a continuación, **Company**  > **Company Information** (Empresa > Información de la empresa).

    ![Configurar inicio de sesión único](./media/NetSuite-tutorial/ns-com.png)

    ![Configurar inicio de sesión único](./media/NetSuite-tutorial/ns-account-id.png)

    b. En la página **Company Information** (Información de la empresa), copie el valor de **Account ID** (Id. de cuenta).

    c. Pegue el valor de **Account ID** (Id. de cuenta) que ha copiado de la cuenta de NetSuite en el cuadro **Valor de atributo** de Azure AD.

10. Antes de que los usuarios puedan realizar el inicio de sesión único en NetSuite, se les deben asignar primero los permisos adecuados en NetSuite. Para asignar estos permisos, haga lo siguiente:

    a. En la barra de navegación superior, seleccione **Setup** (Configurar).

    ![Configurar inicio de sesión único](./media/NetSuite-tutorial/ns-setup.png)

    b. En el panel izquierdo, seleccione **Users/Roles** (Usuarios/Roles) y **Manage roles** (Administrar roles).

    ![Configurar inicio de sesión único](./media/NetSuite-tutorial/ns-manage-roles.png)

    c. Seleccione **New Role** (Nuevo rol).

    d. Escriba un nombre en **Name** (Nombre) para el nuevo rol.

    ![Configurar inicio de sesión único](./media/NetSuite-tutorial/ns-new-role.png)

    e. Seleccione **Guardar**.

    f. En la barra de navegación de la izquierda, seleccione **Permissions** (Permisos). Seleccione **Setup** (Configuración).

    ![Configurar inicio de sesión único](./media/NetSuite-tutorial/ns-sso.png)

    g. Seleccione **SAML Single Sign-on** (Inicio de sesión único de SAML) y **Add** (Agregar).

    h. Seleccione **Guardar**.

    i. En la barra de navegación superior, seleccione **Setup** (Configuración) y **Setup Manager** (Administrador de instalación).

    ![Configurar inicio de sesión único](./media/NetSuite-tutorial/ns-setup.png)

    j. En el panel izquierdo, seleccione **Users/Roles** (Usuarios/Roles) y **Manage Users** (Administrar usuarios).

    ![Configurar inicio de sesión único](./media/NetSuite-tutorial/ns-manage-users.png)

    k. Seleccione un usuario de prueba, **Edit** (Editar) y la pestaña **Access** (Acceder).

    ![Configurar inicio de sesión único](./media/NetSuite-tutorial/ns-edit-user.png)

    l. En el panel **Roles** (Roles), asigne el rol adecuado que haya creado.

    ![Configurar inicio de sesión único](./media/NetSuite-tutorial/ns-add-role.png)

    m. Seleccione **Guardar**.

### <a name="create-the-netsuite-test-user"></a>Creación de un usuario de prueba en NetSuite

En esta sección se crea un usuario llamado B.Simon en NetSuite. NetSuite admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en NetSuite, se crea uno nuevo después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al seleccionar el icono de NetSuite en el panel de acceso, debería iniciar sesión automáticamente en la versión de NetSuite para la que configurara el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)
- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)
- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
- [Pruebe NetSuite con Azure AD](https://aad.portal.azure.com/)
- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de NetSuite con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)