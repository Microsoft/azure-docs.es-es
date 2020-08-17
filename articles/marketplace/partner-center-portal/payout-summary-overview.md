---
title: 'Introducción al resumen de pagos: Azure Marketplace'
description: En el resumen de pagos se muestran los detalles sobre el dinero que ha ganado con su oferta. También le permite saber cuándo recibirá los pagos y cuánto le pagarán.
ms.subservice: partnercenter-marketplace-publisher
ms.service: marketplace
ms.topic: article
author: mingshen-ms
ms.author: mingshen
ms.date: 07/14/2020
ms.openlocfilehash: 2e7658e965931dd78a6db0e2c9fee36edffff98b
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87479819"
---
# <a name="payout-summary-overview"></a>Introducción al resumen de pagos

El [resumen de pagos](./payout-summary.md) muestra detalles sobre el dinero que ha ganado con Microsoft. También le permite saber cuándo recibirá los pagos y cuánto le pagarán.

Si vende ofertas en Azure Marketplace, también verá información sobre los pagos realizados correctamente. Para más información sobre el pago en Azure Marketplace, consulte las [directivas de participación de Azure Marketplace](https://docs.microsoft.com/legal/marketplace/participation-policy) y el [contrato del editor de Microsoft Azure Marketplace](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RE4xqkx).

> [!NOTE]
> Para poder acceder a esta opción, sus ganancias deben alcanzar el [umbral de pago](./payment-thresholds-methods-timeframes.md) de 50 USD. Para más información sobre el umbral de pago, consulte el [contrato del editor de Microsoft Azure Marketplace](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RE4xqkx).

Para ver los detalles de pago, inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home) y seleccione el icono de pago en la esquina superior derecha de la pantalla:

![Se muestra el icono Pago en la esquina superior derecha del portal del Centro de partners.](./media/payout-overview.png)

## <a name="roles-and-permissions"></a>Roles y permisos

Estos son los roles y permisos necesarios para acceder al informe de pago:

| Informes/páginas | Propietario de cuenta | Manager | Desarrollador | Colaborador empresarial | Colaborador financiero | Vendedor |
| --- | --- | --- | --- | --- | --- | --- |
| Informe de adquisición (incluidos los datos casi en tiempo real) | Puede ver | Puede ver | Sin acceso | Sin acceso | Puede ver | Sin acceso |
| Respuesta/informe de comentarios | Puede ver y enviar comentarios | Puede ver y enviar comentarios | Puede ver y enviar comentarios | Sin acceso | Sin acceso | Puede ver y enviar comentarios |
| --- | --- | --- | --- | --- | --- | --- |
| Informe de mantenimiento (incluidos los datos casi en tiempo real) | Puede ver | Puede ver | Puede ver | Puede ver | Sin acceso | Sin acceso |
| Informe de uso | Puede ver | Puede ver | Puede ver | Puede ver | Sin acceso | Sin acceso |
| Cuenta de pago | Puede actualizar | Sin acceso | Sin acceso | Sin acceso | Puede actualizar | Sin acceso |
| Perfil fiscal | Puede actualizar | Sin acceso | Sin acceso | Sin acceso | Puede actualizar | Sin acceso |
| Resumen de pagos | Puede ver | Sin acceso | Sin acceso | Sin acceso | Puede ver | Sin acceso  |
| | | | | | | |

## <a name="payment-schedules"></a>Programaciones de pago

