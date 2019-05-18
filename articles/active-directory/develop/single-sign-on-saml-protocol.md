---
title: Protocolo SAML de inicio de sesión único de Azure | Microsoft Docs
description: En este artículo se describe el protocolo SAML de inicio de sesión único de Azure Active Directory.
services: active-directory
documentationcenter: .net
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: ad8437f5-b887-41ff-bd77-779ddafc33fb
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/19/2017
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: hirsin
ms.collection: M365-identity-device-management
ms.openlocfilehash: 593f07b27fec16c3df90a073479effb130bc5721
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545275"
---
# <a name="single-sign-on-saml-protocol"></a>Protocolo SAML de inicio de sesión único

Este artículo se centra en las solicitudes y las respuestas de autenticación SAML 2.0 de Azure Active Directory (Azure AD) compatibles con el inicio de sesión único.

En el siguiente diagrama de protocolo se describe la secuencia de inicio de sesión único. El servicio en la nube (proveedor de servicios) utiliza un enlace de redirección HTTP para pasar un elemento `AuthnRequest` (solicitud de autenticación) a Azure AD (el proveedor de identidades). Después, Azure AD utiliza un enlace HTTP POST para registrar un elemento `Response` en el servicio en la nube.

![Flujo de trabajo de inicio de sesión único](./media/single-sign-on-saml-protocol/active-directory-saml-single-sign-on-workflow.png)

## <a name="authnrequest"></a>AuthnRequest

Para solicitar una autenticación de usuario, los servicios en la nube envían un elemento `AuthnRequest` a Azure AD. Una muestra de SAML 2.0 `AuthnRequest` debería tener un aspecto similar al ejemplo siguiente:

```
<samlp:AuthnRequest
xmlns="urn:oasis:names:tc:SAML:2.0:metadata"
ID="id6c1c178c166d486687be4aaf5e482730"
Version="2.0" IssueInstant="2013-03-18T03:28:54.1839884Z"
xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
<Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://www.contoso.com</Issuer>
</samlp:AuthnRequest>
```

