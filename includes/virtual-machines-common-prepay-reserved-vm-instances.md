---
author: yashesvi
ms.author: banders
ms.service: virtual-machines-windows
ms.topic: include
ms.date: 05/14/2019
ms.openlocfilehash: c7ff7ab0800449c2a3aa9d58bd036981caccaa1c
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65815514"
---
# <a name="prepay-for-virtual-machines-with-azure-reserved-vm-instances-ri"></a>Pago por adelantado de máquinas virtuales con instancias reservadas de VM de Azure (RI)

Pague las máquinas virtuales por adelantado y ahorre dinero con Azure Reserved Virtual Machine (VM) Instances. Para más información, consulte las [ofertas de Azure Reserved VM Instances](https://azure.microsoft.com/pricing/reserved-vm-instances/).

Puede comprar una instancia reservada de máquina virtual en [Azure Portal](https://portal.azure.com). Para comprar una instancia:

- Debe tener un rol de propietario al menos en suscripción Enterprise o de Pago por uso.
- En el caso de las suscripciones Enterprise, la opción **Agregar instancias reservadas** debe estar habilitada en el [portal de EA](https://ea.azure.com). O bien, si esa opción está deshabilitada, debe ser un administrador de EA en la suscripción.
- En el caso del programa Proveedor de soluciones en la nube (CSP), solo los agentes de administración o de ventas pueden adquirir reservas.

El descuento de la reserva se aplica automáticamente el número de máquinas virtuales en ejecución que coincidan con el ámbito y los atributos de la reserva. Puede actualizar el ámbito de la reserva a través de [Azure Portal](https://portal.azure.com), PowerShell, CLI o a través de la API.

## <a name="determine-the-right-vm-size-before-you-buy"></a>Determinación del tamaño adecuado de una máquina virtual antes de su adquisición

Antes de comprar una reserva, debe determinar el tamaño de la máquina virtual que necesita. Las siguientes secciones le ayudará a determinar el tamaño correcto de la máquina virtual.

### <a name="use-reservation-recommendations"></a>Uso de las recomendaciones de reserva

Puede usar las recomendaciones de reserva para ayudar a determinar las reservas de direcciones que debe comprar.

- Recomendaciones de compra y la cantidad recomendada se muestran al adquirir una instancia reservada de máquina virtual en Azure portal.
- Azure Advisor proporciona recomendaciones de compra para suscripciones individuales.  
- Puede usar las API para obtener recomendaciones de compra para el ámbito compartido y ámbito de la suscripción única. Para obtener más información, consulte [reservado recomendación de compra de instancia API para los clientes empresariales](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation).
- Para los clientes EA, recomendaciones de compra para compartido y los ámbitos de suscripción solo están disponibles con el [paquete de contenido de consumo de Azure Insights Power BI](/power-bi/service-connect-to-azure-consumption-insights).

### <a name="classic-vms-and-cloud-services"></a>Las máquinas virtuales clásicas y servicios en la nube

Instancias reservadas de máquina virtual automáticamente se aplican a ambas máquinas virtuales clásicas y servicios en la nube cuando se habilita la flexibilidad de tamaño de instancia. No existen cualquier SKU especial para las máquinas virtuales clásicas o servicios en la nube. La misma SKU de máquina virtual se aplican a ellos.

Por ejemplo, podría convertir máquinas virtuales clásicas o servicios en la nube para las máquinas virtuales basadas en Azure Resource Manager. En este ejemplo, el descuento de reserva se aplica automáticamente a los correspondientes a las máquinas virtuales. No hay ninguna necesidad de *exchange* una instancia reservada existente: se aplica automáticamente.

### <a name="analyze-your-usage-information"></a>Analizar la información de uso
Debe analizar la información de uso para ayudar a determinar qué debe comprar las reservas de direcciones.

Datos de uso están disponibles en el archivo de uso y la API. Utilizarlos juntos para determinar qué reserva de compra. Debe comprobar si hay instancias de máquina virtual que tienen un uso elevado de diario para determinar la cantidad de compra de reservas de direcciones.

Evitar la `Meter` subcategoría y `Product` los campos de datos de uso. No distinguen entre distintos tamaños de máquina virtual que usan premium storage. Si usa estos campos para determinar el tamaño de máquina virtual para la compra de reserva, puede comprar un tamaño erróneo. A continuación, no obtendrá el descuento de reserva que espera. En su lugar, consulte el `AdditionalInfo` campo en el archivo de uso o la API de uso para determinar el tamaño correcto de la máquina virtual.

### <a name="purchase-restriction-considerations"></a>Consideraciones sobre la restricción de compra

Las instancias reservadas de máquina virtual están disponibles para la mayoría de tamaños de máquina virtual con algunas excepciones. No se aplican descuentos de reserva para las máquinas virtuales siguientes:

- **Series de máquinas virtuales** -serie G, serie Av2 o serie.

- **Las máquinas virtuales en versión preliminar** -cualquier serie de máquinas virtuales o el tamaño que se encuentra en versión preliminar.

- **Nubes** -reservas no están disponibles para su compra en regiones de Alemania o China.

- **Cuota insuficiente** -una reserva en el ámbito de una sola suscripción debe tener una cuota de vCPU disponible en la suscripción para la nueva instancia reservada. Por ejemplo, si la suscripción de destino tiene un límite de cuota de 10 vCPU para la serie D, no podrá comprar una reserva para 11 instancias Standard_D1. La comprobación de cuota para las reservas incluye las máquinas virtuales ya implementadas en la suscripción. Por ejemplo, si la suscripción tiene una cuota de 10 vCPU para la serie D y tiene implementadas dos instancias standard_D1, podrá comprar una reserva para 10 instancias standard_D1 en esta suscripción. También puede [crear oferta de solicitud de aumento](../articles/azure-supportability/resource-manager-core-quotas-request.md) para resolver este problema.

- **Restricciones de capacidad** - en circunstancias excepcionales, límites de los tamaños de la compra de nuevas reservas para un subconjunto de la máquina virtual, debido a una capacidad baja en una región de Azure.

## <a name="buy-a-reserved-vm-instance"></a>Compra de una instancia reservada de máquina virtual

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los servicios** > **Reservations**.
3. Haga clic en **Agregar** para comprar una nueva reserva.
4. Rellene todos los campos obligatorios. Las instancias de máquina virtual en ejecución que coinciden con los atributos seleccionados cumplen los requisitos para obtener el descuento de la reserva. El número real de las instancias de máquina virtual que obtienen el descuento depende el ámbito y la cantidad seleccionada.

    | Campo      | DESCRIPCIÓN|
    |------------|--------------|
    |NOMBRE        |Nombre de esta reserva.|
    |Subscription|Suscripción que se usa para pagar la reserva. Los costos anticipados de la reserva se cobran en el método de pago de la suscripción. El tipo de suscripción debe ser Contrato Enterprise (números de oferta: MS-AZR-0017P o MS-AZR-0148P) o Pago por uso (números de oferta: MS-AZR-0003P o MS-AZR-0023P). Para una suscripción Enterprise, los cargos se deducen del saldo de compromiso monetario de la inscripción o se cobran como uso por encima del límite. Para la suscripción Pago por uso, los cargos se cobran en el método de pago de tarjeta de crédito o factura de la suscripción.|    
    |Scope       |El ámbito de la reserva puede cubrir una o varias suscripciones (ámbito compartido). Si selecciona: <ul><li>Suscripción única: el descuento de la reserva se aplica a las máquinas virtuales de esta suscripción. </li><li>Compartido: el descuento de la reserva se aplica a las máquinas virtuales en ejecución en cualquiera de las suscripciones en el contexto de facturación. Para los clientes Enterprise, el ámbito compartido es la inscripción e incluye todas las suscripciones que esta contiene. Para los clientes de Pago por uso, el ámbito compartido incluye todas las suscripciones de Pago por uso creadas por el administrador de la cuenta.</li></ul>|
    |Área    |Región de Azure que está cubierta por la reserva.|    
    |Tamaño de VM     |Tamaño de las instancias de máquina virtual.|
    |Optimizar para     |La opción de flexibilidad del tamaño de instancia de máquina virtual aplicará el descuento por la reserva a otras máquinas virtuales del mismo [grupo](https://aka.ms/RIVMGroups). La prioridad de capacidad da preferencia a la capacidad del centro de datos para las implementaciones. Esto ofrece una mayor confianza en su capacidad para iniciar instancias de máquinas virtuales cuando las necesite. La prioridad de capacidad solo está disponible si el ámbito de la reserva es de suscripción única. |
    |Término        |Un año o tres años.|
    |Cantidad    |Número de instancias que se compran dentro de la reserva. La cantidad es el número de instancias de máquina virtual en ejecución a las que se aplica el descuento de facturación. Por ejemplo, si tiene 10 máquinas virtuales Standard_D2 en la región Este de EE. UU., debería especificar 10 como cantidad para maximizar el beneficio de todas las máquinas en ejecución. |

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2PjmT]

## <a name="change-a-reservation-after-purchase"></a>Cambiar una reserva después de la compra

Puede realizar los siguientes tipos de cambios a una reserva después de la compra:

- Actualizar el ámbito de reserva
- Flexibilidad de tamaño de instancia (si procede)
- Propiedad

También puede dividir una reserva en más pequeños fragmentos y ya ha dividido las reservas de mezcla. Ninguno de los cambios que una nueva transacción comercial o cambie la fecha de finalización de la reserva.

No se puede realizar los siguientes tipos de cambios después de la compra, directamente:

- Región de una reserva existente
- SKU
- Cantidad
- Duration

Sin embargo, puede *exchange* una reserva si desea realizar cambios.

## <a name="cancellations-and-exchanges"></a>Cancelaciones e intercambios

Si necesita cancelar la reserva, podría aplicarse una tasa de terminación anticipada del 12 %. Los reembolsos se calculan conforme al precio de compra o el precio actual de la reserva, el que sea más bajo. Los reembolsos están limitados a 50 000 dólares al año. El reembolso que recibe es el saldo prorrateado restante menos la tarifa de terminación anticipada del 12 %. Para solicitar una cancelación, vaya a la hoja Reserva de Azure Portal y seleccione **Reembolso** para crear una solicitud de soporte técnico.

Si necesita cambiar la reserva de las instancias de máquinas virtuales reservadas a otra región, otro grupo de tamaños de máquina virtual u otro plazo, puede cambiarla por otra reserva de igual o mayor valor. La fecha de inicio del período de la nueva reserva no se extiende desde la reserva intercambiada. Al crear la nueva reserva empieza el período de 1 o 3 años. Para solicitar un intercambio, vaya a la reserva en Azure Portal y seleccione **Cambio** para crear una solicitud de soporte técnico.

Para obtener más información acerca de cómo las reservas de direcciones de exchange o un reembolso, consulte [intercambios de reserva y los reembolsos](../articles/billing/billing-azure-reservations-self-service-exchange-and-refund.md).

## <a name="need-help-contact-us"></a>¿Ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre cómo administrar una reserva, consulte [Administración de Azure Reservations](../articles/billing/billing-manage-reserved-vm-instance.md).
- Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:
    - [¿Qué es Azure Reservations?](../articles/billing/billing-save-compute-costs-reservations.md)
    - [Administración de reservas en Azure](../articles/billing/billing-manage-reserved-vm-instance.md)
    - [Información sobre cómo se aplica el descuento por la reserva](../articles/billing/billing-understand-vm-reservation-charges.md)
    - [Información sobre el uso de reservas para suscripciones de pago por uso](../articles/billing/billing-understand-reserved-instance-usage.md)
    - [Información sobre el uso de reservas para la inscripción Enterprise](../articles/billing/billing-understand-reserved-instance-usage-ea.md)
    - [Costos de software de Windows no incluidos con reservas](../articles/billing/billing-reserved-instance-windows-software-costs.md)
    - [Azure Reservations en el programa del Proveedor de soluciones en la nube (CSP) del Centro de partners](https://docs.microsoft.com/partner-center/azure-reservations)
