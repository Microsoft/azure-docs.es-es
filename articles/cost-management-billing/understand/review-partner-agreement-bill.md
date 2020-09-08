---
title: 'Revisión de la factura de Microsoft Partner Agreement: Azure'
description: Aprenda a revisar la factura y el uso de recursos, y a verificar los cargos facturados en virtud de Microsoft Partner Agreement.
author: bandersmsft
ms.reviewer: judupont
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: tutorial
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 9e5ecc67fe86afa15c66d175f0705818154588bf
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88684361"
---
# <a name="tutorial-review-your-microsoft-partner-agreement-invoice"></a>Tutorial: Revisión de la factura de Microsoft Partner Agreement

 En la cuenta de facturación de un contrato Microsoft Partner Agreement, se genera una factura cada mes para cada perfil de facturación. La factura incluye todos los cargos del cliente del mes anterior. Puede comprender los cargos en la factura mediante el análisis de las transacciones individuales en Azure Portal. También puede ver las facturas en Azure Portal y comparar los cargos con el archivo de detalles de uso.

Para más información, consulte [Descarga de facturas en Azure Portal](download-azure-invoice.md).

Este tutorial se aplica solo a los asociados de Azure que tienen un contrato Microsoft Partner Agreement.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Revisión de las transacciones facturadas en Azure Portal
> * Revisión de los cargos pendientes para calcular la próxima factura
> * Análisis de los cargos por uso de Azure

## <a name="prerequisites"></a>Requisitos previos

Debe tener acceso a una cuenta de facturación para un contrato Microsoft Partner Agreement.

Deben haber transcurrido más de 30 días desde el momento en que completó su suscripción a Azure. Azure le cobra al final del período de facturación.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

- Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="check-access-to-a-microsoft-customer-agreement"></a>Comprobación del acceso a un contrato de cliente de Microsoft

Compruebe el tipo de contrato para determinar si tiene acceso a una cuenta de facturación para un contrato Microsoft Partner Agreement.

En Azure portal, escriba *administración de costos + facturación* en el cuadro de búsqueda y seleccione **Administración de costos + facturación**.

![Captura de pantalla que muestra la búsqueda en Azure Portal](./media/review-partner-agreement-bill/billing-search-cost-management-billing.png)

Si tiene acceso a un solo ámbito de facturación, seleccione **Propiedades** en el lado izquierdo. Tiene acceso a una cuenta de facturación para un contrato Microsoft Partner Agreement si el tipo de cuenta de facturación es **Microsoft Partner Agreement**.

![Captura de pantalla que muestra el contrato Microsoft Partner Agreement en la página de propiedades](./media/review-partner-agreement-bill/billing-account-properties-partner-agreement.png)

Si tiene acceso a varios ámbitos de facturación, compruebe el tipo en la columna de la cuenta de facturación. Tiene acceso a una cuenta de facturación para un contrato Microsoft Partner Agreement si el tipo de cuenta de facturación para cualquier ámbito es **Microsoft Partner Agreement**.

![Captura de pantalla que muestra el contrato Microsoft Partner Agreement en la página de lista de la cuenta de facturación](./media/review-partner-agreement-bill/mpa-in-the-list.png)

## <a name="review-invoiced-transactions-in-the-azure-portal"></a>Revisión de las transacciones facturadas en Azure Portal

En Administración de costos + facturación, seleccione **Todas las transacciones** en el lado izquierdo de la página. Dependiendo del acceso, es posible que tenga que seleccionar una cuenta de facturación, un perfil de facturación o un cliente y, luego, **Todas las transacciones**.

En la página Todas las transacciones se muestra la siguiente información:

![Captura de pantalla que muestra la lista de transacciones facturadas](./media/review-partner-agreement-bill/all-transactions.png)

|Columna  |Definición  |
|---------|---------|
|Date     | La fecha de la transacción  |
|El identificador de la factura     | El identificador de la factura en la se ha facturado la transacción. Si envía una solicitud de soporte técnico, comparta el identificador con el Soporte técnico de Azure para agilizar la solicitud de soporte técnico. |
|Tipo de transacción     |  El tipo de transacción, como los cargos por compra, cancelación y uso  |
|Familia de productos     | La categoría del producto, como un proceso para máquinas virtuales o una base de datos para Azure SQL Database|
|SDKU de producto     | Un código único que identifica la instancia del producto |
|Importe     |  El importe de la transacción      |
|Perfil de facturación     | La transacción se muestra en la factura de este perfil de facturación. |

Busque el identificador de la factura para filtrar las transacciones de esta.

## <a name="review-pending-charges-to-estimate-your-next-invoice"></a>Revisión de los cargos pendientes para calcular la próxima factura

