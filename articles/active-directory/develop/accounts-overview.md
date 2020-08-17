---
title: Cuentas de la Plataforma de identidad de Microsoft y perfiles de inquilino en Android | Azure
description: Información general de las cuentas de la Plataforma de identidad de Microsoft para Android
services: active-directory
author: shoatman
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.devlang: java
ms.date: 09/14/2019
ms.author: shoatman
ms.custom: aaddev, devx-track-java
ms.reviewer: shoatman
ms.openlocfilehash: 404ffbc09a69b623a421bd0c01550d72e5c03158
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88115992"
---
# <a name="accounts--tenant-profiles-android"></a>Cuentas y perfiles de inquilinos (Android)

En este artículo se proporciona información general sobre qué es una `account` en la Plataforma de identidad de Microsoft.

La API de la biblioteca de autenticación de Microsoft (MSAL) reemplaza el término *user* por el término *account*. Una razón es que un usuario (agente humano o de software) puede tener, o utilizar, varias cuentas. Estas cuentas pueden encontrarse en la propia organización del usuario o en otras organizaciones de las que el usuario es miembro.

Una cuenta de la Plataforma de identidad de Microsoft consta de lo siguiente:

- Identificador único.  
- Una o varias credenciales usadas para demostrar la propiedad o el control de la cuenta.
- Uno o varios perfiles que constan de atributos como:
  - Imagen, nombre, apellido, puesto y ubicación de la oficina
- Una cuenta tiene un origen de autoridad o un sistema de registro. Este es el sistema en el que se crea la cuenta y donde se almacenan las credenciales asociadas con esa cuenta. En sistemas multiinquilino, como la Plataforma de identidad de Microsoft, el sistema de registro es el `tenant` en el que se creó la cuenta. Este inquilino también se conoce como `home tenant`.
- Las cuentas de la Plataforma de identidad de Microsoft tienen los siguientes sistemas de registro:
  - Azure Active Directory, incluido Azure Active Directory B2C.
  - Cuenta de Microsoft (Live).
- Las cuentas de los sistemas de registro fuera de la plataforma de identidad de Microsoft se representan dentro de la Plataforma de identidad de Microsoft, incluidas:
  - las identidades de directorios locales conectados (Windows Server Active Directory)
  - las identidades externas de LinkedIn, GitHub, etc.
  En estos casos, una cuenta tiene a la vez un sistema de origen de registro y un sistema de registro dentro de la Plataforma de identidad de Microsoft.
- La Plataforma de identidad de Microsoft permite usar una cuenta para acceder a los recursos que pertenecen a varias organizaciones (inquilinos de Azure Active Directory).
  - Para registrar si una cuenta de un sistema de registro (inquilino A de AAD) tiene acceso a un recurso de otro sistema de registro (inquilino B de AAD), la cuenta debe estar representada en el inquilino en el que se define el recurso. Esto se hace mediante la creación de un registro local de la cuenta del sistema A en el sistema B.
  - Este registro local, que es la representación de la cuenta, está vinculado a la cuenta original.
  - MSAL muestra este registro local como `Tenant Profile`.
  - El perfil de inquilino puede tener atributos diferentes que sean adecuados para el contexto local, como puesto de trabajo, ubicación de la oficina, información de contacto, etc.
- Como una cuenta puede estar presente en uno o varios inquilinos, la cuenta puede tener más de un perfil.

> [!NOTE]
> MSAL trata el sistema de cuentas de Microsoft (Live, MSA) como otro inquilino dentro de la Plataforma de identidad de Microsoft. El identificador de inquilino del inquilino de la cuenta de Microsoft es: `9188040d-6c67-4c5b-b112-36a304b66dad`

## <a name="account-overview-diagram"></a>Diagrama general de cuentas

![Diagrama general de cuentas](./media/accounts-overview/accounts-overview.svg)

En el diagrama anterior:

- La cuenta `bob@contoso.com` se crea en la instancia local de Windows Server Active Directory (sistema local de origen del registro).
- La cuenta `tom@live.com` se crea en el inquilino de la cuenta de Microsoft.
- `bob@contoso.com` tiene acceso al menos a un recurso en los inquilinos de Azure Active Directory siguientes:
  - contoso.com (sistema en la nube de registro, vinculado al sistema local de registro)
  - fabrikam.com
  - woodgrovebank.com
  - Existe un perfil de inquilino para `bob@contoso.com` en cada uno de estos inquilinos.
- `tom@live.com` tiene acceso a los recursos de los siguientes inquilinos de Microsoft:
  - contoso.com
  - fabrikam.com
  - Existe un perfil de inquilino para `tom@live.com` en cada uno de estos inquilinos.
- La información sobre Tom y Bob en otros inquilinos puede diferir de la del sistema de registro. Pueden diferir en atributos como puesto de trabajo, ubicación de la oficina, etc. Pueden ser miembros de grupos o roles dentro de cada organización (inquilino de Azure Active Directory). Nos referimos a esta información como perfil de inquilino bob@contoso.com.

En el diagrama, bob@contoso.com y tom@live.com tienen acceso a los recursos en distintos inquilinos de Azure Active Directory. Para más información, consulte [Incorporación de usuarios de colaboración B2B de Azure Active Directory en Azure Portal](../external-identities/add-users-administrator.md).

## <a name="accounts-and-single-sign-on-sso"></a>Cuentas e inicio de sesión único (SSO)

La caché de token de MSAL almacena un *token de actualización único* por cuenta. Este token de actualización se puede usar para solicitar tokens de acceso de forma silenciosa desde varios inquilinos de la Plataforma de identidad de Microsoft. Cuando se instala un agente en un dispositivo, el agente administra la cuenta y se hace posible el inicio de sesión único en todo el dispositivo.

