---
title: Administración de recursos delegados de Azure
description: La administración de recursos delegados de Azure es una parte clave de Azure Lighthouse, lo que permite a los proveedores de servicios administrar recursos delegados a escala con agilidad y precisión.
ms.date: 08/12/2020
ms.topic: conceptual
ms.openlocfilehash: 9a499ceda546b7ea5c71cd8c770f1a4b99001b08
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88163533"
---
# <a name="azure-delegated-resource-management"></a>Administración de recursos delegados de Azure

La administración de recursos delegados de Azure es uno de los componentes clave de [Azure Lighthouse](../overview.md). Con la administración de recursos delegados de Azure, los proveedores de servicios pueden simplificar las experiencias de involucración e incorporación de clientes, al mismo tiempo que administran los recursos delegados a escala con agilidad y precisión.

## <a name="what-is-azure-delegated-resource-management"></a>¿Qué es la administración de recursos delegados de Azure?

La administración de recursos delegados de Azure permite la proyección lógica de recursos de un inquilino a otro. Esto permite a los usuarios autorizados de un inquilino de Azure Active Directory (Azure AD) realizar operaciones de administración entre distintos inquilinos de Azure AD que pertenecen a sus clientes. Los proveedores de servicios pueden iniciar sesión en su propio inquilino de Azure AD y tener autorización para trabajar en suscripciones y grupos de recursos delegados del cliente. Esto les permite realizar operaciones de administración en nombre de sus clientes, sin tener que iniciar sesión en cada inquilino de cliente individual.

> [!TIP]
> La administración de recursos delegados de Azure también se puede usar [en una empresa que tenga varios inquilinos de Azure AD propios](enterprise.md) para simplificar la administración entre inquilinos.

Con la administración de recursos delegados de Azure, los usuarios autorizados pueden trabajar directamente en el contexto de una suscripción de cliente sin tener una cuenta en el inquilino del cliente ni ser un copropietario del inquilino del cliente.

La [experiencia de administración entre inquilinos](cross-tenant-management-experience.md) le permite trabajar de forma más eficaz con los servicios de administración de Azure, como Azure Policy, Azure Security Center, etc. En el registro de actividad se realiza un seguimiento de toda la actividad del proveedor de servicios, que se almacena en el inquilino del cliente (y se puede ver por usuarios en el inquilino administrador). Esto significa que los usuarios tanto en cliente administrador como en el administrado pueden identificar fácilmente al usuario asociado a los cambios.

Puede [publicar el nuevo tipo de oferta de servicio administrado en Azure Marketplace](../how-to/publish-managed-services-offers.md) para incorporar fácilmente clientes en Azure Lighthouse. Como alternativa, puede [completar el proceso de incorporación implementando plantillas de Azure Resource Manager ](../how-to/onboard-customer.md).

## <a name="how-azure-delegated-resource-management-works"></a>Funcionamiento de la administración de recursos delegados de Azure

En un nivel alto, este es el funcionamiento de la administración de recursos delegados de Azure:

1. Primero tiene que identificar el acceso (roles) que los grupos, las entidades de servicio o los usuarios necesitarán para administrar los recursos de Azure del cliente. La definición de acceso contiene el identificador de inquilino administrador junto con el acceso las identidades **principalId** del inquilino asignadas a valores [integrados **roleDefinition**](../../role-based-access-control/built-in-roles.md) (colaborador, colaborador de máquina virtual, lector, etc.)
2. Especifique este acceso e incorpore el cliente a Azure Lighthouse de una de estas dos maneras:
   - [Publicar una oferta de servicio administrado de Azure Marketplace](../how-to/publish-managed-services-offers.md) (privada o pública) que el cliente aceptará
   - [Implementar una plantilla de Azure Resource Manager en el inquilino del cliente](../how-to/onboard-customer.md) para una o varias suscripciones o grupos de recursos específicos
3. Una vez incorporado el cliente, los usuarios autorizados pueden iniciar sesión en el inquilino administrador y realizar tareas de administración en el ámbito del cliente determinado, en función del acceso que haya definido.

> [!NOTE]
> Puede administrar recursos delegados que estén ubicados en diferentes [regiones](../../availability-zones/az-overview.md#regions). Sin embargo, no se admite la delegación de suscripciones entre una [nube nacional](../../active-directory/develop/authentication-national-cloud.md) y la nube pública de Azure o entre dos nubes nacionales independientes.

## <a name="support-for-azure-delegated-resource-management"></a>Soporte técnico para la administración de recursos delegados de Azure

Si necesita ayuda en relación con la administración de recursos delegados de Azure, puede abrir una solicitud de soporte técnico en Azure Portal. En **Tipo de problema**, elija **Técnico**. Seleccione una suscripción y luego **Lighthouse** (en **Supervisión y administración**).

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre las [experiencias de administración entre inquilinos](cross-tenant-management-experience.md).
- Más información sobre las [ofertas de servicios administrados en Azure Marketplace](managed-services-offers.md).
