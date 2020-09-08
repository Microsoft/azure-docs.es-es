---
title: Aplicación del descuento por reserva a Azure Cache for Redis | Microsoft Docs
description: Conozca cómo se aplica el descuento por reserva a las instancias de Azure Cache for Redis.
author: yegu-ms
manager: maiye
ms.service: cache
ms.topic: conceptual
ms.date: 01/22/2020
ms.author: yegu
ms.openlocfilehash: 10fedaf5680976ae93984c9b2c607882f650019d
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88213276"
---
# <a name="how-the-reservation-discount-is-applied-to-azure-cache-for-redis"></a>Aplicación del descuento por reserva a Azure Cache for Redis

Después de comprar capacidad reservada de Azure Cache for Redis, el descuento por reserva se aplica automáticamente a las instancias de caché que coincidan con los atributos y la cantidad de la reserva. Una reserva solo cubre los costos de proceso de su instancia de Azure Cache for Redis. Por el almacenamiento y la administración de redes se le cobra según las tarifas normales. La capacidad reservada solo está disponible para las memorias caché de [nivel premium](/https://docs.microsoft.com/azure/azure-cache-for-redis/quickstart-create-redis).

## <a name="how-reservation-discount-is-applied"></a>Aplicación del descuento por reserva

Un descuento por reserva es para ***usarlo o perderlo***. Por lo tanto, si no tiene recursos coincidentes para ninguna hora, perderá una cantidad de reserva para esa hora. No se pueden arrastrar las horas reservadas no utilizadas.

Al cerrar un recurso, el descuento por reserva se aplica automáticamente a otro recurso que coincida con el ámbito especificado. Si no se encuentran recursos coincidentes en el ámbito especificado, las horas reservadas se pierden.

## <a name="discount-applied-to-azure-cache-for-redis"></a>Descuento aplicado a Azure Cache for Redis

El descuento por capacidad reservada de Azure Cache for Redis se aplica a las cachés cada hora. La reserva comprada se hace coincidir con el uso de proceso que emiten las cachés en ejecución. Cuando estas cachés no se ejecutan durante la hora completa, la reserva se aplica automáticamente a otras cachés que coincidan con los atributos de la reserva. El descuento se puede aplicar a cachés que se ejecuten de forma simultánea. Si no tiene cachés que se ejecuten durante una hora completa que coincidan con los atributos de la reserva, no obtendrá todas las ventajas del descuento por reserva en esa hora.

En los ejemplos siguientes se muestra cómo se aplica el descuento por capacidad reservada de Azure Cache for Redis en función del número de cachés adquiridas y el momento de su ejecución.

* **Ejemplo 1**: Compra una capacidad reservada de Azure Cache for Redis para una caché de 6 GB. Si va a ejecutar una caché de 13 GB que coincida con el resto de los atributos de la reserva, se le cobrará el precio de pago por uso por 7 GB de su uso de proceso de Azure Cache for Redis y obtendrá el descuento por reserva durante una hora de uso de proceso de la caché de 6 GB.

En el resto de estos ejemplos, supongamos que la capacidad reservada de Azure Cache for Redis que compra es por una caché de 26 GB y el resto de atributos de la reserva coinciden con la caché en ejecución.

* **Ejemplo 2**: Ejecuta dos cachés de 13 GB durante una hora. El descuento por reserva de 26 GB se aplica al uso de proceso de ambas cachés.

* **Ejemplo 3**: Ejecuta una caché de 26 GB de 1 a 1.30 de la tarde. Ejecuta otra caché de 26 GB de 1:30 a 2 de la tarde. Ambas están cubiertas por el descuento en la reserva.

* **Ejemplo 4**: Ejecuta una caché de 26 GB de 1 a 1:45 de la tarde. Ejecuta otra caché de 26 GB de 1:30 a 2 de la tarde. Se le cobra el precio de pago por uso por el solapamiento de 15 minutos. El descuento en la reserva se aplica al uso de proceso por el resto del tiempo.

Para obtener información sobre la aplicación de Azure Reservations en informes de uso de facturación, consulte el artículo [Información sobre el uso de Azure Reservations](/azure/billing/billing-understand-reserved-instance-usage-ea).

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Ponerse en contacto con nosotros
Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).
