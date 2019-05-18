---
title: Inicio de sesión único - ajena a la Galería de aplicaciones - plataforma Microsoft identity | Microsoft Docs
description: Configurar el inicio de sesión único (SSO) a ajena a la Galería aplicaciones en la plataforma de identidad de Microsoft (Azure AD)
services: active-directory
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: article
ms.workload: identity
ms.date: 01/08/2019
ms.author: mimart
ms.reviewer: asmalser,luleon
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8f55de2d081f12f1bddf7e68a795133c4b394823
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65825001"
---
# <a name="configure-single-sign-on-to-non-gallery-applications-in-microsoft-identity-platform"></a>Configurar inicio de sesión único para aplicaciones que no sean de la galería en la plataforma Microsoft identity

Este artículo trata sobre una característica que permite a los administradores configurar el inicio de sesión único de las aplicaciones no presentan en la Galería de aplicaciones de plataforma de identidad de Microsoft *sin escribir código*.

Si por el contrario desea obtener instrucciones para desarrolladores sobre cómo integrar aplicaciones personalizadas con Azure AD a través de código, consulte [Escenarios de autenticación para Azure AD](../develop/authentication-scenarios.md).

La Galería de aplicaciones de plataforma de identidad de Microsoft proporciona una lista de aplicaciones que se sabe que admiten un formulario de inicio de sesión único con la plataforma de identidad de Microsoft, como se describe en [en este artículo](what-is-single-sign-on.md). Una vez que un especialista en TI o un integrador de sistemas de la organización ha encontrado la aplicación que desea conectar, puede empezar por seguir las instrucciones detalladas que se proporcionan en Azure Portal para habilitar el inicio de sesión único.

Estas funcionalidades también están disponibles, según el contrato de licencia. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/active-directory/)para obtener más información.

