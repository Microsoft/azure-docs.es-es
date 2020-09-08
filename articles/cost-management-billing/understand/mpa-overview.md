---
title: 'Introducción a la cuenta de facturación para un contrato Microsoft Partner Agreement: Azure CSP'
description: Descripción de la cuenta de facturación para un contrato Microsoft Partner Agreement (CSP)
author: bandersmsft
ms.reviewer: amberb
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 98941530ba0483e82f43af222e12df7439b6625e
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88688757"
---
# <a name="get-started-with-your-microsoft-partner-agreement-billing-account"></a>Introducción a la cuenta de facturación para un contrato Microsoft Partner Agreement

Cuando se registra para usar Azure, se crea una cuenta de facturación. Use su cuenta de facturación para administrar las facturas, los pagos y hacer seguimiento de los costos. Puede tener acceso a varias cuentas de facturación. Por ejemplo, es posible que te hayas registrado en Azure para tus proyectos personales. También puede tener acceso a Azure mediante el Contrato Enterprise de la organización, el contrato de cliente de Microsoft o el contrato Microsoft Partner Agreement. Para cada uno de estos escenarios, se dispondría de una cuenta de facturación independiente.

Este artículo se aplica a las cuentas de facturación de Microsoft Partner Agreement. Estas cuentas se crean para que los proveedores de soluciones en la nube (CSP) administren la facturación de sus clientes en la nueva experiencia comercial. La nueva experiencia solo está disponible para asociados que tengan al menos un cliente que haya aceptado un contrato de cliente de Microsoft y que cuente con un plan de Azure. [Compruebe si tiene acceso a un contrato Microsoft Partner Agreement](#check-access-to-a-microsoft-partner-agreement). Un [plan de Azure](https://azure.microsoft.com/pricing/purchase-options/microsoft-customer-agreement/) proporciona a los clientes acceso a los servicios de Azure con las tarifas de pago por uso en virtud del Contrato de cliente de Microsoft.

## <a name="your-billing-account"></a>La cuenta de facturación

Su cuenta de facturación para el contrato Microsoft Partner Agreement contiene un perfil de facturación para cada moneda en la que haga negocios. El perfil de facturación le permite administrar las facturas por su moneda. Cuando establece relaciones con los clientes, en función de sus monedas, las suscripciones a Azure y otras compras se facturan a los respectivos perfiles de facturación.

El siguiente diagrama muestra la relación entre una cuenta de facturación, los perfiles de facturación, los clientes y los revendedores.

![Diagrama que muestra la jerarquía de facturación para un contrato Microsoft Partner Agreement](./media/mpa-overview/mpa-hierarchy.svg)

Los usuarios con el rol **Administrador global**  y **Agentes de administrador** de su organización pueden administrar cuentas de facturación, perfiles de facturación y clientes. Para más información, consulte [Centro de partners: Asignar roles y permisos de usuarios](https://docs.microsoft.com/partner-center/permissions-overview).

## <a name="billing-profiles"></a>Perfiles de facturación

Utilice un perfil de facturación para administrar las facturas para una moneda. Una factura mensual se genera al comienzo del mes para cada perfil de facturación de la cuenta. La factura incluye los cargos en la moneda de los perfiles de facturación para todas las suscripciones a Azure y otras compras realizadas el mes anterior.

Puede ver la factura y descargar los documentos relacionados como el archivo de uso y la hoja de precios en Azure Portal. Para más información, consulte [Descarga de facturas para un contrato Microsoft Partner Agreement](download-azure-invoice.md).

> [!IMPORTANT]
>
> Las facturas para perfiles de facturación contienen cargos por clientes con planes de Azure, así como con SaaS, Azure Marketplace y compras de reserva para clientes que no han aceptado el contrato de cliente de Microsoft y no tienen planes de Azure.

## <a name="customers"></a>Clientes

Puede ver y administrar los clientes que han aceptado un contrato de cliente de Microsoft y cuentan con un plan de Azure en Azure Portal. Puede ver cargos y transacciones, así como crear y administrar suscripciones a Azure para estos clientes.

### <a name="enable-policy-to-give-visibility-into-cost"></a>Habilitación de la directiva para proporcionar visibilidad sobre el costo

Aplique la directiva para controlar si los usuarios de la organización de los clientes pueden ver y analizar los costos en las tarifas de pago por uso para su consumo de Azure. De forma predeterminada, la directiva está desactivada y los usuarios no pueden ver el costo. Una vez habilitada, los usuarios que tengan el acceso a [Azure RBAC](https://docs.microsoft.com/azure/role-based-access-control/overview) adecuado en una suscripción pueden ver y analizar el costo de la suscripción.

Para activar la directiva:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Busque en **Administración de costos + facturación**.

   ![Captura de pantalla que muestra la búsqueda en Azure Portal](./media/mpa-overview/search-cmb.png)

1. Seleccione **Clientes** en la parte izquierda y, a continuación, un cliente de la lista.

   ![Captura de pantalla que muestra la selección del cliente](./media/mpa-overview/mpa-customers.png)

1. Seleccione **Directivas** en el lado izquierdo.

   ![Captura de pantalla que muestra las directivas](./media/mpa-overview/mpa-change-policy.png)

1. Seleccione **Sí**.

## <a name="resellers"></a>Revendedores

Los proveedores indirectos del [modelo de dos niveles](https://docs.microsoft.com/partner-center) de CSP pueden seleccionar un revendedor mientras crean suscripciones para los clientes en Azure Portal. Después de la creación, pueden ver la lista de suscripciones, filtradas por un revendedor y analizar el costo de un cliente por revendedores en el análisis de costos de Azure.

## <a name="check-access-to-a-microsoft-partner-agreement"></a>Comprobación del acceso a un contrato Microsoft Partner Agreement
[!INCLUDE [billing-check-mpa](../../../includes/billing-check-mpa.md)]

## <a name="need-help-contact-support"></a>¿Necesita ayuda? Ponerse en contacto con soporte técnico

Si necesita ayuda, [póngase en contacto con soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos para obtener información sobre la cuenta de facturación:

- [Creación de una suscripción de Azure adicional para el contrato Microsoft Partner Agreement](../manage/create-subscription.md)
- Integración de los datos de facturación en su propio sistema de generación de informes mediante las [API de facturación de Azure](https://docs.microsoft.com/rest/api/billing/)
- [Guía de inicio rápido de Azure Cost Management para asociados](https://go.microsoft.com/fwlink/?linkid=2106482)