> [!IMPORTANT]
> El comportamiento de la cuenta de negocio a cliente (B2C) y del token de actualización difieren del resto de la Plataforma de identidad de Microsoft. Para más información, consulte [Directivas de B2C y cuentas](#b2c-policies--accounts).

## <a name="account-identifiers"></a>Identificadores de cuentas

El identificador de la cuenta de MSAL no es un identificador de objeto de cuenta. No está diseñado para ser analizado ni se pretende que se confíe en él para transmitir nada que no sea la exclusividad en la Plataforma de identidad de Microsoft.

Para compatibilidad con la biblioteca de autenticación de Azure AD (ADAL) y para facilitar la migración de ADAL a MSAL, MSAL puede buscar cuentas con cualquier identificador válido para la cuenta disponible en la memoria caché de MSAL.  Por ejemplo, lo siguiente siempre recuperará el mismo objeto de cuenta para tom@live.com porque cada uno de los identificadores es válido:

```java
// The following would always retrieve the same account object for tom@live.com because each identifier is valid

IAccount account = app.getAccount("<tome@live.com msal account id>");
IAccount account = app.getAccount("<tom@live.com contoso user object id>");
IAccount account = app.getAccount("<tom@live.com woodgrovebank user object id>");
```

## <a name="accessing-claims-about-an-account"></a>Acceso a las notificaciones sobre una cuenta

Además de solicitar un token de acceso, MSAL siempre solicita un token de identificador a cada inquilino. Para ello, solicita siempre los siguientes ámbitos:

- openid
- perfile

El token de identificador contiene una lista de notificaciones. `Claims` son pares de nombre/valor sobre la cuenta y se usan para realizar la solicitud.

Tal como se ha mencionado anteriormente, cada inquilino en el que existe una cuenta puede almacenar información diferente acerca de la cuenta, incluidos, entre otros, atributos como puesto de trabajo, ubicación de la oficina, etc.

Aunque una cuenta puede ser miembro o invitado en varias organizaciones, MSAL no consulta a un servicio para obtener una lista de los inquilinos de los que es miembro la cuenta. En su lugar, MSAL crea una lista de los inquilinos en los que está presente la cuenta, como resultado de las solicitudes de token realizadas.

Las notificaciones expuestas en el objeto de cuenta siempre son las notificaciones de 'home tenant'/{authority} para una cuenta. Si esa cuenta no se ha usado para solicitar un token para el inquilino de inicio, MSAL no puede proporcionar notificaciones a través del objeto de cuenta.  Por ejemplo:

```java
// Psuedo Code
IAccount account = getAccount("accountid");

String username = account.getClaims().get("preferred_username");
String tenantId = account.getClaims().get("tid"); // tenant id
String objectId = account.getClaims().get("oid"); // object id
String issuer = account.getClaims().get("iss"); // The tenant specific authority that issued the id_token
```

> [!TIP]
> Para ver una lista de las notificaciones disponibles en el objeto de cuenta, consulte [Notificaciones de un id_token](./id-tokens.md#claims-in-an-id_token)

> [!TIP]
> Para incluir notificaciones adicionales en el id_token, consulte la documentación sobre notificaciones opcionales en [Procedimientos para: Proporcionar notificaciones opcionales a la aplicación de Azure AD](./active-directory-optional-claims.md)

### <a name="access-tenant-profile-claims"></a>Acceso a las notificaciones de perfil de inquilino

Para acceder a las notificaciones sobre una cuenta cuando aparecen en otros inquilinos, primero debe convertir el objeto de cuenta en `IMultiTenantAccount`. Todas las cuentas pueden tener varios inquilinos, pero el número de perfiles de inquilino disponibles a través de MSAL se basa en los inquilinos de los que ha solicitado tokens utilizando la cuenta actual.  Por ejemplo:

```java
// Psuedo Code
IAccount account = getAccount("accountid");
IMultiTenantAccount multiTenantAccount = (IMultiTenantAccount)account;

multiTenantAccount.getTenantProfiles().get("tenantid for fabrikam").getClaims().get("family_name");
multiTenantAccount.getTenantProfiles().get("tenantid for contoso").getClaims().get("family_name");
```

## <a name="b2c-policies--accounts"></a>Directivas de B2C y cuentas

Los tokens de actualización de una cuenta no se comparten entre las directivas de B2C. Por lo tanto, no es posible el inicio de sesión único con tokens. Esto no significa que no sea posible el inicio de sesión único. Significa que el inicio de sesión único tiene que utilizar una experiencia interactiva en la que esté disponible una cookie para habilitar el inicio de sesión único.

Esto también supone que, en el caso de MSAL, si adquiere tokens con directivas de B2C diferentes, estas se tratarán como cuentas independientes, cada una con su propio identificador. Si desea usar una cuenta para solicitar un token con `acquireTokenSilent`, deberá seleccionar la cuenta en la lista de cuentas que coincide con la directiva que está usando con la solicitud de token. Por ejemplo:

```java
// Get Account For Policy

String policyId = "SignIn";
IAccount signInPolicyAccount = getAccountForPolicyId(app, policyId);

private IAccount getAccountForPolicy(IPublicClientApplication app, String policyId)
{
    List<IAccount> accounts = app.getAccounts();

    foreach(IAccount account : accounts)
   {
        if (account.getClaims().get("tfp").equals(policyId))
        {
            return account;
        }
    }

    return null;
}
```