---
title: 'Federación directa con un proveedor de identidades para B2B: Azure AD'
description: Federe directamente con un proveedor de identidades SAML o WS-Fed para que los invitados puedan iniciar sesión en sus aplicaciones de Azure AD.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 06/24/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 78ad8761d3a4ff3e3cdab9dee5f50b469ff840fd
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87907733"
---
# <a name="direct-federation-with-ad-fs-and-third-party-providers-for-guest-users-preview"></a>Federación directa con AD FS y proveedores de terceros para usuarios invitados (versión preliminar)

> [!NOTE]
>  La federación directa es una característica en versión preliminar pública de Azure Active Directory. Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En este artículo se describe cómo establecer una federación directa con otra organización para la colaboración B2B. Puede establecer una federación directa con cualquier organización cuyo proveedor de identidades (IdP) admita el protocolo SAML 2.0 o WS-Fed.
Al configurar una federación directa con el proveedor de identidades de un asociado, los nuevos usuarios invitados de ese dominio pueden utilizar su propia cuenta de organización administrada por el proveedor de identidades para iniciar sesión con su inquilino de Azure AD y empezar a colaborar con usted. No es necesario que el usuario invitado cree otra cuenta de Azure AD.
> [!NOTE]
> Los usuarios invitados de federación directa deben iniciar sesión con un vínculo que incluye el contexto del inquilino (por ejemplo, `https://myapps.microsoft.com/?tenantid=<tenant id>` o `https://portal.azure.com/<tenant id>`, o, en el caso de un dominio comprobado, `https://myapps.microsoft.com/\<verified domain>.onmicrosoft.com`). Los vínculos directos a aplicaciones y los recursos también funcionan siempre que incluyan el contexto del inquilino. Actualmente, los usuarios de federación directa no pueden iniciar sesión con puntos de conexión comunes sin contexto de inquilino. Por ejemplo, si se usa `https://myapps.microsoft.com`, `https://portal.azure.com` o `https://teams.microsoft.com`, se producirá un error.
 
## <a name="when-is-a-guest-user-authenticated-with-direct-federation"></a>¿Cuándo se autentica un usuario invitado con la federación directa?
Después de configurar la federación directa con una organización, cualquier nuevo usuario invitado se autenticará mediante la federación directa. Es importante tener en cuenta que la configuración de la federación directa no cambia el método de autenticación para los usuarios invitados que ya han canjeado una invitación. Estos son algunos ejemplos:
 - Si los usuarios invitados ya han canjeado sus invitaciones y posteriormente configura la federación directa con su organización, esos usuarios invitados seguirán utilizando el mismo método de autenticación que utilizaron antes de configurar la federación directa.
 - Si se configura una federación directa con una organización asociada y se invita a los usuarios y después la organización asociada se traslada a Azure AD, los usuarios invitados que ya han canjeado las invitaciones seguirán utilizando la federación directa, siempre y cuando exista la directiva de federación directa en el inquilino.
 - Si se elimina la federación directa con una organización asociada, los usuarios invitados que actualmente utilizan la federación directa no podrán iniciar sesión.

En cualquiera de estos escenarios, puede actualizar el método de autenticación de un usuario invitado eliminando la cuenta de usuario invitado de su directorio y volviéndole a invitar.

La federación directa está asociada a espacios de nombres de dominio, como contoso.com y fabrikam.com. Al establecer una configuración de federación directa con AD FS o un proveedor de identidades de terceros, las organizaciones asocian uno o más espacios de nombres de dominio a estos proveedor de identidades. 

