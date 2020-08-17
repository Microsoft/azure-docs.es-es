---
title: 'Determinación de la opción de publicación: marketplace comercial de Microsoft'
description: En este artículo se describen los criterios de idoneidad y los requisitos para publicar ofertas en Microsoft AppSource y Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: keferna
ms.author: keferna
ms.date: 07/30/2020
ms.openlocfilehash: 37647a9591c0f686e4fc3f1fd858baa46e01f7ac
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498079"
---
# <a name="determine-your-publishing-option"></a>Determinar la opción de publicación

La opción de publicación que elige para su oferta está relacionada directamente con los requisitos de idoneidad y las ventajas de GTM de Marketplace comercial. Y, aún más importante, la selección de la opción de publicación y el tipo de oferta definen cómo interactúan los usuarios con su oferta de Marketplace comercial.

Para configurar su oferta, necesita comprender los siguientes conceptos clave de Marketplace comercial: las opciones de publicación, la configuración y los tipos de oferta, y las llamadas a la acción que rigen cómo y dónde se presenta la oferta en los escaparates de Marketplace comercial.

En este artículo, aprenderá lo siguiente:

- Cómo determinar el escaparate adecuado para su solución
- Qué opciones de publicación y llamadas a la acción están disponibles en cada escaparate
- Qué tipos de ofertas están disponibles para cada opción de publicación

## <a name="commercial-marketplace-publishing-options"></a>Opciones de publicación de Marketplace comercial

En la tabla siguiente se muestran las opciones de publicación de los tipos de oferta en Microsoft AppSource y Azure Marketplace.

|   | **Lista (contacto)**  | **Lista (evaluación)**  | **Gratis** | **BYOL** | **Transacción**|
| :--------- | :----------- | :------------ | :----------- | :---------- |:---------- |
| **Máquina virtual** |  |  |  | Azure Marketplace |  Azure Marketplace |
| **Aplicaciones de Azure (varias máquinas virtuales)** |  |  | Azure Marketplace | Azure Marketplace | Azure Marketplace  |
| **Imagen de contenedor** |  |  | Azure Marketplace | Azure Marketplace |   |
| **Módulo IoT Edge** |  |  | Azure Marketplace | Azure Marketplace |   |
| **Servicios administrados** |  |  |  | Azure Marketplace |   |
| **Servicios de consultoría** | Ambos escaparates |  |  |  |   |
| **Aplicación SaaS** | Ambos escaparates | Ambos escaparates | Ambos escaparates |  | Ambos escaparates* |
| **Aplicación de Microsoft 365** | AppSource | AppSource |  |  | AppSource**  |
| **Complemento de Dynamics 365** |  AppSource | AppSource |  |  |   |
| **PowerApps** | AppSource |AppSource  |  |  |   |

&#42;Las ofertas de transacción de aplicación SaaS en Microsoft AppSource actualmente son con tarjeta de crédito.

&#42;&#42; Las ofertas de Microsoft 365 se instalan gratis y se pueden monetizar a través de una oferta de SaaS como servicio de licencia. Para más información, consulte el artículo sobre cómo [monetizar el complemento de Office 365 a través de Marketplace comercial de Microsoft](/office/dev/store/monetize-addins-through-microsoft-commercial-marketplace).

## <a name="choose-a-call-to-action"></a>Elegir una llamada a la acción

