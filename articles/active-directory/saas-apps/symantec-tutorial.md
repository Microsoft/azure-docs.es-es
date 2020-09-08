---
title: 'Tutorial: Integración de Azure Active Directory con Symantec Web Security Service (WSS) | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Symantec Web Security Service (WSS).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/25/2018
ms.author: jeedes
ms.openlocfilehash: 7e2087fdd5568156a96645b25cb105cc5f3ca6b7
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88544951"
---
# <a name="tutorial-azure-active-directory-integration-with-symantec-web-security-service-wss"></a>Tutorial: Integración de Azure Active Directory con Symantec Web Security Service (WSS)

En este tutorial aprenderá cómo integrar su cuenta de Symantec Web Security Service (WSS) con su cuenta de Azure Active Directory (Azure AD) para que WSS pueda autenticar un usuario final que se aprovisiona en Azure AD mediante la autenticación de SAML y aplicar reglas de directiva de nivel de usuario o grupo.

La integración de Symantec Web Security Service (WSS) con Azure AD proporciona las siguientes ventajas:

- Administrar todos los usuarios finales y los grupos que usan la cuenta de WSS desde el portal de Azure AD.

- Permitir que los usuarios finales se autentiquen en WSS con sus credenciales de Azure AD.

- Habilitar la aplicación de reglas de directiva de nivel de grupo y usuario definidas en la cuenta de WSS.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Symantec Web Security Service (WSS), necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en Symantec Web Security Service (WSS)

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Symantec Web Security Service (WSS) admite inicio de sesión único iniciado por **IDP**.

## <a name="adding-symantec-web-security-service-wss-from-the-gallery"></a>Agregación de Symantec Web Security Service (WSS) desde la galería

Para configurar la integración de Symantec Web Security Service (WSS) en Azure AD, es preciso agregar Symantec Web Security Service (WSS) desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Symantec Web Security Service (WSS) desde la galería, siga estos pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Symantec Web Security Service (WSS)** , seleccione **Symantec Web Security Service (WSS)** en el panel de resultados y, a continuación, haga clic en el botón **Agregar** para agregar la aplicación.

     ![Symantec Web Security Service (WSS) en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Symantec Web Security Service (WSS) con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Symantec Web Security Service (WSS).

Para configurar y probar el inicio de sesión único de Azure AD con Symantec Web Security Service (WSS), es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **Configuración del inicio de sesión único de Symantec Web Security Service (WSS)** : para configurar el inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Symantec Web Security Service (WSS)](#create-symantec-web-security-service-wss-test-user)** : para tener un homólogo de Britta Simon en Symantec Web Security Service (WSS) que esté vinculado a la representación de ella en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Symantec Web Security Service (WSS), siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Symantec Web Security Service (WSS)** , seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En el cuadro de diálogo **Configuración básica de SAML**, realice los siguientes pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de Symantec Web Security Service (WSS)](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL: `https://saml.threatpulse.net:8443/saml/saml_realm`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL: `https://saml.threatpulse.net:8443/saml/saml_realm/bcsamlpost`

    > [!NOTE]
    > Póngase en contacto con el [equipo de soporte técnico de Symantec Web Security Service (WSS)](https://www.symantec.com/contact-us) si los valores para **Identificador** y **URL de respuesta** no funcionasen por algún motivo. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

### <a name="configure-symantec-web-security-service-wss-single-sign-on"></a>Configuración del inicio de sesión único de Symantec Web Security Service (WSS)

Para configurar el inicio de sesión único en Symantec Web Security Service (WSS), consulte la documentación en línea de WSS. El archivo **XML de metadatos de federación** descargado se debe importar en el portal de WSS. Póngase en contacto con el [equipo de soporte técnico de Symantec Web Security Service (WSS)](https://www.symantec.com/contact-us) si necesita ayuda con la configuración en el portal de WSS.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD 

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba **brittasimon\@yourcompanydomain.extension**.  
    Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Symantec Web Security Service (WSS).

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y, luego, **Symantec Web Security Service (WSS)** .

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, escriba y seleccione **Symantec Web Security Service (WSS)** .

    ![Vínculo a Symantec Web Security Service (WSS) en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-symantec-web-security-service-wss-test-user"></a>Creación de un usuario de prueba de Symantec Web Security Service (WSS)

En esta sección, creará un usuario llamado Britta Simon en Symantec Web Security Service (WSS). El nombre de usuario final correspondiente se puede crear manualmente en el portal de WSS o puede esperar a que los usuarios y grupos que se han aprovisionado en Azure AD se sincronicen con el portal de WSS transcurridos unos minutos (unos 15 minutos). Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único. La dirección IP pública del equipo del usuario final, que se utilizará para examinar sitios web, también debe aprovisionarse en el portal de Symantec Web Security Service (WSS).

> [!NOTE]
> Haga clic [aquí](https://www.bing.com/search?q=my+ip+address&qs=AS&pq=my+ip+a&sc=8-7&cvid=29A720C95C78488CA3F9A6BA0B3F98C5&FORM=QBLH&sp=1) para obtener la dirección IP pública de su máquina.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

Ahora que ha configurado su cuenta de WSS para usar Azure AD para la autenticación de SAML, en esta sección, probará la funcionalidad de inicio de sesión único.

Después de haber configurado el explorador web para canalizar el tráfico hacia WSS, cuando se abra el explorador web y se intente ir a un sitio, se le redirigirá a la página de inicio de sesión de Azure. Escriba las credenciales del usuario final de prueba que se ha aprovisionado en Azure AD (es decir, BrittaSimon) y la contraseña asociada. Una vez autenticado, podrá visitar el sitio web que eligió. Debe crear una regla de directiva en WSS para bloquear la exploración de un sitio determinado a BrittaSimon y, a continuación, debería ver la página de bloqueo de WSS al intentar ir a ese sitio como el usuario BrittaSimon.

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

