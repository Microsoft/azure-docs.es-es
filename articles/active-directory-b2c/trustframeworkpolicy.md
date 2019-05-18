---
title: TrustFrameworkPolicy - Azure Active Directory B2C | Microsoft Docs
description: Especifique el elemento TrustFrameworkPolicy de una directiva personalizada en Azure Active Directory B2C.
services: active-directory-b2c
author: davidmu1
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/10/2018
ms.author: davidmu
ms.subservice: B2C
ms.openlocfilehash: 1a3382e560287502ae8afccae556bc5f56245904
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65812941"
---
# <a name="trustframeworkpolicy"></a>TrustFrameworkPolicy

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Una directiva personalizada se representa como uno o más archivos con formato XML que se hacen referencia entre sí en una cadena jerárquica. Los elementos XML definen los elementos de la directiva, como el esquema de notificaciones, las transformaciones de notificaciones, las definiciones de contenido, los proveedores de notificaciones, los perfiles técnicos, el recorrido del usuario y los pasos de orquestación. Cada archivo de directiva se define con el elemento **TrustFrameworkPolicy** de nivel superior de un archivo de directiva. 

```XML
<TrustFrameworkPolicy
  xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="https://www.w3.org/2001/XMLSchema"
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
  PolicySchemaVersion="0.3.0.0"
  TenantId="mytenant.onmicrosoft.com"
  PolicyId="B2C_1A_TrustFrameworkBase"
  PublicPolicyUri="http://mytenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase">
  ...
```


El elemento **TrustFrameworkPolicy** contiene los atributos siguientes:

| Atributo | Obligatorio | DESCRIPCIÓN |
|---------- | -------- | ----------- |
| PolicySchemaVersion | Sí | Versión del esquema que se usará para ejecutar la directiva. El valor debe ser `0.3.0.0` |
| TenantObjectId | No | Identificador de objeto único del inquilino de Azure Active Directory (Azure AD) B2C. |
| TenantId | Sí | Identificador único del inquilino al que pertenece esta directiva. |
| `PolicyId` | Sí | Identificador único de la directiva. Es necesario agregar el prefijo *B2C_1A_* al identificador. |
| PublicPolicyUri | Sí | URI de la directiva, que es una combinación del id. de inquilino y el id. de directiva. |
| DeploymentMode | No | Valores posibles: `Production`, `Debugging` o `Development`. `Production` es el valor predeterminado. Use esta propiedad para depurar la directiva. Para obtener más información, vea [Recopilación de registros](active-directory-b2c-troubleshoot-custom.md). |
| UserJourneyRecorderEndpoint | No | Punto de conexión que se usará cuando **DeploymentMode** se establece en `Development`. El valor tiene que ser `urn:journeyrecorder:applicationinsights`. Para obtener más información, vea [Recopilación de registros](active-directory-b2c-troubleshoot-custom.md). |


En el ejemplo siguiente, se muestra cómo especificar el elemento **TrustFrameworkPolicy**:

``` XML
<TrustFrameworkPolicy
   xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
   xmlns:xsd="https://www.w3.org/2001/XMLSchema"
   xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
   PolicySchemaVersion="0.3.0.0"
   TenantId="mytenant.onmicrosoft.com"
   PolicyId="B2C_1A_TrustFrameworkBase"
   PublicPolicyUri="http://mytenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase">
```

## <a name="inheritance-model"></a>Modelo de herencia

Estos tipos de archivos de directiva suelen usarse en un recorrido del usuario:

- Un archivo de **base** que contiene la mayoría de las definiciones. Para facilitar la solución de problemas y el mantenimiento a largo plazo de las directivas, le recomendamos que realice un número mínimo de cambios en este archivo.
- Un archivo de **extensiones** que contiene los cambios de configuración únicos del inquilino. Este archivo de directiva se basa en el archivo de base. Use este archivo para agregar nuevas funciones o reemplazar funciones existentes. Por ejemplo, puede usar este archivo para federar con nuevos proveedores de identidades.
- Un archivo de **usuario de confianza** que es el único archivo centrado en tareas que invoca directamente la aplicación de usuario de confianza, como aplicaciones de escritorio, móviles o web. Cada tarea única (como un registro o inicio de sesión, un restablecimiento de contraseña o una edición de perfil) necesita su propio archivo de directiva de usuario de confianza. Este archivo de directiva se basa en el archivo de extensiones. 

Una aplicación de usuario de confianza realiza una llamada al archivo de directiva de usuario de confianza para ejecutar una tarea específica. Por ejemplo, para iniciar el flujo de inicio de sesión. El Marco de experiencia de identidad en Azure AD B2C agrega primero todos los elementos del archivo de base; después, agrega los elementos del archivo de extensiones; y, por último, agrega los elementos del archivo de directiva de usuario de confianza para aplicar la directiva actual. Los elementos del mismo nombre y tipo en el archivo del usuario de confianza reemplazan a esos elementos en las extensiones, y los elementos de las extensiones reemplazan a los elementos del archivo base. En el diagrama siguiente, se muestra la relación entre los archivos de directiva y las aplicaciones de usuario de confianza.

