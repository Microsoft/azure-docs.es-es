---
title: 'Tareas de EA de un Contrato de cliente de Microsoft: Azure'
description: Obtenga información sobre cómo completar tareas del Contrato Enterprise en la nueva cuenta de facturación.
author: amberbhargava
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 928800f22011f6388ec50a1ab5e906b2080a4125
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88682899"
---
# <a name="complete-enterprise-agreement-tasks-in-your-billing-account-for-a-microsoft-customer-agreement"></a>Finalización de tareas del Contrato Enterprise en la cuenta de facturación para un contrato de cliente de Microsoft

Si la organización ha firmado un contrato de cliente de Microsoft para renovar su inscripción al Contrato Enterprise, se crea una nueva cuenta de facturación para el contrato. La facturación de la nueva cuenta se organiza de forma diferente al Contrato Enterprise. En este artículo se describe cómo puede usar la nueva cuenta de facturación para realizar las tareas realizadas en el Contrato Enterprise.

## <a name="billing-organization-in-the-new-account"></a>Organización de facturación en la cuenta nueva

En el diagrama siguiente se describe cómo se organiza la facturación en la nueva cuenta de facturación.

![Imagen de ea-mca-post-transition-hierarchy](./media/mca-enterprise-operations/mca-post-transition-hierarchy.png)

