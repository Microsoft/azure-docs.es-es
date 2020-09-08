---
title: Información sobre el descuento de Azure Reserved VM Instances
description: Obtenga información sobre cómo se aplica el descuento de Azure Reserved VM Instance a las máquinas virtuales en ejecución.
author: yashesvi
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: dd21faa07426c80bb3a95dbd1b6d4ee93f783881
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88686452"
---
# <a name="how-the-azure-reservation-discount-is-applied-to-virtual-machines"></a>Aplicación del descuento por reserva de Azure en las máquinas virtuales

Después de comprar una instancia reservada de máquina virtual de Azure, el descuento de reserva se aplica automáticamente a las máquinas virtuales que coincidan con los atributos y la cantidad de la reserva. Una reserva cubre los costos de proceso de las máquinas virtuales.

Un descuento de reserva se aplica a las máquinas virtuales base que compre en Azure Marketplace.

Para la capacidad reservada de SQL Database, consulte [Información sobre el descuento de instancias reservadas de Azure](../reservations/understand-reservation-charges.md).

En la tabla siguiente se muestran los costos de la máquina virtual después de comprar una instancia reservada de máquina virtual. En cualquier caso, se le cobrará por el almacenamiento y la administración de redes según las tarifas normales.

| Tipo de máquina virtual  | Cargos con instancia reservada de máquina virtual |
|-----------------------|--------------------------------------------|
|Máquinas virtuales con Linux sin software adicional | La reserva cubre los costos de infraestructura de la máquina virtual.|
|Máquinas virtuales con Linux con cargos de software (por ejemplo, Red Hat) | La reserva cubre los costos de infraestructura. Se le cobra por el software adicional.|
|Máquinas virtuales con Windows sin software adicional |La reserva cubre los costos de infraestructura. Se le cobra por el software de Windows.|
|Máquinas virtuales con Windows con software adicional (por ejemplo, SQL Server) | La reserva cubre los costos de infraestructura. Se le cobra por el software de Windows y por el software adicional.|
|Máquinas virtuales con Windows con la [Ventaja híbrida de Azure](../../virtual-machines/windows/hybrid-use-benefit-licensing.md) | La reserva cubre los costos de infraestructura. Los costos de software de Windows están cubiertos por la Ventaja híbrida de Azure. El software adicional se cobra por separado.|

## <a name="how-reservation-discount-is-applied"></a>Aplicación del descuento por reserva

Un descuento de reserva significa "*usarlo o perderlo*". Por lo tanto, si no tiene recursos coincidentes para ninguna hora, perderá una cantidad de reserva para esa hora. No se pueden arrastrar las horas reservadas no utilizadas.

Al cerrar un recurso, el descuento por reserva se aplica automáticamente a otro recurso que coincida con el ámbito especificado. Si no se encuentran recursos coincidentes en el ámbito especificado, las horas reservadas se *pierden*.

## <a name="reservation-discount-for-non-windows-vms"></a>Descuento de reserva para las máquinas virtuales que no son Windows

 El descuento de reserva de Azure se aplica a las instancias de máquinas virtuales en ejecución en períodos de una hora. Las reservas que haya comprado se vinculan con el uso emitido por las máquinas virtuales en ejecución para aplicar el descuento de reserva. Para las máquinas virtuales que no se pueden ejecutar la hora completa, se rellenará la reserva desde otras máquinas virtuales que no usen una reserva, incluidas las que se ejecutan simultáneamente. Al final de la hora, se bloquea la aplicación de reserva para las máquinas virtuales en dicha hora. En el caso de que una máquina virtual no se ejecute durante una hora o que haya máquinas virtuales simultáneas dentro de la hora que no llenen la hora de reserva, la reserva estará infrautilizada durante esa hora. En el siguiente gráfico se muestra la aplicación de una reserva a un uso de máquina virtual facturable. La ilustración se basa en una compra de reserva y dos instancias de máquina virtual coincidentes.

![Captura de pantalla de una reserva aplicada y dos instancias de máquina virtual coincidentes](./media/understand-vm-reservation-charges/billing-reserved-vm-instance-application.png)

1. Cualquier uso por encima de la línea de reservas se cobrará según las tarifas normales de pago por uso. No se le cobrará por ningún uso por debajo de la línea de reservas, ya que el pago se ha hecho como parte de la compra de la reserva.
2. En la hora 1, la instancia 1 se ejecuta durante 0,75 horas y la instancia 2 se ejecuta durante 0,5 horas. El uso total de la hora 1 es de 1,25 horas. Se le cobrará según las tarifas de pago por uso para las 0,25 horas restantes.
3. Para las horas 2 y 3, ambas instancias se ejecutaron durante 1 hora cada una. Una instancia está cubierta por la reserva, mientras que la otra se cobra según las tarifas de pago por uso.
4. Durante la hora 4, la instancia 1 se ejecuta durante 0,5 horas y la instancia 2 se ejecuta durante 1 hora. La instancia 1 está cubierta al completo por la reserva y 0,5 horas de la instancia 2 también. Se le cobrará según la tarifa de pago por uso por las 0,5 horas restantes.

Para comprender y ver la aplicación de Azure Reservations en los informes de uso de facturación consulte [Información sobre el uso de reservas](../reservations/understand-reserved-instance-usage-ea.md).

## <a name="reservation-discount-for-windows-vms"></a>Descuento de reserva para máquinas virtuales Windows

