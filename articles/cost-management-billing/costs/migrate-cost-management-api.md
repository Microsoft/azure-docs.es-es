---
title: Migración del Contrato Enterprise a las API del contrato de cliente de Microsoft - Azure
description: En este artículo se explicarán las consecuencias de migrar un Contrato Enterprise (EA) de Microsoft a un contrato de cliente de Microsoft.
author: bandersmsft
ms.author: banders
ms.date: 07/24/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.reviewer: micflan
ms.openlocfilehash: 1c556f2371ffba239ceee64fd4180143ae9b9230
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87290043"
---
# <a name="migrate-from-enterprise-agreement-to-microsoft-customer-agreement-apis"></a>Migración desde la API de Contrato Enterprise a la API de contrato de cliente de Microsoft

Este artículo lo ayudará a entender la estructura de datos, la API y otras diferencias de integración de sistemas entre las cuentas de Contrato Enterprise (EA) y de contrato de cliente de Microsoft (MCA). Azure Cost Management es compatible con las API de ambos tipos de cuenta. Revise el artículo [Configurar la cuenta de facturación para](../manage/mca-setup-account.md) un contrato de cliente de Microsoft antes de continuar.

Las organizaciones con una cuenta existente de EA deben revisar este artículo junto con la configuración de una cuenta de MCA. Anteriormente, la renovación de una cuenta de EA requería cierto esfuerzo mínimo para migrar desde una inscripción anterior a una nueva. Sin embargo, la migración a una cuenta de MCA requiere un esfuerzo adicional. Este esfuerzo resulta necesario debido a los cambios en el subsistema de facturación subyacente, que afecta a todas las API relacionadas con los costos y a las ofertas de servicio.

## <a name="mca-apis-and-integration"></a>Integración y API de MCA

La nueva integración y las API de MCA le permiten:

- Tener una disponibilidad completa de las API a través de las API nativas de Azure.
- Configurar varias facturas en una sola cuenta de facturación.
- Acceder a una API combinada con el uso del servicio de Azure, el uso de Marketplace de terceros y las compras de Marketplace.
- Ver los perfiles de facturación de costos (lo mismo que las inscripciones) con Azure Cost Management.
- Acceder a API nuevas para mostrar los costos, recibir notificaciones cuando los costos superen los umbrales predefinidos y exportar automáticamente los datos sin procesar.

## <a name="migration-checklist"></a>Lista de comprobación para la migración

Los elementos siguientes lo ayudarán a hacer la transición a las API de MCA.

