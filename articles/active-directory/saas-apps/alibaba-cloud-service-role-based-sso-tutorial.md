---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Alibaba Cloud Service (Role-based SSO) | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Alibaba Cloud Service (inicio de sesión único basado en roles).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/13/2019
ms.author: jeedes
ms.openlocfilehash: 0e9ccb3f4308a1a75a715a16ab4c1a2887b0a915
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88522091"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-alibaba-cloud-service-role-based-sso"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Alibaba Cloud Service (Role-based SSO)

En este tutorial aprenderá a integrar Alibaba Cloud Service (Role-based SSO) con Azure Active Directory (Azure AD). Al integrar Alibaba Cloud Service (Role-based SSO) con Azure AD, se puede realizar lo siguiente:

* Controlar en Azure AD quién tiene acceso a Alibaba Cloud Service (Role-based SSO).
* Permitir que los usuarios inicien sesión automáticamente en Alibaba Cloud Service (Role-based SSO) con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Alibaba Cloud Service (Role-based SSO).

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Alibaba Cloud Service (inicio de sesión único basado en roles) admite SSO iniciado por **IDP**

## <a name="adding-alibaba-cloud-service-role-based-sso-from-the-gallery"></a>Agregar Alibaba Cloud Service (inicio de sesión único basado en roles) desde la galería

Para configurar la integración de Alibaba Cloud Service (inicio de sesión único basado en roles) en Azure AD, es preciso agregar Alibaba Cloud Service (inicio de sesión único basado en roles) desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Alibaba Cloud Service (Role-based SSO)** en el cuadro de búsqueda.
1. Seleccione **Alibaba Cloud Service (Role-based SSO)** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.
5. En la página de **Alibaba Cloud Service (inicio de sesión único basado en roles)** , haga clic en **Properties** (Propiedades) en el panel de navegación izquierdo, copie el **identificador de objeto** y guárdelo en su equipo para usarlo más adelante.

    ![Configuración de propiedades](./media/alibaba-cloud-service-role-based-sso-tutorial/Properties.png)


## <a name="configure-and-test-azure-ad-single-sign-on-for-alibaba-cloud-service-role-based-sso"></a>Configuración y prueba del inicio de sesión único de Azure AD para Alibaba Cloud Service (Role-based SSO)

Configure y pruebe el inicio de sesión único de Azure AD con Alibaba Cloud Service (Role-based SSO) mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Alibaba Cloud Service (Role-based SSO).

