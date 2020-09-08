---
title: Descripción de los gastos de servicios externos de Azure
description: Obtenga información sobre la facturación de los servicios externos, anteriormente conocidos como Marketplace, en Azure.
author: bandersmsft
ms.reviewer: judupont
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 8da85ec5781ff9575cf380092fea9e41d6af8786
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88686129"
---
# <a name="understand-your-azure-external-services-charges"></a>Descripción de los gastos de servicios externos de Azure
Los servicios externos los publican proveedores de software de terceros en Azure Marketplace. Por ejemplo, SendGrid es un servicio externo que puede adquirir en Azure, pero que no está publicado por Microsoft. Algunos productos de Microsoft también se venden a través de Azure Marketplace.

## <a name="how-external-services-are-billed"></a>Cómo se facturan los servicios externos

- Si tiene un Contrato de cliente de Microsoft (MCA) o un contrato Microsoft Partner Agreement (MPA), los servicios de terceros se facturan con el resto de los servicios de Azure. [Compruebe el tipo de cuenta de facturación ](#check-billing-account-type) para ver si tiene acceso a una MCA o MPA.
- Si no tiene una MCA o una MPA, los servicios externos se facturan por separado de los servicios de Azure. Recibirá dos facturas por cada período de facturación: una por los servicios de Azure y otra por las compras de Marketplace.
- Cada servicio externo tiene un modelo de facturación diferente. Algunos servicios se facturan según la modalidad de pago por uso, mientras que otros tienen cargos mensuales fijos.
- No puede usar créditos mensuales gratuitos para los servicios externos. Si usa una suscripción de Azure que incluye [créditos gratuitos](https://azure.microsoft.com/pricing/spending-limits/), estos no se pueden aplicar a los cargos de los servicios externos. Cuando aprovisiona un nuevo servicio externo o un recurso, aparece una advertencia:

    ![Advertencia de compra de Marketplace](./media/understand-azure-marketplace-charges/credit-warning.png)

<!-- ## View external service spending and history in the Azure portal
You can view a list of the external services that are on each subscription within the [Azure portal](https://portal.azure.com/):

1. Sign in to the [Azure portal](https://portal.azure.com/) as the account administrator.
2. In the Hub menu, select **Subscriptions**.

    ![Select Subscriptions in the Hub menu](./media/understand-azure-marketplace-charges/sub-button.png)
3. In the **Subscriptions** blade, select the subscription that you want to view, and then select **External services**.

    ![Select a subscription in the billing blade](./media/understand-azure-marketplace-charges/select-sub-external-services.png)
4. You should see each of your external service orders, the publisher name, service tier you bought, name you gave the resource, and the current order status. To see past bills, select an external service.

    ![Select an external service](./media/understand-azure-marketplace-charges/external-service-blade2.png)
5. From here, you can view past bill amounts including the tax breakdown.

    ![View external services billing history](./media/understand-azure-marketplace-charges/billing-overview-blade.png) -->

## <a name="external-spending-for-ea-customers"></a>Gastos externos para clientes de EA

Los clientes de EA pueden ver los gastos de los servicios externos y descargar los informes en el portal de EA. Lea el artículo sobre [Azure Marketplace para clientes de EA](https://ea.azure.com/helpdocs/azureMarketplace) para empezar a trabajar.

## <a name="view-and-download-invoices-for-external-services"></a>Visualización y descarga de facturas de servicios externos

Si tiene un Contrato de cliente de Microsoft (MCA) o un contrato Microsoft Partner Agreement (MPA), los servicios de terceros se facturan con el resto de los servicios de Azure en una única factura. [Compruebe el tipo de cuenta de facturación ](#check-billing-account-type) para ver si tiene acceso a una MCA o MPA. Si lo tiene, consulte [Visualización y descarga de facturas en Azure Portal](download-azure-invoice.md) para ver los cargos de terceros.

Si no tiene MCA ni MPA, tendrá facturas separadas para los cargos de terceros. 

Los cargos de Azure Marketplace se muestran en la moneda local.

Puede ver y descargar las facturas de Azure Marketplace en Azure Portal. Para ello, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque **Administración de costos + facturación**.
1. En el menú de la izquierda, seleccione **Facturas**.
1. En la lista desplegable de suscripciones, seleccione la que está asociada a los servicios de Marketplace.
1. Revise la columna **Tipo** en la lista de facturas. Si la factura es por un servicio de Marketplace, el tipo será **Azure Marketplace y reservas**. 

    ![Captura de pantalla del tipo de Azure Marketplace en la cuadrícula de facturas](./media/understand-azure-marketplace-charges/marketplace-type-twd.png)

1. Para filtrar por tipo, de modo que solo vea las facturas de Azure Marketplace y reservas, seleccione el filtro **Tipo**. Luego, seleccione **Azure Marketplace y reservas** en el menú desplegable.

    ![Captura de pantalla del filtro Tipo seleccionado, en el que se muestra la opción Azure Marketplace y reservas seleccionada en la lista desplegable](./media/understand-azure-marketplace-charges/type-filter.png)

1. Seleccione el icono de descarga situado a la derecha de la factura que quiere descargar.

    ![Captura de pantalla que muestra el icono de descarga seleccionado de la factura](./media/understand-azure-marketplace-charges/download-icon-marketplace.png)

1. En **Factura**, seleccione el botón azul **Descargar**.

    ![Captura de pantalla que muestra el botón de descarga de la factura en el panel de contexto](./media/understand-azure-marketplace-charges/invoice-download-marketplace.png)

## <a name="pay-for-external-services-in-the-azure-portal"></a>Pago por servicios externos en Azure Portal

Si tiene un Contrato de cliente de Microsoft (MCA) o un contrato Microsoft Partner Agreement (MPA), los servicios de terceros se facturan con el resto de los servicios de Azure. [Compruebe el tipo de cuenta de facturación ](#check-billing-account-type) para ver si tiene acceso a una MCA o MPA. Si lo tiene, puede pagar la factura completa en Azure Portal siguiendo los pasos descritos en [Cómo pagar la factura de Microsoft Azure](pay-bill.md).

Si no tiene una MCA o MPA, puede pagar las facturas de Marketplace en Azure Portal siguiendo estos pasos:

1. Siga los pasos de la sección anterior, [Visualización y descarga de facturas de servicios externos](#view-and-download-invoices-for-external-services) para buscar las facturas de Marketplace.
1. Seleccione el vínculo azul **Pagar ahora** para la factura que quiere pagar.

    ![Captura de pantalla que muestra el vínculo Pagar ahora seleccionado en la cuadrícula de facturas](./media/understand-azure-marketplace-charges/pay-now-twd.png)

    >[!NOTE]
    > Solo verá el vínculo **Pagar ahora** si el tipo de la factura es **Azure Marketplace y reservas** y el estado del pago de la factura es vencido.

1. En la nueva página, haga clic en el vínculo azul **Seleccionar método de pago**.

    ![Captura de pantalla que muestra el vínculo Seleccionar método de pago elegido](./media/understand-azure-marketplace-charges/select-payment-method-pay-now-twd.png)

1. Después de seleccionar un método de pago, haga clic en el botón azul **Pagar ahora** en la parte inferior izquierda de la página.
    ![Captura de pantalla que muestra el botón Pagar ahora seleccionado](./media/understand-azure-marketplace-charges/pay-now-button-twd.png)

## <a name="change-default-payment-for-external-services"></a>Cambio del pago predeterminado de servicios externos

Cuando compra un servicio externo, elige una suscripción de Azure para el recurso. El método de pago de la suscripción de Azure seleccionada se convierte en el método de pago del servicio externo. Para cambiar el método de pago de un servicio externo, debe [cambiar el método de pago de la suscripción de Azure](../manage/change-credit-card.md) asociada a ese servicio externo. Siga estos pasos para saber a qué suscripción está asociado el perdido de servicio externo:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Haga clic en **Todos los recursos** en el menú de navegación de la izquierda.
     ![captura de pantalla del elemento de menú Todos los recursos](./media/understand-azure-marketplace-charges/all-resources.png)
1. Busque el servicio externo.
1. Busque el nombre de la suscripción en la columna **Suscripción**.
    ![captura de pantalla del nombre de la suscripción para el recurso](./media/understand-azure-marketplace-charges/sub-selected.png)
1. Haga clic en el nombre de la suscripción y [actualice el método de pago activo](../manage/change-credit-card.md).

## <a name="cancel-an-external-service-order"></a>Cancelación de un pedido de servicio externo

Si desea cancelar el pedido de servicio externo, debe eliminar el recurso en [Azure Portal](https://portal.azure.com).

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Haga clic en **Todos los recursos** en el menú de navegación de la izquierda.
    ![Captura de pantalla del elemento de menú Todos los recursos](./media/understand-azure-marketplace-charges/all-resources.png)
1. Busque el servicio externo.
1. Active la casilla junto al recurso que quiere eliminar.
1. Seleccione **Eliminar** en la barra de comandos.
    ![Captura de pantalla del botón Eliminar](./media/understand-azure-marketplace-charges/delete-button.png)
1. Escriba *"Sí"* en la hoja de confirmación.
    ![Eliminar recurso](./media/understand-azure-marketplace-charges/delete-resource.PNG)
1. Haga clic en **Eliminar**.

## <a name="check-billing-account-type"></a>Comprobación del tipo de cuenta de facturación
[!INCLUDE [billing-check-account-type](../../../includes/billing-check-mca.md)]

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes
- [Inicio del análisis de costos](../costs/quick-acm-cost-analysis.md)