## <a name="end-user-experience"></a>Experiencia del usuario final 
Con la federación directa, los usuarios invitados inician sesión en el inquilino de Azure AD mediante su propia cuenta de organización. Cuando acceden a los recursos compartidos y se les pide que inicien sesión, los usuarios de la federación directa se redirigen al proveedor de identidades. Después del inicio de sesión correcto, vuelven a Azure AD para acceder a los recursos. Los tokens de actualización de los usuarios de la federación directa son válidos durante 12 horas, [la duración predeterminada del token de actualización de acceso directo](../develop/active-directory-configurable-token-lifetimes.md#exceptions)en Azure AD. Si el proveedor de identidades federado tiene el inicio de sesión único habilitado, el usuario experimentará un inicio de sesión único y no verá ninguna solicitud de inicio de sesión después de la autenticación inicial.

## <a name="limitations"></a>Limitaciones

### <a name="dns-verified-domains-in-azure-ad"></a>Dominios comprobados por DNS en Azure AD
El dominio con el que desea federarse ***no*** puede estar verificado por DNS en Azure AD. Se puede realizar una federación directa en los inquilinos no administrados (comprobados por correo electrónico o "viral") de Azure AD porque no están comprobados por DNS.

### <a name="authentication-url"></a>Dirección URL de autenticación
La federación directa solo se permite para las directivas en las que el dominio de la dirección URL de autenticación coincide con el dominio de destino, o en las que la dirección URL de autenticación es uno de estos proveedores de identidades permitidos (esta lista está sujeta a cambios):

-   accounts.google.com
-   pingidentity.com
-   login.pingone.com
-   okta.com
-   oktapreview.com
-   okta-emea.com
-   my.salesforce.com
-   federation.exostar.com
-   federation.exostartest.com

Por ejemplo, al configurar la federación directa para **fabrikam.com**, la dirección URL de autenticación `https://fabrikam.com/adfs` pasará la validación. Un host en el mismo dominio también pasará, por ejemplo `https://sts.fabrikam.com/adfs`. Sin embargo, la dirección URL de autenticación `https://fabrikamconglomerate.com/adfs` o `https://fabrikam.com.uk/adfs` para el mismo dominio no pasará.

### <a name="signing-certificate-renewal"></a>Renovación del certificado de firma
Si especifica la dirección URL de metadatos en la configuración del proveedor de identidades, Azure AD renovará automáticamente el certificado de firma cuando expire. Sin embargo, si el certificado se gira por cualquier razón antes de la hora de expiración, o si no proporciona una dirección URL de metadatos, Azure AD no podrá renovarlo. En este caso, deberá actualizar manualmente el certificado de firma.

### <a name="limit-on-federation-relationships"></a>Límite de relaciones de federación
Actualmente, se admite un máximo de 1000 relaciones de federación. Este límite incluye tanto las [federaciones internas](https://docs.microsoft.com/powershell/module/msonline/set-msoldomainfederationsettings?view=azureadps-1.0) como las federaciones directas.

### <a name="limit-on-multiple-domains"></a>Límite de varios dominios
Actualmente no se admite la federación directa con varios dominios del mismo inquilino.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes
### <a name="can-i-set-up-direct-federation-with-a-domain-for-which-an-unmanaged-email-verified-tenant-exists"></a>¿Puedo configurar la federación directa con un dominio para el que existe un inquilino no administrado (verificado por correo electrónico)? 
Sí. Si el dominio no se ha comprobado y el inquilino no ha experimentado una [adquisición de administración](../users-groups-roles/domains-admin-takeover.md), puede configurar una federación directa con el dominio. Los inquilinos no administrados o comprobados por correo electrónico se crean cuando un usuario canjea una invitación B2B o realiza un registro de autoservicio para Azure AD mediante un dominio que no existe actualmente. Puede configurar la federación directa con estos dominios. Si intenta configurar la federación directa con un dominio comprobado por DNS, ya sea en Azure Portal o con PowerShell, verá un error.
### <a name="if-direct-federation-and-email-one-time-passcode-authentication-are-both-enabled-which-method-takes-precedence"></a>Si la federación directa y la autenticación con código de acceso de un solo uso por correo electrónico están habilitadas, ¿qué método tiene prioridad?
Cuando se establece la federación directa con una organización asociada, tiene prioridad sobre la autenticación con código de acceso de un solo uso por correo electrónico para los nuevos usuarios invitados de esa organización. Si un usuario invitado ha canjeado una invitación mediante la autenticación de código de acceso de un solo uso antes de configurar la federación directa, seguirá utilizando la autenticación de código de acceso de un solo uso. 
### <a name="does-direct-federation-address-sign-in-issues-due-to-a-partially-synced-tenancy"></a>¿La federación directa se ocupa de los problemas de inicio de sesión debido a un inquilinato parcialmente sincronizado?
No, la característica del [código de acceso de un solo uso por correo electrónico](one-time-passcode.md) se debe usar en este escenario. Un "inquilinato parcialmente sincronizado" se refiere a un inquilino de Azure AD asociado en el que las identidades de usuario locales no están completamente sincronizadas con la nube. Un invitado cuya identidad aún no existe en la nube pero que intenta canjear su invitación de B2B no podrá iniciar sesión. La característica del código de acceso de un solo uso permitiría a este invitado iniciar sesión. La función de federación directa aborda escenarios en los que el invitado tiene su propia cuenta de organización administrada por el proveedor de identidades, pero la organización no tiene ninguna presencia de Azure AD.

## <a name="step-1-configure-the-partner-organizations-identity-provider"></a>Paso 1: Configuración del proveedor de identidades de la organización del asociado
En primer lugar, la organización asociada debe configurar el proveedor de identidades con las notificaciones necesarias y las veracidades de los usuarios de confianza. 

> [!NOTE]
> Para ilustrar cómo configurar un proveedor de identidades para la federación directa, usaremos los Servicios de federación de Active Directory (AD FS) como ejemplo. Consulte el artículo [Configuración de la federación directa con AD FS](direct-federation-adfs.md), que ofrece ejemplos de cómo configurar AD FS como un proveedor de identidades de SAML 2.0 o WS-Fed en preparación para la federación directa.

### <a name="saml-20-configuration"></a>Configuración de SAML 2.0

Azure AD B2B se puede configurar para federarse con proveedores de identidades que usan el protocolo SAML con los requisitos específicos que se indican a continuación. Para más información sobre cómo establecer una confianza entre su proveedor de identidades SAML y Azure AD, consulte [Uso de un proveedor de identidades (IdP) de SAML 2.0 para el inicio de sesión único](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-fed-saml-idp).  

> [!NOTE]
> El dominio de destino para la federación directa no debe ser comprobado por DNS en Azure AD. El dominio de la URL de autenticación debe coincidir con el dominio de destino o debe ser el dominio de un proveedor de identidades permitido. Consulte la sección [Limitaciones](#limitations) para obtener más información. 

#### <a name="required-saml-20-attributes-and-claims"></a>Atributos y notificaciones de SAML 2.0 necesarios
En las tablas siguientes se muestran los requisitos de los atributos y las notificaciones específicos que se deben configurar en el proveedor de identidades de terceros. Para establecer una federación directa, se deben recibir los siguientes atributos en la respuesta de SAML 2.0 del proveedor de identidades. Estos atributos se pueden configurar mediante la vinculación con el archivo XML del servicio de token de seguridad en línea o la introducción manual.

Atributos necesarios para la respuesta de SAML 2.0 desde el proveedor de identidades:

|Atributo  |Value  |
|---------|---------|
|AssertionConsumerService     |`https://login.microsoftonline.com/login.srf`         |
|Público     |`urn:federation:MicrosoftOnline`         |
|Emisor     |El URI del emisor del asociado IdP, por ejemplo`http://www.example.com/exk10l6w90DHM0yi...`         |


Las notificaciones necesarias para el token de SAML 2.0 emitido por el proveedor de identidades:

|Atributo  |Value  |
|---------|---------|
|Formato de NameID     |`urn:oasis:names:tc:SAML:2.0:nameid-format:persistent`         |
|emailaddress     |`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`         |

### <a name="ws-fed-configuration"></a>Configuración de WS-Fed 
Se puede configurar Azure AD B2B para que funcione con proveedores de identidades que usan el protocolo WS-Fed con algunos requisitos específicos, que se indican a continuación. Actualmente, los dos proveedores de WS-Fed se han probado para determinar su compatibilidad con Azure AD e incluyen AD FS y Shibboleth. Para más información sobre cómo establecer la veracidad de un usuario de confianza entre un proveedor compatible con WS-Fed y Azure AD, consulte el documento "STS Integration Paper using WS Protocols" (Documento de integración con STS mediante protocolos WS) en los [documentos de compatibilidad del proveedor de identidades de Azure AD](https://www.microsoft.com/download/details.aspx?id=56843).

> [!NOTE]
> El dominio de destino para la federación directa no debe ser comprobado por DNS en Azure AD. El dominio de la URL de autenticación debe coincidir con el dominio de destino o con el dominio de un proveedor de identidades permitido. Consulte la sección [Limitaciones](#limitations) para obtener más información. 

#### <a name="required-ws-fed-attributes-and-claims"></a>Atributos y notificaciones de WS-Fed necesarios

En las tablas siguientes se muestran los requisitos de los atributos y las notificaciones específicos que se deben configurar en el proveedor de identidades de WS-Fed de terceros. Para configurar una federación directa, se deben recibir los siguientes atributos en el mensaje de WS-Fed del proveedor de identidades. Estos atributos se pueden configurar mediante la vinculación con el archivo XML del servicio de token de seguridad en línea o la introducción manual.

Atributos necesarios en el mensaje de WS-Fed desde el proveedor de identidades:
 
|Atributo  |Value  |
|---------|---------|
|PassiveRequestorEndpoint     |`https://login.microsoftonline.com/login.srf`         |
|Público     |`urn:federation:MicrosoftOnline`         |
|Emisor     |El URI del emisor del asociado IdP, por ejemplo`http://www.example.com/exk10l6w90DHM0yi...`         |

Las notificaciones necesarias para el token de WS-Fed emitido por el proveedor de identidades:

|Atributo  |Value  |
|---------|---------|
|ImmutableID     |`http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID`         |
|emailaddress     |`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`         |

## <a name="step-2-configure-direct-federation-in-azure-ad"></a>Paso 2: Configuración de la federación directa en Azure AD 
A continuación, va a configurar la federación con el proveedor de identidades en Azure AD configurado en el paso 1. Puede usar el portal de Azure AD o PowerShell. Pueden pasar de 5 a 10 minutos antes de que la directiva de la federación directa entre en vigor. Durante este tiempo, no intente canjear una invitación por el dominio de la federación directa. Los atributos siguientes son necesarios:
- URI del emisor del proveedor de identidades del asociado
- Punto de conexión de la autenticación pasiva del proveedor de identidades del asociado (solo se admite https)
- Certificado

### <a name="to-configure-direct-federation-in-the-azure-ad-portal"></a>Para configurar la federación directa en el portal de Azure AD

1. Vaya a [Azure Portal](https://portal.azure.com/). En el panel izquierdo, seleccione **Azure Active Directory**. 
2. Seleccione **External Identities** > **Todos los proveedores de identidades**.
3. Seleccione y, después, **Nuevo proveedor de identidades de SAML/WS-Fed**.

    ![Captura de pantalla que muestra el botón para agregar un nuevo proveedor de identidades de SAML o WS-Fed](media/direct-federation/new-saml-wsfed-idp.png)

4. En la página **New SAML/WS-Fed IdP** (Nuevo proveedor de identidades de SAMS/WS-FED), en **Protocolo de proveedor de identidades**, seleccione **SAML** o **WS-FED**.

    ![Captura de pantalla que muestra el botón de análisis en la página del proveedor de identidades de SAML o WS-Fed](media/direct-federation/new-saml-wsfed-idp-parse.png)

5. Especifique el nombre de dominio de su organización asociada, que será el nombre de dominio de destino para la federación directa.
6. Puede cargar un archivo de metadatos para rellenar los detalles correspondientes. Si decide escribir los metadatos manualmente, introduzca la siguiente información:
   - Nombre de dominio del proveedor de identidades del asociado
   - Identificador de entidad de proveedor de identidades del asociado
   - Punto de conexión de solicitante pasivo del proveedor de identidades del asociado
   - Certificado
   > [!NOTE]
   > La dirección URL de metadatos es opcional, pero se recomienda encarecidamente. Si proporciona la dirección URL de metadatos, Azure AD puede renovar automáticamente el certificado de firma cuando expire. Si el certificado se gira por cualquier razón antes de la hora de expiración, o si no proporciona una dirección URL de metadatos, Azure AD no podrá renovarlo. En este caso, deberá actualizar manualmente el certificado de firma.

7. Seleccione **Guardar**. 

### <a name="to-configure-direct-federation-in-azure-ad-using-powershell"></a>Para configurar la federación directa en Azure AD con PowerShell

1. Instale la versión más reciente de Azure AD PowerShell para el módulo Graph ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview)). (Si necesita conocer pasos detallados, el inicio rápido para agregar un usuario invitado incluye la sección [Instalación del último módulo de AzureADPreview](b2b-quickstart-invite-powershell.md#install-the-latest-azureadpreview-module)). 
2. Ejecute el siguiente comando: 
   ```powershell
   Connect-AzureAD
   ```
1. En el símbolo del sistema, inicie sesión con la cuenta de administrador global administrada. 
2. Ejecute los siguientes comandos, reemplazando los valores del archivo de metadatos de la federación. Para el servidor de AD FS y Okta, el archivo de federación es federationmetadata.xml, por ejemplo: `https://sts.totheclouddemo.com/federationmetadata/2007-06/federationmetadata.xml`. 

   ```powershell
   $federationSettings = New-Object Microsoft.Open.AzureAD.Model.DomainFederationSettings
   $federationSettings.PassiveLogOnUri ="https://sts.totheclouddemo.com/adfs/ls/"
   $federationSettings.LogOffUri = $federationSettings.PassiveLogOnUri
   $federationSettings.IssuerUri = "http://sts.totheclouddemo.com/adfs/services/trust"
   $federationSettings.MetadataExchangeUri="https://sts.totheclouddemo.com/adfs/services/trust/mex"
   $federationSettings.SigningCertificate= <Replace with X509 signing cert’s public key>
   $federationSettings.PreferredAuthenticationProtocol="WsFed" OR "Samlp"
   $domainName = <Replace with domain name>
   New-AzureADExternalDomainFederation -ExternalDomainName $domainName  -FederationSettings $federationSettings
   ```

## <a name="step-3-test-direct-federation-in-azure-ad"></a>Paso 3: Prueba de la federación directa en Azure AD
Ahora pruebe la configuración de la federación directa e invite a un nuevo usuario invitado de B2B. Para más información, consulte [Incorporación de usuarios de colaboración B2B de Azure Active Directory en Azure Portal](add-users-administrator.md).
 
## <a name="how-do-i-edit-a-direct-federation-relationship"></a>¿Cómo se puede editar una relación de federación directa?

1. Vaya a [Azure Portal](https://portal.azure.com/). En el panel izquierdo, seleccione **Azure Active Directory**. 
2. Seleccione **External Identities**.
3. Seleccione **Todos los proveedores de identidades**.
4. En **SAML/WS-Fed identity providers** (Proveedores de identidades SAML/WS-Fed), seleccione el proveedor.
5. En el panel de detalles del proveedor de identidades, actualice los valores.
6. Seleccione **Guardar**.


## <a name="how-do-i-remove-direct-federation"></a>¿Cómo se quita una federación directa?
Puede quitar la configuración de la federación directa. Si lo hace, los usuarios invitados de la federación directa que ya hayan canjeado sus invitaciones no podrán iniciar sesión. Pero puede darles acceso a los recursos de nuevo si los elimina del directorio y los vuelve a invitar. Para quitar una federación directa con un proveedor de identidades en el portal de Azure AD:

1. Vaya a [Azure Portal](https://portal.azure.com/). En el panel izquierdo, seleccione **Azure Active Directory**. 
2. Seleccione **External Identities**.
3. Seleccione **Todos los proveedores de identidades**.
4. Seleccione el proveedor de identidades y, a continuación, seleccione **Eliminar**. 
5. Seleccione **Sí** para confirmar la eliminación. 

Para quitar una federación directa con un proveedor de identidades mediante PowerShell:
1. Instale la versión más reciente de Azure AD PowerShell para el módulo Graph ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview)).
2. Ejecute el siguiente comando: 
   ```powershell
   Connect-AzureAD
   ```
3. En el símbolo del sistema, inicie sesión con la cuenta de administrador global administrada. 
4. Escriba el comando siguiente:
   ```powershell
   Remove-AzureADExternalDomainFederation -ExternalDomainName  $domainName
   ```

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la [experiencia de canje de invitación](redemption-experience.md) cuando los usuarios externos inician sesión con varios proveedores de identidades.