Cuando ejecuta instancias de máquina virtual Windows, la reserva se aplica para cubrir los costos de infraestructuras. La aplicación de la reserva a los costos de infraestructuras de máquinas virtuales con Windows es la misma que para las máquinas virtuales que no ejecutan Windows. Se le cobra por separado por el software de Windows según el número de vCPU. Consulte los [costos de software de Windows con Reservations](../reservations/reserved-instance-windows-software-costs.md). Puede cubrir los costos de licencia de Windows con la [Ventaja híbrida de Azure para Windows Server](../../virtual-machines/windows/hybrid-use-benefit-licensing.md).

## <a name="discount-can-apply-to-different-sizes"></a>Se puede aplicar el descuento a los distintos tamaños

Si se compra una instancia reservada de máquina virtual y selecciona **Optimized for instance size flexibility** (Optimización en la flexibilidad de tamaño de la instancia), la cobertura del descuento se aplica al tamaño de máquina virtual que seleccione. También se puede aplicar a otros tamaños de máquina virtual que se encuentren en el mismo grupo de flexibilidad de tamaño de instancia de la serie. Si quiere saber más, vea [Flexibilidad en el tamaño de las máquinas virtuales con Azure Reserved VM Instances](../../virtual-machines/windows/reserved-vm-instance-size-flexibility.md).

## <a name="premium-storage-vms-dont-get-non-premium-discounts"></a>Las máquinas virtuales de Premium Storage no obtienen descuentos no Premium

Este es un ejemplo. Suponga que ha comprado una reserva para cinco máquinas virtuales Estándar_D1. el descuento de la reserva solo se aplica a las máquinas virtuales Estándar_D1 o a otras máquinas virtuales de la misma familia de instancias. El descuento no se aplica a la máquina virtual Estándar_DS1 o a otros tamaños del grupo de flexibilidad de tamaño de instancia DS1.

La aplicación de descuento de la reserva omite el medidor que se utiliza para las máquinas virtuales y solo evalúa ServiceType. Observe el valor `ServiceType` de `AdditionalInfo` para determinar la información de la serie o grupo de flexibilidad de la instancia para sus máquinas virtuales. Los valores se encuentran en el archivo .csv de uso.

La serie o grupo de flexibilidad de la instancia no se puede cambiar directamente después de la compra. Sin embargo, puede *cambiar* una reserva de máquina virtual de una serie o un grupo de flexibilidad de instancia a otro.

## <a name="services-that-get-vm-reservation-discounts"></a>Servicios que obtienen descuentos de reserva de máquina virtual

Puede aplicar sus reservas de máquina virtual al uso de máquinas virtuales emitidas desde varios servicios, no solo a sus implementaciones de máquinas virtuales. Los recursos que obtienen descuentos de reserva cambian en función de la configuración de flexibilidad de tamaño de instancia.

### <a name="instance-size-flexibility-setting"></a>Configuración de flexibilidad de tamaño de instancia

La configuración de flexibilidad de tamaño de instancia determina qué servicios obtienen los descuentos de la instancia reservada.

Tanto si la configuración está activada como si no, los descuentos de reserva se aplican automáticamente a cualquier uso de máquina virtual que coincida cuando *ConsumedService* sea `Microsoft.Compute`. Por lo tanto, compruebe los datos de uso del valor *ConsumedService*. Estos son algunos ejemplos:

- Máquinas virtuales
- Conjuntos de escalado de máquinas virtuales
- Servicio de contenedor
- Implementaciones de Azure Batch (en modo de suscripciones de usuario)
- Azure Kubernetes Service (AKS)
- Service Fabric

Cuando la configuración está activada, los descuentos de reserva se aplican automáticamente al uso de máquina virtual que coincida cuando *ConsumedService* sea alguno de los elementos siguientes:

- Microsoft.Compute
- Microsoft.ClassicCompute
- Microsoft.Batch
- Microsoft.MachineLearningServices
- Microsoft.Kusto

Compruebe el valor *ConsumedService* en los datos de uso para determinar si el uso puede optar a los descuentos de reserva.

Para más información sobre la flexibilidad de tamaño de instancia, vea [Flexibilidad en el tamaño de las máquinas virtuales con Azure Reserved VM Instances](../../virtual-machines/reserved-vm-instance-size-flexibility.md).


## <a name="need-help-contact-us"></a>¿Necesita ayuda? Ponerse en contacto con nosotros

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:

- [¿Qué son las reservas para Azure?](../reservations/save-compute-costs-reservations.md)
- [Pago por adelantado de máquinas virtuales con Azure Reserved VM Instances](../../virtual-machines/windows/prepay-reserved-vm-instances.md)
- [Pago por adelantado de los recursos de proceso de SQL Database con capacidad reservada de Azure SQL Database](../../azure-sql/database/reserved-capacity-overview.md)
- [Administración de reservas para Azure](../reservations/manage-reserved-vm-instance.md)
- [Información sobre el uso de reservas para suscripciones de pago por uso](../reservations/understand-reserved-instance-usage.md)
- [Información sobre el uso de reservas para la inscripción Enterprise](../reservations/understand-reserved-instance-usage-ea.md)
- [Información sobre el uso de reservas para suscripciones de CSP](/partner-center/azure-reservations)
- [Costos de software de Windows no incluidos con reservas](../reservations/reserved-instance-windows-software-costs.md)
