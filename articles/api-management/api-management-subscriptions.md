---
title: Suscripciones en Azure API Management | Microsoft Docs
description: Información acerca del concepto de las suscripciones en Azure API Management.
services: api-management
documentationcenter: ''
author: miaojiang
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/14/2018
ms.author: apimpm
ms.openlocfilehash: b4a3e8c71757417240af2719c317bbbd49cef9fc
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87475777"
---
# <a name="subscriptions-in-azure-api-management"></a>Suscripciones en Azure API Management

Las suscripciones son un concepto importante en Azure API Management. Son la forma más habitual para los consumidores de obtener acceso a las API publicadas mediante una instancia de API Management. En este artículo se proporciona información general del concepto.

## <a name="what-are-subscriptions"></a>¿Qué son las suscripciones?

Al publicar las API mediante API Management, es sencillo y frecuente proteger el acceso a ellas mediante claves de suscripción. Los desarrolladores que necesiten usar las API publicadas deben incluir una clave de suscripción válida en las solicitudes HTTP al realizar llamadas a esas API. De lo contrario, la puerta de enlace de API Management rechaza las llamadas inmediatamente. No se reenvían a los servicios back-end.

Para obtener una clave de suscripción para acceder a las API se necesita una suscripción. Una suscripción es básicamente un contenedor con nombre para un par de claves de suscripción. Los desarrolladores que necesiten usar las API publicadas pueden obtener suscripciones. Y no necesitan aprobación de los publicadores de API. Los publicadores de API también pueden crear suscripciones directamente para los consumidores de API.

> [!TIP]
> API Management también admite otros mecanismos para proteger el acceso a las API, por ejemplo:
> - [OAuth2.0](api-management-howto-protect-backend-with-aad.md)
> - [Certificados de cliente](api-management-howto-mutual-certificates-for-clients.md)
> - [Lista de direcciones IP permitidas](./api-management-access-restriction-policies.md#RestrictCallerIPs)

## <a name="scope-of-subscriptions"></a>Ámbito de las suscripciones

Las suscripciones se pueden asociar a diversos ámbitos: producto, todas las API o una API individual.

### <a name="subscriptions-for-a-product"></a>Suscripciones para un producto

Tradicionalmente, las suscripciones en API Management se han asociado siempre a un solo ámbito del [producto de API](api-management-terminology.md). Los desarrolladores encontraron la lista de productos en el Portal para desarrolladores. A continuación, enviarían las solicitudes de suscripción para los productos que deseaban usar. Una vez aprobada una solicitud de suscripción (automáticamente o por parte de los publicadores de API), el desarrollador puede usar las claves de esta para acceder a todas las API del producto. En este momento, en la sección del perfil de usuario del portal para desarrolladores solo aparecen las suscripciones del ámbito del producto. 

![Suscripciones de producto](./media/api-management-subscriptions/product-subscription.png)

> [!TIP]
> En determinadas situaciones, es posible que los publicadores de API deseen publicar un producto de API para el público sin necesidad de suscripciones. Pueden anular la selección de la opción **Require subscription** (Requerir suscripción) en la página **Settings** (Configuración) del producto en Azure Portal. Como resultado, se puede tener acceso a todas las API del producto sin una clave de API.

### <a name="subscriptions-for-all-apis-or-an-individual-api"></a>Suscripciones para todas las API o una API individual

Cuando introdujimos el nivel de [consumo](https://aka.ms/apimconsumptionblog) de API Management, realizamos algunos cambios para optimizar la administración de claves:
- En primer lugar, agregamos dos ámbitos más de suscripción: todas las API y una API individual. El ámbito de las suscripciones ya no se limita a un producto de API. Ahora es posible crear claves que concedan acceso a una API (o a todas las API de una instancia de API Management), sin necesidad de crear un producto y agregar las API a este primero. Además, cada instancia de API Management incluye ahora de una suscripción inmutable para todas las API. Esta suscripción facilita y agiliza la prueba y la depuración de las API de la consola de prueba.

- En segundo lugar, API Management permite ahora suscripciones **independientes**. Ya no es necesario que Suscripciones se asocie a una cuenta de desarrollador. Esta característica es útil en situaciones como, por ejemplo, cuando varios desarrolladores o equipos comparten una suscripción.

- Por último, los publicadores de API ahora pueden [crear suscripciones](api-management-howto-create-subscriptions.md) directamente en Azure Portal:

    ![Suscripciones flexibles](./media/api-management-subscriptions/flexible-subscription.png)

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre API Management:

+ Aprenda otros [conceptos](api-management-terminology.md) en API Management.
+ Siga nuestros [tutoriales](import-and-publish.md) para más información sobre API Management.
+ Consulte nuestra [página de preguntas frecuentes](api-management-faq.md) para ver las preguntas habituales.
