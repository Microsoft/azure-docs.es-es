---
title: Autenticación y autorización de Azure Service Bus | Microsoft Docs
description: Autentique aplicaciones en Service Bus con la autenticación de firma de acceso compartido (SAS).
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: d5d0ed03c869bd574e4cfaa52ac7b62e8cb7fb98
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88066869"
---
# <a name="service-bus-authentication-and-authorization"></a>Autenticación y autorización de Service Bus
Hay dos maneras de autenticar y autorizar el acceso a los recursos de Azure Service Bus: Azure Activity Directory (Azure AD) y firma de acceso compartido (SAS). En este artículo se proporcionan detalles sobre el uso de estos dos tipos de mecanismos de seguridad. 

## <a name="azure-active-directory"></a>Azure Active Directory
La integración de Azure AD para recursos de Service Bus proporciona control de acceso basado en rol (RBAC) para el control específico de acceso de los clientes a los recursos. Puede usar el control de acceso basado en rol (RBAC) para conceder permisos a una entidad de seguridad, que puede ser un usuario, un grupo o una entidad de servicio de aplicación. Azure AD autentica la entidad de seguridad para devolver un token de OAuth 2.0. El token se puede usar para autorizar una solicitud de acceso a un recurso de Service Bus (cola, tema, etc.).

Consulte los siguientes artículos para más información sobre la autenticación con Azure AD:

- [Autenticación con identidades administradas](service-bus-managed-service-identity.md)
- [Autenticación desde una aplicación](authenticate-application.md)

> [!NOTE]
> [API REST de Service Bus](/rest/api/servicebus/) admite la autenticación de OAuth con Azure AD.

> [!IMPORTANT]
> La autorización de usuarios o aplicaciones mediante un token de OAuth 2.0 devuelto por Azure AD proporciona una seguridad superior y facilidad de uso sobre las firmas de acceso compartido (SAS). Con Azure AD, no hay ninguna necesidad de almacenar los tokens en su código y arriesgarse a posibles vulnerabilidades de seguridad. Se recomienda usar Azure AD con las aplicaciones de Azure Service Bus siempre que sea posible. 

## <a name="shared-access-signature"></a>Firma de acceso compartido
La [autenticación de SAS](service-bus-sas.md) le permite conceder acceso a un usuario a los recursos de Service Bus con derechos específicos. La autenticación SAS en Service Bus implica la configuración de una clave criptográfica con derechos asociados en un recurso de Service Bus. Los clientes pueden obtener acceso a ese recurso presentando un token SAS, que consta del URI del recurso al que se tiene acceso y una fecha de expiración firmada con la clave configurada.

Puede configurar claves para SAS en un espacio de nombres de Service Bus. La clave se aplica a todas las entidades de mensajes de ese espacio de nombres. También puede configurar claves en temas y colas de Service Bus. SAS también es compatible con [Azure Relay](../azure-relay/relay-authentication-and-authorization.md).

Para usar SAS, puede configurar un objeto [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) en un espacio de nombres, cola o tema. Esta regla está formada por los siguientes elementos:

* *KeyName*: identifica la regla.
* *PrimaryKey*: es una clave criptográfica usada para firmar o validar tokens SAS.
* *SecondaryKey*: es una clave criptográfica usada para firmar o validar tokens SAS.
* *Rights*: representa la recopilación de derechos de **escucha**, **envío** o **administración** concedidos.

Las reglas de autorización configuradas en el nivel de espacio de nombres pueden conceder acceso a todas las entidades de un espacio de nombres a los clientes con tokens firmados con la clave correspondiente. Se puede configurar un máximo de 12 reglas de autorización en un espacio de nombres, cola o tema de Service Bus. De forma predeterminada, se configura un objeto [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) con todos los derechos para cada espacio de nombres cuando se aprovisiona por primera vez.

Para obtener acceso a una entidad, el cliente requiere un token SAS generado con un objeto [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule)determinado. El token SAS se genera mediante el HMAC-SHA256 de una cadena de recurso que consta del URI del recurso al que se notifica el acceso y una fecha de expiración con una clave criptográfica asociada a la regla de autorización.

La compatibilidad de la autenticación de SAS con Service Bus se incluye en el SDK .NET de Azure 2.0 y versiones posteriores. SAS incluye compatibilidad con un objeto [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule). Todas las API que aceptan una cadena de conexión como parámetro incluyen compatibilidad con cadenas de conexión SAS.

> [!IMPORTANT]
> Si usa Azure Active Directory Access Control (también conocido como Access Control Service o ACS) junto con Service Bus, tenga en cuenta que la compatibilidad con este método ahora está limitada y debe [migrar la aplicación para usar SAS](service-bus-migrate-acs-sas.md) o usar la autenticación de OAuth 2.0 con Azure AD (recomendado). Para más información sobre el desuso de ACS, consulte [esta entrada de blog](/archive/blogs/servicebus/upcoming-changes-to-acs-enabled-namespaces).

## <a name="next-steps"></a>Pasos siguientes
Consulte los siguientes artículos para más información sobre la autenticación con Azure AD:

- [Autenticación con identidades administradas](service-bus-managed-service-identity.md)
- [Autenticación desde una aplicación](authenticate-application.md)

Consulte los siguientes artículos para más información sobre la autenticación con SAS:

- [Autenticación con SAS](service-bus-sas.md)