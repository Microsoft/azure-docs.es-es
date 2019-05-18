---
title: 'Marketplace: ‎Funcionalidades y consideraciones sobre transacciones comerciales | Azure'
description: En este artículo se describen las consideraciones de precios, facturación y pago de transacciones para un tipo de oferta.
services: Azure, Marketplace, Compute, Storage, Networking, Transact Offer Type
author: yijenj
manager: nuno costa
ms.service: marketplace
ms.topic: article
ms.date: 10/29/2018
ms.author: yijenj
ms.openlocfilehash: 5dc311538055d9ecb7aba21f95d819a914cd1db7
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65826568"
---
# <a name="azure-marketplace-commercial-transaction-capabilities-and-considerations"></a>Azure Marketplace: ‎Funcionalidades y consideraciones sobre transacciones comerciales

Las opciones de publicación de Azure Marketplace ofrecen maneras exclusivas de conectar proveedores de software y servicios en la nube con los clientes. En este artículo se tratan los siguientes temas relacionados con el comercio en Azure Marketplace:

* Opciones de publicación de Marketplace
* Información general sobre las transacciones
* Modelos de facturación de las transacciones
* Requisitos de las transacciones

## <a name="marketplace-publishing-options"></a>Opciones de publicación de Marketplace

Las opciones de publicación siguientes están disponibles para los publicadores de Azure Marketplace.

### <a name="list--trial-publishing-options"></a>Opciones de publicación de listas y pruebas

En Azure Marketplace, los publicadores pueden aprovechar las opciones de publicación de listas y pruebas con fines promocionales y de adquisición por parte de usuarios. Con las opciones de publicación de listas y pruebas, Microsoft no participa directamente en las transacciones de licencias de software del publicador, y no hay ningún honorario de transacción asociado. Los publicadores son responsables de todos los aspectos de la transacción de las licencias de software, incluidas, entre otras cosas: el pedido, la entrega, la medición, la facturación, el pago y el cobro. Con las opciones de publicación de listas y pruebas, los publicadores mantienen el 100 % de los honorarios de licencias de software del publicador que se cobran al cliente. 

### <a name="transact-publishing-option"></a>Opción de publicación de transacción

Además de las opciones de publicación de listas y pruebas, la opción de publicación de transacción está disponible para los publicadores de Azure Marketplace.   Aprovecha las funcionalidades comerciales de Microsoft disponibles globalmente. Esta opción permite a Microsoft hospedar transacciones de Marketplace en la nube en nombre del publicador.

## <a name="transact-general-overview"></a>Información general sobre las transacciones

Cuando se usa la opción de publicación de transacción, Microsoft habilita la venta de software de terceros y la implementación de algunos tipos de oferta de suscripción de Azure del cliente. El publicador debe tener en cuenta la facturación de los honorarios de infraestructura de Azure y los precios de las licencias de software propias del publicador al seleccionar un modelo de facturación y un tipo de oferta en Azure Marketplace.

La opción de publicación de transacción en Azure Marketplace se admite actualmente para los siguientes tipos de oferta: máquinas virtuales, aplicaciones de Azure o aplicaciones SaaS.

![[Ofertas Enterprise de transacción en Azure Marketplace]](./media/marketplace-publishers-guide/Transact-enterprise-deals.png)

### <a name="billing-infrastructure-costs"></a>Costos de infraestructura de facturación

**Para máquinas virtuales y las aplicaciones de Azure**

Para las máquinas virtuales y aplicaciones de Azure, los precios de uso de la infraestructura de Azure se facturan a la suscripción de Azure del cliente.  Los precios de uso de la infraestructura se cotizan y presentan por separado de los precios de licencias del proveedor de software en la factura del cliente.

**Para aplicaciones SaaS**

