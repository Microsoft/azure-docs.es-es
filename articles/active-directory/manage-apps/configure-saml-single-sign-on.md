---
title: Configuración del inicio de sesión único (SSO) basado en SAML para aplicaciones en Azure AD
description: Configuración del inicio de sesión único (SSO) basado en SAML para aplicaciones en Azure AD
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: how-to
ms.workload: identity
ms.date: 07/28/2020
ms.author: kenwith
ms.reviewer: arvinh,luleon
ms.openlocfilehash: b506d56f8aff2204c705ae8685f475654c1b1705
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88640487"
---
# <a name="configure-saml-based-single-sign-on"></a>Configuración del inicio de sesión único basado en SAML

En la [serie de guías de inicio rápido](view-applications-portal.md) sobre la administración de aplicaciones, aprendió a usar Azure AD como proveedor de identidades (IdP) para una aplicación. Este artículo explica con más detalle la opción basada en SAML para el inicio de sesión único. 


## <a name="before-you-begin"></a>Antes de empezar

El uso de Azure AD como proveedor de identidades (IdP) y la configuración del inicio de sesión único (SSO) pueden ser algo sencillo o complejo en función de la aplicación utilizada. Algunas aplicaciones se pueden configurar con pocos pasos. Otras exigen una configuración detallada. Para empezar rápidamente, consulte la [serie de guías de inicio rápido](view-applications-portal.md) sobre la administración de aplicaciones. Si la aplicación que va a añadir es sencilla, probablemente no necesite leer este artículo. Si la aplicación que va a añadir requiere una configuración personalizada para SSO basado en SAML, este artículo es para usted.

En la [serie de guías de inicio rápido](view-applications-portal.md), hay un artículo sobre la configuración del inicio de sesión único. En el artículo, descubrirá cómo acceder a la página de configuración de SAML para una aplicación. La página de configuración de SAML consta de cinco secciones. Las secciones se abordan en detalle en este artículo.

