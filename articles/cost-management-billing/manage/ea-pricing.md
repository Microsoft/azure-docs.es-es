---
title: Visualización y descarga de los precios de Azure de la organización
description: Obtenga información sobre cómo ver y descargar los precios o calcule los costos con los precios de la organización.
author: bandersmsft
ms.reviewer: amberb
tags: billing
ms.service: cost-management-billing
ms.subservice: enterprise
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.custom: seodec18
ms.openlocfilehash: b4905b1aba11a2f8c457d4785897cbe70dd31f76
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88687423"
---
# <a name="view-and-download-your-organizations-azure-pricing"></a>Visualización y descarga de los precios de Azure de la organización

Los clientes de Azure que tienen un Contrato Enterprise (EA) de Azure, un Contrato de cliente de Microsoft (MCA) o un contrato Microsoft Partner Agreement (MPA) pueden ver y descargar los precios en Azure Portal. [Aprenda a comprobar el tipo de cuenta de facturación](#check-your-billing-account-type).

## <a name="download-pricing-for-an-enterprise-agreement"></a>Descarga de los precios de un Contrato Enterprise

Según las directivas que haya establecido el administrador de empresa para la organización, solo determinados roles administrativos proporcionan acceso a la información de precios de EA de la organización. Para más información, consulte [Roles administrativos del Contrato Enterprise de Azure en Azure](understand-ea-roles.md).

1. Como administrador de Enterprise, inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Busque *Administración de costos + facturación*.

   ![Captura de pantalla que muestra la búsqueda en Azure Portal](./media/ea-pricing/portal-cm-billing-search.png)

1. En la cuenta de facturación, seleccione **Uso y cargos**.

   ![Captura de pantalla que muestra el uso y los cargos en Facturación](./media/ea-pricing/ea-pricing-usage-charges-nav.png)

1. Seleccione ![captura de pantalla que muestra la búsqueda de Azure Portal](./media/ea-pricing/download-icon.png) **Descargar** para el mes.

1. En **Hoja de precios**, seleccione **Descargar CSV**.

   ![Captura de pantalla que muestra el botón Descargar CSV de la hoja de precios](./media/ea-pricing/download-ea-price-sheet.png)

## <a name="download-pricing-for-an-mca-or-mpa-account"></a>Descarga de los precios de una cuenta MCA o MPA

Si tiene una cuenta MCA, debe ser el propietario del perfil de facturación, colaborador, lector o administrador de facturas para ver y descargar los precios. Si tiene una cuenta MPA, debe tener el rol de administrador global y agente de administración en la organización del asociado para ver y descargar los precios.

### <a name="download-price-sheets-for-billed-charges"></a>Descarga de las hojas de precios para los cargos facturados

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com).
1. Busque *Administración de costos + facturación*.
1. Seleccione un perfil de facturación. En función de su acceso, es posible que tenga que seleccionar primero una cuenta de facturación.
1. Seleccione **Facturas**.
1. En la cuadrícula de la factura, busque la fila de la factura correspondiente a la hoja de precios que quiere descargar.
1. Haga clic en el botón de puntos suspensivos (`...`) al final de la fila.
![Captura de pantalla que muestra los puntos suspensivos seleccionados](./media/ea-pricing/billingprofile-invoicegrid-new.png).

1. Si quiere ver los precios de los servicios en la factura seleccionada, seleccione **Hoja de precios de la factura**.
1. Si quiere ver los precios de todos los servicios de Azure para el período de facturación determinado, seleccione **Hoja de precios de Azure**.

![Captura de pantalla que muestra el menú contextual con las hojas de precios.](./media/ea-pricing/contextmenu-pricesheet01.png)

### <a name="download-price-sheets-for-the-current-billing-period"></a>Descarga de las hojas de precios para el período de facturación actual

Si tiene una cuenta MCA, puede descargar los precios del período de facturación actual.

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com).
1. Busque *Administración de costos + facturación*.
1. Seleccione un perfil de facturación. En función de su acceso, es posible que tenga que seleccionar primero una cuenta de facturación.
1. En el área **Información general**, busque los vínculos de descarga debajo de los cargos del mes hasta la fecha.
1. Seleccione **Hoja de precios de Azure**.
![Captura de pantalla que muestra la descarga desde Información general](./media/ea-pricing/open-pricing01.png).

## <a name="estimate-costs-with-the-azure-pricing-calculator"></a>Estimación de costos con la calculadora de precios de Azure

También puede usar los precios de la organización para estimar los costos con la calculadora de precios de Azure.

1. Vaya a la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator).
1. En la esquina superior derecha, seleccione **Iniciar sesión**.
1. En **Programs and Offer** > **Licensing Program** (Programas y oferta > Programa de licencias), seleccione **Enterprise Agreement (EA)** (Contrato Enterprise [EA]).
1. En **Programs and Offer** > **Selected agreement**, (Programas y oferta > Seleccionar contrato), seleccione **None selected** (Ninguno seleccionado).

    ![Captura de pantalla que muestra el botón Descargar CSV de la hoja de precios](./media/ea-pricing/ea-pricing-calculator-estimate.png)

1. Elija la organización.
1. Seleccione **Aplicar**.
1. Busque y, a continuación, agregue productos a su estimación.
1. Los precios estimados mostrados se basan en los precios de la organización que ha seleccionado.

## <a name="check-your-billing-account-type"></a>Comprobación del tipo de cuenta de facturación
[!INCLUDE [billing-check-account-type](../../../includes/billing-check-account-type.md)]

## <a name="next-steps"></a>Pasos siguientes

Si es un cliente EA, consulte:

- [Administración del acceso a la información de facturación de EA de Azure](manage-billing-access.md)
- [Visualización y descarga de la factura de Microsoft Azure](../understand/download-azure-invoice.md)
- [Visualización y descarga de los datos de uso y los cargos de Microsoft Azure](../understand/download-azure-daily-usage.md)
- [Descripción de la factura de los clientes con un Contrato Enterprise](../understand/review-enterprise-agreement-bill.md)

Si tiene un contrato de cliente de Microsoft, vea:

- [Descripción de los términos de la hoja de precios para un contrato de cliente de Microsoft](mca-understand-pricesheet.md)
- [Visualización y descarga de la factura de Microsoft Azure](../understand/download-azure-invoice.md)
- [Visualización y descarga de los datos de uso y los cargos de Microsoft Azure](../understand/download-azure-daily-usage.md)
- [Ver los documentos de impuestos del contrato de cliente de Microsoft](../understand/mca-download-tax-document.md)
- [Descripción de los cargos en la factura del perfil de facturación](../understand/review-customer-agreement-bill.md)
