---
title: Vigencia de tokens configurable en Azure Active Directory | Microsoft Docs
description: Aprenda a establecer la vigencia de los tokens emitidos por Azure AD.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: 06f5b317-053e-44c3-aaaa-cf07d8692735
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/15/2019
ms.author: ryanwi
ms.custom: aaddev, annaba
ms.reviewer: hirsin
ms.collection: M365-identity-device-management
ms.openlocfilehash: cc81f0a5c75d9aeee39f0633521d692c8d30c474
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65823465"
---
# <a name="configurable-token-lifetimes-in-azure-active-directory-preview"></a>Vigencia de tokens configurable en Azure Active Directory (versión preliminar)

Puede especificar la vigencia de un token emitido por Azure Active Directory (Azure AD). La vigencia de los tokens se puede configurar para todas las aplicaciones de una organización, para una aplicación multiinquilino (multiorganización) o para una entidad de servicio específica de una organización.

> [!IMPORTANT]
> Después de la audiencia de los clientes durante la vista previa, hemos reemplazamos la característica de vigencia de tokens configurables con [capacidades de administración de sesión de autenticación](https://go.microsoft.com/fwlink/?linkid=2083106) en acceso condicional de Azure AD. Esta característica dejará de usarse el 1 de noviembre de 2019. Si usa la directiva de vigencia de tokens configurables, cambie a la nueva característica de acceso condicional. 

En Azure AD, un objeto de directiva representa un conjunto de reglas que se exigen en algunas o todas las aplicaciones de una organización. Cada tipo de directiva tiene una estructura única con un conjunto de propiedades que luego se aplican a los objetos a los que están asignadas.

Puede designar una directiva como la directiva predeterminada para su organización. La directiva se aplicará a cualquier aplicación que resida dentro de esa organización, siempre y cuando no se haya reemplazado por una directiva con una prioridad más alta. También puede asignar una directiva a aplicaciones específicas. El orden de prioridad varía según el tipo de directiva.

> [!NOTE]
> La directiva de duración del token configurable no es compatible con SharePoint Online.  Incluso si tiene la capacidad de crear esta directiva a través de PowerShell, SharePoint Online no reconocerá esta directiva. Consulte el [blog de SharePoint Online](https://techcommunity.microsoft.com/t5/SharePoint-Blog/Introducing-Idle-Session-Timeout-in-SharePoint-and-OneDrive/ba-p/119208) para más información sobre cómo configurar los tiempos de expiración de sesiones inactivas.
>* La duración predeterminada para el token de acceso de SharePoint Online es de 1 hora. 
>* El tiempo de inactividad máximo predeterminado del token de actualización de SharePoint Online es de 90 días.

## <a name="token-types"></a>Tipos de token

Las directivas de vigencia del token se pueden establecer para tokens de actualización, tokens de acceso, tokens de sesión y tokens de identificador.

### <a name="access-tokens"></a>Tokens de acceso

Los clientes utilizan tokens de acceso para acceder a un recurso protegido. Solo se puede utilizar un token de acceso para una combinación específica de usuario, cliente y recurso. Los tokens de acceso no se pueden revocar y son válidos hasta que caducan. Un individuo maltintencionado que haya obtenido un token de acceso puede usarlo durante toda su vigencia. El ajuste de la vigencia de un token de acceso es un balance entre la mejora del rendimiento del sistema y el aumento de la cantidad de tiempo que el cliente conserva el acceso después de que la cuenta de usuario está deshabilitada. Se consigue un rendimiento mejorado del sistema al reducir el número de veces que un cliente necesita adquirir un token de acceso nuevo.  El valor predeterminado es 1 hora. Después de 1 hora, el cliente debe usar el token de actualización para adquirir un nuevo token de actualización (normalmente en modo silencioso) y un token de acceso. 

### <a name="refresh-tokens"></a>Tokens de actualización

Cuando un cliente adquiere un token de acceso para acceder a un recurso protegido, recibe también un token de actualización. El token de actualización se usa para obtener nuevos pares de tokens de acceso/actualización cuando el token de acceso actual expira. Un token de actualización se enlaza a una combinación de usuario y cliente. Se puede [revocar en cualquier momento](access-tokens.md#token-revocation) un token de actualización y se comprobará la validez de este cada vez que se use.  Los tokens de actualización no se revocan cuando se utilizan para obtener nuevos tokens de acceso; sin embargo, un procedimiento recomendado consiste en eliminar de forma segura el token antiguo cuando se obtiene uno nuevo. 

Es importante diferenciar entre clientes públicos y confidenciales, ya que esto afecta al período de tiempo que se pueden usar los tokens de actualización. Para más información sobre los diferentes tipos de clientes, consulte [RFC 6749](https://tools.ietf.org/html/rfc6749#section-2.1).

#### <a name="token-lifetimes-with-confidential-client-refresh-tokens"></a>Vigencia de los tokens de actualización de cliente confidencial
Los clientes confidenciales son aplicaciones que pueden almacenar de forma segura una contraseña (secreto) de un cliente. Pueden comprobar que las solicitudes proceden de la aplicación cliente protegida y no de un individuo malintencionado. Por ejemplo, una aplicación web es un cliente confidencial ya que puede almacenar un secreto de cliente en el servidor web. Y, por tanto, este secreto no se expone. Dado que estos flujos resultan más seguros, las duraciones predeterminadas de tokens de actualización emitidos para estos flujos será `until-revoked`, no se podrán cambiar mediante la directiva y no se revocarán en los restablecimientos de contraseña voluntarios.

#### <a name="token-lifetimes-with-public-client-refresh-tokens"></a>Vigencia de los tokens de actualización de cliente público

Los clientes públicos no pueden almacenar de forma segura una contraseña (secreto) de cliente. Por ejemplo, una aplicación de iOS o Android no puede ofuscar un secreto del propietario del recurso y, por ello, se considera a la aplicación un cliente público. Se pueden establecer directivas sobre los recursos para evitar que los tokens de actualización de clientes públicos anteriores a un período especificado obtengan un nuevo par de tokens de acceso/actualización. Para ello, utilice la propiedad de tiempo máximo de inactividad del token de actualización (`MaxInactiveTime`). También puede utilizar directivas para establecer un período más allá del cual ya no se aceptan los tokens de actualización. (Para ello, utilice la propiedad de antigüedad máxima del token de actualización). Puede ajustar la vigencia del token de actualización para controlar cuándo y con qué frecuencia es necesario que el usuario vuelva a escribir las credenciales en lugar de volver a autenticarse de forma silenciosa al usar una aplicación cliente pública.

### <a name="id-tokens"></a>Tokens de id.
Los tokens de identificador se pasan a los clientes nativos y de sitios web. Los tokens de identificador contienen información de perfil de un usuario. Un token de identificador se enlaza a una combinación específica de usuario y cliente. Los tokens de identificador se consideran válidos hasta que expiran. Normalmente, una aplicación web relaciona la vigencia de la sesión de un usuario en la aplicación con la vigencia del token de identificador emitido para el usuario. Puede ajustar la vigencia del token de identificador para controlar la frecuencia con la que la aplicación web expirará la sesión de la aplicación y requerirá que el usuario se vuelva a autenticar en Azure AD (de forma silenciosa o interactiva).

### <a name="single-sign-on-session-tokens"></a>Tokens de sesión de inicio de sesión único
Cuando un usuario se autentica con Azure AD, se establece una sesión mediante inicio de sesión único (SSO) con Azure AD y el explorador del usuario. El token de SSO, en forma de cookie, representa esta sesión. El token de sesión SSO no está enlazado a una aplicación cliente y el recurso específico. Los tokens de sesión SSO se pueden revocar y su validez se comprueba cada vez que se utilizan.

Azure AD usa dos tipos de tokens de sesión SSO: persistente y no persistente. El explorador almacena los tokens de sesión persistentes como cookies. Los tokens de sesión no persistentes se almacenan como cookies de sesión. (Las cookies de sesión se destruyen cuando se cierra el explorador). Por lo general, se almacena un token de sesión no persistente. Pero, si el usuario selecciona la casilla **Mantener la sesión iniciada** durante la autenticación, un token de sesión persistente se almacena.

Los tokens de sesión no persistentes tienen una vigencia de 24 horas. Los tokens persistentes tienen una vigencia de 180 días. Cada vez que se usa un token de sesión SSO dentro de su período de validez, el período de validez se amplía otras 24 horas o 180 días, según el tipo de token. Si un token de sesión SSO no se usa dentro de su período de validez, se considerará caducado y ya no se aceptará.

Puede utilizar una directiva para establecer el período de tiempo después de la emisión del primer token de sesión, más allá del cual ya no se aceptará este token. (Para ello, utilice la propiedad de antigüedad máxima del token de sesión). Puede ajustar la vigencia de un token de sesión para controlar cuándo y con qué frecuencia el usuario debe volver a escribir las credenciales en lugar de autenticarse de forma silenciosa cuando se usa una aplicación web.

### <a name="token-lifetime-policy-properties"></a>Propiedades de la directiva de vigencia del token
Una directiva de vigencia del token es un tipo de objeto de directiva que contiene reglas de vigencia del token. Use las propiedades de la directiva para controlar las vigencias de tokens especificados. Si no se establece ninguna directiva, el sistema aplica el valor de vigencia predeterminado.

### <a name="configurable-token-lifetime-properties"></a>Propiedades de vigencia de tokens configurables
| Propiedad | Cadena de propiedad de directiva | Afecta a | Valor predeterminado | Mínima | Máximo |
| --- | --- | --- | --- | --- | --- |
| Vigencia del token de acceso |AccessTokenLifetime |Tokens de acceso, tokens de identificador, tokens de SAML2 |1 hora |10 minutos |1 día |
| Tiempo máximo de inactividad del token de actualización |MaxInactiveTime |Tokens de actualización |90 días |10 minutos |90 días |
| Antigüedad máxima del token de actualización (un solo factor) |MaxAgeSingleFactor |Tokens de actualización (para los usuarios) |Hasta que se revoca |10 minutos |Hasta que se revoca<sup>1</sup> |
| Antigüedad máxima del token de actualización (varios factores) |MaxAgeMultiFactor |Tokens de actualización (para los usuarios) |Hasta que se revoca |10 minutos |Hasta que se revoca<sup>1</sup> |
| Antigüedad máxima del token de sesión (un solo factor) |MaxAgeSessionSingleFactor<sup>2</sup> |Tokens de sesión (persistentes y no persistentes) |Hasta que se revoca |10 minutos |Hasta que se revoca<sup>1</sup> |
| Antigüedad máxima del token de sesión (varios factores) |MaxAgeSessionMultiFactor<sup>3</sup> |Tokens de sesión (persistentes y no persistentes) |Hasta que se revoca |10 minutos |Hasta que se revoca<sup>1</sup> |

* <sup>1</sup>365 días es la vigencia explícita máxima que se puede establecer para estos atributos.

### <a name="exceptions"></a>Excepciones
| Propiedad | Afecta a | Valor predeterminado |
| --- | --- | --- |
| Antigüedad máxima de los tokens de actualización (emitidos para usuarios federados con información de revocación insuficiente<sup>1</sup>) |Tokens de actualización (emitidos para usuarios federados con información de revocación insuficiente<sup>1</sup>) |12 horas |
| Tiempo máximo de inactividad del token de actualización (emitido para clientes confidenciales) |Tokens de actualización (emitidos para clientes confidenciales) |90 días |
| Antigüedad máxima del token de actualización (emitido para clientes confidenciales) |Tokens de actualización (emitidos para clientes confidenciales) |Hasta que se revoca |

* <sup>1</sup>Entre los usuarios federados que tienen información de revocación insuficiente se incluyen todos los usuarios que no tienen el atributo "LastPasswordChangeTimestamp" sincronizado. A estos usuarios se les da esta antigüedad máxima tan corta porque AAD no puede comprobar cuándo se deben revocar los tokens asociados a una credencial antigua (como una contraseña que se ha modificado) y deben realizar comprobaciones más frecuentes para asegurarse de que el usuario y los tokens asociados están aún activos. Para mejorar esta experiencia, los administradores de los inquilinos deben asegurarse de que están sincronizando el atributo "LastPasswordChangeTimestamp" (esto se puede establecer en el objeto de usuario con PowerShell o a través de AADSync).

### <a name="policy-evaluation-and-prioritization"></a>Evaluación y prioridades de directivas
Puede crear y, a continuación, asignar una directiva de vigencia del token a una aplicación específica, a su organización y a las entidades de servicio. Se pueden aplicar varias directivas a una aplicación específica. La directiva de vigencia del token que entra en vigor sigue estas reglas:

* Si una directiva se asigna explícitamente a la entidad de servicio, se aplicará.
* Si no hay ninguna directiva que se asigne explícitamente a la entidad de servicio, se aplicará una directiva asignada explícitamente a la organización primaria de la entidad de servicio.
* Si no hay ninguna directiva que se asigne explícitamente a la entidad de servicio o a la organización, se aplicará la directiva asignada a la aplicación.
* Si no se ha asignado ninguna directiva a la entidad de servicio, la organización o el objeto de aplicación, se aplicarán los valores predeterminados. (Consulte la tabla que aparece en [Propiedades de vigencia de tokens configurables](#configurable-token-lifetime-properties)).

Para más información sobre la relación entre objetos de aplicación y objetos de entidad de servicio, consulte [Objetos de aplicación y de entidad de servicio de Azure Active Directory](app-objects-and-service-principals.md).

La validez de un token se evalúa en el momento en el que se usa. La directiva con la prioridad más alta en la aplicación a la que se accede es la que se aplica.

Todos los intervalos de tiempo usados aquí tienen formato según el objeto [TimeSpan](/dotnet/api/system.timespan) de C#: D.HH:MM:SS.  Por lo tanto, 80 días y 30 minutos sería `80.00:30:00`.  La D inicial puede eliminarse si es cero, por lo que 90 minutos sería `00:90:00`.  

> [!NOTE]
> Este es un escenario de ejemplo.
>
> Un usuario desea acceder a dos aplicaciones web: Aplicación web A y B.
> 
> Factores:
> * Ambas aplicaciones web están en la misma organización primaria.
> * La directiva 1 de vigencia del token con una antigüedad máxima del token de sesión de ocho horas se establece como valor predeterminado de la organización primaria.
> * La aplicación web A es una aplicación web de uso habitual que no está vinculada a ninguna directiva.
> * La aplicación web B se usa para procesos altamente confidenciales. Su entidad de servicio está vinculada a la directiva 2 de vigencia del token, que tiene una antigüedad máxima de token de sesión de 30 minutos.
>
> A las 12:00 p.m. el usuario inicia una nueva sesión en el explorador e intenta acceder a la aplicación web A. El usuario es redirigido a Azure AD y se le pide que inicie sesión. Esta acción crea una cookie con un token de sesión en el explorador. El usuario es redirigido de nuevo a la aplicación web A con un token de identificador que le permite acceder a la aplicación.
>
> A las 12:15 p.m., el usuario intenta acceder a la aplicación web B. El explorador redirige a Azure AD, que detecta la cookie de sesión. La entidad de servicio de la aplicación web B está vinculada a la directiva 2 de vigencia del token, pero también forma parte de la organización primaria con la directiva 1 de vigencia del token predeterminada. La directiva 2 de vigencia del token se aplica porque las directivas vinculadas a entidades de servicio tienen una prioridad más alta que las directivas predeterminadas de la organización. El token de sesión se emitió originalmente en los últimos 30 minutos, por lo que se considera válido. El usuario es redirigido de nuevo a la aplicación web B con un token de identificador que le concede acceso.
>
> A la 1 p.m., el usuario intenta acceder a la aplicación web A y se le redirige a Azure AD. La aplicación web A no está vinculada a ninguna directiva, pero como está en una organización con la directiva predeterminada 1 de vigencia del token, se aplica esta directiva. Se detectó la cookie de sesión que se emitió originalmente en las últimas ocho horas. En modo silencioso, se redirige al usuario a la aplicación web A con un nuevo token de identificador. No es necesario que el usuario se autentique.
>
> Inmediatamente después, el usuario intenta acceder a la aplicación web B y se le redirige a Azure AD. Como antes, se aplica la directiva 2 de vigencia del token. Dado que el token se emitió hace más de 30 minutos, se le solicita al usuario que vuelva a escribir sus credenciales de inicio de sesión. Se emite un nuevo token de sesión y de identificador. El usuario puede acceder entonces a la aplicación web B.
>
>

## <a name="configurable-policy-property-details"></a>Detalles de las propiedades de directiva configurables
### <a name="access-token-lifetime"></a>Vigencia del token de acceso
**Cadena:** AccessTokenLifetime

**Afecta a:** Tokens de acceso, tokens de identificador

**Resumen:** esta directiva controla cuánto tiempo se consideran válidos los token de acceso y de identificador para este recurso. Reducir la vigencia de los tokens de acceso disminuye el riesgo de que un individuo malintencionado use un token de acceso o de identificador durante un período de tiempo prolongado. (Estos tokens no se pueden revocar). El inconveniente es que afecta negativamente al rendimiento, ya que los tokens tendrán que reemplazarse con más frecuencia.

### <a name="refresh-token-max-inactive-time"></a>Tiempo máximo de inactividad del token de actualización
**Cadena:** MaxInactiveTime

**Afecta a:** Tokens de actualización

**Resumen:** esta directiva controla la antigüedad máxima que puede tener un token de actualización hasta que un cliente ya no pueda usarlo para recuperar un nuevo par de tokens de acceso/actualización al intentar acceder a este recurso. Dado que un nuevo token de actualización normalmente se devuelve cuando se usa un token de actualización, esta directiva impedirá el acceso si el cliente intenta acceder a algún recurso con el token de actualización actual durante el período de tiempo especificado.

Esta directiva obligará a los usuarios que no hayan estado activos en su cliente a volver a autenticarse para recuperar un nuevo token de actualización.

El tiempo máximo de inactividad del token de actualización debe establecerse en un valor inferior a la antigüedad máxima del token de un solo factor y la antigüedad máxima del token de actualización de varios factores.

### <a name="single-factor-refresh-token-max-age"></a>Antigüedad máxima del token de actualización (un solo factor)
**Cadena:** MaxAgeSingleFactor

**Afecta a:** Tokens de actualización

**Resumen:** esta directiva controla cuánto tiempo un usuario puede seguir usando tokens de actualización para obtener nuevos pares de tokens de acceso/actualización desde la última vez que se autenticara correctamente con un solo factor. Después de que un usuario se autentica y recibe un nuevo token de actualización, este puede utilizar el flujo del token de actualización durante el período de tiempo especificado. (Esto ocurre siempre que el token de actualización actual no esté revocado y no se quede sin usar más tiempo que el período de inactividad). En ese momento, el usuario se verá obligado a volver a autenticarse para recibir un nuevo token de actualización.

Reducir la antigüedad máxima obliga a los usuarios a autenticarse con más frecuencia. Puesto que la autenticación de un solo factor se considera menos segura que la autenticación multifactor, se recomienda establecer esta propiedad en un valor igual o inferior al de la propiedad de antigüedad máxima del token de actualización de varios factores.

### <a name="multi-factor-refresh-token-max-age"></a>Antigüedad máxima del token de actualización (varios factores)
**Cadena:** MaxAgeMultiFactor

**Afecta a:** Tokens de actualización

**Resumen:** esta directiva controla cuánto tiempo un usuario puede seguir usando tokens de actualización para obtener nuevos pares de tokens de acceso/actualización desde la última vez que se autenticara correctamente con varios factores. Después de que un usuario se autentica y recibe un nuevo token de actualización, este puede utilizar el flujo del token de actualización durante el período de tiempo especificado. (Esto ocurre siempre que el token de actualización actual no esté revocado y no se quede sin usar más tiempo que el período de inactividad). En ese momento, los usuarios se verán obligados a volver a autenticarse para recibir un nuevo token de actualización.

Reducir la antigüedad máxima obliga a los usuarios a autenticarse con más frecuencia. Puesto que la autenticación de un solo factor se considera menos segura que la autenticación multifactor, se recomienda establecer esta propiedad en un valor igual o superior al de la propiedad de antigüedad máxima del token de actualización de un solo factor.

### <a name="single-factor-session-token-max-age"></a>Antigüedad máxima del token de sesión (un solo factor)
**Cadena:** MaxAgeSessionSingleFactor

**Afecta a:** Tokens de sesión (persistentes y no persistentes)

**Resumen:** esta directiva controla cuánto tiempo un usuario puede seguir usando tokens de actualización para obtener un nuevo token de identificador y de sesión desde la última vez que se autenticara correctamente con un solo factor. Después de que un usuario se autentica y recibe un nuevo token de sesión, este puede utilizar el flujo del token de sesión durante el período de tiempo especificado. (Esto ocurre siempre que el token de sesión actual no esté revocado y no haya expirado). Tras el período de tiempo especificado, el usuario se ve obligado a autenticarse para recibir un nuevo token de sesión.

Reducir la antigüedad máxima obliga a los usuarios a autenticarse con más frecuencia. Puesto que la autenticación de un solo factor se considera menos segura que la autenticación multifactor, se recomienda establecer esta propiedad en un valor igual o inferior al de la propiedad de antigüedad máxima del token de sesión de varios factores.

### <a name="multi-factor-session-token-max-age"></a>Antigüedad máxima del token de sesión (varios factores)
**Cadena:** MaxAgeSessionMultiFactor

**Afecta a:** Tokens de sesión (persistentes y no persistentes)

**Resumen:** esta directiva controla cuánto tiempo un usuario puede seguir usando tokens de actualización para obtener un nuevo token de identificador y de sesión desde la última vez que se autenticara correctamente con varios factores. Después de que un usuario se autentica y recibe un nuevo token de sesión, este puede utilizar el flujo del token de sesión durante el período de tiempo especificado. (Esto ocurre siempre que el token de sesión actual no esté revocado y no haya expirado). Tras el período de tiempo especificado, el usuario se ve obligado a autenticarse para recibir un nuevo token de sesión.

Reducir la antigüedad máxima obliga a los usuarios a autenticarse con más frecuencia. Puesto que la autenticación de un solo factor se considera menos segura que la autenticación multifactor, se recomienda establecer esta propiedad en un valor igual o superior al de la propiedad de antigüedad máxima del token de sesión de un solo factor.

## <a name="example-token-lifetime-policies"></a>Ejemplo de directivas de vigencia del token
En Azure AD, hay muchos escenarios posibles a la hora de crear y administrar la vigencia de los tokens para aplicaciones, entidades de servicio y la organización en general. En esta sección, se explican algunos escenarios de directiva comunes que le ayudarán a imponer nuevas reglas para:

* Vigencia del token
* Tiempos máximos de inactividad del token
* Antigüedad máxima del token

En los ejemplos, puede aprender a:

* Administrar una directiva predeterminada de una organización
* Crear una directiva para inicio de sesión web
* Crear una directiva para una aplicación nativa que llama a una API web
* Administrar una directiva avanzada

### <a name="prerequisites"></a>Requisitos previos
En los ejemplos siguientes, va a crear, actualizar, vincular y eliminar directivas de aplicaciones, entidades de servicio y de la organización en general. Si no está familiarizado con Azure AD, se recomienda que aprenda [cómo obtener un inquilino de Azure AD](quickstart-create-new-tenant.md) antes de continuar con estos ejemplos.  

Para comenzar, realice uno de los pasos siguientes:

1. Descargue la [versión preliminar pública más reciente del módulo de PowerShell de Azure AD](https://www.powershellgallery.com/packages/AzureADPreview).
2. Ejecute el comando `Connect` para iniciar sesión en la cuenta de administrador de Azure AD. Ejecute este comando cada vez que inicie una nueva sesión.

    ```powershell
    Connect-AzureAD -Confirm
    ```

3. Ejecute el siguiente comando para ver todas las directivas que se han creado en la organización. Este comando debe ejecutarse después de la mayoría de las operaciones en los escenarios siguientes. La ejecución del comando también lo ayudará a obtener el valor de ** ** de sus directivas.

    ```powershell
    Get-AzureADPolicy
    ```

### <a name="example-manage-an-organizations-default-policy"></a>Ejemplo: Administrar una directiva predeterminada de una organización
En este ejemplo, creará una directiva que permite el inicio de sesión de usuarios con menos frecuencia en toda su organización. Para ello, creamos una directiva de vigencia del token para tokens de actualización de un solo factor que se aplica en toda la organización. Esta directiva se aplicará a todas las aplicaciones de su organización y a todas las entidades de servicio que aún no tengan una directiva establecida en ella.

1. Cree una directiva de vigencia del token.

    1. Establezca el token de actualización de un solo factor en "hasta que se revoca". El token no expira hasta que se revoca el acceso. Cree la siguiente definición de directiva:

        ```powershell
        @('{
            "TokenLifetimePolicy":
            {
                "Version":1,
                "MaxAgeSingleFactor":"until-revoked"
            }
        }')
        ```

    2. Ejecute el siguiente comando para crear la directiva:

        ```powershell
        $policy = New-AzureADPolicy -Definition @('{"TokenLifetimePolicy":{"Version":1, "MaxAgeSingleFactor":"until-revoked"}}') -DisplayName "OrganizationDefaultPolicyScenario" -IsOrganizationDefault $true -Type "TokenLifetimePolicy"
        ```

    3. Para ver su nueva directiva y obtener el **ObjectID** de esta, ejecute el siguiente comando:

        ```powershell
        Get-AzureADPolicy -Id $policy.Id
        ```

2. Actualice la directiva.

    Puede decidir que la primera directiva que se establece en este ejemplo no sea tan estricta como exige el servicio. Para establecer que el token de actualización de un solo factor expire en dos días, ejecute el siguiente comando:

    ```powershell
    Set-AzureADPolicy -Id $policy.Id -DisplayName $policy.DisplayName -Definition @('{"TokenLifetimePolicy":{"Version":1,"MaxAgeSingleFactor":"2.00:00:00"}}')
    ```

### <a name="example-create-a-policy-for-web-sign-in"></a>Ejemplo: Crear una directiva para inicio de sesión web

En este ejemplo, va a crear una directiva que requerirá que los usuarios se autentiquen con más frecuencia en la aplicación web. Esta directiva establecerá la vigencia de los tokens de acceso y de identificador y la antigüedad máxima de un token de sesión de varios factores en la entidad de servicio de su aplicación web.

1. Cree una directiva de vigencia del token.

    Para el inicio de sesión web esta directiva establecerá la vigencia del token de acceso y de identificador y la antigüedad máxima del token de sesión de un solo factor en 2 horas.

    1. Ejecute este comando para crear la directiva:

        ```powershell
        $policy = New-AzureADPolicy -Definition @('{"TokenLifetimePolicy":{"Version":1,"AccessTokenLifetime":"02:00:00","MaxAgeSessionSingleFactor":"02:00:00"}}') -DisplayName "WebPolicyScenario" -IsOrganizationDefault $false -Type "TokenLifetimePolicy"
        ```

    2. Para ver su nueva directiva y obtener el **ObjectID** de esta, ejecute el siguiente comando:

        ```powershell
        Get-AzureADPolicy -Id $policy.Id
        ```

2. Asigne la directiva a su entidad de servicio. También necesitará obtener el valor de **ObjectId** de su entidad de servicio.

    1. Use la [Get-AzureADServicePrincipal](/powershell/module/azuread/get-azureadserviceprincipal) cmdlet para ver las entidades de servicio de la de la organización o una entidad de servicio único.
        ```powershell
        # Get ID of the service principal
        $sp = Get-AzureADServicePrincipal -Filter "DisplayName eq '<service principal display name>'"
        ```

    2. Cuando tenga la entidad de servicio, ejecute el siguiente comando:
        ```powershell
        # Assign policy to a service principal
        Add-AzureADServicePrincipalPolicy -Id $sp.ObjectId -RefObjectId $policy.Id
        ```

### <a name="example-create-a-policy-for-a-native-app-that-calls-a-web-api"></a>Ejemplo: Crear una directiva para una aplicación nativa que llama a una API web
En este ejemplo, va a crear una directiva que requerirá que los usuarios se autentiquen con menos frecuencia. La directiva también aumenta la cantidad de tiempo que un usuario puede estar inactivo antes de que este deba volver a autenticarse. La directiva se aplica a la API web. Cuando la aplicación nativa solicita la API web como recurso, se aplica esta directiva.

1. Cree una directiva de vigencia del token.

    1. Para crear una directiva estricta para una API web, ejecute el siguiente comando:

        ```powershell
        $policy = New-AzureADPolicy -Definition @('{"TokenLifetimePolicy":{"Version":1,"MaxInactiveTime":"30.00:00:00","MaxAgeMultiFactor":"until-revoked","MaxAgeSingleFactor":"180.00:00:00"}}') -DisplayName "WebApiDefaultPolicyScenario" -IsOrganizationDefault $false -Type "TokenLifetimePolicy"
        ```

    2. Para ver la nueva directiva, ejecute el siguiente comando:

        ```powershell
        Get-AzureADPolicy -Id $policy.Id
        ```

2. Asigne la directiva a la API web. También necesitará obtener el valor de **ObjectId** de la aplicación. Utilice la [Get-AzureADApplication](/powershell/module/azuread/get-azureadapplication) para encontrar la aplicación **ObjectId**, o usar el [portal de Azure](https://portal.azure.com/).

    Obtener el **ObjectId** de la aplicación y asignar la directiva:

    ```powershell
    # Get the application
    $app = Get-AzureADApplication -Filter "DisplayName eq 'Fourth Coffee Web API'"

    # Assign the policy to your web API.
    Add-AzureADApplicationPolicy -Id $app.ObjectId -RefObjectId $policy.Id
    ```

### <a name="example-manage-an-advanced-policy"></a>Ejemplo: Administrar una directiva avanzada
En este ejemplo, crear unas pocas directivas para obtener información sobre cómo funciona el sistema de prioridad. También aprenderá a administrar varias directivas que se aplican a varios objetos.

1. Cree una directiva de vigencia del token.

    1. Para crear una directiva predeterminada de organización que establece la vigencia del token de actualización de un solo factor en 30 días, ejecute el siguiente comando:

        ```powershell
        $policy = New-AzureADPolicy -Definition @('{"TokenLifetimePolicy":{"Version":1,"MaxAgeSingleFactor":"30.00:00:00"}}') -DisplayName "ComplexPolicyScenario" -IsOrganizationDefault $true -Type "TokenLifetimePolicy"
        ```

    2. Para ver la nueva directiva, ejecute el siguiente comando:

        ```powershell
        Get-AzureADPolicy -Id $policy.Id
        ```

2. Asigne la directiva a una entidad de servicio.

    Ahora tiene una directiva que se aplica a toda la organización. Es posible que quiera conservar esta directiva 30 días para una entidad de servicio específica, pero cambiar la directiva predeterminada de organización para que sea el límite superior de "hasta que se revoque".

    1. Para ver las entidades de servicio de la de la organización, usa el [Get-AzureADServicePrincipal](/powershell/module/azuread/get-azureadserviceprincipal) cmdlet.

    2. Cuando tenga la entidad de servicio, ejecute el siguiente comando:

        ```powershell
        # Get ID of the service principal
        $sp = Get-AzureADServicePrincipal -Filter "DisplayName eq '<service principal display name>'"

        # Assign policy to a service principal
        Add-AzureADServicePrincipalPolicy -Id $sp.ObjectId -RefObjectId $policy.Id
        ```

3. Establezca la marca `IsOrganizationDefault` en false:

    ```powershell
    Set-AzureADPolicy -Id $policy.Id -DisplayName "ComplexPolicyScenario" -IsOrganizationDefault $false
    ```

4. Cree una nueva directiva predeterminada de organización:

    ```powershell
    New-AzureADPolicy -Definition @('{"TokenLifetimePolicy":{"Version":1,"MaxAgeSingleFactor":"until-revoked"}}') -DisplayName "ComplexPolicyScenarioTwo" -IsOrganizationDefault $true -Type "TokenLifetimePolicy"
    ```

    Ahora tiene la directiva original vinculada a la entidad de servicio y la nueva directiva establecida como su directiva predeterminada de organización. Es importante recordar que las directivas aplicadas a las entidades de servicio tienen prioridad sobre las directivas predeterminadas de organización.

## <a name="cmdlet-reference"></a>Referencia de cmdlets

### <a name="manage-policies"></a>Administración de directivas

Los cmdlets siguientes se pueden usar para administrar directivas.

#### <a name="new-azureadpolicy"></a>New-AzureADPolicy

Crea una nueva directiva.

```powershell
New-AzureADPolicy -Definition <Array of Rules> -DisplayName <Name of Policy> -IsOrganizationDefault <boolean> -Type <Policy Type>
```

| Parámetros | DESCRIPCIÓN | Ejemplo |
| --- | --- | --- |
| <code>&#8209;Definition</code> |La matriz de cadenas JSON que contiene todas las reglas de la directiva. | `-Definition @('{"TokenLifetimePolicy":{"Version":1,"MaxInactiveTime":"20:00:00"}}')` |
| <code>&#8209;DisplayName</code> |Cadena del nombre de la directiva. |`-DisplayName "MyTokenPolicy"` |
| <code>&#8209;IsOrganizationDefault</code> |Si es true establece la directiva como directiva predeterminada de la organización. Si es false, no hace nada. |`-IsOrganizationDefault $true` |
| <code>&#8209;Type</code> |Tipo de directiva. Para la vigencia de los tokens, use siempre "TokenLifetimePolicy". | `-Type "TokenLifetimePolicy"` |
| <code>&#8209;AlternativeIdentifier</code> [Opcional] |Establece un id alternativo para la directiva. |`-AlternativeIdentifier "myAltId"` |

</br></br>

#### <a name="get-azureadpolicy"></a>Get-AzureADPolicy
Obtiene todas las directivas de AzureAD o una directiva especificada.

```powershell
Get-AzureADPolicy
```

| Parámetros | DESCRIPCIÓN | Ejemplo |
| --- | --- | --- |
| <code>&#8209;Id</code> [Opcional] |**ObjectId (ID)** de la directiva que desee. |`-Id <ObjectId of Policy>` |

</br></br>

#### <a name="get-azureadpolicyappliedobject"></a>Get-AzureADPolicyAppliedObject
Obtiene todas las aplicaciones y entidades de servicio vinculadas a una directiva.

```powershell
Get-AzureADPolicyAppliedObject -Id <ObjectId of Policy>
```

| Parámetros | DESCRIPCIÓN | Ejemplo |
| --- | --- | --- |
| <code>&#8209;Id</code> |**ObjectId (ID)** de la directiva que desee. |`-Id <ObjectId of Policy>` |

</br></br>

#### <a name="set-azureadpolicy"></a>Set-AzureADPolicy
Actualiza una directiva existente.

```powershell
Set-AzureADPolicy -Id <ObjectId of Policy> -DisplayName <string>
```

| Parámetros | DESCRIPCIÓN | Ejemplo |
| --- | --- | --- |
| <code>&#8209;Id</code> |**ObjectId (ID)** de la directiva que desee. |`-Id <ObjectId of Policy>` |
| <code>&#8209;DisplayName</code> |Cadena del nombre de la directiva. |`-DisplayName "MyTokenPolicy"` |
| <code>&#8209;Definition</code> [Opcional] |La matriz de cadenas JSON que contiene todas las reglas de la directiva. |`-Definition @('{"TokenLifetimePolicy":{"Version":1,"MaxInactiveTime":"20:00:00"}}')` |
| <code>&#8209;IsOrganizationDefault</code> [Opcional] |Si es true establece la directiva como directiva predeterminada de la organización. Si es false, no hace nada. |`-IsOrganizationDefault $true` |
| <code>&#8209;Type</code> [Opcional] |Tipo de directiva. Para la vigencia de los tokens, use siempre "TokenLifetimePolicy". |`-Type "TokenLifetimePolicy"` |
| <code>&#8209;AlternativeIdentifier</code> [Opcional] |Establece un id alternativo para la directiva. |`-AlternativeIdentifier "myAltId"` |

</br></br>

#### <a name="remove-azureadpolicy"></a>Remove-AzureADPolicy
Elimina la directiva especificada.

```powershell
 Remove-AzureADPolicy -Id <ObjectId of Policy>
```

| Parámetros | DESCRIPCIÓN | Ejemplo |
| --- | --- | --- |
| <code>&#8209;Id</code> |**ObjectId (ID)** de la directiva que desee. | `-Id <ObjectId of Policy>` |

</br></br>

### <a name="application-policies"></a>Directivas de aplicación
Los cmdlets siguientes se pueden usar para directivas de aplicación.</br></br>

#### <a name="add-azureadapplicationpolicy"></a>Add-AzureADApplicationPolicy
Vincula la directiva especificada a una aplicación.

```powershell
Add-AzureADApplicationPolicy -Id <ObjectId of Application> -RefObjectId <ObjectId of Policy>
```

| Parámetros | DESCRIPCIÓN | Ejemplo |
| --- | --- | --- |
| <code>&#8209;Id</code> |**ObjectId (ID)** de la aplicación. | `-Id <ObjectId of Application>` |
| <code>&#8209;RefObjectId</code> |El valor de **ObjectId** de la directiva. | `-RefObjectId <ObjectId of Policy>` |

</br></br>

#### <a name="get-azureadapplicationpolicy"></a>Get-AzureADApplicationPolicy
Obtiene la directiva asignada a una aplicación.

```powershell
Get-AzureADApplicationPolicy -Id <ObjectId of Application>
```

| Parámetros | DESCRIPCIÓN | Ejemplo |
| --- | --- | --- |
| <code>&#8209;Id</code> |**ObjectId (ID)** de la aplicación. | `-Id <ObjectId of Application>` |

</br></br>

#### <a name="remove-azureadapplicationpolicy"></a>Remove-AzureADApplicationPolicy
Quita una directiva de una aplicación.

```powershell
Remove-AzureADApplicationPolicy -Id <ObjectId of Application> -PolicyId <ObjectId of Policy>
```

| Parámetros | DESCRIPCIÓN | Ejemplo |
| --- | --- | --- |
| <code>&#8209;Id</code> |**ObjectId (ID)** de la aplicación. | `-Id <ObjectId of Application>` |
| <code>&#8209;PolicyId</code> |El valor de **ObjectId** de la directiva. | `-PolicyId <ObjectId of Policy>` |

</br></br>

### <a name="service-principal-policies"></a>Directivas de entidad de servicio
Los cmdlets siguientes se pueden usar para las directivas de entidad de servicio.

#### <a name="add-azureadserviceprincipalpolicy"></a>Add-AzureADServicePrincipalPolicy
Vincula la directiva especificada a una entidad de servicio.

```powershell
Add-AzureADServicePrincipalPolicy -Id <ObjectId of ServicePrincipal> -RefObjectId <ObjectId of Policy>
```

| Parámetros | DESCRIPCIÓN | Ejemplo |
| --- | --- | --- |
| <code>&#8209;Id</code> |**ObjectId (ID)** de la aplicación. | `-Id <ObjectId of Application>` |
| <code>&#8209;RefObjectId</code> |El valor de **ObjectId** de la directiva. | `-RefObjectId <ObjectId of Policy>` |

</br></br>

#### <a name="get-azureadserviceprincipalpolicy"></a>Get-AzureADServicePrincipalPolicy
Obtiene cualquier directiva vinculada a la entidad de servicio especificada.

```powershell
Get-AzureADServicePrincipalPolicy -Id <ObjectId of ServicePrincipal>
```

| Parámetros | DESCRIPCIÓN | Ejemplo |
| --- | --- | --- |
| <code>&#8209;Id</code> |**ObjectId (ID)** de la aplicación. | `-Id <ObjectId of Application>` |

</br></br>

#### <a name="remove-azureadserviceprincipalpolicy"></a>Remove-AzureADServicePrincipalPolicy
Quita la directiva de la entidad de servicio especificada.

```powershell
Remove-AzureADServicePrincipalPolicy -Id <ObjectId of ServicePrincipal>  -PolicyId <ObjectId of Policy>
```

| Parámetros | DESCRIPCIÓN | Ejemplo |
| --- | --- | --- |
| <code>&#8209;Id</code> |**ObjectId (ID)** de la aplicación. | `-Id <ObjectId of Application>` |
| <code>&#8209;PolicyId</code> |El valor de **ObjectId** de la directiva. | `-PolicyId <ObjectId of Policy>` |
