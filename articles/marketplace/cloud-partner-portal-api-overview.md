---
title: 'Referencia de API de Cloud Partner Portal: Marketplace comercial de Microsoft'
description: Descripción, requisitos previos para el uso y lista de las operaciones de API del Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 07/14/2020
ms.openlocfilehash: bfb6e9cb510f5fe887f108dfdea5932406aafe0d
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87292929"
---
# <a name="cloud-partner-portal-api-reference"></a>Referencia de API del Cloud Partner Portal

> [!NOTE]
> Las API de Cloud Partner Portal se integran con el Centro de partners y seguirán funcionando en este. La transición presenta pequeños cambios. Revise los [cambios de las API de Cloud Partner Portal](#changes-to-cpp-apis-after-the-migration-to-partner-center) enumerados en este documento para asegurarse de que el código sigue funcionando después de la transición al Centro de partners. Las API de CPP solo deben usarse para los productos existentes que ya estaban integrados antes de la transición al Centro de partners. Los nuevos productos deben usar las API de envío del Centro de partners.

Las API de REST de Cloud Partner Portal permiten la recuperación y la manipulación mediante programación de las cargas de trabajo, ofertas y perfiles del publicador. Las API de usan el control de acceso basado en roles (RBAC) para exigir los permisos correctos en el tiempo de procesamiento.

Esta referencia proporciona los detalles técnicos de las API de REST de Cloud Partner Portal. Los ejemplos de carga en este documento son solo para referencia y están sujetos a cambios a medida que se agregan nuevas funcionalidades.

## <a name="prerequisites-and-considerations"></a>Requisitos previos y consideraciones

Antes de usar las API, debe revisar:

- El artículo [API Prerequisites](./cloud-partner-portal-api-prerequisites.md) (Requisitos previos de API) para obtener información sobre cómo agregar una entidad de servicio a su cuenta y obtener el token un acceso de Azure Active Directory (Azure AD) para la autenticación.
- Las dos estrategias de [control de simultaneidad](./cloud-partner-portal-api-concurrency-control.md) disponibles para llamar a estas API.
- [Consideraciones](./cloud-partner-portal-api-considerations.md) de API adicionales, como control de versiones y control de errores.

## <a name="changes-to-cpp-apis-after-the-migration-to-partner-center"></a>Cambios en las API de Cloud Partner Portal después de la migración al Centro de partners

| **API** | **Descripción del cambio** | **Impacto** |
| ------- | ---------------------- | ---------- |
| POST Publish, GoLive, Cancel | En el caso de las ofertas migradas, el encabezado de respuesta tendrá un formato diferente, pero continuará funcionando de la misma manera, lo que denota una ruta de acceso relativa para recuperar el estado de la operación. | Al enviar cualquiera de las solicitudes POST correspondientes para una oferta, el encabezado de ubicación tendrá alguno de los dos formatos disponibles, según el estado de migración de la oferta:<ul><li>Ofertas no migradas<br>`/api/operations/{PublisherId}${offerId}$2$preview?api-version=2017-10-31`</li><li>Ofertas migradas<br>`/api/publishers/{PublisherId}/offers/{offereId}/operations/408a4835-0000-1000-0000-000000000000?api-version=2017-10-31`</li> |
| GET Operation | En el caso de los tipos de oferta que anteriormente admitían el campo "Dirección de correo electrónico para notificaciones" en la respuesta, este campo quedará en desuso y ya no se devolverá para las ofertas migradas. | En el caso de las ofertas migradas, ya no se enviarán notificaciones a la lista de correos electrónicos especificados en las solicitudes. En su lugar, el servicio de API se alineará con el proceso de correo electrónico de notificaciones del Centro de partners para enviar correos electrónicos. En concreto, las notificaciones se enviarán a la dirección de correo electrónico establecida en la sección de información de contacto del vendedor de la configuración de la cuenta en el Centro de partners, para notificarle el progreso de la operación.<br><br>Revise la dirección de correo electrónico establecida en la sección de información de contacto del vendedor de la [configuración de la cuenta](https://partner.microsoft.com/dashboard/account/management) en el Centro de partners para asegurarse de que se proporciona el correo electrónico correcto para las notificaciones.  |

## <a name="common-tasks"></a>Tareas comunes

Esta referencia ofrece información sobre las API adecuadas para realizar las siguientes tareas comunes.

### <a name="offers"></a>Ofertas

- [Recuperar todas las ofertas](./cloud-partner-portal-api-retrieve-offers.md)
- [Recuperar una oferta específica](./cloud-partner-portal-api-retrieve-specific-offer.md)
- [Recuperación del estado de la oferta](./cloud-partner-portal-api-retrieve-offer-status.md)
- [Creación de una oferta](./cloud-partner-portal-api-creating-offer.md)
- [Publicación de una oferta](./cloud-partner-portal-api-publish-offer.md)

### <a name="operations"></a>Operaciones

- [Recuperación de operaciones](./cloud-partner-portal-api-retrieve-operations.md)
- [Cancelación de operaciones](./cloud-partner-portal-api-cancel-operations.md)

### <a name="publish-an-app"></a>Publicar una aplicación

- [Publicación](./cloud-partner-portal-api-go-live.md)

### <a name="other-tasks"></a>Otras tareas

- [Pricing for virtual machine offers](./cloud-partner-portal-api-setting-price.md) (Precios de ofertas de máquinas virtuales)

### <a name="troubleshooting"></a>Solución de problemas

- [Solución de errores de autenticación](./cloud-partner-portal-api-troubleshooting-authentication-errors.md)
