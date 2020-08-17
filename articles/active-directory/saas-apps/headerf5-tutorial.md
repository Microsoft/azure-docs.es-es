---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con F5 | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y F5.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 59a87abb-1ec1-4438-be07-5b115676115f
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 11/19/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 341be30c30f7b4a2a53f70f18e1c2a3a30de1cb4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87034512"
---
# <a name="tutorial-configure-single-sign-on-sso-between-azure-active-directory-and-f5"></a>Tutorial: Configuración del inicio de sesión único entre Azure Active Directory y F5

En este tutorial, obtendrá información sobre cómo integrar F5 con Azure Active Directory (Azure AD). Al integrar F5 con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a F5.
* Permitir que los usuarios inicien sesión automáticamente en F5 con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información acerca de la integración de aplicaciones SaaS con el inicio de sesión único en Azure AD, consulte [Inicio de sesión único en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on).

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).

* Una suscripción habilitada para el inicio de sesión único (SSO) en F5.

* La implementación de la solución conjunta requiere la licencia siguiente:

    * F5 BIG-IP® Best bundle (o) 

    * Licencia independiente de F5 BIG-IP Access Policy Manager™ (APM). 

    * Licencia del complemento F5 BIG-IP Access Policy Manager™ (APM) en una instalación de F5 BIG-IP® Local Traffic Manager™ (LTM) ya existente.

    * Además de la licencia anterior, el sistema F5 también puede tener licencia para: 

        * Una suscripción de filtrado de direcciones URL para usar la base de datos de categorías de URL.

        * Una suscripción a F5 IP Intelligence para detectar y bloquear a atacantes conocidos y tráfico malintencionado.
     
        * Un módulo de seguridad de hardware (HSM) de red para proteger y administrar claves digitales para la autenticación segura.

* El sistema F5 BIG-IP se aprovisiona con módulos de APM (LTM es opcional).

* Aunque es opcional, es muy recomendable implementar los sistemas F5 en un [grupo de dispositivos de sincronización/conmutación por error](https://techdocs.f5.com/content/techdocs/en-us/bigip-14-1-0/big-ip-device-service-clustering-administration-14-1-0.html) (S/F DG), que incluye el par activo en espera, con una dirección IP flotante para la alta disponibilidad (HA). Se puede lograr una mayor redundancia de la interfaz mediante el protocolo de control de agregación de vínculos (LACP). LACP administra las interfaces físicas conectadas como una sola interfaz virtual (grupo agregado) y detecta cualquier error de interfaz dentro del grupo.

* En el caso de las aplicaciones Kerberos, una cuenta de servicio de AD local para la delegación restringida.  Consulte la [Documentación de F5](https://support.f5.com/csp/article/K43063049) para crear una cuenta de delegación de AD.

## <a name="access-guided-configuration"></a>Configuración de acceso guiado

* La configuración de acceso guiado es compatible con F5 TMOS versión 13.1.0.8 y posteriores. Si el sistema BIG-IP ejecuta una versión anterior a la 13.1.0.8, consulte la sección **Configuración avanzada**.

* La configuración de acceso guiado presenta una experiencia de usuario completamente nueva y simplificada. Esta arquitectura basada en el flujo de trabajo proporciona pasos de configuración intuitivos y reentrantes adaptados a la topología seleccionada.

* Antes de continuar con la configuración, actualice la configuración guiada; para ello, descargue el último paquete de casos de uso desde [downloads.f5.com](https://login.f5.com/resource/login.jsp?ctx=719748). Para actualizar, siga el procedimiento siguiente.

    >[!NOTE]
    >Las capturas de pantalla siguientes corresponden a la versión más reciente (BIG-IP 15.0 con AGC versión 5.0). Los pasos de configuración siguientes son válidos para este caso de uso desde la versión 13.1.0.8 hasta la última versión de BIG-IP.

1. En la interfaz de usuario web de F5 BIG-IP, haga clic en **Access >> Guided Configuration** (Acceso >> Configuración guiada).

1. En la página **Guided Configuration** (Configuración guiada), haga clic en **Upgrade Guided Configuration** (Actualizar configuración guiada) en la esquina superior izquierda.

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure14.png) 

1. En la pantalla emergente Upgrade Guided Configuration (Actualizar configuración guiada), seleccione **Choose File** (Elegir archivo) para cargar el paquete de casos de uso descargado y haga clic en el botón **Upload and Install** (Cargar e instalar).

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure15.png) 

1. Cuando se complete la actualización, haga clic en el botón **Continue** (Continuar).

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure16.png)

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* El inicio de sesión único de F5 se puede configurar de tres maneras diferentes:

