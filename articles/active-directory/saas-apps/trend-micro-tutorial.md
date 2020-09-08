---
title: 'Tutorial: Integración del inicio de sesión único de Azure AD con Trend Micro Web Security (TMWS)'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Trend Micro Web Security (TMWS).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/21/2020
ms.author: jeedes
ms.openlocfilehash: f2a6598cc28c39719d73be333bd74c24fce9371b
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88551914"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-trend-micro-web-security-tmws"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Trend Micro Web Security (TMWS)

En este tutorial aprenderá a integrar Trend Micro Web Security (TMWS) con Azure Active Directory (Azure AD). Al integrar TMWS con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a TMWS.
* Permitir que los usuarios inicien sesión automáticamente en TMWS con sus cuentas de Azure AD.
* Administrar sus cuentas en una ubicación central: Azure Portal.

Para más información acerca de la integración de aplicaciones SaaS con Azure AD, consulte [Inicio de sesión único en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on).

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesitará lo siguiente:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción a TMWS habilitada para el inicio de sesión único.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* TMWS admite el inicio de sesión único iniciado por SP.
* Una vez configurado TMWS, puede aplicar el control de sesión, que protege la filtración e infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. Para obtener información sobre cómo aplicar el control de sesión mediante Microsoft Cloud App Security, consulte [Incorporación e implementación de Control de aplicaciones de acceso condicional para cualquier aplicación](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="add-tmws-from-the-gallery"></a>Incorporación de TMWS desde la galería

Para configurar la integración de TMWS en Azure AD, deberá agregar dicha solución desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta Microsoft personal, profesional o educativa.
1. En el panel izquierdo, seleccione el servicio **Azure Active Directory**.
1. Seleccione **Aplicaciones empresariales** y, después, **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Trend Micro Web Security (TMWS)** en el cuadro de búsqueda.
1. Seleccione **Trend Micro Web Security (TMWS)** en los resultados de búsqueda y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-tmws"></a>Configuración y prueba del inicio de sesión único de Azure AD para TMWS

Configure y pruebe el inicio de sesión único de Azure AD con TMWS mediante un usuario de prueba llamado B.Simon. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de TMWS.

Completará estos pasos básicos para configurar y probar el inicio de sesión único de Azure AD con TMWS:

1. [Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso), para que los usuarios puedan utilizar esta característica.
    1. [Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user), para probar el inicio de sesión único de Azure AD.
    1. [Concesión de acceso a TMWS al usuario de prueba de Azure AD](#grant-the-azure-ad-test-user-access-to-tmws).
    1. [Configuración de las opciones de sincronización de usuarios y grupos en Azure AD](#configure-user-and-group-synchronization-settings-in-azure-ad).
1. [Configuración del inicio de sesión único en TMWS](#configure-tmws-sso) en la aplicación.
1. [Prueba de SSO](#test-sso), para comprobar que la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Trend Micro Web Security (TMWS)** , busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, seleccione el botón de lápiz de **Configuración básica de SAML** para editar la configuración:

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de las siguientes casillas:

    a. En el cuadro **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón:

    `https://auth.iws-hybrid.trendmicro.com/([0-9a-f]{16})`

    b. En el cuadro **URL de respuesta**, escriba esta dirección URL:

    `https://auth.iws-hybrid.trendmicro.com/simplesaml/module.php/saml/sp/saml2-acs.php/ics-sp`

    > [!NOTE]
    > El valor del identificador del paso anterior no es el valor que debe indicar. Debe utilizar el identificador real. Puede obtener este valor en la sección **Configuración del proveedor de servicios para el portal de administración de Azure** de la página **Método de autenticación** de Azure AD en **Administración > Servicios de directorio**.

1. TMWS espera las aserciones de SAML en un formato específico, por lo que debe agregar asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. En la siguiente captura se muestran los atributos predeterminados:

    ![Atributos predeterminados](common/default-attributes.png)

1. Además de los atributos de la captura de pantalla anterior, TMWS espera que se pasen dos atributos más en la respuesta de SAML. Estos atributos se muestran en la tabla siguiente. Los atributos se rellenan previamente, pero puede cambiarlos para adaptarlos a sus requisitos.
    
    | Nombre | Atributo de origen|
    | --------------- | --------- |
    | sAMAccountName | user.onpremisessamaccountname |
    | uPN | user.userprincipalname |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** . Seleccione el vínculo **Descargar** junto a este nombre de certificado para descargar el certificado y guardarlo en el equipo:

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Trend Micro Web Security (TMWS)** , copie las direcciones URL adecuadas según sus necesidades:

    ![Copia de las direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, se crea un usuario llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**. Seleccione **Usuarios** y, a continuación, seleccione **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el cuadro **Nombre**, escriba `B.Simon`.  
   1. En el cuadro de texto **Nombre de usuario**, escriba ***nombreDeUsuario *@* dominioDeEmpresa *.* extensión***. Por ejemplo, `B.Simon@contoso.com`.
   1. Seleccione **Mostrar contraseña** y, a continuación, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Seleccione **Crear**.

### <a name="grant-the-azure-ad-test-user-access-to-tmws"></a>Concesión de acceso a TMWS al usuario de prueba de Azure AD

En esta sección va a permitir que B.Simon acceda a TMWS mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales** y, a continuación, seleccione **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Trend Micro Web Security (TMWS)** .
1. En la sección **Administrar** de la página de información general de la aplicación, seleccione **Usuarios y grupos**:

   ![Seleccionar Usuarios y grupos](common/users-groups-blade.png)

1. Seleccione **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.

    ![Selección de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** en la lista **Usuarios** y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

### <a name="configure-user-and-group-synchronization-settings-in-azure-ad"></a>Configuración de las opciones de sincronización de usuarios y grupos en Azure AD

1. En el panel izquierdo, seleccione **Azure Active Directory**.

1. En **Administrar**, seleccione **Registros de aplicaciones** y, a continuación, seleccione la nueva aplicación empresarial bajo **Todas las aplicaciones**.

1. En **Administrar**, seleccione **Certificados y secretos**.

1. En el área **Secretos de cliente**, seleccione **Nuevo secreto de cliente**.

1. En la **pantalla Add a client secret** (Agregar un secreto de cliente) puede agregar una descripción y seleccionar un período de expiración para el secreto de cliente; a continuación, haga clic en **Agregar**. El nuevo secreto de cliente aparece en el área **Secretos de cliente**.

1. Anote el valor de secreto de cliente. Más adelante, lo escribirá en TMWS.

1. En **Administrar**, seleccione **Permisos de API**. 

1. En la ventana **Permisos de API**, seleccione **Agregar un permiso**.

1. En la pestaña **API de Microsoft** de la pantalla **Solicitud de permisos de API**, seleccione **Microsoft Graph** y, a continuación, **Permisos de la aplicación**.

1. Busque y agregue estos permisos: 

    * Group.Read.All
    * User.Read.All

1. Seleccione **Agregar permisos**. Aparece un mensaje para confirmar que la configuración se guardó. Los nuevos permisos aparecen en la ventana **Permisos de API**.

1. En el área **Otorgar consentimiento**, seleccione **Conceder consentimiento de administrador para *la cuenta de administrador* (directorio predeterminado)** y, a continuación, seleccione **Sí**. Aparece un mensaje para confirmar que se ha otorgado el consentimiento del administrador para los permisos solicitados.

1. Seleccione **Información general**. 

1. Tome nota de los valores de **Id. de aplicación (cliente)** e **Id. de directorio (inquilino)** que aparecen en el panel derecho. Más adelante, escribirá esa información en TMWS. También puede seleccionar **Nombres de dominio personalizados** en **Azure Active Directory > Administrar** y tomar nota del nombre de dominio que aparece en el panel derecho.

## <a name="configure-tmws-sso"></a>Configuración del inicio de sesión único en TMWS

Siga estos pasos para configurar el inicio de sesión único de TMWS en la aplicación.

1. Inicie sesión en la consola de administración de TMWS y vaya a **Administration** > **USERS & AUTHENTICATION** > **Directory Services** (Administración > USUARIOS Y AUTENTICACIÓN > Servicios de directorio).

1. Seleccione **here** (aquí) en el área superior de la pantalla.

1. En la página **Authentication Method** (Método de autenticación), seleccione **Azure AD**.

1. Haga clic en **On** (Activado) o en **Off** (Desactivado) para decidir si se permite que los usuarios de Azure AD de su organización visiten sitios web con TMWS si sus datos no están sincronizados en esta aplicación.

    > [!NOTE]
    > Los usuarios que no se han sincronizado desde Azure AD solo se pueden autenticar mediante puertas de enlace de TMWS conocidas o el puerto dedicado para su organización.

1. En la página **Identity Provider Settings** (Configuración del proveedor de identidades), siga estos pasos:

    a. En el cuadro **Service URL** (Dirección URL del servicio), escriba el valor de **URL de inicio de sesión** que copió de Azure Portal.

    b. En el campo **Logon name attribute** (Atributo de nombre de inicio de sesión), escriba el **nombre de la notificación de usuario** con el atributo de origen **user.onpremisessamaccountname** de Azure Portal.

    c. En el cuadro **Public SSL certificate** (Certificado SSL público), use el **Certificado (Base64)** descargado de Azure Portal.

1. En la sección **Synchronization Settings** (Configuración de sincronización), siga estos pasos:

    a. En el cuadro **Tenant** (Inquilino), use el valor de **Id. de directorio (inquilino)** o de **Nombre de dominio personalizado** de Azure Portal.

    b. En el cuadro **Application ID** (Id. de aplicación), escriba el valor de **Id. de aplicación (cliente)** de Azure Portal.

    c. En el cuadro **Client secret** (Secreto de cliente), utilice el valor de **Secreto de cliente** de Azure Portal.

    d. Seleccione un valor de **Synchronization schedule** (Programación de sincronización) para realizar la sincronización con Azure AD de forma manual o según una programación. Si selecciona **Manually** (Manualmente), no olvide volver a la página **Directory Services** (Servicios de directorio) y realizar la sincronización manual siempre que haya cambios en la información de usuario de Active Directory, para que la información de TMWS siga estando actualizada.

    e. Seleccione **Test Connection** (Probar conexión) para comprobar si el servicio de Azure AD puede conectarse correctamente.
    
    f. Seleccione **Guardar**.
 
 > [!NOTE]
 > Para obtener más información sobre cómo configurar TMWS con Azure AD, consulte la página sobre [configuración de Azure AD en TMWS](https://docs.trendmicro.com/en-us/enterprise/trend-micro-web-security-online-help/administration_001/directory-services/azure-active-directo/configuring-azure-ad.aspx).

## <a name="test-sso"></a>Prueba de SSO 

Después de configurar correctamente el servicio Azure AD y especificar Azure AD como método de autenticación de usuarios, puede iniciar sesión en el servidor proxy de TMWS para verificar la configuración. Después de que el inicio de sesión de Azure AD verifique la cuenta, puede visitar Internet.

> [!NOTE]
> TMWS no admite la prueba del inicio de sesión único desde el portal de Azure AD, en **Información general** > **Inicio de sesión único** > **Configuración del inicio de sesión único con SAML** > **Prueba** de la nueva aplicación empresarial.

1. Borre todas las cookies del explorador y, a continuación, reinícielo. 

1. Apunte el explorador al servidor proxy de TMWS. Para más información, consulte [Reenvío de tráfico mediante archivos PAC](https://docs.trendmicro.com/en-us/enterprise/trend-micro-web-security-online-help/administration_001/pac-files/traffic-forwarding-u.aspx#GUID-A4A83827-7A29-4596-B866-01ACCEDCC36B).

1. Visite cualquier sitio web de Internet. TMWS le dirigirá al portal cautivo de TMWS.

1. Especifique una cuenta de Active Directory (formato: *dominio*\\*nombreDeCuentaDeSam* o *nombreDeCuentaDeSam*@*dominio*), dirección de correo electrónico o UPN y, a continuación, seleccione **iniciar sesión**. TMWS le envía a la ventana de inicio de sesión de Azure AD.

1. En la ventana de inicio de sesión de Azure AD, escriba las credenciales de la cuenta de Azure AD. Con esto debe haber iniciado sesión en TMWS.

## <a name="additional-resources"></a>Recursos adicionales

- [Tutoriales para la integración de aplicaciones SaaS con Azure Active Directory ](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Trend Micro Web Security con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Trend Micro Web Security con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

