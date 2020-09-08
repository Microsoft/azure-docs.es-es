---
title: Interpretación del esquema de registro de auditoría de Azure Active Directory en Azure Monitor | Microsoft Docs
description: Se describe el esquema de registro de auditoría de Azure AD para su uso en Azure Monitor.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 4b18127b-d1d0-4bdc-8f9c-6a4c991c5f75
ms.service: active-directory
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/18/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: cc7a5e2e45cd3d76c0d8af612291ecaf9893b5b5
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89231102"
---
# <a name="interpret-the-azure-ad-audit-logs-schema-in-azure-monitor-preview"></a>Interpretación del esquema de registros de auditoría de Azure AD en Azure Monitor (versión preliminar)

En este artículo se describe el esquema de registro de auditoría de Azure Active Directory (Azure AD) en Azure Monitor. Cada entrada de registro individual se almacena como texto, con formato de blob JSON, como se muestra en los dos ejemplos siguientes: 

```json
{ 
    "records": [ 
    { 
        "time": "2018-03-17T00:14:31.2585575Z", 
        "operationName": "Change password (self-service)",
        "operationVersion": "1.0",
        "category": "Audit", 
        "tenantId": "bf85dc9d-cb43-44a4-80c4-469e8c58249e", 
        "resultType": "Success", 
        "resultSignature": "-1", 
        "resultDescription": "None", 
        "durationMs": "-1", 
        "correlationId": "60d5e89a-b890-413f-9e25-a047734afe9f", 
        "identity": "sreens@wingtiptoysonline.com", 
        "Level": "Informational", 
        "location": "WUS", 
        "properties": { 
            "identityType": "UPN", 
            "operationType": "Update", 
            "additionalDetails": "None", 
            "additionalTargets": "", 
            "targetUpdatedProperties": "", 
            "targetResourceType": "UPN__TenantContextID__PUID__ObjectID__ObjectClass", 
            "targetResourceName": "sreens@wingtiptoysonline.com__bf85dc9d-cb43-44a4-80c4-469e8c58249e__1003BFFD9FEB17DB__7a408bdd-7d97-4574-8511-dd747b56465d__User", 
            "auditEventCategory": "UserManagement" 
        } 
    } 
    ] 
} 
```

```json
{ 
    "records": [ 
    { 
        "time": "2018-03-18T19:47:43.0368859Z", 
        "operationName": "Update service principal.", 
        "operationVersion": "1.0", 
        "category": "Audit", 
        "tenantId": "bf85dc9d-cb43-44a4-80c4-469e8c58249e", 
        "resultType": "Success", 
        "resultSignature": "-1", 
        "durationMs": "-1", 
        "callerIpAddress": "<null>", 
        "correlationId": "14916c7a-5a7d-44e8-9b06-74b49efb08ee", 
        "identity": "NA", 
        "Level": "Informational", 
        "properties": { 
            "identityType": "NA", 
            "operationType": "Update", 
            "additionalDetails": {}, 
            "additionalTargets": "", 
            "targetUpdatedProperties": [ 
            { 
                "Name": "Included Updated Properties", 
                "OldValue": null, 
                "NewValue": "" 
            }, 
            { 
                "Name": "TargetId.ServicePrincipalNames", 
                "OldValue": null, 
                "NewValue": "http://adapplicationregistry.onmicrosoft.com/salesforce.com/primary;cd3ed3de-93ee-400b-8b19-b61ef44a0f29" 
            } 
            ], 
        "targetResourceType": "Other__ObjectID__ObjectClass__Name__AppId__SPN", 
        "targetResourceName": "ServicePrincipal_ea70a262-4da3-440a-b396-9734ddfd9df2__ea70a262-4da3-440a-b396-9734ddfd9df2__ServicePrincipal__Salesforce__cd3ed3de-93ee-400b-8b19-b61ef44a0f29__http://adapplicationregistry.onmicrosoft.com/salesforce.com/primary;cd3ed3de-93ee-400b-8b19-b61ef44a0f29", 
        "auditEventCategory": "ApplicationManagement" 
      } 
    } 
    ] 
} 
```

