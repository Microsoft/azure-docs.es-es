---
title: Información sobre el esquema y las expresiones personalizadas de Azure AD
description: En este artículo se describe el esquema de Azure AD, los atributos del agente de aprovisionamiento y las expresiones personalizadas.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/18/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1fff90777c63820b54a1cb37156021c894de19c8
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89226937"
---
# <a name="understand-the-azure-ad-schema"></a>Información sobre el esquema de Azure AD
Un objeto de Azure Active Directory (Azure AD), al igual que cualquier directorio, es una construcción de datos general mediante programación que representa elementos como usuarios, grupos y contactos. Al crear un nuevo usuario o contacto en Azure AD, está creando una nueva instancia de ese objeto. Estas instancias se pueden diferenciar en función de sus propiedades.

En Azure AD, las propiedades son los elementos responsables de almacenar información sobre una instancia de un objeto en Azure AD.

El esquema de Azure AD define las reglas sobre qué propiedades se pueden usar en una entrada, los tipos de valores que esas propiedades pueden tener y el modo en que los usuarios pueden interactuar con esos valores. 

Azure AD tiene dos tipos de propiedades:
- **Propiedades integradas**: propiedades predefinidas por el esquema de Azure AD. Proporcionan diferentes usos y pueden o no ser accesibles.
- **Extensiones de directorio**: propiedades que se proporcionan para que pueda personalizar Azure AD para su propio uso. Por ejemplo, si ha ampliado la instancia local de Active Directory con un determinado atributo y desea pasar ese atributo, puede usar una de las propiedades personalizadas que se proporcionan. 

## <a name="attributes-and-expressions"></a>Atributos y expresiones
Cuando un objeto, como un usuario, se aprovisiona en Azure AD, se crea una nueva instancia del objeto de usuario. Esta creación incluye las propiedades de ese objeto, que también se conocen como atributos. Inicialmente, el objeto recién creado tiene los atributos establecidos en los valores que las reglas de sincronización determinan. Estos atributos se mantienen entonces actualizados a través del agente de aprovisionamiento en la nube.

![Aprovisionamiento de objetos](media/concept-attributes/attribute1.png)

Por ejemplo, un usuario puede formar parte de un departamento de Marketing. Cuando se realiza el aprovisionamiento, se crea inicialmente el atributo de departamento de Azure AD y su valor se establece en Marketing. Seis meses más tarde, si el usuario cambia a Ventas, su atributo de departamento de Active Directory local cambia a Ventas. Este cambio se sincroniza con Azure AD y se refleja en el objeto de usuario de Azure AD.

La sincronización de atributos puede ser directa, en la que el valor de Azure AD se establece directamente en el valor del atributo local, o bien se puede administrar mediante una expresión de programación. Una expresión de programación es necesaria en los casos en los que es necesario aplicar alguna lógica o una determinación para rellenar el valor.

Por ejemplo, si tuviera el atributo de correo electrónico "john.smith@contoso.com" y necesitara quitar la parte "@contoso.com" para pasar solo el valor "john.smith", usaría algo parecido a esto:

`Replace([mail], "@contoso.com", , ,"", ,)`  

**Entrada/salida de ejemplo:** <br>

* **ENTRADA** (mail): "john.smith@contoso.com"
* **SALIDA**: "john.smith"

Para más información sobre la escritura de expresiones personalizadas y la sintaxis, consulte [Escritura de expresiones para la asignación de atributos en Azure Active Directory](../app-provisioning/functions-for-customizing-application-data.md).

En la tabla siguiente se enumeran algunos atributos comunes y se describe cómo se sincronizan con Azure AD.


|Active Directory local|Tipo de asignación|Azure AD|
|-----|-----|-----|
|cn|Directo|commonName
|countryCode|Directo|countryCode|
|DisplayName|Directo|DisplayName|
|givenName|Expression|givenName|
|objectGUID|Directo|sourceAnchorBinary|  
|userprincipalName|Directo|userPrincipalName|
|ProxyAdress|Directo|ProxyAddress|