- Integración de autoservicio de una aplicación que use un protocolo modernos como [OpenId Connect y OAuth](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-protocols) para autenticar sus usuarios y obtener tokens para [Microsoft Graph](https://graph.microsoft.com).
- Integración de autoservicio de cualquier aplicación que admita proveedores de identidades SAML 2.0 (iniciado por el proveedor de servicios o por el proveedor de identidades)
- Integración de autoservicio de cualquier aplicación web que tenga una página de inicio de sesión basada en HTML que use [SSO basado en contraseña](what-is-single-sign-on.md#password-based-sso)
- Conexión autoservicio de las aplicaciones que usan el protocolo SCIM para el aprovisionamiento de usuarios ([se describe aquí](use-scim-to-provision-users-and-groups.md))
- Capacidad para agregar vínculos a cualquier aplicación del [iniciador de aplicaciones de Office 365](https://blogs.office.com/2014/10/16/organize-office-365-new-app-launcher-2/) o del [panel de acceso de Azure AD](what-is-single-sign-on.md#linked-sign-on)

Aquí puede incluir no solo las aplicaciones SaaS que usa, pero que aún ha integrado en la Galería de aplicaciones de Azure AD, sino también las aplicaciones web de terceros que la organización ha implementado en los servidores que controla, ya sea en la nube o locales.

Estas funcionalidades, también conocidas como *plantillas de integración de aplicaciones*, proporcionan puntos de conexión basados en estándares para aplicaciones que admiten SAML, SCIM o autenticación basada en formularios, e incluyen opciones y configuraciones flexibles para compatibilidad con un amplio número de aplicaciones.

## <a name="adding-an-unlisted-application"></a>Adición de una aplicación que no figura en la lista

La plataforma Microsoft Identity proporciona dos mecanismos para registrar aplicaciones.

Una aplicación que utiliza un protocolo modernos como [OpenId Connect y OAuth](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-protocols) autenticar sus usuarios se ha registrado mediante el [registro de la aplicación Portal de](https://docs.microsoft.com/azure/active-directory/develop/quickstart-v2-register-an-app).

El registro de las aplicaciones que usan todos los demás tipos de [admite mecanismos de autenticación](what-is-single-sign-on.md) como el [SAML](https://docs.microsoft.com/azure/active-directory/develop/single-sign-on-saml-protocol) protocolo utilice el **aplicaciones empresariales** hoja para Conéctese a sí mismos con la plataforma Microsoft Identity.

Para conectar una aplicación con una plantilla de integración de aplicaciones, inicie sesión en el portal de Azure con su cuenta de administrador de plataforma de identidad de Microsoft. Vaya a la sección **Active Directory > Aplicaciones empresariales > Nueva aplicación > Aplicación situada fuera de la galería**, seleccione **Agregar** y, a continuación, **Agregar una aplicación de la galería** .

![Agregar aplicación](./media/configure-single-sign-on-non-gallery-applications/customapp1.png)

Si no se encontró la aplicación deseada en la galería de aplicaciones, puede agregar una aplicación que no figura en la lista mediante el icono **Aplicación situada fuera de la galería** que se muestra en los resultados de búsqueda. Después de escribir el nombre de la aplicación, puede configurar las opciones y el comportamiento de inicio de sesión único.

> [!TIP]
> Como procedimiento recomendado, use la función de búsqueda para comprobar si la aplicación ya existe en la Galería de aplicaciones. Si se encuentra la aplicación y su descripción menciona el inicio de sesión único, la aplicación ya es compatible con el inicio de sesión único federado.

![Search](./media/configure-single-sign-on-non-gallery-applications/customapp2.png)

La adición de una aplicación de esta forma supone una experiencia parecida a la disponible para las aplicaciones preintegradas. Para comenzar, seleccione **Configurar inicio de sesión único** o haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación. La siguiente pantalla presenta las opciones para configurar el inicio de sesión único. Las opciones se describen en las secciones siguientes de este artículo.
  
![Opciones de configuración](./media/configure-single-sign-on-non-gallery-applications/customapp3.png)

## <a name="saml-based-single-sign-on"></a>Inicio de sesión único basado en SAML

Seleccione esta opción para configurar la autenticación basada en SAML para la aplicación. Esto requiere que la aplicación admita SAML 2.0. Se debe recopilar información acerca de cómo usar las funcionalidades de SAML de la aplicación antes de continuar. Complete las siguientes secciones para configurar el inicio de sesión único entre la aplicación y Azure AD.

### <a name="enter-basic-saml-configuration"></a>Especificación de la configuración básica de SAML

Para configurar Azure AD, especifique la configuración básica de SAML. Puede especificar los valores manualmente o cargar un archivo de metadatos para extraer el valor de los campos.

  ![Dominio y direcciones URL de Litware](./media/configure-single-sign-on-non-gallery-applications/customapp4.png)

- **Dirección URL de inicio de sesión (iniciado por SP sólo)** : donde el usuario va a iniciar sesión en esta aplicación. Si la aplicación está configurada para llevar a cabo servicio iniciado por el proveedor de sesión único, a continuación, cuando un usuario navega a esta dirección URL, el proveedor de servicios realizará la redirección necesaria a Azure AD para autenticar e iniciar sesión el usuario. Si este campo se rellena, Azure AD utilizará esta dirección URL para iniciar la aplicación desde el panel de acceso de Azure AD y Office 365. Si se omite este campo, Azure AD realizará de proveedor de identidades-inicia sesión cuando se inicia la aplicación de Office 365, el Panel de acceso de Azure AD, o desde el Azure único inicio de sesión en dirección URL de AD (se puede copiar desde la pestaña panel).
- **Identificador**: debería identificar de forma exclusiva la aplicación para la que se configura el inicio de sesión único. Puede encontrar este valor como el elemento Issuer en el elemento AuthRequest (solicitud SAML) enviado por la aplicación. Este valor también aparece como el **Id. de entidad** en los metadatos SAML proporcionados por la aplicación. Compruebe la documentación de SAML de la aplicación para más información sobre qué es el identificador de entidad o el valor de Audiencia. 

    El siguiente es un ejemplo de cómo el identificador o el emisor aparecen en la solicitud de SAML enviada por la aplicación en Azure AD:

    ```XML
    <samlp:AuthnRequest
    xmlns="urn:oasis:names:tc:SAML:2.0:metadata"
    ID="id6c1c178c166d486687be4aaf5e482730"
    Version="2.0" IssueInstant="2013-03-18T03:28:54.1839884Z"
    xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
    <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://www.contoso.com</Issuer>
    </samlp:AuthnRequest>
    ```

- **URL de respuesta** : la dirección URL de respuesta es el lugar donde la aplicación espera recibir el token SAML. Esto también se conoce como dirección URL del Servicio de consumidor de aserciones (ACS). Compruebe la documentación de SAML de la aplicación para más información sobre qué es la URL de respuesta del token SAML o la URL de ACS. 

    Puede usar el siguiente script de PowerShell para configurar varias URL de respuesta.

    ```powershell
    $sp = Get-AzureADServicePrincipal -SearchString "<Exact App  name>"
    $app = Get-AzureADApplication -SearchString "<Exact app name>"
    Set-AzureADApplication -ObjectId $app.ObjectId -ReplyUrls "<ReplyURLs>"
    Set-AzureADServicePrincipal -ObjectId $sp.ObjectId -ReplyUrls "<ReplyURLs>"
    ```

Para obtener más información, consulte [las solicitudes de autenticación de SAML 2.0 y las respuestas que admite la plataforma de identidad de Microsoft (Azure AD)](https://docs.microsoft.com/azure/active-directory/develop/active-directory-single-sign-on-protocol-reference?/?WT.mc_id=DOC_AAD_How_to_Debug_SAML)


### <a name="review-or-customize-the-claims-issued-in-the-saml-token"></a>Revisión o personalización de las notificaciones emitidas en el token SAML

Cuando un usuario se autentique en la aplicación, Azure AD emitirá un token SAML a la aplicación que contiene información (o notificaciones) sobre el usuario que lo identifica de forma única. De forma predeterminada, dicha información incluye el nombre de usuario, la dirección de correo electrónico, el nombre y los apellidos del usuario. 

Las notificaciones enviadas en el token SAML a la aplicación se pueden ver o editar en la pestaña **Atributos** .

  ![Atributos](./media/configure-single-sign-on-non-gallery-applications/customapp7.png)

Hay dos razones por las que tendría que editar las notificaciones emitidas en el token SAML:

- La aplicación se ha creado para requerir un conjunto diferente de URI o valores de notificación.
- La aplicación se implementó de forma que requiere la notificación NameIdentifier no sea el nombre de usuario (es decir, nombre principal de usuario) almacenado en la plataforma Microsoft identity.

Para más información, consulte [Personalización de las notificaciones emitidas en el token SAML para aplicaciones empresariales](./../develop/../develop/active-directory-saml-claims-customization.md). 

### <a name="review-certificate-expiration-data-status-and-email-notification"></a>Revisión de los datos de expiración, el estado y la notificación por correo electrónico de los certificados

Al crear una aplicación de la galería o de fuera de la galería, Azure AD creará un certificado específico de la aplicación con una fecha de expiración de tres años a partir de la fecha de creación. Necesita este certificado para establecer la confianza entre Azure AD y la aplicación. Para obtener detalles sobre el formato del certificado, consulte la documentación de SAML de la aplicación. 

De Azure AD, puede descargar el certificado en formato sin procesar o Base64. Además, puede obtener el certificado al descargar el archivo XML de metadatos de la aplicación o mediante el uso de la URL de metadatos de la federación de aplicaciones.

  ![Certificate](./media/configure-single-sign-on-non-gallery-applications/certificate.png)

Compruebe que el certificado tiene:

- La fecha de expiración deseada. Puede configurar la fecha de expiración de tres años como máximo.
- Un estado activo. Si el estado está inactivo, cambie el estado a activo. Para cambiar el estado, active la casilla **Activo** y, guarde la configuración. 
- El correo electrónico de notificación correcto. Cuando el certificado active esté cerca de la fecha de expiración, Azure AD enviará una notificación a la dirección de correo electrónico configurada en este campo.  

Para obtener más información, consulte [administrar certificados para federada single sign-on en la plataforma Microsoft identity](manage-certificates-for-federated-single-sign-on.md).

### <a name="set-up-target-application"></a>Configuración de la aplicación de destino

Para configurar la aplicación para inicio de sesión único, busque la documentación de la aplicación. Para buscar la documentación, desplácese hasta el final de la página de configuración basado en SAML sign-on y, a continuación, haga clic en **configurar \<nombreDeAplicación >**. 

Los valores necesarios varían dependiendo de la aplicación. Para más información, consulte la documentación de SAML de la aplicación. Las URL de los servicios de inicio de sesión y cierre de sesión se resuelven en el mismo punto de conexión, que es el punto de conexión de control de solicitudes de SAML de su instancia de Azure AD. El identificador de la entidad de SAML es el valor que aparece como Emisor en el token SAML que se emite para la aplicación.


### <a name="assign-users-and-groups-to-your-saml-application"></a>Asignación de usuarios y grupos a una aplicación SAML

Tras configurar la aplicación para que use Azure AD como proveedor de identidades basado en SAML, ya casi está lista para probarla. Como control de seguridad, Azure AD no emitirá un token que permita al usuario iniciar sesión en la aplicación, a menos que Azure AD le haya concedido acceso. A los usuarios se les puede conceder acceso directamente o a través de la pertenencia al grupo. 

Para asignar un usuario o grupo a una aplicación, haga clic en el botón **Asignar usuarios** . Seleccione el usuario o grupo que desea asignar y, a continuación, haga clic en el botón **Asignar** .

  ![Asignar usuarios](./media/configure-single-sign-on-non-gallery-applications/customapp6.png)

La asignación de un usuario permitirá a Azure AD emitir un token para el usuario. También hace que aparezca un icono para esta aplicación en el panel de acceso del usuario. Si el usuario usa Office 365, también aparecerá un icono de la aplicación en el iniciador de aplicaciones de Office 365. 

> [!NOTE] 
> Para cargar un logotipo de icono de la aplicación, pulse el botón **Cargar logotipo** en la pestaña **Configura**r de la aplicación. 


### <a name="test-the-saml-application"></a>Prueba de la aplicación SAML

Antes de probar la aplicación SAML, debe tener configurada la aplicación con Azure AD y usuarios o grupos asignados a la aplicación. Para probar la aplicación SAML, consulte [cómo depurar basado en SAML único inicio de sesión en aplicaciones en la plataforma Microsoft identity](../develop/howto-v1-debug-saml-sso-issues.md).

## <a name="password-single-sign-on"></a>Inicio de sesión único con contraseña

Seleccione esta opción para configurar [el inicio de sesión único basado en contraseña](what-is-single-sign-on.md) para una aplicación web que tiene una página de inicio de sesión HTML. El SSO basado en contraseña, también conocido como almacenamiento de contraseñas, permite administrar el acceso y las contraseñas de los usuarios en aplicaciones web que no admiten la federación de identidades. También es útil para escenarios en los que varios usuarios necesitan compartir una sola cuenta, como las cuentas de aplicaciones de redes sociales de la organización. 

Después de seleccionar **Siguiente**, se le solicitará que escriba la dirección URL de página de inicio de sesión basada en web de la aplicación. Tenga en cuenta que dicha página debe incluir los campos en que se especifican el nombre de usuario y la contraseña. Una vez especificados, Azure AD inicia un proceso para analizar la página de inicio de sesión para detectar si se han escrito un nombre de usuario y una contraseña. Si el resultado del proceso no es satisfactorio, le guía a través de un proceso alternativo de instalación de una extensión de explorador (requiere Internet Explorer, Chrome o Firefox) que le permitirá capturar manualmente los campos.

Una vez que se captura la página de inicio de sesión, es posible asignar usuarios y grupos, las directivas de credenciales se pueden establecer como [aplicaciones de SSO con contraseña](what-is-single-sign-on.md)normales.

> [!NOTE] 
> Para cargar un logotipo de icono de la aplicación, pulse el botón **Cargar logotipo** en la pestaña **Configura**r de la aplicación. 
>

## <a name="existing-single-sign-on"></a>Inicio de sesión único existente
Seleccione esta opción si desea agregar un vínculo a una aplicación en el panel de acceso de Azure AD o en el Portal de Office 365 de la organización. Esto se puede usar para agregar vínculos a aplicaciones web personalizadas que actualmente utilizan los Servicios de federación de Active Directory (u otro servicio de federación), en lugar de Azure AD para la autenticación. O bien, puede agregar vínculos profundos a páginas específicas de SharePoint o a otras páginas web que desee que aparezcan en los paneles de acceso de su usuario. 

Después de seleccionar **Siguiente**, se le solicitará que escriba la dirección URL de la aplicación con la que se va a establecer el vínculo. Una vez completada la operación, ya es posible asignar usuarios y grupos a la aplicación, lo que hace que la aplicación aparezca en el [iniciador de aplicaciones de Office 365](https://blogs.office.com/2014/10/16/organize-office-365-new-app-launcher-2/) o en el [panel de acceso de Azure AD](end-user-experiences.md) de dichos usuarios.

> [!NOTE] 
> Para cargar un logotipo de icono de la aplicación, pulse el botón **Cargar logotipo** en la pestaña **Configura**r de la aplicación. 
>

## <a name="related-articles"></a>Artículos relacionados

- [Personalización de notificaciones emitidas en el token SAML para aplicaciones previamente integradas en Azure Active Directory](../develop/active-directory-saml-claims-customization.md)
- [Cómo depurar el inicio de sesión único basado en SAML en aplicaciones de Azure Active Directory](../develop/howto-v1-debug-saml-sso-issues.md)
- [Plataforma de identidad (Azure Active Directory para desarrolladores de Microsoft](https://aka.ms/aaddev)
