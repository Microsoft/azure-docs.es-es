---
title: 'Solución de problemas de autenticación y autorización: Azure Event Hubs'
description: En este artículo se proporciona información sobre cómo solucionar problemas de autenticación y autorización con Azure Event Hubs.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: bed415c6180953b5a5728032a50d51618c084343
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87533893"
---
# <a name="troubleshoot-authentication-and-authorization-issues---azure-event-hubs"></a>Solución de problemas de autenticación y autorización: Azure Event Hubs
En el artículo [Solución de problemas de conectividad](troubleshooting-guide.md) se ofrecen sugerencias para solucionar problemas de conectividad con Azure Event Hubs. En este artículo se proporcionan sugerencias y recomendaciones para solucionar problemas de autenticación y autorización con Azure Event Hubs. 

## <a name="if-you-are-using-azure-active-directory"></a>Si usa Azure Active Directory
Si usa Azure Active Directory (Azure AD) para la autenticación y la autorización con Azure Event Hubs, confirme que la identidad que tiene acceso al centro de eventos es miembro del **rol de Azure** adecuado en el **ámbito de recursos** adecuado (grupo de consumidores, centro de eventos, espacio de nombres, grupo de recursos o suscripción).

### <a name="azure-roles"></a>Roles de Azure
- [Propietario de datos de Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-owner) para tener acceso completo a los recursos de Event Hubs.
- [Remitente de los datos de Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-receiver) para el acceso de emisión.
- [Receptor de datos de Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-sender) para el acceso de recepción.

### <a name="resource-scopes"></a>Ámbitos de recursos
- **Grupo de consumidores**: en este ámbito, la asignación de roles solo se aplica a esta entidad. Actualmente, Azure Portal no admite la asignación de un rol de Azure a una entidad de seguridad en este nivel. 
- **Centro de eventos:** la asignación de roles se aplica a la entidad de Event Hubs y al grupo de consumidores que hay en ella.
- **Espacio de nombres**: la asignación de roles abarca toda la topología de Event Hubs en el espacio de nombres y el grupo de consumidores que tiene asociado.
- **Grupo de recursos**: la asignación de roles se aplica a todos los recursos de Event Hubs del grupo de recursos.
- **Suscripción**: la asignación de roles se aplica a todos los recursos de Event Hubs de todos los grupos de recursos de la suscripción.

Para más información, consulte los siguientes artículos.

- [Autenticación de una aplicación con Azure Active Directory para acceder a recursos de Event Hubs](authenticate-application.md)
- [Autorización del acceso a recursos de Event Hubs mediante Azure Active Directory](authorize-access-azure-active-directory.md)

## <a name="if-you-are-using-shared-access-signatures-sas"></a>Si usa firmas de acceso compartido (SAS)
Si usa [SAS](authenticate-shared-access-signature.md), siga estos pasos: 

- Asegúrese de que la clave SAS que está usando es correcta. Si no es así, use la clave SAS correcta.
- Compruebe que la clave tiene los permisos correctos (enviar, recibir o administrar). Si no es así, use una clave que tenga el permiso que necesita. 
- Compruebe si la clave ha expirado. Se recomienda renovar la SAS antes de que caduque. Si hay un sesgo de reloj entre el cliente y los nodos de servicio de Event Hubs, el token de autenticación podría expirar antes de que el cliente se dé cuenta. El reloj actual de las cuentas de implementación tiene un sesgo de hasta 5 minutos; es decir, el cliente renueva el token 5 minutos antes de que caduque. Por lo tanto, si el sesgo del reloj es mayor que 5 minutos, el cliente puede observar errores de autenticación intermitentes.
- Si la **hora de inicio de la SAS** está establecida en **ahora**, es posible que vea errores intermitentes durante los primeros minutos debido al sesgo del reloj (diferencias en la hora actual en diferentes máquinas). Establezca la hora de inicio para que sea al menos 15 minutos atrás o no la configure en modo alguno. Lo mismo suele aplicarse también a la hora de expiración. 

Para más información, consulte los siguientes artículos. 

- [Autenticación con firmas de acceso compartido (SAS)](authenticate-shared-access-signature.md) 
- [Autorización del acceso a recursos de Event Hubs mediante firmas de acceso compartido](authorize-access-shared-access-signature.md)

## <a name="next-steps"></a>Pasos siguientes

Vea los artículos siguientes:

* [Solución de problemas de conectividad](troubleshooting-guide.md)
