---
title: 'Tutorial: Integración de Azure Active Directory con Sage Intacct | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Sage Intacct.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/05/2020
ms.author: jeedes
ms.openlocfilehash: 79a77d3c55ee4ac545712329f0054123bec8fcae
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88535870"
---
# <a name="tutorial-integrate-sage-intacct-with-azure-active-directory"></a>Tutorial: Integración de Sage Intacct con Azure Active Directory

En este tutorial, obtendrá información sobre cómo integrar Sage Intacct con Azure Active Directory (Azure AD). Cuando integra Sage Intacct con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Sage Intacct.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Sage Intacct con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Sage Intacct.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Sage Intacct admite el inicio de sesión único iniciado por **IDP**.
* Una vez configurado Sage Intacct, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-sage-intacct-from-the-gallery"></a>Incorporación de Sage Intacct desde la Galería

Para configurar la integración de Sage Intacct en Azure AD, será preciso agregar Sage Intacct desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Sage Intacct** en el cuadro de búsqueda.
1. Seleccione **Sage Intacct** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-sage-intacct"></a>Configuración y prueba del inicio de sesión único de Azure AD para Sage Intacct

Configure y pruebe el inicio de sesión único de Azure AD con Sage Intacct mediante un usuario de prueba llamado **B.Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Sage Intacct.

Para configurar y probar el inicio de sesión único de Azure AD con Sage Intacct, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
2. **[Configuración del inicio de sesión único en Sage Intacct](#configure-sage-intacct-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Sage Intacct](#create-sage-intacct-test-user)** , para tener un homólogo de B.Simon en Sage Intacct que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Sage Intacct**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    En el cuadro de texto **URL de respuesta**, escriba una dirección URL: `https://www.intacct.com/ia/acct/sso_response.phtml`

1. La aplicación Sage Intacct espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados. Haga clic en el icono **Editar** para abrir el cuadro de diálogo Atributos de usuario.

    ![imagen](common/edit-attribute.png)

1. Además de lo anterior, la aplicación Sage Intacct espera que se usen algunos atributos más en la respuesta de SAML. En el cuadro de diálogo **Atributos y notificaciones de usuario**, haga lo siguiente para agregar el atributo Token SAML como se muestra en la tabla siguiente:

    | Nombre del atributo  |  Atributo de origen|
    | ---------------| --------------- |
    | Nombre de la empresa | **Id. de la empresa de Sage Intacct** |
    | name | El valor debe ser el mismo que el **id. de usuario** de Sage Intacct, que debe especificar en la **sección Creación de un usuario de prueba de Sage Intacct** incluida más adelante en este tutorial. |

    a. Haga clic en **Agregar nueva notificación** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

    b. En el cuadro de texto **Nombre**, escriba el nombre que se muestra para la fila.

    c. Deje **Espacio de nombres** en blanco.

    d. Seleccione **Atributo** como origen.

    e. En la lista **Atributo de origen**, escriba o seleccione el valor de atributo que se muestra para esa fila.

    f. Haga clic en **Aceptar**.

    g. Haga clic en **Save**(Guardar).

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up Sage Intacct** (Configurar Sage Intacct), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Sage Intacct mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Sage Intacct**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-sage-intacct-sso"></a>Configuración del inicio de sesión único de Sage Intacct

1. En otra ventana del explorador web, inicie sesión en el sitio de la compañía de Sage Intacct como administrador.

1. Haga clic en la pestaña **Compañía** y luego haga clic en**Información de la compañía**.

    ![Company](./media/intacct-tutorial/ic790037.png "Compañía")

1. Haga clic en la pestaña **Seguridad** y luego haga clic en **Editar**.

    ![Seguridad](./media/intacct-tutorial/ic790038.png "Seguridad")

1. En la sección **Inicio de sesión único (SSO)** , siga estos pasos:

    ![Inicio de sesión único](./media/intacct-tutorial/ic790039.png "inicio de sesión único")

    a. Seleccione **Habilitar inicio de sesión único**.

    b. En **Tipo de proveedor de identidades**, seleccione **SAML 2.0**.

    c. En el cuadro de texto **URL del emisor**, pegue el valor de **Identificador de Azure AD**  que ha copiado de Azure Portal.

    d. En el cuadro de texto **Login URL**, (Dirección URL de inicio de sesión) pegue el valor de **Dirección URL de inicio de sesión**, que ha copiado de Azure Portal.

    e. Abra el certificado codificado en **base-64** en el Bloc de notas, copie el contenido del mismo en el Portapapeles y luego péguelo en el cuadro de texto **Certificado**.

    f. Haga clic en **Save**(Guardar).

### <a name="create-sage-intacct-test-user"></a>Creación de un usuario de prueba en Sage Intacct

Para configurar los usuarios de Azure AD para que puedan iniciar sesión en Sage Intacct, tienen que aprovisionarse en Sage Intacct. Para Sage Intacct, el aprovisionamiento es una tarea manual.

**Para aprovisionar cuentas de usuario, realice estos pasos:**

1. Inicie sesión en su inquilino de **Sage Intacct**.

1. Haga clic en la pestaña **Compañía** y luego haga clic en **Usuarios**.

    ![Usuarios](./media/intacct-tutorial/ic790041.png "Usuarios")

1. Haga clic en la pestaña **Agregar**.

    ![Add (Agregar)](./media/intacct-tutorial/ic790042.png "Sumar")

1. En la sección **Información del usuario** , lleve a cabo estos pasos:

    ![Información del usuario](./media/intacct-tutorial/ic790043.png "Información del usuario")

    a. Escriba el **Id. de usuario**, **Apellidos**, **Nombre**, **Dirección de correo electrónico**, **Puesto** y **Teléfono** de una cuenta de Azure AD que quiera aprovisionar en la sección **Información del usuario**.

    > [!NOTE]
    > Asegúrese de que el **id. de usuario**  de la captura de pantalla anterior y el valor de **atributo de origen**  que se asigna con el atributo **name** en la sección  **Atributos del usuario** de Azure Portal sea el mismo.

    b. Seleccione **Privilegios administrativos** de una cuenta de Azure AD que quiera aprovisionar.

    c. Haga clic en **Save**(Guardar). 
    
    d. El titular de la cuenta de Azure AD recibe un mensaje de correo y sigue un vínculo para confirmar su cuenta antes de que se active.

1. Haga clic en la pestaña **Inicio de sesión único** y asegúrese de que el **id. de usuario de SSO federado** de la captura de pantalla anterior y el valor de **atributo de origen** que se asigna con `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier` de la sección **Atributos de usuario** de Azure Portal sea el mismo.

    ![Información del usuario](./media/intacct-tutorial/ic790044.png "Información del usuario")

> [!NOTE]
> Para aprovisionar cuentas de usuario de Azure AD, puede usar cualquier otra API o herramienta de creación de cuentas de usuario de Sage Intacct ofrecida por Sage Intacct.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Sage Intacct en el Panel de acceso, debería iniciar sesión automáticamente en la aplicación Sage Intacct para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
