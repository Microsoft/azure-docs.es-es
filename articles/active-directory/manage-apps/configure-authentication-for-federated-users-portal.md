---
title: Configuración de la aceleración automática de inicio de sesión de una aplicación mediante la directiva de detección del dominio de inicio | Microsoft Docs
description: Obtenga información sobre cómo configurar la directiva de Home Realm Discovery para la autenticación de Azure Active Directory para los usuarios federados, incluidas las sugerencias de la aceleración automática y el dominio.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/08/2019
ms.author: mimart
ms.custom: seoapril2019
ms.collection: M365-identity-device-management
ms.openlocfilehash: 18f7f6588cb4fb3b3b480402c3dad13be4a0ed2c
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65781023"
---
# <a name="configure-azure-active-directory-sign-in-behavior-for-an-application-by-using-a-home-realm-discovery-policy"></a>Configuración del comportamiento de inicio de sesión de Azure Active Directory de una aplicación mediante una directiva de detección del dominio de inicio

En este artículo se proporciona una introducción a la configuración de comportamiento de la autenticación de Azure Active Directory para usuarios federados. Trata sobre la configuración de las restricciones de la aceleración automática y la autenticación para los usuarios de los dominios federados.

## <a name="home-realm-discovery"></a>Detección del dominio de inicio
La detección del dominio de inicio (HDR) es el proceso que permite a Azure Active Directory (Azure AD) determinar dónde debe autenticarse un usuario en el momento del inicio de sesión.  Al iniciar sesión en un inquilino de Azure AD para obtener acceso a un recurso o a la página de inicio de sesión común de Azure AD, el usuario debe escribir un nombre de usuario (UPN). Azure AD usa esa información para detectar dónde debe iniciar sesión el usuario. 

Es posible que el usuario sea dirigido a uno de estos lugares para poder realizar la autenticación:

- El inquilino principal del usuario (puede ser el mismo inquilino que el recurso al que el usuario está intentando obtener acceso). 

- Una cuenta Microsoft.  El usuario es un invitado en el inquilino de recursos.

-  Un proveedor de identidades local, como Servicios de federación de Active Directory (AD FS).

- Otro proveedor de identidades federado con el inquilino de Azure AD.

## <a name="auto-acceleration"></a>Aceleración automática 
Algunas organizaciones configuran dominios en su inquilino de Azure Active Directory para federarlos con otro IdP, como AD FS, y así poder realizar la autenticación de usuario.  

Cuando un usuario inicia sesión en una aplicación, en primer lugar se le muestra una página de inicio de sesión de Azure AD. Una vez que escribe su UPN, si está en un dominio federado, se le redirige a la página de inicio de sesión del IdP que sirve a ese dominio. En determinadas circunstancias, es posible que los administradores quieran dirigir a los usuarios a la página de inicio de sesión cuando inician sesión en aplicaciones específicas. 

Como resultado, los usuarios pueden omitir la página inicial de Azure Active Directory. Este proceso se conoce como "aceleración automática de inicio de sesión".

En casos donde el inquilino está federado a otro IdP para el inicio de sesión, la aceleración automática simplifica más el inicio de sesión del usuario.  Puede configurar la aceleración automática en aplicaciones individuales.

>[!NOTE]
>Si configura la aceleración automática en una aplicación, los usuarios invitados no podrán iniciar sesión. Si lleva a un usuario directamente a un IdP federado para que se autentique, este no podrá volver a la página de inicio de sesión de Azure Active Directory. Los usuarios invitados, que es posible que deban dirigirse a otros inquilinos o a un IdP externo, como una cuenta de Microsoft, no pueden iniciar sesión en esa aplicación porque omiten el paso de detección del dominio de inicio.  

Hay dos formas de controlar la aceleración automática en un IdP federado:   

- Use una sugerencia de dominio en las solicitudes de autenticación de una aplicación. 
- Configure una directiva de detección de dominio de inicio para habilitar la aceleración automática.

