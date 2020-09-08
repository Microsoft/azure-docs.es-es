---
title: Aplicación de descuentos de reserva a Azure Synapse Analytics | Microsoft Docs
description: Obtenga información sobre cómo aplicar descuentos de reserva a Azure Synapse Analytics para ayudarle a ahorrar dinero.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 04/30/2020
ms.author: banders
ms.openlocfilehash: f0bd57befc0ec30473065ac69026a77e21e30e5c
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88682610"
---
# <a name="how-reservation-discounts-apply-to-azure-synapse-analytics"></a>Aplicación de descuentos de reserva a Azure Synapse Analytics

Después de comprar capacidad reservada de Azure Synapse Analytics, el descuento de reserva se aplica automáticamente a las instancias aprovisionadas que existen en esa región. El descuento de la reserva se aplica al uso que emite el medidor de cDWU de Azure Synapse Analytics. El almacenamiento y las redes se cargan con tarifas de pago por uso.

## <a name="reservation-discount-application"></a>Aplicación del descuento de reserva

El descuento sobre la capacidad reservada de Azure Synapse Analytics se aplica a los almacenes en ejecución por hora. Si no tiene un almacén implementado durante una hora, se pierde la capacidad reservada para esa hora. No se transfiere.

Después de la compra, la reserva que ha adquirido se asocia al uso de Azure Synapse Analytics emitido al ejecutar los almacenes en cualquier momento. Si apaga algunos almacenes, los descuentos de reserva se aplican automáticamente a otros almacenes que coincidan.

Para los almacenes que no se ejecuten durante una hora completa, la reserva se aplica automáticamente a otras instancias que coincidan en esa hora.

## <a name="discount-examples"></a>Ejemplos de descuento

En los ejemplos siguientes se muestra cómo se aplica el descuento por la capacidad reservada de Azure Synapse Analytics en función de las implementaciones.

- **Ejemplo 1**: compra 5 unidades de capacidad de reserva de 100 cDWU. Ejecuta una instancia de Azure Synapse Analytics de 1500 cDWU durante una hora. En este caso, se emite el uso de 15 unidades de uso de 100 cDWU. El descuento de reserva se aplica a las 5 unidades que se han usado. Se le cargará con tarifas de pago por uso para las restantes 10 unidades de uso de 100 cDWU que se han usado. En otras palabras, la cobertura parcial es posible para varias reservas.

- **Ejemplo 2**: compra 5 unidades de capacidad de reserva de 100 cDWU. Ejecuta dos instancias de Azure Synapse Analytics de 100 cDWU durante una hora. En este caso, se emiten dos eventos de uso para 1 unidad de utilización de 100 cDWU. Ambos eventos de uso obtienen descuentos de capacidad reservada. Las 3 unidades de capacidad de reserva de 100 cDWU restantes se pierden y no se transfieren para un uso futuro. En otras palabras, una reserva individual puede coincidir con varias instancias de Azure Synapse Analytics.

- **Ejemplo 3**: compra 1 unidad de capacidad de reserva de 100 cDWU. Ejecuta dos instancias de Azure Synapse Analytics de 100 cDWU. Cada una de ellas se ejecuta durante 30 minutos. En este caso, ambos eventos de uso obtienen descuentos de capacidad reservada. No se carga ninguna utilización con tarifas de pago por uso.

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Ponerse en contacto con nosotros

- Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:

- [¿Qué es Azure Reservations?](save-compute-costs-reservations.md)
- [Ver transacciones de reserva](view-reservations.md)
- [Obtener las transacciones y el uso de reserva a través de la API](reservation-apis.md)
- [Administrar reservas](manage-reserved-vm-instance.md)
