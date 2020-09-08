---
title: 'Información sobre el descuento por reserva: Servidor individual de Azure Database for PostgreSQL'
description: Aprenda a aplicar un descuento por reserva a un servidor individual de Azure Database for PostgreSQL
author: kummanish
ms.author: manishku
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 02/13/2020
ms.openlocfilehash: 51b2f4964c01efbfc82008134d47f09648a772ff
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88681981"
---
# <a name="how-a-reservation-discount-is-applied-to-azure-database-for-postgresql-single-server"></a>Aplicación de un descuento por reserva a un servidor individual Azure Database for PostgreSQL

Después de comprar capacidad reservada para un servidor individual Azure Database for PostgreSQL, el descuento por la reserva se aplica automáticamente a las bases de datos de servidores individuales PostgreSQL que coincidan con los atributos y la cantidad de la reserva. Una reserva solo cubre los costos de proceso de su servidor individual Azure Database for PostgreSQL. Por el almacenamiento y la administración de redes se le cobra según las tarifas normales.

## <a name="how-reservation-discount-is-applied"></a>Aplicación del descuento por reserva

Un descuento por reserva es para ***usarlo o perderlo***. Por lo tanto, si no tiene recursos coincidentes para ninguna hora, perderá una cantidad de reserva para esa hora. No se pueden arrastrar las horas reservadas no utilizadas.</br>

Al cerrar un recurso, el descuento por reserva se aplica automáticamente a otro recurso que coincida con el ámbito especificado. Si no se encuentran recursos coincidentes en el ámbito especificado, las horas reservadas se pierden.

## <a name="discount-applied-to-azure-database-for-postgresql-single-server"></a>Descuento aplicado al servidor individual Azure Database for PostgreSQL

El descuento de la capacidad reservada de un servidor individual Azure Database for PostgreSQL se aplica a la ejecución de un solo servidor PostgreSQL cada hora. La reserva que compra coincide con el uso de procesos que emite el servidor Azure Database for PostgreSQL en ejecución. Para los servidores individuales PostgreSQL que no se ejecutan durante una hora entera, la reserva se aplica automáticamente al otro servidor individual Azure Database for PostgreSQL que coincida con los atributos de reserva. El descuento se puede aplicar a servidores Azure Database for PostgreSQL que se están ejecutando simultáneamente. Si no tiene un servidor de PostgreSQL que se ejecute durante toda la hora que coincida con los atributos de reserva, no obtendrá todas las ventajas del descuento por la reserva para esa hora.

En los ejemplos siguientes se muestra cómo se aplica el descuento por la capacidad reservada de Azure Database for PostgreSQL en función del número de núcleos adquiridos y el momento de su ejecución.

* **Ejemplo 1**: compra una capacidad reservada de Azure Database for PostgreSQL para ocho núcleos virtuales. Si va a ejecutar un servidor Azure Database for PostgreSQL de 16 núcleos virtuales que coincida con el resto de los atributos de la reserva, se le cobrará el precio de pago por uso de 8 núcleos virtuales correspondiente al uso de procesos del servidor PostgreSQL y obtendrá el descuento por reserva durante una hora del uso de proceso del servidor PostgreSQL correspondiente a 8 núcleos virtuales.</br>

En el resto de estos ejemplos, supongamos que la capacidad reservada de Azure Database for PostgreSQL que compra es para una instancia de Azure Database for PostgreSQL de 16 núcleos virtuales y el resto de atributos de la reserva coinciden con los servidores individuales PostgreSQL en ejecución.

* **Ejemplo 2**: se ejecutan dos servidores individuales Azure Database for PostgreSQL con ocho núcleos virtuales cada hora. Se aplica el descuento por reserva para 16 núcleos virtuales al uso de proceso para ambos servidores Azure Database for PostgreSQL de 8 núcleos.

* **Ejemplo 3**: se ejecuta un servidor individual Azure Database for PostgreSQL de 16 núcleos virtuales desde la 1 p.m. a la 1:30 p.m. Se ejecuta un servidor individual Azure Database for PostgreSQL de 16 núcleos virtuales desde la 1:30 p.m. a las 2 p.m. Ambas están cubiertas por el descuento en la reserva.

* **Ejemplo 4**: se ejecuta un servidor individual Azure Database for PostgreSQL de 16 núcleos virtuales desde la 1 p.m. a la 1:45 p.m. Se ejecuta un servidor individual Azure Database for PostgreSQL de 16 núcleos virtuales desde la 1:30 p.m. a las 2 p.m. Se le cobra el precio de pago por uso por el solapamiento de 15 minutos. El descuento en la reserva se aplica al uso de proceso por el resto del tiempo.

Para obtener información sobre la aplicación de Azure Reservations en informes de uso de facturación, consulte el artículo [Información sobre el uso de Azure Reservations](https://docs.microsoft.com/azure/billing/billing-understand-reserved-instance-usage-ea).

## <a name="next-steps"></a>Pasos siguientes

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).