## <a name="view-the-schema"></a>Consulta del esquema
> [!WARNING]
> La configuración de aprovisionamiento en la nube crea una entidad de servicio. La entidad de servicio está visible en Azure Portal. No debe modificar las asignaciones de atributo mediante la experiencia de la entidad de servicio en Azure Portal.  Esto no se admite.

Para ver el esquema y comprobarlo, siga estos pasos.

1.  Vaya al [Probador de Graph](https://developer.microsoft.com/graph/graph-explorer).
1.  Inicie sesión con la cuenta de administrador global.
1.  A la izquierda, seleccione **modificar permisos** y asegúrese de que **Directory.ReadWrite.All** tiene *consentimiento*.
1.  Ejecute la consulta `https://graph.microsoft.com/beta/serviceprincipals/?$filter=startswith(Displayname,'Active')`. Esta consulta devuelve una lista filtrada de entidades de servicio.
1.  Busque `"appDisplayName": "Active Directory to Azure Active Directory Provisioning"` y anote el valor de `"id"`.
    ```
    "value": [
            {
                "id": "00d41b14-7958-45ad-9d75-d52fa29e02a1",
                "deletedDateTime": null,
                "accountEnabled": true,
                "appDisplayName": "Active Directory to Azure Active Directory Provisioning",
                "appId": "1a4721b3-e57f-4451-ae87-ef078703ec94",
                "applicationTemplateId": null,
                "appOwnerOrganizationId": "47df5bb7-e6bc-4256-afb0-dd8c8e3c1ce8",
                "appRoleAssignmentRequired": false,
                "displayName": "Active Directory to Azure Active Directory Provisioning",
                "errorUrl": null,
                "homepage": "https://account.activedirectory.windowsazure.com:444/applications/default.aspx?metadata=AD2AADProvisioning|ISV9.1|primary|z",
                "loginUrl": null,
                "logoutUrl": null,
                "notificationEmailAddresses": [],
                "preferredSingleSignOnMode": null,
                "preferredTokenSigningKeyEndDateTime": null,
                "preferredTokenSigningKeyThumbprint": null,
                "publisherName": "Active Directory Application Registry",
                "replyUrls": [],
                "samlMetadataUrl": null,
                "samlSingleSignOnSettings": null,
                "servicePrincipalNames": [
                    "http://adapplicationregistry.onmicrosoft.com/adprovisioningtoaad/primary",
                    "1a4721b3-e57f-4451-ae87-ef078703ec94"
                ],
                "signInAudience": "AzureADMultipleOrgs",
                "tags": [
                    "WindowsAzureActiveDirectoryIntegratedApp"
                ],
                "addIns": [],
                "api": {
                    "resourceSpecificApplicationPermissions": []
                },
                "appRoles": [
                    {
                        "allowedMemberTypes": [
                            "User"
                        ],
                        "description": "msiam_access",
                        "displayName": "msiam_access",
                        "id": "a0326856-1f51-4311-8ae7-a034d168eedf",
                        "isEnabled": true,
                        "origin": "Application",
                        "value": null
                    }
                ],
                "info": {
                    "termsOfServiceUrl": null,
                    "supportUrl": null,
                    "privacyStatementUrl": null,
                    "marketingUrl": null,
                    "logoUrl": null
                },
                "keyCredentials": [],
                "publishedPermissionScopes": [
                    {
                        "adminConsentDescription": "Allow the application to access Active Directory to Azure Active Directory Provisioning on behalf of the signed-in user.",
                        "adminConsentDisplayName": "Access Active Directory to Azure Active Directory Provisioning",
                        "id": "d40ed463-646c-4efe-bb3e-3fa7d0006688",
                        "isEnabled": true,
                        "type": "User",
                        "userConsentDescription": "Allow the application to access Active Directory to Azure Active Directory Provisioning on your behalf.",
                        "userConsentDisplayName": "Access Active Directory to Azure Active Directory Provisioning",
                        "value": "user_impersonation"
                    }
                ],
                "passwordCredentials": []
            },
    ```
1. Reemplace `{Service Principal id}` por su valor y ejecute la consulta `https://graph.microsoft.com/beta/serviceprincipals/{Service Principal id}/synchronization/jobs/`.
1. Busque `"id": "AD2AADProvisioning.fd1c9b9e8077402c8bc03a7186c8f976"` y anote el valor de `"id"`.
    ```
    {
                "id": "AD2AADProvisioning.fd1c9b9e8077402c8bc03a7186c8f976",
                "templateId": "AD2AADProvisioning",
                "schedule": {
                    "expiration": null,
                    "interval": "PT2M",
                    "state": "Active"
                },
                "status": {
                    "countSuccessiveCompleteFailures": 0,
                    "escrowsPruned": false,
                    "code": "Active",
                    "lastSuccessfulExecutionWithExports": null,
                    "quarantine": null,
                    "steadyStateFirstAchievedTime": "2019-11-08T15:48:05.7360238Z",
                    "steadyStateLastAchievedTime": "2019-11-20T16:17:24.7957721Z",
                    "troubleshootingUrl": "",
                    "lastExecution": {
                        "activityIdentifier": "2dea06a7-2960-420d-931e-f6c807ebda24",
                        "countEntitled": 0,
                        "countEntitledForProvisioning": 0,
                        "countEscrowed": 15,
                        "countEscrowedRaw": 15,
                        "countExported": 0,
                        "countExports": 0,
                        "countImported": 0,
                        "countImportedDeltas": 0,
                        "countImportedReferenceDeltas": 0,
                        "state": "Succeeded",
                        "error": null,
                        "timeBegan": "2019-11-20T16:15:21.116098Z",
                        "timeEnded": "2019-11-20T16:17:24.7488681Z"
                    },
                    "lastSuccessfulExecution": {
                        "activityIdentifier": null,
                        "countEntitled": 0,
                        "countEntitledForProvisioning": 0,
                        "countEscrowed": 0,
                        "countEscrowedRaw": 0,
                        "countExported": 5,
                        "countExports": 0,
                        "countImported": 0,
                        "countImportedDeltas": 0,
                        "countImportedReferenceDeltas": 0,
                        "state": "Succeeded",
                        "error": null,
                        "timeBegan": "0001-01-01T00:00:00Z",
                        "timeEnded": "2019-11-20T14:09:46.8855027Z"
                    },
                    "progress": [],
                    "synchronizedEntryCountByType": [
                        {
                            "key": "group to Group",
                            "value": 33
                        },
                        {
                            "key": "user to User",
                            "value": 3
                        }
                    ]
                },
                "synchronizationJobSettings": [
                    {
                        "name": "Domain",
                        "value": "{\"DomainFQDN\":\"contoso.com\",\"DomainNetBios\":\"CONTOSO\",\"ForestFQDN\":\"contoso.com\",\"ForestNetBios\":\"CONTOSO\"}"
                    },
                    {
                        "name": "DomainFQDN",
                        "value": "contoso.com"
                    },
                    {
                        "name": "DomainNetBios",
                        "value": "CONTOSO"
                    },
                    {
                        "name": "ForestFQDN",
                        "value": "contoso.com"
                    },
                    {
                        "name": "ForestNetBios",
                        "value": "CONTOSO"
                    },
                    {
                        "name": "QuarantineTooManyDeletesThreshold",
                        "value": "500"
                    }
                ]
            }
    ```
1. Ejecute ahora la consulta `https://graph.microsoft.com/beta/serviceprincipals/{Service Principal Id}/synchronization/jobs/{AD2AAD Provisioning id}/schema`.
 
    Ejemplo: https://graph.microsoft.com/beta/serviceprincipals/653c0018-51f4-4736-a3a3-94da5dcb6862/synchronization/jobs/AD2AADProvisioning.e9287a7367e444c88dc67a531c36d8ec/schema

   Reemplace `{Service Principal Id}` y `{AD2ADD Provisioning Id}` con sus valores.

1. Esta consulta devuelve el esquema.

   ![Esquema devuelto](media/concept-attributes/schema1.png)
 
## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es el aprovisionamiento?](what-is-provisioning.md)
- [¿Qué es el aprovisionamiento en la nube de Azure AD Connect?](what-is-cloud-provisioning.md)