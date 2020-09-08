---
title: 'Términos de la hoja de precios del Contrato de cliente de Microsoft: Azure'
description: Aprenda a leer y entender el uso y la factura de un contrato de cliente de Microsoft.
author: bandersmsft
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 6205867cbcb65a3b5720fefd1356b36bcd8c6838
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88682729"
---
# <a name="terms-in-your-microsoft-customer-agreement-price-sheet"></a>Términos de la hoja de precios del contrato de cliente de Microsoft

Este artículo se aplica a una cuenta de facturación de Azure de un contrato de cliente de Microsoft. [Compruebe si tiene acceso a un contrato de cliente de Microsoft](#check-access-to-a-microsoft-customer-agreement).

Si es el propietario, colaborador, lector o administrador de facturación de un perfil de facturación, puede descargar la hoja de precios de la organización desde Azure Portal. Consulte [Visualización y descarga de los precios de la organización](ea-pricing.md).

## <a name="terms-and-descriptions-in-your-price-sheet"></a>Términos y descripciones de la hoja de precios

En la siguiente sección se describen los términos importantes que aparecen en la hoja de precios del contrato de cliente de Microsoft.

| **Nombre del campo**   | **Descripción**   |
| --- | --- |
| basePrice  | El precio de mercado en el momento en que el cliente firma o el precio de mercado en el momento en que se inicia el medidor de servicio, si es después del inicio de sesión.   |
| billingAccountId  | Identificador único de la cuenta de facturación.   |
| billingAccountName  | Nombre de la cuenta de facturación.  |
| billingCurrency | Moneda en la que se registran los cargos |
| billingProfileId  | Identificador único del perfil de facturación.   |
| billingProfileName  | Nombre del perfil de facturación que está configurado para recibir facturas. Los precios de la hoja de precios se asocian con este perfil de facturación. |
| currency | Moneda en la que se reflejan todos los precios. |
| discount | Descuento en el precio que se ofrece para el nivel de graduación, el nivel gratis, la cantidad incluida o los descuentos negociados cuando proceda. Se representa como un porcentaje. |
| effectiveEndDate  | Fecha de finalización del precio efectivo. |
| effectiveStartDate  | Fecha de inicio en la que entra en vigor el precio. |
| includedQuantity | Cantidades de un servicio específico que un cliente tiene derecho a consumir sin cargos incrementales. |
| marketPrice | El precio actual del mercado vigente para un servicio determinado. |
| meterId  | Identificador único del medidor. |
| meterCategory  | El nombre de la categoría de clasificación del medidor. Por ejemplo, _servicios en la nube_ o _redes_. |
| meterName  | Nombre del medidor. El medidor representa el recurso de un servicio de Azure que se pueden implementar. |
| meterSubCategory  | Nombre de la categoría de subclasificación del medidor.  |
| meterType  |  Nombre del tipo de medidor. |
| meterRegion  | Nombre de la región donde está disponible el medidor para el servicio. Identifica la ubicación del centro de datos para ciertos servicios cuyos precios se establecen según la ubicación del centro de datos.    |
| Producto  | Nombre del producto que acumula los cargos, por ejemplo: base de datos SQL básica frente a base de datos SQL estándar  |
| productId  | Identificador único del producto cuyo medidor se consume. |
| productOrderName  | Nombre del plan del producto comprado. |
| serviceFamily  | Tipo de servicio de Azure. Por ejemplo: proceso, análisis o seguridad |
| tierMinimumUnits  | Define el límite inferior del intervalo de nivel para el que se definen los precios. Por ejemplo, si el intervalo es de 0 a 100, tierMinimumUnits sería 0.  |
| unitOfMeasure  | Identifica las unidades de medida para la facturación del servicio. Por ejemplo, los servicios de proceso se facturan por hora. |
| unitPrice  | Precio por unidad en el momento de la facturación (no el precio combinado efectivo) como específico para un medidor y el nombre del pedido del producto.  Nota: El precio unitario no es el mismo que el precio efectivo en las descargas de detalles de uso, en el caso de servicios que tienen precios diferentes entre niveles.  En el caso de los servicios con precios de varios niveles, el precio efectivo es una tarifa mixta en todos los niveles y no muestra un precio unitario específico por nivel. El precio combinado o precio efectivo es el precio neto de la cantidad consumida que se extiende a lo largo de los distintos niveles (donde cada nivel tiene un precio unitario específico). |


## <a name="check-access-to-a-microsoft-customer-agreement"></a>Comprobación del acceso a un contrato de cliente de Microsoft
[!INCLUDE [billing-check-mca](../../../includes/billing-check-mca.md)]

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

- [Visualización y descarga de los precios de la organización](ea-pricing.md)
