---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Concur Travel and Expense | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Concur Travel and Expense.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/10/2019
ms.author: jeedes
ms.openlocfilehash: 6797d169dbdfdd63ae9cc96ac9fed133bc8d1adc
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88544492"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-concur-travel-and-expense"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Concur Travel and Expense

En este tutorial, aprenderá a integrar Concur Travel and Expense con Azure Active Directory (Azure AD). Al integrar Concur Travel and Expense con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Concur Travel and Expense.
* Permitir que los usuarios inicien sesión automáticamente en Concur Travel and Expense con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción a Concur Travel and Expense.
* Un rol de "administrador de empresa" en su cuenta de usuario de Concur. Puede probar si tiene el acceso correcto; para ello, vaya a [Concur SSO Self-Service Tool](https://www.concursolutions.com/nui/authadmin/ssoadmin), la herramienta de autoservicio de inicio de sesión único de Concur. Si no tiene acceso, póngase en contacto con el servicio de soporte técnico de Concur o con el administrador de proyectos de implementación. 

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD.

* Concur Travel and Expense admite el inicio de sesión único iniciado por **IDP** y **SP**
* Concur Travel and Expense permite la prueba del inicio de sesión único tanto en entornos de producción como de implementación 

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo para cada una de las tres regiones: Estados Unidos, EMEA y China. Por lo tanto, solo se puede configurar una instancia para cada región de un inquilino. 

## <a name="adding-concur-travel-and-expense-from-the-gallery"></a>Incorporación de Concur Travel and Expense desde la galería

Para configurar la integración de Concur Travel and Expense en Azure AD, debe agregar Concur Travel and Expense desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Concur Travel and Expense** en el cuadro de búsqueda.
1. Seleccione **Concur Travel and Expense** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-concur-travel-and-expense"></a>Configuración y prueba del inicio de sesión único de Azure AD para Concur Travel and Expense

Configure y pruebe el inicio de sesión único de Azure AD con Concur Travel and Expense mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Concur Travel and Expense.

Para configurar y probar el inicio de sesión único de Azure AD con Concur Travel and Expense, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Concur Travel and Expense](#configure-concur-travel-and-expense-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Concur Travel and Expense](#create-concur-travel-and-expense-test-user)** , para tener un homólogo de B. Simon en Concur Travel and Expense que esté vinculado a la representación de Azure AD del usuario.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Concur Travel and Expense**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, la aplicación está preconfigurada en el modo iniciado por **IDP** y las direcciones URL necesarias ya se han rellenado previamente con Azure. El usuario debe guardar la configuración, para lo que debe hacer clic en el botón **Guardar**.

    > [!NOTE]
    > El identificador (Id. de entidad) y la dirección URL de respuesta (	URL del Servicio de consumidor de aserciones) son específicos de la región. Seleccione lo que corresponda en función del centro de datos de la entidad Concur. Si no conoce el centro de datos de la entidad Concur, póngase en contacto con el soporte técnico de Concur. 

5. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Atributo de usuario** y modificar la configuración. El identificador de usuario único debe coincidir con el valor de login_id del usuario de Concur. Normalmente, debe cambiar **user.userPrincipalName** a **user.mail**.

    ![Editar atributo de usuario](common/edit-attribute.png)

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el archivo de metadatos y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

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

En esta sección, va a permitir que B. Simon use el inicio de sesión único de Azure concediéndole acceso a Concur Travel and Expense.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Concur Travel and Expense**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-concur-travel-and-expense-sso"></a>Configuración del inicio de sesión único en Concur Travel and Expense

1. Para configurar el inicio de sesión único en **Concur Travel and Expense**, debe cargar el **XML de metadatos de federación** cargado en [Concur SSO Self-Service Tool](https://www.concursolutions.com/nui/authadmin/ssoadmin), la herramienta de autoservicio de SSO de Concur, e iniciar sesión con una cuenta que tenga el rol de administrador de la empresa. 

1. Haga clic en **Agregar**.
1. Escriba un nombre personalizado para el IdP, por ejemplo "Azure AD (EE. UU.)". 
1. Haga clic en **Cargar archivo XML** y adjunte el **XML de metadatos de federación** que descargó anteriormente.
1. Haga clic en **Agregar metadatos** para guardar el cambio.

    ![Captura de pantalla de la herramienta de autoservicio de SSO de Concur](./media/concur-travel-and-expense-tutorial/add-metadata-concur-self-service-tool.png)

### <a name="create-concur-travel-and-expense-test-user"></a>Creación de un usuario de prueba en Concur Travel and Expense

En esta sección creará un usuario llamado B. Simon en Concur Travel and Expense. En colaboración con el equipo de soporte técnico de Concur, agregará los usuarios a la plataforma de Concur Travel and Expense. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único. 

> [!NOTE]
> El identificador de inicio de sesión de Concur de B.Simon debe coincidir con el identificador único de B. Simon en Azure AD. Por ejemplo, si el identificador único de Azure AD de B.Simon es `B.Simon@contoso.com`. El identificador de inicio de sesión de Concur de B.Simon tiene que ser `B.Simon@contoso.com` también. 

## <a name="configure-concur-mobile-sso"></a>Configuración del inicio de sesión único para dispositivos móviles en Concur
Para habilitar el inicio de sesión único para dispositivos móviles en Concur, debe proporcionar al equipo de soporte técnico de Concur la **dirección URL de acceso del usuario**. Siga los pasos que se indican a continuación para obtener la **dirección URL de acceso del usuario** desde Azure AD:
1. Vaya a **Aplicaciones empresariales**.
1. Haga clic en **Concur Travel and Expense**.
1. Haga clic en **Propiedades**.
1. Copie la **dirección URL de acceso del usuario** y proporcione esta dirección URL al soporte técnico de Concur.

> [!NOTE]
> La opción de autoservicio para configurar el inicio de sesión único no está disponible, por lo que debe trabajar con el equipo de soporte técnico de Concur si desea habilitar el inicio de sesión único para dispositivos móviles. 

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Concur Travel and Expense en el Panel de acceso debería iniciar sesión automáticamente en la versión de Concur Travel and Expense para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Probar Concur Travel and Expense con Azure AD](https://aad.portal.azure.com/)

