---
title: Protocolo SAML de cierre de inicio de sesión único de Azure
description: Este artículo describe el protocolo SAML de cierre de sesión único en Azure Active Directory
services: active-directory
author: kenwith
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 07/19/2017
ms.author: kenwith
ms.custom: aaddev
ms.reviewer: paulgarn
ms.openlocfilehash: 1d09355993af96e9e0cd334c57174cdaa771b388
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88118270"
---
# <a name="single-sign-out-saml-protocol"></a>Protocolo SAML de cierre de sesión único

Azure Active Directory (Azure AD) es compatible con el perfil de cierre de sesión único del explorador web de SAML 2.0. Para un correcto funcionamiento del inicio de sesión único, debe registrarse explícitamente **LogoutURL** con Azure AD en la aplicación durante el proceso de registro. Azure AD usa LogoutURL para redirigir a los usuarios después de que cierren sesión.

En el diagrama siguiente se muestra el flujo de trabajo del proceso de cierre de sesión único de Azure AD.

![Flujo de trabajo de cierre de sesión único de Azure AD](./media/single-sign-out-saml-protocol/active-directory-saml-single-sign-out-workflow.png)

## <a name="logoutrequest"></a>LogoutRequest
El servicio en la nube envía un mensaje `LogoutRequest` a Azure AD para indicar que ha finalizado una sesión. El extracto siguiente muestra un ejemplo de elemento `LogoutRequest` .

```
<samlp:LogoutRequest xmlns="urn:oasis:names:tc:SAML:2.0:metadata" ID="idaa6ebe6839094fe4abc4ebd5281ec780" Version="2.0" IssueInstant="2013-03-28T07:10:49.6004822Z" xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
  <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://www.workaad.com</Issuer>
  <NameID xmlns="urn:oasis:names:tc:SAML:2.0:assertion"> Uz2Pqz1X7pxe4XLWxV9KJQ+n59d573SepSAkuYKSde8=</NameID>
</samlp:LogoutRequest>
```

### <a name="logoutrequest"></a>LogoutRequest
El elemento `LogoutRequest` enviado a Azure AD requiere los atributos siguientes:

* `ID`: identifica la solicitud de cierre de sesión. El valor de `ID` no debe empezar con un número. La práctica habitual es anexar **id** a la representación de cadena de un GUID.
* `Version`: establece el valor de este elemento en **2.0**. Este valor es necesario.
* `IssueInstant`: es una cadena `DateTime` con un valor en hora universal coordinada (UTC) y un [formato de ida y vuelta ("o")](/dotnet/standard/base-types/standard-date-and-time-format-strings). Azure AD espera un valor de este tipo, pero no lo fuerza.

### <a name="issuer"></a>Emisor
El elemento `Issuer` de `LogoutRequest` debe coincidir exactamente con uno de los valores de **ServicePrincipalNames** del servicio en la nube de Azure AD. Normalmente, se establece en el identificador **URI de id. de aplicación** , que se especifica durante el registro de la aplicación.

### <a name="nameid"></a>NameID
El valor del elemento `NameID` debe coincidir exactamente con el valor de `NameID` del usuario que cierra sesión.

## <a name="logoutresponse"></a>LogoutResponse
Azure AD envía un elemento `LogoutResponse` en respuesta a un elemento `LogoutRequest`. El extracto siguiente muestra un ejemplo de `LogoutResponse`.

```
<samlp:LogoutResponse ID="_f0961a83-d071-4be5-a18c-9ae7b22987a4" Version="2.0" IssueInstant="2013-03-18T08:49:24.405Z" InResponseTo="iddce91f96e56747b5ace6d2e2aa9d4f8c" xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
  <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://sts.windows.net/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success" />
  </samlp:Status>
</samlp:LogoutResponse>
```

### <a name="logoutresponse"></a>LogoutResponse
Azure AD configura los valores `ID`, `Version` y `IssueInstant` en el elemento `LogoutResponse`. También establece el elemento `InResponseTo` en el valor del atributo `ID` del elemento `LogoutRequest` que obtiene la respuesta.

### <a name="issuer"></a>Emisor
Azure AD establece este valor en `https://login.microsoftonline.com/<TenantIdGUID>/`, donde \<TenantIdGUID> es el identificador de inquilino del inquilino de Azure AD.

Para evaluar el valor del elemento `Issuer` , utilice el valor del identificador **URI de id. de aplicación** proporcionado durante el registro de la aplicación.

### <a name="status"></a>Estado
Azure AD usa el elemento `StatusCode` en `Status` para indicar que el cierre de sesión ha sido satisfactorio o que se ha producido algún error. Cuando se produce un error en el intento de cierre de sesión, el elemento `StatusCode` también puede contener mensajes de error personalizados.