Para aplicaciones SaaS, el publicador debe tener en cuenta los precios de uso de la infraestructura de Azure y los precios de licencias de software como elemento único de costo.  Se representa como una cuota fija para el cliente. El asociado administra y factura directamente el uso de la infraestructura de Azure.  El cliente no ve los honorarios de uso de infraestructura reales.  En general, los publicadores optan por agrupar los honorarios de uso de la infraestructura de Azure en los precios de las licencias de software.  Los precios de las licencias de software no se basan en mediciones ni en consumo.

## <a name="transact-billing-models"></a>Modelos de facturación de las transacciones

Según la opción de transacción usada, los precios de las licencias de software del publicador se pueden presentar como sigue:  

* Gratis: sin cargo por las licencias de software. 

* Traiga su propia licencia (BYOL): los cargos aplicables para las licencias de software se administran directamente entre el publicador y el cliente. Microsoft solo traspasa los honorarios de uso de la infraestructura de Azure. (Solo para máquinas virtuales y aplicaciones de Azure).

* Pago por uso: los precios de las licencia de software se presentan a una tarifa por hora, por núcleo (vCPU) según la infraestructura de Azure usada. Esta lista solo se aplica a máquinas virtuales y aplicaciones de Azure.

* Precios de suscripción (basados en el sitio): los precios de las licencias de software se presentan como una cuota mensual, periódica.  Esto solo se aplica a aplicaciones SaaS y aplicaciones de Azure: aplicaciones administradas.

* Prueba de software gratuita: ningún cargo por las licencias de software durante 30 o 90 días.

### <a name="free-and-bring-your-own-license-byol-pricing"></a>Gratis y precios de traiga su propia licencia (BYOL)

Al publicar una oferta de transacción gratuita o de traiga su propia licencia, Microsoft no participa en facilitar la transacción de venta para los precios de las licencias de software. Al igual que con las opciones de publicación de listas y pruebas, el publicador conserva el 100 % de los honorarios de las licencias de software. 

### <a name="pay-as-you-go-and-subscription-site-based-pricing"></a>Precios de pago por uso y de suscripción (basados en el sitio)

Al publicar una oferta de transacción de pago por uso o de suscripción, Microsoft proporciona la tecnología y los servicios para procesar las compras, las devoluciones y los contracargos de las licencias de software. En este escenario, el publicador autoriza a Microsoft a actuar como agente para estos fines. El publicador permite a Microsoft facilitar la transacción con las licencias de software, a la vez que conserva su designación como vendedor, proveedor, distribuidor y el emisor de licencias.

Microsoft permite a los clientes pedir, licenciar y usar el software del publicador, sujeto a los términos y condiciones de Azure Marketplace y del contrato de licencia del usuario final del publicador (visite Cloud Partner Portal). Los publicadores deben proporcionar su contrato de licencia del usuario final en la oferta de Marketplace.

Los pedidos procesados a través de Marketplace se facturan en la suscripción de Azure del cliente en una única factura, con el mismo método de facturación que los costos de infraestructura de Azure del cliente. Los clientes pueden usar el método de facturación y el instrumento de pago preferidos usados para la facturación de su suscripción de Azure.

### <a name="free-software-trials"></a>Evaluaciones gratuitas del software

Para los escenarios de publicación de transacciones, el publicador puede poner una licencia de software a disposición de manera gratuita durante 30 o 90 días. Esta funcionalidad de descuento no incluye el costo de uso de la infraestructura de Azure que se controla mediante el uso de la solución de asociado.

### <a name="private-offers"></a>Ofertas privadas

Además de usar tipos de ofertas y modelos de facturación para monetizar una oferta, los publicadores pueden comercializar una versión privada de la oferta de la solución, con precios negociados específicos para el negocio, y configuraciones personalizadas con una imagen personalizada. Las ofertas privadas se admiten en las tres opciones de publicación de transacciones.

Esta opción de precios puede ser mayor o menor que los precios mostrados públicamente.  Las ofertas privadas pueden usarse para descontar o agregar una prima por una oferta. Las ofertas privadas pueden estar disponibles para uno o más clientes mediante la inclusión en una lista blanca de su suscripción de Azure en el nivel de oferta.

