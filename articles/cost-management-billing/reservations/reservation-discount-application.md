---
title: Aplicación de un descuento por reserva de Azure
description: Este artículo le ayuda a entender cómo se aplican por lo general los descuentos de instancias reservadas.
author: bandersmsft
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 03/22/2020
ms.author: banders
ms.openlocfilehash: 1c038e896a9f9fc65c3b2201b3e763e966285c56
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88684684"
---
# <a name="how-a-reservation-discount-is-applied"></a>Aplicación de un descuento por reserva

Este artículo le ayuda a entender cómo se aplican por lo general los descuentos de instancias reservadas. El descuento por reserva se aplica a la utilización de recursos que coincide con los atributos seleccionados al comprar la reserva. Los atributos incluyen el ámbito en el que se ejecutan las máquinas virtuales, SQL Database, Azure Cosmos DB u otras ejecuciones de recursos. Por ejemplo, si quiere un descuento por reserva para las cuatro máquinas virtuales Estándar D2 en la región Oeste de EE. UU., seleccione la suscripción en la que se ejecutan las máquinas virtuales.

Un descuento de reserva significa "*usarlo o perderlo*". Si no tiene recursos coincidentes para ninguna hora, perderá una cantidad de reserva para esa hora. No se pueden arrastrar las horas reservadas no utilizadas.

Al cerrar un recurso, el descuento por reserva se aplica automáticamente a otro recurso que coincida con el ámbito especificado. Si no se encuentran recursos coincidentes en el ámbito especificado, las horas reservadas se *pierden*.

Por ejemplo, más adelante podría crear un recurso y tener una reserva que coincida con la que está infrautilizada. El descuento por reserva se aplica automáticamente al nuevo recurso coincidente.

Si, por el contrario, sus máquinas virtuales se ejecutan en diferentes suscripciones dentro de su cuenta o inscripción, seleccione el ámbito compartido. El ámbito compartido le permite aplicar el descuento de la reserva en varias suscripciones. También puede cambiar el ámbito después de la compra de la reserva. Para más información, consulte [Administración de Azure Reservations](manage-reserved-vm-instance.md).

Los descuentos por reserva solo se aplican a los recursos asociados con Enterprise, Microsoft Customer Agreement, CSP o a las suscripciones con tarifas de pago por uso. Los recursos que se ejecutan en una suscripción con otros tipos de oferta no recibirán el descuento por la reserva.

## <a name="when-the-reservation-term-expires"></a>Cuando expira el plazo de reserva

Al final del plazo de reserva, el descuento de facturación expira y los recursos se facturan según los precios de pago por uso. De forma predeterminada, las reservas no se establecen para renovarse automáticamente. Puede optar por habilitar la renovación automática de una reserva mediante la selección de la opción de la configuración de renovación. Con la renovación automática, se comprará una reserva de reemplazo cuando expire la reserva existente. De forma predeterminada, la reserva de reemplazo tiene los mismos atributos que la reserva que expira y, opcionalmente, puede cambiar la frecuencia de facturación, el plazo o la cantidad en la configuración de renovación. Cualquier usuario con acceso de propietario en la reserva y la suscripción que se usa para la facturación puede configurar la renovación.  

## <a name="discount-applies-to-different-sizes"></a>El descuento se aplica a los diferentes tamaños

Al comprar una reserva, puede aplicar el descuento a otras instancias con atributos que están dentro del mismo grupo de tamaño. Esta característica se conoce como flexibilidad de tamaño de instancia. La flexibilidad de la cobertura del descuento depende del tipo de reserva y los atributos que elija al comprar la reserva.

Planes de servicio:

- Instancias reservadas de máquina virtual: al comprar la reserva y seleccionar **Optimizado para: flexibilidad de tamaño de instancia**, la cobertura de descuento dependerá del tamaño de máquina virtual que elija. La reserva se puede aplicar a los tamaños de máquinas virtuales (VM) en el mismo grupo de series de tamaño. Si quiere saber más, vea [Flexibilidad en el tamaño de las máquinas virtuales con Azure Reserved VM Instances](../../virtual-machines/reserved-vm-instance-size-flexibility.md).
- Capacidad reservada de Azure Storage: puede adquirir capacidad reservada para cuentas de Azure Storage estándar en unidades de 100 TiB o de 1 PiB al mes. Para más información sobre qué regiones admiten la capacidad reservada de Azure Storage, consulte [Precios de los blobs en bloques](https://azure.microsoft.com/pricing/details/storage/blobs/). La capacidad reservada de Azure Storage está disponible para cualquier nivel de acceso (frecuente, esporádico o archivo) y para cualquier configuración de replicación (LRS, GRS o ZRS).
- Capacidad reservada de SQL Database: la cobertura del descuento depende del nivel de rendimiento elegido. Para más información, vea [Aprenda cómo se aplica un descuento en la reserva de Azure a las instancias de SQL Database](understand-reservation-charges.md).
- Capacidad reservada de Azure Cosmos DB: la cobertura del descuento depende del rendimiento aprovisionado. Para más información, vea [Aplicación del descuento por reserva a Azure Cosmos DB](understand-cosmosdb-reservation-charges.md).

## <a name="how-discounts-apply-to-specific-azure-services"></a>Aplicación de descuentos a servicios específicos de Azure

Lea los siguientes artículos que correspondan a su caso para saber cómo se aplican los descuentos a un servicio de Azure específico:

- [App Service](reservation-discount-app-service-isolated-stamp.md)
- [Azure Cache for Redis](understand-azure-cache-for-redis-reservation-charges.md)
- [Cosmos DB](understand-cosmosdb-reservation-charges.md)
- [Database for MariaDB](understand-reservation-charges-mariadb.md)
- [Database for MySQL](understand-reservation-charges-mysql.md)
- [Database for PostgreSQL](understand-reservation-charges-postgresql.md)
- [Databricks](reservation-discount-databricks.md)
- [Data Explorer](understand-azure-data-explorer-reservation-charges.md)
- [Hosts dedicados](billing-understand-dedicated-hosts-reservation-charges.md)
- [Disk Storage](understand-disk-reservations.md)
- [Red Hat Linux Enterprise](understand-rhel-reservation-charges.md)
- [Planes de software](understand-suse-reservation-charges.md)
- [Storage](understand-storage-charges.md)
- [SQL Database](understand-reservation-charges.md)
- [Azure Synapse Analytics](reservation-discount-azure-sql-dw.md)
- [Máquinas virtuales](../manage/understand-vm-reservation-charges.md)


## <a name="next-steps"></a>Pasos siguientes

- [Administración de Azure Reservations](manage-reserved-vm-instance.md)
- [Información sobre el uso de reservas de Azure para suscripciones de pago por uso](understand-reserved-instance-usage.md)
- [Información sobre el uso de reservas para la inscripción Enterprise](understand-reserved-instance-usage-ea.md)
- [Costos de software de Windows no incluidos con reservas](reserved-instance-windows-software-costs.md)