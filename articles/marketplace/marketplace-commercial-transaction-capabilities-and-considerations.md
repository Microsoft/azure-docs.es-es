---
title: Capacidades de transacción de marketplace comercial de Microsoft
description: En este artículo se describen las consideraciones de precios, facturación y pago de la opción de transacción de marketplace comercial.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 07/22/2020
ms.author: mingshen
author: mingshen-ms
ms.openlocfilehash: e2ce35158e8f9a3a2ad9da2b3d67a3f35d5a8c80
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503572"
---
# <a name="commercial-marketplace-transact-capabilities"></a>Capacidades de transacción de marketplace comercial

## <a name="transactions-by-publishing-option"></a>Transacciones por opción de publicación

El editor o Microsoft es responsable de administrar las transacciones de licencia de software para las ofertas en marketplace comercial. La opción de publicación que elija para su oferta determinará quién administra la transacción. Vea [Determinación de la opción de publicación](./determine-your-listing-type.md#choose-a-call-to-action) para ver la disponibilidad y las explicaciones de cada opción de publicación.

### <a name="list-trial-and-byol-publishing-options"></a>Opciones de publicación de listas, pruebas y BYOL

Los editores con capacidades de comercio pueden elegir las opciones de publicación de listas, de pruebas y de traiga su propia licencia (BYOL) con fines promocionales y de adquisición por parte de usuarios. Con estas opciones, Microsoft no participa directamente en las transacciones de licencias de software del publicador y no hay ningún honorario de transacción asociado. Los publicadores son responsables de todos los aspectos de la transacción de las licencias de software, incluidas, entre otras cosas, el pedido, la entrega, la medición, la facturación, el pago y el cobro. Con las opciones de publicación de listas y pruebas, los publicadores mantienen el 100 % de los honorarios de licencias de software del publicador que se cobran al cliente.

### <a name="transact-publishing-option"></a>Opción de publicación de transacción

La opción de publicación de transacción aprovecha las funcionalidades de Microsoft Commerce y proporciona una experiencia completa desde la detección y evaluación hasta la compra e implementación. Las ofertas de transacción se facturan a una suscripción de Microsoft existente o a una tarjeta de crédito, lo que permite a Microsoft hospedar las transacciones de marketplace en la nube en nombre del editor.

Puede elegir la opción de transacción al crear una nueva oferta en el Centro de partners. En la página **Configuración de la oferta**, en **Detalles de configuración**, seleccione "Sí, me gustaría vender a través de Microsoft y que Microsoft hospede transacciones en mi nombre". Esta opción solo se mostrará si la transacción está disponible para el tipo de oferta.

## <a name="transact-overview"></a>Información general sobre transacciones

Al usar la opción de publicación de transacción, Microsoft permite la venta del software de terceros y la implementación de algunos tipos de oferta en la suscripción a Azure del cliente. El publicador debe tener en cuenta la facturación de los honorarios de infraestructura y sus propios precios de licencias de software al seleccionar un modelo de facturación y un tipo de oferta.

La opción de publicación de transacción se admite actualmente para los siguientes tipos de oferta:

- Máquinas virtuales
- Aplicaciones de Azure
- Aplicaciones SaaS

### <a name="billing-infrastructure-costs"></a>Costos de infraestructura de facturación

Para las **máquinas virtuales** y **aplicaciones de Azure**, los precios de uso de la infraestructura de Azure se facturan a la suscripción a Azure del cliente. Los precios de uso de la infraestructura se cotizan y presentan por separado de los precios de licencias del proveedor de software en la factura del cliente.

Para **aplicaciones SaaS**, el publicador debe tener en cuenta los precios de uso de la infraestructura de Azure y los precios de licencias de software como elemento único de costo.  Se representa como un honorario plano para el cliente. El asociado administra y factura directamente el uso de la infraestructura de Azure. El cliente no ve los honorarios de uso de infraestructura reales. En general, los publicadores optan por agrupar los honorarios de uso de la infraestructura de Azure en los precios de las licencias de software. Los precios de las licencias de software no se basan en mediciones ni en consumo.

## <a name="transact-billing-models"></a>Modelos de facturación de las transacciones

Según la opción de transacción usada, los precios de las licencias de software son los siguientes:

- **Gratis**: sin cargo por las licencias de software.
- **Traiga su propia licencia (BYOL)** : los cargos aplicables para las licencias de software se administran directamente entre el publicador y el cliente. Microsoft solo traspasa los honorarios de uso de la infraestructura de Azure. Esto solo se aplica a máquinas virtuales y aplicaciones de Azure.
- **Pago por uso**: los precios de las licencia de software se presentan a una tarifa por hora, por núcleo (vCPU) según la infraestructura de Azure usada. Esto solo se aplica a máquinas virtuales y aplicaciones de Azure.
- **Precios de suscripción**: las cuotas de licencia de software se presentan como cuotas periódicas mensuales o anuales que se facturan como una tarifa plana o por puesto. Esto solo se aplica a aplicaciones SaaS (al mes o al año) y aplicaciones de Azure: aplicaciones administradas (al mes).
- **Prueba de software gratuita**: ningún cargo por las licencias de software durante treinta o noventa días.

### <a name="free-and-bring-your-own-license-byol-pricing"></a>Gratis y precios de traiga su propia licencia (BYOL)

Al publicar una oferta de transacción gratuita o de traiga su propia licencia, Microsoft no participa en facilitar la transacción de venta para los precios de las licencias de software. Al igual que con las opciones de publicación de listas y pruebas, el publicador conserva el 100 % de los honorarios de las licencias de software.

### <a name="pay-as-you-go-and-subscription-site-based-pricing"></a>Precios de pago por uso y de suscripción (basados en el sitio)

Al publicar una oferta de transacción de pago por uso o de suscripción, Microsoft proporciona la tecnología y los servicios para procesar las compras, las devoluciones y los contracargos de las licencias de software. En este escenario, el publicador autoriza a Microsoft a actuar como agente para estos fines. El publicador permite a Microsoft facilitar la transacción con las licencias de software, a la vez que conserva su designación como vendedor, proveedor, distribuidor y el emisor de licencias.

Microsoft permite a los clientes pedir, licenciar y usar el su software, sujeto a los términos y condiciones de marketplace comercial de Microsoft y de su contrato de licencia para el usuario final. Debe proporcionar su propio contrato de licencia para el usuario final o seleccionar el [Contrato estándar](./standard-contract.md) al crear la oferta.

### <a name="free-software-trials"></a>Evaluaciones gratuitas del software

Para los escenarios de publicación de transacciones, puede poner una licencia de software a disposición de manera gratuita durante treinta o noventa días. Esta funcionalidad de descuento no incluye el costo de uso de la infraestructura de Azure controlada mediante el uso de la solución de asociado.

### <a name="private-offers"></a>Ofertas privadas

Además de usar tipos de ofertas y modelos de facturación para monetizar una oferta, puede comercializar una oferta privada, con precios negociados específicos para el negocio, o bien con configuraciones personalizadas. Las ofertas privadas se admiten en las tres opciones de publicación de transacciones.

Esta opción permite que los precios sean mayores o menores que la oferta disponible públicamente. Las ofertas privadas pueden usarse para descontar o agregar una prima a una oferta. Las ofertas privadas pueden estar disponibles para uno o más clientes mediante la inclusión en una lista blanca de su suscripción de Azure en el nivel de oferta.

### <a name="examples"></a>Ejemplos

**Pay-As-You-Go** 

El plan Pago por uso tiene la siguiente estructura de costos:

|Costo de licencia  | 1,00 USD por hora   |
|---------|---------|
|Costo de uso de Azure (D1/1 núcleo)    |   0,14 USD por hora     |
|*Microsoft factura al cliente*    |  *1,14 USD por hora*       |
||

En este escenario, Microsoft factura 1,14 USD por hora de uso de la imagen de máquina virtual publicada.

|Microsoft factura  | 1,14 USD por hora  |
|---------|---------|
|Microsoft le paga el 80 % del costo de licencia|   0,80 USD por hora     |
|Microsoft se queda con el 20 % del costo de licencia  |  0,20 USD por hora       |
|Microsoft se queda con el 100 % costo de uso de Azure | 0,14 USD por hora |
||

**Traiga su propia licencia (BYOL)**

BYOL tiene la siguiente estructura de costos:

|Costo de licencia  | Usted negocia y factura el precio de licencia  |
|---------|---------|
|Costo de uso de Azure (D1/1 núcleo)    |   0,14 USD por hora     |
|*Microsoft factura al cliente*    |  *0,14 USD por hora*       |
||

En este escenario, Microsoft factura 0,14 USD por hora de uso de la imagen de máquina virtual publicada.

|Microsoft factura  | 0,14 USD por hora  |
|---------|---------|
|Microsoft se queda con el costo de uso de Azure    |   0,14 USD por hora     |
|Microsoft se queda con 0 % del costo de licencia   |  0,00 USD por hora       |
||

**Suscripción de aplicación SaaS**

Esta opción debe configurarse para vender a través de Microsoft y puede tener un precio con una tarifa plana o por usuario, de forma mensual o anual. Si habilita la opción de **venta mediante Microsoft** de una oferta de SaaS, obtendrá la siguiente estructura de costos:

| Costo de licencia       | 100,00 USD al mes  |
|--------------|---------|
| Costo de uso de Azure (D1/1 núcleo)    | Facturado directamente al publicador, no al cliente |
| *Microsoft factura al cliente*    |  *100,00 USD al mes (el anunciante debe contar con los costos de paso a través o en que se incurran en el precio de la licencia)*  |
||

En este escenario, Microsoft factura 100,00 USD por su licencia de software y paga 80,00 USD al publicador.

Los asociados que sean aptos para la tarifa reducida del servicio Marketplace verán una cuota de transacción reducida en las ofertas de SaaS desde mayo de 2019 hasta junio de 2020.

En este escenario, Microsoft factura 100,00 USD por su licencia de software y paga 90,00 USD al anunciante:

|Microsoft factura  | 100,00 USD al mes  |
|---------|---------|
|Microsoft le paga el 80 % del costo de licencia <br> \* Microsoft le paga el 90 % del costo de licencia de las aplicaciones SaaS aptas.   |   80,00 USD al mes <br> \* 90,00 USD al mes    |
|Microsoft se queda con el 20 % del costo de licencia <br> \* Microsoft se queda con el 10 % del costo de licencia de las aplicaciones SaaS aptas.  |  20,00 USD al mes <br> \* 10,00 USD     |

Para determinadas ofertas que publica en nuestro marketplace comercial, Microsoft reducirá sus honorarios del servicio Marketplace del 20 % (tal como se describe en el contrato para editores de Microsoft) al 10 %. Para que su oferta sea apta, Microsoft debe haber designado sus ofertas como incentivadas para realizar una venta conjunta por IP de Azure. Para recibir el precio reducido del servicio Marketplace durante el mes, se deben cumplir los criterios de idoneidad al menos cinco (5) días hábiles antes del final de cada mes natural. Los honorarios del servicio Marketplace reducidos se aplican a SaaS, máquinas virtuales y aplicaciones administradas con incentivos para realizar una venta conjunta por IP de Azure, así como a cualquier otra oferta de IaaS comercializable cualificada que esté disponible a través del marketplace comercial.

### <a name="customer-invoicing-payment-billing-and-collections"></a>Facturación, pago y cobro al cliente

**Facturación y pago**: el publicador puede usar el método de facturación preferido del cliente para comunicar los precios de las licencias del software de suscripción o de pago por uso.

**Contrato Enterprise**: si el método de facturación preferido del cliente es el contrato Enterprise de Microsoft, los honorarios de las licencias de software se facturarán mediante este método de facturación como un costo detallado, independiente de los costos de uso específicos de Azure.

**Tarjetas de crédito y factura mensual**: los clientes también pueden pagar con tarjeta de crédito y una factura mensual. En este caso, los precios de las licencias de software se facturarán igual que en el escenario de contrato Enterprise, como un costo detallado, independiente de los costos de uso específicos de Azure.

**Créditos gratuitos y compromiso monetario**: algunos clientes eligen pagar Azure por adelantado con un compromiso monetario en el contrato Enterprise, o han recibido créditos gratuitos para su uso con Azure. Aunque estos créditos se pueden usar para pagar por el uso de Azure, no se puede usar para pagar los honorarios de las licencias de software del publicador.

**Facturación y cobro**: la facturación de las licencias de software del publicador se presenta mediante el método de facturación seleccionado por el cliente y sigue el cronograma de facturación. A los clientes sin un contrato Enterprise vigente se les facturan mensualmente las licencias de software de Marketplace. A los clientes con un contrato Enterprise se les factura mensualmente a través de una factura que se presenta cada trimestre.

Cuando se han seleccionado los modelos de precios de suscripción o de pago por uso, Microsoft actúa como agente del publicador y es responsable de todos los aspectos de facturación, pago y cobro.

### <a name="publisher-payout-and-reporting"></a>Pagos e informes al publicador

Los honorarios de las licencias de software cobrados por Microsoft como agente están sujetos a una tarifa de transacción del 20 %, a menos que se especifique lo contrario, y se deducen en el momento del pago al publicador.

Normalmente, los clientes compran con contrato Enterprise o mediante contrato de pago por uso con tarjeta de crédito. El tipo de contrato determina los plazos de facturación, cobro y pago.

>[!NOTE]
>Todos los informes y la información para la opción de publicación de la transacción están disponibles a través de la sección Análisis del Centro de partners.

#### <a name="billing-questions-and-support"></a>Soporte técnico y preguntas sobre facturación

Para obtener más información y las directivas legales, consulte el [contrato del publicador](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RE4xqkx) (disponible en el Centro de partners).

Para obtener ayuda si tiene preguntas sobre facturación, póngase en contacto con el [equipo de soporte técnico para publicadores de Marketplace comercial](https://aka.ms/marketplacepublishersupport).

## <a name="transact-requirements"></a>Requisitos de las transacciones

En esta sección se tratan los requisitos de transacción para distintos tipos de oferta.

### <a name="requirements-for-all-offer-types"></a>Requisitos para todos los tipos de ofertas

- Se necesitan una cuenta de Microsoft y la información financiera para la opción de publicación de transacciones, independientemente del modelo de precios de la oferta.
- Entre la información financiera obligatoria se incluyen el perfil fiscal y la cuenta de pago.

Para más información sobre cómo configurar estas cuentas, consulte [Administración de la cuenta de marketplace comercial en el Centro de partners](partner-center-portal/manage-account.md).

### <a name="requirements-for-specific-offer-types"></a>Requisitos para tipos de ofertas específicos

La opción de publicación de transacciones solo está disponible para su uso con los siguientes tipos de oferta de Marketplace:

- **Máquina virtual**: seleccione entre los modelos de precios gratuito, traiga su propia licencia o pago por uso, y presente la elección como planes definidos en el nivel de oferta. En la factura de Azure para el cliente, Microsoft presenta los precios de las licencias de software del publicador por separado de los precios de la infraestructura de Azure subyacente. Los precios de la infraestructura de Azure corresponden al uso del software del publicador.

- **Aplicación de Azure: plantilla de soluciones o aplicación administrada**: debe aprovisionar una o varias máquinas virtuales y extrae la suma de los precios de las máquinas virtuales. Para aplicaciones administradas en un único plan, se puede seleccionar una suscripción mensual con tarifa plana como modelo de precios, en su lugar del precio de las máquinas virtuales. En algunos casos, el precio por uso de la infraestructura de Azure se traslada al cliente por separado de los precios de las licencias de software, pero en el mismo extracto de facturación. En cambio, si configura una oferta de aplicación administrada para los cargos por infraestructura de ISV, los recursos de Azure se facturan al editor y el cliente recibe una tarifa plana que incluye el costo de la infraestructura, las licencias de software y los servicios de administración.

- **Aplicación SaaS**: debe ser una solución para varios inquilinos, utilice [Azure Active Directory](https://azure.microsoft.com/services/active-directory/) para la autenticación e intégrela con las [API de cumplimiento de SaaS](partner-center-portal/pc-saas-fulfillment-api-v2.md). Se administra y se factura directamente al asociado por el uso de la infraestructura de Azure, por lo que se debe tener en cuenta las tarifas de uso de la infraestructura de Azure y las tarifas de las licencias de software como un único elemento de costo. Para instrucciones detalladas, consulte [Creación de una nueva oferta de SaaS en marketplace comercial](partner-center-portal/create-new-saas-offer.md).

## <a name="next-steps"></a>Pasos siguientes

- Revise los requisitos de idoneidad de las opciones de publicación por tipo de oferta para finalizar la selección y la configuración de la oferta.
- Revise los patrones de publicación por escaparate para ver ejemplos de cómo la solución se asigna a un tipo de oferta y configuración.