| Contrato Enterprise   | Contrato de cliente de Microsoft    |
|------------------------|--------------------------------------------------------|
| Inscripción            | Utilice un perfil de facturación para administrar la facturación de la organización, de forma similar a la inscripción al Contrato Enterprise. Los administradores de Enterprise se convierten en los propietarios del perfil de facturación. Para más información sobre los perfiles de facturación, consulte [Descripción de los perfiles de facturación](../understand/mca-overview.md#billing-profiles).
| department            | Puede usar una sección de la factura para organizar los costos, de manera similar a los departamentos en la inscripción al contrato Enterprise. El departamento se convierte en las secciones de la factura y los administradores de departamento se convierten en los propietarios de las respectivas secciones de la factura. Para más información sobre las secciones de la factura, consulte [Descripción de las secciones de factura](../understand/mca-overview.md#invoice-sections). |
| Cuenta               | No se admiten las cuentas que se crearon en el Contrato Enterprise en la nueva cuenta de facturación. Las suscripciones de la cuenta pertenecen a la respectiva sección de facturas correspondiente al departamento. Los propietarios de la cuenta pueden crear y administrar suscripciones para sus secciones de factura. |

## <a name="changes-for-enterprise-administrators"></a>Cambios en los administradores de empresa

Los siguientes cambios se aplican a los administradores de empresa en un Contrato Enterprise que se renovó a un contrato de cliente de Microsoft.

- Se crea un perfil de facturación para la inscripción. Va a utilizar un perfil de facturación para administrar la facturación de la organización, como la inscripción al Contrato Enterprise. Para más información, [comprenda los perfiles de facturación](../understand/mca-overview.md#billing-profiles).
- Se crea una sección de factura para cada departamento en la inscripción al Contrato Enterprise. Usará las secciones de factura para administrar los departamentos. Puede crear nuevas secciones de factura para configurar otros departamentos. Para más información sobre las secciones de la factura, consulte [Descripción de las secciones de factura](../understand/mca-overview.md#invoice-sections).
- Usará el rol de creador de suscripciones a Azure en las secciones de factura para dar permiso a otros para crear suscripciones a Azure, como las cuentas que se crearon en la inscripción del Contrato Enterprise.
- Deberá usar [Azure Portal](https://portal.azure.com) para administrar la facturación de la organización, en lugar de EA Portal de Azure.

Se proporcionan los roles siguientes en la nueva cuenta de facturación:

**Propietario del perfil de facturación**: Se le asigna el rol de propietario del perfil de facturación en el perfil de facturación que se creó cuando se firmó el contrato. El rol permite administrar la facturación de la organización. Puede ver los cargos y las facturas, organizar los costos en la factura, administrar los métodos de pago y controlar el acceso a la facturación de la organización.

**Propietario de la sección de factura**: se le asigna el rol de propietario de la sección de factura en todas las secciones de factura que se crean para los departamentos en la inscripción de Contrato Enterprise. Este rol permite controlar quién puede crear suscripciones a Azure y comprar otros productos.

### <a name="view-charges-and-credits-balance-for-your-organization"></a>Visualización de los cargos y del saldo de crédito de la organización

Utilice el perfil de facturación para realizar un seguimiento de los cargos y del saldo de crédito de Azure para la organización, de forma similar a la inscripción en Contrato Enterprise.

Para saber cómo ver el saldo del crédito de un perfil de facturación, consulte [Seguimiento del saldo del crédito de Azure para el perfil de facturación](mca-check-azure-credits-balance.md).

Para más información sobre cómo ver los cargos de un perfil de facturación, consulte [Descripción de los cargos en la factura del contrato de cliente de Microsoft](../understand/review-customer-agreement-bill.md).

### <a name="view-charges-for-a-department"></a>Visualización de los cargos de un departamento

Se crea una sección de factura para cada departamento que tiene en el Contrato Enterprise. Puede ver los cargos de una sección de factura en Azure Portal. Para más información, consulte [Visualización de las transacciones por secciones de factura](../understand/review-customer-agreement-bill.md#view-transactions-by-invoice-sections).

### <a name="view-charges-for-an-account"></a>Visualización de los cargos de una cuenta

No se admiten las cuentas que se crearon en la inscripción al Contrato Enterprise en la nueva cuenta de facturación. Las suscripciones de la cuenta pertenecen a la respectiva sección de facturas correspondiente al departamento. Los propietarios de la cuenta pueden crear y administrar suscripciones para sus secciones de factura.

Para ver el costo total de las suscripciones que pertenecían a una cuenta, debe establecer un centro de coste para cada suscripción. Después, puede usar el archivo csv de cargos y de uso de Azure para filtrar las suscripciones por centro de coste.

### <a name="download-usage-and-charges-csv-price-sheet-and-tax-documents"></a>Descarga de csv de cargos y de uso, hoja de precios y documentos fiscales

Se genera una factura mensual por cada perfil de facturación de la cuenta de facturación. Para cada factura, puede descargar los archivos csv de cargos y de uso de Azure, la hoja de precios y el documento fiscal (si procede). También puede descargar los archivos csv de cargos y de uso de Azure para los cargos del mes actual.

Para obtener información sobre cómo descargar archivos de csv de cargos y de uso de Azure, consulte [Descarga de uso para el contrato de cliente de Microsoft](../understand/download-azure-daily-usage.md).

Para saber cómo descargar la hoja de precios, consulte [Descarga de precios para el contrato de cliente de Microsoft](ea-pricing.md).

Para saber cómo descargar los documentos fiscales, consulte [Visualización de los documentos fiscales para el contrato de cliente de Microsoft](../understand/mca-download-tax-document.md#view-and-download-tax-documents).

### <a name="add-an-additional-enterprise-administrator"></a>Adición de un administrador de empresa adicional

Dé a los usuarios acceso al perfil de facturación para que puedan ver y administrar la facturación de la organización. Puede usar la página **Control de acceso (IAM)** en Azure Portal para dar acceso.  Para más información acerca de los roles del perfil de facturación, consulte [Tareas y roles del perfil de facturación](understand-mca-roles.md#billing-profile-roles-and-tasks).

Para más información sobre cómo proporcionar el acceso al perfil de facturación, consulte [Administración de roles de facturación en Azure Portal](understand-mca-roles.md#manage-billing-roles-in-the-azure-portal).

### <a name="create-a-new-department"></a>Creación de un nuevo departamento

Cree una sección de factura para organizar los costos según sus necesidades, al igual que los departamentos de la inscripción del Contrato Enterprise. Puede crear una nueva sección de facturas en Azure Portal. Para más información, consulte [Creación de secciones de la factura para organizar los costos](mca-section-invoice.md).

### <a name="create-a-new-account"></a>Creación de una nueva cuenta

Asigne a los usuarios el rol de creador de suscripciones a Azure en las secciones de factura para darles permiso para crear suscripciones a Azure, como las cuentas que se crearon en la inscripción del Contrato Enterprise. Para más información sobre la asignación de roles, consulte [Administración de roles de facturación en Azure Portal](understand-mca-roles.md#manage-billing-roles-in-the-azure-portal)

## <a name="changes-for-department-administrators"></a>Cambios en los administradores de departamento

Los siguientes cambios se aplican a los administradores de departamento en un Contrato Enterprise que se renovó a un contrato de cliente de Microsoft.

- Se crea una sección de factura para cada departamento en la inscripción al Contrato Enterprise. Va a usar las secciones de factura para administrar los departamentos. Para más información sobre las secciones de la factura, consulte [Descripción de las secciones de factura](../understand/mca-overview.md#invoice-sections).
- Usará el rol de creador de suscripciones a Azure en las secciones de factura para dar permiso a otros para crear suscripciones a Azure, como las cuentas que se crearon en la inscripción del Contrato Enterprise.
- Deberá usar Azure Portal para administrar la facturación de la organización, en lugar de EA Portal de Azure.

Se proporcionan los roles siguientes en la nueva cuenta de facturación:

**Propietario de la sección de factura**: se le asigna el rol de propietario de la sección de factura en la sección de factura que se crea para los departamentos que tenía en Contrato Enterprise. El rol le permite ver y realizar el seguimiento de los cargos, así como controlar quién puede crear suscripciones a Azure y comprar otros productos de la sección de factura.

### <a name="view-charges-for-your-department"></a>Visualización de los cargos del departamento

Puede ver los cargos de la sección de factura que se crea para el departamento en la página [Administración de costos + facturación](https://portal.azure.com/#blade/Microsoft_Azure_Billing/ModernBillingMenuBlade/Overview) de Azure Portal.

### <a name="add-an-additional-department-administrator"></a>Adición de un administrador de departamento adicional

Se crea una sección de factura para cada departamento que tiene en el Contrato Enterprise. Puede usar la página **Control de acceso (IAM)** en Azure Portal para dar acceso a otros usuarios para ver y administrar la sección de factura. Para más información acerca de los roles de la sección de factura, consulte [Tareas y roles de la sección de factura](understand-mca-roles.md#invoice-section-roles-and-tasks).

Para más información sobre cómo proporcionar el acceso a la sección de factura, consulte [Administración de roles de facturación en Azure Portal](understand-mca-roles.md#manage-billing-roles-in-the-azure-portal).

### <a name="create-a-new-account-in-your-department"></a>Creación de una nueva cuenta del departamento

Asigne a los usuarios el rol de creador de la suscripción a Azure en la sección de facturas que se ha creado para el departamento. Para más información sobre la asignación de roles, consulte [Administración de roles de facturación en Azure Portal](understand-mca-roles.md#manage-billing-roles-in-the-azure-portal)

### <a name="view-charges-for-accounts-in-your-departments"></a>Visualización de los cargos para las cuentas de los departamentos

No se admiten las cuentas que se crearon en la inscripción al Contrato Enterprise en la nueva cuenta de facturación. Las suscripciones de la cuenta pertenecen a la respectiva sección de facturas correspondiente al departamento. Los propietarios de la cuenta pueden crear y administrar suscripciones para sus secciones de factura.

Para ver el costo total de las suscripciones que pertenecían a una cuenta del departamento, debe establecer un centro de coste para cada suscripción. Después, puede usar el archivo csv de cargos y de uso de Azure para filtrar las suscripciones por centro de coste.

## <a name="changes-for-account-owners"></a>Cambios en los propietarios de cuentas

Los propietarios de cuentas en el Contrato Enterprise obtienen permiso para crear suscripciones a Azure en la nueva cuenta de facturación. Las suscripciones existentes a Azure pertenecen a la sección de factura que se creó para el departamento. Si la cuenta no pertenece a un departamento, las suscripciones pertenecen a una sección de factura que se conoce como sección de factura predeterminada.

Para crear suscripciones adicionales a Azure, se le asigna el siguiente rol en la nueva cuenta de facturación.

**Creador de la suscripción a Azure**: se le asigna el rol de creador de la suscripción a Azure en la sección de factura que se crea para el departamento en el Contrato Enterprise. Si la cuenta no pertenece a un departamento, obtiene el rol de creador de la suscripción a Azure en una sección que se conoce como sección de factura predeterminada. El rol le permite crear suscripciones a Azure para la sección de factura.

### <a name="create-an-azure-subscription"></a>Cree una suscripción a Azure

Puede crear suscripciones a Azure para la sección de factura en Azure Portal. Para más información, consulte [Creación de una suscripción a Azure adicional para el contrato de cliente de Microsoft](create-subscription.md).

### <a name="view-charges-for-your-account"></a>Visualización de los cargos de la cuenta

Para ver los cargos por suscripciones que pertenecían a una cuenta, vaya a la [página de suscripciones](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) en Azure Portal. La página de suscripciones muestra los cargos de todas las suscripciones.

### <a name="view-charges-for-a-subscription"></a>Visualización de cargos de una suscripción

Puede ver los cargos de una suscripción o en la [página de suscripciones](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) o el análisis de costos de Azure. Para más información sobre análisis de costos de Azure, consulte [Exploración y análisis de los costos con el análisis de costos](../costs/quick-acm-cost-analysis.md).

## <a name="need-help-contact-support"></a>¿Necesita ayuda? Ponerse en contacto con soporte técnico

Si necesita ayuda, [póngase en contacto con soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.

## <a name="next-steps"></a>Pasos siguientes

- [Descripción de la cuenta de facturación del contrato de cliente de Microsoft](../understand/mca-overview.md)
- [Descripción de la factura](../understand/review-individual-bill.md)
- [Descripción de la factura](../understand/understand-invoice.md)
- [Obtención de la propiedad de la facturación de las suscripciones a Azure de otros usuarios](mca-request-billing-ownership.md)
