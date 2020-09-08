---
title: Autorización del acceso con Azure Active Directory
description: En este artículo se proporciona información sobre cómo autorizar el acceso a recursos de Event Hubs mediante Azure Active Directory.
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: 101e40420493156c7b1a0c3c5b767eda023e62c6
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87831845"
---
# <a name="authorize-access-to-event-hubs-resources-using-azure-active-directory"></a>Autorización del acceso a recursos de Event Hubs mediante Azure Active Directory
Azure Event Hubs admite la autorización de solicitudes a recursos de Event Hubs mediante Azure Active Directory (Azure AD). Con Azure AD, puede usar el control de acceso basado en rol (RBAC) para conceder permisos a una entidad de seguridad, que puede ser un usuario o una entidad de servicio de aplicación. Para más información sobre los roles y las asignaciones de roles, consulte [Descripción de los distintos roles](../role-based-access-control/overview.md).

## <a name="overview"></a>Información general
Cuando una entidad de seguridad (un usuario o una aplicación) intenta acceder a un recurso de Event Hubs, la solicitud debe estar autorizada. Con Azure AD, el acceso a un recurso es un proceso de dos pasos. 

 1. En primer lugar, se autentica la identidad de la entidad de seguridad y se devuelve un token de OAuth 2.0. El nombre del recurso para solicitar un token es `https://eventhubs.azure.net/`. En el caso de los clientes de Kafka, el recurso para solicitar un token es `https://<namespace>.servicebus.windows.net`.
 1. Luego, el token se pasa como parte de una solicitud al servicio Event Hubs para autorizar el acceso al recurso especificado.

El paso de autenticación exige que una solicitud de aplicación contenga un token de acceso de OAuth 2.0 en tiempo de ejecución. Si una aplicación se está ejecutando dentro de una entidad de Azure, como puede ser una máquina virtual de Azure, un conjunto de escalado de máquinas virtuales o una aplicación de Azure Functions, puede usar una identidad administrada para acceder a los recursos. Para más información sobre cómo autenticar solicitudes realizadas por una identidad administrada al servicio Event Hubs, vea [Autenticación del acceso a recursos de Azure Event Hubs con Azure Active Directory e identidades administradas para recursos de Azure](authenticate-managed-identity.md). 

