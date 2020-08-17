---
title: ¿Qué son las notificaciones de mantenimiento del servicio de Azure?
description: Las notificaciones de mantenimiento del servicio permiten ver los mensajes de mantenimiento del servicio que publica Microsoft Azure.
ms.topic: conceptual
ms.date: 4/12/2018
ms.openlocfilehash: 4f2b4a0779d775ed5be5bfa28831cccc68c33caf
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86529019"
---
# <a name="use-the-azure-portal-to-view-service-health-notifications"></a>Use Azure Portal para ver las notificaciones de mantenimiento del servicio

Azure publica las notificaciones de mantenimiento del servicio, que contienen información acerca de los recursos en la suscripción. Estas notificaciones son una subclase de los eventos del registro de actividad y también se pueden encontrar en el registro de actividad. En función de la clase, las notificaciones de mantenimiento del servicio pueden ser meramente informativas o pueden requerir acciones.

Hay varias clases de las notificaciones de mantenimiento del servicio:  

- **Acción requerida:** Azure puede haber observado algo inusual en su cuenta y le ayuda a remediarlo. Azure le envía una notificación en la que se detallan las acciones que debe realizar o se proporciona información sobre cómo ponerse en contacto con los equipos de ingeniería o soporte técnico de Azure.  
- **Incidente:** un evento que tiene repercusiones en el servicio está afectando a uno o varios recursos de la suscripción.  
- **Mantenimiento:** una actividad de mantenimiento planeado que puede afectar a uno o varios de los recursos de la suscripción.  
- **Información:** posibles optimizaciones que puedan ayudarle a mejorar el uso de los recursos. 
- **Seguridad:** información urgente relacionada con la seguridad de las soluciones que se ejecutan en Azure.

Cada notificación del estado de un servicio incluye datos acerca del ámbito y el impacto en los recursos. Los detalles incluyen:

Nombre de propiedad | Descripción
-------- | -----------
canales nueva | Uno de los valores siguientes: **Administrador** u **Operación**.
correlationId | Normalmente, un GUID en formato de cadena. Los eventos que pertenecen a la misma acción generalmente suelen compartir el mismo valor de correlationId.
eventDataId | Identificador único de un evento.
eventName | Título del evento.
level | Nivel de un evento
resourceProviderName | Nombre del proveedor de recursos del recurso afectado.
resourceType| Tipo del recurso afectado.
subStatus | Normalmente el código de estado HTTP de la llamada REST correspondiente, pero también puede incluir otras cadenas que describen un subestado. Por ejemplo: Correcto (código de estado HTTP: 200), Creado (código de estado HTTP: 201), Aceptado (código de estado HTTP: 202), Sin contenido (código de estado HTTP: 204), Solicitud incorrecta (código de estado HTTP: 400), No encontrado (código de estado HTTP): 404), Conflicto (código de estado HTTP: 409), Error interno del servidor (código de estado HTTP: 500), Servicio no disponible (código de estado HTTP: 503), Tiempo de espera agotado para la puerta de enlace (código de estado HTTP: 504).
eventTimestamp | Marca de tiempo de cuándo generó el evento el servicio de Azure que procesó la solicitud correspondiente al evento.
submissionTimestamp | Marca de tiempo de cuándo el evento empezó a estar disponible para las consultas.
subscriptionId | Suscripción de Azure en la que se registró este evento.
status | Cadena que describe el estado de la operación. Entre los valores habituales, se incluyen: **Started**, **In Progress**, **Succeeded**, **Failed**, **Active** y **Resolved**.
operationName | Nombre de la operación.
category | Esta propiedad es siempre **ServiceHealth**.
resourceId | Identificador de recurso del recurso afectado.
Properties.title | El título localizado de esta comunicación. El valor predeterminado es inglés.
Properties.communication | Los detalles localizados de la comunicación con el formato HTML. El valor predeterminado es inglés.
Properties.incidentType | Uno de los valores siguientes: **ActionRequired**, **Informational**, **Incident**, **Maintenance** o **Security**.
Properties.trackingId | El incidente al que está asociado este evento. Se utiliza para correlacionar los eventos relacionados con un incidente.
Properties.impactedServices | Un blob JSON con caracteres de escape que describe los servicios y regiones a los que afecta el incidente. Una lista de servicios, cada uno de los cuales tiene un valor de **ServiceName** y una lista de regiones afectadas, cada una de los cuales tiene un valor de **RegionName**.
Properties.defaultLanguageTitle | La comunicación en inglés.
Properties.defaultLanguageContent | La comunicación en inglés en formato HTML o como texto sin formato.
Properties.stage | Los valores posibles de **Incident** y **Security** son **Active**, **Resolved** o **RCA**. Para **ActionRequired** o **Informational**, el único valor posible es **Active**. En el caso de **Maintenance** son: **Active**, **Planned**, **InProgress**, **Canceled**, **Rescheduled**, **Resolved** o **Complete**.
Properties.communicationId | La comunicación con la cual está asociado este evento.

### <a name="details-on-service-health-level-information"></a>Detalles sobre la información de estado del servicio

**Action Required** (properties.incidentType == ActionRequired)
- Información: Se necesita una acción del administrador para evitar que los servicios existentes se vean afectados.
    
**Maintenance** (properties.incidentType == Maintenance)
- Advertencia: Mantenimiento de emergencia
- Información: Mantenimiento planeado estándar

**Information** (properties.incidentType == Information)
- Información: Se necesita que el administrador evite que los servicios existentes se vean afectados.

**Security** (properties.incidentType == Security)
- Advertencia: Aviso de seguridad que afecta a los servicios existentes y puede requerir la acción del administrador.
- Información: Aviso de seguridad que afecta a los servicios existentes.

**Service Issues** (properties.incidentType == Incident)
- Error: Problemas generalizados de acceso a varios servicios en distintas regiones que afectan a numerosos grupos de clientes.
- Advertencia: Problemas de acceso a servicios concretos o en regiones concretas que afectan a un subgrupo de clientes.
- Información: Problemas que afectan a las operaciones de administración o de latencia, sin afectar a la disponibilidad del servicio.