- [Configuración del inicio de sesión único de F5 para la aplicación basada en encabezados](#configure-f5-single-sign-on-for-header-based-application)

- [Configuración del inicio de sesión único de F5 para la aplicación Kerberos](kerbf5-tutorial.md)

- [Configuración del inicio de sesión único de F5 para la aplicación Kerberos avanzada](advance-kerbf5-tutorial.md)

### <a name="key-authentication-scenarios"></a>Escenarios de autenticación de claves

* Además de la compatibilidad de la integración nativa de Azure Active Directory con protocolos de autenticación modernos, como Open ID Connect, SAML y WS-Fed, F5 amplía el acceso seguro para aplicaciones de autenticación heredadas para el acceso interno y externo con Azure AD, lo que permite escenarios modernos (por ejemplo, el acceso sin contraseña) para estas aplicaciones. Esto incluye:

* Aplicaciones de autenticación basadas en encabezados

* Aplicaciones de autenticación Kerberos

* Autenticación anónima o aplicaciones sin autenticación integrada

* Aplicaciones de autenticación NTLM (protección con dos preguntas para el usuario)

* Aplicaciones basadas en formularios (protección con dos preguntas para el usuario)

## <a name="adding-f5-from-the-gallery"></a>Adición de F5 desde la galería

Para configurar la integración de F5 en Azure AD, será preciso agregar F5 desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **F5** en el cuadro de búsqueda.
1. Seleccione **F5** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-f5"></a>Configuración y prueba del inicio de sesión único de Azure AD para F5

Configure y pruebe el inicio de sesión único de Azure AD con F5 mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de F5.

Para configurar y probar el inicio de sesión único de Azure AD con F5, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de F5](#configure-f5-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de F5](#create-f5-test-user)** , para tener un homólogo de B.Simon en F5 que esté vinculado a su representación en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **F5**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<YourCustomFQDN>.f5.com/`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<YourCustomFQDN>.f5.com/`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<YourCustomFQDN>.f5.com/`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de F5](https://support.f5.com/csp/knowledge-center/software/BIG-IP?module=BIG-IP%20APM45) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y **Certificado (Base 64)** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Set up F5** (Configurar F5), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a F5 mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **F5**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.
1. Haga clic en **Acceso condicional**.
1. Haga clic en **Nueva directiva**.
1. Ahora puede ver la aplicación F5 como un recurso para la directiva de CA y aplicar cualquier acceso condicional, como la autenticación multifactor, el control de acceso basado en dispositivo o la directiva de protección de identidades.

## <a name="configure-f5-sso"></a>Configuración del inicio de sesión único de F5

- [Configuración del inicio de sesión único de F5 para la aplicación Kerberos](kerbf5-tutorial.md)

- [Configuración del inicio de sesión único de F5 para la aplicación Kerberos avanzada](advance-kerbf5-tutorial.md)

### <a name="configure-f5-single-sign-on-for-header-based-application"></a>Configuración del inicio de sesión único de F5 para la aplicación basada en encabezados

### <a name="guided-configuration"></a>Configuración guiada

1. Abra una nueva ventana del explorador web e inicie sesión en el sitio de empresa de F5 (basada en encabezados) como administrador. Luego, realice los pasos siguientes:

1. Vaya a **System > Certificate Management > Traffic Certificate Management > SSL Certificate List** (Sistema > Administración de certificados > Administración de certificados de tráfico > Lista de certificados SSL). Seleccione **Import** (Importar) en la esquina derecha. Especifique un valor en **Certificate Name** (Nombre de certificado) (se hará referencia a él más adelante en la configuración). En **Certificate Source** (Origen del certificado), seleccione Upload file (Cargar archivo) y especifique el certificado descargado de Azure al configurar el inicio de sesión único de SAML. Haga clic en **Import**.

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure12.png)
 
