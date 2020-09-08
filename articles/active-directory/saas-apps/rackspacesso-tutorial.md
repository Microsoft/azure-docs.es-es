---
title: 'Tutorial: integración de Azure Active Directory con Rackspace SSO | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Rackspace SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/15/2019
ms.author: jeedes
ms.openlocfilehash: a2c2e7358c32453daf53c40a9322df4fe30642d8
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88548908"
---
# <a name="tutorial-azure-active-directory-integration-with-rackspace-sso"></a>Tutorial: integración de Azure Active Directory con Rackspace SSO

En este tutorial aprenderá a integrar Rackspace SSO con Azure Active Directory (Azure AD).
La integración de Rackspace SSO con Azure AD le proporciona las siguientes ventajas:

* Puede controlar en Azure AD quién tiene acceso a Rackspace SSO.
* Puede permitir que los usuarios inicien sesión automáticamente en Rackspace (inicio de sesión único) con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central: Azure Portal.

Si desea obtener más información sobre la integración de aplicaciones SaaS con Azure AD, vea [Qué es el acceso a las aplicaciones y el inicio de sesión único en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Rackspace SSO, necesita lo siguiente:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/)
* Una suscripción habilitada para inicio de sesión único en Rackspace SSO

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Rackspace SSO admite el inicio de sesión único iniciado por **SP**

## <a name="adding-rackspace-sso-from-the-gallery"></a>Adición de Rackspace SSO desde la galería

Para configurar la integración de Rackspace SSO en Azure AD, tiene que agregar Rackspace SSO desde la galería a la lista de aplicaciones SaaS administradas.

**Para agregar Rackspace SSO desde la galería, realice los pasos siguientes:**

