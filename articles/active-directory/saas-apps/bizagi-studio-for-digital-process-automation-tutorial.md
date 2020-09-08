---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Bizagi for Digital Process Automation | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Bizagi for Digital Process Automation.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/27/2020
ms.author: jeedes
ms.openlocfilehash: 5f8126f497bfd66544576b218a903c50e58106b5
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88544509"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-bizagi-for-digital-process-automation"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Bizagi for Digital Process Automation

En este tutorial, aprenderá a integrar Bizagi for Digital Process Automation Services/Server con Azure Active Directory (Azure AD). Al integrar Bizagi for Digital Process Automation con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a un proyecto de Bizagi for Digital Process Automation Services/Server.
* Permitir que los usuarios inicien sesión automáticamente en un proyecto de Bizagi for Digital Process Automation Services/Server con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Un proyecto de Bizagi con Automation Services/Server. 
* Disponer de sus propios certificados para las firmas de aserción de SAML. Estos certificados se deben generar en formato p12 o pfx.
* Disponer de un archivo de metadatos en formato XML generado a partir del proyecto de Bizagi. 

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un proyecto de Bizagi con Automation Services/Server.

* Bizagi for Digital Process Automation admite el inicio de sesión único iniciado por **SP**.
* Una vez configurado Bizagi for Digital Process Automation, puede aplicar controles de sesión, que protegen la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-bizagi-for-digital-process-automation-from-the-gallery"></a>Adición de Bizagi for Digital Process Automation desde la galería

Para configurar la integración de Bizagi for Digital Process Automation en Azure AD, es preciso agregar Bizagi for Digital Process Automation desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Bizagi for Digital Process Automation** en el cuadro de búsqueda.
1. Seleccione **Bizagi for Digital Process Automation** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-bizagi-for-digital-process-automation"></a>Configuración y prueba del inicio de sesión único de Azure AD para Bizagi for Digital Process Automation

Configure y pruebe el inicio de sesión único de Azure AD con Bizagi for Digital Process Automation mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente del proyecto de Bizagi.

Para configurar y probar el inicio de sesión único de Azure AD con Bizagi for Digital Process Automation, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Bizagi for Digital Process Automation](#configure-bizagi-for-digital-process-automation-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Bizagi for Digital Process Automation](#create-bizagi-for-digital-process-automation-test-user)** : para tener un homólogo de B.Simon en Bizagi for Digital Process Automation vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Bizagi for Digital Process Automation**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. Cargue el archivo de metadatos de Bizagi en la opción **Cargar el archivo de metadatos**.
1. Revise la configuración. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL del proyecto de Bizagi: `https://<COMPANYNAME>.bizagi.com/<PROJECTNAME>`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba la dirección URL del proyecto de Bizagi: `https://<COMPANYNAME>.bizagi.com/<PROJECTNAME>`

    > [!NOTE]
    > Estos valores no son reales. Debe actualizarlos con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de Bizagi for Digital Process Automation](mailto:jarvein.rivera@bizagi.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar la **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)
    
    Esta dirección URL de metadatos debe estar registrada en las opciones de autenticación del proyecto de Bizagi.
    
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz de **Atributos y notificaciones de usuario** para editar el identificador de usuario único.
    
    Establezca el identificador de usuario único como user.mail.

### <a name="create-an-azure-ad-test"></a>Creación de un usuario de prueba de Azure AD 

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir que B.Simon acceda a Bizagi for Digital Process Automation mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Bizagi for Digital Process Automation**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-bizagi-for-digital-process-automation-sso"></a>Configuración del inicio de sesión único de Bizagi for Digital Process Automation

Para configurar el inicio de sesión único en **Bizagi for Digital Process Automation**, debe enviar la **dirección URL de metadatos de federación de aplicación** al [equipo de soporte técnico de Bizagi for Digital Process Automation](mailto:jarvein.rivera@bizagi.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-bizagi-for-digital-process-automation-test-user"></a>Creación de un usuario de prueba de Bizagi for Digital Process Automation

En esta sección, creará un usuario llamado Britta Simon en Bizagi for Digital Process Automation. Colabore con el [equipo de soporte técnico de Bizagi for Digital Process Automation](mailto:jarvein.rivera@bizagi.com) para agregar los usuarios a la plataforma de Bizagi for Digital Process Automation. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Bizagi for Digital Process Automation en el Panel de acceso, debería iniciar sesión automáticamente en el portal de Bizagi for Digital Process Automation para el que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Bizagi for Digital Process Automation con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
