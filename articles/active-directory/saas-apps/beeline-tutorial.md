---
title: 'Tutorial: Integración de Azure Active Directory con Beeline | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Beeline.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 0726859d-1dac-44a0-810b-da56d89039ee
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 02/06/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: cca1b4b9f27a8711d0340389359320a2f99a918a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87018540"
---
# <a name="tutorial-azure-active-directory-integration-with-beeline"></a>Tutorial: Integración de Azure Active Directory con Beeline

En este tutorial, obtendrá información sobre cómo integrar Beeline con Azure Active Directory (Azure AD).
Integrar Beeline con Azure AD le proporciona las siguientes ventajas:

* En Azure AD, puede controlar quién tiene acceso a Beeline.
* Puede permitir que los usuarios inicien sesión automáticamente en Beeline (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para configurar la integración de Azure AD con Beeline, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en Beeline.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Beeline admite el inicio de sesión único iniciado por **IDP**.

## <a name="adding-beeline-from-the-gallery"></a>Adición de Beeline desde la galería

Para configurar la integración de Beeline en Azure AD, deberá agregar Beeline desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Beeline desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Beeline**, seleccione **Beeline** en el panel de resultados y haga clic en el botón **Agregar** para añadir la aplicación.

     ![Beeline en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Beeline con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Beeline.

Para configurar y probar el inicio de sesión único de Azure AD con Beeline, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Beeline](#configure-beeline-single-sign-on)** , para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
5. **[Creación de un usuario de prueba de Beeline](#create-beeline-test-user)** , para tener un homólogo de Britta Simon en Beeline que esté vinculado a la representación del usuario en Azure AD.
6. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Beeline, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Beeline**, haga clic en **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    ![Información sobre dominio y direcciones URL de inicio de sesión único de BeeLine](common/idp-intiated.png)

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://projects.beeline.com/<ProjInstanceName>`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón:

    ```https
    https://projects.beeline.com/<ProjInstanceName>/SSO_External.ashx
    ```

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Póngase en contacto con el [equipo de soporte técnico de Beeline](https://www.beeline.com/support-beeline/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. La aplicación Beeline espera las aserciones de SAML en un formato específico. Trabaje primero con el [equipo de soporte técnico de Beeline](https://www.beeline.com/support-beeline/) para determinar el identificador de usuario correcto que se asignará a la aplicación. Siga también las indicaciones del [equipo de soporte técnico de Beeline](https://www.beeline.com/support-beeline/) en relación con el atributo que desean usar para esta asignación. Puede administrar el valor de este atributo desde la pestaña **User Attributes** (Atributos de usuario) de la aplicación. La siguiente captura de pantalla le muestra un ejemplo de esto. Aquí hemos asignado la notificación **Id. de usuario** con el atributo **userprincipalname**, que proporciona el identificador de usuario único que se enviará a la aplicación Beeline en cada respuesta SAML correcta.

    ![imagen](common/edit-attribute.png)

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

7. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Beeline**, seleccione **Propiedades** y copie la URL de acceso de usuario.

    ![Copiar la URL de acceso de usuario](media/beeline-tutorial/client-access-url.png)


### <a name="configure-beeline-single-sign-on"></a>Configuración del inicio de sesión único de Beeline

Para configurar el inicio de sesión único en **Beeline**, tiene que enviar el **XML de metadatos de federación** descargado y la dirección URL de acceso de usuario de las propiedades de Azure Portal al [equipo de soporte técnico de Beeline](https://www.beeline.com/support-beeline/). Necesitan los metadatos y la dirección URL de acceso de usuario para que la conexión de inicio de sesión único de SAML se configure correctamente en ambos sitios.

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure mediante la concesión de acceso a Beeline.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y, por último, **Beeline**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Beeline**.

    ![El vínculo de Beeline en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-beeline-test-user"></a>Creación de un usuario de prueba de Beeline

En esta sección, creará un usuario llamado Britta Simon en Beeline. La aplicación Beeline requiere que todos los usuarios se aprovisionen en la aplicación antes de efectuar el inicio de sesión único. Trabaje con el [servicio de soporte técnico de Beeline](https://www.beeline.com/support-beeline/) para aprovisionar todos estos usuarios en la aplicación.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Beeline del Panel de acceso y debería iniciar sesión automáticamente en la versión de Beeline para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