1. Además, necesitará el **Certificado SSL para el nombre de host de la aplicación. Vaya a System > Certificate Management > Traffic Certificate Management > SSL Certificate List** (Sistema > Administración de certificados > Administración de certificados de tráfico > Lista de certificados SSL). Seleccione **Import** (Importar) en la esquina derecha. El campo **Import Type** (Tipo de importación) será **PKCS 12 (IIS)** . Especifique un valor para **Key Name** (Nombre de clave) (se hará referencia a él más adelante en la configuración) y especifique el archivo PFX. Especifique el campo **Password** (Contraseña) del archivo PFX. Haga clic en **Import**.

    >[!NOTE]
    >En el ejemplo, el nombre de la aplicación es `Headerapp.superdemo.live` y se usa un certificado comodín; el nombre de clave es `WildCard-SuperDemo.live`.

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure13.png)

1. Usaremos la experiencia guiada para configurar la federación de Azure AD y el acceso a las aplicaciones. Vaya a la página **Main** (Principal) de F5 BIG-IP y seleccione **Access > Guided Configuration > Federation > SAML Service Provider** (Acceso > Configuración guiada > Federación > Proveedor de servicios SAML). Haga clic en **Next** (Siguiente) y, a continuación, en **Next** (Siguiente) para comenzar la configuración.

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure01.png)

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure02.png)
 
1. Proporcione un valor para **Configuration Name** (Nombre de configuración). Especifique el valor de **Entity ID** (Identificador de entidad) (el mismo que configuró en la configuración de la aplicación de Azure AD). Especifique el campo **Host name** (Nombre de host). Agregue un valor para **Description** (Descripción) como referencia. Acepte las entradas predeterminadas restantes y seleccione y, a continuación, haga clic en **Save & Next** (Guardar y siguiente).

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure03.png) 

1. En este ejemplo, vamos a crear un nuevo servidor virtual como 192.168.30.20 con el puerto 443. Especifique la dirección IP del servidor virtual en el campo **Destination Address** (Dirección de destino). Seleccione el campo **SSL Profile** (Perfil SSL) del cliente y seleccione Create new (Crear nuevo). Especifique el certificado de aplicación cargado previamente (el certificado comodín en este ejemplo) y la clave asociada y, a continuación, haga clic en **Save & Next** (Guardar y siguiente).

    >[!NOTE]
    >En este ejemplo, el servidor web interno se ejecuta en el puerto 888 y queremos publicarlo en el 443.

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure04.png) 

1. En **Select method to configure your IdP connector** (Seleccionar el método para configurar el conector del proveedor de identidades), especifique Metadata (Metadatos), haga clic en Choose File (Elegir archivo) y cargue el archivo XML de metadatos que descargó antes desde Azure AD. Especifique un campo **Name** (Nombre) único para el conector del proveedor de identidades de SAML. Elija el **Certificado de firma de metadatos** que cargó anteriormente. Haga clic en **Save & Next** (Guardar y siguiente).

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure05.png)
 
1. En **Select a Pool** (Seleccionar un grupo), especifique **Create New** (Crear nuevo) (también puede seleccionar un grupo que ya existe). Deje los otros valores como predeterminados. En Pool Servers (Servidores de grupo), escriba la dirección IP en **IP Address/Node Name** (Dirección IP/Nombre de nodo). Especifique el campo **Port** (Puerto). Haga clic en **Save & Next** (Guardar y siguiente).

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure06.png)

