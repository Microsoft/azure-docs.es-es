---
title: Introducción a Azure Cost Management para los asociados
description: En este artículo se explica cómo los asociados usan las características de Azure Cost Management y cómo permiten a sus clientes el acceso a Cost Management.
author: bandersmsft
ms.author: banders
ms.date: 08/04/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.reviewer: aparnag
ms.custom: secdec18
ms.openlocfilehash: b5f00ee67803819b4d13cdc630bed3de26ac1be9
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87563949"
---
# <a name="get-started-with-azure-cost-management-for-partners"></a>Introducción a Azure Cost Management para los asociados

Azure Cost Management está disponible de forma nativa para los asociados directos que hayan incorporado sus clientes a un contrato de cliente de Microsoft y hayan [adquirido un plan de Azure](/partner-center/purchase-azure-plan). En este artículo se explica cómo usan los asociados las características de [Azure Cost Management](../index.yml) para ver los costos de las suscripciones en el plan de Azure. También se describe cómo los asociados permiten a sus clientes el acceso a Cost Management a precios minoristas.

En el caso de los asociados directos y los proveedores indirectos, tanto el administrador global como los agentes del administrador pueden acceder a Cost Management en el inquilino del asociado y administrar los costos a los precios de la factura.

Tanto los revendedores como los clientes pueden acceder a Cost Management en el inquilino del cliente y ver los costos de las suscripciones, donde los costos se calculan y se muestran a precios para minoristas. Sin embargo, deben tener acceso de RBAC a la suscripción en el inquilino del cliente para ver los costos. El proveedor debe habilitar la directiva de visibilidad de costos para el inquilino del cliente.

Los clientes pueden usar las características de Cost Management cuando su asociado de CSP lo permita.

Los asociados de CSP usan Cost Management para:

- Comprender los costos facturados y asociarlos al cliente, las suscripciones, los grupos de recursos y los servicios.
- Gracias a las funcionalidades para analizar los costos por cliente, suscripción, grupo de recursos, recurso, medidor, servicio y muchas otras dimensiones, puede tener una vista intuitiva de los costos de Azure en el [análisis de costos](quick-acm-cost-analysis.md).
- Vea los costos de los recursos que tienen aplicado un crédito ganado por el asociado (PEC) en el análisis de costos.
- Configure notificaciones y automatización con [presupuestos](tutorial-acm-create-budgets.md) creados mediante programación, junto con alertas cuando los costos superen los presupuestos.
- Habilite la directiva de Azure Resource Manager que proporciona a los clientes acceso a los datos de Cost Management. De este modo, los clientes pueden ver los datos de los costos de consumo de sus suscripciones con las [tarifas de pago por uso](https://azure.microsoft.com/pricing/calculator/).
- Exporte sus datos de costo y uso a un blob de almacenamiento con una suscripción de pago por uso.

Este es un ejemplo que muestra los costos para todos los clientes.

[![Ejemplo que muestra los costos para todos los clientes](./media/get-started-partners/customer-costs1.png)](./media/get-started-partners/customer-costs1.png#lightbox)

Este es un ejemplo que muestra los costos para un solo cliente.

[![Ejemplo que muestra los costos para un solo cliente](./media/get-started-partners/customer-costs2.png)](./media/get-started-partners/customer-costs2.png#lightbox)

Toda la funcionalidad disponible en Azure Cost Management también lo está con las API REST. Use las API para automatizar las tareas de administración de costos.

## <a name="prerequisites"></a>Requisitos previos

Como asociado, Azure Cost Management está disponible de forma nativa solo para las suscripciones que están en el plan de Azure.

Para habilitar Azure Cost Management en el Azure Portal, debe haber confirmado la aceptación del cliente del Contrato de cliente de Microsoft (en nombre del cliente) y haber pasado al cliente al plan de Azure. Solo los costos de las suscripciones que se pasan al plan de Azure están disponibles en Azure Cost Management.

Azure Cost Management requiere acceso de lectura a la cuenta de facturación o suscripción.

Para más información sobre cómo permitir y asignar el acceso a Azure Cost Management para una cuenta de facturación, consulte [Asignar roles y permisos de usuarios](/partner-center/permissions-overview). Los roles de **administrador global** y **agente de administración** pueden administrar los costos de una cuenta de facturación.

Para acceder a Azure Cost Management en el ámbito de la suscripción, cualquier usuario con acceso de RBAC a una suscripción puede ver los costos en las tarifas comerciales (pago por uso). Sin embargo, la [directiva de visibilidad de costos para el inquilino del cliente](#enable-the-policy-to-view-azure-usage-charges) debe estar habilitada. Para ver una lista completa de tipos de cuenta admitidos, consulte [Descripción de los datos de Cost Management](understand-cost-mgt-data.md).

## <a name="how-cost-management-uses-scopes"></a>Uso de los ámbitos por parte de Cost Management

En los ámbitos es donde puede administrar los datos de facturación, tener roles específicos para pagos, ver facturas y realizar la administración de cuentas generales. Los roles de cuenta y facturación se administran de forma independiente de los ámbitos que se usan en la administración de recursos, que emplean RBAC. Para distinguir claramente la finalidad de los distintos ámbitos, incluidas las diferencias en el control de acceso, se hace referencia a ellos como ámbitos de facturación y ámbitos de RBAC, respectivamente.

Para comprender los ámbitos de facturación y los ámbitos de RBAC y cómo funciona la administración de costos con los ámbitos, consulte [Descripción y uso de ámbitos](understand-work-scopes.md).

## <a name="manage-costs-with-partner-tenant-billing-scopes"></a>Administración de costos con ámbitos de facturación de inquilinos de asociados

Después de incorporar a los clientes a un contrato de cliente de Microsoft, están disponibles los siguientes _ámbitos de facturación_ en su inquilino. Use los ámbitos para administrar los costos en Cost Management.

### <a name="billing-account-scope"></a>Ámbito de la cuenta de facturación

Use el ámbito de la cuenta de facturación para ver los costos antes de los impuestos de todos los clientes y perfiles de facturación. Los costos de las facturas solo se muestran para los productos basados en el consumo de los clientes que forman parte del contrato de cliente de Microsoft. Sin embargo, se muestran los costos de las facturas de los productos de compra relativos a los clientes tanto del contrato de cliente de Microsoft como de la oferta de CSP. Actualmente, la moneda predeterminada para ver los costos en el ámbito es dólares estadounidenses. Los presupuestos establecidos para el ámbito también están en USD.

Con independencia de las distintas monedas de facturación del cliente, los asociados usan el ámbito de la cuenta de facturación para establecer presupuestos y administrar los costos en USD de los clientes, las suscripciones, los recursos y los grupos de recursos.

Los asociados también filtran los costos en una moneda de facturación específica para todos los clientes de la vista de análisis de costos. Seleccione la lista **Costo real** para ver los costos en las monedas de facturación del cliente admitidas.

![Ejemplo que muestra la selección de costos reales para las monedas](./media/get-started-partners/actual-cost-selector.png)

Use la [vista de costo amortizado](quick-acm-cost-analysis.md#customize-cost-views) en los ámbitos de facturación para ver los costos amortizados de instancias reservadas en un plazo de reserva.

### <a name="billing-profile-scope"></a>Ámbito del perfil de facturación

Use el ámbito del perfil de facturación para ver los costos antes de los impuestos en la moneda de facturación relativos a todos los clientes de todos los productos y suscripciones incluidos en una factura. Puede filtrar los costos de un perfil de facturación de una factura específica mediante el filtro **InvoiceID**. El filtro muestra el consumo y los costos de compra de productos de una factura específica. También se pueden filtrar la factura de un cliente concreto para ver los costos antes de los impuestos.

Una vez que haya incorporado clientes a un contrato de cliente de Microsoft, recibirá una factura que incluirá todos los cargos de todos los productos (consumos, compras y derechos) de estos clientes en relación con el contrato de cliente de Microsoft. Cuando se facturan en la misma moneda, estas facturas también incluyen los cargos por los derechos y productos adquiridos como SaaS, Azure Marketplace, así como las reservas para los clientes que todavía están en la oferta de CSP.

Para conciliar los cargos con la factura del cliente, el ámbito del perfil de facturación le permite ver todos los costos que se acumulan en una factura para sus clientes. Al igual que la factura, el ámbito muestra los costos de cada cliente en el nuevo contrato de cliente de Microsoft. El ámbito también muestra todos los cargos de los productos de derechos de cliente que todavía están en la oferta de CSP actual.

Los ámbitos de perfil de facturación y cuenta de facturación son los únicos ámbitos aplicables que muestran los cargos por derechos y productos de compra relativos, como Azure Marketplace y las compras de reservas.

Los perfiles de facturación definen las suscripciones que se incluyen en una factura. Son el equivalente funcional de una inscripción de contrato Enterprise. Un perfil de facturación es el ámbito en el que se generan las facturas.

Actualmente, la moneda de facturación del cliente es la moneda predeterminada cuando se ven los costos en el ámbito del perfil de facturación. Los presupuestos establecidos en este ámbito se encuentran en la moneda de facturación.

Los asociados pueden usar el ámbito para conciliar las facturas. Además, usan el ámbito para establecer los presupuestos en la moneda de facturación para los siguientes elementos:

- Una factura filtrada específica
- Customer
- Suscripción
- Resource group
- Recurso
- Servicio de Azure
- Medidor
- ResellerMPNID

### <a name="customer-scope"></a>Ámbito del cliente

Los asociados usan el ámbito para administrar los costos relacionados con los clientes que se incorporan al contrato de cliente de Microsoft. El ámbito permite a los partners ver los costos antes de los impuestos de un cliente específico en una moneda de facturación. También se pueden ver los costos antes de los impuestos de una suscripción, un grupos de recursos o un recurso específicos.

El ámbito del cliente no incluye a los clientes que se encuentran en la oferta de CSP actual. El ámbito solo incluye a los clientes que tienen un contrato de cliente de Microsoft. Los costos de derechos, no los del uso de Azure, de la oferta de CSP actual están disponibles en los ámbitos de la cuenta de facturación y del perfil de facturación cuando se aplica el filtro de cliente. Los presupuestos establecidos en este ámbito se encuentran en la moneda de facturación.

## <a name="partner-access-to-billing-scopes-in-cost-management"></a>Acceso de los asociados a los ámbitos de facturación en Cost Management

Solo los usuarios con los roles de **administrador global** y **agente de administrador** pueden administrar y ver los costos de las cuentas de facturación, los perfiles de facturación y los clientes directamente en el inquilino de Azure del asociado. Para más información sobre los roles del Centro de partners, consulte [Asignar roles y permisos de usuario](/partner-center/permissions-overview).

## <a name="enable-cost-management-for-customer-tenant-subscriptions"></a>Habilitación de la administración de costos para las suscripciones de inquilino de clientes

Los asociados pueden permitir el acceso a Cost Management una vez que los clientes se incorporan a un contrato de cliente de Microsoft. Luego, los asociados pueden habilitar una directiva que permite a los clientes ver sus costos para los servicios consumidos por Azure calculados con las tarifas minoristas de pago por uso. Los costos muestran el uso consumido en los ámbitos de suscripción y grupos de recursos de RBAC en la moneda de facturación del cliente.

Cuando el asociado habilita la directiva de visibilidad de costos, cualquier usuario con acceso de Azure Resource Manager a la suscripción puede administrar y analizar los costos según las tarifas de pago por uso. De hecho, los revendedores y los clientes que tienen el acceso RBAC adecuado a las suscripciones de Azure pueden ver los costos.

Independientemente de la directiva, los administradores globales y los agentes de los administradores del proveedor pueden ver los costos de la suscripción si tienen acceso a la suscripción y al grupo de recursos.

### <a name="enable-the-policy-to-view-azure-usage-charges"></a>Habilitación de la directiva para ver los cargos de uso de Azure

Debe ser miembro del grupo **agentes administradores** para ver y actualizar la directiva. Utilice la siguiente información para habilitar la directiva y permitir a los clientes ver los cargos de uso de Azure.

En Azure Portal, inicie sesión en el inquilino del asociado y seleccione **Administración de costos + facturación**. Seleccione el ámbito de facturación pertinente en el área Ámbito de facturación y, a continuación, seleccione **Clientes**. La lista de clientes está asociada a la cuenta de facturación.

En la lista de clientes, seleccione el que quiere que vea los costos.

[![Seleccionar clientes en Cost Management](./media/get-started-partners/customer-list.png)](./media/get-started-partners/customer-list.png#lightbox)

En **Configuración**, seleccione **Directivas**.

La directiva actual de visibilidad de los costos se muestra para los cargos de **uso de Azure** asociados a las suscripciones del cliente seleccionado.
![Directiva para permitir que los clientes vean los cargos de pago por uso](./media/get-started-partners/cost-management-billing-policies.png)

Cuando la directiva se establece en **No**, Azure Cost Management no está disponible para los usuarios de suscripciones asociados al cliente. A menos que la habilite un asociado, la directiva de visibilidad de los costos está deshabilitada de forma predeterminada para todos los usuarios de suscripciones.

Cuando esta directiva está establecida en **Sí**, los usuarios de suscripciones asociados al inquilino del cliente pueden ver los cargos de uso según las tarifas de pago por uso.

Cuando la directiva de visibilidad de los costos está habilitada, todos los servicios que tienen la opción de uso de la suscripción muestran los costos según las tarifas de pago por uso. El uso de reservas aparece con cero cargos para los costos reales y amortizados. Las compras y los derechos no están asociados a una suscripción específica. Por lo tanto, las compras no se muestran en el ámbito de la suscripción. El administrador global o agente de administración de un asociado directo o de un proveedor indirecto también puede usar la [API de actualización de cliente](/rest/api/billing/2019-10-01-preview/policies/updatecustomer) para establecer la directiva de visibilidad de costos de cada cliente a gran escala.

### <a name="view-subscription-costs-in-the-customer-tenant"></a>Visualización de los costos de suscripción en el inquilino del cliente

Para ver los costos de una suscripción, abra **Cost Management + Facturación** en el inquilino de Azure del cliente. Seleccione **Análisis de costos** y la suscripción necesaria para iniciar la revisión de los costos. Puede ver los costos de consumo de cada suscripción individualmente en el inquilino del cliente.

[![Visualizar el análisis de costos como cliente](./media/get-started-partners/subscription-costs.png)](./media/get-started-partners/subscription-costs.png#lightbox)

Los ámbitos RBAC de grupo de recursos y suscripción tienen a su disposición análisis de costos, presupuestos y alertas según los costos basados en la tarifa de pago por uso.

Las vistas amortizadas y los costos reales de las instancias reservadas de los ámbitos RBAC muestran cero cargos. Los costos de las instancias reservadas solo se muestran en los ámbitos de facturación en los que se realizaron las compras.

Los precios de minoristas que se usan para calcular los costos que se muestran en la vista son los mismos que se muestran en la calculadora de precios de Azure para todos los clientes. Los costos que se muestran no incluyen los descuentos o créditos que el pueda tener el asociado, como créditos obtenidos de asociados, descuentos por nivel y descuentos de servicio global.

## <a name="analyze-costs-in-cost-analysis"></a>Análisis de los costos en el análisis de costos

Los asociados con acceso a los ámbitos de facturación en el inquilino del asociado pueden explorar y analizar los costos facturados del análisis de costos de los clientes y centrarse en un cliente o una factura específicos. En la vista de [análisis de costos](quick-acm-cost-analysis.md), también puede [guardar las vistas](quick-acm-cost-analysis.md#saving-and-sharing-customized-views) y exportar datos a archivos [CSV y PNG](quick-acm-cost-analysis.md#download-usage-data).

Los usuarios de RBAC con acceso a la suscripción en el inquilino del cliente también pueden analizar los costos comerciales de las suscripciones en el inquilino del cliente, guardar vistas y exportar datos a archivos CSV y PNG.

Puede usar características de filtro y agrupar por en el análisis de costos para analizar los costos por varios campos. Los campos específicos del asociado se muestran en la sección siguiente.

## <a name="data-fields"></a>Campos de datos

Los siguientes campos de datos se encuentran en los archivos de detalles de uso y las API de Cost Management. Si está disponible, se muestra la información equivalente del Centro de partners. En los siguientes campos en negrita, los asociados pueden usar las características de filtro y agrupar por en el análisis de costos para analizar los costos por varios campos. Los campos en negrita solo se aplican a los Contratos de cliente de Microsoft admitidos por los asociados.

| **Nombre del campo** | **Descripción** | **Equivalente del Centro de partners** |
| --- | --- | --- |
| invoiceId | Identificador de factura que aparece en la factura de la transacción específica. | Número de factura en el que se muestra la transacción. |
| previousInvoiceID | Referencia a una factura original si hay un reembolso (costo negativo). Solo se rellena si hay un reembolso. | N/D |
| billingAccountName | Nombre de la cuenta de facturación que representa al asociado. Acumula todos los costos de los clientes que se han incorporado a un Contrato de cliente de Microsoft y los clientes de CSP que han realizado compras de derechos, como SaaS y Azure Marketplace, y reservas. | N/D |
| billingAccountID | Identificador de la cuenta de facturación que representa al asociado. | Identificador de raíz comercial de asociados de MCAPI. Se usa en una solicitud, pero no se incluye en una respuesta.|
| billingProfileID | Identificador del perfil de facturación que agrupa los costos de las facturas en una única moneda de facturación de los clientes que se han incorporado a un Contrato de cliente de Microsoft y los clientes de CSP que han realizado compras de derechos, como SaaS y Azure Marketplace, y reservas. | Identificador del grupo de facturación de asociados de MCAPI. Se usa en una solicitud, pero no se incluye en una respuesta. |
| billingProfileName | Nombre del perfil de facturación que agrupa los costos de las facturas en una única moneda de facturación de los clientes que se han incorporado a un Contrato de cliente de Microsoft y los clientes de CSP que han realizado compras de derechos, como SaaS y Azure Marketplace, y reservas. | N/D |
| invoiceSectionName | Nombre del proyecto que se va a cargar en la factura. No se aplica a los Contratos de cliente de Microsoft a los que se han incorporado los asociados. | N/D |
| invoiceSectionID | Identificador del proyecto que se va a cargar en la factura. No se aplica a los Contratos de cliente de Microsoft a los que se han incorporado los asociados. | N/D |
| **CustomerTenantID** | Identificador del inquilino de Azure Active Directory de la suscripción del cliente. | Identificador de organización del cliente: TenantID de Azure Active Directory del cliente. |
| **CustomerName** | Nombre del inquilino de Azure Active Directory de la suscripción del cliente. | Nombre de la organización del cliente, según figura en el Centro de partners. Es importante para la conciliación de la factura con la información del sistema. |
| **CustomerTenantDomainName** | Nombre de dominio del inquilino de Azure Active Directory de la suscripción del cliente. | Dominio del inquilino de Azure Active Directory del cliente. |
| **PartnerTenantID** | Identificador del inquilino de Azure Active Directory del asociado. | TenantID de Azure Active Directory del asociado, conocido como Id. de asociado, en formato GUID. |
| **PartnerName** | Nombre del inquilino de Azure Active Directory del asociado. | Nombre del asociado. |
| **ResellerMPNID** | MPNID del revendedor asociado a la suscripción. | Identificador de MPN del revendedor del registro de la suscripción. No está disponible para la actividad actual. |
| costCenter | Centro de costo asociado a la suscripción. | N/D |
| billingPeriodStartDate | Fecha de inicio del período de facturación, como se muestra en la factura. | N/D |
| billingPeriodEndDate | Fecha de finalización del período de facturación, como se muestra en la factura. | N/D |
| servicePeriodStartDate | Fecha de inicio del período de valoración en que se han valorado los cargos por el uso del servicio. Los precios de los servicios de Azure se determinan según el período de valoración. | ChargeStartDate en el Centro de partners. Fecha de inicio del ciclo de facturación, excepto cuando se presentan fechas de datos de uso latente que no se hayan cobrado anteriormente de un ciclo de facturación anterior. La hora es siempre el principio del día, 00:00. |
| servicePeriodEndDate | Fecha de finalización del período en que se han valorado los cargos por el uso del servicio. Los precios de los servicios de Azure se determinan en función del período de valoración. | N/D |
| date | En el caso de los datos de consumo de Azure, muestra la fecha de uso. En el caso de la instancia reservada, muestra la fecha de compra. En el caso de cargos periódicos y cargos puntuales como Marketplace y soporte técnico, muestra la fecha de compra. | N/D |
| productID | Identificador del producto que ha acumulado cargos por consumo o compra. Es la clave concatenada de productID y SKuID, como se muestra en el Centro de partners. | Identificador del producto. |
| product | Nombre del producto que ha acumulado cargos por consumo o compra, como se muestra en la factura. | Nombre del producto del catálogo. |
| serviceFamily | Muestra la familia de servicios del producto comprado o cargado. Por ejemplo, Storage o Compute. | N/D |
| productOrderID | Identificador del recurso o nombre del plan de Azure al que pertenece la suscripción. Por ejemplo, plan de Azure. | CommerceSubscriptionID |
| productOrderName | Nombre del plan de Azure al que pertenece la suscripción. Por ejemplo, plan de Azure. | N/D|
| consumedService | Servicio consumido (taxonomía heredada) tal como se usa en los detalles de uso de EA heredados. | Servicio mostrado en el Centro de partners. Por ejemplo, Microsoft.Storage, Microsoft.Compute y Microsoft.operationalinsights. |
| meterID | Identificador medido para el consumo medido. | Identificador del medidor utilizado. |
| meterName | Identifica el nombre del medidor para el consumo medido. | Nombre del medidor consumido. |
| meterCategory | Identifica el servicio de nivel superior para el uso. | Identifica el servicio de nivel superior para el uso. |
| meterSubCategory | Define el tipo o subcategoría de servicio de Azure que puede afectar a la tarifa. | Tipo de servicio de Azure que puede afectar a la tarifa.|
| meterRegion | Identifica la ubicación del centro de datos para ciertos servicios cuyos precios se establecen según la ubicación del centro de datos. | Ubicación regional de un centro de datos para servicios, cuando se indiquen y sea aplicable. |
| Id. de suscripción | Identificador único generado por Microsoft para la suscripción de Azure. | EntitlementID |
| subscriptionName | Nombre de la suscripción de Azure. | N/D |
| Término | Muestra el plazo de validez de la oferta. Por ejemplo, las instancias reservadas muestran 12 meses de un plazo anual de la instancia reservada. Para compras de una sola vez o compras periódicas, el término muestra un mes para SaaS, Azure Marketplace y soporte técnico. Esto no se aplica al consumo de Azure. | N/D |
| publisherType (firstParty, thirdPartyReseller, thirdPartyAgency) | Tipo de anunciante que identifica al anunciante como primera entidad, revendedor de terceros o agencia de terceros. | N/D |
| partNumber | Número de pieza de la instancia reservada sin usar y los servicios de Azure Marketplace. | N/D |
| publisherName | Nombre del anunciante del servicio, incluidos Microsoft o los anunciantes ajenos. | Nombre del anunciante del producto.|
| reservationId | Identificador de la compra de instancia reservada. | N/D |
| reservationName | Nombre de la instancia reservada. | N/D |
| reservationOrderId | Identificador de pedido de la instancia reservada. | N/D |
| frequency | Frecuencia de pago de una instancia reservada. | N/D |
| resourceGroup | Nombre del grupo de recursos de Azure usado para la administración de recursos de ciclo de vida. | Nombre del grupo de recursos. |
| instanceID (o) ResourceID | Identificador de la instancia del recurso. | Se muestra como ResourceURI que incluye las propiedades completas del recurso. |
| resourceLocation | Nombre de la ubicación del recurso. | Ubicación del recurso. |
| Location | Ubicación normalizada del recurso. | N/D |
| effectivePrice | Precio por unidad efectivo del servicio en la moneda de determinación del precio. Es único para cada producto, familia de servicios, medidor y oferta. Se usa con los precios de la hoja de precios de la cuenta de facturación. Cuando hay precios por niveles o una cantidad incluida, muestra el precio combinado por consumo. | Precio por unidad después de que se realicen los ajustes. |
| Cantidad | Cantidad medida comprada o consumida. La cantidad del medidor usado durante el período de facturación. | Número de unidades. Asegúrese de que coincide con la información del sistema de facturación durante la conciliación. |
| unitOfMeasure | Identifica la unidad en que se cobra el servicio. Por ejemplo, GB y horas. | Identifica la unidad en que se cobra el servicio. Por ejemplo, GB, horas y 10 000 s. |
| pricingCurrency | Moneda que define el precio unitario. | Moneda de la lista de precios.|
| billingCurrency | Moneda que define el costo facturado. | Moneda de la región geográfica del cliente. |
| chargeType | Define el tipo de cargo que representa el costo en Azure Cost Management, como compra y reembolso. | Tipo de cargo o ajuste. No está disponible para la actividad actual. |
| costinBillingCurrency | Costo combinado o ampliado en la moneda facturada. | N/D |
| costinPricingCurrency | Costo combinado o ampliado antes de impuestos en la moneda de los precios para poner en correlación con estos. | N/D |
| **costinUSD** | Costo combinado o ampliado estimado antes de impuestos en USD. | N/D |
| **paygCostInBillingCurrency** | Muestra los costos si los precios están en los precios minoristas. Muestra los precios de pago por uso en la moneda de facturación. Solo está disponible en los ámbitos de RBAC. | N/D |
| **paygCostInUSD** | Muestra los costos si los precios están en los precios minoristas. Muestra los precios de pago por uso en USD. Solo está disponible en los ámbitos de RBAC. | N/D |
| exchangeRate | Tipo de cambio que se usa para convertir la moneda de los precios a la moneda de facturación. | Se conoce como PCToBCExchangeRate en el Centro de partners. Moneda de los precios para el tipo de cambio de la moneda de facturación.|
| exchangeRateDate | Fecha del tipo de cambio que se usa para convertir la moneda de los precios a la moneda de facturación. | Se conoce como PCToBCExchangeRateDat en el Centro de partners. Moneda de los precios para la fecha del tipo de cambio de la moneda de facturación.|
| isAzureCreditEligible | Indica si el costo es válido para el pago mediante créditos de Azure. | N/D |
| serviceInfo1 | Campo heredado que captura los metadatos específicos del servicio opcional. | Metadatos del servicio de Azure interno. |
| serviceInfo2 | Campo heredado que captura los metadatos específicos del servicio opcional. | Información del servicio. Por ejemplo, un tipo de imagen de una máquina virtual y un nombre de ISP para ExpressRoute.|
| additionalInfo | Metadatos específicos del servicio. Por ejemplo, un tipo de imagen de una máquina virtual. | Cualquier información adicional no incluida en otras columnas. Metadatos específicos de servicio. Por ejemplo, un tipo de imagen de una máquina virtual.|
| etiquetas | Etiqueta que se asigna al medidor. Use etiquetas para agrupar los registros de facturación. Por ejemplo, puede usar etiquetas para distribuir los costos por el departamento que utiliza el medidor. | Etiquetas agregadas por el cliente.|
| **partnerEarnedCreditRate** | Tarifa de descuento aplicada si hay un crédito ganado por el asociado (PEC) basado en el acceso al vínculo de administrador del asociado. | La tasa de créditos ganados por el asociado (PEC). Por ejemplo, 0 % o 15 %. |
| **partnerEarnedCreditApplied** | Indica si se ha aplicado el crédito ganado por el asociado. | N/D |

## <a name="view-partner-earned-credit-pec-resource-costs"></a>Visualización de los costos de recursos de créditos ganados por el asociado (PEC)

En Azure Cost Management, los asociados pueden usar el análisis de costos para ver los costos que recibieron las ventajas de PEC.

En Azure Portal, inicie sesión en el inquilino del asociado y seleccione **Administración de costos + facturación**. En **Cost Management**, seleccione **Análisis de costos**.

La vista de análisis de costos muestra los costos de la cuenta de facturación del asociado. Seleccione el **ámbito** que sea necesario para el asociado, un cliente específico o un perfil de facturación para conciliar las facturas.

En un gráfico de anillos, seleccione la lista desplegable y seleccione **PartnerEarnedCreditApplied** para profundizar en los costos de PEC.

![Ejemplo que muestra cómo ver el crédito ganado por el asociado](./media/get-started-partners/cost-analysis-pec1.png)

Cuando la propiedad **PartnerEarnedCreditApplied** es _True_, el costo asociado tiene la ventaja del acceso de administrador ganado por el asociado.

Cuando la propiedad **PartnerEarnedCreditApplied** es _False_, el costo asociado no ha satisfecho los requisitos necesarios para el crédito. O bien, el servicio adquirido no reúne los requisitos para ganar créditos de asociado.

Lo habitual es que los datos de uso del servicio tarden entre 8 y 24 horas en aparecer en Cost Management. Para más información, consulte [Retención y actualizaciones de datos de uso y costos](understand-cost-mgt-data.md#cost-and-usage-data-updates-and-retention). Los créditos PEC aparecen a las 48 horas desde el momento del acceso en Azure Cost Management.


También puede agrupar y filtrar por la propiedad **PartnerEarnedCreditApplied** mediante las opciones **Agrupar por**. Use las opciones para examinar los costos que tienen y no tienen PEC.

![Agrupación o filtrado por el crédito ganado por el asociado](./media/get-started-partners/cost-analysis-pec2.png)

## <a name="export-cost-data-to-azure-storage"></a>Exportación de datos de costos a Azure Storage

Los asociados con acceso a los ámbitos de facturación de un inquilino de asociado pueden exportar sus datos de costo y uso a un blob de Azure Storage. El blob debe estar en una suscripción del inquilino del asociado que no sea una suscripción de servicio compartido o una suscripción de cliente. Para habilitar la exportación de datos de costos, se recomienda configurar una suscripción de pago por uso independiente en el inquilino de asociado para hospedar los datos de costos exportados. La cuenta de almacenamiento de exportación se crea en el blob de Azure Storage hospedado en la suscripción de pago por uso. En función del ámbito en el que el asociado crea la exportación, los datos asociados se exportan automáticamente a la cuenta de almacenamiento de forma periódica.

Los usuarios con acceso de RBAC a la suscripción también pueden exportar los datos de costo a un blob de almacenamiento de Azure hospedado en cualquier suscripción del inquilino del cliente.

### <a name="create-an-export-in-a-partner-tenant-or-customer-tenant"></a>Creación de una exportación en un inquilino de asociado o de cliente

En Azure Portal, inicie sesión en el inquilino del asociado o de cliente y seleccione **Administración de costos + facturación**. Seleccione un ámbito adecuado, como una cuenta de facturación de Microsoft Partner Agreement, y seleccione **Análisis de costos**. Cuando se cargue la página, seleccione **Exportar**. En Programar exportación, seleccione **Ver todas las exportaciones**.

![Selección de Exportar y Ver todas las exportaciones](./media/get-started-partners/export01.png)

A continuación, seleccione **Agregar** y escriba el nombre y seleccione un tipo de exportación. Seleccione la pestaña **Almacenamiento** y escriba la información necesaria.

![Incorporación de una exportación nueva y selección de la pestaña Almacenamiento](./media/get-started-partners/export02.png)

Cuando cree una exportación en el inquilino de asociado, seleccione la suscripción de pago por uso en el inquilino de asociado. Cree una cuenta de Azure Storage con esa suscripción.

En el caso de los usuarios de RBAC en el inquilino del cliente, seleccione una suscripción en el inquilino del cliente. Cree una cuenta de Azure Storage con la suscripción.

Revise el contenido y, luego seleccione **Crear** para programar una exportación.

Para comprobar los datos en la lista de exportaciones, seleccione el nombre de la cuenta de almacenamiento. En la página de la cuenta de almacenamiento, seleccione **Containers** y el contenedor. Vaya a la carpeta correspondiente y seleccione el archivo CSV. Seleccione **Descargar** para obtener el archivo CSV y abrirlo. Los datos exportados se asemejan a los datos de costos, similares a los detalles de uso de Azure Portal.

![Ejemplo de datos exportados](./media/get-started-partners/example-export-data.png)

## <a name="cost-management-rest-apis"></a>API REST de Cost Management

Los asociados y los clientes pueden usar las API de Cost Management que se describen en las siguientes secciones para tareas comunes.

### <a name="azure-cost-management-apis---direct-and-indirect-providers"></a>API de Azure Cost Management: proveedores directos e indirectos

Los asociados con acceso a los ámbitos de facturación en el inquilino de un asociado pueden usar las siguientes API para ver los costos facturados.

Un asociado puede llamar a las API en el ámbito de la suscripción (independientemente de la directiva de costos) si tiene acceso a la suscripción. Otros usuarios con acceso a la suscripción, como el cliente o el revendedor, pueden llamar a las API solo después de que el asociado habilite la directiva de costos para el inquilino del cliente.


#### <a name="to-get-a-list-of-billing-accounts"></a>Para obtener una lista de cuentas de facturación

```
GET https://management.azure.com/providers/Microsoft.Billing/billingAccounts?api-version=2019-10-01-preview
```

#### <a name="to-get-a-list-of-customers"></a>Para obtener una lista de clientes

```
GET https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountName}/customers?api-version=2019-10-01-preview
```

#### <a name="to-get-a-list-of-subscriptions"></a>Para obtener una lista de suscripciones

```
GET https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountName}/billingSubscriptions?api-version=2019-10-01-preview
```

#### <a name="to-get-a-list-of-invoices-for-a-period-of-time"></a>Para obtener una lista de las facturas de un período de tiempo

```
GET https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountName}/invoices?api-version=2019-10-01-preview&periodStartDate={periodStartDate}&periodEndDate={periodEndDate}
```

La llamada API devuelve una matriz de facturas que tiene elementos similares al siguiente código JSON.

```
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/{billingAccountID}/billingProfiles/{BillingProfileID}/invoices/{InvoiceID}",
      "name": "{InvoiceID}",
      "properties": {
        "amountDue": {
          "currency": "USD",
          "value": x.xx
        },
        ...
    }
```

Use el valor de campo de identificador devuelto anterior y reemplácelo en el ejemplo siguiente como ámbito para consultar los detalles de uso.

```
GET https://management.azure.com/{id}/providers/Microsoft.Consumption/UsageDetails?api-version=2019-10-01
```

En el ejemplo se devuelven los registros de uso asociados a la factura específica.


#### <a name="to-get-the-policy-for-customers-to-view-costs"></a>Para obtener la directiva para que los clientes vean los costos

```
GET https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountName}/customers/{customerID}/policies/default?api-version=2019-10-01-preview
```

#### <a name="to-set-the-policy-for-customers-to-view-costs"></a>Para establecer la directiva para que los clientes vean los costos

```
PUT https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountName}/customers/{customerID}/policies/default?api-version=2019-10-01-preview
```

#### <a name="to-get-azure-service-usage-for-a-billing-account"></a>Para obtener el uso del servicio de Azure de una cuenta de facturación

```
GET https://management.azure.com/providers/Microsoft.Billing/BillingAccounts/{billingAccountName}/providers/Microsoft.Consumption/usageDetails?api-version=2019-10-01
```

#### <a name="to-download-a-customers-azure-service-usage"></a>Para descargar el uso del servicio de Azure de un cliente

La siguiente llamada GET es una operación asincrónica.

```
GET https://management.azure.com/Microsoft.Billing/billingAccounts/{billingAccountName}/customers/{customerID}/providers/Microsoft.Consumption/usageDetails/download?api-version=2019-10-01 -verbose
```

Llame al URI `Location` devuelto en la respuesta para comprobar el estado de la operación. Cuando el estado es *Completado*, la propiedad `downloadUrl` contiene un vínculo que puede usar para descargar el informe generado.


#### <a name="to-get-or-download-the-price-sheet-for-consumed-azure-services"></a>Para obtener o descargar la hoja de precios de los servicios de Azure consumidos

En primer lugar, use la siguiente entrada.

```
POST https://management.azure.com/providers/Microsoft.Billing/BillingAccounts/{billingAccountName}/billingProfiles/{billingProfileID}/pricesheet/default/download?api-version=2019-10-01-preview&format=csv" -verbose
```

A continuación, llame al valor de la propiedad de operación asincrónica. Por ejemplo:

```
GET https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountName}/billingProfiles/{billingProfileID}/pricesheetDownloadOperations/{operation}?sessiontoken=0:11186&api-version=2019-10-01-preview
```
La llamada GET anterior devuelve el vínculo de descarga que contiene la hoja de precios.


#### <a name="to-get-aggregated-costs"></a>Para obtener los costos agregados

```
POST https://management.azure.com/providers/microsoft.billing/billingAccounts/{billingAccountName}/providers/microsoft.costmanagement/query?api-version=2019-10-01
```

#### <a name="create-a-budget-for-a-partner"></a>Creación de un presupuesto para un asociado

```
PUT https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountName}/providers/Microsoft.CostManagement/budgets/partnerworkshopbudget?api-version=2019-10-01
```

#### <a name="create-a-budget-for-a-customer"></a>Creación de un presupuesto para un cliente

```
PUT https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountName}/customers/{customerID}/providers/Microsoft.Consumption/budgets/{budgetName}?api-version=2019-10-01
```

#### <a name="delete-a-budget"></a>Eliminación de un presupuesto

```
DELETE
https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/providers/Microsoft.CostManagement/budgets/{budgetName}?api-version=2019-10-01
```


## <a name="next-steps"></a>Pasos siguientes
- [Inicio del análisis de costos](quick-acm-cost-analysis.md) en Cost Management
- [Creación y administración de presupuestos](tutorial-acm-create-budgets.md) en Cost Management