Los cargos se calculan y se consideran pendientes hasta que se facturan. Puede ver los cargos pendientes para el perfil de facturación del contrato Microsoft Partner Agreement en Azure Portal para calcular la factura siguiente. Los cargos pendientes son estimados y no incluyen impuestos. Los cargos reales de la próxima factura serán distintos de los cargos pendientes.

### <a name="view-pending-transactions"></a>Visualización de las transacciones pendientes

Una vez que se identifiquen los cargos pendientes, puede entender los cargos mediante el análisis de las transacciones individuales que contribuyeron a los cargos. En este momento, los cargos por uso pendientes no se muestran en la página Todas las transacciones. Puede ver los cargos por uso pendientes en la página de suscripciones a Azure.

En Administración de costos + facturación, seleccione un perfil de facturación. Dependiendo del acceso, es posible que tenga que seleccionar una cuenta de facturación. En la cuenta de facturación, seleccione **Perfiles de facturación** y, después, uno de los perfiles.

Seleccione **Todas las transacciones** en el lado izquierdo de la página.

Busque *los pendientes*. Utilice el filtro **Intervalo de tiempo** para ver los cargos pendientes del mes actual o del último mes.

<!-- ![Screenshot that shows the pending transactions list](./media/billing-mpa-understand-your-bill/mpa-billing-profile-pending-transactions.png) -->

### <a name="view-pending-charges-by-customer"></a>Visualización de cargos pendientes por cliente

En Administración de costos + facturación, seleccione un perfil de facturación. Dependiendo del acceso, es posible que tenga que seleccionar una cuenta de facturación. En la cuenta de facturación, seleccione **Perfiles de facturación** y, después, uno de los perfiles.

Seleccione **Clientes** en el panel izquierdo de la página.

<!-- ![screenshot of billing profile customers list](./media/billing-mpa-understand-your-bill/mpa-billing-profile-customers.png) -->

La página Clientes muestra los cargos del mes actual y del último mes para cada cliente asociado con el perfil de facturación. Los cargos del mes hasta la fecha son los cargos pendientes para el mes actual y se facturan cuando se genera la factura para el mes. Si la factura del mes pasado todavía no se ha generado, entonces los cargos del mes pasado también están pendientes.

### <a name="view-pending-usage-charges"></a>Visualización de los cargos por uso pendientes

En Administración de costos + facturación, seleccione un perfil de facturación. Dependiendo del acceso, es posible que tenga que seleccionar una cuenta de facturación. En la cuenta de facturación, seleccione **Perfiles de facturación** y, después, uno de los perfiles.

Seleccione **Suscripciones de Azure** en el lado izquierdo de la página. La página de suscripciones a Azure muestra los cargos del mes actual y del último mes para cada suscripción en el perfil de facturación. Los cargos del mes hasta la fecha son los cargos pendientes para el mes actual y se facturan cuando se genera la factura para el mes. Si la factura del mes pasado todavía no se ha generado, entonces los cargos del mes pasado también están pendientes.

<!--     ![Screenshot that shows the Azure subscriptions list for MPA billing profile](./media/billing-mpa-understand-your-bill/mpa-billing-profile-subscriptions-list.png) -->

## <a name="analyze-your-azure-usage-charges"></a>Análisis de los cargos por uso de Azure

Utilice el archivo .csv de cargos y de uso de Azure para analizar los cargos basados en el uso. Puede filtrar el archivo de uso y de cargos de Azure para conciliar los cargos de uso de cada producto que aparece en el archivo pdf de la factura. Para ver los cargos detallados de uso de un producto determinado, filtre la columna **product** del archivo CSV de uso y cargos de Azure para que incluya solo el nombre de ese producto.

También puede filtrar la columna **customerName** del archivo CSV de uso y cargos de Azure para ver los cargos de uso diario de cada uno de los clientes. Si desea ver los cargos de uso diario por suscripción de Azure, filtre la columna **subscriptionName**.

## <a name="pay-your-bill"></a>Pagar la factura

Las instrucciones para pagar la factura se muestran en la parte inferior de la factura. Puede pagar por transferencia o por cheque en un plazo de 60 días a partir de la fecha de factura.

Si ya ha pagado la factura, puede comprobar el estado del pago en la página Facturas de Azure Portal.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Revisión de las transacciones facturadas en Azure Portal
> * Revisión de los cargos pendientes para calcular la próxima factura
> * Análisis de los cargos por uso de Azure

Obtenga información sobre el uso de Azure Cost Management para los asociados

> [!div class="nextstepaction"]
> [Introducción a Azure Cost Management para los asociados](../costs/get-started-partners.md)