1. En la pantalla Single Sign-On Settings (Configuración de inicio de sesión único), seleccione **Enable Single Sign-On** (Habilitar inicio de sesión único). En Selected Single Sign-On Type (Tipo de inicio de sesión único seleccionado), elija **HTTP header-based** (Basado en encabezado HTTP). Reemplace **session.saml.last.Identity** por **session.saml.last.attr.name.Identity** en Username Source (esta variable se establece mediante la asignación de notificaciones en Azure AD). En SSO Headers (Encabezados de SSO).

    * **Header Name (Nombre de encabezado): MyAuthorization**

    * **Header Value (Valor de encabezado): %{session.saml.last.attr.name.Identity}**

    * Haga clic en **Save & Next** (Guardar y siguiente).

    Consulte el Apéndice para obtener una lista completa de las variables y los valores. Puede agregar más encabezados según sea necesario.

    >[!NOTE]
    >Account Name (Nombre de cuenta) es la cuenta de delegación de F5 creada (consulte la documentación de F5).

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure07.png) 

1. Para los fines de esta guía, se omitirán las comprobaciones del punto de conexión.  Consulte la documentación de F5 para más información. Seleccione **Save & Next** (Guardar y siguiente).

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure08.png)

1. Acepte los valores predeterminados y haga clic en **Save & Next** (Guardar y siguiente). Consulte la documentación de F5 para más información sobre la configuración de la administración de sesiones de SAML.

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure09.png)

1. Revise la pantalla de resumen y seleccione **Deploy** (Implementar) para configurar BIG-IP. Haga clic en **Finish** (Finalizar).

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure10.png)

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure11.png)

## <a name="advanced-configuration"></a>Configuración avanzada

Esta sección está pensada para emplearse si no puede usar la Configuración guiada o si desea agregar o modificar parámetros adicionales. Necesitará el Certificado TLS/SSL para el nombre de host de la aplicación.

1. Vaya a **System > Certificate Management > Traffic Certificate Management > SSL Certificate List** (Sistema > Administración de certificados > Administración de certificados de tráfico > Lista de certificados SSL). Seleccione **Import** (Importar) en la esquina derecha. El campo **Import Type** (Tipo de importación) será **PKCS 12 (IIS)** . Especifique un valor para **Key Name** (Nombre de clave) (se hará referencia a él más adelante en la configuración) y especifique el archivo PFX. Especifique el campo **Password** (Contraseña) del archivo PFX. Haga clic en **Import**.

    >[!NOTE]
    >En el ejemplo, el nombre de la aplicación es `Headerapp.superdemo.live` y se usa un certificado comodín; el nombre de clave es `WildCard-SuperDemo.live`.
  
    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure17.png)

### <a name="adding-a-new-web-server-to-bigip-f5"></a>Adición de un nuevo servidor web a BigIP-F5

1. Haga clic en **Main > IApps > Application Services > Application > Create** (Principal > IApps > Servicios de aplicación > Aplicación > Crear).

1. Proporcione un valor para **Name** (Nombre) y, en **Template** (Plantilla), elija **f5.http**.
 
    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure18.png)

1. En este caso, publicaremos HeaderApp2 externamente como HTTPS, para **how should the BIG-IP system handle SSL Traffic** (¿Cómo debe controlar el sistema de BIG-IP el tráfico SSL?) se especifica **Terminate SSL from Client, Plaintext to servers (SSL Offload)** (Finalizar SSL del cliente y texto no cifrado en los servidores [descarga SSL]). Especifique el certificado y la clave en los campos **Which SSL certificate do you want to use?** (¿Qué certificado SSL desea usar?) y **Which SSL private key do you want to use?** (¿Qué clave privada SSL desea usar?). Especifique la dirección IP del servidor virtual en **What IP Address do you want to use for the Virtual Server?** (¿Qué dirección IP desea usar para el servidor virtual?). 

    * **Especificar otros detalles**

        * FQDN  

        * Especifique un grupo de aplicaciones existente o cree uno nuevo.

        * Si va a crear un nuevo servidor de aplicaciones, especifique los campos **internal IP Address** (Dirección IP interna) y **port number** (Número de puerto).

        ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure19.png) 

1. Haga clic en **Finished** (Finalizado).

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure20.png) 

