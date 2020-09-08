---
title: Ofertas de servicios administrados en Azure Marketplace
description: Las ofertas de servicios administrados le permiten vender ofertas de administración de recursos a los clientes en Azure Marketplace.
ms.date: 07/28/2020
ms.topic: conceptual
ms.openlocfilehash: 6c3047cd95128f689e75d9c1f5fba5a39f86291c
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88163329"
---
# <a name="managed-service-offers-in-azure-marketplace"></a>Ofertas de servicios administrados en Azure Marketplace

En este artículo se describe el tipo de oferta **Servicio administrado** de [Azure Marketplace](https://azuremarketplace.microsoft.com). Las ofertas de servicio administrado le permiten ofrecer servicios de administración de recursos a los clientes mediante [Azure Lighthouse](../overview.md). Puede hacer que estas ofertas estén disponibles para todos los clientes potenciales o solo para uno o varios clientes más específicos. Dado que los clientes facturan directamente los costos relacionados con estos servicios administrados, Microsoft no aplica ningún cargo.

## <a name="understand-managed-service-offers"></a>Información sobre las ofertas de servicio administrado

Las ofertas de servicio administrado simplifican el proceso de incorporación de clientes a Azure Lighthouse. Cuando un cliente compre una oferta en Azure Marketplace, podrá especificar los suscripciones o grupos de recursos que deben incorporarse.

Después, los usuarios de la organización podrán trabajar con esos recursos desde dentro del inquilino administrador mediante la [Administración de recursos delegados de Azure](azure-delegated-resource-management.md), en función del acceso que se haya definido al crear la oferta. Esto se realiza mediante un manifiesto que especifica los usuarios, grupos y entidades de servicio de Azure Active Directory (Azure AD) que tendrán acceso a los recursos del cliente, junto con los roles que definen su nivel de acceso. Al asignar permisos a un grupo de Azure AD, en lugar de una serie de cuentas de usuario o de aplicación individuales, puede agregar o quitar usuarios individuales cuando cambien sus requisitos de acceso.

## <a name="public-and-private-offers"></a>Ofertas públicas y privadas

Cada oferta de servicios administrados incluye uno o varios planes. Estos planes pueden ser privados o públicos.

Si quiere limitar su oferta a clientes específicos, puede publicar un plan privado. Al hacerlo, el plan solo se puede comprar para los identificadores de suscripción específicos que proporcione. Para obtener más información, consulte [Ofertas privadas](../../marketplace/private-offers.md).

> [!NOTE]
> Las ofertas privadas no son compatibles con las suscripciones que se establecen a través de un revendedor del programa Proveedor de soluciones en la nube (CSP).

Los planes públicos permiten promover los servicios para nuevos clientes. Suelen ser más adecuadas cuando solo se necesita acceso limitado al inquilino del cliente. Una vez establecida una relación con un cliente, si decide conceder acceso adicional a su organización, puede hacerlo mediante la publicación de un nuevo plan privado solo para ese cliente, o bien [mediante su incorporación para ampliar su acceso mediante plantillas de Azure Resource Manager](../how-to/onboard-customer.md).

Si es necesario, puede incluir planes públicos y privados en la misma oferta.

> [!IMPORTANT]
> Una vez publicado un plan como público, no puede cambiarlo a privado. Para controlar qué clientes pueden aceptar su oferta y delegar recursos, use un plan privado. Con un plan público, no puede restringir la disponibilidad a determinados clientes ni a un determinado número de clientes, aunque puede dejar de vender el plan por completo si decide hacerlo. Puede [quitar el acceso a una delegación](../how-to/remove-delegation.md) después de que un cliente acepte una oferta solo si incluyó una **autorización** con la **definición de rol** establecida en [Rol para eliminar la asignación de registros de servicios administrados](../../role-based-access-control/built-in-roles.md#managed-services-registration-assignment-delete-role) al publicar la oferta. También puede ponerse en contacto con el cliente y solicitarle [que quite el acceso](../how-to/view-manage-service-providers.md#add-or-remove-service-provider-offers).

## <a name="publish-managed-service-offers"></a>Publicación de ofertas de servicio administrado

Aprenda a publicar una oferta de servicios administrados, consulte [Publicación de una oferta de servicio administrado en Azure Marketplace](../how-to/publish-managed-services-offers.md).

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información acerca de la [administración de recursos delegados de Azure](azure-delegated-resource-management.md) y las [experiencias de administración entre inquilinos](cross-tenant-management-experience.md).
- [Publicación de una oferta de servicio administrado](../how-to/publish-managed-services-offers.md) en Azure Marketplace.