Las opciones de publicación disponibles ofrecen un compromiso diferenciado del cliente y le ofrecen acceso al uso compartido de clientes potenciales y a [ventajas de Marketplace comercial](https://docs.microsoft.com/azure/marketplace/gtm-your-marketplace-benefits). Tenga en cuenta las llamadas a la acción que se corresponden con la opción de publicación:

| **Opción de publicación**    | **Descripción**  |
| :------------------- | :-------------------|
| **Lista** | Lista sencilla de la aplicación o servicio que permite a los usuarios de Marketplace comercial solicitarle que se conecte con el cliente a través de la llamada a la acción **Ponerse en contacto conmigo**. |
| **Versión de prueba** | Use Marketplace comercial para mejorar la detectabilidad y automatizar el aprovisionamiento de la experiencia de evaluación de la solución, permitiendo a los usuarios potenciales usar su experiencia dentro de la aplicación de Microsoft, SaaS o IaaS sin ningún costo extra durante un tiempo limitado antes de comprarla. Las llamadas a la acción que se usan para la opción de publicación de prueba son: **Evaluación gratuita** o **Versión de prueba**. |
| **BYOL** | Use Marketplace comercial para mejorar la detectabilidad y automatizar el aprovisionamiento de su solución, así como para completar la transacción financiera por separado. Los tipos de ofertas BYOL son ideales para las migraciones desde el entorno local a la nube. La llamada a la acción es **Obténgalo ahora**.
| **Transacción** | Las ofertas de Transact se venden a través de Marketplace comercial. Microsoft es responsable de la facturación y el cobro. La llamada a la acción es **Obténgalo ahora**.|

> [!Note]
> Cuando se usa la opción de publicación de transacciones, es importante comprender las consideraciones sobre precios, facturación y pago antes de seleccionar un tipo de oferta y crear la oferta. Para más información, revise el artículo sobre las [funcionalidades de transacción de Marketplace comercial](./marketplace-commercial-transaction-capabilities-and-considerations.md).

## <a name="selecting-a-storefront"></a>Selección de un escaparate

Cada escaparate cumple con los requisitos únicos del cliente y apunta a audiencias específicas. El tipo de oferta, las funcionalidades de transacción y la categoría determinarán dónde se publicará la oferta. Las categorías y subcategorías se asignan a cada escaparate en función de la audiencia de destino:

**Microsoft AppSource** se dirige a los usuarios empresariales que buscan soluciones de línea de negocio o del sector y servicios de consultoría para Dynamics 365, Microsoft 365 y Power Platform.

**Azure Marketplace** se dirige a los profesionales de TI y a los desarrolladores que buscan soluciones creadas para o en Azure, así como servicios de consultoría que aceleran su uso de Azure.

Seleccione la categoría y la subcategoría que mejor se adapte al público de destino. Por ejemplo, un firewall de aplicaciones web debe publicarse en Azure Marketplace, en la categoría de seguridad, ya que el público previsto son profesionales de TI. En su lugar, se debe publicar una aplicación de administración de contratos en AppSource en la categoría de ventas, ya que el público previsto son usuarios empresariales. La selección de una categoría o subcategoría incorrecta puede hacer que la oferta se publique en el escaparate equivocado.

### <a name="publishing-to-both-storefronts-saas-offers-only"></a>Publicación en ambos escaparates (solo ofertas de SaaS)

Las ofertas de SaaS pueden publicarse en Azure Marketplace o AppSource. Si su oferta de SaaS está diseñada *tanto* para una audiencia técnica (Azure Marketplace) como para una audiencia empresarial (AppSource), seleccione una categoría o subcategoría aplicable a cada escaparate. Las ofertas publicadas en ambos escaparates deben tener una propuesta de valor que se extienda a los profesionales de TI *y* a los usuarios empresariales.

> [!IMPORTANT]
> Las ofertas de SaaS con facturación de uso medido están disponibles a través de Azure Marketplace y Azure Portal. Las ofertas de SaaS con planes privados exclusivamente están disponibles en Azure Portal.

| Facturación de uso medido | Plan público | Plan privado | Disponible en: |
|---|---|---|---|
| Sí             | Sí         | No           | Azure Marketplace y Azure Portal |
| Sí             | Sí         | Sí          | Azure Marketplace y Azure Portal* |
| Sí             | No          | Sí          | Azure Portal solamente |
| No              | No          | Sí          | Azure Portal solamente |

&#42; El plan privado de la oferta solo estará disponible en Azure Portal.

Por ejemplo, una oferta con facturación de uso medido y un plan privado solo (ningún plan público) lo comprarán los clientes en Azure Portal. Obtenga más información sobre [Ofertas privadas en el marketplace comercial de Microsoft](private-offers.md).

### <a name="categories"></a>Categorías

Las categorías y subcategorías se asignan a cada escaparate en función de la audiencia de destino. Seleccione las categorías y subcategorías que mejor se adapten a su oferta y a la audiencia prevista. Puede seleccionar:

- Al menos una categoría, hasta un máximo de dos. Tiene la opción de elegir una categoría principal y una secundaria.
- Hasta dos subcategorías para cada categoría principal o secundaria. Si no selecciona ninguna subcategoría, la oferta se podrá detectar en la categoría seleccionada.

[!INCLUDE [categories and subcategories](./includes/categories.md)]

## <a name="next-steps"></a>Pasos siguientes

- Una vez que decida una opción de publicación, estará listo para [seleccionar el tipo de oferta](./publisher-guide-by-offer-type.md) que se usará para presentar la oferta.
- Revise los requisitos de idoneidad de las opciones de publicación por tipo de oferta para finalizar la selección y la configuración de la oferta.
- Revise los patrones de publicación por escaparate para ver ejemplos de cómo la solución se asigna a un tipo de oferta y configuración.
