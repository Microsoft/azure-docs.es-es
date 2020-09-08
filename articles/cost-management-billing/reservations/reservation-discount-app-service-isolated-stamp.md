---
title: Descuentos de reserva para Azure App Service
description: Obtenga información sobre cómo se aplican los descuentos de reserva a los timbres de Azure App Service en entorno aislado. Los descuentos se aplican automáticamente al impuesto sobre el timbre de una región.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 02/12/2020
ms.author: banders
ms.openlocfilehash: f7d3134652d75cb4ce40b3e9366d3083f5d760df
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88682627"
---
# <a name="how-reservation-discounts-apply-to-azure-app-service-isolated-stamps"></a>Cómo se aplican los descuentos de reserva a los timbres de Azure App Service en entorno aislado

Cuando compra capacidad reservada en el impuesto sobre el timbre de App Service en entorno aislado, el descuento de reserva se aplica automáticamente al impuesto sobre el timbre de una región. El descuento de reserva se aplica al uso que emite el medidor de impuesto sobre el timbre en entorno aislado. Los trabajos, los front-ends adicionales y los demás recursos asociados al timbre se siguen facturando con la tarifa normal.

## <a name="reservation-discount-application"></a>Aplicación del descuento de reserva

El descuento en el impuesto sobre el timbre de App Service en entorno aislado se aplica a la ejecución de timbres aislados por hora. Si no tiene un timbre implementado durante una hora, se pierde la capacidad reservada para esa hora. No se transfiere.

Después de la compra, la reserva que haya adquirido se hará coincidir con un timbre en entorno aislado que se ejecuta en una región específica. Si apaga ese timbre, los descuentos de reserva se aplican automáticamente a cualquier otro timbre que se ejecute en la región. Si no existe ningún timbre, la reserva se aplicará al siguiente timbre que se cree en la región.

Si los timbres no se ejecutan durante una hora completa, la reserva se aplica automáticamente a otros timbres coincidentes de la misma región durante la misma hora.

## <a name="choose-a-stamp-type---windows-or-linux"></a>Selección del tipo de timbre: Windows o Linux

Un timbre vacío en entorno aislado emite el medidor de timbres de Windows de forma predeterminada, por ejemplo, cuando no hay ningún trabajo implementado. Cuando se implementan trabajos de Windows, sigue emitiendo el medidor. Si implementa un trabajo de Linux, el medidor cambia al medidor de timbres de Linux. El timbre emite el medidor de Windows cuando se implementan trabajos de Windows y de Linux.

Como resultado, el medidor de timbres puede cambiar entre Windows y Linux durante la vigencia del timbre. Mientras tanto, las reservas son específicas del sistema operativo. Tendrá que comprar una reserva que admita los trabajos que planea implementar en el timbre. Los timbres mixtos y solo de Windows usan la reserva de Windows. Los timbres que solo tienen trabajos de Linux usan la reserva de Linux.

El único caso en el que debe comprar una reserva de Linux es cuando planee tener _solo_ trabajos de Linux en el timbre.

## <a name="discount-examples"></a>Ejemplos de descuento

En los ejemplos siguientes se muestra cómo se aplica el descuento sobre la instancia reservada en el impuesto sobre el timbre en entorno aislado en función de las implementaciones.

- **Ejemplo 1**: compra una instancia de capacidad de timbre reservado en entorno aislado en una región sin timbres de App Service en entorno aislado. Implementa un nuevo timbre en la región y pagan las tarifas reservadas para dicho timbre.
- **Ejemplo 2**: compra una instancia de capacidad de timbre reservado en entorno aislado en una región que ya tiene implementado un timbre de App Service en entorno aislado. Empieza a recibir la tarifa reservada para el timbre implementado.
- **Ejemplo 3**: compra una instancia de capacidad de timbre reservado en entorno aislado en una región que ya tiene implementado un timbre de App Service en entorno aislado. Empieza a recibir la tarifa reservada para el timbre implementado. Después, elimina el timbre e implementa uno nuevo. Recibe la tarifa reservada para el nuevo timbre. Los descuentos no se transfieren a duraciones sin sellos implementados.
- **Ejemplo 4**: compra una instancia de capacidad de timbre reservado de Linux en entorno aislado en una región y, luego, implementa un nuevo timbre en la región. Cuando el timbre se implementa inicialmente sin trabajos, emite el medidor de timbres de Windows. No se recibe ningún descuento. Cuando el primer trabajo de Linux se implementa en el timbre, emite el medidor de timbres de Linux y se aplica el descuento de reserva. Si un trabajo de Windows se implementa posteriormente en el timbre, el medidor de timbres vuelve a Windows. Ya no recibirá un descuento por la reserva de timbre reservado de Linux en entorno aislado.

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre cómo administrar una reserva, consulte [Administración de Azure Reservations](manage-reserved-vm-instance.md).
- Para obtener más información sobre la compra anticipada de la capacidad reservada de timbre de App Service en entorno aislado para ahorrar dinero, consulte [Pago por adelantado del impuesto sobre el timbre de Azure App Service en entorno aislado con capacidad reservada](prepay-app-service-isolated-stamp.md).
- Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:
  - [¿Qué es Azure Reservations?](save-compute-costs-reservations.md)
  - [Administración de reservas en Azure](manage-reserved-vm-instance.md)
  - [Información sobre el uso de reservas de Azure para suscripciones de pago por uso](understand-reserved-instance-usage.md)
  - [Información sobre el uso de reservas para la inscripción Enterprise](understand-reserved-instance-usage-ea.md)
