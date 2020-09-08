---
title: Capacidades de transacción de marketplace comercial de Microsoft
description: En este artículo se describen las consideraciones de precios, facturación y pago de la opción de transacción de marketplace comercial.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: mingshen
author: mingshen-ms
ms.openlocfilehash: 04a984a2dfa473502fd9e534e52b60b33be52757
ms.sourcegitcommit: e0785ea4f2926f944ff4d65a96cee05b6dcdb792
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88704967"
---
# <a name="commercial-marketplace-transact-capabilities"></a>Capacidades de transacción de marketplace comercial

En este artículo se describen los aspectos que se deben tener en cuenta en relación con los precios, la facturación y el pago de marketplace comercial de Microsoft. 

## <a name="transactions-by-listing-option"></a>Opción de transacciones por publicación

El editor o Microsoft es responsable de administrar las transacciones de licencia de software para las ofertas en marketplace comercial. La opción de publicación que elija para su oferta determinará quién administra la transacción. Consulte [Determinar la opción de publicación](./determine-your-listing-type.md#choose-a-listing-option) para ver la disponibilidad y las explicaciones de cada opción de publicación.

### <a name="contact-me-free-trial-and-byol-options"></a>Opciones Ponerse en contacto conmigo, Evaluación gratuita y BYOL

Los editores pueden elegir las opciones _Ponerse en contacto conmigo_ y _Evaluación gratuita_ con fines promocionales y de captación de usuarios. En el caso de algunos tipos de oferta, los editores pueden elegir la opción "Traiga su propia licencia (BYOL)" para permitir que los clientes compren una suscripción a la oferta mediante una licencia que hayan adquirido directamente. Con estas opciones, Microsoft no participa directamente en las transacciones de licencias de software del publicador y no hay ningún honorario de transacción asociado. 

Los editores son responsables de ayudar en todos los aspectos de la transacción de la licencia de software. Por ejemplo, el pedido, la entrega, la cumplimentación, la medición del uso, la facturación, el pago y el cobro. Con la opción de publicación Ponerse en contacto conmigo, los editores se quedan con el 100 % de las cuotas de licencias de software que se cobran al cliente.

### <a name="transact-publishing-option"></a>Opción de publicación de transacción

La opción de vender a través de Microsoft permite aprovechar las ventajas de las funcionalidades de comercio de Microsoft y proporciona una experiencia completa que se inicia con la detección y la evaluación y finaliza con la compra y la implementación. Una oferta en la que se puede llevar a cabo una transacción es aquella en la que Microsoft facilita el intercambio de dinero por una licencia de software en nombre del editor. Las ofertas en las que se puede llevar a cabo una transacción se facturan en una suscripción de Microsoft existente o una tarjeta de crédito, lo que permite a Microsoft hospedar las transacciones de marketplace en la nube en nombre del editor.

Puede elegir la opción de transacción al crear una nueva oferta en el Centro de partners. Esta opción solo se mostrará si la transacción está disponible para el tipo de oferta.

## <a name="transact-overview"></a>Información general sobre transacciones

Al usar la opción de transacción, Microsoft permite la venta de software de terceros y la implementación de algunos tipos de oferta en la suscripción de Azure del cliente. Al seleccionar un modelo de precio para una oferta, el editor debe tener en cuenta la facturación de los precios de la infraestructura y sus propias cuotas de licencia de software.

La opción de publicación de transacción se admite actualmente para los siguientes tipos de oferta:

- Máquinas virtuales
- Aplicaciones de Azure
- Aplicaciones SaaS

### <a name="billing-infrastructure-costs"></a>Costos de infraestructura de facturación

Para las **máquinas virtuales** y **aplicaciones de Azure**, los precios de uso de la infraestructura de Azure se facturan a la suscripción a Azure del cliente. Los precios de uso de la infraestructura se cotizan y presentan por separado de los precios de licencias del proveedor de software en la factura del cliente.

Para **aplicaciones SaaS**, el publicador debe tener en cuenta los precios de uso de la infraestructura de Azure y los precios de licencias de software como elemento único de costo.  Se representa como un honorario plano para el cliente. La administración y el uso de la infraestructura de Azure se le factura directamente al editor. El cliente no ve los honorarios de uso de infraestructura reales. En general, los publicadores optan por agrupar los honorarios de uso de la infraestructura de Azure en los precios de las licencias de software. Las cuotas de las licencias de software no se basan en el uso medido ni en el consumo del usuario.

## <a name="pricing-models"></a>Modelos de precios

En función de la opción de transacción usada, los cargos de suscripción son los siguientes:

- **Obténgalo ahora (de forma gratuita)** : las licencias de software son gratuitas. Por el uso de una oferta gratuita, no se les cobra a los clientes ninguna cuota de Azure Marketplace. Las ofertas gratuitas no se pueden pasar a ofertas de pago. Los clientes deben solicitar una oferta de pago.
- **Traiga su propia licencia (BYOL)** : los cargos aplicables para las licencias de software se administran directamente entre el publicador y el cliente. Microsoft solo traspasa los honorarios de uso de la infraestructura de Azure. Si se anuncia una oferta en el marketplace comercial, a los clientes que obtienen acceso a la oferta o que la usan fuera del marketplace comercial no se les cobran las cuotas de marketplace comercial.
- **Precios de suscripción**: las cuotas de licencia de software se presentan como cuotas de suscripción periódicas mensuales o anuales que se facturan como tarifa plana o por puesto. Las cuotas de suscripción periódicas no se prorratean por cancelaciones del cliente a mitad del período o por la falta de uso de los servicios. Las cuotas de suscripción periódicas se pueden prorratear si el cliente cambia a una versión superior o inferior de su suscripción a mitad del período.
- **Precios basados en el uso**: en el caso de ofertas de máquinas virtuales de Azure, se factura a los clientes en función del alcance de su uso de la oferta. Si son imágenes de máquina virtual, a los clientes se les cobra una tarifa por hora de Azure Marketplace, establecida por los editores, por el uso de las máquinas virtuales implementadas a partir de las imágenes. La tarifa por horas puede ser uniforme o variar según los distintos tamaños de máquina virtual. Las horas parciales se cobran por minuto. Los planes se facturan mensualmente.
- **Precios de uso medido**: con las ofertas de aplicaciones de Azure y SaaS, los editores pueden usar el [servicio de medición de Marketplace](./partner-center-portal/marketplace-metering-service-apis.md) para facturar el consumo según las dimensiones de los medidores que elijan. Por ejemplo, ancho de banda, vales o correos electrónicos procesados. Los editores pueden definir una o más dimensiones de medidor para cada plan. Ellos son responsables de hacer un seguimiento del uso individual de los clientes, con cada medidor definido en la oferta. Los eventos se deben notificar a Microsoft en el plazo de una hora. Microsoft cobra a los clientes en función de la información de uso notificada por los anunciantes para el período de facturación aplicable.
- **Evaluación gratuita**: no se cobra por las licencias de software que cubren un período de 30 días a seis meses, en función del tipo de oferta. Si los editores proporcionan una evaluación gratuita en varios planes dentro de la misma oferta, los clientes pueden cambiar a una versión de evaluación gratuita de otro plan, pero el período de prueba no se reinicia. En el caso de ofertas de máquina virtual, se cobra a los clientes los costos de la infraestructura de Azure por el uso de la oferta durante un período de prueba. Tras la expiración del período de prueba, se cobra automáticamente a los clientes el último plan que probaron según las tarifas estándar, a menos que lo cancelen antes del final del período de prueba.

> [!NOTE]
> Las ofertas que se facturan según el uso nos on aptas para reembolsos una vez que se ha usado la solución.

Los editores que quieran cambiar las tarifas de uso asociadas a una oferta deben quitar primero la oferta (o el plan específico dentro de esta) del marketplace comercial. La eliminación debe realizarse de acuerdo con los requisitos del [contrato para editores de Microsoft](https://go.microsoft.com/fwlink/?LinkID=699560). Luego, el editor puede publicar una nueva oferta (o plan dentro de una oferta) que incluya las nuevas cuotas de uso. Para información sobre cómo quitar una oferta o plan, consulte [Detención de la venta de una oferta o un plan](./partner-center-portal/update-existing-offer.md#stop-selling-an-offer-or-plan).

### <a name="free-contact-me-and-bring-your-own-license-byol-pricing"></a>Precios de Gratis, Ponerse en contacto conmigo y traiga su propia licencia (BYOL)

Al publicar una oferta con la opción Obténgalo ahora (de forma gratuita), Ponerse en contacto conmigo o BYOL, Microsoft no desempeña ningún papel en la transacción de ventas con las tarifas de licencia de software. Al igual que con las opciones de anuncio y publicación de evaluaciones de prueba, el editor conserva el 100 % de las cuotas de licencias de software.

### <a name="usage-based-and-subscription-pricing"></a>Precios de suscripción y basados en el uso

Al publicar una oferta basada en el uso o una transacción de una suscripción, Microsoft proporciona la tecnología y los servicios para procesar las compras, las devoluciones y los contracargos de las licencias de software. En este escenario, el publicador autoriza a Microsoft a actuar como agente para estos fines. El publicador permite a Microsoft facilitar la transacción con las licencias de software, a la vez que conserva su designación como vendedor, proveedor, distribuidor y el emisor de licencias.

Microsoft permite a los clientes pedir, licenciar y usar el su software, sujeto a los términos y condiciones de marketplace comercial de Microsoft y de su contrato de licencia para el usuario final. Debe proporcionar su propio contrato de licencia para el usuario final o seleccionar el [Contrato estándar](./standard-contract.md) al crear la oferta.

### <a name="free-software-trials"></a>Evaluaciones gratuitas del software

En escenarios de publicación con transacciones, puede hacer que una licencia de software esté disponible de forma gratuita entre 30 y 120 días, en función de la suscripción. Esta funcionalidad de descuento no incluye el costo de uso de la infraestructura de Azure controlada mediante el uso de la solución de asociado.

### <a name="private-offers"></a>Ofertas privadas

Además de usar tipos de ofertas y modelos de facturación para monetizar una oferta, puede comercializar una oferta privada, con precios negociados específicos para el negocio, o bien con configuraciones personalizadas. Las ofertas privadas se admiten en las tres opciones de publicación de transacciones.

Esta opción permite que los precios sean mayores o menores que la oferta disponible públicamente. Las ofertas privadas pueden usarse para descontar o agregar una prima a una oferta. Las ofertas privadas pueden estar disponibles para uno o más clientes mediante la inclusión en una lista blanca de su suscripción de Azure en el nivel de oferta.

### <a name="examples"></a>Ejemplos

**Basado en uso** 

Los precios basados en el uso tienen la siguiente estructura de costos:

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

En este escenario, Microsoft factura 100,00 USD por su licencia de software y paga 90,00 USD al anunciante:

|Microsoft factura  | 100,00 USD al mes  |
|---------|---------|
|Microsoft le paga el 80 % del costo de licencia <br> \* Microsoft le paga el 90 % del costo de licencia de las aplicaciones SaaS aptas.   |   80,00 USD al mes <br> \* 90,00 USD al mes    |
|Microsoft se queda con el 20 % del costo de licencia <br> \* Microsoft se queda con el 10 % del costo de licencia de las aplicaciones SaaS aptas.  |  20,00 USD al mes <br> \* 10,00 USD     |

Para determinadas ofertas que se publican en el marketplace comercial, Microsoft reducirá sus cuotas del servicio Marketplace del 20 % (tal como se describe en el [contrato para editores de Microsoft](https://go.microsoft.com/fwlink/?LinkID=699560)) al 10 %. Para que su oferta sea apta, Microsoft debe haber designado sus ofertas como incentivadas para realizar una venta conjunta por IP de Azure. Para recibir el precio reducido del servicio Marketplace durante el mes, se deben cumplir los criterios de idoneidad al menos cinco días hábiles antes del final de cada mes natural. La cuota reducida del servicio Marketplace se aplica a SaaS, máquinas virtuales y aplicaciones administradas con incentivos por venta conjunta por IP de Azure, así como a cualquier otra oferta de IaaS en la que se pueda llevar a cabo una transacción cualificada que esté disponible a través del marketplace comercial.

### <a name="customer-invoicing-payment-billing-and-collections"></a>Facturación, pago y cobro al cliente

**Facturación y pago**: el publicador puede usar el método de facturación preferido del cliente para comunicar los precios de las licencias del software de suscripción o de pago por uso.

**Contrato Enterprise**: si el método de facturación preferido del cliente es el contrato Enterprise de Microsoft, los honorarios de las licencias de software se facturarán mediante este método de facturación como un costo detallado, independiente de los costos de uso específicos de Azure.

**Tarjetas de crédito y factura mensual**: los clientes también pueden pagar con tarjeta de crédito y una factura mensual. En este caso, los precios de las licencias de software se facturarán igual que en el escenario de contrato Enterprise, como un costo detallado, independiente de los costos de uso específicos de Azure.

**Créditos gratuitos y compromiso monetario**: algunos clientes eligen pagar Azure por adelantado con un compromiso monetario en el contrato Enterprise, o han recibido créditos gratuitos para su uso con Azure. Aunque estos créditos se pueden usar para pagar por el uso de Azure, no se puede usar para pagar los honorarios de las licencias de software del publicador.

**Facturación y cobro**: la facturación de las licencias de software del publicador se presenta mediante el método de facturación seleccionado por el cliente y sigue la escala de tiempo de facturación. A los clientes sin un contrato Enterprise vigente se les facturan mensualmente las licencias de software de Marketplace. A los clientes con un contrato Enterprise se les factura mensualmente a través de una factura que se presenta cada trimestre.

Cuando se han seleccionado los modelos de precios de suscripción o de pago por uso, Microsoft actúa como agente del publicador y es responsable de todos los aspectos de facturación, pago y cobro.

### <a name="publisher-payout-and-reporting"></a>Pagos e informes al publicador

Los honorarios de las licencias de software cobrados por Microsoft como agente están sujetos a una tarifa de transacción del 20 %, a menos que se especifique lo contrario, y se deducen en el momento del pago al publicador.

Normalmente, los clientes compran con contrato Enterprise o mediante contrato de pago por uso con tarjeta de crédito. El tipo de contrato determina los plazos de facturación, cobro y pago.

>[!NOTE]
>Todos los informes y la información para la opción de publicación de la transacción están disponibles a través de la sección Análisis del Centro de partners.

#### <a name="billing-questions-and-support"></a>Soporte técnico y preguntas sobre facturación

Para más información y conocer las directivas legales, consulte el [contrato para editores de Microsoft](https://go.microsoft.com/fwlink/?LinkID=699560) (disponible en el Centro de partners).

Para obtener ayuda si tiene preguntas sobre facturación, póngase en contacto con el [equipo de soporte técnico para publicadores de Marketplace comercial](https://aka.ms/marketplacepublishersupport).

## <a name="transact-requirements"></a>Requisitos de las transacciones

En esta sección se tratan los requisitos de transacción para distintos tipos de oferta.

### <a name="requirements-for-all-offer-types"></a>Requisitos para todos los tipos de ofertas

- Se necesitan una cuenta de Microsoft y la información financiera para la opción de publicación de transacciones, independientemente del modelo de precios de la oferta.
- Entre la información financiera obligatoria se incluyen el perfil fiscal y la cuenta de pago.

Para más información sobre cómo configurar estas cuentas, consulte [Administración de la cuenta de marketplace comercial en el Centro de partners](partner-center-portal/manage-account.md).

### <a name="requirements-for-specific-offer-types"></a>Requisitos para tipos de ofertas específicos

La opción de publicación de transacciones solo está disponible para su uso con los siguientes tipos de oferta de Marketplace:

- **Máquina virtual de Azure**: seleccione entre los modelos de precios gratuito, traiga su propia licencia o pago por uso, y presente la elección como planes definidos en el nivel de oferta. En la factura de Azure para el cliente, Microsoft presenta los precios de las licencias de software del publicador por separado de los precios de la infraestructura de Azure subyacente. Los precios de la infraestructura de Azure corresponden al uso del software del publicador.

- **Aplicación de Azure: plantilla de soluciones o aplicación administrada**: debe aprovisionar una o varias máquinas virtuales y extrae la suma de los precios de las máquinas virtuales. Para aplicaciones administradas en un único plan, se puede seleccionar una suscripción mensual con tarifa plana como modelo de precios, en su lugar del precio de las máquinas virtuales. En algunos casos, el precio por uso de la infraestructura de Azure se traslada al cliente por separado de los precios de las licencias de software, pero en el mismo extracto de facturación. En cambio, si configura una oferta de aplicación administrada para los cargos por infraestructura de ISV, los recursos de Azure se facturan al editor y el cliente recibe una tarifa plana que incluye el costo de la infraestructura, las licencias de software y los servicios de administración.

- **Aplicación SaaS**: debe ser una solución para varios inquilinos, utilice [Azure Active Directory](https://azure.microsoft.com/services/active-directory/) para la autenticación e intégrela con las [API de cumplimiento de SaaS](partner-center-portal/pc-saas-fulfillment-api-v2.md). Se administra y se factura directamente al asociado por el uso de la infraestructura de Azure, por lo que se debe tener en cuenta las tarifas de uso de la infraestructura de Azure y las tarifas de las licencias de software como un único elemento de costo. Para instrucciones detalladas, consulte [Creación de una nueva oferta de SaaS en marketplace comercial](partner-center-portal/create-new-saas-offer.md).

## <a name="next-steps"></a>Pasos siguientes

- Revise los requisitos de idoneidad de las opciones de publicación por tipo de oferta para finalizar la selección y la configuración de la oferta.
- Revise los patrones de publicación por tienda en línea para ver ejemplos de cómo la solución se asigna a un tipo de oferta y configuración.