Para configurar y probar el inicio de sesión único de Azure AD con Alibaba Cloud Service (Role-based SSO), es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
2. **[Configure el inicio de sesión único basado en roles de Alibaba Cloud Service](#configure-role-based-single-sign-on-in-alibaba-cloud-service)** para permitir que los usuarios puedan usar esta característica.
    1. **[Configuración del inicio de sesión único de Alibaba Cloud Service (Role-based SSO)](#configure-alibaba-cloud-service-role-based-sso-sso)** , para configurar los valores del inicio de sesión único en la aplicación.
    1. **[Creación de usuarios de prueba de Alibaba Cloud Service (inicio de sesión único basado en roles)](#create-alibaba-cloud-service-role-based-sso-test-user)** : para tener un homólogo de Britta Simon en Alibaba Cloud Service (inicio de sesión único basado en roles) que esté vinculado a la representación de usuario en Azure AD.
3. **[Prueba del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Alibaba Cloud Service (Role-based SSO)** , busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si tiene el **archivo de metadatos del proveedor de servicios**, lleve a cabo los siguientes pasos:

    >[!NOTE]
    >Obtendrá los metadatos del proveedor de servicios de esta [dirección URL](https://signin.alibabacloud.com/saml-role/sp-metadata.xml)

    a. Haga clic en **Cargar el archivo de metadatos**.

    b. Haga clic en el **logotipo de la carpeta** para seleccionar el archivo de metadatos y luego en **Cargar**.

    c. Una vez que se haya cargado correctamente el archivo de metadatos, los valores **Identificador** y **Dirección URL de respuesta** se rellenan automáticamente en el cuadro de texto de la sección Alibaba Cloud Service (inicio de sesión único basado en roles):

    > [!Note]
    > Si los valores **Identificador** y **Dirección URL de respuesta** no se rellenan automáticamente, hágalo manualmente según sus necesidades.

1. Alibaba Cloud Service (Role-based SSO) necesita que se configuren roles en Azure AD. La notificación de rol está preconfigurada, por lo que no tiene que ocuparse de ello, pero aún tiene que crear los roles en Azure AD con la ayuda de este [artículo](https://docs.microsoft.com/azure/active-directory/develop/active-directory-enterprise-app-role-management).

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar Alibaba Cloud Service (Role-based SSO)** , copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir que B.Simon use el inicio de sesión único de Azure concediéndole acceso a Alibaba Cloud Service (Role-based SSO).

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Alibaba Cloud Service (inicio de sesión único basado en roles)** .
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En la pestaña **Usuarios y grupos**, seleccione u2 en la lista de usuarios y haga clic en **Seleccionar**. Después, haga clic en **Asignar**.

    ![Prueba de la configuración](./media/alibaba-cloud-service-role-based-sso-tutorial/test01.png)

1. Vea el rol asignado y pruebe Alibaba Cloud Service (inicio de sesión único basado en roles).

    ![Prueba de la configuración](./media/alibaba-cloud-service-role-based-sso-tutorial/test02.png)

    >[!NOTE]
    >Después de asignar el usuario (u2), se asociará automáticamente el rol creado al usuario. Si ha creado varios roles, deberá asociar el rol adecuado al usuario según sea necesario. Si desea implementar el inicio de sesión único basado en roles de Azure AD a varias cuentas de Alibaba Cloud, repita los pasos anteriores.

## <a name="configure-role-based-single-sign-on-in-alibaba-cloud-service"></a>Configuración del inicio de sesión único basado en roles de Alibaba Cloud Service

1. Inicie sesión en la [consola RAM](https://account.alibabacloud.com/login/login.htm?oauth_callback=https%3A%2F%2Fram.console.aliyun.com%2F%3Fspm%3Da2c63.p38356.879954.8.7d904e167h6Yg9) de Alibaba Cloud Service mediante Account1.

2. En el panel de navegación izquierdo, seleccione **SSO**.

3. En la pestaña **Role-based SSO** (Inicio de sesión único basado en roles), haga clic en **Create IdP** (Crear IdP).

4. En la página que aparece, escriba `AAD` en el campo IdP Name (Nombre de IdP), escriba una descripción en el campo **Note** (Nota), haga clic en **Upload** (Cargar) para cargar el archivo de metadatos de federación que descargó anteriormente y, después, haga clic en **OK** (Aceptar).

5. Una vez creado el IdP satisfactoriamente, haga clic en **Create RAM Role** (Crear rol de RAM).

6. En el campo **RAM Role Name** (Nombre de rol de RAM) escriba `AADrole`, seleccione `AAD` en la lista desplegable **Select IdP** (Seleccionar IdP) y haga clic en OK (Aceptar).

    >[!NOTE]
    >Puede conceder al rol los permisos que sean necesarios. Después de crear el proveedor de identidades y el rol correspondiente, se recomienda que guarde los ARN del IdP y el rol para su uso posterior. Puede obtener los ARN en la página de información del IdP y en la página de información del rol.

7. Asocie el rol de la RAM de Alibaba Cloud Service (AADrole) con el usuario de Azure AD (u2): Para asociar el rol de la RAM con el usuario de Azure AD, debe crear un rol en Azure AD mediante estos pasos:

    a. Inicie sesión en el [explorador de Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer).

    b. Haga clic en **Modificar permisos** para obtener los permisos necesarios para crear un rol.

    ![Configuración de Graph](./media/alibaba-cloud-service-role-based-sso-tutorial/graph01.png)

    c. Seleccione los permisos siguientes en la lista y haga clic en **Modificar permisos**, como se muestra en la ilustración siguiente.

    ![Configuración de Graph](./media/alibaba-cloud-service-role-based-sso-tutorial/graph02.png)

    >[!NOTE]
    >Una vez concedidos los permisos, inicie sesión en el Explorador de Graph de nuevo.

    d. En la página del Explorador de Graph, seleccione **GET** en la primera lista desplegable y **beta** en la segunda. A continuación, escriba `https://graph.microsoft.com/beta/servicePrincipals` en el campo situado junto a las listas desplegables y haga clic en **Ejecutar consulta**.

    ![Configuración de Graph](./media/alibaba-cloud-service-role-based-sso-tutorial/graph03.png)

    >[!NOTE]
    >Si usa varios directorios, puede escribir `https://graph.microsoft.com/beta/contoso.com/servicePrincipals` en el campo de la consulta.

    e. En la sección **Response Preview** (Vista previa de la respuesta), extraiga la propiedad appRoles de "Entidad de servicio" para su uso posterior.

    ![Configuración de Graph](./media/alibaba-cloud-service-role-based-sso-tutorial/graph05.png)

    >[!NOTE]
    >Puede encontrar la propiedad appRoles escribiendo `https://graph.microsoft.com/beta/servicePrincipals/<objectID>` en el campo de la consulta. Tenga en cuenta que `objectID` es el identificador del objeto que ha copiado de la página de **propiedades** de Azure AD.

    f. Vuelva al Explorador de Graph, cambie el método de **GET** a **PATCH**, pegue el siguiente contenido en la sección **Cuerpo de la solicitud** y haga clic en **Ejecutar consulta**:
    ```
    { 
    "appRoles": [
        { 
        "allowedMemberTypes":[
            "User"
        ],
        "description": "msiam_access",
        "displayName": "msiam_access",
        "id": "41be2db8-48d9-4277-8e86-f6d22d35****",
        "isEnabled": true,
        "origin": "Application",
        "value": null
        },
        { "allowedMemberTypes": [
            "User"
        ],
        "description": "Admin,AzureADProd",
        "displayName": "Admin,AzureADProd",
        "id": "68adae10-8b6b-47e6-9142-6476078cdbce",
        "isEnabled": true,
        "origin": "ServicePrincipal",
        "value": "acs:ram::187125022722****:role/aadrole,acs:ram::187125022722****:saml-provider/AAD"
        }
    ]
    }
    ```
    > [!NOTE]
    > `value` es el ARN del proveedor de identidades y el rol que creó en la consola de la RAM. En este caso, puede agregar varios roles según sea necesario. Azure AD enviará el valor de estos roles como valor de notificación en la respuesta de SAML. Sin embargo, solo puede agregar nuevos roles después del elemento `msiam_access` para la operación de revisión. Para facilitar el proceso de creación, se recomienda que use un generador de identificadores, como GUID Generator, para generar identificadores en tiempo real.

    g. Una vez que haya revisado la entidad de servicio con el rol necesario, asocie el rol con el usuario de Azure AD (u2) siguiendo los pasos de la sección **Asignación del usuario de prueba de Azure AD** del tutorial.

### <a name="configure-alibaba-cloud-service-role-based-sso-sso"></a>Configuración del inicio de sesión único de Alibaba Cloud Service (Role-based SSO)

Para configurar el inicio de sesión único en **Alibaba Cloud Service (inicio de sesión único basado en roles)** , es preciso enviar el **XML de metadatos de federación** descargado y las direcciones URL apropiadas copiadas de Azure Portal al [equipo de soporte técnico de Alibaba Cloud Service (inicio de sesión único basado en roles)](https://www.aliyun.com/service/). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-alibaba-cloud-service-role-based-sso-test-user"></a>Creación de un usuario de prueba en Alibaba Cloud Service (inicio de sesión único basado en roles)

En esta sección, se crea un usuario llamado en Britta Simon en Alibaba Cloud Service (inicio de sesión único basado en roles). Trabaje con el [equipo de soporte técnico de Alibaba Cloud Service (inicio de sesión único basado en roles)](https://www.aliyun.com/service/) para agregar los usuarios a la plataforma de Alibaba Cloud Service (inicio de sesión único basado en roles). Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO 

Una vez completadas las configuraciones anteriores, pruebe Alibaba Cloud Service (inicio de sesión único basado en roles) con estos pasos:

1. En Azure Portal, vaya a la página de **Alibaba Cloud Service (inicio de sesión único basado en roles)** , seleccione **Inicio de sesión único** y haga clic en **Probar**.

    ![Prueba de la configuración](./media/alibaba-cloud-service-role-based-sso-tutorial/test03.png)

2. Haga clic en **Iniciar sesión en nombre del usuario actual**.

    ![Prueba de la configuración](./media/alibaba-cloud-service-role-based-sso-tutorial/test04.png)

3. En la página de selección de la cuenta, seleccione u2.

    ![Prueba de la configuración](./media/alibaba-cloud-service-role-based-sso-tutorial/test05.png)

4. Se abrirá la página siguiente, que indica que el inicio de sesión único basado en roles es correcto.

    ![Prueba de la configuración](./media/alibaba-cloud-service-role-based-sso-tutorial/test06.png)

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Alibaba Cloud Service (Role-based SSO) con Azure AD](https://aad.portal.azure.com/)