### <a name="examples"></a>Ejemplos

**Pay-As-You-Go** 

* Si habilita la opción de Pago por uso, obtendrá la siguiente estructura de costos.

|Costo de licencia  | 1,00 USD por hora  |
|---------|---------|
|Costo de uso de Azure (D1/1 núcleo)    |   0,14 USD por hora     |
|*Microsoft factura al cliente*    |  *1,14 USD por hora*       |

* En este escenario, Microsoft factura 1,14 USD por hora de uso de la imagen de máquina virtual publicada.

|Microsoft factura  | 1,14 USD por hora  |
|---------|---------|
|Microsoft le paga el 80 % del costo de licencia|   0,80 USD por hora     |
|Microsoft se queda con el 20 % del costo de licencia  |  0,20 USD por hora       |
|Microsoft se queda con el 100 % costo de uso de Azure | 0,14 USD por hora |

**Traiga su propia licencia (BYOL)**

* Si habilita la opción BYOL, obtendrá la siguiente estructura de costos.

|Costo de licencia  | Usted negocia y factura el precio de licencia  |
|---------|---------|
|Costo de uso de Azure (D1/1 núcleo)    |   0,14 USD por hora     |
|*Microsoft factura al cliente*    |  *0,14 USD por hora*       |

* En este escenario, Microsoft factura 0,14 USD por hora de uso de la imagen de máquina virtual publicada.

|Microsoft factura  | 0,14 USD por hora  |
|---------|---------|
|Microsoft se queda con el costo de uso de Azure    |   0,14 USD por hora     |
|Microsoft se queda con 0 % del costo de licencia   |  0,00 USD por hora       |

**Suscripción de aplicación SaaS (venta a través de Azure)**

Esta opción debe configurarse para vender a través de Microsoft y puede cotizarse mediante uno o más planes mensuales de tarifa plana definidos en el nivel de oferta.

* Si habilita la opción de venta a través de Azure, obtendrá la siguiente estructura de costos.

|Costo de licencia       | 100,00 USD al mes  |
|--------------|---------|
|Costo de uso de Azure (D1/1 núcleo)    | Facturado directamente al publicador, no al cliente |
|*Microsoft factura al cliente*    |  *100,00 USD al mes (nota: el publicador debe tener en cuenta en el precio de las licencias los costos de infraestructura en los que se incurra o traspasados)*  |

* En este escenario, Microsoft factura 100,00 USD por su licencia de software y paga 80,00 USD al publicador.
* Los asociados que se han calificado para la tarifa del servicio Marketplace reduce verán que ofrece una tarifa reducida de transacciones en el SaaS desde mayo de 2019 hasta junio de 2020. En este escenario, Microsoft factura 100,00 USD por su licencia de software y su recompensa 90,00 USD al publicador.

|Microsoft factura  | 100,00 USD al mes  |
|---------|---------|
|Microsoft le paga el 80 % del costo de licencia <br> \* Microsoft le paga el 90% del costo de licencia para las aplicaciones de SaaS completas   |   80,00 USD al mes <br> \* 90,00 USD al mes    |
|Microsoft se queda con el 20 % del costo de licencia <br> \* Microsoft mantiene el 10% del costo de licencia para las aplicaciones de SaaS completos.  |  20,00 USD al mes <br> \* $10.00     |

* **Tarifa del servicio Marketplace reducida:** Para determinados productos SaaS que se publican en nuestro Marketplace comercial, Microsoft reducirá la tarifa del servicio Marketplace del 20% (como se describe en el contrato del anunciante de Microsoft) en un 10%.  En el orden del producto calificar, al menos uno de sus productos debe designarse por Microsoft como IP de venta colaborativa listo o prioridades de venta conjunta de IP. Para recibir esta tarifa reducida del servicio Marketplace del mes, elegibilidad se debe cumplir al menos cinco (5) días hábiles antes del final de dicho mes natural. Reduce la cuota no se aplicará a las máquinas virtuales, las aplicaciones administradas o cualquier otro producto están disponible a través de nuestro Marketplace comercial de servicio de Marketplace.  Esta tarifa del servicio Marketplace reduce estará disponible para las ofertas de completas, con los cargos de licencia recopilados por Microsoft entre el 31 de mayo de 2019 y el 30 de junio de 2020.  Después de ese momento, la tarifa del servicio Marketplace volverá a su importe normal.

