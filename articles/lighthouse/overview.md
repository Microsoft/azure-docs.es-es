---
title: ¿Qué es Azure Lighthouse?
description: Azure Lighthouse permite a los proveedores de servicios ofrecer servicios administrados para sus clientes con mayor automatización y eficacia a escala.
ms.date: 08/19/2020
ms.topic: overview
ms.openlocfilehash: 22bec7ec8944a11ce0cfdf51776f1f193a1aedaa
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89488820"
---
# <a name="what-is-azure-lighthouse"></a>¿Qué es Azure Lighthouse?

Azure Lighthouse permite la administración de varios inquilinos, lo que proporciona no solo unas mayores automatización y escalabilidad, sino también una mejor gobernanza de recursos e inquilinos. Con Azure Lighthouse, los proveedores de servicios pueden ofrecer servicios administrados mediante herramientas de administración completas y sólidas integradas en la plataforma Azure. Esta oferta también puede ayudar a las organizaciones de TI empresariales a administrar recursos en varios inquilinos.

![Diagrama de información general de Azure Lighthouse](media/azure-lighthouse-overview.jpg)

## <a name="benefits"></a>Ventajas

Azure Lighthouse le ayuda a compilar y ofrecer servicios administrados de forma rentable y eficaz. Entre las ventajas se incluye lo siguiente:

- **Administración a escala**: la involucración del cliente y las operaciones de ciclo de vida para administrar los recursos del cliente son más fáciles y escalables. Las API, las herramientas de administración y los flujos de trabajo existentes se pueden usar con recursos delegados, independientemente de las regiones en que se encuentren.
- **Mayor visibilidad y precisión para los clientes**: los clientes tendrán mayor visibilidad de sus acciones y un control preciso sobre el ámbito que delegan para la administración, incluida la capacidad de quitar el acceso por completo, mientras que la dirección IP se conserva.
- **Herramientas de plataforma completas y unificadas**: nuestra experiencia de herramientas aborda los escenarios clave del proveedor de servicios, incluidos varios modelos de licencia como EA, CSP y pago por uso. Las nuevas funcionalidades funcionan con herramientas y API, modelos de licencias y programas de asociados existentes, como el [Programa Cloud Solution Provider](/partner-center/csp-overview). Azure Lighthouse se puede integrar en sus flujos de trabajo y aplicaciones existentes, y puede realizar un seguimiento de su impacto sobre la involucración del cliente y recibir el crédito obtenido por el asociado [vinculando su identificador de asociado](./how-to/partner-earned-credit.md).

No hay costos adicionales asociados con el uso de Azure Lighthouse para administrar los recursos de Azure. Cualquier cliente o asociado de Azure puede usar Azure Lighthouse.

## <a name="capabilities"></a>Capacidades

Azure Lighthouse incluye varias formas de ayudar a simplificar la administración y la involucración:

- **Administración de recursos delegados de Azure**: Administre los recursos de Azure de sus clientes de forma segura desde su propio inquilino sin tener que cambiar los planos de contexto y control. Las suscripciones y los grupos de recursos se pueden delegar a usuarios y roles especificados en el inquilino de administración, con la capacidad de quitar el acceso según sea necesario. Para obtener más información, consulte la [administración de recursos delegados de Azure](concepts/azure-delegated-resource-management.md).
- **Nuevas experiencias de Azure Portal**: Vea la información de distintos inquilinos en la página [**Mis clientes** de](./how-to/view-manage-customers.md) Azure Portal. Una página de [**proveedores de servicios** correspondiente](how-to/view-manage-service-providers.md) permite a los clientes ver y administrar el acceso de los proveedores de servicios.
- **Plantillas de Azure Resource Manager**: Nuestras plantillas le ayudan a realizar tareas de administración entre inquilinos e incorporar los recursos de clientes delegados. Para más información, consulte nuestro [repositorio de ejemplos](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates) e [Incorporación de un cliente a Azure Lighthouse](how-to/onboard-customer.md).
- **Ofertas de servicio administrado en Azure Marketplace**: Ofrezca sus servicios a los clientes a través de ofertas públicas o privadas e incorpórelos automáticamente a Azure Lighthouse. Para más información, consulte [Ofertas de servicio administrado en Azure Marketplace](concepts/managed-services-offers.md).

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre la [administración de recursos delegados de Azure](concepts/azure-delegated-resource-management.md).
- Explore las [experiencias de administración entre inquilinos](concepts/cross-tenant-management-experience.md).
- Vea cómo [usar Azure Lighthouse en una empresa](concepts/enterprise.md).
