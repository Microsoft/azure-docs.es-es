---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SuccessFactors | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SuccessFactors.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 32bd8898-c2d2-4aa7-8c46-f1f5c2aa05f1
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 01/16/2020
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1de6af0f30453d40348e9e836b2a5989652f49f1
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87925131"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-successfactors"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SuccessFactors

En este tutorial, aprenderá a integrar SuccessFactors con Azure Active Directory (Azure AD). Al integrar SuccessFactors con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a SuccessFactors.
* Permitir que los usuarios inicien sesión automáticamente en SuccessFactors con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción habilitada para el inicio de sesión único (SSO) en SuccessFactors

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SuccessFactors admite el inicio de sesión único iniciado por **SP**.
* Una vez configurado SuccessFactors, puede aplicar controles de sesión, que protegen la filtración y la infiltración de la información confidencial de la organización en tiempo real. Los controles de sesión proceden del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-aad).

## <a name="adding-successfactors-from-the-gallery"></a>Incorporación de SuccessFactors desde la galería

Para configurar la integración de SuccessFactors en Azure AD, deberá agregar SuccessFactors desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **SuccessFactors** en el cuadro de búsqueda.
1. Seleccione **SuccessFactors** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-successfactors"></a>Configuración y prueba del inicio de sesión único de Azure AD para SuccessFactors

Configure y pruebe el inicio de sesión único de Azure AD con SuccessFactors mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de SuccessFactors.