- Familiarícese con la nueva [cuenta de facturación de contrato de cliente de Microsoft](../understand/mca-overview.md).
- Determine cuáles son las API que usa y vea cuáles se reemplazan en la sección siguiente.
- Familiarícese con las [API REST de Azure Resource Manager](/rest/api/azure).
- Si todavía no usa las API de Azure Resource Manager, [registre su aplicación cliente con Azure AD](/rest/api/azure/#register-your-client-application-with-azure-ad).
- Actualice cualquier código de programación para [usar la autenticación de Azure AD](/rest/api/azure/#create-the-request).
- Actualice cualquier código de programación para reemplazar las llamadas API de EA por las llamadas API de MCA.
- Actualice el control de errores para usar códigos de errores nuevos.
- Revise otras ofertas de integración, como Cloudyn y Power BI, en caso de que se requiera alguna otra acción.

## <a name="ea-apis-replaced-with-mca-apis"></a>API de EA reemplazadas por API de MCA

Las API de EA usan una clave de API para la autenticación y la autorización. Las API de MCA usan la autenticación de Azure AD.

| Propósito | API de EA | API de MCA |
| --- | --- | --- |
| Saldo y créditos | [/balancesummary](/rest/api/billing/enterprise/billing-enterprise-api-balance-summary) | Microsoft.Billing/billingAccounts/billingProfiles/availableBalanceussae |
| Uso (JSON) | [/usagedetails](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#json-format)[/usagedetailsbycustomdate](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#json-format) | [Microsoft.Consumption/usageDetails](/rest/api/consumption/usagedetails)<sup>1</sup> |
| Uso (CSV) | [/usagedetails/download](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#csv-format)[/usagedetails/submit](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#csv-format) | [Microsoft.Consumption/usageDetails/download](/rest/api/consumption/usagedetails)<sup>1</sup> |
| Uso de Marketplace (CSV) | [/marketplacecharges](/rest/api/billing/enterprise/billing-enterprise-api-marketplace-storecharge)[/marketplacechargesbycustomdate](/rest/api/billing/enterprise/billing-enterprise-api-marketplace-storecharge) | [Microsoft.Consumption/usageDetails/download](/rest/api/consumption/usagedetails)<sup>1</sup> |
| Períodos de facturación | [/billingperiods](/rest/api/billing/enterprise/billing-enterprise-api-billing-periods) | Microsoft.Billing/billingAccounts/billingProfiles/invoices |
| Hoja de precios | [/pricesheet](/rest/api/billing/enterprise/billing-enterprise-api-pricesheet) | Microsoft.Billing/billingAccounts/billingProfiles/pricesheet/default/download format=json|csv Microsoft.Billing/billingAccounts/…/billingProfiles/…/invoices/… /pricesheet/default/download format=json|csv Microsoft.Billing/billingAccounts/../billingProfiles/../providers/Microsoft.Consumption/pricesheets/download  |
| Compras de la reserva | [/reservationcharges](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-charges) | Microsoft.Billing/billingAccounts/billingProfiles/transactions |
| Recomendaciones de reserva | [/SharedReservationRecommendations](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation#request-for-shared-reserved-instance-recommendations)[/](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation#request-for-single-reserved-instance-recommendations)[SingleReservationRecommendations](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation#request-for-single-reserved-instance-recommendations) | [Microsoft.Consumption/reservationRecommendations](/rest/api/consumption/reservationrecommendations/list) |
| Uso de las reservas | [/reservationdetails](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage#request-for-reserved-instance-usage-details)[/reservationsummaries](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage) | [Microsoft.Consumption/reservationDetails](/rest/api/consumption/reservationsdetails)[Microsoft.Consumption/reservationSummaries](/rest/api/consumption/reservationssummaries) |

<sup>1</sup> El uso de Marketplace de terceros y el servicio de Azure están disponibles con [Usage Details API](/rest/api/consumption/usagedetails).

Las API siguientes están compatibles para las cuentas de facturación de MCA:

| Propósito | API de contrato de cliente de Microsoft (MCA) |
| --- | --- |
| Cuentas de facturación<sup>2</sup> | Microsoft.Billing/billingAccounts |
| Perfiles de facturación<sup>2</sup> | Microsoft.Billing/billingAccounts/billingProfiles |
| Secciones de factura<sup>2</sup> | Microsoft.Billing/billingAccounts/invoiceSections |
| Facturas | Microsoft.Billing/billingAccounts/billingProfiles/invoices |
| Suscripciones de facturación | {ámbito}/billingSubscriptions |

<sup>2</sup> Las API devuelven listas de objetos, que son ámbitos, donde operan las experiencias de Cost Management en Azure Portal y las API. Para más información sobre los ámbitos de Cost Management, consulte [Descripción y uso de ámbitos](understand-work-scopes.md).

Si usa cualquiera de las API de EA existentes, deberá actualizarlas para que admitan las cuentas de facturación de MCA. En la tabla siguiente se muestran otros cambios en la integración:

| Propósito | Oferta anterior | Nueva oferta |
| --- | --- | --- |
| Cloudyn | [Cloudyn.com](https://www.cloudyn.com) | [Azure Cost Management](https://azure.microsoft.com/services/cost-management/) |
| Power BI | Conector y paquete de contenido de [Microsoft Consumption Insights](/power-bi/desktop-connect-azure-consumption-insights) |  [Conector de Azure Consumption Insights](/power-bi/desktop-connect-azure-consumption-insights) |

## <a name="apis-to-get-balance-and-credits"></a>API para obtener el saldo y los créditos

La API [Get Balance Summary](/rest/api/billing/enterprise/billing-enterprise-api-balance-summary) proporciona un resumen mensual de:

- Saldos
- Nuevas compras
- Cargos por servicios de Azure Marketplace
- Ajustes
- Cargos de uso del servicio por encima del límite

Todas las API de consumo se reemplazan por las API nativas de Azure que usan Azure AD para la autenticación y autorización. Para más información sobre cómo llamar a las API REST de Azure, consulte la [introducción a REST](/rest/api/azure/#create-the-request).

La API Get Balance Summary se reemplaza por la API Microsoft.Billing/billingAccounts/billingProfiles/availableBalance.

Para obtener los saldos disponibles con la API Available Balance:

| Método | URI de solicitud |
| --- | --- |
| GET | `https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/billingProfiles/{billingProfileId}/availableBalances?api-version=2018-11-01-preview` |

## <a name="apis-to-get-cost-and-usage"></a>API para obtener el costo y el uso

Obtenga un desglose diario de los costos derivados del uso del servicio de Azure, del uso de Marketplace de terceros y de otras compras de Marketplace con las API siguientes. Se combinaron las siguientes API independientes para el uso de Marketplace de terceros y los servicios de Azure. Las API anteriores las reemplaza la API [Microsoft.Consumption/usageDetails](/rest/api/consumption/usagedetails). Esta API agrega las compras de Marketplace, las que anteriormente solo se mostraban en el resumen del saldo a la fecha.

- [Obtención de detalles del uso/download](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#csv-format)
- [Obtención de detalles del uso/submit](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#csv-format)
- [Obtención de detalles del uso/usagedetails](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#json-format)
- [Obtención de detalles del uso/usagedetailsbycustomdate](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#json-format)
- [Obtención de cargos de la tienda Marketplace/marketplacecharges](/rest/api/billing/enterprise/billing-enterprise-api-marketplace-storecharge)
- [Obtención de cargos de la tienda Marketplace/marketplacechargesbycustomdate](/rest/api/billing/enterprise/billing-enterprise-api-marketplace-storecharge)

Todas las API de consumo se reemplazan por las API nativas de Azure que usan Azure AD para la autenticación y autorización. Para más información sobre cómo llamar a las API REST de Azure, consulte la [introducción a REST](/rest/api/azure/#create-the-request).

La API Consumption/Usage Details reemplazada a todas las API anteriores.

Para obtener los detalles del uso con la API Usage Details:

| Método | URI de solicitud |
| --- | --- |
| GET | `https://management.azure.com/{scope}/providers/Microsoft.Consumption/usageDetails?api-version=2019-01-01` |

La API Usage Details, así como todas las API de Cost Management, está disponible en varios ámbitos. En el caso de los costos facturados, como los recibiría tradicionalmente en un nivel de inscripción, use el ámbito del perfil de facturación.  Para más información sobre los ámbitos de Cost Management, consulte [Descripción y uso de ámbitos](understand-work-scopes.md).

| Tipo | Formato de identificador |
| --- | --- |
| Cuenta de facturación | `/Microsoft.Billing/billingAccounts/{billingAccountId}` |
| Perfil de facturación | `/Microsoft.Billing/billingAccounts/{billingAccountId}/billingProfiles/{billingProfileId}` |
| Subscription | `/subscriptions/{subscriptionId}` |
| Resource group | `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}` |

Use los siguientes parámetros de cadena de consulta para actualizar cualquier código de programación.

| Parámetros antiguos | Parámetros nuevos |
| --- | --- |
| `billingPeriod={billingPeriod}` | No compatible |
| `endTime=yyyy-MM-dd` | `endDate=yyyy-MM-dd` |
| `startTime=yyyy-MM-dd` | `startDate=yyyy-MM-dd` |

También se modificó el cuerpo de la respuesta.

Cuerpo de la respuesta anterior:

```
{
  "id": "string",
  "data": [{...}, ...],
  "nextLink": "string"
}
```

Cuerpo de la respuesta nuevo:

```
{
  "value": [{
    "id": "{scope}/providers/Microsoft.Consumption/usageDetails/###",
    "name": "###",
    "type": "Microsoft.Consumption/usageDetails",
    "tags": {...},
    "properties": [{...}, ...],
    "nextLink": "string"
  }, ...]
}
```

El nombre de la propiedad que contiene la matriz de los registros de uso cambió de datos a _valores_. Cada registro solía tener una lista plana de propiedades detalladas. Sin embargo, ahora todos los detalles se encuentran en una propiedad anidada denominada _propiedades_, excepto las etiquetas. La estructura nueva es coherente con otras API de Azure. Se modificaron algunos nombres de propiedades. En la tabla siguiente se muestran las propiedades correspondientes.

| Propiedad anterior | Propiedad nueva | Notas |
| --- | --- | --- |
| AccountId | N/D | No se hace un seguimiento del creador de la suscripción. Use invoiceSectionId (igual que departmentId). |
| AccountNameAccountOwnerId y AccountOwnerEmail | N/D | No se hace un seguimiento del creador de la suscripción. Use invoiceSectionName (igual que departmentName). |
| AdditionalInfo | additionalInfo | &nbsp;  |
| ChargesBilledSeparately | isAzureCreditEligible | Observe que estas propiedades son opuestas. Si isAzureCreditEnabled es true, ChargesBilledSeparately será false. |
| ConsumedQuantity | quantity | &nbsp; |
| ConsumedService | consumedService | Los valores de cadena exactos pueden diferir. |
| ConsumedServiceId | None | &nbsp; |
| CostCenter | costCenter | &nbsp; |
| Date y usageStartDate | date | &nbsp;  |
| Día | None | Analiza el día de la fecha. |
| DepartmentId | invoiceSectionId | Los valores exactos difieren. |
| DepartmentName | invoiceSectionName | Los valores de cadena exactos pueden diferir. Si es necesario, configure las secciones de factura para que coincidan con los departamentos. |
| ExtendedCost y Cost | costInBillingCurrency | &nbsp;  |
| InstanceId | resourceId | &nbsp;  |
| Is Recurring Charge | None | &nbsp;  |
| Location | ubicación | &nbsp;  |
| MeterCategory | meterCategory | Los valores de cadena exactos pueden diferir. |
| Id. del medidor | meterId | Los valores de cadena exactos difieren. |
| MeterName | meterName | Los valores de cadena exactos pueden diferir. |
| MeterRegion | meterRegion | Los valores de cadena exactos pueden diferir. |
| MeterSubCategory | meterSubCategory | Los valores de cadena exactos pueden diferir. |
| Month | None | Analiza el mes de la fecha. |
| Nombre de la oferta | None | Use publisherName y productOrderName. |
| OfferID | None | &nbsp;  |
| Order Number | None | &nbsp;  |
| PartNumber | None | Use meterId y productOrderName para identificar los precios de manera única. |
| Plan Name | productOrderName | &nbsp;  |
| Producto | Producto |   |
| ProductId | productId | Los valores de cadena exactos difieren. |
| Nombre de publicador | publisherName | &nbsp;  |
| ResourceGroup | resourceGroupName | &nbsp;  |
| ResourceGuid | meterId | Los valores de cadena exactos difieren. |
| ResourceLocation | resourceLocation | &nbsp;  |
| ResourceLocationId | None | &nbsp;  |
| ResourceRate | effectivePrice | &nbsp;  |
| ServiceAdministratorId | N/D | &nbsp;  |
| ServiceInfo1 | serviceInfo1 | &nbsp;  |
| ServiceInfo2 | serviceInfo2 | &nbsp;  |
| ServiceName | meterCategory | Los valores de cadena exactos pueden diferir. |
| ServiceTier | meterSubCategory | Los valores de cadena exactos pueden diferir. |
| StoreServiceIdentifier | N/D | &nbsp;  |
| SubscriptionGuid | subscriptionId | &nbsp;  |
| SubscriptionId | subscriptionId | &nbsp;  |
| SubscriptionName | subscriptionName | &nbsp;  |
| Etiquetas | etiquetas | La propiedad tags se aplica a un objeto raíz y no a la propiedad de propiedades anidadas. |
| UnitOfMeasure | unitOfMeasure | Los valores de cadena exactos difieren. |
| usageEndDate | date | &nbsp;  |
| Year | None | Analiza el año de la fecha. |
| (nueva) | billingCurrency | La moneda que se usa para el cargo. |
| (nueva) | billingProfileId | El identificador único del perfil de facturación (igual que la inscripción). |
| (nueva) | billingProfileName | El nombre del perfil de facturación (igual que la inscripción). |
| (nueva) | chargeType | Úsela para diferenciar el uso del servicio de Azure, el uso de Marketplace y las compras. |
| (nueva) | invoiceId | El identificador único de la factura. Propiedad vacía para el mes abierto actual. |
| (nueva) | publisherType | El tipo de anunciante para las compras. Propiedad vacía para el uso. |
| (nueva) | serviceFamily | El tipo de compra. Propiedad vacía para el uso. |
| (nueva) | servicePeriodEndDate | La fecha de finalización del servicio comprado. |
| (nueva) | servicePeriodStartDate | La fecha de inicio del servicio comprado. |

## <a name="billing-periods-api-replaced-by-invoices-api"></a>Billing Periods API reemplazada por Invoices API

Las cuentas de facturación de MCA no usan los períodos de facturación. En su lugar, usan las facturas para centrar los costos en los períodos de facturación específicos. La [API Billing Periods](/rest/api/billing/enterprise/billing-enterprise-api-billing-periods) es reemplazada por la API Invoices. Todas las API de consumo se reemplazan por las API nativas de Azure que usan Azure AD para la autenticación y autorización. Para más información sobre cómo llamar a las API REST de Azure, consulte la [introducción a REST](/rest/api/azure/#create-the-request).

Para obtener facturas con la API Invoices:

| Método | URI de solicitud |
| --- | --- |
| GET | `https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/billingProfiles/{billingProfileId}/invoices?api-version=2018-11-01-preview` |

## <a name="price-sheet-apis"></a>Price Sheet API

En esta sección se analizan las Price Sheet API y se proporcionan recomendaciones para migrar a la Price Sheet API de los contratos de cliente de Microsoft. También se describe la Price Sheet API de los contratos de cliente de Microsoft y se explican los campos de las hojas de precios. Las API [Enterprise Get price sheet](/rest/api/billing/enterprise/billing-enterprise-api-pricesheet) y [Enterprise Get billing periods](/rest/api/billing/enterprise/billing-enterprise-api-billing-periods) se reemplazan por Price Sheet API para los contratos de cliente de Microsoft (Microsoft.Billing/billingAccounts/billingProfiles/pricesheet). La API nueva admite los formatos JSON y CSV, en formatos asincrónicos de REST. Todas las API de consumo se reemplazan por las API nativas de Azure que usan Azure AD para la autenticación y autorización. Para más información sobre cómo llamar a las API REST de Azure, consulte la [introducción a REST](/rest/api/azure/#create-the-request).

### <a name="billing-enterprise-apis"></a>API Billing Enterprise

Usaba las API Billing Enterprise con las inscripciones de Enterprise para obtener información sobre los precios y el período de facturación. La autenticación y la autorización usaban tokens web de Azure Active Directory.

Para obtener los precios aplicables para la inscripción de Enterprise especificada con las API Price Sheet y Billing Period:

| Método | URI de solicitud |
| --- | --- |
| GET | `https://consumption.azure.com/v2/enrollments/{enrollmentNumber}/pricesheet` |
| GET | `https://consumption.azure.com/v2/enrollments/{enrollmentNumber}/billingPeriods/{billingPeriod}/pricesheet` |

### <a name="price-sheet-api-for-microsoft-customer-agreements"></a>Price Sheet API para los contratos de cliente de Microsoft

Use Price Sheet API para los contratos de cliente de Microsoft para ver los precios de todos los servicios de consumo de Marketplace y de consumo de Azure. Los precios que se muestran para el perfil de facturación se aplican a todas las suscripciones que pertenecen al perfil de facturación.

Use Price Sheet API para ver todos los datos de la hoja de precios de los servicios de consumo de Azure en formato CSV:

| Método | URI de solicitud |
| --- | --- |
| POST | `https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/billingProfiles/{billingProfileId}/pricesheet/default/download?api-version=2018-11-01-preview&startDate=2019-01-01&endDate=2019-01-31&format=csv` |

Use Price Sheet API para ver todos los datos de la hoja de precios de los servicios de consumo de Azure en formato JSON:

| Método | URI de solicitud |
| --- | --- |
| POST | `https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/billingProfiles/{billingProfileId}/pricesheet/default/download?api-version=2018-11-01-preview&startDate=2019-01-01&endDate=2019-01-31&format=json` |

Al usar la API, se devuelve la hoja de precios de toda la cuenta. Sin embargo, también puede obtener una versión resumida de la hoja de precios en formato PDF. El resumen incluye los servicios de consumo de Marketplace y de Azure que se facturan para una factura específica. La factura se identifica con el {invoiceId}, que es igual que el **número de factura** que se muestra en los archivos PDF de resumen de una factura. Este es un ejemplo.

![Imagen de ejemplo que muestra el número de factura correspondiente a InvoiceId](./media/migrate-cost-management-api/invoicesummary.png)

Para ver la información de factura con Price Sheet API en formato CSV:

| Método | URI de solicitud |
| --- | --- |
| POST | `https://management.azure.com/providers/Microsoft.Billing/billingAccounts/2909cffc-b0a2-5de1-bb7b-5d3383764184/billingProfiles/2dcffe0c-ee92-4265-8647-515b8fe7dc78/invoices/{invoiceId}/pricesheet/default/download?api-version=2018-11-01-preview&format=csv` |

Para ver la información de factura con Price Sheet API en formato JSON:

| Método | URI de solicitud |
| --- | --- |
| POST | `https://management.azure.com/providers/Microsoft.Billing/billingAccounts/2909cffc-b0a2-5de1-bb7b-5d3383764184/billingProfiles/2dcffe0c-ee92-4265-8647-515b8fe7dc78/invoices/{invoiceId}/pricesheet/default/download?api-version=2018-11-01-preview&format=json` |

También puede ver los precios estimados correspondientes a cualquier servicio de consumo de Marketplace o de Azure en el período de servicio o en el ciclo de facturación abierto actual.

Para ver los precios estimados de los servicios de consumo con Price Sheet API en formato CSV:

| Método | URI de solicitud |
| --- | --- |
| POST | `https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billing AccountId}/billingProfiles/{billingProfileId}/pricesheet/default/download?api-version=2018-11-01-preview&format=csv` |

Para ver los precios estimados de los servicios de consumo con Price Sheet API en formato JSON:

| Método | URI de solicitud |
| --- | --- |
| POST | `https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billing AccountId}/billingProfiles/{billingProfileId}/pricesheet/default/download?api-version=2018-11-01-preview&format=json` |

Las Price Sheet API de contrato de cliente de Microsoft son *API REST asincrónicas*. Las respuestas para las API cambiaron desde las API sincrónicas anteriores. También se modificó el cuerpo de la respuesta de API.

#### <a name="old-response-body"></a>Cuerpo de la respuesta anterior

Este es un ejemplo de la respuesta de API REST sincrónica:

```
[
        {
            "id": "enrollments/573549891/billingperiods/2016011/products/343/pricesheets",
            "billingPeriodId": "201704",
            "meterId": "dc210ecb-97e8-4522-8134-2385494233c0",
            "meterName": "A1 VM",
            "unitOfMeasure": "100 Hours",
            "includedQuantity": 0,
            "partNumber": "N7H-00015",
            "unitPrice": 0.00,
            "currencyCode": "USD"
        },
        {
    ]
```

#### <a name="new-response-body"></a>Cuerpo de la respuesta nuevo

Las API admiten el formato [asincrónico de REST de Azure](../../azure-resource-manager/management/async-operations.md). Llame a la API mediante GET y recibirá la respuesta siguiente:

```
No Response Body

HTTP Status 202 Accepted
```

Estos encabezados se envían con la ubicación de la salida:

```
Location:https://management.azure.com/providers/Microsoft.Consumption/operationresults/{operationId}?sessiontoken=XZDFSnvdkbkdsb==

Azure-AsyncOperation:https://managment.azure.com/providers/Microsoft.Consumption/operationStatus/{operationId}?sessiontoken=XZDFSnvdkbkdsb==

Retry-After: 10

OData-EntityId: {operationId}

```

Realice otra llamada GET a la ubicación. La respuesta a la llamada GET es la misma hasta que la operación se completa o presenta un error. Cuando se completa, la respuesta a la llamada GET de ubicación devuelve la dirección URL de descarga, tal como si la operación se hubiese ejecutado al mismo tiempo. Este es un ejemplo:

```
HTTP Status 200

{
  "id": "providers/Microsoft.Consumption/operationresults/{operationId}",
  "name": {operationId},
  "type": “Microsoft.Consumption/operationResults",
  "properties" : {
    "downloadUrl": {urltoblob},
    "validTill": "Date"
  }
}
```

El cliente también realiza una llamada GET para `Azure-AsyncOperation`. El punto de conexión devuelve el estado de la operación.

En la tabla siguiente se muestran los campos de la API Enterprise Get price sheet anterior. Incluye los campos correspondientes en la hoja de precios nueva para los contratos de cliente de Microsoft:

| Propiedad anterior | Propiedad nueva | Notas |
| --- | --- | --- |
| billingPeriodId  | _No aplicable_ | No aplicable. En el caso de los contratos de cliente de Microsoft, la factura y la hoja de precios asociada reemplazaron el concepto de billingPeriodId. |
| meterId  | meterId | &nbsp;  |
| unitOfMeasure  | unitOfMeasure | Los valores de cadena exactos pueden diferir. |
| includedQuantity  | includedQuantity | No es aplicable para los servicios en los contratos de cliente de Microsoft. |
| partNumber  | _No aplicable_ | En su lugar, use una combinación de productOrderName (igual que offerID) y meterID. |
| unitPrice  | unitPrice | El precio unitario se aplica a los servicios consumidos en los contratos de cliente de Microsoft. |
| currencyCode  | pricingCurrency | Los contratos de cliente de Microsoft representan los precios en la moneda de precios y la moneda de facturación. currencyCode corresponde a pricingCurrency en los contratos de cliente de Microsoft. |
| offerID | productOrderName | En lugar de OfferID, puede usar productOrderName, pero no es lo mismo que OfferID. Sin embargo, productOrderName y meter determinan los precios en los contratos de cliente de Microsoft relacionados con meterId y OfferID en las inscripciones heredadas. |

## <a name="consumption-price-sheet-api-operations"></a>Operaciones de Consumption Price Sheet API

En el caso de los Contratos Enterprise, usaba las operaciones [Get](/rest/api/consumption/pricesheet/get) y [Get By Billing Period](/rest/api/consumption/pricesheet/getbybillingperiod) de Consumption Price Sheet API para un ámbito por subscriptionId o un período de facturación. La API usa la autenticación de la administración de recursos de Azure.

Para obtener la información de la hoja de precios para un ámbito con Price Sheet API:

| Método | URI de solicitud |
| --- | --- |
| GET | `https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Consumption/pricesheets/default?api-version=2018-10-01` |

Para obtener la información de la hoja de precios por período de facturación con Price Sheet API:

| Método | URI de solicitud |
| --- | --- |
| GET | `https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Billing/billingPeriods/{billingPeriodName}/providers/Microsoft.Consumption/pricesheets/default?api-version=2018-10-01` |

En lugar de los anteriores puntos de conexión de la API, use los siguientes para los contratos de cliente de Microsoft:

**Price Sheet API para contratos de cliente de Microsoft (API REST asincrónica)**

Esta API es para los contratos de cliente de Microsoft y proporciona atributos adicionales.

**Hoja de precios para un ámbito de perfil de facturación en una cuenta de facturación**

Esta API es la API existente. Se actualizó para proporcionar la hoja de precios para un perfil de facturación de una cuenta de facturación.

## <a name="price-sheet-for-a-scope-by-billing-account"></a>Hoja de precios para un ámbito por cuenta de facturación

La autenticación de Azure Resource Manager se usa cuando obtiene la hoja de precios en el ámbito de la inscripción en una cuenta de facturación.

Para obtener la hoja de precios en la cuenta de inscripción de una cuenta de facturación:

| Método | URI de solicitud |
| --- | --- |
| GET | `/providers/Microsoft.Billing/billingAccounts/65085863/providers/Microsoft.Consumption/pricesheets/download?api-version=2019-01-01` |

Para un contrato de cliente de Microsoft, use la información que aparece en la sección siguiente. Proporciona las propiedades de campo que se usan para los contratos de cliente de Microsoft.

### <a name="price-sheet-for-a-billing-profile-scope-in-a-billing-account"></a>Hoja de precios para un ámbito del perfil de facturación en una cuenta de facturación

La API Price Sheet by billing account actualizada obtiene la hoja de precios en formato CSV. Para obtener la hoja de precios en el ámbito del perfil de facturación para un MCA:

| Método | URI de solicitud |
| --- | --- |
| GET | `/providers/Microsoft.Billing/billingAccounts/28ae4b7f-41bb-581e-9fa4-8270c857aa5f/billingProfiles/ef37facb-cd6f-437a-9261-65df15b673f9/providers/Microsoft.Consumption/pricesheets/download?api-version=2019-01-01` |

En el ámbito de la inscripción de EA, la respuesta de la API y las propiedades son idénticas. Las propiedades corresponden a las mismas propiedades del MCA.

Las propiedades anteriores para las [Price Sheet API de Azure Resource Manager](/rest/api/consumption/pricesheet) y las mismas propiedades nuevas están en la tabla siguiente.

| Anterior propiedad de Price Sheet API de Azure Resource Manager  | Nueva propiedad de Price Sheet API de contrato de cliente de Microsoft   | Descripción |
| --- | --- | --- |
| Id. de medidor | _meterId_ | Identificador único del medidor. Igual que meterID. |
| Nombre del medidor | meterName | Nombre del medidor. El medidor representa el recurso de un servicio de Azure que se puede implementar. |
| Categoría del medidor  | service | El nombre de la categoría de clasificación del medidor. Igual que el servicio en la hoja de precios de contrato de cliente de Microsoft. Los valores de cadena exactos difieren. |
| Subcategoría del medidor | meterSubCategory | El nombre de la categoría de subclasificación del medidor. Basado en la clasificación de la diferenciación del conjunto de características de alto nivel del servicio. Por ejemplo, base de datos SQL básica frente a base de datos SQL estándar. |
| Región del medidor | meterRegion | &nbsp;  |
| Unidad | _No aplicable_ | Se puede analizar a partir de unitOfMeasure. |
| Unidad de medida | unitOfMeasure | &nbsp;  |
| Número de pieza | _No aplicable_ | En lugar del número de pieza, use productOrderName y MeterID para identificar de manera única el precio para un perfil de facturación. Los campos se enumeran en la factura de MCA en lugar del número de pieza en las facturas de MCA. |
| Precio unitario | unitPrice | El precio unitario en los contratos de cliente de Microsoft. |
| Código de moneda | pricingCurrency | Los contratos de cliente de Microsoft representan los precios en la moneda de precios y la moneda de facturación. El código de divisa es igual que pricingCurrency en los contratos de cliente de Microsoft. |
| Cantidad incluida | includedQuantity | No es aplicable a los servicios en los contratos de cliente de Microsoft. Se muestra con valores de cero. |
|  Id. de oferta  | productOrderName | En lugar de OfferID, use productOrderName. No es lo mismo que OfferID, pero productOrderName y meter determinan los precios de los contratos de cliente de Microsoft. Relacionado con meterId y OfferiD en las inscripciones heredadas. |

El precio para los contratos de cliente de Microsoft se define de manera distinta a como se hace en los Contratos Enterprise. El precio de los servicios en la inscripción Enterprise es único para el producto, el número de pieza, el medidor y la oferta. El número de pieza no se usa en los contratos de cliente de Microsoft.

El precio del servicio de consumo de Azure que forma parte de un contrato de cliente de Microsoft es único para productOrderName y meterID. Representan el medidor del servicio y el plan del producto.

Para conciliar la hoja de precios y el uso en Usage Details API, puede usar productOrderName y meterID.

Los usuarios que tienen derechos de propietario del perfil de facturación, colaborador, lector y administrador de facturación pueden descargar la hoja de precios.

La hoja de precios incluye los precios de los servicios cuyos precios se basan en el uso. Los servicios incluyen el consumo de Marketplace y el de Azure. El último precio al final de cada período de servicio se bloquea y se aplica al uso en un período de servicio único. En el caso de los servicios de consumo de Azure, el período de servicio suele ser un mes natural.

### <a name="retired-price-sheet-api-fields"></a>Campos retirados de Price Sheet API

Los siguientes campos no están disponibles en las Price Sheet API de contrato de cliente de Microsoft o tienen los mismos campos.

|Campo retirados| Descripción|
|---|---|
| billingPeriodId | No aplicable. Corresponde a InvoiceId para MCA. |
| offerID | No aplicable. Corresponde a productOrderName en MCA. |
| meterCategory  | No aplicable. Corresponde a Service en MCA. |
| unit | No aplicable. Se puede analizar a partir de unitOfMeasure. |
| currencyCode | Igual que pricingCurrency en MCA. |
| meterLocation | Igual que meterRegion en MCA. |
| partNumber partnumber | No se aplica, porque el número de pieza no aparece en las facturas de MCA. En lugar del número de pieza, use la combinación de meterId y productOrderName para identificar los precios de manera única. |
| totalIncludedQuantity | No aplicable. |
| pretaxStandardRate  | No aplicable. |

## <a name="reservation-instance-charge-api-replaced"></a>Reservation Instance Charge API reemplazada

Puede obtener las transacciones de facturación de compras de reservas con [Reserved Instance Charge API](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-charges). La API nueva incluye todas las compras, incluidas ofertas de Marketplace de terceros. Todas las API de consumo se reemplazan por las API nativas de Azure que usan Azure AD para la autenticación y autorización. Para más información sobre cómo llamar a las API REST de Azure, consulte la [introducción a REST](/rest/api/azure/#create-the-request). Reserved Instance Charge API es reemplazada por Transactions API.

Para obtener las transacciones de compras de reservas con Transactions API:

| Método | URI de solicitud |
| --- | --- |
| GET | `https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/billingProfiles/{billingProfileId}/transactions?api-version=2018-11-01-preview` |

## <a name="recommendations-apis-replaced"></a>Recommendations API reemplazadas

Las Reserved Instance Purchase Recommendations API ofrecen el uso de una máquina virtual durante los últimos 7, 30 o 60 días. Las API también ofrecen recomendaciones de compras de reservas. Incluyen:

- [Shared Reserved Instance Recommendation API](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation#request-for-shared-reserved-instance-recommendations)
- [Single Reserved Instance Recommendations API](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation#request-for-single-reserved-instance-recommendations)

Todas las API de consumo se reemplazan por las API nativas de Azure que usan Azure AD para la autenticación y autorización. Para más información sobre cómo llamar a las API REST de Azure, consulte la [introducción a REST](/rest/api/azure/#create-the-request). Las API de recomendaciones de reservas nombradas anteriormente se reemplazan por la API [Microsoft.Consumption/reservationRecommendations](/rest/api/consumption/reservationrecommendations/list).

Para obtener recomendaciones de reservas con Reservation Recommendations API:

| Método | URI de solicitud |
| --- | --- |
| GET | `https://management.azure.com/providers/Microsoft.Consumption/reservationRecommendations?api-version=2019-01-01` |

## <a name="reservation-usage-apis-replaced"></a>Reservation Usage API reemplazadas

Puede obtener el uso de reservas de una inscripción con Reserved Instance Usage API. Si hay más de una instancia reservada en una inscripción, también puede obtener el uso de todas las compras de instancias reservadas con esta API.

Incluyen:

- [Detalles de uso de instancias reservadas](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage#request-for-reserved-instance-usage-details)
- [Resumen de uso de instancias reservadas](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage)

Todas las API de consumo se reemplazan por las API nativas de Azure que usan Azure AD para la autenticación y autorización. Para más información sobre cómo llamar a las API REST de Azure, consulte la [introducción a REST](/rest/api/azure/#create-the-request). Las API de recomendaciones de reservas nombradas anteriormente se reemplazan por la API [Microsoft.Consumption/reservationDetails](/rest/api/consumption/reservationsdetails) y la API [Microsoft.Consumption/reservationSummaries](/rest/api/consumption/reservationssummaries).

Para obtener detalles de reservas con Reservation Details API:

| Método | URI de solicitud |
| --- | --- |
| GET | `https://management.azure.com/providers/Microsoft.Consumption/reservationDetails?api-version=2019-01-01` |

Para obtener resúmenes de reservas con Reservation Summaries API:

| Método | URI de solicitud |
| --- | --- |
| GET | `https://management.azure.com/providers/Microsoft.Consumption/reservationSummaries?api-version=2019-01-01` |



## <a name="move-from-cloudyn-to-cost-management"></a>Migración de Cloudyn a Cost Management

Las organizaciones que usan [Cloudyn](https://cloudyn.com) deben empezar a usar [Azure Cost Management](https://azure.microsoft.com/services/cost-management/) para cualquier necesidad de administración de costos. Cost Management está disponible en Azure Portal sin incorporación y con una latencia de ocho horas. Para más información, consulte la [documentación de Cost Management](../index.yml).

Con Azure Cost Management, puede:

- Ver costos en el tiempo con respecto a un presupuesto predefinido. Analice los patrones de costos diarios para identificar y detener anomalías en los gastos. Desglose los costos por etiquetas, grupo de recursos, servicio y ubicación.
- Crear presupuestos para establecer límites de uso y costos y recibir notificaciones cuando se esté cerca de umbrales importantes. Configure la automatización con grupos de acciones para desencadenar eventos personalizados y aplicar límites estrictos en sus términos.
- Optimizar el costo y el uso con recomendaciones de Azure Advisor. Descubra cómo optimizar una compra con reservas, disminuir las máquinas virtuales infrautilizadas y eliminar los recursos no utilizados para cumplir con los presupuestos.
- Programe una exportación de datos de costo y uso para publicar diariamente un archivo .csv a la cuenta de almacenamiento. Automatice la integración con sistemas externos para mantener los datos de facturación sincronizados y actualizados.

## <a name="power-bi-integration"></a>Integración con Power BI

También puede usar Power BI para la generación de informes de costos. El [conector de Azure Cost Management](/power-bi/desktop-connect-azure-cost-management) para Power BI Desktop se puede usar para crear informes eficaces y personalizados que ayuden a comprender mejor su gasto con Azure. En la actualidad, el conector de Azure Cost Management admite clientes con un Contrato de cliente de Microsoft o un Contrato Enterprise (EA).

## <a name="next-steps"></a>Pasos siguientes

- Lea la [documentación de Cost Management](../index.yml) para información sobre cómo supervisar y controlar los gastos de Azure. O bien si quiere optimizar el uso de los recursos con Cost Management.