```json
{
    "records": [
    {
        "time": "2018-12-10T00:03:46.6161822Z",
        "resourceId": "/tenants/7918d4b5-0442-4a97-be2d-36f9f9962ece/providers/Microsoft.aadiam",
        "operationName": "Update policy",
        "operationVersion": "1.0",
        "category": "AuditLogs",
        "tenantId": "7918d4b5-0442-4a97-be2d-36f9f9962ece",
        "resultSignature": "None",
        "durationMs": 0,
        "callerIpAddress": "<null>",
        "correlationId": "192298c1-0994-4dd6-b05a-a6c5984c31cb",
        "identity": "MS-PIM",
        "level": "Informational",
        "properties": {
            "id": "Directory_VNXV4_28148892",
            "category": "Policy",
            "correlationId": "192298c1-0994-4dd6-b05a-a6c5984c31cb",
            "result": 0,
            "resultReason": "",
            "activityDisplayName": "Update policy",
            "activityDateTime": "2018-12-10T00:03:46.6161822+00:00",
            "loggedByService": "Core Directory",
            "operationType": "Update",
            "initiatedBy": {},
            "targetResources": [
            {
                "id": "5e7a8ae7-165d-44a4-a4f4-6141f8c8ef40",
                "displayName": "Default Policy",
                "type": "Policy",
                "modifiedProperties": []
            }
            ],
            "additionalDetails": []
        }
    }
    ]
}

```

## <a name="field-and-property-descriptions"></a>Descripciones de campos y propiedades

| Nombre del campo | Descripción |
|------------|-------------|
| time       | Fecha y hora (UTC) |
| operationName | Nombre de la operación. |
| operationVersion | Versión de la API REST solicitada por el cliente. |
| category | Actualmente, *Auditoría* es el único valor admitido. |
| tenantId | GUID del inquilino asociado con los registros. |
| resultType | Resultado de la operación. El resultado puede ser *Correcto* o *Error*. |
| resultSignature |  Este campo no se ha asignado y puede ignorarlo de forma segura. | 
| resultDescription | Descripción adicional del resultado, cuando esté disponible. | 
| durationMs |  Este campo no se ha asignado y puede ignorarlo de forma segura. |
| callerIpAddress | Dirección IP del cliente que realizó la solicitud. | 
| correlationId | GUID opcional que el cliente pasa. Puede ayudar a poner en correlación las operaciones de cliente con las operaciones de servidor y es útil al realizar el seguimiento de los registros que se distribuyen entre servicios. |
| identity | Identidad del token que se ha presentado al realizar la solicitud. La identidad puede ser una cuenta de usuario, una cuenta del sistema o una entidad de servicio. |
| level | Tipo de mensaje. Para los registros de auditoría, el nivel siempre es *Informativo*. |
| ubicación | Ubicación del centro de datos. |
| properties | Enumera las propiedades admitidas relacionadas con un registro de auditoría. Para más información, consulte la siguiente tabla. | 

<br>

| Nombre de propiedad | Descripción |
|---------------|-------------|
| AuditEventCategory | Tipo de evento de auditoría. Puede ser *Administración de usuarios*, *Administración de aplicaciones* u otro tipo.|
| Tipo de identidad | El tipo puede ser *Aplicación* o *Usuario*. |
| Tipo de operación | Puede ser *Agregar*, *Actualizar*, *Eliminar* u *Otros*. |
| Tipo de recurso de destino | Especifica el tipo de recurso de destino en que se realizó la operación. El tipo puede ser *Aplicación*, *Usuario*, *Rol* o *Directiva*. | 
| Nombre de recurso de destino | Nombre del recurso de destino. Puede ser un nombre de aplicación, un nombre de rol, un nombre principal de usuario o un nombre de entidad de seguridad de servicio. |
| additionalTargets | Se enumeran las propiedades adicionales para operaciones específicas. Por ejemplo, para una operación de actualización, los valores antiguos y los nuevos valores se muestran en *targetUpdatedProperties*. | 

## <a name="next-steps"></a>Pasos siguientes

* [Interpretación del esquema de registros de inicio de sesión en Azure Monitor](reference-azure-monitor-sign-ins-log-schema.md)
* [Registros de Azure Diagnostics](../../azure-monitor/platform/platform-logs-overview.md)
* [Preguntas frecuentes y problemas conocidos](concept-activity-logs-azure-monitor.md#frequently-asked-questions)