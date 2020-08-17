---
title: 'Azure AD Connect: Uso de un proveedor de identidades (IdP) de SAML 2.0 para un inicio de sesión único - Azure'
description: En este documento se describe el uso de un IdP compatible con SAML 2.0 para el inicio de sesión único.
services: active-directory
author: billmath
manager: daveba
ms.custom: it-pro
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/13/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 274f603fa0d7a48b99a7caed5d465d44bc68e9ca
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87019971"
---
#  <a name="use-a-saml-20-identity-provider-idp-for-single-sign-on"></a>Uso de un proveedor de identidades (IdP) de SAML 2.0 para el inicio de sesión único

Este documento contiene información sobre el uso de un proveedor de identidades basado en un perfil de SP-Lite y compatible con SAML 2.0 como proveedor de identidades o servicio de token de seguridad (STS) preferido. Este escenario resulta útil si ya tiene un directorio de usuario y un almacén de contraseñas local a los que se puede acceder mediante SAML 2.0. Este directorio de usuario existente se puede usar para iniciar sesión en Office 365 y otros recursos protegidos por Azure AD. El perfil SP-Lite de SAML 2.0 se basa en el estándar de identidad federada del lenguaje de marcado de aserción de seguridad (SAML) de uso generalizado y proporciona un marco de inicio de sesión e intercambio de atributos.

>[!NOTE]
>Para obtener una lista de Idp de terceros cuyo uso se ha probado con Azure AD, consulte la [lista de compatibilidad de federación de Azure AD](how-to-connect-fed-compatibility.md).

Microsoft admite esta experiencia de inicio de sesión único como la integración de un servicio en la nube de Microsoft, como Office 365, con el Idp basado en el perfil de SAML 2.0 configurado. Dado que los proveedores de identidades de SAML 2.0 son productos de terceros, Microsoft no proporciona soporte técnico con la implementación, la configuración y los procedimientos recomendados de solución de problemas en relación a ellos. Una vez configurada adecuadamente, se puede probar la integración con el proveedor de identidades de SAML 2.0 para ver si es correcta mediante la herramienta Analizador de conectividad de Microsoft que se describe con más detalle a continuación. Para obtener más información sobre el proveedor de identidades basado en un perfil SP-Lite de SAML 2.0, consulte con la organización que lo suministra.

> [!IMPORTANT]
> Solo un conjunto limitado de clientes están disponibles en este escenario de inicio de sesión con proveedores de identidades de SAML 2.0, entre los que se incluyen:
> 
> - Clientes basados en web como Outlook Web Access y SharePoint Online
> - Clientes enriquecidos para el correo electrónico que usan autenticación básica y un método de acceso de Exchange compatible, como IMAP, POP, Active Sync, MAPI, etc. (es necesario implementar el protocolo de cliente mejorado), entre los que se incluyen:
>     - Microsoft Outlook 2010, Outlook 2013, Outlook 2016, iPhone de Apple (distintas versiones de iOS)
>     - Varios dispositivos de Android de Google
>     - Windows Phone 7, Windows Phone 7.8 y Windows Phone 8.0
>     - Cliente de correo de Windows 8 y Windows 8.1
>     - Cliente de correo de Windows 10

Todos los demás clientes no están disponibles en este escenario de inicio de sesión con el proveedor de identidades de SAML 2.0. Por ejemplo, el cliente de escritorio Lync 2010 no es capaz de iniciar sesión en el servicio con el proveedor de identidades de SAML 2.0 configurado para el inicio de sesión único.

## <a name="azure-ad-saml-20-protocol-requirements"></a>Requisitos del protocolo SAML 2.0 de Azure AD
Este documento contiene los requisitos detallados sobre el protocolo y el formato de mensaje que debe implementar su proveedor de identidades de SAML 2.0 para la federación con Azure AD a fin de habilitar el inicio de sesión en uno o varios servicios en la nube de Microsoft (como Office 365). El usuario de confianza de SAML 2.0 (SP-STS) que se usa en este escenario para un servicio en la nube de Microsoft es Azure AD.

Se recomienda que se asegure de que sus mensajes de salida del proveedor de identidades de SAML 2.0 sean lo más parecidos posibles a los seguimientos de ejemplo proporcionados. Además, cuando sea posible, use valores de atributo específicos de los metadatos de Azure AD suministrados. Cuando esté satisfecho con los mensajes de salida, puede probar con el Analizador de conectividad de Microsoft, como se describe a continuación.

