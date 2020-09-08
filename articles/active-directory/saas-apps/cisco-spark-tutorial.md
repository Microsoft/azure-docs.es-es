---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Cisco Webex | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Cisco Webex.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/31/2020
ms.author: jeedes
ms.openlocfilehash: 4fd6b46353cdeb4493c1a767c92e75d602e52843
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88528907"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-cisco-webex"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Cisco Webex

En este tutorial, aprenderá a integrar Cisco Webex con Azure Active Directory (Azure AD). Al integrar Cisco Webex con Azure AD, puede hacer lo siguiente:

* Puede controlar en Azure AD quién tiene acceso a Cisco Webex.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Cisco Webex con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Cisco Webex.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Cisco Webex admite el inicio de sesión único iniciado por **SP**.
* Cisco Webex admite el aprovisionamiento de usuarios **Automatizado**.
* Una vez configurado Cisco Webex, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-aad).

## <a name="adding-cisco-webex-from-the-gallery"></a>Adición de Cisco Webex desde la galería

Para configurar la integración de Cisco Webex en Azure AD, será preciso que agregue Cisco Webex desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Cisco Webex** en el cuadro de búsqueda.
1. Seleccione **Cisco Webex** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-cisco-webex"></a>Configuración y prueba del inicio de sesión único de Azure AD para Cisco Webex

Configure y pruebe el inicio de sesión único de Azure AD con Cisco Webex utilizando un usuario de prueba llamado **B. Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Cisco Webex.

Para configurar y probar el inicio de sesión único de Azure AD con Cisco Webex, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
2. **[Configuración de Cisco Webex](#configure-cisco-webex)** , para configurar el inicio de sesión único en la aplicación.
    1. **[Creación del usuario de prueba de Cisco Webex](#create-cisco-webex-test-user)** : para tener un homólogo de B. Simon en Cisco Webex vinculado a la representación del usuario en Azure AD.
3. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Cisco Webex**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, cargue el **archivo de metadatos del proveedor de servicios** que descargó y configure la aplicación realizando estos pasos:

    >[!Note]
    >Obtendrá el archivo de metadatos del proveedor de servicios en la sección **Configure Cisco Webex** (Configuración de Cisco Webex), como se explica más adelante en el tutorial. 

    a. Haga clic en **Cargar el archivo de metadatos**.

    b. Haga clic en el **logotipo de la carpeta** para seleccionar el archivo de metadatos y luego en **Cargar**.

    c. Cuando el archivo de metadatos del proveedor de servicios se haya cargado correctamente, los valores **Identificador** y **URL de respuesta** se rellenan automáticamente en la sección **Configuración básica de SAML**:

    En el cuadro de texto **URL de inicio de sesión**, pegue el valor de **URL de respuesta** que se rellena automáticamente mediante la carga de archivo de metadatos de SP.

1. La aplicación Cisco Webex espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación Cisco Webex espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.
  
    | Nombre |  Atributo de origen|
    | ---------------|--------- |
    | uid | user.userprincipalname |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

   ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Set up Cisco Webex** (Configurar Cisco Webex), copie las direcciones URL que necesite.

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

En esta sección, va a permitir que B. Simon acceda a Cisco Webex mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Cisco Webex**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-cisco-webex"></a>Configuración de Cisco Webex

1. Para automatizar la configuración en Cisco Webex, debe instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Set up Cisco Webex** (Configurar Cisco Webex) para ir a la aplicación del mismo nombre. En ella, escriba las credenciales de administrador para iniciar sesión en Cisco Webex. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 8.

    ![Configuración](common/setup-sso.png)

3. Si desea configurar Cisco Webex de forma manual, inicie sesión en [Cisco Cloud Collaboration Management](https://admin.ciscospark.com/) (Administración de colaboración en la nube de Cisco) con sus credenciales completas de administrador.

4. Seleccione **Settings** (Configuración) y en la sección **Authentication** (Autenticación), haga clic en **Modify** (Modificar).

    ![Configurar inicio de sesión único](./media/cisco-spark-tutorial/tutorial-cisco-spark-10.png)
  
5. Seleccione **Integrate a 3rd-party identity provider. (Advanced)** (Integrar un proveedor de identidades de terceros [avanzado]).y vaya a la pantalla siguiente.

6. En la página **Import Idp Metadata** (Importar metadatos de LDP) arrastre y coloque el archivo de metadatos de Azure AD en la página o utilice la opción de explorador de archivos para buscar y cargar el archivo de metadatos de Azure AD. Después, seleccione **Require certificate signed by a certificate authority in Metadata (more secure)** (Requerir certificado firmado por una entidad de certificación en metadatos [más seguro]) y haga clic en **Next** (Siguiente).

    ![Configurar inicio de sesión único](./media/cisco-spark-tutorial/tutorial-cisco-spark-11.png)

7. Seleccione **Test SSO Connection** (Probar SSO de conexión) y cuando se abra una nueva pestaña de explorador, autentíquese con Azure AD mediante el inicio de sesión.

8. Vuelva a la pestaña del explorador de **Cisco Cloud Collaboration Management**. Si la prueba se realizó correctamente, seleccione **This test was successful. Enable Single Sign-On option** (Esta prueba se realizó correctamente. Habilite la opción de inicio de sesión único) y haga clic en **Next** (Siguiente).

### <a name="create-cisco-webex-test-user"></a>Creación del usuario de prueba de Cisco Webex

En esta sección, creará una usuaria llamada B. Simon en Cisco Webex. En esta sección, creará una usuaria llamada B. Simon en Cisco Webex.

1. Vaya a [Cisco Cloud Collaboration Management](https://admin.ciscospark.com/) con sus credenciales de administración completas.

2. Haga clic en **Usuarios** y después en **Administrar usuarios**.
   
    ![Configurar inicio de sesión único](./media/cisco-spark-tutorial/tutorial-cisco-spark-12.png) 

3. En la ventana **Manage User** (Administrar usuario), seleccione **Manually add or modify users** (Agregar o modificar usuarios manualmente) y haga clic en **Next** (Siguiente).

4. Seleccione **Names and Email address** (Nombres y direcciones de correo electrónico). Después, rellene el cuadro de texto de la forma siguiente:

    ![Configurar inicio de sesión único](./media/cisco-spark-tutorial/tutorial-cisco-spark-13.png) 

    a. En el cuadro de texto **First Name** (Nombre), escriba el nombre del usuario, en este caso, **B**.

    b. En el cuadro de texto **Last Name** (Apellidos), escriba los apellidos del usuario, en este caso, **Simon**.

    c. En el cuadro de texto **Email address** (Dirección de correo electrónico), escriba la dirección de correo electrónico del usuario, por ejemplo, b.simon@contoso.com.

5. Haga clic en el signo más para agregar a B. Simon. A continuación, haga clic en **Siguiente**.

6. En la ventana **Add Services for Users** (Agregar servicios para los usuarios), haga clic en **Save** (Guardar) y después en **Finish** (Finalizar).

## <a name="test-sso"></a>Prueba de SSO

Al seleccionar el icono de Cisco Webex en el panel de acceso, debería iniciar sesión automáticamente en la versión de Cisco Webex para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de Cisco Webex con Azure AD](https://aad.portal.azure.com)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Cisco Webex con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/protect-webex)