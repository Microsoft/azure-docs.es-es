---
title: Configuración de la directiva de autorización de claves mediante el SDK de Media Services para .NET | Microsoft Docs
description: Aprenda a configurar una directiva de autorización para una clave de contenido mediante el SDK de Media Services para .NET.
services: media-services
documentationcenter: ''
author: mingfeiy
manager: femila
editor: ''
ms.assetid: 1a0aedda-5b87-4436-8193-09fc2f14310c
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: e05c7a8388b662731c04ac4098d6eecb1d89fe4f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87060151"
---
# <a name="configure-a-content-key-authorization-policy-by-using-the-media-services-net-sdk"></a>Configuración de la directiva de autorización de claves mediante el SDK de Media Services para .NET

[!INCLUDE [media-services-selector-content-key-auth-policy](../../../includes/media-services-selector-content-key-auth-policy.md)]

## <a name="overview"></a>Información general
 Puede usar Azure Media Services para entregar transmisiones MPEG-DASH, Smooth Streaming y HTTP Live Streaming (HLS) protegidas con Estándar de cifrado avanzado (AES) mediante el uso de claves de cifrado de 128 bits o [administración de derechos digitales (DRM) de PlayReady](https://www.microsoft.com/playready/overview/). Con Media Services, también puede entregar transmisiones DASH cifradas con DRM de Widevine. Tanto PlayReady como Widevine se cifran según la especificación de cifrado común (ISO/IEC 23001-7 CENC).

Media Services también proporciona un servicio de entrega de claves/licencias con el que los clientes pueden obtener claves AES o licencias de PlayReady/Widevine con el fin de reproducir el contenido cifrado.

Si quiere que Media Services cifre un recurso, debe asociar una clave de cifrado (CommonEncryption o EnvelopeEncryption) con el recurso. Para más información, consulte [Creación de claves de contenido con .NET](media-services-dotnet-create-contentkey.md). También debe configurar las directivas de autorización de la clave, tal como se describe en este artículo.

Cuando un reproductor solicita una transmisión, Media Services usa la clave especificada para cifrar de forma dinámica el contenido mediante el cifrado AES o DRM. Para descifrar la secuencia, el reproductor solicitará la clave del servicio de entrega de claves. Para determinar si el usuario tiene permiso para obtener la clave, el servicio evalúa las directivas de autorización que especificó para la clave.

Media Services admite varias formas de autenticar a los usuarios que realizan solicitudes de clave. La directiva de autorización de claves de contenido puede tener una o varias restricciones de autorización. Las opciones son restricción de token o abierta. La directiva con restricción de token debe ir acompañada de un token emitido por un servicio de token de seguridad (STS). Media Services admite tokens en formato Token web simple ([SWT](/previous-versions/azure/azure-services/gg185950(v=azure.100)#BKMK_2)) y en formato JSON Web Token ([JWT](/previous-versions/azure/azure-services/gg185950(v=azure.100)#BKMK_3)).

Tenga en cuenta que Media Services no proporciona STS. Puede crear un STS personalizado o usar Azure Access Control Service para emitir tokens. El STS debe configurarse para crear un token firmado con las notificaciones especificadas de clave y el número que especificó en la configuración de restricción de token (como se describe en este artículo). Si el token es válido y las notificaciones del token coinciden con las que se han configurado en la clave de contenido, el servicio de entrega de claves de Media Services devuelve la clave de cifrado al cliente.

Para más información, consulte los siguientes artículos.

- [Autenticación de token JWT](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/)
- [Integración de la aplicación OWIN basada en MVC de Azure Media Services con Azure Active Directory y restricción de la entrega de claves de contenido basada en notificaciones de JWT](http://www.gtrifonov.com/2015/01/24/mvc-owin-azure-media-services-ad-integration/)

### <a name="some-considerations-apply"></a>Se aplican algunas consideraciones
* Cuando se crea la cuenta de Media Services, se agrega un punto de conexión de streaming predeterminado a la cuenta en estado Detenido. Para iniciar la transmisión del contenido y aprovechar el empaquetado dinámico y el cifrado dinámico, el punto de conexión de streaming debe estar en estado "Running" (En ejecución). 
* El recurso debe contener un conjunto de archivos MP4 de velocidad de bits adaptable o archivos Smooth Streaming de velocidad de bits adaptable. Para obtener más información, consulte [Codificación de un recurso](media-services-encode-asset.md).
* Cargue y codifique los recursos con la opción AssetCreationOptions.StorageEncrypted.
* Si planea tener varias claves de contenido que requieran la misma configuración de directiva, se recomienda crear una sola directiva de autorización y volverla a utilizar con varias claves de contenido.
* El servicio de entrega de claves almacena en caché ContentKeyAuthorizationPolicy y los objetos relacionados (opciones y restricciones de directiva) durante 15 minutos. Puede crear ContentKeyAuthorizationPolicy, especificar que se utilice una restricción de token, probarla y actualizar la directiva con la restricción abierta. Este proceso tarda aproximadamente unos 15 minutos antes de que la directiva cambie a su versión abierta.
* Si agrega o actualiza la directiva de entrega de recursos, debe eliminar un localizador existente y crear uno nuevo.
* En este momento no se pueden cifrar las descargas progresivas.
* Un punto de conexión de streaming de Media Services establece el valor del encabezado CORS "Access-Control-Allow-Origin" en la respuesta de la comprobación preliminar como el comodín "\*". Este valor funciona bien con la mayoría de reproductores, incluyendo Azure Media Player, Roku y JWPlayer, entre otros. Sin embargo, algunos reproductores que usan dash.js no funcionan porque, con el modo de las credenciales establecido en "include", el elemento XMLHttpRequest de sus archivos dash.js no permite el comodín "\*" como valor de "Access-Control-Allow-Origin". Como solución alternativa a esta limitación de los archivos dash.js, si hospeda al cliente desde un solo dominio, Media Services puede especificar ese dominio en el encabezado de respuesta de la comprobación preliminar. Para obtener ayuda, abra una incidencia de soporte técnico en Azure Portal.

## <a name="aes-128-dynamic-encryption"></a>Cifrado dinámico AES-128
### <a name="open-restriction"></a>Restricción open
La restricción open significa que el sistema entrega la clave a cualquier persona que realice una solicitud de clave. Esta restricción puede ser útil para realizar pruebas.

En el ejemplo siguiente se crea una directiva de autorización abierta y se agrega a la clave de contenido:
```csharp
    static public void AddOpenAuthorizationPolicy(IContentKey contentKey)
    {
        // Create ContentKeyAuthorizationPolicy with Open restrictions
        // and create authorization policy
        IContentKeyAuthorizationPolicy policy = _context.
        ContentKeyAuthorizationPolicies.
        CreateAsync("Open Authorization Policy").Result;
        
        List<ContentKeyAuthorizationPolicyRestriction> restrictions =
            new List<ContentKeyAuthorizationPolicyRestriction>();

        ContentKeyAuthorizationPolicyRestriction restriction =
            new ContentKeyAuthorizationPolicyRestriction
            {
                Name = "HLS Open Authorization Policy",
                KeyRestrictionType = (int)ContentKeyRestrictionType.Open,
                Requirements = null // no requirements needed for HLS
            };

        restrictions.Add(restriction);

        IContentKeyAuthorizationPolicyOption policyOption =
            _context.ContentKeyAuthorizationPolicyOptions.Create(
            "policy", 
            ContentKeyDeliveryType.BaselineHttp, 
            restrictions, 
            "");

        policy.Options.Add(policyOption);

        // Add ContentKeyAuthorizationPolicy to ContentKey
        contentKey.AuthorizationPolicyId = policy.Id;
        IContentKey updatedKey = contentKey.UpdateAsync().Result;
        Console.WriteLine("Adding Key to Asset: Key ID is " + updatedKey.Id);
    }
```

### <a name="token-restriction"></a>Restricción de token
En esta sección se describe cómo crear una directiva de autorización de claves de contenido y asociarla a la clave de contenido. La directiva de autorización describe los requisitos de autorización que se deben cumplir para determinar si el usuario está autorizado para recibir la clave. Por ejemplo, ¿la lista de claves de comprobación contiene la clave con la que se firmó el token?

Para configurar la opción de restricción del token, debe usar un archivo XML para describir los requisitos de autorización del token. El archivo XML de configuración de restricción de token debe cumplir el siguiente esquema XML:
```csharp
#### Token restriction schema
    <?xml version="1.0" encoding="utf-8"?>
    <xs:schema xmlns:tns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/TokenRestrictionTemplate/v1" elementFormDefault="qualified" targetNamespace="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/TokenRestrictionTemplate/v1" xmlns:xs="https://www.w3.org/2001/XMLSchema">
      <xs:complexType name="TokenClaim">
        <xs:sequence>
          <xs:element name="ClaimType" nillable="true" type="xs:string" />
          <xs:element minOccurs="0" name="ClaimValue" nillable="true" type="xs:string" />
        </xs:sequence>
      </xs:complexType>
      <xs:element name="TokenClaim" nillable="true" type="tns:TokenClaim" />
      <xs:complexType name="TokenRestrictionTemplate">
        <xs:sequence>
          <xs:element minOccurs="0" name="AlternateVerificationKeys" nillable="true" type="tns:ArrayOfTokenVerificationKey" />
          <xs:element name="Audience" nillable="true" type="xs:anyURI" />
          <xs:element name="Issuer" nillable="true" type="xs:anyURI" />
          <xs:element name="PrimaryVerificationKey" nillable="true" type="tns:TokenVerificationKey" />
          <xs:element minOccurs="0" name="RequiredClaims" nillable="true" type="tns:ArrayOfTokenClaim" />
        </xs:sequence>
      </xs:complexType>
      <xs:element name="TokenRestrictionTemplate" nillable="true" type="tns:TokenRestrictionTemplate" />
      <xs:complexType name="ArrayOfTokenVerificationKey">
        <xs:sequence>
          <xs:element minOccurs="0" maxOccurs="unbounded" name="TokenVerificationKey" nillable="true" type="tns:TokenVerificationKey" />
        </xs:sequence>
      </xs:complexType>
      <xs:element name="ArrayOfTokenVerificationKey" nillable="true" type="tns:ArrayOfTokenVerificationKey" />
      <xs:complexType name="TokenVerificationKey">
        <xs:sequence />
      </xs:complexType>
      <xs:element name="TokenVerificationKey" nillable="true" type="tns:TokenVerificationKey" />
      <xs:complexType name="ArrayOfTokenClaim">
        <xs:sequence>
          <xs:element minOccurs="0" maxOccurs="unbounded" name="TokenClaim" nillable="true" type="tns:TokenClaim" />
        </xs:sequence>
      </xs:complexType>
      <xs:element name="ArrayOfTokenClaim" nillable="true" type="tns:ArrayOfTokenClaim" />
      <xs:complexType name="SymmetricVerificationKey">
        <xs:complexContent mixed="false">
          <xs:extension base="tns:TokenVerificationKey">
            <xs:sequence>
              <xs:element name="KeyValue" nillable="true" type="xs:base64Binary" />
            </xs:sequence>
          </xs:extension>
        </xs:complexContent>
      </xs:complexType>
      <xs:element name="SymmetricVerificationKey" nillable="true" type="tns:SymmetricVerificationKey" />
    </xs:schema>
```
Al configurar la directiva de restricción de token, debe especificar los parámetros de clave de comprobación principal, el emisor y el público. La clave de comprobación principal contiene la clave con la que se firmó el token. El emisor es el STS que emite el token. El público (a veces denominado ámbito) describe la intención del token o del recurso cuyo acceso está autorizado por el token. El servicio de entrega de claves de los Media Services valida que estos valores del token coincidan con los valores de la plantilla.

Al utilizar el SDK de Media Services para .NET, puede usar la clase TokenRestrictionTemplate para generar el token de restricción.
En el ejemplo siguiente se crea una directiva de autorización con una restricción de token. En este ejemplo, el cliente tiene que presentar un token que contenga una clave de firma (VerificationKey), un emisor de tokens y las notificaciones necesarias.
```csharp
    public static string AddTokenRestrictedAuthorizationPolicy(IContentKey contentKey)
    {
        string tokenTemplateString = GenerateTokenRequirements();

        IContentKeyAuthorizationPolicy policy = _context.
                                ContentKeyAuthorizationPolicies.
                                CreateAsync("HLS token restricted authorization policy").Result;

        List<ContentKeyAuthorizationPolicyRestriction> restrictions =
                new List<ContentKeyAuthorizationPolicyRestriction>();

        ContentKeyAuthorizationPolicyRestriction restriction =
                new ContentKeyAuthorizationPolicyRestriction
                {
                    Name = "Token Authorization Policy",
                    KeyRestrictionType = (int)ContentKeyRestrictionType.TokenRestricted,
                    Requirements = tokenTemplateString
                };

        restrictions.Add(restriction);

        //You could have multiple options 
        IContentKeyAuthorizationPolicyOption policyOption =
            _context.ContentKeyAuthorizationPolicyOptions.Create(
                "Token option for HLS",
                ContentKeyDeliveryType.BaselineHttp,
                restrictions,
                null  // no key delivery data is needed for HLS
                );

        policy.Options.Add(policyOption);

        // Add ContentKeyAuthorizationPolicy to ContentKey
        contentKey.AuthorizationPolicyId = policy.Id;
        IContentKey updatedKey = contentKey.UpdateAsync().Result;
        Console.WriteLine("Adding Key to Asset: Key ID is " + updatedKey.Id);

        return tokenTemplateString;
    }

    static private string GenerateTokenRequirements()
    {
        TokenRestrictionTemplate template = new TokenRestrictionTemplate(TokenType.SWT);

        template.PrimaryVerificationKey = new SymmetricVerificationKey();
        template.AlternateVerificationKeys.Add(new SymmetricVerificationKey());
            template.Audience = _sampleAudience.ToString();
            template.Issuer = _sampleIssuer.ToString();

        template.RequiredClaims.Add(TokenClaim.ContentKeyIdentifierClaim);

        return TokenRestrictionTemplateSerializer.Serialize(template);
    }
```
#### <a name="test-token"></a>Token de prueba
Para obtener un token de prueba basado en la restricción de token que se usó para la directiva de autorización de claves, haga lo siguiente:
```csharp
    // Deserializes a string containing an Xml representation of a TokenRestrictionTemplate
    // back into a TokenRestrictionTemplate class instance.
    TokenRestrictionTemplate tokenTemplate =
        TokenRestrictionTemplateSerializer.Deserialize(tokenTemplateString);

    // Generate a test token based on the data in the given TokenRestrictionTemplate.
    // Note, you need to pass the key id Guid because we specified 
    // TokenClaim.ContentKeyIdentifierClaim in during the creation of TokenRestrictionTemplate.
    Guid rawkey = EncryptionUtils.GetKeyIdAsGuid(key.Id);

    //The GenerateTestToken method returns the token without the word “Bearer” in front
    //so you have to add it in front of the token string. 
    string testToken = TokenRestrictionTemplateSerializer.GenerateTestToken(tokenTemplate, null, rawkey);
    Console.WriteLine("The authorization token is:\nBearer {0}", testToken);
    Console.WriteLine();
```

## <a name="playready-dynamic-encryption"></a>Cifrado dinámico de PlayReady
Puede usar Media Services para configurar los derechos y restricciones que quiera que el entorno de ejecución de PlayReady DRM aplique cuando un usuario intente reproducir contenido protegido. 

Al proteger su contenido con PlayReady, una de las cosas que debe especificar en la directiva de autorización es una cadena XML que defina la [plantilla de licencias PlayReady](media-services-playready-license-template-overview.md). En el SDK de Media Services para .NET, las clases PlayReadyLicenseResponseTemplate y PlayReadyLicenseTemplate lo ayudan a definir la plantilla de licencia de PlayReady.

Para más información sobre cómo cifrar el contenido con PlayReady y Widevine, consulte [Uso de cifrado dinámico común de PlayReady o Widevine](media-services-protect-with-playready-widevine.md).

### <a name="open-restriction"></a>Restricción open
La restricción open significa que el sistema entrega la clave a cualquier persona que realice una solicitud de clave. Esta restricción puede ser útil para realizar pruebas.

En el ejemplo siguiente se crea una directiva de autorización abierta y se agrega a la clave de contenido:

```csharp
    static public void AddOpenAuthorizationPolicy(IContentKey contentKey)
    {

        // Create ContentKeyAuthorizationPolicy with Open restrictions 
        // and create authorization policy          

        List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
        {
            new ContentKeyAuthorizationPolicyRestriction 
            { 
                Name = "Open", 
                KeyRestrictionType = (int)ContentKeyRestrictionType.Open, 
                Requirements = null
            }
        };

        // Configure PlayReady license template.
        string newLicenseTemplate = ConfigurePlayReadyLicenseTemplate();

        IContentKeyAuthorizationPolicyOption policyOption =
            _context.ContentKeyAuthorizationPolicyOptions.Create("",
                ContentKeyDeliveryType.PlayReadyLicense,
                    restrictions, newLicenseTemplate);

        IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
                    ContentKeyAuthorizationPolicies.
                    CreateAsync("Deliver Common Content Key with no restrictions").
                    Result;


        contentKeyAuthorizationPolicy.Options.Add(policyOption);

        // Associate the content key authorization policy with the content key.
        contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
        contentKey = contentKey.UpdateAsync().Result;
    }
```

### <a name="token-restriction"></a>Restricción de token
Para configurar la opción de restricción del token, debe usar un archivo XML para describir los requisitos de autorización del token. El archivo XML de configuración de restricciones de token debe cumplir el esquema XML que se muestra en la sección "Esquema de restricciones de token".

```csharp
    public static string AddTokenRestrictedAuthorizationPolicy(IContentKey contentKey)
    {
        string tokenTemplateString = GenerateTokenRequirements();

        IContentKeyAuthorizationPolicy policy = _context.
                                ContentKeyAuthorizationPolicies.
                                CreateAsync("HLS token restricted authorization policy").Result;

        List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
        {
            new ContentKeyAuthorizationPolicyRestriction 
            { 
                Name = "Token Authorization Policy", 
                KeyRestrictionType = (int)ContentKeyRestrictionType.TokenRestricted,
                Requirements = tokenTemplateString, 
            }
        };

        // Configure PlayReady license template.
        string newLicenseTemplate = ConfigurePlayReadyLicenseTemplate();

        IContentKeyAuthorizationPolicyOption policyOption =
            _context.ContentKeyAuthorizationPolicyOptions.Create("Token option",
                ContentKeyDeliveryType.PlayReadyLicense,
                    restrictions, newLicenseTemplate);

        IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
                    ContentKeyAuthorizationPolicies.
                    CreateAsync("Deliver Common Content Key with no restrictions").
                    Result;

        policy.Options.Add(policyOption);

        // Add ContentKeyAuthorizationPolicy to ContentKey
        contentKeyAuthorizationPolicy.Options.Add(policyOption);

        // Associate the content key authorization policy with the content key
        contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
        contentKey = contentKey.UpdateAsync().Result;

        return tokenTemplateString;
    }

    static private string GenerateTokenRequirements()
    {

        TokenRestrictionTemplate template = new TokenRestrictionTemplate(TokenType.SWT);

        template.PrimaryVerificationKey = new SymmetricVerificationKey();
        template.AlternateVerificationKeys.Add(new SymmetricVerificationKey());
            template.Audience = _sampleAudience.ToString();
            template.Issuer = _sampleIssuer.ToString();


        template.RequiredClaims.Add(TokenClaim.ContentKeyIdentifierClaim);

        return TokenRestrictionTemplateSerializer.Serialize(template);
    } 

    static private string ConfigurePlayReadyLicenseTemplate()
    {
        // The following code configures PlayReady License Template using .NET classes
        // and returns the XML string.

        //The PlayReadyLicenseResponseTemplate class represents the template for the response sent back to the end user. 
        //It contains a field for a custom data string between the license server and the application 
        //(may be useful for custom app logic) as well as a list of one or more license templates.
        PlayReadyLicenseResponseTemplate responseTemplate = new PlayReadyLicenseResponseTemplate();

        // The PlayReadyLicenseTemplate class represents a license template for creating PlayReady licenses
        // to be returned to the end users. 
        //It contains the data on the content key in the license and any rights or restrictions to be 
        //enforced by the PlayReady DRM runtime when using the content key.
        PlayReadyLicenseTemplate licenseTemplate = new PlayReadyLicenseTemplate();
        //Configure whether the license is persistent (saved in persistent storage on the client) 
        //or non-persistent (only held in memory while the player is using the license).  
        licenseTemplate.LicenseType = PlayReadyLicenseType.Nonpersistent;

        // AllowTestDevices controls whether test devices can use the license or not.  
        // If true, the MinimumSecurityLevel property of the license
        // is set to 150.  If false (the default), the MinimumSecurityLevel property of the license is set to 2000.
        licenseTemplate.AllowTestDevices = true;


        // You can also configure the Play Right in the PlayReady license by using the PlayReadyPlayRight class. 
        // It grants the user the ability to play back the content subject to the zero or more restrictions 
        // configured in the license and on the PlayRight itself (for playback specific policy). 
        // Much of the policy on the PlayRight has to do with output restrictions 
        // which control the types of outputs that the content can be played over and 
        // any restrictions that must be put in place when using a given output.
        // For example, if the DigitalVideoOnlyContentRestriction is enabled, 
        //then the DRM runtime will only allow the video to be displayed over digital outputs 
        //(analog video outputs won’t be allowed to pass the content).

        //IMPORTANT: These types of restrictions can be very powerful but can also affect the consumer experience. 
        // If the output protections are configured too restrictive, 
        // the content might be unplayable on some clients. For more information, see the PlayReady Compliance Rules document.

        // For example:
        //licenseTemplate.PlayRight.AgcAndColorStripeRestriction = new AgcAndColorStripeRestriction(1);

        responseTemplate.LicenseTemplates.Add(licenseTemplate);

        return MediaServicesLicenseTemplateSerializer.Serialize(responseTemplate);
    }
```

Para obtener un token de prueba basado en la restricción de token que se usó para la directiva de autorización de claves, consulte la sección [Token de prueba](#test-token). 

## <a name="types-used-when-you-define-contentkeyauthorizationpolicy"></a><a id="types"></a>Tipos que se usan al definir ContentKeyAuthorizationPolicy
### <a name="contentkeyrestrictiontype"></a><a id="ContentKeyRestrictionType"></a>ContentKeyRestrictionType

```csharp
    public enum ContentKeyRestrictionType
    {
        Open = 0,
        TokenRestricted = 1,
        IPRestricted = 2,
    }
```

### <a name="contentkeydeliverytype"></a><a id="ContentKeyDeliveryType"></a>ContentKeyDeliveryType

```csharp 
    public enum ContentKeyDeliveryType
    {
      None = 0,
      PlayReadyLicense = 1,
      BaselineHttp = 2,
      Widevine = 3
    }
```

### <a name="tokentype"></a><a id="TokenType"></a>TokenType

```csharp
    public enum TokenType
    {
        Undefined = 0,
        SWT = 1,
        JWT = 2,
    }
```

## <a name="additional-notes"></a>Notas adicionales

* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Pasos siguientes
Ahora que ha configurado la directiva de autorización de la clave de contenido, consulte [Configuración de directivas de entrega de recursos](media-services-dotnet-configure-asset-delivery-policy.md).
