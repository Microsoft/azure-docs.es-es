---
title: 'Uso de Microsoft Graph API para configurar el aprovisionamiento: Azure Active Directory | Microsoft Docs'
description: ¿Necesita configurar el aprovisionamiento de varias instancias de una aplicación? Aprenda a ahorrar tiempo mediante el uso de Microsoft Graph API para automatizar la configuración del aprovisionamiento automático.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: how-to
ms.date: 11/15/2019
ms.author: kenwith
ms.reviewer: arvinh
ms.openlocfilehash: 17660df34c8039ae96440c417aef051d51a5c91c
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88234760"
---
# <a name="configure-provisioning-using-microsoft-graph-apis"></a>Configuración del aprovisionamiento mediante las Microsoft Graph API

Azure Portal ofrece una manera cómoda de configurar el aprovisionamiento de aplicaciones individuales de una en una. Pero si va a crear varias instancias de una aplicación, o incluso cientos, puede ser más fácil automatizar la creación y la configuración de las aplicaciones con Microsoft Graph API. En este artículo se explica cómo automatizar la configuración de aprovisionamiento a través de las API. Este método se usa normalmente para aplicaciones como [Amazon Web Services](../saas-apps/amazon-web-service-tutorial.md#configure-azure-ad-sso).

**Información general de los pasos para usar las Microsoft Graph API para automatizar la configuración del aprovisionamiento**


|Paso  |Detalles  |
|---------|---------|
|[Paso 1. Crear la aplicación de galería](#step-1-create-the-gallery-application)     |Iniciar sesión en el cliente API <br> Recuperar la plantilla de la aplicación de galería <br> Crear la aplicación de galería         |
|[Paso 2. Crear el trabajo de aprovisionamiento basado en la plantilla](#step-2-create-the-provisioning-job-based-on-the-template)     |Recuperar la plantilla para el conector de aprovisionamiento <br> Crear el trabajo de aprovisionamiento         |
|[Paso 3. Autorizar el acceso](#step-3-authorize-access)     |Probar la conexión con la aplicación <br> Guardar las credenciales         |
|[Paso 4. Iniciar el trabajo de aprovisionamiento](#step-4-start-the-provisioning-job)     |Inicio del trabajo         |
|[Paso 5. Supervisar el aprovisionamiento](#step-5-monitor-provisioning)     |Comprobar el estado del trabajo de aprovisionamiento <br> Recuperar los registros de aprovisionamiento         |

> [!NOTE]
> Los objetos de respuesta que se muestran en este artículo pueden estar abreviados para mejorar la legibilidad. Todas las propiedades se devolverán desde una llamada real.

## <a name="step-1-create-the-gallery-application"></a>Paso 1: Crear la aplicación de galería

### <a name="sign-in-to-microsoft-graph-explorer-recommended-postman-or-any-other-api-client-you-use"></a>Iniciar sesión en el Explorador de Microsoft Graph (recomendado), Postman o cualquier otro cliente de API que use

1. Inicie el [Explorador de Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer).
1. Seleccione el botón "Iniciar sesión con Microsoft" e inicie sesión con las credenciales de administrador de la aplicación o de administrador global de Azure AD.

    ![Inicio de sesión en Graph](./media/application-provisioning-configure-api/wd_export_02.png)

1. Una vez haya iniciado sesión correctamente, verá los detalles de la cuenta de usuario en el panel izquierdo.

### <a name="retrieve-the-gallery-application-template-identifier"></a>Recuperar el identificador de plantilla de la aplicación de galería
Las aplicaciones de la galería de aplicaciones de Azure AD tienen una [plantilla de aplicación](/graph/api/applicationtemplate-list?tabs=http&view=graph-rest-beta) cada una, que describe los metadatos de esa aplicación. Con esta plantilla, puede crear una instancia de la aplicación y la entidad de servicio en el inquilino para la administración.

#### <a name="request"></a>*Solicitud*

<!-- {
  "blockType": "request",
  "name": "get_applicationtemplates"
}-->

```msgraph-interactive
GET https://graph.microsoft.com/beta/applicationTemplates
```

#### <a name="response"></a>*Respuesta*

<!-- {
  "blockType": "response",
  "truncated": true,
  "@odata.type": "microsoft.graph.applicationTemplate",
  "isCollection": true
} -->

```http
HTTP/1.1 200 OK
Content-type: application/json

{
  "value": [
  {
    "id": "8b1025e4-1dd2-430b-a150-2ef79cd700f5",
        "displayName": "Amazon Web Services (AWS)",
        "homePageUrl": "http://aws.amazon.com/",
        "supportedSingleSignOnModes": [
             "password",
             "saml",
             "external"
         ],
         "supportedProvisioningTypes": [
             "sync"
         ],
         "logoUrl": "https://az495088.vo.msecnd.net/app-logo/aws_215.png",
         "categories": [
             "developerServices"
         ],
         "publisher": "Amazon",
         "description": null    
  
}
```

### <a name="create-the-gallery-application"></a>Crear la aplicación de galería

Use el id. de plantilla recuperado para la aplicación en el último paso para [crear una instancia](/graph/api/applicationtemplate-instantiate?tabs=http&view=graph-rest-beta) de la aplicación y la entidad de servicio en el inquilino.

#### <a name="request"></a>*Solicitud*

<!-- {
  "blockType": "request",
  "name": "applicationtemplate_instantiate"
}-->

```msgraph-interactive
POST https://graph.microsoft.com/beta/applicationTemplates/{id}/instantiate
Content-type: application/json

{
  "displayName": "AWS Contoso"
}
```

#### <a name="response"></a>*Respuesta*


<!-- {
  "blockType": "response",
  "truncated": true,
  "@odata.type": "microsoft.graph.applicationServicePrincipal"
} -->

```http
HTTP/1.1 201 OK
Content-type: application/json


{
    "application": {
        "objectId": "cbc071a6-0fa5-4859-8g55-e983ef63df63",
        "appId": "92653dd4-aa3a-3323-80cf-e8cfefcc8d5d",
        "applicationTemplateId": "8b1025e4-1dd2-430b-a150-2ef79cd700f5",
        "displayName": "AWS Contoso",
        "homepage": "https://signin.aws.amazon.com/saml?metadata=aws|ISV9.1|primary|z",
        "replyUrls": [
            "https://signin.aws.amazon.com/saml"
        ],
        "logoutUrl": null,
        "samlMetadataUrl": null,
    },
    "servicePrincipal": {
        "objectId": "f47a6776-bca7-4f2e-bc6c-eec59d058e3e",
        "appDisplayName": "AWS Contoso",
        "applicationTemplateId": "8b1025e4-1dd2-430b-a150-2ef79cd700f5",
        "appRoleAssignmentRequired": true,
        "displayName": "My custom name",
        "homepage": "https://signin.aws.amazon.com/saml?metadata=aws|ISV9.1|primary|z",
        "replyUrls": [
            "https://signin.aws.amazon.com/saml"
        ],
        "servicePrincipalNames": [
            "93653dd4-aa3a-4323-80cf-e8cfefcc8d7d"
        ],
        "tags": [
            "WindowsAzureActiveDirectoryIntegratedApp"
        ],
    }
}
```

## <a name="step-2-create-the-provisioning-job-based-on-the-template"></a>Paso 2: Crear el trabajo de aprovisionamiento basado en la plantilla

### <a name="retrieve-the-template-for-the-provisioning-connector"></a>Recuperar la plantilla para el conector de aprovisionamiento

Las aplicaciones de la galería que están habilitadas para el aprovisionamiento tienen plantillas para simplificar la configuración. Use la solicitud siguiente para [recuperar la plantilla para la configuración del aprovisionamiento](/graph/api/synchronization-synchronizationtemplate-list?tabs=http&view=graph-rest-beta). Tenga en cuenta que tendrá que proporcionar el identificador. El identificador hace referencia al recurso anterior, que en este caso es ServicePrincipal. 

#### <a name="request"></a>*Solicitud*

<!-- {
  "blockType": "request",
  "name": "get_synchronizationtemplate"
}-->
```msgraph-interactive
GET https://graph.microsoft.com/beta/servicePrincipals/{id}/synchronization/templates
```


#### <a name="response"></a>*Respuesta*
<!-- {
  "blockType": "response",
  "truncated": true,
  "@odata.type": "microsoft.graph.synchronizationTemplate",
  "isCollection": true
} -->
```http
HTTP/1.1 200 OK

{
    "value": [
        {
            "id": "aws",
            "factoryTag": "aws",
            "schema": {
                    "directories": [],
                    "synchronizationRules": []
                    }
        }
    ]
}
```

### <a name="create-the-provisioning-job"></a>Crear el trabajo de aprovisionamiento
Para habilitar el aprovisionamiento, primero deberá [crear un trabajo](/graph/api/synchronization-synchronizationjob-post?tabs=http&view=graph-rest-beta). Use la solicitud siguiente para crear un trabajo de aprovisionamiento. Use el elemento templateId del paso anterior al especificar la plantilla que se va a usar para el trabajo.

#### <a name="request"></a>*Solicitud*
<!-- {
  "blockType": "request",
  "name": "create_synchronizationjob_from_synchronization"
}-->
```msgraph-interactive
POST https://graph.microsoft.com/beta/servicePrincipals/{id}/synchronization/jobs
Content-type: application/json

{ 
    "templateId": "aws"
}
```

#### <a name="response"></a>*Respuesta*
<!-- {
  "blockType": "response",
  "truncated": true,
  "@odata.type": "microsoft.graph.synchronizationJob"
} -->
```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "id": "{jobId}",
    "templateId": "aws",
    "schedule": {
        "expiration": null,
        "interval": "P10675199DT2H48M5.4775807S",
        "state": "Disabled"
    },
    "status": {
        "countSuccessiveCompleteFailures": 0,
        "escrowsPruned": false,
        "synchronizedEntryCountByType": [],
        "code": "NotConfigured",
        "lastExecution": null,
        "lastSuccessfulExecution": null,
        "lastSuccessfulExecutionWithExports": null,
        "steadyStateFirstAchievedTime": "0001-01-01T00:00:00Z",
        "steadyStateLastAchievedTime": "0001-01-01T00:00:00Z",
        "quarantine": null,
        "troubleshootingUrl": null
    }
}
```

## <a name="step-3-authorize-access"></a>Paso 3: Autorizar el acceso

### <a name="test-the-connection-to-the-application"></a>Probar la conexión con la aplicación

Pruebe la conexión con la aplicación de otro fabricante. El ejemplo siguiente es para una aplicación que requiere clientSecret y secretToken. Cada aplicación tiene sus propios requisitos. Las aplicaciones a menudo usan BaseAddress en lugar de ClientSecret. Para determinar qué credenciales requiere la aplicación, vaya a la página de configuración de aprovisionamiento de la aplicación y, en el modo de desarrollador, haga clic en Probar conexión. El tráfico de red mostrará los parámetros que se usan para las credenciales. Puede encontrar la lista completa de credenciales [aquí](/graph/api/synchronization-synchronizationjob-validatecredentials?tabs=http&view=graph-rest-beta). 

#### <a name="request"></a>*Solicitud*
```msgraph-interactive
POST https://graph.microsoft.com/beta/servicePrincipals/{id}/synchronization/jobs/{id}/validateCredentials
{ 
    credentials: [ 
        { key: "ClientSecret", value: "xxxxxxxxxxxxxxxxxxxxx" },
        { key: "SecretToken", value: "xxxxxxxxxxxxxxxxxxxxx" }
    ]
}
```
#### <a name="response"></a>*Respuesta*
<!-- {
  "blockType": "response",
  "truncated": true,
  "@odata.type": "microsoft.graph.None"
} -->
```http
HTTP/1.1 204 No Content
```

### <a name="save-your-credentials"></a>Guardar las credenciales

Para configurar el aprovisionamiento, es necesario establecer una relación de confianza entre Azure AD y la aplicación. Autorice el acceso a la aplicación de terceros. El ejemplo siguiente es para una aplicación que requiere clientSecret y secretToken. Cada aplicación tiene sus propios requisitos. Revise la [documentación de la API](/graph/api/synchronization-synchronizationjob-validatecredentials?tabs=http&view=graph-rest-beta) para ver las opciones disponibles. 

#### <a name="request"></a>*Solicitud*
```msgraph-interactive
PUT https://graph.microsoft.com/beta/servicePrincipals/{id}/synchronization/secrets 
 
{ 
    value: [ 
        { key: "ClientSecret", value: "xxxxxxxxxxxxxxxxxxxxx" },
        { key: "SecretToken", value: "xxxxxxxxxxxxxxxxxxxxx" }
    ]
}
```

#### <a name="response"></a>*Respuesta*
<!-- {
  "blockType": "response",
  "truncated": true,
  "@odata.type": "microsoft.graph.None"
} -->
```http
HTTP/1.1 204 No Content
```

## <a name="step-4-start-the-provisioning-job"></a>Paso 4: Iniciar el trabajo de aprovisionamiento
Ahora que el trabajo de aprovisionamiento está configurado, use el siguiente comando para [iniciar el trabajo](/graph/api/synchronization-synchronizationjob-start?tabs=http&view=graph-rest-beta). 


#### <a name="request"></a>*Solicitud*
<!-- {
  "blockType": "request",
  "name": "synchronizationjob_start"
}-->
```http
POST https://graph.microsoft.com/beta/servicePrincipals/{id}/synchronization/jobs/{jobId}/start
```

#### <a name="response"></a>*Respuesta*
<!-- {
  "blockType": "response",
  "truncated": true,
  "@odata.type": "microsoft.graph.None"
} -->
```http
HTTP/1.1 204 No Content
```


## <a name="step-5-monitor-provisioning"></a>Paso 5: Supervisar el aprovisionamiento

### <a name="monitor-the-provisioning-job-status"></a>Supervisar el estado del trabajo de aprovisionamiento

Ahora que el trabajo de aprovisionamiento está en ejecución, use el siguiente comando para realizar el seguimiento del progreso del ciclo de aprovisionamiento actual, así como las estadísticas hasta la fecha, como el número de usuarios y grupos que se han creado en el sistema de destino. 

#### <a name="request"></a>*Solicitud*
<!-- {
  "blockType": "request",
  "name": "get_synchronizationjob"
}-->
```msgraph-interactive
GET https://graph.microsoft.com/beta/servicePrincipals/{id}/synchronization/jobs/{jobId}/
```

#### <a name="response"></a>*Respuesta*
<!-- {
  "blockType": "response",
  "truncated": true,
  "@odata.type": "microsoft.graph.synchronizationJob"
} -->
```http
HTTP/1.1 200 OK
Content-type: application/json
Content-length: 2577

{
    "id": "{jobId}",
    "templateId": "aws",
    "schedule": {
        "expiration": null,
        "interval": "P10675199DT2H48M5.4775807S",
        "state": "Disabled"
    },
    "status": {
        "countSuccessiveCompleteFailures": 0,
        "escrowsPruned": false,
        "synchronizedEntryCountByType": [],
        "code": "Paused",
        "lastExecution": null,
        "lastSuccessfulExecution": null,
        "progress": [],
        "lastSuccessfulExecutionWithExports": null,
        "steadyStateFirstAchievedTime": "0001-01-01T00:00:00Z",
        "steadyStateLastAchievedTime": "0001-01-01T00:00:00Z",
        "quarantine": null,
        "troubleshootingUrl": null
    },
    "synchronizationJobSettings": [
      {
          "name": "QuarantineTooManyDeletesThreshold",
          "value": "500"
      }
    ]
}
```


### <a name="monitor-provisioning-events-using-the-provisioning-logs"></a>Supervisar eventos de aprovisionamiento mediante los registros de aprovisionamiento
Además de supervisar el estado del trabajo de aprovisionamiento, puede usar los [registros de aprovisionamiento](/graph/api/provisioningobjectsummary-list?tabs=http&view=graph-rest-beta) para consultar todos los eventos que se están produciendo (por ejemplo, realizar una consulta para un usuario determinado y determinar si se aprovisionó correctamente).

#### <a name="request"></a>*Solicitud*
```msgraph-interactive
GET https://graph.microsoft.com/beta/auditLogs/provisioning
```
#### <a name="response"></a>*Respuesta*
<!-- {
  "blockType": "response",
  "truncated": true,
  "@odata.type": "microsoft.graph.provisioningObjectSummary",
  "name": "list_provisioningobjectsummary_error"
} -->

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "@odata.context": "https://graph.microsoft.com/beta/$metadata#auditLogs/provisioning",
    "value": [
        {
            "id": "gc532ff9-r265-ec76-861e-42e2970a8218",
            "activityDateTime": "2019-06-24T20:53:08Z",
            "tenantId": "7928d5b5-7442-4a97-ne2d-66f9j9972ecn",
            "jobId": "BoxOutDelta.7928d5b574424a97ne2d66f9j9972ecn",
            "cycleId": "44576n58-v14b-70fj-8404-3d22tt46ed93",
            "changeId": "eaad2f8b-e6e3-409b-83bd-e4e2e57177d5",
            "action": "Create",
            "durationInMilliseconds": 2785,
            "sourceSystem": {
                "id": "0404601d-a9c0-4ec7-bbcd-02660120d8c9",
                "displayName": "Azure Active Directory",
                "details": {}
            },
            "targetSystem": {
                "id": "cd22f60b-5f2d-1adg-adb4-76ef31db996b",
                "displayName": "Box",
                "details": {
                    "ApplicationId": "f2764360-e0ec-5676-711e-cd6fc0d4dd61",
                    "ServicePrincipalId": "chc46a42-966b-47d7-9774-576b1c8bd0b8",
                    "ServicePrincipalDisplayName": "Box"
                }
            },
            "initiatedBy": {
                "id": "",
                "displayName": "Azure AD Provisioning Service",
                "initiatorType": "system"
            },
            "sourceIdentity": {
                "id": "5e6c9rae-ab4d-5239-8ad0-174391d110eb",
                "displayName": "Self-service Pilot",
                "identityType": "Group",
                "details": {}
            },
            "targetIdentity": {
                "id": "",
                "displayName": "",
                "identityType": "Group",
                "details": {}
            },
            "statusInfo": {
                "@odata.type": "#microsoft.graph.statusDetails",
                "status": "failure",
                "errorCode": "BoxEntryConflict",
                "reason": "Message: Box returned an error response with the HTTP status code 409.  This response indicates that a user or a group already exisits with the same name. This can be avoided by identifying and removing the conflicting user from Box via the Box administrative user interface, or removing the current user from the scope of provisioning either by removing their assignment to the Box application in Azure Active Directory or adding a scoping filter to exclude the user.",
                "additionalDetails": null,
                "errorCategory": "NonServiceFailure",
                "recommendedAction": null
            },
            "provisioningSteps": [
                {
                    "name": "EntryImportAdd",
                    "provisioningStepType": "import",
                    "status": "success",
                    "description": "Received Group 'Self-service Pilot' change of type (Add) from Azure Active Directory",
                    "details": {}
                },
                {
                    "name": "EntrySynchronizationAdd",
                    "provisioningStepType": "matching",
                    "status": "success",
                    "description": "Group 'Self-service Pilot' will be created in Box (Group is active and assigned in Azure Active Directory, but no matching Group was found in Box)",
                    "details": {}
                },
                {
                    "name": "EntryExportAdd",
                    "provisioningStepType": "export",
                    "status": "failure",
                    "description": "Failed to create Group 'Self-service Pilot' in Box",
                    "details": {
                        "ReportableIdentifier": "Self-service Pilot"
                    }
                }
            ],
            "modifiedProperties": [
                {
                    "displayName": "objectId",
                    "oldValue": null,
                    "newValue": "5e0c9eae-ad3d-4139-5ad0-174391d110eb"
                },
                {
                    "displayName": "displayName",
                    "oldValue": null,
                    "newValue": "Self-service Pilot"
                },
                {
                    "displayName": "mailEnabled",
                    "oldValue": null,
                    "newValue": "False"
                },
                {
                    "displayName": "mailNickname",
                    "oldValue": null,
                    "newValue": "5ce25n9a-4c5f-45c9-8362-ef3da29c66c5"
                },
                {
                    "displayName": "securityEnabled",
                    "oldValue": null,
                    "newValue": "True"
                },
                {
                    "displayName": "Name",
                    "oldValue": null,
                    "newValue": "Self-service Pilot"
                }
            ]
       }
    ]
}

```
## <a name="related-articles"></a>Artículos relacionados

- [Revisar la documentación de sincronización de Microsoft Graph](/graph/api/resources/synchronization-overview?view=graph-rest-beta)
- [Integración de una aplicación SCIM personalizada con Azure AD](use-scim-to-provision-users-and-groups.md)