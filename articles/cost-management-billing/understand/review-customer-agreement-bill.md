---
title: 'Revisión de la factura relativa al Contrato de cliente de Microsoft: Azure'
description: Aprenda a revisar la factura y el uso de recursos, y a comprobar los cargos facturados en virtud del Contrato de cliente de Microsoft.
author: bandersmsft
ms.reviewer: judupont
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: tutorial
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 4adb3bb1408357d40e3869c2a0d251b11d015195
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88689531"
---
# <a name="tutorial-review-your-microsoft-customer-agreement-invoice"></a>Tutorial: Revisión de la factura relativa al Contrato de cliente de Microsoft

Puede revisar los cargos facturados mediante el análisis de las distintas transacciones. En la cuenta de facturación de un contrato de cliente de Microsoft, se genera una factura cada mes para cada perfil de facturación. La factura incluye todos los cargos del mes anterior. Puede ver las facturas en Azure Portal y comparar los cargos con el archivo de detalles de uso.

Este artículo se aplica solo a los clientes de Azure con Contrato de cliente de Microsoft.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Revisión de las transacciones facturadas en Azure Portal
> * Revisión de los cargos pendientes para calcular la próxima factura
> * Análisis de los cargos por uso de Azure

## <a name="prerequisites"></a>Prerrequisitos

Debe tener una cuenta de facturación vinculada a un Contrato de cliente de Microsoft.

