---
title: 'Tutorial: Integración de Azure Active Directory con HubSpot | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y HubSpot.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/14/2019
ms.author: jeedes
ms.openlocfilehash: 51e27526b10bcdd74d6e2bffb8bf620d7b022aac
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88551479"
---
# <a name="tutorial-azure-active-directory-integration-with-hubspot"></a>Tutorial: Integración de Azure Active Directory con HubSpot

En este tutorial, aprenderá a integrar HubSpot con Azure Active Directory (Azure AD).

La integración de HubSpot con Azure AD le proporciona las siguientes ventajas:

* Puede usar Azure AD para controlar quién tiene acceso a HubSpot.
* Puede permitir que los usuarios inicien sesión automáticamente en HubSpot (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Para más información acerca de la integración de aplicaciones SaaS (software como servicio) con Azure AD, consulte [Inicio de sesión único en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con HubSpot, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción de Azure AD, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Una suscripción de HubSpot habilitada para el inicio de sesión único.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba e integrar HubSpot con Azure AD.

HubSpot admite las siguientes características:

* **Inicio de sesión único iniciado por SP**
* **Inicio de sesión único iniciado por IDP**

## <a name="add-hubspot-in-the-azure-portal"></a>Adición de HubSpot en Azure Portal

Para integrar HubSpot con Azure AD, debe agregar HubSpot a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Seleccione **Azure Active Directory** en el menú izquierdo.

    ![Opción de Azure Active Directory](common/select-azuread.png)

1. Seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.

    ![Panel Aplicaciones empresariales](common/enterprise-applications.png)

1. Para agregar una aplicación, seleccione **Nueva aplicación**.

    ![Opción Nueva aplicación](common/add-new-app.png)

1. En el cuadro de búsqueda, escriba **HubSpot**. En los resultados de búsqueda, seleccione **HubSpot** y, luego, **Agregar**.

    ![HubSpot en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con HubSpot con un usuario de prueba llamado **Britta Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de HubSpot.

Para configurar y probar el inicio de sesión único de Azure AD con HubSpot, es preciso completar los siguientes bloques de creación:

| Tarea | Descripción |
| --- | --- |
| **[Configuración del inicio de sesión único en Azure AD](#configure-azure-ad-single-sign-on)** | Permite que los usuarios usen esta característica. |
| **[Configuración del inicio de sesión único de HubSpot](#configure-hubspot-single-sign-on)** | Configura los valores de inicio de sesión único en la aplicación. |
| **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** | Prueba el inicio de sesión único de Azure AD con el usuario Britta Simon. |
| **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** | Permite que Britta Simon use el inicio de sesión único de Azure AD. |
| **[Creación de un usuario de prueba de HubSpot](#create-a-hubspot-test-user)** | Crea un homólogo de Britta Simon en HubSpot que está vinculado a la representación del usuario en Azure AD. |
| **[Prueba de inicio de sesión único](#test-single-sign-on)** | Comprueba que la configuración funciona. |

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, configurará el inicio de sesión único de Azure AD con HubSpot en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en el panel de integración de aplicaciones de **HubSpot**, seleccione **Inicio de sesión único**.

    ![Configuración de la opción de inicio de sesión único](common/select-sso.png)

1. En el panel **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML** o **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

1. En el panel **Configurar el inicio de sesión único con SAML**, seleccione **Editar** (icono de lápiz) para abrir el panel **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En el panel **Configuración básica de SAML**, para configurar el *modo iniciado por IDP*, siga estos pasos:

    1. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón: https:\//api.hubspot.com/login-api/v1/saml/login?portalId=\<CUSTOMER ID\>.

    1. En el cuadro de texto **Dirección URL de respuesta**, escriba una dirección URL con el siguiente patrón: https:\//api.hubspot.com/login-api/v1/saml/acs?portalId=\<CUSTOMER ID\>.

    ![Información de dominio y direcciones URL de inicio de sesión único de HubSpot](common/idp-intiated.png)

    > [!NOTE]
    > Para dar formato a las direcciones URL, también puede hacer referencia a los patrones que se muestran en el panel **Configuración básica de SAML** de Azure Portal.

1. Para configurar la aplicación en modo *iniciado por SP*:

    1. Seleccione **Establecer direcciones URL adicionales**.

    1. En el cuadro de texto **Dirección URL de inicio de sesión**, escriba **https:\//app.hubspot.com/login**.

    ![Opción Establecer direcciones URL adicionales](common/metadata-upload-additional-signon.png)

1. En el panel **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione **Descargar** junto a **Certificado (base 64)** . Seleccione una opción de descarga según sus requisitos. Guarde el certificado en el equipo.

    ![Opción de descarga del certificado (Base64)](common/certificatebase64.png)

1. En la sección **Configurar HubSpot**, copie las direcciones URL siguientes según sus necesidades:

    * URL de inicio de sesión
    * Identificador de Azure AD
    * URL de cierre de sesión

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="configure-hubspot-single-sign-on"></a>Configuración del inicio de sesión único de HubSpot

1. Abra una nueva pestaña en el explorador e inicie sesión en su cuenta de administrador de HubSpot.

1. Seleccione el icono **Settings** (Configuración) en la esquina superior derecha de la página.

    ![Icono de configuración de HubSpot](./media/hubspot-tutorial/config1.png)

1. Seleccione **Account Defaults** (Valores predeterminados de cuenta).

    ![Opción Valores predeterminados de cuenta en HubSpot](./media/hubspot-tutorial/config2.png)

1. Desplácese hacia abajo hasta la sección **Security** (Seguridad) y, a continuación, seleccione **Set up** (Configurar).

    ![Opción de configuración en HubSpot](./media/hubspot-tutorial/config3.png)

1. En la sección **Set Up Single Sign-On** (Configurar inicio de sesión único), realice los pasos siguientes:

    1. En el cuadro de texto **Audience URl (Service Provider Entity ID)** (Identificador URl de audiencia [identificador de entidad del proveedor de servicio]), seleccione **Copy** (Copiar) para copiar el valor. En Azure Portal, en el panel **Configuración básica de SAML**, pegue el valor en el cuadro **Identificador**.

    1. En el cuadro de texto **Sign on URl, ACS, Recipient, or Redirect**, seleccione **Copy** (Copiar) para copiar el valor. En Azure Portal, en el panel **Configuración básica de SAML**, pegue el valor en el cuadro **Dirección URL de respuesta**.

    1. En HubSpot, en el cuadro de texto **Identity Provider Identifier or Issuer URL** (Dirección URL del emisor o identificador de proveedor de identidades), pegue el valor de **Identificador de Azure AD** que ha copiado de Azure Portal.

    1. En HubSpot, en el cuadro de texto **Identity Provider Single Sign-On URL** (dirección URL de inicio de sesión único del proveedor de identidades), pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    1. En el Bloc de notas de Windows, abra el archivo de certificado (base64) descargado. Seleccione y copie el contenido del archivo. A continuación, en HubSpot, péguelo en el cuadro de texto **X.509 Certificate** (Certificado X.509).

    1. Seleccione **Comprobar**.

        ![Sección de configuración del inicio de sesión único en HubSpot](./media/hubspot-tutorial/config4.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, creará un usuario de prueba llamado Britta Simon en Azure Portal.

1. En Azure Portal, seleccione **Azure Active Directory** > **Usuarios** > **Todos los usuarios**.

    ![Opciones Usuarios y Todos los usuarios](common/users.png)

1. Seleccione **Nuevo usuario**.

    ![Opción Nuevo usuario](common/new-user.png)

1. En el panel **Usuario**, siga estos pasos:

    1. En el cuadro **Nombre**, escriba **BrittaSimon**.
  
    1. En el cuadro **Nombre de usuario**, escriba **brittasimon\@\<your-company-domain>.\<extension\>** . Por ejemplo, **brittasimon\@contoso.com**.

    1. Active la casilla de verificación **Mostrar contraseña**. Anote el valor que se muestra en el cuadro **Contraseña**.

    1. Seleccione **Crear**.

    ![Panel Usuario](common/user-properties.png)

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, concederá a Britta Simon acceso a HubSpot para que use el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones** > **HubSpot**.

    ![Panel Aplicaciones empresariales](common/enterprise-applications.png)

1. En la lista de aplicaciones, seleccione **HubSpot**.

    ![HubSpot en la lista de aplicaciones](common/all-applications.png)

1. En el menú, seleccione **Usuarios y grupos**.

    ![Opción Usuarios y grupos](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. Después, en el panel **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![El panel Agregar asignación](common/add-assign-user.png)

1. En el panel **Usuarios y grupos**, en la lista de usuarios, seleccione **Britta Simon**. Elija **Seleccionar**.

1. Si espera algún valor de rol en la aserción de SAML, en el panel **Seleccionar rol**, seleccione el rol adecuado para el usuario de la lista. Elija **Seleccionar**.

1. En el panel **Agregar asignación**, seleccione **Asignar**.

### <a name="create-a-hubspot-test-user"></a>Creación de un usuario de prueba de HubSpot

Para permitir que los usuarios de Azure AD inicien sesión en HubSpot, deben aprovisionarse en HubSpot. En HubSpot, el aprovisionamiento es una tarea manual.

Para aprovisionar una cuenta de usuario en HubSpot:

1. Inicie sesión en el sitio de la compañía de HubSpot como administrador.

1. Seleccione el icono **Settings** (Configuración) en la esquina superior derecha de la página.

    ![Icono de configuración de HubSpot](./media/hubspot-tutorial/config1.png)

1. Seleccione **Users & Teams** (Usuarios y equipos).

    ![Opción de usuarios y equipos de HubSpot](./media/hubspot-tutorial/user1.png)

1. Seleccione **Create User** (Crear usuario).

    ![Opción Crear usuario en HubSpot](./media/hubspot-tutorial/user2.png)

1. En el cuadro **Add email addess(es)** (Agregar direcciones de correo electrónico), escriba la dirección de correo electrónico del usuario con el formato brittasimon\@contoso.com y, a continuación, seleccione **Next** (Siguiente).

    ![Cuadro de direcciones de correo electrónico de la sección Crear usuarios de HubSpot](./media/hubspot-tutorial/user3.png)

1. En la sección **Create users** (Crear usuarios), seleccione cada pestaña. En cada pestaña, establezca las opciones y los permisos adecuados para el usuario. Después, seleccione **Siguiente**.

    ![Pestañas de la sección Crear usuarios de Hubspot](./media/hubspot-tutorial/user4.png)

1. Seleccione **Send** (Enviar) para enviar la invitación al usuario.

    ![Opción de envío de HubSpot](./media/hubspot-tutorial/user5.png)

    > [!NOTE]
    > El usuario se activa después de que el usuario acepta la invitación.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el portal Aplicaciones.

Después de configurar el inicio de sesión único, cuando se selecciona **HubSpot** en el portal Aplicaciones, se inicia automáticamente sesión en HubSpot. Para más información acerca del portal Aplicaciones, consulte [Acceso y uso de aplicaciones en el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte estos artículos:

- [Lista de tutoriales para integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)
- [Inicio de sesión único en aplicaciones en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)
- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