El paso de autorización exige que se asignen uno o varios roles de Azure a la entidad de seguridad. Azure Event Hubs proporciona roles de Azure que abarcan conjuntos de permisos para recursos de Event Hubs. Los roles que se asignan a una entidad de seguridad determinan los permisos que tiene esa entidad de seguridad. Para más información sobre los roles de Azure, consulte [Roles integrados de Azure para Azure Event Hubs](#azure-built-in-roles-for-azure-event-hubs). 

Las aplicaciones nativas y las aplicaciones web que realizan solicitudes a Event Hubs también pueden autorizar con Azure AD. Para obtener información sobre cómo solicitar un token de acceso y usarlo para autorizar solicitudes para recursos de Event Hubs, vea [Autenticación del acceso a Azure Event Hubs con Azure AD desde una aplicación](authenticate-application.md). 

## <a name="assign-azure-roles-for-access-rights"></a>Asignación de roles de Azure para derechos de acceso
Azure Active Directory (Azure AD) autoriza derechos de acceso a recursos protegidos mediante el [control de acceso basado en rol de Azure (RBAC de Azure)](../role-based-access-control/overview.md). Azure Event Hubs define un conjunto de roles integrados de Azure que abarcan conjuntos de permisos comunes utilizados para acceder a los datos del centro de eventos y también puede definir roles personalizados para acceder a los datos.

Cuando un rol de Azure se asigna a una entidad de seguridad de Azure AD, Azure concede a esa entidad de seguridad acceso a esos recursos. El acceso puede tener como ámbito el nivel de suscripción, el grupo de recursos, el espacio de nombres de Event Hubs o cualquier recurso que haya en él. Una entidad de seguridad de Azure AD puede ser un usuario, una entidad de servicio de aplicación o una [identidad de servicio administrada para recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md).

## <a name="azure-built-in-roles-for-azure-event-hubs"></a>Roles integrados de Azure para Azure Event Hubs
Azure proporciona los siguientes roles integrados de Azure para autorizar el acceso a datos de Event Hubs con Azure AD y OAuth:

- [Propietario de los datos de Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-owner): use este rol para proporcionar acceso completo a los recursos de Event Hubs.
- [Emisor de datos de Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-sender): use este rol para proporcionar acceso de emisión a los recursos de Event Hubs.
- [Receptor de datos de Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-receiver): use este rol para proporcionar acceso de consumo/recepción a los recursos de Event Hubs.

## <a name="resource-scope"></a>Ámbito de recursos 
Antes de asignar un rol de Azure a una entidad de seguridad, determine el ámbito de acceso que debería tener la entidad de seguridad. Los procedimientos recomendados dictan que siempre es mejor conceder únicamente el ámbito más restringido posible.

En la lista siguiente se describen los niveles en los que puede definir el ámbito de acceso a recursos de Event Hubs, empezando por el ámbito más restringido:

- **Grupo de consumidores**: en este ámbito, la asignación de roles solo se aplica a esta entidad. Actualmente, Azure Portal no admite la asignación de un rol de Azure a una entidad de seguridad en este nivel. 
- **Centro de eventos:** la asignación de roles se aplica a la entidad de Event Hubs y al grupo de consumidores que hay en ella.
- **Espacio de nombres**: la asignación de roles abarca toda la topología de Event Hubs en el espacio de nombres y el grupo de consumidores que tiene asociado.
- **Grupo de recursos**: la asignación de roles se aplica a todos los recursos de Event Hubs del grupo de recursos.
- **Suscripción**: la asignación de roles se aplica a todos los recursos de Event Hubs de todos los grupos de recursos de la suscripción.

> [!NOTE]
> - Tenga en cuenta que las asignaciones de roles de Azure pueden tardar hasta cinco minutos en propagarse. 
> - Este contenido se aplica a Event Hubs y a Event Hubs para Apache Kafka. Para más información sobre la compatibilidad con Event Hubs para Kafka, consulte [Event Hubs para Kafka: seguridad y autenticación](event-hubs-for-kafka-ecosystem-overview.md#security-and-authentication).


Para más información sobre cómo se definen los roles integrados, consulte [Descripción de definiciones de roles](../role-based-access-control/role-definitions.md#management-and-data-operations). Para más información acerca de la creación de roles personalizados de Azure, consulte [Roles personalizados de Azure](../role-based-access-control/custom-roles.md).



## <a name="samples"></a>Ejemplos
- [Ejemplos de Microsoft.Azure.EventHubs](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/Rbac). 
    
    En estos ejemplos se usa la biblioteca anterior **Microsoft.Azure.EventHubs**, pero se puede actualizar fácilmente para usar la biblioteca **Azure.Messaging.EventHubs** más reciente. Para que los ejemplos usen la biblioteca nueva en lugar de la anterior, consulte la [Guía para migrar de Microsoft.Azure.EventHubs a Azure.Messaging.EventHubs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/MigrationGuide.md).
- [Ejemplos de Azure.Messaging.EventHubs](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Azure.Messaging.EventHubs/ManagedIdentityWebApp)

    Este ejemplo se ha actualizado para usar la biblioteca **Azure.Messaging.EventHubs** más reciente.
- [Event Hubs para Kafka: ejemplos de OAuth](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth). 


## <a name="next-steps"></a>Pasos siguientes
- Para información sobre cómo asignar un rol integrado de Azure a una entidad de seguridad, consulte [Autenticación del acceso a recursos de Event Hubs mediante Azure Active Directory](authenticate-application.md).
- Obtenga información sobre [cómo crear roles personalizados con RBAC](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/Rbac/CustomRole).
- Obtenga información sobre [cómo usar Azure Active Directory con EH](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/Rbac/AzureEventHubsSDK).

Consulte los artículos relacionados siguientes:

- [Autenticación de solicitudes a Azure Event Hubs desde una aplicación mediante Azure Active Directory](authenticate-application.md)
- [Autenticación de una identidad administrada con Azure Active Directory para acceder a recursos de Event Hubs](authenticate-managed-identity.md)
- [Autenticación de solicitudes para Azure Event Hubs mediante firmas de acceso compartido](authenticate-shared-access-signature.md)
- [Autorización del acceso a recursos de Event Hubs mediante firmas de acceso compartido](authorize-access-shared-access-signature.md)
