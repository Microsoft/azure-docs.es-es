---
title: 'Tutorial: Integración de Azure Active Directory con PolicyStat | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y PolicyStat.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: af5eb0f1-1c8e-4809-b0c4-8ccfb915ca42
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 03/28/2019
ms.author: jeedes
ms.openlocfilehash: 739e4fa45987f33b3a32503eaedf9aae74a7e000
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86500005"
---
# <a name="tutorial-azure-active-directory-integration-with-policystat"></a>Tutorial: Integración de Azure Active Directory con PolicyStat

En este tutorial, aprenderá a integrar PolicyStat con Azure Active Directory (Azure AD).
La integración de PolicyStat con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a PolicyStat.
* Puede permitir que los usuarios inicien sesión automáticamente en PolicyStat (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con PolicyStat, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener una versión de prueba de un mes [aquí](https://azure.microsoft.com/pricing/free-trial/)
* Una suscripción habilitada para el inicio de sesión único en PolicyStat

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* PolicyStat admite el inicio de sesión único iniciado por **SP**

* PolicyStat admite aprovisionamiento de usuarios **Just-In-Time**

## <a name="adding-policystat-from-the-gallery"></a>Incorporación de PolicyStat desde la galería

Para configurar la integración de PolicyStat en Azure AD, es preciso agregar PolicyStat desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar PolicyStat desde la galería, siga estos pasos:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **PolicyStat**, seleccione **PolicyStat** en el panel de resultados y, luego, haga clic en el botón **Agregar** para agregar la aplicación.

     ![PolicyStat en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con PolicyStat con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de PolicyStat.

Para configurar y probar el inicio de sesión único de Azure AD con PolicyStat, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de PolicyStat](#configure-policystat-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
4. **[Creación de un usuario de prueba de PolicyStat](#create-policystat-test-user)** : para tener un homólogo de Britta Simon en PolicyStat vinculado a la representación del usuario en Azure AD.
5. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con PolicyStat, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **PolicyStat**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de dominio y direcciones URL de inicio de sesión único de PolicyStat](common/sp-identifier.png)

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.policystat.com`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<companyname>.policystat.com/saml2/metadata/`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte al cliente de PolicyStat](http://www.policystat.com/support/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

5. La aplicación PolicyStat espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados. Haga clic en el icono **Editar** para abrir el cuadro de diálogo **Atributos de usuario**.

    ![imagen](common/edit-attribute.png)

6. Además de lo anterior, la aplicación PolicyStat espera que se usen algunos atributos más en la respuesta de SAML. En la sección **Notificaciones del usuario** del cuadro de diálogo **Atributos de usuario**, realice los siguientes pasos para agregar el atributo Token SAML como se muestra en la tabla siguientes:

    | Nombre | Atributo de origen |
    |------------------- | -------------------- |
    | uid | ExtractMailPrefix([mail]) |

    a. Haga clic en **Agregar nueva notificación** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.
    
    ![imagen](common/new-save-attribute.png)

    ![imagen](./media/policystat-tutorial/attribute01.png)

    b. En el cuadro de texto **Nombre**, escriba el nombre que se muestra para la fila.

    c. Deje **Espacio de nombres** en blanco.

    d. Seleccione **Transformación** como origen.

    e. En la lista **Transformación**, escriba el valor de atributo que se muestra para esa fila.
    
    f. En la lista **Parámetro 1**, escriba el valor de atributo que se muestra para esa fila.

    g. Haga clic en **Save**(Guardar).

7. En la sección **Set up PolicyStat** (Configurar PolicyStat), copie las direcciones URL adecuada según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    a. URL de inicio de sesión

    b. Identificador de Azure AD

    c. URL de cierre de sesión

### <a name="configure-policystat-single-sign-on"></a>Configuración del inicio de sesión único de PolicyStat

1. En otra ventana del explorador web, inicie sesión en el sitio de la compañía PolicyStat como administrador.

2. Haga clic en la pestaña **Administración** y en **Configuración de inicio de sesión único** en el panel de navegación izquierdo.
   
    ![Menú Administrador](./media/policystat-tutorial/ic808633.png "Menú Administrador")

3. Haga clic en **Sus metadatos de IDP** y en la sección **Sus metadatos de IDP**, realice los pasos siguientes:
   
    ![Configuración de inicio de sesión único](./media/policystat-tutorial/ic808636.png "Configuración de inicio de sesión único")
   
    a. Abra el archivo de metadatos descargado, copie el contenido y luego péguelo en el cuadro de texto **Metadatos del proveedor de identidades**.

    b. Haga clic en **Guardar cambios**.

4. Haga clic en **Configurar atributos** y, en la sección **Configurar atributos**, realice los pasos siguientes:
   
    a. En el cuadro de texto **Atributo de nombre de usuario**, escriba **uid**.

    b. En el cuadro de texto **First Name Attribute** (Atributo Nombre), escriba el nombre de atributo de notificación Nombre de Azure **`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`** .

    c. En el cuadro de texto **Last Name Attribute** (Atributo Apellidos), escriba el nombre de atributo de notificación Apellidos de Azure **`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`** .

    d. En el cuadro de texto **Email Attribute** (Atributo Correo electrónico), escriba el nombre de atributo de notificación Correo electrónico de Azure **`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`** .

    e. Haga clic en **Guardar cambios**.

5. En la sección **Configuración**, seleccione **Habilitar la integración de inicio de sesión único**.
   
    ![Configuración de inicio de sesión único](./media/policystat-tutorial/ic808634.png "Configuración de inicio de sesión único")


### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, concederá acceso a su propia cuenta a PolicyStat para que use el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones**, **PolicyStat**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **PolicyStat**.

    ![Vínculo a PolicyStat en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione su cuenta en la lista de usuarios y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-policystat-test-user"></a>Creación de un usuario de prueba de PolicyStat

En esta sección, se crea un usuario llamado Britta Simon en PolicyStat. PolicyStat admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe en PolicyStat, se crea otro después de la autenticación.

>[!NOTE]
>Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de PolicyStat ofrecida por PolicyStat para aprovisionar cuentas de usuario de Azure AD.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de PolicyStat en el panel de acceso y debería iniciar sesión automáticamente en la versión de PolicyStat para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
