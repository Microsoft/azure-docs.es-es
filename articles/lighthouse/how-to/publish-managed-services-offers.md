---
title: Publicación de una oferta de servicio administrado en Azure Marketplace
description: Aprenda a publicar una oferta de servicio administrado que incorpore clientes a Azure Lighthouse.
ms.date: 08/18/2020
ms.topic: how-to
ms.openlocfilehash: 11fb0073bcc3ba4437f78de8b263d165f0069db8
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88589731"
---
# <a name="publish-a-managed-service-offer-to-azure-marketplace"></a>Publicación de una oferta de servicio administrado en Azure Marketplace

En este artículo, aprenderá a publicar una oferta de servicio administrado pública o privada en [Azure Marketplace](https://azuremarketplace.microsoft.com) con el programa [Marketplace comercial](../../marketplace/partner-center-portal/commercial-marketplace-overview.md) del Centro de partners. Los clientes que compran la oferta delegarán suscripciones o grupos de recursos, lo que le permitirá administrarlas desde [Azure Lighthouse](../overview.md).

## <a name="publishing-requirements"></a>Requisitos de publicación

Debe tener una [cuenta en el Centro de partners](../../marketplace/partner-center-portal/create-account.md) valida para crear y publicar ofertas. Si aún no tiene una cuenta, el [proceso de registro](https://aka.ms/joinmarketplace) le guiará por los pasos necesarios para crear una cuenta en el Centro de partners e inscribirse en el programa comercial de Marketplace.

Según los [Requisitos de certificación de la oferta de servicio administrado](/legal/marketplace/certification-policies#7004-business-requirements), debe tener un [nivel de competencia de plataforma en la nube Silver o Gold](/partner-center/learn-about-competencies) o ser un [MSP experto de Azure](https://partner.microsoft.com/membership/azure-expert-msp) para publicar una oferta de servicio administrado.

El identificador de Microsoft Partner Network (MPN) [se asociará automáticamente](../../cost-management-billing/manage/link-partner-id.md) con las ofertas que publique para realizar un seguimiento del impacto en las involucraciones de clientes.

Si no quiere publicar una oferta en Azure Marketplace o no cumple todos los requisitos, puede incorporar clientes manualmente mediante plantillas de Azure Resource Manager. Para obtener más información, consulte [Incorporación de un cliente a Azure Lighthouse](onboard-customer.md).

La tabla siguiente puede ayudar a determinar si se deben incorporar clientes mediante la publicación de una oferta de servicio administrada o usando plantillas de Azure Resource Manager.

|**Consideración**  |**Oferta de servicio administrado**  |**Plantillas de ARM**  |
|---------|---------|---------|
|Se necesita una [cuenta del Centro de partners](../../marketplace/partner-center-portal/create-account.md).   |Sí         |No        |
|Se necesita un [nivel de competencia de la plataforma de nube Silver o Gold](/partner-center/learn-about-competencies) o un [MSP experto de Azure](https://partner.microsoft.com/membership/azure-expert-msp).      |Sí         |No         |
|Disponible para nuevos clientes mediante Azure Marketplace     |Sí     |No       |
|Se puede limitar la oferta a clientes específicos     |Sí (solo con ofertas privadas, que no son compatibles con las suscripciones que se establecen a través de un revendedor del programa Proveedor de soluciones en la nube [CSP]).         |Sí         |
|Se requiere la aceptación del cliente en Azure Portal     |Sí     |No   |
|Se puede usar automatización para incorporar varias suscripciones, grupos de recursos o clientes |No     |Sí    |
|Acceso inmediato a nuevos roles integrados y características de Azure Lighthouse     |No siempre (disponible generalmente después de un pequeño retraso)         |Sí         |

## <a name="create-your-offer"></a>Creación de la oferta

Para instrucciones detalladas sobre cómo crear la oferta, incluida toda la información y los recursos que deberá proporcionar, consulte [Creación de una oferta de servicio administrado](../../marketplace/partner-center-portal/create-new-managed-service-offer.md).

Para obtener información sobre el proceso general de publicación, consulte [Guía de publicación de Azure Marketplace y AppSource](../../marketplace/marketplace-publishers-guide.md). También debe revisar [las directivas de certificación de Marketplace comercial](/legal/marketplace/certification-policies), en especial la sección [Servicios administrados](/legal/marketplace/certification-policies#700-managed-services).

Una vez que un cliente agregue su oferta, podrá delegar uno o varios grupos de recursos o suscripciones, que, posteriormente, se [incorporarán a Azure Lighthouse](#the-customer-onboarding-process).

> [!IMPORTANT]
> Cada plan de una oferta de servicio administrado incluye la sección **Detalles del manifiesto**, donde se definen las entidades de Azure Active Directory (Azure AD) del inquilino que tendrán acceso a las suscripciones o los grupos de recursos delegados para los clientes que adquieran el plan. Es importante tener en cuenta que cualquier grupo (o entidad de servicio o usuario) que incluya tendrá los mismos permisos para todos los clientes que compren el plan. Para asignar grupos diferentes para que trabajen con cada cliente, puede publicar un [plan privado](../../marketplace/private-offers.md) independiente que sea exclusivo para cada cliente. Tenga en cuenta que los planes privados no son compatibles con las suscripciones que se establecen a través de un revendedor del programa Proveedor de soluciones en la nube (CSP).

## <a name="publish-your-offer"></a>Publicación de la oferta

Cuando haya completado todas las secciones, el siguiente paso es publicar la oferta en Azure Marketplace. Seleccione el botón **Publicar** para iniciar el proceso de publicación de la oferta. [Aquí](../../marketplace/partner-center-portal/create-new-managed-service-offer.md#publish) puede encontrar más información sobre este proceso. 

Puede [publicar una versión actualizada de la oferta](../..//marketplace/partner-center-portal/update-existing-offer.md) en cualquier momento. Por ejemplo, puede querer agregar una nueva definición de roles a una oferta publicada previamente. Al hacerlo, los clientes que ya hayan agregado la oferta verán un icono en la página [**Proveedores de servicios**](view-manage-service-providers.md) de Azure Portal que les informa que hay disponible una actualización. Cada cliente podrá [revisar los cambios](view-manage-service-providers.md#update-service-provider-offers) y decidir si desea actualizar a la nueva versión. 

## <a name="the-customer-onboarding-process"></a>Proceso de incorporación del cliente

Una vez que un cliente agregue su oferta, podrá [delegar uno o varios grupos de recursos o suscripciones concretos](view-manage-service-providers.md#delegate-resources) que, posteriormente, se incorporarán a Azure Lighthouse. Si un cliente ha aceptado una oferta pero aún no ha delegado los recursos, verá una nota en la parte superior de la sección **Ofertas de proveedor** de la página [**Service providers**](view-manage-service-providers.md) (Proveedores de servicio) en Azure Portal.

> [!IMPORTANT]
> La delegación debe realizarse desde una cuenta que no sea de invitado en el inquilino del cliente que tenga el [rol integrado Propietario](../../role-based-access-control/built-in-roles.md#owner) para la suscripción que se va a incorporar (o que contenga los grupos de recursos que se incorporan). Para ver todos los usuarios que puedan delegar la suscripción, cualquiera de los usuarios del inquilino del cliente puede seleccionar la suscripción en Azure Portal, abrir **Control de acceso (IAM)** y [ver todos los usuarios con el rol Propietario](../../role-based-access-control/role-assignments-list-portal.md#list-owners-of-a-subscription).

Una vez que el cliente delega una suscripción (o uno o varios grupos de recursos dentro de una suscripción), el proveedor de recursos **Microsoft.ManagedServices** se registrará para esa suscripción y los usuarios del inquilino podrán acceder a los recursos delegados según las autorizaciones de la oferta.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre [Marketplace comercial](../../marketplace/partner-center-portal/commercial-marketplace-overview.md).
- Más información sobre las [experiencias de administración entre inquilinos](../concepts/cross-tenant-management-experience.md).
- Puede [ver y administrar clientes](view-manage-customers.md) desde **Mis clientes**, en Azure Portal.
