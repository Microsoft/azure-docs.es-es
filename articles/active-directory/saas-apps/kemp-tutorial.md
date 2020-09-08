---
title: 'Tutorial: Integración del inicio de sesión único de Azure Active Directory con Kemp LoadMaster | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Kemp LoadMaster.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/28/2020
ms.author: jeedes
ms.openlocfilehash: 1d974badd715d7966f552603a77d77aab95d70f7
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88549976"
---
# <a name="tutorial-azure-active-directory-sso-integration-with-kemp-loadmaster-azure-ad-integration"></a>Tutorial: Integración del inicio de sesión único de Azure Active Directory con Kemp LoadMaster

En este tutorial, aprenderá a integrar Kemp LoadMaster con Azure Active Directory (Azure AD). Al integrar Kemp LoadMaster con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Kemp LoadMaster.
* Permitir que los usuarios inicien sesión automáticamente en Kemp LoadMaster con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Kemp LoadMaster.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Kemp LoadMaster admite el inicio de sesión único iniciado por **IDP**.
* Una vez configurado la integración de Azure AD para Kemp LoadMaster, puede aplicar el control de sesión, que protege su organización, en tiempo real, frente a la filtración y la infiltración de la información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-kemp-loadmaster-azure-ad-integration-from-the-gallery"></a>Incorporación de Kemp LoadMaster desde la galería

Para configurar la integración de Kemp LoadMaster en Azure AD, es preciso agregar Kemp LoadMaster desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Kemp LoadMaster** en el cuadro de búsqueda.
1. Seleccione **Kemp LoadMaster** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-kemp-loadmaster-azure-ad-integration"></a>Configuración y prueba del inicio de sesión único de Azure AD para Kemp LoadMaster

Configure y pruebe el inicio de sesión único de Azure AD con Kemp LoadMaster mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Kemp LoadMaster.

