---
title: 'Tutorial: Integración de Azure Active Directory con SharePoint local | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SharePoint local.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/15/2020
ms.author: miguego
ms.openlocfilehash: 996668751ee93d14b18e399035fd345f32c58fbe
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88552180"
---
# <a name="tutorial-azure-active-directory-single-sign-on-integration-with-sharepoint-on-premises"></a>Tutorial: Integración del inicio de sesión único de Azure Active Directory con SharePoint local

En este tutorial se aprende a integrar SharePoint local con Azure Active Directory (Azure AD). Al integrar SharePoint local con Azure AD, puede hacer lo siguiente:

* Controlar quién tiene acceso a SharePoint local en Azure AD.
* Permitir que los usuarios inicien sesión automáticamente en SharePoint local con sus cuentas de Azure AD.
* Administrar las cuentas en Azure Portal.

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con SharePoint local, necesita estos elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una granja de servidores de SharePoint 2013, o posterior.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único (SSO) de Azure AD en un entorno de prueba. Los usuarios de Azure AD podrán acceder a su instancia de SharePoint local.

## <a name="create-enterprise-applications-in-the-azure-portal"></a>Creación de las aplicaciones empresariales en Azure Portal

Para configurar la integración de SharePoint local en Azure AD, tiene que agregar SharePoint local desde la galería a la lista de aplicaciones SaaS administradas.

Para agregar SharePoint local desde la galería:

