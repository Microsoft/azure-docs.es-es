---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con TimeOffManager | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y TimeOffManager.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/10/2019
ms.author: jeedes
ms.openlocfilehash: 59eb2e320a828b644af6f740cf9f8381370a0dda
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88527356"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-timeoffmanager"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con TimeOffManager

En este tutorial, obtendrá información sobre cómo integrar TimeOffManager con Azure Active Directory (Azure AD). Al integrar TimeOffManager con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a TimeOffManager.
* Permitir que los usuarios inicien sesión automáticamente en TimeOffManager con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en TimeOffManager.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.


* TimeOffManager admite inicio de sesión único iniciado por **IDP**

* TimeOffManager admite aprovisionamiento de usuarios **Just-in-time**

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.


## <a name="adding-timeoffmanager-from-the-gallery"></a>Adición de TimeOffManager desde la galería

Para configurar la integración de TimeOffManager en Azure AD, deberá agregar TimeOffManager desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **TimeOffManager** en el cuadro de búsqueda.
1. Seleccione **TimeOffManager** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-single-sign-on-for-timeoffmanager"></a>Configuración y prueba del inicio de sesión único de Azure AD para TimeOffManager

Configure y pruebe el inicio de sesión único de Azure AD con TimeOffManager con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de TimeOffManager.

Para configurar y probar el inicio de sesión único de Azure AD con TimeOffManager, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de TimeOffManager](#configure-timeoffmanager-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de TimeOffManager](#create-timeoffmanager-test-user)** , para tener un homólogo de B.Simon en TimeOffManager que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **TimeOffManager**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://www.timeoffmanager.com/cpanel/sso/consume.aspx?company_id=<companyid>`

    > [!NOTE]
    > Este valor no es real. Actualice este valor con la dirección URL de respuesta real. Puede obtener este valor en la **página de configuración de Inicio de sesión único** que se explica más adelante en el tutorial o ponerse en contacto con el [equipo de soporte técnico de TimeOffManager](https://www.purelyhr.com/contact-us). También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación TimeOffManager espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/edit-attribute.png)

1. Además de lo anterior, la aplicación TimeOffManager espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

    | Nombre | Atributo de origen|
    | --- | --- |
    | Firstname |User.givenname |
    | Lastname |User.surname |
    | Email |User.mail |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up TimeOffManager** (Configurar TimeOffManager), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a conceder a B.Simon acceso a TimeOffManager utilizando el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **TimeOffManager**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-timeoffmanager-sso"></a>Configuración del inicio de sesión único de TimeOffManager

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la compañía de TimeOffManager.

2. Vaya a **Cuenta \> Opciones de cuenta \> Configuración de inicio de sesión único**.
   
    ![Configuración de inicio de sesión único](./media/timeoffmanager-tutorial/ic795917.png "Configuración de inicio de sesión único")

3. En la sección **Configuración del inicio de sesión único** , siga estos pasos:
   
    ![Configuración de inicio de sesión único](./media/timeoffmanager-tutorial/ic795918.png "Configuración de inicio de sesión único")
   
    a. Abra el certificado codificado en base 64 en el Bloc de notas, copie su contenido en el Portapapeles y luego pegue todo el certificado en el cuadro de texto **Certificado X.509** .
   
    b. En el cuadro de texto **Idp Issuer** (Emisor de IdP), pegue el valor de **Identificador Azure AD** que ha copiado de Azure Portal.
   
    c. En el cuadro de texto **IdP EndPoint URL** (URL de punto de conexión de IdP), pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.
   
    d. En **Aplicar SAML**, seleccione **No**.
   
    e. En **Crear usuarios automáticamente**, seleccione **Sí**.
   
    f. En el cuadro de texto **URL de cierre de sesión**, pegue el valor de **Sign-Out URL** (Dirección URL de cierre de sesión) que copió de Azure Portal.
   
    g. Haga clic en **Guardar cambios**.

4. En la página **Configuración de inicio de sesión único**, copie el valor de **URL del Servicio de consumidor de aserciones** y péguelo en el cuadro de texto **URL de respuesta** en la sección **Configuración básica de SAML** de Azure Portal. 

      ![Configuración de inicio de sesión único](./media/timeoffmanager-tutorial/ic795915.png "Configuración de inicio de sesión único")

### <a name="create-timeoffmanager-test-user"></a>Creación de un usuario de prueba de TimeOffManager

En esta sección, se crea un usuario llamado Britta Simon en TimeOffManager. TimeOffManager admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en TimeOffManager, se crea uno nuevo después de la autenticación.

>[!NOTE]
>Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de TimeOffManager ofrecida por TimeOffManager para aprovisionar cuentas de usuario de Azure AD.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de TimeOffManager en el panel de acceso y debería iniciar sesión automáticamente en la versión de TimeOffManager para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe TimeOffManager con Azure AD](https://aad.portal.azure.com/)

