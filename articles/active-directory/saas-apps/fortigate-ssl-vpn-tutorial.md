---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con FortiGate SSL VPN | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y FortiGate SSL VPN.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 18a3d9d5-d81c-478c-be7e-ef38b574cb88
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 08/11/2020
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: ecb53d661b1171f9c1b18d37d0bb35952645ba7e
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89299718"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-fortigate-ssl-vpn"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con FortiGate SSL VPN

En este tutorial, obtendrá información sobre cómo integrar FortiGate SSL VPN con Azure Active Directory (Azure AD). Al integrar FortiGate SSL VPN con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a FortiGate SSL VPN.
* Permitir que los usuarios puedan iniciar sesión automáticamente en FortiGate SSL VPN con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en FortiGate SSL VPN.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* FortiGate SSL VPN admite el inicio de sesión único iniciado por **SP**.
* Una vez que configure FortiGate SSL VPN, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-fortigate-ssl-vpn-from-the-gallery"></a>Adición de FortiGate SSL VPN desde la galería

Para configurar la integración de FortiGate SSL VPN en Azure AD, deberá agregar FortiGate SSL VPN desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **FortiGate SSL VPN** en el cuadro de búsqueda.
1. Seleccione **FortiGate SSL VPN** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-fortigate-ssl-vpn"></a>Configuración y prueba del inicio de sesión único de Azure AD para FortiGate SSL VPN

Configure y pruebe el inicio de sesión único de Azure AD con FortiGate SSL VPN mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de FortiGate SSL VPN.

