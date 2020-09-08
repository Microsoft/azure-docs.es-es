---
title: Uso de SCIM, Microsoft Graph y Azure AD para aprovisionar usuarios y enriquecer aplicaciones con datos
description: Use SCIM y Microsoft Graph juntos para aprovisionar a los usuarios y enriquecer la aplicación con los datos necesarios.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: reference
ms.date: 04/26/2020
ms.author: kenwith
ms.reviewer: arvinh, celested
ms.openlocfilehash: b69e2c9b12b2db34f3eb70e54d2c6aede6b54784
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88235509"
---
# <a name="using-scim-and-microsoft-graph-together-to-provision-users-and-enrich-your-application-with-the-data-it-needs"></a>Uso de SCIM y Microsoft Graph juntos para aprovisionar a los usuarios y enriquecer su aplicación con los datos necesarios

**Público de destino:** Este artículo está dirigido a los desarrolladores que crean aplicaciones integradas con Azure Active Directory (Azure AD). Si desea usar aplicaciones ya integradas con Azure AD, como Zoom, ServiceNow y DropBox, puede omitir este artículo y revisar los [tutoriales](../saas-apps/tutorial-list.md) específicos de la aplicación o consultar [Funcionamiento del aprovisionamiento](./how-provisioning-works.md).

**Escenarios comunes**