![Modelo de herencia](./media/trustframeworkpolicy/custom-policy-Inheritance-model.png)

Este es el modelo de herencia:

- La directiva principal y la directiva secundaria se encuentran en el mismo esquema.
- La directiva secundaria de cualquier nivel puede heredar de la directiva principal y extenderse mediante la adición de nuevos elementos.
- No existe un límite en el número de niveles.

Para obtener información, consulte [Introducción a las directivas personalizadas](active-directory-b2c-get-started-custom.md).

## <a name="base-policy"></a>Directiva de base

Para heredar una directiva de otra directiva, es necesario declarar un elemento **BasePolicy** dentro del elemento **TrustFrameworkPolicy** del archivo de directiva. El elemento **BasePolicy** es una referencia a la directiva de base en las que se basa esta directiva.  

El elemento **BasePolicy** contiene los elementos siguientes:

| Elemento | Repeticiones | DESCRIPCIÓN |
| ------- | ----------- | --------|
| TenantId | 1:1 | El identificador del inquilino de Azure AD B2C. |
| `PolicyId` | 1:1 | El identificador de la directiva principal. |


En el ejemplo siguiente, se muestra cómo especificar una directiva de base. Esta directiva **B2C_1A_TrustFrameworkExtensions** se basa en la directiva **B2C_1A_TrustFrameworkBase**. 

``` XML
<TrustFrameworkPolicy
   xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
   xmlns:xsd="https://www.w3.org/2001/XMLSchema"
   xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
   PolicySchemaVersion="0.3.0.0"
   TenantId="mytenant.onmicrosoft.com"
   PolicyId="B2C_1A_TrustFrameworkExtensions"
   PublicPolicyUri="http://mytenant.onmicrosoft.com/B2C_1A_TrustFrameworkExtensions">

  <BasePolicy>
    <TenantId>yourtenant.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkBase</PolicyId>
  </BasePolicy>
  ...
</TrustFrameworkPolicy>
```

## <a name="policy-execution"></a>Ejecución de directiva

Una aplicación de usuario de confianza (como una aplicación de escritorio, móvil o web) realiza una llamada a la [directiva de usuario de confianza](relyingparty.md). El archivo de directiva de usuario de confianza ejecuta una tarea específica, como iniciar sesión, restablecer una contraseña o editar un perfil. La directiva del usuario de confianza configura la lista de notificaciones que recibirá la aplicación de usuario de confianza como parte del token emitido. Varias aplicaciones pueden usar la misma directiva. Todas las aplicaciones reciben el mismo token con notificaciones y el usuario pasa por el mismo recorrido de usuario. Una misma aplicación puede usar varias directivas.

Dentro del archivo de directiva de usuario de confianza, especifique el elemento **DefaultUserJourney**, que apunta al elemento [UserJourney](userjourneys.md). El recorrido del usuario suele definirse en la directiva base o de extensiones.

Directiva B2C_1A_signup_signin:

```XML
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn">
  ...
```

B2C_1A_TrustFrameWorkBase o B2C_1A_TrustFrameworkExtensionPolicy:

```XML
<UserJourneys>
  <UserJourney Id="SignOrSignIn">
  ...
```

Un recorrido del usuario define la lógica empresarial del proceso por el que pasará el usuario. Cada recorrido del usuario es un conjunto de pasos de orquestación que realiza una serie de acciones, de forma secuencial y en términos de autenticación y recopilación de información. 

El archivo de directiva **SocialAndLocalAccounts** del [paquete inicial](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-get-started-custom#download-starter-pack-and-modify-policies) contiene los recorridos del usuario SignUpOrSignIn, ProfileEdit y PasswordReset. Puede agregar más recorridos de usuario para otros escenarios, como cambiar una dirección de correo electrónico o vincular y desvincular una cuenta de redes sociales. 

Los pasos de orquestación pueden realizar una llamada a un [perfil técnico](technicalprofiles.md). Un perfil técnico proporciona un marco con un mecanismo integrado para comunicarse con distintos tipos de entidades. Por ejemplo, un perfil técnico puede realizar estas acciones, entre otras:

- Representar una experiencia del usuario.
- Permite al usuario iniciar sesión con una cuenta empresarial o de red social, como una cuenta de Facebook, Microsoft, Google, Salesforce o de cualquier otro proveedor de identidades.
- Configurar la verificación por teléfono para MFA.
- Leer y escribir datos en un almacén de identidades de Azure AD B2C.
- Llamar a un servicio de API de RESTful personalizado.

![Ejecución de directiva](./media/trustframeworkpolicy/custom-policy-execution.png)

 El elemento **TrustFrameworkPolicy** contiene los elementos siguientes:

- BasePolicy, como se ha especificado anteriormente.
- [BuildingBlocks](buildingblocks.md)
- [ClaimsProviders](claimsproviders.md)
- [UserJourneys](userjourneys.md)
- [RelyingParty](relyingparty.md)
