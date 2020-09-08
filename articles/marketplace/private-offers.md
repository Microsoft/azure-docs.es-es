---
title: Ofertas privadas en el marketplace comercial de Microsoft
description: Ofertas privadas en el marketplace comercial de Microsoft para publicadores de aplicaciones y servicios.
ms.subservice: partnercenter-marketplace-publisher
ms.service: marketplace
ms.topic: article
author: vikrambmsft
ms.author: vikramb
ms.date: 07/01/2020
ms.openlocfilehash: 2808f0d3b99257d52b2d400424f256f980005863
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88607190"
---
# <a name="private-offers-in-the-microsoft-commercial-marketplace"></a>Ofertas privadas en el marketplace comercial de Microsoft

Las ofertas privadas de [Microsoft Azure Marketplace](https://azuremarketplace.microsoft.com/) permiten que los publicadores creen planes que solo son visibles para clientes específicos. En este artículo se tratan las opciones y ventajas de las ofertas privadas.

## <a name="unlock-enterprise-deals-with-private-offers"></a>Desbloqueo de ofertas empresariales con ofertas privadas

Los clientes empresariales usan cada vez más los marketplaces en línea para buscar, probar y comprar soluciones en la nube. Con las ofertas privadas, ahora los editores pueden usar Marketplace para compartir de manera privada soluciones personalizadas con clientes específicos que tengan las funcionalidades que requieren las empresas:

- Los *precios negociados* permiten que los editores extiendan descuentos y precios fuera de la lista más allá de las ofertas disponibles a todo público.
- Los *términos y condiciones privados* permiten que los editores adapten los términos y condiciones a un cliente específico.
- Las *configuraciones especializadas* permiten que los editores adapten su oferta de Virtual Machines, Aplicaciones de Azure y Aplicaciones de SaaS a las necesidades de un cliente individual. Esta opción también permite que los editores brinden acceso de versión preliminar a características de productos nuevos, antes de lanzarlos de manera más amplia a todos los clientes.

Las ofertas privadas permiten que los editores aprovechen la disponibilidad global y a escala de un marketplace público, con la flexibilidad y el control que se necesitan para negociar y brindar ofertas y configuraciones personalizadas. En conjunto, estas características son el primer paso para que las empresas adopten con firmeza los marketplaces de la nube. Ahora las empresas pueden comprar y vender como lo esperan y exigen.

Las ofertas privadas ya están disponibles para las ofertas de Virtual Machines, Aplicación de Azure (implementadas como plantillas de solución o aplicaciones administradas) y Aplicaciones de SaaS.

<!--- Like public offers, private offers can be created and managed via the [Cloud Partner Portal](./cloud-partner-portal-orig/cloud-partner-portal-azure-private-skus.md). Customers can be granted or revoked access to private offers in minutes.
--->

## <a name="creating-private-offers-using-plans"></a>Creación de ofertas privadas mediante planes

Para las *ofertas nuevas o existentes con planes públicos*, los editores pueden crear fácilmente variaciones privadas nuevas si crean planes nuevos (anteriormente denominados "SKU") y los marcan como privados. Cada oferta puede tener hasta 45 planes privados.

<!--- [Private SKUs](./cloud-partner-portal-orig/cloud-partner-portal-azure-private-skus.md) --->

Los planes privados son componentes de una oferta y solo clientes específicos pueden verlos y comprarlos. Los planes privados pueden volver a usar las imágenes base o metadatos de la oferta ya publicados para un plan público. Esta opción permite que los editores creen múltiples variaciones privadas de una oferta pública sin tener que publicar varias versiones de la misma imagen base y metadatos de oferta. Solo en el caso de las ofertas de Virtual Machines y Azure, cuando un plan privado comparte una imagen base con un plan público, los cambios que se hacen en la imagen base de la oferta se propagan a todos los planes, tanto públicos como privados, que usan esa imagen base.

En el caso de las *ofertas nuevas que solo incluyen planes privados*, los editores pueden crear sus ofertas como cualquier otra oferta y, luego, marcar los planes como privados. Los clientes que no estén asociados con la oferta no podrán detectar ni acceder a través del marketplace comercial de Microsoft o [Azure Portal](https://azure.microsoft.com/features/azure-portal/) a las ofertas que solo tienen planes privados.

>[!NOTE]
>Una oferta que solo contenga planes privados no será visible en el catálogo público de Azure Marketplace ni AppSource.

## <a name="targeting-customers-with-private-offers"></a>Selección de clientes específicos para las ofertas privadas

Tanto para las ofertas privadas nuevas como existentes, los editores pueden seleccionar clientes específicos mediante el uso de identificadores de suscripción. Los editores que usan una oferta de Virtual Machines o Aplicación de Azure pueden restringir la disponibilidad de un plan privado a un identificador de suscripción individual de Azure o cargar un archivo .csv con hasta 20 000 identificadores de suscripción de Azure. Cuando se usa una oferta privada de Aplicación de SaaS, los anunciantes pueden asociar un identificador de inquilino para restringir la disponibilidad de un plan privado, ya sea con el enfoque manual o mediante la carga del archivo .csv.

Una vez que se certifica o publica una oferta, los clientes se pueden actualizar o quitar del plan en cuestión de minutos mediante la característica de sincronización de suscripciones privadas. Esta funcionalidad permite que los editores actualicen rápida y sencillamente la lista de clientes a los que se presenta el plan privado sin que sea necesario volver a certificar ni a publicar la oferta.

## <a name="deploying-private-offers"></a>Implementación de las ofertas privadas

Las ofertas privadas solo se detectan mediante [Azure Portal](https://azure.microsoft.com/features/azure-portal/) y no aparecen en [Microsoft AppSource](https://appsource.microsoft.com/) ni en [Azure Marketplace](https://azuremarketplace.microsoft.com). Para más información sobre la publicación en las distintas tiendas en línea de marketplace comercial, consulte [Determinación de la opción de publicación](./determine-your-listing-type.md).

Una vez que un cliente inicia sesión en Azure Portal, puede seleccionar el elemento de navegación Marketplace para acceder a sus ofertas privadas. Las ofertas privadas también aparecerán en los resultados de la búsqueda y se pueden implementar a través de la línea de comandos y plantillas de Azure Resource Manager como cualquier otra oferta.

![[Ofertas privadas]](./media/marketplace-publishers-guide/private-offer.png)

Las ofertas privadas también aparecerán en los resultados de la búsqueda. Simplemente busque el distintivo que indica **Privada**.

>[!Note]
>Las ofertas privadas no son compatibles con las suscripciones que se establecen a través de un revendedor del programa Proveedor de soluciones en la nube (CSP).


<!---
## Next steps

To start using private offers, follow the steps in the [Private SKUs and Plans](./cloud-partner-portal-orig/cloud-partner-portal-azure-private-skus.md) guide.
--->
