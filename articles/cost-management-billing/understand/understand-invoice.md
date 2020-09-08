---
title: Conocimiento de la factura de Azure
description: Aprenda a leer y entender el uso y la factura de la suscripción de Azure
author: bandersmsft
ms.reviewer: jureid
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: b7b115f909c57470bfbdab9cc44e6a258f85b8f1
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88685925"
---
# <a name="understand-terms-on-your-microsoft-azure-invoice"></a>Descripción de los términos en su factura de Microsoft Azure

La factura proporciona un resumen de los gastos e incluye instrucciones del pago. Se encuentra disponible en Portable Document Format (.pdf) desde [Azure Portal](https://portal.azure.com/) o puede enviarse por correo electrónico. Para obtener más información, consulte el artículo sobre [cómo obtener los datos de uso diario y de factura de facturación de Azure](../manage/download-azure-invoice-daily-usage-date.md).

Puntos a tener en cuenta:

-   Si va a usar una suscripción de prueba gratuita, puede obtener información de uso detallado desde Azure Portal, pero no tendrá una factura.

-   En la factura actual pueden aparecer hasta 24 horas de uso al final del período de facturación anterior.

-   Los cargos enumerados en los extractos de facturación para clientes internacionales tienen únicamente carácter estimativo. Los bancos pueden tener costes diferentes para los tipos de conversión.

>[!VIDEO https://www.youtube.com/embed/jWG1lyJe3Mg]

## <a name="detailed-terms-and-descriptions-of-your-invoice"></a>Términos y descripciones detallados de la factura
Las siguientes secciones enumeran los términos importantes que ve en la factura y las descripciones de cada uno.

### <a name="account-information"></a>Información de cuenta

La sección de información de la cuenta de la factura se encuentra en la parte superior de la primera página y muestra información sobre el perfil y la suscripción.

![Sección de información de cuenta de la factura](./media/understand-invoice/account-information-section.png)

| Término | Descripción |
| --- | --- |
| N.º de pedido del cliente |Un número de pedido de compra opcional asignado por usted para el seguimiento |
| Número de factura |Un número de factura único generado por Microsoft usado para fines de seguimiento |
| Ciclo de facturación |Intervalo de fechas que cubre esta factura |
| Fecha de la factura |Fecha en la que se generó la factura, normalmente un día después del final del ciclo de facturación |
| Método de pago |Tipo de pago que se utiliza en la cuenta (factura o tarjeta de crédito). |
| Dirección de facturación |Dirección de facturación que se muestra para la cuenta |
| Oferta de suscripción ("Pago por uso") |Tipo de oferta de suscripción que se adquirió (pago por uso, BizSpark Plus, Pase para Azure, etc.) Para más información, vea [Azure offer types](https://azure.microsoft.com/support/legal/offer-details/) (Tipos de ofertas de Azure). |
| Correo electrónico del propietario de cuenta | La dirección de correo electrónico de la cuenta con que está registrada la cuenta de Microsoft Azure. <br /><br />Para cambiar la dirección de correo electrónico, consulte [Cómo cambiar la información de perfil de la cuenta de Azure, como el correo electrónico, la dirección y el número de teléfono de contacto](../manage/change-azure-account-profile.md). |

### <a name="understand-the-invoice-summary"></a>Comprender el resumen de factura
La sección **Resumen de factura** de la factura muestra las cantidades de transacciones totales desde el último período de facturación y los cargos de uso actuales.

![Sección Resumen de factura](./media/understand-invoice/invoice-summary-section.png)

En nombre de suscripción ("Almacenamiento de producción") es el nombre de la suscripción para esta factura.

#### <a name="understand-the-previous-charges"></a>Conocimiento de los cargos anteriores
Las secciones Saldo anterior, Pagos y Saldo pendiente de la factura resumen las transacciones desde el último período de facturación.

| Término | Descripción |
| --- | --- |
| Saldo anterior |El importe total debido del último período de facturación |
| Pagos |Los pagos y créditos totales aplicados al último período de facturación |
| Saldo pendiente (desde el ciclo de facturación anterior) |Los créditos o el balance restante en la cuenta desde el último período de facturación |

#### <a name="understand-the-current-charges"></a>Comprender los cargos actuales
La sección Cargos actuales de la factura muestra detalles sobre el período de facturación actual.

| Término | Descripción |
| --- | --- |
| Cargos de uso |Los cargos de uso son los cargos totales mensuales de una suscripción para el período de facturación actual.|
| Descuentos |Descuentos de servicio aplicados al período de facturación actual.|
| Ajustes |Créditos diversos (uso gratuito, créditos, etc.) o cargos pendientes aplicados al período de facturación actual.<br/><br/>Por ejemplo, si tiene la oferta de Visual Studio Enterprise con MSDN, verá un crédito mensual. Si cancela su suscripción, verá los cargos por uso mensual que superan el crédito mensual que obtiene con su oferta de suscripción. Los cargos son desde el principio de su período de facturación actual hasta la fecha de cancelación de suscripción. |

#### <a name="sold-to-and-payment-instructions"></a>Dirección de venta e instrucciones de pago

En la tabla siguiente se describen la dirección de venta y las instrucciones de pago que se muestran en la segunda página de la factura.

| Término |Descripción |
| --- | --- |
| Dirección de venta |Dirección de perfil que figura en la cuenta. <br/><br/>Si tiene que modificar la dirección, consulte [Cómo cambiar la información de perfil de la cuenta de Azure, como el correo electrónico, la dirección y el número de teléfono de contacto](../manage/change-azure-account-profile.md).|
| Instrucciones de pago |Instrucciones sobre cómo realizar un pago según el método de pago (por ejemplo, mediante tarjeta de crédito o factura). |

#### <a name="usage-charges"></a>Cargos de uso

La sección Cargos de uso de la factura muestra la información de nivel de medidor sobre los cargos.

![Sección Cargos de uso](./media/understand-invoice/usage-charges-section.png)

La tabla siguiente describe los encabezados de columna de los cargos de uso que se muestra en la factura.

| Término |Descripción |
| --- | --- |
| Nombre |Identifica el servicio de nivel superior para el uso. |
| Tipo |Define el tipo de servicio de Azure que puede afectar a la tarifa. |
| Resource |Identifica la unidad de medida del medidor que se está utilizando. |
| Region |Identifica la ubicación del centro de datos para ciertos servicios cuyos precios se establecen según la ubicación del centro de datos. |
| Consumida |La cantidad del medidor usado durante el período de facturación. |
| Se incluye |El importe del medidor incluido sin cargo en el período de facturación actual. |
| Facturable |Muestra la diferencia entre la cantidad consumida y la cantidad incluida. Se factura este importe. En el caso de ofertas de pago por uso que no incluyen el importe en la oferta, este total será igual a la cantidad consumida. |
| Tarifa |La tarifa que se le carga por unidad facturable. |
| Value |Muestra el resultado de multiplicar las columnas Cantidad superior al límite y Tarifa. Si la cantidad consumida no supera la cantidad incluida, no hay ningún cargo en esta columna. |
| Subtotal |La suma de todos los cargos antes de impuestos para este período de facturación. |
| Total general |La suma de todos los cargos después de impuestos para este período de facturación. |

## <a name="how-do-i-make-sure-that-the-charges-in-my-invoice-are-correct"></a>¿Cómo puedo asegurarme de que los cargos de mi factura son correctos?
Si hay un cargo en la factura del que quiera tener más detalles, vea [Descripción de la factura de Microsoft Azure.](review-individual-bill.md)

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).