### <a name="customer-invoicing-payment-billing-and-collections"></a>Facturación, pago y cobro al cliente

**Facturación y pago**

El publicador puede usar el método de facturación preferido del cliente para comunicar los precios de las licencias del software de suscripción o de pago por uso.

**Contrato Enterprise** 

Si el método de facturación preferido del cliente es el contrato Enterprise de Microsoft, los honorarios de las licencias de software se facturarán mediante este método de facturación como un costo detallado, independiente de los costos de uso específicos de Azure.

**Tarjetas de crédito y factura mensual** 

Los clientes también pueden pagar con tarjeta de crédito y una factura mensual. En este caso, los precios de las licencias de software se facturarán igual que en el escenario de contrato Enterprise, como un costo detallado, independiente de los costos de uso específicos de Azure.

Por ejemplo, si el cliente compra con tarjeta de crédito:

|DESCRIPCIÓN    |    Date  |
|----------|----------|
|Período del pedido   | Del 15 de agosto al 30 de agosto de 2018 |
|Final del período (mes)   | 30 de agosto de 2018 |
|Fecha de facturación | 1 de septiembre de 2018 |
|Fecha de pago del cliente | 1 de septiembre de 2018 |
|Período de custodia (solo tarjetas de crédito, 30 días) | Del 1 de septiembre al 30 de septiembre de 2018 |
|Inicio del período de cobro | 1 de septiembre de 2018 |
|Final de período de cobro (30 días como máximo) | 30 de septiembre de 2018 |
|Fecha de cálculo de pago (mensualmente el día 15) | 1 de octubre de 2018 |
|Payout Date (Fecha de pago) | 15 de octubre de 2018 |

Si el cliente compra mediante un contrato Enterprise:

| DESCRIPCIÓN |    Date  |
|----------|----------|
|Período del pedido | Del 15 de agosto al 30 de agosto de 2018 |
|Final del período (trimestre) | 30 de septiembre de 2018 |
|Fecha de facturación | 15 de octubre de 2018 |
|Período de custodia (solo tarjetas de crédito, 30 días) | N/D |
|Inicio del período de cobro | 15 de octubre de 2018 |
|Final de período de cobro (90 días como máximo) | 15 de enero de 2019 |
|Fecha de pago del cliente | 30 de diciembre de 2018 |
|Fecha de cálculo de pago (mensualmente el día 15) | 15 de enero de 2019 |
|Payout Date (Fecha de pago) | 15 de febrero de 2019 |

**Créditos gratuitos y compromiso monetario** 

Algunos clientes eligen pagar Azure por adelantado con un compromiso monetario en el contrato Enterprise, o han recibido créditos gratuitos para su uso con Azure. Aunque estos créditos se pueden usar para pagar por el uso de Azure, no se puede usar para pagar los honorarios de las licencias de software del publicador.

**Facturación y cobro** 

La facturación de las licencias de software del publicador se presenta mediante el método de facturación seleccionado por el cliente y sigue el cronograma de facturación. A los clientes sin un contrato Enterprise vigente se les facturan mensualmente las licencias de software de Marketplace. A los clientes con un contrato Enterprise se les factura mensualmente a través de una factura que se presenta cada trimestre.

Cuando se han seleccionado los modelos de precios de suscripción o de pago por uso, Microsoft actúa como agente del publicador y es responsable de todos los aspectos de facturación, pago y cobro.

### <a name="publisher-payout-and-reporting"></a>Pagos e informes al publicador

* Los honorarios de las licencias de software cobrados por Microsoft como agente están sujetos a una tarifa de transacción del 20 %, a menos que se especifique lo contrario, y se deducen en el momento del pago al publicador.