Para configurar y probar el inicio de sesión único de Azure AD con SuccessFactors, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
2. **[Configuración del inicio de sesión único de SuccessFactors](#configure-successfactors-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación del usuario de prueba de SuccessFactors](#create-successfactors-test-user)** , para tener un homólogo de B.Simon en SuccessFactors vinculado a su representación en Azure AD.
3. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **SuccessFactors**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón:

    - `https://<companyname>.successfactors.com/<companyname>`
    - `https://<companyname>.sapsf.com/<companyname>`
    - `https://<companyname>.successfactors.eu/<companyname>`
    - `https://<companyname>.sapsf.eu`

    b. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón:

    - `https://www.successfactors.com/<companyname>`
    - `https://www.successfactors.com`
    - `https://<companyname>.successfactors.eu`
    - `https://www.successfactors.eu/<companyname>`
    - `https://<companyname>.sapsf.com`
    - `https://hcm4preview.sapsf.com/<companyname>`
    - `https://<companyname>.sapsf.eu`
    - `https://www.successfactors.cn`
    - `https://www.successfactors.cn/<companyname>`

    c. En el cuadro de texto **URL de respuesta** , escriba una dirección URL con el siguiente patrón:

    - `https://<companyname>.successfactors.com/<companyname>`
    - `https://<companyname>.successfactors.com`
    - `https://<companyname>.sapsf.com/<companyname>`
    - `https://<companyname>.sapsf.com`
    - `https://<companyname>.successfactors.eu/<companyname>`
    - `https://<companyname>.successfactors.eu`
    - `https://<companyname>.sapsf.eu`
    - `https://<companyname>.sapsf.eu/<companyname>`
    - `https://<companyname>.sapsf.cn`
    - `https://<companyname>.sapsf.cn/<companyname>`

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de inicio de sesión, el identificador y la dirección URL de respuesta reales. Para obtener estos valores, póngase en contacto con el [equipo de soporte técnico de SuccessFactors](https://www.sap.com/support.html).

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up SuccessFactors** (Configurar SuccessFactors), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a SuccessFactors mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **SuccessFactors**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-successfactors-sso"></a>Configuración del inicio de sesión único de SuccessFactors

1. En otra ventana del explorador web, inicie sesión en el **Portal de administración de SuccessFactors** como administrador.

2. Visite **Application Security** (Seguridad de aplicaciones) y establezca nativo en **Single Sign On Features** (Características de Inicio de sesión único).

3. Coloque cualquier valor en **Reset Token** (Restablecer Token) y haga clic en **Save Token** (Guardar Token) para habilitar SSO de SAML.

    ![Configuración del inicio de sesión único en la aplicación][11]

    > [!NOTE]
    > Este valor se utiliza como el conmutador de activado y desactivado. Si se guarda algún valor, el SSO de SAML está activado. Si se guarda un valor en blanco, el SSO de SAML está desactivado.

4. Vaya a la siguiente captura de pantalla y realice las acciones siguientes:

    ![Configuración del inicio de sesión único en la aplicación][12]
  
    a. Seleccione el botón de selección **SAML v2 SSO** (SSO de SAML v2).
  
    b. Establezca el **nombre de entidad asertivo de SAML** (por ejemplo, emisor de SAML + nombre de la empresa).

    c. En el cuadro de texto **URL del emisor**, pegue el valor del **identificador de Azure AD** que ha copiado de Azure Portal.

    d. Seleccione **Assertion** (Aserción) como **Require Mandatory Signature** (Solicitar firma obligatoria).

    e. Seleccione **Enabled** (Habilitado) como **Enable SAML Flag** (Habilitar marca SAML).

    f. Seleccione **No** como **Login Request Signature(SF Generated/SP/RP)** [Firma de solicitud de inicio de sesión (SF generado/SP/RP)].

    g. Seleccione **Browser/Post Profile** (Perfil de explorador/envío) como **SAML Profile** (Perfil SAML).

    h. Seleccione **No** como **Enforce Certificate Valid Period** (Aplicar período válido de certificado).

    i. Copie el contenido del archivo de certificado descargado de Azure Portal y péguelo en el cuadro de texto **SAML Verifying Certificate** (Certificado de verificación de firma).

    > [!NOTE] 
    > El contenido del certificado debe tener etiquetas de inicio y fin del certificado.

5. Vaya a SAML V2 y realice los pasos siguientes:

    ![Configuración del inicio de sesión único en la aplicación][13]

    a. Seleccione **Yes** (Sí) como **Support SP-initiated Global Logout** (Permitir cierre de sesión global iniciado por SP).

    b. En el cuadro de texto **Global Logout Service URL (LogoutRequest destination)** (URL del servicio de cierre de sesión global (destino LogoutRequest)), pegue el valor **Sign-Out URL** (Dirección URL de cierre de sesión) que copió de Azure Portal.

    c. Seleccione **No** en **Require sp must encrypt all NameID element** (Requerir que sp cifre todos los elementos NameID).

    d. Seleccione **Unspecified** (Sin especificar) como **NameID Format** (Formato de NameID).

    e. Seleccione **Yes** (Sí) como **Enable sp initiated login (AuthnRequest)** [Permitir inicio de sesión iniciado por sp (AuthnRequest)].

    f. En el cuadro de texto **Send request as Company-Wide issuer** (Enviar solicitud como emisor para toda la compañía), pegue el valor de **Dirección URL de inicio de sesión** que copió de Azure Portal.

6. Siga estos pasos si desea que los nombres de usuario de inicio de sesión no distingan mayúsculas de minúsculas.

    ![Configurar inicio de sesión único][29]

    a. Visite **Company Settings**(Configuración de la empresa) en la parte inferior.

    b. Seleccione la casilla junto a **Enable Non-Case-Sensitive Username**(Habilitar nombre de usuario sin distinción de mayúsculas y minúsculas).

    c. Haga clic en **Save**(Guardar).

    > [!NOTE]
    > Si intenta habilitar esta opción, el sistema comprueba si creará un nombre de inicio de sesión de SAML duplicado. Por ejemplo, si el cliente tiene nombres de usuario User1 y user1. Al no distinguir mayúsculas de minúsculas, estos nombres pasan a ser duplicados. El sistema mostrará un mensaje de error y no se habilitará la característica. El cliente deberá cambiar uno de los nombres de usuario, para que esté escrito diferente.

### <a name="create-successfactors-test-user"></a>Creación del usuario de prueba de SuccessFactors

Para permitir que los usuarios de Azure AD inicien sesión en SuccessFactors, deben aprovisionarse en SuccessFactors. En el caso de SuccessFactors, el aprovisionamiento es una tarea manual.

Para que se creen los usuarios en SuccessFactors, deberá ponerse en contacto con el [equipo de soporte técnico de SuccessFactors](https://www.sap.com/support.html).

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de SuccessFactors en el panel de acceso, debería iniciar sesión automáticamente en la versión de SuccessFactors para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe SuccessFactors con Azure AD](https://aad.portal.azure.com).

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de SuccessFactors con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

<!--Image references-->

[11]: ./media/successfactors-tutorial/tutorial_successfactors_07.png
[12]: ./media/successfactors-tutorial/tutorial_successfactors_08.png
[13]: ./media/successfactors-tutorial/tutorial_successfactors_09.png
[29]: ./media/successfactors-tutorial/tutorial_successfactors_10.png