Debe tener acceso a un Contrato de cliente de Microsoft. Debe ser propietario, colaborador, lector o administrador de facturación del perfil de facturación para ver la información de facturación y de uso. Para más información acerca de los roles de facturación para los contratos de cliente de Microsoft, consulte [Tareas y roles del perfil de facturación](../manage/understand-mca-roles.md#billing-profile-roles-and-tasks).

Deben haber transcurrido más de 30 días desde el momento de suscripción a Azure. Azure le cobra al final del período de facturación.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

- Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="check-access-to-a-microsoft-customer-agreement"></a>Comprobación del acceso a un contrato de cliente de Microsoft

Compruebe el tipo de contrato para determinar si tienen acceso a una cuenta de facturación para un contrato de cliente de Microsoft.

En Azure portal, escriba *administración de costos + facturación* en el cuadro de búsqueda y seleccione **Administración de costos + facturación**.

![Captura de pantalla que muestra la búsqueda en Azure Portal de Administración de costos + facturación](./media/review-customer-agreement-bill/billing-search-cost-management-billing.png)

Si tiene acceso a un solo ámbito de facturación, seleccione **Propiedades** en el lado izquierdo. Tiene acceso a una cuenta de facturación para un contrato de cliente de Microsoft si el tipo de cuenta de facturación es **Contrato de cliente de Microsoft**.

![Captura de pantalla que muestra el Contrato de cliente de Microsoft en la página de propiedades](./media/review-customer-agreement-bill/billing-mca-property.png)

Si tiene acceso a varios ámbitos de facturación, compruebe el tipo en la columna de la cuenta de facturación. Tiene acceso a una cuenta de facturación para un contrato de cliente de Microsoft si el tipo de cuenta de facturación de los ámbitos es **Contrato de cliente de Microsoft**.

![Captura de pantalla que muestra el Contrato de cliente de Microsoft en la página de la lista de la cuenta de facturación](./media/review-customer-agreement-bill/billing-mca-in-the-list.png)

## <a name="review-invoiced-transactions-in-the-azure-portal"></a>Revisión de las transacciones facturadas en Azure Portal

En Azure Portal, seleccione **Todas las transacciones** en el lado izquierdo de la página. En función del acceso, es posible que tenga que seleccionar una cuenta de facturación, un perfil de facturación o una sección de factura y, luego, seleccionar **Todas las transacciones**.

En la página Todas las transacciones se muestra la siguiente información:

![Captura de pantalla que muestra la lista de transacciones facturadas](./media/review-customer-agreement-bill/mca-billed-transactions-list.png)

|Columna  |Definición  |
|---------|---------|
|Date     | La fecha de la transacción  |
|El identificador de la factura     | El identificador de la factura en la se ha facturado la transacción. Si envía una solicitud de soporte técnico, comparta el identificador con el Soporte técnico de Azure para agilizar la solicitud de soporte técnico. |
|Tipo de transacción     |  El tipo de transacción, como los cargos por compra, cancelación y uso  |
|Familia de productos     | La categoría del producto, como proceso para máquinas virtuales o base de datos para la base de datos de Azure SQL|
|SKU de producto     | Un código único que identifica la instancia del producto |
|Importe     |  El importe de la transacción      |
|Sección de factura     | La transacción se muestra en esta sección de la factura del perfil de facturación. |
|Perfil de facturación     | La transacción se muestra en la factura de este perfil de facturación. |

Busque el identificador de la factura para filtrar las transacciones de esta.

### <a name="view-transactions-by-invoice-sections"></a>Visualización de las transacciones por secciones de factura

Las secciones de factura le permiten organizar los costos en una factura del perfil de facturación. Para obtener más información, Cuando se genera una factura, los cargos de todas las secciones del perfil de facturación se muestran en la factura.

En la imagen siguiente se muestran los cargos por la sección de factura del departamento de contabilidad en una factura de ejemplo.

![Imagen de ejemplo que muestra los detalles de la información de la sección de factura](./media/review-customer-agreement-bill/invoicesection-details.png)

Cuando haya identificado los cargos por una sección de factura, puede ver las transacciones en Azure Portal para entender los cargos.

Vaya a la página Todas las transacciones en Azure Portal para ver las transacciones de una factura.

Filtre por nombre de sección de factura para ver las transacciones.

## <a name="review-pending-charges-to-estimate-your-next-invoice"></a>Revisión de los cargos pendientes para calcular la próxima factura

En la cuenta de facturación de un Contrato de cliente de Microsoft, los cargos se calculan y se consideran pendientes hasta que se facturan. Puede ver cargos pendientes en Azure Portal para calcular la próxima factura. Los cargos pendientes son estimados y no incluyen impuestos. Los cargos reales de la próxima factura serán distintos de los cargos pendientes.

### <a name="view-summary-of-pending-charges"></a>Visualización del resumen de los cargos pendientes

Inicie sesión en [Azure Portal](https://portal.azure.com).

Seleccione un perfil de facturación. Dependiendo del acceso, es posible que tenga que seleccionar una cuenta de facturación. En la cuenta de facturación, seleccione **Perfiles de facturación** y, después, uno de los perfiles.

Seleccione la pestaña **Resumen** en la parte superior de la pantalla.

La sección de cargos muestra los cargos del mes hasta la fecha y del mes pasado.

![Captura de pantalla que muestra la búsqueda en Azure Portal de Administración de costos + facturación](./media/review-customer-agreement-bill/mca-billing-profile-summary.png)

Los cargos del mes hasta la fecha son los cargos pendientes para el mes actual y se facturan cuando se genera la factura para el mes. Si la factura del mes pasado todavía no se ha generado, entonces los cargos del mes pasado también están pendientes y aparecerán en la próxima factura.

### <a name="view-pending-transactions"></a>Visualización de las transacciones pendientes

Una vez que se identifiquen los cargos pendientes, puede entender los cargos mediante el análisis de las transacciones individuales que contribuyeron a los cargos. En este momento, los cargos por uso pendientes no se muestran en la página Todas las transacciones. Puede ver los cargos por uso pendientes en la página de suscripciones a Azure.

Inicie sesión en [Azure Portal](https://portal.azure.com).

En Azure portal, escriba *administración de costos + facturación* en el cuadro de búsqueda y seleccione **Administración de costos + facturación**.

Seleccione un perfil de facturación. Dependiendo del acceso, es posible que tenga que seleccionar una cuenta de facturación. En la cuenta de facturación, seleccione **Perfiles de facturación** y, después, uno de los perfiles.

Seleccione **Todas las transacciones** en el lado izquierdo de la página.

Busque *los pendientes*. Utilice el filtro **Intervalo de tiempo** para ver los cargos pendientes del mes actual o del último mes.

![Captura de pantalla que muestra la lista de transacciones pendientes](./media/review-customer-agreement-bill/mca-pending-transactions-list.png)

### <a name="view-pending-usage-charges"></a>Visualización de los cargos por uso pendientes

Inicie sesión en [Azure Portal](https://portal.azure.com).

En Azure portal, escriba *administración de costos + facturación* en el cuadro de búsqueda y seleccione **Administración de costos + facturación**.

Seleccione un perfil de facturación. Dependiendo del acceso, es posible que tenga que seleccionar una cuenta de facturación. En la cuenta de facturación, seleccione **Perfiles de facturación** y, después, uno de los perfiles.

Seleccione **Todas las suscripciones** en el lado izquierdo de la página.

La página de suscripciones a Azure muestra los cargos del mes actual y del último mes para cada suscripción en el perfil de facturación. Los cargos del mes hasta la fecha son los cargos pendientes para el mes actual y se facturan cuando se genera la factura para el mes. Si la factura del mes pasado todavía no se ha generado, entonces los cargos del mes pasado también están pendientes.

![Captura de pantalla que muestra la lista de suscripciones a Azure para el perfil de facturación](./media/review-customer-agreement-bill/mca-billing-profile-subscriptions-list.png)

## <a name="analyze-your-azure-usage-charges"></a>Análisis de los cargos por uso de Azure

Utilice el archivo .csv de cargos y de uso de Azure para analizar los cargos basados en el uso. Puede descargar el archivo para una factura o para cargos pendientes.

### <a name="download-your-invoice-and-usage-details"></a>Descarga de la factura y los detalles de uso

Dependiendo del acceso, en Administración de costos + facturación deberá seleccionar una cuenta de facturación o un perfil de facturación. En el menú de la izquierda, seleccione **Facturas** en **Facturación**. En la cuadrícula de la factura, busque la fila de la factura que desea descargar. Haga clic en el símbolo de descarga o los puntos suspensivos (...) al final de la fila. En el cuadro **Descargar**, descargue los detalles de uso y la factura.

### <a name="view-detailed-usage-by-invoice-section"></a>Visualización del uso detallado por sección de factura

Puede filtrar el archivo de cargos y de uso de Azure para conciliar los cargos de uso para las secciones de factura.

La información siguiente le guiará por la conciliación de los cargos de proceso para la sección de facturación de Accounting Dept (Departamento de contabilidad):

![Imagen de ejemplo que muestra los detalles de la información de la sección de factura](./media/review-customer-agreement-bill/invoicesection-details.png)

| Factura (PDF) | CSV de cargos y de uso de Azure |
| --- | --- |
|Departamento de contabilidad |invoiceSectionName |
|Cargos por uso: plan de Microsoft Azure |productOrderName |
|Proceso |serviceFamily |

Filtre la columna **invoiceSectionName** del archivo CSV  por **Departamento de contabilidad**. A continuación, filtre la columna **productOrderName** del archivo CSV por **Plan de Microsoft Azure**. Después, filtre la columna **serviceFamily** del archivo CSV por **Microsoft.Compute**.

![Captura de pantalla que muestra el archivo de cargos y de uso filtrado por la sección de factura](./media/review-customer-agreement-bill/billing-usage-file-filtered-by-invoice-section.png)

### <a name="view-detailed-usage-by-subscription"></a>Visualización del uso detallado por suscripción

Puede filtrar el archivo .csv de cargos y de uso de Azure para conciliar los cargos de uso para las suscripciones. Una vez identificados los cargos de una suscripción, utilice el archivo .csv de cargos y de uso de Azure para analizar los cargos.

En la imagen siguiente se muestra la lista de suscripciones en Azure Portal.

![Captura de pantalla que muestra la lista de suscripciones a Azure para el perfil de facturación](./media/review-customer-agreement-bill/mca-billing-profile-subscriptions-list-highlighted.png)

Filtre la columna **subscriptionName** en el archivo CSV de cargos y de uso de Azure para **WA_Subscription** con el fin de ver los cargos de uso detallados de WA_Subscription.

![Captura de pantalla que muestra el archivo de cargos y de uso filtrado por suscripción](./media/review-customer-agreement-bill/billing-usage-file-filtered-by-subscription.png)

## <a name="pay-your-bill"></a>Pagar la factura

Las instrucciones para pagar la factura se muestran en la parte inferior de la factura. [Aprenda cómo pagar](mca-understand-your-invoice.md#how-to-pay).

Si ya ha pagado la factura, puede comprobar el estado del pago en la página Facturas de Azure Portal.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Revisión de las transacciones facturadas en Azure Portal
> * Revisión de los cargos pendientes para calcular la próxima factura
> * Análisis de los cargos por uso de Azure

Complete el inicio rápido para empezar a usar el análisis de costos.

> [!div class="nextstepaction"]
> [Inicio del análisis de costos](../costs/quick-acm-cost-analysis.md)