> [!IMPORTANT] 
> Hay escenarios en los que la opción **Inicio de sesión único** no estará disponible al desplazarse por una aplicación en las **aplicaciones empresariales**. 
>
> Por ejemplo, si la aplicación se registró mediante **Registros de aplicaciones**, la característica de inicio de sesión único estará configurada para usar OAuth de OIDC de forma predeterminada. En este caso, la opción de **Inicio de sesión único** no se mostrará en la navegación en **Aplicaciones empresariales**. Cuando use **Registros de aplicaciones** para agregar su aplicación personalizada, configure las opciones en el archivo de manifiesto. Para obtener más información sobre el archivo de manifiesto, consulte [Manifiesto de aplicación de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/reference-app-manifest). Para obtener más información sobre los estándares de SSO, consulte [Autenticación y autorización mediante la Plataforma de identidad de Microsoft](https://docs.microsoft.com/azure/active-directory/develop/authentication-vs-authorization#authentication-and-authorization-using-microsoft-identity-platform). 
>
> Otros escenarios en los que falta **inicio de sesión único** en la navegación incluyen cuándo se hospeda una aplicación en otro inquilino o si su cuenta no tiene los permisos necesarios (Administrador global, Administrador de aplicaciones en la nube, Administrador de la aplicación o propietario de la entidad de servicio). Los permisos también pueden provocar un escenario en el que puede abrir **Inicio de sesión único** pero no podrá guardar. Para más información acerca de los roles administrativos de Azure AD, consulte (https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles).


## <a name="basic-saml-configuration"></a>Configuración básica de SAML

El proveedor de la aplicación le proporcionará estos valores. Puede especificar los valores manualmente o cargar un archivo de metadatos para extraer el valor de los campos.

> [!TIP]
> Muchas aplicaciones se han configurado previamente para funcionar con Azure AD. Estas aplicaciones se enumeran en la galería de aplicaciones, que puede examinar cuando añada una aplicación al inquilino de Azure AD. La [serie de guías de inicio rápido](view-applications-portal.md) le orientará a lo largo del proceso. En el caso de las aplicaciones de la galería, encontrará instrucciones de configuración detalladas paso a paso. Para acceder a los pasos, haga clic en el vínculo de la página de configuración de SAML correspondiente a la aplicación, tal y como se escribe en la serie de guías de inicio rápido, o bien examine una lista de todos los tutoriales de configuración de aplicaciones en [Tutoriales para integrar aplicaciones SaaS con Azure Active Directory](../saas-apps/tutorial-list.md).

| Opción de configuración básica de SAML | Iniciado por el proveedor de servicios | Iniciado por IdP | Descripción |
|:--|:--|:--|:--|
| **Identificador (identificador de entidad)** | Obligatorio para algunas aplicaciones | Obligatorio para algunas aplicaciones | Identifica de forma única la aplicación. Azure AD envía el identificador a la aplicación como el parámetro Audiencia del token SAML. Se espera que la aplicación lo valide. Este valor también aparece como el id. de entidad en los metadatos SAML proporcionados por la aplicación. Escriba una dirección URL que use el siguiente modelo: "https://<subdomain>.contoso.com" *Puede encontrar este valor como el elemento **Issuer** (Emisor) en el elemento **AuthnRequest** (solicitud SAML) enviado por la aplicación.* |
| **URL de respuesta** | Obligatorio | Obligatorio | Especifica el lugar donde la aplicación espera recibir el token SAML. La dirección URL de respuesta también se conoce como dirección URL del Servicio de consumidor de aserciones (ACS). Puede usar los campos adicionales de URL de respuesta para especificar varias direcciones URL de respuesta. Por ejemplo, puede que necesite direcciones URL de respuesta adicionales para varios subdominios. O bien, con fines de prueba, puede especificar varias direcciones URL de respuesta (host local y direcciones URL públicas) al mismo tiempo. |
| **URL de inicio de sesión** | Obligatorio | No especificar | Cuando un usuario abre esta dirección URL, el proveedor de servicios lo redirige a Azure AD para autenticar el usuario e iniciar sesión. Azure AD usa la dirección URL para iniciar la aplicación desde Office 365 o la página Aplicaciones de Azure AD. Si está en blanco, Azure AD efectúa un inicio de sesión iniciado por IdP cuando el usuario abre la aplicación desde Office 365, la página Aplicaciones de Azure AD o la dirección URL de inicio de sesión único de Azure AD.|
| **Estado de la retransmisión** | Opcional | Opcional | Especifica a la aplicación a dónde debe redirigir al usuario una vez completada la autenticación. Normalmente, el valor es una dirección URL válida para la aplicación. Sin embargo, algunas aplicaciones usan este campo de forma diferente. Para más información, pregunte al proveedor de la aplicación.
| **Dirección URL de cierre de sesión** | Opcional | Opcional | Se usa para devolver las respuestas de cierre de sesión SAML a la aplicación.


## <a name="user-attributes-and-claims"></a>Atributos y notificaciones de usuario 

Cuando un usuario se autentique en la aplicación, Azure AD emite un token SAML a la aplicación con información (o notificaciones) sobre el usuario que lo identifica de forma única. De forma predeterminada, dicha información incluye el nombre de usuario, la dirección de correo electrónico, el nombre y los apellidos del usuario. Es posible que tenga que personalizar estas notificaciones si, por ejemplo, la aplicación requiere valores de notificación específicos o un formato de **nombre** distinto del nombre de usuario. 

> [!IMPORTANT]
> Muchas aplicaciones están configuradas previamente y se muestran en la galería de aplicaciones, por lo que no tiene que configurar notificaciones de usuario ni de grupo. La [serie de guías de inicio rápido](view-applications-portal.md) le orientará a la hora de añadir y configurar aplicaciones.


El valor de **identificador de usuario único (id. de nombre)** es una notificación necesaria y relevante. El valor predeterminado es *user.userprincipalname*. El identificador de usuario permite identificar de manera exclusiva a cada usuario dentro de la aplicación. Por ejemplo, si la dirección de correo electrónico es a la vez el nombre de usuario y el identificador único, establezca el valor en *user.mail*.

Para más información sobre cómo personalizar las notificaciones, consulte el artículo de [procedimientos para personalizar las notificaciones emitidas en el token SAML para aplicaciones empresariales](../develop/active-directory-saml-claims-customization.md).

Se pueden añadir nuevas notificaciones. Para obtener más información, consulte [Incorporación de notificaciones específicas de la aplicación](../develop/active-directory-saml-claims-customization.md#adding-application-specific-claims) y para agregar notificaciones de grupo, consulte [Configuración de notificaciones de grupo](../hybrid/how-to-connect-fed-group-claims.md).


> [!NOTE]
> Para más formas de personalizar el token SAML desde Azure AD a la aplicación, consulte los siguientes recursos.
>- Para crear roles personalizados a través de Azure Portal, consulte [Configuración de notificaciones de rol](../develop/active-directory-enterprise-app-role-management.md).
>- Para personalizar las notificaciones a través de PowerShell, consulte [Personalización de notificaciones: PowerShell](../develop/active-directory-claims-mapping.md).
>- Para modificar el manifiesto de aplicación para configurar notificaciones opcionales, consulte [Configuración de notificaciones opcionales](../develop/active-directory-optional-claims.md).
>- Para establecer directivas de vigencia de los tokens de actualización, tokens de acceso, tokens de sesión y tokens de identificador consulte [Configuración de la vigencia de los tokens](../develop/active-directory-configurable-token-lifetimes.md). O bien, para restringir las sesiones de autenticación a través del acceso condicional de Azure AD, consulte las [funcionalidades de administración de sesiones de autenticación](https://go.microsoft.com/fwlink/?linkid=2083106).

## <a name="saml-signing-certificate"></a>Certificado de firma de SAML

Azure AD usa un certificado para firmar los tokens SAML que envía a la aplicación. Necesita este certificado para establecer la confianza entre Azure AD y la aplicación. Para obtener detalles sobre el formato del certificado, consulte la documentación de SAML de la aplicación. Para más información, consulte [Administración de certificados para el inicio de sesión único federado](manage-certificates-for-federated-single-sign-on.md) y [Opciones avanzadas de firma de certificados en el token SAML](certificate-signing-options.md).

> [!IMPORTANT]
> Muchas aplicaciones están configuradas previamente y se muestran en la galería de aplicaciones, por lo que no tendrá que buscar entre los certificados. La [serie de guías de inicio rápido](view-applications-portal.md) le orientará a la hora de añadir y configurar aplicaciones.

Desde Azure AD, puede descargar el certificado activo en formato Base64 o Raw directamente desde la página principal **Configurar el inicio de sesión único con SAML**. Además, puede obtener el certificado activo al descargar el archivo XML de metadatos de la aplicación o mediante la URL de metadatos de federación de aplicación. Para ver, crear o descargar certificados (activos o inactivos), siga estos pasos.

Algunos aspectos comunes que se deben comprobar para verificar un certificado son, entre otros, los siguientes: 
   - *La fecha de expiración es correcta.* Puede configurar la fecha de expiración hasta tres años en el futuro.
   - *El estado del certificado correspondiente es activo.* Si el estado es **Inactivo**, cambie el estado a **Activo**. Para cambiar el estado, haga clic con el botón derecho en la fila del certificado y seleccione **Activar el certificado**.
   - *La opción de firma y el algoritmo correctos.*
   - *Los correos electrónicos de notificación correctos.* Cuando el certificado activo esté cerca de la fecha de expiración, Azure AD envía una notificación a la dirección de correo electrónico configurada en este campo.

En ocasiones, es posible que tenga que descargar el certificado. Tenga en cuenta la ubicación donde lo guarda. Para descargar el certificado, seleccione una de las opciones de formato Base64, formato sin procesar o XML de metadatos de federación. Azure AD también proporciona la **dirección URL de metadatos de federación de la aplicación**, en donde puede acceder a los `https://login.microsoftonline.com/<Directory ID>/federationmetadata/2007-06/federationmetadata.xml?appid=<Application ID>`metadatos específicos de la aplicación en el formato.

Para hacer cambios en el certificado, seleccione el botón Editar. Hay varias cosas que puede hacer en la página **Certificado de firma SAML**:
   - Cree un certificado nuevo. Para ello, seleccione **Nuevo certificado**, **Fecha de expiración** y, por último, **Guardar**. Para activar el certificado, seleccione el menú contextual ( **...** ) y seleccione **Activar el certificado**.
   - Cargue un certificado con una clave privada y credenciales pfx. Para ello, seleccione **Importar certificado** y busque la ubicación del certificado. Escriba la **Contraseña PFX**, y seleccione **Agregar**.  
   - Configure la firma avanzada del certificado. Para obtener más información sobre estas opciones, consulte [Opciones avanzadas de firma de certificados](certificate-signing-options.md).
   - Notifique a otras personas cuando la fecha de expiración del certificado activo se aproxime. Para ello, escriba las direcciones de correo electrónico en los campos **Direcciones de correo electrónico de notificación**.

## <a name="set-up-the-application-to-use-azure-ad"></a>Configuración de la aplicación para que use Azure AD

En la sección **Configurar \<applicationName>** se muestran los valores que deben configurarse en la aplicación para que usen Azure AD como proveedor de identidades de SAML. Los valores se establecen en la página de configuración, en el sitio web de aplicaciones. Por ejemplo, si va a configurar GitHub, vaya al sitio github.com y establezca los valores. Si la aplicación está configurada previamente y se muestra en la galería de Azure AD, encontrará un vínculo para **ver instrucciones detalladas**. De lo contrario, tendrá que buscar la documentación de la aplicación que está configurando. 

Los valores de **URL de inicio de sesión** y **URL de cierre de sesión** se resuelven en el mismo punto de conexión, que es el punto de conexión de control de solicitudes SAML de su inquilino de Azure AD. 

**Identificador de Azure AD** es el valor del **Emisor** en el token SAML emitido a la aplicación.

## <a name="test-single-sign-on"></a>Prueba de inicio de sesión único

Una vez que haya configurado la aplicación para que use Azure AD como proveedor de identidades basado en SAML, puede probar la configuración para ver si el inicio de sesión único funciona para su cuenta. 

Seleccione **Prueba** y, a continuación, haga la prueba con el usuario que ha iniciado sesión en este momento o con otra identidad. 

Si el inicio de sesión se realiza correctamente, está listo para asignar usuarios y grupos a la aplicación SAML. Felicidades.

Si aparece un mensaje de error, realice los pasos siguientes:

1. Copie y pegue los detalles en el cuadro **What does the error look like?** (¿Qué aspecto tiene el error?).

    ![Obtención de instrucciones para la resolución](media/configure-single-sign-on-non-gallery-applications/error-guidance.png)

2. Seleccione **Obtenga instrucciones para la resolución**. Se muestran la causa principal e instrucciones para la resolución.  En este ejemplo, el usuario no se asignó a la aplicación.

3. Lea las directrices de resolución e intente solucionar el problema.

4. Vuelva a ejecutar la prueba hasta que se complete correctamente.

Para más información, consulte [Depuración del inicio de sesión único basado en SAML en aplicaciones de Azure Active Directory](../azuread-dev/howto-v1-debug-saml-sso-issues.md).


## <a name="next-steps"></a>Pasos siguientes

- [Serie de guías de inicio rápido sobre la administración de aplicaciones](view-applications-portal.md)
- [Asignación de usuarios y grupos a la aplicación](methods-for-assigning-users-and-groups.md)
- [Configuración del aprovisionamiento automático de cuentas de usuario](../app-provisioning/configure-automatic-user-provisioning-portal.md)
- [Protocolo SAML de inicio de sesión único](../develop/single-sign-on-saml-protocol.md)