### <a name="domain-hints"></a>Sugerencias de dominio    
Las sugerencias de dominio son directivas que se incluyen en la solicitud de autenticación de una aplicación. Se pueden usar para enviar el usuario a su página de inicio de sesión del IdP federado. O también las puede usar una aplicación de varios inquilinos para enviar al usuario directamente a la página de inicio de sesión de Azure AD del inquilino.  

Por ejemplo, la aplicación "largeapp.com" puede permitir que sus clientes obtengan acceso a la aplicación mediante la dirección URL personalizada "contoso.largeapp.com". La aplicación también puede incluir una sugerencia de dominio contoso.com en la solicitud de autenticación. 

La sintaxis de la sugerencia de dominio varía en función del protocolo que se usa y se suele configurar en la aplicación.

**WS-Federation**: whr=contoso.com en la cadena de consulta.

**SAML**:  una solicitud de autenticación de SAML que contiene una sugerencia de dominio o una cadena de consulta whr=contoso.com.

**Open ID Connect**: una cadena de consulta domain_hint=contoso.com. 

Si una sugerencia de dominio se incluye en la solicitud de autenticación de la aplicación y el inquilino está federado con ese dominio, Azure AD intentará redirigir el inicio de sesión al IdP configurado para ese dominio. 

Si la sugerencia de dominio no hace referencia a un dominio federado verificado, se omite y se invoca la detección del dominio de inicio normal.

