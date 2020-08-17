---
title: Descripción del descuento de reserva para Azure SQL Database | Microsoft Docs
description: Aprenda cómo se aplica un descuento de reserva en las bases de datos de Azure SQL Database en ejecución. El descuento se aplica a dichas bases de datos cada hora.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.topic: conceptual
ms.date: 06/11/2020
ms.author: banders
ms.openlocfilehash: 9398a68d384eb7acba15fb64e27d7a2672006bc6
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87461296"
---
# <a name="how-a-reservation-discount-is-applied-to-azure-sql-database"></a>Aplicación de un descuento de reserva en Azure SQL Database

Después de comprar capacidad reservada en Azure SQL Database, el descuento de reserva se aplica automáticamente a las instancias que coincidan con los atributos y la cantidad de la reserva. Una reserva cubre los costos de proceso de la instancia de SQL Database. Se le cobra por el software, el almacenamiento y la administración de redes según las tarifas normales. Puede cubrir los costos de licencia de SQL Database con [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/).

Tenga en cuenta que los descuentos de reserva no se aplican a Azure SQL Database sin servidor.

Para las instancias reservadas de máquina virtual, consulte el artículo de [información sobre el descuento en las instancias reservadas de máquina virtual de Azure](../manage/understand-vm-reservation-charges.md).

## <a name="how-reservation-discount-is-applied"></a>Aplicación del descuento por reserva

Un descuento de reserva significa "*usarlo o perderlo*". Por lo tanto, si no tiene recursos coincidentes para ninguna hora, perderá una cantidad de reserva para esa hora. No se pueden arrastrar las horas reservadas no utilizadas.

Al cerrar un recurso, el descuento por reserva se aplica automáticamente a otro recurso que coincida con el ámbito especificado. Si no se encuentran recursos coincidentes en el ámbito especificado, las horas reservadas se *pierden*.

## <a name="discount-applied-to-running-sql-databases"></a>Descuento aplicado a las base de datos SQL en ejecución

 El descuento sobre la capacidad reservada de SQL Database se aplica a las bases de datos SQL en ejecución por hora. La reserva que compra se compara con el uso de procesos que emiten las bases de datos SQL en ejecución. Si una base de datos SQL no se ejecuta durante una hora entera, su reserva se aplica automáticamente a otras bases de datos que coincidan con los atributos de reserva. El descuento se puede aplicar a bases de datos SQL en ejecución simultáneamente. Si no tiene bases de datos SQL que se ejecuten durante toda la hora y que coincidan con los atributos de reserva, no obtendrá todas las ventajas del descuento de reserva para esa hora.

En los ejemplos siguientes se muestra cómo se aplica el descuento en la capacidad reservada de SQL Database en función del número de núcleos adquiridos y su momento de ejecución.

- Escenario 1: compra capacidad reservada de SQL Database para una instancia de 8 núcleos. Ejecuta una instancia de SQL Database con 16 núcleos que coincide con el resto de los atributos de la reserva. Se le cobra el precio de pago por uso de 8 núcleos por el proceso de SQL Database. Obtiene el descuento en la reserva para una hora de uso de proceso para una instancia de SQL Database de 8 núcleos.

En el resto de estos ejemplos, suponemos que la capacidad reservada de SQL Database que compra es para una instancia de 16 núcleos y el resto de atributos de la reserva coinciden con las bases de datos SQL en ejecución.

- Escenario 2: ejecuta dos bases de datos SQL con 8 núcleos cada una durante una hora. Se aplica el descuento en la reserva para 16 núcleos al uso de proceso para ambas bases de datos SQL de 8 núcleos.
- Escenario 3: ejecuta una instancia de SQL Database de 16 núcleos desde las 13:00 hasta las 13:30. Ejecuta otra instancia de SQL Database de 16 núcleos de las 13:30 a las 14:00. Ambas están cubiertas por el descuento en la reserva.
- Escenario 4: ejecuta una instancia de SQL Database de 16 núcleos desde las 13:00 hasta las 13:45. Ejecuta otra instancia de SQL Database de 16 núcleos de las 13:30 a las 14:00. Se le cobra el precio de pago por uso por el solapamiento de 15 minutos. El descuento en la reserva se aplica al uso de proceso por el resto del tiempo.

Para obtener información sobre la aplicación de Azure Reservations en informes de uso de facturación, consulte el artículo [Información sobre el uso de Azure Reservations](understand-reserved-instance-usage-ea.md).

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Ponerse en contacto con nosotros

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:

- [¿Qué es Azure Reservations?](save-compute-costs-reservations.md)
- [Pago por adelantado de máquinas virtuales con Azure Reserved VM Instances](../../virtual-machines/windows/prepay-reserved-vm-instances.md)
- [Pago por adelantado de los recursos de proceso de SQL Database con capacidad reservada de Azure SQL Database](../../azure-sql/database/reserved-capacity-overview.md)
- [Administración de Azure Reservations](manage-reserved-vm-instance.md)
- [Información sobre el uso de reservas para suscripciones de pago por uso](understand-reserved-instance-usage.md)
- [Información sobre el uso de reservas para la inscripción Enterprise](understand-reserved-instance-usage-ea.md)
- [Información sobre el uso de reservas para suscripciones de CSP](/partner-center/azure-reservations)
