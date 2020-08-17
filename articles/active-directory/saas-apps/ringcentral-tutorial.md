---
title: 'Tutorial: integración de Azure Active Directory con RingCentral | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y RingCentral.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: celested
ms.assetid: 5848c875-5185-4f91-8279-1a030e67c510
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 10/24/2019
ms.author: jeedes
ms.openlocfilehash: 15dc4ee8a5c1021587bb5c37e634070a76b3f05d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87058227"
---
# <a name="tutorial-integrate-ringcentral-with-azure-active-directory"></a>Tutorial: Integración de RingCentral con Azure Active Directory

En este tutorial, aprenderá a integrar RingCentral con Azure Active Directory (Azure AD). Al integrar RingCentral con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a RingCentral.
* Permitir que los usuarios puedan iniciar sesión automáticamente en RingCentral con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción de RingCentral habilitada para el inicio de sesión único (SSO).

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* RingCentral admite el inicio de sesión único iniciado por **IDP**.

## <a name="adding-ringcentral-from-the-gallery"></a>Agregar RingCentral desde la Galería

Para configurar la integración de RingCentral en Azure AD, será preciso que agregue RingCentral desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **RingCentral** en el cuadro de búsqueda.
1. Seleccione **RingCentral** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

Configure y pruebe el inicio de sesión único de Azure AD con RingCentral utilizando un usuario de prueba llamado **Britta Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de RingCentral.

Para configurar y probar el inicio de sesión único de Azure AD con RingCentral, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    * **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    * **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en RingCentral](#configure-ringcentral-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    * **[Creación de un usuario de prueba en RingCentral](#create-ringcentral-test-user)** , para tener un homólogo de B.Simon en RingCentral que esté vinculado a su representación en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **RingCentral**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si tiene el **archivo de metadatos del proveedor de servicios**, lleve a cabo los siguientes pasos:

    1. Haga clic en **Cargar el archivo de metadatos**.
    1. Haga clic en el **logotipo de la carpeta** para seleccionar el archivo de metadatos y luego en **Cargar**.
    1. Una vez que se haya cargado correctamente el archivo de metadatos, los valores **Identificador** y **URL de respuesta** se rellenan automáticamente en la sección **Configuración básica de SAML**.

    > [!Note]
    > Obtendrá el **archivo de metadatos del proveedor de servicios** en la página Configuración de inicio de sesión único de RingCentral que se explica más adelante en el tutorial.

1. Si no tiene el **archivo de metadatos del proveedor de servicios**, escriba los valores para los campos siguientes:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL:
  
    | Identificador |
    |--|
    |  `https://sso.ringcentral.com` |
    | `https://ssoeuro.ringcentral.com` |

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL:

    | URL de respuesta |
    |--|
    | `https://sso.ringcentral.com/sp/ACS.saml2` |
    | `https://ssoeuro.ringcentral.com/sp/ACS.saml2` |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado Britta Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `Britta Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `BrittaSimon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a RingCentral.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **RingCentral**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En la lista de usuarios del cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-ringcentral-sso"></a>Configuración del inicio de sesión único de RingCentral

1. Para automatizar la configuración en RingCentral, debe instalar la **extensión de explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instale la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Configurar RingCentral** para ir a esta aplicación. En ella, escriba las credenciales de administrador para iniciar sesión en RingCentral. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 7.

    ![Configuración](common/setup-sso.png)

1. Si quiere configurar RingCentral manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de RingCentral como administrador y lleve a cabo los siguientes pasos:

1. En la parte superior, haga clic en **Herramientas**.

    ![imagen](./media/ringcentral-tutorial/ringcentral1.png)

1. Vaya a **Inicio de sesión único**.

    ![imagen](./media/ringcentral-tutorial/ringcentral2.png)

1. En la página **Inicio de sesión único**, en la sección **Configuración de inicio de sesión único**, desde **Paso 1** haga clic en **Editar** y realice los siguientes pasos:

    ![imagen](./media/ringcentral-tutorial/ringcentral3.png)

1. En la página **Configurar inicio de sesión único** realice los siguientes pasos:

    ![imagen](./media/ringcentral-tutorial/ringcentral4.png)

    a. Haga clic en **Examinar** para cargar el archivo de metadatos que descargó de Azure Portal.

    b. Después de cargar los metadatos los valores se rellenan automáticamente en la sección **Información general de inicio de sesión único**.

    c. En la sección **Asignación de atributos**, seleccione **Asignar atributo de correo electrónico a** como `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`

    d. Haga clic en **Save**(Guardar).

    e. En **Paso 2** haga clic en **Descargar** para descargar el **archivo de metadatos del proveedor de servicios** y cargarlo en la sección **Configuración básica de SAML** para rellenar de forma automática los valores **Identificador** y **URL de respuesta** en Azure Portal.

    ![imagen](./media/ringcentral-tutorial/ringcentral6.png) 

    f. En la misma página, vaya a la sección **Habilitar Inicio de sesión único** y realice los siguientes pasos:

    ![imagen](./media/ringcentral-tutorial/ringcentral5.png)

    * Seleccione **Habilitar inicio de sesión único**.

    * Seleccione **Permitir a los usuarios iniciar sesión con credenciales de inicio de sesión único o de RingCentral**.

    * Haga clic en **Save**(Guardar).

### <a name="create-ringcentral-test-user"></a>Creación de un usuario de prueba de RingCentral

En esta sección, creará un usuario llamado Britta Simon en RingCentral. Colabore con el [equipo de soporte técnico de RingCentral](https://success.ringcentral.com/RCContactSupp) para agregar usuarios en la plataforma de RingCentral. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

### <a name="test-sso"></a>Prueba de SSO

Al seleccionar el icono de RingCentral en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de RingCentral para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe RingCentral con Azure AD](https://aad.portal.azure.com/)
