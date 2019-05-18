---
title: Comprender la factura de contrato de cliente de Microsoft | Microsoft Docs
description: Obtenga información sobre cómo leer y comprender la factura MCA
services: ''
documentationcenter: ''
author: jureid
manager: jureid
editor: ''
tags: billing
ms.service: billing
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/19/2019
ms.author: banders
ms.openlocfilehash: aee51793c66ae57f740300797b8fdc1799e685cd
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65604042"
---
# <a name="understand-terms-on-your-microsoft-customer-agreement-invoice"></a>Descripción de los términos en la factura del contrato de cliente de Microsoft

En este artículo se aplica a una cuenta de facturación para un contrato de cliente de Microsoft. [Compruebe si tiene acceso a un contrato de cliente de Microsoft](#check-access-to-a-microsoft-customer-agreement).

La factura proporciona un resumen de los cargos y las instrucciones de pago. Se encuentra disponible en Portable Document Format (.pdf) desde [Azure Portal](https://portal.azure.com/) o puede enviarse por correo electrónico. Para obtener más información, consulte [vista y descarga de Microsoft Azure factura](billing-download-azure-invoice.md).

<!-- ## When am I billed?

You are invoiced on a monthly basis. You can find out which day of the month you receive invoices by checking *invoice date* under billing profile properties in the [Azure portal](https://portal.azure.com/). Charges that occur between the end of the billing period and the invoice date are included in the next month's invoice, since they are in the next billing period. The billing period start and end dates for each invoice are listed in the invoice PDF above **Billing Summary**. -->

## <a name="invoice-terms-and-descriptions"></a>Términos de factura y descripciones

Las secciones siguientes enumeran los términos importantes que ve en la factura y las descripciones de cada término.

### <a name="invoice-summary"></a>Resumen de factura

El **resumen de factura** en la parte superior de la primera página y muestra información sobre el perfil de facturación y la forma de pago.

![Sección Resumen de factura](./media/billing-understand-your-invoice-mca/invoicesummary.png)

| Término | DESCRIPCIÓN |
| --- | --- |
| Dirección de venta |Dirección de la entidad jurídica, que se encuentra en las propiedades de cuenta de facturación|
| Dirección de facturación |Dirección del perfil de facturación recibir la factura de facturación, se encuentra en las propiedades de perfil de facturación|
| Perfil de facturación |El nombre del perfil de facturación recibe la factura |
| P.O. de serie |Un número de pedido de compra opcional asignado por usted para el seguimiento |
| Número de factura |Un número de factura único, generado por Microsoft utilizado con fines de seguimiento |
| Fecha de la factura |Fecha en que se genera la factura, normalmente cinco a 12 días después del final del ciclo de facturación. Puede comprobar la fecha de factura de facturación de las propiedades de perfil.|
| Términos de pago |Forma de pago para su factura de Microsoft. *30 días de NET* significa que paga dentro de 30 días de la fecha de factura. |

### <a name="billing-summary"></a>Resumen de facturación

El **resumen de facturación** muestra los cargos en el perfil de facturación desde el período de facturación anterior, los créditos que se aplicaron, impuestos y el importe total due.

![Sección Resumen de facturación](./media/billing-understand-your-invoice-mca/billingsummary.png)

| Término | DESCRIPCIÓN |
| --- | --- |
| Cargos|Número total de cargos de Microsoft para este perfil de facturación desde el último período de facturación |
| Créditos |Créditos que recibió de devoluciones |
| Aplicar créditos de Azure | Créditos de Azure que se aplican automáticamente a Azure cobra cada período de facturación |
| Subtotal |La cantidad sin descontar impuesta de vencimiento |
| Impuestos |El tipo y el importe de impuesto que pagar, según el país o región de su perfil de facturación. Si no tiene que pagar impuestos, no verá impuestos en la factura. |
| Ahorro total estimado |La cantidad total estimada que guardó en efectivos descuentos. Si procede, efectivas tarifas reducidas aparecen debajo de los elementos de línea de compra en detalles de la sección de factura. |

### <a name="invoice-sections"></a>Secciones de factura

Para cada sección de la factura en su perfil de facturación, verá los cargos, la cantidad de créditos de Azure aplicado, impuestos y el importe total a pagar.

`Total = Charges - Azure Credit + Tax`

### <a name="details-by-invoice-section"></a>Detalles de sección de factura

Los detalles muestran el costo para cada sección de la factura desglosada por pedido de producto. Dentro de cada pedido de producto, costo se divide por el tipo de servicio. Puede encontrar los cargos diarios para sus productos y servicios en el portal de Azure y uso de Azure y los cargos de CSV. Para más información, consulte [comprender los cargos en su factura para un contrato de cliente de Microsoft](billing-mca-understand-your-bill.md).

El importe total debido de cada familia de servicios se calcula restando *créditos de Azure* desde *créditos/cargos* y agregando *impuestos*:

<!-- `Total = Charges/Credits - Azure Credit + Tax` -->

![Detalles de sección de factura](./media/billing-understand-your-invoice-mca/invoicesectiondetails.png)

| Término |DESCRIPCIÓN |
| --- | --- |
| Precio unitario | El precio efectivo del servicio (precios de moneda) que se usa para la velocidad del uso. Esto es único para un producto, familia de servicios, medidor y oferta. |
| Qty | Cantidad comprado o consumido durante el período de facturación |
| Los cargos y créditos | Importe neto de cargos después de aplicar créditos y reembolsos |
| Crédito de Azure | La cantidad de créditos de Azure aplicado a los créditos de cargos|
| Tipo impositivo | Tipos impositivos según el país o región |
| Importe de impuestos | Importe de impuesto aplicado para comprar según el tipo impositivo |
| Total | El importe total debido de la compra |

### <a name="how-to-pay"></a>Cómo pagar

En la parte inferior de la factura, hay instrucciones para pagar la factura. Puede pagar por verificación, conexión, o en línea. Si paga en línea, puede usar una tarjeta de crédito o débito o créditos de Azure, si procede.

### <a name="publisher-information"></a>Información del anunciante

Si tiene servicios de terceros en su factura, el nombre y la dirección de cada publicador se muestra en la parte inferior de la factura.

## <a name="check-access-to-a-microsoft-customer-agreement"></a>Comprobar el acceso a un contrato de cliente de Microsoft
[!INCLUDE [billing-check-mca](../../includes/billing-check-mca.md)]

## <a name="need-help-contact-us"></a>¿Ayuda? Ponerse en contacto con nosotros

Si tiene alguna pregunta o necesita ayuda, [crear una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

- [Comprender los cargos de facturación de su perfil de facturación](billing-mca-understand-your-bill.md)
- [Obtención de los datos de uso diario y la factura de Azure](billing-download-azure-invoice-daily-usage-date.md)
- [Ver los precios de Azure de su organización](billing-ea-pricing.md)
- [Vista de documentos de impuestos para el perfil de facturación](billing-mca-download-tax-document.md)