* Normalmente, los clientes compran con contrato Enterprise o mediante contrato de pago por uso con tarjeta de crédito. El tipo de contrato determina los plazos de facturación, cobro y pago.

>[!NOTE] 
>Todos los informes y la información para la opción de publicación de la transacción están disponibles a través de la sección Insights de Cloud Partner Portal.

#### <a name="billing-questions-and-support"></a>Soporte técnico y preguntas sobre facturación

Para obtener más información y las directivas legales, consulte el [contrato del publicador](https://cloudpartner.azure.com/Content/Unversioned/PublisherAgreement2.pdf) (disponible en Cloud Partner Portal).

Para obtener ayuda sobre preguntas de facturación, [cree un incidente de soporte técnico](https://support.microsoft.com/getsupport?wf=0&tenant=classiccommercial&oaspworkflow=start_1.0.0.0&pesid=16230&forceorigin=esmc&ccsid=636764613233453423) y elija máquinas virtuales o aplicaciones web (también conocidas como aplicaciones SaaS) según el tipo de oferta que se usa.

## <a name="transact-requirements"></a>Requisitos de las transacciones

En esta sección se tratan los requisitos de transacción para distintos tipos de oferta.

### <a name="requirements-for-all-offer-types"></a>Requisitos para todos los tipos de ofertas

**Cuenta del centro de desarrollo y de Microsoft** 

* Se necesita una cuenta tanto del centro de desarrollo como de Microsoft para el modelo de publicación de transacciones, independientemente del modelo de precios de la oferta.
* La cuenta del centro de desarrollo contiene todos los detalles financieros necesarios para que Microsoft cobre los honorarios del cliente en nombre del publicador y pague a este.
* Aunque puede utilizar la misma organización o detalles de inicio de sesión de Microsoft a través de las dos cuentas, el centro de desarrollo es una cuenta independiente de la cuenta de Cloud Partner Portal. Para usar la opción de publicación de transacciones, el publicador debe completar el proceso de registro de una cuenta en el centro de desarrollo, además de registrarse para acceder a Cloud Partner Portal.

*Para obtener más información acerca de cómo configurar estas cuentas, consulte [Conviértase en anunciante de Cloud Marketplace](https://docs.microsoft.com/azure/marketplace/become-publisher).*

### <a name="requirements-for-specific-offer-types"></a>Requisitos para tipos de ofertas específicos

La opción de publicación de transacciones solo está disponible para su uso con los siguientes tipos de oferta de Marketplace: 

**Máquina virtual** 

Seleccione entre los modelos de precios gratuito, traiga su propia licencia o pago por uso, y preséntelo como SKU definida en el nivel de oferta. En la factura de Azure para el cliente, Microsoft presenta los precios de las licencias de software del publicador por separado de los precios de la infraestructura de Azure subyacente. Los precios de la infraestructura de Azure corresponden al uso del software del publicador.

**Aplicaciones de Azure: plantilla de soluciones o aplicación administrada** 

Debe aprovisionar una o varias máquinas virtuales y extrae la suma de los precios de máquina virtual. Para aplicaciones administradas en un único plan, se puede seleccionar una suscripción mensual con tarifa plana como modelo de precios, en su lugar del precio de las máquinas virtuales. En ambos casos, el precio por uso de la infraestructura de Azure se traslada al cliente por separado de los precios de las licencias de software, pero en el mismo extracto de facturación.

## <a name="next-steps"></a>Pasos siguientes

* Revise los requisitos de idoneidad de las opciones de publicación por tipo de oferta para finalizar la selección y la configuración de la oferta.
* Revise los patrones de publicación por escaparate para ver ejemplos de cómo la solución se asigna a un tipo de oferta y configuración.
* Conviértase en anunciante de Marketplace, e inicie sesión en [Cloud Partner Portal](https://cloudpartner.azure.com) para crear y configurar la oferta.
