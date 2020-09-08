---
title: 'Tutorial: Integración del inicio de sesión único de Azure Active Directory con Citrix NetScaler (autenticación basada en encabezados) | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único (SSO) entre Azure Active Directory y Citrix NetScaler mediante la autenticación basada en encabezados.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/13/2019
ms.author: jeedes
ms.openlocfilehash: 48fedf7f3a73ce7fde60a1df80d971a5d7f88dd7
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88540650"
---
# <a name="tutorial-azure-active-directory-single-sign-on-integration-with-citrix-netscaler-header-based-authentication"></a>Tutorial: Integración del inicio de sesión único de Azure Active Directory con Citrix NetScaler (autenticación basada en encabezados)

En este tutorial, aprenderá a integrar Citrix NetScaler con Azure Active Directory (Azure AD). Al integrar Citrix NetScaler con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Citrix NetScaler.
* Permitir que los usuarios inicien sesión automáticamente en Citrix NetScaler con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Citrix NetScaler.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. El tutorial incluye estos escenarios:

* Inicio de sesión único **iniciado por SP** para Citrix NetScaler

* Aprovisionamiento de usuarios **Just-In-Time** para Citrix NetScaler

* [Autenticación basada en encabezados para Citrix NetScaler](#publish-the-web-server)

* [Autenticación basada en Kerberos para Citrix NetScaler](citrix-netscaler-tutorial.md#publish-the-web-server)

## <a name="add-citrix-netscaler-from-the-gallery"></a>Incorporación de Citrix NetScaler desde la galería

Para integrar Citrix NetScaler con Azure AD, primero agregue esta aplicación a la lista de aplicaciones SaaS administradas desde la galería:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.

1. Seleccione **Azure Active Directory** en el menú izquierdo.

1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.

1. En la sección **Agregar desde la galería**, escriba **Citrix NetScaler** en el cuadro de búsqueda.

1. En los resultados, seleccione **Citrix NetScaler** y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-citrix-netscaler"></a>Configuración y prueba del inicio de sesión único de Azure AD para Citrix NetScaler

Configure y pruebe el inicio de sesión único (SSO) de Azure AD con Citrix NetScaler mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Citrix NetScaler.

Para configurar y probar el inicio de sesión único de Azure AD con Citrix NetScaler, es preciso completar los siguientes bloques de creación:

1. [Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso), para que los usuarios puedan utilizar esta característica.

    1. [Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user), para probar el inicio de sesión único (SSO) de Azure AD con el usuario B.Simon.

    1. [Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user), para que B.Simon pueda usar el inicio de sesión único de Azure AD.

1. [Configuración del inicio de sesión único de Citrix NetScaler](#configure-citrix-netscaler-sso), para configurar el inicio de sesión único (SSO) en la aplicación.

    * [Creación de un usuario de prueba de Citrix NetScaler](#create-a-citrix-netscaler-test-user), para tener un homólogo de B.Simon en Citrix NetScaler que esté vinculado a la representación del usuario en Azure AD.

1. [Comprobación del inicio de sesión único](#test-sso), para comprobar que la configuración funciona correctamente.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Para habilitar el inicio de sesión único de Azure AD en Azure Portal, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), en el panel de integración de aplicaciones de **Citrix NetScaler**, en **Administrar**, seleccione **Inicio de sesión único**.

1. En el panel **Seleccione un método de inicio de sesión único**, seleccione **SAML**.

1. En el panel **Configurar el inicio de sesión único con SAML**, seleccione el icono con forma de lápiz **Editar** para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, para configurar la aplicación en modo **iniciado por IDP**:

    1. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente formato: `https://<Your FQDN>`.

    1. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente formato: `https://<Your FQDN>/CitrixAuthService/AuthService.asmx`.

1. Para configurar la aplicación en modo **iniciado por SP**, seleccione **Establecer direcciones URL adicionales** y haga lo siguiente:

    * En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente formato: `https://<Your FQDN>/CitrixAuthService/AuthService.asmx`.

    > [!NOTE]
    > * Las direcciones URL que se usan en esta sección no son valores reales. Actualice estos valores con los reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de Citrix NetScaler](https://www.citrix.com/contact/technical-support.html) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.
    > * Para configurar el inicio de sesión único, las direcciones URL deben ser accesibles desde sitios web públicos. Deberá habilitar el firewall u otras opciones de seguridad en Citrix NetScaler para que Azure AD pueda publicar el token en la dirección URL configurada.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, en **Dirección URL de metadatos de federación de aplicación**, copie la dirección URL y guárdela en el Bloc de notas.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. La aplicación Citrix NetScaler espera que las aserciones de SAML tengan un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados. Seleccione el icono **Editar** y cambie la asignación de atributos.

    ![Edición de las asignaciones de atributos de SAML](common/edit-attribute.png)

1. La aplicación Citrix NetScaler también espera que se devuelvan algunos atributos más en la respuesta de SAML. En el cuadro de diálogo **Atributos de usuario**, en **Notificaciones de usuario**, complete los siguientes pasos para agregar los atributos del token SAML como se muestra en la tabla:

    | Nombre | Atributo de origen|
    | ---------------| --------------- |
    | mySecretID  | user.userprincipalname |
    
    1. Seleccione **Agregar nueva notificación** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

    1. En el cuadro de texto **Nombre**, escriba el nombre del atributo que se muestra para esa fila.

    1. Deje **Espacio de nombres** en blanco.

    1. En **Atributo**, seleccione **Origen**.

    1. En la lista **Atributo de origen**, escriba el valor del atributo que se muestra para esa fila.

    1. Seleccione **Aceptar**.

    1. Seleccione **Guardar**.

1. En la sección **Set up Citrix NetScaler** (Configurar Citrix NetScaler), copie las direcciones URL pertinentes según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En el menú izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.

1. Seleccione **Nuevo usuario** en la parte superior del panel.

1. En las propiedades de **Usuario**, realice estos pasos:

   1. En **Nombre**, escriba `B.Simon`.  

   1. En **Nombre de usuario**, escriba _username@companydomain.extension_ . Por ejemplo, `B.Simon@contoso.com`.

   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Password** (Contraseña).

   1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, concederá a B.Simon acceso a Citrix NetScaler para que pueda usar el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.

1. En la lista de aplicaciones, seleccione **Citrix NetScaler**.

1. En la información general de la aplicación, en **Administrar**, seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. Después, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, en la lista **Usuarios** seleccione **B.Simon**. Elija **Seleccionar**.

1. Si espera algún valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione el rol pertinente para el usuario en la lista y, a continuación, elija **Seleccionar**.

1. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

## <a name="configure-citrix-netscaler-sso"></a>Configuración del inicio de sesión único de Citrix NetScaler

Seleccione un vínculo para ver los pasos para el tipo de autenticación que desea configurar:

- [Configuración del inicio de sesión único de Citrix NetScaler para la autenticación basada en encabezados](#publish-the-web-server)

- [Configuración del inicio de sesión único de Citrix NetScaler para la autenticación basada en Kerberos](citrix-netscaler-tutorial.md#publish-the-web-server)

### <a name="publish-the-web-server"></a>Publicación del servidor web 

Para crear un servidor virtual:

1. Seleccione **Traffic Management** > **Load Balancing** > **Services** (Administración del tráfico > Equilibrio de carga > Servicios).
    
1. Seleccione **Agregar**.

    ![Configuración de Citrix NetScaler: panel Services (Servicios)](./media/header-citrix-netscaler-tutorial/web01.png)

1. Establezca los siguientes valores para el servidor web que ejecuta las aplicaciones:

   * **Nombre del servicio**
   * **IP de servidor/ Servidor existente**
   * **Protocolo**
   * **Puerto**

     ![Panel de configuración de Citrix NetScaler](./media/header-citrix-netscaler-tutorial/web01.png)

### <a name="configure-the-load-balancer"></a>Configuración del equilibrador de carga

Para configurar el equilibrador de carga:

1. Vaya a **Traffic Management** > **Load Balancing** > **Services** (Administración del tráfico > Equilibrio de carga > Servicios).

1. Seleccione **Agregar**.

1. Establezca los valores siguientes tal y como se describe en la captura de pantalla:

    * **Nombre**
    * **Protocolo**
    * **Dirección IP**
    * **Puerto**

1. Seleccione **Aceptar**.

    ![Configuración de Citrix NetScaler: panel Basic Settings (Configuración básica)](./media/header-citrix-netscaler-tutorial/load01.png)

### <a name="bind-the-virtual-server"></a>Enlace del servidor virtual

Para enlazar el equilibrador de carga con el servidor virtual:

1. En el panel **Services and Service Groups** (Servicios y grupos de servicios), seleccione **No Load Balancing Virtual Server Service Binding** (No hay enlace de servicio del servidor virtual de equilibrio de carga).

   ![Configuración de Citrix NetScaler: panel Load Balancing Virtual Server Service Binding (Enlace de servicio del servidor virtual de equilibrio de carga)](./media/header-citrix-netscaler-tutorial/bind01.png)

1. Compruebe la configuración como se muestra en la siguiente captura de pantalla y, a continuación, seleccione **Close** (Cerrar).

   ![Configuración de Citrix NetScaler: comprobación del enlace de servicios de servidor virtual](./media/header-citrix-netscaler-tutorial/bind02.png)

### <a name="bind-the-certificate"></a>Enlace del certificado

Para publicar este servicio como TLS, enlace el certificado de servidor y, a continuación, pruebe la aplicación:

1. En **Certificate** (Certificado), seleccione **No Server Certificate** (Ningún certificado de servidor).

   ![Configuración de Citrix NetScaler: panel Server Certificate (Certificado de servidor)](./media/header-citrix-netscaler-tutorial/bind03.png)

1. Compruebe la configuración como se muestra en la siguiente captura de pantalla y, a continuación, seleccione **Close** (Cerrar).

   ![Configuración de Citrix NetScaler: comprobación del certificado](./media/header-citrix-netscaler-tutorial/bind04.png)

## <a name="citrix-adc-saml-profile"></a>Perfil de SAML de Citrix ADC

Para configurar el perfil de SAML de Citrix ADC, complete las secciones siguientes:

### <a name="create-an-authentication-policy"></a>Creación de una directiva de autenticación

Para crear una directiva de autenticación:

1. Vaya a **Security** > **AAA – Application Traffic** > **Policies** > **Authentication** > **Authentication Policies** (Seguridad > AAA – Tráfico de aplicación > Directivas > Autenticación > Directivas de autenticación).

1. Seleccione **Agregar**.

1. En el panel **Create Authentication Policy** (Crear directiva de autenticación), escriba o seleccione los valores siguientes:

    * **Name**: escriba un nombre para la directiva de autenticación.
    * **Acción**: Escriba **SAML** y seleccione **Add** (Agregar).
    * **Expression** (Expresión):  escriba **true**.     
    
    ![Configuración de Citrix NetScaler: panel Create Authentication Policy (Crear directiva de autenticación)](./media/header-citrix-netscaler-tutorial/policy01.png)

1. Seleccione **Crear**.

### <a name="create-an-authentication-saml-server"></a>Creación de un servidor SAML de autenticación

Para crear un servidor SAML de autenticación, vaya al panel **Create Authentication SAML Server** (Crear servidor SAML de autenticación) y, a continuación, complete los pasos siguientes:

1. En **Name** (Nombre), escriba un nombre para el servidor SAML de autenticación.

1. En **Export SAML Metadata** (Exportar metadatos de SAML):

   1. Active la casilla **Import Metadata** (Importar metadatos).

   1. Escriba la dirección URL de metadatos de federación para la interfaz de usuario de SAML de Azure que haya copiado de Azure Portal.
    
1. En **Issuer Name** (Nombre del emisor), escriba la dirección URL correspondiente.

1. Seleccione **Crear**.

![Configuración de Citrix NetScaler: panel Create Authentication SAML Server (Crear servidor SAML de autenticación)](./media/header-citrix-netscaler-tutorial/server01.png)

### <a name="create-an-authentication-virtual-server"></a>Creación de un servidor virtual de autenticación

Para crear un servidor virtual de autenticación:

1.  Vaya a **Security** > **AAA – Application Traffic** > **Policies** > **Authentication**  > **Authentication Virtual Servers** (Seguridad > AAA - Tráfico de aplicación > Directivas > Autenticación > Servidores virtuales de autenticación).

1.  Seleccione **Add** (Agregar) y, después, complete los pasos siguientes:

    1. En **Name** (Nombre), escriba un nombre para el servidor virtual de autenticación.

    1. Active la casilla **Non-Addressable** (No direccionable).

    1. En **Protocol** (Protocolo), seleccione **SSL**.

    1. Seleccione **Aceptar**.

    ![Configuración de Citrix NetScaler: panel Authentication Virtual Server (Servidor de autenticación virtual)](./media/header-citrix-netscaler-tutorial/server02.png)
    
### <a name="configure-the-authentication-virtual-server-to-use-azure-ad"></a>Configuración del servidor virtual de autenticación para usar Azure AD

Modifique dos secciones para el servidor virtual de autenticación:

1.  En el panel **Advanced Authentication Policies** (Directivas de autenticación avanzadas), seleccione **No Authentication Policy** (Sin directiva de autenticación).

    ![Configuración de Citrix NetScaler: panel Advanced Authentication Policies (Directivas de autenticación avanzadas)](./media/header-citrix-netscaler-tutorial/virtual01.png)

1. En el panel **Policy Binding** (Enlace de directiva), seleccione la directiva de autenticación y, a continuación, seleccione **Bind** (Enlazar).

    ![Configuración de Citrix NetScaler: panel Policy Binding (Enlace de directiva)](./media/header-citrix-netscaler-tutorial/virtual02.png)

1. En el panel **Form Based Virtual Servers** (Servidores virtuales basados en formularios), seleccione **No Load Balancing Virtual Server** (Ningún servidor virtual de equilibrio de carga).

    ![Configuración de Citrix NetScaler: panel Form Based Virtual Servers (Servidores virtuales basados en formularios)](./media/header-citrix-netscaler-tutorial/virtual03.png)

1. En **Authentication FQDN** (FQDN de autenticación), escriba un nombre de dominio completo (obligatorio).

1. Seleccione el servidor virtual de equilibrio de carga que desee proteger con la autenticación de Azure AD.

1. Seleccione **Bind** (Enlace).

    ![Configuración de Citrix NetScaler: panel Load Balancing Virtual Server Binding (Enlace del servidor virtual de equilibrio de carga)](./media/header-citrix-netscaler-tutorial/virtual04.png)

    > [!NOTE]
    > Asegúrese también de seleccionar **Done** (Listo) en el panel **Authentication Virtual Server Configuration** (Configuración del servidor virtual de autenticación).

1. Para comprobar los cambios, en un explorador, vaya a la dirección URL de la aplicación. Debería ver la página de inicio de sesión de inquilino en lugar del acceso no autenticado que habría visto anteriormente.

    ![Configuración de Citrix NetScaler: una página de inicio de sesión en un explorador web](./media/header-citrix-netscaler-tutorial/virtual05.png)

## <a name="configure-citrix-netscaler-sso-for-header-based-authentication"></a>Configuración del inicio de sesión único de Citrix NetScaler para la autenticación basada en encabezados

### <a name="configure-citrix-adc"></a>Configuración de Citrix ADC

Para configurar Citrix ADC para la autenticación basada en encabezados, complete las secciones siguientes.

#### <a name="create-a-rewrite-action"></a>Creación de una acción de reescritura

1. Vaya a **AppExpert** > **Rewrite** > **Rewrite Actions** (AppExpert > Reescritura > Acciones de reescritura).
 
    ![Configuración de Citrix NetScaler: panel Rewrite Actions (Acciones de reescritura)](./media/header-citrix-netscaler-tutorial/header01.png)

1.  Seleccione **Add** (Agregar) y, después, complete los pasos siguientes:

    1. En **Name** (Nombre), escriba una nombre para la acción de reescritura.

    1. En **Type** (Tipo), escriba **INSERT_HTTP_HEADER**.

    1. En **Header Name** (Nombre de encabezado), escriba un nombre de encabezado (en este ejemplo, se usa _SecretID_).

    1. En **Expression** (Expresión), escriba **aaa.USER.ATTRIBUTE("mySecretID")** , donde **mySecretID** es la notificación de SAML de Azure AD enviada a Citrix ADC.

    1. Seleccione **Crear**.

    ![Configuración de Citrix NetScaler: panel Create Rewrite Action (Crear acción de reescritura)](./media/header-citrix-netscaler-tutorial/header02.png)
 
#### <a name="create-a-rewrite-policy"></a>Creación de una directiva de reescritura

1.  Vaya a **AppExpert** > **Rewrite** > **Rewrite Policies** (AppExpert > Reescritura > Directivas de reescritura).
 
    ![Configuración de Citrix NetScaler: panel Rewrite Policies (Directivas de reescritura)](./media/header-citrix-netscaler-tutorial/header03.png)

1.  Seleccione **Add** (Agregar) y, después, complete los pasos siguientes:

    1. En **Name** (Nombre), escriba un nombre para la directiva de reescritura.

    1. En **Action** (Acción), seleccione la acción de reescritura que ha creado en la sección anterior.

    1. En **Expression** (Expresión), escriba **true**.

    1. Seleccione **Crear**.

    ![Configuración de Citrix NetScaler: panel Create Rewrite Policy (Crear directiva de reescritura)](./media/header-citrix-netscaler-tutorial/header04.png)

### <a name="bind-a-rewrite-policy-to-a-virtual-server"></a>Enlace de la directiva de reescritura a un servidor virtual

Para enlazar una directiva de reescritura a un servidor virtual mediante la GUI:

1. Vaya a **Traffic Management** > **Load Balancing** > **Services** (Administración del tráfico > Equilibrio de carga > Servicios).

1. En la lista de servidores virtuales, seleccione el servidor virtual al que desee enlazar la directiva de reescritura y, después, seleccione **Open** (Abrir).

1. En el panel **Load Balancing Virtual Server** (Servidor virtual de equilibrio de carga), en **Advanced Settings** (Configuración avanzada), seleccione **Policies** (Directivas). Todas las directivas configuradas para la instancia de NetScaler aparecen en la lista.
 
    ![Configuración de Citrix NetScaler: panel Load Balancing Virtual Server (Servidor virtual de equilibrio de carga)](./media/header-citrix-netscaler-tutorial/header05.png)

    ![Configuración de Citrix NetScaler: panel Load Balancing Virtual Server (Servidor virtual de equilibrio de carga)](./media/header-citrix-netscaler-tutorial/header06.png)

1.  Active la casilla situada junto al nombre de la directiva que desea enlazar a este servidor virtual.
 
    ![Configuración de Citrix NetScaler: panel Load Balancing Virtual Server Traffic Policy Binding (Enlace de la directiva de tráfico del servidor virtual de equilibrio de carga)](./media/header-citrix-netscaler-tutorial/header08.png)

1. En el cuadro de diálogo **Choose Type** (Elegir tipo):

    1. En **Choose Policy** (Elegir directiva), seleccione **Traffic** (Tráfico).

    1. En **Choose Type** (Elegir tipo), seleccione **Request** (Solicitud).

    ![Configuración de Citrix NetScaler: cuadro de diálogo Policies (Directivas)](./media/header-citrix-netscaler-tutorial/header07.png)

1.  Seleccione **Aceptar**. Un mensaje en la barra de estado indica que la directiva se ha configurado correctamente.

### <a name="modify-the-saml-server-to-extract-attributes-from-a-claim"></a>Modificación del servidor SAML para extraer atributos de una notificación

1.  Vaya a **Security** > **AAA – Application Traffic** > **Policies** > **Authentication** > **Advanced Policies** > **Actions** > **Servers** (Seguridad > AAA – Tráfico de aplicación > Directivas > Autenticación > Directivas avanzadas > Acciones > Servidores).

1.  Seleccione el servidor SAML de autenticación adecuado para la aplicación.
 
    ![Configuración de Citrix NetScaler: panel Configure Authentication SAML Server (Configurar servidor SAML de autenticación)](./media/header-citrix-netscaler-tutorial/header09.png)

1. En el panel **Attributes** (Atributos), especifique los atributos de SAML que desee extraer, separados por comas. En nuestro ejemplo, se especifica el atributo `mySecretID`.
 
    ![Configuración de Citrix NetScaler: panel Attributes (Atributos)](./media/header-citrix-netscaler-tutorial/header10.png)

1. Para comprobar el acceso, en la dirección URL de un explorador, busque el atributo de SAML en **Colección de encabezados**.

    ![Configuración de Citrix NetScaler: colección de encabezados en la dirección URL](./media/header-citrix-netscaler-tutorial/header11.png)

### <a name="create-a-citrix-netscaler-test-user"></a>Creación de un usuario de prueba de Citrix NetScaler

En esta sección, se crea un usuario llamado B.Simon en Citrix NetScaler. Citrix NetScaler admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. El usuario no tiene que hacer nada en esta sección. Si el usuario no existe ya en Citrix NetScaler, se crea uno después de la autenticación.

> [!NOTE]
> Si necesita crear manualmente un usuario, póngase en contacto con el [equipo de soporte técnico de Citrix NetScaler](https://www.citrix.com/contact/technical-support.html).

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el panel de acceso.

Al seleccionar el icono de Citrix NetScaler en el panel de acceso, debería iniciar sesión automáticamente en la instancia de Citrix NetScaler para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Citrix NetScaler con Azure AD](https://aad.portal.azure.com/)

- [Configuración del inicio de sesión único de Citrix NetScaler para la autenticación basada en Kerberos](citrix-netscaler-tutorial.md)