Azure AD proporciona un servicio preconfigurado para el aprovisionamiento y una plataforma extensible en la que compilar las aplicaciones. El árbol de decisión describe el modo en que un desarrollador usaría [SCIM](https://aka.ms/scimoverview) y [Microsoft Graph](/graph/overview) para automatizar el aprovisionamiento. 

> [!div class="checklist"]
> * Creación automática de usuarios en la aplicación
> * Eliminación automática de usuarios de la aplicación cuando no deban ya tener acceso
> * Integración de la aplicación con varios proveedores de identidades para el aprovisionamiento
> * Enriquecimiento de la aplicación con datos de servicios de Microsoft, como Teams, Outlook y Office
> * Creación, actualización y eliminación automáticas de usuarios y grupos en Azure AD y Active Directory

![Árbol de decisión de SCIM Graph](./media/user-provisioning/scim-graph.png)

## <a name="scenario-1-automatically-create-users-in-my-app"></a>Escenario 1: Creación automática de usuarios en la aplicación
Hoy en día, los administradores de TI aprovisionan a los usuarios mediante la creación manual de cuentas de usuario o la carga periódica de archivos CSV en mi aplicación. El proceso es lento para los clientes y ralentiza la adopción de la aplicación. Lo único que hace falta es información básica del usuario, como el nombre, el correo electrónico y el nombre principal de usuario para crearlo. 

**Recomendación:** 
* Si los clientes usan varios proveedores de identidades, y no desea mantener un motor de sincronización para integrarlos con cada uno, admita un punto de conexión [/Users](https://aka.ms/scimreferencecode) compatible con SCIM. Los clientes podrán usar fácilmente este punto de conexión para la integración con el servicio de aprovisionamiento de Azure AD y crear automáticamente cuentas de usuario cuando necesiten tener acceso. Puede compilar el punto de conexión una vez y será compatible con todos los proveedores de identidades. Consulte la solicitud de ejemplo siguiente para ver cómo se crearía un usuario con SCIM.
* Si necesita datos de usuario que se encuentran en el objeto de usuario en Azure AD y otros datos de Microsoft, considere la posibilidad de crear un punto de conexión de SCIM para el aprovisionamiento de usuarios y llamar a Microsoft Graph para obtener el resto de los datos. 

```json
POST /Users
{
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User",
        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"],
    "externalId": "0a21f0f2-8d2a-4f8e-bf98-7363c4aed4ef",
    "userName": "BillG",
    "active": true,
    "meta": {
        "resourceType": "User"
    },
    "name": {
        "formatted": "Bill Gates",
        "familyName": "Gates",
        "givenName": "Bill"
    },
    "roles": []
}
```
    
## <a name="scenario-2-automatically-remove-users-from-my-app"></a>Escenario 2: Eliminación automática de usuarios de la aplicación
Los clientes que usan mi aplicación están centrados en la seguridad, y entre sus requisitos de gobernanza, está el de quitar las cuentas cuando los empleados ya no las necesiten. ¿Cómo se puede automatizar el desaprovisionamiento de la aplicación?

**Recomendación:** admita un punto de conexión /Users compatible con SCIM. El servicio de aprovisionamiento de Azure AD enviará solicitudes para deshabilitar y eliminar el usuario cuando ya no vaya a tener acceso nunca más. Se recomienda la compatibilidad con la deshabilitación y la eliminación de usuarios. Vea los ejemplos siguientes de cómo es una solicitud de deshabilitación y eliminación. 

Deshabilitación del usuario
```json
PATCH /Users/5171a35d82074e068ce2 HTTP/1.1
{
    "Operations": [
        {
            "op": "Replace",
            "path": "active",
            "value": false
        }
    ],
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:PatchOp"
    ]
}
```
Eliminación de usuario
```json
DELETE /Users/5171a35d82074e068ce2 HTTP/1.1
```

## <a name="scenario-3-automate-managing-group-memberships-in-my-app"></a>Escenario 3: Automatización de la administración de las pertenencias a grupos en la aplicación
Mi aplicación se basa en grupos de acceso a varios recursos, y los clientes quieren volver a usar los grupos que tienen en Azure AD. ¿Cómo se pueden importar los grupos desde Azure AD y mantenerlos actualizados a medida que cambian las pertenencias?  

**Recomendación:** admita un [punto de conexión](https://aka.ms/scimreferencecode) /Groups compatible con SCIM. El servicio de aprovisionamiento de Azure AD se encargará de crear grupos y administrar las actualizaciones de pertenencia en la aplicación. 

## <a name="scenario-4-enrich-my-app-with-data-from-microsoft-services-such-as-teams-outlook-and-onedrive"></a>Escenario 4: Enriquecimiento de la aplicación con datos de servicios de Microsoft, como Teams, Outlook y OneDrive
Mi aplicación está integrada en Microsoft Teams y depende de datos de mensajes. Además, los archivos de los usuarios se almacenan en OneDrive. ¿Cómo se puede enriquecer la aplicación con los datos de estos servicios y en Microsoft?

**Recomendación:** [Microsoft Graph](/graph/) es el punto de entrada para acceder a los datos de Microsoft. Cada carga de trabajo expone las API con los datos que necesita. Microsoft Graph se puede usar junto con el [aprovisionamiento de SCIM](./use-scim-to-provision-users-and-groups.md) en los escenarios anteriores. Puede usar SCIM para aprovisionar atributos de usuario básicos en la aplicación, al tiempo que llama a Graph para obtener cualquier otro dato que necesite. 

## <a name="scenario-5-track-changes-in-microsoft-services-such-as-teams-outlook-and-azure-ad"></a>Escenario 5: Seguimiento de los cambios en servicios de Microsoft, como Teams, Outlook y Azure AD
Se necesita poder realizar un seguimiento de los cambios en los mensajes de Teams y Outlook y reaccionar a ellos en tiempo real. ¿Cómo se pueden insertar estos cambios en la aplicación?

**Recomendación:** Microsoft Graph proporciona [notificaciones de cambios](/graph/webhooks) y [seguimiento de cambios](/graph/delta-query-overview) para varios recursos. Tenga en cuenta las siguientes limitaciones de las notificaciones de cambios:
- Si un receptor de eventos confirma un evento, pero por algún motivo no puede actuar sobre él, el evento podría perderse.
- No se garantiza que el orden en el que se reciben los cambios sea cronológico.
- Las notificaciones de cambios no siempre contienen los [datos de recursos](/graph/webhooks-with-resource-data). Por las razones mencionadas anteriormente, los desarrolladores suelen usar las notificaciones de cambios junto con el seguimiento de cambios en los escenarios de sincronización. 

## <a name="scenario-6-provision-users-and-groups-in-azure-ad"></a>Escenario 6: Aprovisionamiento de usuarios y grupos de Azure AD
Mi aplicación crea información sobre un usuario que los clientes necesitan en Azure AD. Podría tratarse de una aplicación de RR. HH. que administra las contrataciones, una aplicación de comunicaciones que crea números de teléfono para los usuarios o alguna otra aplicación que genera datos que serían valiosos en Azure AD. ¿Cómo se rellena el registro de usuarios en Azure AD con los datos? 

**Recomendación:** Microsoft Graph expone los puntos de conexión /Users y /Groups que puede integrar actualmente para aprovisionar a los usuarios en Azure AD. Tenga en cuenta que Azure Active Directory no admite volver a escribir esos usuarios en Active Directory. 

> [!NOTE]
> Microsoft cuenta con un servicio de aprovisionamiento que extrae datos de las aplicaciones de RR. HH., como Workday y SuccessFactors. Microsoft crea y administra estas integraciones. Para incorporar una nueva aplicación de RR. HH. a nuestro servicio, puede solicitarla en [UserVoice](https://feedback.azure.com/forums/374982-azure-active-directory-application-requests). 

## <a name="related-articles"></a>Artículos relacionados

- [Revisar la documentación de sincronización de Microsoft Graph](/graph/api/resources/synchronization-overview?view=graph-rest-beta)
- [Integración de una aplicación SCIM personalizada con Azure AD](use-scim-to-provision-users-and-groups.md)