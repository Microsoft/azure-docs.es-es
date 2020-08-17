---
title: 'Informes de pagos: Marketplace comercial de Microsoft'
description: En los informes de pagos se muestran detalles sobre el dinero que ha obtenido con la oferta, incluidos el importe del pago y el momento en que se le pagará.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: mingshen-ms
ms.author: mingshen
ms.date: 04/24/2020
ms.openlocfilehash: a013dd875fc5a97e5e2e619553cdeb8c6d729298
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502637"
---
# <a name="payout-summaries"></a>Resúmenes de pagos

El resumen de pagos muestra detalles sobre el dinero que ha ganado con Microsoft. También le permite saber cuándo recibirá los pagos y cuánto le pagarán.

Si vende productos en Azure Marketplace, también verá información sobre los pagos realizados correctamente en el resumen de pagos. Para obtener detalles, consulte [Directivas de participación de Microsoft Azure Marketplace](https://docs.microsoft.com/legal/marketplace/participation-policy) y [Acuerdo del anunciante de Microsoft Azure Marketplace](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RE4xqkx).

> [!NOTE]
> Para poder acceder a esta opción, sus ganancias deben alcanzar el [umbral de pago](payment-thresholds-methods-timeframes.md) de 50 USD. Para más información sobre el umbral de pago, consulte esta página y revise el acuerdo para desarrolladores de aplicaciones.

> [!NOTE]
> Si busca soporte técnico en relación con los pagos, incluidos la configuración de cuentas de pago, los pagos que faltan, la puesta en espera de los pagos o cualquier otra cosa, póngase en contacto con el servicio de soporte técnico [aquí](https://developer.microsoft.com/windows/support).

## <a name="access-the-payout-summary-pages"></a>Acceso a las páginas Resúmenes de pagos

Para abrir una de las páginas Resúmenes de pagos:

1. Seleccione el icono Pago de la esquina superior derecha.
2. Seleccione Historial de transacciones, Pagos o Exportar datos.

## <a name="transaction-history-page"></a>Página Historial de transacciones

En esta página se muestran todas sus ganancias individuales, incluidos la fecha, el tipo y la obtención de cada uno de ellos. Puede seleccionar un período de tiempo para verlo y también puede filtrar por identificador de inscripción, programa, identificador de pago, tipo de ganancia, palanca y estado. Los datos están disponibles durante el año fiscal actual (del 1 de julio al 30 de junio) y los dos años fiscales anteriores.

Para ver más detalles sobre una ganancia, seleccione la flecha abajo que se encuentra en el lado derecho de la página. Esto mostrará la palanca, el importe de los ingresos y el producto. Si algunos de estos datos no están disponibles, pero necesita tener acceso a dicha información, póngase en contacto con el [servicio de soporte técnico](https://developer.microsoft.com/windows/support). Si la ganancia es el resultado de un ajuste y no de una transacción, no se mostrarán los campos destinados al producto.

Para exportar cualquiera de los datos de transacción en esta página, use la página **Exportar datos**.

## <a name="payments-page"></a>Página Pagos

Los totales de esta página representan todos los programas en los que participa. Puede filtrar por identificador de participante, programa, identificador de pago y tipo de ganancia. Los importes se indican en dólares estadounidenses. El valor de pago también se muestra en la moneda del destinatario del pago.

| Área                   | Descripción                                                                                |
|------------------------|---------------------------------------------------------------------------------------------|
| Total pagado este año   | El total combinado que se le ha pagado este año, en dólares estadounidenses, por todos sus programas.       |
| Siguiente pago estimado | El siguiente pago único que recibe (incluso si hay otros próximamente), en dólares estadounidenses. |
| Último pago           | El importe (en dólares estadounidenses), el nombre del programa y el programa de su pago más reciente.           |
| Pagos por origen     | Importe de los pagos (en dólares estadounidenses) representado por el programa en los últimos 12 meses.           |
| Pagos               | Seleccione **Pagado** o **Pendiente** y, a continuación, ordene como desee. Para obtener detalles adicionales de un pago específico, seleccione **Ver**. Para descargar una copia del extracto de remesa de pago, seleccione **Descargar**. Dado que los datos del historial de transacciones pueden tardar hasta 24 horas en aparecer, puede que no vea las ganancias asociadas de inmediato. |
|||

Para exportar cualquiera de los datos de esta página, seleccione **Exportar** y siga las instrucciones de la página Exportar datos.

## <a name="transaction-history-page"></a>Página Historial de transacciones

En esta página se muestran todas sus ganancias individuales, incluidos la fecha, el tipo y la obtención de cada uno de ellos. Puede seleccionar un período de tiempo para verlo y puede filtrar por identificador de inscripción, programa, identificador de pago, tipo de ganancia, palanca y estado. Los datos están disponibles durante el año fiscal actual (del 1 de julio al 30 de junio) y los dos años fiscales anteriores.

Para ver más detalles sobre una ganancia, seleccione la flecha abajo que se encuentra en el lado derecho de la página. Esto mostrará la palanca, el importe de los ingresos y el producto. Si algunos de estos datos no están disponibles, pero necesita tener acceso a dicha información, póngase en contacto con el [servicio de soporte técnico](https://developer.microsoft.com/windows/support)]. Si la ganancia es el resultado de un ajuste y no de una transacción, no se mostrarán los campos destinados al producto.

Para exportar cualquiera de los datos de transacción en esta página, seleccione **Exportar** y siga las instrucciones de la página Exportar datos. Los archivos exportados desde la página Historial de transacciones muestran los datos en la moneda de la transacción, las ganancias en la moneda de la transacción y en dólares estadounidenses, y el valor de pago en la moneda del destinatario del pago.

## <a name="payment-status"></a>Estado del pago.

| Estado de la ganancia           | Motivo                                                                                                                                      | ¿Se requiere una acción del asociado?                                   |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------|
| Sin procesar              | La ganancia es válida para el pago. Permanece en este estado durante un período de enfriamiento, tal como se define en la guía del programa de incentivos. | No                                                         |
| Próximamente                 | Revisiones internas pendientes generadas por la orden de pago antes de que se procese el pago.                                                               | No                                                         |
| Factura de impuestos pendiente      | Su factura de impuestos está incompleta o no es válida.                                                                                                  | Debe actualizar su factura de impuestos antes de recibir el pago. |
| Rechazado durante la revisión   | Se rechazó el pago durante la revisión.                                                                                                     | Para obtener información, póngase en contacto con el [servicio de soporte técnico de Microsoft](https://developer.microsoft.com/windows/support).                      |
| Con error                   | No se pudo realizar el pago debido a un error del sistema de Microsoft.                                                                                         | Para obtener información, póngase en contacto con el [servicio de soporte técnico de Microsoft](https://developer.microsoft.com/windows/support).                      |
| En curso              | El pago está en curso.                                                                                                                 | No                                                         |
| Pago incorrecto        | La recuperación del pago está en curso.                                                                                                       | No                                                         |
| Enviado                     | El pago se ha enviado al banco.                                                                                                     | No                                                         |
| Reprocesamiento             | Se ha detectado un error del sistema de Microsoft durante el pago y se está procesando de nuevo.                                                                  | No                                                         |
| Reversed                 | El banco revirtió el pago y se enviará de nuevo en el siguiente ciclo de pago.                                                     | No                                                         |
| Factura de impuestos rechazada     | Se rechazó su factura de impuestos durante la revisión. Todos los pagos pendientes se pondrán en espera hasta que se complete la revisión de la factura de impuestos.                 | Para obtener información, póngase en contacto con el [servicio de soporte técnico de Microsoft](https://developer.microsoft.com/windows/support).                      |
| Factura de impuestos en revisión | Se está revisando su factura de impuestos. El pago se publicará una vez que se haya aprobado la factura de impuestos.                                   | No                                                         |
| Rechazada                 | El banco rechazó el pago.                                                                                                      | Póngase en contacto con su banco para obtener información.                             |
|||

## <a name="export-data-page"></a>Página Exportar datos

Siga estas instrucciones para exportar datos.

La página Exportar datos no se actualiza por sí misma. Es posible que tenga que actualizar la página manualmente para ver los datos más recientes.

El filtro puede dar lugar a un error **No hay datos disponibles**. Probablemente esto signifique que ha dejado el período de tiempo predeterminado seleccionado en tres meses y, a continuación, ha seleccionado un identificador de pago de una ganancia que no pertenece a ese período. Amplíe el período de tiempo y vuelva a intentarlo.

## <a name="payments"></a>Pagos

![Exportar pagos](./media/pc-export-payments.png)

Esta opción proporciona una descarga de los pagos que recibió en el banco por un programa determinado, los impuestos asociados y el importe de la ganancia agregada. Este informe se usa para muchos programas del Centro de partners, por lo que es posible que algunas columnas no sean aplicables a su informe. Esas columnas se marcan a continuación.

| Nombre de la columna              | Descripción                                                                                                                               |
|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------  |
| participantID            | Identidad principal del asociado que gana con el programa.                                                                             |
| participantIDType        | Normalmente el identificador de programa de los programas de incentivos y el identificador de vendedor de los programas de la Tienda.                                                                |
| participantName          | Nombre del asociado que gana.                                                                                                               |
| programName              | Nombre del programa de incentivos/tienda.                                                                                                              |
| earned                   | Importe obtenido en la moneda del destinatario del pago para ese programa/participantID.                                                                       |
| earnedUSD                | Importe obtenido para el programa/identificador de participante, en dólares estadounidenses.                                                                                      |
| withheldTax              | Importe de los impuestos retenidos en la moneda del destinatario del pago para el programa/participantID.                                                               |
| salesTax                 | Importe total del impuesto sobre las ventas en la moneda del destinatario del pago para el programa/participantID (aplicable solo para programas de incentivos).                   |
| serviceFeeTax            | Importe total de serviceFeeTax en la moneda del destinatario del pago para el programa/participantID (aplicable solo para programas de la Tienda y Azure Marketplace). |
| totalPayment             | Pago total en la moneda local que excluye la retención de impuestos e incluye el impuesto sobre las ventas (si es necesario) para el programa/participantID.   |
| currencyCode             | Código de la moneda del destinatario del pago.                                                                                                                      |
| paymentMethod            | El método que se usa para pagar al asociado, como, por ejemplo, transferencia bancaria electrónica o nota de crédito.                                                     |
| paymentID                | Identificador único del pago. Este número suele aparecer en el extracto del banco (aplicable solo para pagos de SAP).              |
| paymentStatus            | Estado del pago.                                                                                                                            |
| paymentStatusDescription | Descripción detallada del estado del pago.                                                                                                    |
| paymentDate              | Fecha en que Microsoft envió el pago.                                                                                                      |
|||

## <a name="transaction-history"></a>Historial de transacciones

![Exportar historial de transacciones](./media/pc-export-transaction.png)

Esta opción proporciona la descarga de cada artículo de línea de ganancia que se ve en la página Historial de transacciones, el tipo de ganancia, la fecha, el importe de transacción asociado, el cliente, el producto y otros detalles transaccionales aplicables a sus programas.

| Nombre de la columna                    | Descripción                                                                                                                              | Aplicabilidad de incentivos/Tienda/Azure Marketplace           |
|--------------------------------|------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------|
| earningId                      | Identificador único de cada ganancia.                                                                                                       | All                                                            |
| participantId                  | Identidad principal del asociado que gana con el programa.                                                                            | All                                                            |
| participantIdType              | Principalmente el identificador de programa de los programas de incentivos y el IF de vendedor de los programas de la tienda y de Azure Marketplace.                                          | All                                                            |
| participantName                | Nombre del asociado que gana.                                                                                                              | All                                                            |
| partnerCountryCode             | Ubicación, país o región del asociado que gana.                                                                                                  | All                                                            |
| programName                    | Nombre del programa de incentivos/tienda.                                                                                                             | All                                                            |
| transactionId                  | Identificador único de la transacción.                                                                                                    | All                                                            |
| transactionCurrency            | Moneda en la que se produjo la transacción original del cliente (no es la moneda de la ubicación del asociado).                                     | All                                                            |
| transactionDate                | Fecha de la transacción. Útil para programas en los que muchas transacciones contribuyen a una ganancia.                                           | All                                                            |
| transactionExchangeRate        | Tipo de cambio usado para mostrar el importe en USD correspondiente de la transacción.                                                                 | All                                                            |
| transactionAmount              | Importe de la transacción en la moneda original de la transacción en función de la ganancia generada.                                              | All                                                            |
| transactionAmountUSD           | Importe de la transacción en USD.                                                                                                                | All                                                            |
| lever                          | Indica la regla de negocios de la ganancia.                                                                                                  | All                                                            |
| earningRate                    | Tasa de incentivos aplicada sobre el importe de la transacción para generar una ganancia.                                                                      | All                                                            |
| quantity                       | Varía en función del programa. Indica la cantidad facturada en los programas de transacción.                                                            | All                                                            |
| quantityType                   | Indica el tipo de cantidad, como, por ejemplo, la cantidad facturada o MAU.                                                                             | All                                                            |
| earningType                    | Indica si se trata de honorarios, devolución, cooperativo, venta, etc.                                                                                          | All                                                            |
| earningAmount                  | Importe de la ganancia en la moneda original de la transacción.                                                                                      | All                                                            |
| earningAmountUSD               | Importe de la ganancia en USD.                                                                                                                    | All                                                            |
| earningDate                    | Fecha de la ganancia.                                                                                                                      | All                                                            |
| calculationDate                | Fecha en que se calculó la ganancia en el sistema.                                                                                            | All                                                            |
| earningExchangeRate            | Tipo de cambio usado para mostrar el importe en USD correspondiente.                                                                                  | All                                                            |
| exchangeRateDate               | Fecha del tipo de cambio usada para calcular earningAmountUSD.                                                                                   | All                                                            |
| paymentAmountWOTax             | Importe de la ganancia (sin impuestos) en la moneda del destinatario del pago solo para los pagos "enviados".                                                                 | All                                                            |
| paymentCurrency                | Moneda del destinatario del pago elegida por el asociado en el perfil de pago. Solo se muestra para los pagos enviados.                                                   | All                                                            |
| paymentExchangeRate            | Tipo de cambio usado para calcular paymentAmountWOTax en la moneda de pago mediante ExchangeRateDate.                                            | All                                                            |
| claimId                        | Identificador único de una notificación.                                                                                                              | Incentivos: solo algunos programas                                |
| planId                         | Identificador único de un plan.                                                                                                               | Incentivos: solo algunos programas                                |
| paymentId                      | Identificador único del pago. Este número suele aparecer en el extracto del banco.                                                 | Solo pagos de SAP                                              |
| paymentStatus                  | Estado del pago.                                                                                                                           | All                                                            |
| paymentStatusDescription       | Descripción detallada del estado del pago.                                                                                                   | All                                                            |
| customerId                     | Siempre estará en blanco.                                                                                                                     | Solo programas de incentivos (excepción: OEM) y Azure Marketplace |
| customerName                   | Siempre estará en blanco.                                                                                                                     | Solo programas de incentivos (excepción: OEM) y Azure Marketplace |
| partNumber                     | Siempre estará en blanco.                                                                                                                     | Algunos programas de la Tienda e incentivos y Azure Marketplace.        |
| ProductName                    | Nombre del producto vinculado a la transacción.                                                                                                       | All                                                            |
| productId                      | Identificador único del producto.                                                                                                                | Tienda y Azure Marketplace                                    |
| parentProductId                | Identificador único del producto principal. Si no hay un producto principal para la transacción, el identificador del producto principal es igual al identificador del producto. | Tienda y Azure Marketplace                                    |
| parentProductName              | Nombre del producto principal. Si no hay un producto principal para la transacción, el nombre del producto principal es igual al nombre del producto.   | Tienda y Azure Marketplace                                    |
| productType                    | Tipo de producto, como, por ejemplo, aplicación, complemento o juego.                                                                                        | Tienda y Azure Marketplace                                    |
| invoiceNumber                  | Número de factura (aplicable solo para EA).                                                                                                  | Incentivo y Azure Marketplace: solo algunos programas           |
| subscriptionId                 | Identificador de suscripción asociado al cliente.                                                                                         | Incentivo: solo algunos programas                                 |
| subscriptionStartDate          | Fecha de inicio de la suscripción.                                                                                                                  | Incentivo: solo algunos programas                                 |
| subscriptionEndDate            | Fecha de finalización de la suscripción.                                                                                                                    | Incentivo: solo algunos programas                                 |
| resellerId                     | Identificador del revendedor.                                                                                                                      | Incentivo: solo algunos programas                                 |
| resellerName                   | Nombre del revendedor.                                                                                                                            | Incentivo: solo algunos programas                                 |
| distributorId                  | Identificador de distribuidor.                                                                                                                   | Incentivo: solo algunos programas                                 |
| distributorName                | Nombre del distribuidor.                                                                                                                         | Incentivo: solo algunos programas                                 |
| agreementNumber                | Número de acuerdo.                                                                                                                         | Incentivo: solo algunos programas                                 |
| agreementStartDate             | Fecha de inicio del acuerdo.                                                                                                                     | Incentivo: solo algunos programas                                 |
| agreementEndDate               | Fecha de finalización del acuerdo.                                                                                                                       | Incentivo: solo algunos programas                                 |
| carga de trabajo                       | Carga de trabajo                                                                                                                                 | Incentivo: solo algunos programas                                 |
| transactionType                | Tipo de transacción, como, por ejemplo, compra, reembolso, revocación o contracargo.                                                               | Tienda y Azure Marketplace                                    |
| localProviderSeller            | Proveedor o vendedor local del registro.                                                                                                          | Solo Tienda                                                     |
| taxRemitted                    | Importe del impuesto remitido (ventas, uso o impuestos de IVA/GST).                                                                                   | Tienda y Azure Marketplace                                    |
| taxRemitModel                  | Entidad responsable de la emisión de impuestos (ventas, uso o impuestos de IVA/GST).                                                                    | Solo Tienda                                                     |
| storeFee                       | El importe retenido por Microsoft como honorarios por hacer que la aplicación o el complemento estén disponibles en la Tienda.                                           | Solo Tienda                                                     |
| transactionPaymentMethod       | Instrumento de pago del cliente usado para la transacción, como, por ejemplo, tarjeta, facturación del operador móvil o PayPal.                                | Tienda y Azure Marketplace                                    |
| tpan                           | Indica la red de anuncios de terceros.                                                                                                     | Tienda: solo anuncios                                               |
| customerCountry                | Región o país del cliente                                                                                                                         | Tienda y Azure Marketplace                                    |
| customerCity                   | Ciudad del cliente.                                                                                                                            | Tienda y Azure Marketplace                                    |
| customerState                  | Estado del cliente.                                                                                                                           | Tienda y Azure Marketplace                                    |
| customerZip                    | Código postal del cliente.                                                                                                                 | Tienda y Azure Marketplace                                    |
| purchaseTypeCode               | Siempre estará en blanco.                                                                                                                     | Programa de incentivos: CRI                                        |
| purchaseOrderType              | Siempre estará en blanco.                                                                                                                     | Programa de incentivos: CRI                                        |
| purchaseOrderCoverageStartDate | Siempre estará en blanco.                                                                                                                     | Programa de incentivos: CRI                                        |
| purchaseOrderCoverageEndDate   | Siempre estará en blanco.                                                                                                                     | Programa de incentivos: CRI                                        |
| programOfferingLevel           |                                                                                                                                          | Programa de incentivos: CRI                                        |
| TenantID                       |                                                                                                                                          | Programas de incentivos                                             |
| externalReferenceId            | Identificador único del programa.                                                                                                        | Programas de pago directo (incentivo y Tienda)                      |
| externalReferenceIdLabel       | Etiqueta del identificador único.                                                                                                                  | Programas de pago directo (incentivo y Tienda)                      |
|||

## <a name="historical-statements"></a>Extractos históricos

![Exportar extractos históricos](./media/pc-export-statements.png)

El historial de transacciones anterior al 1 de julio de 2019 se administra por separado. Los extractos usarán los campos siguientes en lugar de los actuales.

> [!NOTE]
> El historial de transacciones heredadas tiene una columna llamada "Reservado", equivalente a la columna "Ganancias" del historial moderno, salvo en que excluye todas las ganancias con estado "Pago enviado".

> [!NOTE]
> Filtros como 3 M, 6 M o 12 M no se aplicarán a la sección **Extractos históricos**.

| Nombre del campo              | Descripción                                                                                                                                                             |
|-------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Origen de ingresos          | El origen de los ingresos en función de dónde se ha producido la transacción, como, por ejemplo, Microsoft Store, la Tienda Windows Phone, la Tienda Windows 8 o publicidad.                  |
| Identificador del pedido                | Identificador único del pedido. Este identificador permite identificar las transacciones de compra con sus transacciones que no son de compra correspondientes, como, por ejemplo, reembolsos o contracargos. El identificador del pedido será el mismo en ambos casos. Además, en el caso de un cargo de división en el que se usaron varios métodos de pago para una sola compra, le permite vincular las transacciones de compra. |
| Transaction ID          | Identificador único de la transacción.                                                                                                                                          |
| Fecha y hora de la transacción   | La fecha y la hora de la transacción (UTC).                                                                                                                       |
| Id. del producto principal       | Identificador único del producto principal. Si no hay un producto principal para la transacción, el identificador del producto principal es igual al identificador del producto.                                |
| Product ID              | Identificador único del producto.                                                                                                                                              |
| Nombre del producto principal     | Nombre del producto principal. Si no hay un producto principal para la transacción, el nombre del producto principal es igual al nombre del producto.                                  |
| Nombre de producto            | Nombre del producto.                                                                                                                                                    |
| Tipo de producto            | Tipo de producto, como, por ejemplo, aplicación, complemento o juego.                                                                                                                       |
| Cantidad                | Cuando el origen de ingresos es Microsoft Store para Empresas, la cantidad representa el número de licencias adquiridas. En el caso del resto de orígenes de ingresos, la cantidad siempre será 1. Incluso cuando una sola transacción se divida en dos elementos de línea al haberse usado dos métodos de pago diferentes, la cantidad de cada elemento de línea será 1. |
| Tipo de transacción        | Tipo de transacción, como, por ejemplo, compra, reembolso, revocación o contracargo.                                                                                              |
| Método de pago          | Instrumento de pago del cliente usado para la transacción, como, por ejemplo, tarjeta, facturación del operador móvil o PayPal.                                                               |
| País o región        | País o región donde se ha producido la transacción.                                                                                                                          |
| Proveedor o vendedor local | Proveedor o vendedor local del registro.                                                                                                                                        |
| Moneda de transacción    | Moneda de la transacción.                                                                                                                                            |
| Importe de transacción      | Importe de la transacción.                                                                                                                                              |
| Impuestos remitidos            | Importe del impuesto remitido (ventas, uso o impuestos de IVA/GST).                                                                                                                  |
| Recibos netos            | Importe de la transacción menos el impuesto remitido.                                                                                                                                   |
| Honorarios de la Tienda               | El porcentaje de recibos netos retenido por Microsoft como honorarios por hacer que la aplicación o el complemento estén disponibles en la Tienda.                                                      |
| Ganancias por la aplicación            | Recibos netos menos los honorarios de la Tienda.                                                                                                                                       |
| Impuestos retenidos          | Importe de los impuestos retenidos (no se incluye en el archivo CSV **Reserved**).                                                                                                |
| Payment                 | Ganancias por la aplicación menos la retención de impuestos aplicable (el importe se muestra en la moneda de la transacción). No se incluye en el archivo CSV **Reserved**.                               |
| Tipo de cambio                 | Tipo de cambio usado para convertir la moneda de la transacción a la moneda del pago.                                                                                         |
| Moneda del pago        | Moneda en la que se realiza el pago.                                                                                                                                       |
| Pago convertido       | Importe del pago convertido a la moneda del pago mediante el tipo de cambio.                                                                                                         |
| Modelo de remisión de impuestos         | Entidad responsable de la emisión de impuestos (ventas, uso o impuestos de IVA/GST).                                                                                                   |
| Fecha y hora de la idoneidad   | La fecha y la hora en que las ganancias por la transacción se vuelven aptas para el pago (UTC). Cuando se crea un pago, incluye ganancias por la transacción con una fecha y hora de la idoneidad anterior a la fecha de creación del pago (solo se incluye en el archivo CSV **Reserved**). |
| Charges                 | Muestra un desglose de todos los detalles de los cargos agregados en la columna Importe de transacción (solo se incluye para Azure Marketplace; no se incluye en el archivo CSV **Reserved**). |
|||

## <a name="next-step"></a>siguiente paso

- [Detalles de directiva de pagos](./payout-policy-details.md)