1. Asegúrese de que se pueden modificar las propiedades de la aplicación. Haga clic en **Main > IApps > Application Services: Applications >> HeaderApp2** (Principal > IApps > Servicios de aplicación: Aplicaciones > HeaderApp2). Desactive **Strict Updates** (Actualizaciones estrictas) (modificaremos algunos valores fuera de la GUI). Haga clic en el botón **Update** (Actualizar).

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure21.png) 

1. En este momento, debería poder examinar el servidor virtual.

### <a name="configuring-f5-as-sp-and-azure-as-idp"></a>Configuración de F5 como SP y Azure como IDP

1.  Haga clic en **Access > Federation> SAML Service Provider > Local SP Service > (Acceso > Federación > Proveedor de servicios de SAML > Servicio SP local) y haga clic en Create (Crear) o el signo +** .

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure22.png)

1. Especifique los detalles del servicio del proveedor de servicios. Especifique el campo **Name** (Nombre) que representa la configuración de SP de F5. Especifique el campo **Entity ID** (Identificador de entidad) (suele ser la dirección URL de la aplicación).

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure23.png)

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure24.png)

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure25.png)

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure26.png)

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure27.png)

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure28.png)

### <a name="create-idp-connector"></a>Creación del conector del proveedor de identidades