Para configurar y probar el inicio de sesión único de Azure AD con Kemp LoadMaster, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Kemp LoadMaster](#configure-kemp-loadmaster-azure-ad-integration-sso)** , para configurar los valores de inicio de sesión único en la aplicación.

1. **[Publicación del servidor web](#publishing-web-server)**
    1. **[Creación de un servicio virtual](#create-a-virtual-service)**
    1. **[Certificados y seguridad](#certificates-and-security)**
    1. **[Perfil SAML de Kemp LoadMaster](#kemp-loadmaster-azure-ad-integration-saml-profile)**
    1. **[Comprobación de los cambios](#verify-the-changes)**
1. **[Configuración de la autenticación basada en Kerberos](#configuring-kerberos-based-authentication)**
    1. **[Creación de una cuenta de delegación de Kerberos para Kemp LoadMaster](#create-a-kerberos-delegation-account-for-kemp-loadmaster-azure-ad-integration)**
    1. **[KCD (cuentas de delegación de Kerberos) de Kemp LoadMaster](#kemp-loadmaster-azure-ad-integration-kcd-kerberos-delegation-accounts)**
    1. **[ESP de Kemp LoadMaster](#kemp-loadmaster-azure-ad-integration-esp)**
    1. **[Creación de un usuario de prueba de Kemp LoadMaster](#create-kemp-loadmaster-azure-ad-integration-test-user)** , para tener un homólogo de B.Simon en Kemp LoadMaster vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Kemp LoadMaster**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador (Id. de entidad)** , escriba una dirección URL: `https://<KEMP-CUSTOMER-DOMAIN>.com/`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL: `https://<KEMP-CUSTOMER-DOMAIN>.com/`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Póngase en contacto con el [equipo de soporte técnico para clientes de Kemp LoadMaster](mailto:support@kemp.ax) para obtener estos valores. También puede consultar los patrones que se muestran en la sección Configuración básica de SAML de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base 64)** y **XML de metadatos de federación**, y seleccione **Descargar** para descargar el certificado y los archivos XML de metadatos de federación y guardarlos en su equipo.

    ![Vínculo de descarga del certificado](./media/kemp-tutorial/certificate-base-64.png)

1. En la sección **Configurar Kemp LoadMaster**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección va a permitir que B.Simon acceda a Kemp LoadMaster mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Kemp LoadMaster**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-kemp-loadmaster-azure-ad-integration-sso"></a>Configuración del inicio de sesión único de Kemp LoadMaster

## <a name="publishing-web-server"></a>Publicación del servidor web 

### <a name="create-a-virtual-service"></a>Creación de un servicio virtual 

1. Vaya a la interfaz de usuario web de Kemp LoadMaster > Virtual Services > Add New (Servicios virtuales > Agregar nuevo).

1. Haga clic en Add New (Agregar nuevo).

1. Especifique los parámetros del servicio virtual.

    ![Servidor web de Kemp LoadMaster](./media/kemp-tutorial/kemp-1.png)

    a. Virtual Address (Dirección virtual)
    
    b. Port
    
    c. Service Name (Optional) (Nombre del servicio [opcional])
    
    d. Protocolo 

1. Vaya a la sección Real Servers (Servidores reales).

1. Haga clic en Add New (Agregar nuevo).

1. Especifique los parámetros del servidor real.
    
    ![Servidor web de Kemp LoadMaster](./media/kemp-tutorial/kemp-2.png)

    a. Seleccione Allow Remote Addresses (Permitir direcciones remotas).
    
    b. Escriba el valor de Real Server Address (Dirección real del servidor).
    
    c. Port
    
    d. Forwarding method (Método de reenvío)
    
    e. Peso
    
    f. Connection Limit (Límite de conexiones)
    
    g. Haga clic en Add This Real Server (Agregar este servidor real).

## <a name="certificates-and-security"></a>Certificados y seguridad
 
### <a name="import-certificate-on-kemp-loadmaster-azure-ad-integration"></a>Importación del certificado en Kemp LoadMaster 
 
1. Vaya al portal web de Kemp LoadMaster > Certificates & Security > SSL Certificates (Certificados y seguridad > Certificados SSL).

1. En Manage Certificates > Certificate Configuration (Administrar certificados > Configuración del certificado).

1. Haga clic en Import Certificate (Importar certificado).

1. Especifique el nombre del archivo que contiene el certificado. El archivo también puede contener la clave privada. Si el archivo no contiene la clave privada, también se debe especificar el archivo que la contiene. El certificado puede estar en formato .PEM o .PFX (IIS).

1. En Certificate File (Archivo de certificado), haga clic en Choose file (Elegir archivo).

1. Haga clic en Key File (Archivo de clave) (opcional).

1. Haga clic en Guardar.

### <a name="ssl-acceleration"></a>Aceleración de SSL
 
1. Vaya a la interfaz de usuario web de Kemp LoadMaster > Virtual Services > Virtual Services (Servicios virtuales > Servicios virtuales).

1. Haga clic en Modify under Operation (Modificar en funcionamiento).

1. En Properties for tcp/x.x.x.:443 (Id:6) - Operating at Layer 7 (Propiedades de tcp/x.x.x.:443 (Id:6) - Funcionando en el nivel 7), haga clic en SSL Properties (Propiedades de SSL).
    
    ![Servidor web de Kemp LoadMaster](./media/kemp-tutorial/kemp-3.png)
    
    a. Haga clic en Enabled (Habilitado) en SSL Acceleration (Aceleración de SSL).
    
    b. En Available Certificates (Certificados disponibles), seleccione el certificado importado y haga clic en el símbolo `>`.
    
    c. Una vez que aparezca el certificado SSL deseado en Assigned Certificates (Certificados asignados), haga clic en **Set Certificates** (Establecer certificados).

    > [!NOTE]
    > Asegúrese de hacer clic en **Set Certificates** (Establecer certificados).

## <a name="kemp-loadmaster-azure-ad-integration-saml-profile"></a>Perfil SAML de Kemp LoadMaster
 
### <a name="import-idp-certificate"></a>Importación de certificado de IdP

Vaya a la consola web de Kemp LoadMaster. 

1. En Certificates and Authority (Certificados y autoridad), haga clic en Intermediate Certificates (Certificados intermedios).

    ![Servidor web de Kemp LoadMaster](./media/kemp-tutorial/kemp-6.png)

    a. Haga clic en Choose File (Elegir archivo) en Add a new Intermediate Certificate (Agregar nuevo certificado intermedio).
    
    b. Vaya al archivo de certificado descargado previamente de la aplicación empresarial de Azure AD.
    
    c. Haga clic en Open (Abrir).
    
    d. Indique un nombre en Certificate Name (Nombre de certificado).
    
    e. Haga clic en Add Certificate (Agregar certificado).

### <a name="create-authentication-policy"></a>Creación de la directiva de autenticación 
 
Vaya a Manage SSO (Administrar SSO) en Virtual Services (Servicios virtuales).

   ![Servidor web de Kemp LoadMaster](./media/kemp-tutorial/kemp-7.png)
   
   a. En Add new Client Side Configuration (Agregar nueva configuración del lado cliente), haga clic en Add (Agregar) después de darle un nombre.

   b. En Authentication Protocol (Protocolo de autenticación), seleccione SAML.

   c. En IdP Provisioning (Aprovisionamiento de IdP), seleccione MetaData File (Archivo de metadatos). 

   d. Haga clic en Choose File (Elegir archivo).

   e. Vaya al archivo XML descargado previamente de Azure Portal.

   f. Haga clic en Open (Abrir) y en Import IdP MetaData file (Importar archivo de metadatos de IdP).

   g. Seleccione el certificado intermedio en IdP Certificate (Certificado de IdP).

   h. Establezca el valor de SP Entity ID (Identificador de entidad del proveedor de servicios), que debe coincidir con la identidad creada en Azure Portal. 

   i. Haga clic en Set SP Entity ID (Establecer id. de entidad del proveedor de servicios).

### <a name="set-authentication"></a>Establecimiento de la autenticación  
 
En la consola web de Kemp LoadMaster:

1. Haga clic en Virtual Services (Servicios virtuales).

1. Haga clic en View/Modify Services (Ver o modificar servicios).

1. Haga clic en Modify (Modificar) y vaya a ESP Options (Opciones de ESP).
    
    ![Servidor web de Kemp LoadMaster](./media/kemp-tutorial/kemp-8.png)

    a. Haga clic en Enable ESP (Habilitar ESP).
    
    b. En Client Authentication Mode (Modo de autenticación de cliente), seleccione SAML.
    
    c. En SSO Domain (Dominio de SSO), seleccione la autenticación de cliente creada anteriormente.
    
    d. Escriba el nombre de host en Allowed Virtual Hosts (Hosts virtuales permitidos) y haga clic en Set Allowed Virtual Hosts (Establecer hosts virtuales permitidos).
    
    e. En Allowed Virtual Directories (Directorios virtuales permitidos), escriba /* (según los requisitos de acceso) y haga clic en Set Allowed Directories (Establecer directorios permitidos).

### <a name="verify-the-changes"></a>Comprobación de los cambios 
 
Vaya a la dirección URL de la aplicación. 

Debería ver la página de inicio de sesión del inquilino en lugar del acceso no autenticado anterior. 

![Servidor web de Kemp LoadMaster](./media/kemp-tutorial/kemp-9.png)

## <a name="configuring-kerberos-based-authentication"></a>Configuración de la autenticación basada en Kerberos 
 
### <a name="create-a-kerberos-delegation-account-for-kemp-loadmaster-azure-ad-integration"></a>Creación de una cuenta de delegación de Kerberos para Kemp LoadMaster 

1. Cree una cuenta de usuario (en este ejemplo, AppDelegation).
    
    ![Servidor web de Kemp LoadMaster](./media/kemp-tutorial/kemp-10.png)


    a. Seleccione la pestaña Editor de atributos.

    b. Vaya a servicePrincipalName. 

    c. Seleccione servicePrincipalName y haga clic en Editar.

    d. Escriba http/kcduser en el campo Valor que se agregará y haga clic en Agregar. 

    e. Haga clic en Aplicar y en Aceptar. La ventana debe cerrarse antes de abrirla de nuevo (para ver la nueva pestaña Delegación). 

1. Vuelva a abrir la ventana de propiedades de usuario, que ahora tendrá la pestaña Delegación disponible. 

1. Seleccione la pestaña Delegación.

    ![Servidor web de Kemp LoadMaster](./media/kemp-tutorial/kemp-11.png)

    a. Seleccione Confiar en este usuario para la delegación solo a los servicios especificados.

    b. Seleccione Usar cualquier protocolo de autenticación.

    c. Agregue los servidores reales y agregue http como servicio. 
    
    d. Active la casilla Expandido. 

    e. Puede ver todos los servidores con el nombre de host y el FQDN.
    
    f. Haga clic en Aceptar.

> [!NOTE]
> Establezca el SPN en la aplicación o el sitio web según corresponda, para acceder a la aplicación cuando se haya establecido la identidad del grupo de aplicaciones. Para acceder a la aplicación de IIS con el nombre FQDN, vaya a símbolo del sistema del servidor real y escriba SetSpn con los parámetros necesarios. Por ejemplo, Setspn –S HTTP/sescoindc.sunehes.co.in suneshes\kdcuser. 

### <a name="kemp-loadmaster-azure-ad-integration-kcd-kerberos-delegation-accounts"></a>KCD (cuentas de delegación de Kerberos) de Kemp LoadMaster 

Vaya a la consola web de Kemp LoadMaster > Virtual Services Manage SSO (Servicios virtuales > Administrar SSO).

![Servidor web de Kemp LoadMaster](./media/kemp-tutorial/kemp-12.png)

a. Vaya a Server Side Single Sign On Configurations (Configuraciones de inicio de sesión único del lado servidor).

b. Escriba un nombre en Add new Server-Side Configuration (Agregar nueva configuración del lado servidor) y haga clic en Add (Agregar).

c. Seleccione Kerberos Constrained Delegation (Delegación restringida de Kerberos) en Authentication Protocol (Protocolo de autenticación).

d. Escriba el nombre de dominio en Kerberos Realm (Dominio Kerberos).

e. Haga clic en Set Kerberos realm (Establecer dominio Kerberos).

f. Escriba la dirección IP del controlador de dominio en Kerberos Key Distribution Center (Centro de distribución de claves Kerberos).

g. Haga clic en Set Kerberos KDC (Establecer KDC de Kerberos).

h. Escriba el nombre de usuario de KCD en Kerberos Trusted User Name (Nombre de usuario de confianza de Kerberos).

i. Haga clic en Set KDC trusted user name (Establecer nombre de usuario de confianza de KDC).

j. Escriba la contraseña en Kerberos Trusted User Password (Contraseña de usuario de confianza de Kerberos).

k. Haga clic en Set KCD trusted user password (Establecer contraseña de usuario de confianza de KCD).

### <a name="kemp-loadmaster-azure-ad-integration-esp"></a>ESP de Kemp LoadMaster 

Vaya a Virtual Services > View/Modify Services (Servicios virtuales > Ver o modificar servicios).

![Servidor web de Kemp LoadMaster](./media/kemp-tutorial/kemp-13.png)

a. Haga clic en Modify (Modificar) en el nombre de alias del servicio virtual.
    
b. Haga clic en ESP Options (Opciones de ESP).
    
c. En Server Authentication Mode (Modo de autenticación del servidor), seleccione KCD.
        
d. En Server-Side configuration (Configuración del lado servidor), seleccione el perfil de servidor creado anteriormente.

### <a name="create-kemp-loadmaster-azure-ad-integration-test-user"></a>Creación de un usuario de prueba de Kemp LoadMaster

En esta sección, va a crear un usuario llamado B.Simon en Kemp LoadMaster. Colabore con el [equipo de atención al cliente de Kemp LoadMaster](mailto:support@kemp.ax) para agregar los usuarios a la plataforma de Kemp LoadMaster. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Kemp LoadMaster en el Panel de acceso, debería iniciar sesión automáticamente en la instancia de Kemp LoadMaster para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de Kemp LoadMaster con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Kemp LoadMaster con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