Para obtener más información sobre la aceleración automática con las sugerencias de dominio que son compatibles con Azure Active Directory, consulte el [blog de Enterprise Mobility + Security](https://cloudblogs.microsoft.com/enterprisemobility/2015/02/11/using-azure-ad-to-land-users-on-their-custom-login-page-from-within-your-app/).

>[!NOTE]
>Si se incluye una sugerencia de dominio en una solicitud de autenticación, su presencia invalida cualquier aceleración automática que se establezca para la aplicación en la directiva de HRD.

### <a name="home-realm-discovery-policy-for-auto-acceleration"></a>Directiva de detección del dominio de inicio para la aceleración automática
Algunas aplicaciones no proporcionan ninguna manera de configurar la solicitud de autenticación que emiten. En estos casos, no es posible utilizar sugerencias de dominio para controlar la aceleración automática. Debido a ello, la aceleración automática se puede configurar mediante la directiva y así lograr el mismo comportamiento.  

## <a name="enable-direct-authentication-for-legacy-applications"></a>Habilitación de la autenticación directa para las aplicaciones heredadas
El procedimiento recomendado es para que las aplicaciones usen el inicio de sesión interactivo y las bibliotecas de AAD para autenticar a los usuarios. Las bibliotecas se ocupan de los flujos de los usuarios federados.  A veces, las aplicaciones heredadas no se escriben para comprender la federación. No realizan la detección del dominio de inicio y no interactúan con el punto de conexión federado correcto para autenticar a un usuario. Si quiere, puede usar la directiva de HRD para permitir que determinadas aplicaciones heredadas que envían las credenciales de nombre de usuario y contraseña se autentiquen directamente con Azure Active Directory. La sincronización de hash de contraseña debe estar habilitada. 

> [!IMPORTANT]
> Habilite solo la autenticación directa si ha activado la sincronización de hash de contraseña y sabe que es correcto autenticar esta aplicación sin las directivas implementadas por el IdP local. Si, por cualquier motivo, desactiva la sincronización de hash de contraseña o la sincronización de directorios con AD Connect, debe quitar esta directiva para evitar la posibilidad de que la autenticación directa use un hash de contraseña obsoleto.

## <a name="set-hrd-policy"></a>Establecimiento de la directiva HRD
Hay que seguir tres pasos para configurar la directiva de HRD en una aplicación para la aceleración automática del inicio de sesión federado o las aplicaciones directas basadas en la nube:

1. Cree una directiva de HRD.

2. Busque la entidad de servicio a la que se debe asociar la directiva.

3. Asocie la directiva a la entidad de servicio. 

Las directivas solo surten efecto para una aplicación específica cuando se adjuntan a una entidad de servicio. 

Solo puede haber una directiva de HRD activa en una entidad de servicio en un momento dado.  

Puede usar Microsoft Azure Active Directory Graph API directamente o los cmdlets de PowerShell de Azure Active Directory para crear y administrar la directiva de HRD.

Graph API que manipula la directiva se describe en el artículo [Operations on policy](https://msdn.microsoft.com/library/azure/ad/graph/api/policy-operations) (Operaciones en la directiva) de MSDN.

Este es un ejemplo de la definición de la directiva de HRD:
    
 ```
   {  
    "HomeRealmDiscoveryPolicy":
    {  
    "AccelerateToFederatedDomain":true,
    "PreferredDomain":"federated.example.edu",
    "AllowCloudPasswordValidation":true
    }
   }
```

El tipo de directiva es "HomeRealmDiscoveryPolicy".

El valor **AccelerateToFederatedDomain** es opcional. Si **AccelerateToFederatedDomain** es false, la directiva no tiene ningún efecto en la aceleración automática. Si **AccelerateToFederatedDomain** es true y solo hay un dominio federado y verificado en el inquilino, los usuarios se dirigirán directamente al IdP federado para el inicio de sesión. Si es true y hay más de un dominio verificado en el inquilino, debe especificarse **PreferredDomain**.

El valor **PreferredDomain** es opcional. **PreferredDomain** debe indicar un dominio al que se debe acelerar. Puede omitirse si el inquilino tiene solo un dominio federado.  Si se omite y se verifica que hay más de un dominio federado, entonces la directiva no tendrá ningún efecto.

 Si se especifica **PreferredDomain**, debe coincidir con un dominio federado verificado para el inquilino. Todos los usuarios de la aplicación deben poder iniciar sesión en ese dominio.

El valor **AllowCloudPasswordValidation** es opcional. Si **AllowCloudPasswordValidation** es true, la aplicación tendrá autorización para autenticar a un usuario federado si presenta las credenciales de nombre de usuario y contraseña directamente al punto de conexión del token de Azure Active Directory. Esto solo funciona si se habilita la sincronización de hash de contraseña.

### <a name="priority-and-evaluation-of-hrd-policies"></a>Prioridad y evaluación de las directivas de HRD
Se pueden crear directivas de HRD y asignarlas a organizaciones y entidades de servicio específicas. Esto significa que se pueden aplicar varias directivas a una aplicación específica. La directiva de HRD que entra en vigor sigue estas reglas:


- Si hay una sugerencia de dominio en la solicitud de autenticación, se ignora cualquier directiva de HRD para la aceleración automática. Se usa el comportamiento que especificó la sugerencia de dominio.

- Por otro lado, si una directiva se asigna explícitamente a la entidad de servicio, esta se aplicará. 

- Si no hay sugerencias de dominio y ninguna directiva se asigna explícitamente a la entidad de servicio, se aplicará una directiva asignada explícitamente a la organización principal de la entidad de servicio. 

- Si no hay ninguna sugerencia de dominio y no se asignó ninguna directiva a la entidad de servicio o a la organización, se usa el comportamiento de HRD predeterminado.

## <a name="tutorial-for-setting-hrd-policy-on-an-application"></a>Tutorial para configurar la directiva HRD en una aplicación 
Usaremos los cmdlets de PowerShell de Azure AD para desplazarse por algunos escenarios, incluidos:


- La configuración de la directiva HRD para la aceleración automática de una aplicación en un inquilino con un dominio federado único.

- La configuración de la directiva HRD para la aceleración automática de una aplicación en uno de varios dominios que se verifican en el inquilino.

- La configuración de la directiva HRD para habilitar una aplicación heredada para que haga la autenticación de nombre de usuario y contraseña directamente a Azure Active Directory para un usuario federado.

- Enumerar las aplicaciones para las que se configura una directiva.


### <a name="prerequisites"></a>Requisitos previos
En los ejemplos siguientes, podrá crear, actualizar, vincular y eliminar directivas en las entidades de servicio de aplicación en Azure AD.

1.  Para comenzar, descargue la versión preliminar más reciente de los cmdlets de PowerShell de Azure AD. 

2.  Cuando tenga los cmdlets de PowerShell de Azure AD descargados, ejecute el comando Connect para iniciar sesión en la cuenta de administrador de Azure AD:

    ``` powershell
    Connect-AzureAD -Confirm
    ```
3.  Ejecute el siguiente comando para ver todas las directivas de la organización:

    ``` powershell
    Get-AzureADPolicy
    ```

Si no se devuelve nada, significa que no tiene directivas creadas en el inquilino.

### <a name="example-set-hrd-policy-for-an-application"></a>Ejemplo: Configuración de la directiva HRD para una aplicación 

En este ejemplo, puede crear una directiva que, cuando se asigne a una aplicación: 
- Acelere automáticamente a los usuarios hacia una pantalla de inicio de sesión de AD FS cuando estos inicien sesión en una aplicación y cuando haya un solo dominio en su inquilino. 
- Acelere automáticamente a los usuarios hacia una pantalla de inicio de sesión de AD FS cuando haya más de un dominio federado en su inquilino.
- Permita el inicio de sesión no interactivo de nombre de usuario y contraseña directamente a Azure Active Directory a los usuarios federados para las aplicaciones a las que está asignada la directiva.

#### <a name="step-1-create-an-hrd-policy"></a>Paso 1: Creación de una directiva de HRD

La directiva siguiente acelera automáticamente a los usuarios hacia una pantalla de inicio de sesión de AD FS cuando estos inician sesión en una aplicación y cuando hay un solo dominio en su inquilino.

``` powershell
New-AzureADPolicy -Definition @("{`"HomeRealmDiscoveryPolicy`":{`"AccelerateToFederatedDomain`":true}}") -DisplayName BasicAutoAccelerationPolicy -Type HomeRealmDiscoveryPolicy
```
La siguiente directiva acelera automáticamente a los usuarios hacia una pantalla de inicio de sesión de AD FS cuando hay más de un dominio federado en su inquilino. Si tiene más de un dominio federado que autentica a los usuarios para las aplicaciones, es necesario especificar el dominio para la aceleración automática.

``` powershell
New-AzureADPolicy -Definition @("{`"HomeRealmDiscoveryPolicy`":{`"AccelerateToFederatedDomain`":true, `"PreferredDomain`":`"federated.example.edu`"}}") -DisplayName MultiDomainAutoAccelerationPolicy -Type HomeRealmDiscoveryPolicy
```

Para crear una directiva a fin de habilitar la autenticación de nombre de usuario y contraseña para los usuarios federados directamente con Azure Active Directory para aplicaciones específicas, ejecute el siguiente comando:

``` powershell
New-AzureADPolicy -Definition @("{`"HomeRealmDiscoveryPolicy`":{`"AllowCloudPasswordValidation`":true}}") -DisplayName EnableDirectAuthPolicy -Type HomeRealmDiscoveryPolicy
```


Para ver la nueva directiva y obtener el valor de **ObjectID**, ejecute el siguiente comando:

``` powershell
Get-AzureADPolicy
```


Para aplicar la directiva de HRD después de crearla, puede asignarla a varias entidades de servicio de aplicación.

#### <a name="step-2-locate-the-service-principal-to-which-to-assign-the-policy"></a>Paso 2: Búsqueda de la entidad de servicio a la cual se asignará la directiva  
Para ello, necesita el valor de **ObjectID** de las entidades de servicio a las que quiere asignar la directiva. Hay varias maneras de buscar el valor de **ObjectID** de las entidades de servicio.    

Puede usar el portal, o bien puede consultar [Microsoft Graph](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#serviceprincipal-entity). También puede ir a la [herramienta Probador de Graph](https://developer.microsoft.com/graph/graph-explorer) e iniciar sesión en su cuenta de Azure AD para ver todas las entidades de servicio de su organización. Dado que usa PowerShell, puede usar el cmdlet get-AzureADServicePrincipal para enumerar las entidades de servicio y sus identificadores.

#### <a name="step-3-assign-the-policy-to-your-service-principal"></a>Paso 3: Asignación de la directiva a su entidad de servicio  
Una vez tenga el valor de **ObjectID** de la entidad de servicio de la aplicación para el que quiere configurar la aceleración automática, ejecute el siguiente comando. Este comando asocia la directiva HRD que creó en el paso 1 con la entidad de servicio que encuentra en el paso 2.

``` powershell
Add-AzureADServicePrincipalPolicy -Id <ObjectID of the Service Principal> -RefObjectId <ObjectId of the Policy>
```

Puede repetir este comando para cada entidad de servicio a la que quiera agregar la directiva.

En caso de que una aplicación ya tenga una directiva HomeRealmDiscovery asignada, no podrá agregar un segunda.  En ese caso, cambie la definición de la directiva de detección del dominio de inicio que está asignada a la aplicación para agregar parámetros adicionales.

#### <a name="step-4-check-which-application-service-principals-your-hrd-policy-is-assigned-to"></a>Paso 4: Comprobación de a qué entidades de servicio de aplicación está asignada la directiva de HRD
Para comprobar qué aplicaciones tienen configurada la directiva de aceleración automática, use el cmdlet **Get-AzureADPolicyAppliedObject**. Pásele el valor de **ObjectID** de la directiva que quiera comprobar.

``` powershell
Get-AzureADPolicyAppliedObject -ObjectId <ObjectId of the Policy>
```
#### <a name="step-5-youre-done"></a>Paso 5: ¡Y ya está!
Pruebe la aplicación para comprobar que la nueva directiva funciona correctamente.

### <a name="example-list-the-applications-for-which-hrd-policy-is-configured"></a>Ejemplo: Enumere las aplicaciones para las que se configuró una directiva de HRD.

#### <a name="step-1-list-all-policies-that-were-created-in-your-organization"></a>Paso 1: Enumeración de todas las directivas creadas en la organización 

``` powershell
Get-AzureADPolicy
```

Apunte el valor de **ObjectID** de la directiva de la cual quiere enumerar las asignaciones.

#### <a name="step-2-list-the-service-principals-to-which-the-policy-is-assigned"></a>Paso 2: Enumeración de las entidades de servicio a las que se asignó la directiva  

``` powershell
Get-AzureADPolicyAppliedObject -ObjectId <ObjectId of the Policy>
```

### <a name="example-remove-an-hrd-policy-for-an-application"></a>Ejemplo: Supresión de una directiva de HRD para una aplicación
#### <a name="step-1-get-the-objectid"></a>Paso 1: Obtención del valor de ObjectID
Use el ejemplo anterior para obtener el valor de **ObjectID** de la directiva y el de la entidad de servicio de aplicación de la que quiere eliminarlo. 

#### <a name="step-2-remove-the-policy-assignment-from-the-application-service-principal"></a>Paso 2: Supresión de la asignación de directiva de la entidad de servicio de aplicación  

``` powershell
Remove-AzureADApplicationPolicy -ObjectId <ObjectId of the Service Principal>  -PolicyId <ObjectId of the policy>
```

#### <a name="step-3-check-removal-by-listing-the-service-principals-to-which-the-policy-is-assigned"></a>Paso 3: Comprobación de que se eliminó correctamente; para ello, enumere las entidades de servicio a las que se asignó la directiva 

``` powershell
Get-AzureADPolicyAppliedObject -ObjectId <ObjectId of the Policy>
```
## <a name="next-steps"></a>Pasos siguientes
- Para obtener más información sobre el funcionamiento de la autenticación en Azure AD, consulte [Escenarios de autenticación para Azure AD](../develop/authentication-scenarios.md).
- Para obtener más información sobre el inicio de sesión único del usuario, consulte [Acceso a la aplicación e inicio de sesión único con Azure Active Directory](configure-single-sign-on-portal.md).
- Consulte la [Guía del desarrollador de Active Directory](../develop/v1-overview.md) para información general sobre todo el contenido de interés para los desarrolladores.
