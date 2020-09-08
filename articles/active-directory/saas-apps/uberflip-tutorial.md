---
title: 'Tutorial: Integración de Azure Active Directory con Uberflip | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Uberflip.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/28/2019
ms.author: jeedes
ms.openlocfilehash: 8936c2150c32a22877f9b56736ecc4b8f3424b32
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88533187"
---
# <a name="tutorial-azure-active-directory-integration-with-uberflip"></a>Tutorial: Integración de Azure Active Directory con Uberflip

En este tutorial, aprenderá a integrar Uberflip con Azure Active Directory (Azure AD).

La integración de Uberflip con Azure AD proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Uberflip.
* Puede permitir que los usuarios inicien sesión automáticamente en Uberflip (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Para más información sobre la integración de aplicaciones de software como servicio (SaaS) con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Uberflip, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Una suscripción de Uberflip habilitada para el inicio de sesión único.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

Uberflip admite las siguientes características:

* Inicio de sesión único (SSO) con SP iniciado o IDP iniciado.
* Aprovisionamiento de usuarios Just-In-Time.

## <a name="add-uberflip-from-the-azure-marketplace"></a>Incorporación de Uberflip desde Azure Marketplace

Para configurar la integración de Uberflip en Azure AD, deberá agregar Uberflip desde Azure Marketplace a la lista de aplicaciones SaaS administradas:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el panel izquierdo, seleccione **Azure Active Directory**.

   ![Opción de Azure Active Directory](common/select-azuread.png)

1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

   ![Panel Aplicaciones empresariales](common/enterprise-applications.png)

1. Para agregar una aplicación nueva, en la parte superior del panel, seleccione **+ Nueva aplicación**.

   ![Opción Nueva aplicación](common/add-new-app.png)

1. En el cuadro de búsqueda, escriba **Uberflip**. En los resultados de búsqueda, seleccione **Uberflip** y, a continuación, seleccione **Agregar** para agregar la aplicación.

   ![Uberflip en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Uberflip con un usuario de prueba llamado **B Simon**. Para que el inicio de sesión único funcione, es preciso establecer un vínculo entre un usuario de Azure AD y un usuario relacionado de Uberflip.

Para configurar y probar el inicio de sesión único de Azure AD con Uberflip, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** , para permitir que los usuarios utilicen esta característica.
1. **[Configuración del inicio de sesión único de Uberflip](#configure-uberflip-single-sign-on)** para configurar los valores de inicio de sesión único en la aplicación.
1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B. Simon.
1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B. Simon para que use el inicio de sesión único de Azure AD.
1. **[Creación de un usuario de prueba de Uberflip](#create-an-uberflip-test-user)** , para que haya un usuario llamado B. Simon en Uberflip que esté vinculado a ese mismo nombre de usuario de Azure AD.
1. **[Prueba del inicio de sesión único](#test-single-sign-on)** , para comprobar si funciona la configuración.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Uberflip, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Uberflip**, seleccione **Inicio de sesión único**.

    ![Configuración de la opción de inicio de sesión único](common/select-sso.png)

1. En el panel **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

1. En la página **Configurar el inicio de sesión único con SAML**, seleccione **Editar** (icono de lápiz) para abrir el panel **Configuración básica de SAML**.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En el panel **Configuración básica de SAML**, realice uno de los pasos siguientes, según el modo de inicio de sesión único que quiera configurar:

   * Para configurar la aplicación en modo de inicio de sesión único con IDP iniciado, en el cuadro **Dirección URL de respuesta (URL del Servicio de consumidor de aserciones)** , escriba una dirección URL mediante el siguiente patrón:

     `https://app.uberflip.com/sso/saml2/<IDPID>/<ACCOUNTID>`

     ![Información sobre dominio y direcciones URL de inicio de sesión único de Uberflip](common/both-replyurl.png)

     > [!NOTE]
     > Este valor no es real. Actualice este valor con la dirección URL de respuesta real. Para obtener los valores reales, póngase en contacto con el [equipo de soporte técnico de Uberflip](mailto:support@uberflip.com). También puede hacer referencia a los patrones que se muestran en el panel **Configuración básica de SAML** de Azure Portal.

   * Para configurar la aplicación en modo SSO con SP iniciado, seleccione **Establecer direcciones URL adicionales** y, en el cuadro **URL de inicio de sesión**, escriba esta dirección URL:

     `https://app.uberflip.com/users/login`

     ![Información sobre dominio y direcciones URL de inicio de sesión único de Uberflip](common/both-signonurl.png)

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas y guárdelo en el equipo.

   ![Opción de descarga del XML de metadatos de federación](common/metadataxml.png)

1. En el panel **Configurar Uberflip**, copie las direcciones URL que necesite:

   * **Dirección URL de inicio de sesión**
   * **Identificador de Azure AD**
   * **Dirección URL de cierre de sesión**

   ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="configure-uberflip-single-sign-on"></a>Configuración del inicio de sesión único de Uberflip

Para configurar el inicio de sesión único en Uberflip, es preciso enviar el XML de metadatos de federación descargado y las direcciones URL apropiadas copiadas de Azure Portal al [equipo de soporte técnico de Uberflip](mailto:support@uberflip.com). El equipo de Uberflip se asegurará de que la conexión de inicio de sesión único de SAML esté establecida correctamente en ambos lados.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, creará un usuario de prueba llamado B. Simon en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory** > **Usuarios** > **Todos los usuarios**.

    ![Opciones "Usuarios" y "Todos los usuarios"](common/users.png)

1. En la parte superior de la pantalla, seleccione **+ Nuevo usuario**.

    ![Nueva opción de usuario](common/new-user.png)

1. En el panel **Usuario**, realice los pasos siguientes:

    ![Panel Usuario](common/user-properties.png)

    1. En el cuadro **Nombre**, escriba **BSimon**.
  
    1. En el cuadro **Nombre de usuario**, escriba **BSimon\@\<yourcompanydomain>.\<extension>** . Por ejemplo, **BSimon\@contoso.com**.

    1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.

    1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a B. Simon para que use el inicio de sesión único de Azure concediéndole acceso a Uberflip.

1. En Azure Portal, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones** > **Uberflip**.

    ![Panel Aplicaciones empresariales](common/enterprise-applications.png)

1. En la lista de aplicaciones, seleccione **Uberflip**.

    ![Uberflip en la lista de aplicaciones](common/all-applications.png)

1. En el panel izquierdo, seleccione **ADMINISTRAR** y **Usuarios y grupos**.

    ![Opción "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **+ Agregar usuario** y, después, seleccione **Usuarios y grupos** en el panel **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

1. En el panel **Usuarios y grupos**, seleccione **B Simon** en la lista **Usuarios** y, luego, elija **Seleccionar** en la parte inferior del panel.

1. Si espera algún valor de rol en la aserción de SAML, en el panel **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. Elija **Seleccionar** en la parte inferior del panel.

1. En el panel **Agregar asignación**, seleccione **Asignar**.

### <a name="create-an-uberflip-test-user"></a>Creación de un usuario de prueba de Uberflip

Ahora se crea un usuario llamado B. Simon en Uberflip. No tiene que hacer nada para crear este usuario. Uberflip admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. Si el usuario llamado B. Simon no existe en Uberflip, se crea uno después de la autenticación.

> [!NOTE]
> Si necesita crear manualmente un usuario, póngase en contacto con el [equipo de soporte técnico de Uberflip](mailto:support@uberflip.com).

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el portal Aplicaciones.

Al seleccionar **Uberflip** en el portal Aplicaciones, debería iniciar sesión automáticamente en la suscripción de Uberflip para la que configuró el inicio de sesión único. Para más información acerca del portal Aplicaciones, consulte [Access and use apps on the My Apps portal](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction) (Uso de aplicaciones y acceso a ellas en el portal Aplicaciones).

## <a name="additional-resources"></a>Recursos adicionales

* [Lista de tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

* [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
