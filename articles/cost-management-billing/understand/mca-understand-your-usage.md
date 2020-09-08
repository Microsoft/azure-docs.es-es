---
title: Términos del archivo de uso y cargos de Azure para el Contrato de cliente de Microsoft
description: Aprenda a leer y entender las secciones del archivo de cargos y de uso de Azure para el perfil de facturación.
author: bandersmsft
ms.reviewer: amberb
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 826217091b3297da0c068336ff67a0a58841716d
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690466"
---
# <a name="terms-in-the-azure-usage-and-charges-file-for-a-microsoft-customer-agreement"></a>Términos en el archivo de uso y cargos de Azure para un contrato de cliente de Microsoft

Este artículo se aplica a una cuenta de facturación para un contrato de cliente de Microsoft. [Compruebe si tiene acceso a un contrato de cliente de Microsoft](#check-access-to-a-microsoft-customer-agreement).

El archivo .csv de cargos y de uso de Azure contiene los cargos por uso diario y por medidor del período de facturación actual.

Para obtener el archivo de cargos y de uso de Azure, consulte [Visualización y descarga de cargos y de uso de Azure del contrato de cliente de Microsoft](download-azure-daily-usage.md). Está disponible en un formato de archivo de valores separados por comas (.csv) que se puede abrir en una aplicación de hoja de cálculo.

Los cargos de uso son los cargos **mensuales** totales de una suscripción. Los cargos por uso no tienen en cuenta créditos ni descuentos.

## <a name="changes-from-azure-ea-usage-and-charges"></a>Cambios en el uso y los cargos de Azure EA

Si era cliente de EA, verá que los términos del archivo .csv de uso del perfil de facturación de Azure son distintos de los términos del archivo .csv de uso de Azure EA. A continuación se muestra una asignación de los términos de uso del Contrato Enterprise a los términos de uso del perfil de facturación:

| Archivo .csv de uso de Azure EA | CSV de cargos y de uso de Azure para el contrato de cliente de Microsoft |
| --- | --- |
| Date | date |
| Month| date |
| Día | date |
| Year | date |
| Producto | product |
| Id. del medidor | meterID |
| MeterCategory | meterCategory |
| MeterSubCategory | meterSubCategory |
| MeterRegion | meterRegion |
| MeterName | meterName |
| ConsumedQuantity | quantity |
| ResourceRate | effectivePrice |
| ExtendedCost | cost |
| ResourceLocation | resourceLocation |
| ConsumedService | consumedService |
| InstanceId | instanceId |
| ServiceInfo1 | serviceInfo1 |
| ServiceInfo2 | serviceInfo2 |
| AdditionalInfo | additionalInfo |
| Etiquetas | etiquetas |
| StoreServiceIdentifier | N/D |
| DepartmentName | invoiceSection |
| CostCenter | costCenter |
| UnitOfMeasure | unitofMeasure |
| ResourceGroup | resourceGroup |
| ChargesBilledSeparately | isAzureCreditEligible |

## <a name="detailed-terms-and-descriptions"></a>Detalle de términos y descripciones

Los términos siguientes se muestran en el archivo de uso y cargos de Azure.

Término | Descripción
--- | ---
invoiceId | Identificador único del documento mostrado en el PDF de la factura
previousInvoiceId | Referencia a una factura original si este elemento de línea es un reembolso
billingAccountName | Nombre de la cuenta de facturación
billingAccountId | Identificador único de la cuenta raíz
billingProfileId | Nombre del perfil de facturación que acumulará los cargos que se facturan
billingProfileName | Identificador único para el perfil de facturación que acumulará los cargos que se facturan
invoiceSectionId | Identificador único de la sección de facturas
invoiceSectionName | Nombre de la sección de facturas
costCenter | Centro de coste definido en la suscripción para el seguimiento de los costos (disponible solo en períodos de facturación abiertos)
billingPeriodStartDate | Fecha de inicio del período de facturación para el que se genera la factura
billingPeriodEndDate | Fecha de finalización del período de facturación para el que se genera la factura
servicePeriodStartDate | Fecha de inicio del período de calificación que ha definido y bloqueado los precios para el servicio consumido o adquirido.
servicePeriodEndDate | Fecha de finalización del período de calificación que ha definido y bloqueado los precios para el servicio consumido o adquirido.
date | Para cargos basados en uso de Azure y Marketplace, esta es la fecha de calificación. Para las compras únicas (reservas, Marketplace) o los cargos periódicos fijos (ofertas de soporte técnico), esta es la fecha de compra.
serviceFamily | Familia de servicios a la que pertenece el servicio
productOrderId | Identificador único del pedido de productos
productOrderName | Nombre único para el pedido del producto
consumedService | Nombre del servicio consumido
meterId | Identificador único para el medidor
meterName | Nombre del medidor
meterCategory | El nombre de la categoría de clasificación del medidor. Por ejemplo, *servicios en la nube* o *redes*.
meterSubCategory | Nombre de la categoría de subclasificación del medidor
meterRegion | Nombre de la región donde está disponible el medidor para el servicio. Identifica la ubicación del centro de datos para ciertos servicios cuyos precios se establecen según la ubicación del centro de datos.
offer | Nombre de la oferta comprada
PayGPrice | Precio comercial del recurso.
PricingModel | Identificador que indica cómo se calcula el precio del medidor (valores: a petición, reserva, al contado)
productId | Identificador único para el producto que acumula los cargos
product | Nombre del producto que acumula los cargos
Id. de suscripción | Identificador único para la suscripción que acumula los cargos
subscriptionName | Nombre de la suscripción que acumula los cargos
reservationId | Identificador único de la instancia de la reserva comprada
reservationName | Nombre de la instancia de la reserva comprada
publisherType | Tipo de publicador (valores: firstParty, thirdPartyReseller, thirdPartyAgency)
publisherName | Publicador de los servicios de Marketplace
ResourceGroupId | Nombre del grupo de recursos de Azure asociado al recurso
resourceGroupName | Nombre del grupo de recursos asociado al recurso
resourceId | Identificador único para la instancia del recurso
resourceType | Tipo de instancia del recurso
resourceLocation | Identifica la ubicación del centro de datos donde se está ejecutando el recurso
ubicación | Ubicación normalizada del recurso si se configuran diferentes ubicaciones de recursos para las mismas regiones
quantity | Número de unidades compradas o consumidas
unitOfMeasure | Unidad de medida para facturar por el servicio Por ejemplo, los servicios de proceso se facturan por hora.
chargeType | Tipo de cargo. Valores: <ul><li>AsCharged-Usage: Los cargos que se acumulan en función del uso de un servicio de Azure. Esto incluye el uso en máquinas virtuales que no se cobran debido a instancias reservadas.</li><li>AsCharged-PurchaseMarketplace: Cargos únicos o periódicos fijos por compras en Marketplace</li><li>AsCharged-UsageMarketplace: Cargos por servicios de Marketplace que se cobran en función de unidades de consumo</li></ul>
isAzureCreditEligible | Marca que indica si el cargo por el servicio es elegible para su pago mediante créditos de Azure (valores: True, False)
serviceInfo1 | Metadatos específicos del servicio
serviceInfo2 | Campo heredado que captura los metadatos específicos del servicio opcional.
additionalInfo | Metadatos adicionales específicos del servicio.
etiquetas | Etiqueta asignada al recurso

### <a name="make-sure-that-charges-are-correct"></a>Asegúrese de que los cargos sean correctos

Si quiere asegurarse de que los cargos del archivo de uso detallado son correctos, puede comprobarlos. Consulte [Descripción de los cargos en la factura del perfil de facturación](review-customer-agreement-bill.md).

## <a name="check-access-to-a-microsoft-customer-agreement"></a>Comprobación del acceso a un contrato de cliente de Microsoft
[!INCLUDE [billing-check-mca](../../../includes/billing-check-mca.md)]

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

- [Visualización y descarga de la factura de Microsoft Azure](download-azure-invoice.md)
- [Visualización y descarga de los datos de uso y los cargos de Microsoft Azure](download-azure-daily-usage.md)