1. Haga clic en el botón **Bind/Unbind IdP Connectors** (Enlazar/desenlazar conectores del IdP), seleccione **Create New IdP Connector** (Crear nuevo conector del IdP), elija la opción **Metadata** (Metadatos) y, a continuación, realice los pasos siguientes:
 
    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure29.png)

    a. Vaya al archivo metadata.xml descargado de Azure AD y especifique un valor para **Identity Provider Name** (Nombre del proveedor de identidades).

    b. Haga clic en **Ok** (Aceptar).

    c. Se crea el conector y el certificado está listo automáticamente desde el archivo XML de metadatos.
    
    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure30.png)

    d. Configure F5BIG-IP para que envíe todas las solicitudes a Azure AD.

    e. Haga clic en **Add New Row** (Agregar nueva fila), elija **AzureIDP** (tal y como se creó en los pasos anteriores), especifique 

    f. **Matching Source   =  %{session.server.landinguri}** 

    g. **Matching Value     = /** *

    h. Haga clic en **Update** (Actualizar).

    i. Haga clic en **Aceptar**

    j. **Se completó la configuración del IDP de SAML**
    
    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure31.png)

### <a name="configure-f5-policy-to-redirect-users-to-azure-saml-idp"></a>Configuración de la directiva de F5 para redirigir a los usuarios al IDP de SAML de Azure

1. Para configurar la directiva de F5 para redirigir a los usuarios al IDP de SAML de Azure, realice los pasos siguientes:

    a. Haga clic en **Main > Access > Profile/Policies > Access Profiles** (Principal > Acceso > Perfiles/Directivas > Perfiles de acceso).

    b. Haga clic en el botón **Crear**.

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure32.png)
 
    c. Especifique un valor para **Name** (Nombre) (en el ejemplo, HeaderAppAzureSAMLPolicy).

    d. Puede personalizar otras opciones de configuración. Consulte la documentación de F5.

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure33.png)

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure34.png) 

    e. Haga clic en **Finished** (Finalizado).

    f. Una vez completada la creación de la directiva, haga clic en la directiva y vaya a la pestaña **Access Policy** (Directiva de acceso).

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure35.png)
 
    g. Haga clic en **Visual Policy editor** (Editor visual de directivas) y edite el vínculo **Access Policy for Profile** (Directiva de acceso del perfil).

    h. Haga clic en el signo + en el editor visual de directivas y elija **SAML Auth** (Autenticación de SAML).

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure36.png)

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure37.png)
 
    i. Haga clic en **Add Item** (Agregar elemento).

    j. En **Properties** (Propiedades), especifique un valor para **Name** (Nombre) y en **AAA Server** (Servidor de AAA), seleccione el SP configurado previamente y haga clic en **Save** (Guardar).
 
    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure38.png)

    k. La directiva básica está lista y puede personalizar la directiva para incorporar almacenes de orígenes y atributos adicionales.

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure39.png)
 
    l. Asegúrese de hacer clic en el vínculo **Apply Access Policy** (Aplicar directiva de acceso) en la parte superior.

### <a name="apply-access-profile-to-the-virtual-server"></a>Aplicación del perfil de acceso al servidor virtual

1. Asigne el perfil de acceso al servidor virtual con el fin de que F5 BIG-IP APM aplique la configuración del perfil al tráfico entrante y ejecute la directiva de acceso definida previamente.

    a. Haga clic en **Main** > **Local Traffic** > **Virtual Servers** (Principal > Tráfico local > Servidores virtuales).

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure40.png)
 
    b. Haga clic en el servidor virtual, desplácese a la sección **Access Policy** (Directiva de acceso) en la lista desplegable **Access Profile** (Perfil de acceso) y seleccione la directiva de SAML creada (en el ejemplo, HeaderAppAzureSAMLPolicy).

    c. Haga clic en **Update** (Actualizar).
 
    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure41.png)

    d. Cree una regla iRule® de F5 BIG-IP para extraer los atributos de SAML personalizados de la aserción entrante y pasarlos como encabezados HTTP a la aplicación de prueba de back-end. Haga clic en **Main > Local Traffic > iRules > iRule List (Principal > Tráfico local > iRules > Lista de iRules) y haga clic en Create (Crear)** .

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure42.png)
 
    e. Pegue el texto siguiente de la regla iRule de F5 BIG-IP en la ventana de definición.

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure43.png)
 
    when RULE_INIT {  set static::debug 0  }  when ACCESS_ACL_ALLOWED {

    set AZUREAD_USERNAME [ACCESS::session data get "session.saml.last.attr.name. http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name"] if { $static::debug } { log local0. "AZUREAD_USERNAME = $AZUREAD_USERNAME" } if { !([HTTP::header exists "AZUREAD_USERNAME"]) } { HTTP::header insert "AZUREAD_USERNAME" $AZUREAD_USERNAME }

    set AZUREAD_DISPLAYNAME [ACCESS::session data get "session.saml.last.attr.name. http://schemas.microsoft.com/identity/claims/displayname"] if { $static::debug } { log local0. "AZUREAD_DISPLAYNAME = $AZUREAD_DISPLAYNAME" } if { !([HTTP::header exists "AZUREAD_DISPLAYNAME"]) } { HTTP::header insert "AZUREAD_DISPLAYNAME" $AZUREAD_DISPLAYNAME }

    set AZUREAD_EMAILADDRESS [ACCESS::session data get "session.saml.last.attr.name. http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress"] if { $static::debug } { log local0. "AZUREAD_EMAILADDRESS = $AZUREAD_EMAILADDRESS" } if { !([HTTP::header exists "AZUREAD_EMAILADDRESS"]) } { HTTP::header insert "AZUREAD_EMAILADDRESS" $AZUREAD_EMAILADDRESS }}

    **Salida de ejemplo a continuación**

    ![Configuración de F5 (basada en encabezados)](./media/headerf5-tutorial/configure44.png)
 
### <a name="create-f5-test-user"></a>Creación de un usuario de prueba de F5

En esta sección, creará un usuario llamado B.Simon en F5. Trabaje con el  [equipo de soporte técnico del cliente de F5](https://support.f5.com/csp/knowledge-center/software/BIG-IP?module=BIG-IP%20APM45) para agregar los usuarios a la plataforma de F5. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único. 

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de F5 en el panel de acceso, debería iniciar sesión automáticamente en la versión de F5 para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prueba de F5 con Azure AD](https://aad.portal.azure.com/)

- [Configuración del inicio de sesión único de F5 para la aplicación Kerberos](kerbf5-tutorial.md)

- [Configuración del inicio de sesión único de F5 para la aplicación Kerberos avanzada](advance-kerbf5-tutorial.md)

