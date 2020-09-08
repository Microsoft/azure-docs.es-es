---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Moxtra | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Moxtra.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/05/2019
ms.author: jeedes
ms.openlocfilehash: 7fffe58dfdb63da28edc19c19b56b576f4fbadd3
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88544050"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-moxtra"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Moxtra

En este tutorial, aprenderá a integrar Moxtra con Azure Active Directory (Azure AD). Al integrar Moxtra con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Moxtra.
* Permitir que los usuarios inicien sesión automáticamente en Moxtra con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Moxtra.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Moxtra admite el inicio de sesión único iniciado por **SP**

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-moxtra-from-the-gallery"></a>Incorporación de Moxtra desde la galería

Para configurar la integración de Moxtra en Azure AD, es preciso agregar Moxtra desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Moxtra** en el cuadro de búsqueda.
1. Seleccione **Moxtra** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-single-sign-on-for-moxtra"></a>Configuración y prueba del inicio de sesión único de Azure AD para Moxtra

Configure y pruebe el inicio de sesión único de Azure AD con Moxtra mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Moxtra.

Para configurar y probar el inicio de sesión único de Azure AD con Moxtra, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Moxtra](#configure-moxtra-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Moxtra](#create-moxtra-test-user)** , para tener un homólogo de B.Simon en Moxtra que esté vinculado a la representación de este usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Moxtra**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL: `https://www.moxtra.com/service/#login`

1. La aplicación Moxtra espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados. Haga clic en el icono **Editar** para abrir el cuadro de diálogo Atributos de usuario.

    ![imagen](common/edit-attribute.png)

1. Además de lo anterior, la aplicación Moxtra espera que se usen algunos atributos más en la respuesta de SAML. En la sección Notificaciones del usuario del cuadro de diálogo Atributos de usuario, haga lo siguiente para agregar el atributo Token SAML como se muestra en la tabla siguiente:

    | Nombre | Atributo de origen|
    | ------------------- | -------------------- |    
    | firstname | user.givenname |
    | lastname | user.surname |
    | idpid    | <Identificador de Azure AD>

    > [!Note]
    > El valor del atributo **idpid** no es real. Puede obtener el valor real de la sección **Set up Moxtra** (Configurar Moxtra) del paso 8. 

    1. Haga clic en **Agregar nueva notificación** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

    1. En el cuadro de texto **Nombre**, escriba el nombre que se muestra para la fila.

    1. Deje **Espacio de nombres** en blanco.

    1. Seleccione **Atributo** como origen.

    1. En la lista **Atributo de origen**, escriba el valor de atributo que se muestra para esa fila.

    1. Haga clic en **Aceptar**.

    1. Haga clic en **Save**(Guardar).

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Moxtra**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Moxtra mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Moxtra**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-moxtra-sso"></a>Configuración del inicio de sesión único en Moxtra

1. En otra ventana del explorador, inicie sesión en su sitio de la compañía de Moxtra como administrador.

2. En la barra de herramientas de la izquierda, haga clic en **Admin Console > SAML Single Sign-on** (Consola de administración > Inicio de sesión único de SAML) y, luego, en **New** (Nuevo).
   
    ![Configurar inicio de sesión único](./media/moxtra-tutorial/tutorial_moxtra_06.png) 

3. En la página **SAML** , realice los siguientes pasos:
   
    ![Configurar inicio de sesión único](./media/moxtra-tutorial/tutorial_moxtra_08.png)   
 
    a. En el cuadro de texto **Name** (Nombre), escriba el nombre de la configuración (por ejemplo, *SAML*). 
  
    b. En el cuadro de texto **IdP Entity ID** (Identificador de la entidad de IdP), pegue el valor de **Identificador de Azure AD** que copió de Azure Portal. 
 
    c. En el cuadro de texto **Dirección URL de inicio de sesión**, pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal. 
 
    d. En el cuadro de texto **AuthnContextClassRef**, escriba **urn: oasis: nombres: tc: SAML:2.0:ac:classes:Password**. 
 
    e. En el cuadro de texto **NameID format** (Formato de id. de nombre), escriba **urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress**. 
 
    f. Abra el certificado que ha descargado desde Azure Portal en el Bloc de notas, copie el contenido y luego péguelo en el cuadro de texto **Certificate** (Certificado).    
 
    g. En el cuadro de texto del dominio de correo electrónico SAML, escriba su dominio de correo electrónico SAML.    
  
    >[!NOTE]
    >Para ver los pasos para comprobar el dominio, haga clic en la "**i**" a continuación.

    h. Haga clic en **Update**(Actualizar).

### <a name="create-moxtra-test-user"></a>Creación de un usuario de prueba de Moxtra

El objetivo de esta sección es crear un usuario de prueba llamado B.Simon en Moxtra.

**Para crear un usuario llamado B.S en Moxtra, realice los pasos siguientes:**

1. Inicie sesión en su sitio de la compañía de Moxtra como administrador.

1. En la barra de herramientas de la izquierda, haga clic en **Consola de administración > Administración de usuarios** y, luego, en **Agregar usuario**.
   
    ![Configurar inicio de sesión único](./media/moxtra-tutorial/tutorial_moxtra_10.png) 

1. En el cuadro de diálogo **Agregar usuario** , realice los pasos siguientes:
  
    a. En el cuadro de texto **Name** (Nombre), escriba **B**.
  
    b. En el cuadro de texto **Apellidos**, escriba **Simon**.
  
    c. En el cuadro de texto **Email** (Correo electrónico), escriba la misma dirección de correo electrónico de B.Simon que en Azure Portal.
  
    d. En el cuadro de texto **Division** (División), escriba **Dev**.
  
    e. En el cuadro de texto **Department** (Departamento), escriba **IT**.
  
    f. Seleccione **Administrator** (Administrador).
  
    g. Haga clic en **Agregar**.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Moxtra en el panel de acceso, debería iniciar sesión automáticamente en la versión de Moxtra para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de Moxtra con Azure AD](https://aad.portal.azure.com/)