Los metadatos de Azure AD se pueden descargar desde esta dirección URL: [https://nexus.microsoftonline-p.com/federationmetadata/saml20/federationmetadata.xml](https://nexus.microsoftonline-p.com/federationmetadata/saml20/federationmetadata.xml).
Para los clientes en China que usan la instancia de Office 365 específica para China, se debe usar el punto de conexión de federación siguiente: [https://nexus.partner.microsoftonline-p.cn/federationmetadata/saml20/federationmetadata.xml](https://nexus.partner.microsoftonline-p.cn/federationmetadata/saml20/federationmetadata.xml).

## <a name="saml-protocol-requirements"></a>Requisitos del protocolo SAML
En esta sección se detalla como se unen los pares de mensajes de solicitud y respuesta para ayudarle a dar formato a los mensajes correctamente.

Azure AD se puede configurar para funcionar con proveedores de identidades que usan el perfil SP-Lite de SAML 2.0 con algunos requisitos específicos, que se indican a continuación. Mediante el uso de los mensajes de solicitud y respuesta de SAML de ejemplo, junto con las pruebas manuales automatizadas, puede trabajar para conseguir la interoperabilidad con Azure AD.

## <a name="signature-block-requirements"></a>Requisitos del bloque de firma
En el mensaje de respuesta de SAML, el nodo de firma contiene información sobre la firma digital del propio mensaje. El bloque de firma tiene los siguientes requisitos:

1. El propio nodo de aserción debe estar firmado.
2.  El algoritmo RSA-sha1 debe usarse como DigestMethod. No se aceptan otros algoritmos de firma digital.
   `<ds:DigestMethod Algorithm="https://www.w3.org/2000/09/xmldsig#sha1"/>`
3.  También puede firmar el documento XML. 
4.  El algoritmo Transform debe coincidir con los valores del ejemplo siguiente:    `<ds:Transform Algorithm="https://www.w3.org/2000/09/xmldsig#enveloped-signature"/>
       <ds:Transform Algorithm="https://www.w3.org/2001/10/xml-exc-c14n#"/>`
9.  El algoritmo SignatureMethod debe coincidir con el ejemplo siguiente:    `<ds:SignatureMethod Algorithm="https://www.w3.org/2000/09/xmldsig#rsa-sha1"/>`

## <a name="supported-bindings"></a>Enlaces admitidos
Los enlaces son los parámetros de comunicación relacionados con el transporte que son necesarios. Los siguientes requisitos se aplican a los enlaces:

1. HTTPS es el transporte requerido.
2.  Azure AD necesitará HTTP POST para el envío del token durante el inicio de sesión.
3.  Azure AD usará HTTP POST en la solicitud de autenticación al proveedor de identidades y REDIRECT en el mensaje de cierre de sesión a este proveedor de identidades.

## <a name="required-attributes"></a>Atributos necesarios
En esta tabla se muestran los requisitos de atributos específicos en el mensaje de SAML 2.0.
 
|Atributo|Descripción|
| ----- | ----- |
|NameID|El valor de esta aserción debe ser el mismo que el valor de ImmutableID del usuario de Azure AD. Puede tener hasta 64 caracteres alfanuméricos. Los caracteres seguros que no sean HTML deben estar codificados, por ejemplo, un carácter "+" se muestra como ".2B".|
|IDPEmail|El nombre principal de usuario (UPN) aparece en la respuesta SAML como un elemento con el nombre IDPEmail. Este es el valor de UserPrincipalName (UPN) del usuario en Azure AD/Office 365. El UPN está en formato de dirección de correo electrónico. Valor UPN en Windows Office 365 (Azure Active Directory).|
|Emisor|Es necesario que sea un URI del proveedor de identidades. No debe volver a usar el emisor de los mensajes de ejemplo. Si tiene varios dominios de nivel superior en sus inquilinos de Azure AD, el emisor debe coincidir con el valor de URI especificado configurado por dominio.|

>[!IMPORTANT]
>Actualmente, Azure AD admite el siguiente URI de formato NameID para SAML 2.0:urn:oasis:names:tc:SAML:2.0:nameid-format:persistent.

## <a name="sample-saml-request-and-response-messages"></a>Ejemplo de mensajes de solicitud y respuesta de SAML
Se muestra un mensaje de solicitud y respuesta para el intercambio de mensajes de inicio de sesión.
Lo siguiente es un mensaje de solicitud de ejemplo que se envía desde Azure AD a un proveedor de identidades de SAML 2.0 de ejemplo. El proveedor de identidades de SAML 2.0 de ejemplo está configurado por los Servicios de federación de Active Directory (AD FS) para usar el protocolo SAML-P. Las pruebas de interoperabilidad también se han realizado con otros proveedores de identidades de SAML 2.0.

```xml
    <samlp:AuthnRequest 
        xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" 
        xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion" 
        ID="_7171b0b2-19f2-4ba2-8f94-24b5e56b7f1e" 
        IssueInstant="2014-01-30T16:18:35Z" 
        Version="2.0" 
        AssertionConsumerServiceIndex="0" >
            <saml:Issuer>urn:federation:MicrosoftOnline</saml:Issuer>
            <samlp:NameIDPolicy Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent"/>
    </samlp:AuthnRequest>
```

Lo siguiente es un mensaje de respuesta de ejemplo que se envía desde el proveedor de identidades compatible con SAML 2.0 de ejemplo a Azure AD u Office 365.

```xml
    <samlp:Response ID="_592c022f-e85e-4d23-b55b-9141c95cd2a5" Version="2.0" IssueInstant="2014-01-31T15:36:31.357Z" Destination="https://login.microsoftonline.com/login.srf" Consent="urn:oasis:names:tc:SAML:2.0:consent:unspecified" InResponseTo="_049917a6-1183-42fd-a190-1d2cbaf9b144" xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
    <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">http://WS2012R2-0.contoso.com/adfs/services/trust</Issuer>
    <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success" />
    </samlp:Status>
    <Assertion ID="_7e3c1bcd-f180-4f78-83e1-7680920793aa" IssueInstant="2014-01-31T15:36:31.279Z" Version="2.0" xmlns="urn:oasis:names:tc:SAML:2.0:assertion">
    <Issuer>http://WS2012R2-0.contoso.com/adfs/services/trust</Issuer>
    <ds:Signature xmlns:ds="https://www.w3.org/2000/09/xmldsig#">
      <ds:SignedInfo>
        <ds:CanonicalizationMethod Algorithm="https://www.w3.org/2001/10/xml-exc-c14n#" />
        <ds:SignatureMethod Algorithm="https://www.w3.org/2000/09/xmldsig#rsa-sha1" />
        <ds:Reference URI="#_7e3c1bcd-f180-4f78-83e1-7680920793aa">
          <ds:Transforms>
            <ds:Transform Algorithm="https://www.w3.org/2000/09/xmldsig#enveloped-signature" />
            <ds:Transform Algorithm="https://www.w3.org/2001/10/xml-exc-c14n#" />
          </ds:Transforms>
          <ds:DigestMethod Algorithm="https://www.w3.org/2000/09/xmldsig#sha1" />
          <ds:DigestValue>CBn/5YqbheaJP425c0pHva9PhNY=</ds:DigestValue>
        </ds:Reference>
      </ds:SignedInfo>
      <ds:SignatureValue>TciWMyHW2ZODrh/2xrvp5ggmcHBFEd9vrp6DYXp+hZWJzmXMmzwmwS8KNRJKy8H7XqBsdELA1Msqi8I3TmWdnoIRfM/ZAyUppo8suMu6Zw+boE32hoQRnX9EWN/f0vH6zA/YKTzrjca6JQ8gAV1ErwvRWDpyMcwdYCiWALv9ScbkAcebOE1s1JctZ5RBXggdZWrYi72X+I4i6WgyZcIGai/rZ4v2otoWAEHS0y1yh1qT7NDPpl/McDaTGkNU6C+8VfjD78DrUXEcAfKvPgKlKrOMZnD1lCGsViimGY+LSuIdY45MLmyaa5UT4KWph6dA==</ds:SignatureValue>
      <KeyInfo xmlns="https://www.w3.org/2000/09/xmldsig#">
        <ds:X509Data>
          <ds:X509Certificate>MIIC7jCCAdagAwIBAgIQRrjsbFPaXIlOG3GTv50fkjANBgkqhkiG9w0BAQsFADAzMTEwLwYDVQQDEyhBREZTIFNpZ25pbmcgLSBXUzIwMTJSMi0wLnN3aW5mb3JtZXIuY29tMB4XDTE0MDEyMDE1MTY0MFoXDTE1MDEyMDE1MTY0MFowMzExMC8GA1UEAxMoQURGUyBTaWduaW5nIC0gV1MyMDEyUjItMC5zd2luZm9ybWVyLmNvbTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKe+rLVmXy1QwCwZwqgbbp1/+3ZWxd9T/jV0hpLIIWr+LCOHqq8n8beJvlivgLmDJo8f+EITnAxWcsJUvVai/35AhHCUq9tc9sqMp5PWtabAEMb2AU72/QlX/72D2/NbGQq1BWYbqUpgpCZ2nSgvlWDHlCiUo//UGsvfox01kjTFlmqQInsJVfRxF5AcCAwEAATANBgkqhkiG9w0BAQsFAAOCAQEAi8c6C4zaTEc7aQiUgvnGQgCbMZbhUXXLGRpjvFLKaQzkwa9eq7WLJibcSNyGXBa/SfT5wJgsm3TPKgSehGAOTirhcqHheZyvBObAScY7GOT+u9pVYp6raFrc7ez3c+CGHeV/tNvy1hJNs12FYH4X+ZCNFIT9tprieR25NCdi5SWUbPZL0tVzJsHc1y92b2M2FxqRDohxQgJvyJOpcg2mSBzZZIkvDg7gfPSUXHVS1MQs0RHSbwq/XdQocUUhl9/e/YWCbNNxlM84BxFsBUok1dH/gzBySx+Fc8zYi7cOq9yaBT3RLT6cGmFGVYZJW4FyhPZOCLVNsLlnPQcX3dDg9A==</ds:X509Certificate>
        </ds:X509Data>
      </KeyInfo>
    </ds:Signature>
    <Subject>
      <NameID Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent">ABCDEG1234567890</NameID>
      <SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
        <SubjectConfirmationData InResponseTo="_049917a6-1183-42fd-a190-1d2cbaf9b144" NotOnOrAfter="2014-01-31T15:41:31.357Z" Recipient="https://login.microsoftonline.com/login.srf" />
      </SubjectConfirmation>
    </Subject>
    <Conditions NotBefore="2014-01-31T15:36:31.263Z" NotOnOrAfter="2014-01-31T16:36:31.263Z">
      <AudienceRestriction>
        <Audience>urn:federation:MicrosoftOnline</Audience>
      </AudienceRestriction>
    </Conditions>
    <AttributeStatement>
      <Attribute Name="IDPEmail">
        <AttributeValue>administrator@contoso.com</AttributeValue>
      </Attribute>
    </AttributeStatement>
    <AuthnStatement AuthnInstant="2014-01-31T15:36:30.200Z" SessionIndex="_7e3c1bcd-f180-4f78-83e1-7680920793aa">
      <AuthnContext>
        <AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport</AuthnContextClassRef>
      </AuthnContext>
    </AuthnStatement>
    </Assertion>
    </samlp:Response>
```

## <a name="configure-your-saml-20-compliant-identity-provider"></a>Configuración del proveedor de identidades compatible con SAML 2.0
Esta sección contiene instrucciones sobre cómo configurar el proveedor de identidades de SAML 2.0 para la federación con Azure AD con el fin de habilitar el acceso de inicio de sesión único a uno o varios servicios en la nube de Microsoft (como Office 365) mediante el protocolo SAML 2.0. El usuario de confianza de SAML 2.0 que se usa en este escenario para un servicio en la nube de Microsoft es Azure AD.

## <a name="add-azure-ad-metadata"></a>Adición de metadatos de Azure AD
El proveedor de identidades de SAML 2.0 debe adherirse a la información sobre el usuario de confianza de Azure AD. Azure AD publica los metadatos en https://nexus.microsoftonline-p.com/federationmetadata/saml20/federationmetadata.xml.

Se recomienda importar siempre los metadatos más recientes de Azure AD al configurar el proveedor de identidades de SAML 2.0.

>[!NOTE]
>Azure AD no lee los metadatos del proveedor de identidades.

## <a name="add-azure-ad-as-a-relying-party"></a>Adición de Azure AD como usuario de confianza
Deberá habilitar la comunicación entre el proveedor de identidades de SAML 2.0 y Azure AD. Esta configuración dependerá de su proveedor de identidades específico, así que debería consultar la documentación correspondiente. Normalmente el id. del usuario de confianza se establecerá en el mismo valor de entityID de los metadatos de Azure AD.

>[!NOTE]
>Compruebe que el reloj del servidor del proveedor de identidades de SAML 2.0 esté sincronizado con una fuente horaria precisa. Una hora inexacta del reloj puede hacer que los inicios de sesión federados produzcan error.

## <a name="install-windows-powershell-for-sign-on-with-saml-20-identity-provider"></a>Instalación de Windows PowerShell para el inicio de sesión con el proveedor de identidades de SAML 2.0
Después de que ha configurado el proveedor de identidades de SAML 2.0 para su uso con el inicio de sesión de Azure AD, el siguiente paso consiste en descargar e instalar el Módulo Azure Active Directory para Windows PowerShell. Una vez instalado, usará estos cmdlets para configurar los dominios de Azure AD como dominios federados.

El Módulo Azure Active Directory para Windows PowerShell es una descarga para administrar los datos de la organización en Azure AD. Este módulo instala un conjunto de cmdlets en Windows PowerShell; esos cmdlets se ejecutan para configurar el acceso de inicio de sesión único a Azure AD y, a su vez, a todos los servicios en la nube que están suscritos a él. Para instrucciones sobre cómo descargar e instalar los cmdlets, consulte [https://technet.microsoft.com/library/jj151815.aspx](https://technet.microsoft.com/library/jj151815.aspx)

## <a name="set-up-a-trust-between-your-saml-identity-provider-and-azure-ad"></a>Configuración de una relación de confianza entre el proveedor de identidades de SAML y Azure AD
Antes de configurar la federación en un dominio de Azure AD, debe tener configurado un dominio personalizado. No se puede federar el dominio predeterminado proporcionado por Microsoft. El dominio predeterminado de Microsoft finaliza con "onmicrosoft.com".
Ejecutará una serie de cmdlets en la interfaz de la línea de comandos de Windows PowerShell para agregar o convertir dominios para el inicio de sesión único.

Todos los dominios de Azure Active Directory que quiera federar mediante su proveedor de identidades de SAML 2.0 se deben agregar como un dominio de inicio de sesión único o convertirse en uno de estos dominios a partir de un dominio estándar. El hecho de agregar o convertir un dominio establece una relación de confianza entre el proveedor de identidades de SAML 2.0 y Azure AD.

En el procedimiento siguiente se explica cómo convertir un dominio estándar existente en un dominio federado mediante SP-Lite de SAML 2.0. 

>[!NOTE]
>Tenga en cuenta que el dominio puede sufrir una interrupción del sistema que afecte a los usuarios durante un período de tiempo de hasta dos horas después de realizar este paso.

## <a name="configuring-a-domain-in-your-azure-ad-directory-for-federation"></a>Configuración de un dominio en Azure AD Directory para federación


1. Conéctese a su instancia de Azure AD Directory como administrador de inquilinos:

    ```powershell
    Connect-MsolService
    ```
    
2. Configure el dominio de Office 365 deseado para usar la federación con SAML 2.0:

    ```powershell
    $dom = "contoso.com" 
    $BrandName - "Sample SAML 2.0 IDP" 
    $LogOnUrl = "https://WS2012R2-0.contoso.com/passiveLogon" 
    $LogOffUrl = "https://WS2012R2-0.contoso.com/passiveLogOff" 
    $ecpUrl = "https://WS2012R2-0.contoso.com/PAOS" 
    $MyURI = "urn:uri:MySamlp2IDP" 
    $MySigningCert = "MIIC7jCCAdagAwIBAgIQRrjsbFPaXIlOG3GTv50fkjANBgkqhkiG9w0BAQsFADAzMTEwLwYDVQQDEyh BREZTIFNpZ25pbmcgLSBXUzIwMTJSMi0wLnN3aW5mb3JtZXIuY29tMB4XDTE0MDEyMDE1MTY0MFoXDT E1MDEyMDE1MTY0MFowMzExMC8GA1UEAxMoQURGUyBTaWduaW5nIC0gV1MyMDEyUjItMC5zd2luZm9yb WVyLmNvbTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKe+rLVmXy1QwCwZwqgbbp1/kupQ VcjKuKLitVDbssFyqbDTjP7WRjlVMWAHBI3kgNT7oE362Gf2WMJFf1b0HcrsgLin7daRXpq4Qi6OA57 sW1YFMj3sqyuTP0eZV3S4+ZbDVob6amsZIdIwxaLP9Zfywg2bLsGnVldB0+XKedZwDbCLCVg+3ZWxd9 T/jV0hpLIIWr+LCOHqq8n8beJvlivgLmDJo8f+EITnAxWcsJUvVai/35AhHCUq9tc9sqMp5PWtabAEM b2AU72/QlX/72D2/NbGQq1BWYbqUpgpCZ2nSgvlWDHlCiUo//UGsvfox01kjTFlmqQInsJVfRxF5AcC AwEAATANBgkqhkiG9w0BAQsFAAOCAQEAi8c6C4zaTEc7aQiUgvnGQgCbMZbhUXXLGRpjvFLKaQzkwa9 eq7WLJibcSNyGXBa/SfT5wJgsm3TPKgSehGAOTirhcqHheZyvBObAScY7GOT+u9pVYp6raFrc7ez3c+ CGHeV/tNvy1hJNs12FYH4X+ZCNFIT9tprieR25NCdi5SWUbPZL0tVzJsHc1y92b2M2FxqRDohxQgJvy JOpcg2mSBzZZIkvDg7gfPSUXHVS1MQs0RHSbwq/XdQocUUhl9/e/YWCbNNxlM84BxFsBUok1dH/gzBy Sx+Fc8zYi7cOq9yaBT3RLT6cGmFGVYZJW4FyhPZOCLVNsLlnPQcX3dDg9A==" 
    $uri = "http://WS2012R2-0.contoso.com/adfs/services/trust" 
    $Protocol = "SAMLP" 
    Set-MsolDomainAuthentication `
        -DomainName $dom `
        -FederationBrandName $BrandName `
        -Authentication Federated `
        -PassiveLogOnUri $LogOnUrl `
        -ActiveLogOnUri $ecpUrl `
        -SigningCertificate $MySigningCert `
        -IssuerUri $MyURI `
        -LogOffUri $LogOffUrl `
        -PreferredAuthenticationProtocol $Protocol
    ``` 

3.  Puede obtener la cadena codificada en base64 del certificado de firma de su archivo de metadatos de IDP. Aunque se ha proporcionado un ejemplo de esta ubicación, puede ser algo diferente dependiendo de su implementación.

    ```xml
    <IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
        <KeyDescriptor use="signing">
          <KeyInfo xmlns="https://www.w3.org/2000/09/xmldsig#">
             <X509Data>
                 <X509Certificate> MIIC5jCCAc6gAwIBAgIQLnaxUPzay6ZJsC8HVv/QfTANBgkqhkiG9w0BAQsFADAvMS0wKwYDVQQDEyRBREZTIFNpZ25pbmcgLSBmcy50ZWNobGFiY2VudHJhbC5vcmcwHhcNMTMxMTA0MTgxMzMyWhcNMTQxMTA0MTgxMzMyWjAvMS0wKwYDVQQDEyRBREZTIFNpZ25pbmcgLSBmcy50ZWNobGFiY2VudHJhbC5vcmcwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCwMdVLTr5YTSRp+ccbSpuuFeXMfABD9mVCi2wtkRwC30TIyPdORz642MkurdxdPCWjwgJ0HW6TvXwcO9afH3OC5V//wEGDoNcI8PV4enCzTYFe/h//w51uqyv48Fbb3lEXs+aVl8155OAj2sO9IX64OJWKey82GQWK3g7LfhWWpp17j5bKpSd9DBH5pvrV+Q1ESU3mx71TEOvikHGCZYitEPywNeVMLRKrevdWI3FAhFjcCSO6nWDiMqCqiTDYOURXIcHVYTSof1YotkJ4tG6mP5Kpjzd4VQvnR7Pjb47nhIYG6iZ3mR1F85Ns9+hBWukQWNN2hcD/uGdPXhpdMVpBAgMBAAEwDQYJKoZIhvcNAQELBQADggEBAK7h7jF7wPzhZ1dPl4e+XMAr8I7TNbhgEU3+oxKyW/IioQbvZVw1mYVCbGq9Rsw4KE06eSMybqHln3w5EeBbLS0MEkApqHY+p68iRpguqa+W7UHKXXQVgPMCpqxMFKonX6VlSQOR64FgpBme2uG+LJ8reTgypEKspQIN0WvtPWmiq4zAwBp08hAacgv868c0MM4WbOYU0rzMIR6Q+ceGVRImlCwZ5b7XKp4mJZ9hlaRjeuyVrDuzBkzROSurX1OXoci08yJvhbtiBJLf3uPOJHrhjKRwIt2TnzS9ElgFZlJiDIA26Athe73n43CT0af2IG6yC7e6sK4L3NEXJrwwUZk=</X509Certificate>
              </X509Data>
            </KeyInfo>
        </KeyDescriptor>
    </IDPSSODescriptor>
    ``` 

Para más información sobre "Set-MsolDomainAuthentication", consulte: [https://technet.microsoft.com/library/dn194112.aspx](https://technet.microsoft.com/library/dn194112.aspx).

>[!NOTE]
>Solo debe usar `$ecpUrl = "https://WS2012R2-0.contoso.com/PAOS"` si configura una extensión ECP para el proveedor de identidades. Los clientes de Exchange Online, excepto Outlook Web Application (OWA), dependen de un punto de conexión activo basado en POST. Si el STS de SAML 2.0 implementa un punto de conexión activo de forma similar a ECP de Shibboleth, es posible que estos clientes enriquecidos puedan interactuar con el servicio Exchange Online.

Una vez configurada la federación, puede cambiar de nuevo a "no federado" (o "administrado"); sin embargo, este cambio tarda hasta dos horas en realizarse y requiere la asignación de nuevas contraseñas aleatorias para el inicio de sesión basado en la nube de cada usuario. Volver a cambiar a "administrado" puede ser necesario en algunos escenarios para restablecer un error de la configuración. Para más información sobre la conversión de dominios, consulte: [https://msdn.microsoft.com/library/windowsazure/dn194122.aspx](https://msdn.microsoft.com/library/windowsazure/dn194122.aspx).

## <a name="provision-user-principals-to-azure-ad--office-365"></a>Aprovisionamiento de entidades de seguridad de usuario en Azure AD y Office 365
Antes de poder autenticar a los usuarios en Office 365, debe aprovisionar Azure AD con entidades de seguridad de usuario que correspondan a la aserción en la notificación de SAML 2.0. Si Azure AD no conoce estas entidades de seguridad de usuario de antemano, no se podrán usar para el inicio de sesión federado. Se puede usar Azure AD Connect o Windows PowerShell para aprovisionar a las entidades de seguridad de usuario.

Azure AD Connect se puede usar para aprovisionar a las entidades de seguridad en los dominios en Azure Active Directory desde el entorno local de Active Directory. Para más información, consulte [Integrate your on-premises directories with Azure Active Directory](whatis-hybrid-identity.md) (Integración de los directorios locales con Azure Active Directory).

También se puede usar Windows PowerShell para automatizar la adición de nuevos usuarios a Azure AD y sincronizar los cambios desde el directorio local. Para usar los cmdlets de Windows PowerShell, debe descargar los [módulos de Azure Active Directory](https://docs.microsoft.com/powershell/azure/active-directory/install-adv2?view=azureadps-2.0).

Este procedimiento muestra cómo agregar un único usuario a Azure AD.


1. Conéctese a su instancia de Azure AD Directory como administrador de inquilinos: Connect-MsolService.
2. Cree una nueva entidad de seguridad de usuario:

    ```powershell
    New-MsolUser `
      -UserPrincipalName elwoodf1@contoso.com `
      -ImmutableId ABCDEFG1234567890 `
      -DisplayName "Elwood Folk" `
      -FirstName Elwood `
      -LastName Folk `
      -AlternateEmailAddresses "Elwood.Folk@contoso.com" `
      -LicenseAssignment "samlp2test:ENTERPRISEPACK" `
      -UsageLocation "US" 
    ```

Para obtener más información sobre "New-MsolUser", consulte [https://technet.microsoft.com/library/dn194096.aspx](https://technet.microsoft.com/library/dn194096.aspx).

>[!NOTE]
>El valor "UserPrinciplName" debe coincidir con el valor que se enviará para "IDPEmail" en la notificación de SAML 2.0, y el valor "ImmutableID" debe coincidir con el valor enviado en la aserción "NameID".

## <a name="verify-single-sign-on-with-your-saml-20-idp"></a>Comprobación del inicio de sesión único con el IDP SAML 2.0
Como administrador, antes de comprobar y administrar el inicio de sesión único (también llamado federación de identidades), revise la información y realice los pasos que se describen en los siguientes artículos para configurar el inicio de sesión único con su proveedor de identidades basado en SP-Lite de SAML 2.0:


1.  Ha revisado los requisitos del protocolo SAML 2.0 de Azure AD.
2.  Ha configurado el proveedor de identidades de SAML 2.0.
3.  Instale Windows PowerShell para el inicio de sesión único con el proveedor de identidades de SAML 2.0.
4.  Configure una relación de confianza entre el proveedor de identidades de SAML 2.0 y Azure AD.
5.  Ha aprovisionado una entidad de seguridad de usuario de prueba conocida en Azure Active Directory (Office 365) mediante Windows PowerShell o Azure AD Connect.
6.  Configure la sincronización de directorios con [Azure AD Connect](whatis-hybrid-identity.md).

Después de configurar el inicio de sesión único con el proveedor de identidades basado en SP-Lite de SAML 2.0, debe comprobar que funciona correctamente.

>[!NOTE]
>Si ha convertido un dominio, en lugar de agregar uno, puede tardar hasta 24 horas en configurar el inicio de sesión único.
Antes de comprobar el inicio de sesión único, debe finalizar la configuración de la sincronización de Active Directory, sincronizar los directorios y activar los usuarios sincronizados.

### <a name="use-the-tool-to-verify-that-single-sign-on-has-been-set-up-correctly"></a>Use la herramienta para comprobar que el inicio de sesión único se ha configurado correctamente
Para comprobar que el inicio de sesión único se ha configurado correctamente, puede realizar el procedimiento siguiente para confirmar que puede iniciar sesión en el servicio en la nube con sus credenciales corporativas.

Microsoft ha proporcionado una herramienta que puede usar para comprobar el proveedor de identidades basado en SAML 2.0. Antes de ejecutar la herramienta de prueba, debe haber configurado un inquilino de Azure AD para federarlo con el proveedor de identidades.

>[!NOTE]
>El Analizador de conectividad requiere Internet Explorer 10 o una versión posterior.



1. Descargue el Analizador de conectividad en [https://testconnectivity.microsoft.com/?tabid=Client](https://testconnectivity.microsoft.com/?tabid=Client).
2.  Haga clic en Instalar ahora para empezar a descargar e instalar la herramienta.
3.  Seleccione “I can’t set up federation with Office 365, Azure, or other services that use Azure Active Directory” (No se puede configurar la federación con Office 365, Azure u otros servicios que usan Azure Active Directory).
4.  Una vez que haya descargado la herramienta y esté en funcionamiento, verá la ventana de diagnóstico de conectividad. La herramienta le lleva por los pasos para probar la conexión de federación.
5.  El Analizador de conectividad abre el IDP de SAML 2.0 para que inicie sesión; escriba las credenciales de la entidad de seguridad de usuario que va a probar: ![SAML](./media/how-to-connect-fed-saml-idp/saml1.png)
6.  En la ventana de inicio de sesión de la prueba de federación, debe escribir un nombre de cuenta y una contraseña para el inquilino de Azure AD que está configurado para federarse con el proveedor de identidades de SAML 2.0. La herramienta intentará iniciar sesión con esas credenciales y se mostrará una salida con resultados detallados de las pruebas realizadas durante el intento de inicio de sesión.
![SAML](./media/how-to-connect-fed-saml-idp/saml2.png)
7. En esta ventana se muestra un resultado erróneo de las pruebas. Al hacer clic en Review detailed results (Revisar resultados detallados), se muestra información sobre los resultados de cada prueba que se ha realizado. También puede guardar los resultados en el disco para compartirlos.
 
>[!NOTE]
>El Analizador de conectividad también prueba la federación activa mediante los protocolos ECP/PAOS y basados en WS*. Si no los está usando, puede omitir el siguiente error: Se está probando el flujo de inicio de sesión activo con el punto de conexión de federación activa de su proveedor de identidades.

### <a name="manually-verify-that-single-sign-on-has-been-set-up-correctly"></a>Comprobación manual de que el inicio de sesión único se ha configurado correctamente
La comprobación manual proporciona pasos adicionales que puede realizar para asegurarse de que el proveedor de identidades de SAML 2.0 funciona correctamente en muchos escenarios.
Para comprobar que ese inicio de sesión único se ha configurado correctamente, realice los pasos siguientes:


1. En un equipo unido a un dominio, inicie sesión en su servicio en la nube con el mismo nombre de inicio de sesión que usa en las credenciales corporativas.
2.  Haga clic dentro del cuadro de contraseña. Si el inicio de sesión único está configurado, el cuadro de contraseña aparecerá sombreado y verá el siguiente mensaje: "Ahora debe iniciar sesión en &lt;su empresa&gt;".
3.  Haga clic en Iniciar sesión en el vínculo de &lt;su empresa&gt;. Si puede iniciar sesión, significa que el inicio de sesión único está configurado.

## <a name="next-steps"></a>Pasos siguientes


- [Servicios de federación de Active Directory y personalización con Azure AD Connect](how-to-connect-fed-management.md)
- [Lista de compatibilidad de federación de AD Azure](how-to-connect-fed-compatibility.md)
- [Instalación personalizada de Azure AD Connect](how-to-connect-install-custom.md)