Para configurar y probar el inicio de sesión único de Azure AD con FortiGate SSL VPN, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en FortiGate SSL VPN](#configure-fortigate-ssl-vpn-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **Creación de un usuario de prueba en FortiGate SSL VPN** , para tener un homólogo de B.Simon en FortiGate SSL VPN que esté vinculado a su representación en Azure AD.
1. **[Prueba del inicio de sesión único](#test-single-sign-on)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **FortiGate SSL VPN**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configurar el inicio de sesión único con SAML**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<FQDN>/remote/login`

    b. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<FQDN>/remote/saml/metadata`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://><FQDN/remote/saml/login`

    d. En el cuadro de texto **URL de cierre de sesión**, escriba una dirección URL con el siguiente patrón: `https://<FQDN>/remote/saml/logout`.

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de inicio de sesión, el identificador, la dirección URL de respuesta y la URL de cierre de sesión reales. Póngase en contacto con el [equipo de soporte técnico de clientes de FortiGate SSL VPN](mailto:tac_amer@fortinet.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación FortiGate SSL VPN espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación FortiGate SSL VPN espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.
    
    | Nombre |  Atributo de origen|
    | ------------ | --------- |
    | username | user.userprincipalname |
    | group | user.groups |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar FortiGate SSL VPN**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección va a permitir que B.Simon acceda a FortiGate SSL VPN mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **FortiGate SSL VPN**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

### <a name="create-a-security-group-for-the-test-user"></a>Creación de un grupo de seguridad para el usuario de prueba

En esta sección, creará un grupo de seguridad en Azure Active Directory para el usuario de prueba. FortiGate usará este grupo de seguridad para conceder al usuario acceso a la red a través de la VPN.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory** y, a continuación, **Grupos**.
1. Seleccione **Nuevo grupo** en la parte superior de la pantalla.
1. En las propiedades del **Nuevo grupo**, siga estos pasos:
   1. En el campo **Tipo de grupo**, seleccione **Seguridad**.
   1. En el campo **Nombre**, escriba `FortiGateAccess`.
   1. En el campo **Descripción del grupo**, escriba `Group for granting FortiGate VPN access`.
   1. En la configuración **Los roles de Azure AD se pueden asignar a un grupo (versión preliminar)** , seleccione **No**.
   1. En el campo **Tipo de pertenencia**, seleccione **Asignado**.
   1. En **Miembros**, seleccione **-no hay miembros seleccionados**.
   1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
   1. Seleccione **Crear**.
1. Una vez que haya regresado a la hoja **Grupos** en Azure Active Directory, busque el grupo **Acceso a FortiGate** y anote el **id. de objeto** para su uso posterior.

## <a name="configure-fortigate-ssl-vpn-sso"></a>Configuración del inicio de sesión único de FortiGate SSL VPN

### <a name="upload-the-base64-saml-certificate-to-the-fortigate-appliance"></a>Carga del certificado SAML de Base64 en el dispositivo FortiGate

Después de completar la configuración de SAML de la aplicación FortiGate en el inquilino, descargó el certificado SAML codificado en Base64. Dicho certificado debe cargarse en el dispositivo FortiGate:

1. Inicie sesión en el portal de administración de la aplicación FortiGate.
1. En el menú de la izquierda, haga clic en **Sistema**.
1. En **Sistema**, haga clic en **Certificados**.
1. Haga clic en **Importar** -> **Certificado remoto**.
1. Busque el certificado descargado de la implementación de la aplicación FortiGate en el inquilino de Azure, selecciónelo y haga clic en **Aceptar**.

Una vez cargado el certificado, anote su nombre en **Sistema** > **Certificados** > **Certificado remoto**. De manera predeterminada, se llamará REMOTE_Cert_**N**, donde **N** es un valor entero.

### <a name="perform-fortigate-command-line-configuration"></a>Configuración de la línea de comandos de FortiGate

Los pasos siguientes requieren que se configure la dirección URL de cierre de sesión de Azure. Esta dirección URL contiene un signo de interrogación (?). Se requieren pasos especiales para enviar este carácter correctamente. Los pasos no se pueden realizar desde la consola de la CLI de FortiGate. En su lugar, establezca una sesión de SSH en el dispositivo FortiGate con una herramienta como PuTTy. Si el dispositivo FortiGate es una máquina virtual de Azure, puede realizar los pasos siguientes desde la consola serie de la máquina virtual de Azure.

Para realizar estos pasos, necesitará los valores que se registraron anteriormente:

- El identificador de entidad
- URL de respuesta
- URL de cierre de sesión
- Dirección URL de inicio de sesión de Azure
- Identificador de Azure AD
- Dirección URL de cierre de sesión de Azure
- Nombre del certificado SAML de Base64 (REMOTE_Cert_N)

1. Establezca una sesión de SSH en el dispositivo FortiGate e inicie sesión con una cuenta de administrador de FortiGate.
1. Ejecute los siguientes comandos:

   ```console
    config user saml
    edit azure
    set entity-id <Entity ID>
    set single-sign-on-url <Reply URL>
    set single-logout-url <Logout URL>
    set idp-single-sign-on-url <Azure Login URL>
    set idp-entity-id <Azure AD Identifier>
    set idp-single-logout-url <Azure Logout URL>
    set idp-cert <Base64 SAML Certificate Name>
    set user-name username
    set group-name group
    end

   ```

   > [!NOTE]
   > La **dirección URL de cierre de sesión de Azure** contiene un carácter `?`. Debe especificar una secuencia de clave especial para proporcionar correctamente la dirección URL a la consola serie de FortiGate. Normalmente, la dirección URL es `https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.0`.
   >
   > Para especificar la dirección URL de cierre de sesión de Azure en la consola serie, escriba `set idp-single-logout-url https://login.microsoftonline.com/common/wsfederation`.
   > 
   > A continuación, seleccione CTRL+V y pegue el resto de la dirección URL para completar la línea: `set idp-single-logout-url https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.0`.

### <a name="configure-fortigate-for-group-matching"></a>Configuración de FortiGate para la coincidencia de grupos

En esta sección, configurará FortiGate para reconocer el id. de objeto del grupo de seguridad en el que reside el usuario de prueba. Esto permitirá a FortiGate tomar decisiones de acceso basadas en la pertenencia a este grupo.

Para realizar estos pasos, necesitará el id. de objeto del grupo de seguridad de **FortiGateAccess** creado anteriormente.

1. Establezca una sesión de SSH en el dispositivo FortiGate e inicie sesión con una cuenta de administrador de FortiGate.
1. Ejecute los siguientes comandos:

   ```
    config user group
    edit FortiGateAccess
    set member azure
    config match
    edit 1
    set server-name azure
    set group-name <Object Id>
    next
    end
    next
    end
   ```

### <a name="create-fortigate-vpn-portals-and-firewall-policy"></a>Creación de portales de VPN y de la directiva de firewall de FortiGate

En esta sección, configurará los portales de VPN y la directiva de firewall de FortiGate que conceden acceso al grupo de seguridad **FortiGateAccess** creado anteriormente.

Trabaje con el  [equipo de soporte técnico de FortiGate](mailto:tac_amer@fortinet.com) para agregar los portales de VPN y la directiva de firewall a la plataforma de VPN de FortiGate. Estos pasos deben completarse antes de usar el inicio de sesión único.

## <a name="test-single-sign-on"></a>Prueba de inicio de sesión único 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de FortiGate SSL VPN en el Panel de acceso, debería iniciar sesión automáticamente en la versión de FortiGate SSL VPN para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

Microsoft y FortiGate recomiendan usar el cliente VPN de Fortinet, FortiClient, para obtener la mejor experiencia del usuario final.

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de FortiGate SSL VPN con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
