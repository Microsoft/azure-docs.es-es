---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con ContractSafe Saml2 SSO | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y ContractSafe Saml2 SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/20/2019
ms.author: jeedes
ms.openlocfilehash: 9459d13e8943f7e9177d2ad493873257f46f6649
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88544404"
---
# <a name="tutorial-integrate-azure-active-directory-single-sign-on-sso-with-contractsafe-saml2-sso"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con ContractSafe Saml2 SSO

En este tutorial, aprenderá a integrar ContractSafe Saml2 SSO con Azure Active Directory (Azure AD). Al integrar ContractSafe Saml2 SSO con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a ContractSafe Saml2 SSO.
* Permitir que los usuarios inicien sesión automáticamente en ContractSafe Saml2 SSO con sus cuentas de Azure AD.
* Administrar sus cuentas en una ubicación central: Azure Portal.

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesitará lo siguiente:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción de ContractSafe Saml2 SSO con inicio de sesión único habilitado.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. ContractSafe Saml2 SSO admite el inicio de sesión único iniciado por **IDP**.

## <a name="add-contractsafe-saml2-sso-from-the-gallery"></a>Incorporación de ContractSafe Saml2 SSO desde la galería

Para configurar la integración de ContractSafe Saml2 SSO en Azure AD, debe agregar ContractSafe Saml2 SSO desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **ContractSafe Saml2 SSO** en el cuadro de búsqueda.
1. Seleccione **ContractSafe Saml2 SSO** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-contractsafe-saml2-sso"></a>Configuración y prueba del inicio de sesión único de Azure AD para ContractSafe Saml2 SSO

Configure y pruebe el inicio de sesión único de Azure AD con ContractSafe Saml2 SSO mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de ContractSafe Saml2 SSO.

Para configurar y probar el inicio de sesión único de Azure AD con ContractSafe Saml2 SSO, complete los siguientes bloques de creación:

1. [Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso), para permitir que los usuarios puedan utilizar esta característica.
   * [Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user), para probar el inicio de sesión único de Azure AD con la cuenta **B.Simon**.
   * [Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user), para habilitar a **B.Simon** para que use el inicio de sesión único de Azure AD.

1. [Configuración del inicio de sesión único de ContractSafe Saml2](#configure-contractsafe-saml2-sso), para configurar los valores de inicio de sesión único en la aplicación.
   * [Creación de un usuario de prueba de ContractSafe Saml2 SSO](#create-a-contractsafe-saml2-sso-test-user), para tener un homólogo de **B.Simon** en ContractSafe Saml2 SSO vinculado a la representación del usuario en Azure AD.
2. [Comprobación del inicio de sesión único](#test-sso), para verificar que la configuración funciona correctamente.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **ContractSafe Saml2 SSO**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, seleccione el icono de edición (con forma de lápiz) para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configuración del inicio de sesión único con SAML**, escriba los siguientes valores en los campos correspondientes:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente formato: `https://app.contractsafe.com/saml2_auth/<UNIQUEID>/acs/`

    b. En el cuadro de texto **URL de respuesta** , escriba una dirección URL con el siguiente formato: `https://app.contractsafe.com/saml2_auth/<UNIQUEID>/acs/`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Póngase en contacto con el [equipo de soporte técnico de clientes de ContractSafe Saml2 SSO](mailto:support@contractsafe.com) para obtener estos valores. También puede consultar los formatos que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación ContractSafe Saml2 SSO espera que las aserciones de SAML estén en un formato específico, lo cual requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![Atributos predeterminados comunes](common/default-attributes.png)

1. Además de estos atributos, la aplicación ContractSafe Saml2 SSO espera que se devuelvan algunos atributos más en la respuesta de SAML. Estos atributos se rellenan previamente, pero puede revisarlos según sus requisitos. En la lista siguiente se muestran los atributos adicionales.

    | Nombre | Atributo de origen|
    | ---------------| --------------- |
    | emailname | user.userprincipalname |
    | email | user.onpremisesuserprincipalname |

1. En la página **Configuración del inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación**. Seleccione **Descargar** para descargar el certificado y, después, guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar ContractSafe Saml2 SSO**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

## <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado **B.Simon** en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**. Seleccione **Usuarios** y, a continuación, seleccione **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba una dirección de correo electrónico con el formato `username@companydomain.extension`. Un ejemplo es `B.Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Seleccione **Crear**.

## <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a conceder a **B.Simon** acceso a ContractSafe Saml2 SSO mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **ContractSafe Saml2 SSO**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

   ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, en la lista **Usuarios** seleccione **B.Simon**. Haga clic en el botón **Seleccionar** situado en la parte inferior de la pantalla.
1. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol adecuado para el usuario en la lista. A continuación, elija el botón **Seleccionar** situado en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-contractsafe-saml2-sso"></a>Configuración de ContractSafe Saml2 SSO

Para configurar el inicio de sesión único en **ContractSafe Saml2 SSO**, debe enviar el archivo **XML de metadatos de federación** descargado y las direcciones URL apropiadas copiadas de Azure Portal al [equipo de soporte técnico de ContractSafe Saml2 SSO](mailto:support@contractsafe.com). El equipo es responsable de establecer la conexión de inicio de sesión único de SAML correctamente en ambos lados.

## <a name="create-a-contractsafe-saml2-sso-test-user"></a>Creación de un usuario de prueba de ContractSafe Saml2 SSO

Cree un usuario llamado B.Simon en ContractSafe Saml2 SSO. Trabaje con el [equipo de soporte técnico de ContractSafe Saml2 SSO](mailto:support@contractsafe.com) para agregar los usuarios en la plataforma de ContractSafe Saml2 SSO. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

Pruebe la configuración de inicio de sesión único de Azure AD con el Panel de acceso. Al seleccionar el icono de ContractSafe Saml2 SSO en el Panel de acceso, debería iniciar sesión automáticamente en la versión de ContractSafe Saml2 SSO para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales para integrar aplicaciones SaaS con Azure Active Directory ](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe ContractSafe Saml2 SSO con Azure AD](https://aad.portal.azure.com/).