Para obtener una descripción de las programaciones de pago, incluidos los períodos de retención, la visibilidad de los asociados y el momento en que el cliente usa una tarjeta de crédito o factura, consulte la sección sobre [programaciones de pago](./payout-policy-details.md#payment-schedules) en el tema sobre **detalles de pago**.

## <a name="transaction-history-download-export"></a>Exportación de la descarga del historial de transacciones

Esta opción proporciona una descarga de cada artículo de línea que se encuentra en la página de historial de transacciones. Incluye el tipo de ganancia, la fecha, el importe de la transacción asociada, el cliente, el producto y otros detalles de la transacción relacionados con el programa de incentivos.

| Nombre de la columna | Descripción |
| --- | --- |
| earningId | Identificador único de cada ganancia. |
| participantId | Identidad principal del asociado que gana con el programa. |
| participantIdType | Identificador de programa en los programas de incentivos y el vendedor si es un programa de la tienda y Azure Marketplace. |
| participantName | Nombre del asociado que gana. |
| partnerCountryCode | Ubicación, país o región del asociado que gana. |
| programName | Nombre del programa de incentivos/tienda. |
| transactionId | Identificador único de la transacción. |
| transactionCurrency | Moneda en la que se produjo la transacción original del cliente (no es la moneda de la ubicación del asociado). |
| transactionDate | Fecha de la transacción. Útil para programas en los que muchas transacciones contribuyen a una ganancia. |
| transactionExchangeRate | Tipo de cambio usado para mostrar el importe en USD correspondiente de la transacción. |
| transactionAmount | Importe de la transacción en la moneda original de la transacción en función de la ganancia generada. |
| transactionAmountUSD | Importe de la transacción en dólares estadounidenses (USD). |
| lever | Regla de negocio de la ganancia. |
| earningRate | Tasa de incentivos aplicada sobre el importe de la transacción para generar una ganancia. |
| quantity | Cantidad facturada en los programas de transacción. Varía en función del programa. |
| quantityType | Tipo de cantidad, por ejemplo, cantidad facturada, usuarios activos mensuales (MAU). |
| earningType | Indica si se trata de honorarios, devolución, cooperativo, venta, etc. |
| earningAmount | Importe de la ganancia en la moneda original de la transacción. |
| earningAmountUSD | Importe de la ganancia en USD. |
| earningDate | Fecha de la ganancia. |
| calculationDate | Fecha en que se calculó la ganancia en el sistema. |
| earningExchangeRate | Tipo de cambio usado para mostrar el importe en USD correspondiente. |
| exchangeRateDate | Fecha del tipo de cambio usada para calcular earningAmountUSD. |
| paymentAmountWOTax | Importe de la ganancia (sin impuestos) en la moneda del destinatario del pago solo para los pagos &quot;enviados&quot;. |
| paymentCurrency | Moneda del destinatario del pago elegida por el asociado en el perfil de pago. Solo se muestra para los pagos enviados. |
| paymentExchangeRate | Tipo de cambio usado para calcular paymentAmountWOTax en la moneda de pago mediante ExchangeRateDate. |
| paymentId | Identificador único del pago. Este número es visible en el extracto bancario. |
| paymentStatus | Estado del pago. |
| paymentStatusDescription | Descripción del estado del pago. |
| customerId | Siempre estará en blanco. |
| customerName | Siempre estará en blanco. |
| partNumber | Siempre estará en blanco. |
| ProductName | Nombre del producto vinculado a la transacción. |
| productId | Identificador único del producto. |
| parentProductId | Identificador único del producto principal. Si no hay un producto principal para la transacción, el identificador del producto principal es igual al identificador del producto. |
| parentProductName | Nombre del producto principal. Si no hay un producto principal para la transacción, el nombre del producto principal es igual al nombre del producto. |
| productType | Tipo de producto (por ejemplo, aplicación, complemento y juego). |
| invoiceNumber | Número de factura (solo para Contratos Enterprise). |
| resellerId | Identificador del revendedor. |
| resellerName | Nombre del revendedor. |
| transactionType | Tipo de transacción (por ejemplo, compra, reembolso, revocación y contracargo). |
| localProviderSeller | Proveedor o vendedor local del registro. |
| taxRemitted | Importe de impuestos remitidos (ventas, uso o impuestos de IVA/GST). |
| taxRemitModel | Entidad responsable de la emisión de impuestos (ventas, uso o impuestos de IVA/GST). |
| storeFee | La cantidad retenida por Microsoft como honorarios por hacer que la aplicación o el complemento estén disponibles en el marketplace comercial. |
| transactionPaymentMethod | Instrumento de pago del cliente usado para la transacción (por ejemplo, tarjeta, facturación del operador móvil y PayPal). |
| tpan | Red de anuncios de terceros. |
| customerCountry | País o región del cliente. |
| customerCity | Ciudad del cliente. |
| customerState | Estado del cliente. |
| customerZip | Código postal del cliente. |
| TenantID | Identificador del inquilino. |
| externalReferenceId | Identificador único del programa. |
| externalReferenceIdLabel | Etiqueta del identificador único. |
| transactionCountryCode | Código de país o región en el que se ha producido la transacción. |
| taxCountry | País o región del cliente. |
| taxState | Estado del cliente. |
| taxCity | Ciudad del cliente. |
| taxZipCode | Código postal del cliente. |
| LicensingProgramName | Nombre del programa de licencias. |
| Código de programa. | Cadena que se va a asignar con el nombre del programa. |
| earningAmountInLastPaymentCurrency | Importe de la ganancia en la moneda del último pago (el campo estará vacío si no ha habido ningún pago anterior). |
| lastPaymentCurrency | Moneda del último pago (el campo estará vacío si no ha habido ningún pago anterior). |
| AssetId | Identificador único de los pedidos de cliente del servicio de Marketplace. Representa los artículos de línea de compra. Puede haber varios recursos. |
| OrderId | Se relaciona con la factura de un cliente. |
| LineItemId | Línea individual en la factura de un cliente. |
| País o región del cliente. | Nombre del país o región proporcionado por el cliente. Podría ser diferente del país o región de la suscripción a Azure del cliente. |
| Customer EmailAddress | Dirección de correo electrónico proporcionada por el cliente. Puede ser diferente de la dirección de correo electrónico de la suscripción a Azure del cliente. |
| SkuId | Identificador de la SKU definida durante la publicación. Una oferta puede tener muchas SKU, pero una SKU solo puede asociarse a una única oferta. |

> [!NOTE]
> Todos los informes y la información detallada de la opción de publicación de transacciones se pueden encontrar en la sección Análisis del Centro de partners.

## <a name="billing-questions-and-support"></a>Soporte técnico y preguntas sobre facturación
Para obtener soporte técnico sobre facturación, póngase en contacto con el [soporte técnico al editor](https://partner.microsoft.com/support/v2/?stage=1) en el marketplace comercial.

## <a name="next-step"></a>siguiente paso

- [Resúmenes de pagos](./payout-summary.md)