| Parámetro |  | DESCRIPCIÓN |
| --- | --- | --- |
| ID | Obligatorio | Azure AD usa este atributo para rellenar el atributo `InResponseTo` de la respuesta devuelta. El id. no debe empezar con un número. La estrategia habitual consiste en anteponer una cadena como "id" en la representación de cadena de un GUID. Por ejemplo, `id6c1c178c166d486687be4aaf5e482730` es un id. válido. |
| Version | Obligatorio | Este parámetro debe establecerse en **2.0**. |
| IssueInstant | Obligatorio | Se trata de una cadena DateTime con un valor de hora UTC y un [formato de tiempo de ida y vuelta ("o")](https://msdn.microsoft.com/library/az4se3k1.aspx). Azure AD espera un valor DateTime de este tipo, pero no evalúa ni utiliza el valor. |
| AssertionConsumerServiceUrl | Opcional | Si se proporciona, este parámetro debe coincidir con el elemento `RedirectUri` del servicio en la nube de Azure AD. |
| ForceAuthn | Opcional | Se trata de un elemento booleano. Si es true, significa que el usuario se verá forzado a volver a autenticarse, incluso si tiene una sesión válida con Azure AD. |
| IsPassive | Opcional | Se trata de un valor booleano que especifica si Azure AD debe autenticar al usuario en modo silencioso, sin interacción del usuario, usando la cookie de sesión, si existe. Si es true, Azure AD intentará autenticar al usuario mediante la cookie de sesión. |

Todos los demás atributos `AuthnRequest`, como Consent, Destination, AssertionConsumerServiceIndex, AttributeConsumerServiceIndex y ProviderName, se **omiten**.

Azure AD también omite el elemento `Conditions` de `AuthnRequest`.

### <a name="issuer"></a>Emisor

El elemento `Issuer` de `AuthnRequest` debe coincidir exactamente con uno de los valores de **ServicePrincipalNames** del servicio en la nube de Azure AD. Normalmente, se establece en el identificador **URI de id. de aplicación** , que se especifica durante el registro de la aplicación.

Un extracto SAML que contiene el elemento `Issuer` tiene el siguiente aspecto:

```
<Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://www.contoso.com</Issuer>
```

### <a name="nameidpolicy"></a>NameIDPolicy

Este elemento solicita un formato de id. de nombre determinado en la respuesta y es opcional en los elementos `AuthnRequest` enviados a Azure AD.

Un elemento `NameIdPolicy` tiene el siguiente aspecto:

```
<NameIDPolicy Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent"/>
```

Si se proporciona `NameIDPolicy`, puede incluir su atributo `Format` opcional. El atributo `Format` solo puede tener uno de los siguientes valores; cualquier otro valor generará un error.

* `urn:oasis:names:tc:SAML:2.0:nameid-format:persistent`: Azure Active Directory emite la notificación NameID como identificador en pares.
* `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`: Azure Active Directory emite la notificación NameID en formato de dirección de correo electrónico.
* `urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified`: este valor permite a Azure Active Directory seleccionar el formato de notificación. Azure Active Directory emite la notificación NameID como identificador en pares.
* `urn:oasis:names:tc:SAML:2.0:nameid-format:transient`: Azure Active Directory emite la notificación NameID como un valor generado aleatoriamente que es único para la operación de SSO actual. Esto significa que el valor es temporal y no se puede usar para identificar al usuario en proceso de autenticación.

Azure AD omite el atributo `AllowCreate` .

### <a name="requestauthncontext"></a>RequestAuthnContext
El elemento `RequestedAuthnContext` especifica los métodos de autenticación deseados. Es opcional en los elementos `AuthnRequest` enviados a Azure AD. Azure AD admite `AuthnContextClassRef` valores como `urn:oasis:names:tc:SAML:2.0:ac:classes:Password`.

### <a name="scoping"></a>Ámbito
El elemento `Scoping`, que incluye una lista de proveedores de identidades, es opcional en los elementos `AuthnRequest` enviados a Azure AD.

Si se proporciona, no incluya el atributo `ProxyCount` ni el elemento `IDPListOption` o `RequesterID`, ya que no se admiten.

### <a name="signature"></a>Firma
No incluya un elemento `Signature` en los elementos `AuthnRequest`, ya que Azure AD no es compatible con la firma de solicitudes de autenticación.

### <a name="subject"></a>Subject
Azure AD omite el elemento `Subject` de los elementos `AuthnRequest`.

## <a name="response"></a>Respuesta
Cuando un proceso de inicio de sesión solicitado se completa correctamente, Azure AD envía una respuesta al servicio en la nube. Una respuesta a un intento de inicio de sesión correcto tiene este aspecto:

```
<samlp:Response ID="_a4958bfd-e107-4e67-b06d-0d85ade2e76a" Version="2.0" IssueInstant="2013-03-18T07:38:15.144Z" Destination="https://contoso.com/identity/inboundsso.aspx" InResponseTo="id758d0ef385634593a77bdf7e632984b6" xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
  <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion"> https://login.microsoftonline.com/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
  <ds:Signature xmlns:ds="https://www.w3.org/2000/09/xmldsig#">
    ...
  </ds:Signature>
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success" />
  </samlp:Status>
  <Assertion ID="_bf9c623d-cc20-407a-9a59-c2d0aee84d12" IssueInstant="2013-03-18T07:38:15.144Z" Version="2.0" xmlns="urn:oasis:names:tc:SAML:2.0:assertion">
    <Issuer>https://login.microsoftonline.com/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
    <ds:Signature xmlns:ds="https://www.w3.org/2000/09/xmldsig#">
      ...
    </ds:Signature>
    <Subject>
      <NameID>Uz2Pqz1X7pxe4XLWxV9KJQ+n59d573SepSAkuYKSde8=</NameID>
      <SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
        <SubjectConfirmationData InResponseTo="id758d0ef385634593a77bdf7e632984b6" NotOnOrAfter="2013-03-18T07:43:15.144Z" Recipient="https://contoso.com/identity/inboundsso.aspx" />
      </SubjectConfirmation>
    </Subject>
    <Conditions NotBefore="2013-03-18T07:38:15.128Z" NotOnOrAfter="2013-03-18T08:48:15.128Z">
      <AudienceRestriction>
        <Audience>https://www.contoso.com</Audience>
      </AudienceRestriction>
    </Conditions>
    <AttributeStatement>
      <Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name">
        <AttributeValue>testuser@contoso.com</AttributeValue>
      </Attribute>
      <Attribute Name="http://schemas.microsoft.com/identity/claims/objectidentifier">
        <AttributeValue>3F2504E0-4F89-11D3-9A0C-0305E82C3301</AttributeValue>
      </Attribute>
      ...
    </AttributeStatement>
    <AuthnStatement AuthnInstant="2013-03-18T07:33:56.000Z" SessionIndex="_bf9c623d-cc20-407a-9a59-c2d0aee84d12">
      <AuthnContext>
        <AuthnContextClassRef> urn:oasis:names:tc:SAML:2.0:ac:classes:Password</AuthnContextClassRef>
      </AuthnContext>
    </AuthnStatement>
  </Assertion>
</samlp:Response>
```

### <a name="response"></a>Respuesta

El elemento `Response` incluye el resultado de la solicitud de autorización. Azure AD configura los valores `ID`, `Version` y `IssueInstant` en el elemento `Response`. También establece los siguientes atributos:

* `Destination`: cuando el inicio de sesión se realiza correctamente, se establece en el `RedirectUri` del proveedor de servicios (servicio en la nube).
* `InResponseTo`: se establece en el atributo `ID` del elemento `AuthnRequest` que inició la respuesta.

### <a name="issuer"></a>Emisor

Azure AD establece el `Issuer` elemento `https://login.microsoftonline.com/<TenantIDGUID>/` donde \<TenantIDGUID > es el identificador del inquilino del inquilino de Azure AD.

Por ejemplo, una respuesta con el elemento Issuer podría ser similar a la siguiente:

```
<Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion"> https://login.microsoftonline.com/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
```

### <a name="status"></a>Status

El elemento `Status` transmite el éxito o el fracaso del inicio de sesión. Incluye el elemento `StatusCode`, que contiene un código o un conjunto de códigos anidados que representan el estado de la solicitud. También incluye el elemento `StatusMessage` , que contiene mensajes de error personalizados que se generan durante el proceso de inicio de sesión.

<!-- TODO: Add an authentication protocol error reference -->

A continuación, veremos una respuesta SAML a un intento fallido de inicio de sesión.

```
<samlp:Response ID="_f0961a83-d071-4be5-a18c-9ae7b22987a4" Version="2.0" IssueInstant="2013-03-18T08:49:24.405Z" InResponseTo="iddce91f96e56747b5ace6d2e2aa9d4f8c" xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
  <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://sts.windows.net/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Requester">
      <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:RequestUnsupported" />
    </samlp:StatusCode>
    <samlp:StatusMessage>AADSTS75006: An error occurred while processing a SAML2 Authentication request. AADSTS90011: The SAML authentication request property 'NameIdentifierPolicy/SPNameQualifier' is not supported.
Trace ID: 66febed4-e737-49ff-ac23-464ba090d57c
Timestamp: 2013-03-18 08:49:24Z</samlp:StatusMessage>
  </samlp:Status>
```

### <a name="assertion"></a>Aserción

Además de `ID`, `IssueInstant` y `Version`, Azure AD establece los elementos siguientes en el elemento `Assertion` de la respuesta.

#### <a name="issuer"></a>Emisor

Se establece en `https://sts.windows.net/<TenantIDGUID>/`donde \<TenantIDGUID > es el identificador del inquilino del inquilino de Azure AD.

```
<Issuer>https://login.microsoftonline.com/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
```

#### <a name="signature"></a>Firma

Azure AD firma la aserción como respuesta a un inicio de sesión correcto. El elemento `Signature` contiene una firma digital que el servicio en la nube puede utilizar para autenticar el origen con el fin de comprobar la integridad de la aserción.

Para generar esta firma digital, Azure AD utiliza la clave de firma del elemento `IDPSSODescriptor` de su documento de metadatos.

```
<ds:Signature xmlns:ds="https://www.w3.org/2000/09/xmldsig#">
      digital_signature_here
    </ds:Signature>
```

#### <a name="subject"></a>Subject

Especifica la entidad que es el firmante de las instrucciones de la aserción. Contiene un elemento `NameID` , que representa al usuario autenticado. El valor de `NameID` es un identificador de destino que se dirige únicamente al proveedor de servicios que es la audiencia del token. Es persistente; es decir se puede revocar, pero nunca volver a asignar. También es opaco, ya que no revela ningún dato del usuario y no se puede utilizar como identificador en las consultas de atributo.

El valor del atributo `Method` del elemento `SubjectConfirmation` siempre se establece en `urn:oasis:names:tc:SAML:2.0:cm:bearer`.

```
<Subject>
      <NameID>Uz2Pqz1X7pxe4XLWxV9KJQ+n59d573SepSAkuYKSde8=</NameID>
      <SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
        <SubjectConfirmationData InResponseTo="id758d0ef385634593a77bdf7e632984b6" NotOnOrAfter="2013-03-18T07:43:15.144Z" Recipient="https://contoso.com/identity/inboundsso.aspx" />
      </SubjectConfirmation>
</Subject>
```

#### <a name="conditions"></a>Condiciones

Este elemento especifica las condiciones que definen el uso aceptable de las aserciones de SAML.

```
<Conditions NotBefore="2013-03-18T07:38:15.128Z" NotOnOrAfter="2013-03-18T08:48:15.128Z">
      <AudienceRestriction>
        <Audience>https://www.contoso.com</Audience>
      </AudienceRestriction>
</Conditions>
```

Los atributos `NotBefore` y `NotOnOrAfter` especifican el intervalo durante el cual es válida la aserción.

* El valor del atributo `NotBefore` es igual a o ligeramente posterior (menos de un segundo) al del atributo `IssueInstant` del elemento `Assertion`. Azure AD no tiene en cuenta ninguna diferencia horaria entre sí y el servicio en la nube (proveedor de servicios). Además, no agrega ningún búfer a esa hora.
* El valor del atributo `NotOnOrAfter` es 70 minutos posterior al del atributo `NotBefore`.

#### <a name="audience"></a>Público

Contiene un URI que identifica una audiencia prevista. Azure AD establece el valor de este elemento en el valor del elemento `Issuer` del objeto `AuthnRequest` que inició la sesión. Para evaluar el valor de `Audience`, utilice el valor del elemento `App ID URI` que se especificó durante el registro de la aplicación.

```
<AudienceRestriction>
        <Audience>https://www.contoso.com</Audience>
</AudienceRestriction>
```

Al igual que el valor de `Issuer`, el de `Audience` debe coincidir exactamente con uno de los nombres de entidad de seguridad de servicio que representa el servicio en la nube de Azure AD. Sin embargo, si el valor del elemento `Issuer` no es un valor de URI, el de `Audience` de la respuesta será el valor de `Issuer` con el prefijo `spn:`.

#### <a name="attributestatement"></a>AttributeStatement

Contiene notificaciones sobre el firmante o el usuario. El extracto siguiente contiene un elemento `AttributeStatement` de ejemplo. Los puntos suspensivos indican que el elemento puede incluir varios atributos y valores de atributo.

```
<AttributeStatement>
      <Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name">
        <AttributeValue>testuser@contoso.com</AttributeValue>
      </Attribute>
      <Attribute Name="http://schemas.microsoft.com/identity/claims/objectidentifier">
        <AttributeValue>3F2504E0-4F89-11D3-9A0C-0305E82C3301</AttributeValue>
      </Attribute>
      ...
</AttributeStatement>
```        

* **Notificación de Name**: el valor del atributo `Name` (`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`) es el nombre principal de usuario del usuario autenticado, como `testuser@managedtenant.com`.
* **Notificación de ObjectIdentifier**: el valor del atributo `ObjectIdentifier` (`http://schemas.microsoft.com/identity/claims/objectidentifier`) es el `ObjectId` del objeto de directorio que representa al usuario autenticado en Azure AD. `ObjectId` es un identificador único global seguro, reutilizable e inmutable del usuario autenticado.

#### <a name="authnstatement"></a>AuthnStatement

Este elemento declara que se autenticó el firmante de la aserción por un medio concreto y en una hora determinada.

* El atributo `AuthnInstant` especifica la hora a la que el usuario se autenticó con Azure AD.
* El elemento `AuthnContext` especifica el contexto de autenticación utilizado para autenticar al usuario.

```
<AuthnStatement AuthnInstant="2013-03-18T07:33:56.000Z" SessionIndex="_bf9c623d-cc20-407a-9a59-c2d0aee84d12">
      <AuthnContext>
        <AuthnContextClassRef> urn:oasis:names:tc:SAML:2.0:ac:classes:Password</AuthnContextClassRef>
      </AuthnContext>
</AuthnStatement>
```
