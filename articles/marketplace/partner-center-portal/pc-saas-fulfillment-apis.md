---
title: API de suministro de SaaS en el marketplace comercial de Microsoft
description: Introducción a las API de suministro que le permiten integrar su ofertas de SaaS en Microsoft AppSource y Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 05/18/2020
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: d9413deb99ce938429d05f7c6348c5914e7dd2b1
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87304146"
---
# <a name="saas-fulfillment-apis-in-microsoft-commercial-marketplace"></a>API de suministro de SaaS en el marketplace comercial de Microsoft

Las API de suministro de SaaS permiten que fabricantes de software independientes (ISV) publiquen y vendan sus aplicaciones SaaS con Microsoft AppSource, Azure Marketplace y Azure Portal. Estas API permiten a las aplicaciones de ISV participar en todos los canales de comercio: directo, dirigidos por asociados (revendedor) y sobre el terreno.  La integración con estas API es un requisito para crear y publicar una oferta de SaaS que permite transacciones en el Centro de partners.

Los ISV deben implementar los siguientes flujos de API agregando a su código de servicio SaaS para mantener el mismo estado de suscripción para ISV y Microsoft:

* Flujo de la página de aterrizaje:  Microsoft notifica al anunciante que un cliente de Marketplace ha adquirido la oferta de SaaS del publicador.
* Flujo de activación:  El publicador notifica a Microsoft que se ha configurado una cuenta de SaaS recién adquirida en el lado del publicador.
* Flujo de actualización: Cambio del plan adquirido y/o el número de puestos comprados.
* Flujo de suspensión y restablecimiento: Suspender la oferta de SaaS adquirida en caso de que el método de pago del cliente ya no sea válido. La oferta suspendida se puede restablecer cuando se resuelve el problema con el método de pago.
* Flujos de webhook: Microsoft enviará una notificación al anunciante sobre los cambios de suscripción de SaaS y la cancelación desencadenada por el cliente del lado de Microsoft.

Para la cancelación de la suscripción de SaaS adquirida, la integración es opcional porque la puede realizar el cliente del lado de Microsoft.

La integración correcta con las API de cumplimiento de SaaS es fundamental para asegurarse de que

* Microsoft factura correctamente a los clientes finales que compraron la oferta de SaaS del anunciante.
* los clientes finales obtienen la experiencia de usuario correcta para comprar, configurar, usar y administrar las suscripciones de SaaS adquiridas en Marketplace.

Estas API permiten a las ofertas del publicador participar en todos los canales habilitados para comercio:

* direct
* dirigido por asociados (revendedor, CSP)
* LED de campo

En el escenario de revendedor (CSP), un proveedor de servicios de cifrado adquiere la oferta de SaaS en nombre del cliente final. Se espera que un cliente use la oferta de SaaS, pero el CSP es la entidad que hace lo siguiente:

* facturación del cliente
* cambio de los planes de suscripción/cantidad de puestos comprados
* cancelar suscripciones

No es necesario que el publicador implemente ninguno de los flujos de llamada de API de forma diferente para este escenario.

Para obtener más información acerca de CSP, consulte https://partner.microsoft.com/licensing.

>[!Warning]
>La versión actual de esta API es la versión 2, que se debe usar para todas las nuevas ofertas de SaaS. La versión 1 de la API está en desuso y se mantiene para proporcionar soporte a las ofertas existentes.

>[!Note]
>Las API de cumplimiento de SaaS solo están pensadas para ser llamadas desde un servicio back-end del publicador. No se admite la integración con las API directamente desde la página web del publicador. Solo se debe usar el flujo de autenticación de servicio a servicio.

## <a name="next-steps"></a>Pasos siguientes

Si no lo ha hecho ya, registre su aplicación de SaaS en [Azure Portal](https://ms.portal.azure.com), tal como se explica en [Registro de una aplicación SaaS](./pc-saas-registration.md).  Después, use la versión más reciente de esta interfaz para el desarrollo: [API de suministro de SaaS, versión 2](./pc-saas-fulfillment-api-v2.md).
