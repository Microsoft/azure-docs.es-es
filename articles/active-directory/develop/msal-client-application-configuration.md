---
title: Configuración de la aplicación cliente (biblioteca de autenticación de Microsoft) | Azure
description: Obtenga información sobre las opciones de configuración de cliente público y cliente confidencial aplicaciones en la biblioteca de autenticación de Microsoft (MSAL).
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/12/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4d731a8153dc6a70382c0d87cc20d8c961d9fe24
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65546006"
---
# <a name="application-configuration-options"></a>Opciones de configuración de aplicación

En el código, inicializa un nuevo cliente público o confidencial (o el agente de usuario para MSAL.js) la aplicación para autenticar y obtener tokens.  Hay una serie de opciones de configuración diferentes que se pueden establecer al inicializar la aplicación cliente en MSAL. Estas opciones se pueden dividir en dos grupos:

- Opciones de registro, incluidos:
    - [Autoridad](#authority) (formado por el proveedor de identidades [instancia](#cloud-instance) e inicio de sesión [audiencia](#application-audience) para la aplicación y, posiblemente, el identificador del inquilino).
    - [Id. de cliente](#client-id)
    - [URI de redireccionamiento](#redirect-uri)
    - [secreto de cliente](#client-secret) (para aplicaciones de cliente confidencial).
- [Opciones de registro](#logging), incluido el nivel de registro, control de los datos personales y el nombre del componente mediante la biblioteca.

## <a name="authority"></a>Autoridad
La entidad es una dirección URL que indica un directorio que MSAL puede solicitar tokens de. Autoridades habituales son:

- https://login.microsoftonline.com/&lt; inquilino&gt;/, donde &lt;inquilino&gt; es el identificador del inquilino del inquilino de Azure AD o un dominio asociado a este inquilino de Azure AD.  Se usa solo para iniciar sesión en los usuarios de una organización específica.
- https://login.microsoftonline.com/common/. Se usa para iniciar sesión en los usuarios con el trabajo y cuentas educativas o una cuenta personal de Microsoft.
- https://login.microsoftonline.com/organizations/. Se usa para iniciar sesión en los usuarios con cuentas profesionales y educativas.
- https://login.microsoftonline.com/consumers/  Se usa para iniciar sesión en los usuarios con solo cuentas de Microsoft (live).

La configuración de la entidad debe ser coherente con lo que se declara en el portal de registro de aplicación.

La dirección URL de la entidad se compone de la instancia y el público.

Puede ser la entidad:
- una entidad de la nube de Active Directory de Azure
- una entidad de Azure AD B2C. Consulte [características B2C](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/AAD-B2C-specifics).
- una autoridad ADFS. Consulte [compatible con ADFS](https://aka.ms/msal-net-adfs-support).

Las entidades en la nube de Azure AD tienen dos partes:
- el proveedor de identidades **instancia**
- el inicio de sesión **audiencia** para la aplicación

La instancia y la audiencia pueden concatenan y se proporcionan como la dirección URL de la entidad. En las versiones de MSAL.NET antes de MSAL 3.x, tenía que compose de la entidad de forma manual según la nube que desea la audiencia de inicio de sesión y de destino.  El diagrama siguiente muestra cómo se compone la dirección URL de la entidad.

![Autoridad](media/msal-client-application-configuration/authority.png)

## <a name="cloud-instance"></a>Instancia en la nube
El **instancia** se utiliza para especificar si la aplicación que inicia sesión a los usuarios de la nube pública de Microsoft Azure o de nubes nacionales. Con MSAL en el código, la instancia de la nube de Azure se puede establecer mediante el uso de una enumeración o pasando la dirección URL para el [instancia de la nube nacional](authentication-national-cloud.md#azure-ad-authentication-endpoints) como el `Instance` miembro (si lo sabe).

MSAL.NET se iniciará una excepción explícita si ambos `Instance` y `AzureCloudInstance` se especifican. 

Si no se especifica una instancia, la aplicación dirigirá a la instancia de nube pública de Azure (la instancia de la dirección URL `https://login.onmicrosoftonline.com`).

## <a name="application-audience"></a>Audiencia de la aplicación

La audiencia de inicio de sesión depende de las necesidades empresariales de la aplicación:
- Si es una línea de desarrollador de negocio (LOB), probablemente deberá generar una aplicación de inquilino único, que se usará solo en su organización. En ese caso, deberá especificar lo que es esta organización, ya sea por su identificador de inquilino (Id. de Azure Active Directory) o un nombre de dominio asociado con Azure Active Directory.
- Si es un ISV, es posible que desee iniciar sesión en los usuarios con su trabajo y cuentas educativas de cualquier organización, o algunas organizaciones (aplicación de varios inquilinos), pero también puede tener los usuarios iniciar sesión con su cuenta personal de Microsoft.

### <a name="how-to-specify-the-audience-in-your-codeconfiguration"></a>Cómo especificar la audiencia en la configuración de código
Con MSAL en el código, especificar la audiencia mediante:
- ya sea Azure AD autoridad audiencia enumeración. 
- o el identificador del inquilino, que puede ser:
  - un GUID (identificador de Azure Active Directory), aplicaciones de inquilino único
  - un nombre de dominio asociado con Azure Active Directory (también para aplicaciones de inquilino único)
- o bien uno de estos marcadores de posición como un identificador de inquilino en lugar de la enumeración de audiencia de autoridad de Azure AD:
    - `organizations` para una aplicación de varios inquilinos
    - `consumers` Para iniciar sesión en los usuarios sólo con sus cuentas personales
    - `common` Para iniciar sesión en los usuarios con su trabajo y la cuenta educativa o la cuenta personal de Microsoft

MSAL producirá una excepción significativa si especifica la audiencia de autoridad de Azure AD y el identificador del inquilino. 

Si no se especifica una audiencia dirigirá a la aplicación Azure AD y las cuentas Microsoft personales como una audiencia (es decir `common`).

### <a name="effective-audience"></a>Audiencia efectivo
La audiencia para su aplicación eficaz será el mínimo (si hay una intersección) de la audiencia que se establece en la aplicación y los destinatarios especificados en el registro de la aplicación. De hecho, la experiencia de registro de aplicación ([registro de aplicación](https://aka.ms/appregistrations)) le permite especificar la audiencia (tipos de cuenta compatibles) para la aplicación. Consulte [Quickstart: Registrar una aplicación con la plataforma Microsoft identity](quickstart-register-app.md) para obtener más información.

Actualmente, la única manera de obtener una aplicación para iniciar sesión en los usuarios con sólo cuentas personales de Microsoft es establecer:
- Establezca la audiencia del registro de aplicación "Trabajo y cuentas educativas y cuentas personales" y,
- y establezca la audiencia en el código o configuración `AadAuthorityAudience.PersonalMicrosoftAccount` (o `TenantID `= "consumers")

## <a name="client-id"></a>Identificador de cliente
El identificador único de aplicación (cliente) asignado a la aplicación cuando se registró la aplicación de Azure AD.

## <a name="redirect-uri"></a>URI de redirección
El URI de redireccionamiento es el URI donde el proveedor de identidades enviará los tokens de seguridad nuevo. 

### <a name="redirect-uri-for-public-client-applications"></a>URI de redirección de las aplicaciones de cliente público
Para los desarrolladores de aplicaciones de cliente público con MSAL:
- No es necesario pasar el ``RedirectUri`` tal como se calcula automáticamente, MSAL. Esta redirección URI se establece en los valores siguientes, dependiendo de la plataforma:

- ``urn:ietf:wg:oauth:2.0:oob`` para todas las plataformas de Windows
- ``msal{ClientId}://auth`` para iOS y Xamarin Android

Sin embargo, la redirección URI debe configurarse en el [registro de la aplicación](https://aka.ms/appregistrations).

![URI de redirección en el portal](media/msal-client-application-configuration/redirect-uri.png)

Es posible reemplazar el identificador URI de redireccionamiento mediante el `RedirectUri` propiedad, por ejemplo si utiliza agentes. Estos son algunos ejemplos de URI de redireccionamiento en ese caso:

- `RedirectUriOnAndroid` = "msauth-5a434691-ccb2-4fd1-b97b-b64bcfbc03fc://com.microsoft.identity.client.sample";
- `RedirectUriOnIos` = $"msauth.{Bundle.ID}://auth";

Para obtener detalles vea detalles de Android y [aspectos específicos de iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Leveraging-the-broker-on-iOS)

### <a name="redirect-uri-for-confidential-client-applications"></a>URI de redirección de las aplicaciones cliente confidencial
Para las aplicaciones web, el URI de redireccionamiento (o URI de respuesta), es el URI en el que Azure AD se comunicará con atrás la aplicación con el token. Esto puede ser la dirección URL de la aplicación Web / API de Web si la información confidencial es uno de ellos.  Este URI de redirección debe estar registrada en el registro de aplicación. Esto es especialmente importante al implementar una aplicación que inicialmente ha probado localmente. A continuación, deberá agregar la dirección URL de respuesta de la aplicación implementada en el portal de registro de aplicación.

Para las aplicaciones de demonio, no es necesario especificar un URI de redirección.

## <a name="client-secret"></a>Secreto de cliente
El secreto de cliente para la aplicación cliente confidencial. Este secreto (contraseña de aplicación) es proporcionada por el portal de registro de aplicación, o proporciona a Azure AD durante el registro de aplicaciones con PowerShell AzureAD, AzureRM de PowerShell o CLI de Azure.

## <a name="logging"></a>Registro
Las otras opciones de habilitar el registro y solución de problemas. Para obtener más información, consulte el [registro](msal-logging.md) para obtener más detalles sobre cómo usarlas.

## <a name="next-steps"></a>Pasos siguientes
Obtenga información sobre [crear instancias de las aplicaciones de cliente mediante MSAL.NET](msal-net-initializing-client-applications.md).

Obtenga información sobre [crear instancias de las aplicaciones cliente mediante MSAL.js](msal-js-initializing-client-applications.md).