1. En [Azure Portal](https://portal.azure.com), en el panel izquierdo, seleccione **Azure Active Directory**.

   > [!NOTE]
   > Si el elemento no está disponible, también puede abrirlo mediante el vínculo **Todos los servicios** en la parte superior del panel izquierdo. En la información general siguiente, el vínculo **Azure Active Directory** está en la sección **Identidad**. También puede buscarlo mediante el cuadro de filtro.

1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

1. Para agregar una aplicación nueva, en la parte superior del cuadro de diálogo, seleccione **Nueva aplicación**.

1. En el cuadro de búsqueda, escriba **SharePoint local**. Seleccione **SharePoint local** en el panel de resultados.

    <kbd>![SharePoint local en la lista de resultados](./media/sharepoint-on-premises-tutorial/search-new-app.png)</kbd>

1. Especifique un nombre para la instancia de SharePoint local y seleccione **Agregar** para agregar la aplicación.

1. En la nueva aplicación empresarial, seleccione **Propiedades** y compruebe el valor de **¿Asignación de usuarios?** .

   <kbd>![Conmutador ¿Asignación de usuarios?](./media/sharepoint-on-premises-tutorial/user-assignment-required.png)</kbd>

   En este escenario, el valor se establece en **No**.

## <a name="configure-and-test-azure-ad"></a>Configuración y prueba del inicio de sesión único de Azure AD

En esta sección, va a configurar el inicio de sesión único de Azure AD con SharePoint local. Para que el inicio de sesión único funcione, se debe establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de SharePoint local.

Para configurar y probar el inicio de sesión único de Azure AD con SharePoint local, complete estos bloques de creación:

- [Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-single-sign-on), para permitir que los usuarios utilicen esta característica.
- [Configuración de SharePoint local](#configure-sharepoint-on-premises), para configurar los valores de inicio de sesión único en la aplicación.
- [Creación de un usuario de prueba de Azure AD en Azure Portal](#create-an-azure-ad-test-user-in-the-azure-portal), para crear un nuevo usuario en Azure AD para el inicio de sesión único.
- [Creación de un grupo de seguridad de Azure AD en Azure Portal](#create-an-azure-ad-security-group-in-the-azure-portal), para crear un nuevo grupo de seguridad en Azure AD para el inicio de sesión único.
- [Concesión de permisos a la cuenta de Azure AD en SharePoint local](#grant-permissions-to-an-azure-ad-account-in-sharepoint-on-premises), para conceder permisos al usuario de Azure AD.
- [Concesión de permisos al grupo de Azure AD en SharePoint local](#grant-permissions-to-an-azure-ad-group-in-sharepoint-on-premises), para conceder permisos al grupo de Azure AD.
- [Concesión de acceso a una cuenta de invitado a SharePoint local en Azure Portal](#grant-access-to-a-guest-account-to-sharepoint-on-premises-in-the-azure-portal), para conceder permisos a la cuenta de invitado en Azure AD para SharePoint local.
- [Configuración del proveedor de identidades de confianza para varias aplicaciones web](#configure-the-trusted-identity-provider-for-multiple-web-applications), para usar el mismo proveedor de identidades de confianza para varias aplicaciones web.

### <a name="configure-azure-ad-single-sign-on"></a>Configuración del inicio de sesión único de Azure AD

En esta sección va a habilitar el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con SharePoint local:

1. En [Azure Portal](https://portal.azure.com/), seleccione **Azure Active Directory** > **Aplicaciones empresariales**. Seleccione el nombre de la aplicación empresarial creada anteriormente y seleccione **Inicio de sesión único**.

1. En el cuadro de diálogo **Seleccione un método de inicio de sesión único**, seleccione el modo **SAML** para habilitar el inicio de sesión único.
 
1. En la página **Set-up Single Sign-On with SAML** (Configurar inicio de sesión único con SAML), seleccione el icono **Edit** (Editar) para abrir el cuadro de diálogo **Basic SAML Configuration** (Configuración básica de SAML).

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    ![Información de inicio de sesión único de dominio y direcciones URL de SharePoint local](./media/sharepoint-on-premises-tutorial/sp-identifier-reply.png)

    1. En el cuadro **Identificador**, escriba una dirección URL con este patrón: `urn:<sharepointFarmName>:<federationName>`.

    1. En el cuadro **Dirección URL de respuesta**, escriba una dirección URL con este patrón: `https://<YourSharePointSiteURL>/_trust/`.

    1. En el cuadro **URL de inicio de sesión**, escriba una dirección URL con este patrón: `https://<YourSharePointSiteURL>/`.
    1. Seleccione **Guardar**.

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de inicio de sesión, el identificador y la dirección URL de respuesta reales.

1. En la página **Configuración del inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione **Descargar** para descargar el **Certificado (Base 64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![El vínculo de descarga del certificado](./media/sharepoint-on-premises-tutorial/certificatebase64.png)

1. En la sección **Configurar SharePoint local**, copie las direcciones URL adecuadas según sus necesidades:
    
    - **Dirección URL de inicio de sesión**
    
        Copie la dirección URL de inicio de sesión y reemplace la parte **/saml2** del final por **/wsfed** para que tenga el siguiente aspecto: https://login.microsoftonline.com/2c4f1a9f-be5f-10ee-327d-a95dac567e4f/wsfed. (Esta dirección URL no es exacta).

    - **Identificador de Azure AD**
    - **Dirección URL de cierre de sesión**

    > [!NOTE]
    > Esta dirección URL no se puede usar tal cual en SharePoint. Debe reemplazar **/saml2** por **/wsfed**. La aplicación SharePoint local usa un token SAML 1.1, por lo que Azure AD espera una solicitud de WS Fed del servidor de SharePoint. Después de la autenticación, emite el token SAML 1.1.

### <a name="configure-sharepoint-on-premises"></a>Configuración de SharePoint local

1. Cree un nuevo proveedor de identidades de confianza en SharePoint Server 2016.

    Inicie sesión en el servidor de SharePoint y abra el shell de administración de SharePoint. Rellene los valores:
    - **$realm** es el valor del identificador de la sección Dominio y direcciones URL de SharePoint local en Azure Portal.
    - **$wsfedurl** es la dirección URL del servicio de inicio de sesión único.
    - **$filepath** es la ruta de acceso en la que ha descargado el archivo de certificado desde Azure Portal.

    Ejecute los siguientes comandos para configurar un nuevo proveedor de identidades de confianza.

    > [!TIP]
    > Si no está familiarizado con PowerShell o quiere obtener más información sobre el funcionamiento de PowerShell, vea [SharePoint PowerShell](https://docs.microsoft.com/powershell/sharepoint/overview?view=sharepoint-ps).


    ```
    $realm = "urn:sharepoint:sps201x"
    $wsfedurl="https://login.microsoftonline.com/2c4f1a9f-be5f-10ee-327d-a95dac567e4f/wsfed"
    $filepath="C:\temp\SharePoint 2019 OnPrem.cer"
    $cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($filepath)
    New-SPTrustedRootAuthority -Name "AzureAD" -Certificate $cert
    $map1 = New-SPClaimTypeMapping -IncomingClaimType "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name" -IncomingClaimTypeDisplayName "name" -LocalClaimType "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn"
    $map2 = New-SPClaimTypeMapping -IncomingClaimType "http://schemas.microsoft.com/ws/2008/06/identity/claims/role" -IncomingClaimTypeDisplayName "Role" -SameAsIncoming
    $ap = New-SPTrustedIdentityTokenIssuer -Name "AzureAD" -Description "Azure AD SharePoint server 201x" -realm $realm -ImportTrustCertificate $cert -ClaimsMappings $map1,$map2 -SignInUrl $wsfedurl -IdentifierClaim $map1.InputClaimType
    ```
1. Habilite el proveedor de identidades de confianza para la aplicación.

    1. En **Administración central**, vaya a **Administrar aplicación web** y seleccione la aplicación web que quiere proteger con Azure AD.

    1. En la cinta, seleccione **Proveedores de autenticación** y elija la zona que quiere usar.

    1. Seleccione **Proveedor de identidades de confianza** y luego seleccione el proveedor de identidades que acaba de registrar, llamado *AzureAD*.

    1. Seleccione **Aceptar**.

    ![Configuración del proveedor de autenticación](./media/sharepoint-on-premises-tutorial/config-auth-provider.png)

### <a name="create-an-azure-ad-test-user-in-the-azure-portal"></a>Creación de un usuario de prueba de Azure AD en Azure Portal

El objetivo de esta sección es crear un usuario de prueba en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**. En el panel **Administrar**, seleccione **Usuarios**.

1. Seleccione **Todos los usuarios** > **Nuevo usuario** en la parte superior de la pantalla.

1. Seleccione **Crear usuario** y, en las propiedades del usuario, siga estos pasos. Debería poder crear usuarios en Azure AD mediante el uso del sufijo de inquilino o cualquier dominio verificado. 

    1. En el cuadro **Nombre**, escriba el nombre del usuario. Vamos a utilizar **TestUser**.
  
    1. En el cuadro **Nombre de usuario**, escriba `TestUser@yourcompanydomain.extension`. En este ejemplo se muestra `TestUser@contoso.com`.

       ![Cuadro de diálogo Usuario](./media/sharepoint-on-premises-tutorial/user-properties.png)

    1. Seleccione la casilla **Mostrar contraseña** y, después, anote el valor que aparece en el cuadro **Contraseña**.

    1. Seleccione **Crear**.

    1. Ahora puede compartir el sitio con TestUser@contoso.com y permitir que este usuario tenga acceso a él.

### <a name="create-an-azure-ad-security-group-in-the-azure-portal"></a>Creación de un grupo de seguridad de Azure AD en Azure Portal

1. Seleccione **Azure Active Directory** > **Grupos**.

1. Seleccione **Nuevo grupo**.

1. Rellene los cuadros de texto **Tipo de grupo**, **Nombre del grupo**, **Descripción del grupo** y **Tipo de pertenencia**. Seleccione las flechas para seleccionar miembros y, a continuación, busque o seleccione los miembros que desea agregar al grupo. Elija **Seleccionar** para agregar los miembros seleccionados y, a continuación, seleccione **Crear**.

![Creación de un grupo de seguridad de Azure AD](./media/sharepoint-on-premises-tutorial/new-group.png)

### <a name="grant-permissions-to-an-azure-ad-account-in-sharepoint-on-premises"></a>Concesión de permisos a una cuenta de Azure AD en SharePoint local

Para conceder acceso a un usuario de Azure AD en SharePoint local, comparta la colección de sitios o agregue el usuario de Azure AD a uno de los grupos de la colección de sitios. Los usuarios ahora pueden iniciar sesión en SharePoint 201x mediante identidades de Azure AD, pero todavía hay oportunidades de mejora de la experiencia del usuario. Por ejemplo, la búsqueda de un usuario da lugar a varios resultados de búsqueda en el selector de personas. Hay un resultado de búsqueda por cada uno de los tipos de notificación creados en la asignación de notificaciones. Para elegir un usuario mediante el selector de personas, debe escribir su nombre de usuario exactamente y elegir el resultado de notificación **Nombre**.

![Resultados de búsqueda de notificaciones](./media/sharepoint-on-premises-tutorial/claims-search-results.png)

No hay ninguna validación de los valores que se buscan, lo cual puede dar lugar a errores ortográficos o a que los usuarios elijan accidentalmente el tipo de notificación incorrecto. Esto puede impedir que los usuarios accedan correctamente a los recursos.

Para corregir esta situación con el selector de personas, hay una solución de código abierto llamada [AzureCP](https://yvand.github.io/AzureCP/) que proporciona un proveedor de notificaciones personalizado para SharePoint 2013, 2016 y 2019. Usa Microsoft Graph API para resolver lo que escriben los usuarios y realizar la validación. Para más información, consulte [AzureCP](https://yvand.github.io/AzureCP/).

  > [!NOTE]
  > Sin AzureCP, puede agregar grupos mediante la adición del identificador del grupo de Azure AD, pero este método no es fácil ni confiable para el usuario. Este es su aspecto:
  > 
  >![Adición de un grupo de Azure AD a un grupo de SharePoint](./media/sharepoint-on-premises-tutorial/adding-group-by-id.png)
  
### <a name="grant-permissions-to-an-azure-ad-group-in-sharepoint-on-premises"></a>Concesión de permisos a un grupo de Azure AD en SharePoint local

Para asignar grupos de seguridad de Azure AD a SharePoint local, es necesario usar un proveedor de notificaciones personalizado para el servidor de SharePoint. En este ejemplo se usa AzureCP.

 > [!NOTE]
 > AzureCP no es un producto de Microsoft y no dispone de soporte técnico por parte del equipo de Soporte técnico de Microsoft. Para descargar, instalar y configurar AzureCP en la granja de servidores de SharePoint local, consulte el sitio web de [AzureCP](https://yvand.github.io/AzureCP/). 

1. Configure AzureCP en la granja de servidores de SharePoint local o una solución alternativa de proveedor de notificaciones personalizado. Para configurar AzureCP, consulte este sitio web de [AzureCP](https://yvand.github.io/AzureCP/Register-App-In-AAD.html).

1. En Azure Portal, seleccione **Azure Active Directory** > **Aplicaciones empresariales**. Seleccione el nombre de la aplicación empresarial creada anteriormente y seleccione **Inicio de sesión único**.

1. En la página **Configuración del inicio de sesión único con SAML**, edite la sección **Atributos y notificaciones de usuario**.

1. Seleccione **Agregar una notificación de grupo**.

1. Seleccione los grupos asociados al usuario que se deben devolver en la notificación. En este caso, seleccione **Todos los grupos**. En la sección **Atributo de origen**, seleccione **Id. de grupo** y seleccione **Guardar**.

Para conceder acceso al grupo de seguridad de Azure AD en SharePoint local, comparta la colección de sitios o agregue el grupo de seguridad de Azure AD a uno de los grupos de la colección de sitios.

1. Vaya a **Colección de sitios de SharePoint**. En **Configuración del sitio** de la colección de sitios, seleccione **Personas y grupos**. 

1. Seleccione el grupo de SharePoint y, a continuación, seleccione **Nuevo** > **Agregar usuarios a este grupo**. A medida que se escribe el nombre del grupo, el selector de personas muestra el grupo de seguridad de Azure AD.

    ![Adición de un grupo de Azure AD a un grupo de SharePoint](./media/sharepoint-on-premises-tutorial/permission-azure-ad-group.png)

### <a name="grant-access-to-a-guest-account-to-sharepoint-on-premises-in-the-azure-portal"></a>Concesión de acceso a una cuenta de invitado a SharePoint local en Azure Portal

Puede conceder acceso al sitio de SharePoint a una cuenta de invitado de una manera coherente, ya que ahora se modifica el UPN. Por ejemplo, el usuario `jdoe@outlook.com` se representa como `jdoe_outlook.com#ext#@TENANT.onmicrosoft.com`. Para compartir el sitio con usuarios externos, debe agregar algunas modificaciones en la sección **Atributos y notificaciones de usuario** en Azure Portal.

1. En Azure Portal, seleccione **Azure Active Directory** > **Aplicaciones empresariales**. Seleccione el nombre de la aplicación empresarial creada anteriormente y seleccione **Inicio de sesión único**.

1. En la página **Configuración del inicio de sesión único con SAML**, edite la sección **Atributos y notificaciones de usuario**.

1. En la zona **Notificación requerida**, seleccione **Identificador de usuario único (id. de nombre)** .

1. Cambie la propiedad **Atributo de origen** con el valor **user.localuserprincipalname** y seleccione **Guardar**.

    ![Estado inicial de Atributos y notificaciones de usuario](./media/sharepoint-on-premises-tutorial/manage-claim.png)

1. Mediante la cinta de opciones, vuelva a **Inicio de sesión basado en SAML**. Ahora, la sección **Atributos y notificaciones de usuario** tiene el siguiente aspecto: 

    ![Estado final de Atributos y notificaciones de usuario](./media/sharepoint-on-premises-tutorial/user-attributes-claims-final.png)

    > [!NOTE]
    > No se necesitan los apellidos ni el nombre propio en esta configuración.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory** y, a continuación, seleccione **Usuarios**.

1. Seleccione **Nuevo usuario invitado**.

1. Seleccione la opción **Invitar usuario**. Rellene las propiedades de usuario y seleccione **Invitar**.

1. Ahora puede compartir el sitio con MyGuestAccount@outlook.com y permitir que este usuario tenga acceso a él.

    ![Compartición de un sitio con una cuenta de invitado](./media/sharepoint-on-premises-tutorial/sharing-guest-account.png)

### <a name="configure-the-trusted-identity-provider-for-multiple-web-applications"></a>Configuración del proveedor de identidades de confianza para varias aplicaciones web

La configuración funciona para una sola aplicación web, pero se necesitará configuración adicional si va a usar el mismo proveedor de identidades de confianza con varias aplicaciones web. Por ejemplo, suponga que se ha ampliado una aplicación web para usar la dirección URL `https://sales.contoso.com` y ahora quiere autenticar a los usuarios en `https://marketing.contoso.com`. Para ello, actualice el proveedor de identidades para respetar el parámetro WReply y actualice el registro de aplicación en Azure AD para agregar una dirección URL de respuesta.

1. En Azure Portal, seleccione **Azure Active Directory** > **Aplicaciones empresariales**. Seleccione el nombre de la aplicación empresarial creada anteriormente y seleccione **Inicio de sesión único**.

1. En la página **Configuración del inicio de sesión único con SAML**, edite la sección **Configuración básica de SAML**.

    ![Configuración básica de SAML](./media/sharepoint-on-premises-tutorial/add-reply-url.png)

1. En **Dirección URL de respuesta (URL del Servicio de consumidor de aserciones)** , agregue la dirección URL de las aplicaciones web adicionales y seleccione **Guardar**.

    ![Edición de la configuración básica de SAML](./media/sharepoint-on-premises-tutorial/reply-url-for-web-application.png)

1. En el servidor de SharePoint, abra el shell de administración de SharePoint 201x y ejecute los siguientes comandos. Use el nombre del emisor del token de identidad de confianza que utilizó anteriormente.
    ```
    $t = Get-SPTrustedIdentityTokenIssuer "AzureAD"
    $t.UseWReplyParameter=$true
    $t.Update()
    ```
1. En **Administración Central**, vaya a la aplicación web y habilite el proveedor de identidades de confianza existente.

Es posible que tenga otros escenarios en los que quiera conceder acceso a la instancia de SharePoint local para los usuarios internos. En este escenario, debe implementar Microsoft Azure Active Directory Connect para permitir la sincronización de los usuarios locales con Azure AD. Esta configuración se describe en otro artículo.

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)
- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)
- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)
- [¿Qué es la identidad híbrida con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/hybrid/whatis-hybrid-identity)
