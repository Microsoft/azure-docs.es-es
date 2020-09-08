---
title: 'Empezar a trabajar con la cuenta de facturación para un Contrato de cliente de Microsoft: Azure'
description: Obtenga información acerca de la cuenta de facturación de su Contrato de cliente de Microsoft, incluidos los perfiles de facturación y los métodos de pago mediante factura.
author: bandersmsft
ms.reviewer: amberbhargava
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 632fac953a54e661164152cda79082aed2c0bc06
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690483"
---
# <a name="get-started-with-your-microsoft-customer-agreement-billing-account"></a>Empezar a trabajar con la cuenta de facturación para un contrato de cliente de Microsoft

Cuando se registra para usar Azure, se crea una cuenta de facturación. Use su cuenta de facturación para administrar las facturas, los pagos y hacer seguimiento de los costos. Puede tener acceso a varias cuentas de facturación. Por ejemplo, es posible que te hayas registrado en Azure para tus proyectos personales. También se puede tener acceso a Azure mediante el Contrato Enterprise de la organización o del contrato de cliente de Microsoft. Para cada uno de estos escenarios, se dispondría de una cuenta de facturación independiente.

Este artículo se aplica a una cuenta de facturación para un contrato de cliente de Microsoft. [Compruebe si tiene acceso a un contrato de cliente de Microsoft](#check-access-to-a-microsoft-customer-agreement).

## <a name="your-billing-account"></a>La cuenta de facturación

La cuenta de facturación para el contrato de cliente de Microsoft contiene uno o más perfiles de facturación que le permiten administrar las facturas y los métodos de pago. Cada perfil de facturación contiene una o más secciones de factura que permiten organizar los costos en la factura del perfil de facturación.

El siguiente diagrama muestra la relación entre una cuenta de facturación, los perfiles de facturación y las secciones de factura.

![Diagrama que muestra la jerarquía de facturación para un contrato de cliente de Microsoft](./media/mca-overview/mca-billing-hierarchy.png)

Los roles en la cuenta de facturación tienen el nivel más alto de permisos. De manera predeterminada, solo el usuario que se registró en Azure obtiene acceso a la cuenta de facturación. Estos roles deben asignarse a los usuarios que necesitan ver las facturas y realizar el seguimiento de los costos de toda la organización, como los administradores de finanzas o de TI. Para más información, consulte [Tareas y roles de la cuenta de facturación](../manage/understand-mca-roles.md#billing-account-roles-and-tasks).

## <a name="billing-profiles"></a>Perfiles de facturación

Utilice un perfil de facturación para administrar la factura y los métodos de pago. Una factura mensual se genera al comienzo del mes para cada perfil de facturación de la cuenta. La factura incluye los cargos respectivos de todas las suscripciones de Azure y otras compras realizadas el mes anterior.

Se crea automáticamente un perfil de facturación para la cuenta de facturación. Contiene una sección de factura de manera predeterminada. Puede crear secciones adicionales para organizar y realizar un seguimiento de los costos fácilmente en función de sus necesidades, ya sea por proyecto, departamento o entorno de desarrollo. Verá estas secciones sobre cómo la factura del perfil de facturación refleja el uso de cada suscripción y las compras que le ha asignado.

Los roles en los perfiles de facturación tienen permisos para ver y administrar las facturas y los métodos de pago. Asigne esos roles a los usuarios que pagan facturas, como miembros del equipo de contabilidad de la organización. Para más información, consulte [Tareas y roles del perfil de facturación](../manage/understand-mca-roles.md#billing-profile-roles-and-tasks).

### <a name="each-billing-profile-gets-a-monthly-invoice"></a>Cada perfil de facturación recibe una factura mensual

Una factura mensual se genera al comienzo del mes para cada perfil de facturación. La factura incluye todos los cargos del mes anterior.

En Azure Portal puede ver la factura, descargar documentos y cambiar la configuración para recibir las facturas futuras por correo electrónico. Para más información, consulte [Descarga de facturas para un contrato de cliente de Microsoft](../manage/download-azure-invoice-daily-usage-date.md#download-invoices-for-a-microsoft-customer-agreement).

### <a name="invoice-payment-methods"></a>Métodos de pago de las facturas

Cada perfil de facturación dispone de sus propios métodos de pago, que se usan para pagar las facturas. Se admiten los siguientes métodos de pago:

| Tipo             | Definición  |
|------------------|-------------|
|Créditos de Azure    |  Los créditos se aplican automáticamente a los cargos elegibles en la factura, con lo que baja el importe que debe pagar. Para más información, consulte [Seguimiento del saldo del crédito de Azure para el perfil de facturación](../manage/mca-check-azure-credits-balance.md). |
|Cheque o transferencia bancaria | Si la cuenta está aprobada para el pago con cheque o transferencia bancaria. Puede pagar el importe de la factura por cheque o transferencia bancaria. Las instrucciones de pago se incluyen en la factura. |
|Tarjeta de crédito | Los clientes que se registran en Azure a través del sitio web de Azure pueden pagar con tarjeta de crédito. |

### <a name="apply-policies-to-control-purchases"></a>Aplicación de directivas para controlar las compras

Aplique directivas para controlar las compras de reservas y de Azure Marketplace mediante un perfil de facturación. Puede establecer directivas para deshabilitar la compra de reservas de Azure y de productos de Marketplace. Cuando se aplican las directivas, las suscripciones que se facturan en el perfil de facturación no se pueden usar para realizar estas compras.

### <a name="azure-plans-determine-pricing-and-service-level-agreement-for-subscriptions"></a>Los planes de Azure determinan los acuerdos de nivel de servicio y los precios de las suscripciones

Los planes de Azure determinan los acuerdos de nivel de servicio y los precios de las suscripciones de Azure. Se activan automáticamente al crear un perfil de facturación. Todas las secciones de factura que están asociadas con el perfil de facturación pueden usar estos planes. Los usuarios con acceso a la sección de factura utilizan los planes para crear suscripciones a Azure. Se admiten los siguientes planes de Azure en las cuentas de facturación para el contrato de cliente de Microsoft:

| Plan             | Definición  |
|------------------|-------------|
|Plan de Microsoft Azure   | Permita a los usuarios crear suscripciones que puedan ejecutar cualquier carga de trabajo.  |
|Plan de Microsoft Azure para desarrollo y pruebas | Permita a los suscriptores de Visual Studio crear suscripciones restringidas para desarrollar o probar cargas de trabajo. Estas suscripciones aportan ventajas como tarifas más bajas y acceso a imágenes exclusivas de máquinas virtuales en Azure Portal. |

## <a name="invoice-sections"></a>Secciones de la factura

Cree secciones en la factura para organizar los costos en ella. Por ejemplo, puede que necesite una única factura para la organización, pero desea organizar los costos por departamento, equipo o proyecto. Para este escenario, cuenta con un único perfil de facturación en el que se crea una sección de factura para cada departamento, equipo o proyecto.

Cuando se crea la sección de factura, puede dar permiso a otros usuarios para crear suscripciones a Azure que se facturan a la sección. Luego, los cargos de uso y las compras correspondientes a las suscripciones se facturan a la sección.

Los roles en la sección de factura tienen permisos para controlar quién crea suscripciones a Azure. Asigne estos roles a los usuarios que configuren el entorno de Azure para los equipos de nuestra organización, como los jefes de ingeniería y los arquitectos técnicos. Para más información, consulte [Tareas y roles de la sección de factura](../manage/understand-mca-roles.md#invoice-section-roles-and-tasks).

## <a name="check-access-to-a-microsoft-customer-agreement"></a>Comprobación del acceso a un contrato de cliente de Microsoft
[!INCLUDE [billing-check-mca](../../../includes/billing-check-mca.md)]

## <a name="need-help-contact-support"></a>¿Necesita ayuda? Póngase en contacto con el servicio de soporte técnico.

Si necesita ayuda, [póngase en contacto con soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos para obtener información sobre la cuenta de facturación:

- [Descripción de los roles administrativos del contrato de cliente de Microsoft en Azure](../manage/understand-mca-roles.md)
- [Creación de una suscripción a Azure adicional para el contrato de cliente de Microsoft](../manage/create-subscription.md)
- [Creación de secciones en la factura para organizar los costos](../manage/mca-section-invoice.md)