1. En el panel de navegación izquierdo de **[Azure Portal](https://portal.azure.com)** , haga clic en el icono de **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione la opción **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, haga clic en el botón **Nueva aplicación** de la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Rackspace SSO**, seleccione **Rackspace SSO** en el panel de resultados y, a continuación, haga clic en el botón **Agregar** para agregar la aplicación.

     ![Rackspace SSO en la lista de resultados](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configuración y prueba del inicio de sesión único en Azure AD

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Rackspace SSO con un usuario de prueba llamado **Britta Simon**.
Si se usa el inicio de sesión único con Rackspace, los usuarios de este se crearán automáticamente la primera vez que inicien sesión en el portal de Rackspace. 

Para configurar y probar el inicio de sesión único de Azure AD con Rackspace SSO, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on)** : para que los usuarios puedan usar esta característica.
2. **[Configuración del inicio de sesión único de Rackspace SSO](#configure-rackspace-sso-single-sign-on)** : para configurar los valores de Inicio de sesión único en la aplicación.
3. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
1. **[Configuración de la asignación de atributos en el panel de control de Rackspace](#set-up-attribute-mapping-in-the-rackspace-control-panel)** : para asignar roles de Rackspace a usuarios de Azure AD.
1. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Rackspace SSO, realice los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Rackspace SSO**, seleccione **Inicio de sesión único**.

    ![Vínculo Configurar inicio de sesión único](common/select-sso.png)

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

    ![Modo de selección de inicio de sesión único](common/select-saml-option.png)

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, cargue el **archivo de metadatos del proveedor de servicios**, que se puede descargar de la [dirección URL](https://login.rackspace.com/federate/sp.xml), y siga estos pasos:

    a. Haga clic en **Cargar el archivo de metadatos**.

    ![imagen](common/upload-metadata.png)

    b. Haga clic en el **logotipo de la carpeta** para seleccionar el archivo de metadatos y luego en **Cargar**.

    ![imagen](common/browse-upload-metadata.png)

    c. Una vez que se ha cargado correctamente el archivo de metadatos, las direcciones URL necesarias se rellenan automáticamente.

    d. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL: `https://login.rackspace.com/federate/`

    ![Información de dominio y direcciones URL de inicio de sesión único de Rackspace SSO](common/sp-signonurl.png)   

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

Este archivo se cargará en Rackspace para llenar los valores de configuración de Identity Federation requeridos.

### <a name="configure-rackspace-sso-single-sign-on"></a>Configuración del inicio de sesión único de Rackspace SSO

Para configurar el inicio de sesión único en **Rackspace SSO**:

1. Consulte la documentación de [Add an Identity Provider to the Control Panel](https://developer.rackspace.com/docs/rackspace-federation/gettingstarted/add-idp-cp/) (Agregar un proveedor de identidades al panel de control)
1. En ella se proporcionan los pasos necesarios para:
    1. Crear un proveedor de identidades
    1. Especificar un dominio de correo electrónico que los usuarios utilizarán para identificar la compañía al iniciar sesión.
    1. Cargar el **XML de metadatos de federación**  que ha descargado previamente desde el panel de control de Azure.

Así se configurarán los valores básicos de SSO necesarios para la conexión entre Azure y Rackspace.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

El objetivo de esta sección es crear un usuario de prueba en Azure Portal llamado "Britta Simon".

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

    ![Vínculos "Usuarios y grupos" y "Todos los usuarios"](common/users.png)

2. Seleccione **Nuevo usuario** en la parte superior de la pantalla.

    ![Botón Nuevo usuario](common/new-user.png)

3. En las propiedades Usuario, siga estos pasos.

    ![Cuadro de diálogo Usuario](common/user-properties.png)

    a. En el campo **Nombre**, escriba **BrittaSimon**.
  
    b. En el campo **Nombre de usuario**, escriba `brittasimon@yourcompanydomain.extension`. Por ejemplo: BrittaSimon@contoso.com

    c. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro Contraseña.

    d. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a Rackspace SSO.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y **Rackspace SSO**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Rackspace SSO**.

    ![Vínculo a Rackspace SSO en la lista de aplicaciones](common/all-applications.png)

3. En el menú de la izquierda, seleccione **Usuarios y grupos**.

    ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Panel Agregar asignación](common/add-assign-user.png)

5. En el cuadro de diálogo **Usuarios y grupos**, seleccione **Britta Simon** en la lista Usuarios y, luego, haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

6. Si espera cualquier valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol** seleccione en la lista el rol adecuado para el usuario y, después, haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.

7. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="set-up-attribute-mapping-in-the-rackspace-control-panel"></a>Configuración de la asignación de atributos en el panel de control de Rackspace

Rackspace usa una **directiva de asignación de atributos** para asignar roles y grupos de Rackspace a los usuarios de inicio de sesión único. La **directiva de asignación de atributos** convierte las notificaciones SAML de Azure AD en los campos de la configuración de usuario que Rackspace requiere. Se puede encontrar más información en la [documentación de los conceptos básicos de la asignación de atributos](https://developer.rackspace.com/docs/rackspace-federation/appendix/map/) de Rackspace. Algunas consideraciones:

* Si desea asignar distintos niveles de acceso a Rackspace mediante grupos de Azure AD, deberá habilitar la notificación de grupos en la configuración del inicio de sesión único de **Rackspace SSO** en Azure. La **directiva de asignación de atributos** se usará para que coincidan esos grupos con los roles y grupos de Rackspace deseados:

    ![Configuración de la notificación de grupos](common/sso-groups-claim.png)

* De forma predeterminada, Azure AD envía el identificador de usuario a los grupos de Azure AD en la notificación SAML, en lugar del nombre del grupo. Sin embargo, si se va a sincronizar la instancia local de Active Directory con Azure AD, tiene la opción de enviar los nombres reales de los grupos:

    ![Configuración del nombre de la notificación de grupos](common/sso-groups-claims-names.png)

En el siguiente ejemplo de **directiva de asignación de atributo** se muestra:
1. El establecimiento del nombre de usuario de Rackspace en la notificación SAML `user.name`. Se puede usar cualquier notificación, pero es más común seleccionar un campo que contenga la dirección de correo electrónico del usuario.
1. El establecimiento de los roles `admin` y `billing:admin` de Rackspace en un usuario, para lo que se hace coincidir un grupo de Azure AD con un nombre de grupo o un identificador de usuario de grupo. Se usa una *sustitución*  de `"{0}"` en el campo `roles` y se reemplazará por los resultados de la expresiones de reglas `remote`.
1. El uso de la *sustitución predeterminada*`"{D}"` para que Rackspace pueda recuperar campos de SAML adicionales mediante la búsqueda de notificaciones de SAML estándar y conocidas en el intercambio de SAML.

```yaml
---
mapping:
    rules:
    - local:
        user:
          domain: "{D}"
          name: "{At(http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name)}"
          email: "{D}"
          roles:
              - "{0}"
          expire: "{D}"
      remote:
          - path: |
              (
                if (mapping:get-attributes('http://schemas.microsoft.com/ws/2008/06/identity/claims/groups')='7269f9a2-aabb-9393-8e6d-282e0f945985') then ('admin', 'billing:admin') else (),
                if (mapping:get-attributes('http://schemas.microsoft.com/ws/2008/06/identity/claims/groups')='MyAzureGroup') then ('admin', 'billing:admin') else ()
              )
            multiValue: true
  version: RAX-1
```
> [!TIP]
> Asegúrese de que utiliza un editor de texto que valide la sintaxis de YAML al editar el archivo de directiva.

Para ver más ejemplos, consulte la [documentación de los documentación de los conceptos básicos de la asignación de atributos](https://developer.rackspace.com/docs/rackspace-federation/appendix/map/) de Rackspace.

### <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Rackspace SSO en el panel de acceso, debería iniciar sesión automáticamente en la versión de Rackspace SSO para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

También puede usar el botón **Validate** (Validar) de la configuración del inicio de sesión único de **Rackspace SSO**:

   ![Botón Validate (Validar) de SSO](common/sso-validate-sign-on.png)

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

