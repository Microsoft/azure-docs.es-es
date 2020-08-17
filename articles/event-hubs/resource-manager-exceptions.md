---
title: 'Azure Event Hubs: excepciones de Resource Manager | Microsoft Docs'
description: Lista de excepciones de Azure Event Hubs que se exhiben en Azure Resource Manager y acciones sugeridas.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: aa1a3ca647bbf9e6590446549455a9853411fd7d
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87281045"
---
# <a name="azure-event-hubs---resource-manager-exceptions"></a>Azure Event Hubs: excepciones de Resource Manager
En este artículo se enumeran las excepciones generadas al interactuar con Azure Event Hubs mediante Azure Resource Manager por medio de plantillas o llamadas directas.

> [!IMPORTANT]
> Este documento se actualiza con frecuencia. Compruebe si hay actualizaciones.

En las secciones siguientes se proporcionan varias excepciones o errores que se exhiben mediante Azure Resource Manager.

## <a name="error-code-conflict"></a>Código de error: Conflicto

| Código de error | Subcódigo de error | Mensaje de error | Descripción | Recomendación |
| ---------- | ------------- | ------------- | ----------- | -------------- |
| Conflicto | 40300 | Se ha alcanzado o superado el número máximo de recursos de tipo EventHub. Real: #, máximo permitido: # | El espacio de nombres ha alcanzado su [cuota](event-hubs-quotas.md) del número de centros de eventos que puede contener. | Elimine los centros de eventos no usados o extraños del espacio de nombres o considere la posibilidad de actualizar a un [clúster dedicado](event-hubs-dedicated-overview.md). |
| Conflicto | None | No se puede eliminar la configuración de recuperación ante desastres (DR) porque la replicación está en curso. Conmute por error o interrumpa el emparejamiento antes de intentar eliminar la configuración de recuperación ante desastres. | La [replicación de GeoDR](event-hubs-geo-dr.md) está en curso, por lo que en este momento no se puede eliminar la configuración. | Para desbloquear la eliminación de la configuración, espere a que la replicación se haya completado, desencadene una conmutación por error o interrumpa el emparejamiento de GeoDR. |
| Conflicto | None | No se pudo actualizar el espacio de nombres y se generó un conflicto en el back-end. | Actualmente se está realizando otra operación en este espacio de nombres. | Espere hasta que se complete la operación actual y, luego, vuelva a intentarlo. |

## <a name="error-code-429"></a>Código de error: 429

| Código de error | Subcódigo de error | Mensaje de error | Descripción | Recomendación |
| ---------- | ------------- | ------------- | ----------- | -------------- |
| 429 | None | Aprovisionamiento del espacio de nombres en transición | Actualmente se está realizando otra operación en este espacio de nombres. | Espere hasta que se complete la operación actual y, luego, vuelva a intentarlo. |
| 429 | None | Operación de recuperación ante desastres en curso. | Actualmente se está realizando una operación [GeoDR](event-hubs-geo-dr.md) en este espacio de nombres o emparejamiento. | Espere hasta que se complete la operación GeoDR actual y, luego, vuelva a intentarlo. |

## <a name="error-code-badrequest"></a>Código de error: BadRequest

| Código de error | Subcódigo de error | Mensaje de error | Descripción | Recomendación |
| ---------- | ------------- | ------------- | ----------- | -------------- |
| BadRequest | 40000 | No se puede cambiar PartitionCount para un centro de eventos. | El nivel básico o estándar de Azure Event Hubs no admite el cambio de particiones. | Cree un centro de eventos con el número deseado de particiones en el espacio de nombres de nivel básico o estándar. La escalabilidad horizontal de la partición es compatible con [clústeres dedicados](event-hubs-dedicated-overview.md). |
| BadRequest | 40000 | El valor "#" de MessageRetentionInDays no es válido para el nivel básico. El valor no puede superar "1" día(s). | Los espacios de nombres de Event Hubs del nivel básico solo admiten la retención de mensajes de hasta 1 día. | Si quiere más de un día de retención de mensajes, [cree un espacio de nombres de Event Hubs estándar](event-hubs-create.md). | 
| BadRequest | None | El nombre especificado no está disponible. | Los nombres de espacios de nombres deben ser únicos, sin embargo, el nombre especificado ya se ha usado. | Si es el propietario del espacio de nombres existente con el nombre especificado, puede eliminarlo, lo que producirá la pérdida de datos. Luego, vuelva a intentarlo con el mismo nombre. Si no es seguro eliminar el espacio de nombres (o no es el propietario), elija otro nombre de espacio de nombres. |
| BadRequest | None | La suscripción especificada ha alcanzado su cuota de espacios de nombres. | Su suscripción ha alcanzado la [cuota](event-hubs-quotas.md) del número de espacios de nombres que puede contener. | Considere la posibilidad de eliminar los espacios de nombres no usados en esta suscripción, crear otra suscripción o actualizar a un [clúster dedicado](event-hubs-dedicated-overview.md). |
| BadRequest | None | No se puede actualizar un espacio de nombres secundario. | El espacio de nombres no se puede actualizar porque es el espacio de nombres secundario de un [emparejamiento de GeoDR](event-hubs-geo-dr.md). | Si es necesario, realice el cambio en el espacio de nombres principal de este emparejamiento. De lo contrario, interrumpa el emparejamiento de GeoDR para efectuar el cambio. |
| BadRequest | None | No se puede establecer el inflado automático en la SKU básica. | No se puede habilitar el inflado automático en los espacios de nombres de Event Hubs de nivel básico. | Para [habilitar el inflado automático](event-hubs-auto-inflate.md) en un espacio de nombres, asegúrese de que sea de nivel estándar. |
| BadRequest | None | No hay suficiente capacidad para crear el espacio de nombres. Póngase en contacto con el administrador de Event Hubs. | La región seleccionada está al máximo de su capacidad y no se pueden crear más espacios de nombres. | Seleccione otra región para hospedar el espacio de nombres. |
| BadRequest | None | No se puede realizar la operación en el tipo de entidad "ConsumerGroup" porque el espacio de nombres "nombre del espacio de nombres" usa el nivel "Básico".  | Los espacios de nombres de Event Hubs de nivel básico tienen una [cuota](event-hubs-quotas.md#event-hubs-basic-and-standard---quotas-and-limits) de un grupo de consumidores (valor predeterminado). No se admite la creación de más grupos de consumidores. | Siga usando el grupo de consumidores predeterminado ($Default) o, si necesita más, considere la posibilidad de usar en su lugar un espacio de nombres de Event Hubs de nivel estándar. | 
| BadRequest | None | El espacio de nombres "nombre del espacio de nombres" no existe. | No se pudo encontrar el espacio de nombres proporcionado. | Compruebe que el nombre del espacio de nombres sea correcto y se encuentre en su suscripción. Si no, [cree un espacio de nombres de Event Hubs](event-hubs-create.md). | 
| BadRequest | None | La propiedad location del recurso no coincide con el espacio de nombres contenedor. | No se pudo crear un centro de eventos en una región específica porque no coincidía con la región del espacio de nombres. | Intente crear el centro de eventos en la misma región que el espacio de nombres. | 

## <a name="error-code-internal-server-error"></a>Código de error: Error interno del servidor

| Código de error | Subcódigo de error | Mensaje de error | Descripción | Recomendación |
| ---------- | ------------- | ------------- | ----------- | -------------- |
| Internal Server Error | None | Error interno del servidor. | Error interno del servicio Event Hubs. | Vuelva a intentar la operación con error. Si la operación sigue produciendo errores, póngase en contacto con el servicio de soporte técnico. |